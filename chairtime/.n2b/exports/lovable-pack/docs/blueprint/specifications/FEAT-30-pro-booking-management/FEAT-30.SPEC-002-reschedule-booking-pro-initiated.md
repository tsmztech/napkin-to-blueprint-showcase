---
document_type: spec
spec_type: screen
spec_id: FEAT-30.SPEC-002
spec_name: Reschedule Booking (Pro-Initiated)
spec_slug: reschedule-booking-pro-initiated
parent_feature: FEAT-30
parent_feature_name: Pro Booking Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: Reschedule Booking (Pro-Initiated)

## Overview

**Name:** Reschedule Booking (Pro-Initiated)
**ID:** FEAT-30.SPEC-002
**Type:** Screen
**Purpose:** Talia picks a new genuinely free time for a client's booking, inside her own notice/horizon exception, seeing that the deposit carries over and a fresh manage link will go to the client.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Re-using the live slot list mechanism (FEAT-03) with the Pro-only notice/horizon exception applied
- Showing that the deposit carries over automatically and a fresh manage link will be issued, before Talia confirms
- Triggering the commit and reflecting its success, rejection, or failure

**Non-Goals:**
- Computing which times are genuinely free -- owned entirely by FEAT-03 (Real-Time Slot Availability Engine); this screen only displays what FEAT-03 returns, with the Pro-only exception layered on by FEAT-30.SPEC-006
- Cancelling the booking instead of rescheduling it -- owned by FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated)
- Issuing the fresh manage link itself -- owned by FEAT-06 (Client Booking Identity) and FEAT-08 (Automated Booking Messaging), triggered by a successful commit (XBR-18); this screen only states that it will happen

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard) | Talia taps a booking row and chooses "Reschedule" | Booking reference, current appointment time |
| FEAT-17.SPEC-003 (Time Block Conflict Review) | Conflict review confirms with a booking chosen "Reschedule" | The conflicting Booking reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen, for bookings she owns only | Select a new time and confirm, or back out with nothing changed | -- |
| The Client (Riley) | No | No | No control on any Client-facing surface reaches this screen; the Client reschedules her own bookings through FEAT-10 |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view | View only | Slot selection and confirm controls are not shown, consistent with SC-05 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an unconfirmed new-time selection is discarded, since no reschedule has been submitted yet |

## Layout and Content

**Header:** Back arrow (returns to FEAT-12) with the title "Reschedule booking" and the client's name and current appointment time shown beneath it as persistent context.

**Body:** A live list of available time slots for the same service, grouped by day, in chronological order -- the same slot-list mechanism as FEAT-05.SPEC-002 and FEAT-30.SPEC-004, with Talia's own notice/horizon exception applied. Below the list, once a new time is selected: a confirmation summary stating "{client_name}'s deposit carries over -- they'll get a new confirmation with this time." with "Confirm reschedule" and "Choose a different time" actions.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Slots list in a single column, grouped by day heading, full width; confirmation summary stacks below.
- **Medium size class and above:** Slots list may show more times per row within the same day grouping; confirmation summary remains single-column, capped at a consistent platform-wide width.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12 | Screen closes | Standard backward transition |
| Time slot | Tap | Re-validates the candidate against FEAT-03.SPEC-004 (with the Pro-only notice/horizon exception, per FEAT-30.SPEC-006) | Slot selected, confirmation summary appears | If valid: summary shown. If contested: plain "just taken" message, list refreshed |
| "Confirm reschedule" button | Tap | Triggers FEAT-30.SPEC-006 eligibility re-check, then FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Button shows loading state during commit | Success: confirmation shown, then navigate to FEAT-12. Rejection: exact denial message shown inline. Failure: retry prompt shown, booking unchanged |
| "Confirm reschedule" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "Choose a different time" link | Tap | Clears the selected time | Returns to the slot list, selection cleared | Slot list remains visible for re-selection |

### Accessibility Notes

- **Focus order:** Back arrow -> current-appointment context -> day groupings top to bottom -> time slots -> (once selected) confirmation summary -> Confirm reschedule button -> Choose a different time link.
- **Announcements:** The confirmation summary and any denial or contention message are announced to assistive technology when they appear.
- **Keyboard alternatives:** Every time slot and action is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | A neutral loading placeholder in place of the slot list | Screen first opens | Slot list loads successfully or a load error occurs |
| Selecting | Slot list rendered, no time yet selected | Live slot data returns at least one available time | Talia taps a slot |
| Slot contested | A plain message "That time was just taken." appears briefly, list refreshes | The tapped slot is lost to another client or booking | Talia picks a different slot |
| Selected | Confirmation summary shown with Confirm/Choose-different-time actions | Talia's candidate slot passes re-validation | Talia confirms or chooses a different time |
| Ineligible | The exact denial message from FEAT-30.SPEC-006 shown in place of the slot list (e.g., booking already Completed) | Eligibility check on screen entry fails | Talia backs out (only path forward) |
| Committing | Confirm button shows a loading state, all inputs disabled | Talia taps "Confirm reschedule" | Commit completes (success, rejection, or failure) |
| Committed | Confirmation shown ("Booking moved. {client_name} will get the new details.") before returning to FEAT-12 | FEAT-30.SPEC-007 reports success | Talia is navigated to FEAT-12 |
| Rejected | Inline message showing the booking's current state | FEAT-30.SPEC-007 reports a conflicting transition already won, or the held slot was lost between selection and commit | Talia backs out, or (for a lost slot) returns to slot selection with a refreshed list |
| Error | Inline error message "Couldn't load available times. Try again." or "Couldn't reschedule this booking. Try again." with Retry | Live slot data fails to load, or the commit reports a processing failure | Talia taps Retry and it succeeds, or she backs out |
| Offline/Degraded | Banner "Check your connection and try again." replaces the slot list or confirm control | Connectivity is lost while the screen is open | Connectivity is restored and Talia can select or confirm |

## Validation Rules

Slot validity governed by FEAT-03.SPEC-004 (Slot Validation & Timing Rules), with the Pro-only notice/horizon exception applied per FEAT-30.SPEC-006. Action eligibility (ownership, Booking.state) governed by FEAT-30.SPEC-006. Both are checked on screen entry and again on confirm.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | FEAT-12 (Pro Daily Schedule Dashboard) | -- |
| Successful reschedule | FEAT-12 (Pro Daily Schedule Dashboard) | -- |

## Data Model

**Reads:** Booking -- state, start_time, service, client, deposit_amount; live slot list for the same service (FEAT-03); Availability Rule -- minimum_booking_notice, booking_horizon (exempted for this screen's re-validation, per FEAT-30.SPEC-006).
**Creates:** None directly.
**Updates:** None directly -- delegated entirely to FEAT-30.SPEC-007 on confirm, which updates Booking.start_time.
**Deletes:** None.

## Business Rules

- XBR-09: a Pro-made reschedule never exposes the client to the cancellation window -- the deposit always carries over automatically, whatever the new time.
- XBR-18: a successful reschedule triggers a fresh manage link for the client, since the previous link's context (the old appointment time) is now stale.
- XBR-03: Talia may reschedule inside her own minimum_booking_notice or beyond her booking_horizon; the candidate slot's fit rule (duration + buffer, no conflict) is never exempted.
- The "see the outcome before confirming" pattern (Brief's Shared UI Patterns) applies here identically to FEAT-30.SPEC-001: the deposit-carries-over outcome is shown before Talia confirms.

## Edge Cases

- **Talia's held candidate slot is lost to a contesting booking between selection and confirm** -- Per FEAT-03.SPEC-005's contention resolution, Talia is returned to slot selection with a refreshed list and a "just taken" message; nothing is committed.
- **A client reschedules or cancels this same booking through FEAT-10 while Talia is mid-selection** -- Reject-with-refresh at confirm time: Talia's commit attempt is denied and she sees the booking's current state, per FEAT-30.SPEC-007's Contention handling.
- **Talia navigates away with a time selected but unconfirmed** -- No confirmation dialog is shown; the selection is discarded and the booking is untouched.
- **Talia taps "Confirm reschedule" twice rapidly** -- The second tap is ignored while the first commit is in progress.
- **The booking becomes ineligible (e.g., cancelled by the client) between screen load and Talia's slot selection** -- The confirm-time eligibility re-check catches this and denies with the exact current-state message.
- **Network failure during the commit** -- Error state shown: "Couldn't reschedule this booking. Try again." with Retry; the booking remains at its original time.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (inbound) | Notice/horizon exception and eligibility check |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Triggers (outbound) | Confirm triggers the commit |
| FEAT-03 (Real-Time Slot Availability Engine) | References (inbound) | Source of the live slot list this screen displays |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound/outbound) | Entry point and return destination |
| FEAT-06 (Client Booking Identity) / FEAT-08 (Automated Booking Messaging) | Affects (outbound) | A successful commit triggers a fresh client manage link |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| pro_reschedule_screen_opened | () | Screen loads | supports success-metrics.md: "Pro Change Correctness" |
| pro_reschedule_slot_selected | time-to-selection since screen open | Talia taps a valid slot | supports success-metrics.md: "Pro Change Correctness" |
| pro_reschedule_confirmed | time from screen open to confirm | The commit succeeds | supports success-metrics.md: "Pro Change Correctness" |
| pro_reschedule_slot_lost_to_contention | () | The selected slot is lost between selection and confirm | supports success-metrics.md: "Zero Double-Booking Confidence" |

## Acceptance Criteria

**FEAT-30.SPEC-002-AC-01:** Given Talia opens this screen for a Confirmed booking, when the screen loads, then she sees a live slot list for the same service with her own notice/horizon exception applied.

**FEAT-30.SPEC-002-AC-02:** Given Talia selects a candidate time inside her own minimum_booking_notice, when the slot is re-validated, then it is accepted, per the Pro-only exception.

**FEAT-30.SPEC-002-AC-03:** Given Talia selects a valid new time, when she taps "Confirm reschedule", then the commit succeeds, the booking's start_time updates, and she sees confirmation before returning to FEAT-12.

**FEAT-30.SPEC-002-AC-04:** Given Talia selects a time, when the confirmation summary appears, then it states the deposit carries over and the client will get a new confirmation with the new time.

**FEAT-30.SPEC-002-AC-05:** Given Talia's selected slot is lost to another booking before she confirms, when she attempts to confirm, then she sees "That time was just taken." and returns to a refreshed slot list.

**FEAT-30.SPEC-002-AC-06:** Given Riley cancels the same booking through FEAT-10 while Talia is mid-selection, when Talia taps "Confirm reschedule", then she sees the booking's current (client-cancelled) state rather than a merged outcome.

**FEAT-30.SPEC-002-AC-07:** Given Talia opens this screen for a booking already Completed, when eligibility is checked, then she sees the exact denial message from FEAT-30.SPEC-006 and no slot list is offered.

**FEAT-30.SPEC-002-AC-08:** Given a successful reschedule commit, when it completes, then Riley receives a fresh manage link, per XBR-18.

**FEAT-30.SPEC-002-AC-09:** Given Talia taps "Confirm reschedule" twice rapidly, when the first tap's commit is in progress, then the second tap has no effect until the first resolves.

**FEAT-30.SPEC-002-AC-10:** Given the commit fails due to a processing error, when the failure occurs, then Talia sees "Couldn't reschedule this booking. Try again." with a Retry action, and the booking remains at its original time.

**FEAT-30.SPEC-002-AC-11:** Given Talia loses connectivity while viewing this screen, when she attempts to select a slot or confirm, then she sees "Check your connection and try again." and nothing is committed.

**FEAT-30.SPEC-002-AC-12:** Given Riley (the Client) has no path to this screen, when the product's screens are reviewed for a reachable control, then none exists.

**FEAT-30.SPEC-002-AC-13:** Given Talia taps "Choose a different time" after selecting a slot, when the tap registers, then the selection clears and she returns to the slot list.

**FEAT-30.SPEC-002-AC-14:** Given Talia navigates away from this screen with a time selected but unconfirmed, when she leaves, then no confirmation dialog appears and the booking is untouched.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 10 (loading, selecting, contested, selected, ineligible, committing, committed, rejected, error, offline) | 10 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
