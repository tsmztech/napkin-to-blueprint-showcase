---
document_type: spec
spec_type: screen
spec_id: FEAT-22.SPEC-001
spec_name: Balance Payment
spec_slug: balance-payment
parent_feature: FEAT-22
parent_feature_name: In-App Balance Payment
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 18
---

# Screen Spec: Balance Payment

## Overview

**Name:** Balance Payment
**ID:** FEAT-22.SPEC-001
**Type:** Screen
**Purpose:** Riley views her deposit-paid-vs-balance-remaining running record on her own confirmed booking and optionally pays the balance in-app, seeing processing, decline, and success states.
**Parent Feature:** FEAT-22 -- In-App Balance Payment

## Scope and Non-Goals

**In Scope:**
- Displaying the deposit already paid and the balance remaining on Riley's own confirmed booking
- Collecting card details through the payment-processing capability's own entry element and submitting them for authorization and capture of the balance
- Processing, decline/retry, and success states for a single balance payment attempt and any immediate retries
- Reflecting a cancellation that arrives while Riley is on this screen, per the contention rule this feature owns

**Non-Goals:**
- Computing the balance amount -- governed exclusively by FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules); this screen only displays the value SPEC-003 locked
- Guaranteeing the charge is never duplicated, resolving the outcome across a dropped connection, and resolving a race with a Pro cancellation -- governed by FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention); this screen only reflects the outcome that spec resolves
- Displaying the rest of the booking's details (service, appointment time, studio address, cancellation policy) -- owned by FEAT-06.SPEC-004 (Booking Detail via Manage Link), from which Riley reaches this screen; this screen shows only the payment-specific running record
- Paying the balance in person, or any card-reader or in-person handling of the balance -- excluded per SC-16: the in-person path remains the unaffected default; this screen is purely the optional in-app alternative

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Riley taps "Pay Balance" on her own confirmed booking | The Booking reference (service, price_agreed, deposit_amount via the Deposit Transaction, current balance-due state) |

This is the feature's only entry point (Access field, product-features.md: "only appears against an existing confirmed booking with a balance due"). There is no standalone or directly-linked way to reach this screen -- it is always reached from Riley's own existing booking, never from a fresh booking flow.

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen, scoped to her own booking with a balance due only | Enter card details and submit a balance payment for her own booking only | -- |
| The Pro (Talia) | No | No | N/A -- this screen belongs to the client-facing "my bookings" flow and never appears on any of the Pro's own surfaces (FEAT-12, FEAT-30). The Pro's own view of the resulting paid/unpaid status lives on FEAT-12 (Pro Daily Schedule Dashboard) and FEAT-30 (Pro Booking Management), never here, per the Access Matrix's Booking & Payment row (Full for the Pro, but on those surfaces, not this one). |
| Platform Operator (Support) | No | No | N/A -- Support's view-only access to balance status is on FEAT-16 (Booking & Payment Activity Record) and FEAT-19 (Platform Support Read-Only Access), and never to card data (Access Matrix, Payouts row: "View: status and money list only"). Support never opens this in-flow payment screen. |
| Unauthenticated / no valid access link | No | No | Per FEAT-29 (Pro Sign-In & Account Lifecycle) and FEAT-06.SPEC-004's and FEAT-06's access-link scoping (XBR-18), a visitor without a valid access link for this specific booking sees "Request a new link" rather than this screen -- the same experience as any other attempt to reach a client-facing booking view without valid access. |
| Expired access link | No | No | The access link governing Riley's "my bookings" session has its own expiry (XBR-18); once it expires, she is returned to the "Request a new link" prompt exactly as an unauthenticated visitor would be. Any card details she had begun entering are discarded -- nothing is preserved across an expired link, since re-entry always starts this screen fresh. |

This product has no sign-in concept for the Client (BRIEF.md, Target Users & Roles: clients "must not face a signup wall"); access to this screen is scoped entirely to Riley's valid access link for this specific booking (FEAT-06.SPEC-004, XBR-18), not to a login.

## Layout and Content

**Header:** A back arrow (returns Riley to FEAT-06.SPEC-004, Booking Detail via Manage Link) and the screen title "Pay your balance." Below the title, a one-line booking summary: service name, the Pro's display name, and the appointment date and time in the Pro's timezone (labeled, per XBR-25).

**Body, in order, top to bottom:**
- A running-record block showing, as two lines: "Deposit paid: {deposit_amount}" (read from the Deposit Transaction) above "Balance due: {balance amount locked by FEAT-22.SPEC-003}" as the primary figure. Both figures are read directly from existing records; this screen performs no computation of its own.
- The payment-processing capability's own card-detail entry element -- the fields Riley fills (card number, expiry, security code, and postal code where the capability requires it) are entered directly into that capability's element and never pass through the product's own code, per SC-11.
- A single "Pay {balance amount} balance" button, below the card entry element, spanning the width of the body.
- A status/error region directly below the button, empty in the default state, populated in the Processing, Error, and Cancelled states described below.

**Footer:** A single line of plain text: "Do not close this page while your payment is processing." -- shown persistently, not only during the Processing state, so Riley is prepared before she taps Pay.

### Responsive Behavior

- **Compact size class (phone, the product's primary usage per BRIEF.md's Scale & Non-Functional Expectations):** Single-column layout exactly as described above, full width, footer text pinned below the status region.
- **Medium size class and above:** The body content (running-record block, card element, Pay button, status region) is capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.
- **Booking summary line:** Wraps to two lines on the narrowest supported widths rather than truncating -- the service name and appointment time are never cut off.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Screen closes | Standard back transition |
| Card entry element | Type | Captures card details directly with the payment-processing capability | Element shows entered data per the capability's own input treatment | Standard input focus/validation state from the capability's element |
| Pay button | Tap | 1. Confirm the Booking is still in a payable state and no prior Balance Payment has already succeeded. 2. Request eligibility confirmation from FEAT-22.SPEC-003. 3. If eligible, submit the charge through FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund). | Button enters the Processing state (loading indicator, disabled) | Status region shows "Processing payment, do not close this page." |
| Pay button (while processing) | Tap | No action -- ignored while a request is already in flight (double-submit prevention, governed by FEAT-22.SPEC-004) | None | Button remains in Processing state |
| "Try a different card" (shown only in the Error state) | Tap | Clears the card entry element and re-enables the Pay button | Screen returns to the default entry state with the same locked balance amount shown | Card entry element is cleared and refocused |
| "Back to my bookings" (shown only in the Cancelled state) | Tap | Navigate to FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Screen closes; the booking's now-cancelled state is reflected there | The updated booking state is visible on FEAT-06.SPEC-004, which Riley returns to |

### Accessibility Notes

- **Focus order:** Back arrow -> booking summary (read-only, not a focus stop) -> running-record block (read-only, not a focus stop) -> card entry element's own internal focus order -> Pay button -> status/error region (receives focus only when it updates).
- **Dynamic announcements:** When the status region updates (Processing begins, a decline message appears, success is reached, or the Cancelled message appears), the new text is announced to assistive technology and the region is programmatically associated as the outcome of the Pay action.
- **Success feedback:** On success, focus moves to the success message.
- **Keyboard alternatives:** Every action on this screen (Pay, Try a different card, Back to my bookings, back arrow) is reachable by keyboard; there are no pointer-only gestures. The card entry element's own keyboard accessibility is owned by the payment-processing capability.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default (entry) | Running-record block and empty card entry element shown; Pay button enabled once the element reports valid input | Screen first opens from FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Riley taps Pay |
| Processing | Pay button shows a loading indicator and is disabled; status region reads "Processing payment, do not close this page." | Riley taps Pay and the charge request is sent | FEAT-22.SPEC-005 reports a result (success or decline), or the connection drops |
| Success | Status region shows "Balance received -- your booking is now fully paid." and the running-record block updates to show a zero balance | FEAT-22.SPEC-002 (Balance Capture & Booking Status Update) reports the Booking's balance-due status is now fully paid | Riley taps Back to return to "my bookings", where the fully-paid status is also reflected |
| Error (declined) | Status region shows the plain-language decline reason (translated by FEAT-22.SPEC-005) and a "Try a different card" action; card entry element is cleared | FEAT-22.SPEC-005 reports a decline | Riley taps "Try a different card," returning to Default with the same locked balance amount and no partial or ambiguous state on the booking (FEAT-22.SPEC-004) |
| Cancelled (contention resolved against the attempt) | Status region shows "This booking was just cancelled, so there's nothing to pay. Your card was not charged." with the "Back to my bookings" action; Pay button is disabled | FEAT-22.SPEC-004's contention rule determines a Pro cancellation committed before Riley's payment attempt (reject-with-refresh) | Riley taps "Back to my bookings" |
| Offline/Degraded | If connectivity is lost before Riley taps Pay, the Pay button is disabled with the message "You're offline -- reconnect to pay your balance." If connectivity drops mid-processing, this is treated per FEAT-22.SPEC-004: the status region shows "We couldn't confirm your payment. Check your connection; if you were charged, your booking will show as fully paid the moment you reconnect -- you will never be charged twice." with a "Try again" action | Connectivity is lost while this screen is open, at any point before or during a charge attempt | Connectivity is restored: the Default state resumes if no charge was in flight; if a charge was in flight, FEAT-22.SPEC-004's resolution determines whether Success or Error is shown |

## Validation Rules

Validation of the balance amount and the preconditions for attempting a charge (Booking still in a payable state, currency match, no existing succeeded Balance Payment, active payout account, no committed cancellation) is governed by FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules) and FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention). Card-detail format validation is owned entirely by the payment-processing capability's own entry element; this screen never validates card fields itself. This screen enforces no field-level rules of its own beyond disabling Pay until the card element reports it has valid input.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-06.SPEC-004 (Booking Detail via Manage Link) | FEAT-06 (Client Booking Identity) |
| Successful payment | FEAT-06.SPEC-004 (Booking Detail via Manage Link), reached only when Riley taps Back after Success | FEAT-06 (Client Booking Identity) |
| Pro cancellation resolved against the attempt | FEAT-06.SPEC-004 (Booking Detail via Manage Link) | FEAT-06 (Client Booking Identity) |

## Data Model

**Creates:** None directly -- the Balance Payment record is created by FEAT-22.SPEC-002 on a successful charge, not by this screen.
**Reads:** Booking -- service, start_time, price_agreed, balance-due state. Deposit Transaction -- amount (for the "Deposit paid" line). Pro Account -- display_name, timezone (for the booking summary line, per XBR-25).
**Updates:** None directly -- the Booking's balance-due status update is written by FEAT-22.SPEC-002, never by this screen.
**Deletes:** None.

## Business Rules

- The balance amount shown is always the value FEAT-22.SPEC-003 computed and locked; this screen never performs its own computation and never lets Riley edit it (per the Brief's Shared UI Patterns).
- Every payment attempt, and every retry, is subject to FEAT-22.SPEC-003's eligibility preconditions and FEAT-22.SPEC-004's contention check before a charge is requested.
- FEAT-22.SPEC-004 governs the guarantee that this screen never shows an ambiguous outcome: every attempt resolves to exactly one of Success, Error, or Cancelled, even across a dropped connection or a concurrent Pro cancellation.
- Retrying after a decline never requires Riley to re-enter or re-confirm anything about the booking -- only the card entry element is reset.
- In-app balance payment is purely additive: paying in person remains the unaffected default for any client who never opens this screen (per the feature's Alternate flow and BRIEF.md's Open Questions resolution).
- XBR-07: Chairtime's fee on the balance is always zero; the amount Riley pays goes to Talia's payout account with no platform cut.

## Edge Cases

- **Riley taps Pay twice in rapid succession** -- The second tap is ignored while the first request is in flight (Pay button disabled in the Processing state, per FEAT-22.SPEC-004's double-charge prevention).
- **Riley navigates back and returns to this screen** -- The screen re-reads the current Booking and Balance Payment state; if a Balance Payment already Succeeded (a prior attempt succeeded but she navigated away before seeing it), she is shown the Success state immediately rather than a fresh entry form, per FEAT-22.SPEC-004.
- **Talia commits a cancellation while Riley is mid-attempt (concurrent-edit conflict on the Booking)** -- Per the dependency map's Contention note for Balance Payment ("a cancellation committed first blocks the payment"), FEAT-22.SPEC-004's contention check resolves this as reject-with-refresh: if the cancellation commits first, no charge is requested (or, if already sent, it is not applied) and this screen shows the Cancelled state; Riley is never charged for a cancelled booking.
- **Riley's payment is captured, and the booking is cancelled moments afterward** -- Handled entirely by FEAT-22.SPEC-004 and FEAT-30 -- the payment already succeeded is refunded in full alongside the deposit (XBR-23); this screen has already shown Success and is not reopened to show the refund, which Riley sees reflected on her "my bookings" view instead.
- **Connection drops mid-payment** -- Handled by the Offline/Degraded state above; the outcome is resolved by FEAT-22.SPEC-004 and never results in an ambiguous charge.
- **The payment-processing capability itself is slow or unavailable** -- Covered by FEAT-22.SPEC-005's Degradation Behavior for this screen; this screen shows whatever exact message that spec defines for the Slow/Down/Rejects conditions.
- **Riley reaches this screen for a booking whose balance is already fully paid (in person or a prior in-app payment)** -- Cannot occur under normal operation, since FEAT-06.SPEC-004 only shows its "Pay Balance" button while a balance is genuinely due (per that spec's own Business Rules); if reached anyway (a stale link), FEAT-22.SPEC-003's eligibility check finds no balance due and the screen shows "There's nothing left to pay on this booking." with a single "Back to my bookings" action.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Navigation (inbound) | Riley arrives here by tapping "Pay Balance" on her own confirmed booking |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Navigation (outbound) | Back arrow, and the outcome of every terminal state, return Riley here |
| FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules) | References (inbound) | Supplies the locked balance amount and the eligibility preconditions checked before every charge attempt |
| FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention) | References (inbound) | Governs double-submit prevention, connection-drop resolution, the never-ambiguous-outcome guarantee, and the cancellation-contention resolution this screen reflects |
| FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund) | Triggers (outbound) | The Pay action requests authorization and capture through this integration; its decline translation populates the Error state |
| FEAT-22.SPEC-002 (Balance Capture & Booking Status Update) | Triggered by (inbound) | This automation's successful outcome is what moves the screen into the Success state |
| FEAT-23.SPEC-002 (Tip Amount Validation) -- within FEAT-23 (Tipping at Checkout, Later) | References (inbound) | Rule spec enforced at this screen (Enforced-By): once FEAT-23 ships, the tip value is validated against FEAT-23.SPEC-002's tip-only constraints before it is handed to the Pay submission, and an invalid tip blocks the submission. This screen never validates or alters `tip`; at v1 (no tip entered) the check is a no-op |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| balance_payment_attempted | booking reference, balance amount, is_retry (true/false) | Riley taps Pay | supports success-metrics.md: "Payout Transparency" (a visible, correctly attempted balance payment is part of a Pro seeing every deduction and deposit-like flow accounted for without asking) |
| balance_payment_succeeded | booking reference, balance amount | FEAT-22.SPEC-002 confirms the Booking's balance-due status is fully paid | supports success-metrics.md: "Payout Transparency" |
| balance_payment_failed | booking reference, decline reason category | FEAT-22.SPEC-005 reports a decline | supports success-metrics.md: "Payout Transparency" |
| balance_payment_blocked_by_cancellation | booking reference | FEAT-22.SPEC-004's contention check resolves against the attempt because a cancellation committed first | N/A -- no Stage 2 metric measures this specific contention outcome; retained so the never-double-charge, never-forfeited guarantee's exercise rate is observable to the Pro's activity record (FEAT-16), not to a success metric |

## Acceptance Criteria

**FEAT-22.SPEC-001-AC-01:** Given Riley is on the Balance Payment screen for a booking with a deposit of {D} and a locked balance of {B}, when the screen loads, then she sees "Deposit paid: {D}" and "Balance due: {B}" with no way to edit either figure.

**FEAT-22.SPEC-001-AC-02:** Given Riley has entered valid card details, when she taps "Pay {B} balance," then the button enters the Processing state and the status region reads "Processing payment, do not close this page."

**FEAT-22.SPEC-001-AC-03:** Given Riley's payment is in the Processing state, when she taps the Pay button again, then nothing happens and the button remains in the Processing state.

**FEAT-22.SPEC-001-AC-04:** Given Riley's card is declined, when FEAT-22.SPEC-005 reports the decline, then the status region shows the plain-language decline reason and a "Try a different card" action.

**FEAT-22.SPEC-001-AC-05:** Given Riley sees a decline message, when she taps "Try a different card," then the card entry element clears and refocuses, and the same locked balance amount is still shown.

**FEAT-22.SPEC-001-AC-06:** Given Riley's balance charge succeeds, when FEAT-22.SPEC-002 confirms the Booking's balance-due status is fully paid, then the status region shows "Balance received -- your booking is now fully paid." and the running-record block updates to show a zero balance.

**FEAT-22.SPEC-001-AC-07:** Given Talia commits a cancellation on Riley's booking in the instant before Riley's payment attempt is applied, when FEAT-22.SPEC-004's contention check resolves, then Riley sees the Cancelled state with "This booking was just cancelled, so there's nothing to pay. Your card was not charged." and no charge is applied.

**FEAT-22.SPEC-001-AC-08:** Given Riley is on the Cancelled state, when she taps "Back to my bookings," then she is navigated to FEAT-06.SPEC-004 (Booking Detail via Manage Link) showing the booking's current cancelled state.

**FEAT-22.SPEC-001-AC-09:** Given Riley loses connectivity before tapping Pay, when she attempts to tap Pay, then the button is disabled with the message "You're offline -- reconnect to pay your balance."

**FEAT-22.SPEC-001-AC-10:** Given Riley's connection drops after she taps Pay but before a result is confirmed, when the drop is detected, then the status region shows "We couldn't confirm your payment. Check your connection; if you were charged, your booking will show as fully paid the moment you reconnect -- you will never be charged twice." with a "Try again" action.

**FEAT-22.SPEC-001-AC-11:** Given Riley reconnects after a dropped connection and her charge had in fact succeeded, when the screen re-checks the Booking state, then it shows the Success state directly rather than allowing a second charge attempt.

**FEAT-22.SPEC-001-AC-12:** Given Riley navigates away from this screen after a successful charge, when she returns to this screen, then she is shown the Success state immediately rather than a fresh entry form.

**FEAT-22.SPEC-001-AC-13:** Given a visitor reaches this screen's address without a valid access link for the booking, when the screen attempts to load, then it shows "Request a new link" rather than the payment screen.

**FEAT-22.SPEC-001-AC-14:** Given Riley's access link expires while she is viewing this screen, when she attempts to interact further, then she is returned to the "Request a new link" prompt and any card details she had begun entering are discarded.

**FEAT-22.SPEC-001-AC-15:** Given Talia (the Pro) has no path that reaches this screen, when she looks for the resulting paid/unpaid status of a client's balance, then she finds it on FEAT-12 (Pro Daily Schedule Dashboard) or FEAT-30 (Pro Booking Management), never on this screen.

**FEAT-22.SPEC-001-AC-16:** Given Platform Operator (Support) opens a Pro's account after a help request, when they look at a booking's balance status, then they see the transaction status only, via FEAT-16/FEAT-19, never this in-flow payment screen or card data.

**FEAT-22.SPEC-001-AC-17:** Given Riley's payment succeeds and the booking is cancelled by either party moments afterward, when the cancellation is committed, then the already-shown Success state on this screen is not reopened or altered, and the resulting full refund is reflected on Riley's "my bookings" view instead (FEAT-22.SPEC-004, XBR-23).

**FEAT-22.SPEC-001-AC-18:** Given Riley reaches this screen for a booking whose balance is already fully paid, when the screen loads and FEAT-22.SPEC-003's eligibility check finds no balance due, then she sees "There's nothing left to pay on this booking." with a single "Back to my bookings" action.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 6 (default, processing, success, error, cancelled, offline/degraded) | 6 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |
