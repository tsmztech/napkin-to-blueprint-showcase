---
document_type: spec
spec_type: notification
spec_id: FEAT-25.SPEC-007
spec_name: Refund & Cancellation Notification
spec_slug: refund-cancellation-notification
parent_feature: FEAT-25
parent_feature_name: Refund & Cancelled Project Handling
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Notification Spec: Refund & Cancellation Notification

## Overview

**Name:** Refund & Cancellation Notification
**ID:** FEAT-25.SPEC-007
**Type:** Notification
**Purpose:** Emails Owen when an invoice he was billed is marked Refunded/Partially refunded or when his project is marked Cancelled, so his own record of the relationship stays accurate without asking Nadia.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling

## Scope and Non-Goals

**In Scope:**
- The email delivered to Owen when Nadia marks one of his invoices Refunded or Partially refunded
- The email delivered to Owen when Nadia marks his project Cancelled
- Delivery, deduplication, retry, and expiry behavior for both trigger variants

**Non-Goals:**
- Notifying Owen of a payment reversal/chargeback -- this product records reversals as a fact for Nadia's own attention (she is the one who must respond in her own processor account); product-features.md's Communications field names no client-facing reversal email, so none exists here
- Notifying Nadia of her own refund or cancellation action -- she performed the action herself and sees its result immediately on FEAT-25.SPEC-001/FEAT-25.SPEC-002's own success toast; a separate email to the person who just took the action would be redundant
- Notifying Priya -- excluded per the Access Matrix: Priya has no billing visibility at all, and product-features.md's Communications field names Owen as the sole recipient
- Any in-app channel -- product-features.md's Communications field names only "Notification email to Owen"; this product's in-app notification feed (FEAT-29) is a Later-phase feature not yet built, so email is the only channel this spec defines

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, on both trigger variants | Owen's sessions are short and triggered by a specific email (BRIEF.md, Target Users & Roles); he is not routinely browsing the portal, so an in-portal-only status change would go unseen until his next unrelated visit |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Refund or partial refund recorded | FEAT-25.SPEC-003 (Refund & Partial Refund Recording) | Fires when a refund commit succeeds | Invoice reference, invoice number, refund type (full / partial), refunded amount, currency, client name, project name |
| Project cancellation recorded | FEAT-25.SPEC-004 (Project Cancellation Recording) | Fires when a cancellation commit succeeds | Project reference, project name, client name |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact) -- the Access Matrix entitles him to Own-only view of his own company's invoices and projects; he is the party billed on the refunded invoice or the primary contact for the cancelled project. Owen only, never Priya (no billing visibility per the Access Matrix) and never Nadia (she is the notification's actor, not its recipient).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this is a transactional record email per XBR-30 ("transactional emails core to the record... always send") | -- | Always on, no opt-out | N/A -- no preference surface exists for this notification; it is not shown as a toggleable row on any Notification Preferences screen, consistent with the transactional treatment product-features.md applies to record-status emails |

**Quiet Hours:** N/A -- the product defines quiet hours for optional, non-transactional notifications only (XBR-30); this notification is transactional and sends immediately regardless of the time of day, consistent with the refund or cancellation it reports being a permanent, timestamped record the moment it is confirmed.

## Content Definition

**Email (refund variant -- full refund):**
- **Subject:** Invoice {invoice_number} has been refunded
- **Body:**
  Hi {owen_first_name},

  Your invoice {invoice_number} for {project_name} has been refunded in full ({refunded_amount} {currency}).

  The original payment record remains on file alongside this update.
- **CTA (button):** View invoice -- deep-links to FEAT-09.SPEC-002 (Invoice Detail) for the affected invoice

**Email (refund variant -- partial refund):**
- **Subject:** Invoice {invoice_number} has been partially refunded
- **Body:**
  Hi {owen_first_name},

  {refunded_amount} {currency} has been refunded against your invoice {invoice_number} for {project_name}.

  The original payment record remains on file alongside this update.
- **CTA (button):** View invoice -- deep-links to FEAT-09.SPEC-002 (Invoice Detail) for the affected invoice

**Email (cancellation variant):**
- **Subject:** {project_name} has been marked cancelled
- **Body:**
  Hi {owen_first_name},

  {freelancer_first_name} has marked {project_name} as cancelled.

  Everything already recorded for this project -- your proposal, approvals, deliverables, and invoices -- remains exactly as it was and stays available to you.
- **CTA (button):** View project -- deep-links to the client-facing project view Owen's own portal provides (FEAT-05, Own-only)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {owen_first_name} | Client Contact -- name | Owen | Greeting renders as "Hi," |
| {invoice_number} | Invoice -- invoice_number | INV-0042 | Never empty -- invoice_number is required at generation (FEAT-09.SPEC-007) |
| {project_name} | Project -- project_name | Brand Refresh | Never empty -- project_name is required at project creation (FEAT-01.SPEC-002) |
| {refunded_amount} | Payment -- refunded_amount | 450.00 | Never empty -- this email only fires once a refund amount has been validated and committed (FEAT-25.SPEC-003) |
| {currency} | Invoice -- currency | USD | Never empty -- currency is required before a project's first invoice (XBR-17) |
| {freelancer_first_name} | Freelancer Account -- name | Nadia | Renders as "Your freelancer" when unavailable, though this field is required on every Freelancer Account and so is never actually empty in practice |

## Delivery Rules

**Batching:** None -- each refund or cancellation event is delivered as its own, individual email the moment it is recorded; the two trigger variants are never combined into one message even if both occur for the same project in quick succession, since a refund and a cancellation are distinct facts about distinct records (an invoice and a project) that Owen may need to act on separately.
**Deduplication:** At most one email per refund commit and one email per cancellation commit -- each is tied to a single, one-time state transition (FEAT-25.SPEC-003, FEAT-25.SPEC-004) that can happen at most once per invoice (Refunded/Partially refunded is not re-entered once set) or once per project (Cancelled is a terminal, non-repeatable transition).
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the refunded or cancelled status itself remains fully visible to Owen the next time he opens the affected invoice or project in his portal, regardless of this email's delivery outcome.
**Expiry:** This notification never expires undelivered in the sense of becoming irrelevant -- a refund or cancellation is a permanent record, so a delayed delivery still carries an accurate message whenever it eventually lands; retries continue for the full retry window above, and after that the delivery-failure warning (not a silently dropped message) is the surviving signal.

## Edge Cases

- **The invoice is later reported Disputed by a reversal after this refund email was already sent** -- No second email is sent by this spec for the reversal; that event is Nadia's own notification (FEAT-25.SPEC-008), never Owen's, per this spec's Non-Goals.
- **Owen's email address bounces on the first delivery attempt** -- Retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`; after the final failure, Nadia sees a delivery warning on the affected project and Owen still sees the refunded or cancelled status directly in his portal.
- **Owen's Client Contact record is removed (erasure request) before this email is delivered** -- Per XBR-27, an erasure request ends access and removes contact details immediately; this notification is cancelled silently rather than delivered to a now-invalid address, since there is no longer a contact entitled to receive it.
- **Two refunds are recorded on two different invoices for the same project at effectively the same time** -- Each produces its own, separate email; they are never merged into one message, since Delivery Rules define no batching for this notification.
- **The project is cancelled and, moments later, one of its invoices is also refunded** -- Two separate emails are sent, one per trigger variant, in the order the two commits actually occurred.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-25.SPEC-003 (Refund & Partial Refund Recording) | Triggered by (inbound) | A successful refund commit fires the refund variant of this notification |
| FEAT-25.SPEC-004 (Project Cancellation Recording) | Triggered by (inbound) | A successful cancellation commit fires the cancellation variant of this notification |
| FEAT-09.SPEC-002 (Invoice Detail) | Navigation (outbound) | The refund variant's CTA deep-links here |
| FEAT-05 (Client Portal Access) | Navigation (outbound) | The cancellation variant's CTA deep-links to Owen's own client-facing project view |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (inbound) | Delivers this notification and reports delivery, bounce, and failure status |
| FEAT-14.SPEC-006 (Delivery Failure Warning to Freelancer) | Triggers (outbound) | A final delivery failure surfaces as a warning to Nadia on the affected project |

## Analytics and Success Signals

- **refund_cancellation_notification_delivered** (variant: full_refund / partial_refund / cancellation) -- N/A -- no success-metrics.md metric is connected to this feature; retained so delivery of this record-affecting email is observable rather than invisible.
- **refund_cancellation_notification_delivery_failed** (variant, final_failure: yes / no) -- N/A -- no success-metrics.md metric is connected to this feature; retained so silent delivery loss is observable rather than invisible, consistent with the general Notification Delivery Reliability goal this product tracks across features (success-metrics.md, "Notification Delivery Reliability" -- that metric's Connected Feature is Notifications (Email), not this feature, so it is not cited here as this feature's own signal).

## Acceptance Criteria

**FEAT-25.SPEC-007-AC-01:** Given Nadia records a full refund on Owen's invoice, when the commit succeeds, then Owen receives an email with subject "Invoice {invoice_number} has been refunded" and a "View invoice" CTA.

**FEAT-25.SPEC-007-AC-02:** Given Nadia records a partial refund on Owen's invoice, when the commit succeeds, then Owen receives an email with subject "Invoice {invoice_number} has been partially refunded" naming the refunded amount.

**FEAT-25.SPEC-007-AC-03:** Given Nadia marks Owen's project Cancelled, when the commit succeeds, then Owen receives an email with subject "{project_name} has been marked cancelled" and a "View project" CTA to his own portal view.

**FEAT-25.SPEC-007-AC-04:** Given Owen taps "View invoice" on a refund email, then he lands on FEAT-09.SPEC-002 showing the invoice's Refunded or Partially refunded status.

**FEAT-25.SPEC-007-AC-05:** Given Owen taps "View project" on a cancellation email, then he lands on his own client-facing project view showing the Cancelled status.

**FEAT-25.SPEC-007-AC-06:** Given Nadia performs the refund or cancellation herself, then she receives no email from this spec -- only Owen is a recipient.

**FEAT-25.SPEC-007-AC-07:** Given Priya is a contact on the same client company, when a refund or cancellation is recorded, then she receives no email from this spec.

**FEAT-25.SPEC-007-AC-08:** Given Owen's email address bounces on the first delivery attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the project for Nadia.

**FEAT-25.SPEC-007-AC-09:** Given a delivery to Owen fails permanently after retries are exhausted, then Nadia sees a delivery warning on the affected project, and Owen still sees the refunded or cancelled status directly in his portal.

**FEAT-25.SPEC-007-AC-10:** Given Owen's Client Contact record is removed by an erasure request before this email is delivered, when the delivery would otherwise fire, then it is cancelled silently and no email is sent to the removed address.

**FEAT-25.SPEC-007-AC-11:** Given two refunds are recorded on two different invoices for the same project at effectively the same time, then Owen receives two separate emails, never one combined message.

**FEAT-25.SPEC-007-AC-12:** Given a project is cancelled and one of its invoices is separately refunded moments later, then Owen receives two separate emails in the order the two events occurred.

**FEAT-25.SPEC-007-AC-13:** Given this notification is transactional, when Owen looks for a way to turn it off, then no preference control for it exists anywhere.

**FEAT-25.SPEC-007-AC-14:** Given a refund or cancellation is recorded at any hour, when the email is ready to send, then it sends immediately with no quiet-hours hold, since this notification is transactional.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 2 (refund/partial refund, cancellation) | 2 |
| Preference States | 1 (always on, no opt-out) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
