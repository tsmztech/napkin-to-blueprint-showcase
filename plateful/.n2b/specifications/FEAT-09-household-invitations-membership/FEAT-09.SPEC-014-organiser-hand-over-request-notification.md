---
document_type: spec
spec_type: notification
spec_id: FEAT-09.SPEC-014
spec_name: Organiser Hand-Over Request Notification
spec_slug: organiser-hand-over-request-notification
parent_feature: FEAT-09
parent_feature_name: Household Invitations & Membership
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Notification Spec: Organiser Hand-Over Request Notification

## Overview

**Name:** Organiser Hand-Over Request Notification
**ID:** FEAT-09.SPEC-014
**Type:** Notification
**Purpose:** Tells the chosen adult member that the organiser has asked them to accept the organiser role, so they know to open the request and respond.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- The notification delivered to the selected recipient when the organiser initiates a hand-over request
- Its single channel (in-app), content, and delivery behavior, including what happens if the request is withdrawn before the recipient acts

**Non-Goals:**
- The recipient's accept/decline decision itself -- owned by FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance), which this notification's CTA opens
- Notifying the organiser of the outcome (accept/decline) -- feature-overview.md's Side-Effect Inventory routes a decline back inline into FEAT-09.SPEC-003, and an acceptance's outcome is visible there directly; neither is a separate Notification spec
- Email or push delivery -- consistent with FEAT-09.SPEC-012 and FEAT-09.SPEC-013, this feature carries no Integration spec for transactional email or device-notification delivery (feature-dependency-map.md, External Touchpoints)

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when a hand-over request is sent | Sam (the recipient) is an active household member who uses the product regularly for the shared plan and list; an in-app signal reaches him without adding an email or push dependency this feature does not otherwise carry, and a role hand-over is a considered decision, not an urgent same-minute interruption |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Organiser sends a hand-over request | FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | Fires once, immediately after the request is recorded as outstanding | Current organiser's display_name, recipient's Member Profile reference |

## Audience and Preferences

**Recipients:** Sam -- the specific Active adult member the organiser selected as the hand-over recipient, per the Access Matrix in user-persona.md. Only the addressed recipient receives this notification; no other role sees it, since at most one hand-over request is outstanding per household (XBR-15) and it is addressed to exactly one member.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- this notification has no dedicated on/off preference | -- | Always on | -- |

There is no preference toggle for this notification: a hand-over request is a direct, deliberate action from the organiser to one named recipient, and feature-overview.md's Communications field states plainly that "the new organiser is asked to accept a hand-over," with no stated opt-out.

**Quiet Hours:** N/A -- the product defines no quiet-hours window for this feature's notifications; a hand-over request has no stated urgency window that quiet hours would need to hold against, since the recipient can act on it whenever they next open the product.

## Content Definition

**In-app:**
- **Title:** {organiser_name} wants you to become the organiser
- **Body:** {organiser_name} has asked you to take over as the household's organiser -- managing the weekly budget, schedule, and settings. Open this to accept or decline.
- **CTA:** Respond -- deep-links to FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) for this request

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {organiser_name} | Member Profile -- display_name (the current organiser at the moment the request is sent) | Maya | Never empty -- display_name is a required field for every Member Profile (FEAT-01.SPEC-014) |

## Delivery Rules

**Batching:** No batching -- at most one hand-over request is ever outstanding per household (XBR-15), so this notification never has more than one pending instance to batch.
**Deduplication:** At most one notification per outstanding request. FEAT-09.SPEC-003 creates one pending request per Send Request action; if the organiser cancels and sends a new request (to the same or a different recipient), that is a new request and produces its own new notification -- the prior one is withdrawn per the Edge Cases below.
**Retry on failure:** N/A -- an in-app notification has no separate delivery step to retry; it renders directly from the current state of the recipient's notification list the next time they open the product.
**Expiry:** This notification does not expire on a timer -- unlike an Invitation, a hand-over request has no stated 14-day window (feature-overview.md defines the 14-day expiry only for Invitations, not for hand-over requests). It remains actionable until the organiser cancels it, the recipient responds, or the recipient loses eligibility, at which point the notification is withdrawn per the Edge Cases below.

## Edge Cases

- **Organiser cancels the request before the recipient responds** -- The notification is withdrawn: if still unread, it no longer appears in the recipient's notification list; if already read but not yet acted on, opening its CTA now shows FEAT-09.SPEC-004's Withdrawn state rather than a live decision. A request must never remain actionable after the organiser has cancelled it.
- **Recipient's eligibility is lost while the notification is pending (e.g., they leave the household)** -- The notification is withdrawn along with the request itself (FEAT-09.SPEC-003's own edge case); a departed member is never shown a still-open request to become organiser of a household they no longer belong to.
- **Organiser cancels one request and immediately sends a new one to the same recipient** -- The first notification is withdrawn and a second, distinct notification is delivered for the new request; they are never merged or treated as an update to the same instance.
- **Recipient reads the notification but does not act, and the organiser reaches for the request's own screen (FEAT-09.SPEC-003) meanwhile** -- The organiser sees the request is still Pending; nothing about the notification's read/unread state changes the request's own outstanding status, which is governed entirely by FEAT-09.SPEC-010.
- **Recipient is signed in on two devices when the request is sent** -- The notification appears in the recipient's in-app notification list on both devices, since it belongs to the recipient's identity, not to a single device session.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | Triggered by (inbound) | Send Request fires this notification |
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | References (inbound) | A cancel withdraws this notification |
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | Navigation (outbound) | The CTA deep-links here |
| FEAT-09.SPEC-010 (Invitation & Membership Validation Rules) | References (inbound) | Whether the underlying request remains outstanding |

## Analytics and Success Signals

- **handover_request_notification_delivered** (-- ) -- N/A -- no Stage 2 metric measures hand-over activity; "Household Member Participation" measures a member's joining and use, not the organiser role's transfer
- **handover_request_notification_opened** (-- ) -- N/A -- no Stage 2 metric measures notification engagement for this event

## Acceptance Criteria

**FEAT-09.SPEC-014-AC-01:** Given Maya sends a hand-over request to Sam, when the request is recorded, then Sam receives the in-app notification "Maya wants you to become the organiser."

**FEAT-09.SPEC-014-AC-02:** Given Sam opens the notification, when he taps "Respond", then he is taken to FEAT-09.SPEC-004 with the request shown.

**FEAT-09.SPEC-014-AC-03:** Given Maya cancels the request before Sam has responded, when the cancellation is recorded, then the notification is withdrawn from Sam's unread list, or shows the Withdrawn state if he opens it afterward.

**FEAT-09.SPEC-014-AC-04:** Given Sam leaves the household while the request is pending to him, when his departure is processed, then the notification is withdrawn along with the request.

**FEAT-09.SPEC-014-AC-05:** Given Maya cancels her request to Sam and immediately sends a new request to him, when the new request is recorded, then Sam receives a second, distinct notification, and the first is withdrawn.

**FEAT-09.SPEC-014-AC-06:** Given Sam is signed in on two devices, when the request is sent, then the notification appears in his in-app notification list on both devices.

**FEAT-09.SPEC-014-AC-07:** Given this notification has no preference toggle, when a request is sent, then it is always delivered regardless of any other notification setting Sam has configured.

**FEAT-09.SPEC-014-AC-08:** Given the notification has been delivered and sits unread for several days with no organiser cancellation, when Sam eventually opens it, then it remains actionable, since this notification does not expire on a timer.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no toggle) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
