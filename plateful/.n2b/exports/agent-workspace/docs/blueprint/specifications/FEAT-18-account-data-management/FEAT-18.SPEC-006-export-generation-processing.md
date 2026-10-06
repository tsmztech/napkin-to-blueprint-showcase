---
document_type: spec
spec_type: automation
spec_id: FEAT-18.SPEC-006
spec_name: Export Generation Processing
spec_slug: export-generation-processing
parent_feature: FEAT-18
parent_feature_name: Account & Data Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Export Generation Processing

## Overview

**Name:** Export Generation Processing
**ID:** FEAT-18.SPEC-006
**Type:** Automation
**Purpose:** Compiles all of a household's records into a readable export file and makes it available for download.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Compiling the household's plans, ratings, lists, dietary rules, and settings into one readable export file
- Retrying on failure and reporting exhausted retries clearly
- Making the completed file available for download and signaling readiness for the export-ready notification

**Non-Goals:**
- Deciding when an export may be requested -- the rate-limit condition is governed by FEAT-18.SPEC-010 (Account & Data Validation Rules) and enforced before this automation is triggered by FEAT-18.SPEC-001
- Delivering the export-ready confirmation -- owned by FEAT-18.SPEC-013 (Export Ready Notification), which this automation's success outcome triggers
- Deleting any data -- excluded per product-features.md: export is read-only compilation; it never removes or modifies the source records it reads

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Export requested | FEAT-18.SPEC-001 (Export Household Data) | Fires when Maya taps Request Export and FEAT-18.SPEC-010's rate-limit check passes | Household reference, request timestamp |

## Processing Logic

1. Receive the export request for the household (household reference, request timestamp).
2. Read every record belonging to the household across its owning features: Household settings, every Member Profile (including kid profiles' minimal data), every Dietary Rule, every Weekly Plan and its Planned Meals, the current and archived Grocery Lists, every Rating, and Support Request history.
3. Compile the read records into one readable export file, organized by record type, in a format a household member can open and read without specialized software.
4. Mark the export as ready and record its ready date once compilation completes.
5. Make the completed file available for download from FEAT-18.SPEC-001 and retain it as a previous export entry.
6. Signal FEAT-18.SPEC-013 (Export Ready Notification) that the export is ready.
7. If compilation fails at any step, retry automatically up to the automation's retry limit before reporting failure.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|---------------|------------------|--------------------|
| Export ready | Compilation completes successfully | A new export file record is created for the household, marked ready with its ready date | FEAT-18.SPEC-001 shows "Download Export"; FEAT-18.SPEC-013 delivers the ready confirmation | FEAT-18.SPEC-001, FEAT-18.SPEC-013, FEAT-18.SPEC-012 |
| Retry in progress | Compilation fails on an attempt but retries remain | No export file record is finalized yet | FEAT-18.SPEC-001 continues showing "Compiling your export..." -- retries are not surfaced individually as a distinct visible state | FEAT-18.SPEC-001 |
| Export failed | Compilation fails and the retry limit is exhausted | No export file record is created for this request | FEAT-18.SPEC-001 shows "We couldn't finish your export. It's been retried automatically -- if this keeps happening, contact support." | FEAT-18.SPEC-001, FEAT-18.SPEC-005 |

## Data Model

**Reads:** Household -- all fields; Member Profile -- all fields for every household member; Dietary Rule -- all fields; Weekly Plan and Planned Meal -- all fields; Grocery List and Grocery List Item -- all fields; Rating -- all fields; Support Request -- all fields.
**Creates:** An export file record for the household, with its ready date and download reference.
**Updates:** None -- compilation is read-only against the source records.
**Deletes:** None.

## Business Rules

- Export generation is asynchronous and always shows progress rather than an indefinite wait, per product-features.md's States field.
- A failed compilation is retried automatically; only after retries are exhausted is the failure reported to the household, per product-features.md's States field.
- Each successful compilation produces exactly one export file record, retained as a previous export the household can re-download.
- Export generation never modifies or removes any source record it reads (product-features.md, Data Notes: export is derived data, never a mutation).

## Edge Cases

- **A household's data volume is unusually large (years of accumulated plans and ratings)** -- Compilation still runs to completion; the household sees the same "Compiling your export..." progress state for as long as it takes, per the Non-Functional Notes' expectation that generation stays reasonably fast even for a household with years of history.
- **A member's profile is removed (FEAT-18.SPEC-007) while an export is compiling** -- The export reflects the household's data as of the moment compilation began; a member removed mid-compilation may or may not appear in the resulting file depending on exactly when their records were read, and this is an accepted characteristic of a point-in-time export rather than a defect.
- **Concurrent trigger firing (two export requests for the same household at effectively the same time)** -- Cannot occur in practice: FEAT-18.SPEC-010's rate-limit check and FEAT-18.SPEC-001's disabled-while-compiling state together ensure only one export request per household is accepted while a prior one is in flight.
- **Trigger fires while a previous run is in flight** -- A second request for the same household is rejected before reaching this automation, per FEAT-18.SPEC-001's Compiling state disabling further requests; this automation therefore never runs two compilations for the same household concurrently.
- **Household is deleted while an export is compiling** -- Household deletion (FEAT-18.SPEC-008) supersedes the in-flight export per the dependency map's Contention note for Household; the compilation is cancelled and no export file is finalized or delivered.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-001 (Export Household Data) | Triggered by (inbound) | Request Export starts this automation |
| FEAT-18.SPEC-001 (Export Household Data) | Affects (outbound) | Progress, ready, and error states surface here |
| FEAT-18.SPEC-010 (Account & Data Validation Rules) | References (inbound) | Rate-limit check runs before this automation is triggered |
| FEAT-18.SPEC-013 (Export Ready Notification) | Triggers (outbound) | The export-ready outcome fires this notification |
| FEAT-18.SPEC-008 (Household Deletion Processing) | References (inbound) | Household deletion cancels an in-flight compilation |

## Analytics and Success Signals

- **data_export_completed** (record_count_by_type) -- N/A -- no Stage 2 success metric measures export completion; retained per product-features.md's Signals field (data_export_completed) as the operational record of this lifecycle action
- **data_export_failed** (retries_attempted) -- N/A -- no Stage 2 success metric measures export failures; retained to observe whether the automated-retry-then-report behavior is ever actually exercised

## Acceptance Criteria

**FEAT-18.SPEC-006-AC-01:** Given Maya's export request passes the rate-limit check, when this automation starts, then it reads every record type belonging to her household and begins compiling the export file.

**FEAT-18.SPEC-006-AC-02:** Given compilation completes successfully, when the export file is finalized, then FEAT-18.SPEC-001 shows "Download Export" and FEAT-18.SPEC-013 is triggered.

**FEAT-18.SPEC-006-AC-03:** Given compilation fails on its first attempt, when a retry remains, then the automation retries automatically without reporting failure to Maya.

**FEAT-18.SPEC-006-AC-04:** Given compilation fails and retries are exhausted, when the final attempt fails, then FEAT-18.SPEC-001 shows "We couldn't finish your export. It's been retried automatically -- if this keeps happening, contact support."

**FEAT-18.SPEC-006-AC-05:** Given a completed export file exists for a household, when Maya requests a new export later, then a new, independent export file is created and the prior one remains available as a previous export.

**FEAT-18.SPEC-006-AC-06:** Given a household is deleted while its export is compiling, when household deletion processing (FEAT-18.SPEC-008) completes its cascade, then the in-flight compilation is cancelled and no export file is finalized.

**FEAT-18.SPEC-006-AC-07:** Given no export request currently exists for a household, when a rate-limited request attempt is rejected by FEAT-18.SPEC-010, then this automation is never triggered.

**FEAT-18.SPEC-006-AC-08:** Given a household's data spans several years of plans and ratings, when this automation compiles the export, then it produces one complete export file covering the full history, with progress shown throughout.

**FEAT-18.SPEC-006-AC-09:** Given Maya's household already has an export compiling, when a second export request for the same household is attempted, then it never reaches this automation, since FEAT-18.SPEC-001's disabled Compiling state prevents the second trigger.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (ready, retry in progress, failed) | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
