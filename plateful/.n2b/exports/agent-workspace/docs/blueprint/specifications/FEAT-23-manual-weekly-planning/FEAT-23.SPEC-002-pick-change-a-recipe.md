---
document_type: spec
spec_type: screen
spec_id: FEAT-23.SPEC-002
spec_name: Pick / Change a Recipe
spec_slug: pick-change-a-recipe
parent_feature: FEAT-23
parent_feature_name: Manual Weekly Planning
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: Pick / Change a Recipe

## Overview

**Name:** Pick / Change a Recipe
**ID:** FEAT-23.SPEC-002
**Type:** Screen
**Purpose:** The organiser browses or searches the recipe library's safe choices for a target night and places or replaces that night's pick.
**Parent Feature:** FEAT-23 -- Manual Weekly Planning

## Scope and Non-Goals

**In Scope:**
- Browsing and searching starter-library and household-imported recipes for a specific night
- Showing every candidate's eligibility (safe / ineligible with a plain reason), per FEAT-23.SPEC-004
- Placing a new pick on an empty night, or replacing an existing pick, for the organiser only
- Loading the existing pick's context when the organiser arrives to change a night

**Non-Goals:**
- Sam using this screen to place a pick himself -- excluded per scope-boundaries.md SC-04 and the Access Matrix (Manual Planning: Own-only for Sam); he uses FEAT-23.SPEC-003 (Suggest a Pick) instead, which shares this screen's browse/search pattern but ends in sending a suggestion, not placing
- Determining which recipes are eligible -- owned by FEAT-23.SPEC-004 (Safe-Choice Filtering & Placement Block); this screen only displays that spec's result and blocks selection accordingly
- Writing the Planned Meal record, recalculating the week's total, or signalling the grocery list -- owned by FEAT-23.SPEC-006 (Apply Manual Pick), which this screen triggers on selection

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-23.SPEC-001 (Weekly Plan) | Maya taps an empty night | Target night; no existing pick |
| FEAT-23.SPEC-001 (Weekly Plan) | Maya taps "Change" on a picked night | Target night; the existing Planned Meal's recipe, cook_time, and rough_cost, shown pre-selected in the list |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Search, browse, select an eligible recipe, place or replace the night's pick | -- |
| Sam (Other Adult Member) | No | No | This screen is not offered to Sam; tapping a night on FEAT-23.SPEC-001 takes him to FEAT-23.SPEC-003 (Suggest a Pick) instead, per scope-boundaries.md SC-04 |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile; the screen is not reachable |
| Jordan (older kid, limited login -- Later) | No | No | Manual Planning access is None for this row; navigation to this screen is not offered |
| Riley (Operator, support -- from v1) | No | No | Manual Planning access is None for Riley |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- the target night and any in-progress search text are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title naming the target night (e.g., "Pick a dinner for Wednesday"), a back arrow (returns to FEAT-23.SPEC-001 without changing the night), and a search input.

**Body:** A scrollable list of candidate recipes -- starter-library and the household's imported recipes (FEAT-08, FEAT-10) -- each card showing: recipe name, cook_time, rough_cost, and either the "checked against allergies" safety_badge (eligible) or a plain ineligibility reason naming the affected member and rule (ineligible, per FEAT-23.SPEC-004). Eligible cards carry a "Place" (or, when changing an existing pick, "Replace") action; ineligible cards show their reason with no selectable action. If the organiser arrived to change a night, the currently placed recipe's card is marked "Currently picked" at the top of the list.

**Footer:** None -- selection happens inline on each card.

### Responsive Behavior

- **Compact breakpoint:** Single-column card list, full width; search input spans the header width.
- **Medium size class and above:** Same single-column list, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-23.SPEC-001 (Weekly Plan) | Screen closes | Standard transition; target night unchanged |
| Search input | Type | Filters the candidate list by name or ingredient | List updates | Results within about a second, with an inline loading indicator (product-features.md, States) |
| Eligible recipe card's "Place"/"Replace" action | Tap | Triggers FEAT-23.SPEC-006 (Apply Manual Pick) create/update path for the target night with this recipe | Button shows loading state during the write | Success: navigates back to FEAT-23.SPEC-001 with the night showing the new pick. Failure: inline error, pick stays on screen with a retry (see Edge Cases) |
| Ineligible recipe card | Tap | No selection action -- card is display-only for its ineligibility reason | None | The plain reason (e.g., "Not safe for Jordan -- contains peanuts") remains visible; no action fires |
| "Place"/"Replace" action (while a save is in progress) | Tap | No action -- debounced | None | Button remains in its loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> search input -> candidate cards in list order, each card's action (when eligible) reachable immediately after its content.
- **Dynamic-change announcements:** Search results updating, an ineligibility reason appearing, and a save's success or failure are each announced to assistive technology.
- **Keyboard alternatives:** Every action on this screen (search, select, place/replace) is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading candidates | Inline loading indicator over the (empty) candidate list | Screen first opens or a search query changes | Candidates finish loading (within about a second) |
| Populated | Candidate list shows eligible and ineligible recipes per FEAT-23.SPEC-004 | Candidates finish loading | User selects a recipe, searches again, or navigates away |
| No results | Plain "Nothing found" message with a suggestion to broaden the search | A search query matches no candidates | User clears or changes the search query |
| Saving | Selected card's action shows a loading state; other cards remain visible but inactive | User taps Place or Replace | Save completes or fails |
| Error | Inline error banner on the selected card: "Couldn't save this pick -- check your connection and try again," with a Retry control; the attempted selection remains visible | The save triggered by Place/Replace fails | User taps Retry (succeeds) or navigates away |
| Offline/Degraded | Previously loaded candidates remain viewable and browsable; Place/Replace actions show "Connect to place a pick" and are disabled, since every placement must pass the safety check before it lands on the plan | Connectivity is lost while the screen is open | Connectivity is restored -- actions re-enable |

## Validation Rules

Validation governed by FEAT-23.SPEC-004 (Safe-Choice Filtering & Placement Block) for candidate eligibility, and FEAT-23.SPEC-005 (Manual Planning Validation & Limits) for the one-dinner-per-night and one-week-ahead limits that bound which nights this screen can be reached for.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-23.SPEC-001 (Weekly Plan) | -- |
| Successful Place or Replace | FEAT-23.SPEC-001 (Weekly Plan) | -- |

## Data Model

**Creates:** None directly -- a successful "Place" triggers FEAT-23.SPEC-006 to create the Planned Meal record.
**Reads:** Recipe (name, ingredients, cook_time, rough_cost, origin) for the candidate list, from FEAT-08 (starter library) and FEAT-10 (household imports); Planned Meal (recipe, cook_time, rough_cost) for the existing pick when changing a night; Dietary Rule is never read directly by this screen -- eligibility arrives pre-computed from FEAT-23.SPEC-004.
**Updates:** None directly -- a successful "Replace" triggers FEAT-23.SPEC-006 to update the existing Planned Meal record.
**Deletes:** None.

## Business Rules

- XBR-01: every candidate shown as eligible has already passed the same app-enforced, fail-closed allergy and religious-rule check as every other path onto the plan (FEAT-23.SPEC-004); an ineligible recipe can never be selected here.
- XBR-19: imported recipes (FEAT-10) appear in the same candidate pool as starter recipes (FEAT-08); an imported recipe that was edited must re-pass the safety check before it can appear here again.
- FEAT-23.SPEC-005 governs the one-dinner-per-night rule -- this screen's "Place" action is only ever reachable for a night with no existing dinner, and "Replace" only for a night that already has one.

## Edge Cases

- **Saving a placement fails (e.g., dropped connection)** -- The attempted pick stays visible with a retry option; it is never silently dropped (per the Side-Effect Inventory's failure disposition).
- **Contact changed by another user between load and save -- the target night was picked, changed, or cleared by Maya from another device while this screen was open** -- The Place/Replace action is rejected on save with the message "This night changed while you were choosing -- reload to see the latest pick," and the screen reloads the night's current state on acknowledgement. Resolution: reject-with-refresh, per the dependency map's Contention note for Planned Meal.
- **A household's hard dietary rule tightens mid-week while this screen is open (XBR-02)** -- The candidate list re-filters on the next load or search, so a recipe that was eligible when the screen opened may become ineligible before the organiser selects it; an attempted "Place" on a recipe that failed the re-check in the background is rejected with the same ineligibility reason shown inline.
- **The organiser taps Place twice rapidly on the same card** -- The second tap has no effect while the first save is in progress (per the debounced Interactions row).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-001 (Weekly Plan) | Navigation (inbound/outbound) | Entry point for both empty-night picks and picked-night changes; returns here on success or back |
| FEAT-23.SPEC-004 (Safe-Choice Filtering & Placement Block) | References (inbound) | Supplies every candidate's eligibility and ineligibility reason |
| FEAT-23.SPEC-005 (Manual Planning Validation & Limits) | References (inbound) | Governs which nights this screen can be reached for and the one-dinner-per-night rule |
| FEAT-23.SPEC-006 (Apply Manual Pick) | Triggers (outbound) | Place/Replace actions trigger this automation's create/update path |
| FEAT-08 (Recipe Library, Starter Recipes) | References (inbound) | Source of starter-library candidates |
| FEAT-10 (Recipe Import from Web Link) | References (inbound) | Source of the household's imported-recipe candidates |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| manual_meal_picked | night, recipe eligibility outcome (eligible), pick type (new / replace) | A Place or Replace action completes successfully | supports success-metrics.md: "Manual Week Completion" |
| manual_pick_blocked_unsafe | night, count of ineligible candidates in view | The organiser views a candidate list containing at least one ineligible recipe, or attempts to select one | supports success-metrics.md: "Zero Allergy Incidents" |

## Acceptance Criteria

**FEAT-23.SPEC-002-AC-01:** Given Maya taps Wednesday's empty night on FEAT-23.SPEC-001, when this screen opens, then the candidate list shows both eligible and ineligible recipes, with ineligible ones carrying a plain reason and no selectable action.

**FEAT-23.SPEC-002-AC-02:** Given Maya is on this screen for Wednesday, when she types a search term, then results within about a second show matching recipes with an inline loading indicator while they load.

**FEAT-23.SPEC-002-AC-03:** Given Maya is choosing for Wednesday and a recipe is eligible, when she taps "Place," then FEAT-23.SPEC-006 writes the pick and she is returned to FEAT-23.SPEC-001 with Wednesday showing the new recipe.

**FEAT-23.SPEC-002-AC-04:** Given Maya arrived to change Friday's existing pick, when this screen opens, then Friday's currently placed recipe is shown marked "Currently picked" at the top of the candidate list.

**FEAT-23.SPEC-002-AC-05:** Given Maya is choosing a replacement for Friday, when she taps "Replace" on a different eligible recipe, then FEAT-23.SPEC-006 updates Friday's Planned Meal and she is returned to FEAT-23.SPEC-001 with the new recipe shown.

**FEAT-23.SPEC-002-AC-06:** Given Maya sees a recipe marked ineligible because it breaks Jordan's peanut allergy, when she looks at that card, then no Place action is offered and the plain reason names Jordan and the allergy.

**FEAT-23.SPEC-002-AC-07:** Given Maya searches for a recipe with no matches, when the search completes, then a plain "Nothing found" message appears with a suggestion to broaden the search.

**FEAT-23.SPEC-002-AC-08:** Given Maya's Place action fails to save due to a dropped connection, when the failure occurs, then the attempted pick stays visible with an inline error and a Retry control.

**FEAT-23.SPEC-002-AC-09:** Given Maya loses connectivity while browsing candidates, when she looks at any card's Place/Replace action, then it shows "Connect to place a pick" and is disabled, while previously loaded candidates remain browsable.

**FEAT-23.SPEC-002-AC-10:** Given Maya opened this screen to change Thursday, when Thursday's pick is cleared from another of her devices while this screen is still open, when she then taps Replace, then the save is rejected with "This night changed while you were choosing -- reload to see the latest pick," and the screen reloads Thursday's current state.

**FEAT-23.SPEC-002-AC-11:** Given a household member's allergy tightens mid-week while Maya is browsing candidates, when she attempts to place a recipe that has since become ineligible, then the placement is rejected and the recipe's card shows its ineligibility reason.

**FEAT-23.SPEC-002-AC-12:** Given Maya taps "Place" twice rapidly on the same eligible recipe, when the first tap begins saving, then the second tap has no effect and no duplicate Planned Meal is created.

**FEAT-23.SPEC-002-AC-13:** Given Maya places an eligible recipe imported by the household (FEAT-10), when the placement succeeds, then it is treated identically to a starter-library placement, per XBR-19.

**FEAT-23.SPEC-002-AC-14:** Given Maya is choosing a recipe for a night that already has a dinner, when she opens this screen from "Change," then the action offered on eligible cards reads "Replace," never "Place," per FEAT-23.SPEC-005's one-dinner-per-night rule.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 (loading, populated, no results, saving, error, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |
