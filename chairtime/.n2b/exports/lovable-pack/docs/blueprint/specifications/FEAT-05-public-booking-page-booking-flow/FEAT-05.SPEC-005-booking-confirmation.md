---
document_type: spec
spec_type: screen
spec_id: FEAT-05.SPEC-005
spec_name: Booking Confirmation
spec_slug: booking-confirmation
parent_feature: FEAT-05
parent_feature_name: Public Booking Page & Booking Flow
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Booking Confirmation

## Overview

**Name:** Booking Confirmation
**ID:** FEAT-05.SPEC-005
**Type:** Screen
**Purpose:** Client sees an immediate on-screen confirmation of the completed, paid booking.
**Parent Feature:** FEAT-05 -- Public Booking Page & Booking Flow

## Scope and Non-Goals

**In Scope:**
- Displaying the confirmed booking's details immediately after successful deposit payment
- Reading the completed Booking record to render this confirmation, including when reached on a fresh load after a payment success whose navigation initially failed
- Offering the client a "Make this a standing appointment" action that leads to FEAT-21.SPEC-001 (Set Up Recurring Series)

**Non-Goals:**
- Sending the confirmation message (text or email) -- entirely owned by FEAT-08 (Automated Booking Messaging), triggered by the payment success, not by this screen
- Setting up the recurring series itself -- owned by FEAT-21.SPEC-001; this screen only offers the entry point
- Recording the Activity Event for the new Booking -- owned by FEAT-16.SPEC-002 (Activity Event Recording), triggered by the same confirmed Booking this screen displays
- Any post-confirmation self-service (viewing, cancelling, or rescheduling later) -- owned by FEAT-06 (Client Booking Identity) and FEAT-10 (Client-Initiated Cancel/Reschedule), reached through the confirmation message's manage link, not from this screen directly
- Updating the Pro's schedule or connected calendar -- owned by FEAT-12 (Pro Daily Schedule Dashboard) and FEAT-04 (Two-Way Calendar Sync) respectively, both triggered by the same payment success this screen displays the result of

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-07.SPEC-001 (Deposit Payment) | Deposit payment succeeds and the Booking reaches Confirmed (the payment screen hands off here automatically) | The now-Confirmed Booking's full details |
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | A fresh load of the flow after payment succeeded but the confirmation navigation initially failed | The Confirmed Booking, read directly rather than carried in navigation state |
| FEAT-21.SPEC-001 (Set Up Recurring Series) | Client backs out of the recurring set-up without creating a series | The confirmed Booking reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen | No further action required; may optionally tap "Make this a standing appointment" | -- |
| The Pro (Talia), preview mode | Full screen, identical rendering, showing a simulated confirmed booking | No further action; the recurring action is shown but no series is created (no real Booking exists) | -- |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view showing a past confirmation rendering for troubleshooting | View only | Not applicable -- this screen has no actions to restrict |
| Unauthenticated | Yes -- the default and intended state for the Client role | Yes, identical to the Client row above | -- |
| Expired session | N/A -- no session exists to expire; the screen renders from the completed Booking record itself, not from in-flow session state | N/A | N/A |

## Layout and Content

**Header:** A success indicator (e.g., a checkmark treatment) with the heading "You're booked!"

**Body:** The confirmed booking's details:
- Service name
- Date and time
- Deposit amount paid
- Balance due at the appointment (price minus deposit)
- The Pro's studio address (shown here for the first time in the flow, per its confirmation-only disclosure rule)
- A short line noting that a confirmation has been sent to the client's chosen channel (text or email, matching what was selected on FEAT-05.SPEC-003)

**Footer:** A secondary "Make this a standing appointment" action offering to turn the just-confirmed booking into a recurring series. It is optional and never blocks reading the confirmation; the client's ongoing interaction with their booking otherwise happens through the confirmation message's manage link, outside this screen.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content remains single-column, capped at a comfortable reading width and horizontally centered; no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Success heading, booking details, studio address | -- | Display-only, non-interactive | None | -- |
| "Make this a standing appointment" | Tap | Navigate to FEAT-21.SPEC-001 (Set Up Recurring Series) carrying the just-confirmed Booking's reference, service, and start time | Screen closes | Standard forward transition |

### Accessibility Notes

- **Focus order:** Success heading -> service and time -> deposit and balance -> studio address -> confirmation-sent notice -> "Make this a standing appointment" action.
- **Announcements:** The success heading and the fact that a confirmation was sent are announced to assistive technology as soon as the screen renders.
- **Keyboard alternatives:** The "Make this a standing appointment" action is reachable and activatable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty | N/A -- this screen only ever renders a single completed Booking, never a collection that can be empty | N/A | N/A |
| Confirmed | Full booking details rendered as described in Layout and Content | Deposit payment succeeded and the Booking is Confirmed | Client leaves the page (terminal state within this flow) |
| Loading | A neutral loading placeholder while the completed Booking is read | Screen is reached via a fresh load after payment success (navigation retry case) | Booking data loads successfully |
| Error | A plain message: "We're confirming your booking -- check your text or email for confirmation, or contact the pro directly if you don't see it shortly." | The completed Booking cannot be read on a fresh-load attempt, even though payment succeeded | Client leaves the page; the confirmation message (FEAT-08) still arrives independently of this screen's own load outcome |
| Offline/Degraded | A plain banner: "Check your connection to see your booking details." in place of the booking details; the success heading remains visible if already rendered | Connectivity is lost after payment succeeded but before this screen's data finishes loading | Connectivity is restored and the booking details load |

## Validation Rules

Not applicable -- this screen has no user input.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| "Make this a standing appointment" tap | FEAT-21.SPEC-001 (Set Up Recurring Series) | FEAT-21 (Recurring/Standing Appointments) |

Managing the booking afterward happens outside this flow, through the confirmation message's manage link (FEAT-06, FEAT-08).

## Data Model

**Creates:** None.
**Reads:** Booking -- service, start_time, duration, price_agreed, deposit_amount, balance_due, state (Confirmed). Pro Account -- studio_address (shown here for the first time, per its confirmation-only disclosure rule). Client -- the chosen consent channel (text or email), to state which channel the confirmation was sent to.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The recurring offer is optional and appears only for a real Confirmed Booking; declining it (simply leaving the screen) changes nothing about the booking, and the series set-up and its rules belong entirely to FEAT-21.SPEC-001.
- This screen renders only once the Booking has actually transitioned to Confirmed by FEAT-07's successful deposit capture; it never shows a confirmed state on an unconfirmed booking.
- The immediate booking confirmation message is triggered by the deposit payment succeeding, not by this screen rendering -- the message (owned entirely by FEAT-08) and this on-screen display are two independent, parallel effects of the same payment-success event, so a failure of one never blocks or delays the other.
- The studio address is shown here for the first time in the client's flow, consistent with its confirmation-only disclosure rule (it is never shown earlier, since it may be a home address).
- The full flow from FEAT-05.SPEC-001 through this screen completes in under one minute for the under-one-minute benchmark (Non-Functional Notes; success-metrics.md: "Booking Completion Speed").
- In preview mode, this screen shows a simulated confirmed booking; no real Booking, Deposit Transaction, or confirmation message is created.

## Edge Cases

- **Payment succeeded but the direct navigation to this screen failed** -- The client reaches this screen via a fresh load of the flow, which reads the already-Confirmed Booking directly rather than relying on carried navigation state, so the client sees their confirmation correctly rather than being asked to pay again.
- **The completed Booking cannot be read even on a fresh load (a rare data-access failure)** -- The Error state renders, reassuring the client that their confirmation message (already triggered independently by the successful payment) will still arrive.
- **Client loses connectivity immediately after payment succeeds, before this screen finishes loading** -- The Offline/Degraded state renders; the booking is still correctly Confirmed on the server regardless of this screen's own load outcome, and the confirmation message still arrives.
- **Client refreshes or reopens this screen later** -- The screen re-reads the Booking and renders the same confirmed details again; there is no time limit on viewing this screen directly (though it is not the ongoing way to check a booking -- that is the manage link).

## Connected Specs

| Connected Spec | Connection Type | Description |
|-----------------|-------------------|--------------|
| FEAT-07.SPEC-001 (Deposit Payment) | Navigation (inbound) | Client arrives here automatically after successful deposit payment |
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Navigation (inbound) | A fresh load of the flow after a successful payment resolves to this screen |
| FEAT-21.SPEC-001 (Set Up Recurring Series) -- within FEAT-21 (Recurring/Standing Appointments) | Navigation (outbound) | The "Make this a standing appointment" action leads here |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Triggers (outbound) | The newly Confirmed Booking this screen displays triggers activity event recording, in parallel with this display |
| FEAT-08 (Automated Booking Messaging) | References (outbound) | Triggers the immediate booking confirmation message, in parallel with this screen's own display |
| FEAT-12 (Pro Daily Schedule Dashboard) | References (outbound) | The confirmed booking becomes visible on the Pro's schedule as a parallel effect of the same payment success |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| booking_confirmation_viewed | total elapsed time from FEAT-05.SPEC-001, confirmation channel (text/email) | This screen renders in the Confirmed state | supports success-metrics.md: "Booking Completion Speed" |
| booking_confirmation_load_failed | reason (error / offline) | The Error or Offline/Degraded state renders | supports success-metrics.md: "Deposit Capture Rate" (a lost confirmation display, even after a successful charge, is a trust risk the metric's clean-outcome bar is meant to catch) |

## Acceptance Criteria

**FEAT-05.SPEC-005-AC-01:** Given Riley's deposit payment succeeds on FEAT-07.SPEC-001, when she advances to this screen, then she sees "You're booked!" along with the service, date and time, deposit paid, balance due, and the studio address.

**FEAT-05.SPEC-005-AC-02:** Given Riley reaches this screen, when it renders, then Riley sees a line confirming that a confirmation has been sent to the channel she chose (text or email).

**FEAT-05.SPEC-005-AC-03:** Given Riley's payment succeeded but the direct navigation to this screen failed, when Riley reopens the flow, then she sees the same confirmed booking details, correctly reflecting her Confirmed booking rather than being asked to pay again.

**FEAT-05.SPEC-005-AC-04:** Given the completed Booking cannot be read on a fresh-load attempt, when this screen is reached that way, then Riley sees a reassuring message that her confirmation is on its way by text or email.

**FEAT-05.SPEC-005-AC-05:** Given Riley loses connectivity immediately after payment succeeds, when this screen attempts to load, then she sees "Check your connection to see your booking details." while her booking remains correctly Confirmed on the server.

**FEAT-05.SPEC-005-AC-06:** Given Talia previews her own booking page through to this screen, when the simulated payment completes, then she sees the identical confirmation screen with no real Booking or confirmation message created.

**FEAT-05.SPEC-005-AC-07:** Given Riley reaches this screen, when the studio address is displayed, then it is the first point in the flow where that address has been shown to her.

**FEAT-05.SPEC-005-AC-08:** Given Riley's booking is confirmed, when the confirmation message and this screen's own display both fire from the same payment-success event, then a delay or failure of the confirmation message never blocks this screen from rendering, and vice versa.

**FEAT-05.SPEC-005-AC-09:** Given Riley completes her booking end to end from FEAT-05.SPEC-001 to this screen, when she reaches this screen, then the total elapsed time is recorded to validate the under-one-minute benchmark.

**FEAT-05.SPEC-005-AC-10:** Given Riley refreshes this screen later the same day, when it reloads, then it re-reads and re-renders the same confirmed booking details.

**FEAT-05.SPEC-005-AC-11:** Given Riley is viewing her confirmed booking, when she taps "Make this a standing appointment", then she navigates to FEAT-21.SPEC-001 (Set Up Recurring Series) with the confirmed Booking's reference, service, and start time carried.

**FEAT-05.SPEC-005-AC-12:** Given Riley's booking is confirmed, when the Booking reaches Confirmed, then FEAT-16.SPEC-002 is triggered to record the activity event in parallel with this screen's display, and a delay or failure of that recording never blocks this screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 2 (display-only content, recurring offer) | 2 |
| States | 5 (empty, confirmed, loading, error, offline) | 5 |
| Business Rules | 6 | 6 |
| Edge Cases | 4 | 4 |
