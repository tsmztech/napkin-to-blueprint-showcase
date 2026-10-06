---
document_type: spec
spec_type: notification
spec_id: FEAT-13.SPEC-004
spec_name: Same-Day Swap Correction Message
spec_slug: same-day-swap-correction-message
parent_feature: FEAT-13
parent_feature_name: Tonight's Dinner Reminder
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Notification Spec: Same-Day Swap Correction Message

## Overview

**Name:** Same-Day Swap Correction Message
**ID:** FEAT-13.SPEC-004
**Type:** Notification
**Purpose:** Delivers a brief follow-up naming tonight's new dinner when a same-day swap changes it after the original nudge already went out, so no household member cooks the wrong thing.
**Parent Feature:** FEAT-13 -- Tonight's Dinner Reminder

## Scope and Non-Goals

**In Scope:**
- The correction delivered when FEAT-13.SPEC-003 decides a same-day swap owes one
- Both surfaces this correction can appear on: push (device notification) and the same in-app "Tonight" card FEAT-13.SPEC-002 already shows, refreshed to the new dinner
- Content, placeholders, and delivery rules for this correction

**Non-Goals:**
- Deciding whether a correction is owed, to whom, and enforcing the at-most-one-correction-per-day cap -- owned by FEAT-13.SPEC-003 (Same-Day Swap Correction Trigger) and FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules); this spec begins once that decision has already been made.
- Deriving the new prep-reminder text -- owned by FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule); this spec only renders that rule's output for the post-swap recipe.
- The original nudge this corrects -- owned by FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message); this spec is a distinct, separate message, not an edit to that one's already-delivered content.
- Repeated corrections for further same-day changes -- excluded per the feature's own Validation & Limits (at most one follow-up correction per household per day); a second same-day swap after a correction has already been sent produces no further message (FEAT-13.SPEC-003).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Push (device notification) | When the recipient's channel, resolved fresh at correction-dispatch time via FEAT-13.SPEC-005, is device notification | The correction exists precisely so a member who already saw the original "Tonight: ..." push is not left cooking the wrong thing; reaching them the same way is the most direct correction |
| In-app "Tonight" card | When the recipient's resolved channel is not device notification | The same card FEAT-13.SPEC-002 already shows is refreshed to the new dinner rather than a second card being added, so the member simply sees the current, correct dinner whenever they next open the plan |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Same-day swap correction fires | FEAT-13.SPEC-003 (Same-Day Swap Correction Trigger) | Fires when that automation's "Correction dispatched" outcome completes, once per recipient | Member id, new meal_name (via the post-swap Planned Meal.recipe), new prep_reminder_text or its confirmed absence (FEAT-13.SPEC-006) |

## Audience and Preferences

**Recipients:** Exactly the members who received today's original nudge (FEAT-13.SPEC-002), as resolved by FEAT-13.SPEC-003 via FEAT-13.SPEC-005 -- never a freshly re-evaluated eligible list. Neither kid row nor Riley is ever included, for the same reasons as the original nudge (Access Matrix: Notification Prefs None); an unauthorized visitor receives nothing.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Nightly nudge | On / Off | On | FEAT-01.SPEC-005 (Member Profile Detail) -- this spec has no preference control of its own; receiving today's original nudge is what puts a member in scope for its correction |

**Quiet Hours:** N/A -- like the original nudge, this correction is a single immediate follow-up sent right after a same-day swap completes, not a scheduled window; the product defines no quiet-hours window for either message.

## Content Definition

**Push (device notification):**
- **Title (prep step present):** Dinner update: {meal_name} — {prep_reminder_text}
- **Title (no prep step):** Dinner update: {meal_name}
- **Body:** -- none; the same single-line discipline as the original nudge
- **CTA:** Open tonight's dinner -- deep-links to FEAT-03.SPEC-001 (Weekly Plan View) or FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)), whichever holds the household's current Weekly Plan

**In-app "Tonight" card:**
- **Behavior:** The existing "Tonight" card (FEAT-13.SPEC-002) is refreshed in place to the new {meal_name} and {prep_reminder_text} -- this spec never adds a second card. A member who has not opened the app since the swap simply sees the current, correct dinner the next time they do.
- **CTA:** Tap the card -- opens tonight's dinner within the same screen (FEAT-03.SPEC-001 or FEAT-23.SPEC-001)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {meal_name} | Recipe -- name, via the post-swap Planned Meal.recipe | slow-cooker chili | Never empty -- Recipe.name is required and Planned Meal.recipe is required, so a dispatched correction always has a new meal name |
| {prep_reminder_text} | Derived -- FEAT-13.SPEC-006's output from the new Recipe.prep_requirements | -- (no prep needed) | When no prep is required, this placeholder and its preceding " — " are omitted entirely; the title reads "Dinner update: {meal_name}" alone |

## Delivery Rules

**Batching:** N/A -- FEAT-13.SPEC-005's at-most-one-correction-per-household-per-day cap guarantees at most one correction instance per member per day, so no same-day duplicate correction instances ever exist to batch.
**Deduplication:** The at-most-one-correction cap, enforced by FEAT-13.SPEC-003 before this spec is invoked, guarantees no member ever receives two corrections the same day even if multiple swaps occur.
**Retry on failure:** Identical to FEAT-13.SPEC-002 -- delivered through FEAT-07.SPEC-005 with its own queuing and redelivery; this spec defines no separate retry. Failure is silent and non-blocking, with the refreshed "Tonight" card as the surviving signal.
**Expiry:** A correction not delivered by the end of tonight (local time) expires undelivered, for the same reason as the original nudge -- a stale correction about a dinner from a previous night would confuse rather than help. The in-app card remains accurate regardless, since it reflects the Planned Meal's current state whenever the member next opens it.

## Edge Cases

- **The swap is itself reversed by a second same-day swap before the correction is delivered** -- Per FEAT-13.SPEC-003's at-most-one-correction cap, only the first eligible swap's correction can ever be attempted; if it has not yet been delivered when the reversal happens, it still delivers naming whichever recipe was current when FEAT-13.SPEC-003 read it, and the in-app card (always reflecting current state) is the accurate fallback in the moment of any lag.
- **A member's nightly-nudge preference is turned off between the original nudge and the correction** -- The correction still reaches them: eligibility for the correction is "received today's original nudge," a state already fixed at nudge time, not re-evaluated against the current preference; disabling the preference going forward affects only future nudges and corrections, not the one already owed for tonight.
- **The underlying Planned Meal is removed (safety concern) between the swap and the correction's delivery** -- The correction is cancelled silently; a correction about a dinner no longer on the plan is never delivered, the same disposition as the original nudge's own removed-Planned-Meal edge case.
- **Quiet hours colliding with expiry** -- Cannot occur, since this notification defines no quiet-hours window (Audience and Preferences).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-13.SPEC-003 (Same-Day Swap Correction Trigger) | Triggered by (inbound) | Its "Correction dispatched" outcome fires this notification |
| FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules) | References (inbound) | Correction audience, cap, and channel resolution govern who receives this and where |
| FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule) | References (inbound) | Supplies the prep_reminder_text content for the new recipe |
| FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message) | References (inbound) | The original nudge this corrects, and the shared "Tonight" card it refreshes rather than duplicates |
| FEAT-07.SPEC-005 (Device-Notification Delivery Integration) | Triggers (outbound) | Carries out push delivery, offline queuing, and redelivery |
| FEAT-03.SPEC-001 (Weekly Plan View) | Navigation (outbound) | CTA destination and the surface for the refreshed "Tonight" card |
| FEAT-23.SPEC-001 (Weekly Plan (Manual Week Builder)) | Navigation (outbound) | CTA destination and the surface for the refreshed "Tonight" card |

## Analytics and Success Signals

- **dinner_nudge_correction_delivered** (channel: push / in_app_card) -- N/A -- no metric in success-metrics.md names Tonight's Dinner Reminder as its Connected Feature; retained per product-features.md's own Signals field (dinner_nudge_correction_sent) so correction delivery is observable per member and channel.
- **dinner_nudge_correction_opened** (channel) -- N/A -- same reason; observes whether the correction actually reaches the member before dinner.

## Acceptance Criteria

**FEAT-13.SPEC-004-AC-01:** Given Maya received today's original nudge for "20-minute pasta" and Sam swaps tonight's dinner to "slow-cooker chili," when FEAT-13.SPEC-003 dispatches the correction, then Maya receives a push titled "Dinner update: slow-cooker chili" (with any new prep step included).

**FEAT-13.SPEC-004-AC-02:** Given the new recipe has no early-prep requirement, when the correction is dispatched, then its title reads "Dinner update: {meal_name}" alone, with no invented prep step.

**FEAT-13.SPEC-004-AC-03:** Given Sam's channel is device notification, when his correction is dispatched, then he receives a push and no duplicate "Tonight" card is added.

**FEAT-13.SPEC-004-AC-04:** Given Maya has no working device-notification channel, when her correction is dispatched, then her existing "Tonight" card is refreshed in place to the new dinner rather than a second card appearing.

**FEAT-13.SPEC-004-AC-05:** Given Sam turns his nightly-nudge preference off after receiving today's original nudge but before a same-day swap, when the correction is dispatched, then he still receives it, since eligibility for the correction was already fixed at the time he received the original nudge.

**FEAT-13.SPEC-004-AC-06:** Given the underlying Planned Meal is removed by a safety-concern report between the swap and delivery, when delivery would otherwise occur, then no correction is delivered to anyone.

**FEAT-13.SPEC-004-AC-07:** Given a second same-day swap reverses tonight's dinner before the first correction is delivered, when delivery proceeds, then it names whichever recipe was current at the moment FEAT-13.SPEC-003 read it, and the refreshed "Tonight" card shows the accurate current dinner regardless of any lag.

**FEAT-13.SPEC-004-AC-08:** Given a correction is not delivered by midnight, when the expiry cutoff passes, then it is not delivered the next morning, and the "Tonight" card (reflecting the Planned Meal's current state) remains the accurate signal.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (push, in-app card) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 2 (received original nudge; preference later turned off) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |
