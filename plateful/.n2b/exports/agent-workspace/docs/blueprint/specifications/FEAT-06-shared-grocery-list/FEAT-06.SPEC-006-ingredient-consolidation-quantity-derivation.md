---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-06.SPEC-006
spec_name: Ingredient Consolidation & Quantity Derivation
spec_slug: ingredient-consolidation-quantity-derivation
parent_feature: FEAT-06
parent_feature_name: Shared Grocery List
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 8
acceptance_criteria_count: 10
---

# Logic/Rule Spec: Ingredient Consolidation & Quantity Derivation

## Overview

**Name:** Ingredient Consolidation & Quantity Derivation
**ID:** FEAT-06.SPEC-006
**Type:** Logic/Rule
**Purpose:** Derives the plan-derived portion of the grocery list by combining the same ingredient across dinners into one line with a household-sized combined quantity, minus logged pantry items.
**Parent Feature:** FEAT-06 -- Shared Grocery List
**Governed Entity:** Grocery List Item (plan-derived subset)

## Scope and Non-Goals

**In Scope:**
- Matching the same ingredient across multiple Planned Meals and combining them into one line
- Converting and summing quantities into the household's configured unit system
- Assigning each combined line's aisle from the household's configured aisle categories
- Excluding a combined ingredient entirely when a matching Active Pantry Item exists

**Non-Goals:**
- Manual item validation and duplicate merge -- owned by FEAT-06.SPEC-007; this spec governs plan-derived lines only
- Deciding when recalculation runs -- owned by FEAT-06.SPEC-002, which invokes this spec's computation
- Partial pantry-quantity subtraction -- excluded because Pantry Item carries no quantity field (per the Feature Dependency Map); pantry exclusion is necessarily binary, not a partial-amount reduction
- Access to view or act on the resulting lines -- owned by FEAT-06.SPEC-009

## Governed Entity

**Entity:** Grocery List Item (plan-derived subset)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| ingredient_name | text | The combined ingredient's name, copied from the contributing Recipes' ingredient lists |
| quantity_and_unit | text | The combined, household-sized quantity, in the household's configured unit system |
| aisle | text | The aisle category this ingredient is grouped under, from the household's configured `aisle_grouping` (FEAT-16) |
| origin | enum | Always "plan-derived" for lines this spec produces |
| ticked | boolean | Whether the line has been ticked; preserved or reset per FEAT-06.SPEC-002's recalculation rules, not set by this spec |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-002 | Grocery List Generation & Recalculation | Invokes this spec's computation on every generation and recalculation trigger |
| FEAT-06.SPEC-001 | Grocery List | Displays the computed lines this spec produces; does not recompute them itself |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| ingredient_name | No validation beyond data type -- copied from Recipe data, not user-entered for plan-derived lines | Always | -- | -- | -- |
| quantity_and_unit | No validation beyond data type -- system-computed by this spec's derivation logic (see Defaults and Derivations) | Always | -- | -- | -- |
| aisle | No validation beyond data type -- system-assigned by lookup against the household's configured aisle categories | Always | -- | -- | -- |
| origin | No validation beyond data type -- always set to "plan-derived" by this spec | Always | -- | -- | -- |
| ticked | No validation beyond data type -- managed by FEAT-06.SPEC-002's recalculation rules, not set by this spec | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Same-ingredient combination | ingredient_name, quantity_and_unit | Grocery List Items derived from two or more Planned Meals combine into a single line when their `ingredient_name` matches (case-insensitive, trimmed) and the underlying ingredient and unit are literally the same (a vegetarian-variant substitution is treated as a distinct ingredient, not combined with the standard version) | N/A -- not a validation failure, a derivation rule (see Defaults and Derivations) |
| Pantry exclusion | ingredient_name (Grocery List Item) matched against item_name (Pantry Item) | If an Active Pantry Item's `item_name` matches a combined ingredient's `ingredient_name` (case-insensitive, trimmed), that ingredient's entire combined line is excluded from the plan-derived list (XBR-04). The exclusion is binary -- Pantry Item carries no quantity, so partial coverage is not modeled | N/A -- not a validation failure, a derivation rule |

## Authorization Rules

{This spec computes list content; it introduces no authorization surface of its own beyond the resulting lines' visibility, which is fully governed by FEAT-06.SPEC-009. The row below states only that computation runs regardless of who is viewing.}

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View a computed plan-derived line | Maya, Sam, Jordan (older kid, limited login), Riley (Operator, support, while an open Support Request exists) | Always, once the viewer already has Grocery List access per FEAT-06.SPEC-009 | -- (full detail governed by FEAT-06.SPEC-009, not restated here) |
| View a computed plan-derived line | Jordan (young kid profile, no login) | Never | This profile has no login and cannot reach the screen at all |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| ingredient_name | Copied from the Recipe's ingredient list for each contributing Planned Meal | On every recalculation | No |
| quantity_and_unit | Sum of each contributing dinner's required amount for the same ingredient, converted into the household's configured `unit_system` (FEAT-16) before summing | On every recalculation | No -- always recomputed; a member's direct quantity edit on FEAT-06.SPEC-001 is preserved by FEAT-06.SPEC-002 only as long as the line's contributing dinners are unchanged (see FEAT-06.SPEC-002 Business Rules) |
| aisle | Looked up from the household's configured `aisle_grouping` (FEAT-16) by matching the ingredient against its known category; falls back to a household catch-all "Other" aisle when no match is found | On every recalculation | No |
| origin | Set to "plan-derived" | On creation | No |
| ticked | Not set by this spec -- preserved or reset by FEAT-06.SPEC-002's recalculation rules | N/A | N/A |

## Business Rules

- XBR-03: this computation runs on every plan-affecting change so the list never falls out of sync with the plan.
- XBR-04: pantry exclusion is applied as part of this computation, not as a separate downstream filter.
- XBR-11: the household's currently configured unit system and aisle categories (FEAT-16) apply at computation time; a later change to those settings is reflected on the next recalculation, not retroactively rewritten.

## Edge Cases

- **A recipe is safety-removed but shares an ingredient with a dinner still on the plan** -- The combined line reduces to reflect only the remaining dinner's need rather than being removed entirely.
- **An ingredient name is a near-miss, not an exact match (e.g., "tomatoes" vs. "tomato")** -- Matching is exact (case-insensitive, trimmed); a near-miss does not combine and both spellings would appear as distinct lines if both occurred, a limitation the household resolves by marking either line "already have it" or removing it directly.
- **A shared meal's vegetarian variant and its standard version both appear in the same week** -- Their ingredients are not combined, since the underlying ingredient differs between the two versions.
- **Exactly two dinners need the same ingredient in different units (e.g., cups vs. ounces)** -- Both are converted to the household's single configured unit system before summing into one line.
- **A Pantry Item's `item_name` exactly matches a combined ingredient, but the household still needs more of it than they have on hand** -- The line is still excluded entirely, since Pantry Item carries no quantity and the exclusion is binary by design; the household re-adds it manually if they need more.
- **Only one dinner needs an ingredient (no combination case)** -- The line still passes through this computation as a single-source line, aisle-assigned and pantry-checked the same as any combined line.

## Acceptance Criteria

**FEAT-06.SPEC-006-AC-01:** Given two dinners this week both use "chicken breast," when the plan-derived list is computed, then a single "chicken breast" line appears with the summed quantity from both dinners.

**FEAT-06.SPEC-006-AC-02:** Given Maya has logged "spinach" as an Active Pantry Item, when the plan-derived list is computed and a dinner needs spinach, then no "spinach" line appears on the list.

**FEAT-06.SPEC-006-AC-03:** Given two dinners need the same ingredient in different units, when the combined line is computed, then both amounts are converted to the household's configured unit system before being summed.

**FEAT-06.SPEC-006-AC-04:** Given a shared meal has both a standard and a vegetarian-variant version this week, when the plan-derived list is computed, then their distinct ingredients are not combined into one line.

**FEAT-06.SPEC-006-AC-05:** Given only one dinner this week needs "basil," when the plan-derived list is computed, then a single-source "basil" line appears, aisle-assigned and pantry-checked the same as any other line.

**FEAT-06.SPEC-006-AC-06:** Given a safety-concern removal drops a recipe that shared "onions" with another dinner still on the plan, when recalculation runs, then the "onions" line is reduced to reflect only the remaining dinner's need, not removed.

**FEAT-06.SPEC-006-AC-07:** Given the household's aisle categories don't recognize a given ingredient, when its line is computed, then it is grouped under the household's catch-all "Other" aisle.

**FEAT-06.SPEC-006-AC-08:** Given an ingredient is logged in the pantry as "tomato" and the plan needs "tomatoes," when the exclusion check runs, then the near-miss does not match and the ingredient still appears on the list.

**FEAT-06.SPEC-006-AC-09:** Given Riley (Operator) views a household's list during an open Support Request, when a computed plan-derived line renders, then it displays the same computed line Maya and Sam see, read-only.

**FEAT-06.SPEC-006-AC-10:** Given Jordan (young kid profile, no login) has no way to sign in, when any attempt is made to view the list, then no computed line is ever shown to this profile, since it cannot reach the screen at all.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 2 | 2 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |
