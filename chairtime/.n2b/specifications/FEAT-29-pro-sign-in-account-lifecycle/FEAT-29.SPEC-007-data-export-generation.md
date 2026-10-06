---
document_type: spec
spec_type: automation
spec_id: FEAT-29.SPEC-007
spec_name: Data Export Generation
spec_slug: data-export-generation
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Data Export Generation

## Overview

**Name:** Data Export Generation
**ID:** FEAT-29.SPEC-007
**Type:** Automation
**Purpose:** Assembles the requested spreadsheet-friendly file from the Pro's own Client, Booking, and Deposit Transaction records on request.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Reading the Pro's own Client, Booking, and Deposit Transaction records
- Assembling a spreadsheet-friendly file scoped exactly per FEAT-29.SPEC-013's export-scope rule
- Reporting generation progress and completion (or failure) back to the requesting screen
- Reporting the current export status (no export, in progress, or ready) on request, without starting a new generation

**Non-Goals:**
- Requesting the export and presenting the download -- owned by FEAT-29.SPEC-004 (Data Export Screen); this automation only assembles the file
- Defining what the export includes or excludes -- owned by FEAT-29.SPEC-013 (Account Closure & Retention Rules); this automation implements that scope, it does not decide it
- Importing any data -- excluded per scope-boundaries.md SC-09: no import capability exists in the product definition
- Including the Client entity's private_note field -- excluded per the Feature Breakdown Brief's Data Notes disposition: private_note is Pro-only working notes, not a client or booking record proper

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro requests an export | FEAT-29.SPEC-004 (Data Export Screen) | Pro taps "Request export" with no other export currently generating for this Pro Account | Pro Account reference |
| Screen requests current export status | FEAT-29.SPEC-004 (Data Export Screen) | The screen opens, on every visit (first open or a return after navigating away) | Pro Account reference |

## Processing Logic

**Generation path:**
1. Receive the export request with the requesting Pro Account's reference.
2. Read all Client records belonging to this Pro Account: name, phone, email, booking_history reference (private_note is never read for this purpose).
3. Read all Booking records belonging to this Pro Account: service, start_time, duration, price_agreed, deposit_amount, state, cancellation/reschedule timestamps, source.
4. Read all Deposit Transaction records tied to those Bookings: amount, currency, status, outcome_reason, timestamps.
5. Assemble the three record sets into a single spreadsheet-friendly file, with one sheet or section per entity, using the field names above as column headers.
6. Report the file as ready for download, along with its generation timestamp.

**Status-check path:**
1. Receive the status request with the requesting Pro Account's reference. This path never reads Client, Booking, or Deposit Transaction data and never starts a new generation.
2. Determine the Pro Account's current export state: a generation currently in progress, a previously completed file that has not been superseded, or no export on record.
3. Report that state back to the requesting screen (plus the generation timestamp, when a completed file exists).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Export ready | Assembly completes successfully | The generated file is stored, superseding any previously generated file for this Pro Account | FEAT-29.SPEC-004 shows the "Download" action with the generation timestamp | FEAT-29.SPEC-004 |
| Export empty (no data) | Pro has zero Clients and zero Bookings | A file with headers only and no data rows is stored | FEAT-29.SPEC-004 shows the "Download" action normally -- no special empty-state messaging | FEAT-29.SPEC-004 |
| Export failure | Assembly encounters a processing error | No new file replaces the previous one, if any | FEAT-29.SPEC-004 shows "Couldn't prepare your export. Try again." with a retry action | FEAT-29.SPEC-004 |
| Status reported | Screen requests current export status (any state: none, in progress, or a completed file) | None -- this is a read-only report | FEAT-29.SPEC-004 routes its Loading state to Ready to request, Generating, or Ready to download, matching the reported state exactly | FEAT-29.SPEC-004 |
| Status check failed | The status query itself fails (e.g., a data-store read error) | None -- no generation is started as a side effect of a failed status check | FEAT-29.SPEC-004 shows "Couldn't check your export status. Try again." with a retry action | FEAT-29.SPEC-004 |

## Data Model

**Reads:** Client (name, phone, email -- excluding private_note), Booking (service, start_time, duration, price_agreed, deposit_amount, state, cancellation/reschedule timestamps, source), Deposit Transaction (amount, currency, status, outcome_reason, timestamps) -- all scoped to the requesting Pro Account, per the dependency map's entity relationships (each belongs to exactly one Pro Account). The status-check path reads only this Pro Account's export-file record (in-progress/completed/none plus generation timestamp), never the Client/Booking/Deposit Transaction data itself.
**Creates:** The generated export file, associated with the requesting Pro Account and a generation timestamp.
**Updates:** None to the source entities -- this automation never modifies Client, Booking, or Deposit Transaction data.
**Deletes:** The previously generated export file for this Pro Account, when a new one supersedes it.

## Business Rules

- Export scope is fixed to structured fields of Client, Booking, and Deposit Transaction, excluding Client.private_note (FEAT-29.SPEC-013).
- The export never includes any other Pro Account's data -- scoping is by Pro Account reference on every read, with no cross-account query path (dependency map: "No record is ever shared between two Pro Accounts").
- Generation is asynchronous relative to the requesting screen: the Pro may navigate away and the export continues; only one export generates at a time per Pro Account.
- A newly requested export always supersedes a previously completed one -- there is no archive of past exports.
- The status-check path is read-only and idempotent -- it never creates, modifies, or deletes the export file, and never starts a generation; only the "Pro requests an export" trigger can start one.

## Edge Cases

- **Pro has years of accumulated history at the upper end of the stated scale (roughly 100-500 clients, several years of bookings)** -- Generation completes reliably at this volume (ASMP-22), showing progress rather than a blank wait for however long assembly takes.
- **A booking is created or changed while generation is running** -- The in-progress export reflects the data as read at the moment each entity was read; a change arriving mid-generation is not guaranteed to appear in that same file and is captured only by a subsequently requested export.
- **Concurrent trigger firing -- the Pro requests an export from two open sessions (e.g., two signed-in devices) at nearly the same time** -- Only one generation runs per Pro Account; the second request is treated as a no-op while the first is in flight (per FEAT-29.SPEC-004's Edge Cases), so no duplicate generation occurs and no conflicting file is produced.
- **Trigger fires while a previous run is in flight** -- A second request for the same Pro Account while generation is already running does not start a new run; it is ignored until the in-flight run completes, at which point a genuinely new request may be made.
- **Generation fails partway through (e.g., a read of one entity succeeds, another fails)** -- The partial result is discarded entirely; no partial file is ever presented as ready. The failure outcome applies and the previous completed file (if any) remains available for download until a new request succeeds.
- **The screen requests status while a generation is mid-flight** -- The status check reports "in progress" without interfering with or restarting that generation; the status-check path never mutates the export-file record.
- **The screen requests status repeatedly (e.g., the Pro reopens the screen several times in a row)** -- Each status request is independently read-only and idempotent; repeated checks never start a generation and never affect one already running.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-004 (Data Export Screen) | Triggered by (inbound) | "Request export" starts this automation |
| FEAT-29.SPEC-004 (Data Export Screen) | Triggered by (inbound) | On every screen open, the status-check path reports current export status to resolve the screen's Loading state |
| FEAT-29.SPEC-004 (Data Export Screen) | Affects (outbound) | Reports progress, ready-to-download, and failure states |
| FEAT-29.SPEC-013 (Account Closure & Retention Rules) | References (inbound) | Defines the export's scope, which this automation implements |

## Analytics and Success Signals

- **data_export_generation_completed** (record_counts: client_count, booking_count) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list so export reliability remains observable
- **data_export_generation_failed** (-- no properties beyond the failure itself) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so export failures remain observable rather than silent

## Acceptance Criteria

**FEAT-29.SPEC-007-AC-01:** Given Talia requests an export with existing clients and bookings, when generation completes, then a file is produced containing her Client, Booking, and Deposit Transaction records, scoped to her own Pro Account.

**FEAT-29.SPEC-007-AC-02:** Given Talia's export is generated, when its contents are inspected, then no client's private_note field appears anywhere in the file.

**FEAT-29.SPEC-007-AC-03:** Given Talia has zero clients and zero bookings, when she requests an export, then generation completes successfully producing a file with headers only and no data rows.

**FEAT-29.SPEC-007-AC-04:** Given Talia has several years of accumulated history at the upper end of the product's stated scale, when she requests an export, then generation completes reliably, showing progress rather than an indefinite blank wait.

**FEAT-29.SPEC-007-AC-05:** Given a new booking is created while a previously requested export is still generating, when generation completes, then the new booking is not guaranteed to appear in that file, and a subsequently requested export captures it.

**FEAT-29.SPEC-007-AC-06:** Given Talia requests a second export from another signed-in device while the first is still generating, when the second request arrives, then it is ignored until the first completes -- no duplicate generation runs.

**FEAT-29.SPEC-007-AC-07:** Given a processing error occurs partway through assembly, when the failure is detected, then no partial file is produced and FEAT-29.SPEC-004 shows the failure state with a retry action.

**FEAT-29.SPEC-007-AC-08:** Given Talia already has a completed export file, when she requests a new one that completes successfully, then the new file supersedes and replaces the previous one.

**FEAT-29.SPEC-007-AC-09:** Given Talia's export reads only her own Pro Account's records, when generation runs, then no other Pro's Client, Booking, or Deposit Transaction data is included under any condition.

**FEAT-29.SPEC-007-AC-10:** Given Talia's export completes, when FEAT-29.SPEC-004 checks its state, then it reports the generation timestamp alongside the ready-to-download state.

**FEAT-29.SPEC-007-AC-11:** Given Talia opens the Data Export Screen when no export has ever been requested, when the screen's status check runs, then this automation reports no export exists, no generation is started, and FEAT-29.SPEC-004 resolves to Ready to request.

**FEAT-29.SPEC-007-AC-12:** Given a status check for Talia's Pro Account fails (e.g., a data-store read error), when FEAT-29.SPEC-004 requests current status, then this automation reports the status-check failure, starts no generation as a side effect, and FEAT-29.SPEC-004 shows its Error state with a retry action.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |
