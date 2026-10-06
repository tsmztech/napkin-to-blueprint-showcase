---
document_type: spec
spec_type: notification
spec_id: FEAT-13.SPEC-002
spec_name: Tonight's Dinner Nudge Message
spec_slug: tonights-dinner-nudge-message
parent_feature: FEAT-13
parent_feature_name: Tonight's Dinner Reminder
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Notification Spec: Tonight's Dinner Nudge Message

## Overview

**Name:** Tonight's Dinner Nudge Message
**ID:** FEAT-13.SPEC-002
**Type:** Notification
**Purpose:** Tells each eligible household member what's for dinner tonight and any prep step it needs, by device notification with an in-app "Tonight" card fallback.
**Parent Feature:** FEAT-13 -- Tonight's Dinner Reminder

## Scope and Non-Goals

**In Scope:**
- The nudge delivered once per household per day when FEAT-13.SPEC-001 dispatches it
- Both surfaces this nudge can appear on: push (device notification) and the in-app "Tonight" card fallback
- Content, placeholders, and delivery rules (deduplication, retry, expiry) for this nudge

**Non-Goals:**
- Deciding whether tonight's nudge fires at all, who is eligible, and which channel each member resolves to -- owned by FEAT-13.SPEC-001 (Tonight's Nudge Trigger) and FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules); this spec begins once a dispatch has already been decided.
- Deriving the prep-reminder text -- owned by FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule); this spec only renders that rule's output.
- The follow-up sent when a same-day swap changes tonight's dinner after this nudge already went out -- excluded per the feature's own Communications field, which treats the correction as a distinct message; owned by FEAT-13.SPEC-004 (Same-Day Swap Correction Message).
- Using email as a channel for this nudge -- excluded per the Feature Dependency Map's External Touchpoints note, which states directly that "email is deliberately NOT used for the nudge -- the fallback is the in-app 'Tonight' card," keeping email reserved for the weekly plan-ready message and account messages.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Push (device notification) | When the member's resolved channel (FEAT-13.SPEC-005) is device notification | Maya's and Sam's day includes stretches away from the app before dinner starts; a push reaches them at the moment they need to act (start cooking, take something out of the freezer), matching the Brief's own "Tonight: ..." example |
| In-app "Tonight" card | When the member's resolved channel is not device notification (no working push channel for that member) | The Brief and the Feature Dependency Map deliberately exclude email as this nudge's fallback; a persistent card at the top of the plan (FEAT-03.SPEC-001, FEAT-23.SPEC-001) still tells that member what's cooking the moment they open the app |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nudge dispatch for an eligible member | FEAT-13.SPEC-001 (Tonight's Nudge Trigger) | Fires when that automation's "Nudge dispatched" outcome completes, once per eligible member | Member id, meal_name (tonight's Planned Meal's recipe name), prep_reminder_text or its confirmed absence (FEAT-13.SPEC-006) |

## Audience and Preferences

**Recipients:** Maya and Sam -- only when Active and their own notification_preferences.nightly_nudge is on (FEAT-13.SPEC-005). Neither kid row (Access Matrix: Notification Prefs None for both the young-kid, no-login row and the older-kid, Later-phase login row) ever receives it; Riley (Operator, support) has no Notification Prefs access and never receives it; an unauthorized visitor receives nothing, since this is a background message with no screen of its own to reach.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Nightly nudge | On / Off | On | FEAT-01.SPEC-005 (Member Profile Detail), opened from FEAT-01.SPEC-010 (Household Settings Hub) |

**Quiet Hours:** N/A -- the nudge fires once per day at a single platform-set pre-dinner time (platform parameter: `nightly-nudge-send-time`) already chosen to fall within normal waking hours before dinner; the product defines no separate quiet-hours window for this notification.

## Content Definition

**Push (device notification):**
- **Title (prep step present):** Tonight: {meal_name} — {prep_reminder_text}
- **Title (no prep step):** Tonight: {meal_name}
- **Body:** -- none; the title carries the complete message for this brief nudge, matching the Brief's own example ("Tonight: 20-minute pasta — take the chicken out of the freezer.")
- **CTA:** Open tonight's dinner -- deep-links to FEAT-03.SPEC-001 (Weekly Plan View) or FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)), whichever produced the household's current Weekly Plan, at tonight's slot

**In-app "Tonight" card:**
- **Title (prep step present):** Tonight: {meal_name} — {prep_reminder_text}
- **Title (no prep step):** Tonight: {meal_name}
- **Body:** -- none; the same single-line content as the push title, shown at the top of the plan
- **CTA:** Tap the card -- opens tonight's dinner within the same screen (FEAT-03.SPEC-001 or FEAT-23.SPEC-001); no separate navigation destination, since the card already sits on that screen

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {meal_name} | Recipe -- name, via tonight's Planned Meal.recipe | 20-minute pasta | Never empty -- Recipe.name is required (Feature Dependency Map, Recipe fields) and Planned Meal.recipe is required, so a dispatched nudge always has a meal name |
| {prep_reminder_text} | Derived -- FEAT-13.SPEC-006's output from Recipe.prep_requirements | take the chicken out of the freezer | When no prep is required, this placeholder and its preceding " — " are omitted entirely; the title reads "Tonight: {meal_name}" alone, per FEAT-13.SPEC-006's no-invented-prep-step disposition |

## Delivery Rules

**Batching:** N/A -- FEAT-13.SPEC-005's once-per-household-per-day firing cap guarantees at most one Planned-Meal-triggered instance per member per day, so no same-day duplicate instances ever exist to collapse into a batch.
**Deduplication:** FEAT-13.SPEC-005's once-per-household-per-day firing cap guarantees at most one dispatch attempt per member per day; a duplicate trigger for the same day is suppressed inside FEAT-13.SPEC-001 before this spec is ever invoked.
**Retry on failure:** Push delivery failure and offline queuing/redelivery are handled entirely by FEAT-07.SPEC-005 (Device-Notification Delivery Integration), the shared delivery boundary this notification travels on; this spec defines no retry of its own. When that capability reports the device unreachable by any means, no further attempt is made and no error is shown -- the in-app "Tonight" card (already the surface for that member per FEAT-13.SPEC-005's channel resolution, or simply the plan itself if the member is not otherwise eligible) stands as the surviving signal.
**Expiry:** A push instance not delivered by the end of tonight (local time) expires undelivered rather than arriving the next morning about last night's dinner, which would confuse the household. The in-app card is not itself time-limited by this rule -- it is simply superseded once tomorrow's Planned Meal becomes "tonight" the next day.

## Edge Cases

- **Planned Meal removed (safety concern) between trigger and delivery** -- The nudge is cancelled silently on every surface; a nudge about a dinner no longer on the plan is never delivered, matching FEAT-13.SPEC-001's "no dinner planned tonight" disposition for the underlying trigger cancellation.
- **Nightly-nudge preference turned off between trigger and delivery** -- Per FEAT-13.SPEC-005, eligibility is evaluated at dispatch time inside FEAT-13.SPEC-001, not at this spec's delivery step, so a member already dispatched to is not retroactively un-sent; a member who disables the preference before FEAT-13.SPEC-001's dispatch step runs is excluded there instead, before this spec ever fires for them.
- **Quiet hours colliding with expiry** -- Cannot occur: this notification defines no quiet-hours window (Audience and Preferences), so there is no hold to collide with the nightly expiry cutoff.
- **Device notifications unavailable for a member who does have their preference on** -- The in-app card is the delivery of record for that member for that day; no push is attempted or retried beyond FEAT-07.SPEC-005's own handling, and no error of any kind is shown.
- **A same-day swap changes tonight's dinner after this nudge was already delivered** -- This spec's already-delivered content is never edited or re-sent; FEAT-13.SPEC-004 (Same-Day Swap Correction Message) delivers a separate, brief follow-up instead (XBR-09), rather than this spec attempting to update its own past delivery.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-13.SPEC-001 (Tonight's Nudge Trigger) | Triggered by (inbound) | Its "Nudge dispatched" outcome fires this notification |
| FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules) | References (inbound) | Eligibility, cap, and channel resolution govern who receives this and on which surface |
| FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule) | References (inbound) | Supplies the prep_reminder_text content |
| FEAT-01.SPEC-005 (Member Profile Detail) | References (inbound) | Where each adult sets their own nightly-nudge preference |
| FEAT-01.SPEC-010 (Household Settings Hub) | References (inbound) | Entry point into the preference toggle |
| FEAT-07.SPEC-005 (Device-Notification Delivery Integration) | Triggers (outbound) | Carries out push delivery, offline queuing, and redelivery |
| FEAT-03.SPEC-001 (Weekly Plan View) | Navigation (outbound) | CTA destination and the surface for the "Tonight" card fallback |
| FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)) | Navigation (outbound) | CTA destination and the surface for the "Tonight" card fallback |
| FEAT-13.SPEC-004 (Same-Day Swap Correction Message) | References (outbound) | The separate follow-up that handles a same-day swap, rather than this spec re-sending |

## Analytics and Success Signals

- **dinner_nudge_delivered** (channel: push / in_app_card) -- N/A -- no metric in success-metrics.md names Tonight's Dinner Reminder as its Connected Feature; retained per product-features.md's own Signals field (dinner_nudge_sent) so delivery reach is observable per member and channel even without a Stage 2 metric attached.
- **dinner_nudge_opened** (channel) -- N/A -- same reason; retained per product-features.md's Signals field (dinner_nudge_opened).
- **dinner_nudge_cta_tapped** (channel; destination: FEAT-03.SPEC-001 / FEAT-23.SPEC-001) -- N/A -- same reason; this is the closest observable signal this feature can emit toward whether nudged dinners actually get cooked, which the Reported Food Waste and Spend Reduction metric (Connected Feature: Weekly Waste & Spend Check-In) ultimately depends on but does not measure through this feature's own events.
- **dinner_nudge_disabled** (member id) -- N/A -- same reason; retained per product-features.md's Signals field (dinner_nudge_disabled), observed here as the state a member is in when this notification's audience is resolved, even though the toggle interaction itself is FEAT-01.SPEC-005's screen action.

## Acceptance Criteria

**FEAT-13.SPEC-002-AC-01:** Given Maya has device notifications available and her nightly nudge on, and tonight's dinner is "20-minute pasta" with a prep requirement of "take the chicken out of the freezer," when FEAT-13.SPEC-001 dispatches her nudge, then she receives a push titled "Tonight: 20-minute pasta — take the chicken out of the freezer."

**FEAT-13.SPEC-002-AC-02:** Given Sam's tonight's dinner has no early-prep requirement, when his nudge is dispatched, then he receives a push titled "Tonight: {meal_name}" with no trailing prep text and no invented prep step.

**FEAT-13.SPEC-002-AC-03:** Given Maya taps her push nudge, when the tap is registered, then she lands on tonight's dinner within FEAT-03.SPEC-001 or FEAT-23.SPEC-001, whichever holds the household's current Weekly Plan.

**FEAT-13.SPEC-002-AC-04:** Given Sam has no working device-notification channel and his nightly nudge is on, when his nudge is dispatched, then he receives no push and instead sees the "Tonight" card at the top of the plan the next time he opens it.

**FEAT-13.SPEC-002-AC-05:** Given Maya has turned her nightly nudge off, when tonight's dispatch runs, then she receives no push and no "Tonight" card.

**FEAT-13.SPEC-002-AC-06:** Given a device reports it cannot be reached at all when Sam's push is attempted, when that report arrives, then no error is shown anywhere and the "Tonight" card remains his surviving signal for tonight's dinner.

**FEAT-13.SPEC-002-AC-07:** Given Maya's push for tonight's dinner has not been delivered by midnight, when the expiry cutoff passes, then the push is not delivered the next morning, and the plan itself (not a stale push) is what she sees.

**FEAT-13.SPEC-002-AC-08:** Given tonight's dinner is swapped after Maya's nudge already arrived, when the swap completes, then this spec's already-delivered nudge is left unchanged and FEAT-13.SPEC-004 delivers a separate correction instead.

**FEAT-13.SPEC-002-AC-09:** Given the underlying Planned Meal is removed by a safety-concern report between dispatch and delivery, when delivery would otherwise occur, then no nudge is delivered on any surface.

**FEAT-13.SPEC-002-AC-10:** Given both Maya and Sam turn their nightly nudge off before FEAT-13.SPEC-001's dispatch step runs, when tonight's evaluation completes, then neither receives a push nor a "Tonight" card, and both simply see tonight's dinner the next time they open the plan.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (push, in-app card) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 3 (on with push, on without push, off) | 3 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
