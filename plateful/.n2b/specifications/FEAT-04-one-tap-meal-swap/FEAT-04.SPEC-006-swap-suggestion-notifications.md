---
document_type: spec
spec_type: notification
spec_id: FEAT-04.SPEC-006
spec_name: Swap Suggestion Notifications
spec_slug: swap-suggestion-notifications
parent_feature: FEAT-04
parent_feature_name: One-Tap Meal Swap
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Notification Spec: Swap Suggestion Notifications

## Overview

**Name:** Swap Suggestion Notifications
**ID:** FEAT-04.SPEC-006
**Type:** Notification
**Purpose:** Tells the organiser when a swap suggestion arrives, and tells the suggesting member the outcome once it is accepted, declined, or lapses.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap

## Scope and Non-Goals

**In Scope:**
- The "suggestion arrived" message to the organiser (Maya)
- The "suggestion accepted," "suggestion declined," and "suggestion lapsed" messages to the suggesting member (Sam)
- Delivery, preference, and expiry behavior for all four message variants

**Non-Goals:**
- Deciding when a suggestion lapses -- owned by FEAT-04.SPEC-005 (Suggestion Lapse); this spec begins where that automation's trigger fires
- Deciding when a swap is applied following an accept -- owned by FEAT-04.SPEC-004 (Apply Meal Swap); this spec only sends the resulting notification
- The nightly "tonight's dinner" nudge and its same-day swap correction -- excluded per feature-dependency-map.md: those belong to FEAT-13 (Tonight's Dinner Reminder), a distinct communication with its own trigger and content
- In-product messaging between household members about a suggestion -- excluded per scope-boundaries.md SC-14: coordination runs through this structured notify/accept/decline/lapse flow, not a chat layer

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, on every variant | Both Maya and Sam use the shared plan inside the product as their primary touchpoint, and the pending state is always visible there regardless of whether the outbound message is seen |
| Push (device notification) | Always attempted, when the recipient's device supports it, via the device-notification-delivery capability (FEAT-07.SPEC-005) | Maya's and Sam's day-to-day interaction happens in short phone sessions away from an open app (user-persona.md, Behavioral Context); a suggestion or its outcome needs to reach them promptly for the "one tap" promise to hold in practice |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Suggestion submitted | FEAT-04.SPEC-002 (Suggest a Swap), via FEAT-04.SPEC-010 | Fires when a new Swap Suggestion is created with outcome Suggested | Suggesting member name, night, proposed recipe |
| Suggestion accepted | FEAT-04.SPEC-004 (Apply Meal Swap) | Fires when an accepted suggestion's swap completes successfully | Suggesting member name, night, applied recipe |
| Suggestion declined | FEAT-04.SPEC-003 (Review Swap Suggestions) | Fires when Maya declines a pending suggestion | Suggesting member name, night, declined recipe |
| Suggestion lapsed | FEAT-04.SPEC-005 (Suggestion Lapse) | Fires when the nightly lapse check marks a suggestion Lapsed | Suggesting member name, night, lapsed recipe |

## Audience and Preferences

**Recipients:** Maya (Organiser) receives the "suggestion arrived" variant, since she alone holds accept/decline authority (Access Matrix, Meal Swap: Full). Sam (Other Adult Member) receives the "accepted," "declined," and "lapsed" variants, since he alone is the suggester whose suggestions can resolve this way (Access Matrix, Meal Swap: Own-only). No other role is entitled to either variant: kids have no Meal Swap access, and Riley's support view is read-only and never receives notifications.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- no dedicated preference exists for this notification | -- | Always on | -- |

This notification carries no independent on/off toggle: the Member Profile's notification_preferences field (per the Feature Dependency Map) defines only plan-ready and nightly-nudge preferences for adults; swap-suggestion messages are not among them. This is a deliberate scope decision, not an oversight -- product-features.md's Communications field describes these messages as the mechanism that carries the suggest-and-approve coordination the product replaces a group chat with (Problem Statement); turning them off would leave a submitted suggestion silently stranded with no other route to the recipient's attention, contradicting XBR-06's requirement that the suggester is always told the outcome.

**Quiet Hours:** N/A -- the product defines quiet hours nowhere in its notification model (no ASMP, XBR, or Communications field establishes one for any notification in Plateful); every notification, including this one, is delivered as soon as its trigger fires.

## Content Definition

**In-app -- Suggestion arrived (to Maya):**
- **Title:** Sam suggested a swap for {night}
- **Body:** {proposed_recipe_name} instead of tonight's plan
- **CTA:** Review -- deep-links to FEAT-04.SPEC-003 (Review Swap Suggestions) for the specific suggestion

**Push -- Suggestion arrived (to Maya):**
- **Title:** Swap suggestion waiting
- **Body:** Sam suggested {proposed_recipe_name} for {night}
- **CTA:** Opens FEAT-04.SPEC-003 (Review Swap Suggestions) for the specific suggestion

**In-app -- Suggestion accepted (to Sam):**
- **Title:** Your suggestion was accepted
- **Body:** {night}'s dinner is now {proposed_recipe_name}
- **CTA:** View plan -- deep-links to FEAT-04.SPEC-002 (Suggest a Swap) for that slot

**Push -- Suggestion accepted (to Sam):**
- **Title:** Maya accepted your swap
- **Body:** {night} is now {proposed_recipe_name}
- **CTA:** Opens FEAT-04.SPEC-002 (Suggest a Swap) for that slot

**In-app -- Suggestion declined (to Sam):**
- **Title:** Your suggestion was declined
- **Body:** {night}'s original dinner stays on the plan
- **CTA:** View plan -- deep-links to FEAT-04.SPEC-002 (Suggest a Swap) for that slot

**Push -- Suggestion declined (to Sam):**
- **Title:** Maya declined your swap
- **Body:** {night}'s dinner is unchanged
- **CTA:** Opens FEAT-04.SPEC-002 (Suggest a Swap) for that slot

**In-app -- Suggestion lapsed (to Sam):**
- **Title:** Your suggestion lapsed
- **Body:** {night} passed before Maya answered -- the original dinner stayed on the plan
- **CTA:** View plan -- deep-links to FEAT-04.SPEC-002 (Suggest a Swap) for that slot

**Push -- Suggestion lapsed (to Sam):**
- **Title:** Swap suggestion lapsed
- **Body:** {night} passed with no answer -- original dinner kept
- **CTA:** Opens FEAT-04.SPEC-002 (Suggest a Swap) for that slot

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {night} | Swap Suggestion -- night | Wednesday | Never empty -- night is required at suggestion creation |
| {proposed_recipe_name} | Swap Suggestion -- proposed_recipe (recipe name) | Sheet-pan salmon | Never empty -- proposed_recipe is required at suggestion creation |

## Delivery Rules

**Batching:** No batching -- each suggestion's arrival and each resolution is delivered as its own notification the moment its trigger fires, matching the "one tap" immediacy the product commits to for swap coordination. Multiple suggestions arriving close together each produce their own separate notification rather than a combined digest.
**Deduplication:** At most one notification per Swap Suggestion per trigger event. A suggestion produces exactly one "arrived" notification (on creation), and exactly one resolution notification (accepted, declined, or lapsed -- these are mutually exclusive terminal outcomes, so only one can ever fire per suggestion). A retried or re-run automation that reaches an already-notified state (e.g., FEAT-04.SPEC-005 processing an already-lapsed suggestion, per its own Edge Cases) does not re-send.
**Retry on failure:** Push delivery failure is retried up to 3 times over 30 minutes. After the final failure, the in-app notification stands as the delivery of record -- the recipient sees it the next time they open the product, and no separate failure message is shown to them.
**Expiry:** These notifications do not expire undelivered: unlike a time-sensitive reminder, the underlying state (a pending suggestion, or its resolved outcome) remains fully visible and actionable inside the product indefinitely, so a delayed push delivery still reaches a recipient with an accurate, current message when it eventually arrives.

## Edge Cases

- **The suggestion resolves (is accepted, declined, or superseded) before the "arrived" push notification is delivered** -- The "arrived" push is cancelled; delivering a stale "review this" message about an already-resolved suggestion would contradict the state Maya would see on opening the app. The in-app pending-suggestion indicator itself is unaffected since it already reflects current state when viewed live.
- **A suggestion is superseded by a direct swap on the same slot (FEAT-04.SPEC-010) rather than declined or accepted by Maya** -- This is treated as the "declined" variant from Sam's perspective content-wise, since the practical outcome (his suggestion did not take effect and the slot changed by another route) matches; the notification text is generated from the suggestion's final outcome value as set by the superseding rule.
- **Sam is removed from the household between suggestion creation and the notification firing** -- The notification is cancelled silently on every channel; a former member is never notified about a household's plan.
- **Both the "accepted" notification and FEAT-13's same-day correction notification (XBR-09) would fire for the same swap** -- These are independent notifications to different concerns (Sam learns his suggestion succeeded; the household is told the nightly nudge is now stale) and both are delivered; neither one's Delivery Rules suppress the other.
- **Push delivery capability is unavailable at trigger time** -- Per ASMP-31, delivery degrades to in-app-only discovery: the in-app notification is still recorded immediately and the recipient sees it on next app open; no email fallback exists for this notification (unlike the plan-ready notification's email fallback in FEAT-07), since this is a same-session coordination message rather than a weekly milestone.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-002 (Suggest a Swap) | Triggered by (inbound) | Suggestion creation fires the "arrived" variant |
| FEAT-04.SPEC-004 (Apply Meal Swap) | Triggered by (inbound) | Successful accept fires the "accepted" variant |
| FEAT-04.SPEC-003 (Review Swap Suggestions) | Triggered by (inbound) | Decline action fires the "declined" variant |
| FEAT-04.SPEC-005 (Suggestion Lapse) | Triggered by (inbound) | Lapse fires the "lapsed" variant |
| FEAT-04.SPEC-003 (Review Swap Suggestions) | Navigation (outbound) | The "arrived" CTA deep-links here |
| FEAT-04.SPEC-002 (Suggest a Swap) | Navigation (outbound) | The "accepted," "declined," and "lapsed" CTAs deep-link here |
| FEAT-07.SPEC-005 (device-notification delivery boundary) | References (outbound) | Owns the push-delivery capability this notification relies on |
| FEAT-13 (Tonight's Dinner Reminder) | References (outbound) | Its own same-day correction notification is independent of this spec's "accepted" variant (XBR-09) |

## Analytics and Success Signals

- **swap_suggestion_arrived_notified** (channel: in_app / push) -- supports success-metrics.md: "Weekly Planning Time"
- **swap_suggestion_outcome_notified** (outcome: accepted / declined / lapsed; channel) -- supports success-metrics.md: "Weekly Planning Time"
- **swap_suggestion_notification_delivery_degraded** (variant; reason: push_unavailable) -- N/A -- no Stage 2 metric measures notification delivery degradation specifically; retained so silent delivery loss on this coordination path is observable rather than invisible.

## Acceptance Criteria

**FEAT-04.SPEC-006-AC-01:** Given Sam submits a suggestion for Thursday's dinner, when the suggestion is created, then Maya receives an in-app notification titled "Sam suggested a swap for Thursday" and, where her device supports it, a push notification titled "Swap suggestion waiting."

**FEAT-04.SPEC-006-AC-02:** Given Maya taps the "arrived" notification's Review CTA, when it opens, then she lands on FEAT-04.SPEC-003 (Review Swap Suggestions) with that suggestion in view.

**FEAT-04.SPEC-006-AC-03:** Given Maya accepts Sam's suggestion, when FEAT-04.SPEC-004 completes successfully, then Sam receives an in-app notification titled "Your suggestion was accepted" and a push notification titled "Maya accepted your swap."

**FEAT-04.SPEC-006-AC-04:** Given Maya declines Sam's suggestion, when the decline is recorded, then Sam receives an in-app notification titled "Your suggestion was declined" and a push notification titled "Maya declined your swap."

**FEAT-04.SPEC-006-AC-05:** Given Sam's suggestion lapses because its night passed unanswered, when FEAT-04.SPEC-005 marks it Lapsed, then Sam receives an in-app notification titled "Your suggestion lapsed" and a push notification titled "Swap suggestion lapsed."

**FEAT-04.SPEC-006-AC-06:** Given push delivery fails three times over 30 minutes, when the final retry fails, then no error is shown to the recipient and the in-app notification stands as the delivery of record.

**FEAT-04.SPEC-006-AC-07:** Given the device-notification-delivery capability is unavailable, when a suggestion is created, then the in-app notification is still recorded immediately and no email fallback is attempted.

**FEAT-04.SPEC-006-AC-08:** Given Maya's suggestion (from Sam) resolves before the "arrived" push is delivered, when the resolution completes first, then the pending "arrived" push is cancelled.

**FEAT-04.SPEC-006-AC-09:** Given three suggestions arrive for Maya within the same minute, when each is created, then each produces its own separate notification -- no batched digest is sent.

**FEAT-04.SPEC-006-AC-10:** Given a suggestion is superseded by Maya's direct swap on the same slot rather than explicitly declined, when the superseding rule sets its outcome, then Sam receives the "declined"-style notification reflecting that outcome.

**FEAT-04.SPEC-006-AC-11:** Given this notification exists with no on/off preference, when Sam or Maya looks for a way to turn it off, then no such control exists in the product, consistent with the notification's always-on design.

**FEAT-04.SPEC-006-AC-12:** Given Sam is removed from the household after suggesting a swap but before the resulting notification fires, when the trigger fires, then the notification is cancelled silently on every channel.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (in-app, push) | 2 |
| Trigger Paths | 4 (arrived, accepted, declined, lapsed) | 4 |
| Preference States | 1 (always on -- no toggle exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
