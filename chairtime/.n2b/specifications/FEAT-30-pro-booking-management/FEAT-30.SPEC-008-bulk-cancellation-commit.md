---
document_type: spec
spec_type: automation
spec_id: FEAT-30.SPEC-008
spec_name: Bulk Cancellation Commit
spec_slug: bulk-cancellation-commit
parent_feature: FEAT-30
parent_feature_name: Pro Booking Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Automation Spec: Bulk Cancellation Commit

## Overview

**Name:** Bulk Cancellation Commit
**ID:** FEAT-30.SPEC-008
**Type:** Automation
**Purpose:** Commits a Pro-initiated cancellation of several bookings at once, reporting a per-booking outcome and coordinating each booking's refund, calendar removal, and client notice independently.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Committing Cancelled by Pro to every booking Talia selects from a reviewed conflict set
- Processing each booking independently so one booking's failure never blocks or rolls back the others
- Reporting a per-booking success/failure outcome back to the triggering screen
- Refunding any already-paid Balance Payment per affected booking (XBR-23, v1)
- Coordinating each booking's calendar removal, activity logging, freed-slot handoff, and client notice

**Non-Goals:**
- Reviewing or presenting the conflicting booking set to Talia -- owned by FEAT-30.SPEC-005 (Cancel Several Bookings at Once), which this automation is triggered by
- A single-booking cancellation or reschedule -- owned by FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit), a distinct processing shape (one booking, one outcome) from this automation's per-item outcome tracking
- Determining or executing the deposit refund itself for each booking -- owned by FEAT-09 for the refund-outcome evaluation this automation hands off to, per XBR-09
- Rolling back bookings that already succeeded when a later booking in the same set fails -- excluded per the Brief's States field ("a multi-booking cancellation reports the outcome for each booking"): each booking's outcome is independent and final once committed, never undone by a sibling's failure

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia confirms "cancel all" on the bulk-cancellation review screen | FEAT-30.SPEC-005 (Cancel Several Bookings at Once) | Fires when Talia confirms cancelling the reviewed set of bookings a new Time Block conflicts with | The set of Booking references to cancel, optional shared private Pro reason |

## Processing Logic

1. Receive the set of Booking references from FEAT-30.SPEC-005's confirm action.
2. For each Booking in the set, independently:
   a. Re-check eligibility against FEAT-30.SPEC-006 (ownership, and Booking.state is Confirmed or Awaiting Outcome) immediately before the write.
   b. If eligibility fails (a conflicting transition already committed, or the booking is no longer eligible for another reason), mark this booking's outcome as Failed with the specific denial reason and continue to the next booking without affecting it.
   c. If eligibility passes, set Booking.state to Cancelled by Pro and record the cancellation timestamp and any shared private Pro reason.
   d. Check whether a Balance Payment exists with status Succeeded (v1); if so, request its full refund (XBR-23) through FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund).
   e. Hand off the cancellation event to FEAT-09 for deposit-outcome evaluation (always full refund, per XBR-09).
   f. Trigger the calendar mirror to remove the corresponding personal-calendar entry (FEAT-04, XBR-13).
   g. Write an append-only activity event for this booking (FEAT-16, XBR-21).
   h. Hand the freed slot to FEAT-20's waitlist-priority check before it returns to general availability (XBR-28).
   i. Trigger FEAT-30.SPEC-012 (Pro Action Client Notice) for this booking's client.
   j. Mark this booking's outcome as Succeeded.
3. Once every booking in the set has been processed, return the complete per-booking outcome report to FEAT-30.SPEC-005.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| All bookings cancelled | Every booking in the set passes eligibility | Every Booking.state -> Cancelled by Pro | Talia sees a per-booking success summary on FEAT-30.SPEC-005; each client receives a cancellation-with-refund notice | FEAT-30.SPEC-005, FEAT-09, FEAT-04, FEAT-16, FEAT-20, FEAT-30.SPEC-012 |
| Partial success -- one or more bookings fail | At least one booking in the set fails eligibility while others pass | Only the passing bookings' states change | Talia sees which specific booking(s) failed and why, alongside the successful cancellations, on FEAT-30.SPEC-005; she can act on the failed one separately (e.g., via FEAT-30.SPEC-001) | FEAT-30.SPEC-005, FEAT-30.SPEC-001 |
| A single booking's Balance Payment refunded (v1) | That booking has a Succeeded Balance Payment | Balance Payment.state -> Refunded for that booking | Included in that booking's client notice, never a separate message | FEAT-22, FEAT-30.SPEC-012 |
| Whole-set write failure (processing error before any booking commits) | The commit cannot begin for a reason other than a per-booking eligibility conflict | No data changes | Talia sees a retry prompt on FEAT-30.SPEC-005; no booking in the set is affected | FEAT-30.SPEC-005 |

## Data Model

**Reads:** Booking (state, start_time, owning Pro Account) for each booking in the set; Deposit Transaction (status, per booking); Balance Payment (status, per booking, v1).
**Creates:** Activity Event (FEAT-16) -- one per successfully cancelled booking.
**Updates:** Booking -- state to Cancelled by Pro, plus cancellation timestamp and optional shared reason, per successfully processed booking. Balance Payment -- state to Refunded where one exists and has Succeeded, per booking (v1).
**Deletes:** None -- every cancelled booking is retained as history (SC-22).

## Business Rules

- Each booking in the set is processed and evaluated fully independently -- no booking's outcome depends on or is rolled back by another's, per the Brief's per-booking outcome-reporting requirement.
- XBR-09, XBR-13, XBR-21, XBR-23, and XBR-28 apply identically to each booking in the set as they do to a single Pro-initiated cancellation (FEAT-30.SPEC-007) -- this automation differs only in operating over several bookings under one confirm action.
- A booking that fails eligibility is reported, never silently dropped -- Talia always sees exactly which booking failed and why, consistent with pipeline-rules.md's output-completeness expectation carried into product behavior.
- The set's shared private Pro reason (if provided) is recorded identically on every successfully cancelled booking in the set; it is never inferred or altered per booking.

## Edge Cases

- **One booking in the set was already completed by the Auto-Completion Sweep moments before this commit runs** -- That booking's eligibility check fails with the completed-state message from FEAT-30.SPEC-006; it is reported as a failed outcome while the remaining bookings in the set are cancelled normally.
- **A client cancels one of the set's bookings through FEAT-10 in the instant before this commit processes it** -- Reject-with-refresh for that single booking: it is reported as a failed outcome (already resolved by the client), and every other booking in the set is unaffected.
- **Every booking in the set fails eligibility** -- The outcome report shows every booking as failed with its specific reason; no client notices fire, and Talia sees the full set needs a separate look.
- **The set contains only one booking** -- Processed identically to a full multi-booking set; the per-booking outcome mechanics do not special-case a set of size one, though the triggering screen (FEAT-30.SPEC-005) is the one that decides when this path versus FEAT-30.SPEC-007 applies.
- **Concurrent trigger firing (two different bulk-cancel confirms from two different time-block conflicts, with no overlapping bookings)** -- Each bulk commit processes its own distinct set independently; no interference occurs since the underlying bookings do not overlap.
- **Trigger fires while a previous bulk commit for the same set is still in flight** -- FEAT-30.SPEC-005's confirm control is disabled during submission, preventing a duplicate bulk-commit request for the same reviewed set.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-005 (Cancel Several Bookings at Once) | Triggered by (inbound) / Affects (outbound) | Confirm triggers this commit; the per-booking outcome report is shown here |
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (outbound) | Per-booking eligibility, ownership, and state-cutoff rules re-checked before each write |
| FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated) | Affects (outbound) | A failed booking in the set can be revisited individually through this screen |
| FEAT-09 (Cancellation & No-Show Policy Engine) | Triggers (outbound) | Hands off each booking's cancellation event for deposit-outcome evaluation and execution (XBR-09) |
| FEAT-04.SPEC-005 (Booking-to-Calendar Sync) -- within FEAT-04 (Two-Way Calendar Sync) | Triggers (outbound) | Each successfully cancelled booking's calendar entry is removed |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Triggers (outbound) | Each successfully cancelled booking writes its own activity event |
| FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching) -- within FEAT-20 (Waitlist for Cancelled Slots) | Triggers (outbound) | Each freed slot is handed to waitlist-priority notification first |
| FEAT-30.SPEC-012 (Pro Action Client Notice) | Triggers (outbound) | Each successfully cancelled booking's client receives the cancellation-with-refund notice |
| FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund) -- within FEAT-22 (In-App Balance Payment) | Triggers (outbound, v1) | Any already-succeeded Balance Payment per booking is refunded alongside the deposit |
| FEAT-17 (Manual Time Blocking) | Triggered by (inbound, indirect) | The Time Block whose conflicting bookings Talia reviewed on FEAT-30.SPEC-005 originates here |

## Analytics and Success Signals

- **bulk_cancellation_completed** (booking_count, success_count, failure_count) -- supports success-metrics.md: "Pro Change Correctness"
- **booking_cancelled_by_pro** (context: bulk) -- supports success-metrics.md: "Pro Change Correctness" (per-booking event, emitted once for each booking successfully cancelled within the set)
- **bulk_cancellation_booking_failed** (reason category) -- supports success-metrics.md: "Automatic Refund Correctness" (a failed booking within a bulk action must never be silently dropped; this event measures how often the per-booking failure path is exercised)

## Acceptance Criteria

**FEAT-30.SPEC-008-AC-01:** Given Talia confirms "cancel all" on four bookings her new time block conflicts with, when this commit runs and all four pass eligibility, then all four transition to Cancelled by Pro, and Talia sees a success summary for all four on FEAT-30.SPEC-005.

**FEAT-30.SPEC-008-AC-02:** Given one of the four bookings was already completed before this commit runs, when the set is processed, then that booking is reported as a failed outcome with the completed-state reason while the other three are cancelled successfully.

**FEAT-30.SPEC-008-AC-03:** Given a booking in the set is successfully cancelled, when the commit processes it, then its personal-calendar entry is removed, an activity event is written, its freed slot is handed to FEAT-20, and its client receives the cancellation-with-refund notice.

**FEAT-30.SPEC-008-AC-04:** Given a booking in the set already has a Succeeded Balance Payment (v1), when it is cancelled, then that Balance Payment is refunded in full alongside its deposit.

**FEAT-30.SPEC-008-AC-05:** Given Talia provides a shared private reason when confirming the bulk cancellation, when each booking is cancelled, then that same reason is recorded on every successfully cancelled booking in the set.

**FEAT-30.SPEC-008-AC-06:** Given a client cancels one of the set's bookings through FEAT-10 moments before this commit processes it, when that booking is evaluated, then it is reported as a failed outcome (reject-with-refresh) and the remaining bookings in the set are unaffected.

**FEAT-30.SPEC-008-AC-07:** Given every booking in the reviewed set fails eligibility, when the commit runs, then every booking is reported as failed with its specific reason and no client notices fire.

**FEAT-30.SPEC-008-AC-08:** Given the set contains exactly one booking, when this automation processes it, then the single-booking outcome mechanics apply identically to a larger set.

**FEAT-30.SPEC-008-AC-09:** Given the whole-set commit cannot begin due to a processing error, when the failure occurs, then Talia sees a retry prompt on FEAT-30.SPEC-005 and no booking in the set is affected.

**FEAT-30.SPEC-008-AC-10:** Given two different bulk-cancel confirms fire at effectively the same time for two non-overlapping sets, when both commits run, then each processes its own set independently with no interference.

**FEAT-30.SPEC-008-AC-11:** Given a bulk commit is already in flight for a reviewed set, when Talia's confirm control is tapped again before it resolves, then no duplicate commit is submitted, since the control is disabled during submission.

**FEAT-30.SPEC-008-AC-12:** Given a booking that failed within a bulk cancellation, when Talia looks for a way to act on it separately, then she can reach FEAT-30.SPEC-001 for that individual booking.

**FEAT-30.SPEC-008-AC-13:** Given all bookings in the set succeed, when the final outcome is reported, then Talia's summary distinguishes success from failure per booking rather than showing one aggregate result.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
