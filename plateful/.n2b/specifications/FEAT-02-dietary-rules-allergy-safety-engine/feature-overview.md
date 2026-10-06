---
document_type: feature-overview
feature_number: FEAT-02
feature_name: Dietary Rules & Allergy Safety Engine
feature_slug: dietary-rules-allergy-safety-engine
priority_tier: Core
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 14
screen_count: 1
automation_count: 4
logic_rule_count: 4
integration_count: 1
notification_count: 4
---

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
