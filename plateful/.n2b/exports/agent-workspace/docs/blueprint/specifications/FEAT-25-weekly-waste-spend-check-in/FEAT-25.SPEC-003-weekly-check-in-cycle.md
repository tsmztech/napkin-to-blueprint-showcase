---
document_type: spec
spec_type: automation
spec_id: FEAT-25.SPEC-003
spec_name: Weekly Check-In Cycle
spec_slug: weekly-check-in-cycle
parent_feature: FEAT-25
parent_feature_name: Weekly Waste & Spend Check-In
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Weekly Check-In Cycle

## Overview

**Name:** Weekly Check-In Cycle
**ID:** FEAT-25.SPEC-003
**Type:** Automation
**Purpose:** Opens each week's check-in, marks an unanswered week as skipped, and locks the prior week's answer once the next one opens.
**Parent Feature:** FEAT-25 -- Weekly Waste & Spend Check-In

## Scope and Non-Goals

**In Scope:**
- Creating a new Waste & Spend Check-In record with status Offered for each household at the start of its new planning week
- Marking the previous week's record Skipped if it was never answered
- Locking the previous week's record if it was answered (setting its locked flag, without altering its data)
- Running once per household per week boundary, aligned with the household's own Weekly Plan week

**Non-Goals:**
- Sending a reminder or notification when a week opens or is marked Skipped -- excluded per feature-overview.md's Non-Goals: "a skipped week shows as a gap in the trend, with no follow-up reminder," keeping notifications limited to the weekly plan and nightly nudge
- Validating or accepting the content of any answer -- owned by FEAT-25.SPEC-005 (Check-In Validation & Access Rules); this automation only manages the record's status and lock state, never its waste_amount or spend values
- Setting or changing the household's one-time starting point -- owned by FEAT-25.SPEC-001 (Weekly Check-In Card); this automation never writes starting_point_waste or starting_point_spend
- Cascading or exporting check-in history when a household is deleted -- owned by FEAT-18 (Account & Data Management), per the Feature Breakdown Brief's Side-Effect Inventory

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A new week begins for a household | system (schedule-based) | Fires once per household at the boundary between one calendar week and the next, aligned with the same week boundary the household's Weekly Plan uses (FEAT-03, FEAT-23) | Household id; the household's most recent Waste & Spend Check-In record (if any) and its current status and locked flag |

## Processing Logic

1. At the start of each household's new planning week, check whether a Waste & Spend Check-In record already exists for the new week.
2. If none exists, create a new record for the new week: status Offered, waste_amount and spend empty, locked false.
3. If a record for the new week already exists (e.g., this cycle is re-evaluating after a prior partial run), skip step 2 -- no duplicate record is created.
4. Identify the record for the week immediately before the new week (the household's just-ended week).
5. If that record's status is Offered (no answer was ever submitted for it), set its status to Skipped.
6. If that record's status is Answered, set its locked flag to true; its waste_amount and spend values are left exactly as last submitted.
7. If that record's status is already Skipped, or its locked flag is already true, take no further action on it (idempotent re-run safeguard).
8. If no record exists for the week immediately before the new week (the household's very first week using this feature), skip steps 4-7 entirely -- there is nothing to close.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| New week opened | The new week has no existing record | Waste & Spend Check-In record created with status Offered | Household sees a fresh, unanswered check-in card (or First-Use variant, if no starting point exists) the next time it opens the plan | FEAT-25.SPEC-001 |
| Prior week skipped | Prior week's record status was Offered at cutover | Prior record's status set to Skipped | The trend view shows that week as a gap, with no answer and no notification | FEAT-25.SPEC-002 |
| Prior week locked | Prior week's record status was Answered at cutover | Prior record's locked flag set to true; its data is unchanged | The current card only reflects the new week; the prior week's answer becomes read-only if the household revisits it in the trend view | FEAT-25.SPEC-001, FEAT-25.SPEC-002 |
| No-action (already processed) | The new week's record already exists and the prior week's record is already Skipped or already locked | None | Nothing changes; the cycle's own idempotency safeguard prevents any duplicate effect | -- |
| First-ever week (no prior record) | Household has no record for the week before the new one | Only the create-new-week step runs | Household sees its very first check-in card, the First-Use variant | FEAT-25.SPEC-001 |
| Automation failure | Processing cannot complete for a household (e.g., a transient failure) | No partial state is left: either both the new-week create and the prior-week resolution complete together, or neither does | The household continues to see its previous state (last week's card, if still within its own window) until the cycle successfully retries; the household is never shown two simultaneously open, unlocked weeks nor left with no current week at all | FEAT-25.SPEC-001, FEAT-25.SPEC-002 |

## Data Model

**Reads:** Waste & Spend Check-In -- the household's most recent record and its status and locked flag, to determine what the cutover must do. Household -- to determine the household's own weekly cycle boundary, aligned with its Weekly Plan week.
**Creates:** Waste & Spend Check-In -- one new record per household per week, with status Offered.
**Updates:** Waste & Spend Check-In -- the previous week's record: status set to Skipped, or locked flag set to true.
**Deletes:** None.

## Business Rules

- Exactly one record per household per week is ever created by this cycle; if a record for the new week already exists, no duplicate is created (idempotency).
- The cutover that opens the new week and resolves the previous week happens as a single unit for a given household: the household is never shown two simultaneously open (unlocked, non-final) weeks, and the previous week's final status (Skipped or locked-Answered) is always resolved before or together with the new week opening.
- No reminder, notification, or nudge accompanies either the new week opening or the previous week being marked Skipped, per feature-overview.md's Non-Goals.
- This cycle never modifies a household's one-time starting point (starting_point_waste, starting_point_spend); once set, that data is untouched by every run of this automation.
- A locked record's waste_amount and spend are never altered by this automation -- locking only sets the locked flag; the data itself is exactly what the household last submitted (FEAT-25.SPEC-001).

## Edge Cases

- **Concurrent trigger firing (two cycle runs for the same household fire at effectively the same time)** -- Only one run's create/update actually lands; the other detects that the new week's record already exists and the prior week is already resolved, and performs no further action, per the idempotency business rule above.
- **Trigger fires while a previous run for the same household is still in flight** -- The second firing evaluates state only after the in-flight run completes, so it always sees the post-cutover state and takes no duplicate action; runs for different households never block one another.
- **Household has never answered any prior week (every week Offered, then Skipped)** -- The cycle continues to open a fresh Offered record every week regardless of the household's answer history; the First-Use, starting-point card variant (FEAT-25.SPEC-001) keeps appearing on every new week until the household's first answer sets a starting point.
- **Household's very first-ever week using this feature** -- No previous-week record exists to resolve; the cycle performs only the create-new-week step, per Processing Logic step 8.
- **A household is deleted (FEAT-18) between one cycle run and the next** -- The cycle takes no action for a deleted household; cascade or export behavior for its existing check-in history is FEAT-18's responsibility (feature-overview.md's Side-Effect Inventory), not this automation's.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-25.SPEC-001 (Weekly Check-In Card) | Affects (outbound) | Provides the fresh Offered (or First-Use-triggering) record the card renders, and locks the record the card can no longer edit |
| FEAT-25.SPEC-002 (Check-In Trend View) | Affects (outbound) | Provides the finalized (Skipped or locked-Answered) historical records the trend view reads |
| FEAT-25.SPEC-005 (Check-In Validation & Access Rules) | References (inbound) | The editable-until-next-week-opens window this automation enforces is defined there |

## Analytics and Success Signals

- **checkin_week_opened** (household_id, week) -- supports success-metrics.md: "Reported Food Waste and Spend Reduction" (the denominator against which how many offered weeks a household answers or skips is measured)
- **checkin_skipped** (household_id, week) -- supports success-metrics.md: "Reported Food Waste and Spend Reduction"
- **checkin_locked** (household_id, week) -- N/A -- this event is a state-transition bookkeeping signal with no direct bearing on the waste-and-spend reduction measure; recorded for completeness of the entity's lifecycle instrumentation

## Acceptance Criteria

**FEAT-25.SPEC-003-AC-01:** Given a household's new planning week begins and no record exists for it yet, when the cycle runs, then a new Waste & Spend Check-In record is created with status Offered.

**FEAT-25.SPEC-003-AC-02:** Given a household's previous week's record has status Offered (no answer was ever given), when the new week's cycle runs, then the previous week's record is set to Skipped.

**FEAT-25.SPEC-003-AC-03:** Given a household's previous week's record has status Answered, when the new week's cycle runs, then the previous week's record's locked flag is set to true and its waste_amount and spend values are unchanged.

**FEAT-25.SPEC-003-AC-04:** Given a household's new week's record already exists and its previous week's record is already Skipped, when the cycle runs again for the same boundary, then no data changes and no duplicate record is created.

**FEAT-25.SPEC-003-AC-05:** Given a household is using Plateful for its very first week, when the cycle runs, then only a new Offered record is created, since no previous week's record exists to resolve.

**FEAT-25.SPEC-003-AC-06:** Given two cycle runs fire for the same household at effectively the same time, when both attempt to open the new week and resolve the previous one, then only one run's changes land and the other takes no further action.

**FEAT-25.SPEC-003-AC-07:** Given a cycle run for a household is still in flight, when a second trigger fires for the same household, then the second run waits and, on evaluating state, finds the cutover already complete and takes no action.

**FEAT-25.SPEC-003-AC-08:** Given a household is deleted between one cycle run and the next, when the cycle would otherwise run for that household, then no action is taken, and any cascade or export behavior for its check-in history is handled by FEAT-18.

**FEAT-25.SPEC-003-AC-09:** Given the cycle fails partway through processing a household, when it is retried, then the household is left with neither two open weeks nor zero current weeks -- the retry completes the cutover as a single unit.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 6 | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |
