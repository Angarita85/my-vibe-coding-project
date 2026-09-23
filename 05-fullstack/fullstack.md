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

## Access rules

_Who can see / do what? Where are the auth boundaries?_

A rep can see, create, edit and delete only their own rows. Enforced on all six tables by comparing the row's owner to the signed-in user.

## Edge cases hardened

| Case | Before | After |
|---|---|---|
| Empty / first-run state | New account, no deals | A real empty state that explains what to add — not a blank screen, not seeded silently without saying so |
| Bad / malicious input | All follow-ups logged | The existing "you're done" reward state, unchanged |
| Failure / offline | Write fails (offline, RLS rejection) | Roll the optimistic update back, tell the rep in plain words, keep the action retryable |

## Stress test results

_What you threw at it, and what held / broke._

_____
