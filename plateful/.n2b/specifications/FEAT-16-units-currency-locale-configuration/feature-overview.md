---
document_type: feature-overview
feature_number: FEAT-16
feature_name: Units, Currency & Locale Configuration
feature_slug: units-currency-locale-configuration
priority_tier: Important
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 4
screen_count: 2
automation_count: 0
logic_rule_count: 2
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Units, Currency & Locale Configuration

## Summary

**Feature:** Units, Currency & Locale Configuration
**ID:** FEAT-16
**Description:** The household can set the measurement units, currency, and supermarket aisle names that fit where they live, so the plan and grocery list read naturally for US and UK households alike.
**Priority:** Important
**Phase:** MVP
**Type:** Platform
**Rationale:** The brief states directly that "units (cups vs grams), currency and supermarket aisle names must be configurable, not hard-coded" for the US and UK launch markets (BRIEF.md, Scale & Non-Functional Expectations). Included at MVP because both markets are targeted from launch, not added later.

**Key Capabilities:**
- Choose measurement units -- Household sets cups/oz or grams/ml as its default
- Choose currency -- Household sets its currency for budget and cost display
- Customize aisle names -- Household adjusts the aisle groupings on its grocery list to match how its local store is laid out

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-16.SPEC-001 | Units & Currency Settings | Screen | Maya (Organiser), Sam (Other Adult Member), Riley (Operator, support) | Maya sets or changes the household's default measurement unit system and currency; Sam and Riley see the current settings without changing them |
| FEAT-16.SPEC-002 | Aisle Name Customization | Screen | Maya (Organiser), Sam (Other Adult Member), Riley (Operator, support) | Maya renames or reorders the household's supermarket aisle groupings to match their local store; Sam and Riley see the current groupings without changing them |
| FEAT-16.SPEC-003 | Locale Configuration Validation & Defaults | Logic/Rule | Maya (Organiser) | Governs the supported unit and currency sets, the aisle-name length limit, and the default unit system, currency, and aisle groupings a new household starts with |
| FEAT-16.SPEC-004 | Cross-Feature Value Conversion Rule | Logic/Rule | All | Governs how stored quantities and costs are converted into the household's configured unit system and currency wherever they are displayed, and ensures a locale change updates existing figures for display rather than leaving them inconsistent |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Choose measurement units | FEAT-16.SPEC-001 | Primary purpose of the Units & Currency Settings screen | Phase 2 (Explicit) |
| Choose currency | FEAT-16.SPEC-001 | Primary purpose of the Units & Currency Settings screen | Phase 2 (Explicit) |
| Customize aisle names | FEAT-16.SPEC-002 | Primary purpose of the Aisle Name Customization screen | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-16.SPEC-003 | Locale Configuration Validation & Defaults | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field names a supported unit set, a supported currency set (covering at least USD and GBP), and an aisle-name length limit; the States field additionally implies every household has a default configuration from creation, based on setup-time input -- a derivation rule with no home in a single screen, since both settings screens read and enforce it |
| FEAT-16.SPEC-004 | Cross-Feature Value Conversion Rule | Phase 4 (Trigger-Response Analysis, cross-feature effects) + Phase 5 (Derivation rules) | XBR-11 requires that a locale change convert existing budget figures, recipe quantities, grocery-list quantities, and check-in spend for display rather than leaving them inconsistent, across five other features (FEAT-01, FEAT-03, FEAT-06, FEAT-08, FEAT-23, FEAT-25); this cross-feature derivation formula needed its own Logic/Rule spec rather than living inside either settings screen |

## Entity-Lifecycle Coverage Matrix

**Entity: Household (locale fields: unit_system, currency, aisle_names)**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | The Household record itself is created by FEAT-01 (Household Setup & Member Profiles); this feature does not create the entity, only its locale fields' initial values, which are populated by FEAT-16.SPEC-003's default-derivation rule at the moment FEAT-01 creates the household | Ownership boundary with FEAT-01 |
| Read (single) | FEAT-16.SPEC-001, FEAT-16.SPEC-002 | Both settings screens load and display the household's current unit_system/currency and aisle_names on entry | -- |
| Read (list) | N/A | Each household has exactly one locale configuration; there is no list of locale configurations to browse | One record per household (ASMP -- one household per account, SC-03) |
| Update | FEAT-16.SPEC-001 (unit_system, currency), FEAT-16.SPEC-002 (aisle_names) | Maya edits and saves; FEAT-16.SPEC-004 governs how the updated values are reflected across consuming features' displays | Only Maya can write; Sam and Riley are view-only per the Access Matrix |
| Delete/Archive | N/A -- owned by FEAT-18 | Locale fields have no independent delete/archive path; they are deleted only as part of the whole Household record's deletion, which FEAT-18 (Account & Data Management) owns end to end, including its retention/purge window | Recorded as a non-goal below to avoid a silent omission |
| State Transition | N/A | The product definition gives locale settings no lifecycle states beyond their current saved value -- there is no draft, pending, or archived state for a unit system, currency, or aisle-name set (product-features.md, States: "Empty: N/A ... Loading: N/A ... changes save instantly") | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Household (weekly_budget, other fields) | FEAT-16.SPEC-001 | Currency setting determines how the household's existing budget figure is labeled and displayed, without SPEC-001 itself managing the budget field |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Maya changes unit system or currency and saves | Validate the new value against the supported set; on success, apply the change and make future displays reflect it via the conversion rule | Standalone Logic/Rule (validation) feeding a Standalone Logic/Rule (conversion) | FEAT-16.SPEC-003, FEAT-16.SPEC-004 |
| Maya renames or reorders an aisle grouping and saves | Validate the new name against the length limit; apply the change to future grocery lists only | Standalone Logic/Rule (validation); inline confirmation in triggering screen | FEAT-16.SPEC-003; FEAT-16.SPEC-002 |
| Household is created (FEAT-01) | Populate unit_system, currency, and aisle_names with locale-appropriate defaults based on setup-time input | Standalone Logic/Rule | FEAT-16.SPEC-003 |
| A locale setting (unit system or currency) changes after setup | Existing budget figures, recipe quantities, grocery-list quantities, and check-in spend are converted for display in the newly configured unit/currency, rather than left showing the prior locale's values | Standalone Logic/Rule, cross-feature | FEAT-16.SPEC-004 (consumed by FEAT-01, FEAT-03, FEAT-06, FEAT-08, FEAT-23, FEAT-25) |
| Maya's save to either settings screen fails (e.g., lost connectivity mid-save) | The prior setting remains active; the user sees the save did not take effect rather than a mixed or inconsistent state | Inline in triggering screen | FEAT-16.SPEC-001, FEAT-16.SPEC-002 |
| Sam or Riley opens either settings screen | Display current values in a view-only mode; no edit controls rendered | Inline in triggering screen | FEAT-16.SPEC-001, FEAT-16.SPEC-002 |
| An unauthorized visitor attempts to reach either settings screen | Access denied -- no locale data is shown | Inline in triggering screen | FEAT-16.SPEC-001, FEAT-16.SPEC-002 |
| Either settings screen is opened while offline | Current values are viewable; edit controls are present but saving is disabled until connectivity returns | Inline in triggering screen | FEAT-16.SPEC-001, FEAT-16.SPEC-002 |

**On zero Automation specs:** This feature manages a single, low-contention configuration record with no cascading creates, no scheduled processing, and no asynchronous background work of its own -- every discovered side-effect is either a synchronous validation/derivation rule (SPEC-003, SPEC-004) or a simple inline consequence of a screen save. No trigger-response pair in this feature crosses the "processing logic with its own lifecycle and failure modes" threshold that would justify a standalone Automation spec.

## Shared Context

**Shared Entities:**
- Household locale fields (unit_system, currency, aisle_names) -- read and updated by FEAT-16.SPEC-001 (unit_system, currency) and FEAT-16.SPEC-002 (aisle_names); validated and defaulted by FEAT-16.SPEC-003; consumed for display conversion by FEAT-16.SPEC-004 and, through it, by FEAT-01, FEAT-03, FEAT-06, FEAT-08, FEAT-23, and FEAT-25.

**Shared UI Patterns:**
- Role-gated settings display -- both FEAT-16.SPEC-001 and FEAT-16.SPEC-002 render the same record in two modes: an editable form for Maya, and a read-only display of the same fields for Sam and Riley. Spec Writers for both screens should describe the view-only mode consistently (same layout, controls simply removed/disabled, not hidden behind a different screen).
- Save-failure recovery -- both screens keep the prior value visible and active on a failed save, per the feature's States field; Spec Writers should describe this identically rather than inventing screen-specific error copy.

**Shared Validation:**
- FEAT-16.SPEC-003 defines the supported unit set, the supported currency set (at least USD and GBP), the aisle-name length limit, and the default-value derivation. FEAT-16.SPEC-001 and FEAT-16.SPEC-002 both reference SPEC-003 for their field-level validation and initial values rather than duplicating the rules.

## Internal Dependency Map

```
SPEC-001 (Units & Currency Settings) -> [Maya saves a change] -> SPEC-003 (Locale Configuration Validation & Defaults) -> [valid] -> SPEC-004 (Cross-Feature Value Conversion Rule)
SPEC-002 (Aisle Name Customization) -> [Maya saves a change] -> SPEC-003 (Locale Configuration Validation & Defaults) -> [valid] -> applies to future grocery lists (FEAT-06)
SPEC-003 (Locale Configuration Validation & Defaults) -> [household created by FEAT-01] -> populates default unit_system, currency, aisle_names on SPEC-001 / SPEC-002
SPEC-004 (Cross-Feature Value Conversion Rule) -> [locale setting changes] -> existing displayed figures reflow in FEAT-01, FEAT-03, FEAT-06, FEAT-08, FEAT-23, FEAT-25
```

**Default Entry:** SPEC-001 (Units & Currency Settings) -- the screen shown when the user navigates to this feature's settings area from Household Setup; SPEC-002 (Aisle Name Customization) is reached from within the same settings area as a distinct sub-section.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-16.SPEC-001 | Inbound | FEAT-01 (Household Setup & Member Profiles) | Maya reaches the units and currency step from the guided setup flow | Setup step "Set units, currency and aisle layout" (First Household Setup, step 4) |
| FEAT-16.SPEC-002 | Inbound | FEAT-01 (Household Setup & Member Profiles) | Maya reaches the default aisle-groupings review from the guided setup flow | Setup step "Set units, currency and aisle layout" (First Household Setup, step 4) |
| FEAT-16.SPEC-003 | Outbound | FEAT-01 (Household Setup & Member Profiles) | Default unit system, currency, and aisle groupings are populated onto the new Household record at creation | Household creation |
| FEAT-16.SPEC-004 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Weekly plan cost and total display in the household's configured currency | Plan displayed or locale setting changed |
| FEAT-16.SPEC-004 | Outbound | FEAT-06 (Shared Grocery List) | Grocery-list quantities display in the household's configured unit system; aisle grouping reflects FEAT-16.SPEC-002's current names/order | List generated, item displayed, or locale/aisle setting changed |
| FEAT-16.SPEC-004 | Outbound | FEAT-08 (Recipe Library & Starter Recipes) | Recipe ingredient quantities display in the household's configured unit system | Recipe viewed or unit setting changed |
| FEAT-16.SPEC-004 | Outbound | FEAT-23 (Manual Weekly Planning) | Manual plan's budget figures display in the household's configured currency | Manual plan viewed or currency setting changed |
| FEAT-16.SPEC-004 | Outbound | FEAT-25 (Weekly Waste & Spend Check-In) | Check-in spend figures display in the household's configured currency | Check-in viewed or currency setting changed |

## Non-Functional Notes

**Data volumes / growth:** Negligible -- a single locale configuration record per household (unit_system, currency, aisle_names), unchanged in shape as the household base grows to several thousand households (BRIEF.md, Scale & Non-Functional Expectations, Order of magnitude). Aisle-name customization holds a small, bounded list of aisle groupings per household, not an unbounded collection.

**Responsiveness:** Settings changes save instantly with no perceptible wait, consistent with the feature's own States field ("Loading: N/A -- changes save instantly"); the resulting display conversion must appear immediately wherever the plan, recipes, or grocery list are next shown, not as a delayed batch update.

**Data sensitivity / privacy:** Household locale settings (unit system, currency, aisle names) are personal household configuration data -- private to household members, never sold or used for advertising (ASMP-26); they carry no children's-data sensitivity, since no kid-profile field is stored here. They are exportable and deletable under general personal-data rights as part of the whole Household record (ASMP-27, FEAT-18).

**Compliance flags:** ASMP-28 (Localization) governs this feature directly: measurement units, currency, and aisle names must be configurable to support both US and UK households from launch, and the product remains English-only (SC-13) -- this feature delivers locale-format configurability, not translation. No health, financial, or children's-privacy regime applies to locale settings themselves.

## Non-Goals

- **Languages other than English** -- Excluded per scope-boundaries.md (SC-13): the brief targets the US and UK first and requires configurable units, currency, and aisle names (FEAT-16), but explicitly does not require translated content for the two launch markets.
- **Automatic purge of locale settings** -- Intentional lifecycle decision surfaced by the CRUD matrix: locale fields have no independent delete/archive or retention policy because they are not an independent entity -- they are deleted only as part of the whole Household record's deletion, which FEAT-18 (Account & Data Management) owns end to end, including that feature's 30-day retention window (ASMP-27).
- **Support for currencies or unit systems beyond the supported set** -- Excluded per the feature's own Validation & Limits field, which scopes units to a supported set (e.g., cups/oz, grams/ml) and currency to a supported set covering at least USD and GBP at launch; broader locale support is not a stated MVP need since the brief targets only the US and UK launch markets (BRIEF.md, Scale & Non-Functional Expectations, Geography).
- **Automatic locale detection from device or browser settings** -- Adjacency exclusion: the feature's Data Notes name the Source as "organiser input," and the journey describes Maya confirming, not merely accepting, locale defaults during guided setup (First Household Setup, step 4); this feature derives sensible starting defaults (FEAT-16.SPEC-003) but does not silently auto-apply device-detected locale without the organiser's confirmation.
