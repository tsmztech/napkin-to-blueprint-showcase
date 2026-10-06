---
document_type: spec
spec_type: notification
spec_id: FEAT-24.SPEC-009
spec_name: Account Deletion Final Warning Notification
spec_slug: account-deletion-final-warning-notification
parent_feature: FEAT-24
parent_feature_name: Data Export & Account Deletion
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Notification Spec: Account Deletion Final Warning Notification

## Overview

**Name:** Account Deletion Final Warning Notification
**ID:** FEAT-24.SPEC-009
**Type:** Notification
**Purpose:** Emails Nadia a final confirmation notice at the moment she confirms permanent account deletion, giving her a durable record of exactly when she gave that irreversible confirmation.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion

## Scope and Non-Goals

**In Scope:**
- The final-warning email sent at the moment Nadia's explicit deletion confirmation is given
- Delivery and retry behavior for this email

**Non-Goals:**
- Capturing the confirmation itself -- owned by FEAT-24.SPEC-002 (Account Deletion Screen); this spec begins where that screen's confirmation action fires.
- Executing the deletion -- owned by FEAT-24.SPEC-004 (Account Deletion Processing); this email fires in parallel with that automation and does not gate or wait for it.
- Offering any way to cancel or undo the deletion from this email -- excluded per product-features.md's Validation & Limits field ("irreversibility") and feature-overview.md's Non-Goals ("no undo after explicit confirmation"); this email carries no CTA, since the product deliberately offers no path to reverse a confirmation Nadia has already deliberately given.
- Notifying Owen, Priya, or Dana -- none of these roles has any access to this entity (FEAT-24.SPEC-007: None for all three); this email is Nadia's own record of her own action.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to Nadia, the moment her explicit deletion confirmation is given | This is the single most consequential, irreversible action available in the product; email gives Nadia a durable, external record of exactly when she confirmed, independent of the product itself, which will shortly remove her access entirely |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia gives explicit confirmation | FEAT-24.SPEC-002 (Account Deletion Screen) | Fires the moment the acknowledgment checkbox is checked and the typed confirmation exactly matches "DELETE" and Delete My Account is tapped, in parallel with FEAT-24.SPEC-004 beginning | Freelancer Account name and sign-in email; the confirmation timestamp |

## Audience and Preferences

**Recipients:** Nadia (the Freelancer) only. FEAT-24.SPEC-007 confines every action and every piece of content in this feature to her; Owen, Priya, and Dana have no access to this entity at all.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- always sent | -- | Always on | -- (this is a transactional, record-of-the-fact email about the single most consequential action in the product; it has no opt-out) |

This email is never optional, consistent with XBR-30's treatment of transactional record email -- a confirmation of permanent, irreversible account deletion is not a notification Nadia can choose not to receive a record of.

**Quiet Hours:** N/A -- this email is transactional and exempt from quiet hours (XBR-30); it sends the instant confirmation is given, regardless of the hour, since the deletion it records begins processing immediately.

## Content Definition

**Email:**
- **Subject:** Your Clientroom account is being permanently deleted
- **Body:**
  Hi {nadia_first_name},

  You confirmed permanent deletion of your Clientroom account and all its data on {confirmation_date}. This action cannot be undone.
- **CTA:** None -- feature-overview.md's Non-Goals establish no undo or restore path once confirmation is given; this email's purpose is to give Nadia a clear, permanent record of exactly when she confirmed, not to offer a way to stop or reverse what she deliberately confirmed.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {nadia_first_name} | Freelancer Account -- name (first name portion) | Nadia | Greeting renders as "Hi," |
| {confirmation_date} | Derived -- the timestamp FEAT-24.SPEC-002 captures when Delete My Account is tapped with a valid confirmation | October 4, 2026, 3:14 PM | Never empty -- this notification's own trigger condition requires a captured confirmation timestamp to exist |

## Delivery Rules

**Batching:** None -- a Freelancer Account can be confirmed for deletion at most once (FEAT-24.SPEC-004 treats any further attempt as no-action or as a discarded redundant trigger), so there is never more than one instance of this email per account.
**Deduplication:** At most one email per confirmed-deletion event. FEAT-24.SPEC-004's own concurrency handling (only one confirmed instruction ever proceeds past its trigger check) is the deduplication boundary -- a second, redundant confirmation attempt for the same account never produces a second email.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001).
**Expiry:** This email does not expire in the sense of becoming pointless to send late: it is a record of a moment that already happened and remains true regardless of delay. Unlike other transactional emails in this product, a final failure after all retries has no screen to surface a delivery warning on, since the account this email is about will, by that point, likely already be deleted (FEAT-24.SPEC-004 proceeds regardless of this email's own delivery outcome) -- delivery failure here is simply accepted as unrecoverable rather than retried indefinitely or surfaced anywhere.

## Edge Cases

- **FEAT-24.SPEC-004's cascade fails and reverts the account to Active after this email has already been sent** -- The email already sent is not retracted or followed by a correction; it accurately reported the confirmation Nadia gave at that moment. She sees the reverted-account error directly on FEAT-24.SPEC-002, and no separate "never mind" email is sent, since a second, contradicting email would confuse rather than clarify, and the account remains intact for her to try again.
- **Every retry fails and the account has since been fully deleted** -- No further action is taken; unlike other transactional emails in this product, there is no ongoing screen or account state left to surface the delivery failure on, since deletion proceeds regardless of whether this email itself was ever delivered.
- **Two deletion confirmations for the same account are submitted close together (a double-tap or two open tabs)** -- Only the first reaching FEAT-24.SPEC-004 produces this email, mirroring that automation's own concurrency handling; the second is discarded before it would trigger a duplicate.
- **Quiet hours colliding with expiry** -- N/A, since this email is transactional and exempt from quiet hours, and has no expiry cutoff to collide with in the first place.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-002 (Account Deletion Screen) | Triggered by (inbound) | The confirmation action fires this notification |
| FEAT-24.SPEC-004 (Account Deletion Processing) | References (outbound) | The cascade this email precedes and runs in parallel with, without gating it |
| FEAT-24.SPEC-007 (Export & Deletion Access Rules) | References (inbound) | Confines the recipient to Nadia alone |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | Underlying delivery and retry capability this notification is sent through |

## Analytics and Success Signals

- **account_deletion_final_warning_email_delivered** () -- N/A -- no success-metrics.md metric is connected to Data Export & Account Deletion; retained as the audit-relevant record that Nadia's final warning was actually delivered before her account's irreversible removal completed.
- **account_deletion_final_warning_email_delivery_failed** (retry_count) -- N/A -- same reason; retained so an undelivered final warning stays observable even though nothing further can be done about it once the account is gone.

## Acceptance Criteria

**FEAT-24.SPEC-009-AC-01:** Given Nadia gives explicit confirmation on FEAT-24.SPEC-002, when this notification fires, then she receives an email with subject "Your Clientroom account is being permanently deleted" carrying the exact confirmation timestamp.

**FEAT-24.SPEC-009-AC-02:** Given Nadia opens this email, when she looks for a way to undo or cancel the deletion, then no CTA or link of any kind offers one.

**FEAT-24.SPEC-009-AC-03:** Given Nadia has no way to opt out of this email, when her notification preferences are checked, then no preference control exists for it and it always sends.

**FEAT-24.SPEC-009-AC-04:** Given confirmation is given at any hour, when this notification fires, then it sends immediately regardless of Nadia's configured quiet hours.

**FEAT-24.SPEC-009-AC-05:** Given delivery fails on the first attempt, when the retry logic runs, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`.

**FEAT-24.SPEC-009-AC-06:** Given every retry fails and the account has since been fully deleted, when the final retry would otherwise be attempted, then no further action is taken and no delivery warning is surfaced anywhere.

**FEAT-24.SPEC-009-AC-07:** Given this email has already been sent, when FEAT-24.SPEC-004's cascade subsequently fails and reverts the account to Active, then no correction or "never mind" email is sent.

**FEAT-24.SPEC-009-AC-08:** Given two deletion confirmations for the same account are submitted close together, when both reach FEAT-24.SPEC-004, then only the first produces this email.

**FEAT-24.SPEC-009-AC-09:** Given Owen, Priya, or Dana has no access to this entity, when Nadia confirms deletion, then none of them receives any copy of this email.

**FEAT-24.SPEC-009-AC-10:** Given this email is delivered, when Nadia reads it, then it names the exact date and time she confirmed, with no ambiguity about which action it refers to.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no preference exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |
