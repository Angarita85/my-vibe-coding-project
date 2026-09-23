# Engineering Handoff Note

> Module 4 · Production Specs. Open the black box, make the build legible to an engineer.

## What this is

_One paragraph an engineer can read in 60 seconds._

This is a front-end-only validation prototype — five screens that test one hypothesis: reps will use a CRM daily if the home screen shows today's prioritized follow-ups with one-tap logging tied to a visible team goal. There is no backend, no database, no auth, and no persistence: every deal, blocker, metric, and recommendation is hand-written TypeScript mock data, and all user actions live in an in-memory React context that resets on refresh. The code you inherit is well-typed, cleanly organized by feature, and stable end-to-end (typechecked and manually browser-tested), but nothing in it is wired to a real system — your job is likely to replace the mock data layer with a real one while keeping the screen contracts intact.

## Architecture (plain language)

- **Frontend:** Framework: TanStack Start (React 19, Vite 7, file-based routing via TanStack Router). No other router; no Next.js patterns. Routing: five thin route files in src/routes/ — they exist only to bind a URL to a screen component: Route	Screen component / (index.tsx)	TodayScreen /deal/$id (deal.$id.tsx)	DealDetailScreen /status/$id (status.$id.tsx)	DealStatusScreen /unblock/$id (unblock.$id.tsx)	UnblockGuidanceScreen /evidence (evidence.tsx)	EvidenceScreen Features: screens, their data, and shared components live in src/features/<screen>/. Each screen folder holds the screen component; mock data lives in a sibling data/ folder (src/features/deals/data/, src/features/today/data/, src/features/evidence/data/). Shared state: src/features/activity/session-store.tsx — a single React context (SessionProvider + useSession) holding every mutable thing: logged activities, undo, completed recommendations, goal progress, deal health. Styling: Tailwind v4 with design tokens in src/styles.css (dark "call-sheet" theme, amber primary, Space Grotesk display / DM Sans body via Google Fonts <link> in __root.tsx). UI primitives are shadcn-style components in src/components/ui/.
- **Backend / data:** There is none. No server functions, no API routes, no Lovable Cloud/Supabase, no environment variables or secrets. All content is static mock data in TypeScript: src/features/deals/data/deals.ts — six fictional deals (Deal type): contacts, values, stages, timelines. src/features/deals/data/deal-status.ts — per-deal pipeline stage, health, blocker, and three prioritized recommendations (DealStatus, Blocker, Recommendation types). Note the deliberate design rule encoded here: blocker: null means genuinely unknown — never fabricate one (the Sable deal tests this). src/features/today/data/team-goal.ts — team/rep goal figures, including the mock valuePerActivity: 4200. The only "server" code is the framework's own entry (src/server.ts, src/start.ts) — untouched template plumbing.
- **Key flows:** One-tap logging (Today): tap an activity chip → logActivity() in the session store → card leaves the list, goal bar re-renders (booked + logCount × 4200), undo available.
Deal update (Deal Detail): two-field form → logActivity() with note/next step → prepended to the deal's timeline in session state.
Blocker resolution (Deal Status → Unblock Guidance): open a deal's status → see stage track, health, blocker → CTA routes to /unblock/$id → complete a recommendation → activity logged, blocker cleared if resolvesBlocker, health recomputed to "On Track" from session state (not the static data), auto-return to Deal Status. "How do I unblock this deal?" vs. "How do I keep this deal moving?" is chosen by health.
Honest unknown (Sable): no blocker and no recommendations → the screen shows "We don't have enough information to identify the blocker" with a "Review deal activity" fallback. This behavior is intentional and should survive any rewrite.

## What's solid vs. what's duct tape

| Area | State | Notes |
|---|---|---|
| Feature-folder separation | solid | matching the PRD screen names; route files are thin and unambiguous. |
| Typed data contracts | solid | `Deal`, `DealStatus`, `Blocker`, `Recommendation`, and `Health` can be implemented directly by a real API; screens don't reach into data internals. |
| Behavioral rules | solid | blocker taxonomy, recommendation priority, and the `resolvesBlocker` flag are declarative; `healthFor()` derives health from session state rather than mutating source data. |
| Build and browser checks | solid | `tsgo --noEmit` is clean; every screen and the full logging flow were manually verified in a real browser with zero console errors. |
| Everything is mock. | rough | Deals, metrics, quotes, blockers, and recommendations are hand-written constants. The evidence page numbers are scenario fixtures, not measurements. |
| Goal math | rough | Each logged activity adds a flat $4,200 to the goal bar regardless of the deal's actual value. |
| Deal rankings | rough | Rankings are hand-assigned via a `priority` field on each deal — there is no scoring logic to port. |
| Activity IDs | rough | Activity IDs are `dealId-timestamp-random` strings; fine for a session, but not suitable as a persistence key. |
| Automated tests | rough | Verification is manual (Playwright-driven browser checks run during the build); nothing runs in CI. |
| Persistence | rough | Refresh wipes all logged activity, completed recommendations, and cleared blockers. |
| User model | rough | One hardcoded rep name ("Alex") and one user; there is no multi-user or role concept anywhere. |

## Risks & assumptions for the team

Assumption: the mock data is representative. The blocker categories (pricing approval, legal terms, gatekeeping) and the recommendation structure encode one designer's model of sales reality — validate with a real sales lead before treating them as a product spec.
Risk: the session store's shape gets mistaken for a persistence contract. It's a UI cache. A real build needs its own data model (activities, deals, recommendations as server records) — don't bolt persistence onto this context.
Assumption: health is state, not stored truth. The prototype recomputes "On Track" when a blocker is cleared this session; a real system needs to decide whether health is derived (recommended) or a stored field, and define the transition rules.
Risk: the $4,200-per-log goal math could anchor expectations if the demo is shown to stakeholders. It's a demo device, not a forecast model.
Assumption: single-user, single-tenant, desktop-first. No concurrency, no auth, only informal responsive handling below desktop widths.
Not audited: accessibility, performance under real data volume, and browser support beyond the Chromium used in testing.

## How to run it

```
npm install        # or: bun install
npm run dev        # dev server on http://localhost:8080
Typecheck: bunx tsgo --noEmit (or npx tsc --noEmit).
Production build: npm run build, then npm run preview to serve it locally.
Requirements: none beyond Node.js — no environment variables, no secrets, no external services, no database. Fonts load from Google Fonts at runtime.
Data: all sample data is in src/features/*/data/; edit those files to change what the screens show. All names, companies, phone numbers, and figures are fictional.
Where to plug in a backend: implement the Deal / DealStatus shapes server-side, replace the static imports in the feature data/ modules (or the session store) with real fetches, and swap the in-memory mutations in session-store.tsx for API calls. The screens should not need changes.
```
