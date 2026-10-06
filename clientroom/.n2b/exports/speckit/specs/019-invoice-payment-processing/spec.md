# Feature Specification: Invoice Payment Processing

**Blueprint feature:** FEAT-10
**Priority tier:** Core
**Build order:** 019 of 33
**Depends on:** FEAT-09, FEAT-32
**Blueprint source:** `docs/blueprint/specifications/FEAT-10-invoice-payment-processing/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Pay Invoice Screen (Priority: P1)

Owen views a sent invoice and pays it by card or bank transfer, sees its pending, paid, or declined status, and retries a failed payment immediately.

**Acceptance Scenarios:**

**FEAT-10.SPEC-001-AC-01:** Given Owen opens a Sent invoice's pay link, when the screen loads, then he sees the invoice total, due date, and an active Pay button for the currently available payment methods.

**FEAT-10.SPEC-001-AC-02:** Given Owen selects "Card" and taps Pay, when the payment succeeds, then the status line updates to "Paid on {paid_at_formatted}" and no Pay control remains.

**FEAT-10.SPEC-001-AC-03:** Given Owen selects "Bank transfer" and taps Pay, when the request is submitted, then the status line shows "Payment pending -- waiting on your bank transfer to confirm" and no Pay control remains until the transfer resolves.

**FEAT-10.SPEC-001-AC-04:** Given Owen's card payment is declined, when the decline is reported, then he sees "Payment declined: {failure_reason}" with a "Try again" button in place of Pay.

**FEAT-10.SPEC-001-AC-05:** Given Owen sees the declined state, when he taps "Try again" and the retried payment succeeds, then the status line updates to Paid, with no separate support step required.

**FEAT-10.SPEC-001-AC-06:** Given the freelancer's Payment Account Connection status is Needs attention, when Owen opens this invoice's pay link, then he sees "Online payment is temporarily unavailable. Please contact {freelancer_first_name} to arrange payment another way." instead of a working form.

**FEAT-10.SPEC-001-AC-07:** Given the invoice was already paid in a different session, when Owen taps Pay against his stale view, then the payment is refused and the screen refreshes to show "This invoice was already paid on {paid_at_formatted}."

**FEAT-10.SPEC-001-AC-08:** Given Owen taps Pay, when he taps it again before the first submission resolves, then the second tap has no effect and only one Payment record is created.

**FEAT-10.SPEC-001-AC-09:** Given Owen loses connectivity while viewing this screen, when he attempts to tap Pay, then the button is disabled and the banner "You're offline. Reconnect to pay this invoice." is shown.

**FEAT-10.SPEC-001-AC-10:** Given Priya (Client Reviewer Contact) attempts to open this invoice's pay link, when the screen is requested, then she sees "You don't have access to invoices for this account. Ask {client_name}'s primary contact to handle payment." instead of the invoice.

**FEAT-10.SPEC-001-AC-11:** Given an unauthenticated visitor opens a pay link, when the screen is requested, then they are redirected to sign-in with "Sign in to view and pay this invoice." and no invoice content is shown first.

**FEAT-10.SPEC-001-AC-12:** Given Owen's sign-in link has expired, when he opens it, then he sees "This link has expired. Request a fresh one to continue." with a one-tap way to request a new link.

**FEAT-10.SPEC-001-AC-13:** Given a bank transfer Owen submitted is later reported failed, when the failure is applied, then the screen returns to the Payable state with "Your bank transfer could not be completed. You can try again."

**FEAT-10.SPEC-001-AC-14:** Given Nadia records an off-platform payment on this invoice while Owen has this screen open, when Owen refreshes or reopens the screen, then it shows "Paid (recorded by freelancer)" with no Pay control.

**FEAT-10.SPEC-001-AC-15:** Given Owen is viewing this screen, when he taps "Download a copy," then he navigates to the printable invoice copy owned by FEAT-09.

**FEAT-10.SPEC-001-AC-16:** Given the status badge or status line updates while Owen has this screen open (for example a bank transfer confirming), when the update is applied, then the change is announced to assistive technology without requiring a manual refresh.

### User Story 2 - Record Off-Platform Payment Screen (Priority: P1)

Nadia records that an invoice was paid outside the portal, entering the date and method, so the payment is logged and never silent.

**Acceptance Scenarios:**

**FEAT-10.SPEC-002-AC-01:** Given Nadia opens an unpaid invoice, when she chooses "Record a payment received elsewhere," then this screen shows the invoice total, a date field defaulted to today, and an unselected method field.

**FEAT-10.SPEC-002-AC-02:** Given Nadia selects a payment date and method and taps "Save record," when the save succeeds, then she sees "Payment recorded" and the invoice detail shows "Paid (recorded by freelancer)."

**FEAT-10.SPEC-002-AC-03:** Given Nadia selects a future date, when she attempts to save, then the date field shows "Payment date cannot be in the future." and the save does not proceed.

**FEAT-10.SPEC-002-AC-04:** Given Nadia leaves the method field unselected, when she taps Save, then a validation error is shown and the save does not proceed.

**FEAT-10.SPEC-002-AC-05:** Given the payment-processing capability marks this invoice Paid at the same moment Nadia taps Save, when the save is processed, then it is refused with "This invoice was already paid online. Refresh to see the current status."

**FEAT-10.SPEC-002-AC-06:** Given Nadia taps Save twice rapidly, when the second tap occurs, then it has no effect and only one Payment record is created.

**FEAT-10.SPEC-002-AC-07:** Given a network failure occurs during save, when the failure is detected, then Nadia sees "Could not save this record. Check your connection and try again." with her entered date and method preserved.

**FEAT-10.SPEC-002-AC-08:** Given Nadia loses connectivity while filling this form, when she taps Save, then the offline banner appears and the record is submitted automatically once connectivity returns.

**FEAT-10.SPEC-002-AC-09:** Given Owen (Client Primary Contact) has no path to this screen, when he views his own invoice, then no "Record a payment" action is ever shown to him.

**FEAT-10.SPEC-002-AC-10:** Given Dana (Support Operator) is inside a logged support session, when she views this invoice, then she sees that a manual record exists with its date and method, but no Save control is available to her.

**FEAT-10.SPEC-002-AC-11:** Given Nadia taps "Cancel" with fields filled in, when the tap is processed, then she returns to invoice detail and no record is created.

**FEAT-10.SPEC-002-AC-12:** Given Nadia selects "Other" as the method, when she saves successfully, then the Payment record's method is stored as "Other" with no additional free-text field required.

**FEAT-10.SPEC-002-AC-13:** Given Nadia's session expires while she has entered a date and method, when she re-authenticates, then her entered but unsaved data is restored on this screen.

### User Story 3 - Card & Bank-Transfer Payment Processing (Priority: P1)

Submits Owen's card or bank-transfer payment to the payment-processing capability and receives back its outcome -- succeeded, failed/declined, or pending confirmation -- so the invoice's status can be applied.

**Acceptance Scenarios:**

**FEAT-10.SPEC-003-AC-01:** Given Owen selects Card and taps Pay on an open invoice, when the request is submitted, then the invoice amount, currency, invoice number, and Owen's name and email are sent to the payment-processing capability, and no card number is ever received by the product.

**FEAT-10.SPEC-003-AC-02:** Given a submitted card payment, when the capability reports it succeeded, then the Payment record's status is set to Succeeded with a recorded `paid_at`.

**FEAT-10.SPEC-003-AC-03:** Given a submitted card payment, when the capability reports it declined, then the Payment record's status is set to Failed and Owen sees "Payment declined: {failure_reason}" with an immediate retry option.

**FEAT-10.SPEC-003-AC-04:** Given Owen selects Bank transfer and taps Pay, when the request is submitted, then the Payment record's status is set to Pending and Owen sees "Payment pending -- waiting on your bank transfer to confirm."

**FEAT-10.SPEC-003-AC-05:** Given a Pending bank-transfer Payment, when the capability later confirms it, then the Payment record's status is set to Succeeded with a recorded `paid_at`, and the delayed confirmation notification (FEAT-10.SPEC-007) is released.

**FEAT-10.SPEC-003-AC-06:** Given a Pending bank-transfer Payment, when the capability reports it ultimately failed, then the Payment record's status is set to Failed and both Owen and Nadia see the reverted status.

**FEAT-10.SPEC-003-AC-07:** Given the payment-processing capability is slow to respond, when 10 seconds have elapsed with no outcome, then Owen sees "Still processing -- this is taking longer than usual." below the processing Pay button.

**FEAT-10.SPEC-003-AC-08:** Given the payment-processing capability is unavailable, when Owen attempts to pay, then the Pay button is disabled with "Online payment is temporarily unavailable. Please contact {freelancer_first_name} to arrange payment another way." and the invoice is unchanged.

**FEAT-10.SPEC-003-AC-09:** Given Owen has never submitted a payment to this freelancer before, when he taps Pay for the first time, then the data-sharing disclosure notice appears with "Continue" and "Cancel," and no data leaves the product until he chooses "Continue."

**FEAT-10.SPEC-003-AC-10:** Given a card-payment-succeeded event is delivered twice for the same Payment, when the second delivery arrives, then nothing changes and no duplicate confirmation notification fires.

**FEAT-10.SPEC-003-AC-11:** Given a failed-outcome event for a superseded, earlier attempt arrives after a later attempt on the same invoice already succeeded, when the out-of-order event is processed, then the invoice's status remains Paid from the later, successful attempt.

**FEAT-10.SPEC-003-AC-12:** Given the capability goes down after Owen's request is sent but before any outcome is confirmed, when Owen returns to the screen, then no Payment record is left in an ambiguous state and he can submit again once the capability recovers.

**FEAT-10.SPEC-003-AC-13:** Given Owen's payment is submitted after the invoice's due date has passed, when the request reaches this capability, then it is processed identically to a payment submitted before the due date.

**FEAT-10.SPEC-003-AC-14:** Given the Record Off-Platform Payment Screen (FEAT-10.SPEC-002) never sends a request to this capability, when Nadia records a manual payment, then no degradation state from this spec ever applies to that screen.

**FEAT-10.SPEC-003-AC-15:** Given a payment succeeds, when the outcome is applied, then the payment_outcome_received event fires with outcome: succeeded, supporting the "Time to Payment" metric.

### User Story 4 - Payment Confirmation & Invoice Status Sync (Priority: P1)

Applies the payment-processing capability's reported outcome to the Payment and Invoice records the instant it arrives -- Paid, Payment pending, or back to Unpaid -- visible to both Owen and Nadia, and refuses a second attempt on an already-paid invoice.

**Acceptance Scenarios:**

**FEAT-10.SPEC-004-AC-01:** Given a card payment attempt in flight, when FEAT-10.SPEC-003 reports it succeeded, then Payment status is set to Succeeded with `paid_at` recorded, Invoice status is set to Paid, and FEAT-10.SPEC-007 fires immediately.

**FEAT-10.SPEC-004-AC-02:** Given a card payment attempt in flight, when FEAT-10.SPEC-003 reports it declined, then Payment status is set to Failed and Invoice status remains unchanged (Sent or Overdue).

**FEAT-10.SPEC-004-AC-03:** Given Owen submits a bank-transfer payment, when FEAT-10.SPEC-003 acknowledges the submission, then Payment status is set to Pending, Invoice status is set to Payment pending, and the reminder-pause trigger fires toward FEAT-11.

**FEAT-10.SPEC-004-AC-04:** Given a Pending bank-transfer Payment, when the capability confirms it, then Payment status is set to Succeeded with `paid_at` recorded, Invoice status is set to Paid, and FEAT-10.SPEC-007 fires.

**FEAT-10.SPEC-004-AC-05:** Given a Pending bank-transfer Payment, when the capability reports it failed, then Payment status is set to Failed, Invoice status reverts to Sent or Overdue, and the reminder-resume trigger fires toward FEAT-11.

**FEAT-10.SPEC-004-AC-06:** Given an invoice is already Paid, when a duplicate delivery of the same Succeeded event arrives, then nothing changes and FEAT-10.SPEC-007 does not fire again.

**FEAT-10.SPEC-004-AC-07:** Given an invoice already shows "Paid (recorded by freelancer)" from Nadia's manual entry, when the payment-processing capability reports a genuine Succeeded outcome for the same invoice, then a discrepancy is flagged to Nadia rather than silently discarded.

**FEAT-10.SPEC-004-AC-08:** Given a card-failed event for an earlier attempt arrives after a later attempt on the same invoice already succeeded, when the out-of-order event is processed, then the invoice remains Paid from the later, successful attempt.

**FEAT-10.SPEC-004-AC-09:** Given two events for the same invoice arrive at effectively the same time, when they are processed, then the invoice's final status reflects the attempt the payment-processing capability itself confirms as completed, not arrival order.

**FEAT-10.SPEC-004-AC-10:** Given a trigger fires for an invoice while a previous run for that same invoice is still in flight, when the second trigger is received, then it is processed only after the first run's write completes, and the two never overwrite each other's partial state.

**FEAT-10.SPEC-004-AC-11:** Given this automation fails mid-write, when the failure occurs, then no partial state is visible -- the Payment and Invoice updates and the reminder trigger either all commit together or none do.

**FEAT-10.SPEC-004-AC-12:** Given an outcome is applied to an invoice, when the write completes, then both Owen's and Nadia's views of that invoice reflect the same status without either side needing the other to act first.

**FEAT-10.SPEC-004-AC-13:** Given a card payment is declined, when the automation processes it, then the invoice is never shown as Paid -- it stays correctly unpaid.

**FEAT-10.SPEC-004-AC-14:** Given a bank-transfer payment is confirmed after being Pending, when the confirmation is applied, then the reminder schedule (already paused) is not separately re-paused, and the invoice reaches Paid exactly once.

### User Story 5 - Record Off-Platform Payment (Priority: P1)

Validates and persists Nadia's manual payment record -- full invoice amount, not future-dated -- and sets the invoice to "Paid (recorded by freelancer)."

**Acceptance Scenarios:**

**FEAT-10.SPEC-005-AC-01:** Given Nadia enters a valid date and method for an unpaid invoice, when she saves, then a Payment record is created with the invoice's full total, `status` Recorded manually, and `recorded_by` Nadia, and the Invoice status is set to Paid (recorded by freelancer).

**FEAT-10.SPEC-005-AC-02:** Given Nadia enters a future date, when she attempts to save, then no Payment record is created and FEAT-10.SPEC-002 shows "Payment date cannot be in the future."

**FEAT-10.SPEC-005-AC-03:** Given Nadia leaves the method unselected, when she attempts to save, then no Payment record is created and a required-field error is shown.

**FEAT-10.SPEC-005-AC-04:** Given the invoice was marked Paid by the payment-processing capability moments before Nadia's save commits, when the save is processed, then it is refused with "This invoice was already paid online. Refresh to see the current status." and no Payment record is created.

**FEAT-10.SPEC-005-AC-05:** Given Nadia's save succeeds, when the write completes, then the reminder-stop trigger fires toward FEAT-11 and Owen sees the new status on his next view of FEAT-10.SPEC-001.

**FEAT-10.SPEC-005-AC-06:** Given Nadia enters today's date, when she saves, then the date passes validation (the future-date rule is inclusive of today).

**FEAT-10.SPEC-005-AC-07:** Given this automation fails mid-write, when the failure occurs, then no Payment record is left orphaned against an invoice whose status was not also updated.

**FEAT-10.SPEC-005-AC-08:** Given Nadia's save and the payment-processing capability's confirmation arrive at effectively the same time, when both attempt to write, then only one status change is applied and the losing write is refused rather than silently overwritten.

**FEAT-10.SPEC-005-AC-09:** Given a second save attempt for the same invoice fires while the first is still in flight, when the second run checks the invoice's status, then it finds the first run's already-committed status and returns the refused outcome.

**FEAT-10.SPEC-005-AC-10:** Given Nadia has successfully recorded a manual payment, when she or anyone else looks for an edit or undo control on that record, then none exists -- the record is permanent.

**FEAT-10.SPEC-005-AC-11:** Given an invoice's due date has already passed when Nadia records it, when the automation processes the save, then the Overdue flag has no effect on the outcome.

**FEAT-10.SPEC-005-AC-12:** Given a manual record is saved successfully, when the write completes, then the payment_recorded_manually event fires, supporting the "Time to Payment" metric.

### User Story 6 - Payment Authorization & Validation Rules (Priority: P1)

Governs who may pay, view, or manually record a payment, the full-payment-only and manual-record limits, the reject-with-refresh concurrency behavior, and the payment-account-readiness gate.

**Acceptance Scenarios:**

**FEAT-10.SPEC-006-AC-01:** Given Owen is on his own company's invoice with a Sent status and the Payment Account Connection is Connected, when he attempts to pay, then the attempt is allowed.

**FEAT-10.SPEC-006-AC-02:** Given the invoice is already Paid, when Owen attempts to pay against his stale view, then the attempt is refused and he is shown the refreshed, current status.

**FEAT-10.SPEC-006-AC-03:** Given the Payment Account Connection status is Needs attention, when Owen attempts to pay, then the attempt is blocked with "Online payment is temporarily unavailable. Please contact {freelancer_first_name} to arrange payment another way."

**FEAT-10.SPEC-006-AC-04:** Given Owen's payment was declined, when he taps "Try again," then the same Own-only and readiness conditions are re-checked before the retry is submitted.

**FEAT-10.SPEC-006-AC-05:** Given Nadia views any of her own invoices, when she checks payment status, then she can always see it (Full access).

**FEAT-10.SPEC-006-AC-06:** Given Priya (Client Reviewer Contact) attempts to view any invoice, when the attempt is made, then it is denied with "You don't have access to invoices for this account. Ask {client_name}'s primary contact to handle payment."

**FEAT-10.SPEC-006-AC-07:** Given Dana is inside a logged support session, when she views an invoice's payment status, then she sees it read-only with no Pay or Record control anywhere on the screen.

**FEAT-10.SPEC-006-AC-08:** Given Nadia enters a valid date and method for an eligible invoice, when she records a manual payment, then it is allowed and the amount is set to the invoice's full total automatically.

**FEAT-10.SPEC-006-AC-09:** Given the invoice was already paid online before Nadia's save commits, when she attempts to record a manual payment, then it is refused with "This invoice was already paid online. Refresh to see the current status."

**FEAT-10.SPEC-006-AC-10:** Given Owen looks for a "Record a payment received elsewhere" action on his own screens, when he searches for it, then no such action exists for his role under any condition.

**FEAT-10.SPEC-006-AC-11:** Given Priya looks for any payment-related action, when she searches for it, then none is ever shown, since Invoicing & Payments is None for her role.

**FEAT-10.SPEC-006-AC-12:** Given Nadia attempts to record a manual payment with a future date, when she submits, then the amount rule is never reached because the date rule already blocks the save with "Payment date cannot be in the future."

**FEAT-10.SPEC-006-AC-13:** Given a Payment record is created by any path, when its `amount` is set, then it always equals the invoice's full `total` -- never a partial amount.

**FEAT-10.SPEC-006-AC-14:** Given Nadia enters exactly today's date for a manual record, when she saves, then the date passes validation.

**FEAT-10.SPEC-006-AC-15:** Given two submissions are made on the same invoice in two open sessions, when both reach the point of resolution, then only the first confirmed full payment is applied and the second is refused with the refreshed status.

**FEAT-10.SPEC-006-AC-16:** Given a processor-confirmed payment and a concurrent manual record both target the same invoice, when both attempt to resolve, then the processor's confirmation is authoritative and the manual attempt is refused.

**FEAT-10.SPEC-006-AC-17:** Given the Payment Account Connection changes to Needs attention between page load and Owen's tap on Pay, when he taps Pay, then the readiness gate is re-checked at that moment and the attempt is blocked.

**FEAT-10.SPEC-006-AC-18:** Given an invoice's due date has passed (Overdue), when Owen or Nadia interacts with payment on it, then the Overdue flag has no effect on any authorization or validation outcome in this spec.

**FEAT-10.SPEC-006-AC-19:** Given a client contact's role changes from Reviewer to Primary, when the change takes effect, then that contact immediately gains Pay access per the Authorization Rules above, with no lag.

**FEAT-10.SPEC-006-AC-20:** Given Dana's support session ends, when the session closes, then her View access to that invoice's payment status ends immediately with it.

**FEAT-10.SPEC-006-AC-21:** Given Nadia is viewing one of her own invoices, when she looks for a way to pay it or retry a failed payment herself, then no Pay or "Try again" control exists for her under any condition -- paying and retrying are exclusively Owen's actions.

**FEAT-10.SPEC-006-AC-22:** Given Priya (Client Reviewer Contact) attempts to pay an invoice or retry a failed payment, when the attempt is made, then it is denied with "You don't have access to invoices for this account. Ask {client_name}'s primary contact to handle payment."

**FEAT-10.SPEC-006-AC-23:** Given Dana is inside or outside a logged support session, when she looks for a way to pay an invoice or retry a failed payment, then no Pay or "Try again" control is ever rendered for her, regardless of session state.

### User Story 7 - Payment Confirmation Notification (Priority: P1)

Sends a confirmation email to Owen and Nadia the moment a card payment or a confirmed bank transfer succeeds, so both sides have the same timestamped record without checking the portal.

**Acceptance Scenarios:**

**FEAT-10.SPEC-007-AC-01:** Given a card payment succeeds, when FEAT-10.SPEC-004 applies the Succeeded outcome, then Owen receives an email with subject "Payment confirmed for invoice {invoice_number}" and Nadia receives an email with subject "{client_name} paid invoice {invoice_number}."

**FEAT-10.SPEC-007-AC-02:** Given a bank-transfer payment is Pending, when it remains unconfirmed, then no confirmation email is sent to either recipient.

**FEAT-10.SPEC-007-AC-03:** Given a Pending bank-transfer payment is confirmed by the processor, when FEAT-10.SPEC-004 applies the Succeeded outcome, then this notification fires at that moment, releasing the receipt that was held while Pending.

**FEAT-10.SPEC-007-AC-04:** Given a Pending bank-transfer payment is ultimately reported failed, when the failure is applied, then this notification never fires for that attempt.

**FEAT-10.SPEC-007-AC-05:** Given Owen opens his confirmation email, when he taps "View invoice," then he lands on FEAT-10.SPEC-001 for that invoice.

**FEAT-10.SPEC-007-AC-06:** Given Nadia opens her confirmation email, when she taps "View invoice," then she lands on that invoice's detail in her own project view.

**FEAT-10.SPEC-007-AC-07:** Given neither Owen nor Nadia has any way to opt out of this confirmation, when their respective notification preferences are checked, then no preference control exists for it and it always sends.

**FEAT-10.SPEC-007-AC-08:** Given a payment is confirmed at any hour, when this notification fires, then it sends immediately regardless of either recipient's configured quiet hours.

**FEAT-10.SPEC-007-AC-09:** Given delivery of Owen's copy fails, when the failure occurs, then it is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, and after the final failure Nadia sees a delivery warning on the project.

**FEAT-10.SPEC-007-AC-10:** Given delivery of Nadia's copy fails while Owen's copy succeeds, when this is observed, then Nadia's copy is retried independently of Owen's successful delivery.

**FEAT-10.SPEC-007-AC-11:** Given the same Succeeded event is delivered twice by the payment-processing capability, when FEAT-10.SPEC-004's already-resolved guard blocks the second application, then this notification does not fire a second time.

**FEAT-10.SPEC-007-AC-12:** Given Priya (Client Reviewer Contact) is a contact at the same client company, when a payment succeeds on an invoice for that company, then Priya receives no copy of this confirmation.

**FEAT-10.SPEC-007-AC-13:** Given Nadia records an off-platform payment manually, when the manual record is saved, then this notification does not fire, since its trigger condition is a processor-confirmed Succeeded outcome, not a manual record.

### Edge Cases

- **FEAT-10.SPEC-001 (Pay Invoice Screen):** Double taps on Pay are ignored with no second Payment record, and a stale attempt after the invoice was paid elsewhere is rejected with the refreshed status. A payment account moving to Needs attention before the tap refuses submission with the temporarily-unavailable message, and returning mid-processing re-fetches the attempt's status without duplicating it. Source: `docs/blueprint/specifications/FEAT-10-invoice-payment-processing/FEAT-10.SPEC-001-pay-invoice-screen.md` (section: Edge Cases)
- **FEAT-10.SPEC-002 (Record Off-Platform Payment Screen):** Nothing is recorded until Save, so navigating away needs no confirmation, and double taps are ignored. A save racing a processor-confirmed payment is refused with an already-paid-online message, and a network failure preserves the entered date and method with a retry banner. Source: `docs/blueprint/specifications/FEAT-10-invoice-payment-processing/FEAT-10.SPEC-002-record-off-platform-payment-screen.md` (section: Edge Cases)
- **FEAT-10.SPEC-003 (Card & Bank-Transfer Payment Processing):** A duplicate outcome event changes nothing and keeps the original paid_at, events are matched to their own payment attempt so an earlier failed attempt cannot undo a later success, and processor-confirmed status stays authoritative over a manual record (XBR-20 and XBR-22). If the capability goes down mid-submission no Payment is left ambiguous. Source: `docs/blueprint/specifications/FEAT-10-invoice-payment-processing/FEAT-10.SPEC-003-card-bank-transfer-payment-processing.md` (section: Edge Cases)
- **FEAT-10.SPEC-004 (Payment Confirmation & Invoice Status Sync):** A duplicate event is a no-op and fires no second notification, a late failed event for an earlier attempt is recorded only against that attempt and leaves the invoice Paid, and events for the same invoice are applied in sequence so concurrent outcomes never interleave. Source: `docs/blueprint/specifications/FEAT-10-invoice-payment-processing/FEAT-10.SPEC-004-payment-confirmation-invoice-status-sync.md` (section: Edge Cases)
- **FEAT-10.SPEC-005 (Record Off-Platform Payment):** When a manual record races a processor confirmation the first write to the Invoice wins, and a second quick run finds the invoice already Paid. A date of exactly today passes (the not-in-the-future rule is inclusive), and an Overdue invoice is still eligible since Overdue (FEAT-11) does not affect eligibility. Source: `docs/blueprint/specifications/FEAT-10-invoice-payment-processing/FEAT-10.SPEC-005-record-off-platform-payment.md` (section: Edge Cases)
- **FEAT-10.SPEC-006 (Payment Authorization & Validation Rules):** Two payment attempts from two tabs are each accepted but only the first to Succeed determines the invoice outcome, and the payment-account readiness gate is re-checked at submission rather than at page load. The Overdue flag has no bearing on eligibility, and a manual-record date of exactly today passes. Source: `docs/blueprint/specifications/FEAT-10-invoice-payment-processing/FEAT-10.SPEC-006-payment-authorization-validation-rules.md` (section: Edge Cases)
- **FEAT-10.SPEC-007 (Payment Confirmation Notification):** No receipt is sent while a bank transfer is Pending and one fires exactly once on Succeeded, never for a Pending transfer later reported failed (shown inline instead). Each recipient's copy is tracked and retried independently, and a duplicate Succeeded event does not trigger a second confirmation. Source: `docs/blueprint/specifications/FEAT-10-invoice-payment-processing/FEAT-10.SPEC-007-payment-confirmation-notification.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-10.SPEC-001** (Pay Invoice Screen) as specified: Owen views a sent invoice and pays it by card or bank transfer, sees its pending, paid, or declined status, and retries a failed payment immediately. Full spec: `docs/blueprint/specifications/FEAT-10-invoice-payment-processing/FEAT-10.SPEC-001-pay-invoice-screen.md`
- **FR-002**: The system MUST implement **FEAT-10.SPEC-002** (Record Off-Platform Payment Screen) as specified: Nadia records that an invoice was paid outside the portal, entering the date and method, so the payment is logged and never silent. Full spec: `docs/blueprint/specifications/FEAT-10-invoice-payment-processing/FEAT-10.SPEC-002-record-off-platform-payment-screen.md`
- **FR-003**: The system MUST implement **FEAT-10.SPEC-003** (Card & Bank-Transfer Payment Processing) as specified: Submits Owen's card or bank-transfer payment to the payment-processing capability and receives back its outcome -- succeeded, failed/declined, or pending confirmation -- so the invoice's status can be applied. Full spec: `docs/blueprint/specifications/FEAT-10-invoice-payment-processing/FEAT-10.SPEC-003-card-bank-transfer-payment-processing.md`
- **FR-004**: The system MUST implement **FEAT-10.SPEC-004** (Payment Confirmation & Invoice Status Sync) as specified: Applies the payment-processing capability's reported outcome to the Payment and Invoice records the instant it arrives -- Paid, Payment pending, or back to Unpaid -- visible to both Owen and Nadia, and refuses a second attempt on an already-paid invoice. Full spec: `docs/blueprint/specifications/FEAT-10-invoice-payment-processing/FEAT-10.SPEC-004-payment-confirmation-invoice-status-sync.md`
- **FR-005**: The system MUST implement **FEAT-10.SPEC-005** (Record Off-Platform Payment) as specified: Validates and persists Nadia's manual payment record -- full invoice amount, not future-dated -- and sets the invoice to "Paid (recorded by freelancer)." Full spec: `docs/blueprint/specifications/FEAT-10-invoice-payment-processing/FEAT-10.SPEC-005-record-off-platform-payment.md`
- **FR-006**: The system MUST implement **FEAT-10.SPEC-006** (Payment Authorization & Validation Rules) as specified: Governs who may pay, view, or manually record a payment, the full-payment-only and manual-record limits, the reject-with-refresh concurrency behavior, and the payment-account-readiness gate. Full spec: `docs/blueprint/specifications/FEAT-10-invoice-payment-processing/FEAT-10.SPEC-006-payment-authorization-validation-rules.md`
- **FR-007**: The system MUST implement **FEAT-10.SPEC-007** (Payment Confirmation Notification) as specified: Sends a confirmation email to Owen and Nadia the moment a card payment or a confirmed bank transfer succeeds, so both sides have the same timestamped record without checking the portal. Full spec: `docs/blueprint/specifications/FEAT-10-invoice-payment-processing/FEAT-10.SPEC-007-payment-confirmation-notification.md`

### Key Entities

- Payment (create)
- Invoice (update: paid state)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: The median invoice is paid within 3 days of being sent (metric: Time to Payment). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Payment initiations, successes, failures, retries, pending bank transfers and manual recordings are each observable as distinct signals (payment_initiated, payment_succeeded, payment_failed, payment_retried, payment_pending, payment_recorded_manually). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-14**: The platform never holds or moves client funds; payments go directly into the freelancer's own processor account. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-24**: No card or payment data is ever captured or stored by the product itself. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-28**: Payment-processing capability, including pending bank transfers and reversals, is a required dependency. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-06**: Client Primary Contacts are assumed to act on payment requests within days. Full register: `docs/blueprint/features/assumptions-constraints.md`
