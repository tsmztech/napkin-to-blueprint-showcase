---
document_type: spec
spec_type: screen
spec_id: FEAT-16.SPEC-002
spec_name: Aisle Name Customization
spec_slug: aisle-name-customization
parent_feature: FEAT-16
parent_feature_name: Units, Currency & Locale Configuration
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Aisle Name Customization

## Overview

**Name:** Aisle Name Customization
**ID:** FEAT-16.SPEC-002
**Type:** Screen
**Purpose:** Maya renames or reorders the household's supermarket aisle groupings to match their local store; Sam and Riley see the current groupings without changing them.
**Parent Feature:** FEAT-16 -- Units, Currency & Locale Configuration

## Scope and Non-Goals

**In Scope:**
- Displaying the household's current aisle_names list in its saved order
- Renaming an existing aisle grouping
- Reordering aisle groupings (drag-to-reorder or equivalent up/down controls)
- Adding a new aisle grouping and removing one, within the aisle-name length and list rules FEAT-16.SPEC-003 defines
- Read-only display of the same list for roles without edit access

**Non-Goals:**
- Editing unit_system or currency -- handled by FEAT-16.SPEC-001 (Units & Currency Settings)
- Defining the aisle-name length limit or the default aisle groupings a new household starts with -- governed by FEAT-16.SPEC-003 (Locale Configuration Validation & Defaults); this screen only reads and enforces those rules
- Assigning individual grocery-list items to an aisle -- that mapping is Shared Grocery List's (FEAT-06) own ingredient-to-aisle logic; this screen only manages the names and order of the aisle groupings themselves
- Retroactively re-grouping an already-generated grocery list's items when aisle names change -- per the feature's Side-Effect Inventory, a saved aisle-name change applies to future grocery lists only, not to a list already generated before the change

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-16.SPEC-001 (Units & Currency Settings) | Organiser taps "Customize aisle names," or guided setup advances via Continue after confirming units and currency (wizard step 7 of 8; journey step 4 continuation) | Household id; on first entry from guided setup, the aisle_names list is pre-filled with FEAT-16.SPEC-003's default aisle groupings for the organiser to review |
| FEAT-01.SPEC-010 (Household Settings Hub) | Organiser opens the hub's "Units, currency & aisles" row, then taps "Customize aisle names" on FEAT-16.SPEC-001 | Household id; screen loads the household's current saved aisle_names list |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Rename, reorder, add, and remove aisle groupings; save | -- |
| Sam (Other Adult Member) | Full screen, view-only mode | None -- no edit, reorder, add, or remove controls rendered | Attempting to reach the screen with edit intent still opens it, but every row shows no rename/remove affordance and no drag handle; nothing is hidden, only non-interactive |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this role, so the screen is never reachable |
| Jordan (older kid, limited login -- Later) | No | No | Not part of the older-kid login's access; no navigation entry point is offered, and a direct link redirects to the older-kid login's home screen (the week's plan) |
| Riley (Operator, support) | Full screen, view-only mode | None -- no edit, reorder, add, or remove controls rendered | Reachable only through Operator Read-Only Support Access (FEAT-22) against a household with an open Support Request; outside that context the screen is not reachable at all |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on the Household Settings Hub (FEAT-01.SPEC-010), not directly on this screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- any in-progress unsaved rename, reorder, add, or remove is discarded |

## Layout and Content

**Header:** Screen title "Aisle Names" with a back arrow (returns to FEAT-16.SPEC-001) and, in guided setup, a "Finish" action instead of a back arrow, since this is the last configuration step of guided setup (step 7 of 8); Finish hands off to FEAT-01.SPEC-009, the Step 8 of 8 completion screen.

**Body:** A vertically ordered list of the household's aisle groupings, one row per aisle, in the household's currently saved order:
- Each row shows a drag handle (Maya only), the aisle name as an editable text field (Maya only; static text for view-only roles), and a remove control (Maya only).
- Below the list, an "Add aisle" action (Maya only) that appends a new, empty-named row at the end of the list.
- One line of helper text above the list: "These names group items on your shared grocery list. Drag to reorder, or tap a name to rename it."

For Sam and Riley (view-only mode), the same list renders with static aisle names in their saved order, no drag handles, no remove controls, and no "Add aisle" action.

**Footer:** A single "Save" action button (Maya only; not rendered for view-only roles).

### Responsive Behavior

- **Compact size class:** The aisle list stacks as full-width rows as described above; drag handles remain touch-sized; Save stays in the footer, always visible without scrolling past it.
- **Medium size class and above:** The list remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered; no structural change beyond width capping.
- **Long aisle lists:** The list scrolls within the body region rather than growing the page indefinitely; the header and footer remain fixed.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow (header, outside guided setup) | Tap | Navigate to FEAT-16.SPEC-001 | Screen closes | Standard transition |
| Finish (header, guided setup) | Tap | 1. Validate every pending aisle name and the pending order against FEAT-16.SPEC-003's rules -- the same validate step Save performs. 2. If valid, save the full aisle_names list (names and order) to the Household record -- the same save step Save performs. 3. Trigger FEAT-16.SPEC-004 so future grocery lists group items under the new names. 4. Complete setup and navigate to FEAT-01.SPEC-009 (Setup Complete & Next Steps). Finish behaves as Save followed by navigation, not as navigation alone. | Screen closes on success | Success: standard transition to FEAT-01.SPEC-009. Failure: the same inline error banner Save produces (see States/Error below), including the "Add at least one aisle before saving" error if the pending list is empty; guided setup remains on this screen, pending changes retained, until the save succeeds |
| Aisle name field (Maya only) | Type | Captures the new name as a pending edit for that row; does not save yet | Field shows entered text | Standard input focus state |
| Aisle name field (Maya only) | Blur | Validates the entered name against FEAT-16.SPEC-003's aisle-name length rule | Error state on the row if invalid | Inline error message below the row if the name is empty or exceeds the length limit |
| Drag handle (Maya only) | Drag and drop | Reorders the pending aisle list to the dropped position | List re-renders in the new order | Row visually lifts during drag and settles into its new position on drop |
| Remove control (Maya only) | Tap | Removes that row from the pending aisle list | Row disappears from the list | Brief inline confirmation "Aisle removed" with an "Undo" action available until Save is tapped |
| "Add aisle" action (Maya only) | Tap | Appends a new, empty-named row at the end of the pending list, focused for immediate typing | New row appears | Focus moves to the new row's name field |
| Save button (Maya only) | Tap | 1. Validate every pending aisle name and the pending order against FEAT-16.SPEC-003's rules. 2. If valid, save the full aisle_names list (names and order) to the Household record. 3. Trigger FEAT-16.SPEC-004 so future grocery lists group items under the new names. | Save button shows an inline saving confirmation | Success: inline confirmation "Saved" appears. Failure: inline error banner, described in States/Error below; the prior saved list remains active. |
| Save button (while saving) | Tap | No action -- debounced | None | Button remains in its inline saving state |
| Aisle name field, drag handle, remove control (Sam, Riley) | Tap / drag attempt | No action -- controls are not rendered for these roles | None | No feedback; the row shows static text only |

### Accessibility Notes

- **Focus order (Maya, edit mode):** Back arrow/Finish -> each aisle row's name field, in list order, followed by that row's remove control -> "Add aisle" action -> Save button.
- **Focus order (Sam, Riley, view-only mode):** Back arrow -> each aisle row's static name, in list order (Save is absent from the order since it is not rendered).
- **Reorder announcements:** A keyboard-based reorder alternative (e.g., "Move up" / "Move down" actions per row) is available alongside drag-and-drop, and each move is announced to assistive technology with the row's new position ("Produce moved to position 1 of 6").
- **Validation announcements:** When a name field enters an error state, its error message is announced and programmatically associated with the field.
- **Save feedback:** The "Saved" confirmation is announced on success; on failure, focus moves to the inline error banner.
- **Keyboard alternatives:** Every action on this screen, including reordering, is reachable by keyboard; drag-and-drop is never the only way to reorder.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Current aisle_names shown in saved order (or FEAT-16.SPEC-003's derived default groupings on first entry from guided setup) | Screen opens | Maya edits, reorders, adds, or removes a row |
| Editing (Maya) | Pending list differs from the last saved list; Save button enabled | Maya changes any name, order, addition, or removal | Maya taps Save, or navigates away (see Edge Cases) |
| Saving | Save button shows an inline saving confirmation; list remains interactive but a second Save tap is ignored | Maya taps Save | Save completes (success or failure) |
| Error | Inline error banner above the Save button: "Couldn't save your changes. Check your connection and try again." with a Retry action; the prior saved aisle_names list remains the active, displayed list | Save operation fails, whether triggered by the Save button or by Finish during guided setup | Maya taps Retry and the save succeeds, or Maya navigates away (pending changes are discarded) |
| Offline/Degraded | Banner "You're offline -- changes here will save once you're back online." at the top of the list; renaming, reordering, adding, and removing remain available to Maya, but the Save button is disabled with the same banner explaining why | Connectivity lost while the screen is open | Connectivity restored -- Save button re-enables; nothing was queued for automatic submission since aisle-name changes save instantly rather than in the background |

## Validation Rules

Validation governed by FEAT-16.SPEC-003 (Locale Configuration Validation & Defaults). See that spec for the aisle-name length limit and any list-level rules (minimum aisle count, duplicate-name handling). This screen applies the length rule on field blur and re-confirms all pending names and the pending order on Save before writing to the Household record.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-16.SPEC-001 (Units & Currency Settings) | -- |
| Finish tap (guided setup), after validate/save/trigger FEAT-16.SPEC-004 succeeds | FEAT-01.SPEC-009 (Setup Complete & Next Steps) | FEAT-01 (Household Setup & Member Profiles) |
| Successful save (outside guided setup) | Screen remains on FEAT-16.SPEC-002 with the updated list shown | -- |

## Data Model

**Creates:** None.
**Reads:** Household -- aisle_names (current saved list and order, or FEAT-16.SPEC-003's derived defaults on first entry from guided setup).
**Updates:** Household -- aisle_names (Maya only; the full list of names and their order is replaced on each save).
**Deletes:** None at the Household level -- an individual aisle row can be removed from the pending list before Save, but the aisle_names field itself has no independent delete path; it is removed only as part of the whole Household record's deletion, owned end to end by FEAT-18 (Account & Data Management).

## Business Rules

- Field-level validation for aisle names is governed by FEAT-16.SPEC-003 -- this screen enforces those rules but does not define them.
- A successful save triggers FEAT-16.SPEC-004 (Cross-Feature Value Conversion Rule) so that future grocery lists group items under the new aisle names; an already-generated grocery list keeps its existing grouping until the next list is built (Shared Grocery List, FEAT-06).
- XBR-11: the household's aisle names apply consistently everywhere they appear on the grocery list; this screen is one of the two places (with FEAT-16.SPEC-001) where locale settings are changed.
- Only Maya may rename, reorder, add, or remove aisle groupings; Sam and Riley may only view them, per FEAT-16.SPEC-003's Authorization Rules.

## Edge Cases

- **Maya navigates away with unsaved renames, reorders, additions, or removals** -- No confirmation dialog is shown; the pending changes are simply discarded and the screen reverts to the last saved list on next entry, consistent with the feature's own States field ("changes save instantly," meaning there is no draft state to protect).
- **Maya taps Save twice rapidly** -- The second tap is ignored while the first save is in progress (button in its inline saving state); no duplicate save is submitted.
- **Household's aisle_names changed by Maya on a second device between this screen's load and save** -- Save is accepted and applied last-write-wins per the dependency map's Contention note for the Household entity: whichever save reaches the server last becomes the active list, and a failed save keeps the prior list active rather than leaving a mixed state.
- **Maya renames an aisle to a name already used by another row** -- The save proceeds; duplicate aisle names are permitted (e.g., a household might intentionally use the same label for two physical sections), since the list is a free-text grouping name, not a unique key.
- **Maya removes every aisle row, leaving the list empty** -- Save is blocked with the inline error "Add at least one aisle before saving," since a grocery list with no aisle groupings has nowhere to place items; the same block applies when Maya taps Finish during guided setup with an empty pending list, since Finish runs the identical validate step.
- **Sam or Riley opens this screen while Maya is mid-edit on another device** -- Sam and Riley always see the last successfully saved list; they never see Maya's unsaved pending edits, since pending edits are local to Maya's own screen session.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-16.SPEC-001 (Units & Currency Settings) | Navigation (inbound/outbound) | Entry point and back-arrow destination; shared settings-area journey |
| FEAT-01.SPEC-009 (Setup Complete & Next Steps) | Navigation (outbound) | Guided setup's "Finish" completes the locale-configuration step and hands off here |
| FEAT-16.SPEC-003 (Locale Configuration Validation & Defaults) | References (outbound) | Aisle-name length limit, list rules, and default-value derivation |
| FEAT-16.SPEC-004 (Cross-Feature Value Conversion Rule) | Triggers (outbound) | A successful save triggers the rule that applies the new grouping to future grocery lists |
| FEAT-06.SPEC-002 (Grocery List Generation & Recalculation) | References (outbound) | Consumes the saved aisle_names to group future grocery-list items |

## Analytics and Success Signals

{No success-metrics.md metric names Units, Currency & Locale Configuration as its Connected Feature; the event below is recorded per product-features.md's Signals field for this feature, but it cites N/A since this configuration screen's downstream effect is measured through the metrics of consuming features (e.g., Shared Grocery List's own Grocery List Live-Update Trust metric), not through a metric of its own.}

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| aisle_name_customized | count of aisles renamed, reordered, added, and removed in this save | Save succeeds with any change to the aisle_names list | N/A -- no success-metrics.md metric is connected to Units, Currency & Locale Configuration |
| aisle_customization_save_failed | count of pending changes at the time of failure | Save fails and the error state is shown | N/A -- no success-metrics.md metric is connected to Units, Currency & Locale Configuration |

## Acceptance Criteria

**FEAT-16.SPEC-002-AC-01:** Given Maya is on the Aisle Name Customization screen, when she renames "Frozen" to "Frozen Foods" and taps Save, then the household's aisle_names list is saved with the new name, an inline "Saved" confirmation appears, and FEAT-16.SPEC-004 is triggered so future grocery lists use the new name.

**FEAT-16.SPEC-002-AC-02:** Given Maya drags the "Bakery" row above the "Produce" row and taps Save, then the saved aisle_names order reflects Bakery before Produce.

**FEAT-16.SPEC-002-AC-03:** Given Maya taps "Add aisle," when a new empty row appears, then it is focused for typing, and Save is blocked with an inline error if she attempts to save while that row's name is still empty.

**FEAT-16.SPEC-002-AC-04:** Given Maya removes every aisle row from the pending list, when she taps Save, then the inline error "Add at least one aisle before saving" appears and no save occurs.

**FEAT-16.SPEC-002-AC-05:** Given Maya taps Save and the save fails due to a lost connection, then the inline error banner "Couldn't save your changes. Check your connection and try again." appears with a Retry action, and the household's previously saved aisle_names list remains the active, displayed list.

**FEAT-16.SPEC-002-AC-06:** Given Maya loses connectivity while this screen is open, when she renames an aisle, then the rename still applies to the pending list on screen, but the Save button is disabled and the offline banner is shown.

**FEAT-16.SPEC-002-AC-07:** Given Sam (Other Adult Member) opens this screen, when it loads, then he sees the household's aisle_names in their saved order as static text, with no drag handles, remove controls, "Add aisle" action, or Save button.

**FEAT-16.SPEC-002-AC-08:** Given Riley (Operator, support) is viewing this screen through an open Support Request, when the screen loads, then Riley sees the same non-interactive display Sam sees.

**FEAT-16.SPEC-002-AC-09:** Given Jordan as a young kid profile has no login, when any attempt is made to reach this screen, then no such path exists.

**FEAT-16.SPEC-002-AC-10:** Given an unauthenticated visitor requests this screen's URL directly, then they are redirected to the sign-in screen, and after signing in they land on the Household Settings Hub (FEAT-01.SPEC-010) rather than directly on this screen.

**FEAT-16.SPEC-002-AC-11:** Given Maya reaches this screen from guided setup immediately after confirming units and currency, when the screen first loads, then the aisle list is pre-filled with FEAT-16.SPEC-003's default aisle groupings rather than appearing blank, and tapping Finish saves that list (as confirmed or as edited) and completes setup by navigating to FEAT-01.SPEC-009.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 5 (loaded, editing, saving, error, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
