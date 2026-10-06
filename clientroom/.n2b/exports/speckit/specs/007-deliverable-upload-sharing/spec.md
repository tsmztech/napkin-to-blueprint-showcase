# Feature Specification: Deliverable Upload & Sharing

**Blueprint feature:** FEAT-06
**Priority tier:** Core
**Build order:** 007 of 33
**Depends on:** FEAT-04, FEAT-16
**Blueprint source:** `docs/blueprint/specifications/FEAT-06-deliverable-upload-sharing/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Deliverable Upload (Priority: P1)

Nadia chooses between uploading a file or pasting an external link, and attaches it as a deliverable on a milestone she has already reached.

**Acceptance Scenarios:**

**FEAT-06.SPEC-001-AC-01:** Given Nadia opens Deliverable Upload from FEAT-06.SPEC-002 on a milestone with no existing deliverable, when the screen loads, then the milestone and project name appear read-only in the header and the upload method toggle defaults to "Upload a file".

**FEAT-06.SPEC-001-AC-02:** Given Nadia is on Deliverable Upload with "Upload a file" selected, when she selects a video file under the size ceiling, then the file name and a live progress indicator appear and the transfer begins immediately.

**FEAT-06.SPEC-001-AC-03:** Given Nadia's file transfer reaches 100%, when the transfer fully completes, then the Attach Deliverable button becomes enabled and the deliverable status badge shows Active.

**FEAT-06.SPEC-001-AC-04:** Given Nadia's file transfer is interrupted by a dropped connection, when connectivity returns, then the transfer resumes automatically from where it left off and the offline banner is removed.

**FEAT-06.SPEC-001-AC-05:** Given Nadia's upload fails because the file exceeds platform parameter: `deliverable-file-size-ceiling`, when the failure occurs, then the file name remains on screen, an error banner explains the ceiling was exceeded, and a Retry control is not offered for this specific failure (the file itself must change).

**FEAT-06.SPEC-001-AC-06:** Given Nadia's upload fails for a server-side reason unrelated to size, when the failure occurs, then the selected file remains on screen with a Retry control that resumes the transfer without requiring her to re-select the file.

**FEAT-06.SPEC-001-AC-07:** Given Nadia selects "Paste a link" and enters a Figma URL, when she taps Attach Deliverable, then the button shows "Checking link..." while FEAT-06.SPEC-004 verifies reachability.

**FEAT-06.SPEC-001-AC-08:** Given Nadia's pasted link is confirmed reachable, when the check completes, then a toast "Deliverable attached" appears and she is returned to FEAT-06.SPEC-002 showing the deliverable as Active.

**FEAT-06.SPEC-001-AC-09:** Given Nadia's pasted link fails to resolve, when FEAT-06.SPEC-004 reports it unreachable, then the inline message "This link couldn't be reached. Check that it's shared and try again." appears and the deliverable is never marked ready or shown to the client.

**FEAT-06.SPEC-001-AC-10:** Given Nadia enters a URL from a domain other than Figma, Google Drive, or Dropbox, when she blurs the link field, then the error "Enter a Figma, Google Drive, or Dropbox link" appears and Attach Deliverable stays disabled.

**FEAT-06.SPEC-001-AC-11:** Given Nadia has an in-progress file transfer and navigates to a different screen, when she confirms "Leave anyway?", then the transfer is cancelled and no Deliverable record is created.

**FEAT-06.SPEC-001-AC-12:** Given Nadia loses connectivity while a file is transferring, when the connection drops, then the banner "You're offline -- upload will resume automatically when you reconnect." appears and the link path becomes disabled.

**FEAT-06.SPEC-001-AC-13:** Given Nadia has two sessions open on the same milestone with no existing deliverable, when both sessions attempt to attach a deliverable and the second session's attempt reaches the server after the first has already committed, then the second session is rejected-with-refresh: "This milestone already has a deliverable. Refresh to see it, or use Replace to add a new version."

**FEAT-06.SPEC-001-AC-14:** Given Owen or Priya (client contacts) attempts to reach this screen's URL directly, when the request is made, then they see "This page isn't part of your portal." and are returned to their own portal home.

### User Story 2 - Deliverable List & Management (Priority: P1)

Nadia views a milestone's deliverables, previews them where feasible, and removes or initiates replacement of one.

**Acceptance Scenarios:**

**FEAT-06.SPEC-002-AC-01:** Given Nadia opens a milestone with no deliverables, when the screen loads, then the empty-state prompt "No deliverables yet. Upload the first one for this milestone." appears with an Upload Deliverable action.

**FEAT-06.SPEC-002-AC-02:** Given Nadia opens a milestone with two deliverables, when the screen loads, then both appear as cards in upload order (most recent first) with their status badges.

**FEAT-06.SPEC-002-AC-03:** Given Nadia taps Preview on an uploaded video deliverable, when the preview opens, then it streams inline without requiring a download.

**FEAT-06.SPEC-002-AC-04:** Given Nadia taps "Open link" on a deliverable flagged as unreachable, then the control is disabled and shows the tooltip "This link is flagged as unreachable."

**FEAT-06.SPEC-002-AC-05:** Given Nadia taps Remove on a deliverable whose milestone is not yet Approved, when she confirms "Remove" in the dialog, then the deliverable's status becomes Removed, a toast "Deliverable removed." appears, and an activity-trail entry is written (FEAT-13).

**FEAT-06.SPEC-002-AC-06:** Given Nadia taps Remove on a deliverable whose milestone is Approved, when the eligibility check runs, then the dialog shows "This milestone has been approved. Remove is not available -- use Replace to share a new version instead." and the deliverable's status does not change.

**FEAT-06.SPEC-002-AC-07:** Given Nadia's milestone becomes Approved (by Owen) while her removal confirmation dialog is open, when she confirms "Remove", then the re-check at commit blocks the removal and the dialog updates to the blocked message.

**FEAT-06.SPEC-002-AC-08:** Given Nadia taps Replace on an Active deliverable, when the action fires, then she is navigated to FEAT-17's re-upload flow carrying that deliverable's reference.

**FEAT-06.SPEC-002-AC-09:** Given Dana opens this screen inside a read-only support session, when the screen loads, then Upload, Replace, and Remove controls are not rendered, and Preview never offers a download.

**FEAT-06.SPEC-002-AC-10:** Given Owen or Priya attempts to reach this screen's URL directly, when the request is made, then they are not shown this screen and reach FEAT-07's client-facing view instead if entitled, or their portal home otherwise.

**FEAT-06.SPEC-002-AC-11:** Given Nadia loses connectivity while viewing this screen, when the connection drops, then the offline banner appears and Upload, Replace, and Remove become disabled while Preview of already-loaded deliverables remains available.

**FEAT-06.SPEC-002-AC-12:** Given a background upload for this milestone completes while Nadia has this screen open, when the transfer finishes, then the corresponding card's status badge updates live from Uploading to Active without a manual refresh.

**FEAT-06.SPEC-002-AC-13:** Given the deliverable list fails to load, when the screen attempts to fetch it, then the error banner "Couldn't load this milestone's deliverables." appears with a Retry action.

### User Story 3 - Resumable Upload Handling (Priority: P1)

Processes a file upload in a resumable, progress-tracked way, pausing and auto-resuming through dropped connections, and marks the deliverable ready only on full completion.

**Acceptance Scenarios:**

**FEAT-06.SPEC-003-AC-01:** Given Nadia selects a 600 MB video file for a milestone, when the transfer begins, then a Deliverable record is created with status Uploading and progress is reported back to FEAT-06.SPEC-001 in real time.

**FEAT-06.SPEC-003-AC-02:** Given Nadia's transfer is interrupted by a dropped connection, when connectivity returns, then the transfer resumes automatically from the last committed byte rather than restarting from zero.

**FEAT-06.SPEC-003-AC-03:** Given Nadia's transfer completes fully, when completion is recorded, then the Deliverable's status becomes Active, a Deliverable Version at round_number 1 is created, the Milestone's status becomes "Deliverable Uploaded", and `deliverable_uploaded` is emitted.

**FEAT-06.SPEC-003-AC-04:** Given Nadia's file exceeds platform parameter: `deliverable-file-size-ceiling`, when she selects it, then no Deliverable record is created and FEAT-06.SPEC-001 shows the ceiling-exceeded failure without a Retry control.

**FEAT-06.SPEC-003-AC-05:** Given a transfer chunk fails for a reason other than a dropped connection, when the failure occurs, then the automation retries that chunk automatically before any failure is surfaced to Nadia.

**FEAT-06.SPEC-003-AC-06:** Given automatic chunk retries are exhausted, when the transfer ultimately fails, then the Deliverable remains Uploading and FEAT-06.SPEC-001 shows Upload Failed with a Retry control that resumes without re-selecting the file.

**FEAT-06.SPEC-003-AC-07:** Given the large-file storage capability reports the freelancer's storage allowance is exhausted mid-transfer, when this occurs, then the transfer stops and Nadia sees "You've reached your storage allowance." with a link to the plan view.

**FEAT-06.SPEC-003-AC-08:** Given a transfer completes fully, when FEAT-06.SPEC-006 checks for its trigger, then it fires only after this completion is recorded and never while the Deliverable's status is still Uploading.

**FEAT-06.SPEC-003-AC-09:** Given Nadia selects a zero-byte file, when the automation evaluates it, then it is rejected with "This file appears to be empty." and no Deliverable record is created.

**FEAT-06.SPEC-003-AC-10:** Given two of Nadia's browser tabs each start a file transfer for the same milestone with no existing deliverable, when both complete, then the first to complete becomes the round-1 Active deliverable and the second is rejected-with-refresh toward the Replace flow.

**FEAT-06.SPEC-003-AC-11:** Given Nadia taps Retry while an automatic chunk retry for the same transfer is already in progress, when she taps it, then the manual retry request is ignored and the existing transfer continues uninterrupted.

### User Story 4 - Linked Asset Reachability Check (Priority: P1)

Validates that a pasted external link resolves before the deliverable is marked ready, flagging an unreachable link to the freelancer.

**Acceptance Scenarios:**

**FEAT-06.SPEC-004-AC-01:** Given Nadia submits a valid, publicly shared Figma link, when the reachability check runs, then link_status is set to reachable, the Deliverable becomes Active, the owning Milestone's status becomes "Deliverable Uploaded", and FEAT-06.SPEC-006 is triggered.

**FEAT-06.SPEC-004-AC-02:** Given Nadia submits a Google Drive link that fails to resolve, when the reachability check runs, then link_status is set to flagged, the Deliverable's status remains pending, and FEAT-06.SPEC-001 shows "This link couldn't be reached. Check that it's shared and try again."

**FEAT-06.SPEC-004-AC-03:** Given a deliverable's link is flagged, when Nadia checks the client-facing view before fixing it, then the deliverable is never shown to the client as broken -- it simply does not appear as ready.

**FEAT-06.SPEC-004-AC-04:** Given Nadia edits a flagged link and re-submits it, when the check re-runs, then it evaluates only the new URL, independent of the prior flagged result.

**FEAT-06.SPEC-004-AC-05:** Given the reachability check itself cannot run because the underlying service is unavailable, when this occurs, then Nadia sees "Couldn't verify this link right now. Try again in a moment." with a Retry option, distinct from a flagged link.

**FEAT-06.SPEC-004-AC-06:** Given a linked asset requires sign-in the product cannot provide, when the check runs, then the outcome is flagged with the same "couldn't be reached" message as any other unreachable link.

**FEAT-06.SPEC-004-AC-07:** Given a linked asset resolves slowly but eventually responds within the bounded wait, when the response arrives, then the check completes normally rather than timing out prematurely.

**FEAT-06.SPEC-004-AC-08:** Given two of Nadia's sessions submit different links for the same milestone with no existing deliverable, when both checks return reachable, then the first to complete becomes the round-1 Active deliverable and the second is rejected-with-refresh toward the Replace flow.

**FEAT-06.SPEC-004-AC-09:** Given Nadia taps Attach Deliverable twice rapidly on the same link, when the second tap occurs, then it is ignored while the first check is in progress and only one check runs.

### User Story 5 - Deliverable Validation & Removal Eligibility Rules (Priority: P1)

Governs upload validity (milestone required, valid/reachable link, size ceiling), who may upload or remove a deliverable, and the approved-milestone removal block.

**Acceptance Scenarios:**

**FEAT-06.SPEC-005-AC-01:** Given Nadia selects a file exactly at platform parameter: `deliverable-file-size-ceiling`, when the size check runs, then the file passes validation.

**FEAT-06.SPEC-005-AC-02:** Given Nadia selects a file one byte over platform parameter: `deliverable-file-size-ceiling`, when the size check runs, then she sees "This file is larger than the size limit for deliverables. Compress it or share it by link instead." and the transfer does not begin.

**FEAT-06.SPEC-005-AC-03:** Given Nadia pastes a link from an unrecognized domain, when she blurs the field, then she sees "Enter a Figma, Google Drive, or Dropbox link." and the link is not submitted for a reachability check.

**FEAT-06.SPEC-005-AC-04:** Given Nadia pastes a well-formed Dropbox link with a tracking query parameter appended, when she blurs the field, then the format check passes and the link proceeds to the reachability check.

**FEAT-06.SPEC-005-AC-05:** Given Nadia's milestone reference is removed in the narrow window between form load and submission, when she attempts to attach a deliverable, then the creation is rejected with "This milestone is no longer available." and no Deliverable record is created.

**FEAT-06.SPEC-005-AC-06:** Given Nadia is the freelancer on her own account, when she attaches a deliverable to any of her milestones, then the action is always allowed.

**FEAT-06.SPEC-005-AC-07:** Given Owen (Client Primary Contact) is signed in to his portal, when he looks for a way to upload a deliverable, then no such control exists anywhere in his view.

**FEAT-06.SPEC-005-AC-08:** Given Dana is inside a read-only support session, when she views a milestone's deliverables, then she sees the list and status badges but no upload, remove, or replace controls, and no download option on preview.

**FEAT-06.SPEC-005-AC-09:** Given Nadia attempts to remove a deliverable on a milestone whose status is not Approved, when she confirms removal, then it is allowed and the deliverable's status becomes Removed.

**FEAT-06.SPEC-005-AC-10:** Given Nadia attempts to remove a deliverable on an Approved milestone, when she confirms removal, then it is refused with "This milestone has been approved. Remove is not available -- use Replace to share a new version instead."

**FEAT-06.SPEC-005-AC-11:** Given a milestone becomes Approved between Nadia tapping Remove and confirming, when she confirms, then the re-check at commit blocks the removal.

**FEAT-06.SPEC-005-AC-12:** Given a previously Approved milestone is later Reopened, when Nadia attempts to remove its deliverable, then removal is allowed because the block applies only while the milestone is Approved.

**FEAT-06.SPEC-005-AC-13:** Given Owen views a deliverable on his own company's project, when he looks for a Remove control, then none is shown -- removal is Nadia-only.

**FEAT-06.SPEC-005-AC-14:** Given Nadia removes an eligible deliverable, when the removal completes, then an append-only activity-trail entry is written with her identity and the timestamp.

**FEAT-06.SPEC-005-AC-15:** Given Nadia attempts to remove a deliverable and it is blocked, when the block occurs, then no activity-trail entry is written, since no state change occurred.

**FEAT-06.SPEC-005-AC-16:** Given Nadia chooses Replace on a deliverable whose milestone is Approved, when she initiates it, then the action is allowed, since Replace is never blocked by milestone approval status.

**FEAT-06.SPEC-005-AC-17:** Given Priya (Client Reviewer Contact) views a deliverable, when she looks for a Replace control, then none is shown.

**FEAT-06.SPEC-005-AC-18:** Given a Deliverable's kind is uploaded file, when its transfer has not yet fully completed, then its status can never be Active, regardless of any other field's value.

**FEAT-06.SPEC-005-AC-19:** Given a Deliverable's kind is linked external asset with link_status flagged, when any process checks its readiness, then its status is never Active and the client is never notified.

**FEAT-06.SPEC-005-AC-20:** Given Dana previews a deliverable's file inside her support session, when she looks for a download option, then none is offered, and an attempted download shows "Support sessions are read-only and don't include file downloads."

### User Story 6 - Deliverable Ready Notification (Priority: P1)

Emails the relevant client contacts once an uploaded or linked deliverable is fully ready to review, so they never have to be told by Nadia directly that something is waiting on them.

**Acceptance Scenarios:**

**FEAT-06.SPEC-006-AC-01:** Given Nadia's file upload on a milestone completes fully, when FEAT-06.SPEC-003 signals completion, then Owen and Priya each receive an email with the subject "{freelancer_business_name}: a new deliverable is ready to review".

**FEAT-06.SPEC-006-AC-02:** Given Nadia's pasted link is confirmed reachable, when FEAT-06.SPEC-004 signals the reachable outcome, then Owen and Priya each receive the same email, with {deliverable_label} rendering as "a linked file".

**FEAT-06.SPEC-006-AC-03:** Given a file upload is only partially complete, when its progress is checked, then no notification is sent -- delivery waits for full completion (XBR-12).

**FEAT-06.SPEC-006-AC-04:** Given a pasted link is flagged as unreachable, when the check completes, then no notification is sent to Owen or Priya.

**FEAT-06.SPEC-006-AC-05:** Given Owen opens this notification's "Review deliverable" CTA, when he is not currently signed in, then he is carried through magic-link sign-in (FEAT-05) and lands on the specific deliverable's view in FEAT-07.

**FEAT-06.SPEC-006-AC-06:** Given this notification is a transactional record email, when Nadia checks her account's notification preferences, then no control exists to turn it off.

**FEAT-06.SPEC-006-AC-07:** Given email delivery of this notification fails, when FEAT-14.SPEC-001 retries it up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` and all attempts fail, then a delivery warning appears on the affected project for Nadia (XBR-30).

**FEAT-06.SPEC-006-AC-08:** Given a client has zero Active contacts remaining at the moment a deliverable becomes ready, when the notification would otherwise fire, then delivery is silently skipped and no failure is recorded.

**FEAT-06.SPEC-006-AC-09:** Given a deliverable is removed shortly after this notification is sent, when the email is nonetheless delivered, then delivery is not cancelled and the email is not recalled.

**FEAT-06.SPEC-006-AC-10:** Given two deliverables on different milestones of the same project become ready within the same minute, when both trigger, then two separate emails are sent -- they are never batched into one.

**FEAT-06.SPEC-006-AC-11:** Given a Deliverable that already became Active once, when any process re-evaluates its readiness, then this notification does not fire a second time for the same Deliverable.

**FEAT-06.SPEC-006-AC-12:** Given Nadia's Branding Profile changes between the moment a deliverable becomes ready and the moment the email actually sends, when the email renders, then it reflects the Branding Profile as it stands at send time.

### Edge Cases

- **FEAT-06.SPEC-001 (Deliverable Upload):** Switching between file upload and paste-a-link cancels an in-progress transfer, navigating to another screen mid-upload prompts a confirmation while the transfer continues in the background, and a double tap on Attach is ignored. Only the most recently selected file is tracked, and a removed file's transfer is cancelled. Source: `docs/blueprint/specifications/FEAT-06-deliverable-upload-sharing/FEAT-06.SPEC-001-deliverable-upload.md` (section: Edge Cases)
- **FEAT-06.SPEC-002 (Deliverable List & Management):** A milestone approved between the Remove tap and confirmation is caught by the re-check, which updates the dialog to the blocked message. A client previewing a removed deliverable sees it refreshed on their next interaction, double taps on Remove open one dialog, and a background upload completion updates the card from Uploading to Active live. Source: `docs/blueprint/specifications/FEAT-06-deliverable-upload-sharing/FEAT-06.SPEC-002-deliverable-list-management.md` (section: Edge Cases)
- **FEAT-06.SPEC-003 (Resumable Upload Handling):** A zero-byte file is rejected before a transfer starts with no Deliverable record, flapping connectivity is debounced, and an exhausted storage allowance mid-transfer leaves the Deliverable Uploading with the allowance message. Closing the tab cancels the transfer and no partial Deliverable Version is created. Source: `docs/blueprint/specifications/FEAT-06-deliverable-upload-sharing/FEAT-06.SPEC-003-resumable-upload-handling.md` (section: Edge Cases)
- **FEAT-06.SPEC-004 (Linked Asset Reachability Check):** A link requiring sign-in the product cannot provide, or one that gives no definitive response within the bounded wait, is flagged as could-not-be-reached; slow links are awaited rather than timed out early. Editing a flagged link is a fresh submission with no memory of the prior attempt, and concurrent link submissions run independently. Source: `docs/blueprint/specifications/FEAT-06-deliverable-upload-sharing/FEAT-06.SPEC-004-linked-asset-reachability-check.md` (section: Edge Cases)
- **FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules):** A file exactly at the size ceiling passes and one byte over fails; link whitespace is normalized and tracking query parameters on a recognized domain still pass format validation. Removal eligibility re-runs at commit time, and a milestone deleted before submission is caught by the screen's Error state. Source: `docs/blueprint/specifications/FEAT-06-deliverable-upload-sharing/FEAT-06.SPEC-005-deliverable-validation-removal-eligibility-rules.md` (section: Edge Cases)
- **FEAT-06.SPEC-006 (Deliverable Ready Notification):** The email's CTA still deep-links to the deliverable even if it was removed before delivery (FEAT-07 handles the removed case). If every contact is removed by delivery time the send is skipped, and the notification has no preference control or quiet-hours window to collide with. Source: `docs/blueprint/specifications/FEAT-06-deliverable-upload-sharing/FEAT-06.SPEC-006-deliverable-ready-notification.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-06.SPEC-001** (Deliverable Upload) as specified: Nadia chooses between uploading a file or pasting an external link, and attaches it as a deliverable on a milestone she has already reached. Full spec: `docs/blueprint/specifications/FEAT-06-deliverable-upload-sharing/FEAT-06.SPEC-001-deliverable-upload.md`
- **FR-002**: The system MUST implement **FEAT-06.SPEC-002** (Deliverable List & Management) as specified: Nadia views a milestone's deliverables, previews them where feasible, and removes or initiates replacement of one. Full spec: `docs/blueprint/specifications/FEAT-06-deliverable-upload-sharing/FEAT-06.SPEC-002-deliverable-list-management.md`
- **FR-003**: The system MUST implement **FEAT-06.SPEC-003** (Resumable Upload Handling) as specified: Processes a file upload in a resumable, progress-tracked way, pausing and auto-resuming through dropped connections, and marks the deliverable ready only on full completion. Full spec: `docs/blueprint/specifications/FEAT-06-deliverable-upload-sharing/FEAT-06.SPEC-003-resumable-upload-handling.md`
- **FR-004**: The system MUST implement **FEAT-06.SPEC-004** (Linked Asset Reachability Check) as specified: Validates that a pasted external link resolves before the deliverable is marked ready, flagging an unreachable link to the freelancer. Full spec: `docs/blueprint/specifications/FEAT-06-deliverable-upload-sharing/FEAT-06.SPEC-004-linked-asset-reachability-check.md`
- **FR-005**: The system MUST implement **FEAT-06.SPEC-005** (Deliverable Validation & Removal Eligibility Rules) as specified: Governs upload validity (milestone required, valid/reachable link, size ceiling), who may upload or remove a deliverable, and the approved-milestone removal block. Full spec: `docs/blueprint/specifications/FEAT-06-deliverable-upload-sharing/FEAT-06.SPEC-005-deliverable-validation-removal-eligibility-rules.md`
- **FR-006**: The system MUST implement **FEAT-06.SPEC-006** (Deliverable Ready Notification) as specified: Emails the relevant client contacts once an uploaded or linked deliverable is fully ready to review, so they never have to be told by Nadia directly that something is waiting on them. Full spec: `docs/blueprint/specifications/FEAT-06-deliverable-upload-sharing/FEAT-06.SPEC-006-deliverable-ready-notification.md`

### Key Entities

- Deliverable (create, update)
- Milestone (read)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: At least 98% of deliverable uploads, including files over 500 MB, complete successfully without the user needing to restart from zero (metric: Deliverable Upload Reliability). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Deliverable uploads, linked deliverables, resumed uploads, failed uploads and removals are each observable as distinct signals (deliverable_uploaded, deliverable_linked, upload_resumed, upload_failed, deliverable_removed). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-22**: Deliverables are typically tens of MB and sometimes over 1 GB, retained with version history. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-30**: Large-file storage and delivery capability is a required dependency. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-27**: Screens show real progress while loading and say plainly when an action needs a connection. Full register: `docs/blueprint/features/assumptions-constraints.md`
