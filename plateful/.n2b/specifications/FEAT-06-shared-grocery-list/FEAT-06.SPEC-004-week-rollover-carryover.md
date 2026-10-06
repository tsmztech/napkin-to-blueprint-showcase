---
document_type: spec
spec_type: automation
spec_id: FEAT-06.SPEC-004
spec_name: Week Rollover & Carryover
spec_slug: week-rollover-carryover
parent_feature: FEAT-06
parent_feature_name: Shared Grocery List
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Automation Spec: Week Rollover & Carryover

## Overview

**Name:** Week Rollover & Carryover
**ID:** FEAT-06.SPEC-004
**Type:** Automation
**Purpose:** Archives the current week's grocery list at week end and carries unticked manually-added items into the new week's list.
**Parent Feature:** FEAT-06 -- Shared Grocery List

## Scope and Non-Goals

**In Scope:**
- Archiving the current week's Grocery List when the week ends
- Identifying unticked manual items on the archived list
- Handing those items to FEAT-06.SPEC-002 to seed the new week's list

**Non-Goals:**
- Deciding when a household's "week" boundary falls -- derived from the household's plan cycle (owned by FEAT-01/FEAT-16); this automation only reacts to that boundary being reached
- Creating the new week's plan-derived list content -- owned by FEAT-06.SPEC-002; this automation supplies only the carried manual items
- Making the archived list browsable -- owned by Weekly Plan History (FEAT-19); this automation only sets the archived status the browsing feature reads
- Purging any archived list or item -- excluded per scope-boundaries.md SC-18: archived lists and their items are retained for the life of the household account with no automatic purge

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| The household's current week ends | system (aligned to the household's plan-arrival cycle, owned by FEAT-01/FEAT-16) | Fires once per household per week, at the boundary between the ending week and the next | The current Active Grocery List and its Items |

## Processing Logic

1. Identify the household's current Active Grocery List for the ending week.
2. Set the list's `status` to Archived. It is retained in full, never hard-deleted, per SC-18.
3. Identify every Grocery List Item on the archived list whose `origin` is "manual" and `ticked` is false.
4. Pass those items' `ingredient_name`, `quantity_and_unit`, `aisle`, and originating member ("added by") to FEAT-06.SPEC-002 as carryover input for the new week's list.
5. Leave every ticked manual item and every plan-derived item on the archived list only -- they are not carried forward.
6. Once FEAT-06.SPEC-002 creates or has already created the new week's Grocery List, the carried items appear on it as manual-origin lines with `ticked` reset to false.
7. Make the archived list available for browsing through Weekly Plan History (FEAT-19).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Rollover with carryover | One or more unticked manual items exist on the ending week's list | Current list archived; new week's list seeded with carried manual items | New list shows the carried items alongside plan-derived generation, with no separate notification (silent, per this feature's Communications) | FEAT-06.SPEC-001, FEAT-06.SPEC-002 |
| Rollover, nothing to carry | No unticked manual items exist on the ending week's list | Current list archived only | New list starts from plan-derived generation alone | FEAT-06.SPEC-001, FEAT-06.SPEC-002 |
| Rollover ahead of the new plan | The new week's plan does not exist yet when rollover fires | Current list archived; carried items held pending until FEAT-06.SPEC-002 creates the new list | No visible list until the new week's plan or first pick exists; carried items appear once it does | FEAT-06.SPEC-002 |
| Archive step fails | The archive write cannot complete | The current list remains Active and visible rather than disappearing | No visible change to the household; rollover retries on the next scheduled attempt | FEAT-06.SPEC-001 |

## Data Model

**Reads:** Grocery List Item -- `origin`, `ticked`, `ingredient_name`, `quantity_and_unit`, `aisle`, added-by attribution, for the ending week's list.

**Updates:** Grocery List -- `status` (Active -> Archived).

**Creates:** Grocery List Item (on the new week's list, via FEAT-06.SPEC-002) -- carried copies of qualifying manual items.

**Deletes:** None -- archiving is a status change, not a deletion; no item is removed by this automation.

## Business Rules

- SC-18: archived lists and their items are retained for the life of the household account, browsable after a downgrade to the free tier, with no purge policy while the account is active.
- Archiving has no cascade to other entities -- the archived list's items stay attached to it exactly as they were at archive time, except for the carried copies created fresh on the new list.
- This is the sole Archive path for Grocery List; no other spec transitions a list to Archived.

## Edge Cases

- **Concurrent trigger firing for the same week boundary** -- Rollover cannot fire twice for the same week: a list already Archived is never re-archived, so a duplicate trigger for the same boundary is a no-op against the already-archived list.
- **Trigger fires while a previous rollover is in flight** -- The next week's rollover cannot begin until the current one completes, since a household has exactly one Active list at a time; rollover runs are naturally serialized by that invariant.
- **A manual item is added in the final moments before rollover** -- It is still evaluated by the unticked-manual-item check and carried forward if it qualifies at the moment rollover runs.
- **A household has no grocery-list activity that week** -- Rollover still archives the (empty or near-empty) list; there is simply nothing to carry.
- **A manual item is ticked by one member at the exact moment rollover evaluates it** -- Whichever state the item is in when the evaluation step (Step 3) reads it determines whether it carries forward; there is no separate reconciliation after rollover has run for that week.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-002 (Grocery List Generation & Recalculation) | Triggers (outbound) | Supplies carried-over manual items to seed the new week's list |
| FEAT-06.SPEC-001 (Grocery List) | Affects (outbound) | The new list, including carried items, appears there |
| FEAT-19 (Weekly Plan History) | Affects (outbound) | The archived list becomes browsable there |

## Analytics and Success Signals

- **grocery_items_carried_over** (count) -- N/A -- success-metrics.md defines no metric for week-to-week carryover; retained to monitor how often unticked manual items persist across weeks
- **grocery_list_archived** (item_count) -- N/A -- success-metrics.md defines no metric for archive volume; retained as an operational signal that rollover is completing on schedule

## Acceptance Criteria

**FEAT-06.SPEC-004-AC-01:** Given Maya's household has an unticked manual item "birthday candles" on the current week's list, when the week ends, then the list is archived and "birthday candles" appears on the new week's list as a manual-origin item.

**FEAT-06.SPEC-004-AC-02:** Given a manual item "milk" was ticked before the week ended, when rollover runs, then "milk" stays on the archived list only and does not appear on the new week's list.

**FEAT-06.SPEC-004-AC-03:** Given a household's current week's list has no unticked manual items, when the week ends, then the list is archived and the new week's list starts from plan-derived generation alone.

**FEAT-06.SPEC-004-AC-04:** Given the new week's plan does not exist yet when rollover fires, when rollover completes, then the current list is archived and carried items are held pending until the new week's plan or first pick creates the new list.

**FEAT-06.SPEC-004-AC-05:** Given the archive step fails for a household, when the failure occurs, then the current week's list remains Active and visible, and rollover retries on the next scheduled attempt.

**FEAT-06.SPEC-004-AC-06:** Given a household's list is already Archived for a week boundary, when a duplicate rollover trigger fires for the same boundary, then no further change occurs to that list.

**FEAT-06.SPEC-004-AC-07:** Given a household's rollover for the current week is still in flight, when the next week's boundary is reached before it completes, then the next rollover waits until the in-flight one finishes.

**FEAT-06.SPEC-004-AC-08:** Given an archived list exists for a past week, when a household member opens Weekly Plan History (FEAT-19), then the archived list and its items are visible there in full.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
