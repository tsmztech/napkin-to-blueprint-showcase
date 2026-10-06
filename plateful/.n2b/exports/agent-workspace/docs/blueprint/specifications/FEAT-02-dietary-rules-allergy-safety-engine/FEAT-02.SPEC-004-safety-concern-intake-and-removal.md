---
document_type: spec
spec_type: automation
spec_id: FEAT-02.SPEC-004
spec_name: Safety Concern Intake & Removal
spec_slug: safety-concern-intake-and-removal
parent_feature: FEAT-02
parent_feature_name: Dietary Rules & Allergy Safety Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Safety Concern Intake & Removal

## Overview

**Name:** Safety Concern Intake & Removal
**ID:** FEAT-02.SPEC-004
**Type:** Automation
**Purpose:** Removes a reported meal from the plan immediately, excludes the recipe for the household while the report is open, and opens the review record.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- Removing the reported Planned Meal from the household's plan at once
- Creating the Support Request (kind = safety concern) that opens the review record
- Excluding the reported recipe from the household's future candidate pool while the report is open
- Updating the grocery list and offering safe alternatives for the emptied slot

**Non-Goals:**
- Capturing the report itself (the note, the meal reference) -- owned by FEAT-02.SPEC-001 (Report a Safety Concern), whose submission triggers this automation
- Deciding whether the excluded recipe may return to the candidate pool once reviewed -- owned by FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy)
- Applying the operator's review outcome -- owned by FEAT-02.SPEC-005 (Safety Concern Resolution Outcome), a distinct later step this automation does not perform
- Reviewing the report or changing its status beyond Raised -- owned by FEAT-22 (Operator Read-Only Support Access), which this automation's created Support Request feeds

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An adult submits a safety concern report | FEAT-02.SPEC-001 (Report a Safety Concern) | Fires on every submission from the dialog, regardless of whether the recipe already has an open report | Reporting Member, the reported Planned Meal (and its Recipe), the optional note (up to 500 characters) |

## Processing Logic

1. Receive the reporting Member, the reported Planned Meal reference, its Recipe, and the optional note from FEAT-02.SPEC-001.
2. Check whether the reported Recipe already has an open Support Request (kind = safety concern, status not Resolved) for this Household.
3. If no open report exists for this Recipe, create a new Support Request: kind = safety concern, raised_by = the reporting Member, planned_meal/recipe = the reported meal and recipe, note = the optional text (or none), status = Raised.
4. If an open report already exists for this Recipe, do not create a duplicate Support Request -- proceed to the remaining steps against the existing report.
5. Set the reported Planned Meal's status to Removed (safety) and append the prior recipe to swap_history.
6. Mark the Recipe as excluded from this Household's candidate pool per FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy), for as long as the Support Request remains open.
7. Trigger the grocery list recalculation (FEAT-06) to drop the removed meal's ingredients.
8. Open a swap for the now-empty slot (FEAT-04) so a safe alternative can be selected.
9. Trigger FEAT-02.SPEC-011 (Safety Concern Reporter Acknowledgement) to confirm receipt and removal to the reporting Member.
10. If the reporting Member is not the organiser, trigger FEAT-02.SPEC-012 (Safety Concern Organiser Alert) to tell the organiser.
11. Trigger FEAT-02.SPEC-013 (Safety Concern Operator Alert), which delivers the report to the operator by transactional email through FEAT-02.SPEC-010.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| New report opened, meal removed | No open report existed for this Recipe | New Support Request created (status Raised); Planned Meal set to Removed (safety); Recipe excluded per FEAT-02.SPEC-009; grocery list updated | Reporter sees the removal confirmation (FEAT-02.SPEC-001); organiser alerted if not the reporter; operator alerted by email | FEAT-02.SPEC-001, FEAT-02.SPEC-011, FEAT-02.SPEC-012, FEAT-02.SPEC-013, FEAT-06, FEAT-04, FEAT-02.SPEC-009 |
| Duplicate report against an already-open concern | An open Support Request for this Recipe already exists | No new Support Request created; Planned Meal removal and recipe exclusion proceed (or are confirmed as already applied if the meal was already removed) | Reporter sees the same removal confirmation as a new report | FEAT-02.SPEC-001, FEAT-02.SPEC-011, FEAT-06, FEAT-04 |
| Meal already removed before this report (e.g., by a mid-week rule change) | The reported Planned Meal is already Removed (safety) from a prior cause | Support Request still created (or reused per the duplicate path) so the concern is formally recorded; Planned Meal status unchanged (already Removed) | Reporter sees a confirmation that the concern was recorded; no additional plan disruption since the meal is already gone | FEAT-02.SPEC-001, FEAT-02.SPEC-011, FEAT-02.SPEC-013 |
| Automation cannot complete (processing failure) | The removal or Support Request creation cannot be completed | No partial state -- either the full removal and report creation succeed together or neither does | Reporter sees the error state defined in FEAT-02.SPEC-001 ("Couldn't submit your report...") with a Retry option | FEAT-02.SPEC-001 |

## Data Model

**Reads:** Planned Meal -- current status, recipe reference. Support Request -- existing open reports for the same Recipe and Household, to detect duplicates.
**Creates:** Support Request -- kind (safety concern), raised_by, planned_meal/recipe, note, status (Raised).
**Updates:** Planned Meal -- status (Removed (safety)), swap_history (prior recipe appended).
**Deletes:** None -- removal is a status change; the Planned Meal record is retained in the plan's history, per the dependency map's Planned Meal lifecycle.

## Business Rules

- XBR-08: A safety-concern report removes the meal from the household's plan at once, excludes the recipe for that household while the report is open, drops its ingredients from the list, offers safe alternatives, reaches the operator for review, and tells the household the outcome.
- This automation never creates a second open Support Request for the same recipe while one is already open -- FEAT-02.SPEC-009 is the single source of truth for whether a recipe is under report, and this automation defers to it rather than tracking its own exclusion state.
- Removal is immediate and unconditional on submission -- there is no pending or under-review state in which the meal remains visible on the plan.
- A safety-concern removal always uses the same Removed (safety) status and swap_history mechanic as a mid-week rule-change removal (FEAT-02.SPEC-003); the two share one lifecycle transition on Planned Meal.
- Support Requests raised as safety concerns are never deleted or purged -- retained for the life of the household account as trust and audit history (SC-18).

## Edge Cases

- **Two different adults report the same meal within moments of each other** -- The second submission finds the first's Support Request already open (or being created) and does not create a duplicate; both reporters receive their own reporter acknowledgement, since each genuinely reported independently.
- **The reported meal has already been swapped out by the time the report is submitted** -- The removal and exclusion apply to the Recipe as reported, regardless of whether it currently occupies the slot; if it no longer occupies any slot, no Planned Meal status change occurs but the Support Request and recipe exclusion still proceed.
- **Reporting adult submits with no note** -- The Support Request is created with note = none; the operator alert and reporter acknowledgement proceed with no note content shown.
- **Concurrent trigger firing (two different meals reported by two different adults at effectively the same time)** -- Each report processes independently against its own Planned Meal and Recipe; there is no shared state between the two runs beyond the household's Support Request list, which each run reads and appends to without overwriting the other's write.
- **Trigger fires while a previous report for the same recipe is still being processed** -- The later trigger waits for the in-flight run's Support Request duplicate-check to complete before proceeding, so two near-simultaneous reports for the same recipe cannot both create separate open Support Requests.
- **Network interruption after the Support Request is created but before the Planned Meal status updates** -- The automation treats meal removal and Support Request creation as one atomic outcome for the user; if the removal cannot be confirmed, the reporter sees the failure state (FEAT-02.SPEC-001) and the reporting flow can be retried without creating a second Support Request, since the duplicate check on retry finds the one already created.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Report a Safety Concern) | Triggered by (inbound) | Submitting the report fires this automation |
| FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy) | Triggers (outbound) | Marks the recipe excluded while the report is open |
| FEAT-06 (Shared Grocery List) | Affects (outbound) | The list is recalculated to drop the removed meal's ingredients |
| FEAT-04 (One-Tap Meal Swap) | Affects (outbound) | A swap is opened for the emptied slot |
| FEAT-02.SPEC-011 (Safety Concern Reporter Acknowledgement) | Triggers (outbound) | Confirms receipt and removal to the reporter |
| FEAT-02.SPEC-012 (Safety Concern Organiser Alert) | Triggers (outbound) | Tells the organiser when she was not the reporter |
| FEAT-02.SPEC-013 (Safety Concern Operator Alert) | Triggers (outbound) | Sends the report to the operator for review |
| FEAT-22 (Operator Read-Only Support Access) | Affects (outbound) | The created Support Request is the precondition for the operator's support view |

## Analytics and Success Signals

- **safety_concern_reported** (recipe reference, reporter role, had_note: yes/no) -- supports success-metrics.md: "Zero Allergy Incidents"
- **safety_concern_meal_removed** (recipe reference, plan slot) -- supports success-metrics.md: "Zero Allergy Incidents"
- **safety_concern_duplicate_suppressed** (recipe reference) -- N/A -- no Stage 2 metric measures duplicate report suppression; retained so the pipeline's deduplication behavior is observable rather than silent

## Acceptance Criteria

**FEAT-02.SPEC-004-AC-01:** Given Maya submits a safety concern report with no prior open report on that recipe, when this automation runs, then a new Support Request is created with status Raised and Maya's optional note attached.

**FEAT-02.SPEC-004-AC-02:** Given a safety concern report is processed, when the Planned Meal is removed, then its status is set to Removed (safety), the prior recipe is appended to swap_history, and the grocery list drops its ingredients.

**FEAT-02.SPEC-004-AC-03:** Given Sam (not the organiser) submits a safety concern, when the removal completes, then Maya (the organiser) receives the Safety Concern Organiser Alert (FEAT-02.SPEC-012).

**FEAT-02.SPEC-004-AC-04:** Given Maya (the organiser) submits a safety concern herself, when the removal completes, then no organiser alert fires for her own report, since she is already the reporter.

**FEAT-02.SPEC-004-AC-05:** Given a safety concern report is created, when the automation completes, then the operator receives the Safety Concern Operator Alert (FEAT-02.SPEC-013) by transactional email.

**FEAT-02.SPEC-004-AC-06:** Given Sam reports a recipe that Maya already reported minutes earlier, when Sam's report is processed, then no duplicate Support Request is created, and Sam still receives his own reporter acknowledgement.

**FEAT-02.SPEC-004-AC-07:** Given a report is submitted with no note, when the Support Request is created, then note is recorded as none and the operator alert and reporter acknowledgement proceed without note content.

**FEAT-02.SPEC-004-AC-08:** Given a recipe is excluded by this automation, when any future candidate check runs for that household (FEAT-02.SPEC-002), then the recipe is excluded per FEAT-02.SPEC-009 for as long as the report remains open.

**FEAT-02.SPEC-004-AC-09:** Given the reported meal was already removed by a mid-week rule change before the report was submitted, when the report is processed, then the Support Request is still created, but no additional Planned Meal status change occurs.

**FEAT-02.SPEC-004-AC-10:** Given a swap is opened for the emptied slot by this automation, when Maya opens the swap, then she sees safe alternatives for that night (FEAT-04.SPEC-001).

**FEAT-02.SPEC-004-AC-11:** Given two adults report two different meals at effectively the same time, when both reports are processed, then each creates its own Support Request and neither run interferes with the other's data.

**FEAT-02.SPEC-004-AC-12:** Given a second report for the same recipe arrives while the first report's processing is still in flight, when the second trigger fires, then it waits for the first run's duplicate check before proceeding, so only one Support Request is created for that recipe.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
