---
document_type: spec
spec_type: screen
spec_id: FEAT-22.SPEC-001
spec_name: Accounting Export Screen
spec_slug: accounting-export-screen
parent_feature: FEAT-22
parent_feature_name: Accounting Export
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

# Screen Spec: Accounting Export Screen

## Overview

**Name:** Accounting Export Screen
**ID:** FEAT-22.SPEC-001
**Type:** Screen
**Purpose:** Nadia selects a date range and a file format, generates a bookkeeping export of her invoices and payments, and downloads it; Dana sees the same screen read-only with no generate or download controls.
**Parent Feature:** FEAT-22 -- Accounting Export

## Scope and Non-Goals

**In Scope:**
- Date-range selection, bounded to the account's actual invoice history
- Format choice between CSV and a QuickBooks/Xero-compatible file
- Triggering generation and showing its progress
- Offering the Download action once a file is ready
- The plain "nothing to export" result when the selected range has no invoices
- Dana's read-only rendering of this same screen during a support session

**Non-Goals:**
- Deriving or aggregating the export's actual content (which Invoice and Payment records go in, in what format) -- handled by FEAT-22.SPEC-002 (Export File Generation)
- The date-range bound, the Nadia-only vs. Dana-view-only gate, and the per-currency and totals-source rules -- defined once in FEAT-22.SPEC-003 (Export Scope, Authorization & Currency Rules) and referenced here, not restated
- Live, two-way accounting sync of any kind -- excluded per scope-boundaries.md (SC-07): "v1 provides an export file (CSV or a QuickBooks/Xero-compatible format). No live sync"; this screen's Generate-and-Download pair is the entirety of the product's accounting integration
- Retaining or listing previously generated export files -- excluded per product-features.md's Domain Entity Inventory, which marks the Accounting Export File entity "Managed by: N/A -- a point-in-time generated file, not an ongoing managed record"; there is never more than the single freshly-generated file to show, so this screen has no history or list view

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12.SPEC-001 (Dashboard Overview) | Nadia navigates here to generate the month's export as part of her Month-End Financial Review journey | None -- screen starts with no range or format selected |
| FEAT-31 (Operator Support Access), during a support session | Dana opens this screen while working through Nadia's own screens in a read-only support session | Dana's session context; no export state carries over between sessions |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Select a date range, choose a format, generate, and download (her own data only) | -- |
| Dana (Support Operator) | Full screen, rendered exactly as Nadia would see it, plus a permanent "Read-only support session" banner | None -- Generate and Download controls are not shown (FEAT-22.SPEC-003; XBR-29) | Attempting to reach the Generate or Download action through a stale or cached view of the screen is refused with "Support sessions can't generate or download exports." |
| Owen (Client Primary Contact) | No -- this screen has no client-portal surface | No | This screen is not reachable from the client portal; no navigation path into it exists for a client contact. If a stale link is followed, Owen lands on his own portal home (FEAT-05.SPEC-003, Portal Home) with no error surfaced, since the destination never existed for his role |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- no navigation path exists; a stale link returns Priya to her own portal home (FEAT-05.SPEC-003, Portal Home) |
| Unauthenticated | No | No | Redirected to Nadia's sign-in; after signing in, the person lands on the Freelancer Dashboard (FEAT-12), not this screen, since no context was carried through an unauthenticated visit |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any date range or format already selected is discarded, since nothing is preserved once the session expires; the freelancer returns here fresh after signing in again |

## Layout and Content

**Header:** Screen title "Accounting Export" with a back arrow (returns to FEAT-12.SPEC-001, Dashboard Overview). During a Dana session, the header also carries the permanent "Read-only support session" banner.

**Body:** A single-column export panel with, in order:
- A date-range picker with two fields, "From" and "To," bounded to the account's actual invoice history per FEAT-22.SPEC-003 -- dates outside that bound cannot be selected
- A format choice presented as two options: "CSV" and "QuickBooks/Xero-compatible file" (exactly one selectable at a time)
- A "Generate" action button, below the format choice
- Once a file is ready, a "Download" action button appears in the same position the Generate button occupied, alongside a plain confirmation line naming the covered range and chosen format (for example, "Your February 1 - February 28 CSV export is ready.")
- When the selected range spans more than one currency, an informational line appears above the Generate button: "This range includes invoices in more than one currency. Your export shows totals for each currency separately (per FEAT-22.SPEC-003) -- amounts are never combined." This line is display-only.

For Dana's read-only rendering, the same body layout appears with the Generate and Download buttons omitted entirely -- not disabled, not shown -- consistent with the Access and Visibility table.

**Footer:** None -- all actions live in the body panel.

