---
document_type: spec
spec_type: notification
spec_id: FEAT-02.SPEC-012
spec_name: Safety Concern Organiser Alert
spec_slug: safety-concern-organiser-alert
parent_feature: FEAT-02
parent_feature_name: Dietary Rules & Allergy Safety Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Notification Spec: Safety Concern Organiser Alert

## Overview

**Name:** Safety Concern Organiser Alert
**ID:** FEAT-02.SPEC-012
**Type:** Notification
**Purpose:** Tells the organiser a meal was removed from the plan because of a safety concern or a rule change, even when she wasn't the reporter.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- Alerting Maya (the organiser) whenever a meal is removed for safety reasons and she was not the one who caused the removal
- Covering both trigger sources: a safety-concern report (FEAT-02.SPEC-004) and a mid-week rule change (FEAT-02.SPEC-003)
- Content, channels, and delivery behavior for this alert

**Non-Goals:**
- Acknowledging the reporter's own submission -- owned by FEAT-02.SPEC-011 (Safety Concern Reporter Acknowledgement), a distinct message to a distinct audience (the reporter, not necessarily the organiser)
- Alerting the operator -- owned by FEAT-02.SPEC-013 (Safety Concern Operator Alert)
- Telling the household the eventual resolution outcome -- owned by FEAT-02.SPEC-014 (Safety Concern Resolution Notice), a later, distinct communication
- Suppressing this alert when Maya herself is the actor -- handled as a trigger condition within this spec (see Trigger), not treated as a separate spec

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always | Maya's day-to-day interaction with the plan is in-app (BRIEF.md, Behavioral Context); an in-app alert is visible the next time she opens the plan, which is her normal rhythm |
| Push | When Maya has enabled plan-related notifications for herself | A safety removal is a same-day disruption to the plan she is responsible for; if she is away from the product, a timely push lets her notice and act (e.g., approve a replacement) sooner than waiting for her next open |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Meal removed by a safety-concern report | FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Fires only when the reporting Member is not Maya | Reported Recipe name, removed meal's night, reporting Member's name |
| Meal(s) removed by a mid-week rule change | FEAT-02.SPEC-003 (Mid-Week Rule Change Re-Check) | Fires whenever one or more dinners are removed by a rule-change re-check (Maya is always the actor who changed the rule, but she still needs to be told which meals were affected, since the removal is a downstream automated consequence she did not directly choose meal-by-meal) | Removed Recipe name(s), affected night(s), the changed Member and rule |

## Audience and Preferences

**Recipients:** Maya (Organiser) only -- per the Access Matrix, Weekly Plan changes are Maya's responsibility (Full), and she is the household member accountable for the plan's overall state.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Push notifications for plan changes | On / Off | On | FEAT-01 (household settings, notification preferences) |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for household members (per the Feature Dependency Map and product-features.md, notification preferences cover on/off per member, not time-of-day windows); a safety removal is treated as timely enough to warrant delivery whenever it occurs rather than being held.

## Content Definition

**In-app:**
- **Title:** A meal was removed for safety
- **Body (safety-concern variant):** {recipe_name} was removed from {night} after {reporter_name} reported a safety concern. We've opened a swap so you can pick a safe alternative.
- **Body (rule-change variant):** {recipe_name} was removed from {night} because it's no longer safe for {member_name} after your latest rule update. We've opened a swap so you can pick a safe alternative.
- **CTA:** Choose a replacement -- deep-links to FEAT-04.SPEC-001 (Meal Swap, safe alternatives list) for the emptied slot on {night}

**Push:**
- **Title:** Plateful: a meal was removed for safety
- **Body:** {recipe_name} was removed from {night}. Tap to choose a safe replacement.
- **CTA:** Tapping the push opens FEAT-04.SPEC-001 (Meal Swap) for the emptied slot

**Batched variant (2+ meals removed by the same mid-week rule change):**
- **In-app title:** {count} meals removed for safety
- **In-app body:** {count} dinners were removed after your latest rule update, including {recipe_name} on {night}. We've opened swaps for each so you can pick safe alternatives.
- **Push title:** Plateful: {count} meals removed for safety
- **Push body:** {count} dinners are no longer safe under your latest rule update. Tap to review.
- **CTA:** Deep-links to the current Weekly Plan (FEAT-03) with the affected nights highlighted

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {recipe_name} | Recipe -- name | Thursday's Mushroom Risotto | Never empty -- name is required at recipe creation |
| {night} | Planned Meal -- night | Thursday | Never empty -- required on every Planned Meal |
| {reporter_name} | Member Profile -- display_name (of the reporting member) | Sam | Never empty -- required on every Member Profile |
| {member_name} | Member Profile -- display_name (of the member whose rule changed) | Jordan | Never empty -- required on every Member Profile |
| {count} | Derived -- number of dinners removed by the same rule-change re-check run | 3 | Never empty -- the batched variant only renders with 2 or more removed |

## Delivery Rules

**Batching:** All dinners removed by the same FEAT-02.SPEC-003 rule-change re-check run are delivered as one alert using the batched variant when 2 or more are removed in that run. A single safety-concern removal (always exactly one meal per report) never batches, since each report is its own event.
**Deduplication:** At most one alert per removal event (one safety-concern report, or one rule-change re-check run). A rule-change re-check that removes zero dinners produces no alert at all, per FEAT-02.SPEC-003's "no dinners affected" outcome.
**Retry on failure:** Push delivery failure is retried up to 3 times over 6 hours. After the final failure, the in-app alert stands as the delivery of record; the removal itself and the opened swap are already visible in-app regardless of push delivery success.
**Expiry:** The in-app alert does not expire -- it remains visible until Maya views the affected slot or dismisses it; the push notification, if undelivered after retries, is not resent, since the in-app state (the emptied slot and open swap) is the surviving signal.

## Edge Cases

- **Maya is the reporting adult herself** -- No alert fires for the safety-concern variant, since she is already aware from her own submission's acknowledgement (FEAT-02.SPEC-011); the rule-change variant still fires, since a rule-change removal is an automated downstream consequence distinct from directly reporting a meal.
- **Maya has disabled push notifications but the safety removal is time-sensitive** -- The in-app alert always fires regardless of the push preference; only the push channel is suppressed, consistent with the preference covering push specifically, not the in-app alert.
- **A rule-change re-check removes meals from two different nights in the same run** -- Both are included in a single batched alert (2+ removed), not two separate alerts, per the batching rule.
- **Maya has two open safety-concern removals from Sam in quick succession** -- Each safety-concern removal produces its own, unbatched alert (batching applies only to the rule-change path, since each safety-concern report is independently and immediately actionable), so Maya receives two separate alerts.
- **The affected Planned Meal slot is filled with a new pick before Maya opens the alert** -- The alert's CTA still deep-links to the slot; if a replacement is already chosen (e.g., by Sam suggesting a pick that Maya later accepted through another path), the CTA opens the slot showing its current state rather than an empty one, and Maya's action there simply confirms or changes what is now there.
- **A rule-change re-check run removes meals for a household where Maya has no push preference set yet (mid-onboarding)** -- The default (On) applies, so push is attempted as it would be for any household with a fully completed setup.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Triggered by (inbound) | A removal where Maya was not the reporter fires this alert |
| FEAT-02.SPEC-003 (Mid-Week Rule Change Re-Check) | Triggered by (inbound) | Any dinner(s) removed by a rule-change re-check fire this alert |
| FEAT-01 (Household Setup & Member Profiles) | References (inbound) | Push preference control lives here |
| FEAT-04.SPEC-001 (Meal Swap alternatives list) | Navigation (outbound) | Single-removal CTA deep-links here |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Navigation (outbound) | Batched-removal CTA deep-links to the plan view with affected nights highlighted |

## Analytics and Success Signals

- **organiser_safety_alert_delivered** (channel: in_app / push; trigger: safety_concern / rule_change; batched: yes/no) -- supports success-metrics.md: "Zero Allergy Incidents"
- **organiser_safety_alert_cta_tapped** (destination: meal_swap / plan_view) -- supports success-metrics.md: "Zero Allergy Incidents"
- **organiser_safety_alert_push_failed** (retry_count) -- N/A -- no Stage 2 metric measures push failure specifically for this alert; retained so silent push loss is observable given the in-app fallback's importance

## Acceptance Criteria

**FEAT-02.SPEC-012-AC-01:** Given Sam reports a safety concern and the meal is removed, when the removal completes, then Maya receives the in-app alert naming the recipe, the night, and Sam as the reporter.

**FEAT-02.SPEC-012-AC-02:** Given Maya has push enabled, when the alert in AC-01 fires, then she also receives a push notification.

**FEAT-02.SPEC-012-AC-03:** Given Maya herself reports a safety concern, when the meal is removed, then no organiser alert fires for that removal, since she is already informed via FEAT-02.SPEC-011.

**FEAT-02.SPEC-012-AC-04:** Given Maya adds a new hard allergy rule that causes two dinners to be removed in the same re-check run, when the removals complete, then Maya receives one batched alert naming both dinners, not two separate alerts.

**FEAT-02.SPEC-012-AC-05:** Given Maya has disabled push notifications, when a safety removal occurs, then she still receives the in-app alert, and no push is sent.

**FEAT-02.SPEC-012-AC-06:** Given Maya taps the CTA on a single-removal alert, when the tap registers, then she is taken to the safe alternatives list (FEAT-04.SPEC-001) for the emptied slot.

**FEAT-02.SPEC-012-AC-07:** Given Maya taps the CTA on a batched alert, when the tap registers, then she is taken to the current Weekly Plan with the affected nights highlighted.

**FEAT-02.SPEC-012-AC-08:** Given a rule-change re-check run removes zero dinners, when the run completes, then no organiser alert is generated.

**FEAT-02.SPEC-012-AC-09:** Given push delivery fails for this alert, when retries are exhausted after 6 hours, then the in-app alert remains the delivery of record and no error is shown to Maya.

**FEAT-02.SPEC-012-AC-10:** Given Sam reports two different meals in quick succession, when both are removed, then Maya receives two separate, unbatched alerts.

**FEAT-02.SPEC-012-AC-11:** Given Maya's household completed setup without an explicit push preference change, when a safety removal occurs, then push is attempted per the default (On) preference.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (in-app, push) | 2 |
| Trigger Paths | 2 | 2 |
| Preference States | 2 (push on, push off) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |
