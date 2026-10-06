---
document_type: spec
spec_type: notification
spec_id: FEAT-09.SPEC-013
spec_name: Member Left Household Notification
spec_slug: member-left-household-notification
parent_feature: FEAT-09
parent_feature_name: Household Invitations & Membership
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 7
---

# Notification Spec: Member Left Household Notification

## Overview

**Name:** Member Left Household Notification
**ID:** FEAT-09.SPEC-013
**Type:** Notification
**Purpose:** Tells the organiser when an other adult member leaves on their own, so she understands why the household's membership changed.
**Parent Feature:** FEAT-09 -- Household Invitations & Membership

## Scope and Non-Goals

**In Scope:**
- The notification delivered to the organiser when an Other Adult Member completes leaving the household on their own
- Its single channel (in-app), content, and delivery behavior

**Non-Goals:**
- Delivery to the leaving member -- their own confirmation is the in-screen success message on FEAT-09.SPEC-005 itself, not this notification, which is addressed only to the organiser
- Email or push delivery -- consistent with FEAT-09.SPEC-012, this feature carries no Integration spec for transactional email or device-notification delivery (feature-dependency-map.md, External Touchpoints)
- Notifying the organiser when a member is removed by her own action (FEAT-18) -- that is the organiser's own action and needs no notification to herself; this spec covers only a member leaving on their own initiative
- The departure processing itself (anonymising ratings, removing the profile) -- owned by FEAT-09.SPEC-008, which triggers this notification only after that processing succeeds

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when a member's self-initiated departure is processed | Maya (the organiser) needs to understand why her household's membership changed, and an in-app signal reaches her without adding an email dependency this feature does not otherwise carry |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Member departure processing succeeds | FEAT-09.SPEC-008 (Member Departure Processing) | Fires once, immediately after the leaving member's ratings are anonymised and their Member Profile status is set to Left | Leaving member's display_name (as it was before status changed) |

## Audience and Preferences

**Recipients:** Maya -- the household's current organiser at the moment of departure, per the Access Matrix in user-persona.md. Only the organiser receives this notification; no other role is entitled to it, since household membership changes are Full-visibility only for the organiser.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- this notification has no dedicated on/off preference | -- | Always on | -- |

There is no preference toggle for this notification: feature-overview.md's Communications field states plainly that "the organiser is also told when a member leaves," with no stated opt-out, distinct from the general plan-ready/nightly-nudge preferences owned by FEAT-07/FEAT-13.

**Quiet Hours:** N/A -- this notification is in-app only and reflects a durable state change (a member has left) rather than a time-sensitive interruption; there is no quiet-hours concept for an in-app notification the organiser sees on her next visit, and the product defines no quiet-hours window for this feature's notifications.

## Content Definition

**In-app:**
- **Title:** {member_name} left your household
- **Body:** {member_name} has left. Their household access has ended, and their past ratings continue to quietly influence future meal choices.
- **CTA:** View household -- deep-links to FEAT-01.SPEC-010 (Household Settings Hub) for this household

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {member_name} | Member Profile -- display_name, captured at the moment of departure before the record's status changes | Sam | Never empty -- display_name is a required field for every Member Profile (FEAT-01.SPEC-014) |

## Delivery Rules

**Batching:** No batching -- each departure produces exactly one notification, since each departure is a distinct household event the organiser should see individually.
**Deduplication:** At most one notification per departure. FEAT-09.SPEC-008 sets a member's status to Left exactly once per member (its own idempotency rule prevents a repeat run from re-anonymising or re-transitioning an already-Left profile); this notification fires exactly once per that single transition.
**Retry on failure:** N/A -- an in-app notification has no separate delivery step to retry; it renders directly from the current state of the household's notification list the next time the organiser opens the product.
**Expiry:** This notification does not expire -- it remains visible in the organiser's in-app notification history until she dismisses or reads it, since a record of who left and when is durable household information, not a time-sensitive alert.

## Edge Cases

- **Organiser's role has been handed over between when the leaving member started their confirmation and when departure processing completes** -- The notification is delivered to whoever is the household's organiser at the moment processing completes, not to whoever was organiser earlier in the session; this matches the Recipients definition above (the current organiser).
- **Organiser is signed in on two devices when the departure is processed** -- The notification appears in the organiser's in-app notification list on both devices, since it belongs to the household's organiser identity, not to a single device session.
- **The leaving member is later re-invited and rejoins as a genuinely new Member Profile (XBR-18)** -- This notification's original record is unaffected; it remains a historical account of the earlier departure and is never retroactively updated or removed by a later, unrelated re-invitation.
- **Two members leave in quick succession (e.g., after a household disagreement)** -- Each produces its own notification with its own member_name; they are not merged, per the no-batching rule above.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-008 (Member Departure Processing) | Triggered by (inbound) | Successful departure processing fires this notification |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (outbound) | The CTA deep-links here |

## Analytics and Success Signals

- **member_left_notification_delivered** (-- ) -- N/A -- "Household Member Participation" measures a household gaining and keeping an engaged other adult member; this notification reports the inverse event (a departure) and is not itself a contribution to that target, so it is retained only for organiser-communication completeness, not cited as a metric contributor
- **member_left_notification_opened** (-- ) -- N/A -- no Stage 2 metric measures notification engagement for this event

## Acceptance Criteria

**FEAT-09.SPEC-013-AC-01:** Given Sam's departure is processed successfully by FEAT-09.SPEC-008, when processing completes, then Maya receives the in-app notification "Sam left your household."

**FEAT-09.SPEC-013-AC-02:** Given Maya opens the notification, when she taps "View household", then she is taken to FEAT-01.SPEC-010.

**FEAT-09.SPEC-013-AC-03:** Given Maya has handed over the organiser role before a member's departure that was already in progress completes, when departure processing finishes, then the notification is delivered to the household's current organiser, not to Maya.

**FEAT-09.SPEC-013-AC-04:** Given Maya is signed in on two devices, when a member's departure is processed, then the notification appears in her in-app notification list on both devices.

**FEAT-09.SPEC-013-AC-05:** Given this notification has no preference toggle, when a member leaves, then the notification is always delivered regardless of any other notification setting the organiser has configured.

**FEAT-09.SPEC-013-AC-06:** Given the notification has been delivered and sits unread, when a week passes with no dismissal, then it remains visible, since this notification does not expire.

**FEAT-09.SPEC-013-AC-07:** Given the departed member is later re-invited and rejoins as a new Member Profile, when Maya later views the original notification, then it still shows the earlier departure unchanged, unaffected by the later re-invitation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no toggle) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |
