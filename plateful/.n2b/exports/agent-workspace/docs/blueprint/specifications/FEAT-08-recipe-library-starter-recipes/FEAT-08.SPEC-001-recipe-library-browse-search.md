---
document_type: spec
spec_type: screen
spec_id: FEAT-08.SPEC-001
spec_name: Recipe Library Browse & Search
spec_slug: recipe-library-browse-search
parent_feature: FEAT-08
parent_feature_name: Recipe Library (Starter Recipes)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: Recipe Library Browse & Search

## Overview

**Name:** Recipe Library Browse & Search
**ID:** FEAT-08.SPEC-001
**Type:** Screen
**Purpose:** Household member browses, searches, and filters the household's full recipe pool (starter plus imported) and picks a recipe to open.
**Parent Feature:** FEAT-08 -- Recipe Library (Starter Recipes)

## Scope and Non-Goals

**In Scope:**
- Listing the household's combined starter-plus-imported recipe pool, page-sized with infinite scroll
- Searching by recipe name or ingredient, and filtering by dietary badge
- Showing a consistent recipe card/row summary (name, cook time, rough cost, dietary badges) for every result
- Surfacing an ineligible recipe with a plain explanation, per FEAT-08.SPEC-003, instead of hiding it
- The entry point into importing a recipe from a link (hand-off only; the import flow itself belongs to FEAT-10)

**Non-Goals:**
- Bulk import of recipe collections -- excluded per scope-boundaries.md SC-12: the household's recipe pool grows only through this feature's starter seed and FEAT-10's one-link-at-a-time import; no bulk-import path exists.
- Nutrition, calorie, or diet-quality scoring or filtering -- excluded per scope-boundaries.md SC-06: the library shows cost, time, and safety information only, never a health score.
- Rating or review actions on a recipe from this screen -- ratings are captured against cooked Planned Meals by Meal Rating & Preference Learning (FEAT-12); adding a rating control here would duplicate that mechanism.
- Editing or authoring recipe content from this screen -- starter recipes are read-only for households (maintained by FEAT-08.SPEC-004); editing an imported recipe is FEAT-10's Recipe Edit surface, not this one.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Default entry (app navigation) | Household member navigates to the recipe library directly | None -- list opens to the first page, no active search |
| FEAT-23.SPEC (Manual Weekly Planning, a night in next week) | Household member taps a night, then searches | Target night the pick will fill; on selecting a recipe, control returns to Manual Weekly Planning with that night filled |
| FEAT-04 (One-Tap Meal Swap, safe alternatives list) | Household member searches the library directly for a recipe not offered as a swap alternative | None -- a direct search for a specific recipe; if the recipe is ineligible, FEAT-08.SPEC-003 governs its display |
| FEAT-08.SPEC-002 (Recipe Detail View) | Back navigation from a recipe opened from this screen | Prior search query and filter selections, restored exactly as left |
| FEAT-10.SPEC-004 (Edit Imported Recipe) | Member removes an imported recipe | None -- list opens without the removed recipe |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Search, filter, open any recipe, tap "import from link" | -- |
| Sam (Other Adult Member) | Full screen | Search, filter, open any recipe, tap "import from link" | -- |
| Jordan (young kid profile, no login -- MVP) | None -- no login exists for this profile | None | The profile has no sign-in; there is no session in which this screen could open |
| Jordan (older kid, limited login -- Later) | Full screen | Search, filter, open any recipe | "Import from link" is not shown to this role (Recipe Library access is View, not Full) |
| Riley (Operator, support -- from v1) | Full screen, only while an open Support Request for the household is active (per FEAT-22, XBR-14) | Search, filter, open any recipe -- read-only | Attempting "import from link" (not shown) or any change is not possible; Riley's Recipe Library access is View only |
| Unauthenticated | No | No | Redirected to the sign-in screen; no household data is shown |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- the current search query and filter selections are held locally and restored once sign-in succeeds |

## Layout and Content

**Header:** Screen title "Recipe Library" with a search input (placeholder "Search by name or ingredient") directly below it, and an "Import from link" action to its right (hidden for roles without Full Recipe Library access, per the Access and Visibility table above).

**Filter row:** Below the search input, a row of dietary-badge filter chips (one per badge the household's own dietary rules make relevant, e.g., "Vegetarian," "Nut-free," plus a "Show all" chip that clears active filters). Chips are multi-select.

**Body:** A single-column, scrollable list of recipe cards, one per matching recipe, in the order returned by the current search/filter. Each card shows:
- Recipe name
- Cook time
- Rough cost, in the household's currency (Household entity, currency field, XBR-11)
- Dietary badges, worded and styled per FEAT-02.SPEC-008's badge treatment (never color-only, ASMP-29)
- For an ineligible recipe (per FEAT-08.SPEC-003): the same card layout, with the dietary-badge area replaced by the plain ineligibility reason text

Cards load a page at a time; scrolling to the bottom of the loaded list loads the next page (infinite scroll, ASMP-23 page sizing).

**Footer:** None.

### Responsive Behavior

- **Compact size class (phone):** Single-column card list as described above, full width. Search input and filter chips remain visible at the top; the chip row scrolls horizontally if it does not fit the screen width.
- **Medium size class and above:** Card list remains single-column, capped at a consistent platform-wide content width and horizontally centered; the filter chip row wraps to a second line instead of scrolling horizontally.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Screen open | (automatic) | Requests the first page of the household's combined starter-plus-imported recipe pool; emits recipe_library_opened | Screen enters Loading (initial); card list populates once the first page arrives | Inline loading indicator/placeholder shown in place of the card list while the first page loads |
| Search input | Type | Recomputes matching results against name and ingredient text within about a second (ASMP-23); emits recipe_searched | List updates to matching results; No Results state shown if nothing matches | Inline loading indicator while recomputing; result count updates |
| Filter chip | Tap (select) | Adds the badge to the active filter set and recomputes results | Chip shows selected state; list updates | Result count updates |
| Filter chip | Tap (deselect) | Removes the badge from the active filter set and recomputes results | Chip returns to unselected state; list updates | Result count updates |
| "Show all" chip | Tap | Clears every active filter | All chips return to unselected state; full search results (unfiltered by badge) show | List returns to unfiltered result set |
| Recipe card (eligible) | Tap | Navigate to FEAT-08.SPEC-002 (Recipe Detail View) with this recipe and the current search context | Screen transitions | Recipe Detail View opens |
| Recipe card (ineligible, per FEAT-08.SPEC-003) | Tap | Navigate to FEAT-08.SPEC-002 (Recipe Detail View), which renders the same ineligibility explanation in full | Screen transitions | Recipe Detail View opens showing the ineligibility reason |
| "Import from link" (Maya, Sam only) | Tap | Hand off to FEAT-10 (Recipe Import from Web Link) | Screen transitions out of this feature | FEAT-10's import screen opens |
| Recipe list (scroll to bottom) | Scroll | Loads the next page of results | List appends the next page | Brief inline loading indicator at the list's bottom edge |

### Accessibility Notes

- **Focus order:** Search input -> filter chip row (left to right) -> recipe card list (top to bottom) -> "Import from link" action.
- **Search announcements:** When search results update, the new result count is announced to assistive technology (e.g., "12 recipes found").
- **No Results announcement:** The "nothing found" message is announced when it appears.
- **Ineligible card labeling:** An ineligible recipe's card exposes its plain-language reason as the accessible description for that card, not conveyed through icon or color alone (ASMP-29).
- **Keyboard alternatives:** Filter chips and recipe cards are reachable and activatable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | N/A -- the starter library ships pre-populated at launch (FEAT-08.SPEC-004) and combines with any imported recipes, so this screen's unfiltered result list is never empty for any household | Not applicable | Not applicable |
| Loading (initial) | Header, search input, and filter chip row render immediately; the body shows an inline loading indicator/placeholder in place of the card list, distinct from the pagination indicator below | Screen opens and the first page of the household's recipe pool is requested | First page load completes -- success moves to Populated (default); failure moves to Error |
| Populated (default) | First page of the household's combined starter-plus-imported pool, no active search or filter | Initial load completes successfully with no prior search context | User enters a search term or selects a filter |
| Searching/Filtering | Result list reflects the active query and filters, with an inline loading indicator while recomputing | User types in search or selects/deselects a filter | Recompute completes |
| No Results | Plain "nothing found" message with a suggestion to broaden the search, never a blank screen | A search or filter combination matches nothing | User changes the search term or clears a filter |
| Loading (pagination) | Existing results remain visible; a brief inline loading indicator appears at the bottom of the list | User scrolls to the bottom of a partially loaded result set | Next page finishes loading and appends |
| Error | Inline error message with a retry option; previously loaded results remain visible | A search, filter, or pagination request fails | User taps Retry and the request succeeds |
| Offline/Degraded | Previously viewed recipes and the last-loaded result page remain visible; the search input and filter chips are disabled with the message "Search needs a connection. Reconnect to search the library." | Connectivity lost while the screen is open | Connectivity restored -- search and filters re-enable automatically |

## Validation Rules

**Option B -- Inline (for simple validations not warranting a standalone spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Search input | No minimum length; free text | On change | None -- an empty search input returns the full unfiltered list (Populated state) |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Recipe card tap (eligible or ineligible) | FEAT-08.SPEC-002 (Recipe Detail View) | -- |
| "Import from link" tap | Recipe Import from Web Link entry screen | FEAT-10 (Recipe Import from Web Link) |

## Data Model

**Creates:** None.
**Reads:** Recipe -- name, cook_time, rough_cost, dietary_badges (computed live by FEAT-02 against the household's rules), origin, for every recipe in the household's combined starter-plus-imported pool. Household -- currency and unit_system, to render rough_cost and, where shown, ingredient quantities in the household's configured units (XBR-11).
**Updates:** None.
**Deletes:** None.

## Business Rules

- FEAT-08.SPEC-003 (Ineligible Recipe Search Disclosure Rule) governs whether an ineligible recipe is shown with a plain explanation or excluded on this screen -- this screen defers to it rather than deciding independently.
- XBR-01: every recipe shown here has already passed, or been disclosed as failing, the app-enforced allergy and religious-rule check; no recipe is ever shown unchecked.
- XBR-11: rough cost and any displayed ingredient quantities use the household's configured currency and unit system, applied consistently with every other screen that shows recipe cost or quantities.
- Dietary badge wording, the "checked against allergies" disclaimer, and ineligibility phrasing reuse FEAT-02.SPEC-008's exact treatment rather than composing new wording.

## Edge Cases

- **Search returns no matches** -- Plain "nothing found" message with a suggestion to broaden the search; the search input and active filters remain visible so the user can adjust them directly.
- **User double-taps a recipe card** -- The second tap is ignored while navigation to FEAT-08.SPEC-002 is already in progress.
- **A recipe is corrected or retired by FEAT-08.SPEC-004 while this screen is open** -- The currently loaded list is not live-updated; the change is reflected the next time the list loads or the search re-runs (page refresh or new search), consistent with FEAT-08.SPEC-004's "reflects on next load" behavior. No conflict scenario applies, since this screen never writes to the Recipe entity.
- **User navigates away mid-search and returns via back navigation from FEAT-08.SPEC-002** -- The prior search query, active filters, and scroll position are restored exactly as left.
- **User loses connectivity mid-scroll (pagination in progress)** -- The in-flight page request is treated as a failed load; already-loaded pages remain visible, and the Offline/Degraded state's messaging appears; pagination resumes automatically once connectivity returns.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-002 (Recipe Detail View) | Navigation (outbound) | Tapping any recipe card, eligible or ineligible, opens the recipe's full detail |
| FEAT-08.SPEC-002 (Recipe Detail View) | Navigation (inbound) | Back navigation from detail returns here with search context preserved |
| FEAT-08.SPEC-003 (Ineligible Recipe Search Disclosure Rule) | References (inbound) | Governs whether an ineligible recipe appears with an explanation or is excluded |
| FEAT-08.SPEC-004 (Starter Recipe Content Seeding & Maintenance) | Affects (inbound) | Content created, corrected, or retired by this integration is reflected here on next load |
| FEAT-10 (Recipe Import from Web Link) | Navigation (outbound) | "Import from link" hands off to FEAT-10's import flow |
| FEAT-10 (Recipe Import from Web Link) | Affects (inbound) | A newly saved imported recipe appears in this screen's result pool alongside starter recipes |
| FEAT-23 (Manual Weekly Planning) | Navigation (inbound) | This screen's search and pick list is reused to fill a night by hand |
| FEAT-04 (One-Tap Meal Swap) | Navigation (inbound) | Entry point when a household member searches directly for a recipe not offered as a swap alternative |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| recipe_library_opened | entry source (default nav, manual-planning pick, swap-alternatives search) | Screen opens | N/A -- no Stage 2 metric measures library-open frequency directly; retained to observe entry-point mix |
| recipe_searched | query length, active filter count, result count | Search input changes or a filter is toggled, after results recompute | N/A -- Recipe Library Coverage at Launch (the feature's connected metric) is measured through the seeded corpus's breadth, fed by FEAT-08.SPEC-004, not by individual search actions |
| recipe_search_empty | query text, active filter count | A search or filter combination returns zero results | N/A -- no Stage 2 metric tracks empty-search rate; retained to observe corpus gaps a household actually hits |
| recipe_ineligible_explained | reason category (allergy, religious rule, incomplete data) | An ineligible recipe's card renders on this screen, per FEAT-08.SPEC-003 | N/A -- Zero Allergy Incidents (success-metrics.md) measures whether an unsafe meal ever reaches a plan, not disclosure-on-search events; this event is diagnostic only |

## Acceptance Criteria

**FEAT-08.SPEC-001-AC-01:** Given Maya is on the Recipe Library screen with the default (Populated) view, when she types "chicken" into the search input, then the list recomputes within about a second to show only recipes matching "chicken" by name or ingredient, and the result count updates.

**FEAT-08.SPEC-001-AC-02:** Given Sam is on the Recipe Library screen, when he selects the "Vegetarian" filter chip, then the list recomputes to show only recipes carrying the vegetarian dietary badge, and the chip shows a selected state.

**FEAT-08.SPEC-001-AC-03:** Given Maya searches for a recipe with no matches, when the search completes, then the No Results state appears with a plain "nothing found" message and a suggestion to broaden the search.

**FEAT-08.SPEC-001-AC-04:** Given Maya taps an eligible recipe's card, when the tap registers, then FEAT-08.SPEC-002 (Recipe Detail View) opens for that recipe.

**FEAT-08.SPEC-001-AC-05:** Given Sam searches directly for a recipe that fails his household's allergy rule, when the search returns that recipe, then its card shows the plain ineligibility reason in place of dietary badges, per FEAT-08.SPEC-003, rather than omitting the recipe.

**FEAT-08.SPEC-001-AC-06:** Given the older-kid limited-login role (Later) is on this screen, when they look for the "Import from link" action, then it is not shown, since this role's Recipe Library access is View, not Full.

**FEAT-08.SPEC-001-AC-07:** Given Riley (Operator) is viewing this screen against a household with an open Support Request, when Riley searches the library, then results appear read-only with no "Import from link" action and no way to change any recipe.

**FEAT-08.SPEC-001-AC-08:** Given an unauthenticated visitor attempts to open this screen, when the attempt is made, then they are redirected to the sign-in screen and no recipe data is shown.

**FEAT-08.SPEC-001-AC-09:** Given Maya's session expires while she is searching, when the expiry is detected, then a dialog reads "Your session has expired. Sign in to continue." and her search query and filters are restored after she signs back in.

**FEAT-08.SPEC-001-AC-10:** Given Maya loses connectivity while on this screen, when connectivity drops, then the search input and filter chips disable with the message "Search needs a connection. Reconnect to search the library." while previously loaded results remain visible.

**FEAT-08.SPEC-001-AC-11:** Given Maya has an active search and filter set, when she opens a recipe and then taps back, then she returns to this screen with her exact prior search query, filters, and scroll position restored.

**FEAT-08.SPEC-001-AC-12:** Given Sam scrolls to the bottom of a partially loaded result list, when the scroll reaches the bottom, then the next page of results loads and appends with a brief inline loading indicator.

**FEAT-08.SPEC-001-AC-13:** Given a search request fails due to a transient error, when the failure occurs, then an inline error message with a retry option appears while previously loaded results remain visible.

**FEAT-08.SPEC-001-AC-14:** Given Maya navigates to the Recipe Library for the first time in a session, when the screen opens, then it enters the Loading (initial) state with an inline loading indicator in place of the card list, and the first page of her household's combined starter-plus-imported pool populates once the request completes.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 8 (empty (N/A), loading (initial), populated, searching/filtering, no results, loading (pagination), error, offline/degraded) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
