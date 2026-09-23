# Living PRD

> Module 4 · Production Specs. Refactor for readability; extract a living PRD that stays true as the build evolves.

## Problem

_What user problem does this solve? Tie to the validated hypothesis._

Sales reps don't use the CRM. Weekly active adoption is 18% across licensed seats, internal CSAT is 2.1 / 5, and 63% of reps keep a shadow spreadsheet because logging a deal update takes 8 required fields and ~11 minutes (vs. a 2-minute target). Reps say they don't know where anything lives and that no screen tells them who to call next.

Tie to the validated hypothesis: this PRD exists because the prototype was built to test a specific belief —

We believe a stripped-back home that shows today's prioritized follow-ups with one-tap logging, and how it relates to the overarching team outcome, will cause reps to use it daily instead of a shadow spreadsheet.

Everything in scope below is in service of that hypothesis. The kill switch still applies: if reps still prefer their spreadsheet after this redesign, the tool isn't the blocker and we pivot rather than build further.

## Users & jobs

- **Primary user:** field sales reps (account execs, sales reps, SDRs) at the company running the CRM — the same population quoted in validation research.
- **Job to be done:** "When I start my day, tell me who to call next, let me log what happened in one tap, and show me how it moves the number I'm measured on."  Secondary jobs surfaced by the prototype:  When I open a deal, tell me where it stands, what's blocking it, and what to do next — without hunting. When I update a deal, let me do it in two fields, not eight.

## Scope

- **In:** Today screen (/) — ranked list of 5–7 follow-ups, each with contact, company, deal value, stage, a plain-language reason for its rank, and last activity. One-tap logging (Called / Emailed / Voicemail / Meeting booked) with undo. Team goal bar with quarter target, booked, and the rep's own share. Live counters: activities logged today, follow-ups remaining, streak. Empty list is the reward state.
Deal detail (/deal/$id) — two-field update (outcome + optional note), next-step date chips (Tomorrow / 3 days / Next week), read-only timeline, contact info. Explicit "2 fields vs 8" contrast.
Deal status (/status/$id) — stage track (Discovery → Evaluation → Proposal → Negotiation → Closed) with done/current/upcoming states, days in stage, deal value, last and next scheduled activity, health (On Track / At Risk / Blocked), and a prominent Primary Blocker panel (category, explanation, source, duration, why it matters). Honest "not enough information" state when no blocker is known.
Unblock guidance (/unblock/$id) — three prioritized recommendations (action, why it matters, owner, timing, CTA), first one marked "Recommended next move". Behavior adapts to On Track / At Risk / Blocked / unknown-blocker deals. Skeleton loading state. Completing an action logs it, updates status, and returns the rep to deal status. Fallback "Review deal activity" when no recommendation exists.
Evidence page (/evidence) — the five validation metrics, three verbatim rep quotes, hypothesis, kill switch, and a design-decision → evidence table.
- **Out (explicitly):** Real persistence — all state is in-memory and resets on reload.
Authentication, accounts, roles, permissions.
Backend of any kind; no API, no database.
Search, pipeline/board views, list filtering beyond the built-in ranking.
Admin settings, goal configuration, user management.
Real integrations (telephony, email, calendar, Salesforce/HubSpot sync).
Mobile-native apps (the web UI is desktop-first, responsive enough for a narrow laptop window).

## Requirements

| # | Requirement | Priority | Acceptance criteria |
|---|---|---|---|
| 1 | Home opens on a ranked "who to call next" list | Must | 5–7 follow-up cards shown on load, ordered by priority; each card shows contact, company, value, stage, rank reason, last activity |
| 2 | One-tap activity logging | Must | One click on Called / Emailed / Voicemail / Meeting booked logs the activity, removes the card from the list, and updates counters immediately — no form |
| 3 | Undo on every log | Must | Every logged activity offers Undo; undoing restores the card and reverses counters and goal progress |
| 4 | Team goal bar tied to logging | Must | Goal bar shows quarter target, booked, gap, and "your share"; every logged activity visibly moves both team and rep progress |
| 5 | Empty list is the reward | Should | When all follow-ups are done, the screen shows a clear "you're done for today" state, not a blank page |
| 6 | Two-field deal update | Must | Deal detail allows an update with outcome + optional note and a next-step chip; saving logs it to the timeline. Explicitly contrasted with the 8-field status quo |
| 7 | Deal stage at a glance | Must | Deal status shows the 5-stage track with completed / current / upcoming visually distinct and days in current stage; stage identifiable within ~2 seconds |
| 8 | Deal health and primary blocker | Must | Health (On Track / At Risk / Blocked) shown distinctly from the blocker itself; blocker panel shows category, explanation, source, duration, and why it matters |
| 9 | Never fabricate a blocker | Must | When blocker information is unavailable, the screen says "We don't have enough information to identify the blocker" and offers "Review deal activity" — no invented blocker |
| 10 | Prioritized unblock guidance | Must | Unblock screen shows 3 recommendations with action, why, owner, timing, CTA; the first is visually distinct as the recommended next move |
| 11 | Guidance adapts to deal condition | Should | On Track deals get momentum-preserving recommendations and a "keep this deal moving" framing; At Risk deals get urgency-ordered actions; Blocked deals make the blocker dominant |
| 12 | Completing an action closes the loop | Must | Completing a recommendation logs the activity, shows confirmation with undo, updates last activity, clears the blocker when the action resolves it, and returns the rep to deal status with updated state |
| 13 | Loading and empty states are honest | Should | Unblock screen shows a skeleton preserving page structure while loading; no misleading placeholders; no-recommendation case explains why and offers the fallback |
| 14 | Recent activity readable in place | Should | Deal status and deal detail show the activity timeline without leaving the screen; session-logged entries appear at the top marked "Just now" |
| 15 | Evidence page links design to data | Could | Every major screen choice is traceable to a metric or verbatim quote on the Evidence page |

## Data & events

_What gets stored, what gets tracked._

Mocked (in-app sample data, resets on reload):

Six fictional deals (src/data/deals.ts): contact, title, company, value, stage, rank reason, last activity, priority, phone, email, timeline.
Team goal (src/data/goal.ts): quarter target 1,200,000; booked 862,000; rep share target 180,000; rep booked 121,500; 19 days left; 4,200 modelled value per logged activity.
Deal status (src/data/status.ts): stage, days in stage, next scheduled activity, health, blocker, and three recommendations per deal. Sable Studio deliberately has a null blocker to exercise the "not enough information" state.
Real (would need to exist in production):

A deals/CRM data source with contacts, stages, values, and activity history.
A team quota / bookings source for the goal bar.
A rules or scoring service for follow-up ranking ("why it's ranked here") — currently hardcoded reasons.
A blocker-detection and recommendation engine — currently hand-written per deal.
Events logged in session (not persisted):

activity_logged — deal id, kind (Called / Emailed / Voicemail / Meeting booked), optional note, optional next step.
activity_undone — activity id.
recommendation_completed — deal id, recommendation id, whether it resolved the blocker.
Production analytics would need these as real events, plus screen-level funnels (Today → log; Deal → status → unblock → complete) to measure the hypothesis: weekly active reps and logged activities per rep.

## Open questions

Ranking logic. What actually determines "who to call next"? The prototype uses hand-written reasons; a real system needs a defensible, explainable ranking model.
Value per activity. The goal bar moves by a flat 4,200 per log — a modelling fiction. What is the honest link between an activity and pipeline movement?
Blocker detection. Can blockers be inferred from real CRM data (staleness, missing fields, stage duration), or must reps/managers declare them? The recommendation engine depends on this answer.
Persistence & sync. Where does the source of truth live — is this a layer over the existing CRM, or a replacement? This decides the entire backend.
Measurement plan for the hypothesis. What baseline and lift in weekly active reps and logged activities count as success, and over what window? What instrumentation do we ship to see it?
Team goal attribution. How is "your share" of the team target computed fairly across roles (AE vs SDR)?
Undo semantics in production. In-memory undo is trivial; undo against a real CRM write is a product decision (grace window? soft delete?).
Kill-switch threshold. "Reps still prefer their spreadsheet" needs an operational definition before the next validation round — what behavior, measured how, decides pivot vs. persevere?
