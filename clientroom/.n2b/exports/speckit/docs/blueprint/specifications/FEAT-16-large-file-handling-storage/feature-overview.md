---
document_type: feature-overview
feature_number: FEAT-16
feature_name: Large File Handling & Storage
feature_slug: large-file-handling-storage
priority_tier: Core
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 7
screen_count: 1
automation_count: 4
logic_rule_count: 1
integration_count: 1
notification_count: 0
---

# Feature Breakdown Brief: Large File Handling & Storage

## Summary

**Feature:** Large File Handling & Storage
**ID:** FEAT-16
**Description:** The product reliably accepts and delivers large files — tens of MB, sometimes over 1 GB for video — with resumable uploads, staying within the freelancer's stated infrastructure budget.
**Priority:** Core
**Phase:** MVP
**Type:** Platform
**Rationale:** BRIEF.md, Scale & Non-Functional Expectations: "large files are the norm... typically tens of MB and sometimes over 1 GB for video," alongside a roughly $100/month infrastructure Constraint — this feature is the product's answer to that tension. MVP phase: Deliverable Upload & Sharing (FEAT-06) cannot honestly work without it.

**Key Capabilities:**
- Resumable large-file upload — uploads survive an interrupted connection
- Cost-conscious storage — file handling stays within the stated infrastructure budget
- Reliable delivery — clients stream or download without installing anything

This is a Platform feature: it is the infrastructure behind Deliverable Upload & Sharing (FEAT-06) and Deliverable Version History (FEAT-17), not a standalone screen a user navigates to for its own sake (Stage 2 States field). Its own user-facing surface is limited to the storage-usage visibility the Validation & Limits field requires; everything else is automation, rule enforcement, and the external storage capability it owns on the dependency map.

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-16.SPEC-001 | Storage Usage Summary | Screen | Nadia, Dana | Nadia (and Dana, read-only, in a support session) sees total stored bytes against the freelancer's storage allowance, with a path into the plan view when near the limit |
| FEAT-16.SPEC-002 | Resumable Upload Transfer | Automation | Nadia | Moves an uploading file into storage in resumable, progress-tracked chunks, persisting resume state across dropped connections and retrying automatically before any manual retry is offered |
| FEAT-16.SPEC-003 | Reliable File Delivery | Automation | Nadia, Owen, Priya | Serves a stored deliverable file for in-browser streaming or direct download with no client-side install, retrying a failed transfer automatically before a manual retry is offered |
| FEAT-16.SPEC-004 | Storage Limit & Size Ceiling Rules | Logic/Rule | Nadia | Enforces the per-file size ceiling (accommodating video files somewhat over 1 GB) and the per-freelancer storage allowance, and triggers the pre-limit warning threshold |
| FEAT-16.SPEC-005 | Storage Usage Aggregation | Automation | Nadia | Recalculates the freelancer's total stored bytes whenever a file finishes uploading, a deliverable is removed, or stored bytes are purged, feeding the usage summary and the limit rule |
| FEAT-16.SPEC-006 | Stored File Purge on Account Deletion | Automation | Nadia | Permanently purges every stored file and version's bytes from the storage capability when the freelancer's account is deleted |
| FEAT-16.SPEC-007 | Large-File Storage & Delivery Capability | Integration | All | Owns the product's contract with the external large-file storage and delivery capability (ASMP-30): resumable ingestion, byte-range streaming/download, and reported transfer status, within the stated infrastructure budget |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Resumable large-file upload | FEAT-16.SPEC-002 | The transfer automation chunks the upload, persists resume state, and auto-resumes across a dropped connection | Phase 2 (Explicit) |
| Cost-conscious storage | FEAT-16.SPEC-004, FEAT-16.SPEC-005 | The rule spec enforces the per-file ceiling and per-freelancer allowance; the aggregation automation keeps the usage figure the rule checks against current | Phase 2 (Explicit) |
| Reliable delivery | FEAT-16.SPEC-003 | The delivery automation streams or downloads a stored file directly in the browser, with automatic retry before manual retry | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-16.SPEC-001 | Storage Usage Summary | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field states "the freelancer can always see her storage use against her allowance" -- a data-visibility rule with no interaction surface in the three Key Capabilities, requiring its own screen |
| FEAT-16.SPEC-005 | Storage Usage Aggregation | Phase 4 (Trigger-Response Analysis) | The Data Notes field names a derived field ("storage usage totals per freelancer, used for plan-limit warnings") with no owning spec until this automation is added; it is the trigger-response that keeps SPEC-001 and SPEC-004 correct as uploads, removals, and purges occur |
| FEAT-16.SPEC-006 | Stored File Purge on Account Deletion | Phase 3 (Entity-Lifecycle Analysis) | The CRUD matrix surfaced a gap: the dependency map records Deliverable Version as "never deleted in-product; removed only by FEAT-24," but someone must execute the actual byte-level purge at the storage layer this feature owns -- an unassigned Delete/Archive cell otherwise |
| FEAT-16.SPEC-007 | Large-File Storage & Delivery Capability | Phase 4 (External Dependencies lens) | The Dependencies section of assumptions-constraints.md names ASMP-30 (file storage and delivery for large files) as a category-level external capability this feature relies on; the dependency map's External Touchpoints row names FEAT-16 as the expected owner of its Integration spec |

## Entity-Lifecycle Coverage Matrix

**Entity: Deliverable**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Owned by FEAT-06 (Deliverable Upload screen creates the record) -- this feature never originates a Deliverable | Not a gap: Deliverable is a FEAT-06-owned entity that this feature only updates and reads for storage purposes |
| Read (single) | FEAT-16.SPEC-003 | Resolves a deliverable's stored-file reference in order to stream or serve it, invoked from FEAT-06/FEAT-07 screens | This feature never renders a deliverable's own detail view (owned by FEAT-06/FEAT-07/FEAT-08/FEAT-17/FEAT-31) |
| Read (list) | N/A | Deliverable listing is owned entirely by FEAT-06.SPEC-002 and the client-facing view owned by FEAT-07 | Out of this feature's scope |
| Update | FEAT-16.SPEC-002 | Writes the storage-layer fields captured in Data Notes -- file bytes, size, upload/download timestamps -- onto the Deliverable record as the resumable transfer progresses and completes | FEAT-06.SPEC-003 separately owns the deliverable's own status field (Uploading -> Active); this feature supplies the storage-completion signal that transition consumes, but does not set the status itself |
| Delete/Archive | N/A | The soft removal of a Deliverable (status -> Removed) is owned by FEAT-06.SPEC-002 governed by FEAT-06.SPEC-005; this feature has no removal action of its own on the Deliverable record | This feature's role in deletion is limited to purging the underlying bytes of its Deliverable Versions on account deletion (see below, SPEC-006) |
| State Transition | N/A | Deliverable's status vocabulary (Uploading, Active, Superseded, Removed) belongs to FEAT-06/FEAT-17; this feature supplies the completion event that drives the Active transition but does not own any state itself | -- |

**Entity: Deliverable Version**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-16.SPEC-002 | Performs the actual storage write of the uploaded bytes and returns the stored-file reference and final size that FEAT-06 (round 1) or FEAT-17 (round 2+) use to create the Deliverable Version record | The version metadata record itself is created by FEAT-06/FEAT-17; this feature creates the underlying stored artifact those records point to (dependency map: "Created by ... FEAT-16 (stored file)") |
| Read (single) | FEAT-16.SPEC-003 | Resolves a specific version's stored-file reference to stream or serve it on request | -- |
| Read (list) | N/A | Version-history browsing belongs to FEAT-17 (Deliverable Version History), not this feature | -- |
| Update | N/A | Deliverable Version is immutable once uploaded (dependency map); this feature never updates a version's stored bytes after the initial write commits | Consistent with the entity's stated immutability -- an intentional design decision, not a gap |
| Delete/Archive | FEAT-16.SPEC-006 | Hard delete at the storage layer: when FEAT-24 deletes a freelancer's account, this feature permanently purges every one of that account's stored version files from the storage capability; no restore path exists once account deletion completes (deletion is destructive by design at that boundary); no cascade beyond the deleted account's own records; while the account remains active, versions are retained for the life of the account with no automatic purge (ASMP-22) | This row exists because "never deleted in-product" (dependency map) governs the in-product record, not the storage bytes -- this feature owns the storage capability and must execute the actual purge when FEAT-24 orchestrates account deletion |
| State Transition | N/A | `is_latest` is a derived field, not a stored state; the entity carries no status field at the storage layer either | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Subscription Plan | FEAT-16.SPEC-001, FEAT-16.SPEC-004 | The storage allowance amount is determined by the freelancer's plan tier; the usage summary displays it and the limit rule enforces against it |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia begins uploading a file (via FEAT-06 or FEAT-17's upload flow) | Transfer the file into storage in resumable, progress-tracked chunks; emit `large_upload_started` | Standalone Automation | FEAT-16.SPEC-002 |
| An in-progress transfer's connection drops | Pause the transfer, persist resume state, and auto-resume from where it left off once connectivity returns, without restarting from zero; emit `large_upload_resumed` | Standalone Automation | FEAT-16.SPEC-002 |
| A transfer chunk fails for a reason other than dropped connectivity | Retry automatically before any manual retry option is surfaced (States: Error field) | Standalone Automation | FEAT-16.SPEC-002 |
| A file is selected for upload | Check the file's size against the per-file ceiling before the transfer begins | Standalone Logic/Rule | FEAT-16.SPEC-004 |
| An upload transfer completes fully | Commit the stored file, return the stored-file reference and final size to the calling feature (FEAT-06/FEAT-17), and update the Deliverable's storage-state fields | Standalone Automation | FEAT-16.SPEC-002 |
| A client contact (Owen or Priya) opens a deliverable to view or download it | Stream or serve the stored bytes directly in the browser, no install required | Standalone Automation | FEAT-16.SPEC-003 |
| A streaming or download transfer fails | Retry automatically before any manual retry option is surfaced (States: Error field) | Standalone Automation | FEAT-16.SPEC-003 |
| An upload completes, a deliverable is removed, or stored bytes are purged | Recalculate the freelancer's total stored-bytes figure | Standalone Automation | FEAT-16.SPEC-005 |
| The freelancer's recalculated total approaches the storage allowance | Evaluate the warning threshold and surface the warning inline (no delivery-channel notification -- this feature's Communications field is N/A); emit `storage_limit_warning_shown` | Standalone Logic/Rule | FEAT-16.SPEC-004 |
| Nadia opens the Storage Usage Summary | Display total stored bytes against her allowance, with warning styling if near the limit and a link into the plan view | Inline in triggering screen | FEAT-16.SPEC-001 |
| Nadia's account is deleted | Permanently purge every stored file and version's bytes from the storage capability | Standalone Automation | FEAT-16.SPEC-006 |
| The storage capability reports a transfer or capacity problem (inbound event) | Surface the failure through the relevant automation's retry-then-manual-retry handling (SPEC-002 for uploads, SPEC-003 for delivery) | Standalone Integration (inbound event) | FEAT-16.SPEC-007 |
| A deliverable-ready notification is due | Only fires once the transfer this feature manages has fully completed, never for a partial file (XBR-12) | Cross-feature | FEAT-06 responsibility (FEAT-06.SPEC-006 owns the notification; this feature's completion signal gates it) |
| A deliverable is uploaded, its stored bytes purged, or an upload fails | An append-only activity-trail entry is written with actor and timestamp | Cross-feature | FEAT-13 responsibility |
| Dana opens a read-only support session | Sees the Storage Usage Summary's figures but never downloads a file (XBR-29) | Cross-feature | FEAT-31 responsibility (governs the session; this feature's screen honors the read-only, no-download constraint) |

## Shared Context

**Shared Entities:**
- Deliverable -- updated (storage-state fields only) by SPEC-002; read by SPEC-003 to resolve a stored-file reference. Fields this feature writes: file bytes, size, upload/download timestamps (Data Notes). Its status, link, and milestone fields remain owned by FEAT-06/FEAT-17.
- Deliverable Version -- created (stored file only) and, on account deletion, purged by SPEC-002/SPEC-006; read by SPEC-003. Fields this feature touches: file (the stored bytes), the storage reference and size returned to the calling feature. round_number, uploaded_at, and is_latest remain owned by FEAT-06/FEAT-17.

**Shared UI Patterns:**
- Progress and resume state -- SPEC-002 produces the same progress vocabulary (queued, transferring with percentage, paused/resuming, complete, failed-retrying, failed) that FEAT-06.SPEC-001's upload screen displays; Spec Writers for both should describe this vocabulary identically rather than re-deriving it.
- Storage warning styling -- SPEC-001's near-limit warning and the inline warning surfaced wherever an upload is attempted near the allowance (within FEAT-06's upload screen) should share one visual treatment and one threshold value, both sourced from SPEC-004.

**Shared Validation:**
- SPEC-004 defines the only two limits this feature enforces -- the per-file size ceiling and the per-freelancer storage allowance. SPEC-001 (display), SPEC-002 (pre-transfer check), and SPEC-005 (the figure SPEC-004 checks against) all reference SPEC-004 rather than duplicating the limit values.

**Flagged discrepancy (not resolved by this Brief):** The Communications field for this feature is explicitly N/A -- storage-limit awareness stays an inline, in-product signal (`storage_limit_warning_shown`) rather than a delivery-channel notification, which is why this Brief carries `notification_count: 0` even though a warning exists. This is a deliberate elaboration of the Stage 2 Communications field, not an omission, and is noted here so downstream readers do not mistake the zero count for a coverage gap.

## Internal Dependency Map

```
SPEC-002 (Resumable Upload Transfer) -> [validates size against] -> SPEC-004 (Storage Limit & Size Ceiling Rules)
SPEC-002 (Resumable Upload Transfer) -> [transfers and stores bytes via] -> SPEC-007 (Large-File Storage & Delivery Capability)
SPEC-002 (Resumable Upload Transfer) -> [upload completes] -> SPEC-005 (Storage Usage Aggregation)
SPEC-005 (Storage Usage Aggregation) -> [updated total feeds] -> SPEC-004 (Storage Limit & Size Ceiling Rules) -> [threshold breached] -> SPEC-001 (Storage Usage Summary, warning shown)
SPEC-001 (Storage Usage Summary) -> [Nadia reviews usage near the limit] -> FEAT-23 (Subscription & Account plan view)
SPEC-003 (Reliable File Delivery) -> [resolves and serves stored bytes via] -> SPEC-007 (Large-File Storage & Delivery Capability)
SPEC-006 (Stored File Purge on Account Deletion) -> [purges bytes via] -> SPEC-007 (Large-File Storage & Delivery Capability)
SPEC-006 (Stored File Purge on Account Deletion) -> [triggered by] -> FEAT-24 (Account Deletion)
SPEC-005 (Storage Usage Aggregation) -> [recalculates after] -> SPEC-006 (Stored File Purge on Account Deletion)
```

**Default Entry:** FEAT-16.SPEC-001 (Storage Usage Summary) -- as a Platform feature, this feature has no primary navigable screen of its own; SPEC-001 is the only screen a user reaches directly (from Nadia's account/plan area), while SPEC-002, SPEC-003, SPEC-005, SPEC-006, and SPEC-007 run beneath FEAT-06 and FEAT-17's own screens.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-16.SPEC-002 | Inbound | FEAT-06 (Deliverable Upload & Sharing) | FEAT-06.SPEC-003 invokes this feature's resumable transfer to move and store the uploaded bytes, then reads back the storage-completion signal that drives its own Active transition | Nadia uploads a file on a milestone |
| FEAT-16.SPEC-002 | Inbound | FEAT-17 (Deliverable Version History) | A re-upload's bytes are transferred and stored through this feature the same way a first upload is | Nadia uploads a new version of an existing deliverable |
| FEAT-16.SPEC-002 | Outbound | FEAT-06 (Deliverable Upload & Sharing) | Completion of this feature's transfer is the event FEAT-06.SPEC-006's deliverable-ready notification waits for (XBR-12) -- never fired for a partial file | An upload transfer completes fully |
| FEAT-16.SPEC-003 | Outbound | FEAT-07 (Deliverable Review & Feedback) | Priya's or Owen's stream/open action on the review screen is served by this feature's delivery automation | A client contact opens a deliverable to view |
| FEAT-16.SPEC-003 | Outbound | FEAT-08 (Milestone Approval) | Owen's review of the deliverable before approving is served by this feature's delivery automation | Owen opens the deliverable during milestone review |
| FEAT-16.SPEC-001 | Outbound | FEAT-23 (Subscription & Account Data / Plan Management) | The storage-limit warning links Nadia into the plan view to review or change her plan | Nadia is near her storage allowance |
| FEAT-16.SPEC-004 | Inbound | FEAT-23 (Subscription & Account Data / Plan Management) | The per-freelancer storage allowance amount is read from the freelancer's current plan tier | A file is uploaded or the usage summary is displayed |
| FEAT-16.SPEC-001 | Inbound | FEAT-31 (Operator Support Access) | Dana's read-only, logged support session displays this feature's usage figures without any file-download capability (XBR-29) | Dana opens a support session on the freelancer's account |
| FEAT-16.SPEC-006 | Inbound | FEAT-24 (Account Deletion) | Account deletion orchestrates the purge; this feature executes the actual byte-level deletion at the storage layer | Nadia's account is deleted |
| FEAT-16.SPEC-002 / FEAT-16.SPEC-005 / FEAT-16.SPEC-006 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Uploads, storage-limit warnings, and purges each write an append-only trail entry with actor and timestamp | A file uploads, a warning is shown, or bytes are purged |

## Non-Functional Notes

**Data volumes / growth:** Deliverables typically run tens of MB and sometimes over 1 GB for video, retained with full version history for the life of the account (ASMP-22); this feature's storage capability must keep accepting and serving files at this size and growth rate as a freelancer's total stored bytes accumulate across her account's lifetime, which is exactly what the Large File Upload Success at Scale success metric measures (no more than a brief, visible delay as stored volume approaches typical year-one levels).

**Responsiveness:** Upload and delivery must show real, visible progress with an estimated completion for large files, never an indefinite spinner (feature's States field, ASMP-27); a dropped connection pauses and resumes automatically rather than forcing a restart (ASMP-27's offline posture, feature's Offline-degraded state); the Deliverable Upload Reliability success metric sets the bar concretely -- at least 98% of uploads, including files over 500 MB, complete without the user restarting from zero. This feature's own SPEC-001 screen is a low-traffic account-settings view and is not itself bound by the client-facing 2-second interactivity target (ASMP-21), which governs FEAT-07's review screen instead.

**Data sensitivity / privacy:** Stored files are client-confidential work product (design files, videos, documents) that may themselves contain personal data, strictly isolated per client (ASMP-23); this feature's storage capability never exposes one client's files to another, and Dana's read-only support access can list files but is never able to download the underlying bytes (ASMP-23, XBR-29).

**Compliance flags:** GDPR-class handling applies to any personal data embedded within a stored file's content (ASMP-23); no card, payment, or health data is ever stored by this feature -- that data, where it exists elsewhere in the product, is held by the subscription-billing capability, not here.

## Non-Goals

- **Copying or hosting files from Figma, Google Drive, or Dropbox** -- Excluded per scope-boundaries.md (SC-08) and the SC-22 exclusion notes on this feature: linked external assets are referenced by URL and checked for reachability by FEAT-06; this feature's storage capability only ever stores files the freelancer actually uploads, never a copy of a linked asset's content.
- **Native mobile upload or delivery apps** -- Excluded per scope-boundaries.md (SC-06): the product is a web app with no native apps; resumable upload and reliable delivery are delivered entirely through the mobile-and-desktop-browser experience, which is exactly what "clients stream or download without installing anything" (Key Capabilities) requires.
- **Scoped storage permissions for a freelancer-side team** -- Excluded per scope-boundaries.md (SC-01): the product is solo-freelancer only for v1, so this feature has exactly one account-level storage owner (Nadia); no bookkeeper- or contractor-style scoped storage permission exists.
- **Restoring purged files after account deletion** -- Intentional lifecycle decision surfaced by the CRUD matrix: FEAT-16.SPEC-006's purge is a one-way, destructive action executed only when FEAT-24 deletes the account itself, which is already a terminal, non-reversible action in the product's own definition; there is no separate restore path for storage bytes once that boundary is crossed.
- **Automatic purge of active, superseded, or removed deliverables' storage while the account remains active** -- Intentional lifecycle decision carried from ASMP-22 and mirrored from FEAT-06's own Non-Goals: stored bytes for every version are retained for the life of the account with no automatic purge, because a removed or superseded deliverable's file remains part of the project's evidentiary and version history.
- **A delivery-channel notification for storage-limit warnings** -- Excluded by this feature's own Communications field (product-features.md: "N/A -- this is infrastructure supporting FEAT-06's notifications, not a separate message source"); the warning stays an inline, in-product signal (SPEC-001, SPEC-004) rather than an email or push notification, which is why this Brief legitimately carries `notification_count: 0`.
