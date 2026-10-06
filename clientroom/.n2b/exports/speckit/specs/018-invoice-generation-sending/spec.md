# Feature Specification: Invoice Generation & Sending

**Blueprint feature:** FEAT-09
**Priority tier:** Core
**Build order:** 018 of 33
**Depends on:** FEAT-01, FEAT-03, FEAT-08, FEAT-15, FEAT-21, FEAT-32
**Blueprint source:** `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Invoice List (Priority: P1)

Lets Nadia browse a project's invoices and lets Owen browse his own company's invoices, each scoped to what their role can see, with a clear "no invoices issued" state before the first one exists.

**Acceptance Scenarios:**

**FEAT-09.SPEC-001-AC-01:** Given Nadia opens the invoices area of a project with three invoices, when the screen loads, then all three appear as rows ordered newest-issued first.

**FEAT-09.SPEC-001-AC-02:** Given a project has never had an invoice, when Nadia opens its invoices area, then she sees "No invoices issued" with a "New Invoice" action.

**FEAT-09.SPEC-001-AC-03:** Given Owen opens "Invoices" from his portal home and his company has invoices across two different projects, when the screen loads, then both projects' invoices appear in one list, each row labeled with its project.

**FEAT-09.SPEC-001-AC-04:** Given Priya is signed into her portal, when she looks for an "Invoices" entry, then none is shown anywhere in her navigation.

**FEAT-09.SPEC-001-AC-05:** Given Dana is inside a logged support session, when she opens the invoice list, then she sees every invoice read-only with no "New Invoice" action.

**FEAT-09.SPEC-001-AC-06:** Given Nadia taps an invoice row, when the tap registers, then she lands on FEAT-09.SPEC-002 for that invoice.

**FEAT-09.SPEC-001-AC-07:** Given Nadia taps "New Invoice", when the tap registers, then she lands on FEAT-09.SPEC-003.

**FEAT-09.SPEC-001-AC-08:** Given the list fails to load, when the failure occurs, then Nadia sees "Couldn't load invoices. Check your connection and try again." with a Retry action.

**FEAT-09.SPEC-001-AC-09:** Given Owen loses connectivity while viewing his invoice list, when the loss occurs, then the last-loaded rows remain visible under a "You're offline" banner and refresh silently once connectivity returns.

**FEAT-09.SPEC-001-AC-10:** Given a new invoice is generated automatically while Nadia has the list open, when generation completes, then the new row appears in the list without a manual reload.

**FEAT-09.SPEC-001-AC-11:** Given Owen's session expires while he is on this screen, when he attempts to interact with it, then he sees FEAT-05.SPEC-002's expired-link explanation with a one-tap way to request a fresh link.

### User Story 2 - Invoice Detail (Priority: P1)

Shows one invoice's amount, status, due date, reminder history, and pay-link/download controls, with what each side may do differentiated by role.

**Acceptance Scenarios:**

**FEAT-09.SPEC-002-AC-01:** Given Nadia opens an invoice she issued, when the screen loads, then she sees its amount, currency, tax line, status, issue date, due date, and reminder history.

**FEAT-09.SPEC-002-AC-02:** Given Owen opens one of his company's invoices, when the screen loads, then he sees its content but no reminder history section.

**FEAT-09.SPEC-002-AC-03:** Given Priya attempts to open a direct link to an invoice, when the link resolves, then she is redirected to her portal home with no invoice content ever rendered.

**FEAT-09.SPEC-002-AC-04:** Given the invoice's Payment Account Connection is not yet connected, when Owen views the invoice, then the pay-link banner shows "online payment not yet available" wording and no "Pay now" action appears.

**FEAT-09.SPEC-002-AC-05:** Given Owen views an invoice with a ready, connected payment account, when he taps "Pay now", then he navigates to FEAT-10's pay flow.

**FEAT-09.SPEC-002-AC-06:** Given the invoice's status is Sent, when Nadia views the screen, then no edit control is shown anywhere, only "Correct with a credit note."

**FEAT-09.SPEC-002-AC-07:** Given Nadia taps "Correct with a credit note", when the tap registers, then she navigates to FEAT-09.SPEC-003 with this invoice pre-selected for correction.

**FEAT-09.SPEC-002-AC-08:** Given Nadia or Owen taps Download on a Sent invoice, when the copy generates, then a printable file is offered and the invoice_copy_downloaded event is emitted.

**FEAT-09.SPEC-002-AC-09:** Given Dana opens this screen inside a support session, when she looks for any action control, then none is rendered -- only read-only content.

**FEAT-09.SPEC-002-AC-10:** Given the invoice's status changes to Paid while Nadia has this screen open, when the change lands, then the status badge updates in place with no reload required.

**FEAT-09.SPEC-002-AC-11:** Given two of Nadia's sessions both view an invoice already corrected in one of them, when the second session attempts "Correct with a credit note" on the now-stale view, then it is rejected with a refresh showing the existing correction rather than opening a second correction flow.

**FEAT-09.SPEC-002-AC-12:** Given Owen loses connectivity while viewing an invoice, when the loss occurs, then the last-loaded content remains visible under an offline banner and every connectivity-requiring action is disabled with "Reconnect to continue."

**FEAT-09.SPEC-002-AC-13:** Given the initial load fails, when the failure occurs, then Nadia sees "Couldn't load this invoice. Check your connection and try again." with a Retry action.

**FEAT-09.SPEC-002-AC-14:** Given Nadia views a Paid invoice, when she looks for a refund entry point, then "Record a refund" is shown and navigates to FEAT-25.

### User Story 3 - Manual Invoice & Credit Note Issuance (Priority: P1)

Lets Nadia issue an ad-hoc invoice outside the payment schedule, or a credit note correcting a previously sent invoice, with an adjustable due date.

**Acceptance Scenarios:**

**FEAT-09.SPEC-003-AC-01:** Given Nadia taps "New Invoice" from a project's invoice list, when the form opens, then it is in ad-hoc mode with an empty description and amount, and a due date pre-filled from her default payment terms.

**FEAT-09.SPEC-003-AC-02:** Given Nadia taps "Issue Credit Note" from an invoice's detail screen, when the form opens, then it is in credit-note mode with that invoice's number, total, and issue date shown read-only.

**FEAT-09.SPEC-003-AC-03:** Given Nadia enters a description and a positive amount and taps Submit in ad-hoc mode, when submission succeeds, then she lands on FEAT-09.SPEC-002 for the newly recorded invoice.

**FEAT-09.SPEC-003-AC-04:** Given Nadia leaves the amount field empty and taps Submit, when validation runs, then the amount field shows "Enter an amount greater than zero." and submission does not proceed.

**FEAT-09.SPEC-003-AC-05:** Given Nadia enters a credit amount greater than the original invoice's total, when she taps Submit, then she sees "A credit note cannot exceed the original invoice's total." and submission does not proceed.

**FEAT-09.SPEC-003-AC-06:** Given Nadia adjusts the pre-filled due date on an ad-hoc invoice, when she submits, then the invoice is recorded with her chosen due date, not the default.

**FEAT-09.SPEC-003-AC-07:** Given Nadia navigates away with unsaved input, when she taps the back control, then a confirmation dialog appears asking "You have unsaved changes. Discard?"

**FEAT-09.SPEC-003-AC-08:** Given Nadia taps Submit twice in rapid succession, when the first submission is still in flight, then the second tap has no effect.

**FEAT-09.SPEC-003-AC-09:** Given the invoice Nadia is correcting was already corrected in another session, when she taps Submit, then she sees "This invoice was already corrected. View the existing credit note." and no second credit note is recorded.

**FEAT-09.SPEC-003-AC-10:** Given a network failure occurs during submission, when the failure is detected, then Nadia sees "Couldn't submit this invoice. Check your connection and try again." with her entered data preserved.

**FEAT-09.SPEC-003-AC-11:** Given Nadia opens this screen while offline, when she attempts to submit, then Submit is disabled and a "reconnect to submit" banner is shown; the action never appears to succeed.

**FEAT-09.SPEC-003-AC-12:** Given Owen attempts to reach this screen directly, when the link resolves, then he is redirected to his portal home with no form ever rendered.

**FEAT-09.SPEC-003-AC-13:** Given Dana is inside a logged support session, when she looks for an issuance entry point, then none exists anywhere in her session.

### User Story 4 - Automatic Invoice Generation (Priority: P1)

On a deposit acceptance, milestone approval, or project completion, generates the correct invoice with no freelancer action and hands it off to be sent.

**Acceptance Scenarios:**

**FEAT-09.SPEC-004-AC-01:** Given a proposal is accepted and the Payment Schedule includes a deposit, when FEAT-03.SPEC-003 fires this automation, then a deposit invoice is generated with the correct amount, numbered sequentially, and immediately sent to Owen.

**FEAT-09.SPEC-004-AC-02:** Given a milestone is approved and its `payment_trigger` marks it to invoice on approval, when FEAT-08.SPEC-004 fires this automation, then the next invoice is generated for that milestone's price and sent.

**FEAT-09.SPEC-004-AC-03:** Given a project is marked complete and the schedule includes a `completion_amount`, when FEAT-01.SPEC-006 fires this automation, then the on-completion invoice is generated and sent.

**FEAT-09.SPEC-004-AC-04:** Given the Freelancer Account's business details are incomplete at the moment of firing, when this automation runs, then no invoice is created and Nadia sees the non-blocking warning "Your business details aren't complete yet. Add them in Settings before this invoice can be sent."

**FEAT-09.SPEC-004-AC-05:** Given the Client's billing details are incomplete at the moment of firing, when this automation runs, then no invoice is created and Nadia sees the non-blocking warning "This client's billing details are incomplete. Add a billing name and address before sending an invoice."

**FEAT-09.SPEC-004-AC-06:** Given Nadia has no connected Payment Account Connection at generation time, when the invoice is generated, then it still issues and sends, carrying the "online payment not yet available" wording from FEAT-09.SPEC-009.

**FEAT-09.SPEC-004-AC-07:** Given the invoice is created successfully but the send hand-off to FEAT-09.SPEC-010 fails, when the failure occurs, then the invoice remains visible in Nadia's and Owen's lists, and the send is retried automatically without creating a second invoice record.

**FEAT-09.SPEC-004-AC-08:** Given a milestone approval and a project completion fire at effectively the same time for the same project, when both invocations run, then each produces its own separately numbered invoice with no collision.

**FEAT-09.SPEC-004-AC-09:** Given Nadia is offline when a trigger fires, when the trigger fires, then the invoice still generates and sends with no freelancer action required.

**FEAT-09.SPEC-004-AC-10:** Given the project's currency and tax line were never configured, when a trigger fires, then generation is blocked with the warning "This project's currency isn't set yet. Set it before the first invoice."

**FEAT-09.SPEC-004-AC-11:** Given the deposit invoice generates successfully, when the invoice_generated event is emitted, then it carries the triggering event type and the invoice reference.

**FEAT-09.SPEC-004-AC-12:** Given generation is blocked for missing billing details, when the invoice_generation_blocked event is emitted, then it carries the specific reason (missing_freelancer_details / missing_client_details / missing_currency_tax_config).

**FEAT-09.SPEC-004-AC-13:** Given this automation runs for a second trigger while a first trigger's run on the same project is still completing its send hand-off, when both runs execute, then neither is blocked by the other, since each reads only its own triggering data and shares no state beyond the sequential invoice-numbering counter.

### User Story 5 - Manual Invoice & Credit Note Recording (Priority: P1)

Validates and writes an ad-hoc invoice or a credit note submitted from FEAT-09.SPEC-003, marking the original invoice Corrected when a credit note supersedes it.

**Acceptance Scenarios:**

**FEAT-09.SPEC-005-AC-01:** Given Nadia submits a valid ad-hoc invoice with billing details complete, when this automation runs, then a new invoice is created, numbered sequentially, and sent to Owen.

**FEAT-09.SPEC-005-AC-02:** Given Nadia submits a credit note against a Sent invoice not already Corrected, when this automation runs, then a new credit-note invoice is created and the original invoice's status is set to Corrected.

**FEAT-09.SPEC-005-AC-03:** Given the Freelancer Account's business details are incomplete, when Nadia submits an ad-hoc invoice, then recording is blocked with "Your business details aren't complete yet. Add them in Settings before this invoice can be sent."

**FEAT-09.SPEC-005-AC-04:** Given the credit amount exceeds the original invoice's total, when Nadia submits the credit note, then it is rejected with "A credit note cannot exceed the original invoice's total."

**FEAT-09.SPEC-005-AC-05:** Given the credit amount exactly equals the original invoice's total, when Nadia submits it, then it passes validation and the original is marked Corrected.

**FEAT-09.SPEC-005-AC-06:** Given the original invoice was already corrected by another session before this write runs, when this automation's re-check executes, then it returns "already corrected" and no second credit note is created.

**FEAT-09.SPEC-005-AC-07:** Given two credit notes against the same invoice are submitted at effectively the same time, when both invocations reach the write-time re-check, then exactly one succeeds and the other returns "already corrected."

**FEAT-09.SPEC-005-AC-08:** Given recording succeeds, when the invoice or credit note is ready, then this automation hands off to FEAT-09.SPEC-010 for sending.

**FEAT-09.SPEC-005-AC-09:** Given the send hand-off fails after recording succeeds, when the failure occurs, then the recorded invoice or credit note is unaffected and the send is retried without duplicating the record.

**FEAT-09.SPEC-005-AC-10:** Given Nadia submits a credit note against an invoice that has already been paid, when recording succeeds, then the original is marked Corrected alongside its existing Paid history, with no automatic refund action taken.

**FEAT-09.SPEC-005-AC-11:** Given the ad-hoc invoice is recorded successfully, when the invoice_manually_issued event is emitted, then it carries the project reference and amount.

**FEAT-09.SPEC-005-AC-12:** Given a manual submission is blocked for any reason, when the manual_recording_blocked event is emitted, then it carries the specific path and reason.

### User Story 6 - Invoice Access & Role Authorization Rules (Priority: P1)

Governs who may view, issue, follow the pay link on, or download an invoice, including Priya's total exclusion and Dana's read-only support-session visibility.

**Acceptance Scenarios:**

**FEAT-09.SPEC-006-AC-01:** Given Nadia is signed in, when she opens any of her projects' invoice lists, then she sees every invoice regardless of status.

**FEAT-09.SPEC-006-AC-02:** Given Owen is signed into his portal, when he opens "Invoices", then he sees only his own company's invoices across all of its projects.

**FEAT-09.SPEC-006-AC-03:** Given Priya is signed into her portal, when she looks for any invoice-related entry point, then none exists anywhere in her navigation.

**FEAT-09.SPEC-006-AC-04:** Given Priya is given a direct link to a specific invoice, when the link resolves, then she is redirected to her portal home with no invoice content rendered.

**FEAT-09.SPEC-006-AC-05:** Given Dana opens a Support Access Session for a freelancer's account, when she navigates to that account's invoices, then she sees full content read-only with no action controls.

**FEAT-09.SPEC-006-AC-06:** Given Dana has no open Support Access Session, when she attempts to reach any invoice screen, then none is reachable.

**FEAT-09.SPEC-006-AC-07:** Given Nadia opens FEAT-09.SPEC-003, when the screen loads, then she can issue an ad-hoc invoice or credit note.

**FEAT-09.SPEC-006-AC-08:** Given Owen attempts to reach FEAT-09.SPEC-003 by a direct link, when the link resolves, then he is redirected to his portal home.

**FEAT-09.SPEC-006-AC-09:** Given Dana is inside an open support session, when she attempts to reach FEAT-09.SPEC-003 by any means, then it is never rendered.

**FEAT-09.SPEC-006-AC-10:** Given Owen's invoice has a ready, connected payment account, when he views its detail, then he can follow the pay link.

**FEAT-09.SPEC-006-AC-11:** Given Owen's invoice does not yet have a ready payment account, when he views its detail, then no active pay-link action is offered to him.

**FEAT-09.SPEC-006-AC-12:** Given Nadia or Owen views an invoice each is entitled to see, when they tap Download, then the copy is produced.

**FEAT-09.SPEC-006-AC-13:** Given Dana is inside an open support session, when she looks for a Download control on any invoice, then none is rendered.

**FEAT-09.SPEC-006-AC-14:** Given a Support Access Session closes while Dana has an invoice screen open, when she attempts any further interaction, then the screen is no longer reachable.

**FEAT-09.SPEC-006-AC-15:** Given an invoice has been corrected (status: Corrected), when Owen or Nadia attempts to download the original, then the download still succeeds, since the original record is never hidden by correction.

### User Story 7 - Invoice Content, Numbering, Amount & Due-Date Rules (Priority: P1)

Governs sequential numbering, required business/billing details, amount-matches-trigger validation, and default-payment-terms due-date derivation for every invoice, whether automatically generated or manually issued.

**Acceptance Scenarios:**

**FEAT-09.SPEC-007-AC-01:** Given a freelancer has issued four prior invoices numbered 1-4, when a fifth is created by either path, then it is numbered 5, with no gap and no reuse.

**FEAT-09.SPEC-007-AC-02:** Given a milestone approval triggers an invoice for a price of $500, when FEAT-09.SPEC-004 creates it, then the invoice's `amount` is exactly $500 plus the project's configured tax line.

**FEAT-09.SPEC-007-AC-03:** Given Nadia enters $0 as the amount on the manual-issuance form, when she attempts to submit, then she sees "Enter an amount greater than zero." and submission is blocked.

**FEAT-09.SPEC-007-AC-04:** Given a project has no configured currency, when a trigger attempts to generate its first invoice, then generation is blocked with "This project's currency isn't set yet. Set it before the first invoice."

**FEAT-09.SPEC-007-AC-05:** Given the Freelancer Account's business details are incomplete, when any invoice for that freelancer is created, then sending is blocked with "Your business details aren't complete yet. Add them in Settings before this invoice can be sent."

**FEAT-09.SPEC-007-AC-06:** Given the Client's billing details are incomplete, when any invoice for that client is created, then sending is blocked with "This client's billing details are incomplete. Add a billing name and address before sending an invoice."

**FEAT-09.SPEC-007-AC-07:** Given both business and billing details are complete, when an invoice is created, then it proceeds to `issue_date` and `due_date` assignment with no block.

**FEAT-09.SPEC-007-AC-08:** Given Nadia's default payment terms is "due within 14 days" and an invoice is generated automatically, when it is created, then `due_date` is set to 14 days after `issue_date` with no freelancer interaction.

**FEAT-09.SPEC-007-AC-09:** Given Nadia is issuing an ad-hoc invoice, when the form pre-fills the due date from her default terms, then she can adjust it to any other date before submitting, and the adjusted date is what gets recorded.

**FEAT-09.SPEC-007-AC-10:** Given Nadia clears the pre-filled due date on the manual form without entering another, when she attempts to submit, then she sees "Choose a due date." and submission is blocked.

**FEAT-09.SPEC-007-AC-11:** Given a credit note's entered amount exceeds the original invoice's total, when Nadia submits it, then she sees "A credit note cannot exceed the original invoice's total." and it is not recorded.

**FEAT-09.SPEC-007-AC-12:** Given a credit note's entered amount exactly equals the original invoice's total, when Nadia submits it, then it is accepted.

**FEAT-09.SPEC-007-AC-13:** Given a project's tax line is configured as none, when an invoice is created, then its `total` equals `amount` exactly, with no tax line shown.

**FEAT-09.SPEC-007-AC-14:** Given the client's tax_id is blank, when billing completeness is evaluated, then it never factors into the block.

**FEAT-09.SPEC-007-AC-15:** Given Nadia's default_payment_terms changes after an invoice was already created under the old terms, when she views that invoice, then its `due_date` remains exactly as originally derived, unaffected by the later change.

**FEAT-09.SPEC-007-AC-16:** Given Nadia enters a due date on the manual form that is the same day as the issue date, when she submits, then it is accepted as valid.

### User Story 8 - Invoice Immutability & Correction Rules (Priority: P1)

Enforces that a sent invoice is never silently edited and that a correction is always a visible credit note or a new invoice.

**Acceptance Scenarios:**

**FEAT-09.SPEC-008-AC-01:** Given an invoice's status is Sent, when Nadia views its detail, then no edit control is shown for any field.

**FEAT-09.SPEC-008-AC-02:** Given an invoice's status is Sent, when Nadia wants to correct it, then the only available action is "Correct with a credit note."

**FEAT-09.SPEC-008-AC-03:** Given Nadia issues a credit note against a Sent invoice, when it is recorded, then a new, separate Invoice record is created and the original's status becomes Corrected in the same step.

**FEAT-09.SPEC-008-AC-04:** Given an invoice has been marked Corrected, when Nadia or Owen views its original content, then every original field (amount, dates, business/billing details, invoice number) remains exactly as first sent.

**FEAT-09.SPEC-008-AC-05:** Given two sessions attempt to correct the same invoice at effectively the same time, when both reach the write, then exactly one succeeds and the other is told "This invoice was already corrected. View the existing credit note."

**FEAT-09.SPEC-008-AC-06:** Given a credit note itself reaches Sent status, when Nadia later wants to correct it, then the same immutability and correction rule applies -- a new record links back to it.

**FEAT-09.SPEC-008-AC-07:** Given an invoice was created automatically (FEAT-09.SPEC-004), when it reaches Sent status, then it becomes immutable on exactly the same terms as a manually issued invoice.

**FEAT-09.SPEC-008-AC-08:** Given a Paid invoice is later corrected by a credit note, when Nadia views its detail, then both its Paid payment history and its Corrected status are shown together, unaltered by each other.

**FEAT-09.SPEC-008-AC-09:** Given the original invoice is marked Corrected, when Nadia or Owen looks for its credit note, then a link to the linked credit note is present on the original's detail view.

**FEAT-09.SPEC-008-AC-10:** Given a correction chain exists (an invoice corrected by a credit note, which is itself later corrected), when any record in the chain is viewed, then its link back to its immediate predecessor is traceable.

**FEAT-09.SPEC-008-AC-11:** Given the credit note's own send fails after the original was already marked Corrected, when the failure occurs, then the Corrected status and the credit note record are unaffected, and only the send is retried.

**FEAT-09.SPEC-008-AC-12:** Given no invoice in this product is ever created without immediately reaching Sent status within the same automation run, when any invoice is inspected, then it is never observed sitting in a user-reachable Generated-only, still-editable state.

### User Story 9 - Pay-Link Availability & No-Account Fallback Rule (Priority: P1)

Governs what an invoice's pay link shows and how it is worded when Nadia has no connected, ready payment account, or when a previously working connection later needs attention or disconnects.

**Acceptance Scenarios:**

**FEAT-09.SPEC-009-AC-01:** Given Nadia has a Connected, ready payment account, when an invoice is generated, then its pay link shows "Ready to pay."

**FEAT-09.SPEC-009-AC-02:** Given Nadia has never connected a payment account, when an invoice is generated and sent, then it shows "Online payment isn't set up yet." with her direct-payment instructions, and it still sends.

**FEAT-09.SPEC-009-AC-03:** Given Nadia's connection status is Needs attention, when Owen opens an open invoice, then he sees "Online payment is temporarily unavailable. Please try again shortly, or contact [Freelancer name] directly."

**FEAT-09.SPEC-009-AC-04:** Given Nadia's connection status is Disconnected after having previously been connected, when Owen opens an open invoice, then he sees the same "temporarily unavailable" wording as the Needs-attention state.

**FEAT-09.SPEC-009-AC-05:** Given Nadia connects her account after invoices were sent with "not yet available" wording, when Owen next opens any of those invoices, then the banner shows "Ready to pay" with no further action from Nadia.

**FEAT-09.SPEC-009-AC-06:** Given the connection changes from Connected to Needs attention while Owen has an invoice detail screen open, when the change lands, then the banner updates in place to the temporarily-unavailable wording.

**FEAT-09.SPEC-009-AC-07:** Given Nadia disconnects her account while invoices remain open and unpaid, when the disconnect completes, then every open invoice's pay link switches to "temporarily unavailable" wording.

**FEAT-09.SPEC-009-AC-08:** Given an invoice has already been marked Paid, when its detail is viewed, then the pay-link availability wording from this spec is not shown.

**FEAT-09.SPEC-009-AC-09:** Given Owen opens an invoice email sent while the connection was Ready but the connection has since changed, when he follows the link to the live invoice detail page, then he sees the current, up-to-date wording rather than the state at send time.

**FEAT-09.SPEC-009-AC-10:** Given Nadia has never started the connect flow at all, when her first invoice is generated, then it is treated identically to "Not connected" with the same wording and instructions.

**FEAT-09.SPEC-009-AC-11:** Given FEAT-09.SPEC-010 composes the invoice email at the same moment FEAT-09.SPEC-002 would render the detail banner, when both render, then they show the exact same wording for the same connection state.

**FEAT-09.SPEC-009-AC-12:** Given this spec is asked for a pay-link state and the connection status is any value other than Connected, Not connected, Needs attention, or Disconnected, then no such value exists in the product's definition of Payment Account Connection, so this condition never arises.

### User Story 10 - Invoice Issued & Copy Confirmation Notification (Priority: P1)

Emails Owen the invoice with its pay link and emails Nadia a confirmation copy, whenever any invoice -- automatic, ad hoc, or credit note -- is sent.

**Acceptance Scenarios:**

**FEAT-09.SPEC-010-AC-01:** Given an invoice is generated automatically and reaches Sent status, when this notification fires, then Owen receives the invoice email and Nadia receives the confirmation copy.

**FEAT-09.SPEC-010-AC-02:** Given Nadia issues an ad-hoc invoice successfully, when it reaches Sent status, then the same two emails are sent, worded identically to the automatic path.

**FEAT-09.SPEC-010-AC-03:** Given a credit note is recorded, when it reaches Sent status, then Owen and Nadia each receive their respective emails for the credit note, distinct from the original invoice's emails.

**FEAT-09.SPEC-010-AC-04:** Given Nadia has a Connected, ready payment account, when Owen's email is composed, then {pay_link_status_block} renders the "Pay now" button.

**FEAT-09.SPEC-010-AC-05:** Given Nadia has no connected payment account, when Owen's email is composed, then {pay_link_status_block} renders the "not yet available" wording with no pay button.

**FEAT-09.SPEC-010-AC-06:** Given Priya is a Reviewer contact at the same client company as Owen, when any invoice is sent, then Priya never receives this notification.

**FEAT-09.SPEC-010-AC-07:** Given Owen's email bounces on the first delivery attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the project for Nadia.

**FEAT-09.SPEC-010-AC-08:** Given Nadia's own confirmation copy fails delivery after all retries, when the final failure occurs, then no separate project-level warning is generated, since the invoice remains fully visible to her in-product.

**FEAT-09.SPEC-010-AC-09:** Given Owen taps "View invoice" from the email, when the tap registers, then he lands on FEAT-09.SPEC-002 for that exact invoice.

**FEAT-09.SPEC-010-AC-10:** Given an invoice is corrected after its original email was sent, when Owen later opens that original email's link, then he sees the invoice's current, live Corrected status rather than the email's original static text.

**FEAT-09.SPEC-010-AC-11:** Given a send hand-off is retried after a transient failure, when the retry succeeds, then exactly one pair of emails is ultimately delivered -- never two.

**FEAT-09.SPEC-010-AC-12:** Given the invoice_email_delivered event is emitted for Owen, when it fires, then it carries the recipient and the triggering event type.

**FEAT-09.SPEC-010-AC-13:** Given no quiet-hours window applies to this notification, when an invoice is generated at any hour, then both emails send immediately with no hold.

### Edge Cases

- **FEAT-09.SPEC-001 (Invoice List):** The screen has no filter of its own, so every invoice in scope is always shown, a newly generated invoice appears via silent refresh, and a contact's company-wide scope lists invoices from all its projects with a project label each. Archiving a project does not remove or hide its invoice history. Source: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-001-invoice-list.md` (section: Edge Cases)
- **FEAT-09.SPEC-002 (Invoice Detail):** Status changes while the screen is open re-render in place since it performs no writes, a contact on an invoice with no connected payment account sees the online-payment-not-yet-available banner with no Pay now action, and correcting an already-corrected invoice is rejected-with-refresh. Downloading while offline is disabled with a reconnect note and never yields a partial copy. Source: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-002-invoice-detail.md` (section: Edge Cases)
- **FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance):** Unsaved changes trigger the discard confirmation, double taps on Submit are ignored, and a credit amount exceeding the original total is blocked at submit with a field-level error. An invoice already corrected from another session is rejected-with-refresh. Source: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-003-manual-invoice-credit-note-issuance.md` (section: Edge Cases)
- **FEAT-09.SPEC-004 (Automatic Invoice Generation):** Generation is blocked with a missing currency/tax configuration outcome (XBR-17) when the project was never configured, proceeds in the background while the freelancer is offline, and still sends when no payment account is connected (using the FEAT-09.SPEC-009 wording). Concurrent triggers for one project each produce their own independent generation. Source: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-004-automatic-invoice-generation.md` (section: Edge Cases)
- **FEAT-09.SPEC-005 (Manual Invoice & Credit Note Recording):** Of two simultaneous credit notes against one invoice only one wins the atomic update, an in-flight submission is guarded by the disabled Submit control, and an ad hoc invoice for an unconfigured project is blocked like the automatic path (XBR-17). A credit amount exactly equal to the original total is accepted and marks the original Corrected. Source: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-005-manual-invoice-credit-note-recording.md` (section: Edge Cases)
- **FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules):** A Reviewer given a direct invoice link is redirected to portal home with no invoice content rendered, a closed support session makes the screen unreachable immediately, and downloading a copy of an invoice remains available after it has been corrected. A mid-session role change is not defined by the product (role changes are freelancer-only per FEAT-18). Source: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-006-invoice-access-role-authorization-rules.md` (section: Edge Cases)
- **FEAT-09.SPEC-007 (Invoice Content, Numbering, Amount & Due-Date Rules):** Simultaneous invoices for one freelancer draw distinct numbers from the same sequential source, and the billing-completeness check reads current state at creation time. due_date is a one-time snapshot of the payment terms and is not changed by later term changes, and a manually entered due date earlier than the issue date is accepted. Source: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-007-invoice-content-numbering-amount-due-date-rules.md` (section: Edge Cases)
- **FEAT-09.SPEC-008 (Invoice Immutability & Correction Rules):** A Generated invoice that has not sent is not correctable in practice since both creation paths set Sent in one step, a credit note can itself be corrected, and two sessions correcting the same original resolve first-to-complete-wins. A Sent-then-Paid invoice can still receive a credit note, leaving its payment history untouched beside the Corrected status. Source: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-008-invoice-immutability-correction-rules.md` (section: Edge Cases)
- **FEAT-09.SPEC-009 (Pay-Link Availability & No-Account Fallback Rule):** Open invoices re-derive their pay-link banner to Ready at the next render once the payment account is connected. A move to Needs attention or a full disconnect switches the banner to the temporarily-unavailable wording (XBR-19), and the wording is never shown on an invoice already Paid. Source: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-009-pay-link-availability-no-account-fallback-rule.md` (section: Edge Cases)
- **FEAT-09.SPEC-010 (Invoice Issued & Copy Confirmation Notification):** The original email is never altered after sending even if the invoice is later corrected, and the pay-link status block reflects state at send time while the screen shows current state. A bounced contact address is retried and surfaced as a delivery warning on the project, and a credit note triggers its own independent send and confirmation copy. Source: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-010-invoice-issued-copy-confirmation-notification.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-09.SPEC-001** (Invoice List) as specified: Lets Nadia browse a project's invoices and lets Owen browse his own company's invoices, each scoped to what their role can see, with a clear "no invoices issued" state before the first one exists. Full spec: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-001-invoice-list.md`
- **FR-002**: The system MUST implement **FEAT-09.SPEC-002** (Invoice Detail) as specified: Shows one invoice's amount, status, due date, reminder history, and pay-link/download controls, with what each side may do differentiated by role. Full spec: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-002-invoice-detail.md`
- **FR-003**: The system MUST implement **FEAT-09.SPEC-003** (Manual Invoice & Credit Note Issuance) as specified: Lets Nadia issue an ad-hoc invoice outside the payment schedule, or a credit note correcting a previously sent invoice, with an adjustable due date. Full spec: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-003-manual-invoice-credit-note-issuance.md`
- **FR-004**: The system MUST implement **FEAT-09.SPEC-004** (Automatic Invoice Generation) as specified: On a deposit acceptance, milestone approval, or project completion, generates the correct invoice with no freelancer action and hands it off to be sent. Full spec: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-004-automatic-invoice-generation.md`
- **FR-005**: The system MUST implement **FEAT-09.SPEC-005** (Manual Invoice & Credit Note Recording) as specified: Validates and writes an ad-hoc invoice or a credit note submitted from FEAT-09.SPEC-003, marking the original invoice Corrected when a credit note supersedes it. Full spec: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-005-manual-invoice-credit-note-recording.md`
- **FR-006**: The system MUST implement **FEAT-09.SPEC-006** (Invoice Access & Role Authorization Rules) as specified: Governs who may view, issue, follow the pay link on, or download an invoice, including Priya's total exclusion and Dana's read-only support-session visibility. Full spec: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-006-invoice-access-role-authorization-rules.md`
- **FR-007**: The system MUST implement **FEAT-09.SPEC-007** (Invoice Content, Numbering, Amount & Due-Date Rules) as specified: Governs sequential numbering, required business/billing details, amount-matches-trigger validation, and default-payment-terms due-date derivation for every invoice, whether automatically generated or manually issued. Full spec: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-007-invoice-content-numbering-amount-due-date-rules.md`
- **FR-008**: The system MUST implement **FEAT-09.SPEC-008** (Invoice Immutability & Correction Rules) as specified: Enforces that a sent invoice is never silently edited and that a correction is always a visible credit note or a new invoice. Full spec: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-008-invoice-immutability-correction-rules.md`
- **FR-009**: The system MUST implement **FEAT-09.SPEC-009** (Pay-Link Availability & No-Account Fallback Rule) as specified: Governs what an invoice's pay link shows and how it is worded when Nadia has no connected, ready payment account, or when a previously working connection later needs attention or disconnects. Full spec: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-009-pay-link-availability-no-account-fallback-rule.md`
- **FR-010**: The system MUST implement **FEAT-09.SPEC-010** (Invoice Issued & Copy Confirmation Notification) as specified: Emails Owen the invoice with its pay link and emails Nadia a confirmation copy, whenever any invoice -- automatic, ad hoc, or credit note -- is sent. Full spec: `docs/blueprint/specifications/FEAT-09-invoice-generation-sending/FEAT-09.SPEC-010-invoice-issued-copy-confirmation-notification.md`

### Key Entities

- Invoice (create, send)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: 100% of automatically generated invoices match their triggering milestone, deposit, or completion amount, with no manual correction needed (metric: Invoice Auto-Generation Accuracy). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Invoice generation, sending, manual issuance, correction and copy downloads are each observable as distinct signals (invoice_generated, invoice_sent, invoice_manually_issued, invoice_correction_issued, invoice_copy_downloaded). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-24**: Invoices carry the content commonly required of a valid invoice (sequential number, both parties' business details, issue and due dates, tax line). Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-25**: Sent invoices are timestamped at creation and never silently altered afterward. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-15**: Records are append-only and immutable, so corrections are issued as credit notes. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-08**: Freelancers are assumed to prefer fixed, sensible behavior such as automatic invoicing on approval. Full register: `docs/blueprint/features/assumptions-constraints.md`
