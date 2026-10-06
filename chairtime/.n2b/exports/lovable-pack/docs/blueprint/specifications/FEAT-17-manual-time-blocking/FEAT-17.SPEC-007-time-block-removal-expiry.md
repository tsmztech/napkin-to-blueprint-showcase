---
document_type: spec
spec_type: automation
spec_id: FEAT-17.SPEC-007
spec_name: Time Block Removal & Expiry
spec_slug: time-block-removal-expiry
parent_feature: FEAT-17
parent_feature_name: Manual Time Blocking
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Time Block Removal & Expiry

## Overview

**Name:** Time Block Removal & Expiry
**ID:** FEAT-17.SPEC-007
**Type:** Automation
**Purpose:** Deletes a block Talia removes early, or automatically retires a block once its end time has passed, in either case restoring that time to bookable availability immediately.
**Parent Feature:** FEAT-17 -- Manual Time Blocking

## Scope and Non-Goals

**In Scope:**
- Hard-deleting a Time Block (single-date or one generated occurrence) on Talia's explicit removal
- Automatically retiring a block or occurrence once its own end time passes
- Restoring the freed time to bookable availability immediately in both cases

**Non-Goals:**
- Presenting the removal confirmation to Talia -- owned by FEAT-17.SPEC-002, which triggers this automation
- Any retention, archive, or undo path for a removed or expired block -- excluded per the Brief's own Non-Goals: a Time Block carries no historical-record requirement of its own (Data Notes: displayed only on the Pro's own schedule view), unlike Booking's SC-22 retention
- Retiring the recurrence pattern definition itself -- a pattern has no "end time" of its own to expire; only its individual generated occurrences expire here, one at a time

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia confirms removal of a block | FEAT-17.SPEC-002 (Manage Time Blocks) | Fires when Talia taps "Remove" on the confirmation dialog | The Time Block's identity |
| A block's or occurrence's own end time passes | System (time-based) | Fires when the current time crosses a Time Block record's end value and it has not already been removed | The Time Block's identity and end value |

## Processing Logic

1. Receive the triggering event: either an explicit removal request (with the block's identity) or a scheduled expiry check crossing a block's end time.
2. For an explicit removal, confirm the block still exists (it may have already expired or been removed by another session); if it does not, report the already-gone outcome.
3. For a scheduled expiry, identify every Time Block whose end value has just passed and which has not already been removed.
4. Delete the identified Time Block record (hard delete -- no retention window).
5. Signal the deletion so the next slot computation reads the current, updated set of Time Blocks with the freed time available immediately.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Explicit removal succeeds | Talia confirms removal and the block exists | Time Block record deleted | Row disappears from FEAT-17.SPEC-002 immediately | FEAT-17.SPEC-002, FEAT-03.SPEC-001 |
| Automatic expiry | A block's or occurrence's end time passes | Time Block record deleted | No notice shown -- routine background retirement; the block simply no longer appears on next view of FEAT-17.SPEC-002 or FEAT-12 | FEAT-17.SPEC-002, FEAT-12, FEAT-03.SPEC-001 |
| Already gone | The block targeted for explicit removal no longer exists (already expired or removed elsewhere) | None | Row is simply absent from the list on next refresh; no error shown, since the Pro's intended outcome (the time being free) is already true | FEAT-17.SPEC-002 |
| Removal failure | A processing error prevents the delete from completing | None | Inline error on FEAT-17.SPEC-002: "Couldn't remove this time block. Try again." with the remove affordance still available | FEAT-17.SPEC-002 |

## Data Model

**Reads:** Time Block -- identity and end value, to confirm existence (explicit removal) or to identify newly expired records (scheduled check).
**Creates:** None.
**Updates:** None.
**Deletes:** Time Block record -- hard delete, in both the explicit-removal and automatic-expiry outcomes.

## Business Rules

- Deletion never cascades to a conflicting Booking kept as an exception -- that booking is left completely untouched (Validation & Limits: a block cannot silently delete a booking).
- No retention or undo path applies to a removed or expired block -- this is an intentional lifecycle decision, since the entity carries no historical-record requirement of its own.
- The freed time becomes bookable again within roughly one second of removal or expiry (ASMP-21), since the slot computation reads Time Block data live with no caching (FEAT-03.SPEC-001).
- The Active -> Expired state derivation this automation acts on (a block expires the moment its own end value passes, system-derived only, never overridable by Talia) is defined by FEAT-17.SPEC-008 (Time Block Validation & Conflict-Handling Rules), which lists this automation as an enforcing spec.
- A recurring pattern's individual occurrences expire independently of one another and of the pattern definition -- expiring one occurrence never affects the pattern's other future occurrences (owned by FEAT-17.SPEC-005) or already-generated ones.

## Edge Cases

- **Talia removes a block at the exact moment its end time passes (a race between explicit removal and automatic expiry)** -- Whichever action completes first performs the delete; the other finds the block already gone and reports the already-gone outcome with no error.
- **A generated recurring occurrence expires while its parent pattern is being edited on FEAT-17.SPEC-001** -- The already-elapsed occurrence is unaffected by the concurrent edit (FEAT-17.SPEC-005's regeneration never touches occurrences whose end time has passed); this automation's expiry proceeds independently.
- **The scheduled expiry check runs while a large number of occurrences cross their end time at once (e.g., many Sundays' worth reach end-of-day together)** -- Each is deleted independently; there is no ordering dependency between them, since a solo Pro's block set stays small (Non-Functional Notes: data volumes).
- **Concurrent trigger firing (Talia's explicit removal and the scheduled expiry check target the same block at effectively the same time)** -- The first to complete performs the delete; the second's existence check (step 2) or expiry scan (step 3) finds the record already gone and produces no error, since both actions intend the same outcome.
- **Trigger fires while a previous removal run for the same block is still in flight** -- A second run for the same block's identity is not started while the first is in flight; the second, once the first completes, finds the record already gone and reports the already-gone outcome.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-002 (Manage Time Blocks) | Triggered by (inbound) | Confirmed removal fires this automation |
| FEAT-17.SPEC-005 (Recurring Time Block Occurrence Generation) | References (inbound) | Generated occurrences are the records this automation later expires |
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | Reads the current Time Block set live -- a deletion here is reflected on the next computation with no separate propagation step |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | A removed or expired block no longer appears on the Pro's schedule view |
| FEAT-17.SPEC-008 (Time Block Validation & Conflict-Handling Rules) | Governed by | Defines the Active -> Expired state derivation that the expiry path enforces |

## Analytics and Success Signals

- **time_block_removed** (source: explicit removal or automatic expiry) -- supports success-metrics.md: "Zero Double-Booking Confidence"

## Acceptance Criteria

**FEAT-17.SPEC-007-AC-01:** Given Talia confirms removal of an existing block, when this automation runs, then the record is hard-deleted and the row disappears from FEAT-17.SPEC-002 immediately.

**FEAT-17.SPEC-007-AC-02:** Given a single-date block's end time passes with no Pro action, when the scheduled expiry check runs, then the record is deleted automatically with no notice shown to Talia.

**FEAT-17.SPEC-007-AC-03:** Given a generated recurring occurrence's end time passes, when the scheduled expiry check runs, then only that occurrence is deleted, and the pattern's other future occurrences are unaffected.

**FEAT-17.SPEC-007-AC-04:** Given Talia attempts to remove a block that already expired moments earlier, when this automation checks its existence, then no error is shown and the row is simply absent on the next refresh.

**FEAT-17.SPEC-007-AC-05:** Given a removed block had a confirmed booking kept as an exception, when the deletion completes, then that booking is left completely untouched.

**FEAT-17.SPEC-007-AC-06:** Given a block is deleted (removal or expiry), when the next slot computation runs, then FEAT-03.SPEC-001 reads the current Time Block set and the freed time appears bookable within roughly one second.

**FEAT-17.SPEC-007-AC-07:** Given a processing error occurs during an explicit removal, when the failure is reported, then FEAT-17.SPEC-002 shows "Couldn't remove this time block. Try again." with the remove affordance still available.

**FEAT-17.SPEC-007-AC-08:** Given Talia's explicit removal and the scheduled expiry check target the same block at effectively the same time, when both run, then whichever completes first performs the delete and the other reports no error.

**FEAT-17.SPEC-007-AC-09:** Given many recurring occurrences reach their end time together, when the scheduled expiry check runs, then each is deleted independently with no ordering dependency between them.

**FEAT-17.SPEC-007-AC-10:** Given a removal run for a block is already in flight, when a second removal request for the same block arrives, then no second run starts, and it later finds the record already gone.

**FEAT-17.SPEC-007-AC-11:** Given a block is removed or expires, when the deletion completes, then the `time_block_removed` event is emitted.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 4 (explicit removal, automatic expiry, already gone, removal failure) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
