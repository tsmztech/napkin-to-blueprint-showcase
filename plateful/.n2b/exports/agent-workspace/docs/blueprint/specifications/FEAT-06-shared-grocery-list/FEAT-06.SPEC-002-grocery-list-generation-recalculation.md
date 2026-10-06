---
document_type: spec
spec_type: automation
spec_id: FEAT-06.SPEC-002
spec_name: Grocery List Generation & Recalculation
spec_slug: grocery-list-generation-recalculation
parent_feature: FEAT-06
parent_feature_name: Shared Grocery List
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Grocery List Generation & Recalculation

## Overview

**Name:** Grocery List Generation & Recalculation
**ID:** FEAT-06.SPEC-002
**Type:** Automation
**Purpose:** Builds the current week's Grocery List from the plan's ingredients on generation or first pick, and recalculates the plan-derived portion whenever the plan, a swap, a safety removal, or pantry data changes, while preserving manual items, ticks, and "already have it" marks.
**Parent Feature:** FEAT-06 -- Shared Grocery List

## Scope and Non-Goals

**In Scope:**
- Creating the week's Grocery List the first time a plan exists (generated or manually started)
- Recalculating the plan-derived portion of the list on every plan-affecting change
- Preserving manual items, ticked state, and "already have it" removals across recalculation
- Invoking ingredient consolidation and pantry exclusion for the plan-derived computation

**Non-Goals:**
- The consolidation and quantity-derivation math itself -- owned by FEAT-06.SPEC-006; this automation invokes it rather than duplicating it
- Manual item validation and duplicate merge -- owned by FEAT-06.SPEC-007
- Archiving the list at week end and carrying items forward -- owned by FEAT-06.SPEC-004; this automation only seeds a newly created week's list, it does not decide when a week ends
- Propagating the recalculated list to other devices -- owned by FEAT-06.SPEC-005; this automation persists the data and hands off to that integration

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Plan generated or approved | FEAT-03 (AI Weekly Dinner Plan Generation) | Weekly Plan status moves to Approved or is auto-adopted at week start | Weekly Plan (week, status), all Planned Meals with their recipes |
| Manual pick made | FEAT-23 (Manual Weekly Planning) | A household picks, changes, or clears a night's recipe | The affected Planned Meal (night, recipe, status) |
| Meal swapped or suggestion accepted | FEAT-04 (One-Tap Meal Swap) | A swap or accepted suggestion completes for a Planned Meal | The affected Planned Meal's new recipe |
| Safety-concern removal | FEAT-02 (Dietary Rules & Allergy Safety Engine), XBR-08 | A safety-concern report removes a meal from the plan, or a mid-week rule change (XBR-02) removes one | The removed Planned Meal's prior recipe |
| Pantry item logged or cleared | FEAT-05 (Pantry-Aware Suggestions), XBR-04 | A Pantry Item is created (Active) or removed (Used/Removed) | The affected Pantry Item's `item_name` |
| Week rollover seeds a new list | FEAT-06.SPEC-004 (Week Rollover & Carryover) | The new week's list is being created at week start | Carried-over manual Grocery List Items from the archived list |

## Processing Logic

1. Identify the household's current-week Weekly Plan (from FEAT-03 or FEAT-23).
2. Gather all Planned Meals in that plan (dinners and confirmed leftover lunches) excluding any in a Removed status.
3. Invoke FEAT-06.SPEC-006 (Ingredient Consolidation & Quantity Derivation) with those Planned Meals to compute the combined, aisle-grouped, pantry-excluded set of plan-derived lines.
4. Compare the newly computed plan-derived lines against the Grocery List's existing plan-derived lines (matched by `ingredient_name`).
5. For each plan-derived line whose contributing dinners are unchanged: update `quantity_and_unit` and `aisle` only if they changed, and preserve `ticked` as-is.
6. For each plan-derived line whose contributing dinners changed (a swap or safety removal altered which dinners need it): rewrite the line in place with the newly computed quantity and reset `ticked` to false, since the underlying need has changed.
7. For each plan-derived line no longer needed by any dinner: delete the Grocery List Item.
8. For each newly needed ingredient with no existing line: create a new Grocery List Item with `origin` set to "plan-derived".
9. Leave every manual-origin Grocery List Item untouched, regardless of any of the above.
10. If this run is seeding a newly created week's list (triggered by FEAT-06.SPEC-004), add the carried-over manual items supplied by that spec as manual-origin lines before completing.
11. Set the Grocery List's `status` to "Generated" on first population for the week, then "Active" once it has at least one item.
12. Persist the updated list and items, then hand off to FEAT-06.SPEC-005 (Live Grocery List Sync) to propagate the change to every household device.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Initial generation | No Grocery List exists yet for the current week | Grocery List created (status: Generated -> Active); plan-derived items created | List populates on FEAT-06.SPEC-001 within a couple of seconds, with an inline loading indicator | FEAT-06.SPEC-001, FEAT-06.SPEC-005 |
| Recalculation with changes | A trigger fires and the computed plan-derived set differs from the existing one | Affected Grocery List Items created, updated, or deleted; manual items untouched | List updates live on FEAT-06.SPEC-001, with no separate notification (silent per this feature's Communications) | FEAT-06.SPEC-001, FEAT-06.SPEC-005 |
| No change needed | A trigger fires but the computed set is identical to the existing one (e.g., a swap changed cook time but not ingredients) | None | Nothing visibly changes | FEAT-06.SPEC-001 |
| Recalculation failure | A candidate recipe's ingredient data is incomplete, or the computation cannot complete | The prior, last-known-good list is kept as-is; the incomplete recipe is excluded from this run's computation, consistent with FEAT-02's fail-closed principle | No error shown to the household; the list they see remains valid and usable. The failure is logged for the next successful trigger to retry in full | FEAT-06.SPEC-001 |

## Data Model

**Reads:** Weekly Plan -- `week`, `status`. Planned Meal -- `night`, `recipe`, `status`. Recipe -- ingredients (via FEAT-06.SPEC-006). Pantry Item -- `item_name`, `status`. Household -- `aisle_grouping`, `unit_system`.

**Creates:** Grocery List (on first population for the week) -- `week`, `aisle_grouping`, `status`. Grocery List Item (plan-derived) -- `ingredient_name`, `quantity_and_unit`, `aisle`, `origin` ("plan-derived"), `ticked` (false).

**Updates:** Grocery List -- `status`. Grocery List Item (plan-derived) -- `quantity_and_unit`, `aisle`, `ticked` (reset only when contributing dinners changed).

**Deletes:** Grocery List Item (plan-derived) -- when no longer needed by any dinner.

## Business Rules

- XBR-03: every pick, change, swap, accepted suggestion, or safety removal updates the list immediately; no member ever sees a week's plan without its matching list.
- XBR-04: pantry exclusion (via FEAT-06.SPEC-006) is applied on every recalculation, not only at initial generation.
- XBR-11: aisle grouping and quantities follow the household's current unit system and aisle configuration (FEAT-16) at the time of each recalculation.
- Manual Grocery List Items are never created, modified, or deleted by this automation -- only FEAT-06.SPEC-001 (via FEAT-06.SPEC-007) and FEAT-06.SPEC-004 (carryover) touch them.
- A plan-derived line's `ticked` state survives recalculation only when the same set of contributing dinners still needs the same ingredient; a change in contributing dinners resets it, since the prior tick no longer reflects a verified purchase against the current need.

## Edge Cases

- **Concurrent trigger firing (e.g., a swap and a pantry-item log arrive at nearly the same time)** -- Each trigger recomputes the full plan-derived set from the household's current source data rather than applying a delta, so overlapping runs converge to the same result regardless of order; no update is lost.
- **Trigger fires while a previous run is in flight** -- Recalculation for a household is not re-entrant: a trigger arriving mid-run queues and executes immediately after the in-flight run completes, using the latest source data at that time, so no trigger is dropped.
- **A recipe with incomplete ingredient data is on the plan mid-week** -- That recipe's ingredients are excluded from the computed set this run, consistent with FEAT-02's fail-closed principle; the rest of the plan-derived list still recalculates normally.
- **A safety-removed meal's ingredient is also needed by another dinner still on the plan** -- The ingredient's line is reduced (not removed), reflecting only the remaining dinner's need, via FEAT-06.SPEC-006.
- **An older-kid login (Pantry Input: None) marks "already have it" on an ingredient, and a later swap still needs it** -- Since no Pantry Item was created for this role's action (FEAT-06.SPEC-003), nothing excludes the ingredient from recalculation, and the line is re-added if the plan still needs it. This is a deliberate consequence of the role's lack of pantry access, not a defect.
- **Week rollover fires before the new week's plan exists yet** -- The new Grocery List is created empty except for FEAT-06.SPEC-004's carried-over manual items; plan-derived lines populate once a plan exists for that week and this automation's other triggers fire.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03 (AI Weekly Dinner Plan Generation) | Triggered by (inbound) | Plan generation/approval fires this automation |
| FEAT-23 (Manual Weekly Planning) | Triggered by (inbound) | Manual picks fire this automation |
| FEAT-04 (One-Tap Meal Swap) | Triggered by (inbound) | Swaps and accepted suggestions fire this automation |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | Triggered by (inbound) | Safety removals fire this automation (XBR-08) |
| FEAT-05 (Pantry-Aware Suggestions) | Triggered by (inbound) | Pantry item log/clear fires this automation (XBR-04) |
| FEAT-06.SPEC-006 (Ingredient Consolidation & Quantity Derivation) | Triggers (outbound) | Invoked to compute the combined, pantry-excluded plan-derived set |
| FEAT-06.SPEC-001 (Grocery List) | Affects (outbound) | Displays the resulting list |
| FEAT-06.SPEC-004 (Week Rollover & Carryover) | Triggered by (inbound) | Seeds a newly created week's list with carried-over manual items |
| FEAT-06.SPEC-005 (Live Grocery List Sync) | Triggers (outbound) | Propagates the recalculated list to every household device |

## Analytics and Success Signals

- **grocery_list_generated** (trigger_type: initial/plan/swap/safety/pantry/rollover) -- supports success-metrics.md: "Grocery List Live-Update Trust"
- **grocery_items_combined** (line count reduced through consolidation) -- N/A -- success-metrics.md defines no metric for consolidation volume; retained to observe how often the combine-across-dinners rule (FEAT-06.SPEC-006) applies in practice
- **grocery_list_recalculation_failed** (reason: incomplete_ingredient_data / processing_error) -- N/A -- success-metrics.md defines no metric for recalculation failure rate; retained to monitor the "always matching list" guarantee operationally

## Acceptance Criteria

**FEAT-06.SPEC-002-AC-01:** Given Maya's household has no Grocery List yet this week, when her weekly plan is approved, then a Grocery List is created with plan-derived lines for every ingredient across the week's dinners, grouped by aisle.

**FEAT-06.SPEC-002-AC-02:** Given Sam's household plans manually, when Sam picks a recipe for Tuesday, then the list recalculates to include Tuesday's ingredients within a couple of seconds.

**FEAT-06.SPEC-002-AC-03:** Given a dinner is swapped for a different recipe, when the swap completes, then the list's plan-derived lines update to reflect the new recipe's ingredients, and the old recipe's ingredients no longer needed by any other dinner are removed.

**FEAT-06.SPEC-002-AC-04:** Given a safety-concern report removes a meal from the plan, when the removal completes, then that meal's ingredients are dropped from the list (or reduced, if shared with another dinner still on the plan).

**FEAT-06.SPEC-002-AC-05:** Given Maya logs "spinach" as a pantry item before the week's plan generates, when the plan generates, then "spinach" is excluded from the plan-derived list per FEAT-06.SPEC-006.

**FEAT-06.SPEC-002-AC-06:** Given Maya has manually added "birthday candles" to the list, when the plan is swapped and the list recalculates, then "birthday candles" remains on the list untouched.

**FEAT-06.SPEC-002-AC-07:** Given a plan-derived line "chicken" was ticked and the contributing dinner is unchanged, when a swap affecting a different night triggers recalculation, then "chicken" remains ticked.

**FEAT-06.SPEC-002-AC-08:** Given a plan-derived line "chicken" was ticked and the dinner needing it is swapped for a different recipe, when the swap triggers recalculation, then the "chicken" line (if still needed elsewhere) is unticked, since the contributing dinner changed.

**FEAT-06.SPEC-002-AC-09:** Given a recipe on the plan has incomplete ingredient data, when recalculation runs, then that recipe's ingredients are excluded from this run's computed set and the rest of the list recalculates normally, with no error shown to the household.

**FEAT-06.SPEC-002-AC-10:** Given a swap and a pantry-item log occur for the same household at effectively the same time, when both triggers fire, then both recalculation runs converge to the same final list, with no update lost.

**FEAT-06.SPEC-002-AC-11:** Given a recalculation is already in progress for a household, when a new trigger fires before it completes, then the new trigger's recalculation runs immediately after the in-flight one finishes, using the latest source data.

**FEAT-06.SPEC-002-AC-12:** Given Jordan (older kid, limited login) marks "already have it" on an ingredient with no pantry entry created, when a later swap still needs that ingredient, then the ingredient's line is re-added to the list on recalculation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 6 | 6 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
