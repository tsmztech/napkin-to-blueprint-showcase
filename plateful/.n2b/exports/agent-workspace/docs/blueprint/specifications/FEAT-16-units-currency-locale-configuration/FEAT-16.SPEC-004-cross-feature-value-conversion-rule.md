---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-16.SPEC-004
spec_name: Cross-Feature Value Conversion Rule
spec_slug: cross-feature-value-conversion-rule
parent_feature: FEAT-16
parent_feature_name: Units, Currency & Locale Configuration
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 14
acceptance_criteria_count: 13
---

# Logic/Rule Spec: Cross-Feature Value Conversion Rule

## Overview

**Name:** Cross-Feature Value Conversion Rule
**ID:** FEAT-16.SPEC-004
**Type:** Logic/Rule
**Purpose:** Defines how stored quantities and costs are converted, at the moment they are displayed, into the household's currently configured unit system and currency, so that a locale change is reflected immediately and consistently everywhere those values appear, without leaving any displayed figure inconsistent with the household's current settings.
**Parent Feature:** FEAT-16 -- Units, Currency & Locale Configuration
**Governed Entity:** Displayed quantities and costs across the Household, Weekly Plan, Planned Meal, Recipe, Grocery List Item, and Waste & Spend Check-In entities (a cross-feature derivation rule, not a single entity's own CRUD lifecycle)

## Scope and Non-Goals

**In Scope:**
- The conversion logic applied to any stored quantity (e.g., a recipe ingredient amount) when the household's unit_system differs from the unit system the value was originally recorded in
- The conversion logic applied to any stored monetary figure (budget, plan totals, meal costs, check-in spend) for display under the household's currently configured currency
- The rule that a locale-setting change takes effect for display immediately, on every screen that shows an affected figure, without a delayed or batch update
- Rounding and formatting behavior for converted values
- Which entities' fields this rule touches, by exact field name, across the six affected entities

**Non-Goals:**
- Real-time currency exchange-rate conversion between USD, GBP, or any other supported currency -- excluded because no currency-exchange capability appears in the Feature Dependency Map's External Touchpoints table, and product-features.md's Validation & Limits field frames currency as a display/labeling setting ("currency setting determines how the household's existing budget figure is labeled and displayed"), not a financial conversion; a currency change relabels the same stored numeric figure under the new currency's symbol (see Defaults and Derivations)
- Defining the supported unit and currency sets, or the aisle-name rules -- governed by FEAT-16.SPEC-003 (Locale Configuration Validation & Defaults); this spec only converts values that are already valid under that spec's supported sets
- The screens where unit_system, currency, or aisle_names are changed -- owned by FEAT-16.SPEC-001 and FEAT-16.SPEC-002, which trigger this rule on save but do not define its conversion logic
- Re-grouping an already-generated grocery list's aisle assignments when aisle names change -- per feature-overview.md's Side-Effect Inventory, an aisle-name change applies to future grocery lists only; this spec's conversion behavior for quantities and costs is immediate, but aisle re-grouping is explicitly out of scope for an already-generated list

## Governed Entity

**Entity:** Displayed quantities and costs (cross-feature; each row below names its owning entity per the Feature Dependency Map)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| Household.unit_system | enum | The household's target unit system for all quantity conversions (read from FEAT-16.SPEC-003's supported set) |
| Household.currency | enum | The household's target currency for all monetary display (read from FEAT-16.SPEC-003's supported set) |
| Household.weekly_budget | derived (display) | The household's budget figure, displayed under Household.currency |
| Weekly Plan.estimated_total | derived (display) | The week's estimated cost, displayed under Household.currency |
| Planned Meal.rough_cost | derived (display) | A single dinner's estimated cost, displayed under Household.currency |
| Recipe.ingredients[].quantity_and_unit | derived (display) | Each ingredient's amount, displayed under Household.unit_system |
| Grocery List Item.quantity_and_unit | derived (display) | Each combined grocery-list line's amount, displayed under Household.unit_system |
| Waste & Spend Check-In (spend figures) | derived (display) | The household's self-reported weekly spend answer, displayed under Household.currency |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-16.SPEC-001 | Units & Currency Settings | Triggers this rule on a successful save of unit_system or currency |
| FEAT-16.SPEC-002 | Aisle Name Customization | Triggers this rule on a successful save of aisle_names (for the aisle-grouping consequence only; quantities and costs are unaffected by an aisle-name change) |
| FEAT-01.SPEC-008 | Weekly Budget & Schedule Setup | Applies this rule wherever weekly_budget is shown, per the household's currency |
| FEAT-03.SPEC-001 | Weekly Plan View | Applies this rule to Weekly Plan.estimated_total and each Planned Meal.rough_cost wherever the plan is shown |
| FEAT-03.SPEC-006 | Budget-Fit / Estimated Total Rule | Applies this rule's currency display to the estimated_total figure this rule computes |
| FEAT-06.SPEC-001 | Grocery List | Applies this rule to Grocery List Item.quantity_and_unit wherever the list is shown |
| FEAT-06.SPEC-006 | Ingredient Consolidation & Quantity Derivation | Applies this rule to the combined quantity this spec derives, before display |
| FEAT-08.SPEC-002 | Recipe Detail View | Applies this rule to Recipe.ingredients[].quantity_and_unit wherever a recipe is viewed |
| FEAT-23.SPEC-001 | Weekly Plan Manual Week Builder | Applies this rule to the manually built week's cost figures, the same way FEAT-03.SPEC-001 does |
| FEAT-25 (Weekly Waste & Spend Check-In) | Check-in spend figure display | Applies this rule to the check-in's self-reported spend answer wherever it is shown -- referenced at feature level since FEAT-25's specs are not yet produced as of this run |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Household.unit_system, Household.currency | No validation beyond data type -- these are read, not written, by this spec; their own validation is governed by FEAT-16.SPEC-003 | Always | -- | -- | -- |
| Household.weekly_budget, Weekly Plan.estimated_total, Planned Meal.rough_cost, Waste & Spend Check-In spend figures | No validation beyond data type -- these are derived display values; this spec only formats them for display, it does not validate their underlying accuracy | Always | -- | -- | -- |
| Recipe.ingredients[].quantity_and_unit, Grocery List Item.quantity_and_unit | No validation beyond data type -- this spec only converts these values for display; their underlying accuracy is governed by the entity that authors them (Recipe by FEAT-08/FEAT-10, Grocery List Item by FEAT-06) | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Unit conversion applies only when the source and target unit systems differ | Household.unit_system, Recipe.ingredients[].quantity_and_unit, Grocery List Item.quantity_and_unit | If the value's originally recorded unit system already matches Household.unit_system, it is displayed unconverted; otherwise the conversion table in Defaults and Derivations applies | N/A -- no error; this is a display computation, not a validation |
| Currency relabeling is independent of unit conversion | Household.currency, Household.unit_system | A change to one field never triggers or blocks conversion of the other -- unit_system changes affect only quantity display, currency changes affect only monetary display | N/A -- no error |
| A locale change never rewrites stored values | Household.unit_system, Household.currency, all governed display fields | Conversion happens at display time from the stored, originally recorded value -- the underlying stored quantity_and_unit or monetary figure is never overwritten by a locale change, so reverting unit_system or currency back to a prior value restores the original display exactly | N/A -- no error; this is what keeps the conversion reversible and lossless |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View a locale-converted quantity or cost | Maya (Organiser), Sam (Other Adult Member), Jordan (older kid, limited login -- Later, where that role can view the underlying screen, e.g., the Grocery List), Riley (Operator, support, where FEAT-22 grants view) | Per the viewing role's own access rules on the underlying screen (FEAT-01, FEAT-03, FEAT-06, FEAT-08, FEAT-16.SPEC-001, FEAT-23, FEAT-25); this spec adds no restriction of its own | -- |
| View a locale-converted quantity or cost | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role, so no screen showing a converted value is reachable |
| Change which unit system or currency values convert to | Maya (Organiser) | Always -- via FEAT-16.SPEC-001, not this spec directly | -- |
| Change which unit system or currency values convert to | Sam (Other Adult Member), Riley (Operator, support), both Jordan rows | Never | No control exists on any screen for these roles to change unit_system or currency, per FEAT-16.SPEC-003's Authorization Rules |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Recipe.ingredients[].quantity_and_unit (displayed) | If the ingredient's originally recorded unit belongs to the household's currently configured unit_system, display it unconverted. Otherwise, apply the fixed conversion factor for that unit: 1 cup -> 240 ml; 1 tablespoon -> 15 ml; 1 teaspoon -> 5 ml; 1 fluid ounce -> 30 ml; 1 ounce (weight) -> 28 grams; 1 pound -> 454 grams -- and the inverse factors when converting from metric back to US customary. Metric results round to the nearest whole gram or millilitre; US-customary results round to the nearest common cooking fraction (1/4, 1/3, 1/2, 2/3, 3/4, or whole unit). | Every time the ingredient is displayed (plan, recipe detail, grocery list) | No -- the household's unit_system is the single control; there is no per-view override |
| Grocery List Item.quantity_and_unit (displayed) | Same fixed conversion factors as Recipe ingredients above, applied to the combined quantity after Shared Grocery List (FEAT-06) has already summed the ingredient across dinners | Every time the grocery list is displayed | No |
| Household.weekly_budget, Weekly Plan.estimated_total, Planned Meal.rough_cost, Waste & Spend Check-In spend figures (displayed) | Displayed with the household's currently configured currency's symbol and standard formatting for that currency, applied to the stored numeric figure exactly as recorded -- no exchange-rate multiplication or division is applied; a currency change relabels the same number under the new symbol | Every time the figure is displayed | No |
| Any governed field, on a locale-setting change | Re-computed for display the next time each affected screen is shown -- no background batch job re-renders every past screen at once, and no stored value is rewritten; the conversion is applied fresh at each display | Immediately after FEAT-16.SPEC-001 or FEAT-16.SPEC-002 saves a change | No |

## Business Rules

- XBR-11: the household's units, currency, and aisle names apply consistently everywhere they appear -- plan cost and weekly total, recipe quantities, grocery-list aisle grouping and quantities, budget, and check-in spend -- and a later change converts existing figures for display rather than leaving them inconsistent. This spec is the mechanism that fulfills XBR-11 for quantities and monetary figures; FEAT-16.SPEC-001 and FEAT-16.SPEC-002 fulfill the aisle-grouping half directly.
- Conversion is a pure display computation: it never writes to Recipe, Grocery List Item, Weekly Plan, Planned Meal, or Waste & Spend Check-In records. Only the Household record's own unit_system and currency fields are ever written, and only by FEAT-16.SPEC-001 and FEAT-16.SPEC-002.
- Per the feature's Non-Functional Notes, a converted display must appear immediately wherever the plan, recipes, or grocery list are next shown -- there is no acceptable delay between a locale-setting save and the next screen reflecting it, since conversion happens at display time rather than through a background reflow.
- A recipe or grocery-list item authored in one unit system (e.g., a starter recipe written in cups/oz) is never edited to permanently change its authored units; this rule only affects what is shown to the currently viewing household, and two households with different unit_system settings can view the same starter recipe converted differently at the same time.

## Edge Cases

- **Maya changes unit_system, then immediately opens the current week's plan** -- Every ingredient quantity shown in the plan and its linked recipes reflects the new unit_system on that very screen load; there is no stale-display window, since conversion is computed fresh at display time.
- **Maya changes currency, then reviews a past archived Weekly Plan (Weekly Plan History, FEAT-19, v1)** -- The archived plan's estimated_total displays under the newly configured currency's symbol on the same stored figure; archiving does not freeze the display currency to whatever it was when the plan was active.
- **An ingredient quantity has no clean conversion result (e.g., 1/3 cup converts to 79 ml)** -- The metric result rounds to the nearest whole millilitre (79 ml, not a fraction); this is display rounding only and never alters the stored authored quantity.
- **Maya changes unit_system and currency in the same save session (both fields changed on FEAT-16.SPEC-001 at once)** -- Both conversions apply independently and simultaneously on the next display of any affected screen; there is no ordering dependency between the two.
- **Maya reverts unit_system from metric back to US customary after having changed it earlier** -- Displayed quantities return exactly to their originally authored US-customary values, since the underlying stored quantity_and_unit was never overwritten by the earlier conversion.
- **A household's currency is changed while its Weekly Plan.estimated_total is mid-calculation (a plan is still generating)** -- The in-progress calculation completes and stores its result as before; the newly configured currency's symbol is applied only when the completed total is displayed, not to the calculation itself.
- **Sam views the grocery list on his phone at the same moment Maya changes unit_system on hers** -- Sam sees the list re-render in the new unit_system the next time his screen loads or refreshes the list; per the Grocery List's own live-update behavior (FEAT-06), this is consistent with values never being shown "without its matching list" stated in XBR-03, extended here to unit display.

## Acceptance Criteria

**FEAT-16.SPEC-004-AC-01:** Given Maya changes the household's unit_system from cups/oz to grams/ml and saves on FEAT-16.SPEC-001, when she next opens a recipe originally authored in cups, then its ingredient quantities display converted to grams/ml using the fixed conversion factors, with metric results rounded to the nearest whole gram or millilitre.

**FEAT-16.SPEC-004-AC-02:** Given a household's unit_system is already grams/ml, when Maya opens a recipe originally authored in grams/ml, then its quantities display unconverted, since the source and target unit systems match.

**FEAT-16.SPEC-004-AC-03:** Given Maya changes the household's currency from USD to GBP and saves, when she next opens the current week's plan, then the plan's estimated_total and each meal's rough_cost display with the £ symbol applied to the same stored numeric figures, with no exchange-rate multiplication applied.

**FEAT-16.SPEC-004-AC-04:** Given Maya saves a currency change, when Sam opens the Household Setup screen showing weekly_budget, then the same stored budget number displays under the newly configured currency's symbol.

**FEAT-16.SPEC-004-AC-05:** Given Maya changes unit_system, when Sam opens the shared Grocery List, then its combined quantities display converted to the new unit_system using the same fixed conversion factors applied to the summed amount.

**FEAT-16.SPEC-004-AC-06:** Given a stored ingredient quantity of 1/3 cup, when the household's unit_system is grams/ml, then the displayed value rounds to the nearest whole millilitre rather than showing a fractional millilitre amount.

**FEAT-16.SPEC-004-AC-07:** Given Maya changes both unit_system and currency in the same save on FEAT-16.SPEC-001, when she next opens the plan, then both the displayed quantities and the displayed costs reflect their respective new settings on that same load.

**FEAT-16.SPEC-004-AC-08:** Given Maya changed unit_system to metric last week and now changes it back to cups/oz, when she opens a recipe originally authored in cups, then its quantities display exactly as originally authored, since the stored value was never overwritten by the earlier conversion.

**FEAT-16.SPEC-004-AC-09:** Given Maya reviews an archived past Weekly Plan after changing the household's currency, then the archived plan's estimated_total displays under the newly configured currency, not the currency that was active when the plan was archived.

**FEAT-16.SPEC-004-AC-10:** Given the household's currency is USD, when Maya answers the Weekly Waste & Spend Check-In's spend question, then the answer displays with the $ symbol, and if she later changes currency to GBP, the same recorded answer displays with the £ symbol on next view.

**FEAT-16.SPEC-004-AC-11:** Given Jordan is a young kid profile with no login, when any attempt is made to view a locale-converted value, then no such path exists, since no screen is reachable for this role.

**FEAT-16.SPEC-004-AC-12:** Given Riley (Operator, support) views a household's plan through an open Support Request, when the plan's costs are shown, then they display converted per the household's own currency, the same as any other viewer, since this spec adds no role-specific restriction beyond the underlying screen's own access rules.

**FEAT-16.SPEC-004-AC-13:** Given a starter recipe authored in cups/oz is viewed by two different households, one configured for cups/oz and one configured for grams/ml, when both view the same recipe at the same time, then the first sees the authored cups/oz quantities unconverted and the second sees the grams/ml converted equivalents, without the underlying Recipe record being modified for either household.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |
