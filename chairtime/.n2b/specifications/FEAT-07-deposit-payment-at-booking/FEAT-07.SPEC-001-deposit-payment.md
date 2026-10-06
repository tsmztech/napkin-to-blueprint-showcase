---
document_type: spec
spec_type: screen
spec_id: FEAT-07.SPEC-001
spec_name: Deposit Payment
spec_slug: deposit-payment
parent_feature: FEAT-07
parent_feature_name: Deposit Payment at Booking
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 19
---

# Screen Spec: Deposit Payment

## Overview

**Name:** Deposit Payment
**ID:** FEAT-07.SPEC-001
**Type:** Screen
**Purpose:** Riley enters card details for the exact, already-computed deposit and sees processing, decline/retry, and success states before handing off to the booking confirmation.
**Parent Feature:** FEAT-07 -- Deposit Payment at Booking

## Scope and Non-Goals

**In Scope:**
- Displaying the locked deposit amount and the booking it belongs to
- Collecting card details through the payment-processing capability's own entry element and submitting them for authorization and capture
- Processing, decline/retry, and success states for a single payment attempt and any immediate retries
- Returning the client to the live slot list when a slot hold expires before a declined payment is retried

**Non-Goals:**
- Computing or altering the deposit amount -- governed exclusively by FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules); this screen only displays the value SPEC-003 locked
- Guaranteeing the charge is never duplicated and the booking state is never left ambiguous -- governed by FEAT-07.SPEC-004 (Payment Outcome Consistency & Idempotency); this screen only reflects the outcome that spec resolves
- Acknowledging the cancellation and deposit policy -- excluded per FEAT-05's Brief: policy acknowledgment is FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout), the step immediately before this screen; this screen assumes acknowledgment already happened
- Displaying the full booking confirmation (studio address, balance due, cancellation cut-off, manage link, calendar option) -- owned by FEAT-05.SPEC-005 (Booking Confirmation); this screen only shows a brief success state before handing off

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Riley ticks the policy agreement checkbox and taps "Acknowledge & continue" (FEAT-05.SPEC-004's Navigation Out row to this screen) | The in-progress Booking (service, price, appointment time, the locked deposit amount computed by FEAT-07.SPEC-003, the active checkout hold) |

This screen is the single owner of card entry in the booking flow: FEAT-05.SPEC-004 keeps only the policy acknowledgment and hands off here, and does not describe or collect card details itself. On success this screen returns Riley to FEAT-05.SPEC-005 (Booking Confirmation). This is the feature's only entry point. Payment is always tied to an in-progress booking; there is no standalone or directly-linked way to reach this screen (per product-features.md's States field for this feature).

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen, scoped to her own in-progress booking only | Enter card details and submit for her own booking only | -- |
| The Pro (Talia) | No | No | N/A -- this screen belongs to the client-facing booking flow and never appears on any of the Pro's own surfaces (FEAT-12, FEAT-27, FEAT-30); if Talia opens her own public booking link to preview it, she reaches this screen only in the Client capacity, exactly as any visitor would, not as a Pro-authorized view. The Pro's own view of the resulting deposit status lives on FEAT-12 (Pro Daily Schedule Dashboard) and FEAT-30 (Pro Booking Management), never here. |
| Platform Operator (Support) | No | No | N/A -- Support's view-only access to transaction status is on FEAT-16 (Booking & Payment Activity Record) and FEAT-19 (Platform Support Read-Only Access); Support never opens this in-flow payment screen, per the Access Matrix's Payouts row ("View: status and money list only"). |
| Unauthenticated (no in-progress booking session) | No | No | The screen has nothing to render without an in-progress Booking carried from FEAT-05.SPEC-004; a direct or stale attempt to open it shows "Start a new booking" and returns to FEAT-05.SPEC-001 (Public Booking Page). |
| Expired session (checkout hold expired) | Partial -- the screen remains visible but the pay action is disabled | No | Handled explicitly as the "Hold Expired" state below: a plain message and a single action returning Riley to the live slot list, per FEAT-03's ownership of hold expiry (XBR-02). |

This product has no sign-in concept for the Client (BRIEF.md, Target Users & Roles: clients "must not face a signup wall"); access to this screen is scoped entirely to possession of the active booking session created at FEAT-05.SPEC-004, not to a login.

## Layout and Content

**Header:** A back arrow (returns Riley to FEAT-05.SPEC-004, Policy Acknowledgment & Deposit Checkout, preserving her acknowledgment) and the screen title "Pay your deposit." Below the title, a one-line booking summary: service name, the Pro's display name, and the appointment date and time in the Pro's timezone (labeled, per XBR-25).

**Body, in order, top to bottom:**
- An amount summary block: "Deposit due now: {locked deposit amount}" as the primary figure, with "Balance due at your appointment: {price_agreed minus deposit_amount}" shown beneath it in smaller text. Both figures are read directly from the Booking record locked by FEAT-07.SPEC-003; this screen performs no computation of its own.
- The payment-processing capability's own card-detail entry element -- the fields Riley fills (card number, expiry, security code, and postal code where the capability requires it) are entered directly into that capability's element and never pass through the product's own code, per SC-11.
- A single "Pay {deposit amount} deposit" button, below the card entry element, spanning the width of the body.
- A status/error region directly below the button, empty in the default state, populated in the Capability Unavailable, Processing, Error, and Hold Expired states described below.

**Footer:** A single line of plain text: "Do not close this page while your payment is processing." -- shown persistently, not only during the Processing state, so Riley is prepared before she taps Pay.

### Responsive Behavior

- **Compact size class (phone, the product's primary usage per BRIEF.md's Scale & Non-Functional Expectations):** Single-column layout exactly as described above, full width, footer text pinned below the status region.
- **Medium size class and above:** The body content (amount summary, card element, Pay button, status region) is capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.
- **Booking summary line:** Wraps to two lines on the narrowest supported widths rather than truncating -- the service name and appointment time are never cut off.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Screen closes; Riley's prior acknowledgment is preserved | Standard back transition |
| Card entry element | Type | Captures card details directly with the payment-processing capability | Element shows entered data per the capability's own input treatment | Standard input focus/validation state from the capability's element |
| Pay button | Tap | 1. Confirm the Booking is still Pending Payment and the checkout hold is still active (the pre-charge hold re-validation run by FEAT-05.SPEC-006). 2. Request eligibility confirmation from FEAT-07.SPEC-003. 3. If eligible, submit the charge through FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing). | Button enters the Processing state (loading indicator, disabled) | Status region shows "Processing payment, do not close this page." |
| Pay button (while processing) | Tap | No action -- ignored while a request is already in flight (double-submit prevention, governed by FEAT-07.SPEC-004) | None | Button remains in Processing state |
| Pay button (capability unavailable) | Tap | No action -- the button is disabled before any charge request is sent, so a tap cannot occur | None | Status region shows "We can't take payments right now. Please try again in a few minutes -- your held time is still counting down." |
| "Try a different card" (shown only in the Error state) | Tap | Clears the card entry element and re-enables the Pay button | Screen returns to the default entry state with the same locked deposit amount shown | Card entry element is cleared and refocused |
| "Back to available times" (shown only in the Hold Expired state) | Tap | Navigate to FEAT-05.SPEC-002 (Slot Selection), the live slot list | Screen closes; Booking's held slot has already been released by FEAT-03 | Plain message "Your held time expired. Pick a new time to continue." is carried into the slot list as a one-time banner |

### Accessibility Notes

- **Focus order:** Back arrow -> booking summary (read-only, not a focus stop) -> card entry element's own internal focus order -> Pay button -> status/error region (receives focus only when it updates).
- **Dynamic announcements:** When the status region updates (the capability-unavailable message appears, Processing begins, the extended-wait text is added, a decline message appears, success is reached, or the hold-expired message appears), the new text is announced to assistive technology and the region is programmatically associated as the outcome of the Pay action.
- **Success feedback:** On success, focus moves to the success message before the automatic hand-off to FEAT-05.SPEC-005 occurs.
- **Keyboard alternatives:** Every action on this screen (Pay, Try a different card, Back to available times, back arrow) is reachable by keyboard; there are no pointer-only gestures. The card entry element's own keyboard accessibility is owned by the payment-processing capability.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default (entry) | Amount summary and empty card entry element shown; Pay button enabled once the element reports valid input | Screen first opens from FEAT-05.SPEC-004 | Riley taps Pay |
| Capability Unavailable (pre-submission) | Pay button is disabled before any request is sent; status region reads "We can't take payments right now. Please try again in a few minutes -- your held time is still counting down." Card entry element remains available for Riley to fill in while she waits. | The payment-processing capability reports itself down when the screen loads or at any point before Riley taps Pay, per FEAT-07.SPEC-005's Degradation Behavior (Capability Down) | The capability becomes available again, re-enabling the Pay button once the card element reports valid input; or the checkout hold expires, entering Hold Expired |
| Processing | Pay button shows a loading indicator and is disabled; status region reads "Processing payment, do not close this page." Past a brief processing threshold without a result, the status region adds "Still working -- this is taking longer than usual." while the Pay button and card entry element remain disabled, per FEAT-07.SPEC-005's Degradation Behavior (Capability Slow). | Riley taps Pay and the charge request is sent | FEAT-07.SPEC-005 reports a result (success or decline), or the connection drops |
| Success | Status region shows a brief confirmation message ("Deposit received -- confirming your booking...") before automatic hand-off | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) reports the Booking is now Confirmed | Automatic navigation to FEAT-05.SPEC-005 (Booking Confirmation), within a couple of seconds |
| Error (declined) | Status region shows the plain-language decline reason (translated by FEAT-07.SPEC-005) and a "Try a different card" action; card entry element is cleared | FEAT-07.SPEC-005 reports a decline | Riley taps "Try a different card," returning to Default with the same locked deposit amount and the same held slot (still within its window) |
| Hold Expired | Status region shows "Your held time expired. Pick a new time to continue." with the "Back to available times" action; Pay button is disabled | The checkout hold governed by FEAT-03 (XBR-02) expires while Riley is on this screen, whether before or after a decline | Riley taps "Back to available times" |
| Offline/Degraded | If connectivity is lost before Riley taps Pay, the Pay button is disabled with the message "You're offline -- reconnect to pay your deposit." If connectivity drops mid-processing, this is treated as a payment failure per FEAT-07.SPEC-004: the status region shows "We couldn't confirm your payment. Check your connection; if you were charged, your booking will show as confirmed the moment you reconnect -- you will never be charged twice." with a "Try again" action | Connectivity is lost while this screen is open, at any point before or during a charge attempt | Connectivity is restored: the Default state resumes if no charge was in flight; if a charge was in flight, FEAT-07.SPEC-004's resolution determines whether Success or Error is shown |

## Validation Rules

Validation of the deposit amount and the preconditions for attempting a charge (Booking still Pending Payment, currency match, one-charge-per-booking, active payout account) is governed by FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules). Card-detail format validation is owned entirely by the payment-processing capability's own entry element; this screen never validates card fields itself. This screen enforces no field-level rules of its own beyond disabling Pay until the card element reports it has valid input.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | FEAT-05 (Public Booking Page & Booking Flow) |
| Successful payment (Booking reaches Confirmed) | FEAT-05.SPEC-005 (Booking Confirmation) | FEAT-05 (Public Booking Page & Booking Flow) |
| Checkout hold expires (with or without a prior decline) | FEAT-05.SPEC-002 (Slot Selection) | FEAT-05 (Public Booking Page & Booking Flow) -- hold-expiry ownership is FEAT-03's per XBR-02 |

## Data Model

**Creates:** None directly -- the Deposit Transaction record is created by FEAT-07.SPEC-002 on a successful charge, not by this screen.
**Reads:** Booking -- service, start_time, price_agreed, deposit_amount (as locked by FEAT-07.SPEC-003), state. Pro Account -- display_name, timezone (for the booking summary line, per XBR-25).
**Updates:** None directly -- the Booking's Pending Payment -> Confirmed transition is written by FEAT-07.SPEC-002, never by this screen.
**Deletes:** None.

## Business Rules

- The deposit amount shown is always the value FEAT-07.SPEC-003 computed and locked; this screen never performs its own computation and never lets Riley edit it (per the Brief's Shared UI Patterns).
- Every payment attempt, and every retry, is subject to FEAT-07.SPEC-003's eligibility preconditions before a charge is requested.
- FEAT-07.SPEC-004 governs the guarantee that this screen never shows an ambiguous outcome: every attempt resolves to exactly one of Success or Error, even across a dropped connection.
- Retrying after a decline never requires Riley to re-enter service, time, name, phone, or policy acknowledgment (per the Brief's Side-Effect Inventory) -- only the card entry element is reset.
- XBR-02: the checkout hold on Riley's selected slot is time-limited; its expiry, while she is on this screen, is owned by FEAT-03 and surfaces here only as the Hold Expired state.

## Edge Cases

- **Riley taps Pay twice in rapid succession** -- The second tap is ignored while the first request is in flight (Pay button disabled in the Processing state, per FEAT-07.SPEC-004's double-charge prevention).
- **Riley navigates back and returns to this screen** -- The screen re-reads the current Booking state; if it is already Confirmed (a prior attempt succeeded but she navigated away before the hand-off completed), she is shown the Success state immediately rather than a fresh entry form, per FEAT-07.SPEC-004.
- **Concurrent action on the underlying Booking** -- The dependency map's Contention note for Booking notes it is "High" contention, but the actors who could concurrently act on it (the Pro cancelling, an automation expiring the hold) are the only other writers while a Booking is Pending Payment. If the Pro-side or hold-expiry actor commits a transition first (for example, the checkout hold expires the instant before Riley's charge is submitted), FEAT-07.SPEC-003's eligibility check catches it and the screen shows the Hold Expired state rather than attempting a charge against a booking that is no longer eligible -- reject-with-refresh, consistent with the dependency map's Contention note for Booking.
- **Connection drops mid-payment** -- Handled by the Offline/Degraded state above; the outcome is resolved by FEAT-07.SPEC-004 and never results in an ambiguous charge.
- **Riley's card is declined and the checkout hold then expires before she retries** -- The screen transitions from Error directly to Hold Expired; the "Try a different card" action is replaced by "Back to available times."
- **The payment-processing capability itself is slow or unavailable** -- Covered by FEAT-07.SPEC-005's Degradation Behavior for this screen: a slow in-flight request extends the Processing state with "Still working -- this is taking longer than usual." (FEAT-07.SPEC-005-AC-06), and a capability that is down before submission puts the screen in the Capability Unavailable state with the Pay button disabled and "We can't take payments right now. Please try again in a few minutes -- your held time is still counting down." (FEAT-07.SPEC-005-AC-07); a Rejects outcome is the existing Error (declined) state with the specific plain-language reason FEAT-07.SPEC-005 reports.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Navigation (inbound) | Riley arrives here after ticking the policy agreement and tapping "Acknowledge & continue"; FEAT-05.SPEC-004 holds the acknowledgment, this screen owns card entry |
| FEAT-05.SPEC-005 (Booking Confirmation) | Navigation (outbound) | Successful payment (on success this screen returns to FEAT-05.SPEC-005) hands off to the full confirmation screen |
| FEAT-05.SPEC-002 (Slot Selection) | Navigation (outbound) | A checkout hold expiring on this screen returns Riley to the live slot list |
| FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules) | References (inbound) | Supplies the locked deposit amount and the eligibility preconditions checked before every charge attempt |
| FEAT-07.SPEC-004 (Payment Outcome Consistency & Idempotency) | References (inbound) | Governs double-submit prevention, connection-drop resolution, and the never-ambiguous-outcome guarantee this screen reflects |
| FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing) | Triggers (outbound) | The Pay action requests authorization and capture through this integration; its decline translation populates the Error state |
| FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Triggered by (inbound) | This automation's successful outcome is what moves the screen into the Success state |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| deposit_payment_attempted | booking reference, deposit amount, is_retry (true/false) | Riley taps Pay | supports success-metrics.md: "Deposit Capture Rate" |
| deposit_payment_succeeded | booking reference, deposit amount | FEAT-07.SPEC-002 confirms the Booking is Confirmed | supports success-metrics.md: "Deposit Capture Rate" |
| deposit_payment_failed | booking reference, decline reason category | FEAT-07.SPEC-005 reports a decline | supports success-metrics.md: "Deposit Capture Rate" |
| deposit_payment_hold_expired | booking reference, had_prior_decline (true/false) | The checkout hold expires while this screen is open | supports success-metrics.md: "Deposit Capture Rate" (a hold expiry is neither a clean success nor a clean decline until Riley acts again -- this event tracks it separately so the metric's "clear success or clear, actionable decline" target is measured against attempts that actually reached a resolution) |

## Acceptance Criteria

**FEAT-07.SPEC-001-AC-01:** Given Riley is on the Deposit Payment screen with a locked deposit amount of {X}, when the screen loads, then she sees "Deposit due now: {X}" and "Balance due at your appointment: {price minus X}" with no way to edit either figure.

**FEAT-07.SPEC-001-AC-02:** Given Riley has entered valid card details, when she taps "Pay {X} deposit," then the button enters the Processing state and the status region reads "Processing payment, do not close this page."

**FEAT-07.SPEC-001-AC-03:** Given Riley's payment is in the Processing state, when she taps the Pay button again, then nothing happens and the button remains in the Processing state.

**FEAT-07.SPEC-001-AC-04:** Given Riley's card is declined, when FEAT-07.SPEC-005 reports the decline, then the status region shows the plain-language decline reason and a "Try a different card" action, and her checkout hold remains active.

**FEAT-07.SPEC-001-AC-05:** Given Riley sees a decline message, when she taps "Try a different card," then the card entry element clears and refocuses, and the same locked deposit amount is still shown -- with no need to re-enter her name, phone, or policy acknowledgment.

**FEAT-07.SPEC-001-AC-06:** Given Riley's deposit charge succeeds, when FEAT-07.SPEC-002 confirms the Booking is Confirmed, then the status region shows "Deposit received -- confirming your booking..." and she is automatically navigated to FEAT-05.SPEC-005 (Booking Confirmation).

**FEAT-07.SPEC-001-AC-07:** Given Riley's checkout hold expires while she is viewing this screen and no decline has occurred, when the expiry is reported, then the screen enters the Hold Expired state showing "Your held time expired. Pick a new time to continue." and the Pay button is disabled.

**FEAT-07.SPEC-001-AC-08:** Given Riley sees the Hold Expired state, when she taps "Back to available times," then she is navigated to FEAT-05.SPEC-002 (Slot Selection) with a one-time banner carrying the "Your held time expired" message.

**FEAT-07.SPEC-001-AC-09:** Given Riley was declined and her checkout hold then expires before she retries, when the expiry is reported, then the screen transitions from the Error state to the Hold Expired state and the "Try a different card" action is replaced by "Back to available times."

**FEAT-07.SPEC-001-AC-10:** Given Riley loses connectivity before tapping Pay, when she attempts to tap Pay, then the button is disabled with the message "You're offline -- reconnect to pay your deposit."

**FEAT-07.SPEC-001-AC-11:** Given Riley's connection drops after she taps Pay but before a result is confirmed, when the drop is detected, then the status region shows "We couldn't confirm your payment. Check your connection; if you were charged, your booking will show as confirmed the moment you reconnect -- you will never be charged twice." with a "Try again" action.

**FEAT-07.SPEC-001-AC-12:** Given Riley reconnects after a dropped connection and her charge had in fact succeeded, when the screen re-checks the Booking state, then it shows the Success state directly rather than allowing a second charge attempt.

**FEAT-07.SPEC-001-AC-13:** Given Riley navigates back to the Policy Acknowledgment screen from this screen, when she does so, then her prior acknowledgment on FEAT-05.SPEC-004 is preserved.

**FEAT-07.SPEC-001-AC-14:** Given Riley navigates away from this screen after a successful charge but before the automatic hand-off completes, when she returns to this screen, then she is shown the Success state immediately rather than a fresh entry form.

**FEAT-07.SPEC-001-AC-15:** Given a visitor reaches this screen's address without an in-progress booking session, when the screen attempts to load, then it shows "Start a new booking" and returns to FEAT-05.SPEC-001 (Public Booking Page).

**FEAT-07.SPEC-001-AC-16:** Given Talia (the Pro) opens her own public booking link to preview it, when she reaches this screen, then she experiences it exactly as any client would, in the Client capacity, with no Pro-specific controls or data shown.

**FEAT-07.SPEC-001-AC-17:** Given Riley's slot becomes ineligible between load and her tapping Pay (for example, the checkout hold expired the instant before submission), when FEAT-07.SPEC-003's eligibility check runs, then the screen shows the Hold Expired state rather than attempting a charge.

**FEAT-07.SPEC-001-AC-18:** Given the payment-processing capability is unavailable when Riley reaches this screen or at any point before she taps Pay, when she views the screen, then the Pay button is disabled with "We can't take payments right now. Please try again in a few minutes -- your held time is still counting down." and her checkout hold continues counting down normally.

**FEAT-07.SPEC-001-AC-19:** Given Riley's charge request is in flight, when more than the product's brief processing threshold passes without a result, then the status region adds "Still working -- this is taking longer than usual." while the Pay button and card entry element remain disabled.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 7 (default, capability unavailable, processing, success, error, hold expired, offline/degraded) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
