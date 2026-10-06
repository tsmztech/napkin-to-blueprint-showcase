---
document_type: spec
spec_type: notification
spec_id: FEAT-09.SPEC-010
spec_name: Invoice Issued & Copy Confirmation Notification
spec_slug: invoice-issued-copy-confirmation-notification
parent_feature: FEAT-09
parent_feature_name: Invoice Generation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Notification Spec: Invoice Issued & Copy Confirmation Notification

## Overview

**Name:** Invoice Issued & Copy Confirmation Notification
**ID:** FEAT-09.SPEC-010
**Type:** Notification
**Purpose:** Emails Owen the invoice with its pay link and emails Nadia a confirmation copy, whenever any invoice -- automatic, ad hoc, or credit note -- is sent.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending

## Scope and Non-Goals

**In Scope:**
- The invoice email to Owen, carrying the pay link in whichever of the three availability states applies
- The confirmation copy to Nadia, sent alongside every invoice send
- Retry and delivery-failure behavior for both recipients
- The identical treatment of automatic invoices, ad-hoc invoices, and credit notes -- all three trigger this same notification

**Non-Goals:**
- The pay-link wording itself -- owned by FEAT-09.SPEC-009 (Pay-Link Availability & No-Account Fallback Rule); this spec only renders that spec's output into the email
- Overdue payment reminders after this initial send -- owned entirely by Automated Payment Reminders (FEAT-11), a distinct notification with its own schedule
- The underlying email send and delivery/bounce status mechanism -- owned by the Transactional Email Delivery capability (FEAT-14.SPEC-001); this spec composes the message and consumes that capability's delivery status, but does not implement the sending itself
- Notifying Priya -- excluded per the Access Matrix: invoice content, including its existence, is never communicated to Reviewer contacts

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, for both Owen and Nadia, on every invoice send | Email is the only channel that reaches client contacts (BRIEF.md, Ecosystem & Integrations: "Clients will not install an app"), and this notification's whole purpose is to hand Owen a way to pay and Nadia a record that it went out |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An invoice reaches Sent status | FEAT-09.SPEC-004 (Automatic Invoice Generation) | Fires immediately once an automatically generated invoice's send hand-off begins | Invoice reference, all its content fields, pay-link availability state |
| An invoice or credit note reaches Sent status | FEAT-09.SPEC-005 (Manual Invoice & Credit Note Recording) | Fires immediately once a manually issued invoice or a credit note's send hand-off begins | Invoice or credit-note reference, all its content fields, pay-link availability state |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact) receives the invoice email for his own company's invoices, own-only, per the Access Matrix. Nadia (Freelancer) receives the confirmation copy for every invoice she issues, automatically or manually. Priya (Client Reviewer Contact) never receives this notification, per the Access Matrix's total exclusion of Reviewer contacts from invoice content.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A | -- | Always on | -- |

This is a transactional record email core to the financial relationship (an invoice and its own confirmation copy); per XBR-30, transactional emails central to the record always send and cannot be switched off by either recipient.

**Quiet Hours:** N/A -- the product defines no quiet-hours window for transactional invoice delivery (XBR-30 distinguishes only optional notifications as quiet-hours-eligible); an invoice is time-sensitive to both the freelancer's cash flow and the due-date clock it starts, so it is sent immediately regardless of the hour.

## Content Definition

**Email (to Owen):**
- **Subject:** New invoice from {freelancer_business_name}: {invoice_total} due {due_date}
- **Body:**
  Hi {owen_first_name},

  {freelancer_business_name} has sent you an invoice for {project_name}.

  Invoice {invoice_number} -- {invoice_total} -- due {due_date}

  {pay_link_status_block}

  You can view the full invoice, its status, and a downloadable copy any time from your portal.
- **CTA (button):** View invoice -- deep-links to FEAT-09.SPEC-002 (Invoice Detail) for this invoice
- **Secondary link:** View all invoices -- deep-links to FEAT-09.SPEC-001 (Invoice List), scoped to Owen's own company

**Email (to Nadia -- confirmation copy):**
- **Subject:** Invoice {invoice_number} sent to {client_billing_name}
- **Body:**
  Hi {nadia_first_name},

  Invoice {invoice_number} for {project_name} was sent to {client_billing_name} -- {invoice_total}, due {due_date}.

  {trigger_source_line}
- **CTA (button):** View invoice -- deep-links to FEAT-09.SPEC-002 (Invoice Detail) for this invoice

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_business_name} | Freelancer Account -- business_name | Nadia Reyes Design | Never empty -- FEAT-09.SPEC-007 blocks sending entirely until this field is complete |
| {owen_first_name} | Client Contact -- name (first token) | Owen | "there" (greeting renders "Hi there,") |
| {nadia_first_name} | Freelancer Account -- name (first token) | Nadia | "there" |
| {project_name} | Project -- project_name | Brand Refresh -- Phase 2 | Never empty -- project_name is required at project creation (FEAT-01) |
| {invoice_number} | Invoice -- invoice_number | INV-0047 | Never empty -- always assigned before this notification fires (FEAT-09.SPEC-007) |
| {invoice_total} | Invoice -- total, formatted with currency | $2,400.00 | Never empty -- total is always derived before sending |
| {due_date} | Invoice -- due_date, in the recipient's own time zone | May 14, 2026 | Never empty -- due_date is always set before sending |
| {client_billing_name} | Client -- billing_name | Meridian Studios LLC | Never empty -- billing completeness blocks sending until this exists |
| {pay_link_status_block} | Derived -- FEAT-09.SPEC-009's current availability state, rendered as its own short paragraph plus the pay-now button when Ready | "Pay this invoice online: [Pay now]" | Renders the "not yet available" or "temporarily unavailable" paragraph instead of a button, per FEAT-09.SPEC-009; never blank |
| {trigger_source_line} | Derived -- the triggering event, phrased for Nadia's context | "Triggered by: milestone approval" / "Issued manually" / "Credit note against INV-0041" | Never empty -- every invoice has exactly one recorded triggering_event |

## Delivery Rules

**Batching:** None -- each invoice send produces exactly one email to Owen and one to Nadia; invoices are never batched into a single combined notification, since each carries its own pay link and due date that the recipient must act on individually.
**Deduplication:** At most one invoice email and one confirmation copy per invoice send. A retried send hand-off (per FEAT-09.SPEC-004/FEAT-09.SPEC-005's failure handling) never produces a second pair of emails once the first pair is confirmed delivered; a retry only re-attempts a send that has not yet succeeded.
**Retry on failure:** Delivery failure on either email is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure to Owen, the failure is surfaced to Nadia as a delivery warning on the project (XBR-30); after the final failure to Nadia's own copy, no separate warning is generated, since the invoice itself remains fully visible to her in FEAT-09.SPEC-001 and FEAT-09.SPEC-002 regardless of this email's outcome.
**Expiry:** Not applicable -- an invoice notification does not expire or become stale to withhold; if delivery is still retrying when the retry window ends, the final failure is surfaced per the Retry rule above rather than the notification being silently dropped.

## Edge Cases

- **The invoice is corrected by a credit note before Owen ever opens the original email** -- The original email's content is not retracted or altered (emails are immutable once sent); Owen's link to FEAT-09.SPEC-002 always shows the invoice's current, live status (now Corrected, with a link to the credit note), even though the email text itself still reads as it did at send time.
- **Owen's email address bounces on the first delivery attempt** -- Retried per the Retry rule; after the final failure, Nadia sees a delivery warning on the project naming Owen's contact as undeliverable, consistent with the pattern used across this pipeline's other transactional notifications.
- **The Payment Account Connection changes state between this email being composed and Owen opening it** -- The email's {pay_link_status_block} reflects the state at send time and is not retroactively updated; Owen sees the current state instead once he follows the CTA into FEAT-09.SPEC-002, per FEAT-09.SPEC-009's live-rendering behavior on that screen.
- **A credit note is sent against an invoice that was itself never successfully delivered to Owen** -- The credit note produces its own independent send attempt to Owen and its own confirmation copy to Nadia; a prior delivery failure on the original invoice does not suppress or alter the credit note's own delivery.
- **Nadia issues an ad-hoc invoice while offline, and connectivity returns moments later** -- Per FEAT-09.SPEC-003/FEAT-09.SPEC-005's offline handling, the submission itself only succeeds once connectivity returns; this notification fires only after a successful recording, so no notification is ever queued against a submission that never actually completed.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-004 (Automatic Invoice Generation) | Triggered by (inbound) | Fires this notification once an automatic invoice reaches Sent |
| FEAT-09.SPEC-005 (Manual Invoice & Credit Note Recording) | Triggered by (inbound) | Fires this notification once a manual invoice or credit note reaches Sent |
| FEAT-09.SPEC-009 (Pay-Link Availability & No-Account Fallback Rule) | References (inbound) | Supplies the exact wording for {pay_link_status_block} |
| FEAT-09.SPEC-002 (Invoice Detail) | Navigation (outbound) | Both CTAs deep-link here |
| FEAT-09.SPEC-001 (Invoice List) | Navigation (outbound) | Owen's secondary "View all invoices" link |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (inbound) | Provides the send mechanism and delivery/bounce status this spec consumes |
| FEAT-01.SPEC-005 (Project Detail) | Affects (outbound) | Shows the delivery-failure warning when Owen's copy exhausts retries |

## Analytics and Success Signals

- **invoice_email_delivered** (recipient: owen / nadia; triggering event type) -- supports success-metrics.md: "Notification Delivery Reliability"
- **invoice_email_opened** (recipient) -- supports success-metrics.md: "Notification Delivery Reliability"
- **invoice_email_cta_tapped** (recipient; destination: invoice_detail / invoice_list) -- supports success-metrics.md: "Time to Payment" (Owen's CTA tap is the first observable step toward the pay flow this metric measures)
- **invoice_email_delivery_failed** (recipient; retry_count) -- N/A -- no Stage 2 metric measures failed invoice-email delivery directly, though "Notification Delivery Reliability" measures the aggregate; retained here as this notification's own diagnostic signal, consistent with FEAT-05.SPEC-008's pattern for the same event shape

## Acceptance Criteria

**FEAT-09.SPEC-010-AC-01:** Given an invoice is generated automatically and reaches Sent status, when this notification fires, then Owen receives the invoice email and Nadia receives the confirmation copy.

**FEAT-09.SPEC-010-AC-02:** Given Nadia issues an ad-hoc invoice successfully, when it reaches Sent status, then the same two emails are sent, worded identically to the automatic path.

**FEAT-09.SPEC-010-AC-03:** Given a credit note is recorded, when it reaches Sent status, then Owen and Nadia each receive their respective emails for the credit note, distinct from the original invoice's emails.

**FEAT-09.SPEC-010-AC-04:** Given Nadia has a Connected, ready payment account, when Owen's email is composed, then {pay_link_status_block} renders the "Pay now" button.

**FEAT-09.SPEC-010-AC-05:** Given Nadia has no connected payment account, when Owen's email is composed, then {pay_link_status_block} renders the "not yet available" wording with no pay button.

**FEAT-09.SPEC-010-AC-06:** Given Priya is a Reviewer contact at the same client company as Owen, when any invoice is sent, then Priya never receives this notification.

**FEAT-09.SPEC-010-AC-07:** Given Owen's email bounces on the first delivery attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the project for Nadia.

**FEAT-09.SPEC-010-AC-08:** Given Nadia's own confirmation copy fails delivery after all retries, when the final failure occurs, then no separate project-level warning is generated, since the invoice remains fully visible to her in-product.

**FEAT-09.SPEC-010-AC-09:** Given Owen taps "View invoice" from the email, when the tap registers, then he lands on FEAT-09.SPEC-002 for that exact invoice.

**FEAT-09.SPEC-010-AC-10:** Given an invoice is corrected after its original email was sent, when Owen later opens that original email's link, then he sees the invoice's current, live Corrected status rather than the email's original static text.

**FEAT-09.SPEC-010-AC-11:** Given a send hand-off is retried after a transient failure, when the retry succeeds, then exactly one pair of emails is ultimately delivered -- never two.

**FEAT-09.SPEC-010-AC-12:** Given the invoice_email_delivered event is emitted for Owen, when it fires, then it carries the recipient and the triggering event type.

**FEAT-09.SPEC-010-AC-13:** Given no quiet-hours window applies to this notification, when an invoice is generated at any hour, then both emails send immediately with no hold.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 2 | 2 |
| Preference States | 1 (always on -- transactional) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
