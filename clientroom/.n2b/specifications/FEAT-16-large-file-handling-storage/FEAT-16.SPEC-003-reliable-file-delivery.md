---
document_type: spec
spec_type: automation
spec_id: FEAT-16.SPEC-003
spec_name: Reliable File Delivery
spec_slug: reliable-file-delivery
parent_feature: FEAT-16
parent_feature_name: Large File Handling & Storage
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

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
