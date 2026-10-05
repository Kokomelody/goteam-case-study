# GoTeam — Case Study

### 🔗 [Try the live app → goteam.buildingmelody.com](https://goteam.buildingmelody.com)

GoTeam is a group trip and event coordination tool that removes the chaos of planning anything with other people — finding dates that work for everyone, agreeing on a destination, and collecting accommodation preferences, all without accounts or app downloads.

This repository shares the product and engineering thinking behind the live app: the vision and roadmap, a full PRD for an in-progress feature, and an architecture overview. The application's source code lives in a private repository (it includes third-party API keys and a production database), so this case study exists to share the thinking publicly.

## Contents

- [`VISION.md`](./VISION.md) — product vision, core principles, and phased roadmap (including monetisation strategy)
- [`PRD-planning-module.md`](./PRD-planning-module.md) — a complete PRD for GoTeam's planning module (problem statement, goals, user stories, phased requirements with acceptance criteria)
- [`ARCHITECTURE.md`](./ARCHITECTURE.md) — technical overview: stack, data flow, and conventions

## Stack at a glance

React (Create React App) frontend, Vercel serverless functions for the backend, Supabase for data, and the Anthropic API for AI-generated reconciliation, suggestions, and summaries.
