---
document_type: spec
spec_type: screen
spec_id: FEAT-06.SPEC-002
spec_name: Deliverable List & Management
spec_slug: deliverable-list-management
parent_feature: FEAT-06
parent_feature_name: Deliverable Upload & Sharing
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

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
