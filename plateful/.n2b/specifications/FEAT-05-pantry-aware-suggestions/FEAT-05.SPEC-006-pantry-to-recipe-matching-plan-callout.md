---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-05.SPEC-006
spec_name: Pantry-to-Recipe Matching for Plan Callout
spec_slug: pantry-to-recipe-matching-plan-callout
parent_feature: FEAT-05
parent_feature_name: Pantry-Aware Suggestions
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 7
acceptance_criteria_count: 11
---

# Logic/Rule Spec: Pantry-to-Recipe Matching for Plan Callout

## Overview

**Name:** Pantry-to-Recipe Matching for Plan Callout
**ID:** FEAT-05.SPEC-006
**Type:** Logic/Rule
**Purpose:** Defines how a logged Pantry Item is matched against a candidate recipe's ingredients to produce the plan's pantry callout, and how matches weight the AI plan's dinner selection on the paid tier.
**Parent Feature:** FEAT-05 -- Pantry-Aware Suggestions
**Governed Entity:** Planned Meal (specifically its pantry_callout field, derived by this spec; Pantry Item is read, not governed, by this spec)

## Scope and Non-Goals

**In Scope:**
- The matching rule that determines which Active Pantry Items a candidate recipe's ingredients would use
- How the count and nature of matches weight the AI plan's dinner selection, when FEAT-05.SPEC-005's gate is open
- Deriving the pantry_callout field on the chosen Planned Meal (e.g., "uses the spinach and feta you already have")

**Non-Goals:**
- Whether pantry weighting applies at all this week -- gated entirely by FEAT-05.SPEC-005 (Pantry-Aware Plan Weighting Tier Gate); this spec defines the matching mechanics only, and does not run when that gate is closed
- Recording that a dinner's night has passed and surfacing the "used it up?" prompt -- handled by FEAT-05.SPEC-002 (Used-It-Up Prompt Trigger), which reads this spec's pantry_callout output as its input
- Manual planning's use of pantry data -- Manual Weekly Planning (FEAT-23) lets a household pick recipes by hand without any pantry-weighted ranking; per the Feature Breakdown Brief's Key Capabilities, pantry awareness in plan selection is scoped to the AI-generated plan (FEAT-03) on the paid tier, not to manual picks on either tier

## Governed Entity

**Entity:** Planned Meal
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| night | enum (day of week) | Not governed by this spec -- set by FEAT-03/FEAT-23 |
| meal_kind | enum (dinner, leftover lunch) | Not governed by this spec |
| recipe | reference (Recipe) | The chosen Recipe; this spec reads its ingredients to compute matches, but does not set this field |
| safety_badge | text | Not governed by this spec -- set by FEAT-02 |
| vegetarian_option | boolean | Not governed by this spec |
| cook_time, rough_cost | derived | Not governed by this spec -- carried from the Recipe |
| pantry_callout | derived (list of Pantry Item names) | Which logged Active Pantry Items this dinner's recipe uses -- computed and set entirely by this spec |
| status | enum | Not governed by this spec -- set by FEAT-03/FEAT-23/FEAT-04/FEAT-02 |
| swap_history | list | Not governed by this spec |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-03 AI Weekly Dinner Plan Generation (plan generation processing) | AI Weekly Dinner Plan Generation | Runs once per candidate recipe during generation, after FEAT-05.SPEC-005's gate check confirms weighting applies, and after FEAT-02's safety check has already filtered candidates |
| FEAT-04 One-Tap Meal Swap (alternatives computation) | One-Tap Meal Swap | Runs on each safe alternative offered for a slot, so a swap alternative's pantry callout (if any) is shown consistently with original generation |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| pantry_callout | Derived list of Pantry Item names matched against the chosen recipe's ingredients (see Defaults and Derivations); no direct user entry, so no format validation applies | Only computed when FEAT-05.SPEC-005's gate is open for the household | On plan generation and on swap-alternative computation | N/A -- this is a derived field, not user input; there is no rejection or error state for it | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Callout depends on recipe choice | recipe, pantry_callout | pantry_callout is recomputed whenever the recipe field changes (initial pick, swap) -- it is never carried over from a previous recipe in the same slot | N/A -- silent recomputation, not a validation error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View a Planned Meal's pantry_callout | Maya, Sam | Only when FEAT-05.SPEC-005's gate is open (paid tier, Active/grace standing) and the callout is non-empty for that meal | On a closed gate or an empty match, no callout text is shown on the meal -- absence of a match, not a denial |
| View a Planned Meal's pantry_callout | Jordan (older kid, limited login -- Later) | Same condition as above, consistent with this role's View access to the Weekly Plan | Same as above |
| View a Planned Meal's pantry_callout | Riley (Operator) | Only while an open Support Request exists, and only when the household's own gate is open | Same as above |
| Trigger the matching computation | System only (FEAT-03 during generation, FEAT-04 during swap-alternative computation) | Always, subject to FEAT-05.SPEC-005's gate | N/A -- no user directly triggers this computation; it runs as part of plan generation or swap, which the organiser and other adult members already have access to per FEAT-03/FEAT-04's own Authorization models |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| pantry_callout | For a candidate recipe, compare each of its ingredients (by name) against the household's Active Pantry Items, using the same exact, case-insensitive, whitespace-trimmed name match defined in FEAT-05.SPEC-003; every Active Pantry Item whose name matches one of the recipe's ingredient names is included in the callout list. An empty result means no match -- the callout is simply absent from that meal, not shown as "uses nothing." | Computed once per candidate recipe during generation, and recomputed for the chosen recipe whenever it changes (initial selection or swap) | No -- entirely system-derived; no household member edits the callout text directly |
| Selection weighting | Among candidate recipes that already pass FEAT-02's safety check and FEAT-03's schedule/budget constraints, a recipe whose pantry_callout would be non-empty is weighted more favorably than one with no matches; a higher count of matched items increases the weighting further. Weighting is a preference among otherwise-eligible candidates -- it never overrides a hard dietary rule, schedule fit, or budget constraint, and it never causes an otherwise-ideal recipe to be excluded solely for having no pantry match. | Applied only during FEAT-03's AI generation, when FEAT-05.SPEC-005's gate is open | No -- the organiser cannot manually force pantry weighting; she may always swap to a different recipe afterward through FEAT-04 |

## Business Rules

- Matching uses the same exact-name comparison rule as FEAT-05.SPEC-003 (Pantry Item Duplicate Merge), so an item logged as "spinach" matches a recipe ingredient listed as "spinach" but not one listed as "baby spinach" -- consistent with the product's free-text, no-normalization model (scope-boundaries.md, SC-11).
- Pantry weighting is a preference signal only: it never causes a recipe that fails FEAT-02's safety check, the household's schedule constraint, or its budget to be selected, and it never excludes a safe, schedule-fitting, in-budget recipe for having zero pantry matches (per FEAT-03's own Insufficient-data flow: "pantry-awareness is a refinement, not a precondition for getting a plan").
- The matching computation reads only Active Pantry Items; an item already Used/Removed at generation time is never matched, consistent with FEAT-05.SPEC-002's model that a used item is cleared rather than retained for future matching.
- A swap alternative's pantry_callout is computed the same way as original generation, so a household member sees consistent pantry information whether reviewing the original plan or a swap option.

## Edge Cases

- **A candidate recipe's ingredient list is incomplete** -- Per FEAT-02's own fail-closed rule (XBR-01), a recipe with incomplete ingredient data is already excluded from candidacy before this spec ever computes a callout for it; this spec never runs its matching against incomplete ingredient data.
- **Two Active Pantry Items would both match the same single ingredient name (a duplicate that should not exist)** -- Cannot occur: FEAT-05.SPEC-003 guarantees at most one Active entry per exact name within a household, so at most one Pantry Item can match any given ingredient name.
- **A recipe matches five or more Active Pantry Items** -- All matched items are included in the callout text; the feature places no cap on how many logged items a single dinner's callout may name.
- **The gate closes between initial generation and a same-week swap** -- FEAT-05.SPEC-005 is re-checked at swap-alternative computation time (per its own Enforced By); if the gate has closed since generation (e.g., a grace period expired mid-week), swap alternatives are computed with no pantry callout, even though the original plan may still show one from when the gate was open.
- **A Pantry Item used in this week's callout is cleared by a household member before the dinner's night arrives** -- The Planned Meal's already-computed pantry_callout text is not retroactively edited; it continues to name the item until FEAT-05.SPEC-002's night-passage check would otherwise apply. This is accepted since the callout describes what informed the plan's construction, not a live inventory count.
- **Household has zero Active Pantry Items when generation runs** -- Every candidate recipe computes an empty callout; no weighting preference is applied, and generation proceeds exactly as it would with pantry-aware weighting entirely absent (per the feature's own "Nothing logged" flow).

## Acceptance Criteria

**FEAT-05.SPEC-006-AC-01:** Given Maya's household has Active pantry items "spinach" and "feta" and the gate (FEAT-05.SPEC-005) is open, when the weekly plan generates and a safe, schedule-fitting candidate recipe lists "spinach" and "feta" among its ingredients, then that recipe's Planned Meal shows a pantry_callout naming "spinach" and "feta".

**FEAT-05.SPEC-006-AC-02:** Given Maya's household has an Active pantry item "baby spinach" and a candidate recipe lists "spinach" as an ingredient, when generation runs, then the two do not match and "baby spinach" is not included in that recipe's callout.

**FEAT-05.SPEC-006-AC-03:** Given two safe, schedule-fitting, in-budget candidate recipes exist for a night, one matching two Active pantry items and one matching none, when generation selects between them, then the matching recipe is weighted more favorably, though the non-matching recipe remains eligible.

**FEAT-05.SPEC-006-AC-04:** Given a candidate recipe would match a household's Active pantry items but fails FEAT-02's safety check for that household, when generation runs, then the recipe is excluded regardless of any pantry match, since weighting never overrides a hard safety exclusion.

**FEAT-05.SPEC-006-AC-05:** Given a candidate recipe's ingredient data is incomplete, when generation runs, then the recipe is already excluded by FEAT-02 before this spec's matching ever considers it.

**FEAT-05.SPEC-006-AC-06:** Given Maya's household has zero Active pantry items, when the weekly plan generates, then every candidate recipe computes an empty pantry_callout and selection proceeds with no pantry weighting applied.

**FEAT-05.SPEC-006-AC-07:** Given Sam swaps Thursday's dinner (FEAT-04) while the gate is still open, when the safe alternatives are computed, then each alternative's pantry_callout is computed the same way as original generation.

**FEAT-05.SPEC-006-AC-08:** Given the household's gate closes (grace period expires) between Sunday's generation and a Wednesday swap, when Sam requests swap alternatives on Wednesday, then no pantry_callout is computed for any alternative, even though Sunday's original plan may still display a callout from when the gate was open.

**FEAT-05.SPEC-006-AC-09:** Given Maya's household logged "spinach" before Sunday's generation and a Thursday dinner's callout named it, when Maya clears "spinach" on Monday, then Thursday's already-computed pantry_callout still names "spinach" until the dinner's night passes and FEAT-05.SPEC-002 runs.

**FEAT-05.SPEC-006-AC-10:** Given Jordan (older kid, limited login) views the current week's plan on a paid, Active household, when a dinner carries a pantry_callout, then Jordan can see it, consistent with this role's View access to the Weekly Plan.

**FEAT-05.SPEC-006-AC-11:** Given a candidate recipe matches five distinct Active pantry items, when generation computes its callout, then all five matched item names are included, since no cap limits the callout's item count.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
