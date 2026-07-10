# StudyNest

A full-stack AI-powered study assistant. Upload lecture notes as PDFs and get AI-generated
summaries, quizzes, and flashcards in seconds — plus a study planner, daily goals with a
real streak, and a progress dashboard, all wrapped in a dark-mode-ready, responsive UI.

> **Status:** All 10 build steps complete. Every module described below is implemented,
> wired end-to-end, and backed by real MongoDB data — no placeholders remain.

---

## Table of Contents

- [Tech Stack](#tech-stack)
- [Folder Structure](#folder-structure)
- [Installation](#installation)
- [Environment Variables](#environment-variables)
- [API Documentation](#api-documentation)
- [Database Schema](#database-schema)
- [Security](#security)
- [UI / Theming](#ui--theming)
- [Known Limitations](#known-limitations)

---

## Tech Stack

| Layer          | Technology                                              |
|----------------|----------------------------------------------------------|
| Frontend       | HTML5, CSS3, Vanilla JavaScript (no framework)             |
| Backend        | Node.js, Express.js                                        |
| Database       | MongoDB, Mongoose                                           |
| Auth           | JWT (`jsonwebtoken`), `bcrypt`                               |
| AI             | Google Gemini API (`@google/generative-ai`)                   |
| File Upload    | Multer                                                          |
| PDF Processing | `pdf-parse`                                                      |
| Validation     | `express-validator`                                               |
| Security       | `helmet`, `express-rate-limit`, `cors`, `cookie-parser`             |
| Logging        | `morgan`                                                              |

No React/Vue/Angular/Next.js/TypeScript anywhere — every interactive page is plain HTML
with a page-specific vanilla-JS file, sharing common auth/sidebar/theme logic via
`public/js/app-shell.js` and `public/js/theme.js`.

## Folder Structure

```
studynest/
├── config/
│   ├── db.js               # MongoDB connection (Mongoose)
│   ├── jwt.js               # JWT sign/verify helpers
│   └── gemini.js             # Gemini client + prompts (summary, quiz, flashcards)
├── controllers/
│   ├── authController.js      # register, login, logout, profile, change password
│   ├── dashboardController.js  # stat cards + 7-day chart data (all real MongoDB queries)
│   ├── noteController.js        # PDF upload, list, get, delete (+ cascading cleanup)
│   ├── summaryController.js      # AI summary generate/fetch
│   ├── quizController.js          # AI quiz generate/fetch/attempt/history
│   ├── flashcardController.js      # AI flashcard deck generate/fetch/review
│   ├── plannerController.js         # Study Planner task CRUD
│   └── goalController.js             # Daily Goal CRUD + study streak logic
├── middleware/
│   ├── authMiddleware.js       # `protect` — verifies JWT, attaches req.user
│   ├── errorMiddleware.js       # centralized 404 + error handler
│   ├── uploadMiddleware.js       # Multer config (PDF-only, size limit)
│   └── validators/
│       └── authValidators.js      # express-validator rules for register/login
├── models/                     # Mongoose schemas — see Database Schema below
│   ├── User.js
│   ├── Note.js
│   ├── Summary.js
│   ├── Quiz.js
│   ├── Flashcard.js
│   ├── StudyPlanner.js
│   └── Goal.js
├── routes/                     # One file per resource, mirrors controllers/
│   ├── healthRoutes.js
│   ├── authRoutes.js
│   ├── dashboardRoutes.js
│   ├── noteRoutes.js
│   ├── summaryRoutes.js
│   ├── quizRoutes.js
│   ├── flashcardRoutes.js
│   ├── plannerRoutes.js
│   └── goalRoutes.js
├── utils/
│   └── date.js                 # startOfDay / daysBetween helpers (streaks, calendar)
├── public/                     # Everything Express serves statically
│   ├── index.html                # Landing page (Hero/Features/How It Works/FAQ/Contact)
│   ├── login.html / signup.html    # Auth pages
│   ├── dashboard.html               # Stat cards, activity feed, progress charts
│   ├── notes.html                    # Upload, list, Summarize/Quiz/Flashcards modals
│   ├── planner.html                   # Calendar + Study Planner + Daily Goals
│   ├── css/
│   │   ├── style.css                    # Design tokens (incl. dark theme), landing page
│   │   ├── auth.css                      # Login/signup page styles
│   │   ├── dashboard.css                  # App shell (sidebar/topbar) + charts
│   │   ├── notes.css                       # Notes list + summary/quiz/flashcard modals
│   │   └── planner.css                      # Calendar + task list + goals
│   ├── js/
│   │   ├── theme.js                          # Dark/light toggle, localStorage-backed
│   │   ├── app-shell.js                       # Shared sidebar/logout/toast for app pages
│   │   ├── main.js                             # Landing page interactivity
│   │   ├── auth.js                              # Login/signup form handling
│   │   ├── dashboard.js                          # Stat cards + SVG chart rendering
│   │   ├── notes.js                               # Upload + AI feature modals
│   │   └── planner.js                              # Calendar + planner + goals logic
│   └── images/                 # (reserved for future landing page assets)
├── views/                      # (reserved — this project serves static HTML directly
│                                #  from public/ rather than server-rendered templates)
├── uploads/                    # Uploaded PDFs land here (Multer destination, gitignored)
├── .env.example                # Template — copy to .env and fill in real values
├── .gitignore
├── package.json
└── server.js                   # App entry point: middleware, routes, error handling
```

## Installation

1. **Install dependencies:**
   ```bash
   npm install
   ```
   > `bcrypt` compiles a native binary during install. If that fails on your machine
   > (missing build tools), swap it for `bcryptjs` in `package.json` — same API, pure JS.

2. **Create your `.env` file:**
   ```bash
   cp .env.example .env
   ```
   Fill in a real `MONGO_URI`, a random `JWT_SECRET`, and a `GEMINI_API_KEY` (see below).

3. **Start MongoDB** — either run it locally, or use a free MongoDB Atlas cluster and
   paste its connection string into `MONGO_URI`.

4. **Get a Gemini API key** from Google AI Studio (aistudio.google.com/apikey) and put
   it in `GEMINI_API_KEY`. Without this, everything works except Summarize/Quiz/
   Flashcard generation, which will return a 500 error.

5. **Run the server:**
   ```bash
   npm run dev     # nodemon — auto-restarts on file changes
   # or
   npm start       # plain node
   ```

6. **Open the app:** `http://localhost:5000/` — sign up, upload a PDF, and try
   Summarize / Quiz / Flashcards from the Notes page.

## Environment Variables

All variables live in `.env` (see `.env.example` for the full template):

| Variable           | Required | Description                                              |
|--------------------|----------|----------------------------------------------------------|
| `PORT`             | No       | Port the server listens on (default `5000`)                 |
| `NODE_ENV`         | No       | `development` or `production` — affects cookie `secure` flag  |
| `MONGO_URI`        | **Yes**  | MongoDB connection string                                       |
| `JWT_SECRET`       | **Yes**  | Long random string used to sign JWTs — never commit a real one   |
| `JWT_EXPIRES_IN`   | No       | Token lifetime (default `7d`)                                      |
| `GEMINI_API_KEY`   | **Yes**  | Needed for Summary/Quiz/Flashcard generation                        |
| `MAX_FILE_SIZE_MB` | No       | Max PDF upload size in MB (default `10`)                              |

## API Documentation

All endpoints are prefixed with `/api`. Endpoints marked **Auth: Yes** require either an
`Authorization: Bearer <token>` header or the httpOnly `token` cookie set automatically on
login/register. Every resource endpoint (`:id`/`:noteId`) also checks that the resource
belongs to the requesting user — there is no way to read or modify another user's data by
guessing an ID.

### Health

| Method | Endpoint      | Description                | Auth |
|--------|---------------|------------------------------|------|
| GET    | `/api/health` | Confirms the API is running    | No   |

### Auth (`/api/auth`)

| Method | Endpoint          | Description                                | Auth |
|--------|-------------------|----------------------------------------------|------|
| POST   | `/register`       | Create an account (rate-limited)               | No   |
| POST   | `/login`          | Log in, receive a JWT (rate-limited)             | No   |
| POST   | `/logout`         | Clear the auth cookie                              | Yes  |
| GET    | `/profile`        | Get the logged-in user's profile                     | Yes  |
| PUT    | `/profile`        | Update name / profile picture                          | Yes  |
| PUT    | `/change-password`| Change password                                          | Yes  |

**Frontend:** `/login.html`, `/signup.html` — inline field validation, password
show/hide toggle, and a loading state on submit. `public/js/auth.js` stores the returned
token in `localStorage` and redirects to `/dashboard.html`.

### Dashboard (`/api/dashboard`)

| Method | Endpoint    | Description                                       | Auth |
|--------|-------------|-------------------------------------------------------|------|
| GET    | `/stats`    | Welcome name, study streak, and all four stat counts      | Yes  |
| GET    | `/progress` | 7-day chart data (study minutes/notes/goals) + recent quiz scores | Yes  |

Every number here is a real, live MongoDB query — total notes, total quiz *attempts*
(not just quizzes), total flashcards (individual cards, not decks), and completed goals.
The activity feed merges recent note uploads, quiz attempts, flashcard deck generations,
and completed goals by time. Chart data is rendered as hand-built inline SVG bar charts
in `public/js/dashboard.js` — no charting library.

**Frontend:** `/dashboard.html`.

### Notes (`/api/notes`)

| Method | Endpoint | Description                                              | Auth |
|--------|----------|---------------------------------------------------------------|------|
| POST   | `/upload`| Upload a PDF (`multipart/form-data`, field name `file`)           | Yes  |
| GET    | `/`      | List the user's notes (metadata only, no extracted text)            | Yes  |
| GET    | `/:id`   | Get one note, including its full extracted text                        | Yes  |
| DELETE | `/:id`   | Delete a note — the file, and any Summary/Quiz/Flashcard for it          | Yes  |

PDF-only, enforced by both MIME type and file extension. Text is pulled out with
`pdf-parse`; if a PDF has no real text layer (a pure image scan), the upload still
succeeds but is flagged `textExtracted: false` so the UI can explain why AI features
won't work on it. Deleting a note also deletes its associated Summary, Quiz, and
Flashcard documents so nothing is left orphaned in the database.

**Frontend:** `/notes.html` — drag-and-drop or click-to-browse upload with a real
progress bar, and a list of notes with Summarize / Quiz / Flashcards / View / Delete
actions.

### Summary (`/api/summary`)

| Method | Endpoint   | Description                                     | Auth |
|--------|------------|------------------------------------------------------|------|
| POST   | `/:noteId` | Generate (or regenerate) an AI summary for a note        | Yes  |
| GET    | `/:noteId` | Get the existing summary, if one has been generated         | Yes  |

Sends the note's extracted text to Gemini (`gemini-1.5-flash`) asking for strict JSON: a
short summary, a detailed summary, key points, and definitions. One summary per note —
regenerating replaces it (`upsert`) rather than creating duplicates. Notes with no
extracted text return `422` instead of calling Gemini.

**Frontend:** a modal on `/notes.html`.

### Quiz (`/api/quiz`)

| Method | Endpoint            | Description                                         | Auth |
|--------|---------------------|---------------------------------------------------------|------|
| POST   | `/:noteId`          | Generate (or regenerate) a 5-question quiz for a note      | Yes  |
| GET    | `/:noteId`          | Get the existing quiz — `correctAnswer` stripped out           | Yes  |
| POST   | `/:noteId/attempt`  | Submit answers; scored **server-side**, saved to history          | Yes  |
| GET    | `/:noteId/history`  | Get all past attempts for this note's quiz                          | Yes  |

Every AI-generated question is validated (exactly 4 options, correct answer is one of
them) before saving. `correctAnswer` never reaches the client until after they've
submitted an attempt — scoring happens entirely server-side, so a client can't report
its own score. Regenerating a quiz clears old attempts, since they'd otherwise reference
questions that no longer exist.

**Frontend:** a modal on `/notes.html` — empty state → generate → radio-button form →
scored results with correct/incorrect highlighting → retake/regenerate → attempt history.

### Flashcards (`/api/flashcards`)

| Method | Endpoint                    | Description                                | Auth |
|--------|------------------------------|------------------------------------------------|------|
| POST   | `/:noteId`                   | Generate (or regenerate) a flashcard deck          | Yes  |
| GET    | `/:noteId`                   | Get the existing deck for a note                      | Yes  |
| PUT    | `/:noteId/review/:cardId`    | Record a card review (optionally mark "known")           | Yes  |

Up to 10 front/back cards per deck, validated server-side before saving. Each card
tracks `timesReviewed` / `timesKnown` — not used for scoring anywhere yet, but laid down
for a future spaced-repetition feature.

**Frontend:** a modal on `/notes.html` with a real 3D flip-card viewer, Prev/Next
navigation, and "Still learning" / "I know this" buttons that auto-advance.

### Planner (`/api/planner`)

| Method | Endpoint | Description                                                  | Auth |
|--------|----------|-------------------------------------------------------------------|------|
| GET    | `/`      | List tasks, optionally filtered by `?from=`/`?to=` (ISO date strings) | Yes  |
| POST   | `/`      | Add a task: `subject`, `task?`, `date`, `durationMinutes`               | Yes  |
| PUT    | `/:id`   | Update a task (edit fields or toggle `completed`)                          | Yes  |
| DELETE | `/:id`   | Delete a task                                                                | Yes  |

**Frontend:** `/planner.html` — a real month calendar built in vanilla JS (no library),
with a dot on any day that has a task; clicking a day filters the task list to it.

### Goals (`/api/goals`)

| Method | Endpoint | Description                                          | Auth |
|--------|----------|-----------------------------------------------------------|------|
| GET    | `/`      | List goals, optionally filtered by `?date=`                   | Yes  |
| POST   | `/`      | Add a goal: `title`, `date?` (defaults to today)                 | Yes  |
| PUT    | `/:id`   | Update a goal (edit title or toggle `completed`)                    | Yes  |
| DELETE | `/:id`   | Delete a goal                                                          | Yes  |

The first time (each calendar day) a user completes any goal, `User.studyStreak`
updates: `+1` if they also completed one yesterday, reset to `1` on a gap, unchanged if
they've already logged today. Un-completing a goal deliberately does **not** decrement
the streak, to avoid punishing an accidental un-check.

**Frontend:** the "Daily Goals" section on `/planner.html`, scoped to whichever date is
selected on the calendar.

## Database Schema

7 collections, all scoped to a `user` field (ObjectId ref → `User`) except `User` itself.

### User

| Field                | Type   | Notes                                                  |
|----------------------|--------|-------------------------------------------------------------|
| name                 | String | required, 2–60 chars                                          |
| email                | String | required, unique, lowercased                                    |
| password             | String | required, min 8 chars, bcrypt-hashed, hidden from queries by default |
| profilePicture       | String | optional URL                                                       |
| studyStreak          | Number | default 0 — updated by the Goals module                              |
| lastStudyDate        | Date   | default null — used to calculate streaks                               |
| createdAt/updatedAt  | Date   | automatic (`timestamps: true`)                                           |

### Note

| Field               | Type    | Notes                                                    |
|---------------------|---------|---------------------------------------------------------------|
| user                | ObjectId| ref `User`, required, indexed                                   |
| originalName        | String  | filename as uploaded                                                |
| storedFileName      | String  | sanitized, collision-safe filename on disk in `/uploads`               |
| filePath            | String  | public path, e.g. `/uploads/xyz.pdf`                                     |
| fileSizeBytes       | Number  | file size in bytes                                                         |
| pageCount           | Number  | from `pdf-parse`                                                              |
| extractedText       | String  | full text — feeds Summary/Quiz/Flashcard generation                            |
| textExtracted       | Boolean | false if the PDF had no readable text layer (e.g. a scan)                        |
| createdAt/updatedAt | Date    | automatic                                                                           |

### Summary

One per note (`note` is `unique`). Fields: `shortSummary`, `detailedSummary`,
`keyPoints` (`[String]`), `definitions` (`[{term, definition}]`).

### Quiz

One per note (`note` is `unique`). `questions`: `[{question, options[4], correctAnswer}]`.
`attempts`: `[{answers, score, totalQuestions, attemptedAt}]` — one entry per submission.

### Flashcard

A "Flashcard" document is actually a whole deck (one per note, `note` is `unique`).
`cards`: `[{front, back, timesReviewed, timesKnown}]`.

### StudyPlanner

| Field           | Type    | Notes                        |
|-----------------|---------|-----------------------------------|
| user            | ObjectId| ref `User`, required, indexed        |
| subject         | String  | required, max 100 chars                 |
| task            | String  | optional description, max 200 chars        |
| date            | Date    | required                                     |
| durationMinutes | Number  | required, 5–600                                |
| completed       | Boolean | default false                                    |

### Goal

| Field       | Type    | Notes                                          |
|-------------|---------|------------------------------------------------------|
| user        | ObjectId| ref `User`, required, indexed                            |
| title       | String  | required, max 150 chars                                    |
| date        | Date    | defaults to today, stored at local midnight                   |
| completed   | Boolean | default false                                                    |
| completedAt | Date    | set when marked complete, cleared when un-marked                   |

## Security

- **Passwords** are hashed with `bcrypt` (10 salt rounds) via a Mongoose pre-save hook —
  never stored or logged in plain text. The password field is `select: false` by default,
  so it's never accidentally returned from a query.
- **JWTs** are delivered two ways (JSON body + httpOnly cookie) so the vanilla-JS
  frontend can use whichever transport it needs, while the cookie stays inaccessible to
  page JavaScript. `protect` middleware verifies the token and re-loads the user from the
  DB on every request (not just trusting the token payload).
- **Rate limiting** (`express-rate-limit`) caps `/api/auth/register` and `/api/auth/login`
  at 20 attempts per 15 minutes per IP — basic brute-force protection.
- **Security headers** via `helmet` (X-Content-Type-Options, X-Frame-Options, etc.).
  Content-Security-Policy is explicitly disabled because the anti-flash-of-wrong-theme
  inline `<script>` and Google Fonts CDN would otherwise be blocked by helmet's default
  policy — see the comment in `server.js` for the tradeoff and what a production CSP
  would need to allow.
- **Ownership checks** everywhere: every note/summary/quiz/flashcard/planner-task/goal
  lookup filters by `{ _id, user: req.user._id }`, not just `{ _id }` — there's no
  endpoint where knowing another user's document ID lets you read or modify it.
- **Quiz integrity**: `correctAnswer` is stripped from quiz questions before they're sent
  to the client for taking, and scoring happens entirely server-side — a client can't
  submit its own score.
- **Input validation**: `express-validator` on auth routes; manual required-field and
  type checks in other controllers (planner, goals) with clear 400 errors.
- **XSS**: all AI-generated and user-entered text (note names, summaries, quiz questions,
  flashcard content, goal titles) is inserted into the DOM via `textContent`-based
  escaping helpers, never raw `innerHTML` concatenation of untrusted strings.
- **File upload safety**: PDF-only, checked by both MIME type and extension (MIME alone
  can be spoofed); filenames are sanitized before being written to disk; size capped by
  `MAX_FILE_SIZE_MB`.

## UI / Theming

Dark/light mode is a `data-theme="dark"` attribute on `<html>` that flips a full set of
CSS custom properties defined in `public/css/style.css`. Every color on every page is a
`var(--token)` reference, so theming was additive rather than a rewrite. Preference is
saved to `localStorage` (`sn_theme`) and applied via an inline `<script>` at the very
top of `<head>` on every page — before the stylesheet paints — so there's no flash of the
wrong theme on load. `public/js/theme.js` wires up every `.theme-toggle` button (landing
nav, both auth pages, every app-shell topbar) and falls back to the OS's
`prefers-color-scheme` if the user has never toggled it manually.
v
The visual language throughout is an "open notebook" concept — dark ink-navy surfaces,
warm paper-colored cards, highlighter-yellow/mint accents, a raspberry-ink accent for
calls to action — carried from the landing page's flip-card hero through to the
flashcard deck viewer, index-card-style feature grid, and sticky-note testimonials.

## Known Limitations

- **Cross-timezone date boundaries.** Planner/Goal dates and the study streak are
  computed using each environment's local time (the browser's local time when a date is
  picked, the server's local time when day boundaries are calculated). If your deployed
  server runs in a different timezone than your users, day boundaries can be off by one
  day for users near midnight. A full fix would mean sending explicit UTC offsets (or
  plain `{year, month, day}` values) instead of `Date` objects.
- **No automated tests.** Every change was verified with `node -c` syntax checks and
  manual code review, but there's no Jest/Supertest suite. Worth adding if this grows
  past a class project.
- **Gemini cost/rate limits.** Every summary/quiz/flashcard generation is a live Gemini
  API call with no caching beyond "don't regenerate unless asked" — heavy use will hit
  whatever limits your API key is subject to.
- **Uploaded PDFs are served statically** from `/uploads` with no additional auth check
  on the file URL itself (only the API endpoints that reference a note are protected).
  Anyone with a guessed/leaked file URL could view that PDF. Fine for a class project;
  a production version should proxy file downloads through an authenticated route.
