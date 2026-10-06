---
document_type: spec
spec_type: automation
spec_id: FEAT-11.SPEC-004
spec_name: Leftover Lunch Withdrawal on Source Change
spec_slug: leftover-lunch-withdrawal-on-source-change
parent_feature: FEAT-11
parent_feature_name: Leftover Rollover to Lunches
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Leftover Lunch Withdrawal on Source Change

## Overview

**Name:** Leftover Lunch Withdrawal on Source Change
**ID:** FEAT-11.SPEC-004
**Type:** Automation
**Purpose:** When a linked source dinner is swapped or removed, re-evaluates the leftover-lunch suggestion attached to it and either re-links or withdraws it.
**Parent Feature:** FEAT-11 -- Leftover Rollover to Lunches

## Scope and Non-Goals

**In Scope:**
- Detecting that a dinner Planned Meal with a linked, Suggested leftover lunch has had its recipe swapped, or has been cleared or changed manually
- Re-running FEAT-11.SPEC-003's eligibility determination against the changed dinner
- Re-linking the existing leftover-lunch record when the new dinner is still eligible, or hard-deleting it when it is not
- Leaving an already-Confirmed (Eaten) or already-Skipped leftover lunch untouched regardless of what happens to its former source dinner

**Non-Goals:**
- Determining eligibility or the following-day placement from first principles -- owned by FEAT-11.SPEC-003 (Leftover Lunch Eligibility & Linking Rule); this automation calls that rule rather than re-deriving it
- Creating the first leftover-lunch suggestion for a newly generated week -- owned by FEAT-11.SPEC-002 (Leftover Lunch Suggestion Generation), a distinct trigger path
- Restoring a withdrawn leftover lunch -- product-features.md's Validation & Limits and the Entity-Lifecycle Coverage Matrix establish no restore path; a fresh suggestion appears only if a later plan-generation cycle or this automation's own re-link path attaches a newly eligible dinner to the slot
- Notifying any household member that a leftover lunch was updated or withdrawn -- excluded per this feature's own Communications field (product-features.md): no separate notification is sent for any leftover-lunch outcome

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A source dinner is swapped | FEAT-04.SPEC-004 (Apply Meal Swap) | Fires when a swap completes successfully on a Planned Meal slot that has a linked leftover lunch | The changed Planned Meal's new recipe, night; the linked leftover-lunch Planned Meal's current status, night, and source reference |
| A source dinner is cleared or changed manually | FEAT-23 (Manual Weekly Planning) | Fires when a night's dinner Planned Meal is cleared or its recipe changed in a manually built week, and that slot has a linked leftover lunch | The changed (or now-empty) Planned Meal slot's new recipe if any, night; the linked leftover-lunch Planned Meal's current status, night, and source reference |

## Processing Logic

1. Receive the changed dinner Planned Meal (its new recipe, or notice that the slot is now empty) and the leftover-lunch record currently linked to it.
2. If no leftover-lunch record is linked to the changed slot, take no action -- most swaps and manual changes never touch a linked leftover lunch.
3. If a leftover-lunch record is linked but its status is Eaten or Skipped, take no action -- confirmed history is never altered by a later source change (scope-boundaries.md SC-18).
4. If the linked leftover-lunch record's status is Suggested, pass the new dinner (or the empty slot) through FEAT-11.SPEC-003's eligibility determination.
5. If the new dinner is classified leftover-producing, re-link the existing leftover-lunch record's source reference to it; its night is only recomputed if the slot's own night changed (a swap or manual recipe change never changes which night the slot occupies, so the night typically stays the same).
6. If the new dinner is not classified leftover-producing, or the slot is now empty with no replacement dinner, hard-delete the Suggested leftover-lunch record.
7. Signal the Leftover Lunch Card (FEAT-11.SPEC-001) and the hosting Weekly Plan screen to reflect the re-link or removal.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Re-linked to new source dinner | The changed slot's new dinner is still classified leftover-producing, and the linked leftover lunch is Suggested | The leftover-lunch record's linked source dinner reference is updated; night unchanged unless the slot's night itself changed | The Leftover Lunch Card now shows the new dinner's recipe name; no toast or interruption | FEAT-11.SPEC-001 |
| Withdrawn (removed) | The changed slot's new dinner is not leftover-producing, or the slot is now empty, and the linked leftover lunch is Suggested | The Suggested leftover-lunch record is hard-deleted; no restore path | The Leftover Lunch Card disappears from the day it was attached to on the next read; no error or notification is shown | FEAT-11.SPEC-001 |
| No action -- already resolved | The linked leftover lunch is Eaten or Skipped | None | The card continues to show its resolved Eaten or Skipped state unchanged, even though its former source dinner has changed | FEAT-11.SPEC-001 |
| No action -- no linked leftover lunch | The changed slot has no linked leftover-lunch record at all | None | Nothing changes; this is the most common outcome, since most dinners are not leftover-producing | -- |
| Re-evaluation failure (fail-safe withdrawal) | The eligibility re-check itself cannot complete for the changed dinner | The Suggested leftover-lunch record is withdrawn (hard-deleted) as a conservative default rather than left pointing at a stale or unverified source | The card disappears; no error is shown, consistent with product-features.md's "a failure to compute it simply omits the suggestion rather than showing an error" | FEAT-11.SPEC-001 |

## Data Model

**Reads:** Planned Meal (source dinner, post-change) -- recipe, night, status. Planned Meal (leftover-lunch sub-type) -- status, linked source dinner, night, for the record attached to the changed slot.
**Creates:** None -- re-linking updates the existing record; a fresh record for a different, still-empty slot is only ever created by FEAT-11.SPEC-002's own generation cycle.
**Updates:** Planned Meal (leftover-lunch sub-type) -- linked source dinner (on re-link); night, only when the slot's own night changed.
**Deletes:** Planned Meal (leftover-lunch sub-type) -- hard delete of the Suggested record when withdrawn, per the Entity-Lifecycle Coverage Matrix's Delete/Archive row.

## Business Rules

- XBR-10: a source dinner swap (FEAT-04) or manual clear/change (FEAT-23) always re-evaluates or withdraws its linked leftover suggestion; the link is never left pointing at a dinner that no longer exists in that slot.
- Confirmed Eaten or Skipped leftover lunches are never touched by this automation -- they are retained as permanent plan history (scope-boundaries.md SC-18), regardless of what happens to their former source dinner.
- Withdrawal is a hard delete with no restore path -- a fresh suggestion for that slot appears only if a newly eligible dinner takes it, whether through this automation's own re-link path or a future plan-generation cycle (FEAT-11.SPEC-002).
- A re-evaluation failure defaults to withdrawal rather than leaving the leftover lunch linked to a stale or unverified source -- a missing suggestion is preferred over an incorrect one, consistent with the product's general failure posture for this feature (product-features.md, States).
- This automation never creates a leftover-lunch record for a slot that never had one -- only FEAT-11.SPEC-002's generation cycle originates new suggestions.

## Edge Cases

- **Source dinner swapped to a recipe that is also leftover-producing** -- The existing Suggested record is re-linked to the new recipe rather than deleted and recreated, preserving its following day (the slot's night is unchanged by a swap).
- **Source dinner swapped to a recipe that is not leftover-producing** -- The existing Suggested record is withdrawn; the household simply loses that day's leftover-lunch card with no error shown.
- **A night is cleared entirely in a manually built week (FEAT-23), with no replacement dinner chosen** -- Treated the same as a swap to a non-eligible recipe: the linked Suggested leftover lunch is withdrawn.
- **The linked leftover lunch is already marked Eaten when its source dinner is later swapped** -- No action is taken; the confirmed record is untouched (per Outcome Definitions).
- **The linked leftover lunch is already marked Skipped when its source dinner is later cleared** -- No action is taken; the Skipped record is retained exactly as it was.
- **Concurrent trigger firing (a swap and a manual change targeting the same slot at effectively the same time)** -- Cannot occur: FEAT-04.SPEC-009's concurrency lock (for swaps) and Manual Weekly Planning's own per-slot handling ensure only one change to a given slot completes at a time; this automation is invoked once per completed, settled change, never twice concurrently for the same slot.
- **Trigger fires while a previous run of this automation for the same slot is still in flight** -- Cannot occur for the same reason: a second change to the same slot cannot begin until the first one's own concurrency control (FEAT-04.SPEC-009 for swaps) releases, so this automation's runs for a given slot are naturally serialized.
- **The withdrawn slot later receives a newly eligible dinner in a subsequent weekly plan-generation cycle** -- A brand-new Suggested leftover lunch may be created then by FEAT-11.SPEC-002, entirely independent of the record this automation withdrew earlier.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-004 (Apply Meal Swap) | Triggered by (inbound) | A completed swap on a slot with a linked leftover lunch fires this automation |
| FEAT-23 (Manual Weekly Planning) | Triggered by (inbound) | Clearing or changing a night manually on a slot with a linked leftover lunch fires this automation |
| FEAT-11.SPEC-003 (Leftover Lunch Eligibility & Linking Rule) | References (outbound) | Supplies the eligibility determination this automation re-runs against the changed dinner |
| FEAT-11.SPEC-002 (Leftover Lunch Suggestion Generation) | References (outbound) | Owns fresh suggestion creation; this automation only re-links or withdraws an existing record |
| FEAT-11.SPEC-001 (Leftover Lunch Card) | Affects (outbound) | Displays the re-linked dinner, or stops displaying a withdrawn card, on its next read |

## Analytics and Success Signals

- **leftover_lunch_relinked** (former source dinner night, new source dinner night) -- supports success-metrics.md: "Reported Food Waste and Spend Reduction" (keeping the suggestion accurate after a swap keeps the waste-reduction mechanism trustworthy)
- **leftover_lunch_withdrawn** (reason: source_no_longer_eligible / slot_cleared / reevaluation_failure) -- N/A -- no Stage 2 metric measures withdrawal volume directly; retained to observe how often a source change disrupts an existing suggestion

## Acceptance Criteria

**FEAT-11.SPEC-004-AC-01:** Given a Suggested leftover lunch is linked to a dinner that Maya swaps for another leftover-producing recipe, when the swap completes (FEAT-04.SPEC-004), then this automation re-links the leftover lunch to the new recipe and its card shows the new dinner's name.

**FEAT-11.SPEC-004-AC-02:** Given a Suggested leftover lunch is linked to a dinner that Maya swaps for a recipe that is not leftover-producing, when the swap completes, then this automation withdraws the leftover lunch and its card disappears.

**FEAT-11.SPEC-004-AC-03:** Given a Suggested leftover lunch is linked to a dinner that is cleared entirely in a manually built week with no replacement, when the clear is applied (FEAT-23), then this automation withdraws the leftover lunch.

**FEAT-11.SPEC-004-AC-04:** Given a manually built week's dinner is changed to a still leftover-producing recipe, when the change is applied, then this automation re-links the existing leftover lunch to the new recipe rather than creating a duplicate.

**FEAT-11.SPEC-004-AC-05:** Given a leftover lunch has already been marked Eaten, when its former source dinner is later swapped, then this automation takes no action and the Eaten record is unchanged.

**FEAT-11.SPEC-004-AC-06:** Given a leftover lunch has already been marked Skipped, when its former source dinner is later cleared manually, then this automation takes no action and the Skipped record is unchanged.

**FEAT-11.SPEC-004-AC-07:** Given a dinner with no linked leftover lunch is swapped, when the swap completes, then this automation takes no action, since there is nothing linked to re-evaluate.

**FEAT-11.SPEC-004-AC-08:** Given the eligibility re-check cannot complete for a changed dinner, when this automation processes the trigger, then the linked Suggested leftover lunch is withdrawn as a conservative default rather than left pointing at an unverified source.

**FEAT-11.SPEC-004-AC-09:** Given a swap and a manual change could otherwise target the same slot at the same time, when both are attempted, then the underlying concurrency controls (FEAT-04.SPEC-009 for swaps) ensure only one change completes, and this automation runs exactly once for the settled result.

**FEAT-11.SPEC-004-AC-10:** Given a leftover lunch was withdrawn from a slot, when a later weekly plan-generation cycle places a newly eligible dinner in that same slot, then FEAT-11.SPEC-002 creates a brand-new Suggested leftover lunch independent of the one this automation withdrew.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (swap, manual clear/change) | 2 |
| Outcome Paths | 5 (re-linked, withdrawn, no-action-resolved, no-action-unlinked, reevaluation-failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |
