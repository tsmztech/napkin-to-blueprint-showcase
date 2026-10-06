---
document_type: spec
spec_type: automation
spec_id: FEAT-06.SPEC-003
spec_name: Resumable Upload Handling
spec_slug: resumable-upload-handling
parent_feature: FEAT-06
parent_feature_name: Deliverable Upload & Sharing
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

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
