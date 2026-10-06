# Feature Specification: Milestone & Payment Schedule Setup

**Blueprint feature:** FEAT-04
**Priority tier:** Core
**Build order:** 005 of 33
**Depends on:** FEAT-03
**Blueprint source:** `docs/blueprint/specifications/FEAT-04-milestone-payment-schedule-setup/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Milestone & Payment Schedule Editor (Priority: P1)

Nadia defines, orders, prices, adjusts, and removes a project's milestones and sets the project's payment structure and triggers.

**Acceptance Scenarios:**

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

### User Story 2 - Milestone Timeline (Client View) (Priority: P1)

Read-only display of a project's milestone timeline within each viewer's own project view.

**Acceptance Scenarios:**

**FEAT-04.SPEC-002-AC-01:** Given Owen opens the milestones area of his own project from Portal Home, when the screen loads, then he sees an ordered, read-only list of that project's milestones with name, price/no-charge, payment-trigger tag, target date, and status.

**FEAT-04.SPEC-002-AC-02:** Given Priya opens the same project's milestone timeline, when the screen loads, then she sees the identical list Owen sees, with no edit controls and no Approve control on any row.

**FEAT-04.SPEC-002-AC-03:** Given Dana is in a logged support session on a freelancer's account, when she opens a project's milestone timeline, then she sees the same read-only list with no interactive controls available.

**FEAT-04.SPEC-002-AC-04:** Given Nadia opens this read-only view of one of her own projects, when the screen loads, then she sees the same milestone data she can edit in FEAT-04.SPEC-001, with no editing controls on this screen.

**FEAT-04.SPEC-002-AC-05:** Given a milestone's target date is set by Nadia in her own time zone, when Owen views this screen, then the date displays converted to Owen's own time zone per FEAT-15.SPEC-006.

**FEAT-04.SPEC-002-AC-06:** Given Owen taps a milestone row whose deliverable is ready for his review, when the tap registers, then he is navigated to the Milestone Review & Approval Screen (FEAT-08.SPEC-001).

**FEAT-04.SPEC-002-AC-07:** Given Priya taps a milestone row whose deliverable is ready for review, when the tap registers, then she is navigated to the Milestone Review & Approval Screen (FEAT-08.SPEC-001), where no Approve control is rendered, and never to the deliverable list (FEAT-06.SPEC-002).

**FEAT-04.SPEC-002-AC-08:** Given a project has an accepted proposal but no milestones defined, when Owen opens this screen, then he sees the message "No milestones have been set up for this project yet." rather than an error.

**FEAT-04.SPEC-002-AC-09:** Given the milestone list is loading on a slow connection, when the screen is open, then real incremental loading progress is shown rather than an indefinite blank state.

**FEAT-04.SPEC-002-AC-10:** Given the milestone list fails to load, when the failure occurs, then an error banner with a Retry button appears, and tapping Retry re-fetches the list.

**FEAT-04.SPEC-002-AC-11:** Given Priya loses connectivity while this screen is open with data already loaded, when connectivity drops, then a banner shows "You're offline. Showing the last milestone timeline you loaded." and the previously loaded list remains visible.

**FEAT-04.SPEC-002-AC-12:** Given Nadia adds a milestone in FEAT-04.SPEC-001 while Owen already has this screen open, when Owen does not refresh, then his view does not update live; when he navigates back to this screen or reloads it, then the new milestone appears.

**FEAT-04.SPEC-002-AC-13:** Given Owen is a contact for two different freelancers, when he opens this milestone timeline from each freelancer's portal separately, then each shows only that freelancer's own project data, with no cross-freelancer milestones ever appearing together.

**FEAT-04.SPEC-002-AC-14:** Given an unauthenticated visitor attempts to load this screen's URL directly, when the page attempts to render, then they are redirected to the magic-link sign-in request (FEAT-05.SPEC-001).

**FEAT-04.SPEC-002-AC-15:** Given a milestone has status Defined with no deliverable ready for review, when Owen taps its row, then he is navigated to the Milestone Comment Thread (FEAT-07.SPEC-002) for that milestone and not to FEAT-06.SPEC-002.

**FEAT-04.SPEC-002-AC-16:** Given a milestone has status Defined with no deliverable ready for review, when Priya taps its row, then she is navigated to the Milestone Comment Thread (FEAT-07.SPEC-002), and no denial message from FEAT-06.SPEC-002 is ever shown to her.

**FEAT-04.SPEC-002-AC-17:** Given Nadia opens this view of her own project and a milestone has no deliverable ready, when she taps its row, then she is navigated to that milestone's deliverable list (FEAT-06.SPEC-002).

### User Story 3 - Milestone & Schedule Validation and Edit Rules (Priority: P1)

Governs required fields, invoicing eligibility, approved/invoiced immutability, dated non-retroactive schedule edits, and the price-mismatch flag for the Milestone and Payment Schedule entities.

**Acceptance Scenarios:**

**FEAT-04.SPEC-003-AC-01:** Given Nadia leaves a milestone's name field empty and blurs it, when validation runs, then the field shows "Milestone name is required."

**FEAT-04.SPEC-003-AC-02:** Given Nadia enters a 201-character milestone name, when she blurs the field, then the field shows "Milestone name must be 200 characters or fewer."

**FEAT-04.SPEC-003-AC-03:** Given Nadia enters exactly a 200-character milestone name, when she blurs the field, then no error is shown.

**FEAT-04.SPEC-003-AC-04:** Given Nadia enters 0 as a milestone's price, when she blurs the field, then the field shows "Enter a price greater than zero, or mark this milestone as no separate charge."

**FEAT-04.SPEC-003-AC-05:** Given Nadia enters an invalid date in a milestone's Target Date field, when she blurs the field, then it shows "Enter a valid date."

**FEAT-04.SPEC-003-AC-06:** Given Nadia leaves a milestone's Target Date empty, when she saves, then no error is shown, since the field is optional.

**FEAT-04.SPEC-003-AC-07:** Given Nadia selects "Deposit" as the Payment Schedule's structure and leaves Deposit Amount empty, when she blurs the field, then it shows "Enter a deposit amount greater than zero."

**FEAT-04.SPEC-003-AC-08:** Given Nadia selects "On Completion" as the structure and enters a positive Completion Amount, when she blurs the field, then no error is shown.

**FEAT-04.SPEC-003-AC-09:** Given Nadia has not selected any Payment Schedule structure, when she attempts to save, then she sees "Choose a payment structure."

**FEAT-04.SPEC-003-AC-10:** Given Nadia leaves both price and "no separate charge" unset on a milestone and taps Save, when validation runs, then she sees "Enter a price, or mark this milestone as no separate charge -- not both."

**FEAT-04.SPEC-003-AC-11:** Given Nadia has entered a valid price for a milestone, when she toggles "No separate charge" on afterward, then the price value is cleared and only the toggle remains active.

**FEAT-04.SPEC-003-AC-12:** Given a project's schedule has no deposit, no completion amount, and no milestone marked to trigger on approval, when Nadia saves, then the non-blocking indicator "This project has no payment trigger yet -- no invoice can be generated until you add one" appears and the save still completes.

**FEAT-04.SPEC-003-AC-13:** Given a project's schedule includes at least one milestone marked to trigger on approval, when Nadia saves, then no payment-trigger indicator is shown.

**FEAT-04.SPEC-003-AC-14:** Given the sum of a project's milestone prices does not equal its accepted proposal's total, when Nadia saves, then the price-mismatch banner appears showing both figures and the save still completes.

**FEAT-04.SPEC-003-AC-15:** Given the sum of a project's milestone prices equals its accepted proposal's total, when Nadia saves, then no mismatch banner is shown.

**FEAT-04.SPEC-003-AC-16:** Given Nadia (Freelancer) attempts to create a milestone, when she submits valid data, then the milestone is created, since Create milestone is always allowed for her.

**FEAT-04.SPEC-003-AC-17:** Given Owen (Client Primary Contact) attempts to reach the milestone editor directly, when the page loads, then he is redirected to the read-only Milestone Timeline (FEAT-04.SPEC-002), since Create/Edit/Remove milestone is never allowed for him.

**FEAT-04.SPEC-003-AC-18:** Given Dana (Support Operator) is in a logged support session and attempts to tap Add Milestone, when the tap registers, then the control is disabled showing "Support access is read-only," and no milestone is created.

**FEAT-04.SPEC-003-AC-19:** Given Owen views milestones on his own company's project, when the screen loads, then he sees the full read-only list, since View milestone is always allowed for him on his own company's data.

**FEAT-04.SPEC-003-AC-20:** Given Priya attempts to open a milestone timeline for a project belonging to a different client company, when she follows an out-of-scope link, then she sees a plain explanation and a fresh-link option, never that company's data (XBR-09).

**FEAT-04.SPEC-003-AC-21:** Given Nadia attempts to edit a milestone whose status is Defined, when she changes its name and saves, then the change is accepted, since Edit milestone is allowed while status is Defined.

**FEAT-04.SPEC-003-AC-22:** Given Nadia attempts to edit a milestone whose status is Approved, when she looks for edit controls, then none are enabled, and a direct attempt shows "This milestone has been approved and can't be edited. Reopen it to make changes."

**FEAT-04.SPEC-003-AC-23:** Given Nadia attempts to remove a milestone whose status is Reopened, when she confirms the removal, then it is deleted, since Remove milestone is allowed while status is Reopened.

**FEAT-04.SPEC-003-AC-24:** Given Nadia attempts to remove a milestone whose status is Approved, when she looks for a Remove control, then none is shown, and a direct removal attempt shows "This milestone has been approved or invoiced and can't be removed."

**FEAT-04.SPEC-003-AC-25:** Given Nadia (Freelancer) attempts to create or edit the Payment Schedule, when she submits valid data, then the change is accepted, since this action is always allowed for her.

**FEAT-04.SPEC-003-AC-26:** Given Dana attempts to change the Payment Schedule's structure during a support session, when she attempts the action, then the control is disabled showing "Support access is read-only," and no change is made.

**FEAT-04.SPEC-003-AC-27:** Given a milestone is created with no explicit status set, when it is saved, then its status defaults to "Defined."

**FEAT-04.SPEC-003-AC-28:** Given Nadia saves any change to the Payment Schedule, when the save completes, then a new dated entry recording the prior and new values is appended to change_history.

**FEAT-04.SPEC-003-AC-29:** Given a milestone is approved by Owen, when the approval is recorded (FEAT-08.SPEC-003), then its approved_at and approved_by fields are set once and this feature never alters them afterward.

**FEAT-04.SPEC-003-AC-30:** Given Nadia's schedule is edited by another of her own sessions between her load and save, when she attempts to save, then her save is rejected with a refresh-required message, per the reject-with-refresh Payment Schedule Contention rule.

**FEAT-04.SPEC-003-AC-31:** Given Owen approves a milestone while Nadia is mid-edit on it in another session, when Nadia attempts to save her edit, then it is rejected and she is shown the refreshed, now-Approved milestone, per the reject-with-refresh Milestone Contention rule.

**FEAT-04.SPEC-003-AC-32:** Given a project's schedule structure changes from Mix to Deposit-only after a completion invoice already generated under the prior structure, when the change saves, then the existing completion invoice is unaffected and completion_amount no longer applies to any future trigger.

**FEAT-04.SPEC-003-AC-33:** Given Nadia removes the only milestone in a project carrying a payment trigger, when the removal completes, then the at-least-one-payment-trigger indicator activates and FEAT-09 blocks invoicing on that project until a trigger exists again.

**FEAT-04.SPEC-003-AC-34:** Given Nadia (Freelancer) opens the Milestone & Payment Schedule Editor for one of her own projects, when the screen loads, then she can view the current Payment Schedule, since View Payment Schedule is always allowed for her.

### User Story 4 - Milestone Reorder Recalculation (Priority: P1)

Renumbers the remaining milestones' order whenever one is added, removed, or manually reordered, so the project's milestone sequence stays contiguous.

**Acceptance Scenarios:**

**FEAT-04.SPEC-004-AC-01:** Given Nadia has three milestones in a project and adds a fourth at the end of the list, when the add commits, then the new milestone receives order 4 and no existing milestone's order changes.

**FEAT-04.SPEC-004-AC-02:** Given Nadia has four milestones and removes the first one, when the removal commits, then the remaining three milestones renumber to positions 1, 2, and 3, each shifted down by one.

**FEAT-04.SPEC-004-AC-03:** Given Nadia drags the third milestone in a five-milestone list to the first position, when the move commits, then the moved milestone becomes position 1 and the milestones that were first and second each shift down by one position.

**FEAT-04.SPEC-004-AC-04:** Given Nadia has exactly one milestone remaining after a removal, when the removal commits, then no order recalculation write occurs, since the single milestone is already at position 1.

**FEAT-04.SPEC-004-AC-05:** Given Nadia's removal attempt on an Approved milestone is refused by FEAT-04.SPEC-003, when the refusal occurs, then this automation never runs and the milestone list is unchanged.

**FEAT-04.SPEC-004-AC-06:** Given a recalculation fails partway through writing the updated order values, when the failure occurs, then all partial writes for that run are rolled back and the non-blocking warning "Milestones saved, but reordering couldn't be completed. Refresh to see the current order." appears.

**FEAT-04.SPEC-004-AC-07:** Given Nadia drags a milestone to the exact position it already occupies, when the drag completes, then no order values change and no write occurs.

**FEAT-04.SPEC-004-AC-08:** Given Nadia reorders a milestone whose status is Approved, when the move commits, then only its order field changes, and its name, price, and Approved status remain unaffected.

**FEAT-04.SPEC-004-AC-09:** Given Nadia adds a milestone from one browser tab while removing a different milestone from another tab of the same project at effectively the same time, when both commit, then the final order for the project's milestones is contiguous with no gap or duplicate.

**FEAT-04.SPEC-004-AC-10:** Given a reorder is already in flight when Nadia issues a second reorder on the same project, when she issues the second move, then it is held until the first's writes complete and the editor shows a brief in-progress state preventing a third overlapping reorder.

**FEAT-04.SPEC-004-AC-11:** Given two milestones are added to the same project from two open tabs in rapid succession, when both adds commit, then the second add's renumbering includes the first add's already-inserted milestone, leaving both milestones at distinct, contiguous positions.

**FEAT-04.SPEC-004-AC-12:** Given a project's milestone list is successfully renumbered after an add, remove, or reorder, when Owen next opens the Milestone Timeline (FEAT-04.SPEC-002), then he sees the milestones in the recalculated order.

### Edge Cases

- **FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor):** Unsaved changes trigger the discard confirmation, double taps on Save are ignored, and a failed save preserves the entered milestone and schedule data on screen. A price mismatch on save never blocks; the mismatch banner is shown because scope can legitimately change. Source: `docs/blueprint/specifications/FEAT-04-milestone-payment-schedule-setup/FEAT-04.SPEC-001-milestone-payment-schedule-editor.md` (section: Edge Cases)
- **FEAT-04.SPEC-002 (Milestone Timeline (Client View)):** The screen is a fetched snapshot with no live updates, so milestone edits or approvals made while a client contact has it open appear on the next load (a Reviewer still sees no Approve control). A project with an accepted proposal and zero milestones shows the Empty state, and a contact of two freelancers sees each portal's milestones strictly separately (XBR-09). Source: `docs/blueprint/specifications/FEAT-04-milestone-payment-schedule-setup/FEAT-04.SPEC-002-milestone-timeline-client-view.md` (section: Edge Cases)
- **FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules):** Milestone name passes at 200 characters and fails at 201, a price of exactly 0 is invalid, and leaving both price and no-separate-charge unset is blocked with the cross-field error. When both appear set at once the most recently changed control wins, so they are never both active when persisted. Source: `docs/blueprint/specifications/FEAT-04-milestone-payment-schedule-setup/FEAT-04.SPEC-003-milestone-schedule-validation-and-edit-rules.md` (section: Edge Cases)
- **FEAT-04.SPEC-004 (Milestone Reorder Recalculation):** Appending at the end writes only the new milestone's order, removing the first shifts every remaining milestone down one position, and a no-op drag to the same position writes nothing. Reordering an Approved milestone changes only its order field and leaves its locked fields and status untouched. Source: `docs/blueprint/specifications/FEAT-04-milestone-payment-schedule-setup/FEAT-04.SPEC-004-milestone-reorder-recalculation.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-04.SPEC-001** (Milestone & Payment Schedule Editor) as specified: Nadia defines, orders, prices, adjusts, and removes a project's milestones and sets the project's payment structure and triggers. Full spec: `docs/blueprint/specifications/FEAT-04-milestone-payment-schedule-setup/FEAT-04.SPEC-001-milestone-payment-schedule-editor.md`
- **FR-002**: The system MUST implement **FEAT-04.SPEC-002** (Milestone Timeline (Client View)) as specified: Read-only display of a project's milestone timeline within each viewer's own project view. Full spec: `docs/blueprint/specifications/FEAT-04-milestone-payment-schedule-setup/FEAT-04.SPEC-002-milestone-timeline-client-view.md`
- **FR-003**: The system MUST implement **FEAT-04.SPEC-003** (Milestone & Schedule Validation and Edit Rules) as specified: Governs required fields, invoicing eligibility, approved/invoiced immutability, dated non-retroactive schedule edits, and the price-mismatch flag for the Milestone and Payment Schedule entities. Full spec: `docs/blueprint/specifications/FEAT-04-milestone-payment-schedule-setup/FEAT-04.SPEC-003-milestone-schedule-validation-and-edit-rules.md`
- **FR-004**: The system MUST implement **FEAT-04.SPEC-004** (Milestone Reorder Recalculation) as specified: Renumbers the remaining milestones' order whenever one is added, removed, or manually reordered, so the project's milestone sequence stays contiguous. Full spec: `docs/blueprint/specifications/FEAT-04-milestone-payment-schedule-setup/FEAT-04.SPEC-004-milestone-reorder-recalculation.md`

### Key Entities

- Milestone (create, update)
- Payment Schedule (create, update)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: 100% of projects with an accepted proposal have at least one milestone with an assigned payment trigger before the first deliverable is uploaded (metric: Milestone Schedule Completeness). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Milestone creation, schedule edits, payment trigger assignment and milestone removal are each observable as distinct signals (milestone_created, milestone_schedule_edited, payment_trigger_set, milestone_removed). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-05**: Freelancers are assumed to define milestones and a payment schedule before starting work rather than invoicing ad hoc. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-27**: Screens keep typed input on errors and never pretend an offline action succeeded; this bears on the schedule editor's offline behavior. Full register: `docs/blueprint/features/assumptions-constraints.md`
