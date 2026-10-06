---
document_type: spec
spec_type: notification
spec_id: FEAT-11.SPEC-004
spec_name: Overdue Reminder Email
spec_slug: overdue-reminder-email
parent_feature: FEAT-11
parent_feature_name: Automated Payment Reminders
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Notification Spec: Overdue Reminder Email

## Overview

**Name:** Overdue Reminder Email
**ID:** FEAT-11.SPEC-004
**Type:** Notification
**Purpose:** Sends the client's Primary Contact a polite email about an overdue invoice, on the day-3 automatic reminder, the day-10 automatic reminder, or Nadia's manual reminder -- with a direct way to pay.
**Parent Feature:** FEAT-11 -- Automated Payment Reminders

## Scope and Non-Goals

**In Scope:**
- The email sent to the invoice's Primary Contact for a day-3 automatic, day-10 automatic, or manual reminder
- The three content variants (day 3, day 10, manual) and their shared pay-link CTA
- Delivery, retry, and expiry behavior for this email

**Non-Goals:**
- Deciding when day-3 and day-10 fall, or whether a given send is currently eligible -- owned by FEAT-11.SPEC-001 (Reminder Schedule) and FEAT-11.SPEC-002 (Reminder Eligibility Rule); this spec begins once one of those has already decided to send.
- Any channel beyond email -- excluded per assumptions-constraints.md (ASMP-29) and BRIEF.md's Ecosystem & Integrations: "clients will not install an app," making email the sole channel reaching client contacts; no SMS or push capability exists in the product definition.
- Addressing Priya (Client Reviewer Contact) or any contact other than the invoice's Primary Contact -- grounded in the Access Matrix: Priya's Invoicing & Payments access is None, and reminders are addressed only to the Primary Contact per this feature's Access field.
- The pay flow itself once Owen clicks the link -- owned by FEAT-10.SPEC-001 (Pay Invoice Screen).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always -- every day-3, day-10, or manual reminder is delivered by email | Email is the sole channel reaching client contacts (ASMP-29, BRIEF.md's Ecosystem & Integrations); Owen is not inside the product between sessions triggered by a specific link, so an interruption must reach him where he already is |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Day-3 automatic reminder eligible and ready to send | FEAT-11.SPEC-001 (Reminder Schedule) | Fires when the schedule's eligibility re-check for the day-3 threshold passes | Invoice reference, invoice number, amount, currency, due date, freelancer business name, Primary Contact reference, pay-link availability |
| Day-10 automatic reminder eligible and ready to send | FEAT-11.SPEC-001 (Reminder Schedule) | Fires when the schedule's eligibility re-check for the day-10 threshold passes | Same as above |
| Manual reminder eligible and ready to send | FEAT-11.SPEC-003 (Invoice Reminder Panel) | Fires when Nadia's "Send Reminder Now" action passes FEAT-11.SPEC-002's Eligible-to-send check | Same as above |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact) -- the invoice's Primary Contact, per the Access Matrix's Invoicing & Payments row ("Own-only: view, pay, download copies") and XBR-08 (only Primary contacts see and pay invoices). Priya (Client Reviewer Contact) is never a recipient -- her Invoicing & Payments access is None. Dana (Support Operator) never receives this email; she may view its delivery status read-only inside a logged support session, mirroring the reminder history on FEAT-11.SPEC-003.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- not configurable | -- | Always sends | -- |

This email is a transactional record email core to the invoice's payment lifecycle, not an optional notification; per XBR-30, transactional emails core to the record always send and cannot be disabled by either Nadia or Owen.

**Quiet Hours:** N/A -- the product definition establishes no quiet-hours capability anywhere (no Stage 2 document defines one); every reminder sends the moment it is triggered.

## Content Definition

**Email (day 3):**
- **Subject:** Reminder: Invoice {invoice_number} from {freelancer_business_name} is now overdue
- **Body:**
  Hi {contact_first_name},

  This is a friendly reminder that invoice {invoice_number} for {amount_due} was due on {due_date} and hasn't been paid yet.

  You can take care of it in a couple of minutes using the link below.
- **CTA (button):** Pay Invoice {invoice_number} -- deep-links to FEAT-10.SPEC-001 (Pay Invoice Screen) for this invoice

**Email (day 10):**
- **Subject:** Second reminder: Invoice {invoice_number} from {freelancer_business_name} is still overdue
- **Body:**
  Hi {contact_first_name},

  Invoice {invoice_number} for {amount_due} (due {due_date}) is still showing as unpaid, {days_overdue} days after the due date.

  If there's anything holding this up, please reach out to {freelancer_business_name} directly -- otherwise, you can pay it now using the link below.
- **CTA (button):** Pay Invoice {invoice_number} -- deep-links to FEAT-10.SPEC-001 (Pay Invoice Screen) for this invoice

**Email (manual):**
- **Subject:** Reminder: Invoice {invoice_number} from {freelancer_business_name} is due
- **Body:**
  Hi {contact_first_name},

  {freelancer_business_name} wanted to remind you that invoice {invoice_number} for {amount_due} (due {due_date}) hasn't been paid yet.

  You can take care of it using the link below.
- **CTA (button):** Pay Invoice {invoice_number} -- deep-links to FEAT-10.SPEC-001 (Pay Invoice Screen) for this invoice

**No-pay-link variant (all three, applied when pay-link availability is unavailable per FEAT-09.SPEC-009):** The CTA button is replaced with the plain-text instructions for paying the freelancer directly, exactly as FEAT-09.SPEC-009 defines them for the original invoice email; the subject and body above are otherwise unchanged.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {contact_first_name} | Client Contact -- name (first token) | Owen | Never empty -- name is required at contact creation (FEAT-18) |
| {freelancer_business_name} | Freelancer Account -- business_name | Studio Nadia | Never empty -- business name is required before the first invoice is sent |
| {invoice_number} | Invoice -- invoice_number | INV-0042 | Never empty -- required and system-assigned at generation |
| {amount_due} | Invoice -- total, formatted with its currency | $1,200.00 | Never empty -- required at invoice generation |
| {due_date} | Invoice -- due_date, rendered in the recipient's own time zone per FEAT-15.SPEC-006 | March 3, 2026 | Never empty -- required before the invoice sends |
| {days_overdue} | Derived -- elapsed calendar days since due_date, computed in the freelancer's time zone (FEAT-11.SPEC-002) | 10 | Never empty -- the day-10 variant only renders once this value is exactly 10 |

## Delivery Rules

**Batching:** None -- each reminder is delivered as its own, single email tied to one invoice and one threshold. Reminders for two different overdue invoices are never combined into one email, since each invoice's overdue status and pay link are independent and combining them would blur which invoice needs attention.
**Deduplication:** At most one email per Reminder Log entry. A Reminder Schedule re-run (FEAT-11.SPEC-001) never re-sends the email tied to an already-`sent_at` entry; only a newly created entry (a new threshold, or a new manual send) produces a new email.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the Reminder Log entry itself still shows as sent from Nadia's perspective in FEAT-11.SPEC-003's history, since the send was attempted and logged -- the delivery warning is the surviving signal of the failure, not a change to the history entry.
**Expiry:** N/A -- an overdue reminder has no meaningful expiry window distinct from the retry window itself: once the retry window is exhausted, the failure is reported per the rule above rather than the email being discarded as stale. A reminder is never held past its trigger moment awaiting a better time to send (no quiet hours, no batching window to wait out).

## Edge Cases

- **Invoice is paid moments after the eligibility check passes but before the email is actually delivered** -- The queued send is not cancelled once handed to the delivery capability; a reminder that arrives moments after payment is a rare, narrow timing artifact accepted as a product decision rather than treated as a defect, since eligibility was genuinely true at the moment the send was authorized.
- **The invoice's Primary Contact is removed (erasure request, FEAT-18) between trigger and delivery** -- The send is cancelled; there is no longer a recipient entitled to receive it. This is logged as a delivery-skipped-no-recipient outcome and surfaced to Nadia as a delivery warning on the affected project (XBR-30), since it is a break in the reminder chain she should know about.
- **No preference to collide with quiet hours or a preference change mid-flight** -- Because this email carries no preference control and the product defines no quiet hours (per Audience and Preferences above), the usual preference/quiet-hours collision edge cases do not apply to this notification; this is a resolved decision, not an omission.
- **Pay-link availability changes between trigger and delivery (Nadia's payment account moves from Connected to Needs attention)** -- The email renders with whichever pay-link state (linked CTA or plain-text instructions) is current at the moment of composition, immediately before send, consistent with FEAT-09.SPEC-009's authority over pay-link wording.
- **A day-3 and a day-10 reminder for two different invoices both fire for the same recipient on the same day** -- Each is delivered as its own separate email, per the no-batching rule; Owen receives two distinct emails, each naming its own invoice number.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-11.SPEC-001 (Reminder Schedule) | Triggered by (inbound) | Fires this notification for every eligible day-3 or day-10 send |
| FEAT-11.SPEC-003 (Invoice Reminder Panel) | Triggered by (inbound) | Fires this notification for every eligible manual send |
| FEAT-11.SPEC-002 (Reminder Eligibility Rule) | References (inbound) | Governs whether the triggering send was allowed to happen at all |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | Used for the actual send and for delivery/bounce status reporting |
| FEAT-10.SPEC-001 (Pay Invoice Screen) | Navigation (outbound) | Every CTA deep-links here for the specific invoice |
| FEAT-09.SPEC-009 (Pay-Link Availability & No-Account Fallback Rule) | References (inbound) | Governs the CTA's linked-vs-plain-text rendering |
| FEAT-31.SPEC-002 (Operator Support Session Console) | References (outbound) | Dana views this email's delivery status read-only inside a logged support session |

## Analytics and Success Signals

- **reminder_email_delivered** (reminder_type: day_3 / day_10 / manual) -- supports success-metrics.md: "Notification Delivery Reliability"
- **reminder_email_delivery_failed** (reminder_type: day_3 / day_10 / manual; retry_count) -- supports success-metrics.md: "Notification Delivery Reliability"
- **reminder_pay_link_clicked** (reminder_type: day_3 / day_10 / manual) -- supports success-metrics.md: "Reminder-Driven Payment Recovery"

## Acceptance Criteria

**FEAT-11.SPEC-004-AC-01:** Given Owen's invoice reaches day 3 overdue and eligibility passes, when FEAT-11.SPEC-001 fires this notification, then Owen receives an email with subject "Reminder: Invoice {invoice_number} from {freelancer_business_name} is now overdue" and a "Pay Invoice {invoice_number}" button.

**FEAT-11.SPEC-004-AC-02:** Given Owen's invoice reaches day 10 overdue and eligibility passes, when the notification fires, then Owen receives the day-10 email with subject "Second reminder: Invoice {invoice_number} from {freelancer_business_name} is still overdue" naming {days_overdue} as 10.

**FEAT-11.SPEC-004-AC-03:** Given Nadia sends a manual reminder and eligibility passes, when the notification fires, then Owen receives the manual variant with subject "Reminder: Invoice {invoice_number} from {freelancer_business_name} is due".

**FEAT-11.SPEC-004-AC-04:** Given Owen taps "Pay Invoice {invoice_number}" in any variant, when the link is followed, then he lands on FEAT-10.SPEC-001 for that specific invoice.

**FEAT-11.SPEC-004-AC-05:** Given Nadia has no connected, ready payment account for this invoice, when any reminder variant is sent, then the CTA is replaced with the plain-text pay-the-freelancer-directly instructions from FEAT-09.SPEC-009.

**FEAT-11.SPEC-004-AC-06:** Given the email fails to deliver on the first attempt, when the delivery capability retries, then up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the affected project for Nadia.

**FEAT-11.SPEC-004-AC-07:** Given the invoice's Primary Contact was removed via an erasure request between trigger and delivery, when the send is attempted, then it is cancelled, logged as delivery-skipped-no-recipient, and surfaced to Nadia as a delivery warning.

**FEAT-11.SPEC-004-AC-08:** Given Priya (Client Reviewer Contact) is a contact on the same client company, when any reminder variant is sent, then Priya is never a recipient on any copy of the email.

**FEAT-11.SPEC-004-AC-09:** Given Owen has no notification preferences that could disable this email, when any reminder is triggered, then it always sends -- there is no opt-out control anywhere in Owen's portal for this email.

**FEAT-11.SPEC-004-AC-10:** Given a day-3 reminder for one invoice and a day-10 reminder for a different invoice both become eligible for the same recipient on the same day, when both fire, then Owen receives two separate emails, each naming its own invoice number -- never a combined email.

**FEAT-11.SPEC-004-AC-11:** Given the Reminder Schedule re-evaluates an invoice whose day-3 Reminder Log entry already has a `sent_at` value, when the re-evaluation runs, then no second day-3 email is sent for that invoice.

**FEAT-11.SPEC-004-AC-12:** Given Dana (Support Operator) has an open support session on the freelancer's account, when she views the invoice's reminder history, then she can see this email's delivery status read-only, and never receives a copy of the email herself.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 3 (day-3, day-10, manual) | 3 |
| Preference States | 1 (always sends -- not configurable) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
