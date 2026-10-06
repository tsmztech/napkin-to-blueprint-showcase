---
document_type: spec
spec_type: screen
spec_id: FEAT-10.SPEC-003
spec_name: Manual Recipe Entry
spec_slug: manual-recipe-entry
parent_feature: FEAT-10
parent_feature_name: Recipe Import from Web Link
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Manual Recipe Entry

## Overview

**Name:** Manual Recipe Entry
**ID:** FEAT-10.SPEC-003
**Type:** Screen
**Purpose:** Household member types in ingredients, steps, and cook time by hand when link extraction fails, so the import never dead-ends.
**Parent Feature:** FEAT-10 -- Recipe Import from Web Link

## Scope and Non-Goals

**In Scope:**
- Presenting a blank Recipe details form when extraction has failed and the member chooses to proceed manually
- Validating the entered content via FEAT-10.SPEC-007
- Triggering duplicate detection (FEAT-10.SPEC-006) and the safety re-check (FEAT-10.SPEC-008) on save

**Non-Goals:**
- Attempting extraction again -- retrying extraction happens by returning to FEAT-10.SPEC-001 (Import by Link) and re-submitting a link; this screen assumes extraction has already failed.
- Reviewing a system-extracted draft -- handled by FEAT-10.SPEC-002 (Review Extracted Recipe); this screen's form always starts empty.
- General ad-hoc recipe creation unconnected to an attempted import -- product-features.md's Primary Flows & Alternates ties manual entry specifically to the "extraction failure" alternate path; there is no standalone "create a recipe from scratch" entry point in the product definition.
- Capturing a source link for the manually entered recipe -- since extraction failed, no page was successfully read; the recipe is saved with origin = imported but without a confirmed source link, distinct from FEAT-10.SPEC-002's saved recipes which always carry one.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-10.SPEC-001 (Import by Link) | Member taps "Enter details manually" after an extraction failure | None -- form starts empty; the originally submitted link is not carried into this screen's data model |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Fill in the form and save | -- |
| Sam (Other Adult Member) | Full screen | Fill in the form and save | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile -- there is no path to this screen |
| Jordan (older kid, limited login -- Later) | No | No | Not reachable -- reaching it requires FEAT-10.SPEC-001, which is not offered to this role (Recipe Library access is View, not Full) |
| Riley (Operator, support) | No | No | Not reachable -- Riley's Recipe Library access is View only and carries no import or save path |
| Unauthenticated | No | No | Redirected to the sign-in screen; any in-progress manual entry is discarded since it was never associated with a signed-in session |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- entered form data is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Add Recipe Details" with a back arrow (returns to FEAT-10.SPEC-001) and a "Save" action button (right-aligned).

**Body:** A single-column form, empty by default (the shared Recipe details form pattern -- identical field set and order to FEAT-10.SPEC-002 and FEAT-10.SPEC-004), with the following fields in order:
- Recipe Name (text input, required, empty)
- Ingredients (a repeatable list of ingredient lines, each with a quantity, a unit, and an ingredient name; at least one required; "Add ingredient" control below the list; each line has a "Remove" control), starting with one empty line
- Steps (multi-line text input, method text in order, empty)
- Cook Time (number input, minutes, required, empty)

Below the form, a note states: "We couldn't read the original page, so add the details yourself." No source link is shown, since none was successfully confirmed.

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; the ingredient list's quantity/unit/name inputs sit on one row instead of stacking.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | If any field is filled, show confirmation; otherwise navigate to FEAT-10.SPEC-001 | Screen closes (or confirmation shown) | Confirmation dialog or animated transition |
| Recipe Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Ingredient line (quantity/unit/name) | Type | Captures entered values | Line shows entered values | Standard input focus state |
| "Add ingredient" | Tap | Appends a new empty ingredient line | List grows by one line | New line focused |
| Ingredient line "Remove" | Tap | Removes that ingredient line | List shrinks by one line | Line disappears; removing the only remaining line blocks save per FEAT-10.SPEC-007 |
| Steps input | Type | Captures method text | Field shows entered text | Standard input focus state |
| Cook Time input | Type | Captures entered value | Field shows entered value | Standard input focus state |
| Save button | Tap | 1. Validate all fields via FEAT-10.SPEC-007. 2. If valid, run duplicate detection via FEAT-10.SPEC-006. 3. If no duplicate, create the Recipe record and trigger the safety re-check via FEAT-10.SPEC-008. | Button shows a saving state | Success: toast "Recipe saved" and navigate to FEAT-08.SPEC-002. Duplicate found: FEAT-10.SPEC-006 surfaces the existing recipe instead. Failure: inline error messages from FEAT-10.SPEC-007. |
| Save button (while saving) | Tap | No action -- debounced | None | Button remains in saving state |

### Accessibility Notes

- **Focus order:** Back arrow -> Recipe Name -> each ingredient line (quantity -> unit -> name -> remove) in order -> Add ingredient -> Steps -> Cook Time -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Recipe saved" toast is announced on success; on validation failure, focus moves to the first field in error; when a duplicate is found, the redirect to the existing recipe is announced.
- **Keyboard alternatives:** Every action, including adding and removing ingredient lines, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | All form fields empty except one blank ingredient line, Save enabled | Screen opens after an extraction failure | Member types in any field |
| Filling | Form fields contain member input | Member types in any field | Member taps Save or navigates away |
| Validating | Save button shows a brief loading state | Member taps Save | FEAT-10.SPEC-007's checks complete |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails | Member corrects the field and re-triggers validation |
| Saving | Save button shows loading state, form fields disabled | Validation passes, duplicate check (FEAT-10.SPEC-006) finds no match | Save completes or fails |
| Duplicate Found | Modal or inline message: "You've already imported this recipe." with a "View existing recipe" action | FEAT-10.SPEC-006 finds a matching saved recipe for this household (matched on the recipe details the member entered, since no link is available for this path) | Member taps "View existing recipe", navigating to that recipe; no new Recipe is created |
| Error | Error banner at top of form: "Couldn't save this recipe. Check your connection and try again." with a Retry action | Save operation fails | Member taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline -- this recipe will be saved when you reconnect." at top; form remains editable; Save queues the recipe locally | Connectivity lost while the screen is open | Connectivity restored -- the queued save proceeds automatically and the standard success feedback appears |

## Validation Rules

Validation governed by FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules). See that spec for field-level and cross-field rules (ingredient/step length caps, minimum-one-ingredient requirement). This screen applies validation on field blur and on Save tap. The well-formedness and weekly-limit rules governing link submission (also in FEAT-10.SPEC-007) do not apply here, since this screen has no link field.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (no data entered) | FEAT-10.SPEC-001 (Import by Link) | -- |
| Successful save | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 |
| Duplicate found, "View existing recipe" tap | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 |

## Data Model

**Creates:** Recipe record -- name, ingredients (quantity + unit), steps, cook_time set from the entered form; origin set to imported with no confirmed source link; owning_household set to the saving member's household. Created only after FEAT-10.SPEC-007 validation passes and FEAT-10.SPEC-006 finds no duplicate.
**Reads:** None (this screen starts from an empty form).
**Updates:** None (this screen creates; FEAT-10.SPEC-004 owns later edits).
**Deletes:** None.

## Business Rules

- Duplicate detection (FEAT-10.SPEC-006) runs automatically on save -- the member cannot skip it, matched against the entered recipe details since no source link exists for this path.
- If a duplicate is found, no new Recipe is created; the existing recipe is surfaced instead (XBR-19).
- Field validation (FEAT-10.SPEC-007) rules are enforced -- the member cannot save with invalid data, including fewer than one ingredient.
- Once saved, the safety re-check (FEAT-10.SPEC-008) runs before the recipe can appear in any plan (XBR-01, XBR-19); the save itself completes regardless of the safety outcome, per FEAT-02's fail-closed posture.
- This screen never dead-ends the import: reaching it always means the member has a working path to save a recipe even when extraction fails, per product-features.md's Error state definition.

## Edge Cases

- **Member navigates away with data entered** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Member taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in saving state).
- **Network failure during save** -- Error banner: "Couldn't save this recipe. Check your connection and try again." with a Retry button. Form data preserved.
- **All ingredient lines removed** -- Save is blocked per FEAT-10.SPEC-007's minimum-one-ingredient rule; the ingredient list shows "At least one ingredient is required."
- **Member manually enters details for a recipe that matches one already imported (e.g., re-typing a recipe from memory that was previously imported by link)** -- No live conflict on this creation screen (no existing record is loaded); duplicate detection (FEAT-10.SPEC-006) still runs on the entered details and, if it finds a match, surfaces the existing recipe instead of creating a second record. This is the concurrent-edit conflict entry for this screen, consistent with the Recipe entity's Contention note in feature-dependency-map.md.
- **Entered content contains an ingredient with an unrecognizable name** -- The save still proceeds if all other validation passes; the recipe is excluded from every plan candidate path until the ingredient is clarified, per FEAT-10.SPEC-008 and XBR-01's fail-closed posture.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-001 (Import by Link) | Navigation (inbound) | Extraction failure, if the member proceeds manually, navigates here |
| FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules) | References (inbound) | Field-level and cross-field validation applied on save |
| FEAT-10.SPEC-006 (Duplicate Import Detection) | Triggers (outbound) | Save action triggers duplicate detection |
| FEAT-10.SPEC-008 (Safety Re-check on Import Save/Edit) | Triggers (outbound) | Successful save triggers the safety re-check |
| FEAT-08.SPEC-002 (Recipe Detail View) | Navigation (outbound) | Successful save or duplicate resolution navigates here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| recipe_import_manual_fallback | -- | Member taps "Enter details manually" from FEAT-10.SPEC-001 and this screen opens | N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained for operational visibility into how often extraction fails badly enough to require manual fallback |
| recipe_import_succeeded | field count filled, ingredient count, cook time, entry_path: manual | Save completes and the Recipe record is created | N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained for operational visibility into the import funnel |

## Acceptance Criteria

**FEAT-10.SPEC-003-AC-01:** Given Maya's link extraction failed and she taps "Enter details manually", when the screen opens, then all form fields are empty except one blank ingredient line, and the note "We couldn't read the original page, so add the details yourself." is shown.

**FEAT-10.SPEC-003-AC-02:** Given Sam fills in the recipe name, one ingredient, steps, and cook time, when he taps Save, then duplicate detection (FEAT-10.SPEC-006) runs and, finding no match, the Recipe record is created and he sees a "Recipe saved" toast.

**FEAT-10.SPEC-003-AC-03:** Given Maya has removed every ingredient line, when she taps Save, then the ingredient list shows "At least one ingredient is required." and the save does not proceed.

**FEAT-10.SPEC-003-AC-04:** Given Sam manually enters details matching a recipe his household already imported, when he taps Save, then no new Recipe is created and he sees "You've already imported this recipe." with "View existing recipe".

**FEAT-10.SPEC-003-AC-05:** Given Maya saves a manually entered recipe, when the save completes, then the safety re-check (FEAT-10.SPEC-008) runs before the recipe can appear in any plan.

**FEAT-10.SPEC-003-AC-06:** Given Sam has entered data and taps the back arrow, when the confirmation dialog appears, then it reads "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-10.SPEC-003-AC-07:** Given Maya taps Save twice rapidly, when the first save is still in progress, then the second tap has no effect.

**FEAT-10.SPEC-003-AC-08:** Given Sam loses connectivity while filling the manual entry form, when he taps Save, then the banner "You're offline -- this recipe will be saved when you reconnect." appears and the save is queued.

**FEAT-10.SPEC-003-AC-09:** Given Maya enters a recipe with an ingredient the safety engine cannot recognize, when the save completes, then the recipe is saved to the household's pool but is excluded from every plan candidate path until the ingredient is clarified.

**FEAT-10.SPEC-003-AC-10:** Given the network fails during Sam's save attempt, when the failure is reported, then the error banner "Couldn't save this recipe. Check your connection and try again." appears with a Retry button, and his form data is preserved.

**FEAT-10.SPEC-003-AC-11:** Given the older-kid limited-login role (Later) attempts to reach this screen directly without going through FEAT-10.SPEC-001, when the attempt is made, then it fails since this role's Recipe Library access is View, not Full, and no import entry point is available to it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 8 (empty, filling, validating, validation error, saving, duplicate found, error, offline) | 8 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
