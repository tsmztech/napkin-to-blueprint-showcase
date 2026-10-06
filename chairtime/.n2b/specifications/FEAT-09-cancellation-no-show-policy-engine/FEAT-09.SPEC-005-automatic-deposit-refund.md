---
document_type: spec
spec_type: integration
spec_id: FEAT-09.SPEC-005
spec_name: Automatic Deposit Refund
spec_slug: automatic-deposit-refund
parent_feature: FEAT-09
parent_feature_name: Cancellation & No-Show Policy Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 16
---

# Integration Spec: Automatic Deposit Refund

## Overview

**Name:** Automatic Deposit Refund
**ID:** FEAT-09.SPEC-005
**Type:** Integration
**Purpose:** The product requests a full deposit refund from the payment-processing capability whenever FEAT-09.SPEC-004 determines one is due, drawing on the Pro's connected payout account, and reflects the outcome to both parties without either having to chase it.
**Parent Feature:** FEAT-09 -- Cancellation & No-Show Policy Engine

## Scope and Non-Goals

**In Scope:**
- Requesting a full refund for a specific Deposit Transaction once FEAT-09.SPEC-004 determines a refund is due
- Receiving and applying the refund outcome (succeeded, cannot complete immediately) to the Deposit Transaction
- User-facing behavior when the payment-processing capability is slow, unavailable, or rejects the refund request
- Disclosure of what data this refund request shares with the capability
- Handing a cancellation on a Booking with a Succeeded Balance Payment to FEAT-22.SPEC-005 so the paid balance is refunded in full alongside the deposit (XBR-23); this spec is the trigger and reflects the outcome, and does not call the payment-processing capability for the balance itself

**Non-Goals:**
- Determining that a refund is due in the first place -- owned by FEAT-09.SPEC-004 (Cancellation & No-Show Outcome Evaluation) jointly with FEAT-09.SPEC-003 (Deposit Outcome Rules); this spec only executes a refund already determined
- Retrying a refund that could not complete immediately, or keeping the Pro's dashboard flag and the client's "in progress" status consistent until it resolves -- owned by FEAT-09.SPEC-006 (Refund Idempotency & Retry Rule); this spec defines the single request/response contract that spec's retry loop calls
- Choosing the payment-processing vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate
- Capturing the original deposit charge, or routing it to the Pro's payout account in the first place -- owned by FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing); this spec only reverses an already-captured amount
- Executing the Balance Payment refund call, its idempotency key, its retry loop, and its own Refunded / refund-in-progress states -- owned by FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund); this spec only invokes it and never refunds, forfeits, or partially refunds a balance itself
- Goodwill refunds initiated by the Pro outside this feature's automatic rules -- owned by FEAT-30 (Pro Booking Management), which executes its own refund requests against the same capability for that distinct, manually-triggered case

## Capability Category

**Category:** Payment processing
**Dependency Source:** ASMP-31 -- "Payment-processing capability... required to take client deposits, verify each pro's identity and bank details for a connected payout account, pay deposits out to the pro, issue refunds, notify the product of card-issuer disputes, and bill the pro's own monthly subscription." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Payment processing -- client card charges and refunds (deposits; from v1 balances; Later tips)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-07, FEAT-09, FEAT-30, FEAT-22, FEAT-23; this spec is named directly as the FEAT-09 Integration Spec covering "automatic full deposit refund on client cancellation outside the window and every Pro cancellation... with not-yet-completable refunds reported back for retry")
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Riley's deposit refunds automatically when she cancels outside Talia's window, with no action from Talia | Automatically refund the deposit for a cancellation made outside the window | FEAT-10 (Client-Initiated Cancel/Reschedule) shows the confirmed refund |
| Riley's deposit always refunds in full when Talia cancels, whatever the timing | Apply the counterpart rule: a Pro-initiated cancellation always refunds the client's deposit in full | FEAT-30 (Pro Booking Management) shows the confirmed refund |
| If a refund cannot complete immediately, both Talia and Riley see it as "in progress" rather than silently failing | Automatically refund the deposit for a cancellation made outside the window (Error-state guarantee) | FEAT-12 (Pro Daily Schedule Dashboard) attention flag; Riley's own booking status view |
| Riley's paid balance refunds in full, with no action from Talia, when Riley's cancellation (or an automatic cancellation) lands on a booking whose balance she already paid in-app | Refunded in full if the appointment is later cancelled by either party -- a balance is never subject to forfeiture (FEAT-22, XBR-23) | FEAT-22.SPEC-005 (executes the balance refund); FEAT-10 shows the outcome |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Refund amount and currency | Deposit Transaction -- amount, currency | FEAT-09.SPEC-004 determines a refund is due | The capability must know exactly how much to return, matching the original captured amount |
| Deposit reference | Deposit Transaction -- the reference tying it to the original capture | Refund is requested | Ties the refund to the specific original charge so the capability reverses the correct transaction |
| Payout account reference | Payout Account -- processor_account_reference | Refund is requested | Identifies which of the Pro's connected accounts the refund draws against |

Booking details (service, client note, appointment time), the Client's contact fields, and every other Deposit Transaction field beyond amount, currency, and the deposit reference never leave the product for this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Refund succeeded confirmation, with refund timestamp | The capability completes the refund | Deposit Transaction -- status, refund timestamp |
| Refund cannot complete immediately (e.g., the Pro's payout balance cannot yet cover it), with a retry-eligibility signal | The capability reports the refund could not be completed on this attempt | Deposit Transaction -- status (Refund in Progress); handed to FEAT-09.SPEC-006 for automatic retry |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Refund succeeded | The capability completes the requested refund | Deposit Transaction.status set to Refunded; refund timestamp recorded | The outcome is included in the relevant confirmation message (FEAT-08.SPEC-004), not a separate standalone notice; Riley sees her deposit refunded on her own booking view | FEAT-10 or FEAT-30 (whichever triggered), FEAT-08.SPEC-004 (confirmation content), FEAT-25.SPEC-004 (historical aggregate update on Refunded) |
| Refund could not complete immediately | The capability reports it cannot complete the refund on this attempt (for example, the Pro's payout balance cannot yet cover it) | Deposit Transaction.status set to Refund in Progress | Talia's dashboard shows a clear attention flag; Riley's booking view shows the refund as "in progress," never as failed or silent | FEAT-09.SPEC-006 (owns the retry loop), FEAT-12.SPEC-005 (attention flag aggregation), FEAT-25.SPEC-004 (historical aggregate update on Refund in Progress) |
| Balance refund due | FEAT-09.SPEC-004 records a client-initiated or automatic cancellation on a Booking that has a Balance Payment in Succeeded state (XBR-23), regardless of the deposit outcome (a balance is never forfeited) | No Deposit Transaction change. This spec hands the Booking and Balance Payment references to FEAT-22.SPEC-005, which executes the balance refund exactly once and owns the Balance Payment's Refunded / refund-in-progress state | Riley's cancellation confirmation (FEAT-08.SPEC-004) states that her paid balance is refunded in full, or in progress if FEAT-22.SPEC-005 reports it cannot complete immediately; Talia's dashboard flags an in-progress balance refund the same way as a deposit refund | FEAT-22.SPEC-005 (executes the refund), FEAT-08.SPEC-004, FEAT-12.SPEC-005, FEAT-25.SPEC-004 |
| Balance refund outcome reported back | FEAT-22.SPEC-005 reports its balance refund as Refunded or as refund-in-progress | None in this feature's entities; the outcome is reflected in the same cancellation confirmation content and attention signal as the deposit outcome | The confirmation reads the combined outcome (deposit and balance) so Riley sees one consistent statement, never a separate notice per refund | FEAT-08.SPEC-004, FEAT-12.SPEC-005 |
| Refund completes after a retry | FEAT-09.SPEC-006's retry succeeds on a later attempt | Deposit Transaction.status set to Refunded; refund timestamp recorded | The Pro's attention flag clears; Riley's "in progress" status updates to refunded, and the confirmation content reflects the completed refund | FEAT-09.SPEC-006, FEAT-12.SPEC-005, FEAT-08.SPEC-004, FEAT-25.SPEC-004 |

## Degradation Behavior

This integration is triggered by FEAT-09.SPEC-004's automation, not directly by a screen action, so no screen sends the refund request itself; the rows below cover the screens where this capability's trouble is visible to a user.

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-12 (Pro Daily Schedule Dashboard, attention list) | No visible change -- the refund request is not user-initiated on this screen, so a slow response produces no waiting state here; the attention flag simply has not yet appeared | If the capability is unreachable when the refund is requested, the attention flag reads "A refund for {client name}'s cancelled booking is in progress and will complete automatically." -- Talia sees no action she needs to take, and the flag persists until FEAT-09.SPEC-006's retry succeeds | If the capability explicitly rejects the refund request (for example, the payout account is no longer valid), the attention flag reads "A refund for {client name}'s cancelled booking needs attention -- your payout account may need reconnecting." with a link into FEAT-28 |
| Riley's own booking status view (FEAT-06/FEAT-10) | No visible change -- Riley sees no waiting state for a refund that has not yet been requested to fail or succeed | Riley's booking shows "Your deposit refund is in progress and will complete automatically." -- never a failure message | Riley's booking shows the same "in progress" wording; she is never shown the capability's rejection reason directly, since the resolution (e.g., reconnecting the payout account) is Talia's action, not hers |

## Consent and Disclosure

- **No new disclosure moment for the refund itself** -- Riley already agreed, at the moment she paid her deposit (FEAT-07), that the payment-processing capability holds and processes her payment method; reversing that same capture through the same capability requires no additional consent screen. The booking-time policy acknowledgment (FEAT-09.SPEC-002) already told her that an outside-window cancellation refunds automatically.
- **Payout account reference disclosure** -- Talia was told, when she connected her payout account (FEAT-28), that it would be used to receive deposits and to fund refunds and payouts; this integration's use of that same reference for a refund draws on that existing disclosure and requires no repeated notice.
- **What is never shared** -- Booking details, the Client's contact fields, and every Deposit Transaction field beyond amount, currency, and the deposit reference stay inside the product; the refund request never carries client contact information to the capability.

## Edge Cases

- **A refund-succeeded event arrives for a Deposit Transaction already marked Refunded** -- The second delivery changes nothing: the Deposit Transaction stays Refunded with its original refund timestamp, and no duplicate confirmation content fires.
- **A refund-could-not-complete event arrives after a refund-succeeded event for the same deposit (out-of-order delivery)** -- The Deposit Transaction reflects the most recent true state, not arrival order: since a deposit can be refunded only once (dependency map, Deposit Transaction Contention), a genuine refund-succeeded event is authoritative and a stale not-yet-completed report arriving late is treated as superseded and produces no status change.
- **The Booking or Client the refund relates to is deleted or de-identified before the refund event arrives** -- The event is still applied to the retained, de-identified financial record (per SC-22); no user feedback fires since there is no longer an active client-facing view to show it to.
- **The capability goes down mid-request, before confirming whether the refund was received** -- The Deposit Transaction is left at Refund in Progress rather than a half-resolved state; FEAT-09.SPEC-006's retry re-attempts the request, and a late success or rejection that eventually arrives is applied exactly as an on-time one would be.
- **A refund is requested twice for the same Deposit Transaction (e.g., an automation retry and a manual retry overlap)** -- The deposit reference sent with the request ties both attempts to the same original capture; the capability recognizes the resubmission as the same refund rather than issuing a second one, consistent with the "refunded only once" guarantee this spec and FEAT-09.SPEC-006 jointly uphold.
- **A cancellation lands on a Booking with a Succeeded Balance Payment and a Captured deposit that is forfeited (inside-window client cancellation)** -- The deposit is kept per FEAT-09.SPEC-003 but the balance is still handed to FEAT-22.SPEC-005 for a full refund; a balance is never forfeited, and the two outcomes are stated separately in the confirmation.
- **A cancellation lands on a Booking with no Succeeded Balance Payment (unpaid, declined, or already Refunded)** -- No hand-off to FEAT-22.SPEC-005 is made; the balance-refund path is skipped without any user feedback, and a cancellation racing an in-flight balance charge follows FEAT-22.SPEC-004's contention rule, not this spec.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-004 (Cancellation & No-Show Outcome Evaluation) | Triggered by (inbound) | A refund-due determination initiates this integration's request |
| FEAT-09.SPEC-006 (Refund Idempotency & Retry Rule) | Triggers (outbound) / References (inbound) | A not-yet-completable refund hands off to that spec's retry loop, which calls back into this spec's request contract |
| FEAT-28 (Payout Account Connection & Payout Visibility) | References (outbound) | Refunds draw on the Pro's connected payout account balance |
| FEAT-12.SPEC-005 (Attention Flag Aggregation) -- within FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | A refund that cannot complete immediately is flagged clearly on the Pro's dashboard until it resolves |
| FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund) -- within FEAT-22 (In-App Balance Payment) | Triggers (outbound) | A client-initiated or automatic cancellation on a Booking with a Succeeded Balance Payment invokes that spec's balance refund (XBR-23); its outcome is reflected back into the shared cancellation confirmation and attention signal |
| FEAT-25.SPEC-004 (Historical Aggregate Maintenance) -- within FEAT-25 (Booking Revenue Insights) | Affects (outbound) | Each Deposit Transaction change to Refund in Progress or Refunded fires that automation's aggregate update |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) -- within FEAT-08 (Automated Booking Messaging) | Affects (outbound) | The refund outcome (confirmed or in progress) is included in the relevant confirmation message sent to the client |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Affects (outbound) | Riley's own booking status view reflects the refund outcome |

## Analytics and Success Signals

- **deposit_refund_requested** (trigger type: client-cancel-outside / pro-cancel / pro-reschedule) -- supports success-metrics.md: "Automatic Refund Correctness"
- **deposit_refund_outcome_received** (outcome: succeeded / could-not-complete) -- supports success-metrics.md: "Automatic Refund Correctness"
- **balance_refund_handoff** (trigger type: client-cancel / automatic-cancel; outcome reported: refunded / in-progress) -- supports success-metrics.md: "Automatic Refund Correctness"
- **deposit_refund_failed** (reason category) -- supports success-metrics.md: "Automatic Refund Correctness" (a refund that cannot complete immediately must still be shown as "in progress" and complete without either party chasing it; this event measures how often that path is exercised)

## Acceptance Criteria

**FEAT-09.SPEC-005-AC-01:** Given FEAT-09.SPEC-004 determines Riley's cancellation is a refund-due outcome, when this integration requests the refund and the capability confirms it immediately, then the Deposit Transaction is set to Refunded with a refund timestamp, and Riley sees the confirmation reflecting her refund.

**FEAT-09.SPEC-005-AC-02:** Given a refund request is sent to the capability, when the capability reports it cannot complete on this attempt, then the Deposit Transaction is set to Refund in Progress, Talia's dashboard shows the attention flag, and Riley's booking shows "in progress," never a failure.

**FEAT-09.SPEC-005-AC-03:** Given the capability is unreachable when a refund is requested, then Talia's dashboard shows "A refund for {client name}'s cancelled booking is in progress and will complete automatically." with no action required from her yet.

**FEAT-09.SPEC-005-AC-04:** Given the capability explicitly rejects a refund request because the payout account is no longer valid, then Talia's dashboard shows "A refund for {client name}'s cancelled booking needs attention -- your payout account may need reconnecting." with a link into FEAT-28.

**FEAT-09.SPEC-005-AC-05:** Given a Deposit Transaction is already Refunded, when the same refund-succeeded event is delivered again, then nothing changes and no duplicate confirmation content fires.

**FEAT-09.SPEC-005-AC-06:** Given a refund-could-not-complete event arrives after a refund-succeeded event for the same deposit, then the Deposit Transaction remains Refunded and the late, superseded report produces no status change.

**FEAT-09.SPEC-005-AC-07:** Given the Client whose deposit is being refunded has since been deleted, when the refund event arrives, then it is applied to the retained de-identified financial record with no user feedback fired.

**FEAT-09.SPEC-005-AC-08:** Given the capability goes down mid-request before confirming receipt, then the Deposit Transaction is left at Refund in Progress rather than any half-resolved state, and a later-arriving outcome is applied exactly as an on-time one would be.

**FEAT-09.SPEC-005-AC-09:** Given a refund request for the same Deposit Transaction is submitted twice (an automatic retry overlapping a prior attempt), then the capability recognizes the resubmission via the shared deposit reference and issues only one refund.

**FEAT-09.SPEC-005-AC-10:** Given Riley has never had a deposit refunded before, when her first outside-window cancellation triggers this integration, then no new consent screen appears -- the refund proceeds under the disclosure she already received at booking and at deposit payment.

**FEAT-09.SPEC-005-AC-11:** Given this integration requests a refund, when the request is composed, then only the refund amount, currency, deposit reference, and payout account reference are sent -- Booking details and Client contact fields are never included.

**FEAT-09.SPEC-005-AC-12:** Given Talia looks at the attention flag for a refund in progress, when she reads it, then it never contains technical detail about the capability -- only the plain-language "in progress, will complete automatically" or the "needs attention -- reconnect payout account" wording defined above.

**FEAT-09.SPEC-005-AC-13:** Given a refund that could not complete immediately eventually succeeds after FEAT-09.SPEC-006's retry, then the Deposit Transaction is set to Refunded, Talia's attention flag clears, and Riley's "in progress" status updates to reflect the completed refund.

**FEAT-09.SPEC-005-AC-14:** Given Riley cancels a Booking outside the window and its Balance Payment is in Succeeded state, when FEAT-09.SPEC-004 records the cancellation, then this spec hands the Booking and Balance Payment references to FEAT-22.SPEC-005 alongside the deposit refund, and Riley's confirmation states that both her deposit and her paid balance are refunded in full.

**FEAT-09.SPEC-005-AC-15:** Given Riley cancels a Booking inside the window (deposit forfeited) and its Balance Payment is in Succeeded state, when the cancellation is recorded, then the deposit remains forfeited while the balance is still handed to FEAT-22.SPEC-005 for a full refund, and the confirmation states the two outcomes separately.

**FEAT-09.SPEC-005-AC-16:** Given a cancellation is recorded on a Booking with no Succeeded Balance Payment, when this spec processes the refund-due outcome, then no request is made to FEAT-22.SPEC-005 and no balance-related message is shown to Riley or Talia.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 5 | 5 |
| Degradation Paths | 6 (2 screens x 3 conditions) | 6 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 7 | 7 |
