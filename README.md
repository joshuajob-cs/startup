# Psychic Questions

[My Notes](notes.md)

Psychic Questions is a web application that provides a fun activity for a group of people to engage and connect with one another. Each person is asked four questions. Then they must guess everyone else's responses. The guesser goes through the four questions one at a time, and recieves feedback on what the correct answer was each time. For the first question they rely on random guesswork, but on the fourth question they can make an educated "psychic" guess based on the other three responses. Players win points by guessing correctly.

![Guessing how another player answered](docs/gameplay.png)

## How it's built

**Stack** — React 19 + Vite, Node/Express, WebSocket (`ws`), MongoDB, Jest + Supertest.

**Phase sync** — the server owns the game phase (lobby → answering → guessing → winner) and broadcasts every transition over WebSocket, so every player's screen advances together instead of polling. [`service/websocket.js`](service/websocket.js) broadcasts; [`usePhaseChange`](client/src/hooks/usePhaseChange.js) is what each screen subscribes with.

**Hosting** — ran on AWS EC2 behind a custom domain at about $11/month. I priced a downgrade to a smaller instance at ~$4.75/month ([AWS_COSTS.md](AWS_COSTS.md)), but re-platforming to Fly.io beat it: machines idle down to zero, leaving the $3/year domain as the only standing cost.

## Course deliverables

The rest of this README is the per-deliverable log from CS 260, kept as written.

### 🚀 Specification Deliverable

For this deliverable I did the following:

- [x] Proper use of Markdown
- [x] A concise and compelling elevator pitch
- [x] Description of key features
- [x] Description of how you will use each technology
- [x] One or more rough sketches of your application. Images must be embedded in this file using Markdown image references.

#### Elevator pitch

Psychic questions is a sensational get-to-know-you game that you can play with your friends or classmates on any device. Is your friend a nerd? A jock? An actor? Some weird mesh of all 3? Everything you learn about them gets you deeper into their head, strengthening your psychic powers! Now you know everything about them! Or _do_ you...

#### Design

One user must create an account and sign in to start the game. All of the other players can join with the join code. Each player can see a list of all of the players who have joined. Players must type in their name to join the game.

<img src=docs/Login.jpg alt="Login screen has option to login with username and password to create a new game, login with code to join a game, or sign up if you do not have an account yet." height="300"/> <img src=docs/Join.jpg alt="When someone logs in either to start a game or join a game, a game code is displayed along with the number of players that have joined and a list of their names." height="300"/> <img src=docs/Join2.jpg alt="Each player must type in their name as they join." height="300"/>

Each player must answer four questions about themselves to start off the game.

<img src=docs/AnswerYourself.jpg alt="At the beggining of the game four questions are displayed one at a time for each player to answer." height="500"/>

Each player will recieve the same questions as another player and try to guess what that player said. The questions are given in a random order, so the guesser can not leave the easy ones for last. Each time the guesser guesses they recieve immediate feedback on what the correct answer is. The idea is that the first response they must guess will be a shot in the dark, but for the remaining ones they will be able to make an educated guess based on the first three responses. As the guesser recieves each question the screen will become scrollable, so that it can all fit on the screen at once.

Question 1: 5 points  
Question 2: 10 points  
Question 3: 25 points  
Question 4: 50 points

<img src=docs/GuessOthersAnswers.jpg alt="On this screen a player is trying to guess how Jake responded to a question. The player has four possible answer choices. The player's points are displayed in the upper right corner. The screen is scrollable after the player has answered their first question, so that they can look at the questions they have already answered.]" height="400"/> <img src=docs/GuessOthersAnswers2.jpg alt="On this screen, a player has guessed how Jake responded to a question, and the correct answer is displayed." height="400"/>

Whomever gets the most points wins!

<img src="docs/Winner.jpg" alt="A screen is shown with the name of the winner, how many times they were able to answer the fourth question correctly, and how many total points they had." height="500"/>

In the sequence diagram each player interacts with the server to send in and recieve questions/responses.

```mermaid
sequenceDiagram
    actor Charles
    actor Jenny
    actor Tasha
    actor Service
    actor Server
    Charles->>Service: Responses to questions
    Jenny->>Service: Responses to questions
    Tasha->>Service: Responses to questions
    Service->>Server: Responses formatted for server
    Server->>Service: Responses randomized
    Service->>Charles: Questions to guess the response to
    Service->>Jenny: Questions to guess the response to
    Service->>Tasha: Questions to guess the response to
```

#### Key features

- Login authentication screen with securely stored password
- Join code allows each player to join
- Join screen shows number of players who have joined
- Waiting screen is displayed when waiting for other players responses
- Asks users questions and sends their names and their responses to a server
- Each player recieves the questions and answers of other players from the server
- Randomizes questions and possible responses
- Point system

#### Technologies

I am going to use the required technologies in the following ways.

- **HTML** - Structure each screen (login screen, question screen, guessing screen).
- **CSS** - Make each screen look good (login screen, question screen, guessing screen).
- **React** - Users type in username and password, click to navigate between screens, scroll to look at all of the avaliable information on a screen, type in answers to questions, and click to select the correct response to a question.
- **Service** - Information is formatted correctly before being stored in server. Questions and answers are randomized.
- **DB/Login** - Securely stores credentials in database. Stores questions and answers for each individual. Stores point values of each player connecte dto their name.
- **WebSocket** - Broadcast message when users first enter the game, when users finish answering the questions, and when users finish guessing, so that everybody stays on the same page and goes from one phase to the next at the same time.

### 🚀 AWS deliverable

For this deliverable I did the following. I checked the box `[x]` and added a description for things I completed.

- [x] **Server deployed and accessible with custom domain name** - [My server link](https://joshuajob-cs.click) (deployment since retired — see the hosting note above).

### 🚀 HTML deliverable

For this deliverable I did the following. I checked the box `[x]` and added a description for things I completed.

- [x] **HTML pages** - I wrote HTML for each page of the application.
- [x] **Proper HTML element usage** - I used BODY, NAV, MAIN, HEADER, FOOTER and other HTML tags appropriately.
- [x] **Links** - Links to travel to each page of the application.
- [x] **Text** - Questions for the user.
- [x] **3rd party API placeholder** - I will use a 3rd party API that determines whose answers were similar to eachother. When someone is guessing the reponses of the person who is most like them, they get a "similarity score" that tells them how closely related their answers are.
- [x] **Images** - Images to inspire you.
- [x] **Login placeholder** - Created login page with username and password.
- [x] **DB data placeholder** - Player scores and responses stored in database.
- [x] **WebSocket placeholder** - Websocket used to know how many other players are joined. Sends a message as soon as they join.

### 🚀 CSS deliverable

For this deliverable I did the following. I checked the box `[x]` and added a description for things I completed.

- [x] **Visually appealing colors and layout. No overflowing elements.** - Resizes to make sure nothing overflows.
- [x] **Use of a CSS framework** - Dropdown menu.
- [x] **All visual elements styled using CSS** - Very beautiful and mysterious.
- [x] **Responsive to window resizing using flexbox and/or grid display** - Used flex on most pages and grid for players entering their names.
- [x] **Use of a imported font** - Imported two fonts.
- [x] **Use of different types of selectors including element, class, ID, and pseudo selectors** - Used all of these throughut the application. Used the hover pseudo selector.

### 🚀 React part 1: Routing deliverable

For this deliverable I did the following. I checked the box `[x]` and added a description for things I completed.

- [x] **Bundled using Vite** - The whole application uses Vite.
- [x] **Components** - Has all of the CSS components from earlier and the HTML jas been deleted and replaced with JSX
- [x] **Router** - See routing in app.jsx

### 🚀 React part 2: Reactivity deliverable

For this deliverable I did the following. I checked the box `[x]` and added a description for things I completed.

- [x] **All functionality implemented or mocked out** - Storage is mocked out. Checks if you have the correct join code. The username and name you type in are stored for the rest of the app. Your score is stored. Websocket is mocked out with players being added and the jsx handling it correctly. Questions page correctly switches between questions, and answers page correctly adds new answers. Checks if your answer is correct and gives you points accordingly. Compares your points to anotehr player to see if you won.
- [x] **Hooks** - I added a lot of useState hooks for all of the UI componemts that change. I added effectState hooks for things that need to be done asynchronously. The effectState hooks are used with storage and Websocket mock ups.

### 🚀 Service deliverable

For this deliverable I did the following. I checked the box `[x]` and added a description for things I completed.

- [x] **Node.js/Express HTTP service** - Uses node.js and express.
- [x] **Static middleware for frontend** - app.use(express.static("public")); puts frontend on same port as backend.
- [x] **Calls to third party endpoints** - Random facts at the bottom of the waiting page.
- [x] **Backend service endpoints** - Endpoints for authentication, game data, and player responses.
- [x] **Frontend calls service endpoints** - API for authentication, game data, and player responses called by frontend.
- [x] **Supports registration, login, logout, and restricted endpoint** - All of these are supported and requireSession middleware creates restricted endpoints.

### 🚀 DB deliverable

For this deliverable I did the following. I checked the box `[x]` and added a description for things I completed.

- [x] **Stores data in MongoDB** - Stores game data in MongoDB, uses debouncing to keep database up to date with server
- [x] **Stores credentials in MongoDB** - Username and password are stored persistently after server shut down

### 🚀 WebSocket deliverable

For this deliverable I did the following. I checked the box `[x]` and added a description for things I completed.

- [x] **Backend listens for WebSocket connection** - service/websocket.js listens for names from a client and then sends them to all other clients in the game
- [x] **Frontend makes WebSocket connection** - apis/websocket.js send names from the client
- [x] **Data sent over WebSocket connection** - Names are sent over websocket connection
- [x] **WebSocket data displayed** - Names are displayed on the screen
- [x] **Application is fully functional** - Application looks great!

_Except where otherwise noted, this project is licensed under the MIT License. The Elephant.png and favicon.ico files are not covered under the MIT license. I want to reserve the rights of the elephant I drew for my own logos/branding._
