# Feature Specification: Automated Payment Reminders

**Blueprint feature:** FEAT-11
**Priority tier:** Core
**Build order:** 020 of 33
**Depends on:** FEAT-09, FEAT-10, FEAT-15
**Blueprint source:** `docs/blueprint/specifications/FEAT-11-automated-payment-reminders/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Reminder Schedule (Priority: P1)

Automatically sends the day-3 and day-10 overdue reminder email for an unpaid invoice, re-checking eligibility immediately before each send so a reminder never goes out to an invoice that has since been paid or paused.

**Acceptance Scenarios:**

**FEAT-11.SPEC-001-AC-01:** Given Nadia has an invoice for Owen that reaches exactly 3 elapsed days unpaid past its due date (freelancer's time zone), when the schedule evaluates that invoice, then a `day 3` Reminder Log entry is created, the eligibility check passes, and the Overdue Reminder Email is sent to Owen.

**FEAT-11.SPEC-001-AC-02:** Given the same invoice reaches exactly 10 elapsed days still unpaid, when the schedule evaluates it, then a `day 10` Reminder Log entry is created and sent, and no further automatic reminder is ever scheduled for this invoice.

**FEAT-11.SPEC-001-AC-03:** Given an invoice reaches day 3 but Owen paid it one minute before the schedule's eligibility re-check runs, when the re-check executes, then the send is skipped, no Reminder Log entry is created, and no email is sent.

**FEAT-11.SPEC-001-AC-04:** Given an invoice reaches day 3 while Nadia has paused reminders for it, when the schedule evaluates it, then the send is skipped per FEAT-11.SPEC-002 and no Reminder Log entry is created for day 3.

**FEAT-11.SPEC-001-AC-05:** Given an invoice reaches day 10 while it is Paused while bank transfer pending, when the schedule evaluates it, then the send is skipped, and the invoice never receives the day-10 reminder even if the pending transfer later fails and the invoice becomes unpaid again after day 10 has passed.

**FEAT-11.SPEC-001-AC-06:** Given an invoice's elapsed-day count is 5 (between the two thresholds), when the schedule evaluates it, then no reminder action is taken.

**FEAT-11.SPEC-001-AC-07:** Given the Overdue Reminder Email's hand-off fails on the first attempt for a day-3 candidate, when the automation retries per FEAT-14.SPEC-001's retry rule and the retry succeeds, then the Reminder Log entry's `sent_at` is set on the successful attempt and no delivery warning is shown to Nadia.

**FEAT-11.SPEC-001-AC-08:** Given the Overdue Reminder Email's hand-off fails on every retry attempt, when the final retry is exhausted, then the failure is logged rather than dropped silently and Nadia sees a delivery warning on the affected project.

**FEAT-11.SPEC-001-AC-09:** Given the schedule evaluation was unavailable for two days and an invoice passed day 3 unsent during that gap, when the schedule next runs, then the day-3 candidate is still detected (no existing `day 3` entry) and sent, provided the invoice is still eligible at that later moment.

**FEAT-11.SPEC-001-AC-10:** Given two schedule evaluations fire for the same invoice's day-3 threshold at effectively the same time, when both attempt to create the `day 3` entry, then only one Reminder Log entry is created and only one email is sent -- the second evaluation's dedup check finds the existing entry and takes no action.

**FEAT-11.SPEC-001-AC-11:** Given an invoice's day-3 threshold was skipped because Nadia had paused reminders, and she resumes before day 10, when the invoice reaches day 10 elapsed, then the eligibility check passes and the day-10 reminder sends normally -- the earlier skip does not block the later threshold.

**FEAT-11.SPEC-001-AC-12:** Given an invoice is Refunded before ever reaching day 3, when the schedule evaluates invoices, then this invoice is excluded from candidacy entirely, since its status is no longer one that can be overdue.

### User Story 2 - Reminder Eligibility Rule (Priority: P1)

Defines whether a reminder -- automatic or manual -- may send right now: the paid check, the pause checks, the time-zone day-counting basis, and the one-manual-reminder-per-day limit, plus every role's authority over pausing, resuming, and manually sending.

**Acceptance Scenarios:**

**FEAT-11.SPEC-002-AC-01:** Given Nadia's invoice to Owen is Overdue with `pause_state` Active and no reminder sent today, when the Eligible-to-send rule is checked for a manual send, then it returns eligible.

**FEAT-11.SPEC-002-AC-02:** Given the same invoice has just been marked Paid, when the Eligible-to-send rule is checked (automatic or manual), then it returns not eligible and the Send Reminder Now control is not shown on FEAT-11.SPEC-003.

**FEAT-11.SPEC-002-AC-03:** Given the invoice's `pause_state` is Paused by freelancer, when a manual send is attempted, then it is denied with "Reminders are paused for this invoice. Resume reminders to send one now."

**FEAT-11.SPEC-002-AC-04:** Given Nadia already sent a manual reminder for this invoice earlier today (freelancer's local day), when she attempts to send another today, then it is denied with "You've already sent a reminder for this invoice today. You can send another tomorrow."

**FEAT-11.SPEC-002-AC-05:** Given Nadia sent a manual reminder at 23:59 freelancer-local time, when she attempts another at 00:02 the following freelancer-local day, then the send is allowed, since the two attempts fall on different calendar days.

**FEAT-11.SPEC-002-AC-06:** Given an invoice's `due_date` was yesterday in the freelancer's time zone, when the Overdue-day derivation runs, then the invoice's status is Overdue with elapsed days = 1.

**FEAT-11.SPEC-002-AC-07:** Given an invoice is Refunded, when the Overdue-day derivation runs, then the invoice's status is never set to Overdue, regardless of how many days have elapsed since its due date.

**FEAT-11.SPEC-002-AC-08:** Given an invoice's `pause_state` is currently Active, when the Bank-Transfer-Pending Reminder Pause (FEAT-11.SPEC-005) applies its pause, then `pause_state` becomes Paused while bank transfer pending.

**FEAT-11.SPEC-002-AC-09:** Given an invoice's `pause_state` is currently Paused by freelancer, when a bank transfer for that invoice enters pending status, then `pause_state` remains Paused by freelancer -- the automatic pause never overwrites Nadia's own pause.

**FEAT-11.SPEC-002-AC-10:** Given an invoice's `pause_state` is Paused while bank transfer pending, when Nadia taps Pause on FEAT-11.SPEC-003, then `pause_state` becomes Paused by freelancer, overwriting the automatic pause reason.

**FEAT-11.SPEC-002-AC-11:** Given an invoice's `pause_state` is Paused while bank transfer pending, when the pending transfer resolves to Paid or reverts, then `pause_state` returns to Active (subject to the invoice's paid check for the Paid outcome).

**FEAT-11.SPEC-002-AC-12:** Given Nadia (Freelancer) opens the Invoice Reminder Panel for her own invoice, when the panel loads, then she can view the full reminder history and pause state, and the Pause/Resume and Send Reminder Now controls are active per her invoice's current state.

**FEAT-11.SPEC-002-AC-13:** Given Dana (Support Operator) has an open support session on the freelancer's account, when she opens the same panel, then she can view the same history and state read-only, and every action control appears disabled with "unavailable in a read-only support session."

**FEAT-11.SPEC-002-AC-14:** Given Owen (Client Primary Contact) is signed in to his own portal, when he looks for any path to the Invoice Reminder Panel, then none exists -- the panel is never shown to him.

**FEAT-11.SPEC-002-AC-15:** Given Priya (Client Reviewer Contact) is signed in to her own portal, when she looks for any path to the panel or for a reminder email addressed to her, then neither exists.

**FEAT-11.SPEC-002-AC-16:** Given Nadia has paused reminders and two of her own browser sessions both attempt to send a manual reminder for the same invoice at effectively the same time, when both requests reach the Eligible-to-send check, then at most one succeeds and the second is refused with refresh, per the one-per-day limit reading the first request's just-created entry.

### User Story 3 - Invoice Reminder Panel (Priority: P1)

Shows an invoice's reminder history and lets Nadia pause or resume the automatic reminder schedule and send a manual reminder, all scoped to one invoice.

**Acceptance Scenarios:**

**FEAT-11.SPEC-003-AC-01:** Given Nadia opens the Invoice Detail for an Overdue invoice that has already had its day-3 reminder sent, when the panel loads, then she sees one "Day 3" history row with its send timestamp in her own time zone.

**FEAT-11.SPEC-003-AC-02:** Given Nadia opens the panel for an invoice not yet at day 3, when the panel loads, then she sees "No reminders sent yet -- automatic reminders begin 3 days after the due date if this invoice stays unpaid."

**FEAT-11.SPEC-003-AC-03:** Given Nadia taps "Pause reminders" on an Active invoice, when the action completes, then the indicator switches to "Paused by you", the toggle becomes "Resume reminders", and she sees the toast "Reminders paused for this invoice."

**FEAT-11.SPEC-003-AC-04:** Given the invoice's `pause_state` is Paused while bank transfer pending, when Nadia views the panel, then she sees the static text "Paused while a bank transfer is pending -- this resumes automatically." next to a "Pause reminders" toggle that remains tap-able (there is no Resume button in this state).

**FEAT-11.SPEC-003-AC-05:** Given the invoice is eligible for a manual reminder, when Nadia taps "Send Reminder Now", then the reminder is sent, a new "Manual" row appears in the history, and she sees "Reminder sent to Owen." (or the Primary Contact's actual name).

**FEAT-11.SPEC-003-AC-06:** Given Nadia already sent a manual reminder for this invoice earlier today, when she taps "Send Reminder Now" again, then she sees the inline message "You've already sent a reminder for this invoice today. You can send another tomorrow." and no history row is added.

**FEAT-11.SPEC-003-AC-07:** Given Owen (Client Primary Contact) is viewing his own portal, when he looks for any way to reach this panel, then no such path exists anywhere in his portal.

**FEAT-11.SPEC-003-AC-08:** Given Dana (Support Operator) has an open support session on the freelancer's account, when she opens this panel, then she sees the full history and pause state, but every control (Pause, Resume, Send Reminder Now) is disabled with "unavailable in a read-only support session."

**FEAT-11.SPEC-003-AC-09:** Given the reminder-history data fails to load, when the panel attempts its initial load, then Nadia sees "Couldn't load reminder history. Try again." with a Retry control, and all action controls are disabled until the retry succeeds.

**FEAT-11.SPEC-003-AC-10:** Given Nadia loses connectivity while the panel is open, when she attempts to tap Pause, then the offline banner is shown and the action is not attempted until connectivity returns.

**FEAT-11.SPEC-003-AC-11:** Given Nadia taps "Send Reminder Now" twice in rapid succession, when the first tap is still processing, then the second tap has no effect and the button remains in its loading state.

**FEAT-11.SPEC-003-AC-12:** Given Nadia's session expires while a Pause action is mid-flight, when the expiry is detected, then the dialog "Your session has expired. Sign in to continue." appears and the pause action is discarded, requiring her to retry after re-authenticating.

**FEAT-11.SPEC-003-AC-13:** Given two of Nadia's own sessions both tap "Send Reminder Now" for the same invoice at effectively the same time, when both requests are processed, then only one manual send succeeds and the other is denied with the one-per-day message, with its panel refreshing to show the new entry.

**FEAT-11.SPEC-003-AC-14:** Given the invoice becomes Paid while Nadia is viewing the panel, when she next taps "Send Reminder Now" (without having refreshed), then the send attempt is denied per FEAT-11.SPEC-002 rather than silently succeeding, since eligibility is re-checked at the moment of the tap.

**FEAT-11.SPEC-003-AC-15:** Given Priya (Client Reviewer Contact) is viewing her own portal, when she looks for any way to reach this panel or receive a reminder-related communication, then neither exists anywhere in her portal.

**FEAT-11.SPEC-003-AC-16:** Given the invoice's `pause_state` is Paused while bank transfer pending, when Nadia taps the "Pause reminders" toggle, then `pause_state` becomes Paused by freelancer (overwriting the automatic pause reason, per FEAT-11.SPEC-002-AC-10), the static "Paused while a bank transfer is pending" text is removed, the indicator switches to "Paused by you", the toggle switches to "Resume reminders", and she sees the toast "Reminders paused for this invoice."

### User Story 4 - Overdue Reminder Email (Priority: P1)

Sends the client's Primary Contact a polite email about an overdue invoice, on the day-3 automatic reminder, the day-10 automatic reminder, or Nadia's manual reminder -- with a direct way to pay.

**Acceptance Scenarios:**

**FEAT-11.SPEC-004-AC-01:** Given Owen's invoice reaches day 3 overdue and eligibility passes, when FEAT-11.SPEC-001 fires this notification, then Owen receives an email with subject "Reminder: Invoice {invoice_number} from {freelancer_business_name} is now overdue" and a "Pay Invoice {invoice_number}" button.

**FEAT-11.SPEC-004-AC-02:** Given Owen's invoice reaches day 10 overdue and eligibility passes, when the notification fires, then Owen receives the day-10 email with subject "Second reminder: Invoice {invoice_number} from {freelancer_business_name} is still overdue" naming {days_overdue} as 10.

**FEAT-11.SPEC-004-AC-03:** Given Nadia sends a manual reminder and eligibility passes, when the notification fires, then Owen receives the manual variant with subject "Reminder: Invoice {invoice_number} from {freelancer_business_name} is due".

**FEAT-11.SPEC-004-AC-04:** Given Owen taps "Pay Invoice {invoice_number}" in any variant, when the link is followed, then he lands on FEAT-10.SPEC-001 for that specific invoice.

**FEAT-11.SPEC-004-AC-05:** Given Nadia has no connected, ready payment account for this invoice, when any reminder variant is sent, then the CTA is replaced with the plain-text pay-the-freelancer-directly instructions from FEAT-09.SPEC-009.

**FEAT-11.SPEC-004-AC-06:** Given the email fails to deliver on the first attempt, when the delivery capability retries, then up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the affected project for Nadia.

**FEAT-11.SPEC-004-AC-07:** Given the invoice's Primary Contact was removed via an erasure request between trigger and delivery, when the send is attempted, then it is cancelled, logged as delivery-skipped-no-recipient, and surfaced to Nadia as a delivery warning.

**FEAT-11.SPEC-004-AC-08:** Given Priya (Client Reviewer Contact) is a contact on the same client company, when any reminder variant is sent, then Priya is never a recipient on any copy of the email.

**FEAT-11.SPEC-004-AC-09:** Given Owen has no notification preferences that could disable this email, when any reminder is triggered, then it always sends -- there is no opt-out control anywhere in Owen's portal for this email.

**FEAT-11.SPEC-004-AC-10:** Given a day-3 reminder for one invoice and a day-10 reminder for a different invoice both become eligible for the same recipient on the same day, when both fire, then Owen receives two separate emails, each naming its own invoice number -- never a combined email.

**FEAT-11.SPEC-004-AC-11:** Given the Reminder Schedule re-evaluates an invoice whose day-3 Reminder Log entry already has a `sent_at` value, when the re-evaluation runs, then no second day-3 email is sent for that invoice.

**FEAT-11.SPEC-004-AC-12:** Given Dana (Support Operator) has an open support session on the freelancer's account, when she views the invoice's reminder history, then she can see this email's delivery status read-only, and never receives a copy of the email herself.

### User Story 5 - Bank-Transfer-Pending Reminder Pause (Priority: P1)

Automatically pauses an invoice's reminder schedule while a bank-transfer payment is pending, and resumes it if the pending transfer resolves or reverts, so a reminder never chases a payment that is already on its way.

**Acceptance Scenarios:**

**FEAT-11.SPEC-005-AC-01:** Given Nadia's invoice to Owen has an Active reminder schedule, when a bank-transfer payment for that invoice enters Payment pending, then `pause_state` becomes Paused while bank transfer pending.

**FEAT-11.SPEC-005-AC-02:** Given the invoice's `pause_state` is Paused while bank transfer pending, when the transfer resolves to Paid, then `pause_state` returns to Active.

**FEAT-11.SPEC-005-AC-03:** Given the invoice's `pause_state` is Paused while bank transfer pending, when the transfer fails and the invoice reverts to unpaid, then `pause_state` returns to Active and the invoice becomes eligible for its next due reminder.

**FEAT-11.SPEC-005-AC-04:** Given Nadia has already paused the invoice herself (Paused by freelancer), when a bank-transfer payment for it enters Payment pending, then `pause_state` remains Paused by freelancer, unchanged.

**FEAT-11.SPEC-005-AC-05:** Given the invoice's `pause_state` is Paused by freelancer, when the pending transfer that triggered no automatic pause resolves, then `pause_state` remains Paused by freelancer -- the resolve event never touches it.

**FEAT-11.SPEC-005-AC-06:** Given the invoice's `pause_state` is already Paused while bank transfer pending, when a duplicate Payment pending event is reported for the same invoice, then `pause_state` remains unchanged and no error occurs.

**FEAT-11.SPEC-005-AC-07:** Given the invoice's `pause_state` is already Active, when a duplicate resolve event is reported, then `pause_state` remains unchanged.

**FEAT-11.SPEC-005-AC-08:** Given a Payment pending event and a resolve event for the same invoice arrive at effectively the same time, when both are processed, then the resulting `pause_state` reflects whichever event's write committed first, with the second event's logic applied against that resulting state rather than a stale read.

**FEAT-11.SPEC-005-AC-09:** Given the write to `pause_state` fails on the first attempt, when the automation retries and the retry succeeds, then `pause_state` reflects the correct value with no user-visible interruption.

**FEAT-11.SPEC-005-AC-10:** Given a pending bank transfer for an invoice already past day 10 reverts to unpaid, when `pause_state` returns to Active, then no automatic reminder is scheduled (per FEAT-11.SPEC-001's day-10 ceiling), and only Nadia's manual send remains available.

### Edge Cases

- **FEAT-11.SPEC-001 (Reminder Schedule):** An invoice paid between candidate detection and the eligibility re-check is skipped by the re-check, and elapsed days use the freelancer's local calendar, so a daylight-saving transition day counts as one day. After a missed evaluation the next run catches up on thresholds already passed, but a day-3 window that has passed is not retried while day 10 remains pending. Source: `docs/blueprint/specifications/FEAT-11-automated-payment-reminders/FEAT-11.SPEC-001-reminder-schedule.md` (section: Edge Cases)
- **FEAT-11.SPEC-002 (Reminder Eligibility Rule):** A manual send against an invoice that just became Paid is denied by the current-status check, and when both pause reasons apply the freelancer's pause wins over the bank-transfer pause. The one-reminder-per-day rule uses the freelancer's local calendar day (sends on either side of midnight are different days), and daylight-saving days still count as one elapsed day. Source: `docs/blueprint/specifications/FEAT-11-automated-payment-reminders/FEAT-11.SPEC-002-reminder-eligibility-rule.md` (section: Edge Cases)
- **FEAT-11.SPEC-003 (Invoice Reminder Panel):** Double taps on Send Reminder Now are ignored, and the panel is a snapshot so an automatic reminder sent while it is open appears on the next visit or refresh. A pause that commits before the schedule's eligibility re-check wins, and two sessions sending a manual reminder at once resolve by reject-with-refresh. Source: `docs/blueprint/specifications/FEAT-11-automated-payment-reminders/FEAT-11.SPEC-003-invoice-reminder-panel.md` (section: Edge Cases)
- **FEAT-11.SPEC-004 (Overdue Reminder Email):** A reminder arriving moments after payment is accepted as a rare artifact once handed to delivery, and a Primary Contact removed before delivery cancels the send as delivery-skipped-no-recipient. The email has no preference control or quiet hours, and it renders whichever pay-link state is current at send. Source: `docs/blueprint/specifications/FEAT-11-automated-payment-reminders/FEAT-11.SPEC-004-overdue-reminder-email.md` (section: Edge Cases)
- **FEAT-11.SPEC-005 (Bank-Transfer-Pending Reminder Pause):** The most recently reported pending or resolve event is trusted when events arrive out of order, a freelancer pause takes precedence over a later resolve event, and concurrent events resolve to whichever write commits first. Pause-state writes for one invoice are serialized. Source: `docs/blueprint/specifications/FEAT-11-automated-payment-reminders/FEAT-11.SPEC-005-bank-transfer-pending-reminder-pause.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-11.SPEC-001** (Reminder Schedule) as specified: Automatically sends the day-3 and day-10 overdue reminder email for an unpaid invoice, re-checking eligibility immediately before each send so a reminder never goes out to an invoice that has since been paid or paused. Full spec: `docs/blueprint/specifications/FEAT-11-automated-payment-reminders/FEAT-11.SPEC-001-reminder-schedule.md`
- **FR-002**: The system MUST implement **FEAT-11.SPEC-002** (Reminder Eligibility Rule) as specified: Defines whether a reminder -- automatic or manual -- may send right now: the paid check, the pause checks, the time-zone day-counting basis, and the one-manual-reminder-per-day limit, plus every role's authority over pausing, resuming, and manually sending. Full spec: `docs/blueprint/specifications/FEAT-11-automated-payment-reminders/FEAT-11.SPEC-002-reminder-eligibility-rule.md`
- **FR-003**: The system MUST implement **FEAT-11.SPEC-003** (Invoice Reminder Panel) as specified: Shows an invoice's reminder history and lets Nadia pause or resume the automatic reminder schedule and send a manual reminder, all scoped to one invoice. Full spec: `docs/blueprint/specifications/FEAT-11-automated-payment-reminders/FEAT-11.SPEC-003-invoice-reminder-panel.md`
- **FR-004**: The system MUST implement **FEAT-11.SPEC-004** (Overdue Reminder Email) as specified: Sends the client's Primary Contact a polite email about an overdue invoice, on the day-3 automatic reminder, the day-10 automatic reminder, or Nadia's manual reminder -- with a direct way to pay. Full spec: `docs/blueprint/specifications/FEAT-11-automated-payment-reminders/FEAT-11.SPEC-004-overdue-reminder-email.md`
- **FR-005**: The system MUST implement **FEAT-11.SPEC-005** (Bank-Transfer-Pending Reminder Pause) as specified: Automatically pauses an invoice's reminder schedule while a bank-transfer payment is pending, and resumes it if the pending transfer resolves or reverts, so a reminder never chases a payment that is already on its way. Full spec: `docs/blueprint/specifications/FEAT-11-automated-payment-reminders/FEAT-11.SPEC-005-bank-transfer-pending-reminder-pause.md`

### Key Entities

- Invoice (read)
- Reminder Log (create)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: At least 50% of invoices that go overdue are paid after an automated reminder and before any manual follow-up from the freelancer (metric: Reminder-Driven Payment Recovery). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Reminders sent (day 3 and day 10), pauses, resumes and manual reminders are each observable as distinct signals (reminder_sent, reminder_paused, reminder_resumed, manual_reminder_sent). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-08**: Freelancers are assumed to prefer fixed day-3 and day-10 reminders over configuring their own workflows. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-06**: Client Primary Contacts are assumed to act on payment requests within days once reminded. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-19**: Time zones are never hard-coded, which governs the elapsed-day count. Full register: `docs/blueprint/features/assumptions-constraints.md`
