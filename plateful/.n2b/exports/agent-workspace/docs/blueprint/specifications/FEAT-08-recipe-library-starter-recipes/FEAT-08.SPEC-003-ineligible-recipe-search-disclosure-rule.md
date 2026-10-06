---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-08.SPEC-003
spec_name: Ineligible Recipe Search Disclosure Rule
spec_slug: ineligible-recipe-search-disclosure-rule
parent_feature: FEAT-08
parent_feature_name: Recipe Library (Starter Recipes)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 9
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Ineligible Recipe Search Disclosure Rule

## Overview

**Name:** Ineligible Recipe Search Disclosure Rule
**ID:** FEAT-08.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs when a recipe that fails a household's dietary/allergy rules, or whose ingredient data cannot be fully verified, is still surfaced with a plain ineligibility explanation on a direct library search or open -- instead of being silently excluded as it is everywhere else on the plan.
**Parent Feature:** FEAT-08 -- Recipe Library (Starter Recipes)
**Governed Entity:** Recipe (as surfaced on direct household search or open within FEAT-08.SPEC-001 and FEAT-08.SPEC-002)

## Scope and Non-Goals

**In Scope:**
- The disclosure decision -- shown-with-explanation versus excluded -- for a recipe reached by a household member's own direct search or open action in the recipe library
- The exact wording pattern used for the ineligibility explanation
- Which roles can see a disclosed ineligible recipe, and the exact denied behavior when a household member tries to place it on the plan anyway
- Boundary between this feature's direct-search disclosure and FEAT-02's silent-exclusion behavior everywhere else a recipe could reach the plan

**Non-Goals:**
- Determining whether a recipe passes or fails the safety check itself -- that determination is FEAT-02's authority (XBR-01); this spec governs only whether an already-determined ineligible recipe is disclosed or hidden on direct search.
- Disclosure behavior within automatically assembled candidate pools (AI plan generation, swap alternatives, voting options, re-used past weeks) -- those paths keep FEAT-02's silent exclusion (XBR-01) unchanged; this spec applies only to a household member's own direct search or open action in the library.
- Nutrition or diet-quality basis for ineligibility -- excluded per scope-boundaries.md SC-06: ineligibility under this spec is always an allergy/religious-rule failure or unverifiable ingredient data, never a health or diet-quality judgment.

## Governed Entity

**Entity:** Recipe
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| name | text | Recipe name, shown on the disclosed card/detail regardless of eligibility |
| ingredients | list (quantity + unit per entry) | Complete ingredient data is required for FEAT-02's safety check to pass a recipe; incomplete data is one of the two ineligibility grounds this spec discloses |
| steps | text | Method text; unaffected by this rule -- always shown |
| cook_time | text/number | Unaffected by this rule -- always shown |
| rough_cost | number | Unaffected by this rule -- always shown |
| dietary_badges | derived | Computed live by FEAT-02 against the household's dietary rules; this spec reads its pass/fail outcome and, on failure, the specific reason(s) to disclose |
| origin | enum (starter library \| imported) | Unaffected by this rule -- both starter and imported recipes are subject to the same disclosure decision |
| owning_household | reference (imported recipes only) | Unaffected by this rule |
| prep_requirements | text | Unaffected by this rule -- always shown |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-08.SPEC-001 | Recipe Library Browse & Search | On every search or filter result: a recipe that fails FEAT-02's check for this household, or carries incomplete ingredient data, is disclosed with a plain reason instead of omitted |
| FEAT-08.SPEC-002 | Recipe Detail View | On screen open: if the opened recipe is ineligible for this household, the summary row renders the disclosure in place of dietary badges |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | No validation beyond data type | Always | -- | -- | -- |
| ingredients | Must be complete (every ingredient has a quantity and unit) for the recipe to pass FEAT-02's safety check | Evaluated whenever eligibility is determined | On every search result render and on detail-screen open | "Recipe details are incomplete for this household's safety check" (disclosed reason, not a form error) | No -- this is a disclosure condition, not a blocking input validation; the recipe itself is never edited from this spec's enforcing screens |
| steps | No validation beyond data type | Always | -- | -- | -- |
| cook_time | No validation beyond data type | Always | -- | -- | -- |
| rough_cost | No validation beyond data type | Always | -- | -- | -- |
| dietary_badges | Read-only outcome of FEAT-02's determination; this spec does not validate or alter it, only reads its pass/fail result and reason(s) | Always | On every search result render and on detail-screen open | "Contains {allergen} -- not safe for {member}" (disclosed reason, reusing FEAT-02.SPEC-008's exact phrasing) | No -- disclosure, not a blocking input validation |
| origin | No validation beyond data type | Always | -- | -- | -- |
| owning_household | No validation beyond data type | Always | -- | -- | -- |
| prep_requirements | No validation beyond data type | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Direct-search disclosure | dietary_badges (FEAT-02's pass/fail outcome and reason), ingredients (completeness), origin (search context: direct household search/open vs. automatic candidate-pool assembly) | If the recipe fails FEAT-02's hard-rule check for this household, or its ingredient data cannot be fully verified, AND the household member reached it through a direct search or open action in FEAT-08.SPEC-001 or FEAT-08.SPEC-002 (not through AI plan generation, swap alternatives, voting options, or re-used past weeks), then the recipe is shown with a plain ineligibility explanation instead of excluded | "This recipe isn't eligible for your household. {reason(s)}" -- where {reason(s)} is one or more of "Contains {allergen} -- not safe for {member}" (per failing member) or "Recipe details are incomplete for this household's safety check" |
| Multiple simultaneous ineligibility reasons | dietary_badges, ingredients | When a recipe both fails a hard rule and has incomplete ingredient data, every applicable reason is listed, one line per reason, hard-rule violations listed before the incomplete-data reason | "This recipe isn't eligible for your household. Contains {allergen} -- not safe for {member}. Recipe details are incomplete for this household's safety check." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View an ineligible recipe under direct-search disclosure | Maya (Organiser), Sam (Other Adult Member), Jordan (older kid, limited login -- Later) | Always, when the recipe is reached by a direct search or open action per the Cross-Field Rule above | -- |
| View an ineligible recipe under direct-search disclosure | Jordan (young kid profile, no login -- MVP) | Never -- no login exists for this profile | The profile has no sign-in; there is no session in which any recipe, eligible or not, could be viewed |
| View an ineligible recipe under direct-search disclosure | Riley (Operator, support) | Only while an open Support Request for the household is active (XBR-14) | Outside an open Support Request, Riley has no access to any household's library, eligible or not |
| Place an ineligible recipe onto the plan (any slot, any tier) | Maya, Sam, Jordan (older kid, limited login -- Later), Jordan (young kid profile) | Never -- for every role, an ineligible recipe cannot be placed on the plan, per XBR-01 | No add-to-plan control exists on FEAT-08.SPEC-002 for any recipe (view-only); if a placement is attempted through another feature's pick action, it is refused with "This recipe isn't eligible for your household's plan. {reason(s)}" and the plan is unchanged |
| Bypass the disclosure and view an ineligible recipe as if it were eligible | All roles | Never -- no capability exists to suppress or override the disclosure for any role, including Maya | Not applicable -- no control for this exists anywhere in the product |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| dietary_badges (pass/fail outcome and reason) | Computed live by FEAT-02's safety determination against the household's current Dietary Rules and the recipe's ingredient data; this spec does not compute or alter the outcome, only the disclosure decision built on it | Every time the recipe is rendered on FEAT-08.SPEC-001 or FEAT-08.SPEC-002 | No -- the underlying determination is FEAT-02's authority (XBR-01); no role can override it |
| Disclosure reason text | Derived from FEAT-02's pass/fail reason(s), reusing FEAT-02.SPEC-008's exact allergen phrasing, plus the incomplete-data reason when ingredient data is incomplete | Every time an ineligible recipe is rendered under this rule | No |

## Business Rules

- XBR-01: the app-enforced safety check runs on every path onto the plan and fails closed; this spec never weakens that check -- it only changes whether a failing recipe is shown-with-explanation (direct search) or silently excluded (every other path).
- A recipe disclosed under this rule remains permanently excluded from the household's plan while it stays ineligible; disclosure is informational only and creates no path around FEAT-02's exclusion.
- This spec's disclosure applies identically to starter recipes and imported recipes -- the rule does not distinguish by origin.
- The disclosure reason reuses FEAT-02.SPEC-008's exact phrasing pattern rather than composing independent wording, keeping ineligibility language consistent everywhere it appears in the product.

## Edge Cases

- **A recipe fails for both a hard-rule violation and incomplete ingredient data** -- Both reasons are listed, hard-rule violations first, per the Cross-Field Rules table above.
- **A household with no hard dietary rules searches for a recipe with incomplete ingredient data** -- The recipe is still disclosed as ineligible on the incomplete-data ground alone; the absence of allergy rules does not make an unverifiable recipe eligible.
- **The same recipe appears in FEAT-04's swap-alternatives list and is separately found via a direct library search** -- In the swap-alternatives list it is silently excluded (XBR-01, out of this spec's scope); in the direct search result it is disclosed under this rule. The two behaviors are not a contradiction -- they are the documented boundary this spec defines.
- **A household's dietary rule changes (e.g., a new allergy) while an ineligible recipe's disclosure is on screen** -- The disclosed reason is not live-updated on the open screen; the next render (a fresh search or a fresh detail-screen open) reflects the current determination, consistent with FEAT-02's badges computing fresh at view time rather than being stored.
- **FEAT-08.SPEC-004 corrects a recipe's ingredient data, resolving the incomplete-data reason** -- On the next render, the recipe is re-evaluated by FEAT-02; if it now passes, it appears without the disclosure; if it still fails a hard rule, the disclosure shows only the remaining reason.
- **A household member attempts to place a disclosed ineligible recipe onto the plan through a feature outside this one (e.g., typing its name directly into a manual pick attempt)** -- The placement is refused with the exact denied message in the Authorization Rules table; the plan is unchanged.

## Acceptance Criteria

**FEAT-08.SPEC-003-AC-01:** Given Maya searches the library directly for a recipe that contains an allergen her household cannot eat, when the search returns that recipe, then it is shown with "This recipe isn't eligible for your household. Contains {allergen} -- not safe for {member}." instead of being omitted.

**FEAT-08.SPEC-003-AC-02:** Given Sam opens a recipe directly whose ingredient data cannot be fully verified, when the detail screen loads, then it shows "This recipe isn't eligible for your household. Recipe details are incomplete for this household's safety check."

**FEAT-08.SPEC-003-AC-03:** Given a recipe both violates a hard rule and has incomplete ingredient data, when Maya opens it directly, then both reasons are listed, the allergen violation first.

**FEAT-08.SPEC-003-AC-04:** Given the same ineligible recipe would also have appeared among FEAT-04's swap alternatives, when Maya reaches it through the swap-alternatives list instead of a direct search, then it does not appear at all (silent exclusion, XBR-01), unlike the direct-search case.

**FEAT-08.SPEC-003-AC-05:** Given Sam views a disclosed ineligible recipe, when he looks for a way to place it on the plan, then no such control exists on the recipe detail screen.

**FEAT-08.SPEC-003-AC-06:** Given a manual pick attempt for a disclosed ineligible recipe is made through another feature's pick action, when the attempt is made, then it is refused with "This recipe isn't eligible for your household's plan. {reason(s)}" and the plan is unchanged.

**FEAT-08.SPEC-003-AC-07:** Given the older-kid limited-login role (Later) searches the library directly, when a search result is ineligible for the household, then they see the same disclosure Maya and Sam would see.

**FEAT-08.SPEC-003-AC-08:** Given Jordan is a young kid profile with no login, when any attempt is made to view a disclosed ineligible recipe, then no session exists in which it could happen.

**FEAT-08.SPEC-003-AC-09:** Given Riley (Operator) is viewing a household's library without an open Support Request, when Riley attempts to see any recipe, eligible or not, then access is denied entirely, consistent with XBR-14.

**FEAT-08.SPEC-003-AC-10:** Given Riley (Operator) is viewing a household's library during an open Support Request, when a search result is ineligible, then Riley sees the same disclosure the household would see, read-only.

**FEAT-08.SPEC-003-AC-11:** Given a household has no dietary rules recorded, when Maya searches directly for a recipe with incomplete ingredient data, then it is still disclosed as ineligible on the incomplete-data ground.

**FEAT-08.SPEC-003-AC-12:** Given FEAT-08.SPEC-004 corrects a recipe's previously incomplete ingredient data, when Sam next opens that recipe, then it no longer shows the incomplete-data reason, and shows only any remaining hard-rule reason, or full eligibility if none remains.

**FEAT-08.SPEC-003-AC-13:** Given a household's dietary rule changes while a disclosed recipe's detail screen is already open, when the change occurs, then the on-screen disclosure text is not live-updated; the next fresh open reflects the current determination.

**FEAT-08.SPEC-003-AC-14:** Given no role in the product has a control to override or suppress this disclosure, when any household member views an ineligible recipe reached by direct search, then the disclosure always appears -- there is no bypass path for any role, including Maya.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
