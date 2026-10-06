---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-15.SPEC-007
spec_name: Multi-Currency Dashboard Non-Aggregation Rule
spec_slug: multi-currency-dashboard-non-aggregation-rule
parent_feature: FEAT-15
parent_feature_name: Currency & Tax Handling
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 2
acceptance_criteria_count: 8
---

# Logic/Rule Spec: Multi-Currency Dashboard Non-Aggregation Rule

## Overview

**Name:** Multi-Currency Dashboard Non-Aggregation Rule
**ID:** FEAT-15.SPEC-007
**Type:** Logic/Rule
**Purpose:** Governs that financial totals are always shown grouped per currency and are never converted or summed across currencies.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling
**Governed Entity:** Invoice and Payment (read-only, as the source records behind any financial total this rule governs)

## Scope and Non-Goals

**In Scope:**
- The grouping rule: any financial total spanning more than one project or client is grouped per currency
- The prohibition rule: amounts in different currencies are never converted to a common currency, and never added together
- Establishing this spec as the cross-feature authority the Freelancer Financial Dashboard (FEAT-12) consumes when it aggregates earned/outstanding/overdue totals

**Non-Goals:**
- Computing the totals themselves (which invoices/payments count toward earned, outstanding, or overdue) -- owned entirely by Freelancer Financial Dashboard (FEAT-12); this spec only governs how those already-computed per-currency totals may be combined for display, which is: not at all
- Any currency conversion or live exchange-rate lookup capability -- excluded per this feature's Non-Goals: the assumptions-constraints.md Dependencies slice names no external capability this feature relies on, and building conversion would directly contradict this spec's own prohibition
- A single invoice spanning multiple currencies -- excluded per this feature's Non-Goals: XBR-17 ties every invoice to its project's one configured currency, so no per-invoice multi-currency scenario exists for this rule to govern
- Accounting Export's per-currency handling -- Accounting Export (FEAT-22) is a separate consumer of Invoice/Payment records with its own export-format rules; this spec only names it in the Business Rules below as one of the two features the underlying prohibition (XBR-18) governs

## Governed Entity

**Entity:** Invoice, Payment (read-only)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| Invoice.currency | enum (recognized world currency code) | The currency an invoice's amount, tax, and total are denominated in (stamped from the project's configuration, FEAT-15.SPEC-008) |
| Invoice.total | number | The invoice total in its own `currency`; the figure this rule's grouping applies to |
| Payment.amount | number | A payment's amount, always equal to its invoice's full amount and therefore in that invoice's `currency` |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-12 (Freelancer Financial Dashboard) | Financial totals aggregation and dashboard/drill-down display | Whenever earned, outstanding, or overdue totals are computed or displayed across more than one client or project |
| FEAT-22 (Accounting Export) | Export file generation | Whenever exported totals span more than one currency, the export preserves the per-currency grouping rather than summing across currencies |

## Field Validation Rules

Not applicable beyond data type -- this spec places no new validation rule on Invoice.currency, Invoice.total, or Payment.amount; each is validated where it is written (Invoice.currency by FEAT-15.SPEC-003 at generation time, per FEAT-15.SPEC-008). This spec governs only how these already-valid, already-stamped values may be combined for display.

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Currency grouping | Invoice.currency, Invoice.total, Payment.amount | Any total that would span invoices or payments in more than one currency is computed and shown as separate per-currency subtotals, never as one combined figure | N/A -- this is a display-composition rule, not a validation failure; there is no user-facing error, only the correct grouped presentation |

## Authorization Rules

Not applicable -- this is a computation and display-composition rule, not an access-gated action. Whether Nadia (or any role) may view a given total at all is governed by the consuming spec's own Access Matrix rules (e.g., FEAT-12's own Access and Visibility); this spec governs only how a visible total is composed once access is already granted.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-----------------------|
| Grouped total (per currency) | Derived: for each distinct currency present among the underlying Invoice/Payment records, sum only the amounts in that currency; present each currency's sum as its own subtotal | Every time a financial total spanning more than one project or client is computed for display (FEAT-12) or export (FEAT-22) | No -- there is no setting or toggle that produces a combined, converted total; the grouped presentation is the only presentation the product defines |

## Business Rules

- XBR-18: financial totals are shown per currency; amounts in different currencies are never converted or added together -- this spec is the rule's sole definition; FEAT-12 and FEAT-22 consume it.
- This rule exists specifically because a freelancer serving clients in different countries sets a different currency per client (this feature's Primary Flows & Alternates), so any aggregate view that spans clients must, by construction, potentially span currencies.
- The non-aggregation rule is unconditional: there is no threshold, permission level, or display mode under which the product ever shows a single combined figure across currencies -- not even an approximate or clearly-labeled estimate.
- A single-currency freelancer (one who has configured every project in the same currency) sees what looks like one combined total, but this is the grouping rule producing exactly one group, not an exception to the rule.

## Edge Cases

- **A freelancer has exactly one currency across all active projects** -- The grouped presentation naturally collapses to a single subtotal, which is visually indistinguishable from "one combined total" but is still produced by the same per-currency grouping logic, not a special case.
- **A freelancer adds a client in a new currency after months of single-currency operation** -- The next time totals are computed, a second per-currency group appears; no historical total is retroactively recomputed or merged.
- **A project's currency configuration is locked (FEAT-15.SPEC-004) but an older, pre-lock invoice exists in a different currency due to a since-corrected configuration error** -- Not applicable: FEAT-15.SPEC-004 locks currency at the first invoice, so no project ever produces invoices in more than one currency; this scenario cannot occur under the product's own rules, and is noted here to confirm no such edge case exists.
- **A total includes both a positive invoice amount and a negative refund/reversal figure in the same currency** -- Both net within their shared currency's subtotal normally; a refund or reversal in one currency never offsets or nets against a total in a different currency.
- **The Accounting Export (FEAT-22) is generated for a freelancer with multiple currencies** -- The export preserves the same per-currency grouping this spec defines, rather than summing across currencies into the export's file.

## Acceptance Criteria

**FEAT-15.SPEC-007-AC-01:** Given Nadia has clients billed in USD and clients billed in EUR, when she opens her financial dashboard, then earned, outstanding, and overdue totals are shown as separate USD and EUR subtotals, never combined into one figure.

**FEAT-15.SPEC-007-AC-02:** Given Nadia's dashboard totals span two currencies, when she looks for a single combined "total across all currencies" figure, then none is shown anywhere on the dashboard.

**FEAT-15.SPEC-007-AC-03:** Given Nadia has configured every active project in the same currency, when she opens her dashboard, then she sees exactly one subtotal group, produced by the same per-currency grouping logic as a multi-currency account.

**FEAT-15.SPEC-007-AC-04:** Given Nadia adds her first client in a new currency, when totals are next computed, then a new per-currency group appears alongside her existing group(s), with no retroactive recomputation of prior totals.

**FEAT-15.SPEC-007-AC-05:** Given Nadia records a refund in EUR against an EUR invoice, when the dashboard recomputes her EUR subtotal, then the refund nets within the EUR group only, never against a USD or other-currency group.

**FEAT-15.SPEC-007-AC-06:** Given Nadia generates an accounting export (FEAT-22) across multiple currencies, when the export file is produced, then it preserves the same per-currency grouping the dashboard uses, with no converted or summed cross-currency figure.

**FEAT-15.SPEC-007-AC-07:** Given Nadia drills down from the dashboard into a specific client's totals, when the client's projects span more than one currency, then that drill-down also shows per-currency subtotals, consistent with the dashboard-level rule.

**FEAT-15.SPEC-007-AC-08:** Given the non-aggregation rule is unconditional, when any future display mode or permission level is considered by a downstream builder, then no exception path exists in this spec that would permit a combined cross-currency figure under any condition.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- no new validation beyond data type) | 0 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 0 (N/A -- computation/display rule, not access-gated) | 0 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
