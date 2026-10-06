---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-16.SPEC-003
spec_name: Locale Configuration Validation & Defaults
spec_slug: locale-configuration-validation-defaults
parent_feature: FEAT-16
parent_feature_name: Units, Currency & Locale Configuration
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 20
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Locale Configuration Validation & Defaults

## Overview

**Name:** Locale Configuration Validation & Defaults
**ID:** FEAT-16.SPEC-003
**Type:** Logic/Rule
**Purpose:** Defines the supported unit and currency sets, the aisle-name and aisle-list rules, the authorization rules for changing locale settings, and the default unit system, currency, and aisle groupings a new household starts with.
**Parent Feature:** FEAT-16 -- Units, Currency & Locale Configuration
**Governed Entity:** Household -- locale fields (unit_system, currency, aisle_names)

## Scope and Non-Goals

**In Scope:**
- The supported unit_system set and its validation
- The supported currency set and its validation
- The aisle_names list's per-item length rule and list-level rules (minimum count, duplicate handling)
- Authorization rules for viewing and changing every locale field
- Default unit_system, currency, and aisle_names values applied when a new Household record is created
- Error messages for every locale-field validation failure

**Non-Goals:**
- The screens' layout, interactions, and save-failure recovery experience -- owned by FEAT-16.SPEC-001 (Units & Currency Settings) and FEAT-16.SPEC-002 (Aisle Name Customization), which enforce these rules but do not define them
- How a saved locale-setting change is reflected across other features' displayed figures -- governed by FEAT-16.SPEC-004 (Cross-Feature Value Conversion Rule)
- Validation of any other Household field (household_name, weekly_budget, weekly_schedule, plan_arrival_day_time, status, organiser) -- excluded per the Feature Dependency Map's Household entity definition, which assigns those fields to FEAT-01, FEAT-07, and FEAT-14 respectively; this spec addresses only the three locale fields named in its Governed Entity
- Automatic locale detection from the device or browser as a source for the default values -- excluded per feature-overview.md's Non-Goals: defaults are derived from setup-time organiser input and confirmed on screen, never silently auto-applied from device signals

## Governed Entity

**Entity:** Household -- locale fields only
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| unit_system | enum | The household's default measurement unit system: cups/oz (US customary) or grams/ml (metric) |
| currency | enum | The household's currency for budget and cost display, from the supported currency set |
| aisle_names | ordered list of text | The household's aisle groupings and their display order, used to group the shared grocery list |
| aisle_names[].name | text | One aisle grouping's display name (e.g., "Produce," "Dairy & Eggs") |
| aisle_names[].order | derived (position) | The aisle grouping's position in the list, determined by its index |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-16.SPEC-001 | Units & Currency Settings | On selection (only supported-set values are offered) and on Save (re-confirmed before writing to the Household record); authorization on screen entry (edit controls rendered only for Maya) |
| FEAT-16.SPEC-002 | Aisle Name Customization | On field blur (per-name length rule) and on Save (full list re-validated, including the minimum-count rule); authorization on screen entry (edit controls rendered only for Maya) |
| FEAT-01.SPEC-003 | Household Naming & Guided Setup Start | On Household record creation -- this spec's default-derivation rules populate unit_system, currency, and aisle_names at the moment the Household record is created, before FEAT-16.SPEC-001 is first shown |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| unit_system | Must be one of the supported unit systems: "cups/oz" (US customary) or "grams/ml" (metric) | Always | On selection and on submit | "Choose a measurement unit system to continue." | Yes |
| currency | Must be one of the supported currencies: at minimum "USD" (US Dollar) and "GBP" (British Pound) at launch | Always | On selection and on submit | "Choose a currency to continue." | Yes |
| aisle_names[].name | Required, non-empty after trimming whitespace, 1-30 characters | Always, per row | On blur and on submit | "Give this aisle a name." (empty) / "Aisle names must be 30 characters or fewer." (too long) | Yes |
| aisle_names (list) | The list must contain at least one aisle | Always | On submit | "Add at least one aisle before saving." | Yes |
| aisle_names[].order | No validation beyond data type -- position is derived automatically from the row's index in the saved list, never entered directly | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Duplicate aisle names permitted | aisle_names[].name | No uniqueness constraint across rows -- two aisles may share the same display name, since aisle_names entries are free-text grouping labels, not unique keys | N/A -- no error; duplicates are allowed |
| Currency independent of unit_system | unit_system, currency | The two fields validate and save independently; no combination of unit_system and currency is disallowed (e.g., "grams/ml" with "USD" is valid, matching a household that has moved between the US and UK) | N/A -- no combination is invalid |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View unit_system, currency | Maya (Organiser), Sam (Other Adult Member), Riley (Operator, support) | Always (Riley: only against a household with an open Support Request, per FEAT-22) | -- |
| View unit_system, currency | Jordan (young kid profile, no login -- MVP), Jordan (older kid, limited login -- Later) | Never | No login exists for the young-kid row, so the screen is unreachable; the older-kid login has no navigation entry point to it and a direct link redirects to that login's home screen |
| Change unit_system, currency | Maya (Organiser) | Always | -- |
| Change unit_system, currency | Sam (Other Adult Member) | Never | Edit controls are not rendered on FEAT-16.SPEC-001 for Sam; the screen shows the current values as static, non-interactive text |
| Change unit_system, currency | Riley (Operator, support) | Never | Edit controls are not rendered on FEAT-16.SPEC-001 for Riley; Operator Read-Only Support Access (FEAT-22) grants view only |
| View aisle_names | Maya (Organiser), Sam (Other Adult Member), Riley (Operator, support) | Always (Riley: only against a household with an open Support Request, per FEAT-22) | -- |
| View aisle_names | Jordan (young kid profile, no login -- MVP), Jordan (older kid, limited login -- Later) | Never | Same as above -- no reachable path to FEAT-16.SPEC-002 |
| Rename, reorder, add, or remove an aisle | Maya (Organiser) | Always | -- |
| Rename, reorder, add, or remove an aisle | Sam (Other Adult Member) | Never | Edit controls (drag handle, remove control, "Add aisle" action) are not rendered on FEAT-16.SPEC-002 for Sam; the screen shows the current list as static text |
| Rename, reorder, add, or remove an aisle | Riley (Operator, support) | Never | Edit controls are not rendered on FEAT-16.SPEC-002 for Riley; Operator Read-Only Support Access (FEAT-22) grants view only |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| unit_system | Defaults to "cups/oz" (US customary) as the initial pre-filled value shown on FEAT-16.SPEC-001 | On Household record creation (triggered by FEAT-01.SPEC-003) | Yes -- Maya confirms or changes it on FEAT-16.SPEC-001 before it is saved; this is a starting point for her explicit confirmation, not a silent auto-apply |
| currency | Defaults to "USD" (US Dollar) as the initial pre-filled value shown on FEAT-16.SPEC-001 | On Household record creation (triggered by FEAT-01.SPEC-003) | Yes -- Maya confirms or changes it on FEAT-16.SPEC-001 before it is saved |
| aisle_names | Defaults to a standard seven-grouping list, in this order: "Produce," "Meat & Seafood," "Dairy & Eggs," "Bakery," "Pantry & Dry Goods," "Frozen," "Household & Other" -- a generic grouping common to both US and UK supermarkets, shown pre-filled on FEAT-16.SPEC-002 | On Household record creation (triggered by FEAT-01.SPEC-003) | Yes -- Maya reviews, renames, reorders, adds to, or removes from this list on FEAT-16.SPEC-002 before it is saved |

## Business Rules

- Every locale field has a value from the moment the Household record is created -- there is no "unset" or null state for unit_system, currency, or aisle_names, consistent with product-features.md's States field for this feature ("Empty: N/A -- every household has a default locale configuration from creation").
- XBR-11: the supported unit and currency sets, and the aisle-name rules defined here, apply consistently everywhere locale settings are read (plan cost and weekly total, recipe quantities, grocery-list aisle grouping and quantities, budget, check-in spend) -- FEAT-16.SPEC-004 governs the display conversion itself, but the values it converts to or from must always come from this spec's supported sets.
- The default-derivation rule runs exactly once, at Household creation -- it is never re-applied afterward, even if a household later clears all its locale settings through some future capability; there is no such capability in this product definition, so this scenario does not currently arise.
- The supported currency set may be extended beyond USD and GBP in a future release without changing this spec's structure -- product-features.md's Validation & Limits field states the set covers "at least USD and GBP at launch," leaving room for growth, but this spec's rules apply to whatever the current supported set is at any time.

## Edge Cases

- **Aisle name at exactly 30 characters** -- Passes validation. 31 characters shows the length error.
- **Aisle name that is only whitespace** -- Treated as empty after trimming; shows "Give this aisle a name."
- **Maya removes all aisle rows in one edit session, then adds one back before saving** -- The minimum-count rule is evaluated only at submit time against the final pending list, so this sequence saves successfully once at least one aisle remains.
- **A household created before this spec's default aisle list changes (a future product change)** -- Not applicable to this run: the default-derivation rule runs once at creation and existing households' saved aisle_names are never overwritten by a later change to the default list; only new households receive the updated defaults.
- **Maya selects "grams/ml" for unit_system while currency remains "USD"** -- Both save successfully; no cross-field rule disallows this combination (a household that relocated from the UK to the US, for example, may want metric units with US dollars).
- **Two aisle rows are given the exact same name** -- Both save successfully; duplicate names are permitted per the Cross-Field Rules above.
- **Sam or Riley attempts to submit a direct edit request to a locale field by bypassing the screen's rendered controls** -- The authorization check runs independently of what the screen displays; the change is rejected and the field's value is unchanged, since Change actions are Never for these roles regardless of how the attempt reaches the system.

## Acceptance Criteria

**FEAT-16.SPEC-003-AC-01:** Given Maya is choosing a unit system on FEAT-16.SPEC-001, when only "cups/oz" and "grams/ml" are offered, then she cannot select or submit any other value.

**FEAT-16.SPEC-003-AC-02:** Given Maya is choosing a currency on FEAT-16.SPEC-001, when the selector lists the supported currencies, then "US Dollar" and "British Pound" both appear among the options.

**FEAT-16.SPEC-003-AC-03:** Given Maya leaves an aisle name empty on FEAT-16.SPEC-002 and moves focus away from the field, then the field shows the error "Give this aisle a name."

**FEAT-16.SPEC-003-AC-04:** Given Maya enters an aisle name of exactly 30 characters, then no length error is shown; given she enters 31 characters, then the error "Aisle names must be 30 characters or fewer." appears.

**FEAT-16.SPEC-003-AC-05:** Given Maya removes every aisle row and taps Save on FEAT-16.SPEC-002, then the error "Add at least one aisle before saving." appears and no save occurs.

**FEAT-16.SPEC-003-AC-06:** Given Maya renames two different aisles to the same name and taps Save, then the save succeeds without any duplicate-name error.

**FEAT-16.SPEC-003-AC-07:** Given Maya (Organiser) is on FEAT-16.SPEC-001, when she changes and saves the unit_system, then the change is applied, since Change is allowed for Maya always.

**FEAT-16.SPEC-003-AC-08:** Given Sam (Other Adult Member) is on FEAT-16.SPEC-001, when he looks for an edit control on the unit_system or currency selector, then none is shown, since Change is never allowed for Sam.

**FEAT-16.SPEC-003-AC-09:** Given Riley (Operator, support) is viewing FEAT-16.SPEC-002 through an open Support Request, when Riley looks for a drag handle or remove control on any aisle row, then none is shown, since Change is never allowed for Riley.

**FEAT-16.SPEC-003-AC-10:** Given Jordan as a young kid profile has no login, when any attempt is made to view or change locale settings, then no such path exists, since View and Change are both Never for this role.

**FEAT-16.SPEC-003-AC-11:** Given a new Household record is created through FEAT-01.SPEC-003, when the default-derivation rule runs, then unit_system is set to "cups/oz," currency is set to "USD," and aisle_names is set to the standard seven-grouping default list, all before Maya reaches FEAT-16.SPEC-001.

**FEAT-16.SPEC-003-AC-12:** Given Maya reaches FEAT-16.SPEC-001 immediately after household creation, when she changes the pre-filled unit_system default from "cups/oz" to "grams/ml" and confirms, then her chosen value overrides the default and is what saves.

**FEAT-16.SPEC-003-AC-13:** Given Maya selects "grams/ml" for unit_system and leaves currency as "USD," when she saves, then both values save successfully, since no cross-field rule links the two.

**FEAT-16.SPEC-003-AC-14:** Given Maya enters an aisle name of only spaces, when she moves focus away from the field, then it is treated as empty and shows "Give this aisle a name."

**FEAT-16.SPEC-003-AC-15:** Given a request to change Sam's currency-editing permission bypasses the FEAT-16.SPEC-001 screen entirely, when the change is evaluated against this spec's Authorization Rules, then it is rejected and the household's currency remains unchanged, since Change is Never for Sam regardless of how the request arrives.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 10 | 10 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |
