---
document_type: feature-overview
feature_number: FEAT-12
feature_name: Freelancer Financial Dashboard
feature_slug: freelancer-financial-dashboard
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 4
screen_count: 2
automation_count: 1
logic_rule_count: 1
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Freelancer Financial Dashboard

## Summary

**Feature:** Freelancer Financial Dashboard
**ID:** FEAT-12
**Description:** The freelancer sees earned, outstanding, and overdue totals across all clients and projects at a glance, and can drill into a single client or project's financial detail.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md, Experience narrative: "At month end you open your dashboard and see earned, outstanding and overdue, per client" — this is the brief's explicit description of the freelancer's core recurring need. MVP phase: the "money in one place" promise depends on it.

**Key Capabilities:**
- View aggregate totals — earned, outstanding, overdue across all clients
- Drill into a client or project — see the financial detail behind the aggregate
- See status at a glance — visually distinguish paid, due, and overdue

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-12.SPEC-001 | Dashboard Overview | Screen | Nadia (Freelancer), Dana (Support Operator) | Nadia sees aggregate earned/outstanding/overdue totals across every client, filterable by client or period, with a zero-state for a new account |
| FEAT-12.SPEC-002 | Client/Project Financial Drill-down | Screen | Nadia (Freelancer), Dana (Support Operator) | Nadia opens one client or project to see the specific invoices behind its aggregate totals |
| FEAT-12.SPEC-003 | Financial Totals Aggregation | Logic/Rule | Nadia (Freelancer), Dana (Support Operator) | Derives earned/outstanding/overdue totals (and their paid/due/overdue status classification) from Invoice and Payment records, per currency, honoring refunds and manual payments |
| FEAT-12.SPEC-004 | Dashboard Totals Refresh | Automation | Nadia (Freelancer), Dana (Support Operator) | Recomputes affected totals when an underlying invoice or payment changes elsewhere, and falls back to the last successfully computed totals with retry if aggregation fails |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| View aggregate totals — earned, outstanding, overdue across all clients | FEAT-12.SPEC-001, FEAT-12.SPEC-003 | Dashboard Overview is the primary screen; the aggregation rule computes the totals it displays | Phase 2 (Explicit) |
| Drill into a client or project — see the financial detail behind the aggregate | FEAT-12.SPEC-002 | Primary purpose of the Client/Project Financial Drill-down screen | Phase 2 (Explicit) |
| See status at a glance — visually distinguish paid, due, and overdue | FEAT-12.SPEC-001, FEAT-12.SPEC-002, FEAT-12.SPEC-003 | Status badges/color treatment on both screens, driven by the classification the aggregation rule produces | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |
| Filter by client or period (Primary Flows & Alternates) | FEAT-12.SPEC-001 | Alternate flow on the Dashboard Overview screen narrows totals via FEAT-12.SPEC-003 | Phase 2 (Explicit elaboration of stated alternate flow) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-12.SPEC-003 | Financial Totals Aggregation | Phase 5 (Rule Discovery) | The feature's Data Notes state all totals are "derived... nothing is captured directly in this view," and the Validation & Limits field plus XBR-18/XBR-20/XBR-22 impose a multi-condition derivation (per-currency separation, single-full-payment rule, refunds/reversals/manual payments) that exceeds the inline-validation threshold and is shared by both screens and the refresh automation |
| FEAT-12.SPEC-004 | Dashboard Totals Refresh | Phase 4 (Trigger-Response) | The Send a Proposal journey's Step 5 requires the dashboard to reflect a payment "instantly," and the feature's Error state requires the last successfully computed totals plus retry on aggregation failure — a cross-feature, stateful side-effect nobody named as a capability |

## Entity-Lifecycle Coverage Matrix

This feature manages no entity of its own. Its Connected Entities (product-features.md) are Invoice, Payment, and Project, all listed as **read** — the dashboard is a derived view with no create, update, delete, or state-transition surface of its own (Data Notes: "nothing is captured directly in this view"). No CRUD matrix applies; the full lifecycle for each entity is owned and specified by the features named below.

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Invoice | FEAT-12.SPEC-001, FEAT-12.SPEC-002, FEAT-12.SPEC-003, FEAT-12.SPEC-004 | Status (Sent, Payment pending, Paid, Overdue, Refunded, Partially refunded, Disputed, Corrected), amount, currency, and due date drive every total and the client/project drill-down list; owned by FEAT-09 (creation, sending) and updated by FEAT-10/FEAT-11/FEAT-25 |
| Payment | FEAT-12.SPEC-003, FEAT-12.SPEC-004 | Succeeded/Failed/Reversed status and amount determine the earned figure and whether a refund or reversal reduces it (XBR-20, XBR-22); owned by FEAT-10 |
| Project | FEAT-12.SPEC-001, FEAT-12.SPEC-002 | Groups invoices for the drill-down and names the client/project the freelancer narrows into; owned by FEAT-01 |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia opens the dashboard | Compute aggregate earned/outstanding/overdue totals across every client, per currency | Standalone Logic/Rule | FEAT-12.SPEC-003 |
| Nadia narrows totals to one client or a date range | Recompute filtered totals using the same aggregation rule | Inline in triggering screen (uses SPEC-003) | FEAT-12.SPEC-001 |
| Nadia drills into a specific client or project | Load that entity's invoice-level financial detail | Inline in triggering screen (uses SPEC-003) | FEAT-12.SPEC-002 |
| An invoice or payment underlying the totals changes elsewhere (sent, paid, overdue flag set, refunded, reversed, manually recorded) | Recompute the affected totals so the dashboard reflects the change without a manual refresh | Standalone Automation | FEAT-12.SPEC-004 |
| Aggregation across many clients takes noticeable time | Show a lightweight progress indicator while totals render | Inline in triggering screen | FEAT-12.SPEC-001 / FEAT-12.SPEC-002 |
| Aggregation fails | Show the last successfully computed totals with a retry option — never a blank or misleading number | Standalone Automation | FEAT-12.SPEC-004 |
| Freelancer has no invoices yet | Show an explanatory zero-state with a prompt toward sending a first proposal | Inline in triggering screen (navigates cross-feature to FEAT-02) | FEAT-12.SPEC-001 |
| A filter (client or period) matches no invoices | Show no-results messaging distinct from the true zero-state, with a way to clear the filter | Inline in triggering screen | FEAT-12.SPEC-001 / FEAT-12.SPEC-002 |
| Connectivity is lost while viewing | Keep the most recently loaded totals viewable, read-only | Inline in triggering screen | FEAT-12.SPEC-001 / FEAT-12.SPEC-002 |
| Totals span more than one currency | Keep each currency's totals separate; never convert or sum across currencies (XBR-18) | Standalone Logic/Rule | FEAT-12.SPEC-003 |
| Nadia opens an invoice flagged Overdue on the dashboard | Navigate to that invoice's reminder history and pause control | Cross-feature — owned by FEAT-11 | FEAT-12.SPEC-001 initiates; FEAT-11 owns the destination |
| Nadia opens a specific invoice from the client drill-down | Navigate to invoice detail | Cross-feature — owned by FEAT-09 | FEAT-12.SPEC-002 initiates; FEAT-09 owns the destination |
| Dana opens a logged support session | View the dashboard and drill-down read-only; no filter-driven export control is shown | Cross-feature — owned by FEAT-31 | FEAT-12.SPEC-001 / FEAT-12.SPEC-002 (View), FEAT-31 (session logging) |

## Shared Context

**Shared Entities:**
- **Invoice** (read-only) — read by FEAT-12.SPEC-001 (aggregate and per-currency totals), FEAT-12.SPEC-002 (drill-down invoice list), FEAT-12.SPEC-003 (derivation source), and FEAT-12.SPEC-004 (change detection). Fields read: status, amount, currency, due_date, project.
- **Payment** (read-only) — read by FEAT-12.SPEC-003 and FEAT-12.SPEC-004. Fields read: status, amount, paid_at, invoice.
- **Project** (read-only) — read by FEAT-12.SPEC-001 (client/project grouping) and FEAT-12.SPEC-002 (drill-down scope). Fields read: project_name, client, currency.

**Shared UI Patterns:**
- **Status badge / color treatment (paid, due, overdue)** — the same visual vocabulary appears on FEAT-12.SPEC-001 (aggregate totals) and FEAT-12.SPEC-002 (per-invoice detail), both driven by the classification FEAT-12.SPEC-003 produces; Spec Writers for both screens should describe it identically and never rely on color alone (ASMP-27).
- **Per-currency total block** — both screens render one self-contained total block per currency present, never a combined figure; Spec Writers should describe this the same way on both screens (XBR-18).
- **Empty / Loading / Error / No-Results / Offline-degraded states** — FEAT-12.SPEC-001 and FEAT-12.SPEC-002 share one convention: zero-state prompt toward a first proposal (dashboard only), lightweight progress indicator during aggregation, last-successfully-computed totals with retry on error, distinct no-results messaging for an empty filter result, and read-only viewing of the last-loaded totals when offline.

**Shared Validation:**
- FEAT-12.SPEC-003 (Financial Totals Aggregation) is referenced by FEAT-12.SPEC-001, FEAT-12.SPEC-002, and FEAT-12.SPEC-004 rather than re-deriving the earned/outstanding/overdue formula, the currency-separation rule, or the paid/due/overdue classification in each.

## Internal Dependency Map

```
SPEC-001 (Dashboard Overview) -> [Nadia opens the dashboard] -> SPEC-003 (Financial Totals Aggregation) -> [aggregate totals, per currency] -> SPEC-001
SPEC-001 (Dashboard Overview) -> [Nadia narrows to a client or period] -> SPEC-003 (Financial Totals Aggregation) -> [filtered totals] -> SPEC-001
SPEC-001 (Dashboard Overview) -> [Nadia drills into a client or project] -> SPEC-002 (Client/Project Financial Drill-down)
SPEC-002 (Client/Project Financial Drill-down) -> [loads] -> SPEC-003 (Financial Totals Aggregation) -> [per-client/project totals and invoice list] -> SPEC-002
SPEC-004 (Dashboard Totals Refresh) -> [an Invoice or Payment changes elsewhere] -> SPEC-003 (Financial Totals Aggregation) -> [recomputed totals] -> SPEC-001 / SPEC-002
SPEC-001 (Dashboard Overview) -> [aggregation fails] -> SPEC-004 (Dashboard Totals Refresh) -> [last successfully computed totals + retry] -> SPEC-001
SPEC-002 (Client/Project Financial Drill-down) -> [aggregation fails] -> SPEC-004 (Dashboard Totals Refresh) -> [last successfully computed totals + retry] -> SPEC-002
```

**Default Entry:** SPEC-001 (Dashboard Overview) -- the screen shown when the user navigates to this feature.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-12.SPEC-001 | Outbound | FEAT-11 (Automated Payment Reminders) | Opens an overdue invoice's reminder history and pause control | Nadia opens an invoice flagged Overdue on the dashboard |
| FEAT-12.SPEC-002 | Outbound | FEAT-09 (Invoice Generation & Sending) | Opens a specific invoice's detail | Nadia opens an invoice from the client drill-down |
| FEAT-12.SPEC-001 | Outbound | FEAT-22 (Accounting Export) | Opens the accounting export flow | Nadia generates the month's export from the dashboard |
| FEAT-12.SPEC-001 | Outbound | FEAT-02 (Proposal Creation & Sending) | Zero-state prompt opens a new proposal draft | Freelancer with no invoices yet taps the zero-state prompt |
| FEAT-12.SPEC-001 / FEAT-12.SPEC-002 | Inbound | FEAT-20 (Guided Onboarding) | Onboarding completion lands Nadia on the normal dashboard | Onboarding's first client, project, and draft proposal exist |
| FEAT-12.SPEC-003 / FEAT-12.SPEC-004 | Inbound | FEAT-09 (Invoice Generation & Sending) | Invoice creation, sending, and due-date changes feed the totals | An invoice is generated, sent, or its due date changes |
| FEAT-12.SPEC-003 / FEAT-12.SPEC-004 | Inbound | FEAT-10 (Invoice Payment Processing) | Payment success, failure, and the resulting Paid/Payment-pending status feed the totals | A payment succeeds, fails, or is recorded manually |
| FEAT-12.SPEC-003 | Inbound | FEAT-01 (Client & Project Management) | Client and project roster and archive status determine which totals and drill-down entries appear | Nadia's client/project roster changes |
| FEAT-12.SPEC-003 | Inbound | FEAT-15 (Currency & Tax Handling) | Each project's assigned billing currency determines how totals are segmented | A client/project's currency is set before its first invoice |
| FEAT-12.SPEC-001 / FEAT-12.SPEC-002 | Inbound | FEAT-31 (Support Access) | Dana's logged, read-only support session views the dashboard and drill-down | Dana opens a support session on Nadia's account |

## Non-Functional Notes

**Data volumes / growth:** The dashboard aggregates across a freelancer's full client roster — 3–15 active clients typical, a few thousand freelancers expected in year one (ASMP-22, SC-21) — with no hard limit on history depth (Validation & Limits); the aggregation and drill-down are designed to stay responsive at this scale from MVP onward.

**Responsiveness:** Dashboard totals appear within roughly 1–2 seconds on a typical connection (ASMP-21); the Dashboard Comprehension success metric targets Nadia stating her total earned, outstanding, and overdue amounts within 10 seconds of opening the dashboard, so the aggregation must render before that window elapses; a lightweight progress indicator covers the gap while many clients are aggregated (feature's States field, ASMP-27).

**Data sensitivity / privacy:** Totals are derived entirely from Invoice and Payment records, which carry GDPR-class personal data (client billing name and address, payer identity) per the dependency map's Data Sensitivity lines; the dashboard itself displays no card data (no card numbers or bank credentials are ever captured, ASMP-24/SC-10) and is strictly isolated to Nadia's own account — no client contact ever sees another client's totals or the freelancer's aggregate view (Access field, ASMP-23).

**Compliance flags:** Any operator (Dana) access to this view is read-only, time-limited, and logged in the freelancer's trail (ASMP-23, Access field); financial-record retention (SC-24) governs how long the underlying Invoice and Payment records this feature reads remain available, which this feature has no control over and simply reflects.

## Non-Goals

- **Live, two-way accounting sync from the dashboard** -- Excluded per scope-boundaries.md (SC-07): the brief provides an export file (FEAT-22), not a live sync; this feature's totals stay a read-only view with no accounting-system connection of its own.
- **Converting or summing totals across currencies into one combined figure** -- Excluded per the feature's own Validation & Limits field and XBR-18: amounts in different currencies are never silently converted or added together, even though a single combined number would be a natural dashboard adjacency; each currency's totals are shown separately, always.
- **Recording, editing, or refunding a payment from the dashboard** -- Excluded per the dependency map: Invoice and Payment are Connected Entities marked read-only for this feature; payment recording and refunds are owned exclusively by FEAT-10 and FEAT-25, reachable only by navigating away from the dashboard into invoice/payment detail.
- **Automatic tax calculation or reporting from the dashboard's totals** -- Excluded per scope-boundaries.md (SC-16): tax depth is limited to the freelancer-configured tax label and rate captured elsewhere (FEAT-15); this feature surfaces totals only and performs no tax computation of its own.
