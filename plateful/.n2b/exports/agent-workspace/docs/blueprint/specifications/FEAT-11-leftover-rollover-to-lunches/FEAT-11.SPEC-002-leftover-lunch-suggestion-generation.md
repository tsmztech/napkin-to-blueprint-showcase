---
document_type: spec
spec_type: automation
spec_id: FEAT-11.SPEC-002
spec_name: Leftover Lunch Suggestion Generation
spec_slug: leftover-lunch-suggestion-generation
parent_feature: FEAT-11
parent_feature_name: Leftover Rollover to Lunches
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Automation Spec: Leftover Lunch Suggestion Generation

## Overview

**Name:** Leftover Lunch Suggestion Generation
**ID:** FEAT-11.SPEC-002
**Type:** Automation
**Purpose:** As part of AI weekly plan generation, creates a Suggested leftover-lunch Planned Meal for each dinner the eligibility rule flags as leftover-producing.
**Parent Feature:** FEAT-11 -- Leftover Rollover to Lunches

## Scope and Non-Goals

**In Scope:**
- Evaluating every dinner in a newly generated Weekly Plan against FEAT-11.SPEC-003's eligibility rule
- Resolving each eligible dinner's following day via FEAT-11.SPEC-003 and creating the linked Suggested leftover-lunch Planned Meal
- Handling the case where more than one eligible dinner in the same week would otherwise collide on the same following day

**Non-Goals:**
- Determining what makes a dinner leftover-producing, or which following day to use -- owned by FEAT-11.SPEC-003 (Leftover Lunch Eligibility & Linking Rule); this automation calls that rule rather than re-deriving it
- Re-evaluating a leftover lunch after its source dinner changes post-generation -- owned by FEAT-11.SPEC-004 (Leftover Lunch Withdrawal on Source Change), a distinct event-triggered path
- Generating leftover-lunch suggestions from Manual Weekly Planning (FEAT-23) activity -- this automation's only trigger is a completed FEAT-03 generation run (Trigger Definition above); product-features.md's FEAT-11 entry names AI Weekly Dinner Plan Generation (FEAT-03), not FEAT-23, as the dependency this feature extends. This holds whether FEAT-23 is building a free-tier week from scratch (which never carries a leftover-lunch suggestion, since no FEAT-03 run ever produced its dinners) or hand-editing one night within an already AI-generated plan (paid tier) -- a night changed that way keeps whatever FEAT-11.SPEC-004 decides for its existing leftover-lunch link, if any; this automation itself never re-runs against it, since it only fires once, at the moment a generation run completes
- Confirming or skipping a leftover lunch once created -- owned by FEAT-11.SPEC-001 (Leftover Lunch Card)

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Scheduled weekly plan generation completes | FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) | Fires once the new Weekly Plan and its seven dinner Planned Meals are created (Generation succeeded or Generation succeeded, over budget outcome) | The newly created Weekly Plan and its seven dinner Planned Meals (night, recipe, status) |
| First-plan generation on upgrade completes | FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) | Fires once the household's first AI-generated Weekly Plan and its seven dinner Planned Meals are created | Same as above |

## Processing Logic

1. Receive the newly created Weekly Plan and its seven dinner Planned Meals from the completed generation run.
2. In night order (Monday through Sunday), evaluate each dinner Planned Meal against FEAT-11.SPEC-003's eligibility determination.
3. For each dinner classified leftover-producing, request the following day from FEAT-11.SPEC-003's following-day computation (default: one day after the source dinner's night).
4. If the computed following day already holds another Suggested, Eaten, or Skipped leftover lunch created earlier in this same run, request the two-day fallback day from FEAT-11.SPEC-003 instead.
5. If both the one-day and two-day following days are already occupied by another leftover lunch from this run, do not create a suggestion for this dinner this week (the two-day ceiling in XBR-10 is never exceeded to find a free day).
6. Otherwise, create a new Planned Meal (meal_kind: leftover lunch, status: Suggested) linked to the source dinner, placed on the resolved following day.
7. Repeat steps 2-6 for every dinner in the week.
8. Signal completion so the Weekly Plan View (FEAT-03.SPEC-001) and Leftover Lunch Card (FEAT-11.SPEC-001) reflect every newly created suggestion.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Suggestions created | One or more dinners in the week are classified leftover-producing and a following day is available for each | One new Suggested leftover-lunch Planned Meal per eligible, successfully placed dinner | Each created suggestion appears as a Leftover Lunch Card on its following day | FEAT-11.SPEC-001, FEAT-03.SPEC-001 |
| No eligible dinners this week | No dinner in the week is classified leftover-producing | None | No leftover-lunch cards appear anywhere in the week; this is the normal case, not an error (product-features.md, States) | FEAT-03.SPEC-001 |
| Eligible dinner skipped for day collision | An eligible dinner's one-day and two-day following days are both already occupied by another leftover lunch from this run | No leftover-lunch record created for that dinner | No card appears for that dinner this week; no error is shown anywhere | FEAT-11.SPEC-001 |
| Per-dinner computation failure | The eligibility or day-placement computation fails for one specific dinner | No leftover-lunch record created for that dinner; all other dinners in the run are unaffected | No card appears for that dinner; no error is shown to the household, consistent with product-features.md's "a failure to compute it simply omits the suggestion rather than showing an error" | FEAT-11.SPEC-001 |

## Data Model

**Reads:** Planned Meal (source dinner) -- night, recipe, status, for each of the week's seven dinners. Weekly Plan -- week, to scope the run to the newly generated plan.
**Creates:** Planned Meal (leftover-lunch sub-type) -- meal_kind (leftover lunch), linked source dinner, night (the resolved following day), status (Suggested). One record per eligible, successfully placed dinner.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-10: every leftover-lunch record this automation creates links to exactly one source dinner and a following day no more than two days later.
- Eligibility and day-placement logic are never re-derived here -- both are delegated to FEAT-11.SPEC-003 on every evaluation.
- At most one leftover-lunch Planned Meal is created per leftover-eligible dinner per week per household (feature-dependency-map.md, Non-Functional Notes).
- This automation runs only as part of AI Weekly Dinner Plan Generation (FEAT-03); Manual Weekly Planning (FEAT-23) dinners are never evaluated by it, per the Brief's declared dependency on FEAT-03 alone.
- A per-dinner computation failure never blocks or fails the parent plan generation run -- the affected dinner simply receives no leftover-lunch suggestion.

## Edge Cases

- **Two eligible dinners in the same week would both default to the same following day** -- The second dinner processed (in night order) falls back to its two-day following day instead, per FEAT-11.SPEC-003.
- **Three or more eligible dinners collide on the same day window** -- Each is resolved in night order; any dinner for which both its one-day and two-day following days are already taken receives no suggestion this week, per the Outcome Definitions above.
- **A source dinner falls on the last night of the week (e.g., Sunday)** -- The following day may fall in the next calendar week; the leftover lunch is still created and attached to the day it falls on, since the two-day ceiling is measured in elapsed days, not week boundaries.
- **Zero dinners in the week are leftover-producing** -- No leftover-lunch records are created; this is the normal "No eligible dinners this week" outcome, not a failure.
- **Concurrent trigger firing (two generation completions for the same household at effectively the same time)** -- Cannot occur: FEAT-03.SPEC-003 guarantees only one generation run is in flight per household at a time, so this automation is never invoked twice concurrently for the same household's week.
- **Trigger fires while a previous run of this automation is still in flight** -- Cannot occur for the same reason: this automation only ever runs once per completed generation cycle per household, and the next cycle cannot begin until the current one (and everything it triggers) has settled, per FEAT-03.SPEC-003's own concurrency guarantee.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) | Triggered by (inbound) | Completion of scheduled generation fires this automation (bidirectional reference gap: FEAT-03.SPEC-003's own Connected Specs table does not yet name this spec as an outbound trigger -- flagged for cross-reference reconciliation, since FEAT-03.SPEC-003 is owned by a different feature and outside this spec's authority to edit) |
| FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) | Triggered by (inbound) | Completion of the household's first generation fires this automation |
| FEAT-11.SPEC-003 (Leftover Lunch Eligibility & Linking Rule) | References (outbound) | Supplies the eligibility determination and following-day computation this automation calls for every dinner |
| FEAT-11.SPEC-001 (Leftover Lunch Card) | Affects (outbound) | Displays every leftover-lunch suggestion this automation creates |
| FEAT-03.SPEC-001 (Weekly Plan View) | Affects (outbound) | Shows the newly generated week including its leftover-lunch suggestions |

## Analytics and Success Signals

- **leftover_lunch_suggested** (source dinner night, following day, count created this week) -- supports success-metrics.md: "Reported Food Waste and Spend Reduction"
- **leftover_lunch_generation_skipped** (reason: day_collision / per_dinner_computation_failure) -- N/A -- no Stage 2 metric measures this operationally-only signal directly; retained to observe how often an eligible dinner fails to receive a suggestion

## Acceptance Criteria

**FEAT-11.SPEC-002-AC-01:** Given Maya's household's scheduled weekly plan generation (FEAT-03.SPEC-003) completes with one dinner classified leftover-producing, when this automation runs, then a Suggested leftover-lunch Planned Meal is created, linked to that dinner, on the day immediately following it.

**FEAT-11.SPEC-002-AC-02:** Given a household's first-ever AI plan generation (FEAT-03.SPEC-004) completes with an eligible dinner, when this automation runs, then a Suggested leftover-lunch suggestion is created exactly as it would be for a scheduled generation.

**FEAT-11.SPEC-002-AC-03:** Given a generated week contains no dinner classified leftover-producing, when this automation runs, then no leftover-lunch records are created and no card appears anywhere in the week.

**FEAT-11.SPEC-002-AC-04:** Given two eligible dinners in the same week would default to the same following day, when this automation processes the second one, then it is placed on its two-day fallback day instead of colliding with the first.

**FEAT-11.SPEC-002-AC-05:** Given a third eligible dinner's one-day and two-day following days are both already occupied by other leftover lunches from the same run, when this automation processes it, then no leftover-lunch suggestion is created for that dinner and no error is shown.

**FEAT-11.SPEC-002-AC-06:** Given the eligibility computation fails for one specific dinner in an otherwise successful generation run, when this automation completes, then only that dinner has no leftover-lunch suggestion, and every other eligible dinner in the week is unaffected.

**FEAT-11.SPEC-002-AC-07:** Given a household plans its week through Manual Weekly Planning (FEAT-23) instead of AI generation, when those dinners are picked, then this automation never runs against them and no leftover-lunch suggestions are created for that week.

**FEAT-11.SPEC-002-AC-08:** Given a household's generation run is already in flight, when a second trigger for the same household would otherwise arrive, then it cannot occur, since FEAT-03.SPEC-003 guarantees only one generation run per household at a time.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (scheduled generation, first-plan on upgrade) | 2 |
| Outcome Paths | 4 (suggestions created, no eligible dinners, day-collision skip, per-dinner computation failure) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
