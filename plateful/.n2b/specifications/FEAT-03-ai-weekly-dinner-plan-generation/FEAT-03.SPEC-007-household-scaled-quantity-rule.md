---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-03.SPEC-007
spec_name: Household-Scaled Quantity Rule
spec_slug: household-scaled-quantity-rule
parent_feature: FEAT-03
parent_feature_name: AI Weekly Dinner Plan Generation
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 15
acceptance_criteria_count: 10
---

# Logic/Rule Spec: Household-Scaled Quantity Rule

## Overview

**Name:** Household-Scaled Quantity Rule
**ID:** FEAT-03.SPEC-007
**Type:** Logic/Rule
**Purpose:** Derives per-dinner ingredient quantities sized to the number of people the household is planning for.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation
**Governed Entity:** Planned Meal (cook_time, rough_cost fields, and the scaled ingredient quantities that feed the Grocery List)

## Scope and Non-Goals

**In Scope:**
- Scaling a candidate recipe's ingredient quantities to the household's member count for each of the seven Planned Meals created at generation
- Deriving the household-scaled rough_cost shown per dinner and fed into FEAT-03.SPEC-006's budget computation
- Ensuring scaled quantities carry through consistently to the Grocery List

**Non-Goals:**
- Defining a recipe's base (unscaled) ingredient list, cook_time, or base cost -- owned by Recipe Library (FEAT-08) and Recipe Import (FEAT-10); this rule only transforms those base figures for the household
- Combining scaled quantities across multiple dinners into single grocery-list lines -- owned by Shared Grocery List (FEAT-06), which consumes this rule's per-dinner output
- Determining which household members count toward the scaling number -- household member count itself is owned by Household Setup & Member Profiles (FEAT-01); this rule only reads that count

## Governed Entity

**Entity:** Planned Meal
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| night | enum | The day of the week; at most one dinner per night |
| meal_kind | enum | Dinner, or leftover lunch linked to one source dinner |
| recipe | reference | The chosen Recipe |
| safety_badge | text | "Checked against allergies" plus the "always check labels" disclaimer |
| vegetarian_option | boolean | Whether a shared meal carries a vegetarian variant |
| cook_time | number | Carried from the recipe -- not scaled by household size |
| rough_cost | number | Carried from the recipe, sized for the household -- governed by this spec |
| pantry_callout | text | Which logged pantry items this dinner uses |
| status | enum | Proposed/Picked, Confirmed, Swapped, Removed (safety), Cooked |
| swap_history | list | Prior recipes in this slot |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-03.SPEC-003 | Scheduled Weekly Plan Generation | During candidate selection, after safety filtering and before budget-fit computation, for each of the seven selected dinners |
| FEAT-03.SPEC-004 | First-Plan Generation on Upgrade | Same enforcement point, for a household's first generation run |
| FEAT-03.SPEC-006 | Budget Fit & Estimated Total Rule | Consumes this rule's household-scaled rough_cost as its input for estimated_total |
| FEAT-06 | Shared Grocery List | Consumes this rule's scaled ingredient quantities when building the week's list |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| rough_cost (scaled) | Must be a positive number, derived from the recipe's base rough_cost scaled to household member count | Always | On computation, during generation | N/A -- computed field, not user input | No |
| cook_time | No validation beyond data type -- carried from the recipe unscaled, since cook time does not change with portion count | Always | -- | -- | -- |
| Scaled ingredient quantities (feed Grocery List, not a Planned Meal field directly) | Must be positive, non-zero quantities for every ingredient the recipe defines, scaled to household member count | Always | On computation, during generation | N/A -- computed, not user input | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Cost scales with quantity | rough_cost, household member count | rough_cost scales proportionally with the same scaling factor applied to ingredient quantities, so a doubled household sees roughly double the per-dinner cost | N/A |
| Cook time does not scale | cook_time, household member count | cook_time remains the recipe's base value regardless of household size, since preparation time does not scale linearly with portion count | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Trigger the scaling computation | System (invoked by FEAT-03.SPEC-003, FEAT-03.SPEC-004 during generation) | Always, as part of generation | N/A -- invoked internally, not by direct user action |
| View scaled cook_time and rough_cost | Maya (Organiser), Sam (Other Adult Member), Jordan (older kid, limited login -- Later) | Always, wherever the Weekly Plan is displayed (FEAT-03.SPEC-001) | -- |
| View scaled cook_time and rough_cost | Jordan (young kid profile, no login -- MVP) | Never -- no login exists for this row | No sign-in path exists for this profile |
| View scaled cook_time and rough_cost | Riley (Operator, support -- from v1) | Only while a Support Request for the household is open (FEAT-22, XBR-14) | Outside an open Support Request, no access to any household screen showing this data |
| Change the household member count this rule scales to | Maya (Organiser) | Always, through adding or removing Member Profiles (FEAT-01) | -- |
| Change the household member count | Sam, both Jordan rows, Riley | Never -- Household Setup is View or None for these roles | Member management controls are not shown to these roles |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Scaled ingredient quantities | Recipe's base ingredient quantities multiplied by a scaling factor derived from the household's current member count | On generation, per selected dinner | No -- always derived; a household changes the outcome only by changing its member count (FEAT-01) |
| rough_cost (scaled) | Recipe's base rough_cost multiplied by the same scaling factor applied to ingredient quantities | On generation, per selected dinner | No -- always derived |
| cook_time | Recipe's base cook_time, unscaled | On generation, per selected dinner | No -- cook_time is never scaled |

## Business Rules

- Ingredient quantities are sized for the number of people eating, so the grocery list buys the right amount (product-features.md, FEAT-03 Key Capabilities: "Scale to the household").
- Scaling always precedes the budget-fit computation (FEAT-03.SPEC-006), since an unscaled cost would misstate what the household will actually spend.
- Scaled quantities feed the Grocery List (FEAT-06) directly and consistently -- a household that changes size before its next generation sees the new size reflected starting with that cycle, not retroactively on the current week's already-built list (mirroring FEAT-01's "later edit" rule).
- Household member count for scaling includes every active Member Profile the household is currently planning meals for, consistent with the count Household Setup (FEAT-01) maintains.

## Edge Cases

- **A recipe's base ingredient list includes an ingredient with no meaningful fractional scaling (e.g., "1 lemon")** -- The scaled quantity rounds to the nearest sensible whole unit for that ingredient type rather than producing a fractional or zero quantity; the exact rounding convention per ingredient type is a Recipe Library (FEAT-08) data concern, not this rule's.
- **Household size changes (a member is added or removed) after this week's plan has already generated** -- The current week's already-scaled quantities and rough_cost are not retroactively recalculated; the new member count applies starting the household's next generation cycle, consistent with FEAT-01's later-edit behavior.
- **Household has only one member** -- Scaling still applies; a scaling factor of one produces the recipe's base quantities and cost unchanged.
- **Household is at its maximum of 12 member profiles** -- Scaling applies the same proportional logic at the upper bound as at any other household size; no separate cap or different formula applies.
- **A recipe carries a vegetarian_option variant with different ingredients from the main dish** -- Both the main and vegetarian_option ingredient sets are scaled independently to the respective number of people eating each variant, so the grocery list buys the right amount of each.

## Acceptance Criteria

**FEAT-03.SPEC-007-AC-01:** Given a household of four people, when generation selects a dinner whose base recipe serves two, then the Planned Meal's ingredient quantities and rough_cost are scaled to four servings.

**FEAT-03.SPEC-007-AC-02:** Given a household of four people, when generation selects a dinner, then its cook_time is shown unchanged from the recipe's base cook_time.

**FEAT-03.SPEC-007-AC-03:** Given a household with exactly one member, when generation selects a dinner, then its scaled quantities and rough_cost equal the recipe's base values.

**FEAT-03.SPEC-007-AC-04:** Given a household with 12 member profiles (the maximum), when generation selects a dinner, then quantities and cost scale proportionally using the same logic as any other household size.

**FEAT-03.SPEC-007-AC-05:** Given Maya adds a new member to the household mid-week, when the change is saved, then the current week's already-generated Planned Meals keep their existing scaled quantities, and the new member count applies starting the next generation cycle.

**FEAT-03.SPEC-007-AC-06:** Given a dinner carries a vegetarian_option variant, when generation scales the dinner, then the main dish and the vegetarian variant are each scaled to their respective number of people eating.

**FEAT-03.SPEC-007-AC-07:** Given Maya views the Weekly Plan View, when a dinner card renders, then the shown cook_time and rough_cost reflect this rule's household-scaled output.

**FEAT-03.SPEC-007-AC-08:** Given Sam views the Weekly Plan View, when a dinner card renders, then he sees the same scaled cook_time and rough_cost as Maya.

**FEAT-03.SPEC-007-AC-09:** Given Riley (Operator) has no open Support Request for a household, when Riley attempts to view that household's plan, then no screen showing scaled quantities or cost is reachable.

**FEAT-03.SPEC-007-AC-10:** Given the household's grocery list is built from the current week's plan, when it is generated, then it uses this rule's household-scaled ingredient quantities, not the recipe's unscaled base quantities.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
