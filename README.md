# Feedback UI App 💬

A full-stack feedback collection app built with React and a mock REST API powered by `json-server`. Users can submit ratings (1–10) with written reviews, edit or delete existing feedback, and view live statistics — all with smooth animations via Framer Motion.

---

## UI Layout

```
┌─────────────────────────────────────────────────────┐
│                     HEADER                          │
│   📋 Feedback UI              [ Home ]  [ About ]   │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│                  FEEDBACK FORM                      │
│                                                     │
│  How would you rate your service with us?           │
│                                                     │
│  ① ② ③ ④ ⑤ ⑥ ⑦ ⑧ ⑨ ⑩  ← Rating (1–10)           │
│                                                     │
│  [ Write a review...              ] [ Send ]        │
│                                                     │
│  ⚠ Please enter more than 10 characters! (if short) │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│                FEEDBACK STATS                       │
│    8 Reviews              Average Rating: 4.1       │
└─────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────┐
│                 FEEDBACK LIST                       │
│  ┌───────────────────────────────────────────────┐  │
│  │ ●10   ✏  ✕   "Great service overall!"        │  │
│  └───────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────┐  │
│  │  ● 7   ✏  ✕   "Pretty good experience."      │  │
│  └───────────────────────────────────────────────┘  │
│  ┌───────────────────────────────────────────────┐  │
│  │  ● 4   ✏  ✕   "Could be better..."           │  │
│  └───────────────────────────────────────────────┘  │
│                  (animated fade in/out)             │
└─────────────────────────────────────────────────────┘
```

---

## How the App Works

### Add Feedback Flow

```
   User fills form
         │
         ▼
  ┌─────────────────────┐
  │  Text > 10 chars?   │
  └──────┬──────────────┘
         │ NO                      YES
         ▼                          ▼
  ┌─────────────┐        ┌──────────────────────┐
  │ Show warning│        │ POST /feedback        │
  │ Disable btn │        │ { text, rating }      │
  └─────────────┘        └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │ json-server saves to │
                         │      db.json         │
                         └──────────┬───────────┘
                                    │
                                    ▼
                         ┌──────────────────────┐
                         │ Prepend to feedback  │
                         │ state (newest first) │
                         └──────────────────────┘
```

### Edit Feedback Flow

```
  User clicks ✏ on a card
         │
         ▼
  editFeedback(item) called
  in FeedbackContext
         │
         ▼
  feedbackEdit state set:
  { item: {...}, edit: true }
         │
         ▼
  FeedbackForm detects edit mode
  → pre-fills text & rating
         │
         ▼
  User modifies & submits
         │
         ▼
  PUT /feedback/:id → db.json updated
         │
         ▼
  feedback state updated in place
```

### Delete Feedback Flow

```
  User clicks ✕ on a card
         │
         ▼
  window.confirm("Are you sure?")
         │
    NO ──┘──── YES
                │
                ▼
        DELETE /feedback/:id
                │
                ▼
       Filter item out of
        feedback[] state
                │
                ▼
       Card fades out via
       Framer Motion exit
```

---

## Component Tree

```
App
├── FeedbackProvider  (React Context — global state)
│   └── Router
│       ├── Header
│       │   ├── NavLink → /         (Home)
│       │   └── NavLink → /about    (About)
│       └── Routes
│           ├── / (Home route)
│           │   ├── FeedbackForm
│           │   │   ├── RatingSelect   (radio buttons 1–10)
│           │   │   ├── Input + Button (text + send)
│           │   │   └── Card (wrapper)
│           │   ├── FeedbackStats
│           │   │   └── (count + average from context)
│           │   └── FeedbackList
│           │       └── FeedbackItem × N
│           │           ├── rating badge
│           │           ├── ✏ edit button
│           │           ├── ✕ delete button
│           │           └── Card (wrapper)
│           └── /about → About page
```

---

## Data & State Flow

```
                    ┌─────────────────────────────┐
                    │       FeedbackContext        │
                    │                             │
                    │  feedback[]  ◄──────────────┼── GET /feedback (on mount)
                    │  feedbackEdit               │
                    │  isLoading                  │
                    │                             │
                    │  addFeedback()   → POST      │
                    │  updateFeedback() → PUT      │
                    │  handleDelete()  → DELETE    │
                    │  editFeedback()  (local)     │
                    └────────────┬────────────────┘
                                 │  Context API
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                   ▼
       FeedbackForm        FeedbackStats       FeedbackList
      (add / update)      (count, avg)       (render items)
                                                     │
                                              FeedbackItem
                                            (edit / delete)
```

---

## Backend (json-server)

The app uses `json-server` to simulate a REST API backed by `db.json`.

```
db.json
└── feedback[]
    ├── { id, rating, text }   ← each feedback entry
    ├── { id, rating, text }
    └── ...

Endpoints auto-generated:
  GET    /feedback           → fetch all (sorted by id desc)
  POST   /feedback           → add new entry
  PUT    /feedback/:id       → update entry
  DELETE /feedback/:id       → delete entry

Proxy: React dev server (3000) → json-server (5000)
```

---

## Tech Stack

| Library | Purpose |
|---|---|
| React 18 | UI framework |
| React Router v6 | Client-side routing (/, /about) |
| React Context API | Global state management |
| json-server | Mock REST API backed by db.json |
| Framer Motion | Animated list add/remove transitions |
| react-icons | Edit (✏) and Delete (✕) icons |
| uuid | Unique IDs for new feedback entries |
| concurrently | Run API server and React app together |

---

## Project Structure

```
feedback_ui_app-main/
├── db.json                        # Mock database (json-server)
├── package.json
├── public/
│   └── index.html
└── src/
    ├── App.js                     # Root component + routing
    ├── index.js                   # React entry point
    ├── index.css                  # Global styles
    ├── context/
    │   └── FeedbackContext.jsx    # Global state + API calls
    ├── components/
    │   ├── FeedbackForm.jsx       # Add / edit feedback form
    │   ├── FeedbackList.jsx       # Animated list of feedback cards
    │   ├── FeedbackItem.jsx       # Single card with edit/delete
    │   ├── FeedbackStats.jsx      # Review count + average rating
    │   ├── Header.jsx             # Top nav bar
    │   ├── RatingSelect.jsx       # Radio buttons for rating 1–10
    │   ├── assets/
    │   │   └── spinner.gif        # Loading spinner
    │   └── shared/
    │       ├── Button.jsx         # Reusable button
    │       ├── Card.jsx           # Reusable card wrapper
    │       └── Spinner.jsx        # Loading state component
    ├── data/
    │   └── feedbackdata.js        # Static seed data (fallback)
    └── pages/
        ├── Home.jsx               # Home page
        └── About.jsx              # About page
```

---

## Getting Started

### Prerequisites

- [Node.js](https://nodejs.org/) v14+
- npm

### Installation

1. Clone the repository:
   ```bash
   git clone https://github.com/your-username/feedback_ui_app.git
   cd feedback_ui_app
   ```

2. Install dependencies:
   ```bash
   npm install
   ```

3. Start both the React app and the API server together:
   ```bash
   npm run dev
   ```

   Or run them separately:
   ```bash
   npm run server   # json-server on port 5000
   npm start        # React app on port 3000
   ```

4. Open your browser at `http://localhost:3000`

---

## Available Scripts

| Script | Description |
|---|---|
| `npm start` | Runs React app in development mode (port 3000) |
| `npm run server` | Starts json-server API on port 5000 |
| `npm run dev` | Runs both concurrently |
| `npm run build` | Builds the app for production |
| `npm test` | Runs the test suite |

---

## License

This project is open source and available under the [MIT License](LICENSE).
