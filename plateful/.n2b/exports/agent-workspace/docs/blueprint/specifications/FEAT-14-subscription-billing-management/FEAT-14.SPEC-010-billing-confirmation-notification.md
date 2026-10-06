---
document_type: spec
spec_type: notification
spec_id: FEAT-14.SPEC-010
spec_name: Billing Confirmation Notification
spec_slug: billing-confirmation-notification
parent_feature: FEAT-14
parent_feature_name: Subscription & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Notification Spec: Billing Confirmation Notification

## Overview

**Name:** Billing Confirmation Notification
**ID:** FEAT-14.SPEC-010
**Type:** Notification
**Purpose:** Sends confirmation of an upgrade, downgrade, cancellation, period switch, or grace-period resolution to the organiser.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- The confirmation sent when an upgrade completes
- The confirmation sent when a billing-period switch takes effect
- The confirmation sent when a downgrade or cancellation reversion completes
- The confirmation sent when a grace-period retry succeeds or a grace period expires unresolved

**Non-Goals:**
- The initial grace-period notice itself -- owned by FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice); this spec covers only the confirmation once the grace period is resolved (successfully or by lapsing)
- Deciding which changes are applied and when -- owned by FEAT-14.SPEC-008 (Apply Subscription Change); this spec begins where that automation's trigger fires
- The transactional email capability's own send/delivery mechanics -- owned by FEAT-14.SPEC-012 (Transactional Email Delivery (Billing)), which this notification's email channel is delivered through

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when a billing change is confirmed | Maya reviews her plan and billing inside the product regularly; the confirmation belongs where the change is visible |
| Email | Always, in addition to in-app | Billing changes are consequential enough to warrant a durable record outside the session in which they occurred, and Maya's Sunday-evening usage pattern means she is often away from the product when a renewal-driven change (a period switch or a grace-period resolution) actually takes effect |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Upgrade applied | FEAT-14.SPEC-008 (Apply Subscription Change) | Fires when an upgrade is written to the Subscription record | Household reference, billing_period, currency |
| Period switch applied | FEAT-14.SPEC-008 (Apply Subscription Change) | Fires when a scheduled period switch takes effect at renewal | Household reference, new billing_period |
| Downgrade or cancellation applied | FEAT-14.SPEC-008 (Apply Subscription Change) | Fires when a scheduled downgrade or cancellation reversion completes at period end | Household reference, reversion type (downgrade / cancellation) |
| Grace-period retry succeeded | FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) | Fires when a grace-period retry charge succeeds and billing_state returns to Active | Household reference |
| Grace-period expired unresolved | FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) via FEAT-14.SPEC-008 | Fires when the 7-day grace period lapses and the household reverts to free | Household reference |

## Audience and Preferences

**Recipients:** Maya -- the organiser, per the Billing column of the Access Matrix in user-persona.md (Full). No other household role receives this notification: Sam and both Jordan rows have Billing: None, and Riley's Billing access is View of plan tier only and never includes notifications about it.

**Preference Controls:**

N/A -- this notification carries no user-configurable preference. Member Profile's notification_preferences field (per the Feature Dependency Map) covers only the plan-ready and nightly-nudge toggles owned by FEAT-07 and FEAT-13; the product defines no corresponding on/off control for billing confirmations. Both channels (in-app and email) always fire for every applied change, since a billing confirmation is a required record of a consequential account change, not a discretionary alert Maya can silence.

**Quiet Hours:** N/A -- billing confirmations are not held for quiet hours. A billing change is a direct consequence of Maya's own action (upgrade, period switch, downgrade, cancellation) or a resolution she is actively waiting on (grace-period outcome), so delaying the confirmation would leave her without timely proof that her action took effect.

## Content Definition

**In-app:**
- **Title (upgrade):** You're on the paid plan
- **Body (upgrade):** Your {billing_period} subscription is active. Your first AI-generated plan is on its way.
- **CTA:** View plan -- deep-links to FEAT-14.SPEC-001 (Plan Tier Overview)

- **Title (period switch):** Your billing period changed
- **Body (period switch):** You're now on {billing_period} billing, effective this renewal.
- **CTA:** View billing -- deep-links to FEAT-14.SPEC-003 (Billing & Payment Management)

- **Title (downgrade/cancellation applied):** You're on the free plan
- **Body (downgrade/cancellation applied):** Your household moved to the free plan. Every past plan, rating, recipe, pantry item, and your shared list are still fully available.
- **CTA:** View plan -- deep-links to FEAT-14.SPEC-001 (Plan Tier Overview)

- **Title (grace-period retry succeeded):** Your payment went through
- **Body (grace-period retry succeeded):** Your subscription is active again -- no interruption to your paid features.
- **CTA:** View billing -- deep-links to FEAT-14.SPEC-003 (Billing & Payment Management)

- **Title (grace-period expired):** You're on the free plan
- **Body (grace-period expired):** We couldn't complete your renewal payment, so your household moved to the free plan. Every past plan, rating, recipe, pantry item, and your shared list are still fully available.
- **CTA:** View plan -- deep-links to FEAT-14.SPEC-001 (Plan Tier Overview)

**Email:**
- **Subject (upgrade):** You're subscribed to Plateful {billing_period}
- **Body (upgrade):**
  Hi {organiser_first_name},

  Your {billing_period} Plateful subscription is now active. Your first AI-generated weekly plan is on its way.

  Manage your billing anytime from your account.
- **CTA (button):** View billing -- deep-links to FEAT-14.SPEC-003 (Billing & Payment Management)

- **Subject (period switch):** Your Plateful billing period changed
- **Body (period switch):**
  Hi {organiser_first_name},

  You're now on {billing_period} billing for Plateful, effective this renewal.
- **CTA (button):** View billing -- deep-links to FEAT-14.SPEC-003 (Billing & Payment Management)

- **Subject (downgrade/cancellation applied):** Your Plateful household is now on the free plan
- **Body (downgrade/cancellation applied):**
  Hi {organiser_first_name},

  Your household has moved to the free Plateful plan. Every past plan, rating, recipe, pantry item, and your shared list are still fully available -- nothing has been removed.

  You can plan manually anytime, or upgrade again whenever you're ready.
- **CTA (button):** View plan -- deep-links to FEAT-14.SPEC-001 (Plan Tier Overview)

- **Subject (grace-period retry succeeded):** Your Plateful payment went through
- **Body (grace-period retry succeeded):**
  Hi {organiser_first_name},

  Your payment was successful and your subscription is active again. There's no interruption to your paid features.
- **CTA (button):** View billing -- deep-links to FEAT-14.SPEC-003 (Billing & Payment Management)

- **Subject (grace-period expired):** Your Plateful household is now on the free plan
- **Body (grace-period expired):**
  Hi {organiser_first_name},

  We weren't able to complete your renewal payment, so your household has moved to the free plan. Every past plan, rating, recipe, pantry item, and your shared list are still fully available -- nothing has been removed.

  You can resubscribe anytime.
- **CTA (button):** View plan -- deep-links to FEAT-14.SPEC-001 (Plan Tier Overview)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {billing_period} | Subscription -- billing_period | Yearly | Never empty -- every trigger for this notification fires only once billing_period is a concrete value (monthly or yearly) or, for downgrade/cancellation/grace-expiry, is not referenced in that variant's content |
| {organiser_first_name} | Member Profile -- display_name (of the organiser) | Maya | Greeting renders as "Hi there,"

## Delivery Rules

**Batching:** No batching -- each billing change produces exactly one notification instance, since a household's Subscription changes at most once per confirmed action and these confirmations are consequential enough to warrant individual delivery rather than being combined.
**Deduplication:** At most one confirmation per applied change. FEAT-14.SPEC-008 and FEAT-14.SPEC-007 each fire this notification exactly once per outcome they apply; a re-run of either automation for an already-applied change (per their own idempotency rules) never re-triggers this notification.
**Retry on failure:** Email delivery failure is retried up to 3 times over 6 hours, per FEAT-14.SPEC-012. After the final failure, the in-app confirmation stands as the delivery of record and no additional error is shown to Maya -- a billing confirmation must never generate an alarming failure message of its own.
**Expiry:** The in-app confirmation never expires undelivered -- it is delivered the next time Maya opens the product, since it reflects a durable state change (the current tier and billing_state) rather than a time-sensitive alert. The email variant follows FEAT-14.SPEC-012's own retry-and-give-up behavior; if it is never delivered, the in-app confirmation and the visible tier on FEAT-14.SPEC-001 remain the surviving record.

## Edge Cases

- **Household deleted between the change applying and delivery** -- The notification is cancelled silently on every channel; a confirmation about a household that no longer exists is never delivered.
- **Maya has no registered email on her Member Profile** -- The email channel is skipped for that delivery; the in-app confirmation still delivers as the primary record.
- **A downgrade confirmation and a period-switch confirmation would otherwise both fire for the same Subscription in the same processing window (a downgrade supersedes a pending period switch, per FEAT-14.SPEC-008)** -- Only the downgrade confirmation is sent; the period-switch confirmation is never fired for a switch that FEAT-14.SPEC-008 determined never applied.
- **Grace-period retry succeeds at effectively the same moment the grace-expiry reversion would otherwise have fired** -- Per FEAT-14.SPEC-007's own resolution, only one outcome is applied; only that outcome's confirmation content (retry-succeeded or expired) is sent, never both.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggered by (inbound) | Every applied tier or billing_period change fires this notification |
| FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) | Triggered by (inbound) | A grace-period retry success or expiry fires this notification |
| FEAT-14.SPEC-012 (Transactional Email Delivery (Billing)) | References (outbound) | The email channel is delivered through this integration |
| FEAT-14.SPEC-001 (Plan Tier Overview) | Navigation (outbound) | Upgrade, downgrade, and grace-expiry confirmations deep-link here |
| FEAT-14.SPEC-003 (Billing & Payment Management) | Navigation (outbound) | Period-switch and grace-retry-success confirmations deep-link here |

## Analytics and Success Signals

- **billing_confirmation_delivered** (channel: in_app / email; variant: upgrade / period_switch / downgrade / cancellation / grace_retry / grace_expiry) -- supports success-metrics.md: "Paying Household Retention"
- **billing_confirmation_cta_tapped** (channel; destination: plan_tier_overview / billing_management) -- supports success-metrics.md: "Paying Household Retention"
- **billing_confirmation_email_skipped** (reason: no_email_on_file) -- N/A -- no Stage 2 metric tracks skipped email deliveries; retained so silent delivery gaps remain observable

## Acceptance Criteria

**FEAT-14.SPEC-010-AC-01:** Given Maya's upgrade completes, when FEAT-14.SPEC-008 fires, then she receives an in-app notification titled "You're on the paid plan" and an email with the subject "You're subscribed to Plateful {billing_period}".

**FEAT-14.SPEC-010-AC-02:** Given Maya taps the upgrade confirmation's CTA, when she taps "View plan", then she lands on FEAT-14.SPEC-001.

**FEAT-14.SPEC-010-AC-03:** Given a scheduled period switch takes effect at renewal, when FEAT-14.SPEC-008 applies it, then Maya receives the period-switch confirmation naming the new billing period.

**FEAT-14.SPEC-010-AC-04:** Given a downgrade reversion completes at period end, when FEAT-14.SPEC-008 applies it, then Maya receives "You're on the free plan" stating that all past data remains fully available.

**FEAT-14.SPEC-010-AC-05:** Given a cancellation reversion completes at period end, when FEAT-14.SPEC-008 applies it, then Maya receives the same free-plan confirmation content as a downgrade.

**FEAT-14.SPEC-010-AC-06:** Given a grace-period retry succeeds, when FEAT-14.SPEC-007 clears the grace state, then Maya receives "Your payment went through" confirming no interruption to paid features.

**FEAT-14.SPEC-010-AC-07:** Given a grace period expires unresolved, when the household reverts to free, then Maya receives the grace-expiry confirmation stating the reversion and that all past data remains available.

**FEAT-14.SPEC-010-AC-08:** Given the email delivery for a confirmation fails 3 times over 6 hours, when the final retry fails, then no error is shown to Maya and the in-app confirmation stands as the record.

**FEAT-14.SPEC-010-AC-09:** Given a household is deleted between a change applying and this notification's delivery, when delivery would otherwise occur, then it is cancelled silently on every channel.

**FEAT-14.SPEC-010-AC-10:** Given a pending period switch is superseded by a confirmed downgrade in the same processing window, when FEAT-14.SPEC-008 resolves both, then only the downgrade confirmation is sent.

**FEAT-14.SPEC-010-AC-11:** Given a grace-period retry succeeds at effectively the same moment expiry would otherwise fire, when FEAT-14.SPEC-007 resolves the outcome, then only that single outcome's confirmation is sent, never both.

**FEAT-14.SPEC-010-AC-12:** Given Maya has no registered email on her Member Profile, when a billing change is confirmed, then only the in-app notification is delivered.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (in-app, email) | 2 |
| Trigger Paths | 5 | 5 |
| Preference States | 1 (no configurable preference -- always-on both channels) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |
