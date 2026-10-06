---
document_type: spec
spec_type: notification
spec_id: FEAT-26.SPEC-004
spec_name: Signed-Copy Confirmation Notification
spec_slug: signed-copy-confirmation-notification
parent_feature: FEAT-26
parent_feature_name: Legally Binding E-Signature for Proposals
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Notification Spec: Signed-Copy Confirmation Notification

## Overview

**Name:** Signed-Copy Confirmation Notification
**ID:** FEAT-26.SPEC-004
**Type:** Notification
**Purpose:** Emails Owen and Nadia a signed-copy confirmation the instant a proposal's signature is recorded, in addition to FEAT-03's standard acceptance confirmation, so both parties have a distinct, durable record that this acceptance carries the stronger evidentiary weight of a signature.
**Parent Feature:** FEAT-26 -- Legally Binding E-Signature for Proposals

## Scope and Non-Goals

**In Scope:**
- The signed-copy confirmation email sent to Owen (the signing contact) and to Nadia when a signature is recorded
- Content, delivery rules, and edge cases for this single notification, distinct from FEAT-03.SPEC-006's standard acceptance confirmation

**Non-Goals:**
- The standard acceptance confirmation itself -- owned entirely by FEAT-03.SPEC-006 (Acceptance Confirmation Notification), which FEAT-26.SPEC-002 also fires for a signed acceptance per the Feature Breakdown Brief's Communications field ("in addition to the standard acceptance confirmation"); this spec covers only the signed-copy confirmation, never the standard one's content.
- The deposit invoice's own notification -- owned by FEAT-09 (Invoice Generation & Sending); this spec's content states only whether a deposit invoice was triggered, exactly as FEAT-03.SPEC-006 does.
- An in-app notification channel -- product-features.md phases the In-App Notification Center (FEAT-29) as Later, outside MVP and outside this feature's v1 phase; at this feature's priority tier the only notification channel the product defines is transactional email (ASMP-29), so this spec uses no other channel.
- A preference to turn this notification off -- excluded per XBR-30: transactional emails core to the record (this is the evidentiary confirmation of a signed acceptance) always send and cannot be disabled; only optional notifications carry an on/off preference.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to both Owen and Nadia, the instant the signature is recorded | Owen's sessions are short and triggered by a specific email, not habitual browsing (user-persona.md, Client Primary Contact, Behavioral Context); Nadia is not necessarily inside the product at the moment a client signs, and needs a durable record that the stronger, signed form of acceptance now exists. Email is the product's sole notification channel at this feature's phase (ASMP-29); no in-app channel exists yet (FEAT-29 is Later). |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|-------------------|
| Signature recorded | FEAT-26.SPEC-002 (Signature Recording) | Fires immediately once the signature write succeeds, alongside (and in addition to) FEAT-03.SPEC-006's own trigger from the same write | Proposal reference, `accepted_at`, `accepted_by` (Owen's Client Contact reference), the signer's full legal name, project name, client company name, whether a deposit invoice was triggered |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact, the signing contact) and Nadia (Freelancer, the project owner) -- both are entitled to the signed-acceptance event per the Access Matrix (Proposals & Acceptance: Full for Nadia, Own-only sign/view for Owen) and per the Feature Breakdown Brief's Communications field ("A signed-copy confirmation email to both parties, in addition to the standard acceptance confirmation").

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| N/A | N/A | Always on -- transactional | N/A -- per XBR-30, this is a transactional email core to the evidentiary record and cannot be disabled by either recipient |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for transactional confirmation emails; this notification is the timestamped record of a moment that just happened for both parties, and holding it would misrepresent when the signature was actually confirmed.

## Content Definition

**Email (to Owen):**
- **Subject:** Your signed copy of the proposal for {project_name}
- **Body:**
  Hi {owen_first_name},

  You signed the proposal for {project_name} on {accepted_at_date} as {signer_full_legal_name}. This is your signed copy, carrying stronger evidentiary weight than a recorded acceptance alone.

  {deposit_invoice_line}
- **CTA (button):** View signed proposal -- deep-links to FEAT-26.SPEC-001 (Signature Signing Step) for this proposal, now showing "Signed on {accepted_at_date}"

**Email (to Nadia):**
- **Subject:** {client_company_name} signed the proposal for {project_name}
- **Body:**
  Hi {nadia_first_name},

  {owen_full_name} at {client_company_name} signed the proposal for {project_name} on {accepted_at_date} as {signer_full_legal_name}. This signed record carries stronger evidentiary weight than a plain timestamped acceptance.

  {deposit_invoice_line_nadia}
- **CTA (button):** View project -- deep-links to FEAT-01 (Client & Project Management), project view, for this project

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|-----------------|--------------------------|
| {project_name} | Project -- project_name | Brand Refresh Q1 | Never empty -- required at project creation (FEAT-01) |
| {owen_first_name} | Client Contact -- name (first token) | Owen | Greeting renders as "Hi," |
| {owen_full_name} | Client Contact -- name | Owen Carter | "the client's Primary contact" |
| {client_company_name} | Client -- client_name | Carter & Co | Never empty -- required at client creation (FEAT-01) |
| {accepted_at_date} | Proposal -- accepted_at (formatted in the recipient's own time zone, FEAT-15) | March 4, 2026 | Never empty -- `accepted_at` is written by FEAT-26.SPEC-002 before this notification fires |
| {signer_full_legal_name} | Proposal -- signature record, signature data (the full legal name entered on FEAT-26.SPEC-001) | Owen Carter | Never empty -- FEAT-26.SPEC-003's field validation rule requires a non-empty full legal name before FEAT-26.SPEC-002 can write the signature record |
| {nadia_first_name} | Freelancer Account -- name (first token) | Nadia | Greeting renders as "Hi," |
| {deposit_invoice_line} | Derived -- whether FEAT-26.SPEC-002 triggered a deposit invoice | "A deposit invoice has been sent to you separately." | "No deposit is due under this project's payment schedule." |
| {deposit_invoice_line_nadia} | Derived -- whether FEAT-26.SPEC-002 triggered a deposit invoice | "A deposit invoice has been generated and sent automatically." | "This project's payment schedule has no deposit, so no invoice was generated yet." |

## Delivery Rules

**Batching:** None -- each signature is a single, discrete evidentiary event; it is never combined with any other notification, including FEAT-03.SPEC-006's standard confirmation, which is delivered as its own separate email even though both fire from the same write.
**Deduplication:** At most one signed-copy confirmation email per recipient per signature. Because FEAT-26.SPEC-002 writes the signature exactly once (sign-once, enforced atomically), this notification's trigger fires exactly once per Proposal; a retried Sign attempt that returns "already signed" never re-fires this notification.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` per FEAT-14's transactional email delivery capability (ASMP-29). After the final failure, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the "Signed on {date}" marker on FEAT-26.SPEC-001 stands as the enduring in-product record regardless of email delivery outcome.
**Expiry:** This notification never expires undelivered in the sense of being withdrawn -- it is retried per the rule above, and if all retries fail, the delivery warning (XBR-30) is the surviving signal rather than a silently dropped message, since the underlying signature record itself never disappears.

## Edge Cases

- **Owen's email address bounces** -- The failure is retried per the Retry on failure rule; after the final failure, Nadia sees a delivery warning on the project (XBR-30) and is advised to correct Owen's contact email (FEAT-18). The signature record itself is unaffected -- it does not depend on this email's delivery.
- **FEAT-03.SPEC-006's standard confirmation delivers but this signed-copy confirmation fails** -- The two notifications are independent deliveries from the same trigger; a failure of one does not affect the other's delivery or retry schedule. Owen and Nadia may see one email before the other, but both are eventually delivered or, on final failure, surfaced as a delivery warning.
- **The deposit invoice trigger fails after the signature write succeeds (FEAT-26.SPEC-002's own failure path)** -- This notification still fires with the "no deposit due" fallback line only if the Payment Schedule genuinely has no deposit; if a deposit was due but the invoice trigger failed, the {deposit_invoice_line} placeholder still reflects that a deposit invoice was triggered -- FEAT-09 owns communicating its own delivery outcome.
- **Owen's Client Contact record is removed (access revoked) between signing and this notification's delivery** -- The notification still delivers to the email address captured in `accepted_by` at the moment of signing, since that identity is preserved as evidence even after removal (XBR-27); the notification's content is a record of what happened, not a live view of current access.
- **Nadia's account has no `business_name` set yet** -- Not applicable to this notification's content, since neither email template references the freelancer's business name; both templates identify the freelancer only by first name in the greeting.
- **Two signature confirmation attempts race due to a transient duplicate trigger fire** -- Deduplication holds: since FEAT-26.SPEC-002 only ever writes the signature once and only fires this notification's trigger once per successful write, no second instance of this notification is ever queued for the same signature.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|----------------|
| FEAT-26.SPEC-002 (Signature Recording) | Triggered by (inbound) | Fires this notification immediately once the signature write succeeds, alongside FEAT-03.SPEC-006 |
| FEAT-03.SPEC-006 (Acceptance Confirmation Notification) | References (inbound) | Both notifications fire from the same write; this spec covers only the additional signed-copy content |
| FEAT-26.SPEC-001 (Signature Signing Step) | Navigation (outbound) | Owen's CTA deep-links back to the now-Signed proposal screen |
| FEAT-01 (Client & Project Management) | Navigation (outbound) | Nadia's CTA deep-links to the project view |
| FEAT-14 (Notifications (Email)) | References (outbound) | Owns the transactional email delivery capability and delivery/bounce status reporting this notification relies on |
| FEAT-09 (Invoice Generation & Sending) | References (outbound) | The {deposit_invoice_line} placeholder reflects whether FEAT-26.SPEC-002 triggered FEAT-09's deposit invoice; FEAT-09 sends its own separate notification |

## Analytics and Success Signals

- **signed_copy_confirmation_sent** (recipient: owen / nadia, deposit invoice triggered: yes / no) -- N/A -- no Stage 2 metric in success-metrics.md is connected to FEAT-26 (its Connected Feature slice is empty); retained as the notification-side delivery signal for the product-defined proposal_signed behavior
- **signed_copy_confirmation_delivery_failed** (recipient: owen / nadia, retries exhausted: yes / no) -- N/A -- no Stage 2 metric measures delivery failures for this feature; retained so a silently undelivered evidentiary confirmation is observable via the project's delivery warning (XBR-30) rather than invisible

## Acceptance Criteria

**FEAT-26.SPEC-004-AC-01:** Given Owen signs a proposal with no deposit due, when FEAT-26.SPEC-002 records the signature, then Owen and Nadia each receive a signed-copy confirmation email, and both bodies include "No deposit is due under this project's payment schedule" (Owen's variant) / "no invoice was generated yet" (Nadia's variant).

**FEAT-26.SPEC-004-AC-02:** Given Owen signs a proposal whose Payment Schedule includes a deposit, when FEAT-26.SPEC-002 records the signature and triggers the deposit invoice, then Owen's confirmation email states "A deposit invoice has been sent to you separately" and Nadia's states "A deposit invoice has been generated and sent automatically."

**FEAT-26.SPEC-004-AC-03:** Given Owen signs the proposal, when this notification and FEAT-03.SPEC-006's standard confirmation both fire from the same write, then Owen and Nadia each receive two separate emails -- the standard acceptance confirmation and this signed-copy confirmation -- never combined into one.

**FEAT-26.SPEC-004-AC-04:** Given Owen receives the signed-copy confirmation email, when he taps "View signed proposal", then he lands on FEAT-26.SPEC-001 showing "Signed on {accepted_at_date}".

**FEAT-26.SPEC-004-AC-05:** Given Nadia receives the signed-copy confirmation email, when she taps "View project", then she lands on the project view in FEAT-01.

**FEAT-26.SPEC-004-AC-06:** Given the signature is recorded, when this notification's trigger fires, then no on/off preference is available to either recipient to suppress it -- it always sends.

**FEAT-26.SPEC-004-AC-07:** Given Owen's email address bounces on the first delivery attempt, when the delivery capability retries, then up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window` before Nadia sees a delivery warning on the project.

**FEAT-26.SPEC-004-AC-08:** Given all retries for Owen's email are exhausted, when the final failure occurs, then Nadia sees a delivery warning on the affected project and the "Signed on {date}" marker remains the enduring in-product record regardless.

**FEAT-26.SPEC-004-AC-09:** Given the signature is recorded exactly once (per FEAT-26.SPEC-002's sign-once guarantee), when a second, redundant Sign attempt returns "already signed", then this notification's trigger does not fire a second time.

**FEAT-26.SPEC-004-AC-10:** Given Owen's Client Contact record is later removed, when this notification is still pending delivery, then it still delivers to the email address captured in `accepted_by` at the moment of signing.

**FEAT-26.SPEC-004-AC-11:** Given the signed-copy confirmation is successfully delivered to both recipients, when the analytics signal is emitted, then a signed_copy_confirmation_sent event is recorded once per recipient with the deposit-invoice-triggered flag.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- transactional) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |
