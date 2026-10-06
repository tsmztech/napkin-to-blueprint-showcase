# FEAT-06 — Deliverable Upload & Sharing

This chapter covers Deliverable Upload & Sharing, a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 6 specifications carrying 79 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-06.SPEC-001 | Deliverable Upload | screen | 14 |
| FEAT-06.SPEC-002 | Deliverable List & Management | screen | 13 |
| FEAT-06.SPEC-003 | Resumable Upload Handling | automation | 11 |
| FEAT-06.SPEC-004 | Linked Asset Reachability Check | automation | 9 |
| FEAT-06.SPEC-005 | Deliverable Validation & Removal Eligibility Rules | logic-rule | 20 |
| FEAT-06.SPEC-006 | Deliverable Ready Notification | notification | 12 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Deliverable Upload & Sharing

## Summary

**Feature:** Deliverable Upload & Sharing
**ID:** FEAT-06
**Description:** The freelancer uploads a file — a design file, video, or PDF, up to large sizes — or attaches a link to Figma, Google Drive, or Dropbox, and attaches it to a milestone; the client is notified it is ready to review.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Problem Statement centers on files scattered across "final_v3_REAL" versions in a shared drive; this feature is the direct fix. MVP phase: without it, there is nothing for the client to approve. [RESEARCH-INFORMED: none of the 5 profiled competitors markets deliverable-specific large-file handling or version history beyond generic attachments (Feature Landscape, Absent Features) — a gap this product fills directly] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Upload a file — design file, video, or PDF, resumable for large sizes
- Link an external asset — attach a Figma, Drive, or Dropbox link instead of copying the file in
- Notify the client — the relevant contacts are emailed when a deliverable is ready
- Remove or replace a deliverable — withdraw a deliverable shared by mistake, or upload a new version (FEAT-17) [AUDIT-ADDED: 3 -- entity coverage: Deliverable had no removal path]

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-06.SPEC-001 | Deliverable Upload | Screen | Nadia | Nadia chooses a milestone and uploads a file or pastes an external link to attach a deliverable |
| FEAT-06.SPEC-002 | Deliverable List & Management | Screen | Nadia | Nadia views a milestone's deliverables, previews them where feasible, and removes or initiates replacement of one |
| FEAT-06.SPEC-003 | Resumable Upload Handling | Automation | Nadia | Processes a file upload in a resumable, progress-tracked way, pausing and auto-resuming through dropped connections, and marks the deliverable ready only on full completion |
| FEAT-06.SPEC-004 | Linked Asset Reachability Check | Automation | Nadia | Validates a pasted external link resolves before the deliverable is marked ready, flagging an unreachable link to the freelancer |
| FEAT-06.SPEC-005 | Deliverable Validation & Removal Eligibility Rules | Logic/Rule | Nadia, Owen, Priya, Dana | Governs upload validity (milestone required, valid/reachable link, size ceiling), who may upload or remove, and the approved-milestone removal block |
| FEAT-06.SPEC-006 | Deliverable Ready Notification | Notification | Owen, Priya | Emails the relevant client contacts once an uploaded or linked deliverable is fully ready to review |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Upload a file | FEAT-06.SPEC-001, FEAT-06.SPEC-003 | The upload screen collects the file and milestone; the resumable-upload automation processes and completes the transfer | Phase 2 (Explicit) |
| Link an external asset | FEAT-06.SPEC-001, FEAT-06.SPEC-004 | The upload screen accepts a pasted link; the reachability-check automation validates it before the deliverable is marked ready | Phase 2 (Explicit) |
| Notify the client | FEAT-06.SPEC-006 | Dedicated Notification spec emails relevant client contacts once the deliverable is fully ready | Phase 2 (Explicit) |
| Remove or replace a deliverable | FEAT-06.SPEC-002, governed by FEAT-06.SPEC-005 | The management screen exposes remove and replace actions; SPEC-005 blocks removal on an approved milestone and forces supersession instead | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-06.SPEC-003 | Resumable Upload Handling | Phase 4 (Trigger-Response Analysis) | The States and Primary Flows & Alternates fields require real progress, automatic pause/resume across a dropped connection, and completion-gated notification (XBR-12) — processing logic with failure modes, not a single direct write, crossing the standalone-Automation threshold |
| FEAT-06.SPEC-004 | Linked Asset Reachability Check | Phase 4 (Trigger-Response Analysis) | Primary Flows & Alternates states an unreachable link "is flagged to the freelancer before it is shown to the client as broken" — an asynchronous check against an external resource with its own failure mode, not a simple field validation |
| FEAT-06.SPEC-005 | Deliverable Validation & Removal Eligibility Rules | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits and Access fields carry conditional rules (milestone required, link validity, size ceiling, role-gated write access, approved-milestone removal block) that apply across both write paths (SPEC-001, SPEC-002) and cross the 5+/conditional-logic threshold for a standalone spec |
| FEAT-06.SPEC-006 | Deliverable Ready Notification | Phase 4 (Notification surfacing lens) | The Communications field names a channel (email), an audience (relevant client contacts), content, and a delivery-timing rule (XBR-12, never for a partial file) — more than a same-screen confirmation, requiring a standalone Notification spec |

## Entity-Lifecycle Coverage Matrix

**Entity: Deliverable**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-06.SPEC-001, FEAT-06.SPEC-003 | Upload screen collects milestone, file or link; the record is created in `Uploading` status (for a file) or pending-reachability status (for a link) at submission | Also the point at which the first Deliverable Version is created (see below) |
| Read (single) | FEAT-06.SPEC-002 | Management screen opens one deliverable's detail (status, link_status, preview) | Also read by FEAT-05, FEAT-07, FEAT-08, FEAT-17, FEAT-28, FEAT-31 outside this feature -- their screens are out of this feature's scope |
| Read (list) | FEAT-06.SPEC-002 | Management screen lists all deliverables for a milestone, with an empty-state prompt when none exist yet | The client-facing deliverable view is owned by FEAT-07, not this feature (dependency map: Deliverable is read by FEAT-05/07/08/17/28/31, not by a FEAT-06 client screen) |
| Update | FEAT-06.SPEC-003 (status Uploading -> Active), FEAT-06.SPEC-004 (link_status set to reachable/flagged) | Resumable-upload completion and the reachability check are the only in-feature updates to an existing Deliverable | Supersession on replace (status -> Superseded) is executed by FEAT-17's re-upload flow, not re-implemented here (XBR-13) |
| Delete/Archive | FEAT-06.SPEC-002, governed by FEAT-06.SPEC-005 | Soft removal: the remove action sets status to `Removed`; the record itself, its file/link reference, and its Deliverable Versions are retained (never hard-deleted) because the upload/removal event remains part of the evidentiary activity trail (XBR-05, journey "Pointing to the Record in a Scope Dispute"). No restore path is defined -- a removed deliverable stays Removed; recovery is a fresh upload, not an undo. No cascade: pinned Comments and Deliverable Versions are left in place, governed by their own lifecycle rules. Retention: kept for the life of the account with no automatic purge (ASMP-22), consistent with the entity's evidentiary role; blocked entirely once the owning milestone is Approved -- only supersession (FEAT-17) is available then (XBR-11) | Coordination point with FEAT-08 (approval) and FEAT-17 (supersede) noted in Shared Context |
| State Transition | FEAT-06.SPEC-003 (Uploading -> Active) | Sets Active once upload or link validation fully completes | Active -> Superseded is owned by FEAT-17; Active -> Removed is owned by FEAT-06.SPEC-002 gated by FEAT-06.SPEC-005 |

**Entity: Deliverable Version**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-06.SPEC-003 | On the deliverable's first successful upload, a Deliverable Version record is created at round_number 1, carrying the uploaded file | Re-upload versions (round_number 2+) are created by FEAT-17, out of this feature's scope |
| Read (single) | N/A | Not read by any FEAT-06 spec; read by FEAT-07, FEAT-17, FEAT-31 outside this feature | -- |
| Read (list) | N/A | Version history browsing belongs to FEAT-17 (Deliverable Version History), not this feature | -- |
| Update | N/A | The dependency map states Deliverable Version is "never updated (immutable once uploaded)" -- an explicit product decision, not a gap | -- |
| Delete/Archive | N/A | "Never deleted in-product; removed only by FEAT-24" (dependency map) -- an intentional design decision recorded in Non-Goals, not an omission | -- |
| State Transition | N/A | `is_latest` is a derived field, not a stored state; the entity carries no status field | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Milestone | FEAT-06.SPEC-001, FEAT-06.SPEC-002 | Both screens are scoped to one milestone: the upload screen attaches a new deliverable to the chosen milestone; the management screen lists a milestone's deliverables and checks its approval state before allowing removal |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia selects a milestone and starts a file upload | Create the Deliverable record (Uploading), begin resumable, progress-tracked transfer | Standalone Automation | FEAT-06.SPEC-003 |
| An in-progress upload's connection drops | Pause the transfer and resume automatically from where it left off once connectivity returns, without restarting from zero | Standalone Automation | FEAT-06.SPEC-003 |
| A file upload completes fully | Mark the Deliverable Active, create its first Deliverable Version (round 1), emit `deliverable_uploaded` | Standalone Automation | FEAT-06.SPEC-003 |
| A file upload completes fully or a linked asset is confirmed reachable | Notify the relevant client contacts that a deliverable is ready to review | Standalone Notification | FEAT-06.SPEC-006 |
| Nadia pastes an external link and saves | Check that the link resolves and is reachable | Standalone Automation | FEAT-06.SPEC-004 |
| A pasted link fails to resolve | Flag it to Nadia before save; the deliverable is never marked ready or shown to the client as broken; emit `upload_failed` | Standalone Automation | FEAT-06.SPEC-004 |
| A file upload fails (e.g., server rejection, size ceiling exceeded) | Preserve the selected file on-screen and offer retry without re-selecting it | Inline in triggering screen | FEAT-06.SPEC-001 |
| Nadia attempts to remove a deliverable | Check whether the owning milestone is Approved | Standalone Logic/Rule | FEAT-06.SPEC-005 |
| Removal is allowed (milestone not yet approved) | Set the deliverable's status to Removed; write an activity-trail entry | Standalone Logic/Rule (write itself stays inline in FEAT-06.SPEC-002) | FEAT-06.SPEC-005 |
| Removal is blocked (milestone already approved) | Refuse the action; direct Nadia to replace/supersede instead | Standalone Logic/Rule | FEAT-06.SPEC-005 |
| Nadia chooses "replace" on an existing deliverable | Hand off into the re-upload flow that creates the next Deliverable Version | Cross-feature | FEAT-17 responsibility |
| Any file upload or stored version | Consumes the freelancer's per-file size ceiling and overall storage allowance | Cross-feature | FEAT-16 responsibility (owns the large-file storage Integration spec, pending batch validation per the dependency map's External Touchpoints) |
| FEAT-06.SPEC-006 dispatches a notification | Delivered through the product's transactional email capability, with delivery/bounce status reported back | Cross-feature | FEAT-14 responsibility (owns the transactional-email Integration spec) |
| A deliverable is uploaded or removed | An append-only activity-trail entry is written with actor and timestamp (XBR-05) | Cross-feature | FEAT-13 responsibility |
| A milestone has no deliverables yet | Nadia sees a prompt to upload the first one | Inline in triggering screen | FEAT-06.SPEC-002 |

## Shared Context

**Shared Entities:**
- Deliverable -- created by SPEC-001/SPEC-003, updated by SPEC-003 (status) and SPEC-004 (link_status), listed/removed by SPEC-002 governed by SPEC-005. Fields: kind, file or link, milestone, uploaded_at, size, status, link_status, first_client_view_at (written by FEAT-13, not this feature).
- Deliverable Version -- created (round 1 only) by SPEC-003. Fields: round_number, file, uploaded_at, is_latest (derived). Later rounds and all reads belong to FEAT-17.

**Shared UI Patterns:**
- Upload/link entry -- SPEC-001 offers a single choice between a file picker (feeding SPEC-003) and a link field (feeding SPEC-004); Spec Writers should describe both paths as one screen, not two.
- Deliverable status badge -- the same status/link_status vocabulary (Uploading, Active, Superseded, Removed; reachable/flagged) is shown by SPEC-002 and referenced by SPEC-003, SPEC-004, and SPEC-005; Spec Writers should keep the vocabulary and visual treatment consistent rather than re-deriving it per spec.
- Progress indicator -- SPEC-001 displays the real, resumable progress that SPEC-003 reports; the two specs should describe the same progress states (queued, uploading with percentage, paused/resuming, complete, failed) consistently.

**Shared Validation:**
- SPEC-005 defines every upload-validity and removal-eligibility rule (milestone required, valid/reachable link, size ceiling, role-gated writes, approved-milestone removal block). SPEC-001, SPEC-002, SPEC-003, and SPEC-004 all reference SPEC-005 for these checks rather than duplicating them.

**Flagged discrepancy (not resolved by this Brief):** The Deliverable entity's `first_client_view_at` field is written by FEAT-13 on a client contact's first view, not by any spec in this feature; it is listed here only because it lives on the entity this feature creates. This is surfaced for the Requirements Architect, not resolved by the Feature Analyst.

## Internal Dependency Map

```
SPEC-002 (Deliverable List & Management) -> [Nadia selects "Upload Deliverable"] -> SPEC-001 (Deliverable Upload)
SPEC-001 (Deliverable Upload) -> [Nadia uploads a file] -> SPEC-003 (Resumable Upload Handling) -> [upload completes] -> SPEC-006 (Deliverable Ready Notification)
SPEC-003 (Resumable Upload Handling) -> [upload completes] -> SPEC-002 (deliverable now shows Active)
SPEC-001 (Deliverable Upload) -> [Nadia pastes a link] -> SPEC-004 (Linked Asset Reachability Check) -> [link reachable] -> SPEC-006 (Deliverable Ready Notification)
SPEC-004 (Linked Asset Reachability Check) -> [link unreachable] -> SPEC-001 (flag shown to Nadia before save)
SPEC-001 (Deliverable Upload) -> [validates milestone, file/link, and size using] -> SPEC-005 (Deliverable Validation & Removal Eligibility Rules)
SPEC-002 (Deliverable List & Management) -> [Nadia removes a deliverable] -> SPEC-005 (Deliverable Validation & Removal Eligibility Rules) -> [allowed/blocked] -> SPEC-002
SPEC-002 (Deliverable List & Management) -> [Nadia chooses "replace"] -> FEAT-17 (Deliverable Version History re-upload flow)
```

**Default Entry:** SPEC-002 (Deliverable List & Management) -- the screen Nadia reaches from a milestone's deliverables area; SPEC-001 is reached from there to add a new deliverable.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-06.SPEC-001 | Inbound | FEAT-04 (Milestone & Payment Schedule Setup) | Nadia navigates from a milestone's view into the upload screen | Nadia opens "Upload Deliverable" on a milestone |
| FEAT-06.SPEC-006 | Outbound | FEAT-05 (Client Portal Access & Magic-Link Login) | The deliverable-ready email links into magic-link sign-in and on to the deliverable view | A contact opens the deliverable-ready email |
| FEAT-06.SPEC-002 | Outbound | FEAT-07 (Deliverable Review & Feedback) | The client-facing review and comment screen reads the deliverables this feature manages | A contact opens the deliverable view |
| FEAT-06.SPEC-002 | Outbound | FEAT-08 (Milestone Approval) | Milestone approval checks that a deliverable exists and reads its Active status | Owen reviews a milestone for approval |
| FEAT-06.SPEC-005 | Inbound | FEAT-08 (Milestone Approval) | An Approved milestone gates removal, forcing supersession instead (XBR-11) | Owen approves the milestone |
| FEAT-06.SPEC-005 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Every upload and removal writes an append-only trail entry with actor and timestamp (XBR-05) | Deliverable uploaded or removed |
| FEAT-06.SPEC-002 | Outbound | FEAT-14 (Notifications, Email) | FEAT-06.SPEC-006 dispatches through the shared transactional-email delivery capability | A deliverable becomes Active |
| FEAT-06.SPEC-002 | Outbound | FEAT-17 (Deliverable Version History) | The "replace" action hands off into version re-upload; supersession and later version reads belong there | Nadia chooses "replace" on an existing deliverable |
| FEAT-06.SPEC-003 | Outbound | FEAT-16 (Large File Handling & Storage) | Every upload consumes the shared large-file storage capability and is bounded by its per-file/storage limits (XBR-14) | Any file upload |
| FEAT-06.SPEC-002 | Inbound | FEAT-31 (Operator Support Access) | Dana's read-only, logged support session displays this feature's deliverable list without file download (XBR-29) | Dana opens a support session on the freelancer's account |
| FEAT-06.SPEC-002 | Outbound | FEAT-24 (Account Deletion) | Deliverable and Deliverable Version records are deleted as part of account deletion | Nadia's account is deleted |

## Non-Functional Notes

**Data volumes / growth:** Deliverables typically run tens of MB and sometimes over 1 GB for video, retained with full version history for the life of the account (ASMP-22); this feature's screens and automations must stay reliable at that per-file size, not just at ordinary document sizes.

**Responsiveness:** Upload progress must be real and visible for large files, never an indefinite spinner (feature's States field, ASMP-27); a dropped connection pauses and resumes automatically rather than forcing a restart (ASMP-27's offline posture). This feature's own screens serve Nadia; the client-facing review screen's 2-second interactivity target (ASMP-21) is FEAT-07's responsibility, not restated here.

**Data sensitivity / privacy:** Deliverables are client-confidential work product (design files, videos, documents) that may themselves contain personal data, strictly isolated per client (ASMP-23); the file or link content is never exposed across client boundaries, and the Support Operator can list deliverables but never download the underlying file (ASMP-23, XBR-29).

**Compliance flags:** GDPR-class handling applies to any personal data a deliverable's content or metadata may carry (ASMP-24); no card or payment data is ever involved in this feature.

## Non-Goals

- **Copying or hosting files from Figma, Google Drive, or Dropbox** -- Excluded per scope-boundaries.md (SC-08): linked assets are referenced by URL and their reachability is checked; their content is never mirrored into the product.
- **Scoped upload or removal permissions for a freelancer-side team** -- Excluded per scope-boundaries.md (SC-01): the product is solo-freelancer only for v1, so this feature has exactly one writer role (Nadia); no bookkeeper- or contractor-style scoped upload permission exists.
- **Operator editing, deleting, or downloading deliverables** -- Excluded per scope-boundaries.md (SC-04): Dana's support session is read-only and never downloads deliverable files, matching XBR-29.
- **Restoring a removed deliverable** -- Intentional lifecycle decision surfaced by the CRUD matrix: removal is a one-way status change kept for evidentiary purposes (activity trail, XBR-05); the product's recovery path is a fresh upload or a new version (FEAT-17), not an undo of the removal itself.
- **Automatic purge of removed, superseded, or old-version deliverables** -- Intentional lifecycle decision surfaced by the CRUD matrix: deliverables and their versions are retained for the life of the account with no automatic purge (ASMP-22), since a removed or superseded record remains part of the project's evidentiary history.
- **Native mobile upload app** -- Excluded per scope-boundaries.md (SC-06): the product is a web app with no native apps; large-file upload happens through the web app's mobile-and-desktop-browser experience.



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



# Screen Spec: Deliverable List & Management

## Overview

**Name:** Deliverable List & Management
**ID:** FEAT-06.SPEC-002
**Type:** Screen
**Purpose:** Nadia views a milestone's deliverables, previews them where feasible, and removes or initiates replacement of one.
**Parent Feature:** FEAT-06 -- Deliverable Upload & Sharing

## Scope and Non-Goals

**In Scope:**
- Listing all deliverables attached to one milestone, in upload order
- Previewing a deliverable's file or link where feasible, and its current status/link_status
- Removing a deliverable, governed by FEAT-06.SPEC-005's approved-milestone block
- Initiating replacement, which hands off into FEAT-17's re-upload flow
- The empty-state prompt when a milestone has no deliverables yet
- Rendering read-only for Dana's support session (FEAT-31), without file download

**Non-Goals:**
- The client-facing deliverable review and comment view -- owned by FEAT-07 (Deliverable Review & Feedback); this screen serves Nadia only (dependency map: the client-facing deliverable view is owned by FEAT-07, not this feature)
- Uploading a first deliverable -- handled by FEAT-06.SPEC-001 (Deliverable Upload), reached from this screen's "Upload Deliverable" action
- Executing the replacement upload itself -- excluded per the Brief's Internal Dependency Map: "Nadia chooses 'replace'" hands off into FEAT-17 (Deliverable Version History), which owns the re-upload screen and round 2+ creation
- Restoring a removed deliverable -- intentional lifecycle decision per the Brief's Entity-Lifecycle Coverage Matrix: removal is one-way, kept for evidentiary purposes (XBR-05); recovery is a fresh upload or a new version, never an undo

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-005 (Project Detail, milestones area) | Nadia opens a milestone's deliverables area | Milestone reference |
| FEAT-06.SPEC-001 (Deliverable Upload) | Successful attach (file or link) | Milestone reference, newly Active deliverable highlighted |
| FEAT-31 (Operator Support Access) | Dana opens a read-only support session and navigates to a milestone's deliverables | Milestone reference; screen renders in read-only mode |
| FEAT-04.SPEC-002 (Milestone Timeline (Client View)) | Nadia or Dana taps a milestone row (any milestone status; Dana's rendering is read-only inside her support session) | Milestone reference |
| FEAT-28.SPEC-001 (Global Search) | Nadia selects a Deliverable search result | Milestone reference; the screen opens scrolled and focused to that deliverable |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Upload (navigate to FEAT-06.SPEC-001), preview, remove, initiate replace | -- |
| Owen (Client Primary Contact) | No | No | This screen is never part of the client portal's navigation (FEAT-05, Client Portal Access); the equivalent client-facing view is FEAT-07 (Deliverable Review & Feedback), which he reaches instead |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- reaches FEAT-07 instead of this screen |
| Dana (Support Operator) | Full screen, rendered read-only inside her logged support session (FEAT-31) | View list and status badges only; preview is available without file download; Upload, Remove, and Replace controls are not rendered (XBR-29) | Attempting a write action is not possible -- no write controls exist in this rendering; a direct attempt at the underlying action is refused with "Support sessions are read-only." |
| Unauthenticated | No | No | Redirected to the freelancer sign-in screen; no milestone context is preserved |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- the currently viewed milestone is restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title showing the milestone name and project name, with an "Upload Deliverable" action button (top right; hidden entirely in Dana's read-only rendering).

**Body:** A vertical list of deliverable cards, one per deliverable attached to this milestone, most recent first. Each card shows:
- Kind icon and a preview thumbnail matched to the file's type where feasible (a still image for an image file, the first frame for a video, the first page for a PDF); for a linked asset, the source icon (Figma, Google Drive, or Dropbox) in place of a thumbnail -- display-only
- Deliverable status badge (Uploading, Active, Superseded, Removed) and, for links, the link_status badge (reachable / flagged) -- display-only; shared vocabulary and visual treatment with FEAT-06.SPEC-001 and FEAT-06.SPEC-003/FEAT-06.SPEC-004
- Uploaded date/time and file size (for uploaded files) -- display-only
- Action row: "Preview" (or "Open link" for a linked asset), "Replace", and "Remove" -- each hidden for Dana per Access and Visibility

**Empty state region (replaces the card list when no deliverables exist):** A centered prompt: "No deliverables yet. Upload the first one for this milestone." with an "Upload Deliverable" action button, identical in destination to the header action.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Cards stack in a single column, full width; the action row wraps to a second line beneath the thumbnail if all three actions do not fit on one line.
- **Medium size class and above:** Cards remain single-column, capped at a consistent platform-wide list width and horizontally centered; the action row stays on one line alongside the thumbnail and metadata.
- **Preview thumbnails:** Uniform scaling, no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Upload Deliverable button (header or empty state) | Tap | Navigate to FEAT-06.SPEC-001 (Deliverable Upload) | Screen closes | Standard navigation transition |
| Preview / Open link | Tap | Opens the deliverable's file preview inline, or opens the linked asset in a new tab | Preview panel expands, or a new browser tab opens for a link | Preview loads with its own loading indicator; for a flagged link, "Open link" is disabled with the tooltip "This link is flagged as unreachable" |
| Replace | Tap | Navigate to FEAT-17 (Deliverable Version History)'s re-upload flow, carrying the deliverable reference | Screen closes | Standard navigation transition to FEAT-17 |
| Remove | Tap | Triggers FEAT-06.SPEC-005's removal-eligibility check | A confirmation dialog appears while the check runs | Dialog: "Remove this deliverable? This can't be undone -- to share a new version instead, use Replace." with "Remove" and "Keep" options |
| Remove (confirm) | Tap "Remove" in the dialog | If eligible: sets the deliverable's status to Removed and writes an activity-trail entry (FEAT-13); if blocked: the removal is refused | Card updates to the Removed status badge, or the dialog is replaced with a blocking message | Eligible: toast "Deliverable removed." Blocked: dialog message "This milestone has been approved. Remove is not available -- use Replace to share a new version instead." with a "Replace" shortcut and a "Close" option |
| Remove (cancel) | Tap "Keep" in the dialog | No action | Dialog closes | Card remains unchanged |

### Accessibility Notes

- **Focus order:** Upload Deliverable action (header) -> each deliverable card in list order (preview/open link -> replace -> remove, within each card) -> empty-state action when the list is empty.
- **Status announcements:** A status badge change (e.g., Uploading -> Active as a background upload completes while this screen is open) is announced to assistive technology as it occurs.
- **Removal feedback:** The "Deliverable removed" toast and the blocked-removal dialog message are both announced on appearance; focus moves into the dialog when it opens and returns to the triggering Remove control when it closes.
- **Keyboard alternatives:** All card actions are reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | Empty-state prompt shown in place of the card list | Milestone has zero deliverables | A deliverable is attached (FEAT-06.SPEC-001) |
| Loaded | Card list shown with one card per deliverable | Deliverables exist for this milestone and load succeeds | Screen is navigated away from |
| Loading | Skeleton placeholder cards in the list region | Screen first opens while deliverable data is being fetched | Data load completes (Loaded) or fails (Error) |
| Error | Error banner "Couldn't load this milestone's deliverables." with a Retry action, in place of the card list | The deliverable list fails to load | User taps Retry and the load succeeds |
| Offline/Degraded | Banner "You're offline -- some information may be out of date." at top; already-loaded cards remain visible and interactive for Preview/Open link (cached) and for read-only viewing; Upload, Replace, and Remove are disabled with the tooltip "Reconnect to make changes" | Connectivity is lost while this screen is open | Connectivity returns -- disabled actions re-enable and the list refreshes |

## Validation Rules

Validation governed by FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules). See that spec for the approved-milestone removal block and role-gated write access. This screen checks removal eligibility at the moment Remove is confirmed, not when the dialog first opens, so a milestone approved while the confirmation dialog is open is caught before the removal commits.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Upload Deliverable tap | FEAT-06.SPEC-001 (Deliverable Upload) | -- |
| Replace tap | FEAT-17 (Deliverable Version History, re-upload flow) | FEAT-17 (Deliverable Version History) |
| Back navigation | FEAT-01.SPEC-005 (Project Detail) or the milestone entry point used | FEAT-01 (Client & Project Management) where applicable |

## Data Model

**Creates:** None.
**Reads:** Deliverable -- kind, file or link, uploaded_at, size, status, link_status, for every deliverable on the current Milestone. Milestone -- name, project reference, status (to determine removal eligibility together with FEAT-06.SPEC-005).
**Updates:** Deliverable.status (Active -> Removed), performed by this screen's Remove action but governed entirely by FEAT-06.SPEC-005's eligibility rule.
**Deletes:** None -- removal is a soft status change, never a hard delete (per the Entity-Lifecycle Coverage Matrix).

## Business Rules

- XBR-11: a deliverable on an Approved milestone cannot be removed; the Remove action is blocked and the user is directed to Replace instead.
- XBR-05: every removal writes an append-only activity-trail entry with actor and timestamp (FEAT-13); this screen never permits a removal that skips the trail entry.
- Removal eligibility and role-gated write access are governed entirely by FEAT-06.SPEC-005 -- this screen never duplicates the eligibility check locally.
- Preview never triggers a file download for Dana's read-only rendering (XBR-29): the preview panel streams inline only, consistent with FEAT-16's delivery capability.

## Edge Cases

- **Milestone approved between the Remove tap and the confirmation** -- The eligibility re-check at confirmation catches the change; the dialog updates to the blocked message without requiring the user to re-open it, per the Contention note for Deliverable and Milestone (reject-with-refresh at the moment of commit).
- **A client contact is previewing this deliverable (via FEAT-07) while Nadia removes it here** -- The client's open view is refreshed to reflect the Removed status on their next interaction; the dependency map's Contention note for Deliverable states a client viewing a replaced deliverable is refreshed to the latest version, and the same refresh-on-next-interaction applies to a removal.
- **User taps Remove twice rapidly** -- The confirmation dialog opens once; a second tap while the dialog is open, or while the removal is committing, is ignored.
- **Upload completes in the background while this screen is open (from a concurrent FEAT-06.SPEC-001 session or tab)** -- The affected card's status badge updates live from Uploading to Active without a manual refresh.
- **User taps Replace on a deliverable that was just removed in another session** -- FEAT-17's re-upload flow re-checks the deliverable's current status on entry and, finding it Removed, shows "This deliverable was removed. Upload a new one instead." routing back to FEAT-06.SPEC-001.
- **Deliverable list exceeds a full screen of cards** -- The list scrolls independently of the header; no pagination is applied, since a milestone's deliverable count in this product's typical use (a handful of rounds per milestone) does not approach a volume requiring it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-001 (Deliverable Upload) | Navigation (inbound/outbound) | Reached via "Upload Deliverable"; returns here after a successful attach |
| FEAT-06.SPEC-003 (Resumable Upload Handling) | References (inbound) | Status badge reflects this automation's live progress and completion |
| FEAT-06.SPEC-004 (Linked Asset Reachability Check) | References (inbound) | link_status badge reflects this automation's outcome |
| FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules) | References (inbound) | Removal eligibility and role-gated write access |
| FEAT-17 (Deliverable Version History) | Navigation (outbound) | "Replace" hands off into the re-upload flow |
| FEAT-01.SPEC-005 (Project Detail) | Navigation (inbound) | Entry point from the project's milestones area |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's support session renders this screen read-only |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| deliverable_removed | milestone reference, deliverable kind | Nadia confirms a removal that is eligible | N/A -- no Stage 2 metric measures removal; retained per product-features.md's Signals field (deliverable_removed) so removal activity remains observable |
| deliverable_removal_blocked | milestone status at attempt | Nadia attempts removal on an Approved milestone and is refused | N/A -- no Stage 2 metric measures blocked removals; this event surfaces how often the approved-milestone guard (XBR-11) is exercised |

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 5 (empty, loaded, loading, error, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Resumable Upload Handling

## Overview

**Name:** Resumable Upload Handling
**ID:** FEAT-06.SPEC-003
**Type:** Automation
**Purpose:** Processes a file upload in a resumable, progress-tracked way, pausing and auto-resuming through dropped connections, and marks the deliverable ready only on full completion.
**Parent Feature:** FEAT-06 -- Deliverable Upload & Sharing

## Scope and Non-Goals

**In Scope:**
- Creating the Deliverable record (Uploading status) when a file transfer begins
- Moving the file into storage through the large-file storage capability, resumably and with real progress
- Pausing and automatically resuming a transfer across a dropped connection, without restarting from zero
- Completing the transfer: setting the Deliverable Active, creating its first Deliverable Version (round 1), updating the owning Milestone's status, and emitting the completion signal FEAT-06.SPEC-006 waits for

**Non-Goals:**
- The actual byte-level storage write and resumable chunk protocol -- owned by FEAT-16.SPEC-002 (Resumable Upload Transfer); this spec is the deliverable-layer automation that invokes it and interprets its outcome
- Enforcing the per-file size ceiling -- owned by FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules), which this automation checks against before transferring bytes
- Creating round 2+ Deliverable Versions -- excluded per the dependency map: re-upload versions are created by FEAT-17 (Deliverable Version History), out of this feature's scope
- Validating or checking pasted external links -- handled entirely by FEAT-06.SPEC-004 (Linked Asset Reachability Check); this automation processes uploaded files only

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| File selected for upload | FEAT-06.SPEC-001 (Deliverable Upload) | Fires immediately when Nadia selects or drops a file, before Attach Deliverable is tapped | File bytes, file name, file size, the milestone reference carried into the upload screen |
| Connectivity restored during an in-progress transfer | FEAT-16.SPEC-002 (Resumable Upload Transfer) | Fires when the storage capability reports connectivity has returned for a transfer this automation started | Transfer identifier, bytes already committed, remaining bytes |
| Retry tapped after a non-connectivity failure | FEAT-06.SPEC-001 (Deliverable Upload) | Fires when Nadia taps Retry on a failed transfer, using the already-selected file | Same file bytes as the original attempt, previous failure reason |

## Processing Logic

1. Receive the selected file (bytes, name, size) and the milestone reference from FEAT-06.SPEC-001.
2. Check the file's size against the per-file ceiling defined by FEAT-06.SPEC-005 (platform parameter: `deliverable-file-size-ceiling`). If it exceeds the ceiling, stop and produce the Failure (size ceiling) outcome without creating a Deliverable record.
3. Create the Deliverable record: kind = uploaded file, milestone = the carried reference, uploaded_at = current time, status = Uploading.
4. Hand the file to the large-file storage capability (FEAT-16.SPEC-002) to begin a resumable, chunked transfer, and report progress back to FEAT-06.SPEC-001 as it advances.
5. If the connection drops mid-transfer, pause the transfer, persist the resume point (via FEAT-16.SPEC-002's own resume-state handling), and surface the Paused/Resuming state to FEAT-06.SPEC-001.
6. When connectivity returns, resume the transfer automatically from the persisted point without restarting from zero.
7. If a transfer chunk fails for a reason other than dropped connectivity, retry that chunk automatically (via FEAT-16.SPEC-002's own retry) before surfacing any failure to the user.
8. When the transfer completes fully: set the Deliverable's status to Active and record its final size; create the first Deliverable Version (round_number = 1, file = the stored file reference, uploaded_at = completion time); update the owning Milestone's status to "Deliverable Uploaded" (dependency map: Milestone updated by FEAT-06); emit `deliverable_uploaded`.
9. If the transfer fails for a reason other than a dropped connection (server rejection, storage capability reports a persistent problem), leave the Deliverable's status at Uploading with the transfer marked failed, and surface the failure to FEAT-06.SPEC-001 for manual retry.
10. Signal FEAT-06.SPEC-006 (Deliverable Ready Notification) only after step 8 completes -- never for a partial file (XBR-12).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Size ceiling exceeded | Selected file's size exceeds platform parameter: `deliverable-file-size-ceiling` | No Deliverable record is created | FEAT-06.SPEC-001 shows the Upload Failed state with the ceiling-specific message from FEAT-06.SPEC-005; no Retry offered (the file itself must change) | FEAT-06.SPEC-001 |
| Transfer in progress | Transfer is actively moving bytes | Deliverable exists with status Uploading | FEAT-06.SPEC-001 shows live progress percentage | FEAT-06.SPEC-001, FEAT-06.SPEC-002 (badge shows Uploading if the list screen is open) |
| Paused, auto-resuming | Connectivity drops mid-transfer | Deliverable remains Uploading; transfer resume state persisted | FEAT-06.SPEC-001 shows "Paused -- resuming when your connection returns" | FEAT-06.SPEC-001 |
| Transfer completes fully | All bytes committed to storage | Deliverable set to Active with final size; Deliverable Version round 1 created; Milestone status set to "Deliverable Uploaded" | FEAT-06.SPEC-001 shows Upload Complete and enables Attach Deliverable; FEAT-06.SPEC-002's card shows Active | FEAT-06.SPEC-001, FEAT-06.SPEC-002, FEAT-06.SPEC-006 (notification fires) |
| Transfer fails (non-connectivity) | Server rejection or persistent storage-capability error, after automatic chunk retries are exhausted | Deliverable remains Uploading, transfer marked failed | FEAT-06.SPEC-001 shows Upload Failed with a Retry control and the selected file preserved | FEAT-06.SPEC-001 |
| Retry succeeds | Nadia retries a failed transfer and it completes | Same as "Transfer completes fully" | Same as "Transfer completes fully" | FEAT-06.SPEC-001, FEAT-06.SPEC-002, FEAT-06.SPEC-006 |
| Automation itself unavailable | The large-file storage capability (FEAT-16.SPEC-007) reports it cannot accept transfers at all | No Deliverable status change beyond Uploading; no bytes transferred | FEAT-06.SPEC-001 shows "Uploads aren't available right now. Try again shortly." with a Retry control | FEAT-06.SPEC-001 |

## Data Model

**Reads:** Milestone -- reference and current status, to attach the new Deliverable.
**Creates:** Deliverable record (kind, milestone, uploaded_at, status) on transfer start. Deliverable Version record (round_number = 1, file, uploaded_at) on transfer completion.
**Updates:** Deliverable.status (Uploading -> Active), Deliverable.size (set on completion). Milestone.status (-> "Deliverable Uploaded") on completion.
**Deletes:** None.

## Business Rules

- XBR-12: clients are notified about a deliverable only once its upload has fully completed, never for a partial file; interrupted uploads resume rather than restart.
- XBR-14: the per-file size ceiling (platform parameter: `deliverable-file-size-ceiling`) and the per-freelancer storage allowance are enforced by FEAT-06.SPEC-005 and FEAT-16.SPEC-004 respectively; this automation checks the size ceiling before transferring bytes and defers to FEAT-16.SPEC-004 for the storage-allowance check during the transfer itself.
- This automation runs asynchronously from FEAT-06.SPEC-001's Attach Deliverable action -- the transfer begins the moment a file is selected, not when Attach Deliverable is tapped, so progress is visible immediately.
- The Deliverable created by this automation is a single feature's write: only Nadia's active session can start a transfer, consistent with the dependency map's Contention note that Nadia is the only writer for Deliverable.

## Edge Cases

- **File has zero bytes** -- Rejected before a transfer starts, surfaced to FEAT-06.SPEC-001 as "This file appears to be empty." with no Deliverable record created.
- **Connection drops and returns within the same second (flapping connectivity)** -- The pause/resume cycle is debounced: a reconnect within a short window resumes without visibly flashing the Paused state to the user.
- **Storage capability reports the freelancer's storage allowance is exhausted mid-transfer** -- The transfer stops; the Deliverable remains Uploading with the failure surfaced as "You've reached your storage allowance." per FEAT-16.SPEC-004's warning, with a link into the plan view (FEAT-23); this is treated as a Failure outcome, not a size-ceiling outcome.
- **User closes the browser tab mid-transfer** -- The transfer is cancelled; no partial Deliverable Version is ever created, and the Deliverable record (if already created) remains Uploading indefinitely until Nadia returns and retries or abandons it from FEAT-06.SPEC-002.
- **Concurrent trigger firing (two files selected for the same milestone in two browser tabs of the same session)** -- Each selection starts its own independent transfer and creates its own Deliverable record; whichever completes first becomes the milestone's round-1 deliverable, and the second's completion is rejected-with-refresh per FEAT-06.SPEC-001's concurrent-attach edge case, directing the second transfer's deliverable toward the Replace flow instead.
- **Trigger fires while a previous run is in flight (Retry tapped while the original attempt is still retrying a chunk automatically)** -- The manual Retry request is ignored while an automatic chunk retry is already in progress for the same transfer; the existing transfer continues rather than starting a duplicate.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-001 (Deliverable Upload) | Triggered by (inbound) | File selection and Retry both start or resume this automation |
| FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules) | References (inbound) | Size ceiling rule and its error message |
| FEAT-16.SPEC-002 (Resumable Upload Transfer) | Triggers (outbound) | Performs the actual byte-level resumable transfer and reports progress, pause, and completion |
| FEAT-06.SPEC-006 (Deliverable Ready Notification) | Affects (outbound) | Transfer completion is the trigger this notification waits for |
| FEAT-06.SPEC-002 (Deliverable List & Management) | Affects (outbound) | Displays this automation's live status via the deliverable's status badge |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Upload completion writes an append-only trail entry (XBR-05) |

## Analytics and Success Signals

- **deliverable_uploaded** (file size, milestone reference, time from selection to completion) -- supports success-metrics.md: "Deliverable Upload Reliability"
- **upload_resumed** (pause duration, bytes remaining at pause) -- supports success-metrics.md: "Deliverable Upload Reliability"
- **upload_failed** (failure reason: size_ceiling / storage_allowance / server_rejection / capability_unavailable, file size) -- supports success-metrics.md: "Deliverable Upload Reliability"

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (file selected, connectivity restored, manual retry) | 3 |
| Outcome Paths | 7 | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Linked Asset Reachability Check

## Overview

**Name:** Linked Asset Reachability Check
**ID:** FEAT-06.SPEC-004
**Type:** Automation
**Purpose:** Validates that a pasted external link resolves before the deliverable is marked ready, flagging an unreachable link to the freelancer.
**Parent Feature:** FEAT-06 -- Deliverable Upload & Sharing

## Scope and Non-Goals

**In Scope:**
- Checking that a pasted Figma, Google Drive, or Dropbox link resolves and is reachable
- Setting the Deliverable's link_status to reachable or flagged based on the outcome
- Flagging an unreachable link to Nadia before the deliverable is ever shown to the client as broken

**Non-Goals:**
- Validating the link's format (well-formed URL, recognized domain) -- owned by FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules), which runs before this automation is invoked
- Copying, mirroring, or previewing the linked asset's content -- excluded per scope-boundaries.md (SC-08): linked assets are referenced by URL only, never copied into the product
- Re-checking a link's reachability on an ongoing basis after the deliverable is marked ready -- this automation runs once per submitted link at attach time; no periodic re-verification exists in the product definition
- Processing uploaded files -- handled entirely by FEAT-06.SPEC-003 (Resumable Upload Handling); this automation processes pasted links only

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Link submitted for attach | FEAT-06.SPEC-001 (Deliverable Upload) | Fires when Nadia taps Attach Deliverable with "Paste a link" selected and the link has already passed FEAT-06.SPEC-005's format check | The submitted URL, the milestone reference |
| Link re-submitted after being flagged | FEAT-06.SPEC-001 (Deliverable Upload) | Fires when Nadia edits a flagged link and re-submits | The revised URL, the milestone reference, the prior flagged attempt |

## Processing Logic

1. Receive the format-valid URL and milestone reference from FEAT-06.SPEC-001.
2. Create the Deliverable record if one does not already exist for this submission: kind = linked external asset, milestone = the carried reference, uploaded_at = current time, link_status = pending.
3. Attempt to resolve the link: request the target address and evaluate whether it responds successfully and is accessible (not requiring credentials the product does not hold, not returning a not-found or access-denied response).
4. If the link resolves successfully, set link_status to reachable, the Deliverable's status to Active, and the owning Milestone's status to "Deliverable Uploaded" (dependency map: Milestone updated by FEAT-06), mirroring FEAT-06.SPEC-003's completion step -- a link-delivered deliverable reaches this milestone state exactly as a file-delivered one does.
5. If the link does not resolve, set link_status to flagged and leave the Deliverable's status at its pending state -- the deliverable is never shown to the client while flagged.
6. Signal FEAT-06.SPEC-006 (Deliverable Ready Notification) only when link_status becomes reachable -- never while flagged (XBR-12's "never for a partial file" applies equally to an unresolved link, since it is not yet a usable deliverable).
7. Return the outcome (reachable or flagged) to FEAT-06.SPEC-001 for immediate display.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Link reachable | The target resolves successfully and is accessible | Deliverable.link_status = reachable; Deliverable.status = Active; Milestone.status = "Deliverable Uploaded" | FEAT-06.SPEC-001 shows success and returns Nadia to FEAT-06.SPEC-002; FEAT-06.SPEC-002's card shows reachable | FEAT-06.SPEC-001, FEAT-06.SPEC-002, FEAT-06.SPEC-006 (notification fires) |
| Link flagged | The target fails to resolve, or resolution reports access is denied | Deliverable.link_status = flagged; Deliverable.status remains pending | FEAT-06.SPEC-001 shows the inline message "This link couldn't be reached. Check that it's shared and try again."; the deliverable is not shown to the client | FEAT-06.SPEC-001 |
| Reachability service unavailable | The check itself cannot be performed (the capability behind the check is unreachable, not the target link) | Deliverable.link_status remains pending; no Active transition | FEAT-06.SPEC-001 shows "Couldn't verify this link right now. Try again in a moment." with a Retry option that re-runs the check on the same URL | FEAT-06.SPEC-001 |

## Data Model

**Reads:** Milestone -- reference and current status, to attach the Deliverable.
**Creates:** Deliverable record (kind, milestone, uploaded_at, link_status) when a link is first submitted.
**Updates:** Deliverable.link_status (pending -> reachable or flagged), Deliverable.status (-> Active only when reachable). Milestone.status (-> "Deliverable Uploaded") only when reachable.
**Deletes:** None.

## Business Rules

- XBR-12: the client is never notified, and the deliverable is never shown to the client, while link_status is flagged or pending -- only a reachable link produces the deliverable-ready signal.
- The reachability check runs once per submission; a flagged link requires Nadia to edit and re-submit before another check runs (no automatic periodic re-check).
- This automation defers all link-format validation to FEAT-06.SPEC-005 -- it only ever receives links that have already passed the format check, so it never itself judges whether a URL is well-formed or from a recognized domain.

## Edge Cases

- **Link target requires sign-in the product cannot provide** -- Treated as flagged: the check cannot confirm reachability without credentials it does not hold, so it reports the same "couldn't be reached" outcome rather than a distinct "requires sign-in" message, since the freelancer's remedy (share the link publicly or with link-access) is the same either way.
- **Link resolves slowly (large file behind the link, or a slow third-party service)** -- The check waits for a definitive response rather than timing out prematurely; if no response arrives within a bounded wait, the outcome is "Reachability service unavailable" with a Retry option, not "flagged" -- a slow response is not evidence the link is broken.
- **Nadia edits a flagged link to point at a completely different asset** -- Treated as a fresh submission: the check re-runs against the new URL with no memory of the prior flagged attempt influencing the new result.
- **Concurrent trigger firing (two link submissions for the same milestone with no existing deliverable, from two sessions)** -- Each check runs independently against its own pending Deliverable record; whichever reaches reachable first becomes the milestone's round-1 deliverable, and the second is rejected-with-refresh per FEAT-06.SPEC-001's concurrent-attach edge case.
- **Trigger fires while a previous run is in flight (Nadia taps Attach Deliverable twice on the same link before the first check returns)** -- The second tap is ignored while the first check is in progress, per FEAT-06.SPEC-001's debounced-button interaction; only one check runs per submission.
- **Linked asset is later moved or its sharing permissions are revoked, after the deliverable was already marked reachable** -- Out of scope for this automation, which checks reachability once at attach time only; a subsequently broken link surfaces to the client as an unreachable open attempt, which is a client-facing behavior owned by FEAT-07, not re-checked here.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-001 (Deliverable Upload) | Triggered by (inbound) | Link submission and re-submission both start this automation |
| FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules) | References (inbound) | Link format validation runs before this automation is invoked |
| FEAT-06.SPEC-006 (Deliverable Ready Notification) | Affects (outbound) | A reachable outcome is the trigger this notification waits for |
| FEAT-06.SPEC-002 (Deliverable List & Management) | Affects (outbound) | Displays this automation's link_status badge |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | A successful link attach writes an append-only trail entry (XBR-05) |

## Analytics and Success Signals

- **deliverable_linked** (link domain: figma / drive / dropbox, milestone reference) -- N/A -- no Stage 2 metric measures linked-asset attachment specifically; Deliverable Upload Reliability is defined in terms of file transfers, so this event is retained for feature-level visibility only.
- **upload_failed** (failure reason: link_unreachable, link domain) -- N/A -- FEAT-06.SPEC-001's Analytics section establishes that "Deliverable Upload Reliability" is defined in terms of file transfers, so this link-path event, like SPEC-001's `deliverable_linked`, is N/A for that metric; retained for feature-level visibility only.

## Acceptance Criteria

**FEAT-06.SPEC-004-AC-01:** Given Nadia submits a valid, publicly shared Figma link, when the reachability check runs, then link_status is set to reachable, the Deliverable becomes Active, the owning Milestone's status becomes "Deliverable Uploaded", and FEAT-06.SPEC-006 is triggered.

**FEAT-06.SPEC-004-AC-02:** Given Nadia submits a Google Drive link that fails to resolve, when the reachability check runs, then link_status is set to flagged, the Deliverable's status remains pending, and FEAT-06.SPEC-001 shows "This link couldn't be reached. Check that it's shared and try again."

**FEAT-06.SPEC-004-AC-03:** Given a deliverable's link is flagged, when Nadia checks the client-facing view before fixing it, then the deliverable is never shown to the client as broken -- it simply does not appear as ready.

**FEAT-06.SPEC-004-AC-04:** Given Nadia edits a flagged link and re-submits it, when the check re-runs, then it evaluates only the new URL, independent of the prior flagged result.

**FEAT-06.SPEC-004-AC-05:** Given the reachability check itself cannot run because the underlying service is unavailable, when this occurs, then Nadia sees "Couldn't verify this link right now. Try again in a moment." with a Retry option, distinct from a flagged link.

**FEAT-06.SPEC-004-AC-06:** Given a linked asset requires sign-in the product cannot provide, when the check runs, then the outcome is flagged with the same "couldn't be reached" message as any other unreachable link.

**FEAT-06.SPEC-004-AC-07:** Given a linked asset resolves slowly but eventually responds within the bounded wait, when the response arrives, then the check completes normally rather than timing out prematurely.

**FEAT-06.SPEC-004-AC-08:** Given two of Nadia's sessions submit different links for the same milestone with no existing deliverable, when both checks return reachable, then the first to complete becomes the round-1 Active deliverable and the second is rejected-with-refresh toward the Replace flow.

**FEAT-06.SPEC-004-AC-09:** Given Nadia taps Attach Deliverable twice rapidly on the same link, when the second tap occurs, then it is ignored while the first check is in progress and only one check runs.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (initial submission, re-submission after flag) | 2 |
| Outcome Paths | 3 | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Deliverable Validation & Removal Eligibility Rules

## Overview

**Name:** Deliverable Validation & Removal Eligibility Rules
**ID:** FEAT-06.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs upload validity (milestone required, valid/reachable link, size ceiling), who may upload or remove a deliverable, and the approved-milestone removal block.
**Parent Feature:** FEAT-06 -- Deliverable Upload & Sharing
**Governed Entity:** Deliverable

## Scope and Non-Goals

**In Scope:**
- Field-level validation rules for the Deliverable record: milestone requirement, link format, and the file size ceiling
- Authorization rules for every action on the Deliverable record, per role
- The approved-milestone removal block (XBR-11) and its exact denied experience
- Default values and derived fields on the Deliverable record

**Non-Goals:**
- Checking whether a submitted link actually resolves -- owned by FEAT-06.SPEC-004 (Linked Asset Reachability Check), which this spec's format rule feeds into but does not perform itself
- Performing the resumable file transfer -- owned by FEAT-06.SPEC-003 (Resumable Upload Handling); this spec defines only the size-ceiling threshold it checks against
- Enforcing the per-freelancer storage allowance -- owned by FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules); this spec's size ceiling is the per-file limit only, a distinct rule from the account-wide allowance
- Rules governing Deliverable Version (round 2+) creation, editing, or deletion -- excluded per the dependency map: Deliverable Version is immutable once uploaded and its later rounds belong to FEAT-17 (Deliverable Version History)

## Governed Entity

**Entity:** Deliverable
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| kind | enum | Uploaded file or linked external asset (design-tool, cloud-drive, or file-sharing link) |
| file or link | file reference \| text (URL) | The uploaded file, or a valid, reachable link (not copied in) |
| milestone | reference | Owning Milestone (required) |
| uploaded_at | date/time | Timestamp of creation (required) |
| size | number | File size in bytes, for uploaded files only |
| status | enum | Uploading, Active, Superseded, Removed |
| link_status | enum | reachable or flagged (linked assets only) |
| first_client_view_at | date/time (derived) | Timestamp of a client contact's first view, written by FEAT-13, not by this feature |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-001 | Deliverable Upload | Link format on field blur; milestone-required is structural (always pre-supplied); authorization on screen entry |
| FEAT-06.SPEC-002 | Deliverable List & Management | Removal eligibility check at Remove confirmation; authorization on screen entry and on each action |
| FEAT-06.SPEC-003 | Resumable Upload Handling | File size ceiling check before a transfer begins |
| FEAT-06.SPEC-004 | Linked Asset Reachability Check | Consumes the link-format outcome as a precondition before running its own reachability check |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| kind | Required; must be exactly one of uploaded file or linked external asset, set once at creation and never changed afterward | Always | On creation | "Choose a file to upload or a link to paste." | Yes |
| file or link | Exactly one of a file or a link must be present, never both, never neither | Always | On Attach Deliverable | "Choose a file to upload or a link to paste -- not both." | Yes |
| file or link (file path) | File size must not exceed platform parameter: `deliverable-file-size-ceiling` | When kind is uploaded file | Before the transfer begins (FEAT-06.SPEC-003) | "This file is larger than the size limit for deliverables. Compress it or share it by link instead." | Yes |
| file or link (link path) | Must be a well-formed URL from a recognized design-tool, cloud-drive, or file-sharing domain (Figma, Google Drive, Dropbox) | When kind is linked external asset | On blur | "Enter a Figma, Google Drive, or Dropbox link." | Yes |
| file or link (link path) | Must resolve and be reachable (delegated entirely to FEAT-06.SPEC-004) | When kind is linked external asset and the format rule above passes | On Attach Deliverable, before the deliverable is marked ready | "This link couldn't be reached. Check that it's shared and try again." | Yes -- blocks the deliverable from becoming Active, not the submission itself |
| milestone | Required; must reference an existing Milestone on a Project the requesting freelancer owns | Always | Structural -- always pre-supplied by the entry point (FEAT-06.SPEC-001); never user-editable on this screen | "A milestone is required to attach a deliverable." (defensive message; not reachable through the normal flow) | Yes |
| uploaded_at | System-set on creation; no user input | Always | -- | -- | No validation beyond data type |
| size | System-captured from the completed transfer, for uploaded files only; not applicable to linked assets | When kind is uploaded file | -- | -- | No validation beyond data type (the size ceiling above is the file-selection rule, not a field format rule) |
| status | System-managed enum; never directly editable by any role -- transitions are driven by FEAT-06.SPEC-003, FEAT-06.SPEC-004, FEAT-06.SPEC-002 (removal), and FEAT-17 (supersession) | Always | -- | -- | No validation beyond data type -- transition legality is governed by the Business Rules and Authorization Rules below, not by field-level format checks |
| link_status | System-managed enum, linked assets only; set exclusively by FEAT-06.SPEC-004 | When kind is linked external asset | -- | -- | No validation beyond data type |
| first_client_view_at | Written by FEAT-13 on a client contact's first view; this feature never reads or writes this field | Always | -- | -- | No validation beyond data type -- out of this feature's write scope entirely |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Mutually exclusive content | kind, file or link | A Deliverable of kind "uploaded file" carries file bytes and no link value; a Deliverable of kind "linked external asset" carries a link value and no file bytes -- the two never coexist on one record | "Choose a file to upload or a link to paste -- not both." |
| Ready-state consistency | status, link_status | status can be Active only when: (a) kind is uploaded file and the transfer has fully completed, or (b) kind is linked external asset and link_status is reachable | Not user-facing -- this is an internal consistency rule enforced by FEAT-06.SPEC-003 and FEAT-06.SPEC-004, never violated through any user-facing path |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Upload a file or attach a linked asset (create) | Nadia (Freelancer) | Always, on a milestone within her own account | -- |
| Upload a file or attach a linked asset (create) | Owen (Client Primary Contact) | Never | The Deliverable Upload screen (FEAT-06.SPEC-001) is never part of the client portal's navigation; no control exists for Owen to attempt this action |
| Upload a file or attach a linked asset (create) | Priya (Client Reviewer Contact) | Never | Same as Owen -- no control exists in the client portal |
| Upload a file or attach a linked asset (create) | Dana (Support Operator) | Never | Dana's read-only support session never renders an upload action (XBR-29) |
| View deliverable (Nadia's management screen, FEAT-06.SPEC-002) | Nadia (Freelancer) | Always, on her own account's milestones | -- |
| View deliverable (client-facing, via FEAT-07) | Owen (Client Primary Contact) | Always, own company's projects only | -- |
| View deliverable (client-facing, via FEAT-07) | Priya (Client Reviewer Contact) | Always, own company's projects only | -- |
| View deliverable (Nadia's management screen, read-only rendering) | Dana (Support Operator) | Always, inside a logged support session on the account being supported; list and status only, no file download | Attempting a download is not possible -- no download control is rendered for Dana in a support session (XBR-29) |
| Preview / stream a deliverable's file | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Always, subject to the same view scoping above | -- |
| Preview / stream a deliverable's file | Dana (Support Operator) | Inline preview only, never a download, inside a logged support session | A download attempt shows "Support sessions are read-only and don't include file downloads." |
| Remove a deliverable | Nadia (Freelancer) | Only when the owning Milestone's status is not Approved (XBR-11) | Blocking dialog: "This milestone has been approved. Remove is not available -- use Replace to share a new version instead." |
| Remove a deliverable | Owen (Client Primary Contact) | Never | No Remove control exists on the client-facing view; a contact never removes a deliverable |
| Remove a deliverable | Priya (Client Reviewer Contact) | Never | No Remove control exists on the client-facing view |
| Remove a deliverable | Dana (Support Operator) | Never | No Remove control is rendered in a support session (XBR-29) |
| Replace a deliverable (initiate hand-off into FEAT-17) | Nadia (Freelancer) | Always, on her own account's deliverables, regardless of the owning Milestone's approval status | -- |
| Replace a deliverable (initiate hand-off into FEAT-17) | Owen, Priya, Dana | Never | No Replace control exists on the client-facing view or the read-only support session |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status | Uploading (uploaded file) or pending (linked external asset), transitioning to Active only on completion/reachability | On create, then automatically on completion | No -- always system-derived, never directly set by any role |
| link_status | pending, transitioning to reachable or flagged | On create (linked assets only), then set by FEAT-06.SPEC-004 | No |
| uploaded_at | Current date and time | On create only | No |
| size | The completed transfer's final byte count | On upload completion (uploaded files only) | No |

## Business Rules

- XBR-11: a deliverable on an Approved milestone cannot be removed, only superseded by a new version; every removal is recorded in the activity trail regardless of outcome (attempted-and-blocked removals are not themselves trail-logged, since no state change occurred -- only a successful removal writes a trail entry, per XBR-05's "record-worthy event" scope).
- XBR-05: every successful removal writes an append-only trail entry with actor and timestamp (FEAT-13).
- XBR-12: a Deliverable's status becomes Active, and the client is notified, only once its content is fully ready (upload complete or link reachable) -- this spec's field rules exist specifically to prevent a partial or broken deliverable from reaching Active status.
- XBR-14: the size ceiling in this spec (platform parameter: `deliverable-file-size-ceiling`) is a distinct, per-file rule from FEAT-16.SPEC-004's per-freelancer storage allowance; a file can pass this spec's ceiling check and still be rejected by the storage allowance during transfer.
- Copying or hosting linked content is out of scope (SC-08): this spec's link rules validate format and delegate reachability, but at no point do they cause the linked content itself to be copied into the product.

## Edge Cases

- **File size exactly at the ceiling** -- Passes validation; one byte over fails with the size-ceiling message.
- **Link with trailing whitespace or a tracking query parameter appended** -- The format check normalizes whitespace before evaluating the domain; a recognized domain with extra query parameters still passes format validation, since the reachability check (FEAT-06.SPEC-004) is the authoritative test of whether the link actually works.
- **Milestone deleted between screen load and Attach Deliverable submission** -- Structurally prevented in the normal flow (FEAT-06.SPEC-001's Error state catches an unresolvable milestone reference before the form is usable); if the milestone is removed in the narrow window after the form loads successfully, the create is rejected with "This milestone is no longer available." and no Deliverable record is created.
- **Milestone approved between Remove being tapped and the confirmation being submitted** -- The eligibility check re-runs at the moment of commit, not at the moment Remove was first tapped, per the dependency map's Contention note for Deliverable (reject-with-refresh); the confirmation dialog updates to the blocked message.
- **Milestone reopened (Reopened status) after having been Approved, with the deliverable never removed in between** -- Removal becomes allowed again once the Milestone leaves Approved status, since the block is keyed to Approved specifically, not to having ever been approved.
- **A file passes the size ceiling but the storage allowance is exhausted mid-transfer** -- This spec's ceiling check already passed; the subsequent allowance failure is FEAT-16.SPEC-004's rule, surfaced as a distinct failure by FEAT-06.SPEC-003, not re-evaluated by this spec.
- **Kind field somehow carries both a file and a link (defensive case)** -- Rejected at creation with the mutually-exclusive-content error; this state is not reachable through FEAT-06.SPEC-001's normal toggle-based UI, so this rule exists as a data-integrity backstop.

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 10 | 10 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 15 | 15 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Notification Spec: Deliverable Ready Notification

## Overview

**Name:** Deliverable Ready Notification
**ID:** FEAT-06.SPEC-006
**Type:** Notification
**Purpose:** Emails the relevant client contacts once an uploaded or linked deliverable is fully ready to review, so they never have to be told by Nadia directly that something is waiting on them.
**Parent Feature:** FEAT-06 -- Deliverable Upload & Sharing

## Scope and Non-Goals

**In Scope:**
- The email sent to relevant client contacts when a deliverable becomes fully ready (upload complete, or link confirmed reachable)
- Preference, retry, and expiry behavior for this notification
- The single-deliverable content; this notification is never batched, per its Delivery Rules below

**Non-Goals:**
- Deciding whether an upload or link check has fully completed -- owned by FEAT-06.SPEC-003 (Resumable Upload Handling) and FEAT-06.SPEC-004 (Linked Asset Reachability Check); this spec begins where their completion trigger fires
- The transactional email delivery mechanism itself (sending, bounce and delivery status reporting) -- owned by FEAT-14.SPEC-001 (Transactional Email Delivery), the capability this notification is composed for and dispatched through
- Notifying Nadia that her own upload succeeded -- that is an in-screen confirmation on FEAT-06.SPEC-001 ("Deliverable attached" toast), not a delivery-channel notification to the freelancer
- Comment or approval notifications once a deliverable is being reviewed -- owned by FEAT-07 (Deliverable Review & Feedback) and FEAT-08 (Milestone Approval), which govern the next steps in a client contact's journey after this notification

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, when a deliverable becomes fully ready | BRIEF.md's Ecosystem & Integrations states clients will not install an app, making email the sole channel that reaches Owen and Priya outside a session; both personas' Behavioral Context describes opening the portal from an email link, often on a phone |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| File upload completes fully | FEAT-06.SPEC-003 (Resumable Upload Handling) | Fires only when the Deliverable's status transitions to Active from a completed transfer (never for a partial file, XBR-12) | Deliverable reference, milestone name, project name, freelancer's Branding Profile |
| Linked asset confirmed reachable | FEAT-06.SPEC-004 (Linked Asset Reachability Check) | Fires only when link_status transitions to reachable | Deliverable reference, milestone name, project name, freelancer's Branding Profile |

## Audience and Preferences

**Recipients:** Owen (Client Primary Contact) and Priya (Client Reviewer Contact) -- every Active contact on the Client that owns the Milestone's Project, per the Access Matrix in user-persona.md (both roles have Own-only visibility into Milestones & Deliverables and both are entitled to review a deliverable, per XBR-08's "notification recipients are limited to contacts entitled to the event"). Nadia does not receive this notification -- her own upload-success feedback is the in-screen confirmation on FEAT-06.SPEC-001.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| This notification is a transactional record email | Cannot be switched off | Always on | N/A -- XBR-30: transactional emails core to the record always send; a new deliverable ready to review is exactly this kind of record-carrying event, not an optional update |

**Quiet Hours:** N/A -- the product definition establishes no quiet-hours window for client-facing transactional notifications; XBR-30 treats this class of email as always-sending, and no Stage 2 document defines a quiet-hours preference surface for client contacts.

## Content Definition

**Email:**
- **Subject:** {freelancer_business_name}: a new deliverable is ready to review
- **Body:**
  Hi {recipient_first_name},

  {freelancer_business_name} has shared a new deliverable on {project_name}, for the milestone "{milestone_name}": {deliverable_label}.

  Open it to review and leave feedback whenever you're ready.
- **CTA (button):** Review deliverable -- deep-links through FEAT-05 (Client Portal Access, magic-link sign-in) to FEAT-07.SPEC-001 (Deliverable Comment Thread), the deliverable view, for the specific Deliverable referenced by this notification

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_business_name} | Freelancer Account -- business_name | Nadia Chen Design | Never empty -- business_name is required before the first invoice, but for this earlier-lifecycle event it falls back to the Freelancer Account's name field, which is required from sign-up |
| {recipient_first_name} | Client Contact -- name (first token) | Owen | Never empty -- name is required at contact creation (FEAT-18) |
| {project_name} | Project -- project_name | Brand Refresh Q1 | Never empty -- project_name is required at project creation (FEAT-01) |
| {milestone_name} | Milestone -- name | Concept Round 1 | Never empty -- name is required at milestone creation (FEAT-04) |
| {deliverable_label} | Derived -- "a file" for kind = uploaded file, or "a linked file" for kind = linked external asset | a file / a linked file | Never empty -- kind is required and always one of the two values at the point this notification fires |

## Delivery Rules

**Batching:** None -- each deliverable that becomes ready sends its own individual email. Two deliverables becoming ready in close succession (e.g., two milestones on the same project) each produce a separate notification, because each names a specific milestone and deliverable a contact needs to act on individually; collapsing them would obscure which milestone is actually waiting.
**Deduplication:** At most one "ready" notification per Deliverable. A Deliverable that becomes Active exactly once (this feature never re-fires the trigger for an already-Active deliverable); a later replacement (FEAT-17, round 2+) produces its own, separate ready notification owned by FEAT-17, not a re-send of this one.
**Retry on failure:** Delivery failure is retried per FEAT-14.SPEC-001's transactional email capability, up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`. After the final failure, the delivery is marked Failed and surfaced to Nadia as a delivery warning on the affected project (XBR-30) -- the deliverable itself remains visible and reachable in-product regardless of email delivery outcome.
**Expiry:** This notification does not expire in the sense of becoming stale content -- a deliverable that was ready yesterday is still exactly as ready today, so a delayed or retried delivery still carries fully accurate content. If all retries are exhausted, delivery simply stops (see Retry on failure); there is no "too late to send" cutoff distinct from the retry exhaustion itself.

## Edge Cases

- **Deliverable removed between trigger and delivery** -- The notification's CTA still deep-links to the deliverable view; if the contact opens it after removal, FEAT-07's own removed-deliverable handling applies (out of this spec's scope). The email itself is not cancelled, since XBR-11/XBR-05 treat the removal as a subsequent, separately logged event, not a reason to suppress a record that a deliverable was, in fact, shared and ready at that moment.
- **All of a client's contacts are Removed (status = Removed) between trigger and delivery** -- If zero Active contacts remain entitled to this notification at delivery time, delivery is skipped for that client entirely (there is no recipient to send to); this is a silent no-send, not a failure, since XBR-27 treats contact removal as ending access, not as something a deliverable notification should work around.
- **A contact's preference or channel cannot be turned off (this notification has none)** -- Not applicable: Preference Controls above establish this notification always sends: there is no mid-flight preference change to collide with.
- **Quiet hours colliding with expiry** -- Not applicable: this notification defines no quiet-hours window (see Audience and Preferences), so no collision exists.
- **Two deliverables on the same milestone become ready in the same minute (a file upload and, separately, a re-check confirming a previously flagged link, both resolving together)** -- Each produces its own individual email per the no-batching rule; a recipient may receive two emails in quick succession, which is intentional so each remains individually actionable and traceable.
- **The freelancer's Branding Profile changes between trigger and delivery** -- The email renders with the Branding Profile as it stands at delivery time, not at trigger time, consistent with branding being a presentation concern applied at render (XBR-31), not a fact captured at the moment the deliverable became ready.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-003 (Resumable Upload Handling) | Triggered by (inbound) | Full upload completion fires this notification |
| FEAT-06.SPEC-004 (Linked Asset Reachability Check) | Triggered by (inbound) | A reachable link outcome fires this notification |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | This notification is composed and handed to the shared transactional email capability for sending and delivery/bounce status reporting |
| FEAT-05 (Client Portal Access) | Navigation (outbound) | The CTA deep-links through magic-link sign-in |
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | Navigation (outbound) | The CTA's ultimate destination is the client-facing deliverable view |
| FEAT-13 (Immutable Activity & Audit Trail) | References (outbound) | This notification's send is itself a record-worthy event captured in the activity trail alongside the upload/link event that triggered it |

## Analytics and Success Signals

- **deliverable_ready_notification_sent** (channel: email; deliverable kind: file / link; recipient role: primary / reviewer) -- N/A -- this feature's success-metrics slice contains only "Deliverable Upload Reliability", which measures whether the upload/link transfer itself completes, not whether the resulting notification is sent; retained for feature-level visibility (this event is the deliverable-ready signal reaching its audience, the outcome "Deliverable Upload Reliability" exists to make possible)
- **deliverable_ready_notification_delivery_failed** (retry count exhausted) -- N/A -- same reason: no metric in this feature's slice measures notification delivery outcomes (delivery reliability is Notifications (Email)'s own connected metric, not this feature's); retained so a silent delivery failure remains observable
- **deliverable_ready_notification_opened** (recipient role) -- N/A -- no metric in this feature's slice measures open rate; retained for feature-level visibility only

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 2 (upload completion, link reachable) | 2 |
| Preference States | 1 (always-on, no preference surface) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |
