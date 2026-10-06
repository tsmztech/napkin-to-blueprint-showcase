# Feature Specification: Deliverable Version History

**Blueprint feature:** FEAT-17
**Priority tier:** Important
**Build order:** 010 of 33
**Depends on:** FEAT-06, FEAT-16
**Blueprint source:** `docs/blueprint/specifications/FEAT-17-deliverable-version-history/`

## User Scenarios & Testing (mandatory)

### User Story 1 - New Version Upload (Priority: P2)

Nadia re-uploads a revised file to an existing deliverable, adding a new preserved round rather than replacing the current one.

**Acceptance Scenarios:**

**FEAT-17.SPEC-001-AC-01:** Given Nadia opens New Version Upload from FEAT-06.SPEC-002 on a deliverable with an Active Round 3, when the screen loads, then the header shows "Current: Round 3, uploaded {date}" and the picker is empty.

**FEAT-17.SPEC-001-AC-02:** Given Nadia is on New Version Upload, when she selects a file under the size ceiling, then a live progress indicator appears and the transfer begins immediately.

**FEAT-17.SPEC-001-AC-03:** Given Nadia's file transfer reaches 100%, when the transfer fully completes, then the version is committed, the badge "Round 4 added" appears, and the View versions button becomes enabled.

**FEAT-17.SPEC-001-AC-04:** Given Nadia's transfer has completed and Round 4 is committed, when she taps View versions, then she navigates to FEAT-17.SPEC-002 showing Round 4 as latest and open, and the tap changes no data; if she instead closes the screen without tapping it, Round 4 still exists.

**FEAT-17.SPEC-001-AC-05:** Given Nadia's re-upload fails because the file exceeds platform parameter: `deliverable-file-size-ceiling`, when the failure occurs, then the message "This file is larger than the size limit for deliverables. Compress it or share it by link instead." appears with no Retry control, the current round is stated as unchanged and still active, and no Deliverable Version record is created.

**FEAT-17.SPEC-001-AC-06:** Given Nadia's re-upload fails for a server-side reason unrelated to size, when the failure occurs, then the message "Couldn't upload {file name}. The current version, Round {current round_number}, is unchanged and still active." appears, the selected file remains on screen with a Retry control that resumes the transfer without requiring her to re-select the file.

**FEAT-17.SPEC-001-AC-07:** Given Nadia's transfer is interrupted by a dropped connection, when connectivity returns, then the transfer resumes automatically from where it left off and the offline banner is removed.

**FEAT-17.SPEC-001-AC-08:** Given Nadia has an in-progress re-upload and uses the back arrow, Cancel, or navigates to a different screen, when the dialog "Your upload is still in progress. Leave anyway?" appears and she chooses "Leave", then the transfer is cancelled and no Deliverable Version record is created.

**FEAT-17.SPEC-001-AC-09:** Given two of Nadia's browser sessions each complete a re-upload for the same deliverable at effectively the same moment, when both transfers commit, then both rounds are preserved as separate, sequential Deliverable Versions with no rejection or overwrite.

**FEAT-17.SPEC-001-AC-10:** Given Owen or Priya (client contacts) attempts to reach this screen's URL directly, when the request is made, then they see "This page isn't part of your portal." and are returned to their own portal home.

**FEAT-17.SPEC-001-AC-11:** Given Dana is inside a read-only support session (FEAT-31) viewing the deliverable, when she looks for an "Upload New Version" entry point, then none is rendered.

**FEAT-17.SPEC-001-AC-12:** Given Nadia loses connectivity while a file is transferring, when the connection drops, then the banner "You're offline -- upload will resume automatically when you reconnect." appears.

**FEAT-17.SPEC-001-AC-13:** Given the deliverable Nadia is re-uploading to is removed in another of her sessions before this transfer commits, when the transfer attempts to complete, then it fails with "This deliverable no longer exists." and no Deliverable Version record is created.

**FEAT-17.SPEC-001-AC-14:** Given Nadia selects a zero-byte file, when the failure occurs, then "This file appears to be empty." appears with no Retry control and no Deliverable Version is created.

**FEAT-17.SPEC-001-AC-15:** Given the storage allowance is exhausted, when the transfer cannot proceed, then "You've reached your storage allowance." appears with a "View your plan" link to FEAT-23, no Retry control, and no Deliverable Version is created.

**FEAT-17.SPEC-001-AC-16:** Given the storage capability cannot accept transfers, when Nadia selects a file, then "Uploads aren't available right now. Try again shortly." appears with a Retry control that re-attempts without re-selecting the file.

**FEAT-17.SPEC-001-AC-17:** Given the carried deliverable reference fails to resolve, when the screen opens, then "This deliverable couldn't be loaded." appears and tapping "Go back" returns Nadia to FEAT-06.SPEC-002.

**FEAT-17.SPEC-001-AC-18:** Given Nadia's session expires with a file selected, when the dialog "Your session has expired. Sign in to continue." appears and she taps "Sign in" and succeeds, then the screen returns with the file restored and the transfer resumes.

**FEAT-17.SPEC-001-AC-19:** Given Nadia chooses "Stay" in the leave dialog during an in-progress transfer, when the dialog closes, then the transfer continues uninterrupted and progress keeps updating.

**FEAT-17.SPEC-001-AC-20:** Given Nadia uses only the keyboard, when she focuses the "Choose file" button and presses Enter, then the file chooser opens and a chosen file starts the transfer exactly as a dropped file does.

**FEAT-17.SPEC-001-AC-21:** Given the deliverable is a linked external asset or has no committed first round, when a transfer is attempted, then "New versions can't be added to this deliverable." appears with a "Go back" action and no Retry control.

### User Story 2 - Version Browser & Comparison (Priority: P2)

Any authorized viewer opens the version selector on a deliverable, browses every round by number, and opens any earlier round alongside the latest.

**Acceptance Scenarios:**

**FEAT-17.SPEC-002-AC-01:** Given Owen opens a deliverable with only one version, when the screen loads, then no version selector is shown and the round detail panel shows that version directly.

**FEAT-17.SPEC-002-AC-02:** Given Priya opens a deliverable with 3 versions, when the screen loads, then a version selector with 3 chips appears, Round 3 is highlighted as open, and it is labeled "Latest."

**FEAT-17.SPEC-002-AC-03:** Given Nadia is viewing Round 3 of a large-video deliverable, when she taps the Round 1 chip, then a brief loading indicator appears in the detail panel before Round 1's content displays.

**FEAT-17.SPEC-002-AC-04:** Given Owen is viewing Round 1 of a deliverable and Nadia uploads Round 2 in another session, when Owen's screen is already open, then Round 1 remains fully visible and unchanged, and Round 2 does not appear until Owen reopens or reloads the screen.

**FEAT-17.SPEC-002-AC-05:** Given Dana is inside a read-only support session viewing a deliverable's versions, when she looks for a way to download any round's file, then no download control is rendered for any round.

**FEAT-17.SPEC-002-AC-06:** Given Dana attempts to reach a version's underlying file directly (bypassing the rendered controls) during a support session, when the attempt is made, then it is refused with "Downloads are not available in a support session."

**FEAT-17.SPEC-002-AC-07:** Given Priya taps the comment count indicator on Round 2, when the navigation completes, then she lands in FEAT-07's comment thread scoped to Round 2.

**FEAT-17.SPEC-002-AC-08:** Given Owen attempts to reach a deliverable's version history for a company other than his own, when the request is made, then he sees "This page isn't part of your portal." and no version data is shown.

**FEAT-17.SPEC-002-AC-09:** Given Nadia has just completed an upload via FEAT-17.SPEC-001, when she taps View versions and reaches this screen, then the new round is shown as latest and open by default.

**FEAT-17.SPEC-002-AC-10:** Given a round's file type cannot be previewed inline, when that round is opened, then a generic file icon with name and size is shown in place of a preview, with the download control still available to an authorized viewer.

**FEAT-17.SPEC-002-AC-11:** Given Nadia loses connectivity while this screen is open, when the connection drops, then the offline banner appears, previously viewed rounds remain selectable and viewable, and unopened rounds show as unavailable until reconnection.

**FEAT-17.SPEC-002-AC-12:** Given the deliverable or its version list fails to load, when the screen attempts to open, then "This deliverable's versions couldn't be loaded." appears with a "Try again" action.

**FEAT-17.SPEC-002-AC-13:** Given a viewer taps several round chips in rapid succession, when the loads resolve out of order, then only the most recently tapped round's content is shown.

**FEAT-17.SPEC-002-AC-14:** Given Owen opens a round with zero comments, when the comment count indicator renders, then it shows "0 comments" and remains tappable into FEAT-07's empty thread state for that round.

**FEAT-17.SPEC-002-AC-15:** Given the Error state is showing, when Priya taps "Try again" and resolution succeeds, then the Loading placeholder appears briefly and the version selector and detail panel render with the latest round open.

**FEAT-17.SPEC-002-AC-16:** Given Nadia is offline with an unopened Round 2, when she taps its chip, then nothing happens, the chip reads "Available when you reconnect," and it becomes selectable again once connectivity returns.

**FEAT-17.SPEC-002-AC-17:** Given Owen opens a round whose file is an image, when he taps the preview, then it opens in a zoomable view; and given the round is a video, then tapping plays it inline with playback controls; and given the type is unpreviewable, then tapping the generic icon does nothing.

### User Story 3 - Version Creation & Preservation (Priority: P2)

On a fully completed re-upload, creates the next immutable Deliverable Version, and leaves every prior version's own record untouched; the new round becomes "latest" by derivation (highest round_number), never by a write to a prior version.

**Acceptance Scenarios:**

**FEAT-17.SPEC-003-AC-01:** Given Nadia selects a 400 MB revised video file for a deliverable that already has Round 2 as latest, when the transfer begins, then no entity changes occur yet and Round 2 remains Active and is_latest.

**FEAT-17.SPEC-003-AC-02:** Given Nadia's transfer completes fully, when completion is recorded, then a new Deliverable Version is created at round_number 3, Round 3 is the derived latest (highest round_number), and no field of Round 2 or Round 1 is written or changed.

**FEAT-17.SPEC-003-AC-03:** Given Nadia's transfer is interrupted by a dropped connection, when connectivity returns, then the transfer resumes automatically from the last committed byte rather than restarting from zero.

**FEAT-17.SPEC-003-AC-04:** Given Nadia's file exceeds platform parameter: `deliverable-file-size-ceiling`, when she selects it, then no Deliverable Version is created and FEAT-17.SPEC-001 shows the ceiling-exceeded failure without a Retry control.

**FEAT-17.SPEC-003-AC-05:** Given the deliverable Nadia is re-uploading to is removed in another session before the transfer commits, when the transfer attempts to complete, then no Deliverable Version is created and FEAT-17.SPEC-001 shows "This deliverable no longer exists."

**FEAT-17.SPEC-003-AC-06:** Given a transfer chunk fails for a reason other than a dropped connection, when the failure occurs, then the automation retries that chunk automatically before any failure is surfaced to Nadia.

**FEAT-17.SPEC-003-AC-07:** Given automatic chunk retries are exhausted, when the transfer ultimately fails, then no Deliverable Version is created, the existing latest version is unaffected, and FEAT-17.SPEC-001 shows Upload Failed with a Retry control.

**FEAT-17.SPEC-003-AC-08:** Given the large-file storage capability reports the freelancer's storage allowance is exhausted mid-transfer, when this occurs, then no Deliverable Version is created and Nadia sees "You've reached your storage allowance." with a link to the plan view (FEAT-23) and no Retry control.

**FEAT-17.SPEC-003-AC-09:** Given a transfer completes fully, when FEAT-06.SPEC-006's reused notification checks for its trigger, then it fires only after the new version has committed and never while the transfer is still partial.

**FEAT-17.SPEC-003-AC-10:** Given Nadia selects a zero-byte file, when the automation evaluates it, then it is rejected with "This file appears to be empty." and no Deliverable Version is created.

**FEAT-17.SPEC-003-AC-11:** Given two of Nadia's sessions each complete a re-upload transfer for the same deliverable at effectively the same time, when both commit, then both are preserved as separate sequential rounds (e.g., 3 and 4), the higher-numbered one being the derived latest, with neither rejected or overwritten.

