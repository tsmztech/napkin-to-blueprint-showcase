---
document_type: spec
spec_type: screen
spec_id: FEAT-17.SPEC-001
spec_name: New Version Upload
spec_slug: new-version-upload
parent_feature: FEAT-17
parent_feature_name: Deliverable Version History
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 21
---

# Screen Spec: New Version Upload

## Overview

**Name:** New Version Upload
**ID:** FEAT-17.SPEC-001
**Type:** Screen
**Purpose:** Nadia re-uploads a revised file to an existing deliverable, adding a new preserved round rather than replacing the current one.
**Parent Feature:** FEAT-17 -- Deliverable Version History

## Scope and Non-Goals

**In Scope:**
- Selecting and uploading a replacement file for a deliverable that already has at least one Active, uploaded-file version
- Showing the current (latest) round's number and upload date as read-only context while the new round uploads
- Displaying live upload progress, fed by FEAT-17.SPEC-003 (Version Creation & Preservation)
- Preserving the selected file and offering retry without re-selection when a re-upload fails
- Confirming the new round was added (the version is committed the moment the transfer completes) and offering a way to the version view

**Non-Goals:**
- The deliverable's first-ever upload (round 1) -- handled entirely by FEAT-06.SPEC-001 (Deliverable Upload); this screen only ever adds round 2 or later to a deliverable that already has an uploaded-file version
- Browsing, opening, or comparing existing versions -- handled by FEAT-17.SPEC-002 (Version Browser & Comparison); this screen is upload-only
- Replacing the URL of a linked external asset (Figma, Google Drive, Dropbox) -- excluded because Deliverable Version's own field list carries a `file` field only, not a link (Feature Dependency Map); a linked deliverable's URL is changed through FEAT-06's own deliverable management, never through this feature
- Defining round numbering, immutability, or the version-count/storage-allowance ceiling -- governed by FEAT-17.SPEC-004 (Version Numbering, Immutability & Retention Rules), which this screen only reflects
- Defining who may reach this screen -- governed by FEAT-17.SPEC-005 (Version Access & Comment-Anchoring Rules); this spec's Access and Visibility table restates that authority's outcome for this specific screen

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-002 (Deliverable List & Management) | Nadia chooses "Upload New Version" (the Brief's Cross-Feature Touchpoints table: reached from "replace" on the deliverable's management screen) on a deliverable whose kind is an uploaded file and that already has at least one version | Deliverable reference, the current latest round's round_number and uploaded_at, milestone and project names |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Select a file and submit a new version | -- |
| Owen (Client Primary Contact) | No | No | This screen is never part of the client portal's navigation (FEAT-05); a direct link shows "This page isn't part of your portal." and returns Owen to his portal home. Only Nadia may create versions (FEAT-17.SPEC-005). |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- this screen is never part of the client portal's navigation; a direct link returns Priya to her portal home. |
| Dana (Support Operator) | No | No | Dana's read-only support session (FEAT-31) renders the version browser (FEAT-17.SPEC-002) but never this upload screen; the "Upload New Version" entry point is not rendered inside a support session, consistent with FEAT-17.SPEC-005's authorization rules. |
| Unauthenticated | No | No | Redirected to the freelancer sign-in screen; no deliverable context or in-progress file selection is preserved. |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." with a "Sign in" action (Interactions) -- a selected file is preserved locally and restored after re-authentication succeeds; an in-flight transfer resumes only once sign-in completes. |

## Layout and Content

**Header:** Screen title "Upload New Version" with a back arrow (returns to FEAT-06.SPEC-002) and read-only context: deliverable name, milestone name, project name, and the current latest round's label ("Current: Round {round_number}, uploaded {uploaded_at}").

**Body:** A single-column form with:
- **File picker / drop target** -- a file picker control with a drop target; once a file is chosen, its name and size replace the picker with a progress indicator area below it (queued / uploading with percentage / paused-resuming / complete / failed -- the same vocabulary FEAT-17.SPEC-003 reports), and a "Remove" control to clear the selection while the file is queued, uploading, or failed (not after the version is committed)
- **New round badge** -- display-only; appears only after the transfer completes and the new version is committed, showing "Round {round_number} added" using the numbering FEAT-17.SPEC-004 assigned. It never appears in the future tense.
- **Failure area** -- appears in place of the progress indicator when a transfer fails; shows the reason-specific message, and only where the reason allows it a Retry control, per the Upload Failed breakdown under States.

**Footer:** "View versions" primary action button (disabled until the transfer has completed and the version is committed; has no data effect, it only navigates to FEAT-17.SPEC-002) and a "Cancel" secondary action (rendered only while no version has been committed; after completion it is not rendered, since there is nothing left to cancel).

The header's deliverable, milestone, project, and current-round context is display-only.

### Responsive Behavior

- **Compact breakpoint:** Single-column form, full width; header context and footer actions remain visible without scrolling past the fold for the file picker.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.
- **Progress indicator:** Uniform scaling, no structural change -- the progress bar and percentage text remain a single row at both breakpoints.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow (no transfer running: empty, failed, complete, or error state) | Tap | Navigate to FEAT-06.SPEC-002 (Deliverable List & Management) | Screen closes | Returns to the deliverable's management view; no dialog |
| Back arrow (transfer running or paused) | Tap | Opens the leave confirmation dialog (same dialog as Cancel while a transfer runs) | Dialog shown; transfer keeps running until Leave is chosen | Dialog "Your upload is still in progress. Leave anyway?" with "Leave" and "Stay" |
| File picker / drop target | Tap or drag-and-drop a file | Captures the selected file locally and immediately begins the re-upload transfer via FEAT-17.SPEC-003 | Picker is replaced by the file's name, size, and a progress indicator | Progress shows "Uploading... {percentage}%" and updates in real time |
| "Choose file" button (keyboard equivalent of the drop target) | Enter or Space while focused | Opens the file chooser; a chosen file behaves exactly as the file picker row above | Same as the file picker row | Same as the file picker row |
| Remove (queued or in-progress file) | Tap | Cancels the in-progress transfer (FEAT-17.SPEC-003) and clears the selection; no version exists yet, so nothing is lost | Progress area is replaced by the empty file picker | File picker reappears empty |
| Remove (failed file) | Tap | Clears the selection; no transfer is running and no version exists | Failure area is replaced by the empty file picker | File picker reappears empty |
| View versions button (transfer complete, version committed) | Tap | Navigates to FEAT-17.SPEC-002; performs no data change (the version already exists) | Screen closes | FEAT-17.SPEC-002 opens with the new round as latest and open |
| View versions button (no completed transfer) | Tap | No action -- button is disabled | None | Button remains disabled |
| Retry (failed upload; rendered only for the reasons that allow it, see Upload Failed) | Tap | Re-attempts the transfer for the already-selected file without requiring re-selection (FEAT-17.SPEC-003) | Failure area returns to the progress indicator "Uploading..." | Progress resumes from the last successfully transferred point where the transfer protocol allows it, or restarts the transfer if no partial state survived |
| Cancel (no transfer running: empty or failed state) | Tap | Discards any selection and navigates to FEAT-06.SPEC-002 | Screen closes | No confirmation dialog (no transfer is running and no version exists to lose) |
| Cancel (transfer running or paused) | Tap | Opens the leave confirmation dialog | Dialog shown; transfer keeps running until Leave is chosen | Dialog "Your upload is still in progress. Leave anyway?" with "Leave" and "Stay" |
| "Leave" (confirmation dialog) | Tap | Cancels the transfer (FEAT-17.SPEC-003) and navigates to the destination that triggered the dialog (FEAT-06.SPEC-002 for back arrow and Cancel) | Screen closes; no Deliverable Version is created | Returns to the deliverable's management view |
| "Stay" (confirmation dialog) | Tap | Dismisses the dialog | Dialog closes; transfer continues untouched | Progress indicator remains visible and updating |
| "Sign in" (session-expired dialog) | Tap | Opens the freelancer sign-in flow; on success returns here with the selected file restored | Dialog closes on successful sign-in | Transfer resumes once sign-in completes |
| "Go back" (Error state, deliverable context) | Tap | Navigate to FEAT-06.SPEC-002 | Screen closes | Returns to the deliverable list |
| "View your plan" link (Upload Failed, storage allowance exhausted) | Tap | Navigate to the plan view (FEAT-23) | Screen closes (any file selection is discarded; no version exists) | Plan view opens |

### Accessibility Notes

- **Focus order:** Back arrow -> file picker and its "Choose file" button -> Remove/Retry/plan link (when present) -> View versions -> Cancel (when rendered). The header context (deliverable, milestone, project, current-round label) and the "Round {N} added" badge are announced as static content, not part of the interactive tab order.
- **Status announcements:** Upload progress percentage updates are announced at most once every 10 percentage points, to avoid overwhelming assistive technology with continuous updates. The "Round {N} added" badge and any failure message are announced when they appear.
- **Save feedback:** The "Round {N} added" badge is announced on success and focus moves to View versions; on a failed upload, focus moves to the Retry control where one is rendered, otherwise to Remove.
- **Keyboard alternatives:** The drag-and-drop file target has an equivalent "Choose file" button reachable and operable by keyboard (Interactions); every other action on this screen, including the leave dialog's Leave/Stay and the session-expired dialog's Sign in, is keyboard-reachable.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Picker empty, current-round context shown, View versions button disabled | Screen first opens | User selects a file |
| File Selected / Uploading | File name, size, and live progress bar shown; View versions button disabled | User selects a file | Upload completes, fails, or the file is removed |
| Paused/Resuming | Progress bar shows "Paused -- resuming when your connection returns" | Connectivity drops during transfer (Offline/Degraded, below) | Connectivity returns and the transfer resumes automatically |
| Upload Complete | File name shown with a checkmark and the "Round {N} added" badge; View versions button enabled; Remove and Cancel not rendered. Clients have been notified at this moment (FEAT-06.SPEC-006, via FEAT-17.SPEC-003) | Transfer completes fully (FEAT-17.SPEC-003 reports completion and the version is committed) | User taps View versions or the back arrow |
| Upload Failed | Failure message and controls per the breakdown below; the selected file remains named on screen; the current version, Round {current round_number}, is unchanged and still active; no version exists for the attempt | A transfer fails for a reason other than a dropped connection (see breakdown) | User taps Retry (where rendered), removes the file, or leaves the screen |
| Loading (deliverable context) | Header context area shows a loading placeholder in place of the deliverable/milestone/round labels | Screen first opens while the carried deliverable reference is being resolved | Context resolves, or resolution fails (Error, below) |
| Error (deliverable context) | Full-screen message "This deliverable couldn't be loaded." with a "Go back" action | The carried deliverable reference fails to resolve (e.g., the deliverable was removed between FEAT-06.SPEC-002 and this screen opening) | User taps "Go back" |
| Offline/Degraded | Banner "You're offline -- upload will resume automatically when you reconnect." at top; an in-progress file transfer pauses (see Paused/Resuming) | Connectivity is lost while this screen is open | Connectivity returns |

**Upload Failed breakdown (per failure reason, defined by FEAT-17.SPEC-003 outcomes):**

| Failure Reason | Exact Message | Retry Control | Other Control |
|----------------|---------------|---------------|---------------|
| Size ceiling exceeded (platform parameter: `deliverable-file-size-ceiling`) | "This file is larger than the size limit for deliverables. Compress it or share it by link instead." (text owned by FEAT-17.SPEC-004 / FEAT-06.SPEC-005) | No -- the same file would fail again | Remove |
| Zero-byte file | "This file appears to be empty." | No | Remove |
| Storage allowance exhausted | "You've reached your storage allowance." | No -- retrying cannot succeed until space is freed or the plan changes | "View your plan" link to FEAT-23; Remove |
| Storage capability unavailable | "Uploads aren't available right now. Try again shortly." | Yes | Remove |
| Server rejection or persistent storage error after automatic chunk retries | "Couldn't upload {file name}. The current version, Round {current round_number}, is unchanged and still active." | Yes | Remove |
| Deliverable removed during upload | "This deliverable no longer exists." | No | "Go back" to FEAT-06.SPEC-002 |
| Deliverable not eligible (linked asset or no first upload) | "New versions can't be added to this deliverable." | No | "Go back" to FEAT-06.SPEC-002 |

Every row also states, beneath the message, "Round {current round_number} is unchanged and still active." where not already part of the message.

## Validation Rules

Validation governed by FEAT-17.SPEC-004 (Version Numbering, Immutability & Retention Rules) for round numbering and the version-count/storage ceiling, and by FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules) for the reused per-file size ceiling (platform parameter: `deliverable-file-size-ceiling`). Authorization to reach this screen at all is governed by FEAT-17.SPEC-005. This screen applies no field-level input beyond the file selection itself; it enables the View versions button only once the transfer has completed and the version is committed.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-06.SPEC-002 (Deliverable List & Management) | FEAT-06 (Deliverable Upload & Sharing) |
| View versions tap (after completion) | FEAT-17.SPEC-002 (Version Browser & Comparison), showing the new round as latest | -- |
| Cancel tap with no transfer running | FEAT-06.SPEC-002 (Deliverable List & Management), with no confirmation dialog | FEAT-06 (Deliverable Upload & Sharing) |
| Back arrow or Cancel with a transfer running, then "Leave" | FEAT-06.SPEC-002 (Deliverable List & Management), after the leave confirmation dialog; transfer cancelled | FEAT-06 (Deliverable Upload & Sharing) |
| "Go back" (Error state, or deliverable removed/not eligible failure) | FEAT-06.SPEC-002 (Deliverable List & Management) | FEAT-06 (Deliverable Upload & Sharing) |
| "View your plan" link (storage allowance exhausted) | FEAT-23 plan view | FEAT-23 |

## Data Model

**Creates:** None directly on this screen -- the Deliverable Version record (round_number, file, uploaded_at) is created by FEAT-17.SPEC-003 the instant the transfer completes fully. The View versions button has no data effect; it only navigates.
**Reads:** Deliverable -- name, milestone, project (from entry-point context). Deliverable Version -- the current latest round's round_number and uploaded_at, for the header's "Current: Round {N}" label.
**Updates:** None.
**Deletes:** None.

## Business Rules

- FEAT-17.SPEC-004 governs round numbering, immutability, and retention: the new round always takes the next sequential round_number, and once created neither it nor any prior round can be edited from this or any screen.
- FEAT-17.SPEC-005 governs who may reach this screen at all: only Nadia, as the sole writer of Deliverable Version (dependency map Contention note).
- XBR-13: each re-upload creates a new immutable version; all versions count against the freelancer's storage allowance.
- XBR-14: the per-file size ceiling (platform parameter: `deliverable-file-size-ceiling`) and the per-freelancer storage allowance (platform parameter: `free-tier-storage-allowance` / `paid-tier-storage-allowance`) are enforced during the transfer by FEAT-17.SPEC-003.
- Commit on completion: the new version exists from the moment the transfer completes, and clients are notified at that moment (XBR-12); there is no later confirmation step, and an immutable committed version cannot be undone or "started over" from this screen.
- A failed re-upload never overwrites, replaces, or otherwise touches the existing latest version (feature's States field) -- the prior version remains Active and current throughout.

## Edge Cases

- **Upload fails partway through** -- No Deliverable Version record is ever created for the failed attempt; the deliverable's current latest version remains exactly as it was, visible and unaffected, and the round-number sequence is not advanced or reserved.
- **User navigates away during an in-progress upload** -- The transfer continues in the background if the browser tab remains open; using the back arrow, Cancel, or navigating to a different screen within the product shows the confirmation "Your upload is still in progress. Leave anyway?" with "Leave" (transfer is cancelled, no version created) and "Stay" options. Closing the browser tab entirely cancels the transfer.
- **User closes the tab or loses the session after the transfer completed** -- Nothing is lost: the version was committed at completion, and the next visit to FEAT-17.SPEC-002 shows the new round as latest. Re-opening this screen shows the new round as the current one.
- **File selected, then removed, then a new file selected** -- Each selection starts an independent transfer; only the most recently selected file is tracked, and any prior in-progress transfer for the removed file is cancelled.
- **Two of Nadia's sessions each upload a new version for the same deliverable at effectively the same time** -- No conflict occurs: per the dependency map's Contention note for Deliverable Version ("a new round is appended rather than editing an existing one, so no concurrent modification of a version is possible"), each transfer that completes fully is appended as its own round -- whichever commits first receives the next round_number and the second receives the number after it (FEAT-17.SPEC-004's sequencing). Neither session's upload is rejected or overwritten; both rounds are preserved, and each screen's "Round {N} added" badge shows the number its own transfer received.
- **The underlying deliverable is removed by Nadia in another session while this screen is open** -- Uploading proceeds unaffected only if the deliverable still exists at the moment the transfer commits; if the deliverable no longer exists, the transfer fails with "This deliverable no longer exists." and no version is created (a deliverable on an approved milestone cannot be removed per XBR-11, so this can only occur on a not-yet-approved milestone's deliverable).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-002 (Deliverable List & Management) | Navigation (inbound/outbound) | Entry point via "Upload New Version"; destination on back or cancel |
| FEAT-17.SPEC-003 (Version Creation & Preservation) | Triggers (outbound) | File selection and Retry start the re-upload transfer; this screen displays its live progress, its per-reason failure outcomes, and its completion |
| FEAT-17.SPEC-004 (Version Numbering, Immutability & Retention Rules) | References (inbound) | Round_number assigned to the added round and the size ceiling message shown in this screen's states |
| FEAT-17.SPEC-005 (Version Access & Comment-Anchoring Rules) | References (inbound) | Authorization to reach this screen at all |
| FEAT-17.SPEC-002 (Version Browser & Comparison) | Navigation (outbound) | Destination of View versions after a successful upload, showing the new round as latest |
| FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules) | References (inbound) | Owns the per-file size ceiling and its message text |
| FEAT-16.SPEC-004 (Storage Limits) | References (inbound) | Source of the storage-allowance-exhausted message |
| FEAT-23 (plan view) | Navigation (outbound) | Destination of the "View your plan" link when the storage allowance is exhausted |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| version_upload_started | deliverable reference, file size | Nadia selects a file and the transfer begins | N/A -- no Stage 2 metric measures re-upload initiation specifically; the completion event below is where reliability at scale is measured |
| version_upload_failed | failure reason (size_ceiling / empty_file / storage_allowance / capability_unavailable / server_rejection / deliverable_removed / deliverable_not_eligible), file size | A re-upload transfer fails for a reason other than a dropped connection | N/A -- no Stage 2 metric is connected to this feature's upload-failure path specifically; failures feed the general upload-reliability signal emitted by FEAT-17.SPEC-003 on completion |

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 16 | 16 |
| States | 8 (empty, uploading, paused, complete, failed with 7-reason breakdown, loading, error, offline) | 8 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
