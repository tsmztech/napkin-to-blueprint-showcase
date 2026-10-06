---
document_type: spec
spec_type: automation
spec_id: FEAT-23.SPEC-006
spec_name: Apply Manual Pick
spec_slug: apply-manual-pick
parent_feature: FEAT-23
parent_feature_name: Manual Weekly Planning
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Automation Spec: Apply Manual Pick

## Overview

**Name:** Apply Manual Pick
**ID:** FEAT-23.SPEC-006
**Type:** Automation
**Purpose:** Writes a pick, change, or clear to the night's Planned Meal slot, recalculates the week's estimated total, and signals the shared grocery list to recalculate.
**Parent Feature:** FEAT-23 -- Manual Weekly Planning

## Scope and Non-Goals

**In Scope:**
- Writing a new Planned Meal (create) when a night is picked
- Overwriting an existing Planned Meal's recipe, cook_time, and rough_cost (update) when a night is changed
- Removing a Planned Meal entirely (delete) when a night is cleared
- Recalculating the Weekly Plan's estimated_total after every create, update, or delete
- Signalling the Shared Grocery List (FEAT-06) to recalculate immediately after every create, update, or delete

**Non-Goals:**
- Determining recipe eligibility -- owned by FEAT-23.SPEC-004 (Safe-Choice Filtering & Placement Block); this automation assumes the triggering screen has already confirmed eligibility and re-validates only the structural limits, not safety
- Applying a pick from an accepted Swap Suggestion -- owned by FEAT-04 (One-Tap Meal Swap) per the Side-Effect Inventory's disposition for suggestion acceptance (FEAT-04.SPEC-004 / FEAT-04.SPEC-006 execute that path); this automation's triggers are limited to the direct pick, change, and clear actions on FEAT-23.SPEC-001 and FEAT-23.SPEC-002
- Recalculating the grocery list's contents itself -- owned by FEAT-06 (Shared Grocery List) per feature-dependency-map.md's authority column and XBR-03; this automation only signals that a recalculation is needed

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pick placed | FEAT-23.SPEC-002 (Pick / Change a Recipe) | The target night has no existing dinner-status Planned Meal; the selected recipe passed FEAT-23.SPEC-004's eligibility check; the Weekly Plan's week is within FEAT-23.SPEC-005's one-week-ahead window | Night, recipe (name, cook_time, rough_cost, safety_badge) |
| Recipe changed | FEAT-23.SPEC-002 (Pick / Change a Recipe) | The target night has an existing dinner-status Planned Meal; the newly selected recipe passed FEAT-23.SPEC-004's eligibility check | Night, existing Planned Meal reference, new recipe (name, cook_time, rough_cost, safety_badge) |
| Night cleared | FEAT-23.SPEC-001 (Weekly Plan) | The organiser confirms "Remove" on a picked night's Clear action | Night, existing Planned Meal reference |

## Processing Logic

1. Receive the trigger context: the target night, and either the selected recipe (pick or change) or a clear instruction.
2. For a pick or change: re-confirm the night still satisfies FEAT-23.SPEC-005's structural rules (create requires no existing dinner on the night; change requires an existing one) and that the recipe's eligibility (FEAT-23.SPEC-004) has not lapsed since the triggering screen's last check.
3. If re-confirmation fails (the night's state changed, or the recipe is no longer eligible), stop and return the failure outcome to the triggering screen without writing any data.
4. For a pick: create a new Planned Meal record for the night with status "Picked," and the recipe, cook_time, rough_cost, and safety_badge carried from the selected recipe.
5. For a change: overwrite the existing Planned Meal's recipe, cook_time, rough_cost, and safety_badge with the newly selected recipe's values; the record's night and status are unchanged.
6. For a clear: delete the Planned Meal record for the night entirely.
7. Recalculate the Weekly Plan's estimated_total by summing the rough_cost of every remaining Planned Meal (dinner) in the week.
8. Signal the Shared Grocery List (FEAT-06) that this week's plan has changed, so it recalculates immediately (XBR-03).
9. Determine whether this action brings the organiser's count of filled nights in the current session to five or more for the first time this week; if so, emit the week-completion signal.
10. Return the outcome (success or failure) to the triggering screen.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Pick applied | A new pick's create succeeds | New Planned Meal created (status "Picked"); Weekly Plan.estimated_total recalculated | FEAT-23.SPEC-001 shows the night with its new recipe; FEAT-23.SPEC-002 navigates back to FEAT-23.SPEC-001 | FEAT-23.SPEC-001, FEAT-23.SPEC-002, FEAT-06, FEAT-21.SPEC-003 |
| Change applied | A change's update succeeds | Existing Planned Meal's recipe, cook_time, rough_cost, and safety_badge overwritten; Weekly Plan.estimated_total recalculated | FEAT-23.SPEC-001 shows the night with its new recipe; FEAT-23.SPEC-002 navigates back to FEAT-23.SPEC-001 | FEAT-23.SPEC-001, FEAT-23.SPEC-002, FEAT-06, FEAT-21.SPEC-003 |
| Clear applied | A clear's delete succeeds | Planned Meal removed; Weekly Plan.estimated_total recalculated | FEAT-23.SPEC-001 shows the night as "Nothing planned" | FEAT-23.SPEC-001, FEAT-06 |
| Re-confirmation failed (night state changed) | The night's existing-dinner state no longer matches what the triggering screen expected (e.g., cleared or picked from another device in the interim) | None | FEAT-23.SPEC-002 shows "This night changed while you were choosing -- reload to see the latest pick"; FEAT-23.SPEC-001's Clear shows the night already in its current state | FEAT-23.SPEC-001, FEAT-23.SPEC-002 |
| Re-confirmation failed (recipe no longer eligible) | The selected recipe fails FEAT-23.SPEC-004's check at write time (e.g., a mid-week rule tightened after the screen's last load) | None | FEAT-23.SPEC-002 shows the recipe's ineligibility reason inline; no write occurs | FEAT-23.SPEC-002 |
| Write failure (e.g., dropped connection) | The create, update, or delete cannot be completed for a reason other than eligibility or state conflict | None -- no partial write is left behind | The attempted pick, change, or clear stays visible on the triggering screen with a retry option, never silently dropped | FEAT-23.SPEC-001, FEAT-23.SPEC-002 |

## Data Model

**Reads:** Weekly Plan (week, current estimated_total); Planned Meal (existing record for the target night, when changing or clearing); Recipe (name, cook_time, rough_cost, safety_badge) for the newly selected recipe.
**Creates:** Planned Meal -- night, recipe, cook_time, rough_cost, safety_badge, and status "Picked" (pick trigger only).
**Updates:** Planned Meal -- recipe, cook_time, rough_cost, safety_badge (change trigger only); Weekly Plan.estimated_total (every trigger, recalculated).
**Deletes:** Planned Meal -- the entire record for the target night (clear trigger only).

## Business Rules

- XBR-03: every pick, change, or clear updates the shared grocery list immediately; no member ever sees this week's plan without its matching list.
- FEAT-23.SPEC-005 governs the structural limits (one dinner per night, one week ahead) this automation re-confirms before writing; FEAT-23.SPEC-004 governs the eligibility this automation re-confirms before writing.
- A cleared night's Planned Meal is a hard delete with no restore path -- the organiser simply picks again if she changes her mind, consistent with the Entity-Lifecycle Coverage Matrix's Delete/Archive disposition for Planned Meal under this feature.
- This automation runs synchronously with the triggering screen's save action -- the screen waits for its outcome (success or failure) before completing.

## Edge Cases

- **Maya's pick, change, or clear fails to save (e.g., dropped connection)** -- The attempted action stays visible on the triggering screen with a retry option; never silently dropped, per the Side-Effect Inventory.
- **A hard dietary rule tightens between the triggering screen's last load and this automation's write** -- The write is rejected with the standard ineligibility reason; no Planned Meal is created or updated.
- **The target night's state changes between the triggering screen's last load and this automation's write (e.g., cleared or picked from another device)** -- The write is rejected with "This night changed while you were choosing -- reload to see the latest pick," consistent with reject-with-refresh per the dependency map's Contention note for Planned Meal.
- **A safety-concern removal (FEAT-02) runs concurrently against the same night** -- The safety removal always wins over this automation's concurrent change, per the dependency map's Contention note for Planned Meal; this automation's write is rejected if the removal commits first.
- **Concurrent trigger firing -- Maya picks two different nights from two devices at effectively the same time** -- Each trigger's write proceeds independently against its own night; there is no shared state between different nights, so neither is blocked by the other.
- **A trigger fires for the same night while a previous run for that night is still in flight** -- The triggering screen's Place/Replace/Clear action is disabled while its own save is in progress (per FEAT-23.SPEC-001 and FEAT-23.SPEC-002's Interactions), so a second run for the same night from the same screen cannot start; a second device attempting the same night's write while the first is in flight is handled by the standard state-conflict rejection above once it arrives.
- **Clearing a night that has already been cleared (e.g., a stale Clear button tapped after the night was already cleared elsewhere)** -- No Planned Meal exists to delete; the automation returns the "night already empty" outcome without error, and the triggering screen refreshes to show the night as already "Nothing planned."

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-001 (Weekly Plan) | Triggered by (inbound) | The confirmed Clear action triggers this automation's delete path |
| FEAT-23.SPEC-002 (Pick / Change a Recipe) | Triggered by (inbound) | The Place and Replace actions trigger this automation's create and update paths |
| FEAT-23.SPEC-001 (Weekly Plan) | Affects (outbound) | Displays the resulting night state and the recalculated estimated_total |
| FEAT-23.SPEC-002 (Pick / Change a Recipe) | Affects (outbound) | Displays success navigation or the rejection/error feedback for a failed write |
| FEAT-23.SPEC-004 (Safe-Choice Filtering & Placement Block) | References (inbound) | Re-confirmed at write time before any create or update |
| FEAT-23.SPEC-005 (Manual Planning Validation & Limits) | References (inbound) | Re-confirmed at write time before any create, update, or delete |
| FEAT-06 (Shared Grocery List) | Affects (outbound) | Signalled to recalculate immediately after every successful create, update, or delete |

## Analytics and Success Signals

- **manual_meal_applied** (outcome: created / updated / cleared; night) -- supports success-metrics.md: "Manual Week Completion"
- **manual_week_completed** (nights_filled_count reaching five or more in the session) -- supports success-metrics.md: "Manual Week Completion"
- **manual_pick_write_failed** (reason: state_conflict / eligibility_lapsed / connection_failure) -- N/A -- no success-metrics.md metric measures this automation's failure rate directly; recorded to confirm the "never silently dropped" guarantee is exercised as intended, not to feed a Stage 2 metric
- **grocery_list_recalc_signaled** (night, outcome) -- N/A -- the grocery list's own responsiveness is measured by success-metrics.md: "Grocery List Live-Update Trust," whose Connected Feature is FEAT-06, not this feature; this automation's signal is the trigger for that measurement, not a metric of its own

## Acceptance Criteria

**FEAT-23.SPEC-006-AC-01:** Given Maya has selected an eligible recipe for Wednesday's empty night on FEAT-23.SPEC-002, when she confirms Place, then a new Planned Meal is created with status "Picked" and the Weekly Plan's estimated_total recalculates to include it.

**FEAT-23.SPEC-006-AC-02:** Given Maya has selected a new eligible recipe to replace Friday's existing dinner, when she confirms Replace, then the existing Planned Meal's recipe, cook_time, rough_cost, and safety_badge are overwritten and estimated_total recalculates.

**FEAT-23.SPEC-006-AC-03:** Given Maya confirms "Remove" on Monday's picked dinner, when this automation runs, then the Planned Meal is deleted and estimated_total recalculates to exclude it.

**FEAT-23.SPEC-006-AC-04:** Given a pick, change, or clear completes successfully, when this automation finishes, then the Shared Grocery List (FEAT-06) is signalled to recalculate immediately (XBR-03).

**FEAT-23.SPEC-006-AC-05:** Given Maya's selected recipe passed eligibility on FEAT-23.SPEC-002 but a household member's allergy tightened before this automation writes it, when the automation re-confirms eligibility, then the write is rejected and FEAT-23.SPEC-002 shows the recipe's ineligibility reason.

**FEAT-23.SPEC-006-AC-06:** Given Wednesday's night state changed on another device between FEAT-23.SPEC-002's load and this automation's write, when the write is attempted, then it is rejected with "This night changed while you were choosing -- reload to see the latest pick."

**FEAT-23.SPEC-006-AC-07:** Given a network failure occurs during this automation's write, when the failure happens, then no partial write is left behind and the attempted pick, change, or clear stays visible on the triggering screen with a retry option.

**FEAT-23.SPEC-006-AC-08:** Given a safety-concern removal (FEAT-02) commits against the same night at effectively the same time as this automation's change, when both are in flight, then the safety removal wins and this automation's write is rejected.

**FEAT-23.SPEC-006-AC-09:** Given Maya picks two different nights from two devices at effectively the same time, when both writes run, then each completes independently against its own night with no blocking between them.

**FEAT-23.SPEC-006-AC-10:** Given Maya has already filled four nights this session and successfully picks a fifth, when this automation completes that fifth pick, then a manual_week_completed signal is emitted, supporting success-metrics.md: "Manual Week Completion."

**FEAT-23.SPEC-006-AC-11:** Given Maya taps Clear on a night that was already cleared from another device moments earlier, when this automation runs, then it returns the "night already empty" outcome without error and the screen refreshes to show the night as "Nothing planned."

**FEAT-23.SPEC-006-AC-12:** Given a pick, change, or clear is being saved from FEAT-23.SPEC-001 or FEAT-23.SPEC-002, when the triggering screen's own action control is in its loading state, then a second trigger for the same night from that same screen cannot start until the first completes.

**FEAT-23.SPEC-006-AC-13:** Given Maya applies a pick, change, or clear, when this automation completes any outcome, then no attempt is ever made to apply an accepted Swap Suggestion through this automation -- that path is owned entirely by FEAT-04.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (pick, change, clear) | 3 |
| Outcome Paths | 6 (pick applied, change applied, clear applied, state-conflict rejection, eligibility-lapsed rejection, write failure) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |
