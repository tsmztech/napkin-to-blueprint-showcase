---
document_type: spec
spec_type: screen
spec_id: FEAT-06.SPEC-001
spec_name: Deliverable Upload
spec_slug: deliverable-upload
parent_feature: FEAT-06
parent_feature_name: Deliverable Upload & Sharing
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: Deliverable Upload

## Overview

**Name:** Deliverable Upload
**ID:** FEAT-06.SPEC-001
**Type:** Screen
**Purpose:** Nadia chooses between uploading a file or pasting an external link, and attaches it as a deliverable on a milestone she has already reached.
**Parent Feature:** FEAT-06 -- Deliverable Upload & Sharing

## Scope and Non-Goals

**In Scope:**
- Attaching the first deliverable to a milestone: uploading a file or pasting an external link
- Displaying real upload progress fed by FEAT-06.SPEC-003 (Resumable Upload Handling)
- Showing a link-reachability flag fed by FEAT-06.SPEC-004 (Linked Asset Reachability Check) before save completes
- Preserving a selected file and offering retry without re-selection when an upload fails

**Non-Goals:**
- Listing, previewing, removing, or replacing existing deliverables -- handled by FEAT-06.SPEC-002 (Deliverable List & Management)
- Choosing or changing the milestone from an unscoped starting point -- excluded because both of this screen's entry points (FEAT-06.SPEC-002, FEAT-04.SPEC-001) always carry a milestone reference; the Feature Breakdown Brief's Default Entry and Internal Dependency Map define no milestone-less path into this screen
- Re-uploading a new version of an existing Active deliverable -- excluded per the Brief's Internal Dependency Map: "replace" hands off into FEAT-17 (Deliverable Version History), which owns round 2+ creation (XBR-13)
- Copying or mirroring the content of a linked Figma, Google Drive, or Dropbox asset into the product -- excluded per scope-boundaries.md (SC-08): linked assets are referenced by URL and checked for reachability only

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-002 (Deliverable List & Management) | Nadia taps "Upload Deliverable" from a milestone's deliverable list (including its empty-state prompt) | Milestone reference, locked as the target for the new deliverable |
| FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | Nadia opens "Upload Deliverable" directly from a milestone row | Milestone reference, locked as the target for the new deliverable |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Select a file or paste a link, and submit the upload | -- |
| Owen (Client Primary Contact) | No | No | This screen is never part of the client portal's navigation (FEAT-05, Client Portal Access); a direct link shows "This page isn't part of your portal." and returns Owen to his portal home |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- this screen is never part of the client portal's navigation; a direct link returns Priya to her portal home |
| Dana (Support Operator) | No | No | Dana's read-only support session (FEAT-31) renders this feature's deliverable list (FEAT-06.SPEC-002) but never an upload action; the "Upload Deliverable" entry point itself is not rendered inside a support session |
| Unauthenticated | No | No | Redirected to the freelancer sign-in screen; no milestone context or in-progress selection is preserved |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- a selected file or entered link text is preserved locally and restored after re-authentication succeeds; an in-flight upload transfer resumes only once sign-in completes |

## Layout and Content

**Header:** Screen title "Upload Deliverable" with a back arrow (returns to the entry point per the Navigation Out table) and the read-only milestone context: milestone name and project name, non-interactive.

**Body:** A single-column form with:
- **Upload method toggle** -- two options, "Upload a file" (default) and "Paste a link", presented as a single choice per the Brief's Shared UI Pattern ("Spec Writers should describe both paths as one screen, not two")
- **File path (shown when "Upload a file" is selected):** a file picker control with a drop target; once a file is chosen, its name and size replace the picker with a progress indicator area below it (queued / uploading with percentage / paused-resuming / complete / failed -- the same vocabulary FEAT-06.SPEC-003 reports) and a "Remove" control to clear the selection before upload starts
- **Link path (shown when "Paste a link" is selected):** a single text input for the external URL (Figma, Google Drive, or Dropbox), with inline space below it for the reachability flag once FEAT-06.SPEC-004 reports back
- **Deliverable status badge** -- display-only; shown once a file completes upload or a link is confirmed reachable, using the same status/link_status vocabulary FEAT-06.SPEC-002 displays (Uploading, Active; reachable/flagged)

**Footer:** "Attach Deliverable" primary action button (disabled until a file is selected or a link is entered) and a "Cancel" secondary action.

The header's milestone/project name context is display-only.

### Responsive Behavior

- **Compact breakpoint:** Single-column form, full width; header context and footer actions remain visible without scrolling past the fold for the upload-method toggle.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.
- **Progress indicator:** Uniform scaling, no structural change -- the progress bar and percentage text remain a single row at both breakpoints.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the entry point (Navigation Out) | Screen closes | Returns to the milestone's deliverable list or the milestone editor |
| Upload method toggle | Tap "Upload a file" / "Paste a link" | Switches the visible input path; clears any partial input from the other path | Body region re-renders with the selected path's fields | Selected option highlighted; the other path's fields are hidden, not just disabled |
| File picker / drop target | Tap or drag-and-drop a file | Captures the selected file locally and immediately begins the resumable transfer via FEAT-06.SPEC-003 | Picker is replaced by the file's name, size, and a progress indicator | Progress shows "Uploading... {percentage}%" and updates in real time |
| Remove (queued or in-progress file) | Tap | Cancels the in-progress transfer (FEAT-06.SPEC-003) and clears the selection | Progress area is replaced by the empty file picker | File picker reappears empty |
| Link text input | Type or paste a URL | Captures the link text | Field shows entered text | Standard input focus state |
| Link text input | Blur | Triggers link format validation via FEAT-06.SPEC-005 | Error state on field if malformed | "Enter a valid Figma, Google Drive, or Dropbox link" below the field when the format is invalid |
| Attach Deliverable button | Tap (file path, upload complete) | Creates the Deliverable record and its first Deliverable Version (already committed by FEAT-06.SPEC-003 on upload completion); confirms the attachment | Button shows a brief confirming state | Success: toast "Deliverable attached" and navigate per Navigation Out |
| Attach Deliverable button | Tap (file path, upload still in progress) | No action -- button stays disabled while a transfer is running | None | Button remains disabled with the label "Uploading..." |
| Attach Deliverable button | Tap (link path) | Creates the Deliverable record in pending-reachability status and triggers FEAT-06.SPEC-004 (Linked Asset Reachability Check) | Button shows a brief "Checking link..." state | On reachable: toast "Deliverable attached" and navigate per Navigation Out. On flagged: see States (Link Flagged) |
| Attach Deliverable button (while checking or saving) | Tap | No action -- debounced | None | Button remains in its transient state |
| Retry (failed upload) | Tap | Re-attempts the transfer for the already-selected file without requiring re-selection (FEAT-06.SPEC-003) | Progress area returns to "Uploading..." | Progress resumes from the last successfully transferred point where the transfer protocol allows it, or restarts the transfer if no partial state survived |
| Cancel | Tap | Cancels any in-progress transfer or link check and discards the unsaved form | Screen closes | Navigates to the same destination as the back arrow, with no confirmation dialog (no partial save exists to lose) |

### Accessibility Notes

- **Focus order:** Back arrow -> upload method toggle -> (file picker or link input, depending on selection) -> Remove/Retry control (when present) -> Attach Deliverable -> Cancel. The status badge and header milestone/project context are announced as static content, not part of the interactive tab order.
- **Validation and status announcements:** Link format errors and the "flagged as unreachable" message are announced to assistive technology and programmatically associated with the link input. Upload progress percentage updates are announced at most once every 10 percentage points, to avoid overwhelming assistive technology with continuous updates.
- **Save feedback:** The "Deliverable attached" toast is announced on success; on a failed upload, focus moves to the Retry control.
- **Keyboard alternatives:** The drag-and-drop file target has an equivalent "Choose file" button reachable and operable by keyboard; every other action on this screen is keyboard-reachable.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Upload method toggle set to "Upload a file", picker empty, Attach button disabled | Screen first opens | User selects a file or switches to the link path |
| File Selected / Uploading | File name, size, and live progress bar shown; Attach button disabled | User selects a file | Upload completes, fails, or the file is removed |
| Paused/Resuming | Progress bar shows "Paused -- resuming when your connection returns" | Connectivity drops during transfer (Offline/Degraded, below) | Connectivity returns and the transfer resumes automatically |
| Upload Complete | File name shown with a checkmark and the Active status badge; Attach button enabled | Transfer completes fully (FEAT-06.SPEC-003 reports completion) | User taps Attach Deliverable, or removes the file to start over |
| Upload Failed | Error banner "Couldn't upload {file name}. Check your connection and try again." with a Retry control; the selected file remains named on screen | A transfer fails for a reason other than a dropped connection (server rejection, size ceiling exceeded per FEAT-06.SPEC-005) | User taps Retry, or removes the file |
| Link Entry | Link input empty or being typed, Attach button disabled until non-empty | User selects "Paste a link" | User enters a link and blurs the field |
| Link Checking | Attach button shows "Checking link..."; input is read-only during the check | User taps Attach Deliverable with a link entered | FEAT-06.SPEC-004 reports reachable or flagged |
| Link Flagged | Inline message below the link field: "This link couldn't be reached. Check that it's shared and try again." Input becomes editable again; Attach button re-enabled | FEAT-06.SPEC-004 reports the link unreachable | User edits the link and re-submits |
| Loading (milestone context) | Header context area shows a loading placeholder in place of the milestone/project name | Screen first opens while the carried milestone reference is being resolved | Milestone context resolves, or resolution fails (Error, below) |
| Error (milestone context) | Full-screen message "This milestone couldn't be loaded." with a "Go back" action | The carried milestone reference fails to resolve (e.g., milestone removed between entry-point load and this screen opening) | User taps "Go back" |
| Offline/Degraded | Banner "You're offline -- upload will resume automatically when you reconnect." at top; an in-progress file transfer pauses (see Paused/Resuming); the link path is disabled while offline, since reachability cannot be checked without connectivity | Connectivity is lost while this screen is open | Connectivity returns |

## Validation Rules

Validation governed by FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules). See that spec for the milestone-required rule, link format and reachability rules, and the file size ceiling. This screen applies field-level checks on blur (link format) and gates the Attach Deliverable action on full validation.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-06.SPEC-002 (Deliverable List & Management) or FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor), whichever entry point was used | FEAT-04 (Milestone & Payment Schedule Setup) for the second case |
| Successful attach (file or link) | FEAT-06.SPEC-002 (Deliverable List & Management), showing the newly Active deliverable | -- |
| Cancel | Same destination as the back arrow, with no confirmation dialog (no partial save exists to lose -- an in-progress upload is cancelled per the Remove interaction) | -- |

## Data Model

**Creates:** Deliverable record -- kind (uploaded file or linked external asset), milestone (from entry-point context), uploaded_at (system timestamp). For a file: status set to Uploading at creation, becoming Active on full transfer completion (FEAT-06.SPEC-003); size captured from the completed transfer. For a link: link_status set to pending, becoming reachable or flagged (FEAT-06.SPEC-004).
**Reads:** Milestone -- name, project reference, for the locked header context.
**Updates:** None (this screen only creates a new Deliverable; updates to status and link_status are performed by FEAT-06.SPEC-003 and FEAT-06.SPEC-004 respectively, surfaced here as live state).
**Deletes:** None.

## Business Rules

- Milestone required (FEAT-06.SPEC-005): the milestone is always pre-supplied by an entry point; this screen never allows submission without one.
- File size ceiling (FEAT-06.SPEC-005, platform parameter: `deliverable-file-size-ceiling`) is enforced by FEAT-06.SPEC-003 during transfer; a file exceeding it surfaces as the Upload Failed state with the ceiling-specific message from FEAT-06.SPEC-005.
- Link format and reachability (FEAT-06.SPEC-005, FEAT-06.SPEC-004): a link is never marked ready, and the client is never notified, until it resolves as reachable.
- XBR-12: the client is notified only once the upload (FEAT-06.SPEC-003) or link check (FEAT-06.SPEC-004) fully completes -- this screen's own "Deliverable attached" confirmation to Nadia is independent of, and can occur slightly before, the client-facing notification (FEAT-06.SPEC-006).
- This screen only ever creates the first Deliverable for a milestone; if a milestone already has an Active deliverable, Nadia reaches replacement through FEAT-06.SPEC-002's "replace" action into FEAT-17, not through this screen.

## Edge Cases

- **User switches upload method mid-selection** -- Switching from "Upload a file" to "Paste a link" (or back) while a file transfer is in progress cancels that transfer, per the Remove interaction; a warning is not shown because no data has yet been attached to the deliverable.
- **User navigates away during an in-progress upload** -- The transfer continues in the background if the browser tab remains open; navigating to a different screen within the product shows a confirmation: "Your upload is still in progress. Leave anyway?" with "Leave" (transfer is cancelled) and "Stay" options. Closing the browser tab entirely cancels the transfer.
- **User taps Attach Deliverable twice rapidly** -- Second tap is ignored while the first attach/check is in progress (button in a transient state).
- **File selected, then removed, then a new file selected** -- Each selection starts an independent transfer; only the most recently selected file is tracked, and any prior in-progress transfer for the removed file is cancelled.
- **Two upload attempts on the same milestone from two sessions of Nadia's** -- Both sessions can begin an upload independently; whichever commits first becomes the milestone's Active deliverable (round 1). The second session's Attach Deliverable action is rejected-with-refresh: "This milestone already has a deliverable. Refresh to see it, or use Replace to add a new version." with a link to FEAT-06.SPEC-002, consistent with the dependency map's Contention note for Deliverable (Nadia is the only writer; contended moments are re-checked at commit).
- **Milestone is approved while this screen is open (no existing deliverable)** -- Attaching the first deliverable to an already-Approved milestone is not blocked; XBR-11's removal block applies only once a deliverable exists, and this screen never removes one.
- **Link entered for a domain outside Figma, Google Drive, or Dropbox** -- FEAT-06.SPEC-005's format rule rejects it inline: "Enter a Figma, Google Drive, or Dropbox link." before any reachability check is attempted.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-002 (Deliverable List & Management) | Navigation (inbound/outbound) | Entry point when arriving to add a deliverable; destination after a successful attach |
| FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | Navigation (inbound) | Alternate entry point, carrying the milestone reference |
| FEAT-06.SPEC-003 (Resumable Upload Handling) | Triggers (outbound) | File selection starts the resumable transfer; this screen displays its live progress |
| FEAT-06.SPEC-004 (Linked Asset Reachability Check) | Triggers (outbound) | Attaching a link starts the reachability check; this screen displays the flagged outcome |
| FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules) | References (inbound) | Field validation, size ceiling, and link format rules |
| FEAT-06.SPEC-006 (Deliverable Ready Notification) | References (outbound) | Successful completion (via FEAT-06.SPEC-003 or FEAT-06.SPEC-004) is the event this notification waits for |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| deliverable_uploaded | upload method (file), file size, milestone reference | A file transfer completes fully and the deliverable becomes Active | supports success-metrics.md: "Deliverable Upload Reliability" |
| deliverable_linked | upload method (link), link domain (figma / drive / dropbox) | A pasted link is confirmed reachable and the deliverable becomes Active | N/A -- no Stage 2 metric measures linked-asset attachment specifically; retained for feature-level visibility since Deliverable Upload Reliability is defined in terms of file transfers |
| upload_failed | failure reason (size_ceiling / server_rejection), file size | An upload fails for a reason other than a dropped connection | supports success-metrics.md: "Deliverable Upload Reliability" |
| upload_resumed | pause duration | A paused transfer resumes automatically after a dropped connection | supports success-metrics.md: "Deliverable Upload Reliability" |

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 12 | 12 |
| States | 11 (empty, file selected/uploading, paused/resuming, complete, failed, link entry, link checking, link flagged, loading, error, offline) | 11 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |
