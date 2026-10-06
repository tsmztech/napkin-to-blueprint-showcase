---
document_type: spec
spec_type: automation
spec_id: FEAT-18.SPEC-003
spec_name: Subscription Renewal & Payment-Failure Processing
spec_slug: subscription-renewal-payment-failure-processing
parent_feature: FEAT-18
parent_feature_name: Pro Subscription Billing & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Subscription Renewal & Payment-Failure Processing

## Overview

**Name:** Subscription Renewal & Payment-Failure Processing
**ID:** FEAT-18.SPEC-003
**Type:** Automation
**Purpose:** Runs the monthly renewal charge against the payment-processing capability, branches on success or failure, updates the billing date on success, and starts the 7-day grace-period clock on failure.
**Parent Feature:** FEAT-18 -- Pro Subscription Billing & Account Management

## Scope and Non-Goals

**In Scope:**
- Firing the monthly renewal charge on the subscription's billing cycle
- Branching on the processor's outcome: success or failure
- Updating the Subscription's next_billing_date on success
- Setting the Subscription's status to Payment Failed and starting the grace-period clock on failure
- Updating the Subscription's price field to the new value of platform parameter: `subscription-price` on an announced price change's effective date

**Non-Goals:**
- Pausing the Pro Account when the grace period expires unresolved -- owned by FEAT-18.SPEC-004 (Subscription-Lapse Account Pause Trigger); this spec only starts the clock
- Sending the renewal receipt or payment-failure notice -- owned by FEAT-18.SPEC-007 (Subscription Billing Notifications); this spec triggers those notifications but does not define their content
- The processor charge mechanics themselves -- owned by FEAT-18.SPEC-006 (Subscription Billing Integration); this spec consumes that integration's inbound renewal-outcome events
- Cancellation processing -- owned by FEAT-18.SPEC-005 (timing) and FEAT-18.SPEC-006 (execution); a subscription already in Cancelled (active to period end) status does not renew, per Validation & Limits in product-features.md FEAT-18

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Monthly billing cycle reaches its renewal date | System (schedule-based, driven by the Subscription's own next_billing_date field) | Fires once per Subscription when its next_billing_date arrives, only if the Subscription's status is Active | Subscription reference, payment_method_reference, price (platform parameter: `subscription-price`) |
| Renewal outcome received | FEAT-18.SPEC-006 (Subscription Billing Integration) | Fires when the payment-processing capability reports the renewal charge's result (an external-event trigger) | Outcome (succeeded / failed), failure reason category (on failure), processor's confirmed charge date |
| Price-change effective date arrives | System (schedule-based, driven by the effective_date carried by FEAT-18.SPEC-007's price-change notice, timing governed by FEAT-18.SPEC-005) | Fires once per Subscription on the effective_date of an announced price change, which always falls at least platform parameter: `subscription-price-change-notice-days` after the notice was sent | Subscription reference, new value of platform parameter: `subscription-price`, effective_date |

## Processing Logic

1. On the Subscription's next_billing_date, if its status is Active, submit the renewal charge for its price (platform parameter: `subscription-price`) against its payment_method_reference through FEAT-18.SPEC-006.
2. Receive the renewal outcome event from FEAT-18.SPEC-006.
3. If the outcome is success: set the Subscription's next_billing_date forward by one billing cycle; leave status as Active; trigger the renewal receipt notification (FEAT-18.SPEC-007).
4. If the outcome is failure: set the Subscription's status to Payment Failed (7-day grace); record the grace-period start as the failure's confirmed date; compute the grace deadline as the start date plus platform parameter: `subscription-payment-failure-grace-period-days`; trigger the payment-failure grace notice (FEAT-18.SPEC-007) carrying that deadline.
5. If a Subscription in Payment Failed status has its payment method updated and confirmed before the grace deadline (via FEAT-18.SPEC-002 and FEAT-18.SPEC-006), retry the renewal charge; on success, follow step 3; on continued failure, the grace deadline is not extended -- the clock started at the original failure date continues.
6. On the effective_date of an announced price change (carried by the price-change notice FEAT-18.SPEC-007 already sent, per FEAT-18.SPEC-005's timing rule), update the Subscription's price field to the new platform parameter: `subscription-price` value. This update runs independently of step 1: if the effective_date coincides with a renewal date, the price is updated first, so that renewal's charge (step 1) is submitted at the new price.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Renewal succeeds | The processor confirms the charge | Subscription.next_billing_date advances one billing cycle; status remains Active | Status banner continues to show "Active -- next billing date {new date}"; renewal receipt notification sent | FEAT-18.SPEC-002, FEAT-18.SPEC-007 |
| Renewal fails, grace starts | The processor reports a decline or failure | Subscription.status set to Payment Failed (7-day grace); grace deadline recorded | Status banner shows the grace deadline with an emphasized "Update payment method" action; payment-failure grace notice sent | FEAT-18.SPEC-002, FEAT-18.SPEC-007 |
| Retry during grace succeeds | The Pro updates the payment method and the retried charge succeeds before the grace deadline | Subscription.status returns to Active; next_billing_date advances one billing cycle; grace state cleared | Status banner returns to "Active -- next billing date {date}"; renewal receipt sent | FEAT-18.SPEC-002, FEAT-18.SPEC-004 (clears any pause that had not yet been triggered), FEAT-18.SPEC-007 |
| Retry during grace fails again | The Pro's updated payment method is also declined, or no update is made | Subscription remains Payment Failed; grace deadline unchanged (not extended) | Status banner continues to show the original grace deadline | FEAT-18.SPEC-002 |
| Grace period expires unresolved | No successful charge occurs by the grace deadline | Subscription remains Payment Failed | Handed off to FEAT-18.SPEC-004, which pauses the Pro Account | FEAT-18.SPEC-004 |
| No renewal needed | Subscription status is not Active on its billing date (e.g., already Cancelled active-to-period-end) | None -- the automation does not fire a charge | No feedback -- the cancellation's own timing (FEAT-18.SPEC-005) governs what happens at period end instead | FEAT-18.SPEC-005 |
| Price-change effective date reached | The effective_date carried by an already-sent price-change notice arrives | Subscription.price is updated to the new platform parameter: `subscription-price` value | No direct feedback on the write itself -- the Pro was already informed by the price-change notice at least platform parameter: `subscription-price-change-notice-days` in advance; the Billing screen (FEAT-18.SPEC-002) and any renewal charge from this date forward reflect the new price | FEAT-18.SPEC-002, FEAT-18.SPEC-005, FEAT-18.SPEC-007 |

## Data Model

**Reads:** Subscription -- status, next_billing_date, payment_method_reference, price.
**Creates:** None.
**Updates:** Subscription -- status (Active <-> Payment Failed), next_billing_date (advanced on success), price (updated to the new value of platform parameter: `subscription-price` on an announced price change's effective date).
**Deletes:** None.

## Business Rules

- Only a Subscription in Active status renews on its billing date; a Cancelled (active to period end) subscription never renews, per product-features.md FEAT-18 (Validation & Limits).
- The grace period is fixed at platform parameter: `subscription-payment-failure-grace-period-days`, per FEAT-18.SPEC-005.
- The grace deadline is set once, at the original failure's confirmed date, and is never extended by a subsequent failed retry within the same grace window.
- XBR-11 and XBR-14: this automation never itself cancels bookings or affects existing bookings -- a renewal failure only changes the Subscription's own status; the Pro Account pause (which stops new bookings) is a separate, standalone effect owned by FEAT-18.SPEC-004.
- The payment-processing capability's recorded outcome is authoritative, per the dependency map's Contention note for the Subscription entity -- this automation never infers success or failure locally.
- A price-change effective date is a system-only write, parallel to the grace-status transition: the Subscription's price field is updated to the new platform parameter: `subscription-price` value exactly once, on the effective_date already disclosed by FEAT-18.SPEC-007's price-change notice, and is never triggered by any Pro action or screen input.

## Edge Cases

- **The Pro cancels the subscription on the same day a renewal would have fired** -- Cancellation (FEAT-18.SPEC-005) takes effect for billing purposes immediately: the Subscription's status is Cancelled (active to period end) before this automation's renewal check runs, so no renewal charge fires and the "No renewal needed" outcome applies.
- **The renewal outcome event arrives twice (duplicate delivery from FEAT-18.SPEC-006)** -- The second delivery is a no-op: a Subscription already advanced to the next billing cycle, or already in Payment Failed with its grace deadline already recorded, is not changed again, and no duplicate notification fires.
- **Concurrent trigger firing (the scheduled renewal and a Pro-initiated payment-method-update retry overlap for the same Subscription)** -- Only one outcome is applied: the payment-processing capability processes one charge at a time per Subscription, and this automation applies whichever confirmed outcome arrives first; the other attempt, if it was in fact a duplicate submission, is reconciled to the same confirmed outcome rather than double-charging.
- **A trigger fires while a previous run is still in flight for the same Subscription** -- A second renewal attempt cannot start for a Subscription whose prior renewal charge has not yet returned an outcome; the automation waits for the in-flight outcome before evaluating whether another attempt is needed.
- **The payment method is updated but the retry charge itself times out** -- Treated as a failure outcome (per FEAT-18.SPEC-006's degradation behavior): the Subscription remains Payment Failed with its original grace deadline unchanged, and the Pro sees the standard failure state, not a false success.
- **A price-change effective date coincides with a scheduled renewal date** -- The price update (step 6) is applied before the renewal check (step 1) runs, so the renewal charge submitted that day uses the new platform parameter: `subscription-price` value, and the resulting renewal receipt (FEAT-18.SPEC-007) reflects the updated price rather than the prior one.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-006 (Subscription Billing Integration) | Triggered by (inbound) | The renewal-outcome inbound event fires this automation's outcome branching |
| FEAT-18.SPEC-006 (Subscription Billing Integration) | Triggers (outbound) | This automation submits the renewal (and retry) charge requests |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | Affects (outbound) | Status banner and next billing date reflect this automation's outcomes |
| FEAT-18.SPEC-004 (Subscription-Lapse Account Pause Trigger) | Affects (outbound) | An unresolved grace expiry hands off to this automation |
| FEAT-18.SPEC-005 (Subscription Billing Rules) | References (inbound) | Grace-period length, renewal-eligibility rules, and the price-change notice lead time that sets each effective_date |
| FEAT-18.SPEC-007 (Subscription Billing Notifications) | Triggers (outbound) | Renewal success and failure each trigger their respective notification |

## Analytics and Success Signals

- **subscription_renewed** (renewal number in sequence) -- supports success-metrics.md: "Subscription Retention"
- **subscription_payment_failed** (grace deadline) -- supports success-metrics.md: "Subscription Retention"
- **subscription_payment_recovered** (days into grace period when recovered) -- supports success-metrics.md: "Subscription Retention"

## Acceptance Criteria

**FEAT-18.SPEC-003-AC-01:** Given Talia's subscription is Active and reaches its billing date, when the renewal charge succeeds, then next_billing_date advances one cycle, status remains Active, and she receives the renewal receipt (FEAT-18.SPEC-007).

**FEAT-18.SPEC-003-AC-02:** Given Talia's subscription is Active and reaches its billing date, when the renewal charge fails, then status is set to Payment Failed with a grace deadline platform parameter: `subscription-payment-failure-grace-period-days` from the failure date, and she receives the payment-failure grace notice (FEAT-18.SPEC-007).

**FEAT-18.SPEC-003-AC-03:** Given Talia's subscription is in Payment Failed (grace), when she updates her payment method and the retried charge succeeds before the grace deadline, then status returns to Active, next_billing_date advances, and she receives a renewal receipt.

**FEAT-18.SPEC-003-AC-04:** Given Talia's subscription is in Payment Failed (grace) and her retried payment method is also declined, when the retry fails, then the Subscription remains Payment Failed and the original grace deadline is unchanged.

**FEAT-18.SPEC-003-AC-05:** Given Talia's subscription is in Payment Failed (grace) and no successful charge occurs by the grace deadline, when the deadline passes, then this automation hands off to FEAT-18.SPEC-004, which pauses the Pro Account.

**FEAT-18.SPEC-003-AC-06:** Given Talia's subscription is Cancelled (active to period end), when its former billing date arrives, then no renewal charge fires.

**FEAT-18.SPEC-003-AC-07:** Given the payment-processing capability delivers the same renewal-outcome event twice, when the second delivery arrives, then the Subscription's state does not change again and no duplicate notification fires.

**FEAT-18.SPEC-003-AC-08:** Given a scheduled renewal and a Pro-initiated retry could overlap for the same Subscription, when both are in flight, then only one confirmed outcome is applied and no duplicate charge occurs.

**FEAT-18.SPEC-003-AC-09:** Given a renewal charge for Talia's subscription is already in flight, when the schedule would otherwise fire a second attempt for the same Subscription, then the second attempt waits for the in-flight outcome rather than starting concurrently.

**FEAT-18.SPEC-003-AC-10:** Given Talia updates her payment method during grace and the retry charge itself times out, when the timeout is reported, then the Subscription remains Payment Failed with its original grace deadline, and she is not shown a false success.

**FEAT-18.SPEC-003-AC-11:** Given a price change's effective_date arrives for Talia's subscription, when this automation processes it, then Subscription.price is updated to the new platform parameter: `subscription-price` value, and any renewal charge submitted on or after that date uses the new price.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (scheduled renewal, renewal outcome received, price-change effective date) | 3 |
| Outcome Paths | 7 | 7 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
