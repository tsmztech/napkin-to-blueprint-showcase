---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-07.SPEC-004
spec_name: Payment Outcome Consistency & Idempotency
spec_slug: payment-outcome-consistency-idempotency
parent_feature: FEAT-07
parent_feature_name: Deposit Payment at Booking
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 13
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Payment Outcome Consistency & Idempotency

## Overview

**Name:** Payment Outcome Consistency & Idempotency
**ID:** FEAT-07.SPEC-004
**Type:** Logic/Rule
**Purpose:** Guarantees every payment attempt ends in exactly one of a clean success or a clean, actionable failure -- never a double charge and never an ambiguous booking state, including when the confirmation UI itself fails to load or the connection drops mid-payment.
**Parent Feature:** FEAT-07 -- Deposit Payment at Booking
**Governed Entity:** Deposit Transaction (creation idempotency) jointly with Booking (Pending Payment / Confirmed consistency) -- this spec's correctness guarantee spans both, since a payment attempt is only "resolved" once the two are consistent with each other.

## Scope and Non-Goals

**In Scope:**
- The guarantee that at most one Deposit Transaction is ever created per Booking, regardless of retries, dropped connections, or duplicate event delivery
- The guarantee that the Booking's state always matches the true outcome of the most recent charge attempt, even when the client's device never receives confirmation of that outcome
- Resolution behavior when the confirmation step (FEAT-07.SPEC-001's screen, or the hand-off to FEAT-05.SPEC-005) fails to load after a successful charge
- Resolution behavior when the connection drops between the client submitting card details and the client's device learning the outcome

**Non-Goals:**
- Computing the deposit amount or checking eligibility preconditions before a charge is requested -- owned by FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules); this spec governs correctness *after* a charge attempt has been requested, not whether it should have been requested
- Creating the Deposit Transaction and performing the Confirmed transition themselves -- owned by FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation); this spec defines the guarantee that automation's create-and-confirm step must uphold, not the step's own mechanics
- Authorizing or capturing the card charge, or translating the processor's decline reasons -- owned by FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing)
- Refund or forfeiture correctness after a deposit has been captured -- excluded per the Entity-Lifecycle Coverage Matrix: later Deposit Transaction states (Refunded, Forfeited, Disputed) are owned by FEAT-09, FEAT-11, and FEAT-30, each with their own correctness rules

## Governed Entity

**Entity:** Deposit Transaction (creation) and Booking (Pending Payment / Confirmed transition), governed jointly
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| Deposit Transaction (existence) | derived | Whether a Deposit Transaction record exists for a given Booking -- the fact this spec guarantees is single-valued (zero or one, never more) |
| Deposit Transaction.status | enum | At this feature's stage: Authorized \| Captured. This spec guarantees the status set here is the true, final outcome of the one attempt that succeeded |
| Booking.state (Pending Payment / Confirmed slice) | enum | This spec guarantees Booking.state reflects Confirmed if and only if exactly one successful capture occurred for it, and remains Pending Payment (available for retry) if and only if no capture has yet succeeded |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-07.SPEC-001 | Deposit Payment | Disables the Pay button while a request is in flight (double-submit prevention); on screen load or return, re-checks the Booking's actual state before allowing a new attempt or showing Success directly |
| FEAT-07.SPEC-002 | Deposit Capture & Booking Confirmation | Implements the atomic create-and-confirm step this spec requires: the Deposit Transaction and the Confirmed transition commit together or not at all |
| FEAT-07.SPEC-005 | Card Deposit Charge & Payout Routing | Applies this spec's idempotency key discipline when submitting a charge request, so a retried request for the same attempt is never processed twice by the payment-processing capability |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Deposit Transaction (existence, per Booking) | At most one Deposit Transaction may ever exist for a given Booking | Always | Before every charge request (FEAT-07.SPEC-003) and again at capture time (FEAT-07.SPEC-002) | N/A -- enforced structurally by the create-and-confirm step refusing to create a second record, not by a client-facing error | Yes |
| Booking.state | Must never show as Confirmed unless exactly one Deposit Transaction with status Captured exists for it, and must never remain Pending Payment once that Deposit Transaction exists | Always | Continuously -- this is an invariant the create-and-confirm step (FEAT-07.SPEC-002) maintains, not a point-in-time check | N/A -- this is a system invariant, not a validated input | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Atomic capture-and-confirm | Deposit Transaction (existence, status), Booking.state | The Deposit Transaction's creation with status Captured and the Booking's transition to Confirmed must occur as one indivisible step: a partial state where one exists without the other is never observable to any client, screen, or downstream feature | N/A -- structural guarantee, no client-facing error; if the step cannot complete, it is retried automatically until it does, per Business Rules below |
| Idempotency key discipline | Deposit Transaction (existence), the charge request itself | Every charge request FEAT-07.SPEC-005 submits to the payment-processing capability carries an identifier tied to this specific attempt on this specific Booking, so a request that is resubmitted (due to a client retry after a dropped connection, or a network-level retry) is recognized by the capability as the same attempt rather than a new charge | N/A -- structural guarantee at the integration boundary, not a client-facing error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Submit a new charge attempt for a Booking | The Client (Riley) | Only when no charge attempt is currently in flight for that Booking and no Deposit Transaction yet exists for it (per FEAT-07.SPEC-003's eligibility gate, which this spec's in-flight check extends) | If an attempt is already in flight: the Pay button remains disabled and the tap is ignored (FEAT-07.SPEC-001) -- no error message, since this is prevented before a second request can even be sent. If a Deposit Transaction already exists: no new request is sent; she is shown the Success state directly |
| Resolve an ambiguous outcome on the Client's behalf | The Client (Riley) | Never -- the resolution is always determined by the true state this spec reconciles (whether a capture actually occurred), never by a choice Riley makes | Riley is never asked "were you charged?" or given a manual "I was charged" override; the screen always reflects the system's own determination |
| View or intervene in a payment attempt's reconciliation | The Pro (Talia) | Never during the attempt itself; Full view of the resulting outcome once resolved, on her own schedule/booking-management surfaces | Talia has no visibility into an in-progress attempt; she sees only the resolved Confirmed/paid state or the fact that the Booking remains Pending Payment |
| View reconciliation outcomes for support purposes | Platform Operator (Support) | View-only, on FEAT-16/FEAT-19, after the outcome resolves | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Booking.state, on screen return after an interrupted confirmation | Derived from the true Deposit Transaction state at the moment the screen re-loads: Confirmed if a Captured Deposit Transaction exists, otherwise Pending Payment (available for retry) | Whenever FEAT-07.SPEC-001 loads or re-loads for a given Booking | No -- this is always read fresh from the true state, never cached or assumed from the client's last known screen state |

## Business Rules

- **Never double-charge:** at most one successful capture is ever recorded against a Booking; a second charge attempt for the same Booking, however triggered, either finds an existing Deposit Transaction and is refused before it reaches the payment-processing capability (FEAT-07.SPEC-003), or is submitted with the same idempotency key as an in-flight or already-resolved attempt and is recognized as the same attempt by that capability rather than processed as a new charge.
- **Never leave an ambiguous outcome:** every charge attempt resolves, from the product's point of view, to exactly one of Captured or not-Captured. There is no third "unknown" state that persists -- if the outcome is not yet known (e.g., the connection dropped before the client's device learned it), the product resolves it by checking the true state (has a Deposit Transaction actually been created?) rather than guessing or leaving the Booking in limbo.
- **The Booking's state is the source of truth, never the client's screen history:** if a charge succeeds but the confirmation step (FEAT-07.SPEC-001's screen, or the hand-off to FEAT-05.SPEC-005) fails to load, the Booking's Confirmed state is authoritative and is shown on the next page load or, per product-features.md's Communications field, via the confirmation message itself -- Riley is never left unsure whether she is booked.
- **Automatic retry on interruption before resolution, never after:** if the create-and-confirm step (FEAT-07.SPEC-002) is interrupted before it commits, it is retried automatically until it commits; once it has committed (a Deposit Transaction with status Captured exists), no further retry of the charge itself ever occurs -- only screen-level re-loads of the already-resolved state.
- **Consistent with FEAT-01's price/deposit lock (XBR-04):** this spec's reconciliation never re-derives or re-checks the deposit amount -- it only reconciles whether the one attempt at the one locked amount succeeded, matching FEAT-07.SPEC-003's amount that was fixed before the attempt began.

## Edge Cases

- **Connection drops after Riley submits card details but before her device receives a result** -- On reconnection, FEAT-07.SPEC-001 re-loads the Booking's true state rather than assuming failure: if a Captured Deposit Transaction exists, she is shown Success; if none exists and no attempt is in flight, she is shown the Default state ready for a fresh attempt; she is never shown a state that lets her submit a second charge while the first attempt's true outcome is still unknown to the product itself (the product waits for the payment-processing capability's own resolution before allowing a new attempt).
- **The confirmation screen (FEAT-05.SPEC-005) fails to load after a successful capture** -- The Booking is already Confirmed at the moment of capture (FEAT-07.SPEC-002); Riley sees the confirmation on her next page load, or via the confirmation message FEAT-08 sends, whichever she reaches first -- never a re-prompt to pay again.
- **Riley closes the browser tab immediately after tapping Pay, before any result is shown, then reopens the booking link later** -- The reopened flow re-checks the Booking's true state exactly as on a reconnect; if the earlier attempt had in fact succeeded, she is shown Confirmed; if it failed or never completed, she is shown Pending Payment ready for a fresh attempt (subject to her checkout hold still being active, per FEAT-07.SPEC-001's Hold Expired state).
- **Two devices attempt to pay the same Booking at effectively the same time (for example, Riley reloads the page on a second tab while the first tab's request is still in flight)** -- Only one request can result in a Captured Deposit Transaction; the other, whichever resolves second, finds the Deposit Transaction already exists (per FEAT-07.SPEC-003's pre-request check, or the idempotency key at the capability boundary) and is treated as a no-op, with that device's screen showing Success directly rather than a duplicate charge or an error.
- **The payment-processing capability reports success for a charge request the product had already given up retrying (a very late, delayed response)** -- The late success is still applied through the same create-and-confirm step (FEAT-07.SPEC-002) if no Deposit Transaction yet exists for the Booking; if a different attempt already succeeded in the meantime, the late report is treated as the duplicate-capture case and produces no second Deposit Transaction -- money is never lost or double-recorded either way.

## Acceptance Criteria

**FEAT-07.SPEC-004-AC-01:** Given Riley's connection drops immediately after she taps Pay, when she reconnects and the screen reloads, then it shows Success if the charge in fact succeeded, or the Default state ready for a fresh attempt if it did not -- never an ambiguous "unknown" state.

**FEAT-07.SPEC-004-AC-02:** Given a charge attempt for Riley's Booking has already succeeded, when any later request (a retry, a second tab, a delayed capability response) reaches the point of creating a Deposit Transaction, then no second Deposit Transaction is created and the Booking remains Confirmed with its original outcome.

**FEAT-07.SPEC-004-AC-03:** Given the confirmation screen fails to load right after Riley's deposit is successfully captured, when she reaches the booking again later (by reopening the link or from the confirmation message), then she sees the booking as Confirmed, never a re-prompt to pay.

**FEAT-07.SPEC-004-AC-04:** Given Riley taps Pay and a request is in flight, when she taps Pay again before a result arrives, then the second tap produces no second request -- the button remains disabled and ignores the tap.

**FEAT-07.SPEC-004-AC-05:** Given the create-and-confirm step (FEAT-07.SPEC-002) is interrupted by a processing error before it commits, when the system retries, then it retries until the step commits, and Riley is never shown a state where the Deposit Transaction exists without the Confirmed transition or vice versa.

**FEAT-07.SPEC-004-AC-06:** Given a create-and-confirm step has already committed successfully, when any further retry logic runs for that same attempt, then no further retry of the charge itself occurs -- only the already-resolved state is re-displayed.

**FEAT-07.SPEC-004-AC-07:** Given Riley closes the browser tab right after tapping Pay with no result shown, when she reopens the booking link later, then the screen shows the true current state of the Booking rather than assuming the earlier attempt failed.

**FEAT-07.SPEC-004-AC-08:** Given Riley opens the same in-progress Booking on two devices and taps Pay on both at effectively the same time, when both requests are processed, then only one results in a Captured Deposit Transaction, and the other device's screen shows Success directly rather than a duplicate charge or an error.

**FEAT-07.SPEC-004-AC-09:** Given the payment-processing capability reports a very late success for a request the product had stopped waiting on, when the report arrives and no Deposit Transaction yet exists for the Booking, then it is applied through the normal create-and-confirm step exactly as an on-time success would be.

**FEAT-07.SPEC-004-AC-10:** Given the payment-processing capability reports a very late success for a request whose Booking was already confirmed by a different successful attempt, when the report arrives, then it is treated as a duplicate and produces no second Deposit Transaction.

**FEAT-07.SPEC-004-AC-11:** Given Riley (the Client) is asked to resolve an ambiguous payment outcome, when she looks for a manual "I was charged" option, then none exists -- the screen's state is always determined by the system's own reconciliation of the true Deposit Transaction state.

**FEAT-07.SPEC-004-AC-12:** Given Talia (the Pro) views her schedule while one of her client's payment attempts is still in flight, when she looks at that booking, then she sees no in-progress-attempt detail -- only the resolved Confirmed/paid state once it settles, or Pending Payment if it has not yet succeeded.

**FEAT-07.SPEC-004-AC-13:** Given every charge request this spec governs, when it is submitted to the payment-processing capability, then it carries an identifier tied to that specific attempt so a resubmission of the same request is recognized as the same attempt rather than processed as a new charge.

**FEAT-07.SPEC-004-AC-14:** Given a Deposit Transaction already exists with a Captured status for a Booking, when the eligibility check in FEAT-07.SPEC-003 runs for any further attempt on that Booking, then it denies the request, upholding this spec's never-double-charge guarantee at the point of request as well as at the point of outcome.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |
