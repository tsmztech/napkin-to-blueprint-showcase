---
document_type: spec
spec_type: notification
spec_id: FEAT-22.SPEC-009
spec_name: Support View Recorded Notification
spec_slug: support-view-recorded-notification
parent_feature: FEAT-22
parent_feature_name: Operator Read-Only Support Access
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Notification Spec: Support View Recorded Notification

## Overview

**Name:** Support View Recorded Notification
**ID:** FEAT-22.SPEC-009
**Type:** Notification
**Purpose:** Shows Maya an in-app note each time support views her household, so the visible record the Brief promises reaches her at the moment it happens, not only when she goes looking.
**Parent Feature:** FEAT-22 -- Operator Read-Only Support Access

## Scope and Non-Goals

**In Scope:**
- The in-app note delivered when an access session opens, and again when it closes
- Both variants' exact content, tied to the request's kind

**Non-Goals:**
- The persistent, browsable record of every past session -- owned by FEAT-22.SPEC-003 (Household Support Access Record), which this note only announces the arrival of a new entry for
- Any channel beyond in-app -- excluded per this feature's own definition: the dependency map's External Touchpoints table states "Its support-visit note (FEAT-22.SPEC-009) is in-app," and FEAT-22 inventories no Integration spec that could carry an email or push capability for this note
- Notifying Riley -- this notification exists to tell the household support was viewed; Riley is the one doing the viewing and has no reciprocal notification need
- Any household-data content beyond the kind and timing of the visit -- excluded per FEAT-22.SPEC-007's suppression posture: this note never surfaces what Riley actually saw, only that a session occurred and why

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, on every session open and every session close | Support access is rare and household-facing trust matters more than urgency; an in-app note that Maya sees on her next visit is proportionate, and the dependency map's External Touchpoints table confirms no other channel is defined for this notification |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An access session opens | FEAT-22.SPEC-004 (Support Access Session Logging) | Fires every time Riley opens the household view, including a second or later session for the same request | Household reference, Support Request kind, session start time |
| An access session closes | FEAT-22.SPEC-004 (Support Access Session Logging) | Fires every time Riley closes or navigates away from an open session | Household reference, Support Request kind, session end time |

## Audience and Preferences

**Recipients:** Maya (Organiser) only -- the sole role with Support View access to this household's own record (Access Matrix: Maya View, Sam None, both Jordan rows None, Riley is the subject of the note, not its recipient).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this note has no on/off control | -- | Always on | N/A -- excluded per XBR-14: every support visit must be visible to the organiser without exception; a household must never be able to silence the one signal that bounds an otherwise-invisible access capability |

**Quiet Hours:** N/A -- the product defines quiet hours for time-sensitive reminders (e.g., the nightly dinner nudge), not for a trust-and-audit note whose entire purpose is to reach the organiser promptly; this note is delivered in-app only and has no channel a quiet-hours window would meaningfully hold.

## Content Definition

**In-app (session opened):**
- **Title:** Support viewed your household
- **Body:** {support_visit_reason} on {session_start_date_time}
- **CTA:** View record -- deep-links to FEAT-22.SPEC-003 (Household Support Access Record)

**In-app (session closed):**
- **Title:** Support access ended
- **Body:** {support_visit_reason} -- access ended on {session_end_date_time}
- **CTA:** View record -- deep-links to FEAT-22.SPEC-003 (Household Support Access Record)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {support_visit_reason} | Derived -- Support Request.kind, rendered as "Reviewing a reported safety concern" (kind = safety concern) or "Reviewing a support request" (kind = general support) | Reviewing a reported safety concern | Never empty -- every Support Request has a kind at creation |
| {session_start_date_time} | Support Request.access_record -- the session entry's start timestamp | Sep 27, 2026, 2:14 PM | Never empty -- set by FEAT-22.SPEC-004 at the moment the session opens, before this notification fires |
| {session_end_date_time} | Support Request.access_record -- the session entry's end timestamp | Sep 27, 2026, 2:31 PM | Never empty -- set by FEAT-22.SPEC-004 at the moment the session closes, before this notification fires |

## Delivery Rules

**Batching:** None -- each open and each close is delivered as its own note; sessions are rare enough (per the feature's own Behavioral Context: "rare, on-demand use... never a routine or scheduled interaction") that batching would only delay the visibility this notification exists to provide.
**Deduplication:** Exactly one open note per access-session entry and exactly one close note per access-session entry, since each is fired once by FEAT-22.SPEC-004 at the corresponding step of its processing logic, which itself runs each open and close exactly once per session.
**Retry on failure:** In-app delivery has no retry mechanism of its own: the note is delivered when Maya next opens the product, per standard in-app notification behavior; there is no separate delivery attempt to fail or retry.
**Expiry:** The note itself does not expire in the sense of becoming undeliverable -- it is a record of something that already happened, so it remains available to view until Maya dismisses it; the underlying access-session entry it announces survives indefinitely in FEAT-22.SPEC-003, per SC-18's retention posture.

## Edge Cases

- **The household is deleted between the session and delivery of this note** -- No note is delivered; a deleted household has no organiser left to notify, and FEAT-18's deletion cascade removes the household's data before any pending in-app note could render.
- **Maya has left the organiser role (organiser hand-over, FEAT-09) between the session and delivery** -- The note is delivered to whoever holds the organiser role at the moment of delivery, consistent with Support View being an organiser-role entitlement rather than tied to a specific person.
- **Two sessions open and close in rapid succession for the same household** -- Each open and each close produces its own note in order; they are never merged, since batching is explicitly excluded for this notification.
- **Maya is actively viewing FEAT-22.SPEC-003 when a new session opens** -- The new entry does not auto-insert into that screen mid-view (per FEAT-22.SPEC-003's own edge cases); the in-app note still delivers and is visible the next time Maya checks her notifications, independent of what screen she happens to be on.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-004 (Support Access Session Logging) | Triggered by (inbound) | Both open and close steps fire this notification |
| FEAT-22.SPEC-003 (Household Support Access Record) | Navigation (outbound) | The CTA on both variants deep-links here |

## Analytics and Success Signals

- **support_view_note_delivered** (event: session_opened / session_closed) -- N/A -- no success-metrics.md metric traces to FEAT-22; retained so this trust-facing note's actual delivery is observable
- **support_view_note_cta_tapped** (event: session_opened / session_closed) -- N/A -- no success-metrics.md metric traces to this feature; retained to observe whether organisers actually follow through to the full record

## Acceptance Criteria

**FEAT-22.SPEC-009-AC-01:** Given Riley opens a household's view against a safety-concern request, when the session opens, then Maya receives an in-app note titled "Support viewed your household" with the body "Reviewing a reported safety concern on {session_start_date_time}".

**FEAT-22.SPEC-009-AC-02:** Given Riley opens a household's view against a general-support request, when the session opens, then Maya receives the same note with the body "Reviewing a support request on {session_start_date_time}".

**FEAT-22.SPEC-009-AC-03:** Given an open session closes, when FEAT-22.SPEC-004 records its end, then Maya receives an in-app note titled "Support access ended" with the session's end time.

**FEAT-22.SPEC-009-AC-04:** Given Maya taps the CTA on either note, then she is taken to FEAT-22.SPEC-003 (Household Support Access Record).

**FEAT-22.SPEC-009-AC-05:** Given this notification exists, when Maya looks for a way to turn it off, then no preference control exists anywhere in the product.

**FEAT-22.SPEC-009-AC-06:** Given a session opens at 2:00 AM local time, when the notification fires, then it is delivered in-app immediately, since no quiet-hours window applies to this notification.

**FEAT-22.SPEC-009-AC-07:** Given the household is deleted between the session and delivery, then no note is delivered.

**FEAT-22.SPEC-009-AC-08:** Given the organiser role has been handed over between the session and delivery, when the note delivers, then it reaches whoever currently holds the organiser role.

**FEAT-22.SPEC-009-AC-09:** Given two sessions open and close in rapid succession for the same household, when both complete, then four separate notes are delivered (two opens, two closes), never batched together.

**FEAT-22.SPEC-009-AC-10:** Given Sam or either Jordan role is signed in to the same household, when a session opens or closes, then neither receives this notification, since only Maya holds Support View access to the record.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 2 (open, close) | 2 |
| Preference States | 1 (always on, no control) | 1 |
| Delivery Rules | 4 | 4 |
| Edge Cases | 4 | 4 |
