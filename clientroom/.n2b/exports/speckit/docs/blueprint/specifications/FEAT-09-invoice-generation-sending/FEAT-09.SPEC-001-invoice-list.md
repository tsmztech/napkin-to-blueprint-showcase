---
document_type: spec
spec_type: screen
spec_id: FEAT-09.SPEC-001
spec_name: Invoice List
spec_slug: invoice-list
parent_feature: FEAT-09
parent_feature_name: Invoice Generation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Invoice List

## Overview

**Name:** Invoice List
**ID:** FEAT-09.SPEC-001
**Type:** Screen
**Purpose:** Lets Nadia browse a project's invoices and lets Owen browse his own company's invoices, each scoped to what their role can see, with a clear "no invoices issued" state before the first one exists.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending

## Scope and Non-Goals

**In Scope:**
- Nadia's per-project invoice list, reached from Project Detail (FEAT-01.SPEC-005)
- Owen's own-company invoice list, scoped across all of his company's projects with one freelancer
- The empty state shown before a project's or company's first invoice exists
- Selecting an invoice to open its detail (FEAT-09.SPEC-002)
- Nadia's entry point into issuing an ad-hoc invoice or credit note (FEAT-09.SPEC-003)

**Non-Goals:**
- Invoice detail content (amount, status, reminder history, pay link, download) -- owned entirely by FEAT-09.SPEC-002 (Invoice Detail); this screen shows only summary rows
- Issuing or recording an ad-hoc invoice or credit note -- this screen only provides the entry point; the form and recording logic belong to FEAT-09.SPEC-003 and FEAT-09.SPEC-005
- Cross-client search across a freelancer's whole roster -- excluded per scope-boundaries.md's Deferral Notes (Global Search Across Clients & Projects is a v1-phase feature, FEAT-28); this screen lists only the invoices of one project (Nadia) or one client company (Owen)
- Financial totals or aggregation across invoices -- owned by Freelancer Financial Dashboard (FEAT-12); this screen lists individual invoice rows, not sums

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-005 (Project Detail) | Nadia opens the project's invoices area | Project reference -- list scoped to that project's invoices |
| FEAT-12 (Freelancer Financial Dashboard) | Nadia drills into a client from the dashboard | Client reference -- list scoped to all of that client's invoices across its projects |
| FEAT-05.SPEC-003 (Portal Home) | Owen opens "Invoices" from his portal home | Client Contact identity -- list scoped to his own company's invoices, own-only |
| FEAT-09.SPEC-010 (Invoice Issued & Copy Confirmation Notification) | Owen taps "View all invoices" from an invoice email | Client Contact identity -- same own-company scope as the portal entry |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full list for the scoped project or client -- every invoice regardless of status | Open any row; issue a new ad-hoc invoice or credit note | -- |
| Owen (Client Primary Contact) | Full list of his own company's invoices, own-only, across all of that company's projects with this freelancer | Open any of his own company's invoice rows | -- |
| Priya (Client Reviewer Contact) | None -- per FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules), invoice content is hidden entirely from Reviewer contacts | None | The "Invoices" entry point is not shown anywhere in Priya's portal navigation; a direct link is redirected to her portal home with no error message, consistent with FEAT-09.SPEC-006 |
| Dana (Support Operator) | Full list, read-only, inside a logged support session (FEAT-31) | View only -- no row action creates, sends, or downloads anything | The "New Invoice" action is not rendered for Dana; selecting a row opens FEAT-09.SPEC-002 in its own read-only mode |
| Unauthenticated | No | No | Redirected to FEAT-05.SPEC-001 (Request Sign-In Link) if a client contact, or to sign-in if Nadia; no invoice data is ever rendered before authentication |
| Expired session | No | No | Client contact: FEAT-05.SPEC-002's expired-link explanation with a one-tap fresh-link request. Nadia: redirected to sign-in; the list's scroll position is not preserved across re-authentication |

## Layout and Content

**Header:** Screen title -- "Invoices" for Nadia (with the project or client name shown beneath it, depending on entry context) or "Your invoices" for Owen. Nadia's header includes a "New Invoice" action (top-right) that opens FEAT-09.SPEC-003.

**Body:** A vertical list of invoice summary card rows (the shared pattern also used at full density on FEAT-09.SPEC-002), each row showing: invoice number, the triggering event's short label (Deposit, Milestone: {name}, On completion, Ad hoc, Credit note), amount and currency, a status badge, and the due date. Rows are ordered newest-issued first. Nadia's list additionally shows the project or client's name column when scoped from the dashboard (multi-project context); Owen's list shows which project each invoice belongs to, since his scope spans every project with this freelancer.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Rows stack their contents vertically (invoice number and status badge on the first line, amount/due date on the second); the list scrolls independently of the header.
- **Medium size class and above:** Rows lay out horizontally in a single line (number, project/client, amount, status, due date), capped at a consistent platform-wide content width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Invoice row | Tap | Navigate to FEAT-09.SPEC-002 (Invoice Detail) for that invoice | Screen transitions to detail | Standard navigation transition |
| "New Invoice" action (Nadia only) | Tap | Navigate to FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance) | Screen transitions to the issuance form | Standard navigation transition |
| Status badge | Display only | None | None | Communicates status by label and icon, never colour alone |
| List (on reopen) | Screen regains focus after being backgrounded | Re-fetches the current list | Rows refresh to current status | Rows update silently if unchanged; a changed status shows briefly highlighted |

### Accessibility Notes

- **Focus order:** Header title -> "New Invoice" action (Nadia only) -> invoice rows in display order.
- **Status announcements:** A status badge that changes while the list is open (e.g., an invoice becomes Overdue) is announced to assistive technology as "{invoice number} is now {status}."
- **Keyboard alternatives:** Every row and action is reachable and activatable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | "No invoices issued" message, with a "New Invoice" action for Nadia (Owen and Dana see the message with no action) | The scoped project or client has zero invoices | The first invoice is generated or issued |
| Loading | Skeleton rows in place of invoice cards | Screen first opens, or scope changes | Data finishes loading |
| Populated | Rows as described in Layout and Content | Loading completes with at least one invoice | Scope changes or screen closes |
| Error | Error banner "Couldn't load invoices. Check your connection and try again." with Retry | The list fails to load | User taps Retry and the load succeeds |
| Offline/Degraded | A "You're offline -- reconnect to see the latest invoices" banner sits above the last successfully loaded rows, which remain visible but are marked as possibly out of date | Connectivity is lost while the screen is open or being opened | Connectivity returns and the list re-fetches silently |

## Validation Rules

Not applicable -- this screen has no user input beyond navigation and selection.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Invoice row tap | FEAT-09.SPEC-002 (Invoice Detail) | -- |
| "New Invoice" tap (Nadia) | FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance) | -- |

## Data Model

**Creates:** None.
**Reads:** Invoice -- `invoice_number`, `project`, `triggering_event`, `amount`, `currency`, `total`, `status`, `due_date`, scoped to the current Project (Nadia's project entry) or Client (Nadia's dashboard drill-down and Owen's own-company scope).
**Updates:** None.
**Deletes:** None.

## Business Rules

- FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules) governs which roles reach this screen at all and what each sees; this screen's Access and Visibility table is consistent with that spec.
- Owen's scope is Own-only across every project his company has with this freelancer (per the Access Matrix), not limited to a single project, since a client contact's relationship spans the whole client company.
- The empty state is functionally identical whether reached from a brand-new project or a brand-new client -- "no invoices issued" never distinguishes the reason.

## Edge Cases

- **Nadia's project has invoices but none match the current status filter (if one is applied elsewhere in the UI)** -- Not applicable: this screen defines no filter of its own; every invoice in scope is always shown.
- **A new invoice is generated automatically while the list is open** -- The list re-fetches on the same silent-refresh basis as any status change (per Interactions) and the new row appears without the user reloading the screen manually.
- **Owen's company has invoices across three different projects** -- All are shown in one list, each row carrying its own project label, since his scope is company-wide rather than per-project.
- **The scoped project or client is archived after invoices were issued** -- The list remains reachable and fully populated; archiving a project (FEAT-01) does not remove or hide its invoice history.
- **Two of Nadia's sessions view the same project's invoice list while a third session issues a new invoice** -- No conflict: this is a read-only list with no write path of its own, so both sessions simply refresh to show the new row; there is no concurrent-edit conflict to resolve here, since a list never sets `status` or any other Invoice field. Editing an individual invoice's contention is handled entirely on FEAT-09.SPEC-002.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-002 (Invoice Detail) | Navigation (outbound) | Selecting a row opens that invoice's detail |
| FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance) | Navigation (outbound) | Nadia's "New Invoice" action opens the issuance form |
| FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules) | References (inbound) | Governs the Access and Visibility table above |
| FEAT-01.SPEC-005 (Project Detail) | Navigation (inbound) | Nadia arrives from a project's invoices area |
| FEAT-05.SPEC-003 (Portal Home) | Navigation (inbound) | Owen arrives from his portal home |
| FEAT-09.SPEC-010 (Invoice Issued & Copy Confirmation Notification) | Navigation (inbound) | Owen arrives from an invoice email's "View all invoices" link |
| FEAT-12 (Freelancer Financial Dashboard) | Navigation (inbound) | Nadia arrives from a client drill-down |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| invoice_list_viewed | scope (project / client / own-company), invoice count, viewer role | List finishes loading with a populated or empty state | N/A -- no success-metrics.md metric tracks browsing this list directly; retained as the entry-point signal for FEAT-09.SPEC-002 opens |
| invoice_list_empty_state_shown | scope | List loads with zero invoices | N/A -- no metric measures the empty state; retained to distinguish "no invoices yet" from a load failure in product telemetry |
| invoice_list_row_selected | invoice reference, scope | User taps an invoice row | supports success-metrics.md: "Invoice Auto-Generation Accuracy" (a row selected and opened is the precondition for Nadia or Owen ever noticing an amount, currency, or tax discrepancy, which this metric measures downstream on FEAT-09.SPEC-002) |

## Acceptance Criteria

**FEAT-09.SPEC-001-AC-01:** Given Nadia opens the invoices area of a project with three invoices, when the screen loads, then all three appear as rows ordered newest-issued first.

**FEAT-09.SPEC-001-AC-02:** Given a project has never had an invoice, when Nadia opens its invoices area, then she sees "No invoices issued" with a "New Invoice" action.

**FEAT-09.SPEC-001-AC-03:** Given Owen opens "Invoices" from his portal home and his company has invoices across two different projects, when the screen loads, then both projects' invoices appear in one list, each row labeled with its project.

**FEAT-09.SPEC-001-AC-04:** Given Priya is signed into her portal, when she looks for an "Invoices" entry, then none is shown anywhere in her navigation.

**FEAT-09.SPEC-001-AC-05:** Given Dana is inside a logged support session, when she opens the invoice list, then she sees every invoice read-only with no "New Invoice" action.

**FEAT-09.SPEC-001-AC-06:** Given Nadia taps an invoice row, when the tap registers, then she lands on FEAT-09.SPEC-002 for that invoice.

**FEAT-09.SPEC-001-AC-07:** Given Nadia taps "New Invoice", when the tap registers, then she lands on FEAT-09.SPEC-003.

**FEAT-09.SPEC-001-AC-08:** Given the list fails to load, when the failure occurs, then Nadia sees "Couldn't load invoices. Check your connection and try again." with a Retry action.

**FEAT-09.SPEC-001-AC-09:** Given Owen loses connectivity while viewing his invoice list, when the loss occurs, then the last-loaded rows remain visible under a "You're offline" banner and refresh silently once connectivity returns.

**FEAT-09.SPEC-001-AC-10:** Given a new invoice is generated automatically while Nadia has the list open, when generation completes, then the new row appears in the list without a manual reload.

**FEAT-09.SPEC-001-AC-11:** Given Owen's session expires while he is on this screen, when he attempts to interact with it, then he sees FEAT-05.SPEC-002's expired-link explanation with a one-tap way to request a fresh link.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 5 (empty, loading, populated, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
