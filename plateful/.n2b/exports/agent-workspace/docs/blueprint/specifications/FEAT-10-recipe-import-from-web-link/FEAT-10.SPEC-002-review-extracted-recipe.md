---
document_type: spec
spec_type: screen
spec_id: FEAT-10.SPEC-002
spec_name: Review Extracted Recipe
spec_slug: review-extracted-recipe
parent_feature: FEAT-10
parent_feature_name: Recipe Import from Web Link
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Review Extracted Recipe

## Overview

**Name:** Review Extracted Recipe
**ID:** FEAT-10.SPEC-002
**Type:** Screen
**Purpose:** Household member confirms or edits the extracted ingredients, steps, and cook time, then saves the recipe to the household's pool.
**Parent Feature:** FEAT-10 -- Recipe Import from Web Link

## Scope and Non-Goals

**In Scope:**
- Displaying the draft ingredients, steps, and cook time extracted by FEAT-10.SPEC-005
- Allowing the member to edit any field before saving
- Validating the edited or confirmed content via FEAT-10.SPEC-007
- Triggering duplicate detection (FEAT-10.SPEC-006) and the safety re-check (FEAT-10.SPEC-008) on save

**Non-Goals:**
- Extracting the recipe content from the web page -- performed by FEAT-10.SPEC-005 (Web Page Recipe Extraction); this screen only displays and edits the result.
- Manual entry from a blank form -- handled by FEAT-10.SPEC-003 (Manual Recipe Entry), reached only when extraction fails.
- Editing an already-saved imported recipe later -- handled by FEAT-10.SPEC-004 (Edit Imported Recipe); this screen only covers the initial save of a fresh import.
- Setting a rough cost for the imported recipe -- product-features.md's Data Notes list this feature's owned fields as name, ingredients, steps, cook_time, origin, and owning_household only; rough_cost is not captured for imported content by this feature and the field remains unset for imported Recipe records.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-10.SPEC-001 (Import by Link) | Extraction succeeds | The extracted draft: ingredients (quantity + unit), steps, cook_time, plus the source link |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Edit any field and save | -- |
| Sam (Other Adult Member) | Full screen | Edit any field and save | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile -- there is no path to this screen |
| Jordan (older kid, limited login -- Later) | No | No | This screen is not reachable -- reaching it requires the "Import from link" entry point, which is not offered to this role (Recipe Library access is View, not Full) |
| Riley (Operator, support) | No | No | Not reachable -- Riley's Recipe Library access is View only and carries no import or save path |
| Unauthenticated | No | No | Redirected to the sign-in screen; the in-flight extracted draft is discarded since it was never associated with a signed-in session |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- the extracted draft and any edits made so far are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Review Recipe" with a back arrow (returns to FEAT-10.SPEC-001) and a "Save" action button (right-aligned).

**Body:** A single-column form, pre-filled from the extracted draft, with the following fields in order (the shared Recipe details form pattern -- identical field set and order to FEAT-10.SPEC-003 and FEAT-10.SPEC-004):
- Recipe Name (text input, required, pre-filled from extraction)
- Ingredients (a repeatable list of ingredient lines, each with a quantity, a unit, and an ingredient name; at least one required; "Add ingredient" control below the list; each line has a "Remove" control), pre-filled from extraction
- Steps (multi-line text input, method text in order, pre-filled from extraction)
- Cook Time (number input, minutes, required, pre-filled from extraction)

Below the form, a read-only line shows the source: "Imported from {source link's site name}" with the source link itself available on tap (opens the original page in a new context).

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; the ingredient list's quantity/unit/name inputs sit on one row instead of stacking.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | If unsaved changes exist, show confirmation; otherwise navigate to FEAT-10.SPEC-001 | Screen closes (or confirmation shown) | Confirmation dialog or animated transition |
| Recipe Name input | Type | Captures edited text | Field shows edited text | Standard input focus state |
| Ingredient line (quantity/unit/name) | Type or edit | Captures edited values | Line shows edited values | Standard input focus state |
| "Add ingredient" | Tap | Appends a new empty ingredient line | List grows by one line | New line focused |
| Ingredient line "Remove" | Tap | Removes that ingredient line | List shrinks by one line | Line disappears; if it was the only ingredient, validation blocks save per FEAT-10.SPEC-007 |
| Steps input | Type | Captures edited method text | Field shows edited text | Standard input focus state |
| Cook Time input | Type | Captures edited value | Field shows edited value | Standard input focus state |
| Save button | Tap | 1. Validate all fields via FEAT-10.SPEC-007. 2. If valid, run duplicate detection via FEAT-10.SPEC-006. 3. If no duplicate, create the Recipe record and trigger the safety re-check via FEAT-10.SPEC-008. | Button shows a saving state | Success: toast "Recipe saved" and navigate to the recipe's detail in FEAT-08.SPEC-002. Duplicate found: FEAT-10.SPEC-006 surfaces the existing recipe instead. Failure: inline error messages from FEAT-10.SPEC-007. |
| Save button (while saving) | Tap | No action -- debounced | None | Button remains in saving state |
| Source link | Tap | Opens the original source page | -- | Original page opens in a new context; this screen remains open behind it |

### Accessibility Notes

- **Focus order:** Back arrow -> Recipe Name -> each ingredient line (quantity -> unit -> name -> remove) in order -> Add ingredient -> Steps -> Cook Time -> source link -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Recipe saved" toast is announced on success; on validation failure, focus moves to the first field in error; when a duplicate is found, the redirect to the existing recipe is announced.
- **Keyboard alternatives:** Every action, including adding and removing ingredient lines, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Populated (default) | Form fields pre-filled from the extracted draft, Save enabled | Screen opens after successful extraction | Member edits any field |
| Editing | Form fields contain member edits, Save enabled | Member types in or changes any field | Member taps Save or navigates away |
| Validating | Save button shows a brief loading state | Member taps Save | FEAT-10.SPEC-007's checks complete |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails | Member corrects the field and re-triggers validation |
| Saving | Save button shows loading state, form fields disabled | Validation passes, duplicate check (FEAT-10.SPEC-006) finds no match | Save completes or fails |
| Duplicate Found | Modal or inline message: "You've already imported this recipe." with a "View existing recipe" action | FEAT-10.SPEC-006 finds a link already saved for this household | Member taps "View existing recipe", navigating to that recipe in FEAT-08.SPEC-002; no new Recipe is created |
| Error | Error banner at top of form: "Couldn't save this recipe. Check your connection and try again." with a Retry action | Save operation fails | Member taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline -- this recipe will be saved when you reconnect." at top; form remains editable; Save queues the recipe locally | Connectivity lost while the screen is open | Connectivity restored -- the queued save proceeds automatically (validation, duplicate check, create) and the standard success feedback appears |

## Validation Rules

Validation governed by FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules). See that spec for field-level and cross-field rules (ingredient/step length caps, minimum-one-ingredient requirement). This screen applies validation on field blur and on Save tap.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (no unsaved changes) | FEAT-10.SPEC-001 (Import by Link) | -- |
| Successful save | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 |
| Duplicate found, "View existing recipe" tap | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 |
| Source link tap | External source page (opens outside the product) | -- |

## Data Model

**Creates:** Recipe record -- name, ingredients (quantity + unit), steps, cook_time set from the confirmed/edited form; origin set to imported with the source link recorded; owning_household set to the saving member's household. Created only after FEAT-10.SPEC-007 validation passes and FEAT-10.SPEC-006 finds no duplicate.
**Reads:** The extracted draft passed from FEAT-10.SPEC-005 (not yet a persisted Recipe record).
**Updates:** None (this screen creates; FEAT-10.SPEC-004 owns later edits).
**Deletes:** None.

## Business Rules

- Duplicate detection (FEAT-10.SPEC-006) runs automatically on save -- the member cannot skip it.
- If a duplicate is found, no new Recipe is created; the existing recipe is surfaced instead (XBR-19).
- Field validation (FEAT-10.SPEC-007) rules are enforced -- the member cannot save with invalid data, including fewer than one ingredient.
- Once saved, the safety re-check (FEAT-10.SPEC-008) runs before the recipe can appear in any plan (XBR-01, XBR-19); the save itself completes regardless of the safety outcome, per FEAT-02's fail-closed posture.
- dietary_badges are never set by this screen -- they are computed live by FEAT-02 (Dietary Rules & Allergy Safety Engine) once the safety re-check completes.

## Edge Cases

- **Member navigates away with unsaved edits** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Member taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in saving state).
- **Network failure during save** -- Error banner: "Couldn't save this recipe. Check your connection and try again." with a Retry button. Form data preserved.
- **All ingredients removed** -- Save is blocked per FEAT-10.SPEC-007's minimum-one-ingredient rule; the ingredient list shows "At least one ingredient is required."
- **Another household member (e.g., Sam) saves the same link from a separate device while this screen is open for Maya** -- No live conflict on this creation screen (no existing Recipe record is loaded here); duplicate detection (FEAT-10.SPEC-006) catches the collision at whichever save completes second, surfacing the already-saved recipe to that member instead of creating a second record. This is the concurrent-edit conflict entry for this screen, consistent with the Recipe entity's Contention note (reject-with-refresh at the second save) in feature-dependency-map.md.
- **Extracted content contains an ingredient with an unrecognizable name** -- The save still proceeds if all other validation passes (per FEAT-10.SPEC-007); the recipe is excluded from every plan candidate path until the ingredient is clarified, per FEAT-10.SPEC-008 and XBR-01's fail-closed posture. No blocking error appears here -- the exclusion is enforced downstream by FEAT-02.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-001 (Import by Link) | Navigation (inbound) | Extraction success navigates here with the draft |
| FEAT-10.SPEC-005 (Web Page Recipe Extraction) | References (inbound) | Supplies the extracted draft this screen displays |
| FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules) | References (inbound) | Field-level and cross-field validation applied on save |
| FEAT-10.SPEC-006 (Duplicate Import Detection) | Triggers (outbound) | Save action triggers duplicate detection |
| FEAT-10.SPEC-008 (Safety Re-check on Import Save/Edit) | Triggers (outbound) | Successful save triggers the safety re-check |
| FEAT-08.SPEC-002 (Recipe Detail View) | Navigation (outbound) | Successful save or duplicate resolution navigates here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| recipe_import_succeeded | field-edit count (how many extracted fields the member changed before saving), ingredient count, cook time | Save completes and the Recipe record is created | N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained for operational visibility into how often extraction requires member correction |
| recipe_import_review_abandoned | last field focused | Member discards unsaved changes and navigates away | N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained for operational visibility into import funnel drop-off |

## Acceptance Criteria

**FEAT-10.SPEC-002-AC-01:** Given Maya's link extraction succeeded, when she opens the Review Extracted Recipe screen, then the recipe name, ingredients, steps, and cook time are pre-filled from the extracted draft.

**FEAT-10.SPEC-002-AC-02:** Given Sam is reviewing an extracted recipe, when he edits the cook time and taps Save, then the edited cook time is saved on the new Recipe record.

**FEAT-10.SPEC-002-AC-03:** Given Maya removes all ingredient lines, when she taps Save, then the ingredient list shows "At least one ingredient is required." and the save does not proceed.

**FEAT-10.SPEC-002-AC-04:** Given Sam taps Save on a valid extracted recipe with no existing duplicate, when duplicate detection (FEAT-10.SPEC-006) finds nothing, then the Recipe record is created, a "Recipe saved" toast appears, and he is navigated to FEAT-08.SPEC-002.

**FEAT-10.SPEC-002-AC-05:** Given Maya taps Save on a recipe whose link was already imported by her household, when duplicate detection (FEAT-10.SPEC-006) finds the existing recipe, then no new Recipe is created and she is shown "You've already imported this recipe." with "View existing recipe".

**FEAT-10.SPEC-002-AC-06:** Given Maya saves a recipe, when the save completes, then the safety re-check (FEAT-10.SPEC-008) runs before the recipe can appear in any plan.

**FEAT-10.SPEC-002-AC-07:** Given Sam is on this screen with unsaved edits, when he taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-10.SPEC-002-AC-08:** Given Maya taps Save twice rapidly, when the first save is still in progress, then the second tap has no effect.

**FEAT-10.SPEC-002-AC-09:** Given Sam loses connectivity while editing the review form, when he taps Save, then the banner "You're offline -- this recipe will be saved when you reconnect." appears and the save is queued.

**FEAT-10.SPEC-002-AC-10:** Given Maya and Sam each submit the same link for import from separate devices at nearly the same time, when both reach Save, then whichever save completes second is shown the existing recipe instead of creating a duplicate, per FEAT-10.SPEC-006.

**FEAT-10.SPEC-002-AC-11:** Given Sam saves a recipe containing an ingredient the safety engine cannot recognize, when the save completes, then the recipe is saved to the household's pool but is excluded from every plan candidate path until the ingredient is clarified.

**FEAT-10.SPEC-002-AC-12:** Given the network fails during Maya's save attempt, when the failure is reported, then the error banner "Couldn't save this recipe. Check your connection and try again." appears with a Retry button, and her form data is preserved.

**FEAT-10.SPEC-002-AC-13:** Given Maya taps the source link shown below the form, when it opens, then the original page opens in a new context and this screen remains open behind it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 8 (populated, editing, validating, validation error, saving, duplicate found, error, offline) | 8 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
