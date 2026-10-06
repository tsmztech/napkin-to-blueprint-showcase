---
document_type: spec
spec_type: integration
spec_id: FEAT-07.SPEC-005
spec_name: Device-Notification Delivery Integration
spec_slug: device-notification-delivery-integration
parent_feature: FEAT-07
parent_feature_name: Weekly Plan Ready Notification
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Integration Spec: Device-Notification Delivery Integration

## Overview

**Name:** Device-Notification Delivery Integration
**ID:** FEAT-07.SPEC-005
**Type:** Integration
**Purpose:** The product delivers the plan-ready message to a member's device through an external device-notification delivery capability, including queuing and redelivery when that device is offline.
**Parent Feature:** FEAT-07 -- Weekly Plan Ready Notification

## Scope and Non-Goals

**In Scope:**
- Dispatching the plan-ready message's Push variant to an eligible member's device
- Offline queuing and automatic redelivery once a device reconnects
- User-facing behavior when this capability is slow, unavailable, or reports a delivery problem
- Disclosure of what data is shared with this capability

**Non-Goals:**
- Choosing the device-notification vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate.
- Deciding which members are eligible or which channel they resolve to -- owned by FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules); this spec only carries out delivery once a member has already been resolved to the Push channel.
- The exact notification content -- owned by FEAT-07.SPEC-002 (Plan-Ready Notification Message); this spec transports that content, it does not author it.
- Native mobile push infrastructure or app-store-distributed push -- excluded per scope-boundaries.md SC-05: the platform is a responsive web app for v1 with no native apps, so this capability is scoped to what a web app can deliver.
- The swap-suggestion, nightly-nudge, and manual-planning-pick notifications that also rely on this same capability -- those are owned by One-Tap Meal Swap (FEAT-04), Tonight's Dinner Reminder (FEAT-13), and Manual Weekly Planning (FEAT-23) respectively; this spec is the shared delivery boundary those features' own notifications reference, per the Feature Dependency Map's External Touchpoints table, but this spec's own Product Behaviors Enabled and Data Exchanged sections describe only this feature's plan-ready use of it.

## Capability Category

**Category:** Device-notification delivery
**Dependency Source:** ASMP-31 -- "Device-notification delivery capability -- Required for the weekly 'plan ready' notification, the daily 'tonight's dinner' nudge, and swap-suggestion alerts; without it, these degrade to email (plan ready) or in-app-only discovery, weakening the product's proactive rhythm." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Device-notification delivery (ASMP-31)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-07, FEAT-13, FEAT-04, FEAT-23; this spec is the shared delivery-boundary Integration spec)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| An eligible household member receives "next week's plan is ready" as a device notification | Notify when the plan is ready | FEAT-07.SPEC-002 (Plan-Ready Notification Message) |
| A plan-ready notification sent while a member's device is offline still reaches them once they reconnect | Notify when the plan is ready | FEAT-07.SPEC-002 (Plan-Ready Notification Message) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Notification content (exact title and body text) | FEAT-07.SPEC-002's content templates -- no household or entity fields beyond the fixed text | Each dispatch to a member resolved to the Push channel | The capability needs the exact text to display on the device |
| Recipient device/channel reference | The capability's own registration for that member's device, established when the member's device previously granted permission -- not a Member Profile field the product stores or exposes further | Each dispatch | Routes the notification to the correct device |

Household plan contents, meal names, budget figures, dietary rules, and every Member Profile field beyond the destination device reference never leave the product through this capability.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Delivery outcome (delivered / undelivered -- device unregistered or permission revoked) | The capability reports the result of a dispatch attempt | No dependency-map entity is updated; the outcome feeds only this spec's own delivery-tracking, since a household's Weekly Plan visibility never depends on it (XBR-12) |
| Device-reconnected signal | The capability reports that a previously offline device has come back online | No dependency-map entity is updated; the signal triggers redelivery of any notification still queued for that device |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Delivery confirmed | The capability confirms the notification reached the device | None | None beyond the notification itself now being visible to the member | FEAT-07.SPEC-002 |
| Delivery failed (device unreachable at all -- unregistered or permission revoked) | The capability reports it cannot reach the device by any means, not merely that it is temporarily offline | None -- FEAT-07.SPEC-003's channel resolution for that member is not retroactively changed for this week's already-attempted dispatch | No user-facing error; the plan remains available in-app immediately regardless (XBR-12) | FEAT-07.SPEC-002 |
| Device reconnected | A device with a queued plan-ready notification comes back online | The queued notification is delivered | The member sees the notification as if newly arrived, at the moment they reconnect | FEAT-07.SPEC-002 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | The dispatch attempt is queued and delivered once the capability responds; the household's plan remains fully and immediately visible in-app throughout, unaffected (XBR-12). No delay message is ever shown, since this is a background delivery path with no in-app waiting screen of its own. | No push notification is dispatched for the affected member(s) this week; FEAT-07.SPEC-003's channel-fallback rule treats them as "device notifications unavailable," and Email (FEAT-07.SPEC-006) delivers the message to them instead. | N/A -- this capability accepts any well-formed dispatch of fixed notification text to a registered device; it has no concept of rejecting a plan-ready dispatch the way a payment or content-review capability might reject a request. |

## Consent and Disclosure

- **Device-notification permission** -- Before any Push message can ever reach a member, that member's own device or browser prompts them, through the platform's standard permission mechanism, to allow notifications. The product does not layer a separate in-product disclosure on top of that device-level prompt, since the permission decision is already the member's own, made at the device level. Declining simply means the member is treated as "device notifications unavailable" and receives the Email fallback instead (FEAT-07.SPEC-003).
- **What is shared** -- Only the fixed notification text (FEAT-07.SPEC-002's content templates) and the destination device reference cross this boundary. No meal, budget, dietary, or other household content is ever included, a boundary each adult can see restated wherever this feature's preference is set (FEAT-01.SPEC-005).

## Edge Cases

- **The same delivery-confirmed event is delivered twice for one dispatch** -- No user-visible duplicate results, since the notification itself was sent once; a second delivery-confirmed report changes nothing.
- **A delivery-failed event arrives for a member already removed from the household** -- The event is discarded on arrival; a former member's device is never notified regardless of when the capability's report arrives.
- **A device reconnects after its queued notification has already expired per FEAT-07.SPEC-002's Delivery Rules** -- No delivery occurs on that stale reconnect; expiry takes precedence over a late reconnect signal.
- **The capability goes down mid-dispatch, after some household members' pushes were already confirmed but before others'** -- Each member's outcome is independent; already-confirmed deliveries are unaffected, and the remaining undelivered members fall back to Email for that week.
- **Two reconnect events for the same device arrive out of order or are both delivered** -- The queued notification is delivered at most once regardless of how many reconnect signals arrive, per FEAT-07.SPEC-002's deduplication guarantee.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | Triggered by (inbound) | Hands off Push-channel dispatches to this integration |
| FEAT-07.SPEC-002 (Plan-Ready Notification Message) | Affects (outbound) | Delivery, queuing, and reconnect outcomes govern what that spec's Delivery Rules describe |
| FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules) | References (inbound) | Reads this integration's availability to resolve each member's channel |
| FEAT-04.SPEC-006 (Swap Suggestion Notifications) | References (outbound) | Shares this same delivery boundary for its own Push variant |
| FEAT-13 (Tonight's Dinner Reminder) | References (outbound) | Shares this same delivery boundary for its nightly nudge and same-day correction |
| FEAT-23 (Manual Weekly Planning) | References (outbound) | Shares this same delivery boundary for its pick-suggestion alerts |

## Analytics and Success Signals

- **device_notification_dispatched** (household id, generation_kind: recurring / first_plan) -- supports success-metrics.md: "Weekly Plan Ready Notification Reach"
- **device_notification_delivery_confirmed** (latency_bucket: under_1_min / 1_to_5_min / over_5_min) -- supports success-metrics.md: "Weekly Plan Ready Notification Reach" (the metric's within-one-minute delivery target)
- **device_notification_capability_down** (affected_household_count) -- N/A -- no Stage 2 metric measures capability outage duration or breadth directly; retained to observe how often the email-fallback path is exercised system-wide.

## Acceptance Criteria

**FEAT-07.SPEC-005-AC-01:** Given Maya is resolved to the Push channel for this week's message, when the dispatch is sent and her device is online, then the notification is delivered and confirmed.

**FEAT-07.SPEC-005-AC-02:** Given Sam's device is offline when his Push dispatch is sent, when his device reconnects later, then the queued notification is delivered at that point.

**FEAT-07.SPEC-005-AC-03:** Given a device reports it is unregistered when a dispatch is attempted, when that report is received, then FEAT-07.SPEC-002 shows no error to the household and the plan remains visible in-app.

**FEAT-07.SPEC-005-AC-04:** Given the capability is slow to respond, when a dispatch is attempted, then the attempt is queued and the household's plan remains fully visible in-app throughout, with no in-app delay indicator for this background path.

**FEAT-07.SPEC-005-AC-05:** Given the capability is down for the entire dispatch window, when eligible Push-resolved members are dispatched, then they receive the Email variant instead, per FEAT-07.SPEC-003's fallback rule.

**FEAT-07.SPEC-005-AC-06:** Given Maya has never received a Push notification from Plateful before, when the capability first attempts to reach her device, then her device's own permission prompt governs whether it can, with no separate in-product disclosure required.

**FEAT-07.SPEC-005-AC-07:** Given Maya declines her device's notification permission, when this week's dispatch runs, then she is treated as "device notifications unavailable" and receives the Email fallback.

**FEAT-07.SPEC-005-AC-08:** Given a delivery-confirmed event for Maya's dispatch is delivered twice, when the second copy arrives, then nothing changes and no duplicate notification appears to her.

**FEAT-07.SPEC-005-AC-09:** Given Sam is removed from the household after his dispatch was queued but before a delivery-failed event for it arrives, when that event arrives, then it is discarded and no action is taken on his behalf.

**FEAT-07.SPEC-005-AC-10:** Given Sam's device reconnects only after his queued notification has expired per FEAT-07.SPEC-002's Delivery Rules, when the reconnect signal arrives, then no delivery occurs.

**FEAT-07.SPEC-005-AC-11:** Given the capability goes down partway through dispatching to a household with two eligible Push-resolved members, when one member's push was already confirmed, then that confirmation stands and only the remaining member falls back to Email.

**FEAT-07.SPEC-005-AC-12:** Given only the plan-ready message's fixed text and a device reference are ever sent to this capability, when any dispatch occurs, then no meal, budget, or dietary data is included in what leaves the product.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 2 | 2 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 2 (slow, down; reject cell is N/A and excluded) | 2 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |
