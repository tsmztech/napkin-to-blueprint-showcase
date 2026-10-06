---
document_type: spec
spec_type: automation
spec_id: FEAT-16.SPEC-006
spec_name: Stored File Purge on Account Deletion
spec_slug: stored-file-purge-on-account-deletion
parent_feature: FEAT-16
parent_feature_name: Large File Handling & Storage
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

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
