Please run the file index.html to run the program.

Pathway Studio - AI Career Pathway Planner
Pathway Studio is a browser based prototype that uses AI to help high school students explore 4 year university pathways at UH Mānoa. Students fill out a short profile, then the app asks OpenAI to generate:
•	A semester by semester course plan for 4 years (Fall and Spring)
•	Related co curricular activities and clubs
•	Potential careers aligned with the pathway
All logic runs in the browser. The only external service is the OpenAI API.
1. File structure
The prototype consists of three files:
•	index.html
Profile input page where the student selects grade, interests, dream career, strengths, and long term goals. The profile is saved in sessionStorage.
•	pathways.html
Pathway board that calls OpenAI using the saved profile and renders:
o	4 year course plan by semester
o	Activities and clubs column
o	Potential careers column
•	config.js
Holds the OpenAI API key as window.OPENAI_API_KEY.
2. Data flow
1.	Student opens index.html.
2.	Student fills in the profile form and submits.
3.	The form handler serializes the profile as JSON and stores it in sessionStorage under the key studentProfile.
4.	The app redirects the browser to pathways.html.
5.	pathways.html loads the profile from sessionStorage, displays a summary, and sends a prompt to the OpenAI Chat Completions API.
6.	OpenAI returns structured JSON with:
o	years 4 objects, one per year, with fall and spring courses
o	activities list of activities and clubs
o	careers list of potential careers
7.	The page parses the JSON and renders clickable bubbles for each course, activity, and career.
3. Student profile (page 1)
3.1 Form fields
On index.html, the profile form includes:
•	Grade in high school
o	Single select: 9, 10, 11, 12
•	Main interest areas
o	Multi select: STEM and computing, life science and health, business and economics, social science and education, arts and media
•	Dream career
o	Single select from Occupational Outlook Handbook style titles
(for example: software developers, registered nurses, social workers)
•	Main strengths
o	Multi select: strong math skills, enjoys technology, creativity and design, leadership, etc.
•	Long term goals
o	Multi select: high income, work life balance, helping community, research and innovation, and so on.
3.2 Storing the profile
When the user submits the form:
const profile = {
  grade,
  interests,     // array of interest codes such as ["stem", "business"]
  dreamCareer,   // string, for example "software developers"
  strengths,     // array of strings
  goals          // array of strings
};

sessionStorage.setItem("studentProfile", JSON.stringify(profile));
window.location.href = "pathways.html";

The profile only exists in the browser. It is not sent anywhere except inside the OpenAI request from the second page.

4. Pathway board (page 2)
pathways.html is where AI suggestions are generated and displayed.
4.1 Loading the profile
On load:
1.	The script reads sessionStorage.getItem("studentProfile").
2.	If it is missing or invalid, the page shows a message and disables the generate button.
3.	If it is valid, the summary in the left card shows:
o	Grade
o	Interests (mapped from codes such as "stem" to labels such as "STEM and computing")
o	Dream career
o	Main strengths
o	Long term goals
There are also quick edit selects for primary interest and dream career. When those change, the profile in sessionStorage is updated.
4.2 Layout and columns
The page has two main sections:
1.	Left: Snapshot summary card
o	Profile summary
o	Note about advising
o	"Generate suggestions" button and status text
2.	Right: Courses, activities, and careers board
o	Top legend explaining colors for:
	Courses
	Activities and clubs
	Careers
o	Main grid:
	4 columns for Year 1 to Year 4
	Each year has:
	Fall semester course bubbles
	Spring semester course bubbles
o	Extra grid:
	Column for activities and clubs (bubbles)
	Column for potential careers (bubbles)
o	Detail panel at the bottom:
	Shows description of the selected bubble
4.3 Bubble interactions
•	Each course, activity, or career is rendered as a button with:
o	data-kind attribute: "course", "activity", or "career"
o	data-title
o	data-description
•	On click:
o	The detail panel shows:
	Prefix label such as "Course:", "Activity:", or "Career:"
	The title and description
o	The app highlights related bubbles by scanning titles for overlapping keywords (very lightweight semantic linking).

5. OpenAI integration
The second page calls the OpenAI Chat Completions API using fetch.
5.1 Configuration
config.js must define:
window.OPENAI_API_KEY = "sk-...";  // Replace with a real key in development

Important:
•	Never commit the real key to a public repository.
•	In a production setup, this call should be proxied through a backend.
5.2 System prompt
buildSystemPrompt() defines the AI role and the required JSON structure. The key points:
•	Uses the UH Mānoa 2024 to 2025 course overview.
•	Uses the Occupational Outlook Handbook A to Z index.
•	Must design a four year plan with related courses that follow realistic sequences.
•	Must provide activities and clubs that support the pathway.
•	Must provide potential careers aligned with the pathway.
•	Must return only JSON with the following structure:
{
  "years": [
    {
      "year": 1,
      "fall": [
        { "title": "ICS 111 - Introduction to Computer Science I",
          "description": "..." }
      ],
      "spring": [
        { "title": "ICS 141 - Discrete Mathematics for Computer Science I",
          "description": "..." }
      ]
    },
    { "year": 2, "fall": [], "spring": [] },
    { "year": 3, "fall": [], "spring": [] },
    { "year": 4, "fall": [], "spring": [] }
  ],
  "activities": [
    { "title": "UH Mānoa ACM student chapter", "description": "..." }
  ],
  "careers": [
    { "title": "Software developers", "description": "..." }
  ]
}


Notes:
years must contain exactly four objects.
Each fall and spring array should list at least four course objects.
Each item must have both title and description.
No explanation text outside JSON is allowed.

5.3 User prompt
buildUserPrompt(profile) converts the student profile into text such as:
Student profile:
Grade: 11
Interest area: STEM and computing
Dream career: Software developers
Main strength: Strong math skills
Long term goal: High income
Using this profile, propose a four year bachelor program with Fall and Spring semesters and return JSON as requested.

This helps the model align the 4 year plan, activities, and careers with the student profile.
5.4 Calling the API
Core call:
const payload = {
  model: "gpt-4.1-mini",
  messages: [
    { role: "system", content: systemContent },
    { role: "user", content: userContent }
  ],
  temperature: 0.7,
  max_tokens: 1500
};

const response = await fetch("https://api.openai.com/v1/chat/completions", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer " + apiKey
  },
  body: JSON.stringify(payload)
});

The code then:
1.	Reads data.choices[0].message.content.
2.	Tries to JSON.parse the entire content.
3.	If parsing fails, tries to extract the substring between the first { and the last } and parses that.
4.	Returns the resulting plan object.
6. Rendering logic
6.1 Clearing the board
clearBoard() removes all bubbles from:
•	y1-fall, y1-spring
•	y2-fall, y2-spring
•	y3-fall, y3-spring
•	y4-fall, y4-spring
•	activities-group
•	careers-group
and resets the detail panel text.
6.2 Mapping years
renderBoard(plan) expects:
const groupIds = {
  1: { fall: "y1-fall", spring: "y1-spring" },
  2: { fall: "y2-fall", spring: "y2-spring" },
  3: { fall: "y3-fall", spring: "y3-spring" },
  4: { fall: "y4-fall", spring: "y4-spring" }
};

For each yearObj in plan.years:
•	yearNum = yearObj.year || (index + 1)
•	Courses in yearObj.fall go into the corresponding fall group.
•	Courses in yearObj.spring go into the corresponding spring group.
6.3 Activities and careers
For plan.activities:
•	Each object { title, description } creates a bubble with data-kind="activity" and is appended to activities-group.
For plan.careers:
•	Each object { title, description } creates a bubble with data-kind="career" and is appended to careers-group.

7. Running the prototype locally
1.	Save the three files in the same folder:
o	index.html
o	pathways.html
o	config.js
2.	Edit config.js and set your OpenAI key:
window.OPENAI_API_KEY = "sk-...";  // development only
3.	Open index.html in a modern browser (Chrome, Edge, Firefox).
4.	Fill out the student profile and click "Build pathway board".
5.	On the second page, wait for "Suggestions updated." in the status text and explore the bubbles.
If you run into errors:
•	Check the browser console for messages such as:
o	"OPENAI_API_KEY is not defined in config.js"
o	"Response was not valid JSON"
o	"API error: 401" or other status codes
•	Make sure your key is active and has access to gpt-4.1-mini.
8. Customization ideas
We can extend this prototype by:
•	Allowing the user to choose:
o	Target major or college within UH Mānoa
o	4 or 5 year timeline
•	Adding constraints:
o	Minimum and maximum course load per semester
o	Required general education categories
•	Storing and reloading multiple plans per student profile
•	Exporting the generated plan as PDF or a simple text file for advising sessions
This documentation should give us enough structure to explain the app to collaborators, students, or advisors, and to guide future code changes.

