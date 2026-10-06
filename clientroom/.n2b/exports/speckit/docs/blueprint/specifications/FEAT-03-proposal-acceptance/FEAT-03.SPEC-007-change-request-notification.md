---
document_type: spec
spec_type: notification
spec_id: FEAT-03.SPEC-007
spec_name: Change-Request Notification
spec_slug: change-request-notification
parent_feature: FEAT-03
parent_feature_name: Proposal Acceptance
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Notification Spec: Change-Request Notification

## Overview

**Name:** Change-Request Notification
**ID:** FEAT-03.SPEC-007
**Type:** Notification
**Purpose:** Emails Nadia immediately when Owen submits a change-request note, so she can revise and re-send the proposal without a separate status check.
**Parent Feature:** FEAT-03 -- Proposal Acceptance

## Scope and Non-Goals

**In Scope:**
- The email sent to Nadia the instant a change-request note is recorded
- Content, delivery rules, and edge cases for this single notification

**Non-Goals:**
- Notifying Owen back with any confirmation beyond the in-screen feedback already defined in FEAT-03.SPEC-002 -- the Feature Breakdown Brief's Side-Effect Inventory lists Owen's confirmation as "an on-screen confirmation state (no separate delivery rules), inline in triggering screen," not a standalone notification.
- An in-app notification channel -- product-features.md phases the In-App Notification Center (FEAT-29) as Later, outside MVP; this notification uses the product's sole MVP channel, transactional email (ASMP-29).
- A preference to turn this notification off -- excluded per XBR-30: this is the freelancer's signal that a client is waiting on her, core to the product's promise of removing the "email back-and-forth" (BRIEF.md, Problem Statement); it is not an optional marketing-style email a freelancer would reasonably disable.
- Carrying the note's full text as a distinct, separately-consumable object -- the note is quoted inline in the email body as the change-request content itself; there is no separate record-viewing screen for it within this feature (Feature Breakdown Brief, Entity-Lifecycle Coverage Matrix: no screen re-displays a past change-request note within FEAT-03).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to Nadia, the instant the change-request note is recorded | Nadia's Behavioral Context (user-persona.md) has her checking Clientroom to see whether a client has approved or responded, not watching a live feed; a change request is exactly the kind of moment BRIEF.md's Problem Statement says used to arrive as a disconnected WhatsApp screenshot -- email delivers it into the same inbox she already monitors for client business, immediately. Email is the product's sole notification channel at MVP (ASMP-29). |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|-------------------|
| Change-request note recorded | FEAT-03.SPEC-004 (Change-Request Recording) | Fires immediately once the Comment write succeeds | Proposal reference, project name, client company name, Owen's identity, the note text, `posted_at` |

## Audience and Preferences

**Recipients:** Nadia (Freelancer) -- the sole recipient, per the Access Matrix (Client Contact Management: Full for Nadia) and the Feature Breakdown Brief's Communications field ("A request-changes note emails Nadia immediately"). Owen and Priya are never recipients of this notification: Owen already sees his own submission confirmed on-screen (FEAT-03.SPEC-002), and Priya has no entitlement to proposal content (Access Matrix, XBR-08).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| N/A | N/A | Always on -- transactional | N/A -- per XBR-30, this is a transactional email core to the product's record-and-respond loop and cannot be disabled |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for this notification; a change request is time-sensitive to Nadia's own workflow (she decides when to revise), and holding it would delay her seeing that a client is waiting on a response, contradicting the immediacy the Feature Breakdown Brief specifies ("emails Nadia immediately").

## Content Definition

**Email (to Nadia):**
- **Subject:** {client_company_name} requested changes to the proposal for {project_name}
- **Body:**
  Hi {nadia_first_name},

  {owen_full_name} at {client_company_name} sent a note about the proposal for {project_name} instead of accepting it:

  "{note_text}"

  The proposal itself has not been changed. Open it to revise and re-send.
- **CTA (button):** Open proposal -- deep-links to FEAT-02 (Proposal Creation & Sending), proposal detail, for this proposal, where the note is visible for Nadia to act on

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|-----------------|--------------------------|
| {client_company_name} | Client -- client_name | Carter & Co | Never empty -- required at client creation (FEAT-01) |
| {project_name} | Project -- project_name | Brand Refresh Q1 | Never empty -- required at project creation (FEAT-01) |
| {nadia_first_name} | Freelancer Account -- name (first token) | Nadia | Greeting renders as "Hi," |
| {owen_full_name} | Client Contact -- name | Owen Carter | "The client's Primary contact" |
| {note_text} | Comment -- text (change-request variant, 1--2,000 characters) | "Could we swap the second concept for a lighter palette?" | Never empty -- FEAT-03.SPEC-004 rejects a note with zero characters before this notification's trigger can fire |

## Delivery Rules

**Batching:** None -- each change-request note is a single, discrete moment a freelancer needs to know about promptly; it is never combined with other notifications, even if several arrive close together for different proposals.
**Deduplication:** At most one notification per recorded Comment. FEAT-03.SPEC-004 creates one Comment per submitted note and fires this notification's trigger exactly once per successful write; a submission rejected for invalid length, a voided proposal, or an already-accepted proposal never reaches the trigger.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` per FEAT-14's transactional email delivery capability (ASMP-29). After the final failure, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the Comment itself remains visible to Nadia when she next opens the proposal in FEAT-02, regardless of this email's delivery outcome.
**Expiry:** This notification is never withdrawn once triggered -- delivery is retried per the rule above, and the underlying Comment record persists indefinitely as the change-request evidence even if every retry fails.

## Edge Cases

- **Nadia's email address bounces** -- The failure is retried per the Retry on failure rule; after the final failure, Nadia would not see the delivery warning by email (since her own email is the one failing), but the warning still appears on the affected project inside the product the next time she opens it (XBR-30), and the Comment remains visible there regardless.
- **Owen submits a second change-request note on the same proposal shortly after the first, before Nadia has acted** -- Each note produces its own Comment (dependency map, Comment Contention: append-only, no resolution needed beyond ordering by posted time) and its own separate notification instance; the two are never merged into one email, since each is evidence of a distinct moment.
- **Nadia is already viewing the proposal in FEAT-02 when the note is recorded** -- The notification still sends; this feature defines no live-updating suppression rule, since the email is the evidentiary trail of the moment the note arrived, not merely a live-view convenience.
- **The proposal is accepted by Owen in a separate session moments after he also tried to submit a change-request note (a race already resolved by FEAT-03.SPEC-005 in favor of one outcome)** -- If the change-request write itself succeeded before the accept won the race, this notification still fires normally for that recorded note; if the change-request write was rejected because the proposal was already Accepted (per FEAT-03.SPEC-004's outcome), no Comment was created and this notification's trigger never fires.
- **Nadia's account has multiple client contacts named similarly at the same client** -- {owen_full_name} always renders the specific Client Contact's name captured as the note's `author`, never a generic "a contact," so Nadia always knows exactly who sent the note.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|----------------|
| FEAT-03.SPEC-004 (Change-Request Recording) | Triggered by (inbound) | Fires this notification immediately once the Comment write succeeds |
| FEAT-02 (Proposal Creation & Sending) | Navigation (outbound) | Nadia's CTA deep-links to the proposal detail, where she revises and re-sends |
| FEAT-14 (Notifications (Email)) | References (outbound) | Owns the transactional email delivery capability and delivery/bounce status reporting this notification relies on |

## Analytics and Success Signals

- **change_request_notification_sent** (recipient: nadia, note character count) -- N/A -- no success metric in success-metrics.md is connected to the change-request path; "Time to Proposal Acceptance" measures only the accept path, and this feature has no other connected metric to attribute this signal to
- **change_request_notification_delivery_failed** (retries exhausted: yes / no) -- N/A -- no Stage 2 metric measures delivery failures directly; retained so a silently undelivered change-request alert is observable via the project's delivery warning (XBR-30) rather than invisible

## Acceptance Criteria

**FEAT-03.SPEC-007-AC-01:** Given Owen submits a valid change-request note, when FEAT-03.SPEC-004 records it, then Nadia receives an email with the subject naming the client company and project, quoting the note text in the body.

**FEAT-03.SPEC-007-AC-02:** Given Nadia receives the change-request email, when she taps "Open proposal", then she lands on the proposal detail in FEAT-02 with the note visible for her to act on.

**FEAT-03.SPEC-007-AC-03:** Given the change-request note is recorded, when this notification's trigger fires, then no on/off preference is available to Nadia to suppress it -- it always sends.

**FEAT-03.SPEC-007-AC-04:** Given the change-request note is recorded during Nadia's local nighttime hours, when this notification's trigger fires, then it sends immediately with no quiet-hours hold.

**FEAT-03.SPEC-007-AC-05:** Given Owen's change-request submission is rejected because the proposal was already Accepted (FEAT-03.SPEC-004's outcome), when no Comment is created, then this notification's trigger never fires.

**FEAT-03.SPEC-007-AC-06:** Given Nadia's email address bounces on the first delivery attempt, when the delivery capability retries, then up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the affected project.

**FEAT-03.SPEC-007-AC-07:** Given all retries for Nadia's email are exhausted, when the final failure occurs, then a delivery warning appears on the affected project and the Comment remains visible to Nadia inside the product regardless.

**FEAT-03.SPEC-007-AC-08:** Given Owen submits two change-request notes on the same proposal in succession, when each is recorded, then Nadia receives two separate emails, one per note, never merged into one.

**FEAT-03.SPEC-007-AC-09:** Given the change-request notification is successfully delivered, when the analytics signal is emitted, then a change_request_notification_sent event is recorded with the note's character count.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- transactional) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
