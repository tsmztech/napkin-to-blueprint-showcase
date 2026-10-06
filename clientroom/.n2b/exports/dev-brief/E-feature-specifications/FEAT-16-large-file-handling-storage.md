# FEAT-16 — Large File Handling & Storage

This chapter covers Large File Handling & Storage, a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 7 specifications carrying 84 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-16.SPEC-001 | Storage Usage Summary | screen | 11 |
| FEAT-16.SPEC-002 | Resumable Upload Transfer | automation | 12 |
| FEAT-16.SPEC-003 | Reliable File Delivery | automation | 11 |
| FEAT-16.SPEC-004 | Storage Limit & Size Ceiling Rules | logic-rule | 14 |
| FEAT-16.SPEC-005 | Storage Usage Aggregation | automation | 10 |
| FEAT-16.SPEC-006 | Stored File Purge on Account Deletion | automation | 10 |
| FEAT-16.SPEC-007 | Large-File Storage & Delivery Capability | integration | 16 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Storage Usage Summary

## Overview

**Name:** Storage Usage Summary
**ID:** FEAT-16.SPEC-001
**Type:** Screen
**Purpose:** Nadia sees her total stored bytes against her freelancer storage allowance, with warning styling as she approaches the limit and a path into her plan view; Dana sees the same figures read-only during a logged support session.
**Parent Feature:** FEAT-16 -- Large File Handling & Storage

## Scope and Non-Goals

**In Scope:**
- Displaying the freelancer's current total stored bytes against her storage allowance
- Warning styling when usage approaches or reaches the allowance
- A link into the plan view when usage is near or at the limit
- Dana's read-only view of the same figures during a logged support session

**Non-Goals:**
- Calculating or recalculating the total stored-bytes figure -- owned by FEAT-16.SPEC-005 (Storage Usage Aggregation); this screen only displays the most recently aggregated total
- Determining the allowance amount, the warning threshold, or enforcing any limit -- owned by FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules); this screen only displays the values that spec defines
- Changing or purchasing a plan -- owned by FEAT-23 (Subscription Plan & Billing Management); this screen only links into FEAT-23.SPEC-001 (Plan & Billing Screen)
- Listing or previewing individual stored files -- excluded per this feature's own Summary: "its own user-facing surface is limited to the storage-usage visibility the Validation & Limits field requires"; per-deliverable browsing belongs to FEAT-06.SPEC-002 (Deliverable List & Management) and FEAT-17 (Deliverable Version History)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-21.SPEC-001 (Account Profile) | Nadia opens "Storage" from her account settings | None -- screen loads and fetches the current total and allowance itself |
| FEAT-23.SPEC-001 (Plan & Billing Screen) | Nadia opens the storage detail from a usage indicator on her plan view | None -- screen loads independently |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Dana, inside an open support session on a named freelancer's account, opens that freelancer's Storage screen | The freelancer account the session is scoped to (FEAT-31.SPEC-003); screen renders that account's figures read-only |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen: total stored bytes, allowance, warning styling, link into the plan view | Navigate to FEAT-23.SPEC-001 (Plan & Billing Screen) | -- |
| Owen (Client Primary Contact) | No | No | Storage usage is not part of any client-facing surface; Owen's Subscription & Account Data entitlement is None (Access Matrix) -- no navigation path in the client portal ever reaches this screen |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- Priya's Subscription & Account Data entitlement is None |
| Dana (Support Operator) | Full screen, read-only: total stored bytes, allowance, warning styling -- only while an active support session on that freelancer's account is open (FEAT-31.SPEC-003) | No actions -- the "Review your plan" link is not shown to Dana, since changing a plan is not part of a read-only session | Outside an active support session this screen is unreachable; there is no separate denied experience, since no navigation path to it exists without one |
| Unauthenticated | No | No | Redirected to sign-in; entered no data on this read-only screen to lose |
| Expired session | No | No | "Your session has expired. Sign in to continue." -- no in-progress data to preserve, since this screen accepts no input |

## Layout and Content

**Header:** Screen title "Storage" within the account settings / plan area, with a back control returning to the entry screen (Account Profile or Plan & Billing).

**Body:** A single stat panel, vertically centered in the content area:
- A storage meter (horizontal progress-style indicator) showing bytes used as a proportion of the allowance
- A numeric label beneath the meter: "{used amount} of {allowance amount} used" (e.g., "6.2 GB of 10 GB used")
- A warning banner, shown only in the Near Limit or At Capacity states, with the message defined by FEAT-16.SPEC-004's threshold rule and a "Review your plan" link (Nadia only -- not shown to Dana)

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Stat panel stacks vertically, full width; meter spans the full content width; warning banner, when present, appears directly below the meter, full width.
- **Medium size class and above:** Stat panel is capped at a consistent platform-wide card width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Storage meter | Screen load | Fetches the most recently aggregated total from FEAT-16.SPEC-005 and the current allowance and threshold from FEAT-16.SPEC-004 | Meter renders at the loaded proportion | Visual meter fill plus the numeric label |
| "Review your plan" link | Tap (Nadia only, Near Limit or At Capacity state) | Navigate to FEAT-23.SPEC-001 (Plan & Billing Screen) | Screen closes | Standard navigation transition |
| Screen re-opened | Nadia or Dana returns to this screen after navigating away | Re-fetches the latest aggregated total and allowance | Meter and label update to the current values | Meter reflects the freshest known total; the screen does not live-update while left open in the background |

### Accessibility Notes

- **Focus order:** Back control -> storage meter (announced with its numeric percentage) -> warning banner and "Review your plan" link (when present).
- **Dynamic announcements:** When the warning banner becomes visible on load, its message is announced to assistive technology.
- **Keyboard alternatives:** The meter is a display-only element; "Review your plan" and the back control are both reachable and actionable by keyboard, with no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton meter and label while the current total and allowance load | Screen first opens | Load completes (success or error) |
| Normal | Meter shows usage below the warning threshold; no warning banner | Load completes with usage below platform parameter: `storage-usage-warning-threshold-percent` of the allowance | Usage crosses the warning threshold on a later reload |
| Near Limit | Meter and label show the current usage; warning banner: "You're approaching your storage allowance. {used amount} of {allowance amount} used." with "Review your plan" (Nadia) | Load completes with usage at or above platform parameter: `storage-usage-warning-threshold-percent` but below the full allowance | Usage drops back below the threshold, or reaches full allowance, on a later reload |
| At Capacity | Meter shows a full or near-full fill; warning banner: "You've reached your storage allowance. New uploads won't complete until you free up space or upgrade your plan." with "Review your plan" (Nadia) | Load completes with usage at or effectively at the full allowance | Usage drops below the full allowance on a later reload (e.g., after a removal or purge) |
| Error | Error banner: "Couldn't load your storage usage. Try again." with a Retry control | The total or allowance fails to load | Retry succeeds and the screen re-enters Loading, then Normal/Near Limit/At Capacity |
| Offline/Degraded | Banner: "You're offline -- showing your last known storage usage." above the meter; the last successfully loaded figures remain displayed; "Review your plan" is disabled while offline | Connectivity is lost while this screen is open, or the screen is opened without connectivity and a cached figure exists | Connectivity returns and the screen re-fetches automatically |

## Validation Rules

Not applicable -- this screen accepts no user input; every displayed value is read-only, sourced from FEAT-16.SPEC-004 and FEAT-16.SPEC-005.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back control tap | FEAT-21.SPEC-001 (Account Profile) or FEAT-23.SPEC-001 (Plan & Billing Screen) -- whichever this screen was entered from | FEAT-21 or FEAT-23 |
| "Review your plan" tap (Nadia, Near Limit or At Capacity) | FEAT-23.SPEC-001 (Plan & Billing Screen) | FEAT-23 |

## Data Model

**Creates:** None.
**Reads:** The freelancer's current aggregated total stored bytes (maintained by FEAT-16.SPEC-005, derived from every non-purged Deliverable Version's stored file size). Subscription Plan -- tier, used by FEAT-16.SPEC-004 to resolve the applicable allowance and warning threshold.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The allowance amount, the warning threshold, and the At Capacity condition are all defined by FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules) -- this screen displays those values without redefining them.
- The displayed total is FEAT-16.SPEC-005's most recently completed aggregation; a transfer still in progress is not reflected until it completes and the aggregation recalculates (XBR-14).
- XBR-29: Dana's read-only support session never offers a file-download or plan-change action on this screen.

## Edge Cases

- **Total shown while an upload is mid-transfer** -- The meter reflects only FEAT-16.SPEC-005's last completed aggregation, not bytes from an in-progress transfer; the figure can briefly lag behind a transfer that has not yet finished. This is a display characteristic, not a rule violation, since FEAT-16.SPEC-004 always re-checks the live total at the moment any transfer is actually evaluated against the allowance.
- **Nadia's plan changes while this screen is open in a background tab** -- The allowance shown does not update until the screen is re-opened or reloaded; a stale allowance figure is a display lag only, since FEAT-16.SPEC-004 always re-reads the current plan at the moment of real enforcement.
- **Dana opens this screen for a freelancer account with zero stored bytes** -- The meter shows 0 of {allowance}, in the Normal state, with no warning banner.
- **Usage figure is exactly at the warning threshold** -- The screen enters the Near Limit state; the boundary is inclusive of the threshold value.

This screen updates no shared entity, so no concurrent-edit conflict entry applies -- every value here is read-only and sourced from FEAT-16.SPEC-004 and FEAT-16.SPEC-005.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules) | References (inbound) | Supplies the allowance amount, the warning threshold, and the At Capacity condition this screen displays |
| FEAT-16.SPEC-005 (Storage Usage Aggregation) | References (inbound) | Supplies the current aggregated total stored bytes |
| FEAT-23.SPEC-001 (Plan & Billing Screen) | Navigation (outbound) | "Review your plan" link when Nadia is near or at her allowance |
| FEAT-21.SPEC-001 (Account Profile) | Navigation (inbound) | Settings entry point into this screen |
| FEAT-31.SPEC-002 (Operator Support Session Console) / FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement) | References (inbound) | Gates Dana's read-only, session-scoped access to this screen (XBR-29) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| storage_limit_warning_shown | threshold_crossed (near_limit / at_capacity), percent_used | Screen enters the Near Limit or At Capacity state on load | N/A -- no Stage 2 metric measures storage-warning visibility directly; retained so how often freelancers approach their allowance is observable |
| storage_usage_summary_viewed | percent_used bucket, viewer_role (Nadia / Dana) | Screen finishes loading successfully | N/A -- no Stage 2 metric measures views of this screen; retained to observe usage-checking frequency ahead of any future metric needing it |

## Acceptance Criteria

**FEAT-16.SPEC-001-AC-01:** Given Nadia opens the Storage screen from Account Profile, when the figures load, then the meter shows her current total stored bytes against her allowance with the numeric label "{used} of {allowance} used".

**FEAT-16.SPEC-001-AC-02:** Given Nadia's usage is below platform parameter: `storage-usage-warning-threshold-percent` of her allowance, when the screen loads, then no warning banner is shown.

**FEAT-16.SPEC-001-AC-03:** Given Nadia's usage is at or above platform parameter: `storage-usage-warning-threshold-percent` but below her full allowance, when the screen loads, then the Near Limit warning banner appears with the "Review your plan" link.

**FEAT-16.SPEC-001-AC-04:** Given Nadia taps "Review your plan" while the Near Limit banner is shown, when the tap registers, then she is navigated to FEAT-23.SPEC-001 (Plan & Billing Screen).

**FEAT-16.SPEC-001-AC-05:** Given Nadia's usage reaches her full allowance, when the screen loads, then the At Capacity banner appears stating that new uploads won't complete until space is freed or the plan is upgraded.

**FEAT-16.SPEC-001-AC-06:** Given the storage figures fail to load, when the load fails, then the error banner "Couldn't load your storage usage. Try again." appears with a Retry control.

**FEAT-16.SPEC-001-AC-07:** Given Nadia loses connectivity while this screen is open, when connectivity drops, then the banner "You're offline -- showing your last known storage usage." appears, the last-loaded figures remain visible, and "Review your plan" is disabled.

**FEAT-16.SPEC-001-AC-08:** Given Dana opens a support session on a freelancer's account and navigates to Storage, when the screen loads, then she sees the same total and allowance figures read-only, with no "Review your plan" link shown.

**FEAT-16.SPEC-001-AC-09:** Given Owen or Priya is signed in to the client portal, when they look for any path to a storage screen, then none exists -- this screen is never reachable from the client-facing portal.

**FEAT-16.SPEC-001-AC-10:** Given an unauthenticated visitor requests this screen's address directly, when the request is made, then they are redirected to sign-in.

**FEAT-16.SPEC-001-AC-11:** Given Nadia's usage is exactly at platform parameter: `storage-usage-warning-threshold-percent`, when the screen loads, then it enters the Near Limit state (the boundary is inclusive).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 5 (Loading, Near Limit, At Capacity, Error, Offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |



# Automation Spec: Resumable Upload Transfer

## Overview

**Name:** Resumable Upload Transfer
**ID:** FEAT-16.SPEC-002
**Type:** Automation
**Purpose:** Moves a file into storage in resumable, progress-tracked chunks on behalf of the calling deliverable-layer automation, persisting resume state across a dropped connection and retrying automatically before any manual retry is offered.
**Parent Feature:** FEAT-16 -- Large File Handling & Storage

## Scope and Non-Goals

**In Scope:**
- The byte-level, chunked, resumable transfer of an uploaded file into the large-file storage capability
- Real, visible progress reporting with pause and auto-resume across a dropped connection
- Automatic retry of a failed chunk before any failure is surfaced for manual retry
- Checking the freelancer's storage allowance during the transfer and stopping cleanly if it would be exceeded
- Returning the stored-file reference and final size to the calling spec on completion, and signaling storage-usage recalculation

**Non-Goals:**
- Creating or updating the Deliverable or Deliverable Version metadata record -- owned by the calling spec (FEAT-06.SPEC-003 for a round-1 upload, FEAT-17.SPEC-003 for a re-upload); this automation returns only the stored-file reference and final size for the caller to attach
- Enforcing the per-file size ceiling -- owned by FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules) and checked by the calling spec before this automation is invoked; this automation enforces only the per-freelancer storage allowance, per FEAT-16.SPEC-004
- Serving or streaming a stored file back out for viewing or download -- owned by FEAT-16.SPEC-003 (Reliable File Delivery); this automation is upload-direction only
- Recalculating the freelancer's aggregate stored-bytes total -- owned by FEAT-16.SPEC-005 (Storage Usage Aggregation); this automation signals that recalculation on completion but does not perform it itself

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A round-1 upload transfer is handed off | FEAT-06.SPEC-003 (Resumable Upload Handling) | Fires after FEAT-06.SPEC-003 creates the Deliverable record (status Uploading) and confirms the file is within the per-file size ceiling | File bytes, file name, file size, Deliverable reference, milestone reference |
| A re-upload (version) transfer is handed off | FEAT-17.SPEC-003 (Version Creation & Preservation) | Fires when Nadia uploads a new round for an existing deliverable, after the per-file size ceiling is confirmed by the calling spec | File bytes, file name, file size, Deliverable reference, next round_number |
| Connectivity restored during an in-progress transfer | System (connectivity signal) | Fires when the device regains connectivity while a transfer for this automation is paused | Transfer identifier, bytes already committed, remaining bytes |
| Retry requested after a non-connectivity failure | FEAT-06.SPEC-003 / FEAT-17.SPEC-003 | Fires when the calling spec relays a manual retry request (from FEAT-06.SPEC-001 or FEAT-17's upload screen) for a previously failed transfer | Same file bytes as the original attempt, previous failure reason, transfer identifier |

## Processing Logic

1. Receive the file bytes, name, size, the Deliverable reference, and (for a re-upload) the target round_number from the calling spec.
2. Check the freelancer's current aggregated stored-bytes total (per FEAT-16.SPEC-005's last recalculation) against her storage allowance (per FEAT-16.SPEC-004): if this file's size would push the total over the allowance, stop and produce the Storage Allowance Exceeded outcome without transferring any bytes.
3. Begin the resumable transfer: divide the file into fixed-size chunks and hand each chunk to the large-file storage capability (FEAT-16.SPEC-007) for ingestion, committing chunks in order and persisting the last-committed chunk as the resume point after each commit.
4. Report real, visible progress -- percentage complete and an estimated time to completion -- back to the calling spec as chunks commit (ASMP-27).
5. If the connection drops mid-transfer, pause the transfer and hold the persisted resume point; surface the Paused, Auto-Resuming outcome to the calling spec.
6. When connectivity returns, resume automatically from the persisted resume point -- never restarting the file from zero.
7. If a chunk fails to commit for a reason other than dropped connectivity, retry that chunk automatically, with a bounded number of attempts, before any failure is surfaced to the calling spec.
8. When every chunk has committed, finalize the stored file, compute its final size, and return the stored-file reference and final size to the calling spec; signal FEAT-16.SPEC-005 to recalculate the freelancer's aggregated total.
9. If automatic chunk retries are exhausted, or the storage capability reports a persistent problem, mark the transfer failed without discarding already-committed chunks, and surface the failure to the calling spec for manual retry -- a subsequent retry resumes from the persisted point rather than starting over.
10. If the storage capability itself reports it cannot accept new transfers at all (FEAT-16.SPEC-007), surface an Automation Unavailable outcome before any chunk is sent.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Storage allowance exceeded | The freelancer's projected total (current total + this file's size) would exceed her allowance (FEAT-16.SPEC-004) | No bytes transferred; no stored-file reference created | Calling spec shows "You've reached your storage allowance." with a link to the plan view | FEAT-06.SPEC-003, FEAT-17.SPEC-003, FEAT-16.SPEC-004 |
| Transfer in progress | Chunks are actively committing | No metadata record change (owned by the caller); transfer state is "in progress" with a committed-bytes count | Calling spec shows live progress percentage and estimated completion | FEAT-06.SPEC-003, FEAT-17.SPEC-003 |
| Paused, auto-resuming | Connectivity drops mid-transfer | Resume point persisted at the last committed chunk; no bytes lost | Calling spec shows "Paused -- resuming when your connection returns" | FEAT-06.SPEC-003, FEAT-17.SPEC-003 |
| Transfer completes fully | Every chunk commits and the stored file is finalized | Stored-file reference and final size returned to the caller; FEAT-16.SPEC-005 signaled to recalculate the aggregate total | Calling spec proceeds to create/finalize its own metadata record and shows completion | FEAT-06.SPEC-003, FEAT-17.SPEC-003, FEAT-16.SPEC-005 |
| Transfer fails (non-connectivity) | A chunk's automatic retries are exhausted, or a persistent server-side rejection occurs | No stored-file reference returned; already-committed chunks and the resume point are preserved for a subsequent retry | Calling spec shows a failure with a Retry control that resumes rather than restarts | FEAT-06.SPEC-003, FEAT-17.SPEC-003 |
| Automation unavailable | The large-file storage capability (FEAT-16.SPEC-007) reports it cannot accept transfers at all | No bytes transferred | Calling spec shows "Uploads aren't available right now. Try again shortly." with a Retry control | FEAT-06.SPEC-003, FEAT-17.SPEC-003, FEAT-16.SPEC-007 |

## Data Model

**Reads:** The freelancer's current aggregated stored-bytes total and storage allowance, via FEAT-16.SPEC-004 and FEAT-16.SPEC-005. Deliverable Version -- the incoming file bytes and target round_number, supplied by the calling spec.
**Creates:** The stored file itself, within the large-file storage capability (FEAT-16.SPEC-007) -- the resulting stored-file reference and final size are returned to the caller, which creates the Deliverable Version record around them.
**Updates:** None on the Deliverable or Deliverable Version records directly -- those updates belong to the calling spec.
**Deletes:** None.

## Business Rules

- XBR-12: a partial transfer is never reported as complete; the caller's deliverable-ready notification path waits for this automation's "Transfer completes fully" outcome and never fires for a partial file.
- XBR-14: the per-file size ceiling (platform parameter: `deliverable-file-size-ceiling`) is checked by the caller before this automation begins; this automation checks only the per-freelancer storage allowance (platform parameter: `free-tier-storage-allowance` or `paid-tier-storage-allowance`, per FEAT-16.SPEC-004), re-evaluated against the live total at the moment the transfer starts.
- The storage-allowance check runs once at transfer start against the total known at that instant; because two transfers for the same freelancer can start close together, the allowance is re-checked again at the moment each transfer's bytes are about to commit, so a transfer that started when there was room but would now push the freelancer over is stopped mid-transfer rather than allowed to complete over the allowance.
- Automatic chunk retry and connectivity-drop pause/resume are non-blocking to the calling spec's screen -- the screen remains usable while this automation runs in the background.
- This automation serves both a first upload (via FEAT-06.SPEC-003) and a re-upload (via FEAT-17.SPEC-003) identically; it has no awareness of round numbering, which is entirely the calling spec's concern.

## Edge Cases

- **File has zero bytes** -- Rejected before any chunking begins, surfaced to the calling spec as a failure with the reason "empty file" (the calling spec presents this as its own empty-file message).
- **Connection drops and returns within the same second (flapping connectivity)** -- The pause/resume cycle is debounced: a reconnect within a short window resumes without visibly surfacing the Paused outcome to the calling spec.
- **The freelancer's storage allowance is exhausted mid-transfer (not just at the start)** -- The transfer stops at the next chunk boundary; already-committed chunks are discarded since no complete, valid stored file exists yet; the Storage Allowance Exceeded outcome is surfaced, consistent with FEAT-06.SPEC-003's mid-transfer allowance-exhaustion edge case.
- **The calling context is abandoned mid-transfer (e.g., the browser tab is closed)** -- The transfer is cancelled; no partial stored-file reference is ever returned to a caller, and no Deliverable Version is ever created from an incomplete transfer.
- **Concurrent trigger firing (two transfers for the freelancer's different files start at effectively the same time)** -- Each transfer proceeds independently and each is checked against the allowance using the total known at its own start; because the allowance check re-runs at commit time (per Business Rules), a case where both would individually fit but not together results in only the first to reach commit succeeding, and the second is stopped with the Storage Allowance Exceeded outcome even though its own start-time check passed.
- **Trigger fires while a previous run is in flight (a retry request arrives while an automatic chunk retry for the same transfer is already in progress)** -- The manual retry request is ignored while the automatic retry continues; no duplicate transfer is started for the same file.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-003 (Resumable Upload Handling) | Triggered by (inbound) | Hands off round-1 upload bytes for transfer and receives progress, pause, and completion outcomes |
| FEAT-17.SPEC-003 (Version Creation & Preservation) | Triggered by (inbound) | Hands off re-upload (version) bytes for transfer, identically to a round-1 upload |
| FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules) | References (inbound) | Defines the per-freelancer storage allowance this automation checks against |
| FEAT-16.SPEC-005 (Storage Usage Aggregation) | Triggers (outbound) | Completion of a transfer signals recalculation of the freelancer's aggregated total |
| FEAT-16.SPEC-007 (Large-File Storage & Delivery Capability) | Triggers (outbound) | Performs the actual chunked ingestion and reports transfer status back to this automation |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Transfer completion feeds the calling spec's own trail entry for the upload (XBR-05) -- this automation does not write the trail entry directly |

## Analytics and Success Signals

- **large_upload_started** (file_size_bytes, deliverable_reference, upload_context: first_upload / re_upload) -- supports success-metrics.md: "Large File Upload Success at Scale"
- **large_upload_resumed** (pause_duration, bytes_committed_at_resume) -- supports success-metrics.md: "Deliverable Upload Reliability"
- **large_upload_completed** (file_size_bytes, total_transfer_duration) -- supports success-metrics.md: "Deliverable Upload Reliability"
- **large_upload_failed** (reason: storage_allowance / server_rejection / capability_unavailable / empty_file, file_size_bytes) -- supports success-metrics.md: "Deliverable Upload Reliability"

## Acceptance Criteria

**FEAT-16.SPEC-002-AC-01:** Given FEAT-06.SPEC-003 hands off a 600 MB video file for Nadia's first upload on a milestone, when the transfer begins, then chunks commit in order with real progress and an estimated completion reported back to FEAT-06.SPEC-003.

**FEAT-16.SPEC-002-AC-02:** Given Nadia's transfer is interrupted by a dropped connection, when connectivity returns, then the transfer resumes automatically from the last committed chunk rather than restarting from zero.

**FEAT-16.SPEC-002-AC-03:** Given every chunk of Nadia's file commits successfully, when the last chunk finalizes, then the stored-file reference and final size are returned to the calling spec and FEAT-16.SPEC-005 is signaled to recalculate her aggregated total.

**FEAT-16.SPEC-002-AC-04:** Given transferring Nadia's file would push her aggregated total over her storage allowance, when the transfer is about to begin, then no bytes are transferred and the Storage Allowance Exceeded outcome is returned to the calling spec.

**FEAT-16.SPEC-002-AC-05:** Given a chunk fails to commit for a reason other than a dropped connection, when the failure occurs, then that chunk is retried automatically before any failure is surfaced to the calling spec.

**FEAT-16.SPEC-002-AC-06:** Given automatic chunk retries are exhausted, when the transfer ultimately fails, then already-committed chunks and the resume point are preserved, and a subsequent retry resumes from that point rather than starting over.

**FEAT-16.SPEC-002-AC-07:** Given FEAT-17.SPEC-003 hands off a re-upload for an existing deliverable, when the transfer completes, then it behaves identically to a first upload -- returning a stored-file reference and final size with no awareness of round numbering.

**FEAT-16.SPEC-002-AC-08:** Given the large-file storage capability reports it cannot accept transfers at all, when a transfer is attempted, then no chunks are sent and the calling spec shows "Uploads aren't available right now. Try again shortly."

**FEAT-16.SPEC-002-AC-09:** Given Nadia selects a zero-byte file, when the automation evaluates it, then the transfer is rejected before chunking begins with the reason "empty file".

**FEAT-16.SPEC-002-AC-10:** Given the freelancer's storage allowance is exhausted mid-transfer after passing the start-time check, when the next chunk boundary is reached, then the transfer stops, already-committed chunks are discarded, and the Storage Allowance Exceeded outcome is surfaced.

**FEAT-16.SPEC-002-AC-11:** Given two of Nadia's transfers for different files each individually fit her remaining allowance but not together, when both reach their commit point, then the first to commit succeeds and the second is stopped with the Storage Allowance Exceeded outcome.

**FEAT-16.SPEC-002-AC-12:** Given a manual retry request arrives while an automatic chunk retry for the same transfer is already in progress, when the request arrives, then it is ignored and the existing transfer continues uninterrupted.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 4 (round-1 handoff, re-upload handoff, connectivity restored, manual retry) | 4 |
| Outcome Paths | 6 | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Reliable File Delivery

## Overview

**Name:** Reliable File Delivery
**ID:** FEAT-16.SPEC-003
**Type:** Automation
**Purpose:** Serves a stored deliverable file for in-browser streaming or direct download with no client-side install, resolving the requested version's stored-file reference and retrying a failed transfer automatically before a manual retry is offered.
**Parent Feature:** FEAT-16 -- Large File Handling & Storage

## Scope and Non-Goals

**In Scope:**
- Resolving a Deliverable or a specific Deliverable Version's stored-file reference for viewing or download
- Streaming or serving the stored bytes directly in the browser, with real progress for large files
- Retrying a failed streaming or download transfer automatically before a manual retry is offered
- Respecting the read-only, no-download boundary of an operator support session

**Non-Goals:**
- Uploading or storing a file -- owned by FEAT-16.SPEC-002 (Resumable Upload Transfer); this automation is delivery-direction only
- Deciding who may view a given deliverable -- authorization is owned by FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules) and FEAT-07's own visibility rules; this automation serves bytes only after the calling screen has already confirmed the viewer is authorized
- Listing available versions or letting a viewer choose which round to open -- owned by FEAT-17 (Deliverable Version History); this automation serves whichever version reference it is given
- Generating a printable or downloadable export of any other record type (invoices, data exports) -- out of scope per this feature's Summary, which limits its concern to deliverable file bytes

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia previews her own uploaded deliverable | FEAT-06.SPEC-002 (Deliverable List & Management) | Fires when Nadia opens a preview of a deliverable she owns | Deliverable reference (latest version by default) |
| A client contact opens a deliverable to view | FEAT-07.SPEC-001 (Deliverable Comment Thread) | Fires when Owen or Priya opens a deliverable version to stream, view, or download it, after FEAT-07's own visibility rule has confirmed access | Deliverable Version reference, requesting contact's role |
| Owen reviews a deliverable before approving | FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Fires when Owen opens the deliverable attached to the milestone he is reviewing | Deliverable reference (latest active version) |
| Dana views a deliverable inside a read-only support session | FEAT-31.SPEC-002 (Operator Support Session Console) | Fires when Dana, inside an open support session on a named freelancer's account, opens a deliverable in the mirrored view to stream it (never to download, per XBR-29) | Deliverable Version reference, viewer role: Dana (support session) |
| A previously failed delivery is retried | Any of the above (FEAT-06.SPEC-002, FEAT-07.SPEC-001, FEAT-08.SPEC-001, FEAT-31.SPEC-002) | Fires when the viewer taps Retry on a failed stream or download | Same version reference as the original attempt, previous failure reason |

## Processing Logic

1. Receive the Deliverable or Deliverable Version reference and the requesting viewer's role from the calling spec.
2. Resolve the stored-file reference for the requested version through the large-file storage capability (FEAT-16.SPEC-007).
3. If the requesting viewer is Dana in a support session, confirm the request is a view-only stream and never offer a download control, per XBR-29; if a download was requested rather than a stream, decline before any bytes move.
4. Begin serving the bytes: for a stream, deliver progressively so playback or viewing can begin before the full file arrives; for a download, transfer the full file directly to the viewer's device, with no client-side install required.
5. Report real, visible progress with an estimated completion for large files as bytes are served (ASMP-27), consistent with the same progress vocabulary FEAT-06.SPEC-003 uses for uploads.
6. If the transfer stalls or a portion fails for a reason other than the viewer's own connection dropping, retry automatically before surfacing any failure.
7. If the viewer's own connection drops mid-transfer, pause and resume automatically from where delivery left off once connectivity returns, without restarting from zero.
8. When delivery completes fully, hand control to the calling screen's own viewer/player for streamed content, or complete the file save for a download.
9. If delivery fails after automatic retries are exhausted, surface the failure to the calling spec for a manual retry that resumes rather than restarts.
10. If the large-file storage capability reports it cannot serve requests at all, surface an Automation Unavailable outcome before any bytes are served.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Delivery in progress | Bytes are actively streaming or downloading | None | Calling screen shows live progress with an estimated completion | FEAT-06.SPEC-002, FEAT-07.SPEC-001, FEAT-08.SPEC-001 |
| Paused, auto-resuming | The viewer's connection drops mid-delivery | None | Calling screen shows "Paused -- resuming when your connection returns" | FEAT-06.SPEC-002, FEAT-07.SPEC-001, FEAT-08.SPEC-001 |
| Delivery completes fully | All bytes are served | None | Streamed content plays/displays in the calling screen, or the download completes to the viewer's device | FEAT-06.SPEC-002, FEAT-07.SPEC-001, FEAT-08.SPEC-001 |
| Delivery fails (non-connectivity) | A portion's automatic retries are exhausted, or the storage capability reports a persistent problem for this request | None | Calling screen shows a failure with a Retry control that resumes rather than restarts | FEAT-06.SPEC-002, FEAT-07.SPEC-001, FEAT-08.SPEC-001 |
| Download declined for a support session | Dana's session requests a download rather than a stream | None | Calling screen shows "Downloads aren't available in a support session." and offers the view-only stream instead | FEAT-31.SPEC-003 |
| Automation unavailable | The large-file storage capability (FEAT-16.SPEC-007) reports it cannot serve requests at all | None | Calling screen shows "This file isn't available right now. Try again shortly." with a Retry control | FEAT-06.SPEC-002, FEAT-07.SPEC-001, FEAT-08.SPEC-001, FEAT-16.SPEC-007 |

## Data Model

**Reads:** Deliverable / Deliverable Version -- the stored file reference for the requested version, resolved through the large-file storage capability.
**Creates:** None.
**Updates:** None -- delivery is a read-only operation; it does not set `first_client_view_at` (that field is written by FEAT-13 from the calling screen's own view event, not by this automation).
**Deletes:** None.

## Business Rules

- XBR-29: for an operator support session, this automation serves view-only streaming and never a download, and never for a downloadable file type when the viewer is Dana.
- Real, visible progress with an estimated completion is required for large files -- an indefinite spinner is never shown for a file above a trivial size (ASMP-27, mirroring FEAT-06.SPEC-003's upload progress requirement).
- Retry-then-manual-retry applies uniformly across streaming and download: automatic retry always runs first, and a manual Retry control appears only once automatic retries are exhausted.
- This automation serves a version reference exactly as given by the calling spec; it never substitutes a different version and never assumes "latest" unless the caller explicitly requested the latest version.

## Edge Cases

- **The requested version was purged before the request completes (freelancer's account was deleted mid-session)** -- Delivery fails with "This file is no longer available." and no retry is offered, since the underlying bytes no longer exist (FEAT-16.SPEC-006).
- **Connection drops and returns within the same second (flapping connectivity)** -- The pause/resume cycle is debounced: a brief reconnect resumes without visibly surfacing the Paused outcome to the viewer.
- **Two viewers open the same deliverable version at the same time** -- Each delivery proceeds entirely independently; there is no shared or exclusive lock on a stored file, since delivery is a read-only operation with no data to contend over.
- **Concurrent trigger firing (Owen streams a deliverable while Priya downloads the same version at the same time)** -- Both deliveries proceed independently and simultaneously; neither affects the other's progress or outcome.
- **Trigger fires while a previous run is in flight (viewer taps Retry while an automatic retry for the same request is already in progress)** -- The manual retry request is ignored while the automatic retry continues; no duplicate delivery is started for the same request.
- **A viewer navigates away mid-stream** -- Delivery for that request is simply abandoned; no state is left behind, since delivery makes no data changes.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-002 (Deliverable List & Management) | Triggered by (inbound) | Nadia's own preview action requests delivery |
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | Triggered by (inbound) | Owen's or Priya's stream/download action requests delivery |
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Triggered by (inbound) | Owen's review-before-approval action requests delivery |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Triggered by (inbound) | Dana's in-session viewing action requests delivery as a view-only stream |
| FEAT-16.SPEC-007 (Large-File Storage & Delivery Capability) | Triggers (outbound) | Performs the actual byte-range streaming/download and reports transfer status back to this automation |
| FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement) | References (inbound) | Governs the download-declined outcome for Dana's read-only session (XBR-29) |

## Analytics and Success Signals

- **large_file_delivery_started** (deliverable_reference, mode: stream / download, viewer_role) -- supports success-metrics.md: "Large File Upload Success at Scale"
- **large_file_delivery_completed** (deliverable_reference, mode, total_transfer_duration) -- supports success-metrics.md: "Large File Upload Success at Scale"
- **large_file_delivery_failed** (reason: server_rejection / capability_unavailable / version_purged, mode) -- N/A -- no Stage 2 metric measures delivery failure directly; retained so the frequency of failed streams or downloads is observable

## Acceptance Criteria

**FEAT-16.SPEC-003-AC-01:** Given Owen opens a deliverable version from FEAT-07.SPEC-001 to view it, when the request is made, then the stored-file reference resolves and streaming begins with real, visible progress.

**FEAT-16.SPEC-003-AC-02:** Given Priya's connection drops mid-stream, when connectivity returns, then delivery resumes automatically from where it left off rather than restarting from zero.

**FEAT-16.SPEC-003-AC-03:** Given a delivery transfer stalls for a reason other than a dropped connection, when the stall occurs, then delivery is retried automatically before any failure is surfaced to the viewer.

**FEAT-16.SPEC-003-AC-04:** Given automatic retries for a delivery are exhausted, when the delivery ultimately fails, then the calling screen shows a failure with a Retry control that resumes rather than restarts.

**FEAT-16.SPEC-003-AC-05:** Given Dana is in a read-only support session and requests to download a deliverable file, when the request is made, then the download is declined with "Downloads aren't available in a support session." and only a view-only stream is offered.

**FEAT-16.SPEC-003-AC-06:** Given Owen opens the deliverable attached to a milestone he is reviewing in FEAT-08.SPEC-001, when he opens it, then the file streams in-browser with no client-side install required.

**FEAT-16.SPEC-003-AC-07:** Given the large-file storage capability reports it cannot serve requests at all, when a delivery is attempted, then no bytes are served and "This file isn't available right now. Try again shortly." appears with a Retry control.

**FEAT-16.SPEC-003-AC-08:** Given a requested version's stored bytes were purged by a completed account deletion (FEAT-16.SPEC-006), when delivery is attempted, then it fails with "This file is no longer available." and no retry is offered.

**FEAT-16.SPEC-003-AC-09:** Given Owen streams a deliverable version at the same time Priya downloads it, when both requests are active, then each proceeds independently with no interference between them.

**FEAT-16.SPEC-003-AC-10:** Given a viewer taps Retry while an automatic retry for the same delivery request is already in progress, when the tap registers, then it is ignored and the existing delivery continues uninterrupted.

**FEAT-16.SPEC-003-AC-11:** Given Nadia previews her own deliverable from FEAT-06.SPEC-002, when the preview opens, then the file is delivered the same way as for a client contact, with real progress and no install required.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 5 (Nadia preview, client view, milestone review, Dana support-session view, retry) | 5 |
| Outcome Paths | 6 | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Storage Limit & Size Ceiling Rules

## Overview

**Name:** Storage Limit & Size Ceiling Rules
**ID:** FEAT-16.SPEC-004
**Type:** Logic/Rule
**Purpose:** Defines the per-file size ceiling and the per-freelancer storage allowance -- the only two limits this feature enforces -- and the warning threshold that triggers a pre-limit warning, along with who may trigger these checks.
**Parent Feature:** FEAT-16 -- Large File Handling & Storage
**Governed Entity:** Deliverable (storage-capacity fields only)

## Scope and Non-Goals

**In Scope:**
- The per-file size ceiling for an uploaded deliverable file (accommodating video files somewhat over 1 GB)
- The per-freelancer total storage allowance, resolved from the freelancer's Subscription Plan tier
- The warning threshold that triggers a pre-limit, in-product warning
- Authorization for who may trigger a storage-affecting upload and who may view storage figures

**Non-Goals:**
- Validating every other Deliverable field (kind, link validity, milestone requirement) -- owned by FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules); this spec governs storage-capacity fields only
- Performing the byte-level transfer or the actual size-ceiling and allowance checks at transfer time -- owned by FEAT-16.SPEC-002 (Resumable Upload Transfer), which enforces this spec's rules during the transfer
- Recalculating the freelancer's current aggregated total -- owned by FEAT-16.SPEC-005 (Storage Usage Aggregation); this spec defines the allowance and threshold the aggregation is checked against, not the aggregation itself
- Displaying the usage figures -- owned by FEAT-16.SPEC-001 (Storage Usage Summary), which reads this spec's allowance and threshold values

## Governed Entity

**Entity:** Deliverable
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| kind | enum | Uploaded file or linked external asset -- out of scope for this spec |
| file or link | text/binary | The uploaded file or reachable link -- this spec governs the file's size only, for uploaded files |
| milestone | reference | Owning Milestone -- out of scope for this spec |
| uploaded_at | date | Upload timestamp -- out of scope for this spec |
| size | number (bytes) | For uploaded files, the file's committed size -- governed by this spec's per-file ceiling rule |
| status | enum | Uploading, Active, Superseded, Removed -- out of scope for this spec |
| link_status | enum | Reachable or flagged -- out of scope for this spec |
| first_client_view_at | date | Timestamp of first client view -- out of scope for this spec |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-16.SPEC-002 | Resumable Upload Transfer | Storage-allowance check, before a transfer begins and again at the moment each transfer's bytes are about to commit |
| FEAT-06.SPEC-005 | Deliverable Validation & Removal Eligibility Rules | Per-file size ceiling check, on file selection, before the transfer begins (restates this spec's ceiling value via the shared platform parameter: `deliverable-file-size-ceiling`) |
| FEAT-16.SPEC-001 | Storage Usage Summary | Displays the allowance, the warning threshold, and the At Capacity condition this spec defines; triggers no check itself |
| FEAT-16.SPEC-005 | Storage Usage Aggregation | Reads the allowance and threshold this spec defines to determine whether the recalculated total should raise a warning |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| kind | No validation beyond data type -- governed by FEAT-06.SPEC-005 | Always | -- | -- | No |
| file or link | For uploaded files, the file's size must not exceed platform parameter: `deliverable-file-size-ceiling` | kind = uploaded file | Before the transfer begins (checked authoritatively by FEAT-16.SPEC-002; the message is shown by the calling screen via FEAT-06.SPEC-005) | "This file is larger than the size limit for deliverables. Compress it or share it by link instead." | Yes |
| milestone | No validation beyond data type -- governed by FEAT-06.SPEC-005 | Always | -- | -- | No |
| uploaded_at | No validation beyond data type | Always | -- | -- | No |
| size | Once committed, the persisted size can never exceed platform parameter: `deliverable-file-size-ceiling` (restates the file/link rule on the finalized value) | For uploaded files, once the transfer commits | On transfer completion | Same message as the file/link rule above -- a committed size over the ceiling cannot occur, because the transfer itself is stopped before completion (FEAT-16.SPEC-002) | Yes |
| status | No validation beyond data type -- Deliverable's status vocabulary is owned by FEAT-06/FEAT-17 | Always | -- | -- | No |
| link_status | No validation beyond data type -- governed by FEAT-06.SPEC-004 | Always | -- | -- | No |
| first_client_view_at | No validation beyond data type -- owned by FEAT-13 | Always | -- | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Storage allowance check | Deliverable.size (the file being transferred), the freelancer's aggregated stored-bytes total (external, maintained by FEAT-16.SPEC-005), Subscription Plan.tier | The freelancer's current aggregated total plus this file's size must not exceed the allowance resolved from her plan tier: platform parameter: `free-tier-storage-allowance` for Free, platform parameter: `paid-tier-storage-allowance` for Paid | "You've reached your storage allowance." |
| Warning threshold | The freelancer's aggregated stored-bytes total, her resolved storage allowance | When the total reaches platform parameter: `storage-usage-warning-threshold-percent` of the allowance, the Near Limit warning condition is true for display on FEAT-16.SPEC-001 | N/A -- non-blocking advisory, not an error; displayed inline per FEAT-16.SPEC-001, never as a delivery-channel notification (this feature's Communications field is N/A) |
| At capacity condition | The freelancer's aggregated stored-bytes total, her resolved storage allowance | When the total reaches (or would be pushed to) 100% of the allowance, the At Capacity condition is true, and a further upload is blocked by the storage allowance check above | "You've reached your storage allowance. New uploads won't complete until you free up space or upgrade your plan." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Upload a file (subject to the size ceiling and storage allowance) | Nadia (Freelancer) | Always, subject to the Field Validation and Cross-Field rules above | -- |
| Upload a file (subject to the size ceiling and storage allowance) | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never | No upload control exists on any client-facing screen; the Access Matrix's Milestones & Deliverables entitlement for both roles is Own-only (view, comment, approve for Owen; view, comment for Priya), with no upload capability |
| Upload a file (subject to the size ceiling and storage allowance) | Dana (Support Operator) | Never | No upload control exists in a support session; Dana's Milestones & Deliverables entitlement is View, with no file downloads or uploads (ASMP-18, XBR-29) |
| View storage usage figures (total, allowance, warning state) | Nadia (Freelancer) | Always | -- |
| View storage usage figures (total, allowance, warning state) | Dana (Support Operator) | Always, read-only, only during an active support session on that freelancer's account (FEAT-31.SPEC-003) | Outside an active support session, the figures are unreachable; no separate denied experience, since no navigation path exists |
| View storage usage figures (total, allowance, warning state) | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never | No storage-usage surface exists in the client portal; the Access Matrix's Subscription & Account Data entitlement for both roles is None |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Deliverable.size | Derived from the committed transfer's final byte count (FEAT-16.SPEC-002) | On transfer completion | No |
| Freelancer's aggregated stored-bytes total (external) | Sum of the current size across every non-purged Deliverable Version the freelancer owns, recalculated by FEAT-16.SPEC-005 | On upload completion, deliverable removal, or storage purge | No |
| Resolved storage allowance | Looked up from the freelancer's current Subscription Plan tier: platform parameter: `free-tier-storage-allowance` for Free, platform parameter: `paid-tier-storage-allowance` for Paid | Always -- re-evaluated at each check against the live plan, never cached across a plan change | No |
| Near Limit / At Capacity condition | Derived by comparing the aggregated total to the resolved allowance and the warning threshold | Always -- re-evaluated on every display or enforcement check | No |

## Business Rules

- XBR-14: the per-file size ceiling and the per-freelancer storage allowance are the only two limits this feature enforces, and a pre-limit warning is raised before the allowance is reached; the ceiling is set to accommodate video files somewhat over 1 GB.
- The per-file ceiling and the per-freelancer allowance are independent checks: a file can pass the size ceiling and still be rejected by the storage allowance if the freelancer's account is already near full, and vice versa a file within the allowance's remaining headroom is still rejected if it individually exceeds the per-file ceiling.
- A linked external asset (Figma, Drive, Dropbox) never counts against the per-freelancer storage allowance, since this feature's storage capability only ever stores files the freelancer actually uploads (Non-Goals: linked assets are referenced by URL, never copied in).
- The storage allowance is read from the freelancer's live Subscription Plan tier at the moment of every check -- a plan upgrade takes effect for the very next check, with no separate re-provisioning step.

## Edge Cases

- **File size exactly at the per-file ceiling** -- Passes validation; one byte over fails.
- **Freelancer's projected total exactly at her allowance** -- Passes (the allowance is inclusive of its own boundary); one byte over fails.
- **Freelancer's total exactly at the warning threshold** -- The Near Limit condition is true; the boundary is inclusive of the threshold value.
- **Freelancer downgrades her plan while already over the new, lower allowance's threshold** -- No existing stored bytes are purged or blocked from delivery; the At Capacity condition becomes true immediately for any further upload, and the warning shows on FEAT-16.SPEC-001, but nothing already stored is affected (consistent with this feature's Non-Goal: no automatic purge of active deliverables' storage while the account remains active).
- **Freelancer's plan is upgraded mid-transfer (the allowance increases while a transfer that would have exceeded the old allowance is in flight)** -- The transfer's commit-time allowance re-check (FEAT-16.SPEC-002) uses the plan as it stands at that moment, so an upgrade that lands before the commit point lets the transfer complete against the new, higher allowance.
- **Dana's support session is opened for a freelancer already At Capacity** -- Dana sees the same At Capacity figures read-only; no upload control is ever shown to her regardless of the freelancer's capacity state.

## Acceptance Criteria

**FEAT-16.SPEC-004-AC-01:** Given Nadia selects a file exactly at platform parameter: `deliverable-file-size-ceiling`, when the size check runs, then the file passes.

**FEAT-16.SPEC-004-AC-02:** Given Nadia selects a file one byte over platform parameter: `deliverable-file-size-ceiling`, when the size check runs, then she sees "This file is larger than the size limit for deliverables. Compress it or share it by link instead." and the transfer does not begin.

**FEAT-16.SPEC-004-AC-03:** Given Nadia's projected total (current total plus the new file's size) is exactly at her resolved storage allowance, when the allowance check runs, then the transfer proceeds.

**FEAT-16.SPEC-004-AC-04:** Given Nadia's projected total would be one byte over her resolved storage allowance, when the allowance check runs, then she sees "You've reached your storage allowance." and the transfer does not begin.

**FEAT-16.SPEC-004-AC-05:** Given Nadia's aggregated total reaches exactly platform parameter: `storage-usage-warning-threshold-percent` of her allowance, when FEAT-16.SPEC-001 next loads, then it shows the Near Limit warning.

**FEAT-16.SPEC-004-AC-06:** Given Nadia's aggregated total is below platform parameter: `storage-usage-warning-threshold-percent` of her allowance, when FEAT-16.SPEC-001 next loads, then no warning is shown.

**FEAT-16.SPEC-004-AC-07:** Given Nadia is on the free tier, when her resolved allowance is looked up, then it equals platform parameter: `free-tier-storage-allowance`; given she is on a paid tier, then it equals platform parameter: `paid-tier-storage-allowance`.

**FEAT-16.SPEC-004-AC-08:** Given Nadia attempts to upload a file, when the action is evaluated, then it is allowed (subject to the size and allowance checks); given Owen, Priya, or Dana attempts the same action, then no upload control is available to them at all.

**FEAT-16.SPEC-004-AC-09:** Given Dana opens a support session on a freelancer's account, when she views the storage figures, then she sees them read-only; given Owen or Priya looks for any storage-usage surface, then none exists in the client portal.

**FEAT-16.SPEC-004-AC-10:** Given Nadia references a Figma or Drive link as a deliverable rather than uploading a file, when her aggregated total is recalculated, then the linked asset contributes nothing to it.

**FEAT-16.SPEC-004-AC-11:** Given Nadia downgrades her plan and her existing stored bytes now exceed the new, lower allowance, when the downgrade takes effect, then no existing stored file is purged or blocked from delivery, and any further upload is blocked by the At Capacity condition.

**FEAT-16.SPEC-004-AC-12:** Given a transfer that would have exceeded Nadia's old allowance is in flight when she completes a plan upgrade, when the transfer reaches its commit-time allowance re-check, then it is evaluated against the new, higher allowance.

**FEAT-16.SPEC-004-AC-13:** Given Nadia's committed Deliverable.size is checked after a transfer completes, when the check runs, then it can never exceed platform parameter: `deliverable-file-size-ceiling`, since FEAT-16.SPEC-002 stops any transfer that would breach it before completion.

**FEAT-16.SPEC-004-AC-14:** Given the freelancer's storage allowance is read for any check, when it is looked up, then it always reflects her current Subscription Plan tier at that exact moment, never a cached or stale value.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Storage Usage Aggregation

## Overview

**Name:** Storage Usage Aggregation
**ID:** FEAT-16.SPEC-005
**Type:** Automation
**Purpose:** Recalculates the freelancer's total stored bytes whenever a file finishes uploading, a deliverable is removed, or stored bytes are purged, feeding the usage summary screen and the limit rule.
**Parent Feature:** FEAT-16 -- Large File Handling & Storage

## Scope and Non-Goals

**In Scope:**
- Recalculating the freelancer's aggregated total stored bytes on every event that changes it
- Making the recalculated total available to FEAT-16.SPEC-001 (display) and FEAT-16.SPEC-004 (the limit and warning checks)

**Non-Goals:**
- Defining the storage allowance or the warning threshold the recalculated total is checked against -- owned by FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules); this automation only produces the total, it does not decide what the total means
- Performing the upload transfer that produces a newly stored file's size -- owned by FEAT-16.SPEC-002 (Resumable Upload Transfer), which signals this automation on completion
- Performing the actual byte-level purge on account deletion -- owned by FEAT-16.SPEC-006 (Stored File Purge on Account Deletion), which signals this automation once the purge completes
- Displaying the total -- owned by FEAT-16.SPEC-001 (Storage Usage Summary), which reads this automation's most recent result

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An upload transfer completes fully | FEAT-16.SPEC-002 (Resumable Upload Transfer) | Fires the instant a transfer's "Transfer completes fully" outcome is reached | The newly stored file's final size, the freelancer account it belongs to |
| A deliverable is removed | FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules) | Fires when Nadia removes a deliverable whose bytes are released (not retained) -- see Business Rules; a removal that retains bytes triggers no recalculation | Deliverable reference, freelancer account |
| Stored bytes are purged | FEAT-16.SPEC-006 (Stored File Purge on Account Deletion) | Fires once a purge completes | The freelancer account whose bytes were purged, total bytes purged |

## Processing Logic

1. Receive the triggering event (upload completion, deliverable removal, or purge completion) and the affected freelancer account.
2. Read the current set of the freelancer's non-purged Deliverable Versions and their stored sizes.
3. Sum the size across every one of those versions to produce the freelancer's current total stored bytes.
4. Persist the recalculated total as the freelancer's current aggregated figure, replacing the previous value.
5. Compare the new total against the freelancer's storage allowance and warning threshold (FEAT-16.SPEC-004) to determine whether the Near Limit or At Capacity condition now applies.
6. Make the recalculated total and the resulting Near Limit/At Capacity condition available for the next time FEAT-16.SPEC-001 loads or FEAT-16.SPEC-004 is checked.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Recalculation completes, total unchanged in warning status | New total is still below the warning threshold (or still above it, if it already was) | Aggregated total updated to the new value | No immediate feedback -- the updated figure is picked up the next time FEAT-16.SPEC-001 loads | FEAT-16.SPEC-001, FEAT-16.SPEC-004 |
| Recalculation completes, Near Limit newly reached | New total crosses into the warning threshold for the first time | Aggregated total updated; Near Limit condition now true | No push notification (this feature's Communications field is N/A) -- the warning appears the next time FEAT-16.SPEC-001 loads, or the next time an upload screen in FEAT-06 checks the condition | FEAT-16.SPEC-001, FEAT-06 (upload screen's inline warning) |
| Recalculation completes, total drops back below the threshold | A removal or purge brings the total back under the warning threshold | Aggregated total updated; Near Limit/At Capacity condition now false | No feedback beyond the figure updating on next view | FEAT-16.SPEC-001, FEAT-16.SPEC-004 |
| Recalculation fails | Processing error while reading versions or summing sizes | The previous total remains in place (no partial or zeroed total is ever persisted) | No user-visible failure -- the last known-good total continues to display until the next successful recalculation | FEAT-16.SPEC-001 |

## Data Model

**Reads:** Deliverable Version -- the current size of every non-purged version belonging to the freelancer's account.
**Creates:** None.
**Updates:** The freelancer's aggregated stored-bytes total (the derived field named in this feature's Non-Functional Notes: "storage usage totals per freelancer, used for plan-limit warnings").
**Deletes:** None.

## Business Rules

- This automation is the sole owner of the freelancer's aggregated stored-bytes total; no other spec computes or caches this figure independently -- FEAT-16.SPEC-001 and FEAT-16.SPEC-004 both read this automation's result rather than recalculating it themselves.
- Removing a deliverable in FEAT-06 does not, by itself, purge its stored bytes -- per this feature's Non-Goals, storage is retained for the life of the account with no automatic purge while active; this automation only recalculates the total when FEAT-06.SPEC-005 explicitly signals a removal that does affect the count (i.e., only when bytes are actually released, not on every removal). Where a removed deliverable's version bytes are retained rather than released, this automation makes no change and the total is unaffected.
- A failed recalculation never zeroes or corrupts the previously known total -- the last successful value remains authoritative until a subsequent recalculation succeeds.
- Recalculation runs synchronously with its trigger and completes before the total is considered current for the next display or limit check -- there is no separate scheduled recalculation pass.

## Edge Cases

- **Two triggers arrive for the same freelancer at effectively the same time (an upload completes while a purge is also finishing)** -- Each recalculation reads the version set as it stands at that moment; the recalculation that reads last reflects both changes, and the total converges to the correct value once both triggers have been processed, even if an intermediate read briefly reflects only one of them.
- **Trigger fires while a previous recalculation for the same freelancer is still in flight** -- The second trigger's recalculation waits for the first to finish reading and summing before it begins its own pass, so recalculations for one freelancer never interleave and never persist an inconsistent total.
- **The freelancer has zero stored bytes (new account, or every version purged)** -- The recalculated total is zero; FEAT-16.SPEC-001 displays "0 of {allowance} used" with no warning.
- **A recalculation is triggered for a freelancer account that no longer exists (a race with the final stage of account deletion)** -- The recalculation is a no-op; there is no remaining account to update, and no error is surfaced, since the account and its figures are being removed regardless.
- **Recalculation runs while FEAT-16.SPEC-001 is open and being viewed** -- The screen does not live-update mid-view; the newly recalculated total is picked up the next time the screen loads or is refreshed, consistent with FEAT-16.SPEC-001's own Edge Cases.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-16.SPEC-002 (Resumable Upload Transfer) | Triggered by (inbound) | Transfer completion signals recalculation |
| FEAT-06.SPEC-005 (Deliverable Validation & Removal Eligibility Rules) | Triggered by (inbound) | A removal that releases stored bytes signals recalculation |
| FEAT-16.SPEC-006 (Stored File Purge on Account Deletion) | Triggered by (inbound) | Purge completion signals recalculation |
| FEAT-16.SPEC-001 (Storage Usage Summary) | Affects (outbound) | Reads this automation's most recent aggregated total for display |
| FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules) | Affects (outbound) | The recalculated total is what this spec's allowance and warning-threshold checks are evaluated against |

## Analytics and Success Signals

- **storage_usage_recalculated** (trigger: upload_completed / deliverable_removed / storage_purged, new_total_bytes, warning_state: normal / near_limit / at_capacity) -- supports success-metrics.md: "Large File Upload Success at Scale"
- **storage_usage_recalculation_failed** (trigger) -- N/A -- no Stage 2 metric measures recalculation failure directly; retained so a stale-total condition is observable and diagnosable

## Acceptance Criteria

**FEAT-16.SPEC-005-AC-01:** Given Nadia's upload transfer completes fully, when FEAT-16.SPEC-002 signals completion, then this automation recalculates her total stored bytes to include the newly stored file's size.

**FEAT-16.SPEC-005-AC-02:** Given a removal in FEAT-06.SPEC-005 releases stored bytes, when the removal completes, then this automation recalculates her total to exclude the released bytes.

**FEAT-16.SPEC-005-AC-03:** Given FEAT-16.SPEC-006 completes a storage purge on account deletion, when the purge finishes, then this automation recalculates the total to reflect the purge (down to zero, if all bytes were purged).

**FEAT-16.SPEC-005-AC-04:** Given a recalculation brings Nadia's total to exactly platform parameter: `storage-usage-warning-threshold-percent` of her allowance, when the recalculation completes, then the Near Limit condition becomes true for the next FEAT-16.SPEC-001 load.

**FEAT-16.SPEC-005-AC-05:** Given a recalculation brings Nadia's total back below the warning threshold after a removal, when the recalculation completes, then the Near Limit condition becomes false.

**FEAT-16.SPEC-005-AC-06:** Given a recalculation fails due to a processing error, when the failure occurs, then the previously known total remains in place and is not zeroed or corrupted.

**FEAT-16.SPEC-005-AC-07:** Given a removal in FEAT-06 does not release any stored bytes (the version is retained per this feature's Non-Goals), when the removal completes, then no recalculation is triggered and the total is unaffected.

**FEAT-16.SPEC-005-AC-08:** Given an upload completes and a purge finishes for the same freelancer at effectively the same time, when both recalculations process, then the total converges to correctly reflect both changes.

**FEAT-16.SPEC-005-AC-09:** Given a second recalculation trigger arrives for a freelancer while a prior recalculation for that same freelancer is still in flight, when the second trigger fires, then it waits for the first to finish before it begins, so no inconsistent total is ever persisted.

**FEAT-16.SPEC-005-AC-10:** Given a freelancer has zero stored bytes, when a recalculation runs, then the total resolves to zero and FEAT-16.SPEC-001 shows "0 of {allowance} used" with no warning.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (upload completed, deliverable removed, storage purged) | 3 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: Stored File Purge on Account Deletion

## Overview

**Name:** Stored File Purge on Account Deletion
**ID:** FEAT-16.SPEC-006
**Type:** Automation
**Purpose:** Permanently purges every stored file and version's bytes from the storage capability when the freelancer's account is deleted, as one step in FEAT-24's account-deletion cascade.
**Parent Feature:** FEAT-16 -- Large File Handling & Storage

## Scope and Non-Goals

**In Scope:**
- Permanently purging every stored file belonging to the deleted freelancer's account, across every Deliverable Version, from the large-file storage capability
- Signaling FEAT-16.SPEC-005 to recalculate (to zero) once the purge completes
- Leaving no restore path once the purge completes

**Non-Goals:**
- Deciding that the account should be deleted, warning about open items, or capturing Nadia's confirmation -- owned by FEAT-24 (Data Export & Account Deletion), which orchestrates the overall cascade and invokes this automation as one of its steps
- Removing the Deliverable and Deliverable Version metadata records themselves -- owned by FEAT-24's own cascade processing; this automation purges only the underlying stored bytes at the storage layer
- Purging storage for any reason other than account deletion -- excluded per this feature's Non-Goals: "no automatic purge of active, superseded, or removed deliverables' storage while the account remains active"; this automation runs only as part of FEAT-24's cascade
- Purging financial or other non-storage records subject to legal retention -- owned by FEAT-24.SPEC-005 (Legal Retention Purge); stored deliverable files carry no legal retention requirement and are purged in full, unlike financial records

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Account deletion cascade reaches the storage step | FEAT-24.SPEC-004 (Account Deletion Processing) | Fires once Nadia's account deletion has been confirmed and the cascade has passed its commit point (the payment-account disconnect, FEAT-24.SPEC-004 step 5), as the irreversible stored-byte purge finalization step (FEAT-24.SPEC-004 step 8); never invoked while a revert of the account is still possible | The freelancer account being deleted, the full set of that account's Deliverable Versions and their stored-file references |

## Processing Logic

1. Receive the freelancer account being deleted from FEAT-24.SPEC-004.
2. Enumerate every Deliverable Version belonging to that account, across every Client, Project, and Milestone the freelancer owned, regardless of a version's `is_latest` state or its parent Deliverable's status (Active, Superseded, or Removed) -- every version's bytes are in scope.
3. For each version, request the large-file storage capability (FEAT-16.SPEC-007) permanently delete the stored file referenced by that version.
4. Confirm each deletion succeeds before considering that version's bytes purged.
5. Once every version's bytes are confirmed purged, signal FEAT-16.SPEC-005 to recalculate the freelancer's aggregated total (which resolves to zero, since the account holds no data at this point).
6. Report completion back to FEAT-24.SPEC-004 so the overall cascade can proceed or, if this step fails, so FEAT-24.SPEC-004 retries this step until it succeeds per its own post-commit failure handling (the cascade is never halted or reverted at this point).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Purge completes fully | Every version's stored bytes are confirmed deleted | All of the account's stored files are permanently removed from the storage capability; FEAT-16.SPEC-005 signaled to recalculate | None directly -- the deletion has already been confirmed by Nadia through FEAT-24's own screens; there is no further user-facing feedback at the storage layer | FEAT-16.SPEC-005, FEAT-24.SPEC-004 |
| Purge partially fails | One or more versions' deletions cannot be confirmed | No version is left in an ambiguous "maybe deleted" state -- confirmed deletions are final and irreversible; unconfirmed ones are retried | None directly to Nadia (the account and its confirmation flow have already completed by this point) -- reported to FEAT-24.SPEC-004, which retries this step per its own retry handling | FEAT-24.SPEC-004 |
| Purge automation unavailable | The large-file storage capability (FEAT-16.SPEC-007) reports it cannot process deletion requests at all | No bytes purged in this attempt | None directly to Nadia -- reported to FEAT-24.SPEC-004 for retry | FEAT-24.SPEC-004, FEAT-16.SPEC-007 |

## Data Model

**Reads:** Deliverable Version -- every version belonging to the deleted freelancer's account and its stored-file reference.
**Creates:** None.
**Updates:** None on the Deliverable Version metadata record itself (that removal belongs to FEAT-24's own cascade); this automation acts only on the underlying stored bytes.
**Deletes:** Every stored file referenced by the freelancer's Deliverable Versions, at the storage layer, permanently.

## Business Rules

- This purge is a one-way, destructive action executed only when FEAT-24 deletes the account itself, which is already a terminal, non-reversible action in the product's own definition (this feature's Non-Goals: "Restoring purged files after account deletion" is intentionally excluded).
- Every version's bytes are purged, with no cascade beyond the deleted account's own records and no exception for a version that is `is_latest`, superseded, or belongs to a removed deliverable -- the dependency map records Deliverable Version as "never deleted in-product; removed only by FEAT-24," and this automation is the executor of that removal at the storage layer.
- This automation never purges another freelancer's data -- it acts strictly on the Deliverable Versions belonging to the one account FEAT-24.SPEC-004 names (ASMP-23, strict per-client and per-freelancer isolation).
- If this step fails, the overall account deletion does not report itself complete -- because this step runs only after FEAT-24.SPEC-004's commit point, a failure here never reverts the account to Active; FEAT-24.SPEC-004 keeps the account's data hidden and retries this step until it succeeds before the account is considered deleted.

## Edge Cases

- **A version's storage deletion request fails on the first attempt** -- The deletion is retried before this step is reported failed to FEAT-24.SPEC-004; a failure at the storage layer never halts or reverts the cascade, which has already passed its commit point; FEAT-24.SPEC-004 retries the step until it succeeds.
- **The account has zero stored bytes (a freelancer who never uploaded a file)** -- The purge step completes immediately with nothing to delete, and reports success to FEAT-24.SPEC-004.
- **A delivery request (FEAT-16.SPEC-003) for one of this account's files arrives after the purge completes but before FEAT-24's cascade finishes removing the metadata records** -- The delivery fails with "This file is no longer available." (FEAT-16.SPEC-003's own edge case), since the underlying bytes are already gone even if a metadata record briefly still exists.
- **The account deletion cascade is itself interrupted after this step completes but before later steps finish** -- The purge already completed is not undone; FEAT-24's own retry of the remaining cascade steps does not re-purge already-purged bytes, since a second deletion request for already-deleted storage is a no-op.
- **Concurrent trigger firing (this automation is invoked twice for the same account, due to a cascade retry)** -- The second invocation's deletion requests for already-purged versions are no-ops; the automation completes successfully without erroring on bytes that no longer exist.
- **Trigger fires while a previous purge run for the same account is still in flight** -- A second invocation for the same account waits for the first to finish rather than running a duplicate purge pass concurrently, avoiding two processes racing to delete the same stored files.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-004 (Account Deletion Processing) | Triggered by (inbound) | Invokes this automation as one step of the account-deletion cascade and receives its completion or failure report |
| FEAT-16.SPEC-007 (Large-File Storage & Delivery Capability) | Triggers (outbound) | Performs the actual permanent deletion of each stored file |
| FEAT-16.SPEC-005 (Storage Usage Aggregation) | Triggers (outbound) | Signaled to recalculate (to zero) once the purge completes |
| FEAT-16.SPEC-003 (Reliable File Delivery) | Affects (outbound) | A delivery request for a purged version fails with "This file is no longer available." after this automation completes |

## Analytics and Success Signals

- **storage_purge_completed** (freelancer_account_reference, versions_purged_count, total_bytes_purged) -- N/A -- no Stage 2 metric measures account-deletion storage purges; retained as the audit-relevant record that this destructive, irreversible step actually completed
- **storage_purge_failed** (freelancer_account_reference, reason: capability_unavailable / deletion_unconfirmed) -- N/A -- no Stage 2 metric measures purge failures; retained so an incomplete purge blocking the wider account-deletion cascade is observable

## Acceptance Criteria

**FEAT-16.SPEC-006-AC-01:** Given FEAT-24.SPEC-004 invokes this automation for Nadia's deleted account, when it runs, then every Deliverable Version's stored file belonging to that account is requested for permanent deletion from the storage capability.

**FEAT-16.SPEC-006-AC-02:** Given every version's deletion is confirmed, when the purge completes, then FEAT-16.SPEC-005 is signaled to recalculate the account's total to zero, and completion is reported to FEAT-24.SPEC-004.

**FEAT-16.SPEC-006-AC-03:** Given a version's deletion fails on first attempt, when the failure occurs, then it is retried before this step is reported failed to FEAT-24.SPEC-004.

**FEAT-16.SPEC-006-AC-04:** Given the large-file storage capability reports it cannot process deletion requests at all, when this automation runs, then no bytes are purged in that attempt and the failure is reported to FEAT-24.SPEC-004 for retry.

**FEAT-16.SPEC-006-AC-05:** Given a freelancer's account has zero stored bytes, when this automation runs, then it completes immediately with nothing to delete and reports success.

**FEAT-16.SPEC-006-AC-06:** Given a purge completes and a later delivery request (FEAT-16.SPEC-003) arrives for one of the purged versions, when the request is made, then it fails with "This file is no longer available." and no retry is offered.

**FEAT-16.SPEC-006-AC-07:** Given the account-deletion cascade is interrupted after this step completes, when the remaining cascade steps are later retried, then already-purged bytes are not re-purged and no error occurs for the already-deleted files.

**FEAT-16.SPEC-006-AC-08:** Given this automation is invoked twice for the same account due to a cascade retry, when the second invocation runs, then its deletion requests for already-purged versions succeed as no-ops rather than erroring.

**FEAT-16.SPEC-006-AC-09:** Given a second invocation for the same account arrives while a first purge run is still in flight, when it arrives, then it waits for the first to finish rather than running a duplicate purge pass concurrently.

**FEAT-16.SPEC-006-AC-10:** Given this automation processes a purge for one freelancer's account, when it enumerates versions to delete, then it never touches another freelancer's stored files.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 (account-deletion cascade reaches this step) | 1 |
| Outcome Paths | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Integration Spec: Large-File Storage & Delivery Capability

## Overview

**Name:** Large-File Storage & Delivery Capability
**ID:** FEAT-16.SPEC-007
**Type:** Integration
**Purpose:** Owns the product's contract with the external large-file storage and delivery capability -- resumable ingestion, byte-range streaming and download, and reported transfer status -- within the stated infrastructure budget.
**Parent Feature:** FEAT-16 -- Large File Handling & Storage

## Scope and Non-Goals

**In Scope:**
- Ingesting uploaded file bytes resumably, in chunks, and reporting transfer progress and status back to the product
- Serving stored bytes for in-browser streaming and direct download, with byte-range support for progressive delivery
- Reporting transfer, delivery, and deletion outcomes (committed, paused, completed, rejected, failed, confirmed) back to the product
- Permanently deleting stored files on request
- User-facing behavior when the capability is slow, unavailable, or rejects a request
- Disclosure to the freelancer about what file data is shared with this capability

**Non-Goals:**
- Choosing the storage vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for a specific storage provider
- The chunking strategy, progress calculation, or retry-before-manual-retry logic -- owned by FEAT-16.SPEC-002 (Resumable Upload Transfer) and FEAT-16.SPEC-003 (Reliable File Delivery), which consume this capability's contract; this spec defines only what crosses the boundary to and from the capability
- Deciding the per-file size ceiling or the per-freelancer storage allowance -- owned by FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules); this capability stores and serves whatever the product hands it, within whatever limits the product has already enforced before contacting it
- Hosting or mirroring content from a linked external asset (Figma, Google Drive, Dropbox) -- excluded per scope-boundaries.md (SC-08); this capability only ever stores files the freelancer actually uploads

## Capability Category

**Category:** File storage (large-file storage and delivery, with resumable ingestion and byte-range serving)
**Dependency Source:** ASMP-30 -- "Large-file storage and delivery with version history" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Large-file storage and delivery with version history (ASMP-30)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-06, FEAT-16, FEAT-17; Integration spec: FEAT-16.SPEC-007)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision; BRIEF.md's Constraints name only a roughly $100/month infrastructure budget as a bound on this capability, not a specific vendor.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Nadia's large file upload (tens of MB, sometimes over 1 GB) is accepted, transferred resumably, and survives a dropped connection | Resumable large-file upload | FEAT-16.SPEC-002 (Resumable Upload Transfer) |
| A client contact streams or downloads a deliverable directly in the browser, with no install | Reliable delivery | FEAT-16.SPEC-003 (Reliable File Delivery) |
| Nadia's storage stays within the stated infrastructure budget while still accepting large files and preserving every version | Cost-conscious storage | FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules), FEAT-16.SPEC-005 (Storage Usage Aggregation) |
| Every stored file's bytes are permanently, irreversibly removed when a freelancer's account is deleted | Cost-conscious storage (bounded, non-open-ended retention) | FEAT-16.SPEC-006 (Stored File Purge on Account Deletion) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| File bytes (the uploaded deliverable) | Deliverable Version -- file | A transfer begins and as chunks commit (FEAT-16.SPEC-002) | The capability must have the actual content to store |
| File size | Deliverable -- size | Reported as chunks commit and finalized on completion | The capability tracks and confirms the stored object's size, which the product uses to finalize the Deliverable Version's size field |
| Stored-file reference | Deliverable Version -- file (reference) | A delivery is requested (FEAT-16.SPEC-003) | The capability must know which stored object to stream or serve for download |
| Deletion request (stored-file reference) | Deliverable Version -- file (reference) | FEAT-16.SPEC-006 purges an account's bytes on deletion | The capability must know exactly which objects to permanently delete |

A Deliverable's milestone, project, and client context; any comment, approval, invoice, or other record content; and the viewer's identity beyond what the product itself needs to authorize a request never leave the product to this capability -- authorization decisions (who may upload, view, or download) are made entirely product-side, before this capability is ever contacted.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Chunk-committed status and progress | Continuously during an active upload transfer | Deliverable -- transient transfer progress (not persisted beyond the active transfer); feeds FEAT-16.SPEC-002's progress reporting |
| Transfer completion, with the final stored-file reference and size | The transfer finalizes | Deliverable -- size; Deliverable Version -- file (the stored-file reference) |
| Transfer rejection or persistent failure | The capability cannot commit a chunk after its own retries, or rejects the transfer | No entity field changes -- Deliverable remains Uploading; feeds FEAT-16.SPEC-002's failure outcome |
| Delivery byte-range status and progress | Continuously during an active stream or download | None -- delivery is read-only; feeds FEAT-16.SPEC-003's progress reporting |
| Archive download transfer completion (archive reference, transfer confirmation) | A download transfer of a stored data-export archive file finishes | None on any Deliverable record; reported to FEAT-24.SPEC-003 so it can advance the Data Export Archive from Ready to Downloaded |
| Delivery failure | A stream or download cannot be completed | None; feeds FEAT-16.SPEC-003's failure outcome |
| Deletion confirmation | A requested permanent deletion completes | None directly on the (already-deleted) Deliverable Version record; confirms to FEAT-16.SPEC-006 that the bytes are gone |
| Capacity/availability status | At any time the capability cannot accept transfers, serve requests, or process deletions at all | None; feeds the Automation Unavailable outcome in FEAT-16.SPEC-002, FEAT-16.SPEC-003, and FEAT-16.SPEC-006 |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Chunk committed | A chunk of an active upload finishes writing | Committed-bytes count advances for that transfer | Progress percentage advances on the calling upload screen | FEAT-16.SPEC-002 |
| Transfer paused (connectivity) | The capability detects the connection dropped mid-transfer | Resume point persisted at the last committed chunk | "Paused -- resuming when your connection returns" | FEAT-16.SPEC-002 |
| Transfer completed | Every chunk of an upload commits and the stored file is finalized | Stored-file reference and final size available to return to the caller | Upload screen shows completion | FEAT-16.SPEC-002 |
| Transfer rejected or persistently failed | A non-connectivity problem occurs after the capability's own retries are exhausted | No stored-file reference produced | Failure with a Retry control surfaced by the calling automation | FEAT-16.SPEC-002 |
| Delivery byte range served | During an active stream or download, a range of bytes is served | None | Progress advances on the calling viewing screen | FEAT-16.SPEC-003 |
| Archive download delivered | A download transfer of a stored data-export archive file (submitted by FEAT-24.SPEC-003) completes | Archive reference and transfer confirmation made available to the consumer; no stored file or Deliverable record changes here | None directly -- the resulting "Downloaded on {date}" status is shown by FEAT-24.SPEC-001 after the consumer acts | FEAT-24.SPEC-003 |
| Delivery paused (connectivity) | The viewer's connection drops mid-delivery | None | "Paused -- resuming when your connection returns" | FEAT-16.SPEC-003 |
| Delivery persistently failed | A non-connectivity problem occurs after the capability's own retries are exhausted | None | Failure with a Retry control surfaced by the calling automation | FEAT-16.SPEC-003 |
| Deletion confirmed | A requested permanent deletion completes | The stored file no longer exists | None directly -- the deletion was already confirmed to Nadia through FEAT-24's own account-deletion flow before this step ran | FEAT-16.SPEC-006 |
| Capability capacity/availability problem reported | The capability cannot accept new transfers, serve requests, or process deletions at all | None | The relevant automation's "not available right now" message | FEAT-16.SPEC-002, FEAT-16.SPEC-003, FEAT-16.SPEC-006 |

Multi-step handling of every event above (retry sequencing, resume-point persistence, outcome selection, user feedback wording) is owned by the automation named in Affected Specs; this Integration spec defines only the event and its immediate data.

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-06.SPEC-001 (Deliverable Upload) | The upload progress indicator continues to advance, more slowly, with real percentage and an estimated completion that lengthens -- never an indefinite spinner. No "still working" interruption is shown; slow progress is simply shown as slow progress. | The Upload Failed state appears: "Uploads aren't available right now. Try again shortly." with a Retry control; the selected file remains chosen so Retry does not require re-selecting it. No Deliverable is left in a half-created state. | The Upload Failed state appears with the ceiling- or allowance-specific message from FEAT-06.SPEC-005 or FEAT-16.SPEC-004 when the rejection is a limit; for any other rejection reason, "This upload couldn't be completed. Try again." with Retry, and the selected file preserved. |
| FEAT-17.SPEC-001 (New Version Upload) | Same as FEAT-06.SPEC-001 -- progress continues, more slowly, with a lengthening estimate. | Same as FEAT-06.SPEC-001 -- "Uploads aren't available right now. Try again shortly." with Retry; no partial version is ever created. | Same as FEAT-06.SPEC-001, applied to a re-upload: limit-specific message where applicable, otherwise a generic retry message; the existing prior version remains fully intact and untouched regardless of this attempt's outcome. |
| FEAT-06.SPEC-002 (Deliverable List & Management) | Nadia's own preview shows continuing progress with a lengthening estimate; the rest of the screen (deliverable list, remove/replace controls) remains fully usable. | Preview shows "This file isn't available right now. Try again shortly." with Retry; the deliverable's own record and its list entry are unaffected. | N/A -- a preview request is never itself rejected by content or authorization at the capability level; any authorization decision has already been made product-side before this capability is contacted. |
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | Owen's or Priya's stream/download shows continuing progress with a lengthening estimate; the comment thread itself remains fully usable and unaffected. | "This file isn't available right now. Try again shortly." with Retry; the deliverable remains visible and commentable even while its bytes cannot be streamed. | N/A -- as above, authorization is resolved before this capability is contacted, so it never rejects a request FEAT-07's own visibility rule has already approved. |
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Owen's review of the deliverable shows continuing progress with a lengthening estimate; the Approve control's own enablement (gated on the deliverable and comment thread having loaded per FEAT-08's own rules) is unaffected by delivery slowness once the deliverable content itself has begun rendering. | "This file isn't available right now. Try again shortly." with Retry; Owen can still see the milestone's other context (comments, milestone details) and is not forced to wait on delivery to do anything else on the screen. | N/A -- as above. |
| FEAT-17.SPEC-002 (Version Browser & Comparison) | Whichever round is open shows continuing progress with a lengthening estimate; switching to a different round starts its own independent delivery. | "This file isn't available right now. Try again shortly." with Retry, scoped to the round currently open; other rounds can still be selected and attempted independently. | N/A -- as above. |

Every degraded state above leaves data consistent: no half-created Deliverable Version, no metadata record pointing at a stored file that was never actually finalized.

## Consent and Disclosure

- **First-upload disclosure** -- The first time Nadia uploads a deliverable file (from FEAT-06.SPEC-001), a one-time notice appears before the upload begins: "Files you upload are stored by Clientroom's file storage service and are shared only with the client contacts you send them to." with "Continue" and "Cancel" options. Shown once; afterwards, uploading proceeds without repeating the notice, and a "How your files are stored" link on FEAT-06.SPEC-001 and FEAT-17.SPEC-001 reopens the same notice on request.
- **Delivery and deletion requests carry no separate disclosure** -- A stream, download, or account-deletion purge request references a file already covered by the first-upload disclosure above; no new data category is introduced by these requests, so no additional consent moment is required.
- **What is never shared** -- The Deliverable's milestone, project, and client context, every comment and approval record, and every other record type in the product (invoices, activity trail entries, account details) stay entirely inside the product and are never sent to this capability. This boundary is stated in the first-upload disclosure notice.

## Edge Cases

- **A "transfer completed" event arrives twice for the same upload** -- The second delivery changes nothing: the Deliverable Version already reflects the first completion's stored-file reference and size, and no duplicate version or duplicate completion feedback is produced.
- **Events for the same transfer arrive out of order (a stale "chunk committed" event arrives after "transfer completed")** -- The stale event is ignored once the transfer has already finalized; the transfer's state reflects the most advanced event received, not the most recently arrived one.
- **A delivery event arrives for a Deliverable Version whose bytes were already purged (FEAT-16.SPEC-006)** -- The event is discarded with no effect; a fresh delivery attempt against a purged version independently produces FEAT-16.SPEC-003's "This file is no longer available" outcome.
- **The capability goes down mid-transfer before any chunk commits** -- No Deliverable Version is ever created from an unconfirmed transfer; the Deliverable remains Uploading and the capability-down message is shown, with no half-created stored file left behind.
- **A deletion confirmation for an already-deleted file is received again (duplicate confirmation)** -- Treated as a no-op; FEAT-16.SPEC-006 does not error or attempt a second deletion of bytes already confirmed gone.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-16.SPEC-002 (Resumable Upload Transfer) | Triggered by (inbound) | Hands chunks to this capability for ingestion and consumes its transfer-status events |
| FEAT-16.SPEC-003 (Reliable File Delivery) | Triggered by (inbound) | Requests streaming/download from this capability and consumes its delivery-status events |
| FEAT-16.SPEC-006 (Stored File Purge on Account Deletion) | Triggered by (inbound) | Requests permanent deletion from this capability and consumes its deletion-confirmation events |
| FEAT-24.SPEC-003 (Data Export Archive Generation) | Triggered by (inbound) / Affects (outbound) | Submits the assembled archive file for storage and delivery, and receives the "archive download delivered" event when the download transfer completes |
| FEAT-06.SPEC-001 (Deliverable Upload) | Affects (outbound) | Upload progress, degradation states, and the first-upload disclosure surface here |
| FEAT-17.SPEC-001 (New Version Upload) | Affects (outbound) | Same, for a re-upload |
| FEAT-06.SPEC-002 (Deliverable List & Management) | Affects (outbound) | Nadia's own preview delivery and its degradation states surface here |
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | Affects (outbound) | Client-contact streaming/download and its degradation states surface here |
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Affects (outbound) | Owen's pre-approval review delivery and its degradation states surface here |
| FEAT-17.SPEC-002 (Version Browser & Comparison) | Affects (outbound) | Delivery of any browsed round and its degradation states surface here |

## Analytics and Success Signals

- **storage_capability_degradation_shown** (condition: slow / down / rejects; direction: upload / delivery / deletion; screen: spec ID) -- supports success-metrics.md: "Large File Upload Success at Scale"
- **storage_capability_event_received** (event_type, direction: upload / delivery / deletion) -- N/A -- no Stage 2 metric measures raw event volume; retained so the underlying event traffic behind FEAT-16.SPEC-002/003/006's own analytics is observable during operation

## Acceptance Criteria

**FEAT-16.SPEC-007-AC-01:** Given Nadia begins uploading a 600 MB video file on FEAT-06.SPEC-001, when chunks commit, then this capability enables real, visible progress reporting that never appears as an indefinite spinner.

**FEAT-16.SPEC-007-AC-02:** Given Owen opens a deliverable to stream it from FEAT-07.SPEC-001, when the request is made, then this capability serves the stored bytes progressively with no client-side install required.

**FEAT-16.SPEC-007-AC-03:** Given a "transfer completed" event arrives for Nadia's upload, when it is received, then the stored-file reference and final size become available to FEAT-16.SPEC-002 to return to its caller.

**FEAT-16.SPEC-007-AC-04:** Given a "transfer rejected or persistently failed" event arrives, when it is received, then no stored-file reference is produced and FEAT-16.SPEC-002 surfaces a failure with Retry.

**FEAT-16.SPEC-007-AC-05:** Given a "delivery persistently failed" event arrives, when it is received, then FEAT-16.SPEC-003 surfaces a failure with Retry and no data changes occur.

**FEAT-16.SPEC-007-AC-06:** Given a "deletion confirmed" event arrives for a purged version, when it is received, then FEAT-16.SPEC-006 treats that version's bytes as permanently gone.

**FEAT-16.SPEC-007-AC-07:** Given this capability reports it is slow while Nadia is uploading on FEAT-06.SPEC-001, when the slowdown is detected, then progress continues to advance with a lengthening estimated completion, never an indefinite spinner.

**FEAT-16.SPEC-007-AC-08:** Given this capability reports it is down while Owen is trying to open a deliverable on FEAT-08.SPEC-001, when the request is attempted, then "This file isn't available right now. Try again shortly." appears with Retry, and the rest of the milestone review screen remains usable.

**FEAT-16.SPEC-007-AC-09:** Given this capability reports it is down while Nadia is uploading on FEAT-17.SPEC-001, when the transfer is attempted, then "Uploads aren't available right now. Try again shortly." appears and no partial version is ever created.

**FEAT-16.SPEC-007-AC-10:** Given Nadia has never uploaded a deliverable before, when she attempts her first upload on FEAT-06.SPEC-001, then the first-upload disclosure notice appears with "Continue" and "Cancel", and no file bytes leave the product until she chooses "Continue".

**FEAT-16.SPEC-007-AC-11:** Given Nadia has already seen the first-upload disclosure, when she uploads a subsequent deliverable, then the notice does not reappear automatically, and a "How your files are stored" link is available to reopen it.

**FEAT-16.SPEC-007-AC-12:** Given a "transfer completed" event is delivered twice for the same upload, when the second delivery arrives, then nothing changes and no duplicate Deliverable Version is created.

**FEAT-16.SPEC-007-AC-13:** Given a stale "chunk committed" event for a transfer arrives after that transfer's "transfer completed" event, when it arrives, then it is ignored and the transfer's state remains "completed".

**FEAT-16.SPEC-007-AC-14:** Given a delivery event arrives for a Deliverable Version already purged by FEAT-16.SPEC-006, when it arrives, then it is discarded with no effect, and a fresh delivery attempt against that version produces "This file is no longer available."

**FEAT-16.SPEC-007-AC-15:** Given this capability goes down before any chunk of a transfer commits, when the outage is detected, then no Deliverable Version is created and the Deliverable remains Uploading with the capability-down message shown.

**FEAT-16.SPEC-007-AC-16:** Given Nadia's data-export archive is stored and she downloads it from FEAT-24.SPEC-001, when the download transfer completes, then this capability emits an "archive download delivered" event carrying the archive reference and transfer confirmation to FEAT-24.SPEC-003, and no Deliverable or stored-file record changes.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 10 | 10 |
| Degradation Paths | 15 (6 screens; 3 N/A cells excluded) | 15 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 5 | 5 |
