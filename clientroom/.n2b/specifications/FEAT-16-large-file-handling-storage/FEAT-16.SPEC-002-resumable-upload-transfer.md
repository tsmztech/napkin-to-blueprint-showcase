---
document_type: spec
spec_type: automation
spec_id: FEAT-16.SPEC-002
spec_name: Resumable Upload Transfer
spec_slug: resumable-upload-transfer
parent_feature: FEAT-16
parent_feature_name: Large File Handling & Storage
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

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
