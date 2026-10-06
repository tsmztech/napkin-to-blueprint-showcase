---
document_type: spec
spec_type: notification
spec_id: FEAT-07.SPEC-003
spec_name: Client Comment Alert to Freelancer
spec_slug: client-comment-alert-to-freelancer
parent_feature: FEAT-07
parent_feature_name: Deliverable Review & Feedback
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Notification Spec: Client Comment Alert to Freelancer

## Overview

**Name:** Client Comment Alert to Freelancer
**ID:** FEAT-07.SPEC-003
**Type:** Notification
**Purpose:** Emails Nadia the instant a client contact posts a comment on a deliverable or a milestone, so she learns feedback is waiting without checking the portal on a schedule.
**Parent Feature:** FEAT-07 -- Deliverable Review & Feedback

## Scope and Non-Goals

**In Scope:**
- The email sent to Nadia the instant Owen or Priya posts a comment, on either a Deliverable Version (FEAT-07.SPEC-001) or a Milestone (FEAT-07.SPEC-002)
- Content, delivery rules, and edge cases for this single notification, across both pin targets

**Non-Goals:**
- Notifying Nadia when she herself posts a reply -- this notification exists only for client-originated comments; Nadia's own activity never triggers it.
- An email to the client confirming their comment was received -- the Feature Breakdown Brief's Side-Effect Inventory defines the confirmation as inline, on-screen feedback in the triggering screen (FEAT-07.SPEC-001 / FEAT-07.SPEC-002), not a standalone notification.
- An in-app notification channel -- product-features.md phases the In-App Notification Center (FEAT-29) as Later, outside MVP; email is the product's sole MVP channel (ASMP-29).
- A preference to turn this notification off -- excluded per XBR-30: this is Nadia's signal that a client is waiting on her, core to the product's replacement for WhatsApp screenshots; it is not an optional marketing-style email.
- Notifying Nadia about an edit to an existing comment or a retraction -- the Feature Breakdown Brief's Communications field names only "email to Nadia when a client comments," not edits or retractions; a retraction or edit is visible the next time Nadia opens the thread, with no separate alert.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to Nadia, the instant a client contact's comment is recorded | Nadia's Behavioral Context (user-persona.md) has her opening Clientroom to check whether a client has responded, not watching a live feed; email is the product's sole notification channel at MVP (ASMP-29) and delivers the alert into the same inbox she already monitors for client business. |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client contact posts a comment (deliverable) | FEAT-07.SPEC-001 (Deliverable Comment Thread) | Fires immediately once the Comment write succeeds and the author is Owen or Priya (never Nadia) | Deliverable and project reference, client company name, author identity and role, comment text, `posted_at` |
| Client contact posts a comment (milestone) | FEAT-07.SPEC-002 (Milestone Comment Thread) | Fires immediately once the Comment write succeeds and the author is Owen or Priya | Milestone and project reference, client company name, author identity and role, comment text, `posted_at` |
| Client contact's queued offline comment syncs successfully | FEAT-07.SPEC-008 (Offline Comment Queue & Sync) | Fires immediately once a queued client comment's sync completes and the Comment write succeeds | Same data as the corresponding online trigger above, for whichever target (deliverable or milestone) the queued comment was pinned to |

## Audience and Preferences

**Recipients:** Nadia (Freelancer) -- the sole recipient, per the Access Matrix (Milestones & Deliverables: Full for Nadia) and XBR-08 (notification recipients are limited to contacts entitled to the event) and the Feature Breakdown Brief's Communications field ("Email to Nadia when a client comments"). Neither Owen nor Priya ever receives this notification -- it exists to reach Nadia specifically.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| N/A | N/A | Always on -- transactional | N/A -- per XBR-30, this is a transactional email core to the product's record-and-respond loop and cannot be disabled |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for this notification; a client comment is time-sensitive to Nadia's own response, and holding it would delay her seeing that a client is waiting, contradicting the Feature Breakdown Brief's "emails Nadia" immediacy.

## Content Definition

**Email (deliverable-pinned comment):**
- **Subject:** New feedback from {client_company_name} on {deliverable_name}
- **Body:**
  Hi {nadia_first_name},

  {author_full_name} ({author_role}) at {client_company_name} left a comment on {deliverable_name} for {project_name}:

  "{comment_text}"

  Open the thread to see the full conversation and reply.
- **CTA (button):** Open thread -- deep-links to FEAT-07.SPEC-001 (Deliverable Comment Thread) for the specific Deliverable Version the comment was posted against

**Email (milestone-pinned comment):**
- **Subject:** New feedback from {client_company_name} on {milestone_name}
- **Body:**
  Hi {nadia_first_name},

  {author_full_name} ({author_role}) at {client_company_name} left a comment on the milestone "{milestone_name}" for {project_name}:

  "{comment_text}"

  Open the thread to see the full conversation and reply.
- **CTA (button):** Open thread -- deep-links to FEAT-07.SPEC-002 (Milestone Comment Thread) for the specific Milestone the comment was posted against

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|-----------------|--------------------------|
| {nadia_first_name} | Freelancer Account -- name (first token) | Nadia | Greeting renders as "Hi," |
| {author_full_name} | Client Contact -- name | Owen Carter | Never empty -- required at contact creation (FEAT-18) |
| {author_role} | Client Contact -- role | Primary Contact / Reviewer | Never empty -- role is required and always one of the two values |
| {client_company_name} | Client -- client_name | Carter & Co | Never empty -- required at client creation (FEAT-01) |
| {deliverable_name} | Deliverable -- derived display label (no dedicated name field exists on the entity; see FEAT-07.SPEC-001's Data Model note): the uploaded file's original file name (from `file or link`), or, for a linked asset, the source platform name plus "link" (e.g., "Figma link") | Homepage mockups | Never empty -- a file always carries an original file name at upload, and a link always resolves to a source platform name (FEAT-06) |
| {milestone_name} | Milestone -- name | Round 2 revisions | Never empty -- required at milestone creation (FEAT-04) |
| {project_name} | Project -- project_name | Brand Refresh Q1 | Never empty -- required at project creation (FEAT-01) |
| {comment_text} | Comment -- text (1--2,000 characters) | "Could we see this in the darker blue from the last round?" | Never empty -- FEAT-07.SPEC-005 rejects an empty comment before this notification's trigger can fire |

## Delivery Rules

**Batching:** None -- each client comment is a single, discrete moment Nadia needs to know about promptly; it is never combined with other comments, even if several arrive close together on the same or different threads.
**Deduplication:** At most one notification per recorded Comment. FEAT-07.SPEC-001, FEAT-07.SPEC-002, and FEAT-07.SPEC-008 each fire this notification's trigger exactly once per successful client-authored Comment write; a submission rejected for invalid length never reaches the trigger.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the comment itself remains visible to Nadia the next time she opens the thread, regardless of this email's delivery outcome.
**Expiry:** This notification is never withdrawn once triggered -- delivery is retried per the rule above, and the underlying Comment record persists indefinitely as the feedback record even if every retry fails.

## Edge Cases

- **Nadia's email address bounces** -- The failure is retried per the Retry on failure rule; after the final failure, a delivery warning still appears on the affected project inside the product the next time she opens it (XBR-30), and the comment remains visible there regardless.
- **Owen and Priya each post a comment on the same deliverable within moments of each other** -- Each comment produces its own Comment record (dependency map, Comment Contention: append-only, ordered by posted time) and its own separate notification instance; the two are never merged into one email.
- **A client contact posts a comment while Nadia is already viewing that same thread** -- The notification still sends; this feature defines no live-updating suppression rule, since the email is the evidentiary trail of the moment the comment arrived, not merely a live-view convenience.
- **The client comment is retracted by its author moments after posting, before Nadia opens the email** -- The notification still sends and its CTA still opens the thread; the thread now shows the retracted placeholder in that comment's place, consistent with retraction being a soft, visible change rather than a silent disappearance (FEAT-07.SPEC-006).
- **A comment queued offline (FEAT-07.SPEC-008) syncs successfully hours after it was composed** -- This notification fires at the moment the sync succeeds and the Comment write completes, not at the moment the client originally composed it offline, so Nadia's alert reflects when the feedback actually entered the record.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|----------------|
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | Triggered by (inbound); Navigation (outbound) | A client's post on this screen fires this notification; the deliverable-comment CTA deep-links back here |
| FEAT-07.SPEC-002 (Milestone Comment Thread) | Triggered by (inbound); Navigation (outbound) | A client's post on this screen fires this notification; the milestone-comment CTA deep-links back here |
| FEAT-07.SPEC-008 (Offline Comment Queue & Sync) | Triggered by (inbound) | A successful offline sync of a client-authored comment fires this notification |
| FEAT-07.SPEC-005 (Comment Content & Submission Validation) | References (inbound) | Only a comment that passes validation reaches this notification's trigger |
| FEAT-14 (Notifications (Email), FEAT-14.SPEC-001) | References (outbound) | Owns the transactional email delivery capability and delivery/bounce status reporting this notification relies on |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|---------------|-------------------|
| client_comment_alert_sent | recipient: nadia; pin target type: deliverable / milestone; author role | Delivery succeeds | supports success-metrics.md: "Feedback Consolidation" |
| client_comment_alert_delivery_failed | retries exhausted: yes / no | All retries are exhausted without a successful delivery | N/A -- no Stage 2 metric measures delivery failures directly; retained so a silently undelivered alert is observable via the project's delivery warning (XBR-30) rather than invisible |

## Acceptance Criteria

**FEAT-07.SPEC-003-AC-01:** Given Owen posts a valid comment on a Deliverable Version, when FEAT-07.SPEC-001 records it, then Nadia receives an email with the subject naming the client company and the deliverable, quoting the comment text.

**FEAT-07.SPEC-003-AC-02:** Given Priya posts a valid comment on a Milestone, when FEAT-07.SPEC-002 records it, then Nadia receives an email with the subject naming the client company and the milestone, quoting the comment text.

**FEAT-07.SPEC-003-AC-03:** Given Nadia receives either variant of this email, when she taps "Open thread", then she lands on the corresponding thread screen (FEAT-07.SPEC-001 or FEAT-07.SPEC-002) for the specific deliverable or milestone the comment was posted against.

**FEAT-07.SPEC-003-AC-04:** Given a client comment is recorded, when this notification's trigger fires, then no on/off preference is available to Nadia to suppress it -- it always sends.

**FEAT-07.SPEC-003-AC-05:** Given a client comment is recorded during Nadia's local nighttime hours, when this notification's trigger fires, then it sends immediately with no quiet-hours hold.

**FEAT-07.SPEC-003-AC-06:** Given Nadia posts a comment herself (a reply), when the write succeeds, then this notification's trigger never fires, since the author is Nadia, not a client contact.

**FEAT-07.SPEC-003-AC-07:** Given Nadia's email address bounces on the first delivery attempt, when the delivery capability retries, then up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the affected project.

**FEAT-07.SPEC-003-AC-08:** Given Owen and Priya each post a comment on the same deliverable moments apart, when each is recorded, then Nadia receives two separate emails, one per comment, never merged.

**FEAT-07.SPEC-003-AC-09:** Given a client's comment composed offline syncs successfully hours later via FEAT-07.SPEC-008, when the sync completes, then this notification fires at that moment, not at the original offline composition time.

**FEAT-07.SPEC-003-AC-10:** Given this notification is successfully delivered, when the analytics signal is emitted, then a client_comment_alert_sent event is recorded with the pin target type and author role.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 3 (deliverable post, milestone post, offline sync) | 3 |
| Preference States | 1 (always on -- transactional) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
