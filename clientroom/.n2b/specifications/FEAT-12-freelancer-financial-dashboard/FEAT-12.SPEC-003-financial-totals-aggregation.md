---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-12.SPEC-003
spec_name: Financial Totals Aggregation
spec_slug: financial-totals-aggregation
parent_feature: FEAT-12
parent_feature_name: Freelancer Financial Dashboard
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 40
acceptance_criteria_count: 20
---

# Logic/Rule Spec: Financial Totals Aggregation

## Overview

**Name:** Financial Totals Aggregation
**ID:** FEAT-12.SPEC-003
**Type:** Logic/Rule
**Purpose:** Derives earned, outstanding, and overdue totals -- and the paid/due/overdue status classification behind them -- from Invoice and Payment records, per currency, honoring refunds, reversals, and manually recorded payments.
**Parent Feature:** FEAT-12 -- Freelancer Financial Dashboard
**Governed Entity:** Financial Totals (a derived aggregate view -- not a stored entity; computed entirely from the Invoice and Payment records defined in the Feature Dependency Map, per the feature's Data Notes: "nothing is captured directly in this view")

## Scope and Non-Goals

**In Scope:**
- The derivation formula for earned, outstanding, and overdue totals, per currency, at three scopes: account-wide, per-client, and per-project
- The paid/due/overdue classification applied to each invoice, which drives both the aggregate figures and the per-invoice status badges on FEAT-12.SPEC-001 and FEAT-12.SPEC-002
- Currency-separation rules (XBR-18)
- Honoring refunds, partial refunds, reversals, disputes, and manually recorded payments in the earned figure (XBR-20, XBR-21, XBR-22)
- Which invoice and payment statuses are excluded from the totals entirely

**Non-Goals:**
- Rendering the totals on screen -- owned by FEAT-12.SPEC-001 and FEAT-12.SPEC-002, which reference this spec rather than re-deriving the formula
- Recomputing on a schedule or in response to an underlying change, and retrying on failure -- owned by FEAT-12.SPEC-004 (Dashboard Totals Refresh); this spec defines only the formula, not when it runs
- Determining the exact refunded or reversed amount on a given Payment -- excluded per the dependency map: Payment's refund/reversal fields are owned and written by FEAT-10 and FEAT-25; this spec reads those authoritative values rather than computing them
- Converting or summing totals across currencies into one combined figure -- excluded per the feature's Validation & Limits and XBR-18; each currency is aggregated and reported entirely separately

## Governed Entity

**Entity:** Financial Totals (derived aggregate view)
**Source:** Feature Dependency Map (computed from Invoice and Payment; scoped using Project)

| Field | Data Type | Description |
|-------|-----------|-------------|
| scope | enum (account / client / project) | Which slice of the roster this computation covers |
| currency | text | The currency this total block is computed for; one Financial Totals record exists per currency present within the scope (never combined, XBR-18) |
| earned_total | derived, number | Sum of amounts actually retained by Nadia across Succeeded and Recorded-manually payments in this scope and currency, net of any refunded or reversed amount |
| outstanding_total | derived, number | Sum of amounts on unpaid invoices in this scope and currency that are not yet past their due date |
| overdue_total | derived, number | Sum of amounts on unpaid invoices in this scope and currency that are past their due date |
| per_client_breakdown | derived, list | The same three totals (earned/outstanding/overdue), per currency, scoped to each individual client -- populates FEAT-12.SPEC-001's By Client list |
| per_project_breakdown | derived, list | The same three totals, per currency, scoped to each individual project within one client -- populates FEAT-12.SPEC-002's project narrowing |
| invoice_status_classification | derived, enum (Paid / Due / Overdue) | The at-a-glance status assigned to each invoice contributing to the totals; drives the status badges shared by FEAT-12.SPEC-001 and FEAT-12.SPEC-002 |
| computed_at | derived, timestamp | When this computation ran; carried forward unchanged by FEAT-12.SPEC-004 when a recompute fails, so the screens can keep showing "last known" totals |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-12.SPEC-001 | Dashboard Overview | On screen load and on every Client/Period filter change |
| FEAT-12.SPEC-002 | Client/Project Financial Drill-down | On screen load and on every Project/Period filter change |
| FEAT-12.SPEC-004 | Dashboard Totals Refresh | On every triggered recompute (invoice/payment/roster change, or a manual Retry from either screen) |

## Field Validation Rules

This entity captures no user input -- every field is entirely derived. Each field is addressed below to confirm it was considered, not accidentally skipped.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| scope | No validation -- always one of account / client / project, set by the requesting screen, never free text | Always | -- | -- | No |
| currency | No validation -- entirely derived from the Project.currency of the invoices in scope; see Defaults and Derivations | Always | -- | -- | No |
| earned_total | No validation -- entirely derived; see Defaults and Derivations | Always | -- | -- | No |
| outstanding_total | No validation -- entirely derived; see Defaults and Derivations | Always | -- | -- | No |
| overdue_total | No validation -- entirely derived; see Defaults and Derivations | Always | -- | -- | No |
| per_client_breakdown | No validation -- entirely derived; see Defaults and Derivations | Always | -- | -- | No |
| per_project_breakdown | No validation -- entirely derived; see Defaults and Derivations | Always | -- | -- | No |
| invoice_status_classification | No validation -- entirely derived per invoice; see Defaults and Derivations | Always | -- | -- | No |
| computed_at | No validation -- entirely derived (current time at successful computation); see Defaults and Derivations | Always | -- | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Currency separation (XBR-18) | currency, earned_total, outstanding_total, overdue_total | Each currency present in the scope produces its own complete Financial Totals record; no field on one currency's record is ever added to or converted into another currency's record | N/A -- this is an internal computation guarantee, never a user-facing error; there is no input path that could violate it |
| Mutually exclusive classification | invoice_status_classification, earned_total, outstanding_total, overdue_total | Each invoice's amount contributes to exactly one of earned_total, outstanding_total, or overdue_total in its currency -- never more than one bucket, and never zero for an invoice that is in scope and not excluded (see Business Rules for the excluded statuses) | N/A -- internal computation guarantee |
| Breakdown-to-aggregate consistency | per_client_breakdown, per_project_breakdown, earned_total, outstanding_total, overdue_total | The account-wide totals for a currency equal the sum of that currency's per_client_breakdown entries; a client's total equals the sum of its per_project_breakdown entries | N/A -- internal computation guarantee |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View account-wide financial totals | Nadia (Freelancer) | Always, her own account only | -- |
| View account-wide financial totals | Owen (Client Primary Contact) | Never | Screen not reachable at all -- see FEAT-12.SPEC-001's Access and Visibility for the exact experience |
| View account-wide financial totals | Priya (Client Reviewer Contact) | Never | Screen not reachable at all -- see FEAT-12.SPEC-001's Access and Visibility |
| View account-wide financial totals | Dana (Support Operator) | Always, read-only, only during an open support session (FEAT-31) | -- |
| View per-client/per-project financial detail | Nadia (Freelancer) | Always, her own clients and projects only | -- |
| View per-client/per-project financial detail | Owen (Client Primary Contact) | Never | Screen not reachable at all -- see FEAT-12.SPEC-002's Access and Visibility |
| View per-client/per-project financial detail | Priya (Client Reviewer Contact) | Never | Screen not reachable at all -- see FEAT-12.SPEC-002's Access and Visibility |
| View per-client/per-project financial detail | Dana (Support Operator) | Always, read-only, only during an open support session (FEAT-31) | -- |
| Narrow totals by client or period | Nadia (Freelancer) | Always | -- |
| Narrow totals by client or period | Dana (Support Operator) | Always, read-only, only during an open support session | -- |
| Narrow totals by client or period | Owen, Priya | Never | Filter controls not shown -- screen not reachable |
| Generate an accounting export from these totals | Nadia (Freelancer) | Always | -- |
| Generate an accounting export from these totals | Dana (Support Operator) | Never (XBR-29: support sessions exclude data/accounting exports) | Export control is not shown; a direct navigation attempt is blocked with "Support sessions cannot generate exports." (see FEAT-12.SPEC-001) |
| Generate an accounting export from these totals | Owen, Priya | Never | Export control not shown -- screen not reachable |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| currency | The distinct set of Project.currency values among invoices in scope; one Financial Totals record is derived per distinct currency found | Every computation | No |
| invoice_status_classification (per invoice in scope) | "Paid" when Invoice.status is Paid, Refunded, Partially refunded, or Disputed (XBR-21 keeps the Disputed marker alongside the original Paid record). "Overdue" when Invoice.status is Overdue, or Invoice.status is Sent/Payment pending and due_date has passed. "Due" when Invoice.status is Sent or Payment pending and due_date has not yet passed. Invoices with status Generated (not yet sent) or Corrected (superseded by a correcting invoice) are excluded from classification and from every total | On every computation | No |
| earned_total (per currency, per scope) | Sum, over invoices in scope classified "Paid," of the associated Payment.amount for Payments with status Succeeded or Recorded manually, minus the refunded or reversed portion FEAT-10/FEAT-25 record against that Payment (a fully reversed or fully refunded invoice contributes zero net; a partially refunded invoice contributes the remaining net amount) | On every computation | No |
| outstanding_total (per currency, per scope) | Sum, over invoices in scope classified "Due," of Invoice.total | On every computation | No |
| overdue_total (per currency, per scope) | Sum, over invoices in scope classified "Overdue," of Invoice.total | On every computation | No |
| per_client_breakdown / per_project_breakdown | The same earned/outstanding/overdue derivation, re-run with scope narrowed to each individual client or project | On every computation | No |
| computed_at | The current time at the moment a computation completes successfully | On every successful computation only -- left unchanged by FEAT-12.SPEC-004 on a failed recompute | No |

## Business Rules

- XBR-18: financial totals are shown per currency; amounts in different currencies are never converted or added together, at any scope (account, client, or project).
- XBR-20: an invoice is paid once and in full only; a refund cannot exceed the amount paid; these facts are enforced by FEAT-10 and simply reflected here as the net earned amount.
- XBR-21: when a paid invoice is later disputed, it keeps its "Paid" classification alongside the Disputed marker (owned by FEAT-25) unless and until the underlying Payment is actually reported Reversed, at which point the earned figure is reduced accordingly.
- XBR-22: Financial Dashboard totals are derived only from Invoice and Payment records, including refunds, reversals, and manually recorded payments -- no other data source contributes to earned/outstanding/overdue.
- Invoices with status Generated (drafted but never sent) carry no financial commitment visible to a client and are excluded from every total; a Corrected invoice is excluded in favor of the invoice that corrects it, so a correction is never double-counted.
- The "Due" vs. "Overdue" boundary is evaluated against the invoice's due_date in Nadia's own time zone (consistent with FEAT-15's time-zone handling and the day-boundary convention FEAT-11 uses for its own reminder schedule, XBR-15) -- an invoice becomes Overdue at the start of the day after its due date, regardless of whether FEAT-11 has yet set the Overdue status flag, so this rule's own due_date comparison never lags behind the true due date.
- The view spans the freelancer's full client roster with no hard limit on history depth (feature's Validation & Limits); this rule does not truncate or paginate the underlying computation -- pagination, if any, is a rendering concern for the consuming screen.

## Edge Cases

- **An invoice's due_date falls exactly on the current day (in Nadia's time zone)** -- Classified "Due," not "Overdue"; the boundary is exclusive of the due date itself, matching the day-3/day-10 reminder convention elsewhere in the product.
- **An invoice is exactly one day past due_date** -- Classified "Overdue" from the start of that next day, even if FEAT-11 has not yet processed its day-3 reminder or set the Overdue status flag.
- **A Partially refunded invoice** -- Remains classified "Paid"; earned_total includes only the net amount after the recorded partial refund, never the full original payment.
- **A fully Refunded invoice** -- Remains classified "Paid" for badge purposes (the client did pay at some point), but contributes zero to earned_total since the full amount was returned.
- **A Disputed invoice whose Payment has not yet been reported Reversed** -- Classified "Paid," and earned_total still includes the full net amount; only an actual Reversed report changes the figure.
- **A Disputed invoice whose Payment is later reported Reversed** -- earned_total for the affected scope(s) drops by that amount on the next computation; classification remains "Paid" per XBR-21's "alongside the original Paid record."
- **An invoice manually recorded as paid off-platform (FEAT-10)** -- Counted identically to a processor-confirmed payment in earned_total, per XBR-22's explicit inclusion of manually recorded payments.
- **An ad hoc invoice still in Generated status (drafted, not yet sent)** -- Excluded from every total; it becomes eligible for classification only once its status moves to Sent.
- **A corrected invoice and its correcting invoice both exist** -- Only the correcting invoice (whichever status it currently holds) contributes to totals; the original Corrected invoice is excluded to prevent double-counting the same underlying amount.
- **A client has invoices in three different currencies** -- Three separate Financial Totals records are produced for that client, one per currency; per_client_breakdown never merges them.
- **A project has zero invoices in scope after a filter narrows to it** -- The derivation returns an empty result set for that scope (zero across earned/outstanding/overdue in every currency); the consuming screen (FEAT-12.SPEC-002) renders this as its No-Results state, not as an error.
- **Two overlapping computations for the same scope run at effectively the same time** -- Each reads current source data independently; see FEAT-12.SPEC-004's Edge Cases for how the consuming automation resolves which result is kept.

## Acceptance Criteria

**FEAT-12.SPEC-003-AC-01:** Given Nadia has invoices in a single currency across three clients, when the account-wide computation runs, then earned_total, outstanding_total, and overdue_total are each returned as one figure for that currency, summed across all three clients.

**FEAT-12.SPEC-003-AC-02:** Given Nadia has invoices in two different currencies, when the account-wide computation runs, then two separate Financial Totals records are returned, one per currency, and neither is combined with the other.

**FEAT-12.SPEC-003-AC-03:** Given an invoice has status Sent with a due_date three days in the future, when classification runs, then it is classified "Due" and its amount contributes to outstanding_total.

**FEAT-12.SPEC-003-AC-04:** Given an invoice has status Sent with a due_date one day in the past, when classification runs, then it is classified "Overdue" and its amount contributes to overdue_total, regardless of whether FEAT-11 has yet set the Overdue status flag.

**FEAT-12.SPEC-003-AC-05:** Given an invoice's due_date is exactly today, when classification runs, then it is classified "Due," not "Overdue."

**FEAT-12.SPEC-003-AC-06:** Given an invoice has status Paid with a Payment of status Succeeded and no refund or reversal recorded, when the computation runs, then it is classified "Paid" and its full Payment.amount contributes to earned_total.

**FEAT-12.SPEC-003-AC-07:** Given an invoice has status Partially refunded, when the computation runs, then it remains classified "Paid" and earned_total includes only the net amount after the recorded partial refund.

**FEAT-12.SPEC-003-AC-08:** Given an invoice has status Refunded (in full), when the computation runs, then it remains classified "Paid" but contributes zero to earned_total.

**FEAT-12.SPEC-003-AC-09:** Given an invoice has status Disputed and its Payment has not been reported Reversed, when the computation runs, then it is classified "Paid" and its full net amount still contributes to earned_total.

**FEAT-12.SPEC-003-AC-10:** Given a previously Disputed invoice's Payment is subsequently reported Reversed, when the next computation runs, then earned_total for the affected scope decreases by that Payment's amount.

**FEAT-12.SPEC-003-AC-11:** Given a payment was recorded manually by Nadia (off-platform) with status Recorded manually, when the computation runs, then the invoice is classified "Paid" and the manually recorded amount contributes to earned_total identically to a processor-confirmed payment.

**FEAT-12.SPEC-003-AC-12:** Given an ad hoc invoice has status Generated (drafted, never sent), when the computation runs, then it is excluded from classification and from every total.

**FEAT-12.SPEC-003-AC-13:** Given an invoice has status Corrected and a correcting invoice exists for the same amount, when the computation runs, then only the correcting invoice contributes to totals and the Corrected invoice is excluded.

**FEAT-12.SPEC-003-AC-14:** Given a client's invoices span two currencies, when a per-client breakdown is computed, then two separate currency records are returned for that client and the account-wide total for each currency equals the sum of all clients' breakdowns in that currency.

**FEAT-12.SPEC-003-AC-15:** Given Nadia is the requesting user, when she requests account-wide or per-client/per-project totals, then the computation always proceeds for her own account.

**FEAT-12.SPEC-003-AC-16:** Given Owen (Client Primary Contact) has no route to this feature's screens, when any attempt to reach them occurs, then the totals are never computed for or shown to him.

**FEAT-12.SPEC-003-AC-17:** Given Dana (Support Operator) has an open support session on Nadia's account, when she requests account-wide, per-client, or per-project totals, then the computation runs identically to Nadia's own view, read-only.

**FEAT-12.SPEC-003-AC-18:** Given Dana (Support Operator) is in a support session, when she attempts to trigger an accounting export from these totals, then the action is denied with "Support sessions cannot generate exports."

**FEAT-12.SPEC-003-AC-19:** Given a project has no invoices in the requested scope, when the computation runs, then it returns an empty result (zero across earned/outstanding/overdue in every currency) rather than an error.

**FEAT-12.SPEC-003-AC-20:** Given the computation completes successfully, when computed_at is set, then it reflects the current time of that successful run, and remains unchanged by FEAT-12.SPEC-004 if a subsequent recompute attempt fails.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 9 | 9 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 14 | 14 |
| Defaults/Derivations | 7 | 7 |
| Business Rules | 7 | 7 |
| Edge Cases | 12 | 12 |
