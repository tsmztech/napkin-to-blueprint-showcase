# FEAT-17 — Deliverable Version History

This chapter covers Deliverable Version History, a Important-tier feature. It contains the feature breakdown brief followed by every specification in full: 5 specifications carrying 83 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-17.SPEC-001 | New Version Upload | screen | 21 |
| FEAT-17.SPEC-002 | Version Browser & Comparison | screen | 17 |
| FEAT-17.SPEC-003 | Version Creation & Preservation | automation | 16 |
| FEAT-17.SPEC-004 | Version Numbering, Immutability & Retention Rules | logic-rule | 14 |
| FEAT-17.SPEC-005 | Version Access & Comment-Anchoring Rules | logic-rule | 15 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Deliverable Version History

## Summary

**Feature:** Deliverable Version History
**ID:** FEAT-17
**Description:** Every re-upload to a deliverable keeps the prior version rather than overwriting it, so both freelancer and client can see and open any earlier round rather than losing track in a folder of "final_v3_REAL" files.
**Priority:** Important
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Problem Statement names this exact failure mode by name ("final_v3_REAL" versions), and its Scale section lists "version history per deliverable" as a day-one norm. Ranked Important rather than Core because a single-round deliverable still works without it. [MODIFIED: phase moved from v1 to MVP based on BRIEF.md's Scale section naming version history per deliverable as the norm, the draft scope boundaries (Scale Expectations) already committing to it from MVP, and research finding versioned deliverable handling absent from all 5 profiled competitors (Feature Landscape, Absent Features) — the product's most direct fix for the brief's named problem should not wait for a later release] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Preserve every version — a re-upload never overwrites a prior round
- Browse by version — open and compare any round by number
- Version-anchored comments — feedback stays attached to the version it was made on

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-17.SPEC-001 | New Version Upload | Screen | Nadia | Nadia re-uploads a revised file to an existing deliverable, adding a new preserved round rather than replacing the current one |
| FEAT-17.SPEC-002 | Version Browser & Comparison | Screen | Nadia, Owen, Priya, Dana | Any viewer opens the version selector on a deliverable, browses every round by number, and opens any earlier round alongside the latest |
| FEAT-17.SPEC-003 | Version Creation & Preservation | Automation | Nadia | On a fully completed re-upload, creates the next immutable Deliverable Version, leaves every prior version untouched, and updates which round is "latest" |
| FEAT-17.SPEC-004 | Version Numbering, Immutability & Retention Rules | Logic/Rule | Nadia | Governs sequential round numbering, immutability once uploaded, the uncapped-but-storage-bounded version count, and the "latest version" derivation |
| FEAT-17.SPEC-005 | Version Access & Comment-Anchoring Rules | Logic/Rule | Nadia, Owen, Priya, Dana | Governs who may view or add versions per the Access Matrix, and the rule that a comment stays attached to the version it was posted on even after newer rounds exist |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Preserve every version | FEAT-17.SPEC-001, FEAT-17.SPEC-003, FEAT-17.SPEC-004 | The upload screen collects the re-upload; the creation automation commits it as a new immutable round without touching prior rounds; the numbering/immutability rule spec is the standing contract that makes "never overwrites" true on every future round | Phase 2 (Explicit) |
| Browse by version | FEAT-17.SPEC-002 | The version browser lists every round by number and opens any one, hidden entirely when only one round exists | Phase 2 (Explicit) |
| Version-anchored comments | FEAT-17.SPEC-005 | The access/anchoring rule spec is the authority FEAT-07's comment screens reference so a comment stays tied to the round it was posted on regardless of later uploads | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-17.SPEC-003 | Version Creation & Preservation | Phase 4 (Trigger-Response Analysis) | The feature's States field ("a failed version upload does not overwrite the existing version") and Primary Flows & Alternates describe processing logic with a real failure mode -- version creation is gated on full upload completion, not a direct field write -- crossing the standalone-Automation threshold |
| FEAT-17.SPEC-004 | Version Numbering, Immutability & Retention Rules | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field ("no fixed cap on version count... within the plan's storage allowance," "each version is immutable once uploaded") and the Data Notes field's derivation rule ("latest version" is the most recent upload) are conditional, cross-screen rules that both SPEC-001 and SPEC-003 must obey consistently, crossing the shared-rule threshold |
| FEAT-17.SPEC-005 | Version Access & Comment-Anchoring Rules | Phase 5 (Rule-Constraint Discovery) | The Access field carries four distinct role permissions (Nadia Full, Owen/Priya Own-only view, Dana View-only inside a logged support session) and the Key Capabilities' comment-anchoring rule is authorization- and state-dependent logic shared across this feature's own screens and FEAT-07's comment screens -- not a simple inline validation |

## Entity-Lifecycle Coverage Matrix

**Entity: Deliverable Version**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-17.SPEC-003 | On a fully completed re-upload, creates a new Deliverable Version at the next sequential round_number, carrying the uploaded file | The first version (round_number 1) is created by FEAT-06.SPEC-003 on the deliverable's initial upload -- out of this feature's scope; this feature owns round 2 onward |
| Read (single) | FEAT-17.SPEC-002 | Version browser opens one round's file and metadata (round_number, uploaded_at) | Also read by FEAT-07 (version-anchored comment screens) and FEAT-31 (Dana's support session), outside this feature's own screens |
| Read (list) | FEAT-17.SPEC-002 | Version browser lists every round for a deliverable, ordered by round_number, hidden when only one round exists (Empty state) | -- |
| Update | N/A | The dependency map states Deliverable Version is "never updated (immutable once uploaded)" -- an explicit product decision (Validation & Limits: "each version is immutable once uploaded"), not a gap | Recorded in Non-Goals |
| Delete/Archive | N/A | "Never deleted in-product; removed only by FEAT-24" (dependency map). No soft-delete state exists for a version: hard deletion happens only at account deletion, executed at the storage layer by FEAT-16.SPEC-006; no restore path is needed because no in-product deletion path exists; no cascade beyond the deleted account's own records; retained for the life of the account with no automatic purge while the account is active (ASMP-22) | An explicit non-goal, not an omission -- recorded in Non-Goals |
| State Transition | N/A | `is_latest` is a derived field (Data Notes: "Derived: 'latest version' is the most recent upload"), not a stored state; the entity carries no status field | Derivation logic is owned by FEAT-17.SPEC-004 |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Deliverable | FEAT-17.SPEC-001, FEAT-17.SPEC-002 | The re-upload screen is scoped to one existing deliverable; the version browser is reached from that deliverable's view and shows the deliverable's own identity (name, milestone) alongside its version list |
| Comment | FEAT-17.SPEC-005 | The anchoring rule reasons about which version a comment is pinned to, but comment creation, editing, and retraction remain owned by FEAT-07 -- this feature never writes a Comment record |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia uploads a revised file to an existing deliverable | Begin the re-upload transfer; the deliverable's current (prior) version stays Active and visible throughout | Standalone Automation | FEAT-17.SPEC-003 |
| The re-upload transfer completes fully | Create the next Deliverable Version at the next round_number, mark it `is_latest`, leave every prior version's own record untouched; emit `version_uploaded` | Standalone Automation | FEAT-17.SPEC-003 |
| The re-upload transfer fails or is interrupted | Do not create a new version and do not touch the existing one -- the prior version remains intact and current; no premature client notification fires (XBR-12) | Standalone Automation | FEAT-17.SPEC-003 |
| A new version becomes fully ready | Reuse FEAT-06's "new deliverable ready" notification for the relevant client contacts, for this version | Cross-feature | FEAT-06.SPEC-006 (reused as-is; no new Notification spec required -- see Shared Context) |
| A viewer opens the version browser on a deliverable | Determine round numbering and which round is "latest" | Standalone Logic/Rule | FEAT-17.SPEC-004 |
| A viewer opens the version browser or a re-upload screen | Check role-based access (Nadia Full; Owen/Priya Own-only view; Dana View-only, no download, inside a logged support session) before showing the screen | Standalone Logic/Rule | FEAT-17.SPEC-005 |
| A viewer opens the version browser | Show every round by number; hide the selector entirely when the deliverable has only one version | Inline in triggering screen | FEAT-17.SPEC-002 |
| A viewer switches between versions of a large file | Show a brief loading indicator while the selected round's file loads | Inline in triggering screen | FEAT-17.SPEC-002 |
| Nadia opens the re-upload screen and submits | Confirm the round was added and return to the deliverable/version view | Inline in triggering screen | FEAT-17.SPEC-001 |
| A client contact posts a comment while viewing an older round | The comment stays anchored to that round even after a newer round is uploaded | Standalone Logic/Rule | FEAT-17.SPEC-005 (write itself stays inline in FEAT-07's comment screens) |
| Any version upload | Transfers and stores bytes through the shared large-file storage capability and counts against the freelancer's storage allowance (XBR-13, XBR-14) | Cross-feature | FEAT-16.SPEC-002 / FEAT-16.SPEC-007 responsibility |
| A new version is created, or a removal is blocked on an approved milestone (XBR-11) | An append-only activity-trail entry is written with actor and timestamp | Cross-feature | FEAT-13 responsibility |
| Dana opens a read-only support session | Sees the version browser's list and metadata but never downloads a version's file | Cross-feature | FEAT-31 responsibility (governs the session; FEAT-17.SPEC-005 enforces the no-download constraint within it) |

## Shared Context

**Shared Entities:**
- Deliverable Version -- created (round 2 onward) by SPEC-003; listed and opened by SPEC-002; governed by SPEC-004 (numbering, immutability, retention) and SPEC-005 (access, anchoring). Fields: round_number, file, uploaded_at, is_latest (derived).
- Deliverable (read-only) -- read by SPEC-001 to scope the re-upload and by SPEC-002 to anchor the version browser to its parent deliverable; created and owned entirely by FEAT-06.

**Shared UI Patterns:**
- Version selector -- SPEC-002 is the single implementation FEAT-07's deliverable view navigates into (dependency map navigation row: "deliverable view → version selector / earlier round"); it is hidden when a deliverable has exactly one version (Empty state) and shows a brief loading indicator when switching rounds on a large file (Loading state). Spec Writers should describe this component once and have FEAT-07 reference it rather than re-specifying it.
- Round-number and "latest" labeling -- the same round_number and is_latest vocabulary appears in SPEC-001's post-upload confirmation and throughout SPEC-002's browser; Spec Writers should keep the labeling identical across both.

**Shared Validation:**
- SPEC-004 defines every numbering, immutability, and retention rule (sequential round_number with no gaps, immutability once uploaded, no fixed version-count cap bounded by the storage allowance, is_latest derivation). SPEC-001 and SPEC-003 both reference SPEC-004 rather than duplicating these rules.
- SPEC-005 defines the only access and anchoring rules this feature has. SPEC-001, SPEC-002, and FEAT-07's comment screens all reference SPEC-005 rather than re-deriving who may view/upload a version or how a comment binds to its round.

**Flagged discrepancy (not resolved by this Brief):** This feature carries `integration_count: 0` and `notification_count: 0` by deliberate resolution, not omission. The dependency map's External Touchpoints row lists this feature's large-file storage dependency as "pending, assigned when its batch is validated" -- resolved here the same way FEAT-06 (already validated) resolved its identical dependency: this feature consumes the shared storage capability entirely through FEAT-16.SPEC-007 and owns no Integration spec of its own (mirrors FEAT-06.SPEC-003 → FEAT-16, XBR-14). Separately, the Communications field names only a reuse of FEAT-06's existing "new deliverable ready" notification for each new version, with no new channel, audience, or content of its own -- so no new Notification spec is warranted here either. Both zero counts are surfaced for the Requirements Architect, not silently left as gaps.

## Internal Dependency Map

```
SPEC-001 (New Version Upload) -> [Nadia re-uploads a file to an existing deliverable] -> SPEC-003 (Version Creation & Preservation)
SPEC-003 (Version Creation & Preservation) -> [applies numbering/immutability using] -> SPEC-004 (Version Numbering, Immutability & Retention Rules)
SPEC-003 (Version Creation & Preservation) -> [checks role access using] -> SPEC-005 (Version Access & Comment-Anchoring Rules)
SPEC-003 (Version Creation & Preservation) -> [new round created, is_latest updated] -> SPEC-002 (Version Browser & Comparison)
SPEC-003 (Version Creation & Preservation) -> [version fully ready] -> FEAT-06.SPEC-006 (Deliverable Ready Notification, reused per version)
SPEC-002 (Version Browser & Comparison) -> [checks role access using] -> SPEC-005 (Version Access & Comment-Anchoring Rules)
SPEC-002 (Version Browser & Comparison) -> [viewer opens an older round and comments] -> FEAT-07 (Deliverable Review & Feedback comment thread) -> [comment bound to that round using] -> SPEC-005
```

**Default Entry:** SPEC-002 (Version Browser & Comparison) -- reached from FEAT-07's deliverable view via the version selector whenever more than one round exists; SPEC-001 is reached separately when Nadia chooses to add a new round to an existing deliverable.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-17.SPEC-001 | Inbound | FEAT-06 (Deliverable Upload & Sharing) | Nadia reaches "Upload New Version" from an existing deliverable's management screen | Nadia chooses "replace" on FEAT-06.SPEC-002 |
| FEAT-17.SPEC-003 | Outbound | FEAT-16 (Large File Handling & Storage) | The re-upload's bytes are transferred and stored through FEAT-16's resumable transfer, and count against the storage allowance | Nadia uploads a new version (XBR-13, XBR-14) |
| FEAT-17.SPEC-003 | Outbound | FEAT-06 (Deliverable Upload & Sharing) | On full completion, reuses FEAT-06.SPEC-006's deliverable-ready notification for the new version, never for a partial or failed upload | A new version finishes uploading (XBR-12) |
| FEAT-17.SPEC-003 | Inbound | FEAT-08 (Milestone Approval) | An approved milestone blocks outright deliverable removal but still allows superseding through a new version, which this feature provides | Owen approves the milestone; Nadia later uploads a new round (XBR-11) |
| FEAT-17.SPEC-002 | Inbound | FEAT-07 (Deliverable Review & Feedback) | The deliverable view's version selector opens directly into this feature's version browser | A viewer opens an earlier version of the deliverable |
| FEAT-17.SPEC-005 | Outbound | FEAT-07 (Deliverable Review & Feedback) | Governs that a comment posted on an older version stays anchored to that version regardless of later uploads | Nadia, Owen, or Priya comments on any version |
| FEAT-17.SPEC-002 / FEAT-17.SPEC-005 | Inbound | FEAT-31 (Operator Support Access) | Dana's read-only, logged support session can browse versions but never download a version's file | Dana opens a support session on the freelancer's account |
| FEAT-17.SPEC-003 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Every new version creation, and every removal blocked in favor of supersession, writes an append-only trail entry with actor and timestamp | A new version is created, or removal is blocked on an approved milestone |
| FEAT-17.SPEC-003 / FEAT-17.SPEC-004 | Outbound | FEAT-24 (Account Deletion) | Deliverable Version records and their stored bytes are removed only as part of account deletion, executed at the storage layer by FEAT-16.SPEC-006 | Nadia's account is deleted |

## Non-Functional Notes

**Data volumes / growth:** Every preserved version counts against the freelancer's storage allowance (XBR-13), and version accumulation is named directly as the main driver of year-one storage volume (success-metrics.md, Large File Upload Success at Scale); versions are retained with no fixed cap and no automatic purge for the life of the account (ASMP-22), so this feature's own data volume grows without bound across an account's lifetime by design.

**Responsiveness:** Switching between versions of a large file shows a brief, visible loading indicator rather than an indefinite spinner (feature's States field, ASMP-27); the client-facing version browser and comment-anchoring flow fall under the client-facing 2-second interactivity target (ASMP-21), directly measured by the Client Portal Mobile Responsiveness success metric since clients switch between and open versions almost entirely on mobile browsers.

**Data sensitivity / privacy:** Each Deliverable Version carries the same client-confidential work product as its parent Deliverable (design files, videos, documents that may themselves contain personal data), strictly isolated per client (ASMP-23); Dana's read-only support session can list and browse versions but never download the underlying file (ASMP-23, mirroring FEAT-06/FEAT-16's Support Access boundary).

**Compliance flags:** GDPR-class handling applies only to the extent a version's file content or its anchored comments carry personal data (ASMP-24) -- the version metadata this feature itself manages (round_number, uploaded_at) carries no personal data of its own; no card or payment data is ever involved in this feature.

## Non-Goals

- **Editing or replacing the file of an already-uploaded version** -- Excluded by the feature's own Validation & Limits field ("each version is immutable once uploaded"); the only path to a changed file is a new round through FEAT-17.SPEC-001/SPEC-003, never an in-place edit of an existing round.
- **Manually deleting or purging an individual version while the account is active** -- Intentional lifecycle decision surfaced by the CRUD matrix and carried from the dependency map ("never deleted in-product; removed only by FEAT-24") and ASMP-22 (retained for the life of the account); a version's evidentiary and version-history role means no in-product delete or purge action exists.
- **Manually marking an older round as "current" or "latest"** -- Intentional design decision: Data Notes defines `is_latest` purely as a derived field ("the most recent upload"), with no override mechanism described anywhere in Stage 2; the only way to make an older round's content current again is to upload it again as a brand-new round.
- **Automated cross-version content diffing or side-by-side visual comparison** -- The Key Capabilities line ("open and compare any round by number") and the Data Notes field describe comparison as opening any round through the version selector, with no diff mechanics, supported file types, or comparison UI described anywhere in Stage 2; a computed content-diff tool is out of scope for this Brief absent that elaboration.
- **Scoped upload or view permissions for a freelancer-side team** -- Excluded per scope-boundaries.md (SC-01): the product is solo-freelancer only for v1, so this feature has exactly one writer role (Nadia); no bookkeeper- or contractor-style scoped version permission exists.
- **Operator downloading a version's file, or editing/removing versions in a support session** -- Excluded per scope-boundaries.md (SC-04): Dana's support session is read-only and never downloads deliverable files or their versions, and never edits or removes anything.
- **Native mobile version-browsing app** -- Excluded per scope-boundaries.md (SC-06): the product is a web app with no native apps; version browsing and comparison happen entirely through the mobile-and-desktop-browser client experience.



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



# Screen Spec: Version Browser & Comparison

## Overview

**Name:** Version Browser & Comparison
**ID:** FEAT-17.SPEC-002
**Type:** Screen
**Purpose:** Any authorized viewer opens the version selector on a deliverable, browses every round by number, and opens any earlier round alongside the latest.
**Parent Feature:** FEAT-17 -- Deliverable Version History

## Scope and Non-Goals

**In Scope:**
- Listing every Deliverable Version for a deliverable, ordered by round_number, with the latest round labeled
- Opening any single round's file and metadata (round_number, uploaded_at)
- Hiding the version selector entirely when the deliverable has exactly one version
- Showing a brief loading indicator when switching between rounds on a large file
- Serving as the single shared implementation FEAT-07's deliverable view navigates into for any version-specific viewing

**Non-Goals:**
- Uploading a new version -- handled by FEAT-17.SPEC-001 (New Version Upload); this screen is read-only for the Deliverable Version entity
- Automated cross-version content diffing or a side-by-side visual comparison UI -- excluded per the Brief's Non-Goals: the Key Capability "open and compare any round by number" and Stage 2's Data Notes describe comparison as opening any round through this selector, with no diff mechanics or comparison UI described anywhere in Stage 2
- Posting, editing, or retracting comments -- owned entirely by FEAT-07 (Deliverable Review & Feedback); this screen only reads that a round can be commented on and links into FEAT-07's thread for the open round
- Editing or removing an existing version -- excluded per FEAT-17.SPEC-004: each version is immutable once uploaded, and no in-product delete path exists while the account is active

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-07 (Deliverable Review & Feedback, deliverable view) | A viewer opens an earlier version of the deliverable via the deliverable view's version selector (dependency map navigation row) | Deliverable reference; the round_number the viewer was looking at in FEAT-07, if any |
| FEAT-17.SPEC-001 (New Version Upload) | Nadia taps "View versions" after her upload completes | Deliverable reference; the newly created round's round_number, opened by default as latest |
| FEAT-31 (Operator Support Access) | Dana opens a read-only support session and navigates to a deliverable's version history | Deliverable reference, read-only session flag (no download control rendered) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, every version of her own deliverables | Open any round; download any round's file | -- |
| Owen (Client Primary Contact) | Every version of his own company's deliverables (Own-only) | Open and view any round of his own company's deliverables | Versions belonging to another client company never appear or resolve; a direct link to another company's deliverable shows "This page isn't part of your portal." per XBR-09 |
| Priya (Client Reviewer Contact) | Every version of her own company's deliverables (Own-only) | Open and view any round of her own company's deliverables | Same as Owen -- versions outside her own client company never appear or resolve |
| Dana (Support Operator) | The version list and each round's metadata (round_number, uploaded_at), inside a logged, read-only support session (FEAT-31) | View only -- no file download control is rendered for any round | A direct attempt to open a version's underlying file is refused with "Downloads are not available in a support session." (ASMP-18, ASMP-23; FEAT-17.SPEC-005) |
| Unauthenticated | No | No | Client contacts: redirected to request a fresh magic link (FEAT-05). Nadia: redirected to the freelancer sign-in screen. |
| Expired session | No | No | Client contacts see the expired/invalid link explanation with a one-tap way to request a fresh link (FEAT-05); Nadia sees "Your session has expired. Sign in to continue." Neither preserves the round that was open -- the screen reopens on latest after re-authentication. |

## Layout and Content

**Header:** Deliverable name, milestone name, and project name (read-only), with a back arrow returning to the entry point per the Navigation Out table.

**Body:**
- **Version selector** -- a horizontally scrollable row of round chips ("Round 1", "Round 2", ... "Round {N} (Latest)"), one per Deliverable Version, ordered by round_number ascending, with the currently open round highlighted. Not rendered at all when the deliverable has exactly one version (Single Version state, below).
- **Round detail panel** -- below the selector, shows the open round's file preview area (where feasible) or a generic file icon with name and size, plus its metadata: "Round {round_number} -- uploaded {uploaded_at}" and, when applicable, "Latest version" label.
- **Comment count indicator** -- display-only badge on the round detail panel showing the number of comments anchored to the open round (per FEAT-17.SPEC-005's anchoring rule), linking into FEAT-07's thread for that round.
- **Download control** -- visible to Nadia, Owen, and Priya only; never rendered for Dana's support session.

### Responsive Behavior

- **Compact breakpoint:** Version selector chips scroll horizontally in a single row; round detail panel stacks full-width below it.
- **Medium size class and above:** Version selector and round detail panel remain in the same relative arrangement; the detail panel's file preview area grows to fill the available width, capped at a consistent platform-wide content width.
- **Comment count indicator:** Uniform scaling, no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the entry point per Navigation Out | Screen closes | Returns to the deliverable view, the upload screen's origin, or the support session view |
| Round chip | Tap | Loads the selected round's file and metadata | Selected chip highlighted; round detail panel updates | Brief loading indicator shown while a large file's content loads (Loading state, below) |
| File preview area (round detail panel) | Tap (or keyboard Enter on the focused preview) | Where the file type supports inline preview: images and PDFs open in a zoomable full-width view; video and audio play inline with standard playback controls; other previewable types open static. Unpreviewable types show the generic file icon and this tap does nothing | Preview enlarges or begins playback; no data changes | Standard playback/zoom feedback (browser-native); a second tap or the close control returns to the detail panel |
| Disabled round chip (offline; unopened round) | Tap | No action -- the chip is disabled while offline | None | Chip shows "Available when you reconnect"; it becomes selectable again automatically on reconnection |
| "Try again" (Error state) | Tap | Re-attempts resolving the deliverable and its version list | Full-screen message replaced by the Loading (screen) placeholder | On success, the version selector and detail panel render; on repeated failure, the Error message reappears |
| Comment count indicator | Tap (Nadia, Owen, Priya only) | Navigates into FEAT-07's comment thread, scoped to the open round | Screen navigates away | Opens FEAT-07's deliverable comment thread for this round |
| Download control (Nadia, Owen, Priya only) | Tap | Downloads the open round's file | None on this screen | Standard file-download feedback (browser-native) |

### Accessibility Notes

- **Focus order:** Back arrow -> version selector chips (in round order; disabled chips are announced as unavailable) -> round detail panel content, including the file preview area -> comment count indicator (when visible) -> download control (when visible).
- **Status announcements:** Switching rounds announces the new round's label ("Round {N}, uploaded {date}") to assistive technology once its content finishes loading. The Single Version state's absence of a selector is not separately announced -- the round detail panel alone is the entire interactive surface in that state.
- **Keyboard alternatives:** Round chips are reachable and selectable by keyboard (arrow keys move between chips, Enter/Space selects); the download control has no pointer-only equivalent requiring a mouse.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Single Version | No version selector rendered; round detail panel shows the deliverable's only version directly, with no "Round 1" chip or "Latest" label shown (nothing to distinguish it from) | The deliverable has exactly one Deliverable Version | A second version is created (screen re-renders the selector on next load) |
| Multiple Versions (default) | Version selector with 2+ chips; latest round open by default unless entry-point context specifies another | The deliverable has 2 or more Deliverable Versions | -- |
| Loading (round switch) | Round detail panel shows a brief in-progress indicator in place of the file content | User taps a different round chip on a large file | The selected round's content finishes loading |
| Loading (screen) | Full-panel loading placeholder in place of the version selector and detail panel | Screen first opens while the deliverable and its versions are being resolved | Data resolves, or resolution fails (Error, below) |
| Error | Full-screen message "This deliverable's versions couldn't be loaded." with a "Try again" action | The deliverable or version list fails to resolve | User taps "Try again" and resolution succeeds |
| Offline/Degraded | Banner "You're offline -- showing versions you've already viewed." at top; previously loaded rounds remain viewable read-only; unopened rounds show a disabled chip with "Available when you reconnect" | Connectivity is lost while this screen is open | Connectivity returns and all rounds become selectable again |

## Validation Rules

Validation governed by FEAT-17.SPEC-004 (Version Numbering, Immutability & Retention Rules) for round ordering and the latest-version derivation, and by FEAT-17.SPEC-005 (Version Access & Comment-Anchoring Rules) for who may view or download each round. This screen has no user input beyond selecting a round or a download action.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-07 (deliverable view), FEAT-17.SPEC-001 (post-upload), or the FEAT-31 support session view -- whichever entry point was used | FEAT-07 or FEAT-31, per entry point |
| Comment count indicator tap | FEAT-07 (deliverable comment thread), scoped to the open round | FEAT-07 (Deliverable Review & Feedback) |

## Data Model

**Creates:** None.
**Reads:** Deliverable -- name, milestone, project (header context). Deliverable Version -- round_number, file, uploaded_at for every version of this deliverable (selector and detail panel); the "Latest" label is derived from the highest round_number (FEAT-17.SPEC-004), not read from a stored field. Comment -- count of comments whose target is the open Deliverable Version, for the comment count indicator (FEAT-07 owns Comment; this screen only reads a count).
**Updates:** None.
**Deletes:** None.

## Business Rules

- FEAT-17.SPEC-004 governs round ordering (sequential, no gaps) and which round is labeled "Latest" (the is_latest derivation); this screen never computes its own ordering or latest determination.
- FEAT-17.SPEC-005 governs who may view, open, and download each round; Dana's download control is never rendered regardless of session state (ASMP-18).
- The version selector is hidden entirely when only one version exists (feature's States field), so a first-round deliverable looks identical to a non-versioned one until a second round is uploaded.
- XBR-09: client isolation applies here as everywhere in the portal -- a version belonging to another client's deliverable is never reachable, resolvable, or shown to Owen or Priya.
- This screen is a snapshot per load, not live-updating: a version uploaded by Nadia in another session while a viewer has this screen open does not appear until the screen is reopened or reloaded; the currently open round remains valid and unaffected either way, since versions are immutable and never replaced in place.

## Edge Cases

- **A new version is uploaded while a viewer has this screen open on an older round** -- No live conflict: this screen is a snapshot, not live-updating (Business Rules, above). The round the viewer is looking at remains fully valid and unchanged; the new round simply does not appear in the selector until the screen is reopened. This is not a concurrent-edit conflict because this screen performs no write against Deliverable Version -- the entity's own Contention note states no concurrent modification of a version is possible, since versions are appended, never edited in place.
- **Deliverable has zero versions (upload still in progress or never started)** -- Not reachable through this feature's own entry points, since FEAT-06.SPEC-003 creates round 1 only on full upload completion; a stale link surfaces the Error state.
- **A round's file preview cannot be rendered inline (unsupported file type)** -- The round detail panel falls back to a generic file icon, name, and size, with the download control (where visible) still available.
- **Viewer switches rounds rapidly (taps several chips in quick succession)** -- Only the most recently tapped round's load is shown; earlier in-flight loads for previously tapped rounds are discarded when they resolve.
- **Comment count indicator shows zero** -- Indicator still renders with "0 comments"; tapping it still opens FEAT-07's empty-state thread for that round.
- **Dana's support session ends while a version's content is loading** -- The in-progress load is cancelled and the screen redirects to the support session's closed-session experience, per FEAT-31.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07 (Deliverable Review & Feedback) | Navigation (inbound/outbound) | Deliverable view's version selector opens directly into this screen; comment count indicator navigates into FEAT-07's thread for the open round |
| FEAT-17.SPEC-001 (New Version Upload) | Navigation (inbound) | Destination of the "View versions" button after a completed upload |
| FEAT-17.SPEC-004 (Version Numbering, Immutability & Retention Rules) | References (inbound) | Round ordering and the "Latest" derivation |
| FEAT-17.SPEC-005 (Version Access & Comment-Anchoring Rules) | References (inbound) | Who may view, open, or download each round; Dana's no-download constraint |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Entry point for Dana's read-only support session; governs session timing and closure |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| version_browser_opened | deliverable reference, version count, viewer role | Screen loads successfully | supports success-metrics.md: "Client Portal Mobile Responsiveness" (this screen is the client-facing surface the metric's 2-second interactivity target measures for Owen and Priya's version-viewing sessions; for Nadia and Dana this event is retained for feature-level visibility only) |
| version_opened | round_number, whether it is the derived latest, load duration | A viewer selects a round and its content finishes loading | supports success-metrics.md: "Client Portal Mobile Responsiveness" |
| version_compared | count of distinct rounds opened in the current screen session | A viewer opens a second or further round after already viewing one in the same session | N/A -- no Stage 2 metric measures manual round-to-round comparison specifically; retained for feature-level visibility since the Key Capability "open and compare any round by number" has no dedicated Stage 2 metric |

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 6 (single version, multiple versions, loading round, loading screen, error, offline) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Version Creation & Preservation

## Overview

**Name:** Version Creation & Preservation
**ID:** FEAT-17.SPEC-003
**Type:** Automation
**Purpose:** On a fully completed re-upload, creates the next immutable Deliverable Version, and leaves every prior version's own record untouched; the new round becomes "latest" by derivation (highest round_number), never by a write to a prior version.
**Parent Feature:** FEAT-17 -- Deliverable Version History

## Scope and Non-Goals

**In Scope:**
- Receiving the file selected on FEAT-17.SPEC-001 and beginning the re-upload transfer through the large-file storage capability
- Pausing and automatically resuming the transfer across a dropped connection, without restarting from zero
- Committing the next Deliverable Version only once the transfer completes fully, never for a partial file
- Deriving and assigning the next sequential round_number, leaving every prior version's record completely untouched
- Confirming the triggering session is Nadia's (defense in depth behind FEAT-17.SPEC-001's screen-entry check, per FEAT-17.SPEC-005) and that the deliverable is eligible for a new version
- Ensuring the new round is the derived latest (`is_latest` is computed from the highest round_number per FEAT-17.SPEC-004, never stored, so no prior version is written)
- Emitting the completion signal that FEAT-06.SPEC-006's reused notification waits for, and the append-only trail entry request that FEAT-13 records (actor, timestamp; XBR-05)

**Non-Goals:**
- Creating round 1 of a deliverable -- owned by FEAT-06.SPEC-003 (Resumable Upload Handling); this automation owns round 2 onward only (Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix)
- The byte-level storage write and resumable chunk protocol -- owned by FEAT-16.SPEC-002 (Resumable Upload Transfer); this automation is the version-layer automation that invokes it and interprets its outcome
- Defining the content of the "new deliverable ready" notification -- reused as-is from FEAT-06.SPEC-006; this automation only signals when that notification's trigger condition is met
- Defining who is authorized to trigger a re-upload -- the rule is owned by FEAT-17.SPEC-005 (Version Access & Comment-Anchoring Rules) and enforced first at FEAT-17.SPEC-001's entry; this automation only re-confirms it (Processing Logic step 2) and does not define it
- Storing or editing the trail entry -- FEAT-13 owns the trail's storage and immutability; this automation only supplies the entry's actor and timestamp

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| File selected for a new version | FEAT-17.SPEC-001 (New Version Upload) | Fires immediately when Nadia selects a file on a deliverable that already has at least one uploaded-file version | File bytes, file name, file size, the deliverable reference |
| Connectivity restored during an in-progress transfer | FEAT-16.SPEC-002 (Resumable Upload Transfer) | Fires when the storage capability reports connectivity has returned for a transfer this automation started | Transfer identifier, bytes already committed, remaining bytes |
| Retry tapped after a non-connectivity failure | FEAT-17.SPEC-001 (New Version Upload) | Fires when Nadia taps Retry on a failed transfer, using the already-selected file | Same file bytes as the original attempt, previous failure reason |

## Processing Logic

1. Receive the selected file (bytes, name, size) and the deliverable reference from FEAT-17.SPEC-001.
2. Authorization (per FEAT-17.SPEC-005): confirm the triggering session is Nadia's active, authenticated session. If not (client contact, support operator, expired or missing session), stop and produce the Failure (not authorized) outcome; no transfer starts and no version is created.
3. Validate the file: if its size is zero bytes, stop with the Failure (empty file) outcome; if it exceeds the per-file ceiling (platform parameter: `deliverable-file-size-ceiling`, per FEAT-06.SPEC-005), stop with the Failure (size ceiling) outcome. Neither touches the deliverable's existing versions.
4. Confirm the target deliverable is eligible. The deliverable must (a) still exist, (b) be a deliverable whose kind is an uploaded file (a linked external asset never takes a new version), and (c) already have a committed round 1 (its first upload completed under FEAT-06.SPEC-003). These are the only conditions that block a new version. The milestone's approval status is not one of them: a deliverable on an approved milestone is superseded through a new version exactly as on any other milestone (XBR-11; FEAT-08 approval blocks removal only). If (a) fails, produce the Failure (deliverable removed) outcome; if (b) or (c) fails, produce the Failure (deliverable not eligible) outcome.
5. Confirm the large-file storage capability (FEAT-16.SPEC-007) can accept transfers; if it reports it cannot, produce the Failure (capability unavailable) outcome. Otherwise hand the file to it (FEAT-16.SPEC-002) to begin a resumable, chunked transfer, and report progress back to FEAT-17.SPEC-001 as it advances. The deliverable's current latest version stays Active and fully visible throughout -- nothing about it changes while the transfer runs.
6. If the connection drops mid-transfer, pause the transfer, persist the resume point (via FEAT-16.SPEC-002's own resume-state handling), and surface the Paused/Resuming state to FEAT-17.SPEC-001. When connectivity returns, resume automatically from the persisted point without restarting from zero.
7. If a transfer chunk fails for a reason other than dropped connectivity, retry that chunk automatically (via FEAT-16.SPEC-002's own retry) before surfacing any failure to the user. If the storage capability reports the freelancer's storage allowance (FEAT-16.SPEC-004) is exhausted, stop and produce the Failure (storage allowance exhausted) outcome.
8. When the transfer completes fully, commit: read the deliverable's current highest round_number (per FEAT-17.SPEC-004's sequencing rule) and create a new Deliverable Version with round_number set to that value plus one, file set to the stored file reference, and uploaded_at set to the completion time. This is the only write this automation makes to Deliverable Version, and it is a create; every prior version's record is read but never written to. The new round is the latest by derivation (it now holds the highest round_number, FEAT-17.SPEC-004); no `is_latest` value is stored or updated on any version.
9. Emit `version_uploaded`.
10. Request the append-only trail entry from FEAT-13 (actor: Nadia; timestamp: the completion time; subject: the new Deliverable Version and its deliverable; XBR-05). This happens only on successful commit at step 8; a failed or cancelled transfer writes no trail entry.
11. Signal FEAT-06.SPEC-006 (reused "new deliverable ready" notification) so the relevant client contacts are notified for this new version -- only after step 8 has committed, never for a partial file (XBR-12).
12. If the transfer fails for any other reason after step 7 (server rejection, persistent storage-capability error), create no Deliverable Version at all, write no trail entry, and leave the existing latest version exactly as it was.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Not authorized | The triggering session is not Nadia's active session | No transfer starts; no Deliverable Version created; no trail entry | FEAT-17.SPEC-001 shows its Expired session dialog or the client/support unauthorized experience; no upload proceeds | FEAT-17.SPEC-001, FEAT-17.SPEC-005 |
| Empty file | Selected file is zero bytes | No Deliverable Version created; existing versions untouched | FEAT-17.SPEC-001 shows "This file appears to be empty." without a Retry control | FEAT-17.SPEC-001 |
| Size ceiling exceeded | Selected file's size exceeds platform parameter: `deliverable-file-size-ceiling` | No Deliverable Version created; existing versions untouched | FEAT-17.SPEC-001 shows "This file is larger than the size limit for deliverables. Compress it or share it by link instead." without a Retry control | FEAT-17.SPEC-001, FEAT-17.SPEC-004 |
| Deliverable no longer exists | The target deliverable is removed before the transfer commits | No Deliverable Version created | FEAT-17.SPEC-001 shows "This deliverable no longer exists." without a Retry control | FEAT-17.SPEC-001 |
| Deliverable not eligible | The deliverable is a linked external asset, or has no committed round 1 | No Deliverable Version created | FEAT-17.SPEC-001 shows "New versions can't be added to this deliverable." without a Retry control | FEAT-17.SPEC-001, FEAT-06.SPEC-002 |
| Transfer in progress | Transfer is actively moving bytes | No entity changes yet; existing latest version remains Active | FEAT-17.SPEC-001 shows live progress percentage | FEAT-17.SPEC-001 |
| Paused, auto-resuming | Connectivity drops mid-transfer | No entity changes; transfer resume state persisted | FEAT-17.SPEC-001 shows "Paused -- resuming when your connection returns" | FEAT-17.SPEC-001 |
| Transfer completes fully | All bytes committed to storage | New Deliverable Version created at round_number = previous highest + 1 (create only); no field of any prior version is written, including any latest marker, since latest is derived; one append-only trail entry requested from FEAT-13 (actor, timestamp) | FEAT-17.SPEC-001 shows "Round {N} added" with a "View versions" action; FEAT-17.SPEC-002's selector gains the new round as latest | FEAT-17.SPEC-001, FEAT-17.SPEC-002, FEAT-06.SPEC-006 (notification fires), FEAT-13 (trail entry) |
| Transfer fails (non-connectivity, non-ceiling) | Server rejection or persistent storage-capability error, after automatic chunk retries are exhausted | No Deliverable Version created; existing versions untouched; no trail entry | FEAT-17.SPEC-001 shows Upload Failed with a Retry control and the selected file preserved | FEAT-17.SPEC-001 |
| Storage allowance exhausted | The storage capability reports the freelancer's allowance is exhausted (FEAT-16.SPEC-004) before or during the transfer | No Deliverable Version created; no trail entry | FEAT-17.SPEC-001 shows "You've reached your storage allowance." with a link to the plan view (FEAT-23), without a Retry control | FEAT-17.SPEC-001, FEAT-16.SPEC-004 |
| Retry succeeds | Nadia retries a failed transfer and it completes | Same as "Transfer completes fully" | Same as "Transfer completes fully" | FEAT-17.SPEC-001, FEAT-17.SPEC-002, FEAT-06.SPEC-006, FEAT-13 |
| Automation itself unavailable | The large-file storage capability (FEAT-16.SPEC-007) reports it cannot accept transfers at all | No entity changes | FEAT-17.SPEC-001 shows "Uploads aren't available right now. Try again shortly." with a Retry control | FEAT-17.SPEC-001, FEAT-16.SPEC-007 |

## Data Model

**Reads:** Deliverable -- existence, kind (uploaded file or link), and its milestone's approval status (read only to confirm approval does not block), to confirm the transfer's target is valid. Deliverable Version -- the current highest round_number, to derive the new round's number.
**Creates:** Deliverable Version record (round_number, file, uploaded_at) on transfer completion.
**Updates:** None. No field of any existing Deliverable Version is ever modified (immutability, FEAT-17.SPEC-004); `is_latest` is derived from the highest round_number and is not stored, so nothing is written to the prior latest version.
**Deletes:** None. (Version records are removed only by FEAT-24 account deletion, executed at the storage layer by FEAT-16.SPEC-006.)

## Business Rules

- XBR-12: clients are notified about a new version only once its upload has fully completed, never for a partial file; interrupted uploads resume rather than restart.
- XBR-13: each re-upload creates a new immutable version; comments stay attached to the version they were made on (FEAT-17.SPEC-005); all versions count against the freelancer's storage allowance.
- XBR-14: the per-file size ceiling (platform parameter: `deliverable-file-size-ceiling`) is checked by this automation before transferring bytes; the per-freelancer storage allowance (platform parameter: `free-tier-storage-allowance` / `paid-tier-storage-allowance`) is enforced by FEAT-16.SPEC-004 during the transfer itself.
- FEAT-17.SPEC-004 governs round_number sequencing (no gaps, strictly increasing) and the `is_latest` derivation; this automation is the sole creator of new versions, and committing a new highest round_number is what makes the new round latest.
- The Deliverable Version created by this automation is a single feature's write: only Nadia's active session can start a re-upload transfer, consistent with the dependency map's Contention note that Nadia is the only writer for Deliverable Version.
- Every prior version's record is read-only to this automation once created (immutability): this automation never writes any field of an existing version, including any latest marker. Its only write to Deliverable Version is creating the new record.
- XBR-11: a deliverable on an approved milestone (FEAT-08) cannot be removed but can always be superseded; approval never blocks a new version. The only conditions that block upload are those in Processing Logic step 4.
- XBR-05: each successful version creation causes exactly one append-only FEAT-13 trail entry with actor and timestamp; failures cause none.
- Version records are removed only by FEAT-24 account deletion (FEAT-16.SPEC-006); this automation has no delete path.

## Edge Cases

- **File has zero bytes** -- Rejected before a transfer starts, surfaced to FEAT-17.SPEC-001 as "This file appears to be empty." with no Deliverable Version created.
- **Connection drops and returns within the same second (flapping connectivity)** -- The pause/resume cycle is debounced: a reconnect within a short window resumes without visibly flashing the Paused state to the user.
- **Storage capability reports the freelancer's storage allowance is exhausted mid-transfer** -- The transfer stops; no Deliverable Version is created; the failure is surfaced as "You've reached your storage allowance." per FEAT-16.SPEC-004's warning, with a link into the plan view (FEAT-23); this is treated as a Failure outcome, not a size-ceiling outcome.
- **User closes the browser tab mid-transfer** -- The transfer is cancelled; no partial Deliverable Version is ever created; the deliverable's existing latest version remains exactly as it was.
- **Concurrent trigger firing (two re-upload transfers started for the same deliverable in two sessions of Nadia's at effectively the same time)** -- Each transfer proceeds independently. Whichever transfer commits first is assigned the next round_number; the second transfer, on commit, re-reads the (now-updated) highest round_number and is assigned the number after that, also being the derived latest in turn. Both rounds are preserved -- neither is rejected, discarded, or made to overwrite the other, consistent with the dependency map's Contention note that no concurrent modification of a version is possible because rounds are appended, never edited in place.
- **Trigger fires while a previous run is in flight (Retry tapped while the original attempt is still retrying a chunk automatically)** -- The manual Retry request is ignored while an automatic chunk retry is already in progress for the same transfer; the existing transfer continues rather than starting a duplicate.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-001 (New Version Upload) | Triggered by (inbound) | File selection and Retry both start or resume this automation |
| FEAT-17.SPEC-004 (Version Numbering, Immutability & Retention Rules) | References (inbound) | Round-number sequencing and `is_latest` derivation logic this automation applies (create-only; no prior-version write) |
| FEAT-17.SPEC-005 (Version Access & Comment-Anchoring Rules) | References (inbound) | Authorization check in Processing Logic step 2: the triggering session must be Nadia's |
| FEAT-16.SPEC-002 (Resumable Upload Transfer) | Triggers (outbound) | Performs the actual byte-level resumable transfer and reports progress, pause, and completion |
| FEAT-16.SPEC-007 (Large File Storage Capability) | References (outbound) | Reports whether the storage capability can accept transfers at all (Automation-unavailable outcome) |
| FEAT-16.SPEC-004 (Storage Limits) | References (inbound) | Reports storage-allowance exhaustion, producing the Storage-allowance-exhausted outcome |
| FEAT-08 (Milestone Approval) | References (inbound) | An approved milestone blocks removal but not a new version (XBR-11); this automation does not treat approval as a blocking state |
| FEAT-24 (Account Deletion) | References (inbound) | Sole path by which version records and bytes are removed |
| FEAT-06.SPEC-006 (Deliverable Ready Notification) | Affects (outbound) | Reused as-is; this automation's completion is the trigger it waits for, per version |
| FEAT-17.SPEC-002 (Version Browser & Comparison) | Affects (outbound) | Displays the newly created version and the derived latest label |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Successful new-version creation requests one append-only trail entry with actor and timestamp (XBR-05); failures request none |

## Analytics and Success Signals

- **version_uploaded** (deliverable reference, new round_number, file size, time from selection to completion) -- supports success-metrics.md: "Large File Upload Success at Scale" (the metric's own rationale names version accumulation as the main driver of year-one storage volume, tying this automation's completions directly to that metric's target)
- **upload_resumed** (pause duration, bytes remaining at pause) -- supports success-metrics.md: "Large File Upload Success at Scale"
- **upload_failed** (failure reason: size_ceiling / empty_file / storage_allowance / server_rejection / deliverable_removed / deliverable_not_eligible / capability_unavailable / not_authorized, file size) -- supports success-metrics.md: "Large File Upload Success at Scale"

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (file selected, connectivity restored, manual retry) | 3 |
| Outcome Paths | 12 | 12 |
| Business Rules | 9 | 9 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Version Numbering, Immutability & Retention Rules

## Overview

**Name:** Version Numbering, Immutability & Retention Rules
**ID:** FEAT-17.SPEC-004
**Type:** Logic/Rule
**Purpose:** Governs sequential round numbering, immutability once uploaded, the uncapped-but-storage-bounded version count, and the "latest version" derivation for every Deliverable Version.
**Parent Feature:** FEAT-17 -- Deliverable Version History
**Governed Entity:** Deliverable Version

## Scope and Non-Goals

**In Scope:**
- Round_number sequencing: assignment, ordering, and gaplessness
- Immutability: what can never be changed on an existing version, and by whom
- Version-count and retention rules: the uncapped-but-storage-bounded ceiling, and how long versions are kept
- The `is_latest` derivation formula (highest round_number for the deliverable; derived, never stored)
- Authorization outcomes for actions that would violate immutability (edit, delete, manual "latest" override), for every role

**Non-Goals:**
- Who may view or create a version in the first place -- governed by FEAT-17.SPEC-005 (Version Access & Comment-Anchoring Rules); this spec addresses only the numbering, immutability, and retention constraints on the entity once a create is already authorized
- The comment-anchoring rule (a comment stays attached to the version it was posted on) -- governed by FEAT-17.SPEC-005, since it concerns the Comment entity's relationship to a version, not the version's own numbering or immutability
- The mechanics of the re-upload transfer itself (progress, pause/resume, failure handling) -- owned by FEAT-17.SPEC-003 (Version Creation & Preservation), which applies the rules defined here

## Governed Entity

**Entity:** Deliverable Version
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| round_number | number | Sequential position of this version among the deliverable's rounds, starting at 1 |
| file | file reference | The uploaded file for this specific round |
| uploaded_at | date/time | Timestamp this round's transfer completed |
| is_latest | derived (boolean) | True only for the version with the highest round_number for its deliverable |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-003 | Resumable Upload Handling | Creates round_number = 1 on the deliverable's first upload completion; establishes the sequence this spec continues |
| FEAT-17.SPEC-003 | Version Creation & Preservation | Assigns round_number = previous highest + 1 on every subsequent completion (create-only); never writes to any field of any prior version, and stores no is_latest value -- the new round becomes latest purely by holding the highest round_number |
| FEAT-17.SPEC-001 | New Version Upload | Displays the round_number just added ("Round {N} added") after the transfer completes, reflecting this spec's sequencing formula |
| FEAT-17.SPEC-002 | Version Browser & Comparison | Displays every round ordered by round_number and labels the is_latest round, per this spec's derivation; renders no edit or delete control for any version |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| round_number | Must equal the deliverable's current highest round_number plus one; never user-entered | Always | On version creation (FEAT-17.SPEC-003) | N/A -- system-assigned, never presented to any user as an editable field | Yes |
| file | Required; must be a successfully and fully transferred file (no partial-transfer commit); subject to the reused per-file size ceiling (platform parameter: `deliverable-file-size-ceiling`, per FEAT-06.SPEC-005) | Always | On version creation (FEAT-17.SPEC-003, before commit) | "This file is larger than the size limit for deliverables. Compress it or share it by link instead." (size ceiling); "This file appears to be empty." (zero bytes) | Yes |
| uploaded_at | Required; system-assigned to the transfer's completion time; never user-entered | Always | On version creation | N/A -- system-assigned | Yes |
| is_latest | No validation beyond data type -- purely derived (evaluated from round_number on read, never stored, never user-entered or user-editable) | Always | Evaluated whenever a version list is read | N/A -- not a user-facing input | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Round sequence continuity | round_number (across all versions of one deliverable) | For a given deliverable, the set of round_numbers is exactly {1, 2, ..., N} with no gaps and no duplicates, where N is the deliverable's current version count | N/A -- enforced structurally by FEAT-17.SPEC-003's assignment logic; no user-facing error exists because no user ever supplies this value |
| Latest-uniqueness | is_latest (across all versions of one deliverable) | Exactly one version per deliverable has is_latest = true at any moment: the one with the highest round_number | N/A -- follows structurally from derivation: the version with the highest round_number is the latest, so committing a new highest round_number changes which version qualifies without any write to another version; never presented as an editable state |

## Authorization Rules

Who may create, edit, delete, or otherwise change a Deliverable Version's numbering, content, or latest status. Who may view or create a version at all is governed by FEAT-17.SPEC-005; this spec's authorization table addresses only the actions that immutability and retention constrain.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create the next Deliverable Version | Nadia | Only via a fully completed re-upload transfer (FEAT-17.SPEC-003); who else may trigger this at all is governed by FEAT-17.SPEC-005 | No create control is rendered for Owen, Priya, or Dana anywhere in the product (FEAT-17.SPEC-005) |
| Edit any field of an existing Deliverable Version (round_number, file, uploaded_at) | No one | Never -- immutable once uploaded, regardless of role | No edit control exists anywhere in the product for any role; a version's file and metadata are permanently fixed the instant it is created |
| Delete or purge an individual Deliverable Version while the account is active | No one | Never -- "never deleted in-product; removed only by FEAT-24" (dependency map) | No delete or purge control exists anywhere in the product; the only lifecycle paths are superseding with a new round (FEAT-17.SPEC-001/003) or full account deletion (FEAT-24), executed at the storage layer by FEAT-16.SPEC-006 |
| Manually mark an older round as "latest" / current | No one | Never -- is_latest is purely derived from round_number with no override mechanism described anywhere in Stage 2 | No override control exists; the only way to make an older round's content current again is to upload it again as a brand-new round through FEAT-17.SPEC-001/003 |
| View a version's round_number, uploaded_at, and file | Governed by FEAT-17.SPEC-005 | See FEAT-17.SPEC-005 (Version Access & Comment-Anchoring Rules) | See FEAT-17.SPEC-005 |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| round_number | 1 for a deliverable's first version (FEAT-06.SPEC-003); for every subsequent version, the deliverable's current highest round_number + 1 | On create only, computed at the moment a transfer commits | No |
| uploaded_at | Current date/time at the moment the transfer completes fully | On create only | No |
| is_latest | True for the version whose round_number is the highest among all versions of its deliverable; false for every other version of that deliverable | Evaluated on every read; it changes for a deliverable the instant a new version with a higher round_number commits, with no write to any version | No (no override mechanism exists anywhere in Stage 2) |

## Business Rules

- XBR-13: each re-upload creates a new immutable version; comments stay attached to the version they were made on (FEAT-17.SPEC-005); all versions count against the freelancer's storage allowance.
- XBR-14: no fixed cap exists on version count per deliverable; the only limit is the freelancer's aggregate storage allowance (platform parameter: `free-tier-storage-allowance` for Free, platform parameter: `paid-tier-storage-allowance` for Paid), enforced by FEAT-16.SPEC-004 at the moment a new transfer would begin -- not by this spec, which defines only that no version-count ceiling of its own exists.
- Retention: every Deliverable Version is retained for the life of the account with no automatic purge while the account is active (ASMP-22, dependency map); it is removed only as part of account deletion, executed at the storage layer by FEAT-16.SPEC-006 (FEAT-24).
- A version's `is_latest` state is never itself a stored field a person sets -- it is evaluated as a pure function of round_number (highest round_number for the deliverable) each time versions are read, and no write to any existing version is ever involved when a new version is created (dependency map: "Derived: 'latest version' is the most recent upload").
- Immutability holds by omission: the product defines no edit path for any field of an existing Deliverable Version anywhere in Stage 2, so this spec's Authorization Rules table records "never" rather than describing a blocked-but-existing control.

## Edge Cases

- **Two near-simultaneous re-upload transfers for the same deliverable both complete** -- No round_number collision occurs: FEAT-17.SPEC-003 re-reads the current highest round_number at the moment each transfer commits, so whichever commits first is assigned the next number and the second, committing moments later, is assigned the number after that. Both are preserved; the sequence stays gapless.
- **A re-upload transfer fails partway through** -- No Deliverable Version row is ever created for that attempt; round_number sequencing is untouched -- no round is reserved, skipped, or left as a placeholder before a transfer fully completes.
- **A deliverable has exactly one version** -- Its round_number is 1 and its is_latest is trivially true; the "Latest" label carries no special meaning until a second round exists (surfaced by FEAT-17.SPEC-002 hiding the selector entirely in this case).
- **A deliverable accumulates a very large number of versions (hundreds of rounds over a long-running project)** -- No rule in this spec blocks further creation; the only possible blocker is the freelancer's storage allowance being exhausted (FEAT-16.SPEC-004), which is a separate feature's rule, not a version-count ceiling of this spec's own.
- **An attempt is made to reorder or renumber existing rounds (e.g., after discovering an upload was made in error)** -- Not applicable: the product defines no reordering or renumbering capability anywhere in Stage 2; the only corrective path is uploading a further new round, which always takes the next sequential number regardless of the reason for the upload.
- **Account deletion is requested while multiple versions exist** -- Every Deliverable Version and its stored bytes is removed as part of FEAT-24's account deletion, executed at the storage layer by FEAT-16.SPEC-006; this spec's retention rule ends exactly there, with no partial or selective retention of some rounds over others.

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Version Access & Comment-Anchoring Rules

## Overview

**Name:** Version Access & Comment-Anchoring Rules
**ID:** FEAT-17.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs who may view, open, download, or create a Deliverable Version per the Access Matrix, and the rule that a comment stays attached to the version it was posted on even after newer rounds exist.
**Parent Feature:** FEAT-17 -- Deliverable Version History
**Governed Entity:** Deliverable Version

## Scope and Non-Goals

**In Scope:**
- Per-role authorization for viewing, opening, downloading, and creating a Deliverable Version
- Client isolation for version access (a version never crosses client-company boundaries)
- Dana's read-only, no-download constraint inside a support session
- The comment-anchoring rule: a Comment's target stays fixed to the specific version it was posted on, regardless of later uploads
- The authority both this feature's own screens and FEAT-07's comment screens reference for anchoring, so neither re-derives it independently

**Non-Goals:**
- Round numbering, immutability of a version's own fields, and the version-count/retention ceiling -- governed by FEAT-17.SPEC-004 (Version Numbering, Immutability & Retention Rules)
- Creating, editing, or retracting a comment -- owned entirely by FEAT-07 (Deliverable Review & Feedback); this spec only defines what a comment's target reference means once it exists, not how the comment itself is written or removed
- Authorization for the deliverable itself (as opposed to its versions) -- governed by FEAT-06's own access rules; this spec's authority begins at the version level, consistent with Deliverable Version's Access field ("Follows Deliverable Upload & Sharing's Access field")

## Governed Entity

**Entity:** Deliverable Version
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| round_number | number | No validation beyond data type -- see FEAT-17.SPEC-004 for sequencing and derivation rules; this spec governs only who may see it |
| file | file reference | No validation beyond data type -- see FEAT-17.SPEC-004; this spec governs only who may open or download it |
| uploaded_at | date/time | No validation beyond data type -- see FEAT-17.SPEC-004; this spec governs only who may see it |
| is_latest | derived (boolean) | No validation beyond data type -- see FEAT-17.SPEC-004 for the derivation formula; this spec governs only who may see it |

**Referenced entity (for the anchoring rule):** Comment -- specifically its `target` field, which the dependency map defines as "a deliverable version, a milestone as a whole, or a proposal (change request)." Comment creation, editing, and retraction remain owned by FEAT-07; this spec is the rule authority for what `target` means once it points at a Deliverable Version.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-17.SPEC-001 | New Version Upload | Authorization checked on screen entry (only Nadia can reach this screen at all) |
| FEAT-17.SPEC-002 | Version Browser & Comparison | Authorization checked on screen entry (per-role view/download scoping) and on every round-open and download attempt |
| FEAT-17.SPEC-003 | Version Creation & Preservation | Confirms the triggering session is Nadia's before creating a version (defense-in-depth behind FEAT-17.SPEC-001's screen-entry check) |
| FEAT-07 (Deliverable Review & Feedback, comment screens) | Comment thread per deliverable/version | Reads the anchoring rule when displaying which round a comment belongs to, and when a comment is posted while an older round is open |
| FEAT-31 (Operator Support Access) | Read-only support session | Confirms Dana's session renders view-only, no-download access to version data, per this spec's authorization table |

## Field Validation Rules

No field-level validation rules are defined by this spec -- validation, sequencing, and derivation for every field of Deliverable Version are governed entirely by FEAT-17.SPEC-004. This spec governs authorization (who may see or act on those fields) and the comment-anchoring rule, not the fields' own content.

## Cross-Field Rules

No cross-field validation rules are defined by this spec. The one cross-entity rule this spec owns -- comment anchoring, between Comment.target and Deliverable Version -- is documented under Business Rules below, since it governs behavior over time rather than the validity of a single record's fields.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create the next Deliverable Version (trigger a re-upload) | Nadia (Freelancer) | Always, for any deliverable she owns whose kind is an uploaded file | -- |
| Create the next Deliverable Version | Owen (Client Primary Contact) | Never | "Upload New Version" control is never rendered anywhere in the client portal (FEAT-17.SPEC-001's Access and Visibility table) |
| Create the next Deliverable Version | Priya (Client Reviewer Contact) | Never | Same as Owen -- no upload control exists in the client portal |
| Create the next Deliverable Version | Dana (Support Operator) | Never | No upload entry point is rendered inside a support session, consistent with support sessions never editing anything (ASMP-18) |
| View the version list and open a round's metadata (round_number, uploaded_at) | Nadia (Freelancer) | Full -- every version of every deliverable across her own account | -- |
| View the version list and open a round's metadata | Owen (Client Primary Contact) | Own-only -- only versions of deliverables belonging to his own client company | Versions of another client's deliverables never appear or resolve; a direct link is refused with "This page isn't part of your portal." (XBR-09) |
| View the version list and open a round's metadata | Priya (Client Reviewer Contact) | Own-only -- only versions of deliverables belonging to her own client company | Same as Owen |
| View the version list and open a round's metadata | Dana (Support Operator) | View only, inside a logged, read-only support session (FEAT-31), for the one freelancer account currently under support | Outside an active, logged support session, no version data for any account is reachable |
| Open or download a round's underlying file | Nadia (Freelancer) | Full -- any version of any of her own deliverables | -- |
| Open or download a round's underlying file | Owen (Client Primary Contact) | Own-only -- any version of his own client company's deliverables | Same client-isolation denial as above for versions outside his own company |
| Open or download a round's underlying file | Priya (Client Reviewer Contact) | Own-only -- any version of her own client company's deliverables | Same client-isolation denial as above |
| Open or download a round's underlying file | Dana (Support Operator) | Never | No download control is rendered for any round inside a support session; a direct attempt is refused with "Downloads are not available in a support session." (ASMP-18, ASMP-23) |

## Defaults and Derivations

N/A -- this spec governs authorization and comment anchoring, not field defaults or derivations. Round_number's assignment, uploaded_at's timestamping, and is_latest's derivation formula are all owned by FEAT-17.SPEC-004.

## Business Rules

- Comment anchoring (XBR-13): once a Comment's `target` field is set to a specific Deliverable Version at the moment of posting (by FEAT-07), that reference is permanent -- it is never migrated, re-pointed, or reinterpreted when a newer round is uploaded to the same deliverable. A comment posted on Round 2 stays a Round 2 comment forever, even once Round 5 exists.
- This anchoring holds regardless of which round is currently `is_latest`: FEAT-17.SPEC-002's comment count indicator for a given round only ever counts comments whose `target` is that exact round, never comments from any other round of the same deliverable.
- FEAT-07 owns Comment creation, editing, and retraction; this spec is the sole rule authority for what the `target` reference means and how long it holds -- FEAT-07's own specs reference this spec by ID rather than re-deriving the anchoring behavior.
- XBR-09: client isolation applies to every version-access action in this spec exactly as it applies elsewhere in the portal -- a contact reaches only their own company's deliverable versions, never another company's, under any circumstance.
- ASMP-18 / ASMP-23: Dana's support session is read-only everywhere, including here -- she may list and browse version metadata but never open or download the underlying file of any round, mirroring FEAT-06 and FEAT-16's identical Support Access boundary.
- Access to a Deliverable Version's own data follows Deliverable Upload & Sharing's Access field (feature-overview.md: "Follows Deliverable Upload & Sharing's Access field") -- this spec's per-role rows above are the version-specific restatement of that same boundary, never a divergent one.

## Edge Cases

- **A client contact posts a comment while viewing an older round, then a newer round is uploaded moments later** -- The comment's anchoring is unaffected: it was already fixed to the older round's Deliverable Version the instant it was posted, before the new round existed. The new round has zero comments of its own until someone posts directly on it.
- **Priya comments on Round 2, then Round 2 is (hypothetically) the only round anyone ever views again because Round 3 supersedes it as latest** -- Her comment remains fully visible and intact on Round 2 forever; comment visibility is never tied to a round's `is_latest` status, only to which round the comment's `target` names.
- **Dana's support session is active while Owen is also viewing the same deliverable's versions** -- Both view independently and concurrently with no interaction; Dana's session never affects what Owen sees, and Owen's viewing never appears in Dana's session.
- **A client contact's role changes from Reviewer to Primary (or vice versa) partway through a project with existing version-anchored comments** -- Already-posted comments and their anchoring are unaffected by a later role change; the changed role governs only that contact's future actions (view/comment/approve entitlements), never past comments' anchoring.
- **A contact who is a client contact for two different freelancers views version histories in both portals** -- Each portal's version access is scoped entirely to that freelancer's own clients; no version, comment, or anchoring reference ever crosses between the two freelancers' accounts (XBR-09).
- **Dana attempts to construct a direct link to a version's file, bypassing the rendered support-session UI** -- The download is refused with "Downloads are not available in a support session." regardless of how the request is made, since the no-download rule is enforced at the authorization boundary, not only in the rendered controls.

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 (all N/A -- referred to FEAT-17.SPEC-004) | 4 |
| Cross-Field Rules | 0 (none owned by this spec) | 0 |
| Authorization Rules | 12 | 12 |
| Defaults/Derivations | 0 (N/A -- referred to FEAT-17.SPEC-004) | 0 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
