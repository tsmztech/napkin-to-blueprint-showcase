---
document_type: spec
spec_type: automation
spec_id: FEAT-13.SPEC-003
spec_name: Same-Day Swap Correction Trigger
spec_slug: same-day-swap-correction-trigger
parent_feature: FEAT-13
parent_feature_name: Tonight's Dinner Reminder
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Same-Day Swap Correction Trigger

## Overview

**Name:** Same-Day Swap Correction Trigger
**ID:** FEAT-13.SPEC-003
**Type:** Automation
**Purpose:** When a same-day swap changes tonight's dinner after the original nudge was already sent, fires at most one follow-up correction naming the new dinner.
**Parent Feature:** FEAT-13 -- Tonight's Dinner Reminder

## Scope and Non-Goals

**In Scope:**
- Detecting that a completed swap affects tonight's slot, after today's original nudge (FEAT-13.SPEC-001) has already been dispatched to at least one member
- Enforcing the at-most-one-correction-per-household-per-day cap
- Resolving the correction audience to exactly the members who received today's original nudge
- Dispatching the correction message (FEAT-13.SPEC-004) to that audience

**Non-Goals:**
- Applying the swap itself, re-checking safety, or updating the Planned Meal's recipe -- owned entirely by FEAT-04.SPEC-004 (Apply Meal Swap); this automation only reacts to that automation's completed outcome.
- Defining the correction audience, the once-per-day correction cap, and the channel-fallback logic in detail -- owned by FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules), which this automation calls rather than re-implementing.
- Deriving the new prep-reminder text -- owned by FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule); this automation only reads its output for the post-swap recipe.
- The exact correction content and delivery mechanics -- owned by FEAT-13.SPEC-004 (Same-Day Swap Correction Message); this automation only decides that a correction is owed, to whom, and hands off the dispatch instruction.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A same-day swap completes for tonight's slot | FEAT-04.SPEC-004 (Apply Meal Swap) | Fires on that automation's "Swap applied (direct)" or "Swap applied (accepted suggestion)" outcome, for a Planned Meal whose night is today (XBR-09) | Household id, Planned Meal reference, new recipe, previous recipe, swap completion time |

## Processing Logic

1. Receive the swap-applied signal from FEAT-04.SPEC-004, carrying the affected Planned Meal reference and its night.
2. Check whether the swapped Planned Meal's night is today. If not, stop -- no-action outcome; a swap for a future night has no bearing on tonight's nudge.
3. Check via FEAT-13.SPEC-005 whether today's original nudge (FEAT-13.SPEC-001) was ever dispatched to at least one member for this household -- read the dispatched-members record FEAT-13.SPEC-001 wrote for today. If no original nudge was ever dispatched today, stop -- no-action outcome; there is nothing to correct.
4. Check the at-most-one-correction-per-household-per-day cap via FEAT-13.SPEC-005 -- has a correction already been recorded for this household today? If yes, stop -- no-action outcome.
5. Read the correction audience via FEAT-13.SPEC-005: exactly the members recorded as having received today's original nudge -- never a freshly re-evaluated eligible list.
6. Read the prep-reminder text (or its confirmed absence) for the new recipe via FEAT-13.SPEC-006.
7. Dispatch the correction message (FEAT-13.SPEC-004) to each member from step 5, naming the new dinner.
8. Record the correction firing cap for this household-date.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Correction dispatched | Swap completes for tonight, an original nudge was dispatched today, and the correction cap is not yet recorded | Correction firing cap recorded for this household-date | Each member who received today's original nudge receives FEAT-13.SPEC-004's follow-up naming the new dinner | FEAT-13.SPEC-004, FEAT-13.SPEC-005 |
| No-action: swap not for tonight | The swapped Planned Meal's night is not today | None | Nothing sent; a swap for a future night follows its own normal path | -- |
| No-action: no original nudge sent today | Today's original nudge was never dispatched (no dinner existed at nudge time, no eligible members, or today's evaluation has not yet occurred) | None | Nothing sent; there is no delivered original nudge for anyone to have been left with a stale version of | FEAT-13.SPEC-005 |
| No-action: correction already sent today | The at-most-one-correction cap for this household-date is already recorded | None | Nothing sent; a second same-day swap after the first correction is not separately announced | FEAT-13.SPEC-005 |
| Dispatch failure | Dispatch to one or more of today's original-nudge recipients fails | The correction firing cap is not recorded for members whose dispatch could not be attempted | No error is shown; the Planned Meal remains visible in-app with its new dinner, unaffected | FEAT-13.SPEC-004 |

## Data Model

**Reads:** Planned Meal -- night, recipe (new), swap_history (previous recipe), status; Recipe -- prep_requirements for the new recipe, via FEAT-13.SPEC-006; Member Profile -- the dispatched-members record from today's original nudge (FEAT-13.SPEC-001), via FEAT-13.SPEC-005.
**Creates:** None.
**Updates:** None on Planned Meal, Recipe, or Member Profile -- this automation records only its own correction firing-cap marker (read by FEAT-13.SPEC-005), not a dependency-map entity.
**Deletes:** None.

## Business Rules

- XBR-09: A same-day swap after the nightly nudge was sent triggers at most one follow-up correction naming the new dinner.
- This automation is the sole trigger source for FEAT-13.SPEC-004 -- no other event ever fires the correction message.
- The correction audience is exactly today's original-nudge recipients, never a freshly re-evaluated eligible list -- a member who never received today's original nudge is never owed a correction for it.
- At most one correction per household per day, regardless of how many further swaps occur the same night after the first correction (Brief, Validation & Limits).
- This automation never determines the correction audience or the once-per-day correction cap itself -- it calls FEAT-13.SPEC-005 for both.

## Edge Cases

- **Concurrent trigger firing (two swaps for the same slot complete at effectively the same time)** -- FEAT-04.SPEC-009's concurrency lock, referenced by FEAT-04.SPEC-004, ensures only one swap actually completes for a given slot at a time, so this automation never receives two concurrent swap-applied signals for the same Planned Meal.
- **Trigger fires while a previous correction run is in flight** -- A second swap-applied signal for the same household arriving while an earlier correction dispatch is still recording its cap is held until that run finishes, then evaluated against the now-recorded cap and suppressed as a duplicate.
- **A second same-day swap happens after the first correction was already sent** -- The at-most-one-correction cap suppresses a second correction; the household's Weekly Plan view already reflects the latest dinner regardless (FEAT-03.SPEC-001, FEAT-23.SPEC-001), so only the notification is capped, not the plan's accuracy.
- **The swap completes for tonight while today's original nudge dispatch is itself still in flight** -- Step 3's check reads whatever dispatched-members state exists at the moment this automation runs; if the original nudge has not yet recorded any dispatched members, this run treats it as "no original nudge sent today" and takes no action, since there is nothing yet to correct.
- **The swap is undone by a second swap back toward the original recipe before any correction fires** -- Both swaps are separate "Swap applied" outcomes from FEAT-04.SPEC-004; the at-most-one-correction cap means only the first eligible swap's correction can ever fire, and a second corrective message announcing the reversal is not separately sent.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-004 (Apply Meal Swap) | Triggered by (inbound) | Its "Swap applied" outcomes fire this automation for tonight's slot |
| FEAT-13.SPEC-001 (Tonight's Nudge Trigger) | References (inbound) | Reads that automation's dispatched-members record for today to determine whether a correction is owed |
| FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules) | References (outbound) | Correction audience and once-per-day correction cap are read from this rule |
| FEAT-13.SPEC-006 (Prep-Reminder Derivation Rule) | References (outbound) | Prep-reminder text for the new recipe is read from this rule |
| FEAT-13.SPEC-004 (Same-Day Swap Correction Message) | Triggers (outbound) | Dispatches the correction to the resolved audience |

## Analytics and Success Signals

- **dinner_nudge_correction_dispatched** (household id, recipient_count) -- N/A -- no metric in success-metrics.md names Tonight's Dinner Reminder as its Connected Feature; retained per product-features.md's own Signals field (dinner_nudge_correction_sent) so correction reach is observable even without a Stage 2 metric attached.
- **dinner_nudge_correction_skipped_no_original_sent** (household id) -- N/A -- same reason; observes how often a same-day swap has no original nudge to correct.
- **dinner_nudge_correction_skipped_cap_reached** (household id) -- N/A -- same reason; observes how often the at-most-one-correction cap suppresses a further same-day swap's correction.

## Acceptance Criteria

**FEAT-13.SPEC-003-AC-01:** Given Maya applies a direct swap to tonight's dinner after today's original nudge already reached both her and Sam, when FEAT-04.SPEC-004 reports the swap applied, then both Maya and Sam receive FEAT-13.SPEC-004's correction naming the new dinner.

**FEAT-13.SPEC-003-AC-02:** Given a swap completes for a night that is not tonight, when FEAT-04.SPEC-004 reports it, then no correction is dispatched.

**FEAT-13.SPEC-003-AC-03:** Given today's original nudge was never dispatched (no dinner existed at nudge time), when a same-day swap later adds a dinner to tonight's slot, then no correction is dispatched, since there was no original nudge to correct.

**FEAT-13.SPEC-003-AC-04:** Given a correction has already been sent for tonight, when a second same-day swap changes tonight's dinner again, then no second correction is sent to anyone.

**FEAT-13.SPEC-003-AC-05:** Given Sam turned his nightly nudge off and so never received today's original nudge, when a same-day swap triggers a correction for Maya, then Sam does not receive the correction.

**FEAT-13.SPEC-003-AC-06:** Given two swaps for the same slot attempt to complete at effectively the same time, when FEAT-04.SPEC-009's concurrency lock resolves them, then this automation only ever processes one completed swap-applied signal for that slot.

**FEAT-13.SPEC-003-AC-07:** Given a second swap-applied signal for the same household arrives while an earlier correction run is still recording its cap, when the in-flight run finishes, then the second signal is evaluated against the now-recorded cap and suppressed.

**FEAT-13.SPEC-003-AC-08:** Given the original nudge dispatch for today is itself still in flight when a swap completes, when this automation checks for dispatched members, then it finds none yet recorded and takes no action.

**FEAT-13.SPEC-003-AC-09:** Given a swap is reversed by a second same-day swap before any correction fires, when the reversal completes, then only the first eligible swap's correction can ever be attempted, and no separate message announces the reversal.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 (swap-applied outcome) | 1 |
| Outcome Paths | 5 (dispatched, not-tonight, no-original-sent, cap-reached, dispatch failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |
