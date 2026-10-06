---
document_type: spec
spec_type: screen
spec_id: FEAT-06.SPEC-001
spec_name: Grocery List
spec_slug: grocery-list
parent_feature: FEAT-06
parent_feature_name: Shared Grocery List
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 20
---

# Screen Spec: Grocery List

## Overview

**Name:** Grocery List
**ID:** FEAT-06.SPEC-001
**Type:** Screen
**Purpose:** Household member views the current week's combined, aisle-grouped grocery list and ticks, adds, edits, removes, or marks "already have it" on items, live and while offline.
**Parent Feature:** FEAT-06 -- Shared Grocery List

## Scope and Non-Goals

**In Scope:**
- Displaying the current week's Grocery List, grouped by the household's configured aisles
- Ticking and unticking items
- Adding a manual item
- Editing an item's quantity
- Removing an item
- Marking a plan-derived item "already have it"
- Full read/tick/add/edit/remove usability while offline, with automatic sync on reconnect

**Non-Goals:**
- Household aisle names and unit-system configuration -- owned by Units, Currency & Locale Configuration (FEAT-16); this screen only consumes those settings, per this feature's Non-Goals and XBR-11
- Online grocery ordering or checkout handoff -- deferred to Online Grocery Ordering Handoff (FEAT-20, Later), per scope-boundaries.md's deferral note; this screen only carries the future hand-off point
- Browsing a past week's archived list -- owned by Weekly Plan History (FEAT-19); this screen shows only the current week's list
- Manual item name validation and duplicate-merge logic -- defined by FEAT-06.SPEC-007; this screen calls it, not re-defines it
- Undo for a removed item -- excluded per feature-overview.md Non-Goals: removal is an immediate, hard delete with no restore path, a deliberate simplicity decision prioritizing the product's "instant" responsiveness goal over a recovery flow for a low-stakes, easily-corrected action

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-03 (AI Weekly Dinner Plan Generation) -- Weekly Plan screen | Household member opens the list from an approved or auto-adopted week's plan | Current week reference; list is already populated by FEAT-06.SPEC-002 |
| FEAT-23 (Manual Weekly Planning) -- week-being-built screen | Household member opens the list as picks fill the week | Current week reference; list fills incrementally as SPEC-002 recalculates on each pick |
| FEAT-15 (Member Onboarding) -- first-use landing | A new member lands directly on the current list after their first sign-in | None -- list loads to its current, already-populated state |
| Default entry (persistent navigation) | Household member navigates to the list directly at any time | None -- list loads to its current state |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | All actions: tick, add, edit quantity, remove, already-have-it | -- |
| Sam (Other Adult Member) | Full screen | All actions: tick, add, edit quantity, remove, already-have-it | -- |
| Jordan (young kid profile, no login -- MVP) | No -- this profile has no login and cannot reach any screen | No | This profile has no sign-in; the screen is never reached, not merely restricted |
| Jordan (older kid, limited login -- Later) | Full screen | All item actions: tick, add, edit quantity, remove, already-have-it (per FEAT-06.SPEC-009); no household aisle-setting controls are ever shown to this role, on any role | -- |
| Riley (Operator, support -- from v1) | Full screen, read-only, and only while an open Support Request exists for the household (XBR-14, FEAT-06.SPEC-009) | View only -- no tick, add, edit, remove, or already-have-it controls are rendered | Outside an open Support Request, Riley cannot open the household's list at all: the attempt is blocked and no household data is shown. While viewing, tapping where a control would be for another role does nothing -- no controls are present to tap |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on this screen if it was their intended destination |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- any in-progress add or edit text is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Grocery List" with the current week's date range shown beneath it. A "Hand off list" control appears in the header, shown only to Maya and Sam and only while FEAT-20.SPEC-003 evaluates the household's region as having an available online-ordering capability; the control is never shown to Jordan (either profile), Riley, or an ineligible region, and its eligibility is re-evaluated fresh on FEAT-20.SPEC-001's own load rather than trusted from this screen (per FEAT-20.SPEC-003). When the device is offline, a persistent banner appears directly below the header: "You're offline -- changes will sync when you reconnect." (see States).

**Body:** A single-column list grouped into aisle sections, ordered per the household's configured `aisle_grouping` (FEAT-16). Each aisle section has a non-interactive section header (the aisle name) followed by its item rows. Each item row shows:
- A tick control (checkbox) -- ticked items render with a strikethrough on the ingredient name
- The ingredient name and its `quantity_and_unit`
- An "added by {member name}" label, shown only on manual-origin items
- An overflow control (three-dot menu) offering: "Edit quantity", "Remove" (all items), and "Already have it" (plan-derived items only)

Riley's read-only view renders the same rows with the tick control, "added by" label, and overflow control all omitted -- items display as static text.

**Footer:** A persistent "Add item" row: a single-line text input with placeholder "Add an item" and an adjacent "Add" button, always visible at the bottom of the screen for one-handed, in-store use (per the product's accessibility posture for primary actions). Not shown to Riley.

### Responsive Behavior

- **Compact size class (phone, primary use case):** Single-column list as described, full width; the Add-item row is pinned to the bottom of the viewport so it remains reachable one-handed while scrolling.
- **Medium size class and above:** List remains single-column, capped at a consistent platform-wide content width and horizontally centered; aisle sections and the Add-item row are otherwise unchanged.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Tick control | Tap | Toggles the item's `ticked` state; propagated via FEAT-06.SPEC-005, subject to FEAT-06.SPEC-008's idempotent-tick rule | Row shows strikethrough (ticked) or plain text (unticked) | Immediate local update; no confirmation dialog |
| Add-item input | Type | Captures free text for a new item name | Input shows entered text | Standard input focus state |
| Add button | Tap | Submits the typed name to FEAT-06.SPEC-007 (Manual Item Validation & Duplicate Merge) | New row appears (new line) or an existing line updates (merge), per SPEC-007's outcome | Success: input clears, new/updated row appears in its aisle section. Failure: inline error below the input (see Validation Rules) |
| Overflow > Edit quantity | Tap | Opens an inline quantity field pre-filled with the current `quantity_and_unit` | Row switches to an editable quantity field with a confirm and cancel control | Standard input focus state |
| Edit quantity > Confirm | Tap | Saves the new quantity, subject to FEAT-06.SPEC-008's last-write-wins conflict rule | Row shows the updated quantity | Immediate local update; no confirmation dialog |
| Overflow > Remove | Tap | Immediately and permanently deletes the Grocery List Item -- no confirmation dialog, per this screen's Non-Goals | Row disappears from the list | Immediate; no undo offered |
| Overflow > Already have it (plan-derived items only) | Tap | Triggers FEAT-06.SPEC-003 ("Already Have It" Handling) | Row disappears from the list | Toast: "Marked as already have it -- added to your pantry." with a "View Pantry" action (Maya, Sam -- per FEAT-05.SPEC-007's Pantry Input gating) or "Marked as already have it." with no action (Jordan, older kid -- no pantry access) |
| Toast > View Pantry (Maya, Sam only) | Tap | Navigates to FEAT-05.SPEC-001 (Pantry List & Item Entry) | Screen navigates away | Animated transition to the Pantry List screen |
| Hand off list control (header, Maya/Sam only, region-eligible per FEAT-20.SPEC-003) | Tap | Navigates to FEAT-20.SPEC-001 (Grocery Handoff Screen) | Screen navigates away | Animated transition to the Handoff screen |
| Aisle section header | -- | Display only, non-interactive -- groups the rows beneath it | None | None |
| "Added by" label | -- | Display only, non-interactive | None | None |

### Accessibility Notes

- **Focus order:** Offline banner (when present) -> aisle sections in list order, each row's tick control -> ingredient name/quantity (read-only) -> overflow control -> Add-item input -> Add button.
- **Dynamic-change announcements:** A tick toggling, a new item appearing, an item being removed, and the "Already have it" toast are each announced to assistive technology as they occur, so a member using a screen reader hears the live list change without re-scanning it.
- **Keyboard alternatives:** Every action on this screen (tick, add, edit quantity, remove, already-have-it) is reachable by keyboard focus and activation; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (pre-plan) | Short message: "Your list will appear once this week's plan is ready. You can add items now." plus the Add-item row remains usable | No Grocery List exists yet for the household's current week | FEAT-06.SPEC-002 creates the list (plan generated or first pick made) |
| Populating | Newly generated items appear within a couple of seconds, with an inline loading indicator at the top of the list | A plan is generated or picked and FEAT-06.SPEC-002 is writing plan-derived lines | Generation completes and the full aisle-grouped list renders |
| Populated (default) | Full aisle-grouped list with all current items | List has one or more items | List becomes empty again (all items removed/ticked-and-cleared is not a state -- ticked items remain visible until removed) |
| Error | A failed tick, add, edit, or remove retries automatically in the background; an error surfaces only if it cannot eventually succeed, as a banner: "Couldn't save your change. Check your connection and try again." with a Retry option | A write action ultimately fails after background retries | Member taps Retry and the action succeeds, or the member dismisses the banner |
| Offline/Degraded | Persistent banner: "You're offline -- changes will sync when you reconnect." Full read, tick, add, edit, and remove remain usable; every change queues locally | Connectivity is lost while the screen is open, or the screen opens while already offline | Connectivity returns -- queued changes sync automatically (FEAT-06.SPEC-005) and the banner disappears |

## Validation Rules

**Option A -- Reference Logic/Rule spec:**
Manual item name validation and duplicate-merge behavior are governed by FEAT-06.SPEC-007. See that spec for the exact rules and error messages, applied on the Add-item input on submit.

**Option B -- Inline (quantity edit, not covered by SPEC-007):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Quantity edit field | Must not be left blank | On confirm | "Quantity can't be empty." |

## Navigation Out

This screen is primarily a persistent destination reached from the app's navigation shell (out of scope for this spec), not a step in a linear flow, and every action on this screen (tick, add, edit, remove) resolves inline without leaving the screen. It carries exactly two optional navigation-out targets, both gated to roles with the underlying access and both optional for the member to take:

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Hand off list" header control tap (Maya, Sam only, shown only while the household's region is handoff-eligible per FEAT-20.SPEC-003) | FEAT-20.SPEC-001 (Grocery Handoff Screen) | FEAT-20 (Online Grocery Ordering Handoff) |
| "View Pantry" action on the "already have it" toast (Maya, Sam only -- Jordan (older kid)'s toast carries no pantry mention and no action, since this role's Pantry Input access is None per FEAT-05.SPEC-007) | FEAT-05.SPEC-001 (Pantry List & Item Entry) | FEAT-05 (Pantry-Aware Suggestions) |

"Already have it" itself still triggers FEAT-06.SPEC-003 as a background automation on this screen, not a navigation; the row disappears here regardless of role, and only the follow-on toast action -- offered to Maya and Sam only -- leaves the screen.

## Data Model

**Creates:** Grocery List Item (manual) -- `ingredient_name` from the Add-item input, `quantity_and_unit` left blank unless later edited, `aisle` auto-assigned by matching `ingredient_name` against the household's configured aisle categories (FEAT-16), falling back to a household catch-all "Other" aisle when no match is found, `origin` set to "manual", `ticked` defaults to false. Attribution ("added by") is the creating member.

**Reads:** Grocery List -- `week`, `status`. Grocery List Item -- all fields, for every item on the current list. Household -- `aisle_grouping`, `unit_system` (via FEAT-16) to order sections and format quantities.

**Updates:** Grocery List Item -- `ticked` (tick/untick), `quantity_and_unit` (edit), subject to FEAT-06.SPEC-008's conflict rules; plan-derived lines are also rewritten in place by FEAT-06.SPEC-002's recalculation, shown live on this screen.

**Deletes:** Grocery List Item -- on manual removal (any origin) or as a side effect of FEAT-06.SPEC-003 ("Already have it").

## Business Rules

- XBR-03: the list is always derived from the current plan; every pick, change, swap, accepted suggestion, or safety removal updates the list immediately via FEAT-06.SPEC-002, visible on this screen without the member refreshing.
- XBR-04: pantry-excluded ingredients never appear as plan-derived lines (governed by FEAT-06.SPEC-006); "already have it" can add an item to the pantry in the same tap (FEAT-06.SPEC-003).
- XBR-11: aisle grouping and quantity units follow the household's configuration (FEAT-16) consistently across every rendered line.
- Manual item validation and duplicate merge are governed by FEAT-06.SPEC-007 -- the member cannot bypass them from this screen.
- Offline and concurrent-edit behavior is governed by FEAT-06.SPEC-008 -- this screen never resolves a conflict itself.
- Access to every action on this screen is governed by FEAT-06.SPEC-009 -- this screen's Access and Visibility table above is drawn from it and must not diverge.
- The "Hand off list" header control's visibility (region and role) is governed entirely by FEAT-20.SPEC-003 -- this screen never independently determines regional online-ordering availability or handoff role eligibility, and re-evaluates nothing itself; FEAT-20.SPEC-001 re-checks eligibility fresh on its own load regardless of whether the control was shown here.
- The "View Pantry" action on the "already have it" toast is offered only to a role with Pantry Input access (Maya, Sam), per FEAT-05.SPEC-007's Authorization Rules -- Jordan (older kid, limited login) receives the plain "Marked as already have it." toast with no pantry mention and no action, since a Pantry Item is never created on this role's behalf.

## Edge Cases

- **Household member taps tick twice rapidly, including once while briefly offline** -- The second tap on an already-ticked line is a no-op once synced; per FEAT-06.SPEC-008's idempotent-tick rule, the item settles into exactly one ticked state, not a double-toggle.
- **Another member ticks or edits the same item while this screen is open** -- The screen updates live via FEAT-06.SPEC-005; no conflict dialog appears to either member for a tick (idempotent) or a quantity edit (last-write-wins per FEAT-06.SPEC-008) -- the concurrent-edit outcome resolves silently and both screens converge to the same state.
- **Household member adds an ingredient that is already on the list (manual or plan-derived)** -- FEAT-06.SPEC-007 merges the entry into the existing line rather than creating a duplicate; the member sees the existing line's quantity update (manual-manual match) or a confirmation that the item is already on the list from the week's plan (manual-vs-plan-derived match).
- **Household member removes an item that another member just ticked moments earlier** -- Removal proceeds regardless of ticked state; the household has decided the item is no longer needed.
- **Network failure while adding an item** -- The add is held locally and retried automatically in the background; the new row appears optimistically and is confirmed once the write succeeds, or shows the Error state if it ultimately fails.
- **Member navigates away with an in-progress quantity edit** -- The edit field's typed value is not saved unless Confirm was tapped; navigating away discards the in-progress edit, consistent with this screen having no draft-persistence requirement for a low-stakes, single-field edit.
- **Member navigates away or backgrounds the app with unsubmitted text in the Add-item input (not tapped Add)** -- The typed text is discarded, not preserved and not restored, when the member returns to the screen or reopens the app; this is distinct from the Expired-session case (AC-14), where in-progress Add-item text is preserved specifically across a forced re-authentication interruption, not across an ordinary navigate-away or background/foreground cycle. This keeps the Add-item input consistent with the quantity-edit field above: no draft-persistence requirement for a low-stakes, single-field input outside the session-expiry recovery path.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-002 (Grocery List Generation & Recalculation) | Affects (inbound) | Writes and rewrites plan-derived lines shown on this screen |
| FEAT-06.SPEC-003 ("Already Have It" Handling) | Triggers (outbound) | "Already have it" overflow action fires this automation |
| FEAT-06.SPEC-004 (Week Rollover & Carryover) | Affects (inbound) | Seeds this screen's list with carried-over manual items at week start |
| FEAT-06.SPEC-005 (Live Grocery List Sync) | References (bidirectional) | Every write on this screen propagates through this integration; incoming changes from other devices render here live |
| FEAT-06.SPEC-006 (Ingredient Consolidation & Quantity Derivation) | References (inbound) | Computes the plan-derived line values this screen displays |
| FEAT-06.SPEC-007 (Manual Item Validation & Duplicate Merge) | Triggers (outbound) | Add-item action calls this spec's validation and merge rules |
| FEAT-06.SPEC-008 (Offline Conflict Resolution Rules) | References (bidirectional) | Governs tick, edit, and offline/reconnect behavior on this screen |
| FEAT-06.SPEC-009 (Grocery List Access & Authorization Rules) | References (inbound) | Defines this screen's Access and Visibility table |
| FEAT-20.SPEC-001 (Grocery Handoff Screen) | Navigation (outbound) | "Hand off list" header control navigates here, gated by FEAT-20.SPEC-003 |
| FEAT-20.SPEC-003 (Handoff Eligibility & Authorization Rules) | References (inbound) | Governs the "Hand off list" control's region- and role-based visibility on this screen |
| FEAT-05.SPEC-001 (Pantry List & Item Entry) | Navigation (outbound) | "View Pantry" toast action navigates here after "already have it" |
| FEAT-05.SPEC-007 (Pantry Item Off-Grocery-List Exclusion Rule) | References (inbound) | Governs which roles see the "View Pantry" toast action |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| grocery_item_ticked | item origin (plan-derived/manual), aisle | Member ticks or unticks an item | supports success-metrics.md: "Grocery List Live-Update Trust" |
| grocery_item_added_manually | aisle assigned, merge outcome (new line/merged) | Member's manual add completes | supports success-metrics.md: "Grocery List Live-Update Trust" |
| grocery_item_edited | item origin | Member confirms a quantity edit | supports success-metrics.md: "Grocery List Live-Update Trust" |
| grocery_item_removed | item origin | Member removes an item | supports success-metrics.md: "Grocery List Live-Update Trust" |
| grocery_item_already_have | pantry_logged (yes/no), role | Member taps "Already have it" | supports success-metrics.md: "Grocery List Live-Update Trust" |

## Acceptance Criteria

**FEAT-06.SPEC-001-AC-01:** Given Maya is on the Grocery List screen before any plan exists this week, when she views the screen, then she sees "Your list will appear once this week's plan is ready. You can add items now." and the Add-item row is usable.

**FEAT-06.SPEC-001-AC-02:** Given Sam is on the Grocery List screen, when he types "more yoghurt" in the Add-item input and taps Add, then FEAT-06.SPEC-007 validates the name and a new line "more yoghurt" appears in its aisle section with "added by Sam" beneath it.

**FEAT-06.SPEC-001-AC-03:** Given Maya is standing in the supermarket with no signal, when she taps the tick control on "milk", then the item shows a strikethrough immediately and the offline banner "You're offline -- changes will sync when you reconnect." is visible.

**FEAT-06.SPEC-001-AC-04:** Given Sam is on the Grocery List screen, when connectivity returns after he ticked three items offline, then those ticks sync automatically via FEAT-06.SPEC-005 and the offline banner disappears without Sam taking any action.

**FEAT-06.SPEC-001-AC-05:** Given Maya opens the overflow menu on a plan-derived "spinach" line, when she taps "Already have it", then the line disappears from the list and she sees the toast "Marked as already have it -- added to your pantry."

**FEAT-06.SPEC-001-AC-06:** Given Jordan (older kid, limited login) opens the overflow menu on a plan-derived "feta" line, when he taps "Already have it", then the line disappears from the list and he sees the toast "Marked as already have it." with no pantry mention, since his Pantry Input access is None.

**FEAT-06.SPEC-001-AC-07:** Given Riley (Operator) opens the Grocery List for a household with an open Support Request, when the screen loads, then every row renders as static text with no tick, add, edit, remove, or already-have-it controls.

**FEAT-06.SPEC-001-AC-08:** Given Riley attempts to open a household's Grocery List with no open Support Request, when the attempt is made, then access is blocked entirely and no household list data is shown.

**FEAT-06.SPEC-001-AC-09:** Given Maya taps Edit quantity on a line and leaves the quantity field blank, when she taps Confirm, then the field shows the error "Quantity can't be empty." and the edit does not save.

**FEAT-06.SPEC-001-AC-10:** Given Sam removes an item from the list, when he taps Remove from the overflow menu, then the item disappears immediately with no confirmation dialog and no undo option.

**FEAT-06.SPEC-001-AC-11:** Given Maya adds "milk" and an existing manual line "Milk" is already on the list, when she taps Add, then FEAT-06.SPEC-007 merges the two into a single line rather than creating a duplicate.

**FEAT-06.SPEC-001-AC-12:** Given two devices in the same household both tick the same item while offline, when both reconnect, then the item settles into a single ticked state per FEAT-06.SPEC-008's idempotent-tick rule, with no double-toggle visible on either device.

**FEAT-06.SPEC-001-AC-13:** Given a write action ultimately fails after background retries, when the failure is final, then a banner "Couldn't save your change. Check your connection and try again." appears with a Retry option.

**FEAT-06.SPEC-001-AC-14:** Given Maya's session expires while she has unsaved text in the Add-item input, when she is prompted to sign in again, then the dialog "Your session has expired. Sign in to continue." appears and her typed text is restored after she signs back in.

**FEAT-06.SPEC-001-AC-15:** Given an unauthenticated visitor requests this screen directly, when the request is made, then they are redirected to the sign-in screen and no household list data is shown.

**FEAT-06.SPEC-001-AC-16:** Given Sam has typed "olive oil" into the Add-item input but has not tapped Add, when he navigates away from the Grocery List screen (or backgrounds the app) and then returns, then the Add-item input is empty -- the typed text was discarded, not preserved.

**FEAT-06.SPEC-001-AC-17:** Given Maya's household region has an available online-ordering capability, when she views the Grocery List header, then a "Hand off list" control is shown, and tapping it navigates her to FEAT-20.SPEC-001 (Grocery Handoff Screen).

**FEAT-06.SPEC-001-AC-18:** Given the household's region has no available online-ordering capability, when Sam views the Grocery List header, then no "Hand off list" control appears, per FEAT-20.SPEC-003; the same absence applies to Jordan (older kid, limited login) and Riley regardless of the household's region.

**FEAT-06.SPEC-001-AC-19:** Given Maya opens the overflow menu on a plan-derived "spinach" line and taps "Already have it", when the toast "Marked as already have it -- added to your pantry." appears, then it includes a "View Pantry" action, and tapping it navigates her to FEAT-05.SPEC-001 (Pantry List & Item Entry).

**FEAT-06.SPEC-001-AC-20:** Given Jordan (older kid, limited login) opens the overflow menu on a plan-derived "feta" line and taps "Already have it", when the toast "Marked as already have it." appears, then it contains no "View Pantry" action, since his Pantry Input access is None (FEAT-05.SPEC-007).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 5 (empty, populating, populated, error, offline) | 5 |
| Business Rules | 8 | 8 |
| Edge Cases | 7 | 7 |
