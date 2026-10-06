---
document_type: spec
spec_type: notification
spec_id: FEAT-10.SPEC-007
spec_name: Payment Confirmation Notification
spec_slug: payment-confirmation-notification
parent_feature: FEAT-10
parent_feature_name: Invoice Payment Processing
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Notification Spec: Payment Confirmation Notification

## Overview

**Name:** Payment Confirmation Notification
**ID:** FEAT-10.SPEC-007
**Type:** Notification
**Purpose:** Sends a confirmation email to Owen and Nadia the moment a card payment or a confirmed bank transfer succeeds, so both sides have the same timestamped record without checking the portal.
**Parent Feature:** FEAT-10 -- Invoice Payment Processing

## Scope and Non-Goals

**In Scope:**
- The confirmation email sent to Owen and to Nadia when a card payment or a confirmed bank transfer succeeds
- The delayed-delivery rule for a bank transfer -- no receipt sends until the processor confirms it
- Preference, retry, and expiry behavior for this confirmation

**Non-Goals:**
- Confirming a manually recorded off-platform payment -- product-features.md's Communications field for this feature names only "a payment succeeds" (the processor-confirmed path) as triggering this email; Nadia's manual record is her own action and needs no confirmation email to herself, and Owen made no in-portal payment to confirm.
- Notifying anyone about a failed or declined payment -- the Brief's Side-Effect Inventory routes a decline to an inline screen state (FEAT-10.SPEC-001), not an email; a failure is not the record-worthy, permanent event this confirmation exists to echo.
- Notifying Priya -- she has no Invoicing & Payments access (Access Matrix: None), and a payment confirmation carries financial-commitment content she is not entitled to see (XBR-08).
- In-app notification center delivery -- product-features.md's Communications field names only the email channel for this feature; an in-app surface is owned separately by In-App Notification Center (FEAT-29, Later phase).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to both Owen and Nadia, when a card payment or a confirmed bank transfer succeeds | Owen's portal sessions are short and triggered by a specific email link (user-persona.md, Behavioral Context) -- he is not routinely inside the product afterward to see the confirmation happen live; Nadia works from her own inbox and needs the same permanent, timestamped record of money received without reopening the invoice to check |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Payment succeeded | FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync) | Fires immediately and only once FEAT-10.SPEC-004 applies a Succeeded outcome to the Payment and Invoice records (card success, or a bank transfer's delayed confirmation) | Invoice `invoice_number`, `total`, `currency`; Payment `method`, `paid_at`; Client `client_name`; Client Contact name (Owen's); Freelancer Account contact details for Nadia |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact) and Nadia (the Freelancer), per the Brief's Communications field ("Payment confirmation email to Owen and to Nadia when a payment succeeds"). Both are entitled to this content under the Access Matrix: Owen made or is party to the payment himself, and Nadia's Full access to Invoicing & Payments covers every payment event on her own account's invoices.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- always sent | -- | Always on | -- (this is a transactional record email, not an optional notification) |

This confirmation is a transactional email core to the record (XBR-30): it cannot be switched off by either recipient's notification preferences -- a payment confirmation is named explicitly in product-features.md's Communications rule as one that always sends.

**Quiet Hours:** N/A -- the product defines quiet hours for optional, non-transactional notifications only (XBR-30); this confirmation is transactional and sends immediately regardless of the time of day for either recipient, consistent with the payment it confirms being a permanent, timestamped record the moment it is confirmed.

## Content Definition

**Email (to Owen):**
- **Subject:** Payment confirmed for invoice {invoice_number}
- **Body:**
  Hi {owen_first_name},

  This confirms your payment of {total} {currency} for invoice {invoice_number}, paid by {payment_method_label} on {paid_at_formatted}.

  Keep this email as your receipt.
- **CTA (button):** View invoice -- deep-links to FEAT-10.SPEC-001 (Pay Invoice Screen) for this invoice

**Email (to Nadia):**
- **Subject:** {client_name} paid invoice {invoice_number}
- **Body:**
  Hi {nadia_first_name},

  {owen_name} at {client_name} paid {total} {currency} for invoice {invoice_number} by {payment_method_label} on {paid_at_formatted}. The payment has landed directly in your connected account.
- **CTA (button):** View invoice -- deep-links to the invoice's detail in her own project view (FEAT-09)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {invoice_number} | Invoice -- invoice_number | INV-0142 | Never empty -- unique and sequential per freelancer, required at generation (FEAT-09) |
| {total} | Invoice -- total | 1,200.00 | Never empty -- required at invoice generation |
| {currency} | Invoice -- currency | USD | Never empty -- required at invoice generation (FEAT-15) |
| {payment_method_label} | Payment -- method, rendered as "card" or "bank transfer" | card | Never empty -- set atomically by FEAT-10.SPEC-004 alongside the Succeeded status |
| {paid_at_formatted} | Payment -- paid_at, rendered in each recipient's own time zone (FEAT-15) | September 27, 2026, 3:14 PM | Never empty -- set atomically by FEAT-10.SPEC-004 at the moment this notification's trigger fires |
| {client_name} | Client -- client_name | Acme Co. | Never empty -- required at client creation (FEAT-01) |
| {owen_name} | Client Contact -- name (the paying contact) | Owen Marsh | Never empty -- required at contact creation (FEAT-18) |
| {owen_first_name} | Client Contact -- name (first name portion) | Owen | Greeting renders as "Hi," |
| {nadia_first_name} | Freelancer Account -- name (first name portion) | Nadia | Greeting renders as "Hi," |

## Delivery Rules

**Batching:** None -- each successful payment is its own distinct, permanent event and is confirmed individually; two invoices paid close together each produce their own separate confirmation to each recipient, never merged into one summary email.
**Deduplication:** At most one confirmation email per recipient per Succeeded outcome. FEAT-10.SPEC-004's already-resolved guard is the deduplication boundary: this notification's trigger fires once per invoice reaching Paid via a processor-confirmed payment, so a duplicate delivery of the same underlying Succeeded event (per FEAT-10.SPEC-003's Edge Cases) never produces a second confirmation, since the guard prevents the trigger condition from being met twice.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning on the project (XBR-30) -- for the copy addressed to Owen as well as her own, since a client contact never sees the freelancer's own delivery-warning surface and Nadia is positioned to notice and follow up.
**Expiry:** This confirmation never expires undelivered in the sense of becoming pointless to send late -- the payment it confirms is a permanent record, so a delayed delivery (after retries) still carries accurate, still-true information whenever it eventually lands. There is no cutoff after which the email is withheld; the retry window in the rule above is the only limit, after which delivery is treated as failed (surfaced as a warning) rather than expired.

## Edge Cases

- **A bank-transfer payment is Pending for several days before the processor confirms it** -- No receipt of any kind is sent while Pending (feature-overview.md, Key Capabilities: Pending bank transfers). This notification fires exactly once, only at the moment FEAT-10.SPEC-004 applies the confirmed Succeeded outcome -- never at the earlier Pending moment.
- **A Pending bank transfer is ultimately reported failed rather than confirmed** -- This notification never fires for that attempt, since its trigger condition (a Succeeded outcome) was never met; the failure is communicated inline on FEAT-10.SPEC-001, per that spec's States, not by this notification.
- **Owen's or Nadia's copy fails to deliver but the other's succeeds** -- Each recipient's copy is tracked and retried independently; one recipient's successful delivery has no bearing on the other's retry count or warning surfacing.
- **The same Succeeded event is delivered twice by the payment-processing capability (per FEAT-10.SPEC-003's Edge Cases)** -- FEAT-10.SPEC-004's already-resolved guard prevents a second confirmation-notification trigger; the second delivery changes nothing and no duplicate email is sent.
- **The underlying invoice reaches Paid through a processor confirmation but is later reversed as a chargeback (owned by FEAT-25)** -- This confirmation was accurate when sent, describing a payment that genuinely succeeded at that timestamp; a later reversal is a separate, new event handled entirely by FEAT-25's own notification, and does not retroactively make this confirmation false or worth recalling.
- **Quiet hours colliding with expiry** -- N/A, since this transactional confirmation observes neither quiet hours nor an expiry cutoff (both stated above); there is no collision to resolve.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync) | Triggered by (inbound) | A confirmed Succeeded outcome fires this notification |
| FEAT-10.SPEC-001 (Pay Invoice Screen) | Navigation (outbound) | Owen's CTA deep-links here |
| FEAT-09 (Invoice Generation & Sending) | Navigation (outbound) | Nadia's CTA deep-links to the invoice's detail in her own project view |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | Underlying delivery, retry, and bounce/failure reporting capability this notification is sent through |
| FEAT-10.SPEC-003 (Card & Bank-Transfer Payment Processing) | References (inbound) | The card-succeeded and confirmed-bank-transfer inbound events are what ultimately produce this notification's trigger, via FEAT-10.SPEC-004 |

## Analytics and Success Signals

- **payment_confirmation_delivered** (recipient: owen / nadia, method: card / bank_transfer) -- supports success-metrics.md: "Time to Payment"
- **payment_confirmation_delivery_failed** (recipient: owen / nadia, retry_count) -- N/A -- no Stage 2 metric measures this feature's own delivery-failure rate directly; retained as a standard delivery-quality signal alongside the delivered event above.
- **payment_confirmation_opened** (recipient: owen / nadia) -- N/A -- no Stage 2 metric measures open rates for this specific confirmation; retained as a standard delivery-quality signal.

## Acceptance Criteria

**FEAT-10.SPEC-007-AC-01:** Given a card payment succeeds, when FEAT-10.SPEC-004 applies the Succeeded outcome, then Owen receives an email with subject "Payment confirmed for invoice {invoice_number}" and Nadia receives an email with subject "{client_name} paid invoice {invoice_number}."

**FEAT-10.SPEC-007-AC-02:** Given a bank-transfer payment is Pending, when it remains unconfirmed, then no confirmation email is sent to either recipient.

**FEAT-10.SPEC-007-AC-03:** Given a Pending bank-transfer payment is confirmed by the processor, when FEAT-10.SPEC-004 applies the Succeeded outcome, then this notification fires at that moment, releasing the receipt that was held while Pending.

**FEAT-10.SPEC-007-AC-04:** Given a Pending bank-transfer payment is ultimately reported failed, when the failure is applied, then this notification never fires for that attempt.

**FEAT-10.SPEC-007-AC-05:** Given Owen opens his confirmation email, when he taps "View invoice," then he lands on FEAT-10.SPEC-001 for that invoice.

**FEAT-10.SPEC-007-AC-06:** Given Nadia opens her confirmation email, when she taps "View invoice," then she lands on that invoice's detail in her own project view.

**FEAT-10.SPEC-007-AC-07:** Given neither Owen nor Nadia has any way to opt out of this confirmation, when their respective notification preferences are checked, then no preference control exists for it and it always sends.

**FEAT-10.SPEC-007-AC-08:** Given a payment is confirmed at any hour, when this notification fires, then it sends immediately regardless of either recipient's configured quiet hours.

**FEAT-10.SPEC-007-AC-09:** Given delivery of Owen's copy fails, when the failure occurs, then it is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, and after the final failure Nadia sees a delivery warning on the project.

**FEAT-10.SPEC-007-AC-10:** Given delivery of Nadia's copy fails while Owen's copy succeeds, when this is observed, then Nadia's copy is retried independently of Owen's successful delivery.

**FEAT-10.SPEC-007-AC-11:** Given the same Succeeded event is delivered twice by the payment-processing capability, when FEAT-10.SPEC-004's already-resolved guard blocks the second application, then this notification does not fire a second time.

**FEAT-10.SPEC-007-AC-12:** Given Priya (Client Reviewer Contact) is a contact at the same client company, when a payment succeeds on an invoice for that company, then Priya receives no copy of this confirmation.

**FEAT-10.SPEC-007-AC-13:** Given Nadia records an off-platform payment manually, when the manual record is saved, then this notification does not fire, since its trigger condition is a processor-confirmed Succeeded outcome, not a manual record.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no preference exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |
