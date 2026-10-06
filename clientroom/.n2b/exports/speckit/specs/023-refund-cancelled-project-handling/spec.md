# Feature Specification: Refund & Cancelled Project Handling

**Blueprint feature:** FEAT-25
**Priority tier:** Important
**Build order:** 023 of 33
**Depends on:** FEAT-09, FEAT-10, FEAT-32
**Blueprint source:** `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Mark Invoice Refunded Screen (Priority: P2)

Nadia marks a paid invoice Refunded or Partially refunded, entering the refunded amount and an optional reason, from the invoice detail view.

**Acceptance Scenarios:**

**FEAT-25.SPEC-001-AC-01:** Given Nadia is on the Invoice Detail screen (FEAT-09.SPEC-002) for an invoice with status Paid, when she taps "Record a refund", then she lands on this screen with "Full amount" pre-selected and the amount field showing the full paid total, read-only.

**FEAT-25.SPEC-001-AC-02:** Given Nadia is on this screen with "Full amount" selected, when she taps "Mark Refunded", then FEAT-25.SPEC-003 records the invoice as Refunded and she sees a toast "Invoice marked Refunded" before returning to FEAT-09.SPEC-002.

**FEAT-25.SPEC-001-AC-03:** Given Nadia selects "Partial amount", when the amount field becomes editable, then it starts empty and focus moves to it.

**FEAT-25.SPEC-001-AC-04:** Given Nadia enters an amount greater than the invoice's amount paid, when she blurs the field, then the field shows the exact error defined by FEAT-25.SPEC-006 and Mark Refunded does not proceed.

**FEAT-25.SPEC-001-AC-05:** Given Nadia enters a partial amount equal to the full amount paid, when she taps Mark Refunded, then FEAT-25.SPEC-003 records the invoice as Refunded, not Partially refunded.

**FEAT-25.SPEC-001-AC-06:** Given Nadia enters a partial amount less than the amount paid, when she taps Mark Refunded, then FEAT-25.SPEC-003 records the invoice as Partially refunded and she sees the toast "Invoice marked Partially refunded."

**FEAT-25.SPEC-001-AC-07:** Given Nadia enters an optional reason, when she submits, then the reason is recorded alongside the refund by FEAT-25.SPEC-003.

**FEAT-25.SPEC-001-AC-08:** Given Nadia leaves the reason field empty, when she submits, then the refund is still recorded with no reason stored, since the reason is optional.

**FEAT-25.SPEC-001-AC-09:** Given Nadia taps Mark Refunded and the submission fails due to a network error, then the Error banner "Couldn't record this refund. Try again." appears with her entered amount and reason preserved.

**FEAT-25.SPEC-001-AC-10:** Given Nadia taps Mark Refunded twice rapidly, then the second tap has no effect while the first submission is in progress.

**FEAT-25.SPEC-001-AC-11:** Given the invoice was already marked Refunded, Partially refunded, or Disputed in another session since Nadia opened this screen, when she taps Mark Refunded, then the submission is rejected with "This invoice's status changed since you opened this page. Refresh to see the latest state."

**FEAT-25.SPEC-001-AC-12:** Given a payment reversal is applied to this exact invoice by FEAT-25.SPEC-005 at the same moment Nadia submits a refund, then the processor-confirmed reversal wins, the refund submission is rejected with the refresh message, and the invoice reopens showing Disputed.

**FEAT-25.SPEC-001-AC-13:** Given Nadia loses connectivity while this screen is open, when she looks at the screen, then the banner "You're offline -- this refund can't be recorded until you reconnect." appears and Mark Refunded is disabled.

**FEAT-25.SPEC-001-AC-14:** Given Owen (Client Primary Contact) attempts to reach this screen directly, then it is not reachable from his portal navigation and no such control exists there.

**FEAT-25.SPEC-001-AC-15:** Given Dana (Support Operator) is in a logged support session viewing an invoice that Nadia has marked Refunded, Partially refunded, or Disputed, when she opens the invoice detail screen (FEAT-09.SPEC-002), then she sees the resulting status read-only, no "Record a refund" control is rendered, and she has no path to this screen; a direct link to this screen opens FEAT-09.SPEC-002 in her read-only session with no error message.

**FEAT-25.SPEC-001-AC-16:** Given Nadia's session expires while she has an amount and reason entered, when she re-authenticates, then the entered amount and reason are restored on this screen.

**FEAT-25.SPEC-001-AC-17:** Given Nadia taps "Record a refund" on an invoice, when this screen opens, then a skeleton layout appears for the invoice summary and form until the amount paid and currency finish loading.

**FEAT-25.SPEC-001-AC-18:** Given an invoice with status Paid (recorded by freelancer) whose Payment is in Recorded manually status, when Nadia opens it on FEAT-09.SPEC-002, then "Record a refund" is offered, and when she submits a valid full refund, then FEAT-25.SPEC-003 records the invoice as Refunded and she sees the toast "Invoice marked Refunded."

### User Story 2 - Mark Project Cancelled Screen (Priority: P2)

Nadia marks a project Cancelled, entering an optional reason, from the project detail view, without deleting any project history.

**Acceptance Scenarios:**

**FEAT-25.SPEC-002-AC-01:** Given Nadia is on the Project Detail screen (FEAT-01.SPEC-005), when she selects "Cancel Project" from the overflow menu, then she lands on this screen with the project's name and client shown and the reason field empty.

**FEAT-25.SPEC-002-AC-02:** Given Nadia is on this screen, when she taps Confirm Cancellation, then a blocking dialog appears restating the preservation notice and asking "Cancel this project? This cannot be undone from here." with "Cancel Project" and "Keep Editing" options.

**FEAT-25.SPEC-002-AC-03:** Given Nadia sees the confirming dialog, when she taps "Keep Editing", then the dialog closes and she returns to the Loaded state with her entered reason intact.

**FEAT-25.SPEC-002-AC-04:** Given Nadia sees the confirming dialog, when she taps "Cancel Project" to confirm, then FEAT-25.SPEC-004 records the project as Cancelled and she sees a toast "Project cancelled" before returning to FEAT-01.SPEC-005 showing the Cancelled stage badge.

**FEAT-25.SPEC-002-AC-05:** Given Nadia enters an optional reason, when she confirms cancellation, then the reason is recorded alongside the cancellation by FEAT-25.SPEC-004.

**FEAT-25.SPEC-002-AC-06:** Given Nadia leaves the reason field empty, when she confirms cancellation, then the cancellation is still recorded with no reason stored, since the reason is optional.

**FEAT-25.SPEC-002-AC-07:** Given Nadia confirms cancellation and the submission fails due to a network error, then the Error banner "Couldn't cancel this project. Try again." appears with her entered reason preserved.

**FEAT-25.SPEC-002-AC-08:** Given Nadia taps Confirm Cancellation twice rapidly on the confirming dialog, then the second tap has no effect while the first submission is in progress.

**FEAT-25.SPEC-002-AC-09:** Given the project's state changed in another session since Nadia opened this screen (it was marked Complete, Archived, or already Cancelled), when she confirms cancellation, then the submission is rejected with "This project's state changed since you opened this page. Refresh to see the latest state."

**FEAT-25.SPEC-002-AC-10:** Given a milestone is approved on this same project at the same moment Nadia confirms cancellation, then the milestone approval still records normally and the project's stage shows Cancelled, since Nadia's explicit cancellation is never overwritten by a system-driven stage change.

**FEAT-25.SPEC-002-AC-11:** Given the project has open, unpaid invoices, when Nadia cancels it, then no blocking confirmation about the open invoices appears and the invoices remain unchanged.

**FEAT-25.SPEC-002-AC-12:** Given Nadia loses connectivity while this screen is open, when she looks at the screen, then the banner "You're offline -- this cancellation can't be recorded until you reconnect." appears and Confirm Cancellation is disabled.

**FEAT-25.SPEC-002-AC-13:** Given Owen (Client Primary Contact) attempts to reach this screen directly, then it is not reachable from his portal navigation and no such control exists there.

**FEAT-25.SPEC-002-AC-14:** Given Dana (Support Operator) is in a logged support session viewing a project Nadia has marked Cancelled, when she opens the project detail screen (FEAT-01.SPEC-005), then she sees the Cancelled status read-only, no "Cancel Project" entry is rendered, and she has no path to this screen; a direct link to this screen opens FEAT-01.SPEC-005 in her read-only session with no error message.

**FEAT-25.SPEC-002-AC-15:** Given Nadia's session expires while she has a reason entered, when she re-authenticates, then the entered reason is restored on this screen.

**FEAT-25.SPEC-002-AC-16:** Given Nadia selects "Cancel Project", when this screen opens, then a skeleton layout appears for the project summary and form until the project's name and client finish loading.

**FEAT-25.SPEC-002-AC-17:** Given a project whose derived stage is Complete, Archived, or already Cancelled, when Nadia opens its project detail screen (FEAT-01.SPEC-005), then no "Cancel Project" entry is offered and this screen cannot be reached from that project.

### User Story 3 - Refund & Partial Refund Recording (Priority: P2)

Validates and persists Nadia's refund entry -- full or partial, never exceeding the amount paid -- sets the invoice to Refunded or Partially refunded, and preserves the original Paid record rather than overwriting it.

**Acceptance Scenarios:**

**FEAT-25.SPEC-003-AC-01:** Given Nadia submits a refund equal to the full amount paid on a Paid invoice, when this automation processes it, then the Invoice's status is set to Refunded and the prior Paid record remains visible alongside it.

**FEAT-25.SPEC-003-AC-02:** Given Nadia submits a refund less than the full amount paid, when this automation processes it, then the Invoice's status is set to Partially refunded and the refunded amount is recorded against the Payment.

**FEAT-25.SPEC-003-AC-03:** Given the invoice is no longer in a refund-eligible combination at the moment this automation commits (it changed since the screen was loaded), when the commit-time check runs, then no write occurs and FEAT-25.SPEC-001 shows the refresh message.

**FEAT-25.SPEC-003-AC-04:** Given a refund amount that exceeds the amount paid somehow reaches this automation's commit-time check, when the check runs, then the write is refused and FEAT-25.SPEC-001 shows FEAT-25.SPEC-006's exact error message.

**FEAT-25.SPEC-003-AC-05:** Given a refund is successfully recorded, when the commit completes, then FEAT-25.SPEC-007 is triggered to email Owen and FEAT-13 is signaled to write the trail entry.

**FEAT-25.SPEC-003-AC-06:** Given persisting the Invoice and Payment changes fails after validation passes, when the failure occurs, then FEAT-25.SPEC-001 shows "Couldn't record this refund. Try again." and the Invoice's status remains as it was (Paid, or Paid (recorded by freelancer)).

**FEAT-25.SPEC-003-AC-07:** Given two refund submissions for the same invoice arrive from two sessions at effectively the same time, when the first commits, then the second's commit-time check finds the invoice no longer refund-eligible and is rejected.

**FEAT-25.SPEC-003-AC-08:** Given a reversal is recorded for this exact invoice a moment before this automation's own commit, when the commit-time check runs, then it finds the Invoice already Disputed and rejects the refund.

**FEAT-25.SPEC-003-AC-09:** Given a refund amount exactly equal to the amount paid was entered through the "Partial amount" option, when this automation processes it, then the resulting status is Refunded, not Partially refunded.

**FEAT-25.SPEC-003-AC-10:** Given a refund is recorded, when the Financial Dashboard (FEAT-12) or Accounting Export (FEAT-22) totals are next computed, then they reflect the refunded amount.

**FEAT-25.SPEC-003-AC-11:** Given an optional reason accompanies the refund submission, when this automation commits, then the reason is recorded against the Payment alongside the refunded amount.

**FEAT-25.SPEC-003-AC-12:** Given no reason accompanies the refund submission, when this automation commits, then the refund is still recorded with no reason stored.

**FEAT-25.SPEC-003-AC-13:** Given an invoice with status Paid (recorded by freelancer) whose Payment `status` is Recorded manually, when Nadia submits a refund equal to the amount paid, then the Invoice's status is set to Refunded, the Payment `status` remains Recorded manually, and the prior paid record remains visible alongside the new status.

**FEAT-25.SPEC-003-AC-14:** Given an invoice whose status is Paid but whose Payment `status` is neither Succeeded nor a match for that invoice status (or whose status is Generated, Sent, Payment pending, Overdue, or Corrected), when a refund submission reaches the commit-time check, then no write occurs and FEAT-25.SPEC-001 shows the refresh message.

### User Story 4 - Project Cancellation Recording (Priority: P2)

Persists Nadia's cancellation of a project, sets `cancelled_at`, derives the Cancelled stage, and preserves every existing proposal, milestone, deliverable, and invoice record unchanged.

**Acceptance Scenarios:**

**FEAT-25.SPEC-004-AC-01:** Given Nadia confirms cancellation on a project whose derived stage is none of Cancelled, Archived, or Complete, when this automation processes it, then `cancelled_at` is set and the project's derived stage resolves to Cancelled.

**FEAT-25.SPEC-004-AC-02:** Given a cancellation is recorded, when the Project Detail screen (FEAT-01.SPEC-005) is next opened, then it shows the Cancelled stage badge and every proposal, milestone, deliverable, and invoice record is unchanged.

**FEAT-25.SPEC-004-AC-03:** Given the project's derived stage at commit is Cancelled, Archived, or Complete (changed since the screen loaded), when the commit-time check runs, then no write occurs and FEAT-25.SPEC-002 shows the refresh message.

**FEAT-25.SPEC-004-AC-04:** Given a cancellation is successfully recorded, when the commit completes, then FEAT-25.SPEC-007 is triggered to email Owen and FEAT-13 is signaled to write the trail entry.

**FEAT-25.SPEC-004-AC-05:** Given persisting `cancelled_at` fails after the stale-state check passes, when the failure occurs, then FEAT-25.SPEC-002 shows "Couldn't cancel this project. Try again." and the project's prior stage remains displayed.

**FEAT-25.SPEC-004-AC-06:** Given two cancellation confirmations for the same project arrive from two sessions at effectively the same time, when the first commits, then the second's commit-time check finds the stage already Cancelled and is rejected.

**FEAT-25.SPEC-004-AC-07:** Given a milestone is approved on this same project at effectively the same moment as this automation's commit, when both are processed, then the milestone remains Approved and the project's displayed stage still resolves to Cancelled.

**FEAT-25.SPEC-004-AC-08:** Given Nadia marks the same project Complete via FEAT-01.SPEC-006 at effectively the same moment this automation commits the cancellation, when both attempts race, then only the first to commit succeeds and the second is rejected with its own screen's refresh message; if Complete commits first, this automation's step 3 finds `completed_at` set, writes nothing, and FEAT-25.SPEC-002 shows "This project's state changed since you opened this page. Refresh to see the latest state."

**FEAT-25.SPEC-004-AC-09:** Given the project has an unaccepted proposal, when this automation records the cancellation, then the Proposal record itself is untouched -- it stays open exactly as it was (XBR-25).

**FEAT-25.SPEC-004-AC-10:** Given the project has open, unpaid invoices, when this automation records the cancellation, then those invoices are untouched.

**FEAT-25.SPEC-004-AC-11:** Given no reason accompanies the cancellation submission, when this automation commits, then the cancellation is still recorded with no reason stored.

### User Story 5 - Payment Reversal (Chargeback) Recording (Priority: P2)

Applies an inbound reversal or chargeback notice relayed from the payment-processing capability to a Paid invoice, setting it Disputed alongside its preserved Paid record and marking the underlying Payment Reversed.

**Acceptance Scenarios:**

**FEAT-25.SPEC-005-AC-01:** Given a Paid invoice with a Succeeded Payment, when the payment-processing capability reports a reversal, then the Invoice's status is set to Disputed and the prior Paid record remains visible alongside it.

**FEAT-25.SPEC-005-AC-02:** Given a reversal is recorded, when the commit completes, then the underlying Payment's status is set to Reversed.

**FEAT-25.SPEC-005-AC-03:** Given a reversal is recorded, when the commit completes, then FEAT-25.SPEC-008 is triggered to email Nadia immediately and FEAT-13 is signaled to write the trail entry.

**FEAT-25.SPEC-005-AC-04:** Given an invoice already Refunded or Partially refunded by Nadia's own prior action, when the payment-processing capability reports a reversal against it, then the Invoice's status is still set to Disputed, since the Paid-family check covers all three prior statuses.

**FEAT-25.SPEC-005-AC-05:** Given a reversal notice arrives for an invoice with no Payment ever recorded as Succeeded, when this automation checks the Payment's status, then no write occurs and no notification fires.

**FEAT-25.SPEC-005-AC-06:** Given two reversal notices for the same invoice arrive at effectively the same time, when the first commits, then the second finds the invoice already Disputed and the Payment already Reversed at step 3, ends with the "Duplicate / already-Disputed no-op" outcome, and applies no further change and sends no second email.

**FEAT-25.SPEC-005-AC-07:** Given Nadia submits a manual refund for an invoice at the same moment a reversal for that invoice commits, when the reversal commits first, then FEAT-25.SPEC-003 finds the invoice already Disputed and rejects the refund submission.

**FEAT-25.SPEC-005-AC-08:** Given Nadia's manual refund commits first for an invoice a moment before a reversal notice for that same invoice arrives, when this automation processes the reversal, then it still finds the invoice in a Paid-family status (Refunded or Partially refunded) and sets it to Disputed.

**FEAT-25.SPEC-005-AC-09:** Given a reversal is recorded, when the Financial Dashboard (FEAT-12) or Accounting Export (FEAT-22) totals are next computed, then they reflect the Disputed status and the Reversed payment.

**FEAT-25.SPEC-005-AC-10:** Given persisting the Invoice and Payment changes fails after the Paid-family check passes, when the failure occurs, then no partial write is left behind and the event is retried by FEAT-32.SPEC-002's own retry contract.

**FEAT-25.SPEC-005-AC-11:** Given an invoice already Disputed with its Payment already Reversed, when a further reversal notice for that invoice arrives at any later time, then the outcome is "Duplicate / already-Disputed no-op": no write occurs, FEAT-25.SPEC-008 is not triggered (no second email), no trail entry is written, and payment_reversal_duplicate_ignored is emitted, not payment_reversal_notice_discarded.

**FEAT-25.SPEC-005-AC-12:** Given an invoice with status Paid (recorded by freelancer) whose Payment is in Recorded manually status, when a reversal notice for it arrives, then the outcome is "Notice discarded -- no matching payment": no write, no notification, and payment_reversal_notice_discarded is emitted.

### User Story 6 - Refund, Cancellation & Reversal Authorization and Validation Rules (Priority: P2)

Governs who may mark a refund or cancellation, the refund-amount and no-partial-payment limits, the refunded-cannot-be-repaid-without-correction rule, reject-with-refresh concurrency, which invoice and project states are eligible, and Dana's view-only status visibility.

**Acceptance Scenarios:**

**FEAT-25.SPEC-006-AC-01:** Given Nadia enters a refund amount of exactly the amount paid, when she submits, then validation passes and FEAT-25.SPEC-003 records the invoice as Refunded.

**FEAT-25.SPEC-006-AC-02:** Given Nadia enters a refund amount one unit over the amount paid, when she blurs the field, then she sees "This amount is more than what was paid. The amount paid was {amount paid}." and the field remains in an error state.

**FEAT-25.SPEC-006-AC-03:** Given Nadia enters a refund amount of zero, when she blurs the field, then she sees "Enter an amount greater than zero."

**FEAT-25.SPEC-006-AC-04:** Given Nadia enters a refund amount with more than two decimal places, when she blurs the field, then she sees "Enter a valid amount in {currency}."

**FEAT-25.SPEC-006-AC-05:** Given Nadia (Freelancer) opens her own invoice with status Paid (Payment Succeeded), or with status Paid (recorded by freelancer) (Payment Recorded manually), when she looks for "Record a refund", then it is shown and reachable in both cases.

**FEAT-25.SPEC-006-AC-06:** Given Owen (Client Primary Contact) looks anywhere in his portal navigation, when he searches for a refund or cancellation action, then none exists.

**FEAT-25.SPEC-006-AC-07:** Given Priya (Client Reviewer Contact) attempts to reach an invoice directly, then invoice content is hidden entirely and she is redirected to her portal home with no error message.

**FEAT-25.SPEC-006-AC-08:** Given Dana (Support Operator) opens a logged support session on an account with a Paid invoice, when she views the invoice detail screen (FEAT-09.SPEC-002), then no "Record a refund" control is rendered, she has no path to FEAT-25.SPEC-001, and a direct link to it opens the invoice detail read-only with no error message.

**FEAT-25.SPEC-006-AC-09:** Given Nadia (Freelancer) opens a project whose derived stage is none of Cancelled, Archived, or Complete, when she looks for "Cancel Project", then it is shown and reachable.

**FEAT-25.SPEC-006-AC-10:** Given Dana (Support Operator) opens a logged support session on an account with an active project, when she views the project detail screen (FEAT-01.SPEC-005), then no "Cancel Project" entry is rendered, she has no path to FEAT-25.SPEC-002, and she sees the project's status read-only.

**FEAT-25.SPEC-006-AC-11:** Given Owen (Client Primary Contact) views his own company's invoice, when it is Refunded or Partially refunded, then he sees the resulting status alongside the preserved Paid record.

**FEAT-25.SPEC-006-AC-12:** Given Priya (Client Reviewer Contact) views her own project's portal page, when the project is Cancelled, then she sees the resulting Cancelled status (per the Access Matrix's Own-only portal view), with no billing content mixed into that view.

**FEAT-25.SPEC-006-AC-13:** Given no control anywhere in the product offers "mark Paid" on a Refunded invoice, when Nadia looks for one, then none exists, and any correction must go through a new credit note or invoice via FEAT-09.

**FEAT-25.SPEC-006-AC-14:** Given a reversal commits first for an invoice, when Nadia's concurrent manual refund submission for that same invoice reaches commit, then it is refused with "This invoice's status changed since you opened this page. Refresh to see the latest state."

**FEAT-25.SPEC-006-AC-15:** Given Nadia's manual refund commits first for an invoice, when a reversal notice for that same invoice is then processed, then the reversal still applies and sets the invoice Disputed, since a processor-confirmed reversal is authoritative over the prior manual entry.

**FEAT-25.SPEC-006-AC-16:** Given a project's stage changed (to Complete, Archived, or already Cancelled) between when Nadia loaded the Mark Project Cancelled screen and when she confirms, when she confirms, then the submission is rejected with "This project's state changed since you opened this page. Refresh to see the latest state." and no overwrite occurs; and given the project's stage is any other stage at confirmation, then the cancellation succeeds with the plain "Project cancelled" toast and no stale-state message.

**FEAT-25.SPEC-006-AC-17:** Given an invoice's status changed since Nadia loaded the Mark Invoice Refunded screen, when she submits her refund, then the submission is rejected with the refresh message and no overwrite occurs.

**FEAT-25.SPEC-006-AC-18:** Given an invoice has already reached its one full Payment (SC-17: no instalments exist), when Nadia enters a refund amount, then the ceiling checked is that single Payment's full amount, never a sum across multiple payments.

**FEAT-25.SPEC-006-AC-19:** Given Nadia leaves the refund reason or cancellation reason field empty, when she submits, then no validation error appears, since both reason fields are optional.

**FEAT-25.SPEC-006-AC-20:** Given a reversal is recorded on a previously Refunded or Partially refunded invoice, when the reversal is processed, then it still applies and sets the invoice Disputed, since the Paid-family check covers all three statuses that can precede a reversal.

**FEAT-25.SPEC-006-AC-21:** Given an invoice with status Paid whose Payment is in Succeeded status, or status Paid (recorded by freelancer) whose Payment is in Recorded manually status, when Nadia submits a valid refund amount, then the eligibility check passes and FEAT-25.SPEC-003 records the refund.

**FEAT-25.SPEC-006-AC-22:** Given an invoice whose status is Generated, Sent, Payment pending, Overdue, Refunded, Partially refunded, Disputed, or Corrected, or a Paid invoice whose Payment is Reversed, when Nadia looks for "Record a refund" or a stale submission reaches commit, then the control is not offered, or the submission is refused with "This invoice's status changed since you opened this page. Refresh to see the latest state."

**FEAT-25.SPEC-006-AC-23:** Given a project Nadia has marked Complete, when she opens its project detail screen, then no "Cancel Project" entry is offered, and a stale cancellation confirmation submitted for it is refused with the refresh message.

### User Story 7 - Refund & Cancellation Notification (Priority: P2)

Emails Owen when an invoice he was billed is marked Refunded/Partially refunded or when his project is marked Cancelled, so his own record of the relationship stays accurate without asking Nadia.

**Acceptance Scenarios:**

**FEAT-25.SPEC-007-AC-01:** Given Nadia records a full refund on Owen's invoice, when the commit succeeds, then Owen receives an email with subject "Invoice {invoice_number} has been refunded" and a "View invoice" CTA.

**FEAT-25.SPEC-007-AC-02:** Given Nadia records a partial refund on Owen's invoice, when the commit succeeds, then Owen receives an email with subject "Invoice {invoice_number} has been partially refunded" naming the refunded amount.

**FEAT-25.SPEC-007-AC-03:** Given Nadia marks Owen's project Cancelled, when the commit succeeds, then Owen receives an email with subject "{project_name} has been marked cancelled" and a "View project" CTA to his own portal view.

**FEAT-25.SPEC-007-AC-04:** Given Owen taps "View invoice" on a refund email, then he lands on FEAT-09.SPEC-002 showing the invoice's Refunded or Partially refunded status.

**FEAT-25.SPEC-007-AC-05:** Given Owen taps "View project" on a cancellation email, then he lands on his own client-facing project view showing the Cancelled status.

**FEAT-25.SPEC-007-AC-06:** Given Nadia performs the refund or cancellation herself, then she receives no email from this spec -- only Owen is a recipient.

**FEAT-25.SPEC-007-AC-07:** Given Priya is a contact on the same client company, when a refund or cancellation is recorded, then she receives no email from this spec.

**FEAT-25.SPEC-007-AC-08:** Given Owen's email address bounces on the first delivery attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the project for Nadia.

**FEAT-25.SPEC-007-AC-09:** Given a delivery to Owen fails permanently after retries are exhausted, then Nadia sees a delivery warning on the affected project, and Owen still sees the refunded or cancelled status directly in his portal.

**FEAT-25.SPEC-007-AC-10:** Given Owen's Client Contact record is removed by an erasure request before this email is delivered, when the delivery would otherwise fire, then it is cancelled silently and no email is sent to the removed address.

**FEAT-25.SPEC-007-AC-11:** Given two refunds are recorded on two different invoices for the same project at effectively the same time, then Owen receives two separate emails, never one combined message.

**FEAT-25.SPEC-007-AC-12:** Given a project is cancelled and one of its invoices is separately refunded moments later, then Owen receives two separate emails in the order the two events occurred.

**FEAT-25.SPEC-007-AC-13:** Given this notification is transactional, when Owen looks for a way to turn it off, then no preference control for it exists anywhere.

**FEAT-25.SPEC-007-AC-14:** Given a refund or cancellation is recorded at any hour, when the email is ready to send, then it sends immediately with no quiet-hours hold, since this notification is transactional.

### User Story 8 - Payment Reversal Notification (Priority: P2)

Emails Nadia the moment a payment reversal or chargeback is recorded, so she knows to respond in her own processor account.

**Acceptance Scenarios:**

**FEAT-25.SPEC-008-AC-01:** Given a reversal is recorded on one of Nadia's invoices, when the commit succeeds, then she receives an email with subject "Payment reversal reported on invoice {invoice_number}" immediately.

**FEAT-25.SPEC-008-AC-02:** Given Nadia opens the reversal email, when she taps "View invoice", then she lands on FEAT-09.SPEC-002 showing the invoice's Disputed status alongside its preserved prior record.

**FEAT-25.SPEC-008-AC-03:** Given the reversal email body, when Nadia reads it, then it states plainly that she must respond to the dispute directly in her payment processor account, since Clientroom does not handle chargebacks or disputes itself.

**FEAT-25.SPEC-008-AC-04:** Given Owen is the client on the reversed invoice, when the reversal is recorded, then he receives no email from this spec.

**FEAT-25.SPEC-008-AC-05:** Given Nadia's email bounces on the first delivery attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before a delivery warning appears to her in-product.

**FEAT-25.SPEC-008-AC-06:** Given delivery to Nadia fails permanently after retries are exhausted, then she sees a delivery warning on the affected project, and the invoice's Disputed status is still visible whenever she next opens it.

**FEAT-25.SPEC-008-AC-07:** Given two reversal notices arrive for the same invoice at effectively the same time, then only one email is sent, since the second commit is an idempotent no-op.

**FEAT-25.SPEC-008-AC-08:** Given the invoice was already Partially refunded by Nadia's own action before this reversal, when the reversal email is sent, then it still names the Payment's original amount as the reversed amount.

**FEAT-25.SPEC-008-AC-09:** Given a reversal is recorded at any hour, when the email is ready to send, then it sends immediately with no quiet-hours hold, since this notification is transactional and time-critical.

**FEAT-25.SPEC-008-AC-10:** Given Nadia looks for a way to turn this notification off, when she checks FEAT-21.SPEC-002 (Notification Preferences), then no toggle for it exists there.

**FEAT-25.SPEC-008-AC-11:** Given Nadia's Freelancer Account is deleted between the reversal commit and this email's delivery, when the delivery would otherwise fire, then it is cancelled silently.

**FEAT-25.SPEC-008-AC-12:** Given Nadia is away from email when a reversal is recorded, when she next checks her inbox, then the queued email is present with its original content, unmodified by the passage of time.

### Edge Cases

- **FEAT-25.SPEC-001 (Mark Invoice Refunded Screen):** Leaving with an unsaved amount or reason needs no confirmation since nothing is written, and double taps are ignored. A status change or reversal from another session is rejected with a refresh prompt, and a processor-confirmed reversal is authoritative over a concurrent manual refund. Source: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-001-mark-invoice-refunded-screen.md` (section: Edge Cases)
- **FEAT-25.SPEC-002 (Mark Project Cancelled Screen):** Leaving the reason field discards it without extra confirmation beyond the Confirming dialog, double taps on Confirm Cancellation are ignored, and a project state changed from another session (Complete, Archived or already Cancelled) is rejected with a refresh prompt. The freelancer's explicit Cancelled transition takes precedence over a concurrent system-driven stage change. Source: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-002-mark-project-cancelled-screen.md` (section: Edge Cases)
- **FEAT-25.SPEC-003 (Refund & Partial Refund Recording):** Of two simultaneous refunds the first commit moves status away from Paid and the second re-checks and is rejected as stale, and an amount exactly equal to the amount paid is recorded as Refunded even via the Partial amount option. A reversal committing just before the refund leaves the invoice Disputed and the refund rejected. Source: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-003-refund-partial-refund-recording.md` (section: Edge Cases)
- **FEAT-25.SPEC-004 (Project Cancellation Recording):** Of two simultaneous cancellations the first sets cancelled_at and the second finds the stage already Cancelled, with the screen disabling Confirm during submission. A milestone approval or proposal acceptance committing at the same moment is still recorded, and a concurrent Mark Complete resolves first-commit-wins with the second rejected as stale. Source: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-004-project-cancellation-recording.md` (section: Edge Cases)
- **FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording):** A reversal notice for an invoice with no Succeeded payment, or one paid off-platform with no processor-confirmed payment, is discarded with no write and no notification. Duplicate or concurrent notices for the same invoice apply once (Disputed) with later runs serialized behind the first. Source: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-005-payment-reversal-chargeback-recording.md` (section: Edge Cases)
- **FEAT-25.SPEC-006 (Refund, Cancellation & Reversal Authorization and Validation Rules):** A refund at exactly the amount paid passes and records as Refunded, while one currency unit over fails with the more-than-paid message. No control exists to mark a Refunded invoice Paid again (enforced by omission), and a reversal racing a manual refund resolves to whichever commits first. Source: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-006-refund-cancellation-reversal-authorization-validation-rules.md` (section: Edge Cases)
- **FEAT-25.SPEC-007 (Refund & Cancellation Notification):** A later reversal sends no second email to the client (that is the freelancer's own notification, FEAT-25.SPEC-008), a bounced address retries per the platform-parameter count and window and then surfaces a delivery warning, and an erased contact's pending email is cancelled silently (XBR-27). Refunds on two invoices of one project each send a separate email. Source: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-007-refund-cancellation-notification.md` (section: Edge Cases)
- **FEAT-25.SPEC-008 (Payment Reversal Notification):** A bounced or nonexistent sign-in address is retried per the platform parameters and then surfaced, duplicate reversal notices produce only one email, and a reversal on an invoice already partially refunded still sends with the Payment's original amount. The email is queued when the freelancer is away since email is the only channel. Source: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-008-payment-reversal-notification.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-25.SPEC-001** (Mark Invoice Refunded Screen) as specified: Nadia marks a paid invoice Refunded or Partially refunded, entering the refunded amount and an optional reason, from the invoice detail view. Full spec: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-001-mark-invoice-refunded-screen.md`
- **FR-002**: The system MUST implement **FEAT-25.SPEC-002** (Mark Project Cancelled Screen) as specified: Nadia marks a project Cancelled, entering an optional reason, from the project detail view, without deleting any project history. Full spec: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-002-mark-project-cancelled-screen.md`
- **FR-003**: The system MUST implement **FEAT-25.SPEC-003** (Refund & Partial Refund Recording) as specified: Validates and persists Nadia's refund entry -- full or partial, never exceeding the amount paid -- sets the invoice to Refunded or Partially refunded, and preserves the original Paid record rather than overwriting it. Full spec: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-003-refund-partial-refund-recording.md`
- **FR-004**: The system MUST implement **FEAT-25.SPEC-004** (Project Cancellation Recording) as specified: Persists Nadia's cancellation of a project, sets `cancelled_at`, derives the Cancelled stage, and preserves every existing proposal, milestone, deliverable, and invoice record unchanged. Full spec: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-004-project-cancellation-recording.md`
- **FR-005**: The system MUST implement **FEAT-25.SPEC-005** (Payment Reversal (Chargeback) Recording) as specified: Applies an inbound reversal or chargeback notice relayed from the payment-processing capability to a Paid invoice, setting it Disputed alongside its preserved Paid record and marking the underlying Payment Reversed. Full spec: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-005-payment-reversal-chargeback-recording.md`
- **FR-006**: The system MUST implement **FEAT-25.SPEC-006** (Refund, Cancellation & Reversal Authorization and Validation Rules) as specified: Governs who may mark a refund or cancellation, the refund-amount and no-partial-payment limits, the refunded-cannot-be-repaid-without-correction rule, reject-with-refresh concurrency, which invoice and project states are eligible, and Dana's view-only status visibility. Full spec: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-006-refund-cancellation-reversal-authorization-validation-rules.md`
- **FR-007**: The system MUST implement **FEAT-25.SPEC-007** (Refund & Cancellation Notification) as specified: Emails Owen when an invoice he was billed is marked Refunded/Partially refunded or when his project is marked Cancelled, so his own record of the relationship stays accurate without asking Nadia. Full spec: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-007-refund-cancellation-notification.md`
- **FR-008**: The system MUST implement **FEAT-25.SPEC-008** (Payment Reversal Notification) as specified: Emails Nadia the moment a payment reversal or chargeback is recorded, so she knows to respond in her own processor account. Full spec: `docs/blueprint/specifications/FEAT-25-refund-cancelled-project-handling/FEAT-25.SPEC-008-payment-reversal-notification.md`

### Key Entities

- Invoice (update: refunded/cancelled/disputed state)
- Project (update: cancelled state)
- Payment (update: reversed) [AUDIT-ADDED: 1 -- value-flow walk]

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Invoice refunds, project cancellations, partial refunds and payment reversals are each observable as distinct signals (invoice_marked_refunded, project_marked_cancelled, partial_refund_recorded, payment_reversal_recorded); no metric in the success-metrics register connects to this feature, so the outcome is grounded in its Signals alone. Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-14**: The platform never holds or moves client funds, so refunds and reversals are recorded rather than executed by the product. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-15**: Records are append-only and immutable once created. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-28**: Payment-processing capability reports reversals back to the product. Full register: `docs/blueprint/features/assumptions-constraints.md`
