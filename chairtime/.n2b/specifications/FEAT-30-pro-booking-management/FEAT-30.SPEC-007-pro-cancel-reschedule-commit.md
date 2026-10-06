---
document_type: spec
spec_type: automation
spec_id: FEAT-30.SPEC-007
spec_name: Pro Cancel/Reschedule Commit
spec_slug: pro-cancel-reschedule-commit
parent_feature: FEAT-30
parent_feature_name: Pro Booking Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 16
---

# Automation Spec: Pro Cancel/Reschedule Commit

## Overview

**Name:** Pro Cancel/Reschedule Commit
**ID:** FEAT-30.SPEC-007
**Type:** Automation
**Purpose:** Commits a Pro-initiated single cancellation or reschedule to the Booking, coordinating the deposit-outcome handoff, calendar mirroring, activity logging, freed-slot handoff, and client notice this triggers.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Committing a Pro-initiated single cancellation (Confirmed/Awaiting Outcome -> Cancelled by Pro)
- Committing a Pro-initiated single reschedule (updating start_time in place, per the Brief's flagged-not-resolved lifecycle discrepancy, carried forward here for Stage 4)
- Handing off the deposit-outcome determination to FEAT-09 (always full refund for a Pro cancellation or reschedule, per XBR-09)
- Refunding an already-paid Balance Payment in full, if one exists (XBR-23, v1)
- Triggering calendar mirroring, activity logging, freed-slot/waitlist handoff, historical-aggregate maintenance, and the client notice this commit causes
- Requesting a fresh booking-specific manage link on a reschedule (XBR-18), issued by FEAT-08.SPEC-010
- Rejecting a commit attempt against a booking a conflicting transition has already resolved (reject-with-refresh)

**Non-Goals:**
- Determining or executing the deposit refund itself -- owned by FEAT-09 (Cancellation & No-Show Policy Engine, FEAT-09.SPEC-004/FEAT-09.SPEC-005); this automation only hands off the cancellation/reschedule event, per XBR-09 ("any Pro cancellation = full refund")
- The eligibility check for whether this action is currently allowed -- owned by FEAT-30.SPEC-006 (Pro Booking Action Rules); this automation re-checks it once more immediately before the atomic write, per that spec's Enforced By table
- Committing several bookings in one action -- owned by FEAT-30.SPEC-008 (Bulk Cancellation Commit), a distinct per-booking-outcome processing shape
- Composing or delivering the client's change notice -- FEAT-30.SPEC-012 is the trigger-and-audience contract and FEAT-08.SPEC-004 owns the content; this automation triggers it but does not define its content
- Generating or storing the fresh manage link -- owned by FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance); this automation only requests it after a reschedule commits

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia confirms a single cancellation | FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated) | Fires when Talia taps confirm on the cancel screen | Booking reference, optional private Pro reason |
| Talia confirms a single reschedule | FEAT-30.SPEC-002 (Reschedule Booking, Pro-Initiated) | Fires when Talia taps confirm on the reschedule screen, having selected a validated new time | Booking reference, new start_time, duration (unchanged) |

## Processing Logic

1. Receive the triggering action (cancel or reschedule) and the Booking reference from the triggering screen.
2. Re-check eligibility against FEAT-30.SPEC-006 (ownership, and Booking.state is Confirmed or Awaiting Outcome) immediately before the write. If eligibility fails because a conflicting transition already committed, stop and return the reject-with-refresh outcome.
3. **Cancellation path:** Set Booking.state to Cancelled by Pro; record the cancellation timestamp and any optional private Pro reason.
4. **Reschedule path:** Update Booking.start_time to the new, already-validated time; record the reschedule timestamp and any optional private Pro reason. Booking.state is not changed by a reschedule.
5. Check whether a Balance Payment exists for this Booking with status Succeeded (v1; at MVP this check always finds no record, per SC-16). If one exists, request its full refund (XBR-23) through FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund), which owns the outbound balance refund; the deposit leg continues to run through its own refund path.
6. Hand off the cancellation-or-reschedule event to FEAT-09 for deposit-outcome evaluation; FEAT-09.SPEC-004 always determines a full-refund outcome for a Pro-initiated action (XBR-09), and FEAT-09.SPEC-005 executes it.
7. Trigger the Pro's personal calendar mirror to remove (cancellation) or move (reschedule) the corresponding entry (FEAT-04, XBR-13).
8. **Reschedule only:** Trigger FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) to issue a fresh manage link for this Booking and supersede the previous one (XBR-18); the fresh link is carried into the client notice in step 12.
9. Write an append-only activity event recording the action, its timestamp, and the actor (Talia) (FEAT-16, XBR-21).
10. **Cancellation only:** Hand the freed slot to FEAT-20's waitlist-priority check before it returns to general public availability (XBR-28).
11. Signal FEAT-25.SPEC-004 (Historical Aggregate Maintenance) that the committed cancellation or in-place reschedule changes the Booking's counted state, so revenue and insights aggregates stay current.
12. Trigger FEAT-30.SPEC-012 (Pro Action Client Notice) with the outcome type (cancelled-with-refund or rescheduled-with-new-time), the resulting deposit status, and (reschedule) the fresh manage link from step 8.
13. Return the committed outcome to the triggering screen for its success feedback.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Cancellation committed | Eligibility passes at write time | Booking.state -> Cancelled by Pro; cancellation timestamp and optional reason recorded | Talia sees the cancellation confirmed on FEAT-30.SPEC-001 and returns to her schedule; Riley receives the cancellation-with-refund notice | FEAT-30.SPEC-001, FEAT-09, FEAT-04, FEAT-16, FEAT-20, FEAT-25.SPEC-004, FEAT-30.SPEC-012 |
| Reschedule committed | Eligibility passes at write time | Booking.start_time updated; reschedule timestamp and optional reason recorded; a fresh manage link issued for the Booking and the previous link superseded (FEAT-08.SPEC-010, XBR-18) | Talia sees the reschedule confirmed on FEAT-30.SPEC-002; Riley receives the reschedule-with-new-time notice carrying the fresh manage link | FEAT-30.SPEC-002, FEAT-09, FEAT-04, FEAT-16, FEAT-08.SPEC-010, FEAT-25.SPEC-004, FEAT-30.SPEC-012 |
| Already-paid balance refunded (v1) | A Balance Payment with status Succeeded exists for this Booking | Balance Payment.state -> Refunded | Included in the same client notice as the deposit outcome, never a separate message | FEAT-22, FEAT-30.SPEC-012 |
| Commit rejected -- conflicting transition already won | A client-side action (FEAT-10) commits a conflicting transition first | No data changes | Talia is shown the booking's current state on FEAT-30.SPEC-001/SPEC-002 and must re-decide; the two transitions are never merged | FEAT-30.SPEC-001, FEAT-30.SPEC-002 |
| Commit rejected -- booking no longer eligible | The eligibility re-check fails for a reason other than a concurrent conflict (e.g., the booking auto-completed moments earlier) | No data changes | Talia sees the exact denial message from FEAT-30.SPEC-006 on the triggering screen | FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-006 |
| Write failure (processing error) | The commit cannot be written for a reason other than an eligibility conflict | No data changes | Talia sees a retry prompt on the triggering screen; the booking remains exactly as it was | FEAT-30.SPEC-001, FEAT-30.SPEC-002 |

## Data Model

**Reads:** Booking (state, start_time, owning Pro Account); Deposit Transaction (status, for the eligibility pass-through FEAT-09 evaluates); Balance Payment (status, v1).
**Creates:** Activity Event (FEAT-16) -- one per committed action. On a reschedule, a fresh Access Link for the Booking is created by FEAT-08.SPEC-010 (this automation requests it and does not write it).
**Updates:** Booking -- state (cancellation only) and/or start_time (reschedule only), plus cancellation/reschedule timestamp and optional private Pro reason. Balance Payment -- state to Refunded when one exists and has Succeeded (v1).
**Deletes:** None -- a cancelled or rescheduled booking is retained as history (SC-22), never deleted.

## Business Rules

- XBR-09: any Pro cancellation refunds the client's deposit in full, whatever the timing; a Pro-made reschedule never exposes the client to the cancellation window -- the deposit always carries over.
- XBR-13: this commit's calendar mirror is triggered on every successful cancellation or reschedule, never silently skipped.
- XBR-21: every commit writes exactly one append-only activity event; the event is never edited or deleted afterward.
- XBR-23 (v1): an already-succeeded Balance Payment is refunded in full alongside the deposit whenever either party cancels -- never forfeited.
- XBR-28: a cancellation's freed slot is handed to waitlist-priority notification (FEAT-20) before it returns to general public availability; a reschedule does not free a slot in the same sense, since the booking continues to exist at its new time.
- XBR-18: every Pro-made reschedule issues a fresh booking-specific manage link through FEAT-08.SPEC-010, and the client notice carries that fresh link, never the superseded one; a cancellation issues no new link.
- The Booking entity's Contention resolution is reject-with-refresh (dependency map): the first committed transition wins, and the automation never merges a Pro-side and a client-side transition on the same booking.
- A reschedule never changes Booking.state -- only start_time and the reschedule timestamp/reason; this distinguishes it from a cancellation, which does transition state.

## Edge Cases

- **A client cancels the same booking through FEAT-10 in the instant before this commit runs** -- Reject-with-refresh: whichever transition commits first wins; the later commit attempt (this automation's) fails the eligibility re-check and Talia is shown the booking's current (client-cancelled) state, never a merged or overwritten outcome.
- **Talia reschedules a booking to a time that becomes contested between her selection and this commit** -- The candidate slot was already re-validated and held by the triggering screen (FEAT-30.SPEC-002, via FEAT-03); if the hold itself has since been lost, the commit fails and Talia is returned to slot selection with a refreshed list, per FEAT-03.SPEC-005.
- **Concurrent trigger firing (Talia cancels two different bookings at effectively the same time from two schedule rows)** -- Each commit processes independently against its own distinct Booking; no interference occurs since the bookings are unrelated records.
- **Trigger fires while a previous commit for the same booking is still in flight** -- The triggering screen's confirm control is disabled during submission (FEAT-30.SPEC-001/SPEC-002), preventing a duplicate commit request for the same action on the same booking.
- **A Balance Payment exists but has not yet succeeded (Attempted or Failed, v1)** -- No refund is requested against it; only a Succeeded Balance Payment is eligible for the automatic refund this commit triggers (XBR-23), consistent with there being nothing paid to reverse otherwise.
- **The booking being cancelled or rescheduled is the last one on Talia's day** -- No special handling; the commit, calendar mirror, activity log, and client notice proceed identically regardless of how many other bookings exist that day.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated) | Triggered by (inbound) / Affects (outbound) | Confirm triggers this commit; a rejected or failed commit is shown here |
| FEAT-30.SPEC-002 (Reschedule Booking, Pro-Initiated) | Triggered by (inbound) / Affects (outbound) | Confirm triggers this commit; a rejected or failed commit is shown here |
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (outbound) | Eligibility, ownership, and state-cutoff rules re-checked immediately before the write |
| FEAT-09 (Cancellation & No-Show Policy Engine) | Triggers (outbound) | Hands off the cancellation/reschedule event for deposit-outcome evaluation and execution (XBR-09) |
| FEAT-04.SPEC-005 (Booking-to-Calendar Sync) -- within FEAT-04 (Two-Way Calendar Sync) | Triggers (outbound) | A successful commit removes or moves the corresponding personal-calendar entry |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Triggers (outbound) | Every commit writes an append-only activity event |
| FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching) -- within FEAT-20 (Waitlist for Cancelled Slots) | Triggers (outbound) | A cancellation's freed slot is handed to waitlist-priority notification first |
| FEAT-30.SPEC-012 (Pro Action Client Notice) | Triggers (outbound) | A successful commit triggers the client's cancellation or reschedule notice (content owned by FEAT-08.SPEC-004) |
| FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) -- within FEAT-08 (Automated Booking Messaging) | Triggers (outbound) | A committed reschedule requests a fresh manage link for the Booking (XBR-18) |
| FEAT-25.SPEC-004 (Historical Aggregate Maintenance) -- within FEAT-25 (Booking & Revenue Insights) | Triggers (outbound) | A committed cancellation or reschedule updates the insights aggregates |
| FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund) -- within FEAT-22 (In-App Balance Payment) | Triggers (outbound, v1) | An already-succeeded Balance Payment is refunded alongside the deposit |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | References (outbound) | The source of a conflicting transition this automation may lose to, per the Booking entity's Contention rule |

## Analytics and Success Signals

- **booking_cancelled_by_pro** (has_reason: yes/no) -- supports success-metrics.md: "Pro Change Correctness"
- **booking_rescheduled_by_pro** () -- supports success-metrics.md: "Pro Change Correctness"
- **pro_commit_rejected** (reason: conflicting_transition / no_longer_eligible) -- supports success-metrics.md: "Automatic Refund Correctness" (a rejected commit must never leave the booking or its deposit in an ambiguous state; this event measures how often the reject-with-refresh path is exercised)
- **pro_commit_failed** (action: cancel / reschedule; reason category) -- N/A -- no Stage 2 metric measures processing failures directly; retained so a failed write is never silently unobservable.

## Acceptance Criteria

**FEAT-30.SPEC-007-AC-01:** Given Talia confirms a cancellation on a Confirmed booking she owns, when this commit runs, then Booking.state is set to Cancelled by Pro, the cancellation timestamp is recorded, and Riley's deposit refund is handed off to FEAT-09.

**FEAT-30.SPEC-007-AC-02:** Given Talia confirms a reschedule to an already-validated new time, when this commit runs, then Booking.start_time is updated, Booking.state is unchanged, and Riley's deposit carries over automatically per XBR-09.

**FEAT-30.SPEC-007-AC-03:** Given a successful cancellation or reschedule commit, when it completes, then the corresponding entry on Talia's personal calendar is removed (cancellation) or moved (reschedule).

**FEAT-30.SPEC-007-AC-04:** Given a successful commit, when it completes, then exactly one append-only activity event is written recording the action, timestamp, and Talia as the actor.

**FEAT-30.SPEC-007-AC-05:** Given a successful cancellation commit, when it completes, then the freed slot is handed to FEAT-20's waitlist-priority check before returning to general availability.

**FEAT-30.SPEC-007-AC-06:** Given a successful commit, when it completes, then FEAT-30.SPEC-012 fires the matching client notice for the action taken (cancellation-with-refund or reschedule-with-new-time).

**FEAT-30.SPEC-007-AC-07:** Given a booking already has a Succeeded Balance Payment (v1), when Talia cancels or reschedules it, then the Balance Payment is refunded in full alongside the deposit, per XBR-23.

**FEAT-30.SPEC-007-AC-08:** Given a booking has only an Attempted or Failed Balance Payment, when Talia cancels it, then no Balance Payment refund is requested, since nothing was actually paid.

**FEAT-30.SPEC-007-AC-09:** Given Riley cancels the same booking through FEAT-10 moments before Talia's commit runs, when this commit's eligibility re-check executes, then it is rejected with reject-with-refresh, and Talia is shown the booking's current (client-cancelled) state.

**FEAT-30.SPEC-007-AC-10:** Given a booking has auto-completed since Talia opened the cancel screen, when this commit's eligibility re-check runs, then it is rejected with the completed-state message from FEAT-30.SPEC-006, and no data changes.

**FEAT-30.SPEC-007-AC-11:** Given the commit cannot be written due to a processing error, when the write fails, then Talia sees a retry prompt on the triggering screen and the booking remains exactly as it was.

**FEAT-30.SPEC-007-AC-12:** Given Talia cancels two different bookings from two different schedule rows at effectively the same time, when both commits run, then each succeeds independently with no interference.

**FEAT-30.SPEC-007-AC-13:** Given a cancellation commit is already in flight for a booking, when Talia's confirm control is tapped again before it resolves, then no duplicate commit is submitted, since the control is disabled during submission.

**FEAT-30.SPEC-007-AC-14:** Given Talia's reschedule candidate slot loses its hold between selection and this commit's write, when the commit attempts to proceed, then it fails and Talia returns to FEAT-30.SPEC-002 with a refreshed slot list, per FEAT-03.SPEC-005.

**FEAT-30.SPEC-007-AC-15:** Given Talia confirms a reschedule and the commit succeeds, when the commit completes, then FEAT-08.SPEC-010 issues a fresh manage link for that Booking, the previous link no longer resolves as current, and the client notice triggered by FEAT-30.SPEC-012 carries the fresh link; a cancellation commit issues no new link.

**FEAT-30.SPEC-007-AC-16:** Given a cancellation or in-place reschedule commit succeeds, when it completes, then FEAT-25.SPEC-004 is signalled once with the Booking's Pro Account, service, and start_time so the insights aggregates reflect the change.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 6 | 6 |
| Business Rules | 8 | 8 |
| Edge Cases | 6 | 6 |
