# FEAT-08 — Recipe Library (Starter Recipes)

This chapter covers FEAT-08, Recipe Library (Starter Recipes), a Core-tier feature. It contains 4 specifications carrying 54 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-08.SPEC-001 | Recipe Library Browse & Search | screen | 14 |
| FEAT-08.SPEC-002 | Recipe Detail View | screen | 14 |
| FEAT-08.SPEC-003 | Ineligible Recipe Search Disclosure Rule | logic-rule | 14 |
| FEAT-08.SPEC-004 | Starter Recipe Content Seeding & Maintenance | integration | 12 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Recipe Library (Starter Recipes)

## Summary

**Feature:** Recipe Library (Starter Recipes)
**ID:** FEAT-08
**Description:** A built-in library of starter recipes the AI plan draws from and that household members can browse directly.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** The brief names "a starter recipe library" explicitly as part of the recipe sources the product needs (BRIEF.md, Ecosystem & Integrations). Without a starter library, plan generation would have nothing to propose from day one, before any household has imported recipes of their own.

**Key Capabilities:**
- Browse recipes — Household member looks through the library directly, outside the weekly plan
- View recipe detail — Household member sees ingredients, steps, cook time, rough cost, and dietary badges for one recipe
- Search and filter — Household member finds recipes by name, ingredient, or dietary badge

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-08.SPEC-001 | Recipe Library Browse & Search | Screen | Maya, Sam, Jordan (older kid, Later), Riley | Household member browses, searches, and filters the household's full recipe pool (starter plus imported) and picks a recipe to open |
| FEAT-08.SPEC-002 | Recipe Detail View | Screen | Maya, Sam, Jordan (older kid, Later), Riley | Household member views one recipe's ingredients, steps, cook time, rough cost, and dietary badges, whether opened from the library or cross-referenced from the plan |
| FEAT-08.SPEC-003 | Ineligible Recipe Search Disclosure Rule | Logic/Rule | Maya, Sam, Jordan (older kid, Later), Riley | Governs when a recipe that fails a household's dietary/allergy rules, or whose ingredient data cannot be fully verified, is still surfaced with a plain ineligibility explanation on a direct library search, instead of being silently excluded as it is everywhere else on the plan |
| FEAT-08.SPEC-004 | Starter Recipe Content Seeding & Maintenance | Integration | All | Creates, corrects, and retires the shared starter-recipe content through the product's recipe/food-content data capability, so the library ships pre-populated and stays complete enough for the safety check |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Browse recipes | FEAT-08.SPEC-001 | Primary purpose of the browse screen — household member looks through the pool outside the weekly plan | Phase 2 (Explicit) |
| View recipe detail | FEAT-08.SPEC-002 | Primary purpose of the detail screen — ingredients, steps, cook time, rough cost, and dietary badges for one recipe | Phase 2 (Explicit) |
| Search and filter | FEAT-08.SPEC-001 | Search-by-name/ingredient/badge and filter controls live on the browse screen, co-occurring naturally with browsing per the "minimal screens" heuristic | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — specs not directly tied to a Key Capability, surfaced by Phases 3–6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-08.SPEC-003 | Ineligible Recipe Search Disclosure Rule | Phase 5 (Rule Discovery) | The Primary Flows & Alternates field states an ineligible recipe is shown with a plain explanation when a household member searches for it directly, rather than the silent exclusion used on every other candidate path — a conditional disclosure rule distinct from FEAT-02's own blocking determination, called out by synthesis check 14 (Allergy-Safe Swap Recovery journey) |
| FEAT-08.SPEC-004 | Starter Recipe Content Seeding & Maintenance | Phase 4 (External Dependencies lens) | assumptions-constraints.md (ASMP-34) names a recipe/food-content data capability this feature depends on to seed the starter library; the Architect Notes confirm FEAT-08 owns this ingestion, resolving the External Touchpoints row that the FEAT-02 Brief left pending |

## Entity-Lifecycle Coverage Matrix

**Entity: Recipe** (starter content — this feature's slice of the shared Recipe entity; household-imported recipes are FEAT-10's slice)

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-08.SPEC-004 | Starter Recipe Content Seeding & Maintenance creates starter Recipe records (name, ingredients with quantity/unit, steps, cook_time, rough_cost, origin = starter library, no owning_household) through the recipe/food-content data capability, populating the library before any household exists | Household-imported recipes are created by FEAT-10, not this feature |
| Read (single) | FEAT-08.SPEC-002 | Recipe Detail View loads one recipe's full content, whether reached from the library or cross-referenced from another feature | -- |
| Read (list) | FEAT-08.SPEC-001 | Recipe Library Browse & Search lists and searches the household's combined starter-plus-imported pool, page-sized with infinite scroll | -- |
| Update | FEAT-08.SPEC-004 | The same seeding capability delivers corrections to existing starter content (e.g., an ingredient correction or cost refresh) over time; starter recipes have no household-facing edit surface -- they are read-only for households (per the feature's own Access field) | Household edits to their own imported recipes are FEAT-10's responsibility |
| Delete/Archive | FEAT-08.SPEC-004 | Soft archive: a retired starter recipe is hidden from browse/search and dropped from the candidate pool for future plan generation; no cascade to Planned Meals that already used it (their history stands unchanged); no restore path (re-adding equivalent content is a fresh seed action, not an undo); no purge — retirement is a permanent content-maintenance decision, never a user-initiated or scheduled deletion | Distinct from FEAT-10's household-side "removed from the household's pool" delete, which never applies to starter content |
| State Transition | N/A | The Recipe entity's functional fields (product-features.md Domain Entity Inventory) give starter content no lifecycle state beyond active/archived, matching the Delete/Archive row above; dietary_badges are computed fresh by FEAT-02 at view time rather than stored as a state, so a content correction or a household rule change is reflected immediately without a separate re-check trigger | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Household | FEAT-08.SPEC-001, FEAT-08.SPEC-002 | Both screens read the household's currency and unit system to display rough_cost and ingredient quantities in the household's own units (XBR-11) |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Household member opens the library | Load and display the household's combined starter-plus-imported recipe pool, first page | Inline in triggering screen | FEAT-08.SPEC-001 |
| Household member enters or changes a search/filter | Recompute matching results within about a second (ASMP-23); emit recipe_searched | Inline in triggering screen | FEAT-08.SPEC-001 |
| A search returns no matches | Show a plain "nothing found" message with a suggestion to broaden the search, never an empty white screen; emit recipe_search_empty | Inline in triggering screen (No Results state) | FEAT-08.SPEC-001 |
| Household member opens a recipe (from the library or cross-referenced from the plan) | Load ingredients, steps, cook time, rough cost, and request the current dietary badge for this household; emit recipe_detail_viewed | Inline in triggering screen | FEAT-08.SPEC-002 |
| A recipe's detail load fails | Offer a retry without losing the search context (Error state) | Inline in triggering screen | FEAT-08.SPEC-002 |
| A household member searches for or opens a recipe that fails their household's dietary/allergy rules, or whose ingredient data can't be fully verified | Surface the recipe with a plain, non-blocking explanation of why it isn't eligible for this household, instead of excluding it silently; the recipe still cannot be added to the plan; emit recipe_ineligible_explained | Standalone Logic/Rule | FEAT-08.SPEC-003 |
| The library or a connected household has no live connectivity | Previously viewed recipes remain available (Offline/Degraded state); new searches require connectivity and are disabled with an explanation rather than failing silently | Inline in triggering screens | FEAT-08.SPEC-001 / FEAT-08.SPEC-002 |
| The recipe/food-content data capability delivers new, corrected, or retired starter content | Create, correct, or retire the affected Recipe records; browse, search, and detail views reflect the change on next load, with no separate re-check needed since badges compute live | Standalone Integration | FEAT-08.SPEC-004 |
| Household member taps "import from link" from the library | Hand off to Recipe Import from Web Link | Cross-feature — logged in touchpoints | FEAT-10 responsibility for the import flow itself |
| A household member saves a reviewed imported recipe (FEAT-10) | The recipe appears in this household's library alongside starter recipes, with its allergy badge already checked | Cross-feature — logged in touchpoints | FEAT-10 responsibility for the save; FEAT-08.SPEC-001 for its subsequent display |

## Shared Context

**Shared Entities:**
- Recipe (starter content) — created, corrected, and retired by SPEC-004; listed and searched by SPEC-001; read in full by SPEC-002; surfaced under the disclosure rule by SPEC-003 when ineligible. Fields this feature owns for starter content: name, ingredients (quantity + unit), steps, cook_time, rough_cost, origin (= starter library), owning_household (= none for starter content), prep_requirements. dietary_badges is never written by this feature — it is computed by FEAT-02 at view time and only displayed here.
- Household (read-only) — currency and unit_system are read by SPEC-001 and SPEC-002 to render rough_cost and ingredient quantities consistently (XBR-11); owned and written by FEAT-01/FEAT-16.

**Shared UI Patterns:**
- Recipe card/row summary — SPEC-001 shows a consistent summary (name, cook time, rough cost, dietary badges) for every recipe in a result list; SPEC-002 expands the same recipe into full detail. Spec Writers for both screens should describe the summary fields identically so the transition from list to detail feels seamless.
- Plain ineligibility reason — this feature reuses the exact phrasing pattern FEAT-02.SPEC-008 defines ("contains {allergen} — not safe for {member}"); SPEC-002 and SPEC-003 render it rather than composing their own wording.
- Safety badge treatment — SPEC-002 (and the summary rows in SPEC-001) reuse FEAT-02.SPEC-008's badge wording, disclaimer, and non-color-only presentation (ASMP-29) rather than restyling it.

**Shared Validation:**
- SPEC-003 (Ineligible Recipe Search Disclosure Rule) is the single source of truth, within this feature, for when an ineligible recipe is shown-with-explanation versus omitted; SPEC-001 and SPEC-002 both defer to it rather than each deciding independently. The underlying pass/fail determination itself remains FEAT-02's authority (XBR-01) — SPEC-003 governs disclosure, not the determination.

## Internal Dependency Map

```
SPEC-001 (Recipe Library Browse & Search) -> [user taps a recipe] -> SPEC-002 (Recipe Detail View)
SPEC-002 (Recipe Detail View) -> [back navigation, search context preserved] -> SPEC-001 (Recipe Library Browse & Search)
SPEC-001 (Recipe Library Browse & Search) -> [search/filter matches an ineligible recipe] -> SPEC-003 (Ineligible Recipe Search Disclosure Rule)
SPEC-002 (Recipe Detail View) -> [opened recipe is ineligible for this household] -> SPEC-003 (Ineligible Recipe Search Disclosure Rule)
SPEC-004 (Starter Recipe Content Seeding & Maintenance) -> [creates, corrects, or retires content] -> SPEC-001 (Recipe Library Browse & Search)
SPEC-004 (Starter Recipe Content Seeding & Maintenance) -> [creates, corrects, or retires content] -> SPEC-002 (Recipe Detail View)
SPEC-001 (Recipe Library Browse & Search) -> [reads currency/units from] -> Household (FEAT-01/FEAT-16, cross-feature)
SPEC-002 (Recipe Detail View) -> [reads currency/units from] -> Household (FEAT-01/FEAT-16, cross-feature)
```

**Default Entry:** SPEC-001 (Recipe Library Browse & Search) — the screen shown when a household member navigates to the library directly; the feature is also entered at SPEC-002 when a recipe is opened by cross-reference from another feature.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-08.SPEC-001 | Outbound | FEAT-10 (Recipe Import from Web Link) | The library's "import from link" entry point starts the import journey (Recipe Import & Pantry Update, step 1) | Household member taps "import from link" from the library |
| FEAT-08.SPEC-001 | Inbound | FEAT-10 (Recipe Import from Web Link) | A newly saved imported recipe appears in the household's library alongside starter recipes, badge already checked (Recipe Import & Pantry Update, step 3) | Household member saves a reviewed imported recipe |
| FEAT-08.SPEC-002 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Tapping a planned meal opens the same detail view as browsing the library directly | Household member taps a meal on the weekly plan |
| FEAT-08.SPEC-001 | Inbound | FEAT-23 (Manual Weekly Planning) | The library's search and pick list is reused to fill a night by hand (Free-Tier Manual Week, step 2) | Household member taps a night, then searches |
| FEAT-08.SPEC-001 / FEAT-08.SPEC-003 | Inbound | FEAT-04 (One-Tap Meal Swap) | A household member searches the library directly for a recipe not offered as a swap alternative and gets the ineligibility explanation (Allergy-Safe Swap Recovery, failure variant) | Recipe wasn't offered among safe alternatives |
| FEAT-08.SPEC-001 / FEAT-08.SPEC-002 / FEAT-08.SPEC-003 | Inbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Dietary badges, the "checked against allergies" disclaimer, and every ineligibility reason shown in the library are computed and worded by FEAT-02 (XBR-01, FEAT-02.SPEC-008); this feature only displays them | Every time a recipe is browsed or opened |
| FEAT-08.SPEC-004 | Outbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Complete ingredient data seeded by this Integration is the precondition FEAT-02's safety check requires to pass a starter recipe rather than fail it closed | Safety check runs against any starter recipe |
| FEAT-08.SPEC-001 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation), FEAT-23 (Manual Weekly Planning) | This feature's browse/search results are the candidate pool and pick list both features draw from | Plan generation runs, or a household member picks manually |

## Non-Functional Notes

**Data volumes / growth:** The starter corpus is a single shared, read-only pool (not per-household), so its size does not scale with the several-thousand-household, 2–6-member-per-household growth described in ASMP-24; that growth instead drives concurrent browse/search load across households, which must stay equally responsive as the household base grows. Success-metrics.md's Recipe Library Coverage at Launch target (no repeated dinners, at least one recipe per major cuisine style for a household with typical dietary rules) sets the corpus's required breadth, which FEAT-08.SPEC-004 must satisfy before launch.

**Responsiveness:** Recipe search results appear within about a second (ASMP-23); a failed detail-view load offers a retry without losing the search context. Every primary action (search, filter, open a recipe) is reachable with one thumb and uses large tap targets, and badges and ineligibility reasons are conveyed in words, never by color alone (ASMP-29).

**Data sensitivity / privacy:** Recipe content itself carries no personal data (dependency map, Recipe entity, Data Sensitivity: None). The dietary badges and ineligibility reasons this feature displays are FEAT-02's derived output, not the household's raw allergy or religious-rule data — this feature never reads or stores Dietary Rule records directly, so no health-adjacent data passes through it.

**Compliance flags:** N/A — no compliance regime applies to static, non-personal recipe content; the localization requirement this feature carries (household currency, unit system, and consistent quantity display, ASMP-28/XBR-11) is a functional consistency rule, not a compliance obligation, and is handled by reading Household settings rather than by any regulatory control.

## Non-Goals

- **Nutrition, calorie, or diet-quality scoring** — Excluded per scope-boundaries.md (SC-06): the library shows cost, time, and safety information, never a health score or diet-advice ranking; BRIEF.md states plainly "there is no medical or diet advice."
- **Bulk import of recipe collections into the library** — Excluded per scope-boundaries.md (SC-12): the household's own recipe pool grows only through the starter seed (this feature) and FEAT-10's one-link-at-a-time import; no bulk-import path exists for households to add their own content in quantity.
- **Cross-household recipe sharing, public recipe feeds, or public profiles** — Excluded per scope-boundaries.md (SC-09): the library is per-household content (starter plus that household's own imports); the brief confirms "otherwise the product stands alone," with growth handled by household-to-household invite links (FEAT-24), not a social recipe layer.
- **From-scratch recipe authoring within the library** — Adjacency exclusion: the feature's Connected Entities line names only "create — starter content, read" for FEAT-08, and BRIEF.md's Ecosystem & Integrations names only the starter library and per-link import as recipe sources; a manual ingredient/step authoring surface is not a capability either document establishes, and adding one would introduce an authoring flow the product definition never asked for.
- **Recipe rating or review actions from the browse/detail screens** — Adjacency exclusion: ratings are captured against cooked Planned Meals by Meal Rating & Preference Learning (FEAT-12), which feeds learned soft dislikes back into Dietary Rule; adding a rating action to library browsing would duplicate that mechanism and blur which feature owns preference learning.



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



# Screen Spec: Recipe Detail View

## Overview

**Name:** Recipe Detail View
**ID:** FEAT-08.SPEC-002
**Type:** Screen
**Purpose:** Household member views one recipe's ingredients, steps, cook time, rough cost, and dietary badges, whether opened from the library or cross-referenced from another feature.
**Parent Feature:** FEAT-08 -- Recipe Library (Starter Recipes)

## Scope and Non-Goals

**In Scope:**
- Displaying one recipe's full content: ingredients (with quantity and unit), steps, cook time, rough cost, and dietary badges
- Requesting and rendering the current dietary badge or ineligibility explanation for this household, computed live by FEAT-02
- Serving as the single detail view reached both from browsing the library directly and from cross-reference by other features (e.g., a planned meal)
- Offering a retry, without losing the search context, when the detail load fails

**Non-Goals:**
- Adding a rating or review for this recipe -- ratings are captured against cooked Planned Meals by Meal Rating & Preference Learning (FEAT-12), prompted from the plan rather than the recipe detail; adding a rating control here would duplicate that mechanism.
- Editing recipe content directly on this screen -- this view never edits a field or saves a change itself; for imported recipes owned by the household, an "Edit" control (Maya and Sam only) navigates to FEAT-10.SPEC-004, which owns the actual edit form, validation, and save. Starter recipes show no edit control at all, since they are read-only for households (corrections flow through FEAT-08.SPEC-004's content maintenance, not a household-facing edit path).
- Nutrition, calorie, or diet-quality information -- excluded per scope-boundaries.md SC-06: this screen shows cost, time, and safety information only.
- Picking this recipe for a specific plan slot directly from this screen -- picking a night's dinner is Manual Weekly Planning's (FEAT-23) pick-list action; this screen is a read-only detail view regardless of entry point.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-08.SPEC-001 (Recipe Library Browse & Search) | User taps a recipe card | The recipe's identity; the prior search context is retained for back navigation |
| FEAT-03.SPEC-001 (Weekly Plan View) | Household member taps a dinner card on the weekly plan | The recipe referenced by that Planned Meal |
| FEAT-23 (Manual Weekly Planning) | Household member opens a recipe while building the week by hand | The recipe's identity; the current pick-list search context is retained for back navigation |
| FEAT-08.SPEC-003 (Ineligible Recipe Search Disclosure Rule) | A recipe surfaced under the disclosure rule is opened | The recipe's identity, plus its ineligibility reason to render in full |
| FEAT-10.SPEC-002 (Review Extracted Recipe) | Member saves an imported recipe, or taps "View existing recipe" when a duplicate is found | The saved (or existing duplicate) recipe's identity |
| FEAT-10.SPEC-003 (Manual Recipe Entry) | Member saves a manually entered recipe, or taps "View existing recipe" when a duplicate is found | The saved (or existing duplicate) recipe's identity |
| FEAT-10.SPEC-004 (Edit Imported Recipe) | Member saves an edit to an imported recipe, or taps the back arrow with no unsaved changes | The edited recipe's identity |
| FEAT-19.SPEC-002 (Past Week Detail View) | Household member taps a recipe name in a past week | The recipe's identity |
| FEAT-02.SPEC-014 (Safety Concern Resolution Notice) | Household member taps "View recipe" on the resolution notice | The reviewed recipe's identity |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | For an imported recipe owned by the household: Edit (navigates to FEAT-10.SPEC-004, per that spec's Access and Visibility). For a starter recipe: view only -- no edit or add-to-plan controls exist | -- |
| Sam (Other Adult Member) | Full screen | For an imported recipe owned by the household: Edit (navigates to FEAT-10.SPEC-004, per that spec's Access and Visibility). For a starter recipe: view only -- no edit or add-to-plan controls exist | -- |
| Jordan (young kid profile, no login -- MVP) | None -- no login exists for this profile | None | The profile has no sign-in; there is no session in which this screen could open |
| Jordan (older kid, limited login -- Later) | Full screen | View only | -- |
| Riley (Operator, support -- from v1) | Full screen, only while an open Support Request for the household is active (per FEAT-22, XBR-14) | View only | -- |
| Unauthenticated | No | No | Redirected to the sign-in screen; no recipe data is shown |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- the recipe being viewed and the return path (library search context or plan/pick-list context) are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Recipe name, with a back arrow that returns to the entry point spec with its context preserved (library search, plan, or pick-list, per Entry Points above). When the opened recipe's origin is imported and belongs to the viewer's household, an "Edit" action button appears in the header (right-aligned), visible only to Maya and Sam per Access and Visibility. Starter recipes, and imported recipes viewed by any other role or state, show no "Edit" button in the header.

**Body, in order:**
- **Summary row:** Cook time, rough cost (in the household's currency, XBR-11), and dietary badges -- worded and styled per FEAT-02.SPEC-008's badge treatment, matching the same fields shown in the FEAT-08.SPEC-001 card summary so the transition from list to detail feels seamless. Where the recipe is ineligible for this household (per FEAT-08.SPEC-003), this row instead shows the plain ineligibility explanation, in full, in place of the dietary badges.
- **Ingredients section:** A list of every ingredient with its quantity and unit, sized and displayed in the household's configured unit system (XBR-11).
- **Steps section:** The recipe's method, presented as an ordered list.

**Footer:** None.

### Responsive Behavior

- **Compact size class (phone):** Single-column layout as described above, full width, sections stacked vertically in the order listed.
- **Medium size class and above:** Content column capped at a consistent platform-wide reading width and horizontally centered; the summary row, ingredients, and steps remain stacked in the same order with no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the entry-point spec with its prior context preserved | Screen closes | Transition back to the entry point |
| Screen open | (automatic) | Loads ingredients, steps, cook time, rough cost, and requests the current dietary badge or ineligibility explanation for this household; emits recipe_detail_viewed | Content populates once the load completes | Loading state shown while the request is in flight |
| Retry button (Error state only) | Tap | Re-issues the failed detail load | Screen re-enters Loading state | Content populates on success; error persists with a fresh retry option on repeated failure |
| "Edit" button (imported recipes owned by the household, Maya and Sam only) | Tap | Navigate to FEAT-10.SPEC-004 (Edit Imported Recipe), carrying the Recipe record's ID and current field values | Screen closes | Transition to Edit Imported Recipe |

### Accessibility Notes

- **Focus order:** Back arrow -> "Edit" button (when shown) -> summary row (cook time, cost, badges or ineligibility text) -> ingredients section -> steps section.
- **Load announcements:** When content finishes loading, the recipe name is announced to assistive technology as the new screen context.
- **Ineligibility announcement:** The ineligibility explanation, when shown, is announced as part of the summary row's content, conveyed in words rather than through icon or color alone (ASMP-29).
- **Keyboard alternatives:** Back arrow, "Edit" (when shown), and Retry are reachable and activatable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | N/A -- this screen always opens against exactly one recipe identity carried from its entry point (FEAT-08.SPEC-001, FEAT-03, FEAT-23, or FEAT-08.SPEC-003); there is no listing or collection here that could render with zero items, so no empty-state UI exists for this screen | Not applicable | Not applicable |
| Loading | Header shows the recipe name (carried from the entry point); body shows an inline loading indicator in place of content | Screen opens | Detail load completes (success or failure) |
| Loaded (eligible) | Full content: summary row with dietary badges, ingredients, steps | Detail load succeeds and the recipe is eligible for this household | User navigates away |
| Loaded (ineligible) | Full content: summary row shows the plain ineligibility explanation instead of dietary badges (per FEAT-08.SPEC-003); ingredients and steps still display | Detail load succeeds and the recipe fails this household's dietary/allergy rules or has incomplete ingredient data | User navigates away |
| Error | Error message with a retry option; the header still shows the recipe name and the entry-point context (e.g., prior search) is retained | Detail load fails | User taps Retry and the load succeeds |
| Offline/Degraded | If this recipe was previously viewed, its last-loaded content remains fully available; if not previously viewed, the Error state's messaging appears with a connectivity-specific note: "This recipe hasn't been viewed yet and needs a connection to load." | Connectivity lost while opening or viewing this screen | Connectivity restored -- a fresh load succeeds automatically for a not-yet-viewed recipe |

## Validation Rules

**Option B -- Inline (for simple validations not warranting a standalone spec):**

This screen accepts no user input; there are no fields to validate.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap (from library) | FEAT-08.SPEC-001 (Recipe Library Browse & Search) | -- |
| Back arrow tap (from plan) | Weekly plan view | FEAT-03 (AI Weekly Dinner Plan Generation) |
| Back arrow tap (from pick-list) | Manual planning pick list | FEAT-23 (Manual Weekly Planning) |
| "Edit" tap (imported recipe owned by the household, Maya or Sam) | FEAT-10.SPEC-004 (Edit Imported Recipe) | FEAT-10 (Recipe Import from Web Link) |

## Data Model

**Creates:** None.
**Reads:** Recipe -- name, ingredients (quantity and unit), steps, cook_time, rough_cost, dietary_badges (computed live by FEAT-02 against the household's rules; ineligibility reason wording governed by FEAT-08.SPEC-003 when applicable). Household -- currency and unit_system, to render rough_cost and ingredient quantities consistently (XBR-11).
**Updates:** None.
**Deletes:** None.

## Business Rules

- FEAT-08.SPEC-003 (Ineligible Recipe Search Disclosure Rule) governs whether this screen renders dietary badges or a plain ineligibility explanation for the opened recipe -- this screen defers to it rather than composing its own wording.
- The plain ineligibility reason and the safety badge reuse FEAT-02.SPEC-008's exact phrasing pattern ("contains {allergen} -- not safe for {member}") and badge treatment, never composed independently.
- XBR-01: the pass/fail safety determination this screen displays is FEAT-02's authority; this screen only requests and renders it.
- XBR-11: rough cost and ingredient quantities use the household's configured currency and unit system, consistent with FEAT-08.SPEC-001 and every other screen that shows recipe cost or quantities.
- An "Edit" control appears on this screen only when the opened recipe's origin is imported and the recipe is owned by the viewing household, and only for Maya and Sam (Access and Visibility); it navigates to FEAT-10.SPEC-004, which owns the actual edit and removal behavior. Starter recipes never show an edit control, since they are read-only for households, per the Recipe entity's lifecycle in feature-dependency-map.md ("households edit their own imported recipes; starter recipes are read-only for households").

## Edge Cases

- **Detail load fails** -- Error state shows a retry option; the entry-point context (e.g., the library search that led here) is not lost, so a successful retry or a back navigation returns the user exactly where they left off.
- **User taps Retry twice rapidly** -- The second tap is ignored while the first retry request is in flight.
- **A starter recipe is corrected by FEAT-08.SPEC-004 while this screen is open** -- The currently displayed content is not live-updated; the correction is reflected the next time this screen is opened for the recipe, since badges and content are computed and loaded fresh on each open. No conflict scenario applies, since this screen never writes to the Recipe entity.
- **User opens this screen for a recipe that has since been retired (soft-archived) by FEAT-08.SPEC-004** -- If reached via a stale link or cross-reference, the recipe's last-known content still loads (archived recipes are retained, not deleted); the recipe no longer appears in FEAT-08.SPEC-001's search results going forward.
- **User navigates directly to this screen (deep link) without a prior library search** -- The back arrow returns to FEAT-08.SPEC-001 with no search context (default Populated state) rather than failing.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-001 (Recipe Library Browse & Search) | Navigation (inbound) | User arrives by tapping a recipe card |
| FEAT-08.SPEC-001 (Recipe Library Browse & Search) | Navigation (outbound) | Back navigation returns here with search context preserved |
| FEAT-08.SPEC-003 (Ineligible Recipe Search Disclosure Rule) | References (inbound) | Governs whether this screen shows dietary badges or a plain ineligibility explanation |
| FEAT-08.SPEC-004 (Starter Recipe Content Seeding & Maintenance) | Affects (inbound) | Content created, corrected, or retired by this integration is reflected here on next load |
| FEAT-03.SPEC-001 (Weekly Plan View) | Navigation (inbound) | Tapping a dinner card opens this same detail view |
| FEAT-23 (Manual Weekly Planning) | Navigation (inbound) | Opening a recipe while building a week by hand uses this same detail view |
| FEAT-10.SPEC-004 (Edit Imported Recipe) | Navigation (outbound) | Maya or Sam tapping "Edit" on an imported recipe owned by the household opens this screen, carrying the recipe's ID and current field values |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| recipe_detail_viewed | entry source (library, plan, pick-list, disclosure rule), eligibility outcome (eligible / ineligible) | Detail load completes successfully | N/A -- no Stage 2 metric measures individual detail views; Recipe Library Coverage at Launch (the feature's connected metric) is fed by FEAT-08.SPEC-004's corpus-breadth signal, not per-view events |
| recipe_ineligible_explained | reason category (allergy, religious rule, incomplete data) | This screen renders an ineligibility explanation in place of dietary badges | N/A -- diagnostic only; Zero Allergy Incidents (success-metrics.md) measures whether an unsafe meal ever reaches a plan, not disclosure-on-view events |

## Acceptance Criteria

**FEAT-08.SPEC-002-AC-01:** Given Maya taps an eligible recipe's card from FEAT-08.SPEC-001, when the detail load completes, then she sees the recipe's ingredients, steps, cook time, rough cost, and dietary badges.

**FEAT-08.SPEC-002-AC-02:** Given Sam taps a planned meal on the weekly plan, when the detail load completes, then this same detail view opens for that meal's recipe.

**FEAT-08.SPEC-002-AC-03:** Given Maya opens a recipe that fails her household's allergy rule, when the detail load completes, then the summary row shows the plain ineligibility explanation, per FEAT-08.SPEC-003, in place of dietary badges.

**FEAT-08.SPEC-002-AC-04:** Given the detail load fails for Sam, when the failure occurs, then an error message with a Retry option appears, and his prior library search context is preserved for when he goes back.

**FEAT-08.SPEC-002-AC-05:** Given Sam taps Retry after a failed load, when the retry succeeds, then the recipe's full content displays as in the happy path.

**FEAT-08.SPEC-002-AC-06:** Given Maya has never viewed a particular recipe before and loses connectivity while opening it, when the load is attempted, then she sees "This recipe hasn't been viewed yet and needs a connection to load."

**FEAT-08.SPEC-002-AC-07:** Given Maya previously viewed a recipe and then loses connectivity, when she reopens that recipe, then its last-loaded content remains fully available.

**FEAT-08.SPEC-002-AC-08:** Given the older-kid limited-login role (Later) opens a recipe from the library, when the detail load completes, then they see the full recipe content with no edit or add-to-plan controls.

**FEAT-08.SPEC-002-AC-09:** Given Jordan is a young kid profile with no login, when any attempt is made to reach this screen, then no session exists in which it could open.

**FEAT-08.SPEC-002-AC-10:** Given an unauthenticated visitor attempts to open this screen via a shared link, when the attempt is made, then they are redirected to the sign-in screen and no recipe data is shown.

**FEAT-08.SPEC-002-AC-11:** Given Maya's session expires while viewing a recipe, when the expiry is detected, then a dialog reads "Your session has expired. Sign in to continue." and she returns to the same recipe after signing back in.

**FEAT-08.SPEC-002-AC-12:** Given Riley (Operator) opens this screen against a household with an open Support Request, when the detail load completes, then Riley sees the recipe's full content read-only, with no controls to change it.

**FEAT-08.SPEC-002-AC-13:** Given Maya opens an imported recipe owned by her household, when the detail load completes, then an "Edit" button appears in the header, and tapping it navigates to FEAT-10.SPEC-004 (Edit Imported Recipe) carrying the recipe's ID and current field values.

**FEAT-08.SPEC-002-AC-14:** Given Sam opens a starter recipe, when the detail load completes, then no "Edit" button appears in the header, regardless of his role.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 6 (empty (N/A), loading, loaded-eligible, loaded-ineligible, error, offline/degraded) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



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



# Integration Spec: Starter Recipe Content Seeding & Maintenance

## Overview

**Name:** Starter Recipe Content Seeding & Maintenance
**ID:** FEAT-08.SPEC-004
**Type:** Integration
**Purpose:** Creates, corrects, and retires the shared starter-recipe content through the product's recipe/food-content data capability, so the library ships pre-populated at launch and stays complete enough to pass the allergy safety check.
**Parent Feature:** FEAT-08 -- Recipe Library (Starter Recipes)

## Scope and Non-Goals

**In Scope:**
- Creating starter Recipe records (name, ingredients with quantity/unit, steps, cook_time, rough_cost, origin = starter library, no owning_household, prep_requirements) before any household exists
- Delivering corrections to existing starter content over time (e.g., an ingredient fix, a cost refresh)
- Retiring (soft-archiving) starter content that is no longer offered
- User-facing behavior on FEAT-08.SPEC-001 and FEAT-08.SPEC-002 when this capability is slow, unavailable, or rejects a delivery
- Satisfying the coverage breadth the Recipe Library Coverage at Launch metric requires (no repeated dinners, at least one recipe per major cuisine style, for a household with typical dietary rules)

**Non-Goals:**
- Choosing the recipe/food-content data vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for this capability.
- Household-imported recipe content -- creating, editing, or removing an imported Recipe record is FEAT-10's responsibility; this spec seeds and maintains starter content only.
- Bulk import tooling that lets households add their own recipes in quantity -- excluded per scope-boundaries.md SC-12: households bring their own recipes in one link at a time through FEAT-10, never in bulk, and this spec does not create such a path.
- Performing the allergy/religious-rule safety determination itself -- that determination is FEAT-02's authority (XBR-01); this spec's obligation is to supply ingredient data complete enough for that determination to run, not to run it.

## Capability Category

**Category:** Recipe/food-content data
**Dependency Source:** ASMP-34 -- "Recipe/food-content data capability -- Required to seed the starter recipe library at launch with complete ingredient data the allergy check can verify; without it, a brand-new household would have to rely entirely on manually imported recipes before any plan could be generated." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Recipe/food-content data (ASMP-34)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-08, FEAT-02; Integration Spec: FEAT-08.SPEC-004)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| A brand-new household sees a pre-populated recipe library from the moment setup completes, with no recipes of its own yet | Browse recipes | FEAT-08.SPEC-001 (Recipe Library Browse & Search) |
| A brand-new household's first-week plan draws from a varied, complete starter corpus rather than an empty candidate pool | View recipe detail; feeds AI Weekly Dinner Plan Generation (FEAT-03) as its day-one candidate pool | FEAT-08.SPEC-001, FEAT-08.SPEC-002 (Recipe Detail View) |
| Starter recipe content stays accurate over time as ingredient data is corrected or content is retired | Browse recipes; View recipe detail | FEAT-08.SPEC-001, FEAT-08.SPEC-002 |
| A starter recipe's ingredient data is complete enough that FEAT-02's safety check can pass it rather than fail it closed | (Cross-feature precondition for XBR-01's safety determination) | FEAT-02 (Dietary Rules & Allergy Safety Engine) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Coverage requirements | Not an entity -- functional targets: target cuisine styles, dietary-badge variety needed, minimum active corpus size | A new seeding batch is commissioned by the product's content operations (not triggered by any household action) | Tells the capability what content the library still needs so a new batch fills real gaps rather than duplicating existing coverage |

No household data, member data, or any personal data ever leaves the product through this capability -- starter recipe content carries no personal data (Recipe entity, Data Sensitivity: None, per feature-dependency-map.md), and this integration's outbound side is limited to content-coverage targets set by the product itself.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| New starter recipe content | The capability delivers a new recipe | Recipe -- name, ingredients (quantity + unit), steps, cook_time, rough_cost, origin (set to starter library), prep_requirements; owning_household left unset |
| Corrected starter recipe content | The capability delivers a correction to an existing starter recipe (e.g., an ingredient fix, a cost refresh) | Recipe -- the corrected field(s) only, identified by the existing recipe's name and origin |
| Retirement notice | The capability (or content operations) marks a starter recipe as no longer offered | Recipe -- soft-archived; no field content is deleted |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| New starter recipe delivered | The capability supplies a new recipe meeting the completeness standard (every ingredient has a quantity and unit) | A new Recipe record is created: origin = starter library, owning_household unset, full content populated | None directly -- the recipe simply appears in browse/search and detail on next load | FEAT-08.SPEC-001, FEAT-08.SPEC-002 |
| New starter recipe delivered with incomplete ingredient data | The capability supplies a recipe missing a quantity or unit on one or more ingredients | The record is held out of the Active/browsable state -- it is neither created as visible nor offered to the candidate pool | None -- this is a content-operations condition, not a household-facing one; no household is affected since the recipe never becomes visible | None -- fails closed before reaching any household-facing spec |
| Existing starter recipe corrected | The capability delivers a correction to a recipe already in the library | The named field(s) update on the existing Recipe record | None directly -- the correction is reflected the next time the recipe is loaded in FEAT-08.SPEC-001 or opened in FEAT-08.SPEC-002; FEAT-02's badges re-evaluate live at that next load, with no separate re-check trigger needed | FEAT-08.SPEC-001, FEAT-08.SPEC-002 |
| Starter recipe retired | Content operations marks a starter recipe as no longer offered | The Recipe record is soft-archived: hidden from browse/search, dropped from the candidate pool for future plan generation; no cascade to Planned Meals that already used it (their history stands unchanged); no restore path | None directly -- the recipe simply stops appearing in browse/search and future plan candidates on next load | FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-03 (AI Weekly Dinner Plan Generation, candidate pool) |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-08.SPEC-001 (Recipe Library Browse & Search) | No visible impact -- this screen reads already-seeded Recipe records directly and is not itself waiting on the capability | No visible impact to households -- new or corrected content simply does not appear until the capability is reachable again; households continue browsing the existing library uninterrupted | A delivery batch that fails the completeness standard is held out of the Active/browsable state (per the Inbound Events row above); households see no error and continue browsing only previously accepted content |
| FEAT-08.SPEC-002 (Recipe Detail View) | No visible impact -- this screen reads already-seeded Recipe records directly | No visible impact -- a recipe already in the library remains fully viewable; a recipe that would have arrived from a delayed delivery simply is not yet available to open | N/A -- this screen never submits content to the capability; it only reads what has already been accepted |

## Consent and Disclosure

- **Coverage requirements are not user-facing** -- N/A: the only data leaving the product through this capability is content-coverage targets, set by the product's own content operations rather than on behalf of any household member; no household member is ever asked to consent to this exchange, since no personal or household data is included.
- **What is never shared** -- No household data, member data, dietary rule data, or any other personal data crosses this boundary in either direction; this capability exchanges only static recipe content and coverage targets (Recipe entity, Data Sensitivity: None).

## Edge Cases

- **The same corrected recipe is delivered twice** -- The second delivery is a no-op if the content is unchanged from the first; if the two deliveries differ, the most recently delivered correction wins.
- **A retirement notice arrives for a recipe already retired** -- No-op; the recipe stays archived.
- **A correction arrives for a recipe that has already been retired** -- The correction is discarded; a retired starter recipe accepts no further corrections, since re-adding equivalent content is a fresh seed action, not an undo (per the Recipe entity's Delete/Archive lifecycle in feature-overview.md).
- **A seeding delivery arrives with incomplete ingredient data** -- The recipe is held out of the Active/browsable state and excluded from both browse/search and any candidate pool until a corrected delivery completes the missing data, consistent with FEAT-02's fail-closed safety posture (XBR-01).
- **Seeding events arrive before any household exists** -- Content is created directly into the shared pool (owning_household unset); it is available to the first household that completes setup, with no household-specific step required.
- **A retirement event arrives for a recipe currently referenced by an active Planned Meal in some household's plan** -- The Planned Meal's existing reference and history are unaffected; the recipe is simply excluded from future candidate pools and no longer browsable, per the Recipe entity's Delete/Archive notes in feature-dependency-map.md.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-001 (Recipe Library Browse & Search) | Affects (outbound) | New, corrected, or retired starter content is reflected in browse/search results on next load |
| FEAT-08.SPEC-002 (Recipe Detail View) | Affects (outbound) | New, corrected, or retired starter content is reflected in the detail view on next load |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Affects (outbound) | The seeded and maintained starter corpus is part of the candidate pool plan generation draws from; a retirement removes a recipe from future candidate pools |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | Affects (outbound) | Complete ingredient data delivered by this integration is the precondition FEAT-02's safety check requires to pass a starter recipe rather than fail it closed (XBR-01) |

## Analytics and Success Signals

- **starter_recipe_corpus_updated** (active recipe count, count added / corrected / retired in this delivery, cuisine styles represented in the active corpus) -- supports success-metrics.md: "Recipe Library Coverage at Launch"
- **starter_recipe_seed_incomplete** (recipe name, missing-data reason) -- N/A -- no Stage 2 metric tracks rejected-delivery volume; retained so content operations can see what a rejected delivery still needs corrected

## Acceptance Criteria

**FEAT-08.SPEC-004-AC-01:** Given no household has completed setup yet, when the recipe/food-content data capability delivers the initial starter batch, then Recipe records are created with origin set to starter library and owning_household unset, ready for the first household's library to show them.

**FEAT-08.SPEC-004-AC-02:** Given a household's library already shows a starter recipe, when the capability delivers a correction to that recipe's rough cost, then the corrected cost is reflected the next time Maya opens the recipe in FEAT-08.SPEC-002.

**FEAT-08.SPEC-004-AC-03:** Given a starter recipe is retired by content operations, when the retirement is processed, then the recipe is hidden from FEAT-08.SPEC-001's browse/search results and dropped from future plan-generation candidate pools, while any Planned Meal that already used it keeps its history unchanged.

**FEAT-08.SPEC-004-AC-04:** Given the capability delivers a new recipe missing a quantity on one ingredient, when the delivery is processed, then the recipe is held out of the Active/browsable state and never appears to any household until a corrected delivery completes the data.

**FEAT-08.SPEC-004-AC-05:** Given a household is browsing the library while the capability is unreachable, when they search or scroll, then browsing continues uninterrupted against the already-seeded content, with no visible error.

**FEAT-08.SPEC-004-AC-06:** Given a household opens a recipe detail while the capability is unreachable, when the recipe was already seeded, then it opens normally, since this screen reads already-accepted content directly rather than the capability itself.

**FEAT-08.SPEC-004-AC-07:** Given the same corrected recipe delivery is processed twice with identical content, when the second delivery is processed, then nothing changes and no duplicate update is recorded.

**FEAT-08.SPEC-004-AC-08:** Given a retirement notice arrives for a recipe already retired, when it is processed, then the recipe remains archived with no change.

**FEAT-08.SPEC-004-AC-09:** Given a correction arrives for a recipe that has already been retired, when it is processed, then the correction is discarded and the recipe stays archived.

**FEAT-08.SPEC-004-AC-10:** Given a new household completes setup for the first time, when they open the recipe library, then they see a starter corpus with no repeated dinners and at least one recipe per major cuisine style, per the corpus's coverage requirement.

**FEAT-08.SPEC-004-AC-11:** Given content operations commissions a new seeding batch, when the coverage requirements are sent to the capability, then no household or personal data is included in that request.

**FEAT-08.SPEC-004-AC-12:** Given a starter recipe passes FEAT-02's safety check because its ingredient data is complete, when a household views it, then it shows the standard dietary badge rather than an ineligibility explanation, confirming this integration's data supplied what FEAT-02's determination required.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 5 (2 screens; 1 N/A cell excluded) | 5 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 6 | 6 |
