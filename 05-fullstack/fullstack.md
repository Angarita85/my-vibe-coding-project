# Full-Stack: Data, Access Rules, Edge Cases, Deploy

> Module 5 · Full-Stack. Add data schemas, access rules, and edge cases; stress-test and deploy.

## Deployed link

https://reps-relay.lovable.app

_____


## Data schema

| Entity | Key fields | Notes |
|---|---|---|
| recommendations.kind | activity_kind | so completing a recommendation no longer guesses the activity type from the CTA's wording. |
| deals.due_at | timestamptz, nullable | the actual basis for "today's follow-ups", replacing "everything not yet logged". |
| deals.is_open | boolean, default true | separates "done for today" from "closed deal"; today those are the same thing. |
| profiles.display_name | backfill trigger | a row created on sign-up, so no screen has to fall back to "Alex". |
| Derived, not stored | daysInStage (from stage_entered_at), blockingFor (from blocker_since), last-activity wording (from activities.occurred_at) | These are currently strings in the mock and must never become stored strings. |
| Seed data | six sample deals, statuses, recommendations, one goal row | inserted per new user so an empty account isn't an empty screen during validation. |

## Access rules

_Who can see / do what? Where are the auth boundaries?_

| Rep-private baseline | A rep can see, create, edit and delete only their own rows. | Enforced on all six tables by comparing the row's owner to the signed-in user. |
| profiles | A rep reads and edits only their own profile row. | Row-level security prevents cross-profile visibility. |
| Anonymous access | Nothing is public. | No anonymous read access on any table; a signed-out visitor sees nothing. |
| Child row ownership | activities, deal_statuses and recommendations each carry their own owner column. | Stamped directly rather than inheriting through the deal, so a policy check never has to join. |
| Role architecture | No roles table yet. | There is no manager or admin view in scope. When team-level visibility is requested, it must be a separate roles table with a security-definer check — never a flag on the profile. |
| Write validation | Writes are validated on insert as well as read. | A rep cannot create a row owned by someone else. |

## Edge cases hardened

| Case | Before | After |
|---|---|---|
| First-run state | New account, no deals | A real empty state that explains what to add — not a blank screen, not seeded silently without saying so |
| Completion state | All follow-ups logged | The existing "you're done" reward state, unchanged |
| Missing data / Fallback | Deal has no blocker (Sable today) | "We don't have enough information to identify the blocker." plus the Review deal activity fallback. Never invent one |
| Missing data / Fallback | Deal has no recommendations | Explain why and offer the fallback — do not render three empty cards |
| Concurrent state | Recommendation already completed elsewhere | Show it as done on load; don't allow a second log |
| Failure / Offline | Write fails (offline, RLS rejection) | Roll the optimistic update back, tell the rep in plain words, keep the action retryable |
| Reversal / State sync | Undo after the write landed | Delete the activity row and reverse the goal progress; if undo fails, say so rather than silently diverging |
| Reversal / State sync | Blocker cleared, then undone | Health returns to its stored value — health must be derived, never overwritten by the undo path |
| Concurrent state | Two tabs / two devices | Refetch on focus; last write wins, but never merge a stale goal total |
| Schema mismatch | Deal stage is "Pilot" | The display pipeline has no Pilot step but the deal data does. Either show it or map it — don't render an empty track |
| Data formatting | Deal value is zero or null | Render "—", not "$0", which reads as a real number |
| Auth / Session | Session expires mid-flow | Return to sign-in and come back to the same screen afterwards |
| Layout / UI constraints | Very long blocker text / company name | Wrap, don't clip — the blocker panel is the whole point of the screen |

## Stress test results

I tried spam clicking. It held the line and did not break.

_____
