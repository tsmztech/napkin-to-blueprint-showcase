---
document_type: spec
spec_type: notification
spec_id: FEAT-23.SPEC-008
spec_name: Plan & Billing Notifications
spec_slug: plan-billing-notifications
parent_feature: FEAT-23
parent_feature_name: Subscription Plan & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 18
---

# Notification Spec: Plan & Billing Notifications

## Overview

**Name:** Plan & Billing Notifications
**ID:** FEAT-23.SPEC-008
**Type:** Notification
**Purpose:** Sends Nadia an email confirmation on every plan change (upgrade, downgrade, cancellation, lapse) and an alert when a subscription charge fails.
**Parent Feature:** FEAT-23 -- Subscription Plan & Billing Management

## Scope and Non-Goals

**In Scope:**
- The plan-change confirmation email, sent on upgrade, downgrade (Paid to Free, effective immediately), cancellation confirmed, paid plan ended at period end (moved to Free), and lapse
- The failed-charge alert email, sent when a renewal charge on a Paid plan first fails (never for a failed subscribe attempt or a rejected downgrade, which are shown inline and change nothing)
- Delivery, retry, and expiry behavior for both, using the transactional email delivery capability (FEAT-14.SPEC-001)

**Non-Goals:**
- Routine renewal notifications -- excluded per FEAT-23.SPEC-004's Business Rules: a successful renewal is silent by design; only a tier or status change is worth interrupting Nadia for.
- The mechanics of the transactional email delivery capability itself -- owned by FEAT-14.SPEC-001 (Transactional Email Delivery), which this spec uses without duplicating its contract.
- General account notification preferences (turning all email off, quiet hours across every feature) -- owned by FEAT-21.SPEC-002 (Notification Preferences); this spec defines only what is specific to plan and billing events, both of which are transactional and always send (XBR-30, via FEAT-21.SPEC-008).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, on every plan-change and failed-charge event | Nadia may not be inside the product at the moment her billing state changes (a renewal failure, a period ending); a billing outcome that affects whether she can add clients must reach her even when she is away from the product (ASMP-26: failed deliveries must be surfaced within minutes, not lost) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Plan tier or status changes (upgrade, downgrade, cancellation confirmed, paid plan ended at period end, or lapse) | FEAT-23.SPEC-004 (Plan State Sync), FEAT-23.SPEC-006 (Cancel Subscription) | Fires on every tier/status change this spec's In Scope covers -- never on routine renewal, never on a charge recovered by a successful retry, and never on a failed subscribe attempt or rejected downgrade. A downgrade confirmed while the live client count is over the free-tier limit results in Lapsed and sends the lapse email instead of the downgrade email | Plan reference, prior tier/status, new tier/status, billing_cycle, period end date (cancellation and period end), failure reason (lapse only, if applicable) |
| A renewal charge first fails | FEAT-23.SPEC-003 (Subscription Billing Processing), via FEAT-23.SPEC-004 | Fires once when a Paid, Active plan's renewal charge does not complete and status becomes Charge failed; not fired for failed retries within the same grace window, and not for a subscribe or downgrade attempt | Plan reference, specific failure reason, grace-window end date |

## Audience and Preferences

**Recipients:** Nadia (Freelancer) -- the sole recipient per the Access Matrix in user-persona.md; this is her own billing relationship, never visible to Owen, Priya, or Dana by email (Dana's status-only view is inside a support session, per FEAT-31, and carries no email of its own).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Plan-change and failed-charge emails | Always on (transactional -- cannot be turned off) | On | N/A -- FEAT-21.SPEC-002 (Notification Preferences) lists this as a locked-on transactional category rather than a toggle, per FEAT-21.SPEC-008 (Notification Preference Rules) and XBR-30 |

**Quiet Hours:** N/A -- these are transactional, account-standing emails (a billing outcome affecting whether Nadia can add clients), exempt from any quiet-hours hold, consistent with how FEAT-21.SPEC-008 treats every transactional record email.

## Content Definition

**Email (plan-change confirmation -- upgrade):**
- **Subject:** Your Clientroom plan is now {new_tier_label}
- **Body:**
  Hi {nadia_first_name},

  Your Clientroom plan has changed to {new_tier_label} ({billing_cycle_label}). You can now have up to {new_client_capacity} active clients.

  Review your plan and billing details any time.
- **CTA (button):** View plan -- deep-links to FEAT-23.SPEC-001 (Plan & Billing Screen)

**Email (plan-change confirmation -- downgrade):**
- **Subject:** Your Clientroom plan is now {new_tier_label}
- **Body:**
  Hi {nadia_first_name},

  Your Clientroom plan has changed to {new_tier_label}, effective now. Billing has stopped and you won't be charged again. Your active client limit is now {new_client_capacity}.

  Your existing clients and their portals are untouched. Review your plan and billing details any time.
- **CTA (button):** View plan -- deep-links to FEAT-23.SPEC-001

**Email (plan-change confirmation -- cancellation confirmed):**
- **Subject:** Your Clientroom plan cancellation is confirmed
- **Body:**
  Hi {nadia_first_name},

  Your subscription is cancelled. Your plan stays fully active through {period_end_date}. After that, it moves to the free tier or lapses depending on your active client count at that time.

  You can resubscribe at any point.
- **CTA (button):** View plan -- deep-links to FEAT-23.SPEC-001

**Email (plan-change confirmation -- paid plan ended at period end):**
- **Subject:** Your Clientroom paid plan has ended
- **Body:**
  Hi {nadia_first_name},

  Your paid plan ended on {plan_end_date}, as you asked when you cancelled. You're now on the Free tier, with room for up to {free_tier_client_limit} active clients. Nothing has been lost -- your existing clients and their portals are untouched.

  You can subscribe again at any point.
- **CTA (button):** View plan -- deep-links to FEAT-23.SPEC-001 (Plan & Billing Screen)

Sent when FEAT-23.SPEC-004 applies a period end with the client count within the free-tier limit (tier Free, status Active). When the count is over the limit the period end results in Lapsed and the lapse email below is sent instead; exactly one email is sent for a given period end. A cancellation therefore produces two emails over time by design: the cancellation-confirmed email when Nadia cancels, and this one (or the lapse email) on the day the plan actually ends.

**Email (plan-change confirmation -- lapsed):**
- **Subject:** Your Clientroom plan has lapsed
- **Body:**
  Hi {nadia_first_name},

  Your plan has lapsed as of {lapse_date}. Nothing has been lost -- your existing clients and their portals remain reachable. Adding or reactivating clients beyond {free_tier_client_limit} is on hold until you upgrade again.
- **CTA (button):** Upgrade -- deep-links to FEAT-23.SPEC-001 (the Subscribe action)

**Email (failed-charge alert):**
- **Subject:** We couldn't process your Clientroom subscription charge
- **Body:**
  Hi {nadia_first_name},

  Your subscription charge didn't go through: {failure_reason}. Your plan and client access are unaffected for now -- you have until {grace_window_end_date} to retry before your plan lapses.
- **CTA (button):** Retry now -- deep-links to FEAT-23.SPEC-001 (the Retry action)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|----------------|-----------------------|
| {nadia_first_name} | Freelancer Account -- first name | Priya | Greeting renders as "Hi," |
| {new_tier_label} | Subscription Plan -- tier (rendered as "Paid" or "Free") | Paid | Never empty -- tier is always set on the plan record |
| {billing_cycle_label} | Subscription Plan -- billing_cycle (rendered as "billed monthly" or "billed yearly") | billed yearly | Line is omitted when billing_cycle is unset (e.g., downgrade to Free -- not applicable there since that email variant omits the clause) |
| {new_client_capacity} | Derived -- platform parameter: `free-tier-active-client-limit` for Free, unlimited for Paid (rendered as "unlimited" on Paid) | unlimited | Never empty -- always derivable from tier |
| {period_end_date} | Subscription Plan -- current billing period's end date, reported by the subscription-billing capability (FEAT-23.SPEC-003) | March 14, 2027 | Never empty -- a cancellation is only recorded against an active billing period that has a known end date |
| {plan_end_date} | Subscription Plan -- the period end date the capability reported, as applied by FEAT-23.SPEC-004 | March 14, 2027 | Never empty -- the period-end event carries its own date |
| {lapse_date} | Derived -- the date this spec's trigger event was applied | September 27, 2026 | Never empty -- the lapse event itself carries its own timestamp |
| {free_tier_client_limit} | Derived -- platform parameter: `free-tier-active-client-limit` | 2 | Never empty -- a fixed platform value |
| {failure_reason} | Subscription Plan -- last failure reason, reported by the subscription-billing capability (FEAT-23.SPEC-003) | Your card was declined | "a billing issue" (generic fallback if no specific reason was reported) |
| {grace_window_end_date} | Derived -- first-failure timestamp plus platform parameter: `subscription-charge-grace-window-days` | October 4, 2026 | Never empty -- calculable the moment the failure is recorded |

## Delivery Rules

**Batching:** None -- each plan-change or failed-charge event is delivered as its own, individual email the moment it is recorded; these are infrequent, high-importance events that are never worth collapsing together.
**Deduplication:** At most one email per distinct tier/status-change event or per distinct charge-failure event. A charge-succeeded event already applied and reported does not re-fire its confirmation if the same event is delivered twice to FEAT-23.SPEC-004 (per that spec's own deduplication).
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning inside the product (XBR-30); the underlying plan/status change itself remains visible on the Plan & Billing Screen (FEAT-23.SPEC-001) regardless of this email's delivery outcome.
**Expiry:** These emails never expire undelivered in the ordinary sense -- because the underlying tier/status change or failure reason remains visible on the Plan & Billing Screen indefinitely, a late-delivered confirmation still carries accurate, current information; delivery is retried until it succeeds or the retry count is exhausted, never abandoned as "too late to matter."

## Edge Cases

- **Nadia's account is deleted (FEAT-24) before a pending confirmation email is delivered** -- The pending delivery is cancelled silently; an email about a plan that no longer exists is never sent.
- **Nadia's subscribe attempt fails, or her downgrade request is rejected** -- No email is sent: the outcome is shown inline on the Plan & Billing Screen (FEAT-23.SPEC-001), the plan is unchanged, and no grace window exists that an alert could point to.
- **A cancelled plan reaches its period end** -- One email is sent for that period end: the paid-plan-ended email (Free) or the lapse email (Lapsed), in addition to the cancellation-confirmed email sent earlier when Nadia cancelled.
- **A failed-charge alert is still pending delivery when the retry succeeds** -- The pending failed-charge alert is cancelled if it has not yet sent, and the plan-change confirmation for the successful retry (status returning to Active, no tier change) is not sent either, since a return to the prior status without a tier change is not itself a plan change worth confirming; only a genuine tier/status change (upgrade, downgrade, cancellation, or lapse) triggers this spec.
- **A plan lapses and Nadia immediately re-subscribes before the lapse email is delivered** -- Both emails are sent independently in their own right (the lapse happened and is worth confirming; the upgrade is a separate, later event), since each documents a real state the plan passed through -- this reflects the product's commitment to a truthful, evidentiary trail rather than suppressing history for tidiness.
- **The same failure reason repeats across multiple retry attempts within the grace window** -- Only the first failed-charge alert for a given grace window is sent; repeated retry failures within the same still-open grace window do not generate additional alerts, since Nadia already has the grace-window end date and a working Retry action from the first alert.
- **Email delivery fails on all retries for a lapse confirmation** -- The lapse itself is fully recorded and visible on the Plan & Billing Screen (FEAT-23.SPEC-001) regardless of the email's fate; a delivery warning is added to Nadia's account per XBR-30, so the change is never silently lost even if the email never arrives.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-004 (Plan State Sync) | Triggered by (inbound) | Every tier/status change (except renewal) and every charge-failed event fires this notification |
| FEAT-23.SPEC-006 (Cancel Subscription) | Triggered by (inbound) | A recorded cancellation fires the cancellation-confirmed email |
| FEAT-23.SPEC-001 (Plan & Billing Screen) | Navigation (outbound) | Every CTA deep-links here |
| FEAT-14.SPEC-001 (Transactional Email Delivery, FEAT-14) | Triggers (outbound) | This spec's emails are sent through that capability, which reports delivery and bounce status back |
| FEAT-21.SPEC-008 (Notification Preference Rules, FEAT-21) | References (inbound) | Confirms these emails are locked-on transactional emails per XBR-30, never optional |

## Analytics and Success Signals

- **plan_change_email_sent** (change_type: upgrade / downgrade / cancellation / plan_ended / lapse) -- supports success-metrics.md: "Free-to-Paid Conversion" (documents the confirmed outcomes the conversion metric tracks)
- **failed_charge_alert_sent** (failure_reason) -- N/A -- no Stage 2 metric measures alert volume directly; retained so failure-alert delivery is observable alongside FEAT-23.SPEC-003's own subscription_charge_failed event.
- **plan_change_email_delivery_failed** (change_type) -- N/A -- no Stage 2 metric covers email-delivery reliability for this feature specifically; ASMP-26's minutes-not-lost commitment is the product-level requirement this event supports observability for.

## Acceptance Criteria

**FEAT-23.SPEC-008-AC-01:** Given Nadia's plan upgrades to Paid, when the upgrade is applied (FEAT-23.SPEC-004), then she receives an email "Your Clientroom plan is now Paid" naming her new client capacity.

**FEAT-23.SPEC-008-AC-02:** Given Nadia accepts a downgrade and it is applied (tier Free, status Active), when FEAT-23.SPEC-004 commits it, then she receives "Your Clientroom plan is now Free" stating the change is effective now, billing has stopped, and naming her new client limit.

**FEAT-23.SPEC-008-AC-03:** Given Nadia cancels her subscription, when the cancellation is recorded (FEAT-23.SPEC-006), then she receives "Your Clientroom plan cancellation is confirmed" stating the exact date her plan stays active through.

**FEAT-23.SPEC-008-AC-04:** Given Nadia's plan lapses, when the lapse is applied, then she receives "Your Clientroom plan has lapsed" stating that no data is lost and existing clients remain reachable.

**FEAT-23.SPEC-008-AC-05:** Given a Paid, Active plan's renewal charge fails, when the failure is recorded, then Nadia receives "We couldn't process your Clientroom subscription charge" naming the specific reason and the grace-window end date.

**FEAT-23.SPEC-008-AC-06:** Given Nadia's plan renews successfully, when the renewal event is applied, then no confirmation email is sent for it.

**FEAT-23.SPEC-008-AC-07:** Given Nadia's account is deleted before a pending confirmation email is delivered, when the deletion completes, then the pending email is cancelled and never sent.

**FEAT-23.SPEC-008-AC-08:** Given a failed-charge alert is pending delivery, when Nadia's retry succeeds before it sends, then the pending alert is cancelled and no confirmation email is sent for the return to Active status alone.

**FEAT-23.SPEC-008-AC-09:** Given Nadia's plan lapses and she immediately re-subscribes, when both events are recorded, then she receives both the lapse email and the upgrade confirmation email, each independently.

**FEAT-23.SPEC-008-AC-10:** Given a charge fails twice within the same still-open grace window, when the second failure is recorded, then no second failed-charge alert is sent.

**FEAT-23.SPEC-008-AC-11:** Given the lapse confirmation email fails delivery on every retry, when the final retry fails, then the lapse itself remains fully visible on the Plan & Billing Screen and a delivery warning is recorded per XBR-30.

**FEAT-23.SPEC-008-AC-12:** Given these are transactional emails, when Nadia looks in her Notification Preferences (FEAT-21.SPEC-002), then plan-change and failed-charge emails show as always-on with no toggle.

**FEAT-23.SPEC-008-AC-13:** Given a plan-change email is triggered during Nadia's own quiet hours preference window (set for other notification types), when it is due to send, then it sends immediately regardless, since these emails are exempt from quiet hours.

**FEAT-23.SPEC-008-AC-14:** Given a cancelled plan reaches its period end with Nadia's active-client count within the free-tier limit, when FEAT-23.SPEC-004 applies it, then she receives "Your Clientroom paid plan has ended" stating the {plan_end_date}, that she is now on the Free tier with room for {free_tier_client_limit} active clients, and that nothing was lost.

**FEAT-23.SPEC-008-AC-15:** Given a cancelled plan reaches its period end with the count over the free-tier limit, when FEAT-23.SPEC-004 applies it, then she receives only the lapse email, not the paid-plan-ended email.

**FEAT-23.SPEC-008-AC-16:** Given Nadia cancelled earlier and her plan later reaches its period end, when both events have been applied, then she has received two emails in total for the cancellation journey: the cancellation-confirmed email at cancellation and one period-end email (paid-plan-ended or lapse) at the end.

**FEAT-23.SPEC-008-AC-17:** Given Nadia's subscribe charge fails or her downgrade request is rejected, when the outcome is reported, then no email is sent and the reason appears inline on the Plan & Billing Screen only.

**FEAT-23.SPEC-008-AC-18:** Given a failed retry occurs within an open grace window, when FEAT-23.SPEC-004 records it, then no further failed-charge alert is sent, and a successful retry sends no plan-change email.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 2 | 2 |
| Preference States | 1 (locked-on transactional) | 1 |
| Delivery Rules | 4 | 4 |
| Edge Cases | 7 | 7 |
