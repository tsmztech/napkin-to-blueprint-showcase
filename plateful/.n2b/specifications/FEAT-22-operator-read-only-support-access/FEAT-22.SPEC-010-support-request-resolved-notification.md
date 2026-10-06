---
document_type: spec
spec_type: notification
spec_id: FEAT-22.SPEC-010
spec_name: Support Request Resolved Notification
spec_slug: support-request-resolved-notification
parent_feature: FEAT-22
parent_feature_name: Operator Read-Only Support Access
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Notification Spec: Support Request Resolved Notification

## Overview

**Name:** Support Request Resolved Notification
**ID:** FEAT-22.SPEC-010
**Type:** Notification
**Purpose:** Tells Maya when her general-support Support Request is resolved, so a submitted request never goes silent.
**Parent Feature:** FEAT-22 -- Operator Read-Only Support Access

## Scope and Non-Goals

**In Scope:**
- The in-app note delivered when Riley resolves a general-support Support Request

**Non-Goals:**
- Safety-concern resolutions -- excluded per FEAT-22.SPEC-005's own scope decision: FEAT-02.SPEC-014 (Safety Concern Resolution Notice) already tells the household the safety outcome once FEAT-22.SPEC-005 forwards it to FEAT-02.SPEC-005; a second, separate note here for the same event would duplicate and risk contradicting that outcome-specific message, so this notification fires only for general-support kind requests
- The acknowledgement sent when a general-support request is first submitted -- owned by FEAT-18.SPEC-015 (Support Request Acknowledgement), a distinct earlier communication this notification does not repeat
- Any household-data content about what was discussed or resolved -- excluded per scope-boundaries.md SC-14: coordination happens through named features, not a messaging layer, so this note confirms only that the request is resolved, never a description of the resolution's substance
- Delivery to Riley -- Riley is the one resolving the request, not a recipient of its resolution notice

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, on resolution | This feature inventories no Integration spec and the dependency map's External Touchpoints table assigns FEAT-22 no email or push capability; a general support contact is itself a low-frequency, non-urgent interaction, and an in-app confirmation matches the product's existing pattern of confirming Support Request outcomes in-product (FEAT-18.SPEC-005's own same-screen confirmation on submission) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A general-support Support Request is resolved | FEAT-22.SPEC-005 (Support Request Resolution) | Fires only when the resolved request's kind is general support contact | Household reference, the resolved Support Request's note (the household's original description) |

## Audience and Preferences

**Recipients:** Maya (Organiser) -- the raised_by member on a general-support Support Request is the adult who submitted it (Maya or Sam, per FEAT-18.SPEC-005's Access), but this notification's recipient is always the organiser, consistent with FEAT-22.SPEC-009's audience and with Maya's Support View access being the household's own visibility into support outcomes; Sam's own submission is covered by his same-screen confirmation from FEAT-18.SPEC-005 and this in-app note additionally reaches Maya as the accountable organiser.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this note has no on/off control | -- | Always on | N/A -- excluded per the same reasoning as FEAT-22.SPEC-009: a submitted request's outcome must always reach the organiser, since there is no other product surface that tells her a general support request was resolved |

**Quiet Hours:** N/A -- the product's quiet-hours behavior governs time-sensitive reminders, not a one-time resolution confirmation with no urgency to hold back; this note is in-app only, so no channel exists for quiet hours to meaningfully affect.

## Content Definition

**In-app:**
- **Title:** Your support request is resolved
- **Body:** We've resolved your support request: "{request_note_summary}"
- **CTA:** View record -- deep-links to FEAT-22.SPEC-003 (Household Support Access Record)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {request_note_summary} | Support Request.note, truncated to the first 100 characters if longer | "The grocery list stopped updating after a swap" | The body renders as "We've resolved your support request." with no quoted text, since FEAT-18.SPEC-010 permits an empty note only when validation allows it; in practice a general-support note is required at submission (FEAT-18.SPEC-005), so this fallback covers the case where the note has since been cleared by a data-management action rather than an ordinary submission gap |

## Delivery Rules

**Batching:** None -- each resolution is delivered as its own note; general-support requests are infrequent enough that batching would only delay a resolution the household is specifically waiting to hear about.
**Deduplication:** Exactly one note per Support Request, since FEAT-22.SPEC-008 (Support Request Status Transition Rules) allows a request to reach Resolved exactly once, and FEAT-22.SPEC-005 triggers this notification exactly once at that transition.
**Retry on failure:** In-app delivery has no retry of its own: the note is delivered when Maya next opens the product, per standard in-app notification behavior.
**Expiry:** The note does not expire -- it remains available to view until Maya dismisses it, and the resolution itself remains visible afterward through the request's Resolved status wherever it is shown (FEAT-01.SPEC-010, FEAT-22.SPEC-003).

## Edge Cases

- **The household is deleted between resolution and delivery** -- No note is delivered; a deleted household has no organiser left to notify.
- **The organiser role changes hands between resolution and delivery** -- The note is delivered to whoever currently holds the organiser role, consistent with this being an organiser-role entitlement rather than tied to a specific person.
- **A safety-concern request is resolved** -- This notification never fires; FEAT-02.SPEC-014 is the sole household-facing resolution message for that kind, per this spec's own Non-Goals.
- **The request's note was longer than 100 characters** -- The body shows the first 100 characters with no truncation indicator beyond the quoted text itself reading as a summary, consistent with this being a confirmation rather than a full replay of the original submission.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-005 (Support Request Resolution) | Triggered by (inbound) | Resolving a general-support request fires this notification |
| FEAT-22.SPEC-003 (Household Support Access Record) | Navigation (outbound) | The CTA deep-links here |
| FEAT-18.SPEC-005 (Contact Support) | References (inbound, cross-feature) | The originating screen for the general-support request this notification resolves |
| FEAT-18.SPEC-015 (Support Request Acknowledgement) | References (inbound, cross-feature) | The earlier, submission-time communication this notification does not repeat |

## Analytics and Success Signals

- **support_request_resolved_note_delivered** -- N/A -- no success-metrics.md metric traces to FEAT-22; retained so this trust-facing note's actual delivery is observable
- **support_request_resolved_note_cta_tapped** -- N/A -- no success-metrics.md metric traces to this feature; retained to observe whether organisers follow through to the full record

## Acceptance Criteria

**FEAT-22.SPEC-010-AC-01:** Given Riley resolves a general-support Support Request whose note reads "The grocery list stopped updating after a swap", when resolution completes, then Maya receives the in-app note "We've resolved your support request: \"The grocery list stopped updating after a swap\"".

**FEAT-22.SPEC-010-AC-02:** Given Maya taps the CTA on this note, then she is taken to FEAT-22.SPEC-003 (Household Support Access Record).

**FEAT-22.SPEC-010-AC-03:** Given Riley resolves a safety-concern request instead, when resolution completes, then this notification does not fire.

**FEAT-22.SPEC-010-AC-04:** Given Sam submitted the original general-support request, when it is resolved, then Maya (the organiser) still receives this note, in addition to whatever same-screen confirmation Sam saw at submission.

**FEAT-22.SPEC-010-AC-05:** Given this notification exists, when Maya looks for a way to turn it off, then no preference control exists anywhere in the product.

**FEAT-22.SPEC-010-AC-06:** Given a request is resolved at 3:00 AM local time, when the notification fires, then it is delivered in-app immediately, since no quiet-hours window applies.

**FEAT-22.SPEC-010-AC-07:** Given the household is deleted between resolution and delivery, then no note is delivered.

**FEAT-22.SPEC-010-AC-08:** Given the request's note is longer than 100 characters, when the note renders, then only the first 100 characters are quoted in the body.

**FEAT-22.SPEC-010-AC-09:** Given the organiser role changes hands between resolution and delivery, when the note delivers, then it reaches whoever currently holds the organiser role.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no control) | 1 |
| Delivery Rules | 4 | 4 |
| Edge Cases | 4 | 4 |
