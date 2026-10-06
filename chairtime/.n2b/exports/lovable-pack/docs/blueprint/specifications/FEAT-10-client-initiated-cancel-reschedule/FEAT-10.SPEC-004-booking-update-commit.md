---
document_type: spec
spec_type: automation
spec_id: FEAT-10.SPEC-004
spec_name: Booking Update Commit
spec_slug: booking-update-commit
parent_feature: FEAT-10
parent_feature_name: Client-Initiated Cancel/Reschedule
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 16
---

# Automation Spec: Booking Update Commit

## Overview

**Name:** Booking Update Commit
**ID:** FEAT-10.SPEC-004
**Type:** Automation
**Purpose:** Commits the client's confirmed cancellation or reschedule to the Booking record, coordinating the deposit outcome, calendar mirroring, activity logging, and freed-slot handoff this triggers in other features.
**Parent Feature:** FEAT-10 -- Client-Initiated Cancel/Reschedule

## Scope and Non-Goals

**In Scope:**
- Re-checking eligibility (FEAT-10.SPEC-005) at the instant of commit, as the authoritative gate
- Writing the Booking state transition: Cancelled by Client, or the in-place time update for an outside-window reschedule, or the compound Rescheduled-plus-new-Booking transition for an inside-window (late) reschedule
- Handing off the recorded action to FEAT-09.SPEC-004 for deposit outcome evaluation
- Handling commit failure so the original booking is left untouched and intact
- Resolving a conflicting concurrent transition via reject-with-refresh

**Non-Goals:**
- Determining eligibility or the window comparison itself -- owned by FEAT-10.SPEC-005; this automation re-checks that spec's result at commit time but never redefines it
- Deriving or applying the deposit outcome -- owned by FEAT-09.SPEC-003 (rule set) and FEAT-09.SPEC-004 (evaluation); this automation only records the action and hands off, per the Side-Effect Inventory's disposition of that response as "FEAT-09 responsibility (XBR-09)"
- Moving or removing the Pro's personal calendar entry -- owned by FEAT-04 (Two-Way Calendar Sync, XBR-13); this automation's commit is what FEAT-04 reacts to
- Writing the append-only activity event -- owned by FEAT-16 (Booking & Payment Activity Record, XBR-21); this automation's commit is what FEAT-16 reacts to
- Notifying the client or the Pro -- owned by FEAT-10.SPEC-006 (Cancellation/Reschedule Notification), which this automation triggers on success
- Making the freed slot bookable and notifying the waitlist -- owned by FEAT-20 (Waitlist for Cancelled Slots, XBR-28); this automation's cancellation commit is what FEAT-20 reacts to
- Maintaining the Pro's rolling booking-count and revenue aggregates -- owned by FEAT-25.SPEC-004 (Historical Aggregate Maintenance); this automation's cancellation, in-place reschedule, and terminal-Rescheduled commits are the events FEAT-25.SPEC-004 reacts to, and this automation performs no aggregate write itself

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client confirms a cancellation | FEAT-10.SPEC-001 (Cancel Booking) | Client taps "Confirm Cancellation" in the confirm dialog | Booking reference, cancellation timestamp |
| Client confirms a reschedule | FEAT-10.SPEC-003 (Reschedule -- Outcome & Confirm) | Client taps "Confirm Reschedule" | Booking reference, chosen new start_time, reschedule timestamp |

## Processing Logic

1. Receive the confirmed action (cancel or reschedule) with the Booking reference and, for a reschedule, the chosen new start_time.
2. Re-check eligibility for this Booking (FEAT-10.SPEC-005) against its *current* state -- if ineligible (e.g., a conflicting transition already committed, or the state has since moved to Completed/No-Show), stop and report the conflict outcome (see Outcome Definitions) rather than proceeding.
3. For a reschedule, re-validate the chosen new start_time against live availability one final time (FEAT-03, XBR-01) -- if the slot is no longer free, stop and report the slot-lost outcome.
4. Compute the window comparison (FEAT-10.SPEC-005) against the Booking's original start_time, to determine which branch the commit will produce.
5. **If cancelling:** write the Booking's state to Cancelled by Client and set the cancellation timestamp.
6. **If rescheduling outside the window (Rule 5 applies):** update the same Booking record's start_time to the chosen new time and set the reschedule timestamp; state remains unchanged (Confirmed or Awaiting Outcome, whichever it already was); the existing Deposit Transaction and Access Link continue to apply to this same record.
7. **If rescheduling inside the window (Rule 6 applies -- late reschedule):** (a) transition the original Booking's state to Rescheduled (terminal) and set the reschedule timestamp; (b) create a new Booking record for the chosen new time, carrying over the same service, duration, client, and the policy version current at that moment (a fresh acknowledgment, exactly as a new booking would, since this is functionally a late-cancellation-plus-new-booking per FEAT-09.SPEC-003 Rule 6); (c) flag the new Booking as requiring its own fresh deposit under FEAT-07's ordinary eligibility rules.
8. Hand off the recorded action (Booking reference, action type, initiator: Client, timestamp, and for a reschedule the original and new start_time) to FEAT-09.SPEC-004 for deposit outcome evaluation.
9. On a successful write, trigger FEAT-10.SPEC-006 (Cancellation/Reschedule Notification) with the outcome type and, for a late reschedule, the new Booking's reference.
10. Report success back to the triggering screen with the resulting state, for FEAT-10.SPEC-001/FEAT-10.SPEC-003 to route the client onward (to FEAT-06.SPEC-004, or into FEAT-07's deposit step for a late reschedule).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|-----------------|-------------------|
| Cancellation committed | Eligibility passes; client confirmed a cancel | Booking: state -> Cancelled by Client, cancellation timestamp set | Client is routed to FEAT-06.SPEC-004 showing the booking as Cancelled | FEAT-06.SPEC-004, FEAT-09.SPEC-004, FEAT-10.SPEC-006 |
| Reschedule committed, outside window | Eligibility passes; window comparison is outside; slot re-validation passes | Booking: start_time -> new time, reschedule timestamp set; state unchanged | Client is routed to FEAT-06.SPEC-004 showing the new time; deposit carries over | FEAT-06.SPEC-004, FEAT-09.SPEC-004, FEAT-10.SPEC-006 |
| Reschedule committed, inside window (late reschedule, compound) | Eligibility passes; window comparison is inside; slot re-validation passes | Original Booking: state -> Rescheduled, reschedule timestamp set. New Booking created with the new start_time, same service/duration/client, flagged as requiring a fresh deposit | Client is routed into FEAT-07's deposit-payment step for the new Booking | FEAT-07, FEAT-09.SPEC-004, FEAT-10.SPEC-006 |
| Eligibility conflict (a Pro-side or automated transition committed first) | Re-check at step 2 finds the Booking's current state no longer eligible | No change to any Booking record from this attempt | Client sees "This booking's details changed. Refresh to see the latest before continuing." on the triggering screen (FEAT-10.SPEC-001/FEAT-10.SPEC-003), per the Booking entity's reject-with-refresh Contention resolution | FEAT-10.SPEC-001, FEAT-10.SPEC-003 |
| Slot lost to contention (reschedule only) | Re-validation at step 3 finds the chosen new time no longer free | No change to any Booking record | Client sees "That time was just taken. Choose another." and returns to a refreshed FEAT-10.SPEC-002 | FEAT-10.SPEC-002, FEAT-10.SPEC-003 |
| Commit failure (the write itself does not save) | A transient failure during the write in steps 5-7 | No partial state change is left behind -- the original Booking remains exactly as it was before the attempt | Client sees "We couldn't cancel/reschedule this booking. Try again." with a Retry option on the triggering screen | FEAT-10.SPEC-001, FEAT-10.SPEC-003 |

## Data Model

**Reads:** Booking -- state, start_time, policy_version, service, duration, client reference. Cancellation Policy -- window_hours and computed cutoff, via FEAT-09.SPEC-002 (through FEAT-10.SPEC-005).
**Creates:** Booking -- a new record for the inside-window (late reschedule) compound outcome only, with service, duration, client, start_time (the chosen new time), and a freshly acknowledged policy_version, mirroring the fields a fresh booking (FEAT-05) would set; flagged as requiring its own deposit rather than assigning one directly (FEAT-07 owns deposit capture).
**Updates:** Booking -- state (to Cancelled by Client, or to Rescheduled for the original record in a late reschedule), start_time (for an outside-window reschedule, in place on the same record), and the cancellation/reschedule timestamp.
**Deletes:** None -- bookings are never deleted (SC-22); a cancelled or rescheduled booking remains as history with an updated state.

## Business Rules

- **Resolution of the Entity-Lifecycle Coverage Matrix's flagged discrepancy:** a client-initiated reschedule outside the cancellation window updates the existing Booking record's start_time in place -- the record's state is unchanged and no new record is created, matching FEAT-09.SPEC-003 Rule 5's "No Change... the deposit carries over" wording, which presumes one continuing record and deposit. A client-initiated reschedule inside the window is the sole case in this feature that creates a new Booking record: the original transitions to the terminal Rescheduled state and a new Booking is created for the new time, matching FEAT-09.SPEC-003 Rule 6's explicit "newly created Booking" language and its Edge Case confirming the original reaches "a terminal transition (Rescheduled)." This means this automation is a Booking Creator for the narrow late-reschedule compound case -- a fact not reflected in the dependency map's Booking "Creators: FEAT-05, FEAT-30, FEAT-21" list, which is flagged here as a carry-forward item for Stage 4/reconciliation to add FEAT-10 to that list for this one scenario, rather than silently contradicting the dependency map.
- **Consistency with feature-overview.md:** this feature's Brief (Entity-Lifecycle Coverage Matrix, Create row) states that FEAT-10.SPEC-004 creates one new Booking record, only on an inside-window (late) reschedule, and nothing on a cancellation or outside-window reschedule; this spec's step 7(b) and Data Model Creates entry are exactly that mechanism. The dependency map's Booking Creators line still omits FEAT-10 for this one scenario; that map-level delta is carried to Stage 4 (SG-01) and is not edited here.
- XBR-08: the new Booking created in a late reschedule acknowledges the cancellation policy version current at that moment, exactly as a fresh booking would -- it never inherits the original booking's bound version.
- XBR-13: this commit is the event FEAT-04 mirrors to (or removes from) the Pro's personal calendar; this automation performs no calendar write itself.
- XBR-18: a fresh manage link is not issued by this automation for an outside-window reschedule, since the same Booking record and its existing Access Link continue to apply; a late reschedule's new Booking is reached through the deposit-payment confirmation's own manage-link issuance (FEAT-08.SPEC-010), exactly as any new booking is.
- XBR-21: this commit is the event FEAT-16 writes an append-only activity event for; this automation performs no activity-log write itself.
- XBR-28: a cancellation commit is the event FEAT-20 reacts to for waitlist priority notification; this automation performs no waitlist write itself.
- The Booking entity's Contention resolution (reject-with-refresh) governs step 2's re-check: the first committed state transition always wins, and every transition is validated against the current state before this automation writes anything.

## Edge Cases

- **A client cancellation and a Pro-side action (FEAT-30) arrive for the same Booking at effectively the same time** -- Whichever commits first wins; the second arrival's re-check at step 2 finds the Booking already in a non-eligible current state and reports the eligibility-conflict outcome. Only one terminal transition is ever written for a given Booking.
- **The commit fails partway through the compound inside-window write (original transitioned to Rescheduled but the new Booking's creation fails)** -- The write is treated as a single atomic step: if any part fails, no part commits -- the original Booking is left exactly as it was (not transitioned) and no new Booking is created, so the client sees the ordinary commit-failure outcome and can retry cleanly rather than being left with an orphaned Rescheduled original and no replacement.
- **Trigger fires while a previous run is already in flight for the same Booking** -- Not possible in practice: FEAT-10.SPEC-001 and FEAT-10.SPEC-003 disable their confirm controls while a commit is in progress, and a Booking has at most one terminal cancellation/reschedule transition ever recorded, so a second commit attempt for the same Booking cannot start while the first is in flight.
- **Concurrent commit attempts for two different Bookings** -- Proceed independently; neither is delayed by the other.
- **A late reschedule's new Booking fails to reach FEAT-07's deposit step (e.g., the client closes the app before paying)** -- Governed by FEAT-03's standard slot-hold and Pro-created-deposit-request expiration rules (XBR-02) applied to the new Booking exactly as to any unpaid booking; this automation's own responsibility ends once the new Booking is created and flagged.
- **The Booking's bound policy version cannot be read at the instant of commit (the same rare inconsistency FEAT-09.SPEC-004 notes)** -- The commit is held rather than writing an outcome-less transition; the client sees the ordinary loading/error handling on the triggering screen and can retry, consistent with FEAT-09.SPEC-004's own "Outcome pending" handling once the write does proceed.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-001 (Cancel Booking) | Triggered by (inbound) | "Confirm Cancellation" fires this automation |
| FEAT-10.SPEC-003 (Reschedule -- Outcome & Confirm) | Triggered by (inbound) | "Confirm Reschedule" fires this automation |
| FEAT-10.SPEC-005 (Cancellation Window & Eligibility Rule) | References (outbound) | Re-checked at commit as the authoritative eligibility gate |
| FEAT-03 (Real-Time Slot Availability Engine) | References (outbound) | Final slot re-validation for a reschedule's chosen time |
| FEAT-09.SPEC-004 (Cancellation & No-Show Outcome Evaluation) | Triggers (outbound) | Hands off the recorded action for deposit outcome evaluation |
| FEAT-07 (Deposit Payment at Booking) | Triggers (outbound) | A late reschedule's new Booking is routed into the standard deposit-payment step |
| FEAT-04.SPEC-005 (Booking-to-Calendar Sync) -- within FEAT-04 (Two-Way Calendar Sync) | Affects (outbound) | This commit is the event the Pro's personal calendar mirrors |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | This commit is the event the append-only activity log records |
| FEAT-25.SPEC-004 (Historical Aggregate Maintenance) -- within FEAT-25 (Booking & Revenue Insights) | Affects (outbound) | A cancellation, outside-window reschedule, or the original Booking's terminal Rescheduled transition in a late reschedule is the event the rolling insights aggregates reverse or move a booking count for; the late reschedule's new Booking is counted separately, only once it reaches Confirmed |
| FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching) -- within FEAT-20 (Waitlist for Cancelled Slots) | Affects (outbound) | A cancellation commit is the event that frees the slot for waitlist priority |
| FEAT-10.SPEC-006 (Cancellation/Reschedule Notification) | Triggers (outbound) | A successful commit fires the client confirmation and Pro change notice |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Affects (outbound) | The client is routed here after a successful commit (except a late reschedule) |

## Analytics and Success Signals

- **booking_cancelled_by_client** (Booking reference, window_state: outside/inside) -- supports success-metrics.md: "Self-Service Reschedule Rate"
- **booking_rescheduled_by_client** (Booking reference, window_state: outside/inside) -- supports success-metrics.md: "Self-Service Reschedule Rate"
- **booking_update_commit_conflict** (action_type: cancel/reschedule) -- N/A -- no Stage 2 metric measures contention-loss rate specifically; retained per operational visibility into how often the Booking entity's reject-with-refresh resolution is exercised on the client side
- **booking_update_commit_failed** (action_type: cancel/reschedule) -- supports success-metrics.md: "Self-Service Reschedule Rate" (a failed commit that cannot be completed self-service is exactly the gap this metric measures against)

## Acceptance Criteria

**FEAT-10.SPEC-004-AC-01:** Given Riley confirms a cancellation on an eligible booking, when this automation commits, then the Booking's state is written to Cancelled by Client with the cancellation timestamp set, and FEAT-09.SPEC-004 is handed the action.

**FEAT-10.SPEC-004-AC-02:** Given Riley confirms a reschedule outside the cancellation window, when this automation commits, then the same Booking record's start_time is updated to the new time, its state is unchanged, and FEAT-09.SPEC-004 is handed the action.

**FEAT-10.SPEC-004-AC-03:** Given Riley confirms a reschedule inside the cancellation window, when this automation commits, then the original Booking's state is written to Rescheduled and a new Booking record is created at the new time, flagged as requiring its own fresh deposit.

**FEAT-10.SPEC-004-AC-04:** Given Riley confirms a cancel or reschedule and a Pro-side transition already committed against the same Booking moments earlier, when the eligibility re-check runs, then this attempt is rejected with the conflict outcome and no Booking record is changed by this attempt.

**FEAT-10.SPEC-004-AC-05:** Given Riley confirms a reschedule and the chosen new time is taken by another client between selection and commit, when the final slot re-validation runs, then this attempt is rejected with the slot-lost outcome and no Booking record is changed.

**FEAT-10.SPEC-004-AC-06:** Given Riley confirms a cancel or reschedule and the write itself fails to save, when the failure occurs, then the original Booking remains exactly as it was, with no partial state change.

**FEAT-10.SPEC-004-AC-07:** Given the inside-window compound write fails partway through (original transitioned but the new Booking's creation does not complete), when the failure is detected, then the entire attempt is rolled back as one unit -- the original Booking is left untransitioned and no new Booking exists.

**FEAT-10.SPEC-004-AC-08:** Given a successful cancellation commits, when the commit completes, then FEAT-10.SPEC-006 is triggered for the client confirmation and Pro change notice.

**FEAT-10.SPEC-004-AC-09:** Given a successful outside-window reschedule commits, when the commit completes, then the client is routed to FEAT-06.SPEC-004 showing the updated time.

**FEAT-10.SPEC-004-AC-10:** Given a successful inside-window reschedule commits, when the commit completes, then the client is routed into FEAT-07's deposit-payment step for the newly created Booking.

**FEAT-10.SPEC-004-AC-11:** Given a client cancellation and a Pro no-show marking are both attempted on the same Booking at effectively the same time, when both reach this automation's domain, then only the first to commit succeeds, and the second is rejected by the eligibility re-check.

**FEAT-10.SPEC-004-AC-12:** Given this automation is processing a commit for one Booking, when a separate commit attempt fires for a different Booking at the same time, then the two proceed independently and neither is delayed by the other.

**FEAT-10.SPEC-004-AC-13:** Given a late reschedule's new Booking is created but never paid, when it is left unpaid, then it is governed by FEAT-03's standard slot-hold and expiration rules (XBR-02), exactly as any other unpaid booking.

**FEAT-10.SPEC-004-AC-14:** Given Riley's booking's bound policy version cannot be read at the instant of commit, when this automation attempts the write, then the commit is held and the triggering screen shows its ordinary error/retry handling rather than writing an outcome-less transition.

**FEAT-10.SPEC-004-AC-15:** Given a new Booking is created for a late reschedule, when its policy_version is set, then it acknowledges the version current at that moment, never the original Booking's bound version.

**FEAT-10.SPEC-004-AC-16:** Given a cancellation commit succeeds, when the commit completes, then FEAT-20 and FEAT-04 each react independently to the commit (freed-slot handoff and calendar mirroring), with no calendar or waitlist write performed by this automation itself; FEAT-25.SPEC-004 likewise reacts independently to any committed cancellation or reschedule, and this automation performs no insights aggregate write.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 6 | 6 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
