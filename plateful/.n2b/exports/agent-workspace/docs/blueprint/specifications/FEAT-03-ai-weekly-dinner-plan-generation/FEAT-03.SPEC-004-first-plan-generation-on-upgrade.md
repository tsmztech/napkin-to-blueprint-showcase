---
document_type: spec
spec_type: automation
spec_id: FEAT-03.SPEC-004
spec_name: First-Plan Generation on Upgrade
spec_slug: first-plan-generation-on-upgrade
parent_feature: FEAT-03
parent_feature_name: AI Weekly Dinner Plan Generation
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: First-Plan Generation on Upgrade

## Overview

**Name:** First-Plan Generation on Upgrade
**ID:** FEAT-03.SPEC-004
**Type:** Automation
**Purpose:** System generates a household's very first AI plan immediately after it upgrades to the paid tier.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation

## Scope and Non-Goals

**In Scope:**
- Firing once, immediately, when a household's subscription transitions from free to paid
- Running the same eligibility check, safety filtering, pantry weighting, budget fit, and scaling logic as the weekly schedule, for this one out-of-cycle run
- Producing the household's first Weekly Plan and seven Planned Meals so the "your first plan is on its way" empty state resolves promptly

**Non-Goals:**
- The recurring weekly generation cycle -- owned by FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation), which continues on the household's normal schedule from the following cycle onward
- Processing the upgrade itself (payment, plan selection) -- owned by Subscription & Billing Management (FEAT-14); this automation only reacts to the upgrade's completion
- Re-running for a household that upgrades, downgrades, and upgrades again within the same week -- see Edge Cases for the exact re-trigger behavior, which stays within this spec's scope but does not duplicate FEAT-03.SPEC-003's recurring cadence

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household upgrades from free to paid tier | FEAT-14.SPEC-008 (Apply Subscription Change) | Fires once, immediately, when Subscription.tier transitions to paid and Subscription.billing_state becomes Active; does not fire again for a household that later renews, switches billing period, or re-upgrades after a downgrade within the same billing cycle (see Edge Cases) | Household constraints (weekly_budget, weekly_schedule, member count), all Member Profiles' Dietary Rules, current Pantry Items, Rating history (if any exists from prior free-tier use), the Recipe candidate pool |

## Processing Logic

1. On the upgrade-confirmed event from FEAT-14.SPEC-008 (Apply Subscription Change), check generation eligibility via FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule) -- the tier-gating half now passes by definition, but the non-tier prerequisites (complete dietary data, a schedule set) are still checked.
2. If eligibility fails on a non-tier prerequisite, do not generate; the outcome is handled by FEAT-03.SPEC-009 (the household sees its incomplete-setup messaging, not the free-tier placeholder, since it is now paid).
3. If eligible, assemble the household's constraint set exactly as FEAT-03.SPEC-003 does at its step 3: all active Dietary Rules, weekly_schedule, weekly_budget, and member count.
4. Read the current Recipe candidate pool and current Pantry Items (a household may have logged pantry items on the free tier already, since pantry logging is free per XBR-04).
5. Read any Rating history the household accumulated on the free tier via Manual Weekly Planning (FEAT-23.SPEC-001) ratings, if present.
6. Compose the request to the AI plan-generation capability via FEAT-03.SPEC-010 and receive proposed candidates.
7. Pass every candidate through the safety check (FEAT-02.SPEC-002, Candidate Safety Check Execution); exclude failures per XBR-01.
8. Select seven dinners, weighting toward pantry items and any existing ratings, fitting time-constrained nights.
9. Scale quantities via FEAT-03.SPEC-007.
10. Compute budget fit via FEAT-03.SPEC-006, attaching an over-budget note if no safe week fits.
11. Create the household's first Weekly Plan (status: Generated, origin: AI-generated) and seven Planned Meals (status: Proposed). There is no prior Active AI-generated plan to archive; any existing free-tier manually built plan for the current week remains a separate record owned by FEAT-23.SPEC-001 and is not modified by this automation (see Edge Cases).
12. Signal completion to trigger FEAT-07.SPEC-001 (plan-ready notification, using the "first plan" framing) and FEAT-06.SPEC-002 (grocery list recalculation).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| First-plan generation succeeded | Steps 1-12 complete with a full seven-dinner plan fitting the budget | First Weekly Plan (Generated) and seven Planned Meals (Proposed) created | FEAT-03.SPEC-001 resolves from its Empty state to a populated plan; FEAT-07.SPEC-001 sends the plan-ready notification with first-plan framing; FEAT-06.SPEC-002 builds the grocery list | FEAT-03.SPEC-001, FEAT-07.SPEC-001, FEAT-06.SPEC-002, FEAT-11.SPEC-002 |
| First-plan generation succeeded, over budget | Steps 1-12 complete, but no safe week fits weekly_budget | Same as above, plus over_budget_note set | FEAT-03.SPEC-001 shows the plan with the over-budget note | FEAT-03.SPEC-001, FEAT-03.SPEC-006, FEAT-11.SPEC-002 |
| Not eligible (non-tier prerequisite unmet) | FEAT-03.SPEC-009's non-tier eligibility check fails | No Weekly Plan created | Handled by FEAT-03.SPEC-009's own outcome definitions; Maya sees what setup is still missing, not the free-tier placeholder | FEAT-03.SPEC-009 |
| Generation failure | The AI plan-generation capability cannot return a usable proposal, or fewer than seven safety-passed candidates exist | No Weekly Plan created | FEAT-03.SPEC-001's Empty state persists with a note that the first plan is taking longer than expected, plus a Retry option | FEAT-03.SPEC-001 |
| Retry succeeds | Maya or the system retries after a failure | Same as "First-plan generation succeeded" | FEAT-03.SPEC-001 resolves to the populated plan | FEAT-03.SPEC-001 |

## Data Model

**Reads:** Subscription -- tier, billing_state (to detect the free-to-paid transition). Household -- weekly_budget, weekly_schedule, member count. Member Profile -- household size context. Dietary Rule -- all active rules. Recipe -- candidate pool. Pantry Item -- Active items. Rating -- any existing free-tier history.
**Creates:** Weekly Plan -- week, origin (AI-generated), status (Generated), estimated_total, over_budget_note (when applicable). Planned Meal -- seven records with the same fields as FEAT-03.SPEC-003 creates.
**Updates:** None -- no prior AI-generated Weekly Plan exists to archive on a household's very first run.
**Deletes:** None.

## Business Rules

- XBR-01: Every candidate dinner passes the app-enforced safety check before selection, identically to the scheduled cycle.
- XBR-05: This automation only fires because the household is now paid; a household that upgrades and immediately downgrades before this run completes is handled per Edge Cases, never left mid-generation on a tier it no longer holds.
- XBR-12: Completion triggers at most one plan-ready message, using first-plan framing distinct from the recurring weekly message (FEAT-07.SPEC-002, Plan-Ready Notification Message, owns the exact wording distinction).
- This automation fires exactly once per upgrade event; it does not replace or pre-empt the household's subsequent regular weekly cycle owned by FEAT-03.SPEC-003, which continues on the household's configured plan-arrival schedule from the following cycle.
- Eligibility, safety filtering, budget fit, and scaling logic are identical to FEAT-03.SPEC-003's -- this spec reuses those rules rather than defining separate ones, so a household's first plan and its subsequent plans behave consistently.

## Edge Cases

- **Household has an existing free-tier manual plan for the current week when it upgrades** -- The manually built Weekly Plan (owned by FEAT-23.SPEC-001) is left untouched; this automation creates a separate, new AI-generated Weekly Plan. FEAT-03.SPEC-001 (now reachable, since the household is paid) shows the new AI-generated plan; the household's prior manual plan remains part of its history (FEAT-19.SPEC-001) and is not merged or overwritten.
- **Household upgrades, then downgrades, then upgrades again within a short window** -- Each free-to-paid transition fires this automation independently; a downgrade that occurs before this automation completes does not cancel an in-flight run (the run still completes, since the household did pay for that period), but no further first-plan run fires for the same household until another genuine free-to-paid transition occurs.
- **Household has already accumulated ratings on the free tier before upgrading** -- Those ratings are read and weighted at step 5, exactly as later cycles would; the household's very first AI plan is not treated as a cold start if preference data already exists.
- **This automation and a scheduled cycle (FEAT-03.SPEC-003) become due for the same household at effectively the same time** -- Only one generation run is in flight per household at a time (the same rule FEAT-03.SPEC-003 applies); if the upgrade-triggered run is already in progress when the schedule would otherwise fire, the schedule's trigger is deferred and, if it becomes redundant once the first-plan run completes for the current week, it is discarded.
- **The AI plan-generation capability is unavailable at the moment of upgrade** -- The Empty state on FEAT-03.SPEC-001 persists with a "taking longer than expected" note and Retry, rather than showing a hard error immediately after a household has just paid.
- **Household upgrades but has incomplete dietary data** -- Generation does not run; FEAT-03.SPEC-009 surfaces what is missing on FEAT-03.SPEC-001, and the household is guided to complete it rather than silently waiting for a plan that will never generate.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggered by (inbound) | Upgrade confirmation fires this automation |
| FEAT-03.SPEC-009 (Generation Eligibility & Tier-Gating Rule) | Triggers (outbound) | Non-tier eligibility checked before generation |
| FEAT-03.SPEC-010 (AI Plan Generation Capability Integration) | Triggers (outbound) | Requests the candidate seven-dinner proposal |
| FEAT-03.SPEC-006 (Budget Fit & Estimated Total Rule) | Triggers (outbound) | Computes the week's estimated total |
| FEAT-03.SPEC-007 (Household-Scaled Quantity Rule) | Triggers (outbound) | Scales ingredient quantities |
| FEAT-03.SPEC-001 (Weekly Plan View) | Affects (outbound) | Resolves the Empty state to a populated plan, or shows the retry note on failure |
| FEAT-07.SPEC-001 (Plan-Ready Notification Trigger) | Triggers (outbound) | Completion fires the first-plan-ready message |
| FEAT-06.SPEC-002 (Grocery List Generation & Recalculation) | Triggers (outbound) | Completion triggers grocery list creation |
| FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)) | References (inbound) | Any existing free-tier manual plan for the current week is left untouched |

## Analytics and Success Signals

- **first_plan_generation_started** (household id) -- N/A -- no Stage 2 metric measures generation start events directly; retained to observe first-plan latency operationally
- **first_plan_generation_completed** (outcome: on_budget / over_budget, duration) -- supports success-metrics.md: "Weekly Planning Time"
- **first_plan_generation_failed** (reason) -- supports success-metrics.md: "Weekly Planning Time"
- **first_plan_weeknight_time_fit** (night, cook_time, within_limit: yes/no) -- supports success-metrics.md: "Weeknight Time-Fit Accuracy"

## Acceptance Criteria

**FEAT-03.SPEC-004-AC-01:** Given Maya's household completes its upgrade to paid and meets every non-tier eligibility prerequisite, when the upgrade is confirmed, then this automation fires immediately and creates the household's first Weekly Plan with seven safety-passed Planned Meals.

**FEAT-03.SPEC-004-AC-02:** Given Maya's household upgrades but has an incomplete member's dietary data, when the upgrade is confirmed, then no plan is generated and FEAT-03.SPEC-001 shows what is still missing, per FEAT-03.SPEC-009.

**FEAT-03.SPEC-004-AC-03:** Given the household's first-plan generation succeeds, when completion is signaled, then FEAT-07 sends a plan-ready message using first-plan framing and FEAT-06 builds the grocery list.

**FEAT-03.SPEC-004-AC-04:** Given no safe combination of seven dinners fits the household's weekly_budget on its first generation, when generation completes, then the closest-fitting safe plan is created with an over-budget note.

**FEAT-03.SPEC-004-AC-05:** Given the AI plan-generation capability is unavailable at the moment of upgrade, when generation is attempted, then FEAT-03.SPEC-001's Empty state shows a "taking longer than expected" note with Retry, rather than a hard failure.

**FEAT-03.SPEC-004-AC-06:** Given a household has an existing free-tier manual plan for the current week when it upgrades, when this automation completes, then the manual plan remains unchanged and a separate AI-generated Weekly Plan is created and displayed.

**FEAT-03.SPEC-004-AC-07:** Given a household accumulated ratings while on the free tier, when its first AI plan generates, then those ratings weight the selection exactly as they would in a later scheduled cycle.

**FEAT-03.SPEC-004-AC-08:** Given a household's first-plan generation is already in flight when its regular scheduled generation would also become due, when the schedule trigger arrives, then it is deferred and discarded once the first-plan run completes for that week, rather than starting a second parallel run.

**FEAT-03.SPEC-004-AC-09:** Given a household upgrades, downgrades, and upgrades again, when each free-to-paid transition completes, then this automation fires independently for each genuine transition, without duplicating a run already completed for the same period.

**FEAT-03.SPEC-004-AC-10:** Given Maya retries generation after a first-plan failure and the retry succeeds, when the retry completes, then FEAT-03.SPEC-001 shows the populated plan and the retry note clears.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 (upgrade confirmed) | 1 |
| Outcome Paths | 5 (success, success over-budget, not eligible, failure, retry succeeds) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
