---
document_type: spec
spec_type: notification
spec_id: FEAT-08.SPEC-007
spec_name: Milestone Approval Confirmation Notification
spec_slug: milestone-approval-confirmation-notification
parent_feature: FEAT-08
parent_feature_name: Milestone Approval
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Notification Spec: Milestone Approval Confirmation Notification

## Overview

**Name:** Milestone Approval Confirmation Notification
**ID:** FEAT-08.SPEC-007
**Type:** Notification
**Purpose:** Confirms to both Owen and Nadia, the moment an approval is recorded, exactly what happened -- so the client-side record and the freelancer-side record of the same event match from the start.
**Parent Feature:** FEAT-08 -- Milestone Approval

## Scope and Non-Goals

**In Scope:**
- The confirmation email sent to Owen and to Nadia when a milestone approval is successfully recorded
- Preference, retry, and expiry behavior for this confirmation

**Non-Goals:**
- Notifying anyone about the resulting invoice -- the auto-issued invoice sends its own notification, owned by Invoice Generation & Sending (FEAT-09); this spec covers only the approval confirmation itself (product-features.md, Communications).
- Notifying anyone about a reopen -- the Brief names no separate email for reopen; the reopen's non-silence is carried entirely by its Activity Log entry (FEAT-13), not a notification (Side-Effect Inventory).
- Notifying Priya -- she is not the approving contact and the Access Matrix limits invoice-and-approval-adjacent communications to the Primary Contact; a Reviewer receiving a financial-commitment confirmation would exceed her role's entitlement (XBR-08).
- In-app notification center delivery -- product-features.md's Communications field for this feature names only the email channel; an in-app surface is owned separately by In-App Notification Center (FEAT-29, Later phase), which is out of scope for this MVP-phase spec.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to both Owen and Nadia, when an approval is recorded | Owen's portal sessions are short and triggered by a specific email link (user-persona.md, Behavioral Context) -- he is not routinely inside the product to see a status change happen live; Nadia works from her own inbox and desktop workflow and needs the same confirmation as her defensible, timestamped record without having to reopen the project to check |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Milestone approval successfully recorded | FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard) | Fires immediately and only after FEAT-08.SPEC-003 confirms the approval write committed | Milestone name and project, `approved_at`, `approved_by` (Owen's identity), Client `client_name`, Freelancer Account contact details for Nadia |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact) and Nadia (the Freelancer), per the Brief's Communications field ("Confirmation email to Owen and Nadia when approval is recorded"). Both are entitled to this content under the Access Matrix: Owen approved the milestone himself, and Nadia's Full access to Milestones & Deliverables covers every event on her own account's milestones.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- always sent | -- | Always on | -- (this is a transactional record email, not an optional notification) |

This confirmation is a transactional email core to the record (XBR-30): it cannot be switched off by either recipient's notification preferences, in the same way a payment confirmation cannot -- it is the evidentiary echo of a permanent, timestamped decision, not a discretionary update.

**Quiet Hours:** N/A -- the product defines quiet hours for optional, non-transactional notifications only (XBR-30); this confirmation is transactional and sends immediately regardless of the time of day for either recipient, consistent with the approval event itself being permanent and time-stamped at the moment it occurred.

## Content Definition

**Email (to Owen):**
- **Subject:** You approved {milestone_name}
- **Body:**
  Hi {owen_first_name},

  This confirms your approval of {milestone_name} on {project_name}, recorded on {approved_at_formatted}.

  This records your approval and issues the next invoice in the payment schedule, if one applies. You'll receive it separately if so.
- **CTA (button):** View milestone -- deep-links to FEAT-08.SPEC-001 (Milestone Review & Approval Screen) for this milestone

**Email (to Nadia):**
- **Subject:** {client_name} approved {milestone_name}
- **Body:**
  Hi {nadia_first_name},

  {approved_by_name} at {client_name} approved {milestone_name} on {project_name} on {approved_at_formatted}.

  If the payment schedule includes an invoice for this milestone, it has been generated and sent automatically -- no action needed from you.
- **CTA (button):** View milestone -- deep-links to the milestone's detail in her own project view (FEAT-01)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {milestone_name} | Milestone -- name | Homepage Redesign | Never empty -- required at milestone creation (FEAT-04.SPEC-003) |
| {project_name} | Project -- project_name | Acme Rebrand | Never empty -- required at project creation (FEAT-01) |
| {approved_at_formatted} | Milestone -- approved_at, rendered in each recipient's own time zone (FEAT-15) | September 27, 2026, 3:14 PM | Never empty -- set atomically by FEAT-08.SPEC-003 at the moment this notification's trigger fires |
| {approved_by_name} | Client Contact -- name (the approving contact, from Milestone.approved_by) | Owen Marsh | Never empty -- set atomically by FEAT-08.SPEC-003 alongside approved_at |
| {client_name} | Client -- client_name | Acme Co. | Never empty -- required at client creation (FEAT-01) |
| {owen_first_name} | Client Contact -- name (first name portion) | Owen | Greeting renders as "Hi," |
| {nadia_first_name} | Freelancer Account -- name (first name portion) | Nadia | Greeting renders as "Hi," |

## Delivery Rules

**Batching:** None -- each approval is its own distinct, permanent event and is confirmed individually; two milestones approved close together each produce their own separate confirmation to each recipient, never merged into one summary email.
**Deduplication:** At most one confirmation email per recipient per approval event. FEAT-08.SPEC-003's exactly-once guarantee on the approval write itself is the deduplication boundary: this notification's trigger fires once per confirmed write, so a retried or refused approval attempt (stale, unauthorized, connectivity, or write failure, per FEAT-08.SPEC-003) never produces a confirmation, since the trigger condition -- a confirmed committed write -- was never met.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning on the project (XBR-30) -- for the copy addressed to Owen as well as her own, since a client contact never sees the freelancer's own delivery-warning surface and Nadia is the one positioned to notice and follow up.
**Expiry:** This confirmation never expires undelivered in the sense of becoming pointless to send late -- the approval it confirms is a permanent record, so a delayed delivery (after retries) still carries accurate, still-true information whenever it eventually lands. There is no cutoff after which the email is withheld; the retry window in the rule above is the only limit, after which delivery is treated as failed (surfaced as a warning) rather than expired.

## Edge Cases

- **The Milestone record is later reopened after this confirmation was sent but before Nadia or Owen reads it** -- The confirmation remains accurate as sent: it describes the approval event that genuinely occurred at that timestamp, and a later reopen is a separate, new event that does not retroactively make the original confirmation false or worth recalling.
- **Owen's or Nadia's email address changes between the approval and delivery** -- Not applicable in practice, since this notification fires and is handed to the delivery capability immediately upon the confirmed write (no batching or delay); if a bounce nonetheless occurs because an address was already invalid, the standard retry-then-warning rule above applies.
- **Owen's copy fails to deliver but Nadia's succeeds (or vice versa)** -- Each recipient's copy is tracked and retried independently; one recipient's successful delivery has no bearing on the other's retry count or warning surfacing.
- **The milestone carries `no_separate_charge` (no invoice will follow)** -- Owen's copy still states the general consent language ("issues the next invoice ... if one applies") accurately, since it is conditional wording, not a promise of an invoice that will not arrive; Nadia's copy is equally accurate for the same reason.
- **A second, later approval (after a reopen) is recorded for the same milestone** -- This notification fires again as its own, independent trigger instance, with its own `approved_at`/`approved_by` values; it is never treated as a duplicate of the first confirmation, since each approval is a genuinely distinct, permanent event.
- **Quiet hours colliding with expiry** -- N/A, since this transactional confirmation observes neither quiet hours nor an expiry cutoff (both stated above); there is no collision to resolve.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard) | Triggered by (inbound) | A confirmed successful approval write fires this notification |
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Navigation (outbound) | Owen's CTA deep-links here |
| FEAT-01 (Client & Project Management) | Navigation (outbound) | Nadia's CTA deep-links to the milestone's detail in her own project view |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | Underlying delivery, retry, and bounce/failure reporting capability this notification is sent through |

## Analytics and Success Signals

- **milestone_approval_confirmation_delivered** (recipient: owen / nadia) -- supports success-metrics.md: "Notification Delivery Reliability"
- **milestone_approval_confirmation_delivery_failed** (recipient: owen / nadia, retry_count) -- supports success-metrics.md: "Notification Delivery Reliability"
- **milestone_approval_confirmation_opened** (recipient: owen / nadia) -- N/A -- no Stage 2 metric measures open rates for this specific confirmation; retained as a standard delivery-quality signal alongside the delivered/failed pair above.

## Acceptance Criteria

**FEAT-08.SPEC-007-AC-01:** Given Owen's approval of a milestone is successfully recorded, when FEAT-08.SPEC-003 confirms the write, then Owen receives an email with subject "You approved {milestone_name}" and Nadia receives an email with subject "{client_name} approved {milestone_name}."

**FEAT-08.SPEC-007-AC-02:** Given Owen opens his confirmation email, when he taps "View milestone," then he lands on FEAT-08.SPEC-001 for that milestone.

**FEAT-08.SPEC-007-AC-03:** Given Nadia opens her confirmation email, when she taps "View milestone," then she lands on that milestone's detail in her own project view.

**FEAT-08.SPEC-007-AC-04:** Given an approval attempt is refused by FEAT-08.SPEC-003 as stale, unauthorized, a connectivity failure, or a write failure, when the refusal occurs, then this notification never fires for that attempt.

**FEAT-08.SPEC-007-AC-05:** Given neither Owen nor Nadia has any way to opt out of this confirmation, when their respective notification preferences are checked, then no preference control exists for it and it always sends.

**FEAT-08.SPEC-007-AC-06:** Given an approval is recorded at any hour, when this notification fires, then it sends immediately regardless of either recipient's configured quiet hours, since this confirmation is transactional.

**FEAT-08.SPEC-007-AC-07:** Given delivery of Owen's copy fails, when the failure occurs, then it is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, and after the final failure Nadia sees a delivery warning on the project.

**FEAT-08.SPEC-007-AC-08:** Given delivery of Nadia's copy fails while Owen's copy succeeds, when this is observed, then Nadia's copy is retried independently of Owen's successful delivery.

**FEAT-08.SPEC-007-AC-09:** Given the approved milestone carries `no_separate_charge`, when Owen's confirmation is generated, then its consent language about issuing the next invoice remains accurate as conditional wording, with no false promise of an invoice.

**FEAT-08.SPEC-007-AC-10:** Given a milestone is reopened and approved a second time, when the second approval is confirmed, then this notification fires again as an independent instance with the second approval's own `approved_at` and `approved_by` values.

**FEAT-08.SPEC-007-AC-11:** Given Priya is a Reviewer contact on the same client company, when a milestone is approved by Owen, then Priya receives no copy of this confirmation.

**FEAT-08.SPEC-007-AC-12:** Given this confirmation is delivered successfully to both recipients, when delivery completes, then the milestone_approval_confirmation_delivered event fires once per recipient.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no preference exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |
