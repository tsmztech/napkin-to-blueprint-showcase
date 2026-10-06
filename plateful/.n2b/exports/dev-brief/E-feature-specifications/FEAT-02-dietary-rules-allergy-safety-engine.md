# FEAT-02 — Dietary Rules & Allergy Safety Engine

This chapter covers FEAT-02, Dietary Rules & Allergy Safety Engine, a Core-tier feature. It contains 14 specifications carrying 158 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-02.SPEC-001 | Report a Safety Concern | screen | 12 |
| FEAT-02.SPEC-002 | Candidate Safety Check Execution | automation | 13 |
| FEAT-02.SPEC-003 | Mid-Week Rule Change Re-Check | automation | 11 |
| FEAT-02.SPEC-004 | Safety Concern Intake & Removal | automation | 12 |
| FEAT-02.SPEC-005 | Safety Concern Resolution Outcome | automation | 10 |
| FEAT-02.SPEC-006 | Rule Strength & Blocking Policy | logic-rule | 13 |
| FEAT-02.SPEC-007 | Ingredient Data Completeness & Fail-Closed Policy | logic-rule | 11 |
| FEAT-02.SPEC-008 | Safety Badge & Disclaimer Display Rule | logic-rule | 12 |
| FEAT-02.SPEC-009 | Safety Concern Eligibility & Re-offer Policy | logic-rule | 11 |
| FEAT-02.SPEC-010 | Transactional Email Delivery (Safety Reports) | integration | 12 |
| FEAT-02.SPEC-011 | Safety Concern Reporter Acknowledgement | notification | 9 |
| FEAT-02.SPEC-012 | Safety Concern Organiser Alert | notification | 11 |
| FEAT-02.SPEC-013 | Safety Concern Operator Alert | notification | 10 |
| FEAT-02.SPEC-014 | Safety Concern Resolution Notice | notification | 11 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Dietary Rules & Allergy Safety Engine

## Summary

**Feature:** Dietary Rules & Allergy Safety Engine
**ID:** FEAT-02
**Description:** Before any suggested meal reaches a household member, the app itself checks it against every household member's allergies and religious rules. This check runs independently of the AI — the AI proposes, the app verifies — so a single AI mistake can never reach the table.
**Priority:** Core
**Phase:** MVP
**Type:** Platform
**Rationale:** The brief states this as a hard rule, not a preference: "the app itself checks every suggestion against the household's allergy list... the AI is never the last line of defence" (BRIEF.md, Vision, Constraints: Safety). This is the single most trust-critical feature in the product and must exist from day one. No profiled competitor describes an app-enforced allergy check distinct from AI personalization or a manual exclusion filter, and a publicized supermarket AI meal-planner produced dangerous recipes — the failure mode this engine exists to prevent.

**Key Capabilities:**
- Verify every suggestion — every recipe considered for a household's plan is checked against that household's allergies and religious rules before it can appear, including recipes an adult picks by hand (Manual Weekly Planning, FEAT-23) and past weeks re-used from history (Weekly Plan History, FEAT-19)
- Show a safety badge — every meal in the plan carries a "checked against allergies" badge with a standard "always check labels" disclaimer
- Block unsafe suggestions — a recipe that fails the check for any household member is removed from consideration for that household entirely, not just flagged
- Distinguish rule strength — allergies and religious rules are hard filters; vegetarian settings apply per person with a shared-meal vegetarian option; dislikes are soft and never block a suggestion
- Report a safety concern — any adult member can flag a meal they believe is unsafe; it is removed from the household's plan at once, safe alternatives are offered, and the report reaches the operator for review

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-02.SPEC-001 | Report a Safety Concern | Screen | Maya, Sam | An adult flags a meal they believe is unsafe, with an optional short note, from wherever the meal is shown |
| FEAT-02.SPEC-002 | Candidate Safety Check Execution | Automation | All | Checks a candidate recipe's ingredients against every household member's hard rules before it can ever be shown, on every path onto the plan |
| FEAT-02.SPEC-003 | Mid-Week Rule Change Re-Check | Automation | All | Re-verifies every remaining dinner in the current week the moment a hard rule is added or tightened |
| FEAT-02.SPEC-004 | Safety Concern Intake & Removal | Automation | Maya, Sam, Riley | Removes a reported meal from the plan immediately, excludes the recipe for the household while the report is open, and opens the review record |
| FEAT-02.SPEC-005 | Safety Concern Resolution Outcome | Automation | Maya, Sam, Riley | Applies the operator's review outcome to the excluded recipe's eligibility and triggers the household's outcome notice |
| FEAT-02.SPEC-006 | Rule Strength & Blocking Policy | Logic/Rule | All | Defines which rule kinds are hard filters (allergy, religious rule, per-person/shared vegetarian) versus soft, non-blocking preferences (dislikes) |
| FEAT-02.SPEC-007 | Ingredient Data Completeness & Fail-Closed Policy | Logic/Rule | All | Requires complete ingredient data with no partial matching shortcuts; a recipe with incomplete data is excluded rather than assumed safe |
| FEAT-02.SPEC-008 | Safety Badge & Disclaimer Display Rule | Logic/Rule | Maya, Sam, Jordan (older kid, Later) | Governs the "checked against allergies" badge, its disclaimer, and the plain ineligibility reason shown wherever a meal or candidate recipe appears |
| FEAT-02.SPEC-009 | Safety Concern Eligibility & Re-offer Policy | Logic/Rule | All | Keeps a recipe under an open safety report out of the household's candidate pool until the report resolves |
| FEAT-02.SPEC-010 | Transactional Email Delivery (Safety Reports) | Integration | Maya, Sam, Riley | Delivers the operator-facing safety-concern alert and the household's resolution notice through the product's transactional email capability |
| FEAT-02.SPEC-011 | Safety Concern Reporter Acknowledgement | Notification | Maya, Sam | Confirms to the reporting adult that their safety concern was received and the meal has been removed |
| FEAT-02.SPEC-012 | Safety Concern Organiser Alert | Notification | Maya | Tells the organiser a meal was removed from the plan because of a safety concern or a rule change, even when she wasn't the reporter |
| FEAT-02.SPEC-013 | Safety Concern Operator Alert | Notification | Riley | Sends the safety-concern report to the operator by transactional email for review |
| FEAT-02.SPEC-014 | Safety Concern Resolution Notice | Notification | Maya, Sam | Tells the household the outcome once the operator has resolved the report |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Verify every suggestion | FEAT-02.SPEC-002 | The Candidate Safety Check Execution automation runs on every candidate recipe from every path onto the plan (AI generation, manual pick, swap, vote, history re-use) before it is ever shown | Phase 2 (Explicit) |
| Show a safety badge | FEAT-02.SPEC-008 | The display rule attaches the "checked against allergies" badge and disclaimer to every shown meal and the plain reason to every ineligible one | Phase 2 (Explicit) |
| Block unsafe suggestions | FEAT-02.SPEC-002, FEAT-02.SPEC-007 | The check excludes a failing recipe from the household's candidate pool entirely; incomplete ingredient data fails closed under the same exclusion | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |
| Distinguish rule strength | FEAT-02.SPEC-006 | The policy classifies allergy and religious rules as hard, vegetarian as per-person with a shared-meal option, and dislikes as soft and non-blocking | Phase 2 (Explicit) |
| Report a safety concern | FEAT-02.SPEC-001, FEAT-02.SPEC-004 | The screen captures the report; the intake automation removes the meal at once, excludes the recipe, and opens the review record | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — specs not directly tied to a Key Capability, surfaced by Phases 3–6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-02.SPEC-003 | Mid-Week Rule Change Re-Check | Phase 4 (Trigger-Response) | XBR-02 requires that a new or tightened hard rule take effect on the current week immediately; the feature description names the check only at suggestion time, not at rule-change time |
| FEAT-02.SPEC-005 | Safety Concern Resolution Outcome | Phase 4 (Trigger-Response) / Phase 6 (Failure Analysis) | The "report a safety concern" capability describes the report reaching the operator, but not what happens to the excluded recipe or the household once the operator resolves it — a counterpart step the journey walk (Allergy-Safe Swap Recovery) requires |
| FEAT-02.SPEC-009 | Safety Concern Eligibility & Re-offer Policy | Phase 5 (Rule Discovery) | The Validation & Limits field states a recipe under an open report is never re-offered until resolved — a conditional rule shared by both the Candidate Safety Check Execution and Safety Concern Intake automations |
| FEAT-02.SPEC-010 | Transactional Email Delivery (Safety Reports) | Phase 4 (External Dependencies lens) | assumptions-constraints.md names transactional email as a category-level dependency for safety-concern reports reaching the operator; the external contract needed its own Integration spec |
| FEAT-02.SPEC-011 | Safety Concern Reporter Acknowledgement | Phase 4 (Notification surfacing) | The Communications field names a reporter acknowledgement with defined content and audience — beyond a same-screen toast |
| FEAT-02.SPEC-012 | Safety Concern Organiser Alert | Phase 4 (Notification surfacing) | The Communications field names an organiser notice distinct from the reporter's own acknowledgement |
| FEAT-02.SPEC-013 | Safety Concern Operator Alert | Phase 4 (Notification surfacing) | The Communications field names the operator-bound report as a transactional email with its own delivery rules |
| FEAT-02.SPEC-014 | Safety Concern Resolution Notice | Phase 4 (Notification surfacing) | The Communications field states the household is told when the review is resolved — a distinct, later message from the initial acknowledgement |

## Entity-Lifecycle Coverage Matrix

**Entity: Support Request**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-02.SPEC-004 | Safety Concern Intake & Removal creates the Support Request (kind = safety concern; raised_by; planned_meal/recipe; optional note up to 500 characters) the moment SPEC-001 is submitted | -- |
| Read (single) | N/A for FEAT-02 | Owned by FEAT-01 (organiser's open-request view) and FEAT-22 (operator review); FEAT-02 surfaces no standalone read screen of its own | Cross-feature: FEAT-01, FEAT-22 |
| Read (list) | N/A for FEAT-02 | Same as above -- listing open requests is FEAT-01's and FEAT-22's responsibility | Cross-feature: FEAT-01, FEAT-22 |
| Update | N/A for FEAT-02 | Status (Raised -> Under review -> Resolved) and the access_record are written only by FEAT-22 | FEAT-02.SPEC-005 consumes the terminal Resolved transition as an inbound trigger |
| Delete/Archive | N/A -- explicit non-goal | Safety-concern records are never deleted; they are retained for the life of the household account as trust and audit history (SC-18 history depth; ASMP-26/27 trust rationale), matching the change-history retention already established for Dietary Rule | No restore path needed, since no delete occurs |
| State Transition | N/A for FEAT-02 | Transition mechanics (Raised -> Under review -> Resolved) belong to FEAT-22 | FEAT-02.SPEC-005 is the terminal-transition consumer, not the owner |

**Entity: Planned Meal**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A for FEAT-02 | Created by FEAT-03 (AI proposal), FEAT-23 (manual pick), and FEAT-11 (leftover lunch suggestion); FEAT-02 never originates a Planned Meal | Cross-feature: FEAT-03, FEAT-23, FEAT-11 |
| Read (single) | FEAT-02.SPEC-004, FEAT-02.SPEC-005 | Both read the specific Planned Meal named in a safety-concern report to act on it | -- |
| Read (list) | FEAT-02.SPEC-003 | Mid-Week Rule Change Re-Check reads every remaining dinner in the current week's plan to re-verify each one | -- |
| Update | FEAT-02.SPEC-002, FEAT-02.SPEC-003, FEAT-02.SPEC-004 | SPEC-002 writes the safety_badge field when a candidate passes; SPEC-003 and SPEC-004 write status = Removed (safety) and append to swap_history | -- |
| Delete/Archive | FEAT-02.SPEC-004 | Soft removal: status is set to Removed (safety), not hard-deleted -- the record stays in the plan's history rather than being purged; no restore path (a fresh safe pick fills the slot through FEAT-04, it does not undo the removal); cascades to the Grocery List, which drops the removed meal's ingredients (FEAT-06); retained for the life of the account per SC-18 | Mid-week rule changes (SPEC-003) use the same removal mechanic |
| State Transition | FEAT-02.SPEC-002 (precondition: badge attach on candidate pass, before the slot is filled), FEAT-02.SPEC-003 / FEAT-02.SPEC-004 (Proposed/Picked/Confirmed -> Removed (safety)) | -- | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Dietary Rule | FEAT-02.SPEC-002, FEAT-02.SPEC-003, FEAT-02.SPEC-006, FEAT-02.SPEC-009 | Every household member's hard and soft rules are read to check candidate recipes, re-check the current week, classify rule strength, and determine re-offer eligibility |
| Recipe | FEAT-02.SPEC-002, FEAT-02.SPEC-007, FEAT-02.SPEC-009 | Every candidate recipe's ingredient list is read to run the check, verify data completeness, and determine whether an excluded recipe may re-enter the pool |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| A candidate recipe is proposed onto any household's plan (AI generation, manual pick, swap, vote, history re-use) | Check the recipe's ingredients against every household member's hard rules; exclude it from the candidate pool if any hard rule fails or ingredient data is incomplete | Standalone Automation | FEAT-02.SPEC-002 |
| A candidate recipe passes the check | Attach the "checked against allergies" badge and disclaimer before the recipe can be shown or placed | Inline in FEAT-02.SPEC-002, governed by FEAT-02.SPEC-008 | FEAT-02.SPEC-002 / FEAT-02.SPEC-008 |
| A candidate recipe has incomplete ingredient data | Exclude the recipe by default (fail closed) rather than showing it unchecked; emit safety_check_data_incomplete | Standalone Logic/Rule | FEAT-02.SPEC-007 |
| Maya adds or tightens a hard rule (allergy or religious rule) for any member | Re-check every remaining dinner in the current week; remove and flag any that now fail; offer safe alternatives through swap; recalculate the grocery list; tell the organiser | Standalone Automation (with cross-feature effects on Grocery List and Meal Swap) | FEAT-02.SPEC-003 |
| An adult taps "report a safety concern" on a meal | Remove the meal from the household's plan at once; exclude the recipe for the household while the report is open; drop its ingredients from the grocery list; create the Support Request; offer safe alternatives through swap | Standalone Automation | FEAT-02.SPEC-004 |
| A safety-concern report is submitted | Confirm receipt and removal to the reporter | Standalone Notification | FEAT-02.SPEC-011 |
| A meal is removed from the plan (safety concern or mid-week rule change) and the organiser was not the actor | Tell the organiser the meal was removed and why | Standalone Notification | FEAT-02.SPEC-012 |
| A Support Request (safety concern) is created | Send the report to the operator by transactional email for review | Standalone Notification, delivered through Standalone Integration | FEAT-02.SPEC-013 / FEAT-02.SPEC-010 |
| The operator resolves a safety-concern Support Request (FEAT-22) | Apply the outcome to the excluded recipe's eligibility (release it or keep it excluded) and tell the household the outcome | Standalone Automation, followed by Standalone Notification delivered through the Integration spec | FEAT-02.SPEC-005 / FEAT-02.SPEC-014 / FEAT-02.SPEC-010 |
| A recipe is placed under an open safety report | Keep the recipe out of that household's candidate pool for every future check until the report resolves | Standalone Logic/Rule, read by FEAT-02.SPEC-002 and FEAT-02.SPEC-004 | FEAT-02.SPEC-009 |
| A recipe (starter or household-imported) is edited or re-imported | The edited recipe must pass the safety check again before it can appear in any plan (XBR-19) | Cross-feature -- logged in touchpoints; re-check itself runs through FEAT-02.SPEC-002 | FEAT-10 responsibility for the edit; FEAT-02.SPEC-002 for the re-check |

## Shared Context

**Shared Entities:**
- Dietary Rule -- read by SPEC-002, SPEC-003, SPEC-006, and SPEC-009. Fields: member, rule_kind (allergy, religious rule, per-person vegetarian, dislike), strength (hard/soft), allergen (from the standard allergen list, optionally a named extra ingredient), origin, change_history. Owned and written by FEAT-01; FEAT-02 never writes to it.
- Recipe -- read by SPEC-002, SPEC-007, and SPEC-009. Fields relevant here: ingredients (quantity + unit, required complete for the check to pass), dietary_badges (computed per household by this feature when viewed). Owned by FEAT-08/FEAT-10; FEAT-02 never writes recipe content.
- Planned Meal -- read and updated by SPEC-002, SPEC-003, SPEC-004, and SPEC-005 as described in the Entity-Lifecycle Coverage Matrix. Fields this feature writes: safety_badge, status (Removed (safety)), swap_history.
- Support Request -- created by SPEC-004, consumed at resolution by SPEC-005. Fields: kind (safety concern), raised_by, planned_meal/recipe, note (optional, up to 500 characters), status.

**Shared UI Patterns:**
- Plain ineligibility reason -- SPEC-008 defines a single, consistent phrasing pattern ("contains {allergen} -- not safe for {member}") used everywhere a candidate recipe is excluded: Manual Weekly Planning search results, Recipe Library browsing, and swap alternative lists. Spec Writers for those screens describe the reason using this pattern rather than inventing their own wording.
- Safety badge treatment -- SPEC-008 defines the badge's wording, disclaimer, and non-color-only presentation (ASMP-29); every screen that shows a Planned Meal (FEAT-03, FEAT-04, FEAT-17, FEAT-19, FEAT-23) reuses this treatment rather than restyling it.

**Shared Validation:**
- SPEC-006 (Rule Strength & Blocking Policy) and SPEC-007 (Ingredient Data Completeness & Fail-Closed Policy) are the two rules every other spec in this feature -- and every consuming feature's screens -- must reference rather than re-deriving which rules block and when a recipe fails closed.
- SPEC-009 (Safety Concern Eligibility & Re-offer Policy) is the single source of truth for whether a recipe under report may be offered; SPEC-002 and SPEC-004 both defer to it rather than each keeping their own exclusion state.

## Internal Dependency Map

```
SPEC-001 (Report a Safety Concern) -> [adult submits report] -> SPEC-004 (Safety Concern Intake & Removal)
SPEC-004 (Safety Concern Intake & Removal) -> [meal removed] -> SPEC-011 (Reporter Acknowledgement)
SPEC-004 (Safety Concern Intake & Removal) -> [meal removed, organiser not the actor] -> SPEC-012 (Organiser Alert)
SPEC-004 (Safety Concern Intake & Removal) -> [report created] -> SPEC-013 (Operator Alert) -> [delivered via] -> SPEC-010 (Transactional Email Delivery)
SPEC-004 (Safety Concern Intake & Removal) -> [excludes recipe] -> SPEC-009 (Safety Concern Eligibility & Re-offer Policy)
SPEC-009 (Safety Concern Eligibility & Re-offer Policy) -> [governs future candidates] -> SPEC-002 (Candidate Safety Check Execution)
[FEAT-22 resolves the Support Request] -> [inbound trigger] -> SPEC-005 (Safety Concern Resolution Outcome)
SPEC-005 (Safety Concern Resolution Outcome) -> [releases or keeps excluded] -> SPEC-009 (Safety Concern Eligibility & Re-offer Policy)
SPEC-005 (Safety Concern Resolution Outcome) -> [tells household] -> SPEC-014 (Resolution Notice) -> [delivered via] -> SPEC-010 (Transactional Email Delivery)
SPEC-002 (Candidate Safety Check Execution) -> [classifies rules using] -> SPEC-006 (Rule Strength & Blocking Policy)
SPEC-002 (Candidate Safety Check Execution) -> [requires complete data per] -> SPEC-007 (Ingredient Data Completeness & Fail-Closed Policy)
SPEC-002 (Candidate Safety Check Execution) -> [attaches badge/reason per] -> SPEC-008 (Safety Badge & Disclaimer Display Rule)
[Maya adds/tightens a hard rule, FEAT-01] -> [inbound trigger] -> SPEC-003 (Mid-Week Rule Change Re-Check)
SPEC-003 (Mid-Week Rule Change Re-Check) -> [re-runs check on remaining dinners using] -> SPEC-002 (Candidate Safety Check Execution)
SPEC-003 (Mid-Week Rule Change Re-Check) -> [removes failing meals, tells organiser] -> SPEC-012 (Organiser Alert)
SPEC-003 (Mid-Week Rule Change Re-Check) -> [flags removal per] -> SPEC-008 (Safety Badge & Disclaimer Display Rule)
```

**Default Entry:** SPEC-002 (Candidate Safety Check Execution) -- this feature has no user-facing entry screen of its own; the automation is the feature's functional entry point, running invisibly inside every other feature that offers a candidate recipe. The only direct user-facing entry point into this feature is SPEC-001 (Report a Safety Concern), reached from a meal already shown on the plan by another feature.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-02.SPEC-002 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Every AI-proposed recipe is checked before it can appear in a generated plan | AI generates a candidate week |
| FEAT-02.SPEC-002 | Inbound | FEAT-23 (Manual Weekly Planning) | Every manually searched or picked recipe is checked before it can be placed | Adult searches or picks a recipe for a night |
| FEAT-02.SPEC-002 | Inbound | FEAT-04 (One-Tap Meal Swap) | Every swap alternative is checked before it is offered | Adult requests alternatives for a night |
| FEAT-02.SPEC-002 | Inbound | FEAT-17 (Older-Kid Dinner Voting, Later) | Every voting option is checked before it is offered to the older kid | A vote is opened for a night |
| FEAT-02.SPEC-002 | Inbound | FEAT-19 (Weekly Plan History) | A past week re-used as a starting point is re-checked before its meals return to the plan | Household re-uses a past week |
| FEAT-02.SPEC-002 | Inbound | FEAT-01 (Household Setup & Member Profiles) | Every household member's allergy and religious-rule data is read from Household Setup | Any check runs |
| FEAT-02.SPEC-008 | Outbound | FEAT-08 (Recipe Library), FEAT-10 (Recipe Import) | The safety badge and ineligibility reason are shown wherever this content is displayed | Library or import content is shown to a household |
| FEAT-02.SPEC-004 | Outbound | FEAT-06 (Shared Grocery List) | Removing a meal drops its ingredients from the list | A safety concern is confirmed or a rule-change removal occurs |
| FEAT-02.SPEC-004 | Outbound | FEAT-04 (One-Tap Meal Swap) | Safe alternatives are offered for the newly empty slot | A meal is removed for safety |
| FEAT-02.SPEC-004 / FEAT-02.SPEC-005 | Outbound | FEAT-22 (Operator Read-Only Support Access) | The Support Request opened by a safety concern is the precondition for the operator's read-only support view | A safety concern is reported; the operator opens the household's record to review it |
| FEAT-02.SPEC-010 / FEAT-02.SPEC-013 / FEAT-02.SPEC-014 | Outbound | FEAT-07 (Notification Preferences, if applicable), FEAT-18 (Account & Data Management) | Shares the transactional email capability that account, billing, and export notices also rely on | Any safety-concern email is sent |
| FEAT-02.SPEC-002 / FEAT-02.SPEC-007 | Inbound | FEAT-08 (Recipe Library, starter content) | Depends on complete ingredient data seeded at launch (Recipe/food-content data capability, ASMP-34); FEAT-08 owns the ingestion, FEAT-02 only consumes and validates completeness | Starter library is seeded or a recipe is imported |

## Non-Functional Notes

**Data volumes / growth:** The check runs once per candidate recipe per household on every plan generation, swap, manual pick, vote, and history re-use, across several thousand households of 2–6 members each in the first year (SC-15); it must stay equally fast as the household base and each household's recipe pool (starter plus imports) grow, with no full re-scan shortcuts that would trade completeness for speed.

**Responsiveness:** The check completes before a suggestion is ever shown, so it carries no separate loading state and must fit inside the plan-generation and swap responsiveness budgets already set for those features -- well under a minute for a full week, seconds for a swap (ASMP-22, ASMP-23). A safety-concern removal and its grocery-list update must feel as close to instant as any other list change (ASMP-22).

**Data sensitivity / privacy:** Dietary Rule data is health-adjacent personal data, including children's allergy information -- the most sensitive data in the product -- and carries children's-privacy-class protection: minimal, parent-controlled, used only for the household's own plan, never sold or used for advertising (ASMP-26, ASMP-27). Support Requests raised as safety concerns may contain the same class of data and are handled under the same protection; the operator (Riley) sees a kid's allergy detail only inside a specific safety report, never Kid Profile Data generally (Access Matrix notes).

**Compliance flags:** Children's-privacy-class handling applies wherever this feature touches a kid profile's allergy data, including within a safety report (ASMP-27). No medical-data regime applies, and this feature is never a source of medical or diet advice -- it renders a pass/fail safety determination against stated household rules, nothing more (SC-06).

## Non-Goals

- **Medical or diet advice** -- Excluded per scope-boundaries.md (SC-06): the engine renders a pass/fail safety determination against stated allergy and religious rules; it never scores, ranks, or advises on nutrition, calories, or diet quality, matching the brief's explicit "no medical or diet advice" position.
- **Automatic purge or deletion of safety-concern records** -- Intentional lifecycle decision surfaced by the CRUD matrix: Support Requests and Dietary Rule change history are retained for the life of the household account as trust and audit history, matching the account-wide history-depth expectation (scope-boundaries.md, SC-18) and the change-history retention already established for Dietary Rule (product-features.md, FEAT-01 Data Notes). No retention window or purge job applies.
- **Fuzzy or partial ingredient matching** -- Excluded per the feature's own Validation & Limits ("no partial matching shortcuts"): a convenience-oriented fuzzy match was considered during adjacency analysis and rejected because any tolerance for a near-miss match directly contradicts the brief's zero-incident success criterion ("it has never once suggested a meal that breaks a family member's allergy," BRIEF.md, Success Criteria).
- **Offline generation of new suggestions** -- N/A per the feature's own States field: generating a new candidate to check requires connectivity in the first place (plan generation, swap, and manual search all depend on it), so there is no offline mode for this engine to degrade into; offline behavior for already-generated plans and the grocery list belongs to FEAT-03 and FEAT-06 (ASMP-25).
- **Kid-initiated safety reports** -- Excluded per scope-boundaries.md (SC-02) and the Access Matrix: young kid profiles have no login in v1, so they cannot raise or view a safety concern; only Maya (Full) and Sam (Own-only) can report, consistent with the brief's parent-managed-profile default.



# Screen Spec: Report a Safety Concern

## Overview

**Name:** Report a Safety Concern
**ID:** FEAT-02.SPEC-001
**Type:** Screen
**Purpose:** An adult flags a meal they believe is unsafe, with an optional short note, from wherever the meal is currently shown.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- The confirmation dialog an adult reaches by tapping "report a safety concern" on a shown Planned Meal
- Capturing the optional short note (up to 500 characters)
- Submitting the report and showing the immediate in-screen result of that submission

**Non-Goals:**
- Removing the meal from the plan, excluding the recipe, and creating the Support Request -- performed by FEAT-02.SPEC-004 (Safety Concern Intake & Removal), which this screen's submission triggers
- Reviewing or resolving the report -- owned by the operator's Support View (FEAT-22), outside this feature
- Reporting anything other than a currently shown Planned Meal -- excluded per the feature's own Access field (only adults reporting on a meal already displayed by another feature can reach this dialog); there is no standalone "browse past concerns" screen in this feature
- Kid-initiated reports -- excluded per scope-boundaries.md (SC-02): young kid profiles have no login in v1, so only Maya and Sam can reach this screen

## Entry Points

{This screen has no feature of its own that opens it directly -- it is reached only from a "report a safety concern" control that other features' meal-display specs place next to a shown Planned Meal.}

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-03 (AI Weekly Dinner Plan Generation), planned meal card | Adult taps "report a safety concern" on a dinner | The Planned Meal reference and its Recipe |
| FEAT-23 (Manual Weekly Planning), planned meal card | Adult taps "report a safety concern" on a picked dinner | The Planned Meal reference and its Recipe |
| FEAT-19 (Weekly Plan History), a re-used past week's meal card | Adult taps "report a safety concern" on a meal shown from history | The Planned Meal reference and its Recipe |
| FEAT-08 / FEAT-10 (Recipe Library / Recipe Import), recipe detail when opened from a plan context | Adult taps "report a safety concern" on the recipe as placed in the current plan | The Planned Meal reference and its Recipe |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full dialog | Submit a report on any household meal | -- |
| Sam (Other Adult Member) | Full dialog | Submit a report on any household meal (Safety Reports: Own-only means Sam acts only through his own submissions, not that he can report only his own meals -- any adult may flag any meal) | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | The "report a safety concern" control is never shown on any screen a young kid profile could reach, since young kid profiles have no login at all |
| Jordan (older kid, limited login -- Later) | No | No | The "report a safety concern" control does not appear on any screen the older-kid login can reach; Dinner Voting and Grocery List are the only surfaces available to this role, and neither shows the control |
| Riley (Operator, support) | No | No | Riley has no access to any household screen outside the read-only Support View (FEAT-22); this dialog never renders for Riley |
| Unauthenticated | No | No | Redirected to the sign-in screen; no in-progress report context survives, since none can exist before sign-in |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any typed note is preserved locally and restored in the dialog after re-authentication succeeds |

## Layout and Content

**Header:** Dialog title "Report a safety concern" with a close control (top-right, dismisses without submitting) and, below the title, the name of the meal being reported (recipe name and the night it is planned for).

**Body:** A short explanatory line: "This meal will be removed from your plan right away, and we'll offer safe alternatives. Your household's operator will review the ingredients." Below it, a single optional multi-line text field labeled "What's wrong? (optional)" with a visible remaining-character count that starts at 500 and counts down as the adult types.

**Footer:** Two actions, right-aligned: "Cancel" (secondary, dismisses without submitting) and "Report and remove" (primary).

### Responsive Behavior

- **Compact breakpoint:** The dialog occupies the full screen width with the meal name and note field stacked vertically; footer actions stack full-width, "Report and remove" above "Cancel".
- **Medium size class and above:** The dialog renders as a centered modal capped at a consistent platform-wide dialog width (exact value is the design layer's decision); footer actions remain side by side, right-aligned.
- **Note field:** Grows from 3 visible lines (compact) to 4 visible lines (medium and above); no structural change beyond line count.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Close control | Tap | Dismiss the dialog without submitting | Dialog closes, returns to the calling screen | No confirmation needed -- nothing has been submitted |
| Note field | Type | Captures free text up to 500 characters | Remaining-character count updates | Count updates live; typing beyond 500 characters is blocked at the field |
| Cancel button | Tap | Dismiss the dialog without submitting | Dialog closes, returns to the calling screen | No confirmation needed -- nothing has been submitted |
| Report and remove button | Tap | 1. Submit the report (meal reference, reporting member, optional note) to FEAT-02.SPEC-004 (Safety Concern Intake & Removal). 2. Wait for confirmation that the meal was removed. | Button shows a brief loading state during submission | Success: dialog replaces its content with a confirmation message and a "Show me alternatives" action; the calling screen updates to show the meal as removed. Failure: inline error banner in the dialog with a Retry option; the meal and note are unchanged. |
| Report and remove button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "Show me alternatives" (post-submission) | Tap | Navigate to FEAT-04.SPEC-001 (Meal Swap, safe alternatives list) for the now-empty slot | Dialog closes | Alternatives list opens for the affected night |

### Accessibility Notes

- **Focus order:** Close control -> meal name (read-only, announced but not focusable) -> note field -> Cancel -> Report and remove.
- **Submission announcements:** On successful submission, the confirmation message and "Show me alternatives" action are announced to assistive technology as the dialog's content changes. On failure, the error banner is announced and focus moves to it.
- **Character count:** The remaining-character count is associated with the note field so assistive technology announces it as the field's description, not as a separate unlabelled element.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Ready (default) | Meal name shown, note field empty, both actions enabled | Dialog opens | Adult types in the note field or taps an action |
| Filling | Note field contains text, remaining-character count updated | Adult types in the note field | Adult taps Cancel, Close, or Report and remove |
| Submitting | "Report and remove" shows a loading state, both actions disabled | Adult taps "Report and remove" | Submission completes or fails |
| Submitted | Confirmation message and "Show me alternatives" action replace the form | Submission completes successfully | Adult taps "Show me alternatives" or Close |
| Error | Error banner "Couldn't submit your report. Check your connection and try again." with Retry; note text preserved | Submission fails | Adult taps Retry (returns to Submitting) or Close |
| Offline/Degraded | Banner "You're offline -- reporting a safety concern needs a connection, since the meal must be removed right away." "Report and remove" is disabled while offline | Connectivity lost while the dialog is open | Connectivity restored -- banner clears and "Report and remove" re-enables; the adult resubmits manually (nothing is auto-queued, since a safety removal must happen the moment it is confirmed, not later) |

## Validation Rules

**Option B -- Inline (simple validation not warranting a standalone Logic/Rule spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Note | Optional; maximum 500 characters | On change (input is blocked past 500) | "Your note can be up to 500 characters." (shown only if pasted text exceeds the limit; typing is capped silently at 500) |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Close or Cancel | Returns to the calling screen (no navigation) | -- |
| Successful submission, "Show me alternatives" tapped | FEAT-04.SPEC-001 (Meal Swap) | FEAT-04 (One-Tap Meal Swap) |
| Successful submission, Close tapped instead | Returns to the calling screen, now showing the meal as removed | -- |

## Data Model

**Creates:** Support Request -- kind (safety concern), raised_by (the reporting member), planned_meal/recipe (the reported meal and its recipe), note (the optional text entered here, up to 500 characters), status (Raised). Created by FEAT-02.SPEC-004 on submission from this screen.
**Reads:** Planned Meal -- night, recipe, for display in the dialog header.
**Updates:** None directly -- Planned Meal status and swap_history are updated by FEAT-02.SPEC-004, not by this screen.
**Deletes:** None.

## Business Rules

- Submitting from this screen always triggers FEAT-02.SPEC-004 (Safety Concern Intake & Removal), which removes the meal, excludes the recipe for the household, and creates the Support Request -- this screen never partially completes a report.
- FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy) governs the excluded recipe's re-offer eligibility once this report is submitted; this screen has no visibility into that eligibility state.
- XBR-08: A safety-concern report removes the meal from the plan at once, excludes the recipe while the report is open, drops its ingredients from the grocery list, offers safe alternatives, reaches the operator for review, and tells the household the outcome.

## Edge Cases

- **Adult opens the dialog on a meal another adult has already reported** -- The dialog still opens (this screen does not check for an existing open report); FEAT-02.SPEC-004 recognizes the meal already has an open Support Request and does not create a duplicate, and the submission confirms with the same "removed" message since the meal is already off the plan.
- **Adult taps "Report and remove" twice rapidly** -- The second tap is ignored while the first submission is in progress (button in loading state).
- **Meal is swapped out by another household member while this dialog is open** -- Submission proceeds against the meal reference captured when the dialog opened; if that slot no longer holds the reported recipe, FEAT-02.SPEC-004 still creates the report against the recipe as reported, and the confirmation message states the report was recorded, since the underlying safety concern about that recipe stands regardless of the slot's current contents. There is no concurrent-edit conflict here because this screen never re-saves the Planned Meal itself -- it only submits a new report, which FEAT-02.SPEC-004 owns.
- **Network failure during submission** -- Error banner: "Couldn't submit your report. Check your connection and try again." with a Retry button. Note text preserved.
- **Adult closes the dialog with unsaved note text** -- No confirmation dialog is shown; a note is not a commitment until submitted, so closing simply discards it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Triggers (outbound) | Submitting the report fires this automation |
| FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy) | References (outbound) | Governs the excluded recipe's re-offer state after submission |
| FEAT-04.SPEC-001 (Meal Swap alternatives list) | Navigation (outbound) | "Show me alternatives" opens the safe alternatives list for the emptied slot |
| FEAT-03 (AI Weekly Dinner Plan Generation), planned meal card | Navigation (inbound) | Entry point when reporting from the AI-generated plan |
| FEAT-23 (Manual Weekly Planning), planned meal card | Navigation (inbound) | Entry point when reporting from a manually picked plan |
| FEAT-19 (Weekly Plan History), re-used meal card | Navigation (inbound) | Entry point when reporting from a re-used past week |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| safety_concern_report_opened | entry source (plan / manual plan / history) | Dialog opens | supports success-metrics.md: "Zero Allergy Incidents" |
| safety_concern_reported | note provided (yes/no), entry source | Submission succeeds | supports success-metrics.md: "Zero Allergy Incidents" |
| safety_concern_report_failed | reason (network) | Submission fails | supports success-metrics.md: "Zero Allergy Incidents" |
| safety_concern_report_abandoned | note text entered (yes/no) | Adult closes or cancels without submitting | N/A -- no Stage 2 metric measures abandonment of this dialog; retained so the reporting flow's friction is observable |

## Acceptance Criteria

**FEAT-02.SPEC-001-AC-01:** Given Maya is viewing Thursday's dinner on her weekly plan, when she taps "report a safety concern" on it, then the dialog opens showing the recipe name and Thursday as the night.

**FEAT-02.SPEC-001-AC-02:** Given Maya has the dialog open with the note field empty, when she taps "Report and remove", then the meal is removed from the plan, the dialog shows a confirmation message, and a "Show me alternatives" action appears.

**FEAT-02.SPEC-001-AC-03:** Given Sam has the dialog open, when he types a note describing the concern and taps "Report and remove", then the report is submitted with his note attached and he sees the same confirmation Maya would see.

**FEAT-02.SPEC-001-AC-04:** Given Maya is typing in the note field, when her note reaches 500 characters, then further typing is blocked and the remaining-character count shows 0.

**FEAT-02.SPEC-001-AC-05:** Given Maya has the dialog open, when she taps the close control, then the dialog closes without submitting anything and no Support Request is created.

**FEAT-02.SPEC-001-AC-06:** Given Sam taps "Report and remove" and the submission fails due to a network error, then the error banner "Couldn't submit your report. Check your connection and try again." appears with a Retry button, and his note text is preserved.

**FEAT-02.SPEC-001-AC-07:** Given Maya taps "Report and remove" twice in rapid succession, when the first tap is already processing, then the second tap has no effect and only one report is submitted.

**FEAT-02.SPEC-001-AC-08:** Given Maya's session expires while the dialog is open with note text entered, when the session-expired dialog appears and she signs back in, then the safety concern dialog reopens with her note text restored.

**FEAT-02.SPEC-001-AC-09:** Given Sam successfully submits a report and sees the confirmation, when he taps "Show me alternatives", then he is taken to the safe alternatives list (FEAT-04.SPEC-001) for the emptied slot.

**FEAT-02.SPEC-001-AC-10:** Given Jordan is signed in through the Later-phase older-kid limited login, when Jordan views a planned meal on any screen that login can reach, then no "report a safety concern" control is shown.

**FEAT-02.SPEC-001-AC-11:** Given Maya loses connectivity while the dialog is open, then a banner states reporting needs a connection and "Report and remove" is disabled until connectivity returns.

**FEAT-02.SPEC-001-AC-12:** Given a meal already has an open safety report from Sam, when Maya opens the dialog on the same meal and submits her own report, then her submission still confirms as removed, and no duplicate Support Request is created (per FEAT-02.SPEC-004).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 4 (ready/filling, submitting error, offline, submitted) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Automation Spec: Candidate Safety Check Execution

## Overview

**Name:** Candidate Safety Check Execution
**ID:** FEAT-02.SPEC-002
**Type:** Automation
**Purpose:** Checks a candidate recipe's ingredients against every household member's hard rules before it can ever be shown, on every path onto the plan.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- Running the hard-rule safety check on every candidate recipe from every path onto a household's plan (AI generation, manual pick, swap, vote, history re-use)
- Attaching the "checked against allergies" safety badge to a recipe that passes
- Excluding a recipe from the household's candidate pool when it fails the check for any member, or when its ingredient data is incomplete

**Non-Goals:**
- Classifying which rule kinds are hard versus soft -- owned by FEAT-02.SPEC-006 (Rule Strength & Blocking Policy), which this automation reads
- Defining the fail-closed behavior for incomplete ingredient data in detail -- owned by FEAT-02.SPEC-007 (Ingredient Data Completeness & Fail-Closed Policy), which this automation applies
- Determining the badge's exact wording and disclaimer text, or the ineligibility reason's wording -- owned by FEAT-02.SPEC-008 (Safety Badge & Disclaimer Display Rule)
- Deciding whether a recipe under an open safety report may re-enter the pool -- owned by FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy), which this automation consults before running

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| AI plan generation proposes a candidate recipe | FEAT-03 (AI Weekly Dinner Plan Generation) | Fires once per candidate recipe considered during generation | Candidate Recipe (ingredients, cook_time), the household's full set of Dietary Rules |
| Adult searches or picks a recipe manually | FEAT-23 (Manual Weekly Planning) | Fires when a recipe is displayed in search results or attempted for placement | Candidate Recipe, the household's Dietary Rules |
| Adult requests swap alternatives | FEAT-04 (One-Tap Meal Swap) | Fires for each candidate alternative considered for the slot | Candidate Recipe, the household's Dietary Rules |
| A dinner-voting round is opened | FEAT-17 (Older-Kid Dinner Voting, Later) | Fires for each option before it can be included in the round | Candidate Recipe, the household's Dietary Rules |
| Household re-uses a past week as a starting point | FEAT-19 (Weekly Plan History) | Fires for every meal in the copied week before it returns to the plan | Candidate Recipe (as it exists now, not as it existed when originally checked), the household's current Dietary Rules |
| A hard rule is added or tightened for the current week | FEAT-02.SPEC-003 (Mid-Week Rule Change Re-Check) | Fires once per remaining dinner already on the plan, re-running this same check | The Planned Meal's Recipe, the household's updated Dietary Rules |
| An imported recipe is edited or re-imported | FEAT-10 (Recipe Import from Web Link) | Fires per XBR-19 before the edited recipe can appear in any plan again | The edited Recipe's current ingredient list, the household's Dietary Rules |

## Processing Logic

1. Receive the candidate Recipe and the requesting Household's identity from the triggering spec.
2. Consult FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy): if this Recipe is under an open safety report for this Household, exclude it immediately and skip the remaining steps.
3. Read the Recipe's ingredient list (quantity and unit per ingredient).
4. Verify the ingredient data is complete per FEAT-02.SPEC-007 (Ingredient Data Completeness & Fail-Closed Policy). If incomplete, exclude the Recipe and emit safety_check_data_incomplete; skip the remaining steps.
5. Read every Household Member's Dietary Rules for this Household.
6. For each Member, compare every ingredient against the Member's hard rules (per FEAT-02.SPEC-006: allergy and religious-rule entries, and the per-person vegetarian setting where it applies) using the full ingredient text -- no partial or fuzzy matching.
7. If any ingredient matches any Member's allergen (from the standard allergen list or a named extra ingredient) or violates a religious rule, mark the Recipe as failing for that Member.
8. If the Recipe is a shared dinner and any vegetarian Member's vegetarian setting is not met by the base recipe, check whether the Recipe carries a vegetarian_option variant; if it does, the base recipe is not excluded on vegetarian grounds alone -- the variant satisfies that Member (see Business Rules).
9. If the Recipe fails for any Member on an allergy or religious-rule basis (vegetarian handled per Step 8), exclude the Recipe from the candidate pool for this Household entirely.
10. If the Recipe passes for every Member, attach the safety_badge (per FEAT-02.SPEC-008) to the candidate before it can be shown or placed.
11. Return the pass/exclude determination, plus the ineligibility reason (per FEAT-02.SPEC-008's plain-phrasing pattern) when excluded, to the triggering spec.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Recipe passes | No hard-rule failure for any Member and ingredient data is complete | Planned Meal's safety_badge field is set when the recipe is placed | Recipe displays with the "checked against allergies" badge and disclaimer | FEAT-03, FEAT-04, FEAT-08, FEAT-10, FEAT-17, FEAT-19, FEAT-23, FEAT-02.SPEC-008 |
| Recipe excluded -- hard-rule failure | The recipe violates an allergy or religious rule for at least one Member | None -- the recipe never enters the candidate pool shown to the household | Recipe does not appear in results; where a specific recipe was searched for directly, the plain ineligibility reason is shown ("contains {allergen} -- not safe for {member}") | FEAT-03, FEAT-04, FEAT-08, FEAT-17, FEAT-19, FEAT-23, FEAT-02.SPEC-008 |
| Recipe excluded -- incomplete data | The Recipe's ingredient data cannot be fully verified | None | Recipe does not appear in results; ineligibility reason states the data is incomplete, per FEAT-02.SPEC-007/008 | FEAT-02.SPEC-007, FEAT-08, FEAT-03, FEAT-23 |
| Recipe excluded -- under open safety report | The Recipe is currently excluded per FEAT-02.SPEC-009 | None | Recipe does not appear in results for this household while the report is open | FEAT-02.SPEC-009, FEAT-02.SPEC-004 |
| Shared-meal vegetarian satisfied via variant | The base recipe does not meet a vegetarian Member's setting but a vegetarian_option variant exists | None beyond badge attachment | The plan shows the vegetarian variant is available for the vegetarian household member | FEAT-02.SPEC-006, FEAT-03, FEAT-23 |
| Check cannot complete (processing failure) | The check itself cannot run to completion for reasons other than incomplete ingredient data | None -- fails closed | Recipe is excluded, treated identically to the "incomplete data" outcome, since an unverifiable check must never default to "safe" | FEAT-02.SPEC-007, FEAT-03, FEAT-08, FEAT-23 |

## Data Model

**Reads:** Dietary Rule -- member, rule_kind, strength, allergen, for every household member. Recipe -- ingredients (quantity and unit), dietary_badges, for the candidate under check.
**Creates:** None.
**Updates:** Planned Meal -- safety_badge (set when a candidate passes and is placed onto the plan). Recipe -- dietary_badges (computed per household when viewed, not persisted per household).
**Deletes:** None.

## Business Rules

- XBR-01: Every path onto the plan -- AI generation, manual picks, swaps and swap/pick suggestions, voting options, and re-used past weeks -- passes this same app-enforced check before anyone sees it; the check fails closed, and every shown meal carries the badge and disclaimer.
- Allergy and religious-rule failures always exclude a recipe entirely for the household -- there is no partial or per-member visibility of an unsafe recipe (per FEAT-02.SPEC-006).
- A shared meal that is not inherently vegetarian may still pass for a household with a vegetarian member if the recipe carries a vegetarian_option variant; the base recipe is not treated as failing on vegetarian grounds alone, per FEAT-02.SPEC-006.
- Dislikes (soft rules) never factor into this check -- they influence selection ranking elsewhere (FEAT-03, FEAT-12) but cannot exclude a recipe here, per FEAT-02.SPEC-006.
- The check runs synchronously and completes before a candidate is ever shown to any household member, on any path -- there is no state where an unchecked recipe is visible.
- This automation is the single owner of the pass/exclude determination; no other spec in the product performs its own allergy or religious-rule matching.

## Edge Cases

- **Recipe has zero ingredients recorded** -- Treated as incomplete ingredient data (FEAT-02.SPEC-007); excluded, never assumed safe by default.
- **Member has no dietary rules recorded at all** -- Absence of rules is not the same as "no restrictions" being explicitly stated; per FEAT-01's setup flow every member has at least an explicit "no restrictions" statement recorded, so this automation always has a rule set (possibly empty by explicit statement) to check against.
- **Two members share the same allergen but different named extra ingredients** -- Each member's rule set is checked independently against the full ingredient list; a match against either member's allergen or extra ingredient excludes the recipe for the whole household.
- **Recipe passes for AI generation but a member's hard rule is added moments later, before the plan is shown** -- The check reflects the Dietary Rule data as read at the moment this automation runs; a rule change after the check completes but before display is handled by FEAT-02.SPEC-003 (Mid-Week Rule Change Re-Check) once the rule is saved, not by this automation re-running speculatively.
- **Concurrent trigger firing (two paths check the same recipe for the same household at effectively the same time, e.g., AI generation and a manual search)** -- Each run reads the same Dietary Rule and Recipe data and reaches the same determination independently; both complete without blocking each other, since the check is read-only against Dietary Rule and Recipe and its only write (safety_badge) is scoped to the Planned Meal the calling spec is placing.
- **Trigger fires while a previous run for the same recipe and household is in flight** -- Each run is independent and stateless; a second run for the same recipe does not need to wait on the first, since neither run mutates shared state that the other reads mid-check.
- **Ingredient text contains an ambiguous or compound term (e.g., "mixed nuts")** -- Treated as matching every allergen the compound term could reasonably contain (e.g., matches a peanut, tree-nut, or specific named-nut allergy); no partial matching shortcut narrows a compound ingredient to a subset of its possible allergens.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03 (AI Weekly Dinner Plan Generation) | Triggered by (inbound) | Every AI-proposed candidate is checked before inclusion |
| FEAT-23 (Manual Weekly Planning) | Triggered by (inbound) | Every manually searched or picked recipe is checked |
| FEAT-04 (One-Tap Meal Swap) | Triggered by (inbound) | Every swap alternative is checked before it is offered |
| FEAT-17 (Older-Kid Dinner Voting) | Triggered by (inbound) | Every voting option is checked before a round opens |
| FEAT-19 (Weekly Plan History) | Triggered by (inbound) | A re-used past week's meals are re-checked before returning to the plan |
| FEAT-02.SPEC-003 (Mid-Week Rule Change Re-Check) | Triggered by (inbound) | Re-runs this check against every remaining dinner when a hard rule changes |
| FEAT-01 (Household Setup & Member Profiles) | Reads (outbound) | Source of every household member's Dietary Rule data |
| FEAT-02.SPEC-006 (Rule Strength & Blocking Policy) | References (outbound) | Classifies which rules are hard filters versus soft |
| FEAT-02.SPEC-007 (Ingredient Data Completeness & Fail-Closed Policy) | References (outbound) | Governs the incomplete-data exclusion path |
| FEAT-02.SPEC-008 (Safety Badge & Disclaimer Display Rule) | Affects (outbound) | Supplies the badge and ineligibility-reason wording this automation attaches |
| FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy) | References (outbound) | Consulted first to exclude recipes under an open report |
| FEAT-08 (Recipe Library) / FEAT-10 (Recipe Import) | Affects (outbound) | Source of the candidate Recipe data this automation checks |

## Analytics and Success Signals

- **safety_check_run** (household reference, recipe reference, trigger source) -- supports success-metrics.md: "Zero Allergy Incidents"
- **safety_check_failed** (allergen or rule matched, member reference, trigger source) -- supports success-metrics.md: "Zero Allergy Incidents"
- **safety_badge_shown** (recipe reference, trigger source) -- supports success-metrics.md: "Zero Allergy Incidents"
- **safety_check_data_incomplete** (recipe reference) -- supports success-metrics.md: "Zero Allergy Incidents"

## Acceptance Criteria

**FEAT-02.SPEC-002-AC-01:** Given Maya's household has a member with a peanut allergy, when the AI plan generation considers a recipe containing peanuts, then this automation excludes the recipe from the candidate pool and it never appears in the generated plan.

**FEAT-02.SPEC-002-AC-02:** Given a candidate recipe contains no ingredient that violates any household member's hard rules and has complete ingredient data, when the check runs, then the recipe passes and carries the "checked against allergies" badge once placed.

**FEAT-02.SPEC-002-AC-03:** Given Sam searches the Recipe Library for a dinner during Manual Weekly Planning, when a recipe in the results would violate his child's allergy, then that recipe is excluded from the results and shows the plain ineligibility reason.

**FEAT-02.SPEC-002-AC-04:** Given Maya requests swap alternatives for Friday's dinner, when the alternatives list is generated, then every alternative shown has already passed this safety check.

**FEAT-02.SPEC-002-AC-05:** Given a household re-uses a past week from Weekly Plan History, when the copied week's meals are re-checked, then any meal that no longer passes (e.g., a rule added since it was last checked) is excluded from the copy.

**FEAT-02.SPEC-002-AC-06:** Given a candidate recipe has an incomplete ingredient list (missing quantity or unit on one item), when the check runs, then the recipe is excluded and safety_check_data_incomplete is emitted, regardless of whether the visible ingredients would otherwise pass.

**FEAT-02.SPEC-002-AC-07:** Given a shared dinner recipe is not inherently vegetarian but carries a vegetarian_option variant, when a household with one vegetarian member and one non-vegetarian member is checked, then the recipe passes and the vegetarian variant is made available rather than the recipe being excluded.

**FEAT-02.SPEC-002-AC-08:** Given a household member has a recorded dislike of mushrooms (a soft rule) and no allergy to them, when a recipe containing mushrooms is checked, then the recipe passes this safety check regardless of the dislike.

**FEAT-02.SPEC-002-AC-09:** Given a recipe is currently under an open safety report for Maya's household (FEAT-02.SPEC-009), when any path attempts to check that recipe again, then it is excluded without re-running the ingredient comparison.

**FEAT-02.SPEC-002-AC-10:** Given Maya adds a new allergy for one of her children (FEAT-02.SPEC-003 triggers this automation for each remaining dinner), when a remaining dinner's recipe now contains that allergen, then this automation excludes it from the current week just as it would for a new candidate.

**FEAT-02.SPEC-002-AC-11:** Given an older kid's dinner-voting round is being opened (Later), when each candidate option is checked, then only options that pass this automation's check are included in the round.

**FEAT-02.SPEC-002-AC-12:** Given two different features check the same candidate recipe for the same household at effectively the same time, when both checks run, then both complete independently and reach the same pass/exclude determination without blocking each other.

**FEAT-02.SPEC-002-AC-13:** Given an imported recipe is edited to add a new ingredient (FEAT-10, per XBR-19), when the edited recipe is next considered as a candidate, then this automation re-checks it against every household's rules before it can appear in any plan again.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 7 | 7 |
| Outcome Paths | 6 | 6 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |



# Automation Spec: Mid-Week Rule Change Re-Check

## Overview

**Name:** Mid-Week Rule Change Re-Check
**ID:** FEAT-02.SPEC-003
**Type:** Automation
**Purpose:** Re-verifies every remaining dinner in the current week the moment a hard rule is added or tightened, so a newly unsafe meal never stays on the plan.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- Detecting when a new or tightened hard rule (allergy or religious rule) is saved for any household member
- Re-running the candidate safety check against every remaining dinner in the current week's plan
- Removing any dinner that now fails, updating the grocery list, offering safe alternatives, and telling the organiser

**Non-Goals:**
- Capturing or saving the dietary rule change itself -- owned by FEAT-01 (Household Setup & Member Profiles), whose save action is this automation's trigger
- Running the actual ingredient-versus-rule comparison logic -- delegated to FEAT-02.SPEC-002 (Candidate Safety Check Execution), which this automation invokes per remaining dinner
- Re-checking a softened or removed rule -- excluded per the feature's own scope: a rule becoming less restrictive (removing an allergy, loosening a religious rule) cannot make an already-approved meal unsafe, so no re-check is needed; only a new or tightened hard rule triggers this automation, per the Brief's Side-Effect Inventory
- Selecting the specific replacement recipe for a removed slot -- owned by FEAT-04 (One-Tap Meal Swap), which this automation opens on the household's behalf

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A hard rule (allergy or religious rule) is added for a household member | FEAT-01 (Household Setup & Member Profiles) | Fires when Maya saves a new Dietary Rule with strength = hard | The affected Member, the new Dietary Rule (rule_kind, allergen), the Household's current Weekly Plan |
| A hard rule is tightened for a household member | FEAT-01 (Household Setup & Member Profiles) | Fires when Maya edits an existing hard Dietary Rule to add a named extra ingredient or otherwise narrow what is safe | The affected Member, the updated Dietary Rule, the Household's current Weekly Plan |

## Processing Logic

1. Receive the Household reference and the changed Dietary Rule (added or tightened, strength = hard) from FEAT-01.
2. Read the Household's current Weekly Plan and identify every remaining dinner Planned Meal (nights that have not yet passed).
3. For each remaining dinner, invoke FEAT-02.SPEC-002 (Candidate Safety Check Execution) against that dinner's Recipe, using the Household's now-updated full Dietary Rule set.
4. For each dinner that still passes, leave it unchanged on the plan.
5. For each dinner that now fails, set its Planned Meal status to Removed (safety) and append the prior recipe to swap_history.
6. For every removed dinner, trigger the grocery list recalculation (FEAT-06) to drop that dinner's ingredients.
7. For every removed dinner, open a swap for the now-empty slot (FEAT-04) so a safe alternative can be selected.
8. If one or more dinners were removed, trigger FEAT-02.SPEC-012 (Safety Concern Organiser Alert) to tell the organiser which dinners were removed and why (rule change, not a safety-concern report).
9. If no remaining dinner fails, complete silently -- the rule change is saved with no visible plan disruption.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| No dinners affected | Every remaining dinner still passes the re-check | None | None -- the rule save completes normally with no plan disruption message | FEAT-01 |
| One or more dinners removed | At least one remaining dinner now fails the re-check | Affected Planned Meal(s) set to Removed (safety); swap_history appended; grocery list recalculated | Affected slot(s) show as empty with the plain reason (FEAT-02.SPEC-008); a swap is opened offering safe alternatives; the organiser receives the removal alert (FEAT-02.SPEC-012) | FEAT-04, FEAT-06, FEAT-02.SPEC-012, FEAT-02.SPEC-008 |
| Re-check itself cannot complete for a dinner | The safety check (FEAT-02.SPEC-002) cannot verify a dinner's recipe (e.g., incomplete data surfaces only now) | That dinner is treated as failing and removed -- fails closed, consistent with FEAT-02.SPEC-007 | Same as "one or more dinners removed" for that slot | FEAT-04, FEAT-06, FEAT-02.SPEC-012, FEAT-02.SPEC-007 |

## Data Model

**Reads:** Dietary Rule -- the changed rule and every other member's current rules. Weekly Plan -- the current week's remaining Planned Meals. Recipe -- ingredients for each remaining dinner, via FEAT-02.SPEC-002.
**Creates:** None directly (FEAT-02.SPEC-004's Support Request mechanism does not apply here -- this is a rule-change removal, not a reported concern).
**Updates:** Planned Meal -- status (set to Removed (safety) for failing dinners), swap_history (prior recipe appended).
**Deletes:** None -- removal is a status change, not a hard delete, matching the Planned Meal lifecycle in the dependency map.

## Business Rules

- XBR-02: A new or tightened hard rule takes effect on the current week immediately: remaining dinners are re-checked, any that now fail are flagged and removed, safe alternatives are offered through swap, the grocery list updates, and the organiser is told.
- Only remaining (not-yet-cooked) dinners are re-checked -- a dinner already marked Cooked is history and is never retroactively altered.
- A removal from this automation uses the same Removed (safety) status and mechanic as a safety-concern removal (FEAT-02.SPEC-004); the two share one lifecycle transition on Planned Meal.
- This automation never touches Dietary Rule data itself -- it only reacts to a change FEAT-01 has already saved.
- A rule change that only loosens or removes a restriction never triggers this automation, since it cannot newly endanger an already-safe meal.

## Edge Cases

- **Two hard rules are added in quick succession for different members** -- Each triggers its own re-check run; the second run reads the Weekly Plan as it stands after the first run's removals, so a dinner already removed by the first change is not re-processed by the second.
- **The changed member has no remaining dinners left in the week (all already cooked)** -- The re-check runs, finds no remaining dinners to evaluate, and completes with the "no dinners affected" outcome.
- **A dinner already has an open safety report when the rule change re-check runs** -- The dinner is already excluded from candidacy per FEAT-02.SPEC-009; the re-check still evaluates its current placement on the plan for the new rule and removes it under the rule-change path if it also fails the new rule, without creating a duplicate Support Request.
- **Concurrent trigger firing (two members' hard rules change at effectively the same time, e.g., Maya edits two profiles back-to-back)** -- Each rule save fires its own re-check run against the plan state at that moment; the second run reflects any removals the first run already made, so no dinner is independently removed twice for the same reason, though a dinner could be removed once by each run if it fails both new rules (recorded once, since the Planned Meal's status only needs to be Removed (safety) regardless of how many rules it failed).
- **Trigger fires while a previous re-check run is still in flight** -- A second rule-change trigger for the same household waits for the first run to finish reading and updating the Weekly Plan before it begins its own pass, so the two runs never evaluate the plan against inconsistent intermediate states.
- **Household has no remaining dinners planned at all this week (a free-tier household mid-way through manual planning with empty nights)** -- The re-check finds zero remaining dinners to evaluate and completes with no visible effect; empty nights are not Planned Meals and are not in scope for removal.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01 (Household Setup & Member Profiles) | Triggered by (inbound) | A hard-rule save fires this automation |
| FEAT-02.SPEC-002 (Candidate Safety Check Execution) | Triggers (outbound) | Re-runs the same check against every remaining dinner |
| FEAT-04 (One-Tap Meal Swap) | Affects (outbound) | A swap is opened for every emptied slot |
| FEAT-06 (Shared Grocery List) | Affects (outbound) | The list is recalculated to drop removed dinners' ingredients |
| FEAT-02.SPEC-012 (Safety Concern Organiser Alert) | Triggers (outbound) | Tells the organiser which dinners were removed and why |
| FEAT-02.SPEC-008 (Safety Badge & Disclaimer Display Rule) | References (outbound) | Governs the plain reason shown on the now-empty slot |
| FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy) | References (outbound) | A dinner already excluded under an open report is not double-counted as a new removal reason |

## Analytics and Success Signals

- **midweek_rule_change_recheck_run** (household reference, remaining-dinner count evaluated) -- supports success-metrics.md: "Zero Allergy Incidents"
- **midweek_rule_change_meal_removed** (recipe reference, member/allergen matched) -- supports success-metrics.md: "Zero Allergy Incidents"
- **midweek_rule_change_no_impact** (household reference) -- N/A -- no Stage 2 metric measures rule changes with zero plan impact; retained so the re-check's actual disruption rate is observable

## Acceptance Criteria

**FEAT-02.SPEC-003-AC-01:** Given Maya adds a new peanut allergy for one of her children mid-week, when the save completes, then this automation re-checks every remaining dinner in the current week's plan.

**FEAT-02.SPEC-003-AC-02:** Given a remaining Wednesday dinner contains peanuts and Maya just added a peanut allergy for her child, when the re-check runs, then Wednesday's dinner is set to Removed (safety), the grocery list drops its ingredients, and a swap opens offering safe alternatives.

**FEAT-02.SPEC-003-AC-03:** Given none of the remaining dinners contain the newly restricted allergen, when the re-check completes, then no dinner is removed and no organiser alert fires.

**FEAT-02.SPEC-003-AC-04:** Given one or more dinners are removed by this automation, when the removal completes, then Maya (the organiser) receives the safety concern organiser alert (FEAT-02.SPEC-012) naming the removed meal and the rule-change reason.

**FEAT-02.SPEC-003-AC-05:** Given Maya tightens an existing halal rule by naming a specific extra ingredient to avoid, when the save completes, then this automation re-checks remaining dinners against the updated rule, not just the original rule_kind.

**FEAT-02.SPEC-003-AC-06:** Given Maya removes (rather than adds) a hard allergy rule, when the save completes, then this automation does not fire, since a loosened rule cannot newly endanger a meal.

**FEAT-02.SPEC-003-AC-07:** Given a dinner already marked Cooked exists in the current week when a hard rule changes, when the re-check runs, then the Cooked dinner is not evaluated or altered.

**FEAT-02.SPEC-003-AC-08:** Given a remaining dinner's recipe now has incomplete ingredient data discovered only at re-check time, when the re-check runs, then that dinner is removed under the fail-closed policy (FEAT-02.SPEC-007), the same as an explicit rule violation.

**FEAT-02.SPEC-003-AC-09:** Given two hard-rule changes are saved back-to-back for different members, when both re-checks run, then a dinner removed by the first change is not independently reprocessed as a new removal by the second.

**FEAT-02.SPEC-003-AC-10:** Given a household has no remaining dinners left this week when a hard rule changes, when the re-check runs, then it completes with no plan impact and no organiser alert.

**FEAT-02.SPEC-003-AC-11:** Given a second hard-rule change for the same household fires while an earlier re-check run is still in progress, when the second trigger arrives, then it waits for the first run to finish before evaluating the plan, so neither run acts on an inconsistent intermediate state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Safety Concern Intake & Removal

## Overview

**Name:** Safety Concern Intake & Removal
**ID:** FEAT-02.SPEC-004
**Type:** Automation
**Purpose:** Removes a reported meal from the plan immediately, excludes the recipe for the household while the report is open, and opens the review record.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- Removing the reported Planned Meal from the household's plan at once
- Creating the Support Request (kind = safety concern) that opens the review record
- Excluding the reported recipe from the household's future candidate pool while the report is open
- Updating the grocery list and offering safe alternatives for the emptied slot

**Non-Goals:**
- Capturing the report itself (the note, the meal reference) -- owned by FEAT-02.SPEC-001 (Report a Safety Concern), whose submission triggers this automation
- Deciding whether the excluded recipe may return to the candidate pool once reviewed -- owned by FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy)
- Applying the operator's review outcome -- owned by FEAT-02.SPEC-005 (Safety Concern Resolution Outcome), a distinct later step this automation does not perform
- Reviewing the report or changing its status beyond Raised -- owned by FEAT-22 (Operator Read-Only Support Access), which this automation's created Support Request feeds

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An adult submits a safety concern report | FEAT-02.SPEC-001 (Report a Safety Concern) | Fires on every submission from the dialog, regardless of whether the recipe already has an open report | Reporting Member, the reported Planned Meal (and its Recipe), the optional note (up to 500 characters) |

## Processing Logic

1. Receive the reporting Member, the reported Planned Meal reference, its Recipe, and the optional note from FEAT-02.SPEC-001.
2. Check whether the reported Recipe already has an open Support Request (kind = safety concern, status not Resolved) for this Household.
3. If no open report exists for this Recipe, create a new Support Request: kind = safety concern, raised_by = the reporting Member, planned_meal/recipe = the reported meal and recipe, note = the optional text (or none), status = Raised.
4. If an open report already exists for this Recipe, do not create a duplicate Support Request -- proceed to the remaining steps against the existing report.
5. Set the reported Planned Meal's status to Removed (safety) and append the prior recipe to swap_history.
6. Mark the Recipe as excluded from this Household's candidate pool per FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy), for as long as the Support Request remains open.
7. Trigger the grocery list recalculation (FEAT-06) to drop the removed meal's ingredients.
8. Open a swap for the now-empty slot (FEAT-04) so a safe alternative can be selected.
9. Trigger FEAT-02.SPEC-011 (Safety Concern Reporter Acknowledgement) to confirm receipt and removal to the reporting Member.
10. If the reporting Member is not the organiser, trigger FEAT-02.SPEC-012 (Safety Concern Organiser Alert) to tell the organiser.
11. Trigger FEAT-02.SPEC-013 (Safety Concern Operator Alert), which delivers the report to the operator by transactional email through FEAT-02.SPEC-010.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| New report opened, meal removed | No open report existed for this Recipe | New Support Request created (status Raised); Planned Meal set to Removed (safety); Recipe excluded per FEAT-02.SPEC-009; grocery list updated | Reporter sees the removal confirmation (FEAT-02.SPEC-001); organiser alerted if not the reporter; operator alerted by email | FEAT-02.SPEC-001, FEAT-02.SPEC-011, FEAT-02.SPEC-012, FEAT-02.SPEC-013, FEAT-06, FEAT-04, FEAT-02.SPEC-009 |
| Duplicate report against an already-open concern | An open Support Request for this Recipe already exists | No new Support Request created; Planned Meal removal and recipe exclusion proceed (or are confirmed as already applied if the meal was already removed) | Reporter sees the same removal confirmation as a new report | FEAT-02.SPEC-001, FEAT-02.SPEC-011, FEAT-06, FEAT-04 |
| Meal already removed before this report (e.g., by a mid-week rule change) | The reported Planned Meal is already Removed (safety) from a prior cause | Support Request still created (or reused per the duplicate path) so the concern is formally recorded; Planned Meal status unchanged (already Removed) | Reporter sees a confirmation that the concern was recorded; no additional plan disruption since the meal is already gone | FEAT-02.SPEC-001, FEAT-02.SPEC-011, FEAT-02.SPEC-013 |
| Automation cannot complete (processing failure) | The removal or Support Request creation cannot be completed | No partial state -- either the full removal and report creation succeed together or neither does | Reporter sees the error state defined in FEAT-02.SPEC-001 ("Couldn't submit your report...") with a Retry option | FEAT-02.SPEC-001 |

## Data Model

**Reads:** Planned Meal -- current status, recipe reference. Support Request -- existing open reports for the same Recipe and Household, to detect duplicates.
**Creates:** Support Request -- kind (safety concern), raised_by, planned_meal/recipe, note, status (Raised).
**Updates:** Planned Meal -- status (Removed (safety)), swap_history (prior recipe appended).
**Deletes:** None -- removal is a status change; the Planned Meal record is retained in the plan's history, per the dependency map's Planned Meal lifecycle.

## Business Rules

- XBR-08: A safety-concern report removes the meal from the household's plan at once, excludes the recipe for that household while the report is open, drops its ingredients from the list, offers safe alternatives, reaches the operator for review, and tells the household the outcome.
- This automation never creates a second open Support Request for the same recipe while one is already open -- FEAT-02.SPEC-009 is the single source of truth for whether a recipe is under report, and this automation defers to it rather than tracking its own exclusion state.
- Removal is immediate and unconditional on submission -- there is no pending or under-review state in which the meal remains visible on the plan.
- A safety-concern removal always uses the same Removed (safety) status and swap_history mechanic as a mid-week rule-change removal (FEAT-02.SPEC-003); the two share one lifecycle transition on Planned Meal.
- Support Requests raised as safety concerns are never deleted or purged -- retained for the life of the household account as trust and audit history (SC-18).

## Edge Cases

- **Two different adults report the same meal within moments of each other** -- The second submission finds the first's Support Request already open (or being created) and does not create a duplicate; both reporters receive their own reporter acknowledgement, since each genuinely reported independently.
- **The reported meal has already been swapped out by the time the report is submitted** -- The removal and exclusion apply to the Recipe as reported, regardless of whether it currently occupies the slot; if it no longer occupies any slot, no Planned Meal status change occurs but the Support Request and recipe exclusion still proceed.
- **Reporting adult submits with no note** -- The Support Request is created with note = none; the operator alert and reporter acknowledgement proceed with no note content shown.
- **Concurrent trigger firing (two different meals reported by two different adults at effectively the same time)** -- Each report processes independently against its own Planned Meal and Recipe; there is no shared state between the two runs beyond the household's Support Request list, which each run reads and appends to without overwriting the other's write.
- **Trigger fires while a previous report for the same recipe is still being processed** -- The later trigger waits for the in-flight run's Support Request duplicate-check to complete before proceeding, so two near-simultaneous reports for the same recipe cannot both create separate open Support Requests.
- **Network interruption after the Support Request is created but before the Planned Meal status updates** -- The automation treats meal removal and Support Request creation as one atomic outcome for the user; if the removal cannot be confirmed, the reporter sees the failure state (FEAT-02.SPEC-001) and the reporting flow can be retried without creating a second Support Request, since the duplicate check on retry finds the one already created.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Report a Safety Concern) | Triggered by (inbound) | Submitting the report fires this automation |
| FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy) | Triggers (outbound) | Marks the recipe excluded while the report is open |
| FEAT-06 (Shared Grocery List) | Affects (outbound) | The list is recalculated to drop the removed meal's ingredients |
| FEAT-04 (One-Tap Meal Swap) | Affects (outbound) | A swap is opened for the emptied slot |
| FEAT-02.SPEC-011 (Safety Concern Reporter Acknowledgement) | Triggers (outbound) | Confirms receipt and removal to the reporter |
| FEAT-02.SPEC-012 (Safety Concern Organiser Alert) | Triggers (outbound) | Tells the organiser when she was not the reporter |
| FEAT-02.SPEC-013 (Safety Concern Operator Alert) | Triggers (outbound) | Sends the report to the operator for review |
| FEAT-22 (Operator Read-Only Support Access) | Affects (outbound) | The created Support Request is the precondition for the operator's support view |

## Analytics and Success Signals

- **safety_concern_reported** (recipe reference, reporter role, had_note: yes/no) -- supports success-metrics.md: "Zero Allergy Incidents"
- **safety_concern_meal_removed** (recipe reference, plan slot) -- supports success-metrics.md: "Zero Allergy Incidents"
- **safety_concern_duplicate_suppressed** (recipe reference) -- N/A -- no Stage 2 metric measures duplicate report suppression; retained so the pipeline's deduplication behavior is observable rather than silent

## Acceptance Criteria

**FEAT-02.SPEC-004-AC-01:** Given Maya submits a safety concern report with no prior open report on that recipe, when this automation runs, then a new Support Request is created with status Raised and Maya's optional note attached.

**FEAT-02.SPEC-004-AC-02:** Given a safety concern report is processed, when the Planned Meal is removed, then its status is set to Removed (safety), the prior recipe is appended to swap_history, and the grocery list drops its ingredients.

**FEAT-02.SPEC-004-AC-03:** Given Sam (not the organiser) submits a safety concern, when the removal completes, then Maya (the organiser) receives the Safety Concern Organiser Alert (FEAT-02.SPEC-012).

**FEAT-02.SPEC-004-AC-04:** Given Maya (the organiser) submits a safety concern herself, when the removal completes, then no organiser alert fires for her own report, since she is already the reporter.

**FEAT-02.SPEC-004-AC-05:** Given a safety concern report is created, when the automation completes, then the operator receives the Safety Concern Operator Alert (FEAT-02.SPEC-013) by transactional email.

**FEAT-02.SPEC-004-AC-06:** Given Sam reports a recipe that Maya already reported minutes earlier, when Sam's report is processed, then no duplicate Support Request is created, and Sam still receives his own reporter acknowledgement.

**FEAT-02.SPEC-004-AC-07:** Given a report is submitted with no note, when the Support Request is created, then note is recorded as none and the operator alert and reporter acknowledgement proceed without note content.

**FEAT-02.SPEC-004-AC-08:** Given a recipe is excluded by this automation, when any future candidate check runs for that household (FEAT-02.SPEC-002), then the recipe is excluded per FEAT-02.SPEC-009 for as long as the report remains open.

**FEAT-02.SPEC-004-AC-09:** Given the reported meal was already removed by a mid-week rule change before the report was submitted, when the report is processed, then the Support Request is still created, but no additional Planned Meal status change occurs.

**FEAT-02.SPEC-004-AC-10:** Given a swap is opened for the emptied slot by this automation, when Maya opens the swap, then she sees safe alternatives for that night (FEAT-04.SPEC-001).

**FEAT-02.SPEC-004-AC-11:** Given two adults report two different meals at effectively the same time, when both reports are processed, then each creates its own Support Request and neither run interferes with the other's data.

**FEAT-02.SPEC-004-AC-12:** Given a second report for the same recipe arrives while the first report's processing is still in flight, when the second trigger fires, then it waits for the first run's duplicate check before proceeding, so only one Support Request is created for that recipe.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Safety Concern Resolution Outcome

## Overview

**Name:** Safety Concern Resolution Outcome
**ID:** FEAT-02.SPEC-005
**Type:** Automation
**Purpose:** Applies the operator's review outcome to the excluded recipe's eligibility and triggers the household's outcome notice.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- Reacting to the operator's Resolved transition on a safety-concern Support Request (FEAT-22)
- Releasing the recipe back to the household's candidate pool, or keeping it permanently excluded, based on the recorded outcome
- Triggering the household's resolution notice

**Non-Goals:**
- Performing the operator's review itself, or writing the Support Request's status or access_record -- owned entirely by FEAT-22 (Operator Read-Only Support Access); this automation only consumes the terminal Resolved transition
- Removing the meal or creating the Support Request -- already done by FEAT-02.SPEC-004 (Safety Concern Intake & Removal), a prior step this automation does not repeat
- Deciding what "released" means for future candidate checks in detail -- owned by FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy), which this automation updates
- Restoring the specific removed Planned Meal slot -- excluded per the feature's own scope: a fresh safe pick fills the slot through FEAT-04, the resolution does not undo the removal itself

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A safety-concern Support Request is resolved | FEAT-22 (Operator Read-Only Support Access) | Fires when the operator (Riley) transitions the Support Request's status from Under review to Resolved | Support Request (kind = safety concern, recipe reference, raised_by), the operator's recorded resolution outcome (recipe confirmed safe / recipe confirmed unsafe) |

## Processing Logic

1. Receive the resolved Support Request and its recorded outcome from FEAT-22.
2. Identify the Recipe the Support Request references and the Household it belongs to.
3. If the outcome confirms the recipe is safe (the reported concern did not hold up), instruct FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy) to release the recipe back to this Household's candidate pool.
4. If the outcome confirms the recipe is genuinely unsafe, instruct FEAT-02.SPEC-009 to keep the recipe permanently excluded for this Household (distinct from the temporary "open report" exclusion -- this is a confirmed, standing exclusion).
5. Trigger FEAT-02.SPEC-014 (Safety Concern Resolution Notice) to tell the household the outcome.
6. Take no action on the previously removed Planned Meal slot itself -- that slot remains empty (or however it was subsequently filled by a swap) regardless of the outcome.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Recipe released | Operator confirms the recipe is safe | Recipe's exclusion for this Household is lifted per FEAT-02.SPEC-009 -- it may pass future candidate checks again | Household receives the resolution notice stating the recipe is safe and available again | FEAT-02.SPEC-009, FEAT-02.SPEC-014, FEAT-02.SPEC-002 |
| Recipe kept excluded | Operator confirms the recipe is genuinely unsafe for this household | Recipe's exclusion for this Household becomes a standing exclusion per FEAT-02.SPEC-009 -- it never re-enters this household's candidate pool | Household receives the resolution notice stating the recipe remains excluded and why | FEAT-02.SPEC-009, FEAT-02.SPEC-014, FEAT-02.SPEC-002 |
| Resolution cannot be applied (processing failure) | The eligibility instruction to FEAT-02.SPEC-009 or the notice trigger to FEAT-02.SPEC-014 cannot be completed | No partial state -- the eligibility update and the resolution notice are treated as one atomic outcome; the recipe's exclusion state stays exactly as it was before this automation ran (fails closed, consistent with the feature's fail-closed posture) until a retry succeeds | Household receives no resolution notice until the retry succeeds; the Support Request stays Resolved in FEAT-22 (that write is not undone), but this automation retries the eligibility update and notice trigger automatically without requiring the operator to re-resolve it | FEAT-02.SPEC-009, FEAT-02.SPEC-014 |

## Data Model

**Reads:** Support Request -- status, kind, planned_meal/recipe, raised_by, and the operator's recorded resolution outcome (written by FEAT-22).
**Creates:** None.
**Updates:** None directly on Support Request (FEAT-22 owns that write); instructs FEAT-02.SPEC-009's exclusion-state tracking for the Recipe/Household pair.
**Deletes:** None.

## Business Rules

- XBR-08: A safety-concern report reaches the operator for review and tells the household the outcome once resolved -- this automation is the household-facing half of that lifecycle, paired with FEAT-02.SPEC-004's intake half.
- Only a Resolved transition on a safety-concern kind Support Request triggers this automation -- a general support Support Request's resolution (FEAT-18) has no recipe-eligibility consequence and does not invoke this spec.
- The recipe's eligibility state (released or kept excluded) is owned exclusively by FEAT-02.SPEC-009; this automation issues the instruction but never writes candidate-pool state itself.
- A "kept excluded" outcome is a standing decision distinct from the temporary open-report exclusion FEAT-02.SPEC-004 applied -- resolving a report to "unsafe" does not merely extend the temporary state, it converts it to permanent for that household.

## Edge Cases

- **The same recipe has two open Support Requests from different households, and one household's report resolves** -- Each Household's exclusion state is tracked independently per FEAT-02.SPEC-009; resolving one household's report has no effect on any other household's exclusion of the same recipe.
- **The recipe was already edited or re-imported (FEAT-10) between the report and its resolution** -- The resolution outcome applies to the recipe's current identity regardless of intervening edits; if released, the edited recipe still must pass FEAT-02.SPEC-002's ordinary safety check again before appearing in a plan, per XBR-19 -- release from this automation lifts only the report-driven exclusion, not the ordinary safety check.
- **The household is deleted before the resolution completes** -- No resolution notice is delivered (there is no household left to notify); the eligibility update is a no-op since a deleted household has no future candidate pool to affect.
- **Concurrent trigger firing (two different households' reports on two different recipes resolve at effectively the same time)** -- Each resolution processes independently against its own Recipe/Household pair; there is no shared state between the two runs.
- **Trigger fires while a previous resolution for the same recipe and household is still in flight** -- This scenario cannot occur under FEAT-22's ownership: a single Support Request can only be resolved once (status transitions from Under review to Resolved exactly one time), so no second resolution trigger for the same Support Request exists to overlap with the first.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22 (Operator Read-Only Support Access) | Triggered by (inbound) | The operator's Resolved transition fires this automation |
| FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | References (inbound) | The prior step this automation's Support Request originated from |
| FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy) | Triggers (outbound) | Releases or permanently excludes the recipe based on the outcome |
| FEAT-02.SPEC-014 (Safety Concern Resolution Notice) | Triggers (outbound) | Tells the household the resolved outcome |
| FEAT-02.SPEC-002 (Candidate Safety Check Execution) | Affects (outbound) | Future checks reflect the released or permanently excluded state |

## Analytics and Success Signals

- **safety_concern_resolved** (outcome: released / kept_excluded) -- supports success-metrics.md: "Zero Allergy Incidents"
- **safety_concern_recipe_released** (recipe reference) -- supports success-metrics.md: "Zero Allergy Incidents"

## Acceptance Criteria

**FEAT-02.SPEC-005-AC-01:** Given Riley resolves a safety-concern Support Request confirming the recipe is safe, when this automation runs, then the recipe's exclusion for that household is lifted and it may pass future candidate checks again.

**FEAT-02.SPEC-005-AC-02:** Given Riley resolves a safety-concern Support Request confirming the recipe is genuinely unsafe, when this automation runs, then the recipe becomes permanently excluded for that household and never re-enters its candidate pool.

**FEAT-02.SPEC-005-AC-03:** Given a Support Request is resolved, when this automation completes, then the household receives the Safety Concern Resolution Notice (FEAT-02.SPEC-014) stating the outcome.

**FEAT-02.SPEC-005-AC-04:** Given a general support Support Request (not a safety concern) is resolved by FEAT-18, when the resolution completes, then this automation does not fire, since it applies only to safety-concern kind requests.

**FEAT-02.SPEC-005-AC-05:** Given a recipe is released back to the candidate pool by this automation, when FEAT-02.SPEC-002 next checks that recipe for the household, then it evaluates the recipe's ingredients normally, since the report-driven exclusion no longer applies.

**FEAT-02.SPEC-005-AC-06:** Given the same recipe has open reports from two different households and one household's report resolves, when this automation runs, then only that household's exclusion state changes; the other household's exclusion is unaffected.

**FEAT-02.SPEC-005-AC-07:** Given a recipe was edited after the report but before resolution, when the resolution releases it, then the released recipe must still pass FEAT-02.SPEC-002's ordinary safety check again before it can appear on any plan.

**FEAT-02.SPEC-005-AC-08:** Given the household is deleted before the resolution completes, when this automation runs, then no resolution notice is delivered and the eligibility update has no effect.

**FEAT-02.SPEC-005-AC-09:** Given two different households' reports on two unrelated recipes resolve at effectively the same time, when both trigger this automation, then each resolution processes independently without affecting the other.

**FEAT-02.SPEC-005-AC-10:** Given the eligibility instruction to FEAT-02.SPEC-009 or the notice trigger to FEAT-02.SPEC-014 cannot be completed, when this automation runs, then the recipe's exclusion state remains exactly as it was before the automation ran, no resolution notice is delivered, and the automation retries both steps together automatically until they succeed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Rule Strength & Blocking Policy

## Overview

**Name:** Rule Strength & Blocking Policy
**ID:** FEAT-02.SPEC-006
**Type:** Logic/Rule
**Purpose:** Defines which rule kinds are hard filters (allergy, religious rule, per-person/shared vegetarian) versus soft, non-blocking preferences (dislikes).
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine
**Governed Entity:** Dietary Rule

## Scope and Non-Goals

**In Scope:**
- Classifying every rule_kind value (allergy, religious rule, per-person vegetarian, dislike) as hard or soft
- Defining how a shared-meal vegetarian option satisfies a per-person vegetarian setting
- Authorization for reading and applying rule-strength classifications across the roles that interact with this engine

**Non-Goals:**
- Creating, editing, or deleting Dietary Rule records -- owned by FEAT-01 (Household Setup & Member Profiles); this spec only classifies rule strength, it does not manage the rule lifecycle
- Running the ingredient-versus-rule comparison itself -- owned by FEAT-02.SPEC-002 (Candidate Safety Check Execution), which reads this spec's classification
- Governing what happens when ingredient data is incomplete -- owned by FEAT-02.SPEC-007 (Ingredient Data Completeness & Fail-Closed Policy)
- Learning new soft dislikes from repeated down-ratings -- owned by FEAT-12 (Meal Rating & Preference Learning), which writes new soft Dietary Rule entries this spec then classifies as soft like any other dislike

## Governed Entity

**Entity:** Dietary Rule
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| member | reference | The Member Profile the rule belongs to |
| rule_kind | enum | allergy, religious rule, per-person vegetarian, or dislike |
| strength | enum | hard (allergy, religious rule) or soft (dislike); vegetarian applies per person with a shared-meal option |
| allergen | text | From the standard allergen list, optionally a named extra ingredient (required for allergies) |
| origin | enum | Entered by the organiser or learned from ratings |
| change_history | derived | Who changed the rule and when |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-02.SPEC-002 | Candidate Safety Check Execution | During ingredient comparison, to determine which rules can exclude a recipe |
| FEAT-02.SPEC-003 | Mid-Week Rule Change Re-Check | To determine whether a changed rule is hard (triggers re-check) or soft (does not) |
| FEAT-01 | Household Setup & Member Profiles (Dietary Rule creation/edit) | On save, to require strength = hard for allergy and religious rule kinds and strength = soft for dislike kind |
| FEAT-03 | AI Weekly Dinner Plan Generation (selection weighting) | To apply soft dislikes as ranking influence rather than exclusion |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| rule_kind | Must be one of: allergy, religious rule, per-person vegetarian, dislike | Always | On save (FEAT-01) | "Choose a rule type." | Yes |
| strength | Must be hard when rule_kind is allergy or religious rule; must be soft when rule_kind is dislike; per-person vegetarian carries no independent strength value (see Cross-Field Rules) | Conditional on rule_kind | On save (FEAT-01) | "Allergies and religious rules are always treated as hard, non-negotiable rules." | Yes |
| allergen | No validation beyond data type in this spec (required-for-allergies validation is owned by FEAT-01) | Always | -- | -- | -- |
| member | No validation beyond data type in this spec -- the referenced Member Profile's existence and validity is owned by FEAT-01 (Household Setup & Member Profiles), which this spec only reads to classify the rule | Always | -- | -- | -- |
| origin | No validation beyond data type in this spec -- whether a rule is organiser-entered or learned from ratings has no bearing on its strength classification; both origins are classified identically by rule_kind | Always | -- | -- | -- |
| change_history | No validation beyond data type in this spec -- this is a derived, system-written audit trail with no user-facing validation; ownership of what it records sits with FEAT-01 | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Strength follows rule_kind | rule_kind, strength | strength is derived from rule_kind, not independently settable: allergy and religious rule are always hard; dislike is always soft; per-person vegetarian is always treated as hard for the purpose of this spec's blocking behavior, with the shared-meal vegetarian_option variant as its satisfaction mechanism (see Business Rules) | N/A -- this is a derived value, not a user-facing validation failure |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Read rule-strength classification (used internally by the safety check) | All roles that can view a Planned Meal or Recipe (Maya, Sam, older-kid limited login) | Always -- classification is not itself sensitive data, only its consequence (the badge or exclusion reason) is user-visible | -- |
| Apply hard-rule exclusion during a candidate check | System (via FEAT-02.SPEC-002) | Always, on every candidate | -- |
| Override or bypass a hard-rule exclusion | No role, ever | Never -- allergies and religious rules cannot be overridden by any user action, per the Brief's Constraints: Safety | The recipe is excluded from the candidate pool entirely; there is no "show anyway" or override control on any screen |
| Change a rule's strength classification directly | No role -- strength is derived from rule_kind, not independently editable | Never | The Household Setup rule-entry form (FEAT-01) exposes no strength control; strength is set automatically from the chosen rule_kind |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| strength | allergy -> hard; religious rule -> hard; per-person vegetarian -> hard (with shared-meal variant satisfaction); dislike -> soft | On create and on any edit of rule_kind | No -- derived automatically from rule_kind, never independently set |

## Business Rules

- Allergies and religious rules are hard filters: a candidate recipe that fails either for any household member is excluded from the household's candidate pool entirely, per FEAT-02.SPEC-002.
- Per-person vegetarian settings apply per household member and are treated as hard for that member, but a shared dinner recipe is not excluded on vegetarian grounds alone if it carries a vegetarian_option variant that satisfies the vegetarian member -- this is the shared-meal vegetarian accommodation the Brief names explicitly.
- Dislikes are soft: they never block a suggestion. They influence selection ranking in AI Weekly Dinner Plan Generation (FEAT-03) and are the target of learned entries from Meal Rating & Preference Learning (FEAT-12, XBR-17), but FEAT-02.SPEC-002's safety check never excludes a recipe for a dislike alone.
- XBR-17: A meal rated down repeatedly by the same member becomes a learned soft dislike on that member's dietary rules; soft dislikes influence selection but never block a suggestion and never override an explicit rule -- this spec is the classification XBR-17's learned entries are subject to, same as any organiser-entered dislike.
- A learned dislike (origin = learned from ratings) never overwrites or weakens an explicit hard rule for the same ingredient; an explicit organiser edit always takes precedence for the same ingredient, per the dependency map's Contention note for Dietary Rule.

## Edge Cases

- **A member has both an explicit dislike and a hard allergy naming the same ingredient** -- The hard allergy classification governs: the recipe is excluded as a hard-rule failure, and the separate soft dislike entry has no additional effect since the recipe never reaches the ranking stage where dislikes matter.
- **A shared dinner has no vegetarian_option variant and one member is vegetarian** -- The recipe is excluded as a hard-rule failure for that member, since no variant exists to satisfy the vegetarian setting; this is the same outcome as any other hard-rule violation.
- **A rule_kind is per-person vegetarian for one member but the household has no other dietary restrictions** -- The recipe still must either be inherently vegetarian or carry a satisfying vegetarian_option variant; the absence of other restrictions does not loosen the vegetarian rule's hard treatment.
- **A learned soft dislike is added for an ingredient the member is also allergic to** -- FEAT-12 never creates a dislike entry that duplicates or conflicts with an existing hard allergy for the same allergen, per the Contention note; if it were ever attempted, the existing hard allergy classification takes precedence and the duplicate soft entry has no effect on the safety check.
- **Strength field is present with a value inconsistent with rule_kind (a data anomaly, e.g., an allergy recorded as soft)** -- This spec's derivation rule means strength is never independently set, so this state cannot arise through the product's own save path; FEAT-02.SPEC-002 always treats allergy and religious-rule kinds as hard regardless of any stored strength value, since the classification in this spec -- not the raw field -- is authoritative.

## Acceptance Criteria

**FEAT-02.SPEC-006-AC-01:** Given a household member has a peanut allergy recorded, when a candidate recipe containing peanuts is checked, then the recipe is excluded from the candidate pool as a hard-rule failure.

**FEAT-02.SPEC-006-AC-02:** Given a household member has a halal religious rule recorded, when a candidate recipe containing a non-halal ingredient is checked, then the recipe is excluded from the candidate pool as a hard-rule failure.

**FEAT-02.SPEC-006-AC-03:** Given a household member has a recorded dislike of mushrooms, when a candidate recipe containing mushrooms is checked, then the recipe is not excluded, since dislikes are soft and non-blocking.

**FEAT-02.SPEC-006-AC-04:** Given a household member has a per-person vegetarian setting and a shared dinner recipe is not inherently vegetarian but carries a vegetarian_option variant, when the recipe is checked, then the recipe passes and the variant is made available rather than the recipe being excluded.

**FEAT-02.SPEC-006-AC-05:** Given a household member has a per-person vegetarian setting and a shared dinner recipe is not inherently vegetarian and carries no vegetarian_option variant, when the recipe is checked, then the recipe is excluded as a hard-rule failure.

**FEAT-02.SPEC-006-AC-06:** Given Maya is entering a new Dietary Rule and selects rule_kind "allergy", when she saves it, then its strength is automatically set to hard with no independent strength control shown.

**FEAT-02.SPEC-006-AC-07:** Given Maya is entering a new Dietary Rule and selects rule_kind "dislike", when she saves it, then its strength is automatically set to soft.

**FEAT-02.SPEC-006-AC-08:** Given a meal is rated down repeatedly by Sam (FEAT-12, XBR-17), when the learned dislike is created, then this spec classifies it as soft, and it influences future plan selection without ever blocking a suggestion.

**FEAT-02.SPEC-006-AC-09:** Given a household member has both a hard allergy and a soft dislike naming the same ingredient, when a candidate recipe containing that ingredient is checked, then it is excluded as a hard-rule failure, and the soft dislike entry has no additional bearing on the outcome.

**FEAT-02.SPEC-006-AC-10:** Given no user role or screen offers an "override" or "show anyway" control on an excluded recipe, when any adult views the ineligible recipe, then no path exists to bypass the hard-rule exclusion.

**FEAT-02.SPEC-006-AC-11:** Given an explicit organiser-entered dietary rule and a learned soft dislike exist for the same ingredient, when FEAT-12 attempts to record the learned entry, then the explicit rule's classification and content take precedence and are never weakened by the learned entry, per the dependency map's Contention note.

**FEAT-02.SPEC-006-AC-12:** Given a household member's vegetarian setting is per-person and only one member holds it, when a candidate recipe is checked for the whole household, then only that member's vegetarian requirement is evaluated against the recipe -- other members' lack of a vegetarian setting has no bearing.

**FEAT-02.SPEC-006-AC-13:** Given Maya is entering a new Dietary Rule and selects rule_kind "per-person vegetarian", when she saves it, then the rule is treated as hard for exclusion purposes, subject to the shared-meal vegetarian_option accommodation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Ingredient Data Completeness & Fail-Closed Policy

## Overview

**Name:** Ingredient Data Completeness & Fail-Closed Policy
**ID:** FEAT-02.SPEC-007
**Type:** Logic/Rule
**Purpose:** Requires complete ingredient data with no partial matching shortcuts; a recipe with incomplete data is excluded rather than assumed safe.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine
**Governed Entity:** Recipe

## Scope and Non-Goals

**In Scope:**
- Defining what "complete" ingredient data means for the purpose of the safety check
- The fail-closed exclusion outcome when a recipe's ingredient data is incomplete
- Prohibiting fuzzy or partial ingredient matching as a substitute for complete data

**Non-Goals:**
- Running the ingredient-versus-rule comparison itself once data is confirmed complete -- owned by FEAT-02.SPEC-002 (Candidate Safety Check Execution)
- Authoring or editing recipe ingredient content -- owned by FEAT-08 (Recipe Library) for starter content and FEAT-10 (Recipe Import from Web Link) for imported content; this spec only validates what those features produce
- Fuzzy or partial ingredient matching as a feature -- excluded outright per the feature's own Non-Goals: any tolerance for a near-miss match directly contradicts the brief's zero-incident success criterion
- Ingredient-quantity accuracy for cost or grocery-list purposes -- owned by FEAT-06 (Shared Grocery List); this spec only cares whether quantity and unit are present, not whether they are accurate for shopping

## Governed Entity

**Entity:** Recipe
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| ingredients | list of {name, quantity, unit} | Each ingredient with quantity and unit; complete ingredient data is required to pass the safety check (at least one ingredient to save an import) |
| dietary_badges | derived | Computed per household by FEAT-02 when viewed |
| origin | enum | Starter library or imported (with source link) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-02.SPEC-002 | Candidate Safety Check Execution | Step 4 of its processing logic: completeness is verified before any ingredient-versus-rule comparison runs |
| FEAT-08 | Recipe Library (Starter Recipes) | On starter content seeding -- starter recipes are expected to be complete at ingestion, per ASMP-34 |
| FEAT-10 | Recipe Import from Web Link | On import save -- an imported recipe with incomplete extracted ingredient data is flagged for the household to complete before it can pass this check |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| ingredients | Every listed ingredient must have a non-empty name, a quantity, and a unit for the recipe to be considered complete | Always, at safety-check time | On every candidate safety check (FEAT-02.SPEC-002) | "This recipe's ingredient list is incomplete, so it can't be checked for safety yet." (shown as the plain ineligibility reason, per FEAT-02.SPEC-008) | Yes |
| ingredients | Must contain at least one ingredient | Always | On every candidate safety check | Same as above -- an empty ingredient list is treated identically to an incomplete one | Yes |
| dietary_badges | No validation beyond data type in this spec -- badges are a derived, system-computed field (recomputed per household by FEAT-02.SPEC-008) and carry no independent completeness requirement of their own; badge accuracy is only as good as this spec's ingredients determination that feeds it | Always | -- | -- | -- |
| origin | No validation beyond data type in this spec -- whether a recipe is starter-library or imported content has no bearing on the completeness policy; both origins are held to the identical completeness bar defined above | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| No cross-field rules beyond per-ingredient completeness | ingredients | Completeness is evaluated per ingredient entry independently; there is no rule combining ingredients or other Recipe fields for this policy | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Determine whether a recipe's ingredient data is complete | System (via FEAT-02.SPEC-002, FEAT-08, FEAT-10) | Always, on every check or ingestion | -- |
| Override an incomplete-data exclusion to force a recipe through the safety check | No role, ever | Never | No screen offers a "check anyway" or "trust this recipe" control; the recipe simply does not appear as a candidate until its data is completed |
| Complete or correct a recipe's ingredient data | Maya and Sam (Recipe Library: Full), for imported recipes only, per FEAT-10 | Only for recipes the household owns (imported recipes); starter recipes are read-only for households and are corrected only by the recipe/food-content data capability (FEAT-08) | Households have no edit control on starter recipes; an attempt to edit one shows "Starter recipes can't be edited. Import your own version instead." |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| completeness determination | Derived: complete when every ingredient has name, quantity, and unit and at least one ingredient exists; incomplete otherwise | Evaluated fresh on every safety check (not cached), since an edit (FEAT-10) can change completeness between checks | No -- this is a computed determination, not a stored, user-editable field |

## Business Rules

- A recipe with incomplete ingredient data is excluded rather than assumed safe -- this is the fail-closed guarantee: an unverifiable check must never default to "safe."
- No partial matching shortcut narrows what counts as complete -- a recipe cannot pass with some ingredients verified and others assumed, per the feature's Validation & Limits ("no partial matching shortcuts").
- Fuzzy or approximate ingredient matching is never used as a substitute for missing quantity/unit data -- this policy requires exact, complete data or exclusion, with no middle ground.
- XBR-19: An edited imported recipe must pass the safety check again before it can appear in any plan -- if an edit removes or corrupts ingredient completeness, the recipe fails this policy and is excluded until corrected.
- ASMP-34 (Recipe/food-content data): FEAT-08 owns starter-content ingestion and is expected to seed complete data; this spec is the validation gate that catches any starter-content gap rather than assuming vendor-supplied data is automatically trustworthy.

## Edge Cases

- **Recipe has an ingredient with a quantity but no unit (e.g., "2" with no "cups" or "grams")** -- Treated as incomplete; the recipe is excluded exactly as if the ingredient were entirely missing.
- **Recipe has an ingredient name with a quantity/unit but the name is a placeholder or empty string** -- Treated as incomplete; a non-empty name is required for every listed ingredient.
- **Starter recipe is seeded with a data gap** -- Treated identically to a household-imported recipe with incomplete data: excluded from every household's candidate pool until FEAT-08 corrects the seeded content, since starter content receives no special trust exemption from this policy.
- **An imported recipe passes the check, is later edited to add an ingredient with no unit, and is then re-checked** -- The recipe fails the completeness check on the next safety check (per XBR-19) and is excluded until the household corrects the new ingredient's unit.
- **A recipe's ingredient list is complete but extremely long** -- Length has no bearing on completeness; every ingredient, regardless of count, must individually satisfy name/quantity/unit presence.

## Acceptance Criteria

**FEAT-02.SPEC-007-AC-01:** Given a candidate recipe has every ingredient with a name, quantity, and unit, when the safety check runs, then the recipe is treated as complete and proceeds to the ingredient-versus-rule comparison.

**FEAT-02.SPEC-007-AC-02:** Given a candidate recipe has one ingredient missing its unit, when the safety check runs, then the recipe is excluded and the plain reason states the ingredient list is incomplete.

**FEAT-02.SPEC-007-AC-03:** Given a candidate recipe has zero ingredients listed, when the safety check runs, then the recipe is excluded under the same incomplete-data policy.

**FEAT-02.SPEC-007-AC-04:** Given a recipe's ingredient data cannot be fully verified for any reason, when the safety check runs, then the recipe is excluded rather than shown as safe by default.

**FEAT-02.SPEC-007-AC-05:** Given a starter recipe is seeded with an incomplete ingredient entry, when any household's candidate check considers it, then it is excluded the same as an incomplete household-imported recipe.

**FEAT-02.SPEC-007-AC-06:** Given Sam attempts to edit a starter recipe's ingredient list, when he looks for an edit control, then none is available and an attempt shows "Starter recipes can't be edited. Import your own version instead."

**FEAT-02.SPEC-007-AC-07:** Given Maya edits her household's imported recipe to add a new ingredient with a name and quantity but no unit, when the recipe is next considered as a candidate, then it is excluded until the unit is added, per XBR-19.

**FEAT-02.SPEC-007-AC-08:** Given a household member views an excluded recipe with incomplete data, when they look for a way to proceed anyway, then no override control exists on any screen.

**FEAT-02.SPEC-007-AC-09:** Given a recipe's ingredient data was incomplete at one check and is later corrected by the household (for an imported recipe), when the next safety check runs, then completeness is re-evaluated fresh and the recipe may pass if its ingredients now satisfy the rule comparison.

**FEAT-02.SPEC-007-AC-10:** Given a recipe has a very long ingredient list where every entry is individually complete, when the safety check runs, then the recipe is treated as complete regardless of ingredient count.

**FEAT-02.SPEC-007-AC-11:** Given no fuzzy-matching mechanism exists anywhere in the safety check, when a recipe's ingredient name is ambiguous or a near-miss to an allergen term, then the check either matches it fully (per FEAT-02.SPEC-002's compound-ingredient handling) or treats the data as incomplete -- it never partially matches to assume safety.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



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



# Logic/Rule Spec: Safety Concern Eligibility & Re-offer Policy

## Overview

**Name:** Safety Concern Eligibility & Re-offer Policy
**ID:** FEAT-02.SPEC-009
**Type:** Logic/Rule
**Purpose:** Keeps a recipe under an open safety report out of the household's candidate pool until the report resolves.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine
**Governed Entity:** Recipe (exclusion state, scoped per Household)

## Scope and Non-Goals

**In Scope:**
- The exclusion state of a recipe under an open safety-concern Support Request, per household
- Transitioning that state when a report is resolved: released back to the pool, or kept permanently excluded
- Authorization for who can change this exclusion state

**Non-Goals:**
- Creating the Support Request or performing the initial removal -- owned by FEAT-02.SPEC-004 (Safety Concern Intake & Removal), which instructs this policy
- Applying the operator's review outcome -- owned by FEAT-02.SPEC-005 (Safety Concern Resolution Outcome), which instructs this policy's release/keep-excluded transition
- The ordinary hard-rule safety check for a recipe not under any report -- owned by FEAT-02.SPEC-002 and FEAT-02.SPEC-006/007, which this policy's exclusion sits in front of, not in place of
- Cross-household exclusion -- excluded per the feature's own design: an open report against a recipe for one household never affects another household's candidate pool for the same recipe, since each household's Dietary Rule and report history are independent

## Governed Entity

**Entity:** Recipe (with its exclusion state tracked against the Support Request and Household that opened it)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| ingredients | list | Read by FEAT-02.SPEC-002, not altered by this policy |
| dietary_badges | derived | Reflects the exclusion state this policy governs when computed per household |
| (exclusion state, cross-referenced) Support Request.status | enum | Raised, Under review, Resolved -- the driving state for this policy's exclusion window |
| (exclusion state, cross-referenced) Support Request.planned_meal/recipe | reference | Identifies which Recipe this policy's exclusion applies to, and for which Household |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-02.SPEC-002 | Candidate Safety Check Execution | Step 2 of its processing logic -- consulted before any ingredient comparison runs |
| FEAT-02.SPEC-004 | Safety Concern Intake & Removal | Instructs this policy to begin the temporary exclusion when a report is created |
| FEAT-02.SPEC-005 | Safety Concern Resolution Outcome | Instructs this policy to release or permanently exclude the recipe when a report resolves |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| exclusion state | No validation beyond data type -- this is a derived state, not user-entered data | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Exclusion tracks the open Support Request | Support Request.status, Recipe exclusion state | A recipe is excluded for a household exactly while that household has a Support Request (kind = safety concern, referencing that recipe) with status Raised or Under review; the exclusion ends the moment status becomes Resolved, at which point FEAT-02.SPEC-005's outcome determines whether it becomes a standing permanent exclusion or is fully released | N/A -- this is a state-tracking rule, not a user-facing validation |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Trigger the start of an exclusion window | System (via FEAT-02.SPEC-004), on report creation | Always, on every new safety-concern Support Request | -- |
| Release a recipe from exclusion | System (via FEAT-02.SPEC-005), on a Resolved outcome confirming the recipe is safe | Only after the operator's resolution -- no earlier release path exists | -- |
| Convert a temporary exclusion to a permanent one | System (via FEAT-02.SPEC-005), on a Resolved outcome confirming the recipe is unsafe | Only after the operator's resolution | -- |
| Manually override or end an exclusion before resolution | No role, ever | Never -- neither Maya, Sam, nor Riley can lift an exclusion outside the FEAT-22 resolution workflow | No screen offers a "restore this recipe" control while a report is open; the recipe remains absent from every candidate-pool view for that household until FEAT-22 resolves the report |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| exclusion state | Derived from the most recent Support Request status for the Recipe/Household pair: Raised or Under review -> excluded (temporary); Resolved with "safe" outcome -> not excluded; Resolved with "unsafe" outcome -> excluded (permanent) | Re-evaluated on every candidate safety check (FEAT-02.SPEC-002) and whenever a Support Request transitions | No -- fully system-derived |

## Business Rules

- A recipe under an open safety report is never re-offered to that household until the report resolves -- this is the Brief's own Validation & Limits statement, and the single source of truth every other spec in this feature defers to rather than tracking its own exclusion state.
- The exclusion is scoped per household -- an open report from one household never excludes the recipe for any other household.
- A permanent exclusion (from a "confirmed unsafe" resolution) is distinct from the temporary open-report exclusion: it does not expire or require renewal, and no automated process re-admits the recipe to that household's pool afterward.
- FEAT-02.SPEC-002 and FEAT-02.SPEC-004 both defer to this policy rather than each keeping their own exclusion state, per the Brief's Shared Validation section.

## Edge Cases

- **A household's Support Request for a recipe is Resolved, then the same household reports the same recipe again later** -- If the prior resolution was "confirmed unsafe," the recipe is already permanently excluded and the new report is still recorded (per FEAT-02.SPEC-004's duplicate-safe handling) but has no additional exclusion effect, since the recipe was already excluded. If the prior resolution was "confirmed safe," the new report opens a fresh temporary exclusion exactly as any first report would.
- **Two households report the same recipe independently** -- Each Household's exclusion state is tracked and resolved entirely independently; one household's permanent exclusion has no bearing on the other household's candidate pool.
- **A report is created for a recipe that is also excluded for unrelated reasons (e.g., incomplete ingredient data)** -- Both exclusions apply; FEAT-02.SPEC-002 checks this policy first (Step 2) and, finding the report-driven exclusion, stops there without needing to also evaluate the ingredient-completeness policy -- the recipe is excluded either way.
- **The household is deleted while a report is open** -- The exclusion state becomes moot with no household left to exclude the recipe for; no orphaned exclusion record persists in a way that could affect a future household, since exclusion is always keyed to a specific Household.

## Acceptance Criteria

**FEAT-02.SPEC-009-AC-01:** Given a safety-concern Support Request is created for a recipe (FEAT-02.SPEC-004), when any future candidate check runs for that household, then the recipe is excluded per this policy without re-running the ingredient comparison.

**FEAT-02.SPEC-009-AC-02:** Given a household's Support Request for a recipe is resolved with a "confirmed safe" outcome, when the next candidate check runs, then the recipe is no longer excluded under this policy and is evaluated normally by FEAT-02.SPEC-002.

**FEAT-02.SPEC-009-AC-03:** Given a household's Support Request for a recipe is resolved with a "confirmed unsafe" outcome, when any future candidate check runs, then the recipe remains permanently excluded for that household.

**FEAT-02.SPEC-009-AC-04:** Given one household has an open report on a recipe, when a different household considers the same recipe as a candidate, then that other household's check is unaffected by the first household's exclusion.

**FEAT-02.SPEC-009-AC-05:** Given a recipe was previously confirmed unsafe for a household and is reported again by the same household, when the new report is processed, then the recipe remains permanently excluded with no additional exclusion effect from the new report.

**FEAT-02.SPEC-009-AC-06:** Given a recipe was previously confirmed safe for a household and is reported again by the same household, when the new report is created, then a fresh temporary exclusion begins exactly as it would for a first-time report.

**FEAT-02.SPEC-009-AC-07:** Given no role has a control to manually lift an exclusion, when Maya looks for a way to restore an excluded recipe before the report resolves, then no such control exists anywhere in the product.

**FEAT-02.SPEC-009-AC-08:** Given a recipe under an open report also has incomplete ingredient data, when the candidate check runs, then the report-driven exclusion from this policy applies first and the check stops there.

**FEAT-02.SPEC-009-AC-09:** Given a household is deleted while a report on one of its recipes is open, when the deletion completes, then the exclusion state for that household is moot and has no effect on any other household.

**FEAT-02.SPEC-009-AC-10:** Given a Support Request's status is Raised or Under review, when this policy is consulted, then the recipe is treated as excluded for that household.

**FEAT-02.SPEC-009-AC-11:** Given a Support Request's status becomes Resolved, when this policy is next consulted, then the exclusion state reflects the recorded outcome (released or permanently excluded) rather than the prior temporary state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |



# Integration Spec: Transactional Email Delivery (Safety Reports)

## Overview

**Name:** Transactional Email Delivery (Safety Reports)
**ID:** FEAT-02.SPEC-010
**Type:** Integration
**Purpose:** Delivers the operator-facing safety-concern alert and the household's resolution notice through the product's transactional email capability.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- Sending the safety-concern report email to the operator (Riley) for review
- Sending the household's resolution notice by email where email is that notification's delivery channel
- User-facing behavior when the transactional email capability is slow, unavailable, or rejects a send for either of these two safety-report emails
- Disclosure of what data these two emails carry to the email capability

**Non-Goals:**
- Choosing the email vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate.
- Any other transactional email the product sends (account & recovery, plan-ready fallback, billing confirmations, export/deletion/support-acknowledgement) -- those are owned by FEAT-01.SPEC-017, FEAT-07.SPEC-006, FEAT-14.SPEC-012, and FEAT-18.SPEC-012 respectively, per the Feature Dependency Map's External Touchpoints table; this spec covers only the two safety-report emails FEAT-02 originates.
- Composing the exact subject/body content of the two emails -- owned by FEAT-02.SPEC-013 (Safety Concern Operator Alert) and FEAT-02.SPEC-014 (Safety Concern Resolution Notice); this spec defines only the delivery contract those notifications rely on.
- In-app or push delivery of these notices -- FEAT-02.SPEC-011 through FEAT-02.SPEC-014 define their own channel mixes; this spec covers the email channel specifically.

## Capability Category

**Category:** Transactional email
**Dependency Source:** ASMP-32 -- "Requires a transactional email capability" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Transactional email (ASMP-32)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-01, FEAT-02, FEAT-07, FEAT-14, FEAT-18; this spec is the FEAT-02 entry: "FEAT-02.SPEC-010 (safety reports)")
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Riley receives the safety-concern report by email so it can be reviewed without being signed into the product's own screens | Report a safety concern | FEAT-02.SPEC-013 (Safety Concern Operator Alert) |
| Maya and Sam receive the household's resolution outcome by email if they are not reachable in-app at the moment it resolves | Report a safety concern (resolution notice) | FEAT-02.SPEC-014 (Safety Concern Resolution Notice) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Reported recipe name and ingredient list | Recipe -- name, ingredients | A safety-concern Support Request is created | The operator needs the recipe's content to review the reported concern |
| Reporter's note (if provided) | Support Request -- note | A safety-concern Support Request is created | The operator needs the reporter's own description of the concern |
| Reporting member's role and first name | Member Profile -- display_name, member_type | A safety-concern Support Request is created | The operator needs to know who raised the concern and in what capacity (organiser or other adult) |
| Household reference and reported member's age band (kid profile only, when the reported concern involves a kid's allergy) | Household -- household_name; Member Profile -- age_band | A safety-concern Support Request is created | The operator needs enough context to locate the household and understand which member's allergy is at issue; per the Access Matrix notes, this is the one context in which the operator sees a kid's allergy detail |
| Resolution outcome (recipe confirmed safe / kept excluded) | Support Request -- status, resolution outcome | A safety-concern Support Request is resolved | The household needs to know the outcome of its report |
| Recipient's first name and email | Member Profile -- display_name, sign_in (email) | Either email is sent | The email capability must know where and to whom to address the message |

No payment details, no Weekly Plan or Grocery List content beyond the single reported recipe, and no other household member's Dietary Rule data ever leaves the product through this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Delivery outcome (delivered / bounced / failed) | The email capability reports the result of a send attempt | No entity field is updated by this outcome alone -- it drives only this spec's own retry logic (see Delivery Rules in FEAT-02.SPEC-013 and FEAT-02.SPEC-014); Support Request status is never derived from email delivery outcome |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Send succeeded | The email capability confirms the operator alert or resolution notice was accepted for delivery | None | No user-facing confirmation is shown for a successful transactional send -- the household and operator experience is simply "the email arrives" | FEAT-02.SPEC-013, FEAT-02.SPEC-014 |
| Send failed (transient) | The email capability reports a temporary failure (e.g., a momentary outage) | None -- retry is scheduled per FEAT-02.SPEC-013's and FEAT-02.SPEC-014's own Delivery Rules | No user feedback during retry -- the in-app safety-concern flow (FEAT-02.SPEC-001, FEAT-02.SPEC-004) already confirmed the meal's removal independent of email delivery | FEAT-02.SPEC-013, FEAT-02.SPEC-014 |
| Send failed (permanent, e.g., invalid recipient address) | The email capability reports the address cannot receive mail | None to Support Request; the failure is recorded for operational visibility only | No household- or operator-facing feedback -- since the operator alert has no fallback recipient, a permanent failure here is treated as an operational incident (see Edge Cases), not a user-facing message | FEAT-02.SPEC-013, FEAT-02.SPEC-014 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-02.SPEC-001 (Report a Safety Concern) | N/A -- this screen's own submission (meal removal, Support Request creation) does not wait on email delivery; the confirmation shown to the reporter is independent of this integration's state. | N/A -- same as above; the report is recorded and the meal removed regardless of the email capability's availability. | N/A -- the screen sends no request to this capability directly; FEAT-02.SPEC-004 triggers the email asynchronously after the screen's own confirmation. |
| FEAT-22.SPEC-001 (Support Request Queue, FEAT-22) | The operator alert email may arrive later than usual; the underlying Support Request is visible to Riley in the Support View immediately regardless of email delivery timing. | The operator alert email does not arrive; Riley can still discover and review the open Support Request directly through the Support View, which does not depend on email. | N/A -- a rejection from the capability (e.g., malformed address, which cannot occur for a fixed operator address) is treated as a down-equivalent operational condition; the Support View remains the reliable path. |

## Consent and Disclosure

- **Safety-report data shared with the email capability** -- The household is not shown a separate consent prompt before this data is shared, since sending the operator alert is an inseparable part of submitting a safety concern (FEAT-02.SPEC-001's own confirmation states "your household's operator will review the ingredients," which discloses that the report -- including the recipe and any note -- reaches the operator). No further opt-out exists for this specific email, since it is core to the safety-review process the report itself initiates.
- **Resolution notice recipient disclosure** -- The household is told, as part of FEAT-02.SPEC-001's confirmation, that it will be told the outcome once reviewed; the resolution notice's email channel is one of its stated delivery channels (FEAT-02.SPEC-014), not a separately disclosed sharing event.
- **What is never shared** -- Payment details, the full Weekly Plan, the Grocery List, and any other household member's Dietary Rule data beyond the one member whose allergy is at issue in the specific report never leave the product through this integration.

## Edge Cases

- **The operator alert email fails permanently (e.g., the operator's configured address is temporarily unreachable)** -- Since Riley discovers open Support Requests directly through the Support View (FEAT-22) rather than solely through email, a permanent failure here does not block the review process; it is logged as an operational condition for the founder to notice, not surfaced to the household.
- **The same resolution event is delivered to the email capability twice due to a retry** -- FEAT-02.SPEC-014's deduplication rule (at most one resolution notice per resolved Support Request) prevents a duplicate email from reaching the household even if this integration's send call is retried.
- **An email event arrives for a Support Request that has since been superseded (e.g., resolved twice due to a data anomaly)** -- The email reflects the Support Request's status at the moment the send was triggered; this integration does not re-fetch status at delivery time, so a stale send is possible only if the underlying automation (FEAT-02.SPEC-005) itself fired twice, which its own concurrency handling prevents.
- **Capability goes down mid-send for the resolution notice** -- The household still has the resolution outcome visible in-app (FEAT-02.SPEC-014's other channel, if any) and via the Support Request's status; the email is retried per FEAT-02.SPEC-014's Delivery Rules once the capability recovers.
- **Operator alert and resolution notice for the same Support Request are queued for delivery at effectively the same time (a report resolved unusually quickly)** -- Each is a distinct message to a distinct recipient (operator vs. household) and is sent independently; no batching or merging occurs between the two, since they serve different audiences.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-013 (Safety Concern Operator Alert) | Triggered by (inbound) | The operator alert's email channel is delivered through this integration |
| FEAT-02.SPEC-014 (Safety Concern Resolution Notice) | Triggered by (inbound) | The resolution notice's email channel is delivered through this integration |
| FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Triggers (outbound, indirect) | Report creation is the originating event that leads to the operator alert send |
| FEAT-02.SPEC-005 (Safety Concern Resolution Outcome) | Triggers (outbound, indirect) | Resolution is the originating event that leads to the resolution notice send |
| FEAT-22 (Operator Read-Only Support Access) | Affects (outbound) | The reliable fallback path when the operator alert email is degraded |

## Analytics and Success Signals

- **safety_report_email_sent** (recipient: operator / household; email: operator_alert / resolution_notice) -- N/A -- no Stage 2 metric measures safety-report email delivery specifically; retained so the reliability of this delivery path is observable rather than invisible, given the trust-critical nature of the feature it supports.
- **safety_report_email_failed** (recipient, failure type: transient / permanent) -- supports success-metrics.md: "Zero Allergy Incidents" (a household or operator who never receives a safety-report communication is a gap in the zero-incident trust promise, so failures here are tracked against that metric).

## Acceptance Criteria

**FEAT-02.SPEC-010-AC-01:** Given a safety-concern Support Request is created, when this integration sends the operator alert, then the recipe name, ingredients, reporter's role and first name, and any note are included in the email sent to the operator.

**FEAT-02.SPEC-010-AC-02:** Given a safety-concern Support Request is resolved, when this integration sends the resolution notice, then the household's recipient receives an email stating the outcome.

**FEAT-02.SPEC-010-AC-03:** Given the transactional email capability is temporarily unavailable when a safety concern is reported, when FEAT-02.SPEC-001's submission completes, then the meal is still removed and the reporter still sees the confirmation, independent of the email capability's state.

**FEAT-02.SPEC-010-AC-04:** Given the operator alert email fails permanently, when Riley checks for open Support Requests, then the Support View (FEAT-22) still shows the report, since it does not depend on email delivery.

**FEAT-02.SPEC-010-AC-05:** Given a resolution-notice send is retried after a transient failure, when the retry succeeds, then only one resolution notice reaches the household, per FEAT-02.SPEC-014's deduplication rule.

**FEAT-02.SPEC-010-AC-06:** Given a household submits a safety concern, when they read FEAT-02.SPEC-001's confirmation, then it discloses that the operator will review the ingredients, which is the disclosure covering this integration's operator-alert data share.

**FEAT-02.SPEC-010-AC-07:** Given a reported concern involves a kid profile's allergy, when the operator alert is composed, then it includes only that kid's age band and the allergy detail relevant to the report -- never the kid's other profile data.

**FEAT-02.SPEC-010-AC-08:** Given no payment or full Weekly Plan data is ever part of a safety-report email, when either email is composed, then it contains only the recipe, note, reporter/recipient identity, and resolution outcome as defined in Data Exchanged.

**FEAT-02.SPEC-010-AC-09:** Given an operator alert and a resolution notice for the same Support Request become due at effectively the same time, when both are sent, then each is delivered independently to its own recipient with no merging.

**FEAT-02.SPEC-010-AC-10:** Given the email capability reports a permanent failure for a resolution-notice send, when the failure is recorded, then no household-facing error message appears, since the outcome remains visible through the product's own Support Request status.

**FEAT-02.SPEC-010-AC-11:** Given the email capability is down when a safety concern is reported, when it later recovers, then the queued operator alert is delivered per FEAT-02.SPEC-013's retry rules without being lost.

**FEAT-02.SPEC-010-AC-12:** Given the same resolution event triggers a duplicate send attempt due to a retry, when the second attempt is processed, then it changes nothing the household sees and no second email is delivered.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 2 | 2 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 6 (2 screens x 3 conditions, 3 N/A cells justified and excluded from the count of active paths but listed) | 6 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 5 | 5 |



# Notification Spec: Safety Concern Reporter Acknowledgement

## Overview

**Name:** Safety Concern Reporter Acknowledgement
**ID:** FEAT-02.SPEC-011
**Type:** Notification
**Purpose:** Confirms to the reporting adult that their safety concern was received and the meal has been removed.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- The in-app acknowledgement shown to the reporting adult the moment their report is processed
- Its content, delivery, and preference behavior

**Non-Goals:**
- Telling the organiser when she is not the reporter -- owned by FEAT-02.SPEC-012 (Safety Concern Organiser Alert), a distinct recipient and message
- Telling the operator -- owned by FEAT-02.SPEC-013 (Safety Concern Operator Alert)
- Telling the household the resolution outcome once reviewed -- owned by FEAT-02.SPEC-014 (Safety Concern Resolution Notice), a later, distinct communication
- Removing the meal or creating the Support Request -- owned by FEAT-02.SPEC-004 (Safety Concern Intake & Removal), whose outcome this notification confirms

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when the report is processed | The reporter is already inside the product, on the screen where they just submitted the report (FEAT-02.SPEC-001); the acknowledgement completes that same interaction rather than requiring them to check elsewhere |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Safety concern report processed | FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Fires immediately after the reported meal is removed and the Support Request is created (or the duplicate-safe path is taken) | Reporting Member, reported Recipe name, reported meal's night |

## Audience and Preferences

**Recipients:** Maya (Organiser) or Sam (Other Adult Member) -- whichever adult submitted the report, per the Access Matrix's Safety Reports column (Maya: Full, Sam: Own-only). The acknowledgement is delivered only to the reporting member themselves, never to the other adult.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this acknowledgement has no independent preference control | -- | Always on | -- |

**Quiet Hours:** N/A -- this is an in-session, in-app confirmation delivered as the direct result of the reporter's own action; quiet hours govern notifications that interrupt the recipient away from an active task, and this one occurs within the same interaction the reporter just initiated.

## Content Definition

**In-app:**
- **Title:** Report received
- **Body:** {recipe_name} has been removed from your plan. Your household's operator will review it, and we'll let you know the outcome.
- **CTA:** Show me alternatives -- deep-links to FEAT-04.SPEC-001 (Meal Swap, safe alternatives list) for the emptied slot on {night}

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {recipe_name} | Recipe -- name | Thursday's Mushroom Risotto | Never empty -- name is required at recipe creation (FEAT-08, FEAT-10) |
| {night} | Planned Meal -- night | Thursday | Never empty -- night is required on every Planned Meal |

## Delivery Rules

**Batching:** None -- each report produces exactly one acknowledgement to its own reporter; two separate reports (even from the same reporter on the same day) each produce their own acknowledgement, since each report is submitted as a distinct interaction the reporter is actively completing.
**Deduplication:** At most one acknowledgement per report submission. FEAT-02.SPEC-004's duplicate-report handling (an existing open report on the same recipe) does not suppress this acknowledgement for a genuinely new reporter -- each reporting adult receives their own acknowledgement for their own submission, even if the recipe was already reported by someone else.
**Retry on failure:** N/A -- this is rendered directly within the same screen interaction (FEAT-02.SPEC-001) that produced it; there is no separate delivery channel to retry, since it is not a message sent elsewhere but the screen's own state change.
**Expiry:** N/A -- the acknowledgement is shown once, within the dialog, immediately upon successful submission; it does not wait to be delivered and cannot become stale.

## Edge Cases

- **The reporting adult closes the dialog before reading the acknowledgement fully** -- No re-delivery occurs; the meal's removal is already reflected on the calling screen (the emptied slot), so the reporter can confirm the outcome there even if the dialog's acknowledgement was dismissed quickly.
- **The report is a duplicate against an already-open concern raised by the other adult** -- The current reporter still receives their own acknowledgement worded identically, since from their perspective they successfully reported and the meal is (or already was) removed.
- **The submission itself fails (network error)** -- No acknowledgement is shown; FEAT-02.SPEC-001's own error state ("Couldn't submit your report...") applies instead, and this notification only fires on a successful FEAT-02.SPEC-004 outcome.
- **The reporting adult's session expires between submission and the acknowledgement rendering** -- This cannot occur in practice, since the acknowledgement is the direct synchronous result of a submission that itself required an active session; if the session expired, the submission itself would have failed first per FEAT-02.SPEC-001's Access and Visibility table.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Triggered by (inbound) | A processed report fires this acknowledgement |
| FEAT-02.SPEC-001 (Report a Safety Concern) | References (inbound) | The acknowledgement renders within this screen's dialog |
| FEAT-04.SPEC-001 (Meal Swap alternatives list) | Navigation (outbound) | The CTA deep-links here for the emptied slot |

## Analytics and Success Signals

- **reporter_acknowledgement_shown** (reporter role) -- supports success-metrics.md: "Zero Allergy Incidents"
- **reporter_acknowledgement_cta_tapped** (destination: meal_swap) -- supports success-metrics.md: "Zero Allergy Incidents"

## Acceptance Criteria

**FEAT-02.SPEC-011-AC-01:** Given Maya submits a safety concern report, when FEAT-02.SPEC-004 processes it successfully, then she sees the acknowledgement "Report received" with the body naming the recipe and confirming its removal.

**FEAT-02.SPEC-011-AC-02:** Given Sam submits a safety concern report, when it processes successfully, then he -- and only he, not Maya -- sees the acknowledgement.

**FEAT-02.SPEC-011-AC-03:** Given Maya sees the acknowledgement, when she taps "Show me alternatives", then she is taken to the safe alternatives list (FEAT-04.SPEC-001) for the emptied slot.

**FEAT-02.SPEC-011-AC-04:** Given a recipe already has an open report from Maya, when Sam later submits his own report on the same recipe, then Sam still receives his own acknowledgement worded the same as a first-time report.

**FEAT-02.SPEC-011-AC-05:** Given a submission fails due to a network error, when FEAT-02.SPEC-001 shows its error state, then this acknowledgement does not appear.

**FEAT-02.SPEC-011-AC-06:** Given no preference control exists for this acknowledgement, when any reporting adult submits a report, then the acknowledgement always appears -- there is no way to turn it off.

**FEAT-02.SPEC-011-AC-07:** Given the acknowledgement is an in-app, in-session confirmation, when it is shown, then no email or push notification is sent for it.

**FEAT-02.SPEC-011-AC-08:** Given the reporting adult dismisses the dialog immediately after the acknowledgement appears, when they return to the plan, then the emptied slot itself confirms the removal, independent of whether they read the acknowledgement text.

**FEAT-02.SPEC-011-AC-09:** Given two reports are submitted by the same reporter on two different meals on the same day, when each is processed, then each produces its own acknowledgement -- they are never batched into one.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no control) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry N/A, expiry N/A) | 4 |
| Edge Cases | 4 | 4 |



# Notification Spec: Safety Concern Organiser Alert

## Overview

**Name:** Safety Concern Organiser Alert
**ID:** FEAT-02.SPEC-012
**Type:** Notification
**Purpose:** Tells the organiser a meal was removed from the plan because of a safety concern or a rule change, even when she wasn't the reporter.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- Alerting Maya (the organiser) whenever a meal is removed for safety reasons and she was not the one who caused the removal
- Covering both trigger sources: a safety-concern report (FEAT-02.SPEC-004) and a mid-week rule change (FEAT-02.SPEC-003)
- Content, channels, and delivery behavior for this alert

**Non-Goals:**
- Acknowledging the reporter's own submission -- owned by FEAT-02.SPEC-011 (Safety Concern Reporter Acknowledgement), a distinct message to a distinct audience (the reporter, not necessarily the organiser)
- Alerting the operator -- owned by FEAT-02.SPEC-013 (Safety Concern Operator Alert)
- Telling the household the eventual resolution outcome -- owned by FEAT-02.SPEC-014 (Safety Concern Resolution Notice), a later, distinct communication
- Suppressing this alert when Maya herself is the actor -- handled as a trigger condition within this spec (see Trigger), not treated as a separate spec

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always | Maya's day-to-day interaction with the plan is in-app (BRIEF.md, Behavioral Context); an in-app alert is visible the next time she opens the plan, which is her normal rhythm |
| Push | When Maya has enabled plan-related notifications for herself | A safety removal is a same-day disruption to the plan she is responsible for; if she is away from the product, a timely push lets her notice and act (e.g., approve a replacement) sooner than waiting for her next open |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Meal removed by a safety-concern report | FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Fires only when the reporting Member is not Maya | Reported Recipe name, removed meal's night, reporting Member's name |
| Meal(s) removed by a mid-week rule change | FEAT-02.SPEC-003 (Mid-Week Rule Change Re-Check) | Fires whenever one or more dinners are removed by a rule-change re-check (Maya is always the actor who changed the rule, but she still needs to be told which meals were affected, since the removal is a downstream automated consequence she did not directly choose meal-by-meal) | Removed Recipe name(s), affected night(s), the changed Member and rule |

## Audience and Preferences

**Recipients:** Maya (Organiser) only -- per the Access Matrix, Weekly Plan changes are Maya's responsibility (Full), and she is the household member accountable for the plan's overall state.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Push notifications for plan changes | On / Off | On | FEAT-01 (household settings, notification preferences) |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for household members (per the Feature Dependency Map and product-features.md, notification preferences cover on/off per member, not time-of-day windows); a safety removal is treated as timely enough to warrant delivery whenever it occurs rather than being held.

## Content Definition

**In-app:**
- **Title:** A meal was removed for safety
- **Body (safety-concern variant):** {recipe_name} was removed from {night} after {reporter_name} reported a safety concern. We've opened a swap so you can pick a safe alternative.
- **Body (rule-change variant):** {recipe_name} was removed from {night} because it's no longer safe for {member_name} after your latest rule update. We've opened a swap so you can pick a safe alternative.
- **CTA:** Choose a replacement -- deep-links to FEAT-04.SPEC-001 (Meal Swap, safe alternatives list) for the emptied slot on {night}

**Push:**
- **Title:** Plateful: a meal was removed for safety
- **Body:** {recipe_name} was removed from {night}. Tap to choose a safe replacement.
- **CTA:** Tapping the push opens FEAT-04.SPEC-001 (Meal Swap) for the emptied slot

**Batched variant (2+ meals removed by the same mid-week rule change):**
- **In-app title:** {count} meals removed for safety
- **In-app body:** {count} dinners were removed after your latest rule update, including {recipe_name} on {night}. We've opened swaps for each so you can pick safe alternatives.
- **Push title:** Plateful: {count} meals removed for safety
- **Push body:** {count} dinners are no longer safe under your latest rule update. Tap to review.
- **CTA:** Deep-links to the current Weekly Plan (FEAT-03) with the affected nights highlighted

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {recipe_name} | Recipe -- name | Thursday's Mushroom Risotto | Never empty -- name is required at recipe creation |
| {night} | Planned Meal -- night | Thursday | Never empty -- required on every Planned Meal |
| {reporter_name} | Member Profile -- display_name (of the reporting member) | Sam | Never empty -- required on every Member Profile |
| {member_name} | Member Profile -- display_name (of the member whose rule changed) | Jordan | Never empty -- required on every Member Profile |
| {count} | Derived -- number of dinners removed by the same rule-change re-check run | 3 | Never empty -- the batched variant only renders with 2 or more removed |

## Delivery Rules

**Batching:** All dinners removed by the same FEAT-02.SPEC-003 rule-change re-check run are delivered as one alert using the batched variant when 2 or more are removed in that run. A single safety-concern removal (always exactly one meal per report) never batches, since each report is its own event.
**Deduplication:** At most one alert per removal event (one safety-concern report, or one rule-change re-check run). A rule-change re-check that removes zero dinners produces no alert at all, per FEAT-02.SPEC-003's "no dinners affected" outcome.
**Retry on failure:** Push delivery failure is retried up to 3 times over 6 hours. After the final failure, the in-app alert stands as the delivery of record; the removal itself and the opened swap are already visible in-app regardless of push delivery success.
**Expiry:** The in-app alert does not expire -- it remains visible until Maya views the affected slot or dismisses it; the push notification, if undelivered after retries, is not resent, since the in-app state (the emptied slot and open swap) is the surviving signal.

## Edge Cases

- **Maya is the reporting adult herself** -- No alert fires for the safety-concern variant, since she is already aware from her own submission's acknowledgement (FEAT-02.SPEC-011); the rule-change variant still fires, since a rule-change removal is an automated downstream consequence distinct from directly reporting a meal.
- **Maya has disabled push notifications but the safety removal is time-sensitive** -- The in-app alert always fires regardless of the push preference; only the push channel is suppressed, consistent with the preference covering push specifically, not the in-app alert.
- **A rule-change re-check removes meals from two different nights in the same run** -- Both are included in a single batched alert (2+ removed), not two separate alerts, per the batching rule.
- **Maya has two open safety-concern removals from Sam in quick succession** -- Each safety-concern removal produces its own, unbatched alert (batching applies only to the rule-change path, since each safety-concern report is independently and immediately actionable), so Maya receives two separate alerts.
- **The affected Planned Meal slot is filled with a new pick before Maya opens the alert** -- The alert's CTA still deep-links to the slot; if a replacement is already chosen (e.g., by Sam suggesting a pick that Maya later accepted through another path), the CTA opens the slot showing its current state rather than an empty one, and Maya's action there simply confirms or changes what is now there.
- **A rule-change re-check run removes meals for a household where Maya has no push preference set yet (mid-onboarding)** -- The default (On) applies, so push is attempted as it would be for any household with a fully completed setup.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Triggered by (inbound) | A removal where Maya was not the reporter fires this alert |
| FEAT-02.SPEC-003 (Mid-Week Rule Change Re-Check) | Triggered by (inbound) | Any dinner(s) removed by a rule-change re-check fire this alert |
| FEAT-01 (Household Setup & Member Profiles) | References (inbound) | Push preference control lives here |
| FEAT-04.SPEC-001 (Meal Swap alternatives list) | Navigation (outbound) | Single-removal CTA deep-links here |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Navigation (outbound) | Batched-removal CTA deep-links to the plan view with affected nights highlighted |

## Analytics and Success Signals

- **organiser_safety_alert_delivered** (channel: in_app / push; trigger: safety_concern / rule_change; batched: yes/no) -- supports success-metrics.md: "Zero Allergy Incidents"
- **organiser_safety_alert_cta_tapped** (destination: meal_swap / plan_view) -- supports success-metrics.md: "Zero Allergy Incidents"
- **organiser_safety_alert_push_failed** (retry_count) -- N/A -- no Stage 2 metric measures push failure specifically for this alert; retained so silent push loss is observable given the in-app fallback's importance

## Acceptance Criteria

**FEAT-02.SPEC-012-AC-01:** Given Sam reports a safety concern and the meal is removed, when the removal completes, then Maya receives the in-app alert naming the recipe, the night, and Sam as the reporter.

**FEAT-02.SPEC-012-AC-02:** Given Maya has push enabled, when the alert in AC-01 fires, then she also receives a push notification.

**FEAT-02.SPEC-012-AC-03:** Given Maya herself reports a safety concern, when the meal is removed, then no organiser alert fires for that removal, since she is already informed via FEAT-02.SPEC-011.

**FEAT-02.SPEC-012-AC-04:** Given Maya adds a new hard allergy rule that causes two dinners to be removed in the same re-check run, when the removals complete, then Maya receives one batched alert naming both dinners, not two separate alerts.

**FEAT-02.SPEC-012-AC-05:** Given Maya has disabled push notifications, when a safety removal occurs, then she still receives the in-app alert, and no push is sent.

**FEAT-02.SPEC-012-AC-06:** Given Maya taps the CTA on a single-removal alert, when the tap registers, then she is taken to the safe alternatives list (FEAT-04.SPEC-001) for the emptied slot.

**FEAT-02.SPEC-012-AC-07:** Given Maya taps the CTA on a batched alert, when the tap registers, then she is taken to the current Weekly Plan with the affected nights highlighted.

**FEAT-02.SPEC-012-AC-08:** Given a rule-change re-check run removes zero dinners, when the run completes, then no organiser alert is generated.

**FEAT-02.SPEC-012-AC-09:** Given push delivery fails for this alert, when retries are exhausted after 6 hours, then the in-app alert remains the delivery of record and no error is shown to Maya.

**FEAT-02.SPEC-012-AC-10:** Given Sam reports two different meals in quick succession, when both are removed, then Maya receives two separate, unbatched alerts.

**FEAT-02.SPEC-012-AC-11:** Given Maya's household completed setup without an explicit push preference change, when a safety removal occurs, then push is attempted per the default (On) preference.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (in-app, push) | 2 |
| Trigger Paths | 2 | 2 |
| Preference States | 2 (push on, push off) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |



# Notification Spec: Safety Concern Operator Alert

## Overview

**Name:** Safety Concern Operator Alert
**ID:** FEAT-02.SPEC-013
**Type:** Notification
**Purpose:** Sends the safety-concern report to the operator by transactional email for review.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- The email alert sent to Riley (Operator) whenever a safety-concern Support Request is created
- Its content, delivery, and retry behavior over the transactional email capability

**Non-Goals:**
- Acknowledging the reporter -- owned by FEAT-02.SPEC-011 (Safety Concern Reporter Acknowledgement)
- Telling the organiser -- owned by FEAT-02.SPEC-012 (Safety Concern Organiser Alert)
- Telling the household the resolution -- owned by FEAT-02.SPEC-014 (Safety Concern Resolution Notice)
- The email transport mechanics (send retries, capability degradation) -- owned by FEAT-02.SPEC-010 (Transactional Email Delivery, Safety Reports); this spec defines the content and trigger, that spec defines the delivery contract
- Any in-app surface for the operator -- Riley has no in-app notification surface of his own in this feature; his review happens entirely through FEAT-22's Support View, which this alert's CTA points to

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always | Riley (Operator) is not a routine, signed-in user of the product's day-to-day screens (BRIEF.md, Target Users & Roles: "not a product role"); email is the reliable way to reach him for an on-demand review task, per the External Touchpoints table's transactional-email entry for FEAT-02 |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Safety-concern Support Request created | FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Fires once per newly created Support Request (not for a duplicate report against an already-open concern, since no new Support Request is created in that case) | Recipe name and ingredient list, reporter's role and first name, household name, reporting member's optional note, kid's age band and allergy detail if the concern involves a kid profile |

## Audience and Preferences

**Recipients:** Riley (Operator, support -- from v1) only, per the Access Matrix's Support View column (Full for Riley). No other role receives this alert.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this is a fixed operational alert with no recipient-controlled preference | -- | Always on | -- |

**Quiet Hours:** N/A -- the operator role has no quiet-hours concept in the product definition; a safety concern is treated as review-worthy whenever it is raised, consistent with the trust-critical nature of the feature.

## Content Definition

**Email:**
- **Subject:** Safety concern reported -- {household_name}
- **Body:**
  A household has reported a safety concern.

  Household: {household_name}
  Reported by: {reporter_name} ({reporter_role})
  Recipe: {recipe_name}
  Ingredients: {ingredient_list}
  Note from reporter: {reporter_note}

  Review this report in the Support View to record your findings.
- **CTA (button):** Review report -- deep-links to FEAT-22.SPEC-001 (Support Request Queue) with this household's open Support Request highlighted

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {household_name} | Household -- household_name | The Nguyen Family | Never empty -- required at household creation |
| {reporter_name} | Member Profile -- display_name | Maya | Never empty -- required on every Member Profile |
| {reporter_role} | Member Profile -- member_type | Organiser | Never empty -- required on every Member Profile |
| {recipe_name} | Recipe -- name | Thursday's Mushroom Risotto | Never empty -- required at recipe creation |
| {ingredient_list} | Recipe -- ingredients (names only, for review context) | Mushrooms, arborio rice, vegetable stock, parmesan | Never empty -- a recipe with zero ingredients cannot exist per FEAT-08/FEAT-10's "at least one ingredient to save" rule |
| {reporter_note} | Support Request -- note | "This has peanuts listed on the box but not in the app's ingredient list." | Renders as "No note provided." when the reporter left the note field empty |

## Delivery Rules

**Batching:** None -- each safety-concern Support Request is its own review case and is sent as its own email; batching two households' concerns into one email would slow the operator's per-household review and blur the audit trail.
**Deduplication:** At most one operator alert per Support Request. A duplicate report against an already-open concern (per FEAT-02.SPEC-004) does not create a new Support Request and therefore does not trigger a second alert.
**Retry on failure:** Governed by FEAT-02.SPEC-010's transactional email delivery contract -- transient failures retry per that spec's rules; regardless of email outcome, the Support Request remains visible and reviewable through FEAT-22's Support View.
**Expiry:** This alert does not expire in the sense of becoming unsendable -- there is no time window after which the report becomes not-worth-alerting-on, since a household's safety concern remains open and reviewable until resolved. If delivery is never confirmed, the underlying Support Request itself is the surviving signal Riley can find through the Support View.

## Edge Cases

- **Riley never opens the email** -- The Support Request remains Raised and visible in the Support View indefinitely until reviewed; this notification has no re-send or nagging behavior, since operator support access is on-demand per the feature's own scope, not a service-level-tracked queue.
- **Two households report safety concerns at effectively the same time** -- Each produces its own independent email; there is no cross-household batching.
- **A safety concern involves a kid profile's allergy** -- The email includes only that kid's age band and the specific allergy detail relevant to this report, never the kid's other profile data, per the Access Matrix notes on Riley's Kid Profile Data access.
- **The recipe's ingredient list is very long** -- The full ingredient list is included regardless of length, since the operator's review depends on seeing the complete list, not a truncated summary.
- **A second safety concern is reported for a different recipe by the same household while the first is still open** -- Each Support Request produces its own alert; the two are never merged, since they concern different recipes and may resolve independently.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Triggered by (inbound) | Report creation fires this alert |
| FEAT-02.SPEC-010 (Transactional Email Delivery, Safety Reports) | References (outbound) | Delivery mechanics and degradation behavior for this email are defined there |
| FEAT-22 (Operator Read-Only Support Access) | Navigation (outbound) | The CTA deep-links to the Support View for this household's open request |

## Analytics and Success Signals

- **operator_alert_sent** (household reference) -- supports success-metrics.md: "Zero Allergy Incidents"
- **operator_alert_cta_tapped** -- N/A -- email link engagement by the operator is not tracked as a Stage 2 metric; retained only if the transactional email capability reports open/click data, which is not guaranteed, so this event is emitted only when available.

## Acceptance Criteria

**FEAT-02.SPEC-013-AC-01:** Given a safety-concern Support Request is created, when this notification fires, then Riley receives an email with the subject "Safety concern reported -- {household_name}" and the recipe, ingredients, reporter identity, and note in the body.

**FEAT-02.SPEC-013-AC-02:** Given the reporter left the note field empty, when the email is composed, then the note line renders as "No note provided."

**FEAT-02.SPEC-013-AC-03:** Given Riley taps "Review report" in the email, when the link opens, then he lands on the Support View (FEAT-22) for that household's open Support Request.

**FEAT-02.SPEC-013-AC-04:** Given a duplicate report is submitted against an already-open concern on the same recipe, when FEAT-02.SPEC-004 processes it, then no second operator alert is sent, since no new Support Request was created.

**FEAT-02.SPEC-013-AC-05:** Given the reported concern involves a kid profile's allergy, when the email is composed, then it includes only the kid's age band and the specific allergy detail, never other Kid Profile Data.

**FEAT-02.SPEC-013-AC-06:** Given two households report safety concerns at effectively the same time, when both are processed, then each produces its own independent email to Riley.

**FEAT-02.SPEC-013-AC-07:** Given Riley never opens the alert email, when he later checks the Support View directly, then the open Support Request is still fully visible and reviewable there.

**FEAT-02.SPEC-013-AC-08:** Given no preference control exists for this alert, when any safety concern is reported, then the alert is always sent -- there is no way for it to be turned off.

**FEAT-02.SPEC-013-AC-09:** Given the recipe has a long ingredient list, when the email is composed, then the full list is included without truncation.

**FEAT-02.SPEC-013-AC-10:** Given the transactional email capability is temporarily unavailable, when the send is attempted, then delivery is retried per FEAT-02.SPEC-010's rules, and the Support Request remains reviewable regardless of the email's delivery status.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no control) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Safety Concern Resolution Notice

## Overview

**Name:** Safety Concern Resolution Notice
**ID:** FEAT-02.SPEC-014
**Type:** Notification
**Purpose:** Tells the household the outcome once the operator has resolved the report.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- Notifying the household when a safety-concern Support Request is resolved
- Covering both resolution outcomes: recipe confirmed safe (released) and recipe confirmed unsafe (kept excluded)
- Content, channels, and delivery behavior across in-app and email

**Non-Goals:**
- Acknowledging the original report -- owned by FEAT-02.SPEC-011 (Safety Concern Reporter Acknowledgement), a distinct, earlier message
- Alerting the organiser at removal time -- owned by FEAT-02.SPEC-012 (Safety Concern Organiser Alert), a distinct, earlier message
- Performing the review or writing the resolution outcome -- owned by FEAT-22 (Operator Read-Only Support Access) and FEAT-02.SPEC-005 (Safety Concern Resolution Outcome), which triggers this notification
- The email transport mechanics -- owned by FEAT-02.SPEC-010 (Transactional Email Delivery, Safety Reports); this spec defines content and trigger for its email channel, that spec defines the delivery contract

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always | The household's normal rhythm is checking the plan in-app; the resolution is a plan-relevant fact that belongs where the plan itself lives |
| Email | Always, in addition to in-app | A safety-concern resolution can arrive days after the original report, when the household may not be actively in the product; per the transactional-email External Touchpoint (ASMP-32), a safety-report resolution is one of the emails this capability delivers, ensuring the outcome reaches the household even if they have not opened the app since the report |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Safety-concern Support Request resolved | FEAT-02.SPEC-005 (Safety Concern Resolution Outcome) | Fires once per resolution, for either outcome (released or kept excluded) | Recipe name, resolution outcome, reporting Member's name, household reference |

## Audience and Preferences

**Recipients:** Maya (Organiser) and Sam (Other Adult Member) -- the whole household's adults, per the Access Matrix's Safety Reports column (Maya: Full, Sam: Own-only), since the outcome affects the shared plan and candidate pool both adults act within, regardless of who originally reported.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this is a safety-outcome notice with no independent on/off control | -- | Always on | -- |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for household members, and a safety resolution is treated as timely enough to deliver whenever the review completes rather than being held for a preferred hour.

## Content Definition

**In-app:**
- **Title (released):** {recipe_name} is safe again
- **Body (released):** Our operator reviewed the safety concern about {recipe_name} and confirmed it's safe. It may appear in your plan again going forward.
- **Title (kept excluded):** {recipe_name} will stay off your plan
- **Body (kept excluded):** Our operator reviewed the safety concern about {recipe_name} and confirmed the concern was valid. This recipe will not be suggested to your household again.
- **CTA:** View recipe -- deep-links to FEAT-08.SPEC-002 (Recipe Detail View) for {recipe_name}

**Email:**
- **Subject (released):** Update on your safety report: {recipe_name} is safe
- **Subject (kept excluded):** Update on your safety report: {recipe_name} stays off your plan
- **Body:**
  Hi,

  You reported a safety concern about {recipe_name} on {report_date}. Our operator has reviewed it.

  {resolution_detail}

  You can review this and any other reports in your household settings.
- **CTA (button):** Open Plateful -- deep-links to the household's plan (FEAT-03)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {recipe_name} | Recipe -- name | Thursday's Mushroom Risotto | Never empty -- required at recipe creation |
| {report_date} | Support Request -- derived from its creation timestamp | September 20 | Never empty -- every Support Request has a creation time |
| {resolution_detail} | Derived from Support Request's resolution outcome | "Good news -- the recipe is confirmed safe and may appear in your plan again." or "The concern was valid, and this recipe will not be suggested to your household again." | Never empty -- the notice only sends once an outcome is recorded |

## Delivery Rules

**Batching:** None -- each resolved Support Request produces its own notice; two households' resolutions are always separate, and even two resolutions for the same household on different recipes are sent as separate notices, since each concerns a distinct recipe and reporter context.
**Deduplication:** At most one resolution notice per resolved Support Request. A Support Request cannot be resolved twice (FEAT-22 transitions status Under review -> Resolved exactly once), so no duplicate-resolution scenario can arise from the source data itself; if a delivery retry occurs, the notice is not re-sent once delivery is confirmed.
**Retry on failure:** In-app delivery has no retry -- it is shown the next time any household adult opens the product. Email delivery failure is retried per FEAT-02.SPEC-010's transactional email delivery contract; after retries are exhausted, the in-app notice stands as the delivery of record.
**Expiry:** The in-app notice does not expire -- it remains visible until an adult views it or navigates to the affected recipe; the resolution fact itself (the recipe's current eligibility) is also reflected the next time the recipe is considered as a candidate (FEAT-02.SPEC-002), so the outcome is never lost even if the notice itself goes unread.

## Edge Cases

- **The household is deleted before the notice can be delivered** -- No notice is delivered, consistent with FEAT-02.SPEC-005's edge case for a deleted household; there is no recipient left to notify.
- **The recipe was already edited or re-imported between the report and the resolution (FEAT-10)** -- The notice still names the original recipe; if released, the notice's "may appear in your plan again" wording holds true only once the edited recipe also passes the ordinary safety check again (per XBR-19), which this notice does not itself guarantee -- it reports the report's resolution, not a renewed pass/fail determination.
- **Both Maya and Sam are entitled recipients and both are actively using the app when the resolution completes** -- Each receives their own copy of the in-app and email notice; the notice is not deduplicated across recipients, only across separate resolution events for the same recipient.
- **The original reporter is no longer a household member (has left) by the time the resolution completes** -- The notice still goes to Maya and Sam as the current household adults, since the recipe's eligibility affects the household's ongoing plan regardless of who originally reported it; the departed member's own historical reporting is preserved per SC-18 but they receive no further notice as a non-member.
- **The recipe is kept excluded, and the household later tries to search for it directly in the Recipe Library** -- The recipe shows the ineligibility reason (FEAT-02.SPEC-008) consistent with a permanent exclusion, which is the lasting, always-current signal beyond this one-time notice.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-005 (Safety Concern Resolution Outcome) | Triggered by (inbound) | The resolution fires this notice |
| FEAT-02.SPEC-010 (Transactional Email Delivery, Safety Reports) | References (outbound) | Delivery mechanics for the email channel |
| FEAT-08 (Recipe Library recipe detail) | Navigation (outbound) | The in-app CTA deep-links to the recipe's detail view |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Navigation (outbound) | The email CTA deep-links to the household's current plan |
| FEAT-02.SPEC-008 (Safety Badge & Disclaimer Display Rule) | References (outbound) | The lasting eligibility signal beyond this one-time notice |

## Analytics and Success Signals

- **resolution_notice_delivered** (channel: in_app / email; outcome: released / kept_excluded) -- supports success-metrics.md: "Zero Allergy Incidents"
- **resolution_notice_cta_tapped** (channel, destination: recipe_detail / plan_view) -- supports success-metrics.md: "Zero Allergy Incidents"

## Acceptance Criteria

**FEAT-02.SPEC-014-AC-01:** Given a safety-concern Support Request is resolved with a "confirmed safe" outcome, when this notice fires, then both Maya and Sam receive an in-app and email notice titled "{recipe_name} is safe again."

**FEAT-02.SPEC-014-AC-02:** Given a safety-concern Support Request is resolved with a "confirmed unsafe" outcome, when this notice fires, then both Maya and Sam receive an in-app and email notice titled "{recipe_name} will stay off your plan."

**FEAT-02.SPEC-014-AC-03:** Given Sam taps the in-app notice's "View recipe" CTA, when the tap registers, then he lands on the recipe's detail view.

**FEAT-02.SPEC-014-AC-04:** Given the household is deleted before the resolution completes, when FEAT-02.SPEC-005 processes the resolution, then no notice is delivered to anyone.

**FEAT-02.SPEC-014-AC-05:** Given a resolution's email delivery fails and retries are exhausted, when the household later opens the app, then the in-app notice still stands as the delivery of record.

**FEAT-02.SPEC-014-AC-06:** Given no preference control exists for this notice, when any resolution completes, then the notice is always delivered to Maya and Sam -- there is no way to turn it off.

**FEAT-02.SPEC-014-AC-07:** Given the original reporter has since left the household, when the resolution completes, then the notice is still delivered to the current household adults (Maya and Sam), not to the departed member.

**FEAT-02.SPEC-014-AC-08:** Given two households have concerns resolved at effectively the same time, when both notices are sent, then each household receives only its own notice, with no batching across households.

**FEAT-02.SPEC-014-AC-09:** Given a recipe is kept excluded and a household later searches for it in the Recipe Library, when they view it, then it shows the ineligibility reason consistent with a permanent exclusion (FEAT-02.SPEC-008).

**FEAT-02.SPEC-014-AC-10:** Given Maya and Sam are both actively using the app when a resolution completes, when the notice is delivered, then each receives their own independent copy of both the in-app and email notice.

**FEAT-02.SPEC-014-AC-11:** Given a Support Request can only be resolved once, when a resolution completes, then exactly one resolution notice is sent per Support Request -- never more.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (in-app, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on, no control) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
