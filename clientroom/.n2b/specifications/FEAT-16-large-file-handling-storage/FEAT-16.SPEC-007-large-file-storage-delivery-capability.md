---
document_type: spec
spec_type: integration
spec_id: FEAT-16.SPEC-007
spec_name: Large-File Storage & Delivery Capability
spec_slug: large-file-storage-delivery-capability
parent_feature: FEAT-16
parent_feature_name: Large File Handling & Storage
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 16
---

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
