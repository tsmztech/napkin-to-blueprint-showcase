---
document_type: feature-overview
feature_number: FEAT-08
feature_name: Recipe Library (Starter Recipes)
feature_slug: recipe-library-starter-recipes
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 4
screen_count: 2
automation_count: 0
logic_rule_count: 1
integration_count: 1
notification_count: 0
---

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
