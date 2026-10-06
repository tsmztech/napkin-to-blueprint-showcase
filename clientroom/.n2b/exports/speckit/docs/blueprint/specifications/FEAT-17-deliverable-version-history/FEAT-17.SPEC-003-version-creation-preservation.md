---
document_type: spec
spec_type: automation
spec_id: FEAT-17.SPEC-003
spec_name: Version Creation & Preservation
spec_slug: version-creation-preservation
parent_feature: FEAT-17
parent_feature_name: Deliverable Version History
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 16
---

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
