# PRD: GoTeam Planning Module — Dates & Destination Discovery

**Status:** Draft  
**Author:** Melody  
**Date:** June 2026  
**Scope:** v1 MVP

---

## Problem Statement

Groups planning trips together currently waste significant time in fragmented back-and-forth across WhatsApp, email, and spreadsheets, trying to find dates that work for everyone and a destination that suits the group's vibe. GoTeam currently assumes dates and destination are already decided, which means it misses the hardest and most chaotic part of group trip planning. Without a structured way to collect availability and preferences upfront, groups either never agree, or one person imposes their preference on everyone else.

---

## Goals

1. **Reduce date-finding friction** — groups can identify overlapping availability without a single WhatsApp thread
2. **Enable vibe-based destination discovery** — groups who don't know where they're going get personalised suggestions based on aggregated preferences, not just the loudest voice
3. **Make the planning module skippable** — groups who already have dates and/or a destination can bypass relevant steps without friction
4. **Feed cleanly into the existing flow** — outputs from this module (agreed dates, shortlisted destinations) flow directly into CreateTrip, eliminating duplicate data entry
5. **Support multiple event types** — the module works for a weekend villa trip, a ski holiday, a family reunion, and a city day out, activating only the relevant sub-flows

---

## Non-Goals

- **Booking or reserving anything** — this module is purely discovery and alignment; no calendar integrations, no booking APIs in v1
- **Real-time conflict detection** — we are not building a calendar sync (Google Calendar, iCal etc.); participants enter availability manually as date ranges
- **Destination search or filtering** — we surface AI-generated destination suggestions based on group vibe; we are not building a search or filter UI over a destination database in v1
- **More than 10 participants** — finding date overlap beyond 10 people requires a fundamentally different UX; out of scope for v1
- **Activity or restaurant recommendations** — Day out and Gathering events may need this in future, but it is a separate module and out of scope here

---

## User Stories

### Organiser

- As an organiser, I want to tell GoTeam what kind of event I'm planning so that it only asks me relevant questions and doesn't waste my time
- As an organiser, I want to share a single link with my group so that everyone can submit all their answers in one go without me chasing individuals
- As an organiser, I want to trigger reconciliation myself once everyone has submitted so that I am in control of when results are processed
- As an organiser, I want to see which date windows work for everyone (or most people) so that I can make a confident decision without running a poll manually
- As an organiser, I want the app to suggest destinations that match my group's aggregated vibe so that I can propose options people will actually be excited about
- As an organiser, I want to skip the dates module if we've already agreed on dates, and skip the destination module if we've already chosen a location
- As an organiser, I want the outputs of this module (dates, destination shortlist) to flow automatically into the trip setup so that I don't have to re-enter anything

### Participant

- As a participant, I want to complete everything — availability, vibe preferences — in a single session so that I don't have to come back multiple times
- As a participant, I want to select my available dates on a single calendar rather than filling in multiple separate fields so that the process is quick and visual
- As a participant, I want to answer a short set of swipeable preference questions about trip vibe so that my voice is heard even if I'm not the loudest person in the group chat
- As a participant, I want the experience to feel light and fun, not like filling in a form, so that I actually complete it

---

## Requirements

### P0 — Must-Have for v1

**1. Event type selection**
- Organiser selects what they're planning at the start of the flow
- Options (v1): Trip away, Day out, Gathering
- "Day out" replaces both "Day out" and "City event" — they are the same thing
- "Gathering" covers any event at a home, venue, or restaurant — replaces "Home event"
- Selection determines which modules activate (dates and/or destination)
- Acceptance criteria:
  - [ ] Organiser sees event type selection before any other questions
  - [ ] Selecting "Gathering" skips destination module entirely
  - [ ] Selecting "Day out" retains dates module; destination module is optional (organiser decides)
  - [ ] Selecting "Trip away" activates both modules unless organiser skips

**2. Skip logic**
- After event type, organiser is asked: "Have you already decided on dates?" and "Have you already decided on destination?" (where relevant)
- Yes → skip that module, enter known value manually
- No → module activates
- Acceptance criteria:
  - [ ] Both skip questions shown only when relevant to the event type
  - [ ] Skipping dates module goes directly to destination module (if active) or existing CreateTrip flow
  - [ ] Manual date entry on skip feeds into CreateTrip exactly as before

**3. Participant submission — single end-to-end flow**
- Participants complete everything in one sitting: availability calendar + vibe swipes (if applicable)
- No module is gated on other participants having submitted first
- Participants do not see any results — only the organiser does
- Organiser's own availability and vibe preferences are collected with equal weight to all other participants
- Acceptance criteria:
  - [ ] Participant sees availability calendar and vibe swipes (where applicable) in a single uninterrupted flow
  - [ ] Participant can submit without waiting for others
  - [ ] No results, overlap data, or destination suggestions are visible to participants at any point
  - [ ] Organiser is prompted to submit their own responses after creating the planning session

**4. Availability collection**
- Single scrollable calendar spanning 12 months from today
- Participant selects multiple date ranges directly on the calendar
- Library: `react-date-range` (native multi-range support)
- Organiser's availability weighted equally to participants
- Acceptance criteria:
  - [ ] Calendar renders as a single scrollable view covering today + 12 months
  - [ ] Participant can select unlimited non-contiguous date ranges on the same calendar
  - [ ] Selected ranges are visually distinct from each other and from unselected dates
  - [ ] Date ranges stored as JSONB array per participant: `[{start: 'YYYY-MM-DD', end: 'YYYY-MM-DD'}, ...]`
  - [ ] Past dates are disabled
  - [ ] Participant can deselect a range by tapping it again

**5. Organiser-triggered reconciliation**
- App does not process results automatically
- Organiser sees a dashboard showing how many participants have submitted vs. total invited
- When ready, organiser taps "Everyone's in — see results" to trigger reconciliation
- Reconciliation runs all modules in sequence: dates first, then destination (if applicable)
- Acceptance criteria:
  - [ ] Organiser dashboard shows submission count (e.g. "4 of 6 submitted")
  - [ ] "See results" button is available at any point — organiser decides when enough people have submitted
  - [ ] Tapping "See results" triggers overlap calculation and (if applicable) destination suggestion generation
  - [ ] Results are shown to organiser only

**6. Overlap calculation**
- App calculates which date windows have the most participant coverage across all submitted ranges
- Surfaces top 1–3 windows ranked by coverage count
- Partial overlap always shown — no dead ends
- Acceptance criteria:
  - [ ] Overlap calculated correctly across all submitted ranges from all participants
  - [ ] Results ranked by number of participants available (e.g. "5 of 6 people free 12–15 Sept")
  - [ ] Minimum trip length respected (e.g. at least 2 nights for a trip away, 1 day for a day out)
  - [ ] If no perfect overlap exists, best partial coverage shown with clear participant count
  - [ ] Organiser selects their preferred window; selection feeds into CreateTrip

**7. Vibe swipes — dynamic axis selection**
- Organiser inputs a rough trip description at setup (free text)
- AI infers which vibe axes are already implied and skips them
- Remaining open axes presented as binary swipe choices to each participant within their single submission flow
- Five possible axes (presented only when not already implied):
  1. Beach vs Mountains
  2. City vs Countryside
  3. Adventure vs Chill
  4. Lively vs Low-key
  5. Foodie vs Active
- Acceptance criteria:
  - [ ] AI correctly skips axes already implied by trip description (e.g. "skiing" skips Beach vs Mountains and City vs Countryside)
  - [ ] Minimum 2, maximum 5 axes shown per participant
  - [ ] Swipe UI is mobile-first and completable in under 60 seconds
  - [ ] Vibe responses stored as JSONB per participant: `{beach_mountains: 'beach', adventure_chill: 'chill', ...}`
  - [ ] If organiser skips destination module, vibe swipes do not appear in participant flow

**8. Destination suggestions**
- Generated at reconciliation time, not before
- App generates 3 destination suggestions via Anthropic API using: aggregated vibe profile, agreed date window (for climate/season), event type, and trip description
- Each suggestion shows destination name and 1–2 sentence rationale
- Acceptance criteria:
  - [ ] Suggestions generated only when organiser triggers reconciliation
  - [ ] Each suggestion includes destination name and rationale
  - [ ] Suggestions account for season and climate implied by the agreed date window
  - [ ] Organiser can request one fresh set of suggestions if they reject all three

**9. Group vote on destination shortlist**
- Organiser shares the 3 suggestions with the group for a vote
- Each participant votes for their top choice (single choice)
- Organiser sees results and makes final call, which feeds into CreateTrip
- Acceptance criteria:
  - [ ] Voting is single-choice per participant
  - [ ] Results visible to organiser only in v1
  - [ ] Organiser can override the vote result and pick any option
  - [ ] Chosen destination feeds directly into CreateTrip destination field

---

### P1 — Nice-to-Have

- **Visual calendar display** for overlap results — highlight the winning windows on a calendar view rather than text only
- **Participant nudging** — organiser can send a reminder to participants who haven't submitted yet (via Resend, same pattern as existing notify-organiser)
- **"None of these" option on destination vote** — allows group to flag that none of the suggestions work, prompting a fresh set
- **Confidence indicators on destinations** — show how many axes each suggestion matches (e.g. 4/5)
- **Climate note on date windows** — a one-line AI note on each suggested window e.g. "Late September in southern Europe: warm but not peak heat"

---

### P2 — Future Considerations

- **Calendar sync** — connect to Google Calendar / iCal to auto-populate availability
- **Activity and restaurant recommendations** — for Day out and Gathering event types
- **Public voting** — participants see each other's votes to encourage discussion
- **Multi-destination comparison** — destinations shown side by side with pros/cons
- **Budget filter** — filter destination suggestions by estimated cost tier

---

## Success Metrics

### Leading indicators (first 2–4 weeks post-launch)
- % of new trips that use the planning module rather than skipping it entirely — target >40%
- Participant completion rate for single-session submission — target >70% of invited participants
- Participant completion rate for vibe swipes specifically — target >80% of those who see them
- Time from organiser creating planning session to triggering reconciliation — target <48 hours

### Lagging indicators (4–8 weeks post-launch)
- % of planning sessions that result in a trip being created in GoTeam — target >60%
- Organiser satisfaction with destination suggestions — target >70% positive
- Retention: do organisers who use the planning module return to create a second trip at higher rates than those who skip it?

---

## Open Questions

| Question | Owner | Blocking? |
|---|---|---|
| ~~Date range input UX?~~ **Decided: single scrollable multi-range calendar using `react-date-range`, 12 months ahead.** | Engineering | Closed |
| ~~Zero overlap?~~ **Decided: show partial coverage ranked by participant count. No dead end.** | Design | Closed |
| ~~Organiser weighting?~~ **Decided: equal weight to all participants.** | Product | Closed |
| Should vibe swipes appear before or after the availability calendar in the participant flow? Completing swipes first may improve engagement; availability first is more logical. | Design | No |
| How do we handle a participant who submits after the organiser has already triggered reconciliation? Can they still submit, and does it update the results? | Engineering | No |
| Does the destination vote happen in the same participant flow, or is it a separate link sent after reconciliation? | Design | No |

---

## Architecture Notes

- This module sits **before** CreateTrip in the flow, feeding its outputs into CreateTrip on completion
- New `planning_sessions` table in Supabase needed, linked to `trips` once created
- New `planning_responses` table per participant, linked to `planning_sessions`
- Availability stored as JSONB array: `[{start: 'YYYY-MM-DD', end: 'YYYY-MM-DD'}, ...]`
- Vibe swipes stored as JSONB object: `{beach_mountains: 'beach', adventure_chill: 'chill', ...}`
- Overlap calculation runs client-side at reconciliation time (≤10 participants, no performance concern)
- Destination suggestions via single Anthropic API call at reconciliation time with full group context
- Organiser dashboard polls Supabase for submission count in real time (or on page refresh in v1)

---

## Timeline Considerations

- No hard deadline
- Suggested phasing:
  - **Phase 1:** Dates module only — event type selection, skip logic, availability collection, overlap calculation, organiser-triggered reconciliation
  - **Phase 2:** Vibe swipes + destination suggestions — depends on Phase 1 participant flow being solid
  - **Phase 3:** Destination voting — depends on Phase 2 suggestion quality being validated
- Phase 1 is independently valuable and shippable without Phase 2
