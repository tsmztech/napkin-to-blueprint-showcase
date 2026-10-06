---
document_type: spec
spec_type: automation
spec_id: FEAT-08.SPEC-007
spec_name: Reminder Scheduling & Timing Window Enforcement
spec_slug: reminder-scheduling-timing-window-enforcement
parent_feature: FEAT-08
parent_feature_name: Automated Booking Messaging
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Reminder Scheduling & Timing Window Enforcement

## Overview

**Name:** Reminder Scheduling & Timing Window Enforcement
**ID:** FEAT-08.SPEC-007
**Type:** Automation
**Purpose:** Computes when each confirmed booking's reminder should fire, keeps every reminder inside the daytime send window (roughly 8am--9pm per XBR-16; platform parameter: `reminder-window-start-hour` to platform parameter: `reminder-window-end-hour`) in the Pro's timezone, and suppresses a separate reminder when a booking is made after its own reminder point has already passed.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- Computing the reminder send time for every confirmed Booking (default lead time before the appointment)
- Enforcing the daytime-hours send window and its nearest-allowed-time fallback
- Suppressing a reminder for a booking made after its own reminder point would already have passed
- Cancelling a scheduled reminder when the underlying booking is cancelled or rescheduled before the reminder fires

**Non-Goals:**
- The reminder's content and channel -- owned by FEAT-08.SPEC-002 (Appointment Reminder Message); this spec only decides when (or whether) that notification fires.
- Processing a client's reply once the reminder is sent -- owned by FEAT-08.SPEC-008 (Reminder Reply Routing).
- Retrying a failed reminder send -- owned by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback); this spec's job ends once it fires the reminder trigger at the correct time.
- Letting the Pro configure the lead time or window per account -- product-features.md defines no such setting for this feature; the lead time and window are single values for every Pro (XBR-16; platform parameter: `reminder-lead-time-days`, platform parameter: `reminder-window-start-hour`, platform parameter: `reminder-window-end-hour`), not Pro-configurable preferences, consistent with scope-boundaries.md's absence of any reminder-timing customization capability.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A Booking is confirmed (deposit captured) | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Fires once per Booking, immediately on confirmation, to compute (or suppress) that booking's reminder time | Booking (start_time), Pro Account (timezone) |
| A confirmed Booking's reminder time is reached | Schedule-based (system clock, evaluated against each Booking's computed reminder_send_time) | Fires when the current time in the Pro's timezone reaches the computed and window-adjusted send time | Booking (service, start_time, deposit_amount, balance_due) |
| A Booking with a scheduled, not-yet-fired reminder is cancelled or rescheduled | FEAT-10 (Client-Initiated Cancel/Reschedule) / FEAT-30 (Pro Booking Management) | Fires whenever a Booking's state changes away from Confirmed, or its start_time changes, before its reminder has fired | Booking (new state or new start_time) |

## Processing Logic

1. On Booking confirmation, read the Booking's start_time and the Pro Account's timezone.
2. Compute the candidate reminder time as start_time minus platform parameter: `reminder-lead-time-days` (BRIEF.md's stated example: two days before the appointment).
3. Compare the candidate reminder time to the current time. If the candidate reminder time has already passed (the booking was made too close to its own appointment for a two-day-ahead reminder to make sense), suppress the reminder entirely for this Booking -- no reminder is ever scheduled, and the confirmation already sent (FEAT-08.SPEC-001) serves as the client's only pre-appointment message.
4. If the candidate reminder time has not yet passed, check whether it falls inside the daytime send window (platform parameter: `reminder-window-start-hour` to platform parameter: `reminder-window-end-hour`, in the Pro's timezone -- BRIEF.md's stated example: roughly 8am to 9pm).
5. If the candidate time falls outside the window, move it forward to the window's start time on the same day if the candidate was before the window opened, or to the window's start time on the next day if the candidate was after the window closed.
6. Store the resulting reminder_send_time on the Booking.
7. At the stored reminder_send_time, fire the trigger that initiates FEAT-08.SPEC-002 (Appointment Reminder Message), carrying the Booking reference.
8. If the Booking is cancelled, rescheduled, or otherwise leaves the Confirmed state before its stored reminder_send_time is reached, cancel the scheduled reminder -- recompute a fresh reminder_send_time from the new start_time if the booking was rescheduled and remains Confirmed; clear the scheduled reminder entirely if the booking was cancelled.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Reminder scheduled at the default time | Candidate reminder time is within the daytime window | Booking.reminder_send_time set | None directly -- the client sees only the reminder itself when it later fires | FEAT-08.SPEC-002 |
| Reminder scheduled with a window-adjusted time | Candidate reminder time falls outside 8am--9pm | Booking.reminder_send_time set to the nearest allowed time | None directly | FEAT-08.SPEC-002 |
| Reminder suppressed (late booking) | The candidate reminder time has already passed at confirmation time | Booking.reminder_send_time left unset; a suppressed flag recorded for traceability | None -- no reminder is ever shown as pending; the confirmation already sent is the client's only pre-appointment message | FEAT-08.SPEC-001, FEAT-08.SPEC-002 |
| Reminder fires | The system clock reaches Booking.reminder_send_time and the Booking is still Confirmed | None on the Booking itself -- triggers FEAT-08.SPEC-002 | Client receives the reminder message | FEAT-08.SPEC-002 |
| Reminder cancelled (booking cancelled) | Booking leaves Confirmed state before reminder_send_time | Booking.reminder_send_time cleared | None -- no reminder fires for a cancelled booking | FEAT-08.SPEC-002 |
| Reminder rescheduled (booking rescheduled) | Booking's start_time changes while it remains Confirmed | Booking.reminder_send_time recomputed from the new start_time, re-running Steps 2--6 | None directly -- a fresh reminder is scheduled at the newly computed time | FEAT-08.SPEC-002 |
| Automation failure | The scheduling computation itself cannot run (e.g., timezone data unavailable at confirmation time) | No reminder_send_time is set | No client-facing feedback; this automation's own failure directly fires FEAT-08.SPEC-006 (Pro Attention Alert)'s reminder-scheduling-failure trigger, so the Pro sees the gap rather than discovering a missing reminder only when the appointment arrives | FEAT-08.SPEC-006 |

## Data Model

**Reads:** Booking -- start_time, state; Pro Account -- timezone.
**Creates:** None.
**Updates:** Booking -- reminder_send_time (computed field owned by this automation).
**Deletes:** None.

## Business Rules

- XBR-16 governs this spec entirely: reminders go out only between roughly 8am and 9pm in the Pro's timezone (platform parameter: `reminder-window-start-hour` / platform parameter: `reminder-window-end-hour`); confirmations are unaffected by this window (they are handled by FEAT-08.SPEC-001, which is not a discretionary reminder); a booking made after its own reminder point gets no separate reminder.
- The default lead time is platform parameter: `reminder-lead-time-days`, matching BRIEF.md's stated example of two days before the appointment.
- The window-adjustment rule always moves a candidate time forward in time (to later the same day or to the next day), never backward, so a reminder is never sent earlier than intended to fit the window.
- Suppression (Step 3) is evaluated once, at confirmation time, against the current time -- not re-evaluated later, so a booking that was made in time for a reminder is never retroactively suppressed just because the window computation later needs adjustment.

## Edge Cases

- **A booking is made exactly at the boundary of its own reminder point** -- If the two-day-ahead time has not yet passed at the moment of confirmation (even by a small margin), the reminder is scheduled normally; only a candidate time already in the past at confirmation is suppressed.
- **The Pro's timezone changes between booking confirmation and the reminder firing** -- Per XBR-25, the Pro's timezone is a per-account setting; if it changes, the window check re-evaluates against the currently configured timezone for any not-yet-fired reminder, since Booking.reminder_send_time was computed once but the daytime-window boundary is a live, timezone-relative concept the system re-derives at fire time for still-pending reminders.
- **A rescheduled booking's new time also falls after its own new reminder point would have passed** -- The same suppression rule (Step 3) applies to the recomputed time: if the new appointment is too soon for a two-day-ahead reminder, no reminder is scheduled for the rescheduled booking either, and the change notice (FEAT-08.SPEC-004) is the client's only additional message.
- **Concurrent trigger firing (two bookings confirmed at effectively the same time)** -- Each Booking's reminder computation runs independently against its own start_time and the shared Pro Account timezone; neither computation affects the other, since reminder_send_time is a per-booking field.
- **Trigger fires while a previous run is in flight for the same Booking** -- A second confirmation event cannot occur for the same Booking (FEAT-07.SPEC-002 guarantees exactly one capture per booking), so no two scheduling computations ever run concurrently for one Booking; a cancellation/reschedule event for a Booking whose reminder computation is still in flight is queued to apply immediately after the in-flight computation completes, so the final stored reminder_send_time always reflects the Booking's latest state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Triggered by (inbound) | Booking confirmation starts the reminder-time computation |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Triggered by (inbound) | A client cancellation or reschedule re-triggers cancellation or recomputation |
| FEAT-30 (Pro Booking Management) | Triggered by (inbound) | A Pro-initiated cancellation or reschedule re-triggers cancellation or recomputation |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Affects (outbound) | This automation's fired trigger is what starts that notification |
| FEAT-08.SPEC-001 (Booking Confirmation Message) | References (outbound) | Serves as the client's sole pre-appointment message when this automation suppresses a reminder |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggers (outbound) | This automation's own failure to compute or store a reminder_send_time fires FEAT-08.SPEC-006's reminder-scheduling-failure alert |

## Analytics and Success Signals

- **reminder_scheduled** (lead_time_days, window_adjusted: yes / no) -- supports success-metrics.md: "Reminder Response Rate"
- **reminder_suppressed_late_booking** () -- N/A -- no Stage 2 metric measures suppression frequency directly; retained so the late-booking exception's actual frequency is observable rather than assumed.
- **reminder_schedule_cancelled** (reason: booking_cancelled / booking_rescheduled) -- N/A -- no Stage 2 metric tracks cancelled reminder schedules; this event supports operational visibility into the scheduling pipeline's correctness rather than a named success metric.

## Acceptance Criteria

**FEAT-08.SPEC-007-AC-01:** Given Riley books an appointment 10 days out, when the booking confirms, then the reminder is scheduled for platform parameter: `reminder-lead-time-days` before the appointment, provided that time falls within the daytime window.

**FEAT-08.SPEC-007-AC-02:** Given Riley's computed reminder time falls before platform parameter: `reminder-window-start-hour` in the Pro's timezone, when the scheduling computation runs, then the reminder is moved forward to platform parameter: `reminder-window-start-hour` the same day.

**FEAT-08.SPEC-007-AC-03:** Given Riley's computed reminder time falls after platform parameter: `reminder-window-end-hour` in the Pro's timezone, when the scheduling computation runs, then the reminder is moved forward to platform parameter: `reminder-window-start-hour` the next day.

**FEAT-08.SPEC-007-AC-04:** Given Riley books an appointment sooner than platform parameter: `reminder-lead-time-days` away, when the booking confirms, then no reminder is scheduled, and the confirmation already sent is her only pre-appointment message.

**FEAT-08.SPEC-007-AC-05:** Given Riley's booking has a scheduled reminder that has not yet fired, when Riley cancels the booking, then the scheduled reminder is cleared and never fires.

**FEAT-08.SPEC-007-AC-06:** Given Riley's booking has a scheduled reminder that has not yet fired, when the Pro reschedules the booking to a new time, then the reminder time is recomputed from the new start_time.

**FEAT-08.SPEC-007-AC-07:** Given a rescheduled booking's new appointment is now less than two days out, when the recomputation runs, then the reminder is suppressed for the rescheduled booking as well.

**FEAT-08.SPEC-007-AC-08:** Given a Booking's stored reminder_send_time is reached and the Booking is still Confirmed, when the system clock crosses that time, then FEAT-08.SPEC-002 is triggered for that Booking.

**FEAT-08.SPEC-007-AC-09:** Given two bookings confirm at effectively the same instant, when both scheduling computations run, then each Booking's reminder_send_time is computed independently and correctly.

**FEAT-08.SPEC-007-AC-10:** Given a Booking's reminder computation is in flight when a cancellation event for the same Booking arrives, when the in-flight computation completes, then the cancellation is applied immediately afterward and no reminder fires for the cancelled Booking.

**FEAT-08.SPEC-007-AC-11:** Given the Pro changes her account timezone while a Booking's reminder is still pending, when the window is next evaluated for that pending reminder, then it uses the currently configured timezone.

**FEAT-08.SPEC-007-AC-12:** Given the scheduling computation itself fails to run for a confirmed Booking, when no reminder_send_time is set, then no client-facing error appears, and this automation's failure fires FEAT-08.SPEC-006's reminder-scheduling-failure alert to the Pro.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (confirmation, scheduled fire, cancel/reschedule) | 3 |
| Outcome Paths | 7 | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