### Responsive Behavior

- **Compact breakpoint:** The date-range picker's "From" and "To" fields stack vertically; the format choice stacks below them; Generate/Download remain full-width beneath the format choice.
- **Medium size class and above:** The "From" and "To" fields sit side by side; the format choice sits below them as two side-by-side options; Generate/Download remain in the same relative position, capped at a consistent platform-wide panel width and horizontally centered. No structural change beyond this reflow.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12.SPEC-001 (Dashboard Overview) | Screen closes | Animated transition back to the dashboard |
| "From" date field | Select a date | Captures the range start, checked against FEAT-22.SPEC-003's bound | Field shows the chosen date | Dates before the account's earliest invoice are not selectable in the picker |
| "To" date field | Select a date | Captures the range end, checked against FEAT-22.SPEC-003's bound | Field shows the chosen date | Dates after today, or before the selected "From" date, are not selectable |
| Format choice (CSV / QuickBooks-Xero) | Select | Sets the chosen output format | Selected option is visually marked | Selected format name displayed |
| Generate button (Nadia only) | Tap | 1. Validate the range and Nadia's authorization via FEAT-22.SPEC-003. 2. On pass, trigger FEAT-22.SPEC-002 (Export File Generation). | Button enters a loading state; for large ranges a progress indicator appears | Progress text: "Generating your export..." |
| Generate button (while generating) | Tap | No action -- debounced | None | Button remains in its loading state |
| Download button (Nadia only) | Tap while in Ready for Download | Trigger FEAT-22.SPEC-002 to serve the file; the file's status moves Generated -> Downloaded at the moment the file is served (FEAT-22.SPEC-002 Step 11) | Screen moves to the Downloaded state; the button relabels to "Download again" and stays visible | Standard file-download behavior for the browser; confirmation text "Downloaded" appears next to the button |
| "Download again" button (Nadia only) | Tap while in Downloaded | Trigger FEAT-22.SPEC-002 to serve the same held file again (no status change; the file is already Downloaded) | Screen stays in Downloaded; button stays visible | Standard file-download behavior for the browser again; "Downloaded" text remains |
| Retry action on the error banner (Nadia only) | Tap while in Error | Re-runs Generate with the same range and format Nadia last submitted (they remain selected in the fields): 1. re-validate the range and Nadia's authorization via FEAT-22.SPEC-003. 2. On pass, trigger FEAT-22.SPEC-002 afresh. If validation fails, the SPEC-003 message for that rule appears inline next to the offending field and the screen returns to Configuring | On pass: screen leaves Error and enters Generating (banner cleared, Generate-style loading state); on validation failure: Configuring | Progress text: "Generating your export..."; a second tap on Retry while Generating is ignored (debounced, same as Generate) |

### Accessibility Notes

- **Focus order:** Back arrow -> "From" field -> "To" field -> format choice -> Generate (or Download, once the file is ready).
- **Progress announcement:** When generation begins, "Generating your export..." is announced to assistive technology; when the file becomes ready, the readiness confirmation line and the Download button's appearance are announced.
- **Empty-result announcement:** The "nothing to export" message (States: No Results) is announced to assistive technology the moment it appears, since it replaces the expected Download button.
- **Keyboard alternatives:** Every action on this screen (date selection, format choice, Generate, Download) is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Date-range picker and format choice both unset; Generate disabled until both are set | Screen first opens | Nadia sets a range and a format |
| Configuring | Range and format selected; Generate enabled | A valid range and format are both set | Nadia taps Generate |
| Generating | Generate button shows a loading state; for a range spanning more than roughly a billing month, a visible progress indicator accompanies it | Generate tapped, validation passed | FEAT-22.SPEC-002 returns a result (ready, no-results, or failure) |
| Ready for Download | Confirmation line naming the range and format; Download button shown in place of Generate | FEAT-22.SPEC-002 signals the file is generated | Nadia taps Download (-> Downloaded), or she changes the range/format (-> Configuring), or leaves the screen |
| Downloaded | Same confirmation line; the text "Downloaded" beside the button; the button relabeled "Download again" and still shown in the Generate/Download position | Nadia taps Download and FEAT-22.SPEC-002 serves the file (status Generated -> Downloaded at the moment of serving) | Nadia changes the range or format (-> Configuring; the held file is discarded), or leaves the screen (nothing retained). Tapping "Download again" keeps this state |
| No Results | Plain message: "No invoices found for this date range. Try a different range." No Download button appears. | FEAT-22.SPEC-002 finds no invoices in the selected range | Nadia changes the range or format and generates again |
| Error | Error banner: "We couldn't generate your export. Nothing was created or corrupted -- try again." with a Retry action; the last range and format stay selected | FEAT-22.SPEC-002 reports a generation failure or a refused stale request | Nadia taps Retry (-> Generating, or Configuring if re-validation fails), or changes the range/format (-> Configuring) |
| Offline/Degraded | N/A -- generating and downloading an export both require connectivity; the screen shows the platform's standard "You're offline" indicator and disables Generate and Download until connectivity returns, with nothing queued for later since a stale range could no longer reflect the account's current invoice history | Connectivity lost while this screen is open | Connectivity restored |

## Validation Rules

Validation governed by FEAT-22.SPEC-003 (Export Scope, Authorization & Currency Rules). See that spec for the date-range bound, the Nadia-only vs. Dana-view-only authorization gate, and the per-currency and totals-source rules. This screen applies the range bound at date-selection time (dates outside the bound are simply not selectable in the picker) and re-checks both the range and Nadia's authorization on Generate.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-12.SPEC-001 (Dashboard Overview) | FEAT-12 (Freelancer Financial Dashboard) |
| Successful download | Stays on this screen -- no navigation | -- |
| Session expiry | Sign-in screen, then FEAT-12.SPEC-001 after re-authentication | FEAT-05 (Client Portal Access) is not applicable here; re-authentication is on Nadia's own account |

## Data Model

**Creates:** None directly -- Generate triggers FEAT-22.SPEC-002, which creates the Accounting Export File (date_range_start, date_range_end, format, status).
**Reads:** Accounting Export File -- status, to determine whether to show Generating, Ready for Download, Downloaded, No Results, or Error; date_range_start/date_range_end and format, to render the readiness confirmation line. The account's earliest invoice issue_date (Invoice, read-only via FEAT-22.SPEC-003), to bound the date-range picker.
**Updates:** None directly -- Download triggers FEAT-22.SPEC-002, which updates the Accounting Export File's status from Generated to Downloaded.
**Deletes:** None -- the Accounting Export File is never deleted in-product (product-features.md's Domain Entity Inventory: "Managed by: N/A").

## Business Rules

- FEAT-22.SPEC-003 governs who may generate and download versus view only -- this screen renders its Generate/Download visibility exactly per that spec's Authorization Rules table.
- FEAT-22.SPEC-003 bounds the selectable date range to the account's actual invoice history; the picker never offers a date outside that bound.
- XBR-18 and XBR-22: totals in the generated file are derived only from Invoice and Payment records and are never converted or summed across currencies; the informational line shown when a range spans multiple currencies reflects this rule without restating it.
- XBR-29: a Dana support session excludes downloads and exports entirely -- Dana's rendering of this screen never exposes a working Generate or Download control, regardless of how the session was reached.

## Edge Cases

- **Nadia taps Generate twice in rapid succession** -- The second tap is ignored while the first generation is in progress (Generate button is in its loading state and debounced).
- **The selected range has no invoices** -- FEAT-22.SPEC-002 returns the No Results outcome; the screen shows "No invoices found for this date range. Try a different range." rather than producing a broken empty file (product-features.md, States field).
- **Generation fails partway through** -- FEAT-22.SPEC-002 retries safely; the screen shows the Error state with Retry, and no partial or corrupted file is ever offered for download (product-features.md, States field).
- **Nadia navigates away while a file is Ready for Download or Downloaded** -- No warning is shown and nothing is preserved; since the entity is never retained, returning to this screen later starts fresh at the Empty state and she must generate again.
- **The browser download fails or is cancelled after the file was served** -- The file is already Downloaded, but the screen stays in Downloaded with "Download again" available; Nadia taps it to receive the same held file without regenerating. Nothing is lost unless she leaves the screen or changes the range/format.
- **Nadia changes the date range or format after a file is already Ready for Download or Downloaded** -- The existing readiness state is discarded immediately; the screen returns to Configuring for the new selection, and the earlier file (not retained) is not affected by this navigation.
- **Dana's support session ends while she is viewing this screen** -- The screen becomes fully inaccessible per FEAT-31's session rules, not merely a loss of the (already absent) Generate/Download controls; Dana is returned to the session queue.
- **Concurrent-edit conflict** -- N/A -- the Accounting Export File is a single-feature, point-in-time generated entity with no shared-entity Contention note in the Feature Dependency Map (it is explicitly excluded from that map's Shared Data Entities list); Nadia is the only writer and each generation is independent, so no concurrent-edit conflict scenario exists for this screen to resolve.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-002 (Export File Generation) | Triggers (outbound) | Generate and Download actions both trigger this automation |
| FEAT-22.SPEC-003 (Export Scope, Authorization & Currency Rules) | References (inbound) | Date-range bound, Nadia-only vs. Dana-view-only gate, and per-currency/totals rules applied on this screen |
| FEAT-12.SPEC-001 (Dashboard Overview) | Navigation (inbound) | Nadia arrives here from the dashboard to generate the month's export |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| export_empty_result_shown | selected date range length (days), format chosen | The selected range returns zero invoices and the No Results state is shown | N/A -- no metric in success-metrics.md is connected to Accounting Export (FEAT-22); this bookkeeping capability sits outside that document's 21 tracked metrics, all of which are scoped to other features |

## Acceptance Criteria

**FEAT-22.SPEC-001-AC-01:** Given Nadia is on the Accounting Export Screen with no range or format selected, when the screen loads, then it shows the Empty state with Generate disabled.

**FEAT-22.SPEC-001-AC-02:** Given Nadia selects a "From" and "To" date both within her account's invoice history and chooses CSV, when she taps Generate, then the screen enters the Generating state and shows "Generating your export...".

**FEAT-22.SPEC-001-AC-03:** Given Nadia's export finishes generating successfully, when FEAT-22.SPEC-002 signals readiness, then the screen shows the Ready for Download state with a confirmation line naming the range and format, and a Download button in place of Generate.

**FEAT-22.SPEC-001-AC-04:** Given Nadia is on the Ready for Download state, when she taps Download, then FEAT-22.SPEC-002 serves the file, the browser's standard download behavior occurs, the screen enters the Downloaded state with "Downloaded" text, and the button relabels to "Download again".

**FEAT-22.SPEC-001-AC-05:** Given Nadia selects a date range that contains no invoices, when she taps Generate, then the screen shows "No invoices found for this date range. Try a different range." and fires export_empty_result_shown.

**FEAT-22.SPEC-001-AC-06:** Given Nadia taps Generate and the generation fails partway through, when the failure is reported, then the screen shows "We couldn't generate your export. Nothing was created or corrupted -- try again." with a Retry action, and no partial file is offered.

**FEAT-22.SPEC-001-AC-07:** Given Nadia taps Generate while a previous generation is already in progress, when the second tap occurs, then it is ignored and the button remains in its loading state.

**FEAT-22.SPEC-001-AC-08:** Given Nadia attempts to select a date before her account's earliest invoice, when she opens the "From" picker, then that date is not selectable.

**FEAT-22.SPEC-001-AC-09:** Given Nadia selects a range that spans invoices in two currencies, when the range is set, then the informational line about per-currency totals appears above Generate.

**FEAT-22.SPEC-001-AC-10:** Given Dana (Support Operator) opens this screen during a read-only support session, when the screen renders, then it shows the full layout with the "Read-only support session" banner and no Generate or Download controls.

**FEAT-22.SPEC-001-AC-11:** Given Dana is viewing this screen in a support session, when a stale or cached view exposes a Generate or Download control, then attempting it is refused with "Support sessions can't generate or download exports."

**FEAT-22.SPEC-001-AC-12:** Given Owen (Client Primary Contact) is signed in to his portal, when he looks for any path to this screen, then none exists -- the screen is not reachable from the client portal.

**FEAT-22.SPEC-001-AC-13:** Given an unauthenticated visitor reaches this screen's address directly, when the page loads, then they are redirected to Nadia's sign-in, and after signing in they land on FEAT-12.SPEC-001, not this screen.

**FEAT-22.SPEC-001-AC-14:** Given Nadia's session expires while she has a range and format selected but has not generated, when she next interacts with the screen, then the dialog "Your session has expired. Sign in to continue." appears and her selections are discarded.

**FEAT-22.SPEC-001-AC-15:** Given Nadia loses connectivity while this screen is open, when she attempts to generate or download, then the platform's standard "You're offline" indicator appears and both actions are disabled until connectivity returns.

**FEAT-22.SPEC-001-AC-16:** Given Nadia is on the Error state after a failed generation with a range and format still selected, when she taps Retry, then the same range and format are re-validated via FEAT-22.SPEC-003 and, on pass, the screen enters Generating with "Generating your export..."; if re-validation fails, the SPEC-003 message appears next to the offending field and the screen returns to Configuring.

**FEAT-22.SPEC-001-AC-17:** Given Nadia has tapped Download and the screen is in the Downloaded state, when the browser download fails or she taps "Download again", then FEAT-22.SPEC-002 serves the same file again, the screen stays in Downloaded with "Download again" visible, and no regeneration is required.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 (back, from, to, format, generate, generate-while-generating, download, download again, retry) | 9 |
| States | 8 (empty, configuring, generating, ready, downloaded, no results, error, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 8 | 8 |
