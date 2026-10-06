---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-02.SPEC-008
spec_name: Safety Badge & Disclaimer Display Rule
spec_slug: safety-badge-and-disclaimer-display-rule
parent_feature: FEAT-02
parent_feature_name: Dietary Rules & Allergy Safety Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 9
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Safety Badge & Disclaimer Display Rule

## Overview

**Name:** Safety Badge & Disclaimer Display Rule
**ID:** FEAT-02.SPEC-008
**Type:** Logic/Rule
**Purpose:** Governs the "checked against allergies" badge, its disclaimer, and the plain ineligibility reason shown wherever a meal or candidate recipe appears.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine
**Governed Entity:** Planned Meal

## Scope and Non-Goals

**In Scope:**
- The exact wording of the "checked against allergies" badge and its "always check labels" disclaimer
- The exact phrasing pattern for the plain ineligibility reason shown on an excluded candidate recipe
- Non-color-only presentation requirements for the badge (ASMP-29)
- Authorization for who sees the badge and reason wherever a meal or candidate appears

**Non-Goals:**
- Determining whether a recipe passes or fails the check -- owned by FEAT-02.SPEC-002 (Candidate Safety Check Execution), which this spec's badge and reason display
- The visual styling of the badge (color, icon shape, placement pixel values) -- design-agnostic per pipeline-rules.md; this spec defines wording and non-color-only requirement, not visual treatment
- Per-screen layout of where the badge sits -- owned by each consuming screen spec (FEAT-03, FEAT-04, FEAT-08, FEAT-17, FEAT-19, FEAT-23); this spec supplies the shared wording those screens reuse

## Governed Entity

**Entity:** Planned Meal
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| safety_badge | text | "checked against allergies" plus the "always check labels" disclaimer |
| recipe | reference | The chosen Recipe -- its dietary_badges field is the ineligibility-reason source when a candidate is excluded |
| status | enum | Proposed/Picked, Confirmed, Swapped, Removed (safety), Cooked |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-02.SPEC-002 | Candidate Safety Check Execution | Attaches the badge (on pass) or the ineligibility reason (on exclusion) as its final processing step |
| FEAT-03 | AI Weekly Dinner Plan Generation (plan display) | Renders the badge on every shown Planned Meal |
| FEAT-23 | Manual Weekly Planning (pick list, plan display) | Renders the badge on every placed pick and the ineligibility reason on every excluded search result |
| FEAT-04 | One-Tap Meal Swap (alternatives list) | Renders the badge on every offered alternative |
| FEAT-08 / FEAT-10 | Recipe Library / Recipe Import (browsing, detail view) | Renders the badge or ineligibility reason wherever household-specific dietary_badges are shown |
| FEAT-17 | Older-Kid Dinner Voting (Later, voting options) | Renders the badge on every voting option |
| FEAT-19 | Weekly Plan History (re-used week display) | Renders the badge on re-checked meals from a re-used week |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| safety_badge | Must display the exact wording "Checked against allergies" as the badge label, with the disclaimer "Always check labels" shown alongside or on the same element | Whenever a Planned Meal has passed the safety check and is shown | On every render of a Planned Meal to any entitled role | N/A -- this is a display rule, not a form validation; there is no error state, only a display-completeness requirement | Yes (display is mandatory wherever a passed meal is shown) |
| ineligibility reason | Must follow the exact phrasing pattern: "Contains {allergen} -- not safe for {member}." for an allergy or religious-rule failure; "This recipe's ingredient list is incomplete, so it can't be checked for safety yet." for incomplete data (per FEAT-02.SPEC-007) | Whenever a candidate recipe is excluded and a specific search or browse context calls for showing why | On every render of an excluded candidate in a searchable or browsable context (FEAT-08, FEAT-23) | The phrasing itself is the message -- there is no separate error state | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Badge and reason are mutually exclusive per Planned Meal or candidate | safety_badge, recipe (exclusion state) | A given Planned Meal or candidate shows either the passed badge or the excluded reason, never both -- these reflect the two outcomes of FEAT-02.SPEC-002's determination | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the safety badge on a shown Planned Meal | Maya, Sam, Jordan (older kid, limited login -- Later) | Always, wherever the consuming screen already shows them the meal (per that screen's own Access and Visibility) | -- |
| View the plain ineligibility reason on an excluded candidate | Maya, Sam | Always, wherever the consuming screen already shows them search or browse results (FEAT-08, FEAT-23) | -- |
| View the plain ineligibility reason on an excluded candidate | Jordan (older kid, limited login -- Later) | Never -- older-kid access is limited to Dinner Voting (Own-only) and Grocery List (Full); voting options are pre-filtered to safe choices only, so this role never encounters an excluded candidate to see a reason for | The voting screen (FEAT-17) never presents an ineligible option; there is nothing to deny since the scenario cannot arise |
| Change the badge or disclaimer wording | No role -- it is a fixed, product-wide display rule | Never | No screen exposes a control to edit the badge or disclaimer text |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| safety_badge | Set to "Checked against allergies" plus "Always check labels" whenever FEAT-02.SPEC-002 determines a pass | On every candidate check that results in a pass, before the recipe is shown or placed | No |
| ineligibility reason text | Derived from the specific failure per FEAT-02.SPEC-002's outcome: allergen/member name for a hard-rule failure, or the fixed incomplete-data message for FEAT-02.SPEC-007 | On every candidate check that results in an exclusion | No |

## Business Rules

- XBR-01: Every shown meal carries the "checked against allergies" badge and "always check labels" disclaimer -- there is no path onto the plan that bypasses this display requirement.
- ASMP-29: The badge's presentation is never color-only -- it always carries the text label "Checked against allergies" so the safety signal does not depend on color perception.
- The plain ineligibility reason uses one consistent phrasing pattern everywhere a candidate recipe is excluded -- Manual Weekly Planning search results, Recipe Library browsing, and swap alternative lists all reuse this spec's wording rather than each screen inventing its own, per the Brief's Shared UI Patterns.
- The badge and disclaimer are informational, not a warranty -- the disclaimer exists precisely because the engine checks against stated household rules, not against every possible real-world risk (e.g., cross-contamination in a physical kitchen), and this spec never claims otherwise.

## Edge Cases

- **A Planned Meal transitions to Removed (safety) after having shown the badge** -- The badge is no longer relevant once the slot is empty; the removed meal's history entry (swap_history) retains no ongoing badge display, since a removed meal is not "shown" in the sense this rule governs.
- **The same recipe is safe for one household and excluded for another** -- The badge or reason is computed per household (dietary_badges is described as "computed per household by this feature when viewed"), so the same Recipe can carry the badge for one household and the ineligibility reason for another simultaneously, with no shared cached state between households.
- **A household member views a recipe detail page with no plan context (pure library browsing)** -- The badge or reason still displays, computed against that household's current rules, even though no specific Planned Meal exists yet -- the display rule applies to any place dietary_badges are shown, not only to placed meals.
- **An older-kid limited-login voting round somehow includes options that were not pre-filtered (a defect scenario)** -- Per FEAT-02.SPEC-002, every voting option must already have passed the check before the round opens; this spec's badge (not a reason) is the only display this role would ever see, since an unsafe option should never reach voting to begin with.

## Acceptance Criteria

**FEAT-02.SPEC-008-AC-01:** Given a candidate recipe passes the safety check for Maya's household, when the recipe is placed on the plan, then it displays the badge text "Checked against allergies" with the disclaimer "Always check labels."

**FEAT-02.SPEC-008-AC-02:** Given a candidate recipe fails the check because it contains an allergen a household member cannot have, when Sam searches for it in Manual Weekly Planning, then it shows the reason "Contains {allergen} -- not safe for {member}."

**FEAT-02.SPEC-008-AC-03:** Given a candidate recipe is excluded for incomplete ingredient data, when it appears in a search context, then it shows "This recipe's ingredient list is incomplete, so it can't be checked for safety yet."

**FEAT-02.SPEC-008-AC-04:** Given the same recipe passes for Maya's household but fails for a different household with a conflicting allergy, when each household views it, then Maya's household sees the badge and the other household sees the ineligibility reason, computed independently.

**FEAT-02.SPEC-008-AC-05:** Given Jordan is signed in through the Later-phase older-kid limited login viewing a dinner-voting round, when the round's options are displayed, then each shows the badge, and none shows an ineligibility reason, since every option already passed the check.

**FEAT-02.SPEC-008-AC-06:** Given a household member is on a screen that cannot render color (or has color perception differences), when they view the safety badge, then the text label "Checked against allergies" is still fully legible, since the badge is never color-only.

**FEAT-02.SPEC-008-AC-07:** Given Sam browses the Recipe Library outside any specific plan context, when he views a recipe his household cannot safely eat, then the same ineligibility reason phrasing appears as it would in Manual Weekly Planning search results.

**FEAT-02.SPEC-008-AC-08:** Given no household member has a control to edit the badge or disclaimer wording, when any adult looks for a customization option, then none is available anywhere in the product.

**FEAT-02.SPEC-008-AC-09:** Given a Planned Meal is removed for a safety concern, when the plan is viewed afterward, then no badge displays for the emptied slot, since the meal is no longer shown.

**FEAT-02.SPEC-008-AC-10:** Given a candidate recipe appears in swap alternatives (FEAT-04), when Maya views the alternatives list, then every alternative shows the badge, consistent with FEAT-04's guarantee that only checked recipes are offered.

**FEAT-02.SPEC-008-AC-11:** Given Riley (Operator) views a household's plan through the read-only Support View (FEAT-22), when a meal is displayed, then the same badge wording appears as it would for Maya or Sam, since the badge itself carries no sensitive data beyond the pass/fail determination.

**FEAT-02.SPEC-008-AC-12:** Given the plain ineligibility reason pattern is used across Manual Weekly Planning, Recipe Library, and swap alternatives, when the same excluded recipe is viewed from each of these three contexts, then the wording is identical in all three.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |
