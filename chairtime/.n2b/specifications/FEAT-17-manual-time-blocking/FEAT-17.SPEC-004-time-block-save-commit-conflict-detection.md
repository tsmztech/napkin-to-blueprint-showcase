---
document_type: spec
spec_type: automation
spec_id: FEAT-17.SPEC-004
spec_name: Time Block Save Commit & Conflict Detection
spec_slug: time-block-save-commit-conflict-detection
parent_feature: FEAT-17
parent_feature_name: Manual Time Blocking
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Time Block Save Commit & Conflict Detection

## Overview

**Name:** Time Block Save Commit & Conflict Detection
**ID:** FEAT-17.SPEC-004
**Type:** Automation
**Purpose:** Validates and commits a created or edited Time Block, checking it against existing confirmed bookings and routing to the Conflict Review screen when any are found.
**Parent Feature:** FEAT-17 -- Manual Time Blocking

## Scope and Non-Goals

**In Scope:**
- Re-validating the submitted values against FEAT-17.SPEC-008's field rules at save time
- Checking the block's date/time range against every confirmed Booking for the same Pro Account
- Committing the block immediately when no conflict is found
- Routing to FEAT-17.SPEC-003 when one or more conflicts are found, without committing the block first

**Non-Goals:**
- Generating a recurring pattern's future occurrences -- owned by FEAT-17.SPEC-005, which runs each occurrence through this same conflict scope independently
- Resolving a detected conflict -- owned by FEAT-17.SPEC-006, once Talia's per-booking choices are made on FEAT-17.SPEC-003
- Checking against a client's in-progress checkout hold -- that contention is FEAT-03.SPEC-005's responsibility (first-committed-wins between a block save and a client checkout); this automation only checks against already-confirmed Bookings

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia taps Save on a new or edited single-date block | FEAT-17.SPEC-001 (Create/Edit Time Block) | Fires after the screen's own inline field validation passes | Start, end, label, and (for an edit) the existing Time Block's identity |

## Processing Logic

1. Receive the submitted block values (start, end, optional label) from FEAT-17.SPEC-001, and, for an edit, the identity of the existing Time Block record.
2. Re-validate start, end, and label against FEAT-17.SPEC-008's field rules (end must be after start; label within its length limit).
3. If validation fails, return the specific field error to the triggering screen without proceeding further.
4. For an edit, confirm the existing Time Block record still exists and has not been removed by another session; if it no longer exists, report the removed-record outcome.
5. Read every confirmed Booking (state Confirmed or Awaiting Outcome) belonging to the same Pro Account whose scheduled range overlaps the block's start/end range, per FEAT-17.SPEC-008's definition of a conflicting booking.
6. If zero conflicting bookings are found, commit the Time Block record (create or update) and signal success to the triggering screen.
7. If one or more conflicting bookings are found, hold the block's values in a pending state (not yet committed) and hand the full conflicting set, plus the pending block's values, to FEAT-17.SPEC-003 for Talia's review.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Committed, no conflicts | Zero confirmed bookings overlap the block's range | Time Block record created or updated | Talia is returned to FEAT-17.SPEC-002 with the new/updated block visible | FEAT-17.SPEC-001, FEAT-17.SPEC-002 |
| Routed to conflict review | One or more confirmed bookings overlap the block's range | No commit yet -- the block's values are held pending Talia's choices | Talia is navigated to FEAT-17.SPEC-003 with the conflicting set shown | FEAT-17.SPEC-003 |
| Validation failed | Start/end or label fails FEAT-17.SPEC-008's rules | None | Field-level error shown on FEAT-17.SPEC-001 | FEAT-17.SPEC-001 |
| Edited record no longer exists | The block being edited was removed by another session before this save completed | None | "This time block was removed. Start over?" shown on FEAT-17.SPEC-001 | FEAT-17.SPEC-001 |
| Automation failure | A processing error prevents the save from completing | None | Error banner "Couldn't save this time block. Check your connection and try again." on FEAT-17.SPEC-001, with Retry | FEAT-17.SPEC-001 |

## Data Model

**Reads:** Time Block (existing record, on edit); Booking -- state, start_time, duration, per the Pro Account, to compute overlap against the pending block's range.
**Creates:** Time Block record, on a conflict-free create.
**Updates:** Time Block record, on a conflict-free edit.
**Deletes:** None.

## Business Rules

- Only Bookings in state Confirmed or Awaiting Outcome count as conflicting; Pending Payment, Completed, No-Show, Cancelled, Rescheduled, and Expired (unpaid) bookings never block a save (FEAT-17.SPEC-008: what counts as a conflicting booking).
- The block never commits while a conflict is unresolved -- committing happens either here (zero conflicts) or in FEAT-17.SPEC-006 (once Talia's choices are captured), never both, and never silently (XBR-11, FEAT-17.SPEC-008).
- Validation runs synchronously -- the triggering screen waits for this automation's result before showing any outcome.
- A block save and a client's in-progress checkout for the same instant of time resolve by FEAT-03.SPEC-005's first-committed-wins rule; this automation checks only already-confirmed bookings, not in-progress holds.

## Edge Cases

- **The block's range overlaps a Booking that is Pending Payment (not yet confirmed)** -- Not treated as a conflict; an unpaid, unconfirmed hold does not block the save. If that hold later completes into a Confirmed booking after this block already committed, FEAT-03.SPEC-005's contention rule governs, not this automation.
- **Talia edits a block to shrink its range so a previously-conflicting booking no longer overlaps** -- The re-validation in step 5 finds zero conflicts for the new range and the edit commits immediately, even if the block previously had a conflict on an earlier save attempt.
- **The block's range exactly touches a Booking's start or end with no overlap (adjacent, not overlapping)** -- Not a conflict, per FEAT-17.SPEC-008's boundary definition.
- **Automation processing fails partway through the conflict check** -- No partial commit occurs; the save fails cleanly and the triggering screen shows the automation-failure outcome with entered values preserved.
- **Concurrent trigger firing (Talia saves the same block from two open sessions at effectively the same time)** -- Each save runs its own conflict check independently against the data visible when it starts; the save that commits first wins, and the second save's re-validation (step 4, for an edit) or a subsequent read will reflect the first save's result on its next attempt.
- **Trigger fires while a previous run for the same block is still in flight** -- FEAT-17.SPEC-001's Save button is disabled while a save is in progress, so a second run for the same submission cannot start; a save for a different block proceeds independently.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-001 (Create/Edit Time Block) | Triggered by (inbound) | Fires on Save after field validation passes |
| FEAT-17.SPEC-003 (Time Block Conflict Review) | Affects (outbound) | Receives the pending block and conflicting set when conflicts are found |
| FEAT-17.SPEC-008 (Time Block Validation & Conflict Handling Rules) | References (inbound) | Field validation and conflicting-booking definition |
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | A committed block is read live by the next slot computation |
| FEAT-03.SPEC-005 (Slot Contention Resolution Rules) | References (inbound) | Governs contention against an in-progress client checkout, outside this automation's own scope |

## Analytics and Success Signals

- **time_block_added** (source: single-date create or edit) -- supports success-metrics.md: "Zero Double-Booking Confidence"
- **time_block_conflict_flagged** (conflicting booking count) -- supports success-metrics.md: "Zero Double-Booking Confidence"

## Acceptance Criteria

**FEAT-17.SPEC-004-AC-01:** Given Talia submits a new block whose range overlaps zero confirmed bookings, when this automation runs, then the block commits immediately and she is returned to FEAT-17.SPEC-002.

**FEAT-17.SPEC-004-AC-02:** Given Talia submits a new block whose range overlaps one confirmed booking, when this automation runs, then the block is held pending and she is routed to FEAT-17.SPEC-003 with that booking listed.

**FEAT-17.SPEC-004-AC-03:** Given Talia submits a block with an end time not after its start time, when this automation re-validates, then the specific field error is returned to FEAT-17.SPEC-001 and nothing commits.

**FEAT-17.SPEC-004-AC-04:** Given Talia edits an existing block that another session already removed, when she saves, then the "removed record" outcome is returned and FEAT-17.SPEC-001 shows "This time block was removed. Start over?"

**FEAT-17.SPEC-004-AC-05:** Given a Booking in the block's range is Pending Payment and not yet confirmed, when the conflict check runs, then that booking is not counted as a conflict and the save proceeds toward commit.

**FEAT-17.SPEC-004-AC-06:** Given Talia edits a block to a smaller range that no longer overlaps a previously conflicting booking, when she saves, then the edit commits immediately with no routing to conflict review.

**FEAT-17.SPEC-004-AC-07:** Given a block's range ends exactly at a confirmed booking's start time, when the conflict check runs, then it is treated as adjacent, not overlapping, and is not a conflict.

**FEAT-17.SPEC-004-AC-08:** Given a processing error occurs during the conflict check, when the automation reports failure, then FEAT-17.SPEC-001 shows "Couldn't save this time block. Check your connection and try again." with entered values preserved.

**FEAT-17.SPEC-004-AC-09:** Given Talia saves the same block from two open sessions at effectively the same time, when both saves run, then the one that commits first succeeds and the second reflects that result on its own re-validation.

**FEAT-17.SPEC-004-AC-10:** Given a block commits with zero conflicts, when the next slot computation runs, then FEAT-03.SPEC-001 reads the new block and removes its time from the bookable list.

**FEAT-17.SPEC-004-AC-11:** Given a block's save is in flight, when FEAT-17.SPEC-001's Save button is tapped again, then no second run starts for that same submission.

**FEAT-17.SPEC-004-AC-12:** Given a conflict is found and Talia is routed to FEAT-17.SPEC-003, when the routing occurs, then the `time_block_conflict_flagged` event is emitted with the conflicting booking count.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (committed, routed, validation failed, removed record, automation failure) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
