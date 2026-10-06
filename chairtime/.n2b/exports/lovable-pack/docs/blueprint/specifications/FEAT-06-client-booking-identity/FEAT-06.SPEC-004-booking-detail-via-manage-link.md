---
document_type: spec
spec_type: screen
spec_id: FEAT-06.SPEC-004
spec_name: Booking Detail via Manage Link
spec_slug: booking-detail-via-manage-link
parent_feature: FEAT-06
parent_feature_name: Client Booking Identity
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Booking Detail via Manage Link

## Overview

**Name:** Booking Detail via Manage Link
**ID:** FEAT-06.SPEC-004
**Type:** Screen
**Purpose:** Client views one booking's full detail, opened either from the My Bookings list or directly via a booking-specific manage link, and starts a cancel, reschedule, balance payment, or preferences action from it.
**Parent Feature:** FEAT-06 -- Client Booking Identity

## Scope and Non-Goals

**In Scope:**
- Displaying one booking's full detail for the matched Client
- Entry points into cancelling/rescheduling (FEAT-10), paying the balance (FEAT-22, v1), and preferences (FEAT-06.SPEC-005)
- The concurrent-edit conflict behavior when the Pro changes the booking while this screen is open

**Non-Goals:**
- Listing multiple bookings -- owned by FEAT-06.SPEC-003 (My Bookings List)
- Performing the actual cancel or reschedule -- owned by FEAT-10; this screen only starts that flow
- Performing the actual balance payment -- owned by FEAT-22 (v1); this screen only starts that flow
- Any Pro-side view of this booking -- the Pro's equivalent is FEAT-12 (Pro Daily Schedule Dashboard) and FEAT-30 (Pro Booking Management), entirely separate screens reached only through Pro sign-in

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-003 (My Bookings List) | Client taps an upcoming or past booking row | The selected Booking's reference |
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | A valid, unexpired, unused booking-specific manage link is redeemed | The one Booking the link is scoped to |
| FEAT-10.SPEC-001 (Cancel Booking) | Client completes a cancellation, taps "Keep my booking", or taps back | The same Booking reference, showing its current state |
| FEAT-10.SPEC-003 (Reschedule -- Outcome & Confirm) | Client confirms an outside-window reschedule successfully | The same Booking reference with its new start_time |
| FEAT-10.SPEC-006 (Cancellation/Reschedule Notification) | Client taps "Manage my booking" in the cancellation/reschedule notice | The affected Booking reference (the original booking for a cancellation, the new booking for a late reschedule) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full detail of a booking that belongs to their own matched Client record with this Pro | Start cancel/reschedule, start balance payment (v1), open preferences | -- |
| The Pro (Talia) | No | No | This is not the Pro's own booking view; the Pro's equivalent is reached through FEAT-12/FEAT-30 via Pro sign-in, never through this screen |
| Platform Operator (Support) | No | No | Support access never uses or bypasses client identity (scope-boundaries SC-05); no support entry point exists here |
| Unauthenticated | No | No | Reachable only via FEAT-06.SPEC-003 (already-authenticated navigation) or a valid booking-specific link redeemed by FEAT-06.SPEC-002; a direct, unauthenticated attempt is redirected to FEAT-06.SPEC-001 |
| Expired session | No | No | The viewing session lasts only for the current page; reloading after the underlying access link has transitioned to Used is treated as unauthenticated and redirected to FEAT-06.SPEC-001 with the "request a new link" prompt |

## Layout and Content

**Header:** Screen title showing the service name, with a back arrow (returns to FEAT-06.SPEC-003 when arrived from there, or shows no back arrow when arrived directly via a booking-specific link, since there is no list to return to in that session).

**Body:**
- Appointment summary: date, time, duration, and the studio address (shown per this booking's confirmation-only disclosure)
- Payment summary: price agreed, deposit amount and paid status, balance due
- Cancellation policy summary: the plain-language wording and window that was acknowledged at booking, and the resulting outcome if cancelled now (refund vs. kept), consistent with FEAT-09's policy engine
- Status line: current booking state (Confirmed, Awaiting Outcome, Completed, No-Show, Cancelled, Rescheduled)
- Action row: "Cancel or Reschedule" button (only when the booking's state allows it, per FEAT-10/FEAT-09's rules); "Pay Balance" button (v1, only when a balance is due and unpaid); a "Preferences" link to FEAT-06.SPEC-005

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically in the order given above, full width; the action row's buttons stack full width.
- **Medium size class and above:** Same vertical section order, content column capped at a consistent platform-wide reading width and horizontally centered; action row buttons sit side by side instead of stacking.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow (when present) | Tap | Navigate to FEAT-06.SPEC-003 | Screen closes | Standard navigation transition |
| "Cancel or Reschedule" button | Tap | Navigate to FEAT-10 (Client-Initiated Cancel/Reschedule) for this booking | Screen transitions | Standard navigation transition |
| "Pay Balance" button (v1) | Tap | Navigate to FEAT-22 (In-App Balance Payment) for this booking | Screen transitions | Standard navigation transition |
| "Preferences" link | Tap | Navigate to FEAT-06.SPEC-005 | Screen transitions | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Back arrow (when present) -> appointment summary -> payment summary -> cancellation policy summary -> status line -> "Cancel or Reschedule" -> "Pay Balance" (when shown) -> "Preferences".
- **Dynamic announcements:** A concurrent-edit conflict message (see Edge Cases) is announced to assistive technology as soon as it appears.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief in-place loading indicator where the detail will appear | Screen first opens | Data finishes loading |
| Populated | Full booking detail shown as described in Layout and Content | Data loads successfully | Client navigates away |
| Error | Error banner "We couldn't load this booking. Try again." with a retry action | The initial data load fails | Client taps Retry and the load succeeds |
| Booking No Longer Available | Plain message "This booking is no longer available." replaces the detail, with no further detail shown | The booking's underlying record cannot be resolved for this client (e.g., a scope mismatch) | Client navigates back to FEAT-06.SPEC-003 (when reachable) or requests a new link |
| Offline/Degraded | The already-loaded detail remains visible read-only; a banner "You're offline -- reconnect to take action on this booking." appears; the action row's buttons are disabled | Connectivity is lost while this screen is open | Connectivity is restored -- the banner clears and buttons re-enable |

## Validation Rules

Not applicable -- this screen has no user input fields. Access to it is governed by FEAT-06.SPEC-002 and FEAT-06.SPEC-008; the actions it exposes and their own rules belong to FEAT-10, FEAT-22, and FEAT-09.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap (when present) | FEAT-06.SPEC-003 (My Bookings List) | -- |
| "Cancel or Reschedule" tap | Cancel/reschedule flow, FEAT-10.SPEC-001 (Cancel Booking) | FEAT-10 (Client-Initiated Cancel/Reschedule) |
| "Pay Balance" tap (v1) | Balance payment flow, FEAT-22.SPEC-001 (Balance Payment) | FEAT-22 (In-App Balance Payment) |
| "Preferences" tap | FEAT-06.SPEC-005 (Consent & Email Preferences) | -- |

## Data Model

**Creates:** None.
**Reads:** Booking -- service, start_time, duration, price_agreed, deposit_amount, policy_version, state, balance_due, cancellation/reschedule timestamps, scoped to the matched Client with this Pro (FEAT-06.SPEC-008). Cancellation Policy -- the version referenced by the booking, for the plain-language wording and outcome preview.
**Updates:** None directly -- cancel, reschedule, and balance payment are all performed by the destination specs this screen navigates to.
**Deletes:** None.

## Business Rules

- Every field shown is scoped to the one Booking resolved by the redeemed link or by selection from FEAT-06.SPEC-003 -- never another client's or another Pro's booking.
- The "Cancel or Reschedule" button is shown only when the booking's current state permits it (per FEAT-09's cancellation policy engine and XBR-12's outcome windows); a Completed or No-Show booking never shows this button.
- The "Pay Balance" button (v1) is shown only when balance_due is greater than zero and unpaid, per XBR-23.
- The studio address is shown here because this is a booked client's own confirmation-equivalent view, consistent with the Pro Account's confirmation-only disclosure rule.
- XBR-18: this screen's contents are scoped to the client identified by the redeemed access link (or by the prior selection from FEAT-06.SPEC-003, itself scoped the same way).

## Edge Cases

- **Talia cancels or reschedules this booking on her side while Riley has this screen open** -- The Booking entity's dependency-map Contention note calls for reject-with-refresh under high contention. If Riley taps "Cancel or Reschedule" after Talia's change has committed, her action is rejected and she sees a dialog: "This booking's details changed. Refresh to see the latest before continuing." with a "Refresh" action that reloads the current state; her original tap is not carried through to FEAT-10 against stale data.
- **Booking is marked Completed or No-Show by Talia while Riley is viewing** -- The already-loaded screen does not silently rewrite itself mid-view; the updated status and the disappearance of the "Cancel or Reschedule" button take effect the next time the screen is loaded (fresh navigation or reload), consistent with this being a snapshot view, not a live-updating one.
- **Booking-specific link is redeemed for a booking that has since been fully cancelled and its record scope no longer matches** -- The screen shows the "Booking No Longer Available" state rather than stale or partial detail.
- **Client taps "Cancel or Reschedule" twice rapidly** -- The second tap is ignored while the navigation to FEAT-10 is already in progress.
- **Client loses connectivity mid-view, then taps "Pay Balance"** -- The button is disabled while offline (per the Offline/Degraded state), so the tap has no effect until connectivity returns.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-003 (My Bookings List) | Navigation (inbound) | Selecting a booking there opens this screen |
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | Navigation (inbound) | A valid booking-specific redemption routes directly here |
| FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule) | References (inbound) | Scopes the displayed booking to the matched Client with this Pro |
| FEAT-06.SPEC-005 (Consent & Email Preferences) | Navigation (outbound) | The "Preferences" link opens the client's own settings |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Navigation (outbound) | "Cancel or Reschedule" starts that flow for this booking |
| FEAT-22 (In-App Balance Payment) | Navigation (outbound) | "Pay Balance" starts that flow for this booking (v1) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| booking_detail_viewed | entry source (list / manage_link), booking_state | Screen finishes loading with data | supports success-metrics.md: "Self-Service Access Success" |
| booking_detail_conflict_shown | -- | Riley's action is rejected because Talia's change committed first | supports success-metrics.md: "Self-Service Access Success" |

## Acceptance Criteria

**FEAT-06.SPEC-004-AC-01:** Given Riley taps a valid booking-specific manage link, when FEAT-06.SPEC-002 redeems it, then she lands directly on this booking's detail with no code or password step.

**FEAT-06.SPEC-004-AC-02:** Given Riley selects a booking from FEAT-06.SPEC-003, when the detail screen loads, then it shows the appointment summary, payment summary, cancellation policy summary, and status line for that one booking.

**FEAT-06.SPEC-004-AC-03:** Given Riley's booking is in a Completed state, when the detail screen loads, then no "Cancel or Reschedule" button is shown.

**FEAT-06.SPEC-004-AC-04:** Given Riley's booking has a balance due and unpaid (v1), when the detail screen loads, then a "Pay Balance" button is shown; given the balance is already paid in full, then the button is not shown.

**FEAT-06.SPEC-004-AC-05:** Given Riley taps "Cancel or Reschedule" on a cancellable booking, when the tap registers, then she is taken to FEAT-10 for that booking.

**FEAT-06.SPEC-004-AC-06:** Given Talia cancels Riley's booking on her side while Riley is viewing this screen, when Riley then taps "Cancel or Reschedule", then her action is rejected with the dialog "This booking's details changed. Refresh to see the latest before continuing." and no stale request reaches FEAT-10.

**FEAT-06.SPEC-004-AC-07:** Given Riley taps "Refresh" in the conflict dialog, when the reload completes, then the screen shows the booking's current, up-to-date state.

**FEAT-06.SPEC-004-AC-08:** Given the initial load of this booking's detail fails, when the failure occurs, then an error banner "We couldn't load this booking. Try again." appears with a retry action.

**FEAT-06.SPEC-004-AC-09:** Given the booking this screen would show can no longer be resolved for Riley's Client record, when the screen attempts to load it, then the "Booking No Longer Available" message is shown instead of any booking detail.

**FEAT-06.SPEC-004-AC-10:** Given Riley loses connectivity while viewing this screen, when connectivity drops, then the loaded detail remains visible read-only, an offline banner appears, and the action buttons are disabled.

**FEAT-06.SPEC-004-AC-11:** Given Riley taps "Preferences" from this screen, when the tap registers, then she is taken to FEAT-06.SPEC-005 (Consent & Email Preferences).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 4 (loading, error, booking no longer available, offline) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
