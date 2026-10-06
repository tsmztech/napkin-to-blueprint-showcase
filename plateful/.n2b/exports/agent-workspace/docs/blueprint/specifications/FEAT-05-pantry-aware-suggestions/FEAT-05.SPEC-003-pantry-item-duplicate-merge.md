---
document_type: spec
spec_type: automation
spec_id: FEAT-05.SPEC-003
spec_name: Pantry Item Duplicate Merge
spec_slug: pantry-item-duplicate-merge
parent_feature: FEAT-05
parent_feature_name: Pantry-Aware Suggestions
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Pantry Item Duplicate Merge

## Overview

**Name:** Pantry Item Duplicate Merge
**ID:** FEAT-05.SPEC-003
**Type:** Automation
**Purpose:** Merges a newly logged or synced Pantry Item into an existing Active entry with the same name instead of creating a second row.
**Parent Feature:** FEAT-05 -- Pantry-Aware Suggestions

## Scope and Non-Goals

**In Scope:**
- Comparing a submitted or synced item name against the household's existing Active Pantry Items
- Merging into the existing entry when a match is found, on direct add, offline-sync reconciliation, and the inbound FEAT-06 "already have it" create path
- Idempotent behavior so a double submission of the same name never creates two rows

**Non-Goals:**
- Validating the item name's format or length -- handled by FEAT-05.SPEC-004 (Pantry Item Field Validation), which runs before this automation ever sees the name
- Fuzzy or semantic matching across different wordings of the same ingredient (e.g., "spinach" vs. "baby spinach") -- excluded per the Feature Breakdown Brief's own Validation & Limits: item_name is free text with no rigid inventory schema, and the product follows the lighter "tell me what I have" model (scope-boundaries.md, SC-11) rather than a normalized ingredient taxonomy that exact-name matching alone cannot provide
- Merging items across different households -- Pantry Item belongs to exactly one Household per the dependency map; this automation only ever compares entries within the same household

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household member submits a new item on the pantry list | FEAT-05.SPEC-001 (Pantry List & Item Entry) | Fires after the name passes validation (FEAT-05.SPEC-004) | The validated item name, the submitting household, the submitting member |
| An offline-queued add syncs on reconnect | FEAT-05.SPEC-001 (Pantry List & Item Entry) | Fires when connectivity returns and a locally queued add is sent | The validated item name, the household, the member who queued it, the queued timestamp |
| "Already have it" tap logs an item from the grocery list | FEAT-05.SPEC-007 (Pantry Item Off-Grocery-List Exclusion Rule) | Fires when a grocery list line is marked "already have it" by a role with Pantry Input access | The grocery list line's ingredient name, the household, the tapping member |

## Processing Logic

1. Receive the validated item name and the household it belongs to, from whichever trigger fired.
2. Normalize the submitted name for comparison (case-insensitive, leading/trailing whitespace trimmed).
3. Steps 4-6 (read, compare, and create-or-merge) run as one serialized unit keyed on the household plus the normalized name: only one submission for that exact household-and-name pair may be inside this sequence at a time. A second submission for the same household and normalized name that arrives while the first is still inside the sequence waits for the first to finish (create or merge) before it begins its own read, so the two submissions are never comparing against the list at the same moment. Submissions for different names, or for the same name in different households, proceed independently and do not wait on each other.
4. Read every Active Pantry Item currently belonging to that household.
5. Compare the normalized submitted name against each existing Active item's name (also normalized) for an exact match.
6. If exactly one Active item matches, treat this submission as a duplicate of that entry: no new row is created. If no Active item matches, create a new Pantry Item with the submitted name, added_by the submitting member, and status Active.
7. In all cases, the resulting entry (matched or newly created) is what the triggering screen displays as the outcome. Because step 3 serializes the sequence per household and normalized name, this holds even when two submissions for the same name arrive at effectively the same time: the second always finds and merges into the first's result rather than creating a sibling row.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| New entry created | No existing Active item matches the submitted name | A new Pantry Item is created (Active) | The new row appears on the pantry list | FEAT-05.SPEC-001 |
| Merged into existing entry | An existing Active item matches the submitted name | No new row is created; the existing entry's presence on the list is unchanged | The existing row is shown (no visible duplicate); if the merge originated from an offline sync, the pending marker on the local queued item simply clears with no new row appearing | FEAT-05.SPEC-001 |
| Merged via inbound "already have it" | The submitted name (from FEAT-06) matches an existing Active item | No new row is created | The existing pantry row is unchanged; the grocery list line still leaves the list per FEAT-05.SPEC-007 | FEAT-05.SPEC-001, FEAT-05.SPEC-007 |
| Automation failure | The merge comparison cannot complete | No item is saved | The triggering screen's own Error state applies (FEAT-05.SPEC-001: typed item remains visible with a retry option) | FEAT-05.SPEC-001 |

## Data Model

**Reads:** Pantry Item -- item_name, status (Active only), scoped to the submitting household.
**Creates:** Pantry Item -- item_name, added_by, status: Active, when no match is found.
**Updates:** None -- a matched duplicate leaves the existing entry's fields unchanged; added_by remains the original logger, not the later submitter.
**Deletes:** None.

## Business Rules

- Matching is exact-name, case-insensitive, whitespace-trimmed -- there is no partial or fuzzy matching, consistent with item_name's free-text, no-rigid-schema definition (FEAT-05.SPEC-004).
- This automation is the single mechanism that prevents duplicate Active rows for the same name; the dependency map's Contention note for Pantry Item ("the same item added twice becomes one entry") is enforced entirely here, unconditionally -- including when two submissions for the same household and name arrive at effectively the same time, because the check-and-create sequence for a given household-and-normalized-name pair is serialized (Processing Logic, step 3): only one Active entry ever results, never two.
- The automation runs synchronously with the triggering add -- the pantry list screen waits for the merge-or-create result before showing the outcome row.
- A Used/Removed item is never matched against -- re-adding a previously cleared name always creates a fresh Active entry (per the Entity-Lifecycle Coverage Matrix), it does not revive the cleared one.

## Edge Cases

- **Submitted name differs only in capitalization or surrounding whitespace ("Spinach " vs. "spinach")** -- Matches the existing "spinach" entry; merged, no new row.
- **Submitted name differs in wording for the same ingredient ("spinach" vs. "baby spinach")** -- Does not match; a separate Active entry is created, per the Non-Goals (no fuzzy matching).
- **Two household members add the same name from different devices at effectively the same time** -- The check-and-create sequence for that household-and-name pair is serialized (Processing Logic, step 3): whichever submission enters the sequence first runs its read-compare-create and creates the entry; the other submission waits, then runs its own read-compare-create against the now-committed entry and merges into it. Only one Active entry ever results, regardless of which device's request physically arrived first.
- **Offline add later matches an item added by someone else while the device was offline** -- On reconnect, the queued add is compared against the current Active list (which now includes the other member's item); if it matches, the queued add merges and no duplicate appears, per the feature's own Offline-degraded expectation ("sync and reconcile once connectivity returns").
- **Inbound "already have it" tap names an ingredient that already exists as a Pantry Item** -- Merges per Outcome "Merged via inbound 'already have it'"; the grocery list line still leaves the list per FEAT-05.SPEC-007 regardless of merge outcome.
- **Concurrent trigger firing (two adds for different names in the same household fire at effectively the same time)** -- Each name is compared and resolved independently; there is no shared lock between different names.
- **Trigger fires while a previous run is in flight for the same name** -- A second submission of the same name while the first is still inside the serialized sequence (Processing Logic, step 3) does not start its own read until the first run has created or merged its entry. The second run then reads the updated Active list, finds the match, and merges -- no double-create is possible for the same household-and-normalized-name pair.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-001 (Pantry List & Item Entry) | Triggered by (inbound) | Fires on direct add and on offline-sync reconciliation |
| FEAT-05.SPEC-004 (Pantry Item Field Validation) | References (inbound) | The submitted name has already passed this spec's rules before the merge check runs |
| FEAT-05.SPEC-007 (Pantry Item Off-Grocery-List Exclusion Rule) | Triggered by (inbound) | Fires when an "already have it" tap on the grocery list logs an item to the pantry |
| FEAT-05.SPEC-001 (Pantry List & Item Entry) | Affects (outbound) | The merged-or-created row is what the pantry list displays |

## Analytics and Success Signals

- **pantry_item_merged** (merge source: direct add / offline sync / already-have-it) -- N/A -- this event has no directly connected metric in success-metrics.md; it is recorded to confirm the merge guarantee is exercised, feeding no metric this feature is measured by
- **pantry_item_added** (created: true) -- supports success-metrics.md: "Pantry Items Used" (a genuinely new entry contributes to the pool of items a plan can use up)
- **pantry_merge_failed** (trigger source) -- N/A -- a failed merge check falls back to the triggering screen's own error handling and is not itself a success signal for any metric in success-metrics.md

## Acceptance Criteria

**FEAT-05.SPEC-003-AC-01:** Given Maya's household has no Active pantry item named "spinach", when she submits "spinach" on the Pantry List screen, then a new Pantry Item "spinach" is created and shown.

**FEAT-05.SPEC-003-AC-02:** Given Maya's household already has an Active pantry item "spinach", when Sam submits "Spinach " (different capitalization, trailing space), then no new row is created and the existing "spinach" entry is what remains on the list.

**FEAT-05.SPEC-003-AC-03:** Given Maya's household has an Active pantry item "spinach", when Sam submits "baby spinach", then a separate new Pantry Item "baby spinach" is created (no fuzzy match).

**FEAT-05.SPEC-003-AC-04:** Given Sam added "eggs" while offline, when connectivity returns and no household member has added "eggs" in the meantime, then "eggs" syncs as a new Active entry.

**FEAT-05.SPEC-003-AC-05:** Given Sam added "eggs" while offline and Maya separately added "eggs" while he was offline, when Sam's device reconnects and syncs, then his queued "eggs" merges into Maya's existing entry and no duplicate row appears.

**FEAT-05.SPEC-003-AC-06:** Given Maya's household has an Active pantry item "yoghurt", when a grocery list line "yoghurt" is marked "already have it" (FEAT-05.SPEC-007), then no new pantry row is created and the existing "yoghurt" entry is unaffected, while the grocery list line still leaves the list.

**FEAT-05.SPEC-003-AC-07:** Given Maya previously cleared a pantry item named "flour", when she later re-adds "flour", then a fresh Active entry is created rather than any prior cleared record being restored.

**FEAT-05.SPEC-003-AC-08:** Given Maya and Sam each submit "onions" from different devices at effectively the same time, when both submissions are processed, then the check-and-create sequence for that household-and-name pair is serialized so only one Active "onions" entry ever exists: whichever submission enters the sequence second finds and merges into the entry the first one created, with no duplicate row appearing.

**FEAT-05.SPEC-003-AC-09:** Given the merge comparison cannot complete due to a processing error, when Maya submits a new item, then FEAT-05.SPEC-001's Error state applies: the typed item remains visible with a retry option and no item is saved.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |
