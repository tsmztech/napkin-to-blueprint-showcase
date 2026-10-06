---
document_type: spec
spec_type: integration
spec_id: FEAT-04.SPEC-007
spec_name: Swap Alternatives Generation
spec_slug: swap-alternatives-generation
parent_feature: FEAT-04
parent_feature_name: One-Tap Meal Swap
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Integration Spec: Swap Alternatives Generation

## Overview

**Name:** Swap Alternatives Generation
**ID:** FEAT-04.SPEC-007
**Type:** Integration
**Purpose:** Produces a scoped, one-slot set of candidate alternatives for a swap when the underlying plan was AI-generated, using the same AI text/plan-generation capability that built the original plan.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap

## Scope and Non-Goals

**In Scope:**
- Requesting one-slot candidate recipes from the AI text/plan-generation capability when the target Weekly Plan's origin is AI-generated
- Receiving generated candidates back and handing them to FEAT-04.SPEC-008 for the same safety/schedule filter original generation uses
- User-facing behavior when the capability is slow, unavailable, or rejects a request
- Disclosure of what data is shared with the capability for this scoped request

**Non-Goals:**
- Sourcing alternatives for a manually-built plan -- those come from a direct recipe-library filter with no AI involvement (FEAT-04.SPEC-008), keeping free-tier households at zero AI cost (scope-boundaries.md SC-16)
- Choosing the AI vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate
- The original weekly plan generation request -- owned by FEAT-03.SPEC-010 (the sibling Integration spec for full-week generation); this spec covers only the scoped, single-slot swap request
- Applying the safety/schedule filter to returned candidates, or producing the scarcity explanation -- owned by FEAT-04.SPEC-008; this spec only produces raw candidates for that filter to evaluate

## Capability Category

**Category:** AI text/plan generation
**Dependency Source:** ASMP-30 -- "AI text/plan-generation capability -- Required to generate the weekly dinner plan and swap alternatives on the paid tier" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "AI text/plan generation (ASMP-30)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-03, FEAT-04; Integration Specs: FEAT-03.SPEC-010, FEAT-04.SPEC-007)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Maya sees a short list of freshly-generated safe alternatives when she swaps a meal on an AI-originated plan | Swap a meal / See safe alternatives only | FEAT-04.SPEC-001 (Meal Swap Direct) |
| Sam sees the same freshly-generated alternatives when suggesting a swap on an AI-originated plan | Suggest a swap | FEAT-04.SPEC-002 (Suggest a Swap) |
| The alternatives set reflects the household's schedule constraint and avoids repeating dinners already in the week's plan | See safe alternatives only | FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Slot schedule constraint | Weekly Plan -- weekly_schedule (the night's time limit, if any) | A swap alternatives request is made on an AI-originated plan | The capability needs the same time-fit constraint original generation used, so alternatives fit the household's stated weeknight limit (Weekly Planning consistency; success-metrics.md, Weeknight Time-Fit Accuracy) |
| Recently-served recipe names | Planned Meal -- recipe (names of dinners already in the current week's plan) | Same as above | Prevents the capability from generating a repeat of a dinner already planned that week |
| Household size | Household -- (member count, functionally) | Same as above | The capability sizes portions and cost estimates for the returned recipe candidates |

Dietary Rule content -- allergies, religious rules, vegetarian settings, dislikes -- never leaves the product through this integration. The capability generates candidates from schedule and repeat-avoidance constraints only; every candidate is safety-checked inside the product by FEAT-02 before it can reach either screen, so the capability never needs and is never given the household's dietary rule data (consistent with the dependency map's Data Sensitivity note: FEAT-04 "neither stores nor displays raw Dietary Rule records itself"). Budget, member names, and every other Household or Member Profile field also never leave the product through this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Candidate recipe (name, ingredients, steps, cook time, rough cost) | The capability returns generated candidates for the request | Recipe -- treated as candidate data passed directly to FEAT-04.SPEC-008's safety/schedule filter; only candidates that pass are ever displayed or could be written to a Planned Meal |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Candidates generated | The capability successfully returns one or more candidate recipes for the request | None directly -- candidates are handed to FEAT-04.SPEC-008 for filtering, which is the point at which any user-visible result is produced | The alternatives list renders on FEAT-04.SPEC-001 or FEAT-04.SPEC-002 once FEAT-04.SPEC-008's filter completes | FEAT-04.SPEC-008 |
| Generation returned zero candidates | The capability completes but returns no candidates for the constraints given | None | FEAT-04.SPEC-008 treats this as the zero-alternatives case and produces its scarcity explanation | FEAT-04.SPEC-008 |
| Generation failed | The capability reports an error processing the request | None | Handled as the "Capability Rejects" degradation path below | FEAT-04.SPEC-001, FEAT-04.SPEC-002 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-04.SPEC-001 (Meal Swap Direct) | The screen's Loading Alternatives indicator continues showing past the couple-of-seconds baseline; after 8 seconds a note appears alongside it: "Still finding alternatives -- this is taking longer than usual." The Current Meal card and the rest of the plan remain fully usable while waiting. | Error (alternatives fetch) state: "Couldn't load alternatives. Check your connection and try again." with a Retry button; the original meal is unchanged and remains on the plan. | Same as Capability Down -- from the user's perspective a rejected request and an unavailable capability both present as the alternatives-fetch failure with the retry option; the original meal is unaffected either way. |
| FEAT-04.SPEC-002 (Suggest a Swap) | Same behavior as FEAT-04.SPEC-001: the Loading Alternatives indicator persists with the same "Still finding alternatives" note after 8 seconds. | Error (alternatives fetch) state: "Couldn't load alternatives. Check your connection and try again." with a Retry button; no suggestion is created and the current meal is unaffected. | Same as Capability Down. |
| FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation) | N/A -- this spec only evaluates whatever candidates arrive; it has no independent request to this capability and cannot itself experience slowness. | N/A -- if no candidates arrive, FEAT-04.SPEC-008 never runs its filter on AI-originated candidates for that request; the calling screen's degradation message covers the user experience. | N/A -- same reasoning as Capability Down. |

Every degraded path leaves the underlying plan and any prior suggestion state unchanged -- no half-created Swap Suggestion or partially-written Planned Meal results from a failed or slow request.

## Consent and Disclosure

- **Continuation of AI plan generation consent** -- The household already disclosed and consented to sharing plan-generation data with the AI text/plan-generation capability when it subscribed to the paid tier and received its first AI-generated plan (FEAT-03.SPEC-010's disclosure moment). This integration is a scoped continuation of that same capability for a single slot rather than a new relationship, so no additional disclosure moment interrupts the swap flow. The "How your plan is generated" reference available from account settings (owned by FEAT-03.SPEC-010) documents this integration's scope alongside full-week generation.
- **What is never shared** -- Dietary Rule content of any kind, budget, and all Member Profile and Household fields beyond household size stay inside the product for every swap-alternatives request, as stated in Data Exchanged above; this boundary is documented in the same "How your plan is generated" reference.

## Edge Cases

- **A generation request is made for a plan that was AI-generated but has since been downgraded to a free-tier household mid-week** -- The request is not sent; per XBR-05, a downgrade never removes existing plan data, but new AI generation (including scoped swap requests) stops immediately on downgrade, so FEAT-04.SPEC-008 routes to the manual recipe-library filter instead for any swap attempted after the downgrade takes effect.
- **The same generation request is somehow submitted twice (a retry after a slow response that eventually also returns)** -- Both responses are evaluated by FEAT-04.SPEC-008 independently; whichever the user acts on first is the one applied, and the other's candidates are simply discarded once the screen leaves the Loading Alternatives state.
- **Candidates arrive for a slot the user has since navigated away from** -- The response is discarded; no notification or list update occurs on a screen the user is no longer viewing.
- **The capability goes down mid-request after already committing to generate** -- No partial candidate list is ever shown; the screen shows the full Capability Down message only once the request is confirmed as failed, never a partially-populated list.
- **Generation returns candidates but every one fails FEAT-04.SPEC-008's safety/schedule filter** -- Treated identically to "Generation returned zero candidates" from the user's perspective: FEAT-04.SPEC-008 produces its zero-alternatives scarcity explanation, since no candidate reaches the user regardless of whether the capability generated none or generated some that were filtered out.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-001 (Meal Swap Direct) | Triggered by (inbound) | Requests a scoped alternatives set when the plan is AI-originated |
| FEAT-04.SPEC-002 (Suggest a Swap) | Triggered by (inbound) | Same request path for Sam's suggestion flow |
| FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation) | Affects (outbound) | Receives raw candidates for safety/schedule filtering and routing |
| FEAT-03.SPEC-010 (AI Weekly Dinner Plan Generation Integration) | References (outbound) | Sibling Integration spec for full-week generation; shares the same capability and consent moment |

## Analytics and Success Signals

- **swap_alternatives_generation_requested** (slot night) -- supports success-metrics.md: "One-Tap Swap Completion"
- **swap_alternatives_generation_succeeded** (candidate count, request duration) -- supports success-metrics.md: "One-Tap Swap Completion"
- **swap_alternatives_generation_degraded** (condition: slow / down / rejected; screen: spec ID) -- N/A -- no Stage 2 metric measures generation-capability degradation specifically; retained so the product's tolerance for capability trouble on the swap path is observable.

## Acceptance Criteria

**FEAT-04.SPEC-007-AC-01:** Given Maya opens FEAT-04.SPEC-001 for a slot on an AI-originated plan and taps Swap, when this integration requests candidates, then the request carries the night's schedule constraint, the week's already-planned recipe names, and household size, and no dietary rule data.

**FEAT-04.SPEC-007-AC-02:** Given the capability returns three candidate recipes, when this integration receives them, then they are handed to FEAT-04.SPEC-008 for the safety/schedule filter before anything is shown to Maya.

**FEAT-04.SPEC-007-AC-03:** Given the capability returns zero candidates for the request, when FEAT-04.SPEC-008 processes the empty result, then the zero-alternatives scarcity explanation is shown.

**FEAT-04.SPEC-007-AC-04:** Given the capability takes longer than 8 seconds to respond, when Maya is still waiting on FEAT-04.SPEC-001, then the note "Still finding alternatives -- this is taking longer than usual." appears alongside the loading indicator.

**FEAT-04.SPEC-007-AC-05:** Given the capability is unavailable, when Maya taps Swap on an AI-originated plan's slot, then FEAT-04.SPEC-001 shows "Couldn't load alternatives. Check your connection and try again." with a Retry button, and the original meal is unchanged.

**FEAT-04.SPEC-007-AC-06:** Given the capability rejects the request, when the rejection is received, then Sam's FEAT-04.SPEC-002 shows the same fetch-failure message as the capability-down case, and no suggestion is created.

**FEAT-04.SPEC-007-AC-07:** Given a household downgrades to the free tier mid-week, when Maya attempts a swap on a slot from her prior AI-generated plan after the downgrade, then no request is sent to this capability and FEAT-04.SPEC-008 routes to the manual recipe-library filter instead.

**FEAT-04.SPEC-007-AC-08:** Given Maya navigates away from FEAT-04.SPEC-001 while a request is in flight, when the capability's response later arrives, then it is discarded with no effect on any screen.

**FEAT-04.SPEC-007-AC-09:** Given generation returns candidates that all fail the safety/schedule filter, when FEAT-04.SPEC-008 completes its evaluation, then the zero-alternatives scarcity explanation is shown, identical to a zero-candidate response from the capability.

**FEAT-04.SPEC-007-AC-10:** Given this is Sam's first swap suggestion attempt on an AI-originated plan, when he views what the app shares with the AI capability, then the "How your plan is generated" reference confirms dietary rule data is never sent.

**FEAT-04.SPEC-007-AC-11:** Given the capability goes down mid-request, when the failure is confirmed, then no partially-populated alternatives list is ever shown -- only the full Capability Down message.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 3 | 3 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 6 (2 screens x 3 conditions; SPEC-008 rows are N/A) | 6 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |
