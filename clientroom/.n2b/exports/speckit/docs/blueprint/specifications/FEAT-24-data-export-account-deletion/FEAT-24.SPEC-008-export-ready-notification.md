---
document_type: spec
spec_type: notification
spec_id: FEAT-24.SPEC-008
spec_name: Export Ready Notification
spec_slug: export-ready-notification
parent_feature: FEAT-24
parent_feature_name: Data Export & Account Deletion
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Notification Spec: Export Ready Notification

## Overview

**Name:** Export Ready Notification
**ID:** FEAT-24.SPEC-008
**Type:** Notification
**Purpose:** Emails Nadia when her requested data export archive is ready to download, so she learns about it even if she is away from the product while it generates.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion

## Scope and Non-Goals

**In Scope:**
- The confirmation email sent when the archive reaches Ready
- Delivery, retry, and expiry behavior for this email

**Non-Goals:**
- Deciding when the archive reaches Ready -- owned entirely by FEAT-24.SPEC-003 (Data Export Archive Generation); this spec begins where that automation's trigger fires.
- Notifying anyone about a generation failure -- product-features.md's Communications field for this feature names only the ready-to-download confirmation as triggering email; a failed generation is surfaced as an in-screen error on FEAT-24.SPEC-001, not by a separate email.
- Notifying Owen, Priya, or Dana -- none of these roles has any access to this entity (FEAT-24.SPEC-007: None for all three); this email carries content about Nadia's own data export that no one else is entitled to know exists.
- In-app notification center delivery -- product-features.md's Communications field names only the email channel for this feature; an in-app surface is owned separately by In-App Notification Center (FEAT-29, Later phase).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to Nadia, when the archive reaches Ready | Nadia works from a laptop or desktop throughout her day and is not necessarily inside the product at the moment a large archive finishes generating (user-persona.md, Behavioral Context); since the archive expires after a limited download window, email is the channel that ensures she learns it exists in time to retrieve it |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Archive becomes Ready | FEAT-24.SPEC-003 (Data Export Archive Generation) | Fires when FEAT-24.SPEC-003 applies the Ready outcome to Nadia's Data Export Archive | Freelancer Account name and sign-in email; the archive's download link/window expiry |

## Audience and Preferences

**Recipients:** Nadia (the Freelancer) only. FEAT-24.SPEC-007 confines every action and every piece of content in this feature to her; Owen, Priya, and Dana have no access to this entity at all.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- always sent | -- | Always on | -- (this is a transactional email tied to a data-readiness event Nadia herself requested, not an optional notification) |

Notification preferences (FEAT-21.SPEC-002) can switch off only optional notifications; this email confirms the outcome of an action Nadia deliberately took (requesting her own export) and has no opt-out, consistent with XBR-30's treatment of record-core transactional email.

**Quiet Hours:** N/A -- this email is transactional and exempt from quiet hours (XBR-30 defines quiet hours only for optional, non-transactional notifications); it sends as soon as the archive reaches Ready regardless of the hour, since the download window begins counting from that moment.

## Content Definition

**Email:**
- **Subject:** Your data export is ready to download
- **Body:**
  Hi {nadia_first_name},

  Your requested data export is ready. Download it before {expiry_date} -- after that, you'll need to request a new export.
- **CTA (button):** Download your export -- deep-links to FEAT-24.SPEC-001 (Data Export Screen)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {nadia_first_name} | Freelancer Account -- name (first name portion) | Nadia | Greeting renders as "Hi," |
| {expiry_date} | Data Export Archive -- download link/window (the expiry timestamp set when FEAT-24.SPEC-003 applies the Ready outcome) | October 4, 2026 | Never empty -- FEAT-24.SPEC-003 always sets the download window at the same moment it applies the Ready outcome that triggers this email |

## Delivery Rules

**Batching:** None -- at most one active archive exists per account (FEAT-24.SPEC-003), so there is never more than one pending instance of this email to batch.
**Deduplication:** At most one email per FEAT-24.SPEC-003 Ready application. A new request that supersedes the current archive (FEAT-24.SPEC-003's own discard-and-recreate step) cancels any pending email for the superseded archive; only the newest archive's Ready event produces a delivered email.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001).
**Expiry:** If every retry fails and the archive has since reached its own Expired state (per FEAT-24.SPEC-003) before the email would be attempted again, the email is not sent at all -- an email urging Nadia to download an archive that no longer exists would be actively misleading. If the archive is still Ready or Downloaded when a delayed retry succeeds, the email still sends with the archive's actual current expiry date.

## Edge Cases

- **The archive expires before this email is delivered (all retries exhausted)** -- Per the Expiry rule above, the email is not sent; Nadia simply sees the Expired state directly on FEAT-24.SPEC-001 the next time she visits.
- **Nadia requests a new export before the email for the prior archive has been delivered** -- The pending email for the superseded archive is cancelled, since it would reference an archive that FEAT-24.SPEC-003 has already discarded; only the new request's eventual Ready event produces an email.
- **Nadia's account is deleted (FEAT-24.SPEC-004) while this email is still queued for retry** -- The pending delivery is cancelled once account deletion finalizes and removes her Freelancer Account and its Notifications, per the dependency map's Notification lifecycle (Deleted by FEAT-24).
- **Quiet hours colliding with expiry** -- N/A, since this email is transactional and exempt from quiet hours, so there is no quiet-hours hold to collide with the expiry cutoff.
- **Nadia downloads the archive before this email is even delivered (a fast retry succeeds after she has already returned to the product on her own)** -- The email still sends; it accurately confirms the export became ready and remains useful as a record of the event, even though she no longer needs its CTA to find it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-003 (Data Export Archive Generation) | Triggered by (inbound) | The Ready outcome fires this notification |
| FEAT-24.SPEC-001 (Data Export Screen) | Navigation (outbound) | The CTA deep-links here |
| FEAT-24.SPEC-007 (Export & Deletion Access Rules) | References (inbound) | Confines the recipient to Nadia alone |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | Underlying delivery, retry, and bounce/failure reporting capability this notification is sent through |

## Analytics and Success Signals

- **data_export_ready_email_delivered** () -- N/A -- no success-metrics.md metric is connected to Data Export & Account Deletion; retained because product-features.md's Signals field names "data_export_ready" explicitly as this feature's defined signal.
- **data_export_ready_email_delivery_failed** (retry_count) -- N/A -- same reason; retained as a standard delivery-quality signal so a silently undelivered export-ready email stays observable.

## Acceptance Criteria

**FEAT-24.SPEC-008-AC-01:** Given FEAT-24.SPEC-003 applies a Ready outcome to Nadia's archive, when this notification fires, then she receives an email with subject "Your data export is ready to download."

**FEAT-24.SPEC-008-AC-02:** Given Nadia opens the export-ready email, when she taps "Download your export," then she lands on FEAT-24.SPEC-001 (Data Export Screen).

**FEAT-24.SPEC-008-AC-03:** Given Nadia has no way to opt out of this email, when her notification preferences are checked, then no preference control exists for it and it always sends.

**FEAT-24.SPEC-008-AC-04:** Given the archive reaches Ready at any hour, when this notification fires, then it sends immediately regardless of Nadia's configured quiet hours.

**FEAT-24.SPEC-008-AC-05:** Given delivery fails on the first attempt, when the retry logic runs, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`.

**FEAT-24.SPEC-008-AC-06:** Given every retry fails and the archive has since expired, when the final retry would otherwise be attempted, then the email is not sent.

**FEAT-24.SPEC-008-AC-07:** Given Nadia requests a new export before the prior archive's email is delivered, when the new request supersedes the prior archive, then the pending email for the superseded archive is cancelled.

**FEAT-24.SPEC-008-AC-08:** Given Nadia's account is deleted while this email is still queued for retry, when the deletion finalizes, then the pending delivery is cancelled.

**FEAT-24.SPEC-008-AC-09:** Given Owen, Priya, or Dana has no access to this entity, when the archive reaches Ready, then none of them receives any copy of this email.

**FEAT-24.SPEC-008-AC-10:** Given a delayed retry succeeds while the archive is still Ready, when the email is finally delivered, then it shows the archive's actual current expiry date.

**FEAT-24.SPEC-008-AC-11:** Given Nadia has already downloaded the archive by the time a delayed retry succeeds, when the email is delivered, then it still sends as an accurate record of the export becoming ready.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no preference exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
