# Architecture Overview

## Commands

```bash
npm start          # dev server at localhost:3000
npm run build      # production build (ESLint disabled — intentional)
npm test           # run tests in watch mode
npx vercel dev     # run with Vercel serverless functions locally (required to test /api routes)
```

## GoTeam has two parallel flows

### Planning flow (newer)
`/plan` → `/plan-response/:sessionId` → `/plan-results/:sessionId`

Organiser creates a `planning_sessions` record in Supabase. Participants submit availability (date ranges) and activity preferences. The organiser triggers reconciliation, which runs client-side in `PlanningResults.js` — it finds overlapping date windows ranked by participant count, then calls `/api/suggest-activities` for AI activity suggestions.

### Accommodation flow (older)
`/` (homepage) → `/respond/:tripId` → `/results/:tripId`

Organiser creates a `trips` record. Participants submit accommodation preferences. The organiser triggers `/api/reconcile`, which calls the Anthropic API with a detailed system prompt and returns a structured JSON brief with search links for Booking.com and Airbnb.

### Backend
`/api/` contains Vercel serverless functions written in CommonJS (`require`/`module.exports`), separate from the React frontend's ES module imports.

- `reconcile.js` — AI accommodation reconciliation (Anthropic API, heavy system prompt)
- `suggest-activities.js` — AI activity suggestions for day-out events
- `send-organiser-email.js`, `send-planning-email.js` — transactional email via Resend
- `notify-*.js` — participant notification emails

### Data layer
A single Supabase client is used throughout the frontend. Recent trips are persisted to `localStorage` (array, max 10, newest first) so organisers can pick up where they left off on the homepage.

## UI conventions

All styling is inline styles via a `styles` object at the bottom of each file — no CSS framework, no CSS modules.

**UI pattern:** every page is a chat-bubble interface — the app asks questions, the user answers, and the conversation scrolls down. Step state is managed locally with a `step` string and a `stepHistory` stack for back navigation.

**Event types:** `trip_away`, `day_out`, `gathering` — each activates different modules and questions throughout both flows.

**Date inputs:** `react-datepicker` in inline mode. Range selection for trips, single-date for day outs and gatherings.

**Share:** uses the Web Share API on mobile with a clipboard fallback on desktop.

## Deployment

Deployed on Vercel. The `/api/` folder is auto-detected as serverless functions.
