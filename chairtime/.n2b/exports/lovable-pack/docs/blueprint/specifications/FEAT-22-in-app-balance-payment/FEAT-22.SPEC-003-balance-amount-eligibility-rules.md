---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-22.SPEC-003
spec_name: Balance Amount & Eligibility Rules
spec_slug: balance-amount-eligibility-rules
parent_feature: FEAT-22
parent_feature_name: In-App Balance Payment
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 22
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Balance Amount & Eligibility Rules

## Overview

**Name:** Balance Amount & Eligibility Rules
**ID:** FEAT-22.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs how the balance amount is derived once from the service price and the deposit already paid, that it can never be altered by the client, and the preconditions that must hold before any in-app balance charge is attempted.
**Parent Feature:** FEAT-22 -- In-App Balance Payment
**Governed Entity:** Balance Payment (the amount field, as computed) jointly Booking (the balance-payability precondition fields only: the balance-payable slice of state, price_agreed, and the derived balance_due). All other Booking fields and state transitions are owned by other features' Logic/Rule specs, per the dependency map's Entity-Lifecycle ownership; the tip field on Balance Payment is owned by FEAT-23 (Later), not this spec.

## Scope and Non-Goals

**In Scope:**
- Deriving the balance amount exactly once from the Booking's price_agreed and the already-captured Deposit Transaction amount, in the Pro's account currency
- The precondition checks that must all hold before any balance charge attempt is authorized (Booking still in a payable state, balance_due greater than zero, currency locked and matching, payout account Active, no existing Succeeded Balance Payment for the Booking)
- Preventing the client from altering the derived amount by any means
- The one-succeeded-payment-per-booking rule at the point a charge is requested (working with FEAT-22.SPEC-004's idempotency and contention guarantees for the point a charge is captured or contested by a concurrent cancellation)

**Non-Goals:**
- Creating the Balance Payment record and updating the Booking's balance-due status on a successful charge -- owned by FEAT-22.SPEC-002 (Balance Capture & Booking Status Update); this spec only gates whether a charge attempt may proceed
- Guaranteeing correctness across a dropped connection, duplicate delivery, or a race with a Pro cancellation -- owned by FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention)
- Authorizing or capturing the card charge itself, or executing the refund when a paid balance is cancelled -- owned by FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund); this spec defines only whether a charge may be requested, never how the charge or refund is processed
- Computing or validating the original deposit amount -- owned by FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules); this spec only reads the already-locked Deposit Transaction amount as an input to its own derivation

## Governed Entity

**Entity:** Balance Payment (amount) and Booking (balance-payability precondition fields), governed jointly
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| amount (Balance Payment) | number | The balance derived once from price_agreed minus the Deposit Transaction amount, fixed at the moment a charge succeeds; not client-alterable |
| tip (Balance Payment) | number, optional | Not governed by this spec -- owned entirely by FEAT-23 (Tipping at Checkout, Later); this spec's derivation and eligibility checks never read or depend on it |
| state (Balance Payment) | enum | Attempted \| Succeeded \| Failed \| Refunded. This spec reads only whether a Succeeded record already exists, as an eligibility precondition; it never writes this field (FEAT-22.SPEC-002 owns the transition to Succeeded) |
| price_agreed (Booking) | number | The service price agreed and fixed at the moment of booking, in the Pro's account currency (read-only here; owned by FEAT-05/FEAT-01) |
| balance_due (Booking) | derived | price_agreed minus deposit_amount minus any in-app balance payment; this spec derives the amount to attempt a charge for from this same formula, before any Balance Payment yet exists |
| state (Booking, balance-payable slice) | enum | Must be Confirmed or Awaiting Outcome for a charge attempt to be eligible; this spec reads it as a precondition and never writes it |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-22.SPEC-001 | Balance Payment | Reads the locked balance amount for display on screen load; requests an eligibility check from this spec on every Pay tap, including retries |
| FEAT-22.SPEC-002 | Balance Capture & Booking Status Update | Confirms, before creating the Balance Payment, that the captured amount matches the amount this spec locked and that the eligibility precondition held at charge time |
| FEAT-22.SPEC-005 | Balance Charge, Payout Routing & Refund | Requests this spec's eligibility check immediately before submitting an authorization/capture request to the payment-processing capability |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| amount (Balance Payment) | Derived once, exactly, as price_agreed minus the Deposit Transaction's captured amount, at the moment a charge attempt is requested; never accepted as client-supplied input | Always | On every eligibility check, immediately before a charge is requested | N/A -- this is a system computation with no client-facing input to reject; Riley is never shown an editable amount field | Yes |
| amount (Balance Payment) | Must be greater than zero | Always -- a Booking whose deposit already equals its price has nothing left to pay in-app | On every eligibility check | "There's nothing left to pay on this booking." | Yes |
| amount (Balance Payment) | Must be at least the minimum chargeable amount (platform parameter: `minimum-chargeable-deposit` -- reused here as the platform's single "smallest amount a card payment can be taken for" floor, since the underlying value is not specific to deposits; see Business Rules) | Always | On every eligibility check | "This balance is too small to charge by card right now. It can still be paid in person at the appointment." | Yes |
| currency | Must equal the Pro Account's locked currency (XBR-25) | Always | On every eligibility check | "This booking's currency no longer matches the Pro's account. Please try again later or pay in person." | Yes |
| state (Booking) | Must be Confirmed or Awaiting Outcome | Always | On every eligibility check | "This time is no longer available." (if the Booking has entered a state incompatible with payment, e.g. Cancelled or Completed) | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Amount immutability | amount (Balance Payment) | The value derived at charge time must be byte-identical to price_agreed minus the Deposit Transaction amount; no interaction on FEAT-22.SPEC-001 (Balance Payment) or FEAT-22.SPEC-005 accepts or applies an override, discount, or client-entered amount | N/A -- there is no field through which an override could be entered, so this is enforced by omission rather than by rejecting an input |
| Eligibility precondition set | state (Booking), currency, balance_due, Payout Account.status, Balance Payment (existence check) | All of the following must hold simultaneously before a charge attempt is authorized: the Booking is Confirmed or Awaiting Outcome; currency matches the Pro Account's locked currency; balance_due is greater than zero and meets the minimum-chargeable floor; the Pro's Payout Account status is Active (XBR-06's payment-eligibility principle applied to the balance); and no Balance Payment with state Succeeded already exists for this Booking (one-succeeded-payment-per-booking) | See the per-condition messages above; the eligibility check as a whole fails closed -- if any condition is unmet, no charge is requested |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Request a balance charge for a Booking | The Client (Riley) | Only for her own confirmed Booking, and only while the eligibility precondition set (above) holds | If the Booking is not Confirmed or Awaiting Outcome: "This time is no longer available." If balance_due is zero: "There's nothing left to pay on this booking." If the currency check fails: "This booking's currency no longer matches the Pro's account. Please try again later or pay in person." If a Succeeded Balance Payment already exists (a second attempt after an already-successful charge, e.g. from a stale page): no new charge is requested; the screen is shown the Success state directly, per FEAT-22.SPEC-004. If the Payout Account is not Active: "This booking can't be paid right now. Please try again shortly, or pay {Pro's display name} in person." |
| View the derived balance amount and deposit-paid figure | The Client (Riley) | Only for her own booking | -- |
| Alter the derived balance amount | The Client (Riley) | Never | No control exists anywhere in the product for the client to alter the amount; the field is display-only everywhere it appears |
| View the resulting paid/unpaid status | The Pro (Talia) | Full, on her own schedule/booking-management surfaces (FEAT-12, FEAT-30) once the outcome resolves -- never during Riley's in-progress attempt, which she has no visibility into | -- |
| View balance status for support purposes | Platform Operator (Support) | View-only, on FEAT-16/FEAT-19, never card data or Riley's in-progress screen | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| amount (Balance Payment) | price_agreed minus the Deposit Transaction's captured amount minus any prior Balance Payment amount for this Booking (structurally always zero prior, since at most one Succeeded Balance Payment ever exists per Booking) | Derived fresh on every eligibility check, and locked into the Balance Payment record only at the moment a charge succeeds (FEAT-22.SPEC-002) | No -- not by the Client, not by the Pro, not by Support |
| currency | The Pro Account's currency, locked at that account's first-ever successful deposit (XBR-25) | Read at every eligibility check | No |

## Business Rules

- XBR-23: Balance due equals service price minus deposit minus any in-app balance payment; a paid balance is never forfeited and is refunded in full if either party cancels.
- XBR-07: Chairtime's fee on the balance is always zero, exactly as for deposits and tips; the routing of the captured amount is FEAT-22.SPEC-005's own responsibility, not a rule this spec computes.
- XBR-06's underlying principle (no deposit can be taken unless the Pro's payout account is Active) is applied identically to the balance: no in-app balance charge is requested unless the Payout Account is Active, since the same connected account receives both.
- XBR-25: Currency is locked to the Pro Account's currency once the account's first deposit is taken; every eligibility check re-confirms the Booking's currency still matches.
- The marker platform parameter: `minimum-chargeable-deposit` is reused verbatim for the balance's minimum-chargeable floor rather than minting a second marker: both represent the same real-world constraint (the smallest amount the payment-processing capability can charge a card for), and the parameter's name reflects where it was first defined, not a deposit-only scope. Reconciliation (Pass D) confirmed this reuse: the registry carries one row for this slug covering both floors.
- One-succeeded-payment-per-booking is enforced at two points for defense-in-depth: this spec blocks a second charge *request* once a Succeeded Balance Payment exists (checked before the request is even sent), and FEAT-22.SPEC-004 guarantees correctness of the charge *outcome* even if two requests were somehow both sent.
- The eligibility check runs identically on the first attempt and on every retry after a decline -- there is no relaxed re-check path for retries.
- Unlike the deposit's checkout-hold-scoped eligibility (FEAT-07.SPEC-003), this spec's eligibility check has no time-limited hold: Riley may return to pay the balance "at any point before or at the appointment" (product-features.md Key Capabilities), so the check simply re-runs fresh on every attempt against the Booking's current state rather than against a slot hold.

## Edge Cases

- **Riley's Booking's deposit already equals its full price (a service whose deposit_rule is 100%)** -- balance_due is zero; FEAT-06.SPEC-004 (Booking Detail via Manage Link) never shows its "Pay Balance" button for such a booking in the first place, and if this screen is reached anyway (a stale link), the eligibility check fails with "There's nothing left to pay on this booking."
- **Percentage or fixed deposit computation left a very small remaining balance, below the minimum-chargeable floor** -- The eligibility check fails with "This balance is too small to charge by card right now. It can still be paid in person at the appointment." -- the in-person default remains available exactly as it always is.
- **Payout Account transitions from Active to Action Required between screen load and Pay tap** -- The eligibility check re-runs at the Pay tap (not only at screen load) and catches the change; the charge is not requested and Riley sees "This booking can't be paid right now..." while Talia separately sees the Action Required banner on her own dashboard (FEAT-12).
- **Two rapid attempts both reach the eligibility check before either creates a Balance Payment** -- Both may pass eligibility (no Succeeded Balance Payment exists yet for either), but only one may actually be captured as the successful charge; the second's capture attempt is resolved by FEAT-22.SPEC-004's idempotency guarantee, never by this spec, which only gates the request, not the outcome.
- **A second Pay attempt is made after an already-successful charge (e.g., a stale reloaded page)** -- The eligibility check finds an existing Succeeded Balance Payment and denies the request; the screen shows the Success state directly rather than requesting a second charge.
- **The Booking's balance-payable state changes to Cancelled between screen load and Pay tap because Talia commits a cancellation** -- The eligibility check re-runs at the Pay tap and fails on the state precondition; no charge is requested. This is the eligibility-side half of the contention FEAT-22.SPEC-004 fully governs for the case where the cancellation and the charge race even more tightly than a screen-load-to-tap gap.

## Acceptance Criteria

**FEAT-22.SPEC-003-AC-01:** Given a Booking with price_agreed of {P} and a captured Deposit Transaction of {D}, when Riley's eligibility check runs, then the balance amount is derived as exactly {P minus D}.

**FEAT-22.SPEC-003-AC-02:** Given Riley's Booking has a derived balance amount, when she views the Balance Payment screen, then no control anywhere lets her change that amount.

**FEAT-22.SPEC-003-AC-03:** Given Riley's Booking is Confirmed, currency matches, the Pro's payout account is Active, balance_due is greater than zero and above the minimum-chargeable floor, and no Succeeded Balance Payment exists for it, when she taps Pay, then the eligibility check passes and a charge is requested.

**FEAT-22.SPEC-003-AC-04:** Given Riley's Booking's balance_due is zero because the deposit already equals the full price, when she reaches this screen and the eligibility check runs, then no charge is requested and she sees "There's nothing left to pay on this booking."

**FEAT-22.SPEC-003-AC-05:** Given Riley's Booking's derived balance amount falls below platform parameter: `minimum-chargeable-deposit`, when the eligibility check runs, then no charge is requested and she sees "This balance is too small to charge by card right now. It can still be paid in person at the appointment."

**FEAT-22.SPEC-003-AC-06:** Given the Pro's payout account is not Active at the moment Riley attempts to pay, when the eligibility check runs, then no charge is requested and she sees "This booking can't be paid right now. Please try again shortly, or pay {Pro's display name} in person."

**FEAT-22.SPEC-003-AC-07:** Given a Succeeded Balance Payment already exists for Riley's Booking, when she taps Pay again from a stale page, then no new charge is requested and she is shown the Success state directly.

**FEAT-22.SPEC-003-AC-08:** Given the Booking's currency no longer matches the Pro Account's locked currency, when the eligibility check runs, then no charge is requested and Riley sees "This booking's currency no longer matches the Pro's account. Please try again later or pay in person."

**FEAT-22.SPEC-003-AC-09:** Given Riley's Booking is no longer Confirmed or Awaiting Outcome (for example, it was cancelled), when she taps Pay, then the eligibility check fails and she sees "This time is no longer available."

**FEAT-22.SPEC-003-AC-10:** Given Riley (the Client) is viewing her own booking, when she looks at the balance amount, then she sees exactly the derived value with no edit affordance.

**FEAT-22.SPEC-003-AC-11:** Given Talia (the Pro) has not yet had this Booking's balance captured, when she looks at her own schedule or booking-management surfaces, then she sees no visibility into Riley's in-progress balance-payment attempt -- only the eventual fully-paid status once capture succeeds.

**FEAT-22.SPEC-003-AC-12:** Given Platform Operator (Support) opens a Pro's account after a help request, when they look at a booking's balance status, then they see the transaction status only, via FEAT-16/FEAT-19, never the in-progress payment screen or card data.

**FEAT-22.SPEC-003-AC-13:** Given the Pro's payout account transitions to Action Required between Riley's screen load and her Pay tap, when she taps Pay, then the eligibility check re-runs and denies the charge with the same message as AC-06, rather than relying on the state at screen load.

**FEAT-22.SPEC-003-AC-14:** Given Riley retries payment after an earlier decline, when the retry's eligibility check runs, then it applies the identical full precondition set as the first attempt -- no relaxed re-check path exists for retries.

**FEAT-22.SPEC-003-AC-15:** Given Riley opens the Balance Payment screen for a booking and then Talia commits a cancellation before Riley taps Pay, when the eligibility check runs at the Pay tap, then it fails on the Booking-state precondition and no charge is requested.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 8 | 8 |
| Edge Cases | 6 | 6 |
