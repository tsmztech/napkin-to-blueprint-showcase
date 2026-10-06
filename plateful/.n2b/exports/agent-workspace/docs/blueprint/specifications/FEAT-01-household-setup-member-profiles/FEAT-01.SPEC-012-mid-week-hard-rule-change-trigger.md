---
document_type: spec
spec_type: automation
spec_id: FEAT-01.SPEC-012
spec_name: Mid-Week Hard-Rule Change Trigger
spec_slug: mid-week-hard-rule-change-trigger
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Automation Spec: Mid-Week Hard-Rule Change Trigger

## Overview

**Name:** Mid-Week Hard-Rule Change Trigger
**ID:** FEAT-01.SPEC-012
**Type:** Automation
**Purpose:** A new or tightened hard dietary rule triggers an immediate re-check of the current week's plan, so a meal made unsafe by a rule change never stays on the plan.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Detecting when a saved Dietary Rule change is a new or tightened hard rule (an allergy or religious rule added, or a rule's strength or allergen scope widened)
- Handing off the current week's plan for re-checking to the Dietary Rules & Allergy Safety Engine (FEAT-02)
- Firing FEAT-01.SPEC-018 (Mid-Week Rule Change Notification) once the re-check identifies a removed meal

**Non-Goals:**
- Performing the safety re-check itself (which recipes fail, which pass) -- owned entirely by FEAT-02 (Dietary Rules & Allergy Safety Engine); this automation only hands off the request and reacts to its outcome
- Offering safe alternatives for a removed meal -- owned by One-Tap Meal Swap (FEAT-04), per XBR-02
- Updating the grocery list -- owned by Shared Grocery List (FEAT-06), which recalculates automatically once the plan changes, per XBR-03

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Hard dietary rule added | FEAT-01.SPEC-006 (Dietary Rules Editor) | The saved rule is an allergy or religious rule (rule_kind, strength=hard) that did not previously exist for this member | Member reference, rule_kind, allergen (if applicable), household reference |
| Hard dietary rule tightened | FEAT-01.SPEC-006 (Dietary Rules Editor) | An existing hard rule is edited to widen its scope (e.g., a broader allergen match, an added named ingredient making more recipes unsafe) | Member reference, the rule's previous and new values, household reference |

## Processing Logic

1. Receive the saved Dietary Rule change from FEAT-01.SPEC-006, including whether it is a creation or an edit and the rule's strength.
2. Determine whether the change qualifies as "new or tightened hard": a newly created allergy or religious rule always qualifies; an edited rule qualifies only if its scope widened (never on a softening edit, since a loosened rule cannot make an already-approved meal newly unsafe).
3. If the change does not qualify (e.g., a new soft dislike, or a hard rule edited to narrow its scope), take no further action -- this automation ends here silently.
4. If the change qualifies, identify the household's current week's Weekly Plan, if one exists.
5. If a current-week plan exists, request a re-check of its remaining (not-yet-cooked) Planned Meals against the household's full, updated Dietary Rule set from the Dietary Rules & Allergy Safety Engine (FEAT-02).
6. Receive the re-check's result: a list of Planned Meals that now fail the safety check (if any).
7. For each failing Planned Meal, request its removal from the plan (per FEAT-02's ownership of safety-driven removal) and its ingredients' removal from the current Grocery List (handled automatically by FEAT-06 once the plan changes, per XBR-03).
8. If one or more meals were removed, fire FEAT-01.SPEC-018 (Mid-Week Rule Change Notification) naming each removed meal.
9. If no current-week plan exists, or the re-check finds no failing meals, end silently -- there is nothing for the organiser to be told.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| No current-week plan | Household has no active Weekly Plan for the current week | None | None | -- |
| Re-check finds no unsafe meals | A current-week plan exists but every remaining meal still passes the updated rule set | None | None -- the organiser is not told a re-check happened when nothing changed | FEAT-02 |
| One or more meals removed | The re-check finds one or more remaining meals that now violate the new/tightened rule | Affected Planned Meals removed from the current Weekly Plan; Grocery List recalculates | Organiser is told which meal(s) were removed, via FEAT-01.SPEC-018 | FEAT-02, FEAT-04 (offers alternatives), FEAT-06 (list recalculates), FEAT-01.SPEC-018 |
| Automation failure (re-check cannot complete) | The safety re-check itself cannot be completed against the current plan | No plan changes are made -- the automation fails closed, leaving the plan as it was rather than guessing | No immediate notification; the existing "checked against allergies" badges on the plan are treated as stale until a re-check succeeds, and the household's next scheduled interaction with the plan (e.g., opening it) re-attempts the check | FEAT-02, FEAT-01.SPEC-006 (the rule save itself still succeeds independently of the re-check's outcome) |

## Data Model

**Reads:** Dietary Rule -- the newly saved or edited rule, plus the household's full current rule set for the re-check; Weekly Plan and Planned Meal -- the current week's plan and its remaining meals.
**Creates:** None directly -- this automation orchestrates a hand-off; FEAT-02 and FEAT-04 own the entities they create or modify as a result.
**Updates:** Planned Meal -- status set to Removed (safety) for any meal the re-check fails, via FEAT-02's ownership.
**Deletes:** None directly.

## Business Rules

- This automation applies XBR-02 exactly: "A new or tightened hard rule takes effect on the current week immediately: remaining dinners are re-checked, any that now fail are flagged and removed, safe alternatives are offered through swap, the grocery list updates, and the organiser is told."
- Only hard rules (allergies and religious rules) trigger this automation -- a new dislike or a per-person vegetarian setting change never does, since dislikes are soft and never block a suggestion (per this feature's Dietary Rule Classification, FEAT-01.SPEC-015).
- Only the remaining (not-yet-cooked) portion of the current week's plan is re-checked -- meals already cooked are historical and are not retroactively altered.
- This automation never runs the safety determination itself; it is the trigger and hand-off point, while FEAT-02 owns the actual pass/fail logic per XBR-01.

## Edge Cases

- **Household has no current-week plan when the rule change is saved** -- The automation ends silently; there is nothing to re-check, and the new rule simply governs the next plan generated or built.
- **The tightened rule affects a meal already cooked earlier in the week** -- That meal is left untouched; only remaining, not-yet-cooked meals are in scope for re-check and removal.
- **Every remaining meal in the plan fails the re-check** -- Each is removed individually and each generates its own line in the FEAT-01.SPEC-018 notification; the household is left with an empty remainder of the week rather than a plan silently left inconsistent, and FEAT-04 offers alternatives for each open slot.
- **Concurrent trigger firing (two hard rules for different members saved within moments of each other)** -- Each triggers its own re-check independently against the household's rule set as it exists at that re-check's own start; a meal already removed by the first re-check is simply absent from the second re-check's remaining-meals list, so no duplicate removal or duplicate notification occurs for the same meal.
- **Trigger fires while a previous run is in flight** -- A second hard-rule change saved while an in-flight re-check has not yet completed queues behind it: the second re-check begins only after the first's removals (if any) are applied, ensuring it always evaluates the plan's true current state rather than a stale snapshot.
- **The dietary rule change is itself later reverted (e.g., an allergy entered in error and then removed) before the re-check completes** -- The in-flight re-check still completes against the rule set as it stood when the re-check began; if this produces a removal that the reversion would have avoided, the removed meal is not automatically restored (no restore path per this feature's Entity-Lifecycle Coverage Matrix) and the organiser must re-add it manually through FEAT-04 or FEAT-23.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-006 (Dietary Rules Editor) | Triggered by (inbound) | A new or tightened hard rule save fires this automation |
| FEAT-01.SPEC-015 (Dietary Rule Classification & Allergen Matching Rules) | References (inbound) | Defines what counts as "hard" and "tightened" |
| FEAT-01.SPEC-018 (Mid-Week Rule Change Notification) | Triggers (outbound) | Fires when one or more meals are removed |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | Triggers (outbound) | Performs the actual re-check and owns removal |
| FEAT-04 (One-Tap Meal Swap) | Affects (outbound) | Offers safe alternatives for any removed meal |
| FEAT-06 (Shared Grocery List) | Affects (outbound) | Recalculates automatically once the plan changes |

## Analytics and Success Signals

- **midweek_rule_change_recheck** (result: no_plan / no_unsafe_meals / meals_removed; removed_meal_count) -- supports success-metrics.md: "Zero Allergy Incidents" (this event is the concrete mechanism by which a tightened rule is enforced against an already-approved plan, directly serving the product's hard-zero allergy-safety promise).
- **midweek_recheck_failed** (household reference) -- N/A -- no Stage 2 metric tracks re-check failures directly, but this signal is retained because a silent failure here would otherwise undermine the Zero Allergy Incidents commitment without being observable.

## Acceptance Criteria

**FEAT-01.SPEC-012-AC-01:** Given Maya's household has a current-week plan with a Thursday dinner containing peanuts, when she adds a new peanut allergy for Jordan on FEAT-01.SPEC-006, then this automation fires, the re-check runs against the updated rule set, and Thursday's dinner is removed from the plan.

**FEAT-01.SPEC-012-AC-02:** Given a meal is removed by the re-check, then FEAT-01.SPEC-018 fires, telling Maya which meal was removed.

**FEAT-01.SPEC-012-AC-03:** Given Maya's household has no current-week plan, when she adds a new hard allergy, then this automation ends silently with no plan changes and no notification.

**FEAT-01.SPEC-012-AC-04:** Given Maya adds a new soft dislike (not a hard rule), when it is saved, then this automation does not fire at all.

**FEAT-01.SPEC-012-AC-05:** Given Maya edits an existing hard allergy to narrow its scope (a softening edit), when it is saved, then this automation does not fire.

**FEAT-01.SPEC-012-AC-06:** Given every remaining meal in the current week's plan fails the re-check after a new hard allergy is added, when the re-check completes, then each failing meal is individually removed and each is named in the resulting notification.

**FEAT-01.SPEC-012-AC-07:** Given two different hard rules for two different members are saved within moments of each other, when both trigger a re-check, then no meal is removed twice and no duplicate notification is sent for the same meal.

**FEAT-01.SPEC-012-AC-08:** Given the safety re-check cannot complete due to a processing failure, when this automation detects it, then no plan changes are made and the rule save itself still succeeds independently.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (new hard rule, tightened hard rule) | 2 |
| Outcome Paths | 4 (no plan, no unsafe meals, meals removed, automation failure) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
