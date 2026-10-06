---
document_type: spec
spec_type: screen
spec_id: FEAT-21.SPEC-001
spec_name: Set Up Recurring Series
spec_slug: set-up-recurring-series
parent_feature: FEAT-21
parent_feature_name: Recurring/Standing Appointments
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Set Up Recurring Series

## Overview

**Name:** Set Up Recurring Series
**ID:** FEAT-21.SPEC-001
**Type:** Screen
**Purpose:** Lets Riley turn the booking she just made into a standing appointment by choosing how often it repeats, so future visits with Talia are generated automatically instead of booked one at a time.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- Offering the recurring option immediately after a booking confirms, and capturing the repeat interval (every 1 to 12 weeks)
- Creating the Recurring Series in state Active from the just-confirmed Booking, and handing off to occurrence generation
- Showing a plain reason and keeping the client on the form when the chosen interval is out of range
- The offline/degraded behavior for this occasional, connectivity-dependent action

**Non-Goals:**
- Editing a series' interval after creation -- excluded per this Brief's Non-Goals: the feature's three Key Capabilities name only setup, group viewing/management, and cancellation; a client who wants a different cadence cancels and sets up a fresh series here again.
- Viewing or managing existing series and their occurrences -- owned by FEAT-21.SPEC-002 (My Recurring Series); this screen only ever creates a new series from a booking that was just confirmed.
- The Pro setting up a series for a client at the chair -- per the Access field's own wording, that path runs through FEAT-21.SPEC-010 (Pro Recurring Series Management, reached from FEAT-30's Pro booking detail), which invokes the same interval and horizon validation (FEAT-21.SPEC-003) from its own screen rather than this one.
- Validating the chosen interval and the booking-horizon ceiling -- owned by FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits); this screen only surfaces the result.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05.SPEC-005 (Booking Confirmation) | Riley sees her booking's on-screen confirmation (FEAT-05.SPEC-005's footer action) and taps "Make this a standing appointment" | The just-confirmed Booking's reference, service, and start time; no separate sign-in step, since this follows directly from the confirmation the client is already viewing |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen | Choose an interval and submit, for her own just-confirmed booking only | -- |
| The Pro (Talia) | No | No | Talia never reaches this screen; her equivalent action -- setting up a series for a client at the chair -- runs through FEAT-21.SPEC-010 (Pro Recurring Series Management), a distinct screen from this one (Non-Goals) |
| Platform Operator (Support) | No | No | This screen is client-facing and reached only from a client's own booking confirmation; Support's read-only view of a Recurring Series is surfaced through FEAT-19's own screen, never through this feature's screens (Capability Coverage Map) |
| Unauthenticated | Yes -- this screen carries no separate sign-in of its own | Yes, for the one booking just confirmed in the same session | There is no "unauthenticated" denial state on this screen: the client's identity for this one action is the booking session they are already in, not a signed-in account (consistent with the Client persona never holding a password-style account) |
| Expired session | Partial -- the confirmation context this screen depends on is time-bound | No, once the underlying confirmation context has lapsed | If Riley reaches this offer after the booking confirmation context has expired (for example, from a stale bookmark), she sees "This offer has expired. Open your booking to set up a recurring series." with a link to request access via FEAT-06 (Access Link Request), rather than a session sign-in prompt |

## Layout and Content

**Header:** Screen title "Make this a standing appointment?" with a back arrow (returns to FEAT-05.SPEC-005, Booking Confirmation, without setting up a series).

**Body:** A single-column form with the following elements, in order:
- Short explanatory line: "We'll book this automatically with {pro_display_name} every time it's due, using the same service and time."
- **Repeat interval** (numeric stepper input, required): a whole-number-of-weeks value, from 1 to 12, labeled "Repeat every ___ weeks." Defaults to no value pre-selected -- Riley must choose one.
- **Skip this** (secondary text link, below the stepper): declines the offer without creating a series.

**Footer:** A single primary "Set up recurring" action button, full width.

### Responsive Behavior

- **Compact size class:** Single-column form as described above, full width; the primary action stays pinned in the footer.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-05.SPEC-005 (Booking Confirmation) without creating a series | Screen closes | Animated transition back to the confirmation |
| Repeat interval stepper | Increment/decrement or type a value | Captures the chosen interval | Stepper shows the new value; "Set up recurring" becomes enabled once a value is chosen | Stepper shows the new value |
| Skip this | Tap | Declines the offer; no Recurring Series is created | Screen closes | Animated transition back to FEAT-05.SPEC-005 (Booking Confirmation), unchanged |
| Set up recurring button | Tap | 1. Validate the chosen interval via FEAT-21.SPEC-003. 2. If valid, create the Recurring Series in state Active from the just-confirmed Booking and hand off to FEAT-21.SPEC-004 for its first occurrence(s). | Button shows a loading state during submission | Success: toast "Recurring series set up -- every {interval} weeks" and navigate to FEAT-21.SPEC-002 (My Recurring Series). Failure: inline error banner or field-level message. |
| Set up recurring button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> explanatory line (read-only, not a focus stop) -> Repeat interval stepper -> Skip this -> Set up recurring.
- **Validation announcements:** When the interval is out of range, the error message is announced to assistive technology and programmatically associated with the stepper.
- **Success/failure announcements:** The "Recurring series set up" toast and any error banner are announced to assistive technology on appearance.
- **Keyboard alternatives:** The stepper's increment/decrement is reachable by keyboard (arrow keys or direct numeric entry); there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Stepper unset, "Set up recurring" disabled | Screen first opens from the booking confirmation | Riley chooses an interval value |
| Filling | Stepper shows Riley's chosen value, "Set up recurring" enabled | Riley sets an interval | Riley taps "Set up recurring" or "Skip this," or navigates away |
| Validating/Submitting | "Set up recurring" shows a loading spinner, stepper disabled | Riley taps "Set up recurring" with a chosen value | Validation and creation complete or fail |
| Validation Error | Stepper shows an error state with the message below it; "Set up recurring" re-enabled | FEAT-21.SPEC-003 rejects the chosen interval | Riley corrects the value |
| Error | Error banner at the top of the form with a Retry option; the chosen value is preserved | Series creation fails after passing validation (e.g., a processing error) | Riley taps Retry or navigates away |
| Offline/Degraded | N/A -- requires connectivity, consistent with the rest of scheduling (per this Brief's States field); a submission attempted without connectivity shows the plain message "You'll need to be online to set this up. Please check your connection and try again." and nothing is submitted or queued | Connectivity lost while attempting to submit | Connectivity restored and Riley retries |

## Validation Rules

Validation governed by FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits). See that spec for the interval range rule and its exact error message. This screen applies validation on submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-05.SPEC-005 (Booking Confirmation) | FEAT-05 (Public Booking Page & Booking Flow) |
| Skip this tap | FEAT-05.SPEC-005 (Booking Confirmation) | FEAT-05 (Public Booking Page & Booking Flow) |
| Successful setup | FEAT-21.SPEC-002 (My Recurring Series) | -- |

## Data Model

**Creates:** Recurring Series -- interval (from the stepper), originating service and time (auto-populated from the just-confirmed Booking, not independently entered), state set to Active.
**Reads:** Booking -- service, start_time, from the just-confirmed booking this screen was reached from.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Interval validation and the booking-horizon ceiling are governed entirely by FEAT-21.SPEC-003 -- this screen cannot save a series that spec would reject.
- Creating the series hands off immediately to FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) for its first occurrence(s) within the Pro's current booking horizon; Riley does not wait on this screen for that generation to complete.
- XBR-01: the originating service and time are fixed at the moment the underlying Booking was confirmed; this screen never lets Riley pick a different service or time for the series -- it repeats the booking she just made.

## Edge Cases

- **Riley navigates away with a chosen interval but before submitting** -- No confirmation dialog is shown and no series is created; unlike a data-entry form, declining this optional offer carries no risk of lost work worth interrupting for.
- **Riley taps "Set up recurring" twice rapidly** -- The second tap is ignored while the first submission is in progress (button in loading state).
- **Network failure during submission** -- Error banner: "Could not set up your recurring series. Check your connection and try again." with a Retry button. The chosen interval is preserved.
- **Riley chooses an interval of 0 or 13+ weeks** -- FEAT-21.SPEC-003 rejects it; the stepper shows the error and the client stays on this screen with her attempted value visible.
- **The underlying booking is cancelled in the moments between confirmation and this screen loading** -- The offer is withdrawn: the screen shows "This booking is no longer active, so it can't be made recurring." with a single option returning to FEAT-06 (Client Booking Identity), since there is no confirmed booking left to repeat.
- **Riley reaches this screen a second time for the same booking (for example, via back navigation) after already setting up a series from it** -- The offer is not shown again; she is routed directly to FEAT-21.SPEC-002 (My Recurring Series) instead, since a booking can originate at most one series.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-005 (Booking Confirmation) | Navigation (inbound/outbound) | Riley arrives here when she taps "Make this a standing appointment" on the confirmation; back arrow and "Skip this" return her there |
| FEAT-21.SPEC-003 (Recurring Series Setup & Generation Limits) | References (outbound) | Supplies the interval validation rule and error message |
| FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Triggers (outbound) | Successful setup hands off to generate the series' first occurrence(s) |
| FEAT-21.SPEC-002 (My Recurring Series) | Navigation (outbound) | Successful setup navigates here |
| FEAT-06 (Client Booking Identity) | Navigation (outbound) | The withdrawn-offer edge case routes here when the underlying booking is no longer active |
| FEAT-21.SPEC-010 (Pro Recurring Series Management) | References (sibling) | The Pro-side counterpart that creates a series through the same FEAT-21.SPEC-003 validation and the same hand-off to FEAT-21.SPEC-004 |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| recurring_series_created | interval_weeks, entry source (client post-booking) | Successful submission creates the Recurring Series | supports success-metrics.md: "Self-Service Reschedule Rate" (a client managing their own standing cadence in-app, without contacting the Pro, is the same self-service pattern this metric measures) |
| recurring_setup_offer_declined | reason (skip / navigated away) | Riley taps "Skip this," or navigates away without submitting | N/A -- no Stage 2 metric measures decline rate for this optional offer; retained so adoption of the capability is observable |
| recurring_setup_validation_failed | attempted interval value | FEAT-21.SPEC-003 rejects the chosen interval | N/A -- no Stage 2 metric measures setup validation failures; retained so setup friction on this screen is observable |

## Acceptance Criteria

**FEAT-21.SPEC-001-AC-01:** Given Riley just saw her booking confirmed, when the confirmation offers to make it recurring and she taps it, then she lands on this screen with the interval stepper unset and "Set up recurring" disabled.

**FEAT-21.SPEC-001-AC-02:** Given Riley sets the interval to 3 weeks and taps "Set up recurring," when validation passes, then a Recurring Series is created in state Active from her booking, she sees the toast "Recurring series set up -- every 3 weeks," and she lands on FEAT-21.SPEC-002 (My Recurring Series).

**FEAT-21.SPEC-001-AC-03:** Given Riley sets the interval to 0 weeks and taps "Set up recurring," then the stepper shows FEAT-21.SPEC-003's error message and she remains on this screen with her attempted value visible.

**FEAT-21.SPEC-001-AC-04:** Given Riley sets the interval to 13 weeks and taps "Set up recurring," then the stepper shows FEAT-21.SPEC-003's error message and she remains on this screen.

**FEAT-21.SPEC-001-AC-05:** Given Riley taps "Skip this," then the screen closes without creating a series and she returns to her booking confirmation unchanged.

**FEAT-21.SPEC-001-AC-06:** Given Riley taps the back arrow with an interval already chosen but not submitted, then she returns to her booking confirmation and no series is created.

**FEAT-21.SPEC-001-AC-07:** Given Riley successfully creates a series, then FEAT-21.SPEC-004 is triggered to generate its first occurrence(s) within the Pro's current booking horizon.

**FEAT-21.SPEC-001-AC-08:** Given Riley taps "Set up recurring" and the operation fails due to a network error, then an error banner reads "Could not set up your recurring series. Check your connection and try again." with a Retry button, and her chosen interval is preserved.

**FEAT-21.SPEC-001-AC-09:** Given Riley loses connectivity while on this screen and attempts to submit, then she sees "You'll need to be online to set this up. Please check your connection and try again." and nothing is submitted.

**FEAT-21.SPEC-001-AC-10:** Given Riley's underlying booking is cancelled before she reaches this screen, when the screen loads, then she sees "This booking is no longer active, so it can't be made recurring." with a single option returning to FEAT-06.

**FEAT-21.SPEC-001-AC-11:** Given Riley already set up a series from this booking and navigates back to this offer, when the screen loads, then she is routed directly to FEAT-21.SPEC-002 instead of seeing the offer again.

**FEAT-21.SPEC-001-AC-12:** Given Talia wants to set up a standing appointment for a client at the chair, when she looks for that action, then it is not on this screen -- it is on FEAT-21.SPEC-010 (Pro Recurring Series Management), reached from FEAT-30's Pro booking detail.

**FEAT-21.SPEC-001-AC-13:** Given Riley reaches this offer through a stale link after the confirmation context has expired, then she sees "This offer has expired. Open your booking to set up a recurring series." with a link to FEAT-06 (Access Link Request), not a sign-in prompt.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 (empty, filling, validating, validation error, error) plus offline N/A | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |
