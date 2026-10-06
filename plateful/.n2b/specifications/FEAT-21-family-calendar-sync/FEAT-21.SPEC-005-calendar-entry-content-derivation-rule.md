---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-21.SPEC-005
spec_name: Calendar Entry Content Derivation Rule
spec_slug: calendar-entry-content-derivation-rule
parent_feature: FEAT-21
parent_feature_name: Family Calendar Sync
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 12
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Calendar Entry Content Derivation Rule

## Overview

**Name:** Calendar Entry Content Derivation Rule
**ID:** FEAT-21.SPEC-005
**Type:** Logic/Rule
**Purpose:** Derives what a synced calendar entry contains (night, dish name, timing) from Weekly Plan and Planned Meal data, and keeps one entry per planned night stable across swaps.
**Parent Feature:** FEAT-21 -- Family Calendar Sync
**Governed Entity:** Calendar Entry

## Scope and Non-Goals

**In Scope:**
- The exact content placed on a synced calendar entry (night, dish name, timing) and the fields explicitly excluded from it
- The stable per-night entry-identity rule that makes an update possible instead of a duplicate create
- Derivation logic for each Calendar Entry field, sourced from Weekly Plan and Planned Meal data
- Authorization for who (if anyone) can influence entry content directly

**Non-Goals:**
- Creating the entry on the household's calendar, or handling the capability's create/update response -- owned by FEAT-21.SPEC-002 (Family Calendar Integration), which sends the content this spec derives.
- Deciding when a night needs to be evaluated for sync -- owned by FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync), which calls this spec's derivation and identity rules but owns the evaluation schedule itself.
- Governing the connection this content travels over, retry behavior, or the one-connection-per-household limit -- owned by FEAT-21.SPEC-004 (Calendar Connection & Sync Governance Rules).
- Deriving content for leftover-lunch Planned Meals -- excluded per FEAT-21.SPEC-003's Non-Goals; this feature syncs dinners only, so no leftover-lunch content is ever derived here.

## Governed Entity

**Entity:** Calendar Entry
**Source:** Feature-local entity, derived by this spec from the dependency map's Weekly Plan and Planned Meal entities (feature-overview.md, Referenced Entities: Weekly Plan and Planned Meal). It is not itself a Connected Entity in product-features.md and lives entirely on the household's external calendar once synced (FEAT-21.SPEC-002).

| Field | Data Type | Description |
|-------|-----------|-------------|
| night_date | derived | The calendar date of the dinner's night, computed from the Weekly Plan's week and the Planned Meal's night |
| dish_name | derived | The entry's title text, derived from the Planned Meal's recipe name |
| timing | derived | A generic evening dinner-time marker for the entry, since neither Household nor Planned Meal records a specific dinner hour |
| entry_identity_key | derived | The stable key (household, night_date) that identifies "this night's entry" independent of which recipe currently occupies it |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-21.SPEC-003 | Weekly Dinner Calendar Sync | On every sync evaluation: calls this spec's derivation and identity rules to decide whether a night's content is new, unchanged, or updated |
| FEAT-21.SPEC-002 | Family Calendar Integration | Carries the already-derived content across the boundary to the calendar capability; does not re-derive it |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| night_date | Must resolve to exactly one calendar date within the Weekly Plan's week, matching the Planned Meal's night; system-derived, never entered by any role | Always | On every derivation (FEAT-21.SPEC-003 evaluation) | N/A -- system-computed; a night that cannot resolve to a date (e.g., an incomplete or unapproved plan) is simply not yet eligible for sync | Yes |
| dish_name | Must be non-empty, taken directly from the Planned Meal's recipe name; system-derived, never entered or edited by any role for this entry | Always | On every derivation | N/A -- system-computed; recipe name is itself a required field on the Recipe entity (dependency map), so this never resolves to empty for a valid Planned Meal | Yes |
| timing | Fixed generic evening marker; does not vary by household, night, or recipe | Always | On every derivation | N/A -- system-computed, constant value; no per-household or per-recipe timing exists to derive from (neither Household nor Planned Meal records a specific dinner hour) | Yes |
| entry_identity_key | Must be composed of (household, night_date) only -- never includes the recipe, dish name, or any other content field | Always | On every derivation | N/A -- system-computed; this is the rule that keeps the identity stable across a swap (see Cross-Field Rules) | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Identity is content-independent | entry_identity_key, dish_name | entry_identity_key never changes when dish_name changes for the same night (e.g., after a swap); only night_date and household determine identity | N/A -- structurally enforced; FEAT-21.SPEC-003 always resolves a night to its existing entry_identity_key before comparing content, so a changed dish_name alone can never produce a new identity |
| One dish name per night | night_date, dish_name | At most one dish_name is associated with a given night's entry at any time -- a swap replaces the prior dish_name in the derivation, it never adds a second | N/A -- structurally enforced by the Weekly Plan's own one-dinner-per-night rule (dependency map, Planned Meal: "at most one dinner per night"), which this spec inherits rather than re-validates |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Derive calendar entry content for a night | No household role -- system-automation only (FEAT-21.SPEC-003) | Fires only during a sync evaluation for a connected household | Not offered as a manual action to any role; no household member edits or authors calendar entry content directly -- content always traces back to whichever feature (FEAT-03, FEAT-04, FEAT-23) wrote the underlying Planned Meal |
| View derived entry content in-app | No household role -- this content has no in-app surface | Always | Not shown anywhere in the product beyond FEAT-21.SPEC-001's connection status, per the feature's Data Notes ("Displayed: N/A within the product beyond a connection status"); every role views the resulting entry only outside the product, on the household's own calendar |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| night_date | The calendar date obtained by combining the Weekly Plan's week with the Planned Meal's night (e.g., "Tuesday" within the week of March 3 resolves to March 4) | Evaluated on every sync run for every dinner night in scope | No |
| dish_name | Set to the current Planned Meal's recipe name at evaluation time; recomputed on every evaluation so a swap is always reflected | Evaluated on every sync run | No -- no household role edits an entry's dish name directly; changing it means swapping or changing the underlying Planned Meal (FEAT-04, FEAT-23), which this spec then re-derives from |
| timing | A fixed, generic evening dinner-time marker (never a specific clock time) | Applied to every entry, always | No -- no per-night or per-household dinner time exists in the product to derive a specific hour from |
| entry_identity_key | Composed once, the first time a given night is synced, from (household, night_date); reused unchanged for every later evaluation of the same night, even across a swap | Set on first sync for a night; read (never recomputed) on every later evaluation of that same night | No |
| Content Exclusion (derivation constraint, not a stored field) | The entry never carries vegetarian_option, cook_time, rough_cost, pantry_callout, swap_history, allergy or Dietary Rule data, or any member-identifying detail -- only night_date, dish_name, and timing are ever placed on the entry (feature's Non-Functional Notes, Data sensitivity/privacy) | Applied to every derivation, always | No -- this exclusion is not user-configurable; it is a fixed product decision to keep children's and household-sensitive data off content the family sees outside the product |

## Business Rules

- **Stable per-night identity enables "update, not duplicate":** Because entry_identity_key depends only on (household, night_date) and never on dish_name, FEAT-21.SPEC-003 can always tell whether a night already has an entry and update it, rather than ever creating a second entry for the same night (Key Capabilities: "Keep it current").
- **Content minimization is a fixed rule, not a per-household setting:** No household, including Maya, can configure additional fields onto a synced entry; the excluded-field list in Defaults and Derivations applies identically to every household, keeping children's data (ASMP-26) and cost/allergy detail out of content visible outside the product.
- **Derivation always reflects the current Planned Meal, never a cached snapshot:** Every sync evaluation re-derives dish_name from the Planned Meal's current recipe, so a swap is always picked up on the next evaluation (FEAT-21.SPEC-003) rather than requiring a manual refresh.
- **A night with no dinner has no entry content to derive:** If a night in the evaluation window holds no dinner Planned Meal (cleared, or not yet picked), no Calendar Entry content is derived for it, and FEAT-21.SPEC-003 takes no create/update action for that night.

## Edge Cases

- **A dinner is swapped twice in quick succession before either sync completes** -- Each evaluation re-derives dish_name from whatever recipe is current at that moment; the entry_identity_key is unaffected by either swap, so both evaluations target the same existing entry and the final synced content reflects the most recent recipe.
- **A recipe name is unusually long** -- The full recipe name is used as dish_name without truncation by this rule; any display-length handling on the external calendar is outside the product's control and outside this spec's scope.
- **The Weekly Plan's week boundary changes (e.g., a plan is rebuilt for a different week)** -- night_date is recomputed from the new week; if this produces a different calendar date for what was "the same slot," a new entry_identity_key results and a new entry is created rather than the old one being reinterpreted, since identity is date-based, not slot-based.
- **A night's dinner is cleared and a different dinner is later placed in the same night** -- The existing entry_identity_key for that night_date is reused (identity depends on night_date, not on which dinner occupies it), so the new dinner's content updates the existing entry rather than creating a second one.
- **Two different households happen to share the same calendar date for their respective dinners** -- entry_identity_key includes household, so each household's entry is fully independent; no cross-household collision can occur.
- **Maya attempts to view or edit a calendar entry's derived content from within the product** -- No such control exists anywhere in the product (Authorization Rules); the only in-app surface for this feature is the connection status on FEAT-21.SPEC-001.

## Acceptance Criteria

**FEAT-21.SPEC-005-AC-01:** Given a connected household's Weekly Plan has a dinner for Tuesday of the current week, when FEAT-21.SPEC-003 derives that night's content, then night_date resolves to Tuesday's calendar date and dish_name is set to that dinner's recipe name.

**FEAT-21.SPEC-005-AC-02:** Given a synced night's dinner is swapped to a different recipe, when FEAT-21.SPEC-003 next evaluates that night, then dish_name is re-derived to the new recipe's name while entry_identity_key stays exactly as it was.

**FEAT-21.SPEC-005-AC-03:** Given a night is synced for the first time, when entry_identity_key is composed, then it is derived only from (household, night_date), never from dish_name or any other content field.

**FEAT-21.SPEC-005-AC-04:** Given a night's dinner is cleared and later replaced with a different dinner, when FEAT-21.SPEC-003 evaluates the replacement, then the existing entry_identity_key for that night_date is reused and the entry is updated rather than a new one created.

**FEAT-21.SPEC-005-AC-05:** Given any dinner Planned Meal in a connected household's plan, when its content is derived, then the resulting Calendar Entry carries only night_date, dish_name, and timing -- never vegetarian_option, cook_time, rough_cost, pantry_callout, swap_history, allergy data, or any member-identifying detail.

**FEAT-21.SPEC-005-AC-06:** Given a night in the evaluation window has no dinner Planned Meal, when FEAT-21.SPEC-003 evaluates that night, then no Calendar Entry content is derived for it.

**FEAT-21.SPEC-005-AC-07:** Given two different connected households each have a dinner on the same calendar date, when their respective entry_identity_keys are composed, then the household reference keeps the two identities fully distinct.

**FEAT-21.SPEC-005-AC-08:** Given a Weekly Plan is rebuilt so a night's slot maps to a different calendar date than before, when night_date is recomputed, then a new entry_identity_key results and a new entry is created for the new date rather than reinterpreting the prior entry.

**FEAT-21.SPEC-005-AC-09:** Given a dinner's recipe name is unusually long, when dish_name is derived, then the full recipe name is used without truncation by this rule.

**FEAT-21.SPEC-005-AC-10:** Given any household's synced entry, when timing is derived, then it is set to the same fixed, generic evening marker regardless of household, night, or recipe.

**FEAT-21.SPEC-005-AC-11:** Given Maya looks for a way to view or edit a synced entry's content from within the product, when she searches the household settings and Weekly Plan screens, then no such control exists anywhere -- the only in-app surface for this feature is the connection status on FEAT-21.SPEC-001.

**FEAT-21.SPEC-005-AC-12:** Given a dinner is swapped twice before either sync attempt completes, when both evaluations run, then both target the same entry_identity_key and the final synced dish_name reflects the most recently derived recipe.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 2 | 2 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
