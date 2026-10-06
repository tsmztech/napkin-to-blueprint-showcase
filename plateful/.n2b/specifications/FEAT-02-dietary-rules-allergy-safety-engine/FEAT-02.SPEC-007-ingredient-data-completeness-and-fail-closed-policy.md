---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-02.SPEC-007
spec_name: Ingredient Data Completeness & Fail-Closed Policy
spec_slug: ingredient-data-completeness-and-fail-closed-policy
parent_feature: FEAT-02
parent_feature_name: Dietary Rules & Allergy Safety Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 9
acceptance_criteria_count: 11
---

# Logic/Rule Spec: Ingredient Data Completeness & Fail-Closed Policy

## Overview

**Name:** Ingredient Data Completeness & Fail-Closed Policy
**ID:** FEAT-02.SPEC-007
**Type:** Logic/Rule
**Purpose:** Requires complete ingredient data with no partial matching shortcuts; a recipe with incomplete data is excluded rather than assumed safe.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine
**Governed Entity:** Recipe

## Scope and Non-Goals

**In Scope:**
- Defining what "complete" ingredient data means for the purpose of the safety check
- The fail-closed exclusion outcome when a recipe's ingredient data is incomplete
- Prohibiting fuzzy or partial ingredient matching as a substitute for complete data

**Non-Goals:**
- Running the ingredient-versus-rule comparison itself once data is confirmed complete -- owned by FEAT-02.SPEC-002 (Candidate Safety Check Execution)
- Authoring or editing recipe ingredient content -- owned by FEAT-08 (Recipe Library) for starter content and FEAT-10 (Recipe Import from Web Link) for imported content; this spec only validates what those features produce
- Fuzzy or partial ingredient matching as a feature -- excluded outright per the feature's own Non-Goals: any tolerance for a near-miss match directly contradicts the brief's zero-incident success criterion
- Ingredient-quantity accuracy for cost or grocery-list purposes -- owned by FEAT-06 (Shared Grocery List); this spec only cares whether quantity and unit are present, not whether they are accurate for shopping

## Governed Entity

**Entity:** Recipe
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| ingredients | list of {name, quantity, unit} | Each ingredient with quantity and unit; complete ingredient data is required to pass the safety check (at least one ingredient to save an import) |
| dietary_badges | derived | Computed per household by FEAT-02 when viewed |
| origin | enum | Starter library or imported (with source link) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-02.SPEC-002 | Candidate Safety Check Execution | Step 4 of its processing logic: completeness is verified before any ingredient-versus-rule comparison runs |
| FEAT-08 | Recipe Library (Starter Recipes) | On starter content seeding -- starter recipes are expected to be complete at ingestion, per ASMP-34 |
| FEAT-10 | Recipe Import from Web Link | On import save -- an imported recipe with incomplete extracted ingredient data is flagged for the household to complete before it can pass this check |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| ingredients | Every listed ingredient must have a non-empty name, a quantity, and a unit for the recipe to be considered complete | Always, at safety-check time | On every candidate safety check (FEAT-02.SPEC-002) | "This recipe's ingredient list is incomplete, so it can't be checked for safety yet." (shown as the plain ineligibility reason, per FEAT-02.SPEC-008) | Yes |
| ingredients | Must contain at least one ingredient | Always | On every candidate safety check | Same as above -- an empty ingredient list is treated identically to an incomplete one | Yes |
| dietary_badges | No validation beyond data type in this spec -- badges are a derived, system-computed field (recomputed per household by FEAT-02.SPEC-008) and carry no independent completeness requirement of their own; badge accuracy is only as good as this spec's ingredients determination that feeds it | Always | -- | -- | -- |
| origin | No validation beyond data type in this spec -- whether a recipe is starter-library or imported content has no bearing on the completeness policy; both origins are held to the identical completeness bar defined above | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| No cross-field rules beyond per-ingredient completeness | ingredients | Completeness is evaluated per ingredient entry independently; there is no rule combining ingredients or other Recipe fields for this policy | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Determine whether a recipe's ingredient data is complete | System (via FEAT-02.SPEC-002, FEAT-08, FEAT-10) | Always, on every check or ingestion | -- |
| Override an incomplete-data exclusion to force a recipe through the safety check | No role, ever | Never | No screen offers a "check anyway" or "trust this recipe" control; the recipe simply does not appear as a candidate until its data is completed |
| Complete or correct a recipe's ingredient data | Maya and Sam (Recipe Library: Full), for imported recipes only, per FEAT-10 | Only for recipes the household owns (imported recipes); starter recipes are read-only for households and are corrected only by the recipe/food-content data capability (FEAT-08) | Households have no edit control on starter recipes; an attempt to edit one shows "Starter recipes can't be edited. Import your own version instead." |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| completeness determination | Derived: complete when every ingredient has name, quantity, and unit and at least one ingredient exists; incomplete otherwise | Evaluated fresh on every safety check (not cached), since an edit (FEAT-10) can change completeness between checks | No -- this is a computed determination, not a stored, user-editable field |

## Business Rules

- A recipe with incomplete ingredient data is excluded rather than assumed safe -- this is the fail-closed guarantee: an unverifiable check must never default to "safe."
- No partial matching shortcut narrows what counts as complete -- a recipe cannot pass with some ingredients verified and others assumed, per the feature's Validation & Limits ("no partial matching shortcuts").
- Fuzzy or approximate ingredient matching is never used as a substitute for missing quantity/unit data -- this policy requires exact, complete data or exclusion, with no middle ground.
- XBR-19: An edited imported recipe must pass the safety check again before it can appear in any plan -- if an edit removes or corrupts ingredient completeness, the recipe fails this policy and is excluded until corrected.
- ASMP-34 (Recipe/food-content data): FEAT-08 owns starter-content ingestion and is expected to seed complete data; this spec is the validation gate that catches any starter-content gap rather than assuming vendor-supplied data is automatically trustworthy.

## Edge Cases

- **Recipe has an ingredient with a quantity but no unit (e.g., "2" with no "cups" or "grams")** -- Treated as incomplete; the recipe is excluded exactly as if the ingredient were entirely missing.
- **Recipe has an ingredient name with a quantity/unit but the name is a placeholder or empty string** -- Treated as incomplete; a non-empty name is required for every listed ingredient.
- **Starter recipe is seeded with a data gap** -- Treated identically to a household-imported recipe with incomplete data: excluded from every household's candidate pool until FEAT-08 corrects the seeded content, since starter content receives no special trust exemption from this policy.
- **An imported recipe passes the check, is later edited to add an ingredient with no unit, and is then re-checked** -- The recipe fails the completeness check on the next safety check (per XBR-19) and is excluded until the household corrects the new ingredient's unit.
- **A recipe's ingredient list is complete but extremely long** -- Length has no bearing on completeness; every ingredient, regardless of count, must individually satisfy name/quantity/unit presence.

## Acceptance Criteria

**FEAT-02.SPEC-007-AC-01:** Given a candidate recipe has every ingredient with a name, quantity, and unit, when the safety check runs, then the recipe is treated as complete and proceeds to the ingredient-versus-rule comparison.

**FEAT-02.SPEC-007-AC-02:** Given a candidate recipe has one ingredient missing its unit, when the safety check runs, then the recipe is excluded and the plain reason states the ingredient list is incomplete.

**FEAT-02.SPEC-007-AC-03:** Given a candidate recipe has zero ingredients listed, when the safety check runs, then the recipe is excluded under the same incomplete-data policy.

**FEAT-02.SPEC-007-AC-04:** Given a recipe's ingredient data cannot be fully verified for any reason, when the safety check runs, then the recipe is excluded rather than shown as safe by default.

**FEAT-02.SPEC-007-AC-05:** Given a starter recipe is seeded with an incomplete ingredient entry, when any household's candidate check considers it, then it is excluded the same as an incomplete household-imported recipe.

**FEAT-02.SPEC-007-AC-06:** Given Sam attempts to edit a starter recipe's ingredient list, when he looks for an edit control, then none is available and an attempt shows "Starter recipes can't be edited. Import your own version instead."

**FEAT-02.SPEC-007-AC-07:** Given Maya edits her household's imported recipe to add a new ingredient with a name and quantity but no unit, when the recipe is next considered as a candidate, then it is excluded until the unit is added, per XBR-19.

**FEAT-02.SPEC-007-AC-08:** Given a household member views an excluded recipe with incomplete data, when they look for a way to proceed anyway, then no override control exists on any screen.

**FEAT-02.SPEC-007-AC-09:** Given a recipe's ingredient data was incomplete at one check and is later corrected by the household (for an imported recipe), when the next safety check runs, then completeness is re-evaluated fresh and the recipe may pass if its ingredients now satisfy the rule comparison.

**FEAT-02.SPEC-007-AC-10:** Given a recipe has a very long ingredient list where every entry is individually complete, when the safety check runs, then the recipe is treated as complete regardless of ingredient count.

**FEAT-02.SPEC-007-AC-11:** Given no fuzzy-matching mechanism exists anywhere in the safety check, when a recipe's ingredient name is ambiguous or a near-miss to an allergen term, then the check either matches it fully (per FEAT-02.SPEC-002's compound-ingredient handling) or treats the data as incomplete -- it never partially matches to assume safety.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |
