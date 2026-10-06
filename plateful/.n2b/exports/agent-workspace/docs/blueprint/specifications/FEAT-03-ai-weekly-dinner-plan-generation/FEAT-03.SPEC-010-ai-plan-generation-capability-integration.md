---
document_type: spec
spec_type: integration
spec_id: FEAT-03.SPEC-010
spec_name: AI Plan Generation Capability Integration
spec_slug: ai-plan-generation-capability-integration
parent_feature: FEAT-03
parent_feature_name: AI Weekly Dinner Plan Generation
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Integration Spec: AI Plan Generation Capability Integration

## Overview

**Name:** AI Plan Generation Capability Integration
**ID:** FEAT-03.SPEC-010
**Type:** Integration
**Purpose:** Product boundary to the AI text/plan-generation capability that turns household constraints and candidate recipes into a proposed seven-dinner week.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation

## Scope and Non-Goals

**In Scope:**
- Sending household constraints and candidate recipe data to the AI plan-generation capability and receiving a proposed set of candidate dinners
- The product's behavior when the capability is slow, unavailable, or returns a proposal the product cannot use
- Disclosure of what household data is shared with this capability
- Feeding the raw proposal into the safety check, scaling rule, and budget-fit rule owned elsewhere in this feature

**Non-Goals:**
- Choosing the AI vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for a specific provider
- Verifying the safety of proposed candidates -- owned by Dietary Rules & Allergy Safety Engine (FEAT-02); this spec treats the capability's output as unverified until FEAT-02's check runs
- Scaling quantities or computing budget fit on the proposal -- owned by FEAT-03.SPEC-007 and FEAT-03.SPEC-006 respectively, which run after this integration returns its result
- Swap-time alternative generation -- FEAT-04's own Integration spec (FEAT-04.SPEC-007) covers the AI capability's use during a single-slot swap; this spec covers only full-week generation (FEAT-03.SPEC-003, FEAT-03.SPEC-004)

## Capability Category

**Category:** AI text/plan generation
**Dependency Source:** ASMP-30 -- "AI text/plan-generation capability -- Required to generate the weekly dinner plan and swap alternatives on the paid tier" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "AI text/plan generation (ASMP-30)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-03, FEAT-04; Integration Specs: FEAT-03.SPEC-010, FEAT-04.SPEC-007)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Maya's household receives a proposed seven-dinner week that respects every hard dietary rule, the stated schedule, and the budget | Generate the week's plan | FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation), FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) |
| The proposal favors recipes that use ingredients the household has already logged as on hand | Use up the pantry | FEAT-03.SPEC-003, FEAT-03.SPEC-004 |
| Later proposals favor meals the household has rated highly and avoid ones rated poorly | Learn from ratings over time | FEAT-03.SPEC-003, FEAT-03.SPEC-004 |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Household constraints | Household -- weekly_budget, weekly_schedule; Member Profile -- count of active members | Generation is triggered (FEAT-03.SPEC-003, FEAT-03.SPEC-004) | The capability needs to know the budget ceiling, time constraints per night, and how many people the week must feed |
| Dietary constraints | Dietary Rule -- rule_kind, strength, allergen (per active member, without member-identifying detail beyond what is needed to apply the rule) | Every generation attempt | The capability must avoid proposing candidates that could not pass the household's hard rules, even though FEAT-02's app-side check remains the enforced safety boundary |
| Candidate recipe pool | Recipe -- name, ingredients, cook_time, rough_cost, dietary_badges (starter and imported recipes available to the household) | Every generation attempt | The capability selects and arranges dinners from this pool rather than inventing recipes outside the household's verified candidate set |
| Pantry weighting input | Pantry Item -- item_name (Active items only) | Every generation attempt, paid tier only | The capability weights candidate selection toward dinners that use these ingredients |
| Rating weighting input | Rating -- planned_meal reference, value (thumbs up/down), aggregated per recipe, not per individual member | Every generation attempt | The capability weights candidate selection toward highly-rated meals and away from poorly-rated ones |

Member names, sign-in details, payment information, household budget beyond the weekly figure, and any data belonging to other households never leave the product. Individual members' ratings are never sent broken out by member -- only aggregated per recipe, consistent with product-features.md's Rating Data Sensitivity note that individual ratings are never shown broken out to other members.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Proposed candidate dinners (up to and including seven selections, or a candidate set the household's own selection logic narrows to seven) | The capability returns a generation result | Held transiently as generation working data; each selected candidate becomes a new Planned Meal (recipe, night, meal_kind) only after passing FEAT-02's safety check |
| Generation confidence/coverage signal (e.g., whether the capability found a full week of proposals or a partial set) | The capability returns a generation result | Read by FEAT-03.SPEC-003/004's processing logic to decide between a normal completion and a generation-failure outcome |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Proposal returned, full week available | The capability returns seven or more safety-passable candidates | None directly (the receiving automation processes the proposal into Planned Meals after FEAT-02's check) | None at this boundary -- feedback is owned by the receiving automation's own outcome (FEAT-03.SPEC-003, FEAT-03.SPEC-004) | FEAT-03.SPEC-003, FEAT-03.SPEC-004 |
| Proposal returned, partial or insufficient | The capability returns fewer usable candidates than needed to fill seven nights after safety filtering | None | The receiving automation treats this as a generation failure per its own Outcome Definitions | FEAT-03.SPEC-003, FEAT-03.SPEC-004 |
| Capability request times out | No response is received within the product's expected generation window | None | The receiving automation treats this as a generation failure | FEAT-03.SPEC-003, FEAT-03.SPEC-004 |
| Capability reports an error | The capability responds with an explicit processing error | None | The receiving automation treats this as a generation failure | FEAT-03.SPEC-003, FEAT-03.SPEC-004 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-03.SPEC-001 (Weekly Plan View) | The Loading state's explained-wait message continues to show ("Building this week's plan...") for up to the product's expected generation window (well under a minute, per ASMP-23); if the wait extends meaningfully beyond that, the screen shows "This is taking longer than usual" while continuing to wait rather than failing immediately. | The previous week's plan remains fully visible with the Error banner "This week's plan couldn't be generated. Retry?" and a Retry control; the household is never shown a blank week. | Same as Capability Down -- a rejected request (e.g., malformed constraints) is treated identically to an unavailable capability from the household's point of view: the previous plan stays visible with Retry offered. |
| FEAT-03.SPEC-002 (Free-Tier Plan Placeholder & Upgrade Prompt) | N/A -- this screen never sends a request to the capability; a free-tier household is routed here before any request is made. | N/A -- see above. | N/A -- see above. |

## Consent and Disclosure

- **Generation data-sharing notice** -- Shown once, the first time a household's plan is ever generated (surfaced during FEAT-03.SPEC-004's first-plan flow or, for a household that never previously saw it, alongside its first scheduled generation): "To build your weekly plan, we share your household's dietary rules, budget, schedule, pantry items, and meal ratings with an AI planning service. Recipes and their sources are also shared. Your name, sign-in details, and payment information are never shared." A "How your plan is built" link on FEAT-03.SPEC-001 reopens the same notice at any time; no "cancel" option is offered since generation is core to the paid tier the household has subscribed to, but the household may downgrade (FEAT-14) if it does not want data shared this way.
- **What is never shared** -- Member names, sign-in details, payment information, and any other household's data are stated in the same notice as never leaving the product; individual, member-attributed ratings are also confirmed as never shared -- only aggregated per-recipe ratings.

## Edge Cases

- **Capability returns a proposal referencing a recipe that was removed from the household's pool between candidate-pool read and response** -- The receiving automation discards that candidate before selection; if this drops the usable pool below seven, it is treated as a partial/insufficient result per Inbound Events.
- **The same generation request is somehow submitted twice (e.g., a retry racing an already-in-flight request)** -- Only one generation run is in flight per household at a time, per FEAT-03.SPEC-003's own concurrency rule; a duplicate submission for the same household and week is not sent while a prior request is outstanding.
- **Capability response arrives after the household has already been shown a generation-failure state (very late response)** -- A late-arriving success response for a request the household has already been told failed is discarded; the household's next attempt (manual retry or next scheduled cycle) issues a fresh request rather than relying on the stale one.
- **Capability goes down mid-request, after constraints were sent but before a response returns** -- No Weekly Plan or Planned Meal record is created from a partial exchange; the household sees the Capability Down messaging with no half-created plan.
- **A household's candidate recipe pool is very small (e.g., a brand-new household with only starter recipes and one allergy)** -- The capability may return a valid but repetitive proposal; this is accepted as a full week if it passes safety and fills seven nights, since recipe pool variety is Recipe Library's (FEAT-08) concern, not this integration's.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) | Triggered by (inbound) | Initiates a generation request each scheduled cycle |
| FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) | Triggered by (inbound) | Initiates a generation request on upgrade |
| FEAT-03.SPEC-003 / FEAT-03.SPEC-004 | Affects (outbound) | Both automations' Outcome Definitions depend on this integration's response, timeout, or error |
| FEAT-02.SPEC-002 (Candidate Safety Check Execution) | References (outbound) | Every proposal this integration returns is subject to this safety check before use |
| FEAT-03.SPEC-007 (Household-Scaled Quantity Rule) | References (outbound) | Runs on the safety-passed subset of this integration's proposal |
| FEAT-03.SPEC-006 (Budget Fit & Estimated Total Rule) | References (outbound) | Runs on the scaled subset of this integration's proposal |
| FEAT-04.SPEC-007 (Swap Alternatives Generation) | References (sibling) | Covers the same capability category for single-slot swap alternatives; this spec covers full-week generation only |
| FEAT-05.SPEC-006 (Pantry-to-Recipe Matching for Plan Callout) | References (inbound) | Supplies the pantry weighting input this integration sends |
| FEAT-12.SPEC-004 (Preference Weighting & Tier-Gating Rule) | References (inbound) | Supplies the aggregated rating weighting input this integration sends |

## Analytics and Success Signals

- **ai_generation_requested** (household id, candidate pool size) -- N/A -- no Stage 2 metric measures request volume directly; retained to observe generation demand and capability usage operationally
- **ai_generation_response_received** (outcome: full_week / partial / timeout / error, latency) -- supports success-metrics.md: "Weekly Planning Time" (latency here directly composes the under-10-minutes-a-week goal)
- **ai_generation_degradation_shown** (condition: slow / down / rejected; screen: FEAT-03.SPEC-001) -- N/A -- no Stage 2 metric measures degradation frequency directly; retained so the product's tolerance for capability trouble is observable

## Acceptance Criteria

**FEAT-03.SPEC-010-AC-01:** Given a household's generation is triggered, when this integration sends its request, then it includes weekly_budget, weekly_schedule, member count, active dietary rules, the candidate recipe pool, current pantry items, and aggregated ratings -- and nothing else.

**FEAT-03.SPEC-010-AC-02:** Given the capability returns seven or more safety-passable candidates, when the response arrives, then the receiving automation proceeds to safety filtering and selection.

**FEAT-03.SPEC-010-AC-03:** Given the capability returns fewer usable candidates than needed to fill seven nights after safety filtering, when the response arrives, then the receiving automation treats this as a generation failure.

**FEAT-03.SPEC-010-AC-04:** Given the capability does not respond within the product's expected generation window, when the timeout is reached, then the household sees FEAT-03.SPEC-001's Error state with Retry, and the previous week's plan remains visible.

**FEAT-03.SPEC-010-AC-05:** Given the capability is slow but within the expected window, when the household is on FEAT-03.SPEC-001, then it continues to show the explained-wait Loading state without an error.

**FEAT-03.SPEC-010-AC-06:** Given the capability rejects a generation request, when the rejection is received, then the household experiences the same Retry-offering messaging as a Capability Down condition, with the previous plan unchanged.

**FEAT-03.SPEC-010-AC-07:** Given a household generates its plan for the first time, when generation is about to run, then the household is shown the generation data-sharing notice naming exactly what is shared and what never leaves the product.

**FEAT-03.SPEC-010-AC-08:** Given Maya wants to review what data her plan generation shares, when she taps "How your plan is built" on FEAT-03.SPEC-001, then the same disclosure notice reopens.

**FEAT-03.SPEC-010-AC-09:** Given the capability's response references a recipe removed from the household's pool after the request was sent, when the automation processes the response, then that candidate is discarded before selection.

**FEAT-03.SPEC-010-AC-10:** Given the capability goes down after receiving a request but before returning a response, when the household checks its plan, then no partial Weekly Plan or Planned Meal record exists, and the Capability Down messaging is shown.

**FEAT-03.SPEC-010-AC-11:** Given a late response arrives for a request the household has already been shown as failed, when it is received, then it is discarded and does not silently create a plan behind the household's back.

**FEAT-03.SPEC-010-AC-12:** Given individual members' ratings exist for the household, when this integration sends rating data, then only aggregated per-recipe values are sent, never a rating attributed to a specific member.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 3 | 3 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 3 (FEAT-03.SPEC-001 x 3 conditions; FEAT-03.SPEC-002 N/A cells excluded) | 3 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |
