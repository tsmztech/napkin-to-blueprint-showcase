---
document_type: spec
spec_type: automation
spec_id: FEAT-04.SPEC-005
spec_name: Suggestion Lapse
spec_slug: suggestion-lapse
parent_feature: FEAT-04
parent_feature_name: One-Tap Meal Swap
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Automation Spec: Suggestion Lapse

## Overview

**Name:** Suggestion Lapse
**ID:** FEAT-04.SPEC-005
**Type:** Automation
**Purpose:** Time-based check that marks an unanswered swap suggestion Lapsed when its night passes and frees the slot for a new suggestion.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap

## Scope and Non-Goals

**In Scope:**
- Detecting every Swap Suggestion whose target night has passed while its outcome is still Suggested
- Setting the outcome to Lapsed and freeing the slot for a new suggestion from the same member
- Signaling FEAT-04.SPEC-006 to notify the suggesting member of the lapse

**Non-Goals:**
- Accepting or declining a suggestion -- those are Maya's actions in FEAT-04.SPEC-003; this automation only handles the case where she never acted
- Notifying the organiser about a lapse -- product-features.md's Communications field states only the suggesting member is told when a suggestion lapses; the organiser is not separately notified since she took no action to reverse
- Any retroactive change to the Planned Meal -- a lapsed suggestion simply stops being pending; the slot's current recipe (whatever it was before the suggestion) is untouched, since a suggestion never wrote to the Planned Meal while it was open

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nightly lapse check | system (schedule-based) | Runs once daily, after the last moment a night's dinner could still be swapped or accepted (end of that calendar day, household local time) | All Swap Suggestion records with outcome = Suggested whose night is on or before the day that just ended |

## Processing Logic

1. At the scheduled run time, read every Swap Suggestion with outcome = Suggested.
2. For each, compare its night against the current date in the household's local time.
3. If the suggestion's night is on or before the day that has just ended, mark it as lapsed.
4. Set that suggestion's outcome to Lapsed via FEAT-04.SPEC-010.
5. Signal FEAT-04.SPEC-006 to notify the suggesting member that their suggestion lapsed.
6. Suggestions whose night has not yet passed are left untouched and re-evaluated on the next run.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Suggestion lapsed | The suggestion's night passed with no accept or decline recorded | Swap Suggestion outcome set to Lapsed | The suggesting member is notified their suggestion lapsed (FEAT-04.SPEC-006); the suggestion no longer appears in FEAT-04.SPEC-003's pending list; the suggesting member's screen (FEAT-04.SPEC-002) shows the lapsed outcome next time it opens | FEAT-04.SPEC-002, FEAT-04.SPEC-003, FEAT-04.SPEC-006, FEAT-04.SPEC-010 |
| No lapses found | No Suggested-outcome suggestion has a passed night at run time | None | No user-visible effect | None |

## Data Model

**Reads:** Swap Suggestion -- outcome, night (to find every pending suggestion whose night has passed).
**Creates:** None.
**Updates:** Swap Suggestion -- outcome set to Lapsed, via FEAT-04.SPEC-010.
**Deletes:** None -- lapsed suggestions are retained with a terminal outcome as part of the plan's permanent history (Brief, Entity-Lifecycle Coverage Matrix).

## Business Rules

- A suggestion "lapses quietly" once its night passes (product-features.md, Primary Flows & Alternates) -- there is no grace period and no reminder to Maya before the lapse.
- XBR-06: a suggestion not answered before its night lapses, and the suggester is told the outcome.
- Lapsing a suggestion frees its slot: the suggesting member may raise a new suggestion for a future occurrence of that slot without being blocked by the lapsed one (FEAT-04.SPEC-010's one-open-per-member-per-slot rule only counts Suggested-outcome suggestions).

## Edge Cases

- **Maya accepts or declines a suggestion in the same window this automation runs (race between a human action and the scheduled check)** -- First-decision-wins with reject-with-refresh, per the dependency map's Contention note for Swap Suggestion: whichever resolution (Maya's accept/decline, or this automation's lapse) writes the outcome first stands; the later attempt finds the suggestion already resolved and is a no-op. A late accept specifically is refused per FEAT-04.SPEC-010's late-accept rule.
- **Concurrent trigger firing (two scheduled runs somehow overlap)** -- Each suggestion's outcome write is idempotent: a suggestion already set to Lapsed by one run is a no-op for the other, and no duplicate lapse notification is sent (deduplication owned by FEAT-04.SPEC-006).
- **Trigger fires while a previous run is still in flight** -- The nightly check does not start a new run until the prior one completes; there is only ever one active run, so overlap cannot occur.
- **A suggestion's night was itself changed by a plan edit before the check runs** -- Not applicable: a Swap Suggestion's night is fixed to the slot it targets and is never edited independently of the suggestion itself (per the dependency map's Swap Suggestion field definitions); this scenario cannot arise.
- **The household's local time zone is ambiguous or changes (e.g., daylight saving transition) on the night in question** -- The check uses the household's configured locale settings (FEAT-16) to determine "end of day," consistent with how the household's schedule and nightly nudge are timed elsewhere in the product.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-010 (Suggestion Lifecycle Rules) | Triggers (outbound) | Sets the suggestion's outcome to Lapsed and frees the slot |
| FEAT-04.SPEC-006 (Swap Suggestion Notifications) | Triggers (outbound) | Notifies the suggesting member of the lapse |
| FEAT-04.SPEC-002 (Suggest a Swap) | Affects (outbound) | The suggester's screen reflects the lapsed outcome on next view |
| FEAT-04.SPEC-003 (Review Swap Suggestions) | Affects (outbound) | The suggestion is removed from the pending list |

## Analytics and Success Signals

- **swap_suggestion_lapsed** (slot night, suggesting member, days pending before lapse) -- supports success-metrics.md: "Weekly Planning Time" (a high lapse rate signals suggestions are not being reviewed within the organiser's planning window)

## Acceptance Criteria

**FEAT-04.SPEC-005-AC-01:** Given Sam's suggestion for Wednesday's dinner is still Suggested when Wednesday ends, when the nightly lapse check runs, then the suggestion's outcome is set to Lapsed.

**FEAT-04.SPEC-005-AC-02:** Given a suggestion has just lapsed, when this automation completes, then Sam receives the lapse notification (FEAT-04.SPEC-006).

**FEAT-04.SPEC-005-AC-03:** Given a suggestion lapsed for a slot, when Sam opens FEAT-04.SPEC-002 for that same slot in a future week, then he can raise a new suggestion without being blocked by the lapsed one.

**FEAT-04.SPEC-005-AC-04:** Given no suggestions have a passed night at run time, when the nightly lapse check runs, then no outcomes change and no notifications are sent.

**FEAT-04.SPEC-005-AC-05:** Given Maya accepts a suggestion in the same moment the nightly check would otherwise lapse it, when both attempt to resolve the suggestion, then whichever resolves first stands and the other is a no-op.

**FEAT-04.SPEC-005-AC-06:** Given the nightly lapse check somehow runs twice for the same night, when the second run processes an already-lapsed suggestion, then no duplicate lapse notification is sent.

**FEAT-04.SPEC-005-AC-07:** Given a suggestion's night has not yet passed, when the nightly lapse check runs, then that suggestion is left untouched and remains pending.

**FEAT-04.SPEC-005-AC-08:** Given the suggestion lapses, when Sam next opens FEAT-04.SPEC-002 for that slot, then he sees the banner "Your suggestion lapsed -- the night passed unanswered."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 2 (lapsed, no lapses found) | 2 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
