---
document_type: spec
spec_type: notification
spec_id: FEAT-32.SPEC-006
spec_name: Connection Status Notifications
spec_slug: connection-status-notifications
parent_feature: FEAT-32
parent_feature_name: Payment Account Connection
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

# Notification Spec: Connection Status Notifications

## Overview

**Name:** Connection Status Notifications
**ID:** FEAT-32.SPEC-006
**Type:** Notification
**Purpose:** Sends Nadia a confirmation email when her account connects and an alert email when the connection breaks or needs attention, so she learns about a change to her ability to accept payments even when she is away from the product.
**Parent Feature:** FEAT-32 -- Payment Account Connection

## Scope and Non-Goals

**In Scope:**
- The confirmation email sent when the connection reaches Connected
- The alert email sent when the connection reaches Needs attention
- Delivery, retry, and expiry behavior for both emails

**Non-Goals:**
- Deciding when the connection reaches Connected or Needs attention -- owned entirely by FEAT-32.SPEC-003 (Connection Status Sync); this spec begins where that automation's trigger fires.
- Notifying anyone about a failed or abandoned hand-off -- the Brief's Side-Effect Inventory routes that outcome to FEAT-32.SPEC-001's inline Error state, not to an email; a failed attempt that leaves the prior state intact is not the record-worthy change these two emails exist to announce.
- Notifying anyone about a disconnect -- product-features.md's Communications field for this feature names only the connected and needs-attention events as triggering email; Nadia performed the disconnect herself on FEAT-32.SPEC-001 and sees its confirmation there directly, with no separate email needed to tell her what she just did.
- Notifying Owen or Priya -- neither role has any access to this entity (Access Matrix: None for both); a connection-status email carries content about Nadia's own financial-account readiness that no client contact is entitled to see.
- In-app notification center delivery -- product-features.md's Communications field names only the email channel for this feature; an in-app surface is owned separately by In-App Notification Center (FEAT-29, Later phase).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to Nadia, when the connection reaches Connected or Needs attention | Nadia works from a laptop or desktop throughout her day and is not necessarily inside the product at the moment the payment-processing capability reports a change (user-persona.md, Behavioral Context); a needs-attention change in particular can silently block every client's ability to pay her until she notices, so email reaches her even when she is away |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Connection reaches Connected | FEAT-32.SPEC-003 (Connection Status Sync) | Fires when FEAT-32.SPEC-003 applies the Connected outcome, whether from an initial connect or a reconnect | Freelancer Account name and sign-in email |
| Connection reaches Needs attention | FEAT-32.SPEC-003 (Connection Status Sync) | Fires when FEAT-32.SPEC-003 applies the Needs attention outcome, whether degrading from Connected or failing during an initial connect | Freelancer Account name and sign-in email; the attention reason (non-empty in every case: the processor's specific reason verbatim, or, when FEAT-32.SPEC-003 converted a zero-methods readiness report into Needs attention, that spec's product-defined reason) |

## Audience and Preferences

**Recipients:** Nadia (the Freelancer) only. The Access Matrix entitles only Nadia to any content about this entity; Owen and Priya have None access, and Dana's read-only support-session view of connection status (FEAT-31) is a view surface, not a notification recipient path.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- always sent | -- | Always on | -- (both are transactional record emails, not optional notifications) |

Both emails are transactional and core to the record (XBR-30): a change to whether Nadia can accept payment is not something she can switch off, since it directly affects the "get paid faster" promise (BRIEF.md, Experience narrative) and the Payment Readiness Before First Invoice metric. Neither can be turned off through her notification preferences.

**Quiet Hours:** N/A -- the product defines quiet hours for optional, non-transactional notifications only (XBR-30); both emails are transactional and send immediately regardless of the time of day, consistent with how quickly a needs-attention state can affect a client trying to pay.

## Content Definition

**Email (Connected):**
- **Subject:** Your payment account is connected
- **Body:**
  Hi {nadia_first_name},

  Your payment account is connected and ready to accept payments on your invoices.
- **CTA (button):** View payment settings -- deep-links to FEAT-32.SPEC-001 (Payment Connection Screen)

**Email (Needs attention):**
- **Subject:** Action needed: your payment account needs attention
- **Body:**
  Hi {nadia_first_name},

  Your payment account needs attention: {attention_reason}

  Until this is resolved, online payments on your invoices are temporarily unavailable.
- **CTA (button):** Reconnect your account -- deep-links to FEAT-32.SPEC-001 (Payment Connection Screen)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {nadia_first_name} | Freelancer Account -- name (first name portion) | Nadia | Greeting renders as "Hi," |
| {attention_reason} | Payment Account Connection -- status (the Needs attention value's associated reason: the processor's reason verbatim, or FEAT-32.SPEC-003's product-defined zero-methods reason "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect.") | We need a copy of your government ID to verify your account. | Never empty -- FEAT-32.SPEC-003 discards a restriction event with no stated reason as malformed, and its zero-methods path supplies its own defined reason, so no Needs attention outcome is ever applied without one and this email's trigger never fires without one |

## Delivery Rules

**Batching:** None -- each status change (Connected, Needs attention) is a distinct, permanent account-readiness event confirmed by FEAT-32.SPEC-003, and each is sent individually as it happens. Since only one Payment Account Connection record exists per freelancer, there is never more than one pending instance of either email to batch.
**Deduplication:** At most one email per FEAT-32.SPEC-003 outcome application. FEAT-32.SPEC-003's own already-applied and ordering guards (its Edge Cases) are the deduplication boundary: a duplicate or stale underlying processor event that FEAT-32.SPEC-003 does not apply never produces a second email.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, this spec records a delivery-failure state for that email's status outcome, and FEAT-32.SPEC-001 surfaces it as the delivery warning banner it defines (text "We couldn't email you about this change to your payment account. The status shown below is current.", placed above the status line on the Connected or Needs attention state, with a "Dismiss" button), since this alert has no project to attach a warning to the way other transactional emails do. The warning clears when Nadia dismisses it, when FEAT-32.SPEC-003 applies a new status outcome, or when she disconnects; it is never raised for an email cancelled as superseded (Edge Cases).
**Expiry:** Neither email expires in the sense of becoming pointless to send late: the status each reports remains true until the next status change, so a delayed delivery (after retries) still carries accurate information whenever it lands. There is no withholding cutoff; the retry window above is the only limit, after which delivery is treated as failed (surfaced as a warning) rather than expired.

## Edge Cases

- **The connection returns to Connected while the Needs attention alert is still being retried** -- The pending Needs attention alert is cancelled rather than delivered late alongside a now-contradicting Connected confirmation; the Connected confirmation for the new outcome still sends on its own trigger. A stale "needs attention" email arriving after Nadia already sees "connected" would read as the product not paying attention.
- **The connection is disconnected before a pending Connected or Needs attention email is delivered** -- The pending email still sends: it was accurate at the moment FEAT-32.SPEC-003 applied that outcome, and a later disconnect (Nadia's own separate action) does not retroactively make it false, mirroring how FEAT-10.SPEC-007 treats a confirmation that predates a later reversal.
- **Two Needs attention outcomes are applied in succession with different reasons (the processor changes what it is asking for)** -- Each is a materially new instance of information Nadia needs, so each fires its own alert email rather than being batched or suppressed as a repeat.
- **Nadia's account is deleted (FEAT-24) while an email for either outcome is still queued for retry** -- The pending delivery is cancelled once account deletion finalizes and removes her Freelancer Account and its Notifications, per the dependency map's Notification lifecycle (Deleted by FEAT-24).
- **Quiet hours colliding with expiry** -- N/A, since both emails are transactional and exempt from quiet hours, so there is no quiet-hours hold to collide with an expiry cutoff.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-32.SPEC-003 (Connection Status Sync) | Triggered by (inbound) | The Connected and Needs attention outcomes fire this notification |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Navigation (outbound) | Both CTAs deep-link here |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Affects (outbound) | A final delivery failure surfaces as the delivery warning banner defined in SPEC-001's Layout and Content (text, placement above the status line, Dismiss, and clearing rules are specified there) |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | Underlying delivery, retry, and bounce/failure reporting capability this notification is sent through |

## Analytics and Success Signals

- **payment_account_connected_email_delivered** () -- supports success-metrics.md: "Payment Readiness Before First Invoice"
- **payment_account_needs_attention_email_delivered** () -- supports success-metrics.md: "Payment Readiness Before First Invoice"
- **payment_account_status_email_delivery_failed** (variant: connected / needs_attention, retry_count) -- N/A -- no Stage 2 metric measures this notification's own delivery-failure rate directly; retained as a standard delivery-quality signal.
- **payment_account_status_email_opened** (variant: connected / needs_attention) -- N/A -- no Stage 2 metric measures open rates for these specific emails; retained as a standard delivery-quality signal.

## Acceptance Criteria

**FEAT-32.SPEC-006-AC-01:** Given FEAT-32.SPEC-003 applies a Connected outcome, when this notification fires, then Nadia receives an email with subject "Your payment account is connected."

**FEAT-32.SPEC-006-AC-02:** Given FEAT-32.SPEC-003 applies a Needs attention outcome, when this notification fires, then Nadia receives an email with subject "Action needed: your payment account needs attention" carrying the processor's specific reason verbatim.

**FEAT-32.SPEC-006-AC-03:** Given Nadia opens her Connected confirmation email, when she taps "View payment settings," then she lands on FEAT-32.SPEC-001.

**FEAT-32.SPEC-006-AC-04:** Given Nadia opens her Needs attention alert email, when she taps "Reconnect your account," then she lands on FEAT-32.SPEC-001.

**FEAT-32.SPEC-006-AC-05:** Given Nadia has no way to opt out of either email, when her notification preferences are checked, then no preference control exists for either and both always send.

**FEAT-32.SPEC-006-AC-06:** Given a status change occurs at any hour, when this notification fires, then it sends immediately regardless of Nadia's configured quiet hours.

**FEAT-32.SPEC-006-AC-07:** Given delivery fails on the first attempt, when the retry logic runs, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, and after the final failure Nadia sees, above the status line on FEAT-32.SPEC-001, the banner "We couldn't email you about this change to your payment account. The status shown below is current." with a "Dismiss" button.

**FEAT-32.SPEC-006-AC-08:** Given a Needs attention alert is still being retried, when the connection returns to Connected before the retry succeeds, then the pending Needs attention alert is cancelled and the new Connected confirmation still sends.

**FEAT-32.SPEC-006-AC-09:** Given a Connected confirmation is queued, when the connection is later disconnected before that email is delivered, then the confirmation still sends, since it was accurate at the moment it was triggered.

**FEAT-32.SPEC-006-AC-10:** Given two Needs attention outcomes are applied in succession with different reasons, when each is applied, then each fires its own separate alert email.

**FEAT-32.SPEC-006-AC-11:** Given Nadia's account is deleted while an email is still queued for retry, when the deletion finalizes, then the pending delivery is cancelled.

**FEAT-32.SPEC-006-AC-12:** Given the same Connected outcome is not re-applied by FEAT-32.SPEC-003 for a duplicate or stale underlying event, when that duplicate event is discarded upstream, then this notification does not fire a second time.

**FEAT-32.SPEC-006-AC-13:** Given Owen or Priya is a contact at Nadia's client company, when a status change occurs on her connection, then neither receives any copy of either email.

**FEAT-32.SPEC-006-AC-14:** Given Nadia's connection is disconnected by her own action on FEAT-32.SPEC-001, when the disconnect completes, then neither of this spec's emails fires, since disconnect is not one of this spec's triggers.

**FEAT-32.SPEC-006-AC-15:** Given the delivery warning banner is showing on FEAT-32.SPEC-001 after a final delivery failure, when Nadia taps "Dismiss," or FEAT-32.SPEC-003 applies a new status outcome, or she disconnects, then the banner clears and does not return for that outcome.

**FEAT-32.SPEC-006-AC-16:** Given FEAT-32.SPEC-003 converts a readiness report with zero available payment methods into Needs attention, when this notification fires, then the alert email's `{attention_reason}` reads "Your payment account has no payment methods turned on yet. Turn on card or bank transfer in your payment account, then reconnect." and is never empty.

**FEAT-32.SPEC-006-AC-17:** Given a Needs attention alert is cancelled as superseded by a return to Connected, when the cancellation occurs, then no delivery warning is shown on FEAT-32.SPEC-001.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 2 | 2 |
| Preference States | 1 (always on -- no preference exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
