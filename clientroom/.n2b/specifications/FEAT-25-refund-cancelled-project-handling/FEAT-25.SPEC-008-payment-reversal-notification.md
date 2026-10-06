---
document_type: spec
spec_type: notification
spec_id: FEAT-25.SPEC-008
spec_name: Payment Reversal Notification
spec_slug: payment-reversal-notification
parent_feature: FEAT-25
parent_feature_name: Refund & Cancelled Project Handling
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Notification Spec: Payment Reversal Notification

## Overview

**Name:** Payment Reversal Notification
**ID:** FEAT-25.SPEC-008
**Type:** Notification
**Purpose:** Emails Nadia the moment a payment reversal or chargeback is recorded, so she knows to respond in her own processor account.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling

## Scope and Non-Goals

**In Scope:**
- The email delivered to Nadia the moment a reversal or chargeback is recorded against one of her invoices
- Delivery, deduplication, retry, and expiry behavior for this single trigger

**Non-Goals:**
- Notifying Owen of the reversal -- product-features.md's Communications field names only Nadia as the recipient for a reversal; Owen already knows about any dispute he filed with his own card issuer or bank, and this product's evidentiary role is to alert the freelancer, not the client, per XBR-21
- Carrying any action the recipient can take inside the product -- excluded per scope-boundaries.md (SC-18): responding to the dispute happens entirely inside Nadia's own processor account; this email's CTA opens the invoice for her own record-keeping only, never a response flow this product does not offer
- Any in-app channel -- product-features.md's Communications field names only an email to Nadia; this product's in-app notification feed (FEAT-29) is a Later-phase feature not yet built

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always | Nadia works from a laptop or desktop throughout her working day but is not necessarily inside the product at the moment a processor reports a reversal; a dispute has its own response clock in her processor account, so she needs to be alerted the instant it happens, not only the next time she happens to open Clientroom |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Payment reversal recorded | FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording) | Fires immediately when a reversal commit succeeds | Invoice reference, invoice number, prior status (Paid / Refunded / Partially refunded), amount, currency, client name, project name |

## Audience and Preferences

**Recipients:** Nadia (Freelancer) -- the sole recipient. She is the account owner and the only party who can act on the dispute in her own processor account; per the Access Matrix, Invoicing & Payments is Full for Nadia and this reversal concerns her own financial record.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this is a transactional record email per XBR-30 ("transactional emails core to the record... always send") | -- | Always on, no opt-out | N/A -- no preference surface exists for this notification; it is not shown as a toggleable row on FEAT-21.SPEC-002 (Notification Preferences), consistent with the transactional treatment product-features.md applies to record-status emails |

**Quiet Hours:** N/A -- the product defines quiet hours for optional, non-transactional notifications only (XBR-30); this notification is transactional and time-critical -- a chargeback carries its own response deadline in Nadia's processor account, so holding it for a quiet-hours window would shrink her time to respond, directly contradicting the Feature Breakdown Brief's "notified immediately."

## Content Definition

**Email:**
- **Subject:** Payment reversal reported on invoice {invoice_number}
- **Body:**
  Hi {freelancer_first_name},

  Your payment processor has reported a reversal or chargeback on invoice {invoice_number} for {project_name} ({client_name}), amount {amount} {currency}.

  This invoice now shows as Disputed in Clientroom, alongside its original payment record. Respond to the dispute directly in your payment processor account -- Clientroom does not handle chargebacks or disputes itself.
- **CTA (button):** View invoice -- deep-links to FEAT-09.SPEC-002 (Invoice Detail) for the affected invoice

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_first_name} | Freelancer Account -- name | Nadia | Greeting renders as "Hi," |
| {invoice_number} | Invoice -- invoice_number | INV-0042 | Never empty -- invoice_number is required at generation (FEAT-09.SPEC-007) |
| {project_name} | Project -- project_name | Brand Refresh | Never empty -- project_name is required at project creation (FEAT-01.SPEC-002) |
| {client_name} | Client -- client_name | Acme Co. | Never empty -- client_name is required at client creation (FEAT-01.SPEC-001) |
| {amount} | Payment -- amount | 450.00 | Never empty -- this email only fires once a reversal has been correlated to a Succeeded Payment (FEAT-25.SPEC-005) |
| {currency} | Invoice -- currency | USD | Never empty -- currency is required before a project's first invoice (XBR-17) |

## Delivery Rules

**Batching:** None -- each reversal is delivered as its own, individual email the moment it is recorded, regardless of how many other reversals or refunds may be in flight elsewhere in the account; a chargeback is time-sensitive on its own and must never wait to be bundled with another event.
**Deduplication:** At most one email per reversal commit -- FEAT-25.SPEC-005's idempotent handling of a duplicate or out-of-order reversal event for the same invoice means a second delivery of the same underlying processor report produces no second commit, and so no second email.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the invoice's Disputed status remains fully visible to her the next time she opens FEAT-09.SPEC-002, regardless of this email's delivery outcome.
**Expiry:** This notification never expires undelivered in the sense of becoming irrelevant -- a reversal is a permanent record and Nadia's dispute-response clock runs in her processor account, not inside this email; retries continue for the full retry window above, and after that the delivery-failure warning (not a silently dropped message) is the surviving signal, alongside the Disputed status itself, which is never hidden.

## Edge Cases

- **Nadia's sign-in email address does not exist or bounces on the first delivery attempt** -- Retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`; after the final failure, she sees a delivery warning inside the product the next time she is in-product, and the invoice's Disputed status is visible regardless.
- **Two reversal notices arrive for the same invoice at effectively the same time (duplicate delivery)** -- FEAT-25.SPEC-005 applies only the first as a genuine commit; the second is an idempotent no-op, so only one email is ever sent for the same underlying event.
- **A reversal is recorded on an invoice Nadia had already partially refunded herself** -- The email still fires normally, naming the invoice's prior status implicitly through its Disputed outcome; the amount named is the Payment's original amount, since that is what the processor is reversing, not the smaller amount Nadia had already refunded.
- **Nadia is offline or away from email when the reversal is recorded** -- The email is queued and delivered as soon as delivery succeeds; there is no in-product-only fallback for this notification, since email is its only channel and the Disputed status is separately always visible whenever she next opens the invoice.
- **The Freelancer Account is deleted (FEAT-24) between the reversal commit and this email's delivery** -- Per FEAT-24's account-removal scope, the delivery is cancelled silently rather than sent to a closed account; there is no freelancer left to notify.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording) | Triggered by (inbound) | A successful reversal commit fires this notification immediately |
| FEAT-09.SPEC-002 (Invoice Detail) | Navigation (outbound) | The CTA deep-links here, showing the Disputed status |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (inbound) | Delivers this notification and reports delivery, bounce, and failure status |
| FEAT-14.SPEC-006 (Delivery Failure Warning to Freelancer) | Triggers (outbound) | A final delivery failure surfaces as a warning to Nadia on the affected project |

## Analytics and Success Signals

- **payment_reversal_notification_delivered** (invoice reference) -- N/A -- no success-metrics.md metric is connected to this feature; retained so delivery of this evidentiary email is observable rather than invisible, per XBR-21's evidentiary intent.
- **payment_reversal_notification_delivery_failed** (final_failure: yes / no) -- N/A -- no success-metrics.md metric is connected to this feature; retained so silent delivery loss of a time-critical dispute alert is observable rather than invisible.

## Acceptance Criteria

**FEAT-25.SPEC-008-AC-01:** Given a reversal is recorded on one of Nadia's invoices, when the commit succeeds, then she receives an email with subject "Payment reversal reported on invoice {invoice_number}" immediately.

**FEAT-25.SPEC-008-AC-02:** Given Nadia opens the reversal email, when she taps "View invoice", then she lands on FEAT-09.SPEC-002 showing the invoice's Disputed status alongside its preserved prior record.

**FEAT-25.SPEC-008-AC-03:** Given the reversal email body, when Nadia reads it, then it states plainly that she must respond to the dispute directly in her payment processor account, since Clientroom does not handle chargebacks or disputes itself.

**FEAT-25.SPEC-008-AC-04:** Given Owen is the client on the reversed invoice, when the reversal is recorded, then he receives no email from this spec.

**FEAT-25.SPEC-008-AC-05:** Given Nadia's email bounces on the first delivery attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before a delivery warning appears to her in-product.

**FEAT-25.SPEC-008-AC-06:** Given delivery to Nadia fails permanently after retries are exhausted, then she sees a delivery warning on the affected project, and the invoice's Disputed status is still visible whenever she next opens it.

**FEAT-25.SPEC-008-AC-07:** Given two reversal notices arrive for the same invoice at effectively the same time, then only one email is sent, since the second commit is an idempotent no-op.

**FEAT-25.SPEC-008-AC-08:** Given the invoice was already Partially refunded by Nadia's own action before this reversal, when the reversal email is sent, then it still names the Payment's original amount as the reversed amount.

**FEAT-25.SPEC-008-AC-09:** Given a reversal is recorded at any hour, when the email is ready to send, then it sends immediately with no quiet-hours hold, since this notification is transactional and time-critical.

**FEAT-25.SPEC-008-AC-10:** Given Nadia looks for a way to turn this notification off, when she checks FEAT-21.SPEC-002 (Notification Preferences), then no toggle for it exists there.

**FEAT-25.SPEC-008-AC-11:** Given Nadia's Freelancer Account is deleted between the reversal commit and this email's delivery, when the delivery would otherwise fire, then it is cancelled silently.

**FEAT-25.SPEC-008-AC-12:** Given Nadia is away from email when a reversal is recorded, when she next checks her inbox, then the queued email is present with its original content, unmodified by the passage of time.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no opt-out) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
