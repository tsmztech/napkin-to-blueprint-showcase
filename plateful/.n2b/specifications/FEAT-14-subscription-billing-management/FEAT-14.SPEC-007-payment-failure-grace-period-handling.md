---
document_type: spec
spec_type: automation
spec_id: FEAT-14.SPEC-007
spec_name: Payment Failure & Grace Period Handling
spec_slug: payment-failure-grace-period-handling
parent_feature: FEAT-14
parent_feature_name: Subscription & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Payment Failure & Grace Period Handling

## Overview

**Name:** Payment Failure & Grace Period Handling
**ID:** FEAT-14.SPEC-007
**Type:** Automation
**Purpose:** On a renewal payment failure, opens a 7-day grace period without cutting off access, and reverts the household to free if it lapses unresolved.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- Opening the 7-day grace period the moment a renewal payment failure is reported
- Keeping paid features active throughout the grace period
- Processing a payment-details update during the grace period as a retry attempt
- Clearing the grace state on a successful retry
- Reverting the household to free when the grace period expires unresolved

**Non-Goals:**
- Detecting or reporting the underlying payment failure or retry outcome itself -- owned by FEAT-14.SPEC-009 (Payment Processing Integration); this automation only reacts to the events that spec reports
- Writing the final tier/billing_state values to the Subscription record -- owned by FEAT-14.SPEC-008 (Apply Subscription Change), which this automation invokes for the grace-expiry reversion
- Sending the grace-period notice itself -- owned by FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice); this automation only triggers it
- Collecting the updated payment details -- owned by FEAT-14.SPEC-003 (Billing & Payment Management), which is where Maya enters them during the grace period

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Renewal payment failure reported | FEAT-14.SPEC-009 (Payment Processing Integration) | Fires when the payment-processing capability reports a failed renewal charge for a household currently paid with billing_state Active | Subscription reference, failure reason (plain-language category) |
| Payment details updated during grace | FEAT-14.SPEC-003 (Billing & Payment Management) | Fires when Maya submits updated payment details while billing_state is Payment failed | Updated payment details (passed through to FEAT-14.SPEC-009 for the retry charge) |
| Grace-period boundary reached | System (schedule-based) | Fires once per Subscription, 7 days after payment_failure_date, only if billing_state is still Payment failed at that moment | Subscription reference, payment_failure_date |

## Processing Logic

1. Receive the renewal-payment-failure event from FEAT-14.SPEC-009 for a household currently on billing_state Active.
2. Set billing_state to Payment failed and record payment_failure_date as the failure date, starting the 7-day grace-period timer (grace period end = payment_failure_date + 7 days).
3. Confirm paid features (AI plan generation, pantry-aware plan weighting, rating-based learning) remain fully active -- no gating change accompanies this transition.
4. Trigger FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice) to inform Maya.
5. If Maya updates payment details during the grace period (via FEAT-14.SPEC-003), submit a retry charge through FEAT-14.SPEC-009.
6. On a successful retry: clear the grace state (billing_state back to Active, payment_failure_date cleared), and record the successful charge in billing_history. Trigger FEAT-14.SPEC-010 (Billing Confirmation Notification) to confirm resolution.
7. On a failed retry: billing_state remains Payment failed and the grace-period timer is unaffected -- Maya may retry again at any point before the 7-day boundary.
8. If the 7-day boundary is reached with billing_state still Payment failed, invoke FEAT-14.SPEC-008 (Apply Subscription Change) to revert the household to free tier, setting billing_state to Reverted to free.
9. Confirm no data is removed by the reversion -- past plans, ratings, recipes, pantry items, and the shared list remain fully available, per XBR-05.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Grace period opened | Renewal payment failure reported for a paid, Active household | billing_state set to Payment failed; payment_failure_date recorded | Grace-period notice sent (FEAT-14.SPEC-011); billing-state banner appears on FEAT-14.SPEC-001 and FEAT-14.SPEC-003 | FEAT-14.SPEC-001, FEAT-14.SPEC-003, FEAT-14.SPEC-011 |
| Grace-period retry succeeds | Maya updates payment details during the grace period and the retry charge succeeds | billing_state set back to Active; payment_failure_date cleared; successful charge recorded in billing_history | Billing confirmation notification sent (FEAT-14.SPEC-010); billing-state banner disappears | FEAT-14.SPEC-001, FEAT-14.SPEC-003, FEAT-14.SPEC-010 |
| Grace-period retry fails | Maya updates payment details during the grace period but the retry charge is declined or errors | billing_state remains Payment failed; payment_failure_date unchanged; failed retry recorded in billing_history | FEAT-14.SPEC-003 shows the retry failure inline; the grace-period notice's remaining days are unaffected | FEAT-14.SPEC-003 |
| Grace period expires unresolved | 7 days elapse with billing_state still Payment failed | Household reverts to free via FEAT-14.SPEC-008: tier set to free, billing_period set to none, billing_state set to Reverted to free, payment_failure_date cleared | Confirmation of the reversion sent (FEAT-14.SPEC-010); no data removed | FEAT-14.SPEC-001, FEAT-14.SPEC-003, FEAT-14.SPEC-008, FEAT-14.SPEC-010 |
| Automation failure (grace-state transition itself fails to record) | An internal processing error prevents billing_state from being set on a reported failure | Household remains at its last-known billing_state (no partial change) | No user-visible failure message -- the payment-processing capability's own retry/backoff, per FEAT-14.SPEC-009, ensures the failure event is eventually processed | FEAT-14.SPEC-009 |

## Data Model

**Reads:** Subscription -- tier, billing_state, payment_failure_date (to confirm the household is eligible for grace handling at each step and to evaluate the 7-day boundary).
**Creates:** None.
**Updates:** Subscription -- billing_state (Active -> Payment failed -> Active or Reverted to free); payment_failure_date (set on failure, cleared on resolution); billing_history (appends the failure, retry, and resolution entries).
**Deletes:** None.

## Business Rules

- The payment grace period is exactly 7 days from the failure date; paid features remain fully active throughout, per product-features.md's Validation & Limits.
- A grace-period reversion is functionally identical to a downgrade: no data is removed, and every past plan, rating, recipe, pantry item, and the shared list stay fully available (XBR-05).
- The grace-period timer is not reset or extended by a failed retry attempt -- only a successful retry clears it; only the original 7-day boundary determines expiry.
- This automation is the sole trigger of a grace-period-driven reversion; every other tier or billing_state write is triggered elsewhere (upgrade, manual downgrade, cancellation) and is out of scope here.

## Edge Cases

- **Maya's retry succeeds on day 7 itself, at effectively the same moment the grace-expiry boundary check runs** -- The successful retry, once confirmed, takes precedence: if the retry's confirmation is recorded before the expiry check executes, billing_state is set to Active and no reversion occurs. If the expiry check executes first, the reversion proceeds and Maya's subsequent successful charge (now against a free-tier household) is treated as a fresh upgrade rather than a grace-period retry.
- **Maya submits payment details twice in quick succession during the grace period** -- Only one retry charge is in flight at a time; a second submission while the first retry is processing is held until the first resolves, preventing two concurrent charge attempts against the same Subscription.
- **The renewal-payment-failure event is delivered twice for the same failure** -- The second delivery changes nothing: billing_state is already Payment failed with the same failure date, and no duplicate grace-period notice is sent (per FEAT-14.SPEC-011's deduplication rule).
- **Concurrent trigger firing (a renewal-failure event and a grace-expiry boundary check for the same Subscription land at effectively the same time)** -- These cannot genuinely be concurrent: the grace-expiry check only fires 7 days after a failure was recorded, so a fresh failure event and an expiry check for a prior failure reference different failure dates and are processed independently, in event-time order.
- **Trigger fires while a previous run is in flight (e.g., a retry is processing when the expiry boundary is reached)** -- The expiry check waits for the in-flight retry to resolve before evaluating billing_state, so a retry that resolves at the boundary is never overtaken mid-processing by the reversion.
- **Household is deleted (FEAT-18 cascade) while in a grace period** -- The grace-period timer and any pending retry are cancelled silently along with the rest of the household's data; no reversion or notice fires for a deleted household.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-009 (Payment Processing Integration) | Triggered by (inbound) | The renewal-payment-failure event fires this automation |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Triggered by (inbound) | Maya's payment-details update during grace fires the retry path |
| FEAT-14.SPEC-009 (Payment Processing Integration) | Triggers (outbound) | Retry charges are submitted through this integration |
| FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice) | Triggers (outbound) | Grace period opening fires this notification |
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | Triggers (outbound) | A successful retry or grace-expiry reversion fires this notification |
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggers (outbound) | Grace-expiry reversion is executed through this automation |
| FEAT-14.SPEC-001 (Plan Tier Overview) | Affects (outbound) | The billing-state banner reflects every state this automation sets |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Affects (outbound) | The billing-state banner and retry outcome are shown here |

## Analytics and Success Signals

- **subscription_payment_failed** (billing period) -- supports success-metrics.md: "Paying Household Retention"
- **subscription_grace_period_opened** (days remaining: 7) -- supports success-metrics.md: "Paying Household Retention"
- **subscription_grace_retry_succeeded** (days elapsed since failure) -- supports success-metrics.md: "Paying Household Retention"
- **subscription_grace_retry_failed** (days elapsed since failure) -- N/A -- no Stage 2 metric tracks individual retry failures; retained as an operational signal for grace-period recovery rate
- **subscription_grace_expired** (billing period at time of expiry) -- supports success-metrics.md: "Paying Household Retention" (a lapsed grace period is a retention loss this metric must reflect)

## Acceptance Criteria

**FEAT-14.SPEC-007-AC-01:** Given a paid household with billing_state Active, when FEAT-14.SPEC-009 reports a renewal payment failure, then billing_state is set to Payment failed, payment_failure_date is recorded, the 7-day grace timer starts, and paid features remain active.

**FEAT-14.SPEC-007-AC-02:** Given a household enters the grace period, when the transition completes, then FEAT-14.SPEC-011 sends Maya the grace-period notice.

**FEAT-14.SPEC-007-AC-03:** Given Maya is in a grace period and updates her payment details on FEAT-14.SPEC-003, when the retry charge succeeds, then billing_state is set back to Active and FEAT-14.SPEC-010 sends a confirmation.

**FEAT-14.SPEC-007-AC-04:** Given Maya is in a grace period and updates her payment details, when the retry charge fails, then billing_state remains Payment failed and the grace-period timer is unaffected.

**FEAT-14.SPEC-007-AC-05:** Given a household has been in the grace period for 7 days with no successful retry, when the expiry boundary is reached, then the household reverts to free tier via FEAT-14.SPEC-008 and FEAT-14.SPEC-010 sends a confirmation.

**FEAT-14.SPEC-007-AC-06:** Given a household's grace-period reversion completes, when Maya checks her data, then every past plan, rating, recipe, pantry item, and the shared list remain fully available.

**FEAT-14.SPEC-007-AC-07:** Given Maya's retry succeeds at effectively the same moment the 7-day expiry boundary is reached, when the retry's confirmation is recorded first, then billing_state is set to Active and no reversion occurs.

**FEAT-14.SPEC-007-AC-08:** Given Maya submits payment details twice in quick succession during a grace period, when the first retry is still processing, then the second submission is held until the first resolves, and no two retry charges are ever in flight at once.

**FEAT-14.SPEC-007-AC-09:** Given the same renewal-payment-failure event is delivered twice, when the second delivery arrives, then billing_state is unchanged and no duplicate grace-period notice is sent.

**FEAT-14.SPEC-007-AC-10:** Given a retry is processing when the 7-day expiry boundary is reached, when the expiry check runs, then it waits for the in-flight retry to resolve before evaluating billing_state.

**FEAT-14.SPEC-007-AC-11:** Given a household is deleted while in a grace period, when the deletion completes, then the pending grace-period timer and any in-flight retry are cancelled silently with no reversion or notice.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (failure reported, payment updated during grace, grace-boundary reached) | 3 |
| Outcome Paths | 5 (opened, retry succeeds, retry fails, expires, automation failure) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
