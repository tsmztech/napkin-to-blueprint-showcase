---
document_type: spec
spec_type: screen
spec_id: FEAT-05.SPEC-004
spec_name: Policy Acknowledgment & Deposit Checkout
spec_slug: policy-acknowledgment-deposit-checkout
parent_feature: FEAT-05
parent_feature_name: Public Booking Page & Booking Flow
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Policy Acknowledgment & Deposit Checkout

## Overview

**Name:** Policy Acknowledgment & Deposit Checkout
**ID:** FEAT-05.SPEC-004
**Type:** Screen
**Purpose:** Client sees this booking's exact deposit and cancellation terms, explicitly acknowledges them, and continues into the deposit payment step (FEAT-07.SPEC-001), where the checkout hold starts and the deposit is paid.
**Parent Feature:** FEAT-05 -- Public Booking Page & Booking Flow

## Scope and Non-Goals

**In Scope:**
- Displaying the booking summary (service, time, price) and the exact deposit amount and cancellation cut-off computed for this specific booking
- Capturing the client's explicit acknowledgment of the deposit and cancellation policy
- On "Acknowledge & continue": re-checking the acknowledged policy version and the booking page's availability, triggering the checkout hold and the Pending Payment Booking (FEAT-05.SPEC-006), and navigating the client into the payment step (FEAT-07.SPEC-001)
- Showing the outcome of that hand-off when it fails -- policy changed, slot lost, or page no longer available -- and the corresponding next step

**Non-Goals:**
- Computing the exact deposit amount and cancellation cut-off, or capturing the acknowledged policy version -- owned by FEAT-05.SPEC-009 (Policy Acknowledgment Capture & Integrity Check); this screen displays what that spec computes and calls it to record the acknowledgment
- Card entry, the Pay action, payment processing, declines, retries, and the payment success/hold-expired states -- owned by FEAT-07.SPEC-001 (Deposit Payment), which this screen navigates to; this screen never renders card fields or touches card data (scope-boundaries.md SC-11)
- Placing and expiring the checkout hold, and creating the Pending Payment Booking -- owned by FEAT-05.SPEC-006, aligned to FEAT-03.SPEC-002 (the hold starts when the client advances into the payment step); this screen only triggers it
- Rendering the on-screen confirmation itself -- handled by FEAT-05.SPEC-005 once payment succeeds, reached from FEAT-07.SPEC-001

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05.SPEC-003 (Client Details & Consent) | Client taps Continue after entering valid details | Chosen service, chosen time (not yet held), entered name/phone/opt-in/email/note |
| FEAT-07.SPEC-001 (Deposit Payment) | Client taps the back arrow on the payment screen | The in-progress Booking and its still-active checkout hold; the prior acknowledgment preserved |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen | Acknowledge the policy and continue to payment | -- |
| The Pro (Talia), preview mode | Full screen, identical rendering, including the exact deposit amount and cut-off her own settings would produce | Walk through acknowledgment and continue to the simulated payment step (FEAT-07.SPEC-001) with no real hold, Booking, or card charge (Business Rules) | -- |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view | View the policy display only -- never a real client's in-progress checkout | Acknowledgment checkbox and continue control are not offered; consistent with XBR-24 |
| Unauthenticated | Yes -- the default and intended state for the Client role | Yes, identical to the Client row above | -- |
| Expired session | N/A -- no session exists to expire; on first arrival no checkout hold exists yet, and once the client has continued, the hold's own timeout governs how long they have to complete payment (see Business Rules, Edge Cases) | N/A | N/A |

## Layout and Content

**Header:** Back arrow (returns to FEAT-05.SPEC-003, all entries preserved) with a booking summary: service name, date and time, and full price.

**Body:**
- Deposit and cancellation policy block, in plain language, stating: the exact deposit amount due now for this booking, the exact date and time the cancellation window closes for this appointment, and what happens to the deposit if the client cancels or no-shows inside versus outside that window.
- Policy acknowledgment checkbox, never pre-checked, labeled with a plain-language statement that checking it means agreeing to the terms shown above.

**Footer:** "Acknowledge & continue" action button, full width, disabled until the acknowledgment checkbox is checked. Card entry and the Pay action (labeled with the deposit amount) live on FEAT-07.SPEC-001, the next screen.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width, action button in the footer.
- **Medium size class and above:** Content remains single-column, capped at a comfortable form width and horizontally centered; no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-05.SPEC-003 (Client Details & Consent), entries preserved | Screen closes | Standard backward transition |
| Policy acknowledgment checkbox | Tap | Triggers FEAT-05.SPEC-009 to record the intent to acknowledge; toggles checkbox state | "Acknowledge & continue" enables when checked | Checkbox shows checked/unchecked state |
| Acknowledge & continue button | Tap (checkbox checked) | 1. Re-validate the acknowledged policy version still matches the current version, via FEAT-05.SPEC-009. 2. Re-check the booking page's availability gate (FEAT-05.SPEC-008). 3. If unchanged, trigger FEAT-05.SPEC-006 to request the checkout hold from FEAT-03.SPEC-002 and create the Pending Payment Booking. 4. If the hold is placed, navigate to FEAT-07.SPEC-001 (Deposit Payment). | Button shows loading state | Success: navigate to FEAT-07.SPEC-001 with the slot held. Policy changed: refused with the refreshed wording shown and re-acknowledgment required. Slot lost: plain message and return to FEAT-05.SPEC-002. Page unavailable: the gate's plain "not accepting bookings" message |
| Acknowledge & continue button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> booking summary -> policy block -> acknowledgment checkbox -> Acknowledge & continue button.
- **Announcements:** The exact deposit amount and cancellation cut-off are announced as part of the policy block on load. A policy-changed refusal or a slot-lost message is announced immediately when it appears.
- **Keyboard alternatives:** The acknowledgment checkbox and Acknowledge & continue button are reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty | N/A -- this screen never renders a collection that can be empty; it always shows a single in-progress booking's summary and policy terms | N/A | N/A |
| Loading | Neutral loading placeholder in place of the booking summary and policy block while this screen's own on-open data (booking summary, exact deposit/cut-off from FEAT-05.SPEC-009, current Cancellation Policy wording) is fetched | Screen first opens, before that data has finished loading | Data loads successfully (-> Unacknowledged) or fails (-> Error) |
| Error | Blocking message: "Couldn't load your booking summary. Try again." with a Retry action in place of the booking summary and policy block; acknowledgment checkbox and continue button are not offered until this resolves | This screen's own on-open data (booking summary, exact deposit/cut-off, current Cancellation Policy wording) fails to load | Client taps Retry and the data loads successfully, or navigates away |
| Unacknowledged | Policy block shown, checkbox unchecked, continue button disabled | Screen first opens | Client checks the acknowledgment checkbox |
| Acknowledged | Checkbox checked, continue button enabled | Client checks the acknowledgment checkbox | Client taps Acknowledge & continue, or unchecks the box |
| Starting checkout | Continue button shows a loading indicator, form disabled | Client taps Acknowledge & continue | The hold is placed (navigate to FEAT-07.SPEC-001) or the hand-off fails (policy changed, slot lost, page unavailable) |
| Policy changed | Blocking message: "The cancellation policy has changed since you agreed to it. Please review the updated terms." with the refreshed wording shown and the checkbox reset to unchecked | FEAT-05.SPEC-009 detects the acknowledged version no longer matches the current version when the client continues | Client re-acknowledges the current wording |
| Slot lost | Plain message: "That time was just taken." or "That time is no longer available." and return to FEAT-05.SPEC-002 | FEAT-05.SPEC-006 reports the hold could not be placed (contested or no longer valid) | Client picks a new time on FEAT-05.SPEC-002 |
| Offline/Degraded | Banner "Check your connection and try again." replaces the continue control; the policy block remains visible if already loaded; continue is disabled | Connectivity is lost while loading or before the hand-off completes | Connectivity is restored and the pending action (load or continue) completes |

## Validation Rules

Validation governed by FEAT-05.SPEC-009 (Policy Acknowledgment Capture & Integrity Check) for the acknowledgment step. This screen requires the acknowledgment checkbox to be checked before Acknowledge & continue enables; card-level validation is owned entirely by the payment-processing capability behind FEAT-07.SPEC-001.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | FEAT-05.SPEC-003 (Client Details & Consent) | -- |
| Acknowledge & continue (hold placed) | FEAT-07.SPEC-001 (Deposit Payment) | FEAT-07 (Deposit Payment at Booking) |
| Slot lost when continuing | FEAT-05.SPEC-002 (Slot Selection), refreshed list | -- |
| Service archived when continuing | FEAT-05.SPEC-001 (Public Booking Page), refreshed service list | -- |

Successful payment navigates onward to FEAT-05.SPEC-005 (Booking Confirmation) from FEAT-07.SPEC-001, not from this screen.

## Data Model

**Creates:** None directly -- the Pending Payment Booking and its checkout hold are created by FEAT-05.SPEC-006 when the client continues, and the Deposit Transaction is created by FEAT-07 on payment.
**Reads:** Service -- price, deposit_rule (for the exact deposit computation, performed by FEAT-05.SPEC-009). Cancellation Policy -- current plain_language_wording, window_hours, version (for display and re-validation). The in-progress booking details (service, chosen time), to display its terms.
**Updates:** The recorded policy_version, acknowledged wording, and acknowledgment timestamp, written by FEAT-05.SPEC-009 when the client checks the acknowledgment box (carried onto the Booking when FEAT-05.SPEC-006 creates it).
**Deletes:** None.

## Business Rules

- The deposit is computed once, exactly, from the service's rule in the Pro's account currency, cannot be altered by the client, and is charged once per booking (XBR-05); the amount shown here is the same amount charged on FEAT-07.SPEC-001.
- No deposit can be taken, and this screen is never reached, unless the Pro's payout account is active (XBR-06) -- enforced by FEAT-05.SPEC-008, on load and again when the client taps Acknowledge & continue.
- Every booking is governed by the cancellation policy version shown and acknowledged at booking; policy edits never change existing bookings once confirmed (XBR-08).
- If the current policy version changes between the client's acknowledgment and the Acknowledge & continue tap, the client is refused and must review and re-acknowledge the refreshed wording (FEAT-05.SPEC-009's integrity check, dependency map's Cancellation Policy contention note).
- If the service is archived before the client continues, the client is refused with a refresh back to the service list (dependency map's Service contention note).
- The checkout hold starts only when the client advances into the payment step by tapping Acknowledge & continue (FEAT-03.SPEC-002, XBR-02); earlier steps hold nothing. Once placed, a declined payment or a return to this screen via the payment screen's back arrow does not lose the hold within its window, and the acknowledgment persists (Shared UI Pattern persistence).
- In preview mode, no real hold, Booking, or card charge is ever created; the Pro continues to a simulated payment step.
- Confirming consent at booking submission triggers FEAT-14.SPEC-003 (Consent Capture at Booking), which records the consent decision under FEAT-14's rules (XBR-15).

## Edge Cases

- **The client returns here from the payment screen's back arrow** -- The acknowledgment and entries are preserved and the existing hold remains active; tapping Acknowledge & continue again reuses the existing Pending Payment Booking and hold rather than placing a second one (FEAT-05.SPEC-006).
- **The acknowledged policy version changes between acknowledgment and the continue tap** -- The hand-off is refused with the "Policy changed" state; the client must review and re-acknowledge the current wording before continuing (FEAT-05.SPEC-009). No hold is placed.
- **The chosen slot is taken or no longer valid by the time the client continues** -- FEAT-05.SPEC-006's hold request is refused before any Booking is created; the client sees the "Slot lost" state and returns to FEAT-05.SPEC-002, and is never sent to the payment step for a slot they do not hold.
- **The chosen service is archived by the Pro before the client continues** -- The hand-off is refused with a refresh back to FEAT-05.SPEC-001's service list; nothing is held.
- **A pause or payout change occurs while the client is on this screen** -- The availability gate (FEAT-05.SPEC-008) re-check on continue shows the plain "not accepting bookings" message; no hold is placed.
- **Client loses connectivity while continuing** -- The Offline/Degraded state renders; nothing is held or charged until the action completes on a live connection, and the client is never shown an ambiguous state.
- **Client taps Acknowledge & continue twice rapidly** -- The second tap is ignored while the first hand-off is in progress.
- **Client unchecks the acknowledgment box after checking it** -- The continue button disables again immediately; no data is lost, and re-checking re-enables it.
- **This screen's own on-open data fails to load** -- The booking summary, the exact deposit/cut-off computed by FEAT-05.SPEC-009, or the current Cancellation Policy wording cannot be fetched; the Error state renders with "Couldn't load your booking summary. Try again." and a Retry action; the acknowledgment checkbox and continue button are withheld until the data loads successfully, so the client is never asked to acknowledge or advance against stale or missing terms.

## Connected Specs

| Connected Spec | Connection Type | Description |
|-----------------|-------------------|--------------|
| FEAT-05.SPEC-003 (Client Details & Consent) | Navigation (inbound) | Client arrives here after entering valid details |
| FEAT-07.SPEC-001 (Deposit Payment) -- within FEAT-07 (Deposit Payment at Booking) | Navigation (outbound) / Navigation (inbound) | Acknowledge & continue navigates here once the hold is placed; this screen owns card entry and payment; its back arrow returns here |
| FEAT-05.SPEC-005 (Booking Confirmation) | References (outbound) | Reached from FEAT-07.SPEC-001 after successful payment, not directly from this screen |
| FEAT-05.SPEC-002 (Slot Selection) | Navigation (outbound) | A lost slot returns the client here with a refreshed list |
| FEAT-05.SPEC-006 (Slot Hold & Re-Validation at Checkout) | Triggers (outbound) | Acknowledge & continue triggers the checkout hold and Pending Payment Booking |
| FEAT-05.SPEC-009 (Policy Acknowledgment Capture & Integrity Check) | References (outbound) | Computes the exact deposit/cut-off shown, and records and re-validates the acknowledgment |
| FEAT-05.SPEC-008 (Booking Page Availability Gate) | References (inbound) | Governs whether this screen renders, on load and again on Acknowledge & continue |
| FEAT-14.SPEC-003 (Consent Capture at Booking) -- within FEAT-14 (Messaging Consent Management) | Triggers (outbound) | Booking submission into checkout triggers consent capture for the client's opt-in/opt-out choice |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| policy_acknowledged | -- | Client checks the acknowledgment checkbox | supports success-metrics.md: "Policy Clarity at Booking" |
| checkout_started | deposit amount, service ID | Client taps Acknowledge & continue and the hold is placed | supports success-metrics.md: "Deposit Capture Rate" |
| policy_version_mismatch_at_continue | -- | FEAT-05.SPEC-009 detects a version change between acknowledgment and the continue tap | supports success-metrics.md: "Policy Clarity at Booking" |

## Acceptance Criteria

**FEAT-05.SPEC-004-AC-01:** Given Riley arrives on this screen after entering valid details, when the screen loads, then Riley sees the booking summary and the exact deposit amount and cancellation cut-off time computed for this specific booking.

**FEAT-05.SPEC-004-AC-02:** Given Riley is on this screen, when Riley looks at the acknowledgment checkbox on first load, then it is unchecked and the Acknowledge & continue button is disabled.

**FEAT-05.SPEC-004-AC-03:** Given Riley checks the acknowledgment checkbox, when the check registers, then the Acknowledge & continue button enables and FEAT-05.SPEC-009 records the acknowledged policy version, wording, and timestamp.

**FEAT-05.SPEC-004-AC-04:** Given Riley has acknowledged the policy and the hand-off checks pass, when Riley taps Acknowledge & continue, then the checkout hold is placed (via FEAT-05.SPEC-006) and Riley navigates to FEAT-07.SPEC-001 (Deposit Payment), where card entry and payment happen.

**FEAT-05.SPEC-004-AC-05:** Given Riley is on this screen, when Riley looks for card fields or a Pay button, then none are offered here; card entry is owned by FEAT-07.SPEC-001.

**FEAT-05.SPEC-004-AC-06:** Given the cancellation policy version changes after Riley acknowledged it but before she taps Acknowledge & continue, when she taps it, then the hand-off is refused, no hold is placed, she sees "The cancellation policy has changed since you agreed to it. Please review the updated terms." with the refreshed wording, and must re-acknowledge before continuing.

**FEAT-05.SPEC-004-AC-07:** Given another client holds Riley's chosen time first, when Riley taps Acknowledge & continue, then no Booking is created, Riley sees "That time was just taken." and is returned to FEAT-05.SPEC-002 with a refreshed list, never sent to the payment step.

**FEAT-05.SPEC-004-AC-08:** Given Riley's chosen time no longer passes the live slot rules when she taps Acknowledge & continue, when the hold request is refused, then she sees "That time is no longer available." and is returned to FEAT-05.SPEC-002 with a refreshed list.

**FEAT-05.SPEC-004-AC-09:** Given the chosen service is archived by the Pro before Riley continues, when Riley taps Acknowledge & continue, then Riley is refused with a refresh back to FEAT-05.SPEC-001's service list and nothing is held.

**FEAT-05.SPEC-004-AC-10:** Given Riley taps Acknowledge & continue twice rapidly, when the first hand-off is already in progress, then the second tap has no additional effect.

**FEAT-05.SPEC-004-AC-11:** Given Riley loses connectivity while continuing, when the Offline/Degraded state renders, then Riley sees "Check your connection and try again." and no hold is placed until the action completes.

**FEAT-05.SPEC-004-AC-12:** Given Riley returns to this screen from FEAT-07.SPEC-001's back arrow while her hold is still active, when she taps Acknowledge & continue again, then the existing hold and Pending Payment Booking are reused and no second hold is placed.

**FEAT-05.SPEC-004-AC-13:** Given Talia previews her own booking page and reaches this screen, when she acknowledges the policy and continues, then she reaches a simulated payment step with no real hold, Booking, or card charge created.

**FEAT-05.SPEC-004-AC-14:** Given Riley unchecks the acknowledgment box after checking it, when the uncheck registers, then the Acknowledge & continue button disables again without losing any other entered data.

**FEAT-05.SPEC-004-AC-15:** Given this screen's own on-open data (booking summary, exact deposit/cut-off, or current Cancellation Policy wording) fails to load, when the screen attempts to render, then Riley sees "Couldn't load your booking summary. Try again." with a Retry action, and neither the acknowledgment checkbox nor the continue button is offered until the data loads successfully.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 9 (empty, loading, error, unacknowledged, acknowledged, starting checkout, policy changed, slot lost, offline) | 9 |
| Business Rules | 8 | 8 |
| Edge Cases | 9 | 9 |
