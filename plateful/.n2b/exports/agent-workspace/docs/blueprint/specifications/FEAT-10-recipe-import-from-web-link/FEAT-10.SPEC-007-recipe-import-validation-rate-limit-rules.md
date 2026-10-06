---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-10.SPEC-007
spec_name: Recipe Import Validation & Rate Limit Rules
spec_slug: recipe-import-validation-rate-limit-rules
parent_feature: FEAT-10
parent_feature_name: Recipe Import from Web Link
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 18
acceptance_criteria_count: 22
---

# Logic/Rule Spec: Recipe Import Validation & Rate Limit Rules

## Overview

**Name:** Recipe Import Validation & Rate Limit Rules
**ID:** FEAT-10.SPEC-007
**Type:** Logic/Rule
**Purpose:** Governs link well-formedness, ingredient/step length caps, the minimum-one-ingredient save requirement, and the 30-import-per-week misuse guard for the imported slice of the Recipe entity.
**Parent Feature:** FEAT-10 -- Recipe Import from Web Link
**Governed Entity:** Recipe (imported slice)

## Scope and Non-Goals

**In Scope:**
- Field-level validation for every field this feature owns on an imported Recipe record (name, source link, ingredients, steps, cook_time)
- The minimum-one-ingredient requirement for saving an import
- The 30-import-per-week misuse guard on the Import action
- Authorization rules for Import, Edit, and Remove on the imported slice of the Recipe entity
- Default and derived values this feature sets on create (origin, owning_household)

**Non-Goals:**
- Validating starter recipe content -- starter recipes are seeded and maintained by FEAT-08.SPEC-004 (Starter Recipe Content Seeding & Maintenance); this spec governs only the imported slice households create.
- Determining whether an ingredient is safe against a household's allergies or religious rules -- that determination belongs to FEAT-02 (Dietary Rules & Allergy Safety Engine, XBR-01); this spec only requires that ingredient data be complete enough for FEAT-02's check to run, and defines the save-time consequence of an ingredient FEAT-02 cannot recognize as an eligibility exclusion, not a save-blocking validation failure.
- A lifetime or tier-based cap on the number of imports -- explicitly excluded per product-features.md's Rationale: research flagged a lifetime cap as a hard limit competitors' users criticized, so the founder deliberately rejected one; only the 30-per-week misuse guard defined here applies.
- Validating the recipe's rough cost or dietary badges -- feature-overview.md's Shared Context states this feature never writes rough_cost or dietary_badges for imported content; those fields carry no validation obligation from this spec.

## Governed Entity

**Entity:** Recipe (imported slice)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | The recipe's title |
| source_link | text | The web address the recipe was imported from (link-based imports only) |
| ingredients | list (quantity + unit + name per line) | The recipe's ingredient lines |
| steps | text | The recipe's method, in order |
| cook_time | number | Minutes required to cook the recipe, required for schedule fit |
| rough_cost | number | Shown in the household's currency |
| dietary_badges | derived | Computed live by FEAT-02 against the household's dietary rules |
| origin | enum | starter library or imported (with source link) |
| owning_household | reference | The household that owns this imported recipe |
| prep_requirements | text | Early-prep needs such as defrosting |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-10.SPEC-001 | Import by Link | Link well-formedness and weekly-limit checks on Import tap |
| FEAT-10.SPEC-002 | Review Extracted Recipe | Field-level validation on field blur and Save tap; duplicate/save authorization |
| FEAT-10.SPEC-003 | Manual Recipe Entry | Field-level validation on field blur and Save tap; duplicate/save authorization |
| FEAT-10.SPEC-004 | Edit Imported Recipe | Field-level validation on field blur and Save tap; edit/remove authorization; concurrent-edit conflict handling |
| FEAT-10.SPEC-006 | Duplicate Import Detection | Reads owning_household boundary and source_link/name fields this spec defines |
| FEAT-10.SPEC-008 | Safety Re-check on Import Save/Edit | Reads the completed, validated ingredient data this spec requires before triggering the safety re-check |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | Required, non-empty, max 120 characters | Always | On blur | "Recipe name is required" / "Recipe name must be 120 characters or fewer" | Yes |
| source_link | Must be a well-formed web address (a valid scheme and host) | Import started from FEAT-10.SPEC-001 (link-based import); not applicable to manual entry, which carries no source_link | On Import tap (FEAT-10.SPEC-001) | "Enter a valid web link" | Yes |
| ingredients | At least one ingredient line required | Always | On blur (list changes) and on submit | "At least one ingredient is required" | Yes |
| ingredients (per line: quantity) | Required, positive number | Always, per line | On blur | "Enter a quantity for this ingredient" | Yes |
| ingredients (per line: unit) | Required, non-empty | Always, per line | On blur | "Enter a unit for this ingredient" | Yes |
| ingredients (per line: name) | Required, non-empty, max 80 characters | Always, per line | On blur | "Enter an ingredient name" / "Ingredient name must be 80 characters or fewer" | Yes |
| steps | Max 4,000 characters | Always | On blur | "Steps must be 4,000 characters or fewer" | Yes |
| cook_time | Required, positive whole number of minutes, max 600 | Always | On blur | "Enter the cook time in minutes" / "Cook time must be 600 minutes or fewer" | Yes |
| rough_cost | No validation beyond data type -- this feature never sets this field for imported content | Always | -- | -- | -- |
| dietary_badges | No validation beyond data type -- computed live by FEAT-02; this feature never writes it | Always | -- | -- | -- |
| prep_requirements | No validation beyond data type -- this feature does not capture or set this field for imported content | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Source link required only for link-based imports | source_link, entry path | source_link is required and validated only when the save originated from FEAT-10.SPEC-001/FEAT-10.SPEC-002 (link-based); a save originating from FEAT-10.SPEC-003 (manual entry) carries no source_link and is not blocked by its absence | "Enter a valid web link" (link-based path only; not shown on the manual entry path) |
| Ingredient completeness for the safety check | ingredients (quantity, unit, name per line) | An ingredient with a name the safety engine cannot recognize does not block the save (this rule's own validation only requires quantity, unit, and name to be present); it instead triggers an eligibility exclusion in FEAT-10.SPEC-008/FEAT-02 (XBR-01), which is out of this spec's authority | N/A -- no save-blocking error; the exclusion is communicated as an eligibility state, not a validation failure |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Import a recipe (create) | Maya (Organiser), Sam (Other Adult Member) | Always, subject to the weekly import limit below | -- |
| Import a recipe (create) | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this profile; there is no path to any import screen |
| Import a recipe (create) | Jordan (older kid, limited login -- Later) | Never | "Import from link" is not shown (Recipe Library access is View, not Full); a direct attempt shows "Importing recipes isn't available on this profile." |
| Import a recipe (create) | Riley (Operator, support) | Never | No import entry point exists in the read-only support view |
| Edit an imported recipe | Maya (Organiser), Sam (Other Adult Member) | Always, for any imported recipe owned by their household (not ownership-restricted to the importing member -- the Recipe Library column of the Access Matrix grants both roles Full access to the household's shared pool) | -- |
| Edit an imported recipe | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this profile |
| Edit an imported recipe | Jordan (older kid, limited login -- Later) | Never | "Edit" is not shown (Recipe Library access is View, not Full); a direct attempt shows "Editing recipes isn't available on this profile." |
| Edit an imported recipe | Riley (Operator, support) | Never | Edit controls are not shown in the read-only support view |
| Remove an imported recipe | Maya (Organiser), Sam (Other Adult Member) | Always, for any imported recipe owned by their household | -- |
| Remove an imported recipe | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this profile |
| Remove an imported recipe | Jordan (older kid, limited login -- Later) | Never | "Remove recipe" is not shown (Recipe Library access is View, not Full) |
| Remove an imported recipe | Riley (Operator, support) | Never | Remove controls are not shown in the read-only support view |

**Rate limit condition on Import (create):** For Maya and Sam, the Import action is allowed only while the household's import count for the current rolling week is below 30. At 30 imports in the current rolling week, further imports are blocked for every household member until the rolling week resets, with the denied behavior: "You've reached this week's import limit (30). You can import more recipes once next week starts." No lifetime cap applies to either household member (product-features.md's Validation & Limits field, and the explicit product decision recorded in Non-Goals).

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| origin | Set to "imported" | On create (FEAT-10.SPEC-002 or FEAT-10.SPEC-003 save) | No |
| owning_household | Set to the saving member's household | On create | No |
| source_link | Set to the link the member submitted on FEAT-10.SPEC-001 | On create, link-based path only | No |
| source_link | Left unset | On create, manual-entry path (FEAT-10.SPEC-003) | Not applicable -- no value is captured to override |

## Business Rules

- Field validation (this spec) runs before duplicate detection (FEAT-10.SPEC-006) -- invalid data is never checked for duplicates.
- The 30-import-per-week guard counts imports across both link-based and manual-entry saves for the household, reset on a rolling weekly basis; it is the only import limit the product defines, per the explicit no-lifetime-cap decision in product-features.md's Rationale.
- An ingredient with a name the safety engine cannot recognize does not block the save; it results in the recipe being excluded from every plan candidate path until clarified, per XBR-01's fail-closed posture, enforced through FEAT-10.SPEC-008 and FEAT-02, not through this spec's field validation.
- All validation rules apply identically whether the save originates from FEAT-10.SPEC-002 (review after extraction), FEAT-10.SPEC-003 (manual entry), or FEAT-10.SPEC-004 (later edit) -- the product definition establishes no save-path-specific exceptions beyond the source_link rule above.
- Authorization for Edit and Remove is not ownership-restricted to the member who originally imported the recipe -- both Maya and Sam hold Full access to the household's shared Recipe Library per the Access Matrix, so either may edit or remove a recipe the other imported.

## Edge Cases

- **Recipe name at exactly 120 characters** -- Passes validation. 121 characters shows the length error.
- **Ingredient name at exactly 80 characters** -- Passes validation. 81 characters shows the length error.
- **Steps at exactly 4,000 characters** -- Passes validation. 4,001 characters shows the length error.
- **Cook time at exactly 600 minutes** -- Passes validation. 601 minutes shows the length error; 0 or a negative value shows the required/positive-number error.
- **Household at exactly 29 imports for the current week** -- The 30th import is allowed; the household is not blocked until the count reaches 30.
- **Household at exactly 30 imports for the current week** -- The next import attempt is blocked with the weekly-limit message; the count does not reset until the rolling week boundary passes.
- **Ingredient list has entries with quantity and unit but an ingredient name the safety engine cannot recognize** -- Save is not blocked by this spec's own field rules (name is present and within length); the recipe is excluded from plan eligibility by FEAT-10.SPEC-008/FEAT-02 instead.
- **Manual entry save with no source_link value at all** -- Passes validation; the cross-field rule explicitly exempts the manual-entry path from the source_link requirement.
- **Ownership of an imported recipe "changes" conceptually when Sam edits a recipe Maya originally imported** -- No authorization boundary is crossed: Edit and Remove are not ownership-gated between Maya and Sam, so this is simply an allowed action, not an edge case requiring special handling.
- **A household member attempts to import while at 30/30 and simultaneously another member's in-flight import (started just under the limit) completes, pushing the count to 30 before the first member's check runs** -- The check reads the count at the moment each Import action's validation runs; a member whose check runs after the count reaches 30 is blocked, even if their submission started slightly before the count-pushing import completed. This is a first-decision-wins outcome consistent with the automation processing order in FEAT-10.SPEC-006.

## Acceptance Criteria

**FEAT-10.SPEC-007-AC-01:** Given Maya leaves the recipe name empty on FEAT-10.SPEC-002, when she moves to the next field, then she sees "Recipe name is required."

**FEAT-10.SPEC-007-AC-02:** Given Maya enters a recipe name of exactly 120 characters, when she moves to the next field, then no error is shown.

**FEAT-10.SPEC-007-AC-03:** Given Maya enters a recipe name of 121 characters, when she moves to the next field, then she sees "Recipe name must be 120 characters or fewer."

**FEAT-10.SPEC-007-AC-04:** Given Sam pastes a malformed link on FEAT-10.SPEC-001, when he taps Import, then he sees "Enter a valid web link" and no extraction is requested.

**FEAT-10.SPEC-007-AC-05:** Given Sam is on FEAT-10.SPEC-003 (Manual Recipe Entry), when he saves a recipe with no source link entered, then no "Enter a valid web link" error appears, since manual entries carry no source_link requirement.

**FEAT-10.SPEC-007-AC-06:** Given Maya removes every ingredient line on FEAT-10.SPEC-002, when she taps Save, then she sees "At least one ingredient is required" and the save does not proceed.

**FEAT-10.SPEC-007-AC-07:** Given Maya leaves an ingredient's quantity empty, when she moves to the next field, then she sees "Enter a quantity for this ingredient."

**FEAT-10.SPEC-007-AC-08:** Given Sam enters an ingredient name of exactly 80 characters, when he moves to the next field, then no error is shown.

**FEAT-10.SPEC-007-AC-09:** Given Sam enters steps text of exactly 4,000 characters, when he moves to the next field, then no error is shown.

**FEAT-10.SPEC-007-AC-10:** Given Maya enters steps text of 4,001 characters, when she moves to the next field, then she sees "Steps must be 4,000 characters or fewer."

**FEAT-10.SPEC-007-AC-11:** Given Sam enters a cook time of 0 minutes, when he moves to the next field, then he sees "Enter the cook time in minutes."

**FEAT-10.SPEC-007-AC-12:** Given Maya enters a cook time of exactly 600 minutes, when she moves to the next field, then no error is shown.

**FEAT-10.SPEC-007-AC-13:** Given Sam's household has imported 29 recipes this week, when he submits a 30th, then the import is allowed and proceeds to extraction.

**FEAT-10.SPEC-007-AC-14:** Given Maya's household has imported 30 recipes this week, when any household member attempts a 31st import, then they see "You've reached this week's import limit (30). You can import more recipes once next week starts." and no extraction is requested.

**FEAT-10.SPEC-007-AC-15:** Given a household has been active for many months with no lifetime import cap, when they attempt an import within this week's allowance, then the import is allowed regardless of their total imports to date.

**FEAT-10.SPEC-007-AC-16:** Given Maya (Organiser) is on FEAT-10.SPEC-002, when she saves a recipe that passes all field rules, then the save proceeds to duplicate detection (FEAT-10.SPEC-006).

**FEAT-10.SPEC-007-AC-17:** Given the older-kid limited-login role (Later) attempts to import a recipe, when the attempt is made, then it is denied since Import is never allowed for this role, and "Importing recipes isn't available on this profile." is shown if the screen is reached directly.

**FEAT-10.SPEC-007-AC-18:** Given Riley (Operator) is viewing a household's recipe library, when Riley looks for an import, edit, or remove control, then none is shown, since all three actions are Never for this role.

**FEAT-10.SPEC-007-AC-19:** Given Sam (Other Adult Member) opens a recipe Maya originally imported, when he edits and saves it, then the save is allowed, since Edit is not ownership-restricted between Maya and Sam.

**FEAT-10.SPEC-007-AC-20:** Given Maya saves a recipe with an ingredient named clearly (e.g., "chicken breast") that the safety engine cannot recognize, when the save completes, then it is not blocked by this spec's field rules -- the recipe saves and is excluded from plan eligibility by FEAT-10.SPEC-008/FEAT-02 instead.

**FEAT-10.SPEC-007-AC-21:** Given Maya saves a recipe from FEAT-10.SPEC-002 (link-based), when the record is created, then origin is set to "imported", owning_household is set to Maya's household, and source_link is set to the submitted link, none of which she can override.

**FEAT-10.SPEC-007-AC-22:** Given Sam saves a recipe from FEAT-10.SPEC-003 (manual entry), when the record is created, then origin is set to "imported", owning_household is set to Sam's household, and source_link remains unset.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 11 | 11 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 12 | 12 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 10 | 10 |
