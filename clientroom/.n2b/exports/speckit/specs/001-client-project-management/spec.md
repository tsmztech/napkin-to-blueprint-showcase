# Feature Specification: Client & Project Management

**Blueprint feature:** FEAT-01
**Priority tier:** Core
**Build order:** 001 of 33
**Depends on:** —
**Blueprint source:** `docs/blueprint/specifications/FEAT-01-client-project-management/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Add Client (Priority: P1)

Nadia records a new client company she works with, so a project can be created under it.

**Acceptance Scenarios:**

**FEAT-01.SPEC-001-AC-01:** Given Nadia is on the Add Client screen, when she enters "Acme Studio" as client name and taps Save, then the active-client limit is checked, the client is saved, a "Client added" toast appears, and she returns to the roster with the new client visible.

**FEAT-01.SPEC-001-AC-02:** Given Nadia is on the Add Client screen, when she taps Save with the client name field empty, then the client name field shows the error "Client name is required" and the save does not proceed.

**FEAT-01.SPEC-001-AC-03:** Given Nadia is on the Add Client screen with unsaved input, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?"

**FEAT-01.SPEC-001-AC-04:** Given Nadia is at her plan's active-client limit, when she submits a valid client name and taps Save, then the save is blocked with the message "You've reached your plan's active client limit. Upgrade to add more clients." and her entered data remains in the form.

**FEAT-01.SPEC-001-AC-05:** Given Nadia loses connectivity while filling the form, then the "You're offline. Connect to add a client." banner appears and Save is disabled until connectivity returns.

**FEAT-01.SPEC-001-AC-06:** Given Nadia's connectivity returns after a queued Save attempt made while offline, then the client is saved automatically and the standard "Client added" success feedback appears with no duplicate client created.

**FEAT-01.SPEC-001-AC-07:** Given Nadia leaves Billing Name and Billing Address empty and taps Save, when the client name is valid, then the client saves successfully with billing fields empty, since billing completeness (FEAT-01.SPEC-010) is not required on this screen.

**FEAT-01.SPEC-001-AC-08:** Given a save attempt fails due to a network error, then an error banner "Could not add this client. Check your connection and try again." appears with a Retry button, and all entered data is preserved.

**FEAT-01.SPEC-001-AC-09:** Given Dana is in a read-only support session on Nadia's account, when she looks for a way to add a client, then no such control or screen is reachable from her session.

### User Story 2 - Create Project (Priority: P1)

Nadia creates a project under a specific client, entering the roster at stage "Draft."

**Acceptance Scenarios:**

**FEAT-01.SPEC-002-AC-01:** Given Nadia is on the Create Project screen for client "Acme Studio," when she enters "Website Redesign" as project name and taps Save, then the project is saved with stage "Draft," a "Project created" toast appears, and she is taken to the new project's Project Detail (FEAT-01.SPEC-005).

**FEAT-01.SPEC-002-AC-02:** Given Nadia is on the Create Project screen, when she taps Save with the project name field empty, then the project name field shows the error "Project name is required" and the save does not proceed.

**FEAT-01.SPEC-002-AC-03:** Given Nadia is on the Create Project screen with an unsaved project name, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?"

**FEAT-01.SPEC-002-AC-04:** Given Nadia loses connectivity while filling the form, then the "You're offline. Connect to create a project." banner appears and Save is disabled until connectivity returns.

**FEAT-01.SPEC-002-AC-05:** Given Nadia's connectivity returns after a queued Save made while offline, then the project is saved automatically and the standard "Project created" success feedback appears with no duplicate project created.

**FEAT-01.SPEC-002-AC-06:** Given a save attempt fails due to a network error, then an error banner "Could not create this project. Check your connection and try again." appears with a Retry button, and the entered project name is preserved.

**FEAT-01.SPEC-002-AC-07:** Given the owning client was archived by another session while Nadia had this screen open, when she taps Save, then the save is rejected with "This client is no longer active. Return to the roster and try again." and she is returned to the roster.

**FEAT-01.SPEC-002-AC-08:** Given Dana is in a read-only support session on Nadia's account, when she looks for a way to create a project, then no such control or screen is reachable from her session.

### User Story 3 - Client & Project Roster (Priority: P1)

Nadia (and, read-only, Dana in a support session) sees every client and project she manages in one place, each showing its current stage.

**Acceptance Scenarios:**

**FEAT-01.SPEC-003-AC-01:** Given Nadia has no clients yet, when she opens the Client & Project Roster, then she sees "No clients yet -- add your first one to get started" with an "Add Client" action.

**FEAT-01.SPEC-003-AC-02:** Given Nadia has clients and projects, when the roster loads, then each project shows a stage badge computed by FEAT-01.SPEC-011 (Draft, In Progress, Complete, Cancelled, or Archived).

**FEAT-01.SPEC-003-AC-03:** Given Nadia is on the roster, when she taps "Add Client," then she is taken to FEAT-01.SPEC-001 (Add Client).

**FEAT-01.SPEC-003-AC-04:** Given Nadia is on the roster, when she taps "New Project" under client "Acme Studio," then she is taken to FEAT-01.SPEC-002 (Create Project) pre-scoped to Acme Studio.

**FEAT-01.SPEC-003-AC-05:** Given Nadia is on the roster with the default Active filter, when she switches the filter to Archived, then only archived clients and their projects are shown, and the change is announced to assistive technology.

**FEAT-01.SPEC-003-AC-06:** Given Nadia's roster has more clients than fit in the first render, when it loads, then skeleton rows appear while data is fetched rather than a blank screen.

**FEAT-01.SPEC-003-AC-07:** Given Nadia loses connectivity while viewing an already-loaded roster, then a "You're offline -- showing the last loaded list." banner appears, the last list remains browsable, and Add Client / New Project become disabled.

**FEAT-01.SPEC-003-AC-08:** Given the roster fails to load, then an error banner "Couldn't load your clients and projects. Try again." appears with a Retry button.

**FEAT-01.SPEC-003-AC-09:** Given Dana opens a logged support session on Nadia's account, when she views the roster, then she sees every client and project read-only, with no Add Client or New Project controls rendered.

**FEAT-01.SPEC-003-AC-10:** Given Nadia selects a client or project from Global Search results (FEAT-28), when the roster opens, then the matching row is highlighted and scrolled into view.

### User Story 4 - Client Detail (Priority: P1)

Single view of one client -- rename it, edit billing details, view its contacts, and archive, reactivate, or delete it.

**Acceptance Scenarios:**

**FEAT-01.SPEC-004-AC-01:** Given Nadia is on Client Detail for "Acme Studio," when she renames it to "Acme Studio Inc." and confirms, then the header updates and a "Client renamed" toast appears.

**FEAT-01.SPEC-004-AC-02:** Given Nadia edits the billing address and taps section Save, when the save succeeds, then the completeness indicator updates to reflect whether billing details are now complete.

**FEAT-01.SPEC-004-AC-03:** Given Nadia taps Archive on a client with no unpaid invoices or pending approvals, then the client archives immediately with a "Client archived" toast and no confirmation dialog.

**FEAT-01.SPEC-004-AC-04:** Given Nadia taps Archive on a client with an unpaid invoice, then FEAT-01.SPEC-007's confirmation dialog appears summarizing the open item before the archive completes.

**FEAT-01.SPEC-004-AC-05:** Given Nadia taps Reactivate on an archived client while she is already at her plan's active-client limit, then the action is blocked with "You've reached your plan's active client limit. Upgrade to reactivate this client." and the client stays Archived.

**FEAT-01.SPEC-004-AC-06:** Given Nadia opens the overflow menu on a client with a sent invoice, when she looks at Delete, then it is disabled with the text "This client has a sent proposal, invoice, or activity and can only be archived."

**FEAT-01.SPEC-004-AC-07:** Given Nadia taps Delete on a client with no sent proposal, invoice, or activity, when she confirms, then the client is permanently deleted and she returns to the roster with a "Client deleted" toast.

**FEAT-01.SPEC-004-AC-08:** Given the client's record changed in another session since Nadia loaded this screen, when she attempts to Archive, then the action is rejected with "This client's records changed since you loaded this page. Refresh to see the latest state before archiving/deleting."

**FEAT-01.SPEC-004-AC-09:** Given Dana is viewing this client in a read-only support session, when she looks for Rename, billing Save, Archive, Reactivate, or Delete, then none of those controls are rendered.

**FEAT-01.SPEC-004-AC-10:** Given Nadia loses connectivity while editing billing details, then the offline banner appears, Save is disabled, and her in-progress edits are preserved.

**FEAT-01.SPEC-004-AC-11:** Given Nadia is on Client Detail, when she taps "Manage Contacts," then she is taken to Client Contact Management & Roles (FEAT-18) for this client.

**FEAT-01.SPEC-004-AC-12:** Given Client Detail fails to load, then an error banner "Couldn't load this client. Try again." appears with a Retry button.

**FEAT-01.SPEC-004-AC-13:** Given Nadia taps "New Project" from Client Detail, then she is taken to FEAT-01.SPEC-002 (Create Project) pre-scoped to this client.

### User Story 5 - Project Detail (Open Project) (Priority: P1)

Single view of one project holding its proposal, milestones, deliverables, invoices, and activity, plus rename, archive/reactivate, and mark-complete actions.

**Acceptance Scenarios:**

**FEAT-01.SPEC-005-AC-01:** Given Nadia is on Project Detail for "Website Redesign," when she renames it to "Website Redesign v2" and confirms, then the header updates and a "Project renamed" toast appears.

**FEAT-01.SPEC-005-AC-02:** Given Nadia is on Project Detail, when she taps the client name link, then she is taken to that client's Client Detail (FEAT-01.SPEC-004).

**FEAT-01.SPEC-005-AC-03:** Given Nadia taps Archive on a project with no unpaid invoices or pending approvals, then the project archives immediately with a "Project archived" toast and no confirmation dialog.

**FEAT-01.SPEC-005-AC-04:** Given Nadia taps Archive on a project with an unpaid invoice, then FEAT-01.SPEC-007's confirmation dialog appears summarizing the open item before the archive completes.

**FEAT-01.SPEC-005-AC-05:** Given Nadia taps Mark Complete on a project whose payment schedule includes an on-completion payment, when she confirms, then FEAT-01.SPEC-006 fires the final invoice, completed_at is set, and the stage badge updates to "Complete."

**FEAT-01.SPEC-005-AC-06:** Given Nadia taps Mark Complete on a project whose payment schedule has no on-completion payment, when she confirms, then completed_at is set and the stage badge updates to "Complete" with no invoice issued.

**FEAT-01.SPEC-005-AC-07:** Given the project was cancelled by FEAT-25 in another session since Nadia loaded this screen, when she attempts Mark Complete, then the action is rejected with "This project was cancelled since you loaded this page. Refresh to see the latest state."

**FEAT-01.SPEC-005-AC-08:** Given a client contact opens the project in their portal after it is marked complete, then the project remains visible to them until it is archived.

**FEAT-01.SPEC-005-AC-09:** Given Dana is viewing this project in a read-only support session, when she looks for Rename, Archive, Reactivate, or Mark Complete, then none of those controls are rendered.

**FEAT-01.SPEC-005-AC-10:** Given Nadia is on Project Detail, when she taps the invoices area, then she is taken to Invoice Generation & Sending (FEAT-09) for this project.

**FEAT-01.SPEC-005-AC-11:** Given Nadia is on Project Detail, when she taps the activity area, then she is taken to the project's trail in Immutable Activity & Audit Trail (FEAT-13).

**FEAT-01.SPEC-005-AC-12:** Given Nadia loses connectivity while viewing an already-loaded project, then the offline banner appears and Rename, Archive, Reactivate, and Mark Complete become disabled.

**FEAT-01.SPEC-005-AC-13:** Given Project Detail fails to load, then an error banner "Couldn't load this project. Try again." appears with a Retry button.

**FEAT-01.SPEC-005-AC-14:** Given Nadia taps Mark Complete, when the confirmation dialog appears, then it states plainly "This marks the project complete and, if the payment schedule includes one, issues the final invoice" before she confirms.

**FEAT-01.SPEC-005-AC-15:** Given Nadia taps Reactivate on an Archived project, when it completes, then the project's Archived status clears, the stage badge updates to its recomputed value, and a "Project reactivated" toast appears.

**FEAT-01.SPEC-005-AC-16:** Given Nadia taps Reactivate on an Archived project that has completed_at set, when it completes, then the stage badge shows "Complete," not "In Progress."

**FEAT-01.SPEC-005-AC-17:** Given Nadia opens the overflow menu on a project that is not Archived, when she looks for Reactivate, then it is not shown, since Reactivate only appears once a project's stage is Archived.

### User Story 6 - Completion Invoice Trigger (Priority: P1)

Marking a project complete fires the on-completion invoice in Invoicing & Payments (FEAT-09) when the project's payment schedule includes one.

**Acceptance Scenarios:**

**FEAT-01.SPEC-006-AC-01:** Given Nadia marks a project complete and its payment schedule includes an on-completion payment, when the trigger fires, then the final invoice is created and sent via FEAT-09, completed_at is set, and the stage recomputes to "Complete."

**FEAT-01.SPEC-006-AC-02:** Given Nadia marks a project complete and its payment schedule has no on-completion payment, when the trigger fires, then completed_at is set and the stage recomputes to "Complete" with no invoice issued.

**FEAT-01.SPEC-006-AC-03:** Given a project has no Payment Schedule at all, when Nadia marks it complete, then it is treated as having no on-completion payment and completes with no invoice.

**FEAT-01.SPEC-006-AC-04:** Given the schedule includes an on-completion payment but FEAT-09 cannot create the invoice, when the trigger fires, then the project still completes and Nadia sees "Project marked complete, but the final invoice couldn't be issued. Retry from the invoices area."

**FEAT-01.SPEC-006-AC-05:** Given the automation fails before completed_at is set (e.g., the schedule cannot be read), when Nadia attempts to mark the project complete, then she sees "Couldn't mark this project complete. Try again." and the project remains in its prior stage.

**FEAT-01.SPEC-006-AC-06:** Given the payment schedule is later adjusted after a completion invoice has already been issued, then the already-issued invoice is unaffected, per the schedule-as-it-stood rule.

**FEAT-01.SPEC-006-AC-07:** Given Nadia has two sessions open on the same project and taps Mark Complete in both at effectively the same time, when the first completes, then the second is rejected with a refresh prompt rather than issuing a second completion invoice.

**FEAT-01.SPEC-006-AC-08:** Given a project was already cancelled by FEAT-25, when Nadia attempts Mark Complete, then the attempt is rejected upstream on FEAT-01.SPEC-005 and this automation never fires.

### User Story 7 - Archive Open-Items Check (Priority: P1)

Checks a client or project for unpaid invoices or pending approvals at the moment of archiving and requires explicit confirmation if any exist.

**Acceptance Scenarios:**

**FEAT-01.SPEC-007-AC-01:** Given Nadia taps Archive on a client with no unpaid invoices and no pending approvals across any of its projects, when the check runs, then the archive proceeds immediately with no confirmation dialog.

**FEAT-01.SPEC-007-AC-02:** Given Nadia taps Archive on a project with one unpaid invoice, when the check runs, then a confirmation dialog names that specific invoice before the archive can proceed.

**FEAT-01.SPEC-007-AC-03:** Given Nadia taps Archive on a client whose only open item is a milestone awaiting approval on one of its projects (all invoices Paid), when the check runs, then the confirmation dialog surfaces the pending approval as the open item.

**FEAT-01.SPEC-007-AC-04:** Given Nadia sees the open-items confirmation dialog and taps Confirm, then the archive proceeds and the standard archive toast appears.

**FEAT-01.SPEC-007-AC-05:** Given Nadia sees the open-items confirmation dialog and taps Decline, then the dialog closes and the client or project remains Active, unchanged.

**FEAT-01.SPEC-007-AC-06:** Given the open-items check itself fails to complete, when Nadia taps Archive, then she sees "Couldn't check this item for open invoices or approvals. Try again." and the archive does not proceed.

**FEAT-01.SPEC-007-AC-07:** Given a project has an Overdue invoice, when the check runs, then the invoice is treated as an open item, since "unpaid" includes Overdue and Payment pending statuses.

**FEAT-01.SPEC-007-AC-08:** Given Nadia has two sessions open on the same client and taps Archive in both at effectively the same time, when the first archive completes, then the second is rejected with a refresh prompt rather than running a redundant archive.

**FEAT-01.SPEC-007-AC-09:** Given a milestone on a project is Approved (not Deliverable Uploaded) and all of that project's invoices are Paid, when Nadia taps Archive on that project, then the check finds no open items and the archive proceeds immediately.

**FEAT-01.SPEC-007-AC-10:** Given Nadia taps Archive on a project (not a client) with no unpaid invoices and no pending approvals, when the check runs, then the archive proceeds immediately with no confirmation dialog, the same as for a client target.

### User Story 8 - Active Client Limit Enforcement (Priority: P1)

Gates adding or reactivating an active client against the freelancer's current Subscription Plan limit.

**Acceptance Scenarios:**

**FEAT-01.SPEC-008-AC-01:** Given Nadia is on the free tier with fewer active clients than platform parameter: `free-tier-active-client-limit`, when she adds a new client, then the client saves as Active and the count increments.

**FEAT-01.SPEC-008-AC-02:** Given Nadia is on the free tier already at platform parameter: `free-tier-active-client-limit` active clients, when she attempts to add a new client, then the save is blocked with "You've reached your plan's active client limit. Upgrade to add more clients." and her entered data is preserved.

**FEAT-01.SPEC-008-AC-03:** Given Nadia is on the free tier at her limit, when she attempts to reactivate an archived client, then the reactivation is blocked with "You've reached your plan's active client limit. Upgrade to reactivate this client." and the client stays Archived.

**FEAT-01.SPEC-008-AC-04:** Given Nadia is on a paid plan, when she adds a new client regardless of her current active-client count, then the save succeeds with no limit block.

**FEAT-01.SPEC-008-AC-05:** Given Nadia archives one of her active clients while at the free-tier limit, when she then adds a new client in the same session, then the add succeeds because the archive freed a slot.

**FEAT-01.SPEC-008-AC-06:** Given Nadia's paid plan lapses while she has more active clients than the free-tier limit allows, then all of her existing active clients remain Active and reachable, but any further add or reactivate is blocked until she is on a plan that accommodates the current count.

**FEAT-01.SPEC-008-AC-07:** Given Owen (Client Primary Contact) has no access to the Client & Project Management area, when he looks for any client-limit-related control, then none is shown, since this capability does not exist in his portal.

**FEAT-01.SPEC-008-AC-08:** Given Nadia has two sessions open and both attempt to add a client at exactly one slot below her limit at effectively the same time, when the first save commits, then the second is blocked with the standard limit message.

**FEAT-01.SPEC-008-AC-09:** Given Dana is in a read-only support session on Nadia's account, when she looks for a way to add or reactivate a client, then neither control is reachable from her session.

### User Story 9 - Client Delete Eligibility (Priority: P1)

Determines whether a client may be hard-deleted (no sent proposal, invoice, or activity exists) versus archived only.

**Acceptance Scenarios:**

**FEAT-01.SPEC-009-AC-01:** Given a client has no sent proposal, no invoice, and no activity, when Nadia opens the overflow menu, then Delete is enabled.

**FEAT-01.SPEC-009-AC-02:** Given a client has one sent invoice, when Nadia opens the overflow menu, then Delete is disabled with the inline text "This client has a sent proposal, invoice, or activity and can only be archived."

**FEAT-01.SPEC-009-AC-03:** Given a client is eligible and Nadia confirms Delete, then the client and its empty projects are permanently removed with no restore path.

**FEAT-01.SPEC-009-AC-04:** Given a client has only a Draft proposal that was never sent, when Nadia opens the overflow menu, then Delete is enabled, since an unsent draft does not count as a "sent proposal."

**FEAT-01.SPEC-009-AC-05:** Given a client had a proposal that was sent, then later voided and never accepted, when Nadia opens the overflow menu, then Delete is disabled, since a voided-but-once-sent proposal still counts.

**FEAT-01.SPEC-009-AC-06:** Given a client passes the eligibility check when Nadia opens the overflow menu, but an ad hoc invoice is issued for that client from another session before she confirms Delete, when she confirms, then the delete is rejected with "This client has a sent proposal, invoice, or activity and can only be archived."

**FEAT-01.SPEC-009-AC-07:** Given Owen has no access to the Client & Project Management area, when he looks for a delete control on any client, then none exists in his portal.

**FEAT-01.SPEC-009-AC-08:** Given Dana is in a read-only support session, when she views a client's overflow options, then no Delete control is rendered.

**FEAT-01.SPEC-009-AC-09:** Given a client's only activity is a read-only support session viewing it (no client-scoped event otherwise), when Nadia opens the overflow menu, then Delete eligibility is unaffected by that support-session view alone.

### User Story 10 - Client Billing Completeness Gate (Priority: P1)

Requires billing name and billing address (tax ID always optional) to be captured before any invoice for the client can be sent -- evaluated fresh at every send attempt, most visibly encountered on the client's first invoice.

**Acceptance Scenarios:**

**FEAT-01.SPEC-010-AC-01:** Given a client has both billing_name and billing_address filled, when Nadia attempts to send that client's first invoice, then the send proceeds with no billing block.

**FEAT-01.SPEC-010-AC-02:** Given a client has billing_name filled but billing_address empty, when Nadia attempts to send that client's first invoice, then the send is blocked with "Billing address is required before sending this client's first invoice."

**FEAT-01.SPEC-010-AC-03:** Given a client has both billing fields empty, when Nadia views Client Detail, then the completeness indicator shows "Billing details incomplete -- required before the first invoice."

**FEAT-01.SPEC-010-AC-04:** Given a client has tax_id empty but both billing_name and billing_address filled, when Nadia attempts to send the first invoice, then the send proceeds, since tax_id never factors into completeness.

**FEAT-01.SPEC-010-AC-05:** Given billing_address contains only whitespace, when completeness is evaluated, then it is treated as empty and the client is incomplete.

**FEAT-01.SPEC-010-AC-06:** Given a client's first invoice has already been sent successfully, when Nadia later clears billing_name on Client Detail, then the already-sent invoice is unaffected, but a subsequent ad hoc invoice attempt for that client is blocked again until billing_name is restored.

**FEAT-01.SPEC-010-AC-07:** Given two sessions of Nadia's edit the same client's billing_address concurrently, when both save, then the later save wins and the completeness indicator reflects it.

**FEAT-01.SPEC-010-AC-08:** Given Owen has no access to billing-detail fields, when he views his portal, then no billing-edit control for his company's details is ever shown to him.

**FEAT-01.SPEC-010-AC-09:** Given Dana views a client in a read-only support session, when she looks at the billing completeness indicator, then she sees its current state but has no control to edit the underlying fields.

### User Story 11 - Project Stage Derivation (Priority: P1)

Computes the roster/detail "stage" label (Draft, In Progress, Complete, Cancelled, Archived) from the project's proposal, milestone, invoice, completion, and cancellation state.

**Acceptance Scenarios:**

**FEAT-01.SPEC-011-AC-01:** Given a project has just been created with no proposal yet, when its stage is derived, then it resolves to "Draft."

**FEAT-01.SPEC-011-AC-02:** Given a project's proposal has been accepted and no milestone is yet approved, when its stage is derived, then it resolves to "In Progress."

**FEAT-01.SPEC-011-AC-03:** Given a project has at least one Approved milestone, when its stage is derived, then it resolves to "In Progress" (if not already Complete, Cancelled, or Archived).

**FEAT-01.SPEC-011-AC-04:** Given Nadia marks a project complete via FEAT-01.SPEC-005/SPEC-006, when its stage is next derived, then it resolves to "Complete," taking precedence over any milestone-based "In Progress" condition.

**FEAT-01.SPEC-011-AC-05:** Given FEAT-25 marks a project cancelled, when its stage is next derived, then it resolves to "Cancelled," taking precedence over its milestone and invoice state.

**FEAT-01.SPEC-011-AC-06:** Given a project is Archived, when its stage is derived, then it resolves to "Archived" regardless of whether completed_at or cancelled_at is also set.

**FEAT-01.SPEC-011-AC-07:** Given a project's proposal is voided and re-sent (still unaccepted), when its stage is re-derived, then it does not resolve to "In Progress" on the basis of the voided proposal, since a Voided proposal is never treated as Accepted.

**FEAT-01.SPEC-011-AC-08:** Given a milestone is approved and, moments later, the same project is marked complete, when its stage is derived after both events, then it resolves to "Complete."

**FEAT-01.SPEC-011-AC-09:** Given Owen views his company's project in the client portal, when he checks its stage, then he sees the same derived value Nadia sees on the roster, since derivation is a single, shared formula.

**FEAT-01.SPEC-011-AC-10:** Given Dana views a project in a read-only support session, when she checks its stage, then she sees the current derived value with no ability to set it directly.

**FEAT-01.SPEC-011-AC-11:** Given no role or screen in this product offers a direct control to set stage, when any user looks for one, then none exists anywhere in the product.

**FEAT-01.SPEC-011-AC-12:** Given an Archived project that also has completed_at set is reactivated via FEAT-01.SPEC-005, when its stage is next derived, then it resolves to "Complete" (not "In Progress"), since completed_at still outranks the milestone/invoice condition in precedence once the Archived override is cleared.

### Edge Cases

- **FEAT-01.SPEC-001 (Add Client):** Unsaved edits trigger a Discard / Keep Editing confirmation, a double tap on Save is ignored while saving, and a network failure shows a retry banner with the form data preserved. A save blocked by the active-client limit (FEAT-01.SPEC-008) keeps the entered data and offers an Upgrade action; duplicate client names are allowed, and offline submissions queue and retry without creating duplicates. Source: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-001-add-client.md` (section: Edge Cases)
- **FEAT-01.SPEC-002 (Create Project):** Same unsaved-change, double-tap and network-failure handling as the client form, with form data preserved on failure. If the owning client is archived or deleted from another session before save, the save is rejected and the user returns to the roster. Source: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-002-create-project.md` (section: Edge Cases)
- **FEAT-01.SPEC-003 (Client & Project Roster):** Skeleton rows show while a large list loads; the roster is a per-load snapshot that can be stale until the next load or filter change. All projects of a client are shown without a cap, and a filter with zero matches shows a filter-specific empty message rather than the first-client prompt. Source: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-003-client-project-roster.md` (section: Edge Cases)
- **FEAT-01.SPEC-004 (Client Detail):** Descriptive edits are last-write-wins across sessions, while Archive, Delete and Reactivate use reject-with-refresh when the client changed since load. Archiving with unpaid invoices or pending approvals routes through the open-items confirmation (FEAT-01.SPEC-007), and reactivating beyond the active-client limit is blocked with an Upgrade action. Source: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-004-client-detail.md` (section: Edge Cases)
- **FEAT-01.SPEC-005 (Project Detail (Open Project)):** Renames are last-write-wins; Archive, Reactivate and Mark Complete use reject-with-refresh when state changed since load, including a project cancelled by FEAT-25 in another session. Archiving with open items goes through the confirmation in FEAT-01.SPEC-007. Source: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-005-project-detail.md` (section: Edge Cases)
- **FEAT-01.SPEC-006 (Completion Invoice Trigger):** A project with no payment schedule completes with no invoice; if FEAT-09 cannot create the on-completion invoice, completion is not rolled back and a non-blocking warning with retry is shown. A project already cancelled upstream is rejected at the triggering screen, and concurrent Mark Complete attempts from two sessions fire only once. Source: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-006-completion-invoice-trigger.md` (section: Edge Cases)
- **FEAT-01.SPEC-007 (Archive Open-Items Check):** The open-items confirmation names the specific project and invoice, and requires both the invoice check and the pending-approval check to be clear before reporting no open items. It reflects state read at check time; concurrent archives resolve first-wins with the second rejected. Source: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-007-archive-open-items-check.md` (section: Edge Cases)
- **FEAT-01.SPEC-008 (Active Client Limit Enforcement):** The limit check applies after a just-completed archive, so an add can succeed immediately afterwards; when two sessions race for the last slot the first commit wins and the second is rejected with its data preserved. A lapsed plan keeps existing clients Active (XBR-23) but blocks further create or reactivate attempts, and reactivation is checked one client at a time. Source: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-008-active-client-limit-enforcement.md` (section: Edge Cases)
- **FEAT-01.SPEC-009 (Client Delete Eligibility):** A never-sent draft proposal or entirely empty projects do not block delete, but a voided-and-never-accepted sent proposal does. Eligibility is re-run at commit, so a record created from another session after the menu opened causes a rejection. Source: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-009-client-delete-eligibility.md` (section: Edge Cases)
- **FEAT-01.SPEC-010 (Client Billing Completeness Gate):** A missing billing_name or billing_address makes the gate false and the send-time block names the missing field; whitespace-only values count as empty and tax_id never blocks. Clearing billing details later does not affect invoices already sent (XBR-04). Source: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-010-client-billing-completeness-gate.md` (section: Edge Cases)
- **FEAT-01.SPEC-011 (Project Stage Derivation):** Stage precedence resolves conflicts deterministically: Archived and Cancelled override milestone or invoice-driven In Progress and Complete, an accepted proposal alone yields In Progress, and a project with no proposal falls back to Draft. Source: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-011-project-stage-derivation.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-01.SPEC-001** (Add Client) as specified: Nadia records a new client company she works with, so a project can be created under it. Full spec: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-001-add-client.md`
- **FR-002**: The system MUST implement **FEAT-01.SPEC-002** (Create Project) as specified: Nadia creates a project under a specific client, entering the roster at stage "Draft." Full spec: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-002-create-project.md`
- **FR-003**: The system MUST implement **FEAT-01.SPEC-003** (Client & Project Roster) as specified: Nadia (and, read-only, Dana in a support session) sees every client and project she manages in one place, each showing its current stage. Full spec: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-003-client-project-roster.md`
- **FR-004**: The system MUST implement **FEAT-01.SPEC-004** (Client Detail) as specified: Single view of one client -- rename it, edit billing details, view its contacts, and archive, reactivate, or delete it. Full spec: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-004-client-detail.md`
- **FR-005**: The system MUST implement **FEAT-01.SPEC-005** (Project Detail (Open Project)) as specified: Single view of one project holding its proposal, milestones, deliverables, invoices, and activity, plus rename, archive/reactivate, and mark-complete actions. Full spec: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-005-project-detail.md`
- **FR-006**: The system MUST implement **FEAT-01.SPEC-006** (Completion Invoice Trigger) as specified: Marking a project complete fires the on-completion invoice in Invoicing & Payments (FEAT-09) when the project's payment schedule includes one. Full spec: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-006-completion-invoice-trigger.md`
- **FR-007**: The system MUST implement **FEAT-01.SPEC-007** (Archive Open-Items Check) as specified: Checks a client or project for unpaid invoices or pending approvals at the moment of archiving and requires explicit confirmation if any exist. Full spec: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-007-archive-open-items-check.md`
- **FR-008**: The system MUST implement **FEAT-01.SPEC-008** (Active Client Limit Enforcement) as specified: Gates adding or reactivating an active client against the freelancer's current Subscription Plan limit. Full spec: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-008-active-client-limit-enforcement.md`
- **FR-009**: The system MUST implement **FEAT-01.SPEC-009** (Client Delete Eligibility) as specified: Determines whether a client may be hard-deleted (no sent proposal, invoice, or activity exists) versus archived only. Full spec: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-009-client-delete-eligibility.md`
- **FR-010**: The system MUST implement **FEAT-01.SPEC-010** (Client Billing Completeness Gate) as specified: Requires billing name and billing address (tax ID always optional) to be captured before any invoice for the client can be sent -- evaluated fresh at every send attempt, most visibly encountered on the client's first invoice. Full spec: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-010-client-billing-completeness-gate.md`
- **FR-011**: The system MUST implement **FEAT-01.SPEC-011** (Project Stage Derivation) as specified: Computes the roster/detail "stage" label (Draft, In Progress, Complete, Cancelled, Archived) from the project's proposal, milestone, invoice, completion, and cancellation state. Full spec: `docs/blueprint/specifications/FEAT-01-client-project-management/FEAT-01.SPEC-011-project-stage-derivation.md`

### Key Entities

- Client (create, read, update, archive)
- Project (create, read, update, archive)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Nadia can add a client and create a project under it in under 2 minutes, from opening the add-client action to the project appearing in her roster (metric: Client and Project Setup Speed). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Every lifecycle action on the feature's records (client_created, project_created, project_archived, client_archived, project_opened, project_marked_complete, client_deleted) is observable as a distinct signal. Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-13**: Solo-freelancer-only platform, with no team seats or internal role hierarchy in v1. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-22**: A few thousand freelancers in year one, each with 3-15 active clients, which sizes the roster and active-client limit behavior. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-12**: The permanent free tier for one or two active clients is the basis of the active-client limit this feature enforces. Full register: `docs/blueprint/features/assumptions-constraints.md`
