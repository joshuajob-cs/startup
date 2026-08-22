# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Psychic Questions is a real-time multiplayer party game (BYU CS 260 startup project). Players
answer four questions about themselves, then guess how the other players answered.

## Commands

```bash
npm run dev                 # client (vite :5173) + service (express :4000)
npm run build               # client → client/dist
npm run start -w service    # service alone; `node index.js <port>` to override 4000
npm run clear-db -w service # wipe game/user collections

npm test -w service                      # jest + supertest
npm test -w service -- -t "duplicate signup"   # single test by name
npx playwright test                      # e2e — must run from the REPO ROOT
```

`npm run dev` backgrounds the client with a bare `&`, so Ctrl-C stops only the service and leaves
vite running on :5173. Kill it separately.

**Both test suites fail as of 2026-08-21**, for unrelated reasons:

- `npm test -w service` — every test imports `database.js`, which pings Mongo at import time and
  calls `process.exit(1)` on failure. See the DB note below.
- `npm test -w client` — runs `playwright test` from `client/`, which has no config; it exits with
  "No tests found". The real config is `playwright.config.js` at the repo root (firefox, baseURL
  :5173, auto-starts the client dev server), and `tests/join.test.js` is an empty file. Run
  `npx playwright test` from the root.

Filling in the empty e2e test is a known, deliberate gap — not an oversight to fix or flag.

## Architecture

npm workspaces: `client` (React 19 + Vite) and `service` (Express + `ws` + MongoDB). `shared/` is
**not** a workspace — it is reached through the vite alias `@shared` (and `@` → `client/src`),
configured in [client/vite.config.js](client/vite.config.js).

**Dev vs. prod request paths differ.** In dev, vite proxies `/auth`, `/game`, `/question`, and
`/ws` to :4000. In prod ([Dockerfile](Dockerfile)), the client is built and copied into
`service/public`, and the service serves the SPA itself on :8080 with a catch-all that returns
`index.html`. There is no proxy in prod, so anything relying on vite's proxy config will not exist
there.

**The phase machine is the heart of the app.** A game moves `lobby → answering → guessing →
winner`. Only the server advances it: `advancePhase()` in
[service/game-service.js](service/game-service.js) assigns questions on entry to `answering`,
builds the `questionAnswerMap` on entry to `guessing`, saves, then broadcasts `phase_change` to
every socket in the game. Clients never poll — each screen subscribes with the
[usePhaseChange](client/src/hooks/usePhaseChange.js) hook and navigates when the broadcast lands.
Phases advance when the _last_ player finishes: `/question/answer` and `/question/done-guessing`
check whether every player is done and trigger the transition.

**Game state lives in memory, with Mongo as backing store.** `games` (an object in
[service/game-state.js](service/game-state.js)) is the source of truth during play; `getGame()` in
[service/game-db.js](service/game-db.js) falls back to loading from Mongo and re-hydrating via
`Game.fromMongo`. Hot paths (join, points, leave) **debounce writes by 10s** through
`game._saveTimer` rather than saving inline. Consequences: a service restart mid-game drops
anything not yet flushed, and multiple service instances would not share state.

**Sessions are also in memory** — `tokens` in [service/session-state.js](service/session-state.js)
maps a uuid cookie to `{username, name}`, so every session dies on restart. `requireLogin` demands
an account; `requireSession` also accepts guests who joined by code without signing up. Only the
host has an account; everyone else is a cookie-only guest.

**Client state** is a single `user` object (`name`, `username`, `gameCode`, `score`, `isHost`) in a
context backed by localStorage ([client/src/context.jsx](client/src/context.jsx)), which is why a
refresh mid-game keeps you in the game.

## Gotchas

- **`service/constants.js` is a byte-identical copy of `shared/constants.js`.** The client imports
  `@shared/constants.js`; the service imports its own local copy. Editing one silently desyncs the
  two halves — change both.
- **The MongoDB Atlas cluster no longer resolves.** `service/dbConfig.json` (gitignored, required
  to boot) points at a cluster whose SRV record is gone, so the service exits on start. To work
  locally, run a `mongod` and note that [service/database.js](service/database.js) hardcodes the
  `mongodb+srv://` scheme — a plain `mongodb://localhost:27017` URL needs a code change there.
  `NODE_ENV=test` switches the database name to `psychic-questions-test`.
- **The guessing screen needs 5+ players to show 4 answer choices.** `selectRandomAnswers` in
  [client/src/guess-answers/question-list.jsx](client/src/guess-answers/question-list.jsx) builds
  the wrong answers from players other than the target _and_ the guesser, so a 2-player game
  renders a single radio button. Manual testing of that screen needs five browser contexts.
- **`InputForm` does not await `updateFrontend`**
  ([client/src/components/input-form.jsx](client/src/components/input-form.jsx)), so sign-up
  navigates to `/enter-name` before `createGame()` has stored the game code. Automation must wait
  for `gameCode` to appear in localStorage before submitting the name.
- Nothing is deployed. The app ran on AWS EC2, then Fly.io ([fly.toml](fly.toml) targets port
  8080); both are down deliberately, and `https://joshuajob-cs.click` in the README is dead.

## README

`README.md` is a summary above the `## Course deliverables` heading and a graded per-
deliverable log below it. Keep the top short, and leave the deliverable sections worded as they
were graded. Sketches and screenshots live in `docs/` (there is no `Drawings/` folder, despite
older references).
