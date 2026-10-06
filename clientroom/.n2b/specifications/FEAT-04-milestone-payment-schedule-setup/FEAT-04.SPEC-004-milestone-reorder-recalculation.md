---
document_type: spec
spec_type: automation
spec_id: FEAT-04.SPEC-004
spec_name: Milestone Reorder Recalculation
spec_slug: milestone-reorder-recalculation
parent_feature: FEAT-04
parent_feature_name: Milestone & Payment Schedule Setup
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Milestone Reorder Recalculation

## Overview

**Name:** Milestone Reorder Recalculation
**ID:** FEAT-04.SPEC-004
**Type:** Automation
**Purpose:** Renumbers the remaining milestones' order whenever one is added, removed, or manually reordered, so the project's milestone sequence stays contiguous.
**Parent Feature:** FEAT-04 -- Milestone & Payment Schedule Setup

## Scope and Non-Goals

**In Scope:**
- Recalculating and persisting the `order` field for every milestone whose position shifts as a result of an add, a remove, or a manual reorder
- Keeping the project's milestone sequence contiguous (no gaps, no duplicates) at all times after this automation completes
- Returning the recalculated order to the triggering screen for immediate re-render

**Non-Goals:**
- Determining whether a removal is eligible in the first place -- owned by FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules); this automation only renumbers a removal that spec has already allowed, per XBR-10.
- Field validation, price-mismatch flagging, or authorization on milestone actions -- all owned by FEAT-04.SPEC-003; this automation performs no validation of its own beyond confirming the triggering change is well-formed.
- Changing any milestone field other than `order` -- name, price, payment_trigger, target_date, and status are never touched by this automation, even for milestones whose position shifts.
- Recalculating milestone order across projects -- excluded per the dependency map's Milestone relationship line: a milestone "belongs to one Project," and its `order` is scoped and renumbered within that single project only.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Milestone added (create) | FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | Fires immediately after a new milestone is successfully saved | The new milestone's chosen position (end of list by default) and the full ordered list of the project's existing milestones |
| Milestone removed | FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | Fires immediately after a removal is successfully committed -- i.e., after FEAT-04.SPEC-003's approved/invoiced eligibility check passes | The removed milestone's former order value and the full ordered list of the project's remaining milestones |
| Milestone manually reordered | FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | Fires when Nadia drags or uses Move Up/Down controls to move a milestone and the move is committed | The moved milestone's old and new position and the full ordered list of the project's milestones |

## Processing Logic

1. Receive the triggering change: which milestone was added, removed, or moved, and its position (chosen position for an add, former position for a remove, old and new position for a manual reorder).
2. Read the project's current full list of milestones, ordered by their existing `order` value.
3. For an add: insert the new milestone at its chosen position (end of the list by default) and shift every milestone at or after that position down by one position.
4. For a remove: take the milestone out of the list and shift every milestone that was after its former position up by one position, so the sequence stays contiguous starting at 1.
5. For a manual reorder: take the moved milestone out of its old position, shift the milestones between its old and new position by one to close the gap, then insert the moved milestone at its new position.
6. Write the recalculated `order` value to every milestone whose position changed as a result of steps 3-5 -- not only the milestone directly acted on.
7. Skip any milestone whose position did not change -- no write and no event for those records.
8. Return the updated ordered list to the triggering screen (FEAT-04.SPEC-001) for immediate re-render.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Renumbered successfully | The recalculation completes without conflict | The `order` field updates on every milestone whose position shifted | The milestone list on FEAT-04.SPEC-001 re-renders immediately in the new contiguous order | FEAT-04.SPEC-001, FEAT-04.SPEC-002 (client view reflects it on its next read) |
| No-op (zero or one milestone remains) | The project has zero milestones, or exactly one, after the triggering change | No `order` values need changing | The list shows the single milestone at position 1, or the Empty state prompt | FEAT-04.SPEC-001 |
| Not triggered (removal was refused) | The triggering removal was itself refused by FEAT-04.SPEC-003 (approved/invoiced milestone) | None -- this automation never runs, since the removal that would have fired it did not commit | The milestone list is unchanged; FEAT-04.SPEC-003's denied message is shown instead | FEAT-04.SPEC-001, FEAT-04.SPEC-003 |
| Failure (recalculation cannot complete) | An error occurs while writing the recalculated `order` values | Fully rolled back -- either every affected milestone's `order` updates, or none do | Non-blocking warning on FEAT-04.SPEC-001: "Milestones saved, but reordering couldn't be completed. Refresh to see the current order." | FEAT-04.SPEC-001 |

## Data Model

**Reads:** Milestone -- `order`, project reference, and status (status is read only to confirm which milestones belong to the current project; it is never changed here) -- for every milestone in the triggering project.

**Creates:** None.

**Updates:** Milestone -- `order` field only, for every milestone whose position shifted as a result of the triggering add, remove, or reorder.

**Deletes:** None -- the milestone removal itself is performed by FEAT-04.SPEC-001 under FEAT-04.SPEC-003's eligibility rule; this automation only renumbers what remains afterward.

## Business Rules

- Renumbering is atomic: either every affected milestone's `order` updates, or none do (see the Failure outcome) -- a half-renumbered sequence is never left visible to any viewer.
- Order values stay contiguous starting at 1, with no gaps and no duplicates, at all times after this automation completes.
- This automation never runs against a removal that FEAT-04.SPEC-003 has refused (XBR-10) -- it only renumbers milestones whose add, removal, or move has already been committed.
- Approved or invoiced milestones are still included in renumbering -- their position in the sequence can shift when an earlier milestone is added, removed, or reordered -- even though their other fields (price, name) remain locked by FEAT-04.SPEC-003; position is not "pricing or content," so XBR-10's edit lock does not extend to it.

## Edge Cases

- **Adding a milestone at the very end of an already-populated list** -- Only the new milestone receives an `order` write; no existing milestone's order changes.
- **Removing the first milestone in the list** -- Every remaining milestone shifts down by one position; the milestone that was second becomes first.
- **Reordering a milestone to its own current position (a no-op drag)** -- No `order` values change and no write occurs.
- **Reordering an Approved milestone's position** -- Allowed: only its `order` field changes; its name, price, and other locked fields remain untouched and its Approved status is unaffected.
- **Two milestones are added from two open tabs of the same project in rapid succession** -- Each add is processed as a discrete trigger against the milestone list as it existed the moment that add committed; the second add's renumbering includes the milestone the first add just inserted, so the final sequence remains contiguous with both milestones present at distinct positions.
- **Concurrent trigger firing (an add and a removal committed at effectively the same time)** -- Each trigger's renumbering runs against the milestone list state as of its own commit; the trigger that computes second reads the list including the first trigger's already-applied change, so the final order reflects both changes with no gap or duplicate.
- **A trigger fires while a previous run is still in flight** (e.g., Nadia reorders a milestone again before the prior reorder's renumbering finished writing) -- The second reorder is held until the first's writes complete, then runs against the now-current order; FEAT-04.SPEC-001's reorder controls show a brief in-progress state that prevents a third overlapping reorder from being issued in the meantime.
- **The recalculation fails partway through writing** -- All partial writes for that run are rolled back so no milestone is left with an inconsistent `order`; the non-blocking warning appears and a manual refresh re-fetches the current, still-consistent order.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | Triggered by (inbound) | Fires on a milestone add, removal, or manual reorder committed from this screen |
| FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | Affects (outbound) | Returns the recalculated order for the editor to re-render immediately |
| FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules) | References (inbound) | Only a removal that FEAT-04.SPEC-003 has allowed ever reaches this automation as a trigger |
| FEAT-04.SPEC-002 (Milestone Timeline, Client View) | Affects (outbound) | The client-facing read-only timeline reflects the recalculated order on its next read |

## Analytics and Success Signals

N/A -- no metric in success-metrics.md measures reorder or renumbering behavior; this automation is a structural-integrity mechanism (keeping `order` contiguous) rather than a product outcome tracked in Stage 2. "Milestone Schedule Completeness" (the metric connected to this feature) is driven by the payment-trigger and creation events emitted in FEAT-04.SPEC-001, not by this automation.

## Acceptance Criteria

**FEAT-04.SPEC-004-AC-01:** Given Nadia has three milestones in a project and adds a fourth at the end of the list, when the add commits, then the new milestone receives order 4 and no existing milestone's order changes.

**FEAT-04.SPEC-004-AC-02:** Given Nadia has four milestones and removes the first one, when the removal commits, then the remaining three milestones renumber to positions 1, 2, and 3, each shifted down by one.

**FEAT-04.SPEC-004-AC-03:** Given Nadia drags the third milestone in a five-milestone list to the first position, when the move commits, then the moved milestone becomes position 1 and the milestones that were first and second each shift down by one position.

**FEAT-04.SPEC-004-AC-04:** Given Nadia has exactly one milestone remaining after a removal, when the removal commits, then no order recalculation write occurs, since the single milestone is already at position 1.

**FEAT-04.SPEC-004-AC-05:** Given Nadia's removal attempt on an Approved milestone is refused by FEAT-04.SPEC-003, when the refusal occurs, then this automation never runs and the milestone list is unchanged.

**FEAT-04.SPEC-004-AC-06:** Given a recalculation fails partway through writing the updated order values, when the failure occurs, then all partial writes for that run are rolled back and the non-blocking warning "Milestones saved, but reordering couldn't be completed. Refresh to see the current order." appears.

**FEAT-04.SPEC-004-AC-07:** Given Nadia drags a milestone to the exact position it already occupies, when the drag completes, then no order values change and no write occurs.

**FEAT-04.SPEC-004-AC-08:** Given Nadia reorders a milestone whose status is Approved, when the move commits, then only its order field changes, and its name, price, and Approved status remain unaffected.

**FEAT-04.SPEC-004-AC-09:** Given Nadia adds a milestone from one browser tab while removing a different milestone from another tab of the same project at effectively the same time, when both commit, then the final order for the project's milestones is contiguous with no gap or duplicate.

**FEAT-04.SPEC-004-AC-10:** Given a reorder is already in flight when Nadia issues a second reorder on the same project, when she issues the second move, then it is held until the first's writes complete and the editor shows a brief in-progress state preventing a third overlapping reorder.

**FEAT-04.SPEC-004-AC-11:** Given two milestones are added to the same project from two open tabs in rapid succession, when both adds commit, then the second add's renumbering includes the first add's already-inserted milestone, leaving both milestones at distinct, contiguous positions.

**FEAT-04.SPEC-004-AC-12:** Given a project's milestone list is successfully renumbered after an add, remove, or reorder, when Owen next opens the Milestone Timeline (FEAT-04.SPEC-002), then he sees the milestones in the recalculated order.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (add, remove, manual reorder) | 3 |
| Outcome Paths | 4 (renumbered, no-op, not triggered, failure) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 8 | 8 |
