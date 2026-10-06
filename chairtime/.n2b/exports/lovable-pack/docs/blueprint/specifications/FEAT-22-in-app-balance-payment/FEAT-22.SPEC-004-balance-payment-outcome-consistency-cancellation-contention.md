---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-22.SPEC-004
spec_name: Balance Payment Outcome Consistency & Cancellation Contention
spec_slug: balance-payment-outcome-consistency-cancellation-contention
parent_feature: FEAT-22
parent_feature_name: In-App Balance Payment
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 18
acceptance_criteria_count: 16
---

# Logic/Rule Spec: Balance Payment Outcome Consistency & Cancellation Contention

## Overview

**Name:** Balance Payment Outcome Consistency & Cancellation Contention
**ID:** FEAT-22.SPEC-004
**Type:** Logic/Rule
**Purpose:** Guarantees every balance payment attempt ends in exactly one clean outcome -- never a double charge and never a partial or ambiguous "balance due" state -- and resolves the case where a Pro cancellation and a client's balance payment race each other, per XBR-23.
**Parent Feature:** FEAT-22 -- In-App Balance Payment
**Governed Entity:** Balance Payment (creation idempotency) jointly Booking (the balance-payable / cancelled contention slice) -- this spec's correctness guarantee spans both, since a payment attempt is only "resolved" once the two are consistent with each other and with any concurrent cancellation.

## Scope and Non-Goals

**In Scope:**
- The guarantee that at most one Succeeded Balance Payment is ever created per Booking, regardless of retries, dropped connections, or duplicate event delivery
- The guarantee that the Booking's balance-due status always matches the true outcome of the most recent charge attempt, even when Riley's device never receives confirmation of that outcome
- The contention rule between a Pro-initiated cancellation and a client-initiated balance payment on the same Booking (dependency map, Balance Payment entity, Contention): reject-with-refresh, where the first committed transition wins
- Resolution behavior when a captured balance charge cannot be applied because a cancellation committed first, and when an already-succeeded balance payment is followed by a cancellation

**Non-Goals:**
- Computing the balance amount or checking the eligibility preconditions before a charge is requested -- owned by FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules); this spec governs correctness *after* a charge attempt has been requested, not whether it should have been requested
- Creating the Balance Payment and updating the Booking's balance-due status themselves -- owned by FEAT-22.SPEC-002 (Balance Capture & Booking Status Update); this spec defines the guarantee that automation's create-and-update step must uphold, not the step's own mechanics
- Authorizing or capturing the card charge, or executing the refund call once a refund is due -- owned by FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund)
- The Pro's own decision to cancel, and the cancellation commit mechanics themselves -- owned by FEAT-30 (Pro Booking Management, for a Pro-initiated cancellation) and FEAT-09 (Cancellation & No-Show Policy Engine, for a client-initiated or automatic cancellation); this spec only defines how a balance payment attempt behaves when one of those cancellations wins the race

## Governed Entity

**Entity:** Balance Payment (creation) and Booking (balance-payable / cancelled transition), governed jointly
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| Balance Payment (existence, per Booking) | derived | Whether a Succeeded Balance Payment exists for a given Booking -- the fact this spec guarantees is single-valued (zero or one, never more) |
| Balance Payment.state | enum | At this feature's stage: Attempted \| Succeeded \| Failed. This spec guarantees the state set here is the true, final outcome of the one attempt that succeeded -- or that no attempt succeeded if a cancellation won the race. Refunded is set later by FEAT-30, per the Entity-Lifecycle Coverage Matrix |
| Booking.state (balance-payable / cancelled slice) | enum | This spec guarantees the Booking's balance-due status reflects fully paid if and only if exactly one Balance Payment succeeded for it, and that a cancellation which commits before a balance charge is applied is never overwritten by a late-arriving charge outcome |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-22.SPEC-001 | Balance Payment | Disables the Pay button while a request is in flight (double-submit prevention); on screen load or return, re-checks the Booking's true state before allowing a new attempt, showing Success directly, or showing Cancelled |
| FEAT-22.SPEC-002 | Balance Capture & Booking Status Update | Implements the atomic create-and-update step this spec requires: the Balance Payment and the fully-paid status commit together or not at all, and never commit against a Booking a cancellation has already resolved |
| FEAT-22.SPEC-005 | Balance Charge, Payout Routing & Refund | Applies this spec's idempotency key discipline when submitting a charge request; executes the refund this spec's contention resolution requires when a payment already succeeded and a cancellation follows |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Balance Payment (existence, per Booking) | At most one Succeeded Balance Payment may ever exist for a given Booking | Always | Before every charge request (FEAT-22.SPEC-003) and again at capture time (FEAT-22.SPEC-002) | N/A -- enforced structurally by the create-and-update step refusing to create a second record, not by a client-facing error | Yes |
| Booking.state vs. Balance Payment (existence) | A Booking must never show fully paid unless exactly one Balance Payment with state Succeeded exists for it, and a Balance Payment must never be created against a Booking whose cancellation already committed | Always | Continuously -- this is an invariant the create-and-update step (FEAT-22.SPEC-002) maintains, not a point-in-time check | N/A -- this is a system invariant, not a validated input | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Atomic capture-and-update | Balance Payment (existence, state), Booking.state | The Balance Payment's creation with state Succeeded and the Booking's balance-due status update to fully paid must occur as one indivisible step: a partial state where one exists without the other is never observable to any client, screen, or downstream feature | N/A -- structural guarantee; if the step cannot complete, it is retried automatically until it does, per Business Rules below |
| Idempotency key discipline | Balance Payment (existence), the charge request itself | Every charge request FEAT-22.SPEC-005 submits to the payment-processing capability carries an identifier tied to this specific attempt on this specific Booking, so a request that is resubmitted (due to a client retry after a dropped connection, or a network-level retry) is recognized by the capability as the same attempt rather than a new charge | N/A -- structural guarantee at the integration boundary, not a client-facing error |
| Cancellation-vs-payment race (dependency map, Balance Payment Contention) | Booking.state (cancellation transition), Balance Payment (creation) | Whichever transition -- the Pro's (or client's, or automatic) cancellation, or the client's balance charge -- commits first is authoritative. If the cancellation commits first, the payment attempt is refused before or instead of being applied and no Balance Payment is created for it. If the payment attempt commits first (a Succeeded Balance Payment already exists), a cancellation that follows refunds it in full rather than blocking on it or being refused itself | Payment side: "This booking was just cancelled, so there's nothing to pay. Your card was not charged." Cancellation side: no error -- the cancellation always proceeds and the already-succeeded balance payment is refunded alongside the deposit (XBR-23), per FEAT-30's and FEAT-09's own cancellation commit logic |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Submit a new balance charge attempt for a Booking | The Client (Riley) | Only when no charge attempt is currently in flight for that Booking, no Succeeded Balance Payment yet exists for it, and no cancellation has committed against the Booking (per FEAT-22.SPEC-003's eligibility gate, which this spec's in-flight and contention checks extend) | If an attempt is already in flight: the Pay button remains disabled and the tap is ignored (FEAT-22.SPEC-001) -- no error message, since this is prevented before a second request can even be sent. If a Succeeded Balance Payment already exists: no new request is sent; she is shown the Success state directly. If a cancellation has committed: no charge is requested (or, if one was already sent and not yet applied, it is not applied); she sees "This booking was just cancelled, so there's nothing to pay. Your card was not charged." |
| Resolve an ambiguous outcome on the Client's behalf | The Client (Riley) | Never -- the resolution is always determined by the true state this spec reconciles (whether a capture actually occurred, and whether a cancellation won the race), never by a choice Riley makes | Riley is never asked "were you charged?" or given a manual "I was charged" override; the screen always reflects the system's own determination |
| Commit a cancellation while a balance payment attempt is in flight | The Pro (Talia, via FEAT-30) or the Client (via FEAT-10) or the product automatically (via FEAT-09) | Always -- a cancellation is never blocked by an in-flight or already-succeeded balance payment attempt; it proceeds and the payment attempt (or its already-succeeded outcome) is resolved against it, never the reverse | -- (the cancelling party's own action always proceeds; the resolution is visible to Riley on FEAT-22.SPEC-001 as the Cancelled state or, for an already-succeeded payment, as a refund reflected on her "my bookings" view) |
| View or intervene in a payment attempt's reconciliation | The Pro (Talia) | Never during the attempt itself; Full view of the resulting outcome once resolved, on her own schedule/booking-management surfaces | Talia has no visibility into an in-progress attempt; she sees only the resolved fully-paid/balance-due state, or the fact that her own cancellation won a race |
| View reconciliation outcomes for support purposes | Platform Operator (Support) | View-only, on FEAT-16/FEAT-19, after the outcome resolves | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Booking's balance-due status, on screen return after an interrupted confirmation | Derived from the true Balance Payment state at the moment the screen re-loads: fully paid if a Succeeded Balance Payment exists, Cancelled if the Booking's cancellation committed and no charge succeeded, otherwise unchanged (available for retry) | Whenever FEAT-22.SPEC-001 loads or re-loads for a given Booking | No -- this is always read fresh from the true state, never cached or assumed from the client's last known screen state |

## Business Rules

- **Never double-charge:** at most one successful capture is ever recorded against a Booking; a second charge attempt for the same Booking, however triggered, either finds an existing Succeeded Balance Payment and is refused before it reaches the payment-processing capability (FEAT-22.SPEC-003), or is submitted with the same idempotency key as an in-flight or already-resolved attempt and is recognized as the same attempt by that capability rather than processed as a new charge.
- **Never leave an ambiguous outcome:** every charge attempt resolves, from the product's point of view, to exactly one of Succeeded, not-Succeeded, or Cancelled-before-resolution. There is no third "unknown" state that persists -- if the outcome is not yet known (e.g., the connection dropped before Riley's device learned it), the product resolves it by checking the true state (has a Balance Payment actually been created? has a cancellation committed?) rather than guessing or leaving the Booking in limbo.
- **Cancellation always wins a genuine race, never a charge:** per the dependency map's Contention note for Balance Payment, a cancellation committed first blocks the payment; a payment committed first is refunded in full by a later cancellation (XBR-23). The product never resolves a race by letting a charge proceed against an already-cancelled booking, and never resolves it by blocking a cancellation because a payment succeeded first.
- **The Booking's true state is the source of truth, never the client's screen history:** if a charge succeeds but the confirmation fails to load on FEAT-22.SPEC-001, the fully-paid status is authoritative and is shown on the next page load -- Riley is never left unsure whether her balance is paid.
- **Automatic retry on interruption before resolution, never after:** if the create-and-update step (FEAT-22.SPEC-002) is interrupted before it commits, it is retried automatically until it commits; once it has committed (a Balance Payment with state Succeeded exists), no further retry of the charge itself ever occurs -- only screen-level re-loads of the already-resolved state.
- **Consistent with FEAT-22.SPEC-003's amount lock:** this spec's reconciliation never re-derives or re-checks the balance amount -- it only reconciles whether the one attempt at the one locked amount succeeded, and whether a cancellation intervened.
- **XBR-23 governs the money, never this spec:** once a cancellation-after-success outcome is determined, the actual refund execution is FEAT-22.SPEC-005's integration call, triggered by FEAT-30 or FEAT-09's own cancellation commit logic -- this spec establishes only that the refund is owed, never how it is executed.

## Edge Cases

- **Connection drops after Riley submits card details but before her device receives a result** -- On reconnection, FEAT-22.SPEC-001 re-loads the Booking's true state rather than assuming failure: if a Succeeded Balance Payment exists, she is shown Success; if the Booking was cancelled in the meantime, she is shown Cancelled; if neither and no attempt is in flight, she is shown the Default state ready for a fresh attempt.
- **Talia commits a cancellation the instant before Riley's in-flight charge is reported captured** -- The cancellation's commit is checked as part of the create-and-update step (FEAT-22.SPEC-002): if it committed first, the charge is not applied as a Balance Payment even though the card authorization succeeded, and the captured amount is refunded immediately through FEAT-22.SPEC-005 rather than recorded as a kept balance payment -- Riley is never charged for a cancelled booking.
- **Riley's balance payment succeeds, and Talia cancels the booking moments later** -- The cancellation proceeds normally (it is never blocked by an already-succeeded balance payment); FEAT-30's cancellation commit logic finds the Succeeded Balance Payment and triggers its full refund through FEAT-22.SPEC-005 alongside the deposit, per XBR-23.
- **Riley closes the browser tab immediately after tapping Pay, before any result is shown, then reopens her "my bookings" link later** -- The reopened flow re-checks the Booking's and Balance Payment's true state exactly as on a reconnect; if the earlier attempt had in fact succeeded, she is shown fully paid; if it failed or was superseded by a cancellation, she is shown accordingly.
- **Two devices attempt to pay the same Booking's balance at effectively the same time** -- Only one request can result in a Succeeded Balance Payment; the other, whichever resolves second, finds the Balance Payment already exists (per FEAT-22.SPEC-003's pre-request check, or the idempotency key at the capability boundary) and is treated as a no-op, with that device's screen showing Success directly rather than a duplicate charge or an error.
- **The payment-processing capability reports success for a charge request the product had already given up retrying (a very late, delayed response), and the Booking was cancelled in the meantime** -- The late success is checked against the Booking's current state before being applied: since the cancellation already committed, the late capture is not recorded as a Balance Payment and the captured amount is refunded immediately -- money is never lost or silently kept for a cancelled booking either way.

## Acceptance Criteria

**FEAT-22.SPEC-004-AC-01:** Given Riley's connection drops immediately after she taps Pay, when she reconnects and the screen reloads, then it shows Success if the charge in fact succeeded, Cancelled if the booking was cancelled in the meantime, or the Default state ready for a fresh attempt if neither -- never an ambiguous "unknown" state.

**FEAT-22.SPEC-004-AC-02:** Given a balance charge attempt for Riley's Booking has already succeeded, when any later request (a retry, a second device, a delayed capability response) reaches the point of creating a Balance Payment, then no second Balance Payment is created and the Booking remains fully paid with its original outcome.

**FEAT-22.SPEC-004-AC-03:** Given Talia commits a cancellation on Riley's booking the instant before Riley's in-flight charge is reported captured, when FEAT-22.SPEC-002's create-and-update step checks the Booking's state, then no Balance Payment is created and the captured amount is refunded immediately rather than kept.

**FEAT-22.SPEC-004-AC-04:** Given Riley's balance payment has already succeeded, when Talia later cancels the booking, then the cancellation proceeds normally and the Succeeded Balance Payment is refunded in full alongside the deposit, per XBR-23.

**FEAT-22.SPEC-004-AC-05:** Given Riley taps Pay and a request is in flight, when she taps Pay again before a result arrives, then the second tap produces no second request -- the button remains disabled and ignores the tap.

**FEAT-22.SPEC-004-AC-06:** Given the create-and-update step (FEAT-22.SPEC-002) is interrupted by a processing error before it commits, when the system retries, then it retries until the step commits, and Riley is never shown a state where the Balance Payment exists without the fully-paid status or vice versa.

**FEAT-22.SPEC-004-AC-07:** Given a create-and-update step has already committed successfully, when any further retry logic runs for that same attempt, then no further retry of the charge itself occurs -- only the already-resolved state is re-displayed.

**FEAT-22.SPEC-004-AC-08:** Given Riley closes the browser tab right after tapping Pay with no result shown, when she reopens her "my bookings" link later, then the screen shows the true current state of the Booking rather than assuming the earlier attempt failed.

**FEAT-22.SPEC-004-AC-09:** Given Riley opens the same booking's Balance Payment screen on two devices and taps Pay on both at effectively the same time, when both requests are processed, then only one results in a Succeeded Balance Payment, and the other device's screen shows Success directly rather than a duplicate charge or an error.

**FEAT-22.SPEC-004-AC-10:** Given the payment-processing capability reports a very late success for a request whose Booking was cancelled in the meantime, when the report arrives, then it is not recorded as a Balance Payment and the captured amount is refunded immediately.

**FEAT-22.SPEC-004-AC-11:** Given Riley (the Client) is asked to resolve an ambiguous payment outcome, when she looks for a manual "I was charged" option, then none exists -- the screen's state is always determined by the system's own reconciliation of the true Balance Payment and Booking state.

**FEAT-22.SPEC-004-AC-12:** Given Talia (the Pro) views her schedule while one of her client's balance payment attempts is still in flight, when she looks at that booking, then she sees no in-progress-attempt detail -- only the resolved fully-paid/balance-due state once it settles.

**FEAT-22.SPEC-004-AC-13:** Given every balance charge request this spec governs, when it is submitted to the payment-processing capability, then it carries an identifier tied to that specific attempt so a resubmission of the same request is recognized as the same attempt rather than processed as a new charge.

**FEAT-22.SPEC-004-AC-14:** Given a Balance Payment already exists with state Succeeded for a Booking, when the eligibility check in FEAT-22.SPEC-003 runs for any further attempt on that Booking, then it denies the request, upholding this spec's never-double-charge guarantee at the point of request as well as at the point of outcome.

**FEAT-22.SPEC-004-AC-15:** Given a Pro cancellation, a client cancellation, or an automatic cancellation commits against Riley's Booking while her balance payment attempt is unresolved, when the cancellation is checked against the payment attempt, then the cancellation is never blocked and Riley's attempt is resolved against it, never the reverse.

**FEAT-22.SPEC-004-AC-16:** Given Riley's charge is captured by the payment-processing capability but the cancellation on her Booking committed a moment earlier, when the outcome is resolved, then she sees the Cancelled state with "This booking was just cancelled, so there's nothing to pay. Your card was not charged." and the captured amount is refunded rather than recorded.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 7 | 7 |
| Edge Cases | 6 | 6 |
