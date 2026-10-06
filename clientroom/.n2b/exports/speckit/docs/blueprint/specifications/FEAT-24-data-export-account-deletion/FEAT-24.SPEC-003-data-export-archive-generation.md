---
document_type: spec
spec_type: automation
spec_id: FEAT-24.SPEC-003
spec_name: Data Export Archive Generation
spec_slug: data-export-archive-generation
parent_feature: FEAT-24
parent_feature_name: Data Export & Account Deletion
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Automation Spec: Data Export Archive Generation

## Overview

**Name:** Data Export Archive Generation
**ID:** FEAT-24.SPEC-003
**Type:** Automation
**Purpose:** Aggregates every client, project, proposal, invoice, and activity record Nadia owns into a single downloadable archive, retries cleanly on failure, and expires the archive after its limited download window.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion

## Scope and Non-Goals

**In Scope:**
- Aggregating every entity Nadia owns into one archive file on request
- Storing and delivering the file through the large-file storage & delivery capability
- Retrying generation failures without leaving partial or corrupted output
- Advancing the archive through its lifecycle states and expiring it after its download window

**Non-Goals:**
- Requesting the export and displaying its status -- owned by FEAT-24.SPEC-001 (Data Export Screen); this automation only performs and reports on the generation itself.
- Partial or selective export by data type -- excluded per product-features.md's Key Capabilities, which name only a whole-account export; this automation always aggregates the full set of owned records.
- Deciding who may request an export -- owned by FEAT-24.SPEC-007 (Export & Deletion Access Rules); this automation trusts that authorization has already passed by the time its trigger fires.
- Permanently removing the freelancer's underlying data -- owned by FEAT-24.SPEC-004 (Account Deletion Processing); this automation only reads existing data to build a copy, never deletes the source records.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia requests a full export | FEAT-24.SPEC-001 (Data Export Screen) | Fires whenever Request Export (or Request New Export) is tapped, always on successful authorization via FEAT-24.SPEC-007 | Freelancer account reference |
| Archive successfully delivered | FEAT-16.SPEC-007 (Large-File Storage & Delivery Capability) | Fires when the storage & delivery capability reports the download transfer for this archive completed | Archive reference, transfer confirmation |
| Download window elapses unfetched | System (scheduled sweep) | Fires when an archive's download window (platform parameter: `data-export-download-window`) has elapsed since it reached Ready, regardless of whether it was ever downloaded | Archive reference, elapsed time since Ready |

## Processing Logic

1. Receive the export request from FEAT-24.SPEC-001, already authorized via FEAT-24.SPEC-007 (Nadia, her own account only).
2. If an existing Data Export Archive record for this account is in Ready, Downloaded, or Expired state, discard it and any underlying stored file via FEAT-16.SPEC-007 -- only one active archive exists per account at a time.
3. Create a new Data Export Archive record with status Requested and requested-at set to the current time.
4. Aggregate every record Nadia owns into one archive file: Client, Client Contact, Project, Proposal, Milestone, Payment Schedule, Deliverable, Deliverable Version (as version history), Comment, Invoice, Payment, Reminder Log, and Activity Log Entry (feature-overview.md, Referenced Entities table), scoped strictly to this one freelancer account (ASMP-23 isolation).
5. Submit the assembled file to the large-file storage & delivery capability (FEAT-16.SPEC-007) for storage and delivery.
6. On confirmed storage, set the archive's status to Ready and its download link/window to expire at platform parameter: `data-export-download-window` from the moment it reaches Ready.
7. Signal FEAT-24.SPEC-001 to reflect the Ready state.
8. Trigger FEAT-24.SPEC-008 (Export Ready Notification).
9. When FEAT-16.SPEC-007 reports a completed download transfer for this archive, set its status to Downloaded (the download action itself remains available until the archive expires).
10. When the download window elapses (whether the archive is Ready or Downloaded), set its status to Expired and purge its underlying stored file via FEAT-16.SPEC-007; there is no restore path for an expired archive.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Archive Ready | Aggregation and storage succeed | Data Export Archive: Requested -> Ready; download link/window set | FEAT-24.SPEC-001 shows the Ready status card with a Download action; FEAT-24.SPEC-008 fires | FEAT-24.SPEC-001, FEAT-24.SPEC-008 |
| Archive Downloaded | FEAT-16.SPEC-007 reports a completed transfer | Data Export Archive: Ready -> Downloaded | FEAT-24.SPEC-001 shows "Downloaded on {date}," Download action remains available | FEAT-24.SPEC-001 |
| Generation retried after transient failure | Aggregation or storage submission fails mid-process | No partial file persisted; the same Requested archive record retries automatically | FEAT-24.SPEC-001 continues showing "Preparing your export..." | FEAT-24.SPEC-001 |
| Generation failed (retries exhausted) | Every retry up to platform parameter: `data-export-generation-retry-count` fails | Archive record carries no download link; status remains a failed Requested state | FEAT-24.SPEC-001 shows "We couldn't generate your export. Try again." | FEAT-24.SPEC-001 |
| Archive Expired | Download window elapses since Ready, regardless of Downloaded state | Data Export Archive: -> Expired; underlying stored file purged via FEAT-16.SPEC-007 | FEAT-24.SPEC-001 shows "Your last export has expired." | FEAT-24.SPEC-001 |
| No-action (sweep finds nothing due) | Scheduled sweep runs and no archive has reached its window | None | None | -- |

## Data Model

**Reads:** Client, Client Contact, Project, Proposal, Milestone, Payment Schedule, Deliverable, Deliverable Version, Comment, Invoice, Payment, Reminder Log, Activity Log Entry -- every field of each, scoped to the requesting freelancer's own account.
**Creates:** Data Export Archive -- requested-at, status, download link/window.
**Updates:** Data Export Archive -- status transitions Requested -> Ready -> Downloaded -> Expired.
**Deletes:** Data Export Archive's underlying stored file (via FEAT-16.SPEC-007) once superseded by a new request or once Expired; the record itself is not deleted, it simply carries no further download link.

## Business Rules

- Only one active archive exists per account at a time; a new request always supersedes any prior archive rather than queuing behind it (feature-overview.md, Entity-Lifecycle Coverage Matrix).
- Whole-account export only -- no partial or selective aggregation exists (product-features.md, Key Capabilities).
- The export reflects current data at request time, including Invoice and Payment records that would later be subject to legal retention on account deletion (SC-24) -- retention rules apply only to deletion, never to what an export may include.
- XBR-29: an operator support session never triggers, views, or generates a data export; this automation's trigger is exclusively FEAT-24.SPEC-001, which FEAT-24.SPEC-007 confines to Nadia.
- A failed generation never leaves a partial or corrupted archive visible to Nadia (product-features.md, States field) -- retries occur before any failure is surfaced.

## Edge Cases

- **Nadia's account has almost no data (a brand-new account)** -- The archive still generates normally, containing whatever minimal set of records exists; States field: "this always operates on whatever data exists, even a near-empty account."
- **A deliverable file referenced by a Deliverable Version cannot be included at generation time (e.g., storage capability degradation)** -- Generation retries per the failure path; if retries exhaust with this file still unavailable, the whole generation is reported failed rather than producing an incomplete archive, since a partial archive contradicts the "no partial, corrupted output" guarantee.
- **The archive expires at the same moment Nadia taps Download** -- Whichever completes first wins: if the expiry sweep processes first, the Download action fails with "This export has expired. Request a new one."; if the download transfer is already in flight, it is allowed to complete and the archive is marked Downloaded rather than Expired.
- **Concurrent trigger firing (Nadia requests a new export from two open tabs at effectively the same time)** -- Both requests reach step 2 independently; whichever's discard-and-create sequence commits first becomes the active archive, and the second request's discard step then finds and supersedes that first archive in turn, leaving exactly one active archive reflecting the later request. Neither request errors.
- **Trigger fires while a previous generation run is still in flight** -- A second request for the same account interrupts the in-flight generation by proceeding through the same discard step (step 2); the in-flight run's eventual completion is discarded since its archive record no longer exists, avoiding two archives being finalized for one account.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-001 (Data Export Screen) | Triggered by (inbound) | Request Export fires this automation |
| FEAT-24.SPEC-001 (Data Export Screen) | Affects (outbound) | Every outcome updates the status card shown there |
| FEAT-24.SPEC-007 (Export & Deletion Access Rules) | References (inbound) | Authorization gate this automation trusts has already passed |
| FEAT-24.SPEC-008 (Export Ready Notification) | Triggers (outbound) | Fires when the archive reaches Ready |
| FEAT-16.SPEC-007 (Large-File Storage & Delivery Capability) | Triggers (outbound) / Triggered by (inbound) | Performs the actual storage, delivery, and purge of the archive file; its "Archive download delivered" inbound event triggers the Ready -> Downloaded transition |

## Analytics and Success Signals

- **data_export_archive_ready** (generation_duration_bucket) -- N/A -- no success-metrics.md metric is connected to Data Export & Account Deletion; this event is retained because product-features.md's Signals field names "data_export_ready" explicitly as this feature's defined signal.
- **data_export_generation_failed** (failure_point: aggregation / storage_submission; retries_exhausted: yes/no) -- N/A -- same reason; retained so a generation path that never resolves stays observable rather than silent.
- **data_export_archive_expired** (was_downloaded: yes/no) -- N/A -- same reason; retained to distinguish an archive Nadia never came back for from one she already retrieved.

## Acceptance Criteria

**FEAT-24.SPEC-003-AC-01:** Given Nadia taps Request Export on FEAT-24.SPEC-001 with no prior archive, when this automation runs, then it aggregates every owned record into a new archive and, on success, sets the archive to Ready with a download link/window of platform parameter: `data-export-download-window`.

**FEAT-24.SPEC-003-AC-02:** Given the archive reaches Ready, when this outcome applies, then FEAT-24.SPEC-008 fires and FEAT-24.SPEC-001 shows the Download action.

**FEAT-24.SPEC-003-AC-03:** Given the storage & delivery capability reports a completed download transfer for the archive, when this event arrives, then the archive's status advances from Ready to Downloaded and the Download action remains available.

**FEAT-24.SPEC-003-AC-04:** Given aggregation fails transiently on the first attempt, when the failure occurs, then generation retries automatically without exposing a partial archive to Nadia.

**FEAT-24.SPEC-003-AC-05:** Given every retry up to platform parameter: `data-export-generation-retry-count` fails, when the final retry fails, then the archive carries no download link and FEAT-24.SPEC-001 shows "We couldn't generate your export. Try again."

**FEAT-24.SPEC-003-AC-06:** Given an archive is Ready or Downloaded, when its download window (platform parameter: `data-export-download-window`) elapses, then its status becomes Expired and its underlying stored file is purged via FEAT-16.SPEC-007.

**FEAT-24.SPEC-003-AC-07:** Given Nadia requests a new export while a prior archive is Ready, Downloaded, or Expired, when the new request is received, then the prior archive and its file are discarded and a fresh archive begins generating.

**FEAT-24.SPEC-003-AC-08:** Given a freelancer account with almost no data, when an export is requested, then the archive still generates successfully, containing whatever records exist.

**FEAT-24.SPEC-003-AC-09:** Given the archive's download window elapses at the same moment Nadia's download transfer is already in flight, when both occur, then the in-flight transfer is allowed to complete and the archive is marked Downloaded rather than Expired.

**FEAT-24.SPEC-003-AC-10:** Given the archive's download window elapses before Nadia starts a download, when she then attempts to download, then the attempt fails with "This export has expired. Request a new one." and no file transfer occurs.

**FEAT-24.SPEC-003-AC-11:** Given Nadia requests a new export from two open tabs at effectively the same time, when both requests are processed, then exactly one active archive results, reflecting the later request.

**FEAT-24.SPEC-003-AC-12:** Given a generation run is already in flight when a new request for the same account arrives, when the new request is processed, then it supersedes the in-flight run so that only the new request's resulting archive is ever finalized.

**FEAT-24.SPEC-003-AC-13:** Given this automation aggregates data for one freelancer's export, when it runs, then it never includes another freelancer's records.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 6 | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |
