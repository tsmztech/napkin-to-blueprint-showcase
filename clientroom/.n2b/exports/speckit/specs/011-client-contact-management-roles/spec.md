# Feature Specification: Client Contact Management & Roles

**Blueprint feature:** FEAT-18
**Priority tier:** Important
**Build order:** 011 of 33
**Depends on:** FEAT-01
**Blueprint source:** `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Client Contact List (Priority: P2)

Nadia views and manages the contacts and roles for one client company; Dana views the same list read-only inside a logged support session.

**Acceptance Scenarios:**

**FEAT-18.SPEC-001-AC-01:** Given Nadia opens a client with three contacts, when the screen loads, then she sees all three contacts listed with name, email, role badge, and status.

**FEAT-18.SPEC-001-AC-02:** Given Nadia is on the Client Contact List for a client with no contacts, when the screen loads, then she sees "No contacts yet. Add a Primary contact before you can send a proposal to this client." with an "Add Contact" action.

**FEAT-18.SPEC-001-AC-03:** Given Nadia taps "Add Contact", when the tap registers, then she is navigated to FEAT-18.SPEC-002 in create mode, scoped to this client.

**FEAT-18.SPEC-001-AC-04:** Given Nadia taps an existing contact's row, when the tap registers, then she is navigated to FEAT-18.SPEC-002 in edit mode, pre-filled with that contact's current data.

**FEAT-18.SPEC-001-AC-05:** Given Nadia taps "Remove" on a contact row, when the tap registers, then she is navigated to FEAT-18.SPEC-003 for that contact.

**FEAT-18.SPEC-001-AC-06:** Given Nadia arrives here from a delivery warning on a bouncing contact email, when the screen loads, then that contact's row is highlighted with an inline note "This email is bouncing."

**FEAT-18.SPEC-001-AC-07:** Given a client has one or more Reviewer contacts but no Primary contact, when Nadia views this list, then an inline banner reads "This client has no Primary contact. A proposal cannot be sent until one is added or promoted."

**FEAT-18.SPEC-001-AC-08:** Given Dana is inside an open support session on Nadia's account, when she navigates to a client's contacts, then she sees the same list with no "Add Contact", "Edit", or "Remove" controls rendered, under the permanent read-only banner.

**FEAT-18.SPEC-001-AC-09:** Given Dana views the Client Contact List, when she looks for a way to change a contact, then no such control exists, and there is no way to trigger a change from this screen.

**FEAT-18.SPEC-001-AC-10:** Given Owen or Priya is signed in to the client portal, when they look for any route to this screen, then none exists -- the screen is not part of the client portal's navigation.

**FEAT-18.SPEC-001-AC-11:** Given the initial data load for this screen fails, when the failure occurs, then Nadia sees "Couldn't load contacts. Try again." with a Retry button.

**FEAT-18.SPEC-001-AC-12:** Given a contact was removed in another of Nadia's sessions while this list was open, when Nadia returns to this list from FEAT-18.SPEC-002 or FEAT-18.SPEC-003, then the list re-fetches and no longer shows the removed contact.

### User Story 2 - Add or Edit Client Contact (Priority: P2)

Nadia adds a new contact to a client company or edits an existing contact's name, email, or role.

**Acceptance Scenarios:**

**FEAT-18.SPEC-002-AC-01:** Given Nadia is on this screen in create mode, when she enters "Owen Carter" as name, "owen@acme.test" as email, selects Primary, and taps Save, then the contact is created with status Invited, the invitation email (FEAT-18.SPEC-010) fires, and she sees "Contact added" before returning to FEAT-18.SPEC-001.

**FEAT-18.SPEC-002-AC-02:** Given Nadia taps Save with the name field empty, then the name field shows "Name is required" and the save does not proceed.

**FEAT-18.SPEC-002-AC-03:** Given Nadia enters an email already used by another contact at the same client, when she blurs the email field, then it shows "This email is already used by another contact at this client."

**FEAT-18.SPEC-002-AC-04:** Given Nadia opens an existing Reviewer contact in edit mode and changes the role to Primary, when she taps Save, then a note confirms the change applies to future actions only, and the save completes.

**FEAT-18.SPEC-002-AC-05:** Given Nadia opens the client's only Primary contact in edit mode and changes the role to Reviewer, when she taps Save, then the save is blocked with "This is the client's only Primary contact. Add or promote another Primary before changing this one."

**FEAT-18.SPEC-002-AC-06:** Given Nadia has unsaved changes on this screen, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?"

**FEAT-18.SPEC-002-AC-07:** Given Nadia loses connectivity while attempting to save, then the error banner "Couldn't save this contact. Try again." appears with a Retry button, and her entered data is preserved.

**FEAT-18.SPEC-002-AC-08:** Given a contact Nadia is editing was updated in another of her sessions before she saves, when she taps Save, then the save is rejected with "This contact was updated in another session. Review the latest version before saving." and "View Latest" / "Keep Editing" options.

**FEAT-18.SPEC-002-AC-09:** Given Nadia taps Save twice in rapid succession, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-18.SPEC-002-AC-10:** Given Nadia arrives from a delivery warning on a bouncing contact email, when the screen opens, then it is in edit mode for that contact with the email field focused.

**FEAT-18.SPEC-002-AC-11:** Given Owen, Priya, or Dana attempts to reach this screen through any route, then none exists for any of them -- the screen is not part of the client portal or the support console.

**FEAT-18.SPEC-002-AC-12:** Given Nadia's session expires while she has unsaved form data, when the expiry dialog appears and she re-authenticates, then her entered field values are restored.

**FEAT-18.SPEC-002-AC-13:** Given Nadia enters a name and email but leaves Role at its default, when she taps Save, then the Role selection is required and validation blocks the save if no role was ever selected.

**FEAT-18.SPEC-002-AC-14:** Given Nadia opens an existing contact in edit mode, when the contact is still being fetched, then she sees skeleton placeholders in place of the fields and Save is disabled until the data has loaded.

**FEAT-18.SPEC-002-AC-15:** Given the edit-mode fetch of the contact fails, when the failure occurs, then Nadia sees "Couldn't load this contact. Try again." with a Retry button and no editable form, and when she taps Retry and the fetch succeeds, then the form is pre-filled with the contact's current name, email, and role.

### User Story 3 - Remove Client Contact (Priority: P2)

Nadia confirms removing a contact, sees the last-Primary block when it applies, and is shown erasure-request framing before the removal executes.

**Acceptance Scenarios:**

**FEAT-18.SPEC-003-AC-01:** Given Nadia taps "Remove" on a Reviewer contact, when the screen loads, then she sees the standard confirmation naming the contact and explaining that removal ends access and erases their email and other contact details while their approvals stay on record under their name only.

**FEAT-18.SPEC-003-AC-02:** Given Nadia is on the standard confirmation, when she checks "I understand this cannot be undone" and taps "Remove Contact", then FEAT-18.SPEC-009 is triggered and she sees "Contact removed" before returning to FEAT-18.SPEC-001.

**FEAT-18.SPEC-003-AC-03:** Given Nadia taps "Remove" on the client's only Primary contact, when the screen loads, then she sees the blocked case: "{contact_name} is this client's only Primary contact. Add or promote a replacement Primary before removing them." with no "Remove Contact" action.

**FEAT-18.SPEC-003-AC-04:** Given Nadia opens the standard confirmation for a Primary contact who is not the client's last Primary, when she confirms removal, then the removal proceeds normally.

**FEAT-18.SPEC-003-AC-05:** Given Nadia has the standard confirmation open for a Primary contact and another Primary contact is removed from this client in a different session first, when Nadia taps "Remove Contact", then the commit-time re-check finds this is now the last Primary and the screen switches in place to the blocked case with no removal performed.

**FEAT-18.SPEC-003-AC-06:** Given Nadia is on the blocked case, when she taps "Go to Contacts", then she is navigated to FEAT-18.SPEC-001, from which she can add or promote a replacement Primary.

**FEAT-18.SPEC-003-AC-07:** Given Nadia has not checked the acknowledgement checkbox, when she looks at "Remove Contact", then it is disabled and cannot be tapped.

**FEAT-18.SPEC-003-AC-08:** Given Nadia taps "Remove Contact" twice in rapid succession, when the first removal is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-18.SPEC-003-AC-09:** Given a network failure occurs during removal, when the failure is detected, then the error banner "Couldn't remove this contact. Try again." appears with a Retry button, and the contact remains unremoved.

**FEAT-18.SPEC-003-AC-10:** Given Owen, Priya, or Dana attempts to reach this screen through any route, then none exists for any of them.

**FEAT-18.SPEC-003-AC-11:** Given Nadia removes a contact who is currently signed in to the client portal, when the removal completes, then any subsequent action that contact attempts in the client portal is rejected per FEAT-05's access rules.

**FEAT-18.SPEC-003-AC-12:** Given Nadia taps "Remove" on a contact, when the contact and the client's Primary contacts are still being read, then she sees a loading placeholder with no "Remove Contact" action until the standard or blocked case is determined.

**FEAT-18.SPEC-003-AC-13:** Given the read of the contact or the client's Primary contacts fails, when the failure occurs, then Nadia sees "Couldn't load this contact. Try again." with a Retry button and no "Remove Contact" action, and when she taps Retry and the read succeeds, then the standard or blocked case is shown as appropriate.

### User Story 4 - Invite Reviewer Colleague (Priority: P2)

Owen, a Client Primary Contact, invites a colleague at his own client company as a Reviewer contact from his portal view.

**Acceptance Scenarios:**

**FEAT-18.SPEC-004-AC-01:** Given Owen is on this screen, when he enters "Priya Shah" as name and "priya@acme.test" as email and taps "Send Invite", then a Reviewer contact is created with status Invited, both FEAT-18.SPEC-010 and FEAT-18.SPEC-011 fire, and he sees "Invitation sent" before returning to Portal Home.

**FEAT-18.SPEC-004-AC-02:** Given Owen is on this screen, when he looks for a role selector, then none is shown -- the role is displayed as a fixed "Reviewer" label.

**FEAT-18.SPEC-004-AC-03:** Given Owen taps "Send Invite" with the name field empty, then the name field shows "Name is required" and the send does not proceed.

**FEAT-18.SPEC-004-AC-04:** Given Owen enters an email already used by another contact at his own company, when he blurs the email field, then it shows "This email is already used by another contact at this company."

**FEAT-18.SPEC-004-AC-05:** Given Owen is on this screen, when he expands the existing-contacts list, then he sees every contact currently at his own client company with their role and status, and no contact from any other client company.

**FEAT-18.SPEC-004-AC-06:** Given Owen has unsaved changes on this screen, when he taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?"

**FEAT-18.SPEC-004-AC-07:** Given Owen loses connectivity while sending an invite, then the error banner "Couldn't send this invitation. Try again." appears with a Retry button, and his entered data is preserved.

**FEAT-18.SPEC-004-AC-08:** Given Owen and Nadia both submit a new contact with the same email for the same client at effectively the same time (Owen here, Nadia on FEAT-18.SPEC-002), when the second save is processed, then it is rejected with the per-client email-uniqueness error.

**FEAT-18.SPEC-004-AC-09:** Given Priya is signed in to the client portal, when she looks at Portal Home, then no "Invite a colleague" action is shown to her, and she has no route to this screen.

**FEAT-18.SPEC-004-AC-10:** Given Nadia or Dana attempts to reach this screen through any route, then none exists for either of them -- this screen exists only inside the client portal.

**FEAT-18.SPEC-004-AC-11:** Given Owen taps "Send Invite" twice in rapid succession, when the first send is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-18.SPEC-004-AC-12:** Given Owen opens this screen, when his company's existing contacts are still being fetched, then the list shows skeleton rows and "Send Invite" is disabled until the list has loaded.

**FEAT-18.SPEC-004-AC-13:** Given the fetch of Owen's own company's contact list fails, when the failure occurs, then Owen sees "Couldn't load your colleagues. Try again." with a Retry button and "Send Invite" stays disabled, and when he taps Retry and the fetch succeeds, then the list appears and "Send Invite" is enabled.

### User Story 5 - Contact Field Validation Rules (Priority: P2)

Defines all field validation, cross-field, and role-assignment-scope rules enforced on every contact add, edit, or invite across the feature.

**Acceptance Scenarios:**

**FEAT-18.SPEC-005-AC-01:** Given Nadia is adding a contact and leaves the name field empty, when she blurs the field, then it shows "Name is required."

**FEAT-18.SPEC-005-AC-02:** Given Nadia enters "not-an-email" in the email field, when she blurs the field, then it shows "Please enter a valid email address."

**FEAT-18.SPEC-005-AC-03:** Given Nadia enters an email that already belongs to another live contact at the same client, when she blurs the email field, then it shows "This email is already used by another contact at this client."

**FEAT-18.SPEC-005-AC-04:** Given Nadia enters a valid, unique name and email and selects a role, when she submits, then all field validation passes.

**FEAT-18.SPEC-005-AC-05:** Given Nadia does not select a role, when she submits the form, then she sees "Please select a role" and the save does not proceed.

**FEAT-18.SPEC-005-AC-06:** Given Owen is inviting a colleague, when he views the role field, then no selectable control is shown -- it is a fixed "Reviewer" label, and validation never presents an error for it under normal use.

**FEAT-18.SPEC-005-AC-07:** Given a request reaches this rule set attempting to set role to Primary from Owen's invite enforcement point, when validation runs, then it is blocked with "Only a Reviewer role can be invited from this screen."

**FEAT-18.SPEC-005-AC-08:** Given the same email exists for two different client companies of the same freelancer, when Nadia adds a contact with that email to a third, unrelated client, then no uniqueness violation occurs, since the check is scoped per client company.

**FEAT-18.SPEC-005-AC-09:** Given a contact with a given email was previously removed (and their email erased), when Nadia adds a new contact with that same email at the same client, then no uniqueness violation occurs.

**FEAT-18.SPEC-005-AC-10:** Given Nadia enters "Owen@Acme.test" while "owen@acme.test" already exists at the same client, when she blurs the field, then it is rejected as a duplicate regardless of letter case.

**FEAT-18.SPEC-005-AC-11:** Given Nadia enters an email with leading whitespace, when the field is validated, then the whitespace is trimmed before format and uniqueness checks run.

**FEAT-18.SPEC-005-AC-12:** Given Nadia edits an existing contact's email to match another live contact at the same client, when she attempts to save, then the save is rejected with the duplicate-email error and no change is persisted.

**FEAT-18.SPEC-005-AC-13:** Given Owen and Nadia each submit a new contact with the same new email for the same client at effectively the same time, when the second submission is processed, then it is rejected-with-refresh on the uniqueness check.

**FEAT-18.SPEC-005-AC-14:** Given a new contact is successfully created by Nadia, when the record is saved, then invited_by is set to Nadia's identity and status is set to Invited, with neither field ever presented to Nadia as an entry field.

### User Story 6 - Primary Contact Requirement Rule (Priority: P2)

Enforces that a client must have at least one Primary contact before a proposal can be sent, and blocks removing or demoting the last Primary contact until a replacement is designated.

**Acceptance Scenarios:**

**FEAT-18.SPEC-006-AC-01:** Given a client has exactly one Primary contact, when Nadia attempts to remove that contact on FEAT-18.SPEC-003, then the removal is blocked with "{contact_name} is this client's only Primary contact. Add or promote a replacement Primary before removing them."

**FEAT-18.SPEC-006-AC-02:** Given a client has two Primary contacts, when Nadia removes one of them, then the removal proceeds, since one Primary contact still remains.

**FEAT-18.SPEC-006-AC-03:** Given a client has exactly one Primary contact, when Nadia edits that contact on FEAT-18.SPEC-002 and changes the role to Reviewer, then the save is blocked with "This is the client's only Primary contact. Add or promote another Primary before changing this one."

**FEAT-18.SPEC-006-AC-04:** Given a client has zero Primary contacts, when Nadia or Owen attempts to send a proposal for that client (FEAT-02), then the send is blocked with "This client has no Primary contact. Add one before sending a proposal."

**FEAT-18.SPEC-006-AC-05:** Given a client has zero contacts at all, when Nadia adds the first contact and assigns Primary, then the proposal-send block for that client is lifted immediately.

**FEAT-18.SPEC-006-AC-06:** Given Nadia has the removal confirmation for the client's last Primary open, when another Primary contact is added to that client in a different session before she confirms, then her commit-time re-check finds a Primary contact still exists and the removal proceeds.

**FEAT-18.SPEC-006-AC-07:** Given Nadia has the removal confirmation for the client's last Primary open, when no other Primary is added before she confirms, then the commit-time re-check still finds zero other Primary contacts and the removal remains blocked.

**FEAT-18.SPEC-006-AC-08:** Given a proposal was already accepted under a Primary contact who is later removed as the client's last Primary at that time, when the removal is evaluated, then the prior acceptance record is unaffected and remains valid evidence under that contact's name.

**FEAT-18.SPEC-006-AC-09:** Given Owen or Priya has no remove or role-change entitlement on any contact, when either looks for a way to trigger this rule's block, then no such control exists for them anywhere in the product.

**FEAT-18.SPEC-006-AC-10:** Given a client has one Primary and several Reviewer contacts, when Nadia attempts to remove the sole Primary, then the block message is identical regardless of how many Reviewer contacts also exist.

**FEAT-18.SPEC-006-AC-11:** Given Nadia's account is being deleted via FEAT-24, when the account-deletion process removes the client's sole Primary contact as part of removing the whole account, then this rule does not block that removal, since FEAT-24 removes the freelancer's data as a unit rather than performing an in-product contact removal.

**FEAT-18.SPEC-006-AC-12:** Given a rapid duplicate removal submission targets a contact that a first submission already removed, when the second submission's commit-time check runs, then it finds the contact already Removed and treats the action as already complete rather than surfacing a stale-record error.

### User Story 7 - Role Authorization Rules (Priority: P2)

Enforces every Client Contact action's entitlement by role -- Nadia's full management, Owen's own-company invite-only entitlement, Priya's total exclusion, and Dana's read-only view -- and states exactly what each denied role experiences.

**Acceptance Scenarios:**

**FEAT-18.SPEC-007-AC-01:** Given Nadia owns a client, when she opens FEAT-18.SPEC-001 for that client, then she sees the full contact list with all management controls.

**FEAT-18.SPEC-007-AC-02:** Given Owen is signed in to his client portal, when he looks for a route to FEAT-18.SPEC-001, then none exists.

**FEAT-18.SPEC-007-AC-03:** Given Dana opens a support session on Nadia's account, when she navigates to a client's contacts, then she sees the full list read-only, with no create, edit, or remove control rendered.

**FEAT-18.SPEC-007-AC-04:** Given Owen is on FEAT-18.SPEC-004, when he submits a new contact, then it is created as a Reviewer at his own client company.

**FEAT-18.SPEC-007-AC-05:** Given Owen has no edit entitlement, when he looks for a way to change his own contact record's name, email, or role, then no such control exists anywhere in the client portal.

**FEAT-18.SPEC-007-AC-06:** Given Priya is signed in to her client portal, when she looks for any contact-management action (add, edit, remove, invite), then none is shown to her anywhere.

**FEAT-18.SPEC-007-AC-07:** Given Nadia attempts to remove a contact, when the client has more than one Primary contact or the target is a Reviewer, then the removal proceeds (subject to FEAT-18.SPEC-006's last-Primary check).

**FEAT-18.SPEC-007-AC-08:** Given Owen, Priya, or Dana attempts to remove any contact, when they look for a remove control, then none exists for any of the three roles.

**FEAT-18.SPEC-007-AC-09:** Given Owen views his own company's contact list inline on FEAT-18.SPEC-004, when the list renders, then it shows only contacts at his own client company and never another company's contacts.

**FEAT-18.SPEC-007-AC-10:** Given Dana's support session on Nadia's account closes, when she is returned to the Operator Support Session Console, then her prior read-only access to that account's contacts ends immediately.

**FEAT-18.SPEC-007-AC-11:** Given Nadia is the sole freelancer-side actor with contact-management access, when she views any client's contacts, then no other freelancer-side role or seat exists to share or restrict that access with (Access Matrix: Nadia -- Full; no other internal role, per SC-01).

**FEAT-18.SPEC-007-AC-12:** Given Priya is later promoted to Primary by Nadia, when the change takes effect, then Priya becomes authorized to invite Reviewer colleagues at her own company from that point forward, per FEAT-18.SPEC-008's future-only timing.

**FEAT-18.SPEC-007-AC-13:** Given Owen's own client company already has two Primary contacts, when Owen invites a new Reviewer colleague, then the invite proceeds normally, unaffected by how many Primary contacts exist.

**FEAT-18.SPEC-007-AC-14:** Given a contact's status is Removed, when any role's screen would otherwise list it, then it never appears, for any role including Dana's read-only view.

**FEAT-18.SPEC-007-AC-15:** Given Nadia attempts to create a contact for a client she owns, when she assigns either Primary or Reviewer, then the create is authorized for both role values.

**FEAT-18.SPEC-007-AC-16:** Given Owen attempts to assign a role other than Reviewer on FEAT-18.SPEC-004, when the attempt reaches this rule set, then it is denied, since no such control is ever presented and the field-level restriction (FEAT-18.SPEC-005) additionally blocks it.

**FEAT-18.SPEC-007-AC-17:** Given Dana is not inside an open support session, when she attempts to view any freelancer's contacts, then no access is granted -- the read-only view exists only for the duration of an open session (FEAT-31).

**FEAT-18.SPEC-007-AC-18:** Given Nadia deletes her account via FEAT-24, when the deletion completes, then no role retains any access to that account's former Client Contact records, since the account and its data no longer exist.

**FEAT-18.SPEC-007-AC-19:** Given Owen's own contact record is removed by Nadia while he has an in-progress invite screen open, when FEAT-18.SPEC-009 revokes his access, then his session is ended by FEAT-05's access rules and any invite he had already completed before the removal remains valid.

### User Story 8 - Role Change Effective-Timing Rule (Priority: P2)

Ensures a contact's role change applies to their future actions only, and never alters the record of a past approval or acceptance they gave under their prior role.

**Acceptance Scenarios:**

**FEAT-18.SPEC-008-AC-01:** Given Owen accepted a proposal as a Primary contact, when Nadia later changes his role to Reviewer, then the prior acceptance record remains unchanged and still shows Owen as the accepting party.

**FEAT-18.SPEC-008-AC-02:** Given Priya is promoted from Reviewer to Primary, when she subsequently attempts to accept a proposal, then the acceptance is permitted, since her current role at the moment of acceptance is Primary.

**FEAT-18.SPEC-008-AC-03:** Given a contact's role is changed from Primary to Reviewer, when that contact attempts to approve a milestone after the change, then the approval is denied, per their new current role.

**FEAT-18.SPEC-008-AC-04:** Given Owen has the proposal-accept screen open and Nadia changes his role to Reviewer before he taps Accept, when he taps Accept, then the action is evaluated against his role at that moment (Reviewer) and is denied.

**FEAT-18.SPEC-008-AC-05:** Given Nadia changes a contact's role from Primary to Reviewer and back to Primary within the same day, when the activity trail is viewed, then both role-change events appear as separate, independently timestamped entries.

**FEAT-18.SPEC-008-AC-06:** Given a Reviewer contact posted comments before being promoted to Primary, when their comment history is viewed after the promotion, then those comments remain visible and attributed to them exactly as posted, unaffected by the later promotion.

**FEAT-18.SPEC-008-AC-07:** Given a client's sole Primary contact is demoted to Reviewer immediately after an invoice's pay link was generated in their name, when the invoice is later viewed, then the pay link and invoice remain addressed to that contact as originally recorded.

**FEAT-18.SPEC-008-AC-08:** Given Owen taps Approve on a milestone at the same instant Nadia's demotion of him commits first, when the approval attempt is then evaluated, then it is denied under his now-current Reviewer role.

**FEAT-18.SPEC-008-AC-09:** Given Owen's approval commits before Nadia's concurrent demotion of him, when the demotion is applied afterward, then the already-recorded approval is not undone or altered.

**FEAT-18.SPEC-008-AC-10:** Given a contact's role has never changed since creation, when any of their past or future actions are evaluated, then this rule has no observable effect, since there is only ever one role value in their history.

**FEAT-18.SPEC-008-AC-11:** Given Nadia views a past milestone approval in the activity trail for a contact whose role has since changed, when she reads the entry, then it shows the role the contact held at the time of that approval, not their current role.

### User Story 9 - Contact Removal & Data Erasure (Priority: P2)

Ends a removed contact's access immediately, erases their email address and other contact details, and retains their evidentiary records under their name only.

**Acceptance Scenarios:**

**FEAT-18.SPEC-009-AC-01:** Given Nadia confirms removing a Reviewer contact and the trigger fires, when this automation runs, then the contact's status is set to Removed, their email and other contact details are erased while their name is retained, and an Activity Log Entry carrying the name only is written.

**FEAT-18.SPEC-009-AC-02:** Given the removed contact previously accepted a proposal, when their acceptance record is viewed afterward, then it still shows their name as the accepting party via the retained Client Contact row, shows no email address for them, and is otherwise unaffected by the erasure.

**FEAT-18.SPEC-009-AC-03:** Given the removed contact is currently signed in to the client portal when this automation runs, when they next attempt any action, then it is rejected because their contact record no longer resolves as recognized.

**FEAT-18.SPEC-009-AC-04:** Given this automation completes successfully, when FEAT-18.SPEC-001 is next reloaded, then the removed contact no longer appears in the list.

**FEAT-18.SPEC-009-AC-05:** Given the removed contact's erased email is later entered for a new contact at the same client, when the uniqueness check runs (FEAT-18.SPEC-005), then no conflict is found.

**FEAT-18.SPEC-009-AC-06:** Given a removal is triggered a second time for a contact already Removed (a duplicate rapid submission), when this automation receives the second trigger, then it makes no further change and produces the same success outcome shown for the first.

**FEAT-18.SPEC-009-AC-07:** Given this automation cannot complete the status change or field erasure for a processing reason, when the failure occurs, then no partial change is left in place, and Nadia sees "Couldn't remove this contact. Try again." with a Retry button.

**FEAT-18.SPEC-009-AC-08:** Given a client's other contacts and its own Client and Project records, when a contact is removed, then none of those other records are affected.

**FEAT-18.SPEC-009-AC-09:** Given the removed contact left comments on a deliverable before removal, when those comments are viewed afterward, then they remain visible and attributed to the contact's name, with no email address shown.

**FEAT-18.SPEC-009-AC-10:** Given the removal's Activity Log Entry write fails independently of the status change and erasure succeeding, when this occurs, then the removal itself is not rolled back, Nadia still sees "Contact removed", and the trail write is retried per FEAT-13's own retry behavior.

**FEAT-18.SPEC-009-AC-11:** Given two removal confirmations for the same contact fire in quick succession, when both reach this automation, then the first produces the Removal completed outcome and the second produces the idempotent Already removed outcome, with no duplicate trail entry.

**FEAT-18.SPEC-009-AC-12:** Given a removal is triggered by an explicit data-subject erasure request rather than a routine departure, when this automation runs, then it behaves identically to the routine case -- erasing the email and other contact details and preserving evidentiary records the same way.

**FEAT-18.SPEC-009-AC-13:** Given a removal is in progress for one contact, when a second, unrelated removal is triggered for a different contact at the same client at the same time, then both proceed independently without queuing behind each other.

### User Story 10 - New Contact Invitation Email (Priority: P2)

Sends a newly added or invited contact their first magic-link sign-in invitation, so they can reach the client portal for the first time.

**Acceptance Scenarios:**

**FEAT-18.SPEC-010-AC-01:** Given Nadia adds a new Primary contact directly, when the save completes, then that contact receives an email with subject "You've been added to {freelancer_business_name}'s Clientroom portal" describing their Primary entitlements and a "Get started" link.

**FEAT-18.SPEC-010-AC-02:** Given Owen invites a Reviewer colleague, when the invite send completes, then the new contact receives an email with subject "{Owen's name} invited you to {freelancer_business_name}'s Clientroom portal" describing Reviewer entitlements and a "Get started" link.

**FEAT-18.SPEC-010-AC-03:** Given the new contact taps "Get started" in the email, when the link opens, then they land on FEAT-05.SPEC-002 with their issued token.

**FEAT-18.SPEC-010-AC-04:** Given the freelancer's `business_name` is not yet set, when this email is composed, then the subject and body render with "the Clientroom portal" in place of the business name.

**FEAT-18.SPEC-010-AC-05:** Given a contact is removed before this email is delivered, when the delivery would otherwise occur, then it is cancelled and no email is sent.

**FEAT-18.SPEC-010-AC-06:** Given delivery of this email fails on the first attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before giving up.

**FEAT-18.SPEC-010-AC-07:** Given all retries for this email are exhausted, when the final attempt fails, then Nadia sees a delivery warning on the client's contact record.

**FEAT-18.SPEC-010-AC-08:** Given this notification has no preference control for the recipient, when a new contact is created, then the email always sends -- there is no opt-out surface to check.

**FEAT-18.SPEC-010-AC-09:** Given a new contact is created at any hour, when this notification is triggered, then it sends immediately with no quiet-hours hold, since the underlying credential is time-limited.

**FEAT-18.SPEC-010-AC-10:** Given the freelancer's Branding Profile has a logo and brand colour set, when this email renders, then it displays that branding alongside the "Made with Clientroom" referral mark.

**FEAT-18.SPEC-010-AC-11:** Given the same person is added as a contact to two different clients of the same freelancer, when both creations complete, then each produces its own separate email with its own token scoped to that specific client.

**FEAT-18.SPEC-010-AC-12:** Given the token this email carries expires before the email is delivered due to an extreme delivery delay, when the recipient clicks "Get started", then they see "This link isn't valid anymore" and can request a fresh one from FEAT-05.SPEC-002.

### User Story 11 - Primary-Invited-Colleague Alert (Priority: P2)

Alerts Nadia by email whenever a Primary contact invites a colleague, so she always knows who can see her work.

**Acceptance Scenarios:**

**FEAT-18.SPEC-011-AC-01:** Given Owen invites Priya as a Reviewer colleague, when the invite send completes, then Nadia receives an email with subject "Owen invited a colleague to {client_company_name}" naming Priya and her email.

**FEAT-18.SPEC-011-AC-02:** Given Nadia receives this alert, when she taps "View contacts", then she is navigated to FEAT-18.SPEC-001 scoped to the affected client.

**FEAT-18.SPEC-011-AC-03:** Given Nadia herself adds a new contact directly on FEAT-18.SPEC-002, when the save completes, then this alert is not sent, since it exists only for Primary-contact-initiated invites.

**FEAT-18.SPEC-011-AC-04:** Given two different Primary contacts at two different clients each invite a colleague within moments of each other, when both sends complete, then Nadia receives two separate alerts, one per client.

**FEAT-18.SPEC-011-AC-05:** Given Owen invites two colleagues in quick succession, when both sends complete, then Nadia receives two separate alerts, not one batched alert.

**FEAT-18.SPEC-011-AC-06:** Given the invited contact is removed before this alert is delivered, when delivery proceeds, then the alert still sends, reporting the invite event as it happened.

**FEAT-18.SPEC-011-AC-07:** Given delivery of this alert fails on the first attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`.

**FEAT-18.SPEC-011-AC-08:** Given all retries for this alert are exhausted, when the final attempt fails, then the failure is surfaced to Nadia per XBR-30's general delivery-warning behavior.

**FEAT-18.SPEC-011-AC-09:** Given this notification has no preference control, when a Primary contact invites a colleague, then the alert always sends to Nadia with no opt-out surface to check.

**FEAT-18.SPEC-011-AC-10:** Given this alert is triggered at any hour, when it fires, then it sends immediately with no quiet-hours hold, since the product defines no quiet-hours window for freelancer-side account alerts.

### Edge Cases

- **FEAT-18.SPEC-001 (Client Contact List):** The list is a snapshot loaded on open, so contacts added or changed from other sessions appear only after a reload, and removing the last Primary from another session is guarded by the commit-time re-check in FEAT-18.SPEC-006. A client with only Reviewers displays normally with an inline notice that sending a proposal needs a Primary. Source: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-001-client-contact-list.md` (section: Edge Cases)
- **FEAT-18.SPEC-002 (Add or Edit Client Contact):** Unsaved edits trigger the discard confirmation, double taps on Save are ignored, and a failed load in edit mode shows a Load Error with Retry and no editable form. A network failure on save preserves the form data with a retry banner. Source: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-002-add-or-edit-client-contact.md` (section: Edge Cases)
- **FEAT-18.SPEC-003 (Remove Client Contact):** A contact who becomes the last Primary between load and the Remove tap is caught by the commit-time re-check, which switches the screen to the blocked case. Double taps are ignored, a failed load never shows the standard confirmation, and a network failure leaves the contact unchanged. Source: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-003-remove-client-contact.md` (section: Edge Cases)
- **FEAT-18.SPEC-004 (Invite Reviewer Colleague):** Unsaved edits trigger the discard confirmation, double taps on Send Invite are ignored, and a network failure preserves the form. An email matching another contact at the company is rejected per FEAT-18.SPEC-005. Source: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-004-invite-reviewer-colleague.md` (section: Edge Cases)
- **FEAT-18.SPEC-005 (Contact Field Validation Rules):** Email uniqueness is case-insensitive and whitespace is trimmed before format and uniqueness checks, a whitespace-only name counts as empty, and editing a contact's email to match another live contact is rejected on blur and on submit. Source: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-005-contact-field-validation-rules.md` (section: Edge Cases)
- **FEAT-18.SPEC-006 (Primary Contact Requirement Rule):** Adding a second Primary then removing the original is permitted because one Primary remains at commit time, and a concurrent promotion of a Reviewer wins if it completes before the removal's check. Account deletion (FEAT-24) is not blocked by this rule, and an already Sent or Accepted proposal is not retroactively affected by removing the sole Primary. Source: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-006-primary-contact-requirement-rule.md` (section: Edge Cases)
- **FEAT-18.SPEC-007 (Role Authorization Rules):** A client contact has no route to the freelancer-side contact list, an operator's view-only access ends when the support session closes, and a Primary's invite entitlement is unaffected by other Primaries at the company. A Reviewer promoted to Primary gains the Primary rows going forward. Source: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-007-role-authorization-rules.md` (section: Edge Cases)
- **FEAT-18.SPEC-008 (Role Change Effective-Timing Rule):** An action not yet completed is evaluated against the contact's role at the moment of the attempt, so a Reviewer demoted mid-accept is blocked and a freshly promoted Primary may act at once. Rapid back-and-forth changes are each timestamped, and comments already posted keep their authorship regardless of later role changes. Source: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-008-role-change-effective-timing-rule.md` (section: Edge Cases)
- **FEAT-18.SPEC-009 (Contact Removal & Data Erasure):** A signed-in contact's session ends immediately on removal, an erased email becomes reusable in the per-client uniqueness check, and a concurrent list read keeps showing the contact until reload. Duplicate removal confirmations resolve first-commit-wins. Source: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-009-contact-removal-data-erasure.md` (section: Edge Cases)
- **FEAT-18.SPEC-010 (New Contact Invitation Email):** A contact removed before delivery cancels the pending invitation because the token is already revoked, and the same person added to two clients gets two separate invitations with separate tokens. An inviter name change renders the current name and a token expiring before delivery shows the link-not-valid message on click (FEAT-05.SPEC-002). Source: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-010-new-contact-invitation-email.md` (section: Edge Cases)
- **FEAT-18.SPEC-011 (Primary-Invited-Colleague Alert):** The alert still sends even if the invited contact was removed before delivery, since it reports an event that already happened. Invites from different Primaries or two quick invites from one Primary each produce their own alert, and permanent delivery failure only triggers the standard failure surfacing (XBR-30). Source: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-011-primary-invited-colleague-alert.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-18.SPEC-001** (Client Contact List) as specified: Nadia views and manages the contacts and roles for one client company; Dana views the same list read-only inside a logged support session. Full spec: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-001-client-contact-list.md`
- **FR-002**: The system MUST implement **FEAT-18.SPEC-002** (Add or Edit Client Contact) as specified: Nadia adds a new contact to a client company or edits an existing contact's name, email, or role. Full spec: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-002-add-or-edit-client-contact.md`
- **FR-003**: The system MUST implement **FEAT-18.SPEC-003** (Remove Client Contact) as specified: Nadia confirms removing a contact, sees the last-Primary block when it applies, and is shown erasure-request framing before the removal executes. Full spec: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-003-remove-client-contact.md`
- **FR-004**: The system MUST implement **FEAT-18.SPEC-004** (Invite Reviewer Colleague) as specified: Owen, a Client Primary Contact, invites a colleague at his own client company as a Reviewer contact from his portal view. Full spec: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-004-invite-reviewer-colleague.md`
- **FR-005**: The system MUST implement **FEAT-18.SPEC-005** (Contact Field Validation Rules) as specified: Defines all field validation, cross-field, and role-assignment-scope rules enforced on every contact add, edit, or invite across the feature. Full spec: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-005-contact-field-validation-rules.md`
- **FR-006**: The system MUST implement **FEAT-18.SPEC-006** (Primary Contact Requirement Rule) as specified: Enforces that a client must have at least one Primary contact before a proposal can be sent, and blocks removing or demoting the last Primary contact until a replacement is designated. Full spec: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-006-primary-contact-requirement-rule.md`
- **FR-007**: The system MUST implement **FEAT-18.SPEC-007** (Role Authorization Rules) as specified: Enforces every Client Contact action's entitlement by role -- Nadia's full management, Owen's own-company invite-only entitlement, Priya's total exclusion, and Dana's read-only view -- and states exactly what each denied role experiences. Full spec: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-007-role-authorization-rules.md`
- **FR-008**: The system MUST implement **FEAT-18.SPEC-008** (Role Change Effective-Timing Rule) as specified: Ensures a contact's role change applies to their future actions only, and never alters the record of a past approval or acceptance they gave under their prior role. Full spec: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-008-role-change-effective-timing-rule.md`
- **FR-009**: The system MUST implement **FEAT-18.SPEC-009** (Contact Removal & Data Erasure) as specified: Ends a removed contact's access immediately, erases their email address and other contact details, and retains their evidentiary records under their name only. Full spec: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-009-contact-removal-data-erasure.md`
- **FR-010**: The system MUST implement **FEAT-18.SPEC-010** (New Contact Invitation Email) as specified: Sends a newly added or invited contact their first magic-link sign-in invitation, so they can reach the client portal for the first time. Full spec: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-010-new-contact-invitation-email.md`
- **FR-011**: The system MUST implement **FEAT-18.SPEC-011** (Primary-Invited-Colleague Alert) as specified: Alerts Nadia by email whenever a Primary contact invites a colleague, so she always knows who can see her work. Full spec: `docs/blueprint/specifications/FEAT-18-client-contact-management-roles/FEAT-18.SPEC-011-primary-invited-colleague-alert.md`

### Key Entities

- Client Contact (create, update, remove)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Contact additions, role changes, Primary-initiated invitations and removals are each observable as distinct signals (contact_added, contact_role_changed, contact_invited_by_primary, contact_removed); no metric in the success-metrics register connects to this feature, so the outcome is grounded in its Signals alone. Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-20**: Evidence outlives a contact's erasure request, but only as far as needed. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-23**: Strict data isolation between clients, with a Primary or Reviewer role model. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-24**: Personal data of client contacts is treated as GDPR-class personal data. Full register: `docs/blueprint/features/assumptions-constraints.md`
