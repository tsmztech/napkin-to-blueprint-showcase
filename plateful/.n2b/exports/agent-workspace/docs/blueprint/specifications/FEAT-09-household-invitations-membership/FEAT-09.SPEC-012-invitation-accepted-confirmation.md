---
document_type: spec
spec_type: notification
spec_id: FEAT-09.SPEC-012
spec_name: Invitation Accepted Confirmation
spec_slug: invitation-accepted-confirmation
parent_feature: FEAT-09
parent_feature_name: Household Invitations & Membership
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Notification Spec: Invitation Accepted Confirmation

## Overview

**Name:** Invitation Accepted Confirmation
**ID:** FEAT-09.SPEC-012
**Type:** Notification
**Purpose:** Tells the organiser once an invited adult has accepted and joined, so she knows her household is now shared.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- The confirmation delivered to the organiser when an invitation she sent is accepted
- Its single channel (in-app), content, and delivery behavior

**Non-Goals:**
- Delivery to the newly joined member -- their own confirmation is the first-use onboarding experience itself (FEAT-15), not this notification, which is addressed only to the organiser
- Email or push delivery -- this feature carries no Integration spec for transactional email or device-notification delivery (feature-dependency-map.md, External Touchpoints); the invitation itself is a self-shared link (FEAT-09.SPEC-001), and this confirmation is a lightweight in-app signal consistent with that same pattern
- The acceptance processing itself (creating the Member Profile, routing to onboarding) -- owned by FEAT-09.SPEC-007, which triggers this notification only after that processing succeeds

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when an invitation is accepted | Maya (the organiser) is the household's primary in-product user and checks the product regularly for the plan and list; an in-app signal reaches her without adding an email dependency this feature does not otherwise carry |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Invitation acceptance succeeds | FEAT-09.SPEC-007 (Invitation Acceptance Processing) | Fires once, immediately after the new Member Profile is created and the Invitation's status is set to Accepted | New member's display_name, the organiser's Member Profile reference |

## Audience and Preferences

**Recipients:** Maya -- the household's current organiser at the moment of acceptance, per the Access Matrix in user-persona.md. Only the organiser receives this confirmation; no other role is entitled to it, since sending and tracking invitations is Full only for the organiser (Access Matrix, Household Invitations column).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- this confirmation has no dedicated on/off preference | -- | Always on | -- |

There is no preference toggle for this notification: feature-overview.md's Communications field states it plainly as a standing confirmation the organiser receives whenever an invitation is accepted, with no stated opt-out, distinct from the general plan-ready/nightly-nudge preferences owned by FEAT-07/FEAT-13.

**Quiet Hours:** N/A -- this notification is in-app only and reflects a durable state change (a new member has joined) rather than a time-sensitive interruption; there is no quiet-hours concept for an in-app confirmation the organiser sees on her next visit, and the product defines no quiet-hours window for this feature's notifications.

## Content Definition

**In-app:**
- **Title:** {new_member_name} joined your household
- **Body:** {new_member_name} accepted your invitation and now sees the same plan and grocery list.
- **CTA:** View household -- deep-links to FEAT-01.SPEC-010 (Household Settings Hub) for this household

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {new_member_name} | Member Profile -- display_name (the newly created member) | Sam | Never empty -- display_name is required at acceptance (FEAT-09.SPEC-002, FEAT-09.SPEC-010) |

## Delivery Rules

**Batching:** No batching -- each accepted invitation produces exactly one confirmation, since an organiser accepting multiple invitations sends each one deliberately and expects to see each acceptance distinctly.
**Deduplication:** At most one confirmation per accepted Invitation record. FEAT-09.SPEC-007 creates the Member Profile and transitions the Invitation to Accepted exactly once per invitation (XBR-18); this notification fires exactly once per that single transition, never re-fired by a later re-read of the same Invitation.
**Retry on failure:** N/A -- an in-app notification has no separate delivery step to retry; it renders directly from the current state of the household's notification list the next time the organiser opens the product.
**Expiry:** This confirmation does not expire -- it remains visible in the organiser's in-app notification history until she dismisses or reads it, since a record of who joined and when is durable household information, not a time-sensitive alert.

## Edge Cases

- **Organiser's role has been handed over (FEAT-09.SPEC-009) between the invitation being sent and being accepted** -- The confirmation is delivered to whoever is the household's organiser at the moment of acceptance, not to whoever originally sent the invitation; this matches the Recipients definition above (the current organiser).
- **The newly created Member Profile is somehow removed (FEAT-18) moments after acceptance, before the organiser opens the notification** -- The confirmation still renders normally; it reports a fact that occurred (a member joined) and is not retracted by a later, unrelated removal.
- **Organiser is signed in on two devices when the acceptance occurs** -- The confirmation appears in the organiser's in-app notification list on both devices, since it belongs to the household's organiser identity, not to a single device session.
- **Two invitations are accepted in quick succession** -- Each produces its own confirmation with its own new_member_name; they are not merged, per the no-batching rule above.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-007 (Invitation Acceptance Processing) | Triggered by (inbound) | Successful acceptance fires this notification |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (outbound) | The CTA deep-links here |
| FEAT-09.SPEC-001 (Household Invitations Manager) | References (inbound) | The organiser can also see the accepted invitation directly on that screen without opening this notification |

## Analytics and Success Signals

- **invitation_accepted_confirmation_delivered** (-- ) -- supports success-metrics.md: "Household Member Participation" (confirms the organiser is informed of the exact moment her household gains the participating other adult member the metric measures)
- **invitation_accepted_confirmation_opened** (-- ) -- N/A -- no Stage 2 metric distinguishes a delivered confirmation from an opened one; retained as an engagement-diagnostic signal

## Acceptance Criteria

**FEAT-09.SPEC-012-AC-01:** Given Sam's acceptance of Maya's invitation is processed successfully, when FEAT-09.SPEC-007 completes, then Maya receives the in-app notification "Sam joined your household."

**FEAT-09.SPEC-012-AC-02:** Given Maya opens the confirmation, when she taps "View household", then she is taken to FEAT-01.SPEC-010.

**FEAT-09.SPEC-012-AC-03:** Given Maya has handed over the organiser role to a prior recipient before a separate invitation she originally sent is accepted, when that acceptance completes, then the confirmation is delivered to the household's current organiser, not to Maya.

**FEAT-09.SPEC-012-AC-04:** Given two invitations are accepted within moments of each other, when both complete, then Maya receives two separate confirmations, each naming its own new member.

**FEAT-09.SPEC-012-AC-05:** Given Maya is signed in on two devices, when an invitation is accepted, then the confirmation appears in her in-app notification list on both devices.

**FEAT-09.SPEC-012-AC-06:** Given this notification has no preference toggle, when an invitation is accepted, then the confirmation is always delivered regardless of any other notification setting the organiser has configured.

**FEAT-09.SPEC-012-AC-07:** Given the confirmation has been delivered and sits unread in Maya's in-app notification list, when a week passes with no dismissal, then it remains visible, since this notification does not expire.

**FEAT-09.SPEC-012-AC-08:** Given the newly joined member is removed from the household moments after acceptance, when Maya later opens the still-undismissed confirmation, then it still renders normally, reporting the acceptance that did occur.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no toggle) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |
