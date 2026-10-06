---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-22.SPEC-003
spec_name: Export Scope, Authorization & Currency Rules
spec_slug: export-scope-authorization-currency-rules
parent_feature: FEAT-22
parent_feature_name: Accounting Export
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 12
acceptance_criteria_count: 17
---

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
