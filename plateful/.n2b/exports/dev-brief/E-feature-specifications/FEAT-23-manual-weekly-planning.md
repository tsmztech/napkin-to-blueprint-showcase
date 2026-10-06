# FEAT-23 — Manual Weekly Planning

This chapter covers FEAT-23, Manual Weekly Planning, a Core-tier feature. It contains 6 specifications carrying 85 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-23.SPEC-001 | Weekly Plan (Manual Week Builder) | screen | 16 |
| FEAT-23.SPEC-002 | Pick / Change a Recipe | screen | 14 |
| FEAT-23.SPEC-003 | Suggest a Pick | screen | 13 |
| FEAT-23.SPEC-004 | Safe-Choice Filtering & Placement Block | logic-rule | 15 |
| FEAT-23.SPEC-005 | Manual Planning Validation & Limits | logic-rule | 14 |
| FEAT-23.SPEC-006 | Apply Manual Pick | automation | 13 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Manual Weekly Planning

## Summary

**Feature:** Manual Weekly Planning
**ID:** FEAT-23
**Description:** Any household, on either tier, can build the week's dinners by hand — picking a recipe for each night from the library — and the shared grocery list builds itself from those picks. This is the free tier's planning experience, and the way a paid household hand-picks any night it prefers to choose itself.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md, Business Context states "the free tier covers manual planning and the shared grocery list," yet the draft had no feature that delivers manual planning — free households could only see an upgrade prompt. Market research shows weekly plan-to-shopping-list automation in all four profiled products (Feature Landscape, Common Features). Core because every new household starts on the free tier and plans through this feature until it upgrades, so both the free product and the path to paid depend on it; MVP because the free tier exists at launch.

**Key Capabilities:**
- Pick a dinner for a night — Organiser chooses a recipe from the starter library or the household's imported recipes for any night of the week
- See only safe choices — Recipes that break a household member's allergy or religious rule are marked ineligible with a plain reason and cannot be placed
- Change or clear a night — Organiser replaces or empties any night's pick
- Suggest a pick — Other adult members suggest a dinner for a night, which the organiser accepts or declines
- Build the list automatically — Every pick adds its ingredients to the shared grocery list, and every change updates it

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-23.SPEC-001 | Weekly Plan (Manual Week Builder) | Screen | Maya, Sam | Organiser (and, view-only, other adults) sees the seven-night week, the running budget total, and taps a night to pick, change, or clear it |
| FEAT-23.SPEC-002 | Pick / Change a Recipe | Screen | Maya | Organiser browses or searches the recipe library's safe choices and places or replaces a night's pick |
| FEAT-23.SPEC-003 | Suggest a Pick | Screen | Sam | Other adult member browses safe choices and sends the organiser a suggested pick for a night |
| FEAT-23.SPEC-004 | Safe-Choice Filtering & Placement Block | Logic/Rule | Maya, Sam | Filters every candidate recipe through the household's allergy/religious-rule check, marks ineligible ones with a plain reason, and blocks their placement; fails closed on incomplete ingredient data |
| FEAT-23.SPEC-005 | Manual Planning Validation & Limits | Logic/Rule | Maya, Sam | Enforces one dinner per night, seven nights per week, planning up to one week ahead, and one open pick suggestion per member per night |
| FEAT-23.SPEC-006 | Apply Manual Pick | Automation | Maya | Writes a pick, change, or clear to the night's slot, recalculates the week's estimated total, and signals the grocery list to recalculate |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Pick a dinner for a night | FEAT-23.SPEC-002, FEAT-23.SPEC-006 | Recipe picker screen places the pick; Apply Manual Pick writes it to the slot | Phase 2 (Explicit) |
| See only safe choices | FEAT-23.SPEC-002, FEAT-23.SPEC-004 | Every candidate shown by the picker is filtered by the safety rule before it can be selected | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |
| Change or clear a night | FEAT-23.SPEC-001, FEAT-23.SPEC-002, FEAT-23.SPEC-006 | Clear action on the week screen; replace flow reuses the recipe picker; Apply Manual Pick executes both | Phase 2 (Explicit) |
| Suggest a pick | FEAT-23.SPEC-003 | Other adult member picks a safe alternative and sends it to the organiser as a suggestion | Phase 2 (Explicit) |
| Build the list automatically | FEAT-23.SPEC-006 | Apply Manual Pick signals the Shared Grocery List (FEAT-06) to recalculate the moment a pick, change, or clear completes | Phase 2 (Explicit) — execution is a Cross-Feature Touchpoint |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-23.SPEC-004 | Safe-Choice Filtering & Placement Block | Phase 5 (Rule Discovery) | Marking ineligibility with a plain, member-specific reason and fail-closed handling of incomplete ingredient data (XBR-01) is conditional logic shared across two screens (SPEC-002, SPEC-003) — it crosses the standalone-spec threshold rather than staying inline in either picker |
| FEAT-23.SPEC-005 | Manual Planning Validation & Limits | Phase 5 (Rule Discovery) | The Validation & Limits field names five distinct rules across two entities (one dinner/night, seven nights/week, one-week-ahead window, one open suggestion/member/night) that are shared across three specs — this exceeds the inline-validation threshold |
| FEAT-23.SPEC-006 | Apply Manual Pick | Phase 4 (Trigger-Response) | Pick, change, and clear all end in the same write to the Planned Meal slot plus a total recalculation and a grocery-list signal; factoring it out gives the three entry points (SPEC-001's clear action, SPEC-002's pick/change) one shared write path instead of three divergent ones |

## Entity-Lifecycle Coverage Matrix

**Entity: Weekly Plan**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-23.SPEC-001 | Auto-initializes with origin "manually built" and seven empty nights the first time the organiser opens a week that has not started (inline, single-step) | FEAT-03 also creates a Weekly Plan (generated, paid tier) — the two never both create the same week |
| Read (single) | FEAT-23.SPEC-001 | Displays the week currently being built | -- |
| Read (list) | N/A | Browsing past or multiple weeks is Weekly Plan History (FEAT-19, v1); this feature only ever shows the current/next week being built | -- |
| Update | FEAT-23.SPEC-006 | Recalculates estimated_total each time a pick, change, or clear completes | Approval and status transitions are FEAT-03's responsibility |
| Delete/Archive | N/A | Owned by FEAT-18 (deletion cascade) and archived at week end by the feature that owns week transition; FEAT-23 never deletes a Weekly Plan itself — recorded as an explicit non-goal for this feature, not an omission | -- |
| State Transition | N/A | Status (Generated/Started -> Reviewed -> Approved -> Active -> Archived) is driven by FEAT-03's approval and week-start adoption logic; a manually built week enters at "Started" via SPEC-001's creation and this feature never advances it further | -- |

**Entity: Planned Meal**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-23.SPEC-002, FEAT-23.SPEC-006 | Picker places a recipe on a night; Apply Manual Pick writes the record with status Picked, guarded by SPEC-004 (safety) and SPEC-005 (limits) | FEAT-03 also creates Planned Meals (AI-proposed); FEAT-11 creates leftover-lunch ones |
| Read (single) | FEAT-23.SPEC-001, FEAT-23.SPEC-002 | Week screen shows each night's current pick; picker loads the existing pick when changing it | -- |
| Read (list) | FEAT-23.SPEC-001 | All seven nights shown together, with unplanned nights marked "nothing planned" | -- |
| Update | FEAT-23.SPEC-002, FEAT-23.SPEC-006 | Change flow selects a new recipe; Apply Manual Pick overwrites the slot's recipe, cook_time, and rough_cost | -- |
| Delete/Archive | FEAT-23.SPEC-001, FEAT-23.SPEC-006 | Hard delete: the clear action removes the record entirely, with no restore path — the organiser simply picks again if she changes her mind; the deletion cascades to remove that night's ingredients from the Grocery List's next recalculation; no retention/purge concern applies because a cleared night is a future/current unlived slot, not completed-week history (which is retained for the life of the account under FEAT-19/SC-18, untouched by this operation) | -- |
| State Transition | FEAT-23.SPEC-006 | Sets status to Picked on create; other statuses (Confirmed, Swapped, Removed, Cooked) are set by FEAT-11, FEAT-04, and FEAT-02 respectively | -- |

**Entity: Swap Suggestion**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-23.SPEC-003 | Sam picks a safe recipe for a night and sends it to Maya as a pick suggestion, guarded by SPEC-004 (safety) and SPEC-005 (one open per member per night) | FEAT-04 also creates Swap Suggestions (for an already-picked night's swap) |
| Read (single) | N/A | Owned by FEAT-04's Review Swap Suggestions screen; this feature only creates the record, per the Connected Entities scope ("Swap Suggestion (create — for other adult members)") | Cross-feature — FEAT-04 |
| Read (list) | N/A | Same as above | Cross-feature — FEAT-04 |
| Update | N/A | Accept/decline/lapse outcomes are written by FEAT-04, never by this feature | Cross-feature — FEAT-04 |
| Delete/Archive | N/A | No delete — a suggestion always resolves to a terminal outcome and is retained as part of the plan's history (scope-boundaries.md SC-18); owned by FEAT-04, not this feature | Cross-feature — FEAT-04 |
| State Transition | N/A | Owned by FEAT-04 (Suggestion Lifecycle Rules, Suggestion Lapse) | Cross-feature — FEAT-04 |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Recipe | FEAT-23.SPEC-002, FEAT-23.SPEC-003, FEAT-23.SPEC-004 | Candidate recipes for a night, with cook time, rough cost, and safety badge |
| Dietary Rule | FEAT-23.SPEC-004 (via FEAT-02's safety engine) | The hard-rule check every candidate must pass; this feature never stores or displays raw Dietary Rule records itself |
| Household | FEAT-23.SPEC-001 | Weekly budget and schedule shown at the top of the week; aisle/unit/currency display per FEAT-16 (XBR-11) |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Maya opens a week that has not started | Auto-create the Weekly Plan (origin: manually built) with seven empty nights | Inline in triggering screen | FEAT-23.SPEC-001 |
| Maya taps an empty night and picks a recipe | Filter candidates to safe choices, then create the Planned Meal, recalculate estimated_total, signal the grocery list | Standalone Logic/Rule (filtering) + Standalone Automation (write, recalc, signal) | FEAT-23.SPEC-004 / FEAT-23.SPEC-006 |
| Maya changes an already-picked night | Same as above, on the update path | Standalone Logic/Rule + Standalone Automation | FEAT-23.SPEC-004 / FEAT-23.SPEC-006 |
| Maya clears a picked night | Delete the Planned Meal, recalculate estimated_total, signal the grocery list | Standalone Automation | FEAT-23.SPEC-006 |
| A candidate recipe fails the safety check | Mark it ineligible with the specific member and rule it breaks; block placement | Standalone Logic/Rule | FEAT-23.SPEC-004 |
| A candidate recipe has incomplete ingredient data | Exclude it from the candidate list entirely — never shown unchecked (fail-closed, XBR-01) | Standalone Logic/Rule | FEAT-23.SPEC-004 |
| Sam picks a recipe to suggest for a night | Enforce one open suggestion per member per night, run the safety filter, create the Swap Suggestion | Standalone Logic/Rule | FEAT-23.SPEC-005 / FEAT-23.SPEC-004 |
| A pick suggestion is created | Notify the organiser a suggestion is waiting | Cross-feature (FEAT-04.SPEC-006 executes) | FEAT-04 responsibility |
| The organiser accepts a pick suggestion | Apply the pick to the slot; notify the suggester it was accepted | Cross-feature (FEAT-04.SPEC-004 / FEAT-04.SPEC-006 execute) | FEAT-04 responsibility |
| The organiser declines a pick suggestion | Set outcome to Declined; notify the suggester | Cross-feature (FEAT-04.SPEC-003 / FEAT-04.SPEC-006 execute) | FEAT-04 responsibility |
| A pick suggestion's night passes unanswered | Mark it Lapsed, free the slot, notify the suggester | Cross-feature (FEAT-04.SPEC-005 / FEAT-04.SPEC-006 execute) | FEAT-04 responsibility |
| A pick, change, or clear completes | Signal the grocery list to recalculate immediately | Cross-feature (FEAT-06 executes) | FEAT-06 responsibility |
| A household's hard dietary rule tightens mid-week and an already-placed pick now fails | The now-unsafe night reopens for picking with safe alternatives offered | Cross-feature inbound (FEAT-02/FEAT-01 trigger, XBR-02) | FEAT-23.SPEC-001 / FEAT-23.SPEC-002 receive it |
| Saving a pick, change, or clear fails (e.g., dropped connection) | The attempted pick stays on screen with a retry; never silently dropped | Inline in triggering screen | FEAT-23.SPEC-001 / FEAT-23.SPEC-002 |

## Shared Context

**Shared Entities:**
- Planned Meal -- created and updated by SPEC-002 (via SPEC-006), deleted by SPEC-001 (via SPEC-006), read by SPEC-001 and SPEC-002. Fields relevant here: night, recipe, cook_time, rough_cost, safety_badge, status.
- Swap Suggestion -- created only by SPEC-003 (pick suggestions); read, updated, and resolved entirely by FEAT-04, which also creates the meal-swap variant of the same entity.
- Recipe (read-only) -- candidate source for SPEC-002 and SPEC-003, filtered through SPEC-004 before either screen can display it as selectable.

**Shared UI Patterns:**
- Safe-choice recipe picker -- the same browse/search list, the same inline loading indicator (results within about a second), and the same ineligible-with-plain-reason presentation are used by SPEC-002 (organiser pick/change) and SPEC-003 (Sam's suggestion). Spec Writers for both screens should describe this pattern identically; only the terminal action differs (place directly vs. send as a suggestion).
- Week grid with per-night state -- SPEC-001 renders all seven nights with a consistent per-night state model: empty ("nothing planned"), picked, and (via a pending-suggestion badge, cross-feature with FEAT-04) suggested.

**Shared Validation/Logic:**
- FEAT-23.SPEC-004 defines safe-choice filtering; SPEC-002 and SPEC-003 both reference it rather than duplicating the allergy/religious-rule check or the ineligibility-reason logic.
- FEAT-23.SPEC-005 defines the one-dinner-per-night, seven-nights-per-week, one-week-ahead, and one-open-suggestion-per-member-per-night limits; SPEC-001, SPEC-002, SPEC-003, and SPEC-006 all reference it rather than each re-deriving the limits.

## Internal Dependency Map

```
SPEC-001 (Weekly Plan) -> [Maya taps an empty night] -> SPEC-002 (Pick / Change a Recipe)
SPEC-001 (Weekly Plan) -> [Maya taps change on a picked night] -> SPEC-002 (Pick / Change a Recipe)
SPEC-001 (Weekly Plan) -> [Maya taps clear on a picked night] -> SPEC-006 (Apply Manual Pick) [delete path] -> SPEC-001
SPEC-002 (Pick / Change a Recipe) -> [loads candidates] -> SPEC-004 (Safe-Choice Filtering) -> [filtered / ineligible-marked list] -> SPEC-002
SPEC-002 -> [Maya selects a safe recipe] -> SPEC-005 (Validation & Limits) -> [checks pass] -> SPEC-006 (Apply Manual Pick) -> SPEC-001 [updated night]
SPEC-006 (Apply Manual Pick) -> [pick, change, or clear applied] -> FEAT-06 (Shared Grocery List recalculation, cross-feature)
SPEC-003 (Suggest a Pick) -> [Sam browses] -> SPEC-004 (Safe-Choice Filtering) -> [filtered list] -> SPEC-003
SPEC-003 -> [Sam selects a safe recipe] -> SPEC-005 (Validation & Limits) [one-open-suggestion-per-member-per-night check] -> [creates Swap Suggestion] -> FEAT-04 (Swap Suggestion Notifications, cross-feature)
```

**Default Entry:** FEAT-23.SPEC-001 (Weekly Plan) -- the screen shown when a member navigates to this feature area, whether directly, via FEAT-01's setup-complete "pick this week's dinners" choice, or via FEAT-15's first-use landing for a free-tier household. FEAT-23.SPEC-003 (Suggest a Pick) is Sam's entry point, reached the same way but scoped to his Own-only access.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-23.SPEC-001 | Inbound | FEAT-01 (Household Setup & Member Profiles) | Entry into manual planning from the setup-complete confirmation | Choose "pick this week's dinners" on the free tier (First Household Setup, step 6) |
| FEAT-23.SPEC-001 | Inbound | FEAT-15 | A free-tier household's first-use landing routes directly to its current manually built week | Land in context on a free-tier household's plan |
| FEAT-23.SPEC-001 | Inbound (context) | FEAT-01 (Household Setup & Member Profiles) | Displays the household's weekly budget and schedule at the top of the week | Every week view |
| FEAT-23.SPEC-002 | Outbound | FEAT-08 (Recipe Library & Starter Recipes) | Browsing and searching the recipe library for a night's pick | Tap a night, then search (Free-Tier Manual Week, step 2) |
| FEAT-23.SPEC-002, FEAT-23.SPEC-003 | Outbound | FEAT-10 (Recipe Import from Web Link) | The household's imported recipes appear as candidates alongside starter recipes | Every candidate list load |
| FEAT-23.SPEC-004 | Inbound (authority) | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Owns the safety determination every candidate recipe is filtered through (XBR-01) | Every candidate list load |
| FEAT-23.SPEC-001, FEAT-23.SPEC-002 | Inbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | A mid-week hard-rule tightening reopens an already-placed pick for replacement | Mid-week rule change re-check (XBR-02) |
| FEAT-23.SPEC-003 | Outbound | FEAT-04 (One-Tap Meal Swap) | A pick suggestion created here is reviewed, accepted, or declined through FEAT-04's Swap Suggestion flow, and notified through FEAT-04.SPEC-006 | Accept a suggestion (Free-Tier Manual Week, step 3) |
| FEAT-23.SPEC-006 | Outbound | FEAT-06 (Shared Grocery List) | Every pick, change, or clear recalculates the shared grocery list immediately | Pick/change/clear completes (XBR-03) |
| FEAT-23 (feature) | Outbound | FEAT-12 (Meal Rating & Preference Learning) | A manually picked dinner that gets cooked feeds the same rating pipeline as an AI-picked one | Dinner is cooked and rated |
| FEAT-23 (feature) | Outbound | FEAT-13 (Tonight's Dinner Reminder) | A manually picked dinner is included in the nightly reminder the same as an AI-picked one | Reminder computed for the day |
| FEAT-23 (feature) | Inbound | FEAT-14 (Subscription & Billing Management) | Free-tier sign-up and any downgrade route the household to this feature to plan | Free-tier start or downgrade completes (XBR-05) |
| FEAT-23.SPEC-003 (dependency, resolved) | Outbound (dependency) | FEAT-07 (Weekly Plan Ready Notification) | The pick-suggestion notification FEAT-23.SPEC-003 triggers is executed by FEAT-04.SPEC-006, which relies on the device-notification-delivery capability owned by FEAT-07 — resolving the "pending" note against FEAT-23 in feature-dependency-map.md's External Touchpoints | Every pick suggestion created |
| FEAT-23.SPEC-006 (dependency, resolved) | Outbound (dependency) | FEAT-06 (Shared Grocery List) | Live propagation of a manually built plan's changes across household devices relies on the real-time synchronization capability owned by FEAT-06 — resolving the second "pending" note against FEAT-23 in feature-dependency-map.md's External Touchpoints | Every pick, change, or clear |

## Non-Functional Notes

**Data volumes / growth:** Manual Weekly Planning generates zero AI cost by design (scope-boundaries.md SC-16, BRIEF.md Business Context: free tier); its Weekly Plan and Planned Meal records grow at the same per-household weekly rate as the AI-generated path and share the same base — several thousand households, 2-6 members each, in the first year — and are kept for the life of the account (assumptions-constraints.md ASMP-24; scope-boundaries.md SC-18).

**Responsiveness:** Recipe search results for a pick appear within about a second (assumptions-constraints.md ASMP-23; product-features.md States). A pick, change, or clear must feel instant once it reaches the grocery list (assumptions-constraints.md ASMP-22). Success is measured directly: at least 60% of free-tier organisers who start a manual week fill at least five nights and see the grocery list build from them in the same session (success-metrics.md, Manual Week Completion).

**Data sensitivity / privacy:** Weekly Plan and Planned Meal are household personal data — private to the household, never sold or used for advertising (feature-dependency-map.md, Weekly Plan/Planned Meal Data Sensitivity; assumptions-constraints.md ASMP-26). This feature filters every candidate through Dietary Rule data — health-adjacent personal data including children's allergy information, the product's most sensitive data class — but only ever reads it through FEAT-02's safety engine (FEAT-23.SPEC-004); it neither stores nor displays raw Dietary Rule records itself (feature-dependency-map.md, Dietary Rule Data Sensitivity).

**Compliance flags:** The fail-closed safety check this feature enforces on every placement (XBR-01) is the mechanism behind the non-negotiable "zero allergy incidents, ever" success metric (success-metrics.md, Zero Allergy Incidents) — no medical or diet advice is derived from any pick (scope-boundaries.md SC-06). Children's-privacy-class handling governs the Dietary Rule data this feature depends on indirectly (assumptions-constraints.md ASMP-26/27), though FEAT-23 itself processes no children's data directly. The accessibility baseline (ASMP-29) applies directly to the week grid and picker: one-thumb reachable tap targets, and ineligibility reasons conveyed in words, never by color alone.

## Non-Goals

- **AI-generated plans or AI-sourced pick suggestions** -- Excluded per scope-boundaries.md SC-16 and XBR-05: the free tier this feature serves generates zero AI cost; AI plan generation is FEAT-03's paid-tier responsibility, not manual planning's.
- **Sam picking or clearing a night directly** -- Excluded per scope-boundaries.md SC-04 and the Access Matrix (Sam's Manual Planning access is Own-only): other adult members may only suggest a pick, never place or clear one themselves; the organiser accepts or declines every suggestion.
- **Breakfast and full lunch planning** -- Deferred per scope-boundaries.md's deferral notes: the product plans dinners only for MVP, with lunches covered solely by Leftover Rollover to Lunches (FEAT-11); broader meal-scope planning is a Later-phase decision pending evidence of household need.
- **Withdrawing a submitted pick suggestion** -- Not modeled: product-features.md's Primary Flows & Alternates and Validation & Limits define a suggestion's only outcomes as accepted, declined, or lapsed; no withdrawal path is stated, so a member who wants to change a suggestion waits for the organiser's decision or its lapse.
- **Reviewing, accepting, or declining a pick suggestion** -- Owned by FEAT-04 (One-Tap Meal Swap) per feature-dependency-map.md's Swap Suggestion authority column and XBR-06: this feature only creates the suggestion (Connected Entities: "Swap Suggestion (create — for other adult members)").
- **Grocery-list recalculation logic itself** -- Owned by FEAT-06 (Shared Grocery List) per feature-dependency-map.md's authority column and XBR-03: this feature only triggers the recalculation.
- **Planning more than one week ahead** -- Validation & Limits caps planning at one week ahead; this feature offers no path to build or view weeks further out.
- **Automatic purge of a cleared night or of plan history** -- Intentional lifecycle decision surfaced by the CRUD matrix: clearing a night is a hard delete of an unlived slot with no retention concern, while completed-week history is retained for the life of the account (scope-boundaries.md SC-18) and untouched by this feature; neither path is purged.
- **A distinct native-app planning experience** -- Excluded per scope-boundaries.md SC-05: the product ships as a responsive web app for v1, with no native apps.
- **Medical or diet advice in recipe presentation** -- Excluded per scope-boundaries.md SC-06: picks are filtered for safety only and shown with cost, time, and safety information — never ranked or explained in nutritional or health terms.



# Screen Spec: Weekly Plan (Manual Week Builder)

## Overview

**Name:** Weekly Plan (Manual Week Builder)
**ID:** FEAT-23.SPEC-001
**Type:** Screen
**Purpose:** The organiser (and, view-only, other adult members) sees the seven-night week, the running estimated cost against the household's weekly budget, and taps a night to pick, change, or clear it.
**Parent Feature:** FEAT-23 -- Manual Weekly Planning

## Scope and Non-Goals

**In Scope:**
- Displaying all seven nights of the current/next manually built week with each night's state (nothing planned, picked, suggestion pending)
- Auto-initializing the Weekly Plan the first time the organiser opens a week that has not started
- Displaying the week's running estimated_total against the household's weekly_budget
- Routing Maya's taps to pick, change, or clear a night
- Routing Sam's taps to the suggestion flow, and showing the status of his own open suggestions

**Non-Goals:**
- Sam picking, changing, or clearing a night directly -- excluded per scope-boundaries.md SC-04 and the Access Matrix (Sam's Manual Planning access is Own-only); he can only reach the suggestion flow (FEAT-23.SPEC-003)
- Browsing or searching the recipe library itself -- handled by FEAT-23.SPEC-002 (Pick / Change a Recipe), which this screen navigates to
- Reviewing, accepting, or declining a pick suggestion -- owned by FEAT-04 (One-Tap Meal Swap) per feature-dependency-map.md's Swap Suggestion authority column and XBR-06; this screen only surfaces a pending badge that opens FEAT-04's review
- Browsing weeks beyond the current/next week -- excluded per this feature's Validation & Limits (one week ahead) and FEAT-23.SPEC-005; older or farther weeks are Weekly Plan History's (FEAT-19, v1) responsibility

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01 (Household Setup & Member Profiles) -- setup-complete confirmation | Organiser chooses "pick this week's dinners" on the free tier (First Household Setup, step 6) | None -- opens the current/next week to build |
| FEAT-15 (Member Onboarding) -- first-use landing | A free-tier household's first-use landing routes directly to its current manually built week | None -- opens the household's current week |
| Direct navigation (default entry) | Any household member navigates to the planning area | None |
| FEAT-19 (Weekly Plan History, v1) -- past week reused | Organiser copies a past week into a future week | Night picks pre-filled from the selected past week, each subject to this feature's safety re-check (FEAT-23.SPEC-004, XBR-01) before it displays as placed |
| FEAT-03.SPEC-002 (Free-Tier Plan Placeholder & Upgrade Prompt) | Organiser taps "Plan this week by hand" on the free-tier placeholder | None -- opens the current/next week to build |
| FEAT-24.SPEC-002 (Referral Welcome Screen) | A visitor whose own household is on the free tier taps "Go to your plan" | None -- opens the visitor's own current manually built week |
| FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message) | Household member taps the "Tonight: ..." nudge for a manually built week | Scrolled/highlighted to tonight's slot within the current week |
| FEAT-13.SPEC-004 (Same-Day Swap Correction Message) | Household member taps the same-day correction for a manually built week | Scrolled/highlighted to tonight's slot, showing the new dinner |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen: all seven nights, budget total, schedule | Pick an empty night, change or clear a picked night, open a pending suggestion for review | -- |
| Sam (Other Adult Member) | Full screen: all seven nights, budget total, schedule | Tap a night to suggest a pick (FEAT-23.SPEC-002); cannot pick, change, or clear directly | Change/Clear controls are not shown to Sam on any night; a night he taps opens the suggestion flow (FEAT-23.SPEC-003) instead of the picker |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile; the screen is not reachable |
| Jordan (older kid, limited login -- Later) | No | No | Manual Planning access is None for this row; navigation to this screen is not offered |
| Riley (Operator, support -- from v1) | No | No | Manual Planning access is None for Riley; this screen carries no support-view entry point |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on this screen if it was their original destination |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no in-progress edits exist on this screen to preserve (edits happen on FEAT-23.SPEC-002/003), so nothing is lost |

## Layout and Content

**Header:** The week's label (e.g., the calendar dates it covers) alongside the household's running estimated cost for the week ("estimated_total") shown against its configured weekly_budget, in the household's currency (FEAT-16). Below the header, a short line surfaces the household's weekly_schedule where a night is time-constrained (e.g., a 30-minute-weeknight marker on the affected night rows).

**Body:** Seven night rows, Monday through Sunday, each showing:
- The night's day label
- If picked: the recipe name, cook_time, rough_cost, and the "checked against allergies" safety_badge with its "always check labels" disclaimer
- If nothing is planned: a plain "Nothing planned" marker and (Maya only) a "Pick a dinner" action
- If Sam has an open suggestion pending for that night (cross-feature, FEAT-04): a "Suggestion pending" badge, visible to both Maya and Sam
- If the night was reopened because a hard dietary rule tightened mid-week (XBR-02): a "No longer safe -- pick again" marker in place of the prior pick

Maya sees, on a picked night, a "Change" action and a "Clear" action alongside the pick. Sam sees no Change/Clear controls on any night; tapping a night takes him into the suggestion flow.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** The seven night rows stack vertically, full width, in day order. The header's budget total sits directly under the week label.
- **Medium size class and above:** The same seven rows remain vertically stacked (one primary list, no multi-column restructuring); the header's week label and budget total sit side by side instead of stacked.
- **Recipe/cook-time/cost line within a picked-night row:** Wraps to a second line at the compact breakpoint; stays on one line at medium and above.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Empty night row (Maya) | Tap | Navigate to FEAT-23.SPEC-002 (Pick / Change a Recipe) with the target night in context | Screen transitions | Standard navigation transition |
| Picked night's "Change" action (Maya) | Tap | Navigate to FEAT-23.SPEC-002 with the target night and its existing pick pre-loaded | Screen transitions | Standard navigation transition |
| Picked night's "Clear" action (Maya) | Tap | Confirmation dialog appears; on confirm, triggers FEAT-23.SPEC-006 (Apply Manual Pick) delete path for that night | Dialog opens, then closes on confirm | Dialog text: "Remove this dinner from {Night}?" with "Remove" and "Keep It" options; on success the row updates to "Nothing planned" |
| Night row with no open suggestion (Sam) | Tap | Navigate to FEAT-23.SPEC-003 (Suggest a Pick) with the target night in context | Screen transitions | Standard navigation transition |
| Night row with Sam's own open suggestion (Sam) | Tap | Displays the suggestion's status inline; does not navigate | Row expands to show status | Text: "Suggestion sent -- waiting on {Organiser's name}." |
| "Suggestion pending" badge (Maya) | Tap | Navigate to FEAT-04's suggestion review for that suggestion | Screen transitions to FEAT-04 | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Header (week label, budget total) -> night rows in day order (Monday through Sunday) -> each row's available actions (Pick, or Change then Clear, or the pending-suggestion badge) in that order.
- **Dynamic-change announcements:** When a night's state changes (a pick lands, a clear completes, a suggestion badge appears or clears), the updated row content is announced to assistive technology.
- **Confirmation dialog:** The Clear confirmation dialog traps focus while open and is announced on appearance; dismissing it (either option) returns focus to the row's Clear action.
- **Keyboard alternatives:** Every row action (Pick, Change, Clear, suggestion badge) is reachable and operable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (first open) | Seven nights auto-initialize to "Nothing planned"; a prompt invites picking the first dinner | Organiser opens a week that has not started (no existing Weekly Plan for it) | A night is picked, or the organiser navigates away |
| Loading | Each night row shows an inline loading indicator until its data arrives | Screen first opens or is refreshed | Week data (picks, budget total) finishes loading |
| Error | An inline "Couldn't remove -- retry" control appears on the affected night row; the prior pick remains visible | A Clear action's save fails | User taps Retry (succeeds) or navigates away |
| Offline/Degraded | The current week's picks and budget total remain fully visible; Pick, Change, and Clear controls show "Connect to make changes" and are disabled | Connectivity is lost while the screen is open | Connectivity is restored -- controls re-enable immediately |
| Unsafe-Reopened (per night) | The affected night shows "No longer safe -- pick again" in place of its prior recipe, with a path into FEAT-23.SPEC-002 to choose a safe alternative | A household member's hard dietary rule newly fails a placed night's recipe (XBR-02, run by FEAT-02) | The night is re-picked with a safe recipe via FEAT-23.SPEC-002 |

## Validation Rules

Validation governed by FEAT-23.SPEC-005 (Manual Planning Validation & Limits) -- the one-dinner-per-night, seven-nights-per-week, and one-week-ahead rules that bound what this screen can show and offer. Safety eligibility for any recipe placed through this screen's actions is governed by FEAT-23.SPEC-004 (Safe-Choice Filtering & Placement Block).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Tap an empty night (Maya) | FEAT-23.SPEC-002 (Pick / Change a Recipe) | -- |
| Tap "Change" on a picked night (Maya) | FEAT-23.SPEC-002 (Pick / Change a Recipe) | -- |
| Tap a night with no open suggestion (Sam) | FEAT-23.SPEC-003 (Suggest a Pick) | -- |
| Tap the "Suggestion pending" badge (Maya) | Suggestion review | FEAT-04 (One-Tap Meal Swap) |

## Data Model

**Creates:** Weekly Plan -- auto-initialized with origin "manually built" and seven empty night slots the first time the organiser opens a week that has not started.
**Reads:** Weekly Plan (week, estimated_total, over_budget_note); Planned Meal (night, recipe, cook_time, rough_cost, safety_badge, status -- one per planned night); Household (weekly_budget, weekly_schedule, currency, unit_system -- for the header display).
**Updates:** None directly -- estimated_total is recalculated by FEAT-23.SPEC-006 whenever a pick, change, or clear completes; this screen displays the result.
**Deletes:** None directly -- the Clear action triggers the delete path of FEAT-23.SPEC-006.

## Business Rules

- XBR-01: every recipe shown as placed on this screen has already passed the same fail-closed allergy/religious-rule check as every other path onto the plan; the safety badge and "always check labels" disclaimer are shown on every picked night.
- XBR-03: every pick, change, or clear applied from this screen (via FEAT-23.SPEC-006) recalculates the shared grocery list (FEAT-06) immediately -- no member ever sees this week's plan without its matching list.
- XBR-11: the estimated_total and each night's rough_cost display in the household's configured currency and unit system (FEAT-16); a later locale change converts existing figures for display rather than leaving them inconsistent.
- FEAT-23.SPEC-005 governs which weeks and nights this screen can offer for picking (one week ahead, one dinner per night); this screen never offers a way to plan beyond that window.

## Edge Cases

- **Maya has this week open on two devices and clears the same night from both** -- The first clear commits; the second device's clear request is rejected once it arrives, and that device's view refreshes to show the night already empty. Resolution: reject-with-refresh, per the dependency map's Contention note for Planned Meal.
- **A household's hard dietary rule tightens mid-week and an already-placed pick now fails (XBR-02)** -- The affected night immediately shows "No longer safe -- pick again"; Maya is told a meal was removed from the plan (per FEAT-01/FEAT-02 communications) and can pick a safe alternative through FEAT-23.SPEC-002.
- **Maya taps Clear twice rapidly on the same night** -- The second tap has no effect while the first removal is in progress; no duplicate removal request is sent, and the Clear control shows its in-progress state until the first request resolves.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-002 (Pick / Change a Recipe) | Navigation (outbound) | Tapping an empty night or a picked night's Change action opens the picker with the night in context |
| FEAT-23.SPEC-003 (Suggest a Pick) | Navigation (outbound) | Sam tapping a night with no open suggestion opens the suggestion flow |
| FEAT-23.SPEC-006 (Apply Manual Pick) | Triggers (outbound) | The Clear action's confirmed delete path triggers this automation |
| FEAT-23.SPEC-004 (Safe-Choice Filtering & Placement Block) | References (inbound) | Every displayed pick has already passed this spec's safety filter |
| FEAT-23.SPEC-005 (Manual Planning Validation & Limits) | References (inbound) | Governs which week and which nights this screen can display and offer |
| FEAT-04 (One-Tap Meal Swap) | Navigation (outbound) | Tapping a pending-suggestion badge opens FEAT-04's suggestion review |
| FEAT-01 (Household Setup & Member Profiles) | References (inbound) | Supplies the weekly_budget and weekly_schedule shown at the top of the week |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | References (inbound) | Runs the mid-week re-check (XBR-02) that can reopen an already-placed night |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| manual_week_started | week reference, origin ("manually built") | A Weekly Plan is auto-initialized on this screen's first open for a week that has not started | supports success-metrics.md: "Manual Week Completion" |

## Acceptance Criteria

**FEAT-23.SPEC-001-AC-01:** Given Maya opens next week's plan for the first time, when the screen loads, then a Weekly Plan auto-initializes with origin "manually built" and all seven nights show "Nothing planned."

**FEAT-23.SPEC-001-AC-02:** Given Maya is on the Weekly Plan screen, when she taps Wednesday's empty night, then she is taken to FEAT-23.SPEC-002 with Wednesday as the target night.

**FEAT-23.SPEC-001-AC-03:** Given Maya is on the Weekly Plan screen with Friday already picked, when she taps "Change" on Friday, then she is taken to FEAT-23.SPEC-002 with Friday's existing pick pre-loaded.

**FEAT-23.SPEC-001-AC-04:** Given Maya is on the Weekly Plan screen with Monday already picked, when she taps "Clear" on Monday and confirms "Remove," then FEAT-23.SPEC-006's delete path runs and Monday shows "Nothing planned."

**FEAT-23.SPEC-001-AC-05:** Given Sam is on the Weekly Plan screen with no open suggestion for Tuesday, when he taps Tuesday's night row, then he is taken to FEAT-23.SPEC-003 to suggest a pick for Tuesday.

**FEAT-23.SPEC-001-AC-06:** Given Sam already has an open suggestion for Saturday, when he taps Saturday's night row, then the row shows "Suggestion sent -- waiting on Maya" instead of opening the picker.

**FEAT-23.SPEC-001-AC-07:** Given Maya's connection is slow while the week loads, when the screen first renders, then each night row shows an inline loading indicator until its data arrives.

**FEAT-23.SPEC-001-AC-08:** Given Maya's Clear action fails to save, when the failure occurs, then Monday's dinner remains visible with an inline "Couldn't remove -- retry" control, and the dinner is not silently dropped.

**FEAT-23.SPEC-001-AC-09:** Given Maya loses connectivity while viewing the week, when she looks at any night's Pick, Change, or Clear controls, then they show "Connect to make changes" and are disabled, while the week's existing picks and budget total remain fully visible.

**FEAT-23.SPEC-001-AC-10:** Given the household's weekly_budget is set and two dinners are picked, when Maya views the week, then the header shows the running estimated_total against the weekly_budget in the household's configured currency.

**FEAT-23.SPEC-001-AC-11:** Given Maya has this week open on two devices and clears Thursday's dinner on one device, when the same clear is attempted from the second device after the first commits, then the second request is rejected and that device's view refreshes to show Thursday already empty.

**FEAT-23.SPEC-001-AC-12:** Given a household member's allergy is tightened mid-week and Wednesday's placed dinner now fails the check, when Maya opens the week, then Wednesday shows "No longer safe -- pick again" and offers a path to a safe alternative via FEAT-23.SPEC-002.

**FEAT-23.SPEC-001-AC-13:** Given Sam has an open suggestion for Saturday, when Maya taps the "Suggestion pending" badge on Saturday, then she is taken to FEAT-04's suggestion review to accept or decline it.

**FEAT-23.SPEC-001-AC-14:** Given Maya clears Thursday's dinner, when FEAT-23.SPEC-006 completes the removal, then the shared grocery list (FEAT-06) recalculates to remove Thursday's ingredients (XBR-03).

**FEAT-23.SPEC-001-AC-15:** Given the household is currently working on the next plannable week, when Maya looks for a way to plan two weeks ahead from this screen, then no such option exists anywhere on the screen (validation governed by FEAT-23.SPEC-005).

**FEAT-23.SPEC-001-AC-16:** Given Maya taps Clear twice rapidly on Friday's dinner, when the first tap begins removing it, then the second tap has no effect and no duplicate removal request is sent.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 5 (empty/first-open, loading, error, offline, unsafe-reopened) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 3 | 3 |



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



# Logic/Rule Spec: Manual Planning Validation & Limits

## Overview

**Name:** Manual Planning Validation & Limits
**ID:** FEAT-23.SPEC-005
**Type:** Logic/Rule
**Purpose:** Enforces the structural limits of manual weekly planning -- one dinner per night, seven nights per week, planning up to one week ahead, and one open pick suggestion per member per night.
**Parent Feature:** FEAT-23 -- Manual Weekly Planning
**Governed Entity:** Weekly Plan, Planned Meal, and Swap Suggestion (the three entities whose structural limits this feature enforces for manual planning)

## Scope and Non-Goals

**In Scope:**
- The one-dinner-per-night limit on Planned Meal
- The seven-nights-per-week structure of a Weekly Plan
- The one-week-ahead planning window on Weekly Plan.week
- The one-open-pick-suggestion-per-member-per-night limit on Swap Suggestion
- Authorization for create, update, and delete actions on Planned Meal, and create on the pick-suggestion kind of Swap Suggestion, across every role
- Default values applied when a Weekly Plan or Planned Meal is created through manual planning

**Non-Goals:**
- Safety eligibility of a candidate recipe -- owned by FEAT-23.SPEC-004 (Safe-Choice Filtering & Placement Block); this spec governs structural and quantity limits only, never which recipes are safe
- Approval, week-start adoption, or any other Weekly Plan status transition -- owned by FEAT-03 (AI Weekly Dinner Plan Generation) per the Entity-Lifecycle Coverage Matrix; a manually built week enters at "Started" through FEAT-23.SPEC-001's creation and this feature never advances it further
- Reviewing, accepting, declining, or lapsing a Swap Suggestion once created -- owned entirely by FEAT-04 (One-Tap Meal Swap) per XBR-06; this spec governs only the creation-time limit on new pick suggestions

## Governed Entity

**Entities:** Weekly Plan, Planned Meal, Swap Suggestion
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| Weekly Plan.week | date range | The calendar week the plan covers; planning allowed up to one week ahead |
| Weekly Plan.origin | enum | AI-generated or manually built |
| Weekly Plan.status | enum | Generated/Started, Reviewed, Approved, Active, Archived |
| Weekly Plan.estimated_total | number | The week's estimated cost against budget; recalculated by FEAT-23.SPEC-006, not derived here |
| Planned Meal.night | enum (day of week) | At most one dinner per night |
| Planned Meal.recipe | reference | The chosen Recipe (eligibility governed by FEAT-23.SPEC-004, not here) |
| Planned Meal.status | enum | Proposed/Picked, Confirmed, Swapped, Removed, Cooked |
| Swap Suggestion.suggesting_member | reference | The other adult member proposing the pick |
| Swap Suggestion.night | enum (day of week) | The target slot; one open suggestion per member per night |
| Swap Suggestion.proposed_recipe | reference | A safety-checked recipe (governed by FEAT-23.SPEC-004, not here) |
| Swap Suggestion.outcome | enum | Suggested, Accepted, Declined, Lapsed |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-23.SPEC-001 | Weekly Plan | On screen entry (which weeks and nights are offered) and on the Clear action |
| FEAT-23.SPEC-002 | Pick / Change a Recipe | On screen entry (Place vs. Replace availability) and on save |
| FEAT-23.SPEC-003 | Suggest a Pick | On screen entry (whether the screen is reachable for a given night) and on send |
| FEAT-23.SPEC-006 | Apply Manual Pick | During processing, immediately before writing a create, update, or delete to Planned Meal |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Weekly Plan.week | Must be the current week or exactly one week ahead -- no farther | Always | On Weekly Plan creation (FEAT-23.SPEC-001) and before any pick, change, or suggestion is applied to it | "This week is beyond how far ahead you can plan -- build the current or next week first." | Yes |
| Planned Meal.night | A create (place) is only valid when the target night has no existing dinner-status Planned Meal; an existing night's dinner is changed through the update (replace) path, never a second create | Always | On placement attempt (FEAT-23.SPEC-002, FEAT-23.SPEC-006) | "This night already has a dinner -- use Change to replace it." | Yes |
| Weekly Plan (night count) | No count validation beyond the one-dinner-per-night rule above -- the week's seven nights are a structural property (Monday through Sunday), not a quantity a user could exceed | Always | -- | -- | -- |
| Swap Suggestion.night + suggesting_member | At most one Swap Suggestion with outcome "Suggested" per suggesting_member per night | Always, evaluated at creation | On send attempt (FEAT-23.SPEC-003) | "You already suggested a pick for {Night} -- wait for Maya's decision or its lapse." | Yes |
| Swap Suggestion.proposed_recipe | No validation beyond data type in this spec -- recipe eligibility is governed by FEAT-23.SPEC-004 | Always | -- | -- | -- |
| Weekly Plan.origin | No validation beyond data type -- set once at creation (see Defaults and Derivations) | Always | -- | -- | -- |
| Planned Meal.status | No validation beyond data type in this spec -- the value set on manual create is fixed (see Defaults and Derivations); other transitions belong to other features | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Pick and suggestion actions scoped to the plannable week | Weekly Plan.week, Planned Meal.night, Swap Suggestion.night | Placement, change, clear, and suggestion actions are only available for nights belonging to a Weekly Plan whose week is the current or next week; no control for any farther week is ever shown | "This week can't be changed from here." (defensive message, shown only if reached through a stale link to a week beyond the plannable window) |
| Suggestion night must be open | Planned Meal.night (target), Swap Suggestion.night, Swap Suggestion.outcome | A new pick suggestion may target any night, whether empty or already picked; it does not require the night to be empty, since a suggestion is a proposal the organiser may accept in place of an existing pick | N/A -- this is a permissive rule with no rejection case of its own; the one-open-suggestion-per-member-per-night rule above is what blocks a second attempt |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create Planned Meal (place a dinner) | Maya | Target night has no existing dinner-status Planned Meal, the recipe is eligible (FEAT-23.SPEC-004), and the Weekly Plan's week is within the one-week-ahead window | -- |
| Create Planned Meal (place a dinner) | Sam | Never -- placement is the organiser's action alone (scope-boundaries.md SC-04) | No Place action exists in Sam's screens (FEAT-23.SPEC-003) |
| Update Planned Meal (change a dinner) | Maya | Target night has an existing dinner-status Planned Meal, the new recipe is eligible, and the week is within the one-week-ahead window | -- |
| Update Planned Meal (change a dinner) | Sam | Never | No Change action exists in Sam's screens |
| Delete Planned Meal (clear a night) | Maya | Target night has an existing dinner-status Planned Meal | -- |
| Delete Planned Meal (clear a night) | Sam | Never | No Clear action exists in Sam's screens |
| Create Swap Suggestion (pick suggestion) | Sam | Sam has no other open ("Suggested") suggestion for the target night, the proposed recipe is eligible, and the week is within the one-week-ahead window | Ineligible night: the send action is rejected with "You already suggested a pick for {Night} -- wait for Maya's decision or its lapse." |
| Create Swap Suggestion (pick suggestion) | Maya | Never -- the organiser places directly and never suggests to herself | The suggestion screen (FEAT-23.SPEC-003) is not part of her navigation |
| Create/Update/Delete Planned Meal; Create Swap Suggestion | Jordan (young kid profile, no login -- MVP) | Never -- Manual Planning access is None | No login exists for this profile; none of these actions are reachable |
| Create/Update/Delete Planned Meal; Create Swap Suggestion | Jordan (older kid, limited login -- Later) | Never -- Manual Planning access is None | None of these actions are offered to this row |
| Create/Update/Delete Planned Meal; Create Swap Suggestion | Riley (Operator, support -- from v1) | Never -- Manual Planning access is None | None of these actions are offered to Riley |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Weekly Plan.origin | "manually built" | On creation, when FEAT-23.SPEC-001 auto-initializes a week that has not started | No |
| Weekly Plan.status | "Generated/Started" (entering as "Started") | On creation, when FEAT-23.SPEC-001 auto-initializes the week | No -- further transitions (Reviewed, Approved, Active, Archived) are FEAT-03's responsibility and are never applied by this feature |
| Planned Meal.status | "Proposed/Picked" (entering as "Picked") | On creation, when FEAT-23.SPEC-006 writes a new pick | No -- other statuses (Confirmed, Swapped, Removed, Cooked) are set by FEAT-11, FEAT-04, and FEAT-02 respectively |
| Swap Suggestion.outcome | "Suggested" | On creation, when FEAT-23.SPEC-003 sends a pick suggestion | No -- resolved outcomes (Accepted, Declined, Lapsed) are set by FEAT-04 |

## Business Rules

- XBR-06: at most one open suggestion per member per night; an unanswered suggestion lapses when its night passes and the suggester is told the outcome (the lapse mechanics themselves are owned by FEAT-04; this spec defines the creation-time limit that keeps a member from having two open suggestions on the same night at once).
- XBR-05: these limits apply uniformly regardless of subscription tier -- manual planning, and the limits governing it, are available on both the free and paid tiers.
- These limits are independent of recipe safety: a night can be structurally valid to pick (no existing dinner, week in range) and still be blocked from a specific recipe by FEAT-23.SPEC-004's separate safety check; both must pass for a placement or suggestion to succeed.

## Edge Cases

- **Maya attempts to place a pick on a night exactly seven days from today** -- Allowed; this is the boundary of "up to one week ahead," not beyond it.
- **Maya attempts to place a pick on a night eight days from today** -- Blocked with the week-limit error; no control for that night is shown in the first place, since FEAT-23.SPEC-001 only ever displays the current and next week.
- **Sam attempts to create a Swap Suggestion for the same member and night from two devices within the same moment** -- The first request commits; the second is rejected once it arrives, per first-decision-wins with reject-with-refresh (dependency map's Contention note for Swap Suggestion).
- **Maya places a direct pick on a night while Sam has an open suggestion for that same night** -- Per XBR-06 (owned by FEAT-04), the direct pick supersedes the open suggestion, which FEAT-04 then resolves rather than leaving open; this spec's one-open-suggestion-per-member-per-night rule does not block Maya's direct pick, since it governs only new suggestion creation, not the organiser's placement action.
- **A night's existing dinner is cleared and immediately re-picked in the same session** -- Treated as a fresh create; the one-dinner-per-night rule re-evaluates against the now-empty night and always passes.
- **The current week rolls over to become "last week" while a suggestion is still open for a night in it** -- The night has passed; FEAT-04 lapses the suggestion per XBR-06 rather than this spec extending its window.

## Acceptance Criteria

**FEAT-23.SPEC-005-AC-01:** Given Maya is building the current week, when she opens a night exactly seven days from today, then placing a pick on it is allowed.

**FEAT-23.SPEC-005-AC-02:** Given Maya wants to plan a night eight days from today, when she looks at the Weekly Plan screen, then no such night is shown or reachable, per the one-week-ahead window.

**FEAT-23.SPEC-005-AC-03:** Given Wednesday already has a placed dinner, when Maya attempts to place a second dinner on Wednesday through a create action rather than Change, then the attempt is rejected with "This night already has a dinner -- use Change to replace it."

**FEAT-23.SPEC-005-AC-04:** Given Wednesday already has a placed dinner, when Maya uses the Change flow to replace it, then the update succeeds and no one-dinner-per-night violation occurs.

**FEAT-23.SPEC-005-AC-05:** Given Sam already has an open suggestion for Saturday, when he attempts to send a second suggestion for Saturday, then the attempt is rejected with "You already suggested a pick for Saturday -- wait for Maya's decision or its lapse."

**FEAT-23.SPEC-005-AC-06:** Given Sam has no open suggestion for Sunday, when he sends an eligible recipe as a suggestion for Sunday, then the Swap Suggestion is created with outcome "Suggested."

**FEAT-23.SPEC-005-AC-07:** Given Sam (Other Adult Member) is on FEAT-23.SPEC-002 by way of a stale link, when he attempts to place a dinner directly, then no such action is available to him -- placement is never allowed for Sam.

**FEAT-23.SPEC-005-AC-08:** Given Maya (Organiser) attempts to reach the suggestion-send flow, when she does, then no create-suggestion action is available to her -- suggestions are never allowed for Maya.

**FEAT-23.SPEC-005-AC-09:** Given Jordan is a young kid profile with no login, when any attempt is made to place, change, clear, or suggest a pick on Jordan's behalf, then none of these actions is reachable, since Manual Planning access is None for this row.

**FEAT-23.SPEC-005-AC-10:** Given Maya opens a new week that has not started, when the Weekly Plan is auto-initialized, then its origin is set to "manually built" and its status enters as "Started," with no further status transition applied by this feature.

**FEAT-23.SPEC-005-AC-11:** Given Maya places a new pick, when FEAT-23.SPEC-006 writes it, then the Planned Meal's status is set to "Picked."

**FEAT-23.SPEC-005-AC-12:** Given Sam sends a new suggestion, when it is created, then its outcome is set to "Suggested," never any other value.

**FEAT-23.SPEC-005-AC-13:** Given Maya places a direct pick on a night where Sam has an open suggestion, when the placement completes, then Maya's placement succeeds and the open suggestion is resolved by FEAT-04 per XBR-06, rather than this spec's one-open-suggestion rule blocking Maya's action.

**FEAT-23.SPEC-005-AC-14:** Given Sam attempts to create a Swap Suggestion for the same night from two devices at effectively the same time, when the first request commits, then the second request is rejected once it arrives.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 7 | 7 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 10 | 10 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Automation Spec: Apply Manual Pick

## Overview

**Name:** Apply Manual Pick
**ID:** FEAT-23.SPEC-006
**Type:** Automation
**Purpose:** Writes a pick, change, or clear to the night's Planned Meal slot, recalculates the week's estimated total, and signals the shared grocery list to recalculate.
**Parent Feature:** FEAT-23 -- Manual Weekly Planning

## Scope and Non-Goals

**In Scope:**
- Writing a new Planned Meal (create) when a night is picked
- Overwriting an existing Planned Meal's recipe, cook_time, and rough_cost (update) when a night is changed
- Removing a Planned Meal entirely (delete) when a night is cleared
- Recalculating the Weekly Plan's estimated_total after every create, update, or delete
- Signalling the Shared Grocery List (FEAT-06) to recalculate immediately after every create, update, or delete

**Non-Goals:**
- Determining recipe eligibility -- owned by FEAT-23.SPEC-004 (Safe-Choice Filtering & Placement Block); this automation assumes the triggering screen has already confirmed eligibility and re-validates only the structural limits, not safety
- Applying a pick from an accepted Swap Suggestion -- owned by FEAT-04 (One-Tap Meal Swap) per the Side-Effect Inventory's disposition for suggestion acceptance (FEAT-04.SPEC-004 / FEAT-04.SPEC-006 execute that path); this automation's triggers are limited to the direct pick, change, and clear actions on FEAT-23.SPEC-001 and FEAT-23.SPEC-002
- Recalculating the grocery list's contents itself -- owned by FEAT-06 (Shared Grocery List) per feature-dependency-map.md's authority column and XBR-03; this automation only signals that a recalculation is needed

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pick placed | FEAT-23.SPEC-002 (Pick / Change a Recipe) | The target night has no existing dinner-status Planned Meal; the selected recipe passed FEAT-23.SPEC-004's eligibility check; the Weekly Plan's week is within FEAT-23.SPEC-005's one-week-ahead window | Night, recipe (name, cook_time, rough_cost, safety_badge) |
| Recipe changed | FEAT-23.SPEC-002 (Pick / Change a Recipe) | The target night has an existing dinner-status Planned Meal; the newly selected recipe passed FEAT-23.SPEC-004's eligibility check | Night, existing Planned Meal reference, new recipe (name, cook_time, rough_cost, safety_badge) |
| Night cleared | FEAT-23.SPEC-001 (Weekly Plan) | The organiser confirms "Remove" on a picked night's Clear action | Night, existing Planned Meal reference |

## Processing Logic

1. Receive the trigger context: the target night, and either the selected recipe (pick or change) or a clear instruction.
2. For a pick or change: re-confirm the night still satisfies FEAT-23.SPEC-005's structural rules (create requires no existing dinner on the night; change requires an existing one) and that the recipe's eligibility (FEAT-23.SPEC-004) has not lapsed since the triggering screen's last check.
3. If re-confirmation fails (the night's state changed, or the recipe is no longer eligible), stop and return the failure outcome to the triggering screen without writing any data.
4. For a pick: create a new Planned Meal record for the night with status "Picked," and the recipe, cook_time, rough_cost, and safety_badge carried from the selected recipe.
5. For a change: overwrite the existing Planned Meal's recipe, cook_time, rough_cost, and safety_badge with the newly selected recipe's values; the record's night and status are unchanged.
6. For a clear: delete the Planned Meal record for the night entirely.
7. Recalculate the Weekly Plan's estimated_total by summing the rough_cost of every remaining Planned Meal (dinner) in the week.
8. Signal the Shared Grocery List (FEAT-06) that this week's plan has changed, so it recalculates immediately (XBR-03).
9. Determine whether this action brings the organiser's count of filled nights in the current session to five or more for the first time this week; if so, emit the week-completion signal.
10. Return the outcome (success or failure) to the triggering screen.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Pick applied | A new pick's create succeeds | New Planned Meal created (status "Picked"); Weekly Plan.estimated_total recalculated | FEAT-23.SPEC-001 shows the night with its new recipe; FEAT-23.SPEC-002 navigates back to FEAT-23.SPEC-001 | FEAT-23.SPEC-001, FEAT-23.SPEC-002, FEAT-06, FEAT-21.SPEC-003 |
| Change applied | A change's update succeeds | Existing Planned Meal's recipe, cook_time, rough_cost, and safety_badge overwritten; Weekly Plan.estimated_total recalculated | FEAT-23.SPEC-001 shows the night with its new recipe; FEAT-23.SPEC-002 navigates back to FEAT-23.SPEC-001 | FEAT-23.SPEC-001, FEAT-23.SPEC-002, FEAT-06, FEAT-21.SPEC-003 |
| Clear applied | A clear's delete succeeds | Planned Meal removed; Weekly Plan.estimated_total recalculated | FEAT-23.SPEC-001 shows the night as "Nothing planned" | FEAT-23.SPEC-001, FEAT-06 |
| Re-confirmation failed (night state changed) | The night's existing-dinner state no longer matches what the triggering screen expected (e.g., cleared or picked from another device in the interim) | None | FEAT-23.SPEC-002 shows "This night changed while you were choosing -- reload to see the latest pick"; FEAT-23.SPEC-001's Clear shows the night already in its current state | FEAT-23.SPEC-001, FEAT-23.SPEC-002 |
| Re-confirmation failed (recipe no longer eligible) | The selected recipe fails FEAT-23.SPEC-004's check at write time (e.g., a mid-week rule tightened after the screen's last load) | None | FEAT-23.SPEC-002 shows the recipe's ineligibility reason inline; no write occurs | FEAT-23.SPEC-002 |
| Write failure (e.g., dropped connection) | The create, update, or delete cannot be completed for a reason other than eligibility or state conflict | None -- no partial write is left behind | The attempted pick, change, or clear stays visible on the triggering screen with a retry option, never silently dropped | FEAT-23.SPEC-001, FEAT-23.SPEC-002 |

## Data Model

**Reads:** Weekly Plan (week, current estimated_total); Planned Meal (existing record for the target night, when changing or clearing); Recipe (name, cook_time, rough_cost, safety_badge) for the newly selected recipe.
**Creates:** Planned Meal -- night, recipe, cook_time, rough_cost, safety_badge, and status "Picked" (pick trigger only).
**Updates:** Planned Meal -- recipe, cook_time, rough_cost, safety_badge (change trigger only); Weekly Plan.estimated_total (every trigger, recalculated).
**Deletes:** Planned Meal -- the entire record for the target night (clear trigger only).

## Business Rules

- XBR-03: every pick, change, or clear updates the shared grocery list immediately; no member ever sees this week's plan without its matching list.
- FEAT-23.SPEC-005 governs the structural limits (one dinner per night, one week ahead) this automation re-confirms before writing; FEAT-23.SPEC-004 governs the eligibility this automation re-confirms before writing.
- A cleared night's Planned Meal is a hard delete with no restore path -- the organiser simply picks again if she changes her mind, consistent with the Entity-Lifecycle Coverage Matrix's Delete/Archive disposition for Planned Meal under this feature.
- This automation runs synchronously with the triggering screen's save action -- the screen waits for its outcome (success or failure) before completing.

## Edge Cases

- **Maya's pick, change, or clear fails to save (e.g., dropped connection)** -- The attempted action stays visible on the triggering screen with a retry option; never silently dropped, per the Side-Effect Inventory.
- **A hard dietary rule tightens between the triggering screen's last load and this automation's write** -- The write is rejected with the standard ineligibility reason; no Planned Meal is created or updated.
- **The target night's state changes between the triggering screen's last load and this automation's write (e.g., cleared or picked from another device)** -- The write is rejected with "This night changed while you were choosing -- reload to see the latest pick," consistent with reject-with-refresh per the dependency map's Contention note for Planned Meal.
- **A safety-concern removal (FEAT-02) runs concurrently against the same night** -- The safety removal always wins over this automation's concurrent change, per the dependency map's Contention note for Planned Meal; this automation's write is rejected if the removal commits first.
- **Concurrent trigger firing -- Maya picks two different nights from two devices at effectively the same time** -- Each trigger's write proceeds independently against its own night; there is no shared state between different nights, so neither is blocked by the other.
- **A trigger fires for the same night while a previous run for that night is still in flight** -- The triggering screen's Place/Replace/Clear action is disabled while its own save is in progress (per FEAT-23.SPEC-001 and FEAT-23.SPEC-002's Interactions), so a second run for the same night from the same screen cannot start; a second device attempting the same night's write while the first is in flight is handled by the standard state-conflict rejection above once it arrives.
- **Clearing a night that has already been cleared (e.g., a stale Clear button tapped after the night was already cleared elsewhere)** -- No Planned Meal exists to delete; the automation returns the "night already empty" outcome without error, and the triggering screen refreshes to show the night as already "Nothing planned."

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-001 (Weekly Plan) | Triggered by (inbound) | The confirmed Clear action triggers this automation's delete path |
| FEAT-23.SPEC-002 (Pick / Change a Recipe) | Triggered by (inbound) | The Place and Replace actions trigger this automation's create and update paths |
| FEAT-23.SPEC-001 (Weekly Plan) | Affects (outbound) | Displays the resulting night state and the recalculated estimated_total |
| FEAT-23.SPEC-002 (Pick / Change a Recipe) | Affects (outbound) | Displays success navigation or the rejection/error feedback for a failed write |
| FEAT-23.SPEC-004 (Safe-Choice Filtering & Placement Block) | References (inbound) | Re-confirmed at write time before any create or update |
| FEAT-23.SPEC-005 (Manual Planning Validation & Limits) | References (inbound) | Re-confirmed at write time before any create, update, or delete |
| FEAT-06 (Shared Grocery List) | Affects (outbound) | Signalled to recalculate immediately after every successful create, update, or delete |

## Analytics and Success Signals

- **manual_meal_applied** (outcome: created / updated / cleared; night) -- supports success-metrics.md: "Manual Week Completion"
- **manual_week_completed** (nights_filled_count reaching five or more in the session) -- supports success-metrics.md: "Manual Week Completion"
- **manual_pick_write_failed** (reason: state_conflict / eligibility_lapsed / connection_failure) -- N/A -- no success-metrics.md metric measures this automation's failure rate directly; recorded to confirm the "never silently dropped" guarantee is exercised as intended, not to feed a Stage 2 metric
- **grocery_list_recalc_signaled** (night, outcome) -- N/A -- the grocery list's own responsiveness is measured by success-metrics.md: "Grocery List Live-Update Trust," whose Connected Feature is FEAT-06, not this feature; this automation's signal is the trigger for that measurement, not a metric of its own

## Acceptance Criteria

**FEAT-23.SPEC-006-AC-01:** Given Maya has selected an eligible recipe for Wednesday's empty night on FEAT-23.SPEC-002, when she confirms Place, then a new Planned Meal is created with status "Picked" and the Weekly Plan's estimated_total recalculates to include it.

**FEAT-23.SPEC-006-AC-02:** Given Maya has selected a new eligible recipe to replace Friday's existing dinner, when she confirms Replace, then the existing Planned Meal's recipe, cook_time, rough_cost, and safety_badge are overwritten and estimated_total recalculates.

**FEAT-23.SPEC-006-AC-03:** Given Maya confirms "Remove" on Monday's picked dinner, when this automation runs, then the Planned Meal is deleted and estimated_total recalculates to exclude it.

**FEAT-23.SPEC-006-AC-04:** Given a pick, change, or clear completes successfully, when this automation finishes, then the Shared Grocery List (FEAT-06) is signalled to recalculate immediately (XBR-03).

**FEAT-23.SPEC-006-AC-05:** Given Maya's selected recipe passed eligibility on FEAT-23.SPEC-002 but a household member's allergy tightened before this automation writes it, when the automation re-confirms eligibility, then the write is rejected and FEAT-23.SPEC-002 shows the recipe's ineligibility reason.

**FEAT-23.SPEC-006-AC-06:** Given Wednesday's night state changed on another device between FEAT-23.SPEC-002's load and this automation's write, when the write is attempted, then it is rejected with "This night changed while you were choosing -- reload to see the latest pick."

**FEAT-23.SPEC-006-AC-07:** Given a network failure occurs during this automation's write, when the failure happens, then no partial write is left behind and the attempted pick, change, or clear stays visible on the triggering screen with a retry option.

**FEAT-23.SPEC-006-AC-08:** Given a safety-concern removal (FEAT-02) commits against the same night at effectively the same time as this automation's change, when both are in flight, then the safety removal wins and this automation's write is rejected.

**FEAT-23.SPEC-006-AC-09:** Given Maya picks two different nights from two devices at effectively the same time, when both writes run, then each completes independently against its own night with no blocking between them.

**FEAT-23.SPEC-006-AC-10:** Given Maya has already filled four nights this session and successfully picks a fifth, when this automation completes that fifth pick, then a manual_week_completed signal is emitted, supporting success-metrics.md: "Manual Week Completion."

**FEAT-23.SPEC-006-AC-11:** Given Maya taps Clear on a night that was already cleared from another device moments earlier, when this automation runs, then it returns the "night already empty" outcome without error and the screen refreshes to show the night as "Nothing planned."

**FEAT-23.SPEC-006-AC-12:** Given a pick, change, or clear is being saved from FEAT-23.SPEC-001 or FEAT-23.SPEC-002, when the triggering screen's own action control is in its loading state, then a second trigger for the same night from that same screen cannot start until the first completes.

**FEAT-23.SPEC-006-AC-13:** Given Maya applies a pick, change, or clear, when this automation completes any outcome, then no attempt is ever made to apply an accepted Swap Suggestion through this automation -- that path is owned entirely by FEAT-04.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (pick, change, clear) | 3 |
| Outcome Paths | 6 (pick applied, change applied, clear applied, state-conflict rejection, eligibility-lapsed rejection, write failure) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |
