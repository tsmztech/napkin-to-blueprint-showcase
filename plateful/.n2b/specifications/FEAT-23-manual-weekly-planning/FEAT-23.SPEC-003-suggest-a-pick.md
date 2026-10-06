---
document_type: spec
spec_type: screen
spec_id: FEAT-23.SPEC-003
spec_name: Suggest a Pick
spec_slug: suggest-a-pick
parent_feature: FEAT-23
parent_feature_name: Manual Weekly Planning
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Suggest a Pick

## Overview

**Name:** Suggest a Pick
**ID:** FEAT-23.SPEC-003
**Type:** Screen
**Purpose:** An other adult member browses safe choices for a target night and sends the organiser a suggested pick, which she accepts or declines.
**Parent Feature:** FEAT-23 -- Manual Weekly Planning

## Scope and Non-Goals

**In Scope:**
- Browsing and searching starter-library and household-imported recipes for a specific night, restricted to Sam's Own-only Manual Planning access
- Showing every candidate's eligibility, identically to FEAT-23.SPEC-002, per FEAT-23.SPEC-004
- Creating a Swap Suggestion (the pick-suggestion kind) for the target night, guarded by the one-open-suggestion-per-member-per-night limit (FEAT-23.SPEC-005)
- Showing Sam the status of an already-open suggestion for a night in place of letting him send a second one

**Non-Goals:**
- Maya using this screen -- she places picks directly through FEAT-23.SPEC-002; this screen exists for the suggest-then-approve role split (BRIEF.md, Target Users & Roles; scope-boundaries.md SC-04)
- Withdrawing a submitted suggestion -- not modeled per this feature's Non-Goals: a suggestion's only outcomes are accepted, declined, or lapsed; a member who wants to change one waits for the organiser's decision or its lapse
- Reviewing, accepting, or declining the suggestion once sent -- owned entirely by FEAT-04 (One-Tap Meal Swap) per feature-dependency-map.md's Swap Suggestion authority column and XBR-06; this screen only creates the record

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-23.SPEC-001 (Weekly Plan) | Sam taps a night with no open suggestion of his own | Target night; the night's current pick (if any), for context |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | No | No | This screen is not part of her navigation; her equivalent action is a direct pick or change via FEAT-23.SPEC-002. If reached directly (e.g., a stale link), she is redirected to FEAT-23.SPEC-002 for the same night |
| Sam (Other Adult Member) | Full screen | Search, browse, select an eligible recipe, send it as a suggestion for the target night | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile; the screen is not reachable |
| Jordan (older kid, limited login -- Later) | No | No | Manual Planning access is None for this row |
| Riley (Operator, support -- from v1) | No | No | Manual Planning access is None for Riley |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- the target night and any in-progress search text are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title naming the target night (e.g., "Suggest a dinner for Friday"), a back arrow (returns to FEAT-23.SPEC-001), and a search input.

**Body:** The same browse/search candidate list pattern as FEAT-23.SPEC-002 -- each card shows recipe name, cook_time, rough_cost, and either the safety_badge (eligible) or a plain ineligibility reason (ineligible), per FEAT-23.SPEC-004. Eligible cards carry a "Send as suggestion" action in place of FEAT-23.SPEC-002's "Place"/"Replace." If Friday already has a placed dinner, that recipe is shown at the top marked "Currently planned" for context (not selectable, since Sam is suggesting a change, not confirming the existing pick).

**Footer:** None -- the send action is inline on each card.

### Responsive Behavior

- **Compact breakpoint:** Single-column card list, full width; search input spans the header width.
- **Medium size class and above:** Same single-column list, capped at the same consistent platform-wide content width as FEAT-23.SPEC-002 and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-23.SPEC-001 (Weekly Plan) | Screen closes | Standard transition; no suggestion sent |
| Search input | Type | Filters the candidate list by name or ingredient | List updates | Results within about a second, with an inline loading indicator |
| Eligible recipe card's "Send as suggestion" action | Tap | Creates a Swap Suggestion (suggesting_member: Sam, night, proposed_recipe, outcome: Suggested) | Button shows loading state during the write | Success: navigates back to FEAT-23.SPEC-001, which now shows "Suggestion sent -- waiting on Maya" for that night. Failure: inline error, retry offered |
| Ineligible recipe card | Tap | No selection action -- display-only for its ineligibility reason | None | Plain reason remains visible; no action fires |
| "Send as suggestion" action (while a send is in progress) | Tap | No action -- debounced | None | Button remains in its loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> search input -> candidate cards in list order, each card's action (when eligible) reachable immediately after its content.
- **Dynamic-change announcements:** Search results updating, an ineligibility reason appearing, and a send's success or failure are each announced to assistive technology.
- **Keyboard alternatives:** Every action on this screen (search, select, send) is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading candidates | Inline loading indicator over the (empty) candidate list | Screen first opens or a search query changes | Candidates finish loading (within about a second) |
| Populated | Candidate list shows eligible and ineligible recipes per FEAT-23.SPEC-004 | Candidates finish loading | User selects a recipe, searches again, or navigates away |
| No results | Plain "Nothing found" message with a suggestion to broaden the search | A search query matches no candidates | User clears or changes the search query |
| Sending | Selected card's action shows a loading state; other cards remain visible but inactive | User taps "Send as suggestion" | Send completes or fails |
| Error | Inline error banner on the selected card: "Couldn't send this suggestion -- check your connection and try again," with a Retry control; the attempted selection remains visible | The send fails | User taps Retry (succeeds) or navigates away |
| Offline/Degraded | Previously loaded candidates remain viewable and browsable; the "Send as suggestion" action shows "Connect to send a suggestion" and is disabled, since a suggestion's recipe must pass the safety check before it can be sent | Connectivity is lost while the screen is open | Connectivity is restored -- the action re-enables |

## Validation Rules

Validation governed by FEAT-23.SPEC-004 (Safe-Choice Filtering & Placement Block) for candidate eligibility, and FEAT-23.SPEC-005 (Manual Planning Validation & Limits) for the one-open-suggestion-per-member-per-night limit and the one-week-ahead window this screen's nights are drawn from.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-23.SPEC-001 (Weekly Plan) | -- |
| Successful "Send as suggestion" | FEAT-23.SPEC-001 (Weekly Plan) | -- |

## Data Model

**Creates:** Swap Suggestion -- suggesting_member (Sam), night, proposed_recipe, and outcome set to "Suggested."
**Reads:** Recipe (name, ingredients, cook_time, rough_cost, origin) for the candidate list, from FEAT-08 and FEAT-10; Planned Meal (recipe) for the night's current pick, shown for context only.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-01: every candidate shown as eligible has already passed the same app-enforced, fail-closed allergy and religious-rule check as every other path onto the plan (FEAT-23.SPEC-004).
- XBR-06: Sam suggests rather than places; the organiser accepts or declines every suggestion with one tap; at most one open suggestion per member per night (FEAT-23.SPEC-005); an unanswered suggestion lapses when its night passes and Sam is told the outcome (both the accept/decline/lapse flow and its notifications are owned by FEAT-04).
- FEAT-23.SPEC-005 governs the one-open-suggestion-per-member-per-night limit -- this screen is only reachable for a night where Sam has no other open suggestion.

## Edge Cases

- **Sending a suggestion fails (e.g., dropped connection)** -- The attempted suggestion is not silently dropped; the recipe stays selected on screen with a retry option, per the Side-Effect Inventory's failure disposition.
- **Maya places a direct pick on the same night while Sam is still choosing a suggestion** -- Sam's "Send as suggestion" is rejected on submit with the message "Maya already picked a dinner for {Night} -- choose a different night to suggest," per the dependency map's Swap Suggestion contention note (first-decision-wins with reject-with-refresh: a direct pick by Maya on the same slot supersedes any open suggestion attempt for it).
- **Sam attempts to send a second suggestion for a night where he already has one open** -- This screen is not reachable for that night from FEAT-23.SPEC-001 (which shows the pending status instead); a suggestion send attempted through a stale screen state is rejected with "You already suggested a pick for {Night} -- wait for Maya's decision or its lapse," per FEAT-23.SPEC-005.
- **A household's hard dietary rule tightens mid-week while this screen is open (XBR-02)** -- The candidate list re-filters on the next load or search; an attempted send on a recipe that failed the re-check in the background is rejected with its ineligibility reason shown inline.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-001 (Weekly Plan) | Navigation (inbound/outbound) | Entry point; returns here on send or back, where the pending badge then appears |
| FEAT-23.SPEC-004 (Safe-Choice Filtering & Placement Block) | References (inbound) | Supplies every candidate's eligibility and ineligibility reason, identically to FEAT-23.SPEC-002 |
| FEAT-23.SPEC-005 (Manual Planning Validation & Limits) | References (inbound) | Governs the one-open-suggestion-per-member-per-night limit |
| FEAT-08 (Recipe Library, Starter Recipes) | References (inbound) | Source of starter-library candidates |
| FEAT-10 (Recipe Import from Web Link) | References (inbound) | Source of the household's imported-recipe candidates |
| FEAT-04 (One-Tap Meal Swap) | Affects (outbound) | The created Swap Suggestion is reviewed, accepted, or declined through FEAT-04's flow, and notified through FEAT-04.SPEC-006 |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| manual_pick_suggested | night, recipe eligibility outcome (eligible) | A "Send as suggestion" action completes successfully | supports success-metrics.md: "Household Member Participation" (its target explicitly counts an adult member who has "ticked, added, or suggested something") |
| manual_pick_blocked_unsafe | night, count of ineligible candidates in view | Sam views a candidate list containing at least one ineligible recipe, or attempts to select one | supports success-metrics.md: "Zero Allergy Incidents" |

## Acceptance Criteria

**FEAT-23.SPEC-003-AC-01:** Given Sam taps Friday's night row on FEAT-23.SPEC-001 with no open suggestion of his own, when this screen opens, then the candidate list shows both eligible and ineligible recipes, with ineligible ones carrying a plain reason.

**FEAT-23.SPEC-003-AC-02:** Given Sam is choosing for Friday, when he types a search term, then results within about a second show matching recipes with an inline loading indicator while they load.

**FEAT-23.SPEC-003-AC-03:** Given Sam is choosing for Friday and a recipe is eligible, when he taps "Send as suggestion," then a Swap Suggestion is created with outcome "Suggested" and he is returned to FEAT-23.SPEC-001, which now shows "Suggestion sent -- waiting on Maya" for Friday.

**FEAT-23.SPEC-003-AC-04:** Given Sam sees a recipe marked ineligible because it breaks his own religious rule, when he looks at that card, then no "Send as suggestion" action is offered and the plain reason names the affected rule.

**FEAT-23.SPEC-003-AC-05:** Given Sam searches for a recipe with no matches, when the search completes, then a plain "Nothing found" message appears with a suggestion to broaden the search.

**FEAT-23.SPEC-003-AC-06:** Given Sam's "Send as suggestion" fails to save due to a dropped connection, when the failure occurs, then the attempted suggestion stays selected on screen with an inline error and a Retry control.

**FEAT-23.SPEC-003-AC-07:** Given Sam loses connectivity while browsing candidates, when he looks at any card's send action, then it shows "Connect to send a suggestion" and is disabled, while previously loaded candidates remain browsable.

**FEAT-23.SPEC-003-AC-08:** Given Maya places a direct pick on Friday while Sam is still browsing this screen for Friday, when Sam then taps "Send as suggestion," then the send is rejected with "Maya already picked a dinner for Friday -- choose a different night to suggest."

**FEAT-23.SPEC-003-AC-09:** Given Sam already has an open suggestion for Saturday, when he reaches this screen for Saturday through a stale screen state and attempts to send another, then the send is rejected with "You already suggested a pick for Saturday -- wait for Maya's decision or its lapse."

**FEAT-23.SPEC-003-AC-10:** Given a household member's allergy tightens mid-week while Sam is browsing candidates, when he attempts to send a recipe that has since become ineligible, then the send is rejected and the recipe's card shows its ineligibility reason.

**FEAT-23.SPEC-003-AC-11:** Given Maya (Organiser) attempts to reach this screen directly, when the navigation is attempted, then she is redirected to FEAT-23.SPEC-002 for the same night instead.

**FEAT-23.SPEC-003-AC-12:** Given Sam sends an eligible suggestion, when the send succeeds, then a manual_pick_suggested event is recorded for that night, supporting success-metrics.md: "Household Member Participation."

**FEAT-23.SPEC-003-AC-13:** Given Sam's household is on the free tier, when he suggests a pick, then the suggestion flow behaves identically to a paid household's, since Manual Weekly Planning and its suggestion flow operate on either tier (XBR-05).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 (loading, populated, no results, sending, error, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |
