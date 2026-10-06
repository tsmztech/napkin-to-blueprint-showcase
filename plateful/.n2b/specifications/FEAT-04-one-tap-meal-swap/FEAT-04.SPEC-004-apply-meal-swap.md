---
document_type: spec
spec_type: automation
spec_id: FEAT-04.SPEC-004
spec_name: Apply Meal Swap
spec_slug: apply-meal-swap
parent_feature: FEAT-04
parent_feature_name: One-Tap Meal Swap
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Apply Meal Swap

## Overview

**Name:** Apply Meal Swap
**ID:** FEAT-04.SPEC-004
**Type:** Automation
**Purpose:** Shared core action that re-verifies safety, writes the new recipe onto the plan slot, closes any superseded suggestion, and signals the grocery list to recalculate.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap

## Scope and Non-Goals

**In Scope:**
- The single write path that moves a Planned Meal from its current recipe to a new one, whether initiated by a direct swap or an accepted suggestion
- Re-verifying the chosen recipe's safety immediately before writing (never trusting a safety check performed earlier)
- Superseding any other open suggestion on the same slot
- Signaling the Shared Grocery List (FEAT-06) to recalculate

**Non-Goals:**
- Computing which recipes qualify as alternatives -- owned by FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation); this automation only writes the recipe it is given
- Acquiring the concurrency lock -- owned by FEAT-04.SPEC-009 (Swap Concurrency Lock), which the triggering screen invokes before calling this automation
- Marking a suggestion Declined or Lapsed -- those outcomes are set by FEAT-04.SPEC-003 (a direct decline) and FEAT-04.SPEC-005 (lapse) respectively, not by this automation
- Recalculating the grocery list's contents -- owned by FEAT-06 (Shared Grocery List) per the feature-dependency-map.md authority column and XBR-03; this automation only triggers that recalculation

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Maya selects an alternative on a direct swap | FEAT-04.SPEC-001 (Meal Swap Direct) | Fires after the concurrency lock (FEAT-04.SPEC-009) is acquired for the slot | Target Planned Meal reference, chosen recipe, initiating member (Maya) |
| Maya accepts a pending suggestion | FEAT-04.SPEC-003 (Review Swap Suggestions) | Fires after the concurrency lock is acquired for the slot | Target Planned Meal reference, the suggestion's proposed_recipe, the Swap Suggestion reference, initiating member (Maya) |

## Processing Logic

1. Receive the target Planned Meal reference and the chosen recipe (from either trigger path).
2. Re-run the same allergy/religious hard-rule safety check the recipe would need to pass on original plan generation (XBR-01), scoped to the household's current dietary rules -- never reusing a safety result computed earlier in the flow.
3. If the safety re-check fails, stop processing and report the Safety Re-check Failed outcome (no write occurs).
4. If the safety re-check passes, write the chosen recipe onto the Planned Meal's recipe field, set its status to Swapped, and append the previous recipe to its swap_history.
5. If the trigger path was an accepted suggestion, set that Swap Suggestion's outcome to Accepted (via FEAT-04.SPEC-010).
6. Check for any other open Swap Suggestion on the same slot (from a different member, or a stale one from the same member) and supersede it via FEAT-04.SPEC-010's superseding rule.
7. Release the concurrency lock on the slot (FEAT-04.SPEC-009).
8. Signal the Shared Grocery List (FEAT-06) that the plan changed, so it recalculates.
9. If the initiating action was an accepted suggestion, signal FEAT-04.SPEC-006 to notify the suggesting member that it was accepted.
10. Return success to the triggering screen.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Swap applied (direct) | Safety re-check passes, triggered from FEAT-04.SPEC-001 | Planned Meal recipe, status, swap_history updated; any other open suggestion on the slot superseded (FEAT-04.SPEC-010); grocery list signaled | Weekly Plan shows the new dinner in the slot immediately | FEAT-04.SPEC-001, FEAT-06, FEAT-11.SPEC-004, FEAT-13.SPEC-003, FEAT-21.SPEC-003 |
| Swap applied (accepted suggestion) | Safety re-check passes, triggered from FEAT-04.SPEC-003 | Same as above, plus the accepted Swap Suggestion's outcome set to Accepted | The suggestion card is removed from FEAT-04.SPEC-003's list; the suggesting member is notified their suggestion was accepted (FEAT-04.SPEC-006); Weekly Plan shows the new dinner | FEAT-04.SPEC-003, FEAT-04.SPEC-010, FEAT-04.SPEC-006, FEAT-06, FEAT-11.SPEC-004, FEAT-13.SPEC-003, FEAT-21.SPEC-003 |
| Safety Re-check Failed | The chosen recipe no longer passes the household's current hard dietary rules at write time | No data changes -- the Planned Meal and any suggestion are left exactly as they were | From a direct swap: FEAT-04.SPEC-001 shows "This meal changed while you were choosing. Here's the latest." and the slot is unchanged. From an accepted suggestion: FEAT-04.SPEC-003 shows "This suggestion no longer passes the household's dietary rules and can't be applied." on that card | FEAT-04.SPEC-001, FEAT-04.SPEC-003 |
| Write Failure | The safety re-check passes but the underlying write does not complete (e.g., a dropped connection) | No partial state -- the Planned Meal remains at its prior recipe and status; the lock is released so a retry can proceed | From a direct swap: "Couldn't complete the swap. Your original dinner is still on the plan." with a retry option. From an accepted suggestion: "Couldn't complete this action. Try again." on that card | FEAT-04.SPEC-001, FEAT-04.SPEC-003 |

## Data Model

**Reads:** Planned Meal -- current recipe, status (to confirm the slot is eligible to be written); Swap Suggestion -- proposed_recipe, outcome (when triggered by an accept); Dietary Rule -- read only through FEAT-02's safety-check capability, never stored or displayed directly by this automation (per the dependency map's Data Sensitivity note for FEAT-04).
**Creates:** None.
**Updates:** Planned Meal -- recipe, status (set to Swapped), swap_history (previous recipe appended); Swap Suggestion -- outcome (set to Accepted, when triggered by an accept) and, for any other open suggestion on the slot, outcome (set to Declined via superseding, per FEAT-04.SPEC-010).
**Deletes:** None.

## Business Rules

- The safety re-check is mandatory and synchronous: the write never proceeds ahead of it, regardless of trigger path (XBR-01).
- This automation is the single write path onto Planned Meal.recipe for a swap -- both trigger sources converge here so the safety re-check has exactly one home (Brief, Analyst-Discovered Specs rationale).
- A direct swap by Maya always supersedes any open suggestion on the same slot, including one raised after the swap started but before it completes (FEAT-04.SPEC-010; dependency map, Swap Suggestion Contention).
- XBR-03: completing a swap or accepting a suggestion always signals the Shared Grocery List to recalculate immediately -- no swap completes without that signal firing.
- XBR-09: if a same-day swap completes after the nightly nudge (FEAT-13) has already been sent for that night, this automation's completion is the trigger for FEAT-13's one-time correction notification; this automation does not itself send that notification, only makes the completed swap visible for FEAT-13 to detect.

## Edge Cases

- **The safety re-check fails for a recipe that passed when the alternatives list was built** -- A hard rule was tightened in the interim (XBR-02); the swap does not apply and the triggering screen shows its Safety Re-check Failed message from the Outcome Definitions table above.
- **The target slot no longer exists (removed by a safety-concern report between selection and this automation firing)** -- The write is refused; the triggering screen is told "This meal changed while you were choosing. Here's the latest." and reloads the current slot state, consistent with the Planned Meal Contention note (a safety removal always wins over a concurrent change).
- **Concurrent trigger firing (a direct swap and an accepted suggestion for the same slot fire at effectively the same time)** -- The concurrency lock (FEAT-04.SPEC-009) ensures only one of the two triggers can proceed through this automation at a time; the second is rejected before this automation is even invoked, per FEAT-04.SPEC-009's lock semantics. This automation itself never receives two concurrent invocations for the same slot.
- **Trigger fires while a previous run for the same slot is still in flight** -- Cannot occur: the concurrency lock held by the in-flight run prevents a second invocation for the same slot from starting (FEAT-04.SPEC-009). Invocations for different slots proceed independently and never queue behind each other.
- **The Shared Grocery List signal is not acknowledged (FEAT-06 degraded)** -- The Planned Meal write still completes and is treated as successful; the grocery list recalculates when FEAT-06's own recovery behavior allows, per FEAT-06's degradation handling (not owned by this spec). The user is never told the swap failed because of this.
- **An accepted suggestion's slot was already changed by a prior direct swap moments earlier** -- Treated identically to the Safety Re-check Failed / target-slot-changed case: the accept is refused with the "no longer... can't be applied" message if the recipe fails re-check, or the general concurrent-change message if the slot itself changed underneath it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-001 (Meal Swap Direct) | Triggered by (inbound) | Direct swap selection fires this automation |
| FEAT-04.SPEC-003 (Review Swap Suggestions) | Triggered by (inbound) | Accepting a suggestion fires this automation |
| FEAT-04.SPEC-009 (Swap Concurrency Lock) | References (inbound) | Lock must be held before this automation runs, and is released when it completes |
| FEAT-04.SPEC-010 (Suggestion Lifecycle Rules) | Triggers (outbound) | Sets the accepted suggestion's outcome and supersedes any other open suggestion on the slot |
| FEAT-04.SPEC-006 (Swap Suggestion Notifications) | Triggers (outbound) | Notifies the suggesting member when their suggestion is accepted |
| FEAT-06 (Shared Grocery List) | Triggers (outbound) | Signaled to recalculate immediately on every successful swap |
| FEAT-13 (Tonight's Dinner Reminder) | Affects (outbound) | A same-day completion after the nightly nudge triggers FEAT-13's correction notification (XBR-09) |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | References (outbound) | Provides the safety re-check this automation always runs before writing |

## Analytics and Success Signals

- **meal_swap_completed** (trigger path: direct / accepted_suggestion; slot night; time from trigger to completion) -- supports success-metrics.md: "One-Tap Swap Completion"
- **meal_swap_failed** (trigger path; reason: safety_recheck_failed / write_failure) -- supports success-metrics.md: "One-Tap Swap Completion"
- **swap_suggestion_accepted** (slot night, suggesting member) -- supports success-metrics.md: "Weekly Planning Time"
- **allergy_safety_recheck_blocked_swap** (trigger path) -- supports success-metrics.md: "Zero Allergy Incidents" (confirms the safety re-check is genuinely enforced at the moment of every swap, not only at original plan generation)

## Acceptance Criteria

**FEAT-04.SPEC-004-AC-01:** Given Maya selects a safe alternative on FEAT-04.SPEC-001, when this automation fires, then the safety re-check passes, the Planned Meal's recipe updates, its status becomes Swapped, and the grocery list is signaled to recalculate.

**FEAT-04.SPEC-004-AC-02:** Given Maya accepts Sam's suggestion on FEAT-04.SPEC-003, when this automation fires, then the safety re-check passes, the Planned Meal updates, the suggestion's outcome is set to Accepted, and Sam is notified the suggestion was accepted.

**FEAT-04.SPEC-004-AC-03:** Given a household hard dietary rule was tightened after a suggestion was raised but before Maya accepts it, when this automation runs the safety re-check on accept, then the write is refused and FEAT-04.SPEC-003 shows "This suggestion no longer passes the household's dietary rules and can't be applied."

**FEAT-04.SPEC-004-AC-04:** Given the safety re-check passes but the write itself fails (e.g., a dropped connection), when this automation reports the failure, then no partial state is left on the Planned Meal and the triggering screen shows its retry message.

**FEAT-04.SPEC-004-AC-05:** Given a second open suggestion exists on the same slot as the one Maya just accepted, when this automation completes, then the other suggestion is superseded (its outcome set to Declined) via FEAT-04.SPEC-010.

**FEAT-04.SPEC-004-AC-06:** Given Maya applies a direct swap on a slot that also has a pending suggestion from Sam, when this automation completes, then Sam's pending suggestion is superseded.

**FEAT-04.SPEC-004-AC-07:** Given a swap completes on a night for which FEAT-13's nightly nudge was already sent today, when this automation reports success, then FEAT-13's same-day correction notification fires (XBR-09).

**FEAT-04.SPEC-004-AC-08:** Given a concurrency lock is already held for a slot, when a second trigger for the same slot would otherwise fire this automation, then the automation never runs a second time concurrently for that slot -- the second trigger is rejected before invocation, per FEAT-04.SPEC-009.

**FEAT-04.SPEC-004-AC-09:** Given the target slot was removed by a safety-concern report between selection and this automation firing, when this automation runs, then the write is refused and the triggering screen shows the concurrent-change message.

**FEAT-04.SPEC-004-AC-10:** Given the Shared Grocery List signal is not immediately acknowledged, when this automation completes the Planned Meal write, then the swap is still reported as successful to the user.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (direct swap, accepted suggestion) | 2 |
| Outcome Paths | 4 (applied-direct, applied-accepted, safety-recheck-failed, write-failure) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
