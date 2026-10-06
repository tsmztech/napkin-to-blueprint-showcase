---
document_type: spec
spec_type: notification
spec_id: FEAT-18.SPEC-007
spec_name: Subscription Billing Notifications
spec_slug: subscription-billing-notifications
parent_feature: FEAT-18
parent_feature_name: Pro Subscription Billing & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Notification Spec: Subscription Billing Notifications

## Overview

**Name:** Subscription Billing Notifications
**ID:** FEAT-18.SPEC-007
**Type:** Notification
**Purpose:** Sends Talia the payment-failure grace notice, renewal receipt, cancellation confirmation, and price-change notice, so she is never surprised by her own billing.
**Parent Feature:** FEAT-18 -- Pro Subscription Billing & Account Management

## Scope and Non-Goals

**In Scope:**
- The payment-failure grace notice, with the exact grace deadline
- The renewal receipt, sent on every successful monthly charge
- The cancellation confirmation, with the exact period-end date
- The price-change notice, sent at least 30 days (platform parameter: `subscription-price-change-notice-days`) before a price change applies

**Non-Goals:**
- Deciding when a renewal succeeds, fails, or a grace period expires -- owned by FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing); this spec begins where that automation's outcome fires
- The Pro Account pause itself -- owned by FEAT-18.SPEC-004; a pause is not separately notified by this spec beyond what the payment-failure grace notice already covers, since the grace notice is the Pro's warning before any pause occurs
- Notifying clients about anything related to the Pro's own billing -- excluded per the Access Matrix: clients have no visibility into Subscription & Billing at all; nothing in this spec is ever sent to a Client
- The underlying text/email delivery mechanics (send, retry, delivery-status reporting) -- owned by FEAT-08.SPEC-012 (text) and FEAT-08.SPEC-013 (email), per the dependency map's External Touchpoints table; this spec defines only the content, audience, and delivery rules specific to these four messages

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, for all four notices | Talia checks Chairtime reactively throughout her working day; an in-app notice is always available on her next open, matching her behavioral context |
| Text | Always, when Talia's notification preferences (FEAT-27) include text delivery for Pro notifications | Talia works with her phone in hand between clients; a billing issue -- especially the payment-failure grace notice -- needs to reach her promptly even when she is not inside the app |
| Email | Always, as the fallback when text is not enabled in her preferences, and always in addition to text for the renewal receipt and cancellation confirmation (financial records worth keeping in an inbox) | Per BRIEF.md's stated fallback pattern (ASMP-32) and because a receipt or cancellation confirmation is the kind of record a Pro may want to search for later, unlike a transient reminder |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Renewal charge fails | FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Fires when a renewal charge is confirmed failed and the grace period starts | Subscription reference, grace deadline, price |
| Renewal charge succeeds | FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Fires on every confirmed successful renewal charge | Subscription reference, price charged, new next_billing_date |
| Cancellation confirmed | FEAT-18.SPEC-006 (Subscription Billing Integration) | Fires when the payment-processing capability confirms the subscription will stop billing at period end | Subscription reference, period-end date |
| Price change decided | Product-level decision (FEAT-18.SPEC-005's price-change notice rule) | Fires when a future price change is decided and must be announced at least the fixed notice window ahead | Subscription reference, current price, new price, effective date |

## Audience and Preferences

**Recipients:** The Pro (Talia) -- the sole recipient of every notice in this spec, traced to the Access Matrix (Subscription & Billing = Full for the Pro, None for the Client, View for Platform Operator Support). Support never receives these notices; Support's visibility into billing is the read-only screen (FEAT-18.SPEC-002), not a notification feed.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Pro notification channels (text / email / in-app) | Any combination, in-app always on | Text + in-app (email as fallback if texting is unavailable) | FEAT-27 (Pro Profile & Booking Page Settings, notification preferences) |

There is no separate off switch for these four billing notices: per FEAT-18.SPEC-005's business rules, a payment failure, a successful renewal, a cancellation, and a price change are each consequential enough to Talia's business that the in-app copy is always delivered regardless of her channel preferences; only the choice of text vs. email as the secondary channel is hers to set.

**Quiet Hours:** N/A -- these are billing-status notices tied to financial events (a failed charge, a successful charge, a cancellation, a price change), not time-sensitive reminders bound to a daily schedule; XBR-16's quiet-hours window governs client-facing appointment reminders, not the Pro's own billing notices, so none of the four messages in this spec is held for quiet hours.

## Content Definition

**Payment-Failure Grace Notice:**
- **In-app title:** Payment failed -- update by {grace_deadline}
- **In-app body:** Your subscription payment didn't go through. Update your payment method by {grace_deadline} to keep your booking link live.
- **Text:** "Chairtime: your subscription payment failed. Update your payment method by {grace_deadline} to keep taking bookings: {manage_link}"
- **Email subject:** Action needed: update your Chairtime payment method by {grace_deadline}
- **Email body:**
  Hi {pro_first_name},

  Your latest subscription payment didn't go through. To keep your booking link live and taking new bookings, update your payment method by {grace_deadline}.

  Your existing bookings, reminders, and client access are unaffected either way.
- **CTA:** Update payment method -- deep-links to FEAT-18.SPEC-002 (Billing & Subscription Management Screen)

**Renewal Receipt:**
- **In-app title:** Subscription renewed
- **In-app body:** Your {price} monthly payment went through. Next billing date: {next_billing_date}.
- **Email subject:** Your Chairtime subscription receipt
- **Email body:**
  Hi {pro_first_name},

  This confirms your Chairtime subscription renewed successfully.

  Amount charged: {price}
  Next billing date: {next_billing_date}

  Chairtime takes nothing from your deposits, balances, or tips -- this is the only charge on your account.
- **CTA:** View billing -- deep-links to FEAT-18.SPEC-002 (Billing & Subscription Management Screen)

**Cancellation Confirmation:**
- **In-app title:** Subscription cancelled
- **In-app body:** Your subscription is cancelled. It stays active through {period_end_date}, then your booking link pauses.
- **Text:** "Chairtime: your subscription is cancelled, active through {period_end_date}. Your booking link pauses after that."
- **Email subject:** Your Chairtime subscription is cancelled
- **Email body:**
  Hi {pro_first_name},

  This confirms your Chairtime subscription is cancelled. It remains active through {period_end_date} -- you keep full use of your booking link until then.

  After {period_end_date}, your booking link will pause to new bookings. You can resubscribe at any time.
- **CTA:** View billing -- deep-links to FEAT-18.SPEC-002 (Billing & Subscription Management Screen)

**Price-Change Notice:**
- **In-app title:** Your subscription price is changing on {effective_date}
- **In-app body:** Starting {effective_date}, your monthly price changes from {current_price} to {new_price}.
- **Text:** "Chairtime: your subscription price changes from {current_price} to {new_price} starting {effective_date}."
- **Email subject:** Your Chairtime subscription price is changing
- **Email body:**
  Hi {pro_first_name},

  We're writing to let you know your Chairtime subscription price will change from {current_price} to {new_price}, starting {effective_date}.

  This is your only notice before the change applies -- at least {subscription-price-change-notice-days} days ahead, as always. Everything else about your subscription stays the same.
- **CTA:** View billing -- deep-links to FEAT-18.SPEC-002 (Billing & Subscription Management Screen)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_first_name} | Pro Account -- display_name (first token) | Talia | Greeting renders as "Hi," |
| {grace_deadline} | Subscription -- derived, failure date plus platform parameter: `subscription-payment-failure-grace-period-days` | March 14 | Never empty -- computed at the moment the grace notice fires, per FEAT-18.SPEC-003 |
| {price} | Subscription -- price (platform parameter: `subscription-price`) | $39/month (illustrative only) | Never empty -- one price for every Pro |
| {next_billing_date} | Subscription -- next_billing_date | April 14 | Never empty -- set by the renewal automation before this notice fires |
| {period_end_date} | Subscription -- derived, the date the already-paid period ends | May 1 | Never empty -- computed by FEAT-18.SPEC-005's cancellation-timing rule before this notice fires |
| {manage_link} | Access reference into FEAT-18.SPEC-002, delivered via FEAT-08.SPEC-012/FEAT-08.SPEC-013 | (rendered as a tappable link) | Never empty -- generated at send time |
| {current_price} / {new_price} | Subscription -- price (before) / product-level decided new price (both platform parameter: `subscription-price`, before and after the change) | $39/month / $45/month | Never empty -- both values are fixed at the moment the price change is decided |
| {effective_date} | Product-level decision -- the date the new price applies | May 1 | Never empty -- set when the price change is decided, at least platform parameter: `subscription-price-change-notice-days` in advance |

## Delivery Rules

**Batching:** None of these four notices are ever batched together or with any other notification -- each is a distinct, individually consequential financial event and is delivered as its own message the moment its trigger fires.
**Deduplication:** At most one instance of each notice per triggering event. A renewal-outcome event delivered twice (per FEAT-18.SPEC-006's Edge Cases) produces no second notice, since the underlying Subscription state does not change on the duplicate delivery. A price-change notice is sent exactly once per decided change, keyed to the specific effective_date.
**Retry on failure:** Text delivery failure is retried per platform parameter: `message-delivery-retry-count`, then falls back to email as the delivery of record, consistent with FEAT-08.SPEC-009's product-wide retry-and-fallback rule; the in-app notice always stands regardless of text/email outcome and is never itself retried (it is delivered the next time Talia opens the product).
**Expiry:** None of these four notices expire undelivered in the ordinary sense -- the in-app notice remains visible until Talia views it, and the underlying billing state it describes (grace deadline, cancellation date, price-change date) is also always visible on FEAT-18.SPEC-002, so a delayed text or email never leaves Talia with no way to learn the same information.

## Edge Cases

- **The underlying Subscription changes state before an already-queued notice is delivered (e.g., Talia updates her payment method moments after the payment-failure notice fires, before the text is sent)** -- The already-triggered payment-failure grace notice is still delivered as queued (it accurately reported the failure that just happened); no separate "never mind" message is sent, but the subsequent renewal-recovered outcome (FEAT-18.SPEC-003) triggers no confusing follow-up beyond the ordinary path -- if the retry succeeds, no renewal receipt is sent for a retry succeeding during grace in this spec's inventory (the recovered state is reflected on FEAT-18.SPEC-002 directly), avoiding a contradictory pair of messages.
- **Talia's notification preferences (FEAT-27) are set to no channels beyond in-app between trigger and delivery** -- Per Audience and Preferences, the in-app copy always delivers regardless of her channel preference; only text vs. email as the secondary channel is affected, so she never receives zero notice.
- **A price-change notice and a payment-failure grace notice are both pending for the same Subscription at the same time** -- Each is delivered independently, per FEAT-18.SPEC-005's business rule that these two timers do not interact; Talia may receive both notices in the same period without either being suppressed.
- **The renewal receipt's amount does not match what Talia expects because a price change took effect since her last renewal** -- The receipt always reflects the price actually charged, which is the Subscription's own price field as updated by FEAT-18.SPEC-003 on the price change's effective_date (to the new platform parameter: `subscription-price` value); because that effective_date always falls at least the required notice window after this notice was sent, the new amount on the receipt was never a surprise by the time it was first charged.
- **Cancellation confirmation is triggered by account closure (FEAT-29, XBR-20) rather than Talia's own action on FEAT-18.SPEC-002** -- The same cancellation confirmation content and delivery rules apply regardless of which flow invoked the cancellation; the notice's CTA still deep-links to FEAT-18.SPEC-002, even though Talia is mid-closure, since the billing screen remains the accurate source of her subscription's final status.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Triggered by (inbound) | Renewal failure and renewal success each fire their respective notice |
| FEAT-18.SPEC-006 (Subscription Billing Integration) | Triggered by (inbound) | Cancellation confirmation fires the cancellation notice |
| FEAT-18.SPEC-005 (Subscription Billing Rules) | Triggered by (inbound) | The price-change notice rule fires the price-change notice |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | Navigation (outbound) | Every notice's CTA deep-links here |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (inbound) | The Pro notification channel preference governs the secondary channel |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Triggers (outbound) | Carries the text variant of each notice |
| FEAT-08.SPEC-013 (Transactional Email Capability) | Triggers (outbound) | Carries the email variant of each notice, and the fallback when text is not enabled |

## Analytics and Success Signals

- **billing_notification_delivered** (notice_type: payment_failure_grace / renewal_receipt / cancellation_confirmation / price_change; channel) -- supports success-metrics.md: "Subscription Retention"
- **billing_notification_cta_tapped** (notice_type; destination: FEAT-18.SPEC-002) -- supports success-metrics.md: "Subscription Retention"
- **payment_failure_grace_notice_to_recovery_time** (days between notice delivery and payment method resolution, where resolved) -- supports success-metrics.md: "Subscription Retention"

## Acceptance Criteria

**FEAT-18.SPEC-007-AC-01:** Given Talia's renewal charge fails, when the grace period starts, then she receives the payment-failure grace notice in-app and on her preferred secondary channel, stating the exact grace deadline.

**FEAT-18.SPEC-007-AC-02:** Given Talia's renewal charge succeeds, when the charge is confirmed, then she receives the renewal receipt in-app and by email, stating the exact amount charged and her next billing date.

**FEAT-18.SPEC-007-AC-03:** Given Talia confirms cancellation, when the cancellation is confirmed by the payment-processing capability, then she receives the cancellation confirmation stating the exact date her subscription remains active through.

**FEAT-18.SPEC-007-AC-04:** Given a price change is decided for Talia's subscription, when the notice fires, then she receives it at least the fixed notice window before the new price applies, stating the current price, new price, and effective date.

**FEAT-18.SPEC-007-AC-05:** Given Talia has set her notification channels to email only, when a payment failure occurs, then she still receives the in-app notice and the email notice, with no text sent.

**FEAT-18.SPEC-007-AC-06:** Given Talia taps the CTA on any of these four notices, when she taps "View billing" or "Update payment method", then she lands on FEAT-18.SPEC-002 (Billing & Subscription Management Screen).

**FEAT-18.SPEC-007-AC-07:** Given a renewal-outcome event is delivered twice for the same charge, when the second delivery arrives, then no duplicate notice is sent.

**FEAT-18.SPEC-007-AC-08:** Given Talia's text delivery of the payment-failure grace notice fails, when the retry attempts are exhausted, then the notice falls back to email as the delivery of record, and the in-app notice still stands regardless.

**FEAT-18.SPEC-007-AC-09:** Given Talia has both a pending price-change notice and an active payment-failure grace period, when each notice's own trigger fires, then both are delivered independently without either being suppressed.

**FEAT-18.SPEC-007-AC-10:** Given Talia's account closure (FEAT-29) invokes cancellation rather than her own action on FEAT-18.SPEC-002, when the cancellation is confirmed, then she still receives the same cancellation confirmation content with a CTA back to FEAT-18.SPEC-002.

**FEAT-18.SPEC-007-AC-11:** Given a support operator is viewing a Pro's account, when any of these four notices fires, then it is never sent to Support -- these notices are delivered only to the Pro.

**FEAT-18.SPEC-007-AC-12:** Given Talia's subscription renewal fails at a moment that falls late in the day, when the payment-failure grace notice fires, then it is delivered immediately with no quiet-hours hold, since these billing notices are exempt from the appointment-reminder quiet-hours window (XBR-16 governs client reminders, not Pro billing notices).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 3 (in-app, text, email) | 3 |
| Trigger Paths | 4 | 4 |
| Preference States | 2 (default text+in-app, email-only fallback) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
