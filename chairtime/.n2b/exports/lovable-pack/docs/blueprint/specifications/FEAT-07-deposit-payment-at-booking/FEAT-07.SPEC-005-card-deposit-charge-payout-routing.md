---
document_type: spec
spec_type: integration
spec_id: FEAT-07.SPEC-005
spec_name: Card Deposit Charge & Payout Routing
spec_slug: card-deposit-charge-payout-routing
parent_feature: FEAT-07
parent_feature_name: Deposit Payment at Booking
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 16
---

# Integration Spec: Card Deposit Charge & Payout Routing

## Overview

**Name:** Card Deposit Charge & Payout Routing
**ID:** FEAT-07.SPEC-005
**Type:** Integration
**Purpose:** Authorizes and captures Riley's card charge through the payment-processing capability, reports the processor's own card fee, and routes the captured deposit to Talia's connected payout account with zero platform fee.
**Parent Feature:** FEAT-07 -- Deposit Payment at Booking

## Scope and Non-Goals

**In Scope:**
- Requesting authorization and capture of a deposit charge for an eligible Booking, using the card details Riley enters directly into the capability's own entry element
- Receiving and translating the capability's outcome (captured, declined) into plain-language feedback for FEAT-07.SPEC-001
- Reporting the capability's own processor_fee on the captured transaction
- Routing the captured amount to the Pro's connected Payout Account, with Chairtime's own fee always zero (XBR-07)
- User-facing behavior when the capability is slow, unavailable, or rejects a request
- Disclosure to Riley about what data is shared with the capability

**Non-Goals:**
- Verifying or connecting the Pro's payout account itself (identity and bank verification) -- owned by FEAT-28.SPEC-006 (Payout Account Connection & Payout Visibility); this spec only routes an already-captured deposit to an already-active account
- Computing the deposit amount or checking eligibility preconditions -- owned by FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules); this spec only submits the charge for an amount and Booking that spec has already cleared
- Creating the Deposit Transaction record or confirming the Booking -- owned by FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation); this spec reports the outcome that automation acts on, it does not write the Deposit Transaction itself
- Refunds triggered by cancellation, no-show forfeiture, or goodwill -- excluded per the External Touchpoints table: those flows are FEAT-09.SPEC-005 and FEAT-30.SPEC-011's own integration behavior against the same payment-processing capability, not this spec's; this spec covers only the original deposit charge and its payout routing

## Capability Category

**Category:** Payment processing
**Dependency Source:** ASMP-31 -- "Payment-processing capability -- required to take client deposits, verify each pro's identity and bank details for a connected payout account, pay deposits out to the pro, issue refunds, notify the product of card-issuer disputes, and bill the pro's own monthly subscription." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Payment processing -- client card charges and refunds (deposits; from v1 balances; Later tips)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved includes FEAT-07; Integration Specs column names FEAT-07.SPEC-005 as the deposit card authorization and capture, processor fee reporting, zero platform fee); and the "Payment processing -- connected payout accounts with identity and bank verification" row, which also names FEAT-07.SPEC-005 for routing each captured deposit to the Pro's connected payout account
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision. BRIEF.md's Constraints establish only the hard boundary that card data is never stored or handled by the product's own code, not a named vendor.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Riley pays the exact deposit amount required by the selected service's rule, by card, without her card details ever touching the product's own code | Pay the exact deposit amount required by the selected service's rule, by card | FEAT-07.SPEC-001 (Deposit Payment) |
| Riley sees a clear, on-screen confirmed record that her deposit succeeded | See a clear on-screen and confirmed record that the deposit succeeded | FEAT-07.SPEC-001 (Deposit Payment), FEAT-05.SPEC-005 (Booking Confirmation) |
| Riley sees a failed or declined payment explained in plain language and can retry without losing her held slot | Have a failed or declined payment explained clearly, with the slot held briefly to retry | FEAT-07.SPEC-001 (Deposit Payment) |
| Talia's deposit lands directly in her own payout account with no Chairtime cut, and she can see the processor's own card fee | Deposit lands directly in the Pro's own payout account, with the platform taking no cut; the only deduction is the payment processor's own card fee, shown to the Pro | FEAT-28 (Payout Account Connection & Payout Visibility) -- the money-list display itself; this spec supplies the routed amount and fee it displays |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Deposit amount and currency | Booking -- deposit_amount, currency | Riley taps Pay on FEAT-07.SPEC-001, after FEAT-07.SPEC-003's eligibility check passes | The capability must know exactly what to authorize and capture |
| Booking reference | Booking -- an internal reference sufficient to tie the outcome back to this specific Booking | Same moment as above | Ties the capability's reported outcome (captured, declined) back to the correct Booking, and carries the idempotency key FEAT-07.SPEC-004 requires |
| Destination payout account reference | Payout Account -- processor_account_reference | Same moment as above | Tells the capability where to route the captured funds -- the Pro's own connected account, never Chairtime's |
| Card details Riley enters | Not a product entity -- entered directly into the capability's own entry element and never received by the product's own code, per SC-11 | While Riley fills the card entry element on FEAT-07.SPEC-001 | The capability needs the card details to attempt authorization; the product never touches or stores them |

Booking's service name, appointment time, client name and phone, and every other Client or Booking field never leave the product through this integration -- only the deposit amount, currency, an internal Booking reference, and the payout destination reference are shared.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Capture outcome (captured / declined) | The capability resolves an authorization/capture request | Consumed directly by FEAT-07.SPEC-002 (captured) or shown inline on FEAT-07.SPEC-001 (declined); no Deposit Transaction is written for a decline |
| Decline reason (plain-language category) | The capability reports a decline | Surfaced on FEAT-07.SPEC-001's Error state; not persisted on the Booking, since a decline never reaches the Deposit Transaction entity |
| processor_fee | The capability reports the fee alongside a successful capture | Deposit Transaction -- processor_fee (written by FEAT-07.SPEC-002 at the moment it creates the record from this spec's reported values) |
| Payout routing confirmation | The capability confirms the captured amount was routed to the destination Payout Account | Feeds Payout Account's recent-payouts data, displayed by FEAT-28's money list; this spec supplies the confirmation, FEAT-28 owns the display |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Charge captured | The capability successfully authorizes and captures Riley's card for the requested deposit amount | None directly in this spec -- the captured amount, currency, and processor_fee are reported to FEAT-07.SPEC-002, which creates the Deposit Transaction and confirms the Booking | FEAT-07.SPEC-001 shows the Success state; the Pro's money list (FEAT-28) will show the deposit once FEAT-07.SPEC-002 and the payout routing confirmation complete | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) |
| Deposit outcome set (captured) | The Charge captured event above completes successfully | None in this spec -- the captured amount and processor_fee are reported onward for FEAT-25.SPEC-004 to update its deposits-collected aggregate | None to Riley; Talia's insights figures reflect the deposit on their next refresh | FEAT-25.SPEC-004 (Historical Aggregate Maintenance) |
| Charge declined | The capability cannot capture the charge (insufficient funds, card declined, expired card, or another card-level reason) | None -- no Deposit Transaction is created for a decline | FEAT-07.SPEC-001 shows the Error state with the plain-language decline reason and a "Try a different card" action; the Booking remains Pending Payment and the checkout hold is untouched | FEAT-07.SPEC-001 (Deposit Payment) |
| Payout routing confirmed | The captured amount is successfully routed to the Pro's connected Payout Account | Feeds Payout Account's recent-payouts data (owned and displayed by FEAT-28) | No separate feedback to Riley; Talia sees it reflected in her own money list (FEAT-28) on her own schedule | FEAT-28 (Payout Account Connection & Payout Visibility) |
| Charge request times out with no result received | The capability does not respond within the product's expected response window | None -- no Deposit Transaction is created while the outcome is unknown | FEAT-07.SPEC-001 shows its Offline/Degraded resolution; FEAT-07.SPEC-004 governs how a subsequent late response (success or decline) is reconciled once it does arrive | FEAT-07.SPEC-001 (Deposit Payment), FEAT-07.SPEC-004 (Payment Outcome Consistency & Idempotency) |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-07.SPEC-001 (Deposit Payment) | The status region shows "Processing payment, do not close this page." for as long as the request is in flight; past a brief threshold it adds "Still working -- this is taking longer than usual." The Pay button and card entry element remain disabled throughout; Riley's checkout hold is not affected by processing time alone. | The Pay button is disabled before a request is even sent, with the message "We can't take payments right now. Please try again in a few minutes -- your held time is still counting down." Riley's held slot continues to count down under its normal window (XBR-02); if it expires while the capability is down, the Hold Expired state applies exactly as it would for any other cause. | The specific decline reason is shown in plain language (for example, "Your card was declined. Try a different card." or "This card has expired. Try a different card."), never a raw processor code; a "Try a different card" action is offered and the Booking remains Pending Payment, per the never-ambiguous-outcome guarantee in FEAT-07.SPEC-004. |

No other screen sends requests to this capability or displays its live results; the Pro's own surfaces (FEAT-12, FEAT-28, FEAT-30) reflect only already-resolved outcomes, which are unaffected by a degradation condition that occurs before resolution.

## Consent and Disclosure

- **Card entry disclosure** -- Directly above the card entry element on FEAT-07.SPEC-001, a persistent line reads: "Your card details go straight to our payment processor -- Chairtime never sees or stores your card number." This is shown every time the screen is reached, not only on first use, since each deposit is paid fresh with no card kept on file (per BRIEF.md's Constraints).
- **What the payout destination receives** -- The disclosure line above also implicitly covers that the payment is destined for the Pro's own connected account; no separate consent prompt is needed for this, since Riley is already booking that specific Pro and the amount and recipient are evident from the booking context itself (service, price, and Pro name shown on the same screen).
- **What is never shared** -- The Booking's service name, appointment time, Riley's name, phone, email, and any booking note never leave the product through this integration; only the deposit amount, currency, an internal Booking reference, and the Pro's payout destination reference are sent, exactly as scoped in Data Exchanged. This boundary is stated in the disclosure line's plain wording ("Chairtime never sees or stores your card number") and is never contradicted elsewhere in the flow.
- **Pro-facing disclosure of the fee deduction** -- The first time Talia views her money list after her first captured deposit (FEAT-28), a one-time notice states: "Chairtime never takes a cut of your deposits. The only deduction you'll ever see is your payment processor's own card fee, shown on each transaction." This is FEAT-28's own display responsibility; this spec supplies the processor_fee value that notice and every transaction line reference.

## Edge Cases

- **The same capture-succeeded event is delivered twice** -- The second delivery reaches FEAT-07.SPEC-002, which finds an existing Deposit Transaction and takes no further action; this spec itself performs no state changes on receipt, so duplicate delivery has no direct effect here beyond the automation's own idempotency handling.
- **A capture event arrives for a Booking that has since left Pending Payment for an unrelated reason** -- Handled by FEAT-07.SPEC-002 and FEAT-07.SPEC-004's correctness guarantee, not silently dropped by this spec; this spec's own responsibility ends at reporting the event faithfully with the Booking reference it received.
- **A decline and a late capture-succeeded report both arrive for the same attempt (out-of-order delivery)** -- The capability's own outcome for a single charge request is authoritative and final once reported; a genuinely late success after an already-shown decline can only occur if the two reports describe two separate requests (Riley's retry after the decline), each carrying its own idempotency key per FEAT-07.SPEC-004, so they are never conflated into one ambiguous outcome.
- **The capability goes down mid-authorization, after the request was sent but before any result is received** -- No Deposit Transaction is created while the outcome is unknown; FEAT-07.SPEC-001 shows its Offline/Degraded state, and if the capability's result eventually does arrive (success or decline), FEAT-07.SPEC-004 reconciles it against the Booking's true state rather than assuming failure.
- **The captured amount reported by the capability does not exactly match the deposit_amount locked by FEAT-07.SPEC-003** -- Cannot occur under normal operation, since this spec always requests exactly the locked amount and the capability captures exactly what was requested or declines; if a mismatch were ever reported, FEAT-07.SPEC-002 would refuse to create the Deposit Transaction from a mismatched amount and the outcome would be escalated to FEAT-07.SPEC-004 as an unresolved attempt rather than recorded with an incorrect figure.
- **Payout routing confirmation is delayed after a successful capture** -- The capture outcome (Booking Confirmed, Deposit Transaction Captured) is not held pending on payout routing; Riley's confirmation is never delayed by a slow payout leg. Talia's money list (FEAT-28) shows the deposit as received once routing confirms, on the processor's own reporting timeline.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-001 (Deposit Payment) | Triggered by (inbound) | The Pay action initiates the authorization/capture request |
| FEAT-07.SPEC-001 (Deposit Payment) | Affects (outbound) | Processing, decline, and offline/degraded states surface this spec's reported outcomes and degradation behavior |
| FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules) | Triggered by (inbound) | Eligibility passing is the precondition for this spec ever sending a charge request |
| FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Triggers (outbound) | A reported successful capture fires that automation's create-and-confirm step |
| FEAT-07.SPEC-004 (Payment Outcome Consistency & Idempotency) | References (inbound) | Supplies the idempotency-key discipline this spec applies to every charge request, and governs reconciliation of any late or ambiguous response |
| FEAT-25.SPEC-004 -- within FEAT-25 (Booking & Revenue Insights) | Triggers (outbound) | A captured deposit fires the deposits-collected aggregate update (Inbound Events: Deposit outcome set) |
| FEAT-28 (Payout Account Connection & Payout Visibility) | Affects (outbound) | The routed deposit and its processor_fee are what that feature's money list displays |

## Analytics and Success Signals

- **deposit_charge_requested** (booking reference, deposit amount, currency) -- supports success-metrics.md: "Deposit Capture Rate"
- **deposit_charge_captured** (booking reference, processor_fee) -- supports success-metrics.md: "Deposit Capture Rate"
- **deposit_charge_declined** (booking reference, decline reason category) -- supports success-metrics.md: "Deposit Capture Rate"
- **deposit_payout_routing_confirmed** (booking reference) -- N/A -- no Stage 2 metric measures payout routing timing directly; retained so a delayed or failed routing leg is observable to the Pro's activity record (FEAT-16) and money list (FEAT-28), separately from the deposit capture itself.
- **deposit_capability_degraded** (condition: slow / down / rejected) -- N/A -- no Stage 2 metric measures degradation frequency; retained so the product's tolerance for capability trouble is observable, consistent with how Analytics sections are written for this feature's Screen spec.

## Acceptance Criteria

**FEAT-07.SPEC-005-AC-01:** Given Riley's Booking has passed FEAT-07.SPEC-003's eligibility check, when she taps Pay, then a charge request for exactly the locked deposit_amount and currency is sent to the payment-processing capability, carrying an idempotency key and the Booking reference.

**FEAT-07.SPEC-005-AC-02:** Given the capability successfully captures Riley's charge, when it reports the outcome, then this spec reports the captured amount, currency, and processor_fee to FEAT-07.SPEC-002 without creating the Deposit Transaction itself.

**FEAT-07.SPEC-005-AC-03:** Given the capability declines Riley's card, when it reports the decline, then FEAT-07.SPEC-001 shows the plain-language decline reason and a "Try a different card" action, and no Deposit Transaction is created.

**FEAT-07.SPEC-005-AC-04:** Given a captured deposit, when payout routing completes, then the amount is routed to Talia's connected Payout Account with a Chairtime fee of zero, and the processor's own fee is the only deduction reported.

**FEAT-07.SPEC-005-AC-05:** Given Riley is on the Deposit Payment screen for the first time this session, when she reaches the card entry element, then she sees "Your card details go straight to our payment processor -- Chairtime never sees or stores your card number." above it.

**FEAT-07.SPEC-005-AC-06:** Given a charge request is in flight, when more than the product's brief processing threshold passes without a result, then the status region adds "Still working -- this is taking longer than usual." while the Pay button remains disabled.

**FEAT-07.SPEC-005-AC-07:** Given the payment-processing capability is unavailable when Riley reaches the Pay step, when she views the screen, then the Pay button is disabled with "We can't take payments right now. Please try again in a few minutes -- your held time is still counting down." and her checkout hold continues counting down normally.

**FEAT-07.SPEC-005-AC-08:** Given the capability is down and Riley's checkout hold expires while she waits, when the expiry is reported, then FEAT-07.SPEC-001 shows its Hold Expired state exactly as it would for any other cause of expiry.

**FEAT-07.SPEC-005-AC-09:** Given the capability rejects a charge with a specific reason, when the rejection is reported, then Riley sees that specific plain-language reason (for example, "This card has expired. Try a different card."), never a raw processor code.

**FEAT-07.SPEC-005-AC-10:** Given a capture-succeeded event is delivered twice for the same Booking, when the second delivery reaches FEAT-07.SPEC-002, then no second Deposit Transaction is created.

**FEAT-07.SPEC-005-AC-11:** Given the capability goes down after a charge request was sent but before any result is received, when Riley's screen detects this, then no Deposit Transaction is created while the outcome remains unknown, and the Offline/Degraded state per FEAT-07.SPEC-001 is shown.

**FEAT-07.SPEC-005-AC-12:** Given a capture outcome is confirmed, when Riley's confirmation is shown, then it is never delayed waiting for the separate payout-routing confirmation to Talia's account.

**FEAT-07.SPEC-005-AC-13:** Given Talia views her money list after her first-ever captured deposit, when the notice appears, then it reads "Chairtime never takes a cut of your deposits. The only deduction you'll ever see is your payment processor's own card fee, shown on each transaction."

**FEAT-07.SPEC-005-AC-14:** Given a charge request for Riley's Booking, when it is sent to the capability, then it carries only the deposit amount, currency, an internal Booking reference, and the Pro's payout destination reference -- never Riley's name, phone, email, or the service and appointment details.

**FEAT-07.SPEC-005-AC-15:** Given a Booking's payment attempt has an out-of-order pair of reports (a decline followed by a late success, or the reverse) that in fact describe two separate retried attempts, when both are processed, then each is resolved against its own idempotency key and the two are never conflated into a single ambiguous outcome.

**FEAT-07.SPEC-005-AC-16:** Given the capability reports a captured amount that does not match the locked deposit_amount, when FEAT-07.SPEC-002 checks it, then no Deposit Transaction is created from the mismatched figure and the attempt is escalated to FEAT-07.SPEC-004 as unresolved.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 3 (one screen x three conditions) | 3 |
| Consent and Disclosure | 4 | 4 |
| Edge Cases | 6 | 6 |
