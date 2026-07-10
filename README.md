# StudyNest

A full-stack AI-powered study assistant. Upload lecture notes as PDFs and get AI-generated
summaries, quizzes, and flashcards in seconds — plus a study planner, daily goals with a
real streak, and a progress dashboard, all wrapped in a dark-mode-ready, responsive UI.

> **Status:** All 10 build steps complete. Every module described below is implemented,
> wired end-to-end, and backed by real MongoDB data — no placeholders remain.

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

