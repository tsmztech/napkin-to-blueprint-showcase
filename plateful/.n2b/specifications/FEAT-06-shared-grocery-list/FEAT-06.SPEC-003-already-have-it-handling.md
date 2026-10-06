---
document_type: spec
spec_type: automation
spec_id: FEAT-06.SPEC-003
spec_name: "Already Have It" Handling
spec_slug: already-have-it-handling
parent_feature: FEAT-06
parent_feature_name: Shared Grocery List
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Automation Spec: "Already Have It" Handling

## Overview

**Name:** "Already Have It" Handling
**ID:** FEAT-06.SPEC-003
**Type:** Automation
**Purpose:** Removes a plan-derived item from the grocery list and, for members with pantry access, logs it to the pantry in the same tap.
**Parent Feature:** FEAT-06 -- Shared Grocery List

## Scope and Non-Goals

**In Scope:**
- Removing a plan-derived Grocery List Item when a member marks it "already have it"
- Creating a Pantry Item for members whose Pantry Input access is Full
- Skipping pantry creation entirely for members whose Pantry Input access is None

**Non-Goals:**
- "Already have it" on a manual item -- not offered; manual items are removed directly via FEAT-06.SPEC-001's Remove action, since the household chose to add them and can choose to remove them the same way
- Pantry Item merge and clearing logic beyond creation -- owned by Pantry-Aware Suggestions (FEAT-05), which this automation defers to for how a duplicate pantry entry is handled
- Determining who may see or act on the Grocery List at all -- owned by FEAT-06.SPEC-009; this automation only differentiates the pantry-logging outcome once access is already established

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household member taps "Already have it" on a plan-derived item | FEAT-06.SPEC-001 (Grocery List) | The tapped item's `origin` is "plan-derived"; the member already has Grocery List access per FEAT-06.SPEC-009 | The item's `ingredient_name`, `quantity_and_unit`; the requesting member's role and Pantry Input access level |

## Processing Logic

1. Confirm the tapped item's `origin` is "plan-derived" (the control is not offered on manual items).
2. Remove the Grocery List Item from the list immediately.
3. Check the requesting member's Pantry Input access level per the Access Matrix (user-persona.md).
4. If Pantry Input is Full (Maya, Sam): create a Pantry Item with `item_name` set to the removed item's `ingredient_name`, `added_by` set to the requesting member, and `status` set to Active, per XBR-04. If a matching Pantry Item already exists, Pantry-Aware Suggestions' (FEAT-05) merge behavior applies rather than creating a second entry.
5. If Pantry Input is None (Jordan, older kid, limited login -- Later): skip pantry creation entirely; only the list removal occurs.
6. Hand off to FEAT-06.SPEC-005 (Live Grocery List Sync) to propagate the removal to every household device within seconds.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Removed and logged to pantry | Requesting member's Pantry Input access is Full | Grocery List Item deleted; Pantry Item created (or merged into an existing one) | Toast: "Marked as already have it -- added to your pantry." | FEAT-06.SPEC-001, FEAT-05 |
| Removed, no pantry entry | Requesting member's Pantry Input access is None | Grocery List Item deleted only | Toast: "Marked as already have it." | FEAT-06.SPEC-001 |
| Pantry write fails after list removal | Grocery List Item deletion succeeds but the pantry write cannot complete after background retries | Grocery List Item stays deleted; no Pantry Item created | Toast: "Removed from list. Couldn't add to your pantry -- try adding it there directly." | FEAT-06.SPEC-001, FEAT-05 |

## Data Model

**Reads:** Grocery List Item -- `ingredient_name`, `origin`. Member Profile -- role, to determine Pantry Input access level.

**Updates:** None beyond the delete below.

**Deletes:** Grocery List Item -- the marked item, unconditionally.

**Creates:** Pantry Item -- `item_name`, `added_by`, `status`, only when the requesting member's Pantry Input access is Full.

## Business Rules

- XBR-04: logged pantry items are left off the week's list, and marking a list item "already have it" can add it to the pantry in the same tap; only members with Full Pantry Input access trigger the pantry write.
- Grocery List Item removal is unconditional and immediate regardless of pantry-logging outcome -- a pantry write failure never leaves the item back on the list.
- Eligibility to act on this automation at all is governed by FEAT-06.SPEC-009; this spec only differentiates the pantry outcome once access is already confirmed.

## Edge Cases

- **Concurrent trigger firing (two members tap "Already have it" on the same item at effectively the same time)** -- The first tap removes the item and (if eligible) creates the pantry entry; the second tap targets an item that no longer exists and is a silent no-op -- no duplicate pantry entry is created and no error is shown.
- **Trigger fires while a previous run is in flight for the same item** -- Because live sync (FEAT-06.SPEC-005) removes the item from other devices as soon as the first tap is processed, a second tap on the same item in practice arrives after the item is already gone and is handled by the concurrent-trigger case above; two different items processed at the same time run independently with no queuing between them.
- **Member marks "already have it" while offline** -- The removal (and, where eligible, the pantry creation) queues locally per FEAT-06.SPEC-005 and FEAT-06.SPEC-008, and applies once connectivity returns.
- **A duplicate Pantry Item already exists for the same ingredient** -- Pantry-Aware Suggestions' (FEAT-05) merge rule applies: the two become one entry rather than two, per the dependency map's Contention note for Pantry Item.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-001 (Grocery List) | Triggered by (inbound) | "Already have it" overflow action fires this automation |
| FEAT-06.SPEC-001 (Grocery List) | Affects (outbound) | The item disappears from the screen on success |
| FEAT-06.SPEC-005 (Live Grocery List Sync) | Triggers (outbound) | Propagates the removal to every household device |
| FEAT-05 (Pantry-Aware Suggestions) | Affects (outbound) | Creates or merges a Pantry Item for eligible roles |
| FEAT-06.SPEC-009 (Grocery List Access & Authorization Rules) | References (inbound) | Governs eligibility to act on this automation at all |

## Analytics and Success Signals

- **grocery_item_already_have** (role, pantry_logged: yes/no) -- supports success-metrics.md: "Grocery List Live-Update Trust"
- **grocery_item_already_have_pantry_write_failed** (reason) -- N/A -- success-metrics.md defines no metric for pantry-write reliability specifically; retained to observe how often the non-blocking pantry-write guarantee is exercised

## Acceptance Criteria

**FEAT-06.SPEC-003-AC-01:** Given Maya opens the overflow menu on a plan-derived "spinach" line, when she taps "Already have it", then the item is removed from the list and a Pantry Item "spinach" is created with Maya as `added_by`.

**FEAT-06.SPEC-003-AC-02:** Given Sam taps "Already have it" on a plan-derived "feta" line, when the action completes, then the item is removed and he sees the toast "Marked as already have it -- added to your pantry."

**FEAT-06.SPEC-003-AC-03:** Given Jordan (older kid, limited login, Pantry Input: None) taps "Already have it" on a plan-derived item, when the action completes, then the item is removed from the list, no Pantry Item is created, and he sees the toast "Marked as already have it."

**FEAT-06.SPEC-003-AC-04:** Given the pantry write fails after background retries for a Full-access member's "already have it" action, when the failure is final, then the toast "Removed from list. Couldn't add to your pantry -- try adding it there directly." appears and the item stays removed from the list.

**FEAT-06.SPEC-003-AC-05:** Given a matching Pantry Item already exists when Maya taps "Already have it" on the same ingredient, when the pantry write runs, then the two entries merge into one per FEAT-05's merge rule rather than creating a duplicate.

**FEAT-06.SPEC-003-AC-06:** Given two household members both tap "Already have it" on the same item at effectively the same time, when both taps are processed, then the item is removed once, at most one Pantry Item is created, and the second tap has no additional effect.

**FEAT-06.SPEC-003-AC-07:** Given Sam taps "Already have it" while offline, when he is offline, then the removal (and pantry creation) queue locally and apply automatically once connectivity returns.

**FEAT-06.SPEC-003-AC-08:** Given Maya taps "Already have it" on an item, when the removal completes, then the change propagates to every other household device within seconds via FEAT-06.SPEC-005.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |
