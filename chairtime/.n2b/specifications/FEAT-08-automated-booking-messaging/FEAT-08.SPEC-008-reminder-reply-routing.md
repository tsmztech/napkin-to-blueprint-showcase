---
document_type: spec
spec_type: automation
spec_id: FEAT-08.SPEC-008
spec_name: Reminder Reply Routing
spec_slug: reminder-reply-routing
parent_feature: FEAT-08
parent_feature_name: Automated Booking Messaging
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Reminder Reply Routing

## Overview

**Name:** Reminder Reply Routing
**ID:** FEAT-08.SPEC-008
**Type:** Automation
**Purpose:** Processes whichever one-tap reply a client makes on a reminder -- recording "I'll be there" as an acknowledgment, or handing "I need to reschedule" off into the reschedule flow.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- Validating the tapped booking-specific link before acting on either reply
- Recording the "I'll be there" acknowledgment on the Booking
- Routing "I need to reschedule" into FEAT-10, scoped to the same booking

**Non-Goals:**
- The reminder message content and its two link URLs -- owned by FEAT-08.SPEC-002; this spec only processes a tap on those links.
- The landing screen shown after acknowledgment -- owned by FEAT-08.SPEC-003.
- The reschedule flow itself once handed off -- owned entirely by FEAT-10 from that point on, per the Brief's Internal Dependency Map ("[routes into] FEAT-10 ... from that point on").
- Minting or validating the manage link's scope and expiry rules in general -- owned by FEAT-08.SPEC-010 and FEAT-06; this spec consumes those rules but does not define them.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client taps "I'll be there" | FEAT-08.SPEC-002 (Appointment Reminder Message) | Fires whenever the "I'll be there" link is opened, regardless of channel (text or email) | Booking reference (from the link), current Booking state |
| Client taps "I need to reschedule" | FEAT-08.SPEC-002 (Appointment Reminder Message) | Fires whenever the "I need to reschedule" link is opened, regardless of channel | Booking reference (from the link), current Booking state |

## Processing Logic

1. Receive the tapped link's Booking reference and reply type ("I'll be there" or "I need to reschedule").
2. Validate the link: confirm the referenced Booking exists, its appointment has not yet passed, and the link has not been superseded by a fresher link (e.g., issued after a Pro-initiated reschedule, per FEAT-08.SPEC-010).
3. If the link is invalid or expired, take no action on the Booking and route the client to FEAT-08.SPEC-003's Expired/Invalid Link state.
4. If the link is valid and the reply is "I'll be there", set Booking.attendance_reply to "I'll be there" and record the reply timestamp.
5. If the link is valid and the reply is "I need to reschedule", do not modify Booking.attendance_reply; instead hand off directly into FEAT-10 (Client-Initiated Cancel/Reschedule), carrying the same Booking reference, so the client lands in the reschedule flow for that one booking.
6. For an "I'll be there" reply, forward the client to FEAT-08.SPEC-003 (Reminder Reply Acknowledgment) once Step 4 completes.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Acknowledgment recorded | Valid link, "I'll be there" tapped | Booking.attendance_reply set to "I'll be there" | Client lands on FEAT-08.SPEC-003's Acknowledged state | FEAT-08.SPEC-003, FEAT-12 |
| Routed to reschedule | Valid link, "I need to reschedule" tapped | None on the Booking directly (FEAT-10 owns any subsequent change) | Client lands in FEAT-10's reschedule flow for this booking | FEAT-10 |
| Link invalid or expired | The Booking cannot be found, the appointment has passed, or a fresher link supersedes this one | None | Client lands on FEAT-08.SPEC-003's Expired/Invalid Link state | FEAT-08.SPEC-003 |
| Second reply on an already-acknowledged booking | Client taps "I need to reschedule" after already having tapped "I'll be there" (or the reverse) | The later tap's outcome is evaluated against the Booking's then-current state; if the Booking has already moved out of Confirmed (e.g., already in the reschedule flow), the second tap is routed into FEAT-10 rather than silently ignored | Client is routed into FEAT-10's reschedule flow, or shown FEAT-08.SPEC-003's Acknowledged/Expired state, depending on the Booking's then-current state | FEAT-08.SPEC-003, FEAT-10 |
| Automation failure | The routing step itself cannot complete (e.g., link resolution service unavailable) | None | Client sees FEAT-08.SPEC-003's generic Error state: "Something went wrong loading your confirmation. Try the link again from your reminder message." | FEAT-08.SPEC-003 |

## Data Model

**Reads:** Booking -- state, start_time, attendance_reply; Access Link -- scope, expiry, state (via FEAT-06/FEAT-08.SPEC-010's link resolution).
**Creates:** None.
**Updates:** Booking -- attendance_reply (set only on a valid "I'll be there" reply).
**Deletes:** None.

## Business Rules

- "I'll be there" simply acknowledges -- it never changes the Booking's state, price, or schedule; it is purely informational for the Pro's dashboard (FEAT-12).
- "I need to reschedule" performs no reschedule itself; it only opens the door into FEAT-10, where the client picks a new time subject to that feature's own validation and policy rules (XBR-09).
- Both reply links work identically whether the reminder was received by text or email, per product-features.md's Validation & Limits field ("the one-tap replies are link taps, so a reply works the same by text or email").
- A booking-specific link's scope, expiry, and superseded-by-a-fresher-link rules are owned by FEAT-08.SPEC-010/FEAT-06 (XBR-18); this automation enforces those rules at the moment of the tap but does not define them.

## Edge Cases

- **Client taps "I'll be there" twice from the same link** -- The second tap is a no-op that re-confirms the same attendance_reply value; Booking.attendance_reply is not re-timestamped, and the client sees the same Acknowledged screen.
- **Client taps "I need to reschedule" after already tapping "I'll be there" on the same booking** -- The tap is still honored: the client is routed into FEAT-10, since wanting to reschedule after all is a legitimate, later change of mind that the flat "I'll be there" flag does not block.
- **The appointment passes between the reminder being sent and either reply being tapped** -- Per XBR-18, the booking-specific link has expired; both reply types route to FEAT-08.SPEC-003's Expired/Invalid Link state rather than acting on a stale reply.
- **Concurrent trigger firing (the client taps both links in different browser tabs at nearly the same time)** -- Whichever tap's validation completes first determines the outcome; the Booking's Contention resolution (reject-with-refresh, feature-dependency-map.md) means the second tap is evaluated against the Booking's now-updated state, so it either succeeds consistently (if compatible) or is redirected to reflect the Booking's current state rather than silently overwriting the first outcome.
- **Trigger fires while a previous run is in flight for the same Booking** -- A second tap on the same link while the first tap's processing is still resolving is queued to evaluate against the Booking's post-processing state, preventing two conflicting writes to attendance_reply from racing each other.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Triggered by (inbound) | Both reply links deep-link into this automation |
| FEAT-08.SPEC-003 (Reminder Reply Acknowledgment) | Affects (outbound) | Receives the client after an acknowledgment or an invalid-link outcome |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Affects (outbound) | Receives the client after a reschedule-reply outcome |
| FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) | References (inbound) | Owns the link scope/expiry rules this automation enforces at tap time |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | Displays the "I'll be there" status once recorded |

## Analytics and Success Signals

- **reminder_reply_confirmed** () -- supports success-metrics.md: "Reminder Response Rate"
- **reminder_reply_reschedule_requested** () -- supports success-metrics.md: "Reminder Response Rate"
- **reminder_reply_invalid_link** (reason: expired / superseded / not_found) -- N/A -- no Stage 2 metric measures invalid-link tap frequency; retained to keep the link-expiry design's real-world frequency observable rather than assumed.

## Acceptance Criteria

**FEAT-08.SPEC-008-AC-01:** Given Riley taps "I'll be there" on a valid, unexpired link, when the automation processes the tap, then Booking.attendance_reply is set to "I'll be there" and she is forwarded to FEAT-08.SPEC-003.

**FEAT-08.SPEC-008-AC-02:** Given Riley taps "I need to reschedule" on a valid, unexpired link, when the automation processes the tap, then she is routed into FEAT-10's reschedule flow for that same booking, and Booking.attendance_reply is unchanged.

**FEAT-08.SPEC-008-AC-03:** Given Riley's appointment has already passed, when she taps either reply link, then she is routed to FEAT-08.SPEC-003's Expired/Invalid Link state and no Booking field changes.

**FEAT-08.SPEC-008-AC-04:** Given Riley taps "I'll be there" twice from the same link, when the second tap is processed, then it is a no-op and she sees the same Acknowledged screen without a new timestamp being recorded.

**FEAT-08.SPEC-008-AC-05:** Given Riley already tapped "I'll be there" and later taps "I need to reschedule" on the same reminder, when the second tap is processed, then she is routed into FEAT-10's reschedule flow, honoring her later choice.

**FEAT-08.SPEC-008-AC-06:** Given the Pro reschedules Riley's booking after the reminder was sent, when Riley later taps either link from the superseded reminder, then she is routed to FEAT-08.SPEC-003's Expired/Invalid Link state, since a fresher link now governs the booking.

**FEAT-08.SPEC-008-AC-07:** Given Riley received her reminder by email rather than text, when she taps either reply link, then the outcome is identical to a text-received reminder.

**FEAT-08.SPEC-008-AC-08:** Given Riley taps both reply links in two browser tabs at nearly the same time, when both taps are processed, then the second tap is evaluated against the Booking's state as updated by the first, rather than racing it.

**FEAT-08.SPEC-008-AC-09:** Given the link-resolution capability is temporarily unavailable when Riley taps a reply link, when the automation cannot complete, then she sees FEAT-08.SPEC-003's generic Error state.

**FEAT-08.SPEC-008-AC-10:** Given Riley's "I'll be there" acknowledgment is recorded, when the Pro next views her dashboard (FEAT-12), then the booking shows the "I'll be there" status.

**FEAT-08.SPEC-008-AC-11:** Given a second tap arrives on the same link while the first tap's processing is still in flight, when the first tap's processing completes, then the second tap evaluates against the Booking's post-processing state rather than writing a conflicting attendance_reply concurrently.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (I'll be there, I need to reschedule) | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
