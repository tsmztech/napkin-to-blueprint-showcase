---
document_type: spec
spec_type: notification
spec_id: FEAT-07.SPEC-004
spec_name: Freelancer Reply Alert to Client
spec_slug: freelancer-reply-alert-to-client
parent_feature: FEAT-07
parent_feature_name: Deliverable Review & Feedback
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Notification Spec: Freelancer Reply Alert to Client

## Overview

**Name:** Freelancer Reply Alert to Client
**ID:** FEAT-07.SPEC-004
**Type:** Notification
**Purpose:** Emails the client contact(s) entitled to a thread the instant Nadia replies in it, so a client waiting on her response is not left checking the portal to find out.
**Parent Feature:** FEAT-07 -- Deliverable Review & Feedback

## Scope and Non-Goals

**In Scope:**
- The email sent to the entitled client contact(s) the instant Nadia posts a comment, on either a Deliverable Version (FEAT-07.SPEC-001) or a Milestone (FEAT-07.SPEC-002)
- Content, delivery rules, and edge cases for this single notification, across both pin targets and both client roles (Owen and Priya)

**Non-Goals:**
- Notifying Nadia when a client comments -- that is the sibling notification FEAT-07.SPEC-003 (Client Comment Alert to Freelancer); this spec only covers Nadia's outbound replies.
- An in-app notification channel -- consistent with FEAT-07.SPEC-003, email is the product's sole MVP channel (ASMP-29); the In-App Notification Center (FEAT-29) is Later.
- A preference to turn this notification off -- excluded per XBR-30: this is the client's signal that the person they are waiting on has responded, core to the product's promise of removing the "email back-and-forth" (BRIEF.md, Problem Statement); it is not an optional marketing-style email.
- Notifying a client contact when another client contact from the same company replies -- the Feature Breakdown Brief's Communications field names only "email to the client contact when Nadia replies," not peer-to-peer client notifications; Owen and Priya each see every comment the next time they open the thread, with no separate alert for a peer's post.
- Notifying about an edit to an existing reply or a retraction -- consistent with FEAT-07.SPEC-003, only the original post fires this notification.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to the thread's entitled client contact(s), the instant Nadia's comment is recorded | Owen and Priya's Behavioral Context (user-persona.md) has each of them opening the portal from an emailed link, on their phone, in short sessions triggered by a specific event -- not habitual browsing. Email is the trigger that brings them back to see Nadia's reply, and it is the product's sole notification channel at MVP (ASMP-29). |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia replies in a thread (deliverable) | FEAT-07.SPEC-001 (Deliverable Comment Thread) | Fires immediately once the Comment write succeeds and the author is Nadia | Deliverable and project reference, client company, Nadia's identity, comment text, `posted_at` |
| Nadia replies in a thread (milestone) | FEAT-07.SPEC-002 (Milestone Comment Thread) | Fires immediately once the Comment write succeeds and the author is Nadia | Milestone and project reference, client company, Nadia's identity, comment text, `posted_at` |
| Nadia's queued offline reply syncs successfully | FEAT-07.SPEC-008 (Offline Comment Queue & Sync) | Fires immediately once a queued reply's sync completes and the Comment write succeeds | Same data as the corresponding online trigger, for whichever target the queued reply was pinned to |

## Audience and Preferences

**Recipients:** Every Client Contact (Owen and, where applicable, Priya) entitled to the thread's client company, per the Access Matrix (Milestones & Deliverables: Own-only, view and comment, for both roles) and XBR-08 (notification recipients are limited to contacts entitled to the event). Both Primary and Reviewer contacts at the same client company receive this notification, since both are entitled to view and comment on the same deliverable and milestone threads (Access Matrix: Own-only, view and comment, for both Owen and Priya). Nadia is never a recipient of her own reply's notification.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| N/A | N/A | Always on -- transactional | N/A -- per XBR-30, this is a transactional email core to the product's record-and-respond loop and cannot be disabled |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for this notification; a freelancer's reply is time-sensitive to the client's own next step (reviewing feedback further, or moving toward approval), and holding it would delay the client from seeing it.

## Content Definition

**Email (deliverable-pinned reply):**
- **Subject:** {nadia_business_name} replied on {deliverable_name}
- **Body:**
  Hi {client_first_name},

  {nadia_business_name} replied on {deliverable_name} for {project_name}:

  "{comment_text}"

  Open the thread to see the full conversation.
- **CTA (button):** Open thread -- deep-links to FEAT-07.SPEC-001 (Deliverable Comment Thread) for the specific Deliverable Version the reply was posted against

**Email (milestone-pinned reply):**
- **Subject:** {nadia_business_name} replied on {milestone_name}
- **Body:**
  Hi {client_first_name},

  {nadia_business_name} replied on the milestone "{milestone_name}" for {project_name}:

  "{comment_text}"

  Open the thread to see the full conversation.
- **CTA (button):** Open thread -- deep-links to FEAT-07.SPEC-002 (Milestone Comment Thread) for the specific Milestone the reply was posted against

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|-----------------|--------------------------|
| {client_first_name} | Client Contact -- name (first token) | Owen | Greeting renders as "Hi," |
| {nadia_business_name} | Branding Profile -- business name, falling back to Freelancer Account -- name | Studio Nadia | Renders as "Your freelancer" if neither is set, though a business or personal name is always present by the time a project is active (FEAT-01, FEAT-19) |
| {deliverable_name} | Deliverable -- derived display label (no dedicated name field exists on the entity; see FEAT-07.SPEC-001's Data Model note): the uploaded file's original file name (from `file or link`), or, for a linked asset, the source platform name plus "link" (e.g., "Figma link") | Homepage mockups | Never empty -- a file always carries an original file name at upload, and a link always resolves to a source platform name (FEAT-06) |
| {milestone_name} | Milestone -- name | Round 2 revisions | Never empty -- required at milestone creation (FEAT-04) |
| {project_name} | Project -- project_name | Brand Refresh Q1 | Never empty -- required at project creation (FEAT-01) |
| {comment_text} | Comment -- text (1--2,000 characters) | "Good catch -- I've swapped in the darker blue for this round." | Never empty -- FEAT-07.SPEC-005 rejects an empty comment before this notification's trigger can fire |

## Delivery Rules

**Batching:** None -- each reply is a single, discrete moment the client needs to know about promptly; it is never combined with other replies, even if Nadia replies to several threads close together.
**Deduplication:** At most one notification per recorded Comment, per recipient. FEAT-07.SPEC-001, FEAT-07.SPEC-002, and FEAT-07.SPEC-008 each fire this notification's trigger exactly once per successful Nadia-authored Comment write; a submission rejected for invalid length never reaches the trigger.
**Retry on failure:** Per recipient, delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure for a given recipient, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the reply itself remains visible to that client contact the next time they open the thread, regardless of this email's delivery outcome for them.
**Expiry:** This notification is never withdrawn once triggered -- delivery is retried per the rule above, and the underlying Comment record persists indefinitely even if every retry to a given recipient fails.

## Edge Cases

- **A client contact's email address bounces** -- The failure is retried per the Retry on failure rule for that recipient only; after the final failure, a delivery warning appears on the affected project for Nadia, and the reply remains visible to that contact inside the portal regardless.
- **Both Owen and Priya are entitled to the same thread** -- Each receives their own copy of the notification, addressed individually, since both roles can view and comment on the same deliverable and milestone threads (Access Matrix); this is not treated as a single shared send.
- **A client contact has been removed from the company (FEAT-18) between Nadia's reply and delivery** -- The notification is not sent to a contact whose status is no longer Active at delivery time; recipients are resolved against current entitlement, not entitlement at the moment Nadia's reply was recorded.
- **Nadia replies to a thread where the client company has only a Primary contact (no Reviewer)** -- Only Owen receives the notification; there is no requirement for a second recipient, since entitlement -- not a fixed recipient count -- determines the audience.
- **Nadia's reply, composed offline, syncs successfully hours after she composed it (FEAT-07.SPEC-008)** -- This notification fires at the moment the sync succeeds and the Comment write completes, not at Nadia's original offline composition time.
- **Nadia replies to a comment that its author has since retracted** -- The reply notification still sends normally; a retraction of an earlier comment in the same thread does not affect the delivery of a later reply's own notification.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|----------------|
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | Triggered by (inbound); Navigation (outbound) | Nadia's post on this screen fires this notification; the deliverable-reply CTA deep-links back here |
| FEAT-07.SPEC-002 (Milestone Comment Thread) | Triggered by (inbound); Navigation (outbound) | Nadia's post on this screen fires this notification; the milestone-reply CTA deep-links back here |
| FEAT-07.SPEC-008 (Offline Comment Queue & Sync) | Triggered by (inbound) | A successful offline sync of a Nadia-authored reply fires this notification |
| FEAT-07.SPEC-005 (Comment Content & Submission Validation) | References (inbound) | Only a comment that passes validation reaches this notification's trigger |
| FEAT-07.SPEC-007 (Comment Visibility & Authorization Rule) | References (inbound) | Determines which client contacts are entitled recipients for a given thread |
| FEAT-14 (Notifications (Email), FEAT-14.SPEC-001) | References (outbound) | Owns the transactional email delivery capability and delivery/bounce status reporting this notification relies on |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|---------------|-------------------|
| freelancer_reply_alert_sent | recipient role: primary / reviewer; pin target type: deliverable / milestone | Delivery succeeds, per recipient | supports success-metrics.md: "Feedback Consolidation" |
| freelancer_reply_alert_delivery_failed | retries exhausted: yes / no | All retries for a given recipient are exhausted without a successful delivery | N/A -- no Stage 2 metric measures delivery failures directly; retained so a silently undelivered alert is observable via the project's delivery warning (XBR-30) rather than invisible |

## Acceptance Criteria

**FEAT-07.SPEC-004-AC-01:** Given Nadia replies with a valid comment on a Deliverable Version thread, when FEAT-07.SPEC-001 records it, then Owen receives an email naming Nadia's business and the deliverable, quoting the reply text.

**FEAT-07.SPEC-004-AC-02:** Given Nadia replies with a valid comment on a Milestone thread, when FEAT-07.SPEC-002 records it, then the entitled client contact(s) receive an email naming Nadia's business and the milestone, quoting the reply text.

**FEAT-07.SPEC-004-AC-03:** Given both Owen and Priya are entitled to the same thread, when Nadia replies, then each receives their own separate copy of the notification.

**FEAT-07.SPEC-004-AC-04:** Given a client contact receives this email, when they tap "Open thread", then they land on the corresponding thread screen (FEAT-07.SPEC-001 or FEAT-07.SPEC-002) for the specific deliverable or milestone.

**FEAT-07.SPEC-004-AC-05:** Given Nadia's reply is recorded, when this notification's trigger fires, then no on/off preference is available to any recipient to suppress it -- it always sends.

**FEAT-07.SPEC-004-AC-06:** Given Nadia's reply is recorded during a client contact's local nighttime hours, when this notification's trigger fires, then it sends immediately with no quiet-hours hold.

**FEAT-07.SPEC-004-AC-07:** Given a client contact posts a comment (not Nadia), when the write succeeds, then this notification's trigger never fires for that comment, since the author is not Nadia.

**FEAT-07.SPEC-004-AC-08:** Given a client contact's status has changed to Removed between Nadia's reply and delivery, when recipients are resolved, then that contact does not receive this notification.

**FEAT-07.SPEC-004-AC-09:** Given a client company has only a Primary contact and no Reviewer, when Nadia replies, then only the Primary contact receives the notification.

**FEAT-07.SPEC-004-AC-10:** Given Owen's email address bounces on the first delivery attempt, when the delivery capability retries, then up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the affected project for Nadia.

**FEAT-07.SPEC-004-AC-11:** Given this notification is successfully delivered to a recipient, when the analytics signal is emitted, then a freelancer_reply_alert_sent event is recorded with that recipient's role and the pin target type.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 3 (deliverable reply, milestone reply, offline sync) | 3 |
| Preference States | 1 (always on -- transactional) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |
