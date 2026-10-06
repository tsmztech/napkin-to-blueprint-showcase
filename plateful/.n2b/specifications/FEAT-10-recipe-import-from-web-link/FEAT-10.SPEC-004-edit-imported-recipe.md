---
document_type: spec
spec_type: screen
spec_id: FEAT-10.SPEC-004
spec_name: Edit Imported Recipe
spec_slug: edit-imported-recipe
parent_feature: FEAT-10
parent_feature_name: Recipe Import from Web Link
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Edit Imported Recipe

## Overview

**Name:** Edit Imported Recipe
**ID:** FEAT-10.SPEC-004
**Type:** Screen
**Purpose:** Household member corrects an already-saved imported recipe's details, or removes it from the household's pool.
**Parent Feature:** FEAT-10 -- Recipe Import from Web Link

## Scope and Non-Goals

**In Scope:**
- Loading an existing imported Recipe record's full content for correction
- Editing name, ingredients, steps, and cook time via the shared Recipe details form
- Removing (hard delete) an imported recipe from the household's pool, with confirmation
- Triggering the safety re-check (FEAT-10.SPEC-008) on every accepted edit before the recipe can appear in a plan again

**Non-Goals:**
- Editing a starter recipe -- product-features.md and the dependency map state starter recipes are read-only for households; this screen only opens for imported recipes (origin = imported).
- Undoing a removal or recovering a deleted imported recipe -- excluded per the feature's own Entity-Lifecycle Coverage Matrix: removal is immediate and permanent with no retention window and no restore path, since an imported recipe is the household's own data under their direct delete control, not shared maintained content; re-adding requires a fresh import.
- The initial save of a freshly imported recipe -- handled by FEAT-10.SPEC-002 (Review Extracted Recipe) and FEAT-10.SPEC-003 (Manual Recipe Entry); this screen only covers edits and removal after the first save.
- Editing the recipe's source link -- the source link is fixed at import time and is not an editable field on this screen; correcting a link requires removing the recipe and re-importing.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-08.SPEC-002 (Recipe Detail View) | Maya or Sam taps the header "Edit" button on an imported recipe owned by the household | The Recipe record's ID and current field values |

**Cross-feature reconciliation note:** product-features.md's Key Capabilities for this feature require an edit-and-remove path for imported recipes, and this entry point is the only place that path can start (FEAT-08.SPEC-002 is the sole detail view for any recipe, starter or imported). FEAT-08.SPEC-002 now shows an "Edit" control to Maya and Sam specifically when the recipe's origin is imported and it is owned by the household -- never on starter recipes, which stay read-only for households -- and navigates here carrying the recipe's ID and current field values (FEAT-08.SPEC-002 Interactions, Navigation Out and AC-13). The "Remove recipe" path stays on this screen, per this spec's Footer. Reconciled in Pass D (SG-05).

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Edit any field and save; remove the recipe | -- |
| Sam (Other Adult Member) | Full screen | Edit any field and save; remove the recipe | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile -- there is no path to this screen |
| Jordan (older kid, limited login -- Later) | No | No | "Edit" is not shown on FEAT-08.SPEC-002 for this role (Recipe Library access is View, not Full); if this screen is reached directly, it shows "Editing recipes isn't available on this profile." with a link back to the recipe detail |
| Riley (Operator, support) | View only, through the read-only support view against an open Support Request | No | Edit and Remove controls are not shown to Riley; the recipe's content is visible read-only for diagnosis only |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on FEAT-08.SPEC-001 (Recipe Library), not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- unsaved edits are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Edit Recipe" with a back arrow (returns to FEAT-08.SPEC-002) and a "Save" action button (right-aligned).

**Body:** A single-column form, pre-filled from the existing saved recipe (the shared Recipe details form pattern -- identical field set and order to FEAT-10.SPEC-002 and FEAT-10.SPEC-003), with the following fields in order:
- Recipe Name (text input, required, pre-filled)
- Ingredients (a repeatable list of ingredient lines, each with a quantity, a unit, and an ingredient name; at least one required; "Add ingredient" control below the list; each line has a "Remove" control), pre-filled
- Steps (multi-line text input, method text in order, pre-filled)
- Cook Time (number input, minutes, required, pre-filled)

Below the form, a read-only line shows the source: "Imported from {source link's site name}" with the source link available on tap, when the recipe carries one (recipes saved through FEAT-10.SPEC-003, Manual Recipe Entry, carry no source link and this line is omitted for them).

**Footer:** A "Remove recipe" destructive action, visually separated below the form.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width; Save remains in the header; "Remove recipe" spans the footer width.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; the ingredient list's quantity/unit/name inputs sit on one row instead of stacking.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | If unsaved changes exist, show confirmation; otherwise navigate to FEAT-08.SPEC-002 | Screen closes (or confirmation shown) | Confirmation dialog or animated transition |
| Recipe Name input | Type | Captures edited text | Field shows edited text | Standard input focus state |
| Ingredient line (quantity/unit/name) | Type or edit | Captures edited values | Line shows edited values | Standard input focus state |
| "Add ingredient" | Tap | Appends a new empty ingredient line | List grows by one line | New line focused |
| Ingredient line "Remove" | Tap | Removes that ingredient line | List shrinks by one line | Line disappears; removing the only remaining line blocks save per FEAT-10.SPEC-007 |
| Steps input | Type | Captures edited method text | Field shows edited text | Standard input focus state |
| Cook Time input | Type | Captures edited value | Field shows edited value | Standard input focus state |
| Save button | Tap | 1. Validate all fields via FEAT-10.SPEC-007. 2. If valid, update the Recipe record. 3. Trigger the safety re-check via FEAT-10.SPEC-008. | Button shows a saving state | Success: toast "Recipe updated" and navigate to FEAT-08.SPEC-002. Failure: inline error messages from FEAT-10.SPEC-007, or a conflict dialog if the record changed since load. |
| Save button (while saving) | Tap | No action -- debounced | None | Button remains in saving state |
| "Remove recipe" | Tap | Shows a confirmation dialog | Dialog appears | Dialog: "Remove this recipe? It will be deleted from your household's recipe pool. This can't be undone." with "Remove" and "Cancel" |
| Confirmation dialog "Remove" | Tap | Deletes the Recipe record | Screen closes | Toast "Recipe removed" and navigate to FEAT-08.SPEC-001 (Recipe Library) |
| Confirmation dialog "Cancel" | Tap | Dismisses the dialog | Dialog closes | Returns to the edit form unchanged |

### Accessibility Notes

- **Focus order:** Back arrow -> Recipe Name -> each ingredient line (quantity -> unit -> name -> remove) in order -> Add ingredient -> Steps -> Cook Time -> source link (when present) -> Save -> Remove recipe.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save and removal feedback:** The "Recipe updated" and "Recipe removed" toasts are announced on success; the removal confirmation dialog's message is announced when it opens; on validation failure, focus moves to the first field in error.
- **Keyboard alternatives:** Every action, including adding/removing ingredient lines and the removal confirmation, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Form fields pre-filled from the current saved recipe, Save enabled | Screen opens from FEAT-08.SPEC-002 | Member edits any field or taps Remove recipe |
| Editing | Form fields contain member edits, Save enabled | Member types in or changes any field | Member taps Save or navigates away |
| Validating | Save button shows a brief loading state | Member taps Save | FEAT-10.SPEC-007's checks complete |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails | Member corrects the field and re-triggers validation |
| Saving | Save button shows loading state, form fields disabled | Validation passes | Save completes or fails |
| Conflict | Dialog: "This recipe was updated on another device while you were editing. Review the latest version before saving." with "View Latest" and "Keep Editing" options | Save is rejected because the record changed since load | Member taps "View Latest" (reloads the record; local edits discarded after confirmation) or "Keep Editing" (dialog closes, edits remain, member may retry Save) |
| Removal Confirmation | Dialog: "Remove this recipe? It will be deleted from your household's recipe pool. This can't be undone." with "Remove" and "Cancel" | Member taps "Remove recipe" | Member confirms (recipe deleted) or cancels (dialog closes) |
| Error | Error banner at top of form: "Couldn't save this recipe. Check your connection and try again." with a Retry action | Save or remove operation fails for a reason other than a conflict | Member taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline -- changes will be saved when you reconnect." at top; form remains editable; Save and Remove queue locally | Connectivity lost while the screen is open | Connectivity restored -- the queued change proceeds automatically and the standard success feedback appears |

## Validation Rules

Validation governed by FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules). See that spec for field-level and cross-field rules (ingredient/step length caps, minimum-one-ingredient requirement). This screen applies validation on field blur and on Save tap. The link well-formedness and weekly-import-limit rules do not apply here, since this screen edits an already-saved recipe rather than starting a new import.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (no unsaved changes) | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 |
| Successful save | FEAT-08.SPEC-002 (Recipe Detail View) | FEAT-08 |
| Successful removal | FEAT-08.SPEC-001 (Recipe Library Browse & Search) | FEAT-08 |
| Source link tap | External source page (opens outside the product) | -- |

## Data Model

**Creates:** None.
**Reads:** Recipe record -- name, ingredients (quantity + unit), steps, cook_time, origin, owning_household -- for the imported recipe being edited.
**Updates:** Recipe record -- name, ingredients, steps, cook_time, subject to FEAT-10.SPEC-007 validation; origin and owning_household are never changed by this screen.
**Deletes:** Recipe record -- hard delete, immediate and permanent, on confirmed removal; no cascade to Planned Meals that already used the recipe (their history stands unchanged, per the feature's Entity-Lifecycle Coverage Matrix).

## Business Rules

- Field validation (FEAT-10.SPEC-007) rules are enforced on every edit -- the member cannot save with invalid data, including fewer than one ingredient.
- Every accepted edit re-triggers the safety re-check (FEAT-10.SPEC-008) before the edited recipe can appear in a plan again (XBR-19).
- Removal is a hard delete with no retention window and no restore path -- re-adding the same recipe requires a fresh import via FEAT-10.SPEC-001, which FEAT-10.SPEC-006 treats as a new, non-duplicate import.
- Removing a recipe drops it from the household's candidate pool immediately; it does not affect Planned Meals that already used it, per the feature's Entity-Lifecycle Coverage Matrix.
- Concurrent edits to the same imported recipe resolve reject-with-refresh, per the Recipe entity's Contention note in feature-dependency-map.md.

## Edge Cases

- **Recipe changed by another household member between load and save** -- Save is rejected with the Conflict dialog: "This recipe was updated on another device while you were editing. Review the latest version before saving." with "View Latest" (reloads the record; local edits discarded after confirmation) and "Keep Editing" options. Resolution: reject-with-refresh, per the dependency map's Contention note for the Recipe entity.
- **Member navigates away with unsaved edits** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Member taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in saving state).
- **All ingredient lines removed** -- Save is blocked per FEAT-10.SPEC-007's minimum-one-ingredient rule; the ingredient list shows "At least one ingredient is required."
- **Recipe is removed by one member while another has it open in this edit screen** -- The second member's next Save or Remove attempt is rejected with "This recipe no longer exists in your household's pool." and they are returned to FEAT-08.SPEC-001; their in-progress edits are discarded.
- **Member removes a recipe that is currently used by a Planned Meal in the current week** -- The removal proceeds as specified (no cascade); the Planned Meal keeps its existing reference and cooked/past history unchanged, but the recipe is no longer available for any future candidate pool or re-selection, per the feature's Entity-Lifecycle Coverage Matrix.
- **Edited recipe now contains an ingredient the safety engine cannot recognize** -- The save still proceeds if all other validation passes; the recipe is excluded from every plan candidate path until the ingredient is clarified, per FEAT-10.SPEC-008 and XBR-01's fail-closed posture.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-002 (Recipe Detail View) | Navigation (inbound/outbound) | Member arrives from and returns to the recipe detail |
| FEAT-10.SPEC-007 (Recipe Import Validation & Rate Limit Rules) | References (inbound) | Field-level and cross-field validation applied on save |
| FEAT-10.SPEC-008 (Safety Re-check on Import Save/Edit) | Triggers (outbound) | Every accepted edit re-triggers the safety re-check |
| FEAT-08.SPEC-001 (Recipe Library Browse & Search) | Navigation (outbound) | Successful removal navigates here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| imported_recipe_removed | had_planned_meal_history (yes/no) | Removal is confirmed and the Recipe record is deleted | N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained for operational visibility into household-driven pool cleanup |

Note: `imported_recipe_edited` is emitted by FEAT-10.SPEC-008 (Safety Re-check on Import Save/Edit), which this screen's Save action triggers -- see that spec's Analytics and Success Signals section. This screen's own edit-save action carries no separate analytics event beyond the removal event above.

## Acceptance Criteria

**FEAT-10.SPEC-004-AC-01:** Given Maya opens Edit Imported Recipe on a recipe she previously imported, when the screen loads, then all fields are pre-filled with the recipe's current saved values.

**FEAT-10.SPEC-004-AC-02:** Given Sam edits the cook time of an imported recipe and taps Save, when validation passes, then the Recipe record is updated, a "Recipe updated" toast appears, and the safety re-check (FEAT-10.SPEC-008) runs.

**FEAT-10.SPEC-004-AC-03:** Given Maya removes all ingredient lines while editing, when she taps Save, then the ingredient list shows "At least one ingredient is required." and the save does not proceed.

**FEAT-10.SPEC-004-AC-04:** Given Sam is editing an imported recipe, when another household member updates the same recipe on a different device before Sam saves, then his Save is rejected with "This recipe was updated on another device while you were editing. Review the latest version before saving." and "View Latest" / "Keep Editing" options.

**FEAT-10.SPEC-004-AC-05:** Given Maya taps "Remove recipe", when the confirmation dialog appears, then it reads "Remove this recipe? It will be deleted from your household's recipe pool. This can't be undone." with "Remove" and "Cancel".

**FEAT-10.SPEC-004-AC-06:** Given Maya confirms removal of an imported recipe, when the deletion completes, then the recipe is deleted from the household's pool, a "Recipe removed" toast appears, and she is navigated to FEAT-08.SPEC-001.

**FEAT-10.SPEC-004-AC-07:** Given Sam cancels the removal confirmation dialog, when he taps "Cancel", then the dialog closes and the recipe remains unchanged on the edit form.

**FEAT-10.SPEC-004-AC-08:** Given a Planned Meal already used an imported recipe that Maya then removes, when the removal completes, then the Planned Meal's history is unaffected and the recipe is no longer available for future selection.

**FEAT-10.SPEC-004-AC-09:** Given the older-kid limited-login role (Later) does not see "Edit" on FEAT-08.SPEC-002 for an imported recipe, when they reach this screen directly, then it shows "Editing recipes isn't available on this profile." with a link back to the recipe detail.

**FEAT-10.SPEC-004-AC-10:** Given Riley (Operator) is viewing a household's recipe under an open Support Request, when Riley opens the recipe, then no Edit or Remove controls are shown.

**FEAT-10.SPEC-004-AC-11:** Given Maya has unsaved edits and taps the back arrow, when the confirmation dialog appears, then it reads "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-10.SPEC-004-AC-12:** Given Sam loses connectivity while editing, when he taps Save, then the banner "You're offline -- changes will be saved when you reconnect." appears and the change is queued.

**FEAT-10.SPEC-004-AC-13:** Given Maya edits a recipe to include an ingredient the safety engine cannot recognize, when the save completes, then the recipe stays saved but is excluded from every plan candidate path until the ingredient is clarified.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 9 (loaded, editing, validating, validation error, saving, conflict, removal confirmation, error, offline) | 9 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |
