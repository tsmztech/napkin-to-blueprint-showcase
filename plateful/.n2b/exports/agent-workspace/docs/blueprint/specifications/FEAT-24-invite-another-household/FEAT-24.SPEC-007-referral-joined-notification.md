---
document_type: spec
spec_type: notification
spec_id: FEAT-24.SPEC-007
spec_name: Referral Joined Notification
spec_slug: referral-joined-notification
parent_feature: FEAT-24
parent_feature_name: Invite Another Household
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Notification Spec: Referral Joined Notification

## Overview

**Name:** Referral Joined Notification
**ID:** FEAT-24.SPEC-007
**Type:** Notification
**Purpose:** Tells the inviting member, in-app, when a family they invited finishes setting up its household.
**Parent Feature:** FEAT-24 -- Invite Another Household

## Scope and Non-Goals

**In Scope:**
- The in-app note delivered when a Household Referral record is created for the inviting member's link
- The batched variant when more than one referral completes for the same member on the same day
- Deduplication and delivery behavior for this notification (this notification defines no preference or quiet-hours surface -- see Non-Goals)

**Non-Goals:**
- Deciding whether a referral is eligible to be recorded -- owned by FEAT-24.SPEC-004 (Household Referral Recording) and governed by FEAT-24.SPEC-006 (Household Referral Rules); this notification begins only once a record has already been created
- Email or push delivery of this note -- product-features.md's Communications field for this feature names only an "in-app note," and Member Profile's notification_preferences field (dependency map) enumerates no email or push channel for referral events; adding one would exceed what the product definition establishes
- A dedicated on/off preference toggle for this notification -- Member Profile's notification_preferences field (dependency map) defines only plan-ready and nightly-nudge toggles; product-features.md names no separate control for this note, so none is invented here

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, when a Household Referral record is created | Product-features.md's Communications field states this is delivered as "an in-app note"; the recipient is inside the product whenever they next open it, and the moment carries no urgency that would justify interrupting them through email or push (Feature Breakdown Brief, Non-Functional Notes, Responsiveness: this is not a real-time-critical surface) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household Referral record created | FEAT-24.SPEC-004 (Household Referral Recording) | Fires every time this automation creates a new Household Referral record | The referring_member_link's owning Member Profile (the recipient), the referring Household, and the new household's creation date |

## Audience and Preferences

**Recipients:** The one adult Member Profile that owns the referring_member_link used for this specific referral -- traced to the Household Referrals column of the Access Matrix in user-persona.md, which gives Maya and Sam Full access. Only the member whose own link produced the referral is notified, not every adult member of the household; a household with two members each holding their own link receives this note only for the member whose link was actually used for a given referral.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- no dedicated preference toggle exists for this notification | -- | Always on | -- (product-features.md and the dependency map's Member Profile field list define no such control; see Non-Goals) |

**Quiet Hours:** N/A -- this notification is in-app only, delivered passively the next time the recipient opens the product rather than interrupting them, so no quiet-hours window applies; product-features.md defines quiet hours nowhere for this feature.

## Content Definition

**In-app:**
- **Title:** A family you invited has joined Plateful!
- **Body:** They've set up their own household using your invite link.
- **CTA:** See who's joined -- deep-links to FEAT-24.SPEC-001 (Invite Another Household Screen) for the recipient's own updated joined-families count

**Batched variant (2+ referrals complete for the same recipient on the same day):**
- **In-app title:** {count} families you invited have joined Plateful!
- **In-app body:** They've set up their own households using your invite link.
- **CTA:** See who's joined -- deep-links to FEAT-24.SPEC-001 (Invite Another Household Screen)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {count} | Derived -- number of Household Referral records created for this recipient's referring_member_link on the same calendar day | 3 | Never empty -- the batched variant only renders with 2 or more same-day referrals |

No other placeholders are used: the content names no new household by name, consistent with the dependency map's Household Referral Data Sensitivity note that the record links two households by identity and date only, and that an unauthorized visitor sees only the inviter's first name, never the reverse.

## Delivery Rules

**Batching:** All Household Referral records created for the same recipient's referring_member_link on the same calendar day are delivered as one notification, using the batched variant when 2 or more complete that day. A referral completing on a later day produces its own, separate notification.
**Deduplication:** At most one notification per Household Referral record. FEAT-24.SPEC-004's single-attribution rule (FEAT-24.SPEC-006) already guarantees a given new household produces at most one record, so no record can ever trigger this notification twice.
**Retry on failure:** In-app delivery has no retry: the note is delivered the next time the recipient opens the product, so there is no failure mode analogous to a channel outage to retry against.
**Expiry:** This notification does not expire -- it remains available as an unread in-app note until the recipient views it, since it reports a completed fact (a referral was recorded) rather than a time-sensitive action the recipient must take.

## Edge Cases

- **The Household Referral record's referring_member_link owner leaves the household or is removed before this notification is delivered** -- The notification is still delivered to that same Member Profile if their account still exists and they can still sign in (e.g., they left this household but retain their own account); if the Member Profile itself is removed entirely (FEAT-18, household deletion or member removal), the notification is cancelled silently, since there is no longer a recipient to deliver it to.
- **Two referrals for the same recipient complete on the same day, seconds apart** -- The first referral's notification is held briefly for same-day batching per the Batching rule, and the second referral's completion is folded into the same batched notification rather than producing two separate in-app notes.
- **A referral is recorded, then the referring household is deleted before the recipient opens the app to see the note** -- The notification is still delivered as scheduled; it reports a fact about a completed referral, not a live state of the referring household, so the referring household's own subsequent deletion does not retract it. (This is a rare ordering case, since the recipient is themselves a member of the referring household in every case this feature defines.)
- **The recipient never opens the app** -- The notification has no expiry and remains queued as an unread in-app note indefinitely; since this is the sole channel and there is no fallback channel, delivery is guaranteed only in the sense that it is waiting whenever the recipient next opens the product.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-004 (Household Referral Recording) | Triggered by (inbound) | A newly created Household Referral record fires this notification |
| FEAT-24.SPEC-001 (Invite Another Household Screen) | Navigation (outbound) | The CTA deep-links here, to the recipient's own updated joined-families count |
| FEAT-24.SPEC-006 (Household Referral Rules) | References (inbound) | Single-attribution guarantees this notification is never delivered twice for the same referral |

## Analytics and Success Signals

- **referral_joined_notification_delivered** (batched: yes / no; count) -- supports success-metrics.md: "Household-to-Household Invitation Growth"
- **referral_joined_notification_opened** (batched: yes / no) -- N/A -- no Stage 2 metric measures notification engagement directly for this feature; retained so this notification's own effectiveness at driving members back to FEAT-24.SPEC-001 remains observable

## Acceptance Criteria

**FEAT-24.SPEC-007-AC-01:** Given a Household Referral record is created for Sam's link, when this notification fires, then Sam receives an in-app note titled "A family you invited has joined Plateful!" with the body "They've set up their own household using your invite link."

**FEAT-24.SPEC-007-AC-02:** Given Sam receives this notification, when he taps "See who's joined", then he is taken to FEAT-24.SPEC-001 (Invite Another Household Screen), showing his updated joined-families count.

**FEAT-24.SPEC-007-AC-03:** Given Maya's household has two adult members each with their own link, and only Sam's link produced this referral, when the notification fires, then only Sam receives it -- Maya does not.

**FEAT-24.SPEC-007-AC-04:** Given three referrals complete for Sam's link on the same calendar day, when the notifications are delivered, then Sam receives one batched note titled "3 families you invited have joined Plateful!" rather than three separate notes.

**FEAT-24.SPEC-007-AC-05:** Given a referral completes for Sam today and another completes for him tomorrow, when the notifications are delivered, then today's produces one note and tomorrow's produces its own separate note -- they are never merged across days.

**FEAT-24.SPEC-007-AC-06:** Given Sam has not opened the product since a referral was recorded, when he eventually opens it, then the notification is still there, since it has no expiry.

**FEAT-24.SPEC-007-AC-07:** Given a Household Referral record has already triggered this notification once, when any process re-evaluates that same record, then no second notification is ever produced, since single-attribution guarantees the record itself is created only once.

**FEAT-24.SPEC-007-AC-08:** Given the referring household is deleted after this notification has already been recorded as fired but before Sam opens the app, when Sam eventually opens it, then the notification is still delivered as originally scheduled.

**FEAT-24.SPEC-007-AC-09:** Given Sam's Member Profile is removed entirely before this notification is delivered, when the removal completes, then the notification is cancelled silently, since no recipient remains to deliver it to.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no toggle exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |
