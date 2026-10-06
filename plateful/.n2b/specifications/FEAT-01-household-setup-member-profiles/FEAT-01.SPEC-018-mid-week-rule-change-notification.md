---
document_type: spec
spec_type: notification
spec_id: FEAT-01.SPEC-018
spec_name: Mid-Week Rule Change Notification
spec_slug: mid-week-rule-change-notification
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Notification Spec: Mid-Week Rule Change Notification

## Overview

**Name:** Mid-Week Rule Change Notification
**ID:** FEAT-01.SPEC-018
**Type:** Notification
**Purpose:** Tells the organiser which plan meal was removed after a mid-week hard dietary-rule change, so she is never left wondering why a night's dinner disappeared.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- The notification sent when FEAT-01.SPEC-012's re-check removes one or more meals from the current week's plan
- The single-removal and multiple-removal (batched) content variants
- Preference, retry, and expiry behavior for this notification

**Non-Goals:**
- Deciding which meals are removed -- owned by FEAT-01.SPEC-012 (Mid-Week Hard-Rule Change Trigger) and FEAT-02 (Dietary Rules & Allergy Safety Engine); this spec begins where that decision's outcome is handed to it
- Offering a replacement for the removed meal -- owned by One-Tap Meal Swap (FEAT-04), which the notification's call to action deep-links into
- Any other safety-related notification (e.g., a safety-concern report's acknowledgement, owned by FEAT-02.SPEC-010) -- this spec covers only the mid-week rule-change removal named in this feature's own Communications field

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when a removal occurs | Maya's plan lives in-app; the removal is a change to something she can act on immediately (swap in a replacement), so it must appear where that action happens |
| Email | When the organiser's plan-ready email fallback preference indicates she is not reliably reached by device notification (mirroring FEAT-07's channel logic) | A safety-driven plan change is significant enough that it should not depend solely on Maya having the app open; the same fallback logic already established for plan-ready notifications (FEAT-07) applies here for consistency |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Meal(s) removed by mid-week re-check | FEAT-01.SPEC-012 (Mid-Week Hard-Rule Change Trigger) | Fires whenever the re-check removes one or more Planned Meals from the current week's plan | Household reference, the removed meal(s)' night and recipe name, the member and rule that triggered the change |

## Audience and Preferences

**Recipients:** Maya (Organiser) -- the sole recipient, per the Access Matrix in user-persona.md: this feature's Household Setup and Weekly Plan access is Full for the organiser and View for Sam, and the Communications field states specifically "the organiser is also told," not the whole household. Sam is not a recipient of this notification, since plan-change communications for other adults are scoped elsewhere (e.g., swap-suggestion outcomes in FEAT-04).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Plan-ready notifications (shared toggle also governing this notification's in-app/email channel choice) | On / Off | On | FEAT-01.SPEC-005 (Member Profile Detail), FEAT-07 |

**Quiet Hours:** N/A -- a safety-driven plan removal is delivered immediately regardless of quiet hours, since leaving the organiser unaware that a meal disappeared from her plan for longer than necessary works against the product's zero-incident safety commitment; this is a deliberate exception to the quiet-hours behavior that governs less time-sensitive notifications like FEAT-07's plan-ready message.

## Content Definition

**In-app (single meal removed):**
- **Title:** A meal was removed from your plan
- **Body:** {night}'s {recipe_name} was removed because of {member_name}'s updated {rule_label}.
- **CTA:** Choose a replacement -- deep-links to FEAT-04 (One-Tap Meal Swap) for the open {night} slot

**Email (single meal removed):**
- **Subject:** Your plan changed: {recipe_name} removed
- **Body:**
  Hi {organiser_first_name},

  We removed {recipe_name} from {night} because it's no longer safe for {member_name}'s updated {rule_label}.

  Open your plan to choose a safe replacement for that night.
- **CTA (button):** Choose a replacement -- deep-links to FEAT-04 (One-Tap Meal Swap) for the open {night} slot

**In-app (batched, 2+ meals removed):**
- **Title:** {count} meals were removed from your plan
- **Body:** {night_list} were removed because of {member_name}'s updated {rule_label}.
- **CTA:** Review your plan -- deep-links to FEAT-03 (AI Weekly Dinner Plan Generation) or FEAT-23 (Manual Weekly Planning), whichever produced the current plan, showing the affected week

**Email (batched, 2+ meals removed):**
- **Subject:** Your plan changed: {count} meals removed
- **Body:**
  Hi {organiser_first_name},

  We removed {count} meals from this week's plan because they're no longer safe for {member_name}'s updated {rule_label}: {night_list}.

  Open your plan to choose safe replacements.
- **CTA (button):** Review your plan -- deep-links to FEAT-03 or FEAT-23, showing the affected week

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {night} | Planned Meal -- night | Thursday | Never empty -- night is required on every Planned Meal |
| {recipe_name} | Recipe -- name | Spinach and Feta Pasta | Never empty -- name is required on every Recipe |
| {member_name} | Member Profile -- display_name | Jordan | Never empty -- display_name is required on every Member Profile |
| {rule_label} | Dietary Rule -- rule_kind and allergen (or religious-rule label) | peanut allergy | "dietary rule" (a generic label, used only if the specific rule's label cannot be resolved at send time) |
| {organiser_first_name} | Member Profile (organiser) -- display_name | Maya | "there" (a neutral greeting substitute) |
| {count} | Derived -- number of meals removed by this re-check | 3 | Never empty -- the batched variant only renders with 2 or more removed meals |
| {night_list} | Derived -- comma-separated list of removed nights | Wednesday and Thursday | Never empty -- the batched variant only renders with 2 or more removed meals |

## Delivery Rules

**Batching:** All meals removed by a single mid-week re-check run (FEAT-01.SPEC-012) are delivered as one notification, using the batched variant when 2 or more meals are removed by that run. Meals removed by a later, separate rule change (e.g., a second hard rule added the next day) generate their own, separate notification -- removals from different re-check runs are never merged into one message.
**Deduplication:** At most one notification per re-check run. A re-check that removes zero meals never generates a notification. If the same meal is somehow flagged by two rules evaluated in the same re-check run (e.g., it violates both a new allergy and a tightened religious rule), it is named once in the single resulting notification, not twice.
**Retry on failure:** Email delivery failure is retried up to 3 times over 6 hours, mirroring FEAT-07's plan-ready email fallback pattern; after the final failure, the in-app notification stands as the delivery of record, and no alarming failure message is shown to the organiser. In-app delivery has no retry: it is delivered when the organiser next opens the product, and the underlying plan change is visible in-app regardless of whether the notification itself was seen.
**Expiry:** This notification does not expire in the sense of becoming irrelevant -- the plan change it describes is permanent (no restore path, per this feature's Entity-Lifecycle Coverage Matrix for Dietary Rule and Planned Meal), so the notification remains meaningful and deliverable whenever it is eventually seen. There is no cutoff after which it is withheld.

## Edge Cases

- **The removed meal's recipe is itself later removed from the library entirely (FEAT-10, unrelated event)** -- The notification's content is generated at the moment of removal and does not re-resolve {recipe_name} later, so an already-sent or already-queued notification is unaffected by the recipe's later removal from the library.
- **The organiser turns off plan-ready notifications between the re-check firing and delivery** -- Per XBR-13, each member controls their own notification preferences; if Maya turns this shared preference off before delivery, the pending notification is still delivered, since a safety-driven plan removal is exempt from the ordinary preference-timing rule that governs less time-sensitive notifications, given the zero-incident safety commitment this notification serves.
- **Quiet hours would otherwise apply** -- Not applicable, per the Quiet Hours section above: this notification is exempt and always delivers immediately.
- **The organiser's account is removed or the household is deleted (FEAT-18) between the re-check and delivery** -- The notification is cancelled silently on every channel; a household that no longer exists has no plan to review.
- **Two hard-rule changes trigger two separate re-check runs within moments of each other, each removing a different meal** -- Each run generates its own notification per the Deduplication rule; the organiser receives two separate notifications rather than one merged one, since they originate from two distinct re-check runs.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-012 (Mid-Week Hard-Rule Change Trigger) | Triggered by (inbound) | A re-check that removes one or more meals fires this notification |
| FEAT-01.SPEC-005 (Member Profile Detail) | References (inbound) | Plan-ready preference toggle governs this notification's delivery channel |
| FEAT-07 (Weekly Plan Ready Notification) | References (inbound) | Shares the same preference and email-fallback channel logic |
| FEAT-04 (One-Tap Meal Swap) | Navigation (outbound) | Single-meal CTA deep-links here |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Navigation (outbound) | Batched-meal CTA deep-links here for a generated plan |
| FEAT-23 (Manual Weekly Planning) | Navigation (outbound) | Batched-meal CTA deep-links here for a manually built plan |

## Analytics and Success Signals

- **midweek_rule_change_notification_delivered** (channel: in_app / email; batched: yes / no; removed_meal_count) -- supports success-metrics.md: "Zero Allergy Incidents" (confirms the organiser is reliably told every time a safety-driven removal occurs, which is the transparency half of the zero-incident promise).
- **midweek_rule_change_notification_opened** (channel) -- N/A -- no distinct Stage 2 metric measures notification open rate for this specific message; retained to distinguish delivery from the organiser actually seeing it.
- **midweek_rule_change_cta_tapped** (destination: swap / plan_review) -- supports success-metrics.md: "Zero Allergy Incidents" (measures whether the organiser closes the loop by choosing a safe replacement after being told).

## Acceptance Criteria

**FEAT-01.SPEC-018-AC-01:** Given a mid-week re-check (FEAT-01.SPEC-012) removes one meal from Maya's plan, when the removal completes, then Maya receives an in-app notification titled "A meal was removed from your plan" naming the removed recipe, night, member, and rule.

**FEAT-01.SPEC-018-AC-02:** Given Maya's plan-ready preference indicates email fallback applies, when the removal notification fires, then she also receives an email with the subject "Your plan changed: {recipe_name} removed".

**FEAT-01.SPEC-018-AC-03:** Given a mid-week re-check removes three meals in a single run, when the notification fires, then Maya receives one batched notification titled "3 meals were removed from your plan" -- not three separate notifications.

**FEAT-01.SPEC-018-AC-04:** Given Maya taps the CTA on a single-meal removal notification, when she taps "Choose a replacement", then she lands on FEAT-04 (One-Tap Meal Swap) for the affected night's open slot.

**FEAT-01.SPEC-018-AC-05:** Given Maya has turned off plan-ready notifications, when a mid-week removal occurs, then she still receives this notification, since it is exempt from the ordinary preference-timing rule given its safety significance.

**FEAT-01.SPEC-018-AC-06:** Given a mid-week removal occurs during Maya's quiet hours (if any are configured elsewhere in the product), when the notification fires, then it is delivered immediately rather than held.

**FEAT-01.SPEC-018-AC-07:** Given the email channel fails to deliver after 3 retries over 6 hours, when the final retry fails, then the in-app notification stands as the delivery of record and no failure message is shown to Maya.

**FEAT-01.SPEC-018-AC-08:** Given Maya's household is deleted (FEAT-18) between the re-check and delivery, when the notification would otherwise send, then it is cancelled silently on every channel.

**FEAT-01.SPEC-018-AC-09:** Given a single meal is flagged by two rules evaluated in the same re-check run, when the notification fires, then the meal is named once, not twice.

**FEAT-01.SPEC-018-AC-10:** Given two separate hard-rule changes trigger two distinct re-check runs within moments of each other, each removing a different meal, when both complete, then Maya receives two separate notifications, one per run.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (in-app, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 3 (default on, off, email fallback) | 3 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
