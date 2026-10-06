---
document_type: spec
spec_type: automation
spec_id: FEAT-02.SPEC-003
spec_name: Mid-Week Rule Change Re-Check
spec_slug: mid-week-rule-change-re-check
parent_feature: FEAT-02
parent_feature_name: Dietary Rules & Allergy Safety Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Mid-Week Rule Change Re-Check

## Overview

**Name:** Mid-Week Rule Change Re-Check
**ID:** FEAT-02.SPEC-003
**Type:** Automation
**Purpose:** Re-verifies every remaining dinner in the current week the moment a hard rule is added or tightened, so a newly unsafe meal never stays on the plan.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- Detecting when a new or tightened hard rule (allergy or religious rule) is saved for any household member
- Re-running the candidate safety check against every remaining dinner in the current week's plan
- Removing any dinner that now fails, updating the grocery list, offering safe alternatives, and telling the organiser

**Non-Goals:**
- Capturing or saving the dietary rule change itself -- owned by FEAT-01 (Household Setup & Member Profiles), whose save action is this automation's trigger
- Running the actual ingredient-versus-rule comparison logic -- delegated to FEAT-02.SPEC-002 (Candidate Safety Check Execution), which this automation invokes per remaining dinner
- Re-checking a softened or removed rule -- excluded per the feature's own scope: a rule becoming less restrictive (removing an allergy, loosening a religious rule) cannot make an already-approved meal unsafe, so no re-check is needed; only a new or tightened hard rule triggers this automation, per the Brief's Side-Effect Inventory
- Selecting the specific replacement recipe for a removed slot -- owned by FEAT-04 (One-Tap Meal Swap), which this automation opens on the household's behalf

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A hard rule (allergy or religious rule) is added for a household member | FEAT-01 (Household Setup & Member Profiles) | Fires when Maya saves a new Dietary Rule with strength = hard | The affected Member, the new Dietary Rule (rule_kind, allergen), the Household's current Weekly Plan |
| A hard rule is tightened for a household member | FEAT-01 (Household Setup & Member Profiles) | Fires when Maya edits an existing hard Dietary Rule to add a named extra ingredient or otherwise narrow what is safe | The affected Member, the updated Dietary Rule, the Household's current Weekly Plan |

## Processing Logic

1. Receive the Household reference and the changed Dietary Rule (added or tightened, strength = hard) from FEAT-01.
2. Read the Household's current Weekly Plan and identify every remaining dinner Planned Meal (nights that have not yet passed).
3. For each remaining dinner, invoke FEAT-02.SPEC-002 (Candidate Safety Check Execution) against that dinner's Recipe, using the Household's now-updated full Dietary Rule set.
4. For each dinner that still passes, leave it unchanged on the plan.
5. For each dinner that now fails, set its Planned Meal status to Removed (safety) and append the prior recipe to swap_history.
6. For every removed dinner, trigger the grocery list recalculation (FEAT-06) to drop that dinner's ingredients.
7. For every removed dinner, open a swap for the now-empty slot (FEAT-04) so a safe alternative can be selected.
8. If one or more dinners were removed, trigger FEAT-02.SPEC-012 (Safety Concern Organiser Alert) to tell the organiser which dinners were removed and why (rule change, not a safety-concern report).
9. If no remaining dinner fails, complete silently -- the rule change is saved with no visible plan disruption.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| No dinners affected | Every remaining dinner still passes the re-check | None | None -- the rule save completes normally with no plan disruption message | FEAT-01 |
| One or more dinners removed | At least one remaining dinner now fails the re-check | Affected Planned Meal(s) set to Removed (safety); swap_history appended; grocery list recalculated | Affected slot(s) show as empty with the plain reason (FEAT-02.SPEC-008); a swap is opened offering safe alternatives; the organiser receives the removal alert (FEAT-02.SPEC-012) | FEAT-04, FEAT-06, FEAT-02.SPEC-012, FEAT-02.SPEC-008 |
| Re-check itself cannot complete for a dinner | The safety check (FEAT-02.SPEC-002) cannot verify a dinner's recipe (e.g., incomplete data surfaces only now) | That dinner is treated as failing and removed -- fails closed, consistent with FEAT-02.SPEC-007 | Same as "one or more dinners removed" for that slot | FEAT-04, FEAT-06, FEAT-02.SPEC-012, FEAT-02.SPEC-007 |

## Data Model

**Reads:** Dietary Rule -- the changed rule and every other member's current rules. Weekly Plan -- the current week's remaining Planned Meals. Recipe -- ingredients for each remaining dinner, via FEAT-02.SPEC-002.
**Creates:** None directly (FEAT-02.SPEC-004's Support Request mechanism does not apply here -- this is a rule-change removal, not a reported concern).
**Updates:** Planned Meal -- status (set to Removed (safety) for failing dinners), swap_history (prior recipe appended).
**Deletes:** None -- removal is a status change, not a hard delete, matching the Planned Meal lifecycle in the dependency map.

## Business Rules

- XBR-02: A new or tightened hard rule takes effect on the current week immediately: remaining dinners are re-checked, any that now fail are flagged and removed, safe alternatives are offered through swap, the grocery list updates, and the organiser is told.
- Only remaining (not-yet-cooked) dinners are re-checked -- a dinner already marked Cooked is history and is never retroactively altered.
- A removal from this automation uses the same Removed (safety) status and mechanic as a safety-concern removal (FEAT-02.SPEC-004); the two share one lifecycle transition on Planned Meal.
- This automation never touches Dietary Rule data itself -- it only reacts to a change FEAT-01 has already saved.
- A rule change that only loosens or removes a restriction never triggers this automation, since it cannot newly endanger an already-safe meal.

## Edge Cases

- **Two hard rules are added in quick succession for different members** -- Each triggers its own re-check run; the second run reads the Weekly Plan as it stands after the first run's removals, so a dinner already removed by the first change is not re-processed by the second.
- **The changed member has no remaining dinners left in the week (all already cooked)** -- The re-check runs, finds no remaining dinners to evaluate, and completes with the "no dinners affected" outcome.
- **A dinner already has an open safety report when the rule change re-check runs** -- The dinner is already excluded from candidacy per FEAT-02.SPEC-009; the re-check still evaluates its current placement on the plan for the new rule and removes it under the rule-change path if it also fails the new rule, without creating a duplicate Support Request.
- **Concurrent trigger firing (two members' hard rules change at effectively the same time, e.g., Maya edits two profiles back-to-back)** -- Each rule save fires its own re-check run against the plan state at that moment; the second run reflects any removals the first run already made, so no dinner is independently removed twice for the same reason, though a dinner could be removed once by each run if it fails both new rules (recorded once, since the Planned Meal's status only needs to be Removed (safety) regardless of how many rules it failed).
- **Trigger fires while a previous re-check run is still in flight** -- A second rule-change trigger for the same household waits for the first run to finish reading and updating the Weekly Plan before it begins its own pass, so the two runs never evaluate the plan against inconsistent intermediate states.
- **Household has no remaining dinners planned at all this week (a free-tier household mid-way through manual planning with empty nights)** -- The re-check finds zero remaining dinners to evaluate and completes with no visible effect; empty nights are not Planned Meals and are not in scope for removal.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01 (Household Setup & Member Profiles) | Triggered by (inbound) | A hard-rule save fires this automation |
| FEAT-02.SPEC-002 (Candidate Safety Check Execution) | Triggers (outbound) | Re-runs the same check against every remaining dinner |
| FEAT-04 (One-Tap Meal Swap) | Affects (outbound) | A swap is opened for every emptied slot |
| FEAT-06 (Shared Grocery List) | Affects (outbound) | The list is recalculated to drop removed dinners' ingredients |
| FEAT-02.SPEC-012 (Safety Concern Organiser Alert) | Triggers (outbound) | Tells the organiser which dinners were removed and why |
| FEAT-02.SPEC-008 (Safety Badge & Disclaimer Display Rule) | References (outbound) | Governs the plain reason shown on the now-empty slot |
| FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy) | References (outbound) | A dinner already excluded under an open report is not double-counted as a new removal reason |

## Analytics and Success Signals

- **midweek_rule_change_recheck_run** (household reference, remaining-dinner count evaluated) -- supports success-metrics.md: "Zero Allergy Incidents"
- **midweek_rule_change_meal_removed** (recipe reference, member/allergen matched) -- supports success-metrics.md: "Zero Allergy Incidents"
- **midweek_rule_change_no_impact** (household reference) -- N/A -- no Stage 2 metric measures rule changes with zero plan impact; retained so the re-check's actual disruption rate is observable

## Acceptance Criteria

**FEAT-02.SPEC-003-AC-01:** Given Maya adds a new peanut allergy for one of her children mid-week, when the save completes, then this automation re-checks every remaining dinner in the current week's plan.

**FEAT-02.SPEC-003-AC-02:** Given a remaining Wednesday dinner contains peanuts and Maya just added a peanut allergy for her child, when the re-check runs, then Wednesday's dinner is set to Removed (safety), the grocery list drops its ingredients, and a swap opens offering safe alternatives.

**FEAT-02.SPEC-003-AC-03:** Given none of the remaining dinners contain the newly restricted allergen, when the re-check completes, then no dinner is removed and no organiser alert fires.

**FEAT-02.SPEC-003-AC-04:** Given one or more dinners are removed by this automation, when the removal completes, then Maya (the organiser) receives the safety concern organiser alert (FEAT-02.SPEC-012) naming the removed meal and the rule-change reason.

**FEAT-02.SPEC-003-AC-05:** Given Maya tightens an existing halal rule by naming a specific extra ingredient to avoid, when the save completes, then this automation re-checks remaining dinners against the updated rule, not just the original rule_kind.

**FEAT-02.SPEC-003-AC-06:** Given Maya removes (rather than adds) a hard allergy rule, when the save completes, then this automation does not fire, since a loosened rule cannot newly endanger a meal.

**FEAT-02.SPEC-003-AC-07:** Given a dinner already marked Cooked exists in the current week when a hard rule changes, when the re-check runs, then the Cooked dinner is not evaluated or altered.

**FEAT-02.SPEC-003-AC-08:** Given a remaining dinner's recipe now has incomplete ingredient data discovered only at re-check time, when the re-check runs, then that dinner is removed under the fail-closed policy (FEAT-02.SPEC-007), the same as an explicit rule violation.

**FEAT-02.SPEC-003-AC-09:** Given two hard-rule changes are saved back-to-back for different members, when both re-checks run, then a dinner removed by the first change is not independently reprocessed as a new removal by the second.

**FEAT-02.SPEC-003-AC-10:** Given a household has no remaining dinners left this week when a hard rule changes, when the re-check runs, then it completes with no plan impact and no organiser alert.

**FEAT-02.SPEC-003-AC-11:** Given a second hard-rule change for the same household fires while an earlier re-check run is still in progress, when the second trigger arrives, then it waits for the first run to finish before evaluating the plan, so neither run acts on an inconsistent intermediate state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
