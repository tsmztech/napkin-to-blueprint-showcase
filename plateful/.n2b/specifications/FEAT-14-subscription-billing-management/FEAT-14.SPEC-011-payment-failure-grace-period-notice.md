---
document_type: spec
spec_type: notification
spec_id: FEAT-14.SPEC-011
spec_name: Payment Failure Grace-Period Notice
spec_slug: payment-failure-grace-period-notice
parent_feature: FEAT-14
parent_feature_name: Subscription & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Notification Spec: Payment Failure Grace-Period Notice

## Overview

**Name:** Payment Failure Grace-Period Notice
**ID:** FEAT-14.SPEC-011
**Type:** Notification
**Purpose:** Sends the organiser a clear grace-period notice and a path to update payment details after a failed renewal, distinct in urgency from a routine billing confirmation.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- The notice sent the moment a renewal payment failure opens the 7-day grace period
- Its exact content on every channel it uses, including the path to update payment details

**Non-Goals:**
- The confirmation sent once the grace period is resolved (retry success or expiry) -- owned by FEAT-14.SPEC-010 (Billing Confirmation Notification); this spec covers only the initial notice
- Deciding the grace period's length or timing -- owned by FEAT-14.SPEC-005 (Billing State & Refund Rules); this spec only communicates the outcome of that rule
- Retrying the charge itself -- owned by FEAT-14.SPEC-009 (Payment Processing Integration), triggered from FEAT-14.SPEC-003 where this notice's CTA leads
- The transactional email capability's own send/delivery mechanics -- owned by FEAT-14.SPEC-012 (Transactional Email Delivery (Billing)), which this notification's email channel is delivered through

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when a grace period opens | Maya needs the persistent grace/billing-state banner reinforced by an explicit notice she cannot miss on her next visit |
| Email | Always, in addition to in-app | A renewal failure can occur at any time, including while Maya is away from the product for days; email is the channel most likely to reach her before the 7-day grace window narrows, consistent with the urgency this notice carries |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Grace period opened | FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) | Fires immediately when a renewal payment failure sets billing_state to Payment failed | Household reference, payment_failure_date (grace-period end date = payment_failure_date + 7 days) |

## Audience and Preferences

**Recipients:** Maya -- the organiser, per the Billing column of the Access Matrix in user-persona.md (Full). No other household role receives this notice: Sam and both Jordan rows have Billing: None, and Riley's Billing access is View of plan tier only and never includes payment-related notices.

**Preference Controls:**

N/A -- this notice carries no user-configurable preference, for the same reason as FEAT-14.SPEC-010: it is a required, time-sensitive account notice about the household's own payment status, not a discretionary alert Maya can silence. Both channels always fire.

**Quiet Hours:** N/A -- a renewal payment failure is not held for quiet hours. Delaying this notice would shorten Maya's effective window to act within the 7-day grace period without shortening the grace period itself, working directly against the notice's purpose.

## Content Definition

**In-app:**
- **Title:** Payment failed -- action needed
- **Body:** We couldn't process your renewal payment. Update your card by {grace_period_end_date} to keep your paid features.
- **CTA:** Update payment details -- deep-links to FEAT-14.SPEC-003 (Billing & Payment Management)

**Email:**
- **Subject:** Action needed: your Plateful payment didn't go through
- **Body:**
  Hi {organiser_first_name},

  We weren't able to process your renewal payment for Plateful. Your paid features are still active, but you'll need to update your payment details by {grace_period_end_date} to avoid moving to the free plan.

  If nothing changes by then, your household moves to the free plan -- no data is lost. Every past plan, rating, recipe, pantry item, and your shared list will still be fully available.
- **CTA (button):** Update payment details -- deep-links to FEAT-14.SPEC-003 (Billing & Payment Management)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {grace_period_end_date} | Derived -- Subscription.payment_failure_date + 7 days, computed by FEAT-14.SPEC-007 | March 14 | Never empty -- this notice is fired only once FEAT-14.SPEC-007 has recorded payment_failure_date and computed the grace-period end date |
| {organiser_first_name} | Member Profile -- display_name (of the organiser) | Maya | Greeting renders as "Hi there," |

## Delivery Rules

**Batching:** No batching -- exactly one grace period can be open per household at a time (a household cannot enter a second grace period while already in one), so no scenario produces multiple pending instances to combine.
**Deduplication:** At most one notice per grace-period opening. A duplicate renewal-failure event for the same failure (per FEAT-14.SPEC-009's Edge Cases) never re-triggers this notice, since FEAT-14.SPEC-007 only opens the grace period once per failure.
**Retry on failure:** Email delivery failure is retried up to 3 times over 6 hours, per FEAT-14.SPEC-012. After the final failure, the in-app notice and the persistent billing-state banner (FEAT-14.SPEC-001, FEAT-14.SPEC-003) stand as the surviving record -- Maya still sees the grace state every time she opens the product even if the email never arrives.
**Expiry:** This notice does not expire in the usual sense -- it remains relevant for the entire 7-day grace period. If it has not been delivered by the time the grace period resolves (retry succeeds or expiry occurs), the pending send is superseded by FEAT-14.SPEC-010's resolution confirmation rather than being sent late as a now-irrelevant "you're in a grace period" notice.

## Edge Cases

- **The grace period resolves (retry succeeds or expiry occurs) before this notice's email has been delivered** -- The pending email send is cancelled; delivering a "your payment failed, act by {date}" notice after the grace period has already resolved would be actively confusing, so FEAT-14.SPEC-010's resolution confirmation takes its place.
- **Household is deleted while a grace period is open and this notice is still pending delivery** -- The notice is cancelled silently on every channel.
- **Maya has no registered email on her Member Profile** -- The email channel is skipped for that delivery; the in-app notice and billing-state banner still deliver as the primary record.
- **The same renewal-failure event is delivered twice (per FEAT-14.SPEC-009's Edge Cases)** -- Only one notice is ever sent, since FEAT-14.SPEC-007 sets billing_state to Payment failed only once for the same failure.
- **Maya updates her payment details and the retry fails, still within the grace period** -- This notice is not re-sent; the grace period and its end date are unchanged, and the retry failure is shown inline on FEAT-14.SPEC-003 rather than through a repeated notice.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) | Triggered by (inbound) | Grace-period opening fires this notice |
| FEAT-14.SPEC-012 (Transactional Email Delivery (Billing)) | References (outbound) | The email channel is delivered through this integration |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Navigation (outbound) | The CTA deep-links here to update payment details |
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | References (outbound) | The eventual resolution confirmation supersedes this notice once the grace period resolves |

## Analytics and Success Signals

- **grace_period_notice_delivered** (channel: in_app / email) -- supports success-metrics.md: "Paying Household Retention"
- **grace_period_notice_cta_tapped** (channel) -- supports success-metrics.md: "Paying Household Retention"
- **grace_period_notice_email_skipped** (reason: no_email_on_file) -- N/A -- no Stage 2 metric tracks skipped email deliveries; retained so silent delivery gaps remain observable

## Acceptance Criteria

**FEAT-14.SPEC-011-AC-01:** Given a renewal payment failure opens a grace period for Maya's household, when FEAT-14.SPEC-007 fires, then she receives an in-app notice titled "Payment failed -- action needed" and an email with the subject "Action needed: your Plateful payment didn't go through".

**FEAT-14.SPEC-011-AC-02:** Given Maya receives this notice, when she taps "Update payment details", then she lands on FEAT-14.SPEC-003 (Billing & Payment Management).

**FEAT-14.SPEC-011-AC-03:** Given the notice is delivered, when Maya reads it, then it states the exact date by which she must act, computed as 7 days from the failure date.

**FEAT-14.SPEC-011-AC-04:** Given Maya's grace period resolves via a successful retry before this notice's email has sent, when the retry succeeds, then the pending email send is cancelled and FEAT-14.SPEC-010's confirmation is sent instead.

**FEAT-14.SPEC-011-AC-05:** Given Maya's grace period expires unresolved before this notice's email has sent, when the expiry occurs, then the pending email send is cancelled and FEAT-14.SPEC-010's expiry confirmation is sent instead.

**FEAT-14.SPEC-011-AC-06:** Given the household is deleted while this notice is pending, when the deletion completes, then the notice is cancelled silently on every channel.

**FEAT-14.SPEC-011-AC-07:** Given Maya has no registered email on her Member Profile, when the grace period opens, then only the in-app notice is delivered.

**FEAT-14.SPEC-011-AC-08:** Given the same renewal-failure event is delivered twice, when the second delivery arrives, then only one notice was ever sent.

**FEAT-14.SPEC-011-AC-09:** Given Maya updates her payment details and the retry fails within the same grace period, when the retry outcome is processed, then this notice is not re-sent and the grace-period end date is unchanged.

**FEAT-14.SPEC-011-AC-10:** Given the email delivery for this notice fails 3 times over 6 hours, when the final retry fails, then the in-app notice and the persistent billing-state banner remain the surviving record with no additional error shown.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (in-app, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (no configurable preference -- always-on both channels) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
