# Feature Specification: Milestone Approval

**Blueprint feature:** FEAT-08
**Priority tier:** Core
**Build order:** 009 of 33
**Depends on:** FEAT-06, FEAT-07
**Blueprint source:** `docs/blueprint/specifications/FEAT-08-milestone-approval/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Milestone Review & Approval Screen (Priority: P1)

Owen reviews a milestone's current deliverable and comment thread and approves it with a single, plainly explained, timestamped action; Priya sees the identical status and content with no working Approve control.

**Acceptance Scenarios:**

**FEAT-08.SPEC-001-AC-01:** Given Owen navigates from FEAT-07.SPEC-001 to a milestone whose deliverable and comments have not yet finished loading, when the screen opens, then it shows the Loading state and the Approve control is disabled.

**FEAT-08.SPEC-001-AC-02:** Given Owen is on this screen for a milestone with status "Deliverable Uploaded," when loading completes, then the deliverable panel, comment thread, and an enabled Approve control with its consent line are all shown.

**FEAT-08.SPEC-001-AC-03:** Given Priya is on this screen for the same milestone, when loading completes, then she sees the identical deliverable and comment content, but no Approve control anywhere in the layout.

**FEAT-08.SPEC-001-AC-04:** Given Owen taps the enabled Approve control, when the tap registers, then the control disables, shows an in-progress state, and triggers FEAT-08.SPEC-003.

**FEAT-08.SPEC-001-AC-05:** Given FEAT-08.SPEC-003 returns a successful outcome, when the response arrives, then the Decision area is replaced by "Approved on {date}" with Owen's name, announced to assistive technology as a live update.

**FEAT-08.SPEC-001-AC-06:** Given FEAT-08.SPEC-003 returns a stale-attempt outcome because Nadia changed the milestone since load, when the response arrives, then the stale-refresh dialog appears and, once acknowledged, the screen reloads to the current deliverable, comments, and status.

**FEAT-08.SPEC-001-AC-07:** Given FEAT-08.SPEC-003 returns a write-failure outcome, when the response arrives, then an error banner with a Retry action appears above the Decision area, and the milestone's displayed status is unchanged.

**FEAT-08.SPEC-001-AC-08:** Given Owen loses connectivity while this screen is open, when the loss is detected, then the offline banner "You're offline. Reconnect to approve this milestone." appears, the Approve control disables, and no approval is queued.

**FEAT-08.SPEC-001-AC-09:** Given Owen regains connectivity after the offline banner appeared, when the screen confirms the milestone's current state is fresh, then the Approve control re-enables (if the milestone is still eligible).

**FEAT-08.SPEC-001-AC-10:** Given Owen taps Approve twice in rapid succession, when the first tap disables the control, then the second tap has no effect and only one request is sent.

**FEAT-08.SPEC-001-AC-11:** Given the milestone's status is already "Approved" when this screen first loads, when loading completes, then the "Approved on {date}" marker is shown immediately with no Decision area, for every role that can view the screen.

**FEAT-08.SPEC-001-AC-12:** Given Dana is in a logged, read-only support session viewing this milestone, when the screen loads, then she sees the full deliverable and comment content with every control shown disabled and labeled "Support access is read-only."

**FEAT-08.SPEC-001-AC-13:** Given an unauthenticated visitor follows a milestone link, when the link resolves, then they are redirected to FEAT-05's sign-in request page.

**FEAT-08.SPEC-001-AC-14:** Given a client contact's session has expired, when they attempt to open this screen, then they see FEAT-05's expired-link page with a fresh-link request option.

**FEAT-08.SPEC-001-AC-15:** Given the deliverable panel fails to load, when the failure occurs, then a load-error banner with Retry appears in its place, and the Approve control stays disabled until it succeeds.

**FEAT-08.SPEC-001-AC-16:** Given Owen taps "Reply in thread," when the tap registers, then he navigates to FEAT-07.SPEC-002 and this screen closes.

**FEAT-08.SPEC-001-AC-17:** Given Nadia reopens a milestone that Owen previously approved, when Owen reloads this screen, then he sees the Approve control again instead of the "Approved on {date}" marker.

**FEAT-08.SPEC-001-AC-18:** Given Priya follows a link to a milestone belonging to a different client company than her own, when the link resolves, then she sees a plain explanation and a fresh-link option, never that company's data.

**FEAT-08.SPEC-001-AC-19:** Given Owen approves a milestone whose live state matches what he was shown, when the approval succeeds, then the milestone_approval_confirmed_on_screen event fires with the elapsed time from tap to confirmation.

**FEAT-08.SPEC-001-AC-20:** Given two of Owen's own sessions both have this screen open for the same milestone and one approves it first, when the second session's Approve tap is processed, then FEAT-08.SPEC-003 refuses it as stale and this screen shows the stale-refresh dialog, resolving to the current Approved state -- the concurrent-edit conflict on the shared Milestone entity.

**FEAT-08.SPEC-001-AC-21:** Given Owen or Priya taps the Back control, when the tap registers, then they navigate to FEAT-05's portal home and this screen closes, with no data change of any kind.

**FEAT-08.SPEC-001-AC-22:** Given Owen or Priya taps the Deliverable panel's "Open" link, when the tap registers, then FEAT-06's viewing surface for the current deliverable opens (or the linked external asset opens in a new context), and this screen remains open behind it.

### User Story 2 - Milestone Reopen Screen (Priority: P1)

Nadia reopens an approved milestone through a deliberate, explained, logged confirmation -- never a silent status edit -- when genuinely necessary.

**Acceptance Scenarios:**

**FEAT-08.SPEC-002-AC-01:** Given Nadia opens this screen for a milestone whose approval details have not yet finished loading, when the screen opens, then it shows the Loading state and both controls are disabled.

**FEAT-08.SPEC-002-AC-02:** Given Nadia is on this screen for a milestone that is currently Approved, when loading completes, then she sees "Approved on {date} by {approving contact's name}," the explanation block, and enabled Reopen Milestone and Cancel controls.

**FEAT-08.SPEC-002-AC-03:** Given Nadia taps the enabled Reopen Milestone control, when the tap registers, then both controls disable, an in-progress state shows, and FEAT-08.SPEC-005 is triggered.

**FEAT-08.SPEC-002-AC-04:** Given FEAT-08.SPEC-005 returns a successful outcome, when the response arrives, then Nadia is navigated to the milestone's detail view, now showing "Reopened."

**FEAT-08.SPEC-002-AC-05:** Given FEAT-08.SPEC-005 returns an already-changed outcome, when the response arrives, then the acknowledgment message "This milestone's status has changed. Here's the current state." appears, and acknowledging it navigates to the milestone's current detail view.

**FEAT-08.SPEC-002-AC-06:** Given FEAT-08.SPEC-005 returns a write-failure outcome, when the response arrives, then an error banner with a Retry action appears, and the milestone's approval details remain unchanged.

**FEAT-08.SPEC-002-AC-07:** Given Nadia loses connectivity while this screen is open, when the loss is detected, then the offline banner "You're offline. Reconnect to reopen this milestone." appears and the Reopen control disables.

**FEAT-08.SPEC-002-AC-08:** Given Nadia regains connectivity after the offline banner appeared, when the screen confirms the milestone is still Approved, then the Reopen control re-enables.

**FEAT-08.SPEC-002-AC-09:** Given Nadia taps Cancel, when the tap registers, then no write occurs and she returns to the milestone's detail view unchanged.

**FEAT-08.SPEC-002-AC-10:** Given Nadia taps Reopen Milestone twice in rapid succession, when the first tap disables the controls, then the second tap has no effect and only one request is sent.

**FEAT-08.SPEC-002-AC-11:** Given Owen or Priya attempts to reach this screen through any path, when the attempt is made, then no such route exists in the client portal.

**FEAT-08.SPEC-002-AC-12:** Given Dana is in a support session, when she views this milestone, then she sees its status through FEAT-08.SPEC-001's read-only rendering, never through this screen.

**FEAT-08.SPEC-002-AC-13:** Given an unauthenticated visitor attempts to reach this screen, when the attempt is made, then they are redirected to Nadia's own sign-in.

**FEAT-08.SPEC-002-AC-14:** Given Nadia's session has expired, when she attempts to open this screen, then she is redirected to sign-in with a session-expired notice.

**FEAT-08.SPEC-002-AC-15:** Given the current approval details fail to load, when the failure occurs, then a load-error banner with Retry appears in place of the approval summary, and the Reopen control stays disabled.

**FEAT-08.SPEC-002-AC-16:** Given Nadia reaches this screen through a stale link for a milestone that is no longer Approved, when loading completes, then she sees the already-changed message immediately, with no confirm area shown.

**FEAT-08.SPEC-002-AC-17:** Given Nadia taps the Back control, when the tap registers, then no write occurs and she navigates to the milestone's detail view unchanged, identically to Cancel.

### User Story 3 - Approval Recording & Concurrency Guard (Priority: P1)

Records a milestone's approval exactly once, atomically writing the immutable timestamp and approving contact's identity, and refuses the attempt -- showing the refreshed milestone instead -- if the milestone changed underneath the approving contact since it was shown.

**Acceptance Scenarios:**

**FEAT-08.SPEC-003-AC-01:** Given Owen taps Approve on a milestone whose live state matches what his screen loaded, when the write commits, then `status`, `approved_at`, and `approved_by` are all set together and FEAT-08.SPEC-001 shows "Approved on {date}."

**FEAT-08.SPEC-003-AC-02:** Given a successful approval write, when it commits, then FEAT-08.SPEC-004 (Next-Invoice Trigger) and FEAT-08.SPEC-007 (Confirmation Notification) both fire.

**FEAT-08.SPEC-003-AC-03:** Given a successful approval write, when it commits, then an Activity Log entry is fired to FEAT-13 with event type "milestone approved," Owen as actor, and the Milestone as the affected record.

**FEAT-08.SPEC-003-AC-04:** Given Nadia removed the milestone's deliverable after Owen's screen loaded, when Owen taps Approve, then the attempt is refused as stale and Owen is shown the refreshed milestone with no write to `status`, `approved_at`, or `approved_by`.

**FEAT-08.SPEC-003-AC-05:** Given Nadia re-priced the milestone after Owen's screen loaded, with no status change, when Owen taps Approve, then the attempt is still refused as stale.

**FEAT-08.SPEC-003-AC-06:** Given the milestone is already "Approved" by the time Owen's request reaches this automation, when the comparison in step 6 runs, then the attempt is refused and Owen is shown the current "Approved on {date}" state.

**FEAT-08.SPEC-003-AC-07:** Given Owen has no connectivity at the moment he taps Approve, when the request is evaluated, then it is refused as a connectivity failure and no write is attempted.

**FEAT-08.SPEC-003-AC-08:** Given all eligibility checks pass but the write cannot be confirmed as committed, when Owen sees the resulting error, then he can retry, and the retry re-evaluates eligibility fresh rather than assuming the prior attempt partially succeeded.

**FEAT-08.SPEC-003-AC-09:** Given Owen taps Approve twice in rapid succession from the same session, when the first tap's write is already committing, then the second tap's request is refused as stale, showing the now-Approved milestone, and only one approval is ever recorded.

**FEAT-08.SPEC-003-AC-10:** Given Owen approves the same milestone from two of his own sessions at effectively the same time, when both requests are processed, then exactly one write commits and the other is refused as stale.

**FEAT-08.SPEC-003-AC-11:** Given a second Approve request arrives while a first request's write is still in flight, when both are processed, then no interleaving produces two committed approvals or a partially written record.

**FEAT-08.SPEC-003-AC-12:** Given the Milestone write for an approval succeeds but the subsequent Activity Log write to FEAT-13 fails, when this is observed, then the approval itself still stands (Owen still sees "Approved on {date}," the invoice trigger still fires), and the audit-trail write is retried by FEAT-13's own handling.

### User Story 4 - Next-Invoice Trigger (Priority: P1)

On a successful milestone approval, automatically hands off to invoice generation for the next invoice in the payment schedule, with no freelancer action, reading the schedule exactly as it stood at the moment of approval.

**Acceptance Scenarios:**

**FEAT-08.SPEC-004-AC-01:** Given Owen's approval of a milestone with `payment_trigger` set to invoice on approval is successfully recorded, when FEAT-08.SPEC-003 confirms the write, then this automation hands off to FEAT-09.SPEC-004 immediately, with no action required from Nadia.

**FEAT-08.SPEC-004-AC-02:** Given the approved milestone carries `no_separate_charge`, when the approval is recorded, then no invoice hand-off occurs and no invoice-related feedback appears anywhere.

**FEAT-08.SPEC-004-AC-03:** Given the Payment Schedule has no configured payment trigger at all, when a milestone under it is approved, then no invoice hand-off occurs.

**FEAT-08.SPEC-004-AC-04:** Given Nadia edits the Payment Schedule at the same moment Owen's approval is being recorded, when this automation resolves the trigger, then it uses the schedule exactly as it stood at the moment of approval, not Nadia's concurrent edit.

**FEAT-08.SPEC-004-AC-05:** Given a schedule edit Nadia saved earlier is dated after the approval being processed, when this automation runs, then it is unaffected by that later edit, consistent with the non-retroactive rule.

**FEAT-08.SPEC-004-AC-06:** Given Owen approves two different milestones in the same project at effectively the same time, when both approvals are confirmed, then this automation fires once per approval and hands off two independent invoice-generation requests.

**FEAT-08.SPEC-004-AC-07:** Given FEAT-09.SPEC-004 is unreachable at the moment of hand-off, when the failure occurs, then the approval remains recorded and visible to Owen, the hand-off is retried automatically, and Nadia sees a project-level warning if retries are exhausted.

**FEAT-08.SPEC-004-AC-08:** Given a milestone is reopened and re-approved, when the second approval is confirmed, then this automation re-evaluates the schedule fresh and hands off again if the milestone's `payment_trigger` still applies, independent of any hand-off from the first approval cycle.

**FEAT-08.SPEC-004-AC-09:** Given a milestone's `payment_trigger` marks it to invoice on approval, when the automation completes its hand-off, then the milestone_invoice_auto_generated event is emitted with the milestone reference and trigger type.

**FEAT-08.SPEC-004-AC-10:** Given this automation runs for two milestones approved at effectively the same time, when both runs read the Project's Payment Schedule, then neither run's read is contended by the other, since the schedule is read-only from this automation's perspective.

### User Story 5 - Reopen Recording (Priority: P1)

Writes Nadia's reopen of an approved milestone as a distinct, logged event, resetting the milestone to Reopened status while leaving the original approval record's `approved_at` and `approved_by` fields untouched.

**Acceptance Scenarios:**

**FEAT-08.SPEC-005-AC-01:** Given Nadia confirms reopen on a milestone whose live status is "Approved," when the write commits, then `status` becomes "Reopened" and `approved_at`/`approved_by` remain unchanged.

**FEAT-08.SPEC-005-AC-02:** Given a successful reopen write, when it commits, then an Activity Log entry is fired to FEAT-13 with event type "milestone reopened," Nadia as actor, and the Milestone as the affected record.

**FEAT-08.SPEC-005-AC-03:** Given a successful reopen, when Owen next opens FEAT-08.SPEC-001 for that milestone, then he sees the Approve control again instead of "Approved on {date}."

**FEAT-08.SPEC-005-AC-04:** Given a successful reopen, when Nadia opens FEAT-04.SPEC-001 for that milestone, then its edit controls are enabled again.

**FEAT-08.SPEC-005-AC-05:** Given the milestone's status is no longer "Approved" by the time Nadia's confirmed request is processed, when the check in step 5 runs, then the attempt returns the already-changed outcome with no write.

**FEAT-08.SPEC-005-AC-06:** Given Nadia has no connectivity at the moment she confirms reopen, when the request is evaluated, then it is refused as a connectivity failure and no write is attempted.

**FEAT-08.SPEC-005-AC-07:** Given all eligibility checks pass but the write cannot be confirmed as committed, when Nadia sees the resulting error, then she can retry, and the retry re-evaluates eligibility fresh.

**FEAT-08.SPEC-005-AC-08:** Given Nadia confirms reopen twice in rapid succession, when the first confirmation's write is already committing, then the second is refused as already-changed and only one reopen event is ever recorded.

**FEAT-08.SPEC-005-AC-09:** Given Nadia reopens the same milestone from two of her own sessions at effectively the same time, when both requests are processed, then exactly one write commits and the other returns the already-changed outcome.

**FEAT-08.SPEC-005-AC-10:** Given the Milestone write for a reopen succeeds but the subsequent Activity Log write to FEAT-13 fails, when this is observed, then the reopen itself still stands, and the audit-trail write is retried by FEAT-13's own handling.

### User Story 6 - Approval Authorization & Eligibility Rules (Priority: P1)

Governs who may approve or reopen a milestone, the milestone-state and load preconditions that gate each action, the exactly-once and immutability guarantees on an approval, and the concurrency and connectivity conditions that decide whether an attempt is honored or refused.

**Acceptance Scenarios:**

**FEAT-08.SPEC-006-AC-01:** Given Owen (Client Primary Contact) is viewing a milestone in his own client company's project with status "Deliverable Uploaded" and its deliverable and comment thread fully loaded, when he taps Approve while connected, then the approval is recorded and status moves to "Approved."

**FEAT-08.SPEC-006-AC-02:** Given Owen is viewing a milestone whose status is already "Approved," when the screen renders, then no Approve control is shown -- only the "Approved on {date}" marker.

**FEAT-08.SPEC-006-AC-03:** Given Priya (Client Reviewer Contact) opens the same milestone Owen can approve, when the screen renders, then no Approve control appears anywhere for her.

**FEAT-08.SPEC-006-AC-04:** Given Nadia (Freelancer) opens her own client's milestone review context, when she looks for an approve action, then none exists on any freelancer-facing screen.

**FEAT-08.SPEC-006-AC-05:** Given Dana (Support Operator) is in a logged, read-only support session viewing the milestone, when the screen renders, then no Approve control is shown.

**FEAT-08.SPEC-006-AC-06:** Given Nadia is viewing a milestone with status "Approved," when she looks for a Reopen action, then it is available and enabled.

**FEAT-08.SPEC-006-AC-07:** Given Nadia is viewing a milestone with status "Defined" or "Deliverable Uploaded," when she looks for a Reopen action, then none is shown, and a direct attempt shows "This milestone can only be reopened once it has been approved."

**FEAT-08.SPEC-006-AC-08:** Given Owen or Priya are in the client portal, when either looks for a way to reopen any milestone, then no such action exists anywhere in their portal.

**FEAT-08.SPEC-006-AC-09:** Given Dana is in a support session, when she looks for a Reopen action, then none is shown.

**FEAT-08.SPEC-006-AC-10:** Given Owen views his own client company's milestone, when the screen loads, then he can see its current status and history, since View approval status is always allowed for him on his own company's data.

**FEAT-08.SPEC-006-AC-11:** Given Priya follows a milestone link belonging to a different client company, when the link resolves, then she sees a plain explanation and a fresh-link option, never that company's data (XBR-09).

**FEAT-08.SPEC-006-AC-12:** Given Owen taps Approve twice in rapid succession, when the first tap's write is already in flight, then the second tap is refused as already-approved and no second approval is recorded.

**FEAT-08.SPEC-006-AC-13:** Given Nadia re-prices or removes the milestone's deliverable after Owen's approval screen loaded but before he taps Approve, when Owen taps Approve, then the attempt is refused and Owen is shown the refreshed milestone instead of having his approval recorded.

**FEAT-08.SPEC-006-AC-14:** Given the milestone's deliverable and comment thread have not yet finished loading on Owen's screen, when he looks for the Approve control, then it is disabled until loading completes.

**FEAT-08.SPEC-006-AC-15:** Given Owen has no connectivity, when he attempts to tap Approve, then the action does not proceed and he sees the offline "reconnect to approve" state rather than any success feedback.

**FEAT-08.SPEC-006-AC-16:** Given a milestone is successfully approved, when the write commits, then `status`, `approved_at`, and `approved_by` all change together in the same operation -- no partial state is ever observed.

**FEAT-08.SPEC-006-AC-17:** Given a milestone is successfully reopened, when the write commits, then only `status` changes to "Reopened"; `approved_at` and `approved_by` retain the values from the approval being reopened.

**FEAT-08.SPEC-006-AC-18:** Given a milestone was approved, reopened, and approved again, when the second approval commits, then `approved_at` and `approved_by` reflect the second approval, and both the first and second approval events remain individually visible, unaltered, in the Activity Log (FEAT-13).

**FEAT-08.SPEC-006-AC-19:** Given a milestone is already "Approved," when any role attempts to directly alter `approved_at` or `approved_by`, then no such control or path exists on any spec -- the fields are write-protected outside the approval and reopen automations.

**FEAT-08.SPEC-006-AC-20:** Given the same milestone receives two Approve attempts from two of Owen's own sessions at effectively the same moment, when the first commits, then the second is evaluated against the now-"Approved" state and refused, showing the now-Approved milestone.

**FEAT-08.SPEC-006-AC-21:** Given a milestone has been approved and invoiced, when Nadia attempts to edit or remove it from FEAT-04.SPEC-001, then FEAT-04.SPEC-003's edit-lock rule refuses the attempt, consistent with this spec's exactly-once and immutability guarantees (XBR-10).

**FEAT-08.SPEC-006-AC-22:** Given Dana has a read-only support session open on a milestone at the moment Owen approves it, when Dana next reads the milestone, then she sees the updated "Approved" state with no conflict or error of her own.

### User Story 7 - Milestone Approval Confirmation Notification (Priority: P1)

Confirms to both Owen and Nadia, the moment an approval is recorded, exactly what happened -- so the client-side record and the freelancer-side record of the same event match from the start.

**Acceptance Scenarios:**

**FEAT-08.SPEC-007-AC-01:** Given Owen's approval of a milestone is successfully recorded, when FEAT-08.SPEC-003 confirms the write, then Owen receives an email with subject "You approved {milestone_name}" and Nadia receives an email with subject "{client_name} approved {milestone_name}."

**FEAT-08.SPEC-007-AC-02:** Given Owen opens his confirmation email, when he taps "View milestone," then he lands on FEAT-08.SPEC-001 for that milestone.

**FEAT-08.SPEC-007-AC-03:** Given Nadia opens her confirmation email, when she taps "View milestone," then she lands on that milestone's detail in her own project view.

**FEAT-08.SPEC-007-AC-04:** Given an approval attempt is refused by FEAT-08.SPEC-003 as stale, unauthorized, a connectivity failure, or a write failure, when the refusal occurs, then this notification never fires for that attempt.

**FEAT-08.SPEC-007-AC-05:** Given neither Owen nor Nadia has any way to opt out of this confirmation, when their respective notification preferences are checked, then no preference control exists for it and it always sends.

**FEAT-08.SPEC-007-AC-06:** Given an approval is recorded at any hour, when this notification fires, then it sends immediately regardless of either recipient's configured quiet hours, since this confirmation is transactional.

**FEAT-08.SPEC-007-AC-07:** Given delivery of Owen's copy fails, when the failure occurs, then it is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, and after the final failure Nadia sees a delivery warning on the project.

**FEAT-08.SPEC-007-AC-08:** Given delivery of Nadia's copy fails while Owen's copy succeeds, when this is observed, then Nadia's copy is retried independently of Owen's successful delivery.

**FEAT-08.SPEC-007-AC-09:** Given the approved milestone carries `no_separate_charge`, when Owen's confirmation is generated, then its consent language about issuing the next invoice remains accurate as conditional wording, with no false promise of an invoice.

**FEAT-08.SPEC-007-AC-10:** Given a milestone is reopened and approved a second time, when the second approval is confirmed, then this notification fires again as an independent instance with the second approval's own `approved_at` and `approved_by` values.

**FEAT-08.SPEC-007-AC-11:** Given Priya is a Reviewer contact on the same client company, when a milestone is approved by Owen, then Priya receives no copy of this confirmation.

**FEAT-08.SPEC-007-AC-12:** Given this confirmation is delivered successfully to both recipients, when delivery completes, then the milestone_approval_confirmation_delivered event fires once per recipient.

### Edge Cases

- **FEAT-08.SPEC-001 (Milestone Review & Approval Screen):** The Approve control disables on the first tap so one request is sent, and a milestone re-priced or stripped of its deliverable (or already approved from another session) since load shows the stale-refresh dialog and reloads. Navigating away mid-approval reloads fresh and reflects whatever the in-flight request produced. Source: `docs/blueprint/specifications/FEAT-08-milestone-approval/FEAT-08.SPEC-001-milestone-review-approval-screen.md` (section: Edge Cases)
- **FEAT-08.SPEC-002 (Milestone Reopen Screen):** Reopen controls disable on the first tap so one request is sent, and a milestone already reopened from another session shows an already-changed acknowledgment. Navigating away mid-confirmation reloads fresh, and a stale link to a milestone not currently Approved shows the already-changed message. Source: `docs/blueprint/specifications/FEAT-08-milestone-approval/FEAT-08.SPEC-002-milestone-reopen-screen.md` (section: Edge Cases)
- **FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard):** A retry after a write failure re-runs the full check and records exactly one approval, and any live-state mismatch from what the contact was shown (removed deliverable, re-pricing, status change) refuses the write as stale. A connectivity drop is treated as a write failure with no false Approved state. Source: `docs/blueprint/specifications/FEAT-08-milestone-approval/FEAT-08.SPEC-003-approval-recording-concurrency-guard.md` (section: Edge Cases)
- **FEAT-08.SPEC-004 (Next-Invoice Trigger):** A milestone with no_separate_charge, or a schedule with no payment trigger, generates no invoice and gives no invoicing feedback. The trigger resolves against the schedule as it stood at approval, and approvals of two different milestones at once are each independent confirmed writes. Source: `docs/blueprint/specifications/FEAT-08-milestone-approval/FEAT-08.SPEC-004-next-invoice-trigger.md` (section: Edge Cases)
- **FEAT-08.SPEC-005 (Reopen Recording):** A double-submit is evaluated against the already-Reopened state and returns the already-changed outcome, and a milestone re-approved in a new cycle before the reopen is only refused if it left Approved. Connectivity loss leaves the milestone Approved with an error state, and concurrent reopens resolve first-commit-wins. Source: `docs/blueprint/specifications/FEAT-08-milestone-approval/FEAT-08.SPEC-005-reopen-recording.md` (section: Edge Cases)
- **FEAT-08.SPEC-006 (Approval Authorization & Eligibility Rules):** Double-submitted approvals are evaluated against the current or in-flight state, an approval after the only deliverable was removed is refused, and reopening a milestone that was never approved or is already Reopened is refused with the can-only-be-reopened-once-approved message. Source: `docs/blueprint/specifications/FEAT-08-milestone-approval/FEAT-08.SPEC-006-approval-authorization-eligibility-rules.md` (section: Edge Cases)
- **FEAT-08.SPEC-007 (Milestone Approval Confirmation Notification):** The confirmation remains accurate as sent even if the milestone is later reopened, and each recipient's copy is tracked and retried independently. It fires immediately on the confirmed write, and for a no_separate_charge milestone still uses accurate conditional invoice wording. Source: `docs/blueprint/specifications/FEAT-08-milestone-approval/FEAT-08.SPEC-007-milestone-approval-confirmation-notification.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-08.SPEC-001** (Milestone Review & Approval Screen) as specified: Owen reviews a milestone's current deliverable and comment thread and approves it with a single, plainly explained, timestamped action; Priya sees the identical status and content with no working Approve control. Full spec: `docs/blueprint/specifications/FEAT-08-milestone-approval/FEAT-08.SPEC-001-milestone-review-approval-screen.md`
- **FR-002**: The system MUST implement **FEAT-08.SPEC-002** (Milestone Reopen Screen) as specified: Nadia reopens an approved milestone through a deliberate, explained, logged confirmation -- never a silent status edit -- when genuinely necessary. Full spec: `docs/blueprint/specifications/FEAT-08-milestone-approval/FEAT-08.SPEC-002-milestone-reopen-screen.md`
- **FR-003**: The system MUST implement **FEAT-08.SPEC-003** (Approval Recording & Concurrency Guard) as specified: Records a milestone's approval exactly once, atomically writing the immutable timestamp and approving contact's identity, and refuses the attempt -- showing the refreshed milestone instead -- if the milestone changed underneath the approving contact since it was shown. Full spec: `docs/blueprint/specifications/FEAT-08-milestone-approval/FEAT-08.SPEC-003-approval-recording-concurrency-guard.md`
- **FR-004**: The system MUST implement **FEAT-08.SPEC-004** (Next-Invoice Trigger) as specified: On a successful milestone approval, automatically hands off to invoice generation for the next invoice in the payment schedule, with no freelancer action, reading the schedule exactly as it stood at the moment of approval. Full spec: `docs/blueprint/specifications/FEAT-08-milestone-approval/FEAT-08.SPEC-004-next-invoice-trigger.md`
- **FR-005**: The system MUST implement **FEAT-08.SPEC-005** (Reopen Recording) as specified: Writes Nadia's reopen of an approved milestone as a distinct, logged event, resetting the milestone to Reopened status while leaving the original approval record's `approved_at` and `approved_by` fields untouched. Full spec: `docs/blueprint/specifications/FEAT-08-milestone-approval/FEAT-08.SPEC-005-reopen-recording.md`
- **FR-006**: The system MUST implement **FEAT-08.SPEC-006** (Approval Authorization & Eligibility Rules) as specified: Governs who may approve or reopen a milestone, the milestone-state and load preconditions that gate each action, the exactly-once and immutability guarantees on an approval, and the concurrency and connectivity conditions that decide whether an attempt is honored or refused. Full spec: `docs/blueprint/specifications/FEAT-08-milestone-approval/FEAT-08.SPEC-006-approval-authorization-eligibility-rules.md`
- **FR-007**: The system MUST implement **FEAT-08.SPEC-007** (Milestone Approval Confirmation Notification) as specified: Confirms to both Owen and Nadia, the moment an approval is recorded, exactly what happened -- so the client-side record and the freelancer-side record of the same event match from the start. Full spec: `docs/blueprint/specifications/FEAT-08-milestone-approval/FEAT-08.SPEC-007-milestone-approval-confirmation-notification.md`

### Key Entities

- Milestone (update: approved state)
- Invoice (create: next invoice)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: At least 60% of milestones are approved within 5 days of the deliverable being marked ready for final review (metric: Milestone Approval Turnaround). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Milestone approvals, freelancer reopenings and auto-generated milestone invoices are each observable as distinct signals (milestone_approved, milestone_reopened_by_freelancer, milestone_invoice_auto_generated). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-06**: Client Primary Contacts are assumed to act on approval and payment requests within days, not weeks. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-15**: Approvals are append-only and immutable once created. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-25**: Approvals are timestamped at creation and never silently altered afterward. Full register: `docs/blueprint/features/assumptions-constraints.md`
