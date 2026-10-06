---
document_type: feature-overview
feature_number: FEAT-22
feature_name: Accounting Export
feature_slug: accounting-export
priority_tier: Important
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 3
screen_count: 1
automation_count: 1
logic_rule_count: 1
integration_count: 0
notification_count: 0
---

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
