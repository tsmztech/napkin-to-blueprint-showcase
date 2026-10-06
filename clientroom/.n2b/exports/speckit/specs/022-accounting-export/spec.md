# Feature Specification: Accounting Export

**Blueprint feature:** FEAT-22
**Priority tier:** Important
**Build order:** 022 of 33
**Depends on:** FEAT-09, FEAT-10
**Blueprint source:** `docs/blueprint/specifications/FEAT-22-accounting-export/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Accounting Export Screen (Priority: P2)

Nadia selects a date range and a file format, generates a bookkeeping export of her invoices and payments, and downloads it; Dana sees the same screen read-only with no generate or download controls.

**Acceptance Scenarios:**

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

### User Story 2 - Export File Generation (Priority: P2)

Aggregates the selected date range's Invoice and Payment records into a CSV or QuickBooks/Xero-compatible file, hands it to the screen for download, and marks it Downloaded the moment it is first served.

**Acceptance Scenarios:**

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

### User Story 3 - Export Scope, Authorization & Currency Rules (Priority: P2)

Governs who may generate and download versus view only, bounds the date range to the account's actual invoice history, and enforces that totals are derived only from Invoice/Payment records and never converted or summed across currencies.

**Acceptance Scenarios:**

**FEAT-22.SPEC-003-AC-01:** Given Nadia opens the "From" date picker, when the account's earliest Invoice issue_date is January 5, then no date before January 5 is selectable.

**FEAT-22.SPEC-003-AC-02:** Given Nadia selects an end date after today's date through a stale UI, when Generate is pressed, then the error "End date cannot be in the future." is shown and generation does not proceed.

**FEAT-22.SPEC-003-AC-03:** Given Nadia selects an end date before her chosen start date, when Generate is pressed, then the error "End date must be on or after the start date." is shown and generation does not proceed.

**FEAT-22.SPEC-003-AC-04:** Given Nadia has not chosen a format, when she attempts to press Generate, then the error "Choose a file format to continue." is shown and generation does not proceed.

**FEAT-22.SPEC-003-AC-05:** Given Nadia's account has no Invoices at all, when she selects any date range and presses Generate, then no validation error is shown and FEAT-22.SPEC-002 returns its No Results outcome.

**FEAT-22.SPEC-003-AC-06:** Given Nadia selects a start date exactly equal to her account's earliest Invoice issue_date, when she presses Generate, then the range is accepted as valid.

**FEAT-22.SPEC-003-AC-07:** Given Nadia selects an end date exactly equal to today, when she presses Generate, then the range is accepted as valid.

**FEAT-22.SPEC-003-AC-08:** Given Nadia is signed in to her own account, when she opens the Accounting Export Screen, then she can view it and both Generate and Download are available to her.

**FEAT-22.SPEC-003-AC-09:** Given Dana opens a logged, read-only support session on Nadia's account, when she opens the Accounting Export Screen inside that session, then she can view it but no Generate or Download control is rendered.

**FEAT-22.SPEC-003-AC-10:** Given Dana is viewing this screen in a support session, when she attempts Generate or Download through any stale UI element, then the attempt is refused with "Support sessions can't generate or download exports."

**FEAT-22.SPEC-003-AC-11:** Given Owen (Client Primary Contact) is signed in to his portal, when he looks for a way to reach this screen, then no navigation path exists.

**FEAT-22.SPEC-003-AC-12:** Given Priya (Client Reviewer Contact) is signed in to her portal, when she looks for a way to reach this screen, then no navigation path exists.

**FEAT-22.SPEC-003-AC-13:** Given an Accounting Export File has just been created by FEAT-22.SPEC-002, when it is created, then its status is automatically Generated with no user action.

**FEAT-22.SPEC-003-AC-14:** Given an Accounting Export File is in Generated status, when FEAT-22.SPEC-002 first serves the file to Nadia, then its status is automatically set to Downloaded at that moment with no direct user edit possible on that field, and a later re-download leaves it Downloaded.

**FEAT-22.SPEC-003-AC-15:** Given the selected range spans Invoices in two currencies, when FEAT-22.SPEC-002 computes totals, then each currency's total is computed independently and never converted or summed into the other, per XBR-18.

**FEAT-22.SPEC-003-AC-16:** Given the selected range includes a manually recorded off-platform Payment and a processor-confirmed Payment, when totals are computed, then both contribute to the export on equal footing, per XBR-22.

**FEAT-22.SPEC-003-AC-17:** Given a file is in Downloaded status and still held, when Nadia taps "Download again", then the download is permitted for her; and given the file is no longer held, when a stale Download element is tapped, then nothing is served and the screen returns to Empty.

### Edge Cases

- **FEAT-22.SPEC-001 (Accounting Export Screen):** Double taps on Generate are ignored, a range with no invoices shows a try-a-different-range message rather than an empty file, and a failed generation retries safely with an Error state and no partial file offered. Navigating away from a ready file keeps nothing, since the export is never retained. Source: `docs/blueprint/specifications/FEAT-22-accounting-export/FEAT-22.SPEC-001-accounting-export-screen.md` (section: Edge Cases)
- **FEAT-22.SPEC-002 (Export File Generation):** A refunded or partially refunded invoice shows both its original total and the refunded amount without netting them (XBR-22), manually recorded payments are included on equal footing with processor-confirmed ones, and an unpaid invoice appears with no Payment row. Ranges spanning three or more currencies produce a group and subtotal per currency. Source: `docs/blueprint/specifications/FEAT-22-accounting-export/FEAT-22.SPEC-002-export-file-generation.md` (section: Edge Cases)
- **FEAT-22.SPEC-003 (Export Scope, Authorization & Currency Rules):** An account with no invoices collapses the date bound to today, so any range yields the No Results outcome rather than a validation error. Start and end dates exactly on the earliest invoice date or today pass (inclusive), and processing-time re-checks govern if a new invoice is sent after the screen opened. Source: `docs/blueprint/specifications/FEAT-22-accounting-export/FEAT-22.SPEC-003-export-scope-authorization-currency-rules.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-22.SPEC-001** (Accounting Export Screen) as specified: Nadia selects a date range and a file format, generates a bookkeeping export of her invoices and payments, and downloads it; Dana sees the same screen read-only with no generate or download controls. Full spec: `docs/blueprint/specifications/FEAT-22-accounting-export/FEAT-22.SPEC-001-accounting-export-screen.md`
- **FR-002**: The system MUST implement **FEAT-22.SPEC-002** (Export File Generation) as specified: Aggregates the selected date range's Invoice and Payment records into a CSV or QuickBooks/Xero-compatible file, hands it to the screen for download, and marks it Downloaded the moment it is first served. Full spec: `docs/blueprint/specifications/FEAT-22-accounting-export/FEAT-22.SPEC-002-export-file-generation.md`
- **FR-003**: The system MUST implement **FEAT-22.SPEC-003** (Export Scope, Authorization & Currency Rules) as specified: Governs who may generate and download versus view only, bounds the date range to the account's actual invoice history, and enforces that totals are derived only from Invoice/Payment records and never converted or summed across currencies. Full spec: `docs/blueprint/specifications/FEAT-22-accounting-export/FEAT-22.SPEC-003-export-scope-authorization-currency-rules.md`

### Key Entities

- Accounting Export File (create)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Export generation, downloads and empty-result displays are each observable as distinct signals (export_generated, export_downloaded, export_empty_result_shown); no metric in the success-metrics register connects to this feature, so the outcome is grounded in its Signals alone. Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-19**: Money, dates and time zones are locale-aware, and exports group per currency. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-25**: Financial records are never silently altered, so exports reflect original totals and refunds separately. Full register: `docs/blueprint/features/assumptions-constraints.md`
