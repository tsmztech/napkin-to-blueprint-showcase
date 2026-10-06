---
document_type: spec
spec_type: integration
spec_id: FEAT-22.SPEC-005
spec_name: Balance Charge, Payout Routing & Refund
spec_slug: balance-charge-payout-routing-refund
parent_feature: FEAT-22
parent_feature_name: In-App Balance Payment
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 18
---

# Integration Spec: Balance Charge, Payout Routing & Refund

## Overview

**Name:** Balance Charge, Payout Routing & Refund
**ID:** FEAT-22.SPEC-005
**Type:** Integration
**Purpose:** Authorizes and captures Riley's in-app balance card charge through the payment-processing capability, routes the captured balance to Talia's connected payout account with zero platform fee, and executes the outbound refund call when a paid balance must be refunded in full.
**Parent Feature:** FEAT-22 -- In-App Balance Payment

## Scope and Non-Goals

**In Scope:**
- Requesting authorization and capture of a balance charge for an eligible Booking, using the card details Riley enters directly into the capability's own entry element
- Receiving and translating the capability's outcome (captured, declined) into plain-language feedback for FEAT-22.SPEC-001
- Routing the captured amount to the Pro's connected Payout Account, with Chairtime's own fee always zero (XBR-07)
- Executing the outbound refund call for a Succeeded Balance Payment once FEAT-22.SPEC-004's contention resolution determines a refund is due, guaranteeing it completes exactly once with automatic retry (XBR-23)
- User-facing behavior when the capability is slow, unavailable, or rejects a charge or refund request
- Disclosure to Riley about what data is shared with the capability

**Non-Goals:**
- Verifying or connecting the Pro's payout account itself (identity and bank verification) -- owned by FEAT-28.SPEC-006 (Payout Account Connection & Payout Visibility); this spec only routes an already-captured balance to an already-active account
- Computing the balance amount or checking eligibility preconditions -- owned by FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules); this spec only submits the charge for an amount and Booking that spec has already cleared
- Creating the Balance Payment record or updating the Booking's balance-due status -- owned by FEAT-22.SPEC-002 (Balance Capture & Booking Status Update); this spec reports the charge outcome that automation acts on, it does not write the Balance Payment itself
- Determining that a refund is due, or the underlying cancellation decision -- owned by FEAT-30 (Pro Booking Management, for a Pro-initiated cancellation) and FEAT-09 (Cancellation & No-Show Policy Engine, for a client-initiated or automatic cancellation) together with FEAT-22.SPEC-004's contention rule; this spec only executes a refund already determined to be due

## Capability Category

**Category:** Payment processing
**Dependency Source:** ASMP-31 -- "Payment-processing capability -- required to take client deposits, verify each pro's identity and bank details for a connected payout account, pay deposits out to the pro, issue refunds, notify the product of card-issuer disputes, and bill the pro's own monthly subscription." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Payment processing -- client card charges and refunds (deposits; from v1 balances; Later tips)" row in feature-dependency-map.md, ## External Touchpoints, which names this spec directly as "FEAT-22.SPEC-005 (in-app balance card authorization and capture with zero platform fee, and the outbound full refund of a paid balance when either party cancels, per XBR-23)"; and the "Payment processing -- connected payout accounts with identity and bank verification" row, which names this spec for routing each captured balance to the Pro's connected payout account
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision. BRIEF.md's Constraints establish only the hard boundary that card data is never stored or handled by the product's own code, not a named vendor.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Riley pays the exact balance amount computed for her booking, by card, without her card details ever touching the product's own code | Pay the remaining balance in-app at any point before or at the appointment | FEAT-22.SPEC-001 (Balance Payment) |
| Riley sees a clear, on-screen success record that her balance succeeded, and Talia's dashboard reflects "fully paid" | See a running record of deposit paid vs. balance remaining | FEAT-22.SPEC-001 (Balance Payment), FEAT-22.SPEC-002 (Balance Capture & Booking Status Update), FEAT-12 (Pro Daily Schedule Dashboard) |
| Talia's captured balance lands directly in her own payout account with no Chairtime cut | Balance goes straight to the Pro's payout account with no platform cut | FEAT-28 (Payout Account Connection & Payout Visibility) -- the money-list display itself; this spec supplies the routed amount |
| A paid balance is refunded in full, never partially or forfeited, the instant either party cancels | Refunded in full if the appointment is later cancelled by either party -- a balance is never subject to forfeiture | FEAT-22.SPEC-004 (contention and refund-due determination), FEAT-30 (Pro Booking Management, cancellation commit), FEAT-09.SPEC-005 (Automatic Deposit Refund, client/automatic cancellation), FEAT-23.SPEC-003 (tip refund completeness, once shipped) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Balance amount and currency | Booking -- price_agreed, currency (via the derived balance amount FEAT-22.SPEC-003 locks) | Riley taps Pay on FEAT-22.SPEC-001, after FEAT-22.SPEC-003's eligibility check and FEAT-22.SPEC-004's contention check both pass | The capability must know exactly what to authorize and capture |
| Booking reference | Booking -- an internal reference sufficient to tie the outcome back to this specific Booking | Same moment as above | Ties the capability's reported outcome (captured, declined) back to the correct Booking, and carries the idempotency key FEAT-22.SPEC-004 requires |
| Destination payout account reference | Payout Account -- processor_account_reference | Same moment as above | Tells the capability where to route the captured funds -- the Pro's own connected account, never Chairtime's |
| Card details Riley enters | Not a product entity -- entered directly into the capability's own entry element and never received by the product's own code, per SC-11 | While Riley fills the card entry element on FEAT-22.SPEC-001 | The capability needs the card details to attempt authorization; the product never touches or stores them |
| Refund amount and currency | Balance Payment -- amount | FEAT-22.SPEC-004's contention resolution determines a refund is due (a Pro, client, or automatic cancellation on a Booking with a Succeeded Balance Payment) | The capability must know exactly how much to return, matching the original captured amount |
| Balance Payment reference | Balance Payment -- the reference tying it to the original capture | Refund is requested | Ties the refund to the specific original charge so the capability reverses the correct transaction |
| Refund idempotency key | Derived -- an identifier tied to this specific refund attempt on this specific Balance Payment | Every refund request, including retries | Lets the capability recognize a resubmitted request as the same attempt rather than a second refund |

Booking's service name, appointment time, client name and phone, and every other Client or Booking field never leave the product through this integration -- only the amounts, currency, internal references, and the payout destination reference are shared.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Capture outcome (captured / declined) | The capability resolves an authorization/capture request | Consumed directly by FEAT-22.SPEC-002 (captured) or shown inline on FEAT-22.SPEC-001 (declined); no Balance Payment is written for a decline |
| Decline reason (plain-language category) | The capability reports a decline | Surfaced on FEAT-22.SPEC-001's Error state; not persisted on the Booking |
| Payout routing confirmation | The capability confirms the captured amount was routed to the destination Payout Account | Feeds Payout Account's recent-payouts data, displayed by FEAT-28's money list |
| Refund succeeded confirmation, with refund timestamp | The capability completes the refund | Balance Payment -- state (Refunded), refund timestamp |
| Refund cannot complete immediately, with a retry-eligibility signal | The capability reports the refund could not be completed on this attempt | Balance Payment -- state remains flagged as refund-in-progress; scheduled for this spec's own automatic retry |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Charge captured | The capability successfully authorizes and captures Riley's card for the requested balance amount | None directly in this spec -- the captured amount and currency are reported to FEAT-22.SPEC-002, which creates the Balance Payment and updates the Booking's balance-due status | FEAT-22.SPEC-001 shows the Success state; the Pro's schedule (FEAT-12) and money list (FEAT-28) reflect it once FEAT-22.SPEC-002 and payout routing complete | FEAT-22.SPEC-002 (Balance Capture & Booking Status Update) |
| Charge declined | The capability cannot capture the charge (insufficient funds, card declined, expired card, or another card-level reason) | None -- no Balance Payment is created for a decline | FEAT-22.SPEC-001 shows the Error state with the plain-language decline reason and a "Try a different card" action | FEAT-22.SPEC-001 (Balance Payment) |
| Charge captured but cannot be applied (a cancellation committed first, per FEAT-22.SPEC-004) | The capability reports a successful capture, but FEAT-22.SPEC-002 finds the Booking's cancellation already committed | No Balance Payment is created; this spec immediately requests a refund of the captured amount, using this same integration's refund path | FEAT-22.SPEC-001 shows the Cancelled state ("This booking was just cancelled, so there's nothing to pay. Your card was not charged.") -- the brief moment of capture is never exposed to Riley as a charge that then reverses; the product treats it as never having charged her | FEAT-22.SPEC-002, FEAT-22.SPEC-004 |
| Payout routing confirmed | The captured amount is successfully routed to the Pro's connected Payout Account | Feeds Payout Account's recent-payouts data (owned and displayed by FEAT-28) | No separate feedback to Riley; Talia sees it reflected in her own money list (FEAT-28) on her own schedule | FEAT-28 (Payout Account Connection & Payout Visibility) |
| Refund succeeded | The capability completes a requested balance refund | Balance Payment.state set to Refunded; refund timestamp recorded | Included in the relevant cancellation confirmation content (FEAT-30.SPEC-012 for a Pro-initiated cancellation; the equivalent client-facing cancellation confirmation for a client or automatic cancellation), never a separate standalone notice; Riley sees her balance refunded on her own booking view | FEAT-30.SPEC-007, FEAT-30.SPEC-008, FEAT-09.SPEC-005 |
| Refund could not complete immediately | The capability reports it cannot complete the refund on this attempt (for example, the Pro's payout balance cannot yet cover it) | Balance Payment marked refund-in-progress; a retry is scheduled on a fixed cadence (platform parameter: `refund-retry-interval-hours`, the same cadence FEAT-09.SPEC-006 and FEAT-30.SPEC-011 apply to deposit refunds) | Talia's dashboard shows the same attention flag pattern FEAT-12 already shows for a delayed deposit refund; Riley's booking view shows the balance refund as "in progress," never as failed or silent | FEAT-12 |
| Charge request times out with no result received | The capability does not respond within the product's expected response window | None -- no Balance Payment is created while the outcome is unknown | FEAT-22.SPEC-001 shows its Offline/Degraded resolution; FEAT-22.SPEC-004 governs how a subsequent late response (success or decline) is reconciled once it does arrive | FEAT-22.SPEC-001 (Balance Payment), FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention) |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-22.SPEC-001 (Balance Payment) | The status region shows "Processing payment, do not close this page." for as long as the request is in flight; past a brief threshold it adds "Still working -- this is taking longer than usual." The Pay button and card entry element remain disabled throughout. | The Pay button is disabled before a request is even sent, with the message "We can't take payments right now. Please try again in a few minutes, or pay {Pro's display name} in person." The in-person default remains fully available regardless of this capability's status. | The specific decline reason is shown in plain language (for example, "Your card was declined. Try a different card." or "This card has expired. Try a different card."), never a raw processor code; a "Try a different card" action is offered, per the never-ambiguous-outcome guarantee in FEAT-22.SPEC-004. |
| FEAT-12 (Pro Daily Schedule Dashboard, attention list) -- refund side | No visible change -- the refund request is not user-initiated on this screen, so a slow response produces no waiting state here | If the capability is unreachable when a balance refund is requested, the attention flag reads "A refund for {client name}'s cancelled booking is in progress and will complete automatically." -- mirroring the existing deposit-refund attention pattern (FEAT-30.SPEC-011) | If the capability explicitly rejects the refund request (for example, the payout account is no longer valid), the attention flag reads "A refund for {client name}'s cancelled booking needs attention -- your payout account may need reconnecting." with a link into FEAT-28 |
| Riley's own booking status view (FEAT-06.SPEC-004, Booking Detail via Manage Link) -- refund side | No visible change -- Riley sees no waiting state for a refund that has not yet been requested to fail or succeed | Riley's booking shows "Your balance refund is in progress and will complete automatically." -- never a failure message | Riley's booking shows the same "in progress" wording; she is never shown the capability's rejection reason directly, since the resolution (e.g., reconnecting the payout account) is Talia's action, not hers |

No other screen sends requests to this capability or displays its live results; the Pro's own surfaces (FEAT-12, FEAT-28, FEAT-30) reflect only already-resolved outcomes, which are unaffected by a degradation condition that occurs before resolution.

## Consent and Disclosure

- **Card entry disclosure** -- Directly above the card entry element on FEAT-22.SPEC-001, a persistent line reads: "Your card details go straight to our payment processor -- Chairtime never sees or stores your card number." This is shown every time the screen is reached, not only on first use, since each balance payment is a fresh charge with no card kept on file, consistent with the deposit feature's own practice.
- **What the payout destination receives** -- The disclosure line above also implicitly covers that the payment is destined for the Pro's own connected account; no separate consent prompt is needed for this, since Riley is already paying that specific Pro's booking and the amount and recipient are evident from the booking context itself.
- **What is never shared** -- The Booking's service name, appointment time, Riley's name, phone, email, and any booking note never leave the product through this integration; only the amounts, currency, internal references, and the Pro's payout destination reference are sent, exactly as scoped in Data Exchanged.
- **No new disclosure moment for the refund itself** -- Riley already agreed, at the moment she paid her balance, that the payment-processing capability holds and processes her payment method; reversing that same capture through the same capability requires no additional consent screen, consistent with the existing deposit-refund disclosure practice (FEAT-30.SPEC-011).
- **Pro-facing disclosure of the fee deduction** -- Talia was already told, on viewing her money list after her first captured deposit (FEAT-07.SPEC-005), that Chairtime never takes a cut and the only deduction is the processor's own card fee; this spec's routed balance amounts follow the same disclosure and require no repeated notice.

## Edge Cases

- **The same capture-succeeded event is delivered twice** -- The second delivery reaches FEAT-22.SPEC-002, which finds an existing Balance Payment and takes no further action; this spec itself performs no state changes on receipt, so duplicate delivery has no direct effect here beyond the automation's own idempotency handling.
- **A capture event arrives for a Booking that has since left a payable state for an unrelated reason** -- Handled by FEAT-22.SPEC-002 and FEAT-22.SPEC-004's correctness guarantee, not silently dropped by this spec; if the reason is a committed cancellation, this spec immediately requests a refund of the captured amount rather than leaving it unresolved.
- **A decline and a late capture-succeeded report both arrive for the same attempt (out-of-order delivery)** -- The capability's own outcome for a single charge request is authoritative and final once reported; a genuinely late success after an already-shown decline can only occur if the two reports describe two separate requests (Riley's retry after the decline), each carrying its own idempotency key per FEAT-22.SPEC-004, so they are never conflated into one ambiguous outcome.
- **The capability goes down mid-authorization, after the request was sent but before any result is received** -- No Balance Payment is created while the outcome is unknown; FEAT-22.SPEC-001 shows its Offline/Degraded state, and if the capability's result eventually does arrive (success or decline), FEAT-22.SPEC-004 reconciles it against the Booking's true state rather than assuming failure.
- **A refund-succeeded event arrives for a Balance Payment already marked Refunded** -- The second delivery changes nothing: the Balance Payment stays Refunded with its original refund timestamp, and no duplicate confirmation content fires.
- **Two overlapping retry attempts for the same Balance Payment's refund are both processed** -- Only one results in a Refunded state; the other, whichever resolves second, finds the Balance Payment already Refunded via the idempotency key and is treated as a no-op with no duplicate refund.
- **A refund stays in progress for an extended period because the underlying blocking condition never resolves** -- The retry continues indefinitely on its fixed cadence (platform parameter: `refund-retry-interval-hours`); the Balance Payment is never silently abandoned, and Talia's attention flag persists throughout, per XBR-10's "never dropped" guarantee applied to this balance refund exactly as it applies to a deposit refund.
- **Payout routing confirmation is delayed after a successful capture** -- The capture outcome (Balance Payment Succeeded, Booking fully paid) is not held pending on payout routing; Riley's success state is never delayed by a slow payout leg.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-001 (Balance Payment) | Triggered by (inbound) | The Pay action initiates the authorization/capture request |
| FEAT-22.SPEC-001 (Balance Payment) | Affects (outbound) | Processing, decline, Cancelled, and offline/degraded states surface this spec's reported outcomes and degradation behavior |
| FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules) | Triggered by (inbound) | Eligibility passing is the precondition for this spec ever sending a charge request |
| FEAT-22.SPEC-002 (Balance Capture & Booking Status Update) | Triggers (outbound) | A reported successful capture fires that automation's create-and-update step |
| FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention) | References (inbound) | Supplies the idempotency-key discipline this spec applies to every charge and refund request, and determines when a refund is due |
| FEAT-28 (Payout Account Connection & Payout Visibility) | Affects (outbound) | The routed balance is what that feature's money list displays; refunds draw on the same connected payout account |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | A refund that cannot complete immediately is flagged clearly on the Pro's dashboard until it resolves |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit), FEAT-30.SPEC-008 (Bulk Cancellation Commit) -- within FEAT-30 (Pro Booking Management) | Triggered by (inbound) | A Pro-initiated cancellation on a Booking with a Succeeded Balance Payment initiates this spec's refund request |
| FEAT-09.SPEC-005 (Automatic Deposit Refund) -- within FEAT-09 (Cancellation & No-Show Policy Engine) | Triggered by (inbound) | A client-initiated or automatic cancellation on a Booking with a Succeeded Balance Payment initiates this spec's refund request |
| FEAT-23.SPEC-003 (Tip Payout & Refund Rule) -- within FEAT-23 (Tipping at Checkout, Later) | References (inbound) | Rule spec enforced by this integration (Enforced-By): once FEAT-23 ships, the payout routing passes any tip to the Pro's Payout Account with zero deduction, and the refund request covers the balance plus tip together in full (XBR-23). At v1 no tip exists and the refund equals the balance amount |
| FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution) | References (outbound) | Mirrors this spec's idempotency-key and retry-cadence discipline for the sibling deposit-refund integration, per the Brief's Shared Validation note |

**Cross-feature note for reconciliation:** FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) describes its Balance Payment refund step as running "through the same payment-processing capability path FEAT-30.SPEC-011 uses for other Pro-triggered refunds." This feature's Brief and the External Touchpoints table both assign the outbound balance refund call to this spec (FEAT-22.SPEC-005) instead. This spec is written as the authoritative owner of the balance refund call per that assignment; Pass D reconciliation has aligned FEAT-30.SPEC-007 (and FEAT-30.SPEC-008) to cite FEAT-22.SPEC-005 for the balance leg specifically. Separately, FEAT-09.SPEC-005 (Automatic Deposit Refund) is the inbound trigger for a client-initiated or automatic cancellation: when the cancelled Booking has a Succeeded Balance Payment, FEAT-09.SPEC-005 hands the full balance refund to this spec (XBR-23), alongside its own deposit refund. FEAT-30.SPEC-007 and FEAT-30.SPEC-008 are the callers for a Pro-initiated cancellation and bulk cancellation, so all three callers converge on this spec as the single owner of the outbound balance refund call.

## Analytics and Success Signals

- **balance_charge_requested** (booking reference, balance amount, currency) -- supports success-metrics.md: "Payout Transparency"
- **balance_charge_captured** (booking reference) -- supports success-metrics.md: "Payout Transparency"
- **balance_charge_declined** (booking reference, decline reason category) -- supports success-metrics.md: "Payout Transparency"
- **balance_charge_reversed_after_cancellation** (booking reference) -- N/A -- no Stage 2 metric measures this exact race outcome directly; retained so the never-charge-for-a-cancelled-booking guarantee's exercise rate is observable to the Pro's activity record (FEAT-16), not to a success metric
- **balance_payout_routing_confirmed** (booking reference) -- supports success-metrics.md: "Payout Transparency"
- **balance_refund_requested** (booking reference, source: pro_cancellation / client_cancellation / automatic_cancellation) -- supports success-metrics.md: "Automatic Refund Correctness"
- **balance_refund_outcome_received** (outcome: succeeded / could-not-complete) -- supports success-metrics.md: "Automatic Refund Correctness"

## Acceptance Criteria

**FEAT-22.SPEC-005-AC-01:** Given Riley's Booking has passed FEAT-22.SPEC-003's eligibility check and FEAT-22.SPEC-004's contention check, when she taps Pay, then a charge request for exactly the locked balance amount and currency is sent to the payment-processing capability, carrying an idempotency key and the Booking reference.

**FEAT-22.SPEC-005-AC-02:** Given the capability successfully captures Riley's charge, when it reports the outcome, then this spec reports the captured amount and currency to FEAT-22.SPEC-002 without creating the Balance Payment itself.

**FEAT-22.SPEC-005-AC-03:** Given the capability declines Riley's card, when it reports the decline, then FEAT-22.SPEC-001 shows the plain-language decline reason and a "Try a different card" action, and no Balance Payment is created.

**FEAT-22.SPEC-005-AC-04:** Given a captured balance charge, when payout routing completes, then the amount is routed to Talia's connected Payout Account with a Chairtime fee of zero.

**FEAT-22.SPEC-005-AC-05:** Given Riley is on the Balance Payment screen, when she reaches the card entry element, then she sees "Your card details go straight to our payment processor -- Chairtime never sees or stores your card number." above it.

**FEAT-22.SPEC-005-AC-06:** Given a charge request is in flight, when more than the product's brief processing threshold passes without a result, then the status region adds "Still working -- this is taking longer than usual." while the Pay button remains disabled.

**FEAT-22.SPEC-005-AC-07:** Given the payment-processing capability is unavailable when Riley reaches the Pay step, when she views the screen, then the Pay button is disabled with "We can't take payments right now. Please try again in a few minutes, or pay {Pro's display name} in person."

**FEAT-22.SPEC-005-AC-08:** Given the capability rejects a charge with a specific reason, when the rejection is reported, then Riley sees that specific plain-language reason, never a raw processor code.

**FEAT-22.SPEC-005-AC-09:** Given a capture-succeeded event is delivered twice for the same Booking, when the second delivery reaches FEAT-22.SPEC-002, then no second Balance Payment is created.

**FEAT-22.SPEC-005-AC-10:** Given the capability reports a captured charge but the Booking's cancellation already committed, when FEAT-22.SPEC-002 checks the Booking's state, then no Balance Payment is created and this spec immediately requests a refund of the captured amount.

**FEAT-22.SPEC-005-AC-11:** Given a Succeeded Balance Payment exists on a Booking that is cancelled by Talia (FEAT-30.SPEC-007 or FEAT-30.SPEC-008), by Riley, or automatically (FEAT-09.SPEC-005), when the cancelling spec hands off the refund, then this spec requests the refund and, on the capability's confirmation, sets the Balance Payment to Refunded with a refund timestamp.

**FEAT-22.SPEC-005-AC-12:** Given a refund request is sent to the capability, when the capability reports it cannot complete on this attempt, then the Balance Payment is marked refund-in-progress, Talia's dashboard shows the attention flag, and Riley's booking shows "in progress," never a failure.

**FEAT-22.SPEC-005-AC-13:** Given a refund that could not complete immediately, when the next scheduled retry runs (platform parameter: `refund-retry-interval-hours` after the previous attempt), then a new refund request is submitted carrying the same attempt's idempotency key.

**FEAT-22.SPEC-005-AC-14:** Given a Balance Payment is already Refunded, when the same refund-succeeded event is delivered again, then nothing changes and no duplicate confirmation content fires.

**FEAT-22.SPEC-005-AC-15:** Given two overlapping retry attempts for the same Balance Payment's refund both resolve, then only one results in a Refunded status, and the other is treated as a no-op with no duplicate refund.

**FEAT-22.SPEC-005-AC-16:** Given a refund's blocking condition never resolves, when successive scheduled retries continue to fail, then the Balance Payment remains refund-in-progress indefinitely rather than moving to a terminal failure state, and Talia's attention flag persists throughout.

**FEAT-22.SPEC-005-AC-17:** Given this integration requests a charge or a refund, when the request is composed, then only the amount, currency, an internal reference, the payout account reference, and an idempotency key are sent -- Booking details and Client contact fields are never included.

**FEAT-22.SPEC-005-AC-18:** Given a capture outcome is confirmed, when Riley's Success state is shown, then it is never delayed waiting for the separate payout-routing confirmation to Talia's account.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 7 | 7 |
| Degradation Paths | 9 (3 screens x 3 conditions) | 9 |
| Consent and Disclosure | 5 | 5 |
| Edge Cases | 8 | 8 |
