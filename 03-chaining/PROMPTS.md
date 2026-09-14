# PROMPTS.md: Living Prompt Pack

> Module 3 · Prompt Chaining. Re-architect the build with prompt chains; capture the reusable ones here.

## How to use this pack

_Each prompt is a reusable step. Chain them: the output of one becomes the input to the next._

## Prompt chain: Deal generation improvement chain

### Step 1: Add a deal-status experience that helps sales reps quickly understand where a deal stands, what is blocking it, and what needs to happen next.
```
Build the next screen in the existing Reps Relay prototype: a Deal Status screen.

Use the existing fictional deal data and preserve the current dark "call-sheet" visual system, desktop-first responsive design, routing, and in-memory session state.

The screen should answer these questions in order:

1. Where is this deal in the sales process?
2. What is currently blocking it?
3. Why is it blocked?
4. What needs to happen next?

Add a visual deal progression:
Discovery → Evaluation → Proposal → Negotiation → Closed

Clearly distinguish completed, current, and upcoming stages.

Display:

* Deal value
* Days in current stage
* Last activity
* Next scheduled activity
* Overall deal health: On Track / At Risk / Blocked

Add a prominent "Primary Blocker" section showing:

* Blocker category
* Plain-language explanation
* Who/what is causing the blocker
* How long it has been blocking the deal

Add a "Why this matters" explanation describing the likely impact of the blocker on closing.

Add a primary CTA:
"How do I unblock this deal?"

Create the route for this CTA to lead to the next screen, but do not build the Unblock Guidance screen yet.

Do not modify or rebuild Today, Deal Detail, or Evidence. Do not add authentication, backend persistence, search, pipeline views, or admin functionality.
```

### Step 2: Turn deal status into actionable guidance by defining the rules for different deal conditions, including loading, missing data, errors, and successful actions.
```
Build the Unblock Guidance screen connected to Deal Status.

The purpose of this screen is to answer:

"What should I do next to get this deal to closure?"

Start with:
"Unblock this deal"

Then summarize:
"Your deal is blocked by {{Primary Blocker}}. Here's the fastest path forward."

Provide 3 prioritized recommendations. Each recommendation should include:

* Action
* Why it matters
* Owner
* Timing
* CTA

Make the first recommendation visually distinct as the "Recommended next move."

Implement these behavioral rules:

IF the deal is On Track:

* Show "How do I keep this deal moving?"
* Focus recommendations on maintaining momentum.

IF the deal is At Risk:

* Explain the primary risk.
* Prioritize actions based on urgency and likely impact.

IF the deal is Blocked:

* Make the blocker the dominant information.
* Show a prioritized sequence of actions.
* Make the first action the recommended next move.

IF blocker information is unavailable:

* Do not fabricate a blocker.
* Show "We don't have enough information to identify the blocker."
* Provide "Review deal activity" as the fallback action.

IF the screen is loading:

* Show a skeleton state.
* Preserve the page structure.
* Do not show misleading placeholder information.

IF the recommended action is completed:

* Show a confirmation state.
* Update last activity.
* Update deal status when appropriate.
* Return the rep to Deal Status.
* Reflect the updated in-memory session state across relevant screens.

IF no recommended action is available:

* Explain why no recommendation is available.
* Provide "Review deal activity" as the fallback.

Keep Today, Deal Detail, Evidence, and the existing design system unchanged except where necessary to support this flow.

No backend or persistence is required.
```

### Step 3: Polish the Deal Status experience without disrupting the underlying logic or changing unrelated screens.
```
Refine only the Deal Status screen.

Before editing, audit the screen against these questions:

* Can the rep understand the deal stage within 2 seconds?
* Can they identify the primary blocker within 2 seconds?
* Is deal health clearly distinguished from the specific blocker?
* Is the most important information visually prioritized?
* Is the primary CTA obvious?
* Can the rep understand recent activity without leaving the screen?
* Does the screen feel like a sales tool rather than a generic CRM dashboard?
* Does it maintain the existing dark "call-sheet" visual system?

Identify the gaps, then make only the changes necessary to address them.

Do not:

* Rebuild the screen
* Change the underlying logic
* Change the data model
* Modify Today
* Modify Deal Detail
* Modify Evidence
* Modify Unblock Guidance
* Change routing
* Add new functionality

Preserve the existing information architecture and content unless a change is necessary to improve comprehension or visual hierarchy.

The final experience should make this mental model immediately obvious:

WHERE IS THE DEAL?
↓
WHAT IS BLOCKING IT?
↓
WHY IS IT BLOCKED?
↓
WHAT SHOULD I DO NEXT?

Verify that the "How do I unblock this deal?" CTA still routes correctly to Unblock Guidance.
```

## Reusable techniques learned

- Using the readme and the framework helped to refine my prompts versus me authoring all of the details myself. What the prompt generated was good enough for what I wanted even though it wasn't an exact match.
- Generating the chain one piece at a time was helpful. It allowed me to check that nothing broke.

## What broke (and the fix)

_Where a single mega-prompt failed and chaining fixed it._

Thankfully nothing broke so I didn't have to fix it


_____
