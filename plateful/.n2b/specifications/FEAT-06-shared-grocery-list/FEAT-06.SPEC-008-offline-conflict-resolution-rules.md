---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-06.SPEC-008
spec_name: Offline Conflict Resolution Rules
spec_slug: offline-conflict-resolution-rules
parent_feature: FEAT-06
parent_feature_name: Shared Grocery List
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 8
acceptance_criteria_count: 10
---

# Logic/Rule Spec: Offline Conflict Resolution Rules

## Overview

**Name:** Offline Conflict Resolution Rules
**ID:** FEAT-06.SPEC-008
**Type:** Logic/Rule
**Purpose:** Governs how offline and concurrent changes to grocery list items resolve on sync -- idempotent ticks, last-write-wins quantity edits, and no duplicate lines on reconnect.
**Parent Feature:** FEAT-06 -- Shared Grocery List
**Governed Entity:** Grocery List Item

## Scope and Non-Goals

**In Scope:**
- Resolving two or more tick/untick actions on the same item made offline or concurrently
- Resolving two or more quantity edits on the same item made offline or concurrently
- Ensuring an offline manual add does not create a duplicate line on reconnect
- Ordering queued offline changes by original event time, not sync-arrival time

**Non-Goals:**
- The transport mechanism that delivers changes between devices -- owned by FEAT-06.SPEC-005 (Live Grocery List Sync); this spec defines only how conflicting changes resolve once delivered
- Manual item name validation and the duplicate-merge matching rule itself -- owned by FEAT-06.SPEC-007; this spec only states that reconnect sync defers to it rather than creating a second line
- General access to view or act on the list -- owned by FEAT-06.SPEC-009; this spec assumes access is already established and governs only how conflicting writes from already-authorized members resolve

## Governed Entity

**Entity:** Grocery List Item
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| ingredient_name | text | The item's name; used to detect duplicate lines on reconnect (governed in detail by FEAT-06.SPEC-007) |
| quantity_and_unit | text | The item's quantity; subject to last-write-wins resolution on concurrent edits |
| aisle | text | The item's aisle category; not independently editable, so not subject to conflict resolution on its own |
| origin | enum | Plan-derived or manual; not changed by conflict resolution |
| ticked | boolean | Whether the item is ticked; subject to idempotent-tick resolution |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-005 | Live Grocery List Sync | On every inbound change event and on every reconnect-after-offline sync |
| FEAT-06.SPEC-001 | Grocery List | Reflects the resolved state after conflict resolution runs; does not resolve conflicts itself |
| FEAT-06.SPEC-002 | Grocery List Generation & Recalculation | Interacts with these rules when a recalculation and a queued offline edit target the same line (see Edge Cases) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| ingredient_name | No validation beyond data type -- governed by FEAT-06.SPEC-006 (plan-derived) or FEAT-06.SPEC-007 (manual); this spec defines only conflict resolution for concurrent writes | Always | -- | -- | -- |
| quantity_and_unit | No validation beyond data type -- format rules live in FEAT-06.SPEC-001/FEAT-06.SPEC-007; this spec defines only how concurrent edits to this field resolve | Always | -- | -- | -- |
| aisle | No validation beyond data type -- not independently editable by a member | Always | -- | -- | -- |
| origin | No validation beyond data type -- not changed by conflict resolution | Always | -- | -- | -- |
| ticked | No validation beyond data type -- this spec defines how concurrent tick/untick actions resolve | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Idempotent tick | ticked | A tick or untick action applied to a line already in the target state has no additional effect once synced -- ticking an already-ticked line, or two members ticking the same line while both offline, results in exactly one ticked state, not a double-toggle | N/A -- not an error; the repeated action is silently absorbed |
| Last-write-wins quantity edit | quantity_and_unit | When two quantity edits to the same line occur before either has synced, the edit with the later original event time is the one that persists once both reach the shared list; the earlier edit is discarded without notifying either member | N/A -- not an error; resolved silently per the dependency map's Contention note for Grocery List Item |
| No duplicate lines on reconnect | ingredient_name, origin | A manual item added offline that matches an ingredient already present on the list (added by another member online in the interim) merges per FEAT-06.SPEC-007's duplicate-merge rule at sync time, rather than creating a second line | N/A -- not an error; merge behavior deferred to FEAT-06.SPEC-007 |
| Deletion precedence | ingredient_name | If one member deletes a line while another member has a queued offline edit (tick or quantity change) targeting the same line, the deletion takes precedence: the item stays removed and the queued edit is discarded silently, since removal reflects a conscious decision that the item is no longer needed | N/A -- not an error; resolved silently |

## Authorization Rules

{Conflict resolution applies uniformly to every member who already has write access to the Grocery List; it introduces no additional authorization surface. Full authorization detail is governed by FEAT-06.SPEC-009.}

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Have a queued write reconciled | Maya, Sam, Jordan (older kid, limited login -- Later) | Always, for any member who already holds Grocery List write access per FEAT-06.SPEC-009 | -- |
| Have a queued write reconciled | Riley (Operator, support) | Never | Riley has no write access to the list (FEAT-06.SPEC-009) and therefore triggers no conflicts to resolve |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| event_time (internal ordering value) | Set to the device's local time at the moment of the action; used only to order and resolve concurrent writes to the same line | On every write | No -- not user-facing or overridable |

## Business Rules

- ASMP-25: offline changes sync without creating duplicates or conflicts left unresolved.
- Per the dependency map's Contention note for Grocery List: recalculation (FEAT-06.SPEC-002) and manual/offline edits merge -- plan-derived lines are recomputed while manual items, ticks, and "already have it" marks are preserved, consistent with this spec's resolution rules.
- When two event times are equal (down to the resolution available), the tiebreak is the order in which the changes are received by the shared list -- deterministic, but not itself a further conflict-resolution rule the household is exposed to.

## Edge Cases

- **Two edits to the same line carry identical event times** -- Resolved by order of arrival at the shared list as a deterministic tiebreak; the household sees one final, consistent value either way.
- **A member ticks an item and then immediately edits its quantity while offline, both queued** -- Both apply on reconnect, since they touch different fields; the item ends up ticked with the new quantity, with no conflict between them.
- **A line is deleted by one member while another has a queued quantity edit for it, made offline** -- The deletion wins per the Deletion Precedence rule; the queued edit is discarded silently and the item stays removed.
- **A recalculation (FEAT-06.SPEC-002) rewrites a plan-derived line's contributing dinners while a device has a queued offline tick on that same line** -- If the line (same ingredient) still exists after recalculation, the queued tick still applies; if recalculation removed that exact line because it is no longer needed, the queued tick is discarded, since there is nothing left to tick.
- **A member is removed from the household (FEAT-18) while they have offline changes queued** -- Those queued changes are discarded rather than applied on reconnect, consistent with FEAT-18's removal cascade; a removed member's device can no longer write to a list they no longer have access to.

## Acceptance Criteria

**FEAT-06.SPEC-008-AC-01:** Given Maya ticks an already-ticked item while briefly offline, when the tick syncs, then the item remains ticked with no double-toggle.

**FEAT-06.SPEC-008-AC-02:** Given two devices both tick the same item while both are offline, when both reconnect, then the item settles into a single ticked state.

**FEAT-06.SPEC-008-AC-03:** Given Maya and Sam both edit the same item's quantity before either syncs, when both changes reach the shared list, then the edit with the later original event time persists and the earlier one is discarded silently.

**FEAT-06.SPEC-008-AC-04:** Given Sam adds "bananas" offline while Maya has already added "bananas" online in the interim, when Sam reconnects, then the two merge into one line per FEAT-06.SPEC-007 rather than creating a duplicate.

**FEAT-06.SPEC-008-AC-05:** Given Maya deletes an item while Sam has a queued offline quantity edit for the same item, when Sam reconnects, then the item stays deleted and Sam's queued edit is discarded silently.

**FEAT-06.SPEC-008-AC-06:** Given Sam ticks an item and edits its quantity while offline, when he reconnects, then both changes apply and the item is ticked with the new quantity.

**FEAT-06.SPEC-008-AC-07:** Given a recalculation removes a plan-derived line while a device has a queued offline tick on that exact line, when the device reconnects, then the queued tick is discarded, since the line no longer exists.

**FEAT-06.SPEC-008-AC-08:** Given a recalculation updates but does not remove a plan-derived line while a device has a queued offline tick on it, when the device reconnects, then the queued tick still applies to the updated line.

**FEAT-06.SPEC-008-AC-09:** Given Jordan (older kid, limited login) has queued offline changes when his access to the household is still active, when he reconnects, then his changes reconcile normally like any other member with write access.

**FEAT-06.SPEC-008-AC-10:** Given a member is removed from the household while they have offline changes queued, when their device later reconnects, then those queued changes are discarded rather than applied.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 2 | 2 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
