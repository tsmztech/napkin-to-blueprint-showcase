---
document_type: spec
spec_type: notification
spec_id: FEAT-18.SPEC-015
spec_name: Support Request Acknowledgement
spec_slug: support-request-acknowledgement
parent_feature: FEAT-18
parent_feature_name: Account & Data Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Notification Spec: Support Request Acknowledgement

## Overview

**Name:** Support Request Acknowledgement
**ID:** FEAT-18.SPEC-015
**Type:** Notification
**Purpose:** Sends the member who contacted support a transactional-email acknowledgement that their message was received.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- The acknowledgement delivered when a general support description is submitted, on its one channel (email)
- Retry and expiry behavior for this notification

**Non-Goals:**
- Any further reply, status update, or resolution message -- excluded per scope-boundaries.md SC-14: this feature's Contact Support screen sends a one-way description and receives a one-time acknowledgement; any further exchange happens outside the product, not as an additional in-product notification
- Deciding when a support description is submitted -- owned by FEAT-18.SPEC-005 (Contact Support), whose successful submission triggers this notification
- The email channel's delivery mechanics -- owned by FEAT-18.SPEC-012 (Transactional Email, Account & Data), which this notification's email is sent through
- The safety-concern acknowledgement -- a distinct communication for a distinct Support Request kind, owned by FEAT-02 (Dietary Rules & Allergy Safety Engine)

## Channels

| Channel | Used When | Rationale |
|---------|-----------|--------------|
| Email | Always, once a general support description is submitted | The submitting member already saw a same-screen confirmation on FEAT-18.SPEC-005; the email is the durable record of the acknowledgement and the channel through which any eventual operator follow-up will reach them, per scope-boundaries.md SC-14 |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Support Request created | FEAT-18.SPEC-005 (Contact Support) | Fires when a general-support-contact Support Request is created successfully | Reporting member's email address and first name, submitted description text, submission date |

## Audience and Preferences

**Recipients:** The reporting adult member (Maya or Sam) -- whoever submitted the support description on FEAT-18.SPEC-005. The acknowledgement is delivered only to the member who raised the specific request, since each request is a private, one-way communication with the operator.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|------------------------|
| N/A -- this notification has no on/off control | -- | Always on | -- |

No preference exists for this notification: it confirms an action the member themselves just took, and withholding the acknowledgement would leave them uncertain whether their message was received at all.

**Quiet Hours:** N/A -- this notification fires immediately after submission, whenever that happens to occur; the product defines no quiet-hours window for a confirmation of an action the recipient initiated themselves, since holding it would only delay the reassurance the acknowledgement exists to give.

## Content Definition

**Email:**
- **Subject:** We've got your message
- **Body:**
  Hi {member_first_name},

  Thanks for reaching out. Here's what you sent us on {submission_date}:

  "{submitted_description}"

  We'll follow up by email if we need more information or once we've looked into it.
- **CTA:** N/A -- no in-product destination exists to link to, since this feature defines no status-tracking screen for a submitted request (scope-boundaries.md SC-14); any further exchange happens by email outside the product.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|-----------------------------|-------------------|----------------------------|
| {member_first_name} | Member Profile -- display_name (the reporting member) | Sam | Greeting renders as "Hi," |
| {submission_date} | Support Request -- derived from its creation date | September 27, 2026 | Never empty -- a Support Request always carries a creation date the moment it is created |
| {submitted_description} | Support Request -- note | The grocery list won't load on my phone. | Never empty -- FEAT-18.SPEC-010's required-field rule prevents an empty description from ever being submitted |

## Delivery Rules

**Batching:** No batching applies -- each submitted support description creates its own Support Request and its own independent acknowledgement, even if the same member submits more than one in quick succession (per FEAT-18.SPEC-005's Edge Cases).
**Deduplication:** At most one acknowledgement per Support Request. A Support Request is created exactly once per submission; there is no re-processing path that could re-trigger this notification for the same request.
**Retry on failure:** Email delivery failure is retried up to 3 times over 6 hours, per FEAT-18.SPEC-012's Degradation Behavior. After the final failure, the reporting member's same-screen confirmation on FEAT-18.SPEC-005 stands as the sole delivery-of-record indicator that their message was received in-product, and no further user-facing error is shown for the email specifically.
**Expiry:** This notification does not expire in the usual sense -- an acknowledgement delivered late is still meaningfully useful to the reporting member, since it confirms receipt of a message that remains open regardless of when the email arrives; delivery continues to be attempted for the full retry window.

## Edge Cases

- **The reporting member is removed (FEAT-18.SPEC-007) or deletes their own account (FEAT-18.SPEC-009) between submission and this notification's delivery** -- The email address and first name are captured at submission time (per this spec's Trigger's Available Data) and carried through to send time independently of the Member Profile record's later lifecycle, so the acknowledgement is still delivered to the address that was valid at submission.
- **The household is deleted between submission and this notification's delivery** -- The Support Request itself is retained through household deletion's own record-keeping only up to the point the cascade removes it (FEAT-18.SPEC-008); if the acknowledgement has not yet been sent when the household record is fully removed, it is still delivered using the captured address and content, since the acknowledgement concerns the member's own submission, not the household's continued existence.
- **The email capability is down for the entire 6-hour retry window** -- The acknowledgement is never delivered by email; the reporting member's same-screen confirmation from FEAT-18.SPEC-005 remains the only confirmation they receive, and the failure is logged internally.
- **The submitted description contains characters that render unusually in an email body (e.g., emoji or unusual punctuation)** -- The description is included verbatim as submitted; no sanitization beyond what the platform's standard email-rendering handles is applied by this notification.
- **Two adults in the same household each submit a separate support description around the same time** -- Each submission produces its own independent Support Request and its own acknowledgement addressed to its own submitting member; the two are never merged or cross-referenced.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-005 (Contact Support) | Triggered by (inbound) | Successful submission fires this notification |
| FEAT-18.SPEC-012 (Transactional Email, Account & Data) | Triggers (outbound) | The email channel is sent through this integration |
| FEAT-22 (Operator Read-Only Support Access) | References (inbound, cross-feature) | The Support Request this notification acknowledges is the same record that opens Riley's read-only support access |

## Analytics and Success Signals

- **support_acknowledgement_delivered** (delivery outcome: delivered / failed) -- N/A -- no Stage 2 success metric measures acknowledgement delivery directly; retained per product-features.md's Signals field (support_contacted) as the operational confirmation this trust-facing message actually reaches the reporting member
- **support_acknowledgement_failed** (retries_attempted) -- N/A -- no Stage 2 success metric measures acknowledgement failures; retained to observe whether the reporting member's only durable confirmation ever fails to arrive

## Acceptance Criteria

**FEAT-18.SPEC-015-AC-01:** Given Sam submits a general support description, when FEAT-18.SPEC-005 creates the Support Request, then he receives an email with the subject "We've got your message" quoting his submitted description.

**FEAT-18.SPEC-015-AC-02:** Given Maya submits a support description, when the acknowledgement email is sent, then it addresses her by her own first name and includes her submission date.

**FEAT-18.SPEC-015-AC-03:** Given the acknowledgement email fails to send on its first attempt, when a retry remains within the 6-hour window, then it is retried automatically.

**FEAT-18.SPEC-015-AC-04:** Given the acknowledgement email's retries are exhausted, when this occurs, then no further error is shown to the reporting member, and their same-screen confirmation on FEAT-18.SPEC-005 stands as the record of receipt.

**FEAT-18.SPEC-015-AC-05:** Given Sam has no preference control for this notification, when he submits a support description, then the acknowledgement email is always sent, since no opt-out exists.

**FEAT-18.SPEC-015-AC-06:** Given the reporting member is removed from the household after submitting but before this notification is sent, when it is sent, then it still reaches the email address captured at submission time.

**FEAT-18.SPEC-015-AC-07:** Given two adults in the same household each submit a separate support description around the same time, when both are processed, then each receives their own independent acknowledgement addressed to themselves.

**FEAT-18.SPEC-015-AC-08:** Given a submitted support description, when the acknowledgement email is generated, then no in-product CTA is offered, since this feature defines no in-product status-tracking screen for the request.

**FEAT-18.SPEC-015-AC-09:** Given the submitting member's household is deleted before the acknowledgement email is sent, when the send is attempted, then it still delivers using the address and content captured at submission time.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no preference) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
