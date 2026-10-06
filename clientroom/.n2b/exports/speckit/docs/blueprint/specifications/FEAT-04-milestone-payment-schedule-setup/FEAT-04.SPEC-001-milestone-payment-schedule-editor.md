---
document_type: spec
spec_type: screen
spec_id: FEAT-04.SPEC-001
spec_name: Milestone & Payment Schedule Editor
spec_slug: milestone-payment-schedule-editor
parent_feature: FEAT-04
parent_feature_name: Milestone & Payment Schedule Setup
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 23
---

# Screen Spec: Milestone & Payment Schedule Editor

## Overview

**Name:** Milestone & Payment Schedule Editor
**ID:** FEAT-04.SPEC-001
**Type:** Screen
**Purpose:** Nadia defines, orders, prices, adjusts, and removes a project's milestones and sets the project's payment structure and triggers.
**Parent Feature:** FEAT-04 -- Milestone & Payment Schedule Setup

## Scope and Non-Goals

**In Scope:**
- Creating, renaming, pricing, target-dating, reordering, and removing milestones for one project
- Creating and editing the project's single Payment Schedule (structure: deposit, per-milestone, on completion, or a mix; deposit_amount; completion_amount)
- Setting each milestone's payment trigger (whether its approval issues an invoice)
- Displaying the price-mismatch flag against the accepted proposal's total, and a change-history view of dated schedule adjustments
- Preserving in-progress edits through a failed save or a connectivity loss

**Non-Goals:**
- Read-only display of the milestone timeline to client contacts -- owned by FEAT-04.SPEC-002 (Milestone Timeline, Client View); the Brief's Internal Dependency Map routes Nadia and client contacts to distinct specs since editing and viewing are separate concerns.
- Field-level validation, eligibility, and dated-edit rule logic -- governed by FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules); this screen calls that spec's rules on save rather than duplicating them.
- Milestone status transitions beyond the initial "Defined" state -- Deliverable Uploaded is set by FEAT-06 (Deliverable Upload & Sharing) and Approved/Reopened by FEAT-08 (Milestone Approval), per the dependency map's Milestone lifecycle line.
- Recurring or retainer billing on a fixed calendar -- excluded per scope-boundaries.md (SC-14): the payment structure this feature offers is limited to deposit, per-milestone, on completion, or a mix.
- Standalone deletion or archival of the Payment Schedule record itself -- excluded per the Brief's Non-Goals: the dependency map lists the schedule as deleted only through FEAT-24 (full account deletion), never in-product, because the schedule is foundational to the project's invoicing for its whole life.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-005 (Project Detail / Open Project) | Nadia opens the milestones area of a project | Project reference; the project's existing milestones (if any) and Payment Schedule (if already created) load into the editor |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | All actions: create, rename, price, target-date, reorder, and remove milestones; create and edit the Payment Schedule; save | -- |
| Owen (Client Primary Contact) | No | No | This editor is not part of his portal navigation and no direct link exists to it; if he somehow reaches the URL, he is redirected to his own scoped project view, which shows the read-only Milestone Timeline (FEAT-04.SPEC-002) instead |
| Priya (Client Reviewer Contact) | No | No | Same as Owen: redirected to the read-only Milestone Timeline (FEAT-04.SPEC-002) in her own scoped project view |
| Dana (Support Operator) | Full screen, read-only, inside a logged support session (FEAT-31) | No | Every input, selector, drag handle, Add Milestone, Remove, and Save control is disabled with the tooltip "Support access is read-only." No attempted edit reaches the server |
| Unauthenticated | No | No | Redirected to Nadia's sign-in; after signing in, the user lands on the Client & Project Roster (FEAT-01.SPEC-003), not this screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Any in-progress local milestone or schedule edits are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Breadcrumb back arrow to the Project Detail screen (FEAT-01.SPEC-005), screen title "Milestones & Payment Schedule," and the project's name as a subtitle.

**Body:** Two stacked sections.

*Payment Structure section (top):* A four-option structure selector -- Deposit, Per-Milestone, On Completion, Mix. Selecting Deposit or Mix reveals a Deposit Amount field (currency-formatted number input, in the project's configured currency). Selecting On Completion or Mix reveals a Completion Amount field (same treatment). Below the selector, a "View change history" link expands a dated list of prior schedule adjustments (date, what changed, from/to values), sourced from the Payment Schedule's change_history.

*Milestone List section (below):* An ordered list of milestone rows. Each row shows, in order: a reorder control (drag handle plus explicit Move Up / Move Down buttons), the milestone Name (text input), a Price input paired with a "No separate charge" toggle (exactly one of the two is active per milestone), a Payment Trigger selector ("Issues an invoice on approval" / "No trigger"), a Target Date picker (optional), a read-only Status badge (Defined, Deliverable Uploaded, Approved, or Reopened), and a Remove control. The Remove control is visible only on milestones whose status is Defined or Reopened, per FEAT-04.SPEC-003's approved/invoiced immutability rule. An "Add Milestone" button sits below the last row.

A **price-mismatch banner** appears directly above the Milestone List section whenever the sum of milestone prices does not match the accepted proposal's total price (per FEAT-04.SPEC-003). It states the two figures and that this is informational only.

**Footer:** A "Save" button that persists all schedule and milestone edits together, and a "Done" link that returns to Project Detail (FEAT-01.SPEC-005).

### Responsive Behavior

- **Compact breakpoint:** The two sections stack vertically, full width. Each milestone row collapses to two lines: name and status badge on the first line, price/no-charge toggle, payment trigger, and target date on the second. Reordering uses the Move Up / Move Down buttons only; the drag handle is not shown.
- **Medium size class and above:** Each milestone row renders as a single horizontal row with all fields visible side by side. The drag handle appears alongside the Move Up / Move Down buttons. The Payment Structure selector renders as an inline horizontal group instead of a stacked list.
- **Change-history panel:** Renders as an inline expanding list at all sizes; no structural change beyond width.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-005 (Project Detail) | Screen closes; unsaved local edits prompt confirmation first | If unsaved changes exist: dialog "You have unsaved changes. Discard?" |
| Payment Structure selector | Select an option | Sets the schedule's structure locally; reveals/hides Deposit Amount and Completion Amount fields | Selected option highlighted; dependent fields appear/disappear | Immediate visual update, no save yet |
| Deposit Amount input | Type | Captures the deposit amount locally | Field shows entered value | Standard input focus state; validated via FEAT-04.SPEC-003 on blur |
| Completion Amount input | Type | Captures the completion amount locally | Field shows entered value | Standard input focus state; validated via FEAT-04.SPEC-003 on blur |
| "View change history" link | Tap | Expands the dated change_history list | Panel expands below the link | List of prior dated adjustments shown |
| Add Milestone button | Tap | Adds a new, empty milestone row at the end of the list with status Defined | New row appears; triggers FEAT-04.SPEC-004 (Milestone Reorder Recalculation) to confirm its order position | New row focused on its Name field |
| Milestone Name input | Type / Blur | Captures the milestone name; validated via FEAT-04.SPEC-003 on blur | Field shows entered text; error state if blank on blur | "Milestone name is required" if empty on blur |
| Price input / "No separate charge" toggle | Type or toggle | Sets the milestone's price, or marks it as carrying no separate charge (mutually exclusive); validated via FEAT-04.SPEC-003 | Whichever control is active shows the current value; the other clears | Standard input state; cross-field error via FEAT-04.SPEC-003 if both or neither are set on submit |
| Payment Trigger selector | Select | Sets whether this milestone's approval issues an invoice | Selector shows chosen option | Immediate visual update |
| Target Date picker | Select a date | Sets the milestone's optional target date, shown in each viewer's own time zone (FEAT-15.SPEC-006) | Field shows chosen date | Standard picker feedback |
| Reorder (drag handle or Move Up/Down) | Drag or tap | Moves the milestone to a new position; triggers FEAT-04.SPEC-004 (Milestone Reorder Recalculation) | List re-renders in the new order | Brief in-progress state while recalculation completes, then the renumbered list |
| Remove control (Defined/Reopened milestones only) | Tap | Validates removal eligibility via FEAT-04.SPEC-003; if allowed, deletes the milestone and triggers FEAT-04.SPEC-004 | Row is removed from the list; remaining rows renumber | Confirmation dialog before removal: "Remove this milestone? This can't be undone." |
| Save button | Tap | 1. Validates all fields via FEAT-04.SPEC-003. 2. If valid, persists the Payment Schedule and every changed Milestone. 3. Appends a dated change_history entry for any schedule change. | Button shows loading state during save | Success: toast "Schedule saved" and the editor stays open. Failure: inline field errors or a save-error banner; entered data is preserved |
| Save button (while saving) | Tap | No action -- debounced to prevent a duplicate submission | None | Button remains in loading state |
| Done link | Tap | Navigate to FEAT-01.SPEC-005 (Project Detail) | Screen closes; unsaved changes prompt confirmation first | Same unsaved-changes dialog as the back arrow |

### Accessibility Notes

- **Focus order:** Back arrow -> Payment Structure selector -> Deposit Amount (if shown) -> Completion Amount (if shown) -> "View change history" link -> each milestone row in list order (reorder control -> Name -> Price/No-charge toggle -> Payment Trigger -> Target Date -> Remove) -> Add Milestone -> Save -> Done.
- **Reordering:** Every milestone can be reordered by keyboard using the always-visible Move Up / Move Down buttons; the drag handle is a pointer-only convenience, never the sole way to reorder.
- **Validation announcements:** When a field enters an error state (via FEAT-04.SPEC-003), its error message is announced to assistive technology and programmatically associated with the field. The price-mismatch banner is announced when it first appears.
- **Save feedback:** The "Schedule saved" toast is announced on success; on validation failure, focus moves to the first field in error.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Both sections show a lightweight in-progress indicator while the project's existing milestones and Payment Schedule (if any) are fetched | Screen is opened from FEAT-01.SPEC-005 | Existing data finishes loading, revealing either the populated editor or the Empty state |
| Empty (no milestones) | Milestone List section shows "No milestones yet" with the Add Milestone button prominent; Payment Structure section is fully usable | Project has an accepted proposal but no milestones defined yet | Nadia adds the first milestone |
| Filling (local editing) | Fields reflect entered values immediately; Save button enabled | Nadia types in or selects any field | Nadia taps Save or navigates away |
| Validating | Save button shows a loading spinner | Nadia taps Save | FEAT-04.SPEC-003 validation completes (pass or fail) |
| Validation Error | Failing fields show inline error messages from FEAT-04.SPEC-003; price-mismatch banner shown if applicable (non-blocking) | Blocking validation fails | Nadia corrects the field(s) and re-triggers Save |
| Saving | Save button shows a loading spinner; fields remain editable | Validation passes | Save completes or fails |
| Save Error | Error banner: "Could not save. Check your connection and try again." with a Retry button | Save operation fails (not a validation failure) | Nadia taps Retry or the save succeeds on retry; all entered data remains on screen |
| Offline/Degraded | Banner: "You're offline -- your changes will sync once you reconnect." Fields remain fully editable and changes appear immediately in the list; Save queues the update locally | Connectivity is lost while the screen is open | Connectivity restored -- the queued save submits automatically and the standard success or validation feedback appears |

## Validation Rules

Validation governed by FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules). See that spec for all field-level, cross-field, and authorization rules. This screen applies validation on field blur (name, price/no-charge, deposit/completion amounts, target date) and on Save (the full cross-field and eligibility set, including the price-mismatch and payment-trigger checks).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-01.SPEC-005 (Project Detail / Open Project) | FEAT-01 |
| Done link tap | FEAT-01.SPEC-005 (Project Detail / Open Project) | FEAT-01 |
| Cancel on unsaved-changes dialog | Stays on this screen | -- |
| Confirm discard on unsaved-changes dialog | FEAT-01.SPEC-005 (Project Detail / Open Project) | FEAT-01 |

## Data Model

**Creates:** Milestone record -- name, order (assigned via FEAT-04.SPEC-004), price or no_separate_charge flag, payment_trigger, target_date; initial status set to Defined. Payment Schedule record -- structure, and deposit_amount/completion_amount as applicable; created on the project's first schedule save (one per project).

**Reads:** Milestone -- all fields, for every milestone belonging to the current project. Payment Schedule -- current structure, deposit_amount, completion_amount, and change_history. Proposal (read-only, owned by FEAT-03) -- the accepted proposal's total price, for the price-mismatch check.

**Updates:** Milestone -- name, price/no_separate_charge flag, payment_trigger, target_date, order (while status is Defined or Reopened; blocked once Approved or invoiced, per FEAT-04.SPEC-003). Payment Schedule -- structure, deposit_amount, completion_amount; each successful edit appends a dated entry to change_history.

**Deletes:** Milestone -- hard delete, only while status is Defined or Reopened (never once Approved or invoiced, per FEAT-04.SPEC-003); cascades to that milestone's Deliverables and milestone-level Comments, whose removal runs under FEAT-06's and the comment thread's own lifecycle rules.

## Business Rules

- All save-time field validation, eligibility (approved/invoiced immutability), the dated non-retroactive schedule-edit rule, and price-mismatch flagging are governed by FEAT-04.SPEC-003 -- this screen calls those rules rather than duplicating them.
- XBR-10: A milestone that has been approved or invoiced cannot be removed or re-priced; changes go through a logged reopen (FEAT-08) or an invoice correction (FEAT-09).
- Adding, removing, or manually reordering a milestone triggers FEAT-04.SPEC-004 (Milestone Reorder Recalculation), which renumbers the remaining milestones so the sequence stays contiguous.
- At least one payment trigger must exist somewhere in the project's schedule before the project can invoice at all (FEAT-04.SPEC-003); this is surfaced here as a non-blocking indicator, not a save-blocking rule.
- A saved schedule or milestone-pricing change is dated and applies only to payment triggers that fire afterward -- a trigger that has already fired uses the schedule as it stood at that moment (dependency map, Payment Schedule Contention note).

## Edge Cases

- **User navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **User taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Save fails (e.g., connectivity lost)** -- Entered milestone and schedule data is preserved on-screen; the draft is never discarded (per the Brief's Side-Effect Inventory).
- **Price mismatch on save** -- Save proceeds; the mismatch banner is shown, never blocking, since scope can legitimately change.
- **Removing the milestone that currently carries the project's only payment trigger** -- Removal itself proceeds normally (it is not an approved/invoiced milestone); the "no payment trigger yet" indicator then appears, since invoicing is now blocked until a trigger exists again (enforced at invoice time by FEAT-09).
- **Payment Schedule changed by another of Nadia's own sessions between load and save** -- Save is rejected with dialog "This schedule was updated in another session. Review the latest version before saving." with "View Latest" (reloads the record; local edits discarded after confirmation) and "Keep Editing" options. Resolution: reject-with-refresh, per the dependency map's Contention note for Payment Schedule.
- **A milestone Owen approves while Nadia has it open for editing** -- Nadia's next save on that milestone is rejected with "This milestone was approved by Owen while you were editing. Reopen it to make further changes." (see FEAT-08.SPEC-002, Milestone Reopen Screen). The row switches to read-only and its Status badge updates to Approved. Resolution: reject-with-refresh, per the dependency map's Contention note for Milestone.
- **Removing a milestone that was invoiced by another process a moment earlier** -- Removal is refused with "This milestone has already been invoiced and can't be removed," and the row's Status badge and controls refresh to reflect its current state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules) | References (outbound) | All field validation, eligibility, dated-edit, and price-mismatch rules applied on save |
| FEAT-04.SPEC-004 (Milestone Reorder Recalculation) | Triggers (outbound) | Fires whenever a milestone is added, removed, or manually reordered |
| FEAT-04.SPEC-002 (Milestone Timeline, Client View) | References (outbound) | Client-facing counterpart shows the same milestone row data read-only; edits made here appear there on the client's next read |
| FEAT-01.SPEC-005 (Project Detail / Open Project) | Navigation (inbound) | Entry point via the project's milestones area |
| FEAT-03.SPEC-003 (Acceptance Recording) | References (outbound) | Reads this schedule at proposal acceptance to fire the deposit trigger (XBR-01), using the schedule as it stood at that moment |
| FEAT-06.SPEC-001 (Deliverable Upload) | References (inbound) | Updates a milestone's status to Deliverable Uploaded, reflected in this screen's Status badge |
| FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard) | References (inbound) | Updates a milestone's status to Approved, locking edit and remove per XBR-10 |
| FEAT-08.SPEC-005 (Reopen Recording) | References (inbound) | Updates a milestone's status to Reopened, unlocking edit and remove |
| FEAT-09.SPEC-004 (Automatic Invoice Generation) | References (outbound) | Reads each milestone's payment_trigger and the schedule to drive automatic invoicing (XBR-02) |
| FEAT-15.SPEC-006 (Time Zone & Local Date/Time Display Rule) | References (outbound) | Governs how the Target Date field is rendered in each viewer's own time zone |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| milestone_created | has_price (yes/no), payment_trigger_set (yes/no) | A new milestone is successfully saved | supports success-metrics.md: "Milestone Schedule Completeness" |
| milestone_schedule_edited | change_type (milestone_edit / schedule_edit), structure | A milestone or the Payment Schedule is successfully edited after initial creation | supports success-metrics.md: "Milestone Schedule Completeness" |
| payment_trigger_set | milestone reference, trigger_type (deposit / per_milestone / on_completion) | A milestone's payment trigger is set or changed and saved | supports success-metrics.md: "Milestone Schedule Completeness" |
| milestone_removed | remaining_trigger_count | A milestone is successfully removed | supports success-metrics.md: "Milestone Schedule Completeness" |
| schedule_price_mismatch_flagged | mismatch_amount | The price-mismatch banner is shown on a successful save | N/A -- no success-metrics.md metric measures mismatch-flag frequency; this event is retained for product telemetry so the signal is not silently dropped |

## Acceptance Criteria

**FEAT-04.SPEC-001-AC-01:** Given Nadia opens the Milestone & Payment Schedule Editor from Project Detail, when the screen is fetching the project's existing milestones and schedule, then both sections show a lightweight in-progress indicator rather than an indefinite blank screen.

**FEAT-04.SPEC-001-AC-02:** Given Nadia is on the Milestone & Payment Schedule Editor for a project with no milestones, when the screen loads, then the Milestone List section shows "No milestones yet" with the Add Milestone button prominent.

**FEAT-04.SPEC-001-AC-03:** Given Nadia selects "Mix" in the Payment Structure selector, when the selection completes, then both the Deposit Amount and Completion Amount fields appear.

**FEAT-04.SPEC-001-AC-04:** Given Nadia taps Add Milestone, when the new row appears, then it has an empty Name field focused, status Defined, and FEAT-04.SPEC-004 confirms its order position at the end of the list.

**FEAT-04.SPEC-001-AC-05:** Given Nadia types a milestone name and blurs the field, when the name is non-empty, then no error is shown; when it is empty, then the error "Milestone name is required" appears below the field.

**FEAT-04.SPEC-001-AC-06:** Given Nadia enters a price for a milestone, when she also has "No separate charge" toggled on, then setting the price clears the toggle so only one is active.

**FEAT-04.SPEC-001-AC-07:** Given Nadia sets a milestone's Payment Trigger to "Issues an invoice on approval," when she saves, then the milestone's payment_trigger is persisted and available to FEAT-08 and FEAT-09.

**FEAT-04.SPEC-001-AC-08:** Given Nadia sets a target date on a milestone, when the client Owen later views the same milestone in FEAT-04.SPEC-002, then the date displays converted to Owen's own time zone per FEAT-15.SPEC-006.

**FEAT-04.SPEC-001-AC-09:** Given Nadia drags a milestone to a new position, when the move completes, then FEAT-04.SPEC-004 renumbers the list and it re-renders in the new contiguous order.

**FEAT-04.SPEC-001-AC-10:** Given Nadia taps Remove on a milestone with status Defined, when she confirms the dialog, then the milestone is deleted and FEAT-04.SPEC-004 renumbers the remaining milestones.

**FEAT-04.SPEC-001-AC-11:** Given Nadia is viewing a milestone with status Approved, when she looks for a Remove control, then none is shown, per FEAT-04.SPEC-003's approved/invoiced immutability rule.

**FEAT-04.SPEC-001-AC-12:** Given Nadia has filled valid values across the schedule and milestones, when she taps Save, then the Payment Schedule and all changed Milestones persist, and the toast "Schedule saved" appears with the editor remaining open.

**FEAT-04.SPEC-001-AC-13:** Given Nadia taps Save while a save is already in progress, when she taps again, then the second tap is ignored and the button remains in its loading state.

**FEAT-04.SPEC-001-AC-14:** Given Nadia's milestone prices sum to less than the accepted proposal's total, when she saves, then the save completes and the price-mismatch banner appears showing both figures, without blocking the save.

**FEAT-04.SPEC-001-AC-15:** Given Nadia loses connectivity while editing, when she taps Save, then the banner "You're offline -- your changes will sync once you reconnect" appears and the change is submitted automatically once connectivity returns.

**FEAT-04.SPEC-001-AC-16:** Given Nadia's save fails due to a connection error (not a validation failure), when the failure occurs, then an error banner with a Retry option appears and all entered data remains on screen.

**FEAT-04.SPEC-001-AC-17:** Given Nadia has unsaved changes on this screen, when she taps the back arrow, then a dialog asks "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-04.SPEC-001-AC-18:** Given Nadia has this project's schedule open in a second browser session and saves a change there first, when she then taps Save in this session, then the save is rejected with "This schedule was updated in another session. Review the latest version before saving," offering "View Latest" and "Keep Editing."

**FEAT-04.SPEC-001-AC-19:** Given Owen approves a milestone while Nadia has that milestone's row open for editing, when Nadia attempts to save her edit, then it is rejected with "This milestone was approved by Owen while you were editing. Reopen it to make further changes," and the row becomes read-only showing the Approved badge.

**FEAT-04.SPEC-001-AC-20:** Given Owen (Client Primary Contact) attempts to reach this editor's URL directly, when the page loads, then he is redirected to his own scoped project view showing the read-only Milestone Timeline (FEAT-04.SPEC-002).

**FEAT-04.SPEC-001-AC-21:** Given Dana (Support Operator) is viewing this screen inside a logged support session, when she attempts to type into any field or tap Save, then the control is disabled and shows "Support access is read-only," and no change reaches the server.

**FEAT-04.SPEC-001-AC-22:** Given an unauthenticated visitor loads this screen's URL, when the page attempts to render, then they are redirected to Nadia's sign-in and, after signing in, land on the Client & Project Roster (FEAT-01.SPEC-005) rather than this editor.

**FEAT-04.SPEC-001-AC-23:** Given Nadia's session expires while she has unsaved edits on this screen, when the expiry is detected, then the dialog "Your session has expired. Sign in to continue." appears, and her in-progress edits are restored after she re-authenticates.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 15 | 15 |
| States | 8 (loading, empty, filling, validating, validation error, saving, save error, offline/degraded) | 8 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |
