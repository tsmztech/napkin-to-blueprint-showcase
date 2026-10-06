---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-07.SPEC-003
spec_name: Deposit Amount & Eligibility Rules
spec_slug: deposit-amount-eligibility-rules
parent_feature: FEAT-07
parent_feature_name: Deposit Payment at Booking
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 17
acceptance_criteria_count: 16
---

# Logic/Rule Spec: Deposit Amount & Eligibility Rules

## Overview

**Name:** Deposit Amount & Eligibility Rules
**ID:** FEAT-07.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs how the deposit amount is computed exactly once from the service's rule, that it can never be altered by the client, and the preconditions that must hold before any charge is attempted.
**Parent Feature:** FEAT-07 -- Deposit Payment at Booking
**Governed Entity:** Booking -- the deposit-computation and eligibility-gating fields only (price_agreed, deposit_amount, currency as locked from the Pro Account, and the Pending Payment state precondition). All other Booking fields (client, policy_version, state's later transitions, attendance_reply, balance_due, source, cancellation/reschedule timestamps) are owned by other features' Logic/Rule specs, per the dependency map's Entity-Lifecycle ownership.

## Scope and Non-Goals

**In Scope:**
- Computing the deposit amount exactly once from the Service's deposit_rule and price, in the Pro's account currency
- The precondition checks that must all hold before any charge attempt is authorized (Booking still Pending Payment, currency locked and matching, payout account Active, no existing Deposit Transaction for the Booking)
- Preventing the client from altering the computed amount by any means
- The one-charge-per-booking rule at the point a charge is requested (working with FEAT-07.SPEC-004's idempotency guarantee for the point a charge is captured)

**Non-Goals:**
- Creating the Deposit Transaction and confirming the Booking on a successful charge -- owned by FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation); this spec only gates whether a charge attempt may proceed
- Guaranteeing correctness across a dropped connection or duplicate delivery of a charge outcome -- owned by FEAT-07.SPEC-004 (Payment Outcome Consistency & Idempotency)
- Defining the Service's deposit_rule field itself (fixed amount vs. percentage, its own bounds) -- owned by FEAT-01.SPEC-004 (Service Field & Deposit Rule Validation); this spec only consumes the already-validated rule to compute one booking's deposit
- Authorizing or capturing the card charge itself -- owned by FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing); this spec defines only whether a charge may be requested, never how the charge is processed

## Governed Entity

**Entity:** Booking (deposit-computation and eligibility-gating fields)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| price_agreed | number | The service price agreed and fixed at the moment of booking, in the Pro's account currency |
| deposit_amount | number | The deposit computed once from the Service's deposit_rule and price_agreed, fixed at the moment of booking |
| currency | derived (from Pro Account, at the moment of the account's first-ever deposit) | The currency both price_agreed and deposit_amount are expressed in |
| state | enum | Must be Pending Payment for a charge attempt to be eligible; this spec reads it as a precondition and never writes it (FEAT-07.SPEC-002 owns the Pending Payment -> Confirmed transition) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-07.SPEC-001 | Deposit Payment | Reads the locked deposit_amount for display on screen load; requests an eligibility check from this spec on every Pay tap, including retries |
| FEAT-07.SPEC-002 | Deposit Capture & Booking Confirmation | Confirms, before creating the Deposit Transaction, that the captured amount matches the deposit_amount this spec locked and that the eligibility precondition held at charge time |
| FEAT-07.SPEC-005 | Card Deposit Charge & Payout Routing | Requests this spec's eligibility check immediately before submitting an authorization/capture request to the payment-processing capability |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| deposit_amount | Computed once, exactly, from the Service's deposit_rule (fixed amount, or price_agreed x percentage / 100, rounded to the nearest currency unit) at the moment the Booking is created (FEAT-05.SPEC-006, from the amount FEAT-05.SPEC-009 computes); never recomputed afterward and never accepted as client-supplied input | Always | On Booking creation, before this screen is ever reached | N/A -- this is a system computation with no client-facing input to reject; the client is never shown an editable amount field | Yes |
| deposit_amount | Must be at least the minimum chargeable amount (platform parameter: `minimum-chargeable-deposit`) | Always -- already guaranteed by FEAT-01.SPEC-004 at the Service level, re-confirmed here as a precondition since a Service's price or rule could theoretically change between service selection and payment (XBR-04 forbids this for a Booking already created, but the check remains defense-in-depth) | On every eligibility check | "This booking's deposit could not be processed. Please start a new booking." (shown only in the theoretical case this precondition ever fails; FEAT-01.SPEC-004 makes it unreachable in normal operation) | Yes |
| currency | Must equal the Pro Account's locked currency (XBR-25; lock determined by FEAT-27.SPEC-008, which this check enforces) | Always | On every eligibility check | "This booking's currency no longer matches the Pro's account. Please start a new booking." | Yes |
| price_agreed | No validation beyond data type in this spec -- price_agreed is fixed at Booking creation by FEAT-05 and read here only to display the balance-due figure; its own validation is owned by FEAT-01.SPEC-004 (Service pricing) and FEAT-05.SPEC-009 (price lock at booking time) | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Amount immutability | deposit_amount | The value read at charge time must be byte-identical to the value computed at Booking creation; no interaction on FEAT-07.SPEC-001 (Deposit Payment) or FEAT-07.SPEC-005 accepts or applies an override, discount, or client-entered amount | N/A -- there is no field through which an override could be entered, so this is enforced by omission rather than by rejecting an input |
| Eligibility precondition set | state, currency, deposit_amount, Payout Account.status, Deposit Transaction (existence check) | All of the following must hold simultaneously before a charge attempt is authorized: the Booking is Pending Payment; currency matches the Pro Account's locked currency; deposit_amount meets the minimum-chargeable-deposit floor; the Pro's Payout Account status is Active (XBR-06); and no Deposit Transaction already exists for this Booking (one-charge-per-booking) | See the per-condition messages in Authorization Rules and Field Validation Rules below; the eligibility check as a whole fails closed -- if any condition is unmet, no charge is requested |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Request a deposit charge for a Booking | The Client (Riley) | Only for her own in-progress Booking, and only while the eligibility precondition set (above) holds | If the Booking is not Pending Payment: "This time is no longer available." and she is returned to FEAT-05.SPEC-002 (Slot Selection), per XBR-01. If the currency check fails: "This booking's currency no longer matches the Pro's account. Please start a new booking." If a Deposit Transaction already exists for the Booking (a second attempt after an already-successful charge, e.g. from a stale page): no new charge is requested; the screen is shown the Success state directly, per FEAT-07.SPEC-004. If the Payout Account is not Active: "This booking can't be paid right now. Please try again shortly, or contact {Pro's display name}." (per XBR-06 -- this state should not normally be reachable for a live booking link, since FEAT-05.SPEC-008 gates the link itself on an Active payout account, but the check remains defense-in-depth against a payout account being actioned mid-checkout) |
| View the computed deposit amount | The Client (Riley) | Only for her own in-progress Booking | -- |
| Alter the computed deposit amount | The Client (Riley) | Never | No control exists anywhere in the product for the client to alter deposit_amount; the field is display-only everywhere it appears |
| View the computed deposit amount and eligibility state | The Pro (Talia) | Full, but only after the Booking is Confirmed and visible on her own schedule/booking-management surfaces (FEAT-12, FEAT-30) -- never during the client's in-progress checkout, which she has no visibility into | -- |
| View transaction status resulting from an eligibility check | Platform Operator (Support) | View-only, on FEAT-16/FEAT-19, never card data or the in-progress checkout itself | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| deposit_amount | If Service.deposit_rule is a fixed amount: deposit_amount = the fixed amount. If Service.deposit_rule is a percentage: deposit_amount = round(price_agreed x percentage / 100) to the nearest currency unit | On Booking creation only (FEAT-05.SPEC-006, from the amount FEAT-05.SPEC-009 computes); read-only thereafter | No -- not by the Client, not by the Pro, not by Support. This is a system-computed value fixed for the life of the Booking (XBR-04) |
| currency | The Pro Account's currency, locked at that account's first-ever successful deposit (XBR-25) | Read at Booking creation and re-confirmed at every eligibility check | No |

## Business Rules

- XBR-05: The deposit is computed once, exactly, from the Service's rule in the Pro's account currency, cannot be altered by the client, and is charged once per booking; card data is never held by the product.
- XBR-06: No deposit can be taken unless the Pro's payout account is Active.
- XBR-25: Currency is locked to the Pro Account's currency once the account's first deposit is taken; every eligibility check re-confirms the Booking's currency still matches. Enforcing FEAT-27.SPEC-008 (Currency Lock Rule): this spec consumes that spec's lock determination as the charge precondition and never re-derives or redefines it.
- XBR-04: Service edits and archiving apply to future bookings only -- a confirmed or in-progress Booking keeps the price and deposit agreed at booking, so a Service-side price or rule change after Booking creation never changes this Booking's already-locked deposit_amount.
- One-charge-per-booking is enforced at two points for defense-in-depth: this spec blocks a second charge *request* once a Deposit Transaction exists (checked before the request is even sent), and FEAT-07.SPEC-004 guarantees correctness of the charge *outcome* even if two requests were somehow both sent (e.g., a race between two rapid taps or two tabs).
- The eligibility check runs identically on the first attempt and on every retry after a decline -- there is no relaxed re-check path for retries.

## Edge Cases

- **Service's deposit_rule changes after this Booking was created but before payment** -- No effect: XBR-04 fixes price_agreed and deposit_amount at Booking creation; this spec never re-reads the Service record to recompute after that point.
- **Percentage computation lands on a fractional currency unit** -- Rounded to the nearest whole currency unit at the moment of the one-time computation (e.g., a percentage yielding 24.5 rounds to 25); the rounded value is what is locked and never re-rounded afterward.
- **Payout Account transitions from Active to Action Required between screen load and Pay tap** -- The eligibility check re-runs at the Pay tap (not only at screen load) and catches the change; the charge is not requested and Riley sees "This booking can't be paid right now..." while Talia separately sees the Action Required banner on her own dashboard (FEAT-12).
- **Two rapid attempts both reach the eligibility check before either creates a Deposit Transaction** -- Both may pass eligibility (no Deposit Transaction exists yet for either), but only one may actually be captured as the successful charge; the second's capture attempt is resolved by FEAT-07.SPEC-004's idempotency guarantee, never by this spec, which only gates the request, not the outcome.
- **A second Pay attempt is made after an already-successful charge (e.g., a stale reloaded page)** -- The eligibility check finds an existing Deposit Transaction for the Booking and denies the request; the screen shows the Success state directly rather than requesting a second charge.
- **Currency stored on the Booking at creation differs from the Pro Account's currency due to an account-level change between Booking creation and payment** -- Cannot occur under XBR-25 (currency locks after the account's first deposit and never changes thereafter for that account); this edge case is therefore structurally prevented rather than handled at runtime.

## Acceptance Criteria

**FEAT-07.SPEC-003-AC-01:** Given a Service with a fixed-amount deposit_rule, when Riley's Booking is created for that service, then deposit_amount is set to exactly that fixed amount.

**FEAT-07.SPEC-003-AC-02:** Given a Service with a percentage deposit_rule and a price_agreed of {P}, when Riley's Booking is created, then deposit_amount is set to round(P x percentage / 100) to the nearest currency unit.

**FEAT-07.SPEC-003-AC-03:** Given Riley's Booking has a locked deposit_amount, when she views the Deposit Payment screen, then no control anywhere lets her change that amount.

**FEAT-07.SPEC-003-AC-04:** Given Riley's Booking is Pending Payment, currency matches, the Pro's payout account is Active, and no Deposit Transaction exists for it, when she taps Pay, then the eligibility check passes and a charge is requested.

**FEAT-07.SPEC-003-AC-05:** Given Riley's Booking is no longer Pending Payment (for example, its checkout hold has expired and released the slot), when she taps Pay, then the eligibility check fails and she sees "This time is no longer available." and is returned to FEAT-05.SPEC-002.

**FEAT-07.SPEC-003-AC-06:** Given the Pro's payout account is not Active at the moment Riley attempts to pay, when the eligibility check runs, then no charge is requested and she sees "This booking can't be paid right now. Please try again shortly, or contact {Pro's display name}."

**FEAT-07.SPEC-003-AC-07:** Given a Deposit Transaction already exists for Riley's Booking (a prior attempt already succeeded), when she taps Pay again from a stale page, then no new charge is requested and she is shown the Success state directly.

**FEAT-07.SPEC-003-AC-08:** Given the Booking's currency no longer matches the Pro Account's locked currency, when the eligibility check runs, then no charge is requested and Riley sees "This booking's currency no longer matches the Pro's account. Please start a new booking."

**FEAT-07.SPEC-003-AC-09:** Given a Service's price or deposit_rule is edited by Talia after Riley's Booking was already created, when Riley proceeds to pay, then her Booking's deposit_amount is unaffected by the edit, per XBR-04.

**FEAT-07.SPEC-003-AC-10:** Given Riley (the Client) is viewing her own in-progress Booking, when she looks at the deposit amount, then she sees exactly the locked value with no edit affordance.

**FEAT-07.SPEC-003-AC-11:** Given Talia (the Pro) has not yet had this Booking's deposit captured, when she looks at her own schedule or booking-management surfaces, then she sees no visibility into Riley's in-progress checkout state -- only the eventual Confirmed/paid status once capture succeeds.

**FEAT-07.SPEC-003-AC-12:** Given Platform Operator (Support) opens a Pro's account after a help request, when they look at a booking's payment status, then they see the transaction status only, via FEAT-16/FEAT-19, never the in-progress checkout screen or card data.

**FEAT-07.SPEC-003-AC-13:** Given a percentage computation yields a fractional currency unit of exactly .5, when the one-time computation runs, then the result rounds to the nearest whole currency unit and that rounded value is what is locked.

**FEAT-07.SPEC-003-AC-14:** Given the Pro's payout account transitions to Action Required between Riley's screen load and her Pay tap, when she taps Pay, then the eligibility check re-runs and denies the charge with the same message as AC-06, rather than relying on the state at screen load.

**FEAT-07.SPEC-003-AC-15:** Given Riley retries payment after an earlier decline, when the retry's eligibility check runs, then it applies the identical full precondition set as the first attempt -- no relaxed re-check path exists for retries.

**FEAT-07.SPEC-003-AC-16:** Given a Booking's deposit_amount would fall below platform parameter: `minimum-chargeable-deposit` due to a theoretical precondition failure, when the eligibility check runs, then no charge is requested and Riley sees "This booking's deposit could not be processed. Please start a new booking."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
