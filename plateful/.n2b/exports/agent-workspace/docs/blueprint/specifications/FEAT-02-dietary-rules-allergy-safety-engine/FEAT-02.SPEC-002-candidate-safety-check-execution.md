---
document_type: spec
spec_type: automation
spec_id: FEAT-02.SPEC-002
spec_name: Candidate Safety Check Execution
spec_slug: candidate-safety-check-execution
parent_feature: FEAT-02
parent_feature_name: Dietary Rules & Allergy Safety Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Automation Spec: Candidate Safety Check Execution

## Overview

**Name:** Candidate Safety Check Execution
**ID:** FEAT-02.SPEC-002
**Type:** Automation
**Purpose:** Checks a candidate recipe's ingredients against every household member's hard rules before it can ever be shown, on every path onto the plan.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- Running the hard-rule safety check on every candidate recipe from every path onto a household's plan (AI generation, manual pick, swap, vote, history re-use)
- Attaching the "checked against allergies" safety badge to a recipe that passes
- Excluding a recipe from the household's candidate pool when it fails the check for any member, or when its ingredient data is incomplete

**Non-Goals:**
- Classifying which rule kinds are hard versus soft -- owned by FEAT-02.SPEC-006 (Rule Strength & Blocking Policy), which this automation reads
- Defining the fail-closed behavior for incomplete ingredient data in detail -- owned by FEAT-02.SPEC-007 (Ingredient Data Completeness & Fail-Closed Policy), which this automation applies
- Determining the badge's exact wording and disclaimer text, or the ineligibility reason's wording -- owned by FEAT-02.SPEC-008 (Safety Badge & Disclaimer Display Rule)
- Deciding whether a recipe under an open safety report may re-enter the pool -- owned by FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy), which this automation consults before running

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| AI plan generation proposes a candidate recipe | FEAT-03 (AI Weekly Dinner Plan Generation) | Fires once per candidate recipe considered during generation | Candidate Recipe (ingredients, cook_time), the household's full set of Dietary Rules |
| Adult searches or picks a recipe manually | FEAT-23 (Manual Weekly Planning) | Fires when a recipe is displayed in search results or attempted for placement | Candidate Recipe, the household's Dietary Rules |
| Adult requests swap alternatives | FEAT-04 (One-Tap Meal Swap) | Fires for each candidate alternative considered for the slot | Candidate Recipe, the household's Dietary Rules |
| A dinner-voting round is opened | FEAT-17 (Older-Kid Dinner Voting, Later) | Fires for each option before it can be included in the round | Candidate Recipe, the household's Dietary Rules |
| Household re-uses a past week as a starting point | FEAT-19 (Weekly Plan History) | Fires for every meal in the copied week before it returns to the plan | Candidate Recipe (as it exists now, not as it existed when originally checked), the household's current Dietary Rules |
| A hard rule is added or tightened for the current week | FEAT-02.SPEC-003 (Mid-Week Rule Change Re-Check) | Fires once per remaining dinner already on the plan, re-running this same check | The Planned Meal's Recipe, the household's updated Dietary Rules |
| An imported recipe is edited or re-imported | FEAT-10 (Recipe Import from Web Link) | Fires per XBR-19 before the edited recipe can appear in any plan again | The edited Recipe's current ingredient list, the household's Dietary Rules |

## Processing Logic

1. Receive the candidate Recipe and the requesting Household's identity from the triggering spec.
2. Consult FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy): if this Recipe is under an open safety report for this Household, exclude it immediately and skip the remaining steps.
3. Read the Recipe's ingredient list (quantity and unit per ingredient).
4. Verify the ingredient data is complete per FEAT-02.SPEC-007 (Ingredient Data Completeness & Fail-Closed Policy). If incomplete, exclude the Recipe and emit safety_check_data_incomplete; skip the remaining steps.
5. Read every Household Member's Dietary Rules for this Household.
6. For each Member, compare every ingredient against the Member's hard rules (per FEAT-02.SPEC-006: allergy and religious-rule entries, and the per-person vegetarian setting where it applies) using the full ingredient text -- no partial or fuzzy matching.
7. If any ingredient matches any Member's allergen (from the standard allergen list or a named extra ingredient) or violates a religious rule, mark the Recipe as failing for that Member.
8. If the Recipe is a shared dinner and any vegetarian Member's vegetarian setting is not met by the base recipe, check whether the Recipe carries a vegetarian_option variant; if it does, the base recipe is not excluded on vegetarian grounds alone -- the variant satisfies that Member (see Business Rules).
9. If the Recipe fails for any Member on an allergy or religious-rule basis (vegetarian handled per Step 8), exclude the Recipe from the candidate pool for this Household entirely.
10. If the Recipe passes for every Member, attach the safety_badge (per FEAT-02.SPEC-008) to the candidate before it can be shown or placed.
11. Return the pass/exclude determination, plus the ineligibility reason (per FEAT-02.SPEC-008's plain-phrasing pattern) when excluded, to the triggering spec.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Recipe passes | No hard-rule failure for any Member and ingredient data is complete | Planned Meal's safety_badge field is set when the recipe is placed | Recipe displays with the "checked against allergies" badge and disclaimer | FEAT-03, FEAT-04, FEAT-08, FEAT-10, FEAT-17, FEAT-19, FEAT-23, FEAT-02.SPEC-008 |
| Recipe excluded -- hard-rule failure | The recipe violates an allergy or religious rule for at least one Member | None -- the recipe never enters the candidate pool shown to the household | Recipe does not appear in results; where a specific recipe was searched for directly, the plain ineligibility reason is shown ("contains {allergen} -- not safe for {member}") | FEAT-03, FEAT-04, FEAT-08, FEAT-17, FEAT-19, FEAT-23, FEAT-02.SPEC-008 |
| Recipe excluded -- incomplete data | The Recipe's ingredient data cannot be fully verified | None | Recipe does not appear in results; ineligibility reason states the data is incomplete, per FEAT-02.SPEC-007/008 | FEAT-02.SPEC-007, FEAT-08, FEAT-03, FEAT-23 |
| Recipe excluded -- under open safety report | The Recipe is currently excluded per FEAT-02.SPEC-009 | None | Recipe does not appear in results for this household while the report is open | FEAT-02.SPEC-009, FEAT-02.SPEC-004 |
| Shared-meal vegetarian satisfied via variant | The base recipe does not meet a vegetarian Member's setting but a vegetarian_option variant exists | None beyond badge attachment | The plan shows the vegetarian variant is available for the vegetarian household member | FEAT-02.SPEC-006, FEAT-03, FEAT-23 |
| Check cannot complete (processing failure) | The check itself cannot run to completion for reasons other than incomplete ingredient data | None -- fails closed | Recipe is excluded, treated identically to the "incomplete data" outcome, since an unverifiable check must never default to "safe" | FEAT-02.SPEC-007, FEAT-03, FEAT-08, FEAT-23 |

## Data Model

**Reads:** Dietary Rule -- member, rule_kind, strength, allergen, for every household member. Recipe -- ingredients (quantity and unit), dietary_badges, for the candidate under check.
**Creates:** None.
**Updates:** Planned Meal -- safety_badge (set when a candidate passes and is placed onto the plan). Recipe -- dietary_badges (computed per household when viewed, not persisted per household).
**Deletes:** None.

## Business Rules

- XBR-01: Every path onto the plan -- AI generation, manual picks, swaps and swap/pick suggestions, voting options, and re-used past weeks -- passes this same app-enforced check before anyone sees it; the check fails closed, and every shown meal carries the badge and disclaimer.
- Allergy and religious-rule failures always exclude a recipe entirely for the household -- there is no partial or per-member visibility of an unsafe recipe (per FEAT-02.SPEC-006).
- A shared meal that is not inherently vegetarian may still pass for a household with a vegetarian member if the recipe carries a vegetarian_option variant; the base recipe is not treated as failing on vegetarian grounds alone, per FEAT-02.SPEC-006.
- Dislikes (soft rules) never factor into this check -- they influence selection ranking elsewhere (FEAT-03, FEAT-12) but cannot exclude a recipe here, per FEAT-02.SPEC-006.
- The check runs synchronously and completes before a candidate is ever shown to any household member, on any path -- there is no state where an unchecked recipe is visible.
- This automation is the single owner of the pass/exclude determination; no other spec in the product performs its own allergy or religious-rule matching.

## Edge Cases

- **Recipe has zero ingredients recorded** -- Treated as incomplete ingredient data (FEAT-02.SPEC-007); excluded, never assumed safe by default.
- **Member has no dietary rules recorded at all** -- Absence of rules is not the same as "no restrictions" being explicitly stated; per FEAT-01's setup flow every member has at least an explicit "no restrictions" statement recorded, so this automation always has a rule set (possibly empty by explicit statement) to check against.
- **Two members share the same allergen but different named extra ingredients** -- Each member's rule set is checked independently against the full ingredient list; a match against either member's allergen or extra ingredient excludes the recipe for the whole household.
- **Recipe passes for AI generation but a member's hard rule is added moments later, before the plan is shown** -- The check reflects the Dietary Rule data as read at the moment this automation runs; a rule change after the check completes but before display is handled by FEAT-02.SPEC-003 (Mid-Week Rule Change Re-Check) once the rule is saved, not by this automation re-running speculatively.
- **Concurrent trigger firing (two paths check the same recipe for the same household at effectively the same time, e.g., AI generation and a manual search)** -- Each run reads the same Dietary Rule and Recipe data and reaches the same determination independently; both complete without blocking each other, since the check is read-only against Dietary Rule and Recipe and its only write (safety_badge) is scoped to the Planned Meal the calling spec is placing.
- **Trigger fires while a previous run for the same recipe and household is in flight** -- Each run is independent and stateless; a second run for the same recipe does not need to wait on the first, since neither run mutates shared state that the other reads mid-check.
- **Ingredient text contains an ambiguous or compound term (e.g., "mixed nuts")** -- Treated as matching every allergen the compound term could reasonably contain (e.g., matches a peanut, tree-nut, or specific named-nut allergy); no partial matching shortcut narrows a compound ingredient to a subset of its possible allergens.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03 (AI Weekly Dinner Plan Generation) | Triggered by (inbound) | Every AI-proposed candidate is checked before inclusion |
| FEAT-23 (Manual Weekly Planning) | Triggered by (inbound) | Every manually searched or picked recipe is checked |
| FEAT-04 (One-Tap Meal Swap) | Triggered by (inbound) | Every swap alternative is checked before it is offered |
| FEAT-17 (Older-Kid Dinner Voting) | Triggered by (inbound) | Every voting option is checked before a round opens |
| FEAT-19 (Weekly Plan History) | Triggered by (inbound) | A re-used past week's meals are re-checked before returning to the plan |
| FEAT-02.SPEC-003 (Mid-Week Rule Change Re-Check) | Triggered by (inbound) | Re-runs this check against every remaining dinner when a hard rule changes |
| FEAT-01 (Household Setup & Member Profiles) | Reads (outbound) | Source of every household member's Dietary Rule data |
| FEAT-02.SPEC-006 (Rule Strength & Blocking Policy) | References (outbound) | Classifies which rules are hard filters versus soft |
| FEAT-02.SPEC-007 (Ingredient Data Completeness & Fail-Closed Policy) | References (outbound) | Governs the incomplete-data exclusion path |
| FEAT-02.SPEC-008 (Safety Badge & Disclaimer Display Rule) | Affects (outbound) | Supplies the badge and ineligibility-reason wording this automation attaches |
| FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy) | References (outbound) | Consulted first to exclude recipes under an open report |
| FEAT-08 (Recipe Library) / FEAT-10 (Recipe Import) | Affects (outbound) | Source of the candidate Recipe data this automation checks |

## Analytics and Success Signals

- **safety_check_run** (household reference, recipe reference, trigger source) -- supports success-metrics.md: "Zero Allergy Incidents"
- **safety_check_failed** (allergen or rule matched, member reference, trigger source) -- supports success-metrics.md: "Zero Allergy Incidents"
- **safety_badge_shown** (recipe reference, trigger source) -- supports success-metrics.md: "Zero Allergy Incidents"
- **safety_check_data_incomplete** (recipe reference) -- supports success-metrics.md: "Zero Allergy Incidents"

## Acceptance Criteria

**FEAT-02.SPEC-002-AC-01:** Given Maya's household has a member with a peanut allergy, when the AI plan generation considers a recipe containing peanuts, then this automation excludes the recipe from the candidate pool and it never appears in the generated plan.

**FEAT-02.SPEC-002-AC-02:** Given a candidate recipe contains no ingredient that violates any household member's hard rules and has complete ingredient data, when the check runs, then the recipe passes and carries the "checked against allergies" badge once placed.

**FEAT-02.SPEC-002-AC-03:** Given Sam searches the Recipe Library for a dinner during Manual Weekly Planning, when a recipe in the results would violate his child's allergy, then that recipe is excluded from the results and shows the plain ineligibility reason.

**FEAT-02.SPEC-002-AC-04:** Given Maya requests swap alternatives for Friday's dinner, when the alternatives list is generated, then every alternative shown has already passed this safety check.

**FEAT-02.SPEC-002-AC-05:** Given a household re-uses a past week from Weekly Plan History, when the copied week's meals are re-checked, then any meal that no longer passes (e.g., a rule added since it was last checked) is excluded from the copy.

**FEAT-02.SPEC-002-AC-06:** Given a candidate recipe has an incomplete ingredient list (missing quantity or unit on one item), when the check runs, then the recipe is excluded and safety_check_data_incomplete is emitted, regardless of whether the visible ingredients would otherwise pass.

**FEAT-02.SPEC-002-AC-07:** Given a shared dinner recipe is not inherently vegetarian but carries a vegetarian_option variant, when a household with one vegetarian member and one non-vegetarian member is checked, then the recipe passes and the vegetarian variant is made available rather than the recipe being excluded.

**FEAT-02.SPEC-002-AC-08:** Given a household member has a recorded dislike of mushrooms (a soft rule) and no allergy to them, when a recipe containing mushrooms is checked, then the recipe passes this safety check regardless of the dislike.

**FEAT-02.SPEC-002-AC-09:** Given a recipe is currently under an open safety report for Maya's household (FEAT-02.SPEC-009), when any path attempts to check that recipe again, then it is excluded without re-running the ingredient comparison.

**FEAT-02.SPEC-002-AC-10:** Given Maya adds a new allergy for one of her children (FEAT-02.SPEC-003 triggers this automation for each remaining dinner), when a remaining dinner's recipe now contains that allergen, then this automation excludes it from the current week just as it would for a new candidate.

**FEAT-02.SPEC-002-AC-11:** Given an older kid's dinner-voting round is being opened (Later), when each candidate option is checked, then only options that pass this automation's check are included in the round.

**FEAT-02.SPEC-002-AC-12:** Given two different features check the same candidate recipe for the same household at effectively the same time, when both checks run, then both complete independently and reach the same pass/exclude determination without blocking each other.

**FEAT-02.SPEC-002-AC-13:** Given an imported recipe is edited to add a new ingredient (FEAT-10, per XBR-19), when the edited recipe is next considered as a candidate, then this automation re-checks it against every household's rules before it can appear in any plan again.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 7 | 7 |
| Outcome Paths | 6 | 6 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |
