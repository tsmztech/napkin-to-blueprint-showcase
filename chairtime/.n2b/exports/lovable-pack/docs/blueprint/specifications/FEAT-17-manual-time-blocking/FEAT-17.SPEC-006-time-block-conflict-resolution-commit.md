---
document_type: spec
spec_type: automation
spec_id: FEAT-17.SPEC-006
spec_name: Time Block Conflict Resolution Commit
spec_slug: time-block-conflict-resolution-commit
parent_feature: FEAT-17
parent_feature_name: Manual Time Blocking
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Automation Spec: Time Block Conflict Resolution Commit

## Overview

**Name:** Time Block Conflict Resolution Commit
**ID:** FEAT-17.SPEC-006
**Type:** Automation
**Purpose:** Commits Talia's explicit choice on a conflicting booking set -- hand off to cancellation, hand off to reschedule, or mark the booking as a kept exception -- and finalizes the block once every conflicting booking has a resolved outcome.
**Parent Feature:** FEAT-17 -- Manual Time Blocking

## Scope and Non-Goals

**In Scope:**
- Committing the pending Time Block (or occurrence) once Talia's per-booking choices are submitted from FEAT-17.SPEC-003
- Handing every "Cancel" booking to bulk cancellation as one batch, and every "Reschedule" booking individually to Pro-initiated reschedule
- Marking every "Keep as exception" booking as a resolved exception, flagged for the Pro's attention
- Tracking each conflicting booking's resolved/unresolved state until the whole set is resolved

**Non-Goals:**
- Presenting the choices to Talia -- owned by FEAT-17.SPEC-003, which this automation receives its input from
- Performing the cancellation or reschedule itself -- owned by FEAT-30.SPEC-005 and FEAT-30.SPEC-002 respectively; this automation only hands off and tracks completion
- Deciding a bulk-reschedule outcome -- excluded per the Brief's Non-Goals: FEAT-30 reschedules one booking at a time only

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia taps Confirm on the conflict review screen | FEAT-17.SPEC-003 (Time Block Conflict Review) | Fires once every conflicting booking in the set has a chosen outcome | The pending block's (or occurrence's) values, and the per-booking choice (cancel / reschedule / keep as exception) for every conflicting Booking |

## Processing Logic

1. Receive the pending block (or occurrence) values and the full set of per-booking choices from FEAT-17.SPEC-003.
2. Re-validate each conflicting booking's current state (it may have changed since FEAT-17.SPEC-003 loaded it); drop any booking that is no longer Confirmed or Awaiting Outcome from the set, since it is no longer a conflict.
3. Commit the Time Block (or occurrence) record immediately -- the block itself does not wait for the conflicting bookings' outcomes to complete, per XBR-11: setup changes are never blocked on how a conflict resolves.
4. Group every remaining booking chosen "Cancel" into one batch and hand it to FEAT-30.SPEC-005 (Cancel Several Bookings at Once); mark each as pending resolution until FEAT-30.SPEC-005 reports completion.
5. For every booking chosen "Reschedule," hand it individually to FEAT-30.SPEC-002 (Reschedule Booking, Pro-Initiated); mark each as pending resolution until FEAT-30.SPEC-002 reports completion.
6. For every booking chosen "Keep as exception," mark it resolved immediately as a kept exception, and raise an attention signal for it (consumed by FEAT-12.SPEC-005).
7. Track the resolved/unresolved status of every booking in the original conflicting set.
8. Once every booking in the set reaches a resolved outcome (cancelled, rescheduled, or kept as exception), mark the block fully finalized with no remaining pending conflicts.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Block committed, all bookings resolved immediately | Every booking is "Keep as exception" (no cancel or reschedule hand-offs pending) | Block committed; every booking flagged as a kept exception | Talia is returned to FEAT-17.SPEC-002 | FEAT-17.SPEC-002, FEAT-12.SPEC-005 |
| Block committed, hand-offs pending | One or more bookings chosen "Cancel" or "Reschedule" | Block committed; cancel batch handed to FEAT-30.SPEC-005; reschedule bookings handed individually to FEAT-30.SPEC-002 | Talia is routed to the first pending hand-off screen | FEAT-30.SPEC-005, FEAT-30.SPEC-002 |
| Every pending hand-off completes | All handed-off bookings report a completed outcome from FEAT-30 | Block marked fully finalized | No separate notice -- the block simply shows no pending conflicts on FEAT-17.SPEC-002 | FEAT-17.SPEC-002 |
| A booking dropped from the set before resolution | Re-validation (step 2) finds the booking is no longer Confirmed or Awaiting Outcome | That booking is excluded from any hand-off | No user-visible change beyond a smaller conflicting set | FEAT-17.SPEC-003 |
| A hand-off is abandoned or left incomplete | Talia navigates away from FEAT-30.SPEC-005 or FEAT-30.SPEC-002 before completing it | That booking remains unresolved | The booking is flagged on the dashboard as an unresolved exception (per the touchpoint: "leaves unresolved" feeds an attention flag) until Talia revisits and completes the hand-off | FEAT-12.SPEC-005 |
| Commit failure | A processing error prevents the block itself from committing | No block created; no bookings altered | Error banner on FEAT-17.SPEC-003: "Couldn't save your choices. Check your connection and try again." with Retry | FEAT-17.SPEC-003 |

## Data Model

**Reads:** Booking -- current state, for re-validation before hand-off.
**Creates:** Time Block (or occurrence) record -- committed from the pending values held by FEAT-17.SPEC-004/FEAT-17.SPEC-005.
**Updates:** Booking -- no direct field write by this automation; state changes for cancel/reschedule are owned by FEAT-30.SPEC-005/FEAT-30.SPEC-002. This automation writes only the resolution tracking (resolved/unresolved, and outcome kind) associated with the block's conflict set.
**Deletes:** None.

## Business Rules

- The block never waits for a cancel or reschedule hand-off to complete before it commits -- committing the block and resolving its conflicting bookings are decoupled, so the Pro's setup change is never blocked by a client-facing process (XBR-11).
- A "Keep as exception" booking is never altered in any field -- only a resolution-tracking flag is set (FEAT-17.SPEC-008: never-silently-affect-a-booking).
- An unresolved booking (a hand-off Talia abandoned) is always surfaced on the dashboard -- it is never silently dropped (XBR-11, FEAT-12.SPEC-005).
- Every "Cancel" choice in one Confirm submission is batched into exactly one bulk-cancel hand-off; every "Reschedule" choice is its own individual hand-off -- these are never merged or split further.

## Edge Cases

- **A booking chosen "Cancel" is no longer Confirmed by the time this automation runs (e.g., the client cancelled it themselves moments earlier)** -- Dropped from the cancel batch in step 2; the block commits as if that booking had never conflicted.
- **Talia completes the reschedule hand-off for one booking but abandons the bulk-cancel hand-off for others** -- The rescheduled booking resolves normally; the abandoned cancel batch's bookings remain unresolved and are flagged per the abandoned-hand-off outcome.
- **Every booking in the set is dropped during re-validation (step 2)** -- The block commits with zero remaining conflicts and is immediately finalized, with no hand-off screen shown at all.
- **Concurrent trigger firing (Talia confirms the same conflict set from two open sessions)** -- The first commit to complete wins; the second's re-validation (step 2) finds the block already exists and the bookings already resolved, so it reports the already-finalized outcome rather than duplicating any hand-off.
- **Trigger fires while a previous resolution commit for the same block is still in flight** -- FEAT-17.SPEC-003's Confirm button is disabled while a submission is in progress, so a second run for the same submission cannot start.
- **A kept-exception booking is later cancelled by Talia through an unrelated path (FEAT-30.SPEC-001)** -- The exception flag raised here is cleared by that cancellation's own completion, since the booking it referred to no longer exists in a state that needs flagging; this automation itself performs no further action once the flag has been raised.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-003 (Time Block Conflict Review) | Triggered by (inbound) | Confirm submits the per-booking choices |
| FEAT-30.SPEC-005 (Cancel Several Bookings at Once) | Affects (outbound) | Receives the bulk-cancel batch |
| FEAT-30.SPEC-002 (Reschedule Booking, Pro-Initiated) | Affects (outbound) | Receives each individual reschedule hand-off |
| FEAT-12.SPEC-005 (Attention Flag Aggregation) | Affects (outbound) | Receives the kept-exception and unresolved-conflict flags |
| FEAT-17.SPEC-008 (Time Block Validation & Conflict Handling Rules) | References (inbound) | Never-silently-affect-a-booking rule enforced throughout |
| FEAT-17.SPEC-002 (Manage Time Blocks) | Affects (outbound) | Destination once the block is committed and (fully or partially) resolved |
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | The committed block is read live once finalized |

## Analytics and Success Signals

- **time_block_conflict_resolved** (outcome: cancel / reschedule / keep_as_exception / unresolved, per booking) -- supports success-metrics.md: "Zero Double-Booking Confidence"

## Acceptance Criteria

**FEAT-17.SPEC-006-AC-01:** Given Talia confirms choices for two conflicting bookings, one "Cancel" and one "Reschedule," when this automation runs, then the block commits immediately and Talia is routed to the bulk-cancel hand-off first.

**FEAT-17.SPEC-006-AC-02:** Given Talia confirms "Keep as exception" for every conflicting booking, when this automation runs, then the block commits, every booking is flagged as a kept exception, and Talia returns directly to FEAT-17.SPEC-002.

**FEAT-17.SPEC-006-AC-03:** Given a booking chosen "Cancel" is no longer Confirmed when this automation runs, when re-validation occurs, then that booking is dropped from the batch and the block commits as if it had never conflicted.

**FEAT-17.SPEC-006-AC-04:** Given every booking in the conflicting set is dropped during re-validation, when this automation completes, then the block commits with zero remaining conflicts and no hand-off screen is shown.

**FEAT-17.SPEC-006-AC-05:** Given Talia abandons the bulk-cancel hand-off after confirming her choices, when she navigates away without completing it, then those bookings remain unresolved and are flagged on her dashboard via FEAT-12.SPEC-005.

**FEAT-17.SPEC-006-AC-06:** Given every booking in a set eventually reaches a resolved outcome, when the last one resolves, then the block is marked fully finalized with no remaining pending conflicts.

**FEAT-17.SPEC-006-AC-07:** Given Talia keeps a booking as an exception, when the flag is raised, then no field on that Booking record is altered.

**FEAT-17.SPEC-006-AC-08:** Given a commit failure occurs while this automation runs, when the failure is reported, then FEAT-17.SPEC-003 shows "Couldn't save your choices. Check your connection and try again." with Retry, and no block or booking change persists.

**FEAT-17.SPEC-006-AC-09:** Given Talia confirms the same conflict set from two open sessions at effectively the same time, when both submissions run, then the first to complete commits and the second reports the already-finalized outcome without duplicating any hand-off.

**FEAT-17.SPEC-006-AC-10:** Given a resolution commit for a block is already in flight, when FEAT-17.SPEC-003's Confirm is tapped again for that same submission, then no second run starts.

**FEAT-17.SPEC-006-AC-11:** Given a kept-exception booking is later cancelled through FEAT-30.SPEC-001, when that cancellation completes, then the exception flag raised by this automation is cleared.

**FEAT-17.SPEC-006-AC-12:** Given the block commits regardless of pending hand-offs, when Talia checks FEAT-17.SPEC-002 immediately after confirming, then the new block is already visible even before any cancel or reschedule hand-off has completed.

**FEAT-17.SPEC-006-AC-13:** Given a booking's resolution reaches an outcome, when it does, then the `time_block_conflict_resolved` event is emitted naming that outcome.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 6 (all resolved immediately, hand-offs pending, hand-offs complete, booking dropped, hand-off abandoned, commit failure) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
