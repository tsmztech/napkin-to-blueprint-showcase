---
document_type: spec
spec_type: screen
spec_id: FEAT-17.SPEC-001
spec_name: Create/Edit Time Block
spec_slug: create-edit-time-block
parent_feature: FEAT-17
parent_feature_name: Manual Time Blocking
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

# Screen Spec: Create/Edit Time Block

## Overview

**Name:** Create/Edit Time Block
**ID:** FEAT-17.SPEC-001
**Type:** Screen
**Purpose:** Talia sets a span of time on a specific date, or a recurring weekly pattern, with an optional private label, to create a new Time Block or edit an existing one.
**Parent Feature:** FEAT-17 -- Manual Time Blocking

## Scope and Non-Goals

**In Scope:**
- One form serving four combinations: create single-date, create recurring pattern, edit single-date, edit an existing recurring pattern
- Capturing start, end, an optional recurrence pattern, and an optional private label
- Inline field validation governed by FEAT-17.SPEC-008
- Handing the submitted values to FEAT-17.SPEC-004 on save
- The empty (new-block default), filling, saving, error, and offline/degraded states for this form

**Non-Goals:**
- Listing existing blocks to choose one to edit -- owned by FEAT-17.SPEC-002 (Manage Time Blocks), the entry point into this screen's edit mode
- Detecting or resolving a conflict with an existing booking -- owned by FEAT-17.SPEC-004 (detection) and FEAT-17.SPEC-003 (review); this screen only submits the values and reacts to where FEAT-17.SPEC-004 routes it next
- Generating the individual dated occurrences a recurring pattern implies -- owned by FEAT-17.SPEC-005; this screen only captures the pattern definition
- Multi-staff or shared-calendar blocking -- excluded per scope-boundaries.md SC-01: the product is strictly single-operator, so a block belongs to exactly one Pro Account with no second operator to coordinate against

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard) | Talia taps the "block time" entry point on the schedule view | None -- form starts empty in create mode |
| FEAT-17.SPEC-002 (Manage Time Blocks) | Talia taps "add a block" | None -- form starts empty in create mode |
| FEAT-17.SPEC-002 (Manage Time Blocks) | Talia taps an existing block row | The selected Time Block's start, end, recurrence pattern (if any), and label are loaded into the form in edit mode |
| FEAT-02.SPEC-001 (Availability & Working Hours Setup) | Talia wants to close a single day rather than change her recurring working hours | None -- form starts empty in create mode, pre-focused on the date field |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- her own blocks only | Create and edit her own blocks | -- |
| The Client (Riley) | No | No | This screen exists only inside the Pro's signed-in application; Riley has no navigation path to it at all -- Riley's own experience of a block is limited to its absence from the bookable slot list on the public booking page |
| Platform Operator (Support) | No | No | Support's read-only surface for this feature is the Manage Time Blocks list (FEAT-17.SPEC-002); this create/edit form is outside support's granted session scope, so it is never reachable from the support view |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed or absent sign-in never reveals whether a Pro account exists (XBR-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- entered form values are preserved locally and restored on the form after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Block Time" (create mode) or "Edit Time Block" (edit mode), with a back arrow (returns to the entry screen without saving) and a "Save" action button (right-aligned).

**Body:** A single-column form with the following fields, in order:
- **Date** (date input, required) -- the date of the block, or the first occurrence date when a recurrence pattern is set
- **Start time** (time input, required)
- **End time** (time input, required)
- **Timezone note** -- a non-interactive line beneath the time fields showing "Times are in your account timezone ({Pro Account timezone})," read from the Pro Account so the block is entered and interpreted consistently with the Pro's other schedule data
- **Repeats** (toggle, default off) -- when turned on, reveals a **day-of-week selector** (single selection, defaulting to the day-of-week of the chosen Date) describing the recurring pattern (e.g., "every Sunday")
- **Label** (text input, optional) -- placeholder text "Private note (only you see this)"; a static caption beneath the field states "Clients never see this label."

Editing an existing recurring pattern shows the same Repeats toggle already on, with its day-of-week selector pre-filled; editing a single already-generated occurrence (opened from a specific dated row on FEAT-17.SPEC-002) shows the Repeats toggle off and hidden entirely, since a single generated occurrence is edited as its own dated instance, not as the pattern.

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described above, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the entry screen (FEAT-12, FEAT-17.SPEC-002, or FEAT-02.SPEC-001) discarding unsaved input | Screen closes | If fields were filled, a confirmation dialog appears first (see Edge Cases) |
| Date input | Select a date | Captures the date; if Repeats is on, updates the day-of-week selector's default to match | Field shows selected date | Standard date-picker feedback |
| Start time input | Select a time | Captures start time | Field shows selected time | Standard input feedback |
| End time input | Select a time | Captures end time; triggers end-after-start validation via FEAT-17.SPEC-008 | Field shows selected time | Error state and message if end is not after start |
| Repeats toggle | Tap | Reveals or hides the day-of-week selector | Form layout expands/contracts | Day-of-week selector animates into or out of view |
| Day-of-week selector | Select a day | Captures the recurrence pattern's weekday | Selector shows chosen day | Selected day highlighted |
| Label input | Type | Captures free text, validated via FEAT-17.SPEC-008 (length) | Field shows entered text | Character count shown as the limit is approached |
| Save button | Tap | 1. Validate all fields via FEAT-17.SPEC-008. 2. If valid, submit to FEAT-17.SPEC-004 (Time Block Save Commit & Conflict Detection). | Button shows loading state during submission | See States: Saving, then Error or navigation per FEAT-17.SPEC-004's outcome |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Date -> Start time -> End time -> Repeats toggle -> Day-of-week selector (when visible) -> Label -> Save.
- **Validation announcements:** When a field enters an error state (end-before-start, label too long), its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** A successful commit's confirmation is announced; on validation or conflict-routing outcomes, focus moves to the first field in error or to the routed screen's heading.
- **Keyboard alternatives:** Every action, including the Repeats toggle and day-of-week selection, is reachable by keyboard; date and time inputs offer a typed-entry alternative to any picker gesture.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading (edit mode only) | N/A -- a solo Pro's block set is small enough that the existing block's values load instantly, with no progress indicator needed (Non-Functional Notes: data volumes) | Screen opens in edit mode, before the selected block's values are available | Values load (effectively immediately) and the Loaded state appears |
| Empty (create, default) | All fields empty except Date, which defaults to today; Save enabled | Screen opens in create mode | Talia begins editing any field |
| Loaded (edit, default) | Fields pre-filled from the selected Time Block; Save enabled | Screen opens in edit mode from FEAT-17.SPEC-002 | Talia begins editing any field |
| Filling | Fields contain Talia's input; inline validation runs per FEAT-17.SPEC-008 | Talia types or selects in any field | Talia taps Save or navigates away |
| Saving | Save button shows a loading indicator, fields disabled | Talia taps Save and all inline validation passes | FEAT-17.SPEC-004 returns an outcome (commit, conflict routing, or failure) |
| Error | Error banner at the top of the form: "Couldn't save this time block. Check your connection and try again." with a Retry control; entered values remain in the fields | FEAT-17.SPEC-004 reports a save failure | Talia taps Retry and the save succeeds, or she navigates away |
| Offline/Degraded | Plain message "Connect to the internet to save a time block." replaces the Save button's active state; fields remain visible and editable but Save is disabled | Connectivity is lost while the screen is open, or the screen is opened while already offline | Connectivity is restored -- Save re-enables |

## Validation Rules

Validation governed by FEAT-17.SPEC-008 (Time Block Validation & Conflict Handling Rules). See that spec for all field-level rules, including end-after-start and label length. This screen applies validation on field blur (Date, Start time, End time, Label) and again on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (no unsaved changes) | FEAT-17.SPEC-002 (Manage Time Blocks), FEAT-12, or FEAT-02.SPEC-001, matching the entry source | Varies by entry source |
| Save succeeds with no conflicting bookings | FEAT-17.SPEC-002 (Manage Time Blocks) | -- |
| Save finds one or more conflicting confirmed bookings | FEAT-17.SPEC-003 (Time Block Conflict Review) | -- |
| Discard confirmation -- "Discard" chosen | Same destination as the back arrow, per entry source | Varies by entry source |

## Data Model

**Creates:** Time Block record -- start, end, recurrence (optional), label (optional), owned by the signed-in Pro Account. Setting a recurrence pattern also triggers FEAT-17.SPEC-005 to generate the pattern's future dated occurrences.
**Reads:** Time Block record (in edit mode, all fields, to pre-fill the form); Pro Account -- timezone field only, to label the time inputs consistently with the Pro's other schedule data.
**Updates:** Time Block record -- start, end, recurrence, label, on an existing block Talia opens to edit.
**Deletes:** None -- removal is owned by FEAT-17.SPEC-002 and FEAT-17.SPEC-007.

## Business Rules

- End-after-start and label-length validation are enforced by FEAT-17.SPEC-008 -- Talia cannot save with invalid values.
- Saving never commits or rejects a conflict decision itself; FEAT-17.SPEC-004 determines whether the save completes immediately or routes to FEAT-17.SPEC-003 (XBR-11: setup changes never silently affect a confirmed booking).
- Editing an existing recurring pattern's span, day-of-week, or label affects that pattern's future not-yet-elapsed occurrences (regenerated by FEAT-17.SPEC-005); already-elapsed occurrences are historical and are never altered.
- A block cannot be created or edited without connectivity -- this is a setup-style action requiring a live connection for correctness (ASMP-27).

## Edge Cases

- **Talia navigates away with unsaved changes** -- Confirmation dialog: "Discard this time block?" with "Discard" and "Keep Editing" options.
- **Talia taps Save twice rapidly** -- The second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner as described in States; entered values are preserved and Talia can retry without re-entering anything.
- **Talia turns Repeats on after already entering a Date in the past relative to today** -- The date field itself is not restricted to future dates (a block can start today), but the day-of-week selector always derives from whichever Date is currently entered.
- **Talia edits a single already-generated occurrence (not the parent pattern) and turns Repeats on** -- Not offered: the Repeats toggle is hidden entirely for a single generated occurrence, since converting one occurrence into a new pattern is not a supported edit; Talia would instead create a new recurring block from this screen's create mode.
- **Concurrent edit -- another session (e.g., a second signed-in device) removes or edits this same block while this form is open** -- On Save, FEAT-17.SPEC-004 re-validates against the current record; if the block no longer exists, the save is rejected with "This time block was removed. Start over?" (Discard returns to FEAT-17.SPEC-002); if the block was edited elsewhere, the last commit to complete wins (last-write-wins, per the dependency map's Contention note for Time Block) and this form's save simply overwrites it, matching the block's own single-owner editing model.
- **Label left empty** -- Save proceeds; the block carries no label and displays with only its time range on FEAT-17.SPEC-002 and the Pro's schedule (FEAT-12).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-002 (Manage Time Blocks) | Navigation (inbound/outbound) | Entry point for create and edit; destination after a conflict-free save |
| FEAT-17.SPEC-003 (Time Block Conflict Review) | Navigation (outbound) | Destination when the save finds conflicting bookings |
| FEAT-17.SPEC-004 (Time Block Save Commit & Conflict Detection) | Triggers (outbound) | Save submits the form's values for validation, conflict detection, and commit |
| FEAT-17.SPEC-005 (Recurring Time Block Occurrence Generation) | Triggers (outbound) | A saved or edited recurrence pattern triggers occurrence generation |
| FEAT-17.SPEC-008 (Time Block Validation & Conflict Handling Rules) | References (inbound) | Field-level validation rules applied to this form |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound) | "Block time" entry point |
| FEAT-02.SPEC-001 (Availability & Working Hours Setup) | Navigation (inbound) | Directs a one-off closed day here instead of editing recurring hours |

## Analytics and Success Signals

N/A -- this screen only captures and submits values; the block-creation outcome this feature's Signals track (`time_block_added`, `time_block_conflict_flagged`) is emitted by the automation that actually commits or routes the data (FEAT-17.SPEC-004), not by this entry form. Tracking submission here would double-count the same event the commit automation already emits.

## Acceptance Criteria

**FEAT-17.SPEC-001-AC-01:** Given Talia is on the Create Time Block screen with an empty form, when she sets tomorrow's date, a start and end time, and taps Save, then FEAT-17.SPEC-004 validates and commits the block, and Talia is returned to FEAT-17.SPEC-002 with the new block visible.

**FEAT-17.SPEC-001-AC-02:** Given Talia is on the Create Time Block screen, when she sets an end time earlier than the start time, then the end time field shows an error state with the message defined by FEAT-17.SPEC-008 and Save does not proceed.

**FEAT-17.SPEC-001-AC-03:** Given Talia turns on the Repeats toggle, when the day-of-week selector appears, then it defaults to the day-of-week of the currently entered Date.

**FEAT-17.SPEC-001-AC-04:** Given Talia sets a recurring pattern and taps Save, when the save commits, then FEAT-17.SPEC-005 generates the pattern's future dated occurrences.

**FEAT-17.SPEC-001-AC-05:** Given Talia enters a label describing a personal commitment and saves, when the block is later viewed on FEAT-17.SPEC-002 or the Pro's schedule (FEAT-12), then the label is visible only to Talia and never appears on any client-facing screen.

**FEAT-17.SPEC-001-AC-06:** Given Talia's new block's time range overlaps an existing confirmed booking, when she taps Save, then she is routed to FEAT-17.SPEC-003 (Time Block Conflict Review) instead of seeing an immediate success confirmation.

**FEAT-17.SPEC-001-AC-07:** Given Talia is on the Create Time Block screen with unsaved input, when she taps the back arrow, then a confirmation dialog appears asking "Discard this time block?" with "Discard" and "Keep Editing" options.

**FEAT-17.SPEC-001-AC-08:** Given Talia taps Save and it is in progress, when she taps Save again, then the second tap has no effect and the button remains in its loading state.

**FEAT-17.SPEC-001-AC-09:** Given Talia's save fails due to a connectivity error, when the error banner appears, then her entered values remain in every field and she can retry without re-entering anything.

**FEAT-17.SPEC-001-AC-10:** Given Talia loses connectivity while the form is open, when she looks at the Save button, then it is disabled and the message "Connect to the internet to save a time block." is shown.

**FEAT-17.SPEC-001-AC-11:** Given Talia opens an existing single-date block from FEAT-17.SPEC-002, when the form loads, then the Repeats toggle is hidden and the fields are pre-filled with that block's start, end, and label.

**FEAT-17.SPEC-001-AC-12:** Given Talia opens an existing recurring pattern from FEAT-17.SPEC-002 and changes its end time, when she saves, then the pattern's future not-yet-elapsed occurrences are regenerated with the new end time, and already-elapsed occurrences are unchanged.

**FEAT-17.SPEC-001-AC-13:** Given Talia leaves the Label field empty and saves, when the block is created, then it carries no label and displays with only its time range.

**FEAT-17.SPEC-001-AC-14:** Given Talia is filling the form, when she looks below the time fields, then she sees her account's timezone stated so she can confirm the entered times are interpreted correctly.

**FEAT-17.SPEC-001-AC-15:** Given Talia arrives at this screen from FEAT-02.SPEC-001 to close a single day, when the form opens, then it is in create mode with an empty form pre-focused on the date field.

**FEAT-17.SPEC-001-AC-16:** Given another session removed the block Talia currently has open for editing, when she taps Save, then she sees "This time block was removed. Start over?" with the option to discard back to FEAT-17.SPEC-002.

**FEAT-17.SPEC-001-AC-17:** Given the Client (Riley) has no signed-in Pro session, when any attempt is made to reach this screen's URL, then no such navigation path exists in Riley's experience -- Riley's only exposure to a block is its absence from the bookable slot list.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 7 (loading, empty, loaded, filling, saving, error, offline/degraded) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |
