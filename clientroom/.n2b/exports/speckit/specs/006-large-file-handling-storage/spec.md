# Feature Specification: Large File Handling & Storage

**Blueprint feature:** FEAT-16
**Priority tier:** Core
**Build order:** 006 of 33
**Depends on:** —
**Blueprint source:** `docs/blueprint/specifications/FEAT-16-large-file-handling-storage/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Storage Usage Summary (Priority: P1)

Nadia sees her total stored bytes against her freelancer storage allowance, with warning styling as she approaches the limit and a path into her plan view; Dana sees the same figures read-only during a logged support session.

**Acceptance Scenarios:**

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

### User Story 2 - Resumable Upload Transfer (Priority: P1)

Moves a file into storage in resumable, progress-tracked chunks on behalf of the calling deliverable-layer automation, persisting resume state across a dropped connection and retrying automatically before any manual retry is offered.

**Acceptance Scenarios:**

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

### User Story 3 - Reliable File Delivery (Priority: P1)

Serves a stored deliverable file for in-browser streaming or direct download with no client-side install, resolving the requested version's stored-file reference and retrying a failed transfer automatically before a manual retry is offered.

**Acceptance Scenarios:**

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

### User Story 4 - Storage Limit & Size Ceiling Rules (Priority: P1)

Defines the per-file size ceiling and the per-freelancer storage allowance -- the only two limits this feature enforces -- and the warning threshold that triggers a pre-limit warning, along with who may trigger these checks.

**Acceptance Scenarios:**

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

### User Story 5 - Storage Usage Aggregation (Priority: P1)

Recalculates the freelancer's total stored bytes whenever a file finishes uploading, a deliverable is removed, or stored bytes are purged, feeding the usage summary screen and the limit rule.

**Acceptance Scenarios:**

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

### User Story 6 - Stored File Purge on Account Deletion (Priority: P1)

Permanently purges every stored file and version's bytes from the storage capability when the freelancer's account is deleted, as one step in FEAT-24's account-deletion cascade.

**Acceptance Scenarios:**

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

### User Story 7 - Large-File Storage & Delivery Capability (Priority: P1)

Owns the product's contract with the external large-file storage and delivery capability -- resumable ingestion, byte-range streaming and download, and reported transfer status -- within the stated infrastructure budget.

**Acceptance Scenarios:**

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

### Edge Cases

- **FEAT-16.SPEC-001 (Storage Usage Summary):** The meter reflects only the last completed aggregation, so it can lag an in-progress transfer, and a plan change in a background tab shows a stale allowance until reload. A zero-byte account shows 0 of the allowance in the Normal state, and a usage figure exactly at the warning threshold enters Near Limit (inclusive boundary). Source: `docs/blueprint/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-001-storage-usage-summary.md` (section: Edge Cases)
- **FEAT-16.SPEC-002 (Resumable Upload Transfer):** A zero-byte file is rejected before chunking with an empty-file reason, and flapping connectivity is debounced so a quick reconnect resumes without surfacing Paused. An allowance exhausted mid-transfer stops at the next chunk boundary and discards committed chunks, and an abandoned context cancels the transfer without creating a Deliverable Version. Source: `docs/blueprint/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-002-resumable-upload-transfer.md` (section: Edge Cases)
- **FEAT-16.SPEC-003 (Reliable File Delivery):** A version purged mid-session fails delivery with a no-longer-available message and no retry, and flapping connectivity is debounced. Delivery is read-only, so simultaneous viewers or a stream plus a download of the same version proceed independently. Source: `docs/blueprint/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-003-reliable-file-delivery.md` (section: Edge Cases)
- **FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules):** Limits are inclusive: a file exactly at the ceiling, a projected total exactly at the allowance, or a total exactly at the warning threshold all behave as at-boundary passes (one byte over fails; threshold means Near Limit). A downgrade below existing usage purges or blocks nothing already stored but marks At Capacity for further uploads. Source: `docs/blueprint/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-004-storage-limit-size-ceiling-rules.md` (section: Edge Cases)
- **FEAT-16.SPEC-005 (Storage Usage Aggregation):** Recalculations for one freelancer are serialized so the one that reads last reflects the final version set, a zero-byte account totals zero, and a recalculation for an account already deleted is a silent no-op. Source: `docs/blueprint/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-005-storage-usage-aggregation.md` (section: Edge Cases)
- **FEAT-16.SPEC-006 (Stored File Purge on Account Deletion):** A failed storage deletion is retried before the step is reported failed to FEAT-24.SPEC-004 and never halts or reverts the cascade. An account with no stored bytes completes immediately, deliveries after the purge fail with the no-longer-available message, and an interrupted cascade does not re-purge or undo completed purges. Source: `docs/blueprint/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-006-stored-file-purge-on-account-deletion.md` (section: Edge Cases)
- **FEAT-16.SPEC-007 (Large-File Storage & Delivery Capability):** A duplicate transfer-completed event changes nothing, and a stale out-of-order chunk event is ignored once the transfer has finalized. Events for already-purged versions are discarded, and if the capability goes down mid-transfer the Deliverable stays Uploading with no half-created version. Source: `docs/blueprint/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-007-large-file-storage-delivery-capability.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-16.SPEC-001** (Storage Usage Summary) as specified: Nadia sees her total stored bytes against her freelancer storage allowance, with warning styling as she approaches the limit and a path into her plan view; Dana sees the same figures read-only during a logged support session. Full spec: `docs/blueprint/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-001-storage-usage-summary.md`
- **FR-002**: The system MUST implement **FEAT-16.SPEC-002** (Resumable Upload Transfer) as specified: Moves a file into storage in resumable, progress-tracked chunks on behalf of the calling deliverable-layer automation, persisting resume state across a dropped connection and retrying automatically before any manual retry is offered. Full spec: `docs/blueprint/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-002-resumable-upload-transfer.md`
- **FR-003**: The system MUST implement **FEAT-16.SPEC-003** (Reliable File Delivery) as specified: Serves a stored deliverable file for in-browser streaming or direct download with no client-side install, resolving the requested version's stored-file reference and retrying a failed transfer automatically before a manual retry is offered. Full spec: `docs/blueprint/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-003-reliable-file-delivery.md`
- **FR-004**: The system MUST implement **FEAT-16.SPEC-004** (Storage Limit & Size Ceiling Rules) as specified: Defines the per-file size ceiling and the per-freelancer storage allowance -- the only two limits this feature enforces -- and the warning threshold that triggers a pre-limit warning, along with who may trigger these checks. Full spec: `docs/blueprint/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-004-storage-limit-size-ceiling-rules.md`
- **FR-005**: The system MUST implement **FEAT-16.SPEC-005** (Storage Usage Aggregation) as specified: Recalculates the freelancer's total stored bytes whenever a file finishes uploading, a deliverable is removed, or stored bytes are purged, feeding the usage summary screen and the limit rule. Full spec: `docs/blueprint/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-005-storage-usage-aggregation.md`
- **FR-006**: The system MUST implement **FEAT-16.SPEC-006** (Stored File Purge on Account Deletion) as specified: Permanently purges every stored file and version's bytes from the storage capability when the freelancer's account is deleted, as one step in FEAT-24's account-deletion cascade. Full spec: `docs/blueprint/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-006-stored-file-purge-on-account-deletion.md`
- **FR-007**: The system MUST implement **FEAT-16.SPEC-007** (Large-File Storage & Delivery Capability) as specified: Owns the product's contract with the external large-file storage and delivery capability -- resumable ingestion, byte-range streaming and download, and reported transfer status -- within the stated infrastructure budget. Full spec: `docs/blueprint/specifications/FEAT-16-large-file-handling-storage/FEAT-16.SPEC-007-large-file-storage-delivery-capability.md`

### Key Entities

- Deliverable (read/update — file storage)
- Deliverable Version (create)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Upload and download performance for files up to 1 GB+ remains consistent, with no more than a brief, visible delay, as a freelancer's account approaches typical year-one storage volumes (metric: Large File Upload Success at Scale). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Large upload starts, resumed uploads and storage-limit warnings are each observable as distinct signals (large_upload_started, large_upload_resumed, storage_limit_warning_shown). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-22**: Deliverables are typically tens of MB and sometimes over 1 GB, retained with version history for the life of the account. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-30**: The product requires file storage and delivery capability for large files within a modest budget. Full register: `docs/blueprint/features/assumptions-constraints.md`
