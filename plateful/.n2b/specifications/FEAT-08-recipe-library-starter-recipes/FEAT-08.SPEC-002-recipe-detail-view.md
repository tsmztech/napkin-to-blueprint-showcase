---
document_type: spec
spec_type: screen
spec_id: FEAT-08.SPEC-002
spec_name: Recipe Detail View
spec_slug: recipe-detail-view
parent_feature: FEAT-08
parent_feature_name: Recipe Library (Starter Recipes)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

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
