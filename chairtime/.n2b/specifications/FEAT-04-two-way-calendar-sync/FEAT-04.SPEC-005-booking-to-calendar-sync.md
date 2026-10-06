---
document_type: spec
spec_type: automation
spec_id: FEAT-04.SPEC-005
spec_name: Booking-to-Calendar Sync
spec_slug: booking-to-calendar-sync
parent_feature: FEAT-04
parent_feature_name: Two-Way Calendar Sync
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 13
---

# Automation Spec: Booking-to-Calendar Sync

## Overview

**Name:** Booking-to-Calendar Sync
**ID:** FEAT-04.SPEC-005
**Type:** Automation
**Purpose:** On any Chairtime booking create, reschedule, or cancel, writes, moves, or removes the matching event on the Pro's connected personal calendar, so the personal calendar never shows a stale or ghost appointment.
**Parent Feature:** FEAT-04 -- Two-Way Calendar Sync

## Scope and Non-Goals

**In Scope:**
- Writing a new calendar event when a Chairtime booking is confirmed
- Moving the matching calendar event when a booking is rescheduled
- Removing the matching calendar event when a booking is cancelled
- Retrying a failed write/move/remove operation and flagging persistent failure
- Confirming no calendar action is needed for a booking marked no-show

**Non-Goals:**
- Performing the actual write/move/remove operation against the calendar provider -- owned by FEAT-04.SPEC-003 (Calendar Provider Sync); this automation decides what operation is needed and instructs that spec to perform it
- Reading personal-calendar busy time to block Chairtime availability -- owned by FEAT-04.SPEC-004 (Busy-Time Availability Feed); this automation is the reverse direction (Chairtime-to-calendar)
- Deciding a connection's health status -- owned by FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation); a failed write here feeds that spec's monitoring, it does not set health status itself
- Notifying the client that their booking changed -- owned by FEAT-08 (Automated Booking Messaging); this automation's scope is the Pro's personal calendar only

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Booking confirmed | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation), completing the FEAT-05 (Public Booking Page & Booking Flow) checkout | Fires when a booking transitions to Confirmed | Booking reference: service, start_time, duration; Pro's connected calendar connection(s) |
| Booking confirmed (Pro-initiated) | FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold), confirmed through FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Fires when the Pro books a client in directly and the booking is confirmed | Booking reference: service, start_time, duration |
| Booking rescheduled (client-initiated) | FEAT-10.SPEC-004 (Booking Update Commit) (Client-Initiated Cancel/Reschedule) | Fires when a client reschedules their own booking to a new time | Booking reference with new start_time; previous start_time for locating the existing calendar event |
| Booking rescheduled (Pro-initiated) | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) (Pro Booking Management) | Fires when the Pro reschedules a booking | Booking reference with new start_time; previous start_time |
| Booking cancelled (client-initiated) | FEAT-10.SPEC-004 (Booking Update Commit) (Client-Initiated Cancel/Reschedule) | Fires when a client cancels their own booking | Booking reference; original start_time for locating the existing calendar event |
| Booking cancelled (Pro-initiated) | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) / FEAT-30.SPEC-008 (Bulk Cancellation Commit) (Pro Booking Management) | Fires when the Pro cancels a booking | Booking reference; original start_time |
| Booking marked no-show | FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) | Fires when a booking is marked no-show | Booking reference |

## Processing Logic

1. Receive the triggering Booking reference and the lifecycle event (confirmed, rescheduled, cancelled, marked no-show).
2. Look up the Pro Account's connected calendar connection(s) (at most two, one per kind).
3. If the Pro has no connected calendar, stop -- there is nothing to sync (no-op, not a failure).
4. If the event is "marked no-show," confirm no calendar action is required (the appointment already occurred) and stop -- this path exists to make the no-action decision explicit, not to skip it silently.
5. If the event is "confirmed," instruct FEAT-04.SPEC-003 to write a new calendar event containing the service name, start_time, and duration for every connected calendar.
6. If the event is "rescheduled," instruct FEAT-04.SPEC-003 to move the existing calendar event (located by the booking's previous start_time and service) to the new start_time, for every connected calendar.
7. If the event is "cancelled," instruct FEAT-04.SPEC-003 to remove the existing calendar event (located by the booking's start_time and service), for every connected calendar.
8. Await the write/move/remove confirmation from FEAT-04.SPEC-003 for each connected calendar independently.
9. On confirmed success for a given connection, record the operation as complete for that connection.
10. On failure for a given connection, retry per the failure-path rules in Outcome Definitions; other connected calendars are unaffected by one connection's failure.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Event written | Booking confirmed, write succeeds | Calendar Connection -- last_successful_sync updated for the connection | None directly to the Pro on this automation's own surface; the appointment simply appears on the personal calendar | FEAT-04.SPEC-003 (Calendar Provider Sync) |
| Event moved | Booking rescheduled, move succeeds | Calendar Connection -- last_successful_sync updated | None directly; the appointment's time updates on the personal calendar | FEAT-04.SPEC-003 (Calendar Provider Sync) |
| Event removed | Booking cancelled, remove succeeds | Calendar Connection -- last_successful_sync updated | None directly; the appointment disappears from the personal calendar, leaving no ghost entry | FEAT-04.SPEC-003 (Calendar Provider Sync) |
| No action needed (no-show) | Booking marked no-show | None -- explicitly no calendar action | None -- the appointment already occurred and its calendar entry (if any) is left as-is | -- |
| No calendar connected | Pro has zero connections at trigger time | None | None -- this is a normal, expected state for a Pro who has not connected a calendar (calendar connection is optional per XBR-26) | -- |
| Operation retried | Write/move/remove fails on first attempt | No calendar-side change yet; Chairtime booking state is unaffected | None visible to the Pro during retry; the operation is not user-facing at this stage | FEAT-04.SPEC-003 (Calendar Provider Sync) |
| Operation failed after retries | Write/move/remove fails after exhausting retries | No calendar-side change; Chairtime booking state remains authoritative and unchanged | The Pro's dashboard attention list (FEAT-12) shows a flag: "Couldn't update your calendar for {service} on {date}. Your Chairtime booking is correct -- try reconnecting your calendar." | FEAT-12 (Pro Daily Schedule Dashboard), FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) |

## Data Model

**Reads:** Booking -- service, start_time, duration, state (from FEAT-05, FEAT-10, FEAT-11, FEAT-30); Calendar Connection -- calendar_kind, status for the Pro Account.
**Creates:** None directly -- calendar events are created by FEAT-04.SPEC-003, not persisted as a Chairtime entity.
**Updates:** None directly -- Calendar Connection.last_successful_sync is updated by FEAT-04.SPEC-003 as the operation's confirming spec.
**Deletes:** None directly -- calendar event removal is performed by FEAT-04.SPEC-003.

## Business Rules

- XBR-13: every booking created, rescheduled, or cancelled by either party is written, moved, or removed on the Pro's personal calendar; this automation is the mechanism that enforces that rule for every lifecycle event.
- A booking marked no-show never triggers a calendar action, since the appointment already occurred -- this is a deliberate no-op, not an unhandled case.
- If the Pro has two connections (Google and Apple), the same booking event is written, moved, or removed on both independently; one connection's failure does not block or roll back the operation on the other.
- Client identity and notes never appear in the calendar event content, per FEAT-04.SPEC-003's Data Exchanged section -- only service name, start time, and duration.

## Edge Cases

- **A booking is rescheduled twice in quick succession** -- The second reschedule's move instruction supersedes the first: FEAT-04.SPEC-003 is instructed to move the calendar event to the final new start_time directly, never producing two move operations that could leave the calendar event at an intermediate time.
- **A booking is cancelled immediately after being confirmed, before the write completes** -- The cancel instruction waits for the write to either complete or fail; once the write's outcome is known, the remove instruction is issued against the now-existing (or never-created) event, so no ghost event and no failed removal-of-nothing occurs.
- **Calendar connection is disconnected while a write/move/remove is in flight** -- Per FEAT-04.SPEC-008's contention rule, the disconnect wins: the in-flight operation is discarded, and no retry is attempted against a connection that no longer exists.
- **Concurrent trigger firing (two different bookings for the same Pro change at effectively the same time)** -- Each booking's write/move/remove is instructed independently to FEAT-04.SPEC-003; the calendar-sync capability applies both as separate events, with no interference between them.
- **Trigger fires while a previous run is in flight for the same booking** -- A second lifecycle event for the same booking (e.g., a reschedule arriving while the original confirm's write is still in flight) waits for the in-flight operation's outcome before issuing its own instruction, per the reschedule-in-quick-succession handling above; operations for different bookings never queue behind each other.
- **Pro connects a calendar after several bookings already exist** -- No retroactive backfill occurs from this automation; only bookings whose lifecycle event fires after the connection exists are synced. (Existing bookings created before the connection are unaffected, consistent with the dependency map's Booking lifecycle being independent of Calendar Connection's.)

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) -- within FEAT-05 (Public Booking Page & Booking Flow) checkout | Triggered by (inbound) | Booking confirmation fires the write path |
| FEAT-10.SPEC-004 (Booking Update Commit) (Client-Initiated Cancel/Reschedule) | Triggered by (inbound) | Client reschedule/cancel fires the move/remove paths |
| FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) | Triggered by (inbound) | No-show marking fires the explicit no-action confirmation |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit), FEAT-30.SPEC-008 (Bulk Cancellation Commit), FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) (Pro Booking Management) | Triggered by (inbound) | Pro-initiated confirm/reschedule/cancel fires the corresponding path |
| FEAT-04.SPEC-003 (Calendar Provider Sync) | Triggers (outbound) | Every write/move/remove instruction is carried out through this spec |
| FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) | Affects (outbound) | Persistent write/move/remove failures feed this spec's monitoring |
| FEAT-04.SPEC-008 (Calendar Connection Rules) | References (inbound) | Disconnect-wins-over-in-flight-sync contention rule governs in-flight operations |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | Persistent failure flag surfaces on the dashboard's attention list |

## Analytics and Success Signals

- **booking_calendar_write_attempted** (operation: write / move / remove, calendar_kind) -- supports success-metrics.md: "Calendar Sync Reliability"
- **booking_calendar_write_succeeded** (operation, calendar_kind, latency_bucket: within target / slow) -- supports success-metrics.md: "Calendar Sync Reliability"
- **booking_calendar_write_failed_after_retries** (operation, calendar_kind) -- supports success-metrics.md: "Calendar Sync Reliability"
- **booking_calendar_no_action_confirmed** (reason: no_show / no_connection) -- N/A -- no Stage 2 metric measures the no-action path directly; retained so the deliberate no-op is distinguishable from a missed sync in operational review.

## Acceptance Criteria

**FEAT-04.SPEC-005-AC-01:** Given Talia has a Google calendar connected and a client confirms a booking via FEAT-05, when the confirmation fires, then this automation instructs FEAT-04.SPEC-003 to write a new event on the Google calendar with the service, start time, and duration.

**FEAT-04.SPEC-005-AC-02:** Given a booking has a calendar event written on Talia's Apple calendar, when the client reschedules the booking via FEAT-10, then this automation instructs FEAT-04.SPEC-003 to move the existing event to the new time.

**FEAT-04.SPEC-005-AC-03:** Given a booking has a calendar event on Talia's connected calendar, when Talia cancels the booking via FEAT-30, then this automation instructs FEAT-04.SPEC-003 to remove the matching event so no ghost appointment remains.

**FEAT-04.SPEC-005-AC-04:** Given a booking passes its appointment time, when it is marked no-show via FEAT-11, then this automation confirms no calendar action is needed and takes none.

**FEAT-04.SPEC-005-AC-05:** Given Talia has no calendar connected, when a booking is confirmed, then this automation completes with no calendar action and no error, since calendar connection is optional.

**FEAT-04.SPEC-005-AC-06:** Given Talia has both a Google and an Apple connection, when a booking is confirmed, then this automation instructs a write for both connections independently, and a failure on one does not affect the other.

**FEAT-04.SPEC-005-AC-07:** Given a calendar write fails on first attempt, when the automation retries, then the underlying Chairtime booking remains Confirmed and unaffected throughout the retry.

**FEAT-04.SPEC-005-AC-08:** Given a calendar write fails after exhausting retries, when the failure is final, then Talia's dashboard attention list (FEAT-12) shows "Couldn't update your calendar for {service} on {date}. Your Chairtime booking is correct -- try reconnecting your calendar."

**FEAT-04.SPEC-005-AC-09:** Given a booking is rescheduled twice within moments of each other, when both reschedules process, then the calendar event ends at the final new time, with no intermediate move left stranded.

**FEAT-04.SPEC-005-AC-10:** Given a booking is cancelled immediately after confirmation, before its write completes, when both events process, then the remove instruction is issued only after the write's outcome is known, leaving no ghost event.

**FEAT-04.SPEC-005-AC-11:** Given Talia disconnects her calendar while a write is in flight for a just-confirmed booking, when the disconnect completes, then the in-flight write is discarded and not retried against the removed connection.

**FEAT-04.SPEC-005-AC-12:** Given two different bookings for Talia change at effectively the same time, when both lifecycle events fire, then each is instructed to FEAT-04.SPEC-003 independently with no interference between them.

**FEAT-04.SPEC-005-AC-13:** Given Talia connects a calendar after several bookings already exist, when the connection completes, then none of those pre-existing bookings are retroactively written to the calendar -- only bookings whose lifecycle event fires afterward are synced.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 7 | 7 |
| Outcome Paths | 7 | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