**FEAT-17.SPEC-003-AC-12:** Given Nadia taps Retry while an automatic chunk retry for the same transfer is already in progress, when she taps it, then the manual retry request is ignored and the existing transfer continues uninterrupted.

**FEAT-17.SPEC-003-AC-13:** Given a transfer is triggered from a session that is not Nadia's active session (for example an expired session, or a request attributed to Owen, Priya, or Dana), when the automation runs its authorization step, then no transfer starts and no Deliverable Version is created.

**FEAT-17.SPEC-003-AC-14:** Given Owen has approved the milestone containing Nadia's uploaded-file deliverable, when Nadia re-uploads a revised file and the transfer completes, then a new Deliverable Version is created at the next round_number and the approval does not block it.

**FEAT-17.SPEC-003-AC-15:** Given Nadia's re-upload transfer completes fully, when the version commits, then exactly one FEAT-13 trail entry with actor Nadia and the completion timestamp is requested; and given a transfer fails, is cancelled, or is rejected, then no trail entry is requested.

**FEAT-17.SPEC-003-AC-16:** Given the target deliverable is a linked external asset, or has no committed round 1, when Nadia's session triggers a re-upload, then no Deliverable Version is created and FEAT-17.SPEC-001 shows "New versions can't be added to this deliverable."

### User Story 4 - Version Numbering, Immutability & Retention Rules (Priority: P2)

Governs sequential round numbering, immutability once uploaded, the uncapped-but-storage-bounded version count, and the "latest version" derivation for every Deliverable Version.

**Acceptance Scenarios:**

**FEAT-17.SPEC-004-AC-01:** Given a deliverable's current highest round is 2, when FEAT-17.SPEC-003 commits a new version, then it is assigned round_number 3.

**FEAT-17.SPEC-004-AC-02:** Given a new version commits at round_number 3, when is_latest is evaluated, then Round 3 qualifies as latest and Round 2 does not, with no field of Round 1 or Round 2 written or changed.

**FEAT-17.SPEC-004-AC-03:** Given Nadia (or anyone) looks for an edit control on any existing Deliverable Version anywhere in the product, when the search is made, then none exists -- every field is permanently fixed once the version is created.

**FEAT-17.SPEC-004-AC-04:** Given Nadia looks for a delete or purge control on an individual Deliverable Version while her account is active, when the search is made, then none exists; the only path to a changed current state is uploading a new round.

**FEAT-17.SPEC-004-AC-05:** Given Nadia has just uploaded a new, later round, when she looks for a way to mark an older round "current" again, then no such control exists anywhere in the product.

**FEAT-17.SPEC-004-AC-06:** Given a deliverable has only one version, when its is_latest is evaluated, then it is true, and FEAT-17.SPEC-002 shows no version selector as a result.

**FEAT-17.SPEC-004-AC-07:** Given two of Nadia's sessions each complete a re-upload transfer for the same deliverable within moments of each other, when both commit, then they are assigned sequential, non-colliding round_numbers (e.g., 3 and 4) with no gap and no duplicate.

**FEAT-17.SPEC-004-AC-08:** Given a re-upload transfer fails partway through, when the failure is recorded, then no Deliverable Version row exists for that attempt and the round_number sequence is unaffected.

**FEAT-17.SPEC-004-AC-09:** Given a deliverable has accumulated 40 versions over the life of a long-running project, when a 41st re-upload is attempted, then no version-count rule in this spec blocks it -- only the freelancer's storage allowance (FEAT-16.SPEC-004) can prevent the transfer from completing.

**FEAT-17.SPEC-004-AC-10:** Given a file selected for a new version is zero bytes, when FEAT-17.SPEC-003 evaluates it, then no Deliverable Version is created and the file field validation rejects it before any round_number is assigned.

**FEAT-17.SPEC-004-AC-11:** Given a file selected for a new version exceeds platform parameter: `deliverable-file-size-ceiling`, when FEAT-17.SPEC-003 evaluates it, then the file field validation rejects it and no Deliverable Version is created.

**FEAT-17.SPEC-004-AC-12:** Given Nadia's account is deleted (FEAT-24), when the deletion completes, then every Deliverable Version and its stored bytes belonging to her account has been removed, with no rounds retained selectively.

**FEAT-17.SPEC-004-AC-13:** Given Owen, Priya, or Dana looks for a create control for a new Deliverable Version, when the search is made, then none is rendered for any of them -- only Nadia can trigger a re-upload (per FEAT-17.SPEC-005's authority).

**FEAT-17.SPEC-004-AC-14:** Given a deliverable's version history is inspected at any point in its life, when the round_number sequence is examined, then it forms an unbroken run from 1 to the current version count with no gaps and no duplicate numbers.

### User Story 5 - Version Access & Comment-Anchoring Rules (Priority: P2)

Governs who may view, open, download, or create a Deliverable Version per the Access Matrix, and the rule that a comment stays attached to the version it was posted on even after newer rounds exist.

**Acceptance Scenarios:**

**FEAT-17.SPEC-005-AC-01:** Given Nadia opens New Version Upload on a deliverable she owns, when the screen checks authorization, then she is allowed to proceed.

**FEAT-17.SPEC-005-AC-02:** Given Owen attempts to reach a "New Version Upload" control anywhere in his portal, when he looks for one, then none is rendered -- only Nadia can create a version.

**FEAT-17.SPEC-005-AC-03:** Given Priya attempts to reach a "New Version Upload" control anywhere in her portal, when she looks for one, then none is rendered.

**FEAT-17.SPEC-005-AC-04:** Given Dana is inside a read-only support session, when she looks for a way to trigger a new version upload, then no such entry point is rendered.

**FEAT-17.SPEC-005-AC-05:** Given Owen opens the version browser for a deliverable belonging to his own client company, when the screen loads, then every version of that deliverable is visible to him.

**FEAT-17.SPEC-005-AC-06:** Given Owen attempts to reach a version of a deliverable belonging to a different client company, when the request is made, then he sees "This page isn't part of your portal." and no version data is returned.

**FEAT-17.SPEC-005-AC-07:** Given Priya opens the version browser for her own client company's deliverable, when she selects a round, then she can view and download that round's file.

**FEAT-17.SPEC-005-AC-08:** Given Dana is inside a read-only support session viewing a deliverable's versions, when she looks for a download control on any round, then none is rendered.

**FEAT-17.SPEC-005-AC-09:** Given Dana attempts to download a version's file by a means other than the rendered controls during a support session, when the attempt is made, then it is refused with "Downloads are not available in a support session."

**FEAT-17.SPEC-005-AC-10:** Given Priya posts a comment while viewing Round 2 of a deliverable, when Nadia later uploads Round 3, then Priya's comment remains anchored to Round 2 and does not appear when Round 3 is opened.

**FEAT-17.SPEC-005-AC-11:** Given a comment is anchored to Round 2 and Round 2 is no longer the latest version, when any authorized viewer opens Round 2 directly, then the comment is still fully visible there.

**FEAT-17.SPEC-005-AC-12:** Given Owen's role changes from Reviewer to Primary partway through a project, when his historical comments anchored to earlier rounds are viewed, then they remain unchanged and still correctly anchored.

**FEAT-17.SPEC-005-AC-13:** Given a person is a client contact for two different freelancers, when they view Freelancer A's version history, then no version, comment, or metadata from Freelancer B's account is ever visible.

**FEAT-17.SPEC-005-AC-14:** Given Nadia opens the version browser for any of her own deliverables, when she selects any round, then she can view, open, and download it without restriction.

**FEAT-17.SPEC-005-AC-15:** Given Dana's support session and Owen's own portal session are both open on the same deliverable's versions at the same time, when either views a round, then neither session's activity is visible to or affects the other.

### Edge Cases

- **FEAT-17.SPEC-001 (New Version Upload):** A failed upload creates no Deliverable Version and does not advance the round-number sequence, and navigating away mid-upload prompts a confirmation while the transfer continues in the background. A transfer completed before a tab close is already committed, and only the most recently selected file is tracked. Source: `docs/blueprint/specifications/FEAT-17-deliverable-version-history/FEAT-17.SPEC-001-new-version-upload.md` (section: Edge Cases)
- **FEAT-17.SPEC-002 (Version Browser & Comparison):** The screen is a snapshot, so a newly uploaded version does not disturb the round a viewer has open. A deliverable with zero versions surfaces the Error state on a stale link, an unrenderable file type falls back to a generic icon with name and size, and rapid round switching shows only the most recently tapped round. Source: `docs/blueprint/specifications/FEAT-17-deliverable-version-history/FEAT-17.SPEC-002-version-browser-comparison.md` (section: Edge Cases)
- **FEAT-17.SPEC-003 (Version Creation & Preservation):** A zero-byte file is rejected before transfer with no version created, flapping connectivity is debounced, and an exhausted storage allowance mid-transfer stops the transfer with no version. Closing the tab cancels the transfer and leaves the existing latest version untouched. Source: `docs/blueprint/specifications/FEAT-17-deliverable-version-history/FEAT-17.SPEC-003-version-creation-preservation.md` (section: Edge Cases)
- **FEAT-17.SPEC-004 (Version Numbering, Immutability & Retention Rules):** Round numbers are re-read at commit so near-simultaneous re-uploads never collide, failed transfers reserve or skip no round, and a deliverable with one version is round 1 and trivially latest. There is no cap on version count other than the freelancer's storage allowance. Source: `docs/blueprint/specifications/FEAT-17-deliverable-version-history/FEAT-17.SPEC-004-version-numbering-immutability-retention-rules.md` (section: Edge Cases)
- **FEAT-17.SPEC-005 (Version Access & Comment-Anchoring Rules):** A comment stays anchored to the round it was posted on even if a newer round is uploaded moments later or the round is superseded, and a later role change does not affect already-anchored comments. A concurrent operator support session never interacts with a client's view. Source: `docs/blueprint/specifications/FEAT-17-deliverable-version-history/FEAT-17.SPEC-005-version-access-comment-anchoring-rules.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-17.SPEC-001** (New Version Upload) as specified: Nadia re-uploads a revised file to an existing deliverable, adding a new preserved round rather than replacing the current one. Full spec: `docs/blueprint/specifications/FEAT-17-deliverable-version-history/FEAT-17.SPEC-001-new-version-upload.md`
- **FR-002**: The system MUST implement **FEAT-17.SPEC-002** (Version Browser & Comparison) as specified: Any authorized viewer opens the version selector on a deliverable, browses every round by number, and opens any earlier round alongside the latest. Full spec: `docs/blueprint/specifications/FEAT-17-deliverable-version-history/FEAT-17.SPEC-002-version-browser-comparison.md`
- **FR-003**: The system MUST implement **FEAT-17.SPEC-003** (Version Creation & Preservation) as specified: On a fully completed re-upload, creates the next immutable Deliverable Version, and leaves every prior version's own record untouched; the new round becomes "latest" by derivation (highest round_number), never by a write to a prior version. Full spec: `docs/blueprint/specifications/FEAT-17-deliverable-version-history/FEAT-17.SPEC-003-version-creation-preservation.md`
- **FR-004**: The system MUST implement **FEAT-17.SPEC-004** (Version Numbering, Immutability & Retention Rules) as specified: Governs sequential round numbering, immutability once uploaded, the uncapped-but-storage-bounded version count, and the "latest version" derivation for every Deliverable Version. Full spec: `docs/blueprint/specifications/FEAT-17-deliverable-version-history/FEAT-17.SPEC-004-version-numbering-immutability-retention-rules.md`
- **FR-005**: The system MUST implement **FEAT-17.SPEC-005** (Version Access & Comment-Anchoring Rules) as specified: Governs who may view, open, download, or create a Deliverable Version per the Access Matrix, and the rule that a comment stays attached to the version it was posted on even after newer rounds exist. Full spec: `docs/blueprint/specifications/FEAT-17-deliverable-version-history/FEAT-17.SPEC-005-version-access-comment-anchoring-rules.md`

### Key Entities

- Deliverable Version (create, read)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: New version uploads, version opens and version comparisons are each observable as distinct signals (version_uploaded, version_opened, version_compared); no metric in the success-metrics register connects to this feature, so the outcome is grounded in its Signals alone. Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-22**: Deliverables are retained with version history for the life of the account. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-15**: Records are append-only and immutable once created, which is why versions and anchored comments never change. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-30**: Large-file storage and delivery capability with version history is a required dependency. Full register: `docs/blueprint/features/assumptions-constraints.md`
