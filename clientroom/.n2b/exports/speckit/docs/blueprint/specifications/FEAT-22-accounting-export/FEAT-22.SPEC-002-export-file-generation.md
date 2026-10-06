---
document_type: spec
spec_type: automation
spec_id: FEAT-22.SPEC-002
spec_name: Export File Generation
spec_slug: export-file-generation
parent_feature: FEAT-22
parent_feature_name: Accounting Export
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Automation Spec: Export File Generation

## Overview

**Name:** Export File Generation
**ID:** FEAT-22.SPEC-002
**Type:** Automation
**Purpose:** Aggregates the selected date range's Invoice and Payment records into a CSV or QuickBooks/Xero-compatible file, hands it to the screen for download, and marks it Downloaded the moment it is first served.
**Parent Feature:** FEAT-22 -- Accounting Export

## Scope and Non-Goals

**In Scope:**
- Reading Invoice and Payment records for the selected date range, scoped strictly to Nadia's own account
- Assembling the export content in the chosen format (CSV or QuickBooks/Xero-compatible)
- Segregating totals by currency and deriving them only from Invoice and Payment records (XBR-18, XBR-22)
- Creating the Accounting Export File record and transitioning its status (Generated -> Downloaded)
- Retrying a failed generation without exposing or corrupting a partial file
- Serving the finished file to Nadia's Download action

**Non-Goals:**
- Deciding whether the selected range and Nadia's authorization are valid to generate at all -- that gate is FEAT-22.SPEC-003's Authorization Rules and Field Validation Rules, which this automation re-checks authoritatively but does not define
- Presenting the Generate/Download controls, progress indicator, or result states to the user -- owned by FEAT-22.SPEC-001 (Accounting Export Screen), which this automation only informs
- Any conversion or currency-summing logic across the range's currencies -- excluded per XBR-18 ("amounts in different currencies are never converted or added together"); this automation groups and reports, it never combines
- Retaining the generated file as an ongoing managed record -- excluded per product-features.md's Domain Entity Inventory ("Managed by: N/A -- a point-in-time generated file, not an ongoing managed record"); each invocation produces a fresh file rather than storing one for later retrieval

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia taps Generate | FEAT-22.SPEC-001 (Accounting Export Screen) | Fires only after the selected date range and Nadia's authorization pass FEAT-22.SPEC-003's rules | Selected date_range_start, date_range_end, chosen format (CSV or QuickBooks/Xero-compatible), Nadia's account identity |
| Nadia taps Download or "Download again" | FEAT-22.SPEC-001 (Accounting Export Screen) | Fires only while an Accounting Export File held for this screen session is in the Generated or Downloaded status; re-download is permitted for as long as the held file exists (until Nadia changes the range/format, leaves the screen, or the session ends) | The generated file reference and its status |

## Processing Logic

1. Receive the validated date range, chosen format, and Nadia's account identity from FEAT-22.SPEC-001 (already passed through FEAT-22.SPEC-003's range and authorization checks).
2. Re-check authoritatively, at the moment processing begins, that the range still falls within the account's actual invoice history and that the requester is Nadia (FEAT-22.SPEC-003) -- a stale request that no longer satisfies either check is refused rather than processed.
3. Read every Invoice belonging to Nadia's account with an issue_date within the selected range.
4. If no such Invoice exists, stop and signal the No Results outcome -- no file is created.
5. Read every Payment associated with those Invoices, including refunds, reversals, and manually recorded payments (XBR-22), regardless of the Payment's own paid_at falling inside or outside the range, since it corrects an in-range Invoice.
6. Group the Invoice and Payment records by currency (each Invoice's currency field).
7. For each currency group, compute that currency's own totals (invoiced amount, paid amount, refunded/reversed amount) from Invoice and Payment records only -- never converting or combining a total from one currency group into another (XBR-18, XBR-22).
8. Assemble the file content in the chosen format: CSV rows per Invoice/Payment with per-currency subtotals, or the equivalent structure for the QuickBooks/Xero-compatible format.
9. Create the Accounting Export File record with date_range_start, date_range_end, format, and status set to Generated.
10. Signal FEAT-22.SPEC-001 that the file is ready; hold the assembled content for the Download trigger.
11. On the Download trigger, serve the held file content to Nadia. If the status is Generated, set it to Downloaded at that moment of serving (the product cannot observe whether the browser later completes or cancels the transfer, so serving is the single defined transition point). If the status is already Downloaded, serve the held content again with no status change.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| File generated | One or more Invoices exist in the range; assembly completes | Accounting Export File created with status Generated | Ready for Download state with confirmation line | FEAT-22.SPEC-001 |
| No results | Zero Invoices exist in the range | No Accounting Export File created | "No invoices found for this date range. Try a different range." | FEAT-22.SPEC-001 |
| Generation failure | Processing fails partway through (data read or assembly error) | No partial or corrupted Accounting Export File is left behind; any in-progress assembly is discarded | Error state: "We couldn't generate your export. Nothing was created or corrupted -- try again." with Retry | FEAT-22.SPEC-001 |
| Stale request refused | The range or authorization no longer passes FEAT-22.SPEC-003's checks at processing time (e.g., account state changed since the screen loaded) | No Accounting Export File created | Same Error state, since this is a variant of generation failure the user experiences identically | FEAT-22.SPEC-001 |
| Download served | Nadia taps Download while the file is in Generated status | Accounting Export File status set to Downloaded | Browser download proceeds; "Downloaded" confirmation text appears | FEAT-22.SPEC-001 |
| Re-download served | Nadia taps "Download again" (or re-taps after a failed or cancelled browser transfer) while the file is in Downloaded status and still held | None -- status stays Downloaded; no second transition | Browser download proceeds again; "Downloaded" text remains | FEAT-22.SPEC-001 |
| Download attempted with no ready file | Download is triggered but no Accounting Export File is in Generated or Downloaded status held for the session (e.g., stale UI after navigating away and back) | None | The screen has already returned to its Empty state per FEAT-22.SPEC-001's edge cases, so this outcome is only reachable through a stale UI element; it is refused silently and the screen re-syncs to Empty | FEAT-22.SPEC-001 |

## Data Model

**Reads:** Invoice -- invoice_number, project, amount, currency, tax_label, tax_rate, total, issue_date, status, for every Invoice in Nadia's account within the selected range. Payment -- invoice reference, amount, method, paid_at, status, recorded_by, for every Payment tied to those Invoices, including Reversed and manually recorded entries.
**Creates:** Accounting Export File -- date_range_start, date_range_end, format, status (set to Generated).
**Updates:** Accounting Export File -- status (Generated -> Downloaded).
**Deletes:** None -- the Accounting Export File is never deleted in-product; it is simply not retained once the session that generated it ends (product-features.md's Domain Entity Inventory: "Managed by: N/A").

## Business Rules

- XBR-18: Financial totals are shown per currency; amounts in different currencies are never converted or added together. This automation enforces that boundary at assembly time (Step 7).
- XBR-22: Financial Dashboard and Accounting Export totals are derived only from Invoice and Payment records, including refunds, reversals, and manually recorded payments -- no other data source ever contributes to the export's content.
- FEAT-22.SPEC-003 owns the authoritative range-bound and authorization checks; this automation re-runs them at processing time rather than trusting the screen's earlier pass, per the standard reject-with-refresh discipline this product applies elsewhere to stale client state.
- A failed generation is retried by generating fresh rather than editing or resuming a partial file (product-features.md, States field) -- there is never a partially written Accounting Export File for Nadia to encounter.
- XBR-29: this automation is never triggered from within a Dana support session -- FEAT-22.SPEC-003's authorization rules ensure the Generate and Download controls that trigger it are never rendered for Dana in the first place.

## Edge Cases

- **The range includes a Refunded or Partially refunded Invoice** -- The Invoice's original total and the corresponding Payment's refunded amount both appear, per XBR-22; the export never nets them into a single adjusted figure that would obscure the record.
- **A Payment was recorded manually (off-platform) rather than processor-confirmed** -- It is included on equal footing with processor-confirmed Payments, per XBR-22's explicit inclusion of manually recorded payments.
- **An Invoice in the range has no successful Payment yet (still Sent or Overdue)** -- The Invoice appears in the export with its invoiced amount; no Payment row is generated for it, since none exists.
- **The range spans three or more currencies** -- Each currency gets its own group and subtotal in the output; the number of currency groups has no upper bound enforced by this automation.
- **Generation fails after some currency groups are already assembled** -- The entire in-progress assembly is discarded (Step 4 of Business Rules discipline); no file reflecting only some currency groups is ever created or offered.
- **Concurrent trigger firing (Nadia has this screen open in two browser tabs and taps Generate in both at effectively the same time, for the same or different ranges)** -- Each Generate runs its own independent instance of this automation; each produces its own Accounting Export File, and neither run reads or is affected by the other's in-progress state, since no persisted entity is shared between them beyond the read-only Invoice/Payment source data.
- **Trigger fires while a previous run is in flight (Nadia taps Generate again before the first completes)** -- Prevented at the source: FEAT-22.SPEC-001 debounces the Generate button while its own request is in progress, so a second automation run for the same screen session cannot start; a run from a different tab or session is treated as the concurrent-trigger-firing case above, not as an in-flight collision.
- **Nadia taps Download, then taps again (immediately, or after the browser transfer failed or was cancelled)** -- The status transitioned to Downloaded at the first serve; every later tap while the file is still held serves the same content again with no further status change and no error, so a failed transfer never strands Nadia without her file. Once the held file is gone (range/format changed, screen left), the request falls under the "Download attempted with no ready file" outcome.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-001 (Accounting Export Screen) | Triggered by (inbound) | Generate and Download actions both fire this automation |
| FEAT-22.SPEC-001 (Accounting Export Screen) | Affects (outbound) | Returns the ready/no-results/failure/downloaded outcomes for the screen to render |
| FEAT-22.SPEC-003 (Export Scope, Authorization & Currency Rules) | References (inbound) | Authoritative range bound, authorization gate, and currency-segregation logic this automation re-checks and applies |
| FEAT-09 (Invoice Generation & Sending) | References (inbound) | Sole source, with Payment, of the Invoice records this automation reads |
| FEAT-10 (Invoice Payment Processing) | References (inbound) | Sole source, with Invoice, of the Payment records this automation reads, including manually recorded payments |

## Analytics and Success Signals

- **export_generated** (date range length in days, format chosen, currency-group count) -- N/A -- no metric in success-metrics.md is connected to Accounting Export (FEAT-22); this event is emitted per product-features.md's Signals field regardless, so the export lifecycle remains observable even though no Stage 2 metric currently tracks it
- **export_downloaded** (time elapsed between generation and first serve; emitted on the first serve only, not on re-downloads) -- N/A -- same reason: no success-metrics.md metric is connected to this feature
- **export_generation_failed** (failure point: read / assembly; retry attempted) -- N/A -- same reason: no success-metrics.md metric is connected to this feature

## Acceptance Criteria

**FEAT-22.SPEC-002-AC-01:** Given Nadia has selected a valid date range containing invoices and taps Generate, when this automation runs, then it reads the matching Invoice and Payment records, assembles the file in the chosen format, creates the Accounting Export File in Generated status, and signals FEAT-22.SPEC-001 to show Ready for Download.

**FEAT-22.SPEC-002-AC-02:** Given Nadia selects a date range with zero Invoices, when Generate fires, then this automation stops without creating a file and signals the No Results outcome.

**FEAT-22.SPEC-002-AC-03:** Given the range includes a Refunded Invoice, when the file is assembled, then both the original invoiced total and the refunded Payment amount appear as separate figures, per XBR-22.

**FEAT-22.SPEC-002-AC-04:** Given the range includes a manually recorded off-platform Payment, when the file is assembled, then that Payment is included on equal footing with processor-confirmed Payments.

**FEAT-22.SPEC-002-AC-05:** Given the selected range spans Invoices in two currencies, when the file is assembled, then each currency's totals are computed and reported separately, with no conversion or summing across them.

**FEAT-22.SPEC-002-AC-06:** Given processing fails partway through assembly, when the failure occurs, then no partial or corrupted Accounting Export File is created, and FEAT-22.SPEC-001 shows its Error state.

**FEAT-22.SPEC-002-AC-07:** Given the range or Nadia's authorization no longer passes FEAT-22.SPEC-003's checks at the moment processing begins, when this automation re-validates, then it refuses the request and FEAT-22.SPEC-001 shows the Error state.

**FEAT-22.SPEC-002-AC-08:** Given an Accounting Export File exists in Generated status, when Nadia taps Download, then this automation serves the file and, at the moment of serving, transitions the status to Downloaded.

**FEAT-22.SPEC-002-AC-09:** Given Nadia has two browser tabs open on this screen and taps Generate in both at effectively the same time, when both automation runs execute, then each produces its own independent Accounting Export File without interfering with the other.

**FEAT-22.SPEC-002-AC-10:** Given a generation is already in flight for Nadia's session, when she taps Generate again before it completes, then FEAT-22.SPEC-001's debounce prevents a second run from starting for that same session.

**FEAT-22.SPEC-002-AC-11:** Given a file is in Downloaded status and still held, when Nadia taps "Download again" (including after a failed or cancelled browser transfer), then this automation serves the same file again with no error and no further status change.

**FEAT-22.SPEC-002-AC-12:** Given a file finishes generating successfully, when the automation signals readiness, then it fires export_generated; given Nadia then downloads it, it fires export_downloaded.

**FEAT-22.SPEC-002-AC-13:** Given no Accounting Export File is held for the session (Nadia changed the range/format or left the screen), when a stale Download element is tapped, then nothing is served, the request is refused silently and the screen re-syncs to Empty.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (generate, download/download again) | 2 |
| Outcome Paths | 7 (generated, no results, failure, stale-refused, downloaded, re-download, download-no-file) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |
