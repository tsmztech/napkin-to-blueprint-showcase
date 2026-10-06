---
document_type: spec
spec_type: notification
spec_id: FEAT-03.SPEC-006
spec_name: Acceptance Confirmation Notification
spec_slug: acceptance-confirmation-notification
parent_feature: FEAT-03
parent_feature_name: Proposal Acceptance
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Notification Spec: Acceptance Confirmation Notification

## Overview

**Name:** Acceptance Confirmation Notification
**ID:** FEAT-03.SPEC-006
**Type:** Notification
**Purpose:** Emails Owen and Nadia a confirmation the instant a proposal's acceptance is recorded, giving both parties a durable, timestamped record that the project is now real and billable.
**Parent Feature:** FEAT-03 -- Proposal Acceptance

## Scope and Non-Goals

**In Scope:**
- The confirmation email sent to Owen (the accepting contact) and to Nadia when acceptance is recorded
- Content, delivery rules, and edge cases for this single notification

**Non-Goals:**
- The deposit invoice's own notification -- excluded per the Feature Breakdown Brief's Communications field: "the auto-generated deposit invoice sends its own notification (FEAT-09)"; this spec covers only the acceptance confirmation itself, never invoice content.
- An in-app notification channel -- product-features.md phases the In-App Notification Center (FEAT-29) as Later, outside MVP; at this feature's priority tier (Core, MVP) the only notification channel the product defines is transactional email (ASMP-29), so this spec uses no other channel.
- A preference to turn this notification off -- excluded per XBR-30: transactional emails core to the record (this is the evidentiary confirmation of a proposal's acceptance) always send and cannot be disabled; only optional notifications carry an on/off preference.
- Notifying Priya (Client Reviewer Contact) -- excluded per the Access Matrix (Proposals & Acceptance: None for Reviewers) and XBR-08: notification recipients are limited to contacts entitled to the event, and Priya has no entitlement to proposal content.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to both Owen and Nadia, the instant acceptance is recorded | Owen's sessions are short and triggered by a specific email, not habitual browsing (user-persona.md, Client Primary Contact, Behavioral Context); Nadia is not necessarily inside the product at the moment a client accepts, and needs to know the project just became billable without watching a screen. Email is the product's sole notification channel at MVP (ASMP-29); no in-app channel exists yet (FEAT-29 is Later). |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|-------------------|
| Acceptance recorded | FEAT-03.SPEC-003 (Acceptance Recording) | Fires immediately once the acceptance write succeeds | Proposal reference, `accepted_at`, `accepted_by` (Owen's Client Contact reference), project name, client company name, whether a deposit invoice was triggered |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact, the accepting contact) and Nadia (Freelancer, the project owner) -- both are entitled to the acceptance event per the Access Matrix (Proposals & Acceptance: Full for Nadia, Own-only accept/view for Owen) and per the Feature Breakdown Brief's Communications field ("Confirmation email to Owen and Nadia when acceptance is recorded").

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| N/A | N/A | Always on -- transactional | N/A -- per XBR-30, this is a transactional email core to the evidentiary record and cannot be disabled by either recipient |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for transactional confirmation emails; this notification is the timestamped record of a moment that just happened for both parties, and holding it would misrepresent when the record was actually confirmed.

## Content Definition

**Email (to Owen):**
- **Subject:** You've accepted the proposal for {project_name}
- **Body:**
  Hi {owen_first_name},

  You accepted the proposal for {project_name} on {accepted_at_date}. This confirms the record on file with {freelancer_business_name}.

  {deposit_invoice_line}
- **CTA (button):** View proposal -- deep-links to FEAT-03.SPEC-001 (Proposal Review & Accept) for this proposal, now showing "Accepted on {accepted_at_date}"

**Email (to Nadia):**
- **Subject:** {client_company_name} accepted the proposal for {project_name}
- **Body:**
  Hi {nadia_first_name},

  {owen_full_name} at {client_company_name} accepted the proposal for {project_name} on {accepted_at_date}.

  {deposit_invoice_line_nadia}
- **CTA (button):** View project -- deep-links to FEAT-01 (Client & Project Management), project view, for this project

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|-----------------|--------------------------|
| {project_name} | Project -- project_name | Brand Refresh Q1 | Never empty -- required at project creation (FEAT-01) |
| {owen_first_name} | Client Contact -- name (first token) | Owen | Greeting renders as "Hi," |
| {owen_full_name} | Client Contact -- name | Owen Carter | "the client's Primary contact" |
| {nadia_first_name} | Freelancer Account -- name (first token) | Nadia | Greeting renders as "Hi," |
| {client_company_name} | Client -- client_name | Carter & Co | Never empty -- required at client creation (FEAT-01) |
| {accepted_at_date} | Proposal -- accepted_at (formatted in the recipient's own time zone, FEAT-15) | March 4, 2026 | Never empty -- `accepted_at` is written by FEAT-03.SPEC-003 before this notification fires |
| {freelancer_business_name} | Freelancer Account -- business_name | Nadia Voss Design | Falls back to Nadia's account name if business_name is not yet set |
| {deposit_invoice_line} | Derived -- whether FEAT-03.SPEC-003 triggered a deposit invoice | "A deposit invoice has been sent to you separately." | "No deposit is due under this project's payment schedule." |
| {deposit_invoice_line_nadia} | Derived -- whether FEAT-03.SPEC-003 triggered a deposit invoice | "A deposit invoice has been generated and sent automatically." | "This project's payment schedule has no deposit, so no invoice was generated yet." |

## Delivery Rules

**Batching:** None -- each acceptance is a single, discrete evidentiary event; it is never combined with any other notification, even if Owen or Nadia has other pending emails.
**Deduplication:** At most one confirmation email per recipient per acceptance. Because FEAT-03.SPEC-003 writes the acceptance exactly once (accept-once, enforced atomically), this notification's trigger fires exactly once per Proposal; a retried Accept attempt that returns "already accepted" never re-fires this notification.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` per FEAT-14's transactional email delivery capability (ASMP-29). After the final failure, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the "Accepted on {date}" marker on FEAT-03.SPEC-001 stands as the enduring in-product record regardless of email delivery outcome.
**Expiry:** This notification never expires undelivered in the sense of being withdrawn -- it is retried per the rule above, and if all retries fail, the delivery warning (XBR-30) is the surviving signal rather than a silently dropped message, since the underlying acceptance record itself never disappears.

## Edge Cases

- **Owen's email address bounces** -- The failure is retried per the Retry on failure rule; after the final failure, Nadia sees a delivery warning on the project (XBR-30) and is advised to correct Owen's contact email (FEAT-18). The acceptance record itself is unaffected -- it does not depend on this email's delivery.
- **The deposit invoice trigger fails after the acceptance write succeeds (FEAT-03.SPEC-003's own failure path)** -- This notification still fires with the "no deposit due" fallback line only if the Payment Schedule genuinely has no deposit; if a deposit was due but the invoice trigger failed, the {deposit_invoice_line} placeholder still reflects that a deposit invoice was triggered (this notification reports what FEAT-03.SPEC-003 attempted, not whether FEAT-09's own send later succeeds) -- FEAT-09 owns communicating its own delivery outcome.
- **Owen's Client Contact record is removed (access revoked) between acceptance and this notification's delivery** -- The notification still delivers to the email address captured in `accepted_by` at the moment of acceptance, since that identity is preserved as evidence even after removal (XBR-27); the notification's content is a record of what happened, not a live view of current access.
- **Nadia's account has no `business_name` set yet (pre-FEAT-21 configuration)** -- The {freelancer_business_name} placeholder falls back to her account name, so the email to Owen never renders an empty business name.
- **Two acceptance confirmation attempts race due to a transient duplicate trigger fire** -- Deduplication holds: since FEAT-03.SPEC-003 only ever writes the acceptance once and only fires this notification's trigger once per successful write, no second instance of this notification is ever queued for the same acceptance.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|----------------|
| FEAT-03.SPEC-003 (Acceptance Recording) | Triggered by (inbound) | Fires this notification immediately once the acceptance write succeeds |
| FEAT-03.SPEC-001 (Proposal Review & Accept) | Navigation (outbound) | Owen's CTA deep-links back to the now-Accepted proposal screen |
| FEAT-01 (Client & Project Management) | Navigation (outbound) | Nadia's CTA deep-links to the project view |
| FEAT-14 (Notifications (Email)) | References (outbound) | Owns the transactional email delivery capability and delivery/bounce status reporting this notification relies on |
| FEAT-09 (Invoice Generation & Sending) | References (outbound) | The {deposit_invoice_line} placeholder reflects whether FEAT-03.SPEC-003 triggered FEAT-09's deposit invoice; FEAT-09 sends its own separate notification |

## Analytics and Success Signals

- **acceptance_confirmation_sent** (recipient: owen / nadia, deposit invoice triggered: yes / no) -- supports success-metrics.md: "Time to Proposal Acceptance"
- **acceptance_confirmation_delivery_failed** (recipient: owen / nadia, retries exhausted: yes / no) -- N/A -- no Stage 2 metric measures delivery failures directly; retained so a silently undelivered evidentiary confirmation is observable via the project's delivery warning (XBR-30) rather than invisible

## Acceptance Criteria

**FEAT-03.SPEC-006-AC-01:** Given Owen accepts a proposal with no deposit due, when FEAT-03.SPEC-003 records the acceptance, then Owen and Nadia each receive an email confirmation, and both bodies include "No deposit is due under this project's payment schedule" (Owen's variant) / "no invoice was generated yet" (Nadia's variant).

**FEAT-03.SPEC-006-AC-02:** Given Owen accepts a proposal whose Payment Schedule includes a deposit, when FEAT-03.SPEC-003 records the acceptance and triggers the deposit invoice, then Owen's confirmation email states "A deposit invoice has been sent to you separately" and Nadia's states "A deposit invoice has been generated and sent automatically."

**FEAT-03.SPEC-006-AC-03:** Given Owen receives the confirmation email, when he taps "View proposal", then he lands on FEAT-03.SPEC-001 showing "Accepted on {accepted_at_date}".

**FEAT-03.SPEC-006-AC-04:** Given Nadia receives the confirmation email, when she taps "View project", then she lands on the project view in FEAT-01.

**FEAT-03.SPEC-006-AC-05:** Given the acceptance is recorded, when this notification's trigger fires, then no on/off preference is available to either recipient to suppress it -- it always sends.

**FEAT-03.SPEC-006-AC-06:** Given Owen's email address bounces on the first delivery attempt, when the delivery capability retries, then up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window` before Nadia sees a delivery warning on the project.

**FEAT-03.SPEC-006-AC-07:** Given all retries for Owen's email are exhausted, when the final failure occurs, then Nadia sees a delivery warning on the affected project and the "Accepted on {date}" marker remains the enduring in-product record regardless.

**FEAT-03.SPEC-006-AC-08:** Given the acceptance is recorded exactly once (per FEAT-03.SPEC-003's accept-once guarantee), when a second, redundant Accept attempt returns "already accepted", then this notification's trigger does not fire a second time.

**FEAT-03.SPEC-006-AC-09:** Given Owen's Client Contact record is later removed, when this notification is still pending delivery, then it still delivers to the email address captured in `accepted_by` at the moment of acceptance.

**FEAT-03.SPEC-006-AC-10:** Given the acceptance confirmation is successfully delivered to both recipients, when the analytics signal is emitted, then an acceptance_confirmation_sent event is recorded once per recipient with the deposit-invoice-triggered flag.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- transactional) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
