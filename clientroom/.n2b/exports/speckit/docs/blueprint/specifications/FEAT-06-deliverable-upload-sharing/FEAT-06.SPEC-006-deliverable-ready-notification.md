---
document_type: spec
spec_type: notification
spec_id: FEAT-06.SPEC-006
spec_name: Deliverable Ready Notification
spec_slug: deliverable-ready-notification
parent_feature: FEAT-06
parent_feature_name: Deliverable Upload & Sharing
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Notification Spec: Deliverable Ready Notification

## Overview

**Name:** Deliverable Ready Notification
**ID:** FEAT-06.SPEC-006
**Type:** Notification
**Purpose:** Emails the relevant client contacts once an uploaded or linked deliverable is fully ready to review, so they never have to be told by Nadia directly that something is waiting on them.
**Parent Feature:** FEAT-06 -- Deliverable Upload & Sharing

## Scope and Non-Goals

**In Scope:**
- The email sent to relevant client contacts when a deliverable becomes fully ready (upload complete, or link confirmed reachable)
- Preference, retry, and expiry behavior for this notification
- The single-deliverable content; this notification is never batched, per its Delivery Rules below

**Non-Goals:**
- Deciding whether an upload or link check has fully completed -- owned by FEAT-06.SPEC-003 (Resumable Upload Handling) and FEAT-06.SPEC-004 (Linked Asset Reachability Check); this spec begins where their completion trigger fires
- The transactional email delivery mechanism itself (sending, bounce and delivery status reporting) -- owned by FEAT-14.SPEC-001 (Transactional Email Delivery), the capability this notification is composed for and dispatched through
- Notifying Nadia that her own upload succeeded -- that is an in-screen confirmation on FEAT-06.SPEC-001 ("Deliverable attached" toast), not a delivery-channel notification to the freelancer
- Comment or approval notifications once a deliverable is being reviewed -- owned by FEAT-07 (Deliverable Review & Feedback) and FEAT-08 (Milestone Approval), which govern the next steps in a client contact's journey after this notification

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, when a deliverable becomes fully ready | BRIEF.md's Ecosystem & Integrations states clients will not install an app, making email the sole channel that reaches Owen and Priya outside a session; both personas' Behavioral Context describes opening the portal from an email link, often on a phone |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| File upload completes fully | FEAT-06.SPEC-003 (Resumable Upload Handling) | Fires only when the Deliverable's status transitions to Active from a completed transfer (never for a partial file, XBR-12) | Deliverable reference, milestone name, project name, freelancer's Branding Profile |
| Linked asset confirmed reachable | FEAT-06.SPEC-004 (Linked Asset Reachability Check) | Fires only when link_status transitions to reachable | Deliverable reference, milestone name, project name, freelancer's Branding Profile |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact) and Priya (Client Reviewer Contact) -- every Active contact on the Client that owns the Milestone's Project, per the Access Matrix in user-persona.md (both roles have Own-only visibility into Milestones & Deliverables and both are entitled to review a deliverable, per XBR-08's "notification recipients are limited to contacts entitled to the event"). Nadia does not receive this notification -- her own upload-success feedback is the in-screen confirmation on FEAT-06.SPEC-001.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| This notification is a transactional record email | Cannot be switched off | Always on | N/A -- XBR-30: transactional emails core to the record always send; a new deliverable ready to review is exactly this kind of record-carrying event, not an optional update |

**Quiet Hours:** N/A -- the product definition establishes no quiet-hours window for client-facing transactional notifications; XBR-30 treats this class of email as always-sending, and no Stage 2 document defines a quiet-hours preference surface for client contacts.

## Content Definition

**Email:**
- **Subject:** {freelancer_business_name}: a new deliverable is ready to review
- **Body:**
  Hi {recipient_first_name},

  {freelancer_business_name} has shared a new deliverable on {project_name}, for the milestone "{milestone_name}": {deliverable_label}.

  Open it to review and leave feedback whenever you're ready.
- **CTA (button):** Review deliverable -- deep-links through FEAT-05 (Client Portal Access, magic-link sign-in) to FEAT-07.SPEC-001 (Deliverable Comment Thread), the deliverable view, for the specific Deliverable referenced by this notification

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_business_name} | Freelancer Account -- business_name | Nadia Chen Design | Never empty -- business_name is required before the first invoice, but for this earlier-lifecycle event it falls back to the Freelancer Account's name field, which is required from sign-up |
| {recipient_first_name} | Client Contact -- name (first token) | Owen | Never empty -- name is required at contact creation (FEAT-18) |
| {project_name} | Project -- project_name | Brand Refresh Q1 | Never empty -- project_name is required at project creation (FEAT-01) |
| {milestone_name} | Milestone -- name | Concept Round 1 | Never empty -- name is required at milestone creation (FEAT-04) |
| {deliverable_label} | Derived -- "a file" for kind = uploaded file, or "a linked file" for kind = linked external asset | a file / a linked file | Never empty -- kind is required and always one of the two values at the point this notification fires |

## Delivery Rules

**Batching:** None -- each deliverable that becomes ready sends its own individual email. Two deliverables becoming ready in close succession (e.g., two milestones on the same project) each produce a separate notification, because each names a specific milestone and deliverable a contact needs to act on individually; collapsing them would obscure which milestone is actually waiting.
**Deduplication:** At most one "ready" notification per Deliverable. A Deliverable that becomes Active exactly once (this feature never re-fires the trigger for an already-Active deliverable); a later replacement (FEAT-17, round 2+) produces its own, separate ready notification owned by FEAT-17, not a re-send of this one.
**Retry on failure:** Delivery failure is retried per FEAT-14.SPEC-001's transactional email capability, up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`. After the final failure, the delivery is marked Failed and surfaced to Nadia as a delivery warning on the affected project (XBR-30) -- the deliverable itself remains visible and reachable in-product regardless of email delivery outcome.
**Expiry:** This notification does not expire in the sense of becoming stale content -- a deliverable that was ready yesterday is still exactly as ready today, so a delayed or retried delivery still carries fully accurate content. If all retries are exhausted, delivery simply stops (see Retry on failure); there is no "too late to send" cutoff distinct from the retry exhaustion itself.

## Edge Cases

- **Deliverable removed between trigger and delivery** -- The notification's CTA still deep-links to the deliverable view; if the contact opens it after removal, FEAT-07's own removed-deliverable handling applies (out of this spec's scope). The email itself is not cancelled, since XBR-11/XBR-05 treat the removal as a subsequent, separately logged event, not a reason to suppress a record that a deliverable was, in fact, shared and ready at that moment.
- **All of a client's contacts are Removed (status = Removed) between trigger and delivery** -- If zero Active contacts remain entitled to this notification at delivery time, delivery is skipped for that client entirely (there is no recipient to send to); this is a silent no-send, not a failure, since XBR-27 treats contact removal as ending access, not as something a deliverable notification should work around.
- **A contact's preference or channel cannot be turned off (this notification has none)** -- Not applicable: Preference Controls above establish this notification always sends: there is no mid-flight preference change to collide with.
- **Quiet hours colliding with expiry** -- Not applicable: this notification defines no quiet-hours window (see Audience and Preferences), so no collision exists.
- **Two deliverables on the same milestone become ready in the same minute (a file upload and, separately, a re-check confirming a previously flagged link, both resolving together)** -- Each produces its own individual email per the no-batching rule; a recipient may receive two emails in quick succession, which is intentional so each remains individually actionable and traceable.
- **The freelancer's Branding Profile changes between trigger and delivery** -- The email renders with the Branding Profile as it stands at delivery time, not at trigger time, consistent with branding being a presentation concern applied at render (XBR-31), not a fact captured at the moment the deliverable became ready.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-003 (Resumable Upload Handling) | Triggered by (inbound) | Full upload completion fires this notification |
| FEAT-06.SPEC-004 (Linked Asset Reachability Check) | Triggered by (inbound) | A reachable link outcome fires this notification |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | This notification is composed and handed to the shared transactional email capability for sending and delivery/bounce status reporting |
| FEAT-05 (Client Portal Access) | Navigation (outbound) | The CTA deep-links through magic-link sign-in |
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | Navigation (outbound) | The CTA's ultimate destination is the client-facing deliverable view |
| FEAT-13 (Immutable Activity & Audit Trail) | References (outbound) | This notification's send is itself a record-worthy event captured in the activity trail alongside the upload/link event that triggered it |

## Analytics and Success Signals

- **deliverable_ready_notification_sent** (channel: email; deliverable kind: file / link; recipient role: primary / reviewer) -- N/A -- this feature's success-metrics slice contains only "Deliverable Upload Reliability", which measures whether the upload/link transfer itself completes, not whether the resulting notification is sent; retained for feature-level visibility (this event is the deliverable-ready signal reaching its audience, the outcome "Deliverable Upload Reliability" exists to make possible)
- **deliverable_ready_notification_delivery_failed** (retry count exhausted) -- N/A -- same reason: no metric in this feature's slice measures notification delivery outcomes (delivery reliability is Notifications (Email)'s own connected metric, not this feature's); retained so a silent delivery failure remains observable
- **deliverable_ready_notification_opened** (recipient role) -- N/A -- no metric in this feature's slice measures open rate; retained for feature-level visibility only

## Acceptance Criteria

**FEAT-06.SPEC-006-AC-01:** Given Nadia's file upload on a milestone completes fully, when FEAT-06.SPEC-003 signals completion, then Owen and Priya each receive an email with the subject "{freelancer_business_name}: a new deliverable is ready to review".

**FEAT-06.SPEC-006-AC-02:** Given Nadia's pasted link is confirmed reachable, when FEAT-06.SPEC-004 signals the reachable outcome, then Owen and Priya each receive the same email, with {deliverable_label} rendering as "a linked file".

**FEAT-06.SPEC-006-AC-03:** Given a file upload is only partially complete, when its progress is checked, then no notification is sent -- delivery waits for full completion (XBR-12).

**FEAT-06.SPEC-006-AC-04:** Given a pasted link is flagged as unreachable, when the check completes, then no notification is sent to Owen or Priya.

**FEAT-06.SPEC-006-AC-05:** Given Owen opens this notification's "Review deliverable" CTA, when he is not currently signed in, then he is carried through magic-link sign-in (FEAT-05) and lands on the specific deliverable's view in FEAT-07.

**FEAT-06.SPEC-006-AC-06:** Given this notification is a transactional record email, when Nadia checks her account's notification preferences, then no control exists to turn it off.

**FEAT-06.SPEC-006-AC-07:** Given email delivery of this notification fails, when FEAT-14.SPEC-001 retries it up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` and all attempts fail, then a delivery warning appears on the affected project for Nadia (XBR-30).

**FEAT-06.SPEC-006-AC-08:** Given a client has zero Active contacts remaining at the moment a deliverable becomes ready, when the notification would otherwise fire, then delivery is silently skipped and no failure is recorded.

**FEAT-06.SPEC-006-AC-09:** Given a deliverable is removed shortly after this notification is sent, when the email is nonetheless delivered, then delivery is not cancelled and the email is not recalled.

**FEAT-06.SPEC-006-AC-10:** Given two deliverables on different milestones of the same project become ready within the same minute, when both trigger, then two separate emails are sent -- they are never batched into one.

**FEAT-06.SPEC-006-AC-11:** Given a Deliverable that already became Active once, when any process re-evaluates its readiness, then this notification does not fire a second time for the same Deliverable.

**FEAT-06.SPEC-006-AC-12:** Given Nadia's Branding Profile changes between the moment a deliverable becomes ready and the moment the email actually sends, when the email renders, then it reflects the Branding Profile as it stands at send time.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 2 (upload completion, link reachable) | 2 |
| Preference States | 1 (always-on, no preference surface) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |
