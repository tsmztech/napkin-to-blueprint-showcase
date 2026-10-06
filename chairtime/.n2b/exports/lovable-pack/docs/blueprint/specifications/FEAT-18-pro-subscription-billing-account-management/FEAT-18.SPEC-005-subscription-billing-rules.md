---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-18.SPEC-005
spec_name: Subscription Billing Rules
spec_slug: subscription-billing-rules
parent_feature: FEAT-18
parent_feature_name: Pro Subscription Billing & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 29
acceptance_criteria_count: 19
---

# Logic/Rule Spec: Subscription Billing Rules

## Overview

**Name:** Subscription Billing Rules
**ID:** FEAT-18.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs the single price tier, the 7-day grace threshold (platform parameter: `subscription-payment-failure-grace-period-days`), cancellation-at-period-end timing, the 30-day price-change notice rule (platform parameter: `subscription-price-change-notice-days`), and contention/authority resolution between the Pro and the payment-processing capability for the Subscription entity.
**Parent Feature:** FEAT-18 -- Pro Subscription Billing & Account Management
**Governed Entity:** Subscription

## Scope and Non-Goals

**In Scope:**
- Field validation for every Subscription field
- The single-tier price rule (no plan selection)
- The 7-day payment-failure grace threshold
- Cancellation-at-period-end timing
- The 30-day price-change notice rule (platform parameter: `subscription-price-change-notice-days`)
- Authorization rules for every action on the Subscription record, per role
- Contention/authority resolution between the Pro and the payment-processing capability

**Non-Goals:**
- The renewal charge mechanics and grace-clock processing itself -- owned by FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing); this spec defines the threshold values and timing rules that automation applies
- The Pro Account pause mechanics -- owned by FEAT-18.SPEC-004 (Subscription-Lapse Account Pause Trigger); this spec governs only the Subscription entity itself, not the Pro Account
- The screens' layout and interaction mechanics -- owned by FEAT-18.SPEC-001 and FEAT-18.SPEC-002; this spec is their shared rule source, referenced by ID rather than duplicated
- Support acting on a subscription -- excluded per scope-boundaries.md SC-05: Platform Operator (Support) has view-only access and can never edit, cancel, or update payment details on the Pro's behalf

## Governed Entity

**Entity:** Subscription
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| status | enum | Active \| Payment Failed (7-day grace) \| Cancelled (active to period end) |
| billing_cycle | derived | The recurring monthly interval this Subscription bills on |
| next_billing_date | date | The date the next renewal charge is due |
| payment_method_reference | text | A reference to the card on file; the payment-processing capability holds the actual card data |
| price | number | The single all-inclusive subscription price (platform parameter: `subscription-price`) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-18.SPEC-001 | Subscribe Screen | On submit, applying the single-tier price; authorization on screen entry |
| FEAT-18.SPEC-002 | Billing & Subscription Management Screen | On cancel confirmation and payment-method update; authorization on screen entry and on each action |
| FEAT-18.SPEC-003 | Subscription Renewal & Payment-Failure Processing | On each renewal cycle, applying the grace threshold; on a price change's effective date, writing the new value of platform parameter: `subscription-price` into the Subscription's price field |
| FEAT-18.SPEC-004 | Subscription-Lapse Account Pause Trigger | Reads the grace threshold's expiry to decide when to pause |
| FEAT-18.SPEC-006 | Subscription Billing Integration | Authority resolution when the processor's outcome and the Pro's local view disagree |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| status | Must be one of: Active, Payment Failed (7-day grace), Cancelled (active to period end) | Always | On every write | No validation beyond data type -- status transitions are governed by Business Rules below, not by field-level format checks | No |
| billing_cycle | No validation beyond data type -- derived from next_billing_date and never directly entered | Always | -- | -- | -- |
| next_billing_date | Must be a valid future date at the time it is set | On write (create, renewal advance) | On write | "Could not schedule the next billing date. Please try again." | Yes |
| payment_method_reference | Must be a reference confirmed by the payment-processing capability; never a raw card value | Always | On write (create, payment-method update) | "This card could not be confirmed. Please check your details and try again." | Yes |
| price | Must equal the single platform-set tier (platform parameter: `subscription-price`) | Always | On write | No validation beyond data type -- there is no Pro-facing price input to reject; the value is set by the product, never entered | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Grace status requires a failure basis | status, next_billing_date | status can only be Payment Failed (7-day grace) following a recorded renewal failure against the existing next_billing_date -- it cannot be set directly by any user action | N/A -- this is a system-only transition; no user-facing input can trigger it directly, so no error message applies |
| Cancellation preserves the paid period | status, next_billing_date | When status is set to Cancelled (active to period end), next_billing_date is not advanced further -- it marks the date the account pauses, not a future renewal | N/A -- enforced structurally by FEAT-18.SPEC-006's cancellation processing, not by user input validation |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create Subscription (first subscribe) | The Pro | Only once per Pro Account -- a Pro Account already holding an active or cancelled-pending Subscription cannot create a second one | The Subscribe Screen (FEAT-18.SPEC-001) is not reached a second time; onboarding routes a Pro who already has a Subscription straight past the subscription step |
| Create Subscription | Platform Operator (Support) | Never | No create control exists in Support's view; Support only ever views an existing Subscription, never originates one |
| Create Subscription | The Client | Never | Clients have no subscription-related capability at all, per the Access Matrix (Subscription & Billing = None); no client-facing screen exposes this action |
| View Subscription (status, next billing date, price) | The Pro | Always, own Subscription only | -- |
| View Subscription (status, next billing date, price -- no payment_method_reference) | Platform Operator (Support) | Always, scoped to the one Pro Account being supported at a time | -- |
| View Subscription | The Client | Never | Clients have no visibility into this at all, per the Access Matrix (Subscription & Billing = None); no client-facing screen or link exposes this entity |
| Update payment method | The Pro | Always, own Subscription only | -- |
| Update payment method | Platform Operator (Support) | Never | No control is rendered in Support's view (FEAT-18.SPEC-002); this is a structural absence, not a permission check that could be bypassed |
| Update payment method | The Client | Never | No client-facing screen or capability exists for this action, per the Access Matrix (Subscription & Billing = None) |
| Cancel Subscription (Pro-initiated) | The Pro | Always, own Subscription only, and only when status is Active or Payment Failed (not already Cancelled) | Cancel action is not offered a second time once already Cancelled (active to period end) |
| Cancel Subscription (invoked by account closure) | The Pro, via FEAT-29 | Only as part of the Pro's own account closure flow (XBR-20) | N/A -- this path is always Pro-authorized because it is the Pro's own closure action; there is no separate denial case |
| Cancel Subscription | Platform Operator (Support) | Never | No cancel control exists in Support's view; excluded per scope-boundaries.md SC-05 |
| Cancel Subscription | The Client | Never | No client-facing screen or capability exists for this action, per the Access Matrix (Subscription & Billing = None) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| billing_cycle | Derived as "monthly," fixed by the single-tier definition | On create | No |
| next_billing_date | Set to one billing cycle (one month) from the date the first charge is confirmed | On create | No -- always processor-confirmed, never Pro-entered |
| price | Defaults to platform parameter: `subscription-price` on create; re-written to the new value of platform parameter: `subscription-price` on an announced price change's effective date, executed by FEAT-18.SPEC-003 | On create; and again on each announced price change's effective date | No -- one price tier only, per product-features.md FEAT-18 (Validation & Limits); the value is never Pro-entered on create or on a later price change, only ever platform-set |
| status | Defaults to Active | On create (immediately following a confirmed first charge) | No |

## Business Rules

- **Single price tier:** Exactly one subscription price applies to every Subscription (platform parameter: `subscription-price`); no plan selection exists anywhere in the product, per product-features.md FEAT-18 (Validation & Limits).
- **Grace threshold:** A renewal failure starts a grace period of platform parameter: `subscription-payment-failure-grace-period-days`, measured from the failure's confirmed date; the deadline is fixed at that point and is not extended by a subsequent failed retry within the same window (FEAT-18.SPEC-003).
- **Cancellation timing:** Cancellation takes effect at the end of the already-paid billing period, never an immediate mid-period cutoff -- a cancelled subscription remains Active in every functional sense (booking link stays live, billing already collected) through that date, then the Pro Account pauses (FEAT-18.SPEC-004).
- **Price-change notice:** Any future price change is announced to the Pro at least platform parameter: `subscription-price-change-notice-days` before it applies to their Subscription, per FEAT-18.SPEC-007's price-change notice.
- **Price-change effective-date update:** On the effective_date carried by that notice, FEAT-18.SPEC-003 writes the new platform parameter: `subscription-price` value into the Subscription's price field, exactly once per announced change; this write is a system-only transition, never triggered by a Pro action, and follows the same pattern as the grace-status transition above -- the rule fixes the timing, FEAT-18.SPEC-003 executes the write.
- **Processor authority (contention):** The payment-processing capability's recorded outcome for a charge, refund, or payment-method confirmation is authoritative for the Subscription's status and payment_method_reference; a stale Pro-facing screen showing a status that has since changed at the processor is refused with refresh (FEAT-18.SPEC-002's Edge Cases), never silently overwritten by a locally computed guess.
- **Reactivation authority (contention):** Cancellation is last-write-wins against a reactivation attempted before period end -- if both a cancel and a reactivation were somehow requested close together, whichever the processor confirms last stands; in practice this product offers no Pro-facing reactivation control before period end (FEAT-18.SPEC-002's Edge Cases), so this rule protects against any future or support-invoked path rather than a control the Pro sees today.
- **Cross-feature scope (XBR-14, XBR-11):** These rules govern only the Subscription entity; they never cancel, modify, or flag any Booking, Deposit Transaction, or Messaging Consent record. The Pro Account's pause state is governed separately by FEAT-27/FEAT-18.SPEC-004.
- **Cross-feature scope (XBR-20):** Account closure (owned by FEAT-29) invokes this Subscription's cancellation the same way a Pro-initiated cancel does -- it does not bypass the period-end timing rule.

## Edge Cases

- **Grace deadline computed at exactly the boundary (a retry confirmed on the exact calendar day the grace period ends)** -- The retry is honored if the payment-processing capability confirms it before the deadline moment passes; a confirmation arriving after that moment is treated as "grace expired unresolved," per FEAT-18.SPEC-003/FEAT-18.SPEC-004.
- **Cancellation requested on the exact day of the next scheduled renewal** -- The renewal does not fire once cancellation is confirmed (FEAT-18.SPEC-003's "No renewal needed" outcome); the Subscription is Cancelled (active to period end) through the already-paid period that predates this renewal date, and the account pauses at that existing period's end rather than being charged again first.
- **Price-change notice window overlaps a Pro's own grace period** -- The two timers are independent: a price change and a payment-failure grace period can both be in progress for the same Subscription at once, and each notice (FEAT-18.SPEC-007) is sent on its own schedule without one suppressing the other.
- **A Subscription's payment_method_reference is confirmed by the processor while the Pro's screen is mid-submission on a separate action (e.g., cancel)** -- The processor-authority rule governs: whichever confirmed outcome lands first is applied, and the other in-flight action is refused with refresh if it was evaluated against a status that has since changed.
- **Support views a Subscription whose payment_method_reference the Pro just updated** -- Support's view never includes payment_method_reference at all (per Authorization Rules), so no stale-value scenario applies to Support; they see only status, next billing date, and price, refreshed on each open.

## Acceptance Criteria

**FEAT-18.SPEC-005-AC-01:** Given Talia is subscribing for the first time, when her Subscription is created, then its price is set to the single tier (platform parameter: `subscription-price`) and no plan-selection option is ever offered.

**FEAT-18.SPEC-005-AC-02:** Given Talia's Subscription already exists, when she completes onboarding again for any reason, then she is not routed back through the Subscribe Screen a second time.

**FEAT-18.SPEC-005-AC-03:** Given Talia's renewal charge fails, when the failure is confirmed, then her Subscription's grace deadline is set to the failure date plus the fixed grace-period threshold.

**FEAT-18.SPEC-005-AC-04:** Given Talia's subscription is in Payment Failed (grace) and a retry also fails before the deadline, when the failed retry is confirmed, then the original grace deadline remains unchanged.

**FEAT-18.SPEC-005-AC-05:** Given Talia cancels her subscription, when the cancellation is confirmed, then her Subscription remains functionally active (booking link stays live) through the end of her already-paid billing period, never cutting off immediately.

**FEAT-18.SPEC-005-AC-06:** Given a price change is decided for the product, when the change is announced, then Talia is notified at least the fixed price-change notice window before it applies to her Subscription.

**FEAT-18.SPEC-005-AC-07:** Given Talia's Billing screen shows a status that has since changed at the payment-processing capability, when she attempts an action against it, then the action is refused with refresh rather than trusting the stale local status.

**FEAT-18.SPEC-005-AC-08:** Given Talia (The Pro) views her own Subscription, when she opens the Billing screen, then she sees the full status, next billing date, price, and payment method reference.

**FEAT-18.SPEC-005-AC-09:** Given a support operator (Platform Operator) views a Pro's Subscription, when they open the Billing screen in Support's view, then they see status, next billing date, and price only -- never the payment method reference.

**FEAT-18.SPEC-005-AC-10:** Given a support operator attempts to change a Pro's payment method or cancel their subscription, when they look for a control to do so, then none is shown -- Support's view offers no edit path at all.

**FEAT-18.SPEC-005-AC-11:** Given a Client attempts to reach any subscription-related view, when they try, then no such view or link exists for the Client role.

**FEAT-18.SPEC-005-AC-12:** Given Talia's subscription is already Cancelled (active to period end), when she looks for a cancel action, then it is not offered a second time.

**FEAT-18.SPEC-005-AC-13:** Given Talia closes her account via FEAT-29 (account closure, XBR-20), when the closure flow invokes this Subscription's cancellation, then the same period-end timing rule applies -- there is no immediate cutoff bypass for account closure.

**FEAT-18.SPEC-005-AC-14:** Given Talia's renewal fails on the exact calendar day her account would otherwise cross the grace deadline, when her retry is confirmed just before that deadline moment passes, then the retry is honored and the Subscription returns to Active.

**FEAT-18.SPEC-005-AC-15:** Given Talia's subscription is cancelled effective on the same date a renewal would otherwise have fired, when that date arrives, then no renewal charge fires and the account pauses at the already-paid period's end.

**FEAT-18.SPEC-005-AC-16:** Given Talia has both a price-change notice pending and an active payment-failure grace period at the same time, when each notice's own schedule arrives, then both are delivered independently without one suppressing the other.

**FEAT-18.SPEC-005-AC-17:** Given a support operator or a Client attempts to originate a new Subscription, when they look for a way to do so, then no create path exists for either role -- only the Pro can create a Subscription, and only once.

**FEAT-18.SPEC-005-AC-18:** Given a Client attempts to update a payment method or cancel a subscription, when they look for a way to do so, then no client-facing screen or capability exists for either action.

**FEAT-18.SPEC-005-AC-19:** Given a price change's effective_date arrives for Talia's subscription, when the date is reached, then her Subscription's price field is updated to the new value of platform parameter: `subscription-price` by FEAT-18.SPEC-003, as governed by this rule's timing.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 13 | 13 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 9 | 9 |
| Edge Cases | 5 | 5 |
