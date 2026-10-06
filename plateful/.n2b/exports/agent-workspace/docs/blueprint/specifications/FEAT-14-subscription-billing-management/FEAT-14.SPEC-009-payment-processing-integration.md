---
document_type: spec
spec_type: integration
spec_id: FEAT-14.SPEC-009
spec_name: Payment Processing Integration
spec_slug: payment-processing-integration
parent_feature: FEAT-14
parent_feature_name: Subscription & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Integration Spec: Payment Processing Integration

## Overview

**Name:** Payment Processing Integration
**ID:** FEAT-14.SPEC-009
**Type:** Integration
**Purpose:** Product boundary to the payment-processing capability: submits payment methods, initiates and retries charges, and receives renewal outcome events.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- Submitting a new payment method for an upgrade or a payment-details update
- Initiating the upgrade charge and subsequent recurring renewal charges
- Retrying a charge during the grace period
- Receiving renewal outcome events (success, failure) from the capability
- User-facing behavior when the capability is slow, unavailable, or rejects a request
- Disclosure of what payment and billing data is shared with the capability

**Non-Goals:**
- Choosing the payment vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate
- Deciding grace-period timing or the no-partial-refund policy -- owned by FEAT-14.SPEC-005 (Billing State & Refund Rules); this spec only carries out the charges and retries those rules require
- Writing the resulting tier, billing_period, or billing_state changes to the Subscription record -- owned by FEAT-14.SPEC-008 (Apply Subscription Change), which this spec's outcome events feed into
- Processing payments for anything other than the household subscription -- product-features.md defines no other payable object in this product

## Capability Category

**Category:** Payment processing
**Dependency Source:** ASMP-33 -- "Payment-processing capability -- Required to run the paid household subscription (monthly or yearly)" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Payment processing" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-14 -- the only feature on this row)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Maya enters payment details and subscribes to the paid tier | Upgrade to paid | FEAT-14.SPEC-002 (Upgrade to Paid) |
| Maya updates her payment method and sees billing history | Manage billing | FEAT-14.SPEC-003 (Billing & Payment Management) |
| A failed renewal charge is retried when Maya updates her payment details during the grace period | Manage billing (grace-period recovery) | FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) |
| The household's Subscription reflects the true outcome of every charge and renewal attempt | Upgrade to paid, Manage billing | FEAT-14.SPEC-008 (Apply Subscription Change) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Payment method details entered by Maya | Not stored as a Subscription field -- entered directly on FEAT-14.SPEC-002 or FEAT-14.SPEC-003 and passed through to the capability | Maya submits or updates payment details | The capability needs a payment method to charge |
| Charge amount and currency | Derived from the chosen billing_period (platform parameter: `subscription-price-monthly` or platform parameter: `subscription-price-yearly`) and Household -- currency | An upgrade, renewal, or retry charge is initiated | The capability must know what to charge and in what currency |
| Household reference | Household -- an internal reference only, not household content | Every charge or retry | Ties the payment outcome back to the correct household's Subscription |

No Dietary Rule, Weekly Plan, Grocery List, Recipe, or Pantry Item data -- nor any other Member Profile's data -- ever leaves the product through this integration. Only the organiser's own payment method and the household's billing reference are shared.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Charge outcome (succeeded / failed) | The capability reports the result of an upgrade, renewal, or retry charge | Subscription -- billing_state (via FEAT-14.SPEC-008 for success; via FEAT-14.SPEC-007 for a renewal failure) |
| Failure reason (plain-language category) | A charge fails | Subscription -- billing_history (recorded against the failed attempt) |
| Charged amount and date | A charge succeeds | Subscription -- billing_history |
| Payment method summary (masked) | Maya submits or updates payment details | Displayed on FEAT-14.SPEC-003; not stored as Subscription content beyond the masked summary needed to show what is on file |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Upgrade charge succeeded | Maya's upgrade payment is accepted | None directly -- feeds FEAT-14.SPEC-008 for the Subscription write | FEAT-14.SPEC-002 shows success and navigates to FEAT-14.SPEC-001 | FEAT-14.SPEC-002, FEAT-14.SPEC-008 |
| Upgrade charge failed | Maya's upgrade payment is declined or errors | None -- no Subscription change | FEAT-14.SPEC-002 preserves the chosen plan option and offers a retry, per this feature's Error state | FEAT-14.SPEC-002 |
| Renewal charge succeeded | A recurring renewal charge is accepted | billing_history entry appended | No direct user feedback beyond the ordinary billing history entry -- a successful renewal is the expected, silent case | FEAT-14.SPEC-003 |
| Renewal charge failed | A recurring renewal charge is declined or errors | Feeds FEAT-14.SPEC-007, which sets billing_state to Payment failed | Grace-period notice sent (FEAT-14.SPEC-011) | FEAT-14.SPEC-007, FEAT-14.SPEC-011 |
| Retry charge succeeded (during grace) | Maya's grace-period retry is accepted | Feeds FEAT-14.SPEC-007, which sets billing_state back to Active | Billing confirmation sent (FEAT-14.SPEC-010) | FEAT-14.SPEC-007, FEAT-14.SPEC-010 |
| Retry charge failed (during grace) | Maya's grace-period retry is declined or errors | None -- billing_state remains Payment failed | FEAT-14.SPEC-003 shows the retry failure inline; grace-period timer is unaffected | FEAT-14.SPEC-003, FEAT-14.SPEC-007 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-14.SPEC-002 (Upgrade to Paid) | "Subscribe" shows a loading state; after 10 seconds a note appears: "Still confirming -- this is taking longer than usual." | "Subscribe" is disabled with "Payment collection is temporarily unavailable. Try again in a few minutes." No charge is attempted and no Subscription change occurs. | The chosen plan option is preserved with the rejection reason in plain language: "This payment was declined: {reason}. Check your details and try again." |
| FEAT-14.SPEC-003 (Billing & Payment Management) | "Update payment details" or "Switch to {period}" shows a brief loading state; no separate slow-specific message, since these actions are not time-critical | The relevant action is disabled with "Payment collection is temporarily unavailable. Your current billing details are unchanged." The rest of the screen (viewing tier, billing history, downgrade/cancel navigation) remains usable. | Message with the rejection reason in plain language: "This update was declined: {reason}. Your previous payment details remain in effect." |
| FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling, retry path) | The retry attempt shows a brief in-progress state on FEAT-14.SPEC-003; no user action is blocked while waiting | The retry is queued and reattempted once the capability recovers, without narrowing the 7-day grace window itself -- the grace period's own timing (FEAT-14.SPEC-005) is independent of capability availability | The retry fails with the rejection reason shown on FEAT-14.SPEC-003; billing_state remains Payment failed and Maya may try again |

## Consent and Disclosure

- **First payment-details disclosure** -- The first time Maya enters payment details (on FEAT-14.SPEC-002), a notice appears before the payment form: "To subscribe, your payment details and household billing reference are shared with an external payment-processing service." Options: "Continue" and "Cancel". Shown once; afterwards a "How payment data is shared" link on FEAT-14.SPEC-003 reopens the same notice.
- **Renewal and retry disclosure** -- Recurring renewal charges and grace-period retries reuse the payment method already on file under the same original disclosure; no repeated prompt interrupts each renewal, since the organiser already consented to recurring billing at upgrade.
- **What is never shared** -- Dietary Rule data (including any child's allergy information), Weekly Plan and Grocery List content, Recipe data, Pantry Item data, and every other household member's data never leave the product through this integration; only Maya's own payment method and the household's billing reference are shared.

## Edge Cases

- **A charge outcome event arrives for a household that was deleted between the charge attempt and the event's delivery** -- The event is discarded silently; no Subscription write occurs for a deleted household, and no user feedback fires.
- **The same renewal-failure event is delivered twice** -- The second delivery changes nothing: FEAT-14.SPEC-007 already recorded billing_state as Payment failed with the same failure date, and no duplicate grace-period notice is sent.
- **Events arrive out of order (a renewal-success event for a later period arrives before an earlier renewal-failure event for the same Subscription)** -- The Subscription reflects the most recent event by its own event time, not arrival time; a late-arriving failure event for an already-superseded period is discarded, since a later successful renewal has already resolved billing_state to Active.
- **Capability goes down mid-charge for an upgrade** -- If the charge was not confirmed initiated, FEAT-14.SPEC-002 shows the capability-down message and no Subscription change occurs -- no half-completed upgrade state exists.
- **Maya updates payment details while a renewal charge for the same Subscription is already in flight** -- The in-flight renewal charge completes against the payment method that was on file when it was initiated; Maya's update takes effect for the next charge attempt only, avoiding a mid-charge method swap.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-002 (Upgrade to Paid) | Triggered by (inbound) | "Subscribe" initiates the upgrade charge |
| FEAT-14.SPEC-002 (Upgrade to Paid) | Affects (outbound) | Charge outcome and degradation states surface here |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Triggered by (inbound) | Payment-details updates and billing-history reads initiate here |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Affects (outbound) | Charge outcome, billing history, and degradation states surface here |
| FEAT-14.SPEC-005 (Billing State & Refund Rules) | References (inbound) | Payment-validity format rules this spec enforces |
| FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) | Triggers (outbound) | Renewal-failure and retry-outcome events fire this automation |
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggers (outbound) | A successful upgrade or renewal charge feeds this automation's write |

## Analytics and Success Signals

- **payment_method_submitted** (context: upgrade / update) -- supports success-metrics.md: "Paid Conversion Rate"
- **charge_outcome_received** (context: upgrade / renewal / retry; outcome: succeeded / failed) -- supports success-metrics.md: "Paid Conversion Rate"
- **payment_degradation_shown** (condition: slow / down / rejected; screen: spec ID) -- N/A -- no Stage 2 metric measures degradation frequency directly; retained so the product's tolerance for capability trouble is observable.

## Acceptance Criteria

**FEAT-14.SPEC-009-AC-01:** Given Maya enters valid payment details on FEAT-14.SPEC-002 and selects Monthly, when she taps Subscribe, then this integration submits a charge for the platform parameter: `subscription-price-monthly` amount in her household's currency.

**FEAT-14.SPEC-009-AC-02:** Given the upgrade charge succeeds, when the outcome is received, then FEAT-14.SPEC-008 is fed the success and writes tier=paid.

**FEAT-14.SPEC-009-AC-03:** Given the upgrade charge is declined, when the outcome is received, then FEAT-14.SPEC-002 preserves Maya's chosen plan option and offers a retry.

**FEAT-14.SPEC-009-AC-04:** Given a recurring renewal charge fails, when the outcome is received, then FEAT-14.SPEC-007 is fed the failure and opens the grace period.

**FEAT-14.SPEC-009-AC-05:** Given Maya updates payment details during a grace period and the retry succeeds, when the outcome is received, then FEAT-14.SPEC-007 clears the grace state.

**FEAT-14.SPEC-009-AC-06:** Given Maya taps Subscribe while the capability is unavailable, when the request cannot be sent, then the button is disabled with "Payment collection is temporarily unavailable. Try again in a few minutes." and no charge is attempted.

**FEAT-14.SPEC-009-AC-07:** Given the capability rejects a payment-details update on FEAT-14.SPEC-003, when this occurs, then the rejection reason is shown in plain language and her previous payment details remain in effect.

**FEAT-14.SPEC-009-AC-08:** Given Maya has never entered payment details before, when she reaches the payment form on FEAT-14.SPEC-002, then the data-sharing notice appears with "Continue" and "Cancel", and no data leaves the product until she chooses "Continue".

**FEAT-14.SPEC-009-AC-09:** Given a recurring renewal charge succeeds, when the outcome is received, then no repeated consent prompt interrupts Maya, since renewal reuses the original disclosure.

**FEAT-14.SPEC-009-AC-10:** Given a charge-outcome event arrives for a household deleted since the charge attempt, when this integration processes it, then no Subscription write occurs and no user feedback fires.

**FEAT-14.SPEC-009-AC-11:** Given the same renewal-failure event is delivered twice, when the second delivery arrives, then billing_state is unchanged and no duplicate grace-period notice fires.

**FEAT-14.SPEC-009-AC-12:** Given a late-arriving renewal-failure event for a period already superseded by a later successful renewal, when it is processed, then it is discarded and billing_state remains Active.

**FEAT-14.SPEC-009-AC-13:** Given the capability goes down mid-charge for an upgrade that was not confirmed initiated, when Maya checks her tier, then it remains free -- no half-completed upgrade state exists.

**FEAT-14.SPEC-009-AC-14:** Given Maya updates her payment details while an in-flight renewal charge is already processing against the prior method, when the in-flight charge completes, then it completes against the payment method on file when it was initiated, and Maya's update applies only to the next charge attempt.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 6 | 6 |
| Degradation Paths | 9 (3 screens x 3 conditions) | 9 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 5 | 5 |
