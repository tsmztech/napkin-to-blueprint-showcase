---
document_type: spec
spec_type: automation
spec_id: FEAT-03.SPEC-003
spec_name: Scheduled Weekly Plan Generation
spec_slug: scheduled-weekly-plan-generation
parent_feature: FEAT-03
parent_feature_name: AI Weekly Dinner Plan Generation
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Automation Spec: Scheduled Weekly Plan Generation

## Overview

**Name:** Scheduled Weekly Plan Generation
**ID:** FEAT-03.SPEC-003
**Type:** Automation
**Purpose:** System generates the week's seven-dinner plan on the household's chosen schedule, drawing on safety filtering, pantry weighting, rating history, budget, and schedule.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation

## Scope and Non-Goals

**In Scope:**
- Firing on each household's chosen plan-arrival day and time to generate the next week's seven Planned Meals
- Checking generation eligibility before running
- Requesting a proposal from the AI plan-generation capability, scaling quantities, and computing budget fit
- Archiving the previous week's Active plan when the new week generates
- The generation-failure path that keeps the previous week visible

**Non-Goals:**
- Determining whether a household is eligible to generate at all -- owned by FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule); this automation calls that rule rather than re-implementing its checks
- The very first plan a household ever receives after upgrading -- owned by FEAT-03.SPEC-004 (First-Plan Generation on Upgrade), a distinct event-triggered path
- Sending the "plan ready" notification -- owned by Weekly Plan Ready Notification (FEAT-07), triggered by this automation's completion but specified separately per the Side-Effect Inventory
- Recalculating the grocery list -- owned by Shared Grocery List (FEAT-06), triggered by this automation's completion but specified separately

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household's chosen plan-arrival day and time arrives | Household.plan_arrival_day_time (system schedule, set via FEAT-07) | Fires once per household per week, at the day/time the organiser configured (Sunday evening by default); does not fire for a household whose eligibility check (FEAT-03.SPEC-009) fails | Household constraints (weekly_budget, weekly_schedule, member count), all Member Profiles' Dietary Rules, current Pantry Items, Rating history, the Recipe candidate pool (starter + imported), the prior week's Weekly Plan (for archival) |

## Processing Logic

1. At the household's configured plan-arrival day/time, check generation eligibility and tier via FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule).
2. If eligibility fails, do not generate; the outcome is handled entirely within FEAT-03.SPEC-009 (its own outcome table governs what the household sees).
3. If eligible, assemble the household's constraint set: every active member's Dietary Rules (hard allergies and religious rules, soft dislikes and learned dislikes), the weekly_schedule (time-constrained nights), the weekly_budget, and the household's member count.
4. Read the current Recipe candidate pool (starter library plus the household's imported recipes) and current Pantry Items.
5. Read Rating history for the household to weight candidate selection toward highly-rated meals and away from repeatedly down-rated ones (learned dislikes are supplied by FEAT-12 as Dietary Rule entries and are already reflected in step 3).
6. Compose this data into a request to the AI plan-generation capability via FEAT-03.SPEC-010 (AI Plan Generation Capability Integration) and receive a proposed set of candidate dinners.
7. Pass every candidate dinner through the app-enforced allergy and religious-rule safety check (owned by FEAT-02); exclude any candidate that fails or whose ingredient data is incomplete, per XBR-01.
8. From the safety-passed candidates, select exactly seven dinners (one per night), weighting toward dinners that use logged Pantry Items and toward highly-rated meals, and fitting each time-constrained night's cook time within its stated limit.
9. Scale each selected dinner's ingredient quantities to the household's size via FEAT-03.SPEC-007 (Household-Scaled Quantity Rule).
10. Compute the week's estimated cost against the household's weekly_budget via FEAT-03.SPEC-006 (Budget Fit & Estimated Total Rule); if no safe week fits the budget, select the closest-fitting safe combination and attach an over-budget note.
11. Create a new Weekly Plan (status: Generated, origin: AI-generated) and seven new Planned Meals (status: Proposed), each carrying its recipe, safety badge, cook time, rough cost, and pantry callout.
12. If a prior week's Weekly Plan exists in Active status, transition it to Archived.
13. Signal completion to trigger the "plan ready" notification (FEAT-07.SPEC-001, Plan-Ready Notification Trigger) and grocery list recalculation (FEAT-06.SPEC-002, Grocery List Generation & Recalculation).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Generation succeeded | Steps 1-13 complete with a full seven-dinner plan fitting the budget | New Weekly Plan (Generated) and seven Planned Meals (Proposed) created; prior Active plan archived | FEAT-03.SPEC-001 shows the new week; FEAT-07.SPEC-001 sends the plan-ready notification; FEAT-06.SPEC-002 recalculates the grocery list | FEAT-03.SPEC-001, FEAT-07.SPEC-001, FEAT-06.SPEC-002, FEAT-11.SPEC-002, FEAT-21.SPEC-003 |
| Generation succeeded, over budget | Steps 1-13 complete, but no safe combination of seven dinners fits weekly_budget | Same as above, plus Weekly Plan.over_budget_note set by FEAT-03.SPEC-006 | FEAT-03.SPEC-001 shows the plan with the over-budget note in the weekly total banner | FEAT-03.SPEC-001, FEAT-03.SPEC-006, FEAT-11.SPEC-002, FEAT-21.SPEC-003 |
| Not eligible (no generation) | FEAT-03.SPEC-009's eligibility check fails (missing dietary data, no schedule, free tier) | No Weekly Plan created | Handled entirely by FEAT-03.SPEC-009's own outcome definitions (this automation defers to it) | FEAT-03.SPEC-009 |
| Generation failure | The AI plan-generation capability cannot return a usable proposal, or the safety-passed candidate pool cannot fill all seven nights | No new Weekly Plan created; the prior week's plan is not archived | The previous week's plan remains visible on FEAT-03.SPEC-001 with an Error state banner and Retry control; no member is ever left with no plan at all | FEAT-03.SPEC-001, FEAT-03.SPEC-010 |
| Retry succeeds | Household or system retries generation after a failure | Same as "Generation succeeded" | FEAT-03.SPEC-001 clears the Error state and shows the new week | FEAT-03.SPEC-001 |

## Data Model

**Reads:** Household -- weekly_budget, weekly_schedule, plan_arrival_day_time, member count. Member Profile -- for household size and eligibility context. Dietary Rule -- all active rules per member. Recipe -- candidate pool with ingredients, cook_time, rough_cost. Pantry Item -- Active items for weighting. Rating -- history for weighting. Weekly Plan -- the prior week's plan, for archival.
**Creates:** Weekly Plan -- week, origin (AI-generated), status (Generated), estimated_total, over_budget_note (when applicable). Planned Meal -- seven records: night, meal_kind (dinner), recipe, safety_badge, vegetarian_option, cook_time, rough_cost, pantry_callout, status (Proposed).
**Updates:** Weekly Plan (prior week) -- status transitions from Active to Archived.
**Deletes:** None.

## Business Rules

- XBR-01: Every candidate dinner passes the app-enforced allergy and religious-rule check before it can be selected; a recipe with incomplete ingredient data is excluded, never shown unchecked.
- XBR-03: Generation completion always triggers a grocery list recalculation (FEAT-06.SPEC-002) in the same cycle -- no household ever sees a week's plan without its matching list.
- XBR-07: The plan a household receives here starts in Generated status and requires the organiser's approval (FEAT-03.SPEC-008) or auto-adoption (FEAT-03.SPEC-005) before the week begins.
- XBR-12: This automation's completion triggers at most one "plan ready" message per household per week (FEAT-07.SPEC-001); the plan's in-app availability (this automation's own effect) never depends on that notification's delivery.
- XBR-17: Learned soft dislikes (from repeated down-ratings, FEAT-12.SPEC-005 Repeated-Dislike Learned Update) influence selection weighting but never block a suggestion and never override an explicit hard rule.
- Generation always proposes exactly seven dinners, one per day, regardless of household size (product-features.md, Validation & Limits).
- This automation calls FEAT-03.SPEC-009 for eligibility, FEAT-03.SPEC-006 for budget fit, FEAT-03.SPEC-007 for quantity scaling, and FEAT-03.SPEC-010 for the AI proposal itself -- it does not duplicate any of those rules.

## Edge Cases

- **A brand-new household with no pantry items logged** -- Generation proceeds normally; pantry-awareness simply has nothing to weight toward that week, and the plan is still complete and safe.
- **Fewer than seven safety-passed candidates exist for a household's rules** -- Treated as a generation failure; the previous week's plan remains visible with the Error state and Retry, since a plan that cannot honor every hard rule for all seven nights is never partially generated or filled with an unsafe placeholder.
- **A household member's dietary data changes mid-generation (race with a Household Setup edit)** -- The generation run uses the constraint set read at step 3; if a hard rule tightens after that read but before completion, the newly created plan is immediately re-checked by the mid-week rule-change process (FEAT-01/FEAT-02, XBR-02) the moment it becomes Active, exactly as any other existing plan would be.
- **The AI plan-generation capability returns candidates that all fail the safety check** -- Treated as a generation failure per the Outcome Definitions; the safety check (FEAT-02) is never bypassed to fill a night.
- **Concurrent trigger firing (two schedule fires for the same household at effectively the same time -- e.g., a manual admin retry overlapping the scheduled fire)** -- Only one generation run may be in progress per household at a time; a second trigger for the same household while a run is in flight is treated as the "trigger fires while a previous run is in flight" case below rather than starting a parallel run.
- **Trigger fires while a previous run is in flight** -- The new trigger is deferred until the in-flight run completes (success or failure); if the in-flight run succeeds, the deferred trigger is discarded as redundant for that week; if the in-flight run fails, the deferred trigger proceeds as a retry attempt.
- **The household's plan-arrival time changes (FEAT-07.SPEC-004, Plan-Arrival Day & Time Setting Rule) after this week's generation already ran** -- The change takes effect for the following week's schedule; it does not retroactively re-fire generation for the current week.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule) | Triggers (outbound) | Eligibility and tier gate checked before any generation attempt |
| FEAT-03.SPEC-010 (AI Plan Generation Capability Integration) | Triggers (outbound) | Requests the candidate seven-dinner proposal |
| FEAT-03.SPEC-006 (Budget Fit & Estimated Total Rule) | Triggers (outbound) | Computes the week's estimated total and any over-budget note |
| FEAT-03.SPEC-007 (Household-Scaled Quantity Rule) | Triggers (outbound) | Scales each dinner's ingredient quantities to the household |
| FEAT-03.SPEC-001 (Weekly Plan View) | Affects (outbound) | Displays the generated plan, including the Error/Retry state on failure |
| FEAT-07.SPEC-001 (Plan-Ready Notification Trigger) | Triggers (outbound) | Completion fires the plan-ready message |
| FEAT-06.SPEC-002 (Grocery List Generation & Recalculation) | Triggers (outbound) | Completion triggers grocery list recalculation |
| FEAT-05.SPEC-006 (Pantry-to-Recipe Matching for Plan Callout) | References (inbound) | Supplies logged Pantry Items and the matching logic behind step 4's pantry callout |
| FEAT-12.SPEC-005 (Repeated-Dislike Learned Update) | References (inbound) | Supplies Rating history and learned dislikes read at steps 3 and 5 |
| FEAT-09.SPEC-009 (Organiser Hand-Over Processing) | References (inbound) | An organiser hand-over changes who approves the plan this automation produces, without affecting generation itself |

## Analytics and Success Signals

- **plan_generation_started** (household id, scheduled vs. retry) -- N/A -- no Stage 2 metric measures generation start events directly; retained to observe generation volume and retry rate operationally
- **plan_generation_completed** (outcome: on_budget / over_budget, pantry items used count, duration) -- supports success-metrics.md: "Weekly Planning Time"
- **plan_generation_failed** (reason: insufficient_candidates / capability_unavailable) -- supports success-metrics.md: "Weekly Planning Time" (a failure that delays a usable plan works directly against the under-10-minutes goal)
- **plan_weeknight_time_fit** (night, cook_time, within_limit: yes/no) -- supports success-metrics.md: "Weeknight Time-Fit Accuracy"
- **plan_used_pantry_item** (count of pantry items used) -- N/A -- no metric in this feature's connected-metric slice measures pantry usage; success-metrics.md's "Pantry Items Used" metric (Connected Feature: Pantry-Aware Suggestions) is fed within FEAT-05, not here

## Acceptance Criteria

**FEAT-03.SPEC-003-AC-01:** Given Maya's household reaches its configured Sunday-evening plan-arrival time and meets every eligibility prerequisite, when generation fires, then a new Weekly Plan with seven Planned Meals is created, each passing the allergy safety check.

**FEAT-03.SPEC-003-AC-02:** Given a household has one member with a peanut allergy, when generation runs, then every generated dinner excludes recipes containing peanuts, per XBR-01.

**FEAT-03.SPEC-003-AC-03:** Given a household has logged spinach and feta as pantry items, when generation runs and a candidate dinner uses both, then that dinner is weighted into the plan and its Planned Meal carries a pantry callout naming them.

**FEAT-03.SPEC-003-AC-04:** Given a household's weekly_schedule marks Tuesday as a 30-minute night, when generation selects Tuesday's dinner, then its cook_time is at or under that limit.

**FEAT-03.SPEC-003-AC-05:** Given no safe combination of seven dinners fits the household's weekly_budget, when generation completes, then the closest-fitting safe plan is created with an over-budget note attached via FEAT-03.SPEC-006.

**FEAT-03.SPEC-003-AC-06:** Given a household fails FEAT-03.SPEC-009's eligibility check (e.g., no schedule set), when the plan-arrival time arrives, then no new Weekly Plan is created and the outcome is handled by FEAT-03.SPEC-009.

**FEAT-03.SPEC-003-AC-07:** Given the AI plan-generation capability cannot return a usable proposal, when generation runs, then the previous week's plan remains visible on FEAT-03.SPEC-001 with an Error state and Retry control, and no new plan is created.

**FEAT-03.SPEC-003-AC-08:** Given a household retries generation after a failure and the retry succeeds, when the retry completes, then FEAT-03.SPEC-001 shows the new week and the Error state clears.

**FEAT-03.SPEC-003-AC-09:** Given generation succeeds for a household with a prior Active Weekly Plan, when the new plan is created, then the prior week's plan transitions to Archived.

**FEAT-03.SPEC-003-AC-10:** Given generation succeeds, when completion is signaled, then FEAT-07 sends the plan-ready notification and FEAT-06 recalculates the grocery list in the same cycle.

**FEAT-03.SPEC-003-AC-11:** Given a household has rated three past meals down repeatedly, when generation runs, then those meals are weighted away from selection per the learned dislikes supplied by FEAT-12, without ever appearing as a blocked "error."

**FEAT-03.SPEC-003-AC-12:** Given fewer than seven safety-passed candidates exist for a household's combined dietary rules, when generation runs, then the run is treated as a failure and the previous week's plan remains visible rather than a partially filled or unsafe week being created.

**FEAT-03.SPEC-003-AC-13:** Given a second scheduled trigger fires for a household while its prior week's generation run is still in flight, when the second trigger arrives, then it is deferred until the in-flight run completes, and no parallel run for that household starts.

**FEAT-03.SPEC-003-AC-14:** Given two households' plan-arrival times occur at effectively the same moment, when both trigger, then each household's generation runs independently against its own data and neither affects the other's outcome.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 (scheduled plan-arrival) | 1 |
| Outcome Paths | 5 (success, success over-budget, not eligible, failure, retry succeeds) | 5 |
| Business Rules | 7 | 7 |
| Edge Cases | 7 | 7 |
