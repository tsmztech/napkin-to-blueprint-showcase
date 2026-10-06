---
document_type: spec
spec_type: screen
spec_id: FEAT-19.SPEC-002
spec_name: Past Week Detail View
spec_slug: past-week-detail-view
parent_feature: FEAT-19
parent_feature_name: Weekly Plan History
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: Past Week Detail View

## Overview

**Name:** Past Week Detail View
**ID:** FEAT-19.SPEC-002
**Type:** Screen
**Purpose:** Household member views a single past week's full archived plan and grocery list, with the option for Maya to reuse the week as a starting point for a future week.
**Parent Feature:** FEAT-19 -- Weekly Plan History

## Scope and Non-Goals

**In Scope:**
- Displaying the full archived Weekly Plan for a selected week: every archived Planned Meal (dinner and leftover-lunch), each with its recipe name, dietary badges, cook time, and rough cost
- Displaying the full archived Grocery List for the same week, aisle-grouped, exactly as it stood when the week was archived
- Opening recipe detail for any archived meal
- Presenting the "Reuse this week" control to Maya only, and launching FEAT-19.SPEC-003 (Past Plan Reuse) when she picks a target future week

**Non-Goals:**
- Editing any field of the archived week's contents -- excluded because the Weekly Plan lifecycle treats a week as Archived once it ends (feature-dependency-map.md, Weekly Plan lifecycle) and this feature's Primary Flows only support copying a past week into a future week, never modifying the historical record itself (product-features.md, Primary Flows & Alternates)
- Ticking or unticking archived Grocery List Items -- excluded for the same reason: the archived Grocery List is a read-only historical record, not a live list; a fresh, tickable list is built only once a reused week is edited in Manual Weekly Planning (FEAT-23), per FEAT-06's ownership of Grocery List recalculation
- Performing the reuse copy itself -- handled entirely by FEAT-19.SPEC-003 (Past Plan Reuse); this screen only launches that automation and hands off once the target week is chosen

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-19.SPEC-001 (Weekly Plan History Browse) | User taps a past-week list-item | The selected week's identifier |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen -- archived plan, archived grocery list, and recipe detail | Open recipe detail; tap "Reuse this week" and pick a target future week, launching FEAT-19.SPEC-003 | -- |
| Sam (Other Adult Member) | Full screen -- archived plan, archived grocery list, and recipe detail | Open recipe detail only; the "Reuse this week" control is not shown to this role | -- |
| Jordan (older kid, limited login -- Later) | Full screen -- archived plan, archived grocery list, and recipe detail | Open recipe detail only; the "Reuse this week" control is not shown to this role | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile; there is no screen for it to reach |
| Riley (Operator, support) | Full screen -- archived plan, archived grocery list, and recipe detail, per Weekly Plan View access (user-persona.md Access Matrix) | View only; the "Reuse this week" control is not shown to this role, consistent with Riley never changing household data (feature-dependency-map.md, XBR-14) | -- |
| Unauthenticated | No | No | Redirected to the sign-in screen; no detail content is shown before or during redirect |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." appears; on re-authentication the user returns to FEAT-19.SPEC-001 (Weekly Plan History Browse) rather than directly back into this detail view |

## Layout and Content

**Header:** Screen title showing the week identifier (e.g., "Week of Mar 3") with a back arrow (returns to FEAT-19.SPEC-001) and, for Maya only, a "Reuse this week" action button (right-aligned).

**Body, top section -- Archived Plan:** A seven-night list, one row per night, each showing: the night's name, the recipe name (tap opens recipe detail), the "checked against allergies" safety badge and "always check labels" disclaimer carried from the archived record, cook time, and rough cost. A night with no meal picked that week shows a "nothing planned" marker consistent with the shared empty/loading/error state language (feature-overview.md, Shared UI Patterns). Any leftover-lunch Planned Meal linked to a dinner appears directly beneath its source dinner's row.

**Body, middle section -- Week Summary:** The week's estimated total cost against the household's budget at the time, carried from the archived Weekly Plan's estimated_total field.

**Body, bottom section -- Archived Grocery List:** The full aisle-grouped list as it stood when archived, read-only: each line shows ingredient name, combined quantity and unit, and aisle, with no tick controls.

**Footer:** None -- "Reuse this week" is in the header.

### Responsive Behavior

- **Compact size class:** Plan, week summary, and grocery list sections stack vertically in the single-column order described above, full width.
- **Medium size class and above:** Plan section and Grocery List section lay out side by side as two columns, with the Week Summary spanning the full width between the header and the two-column area; no other structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-19.SPEC-001 (Weekly Plan History Browse) | Screen closes | Animated transition back to the list |
| Recipe name (any night) | Tap | Open recipe detail (FEAT-08.SPEC-002 Recipe Detail View) for the archived recipe | Screen transitions to recipe detail | Animated transition to recipe detail screen |
| "Reuse this week" button (Maya only) | Tap | Present a target-future-week picker | Picker overlay appears | Picker shows the available future weeks Maya may choose |
| Target-week picker option | Tap (select a future week) | Launch FEAT-19.SPEC-003 (Past Plan Reuse) with this past week as source and the selected week as target | Picker closes; reuse automation begins | Brief inline "Copying this week..." indicator while the automation runs |
| Target-week picker (Cancel) | Tap | Dismiss the picker without launching reuse | Picker closes | Detail screen remains unchanged |

### Accessibility Notes

- **Focus order:** Back arrow -> "Reuse this week" button (Maya only) -> each night row in order (Sunday through Saturday, with any linked leftover-lunch row immediately after its source) -> week summary -> each grocery list line in aisle order.
- **Dynamic announcements:** The "Copying this week..." indicator and its eventual success or failure outcome (handed off to FEAT-19.SPEC-003) are announced to assistive technology when they appear.
- **Keyboard alternatives:** Every recipe name and the "Reuse this week" button are independently focusable and activatable via keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | N/A -- this screen is only ever reached by selecting a real past-week list-item from FEAT-19.SPEC-001, so a source week with archived plan data always exists; there is no zero-data variant of this screen to render | N/A | N/A |
| Loading | Brief inline loading indicator in place of the plan and list sections, per the feature's couple-second loading budget (feature-overview.md, Non-Functional Notes) | Screen first opens, before the selected week's archived plan and list return | Load succeeds or fails |
| Populated | Full archived plan, week summary, and grocery list as described in Layout and Content | Load succeeds | User navigates away |
| Error | Error banner "Couldn't load this week. Try again." with a Retry button, in place of the plan and list sections | Load of the selected week's archived data fails | User taps Retry and the load succeeds |
| Offline/Degraded | If this week was previously viewed in this session, its cached plan and list remain visible with a "You're offline -- showing a previously viewed version" banner; "Reuse this week" is disabled with the inline note "Reusing a week needs a connection" while offline | Connectivity is lost while this screen is open, or the screen is opened offline for a week not previously cached this session | Connectivity is restored -- banner clears and "Reuse this week" re-enables for Maya |
| Recipe removed from household pool | The affected night's recipe name shows "Recipe no longer available" in place of a tappable link, with cook time and cost still shown from the archived record; the safety badge remains as archived | A recipe referenced in this archived week has since been removed from the household's recipe pool (FEAT-10) | N/A -- this is a permanent state for this archived record once the recipe is gone |

## Validation Rules

N/A -- this screen has no user input fields beyond the target-future-week picker selection, whose valid range (available future weeks) is governed by FEAT-19.SPEC-003 (Past Plan Reuse) and FEAT-23 (Manual Weekly Planning)'s planning-horizon rule (up to one week ahead, per product-features.md, Validation & Limits).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-19.SPEC-001 (Weekly Plan History Browse) | -- |
| Recipe name tap | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 (Recipe Library) |
| "Reuse this week" -> pick target week | FEAT-19.SPEC-003 (Past Plan Reuse), then on completion to Manual Weekly Planning's week view | FEAT-23 (Manual Weekly Planning) |

## Data Model

**Creates:** None on this screen directly -- the reuse action's copy is created by FEAT-19.SPEC-003 once launched from here.
**Reads:** Weekly Plan (archived) -- week, status, estimated_total, and every archived Planned Meal (night, recipe, safety_badge, cook_time, rough_cost, meal_kind, status). Grocery List (archived) -- aisle_grouping and every archived Grocery List Item (ingredient_name, quantity_and_unit, aisle). Recipe -- name, dietary_badges, and current pool-membership status (to detect a since-removed recipe), read within the plan display. All fields per the definitions in feature-dependency-map.md.
**Updates:** None -- this screen never writes to the archived record.
**Deletes:** None.

## Business Rules

- Reuse-control visibility (whether "Reuse this week" appears at all) is governed by FEAT-19.SPEC-004 (History Access & Reuse Authorization) -- this screen does not re-derive the rule independently (feature-overview.md, Shared Validation).
- XBR-01 (feature-dependency-map.md): the safety badge shown on each archived meal reflects the check performed when that meal was live; reusing the week re-runs the check under today's rules via FEAT-19.SPEC-003, not on this display screen.
- A recipe removed from the household's pool since this week was archived is shown per the "Recipe removed from household pool" state above, consistent with this feature's Entity-Lifecycle Coverage Matrix note on FEAT-10's ownership of recipe removal.

## Edge Cases

- **Concurrent-edit conflict** -- N/A -- this screen only reads the archived Weekly Plan and Grocery List, which are never updated in place once archived (feature-dependency-map.md: Weekly Plan is "Archived when the week ends"; this feature's Entity-Lifecycle Coverage Matrix states its Connected Entities are read-only); there is no live write path here for another user's change to race against.
- **Two household members tap "Reuse this week" for the same past week into different target future weeks at the same time** -- N/A for Sam and the older-kid login, since the control is not shown to them; for Maya, only one active session can hold the control, so no concurrent reuse launch from this screen is possible for the same account.
- **Maya taps "Reuse this week" then immediately taps Back before picking a target week** -- The picker is dismissed and no reuse automation launches; the detail screen closes normally.
- **A night's leftover-lunch link references a source dinner that was itself swapped multiple times before archiving** -- The archived record shows only the final swap_history-resolved recipe that was active at archive time, consistent with how the live plan displayed it.
- **User navigates directly to this screen's URL for a week identifier from an unauthorized role or another household** -- Treated as the Unauthorized Experience defined in Access and Visibility: an unauthenticated or wrong-household request is redirected to sign-in with no detail content shown; no week data is ever returned for a household the requester does not belong to.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-19.SPEC-001 (Weekly Plan History Browse) | Navigation (inbound) | User arrives here from a selected list-item |
| FEAT-19.SPEC-003 (Past Plan Reuse) | Triggers (outbound) | "Reuse this week" launches the reuse automation with this week as source |
| FEAT-19.SPEC-004 (History Access & Reuse Authorization) | References (inbound) | Governs detail visibility and reuse-control visibility |
| FEAT-08.SPEC-002 (Recipe Detail View) | Navigation (outbound) | Recipe name tap opens recipe detail |
| FEAT-06.SPEC-* (Shared Grocery List) | References (inbound, cross-feature) | Source of the archived Grocery List data displayed here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| plan_history_week_detail_viewed | week identifier, meal count, whether any recipe shows "no longer available" | Screen load succeeds | N/A -- no entry in success-metrics.md connects to Weekly Plan History (FEAT-19); this event is retained as an operational signal so the gap is visible rather than silently dropped, per Phase 2.5 Category 8 |
| past_plan_reused | source week identifier, target week identifier | Maya picks a target future week and the reuse automation (FEAT-19.SPEC-003) is launched | N/A -- no entry in success-metrics.md connects to Weekly Plan History (FEAT-19); see plan_history_week_detail_viewed |

## Acceptance Criteria

**FEAT-19.SPEC-002-AC-01:** Given Maya opens a past week's detail from the history list, when the screen loads, then she sees all seven nights with their recipes, safety badges, cook time, and cost, plus the archived grocery list grouped by aisle.

**FEAT-19.SPEC-002-AC-02:** Given Maya is viewing a past week's detail, when she taps a recipe name, then the screen navigates to that recipe's detail view.

**FEAT-19.SPEC-002-AC-03:** Given Maya is viewing a past week's detail, when she looks at the header, then she sees a "Reuse this week" button.

**FEAT-19.SPEC-002-AC-04:** Given Sam is viewing a past week's detail, when he looks at the header, then no "Reuse this week" button is shown.

**FEAT-19.SPEC-002-AC-05:** Given Jordan (older kid, limited login) is viewing a past week's detail, when he looks at the header, then no "Reuse this week" button is shown.

**FEAT-19.SPEC-002-AC-06:** Given Maya taps "Reuse this week" and picks a target future week, when the selection is made, then FEAT-19.SPEC-003 (Past Plan Reuse) launches with this week as the source and the picked week as the target.

**FEAT-19.SPEC-002-AC-07:** Given Maya taps "Reuse this week" and then taps Cancel in the target-week picker, when Cancel is tapped, then the picker closes and no reuse automation launches.

**FEAT-19.SPEC-002-AC-08:** Given the selected week's archived data has not yet returned, when the screen first opens, then a brief inline loading indicator appears in place of the plan and list sections.

**FEAT-19.SPEC-002-AC-09:** Given the selected week's archived data fails to load, when the error appears, then Maya sees "Couldn't load this week. Try again." with a Retry button.

**FEAT-19.SPEC-002-AC-10:** Given Maya loses connectivity while viewing a previously cached week's detail, when connectivity drops, then the cached plan and list remain visible with an offline banner, and "Reuse this week" is disabled with the note "Reusing a week needs a connection."

**FEAT-19.SPEC-002-AC-11:** Given an archived week references a recipe that has since been removed from the household's pool, when Maya views that night, then it shows "Recipe no longer available" in place of a tappable recipe name, while cook time, cost, and the archived safety badge still display.

**FEAT-19.SPEC-002-AC-12:** Given a night in the archived week had no meal picked that week, when Maya views the plan, then that night shows a "nothing planned" marker.

**FEAT-19.SPEC-002-AC-13:** Given Riley (Operator, support) views a past week's detail during an open support request, when the screen loads, then she sees the full archived plan and list read-only, with no "Reuse this week" control shown.

**FEAT-19.SPEC-002-AC-14:** Given Maya's session expires while viewing a past week's detail, when she next interacts with the screen, then a dialog reads "Your session has expired. Sign in to continue." and, after re-authenticating, she lands back on FEAT-19.SPEC-001 (Weekly Plan History Browse) rather than directly on this detail screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 (empty, loading, populated, error, offline, recipe-removed) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
