# Feature Specification: Proposal Acceptance

**Blueprint feature:** FEAT-03
**Priority tier:** Core
**Build order:** 004 of 33
**Depends on:** FEAT-02
**Blueprint source:** `docs/blueprint/specifications/FEAT-03-proposal-acceptance/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Proposal Review & Accept (Priority: P1)

Owen, the client's Primary Contact, reads a sent proposal's scope and price and either accepts it (recording a timestamped, permanent acceptance) or opens Request Changes to send Nadia a note instead.

**Acceptance Scenarios:**

**FEAT-03.SPEC-001-AC-01:** Given Owen opens the proposal link from his portal home, when the screen loads, then the scope description and price render fully before the Accept and Request Changes controls become active.

**FEAT-03.SPEC-001-AC-02:** Given Owen is on this screen with the proposal in Sent status, when he taps Accept, then the acceptance is recorded and the Accept and Request Changes controls are replaced by an "Accepted on {date}" marker.

**FEAT-03.SPEC-001-AC-03:** Given Owen is on this screen, when he taps Request Changes, then he is navigated to FEAT-03.SPEC-002 (Request Changes).

**FEAT-03.SPEC-001-AC-04:** Given Owen is on a proposal already showing "Accepted on {date}", when he views the screen, then no Accept or Request Changes control is shown.

**FEAT-03.SPEC-001-AC-05:** Given Owen taps Accept on a proposal that was already accepted by another Primary contact moments earlier, when the eligibility check runs, then the screen shows "This proposal has already been accepted" and no duplicate acceptance is recorded.

**FEAT-03.SPEC-001-AC-06:** Given Owen opens a proposal link for a version that Nadia has since edited and re-sent, when the screen loads, then he is redirected to the current version of the proposal.

**FEAT-03.SPEC-001-AC-07:** Given Owen taps Accept and loses connectivity mid-flight, when connectivity drops, then the screen shows an inline error message with a retry option and no acceptance is recorded until a retry succeeds.

**FEAT-03.SPEC-001-AC-08:** Given Owen loses connectivity while viewing the screen, when he attempts to tap Accept, then the controls are visibly disabled with the message "Reconnect to accept."

**FEAT-03.SPEC-001-AC-09:** Given Priya (Reviewer) follows a proposal link meant for Owen, when her portal home loads, then she sees only the project's stage label and no scope, price, Accept, or Request Changes control.

**FEAT-03.SPEC-001-AC-10:** Given Dana (Support Operator) opens this screen inside a logged support session, when the screen loads, then the full proposal content renders but no Accept or Request Changes control is shown.

**FEAT-03.SPEC-001-AC-11:** Given an unauthenticated visitor opens the proposal link, when the screen would otherwise load, then they are redirected to the client portal's magic-link sign-in.

**FEAT-03.SPEC-001-AC-12:** Given Owen's sign-in session has expired, when he opens the proposal link, then he sees a plain explanation and a "request a fresh link" option, with no proposal content shown.

**FEAT-03.SPEC-001-AC-13:** Given Owen is on the screen in the Ready-to-decide state, when he views the layout at a compact breakpoint, then the Accept and Request Changes controls are stacked vertically, Accept above Request Changes.

**FEAT-03.SPEC-001-AC-14:** Given Owen successfully accepts the proposal, when the acceptance is recorded, then a proposal_accepted analytics event is emitted with the proposal reference and time elapsed since sent_at.

**FEAT-03.SPEC-001-AC-15:** Given an out-of-scope contact opens a proposal link that does not belong to their own company, when the screen would otherwise load, then they see a plain explanation and a fresh-link option, never another company's proposal data.

### User Story 2 - Request Changes (Priority: P1)

Owen composes and sends a short change-request note to Nadia instead of accepting the proposal.

**Acceptance Scenarios:**

**FEAT-03.SPEC-002-AC-01:** Given Owen is on the Proposal Review & Accept screen, when he taps Request Changes, then he is navigated to this screen with an empty note field.

**FEAT-03.SPEC-002-AC-02:** Given Owen is on this screen, when he types a note and taps Send, then the note is recorded as a Comment on the proposal and Nadia is notified, and Owen sees the confirmation "Your note has been sent to Nadia" before returning to FEAT-03.SPEC-001.

**FEAT-03.SPEC-002-AC-03:** Given Owen is on this screen with an empty note field, when he taps Send, then the note field shows the error "Enter a note before sending." and no note is recorded.

**FEAT-03.SPEC-002-AC-04:** Given Owen has typed 2,000 characters into the note field, when he attempts to type further, then no additional characters are accepted and the character count shows "2,000 / 2,000".

**FEAT-03.SPEC-002-AC-05:** Given Owen has typed a note and not yet sent it, when he taps the back arrow, then a confirmation dialog "Discard this note?" appears with "Discard" and "Keep Editing" options.

**FEAT-03.SPEC-002-AC-06:** Given Owen taps Send and loses connectivity mid-flight, when the send fails, then an inline error banner appears with a Retry option and the note text is preserved.

**FEAT-03.SPEC-002-AC-07:** Given Owen loses connectivity while composing his note, when he attempts to tap Send, then the banner "You're offline -- your note will be sent when you reconnect." appears and the note is submitted automatically once connectivity returns.

**FEAT-03.SPEC-002-AC-08:** Given the proposal Owen is viewing is voided by Nadia editing and re-sending it while he is composing his note, when he taps Send, then the send is rejected and Owen is directed to the current version, with his note text preserved.

**FEAT-03.SPEC-002-AC-09:** Given the proposal Owen is viewing is accepted through another session while he is composing his note, when he taps Send, then the send is rejected with a message that the proposal has already been accepted.

**FEAT-03.SPEC-002-AC-10:** Given Priya (Reviewer) attempts to reach this screen, when she follows any link toward it, then she cannot reach it and her portal home shows only the project's stage label.

**FEAT-03.SPEC-002-AC-11:** Given an unauthenticated visitor attempts to open this screen, when the screen would otherwise load, then they are redirected to the client portal's magic-link sign-in.

**FEAT-03.SPEC-002-AC-12:** Given Owen successfully sends a change-request note, when the note is recorded, then a proposal_changes_requested analytics event is emitted with the proposal reference and note character count.

### User Story 3 - Acceptance Recording (Priority: P1)

Writes the immutable acceptance record on the Proposal when Owen accepts, and fires the downstream deposit-invoice and audit-trail effects.

**Acceptance Scenarios:**

**FEAT-03.SPEC-003-AC-01:** Given Owen taps Accept on a proposal in Sent status with a Payment Schedule that includes no deposit, when this automation runs, then the Proposal's `status` becomes Accepted with `accepted_at` and `accepted_by` set, and no invoice trigger fires.

**FEAT-03.SPEC-003-AC-02:** Given Owen taps Accept on a proposal in Sent status with a Payment Schedule that includes a deposit, when this automation runs, then the acceptance is recorded and the deposit-invoice trigger fires to FEAT-09 (XBR-01), using the schedule as it stood at that moment.

**FEAT-03.SPEC-003-AC-03:** Given the acceptance write succeeds, when this automation completes, then it fires FEAT-03.SPEC-006 to notify Owen and Nadia, and fires the FEAT-13 audit-trail entry for the acceptance event.

**FEAT-03.SPEC-003-AC-04:** Given two Primary contacts at the same client both tap Accept on the same proposal at effectively the same moment, when this automation's write-time re-check runs for each, then exactly one acceptance is recorded and the other invocation returns "already accepted."

**FEAT-03.SPEC-003-AC-05:** Given Owen taps Accept on a proposal that Nadia voids by editing and re-sending before this automation's write-time re-check runs, when the re-check finds `status: Voided`, then no acceptance is recorded and FEAT-03.SPEC-001 redirects Owen to the current version.

**FEAT-03.SPEC-003-AC-06:** Given Owen taps Accept and connectivity drops before the write completes, when the failure occurs, then no partial acceptance record is left and FEAT-03.SPEC-001 shows a retry option.

**FEAT-03.SPEC-003-AC-07:** Given Owen retries Accept after a failed write, when this automation re-runs, then it re-checks eligibility from the current Proposal state rather than assuming the prior attempt's context still holds.

**FEAT-03.SPEC-003-AC-08:** Given Nadia adjusts the Payment Schedule at the exact moment Owen's acceptance is being written, when this automation reads the schedule in step 6, then it uses the schedule as it stood at the moment of acceptance, and Nadia's adjustment applies only to later triggers.

**FEAT-03.SPEC-003-AC-09:** Given the acceptance write succeeds but the deposit-invoice trigger to FEAT-09 fails to fire, when this automation completes, then the acceptance record remains valid and unaffected, and the invoice trigger is retried by FEAT-09's own handling.

**FEAT-03.SPEC-003-AC-10:** Given the acceptance is recorded, when the analytics signal is emitted, then a proposal_accepted event carries the proposal reference, whether a deposit invoice was triggered, and the time elapsed since the proposal was sent.

### User Story 4 - Change-Request Recording (Priority: P1)

Writes Owen's change-request note as a Comment on the proposal and notifies Nadia immediately.

**Acceptance Scenarios:**

**FEAT-03.SPEC-004-AC-01:** Given Owen submits a valid note (1--2,000 characters) on a proposal in Sent status, when this automation runs, then a Comment is created with `target: Proposal`, the note text, Owen as author, and the current timestamp.

**FEAT-03.SPEC-004-AC-02:** Given the Comment write succeeds, when this automation completes, then it fires FEAT-03.SPEC-007 to notify Nadia and fires the FEAT-13 audit-trail entry for the change-request event.

**FEAT-03.SPEC-004-AC-03:** Given a note somehow reaches this automation exceeding 2,000 characters, when the write-time length re-check runs, then no Comment is created and the "invalid note" outcome is returned.

**FEAT-03.SPEC-004-AC-04:** Given Owen submits a note on a proposal that Nadia voids by editing and re-sending before this automation's write-time re-check runs, when the re-check finds `status: Voided`, then no Comment is recorded and FEAT-03.SPEC-002 directs Owen to the current version.

**FEAT-03.SPEC-004-AC-05:** Given Owen submits a note on a proposal that is accepted (by himself in another session, or by another Primary contact) before this automation's write-time re-check runs, when the re-check finds `status: Accepted`, then no Comment is recorded and FEAT-03.SPEC-002 shows that the proposal has already been accepted.

**FEAT-03.SPEC-004-AC-06:** Given Owen submits a note and connectivity drops before the write completes, when the failure occurs, then no Comment is created and FEAT-03.SPEC-002 shows a retry option with the note text preserved.

**FEAT-03.SPEC-004-AC-07:** Given Owen submits two change-request notes on the same proposal in quick succession, when both reach this automation, then each is recorded as its own independent Comment with no conflict, ordered by posted time.

**FEAT-03.SPEC-004-AC-08:** Given a change-request note is successfully recorded, when the analytics signal is emitted, then a proposal_changes_requested event carries the proposal reference and the note's character count.

### User Story 5 - Acceptance & Access Rules (Priority: P1)

Governs who may accept a proposal or request changes on it, enforces single-acceptance and voided-proposal eligibility, defines the change-request note's length rule, and resolves concurrent-action conflicts.

**Acceptance Scenarios:**

**FEAT-03.SPEC-005-AC-01:** Given Owen is viewing a proposal with `status: Sent`, when he taps Accept, then the eligibility check passes and the accept proceeds to FEAT-03.SPEC-003.

**FEAT-03.SPEC-005-AC-02:** Given a proposal already has `status: Accepted`, when Owen (or any Primary contact) attempts to accept it again, then the attempt is denied with "This proposal has already been accepted."

**FEAT-03.SPEC-005-AC-03:** Given a proposal has `status: Voided`, when Owen attempts to accept it, then the attempt is denied and he is redirected to the current proposal version, with no error text shown.

**FEAT-03.SPEC-005-AC-04:** Given Owen submits a change-request note of exactly 2,000 characters, when the length rule is checked, then the note passes validation.

**FEAT-03.SPEC-005-AC-05:** Given Owen submits a change-request note of exactly 1 character, when the length rule is checked, then the note passes validation.

**FEAT-03.SPEC-005-AC-06:** Given Owen submits an empty change-request note, when the length rule is checked, then the note is denied with "Enter a note before sending."

**FEAT-03.SPEC-005-AC-07:** Given a proposal has `status: Voided`, when Owen attempts to submit a change-request note, then the attempt is denied and he is directed to the current proposal version.

**FEAT-03.SPEC-005-AC-08:** Given a proposal has `status: Accepted`, when Owen attempts to submit a change-request note, then the attempt is denied with "This proposal has already been accepted."

**FEAT-03.SPEC-005-AC-09:** Given Owen (Client Primary Contact) at the owning client, when he attempts to view the proposal, then he can view the full screen.

**FEAT-03.SPEC-005-AC-10:** Given Priya (Client Reviewer Contact), when she attempts to view the proposal, then no proposal content is shown and her portal home shows only the project's stage label.

**FEAT-03.SPEC-005-AC-11:** Given Nadia (Freelancer), when she attempts to open this feature's proposal review screen link, then the link is treated as out-of-scope and she sees a plain explanation with a fresh-link option, never proposal content through this feature's own screen.

**FEAT-03.SPEC-005-AC-12:** Given Dana (Support Operator) inside a logged support session, when she views the proposal, then she sees the full content read-only, with no Accept or Request Changes control rendered.

**FEAT-03.SPEC-005-AC-13:** Given two Primary contacts at the same client both attempt Accept at effectively the same moment, when the write-time eligibility check runs for each, then exactly one succeeds and the other is denied with "This proposal has already been accepted."

**FEAT-03.SPEC-005-AC-14:** Given a Primary contact's status is changed to Removed while they are mid-session on the proposal screen, when they attempt to accept, then the action is denied with the same experience as a contact who never had access.

**FEAT-03.SPEC-005-AC-15:** Given a proposal transitions from Sent to Voided between the screen's own eligibility check and the write-time re-check, when FEAT-03.SPEC-003 re-checks eligibility, then the voided-cannot-accept rule denies the write and the client is redirected to the current version.

**FEAT-03.SPEC-005-AC-16:** Given a contact attempts to reach a proposal belonging to a different client than their own, when the access check runs, then the attempt is denied as out-of-scope with a plain explanation and a fresh-link option, never another company's data.

**FEAT-03.SPEC-005-AC-17:** Given the acceptance write succeeds for one Primary contact's attempt, when `accepted_at` and `accepted_by` are derived, then they are set exactly once from that attempt's timestamp and contact reference, with no user override available.

### User Story 6 - Acceptance Confirmation Notification (Priority: P1)

Emails Owen and Nadia a confirmation the instant a proposal's acceptance is recorded, giving both parties a durable, timestamped record that the project is now real and billable.

**Acceptance Scenarios:**

**FEAT-03.SPEC-006-AC-01:** Given Owen accepts a proposal with no deposit due, when FEAT-03.SPEC-003 records the acceptance, then Owen and Nadia each receive an email confirmation, and both bodies include "No deposit is due under this project's payment schedule" (Owen's variant) / "no invoice was generated yet" (Nadia's variant).

**FEAT-03.SPEC-006-AC-02:** Given Owen accepts a proposal whose Payment Schedule includes a deposit, when FEAT-03.SPEC-003 records the acceptance and triggers the deposit invoice, then Owen's confirmation email states "A deposit invoice has been sent to you separately" and Nadia's states "A deposit invoice has been generated and sent automatically."

**FEAT-03.SPEC-006-AC-03:** Given Owen receives the confirmation email, when he taps "View proposal", then he lands on FEAT-03.SPEC-001 showing "Accepted on {accepted_at_date}".

**FEAT-03.SPEC-006-AC-04:** Given Nadia receives the confirmation email, when she taps "View project", then she lands on the project view in FEAT-01.

**FEAT-03.SPEC-006-AC-05:** Given the acceptance is recorded, when this notification's trigger fires, then no on/off preference is available to either recipient to suppress it -- it always sends.

**FEAT-03.SPEC-006-AC-06:** Given Owen's email address bounces on the first delivery attempt, when the delivery capability retries, then up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window` before Nadia sees a delivery warning on the project.

**FEAT-03.SPEC-006-AC-07:** Given all retries for Owen's email are exhausted, when the final failure occurs, then Nadia sees a delivery warning on the affected project and the "Accepted on {date}" marker remains the enduring in-product record regardless.

**FEAT-03.SPEC-006-AC-08:** Given the acceptance is recorded exactly once (per FEAT-03.SPEC-003's accept-once guarantee), when a second, redundant Accept attempt returns "already accepted", then this notification's trigger does not fire a second time.

**FEAT-03.SPEC-006-AC-09:** Given Owen's Client Contact record is later removed, when this notification is still pending delivery, then it still delivers to the email address captured in `accepted_by` at the moment of acceptance.

**FEAT-03.SPEC-006-AC-10:** Given the acceptance confirmation is successfully delivered to both recipients, when the analytics signal is emitted, then an acceptance_confirmation_sent event is recorded once per recipient with the deposit-invoice-triggered flag.

### User Story 7 - Change-Request Notification (Priority: P1)

Emails Nadia immediately when Owen submits a change-request note, so she can revise and re-send the proposal without a separate status check.

**Acceptance Scenarios:**

**FEAT-03.SPEC-007-AC-01:** Given Owen submits a valid change-request note, when FEAT-03.SPEC-004 records it, then Nadia receives an email with the subject naming the client company and project, quoting the note text in the body.

**FEAT-03.SPEC-007-AC-02:** Given Nadia receives the change-request email, when she taps "Open proposal", then she lands on the proposal detail in FEAT-02 with the note visible for her to act on.

**FEAT-03.SPEC-007-AC-03:** Given the change-request note is recorded, when this notification's trigger fires, then no on/off preference is available to Nadia to suppress it -- it always sends.

**FEAT-03.SPEC-007-AC-04:** Given the change-request note is recorded during Nadia's local nighttime hours, when this notification's trigger fires, then it sends immediately with no quiet-hours hold.

**FEAT-03.SPEC-007-AC-05:** Given Owen's change-request submission is rejected because the proposal was already Accepted (FEAT-03.SPEC-004's outcome), when no Comment is created, then this notification's trigger never fires.

**FEAT-03.SPEC-007-AC-06:** Given Nadia's email address bounces on the first delivery attempt, when the delivery capability retries, then up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the affected project.

**FEAT-03.SPEC-007-AC-07:** Given all retries for Nadia's email are exhausted, when the final failure occurs, then a delivery warning appears on the affected project and the Comment remains visible to Nadia inside the product regardless.

**FEAT-03.SPEC-007-AC-08:** Given Owen submits two change-request notes on the same proposal in succession, when each is recorded, then Nadia receives two separate emails, one per note, never merged into one.

**FEAT-03.SPEC-007-AC-09:** Given the change-request notification is successfully delivered, when the analytics signal is emitted, then a change_request_notification_sent event is recorded with the note's character count.

### Edge Cases

- **FEAT-03.SPEC-001 (Proposal Review & Accept):** A repeat Accept or an Accept against a version voided in the meantime is rejected-with-refresh, and when two Primary contacts accept at once only one acceptance is recorded. A connectivity drop after the tap retries without duplicating the acceptance, and navigating away simply re-fetches the current proposal state. Source: `docs/blueprint/specifications/FEAT-03-proposal-acceptance/FEAT-03.SPEC-001-proposal-review-accept.md` (section: Edge Cases)
- **FEAT-03.SPEC-002 (Request Changes):** Navigating back with a non-empty note triggers a Discard confirmation, double taps on Send are ignored, and a network failure preserves the note text with a retry banner. If the proposal is voided while the note is being composed, Send is rejected and the contact is directed to the current version. Source: `docs/blueprint/specifications/FEAT-03-proposal-acceptance/FEAT-03.SPEC-002-request-changes.md` (section: Edge Cases)
- **FEAT-03.SPEC-003 (Acceptance Recording):** The atomic write admits only one acceptance when two Primary contacts accept simultaneously, and the write-time re-check (not the screen-time check) is authoritative if the proposal is voided and re-sent in between. A payment schedule being adjusted at the moment of acceptance is read as it stood at acceptance, and in-flight duplicate runs are guarded by the disabled Accept control. Source: `docs/blueprint/specifications/FEAT-03-proposal-acceptance/FEAT-03.SPEC-003-acceptance-recording.md` (section: Edge Cases)
- **FEAT-03.SPEC-004 (Change-Request Recording):** The write-time re-check catches an over-length note, a proposal voided or already accepted between Send and the write, each returning its own outcome with no Comment created. A double-tap on Send is debounced so one note produces one recording run. Source: `docs/blueprint/specifications/FEAT-03-proposal-acceptance/FEAT-03.SPEC-004-change-request-recording.md` (section: Edge Cases)
- **FEAT-03.SPEC-005 (Acceptance & Access Rules):** Boundary values are inclusive: a change-request note of 1 or 2,000 characters passes, while an empty or whitespace-only note fails with the enter-a-note message. Accepting a proposal in Sent status with accepted_at and accepted_by unset is the standard eligible path. Source: `docs/blueprint/specifications/FEAT-03-proposal-acceptance/FEAT-03.SPEC-005-acceptance-access-rules.md` (section: Edge Cases)
- **FEAT-03.SPEC-006 (Acceptance Confirmation Notification):** A bounced address is retried and then surfaces a delivery warning to the freelancer (XBR-30) without affecting the acceptance. The email still goes to the address captured in accepted_by even if the contact is removed, uses a no-deposit-due fallback line only when the schedule has no deposit, and falls back to the account name when no business name is set. Source: `docs/blueprint/specifications/FEAT-03-proposal-acceptance/FEAT-03.SPEC-006-acceptance-confirmation-notification.md` (section: Edge Cases)
- **FEAT-03.SPEC-007 (Change-Request Notification):** If the freelancer's own address bounces, the delivery warning still appears in the app. A second change-request note produces its own append-only Comment, the email sends even when the freelancer is viewing the proposal, and a race with an acceptance in another session resolves to one outcome per FEAT-03.SPEC-005. Source: `docs/blueprint/specifications/FEAT-03-proposal-acceptance/FEAT-03.SPEC-007-change-request-notification.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-03.SPEC-001** (Proposal Review & Accept) as specified: Owen, the client's Primary Contact, reads a sent proposal's scope and price and either accepts it (recording a timestamped, permanent acceptance) or opens Request Changes to send Nadia a note instead. Full spec: `docs/blueprint/specifications/FEAT-03-proposal-acceptance/FEAT-03.SPEC-001-proposal-review-accept.md`
- **FR-002**: The system MUST implement **FEAT-03.SPEC-002** (Request Changes) as specified: Owen composes and sends a short change-request note to Nadia instead of accepting the proposal. Full spec: `docs/blueprint/specifications/FEAT-03-proposal-acceptance/FEAT-03.SPEC-002-request-changes.md`
- **FR-003**: The system MUST implement **FEAT-03.SPEC-003** (Acceptance Recording) as specified: Writes the immutable acceptance record on the Proposal when Owen accepts, and fires the downstream deposit-invoice and audit-trail effects. Full spec: `docs/blueprint/specifications/FEAT-03-proposal-acceptance/FEAT-03.SPEC-003-acceptance-recording.md`
- **FR-004**: The system MUST implement **FEAT-03.SPEC-004** (Change-Request Recording) as specified: Writes Owen's change-request note as a Comment on the proposal and notifies Nadia immediately. Full spec: `docs/blueprint/specifications/FEAT-03-proposal-acceptance/FEAT-03.SPEC-004-change-request-recording.md`
- **FR-005**: The system MUST implement **FEAT-03.SPEC-005** (Acceptance & Access Rules) as specified: Governs who may accept a proposal or request changes on it, enforces single-acceptance and voided-proposal eligibility, defines the change-request note's length rule, and resolves concurrent-action conflicts. Full spec: `docs/blueprint/specifications/FEAT-03-proposal-acceptance/FEAT-03.SPEC-005-acceptance-access-rules.md`
- **FR-006**: The system MUST implement **FEAT-03.SPEC-006** (Acceptance Confirmation Notification) as specified: Emails Owen and Nadia a confirmation the instant a proposal's acceptance is recorded, giving both parties a durable, timestamped record that the project is now real and billable. Full spec: `docs/blueprint/specifications/FEAT-03-proposal-acceptance/FEAT-03.SPEC-006-acceptance-confirmation-notification.md`
- **FR-007**: The system MUST implement **FEAT-03.SPEC-007** (Change-Request Notification) as specified: Emails Nadia immediately when Owen submits a change-request note, so she can revise and re-send the proposal without a separate status check. Full spec: `docs/blueprint/specifications/FEAT-03-proposal-acceptance/FEAT-03.SPEC-007-change-request-notification.md`

### Key Entities

- Proposal (update: accepted state)
- Invoice (create: deposit invoice)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: At least 70% of accepted proposals are accepted within 24 hours of being sent (metric: Time to Proposal Acceptance). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Proposal views, acceptances, auto-generated deposit invoices and change requests are each observable as distinct signals (proposal_viewed_by_client, proposal_accepted, deposit_invoice_auto_generated, proposal_changes_requested). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-02**: Client contacts review and act primarily from mobile browsers. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-11**: A recorded, timestamped Accept is assumed enough evidence for most scope disputes without a legally binding e-signature. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-15**: Accepted proposals are append-only and immutable once created. Full register: `docs/blueprint/features/assumptions-constraints.md`
