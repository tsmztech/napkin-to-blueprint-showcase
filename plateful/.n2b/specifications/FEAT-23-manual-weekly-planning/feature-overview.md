---
document_type: feature-overview
feature_number: FEAT-23
feature_name: Manual Weekly Planning
feature_slug: manual-weekly-planning
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 6
screen_count: 3
automation_count: 1
logic_rule_count: 2
integration_count: 0
notification_count: 0
---

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
