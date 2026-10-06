---
document_type: feature-overview
feature_number: FEAT-06
feature_name: Deliverable Upload & Sharing
feature_slug: deliverable-upload-sharing
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 6
screen_count: 2
automation_count: 2
logic_rule_count: 1
integration_count: 0
notification_count: 1
---

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
