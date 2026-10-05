# GoTeam - Product Vision & Roadmap

**Status:** Living document  
**Author:** Melody  
**Last updated:** June 2026

---

## What GoTeam is

GoTeam is a group trip and event coordination tool that removes the chaos of planning anything with other people. It handles the hardest parts: finding dates that work, agreeing on a destination, collecting accommodation preferences. Frictionless by default, link-based until a native app is justified by user demand.

The organiser does the setup. Everyone else just answers questions. GoTeam does the reconciliation.

Participants need nothing more than a link - no download, no account, no friction. This is GoTeam's current competitive advantage and will remain the default experience until a native app is justified by user demand.

---

## Core principles

- **Link-based, no accounts required** - participants should never hit a signup wall. The link *is* the experience.
- **Modular** - each coordination problem (dates, destination, accommodation, activities) is a separate module that activates only when relevant. Nobody answers questions that don't apply to their trip.
- **Organiser-first** - the product's primary value is to the organiser. Participant experience should be as frictionless as possible, but the organiser is the person who chose GoTeam and who returns to it.
- **AI does the work** - reconciliation, destination suggestions, itineraries, and summaries are AI-generated. The product's value compounds with the quality of those outputs.
- **Two short forms beat one long one** - never stack modules into a single overwhelming flow. Separate links for separate moments.

---

## The full intended journey

1. **Plan** - organiser creates a planning session, shares a link
2. **Everyone submits** - availability (calendar) + vibe swipes (destination module), in one session
3. **Organiser reconciles** - triggers results when everyone (or enough people) has submitted
4. **Decide** - dates confirmed, destination shortlisted, group votes, organiser makes final call
5. **Everyone submits accommodation preferences** - second link, separate moment
6. **Results** - AI reconciliation produces accommodation brief with pre-built Booking.com/Airbnb affiliate search links
7. **Organiser books externally** - clicks through GoTeam's affiliate links to book

---

## Current state (June 2026)

**Built and live at goteam.buildingmelody.com:**
- Organiser creates a trip (destination, dates, group size, bedroom minimum)
- Participants receive invite link, submit accommodation preferences (budget, location tolerance, must-haves, dealbreakers, notes)
- AI reconciliation produces accommodation brief
- Results page with Booking.com/Airbnb search links (affiliate links - to be properly implemented)
- Organiser confirmation email on trip creation
- Organiser notification email on each participant response

**Recently shipped (June 2026):**
- Must-have and dealbreaker chips on participant form (en-suite, private bathroom, pool, wifi, etc.)
- Web Share API on organiser confirmation screen with clipboard fallback
- Organiser email redesigned: link box + "Or forward this email to everyone"

---

## Roadmap

### Now - Planning Module (Phase 1: Dates)

The planning module sits before the existing accommodation flow and is modular - each sub-module activates only when relevant.

**Event types:**
- Trip away - activates dates module + destination module + accommodation flow
- Day out - activates dates module, destination optional, no accommodation
- Gathering - activates dates module only, no destination, no accommodation

**Flow:**
1. Organiser selects event type
2. Organiser asked: "Already decided on dates?" and/or "Already decided on destination?" - yes skips that module
3. Planning link shared with group
4. Each participant completes in one session: availability calendar + vibe swipes (if applicable)
5. Organiser sees submission count, triggers reconciliation manually ("Everyone's in - see results")
6. Results: top 1–3 date windows by participant coverage, then destination suggestions (Phase 2)
7. Confirmed dates + destination feed into existing CreateTrip flow

**Date input:** single scrollable calendar, 12 months ahead, multi-range selection via `react-date-range`

**Zero overlap:** always show partial coverage ranked by participant count - never a dead end

**Key decisions locked:**
- Participants submit everything in one session, no waiting for others
- No results visible to participants at any point
- Organiser triggers reconciliation, not the app
- Organiser's own availability and vibe preferences carry equal weight to participants

---

### Next - Planning Module (Phase 2: Destination)

**Vibe swipes - dynamic axes:**
- AI infers which axes are already implied by the organiser's trip description and skips them
- Remaining axes shown as binary swipe choices to each participant
- Five possible axes:
  1. Beach vs Mountains
  2. City vs Countryside
  3. Adventure vs Chill
  4. Lively vs Low-key
  5. Foodie vs Active
- Minimum 2, maximum 5 axes per participant

**Destination suggestions:**
- Generated at reconciliation time via Anthropic API
- Inputs: aggregated vibe profile + agreed date window (for climate/season) + event type + trip description
- Output: 3 destination suggestions, each with name and 1–2 sentence rationale
- Organiser can request one fresh set if they reject all three

**Group vote:**
- Organiser shares 3 suggestions with group
- Single-choice vote per participant
- Results visible to organiser only
- Organiser makes final call, chosen destination feeds into CreateTrip

---

### Next - Affiliate Revenue (Booking.com)

- Sign up to Booking.com Partner Programme
- Append affiliate tracking ID to all Booking.com URLs in results page
- No UI change, no user friction
- Revenue per completed booking referred
- Add Airbnb affiliate once Booking.com is live (Airbnb programme requires approval)
- Extend affiliate model to restaurant and activity bookings in future (OpenTable, Viator, GetYourGuide etc.)

---

### Later - Itinerary Feature (Premium)

- AI-generated day-by-day itinerary based on destination, dates, group size, and vibe profile
- Includes suggested activities, restaurants, transport between locations
- First candidate for a paid/premium tier
- Does not require accounts - organiser receives itinerary, can share as a link or PDF
- Pulls on: destination + agreed dates + group vibe profile already collected during planning module

---

### Later - Weather Integration

- Weather data API (e.g. Open-Meteo, free tier; or Weather API) surfaced on results page
- Shows forecast or historical climate data for the destination and date window
- Useful framing: "Late September in Porto: average 22°C, low rainfall" alongside accommodation results
- Also useful during destination suggestion phase - informs which suggestions are climatically appropriate

---

### Later - Transport & Getting There

- Citymapper API integration for in-destination transport
- Useful for Day out and city-based Trip away event types
- Could surface on results page: "Getting around [destination]"
- Requires destination to be confirmed first

---

### Later - Ticket & Document Storage

- Participants and organiser can upload/store trip documents: flights, event tickets, restaurant bookings, travel insurance
- Lives on a persistent trip page
- Requires accounts and persistent trip storage (see Future Vision below)

---

### Later - Activity & Restaurant Affiliate Bookings

- Extend affiliate model beyond accommodation
- Partners: GetYourGuide, Viator (activities); OpenTable, Resy (restaurants)
- Surfaces on results page and/or itinerary
- Low friction: pre-filtered suggestions based on destination + vibe profile, with affiliate links

---

## Future Vision (accounts era)

The following features require GoTeam to have user accounts and persistent trip storage. This is a significant architectural shift - the current link-based model is GoTeam's competitive advantage and should not be abandoned prematurely. Build accounts only when user behaviour demonstrates clear demand (e.g. users asking to save preferences, access trip history, or return to a past trip).

**Accounts and social graph**
- Users register, create a profile, connect with friends
- Organiser creates a trip by selecting group members from friends list rather than sharing a link
- Participant preferences saved and reusable across trips ("use my saved preferences")
- Trip history visible to all members

**Photo repository**
- Shared photo and video storage within a trip
- Participants upload directly from the trip page
- Makes GoTeam the single home for everything related to a trip, before and after
- Not a replacement for Google Photos shared albums - only worth building if the persistent trip home exists and people are already living in it

**Persistent preference profiles:**
Users submit preferences once and the app remembers them across all future trips. No more re-entering "I'm vegetarian and need an en-suite" every time.

Profile includes:
- Dietary requirements and allergies
- Accessibility needs
- Budget tier preference
- Climate preferences
- Travel style (adventure vs. chill, city vs. countryside)
- Accommodation must-haves and dealbreakers

These profiles power:
- Pre-filled accommodation preference forms (only ask what's changed)
- Trip Companion recommendations (never suggest a steakhouse to a vegetarian)
- Itinerary suggestions filtered to the group's collective needs
- Smarter vibe swipes (skip axes where preference is already established)

**Freemium model (only after accounts and multiple modules are validated)**
- Free tier: one complete trip across all modules (full experience, no credit card)
- Paid tier (£6/month or £30/year): unlimited trips, saved preference profiles, trip history, priority reconciliation, itinerary generation
- Do not build this trigger on a trip count - build it when the product has enough depth to justify the unlock

**B2B (only after clear inbound demand)**
- Travel agents, corporate travel coordinators, hen do planners, team offsites
- White-label or API access
- Do not pursue speculatively

---

## Transport & Getting There

Search and book flights and trains directly within GoTeam, with affiliate revenue per completed booking.

**Flights:**
- Search via Skyscanner API or Kiwi.com API (both have affiliate programmes)
- Show results filtered to the group's destination and confirmed dates
- Affiliate link through to booking on the provider's site
- Stored ticket feeds into Trip Companion context

**Trains:**
- Trainline affiliate programme for UK and European rail
- Particularly relevant for day out and city event types
- Group booking context (e.g. 6 people, same origin)

**Why it works here:**
- GoTeam already knows the destination, dates, and group size
- Pre-populated search removes friction vs. going to Skyscanner cold
- Stored tickets power the Trip Companion ("what time does our train leave?")

**Revenue model:** affiliate commission per completed booking

**Car hire:**
- Rentalcars.com or Kayak affiliate programmes
- Pre-populated with destination, dates, and group size
- Particularly relevant for villa trips where a car is essential

**Prerequisites:** confirmed dates and destination (post-reconciliation), accounts era for ticket storage

---

## Live Trip Dashboard

A personalised, auto-updating task list for every member of the trip. The dashboard is the persistent trip home — the thing that ties the whole product together and gives everyone a reason to keep returning to GoTeam between planning and travelling.

**How it works:**
- GoTeam generates a checklist automatically from what it knows about the trip
- Items are ticked off automatically as they happen (preferences submitted, accommodation booked, Splitwise group created)
- Separate views for organiser and participants
- Accessible to everyone in the group

**Organiser dashboard:**
- Book accommodation (links to reconciliation results)
- Share accommodation survey with group
- Set up Splitwise group
- Book flights (links to pre-populated search)
- Create group itinerary
- Check who has/hasn't completed each step

**Participant dashboard:**
- Submit accommodation preferences
- Book your own flights
- Join the Splitwise group
- Add your ticket to the trip
- Check in on arrival

**Why it matters:**
- Makes organiser burden visible and manageable
- Removes the "what still needs doing?" mental load
- Gives everyone in the group a shared source of truth
- Natural entry point for Trip Companion ("ask me anything about your trip")
- The feature that most justifies accounts and a native app

**Prerequisites:** accounts era, persistent trip storage

---

## Ideas & Discovery

A shared ideas list within each trip — restaurants, activities, places to stay — that anyone in the group can add to before and during planning.

**Cross-posting from social media:**
The missing link between how people discover places (Instagram reels, TikTok saves, Google Maps stars) and where group planning actually happens.

- Share a restaurant reel from Instagram directly to a trip's ideas list via the native share sheet
- Same for TikTok, YouTube Shorts, Google Maps saved places
- Each idea stores the source link, title, and any notes
- Group members can upvote or comment on ideas
- Ideas feed naturally into the itinerary builder

**How it works technically:**
- Mobile: Web Share Target API (PWA) or native share extension (app era)
- Web: browser bookmarklet or extension
- Content is saved as a link with metadata (title, image, source) — no content reproduction

**Prerequisites:** persistent trip storage, accounts era for group visibility. Share target API works as a PWA before full native app.

---

## Third-Party Integrations

**Splitwise**
Group expenses are the most universally painful part of group travel. Rather than building expense splitting from scratch, integrate with Splitwise which already does it well.
- "Set up a Splitwise group for this trip" button on the results/trip page
- Splitwise OAuth for the organiser
- Create a group via the Splitwise API pre-named with destination and dates
- Deep link all participants into the Splitwise app to join
- Revenue: none directly, but removes a post-booking friction point that would otherwise cause drop-off
- Requires: accounts era, confirmed trip

---

## Monetisation summary

| Phase | Mechanism | Status |
|---|---|---|
| Now | Booking.com affiliate links on results page | To implement |
| Next | Airbnb affiliate links | Pending approval |
| Later | Restaurant/activity affiliate links (GetYourGuide, OpenTable etc.) | Future |
| Later | Itinerary generation as premium feature | Future |
| Future | Freemium subscription (accounts era) | Future |
| Future | B2B / white-label | Only if inbound demand |

---

---

## Trip Companion

An AI assistant embedded in the trip that knows your context and can answer questions in natural language, by text or voice. Available to everyone in the group once a trip is active.

**What it knows:**
- Accommodation address and check-in/check-out times
- Any bookings stored in the trip (restaurants, activities, flights, tickets)
- Destination, dates, and group size
- Local weather and conditions

**What it can answer:**
- "How do we get to the restaurant we've booked for dinner?"
- "What time do we need to leave to catch our flight?"
- "Where's the nearest pharmacy?"
- "What's the weather like this afternoon?"
- "What's everyone doing tomorrow?"

**Technical approach:**
- Natural language input via Web Speech API (voice) or text
- Directions via Google Maps Directions API (broader global coverage than Citymapper; Citymapper better for public transport in supported cities - consider both)
- Weather via Open-Meteo or similar
- AI layer (Anthropic API) to interpret questions and synthesise answers from available trip context
- Requires accounts and persistent trip data to be genuinely useful

**Prerequisites:** accounts era, ticket/booking storage, persistent trip home

**Intended infrastructure:** Vercel Eve. The Trip Companion is the feature Eve was built for — a durable, stateful agent that knows your trip context and can answer questions across multiple sessions without losing state. Build this when Eve reaches general availability.

---

## Accessibility & Input

- **Voice control** - speak your preferences and availability rather than typing. Particularly useful for mobile users on the go. Built on Web Speech API or a third-party provider. Applies to both organiser setup and participant response flows.

---

## What GoTeam is not (yet)

- A booking engine - GoTeam facilitates, the organiser books externally via affiliate links
- A social network - no accounts, no friend graphs, no feeds in the current model
- A photo sharing app - solved problem without accounts; revisit when persistent trip home exists
- A travel agency - no human intervention, no curated packages
