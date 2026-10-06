---
document_type: spec
spec_type: screen
spec_id: FEAT-30.SPEC-005
spec_name: Cancel Several Bookings at Once
spec_slug: cancel-several-bookings-at-once
parent_feature: FEAT-30
parent_feature_name: Pro Booking Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Cancel Several Bookings at Once

## Overview

**Name:** Cancel Several Bookings at Once
**ID:** FEAT-30.SPEC-005
**Type:** Screen
**Purpose:** Talia reviews the set of bookings a new time block conflicts with and confirms cancelling them together, seeing the full-refund outcome for each before confirming.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Reviewing the conflicting bookings a new Time Block overlaps, as handed over by FEAT-17
- Showing the always-full-refund outcome for each booking before Talia confirms
- Capturing an optional shared private cancellation reason
- Triggering the bulk commit and reflecting its per-booking outcome

**Non-Goals:**
- Creating the Time Block itself -- owned by FEAT-17 (Manual Time Blocking); this screen only reviews the bookings that block already conflicts with
- Cancelling a single booking outside a time-block conflict -- owned by FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated), a distinct one-booking flow
- Letting Talia keep a conflicting booking as an exception to the block instead of cancelling it -- owned by FEAT-17's own conflict-resolution choice, which routes to this screen only when Talia chooses to cancel the affected bookings

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-17.SPEC-003 (Time Block Conflict Review) (Manual Time Blocking) | Talia's new time block conflicts with existing bookings and she chooses to cancel the affected ones | The set of conflicting Booking references |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen, for bookings she owns only | Confirm cancelling the reviewed set, or back out with nothing changed | -- |
| The Client (Riley) | No | No | No control on any Client-facing surface reaches this screen |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view | View only | Confirm control is not shown, consistent with SC-05 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an unconfirmed reason is discarded, since no cancellation has been submitted yet |

## Layout and Content

**Header:** Back arrow (returns to FEAT-17) with the title "Cancel {count} conflicting bookings."

**Body:** A list of the conflicting bookings, one row per booking -- client name, service, date and time, deposit amount -- each row showing the plain outcome statement "{client_name}'s deposit will be refunded in full." Below the list, an optional single-line text field labeled "Reason (private, shared reason for all)." Below that, a "Cancel all" confirm button and a "Never mind" link to back out.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Booking list in a single column, full width, one row per booking; confirm controls stack below the list.
- **Medium size class and above:** List remains single-column, capped at a consistent platform-wide width and horizontally centered; no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-17 (Manual Time Blocking) | Screen closes | Standard backward transition |
| Reason field | Type | Captures optional shared private text | Field shows entered text | Standard input focus state |
| "Cancel all" button | Tap | Triggers FEAT-30.SPEC-006 eligibility re-check per booking, then FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Button shows loading state during commit | Per-booking outcome shown: succeeded rows marked cancelled, failed rows show their specific denial reason |
| "Cancel all" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| A failed booking row's "View" action | Tap | Navigate to FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated) for that individual booking | Screen navigates | Talia can act on the failed booking separately |
| "Never mind" link | Tap | Navigate to FEAT-17 with nothing changed | Screen closes | Standard backward transition |

### Accessibility Notes

- **Focus order:** Back arrow -> booking list rows top to bottom, each with its outcome statement -> Reason field -> Cancel all button -> Never mind link.
- **Announcements:** The per-booking outcome summary is announced to assistive technology once the bulk commit completes.
- **Keyboard alternatives:** Every action on this screen, including each failed row's "View" action, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | A neutral loading placeholder in place of the booking list | Screen opens | The conflicting bookings' data loads successfully or a load error occurs |
| Ready | Booking list with per-row outcome statements and confirm/back-out controls shown | The conflicting bookings' data loads successfully with at least one eligible | Talia confirms or backs out |
| All ineligible | Every booking row shows a denial message in place of the outcome statement; "Cancel all" is disabled | Every booking in the set fails eligibility on screen entry | Talia backs out (only path forward) |
| Committing | Confirm button shows a loading state, all inputs disabled | Talia taps "Cancel all" | Bulk commit completes |
| Committed -- all succeeded | Every row shows "Cancelled -- refund confirmed" before returning to FEAT-17 | FEAT-30.SPEC-008 reports every booking succeeded | Talia is navigated to FEAT-17 |
| Committed -- partial success | Succeeded rows show "Cancelled -- refund confirmed"; failed rows show their specific reason with a "View" action to act on them individually | FEAT-30.SPEC-008 reports at least one failure alongside successes | Talia reviews the summary and either backs out or opens a failed row |
| Error | Inline error banner "Couldn't load these bookings. Try again." or "Couldn't process this cancellation. Try again." with a Retry action | The conflicting bookings' data fails to load, or the whole-set commit fails to begin for a processing reason | Talia taps Retry and it succeeds, or she backs out |
| Offline/Degraded | Banner "Check your connection and try again." replaces the confirm control; the booking list remains viewable read-only | Connectivity is lost while the screen is open | Connectivity is restored and Talia can confirm |

## Validation Rules

Eligibility per booking (ownership, and Booking.state is Confirmed or Awaiting Outcome) governed by FEAT-30.SPEC-006 (Pro Booking Action Rules). See that spec for the exact conditions and denial messages. This screen applies the eligibility check per booking on entry and again on confirm.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | FEAT-17 (Manual Time Blocking) | -- |
| "Never mind" tap | FEAT-17 (Manual Time Blocking) | -- |
| Bulk cancellation completes (fully or partially) | FEAT-17 (Manual Time Blocking) | -- |
| Failed row's "View" action | FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated) | -- |

## Data Model

**Reads:** Booking (state, start_time, service, client, deposit_amount) for each booking in the conflicting set, handed over by FEAT-17; Deposit Transaction -- status, per booking (eligibility precondition).
**Creates:** None directly.
**Updates:** None directly -- delegated entirely to FEAT-30.SPEC-008 on confirm.
**Deletes:** None.

## Business Rules

- XBR-09: every cancellation this screen commits refunds its client's deposit in full, whatever the timing -- shown per booking before Talia confirms, per the feature's "see the outcome before confirming" shared UI pattern.
- Each booking's eligibility and outcome is independent -- one booking's failure never blocks or reverses the others, per FEAT-30.SPEC-008's per-booking processing.
- Eligibility per booking is governed by FEAT-30.SPEC-006 and re-checked on entry and again on confirm.
- The optional shared reason, if provided, is recorded identically on every successfully cancelled booking; it is never shown to any client.

## Edge Cases

- **One booking in the set is already Completed when Talia opens this screen** -- That row shows the completed-state denial message from FEAT-30.SPEC-006 in place of the outcome statement; the remaining eligible bookings can still be cancelled if Talia confirms.
- **A client cancels one of the set's bookings through FEAT-10 while Talia is reviewing this screen** -- The confirm-time eligibility re-check for that specific booking fails (reject-with-refresh); it is reported as a failed outcome while the rest of the set proceeds normally.
- **Talia navigates away with the reason field filled in but unconfirmed** -- No confirmation dialog is shown, since no cancellation has been submitted; nothing changes.
- **Talia taps "Cancel all" twice rapidly** -- The second tap is ignored while the first commit is in progress.
- **The set contains only one conflicting booking** -- The screen and its per-row outcome mechanics render identically to a larger set; FEAT-17 is what decides whether a single conflicting booking routes here or to FEAT-30.SPEC-001 directly.
- **Network failure during the bulk commit** -- Error banner shown: "Couldn't process this cancellation. Try again." with Retry; no booking in the set is affected.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (inbound) | Per-booking eligibility check |
| FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Triggers (outbound) | Confirm triggers the bulk commit |
| FEAT-30.SPEC-001 (Cancel Booking, Pro-Initiated) | Navigation (outbound) | A failed booking row can be revisited individually here |
| FEAT-17 (Manual Time Blocking) | Navigation (inbound/outbound) | Hands over the conflicting booking set; return destination after review |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| pro_bulk_cancel_screen_opened | booking_count | Screen loads | supports success-metrics.md: "Pro Change Correctness" |
| pro_bulk_cancel_confirmed | booking_count, success_count, failure_count | Talia taps "Cancel all" and the commit completes | supports success-metrics.md: "Pro Change Correctness" |
| pro_bulk_cancel_row_failed | reason category | A booking in the set fails eligibility | supports success-metrics.md: "Automatic Refund Correctness" |

## Acceptance Criteria

**FEAT-30.SPEC-005-AC-01:** Given Talia opens this screen for four bookings her new time block conflicts with, when the screen loads, then she sees each booking listed with the plain statement that its client's deposit will be refunded in full.

**FEAT-30.SPEC-005-AC-02:** Given Talia confirms "Cancel all" and every booking passes eligibility, when the commit completes, then every row shows "Cancelled -- refund confirmed" and she returns to FEAT-17.

**FEAT-30.SPEC-005-AC-03:** Given one booking in the set is already Completed, when the screen loads, then that row shows the completed-state denial message while the others show the standard outcome statement.

**FEAT-30.SPEC-005-AC-04:** Given Talia confirms "Cancel all" and one booking fails eligibility while the others succeed, when the commit completes, then the failed row shows its specific reason with a "View" action, and the succeeded rows show "Cancelled -- refund confirmed."

**FEAT-30.SPEC-005-AC-05:** Given a booking failed within this bulk cancellation, when Talia taps its "View" action, then she is taken to FEAT-30.SPEC-001 for that individual booking.

**FEAT-30.SPEC-005-AC-06:** Given Riley cancels one of the set's bookings through FEAT-10 while Talia is reviewing this screen, when Talia confirms "Cancel all", then that booking is reported as a failed outcome while the rest of the set is cancelled successfully.

**FEAT-30.SPEC-005-AC-07:** Given Talia enters a shared reason and confirms, when the commit succeeds, then that same reason is recorded on every successfully cancelled booking and never shown to any client.

**FEAT-30.SPEC-005-AC-08:** Given every booking in the set fails eligibility, when the screen loads, then "Cancel all" is disabled and every row shows its denial message.

**FEAT-30.SPEC-005-AC-09:** Given Talia taps "Cancel all" twice rapidly, when the first commit is in progress, then the second tap has no effect until the first resolves.

**FEAT-30.SPEC-005-AC-10:** Given the whole-set commit fails due to a processing error, when the failure occurs, then Talia sees "Couldn't process this cancellation. Try again." with a Retry action, and no booking in the set is affected.

**FEAT-30.SPEC-005-AC-11:** Given Talia loses connectivity while reviewing this screen, when she attempts to confirm, then she sees "Check your connection and try again." and no commit is attempted.

**FEAT-30.SPEC-005-AC-12:** Given Riley (the Client) has no path to this screen, when the product's screens are reviewed for a reachable control, then none exists.

**FEAT-30.SPEC-005-AC-13:** Given Talia taps "Never mind", when the navigation completes, then she returns to FEAT-17 with nothing changed.

**FEAT-30.SPEC-005-AC-14:** Given Talia opens this screen, when the conflicting bookings' data is being fetched, then she sees a loading placeholder in place of the booking list.

**FEAT-30.SPEC-005-AC-15:** Given the conflicting bookings' data fails to load, when the load fails, then Talia sees "Couldn't load these bookings. Try again." with a Retry action, and no "Cancel all" control is offered until it succeeds.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 8 (loading, ready, all-ineligible, committing, committed-all, committed-partial, error, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
