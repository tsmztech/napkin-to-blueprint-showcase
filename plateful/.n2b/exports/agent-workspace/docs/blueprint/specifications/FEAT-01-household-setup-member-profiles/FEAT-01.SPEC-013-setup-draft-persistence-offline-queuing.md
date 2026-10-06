---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-01.SPEC-013
spec_name: Setup Draft Persistence & Offline Queuing
spec_slug: setup-draft-persistence-offline-queuing
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 7
acceptance_criteria_count: 9
---

# Logic/Rule Spec: Setup Draft Persistence & Offline Queuing

## Overview

**Name:** Setup Draft Persistence & Offline Queuing
**ID:** FEAT-01.SPEC-013
**Type:** Logic/Rule
**Purpose:** Governs how every setup screen saves drafts locally and resumes exactly where the organiser left off, whether online or offline.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles
**Governed Entity:** Setup draft state (a cross-cutting behavior, not itself a Shared Data Entity from the dependency map -- it governs in-progress, not-yet-confirmed input on Household, Member Profile, and Dietary Rule fields across FEAT-01.SPEC-003, 004, 005, 006, and 008)

## Scope and Non-Goals

**In Scope:**
- Saving in-progress form input locally as the organiser types, across every setup screen
- Resuming a setup screen exactly where it was left, after navigating away, closing the app, or losing connectivity
- Queuing a save made while offline and submitting it automatically once connectivity returns
- Failure behavior when a save cannot complete: preserving entered data and offering retry

**Non-Goals:**
- The offline behavior of screens outside this feature (e.g., Shared Grocery List's offline ticking) -- each feature owning offline-capable screens defines its own queuing behavior; this spec governs only FEAT-01's setup screens
- Conflict resolution between two devices editing the same confirmed (already-saved) record -- that is each screen's own concurrent-edit handling (e.g., FEAT-01.SPEC-003's Edge Cases), which is distinct from an in-progress, not-yet-saved draft
- Removing or expiring a draft that was never submitted -- product-features.md defines no automatic draft cleanup; a draft persists locally until it is either submitted or explicitly discarded by the organiser (e.g., tapping "Cancel" with confirmation)

## Governed Entity

**Entity:** Setup draft state (cross-cutting; applies to in-progress input on Household, Member Profile, and Dietary Rule fields)
**Source:** Feature Breakdown Brief, States field ("Offline-degraded: setup can be drafted offline and is held locally until connectivity returns to save") and Side-Effect Inventory ("Organiser closes the app mid-setup, with or without connectivity -> Draft is held locally; setup resumes exactly where it was left off when reopened")

| Field | Data Type | Description |
|-------|-----------|-------------|
| draft_screen | enum | Which setup screen the draft belongs to (FEAT-01.SPEC-003, 004, 005, 006, or 008) |
| draft_target | text | The specific record the draft applies to (e.g., which Member Profile, for FEAT-01.SPEC-005/006) |
| draft_fields | derived | The in-progress field values entered but not yet confirmed/saved |
| queued_operation | boolean | Whether this draft represents a save attempted while offline, awaiting submission |
| last_updated | date | When the draft was last modified, used to determine which of two devices' drafts is current if both exist |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-003 | Household Naming & Guided Setup Start | On every field change (draft save); on screen re-entry (draft restore); on Continue/Save while offline (queuing) |
| FEAT-01.SPEC-004 | Member List & Add Member | On screen re-entry, to reflect any queued member additions still pending sync |
| FEAT-01.SPEC-005 | Member Profile Detail | On every field change; on screen re-entry; on Save while offline |
| FEAT-01.SPEC-006 | Dietary Rules Editor | On every field change within the add/edit rule panel; on screen re-entry; on rule save while offline |
| FEAT-01.SPEC-008 | Weekly Budget & Schedule Setup | On every field change; on screen re-entry; on Continue/Save while offline |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| draft_fields | No validation beyond data type -- a draft may hold partial or invalid values, since validation applies only at actual submission, not at draft-save time | Always | -- | -- | No |
| queued_operation | Must resolve to submitted or discarded -- a queued operation cannot remain indefinitely undecided once connectivity returns | When connectivity is restored | On reconnection | N/A -- resolution is automatic (see Business Rules), not a user-facing error | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Draft freshness on multi-device conflict | draft_fields, last_updated | If a draft exists locally on Device A and the same screen/target has also been saved (confirmed) from Device B since the local draft's last_updated, the confirmed save from Device B wins when Device A's screen reopens -- the stale local draft is discarded in favor of the current confirmed record, and Device A's user starts fresh from the confirmed state, not from their stale draft | N/A -- resolved silently, since the confirmed record is public state and always takes precedence over an unconfirmed local draft |

## Authorization Rules

{Draft persistence is a local, per-device, per-user mechanism -- there is no cross-role authorization surface here: a draft is always private to the device and account that created it, and the actions it governs (saving, resuming, queuing) inherit the authorization already defined by the screen the draft belongs to.}

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create/update a local draft | Maya (Organiser) | On any screen she is authorized to edit per FEAT-01.SPEC-016 | -- |
| Resume a local draft | Maya (Organiser) | Only on the same device/session where the draft was created; a draft never syncs to a different device before it is submitted | Opening the same setup screen on a different device shows the screen in its confirmed (last-saved) state, not the unsynced draft -- there is no error, since this is expected behavior, not a denial |
| Queue an offline save | Maya (Organiser) | Only for screens/actions she is authorized to perform per FEAT-01.SPEC-016 | An action Maya is not authorized to perform (e.g., Sam attempting an organiser-only edit) is blocked by FEAT-01.SPEC-016 before any draft or queue state is created |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| draft_screen | Set to the screen currently being edited | On first field change on any governed screen | No -- set automatically |
| last_updated | Current date and time | On every draft field change | No -- always current |
| queued_operation | False, unless a save is attempted while offline | On save attempt | No -- set automatically based on connectivity at save time |

## Business Rules

- A draft is created locally the moment the organiser begins typing or selecting on any governed screen -- it does not wait for a deliberate "save draft" action.
- Reopening a governed screen (via back navigation, closing and reopening the app, or after a session expiry per each screen's own Access and Visibility row) restores the most recent local draft for that screen/target, if one exists and no newer confirmed save has superseded it.
- A save attempted while offline is queued locally rather than failing outright; it submits automatically the moment connectivity returns, with no further action required from the organiser.
- A save that fails for a reason other than connectivity (e.g., a validation error surfaced by the server) is never silently discarded -- the entered data remains on screen with a retry option, per this feature's States field commitment ("a failed save keeps the entered data on screen and offers a retry, never silently discards input").
- Once a queued offline save submits successfully, the local draft for that screen/target is cleared, since the data is now part of the confirmed record.

## Edge Cases

- **Organiser closes the app mid-entry on FEAT-01.SPEC-005 with no connectivity at all** -- The draft is held locally; reopening the app (online or still offline) restores the exact field values as they were left.
- **Organiser is offline, queues a save on FEAT-01.SPEC-008, then edits the same fields again before connectivity returns** -- The queued save is updated in place to reflect the latest edit; only one submission occurs once connectivity returns, not one per edit.
- **Organiser signs in on a second device while a draft exists, unsynced, on the first device** -- The second device shows the last confirmed state only; the first device's draft remains local to it and is not visible elsewhere until it is submitted.
- **Connectivity returns and submits a queued save, but the underlying record was deleted or changed in a way that makes the queued save invalid** (e.g., the member being edited was removed via FEAT-18 while the device was offline) -- The queued submission fails with a clear message on next screen open: "This member no longer exists. Your changes could not be saved." rather than silently applying to a stale target.
- **Two queued saves for the same screen/target exist because the organiser used the app offline on two devices** -- On reconnection, each device submits independently; the dependency map's per-entity Contention resolution (last-write-wins for Household and Member Profile fields) determines the final state, consistent with those entities' own Contention notes rather than any special rule introduced here.
- **Organiser explicitly cancels an in-progress entry (e.g., taps "Cancel" with confirmation on FEAT-01.SPEC-005)** -- The local draft for that screen/target is discarded immediately; reopening the screen afterward starts fresh from the last confirmed state, not from the discarded draft.

## Acceptance Criteria

**FEAT-01.SPEC-013-AC-01:** Given Maya is entering a new member's name on FEAT-01.SPEC-005 and closes the app, when she reopens it and returns to that screen, then the entered name is still there exactly as she left it.

**FEAT-01.SPEC-013-AC-02:** Given Maya loses connectivity while filling in the weekly budget on FEAT-01.SPEC-008, when she taps "Continue", then the entry is queued locally and the banner confirms it will save once she reconnects.

**FEAT-01.SPEC-013-AC-03:** Given Maya's queued budget save is pending and connectivity returns, when the app detects the reconnection, then the save submits automatically with no action required from Maya.

**FEAT-01.SPEC-013-AC-04:** Given Maya's save fails due to a validation error rather than connectivity, when the failure occurs, then her entered data remains on screen with a retry option, never silently discarded.

**FEAT-01.SPEC-013-AC-05:** Given Maya has a local, unsynced draft on Device A, when she signs in on Device B and opens the same screen, then Device B shows only the last confirmed state, not Device A's draft.

**FEAT-01.SPEC-013-AC-06:** Given Maya is offline and queues a save, then edits the same fields again before reconnecting, when connectivity returns, then only one submission occurs, reflecting her latest edit.

**FEAT-01.SPEC-013-AC-07:** Given a queued save's target member was removed (FEAT-18) while Maya was offline, when connectivity returns and the queued save attempts to submit, then she sees "This member no longer exists. Your changes could not be saved." rather than the save silently applying incorrectly.

**FEAT-01.SPEC-013-AC-08:** Given Maya taps "Cancel" with confirmation on an in-progress member edit, when she confirms discarding, then the local draft is cleared and reopening the screen shows the last confirmed state.

**FEAT-01.SPEC-013-AC-09:** Given Maya has queued the same field's save from two different offline devices, when both reconnect, then the field resolves to the most recently saved value, consistent with the Household/Member Profile entities' own last-write-wins Contention rule.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
