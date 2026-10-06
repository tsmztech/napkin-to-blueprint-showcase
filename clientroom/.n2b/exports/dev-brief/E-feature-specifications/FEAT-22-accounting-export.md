# FEAT-22 — Accounting Export

This chapter covers Accounting Export, a Important-tier feature. It contains the feature breakdown brief followed by every specification in full: 3 specifications carrying 47 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-22.SPEC-001 | Accounting Export Screen | screen | 17 |
| FEAT-22.SPEC-002 | Export File Generation | automation | 13 |
| FEAT-22.SPEC-003 | Export Scope, Authorization & Currency Rules | logic-rule | 17 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Accounting Export

## Summary

**Feature:** Accounting Export
**ID:** FEAT-22
**Description:** The freelancer exports invoices and payments as a CSV or a QuickBooks/Xero-compatible file for her own bookkeeping.
**Priority:** Important
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md, Ecosystem & Integrations: "v1 provides an export file (CSV or a QuickBooks/Xero-compatible format). No live sync." Ranked Important because bookkeeping is a real recurring need but not part of the core client-facing loop. [MODIFIED: phase moved from v1 to MVP based on BRIEF.md using "v1" throughout to mean the first release ("Solo freelancers only for v1", "web app for v1", linked deliverables "for v1"), so its export commitment belongs in the launch product; the regular Month-End Financial Review journey depends on it] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Select a date range -- scope the export to a period
- Generate a file -- CSV or QuickBooks/Xero-compatible format
- Download -- pull the file into her own accounting software

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-22.SPEC-001 | Accounting Export Screen | Screen | Nadia (Freelancer), Dana (Support Operator) | Nadia selects a date range, generates a CSV or QuickBooks/Xero-compatible export, and downloads it; Dana sees the same screen read-only with no generate or download controls |
| FEAT-22.SPEC-002 | Export File Generation | Automation | Nadia (Freelancer) | Aggregates the selected date range's Invoice and Payment records into a CSV or QuickBooks/Xero-compatible file, hands it to the screen for download, and marks it Downloaded once pulled |
| FEAT-22.SPEC-003 | Export Scope, Authorization & Currency Rules | Logic/Rule | Nadia (Freelancer), Dana (Support Operator) | Governs who may generate and download versus view only, bounds the date range to the account's actual invoice history, and enforces that totals are derived only from Invoice/Payment records and never converted or summed across currencies |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Select a date range -- scope the export to a period | FEAT-22.SPEC-001, FEAT-22.SPEC-003 | The screen offers the date-range picker; the rules spec bounds the range to the account's actual invoice history | Phase 2 (Explicit) |
| Generate a file -- CSV or QuickBooks/Xero-compatible format | FEAT-22.SPEC-001, FEAT-22.SPEC-002, FEAT-22.SPEC-003 | The screen offers the Generate action and format choice; the automation aggregates Invoice and Payment records into the chosen format; the rules spec gates who may trigger it and how totals are derived | Phase 2 (Explicit) |
| Download -- pull the file into her own accounting software | FEAT-22.SPEC-001, FEAT-22.SPEC-002 | The screen offers the Download action once the file is ready; the automation serves the file and marks it Downloaded | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-22.SPEC-002 | Export File Generation | Phase 4 (Trigger-Response Analysis) | Aggregating and formatting Invoice/Payment data into CSV or QuickBooks/Xero output is processing logic, not a direct data write; per the standalone-spec decision rule this becomes a standalone Automation spec rather than an inline screen interaction |
| FEAT-22.SPEC-003 | Export Scope, Authorization & Currency Rules | Phase 5 (Rule-Constraint Discovery) | Five or more interacting rules apply across SPEC-001 and SPEC-002: the Nadia-only vs. Dana-view-only gate (Access field), the date-range bound to actual invoice history (Validation & Limits field), the per-currency segregation rule (XBR-18), and the totals-derived-only-from-Invoice/Payment rule (XBR-22) -- past the inline-validation threshold, so these are consolidated into one Logic/Rule spec rather than duplicated across the screen and automation |

## Entity-Lifecycle Coverage Matrix

**Entity: Accounting Export File**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-22.SPEC-002 | Generated on demand from the selected date range's Invoice and Payment records, in the chosen format | -- |
| Read (single) | FEAT-22.SPEC-001 | The Accounting Export Screen serves the generated file to Nadia through the Download action | -- |
| Read (list) | N/A | product-features.md's Domain Entity Inventory marks this entity "Managed by: N/A -- a point-in-time generated file, not an ongoing managed record"; there is never more than the single freshly-generated file to show, so no list/history view applies | -- |
| Update | N/A | The file is immutable once generated; a failed generation is retried by generating fresh rather than editing a partial one (product-features.md, States field: "a failed generation is retried without corrupting a partial file") | -- |
| Delete/Archive | N/A -- explicit non-goal | No retention/purge policy applies: the entity is generated per request and never persisted as an ongoing record (product-features.md, Domain Entity Inventory: "Managed by: N/A"; "Referenced by: N/A -- consumed outside the product"). There is nothing stored in-product to soft-delete, restore, or cascade | Recorded as a non-goal below |
| State Transition | FEAT-22.SPEC-002 | Generated -> Downloaded, applied the moment Nadia's download completes; recorded via the `export_generated` and `export_downloaded` signals | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Invoice | FEAT-22.SPEC-002 | Sole source (with Payment) of export content for the selected date range; sourced from Invoice Generation & Sending (FEAT-09) |
| Payment | FEAT-22.SPEC-002 | Sole source (with Invoice) of export content for the selected date range, including refunds, reversals, and manually recorded payments (XBR-22); sourced from Invoice Payment Processing (FEAT-10) |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia selects a date range and taps Generate | Validate the range and her authorization to generate | Standalone Logic/Rule | SPEC-003 |
| Range and authorization pass validation | Aggregate Invoice and Payment records into the chosen file format | Standalone Automation | SPEC-002 |
| The selected range has no invoices | Show a plain "nothing to export" message rather than a broken empty file | Inline in triggering screen (Empty state) | SPEC-001 |
| Generation fails partway through | Retry safely without exposing or corrupting a partial file | Standalone Automation (failure handling) | SPEC-002 |
| The file finishes generating | Offer the Download action on the screen | Inline in triggering screen | SPEC-001 |
| Nadia taps Download | Serve the file and mark the entity Downloaded | Standalone Automation (state transition) | SPEC-002 |
| Dana opens the export screen during a support session | Show the screen read-only, with no Generate or Download control | Standalone Logic/Rule (access gating) | SPEC-003 |
| The selected range spans more than one currency | Show totals per currency; never convert or sum across currencies | Standalone Logic/Rule | SPEC-003 |
| Any export lifecycle event (generated, downloaded, empty-result shown) | Write an append-only Activity Log Entry | Cross-feature -- owned by Immutable Activity & Audit Trail (FEAT-13) | FEAT-13 responsibility |
| Dana's read-only view of this screen | Excluded from downloads and exports, always announced to Nadia by email, always listed in her trail (XBR-29) | Cross-feature -- owned by Support Access (FEAT-31) | FEAT-31 responsibility |

## Shared Context

**Shared Entities:**
- Accounting Export File -- created by SPEC-002, served for download and displayed by SPEC-001, gated by SPEC-003's scope and authorization rules. Fields (functional): date range covered, format (CSV or QuickBooks/Xero-compatible), status (Generated, Downloaded).
- Invoice, Payment (read-only) -- read by SPEC-002 as the sole source of export content; this feature never creates, updates, or deletes either.

**Shared UI Patterns:**
- Single-surface generate-and-download pattern -- SPEC-001 is the one screen for this feature's entire capability set (date-range selection, format choice, generate, empty/loading/error states, download); its states are all instances of the same screen rather than separate specs, consistent with the "one purpose per screen" heuristic balanced against unnecessary splitting.
- Role-differentiated single view -- rather than a separate screen for Dana, SPEC-001 renders the same screen with Generate/Download controls hidden per SPEC-003's authorization rule, so Nadia and Dana never diverge onto different layouts.

**Shared Validation:**
- SPEC-003 defines the date-range bound, the Nadia-only vs. Dana-view-only gate, and the per-currency and totals-source rules. SPEC-001 and SPEC-002 both reference SPEC-003 rather than restating these rules.

## Internal Dependency Map

```
SPEC-001 (Accounting Export Screen) -> [Nadia selects a date range and taps Generate] -> SPEC-003 (Export Scope, Authorization & Currency Rules) -> [pass] -> SPEC-002 (Export File Generation)
SPEC-002 (Export File Generation) -> [file ready] -> SPEC-001 (Accounting Export Screen shows the Download action)
SPEC-002 (Export File Generation) -> [no invoices in range] -> SPEC-001 (Accounting Export Screen shows the Empty state)
SPEC-001 (Accounting Export Screen) -> [Nadia taps Download] -> SPEC-002 (Export File Generation marks the entity Downloaded)
SPEC-001 (Accounting Export Screen) -> [Dana opens the screen in a support session] -> SPEC-003 (Export Scope, Authorization & Currency Rules gates the screen to view-only)
```

**Default Entry:** SPEC-001 (Accounting Export Screen) -- reached from the Freelancer Financial Dashboard (FEAT-12) navigation connection.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-22.SPEC-001 | Inbound | FEAT-12 (Freelancer Financial Dashboard) | Nadia navigates here to generate the month's export as part of her Month-End Financial Review journey | Nadia taps to export from the dashboard |
| FEAT-22.SPEC-002 | Inbound | FEAT-09 (Invoice Generation & Sending) | Reads Invoice records as one of the two sources of export content | Export generation runs |
| FEAT-22.SPEC-002 | Inbound | FEAT-10 (Invoice Payment Processing) | Reads Payment records as one of the two sources of export content, including refunds, reversals, and manually recorded payments | Export generation runs |
| FEAT-22.SPEC-001, FEAT-22.SPEC-002 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Generation, download, and empty-result events are each written to the append-only trail | Any export lifecycle event |
| FEAT-22.SPEC-001, FEAT-22.SPEC-003 | Inbound | FEAT-31 (Support Access) | Dana's read-only view of this screen occurs inside a logged support session that excludes downloads and exports (XBR-29) | Dana opens a support session |
| FEAT-22.SPEC-003 | Inbound | FEAT-15 (Currency & Tax Handling) | The per-currency segregation rule (XBR-18) derives from FEAT-15's ownership of currency | The selected range spans invoices in more than one currency |

## Non-Functional Notes

**Data volumes / growth:** A freelancer has 3-15 active clients (assumptions-constraints.md, ASMP-22); the export spans a bounded date range of already-stored Invoice and Payment records, so this feature carries no independent growth concern beyond its source data. Large date ranges are the trigger for the Loading state's progress display (product-features.md, States field) rather than an unbounded-volume risk. This feature emits `export_generated`, `export_downloaded`, and `export_empty_result_shown` signals (product-features.md, Signals field); SPEC-002 fires the generated and downloaded signals, and SPEC-001 fires the empty-result signal.

**Responsiveness:** Generation shows real progress for large date ranges rather than a silent wait (product-features.md, States field; assumptions-constraints.md, ASMP-27); a failed generation is retried without corrupting a partial file, so Nadia never receives or is left waiting on a broken output (product-features.md, States field).

**Data sensitivity / privacy:** The exported content is Nadia's own financial records -- invoice and payment data carrying personal data (client billing name and address, contact identity), GDPR-class (feature-dependency-map.md, Entity: Invoice, Data Sensitivity; assumptions-constraints.md, ASMP-24). The export is scoped strictly to the freelancer's own data (product-features.md, Validation & Limits field), and no card or payment credential data ever appears in it, since Invoice and Payment records never hold it (assumptions-constraints.md, ASMP-24).

**Compliance flags:** GDPR-class handling applies to the exported content (assumptions-constraints.md, ASMP-24). The underlying Invoice and Payment records may be subject to legal financial-record retention on account deletion (feature-dependency-map.md, Entity: Invoice and Entity: Payment, Data Sensitivity, citing scope-boundaries.md SC-24); this feature itself retains no export file record of its own to which that retention could apply.

## Non-Goals

- **Live, two-way accounting sync** -- Excluded per scope-boundaries.md (SC-07): BRIEF.md's Ecosystem & Integrations states "v1 provides an export file (CSV or a QuickBooks/Xero-compatible format). No live sync"; this feature is the entirety of the product's accounting integration.
- **Automatic tax calculation per country or region within the export** -- Excluded per scope-boundaries.md (SC-16): tax handling stays at the freelancer-configured tax label and rate set in Currency & Tax Handling (FEAT-15); this feature reports what is stored, it does not calculate tax.
- **Retention of generated export files as an ongoing managed record** -- Intentional lifecycle decision surfaced by the CRUD matrix: product-features.md's Domain Entity Inventory marks the Accounting Export File entity "Managed by: N/A -- a point-in-time generated file, not an ongoing managed record," so no delete/archive/purge policy applies; each export is generated fresh from current Invoice and Payment records rather than stored and retrieved later.
- **Export generation or download by any client contact, or by the Support Operator** -- Excluded per product-features.md's Access field and scope-boundaries.md (SC-01, SC-04): Nadia alone can generate and download; Dana (Support Operator) sees the screen read-only per XBR-29's explicit exclusion of exports from support sessions; no client contact has export access at all.
- **Support for accounting formats beyond CSV and the QuickBooks/Xero-compatible file** -- Excluded per product-features.md's Key Capabilities, which name exactly these two format options, and BRIEF.md's Ecosystem & Integrations, which names no others for v1.



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



# Logic/Rule Spec: Export Scope, Authorization & Currency Rules

## Overview

**Name:** Export Scope, Authorization & Currency Rules
**ID:** FEAT-22.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs who may generate and download versus view only, bounds the date range to the account's actual invoice history, and enforces that totals are derived only from Invoice/Payment records and never converted or summed across currencies.
**Parent Feature:** FEAT-22 -- Accounting Export
**Governed Entity:** Accounting Export File

## Scope and Non-Goals

**In Scope:**
- Field validation for the Accounting Export File's date-range and format fields
- The cross-field rule bounding the selectable date range to the account's actual invoice history
- Authorization rules for every action on this feature's screen and file (view, generate, download), across every role in the Access Matrix
- Default and derived values on the Accounting Export File
- The currency-segregation and totals-source rules (XBR-18, XBR-22) that govern how the export's content may be computed

**Non-Goals:**
- The actual reading of Invoice and Payment records and assembly of the file content -- handled by FEAT-22.SPEC-002 (Export File Generation), which enforces these rules but does not define them
- Rendering the Generate/Download controls, progress states, or result messages -- owned by FEAT-22.SPEC-001 (Accounting Export Screen), which applies this spec's authorization outcomes but does not define them
- Any rule about the Invoice or Payment entities' own field validation -- those entities are owned and validated by Invoice Generation & Sending (FEAT-09) and Invoice Payment Processing (FEAT-10) respectively; this spec only constrains how their data may be read and combined for export
- Automatic currency conversion or tax calculation of any kind -- excluded per scope-boundaries.md (SC-16): tax handling stays at the freelancer-configured label and rate set in Currency & Tax Handling (FEAT-15); this spec reports what is stored, it does not calculate

## Governed Entity

**Entity:** Accounting Export File
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| date_range_start | date | The first date included in the export's scope |
| date_range_end | date | The last date included in the export's scope |
| format | enum (CSV, QuickBooks/Xero-compatible) | The chosen output file format |
| status | enum (Generated, Downloaded) | The file's lifecycle state |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-22.SPEC-001 | Accounting Export Screen | Field validation on date selection (picker bounds) and on Generate tap; authorization on screen entry (which controls render for the signed-in role) and again on each Generate/Download tap |
| FEAT-22.SPEC-002 | Export File Generation | Authoritative re-validation of the date range and requester authorization at the moment processing begins; enforcement of the currency-segregation and totals-source rules during assembly |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| date_range_start | Required; must fall on or after the account's earliest Invoice issue_date | Always | On date selection (picker will not offer earlier dates) and again on Generate | "Select a start date within your invoice history." | Yes |
| date_range_end | Required; must not fall after today | Always | On date selection and again on Generate | "End date cannot be in the future." | Yes |
| date_range_end | Must fall on or after date_range_start | Always | On date selection and again on Generate | "End date must be on or after the start date." | Yes |
| format | Required; must be exactly one of CSV or QuickBooks/Xero-compatible | Always | On Generate | "Choose a file format to continue." | Yes |
| status | No validation beyond data type -- system-managed transition (Generated -> Downloaded only, at first serve; Downloaded never reverts) | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Range bounded to invoice history | date_range_start, date_range_end | Both dates must fall within [the account's earliest Invoice issue_date, today]; when the account has no Invoices at all, the only valid range collapses to today, so any selection returns FEAT-22.SPEC-002's No Results outcome rather than a validation error | "Select a range within your invoice history." |
| Range ordering | date_range_start, date_range_end | date_range_end must not precede date_range_start | "End date must be on or after the start date." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the Accounting Export Screen | Nadia (Freelancer) | Always, her own account only | -- |
| View the Accounting Export Screen | Dana (Support Operator) | Only inside a logged, read-only support session on the named freelancer's account (FEAT-31.SPEC-005) | Outside an active support session, the screen is not reachable at all -- there is no operator-side navigation into any freelancer's export screen except through an open session |
| View the Accounting Export Screen | Owen (Client Primary Contact) | Never | Not shown; no navigation path exists into this screen from the client portal (product-features.md's Access field: "no client contact has export access at all") |
| View the Accounting Export Screen | Priya (Client Reviewer Contact) | Never | Same as Owen -- no navigation path exists |
| Generate an export | Nadia (Freelancer) | Always, scoped strictly to her own account's Invoice and Payment records | -- |
| Generate an export | Dana (Support Operator) | Never | Generate control is never rendered during a support session (XBR-29: "support sessions... exclude file downloads and data/accounting exports"); a stale-UI attempt is refused with "Support sessions can't generate or download exports." |
| Generate an export | Owen (Client Primary Contact) | Never | Not shown -- the screen itself is unreachable (see View, above) |
| Generate an export | Priya (Client Reviewer Contact) | Never | Not shown -- the screen itself is unreachable |
| Download an export (first download or "Download again") | Nadia (Freelancer) | Always, only for a file generated within her own current session and still held (Generated or Downloaded status) | If the file is no longer held (range/format changed or screen left), nothing is served and the screen returns to its Empty state |
| Download an export | Dana (Support Operator) | Never | Download control is never rendered during a support session (XBR-29); a stale-UI attempt is refused with "Support sessions can't generate or download exports." |
| Download an export | Owen (Client Primary Contact) | Never | Not shown -- the screen itself is unreachable |
| Download an export | Priya (Client Reviewer Contact) | Never | Not shown -- the screen itself is unreachable |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| date_range_start | No default -- Nadia selects a start date each time; nothing is pre-filled or remembered between sessions, since the entity is never retained | On screen entry, each session | Yes (by selecting a different date within the bound) |
| date_range_end | No default -- selected each time, same as date_range_start | On screen entry, each session | Yes |
| format | No default -- Nadia must make an explicit choice each time; no format is pre-selected | On screen entry, each session | Yes |
| status | Set to Generated automatically the instant FEAT-22.SPEC-002 creates the record; transitions to Downloaded automatically the instant FEAT-22.SPEC-002 first serves the file to Nadia (the same moment FEAT-22.SPEC-002 Step 11 defines; browser transfer completion is not observable and is not the trigger); later re-downloads leave it Downloaded | On create (Generated); on first serve (Downloaded) | No -- both transitions are system-driven, never a direct user edit |

## Business Rules

- XBR-18: Financial totals are shown per currency; amounts in different currencies are never converted or added together. This applies to every total this feature computes or displays, including the multi-currency informational line on FEAT-22.SPEC-001 and every subtotal FEAT-22.SPEC-002 assembles into the file.
- XBR-22: Financial Dashboard and Accounting Export totals are derived only from Invoice and Payment records, including refunds, reversals, and manually recorded payments -- no other entity or freelancer-entered figure may contribute to an export total.
- XBR-29: The operator's support sessions are read-only in every feature and specifically exclude data/accounting exports -- this spec's Authorization Rules table is the enforcement point for that exclusion within FEAT-22.
- The date-range bound (Cross-Field Rules) is re-evaluated live against the account's current Invoice history at the moment Generate is pressed, not cached from when the screen first loaded, so a newly sent Invoice extends the selectable range within the same session.
- The Nadia-only generate/download gate applies uniformly regardless of file format chosen -- there is no format for which Dana, Owen, or Priya gain any additional access.

## Edge Cases

- **Account has no Invoices at all** -- The date-range bound collapses to today only; Nadia can still open the screen and attempt Generate, but any range she selects yields FEAT-22.SPEC-002's No Results outcome rather than a validation error, since a brand-new account is a valid state, not an invalid range.
- **date_range_start selected exactly on the account's earliest Invoice issue_date** -- Passes validation; the boundary is inclusive.
- **date_range_end selected exactly as today's date** -- Passes validation; the boundary is inclusive.
- **A new Invoice is sent between opening the screen and pressing Generate, extending what "today" or "earliest" would bound** -- FEAT-22.SPEC-002's authoritative re-check at processing time (not the screen's earlier picker state) governs whether the previously selected range is still valid; a range that was valid when selected remains valid, since the bound only ever widens forward in time.
- **Dana's support session is opened on a freelancer account, then closed, then reopened** -- Each session independently grants View-only access for its duration; no session carries forward any elevated access, and Generate/Download remain absent in every session regardless of how many times one is opened.
- **A client contact (Owen or Priya) is somehow given a direct link to this screen's address** -- The screen is not reachable outside the freelancer's own authenticated context (see Authorization Rules); no client-portal session, however constructed, satisfies the requester-is-Nadia condition Generate and Download both require.
- **date_range_start and date_range_end are both selected as the same single date** -- Passes the ordering rule (end is not before start); this is a valid one-day range and is processed normally by FEAT-22.SPEC-002.
- **The selected range spans invoices in exactly two currencies where one has zero eligible Payments** -- Both currency groups still appear in the assembled totals per XBR-18; a currency group with no Payments shows only its invoiced-amount total, never a zero implicitly merged into the other currency's figures.

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 12 | 12 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |
