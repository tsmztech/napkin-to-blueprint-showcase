# Feature Specification: Proposal Creation & Sending

**Blueprint feature:** FEAT-02
**Priority tier:** Core
**Build order:** 003 of 33
**Depends on:** FEAT-01, FEAT-15
**Blueprint source:** `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Proposal Draft Editor (Priority: P1)

Nadia writes scope, price, and currency for a project's proposal from a blank form, from a copied earlier proposal, or by revising a Sent-but-unaccepted proposal.

**Acceptance Scenarios:**

**FEAT-02.SPEC-001-AC-01:** Given Nadia opens the proposal area of a project with no existing proposal, when the screen loads, then the form shows empty Scope Description and Price fields with Currency pre-filled from the project.

**FEAT-02.SPEC-001-AC-02:** Given Nadia fills in a scope description and a positive price and taps "Save Draft", then the system saves the Proposal as Draft and shows the toast "Draft saved".

**FEAT-02.SPEC-001-AC-03:** Given Nadia leaves the Scope Description field empty and moves focus away, then the field shows the error "Scope description is required" per FEAT-02.SPEC-010.

**FEAT-02.SPEC-001-AC-04:** Given Nadia has filled in valid scope and price and taps "Preview", then validation passes, the content saves as Draft, and she is navigated to FEAT-02.SPEC-002 (Proposal Preview).

**FEAT-02.SPEC-001-AC-05:** Given Nadia opens the editor from FEAT-02.SPEC-008 after selecting a source proposal, when the screen loads, then Scope Description, Price, and Currency are pre-filled from the source and the copied-from banner shows the source project's name.

**FEAT-02.SPEC-001-AC-06:** Given Nadia opens a Sent-but-unaccepted proposal in edit mode, when she revises the price and taps "Save & Resend", then FEAT-02.SPEC-006 is triggered and, on success, she is navigated to FEAT-02.SPEC-003 showing the updated Sent state.

**FEAT-02.SPEC-001-AC-07:** Given Nadia is on a Draft and taps "Discard Draft", when she confirms in the dialog, then FEAT-02.SPEC-009 is triggered and she is navigated to FEAT-02.SPEC-003 showing the empty state.

**FEAT-02.SPEC-001-AC-08:** Given Nadia has unsaved changes on the form, when she taps the back arrow, then the dialog "You have unsaved changes. Discard?" appears with "Discard" and "Keep Editing".

**FEAT-02.SPEC-001-AC-09:** Given Nadia loses connectivity while filling the form, then the banner "You're offline -- your changes are kept on this device until you reconnect." appears and Preview, Save Draft, and Save & Resend are disabled until connectivity returns.

**FEAT-02.SPEC-001-AC-10:** Given Dana (Support Operator) opens this screen inside a logged support session, when she views the form, then all field values render read-only with no Save, Preview, Save & Resend, or Discard controls.

**FEAT-02.SPEC-001-AC-11:** Given Nadia edited and saved-and-resent a Sent-but-unaccepted proposal from a second open session while this session's edit was also pending, when this session's "Save & Resend" completes its check, then the save is refused with "This proposal was already edited and resent. Review the current version." and the screen reloads the current version's content.

**FEAT-02.SPEC-001-AC-12:** Given Owen accepted the proposal while Nadia's edit screen was open, when Nadia taps "Save & Resend", then the save is refused with the dialog "This proposal has already been accepted and can no longer be edited." and a "View Current Status" option to FEAT-02.SPEC-003.

### User Story 2 - Proposal Preview (Priority: P1)

Nadia reviews a branded, client-facing rendering of the current draft's scope, price, and payment schedule before sending it to the client.

**Acceptance Scenarios:**

**FEAT-02.SPEC-002-AC-01:** Given Nadia taps "Preview" from a valid Draft, when the Preview screen loads, then it shows the branded rendering of the scope, price, currency, and payment schedule exactly as saved on the Draft.

**FEAT-02.SPEC-002-AC-02:** Given Nadia is viewing the Preview and the client has a Primary contact, when she taps "Send", then FEAT-02.SPEC-005 is triggered and, on success, she is navigated to FEAT-02.SPEC-003 with the confirmation "Proposal sent to {Primary Contact name}."

**FEAT-02.SPEC-002-AC-03:** Given the client has no Primary contact, when Nadia taps "Send" from Preview, then the Send Blocked banner "This client has no Primary contact yet." appears with a link into FEAT-18, and no send occurs.

**FEAT-02.SPEC-002-AC-04:** Given Nadia taps the back arrow on Preview, then she is returned to FEAT-02.SPEC-001 with the same content still populated.

**FEAT-02.SPEC-002-AC-05:** Given Nadia taps "Send" and the operation fails due to a network error, then an error banner appears with a Retry option and the proposal remains in Draft status.

**FEAT-02.SPEC-002-AC-06:** Given Nadia loses connectivity while viewing Preview, then the banner "You're offline -- sending requires a connection." appears and Send is disabled.

**FEAT-02.SPEC-002-AC-07:** Given Dana (Support Operator) opens this screen inside a logged support session, when she views it, then the content renders read-only with no Send control shown.

**FEAT-02.SPEC-002-AC-08:** Given Nadia taps "Send" twice in rapid succession, then the second tap is ignored while the first send is in progress.

**FEAT-02.SPEC-002-AC-09:** Given Nadia returns to Preview after changing the price in the editor, when the screen reloads, then it shows the updated price, not the previously previewed value.

**FEAT-02.SPEC-002-AC-10:** Given Nadia taps "Preview" and the Branding Profile, Payment Schedule, and Client/Project names have not yet finished loading, when the screen opens, then skeleton placeholders appear until the fetch completes, after which the full branded rendering displays.

### User Story 3 - Proposal Detail (Priority: P1)

Nadia (and, read-only, Dana) views a project's current proposal -- its status, history, and the actions available for that status -- and Dana's support view is limited to status only.

**Acceptance Scenarios:**

**FEAT-02.SPEC-003-AC-01:** Given Nadia opens the proposal area of a project with no proposal, when the screen loads, then it shows "This project has no proposal yet." with a "Draft a Proposal" button.

**FEAT-02.SPEC-003-AC-02:** Given Nadia taps "Draft a Proposal", then she is navigated to FEAT-02.SPEC-001 with a blank form.

**FEAT-02.SPEC-003-AC-03:** Given a project has a Draft proposal, when Nadia opens the Detail screen, then she sees the "Draft" status badge with Edit, Send, Discard, and Start from Copy actions.

**FEAT-02.SPEC-003-AC-04:** Given a project has a Sent proposal, when Nadia opens the Detail screen, then she sees the "Sent" status badge, the sent_at timestamp, and Edit and Resend actions (no Discard, no Start from Copy).

**FEAT-02.SPEC-003-AC-05:** Given a project has an Accepted proposal, when Nadia opens the Detail screen, then she sees the "Accepted" status badge with accepted_at and accepted_by, and no action buttons.

**FEAT-02.SPEC-003-AC-06:** Given a Sent proposal has a request-changes note attached, when Nadia opens the Detail screen, then the note's text, author, and posted date are shown.

**FEAT-02.SPEC-003-AC-07:** Given Nadia taps "Resend" on a Sent proposal, then FEAT-02.SPEC-007 is triggered and the toast "Proposal link resent to {Primary Contact name}." appears.

**FEAT-02.SPEC-003-AC-08:** Given Nadia taps "Discard" on a Draft, when she confirms in the dialog, then FEAT-02.SPEC-009 is triggered and the screen shows the empty state afterward.

**FEAT-02.SPEC-003-AC-09:** Given Nadia opens the change-request email from FEAT-03, when she follows the link, then she lands on this screen with the request-changes note visible.

**FEAT-02.SPEC-003-AC-10:** Given Dana (Support Operator) opens this screen inside a logged support session, when she views it, then she sees only the status and history, with no scope description, price, request-changes note text, or action buttons.

**FEAT-02.SPEC-003-AC-11:** Given the screen is loading, then skeleton placeholders appear for the status card and content summary until data arrives.

**FEAT-02.SPEC-003-AC-12:** Given loading the proposal's data fails, then the error banner "Could not load this proposal." appears with a Retry button.

**FEAT-02.SPEC-003-AC-13:** Given Nadia loses connectivity after this screen has loaded, then the banner "You're offline -- showing the last loaded version of this proposal." appears and all action buttons are disabled.

**FEAT-02.SPEC-003-AC-14:** Given the proposal was discarded in another session, when Nadia returns to this screen, then it re-fetches and shows the empty state rather than the stale Draft.

### User Story 4 - Reuse Proposal Picker (Priority: P1)

Nadia browses her earlier proposals across all clients and projects and selects one to start a new draft as a copy.

**Acceptance Scenarios:**

**FEAT-02.SPEC-004-AC-01:** Given Nadia has 3 earlier proposals across 2 clients, when she opens the Reuse Proposal Picker, then all 3 appear, ordered most-recently-sent first, each showing client, project, status, date, price, and a scope excerpt.

**FEAT-02.SPEC-004-AC-02:** Given Nadia has never sent a proposal, when she opens the picker, then it shows "You don't have any earlier proposals to start from yet." with no list.

**FEAT-02.SPEC-004-AC-03:** Given Nadia types a client name into the search input that matches one proposal, then the list filters to show only that proposal.

**FEAT-02.SPEC-004-AC-04:** Given Nadia searches for a name matching no proposal, then "No proposals match {query}." is shown.

**FEAT-02.SPEC-004-AC-05:** Given Nadia taps a proposal row, when the copy completes, then she is navigated to FEAT-02.SPEC-001 with scope, price, and currency pre-filled from the selected proposal.

**FEAT-02.SPEC-004-AC-06:** Given Nadia taps a proposal row and the copy automation fails, then an inline error "Could not start from this proposal. Try again." appears on that row and she remains on the picker.

**FEAT-02.SPEC-004-AC-07:** Given the picker is loading, then skeleton placeholder rows appear until data arrives.

**FEAT-02.SPEC-004-AC-08:** Given loading the list fails, then the error banner "Could not load your earlier proposals." appears with a Retry button.

**FEAT-02.SPEC-004-AC-09:** Given Nadia loses connectivity while viewing the picker, then the banner "You're offline -- showing the last loaded list." appears and row selection is disabled.

### User Story 5 - Proposal Send (Priority: P1)

Validates and transitions a Draft proposal to Sent, recording the send timestamp and locking the payment schedule reference so the client always sees the schedule as it stood at send.

**Acceptance Scenarios:**

**FEAT-02.SPEC-005-AC-01:** Given Nadia's Draft proposal has valid scope, a positive price, and the client has a Primary contact, when Send is triggered from FEAT-02.SPEC-002, then the proposal transitions to Sent, sent_at is recorded, and FEAT-02.SPEC-011 is triggered.

**FEAT-02.SPEC-005-AC-02:** Given the client has no Primary contact, when Send is triggered, then the proposal remains Draft and the Blocked -- no Primary contact outcome is returned.

**FEAT-02.SPEC-005-AC-03:** Given the proposal's scope description is empty, when Send is triggered, then the proposal remains Draft and the Blocked -- invalid fields outcome is returned.

**FEAT-02.SPEC-005-AC-04:** Given the proposal was already sent from another session before this Send call reaches the status check, when this automation runs, then it returns the state-mismatch outcome ("This proposal has already been sent.") and no second Sent version or email is created.

**FEAT-02.SPEC-005-AC-05:** Given the Draft was discarded (FEAT-02.SPEC-009) from another session before this Send call reaches the status check, when this automation runs, then it returns the distinct discarded-draft outcome ("This draft no longer exists -- it was discarded."), not the state-mismatch message, and Preview navigates to FEAT-02.SPEC-003's empty state.

**FEAT-02.SPEC-005-AC-06:** Given Send succeeds, then the proposal's payment_schedule_reference is locked to the Payment Schedule as it stood at that moment.

**FEAT-02.SPEC-005-AC-07:** Given a processing failure occurs after eligibility checks pass but before the transition completes, then the proposal remains Draft and no email is sent.

**FEAT-02.SPEC-005-AC-08:** Given two Preview sessions for the same Draft trigger Send at effectively the same time, then only the first transitions the proposal to Sent and the second receives the state-mismatch outcome.

**FEAT-02.SPEC-005-AC-09:** Given Send succeeds, then an Activity Log Entry recording the send is written (FEAT-13).

### User Story 6 - Proposal Edit-Before-Acceptance Void & Resend (Priority: P1)

When Nadia edits a Sent-but-unaccepted proposal, voids the prior version and creates and sends the new one, so the client is never shown an outdated price (XBR-06).

**Acceptance Scenarios:**

**FEAT-02.SPEC-006-AC-01:** Given Nadia edits a Sent-but-unaccepted proposal's price and taps "Save & Resend", when validation passes, then the prior version is set to Voided, a new Proposal is created and Sent with the edited content, and FEAT-02.SPEC-011 is triggered.

**FEAT-02.SPEC-006-AC-02:** Given the target proposal's status is Accepted at the moment of the check, when this automation runs, then it is refused with "This proposal has already been accepted and can no longer be edited." and no void occurs.

**FEAT-02.SPEC-006-AC-03:** Given the target proposal was already voided by a concurrent edit, when this automation runs for the second session, then it is refused with "This proposal was already edited and resent. Review the current version." and the second session's editor reloads the new current version.

**FEAT-02.SPEC-006-AC-04:** Given the edited scope description is empty, when Save & Resend is triggered, then the void does not occur and the field-level error is shown per FEAT-02.SPEC-010.

**FEAT-02.SPEC-006-AC-05:** Given the client has no Primary contact at the moment of this edit, when Save & Resend is triggered, then the void does not occur and the Primary-contact eligibility error is shown.

**FEAT-02.SPEC-006-AC-06:** Given Owen accepts the proposal in the instant before this automation's status check runs, when the automation proceeds, then it reads Accepted and refuses with the already-accepted outcome, leaving Owen's acceptance intact.

**FEAT-02.SPEC-006-AC-07:** Given two sessions trigger Save & Resend on the same Sent proposal at effectively the same time, then only the first succeeds and the second receives the already-voided-or-superseded outcome.

**FEAT-02.SPEC-006-AC-08:** Given a processing failure occurs after eligibility passes but before the transition completes, then the prior proposal remains Sent and unvoided, and no new version is created.

**FEAT-02.SPEC-006-AC-09:** Given void-and-resend succeeds, then a single Activity Log Entry records both the void and the new send as one event (FEAT-13).

### User Story 7 - Proposal Resend (Priority: P1)

Re-sends the link for an already-Sent proposal without creating a new version, for the case where Owen simply cannot find the original email.

**Acceptance Scenarios:**

**FEAT-02.SPEC-007-AC-01:** Given a proposal is Sent and the client has a Primary contact, when Nadia taps "Resend", then FEAT-02.SPEC-011 is triggered and the toast "Proposal link resent to {Primary Contact name}." appears, with the proposal's status, content, and sent_at unchanged.

**FEAT-02.SPEC-007-AC-02:** Given the proposal was voided since the Detail screen loaded, when Nadia taps "Resend", then the automation reports the state-mismatch outcome and the screen re-fetches to show the current status.

**FEAT-02.SPEC-007-AC-03:** Given Nadia resent the proposal moments ago, when she taps "Resend" again within the cooldown window, then the toast "You already resent this proposal recently. Try again in a few minutes." appears and no email is sent.

**FEAT-02.SPEC-007-AC-04:** Given the client's last Primary contact was removed since the original send, when Nadia taps "Resend", then "This client has no Primary contact yet." appears with a link into FEAT-18.

**FEAT-02.SPEC-007-AC-05:** Given a processing failure occurs during resend, then the toast "Could not resend the proposal link. Try again." appears.

**FEAT-02.SPEC-007-AC-06:** Given two Detail sessions trigger Resend for the same proposal at effectively the same time, then at most one email is delivered and the second trigger receives the rate-limited outcome.

**FEAT-02.SPEC-007-AC-07:** Given Resend succeeds, then an Activity Log Entry recording the resend is written (FEAT-13).

### User Story 8 - Create Draft From Copy (Priority: P1)

Creates a new Draft proposal for the target project, pre-filled from a selected earlier proposal's scope, price, and currency.

**Acceptance Scenarios:**

**FEAT-02.SPEC-008-AC-01:** Given Nadia selects a source proposal for a target project with no existing proposal, when the copy completes, then a new Draft is created with the source's scope description and price, the target project's own currency and payment schedule reference, and copied_from set to the source.

**FEAT-02.SPEC-008-AC-02:** Given the target project already has a Draft proposal, when Nadia selects a source to copy from, then the copy is blocked with "This project already has a proposal. Open it from the project instead." and no new Draft is created.

**FEAT-02.SPEC-008-AC-03:** Given the source proposal's currency differs from the target project's currency, when the copy completes, then the price number is carried over unconverted and the editor displays the target project's own currency alongside it.

**FEAT-02.SPEC-008-AC-04:** Given a processing failure occurs during the copy, then the Reuse Picker shows "Could not start from this proposal. Try again." and Nadia remains on the picker.

**FEAT-02.SPEC-008-AC-05:** Given two copy operations for the same target project fire at effectively the same time, then only the first creates a Draft and the second receives the active-proposal-exists blocked outcome.

**FEAT-02.SPEC-008-AC-06:** Given the copy succeeds, when the editor opens, then it shows the copied-from banner naming the source project.

**FEAT-02.SPEC-008-AC-07:** Given the source proposal is voided in another session after this copy has already completed, then the resulting Draft is unaffected and remains fully editable.

### User Story 9 - Discard Draft (Priority: P1)

Permanently removes an unsent Draft proposal that Nadia no longer wants to keep.

**Acceptance Scenarios:**

**FEAT-02.SPEC-009-AC-01:** Given a project has a Draft proposal, when Nadia confirms "Discard Draft" from FEAT-02.SPEC-001, then the Proposal record is permanently deleted and the screen navigates to the empty state.

**FEAT-02.SPEC-009-AC-02:** Given a project has a Draft proposal, when Nadia confirms "Discard" from FEAT-02.SPEC-003, then the Proposal record is permanently deleted and the empty state is shown.

**FEAT-02.SPEC-009-AC-03:** Given the proposal was sent from another session in the moment before this automation's status check runs, when the discard is attempted, then it is refused with "This proposal was just sent and can no longer be discarded." and the proposal is not deleted.

**FEAT-02.SPEC-009-AC-04:** Given a processing failure occurs during discard, then the error banner "Could not discard this draft. Try again." appears and the Draft is preserved.

**FEAT-02.SPEC-009-AC-05:** Given two sessions confirm Discard for the same Draft at effectively the same time, then only the first deletes the record and the second is redirected to the empty state without error.

**FEAT-02.SPEC-009-AC-06:** Given a Draft that was created from a copy (FEAT-02.SPEC-008) is discarded, then the source proposal it was copied from is unaffected.

### User Story 10 - Proposal Validation & Business Rules (Priority: P1)

Governs required fields, price positivity and currency, the one-active-proposal-per-project cap, send eligibility (XBR-07), immutability after send and after acceptance (XBR-04), and the accept-vs-void contention resolution for the Proposal entity.

**Acceptance Scenarios:**

**FEAT-02.SPEC-010-AC-01:** Given Nadia leaves scope_description empty on a Draft, when she attempts to save or send, then "Scope description is required" is shown and the operation is blocked.

**FEAT-02.SPEC-010-AC-02:** Given Nadia enters a scope description of exactly 10,000 characters, when she saves, then validation passes; at 10,001 characters, "Scope description must be 10,000 characters or fewer" is shown.

**FEAT-02.SPEC-010-AC-03:** Given Nadia leaves price empty, when she attempts to save or send, then "Price is required" is shown.

**FEAT-02.SPEC-010-AC-04:** Given Nadia enters a price of zero, when she attempts to save or send, then "Price must be a positive amount" is shown.

**FEAT-02.SPEC-010-AC-05:** Given Nadia enters a price of 1.00 in the project's currency, when she saves, then validation passes.

**FEAT-02.SPEC-010-AC-06:** Given a project already has an active (Draft, Sent, or Accepted) proposal, when Nadia attempts to start a new Draft for it via the reuse path, then "This project already has a proposal. Open it from the project instead." is shown and no second Draft is created.

**FEAT-02.SPEC-010-AC-07:** Given the client has no Primary contact, when Nadia attempts to send a valid Draft, then "This client has no Primary contact yet." is shown and the send is blocked.

**FEAT-02.SPEC-010-AC-08:** Given the client has at least one Primary contact and the Draft's fields are valid, when Nadia sends it, then the send proceeds.

**FEAT-02.SPEC-010-AC-09:** Given a proposal's status is Accepted, when Nadia attempts to edit it, then "This proposal has already been accepted and can no longer be edited." is shown and no edit occurs.

**FEAT-02.SPEC-010-AC-10:** Given a proposal's status is Sent and unaccepted, when Nadia edits and saves it, then the edit is allowed through the void-and-resend path (FEAT-02.SPEC-006), not a direct update.

**FEAT-02.SPEC-010-AC-11:** Given a proposal's status is Voided, when any attempt is made to accept it, then the acceptance is refused (owned by FEAT-03) and the current version is shown instead.

**FEAT-02.SPEC-010-AC-12:** Given Nadia (Freelancer) views any of her own proposals, then she sees full content (scope, price, status, history) regardless of status.

**FEAT-02.SPEC-010-AC-13:** Given Owen (Client Primary Contact) attempts to reach a proposal belonging to a different client company, then no route exists and, if an out-of-scope link is followed, the plain explanation defined by XBR-09 is shown.

**FEAT-02.SPEC-010-AC-14:** Given Priya (Client Reviewer Contact) views her portal home, then she sees only the project's stage and never the proposal's scope or price.

**FEAT-02.SPEC-010-AC-15:** Given Dana (Support Operator) views a proposal inside a logged support session, then she sees status and history only, with no scope_description, price, or request-changes note text.

**FEAT-02.SPEC-010-AC-16:** Given a proposal's status is Draft, when Nadia looks for Discard, then it is shown and, on confirmation, the proposal is permanently deleted.

**FEAT-02.SPEC-010-AC-17:** Given a proposal's status is Sent, when Nadia looks for Discard, then it is not shown.

**FEAT-02.SPEC-010-AC-18:** Given Owen, Priya, or Dana looks for Create, Edit, Send, Resend, or Discard controls on any proposal, then none of these controls are ever shown to them.

**FEAT-02.SPEC-010-AC-19:** Given a new blank or copied Draft is created, then its status defaults to Draft and its currency is derived from the owning project, never independently settable.

**FEAT-02.SPEC-010-AC-20:** Given a proposal transitions from Draft to Sent, then sent_at is set to the current date and time and payment_schedule_reference is locked to the project's Payment Schedule at that moment.

**FEAT-02.SPEC-010-AC-21:** Given Owen's acceptance and Nadia's void-and-resend race against the same Sent proposal, when one completes first, then exactly one of "acceptance recorded" or "version voided" prevails and the other is refused -- never both.

**FEAT-02.SPEC-010-AC-22:** Given a proposal was created via FEAT-02.SPEC-008 from a source that is later voided, when Nadia views the new Draft, then copied_from still references the source and the new Draft remains fully editable and independent.

### User Story 11 - Proposal Sent/Resent Email (Priority: P1)

Emails the client's Primary Contact the proposal link whenever a proposal is sent, resent, or re-sent after an edit, so the client can review and act on it without a "PDF in email" back-and-forth.

**Acceptance Scenarios:**

**FEAT-02.SPEC-011-AC-01:** Given a Draft proposal is successfully sent (FEAT-02.SPEC-005), when this notification fires, then Owen receives an email with the subject "New proposal from {freelancer_business_name}: {project_name}" and a "View Proposal" CTA.

**FEAT-02.SPEC-011-AC-02:** Given a Sent-but-unaccepted proposal is edited and re-sent (FEAT-02.SPEC-006), when this notification fires, then Owen receives an email with the subject "Updated proposal from {freelancer_business_name}: {project_name}" stating the earlier version is no longer valid.

**FEAT-02.SPEC-011-AC-03:** Given Nadia resends an unchanged Sent proposal (FEAT-02.SPEC-007), when this notification fires, then Owen receives an email with the subject "Reminder: proposal from {freelancer_business_name} for {project_name}".

**FEAT-02.SPEC-011-AC-04:** Given the client has two Primary contacts, when a proposal is sent, then both receive their own copy of the email, each addressed individually.

**FEAT-02.SPEC-011-AC-05:** Given Priya (Reviewer) is a contact on the client company, when a proposal is sent, then she does not receive this email.

**FEAT-02.SPEC-011-AC-06:** Given Owen taps the "View Proposal" CTA, then he is routed through magic-link sign-in (FEAT-05) and lands on his portal proposal view (FEAT-03) for this project.

**FEAT-02.SPEC-011-AC-07:** Given delivery of the original-send email fails, when the retry window is exhausted, then a delivery warning appears on the project for Nadia and no further automatic retry occurs.

**FEAT-02.SPEC-011-AC-08:** Given the original send's email delivery is still retrying when the proposal is edited and re-sent, then the original email is not delivered further and only the edited version's email proceeds.

**FEAT-02.SPEC-011-AC-09:** Given the Primary contact's email address is updated between trigger and actual delivery, then the email is delivered to the current address at delivery time.

**FEAT-02.SPEC-011-AC-10:** Given the freelancer has not set a Branding Profile, when this email renders, then it uses the neutral default branding.

**FEAT-02.SPEC-011-AC-11:** Given Nadia has not set any notification preference for this email, when a proposal is sent, then the email always sends -- there is no preference control that can turn it off, per XBR-30.

### Edge Cases

- **FEAT-02.SPEC-001 (Proposal Draft Editor):** Unsaved edits trigger the discard confirmation, double taps on Preview or Save & Resend are ignored, and a network failure on Save Draft shows a retry banner with content preserved. Editing a Sent-but-unaccepted proposal from two sessions is reject-with-refresh against the proposal's current state. Source: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-001-proposal-draft-editor.md` (section: Edge Cases)
- **FEAT-02.SPEC-002 (Proposal Preview):** Double taps on Send are ignored, and a network failure leaves the proposal in Draft with no partial Sent state. If the client's last Primary contact is removed before Send, the send is blocked with a link into FEAT-18; Preview always reads the Draft's current saved values. Source: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-002-proposal-preview.md` (section: Edge Cases)
- **FEAT-02.SPEC-003 (Proposal Detail):** This read-only snapshot re-fetches on next load or resume rather than live-pushing, so a concurrent void-and-resend, discard or client acceptance appears on the next load. Very long request-changes notes are truncated to a preview with Show more. Source: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-003-proposal-detail.md` (section: Edge Cases)
- **FEAT-02.SPEC-004 (Reuse Proposal Picker):** Large proposal histories (200+) load incrementally, a zero-match search shows a no-match message with the input still editable, and a failed copy clears the row loading state with an inline retry error. The copy reads the source at the moment of selection. Source: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-004-reuse-proposal-picker.md` (section: Edge Cases)
- **FEAT-02.SPEC-005 (Proposal Send):** Send reports distinct outcomes for an already-sent proposal, a concurrently discarded draft, and a removed Primary contact, none creating a partial Sent state. When two sessions send the same Draft at once, only the first to pass the Draft-status check transitions it. Source: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-005-proposal-send.md` (section: Edge Cases)
- **FEAT-02.SPEC-006 (Proposal Edit-Before-Acceptance Void & Resend):** If the client accepts between the Save & Resend tap and the status check, the automation reports the already-accepted outcome and the acceptance stands. Concurrent runs are single-flight (only the first void-and-resend proceeds), and the new version goes to the client's current Primary contact at run time. Source: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-006-proposal-edit-before-acceptance-void-resend.md` (section: Edge Cases)
- **FEAT-02.SPEC-007 (Proposal Resend):** Resend after the proposal is voided or accepted reports a state-mismatch outcome, and a second resend within the cooldown window is blocked as rate-limited with no second email. The rate-limit check doubles as the in-flight guard for concurrent sessions. Source: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-007-proposal-resend.md` (section: Edge Cases)
- **FEAT-02.SPEC-008 (Create Draft From Copy):** A source in a different currency copies its price as-is and shows the target project's currency for adjustment. A concurrently created active proposal on the target project blocks the copy, so only one Draft is created even when two copies race. Source: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-008-create-draft-from-copy.md` (section: Edge Cases)
- **FEAT-02.SPEC-009 (Discard Draft):** A proposal sent from another session just before discard is never deleted (state-mismatch outcome), and concurrent discards resolve first-wins. Discarding a draft pre-filled from a copy leaves the source proposal unaffected. Source: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-009-discard-draft.md` (section: Edge Cases)
- **FEAT-02.SPEC-010 (Proposal Validation & Business Rules):** Price of exactly zero fails the positive-amount rule, a scope description of exactly 10,000 characters passes and 10,001 fails, and the one-active-proposal cap compares only against the current active proposal. Send eligibility re-reads the client's current Primary contact at the moment of send. Source: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-010-proposal-validation-business-rules.md` (section: Edge Cases)
- **FEAT-02.SPEC-011 (Proposal Sent/Resent Email):** A pending retry for the original send is superseded when the proposal is voided and re-sent, and delivery goes to the Primary contact's current email at actual delivery time. If all retries fail, the delivery warning surfaces on the project for the freelancer (XBR-30), who can use Resend (FEAT-02.SPEC-007). Source: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-011-proposal-sent-resent-email.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-02.SPEC-001** (Proposal Draft Editor) as specified: Nadia writes scope, price, and currency for a project's proposal from a blank form, from a copied earlier proposal, or by revising a Sent-but-unaccepted proposal. Full spec: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-001-proposal-draft-editor.md`
- **FR-002**: The system MUST implement **FEAT-02.SPEC-002** (Proposal Preview) as specified: Nadia reviews a branded, client-facing rendering of the current draft's scope, price, and payment schedule before sending it to the client. Full spec: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-002-proposal-preview.md`
- **FR-003**: The system MUST implement **FEAT-02.SPEC-003** (Proposal Detail) as specified: Nadia (and, read-only, Dana) views a project's current proposal -- its status, history, and the actions available for that status -- and Dana's support view is limited to status only. Full spec: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-003-proposal-detail.md`
- **FR-004**: The system MUST implement **FEAT-02.SPEC-004** (Reuse Proposal Picker) as specified: Nadia browses her earlier proposals across all clients and projects and selects one to start a new draft as a copy. Full spec: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-004-reuse-proposal-picker.md`
- **FR-005**: The system MUST implement **FEAT-02.SPEC-005** (Proposal Send) as specified: Validates and transitions a Draft proposal to Sent, recording the send timestamp and locking the payment schedule reference so the client always sees the schedule as it stood at send. Full spec: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-005-proposal-send.md`
- **FR-006**: The system MUST implement **FEAT-02.SPEC-006** (Proposal Edit-Before-Acceptance Void & Resend) as specified: When Nadia edits a Sent-but-unaccepted proposal, voids the prior version and creates and sends the new one, so the client is never shown an outdated price (XBR-06). Full spec: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-006-proposal-edit-before-acceptance-void-resend.md`
- **FR-007**: The system MUST implement **FEAT-02.SPEC-007** (Proposal Resend) as specified: Re-sends the link for an already-Sent proposal without creating a new version, for the case where Owen simply cannot find the original email. Full spec: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-007-proposal-resend.md`
- **FR-008**: The system MUST implement **FEAT-02.SPEC-008** (Create Draft From Copy) as specified: Creates a new Draft proposal for the target project, pre-filled from a selected earlier proposal's scope, price, and currency. Full spec: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-008-create-draft-from-copy.md`
- **FR-009**: The system MUST implement **FEAT-02.SPEC-009** (Discard Draft) as specified: Permanently removes an unsent Draft proposal that Nadia no longer wants to keep. Full spec: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-009-discard-draft.md`
- **FR-010**: The system MUST implement **FEAT-02.SPEC-010** (Proposal Validation & Business Rules) as specified: Governs required fields, price positivity and currency, the one-active-proposal-per-project cap, send eligibility (XBR-07), immutability after send and after acceptance (XBR-04), and the accept-vs-void contention resolution for the Proposal entity. Full spec: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-010-proposal-validation-business-rules.md`
- **FR-011**: The system MUST implement **FEAT-02.SPEC-011** (Proposal Sent/Resent Email) as specified: Emails the client's Primary Contact the proposal link whenever a proposal is sent, resent, or re-sent after an edit, so the client can review and act on it without a "PDF in email" back-and-forth. Full spec: `docs/blueprint/specifications/FEAT-02-proposal-creation-sending/FEAT-02.SPEC-011-proposal-sent-resent-email.md`

### Key Entities

- Proposal (create, update, send)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Nadia can draft and send a proposal in under 10 minutes for a typical project scope (metric: Proposal Send Speed). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Drafting, sending, editing before acceptance, resending, copying and discarding a proposal are each observable as distinct signals (proposal_drafted, proposal_sent, proposal_edited_before_acceptance, proposal_resent, proposal_created_from_copy, proposal_draft_discarded). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-03**: Freelancers and client contacts have reliable, actively monitored email access, which is the product's sole client-facing channel. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-11**: A recorded, timestamped Accept is assumed enough evidence for most scope disputes. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-15**: Records are append-only and immutable once created, which is why an edited sent proposal is voided and re-sent. Full register: `docs/blueprint/features/assumptions-constraints.md`
