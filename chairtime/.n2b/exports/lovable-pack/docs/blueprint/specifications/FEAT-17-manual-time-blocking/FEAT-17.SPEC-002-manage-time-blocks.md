---
document_type: spec
spec_type: screen
spec_id: FEAT-17.SPEC-002
spec_name: Manage Time Blocks
spec_slug: manage-time-blocks
parent_feature: FEAT-17
parent_feature_name: Manual Time Blocking
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: Manage Time Blocks

## Overview

**Name:** Manage Time Blocks
**ID:** FEAT-17.SPEC-002
**Type:** Screen
**Purpose:** Talia (Full) and Platform Operator Support (View-only) see the list of upcoming Time Blocks, with a plain empty state when none exist and an entry point to edit or remove each one.
**Parent Feature:** FEAT-17 -- Manual Time Blocking

## Scope and Non-Goals

**In Scope:**
- Listing upcoming Time Blocks in date order, one row per single-date block and one summary row per recurring pattern
- The "no time blocked" empty state
- Entry points to create a new block, edit an existing one, and remove one
- Support's read-only view of the same list

**Non-Goals:**
- The create/edit form itself -- owned by FEAT-17.SPEC-001, which this screen navigates to
- Executing the removal (the actual delete and slot-freeing) -- owned by FEAT-17.SPEC-007; this screen only provides the entry point and confirmation
- Showing past (already-expired) blocks -- excluded per the feature's own Data Notes: a removed or expired block carries no historical-record requirement of its own, so this list shows upcoming blocks only, not a history view
- Search or filtering across blocks -- excluded per scope-boundaries.md: a solo Pro's block set is small enough (Non-Functional Notes: data volumes) that no search capability is warranted

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard) | Talia opens her schedule navigation to manage blocks | None -- list loads current upcoming blocks |
| FEAT-17.SPEC-001 (Create/Edit Time Block) | Talia completes a save with no conflicts, or discards an edit | None -- list reloads to reflect any change |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) (Platform Support Read-Only Access) | Support opens this feature's view within an active Pro-account review | The Pro account under review; no action controls rendered |
| FEAT-17.SPEC-003 (Time Block Conflict Review) | Conflict review confirms with every remaining booking kept as an exception | None -- list reloads to reflect the saved block |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- her own blocks only | Add, edit, and remove her own blocks | -- |
| Platform Operator (Support) | Full list for the one Pro account under active review, including labels (Support has View-only access to Time Block, including its label, per the dependency map's Data Sensitivity note) | View only -- no add, edit, or remove controls rendered | Any write control is simply not present; there is no denial dialog because no write path is ever rendered for Support |
| The Client (Riley) | No | No | This screen exists only inside the Pro's signed-in application (or support's review session); Riley has no navigation path to it |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed or absent sign-in never reveals whether a Pro account exists (XBR-29) |
| Expired session | No | No | Redirected to the Pro sign-in screen (FEAT-29) on the next data refresh; no unsaved input exists on this screen to preserve, since it is a viewing/entry-point surface with no form state |

## Layout and Content

**Header:** Screen title "Time Blocks" with a back arrow (returns to FEAT-12) and, for the Pro only, an "Add a block" action button (right-aligned). Support sees the title and back arrow only -- no add action.

**Body:** A single vertically scrolling list of upcoming blocks in date order. Each **row** shows:
- Date (and, for a recurring pattern, "Repeats every {day of week}" instead of a single date, with the next occurrence date shown beneath)
- Time range (start--end)
- Label, when one exists (for the Pro, shown in full; for Support, shown in full per its View-only entitlement to the label)
- A remove affordance, visible to the Pro only

**Empty state:** When Talia has no upcoming blocks, the body shows a plain message: "No time blocked." with the "Add a block" action available from the header. Support sees the same message with no add action.

**Footer:** None -- Add is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column list as described, full width.
- **Medium size class and above:** List remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12 (Pro Daily Schedule Dashboard), or exit the review panel for Support | Screen closes | Standard backward transition |
| "Add a block" (Pro only) | Tap | Navigate to FEAT-17.SPEC-001 (Create/Edit Time Block) in create mode | Screen transitions | Standard navigation transition |
| Block row (Pro only) | Tap | Navigate to FEAT-17.SPEC-001 (Create/Edit Time Block) in edit mode, pre-filled with this block's or pattern's values | Screen transitions | Standard navigation transition |
| Block row (Support) | Tap | No action -- display-only for Support | None | Row shows a static, non-interactive treatment |
| Remove affordance (Pro only) | Tap | Prompts a confirmation dialog: "Remove this time block? The time becomes bookable again immediately." | Dialog appears | Confirmation dialog with "Remove" and "Cancel" |
| Remove confirmation -- "Remove" | Tap | Triggers FEAT-17.SPEC-007 (Time Block Removal & Expiry) | Row is removed from the list | List updates in place; if the list becomes empty, the empty state appears |
| Remove confirmation -- "Cancel" | Tap | No action | Dialog closes | Row remains unchanged |
| Pull-to-refresh / manual refresh | Swipe down / tap refresh | Re-fetches the upcoming block list | List reloads | Loading indicator during refresh |

### Accessibility Notes

- **Focus order:** Back arrow -> "Add a block" (Pro only) -> each block row in date order (within a row: date/pattern, time range, label, remove affordance) -> refresh control.
- **Dynamic-change announcements:** When a row is removed, its removal from the list is announced to assistive technology; when the list becomes empty, the "No time blocked." message is announced.
- **Keyboard alternatives:** Every action (add, open a row to edit, remove, confirm/cancel, refresh) is reachable without a pointer-only gesture; pull-to-refresh has an equivalent tappable refresh control.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (has blocks) | Rows populated as described in Layout and Content, in date order | Data fetch succeeds with at least one upcoming block | Data changes (new fetch, add, edit, or remove) |
| Empty | "No time blocked." message, with "Add a block" available (Pro only) | Data fetch succeeds with zero upcoming blocks | A block is created |
| Loading | A lightweight in-place indicator; on first-ever load, a brief full-screen lightweight loading indicator | Screen first opens, or a refresh is triggered | Data fetch completes (success or failure) |
| Error | Error banner: "Couldn't load your time blocks. Check your connection and try again." with a Retry control | Data fetch fails | Retry succeeds, or connectivity is restored and an automatic retry succeeds |
| Offline/Degraded | Banner "You're offline -- showing your most recently loaded time blocks." at the top; the most recently loaded list remains viewable read-only; add, edit, and remove controls are disabled with a note that they require reconnecting | Connectivity is lost while this screen is open, or the screen is opened while already offline with cached data available | Connectivity is restored -- the banner clears and a fresh fetch runs automatically |

## Validation Rules

This screen has no user-entry form fields. The remove confirmation is a binary choice with no field-level validation.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-12 (Pro Daily Schedule Dashboard) | FEAT-12 (Pro only); support exits its review panel |
| "Add a block" tap | FEAT-17.SPEC-001 (Create/Edit Time Block) | -- |
| Block row tap (Pro) | FEAT-17.SPEC-001 (Create/Edit Time Block) | -- |
| Remove confirmed | Stays on this screen; row removed in place | -- |

## Data Model

**Creates:** None directly -- creation is owned by FEAT-17.SPEC-001/FEAT-17.SPEC-004.
**Reads:** Time Block records -- start, end, recurrence, label, for the signed-in Pro Account (or the Pro account under Support's active review), grouped for display into one row per single-date block and one summary row per recurring pattern.
**Updates:** None directly -- editing is owned by FEAT-17.SPEC-001.
**Deletes:** Time Block record -- triggers FEAT-17.SPEC-007 on confirmed removal.

## Business Rules

- Support's access is read-only in every respect on this screen -- no add, edit, or remove control is ever rendered for Support (Access Matrix: Service & Availability Setup = View for Platform Operator).
- Removing a block never affects a confirmed booking that was kept as an exception to it -- the booking is untouched; only the block itself is deleted (Validation & Limits: a block cannot silently delete a conflicting booking).
- The list shows upcoming blocks only; an expired block is retired by FEAT-17.SPEC-007 and no longer appears here.
- Support's view-only rendering of this screen (no write control rendered, no field validation because there are no form fields) is governed by FEAT-17.SPEC-008 (Time Block Validation & Conflict-Handling Rules), which lists this screen as an enforcing spec for its authorization rules.

## Edge Cases

- **Talia removes the last remaining block** -- The list transitions directly to the "No time blocked." empty state.
- **Talia taps remove twice rapidly on the same row** -- The confirmation dialog appears once; a second tap while the dialog is open has no additional effect.
- **A block Talia is viewing expires while this screen is open (its end time passes during the session)** -- On the next refresh (automatic or pull-to-refresh) the expired block no longer appears; no separate expiry notice is shown here, since expiry is a routine background retirement, not an error.
- **Support opens this screen for a Pro account with no blocks** -- The same "No time blocked." message appears, with no add action shown.
- **Network failure during removal** -- The row remains in the list with an inline error: "Couldn't remove this time block. Try again." and the remove affordance remains available to retry.
- **Another session (e.g., a second signed-in device) already removed or edited the block Talia is confirming removal for** -- FEAT-17.SPEC-007 finds the record already gone (or already changed) and reports the already-gone outcome with no error; the row simply disappears from Talia's list on the next refresh, consistent with the dependency map's Contention note for Time Block (last-write-wins between the Pro's own sessions).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-001 (Create/Edit Time Block) | Navigation (outbound) | Add and edit entry points |
| FEAT-17.SPEC-007 (Time Block Removal & Expiry) | Triggers (outbound) | Confirmed removal triggers the delete |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound/outbound) | Entry point and back destination |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's read-only entry point during an account review |
| FEAT-17.SPEC-008 (Time Block Validation & Conflict-Handling Rules) | Governed by | Authorization rules for entry and Support's view-only rendering |

## Analytics and Success Signals

N/A -- this screen is a navigation and confirmation surface; the removal outcome this feature's Signals track (`time_block_removed`) is emitted by FEAT-17.SPEC-007, which performs the actual delete, not by this list screen.

## Acceptance Criteria

**FEAT-17.SPEC-002-AC-01:** Given Talia has two upcoming single-date blocks and one recurring pattern, when she opens Manage Time Blocks, then she sees three rows in date order, the recurring one labeled "Repeats every {day}" with its next occurrence date.

**FEAT-17.SPEC-002-AC-02:** Given Talia has no upcoming blocks, when she opens Manage Time Blocks, then she sees the message "No time blocked." with "Add a block" available.

**FEAT-17.SPEC-002-AC-03:** Given Talia is on Manage Time Blocks, when she taps "Add a block", then she is navigated to FEAT-17.SPEC-001 in create mode.

**FEAT-17.SPEC-002-AC-04:** Given Talia taps an existing block row, when the screen transitions, then FEAT-17.SPEC-001 opens in edit mode pre-filled with that block's values.

**FEAT-17.SPEC-002-AC-05:** Given Talia taps the remove affordance on a block row, when the confirmation dialog appears, then it reads "Remove this time block? The time becomes bookable again immediately." with "Remove" and "Cancel" options.

**FEAT-17.SPEC-002-AC-06:** Given Talia confirms removal of a block, when FEAT-17.SPEC-007 completes the delete, then the row disappears from the list immediately.

**FEAT-17.SPEC-002-AC-07:** Given Talia cancels the remove confirmation dialog, when she taps "Cancel", then the dialog closes and the row remains unchanged.

**FEAT-17.SPEC-002-AC-08:** Given Platform Operator Support is reviewing a Pro's account, when Support opens this screen, then every row is visible, including labels, with no remove or edit control rendered anywhere on the screen.

**FEAT-17.SPEC-002-AC-09:** Given a data fetch fails when Talia opens this screen, when the error appears, then she sees "Couldn't load your time blocks. Check your connection and try again." with a Retry control.

**FEAT-17.SPEC-002-AC-10:** Given Talia loses connectivity while viewing her block list, when the offline banner appears, then her most recently loaded list remains visible and every write control is disabled.

**FEAT-17.SPEC-002-AC-11:** Given Talia removes her only remaining block, when the removal completes, then the screen transitions directly to the "No time blocked." empty state.

**FEAT-17.SPEC-002-AC-12:** Given a network failure occurs while Talia confirms a removal, when the failure is reported, then the row remains in the list with the inline error "Couldn't remove this time block. Try again." and the remove affordance is still available.

**FEAT-17.SPEC-002-AC-13:** Given a block Talia removed had a confirmed booking kept as an exception, when the block is deleted, then that booking is left completely untouched.

**FEAT-17.SPEC-002-AC-14:** Given the Client (Riley) has no signed-in Pro or support session, when any attempt is made to reach this screen, then no such navigation path exists in Riley's experience.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 5 (loaded, empty, loading, error, offline/degraded) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |
