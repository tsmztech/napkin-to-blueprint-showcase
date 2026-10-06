---
document_type: spec
spec_type: screen
spec_id: FEAT-30.SPEC-001
spec_name: Cancel Booking (Pro-Initiated)
spec_slug: cancel-booking-pro-initiated
parent_feature: FEAT-30
parent_feature_name: Pro Booking Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 16
---

# Screen Spec: Cancel Booking (Pro-Initiated)

## Overview

**Name:** Cancel Booking (Pro-Initiated)
**ID:** FEAT-30.SPEC-001
**Type:** Screen
**Purpose:** Talia views a single booking and confirms cancelling it, seeing plainly that the client's deposit will be refunded in full whatever the timing.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Showing the booking's details and the always-full-refund outcome before Talia confirms
- Capturing an optional private cancellation reason
- Triggering the commit and reflecting its success, rejection, or failure

**Non-Goals:**
- Determining or executing the deposit refund itself -- owned by FEAT-09, triggered through FEAT-30.SPEC-007; this screen only shows the outcome that XBR-09 guarantees
- Rescheduling the booking instead of cancelling it -- owned by FEAT-30.SPEC-002 (Reschedule Booking, Pro-Initiated), a distinct screen and action
- Cancelling more than one booking at a time -- owned by FEAT-30.SPEC-005 (Cancel Several Bookings at Once), a distinct review-and-confirm flow for a conflict set
- Setting up, viewing, or cancelling a client's recurring series or an occurrence of one -- recurring management is a Non-Goal of this feature and is owned by FEAT-21.SPEC-010 (Pro Recurring Series Management); this screen only offers an outbound link to it

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard) | Talia taps a booking row and chooses "Cancel" | Booking reference |
| FEAT-13 (Client Record Management) | Talia confirms deleting a client with an upcoming booking | Booking reference, with a note that this cancellation is required before the deletion can proceed |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Talia requests account closure with upcoming bookings, and this booking is one of a small set handled individually rather than in bulk | Booking reference |
| FEAT-30.SPEC-005 (Cancel Several Bookings at Once) | Talia taps "View" on a booking that failed in a bulk cancellation | The failed Booking reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen, for bookings she owns only | Confirm the cancellation, or back out with nothing changed | -- |
| The Client (Riley) | No | No | No control on any Client-facing surface reaches this screen; the Client's Booking & Payment access is Own-only, exercised through FEAT-10 |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view | View only | Cancel confirm control and the Recurring series link are not shown, consistent with SC-05 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); no booking detail is shown |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no unsaved input exists on this screen to preserve, since no cancellation has been confirmed yet |

## Layout and Content

**Header:** Back arrow (returns to FEAT-12) with the title "Cancel booking."

**Body:** A summary of the booking being cancelled -- client name, service, date and time, deposit amount -- followed by a plainly worded outcome statement: "{client_name}'s {deposit_amount} deposit will be refunded in full." An optional single-line text field labeled "Reason (private, not shared with {client_name})" for Talia's own note. Below that, a "Cancel booking" confirm button and a "Never mind" link to back out. Beneath the booking summary sits a "Recurring series" link, labeled "Repeat this booking" when the booking is not part of a series and "Manage recurring series" when it is, which opens FEAT-21.SPEC-010 (Pro Recurring Series Management) for this booking.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12 (Pro Daily Schedule Dashboard) | Screen closes | Standard backward transition |
| Reason field | Type | Captures optional private text | Field shows entered text | Standard input focus state |
| "Cancel booking" button | Tap | Triggers FEAT-30.SPEC-006 eligibility re-check, then FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Button shows loading state during commit | Success: confirmation shown, then navigate to FEAT-12. Rejection: exact denial message from FEAT-30.SPEC-006 shown inline, no navigation. Failure: retry prompt shown, booking unchanged |
| "Cancel booking" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "Recurring series" link ("Repeat this booking" / "Manage recurring series") | Tap | Navigate to FEAT-21.SPEC-010 (Pro Recurring Series Management) carrying the Booking reference; nothing on this screen is changed or committed | Screen closes without cancelling | Standard forward transition |
| "Never mind" link | Tap | Navigate to FEAT-12 with nothing changed | Screen closes | Standard backward transition |

### Accessibility Notes

- **Focus order:** Back arrow -> booking summary -> Recurring series link -> outcome statement -> Reason field -> Cancel booking button -> Never mind link.
- **Announcements:** The outcome statement and any denial message are announced to assistive technology when the screen loads or when a denial occurs.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | A neutral loading placeholder in place of the booking summary and outcome statement | Screen opens | Booking data loads successfully or a load error occurs |
| Ready | Booking summary, outcome statement, and confirm/back-out controls shown | Booking data loads successfully and the eligibility check passes | Talia confirms or backs out |
| Ineligible | The exact denial message from FEAT-30.SPEC-006 shown in place of the confirm control (e.g., "This booking is already completed and can no longer be changed.") | Booking data loads successfully but the eligibility check fails | Talia backs out (only path forward, since no action is available) |
| Committing | Confirm button shows a loading state, all inputs disabled | Talia taps "Cancel booking" | Commit completes (success, rejection, or failure) |
| Committed | Confirmation shown ("Booking cancelled. {client_name}'s deposit is being refunded.") before returning to FEAT-12 | FEAT-30.SPEC-007 reports success | Talia is navigated to FEAT-12 |
| Rejected | Inline message showing the booking's current state (e.g., "This booking was already cancelled by {client_name}.") | FEAT-30.SPEC-007 reports a conflicting transition already won | Talia backs out; no retry of the same action is offered since the booking has moved on |
| Error | Inline error message "Couldn't load this booking. Try again." or "Couldn't cancel this booking. Try again." with a Retry action | The booking's details fail to load, or FEAT-30.SPEC-007 reports a processing failure | Talia taps Retry and it succeeds, or she backs out |
| Offline/Degraded | Banner "Check your connection and try again." replaces the confirm control; the booking summary remains viewable read-only | Connectivity is lost while the screen is open | Connectivity is restored and Talia can confirm |

## Validation Rules

Validation and eligibility governed by FEAT-30.SPEC-006 (Pro Booking Action Rules). See that spec for the exact conditions and denial messages. This screen applies the eligibility check on entry and again on confirm.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | FEAT-12 (Pro Daily Schedule Dashboard) | -- |
| "Never mind" tap | FEAT-12 (Pro Daily Schedule Dashboard) | -- |
| "Repeat this booking" / "Manage recurring series" tap | FEAT-21.SPEC-010 (Pro Recurring Series Management) | FEAT-21 (Recurring/Standing Appointments) |
| Successful cancellation | FEAT-12 (Pro Daily Schedule Dashboard) | -- |

## Data Model

**Reads:** Booking -- state, start_time, service, client, deposit_amount (via Deposit Transaction); Deposit Transaction -- status (eligibility precondition).
**Creates:** None directly -- the cancellation itself is written by FEAT-30.SPEC-007.
**Updates:** None directly -- delegated entirely to FEAT-30.SPEC-007 on confirm.
**Deletes:** None.

## Business Rules

- XBR-09: any Pro-initiated cancellation refunds the client's deposit in full, whatever the timing -- this screen states that outcome plainly before Talia confirms, per the feature's "see the outcome before confirming" shared UI pattern.
- Eligibility (ownership, and Booking.state is Confirmed or Awaiting Outcome) is governed by FEAT-30.SPEC-006 and re-checked on entry and again on confirm.
- The optional private reason is never shown to the client; it is recorded on the Booking for Talia's own record only (per the Brief's Data Notes).

## Edge Cases

- **A client cancels this same booking through FEAT-10 while Talia is viewing this screen** -- Reject-with-refresh: Talia's confirm attempt is denied and she sees the booking's current (client-cancelled) state, per FEAT-30.SPEC-007's Contention handling; nothing is merged.
- **Talia navigates away with the reason field filled in but unconfirmed** -- No confirmation dialog is shown, since no cancellation has been submitted; the reason is discarded, consistent with this being a confirm-only action, not a draft-preserving form.
- **Talia taps "Cancel booking" twice rapidly** -- The second tap is ignored while the first commit is in progress (button in loading state).
- **The booking becomes ineligible (e.g., auto-completes) between screen load and Talia's tap** -- The confirm-time eligibility re-check (FEAT-30.SPEC-006) catches this and denies with the exact current-state message, even though the screen initially showed the action as available.
- **Network failure during the commit** -- Error state shown: "Couldn't cancel this booking. Try again." with Retry; the booking remains exactly as it was.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (inbound) | Eligibility check on entry and confirm |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Triggers (outbound) | Confirm triggers the commit |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound/outbound) | Primary entry point and return destination |
| FEAT-21.SPEC-010 (Pro Recurring Series Management) -- within FEAT-21 (Recurring/Standing Appointments) | Navigation (outbound) | The "Recurring series" link opens the Pro's series set-up and management for this booking; recurring management itself stays outside this feature |
| FEAT-13 (Client Record Management) | Navigation (inbound) | Reaches this screen when deleting a client with an upcoming booking |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Navigation (inbound) | Reaches this screen for an individual booking during account closure |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| pro_cancel_screen_opened | entry source (dashboard / client_deletion / account_closure) | Screen loads | supports success-metrics.md: "Pro Change Correctness" |
| pro_cancel_confirmed | time from screen open to confirm | Talia taps "Cancel booking" and the commit succeeds | supports success-metrics.md: "Pro Change Correctness" |
| pro_cancel_denied | reason category | Eligibility check denies the action | supports success-metrics.md: "Automatic Refund Correctness" |

## Acceptance Criteria

**FEAT-30.SPEC-001-AC-01:** Given Talia opens this screen for a Confirmed booking she owns, when the screen loads, then she sees the booking summary and the plain statement that the client's deposit will be refunded in full.

**FEAT-30.SPEC-001-AC-02:** Given Talia enters a private reason and taps "Cancel booking", when the commit succeeds, then she sees a confirmation and returns to FEAT-12, and the reason is never shown to Riley.

**FEAT-30.SPEC-001-AC-03:** Given Talia opens this screen for a booking already Completed, when eligibility is checked, then she sees the exact denial message from FEAT-30.SPEC-006 and no confirm control is offered.

**FEAT-30.SPEC-001-AC-04:** Given Riley cancels the same booking through FEAT-10 while Talia is viewing this screen, when Talia taps "Cancel booking", then she sees the booking's current (client-cancelled) state rather than a merged or overwritten outcome.

**FEAT-30.SPEC-001-AC-05:** Given Talia taps "Cancel booking" twice rapidly, when the first tap's commit is in progress, then the second tap has no effect until the first resolves.

**FEAT-30.SPEC-001-AC-06:** Given the commit fails due to a processing error, when the failure occurs, then Talia sees "Couldn't cancel this booking. Try again." with a Retry action, and the booking remains unchanged.

**FEAT-30.SPEC-001-AC-07:** Given Talia loses connectivity while viewing this screen, when she attempts to confirm, then she sees "Check your connection and try again." and no commit is attempted.

**FEAT-30.SPEC-001-AC-08:** Given Talia taps "Never mind", when the navigation completes, then she returns to FEAT-12 with nothing changed.

**FEAT-30.SPEC-001-AC-09:** Given Riley (the Client) has no path to this screen, when the product's screens are reviewed for a reachable control, then none exists.

**FEAT-30.SPEC-001-AC-10:** Given Platform Operator (Support) views this screen through FEAT-19's read-only surfaces, when Support looks for a cancel control, then none is shown.

**FEAT-30.SPEC-001-AC-11:** Given Talia reaches this screen from confirming a client deletion (FEAT-13), when she confirms the cancellation, then the deposit refund proceeds identically to any other entry point.

**FEAT-30.SPEC-001-AC-12:** Given a booking becomes ineligible between screen load and Talia's confirm tap, when the confirm-time eligibility check runs, then it denies with the current, correct reason rather than proceeding against a stale screen state.

**FEAT-30.SPEC-001-AC-13:** Given Talia navigates away from this screen with an unconfirmed reason typed in, when she leaves, then no confirmation dialog appears and the cancellation is not submitted.

**FEAT-30.SPEC-001-AC-14:** Given Talia opens this screen, when the booking's details are being fetched, then she sees a loading placeholder in place of the booking summary and outcome statement.

**FEAT-30.SPEC-001-AC-15:** Given the booking's details fail to load, when the load fails, then Talia sees "Couldn't load this booking. Try again." with a Retry action, and no confirm control is offered until it succeeds.

**FEAT-30.SPEC-001-AC-16:** Given Talia is on this screen for a Confirmed booking, when she taps the "Recurring series" link ("Repeat this booking" for a one-off booking, "Manage recurring series" for a booking in a series), then she is navigated to FEAT-21.SPEC-010 with the Booking reference and nothing is cancelled or changed on this screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 8 (loading, ready, ineligible, committing, committed, rejected, error, offline) | 8 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
