---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-23.SPEC-004
spec_name: Safe-Choice Filtering & Placement Block
spec_slug: safe-choice-filtering-placement-block
parent_feature: FEAT-23
parent_feature_name: Manual Weekly Planning
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 24
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Safe-Choice Filtering & Placement Block

## Overview

**Name:** Safe-Choice Filtering & Placement Block
**ID:** FEAT-23.SPEC-004
**Type:** Logic/Rule
**Purpose:** Filters every candidate recipe shown in manual planning through the household's allergy/religious-rule check, marks ineligible ones with a plain, member-specific reason, blocks their placement or suggestion, and fails closed on incomplete ingredient data.
**Parent Feature:** FEAT-23 -- Manual Weekly Planning
**Governed Entity:** Recipe (as evaluated for one household's eligibility to be placed or suggested in that household's plan)

## Scope and Non-Goals

**In Scope:**
- Determining, for every candidate recipe shown by FEAT-23.SPEC-002 or FEAT-23.SPEC-003, whether it is eligible for this household
- Marking an ineligible recipe with a plain reason naming the specific member and the rule it breaks
- Blocking placement or suggestion of any ineligible recipe
- Excluding a recipe with incomplete ingredient data from the candidate list entirely (fail-closed, XBR-01)
- Authorization for who may view candidate eligibility and who may act on an eligible recipe (place directly vs. send as a suggestion)

**Non-Goals:**
- Computing the pass/fail safety determination itself -- owned by FEAT-02 (Dietary Rules & Allergy Safety Engine), which runs the actual check against Dietary Rule data; this spec consumes FEAT-02's determination and governs how manual planning's two screens present and enforce it, per feature-dependency-map.md's authority column for Dietary Rule
- Storing or displaying raw Dietary Rule records -- excluded per feature-dependency-map.md's Data Sensitivity note for Dietary Rule: this feature reads safety status only through FEAT-02's safety engine and never stores or displays the underlying allergy or religious-rule data itself
- Handling a safety-concern report on an already-placed meal -- owned by FEAT-02 (Report a safety concern) per XBR-08; this spec governs the candidate-list filter that runs before placement, not post-placement reporting

## Governed Entity

**Entity:** Recipe
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | Recipe's name |
| ingredients | list (quantity + unit per item) | Complete ingredient data is required to pass the safety check |
| steps | text | Method text |
| cook_time | number | Required for schedule fit |
| rough_cost | number | Shown in the household's currency |
| dietary_badges | derived | Computed per household by FEAT-02 when viewed |
| origin | enum | Starter library or imported (with source link) |
| owning_household | reference | Imported recipes only; starter recipes belong to no household |
| prep_requirements | text | Early-prep needs; not evaluated by this spec |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-23.SPEC-002 | Pick / Change a Recipe | On every candidate-list load and search, before any recipe can be selected for placement |
| FEAT-23.SPEC-003 | Suggest a Pick | On every candidate-list load and search, before any recipe can be selected to send as a suggestion |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| ingredients | Every ingredient must carry complete quantity and unit data before the recipe can be checked; a recipe with any incomplete ingredient entry is excluded from the candidate list entirely rather than shown unchecked | Always | On every candidate-list load | "This recipe can't be checked for safety yet and isn't shown as an option." (shown only if the household ever asks why a known recipe is missing; the default behavior is silent exclusion) | Yes |
| name | No validation beyond data type | Always | -- | -- | -- |
| steps | No validation beyond data type | Always | -- | -- | -- |
| cook_time | No validation beyond data type | Always | -- | -- | -- |
| rough_cost | No validation beyond data type | Always | -- | -- | -- |
| dietary_badges | No validation beyond data type -- this is a derived, computed field (see Defaults and Derivations); it is never entered by any user | Always | -- | -- | -- |
| origin | No validation beyond data type | Always | -- | -- | -- |
| owning_household | No validation beyond data type | Always | -- | -- | -- |
| prep_requirements | No validation beyond data type | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Hard-rule match against household members | Recipe.ingredients (this entity); Dietary Rule.allergen / rule_kind / strength (read via FEAT-02, not stored here) | If any ingredient matches any household member's hard rule (allergy or religious rule), the recipe is ineligible for this household; a soft dislike never blocks | "Not safe for {member name} -- contains {allergen or restricted ingredient}." |
| Vegetarian per-person hard filter | Recipe.ingredients; Dietary Rule (vegetarian setting, per member) | If a shared meal recipe is not vegetarian and any member's vegetarian setting applies with no vegetarian-option variant on the recipe, the recipe is ineligible for that member's presence at the meal | "Not suitable for {member name}'s vegetarian setting -- no vegetarian option available for this recipe." |
| Open safety-concern exclusion | Recipe (identity); Household's open Support Requests (read via FEAT-02, XBR-08) | A recipe under an open safety-concern report for this household is excluded from the candidate list for the duration the report is open | "This recipe is under review after a reported safety concern and isn't available to pick right now." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View candidate list with eligibility markings | Maya, Sam | Always | -- |
| Place a recipe directly (create or replace a night's pick) | Maya | Recipe must be eligible for this household (passed the check, ingredient data complete, no open safety-concern exclusion) | Ineligible: the Place/Replace action is not shown on the card; the card instead shows the plain ineligibility reason. A placement attempted against a recipe that fails re-check in the background (e.g., mid-week rule tightening) is rejected and the reason is shown inline |
| Place a recipe directly | Sam | Never -- placement is the organiser's action alone (scope-boundaries.md SC-04; Access Matrix Manual Planning: Own-only) | No Place/Replace action exists anywhere in Sam's screens (FEAT-23.SPEC-003); his terminal action is "Send as suggestion" |
| Send a recipe as a suggestion | Sam | Recipe must be eligible for this household, and Sam must have no other open suggestion for the target night (FEAT-23.SPEC-005) | Ineligible: the "Send as suggestion" action is not shown on the card, and the plain reason is shown instead. Already has an open suggestion for the night: the screen is not reachable for that night (FEAT-23.SPEC-001 shows the pending status), and a send attempted through a stale screen state is rejected with "You already suggested a pick for {Night} -- wait for Maya's decision or its lapse." |
| Send a recipe as a suggestion | Maya | Never -- the organiser places directly and never suggests to herself | The suggestion screen (FEAT-23.SPEC-003) is not part of Maya's navigation |
| View raw Dietary Rule data behind an eligibility badge or reason | Maya, Sam | Never for either role -- this feature reads Dietary Rule only through FEAT-02's safety engine and never stores or displays the underlying records | No control exists in manual planning to view raw Dietary Rule data; only the plain, member-and-rule-named reason is shown |
| View or act on the candidate list | Jordan (young kid profile, no login -- MVP) | Never -- Manual Planning access is None | No candidate list is reachable; no login exists for this profile |
| View or act on the candidate list | Jordan (older kid, limited login -- Later) | Never -- Manual Planning access is None | No candidate list is reachable |
| View or act on the candidate list | Riley (Operator, support -- from v1) | Never -- Manual Planning access is None | No candidate list is reachable |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| dietary_badges (eligibility outcome) | Computed by FEAT-02's safety engine against every active hard Dietary Rule of every household member, evaluated per household, per recipe | Every candidate-list load and search in FEAT-23.SPEC-002 and FEAT-23.SPEC-003 | No -- the determination is never user-editable |
| Ineligibility reason (display text) | Derived from the specific member and hard rule the recipe fails; when multiple members or rules are broken, the reason names all of them, not only the first found | Whenever a candidate fails the check | No |
| Candidate-list inclusion | A recipe is included only if its ingredient data is complete and it carries no open safety-concern exclusion for the household; otherwise it is silently omitted rather than shown as a zero-eligibility card | Every candidate-list load | No |

## Business Rules

- XBR-01: every path onto the plan -- including manual picks and pick suggestions -- passes the same app-enforced allergy and religious-rule check before anyone sees it; the check fails closed (a recipe with incomplete ingredient data is excluded, never shown unchecked), and every eligible meal carries the "checked against allergies" badge and "always check labels" disclaimer.
- XBR-08: a recipe under an open safety-concern report is excluded from the household's candidate pool for the life of that report; this spec enforces that exclusion within manual planning's two screens.
- XBR-06: Sam may only send an eligible recipe as a suggestion, never place one directly; Maya alone places and changes picks.
- Soft dislikes (Dietary Rule strength: soft) never block placement or suggestion under this spec -- only hard rules (allergy, religious rule) and the per-person vegetarian setting are enforced here.

## Edge Cases

- **Recipe has zero listed ingredients** -- Treated as incomplete ingredient data; excluded from the candidate list entirely (fails closed).
- **An ingredient's allergen tag cannot be matched to the standard allergen list (unrecognized or ambiguous entry)** -- Treated as unverifiable, not as safe; the recipe is excluded rather than assumed safe, consistent with fail-closed handling.
- **A household member whose hard rule made a recipe ineligible is removed from the household** -- The recipe is re-evaluated on the next candidate-list load; it may become eligible if no remaining member is affected by it.
- **A hard dietary rule tightens mid-session while a candidate list is open (XBR-02)** -- The list re-filters on the next load or search; a selection made in the same moment the rule tightened is re-checked before the triggering screen's placement or send completes, and is rejected if it now fails.
- **Two ingredients trigger two different members' allergies simultaneously** -- The ineligibility reason names both affected members and both rules broken, not only the first match found.
- **A recipe is eligible for every current member but a new member with a conflicting hard rule is added mid-session** -- The recipe re-evaluates as ineligible on the next candidate-list load; a placement already in flight at that moment is rejected on save with the standard ineligibility reason.
- **A vegetarian member is present and the recipe has a vegetarian-option variant** -- The recipe is eligible; the vegetarian-option variant is understood to be served to that member per the shared-meal vegetarian handling.

## Acceptance Criteria

**FEAT-23.SPEC-004-AC-01:** Given Maya is browsing candidates for Wednesday and Jordan is allergic to peanuts, when a candidate recipe contains peanuts, then it is shown ineligible with the reason "Not safe for Jordan -- contains peanuts," and no Place action is offered.

**FEAT-23.SPEC-004-AC-02:** Given Maya is browsing candidates and a recipe has complete ingredient data with no hard-rule conflicts for any household member, when the candidate list loads, then the recipe shows the "checked against allergies" badge and a Place action.

**FEAT-23.SPEC-004-AC-03:** Given a recipe has an ingredient missing quantity or unit data, when the candidate list loads for any household, then that recipe does not appear in the list at all.

**FEAT-23.SPEC-004-AC-04:** Given a candidate recipe breaks both Jordan's peanut allergy and Sam's religious dietary rule, when it is shown ineligible, then its reason names both Jordan's allergy and Sam's rule.

**FEAT-23.SPEC-004-AC-05:** Given Maya (Organiser) views an eligible recipe on FEAT-23.SPEC-002, when she looks for a Place action, then it is present and selectable.

**FEAT-23.SPEC-004-AC-06:** Given Sam (Other Adult Member) views an eligible recipe on FEAT-23.SPEC-003, when he looks for a Send as suggestion action, then it is present and selectable, and no Place action exists anywhere in his screens.

**FEAT-23.SPEC-004-AC-07:** Given Sam already has an open suggestion for Saturday, when he would otherwise reach the candidate list for Saturday, then the screen shows his pending suggestion's status instead of an eligible-recipe send action.

**FEAT-23.SPEC-004-AC-08:** Given Jordan is a young kid profile with no login, when any attempt is made to reach the candidate list on Jordan's behalf, then no such access exists -- the profile has no login and no Manual Planning access.

**FEAT-23.SPEC-004-AC-09:** Given Maya or Sam views an ineligibility reason, when they look for the underlying Dietary Rule detail, then no control exists to view it -- only the plain, member-and-rule-named reason is shown.

**FEAT-23.SPEC-004-AC-10:** Given a recipe is under an open safety-concern report for the household, when the candidate list loads, then that recipe is excluded with the reason "This recipe is under review after a reported safety concern and isn't available to pick right now."

**FEAT-23.SPEC-004-AC-11:** Given a shared-meal recipe is not vegetarian and a vegetarian household member has no vegetarian-option variant available on that recipe, when the candidate list loads, then the recipe is shown ineligible with a reason naming that member's vegetarian setting.

**FEAT-23.SPEC-004-AC-12:** Given a household member has a soft dislike (not a hard rule) matching an ingredient, when the candidate list loads, then the recipe is still shown eligible -- soft dislikes never block placement.

**FEAT-23.SPEC-004-AC-13:** Given a household member's allergy is tightened while a candidate list is already open, when the organiser or Sam attempts to select a recipe that has since become ineligible, then the selection is rejected and the ineligibility reason is shown.

**FEAT-23.SPEC-004-AC-14:** Given Riley (Operator, support) has an open support session for a household, when Riley looks for access to that household's manual-planning candidate list, then none exists -- Manual Planning access is None for Riley.

**FEAT-23.SPEC-004-AC-15:** Given a recipe's ingredient data is later completed after previously being excluded, when the candidate list next loads, then the recipe is evaluated normally and appears as eligible or ineligible based on the completed data.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 9 | 9 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |
