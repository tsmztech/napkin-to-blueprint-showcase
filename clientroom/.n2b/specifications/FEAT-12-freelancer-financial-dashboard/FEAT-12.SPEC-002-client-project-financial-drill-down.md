---
document_type: spec
spec_type: screen
spec_id: FEAT-12.SPEC-002
spec_name: Client/Project Financial Drill-down
spec_slug: client-project-financial-drill-down
parent_feature: FEAT-12
parent_feature_name: Freelancer Financial Dashboard
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Client/Project Financial Drill-down

## Overview

**Name:** Client/Project Financial Drill-down
**ID:** FEAT-12.SPEC-002
**Type:** Screen
**Purpose:** Nadia opens one client to see the specific invoices behind its aggregate totals, and can narrow further to a single project within that client.
**Parent Feature:** FEAT-12 -- Freelancer Financial Dashboard

## Scope and Non-Goals

**In Scope:**
- Showing one client's earned/outstanding/overdue totals, per currency, matching the aggregate shown on FEAT-12.SPEC-001
- Narrowing within the client to a single project
- Listing the specific invoices behind the client's (or project's) totals, each with its paid/due/overdue status
- Entry point into an individual invoice's detail (FEAT-09, cross-feature) and into an overdue invoice's reminder history and pause control (FEAT-11, cross-feature)
- Read-only viewing for Dana (Support Operator) during a logged support session

**Non-Goals:**
- Computing the totals themselves -- delegated entirely to FEAT-12.SPEC-003 (Financial Totals Aggregation), identical to FEAT-12.SPEC-001
- Editing, sending, or voiding a proposal or invoice from this screen -- excluded per the dependency map: this feature's Connected Entities (Invoice, Payment, Project) are all read-only; those actions belong to FEAT-02 and FEAT-09
- Recording a payment, refund, or reversal from this screen -- excluded per the dependency map: Payment is a read-only Connected Entity here; those actions belong exclusively to FEAT-10 and FEAT-25
- Converting or summing this client's totals across currencies -- excluded per XBR-18, identical to FEAT-12.SPEC-001

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12.SPEC-001 (Dashboard Overview) | Nadia taps a By Client row | The selected client, and the Period filter active on FEAT-12.SPEC-001 at the time |
| FEAT-31 (Operator Support Access) | Dana opens a logged support session and views a client's detail from the dashboard | Session is read-only and scoped to the one freelancer account being supported |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, her own account only | Narrow to a project, open an invoice, open an overdue invoice's reminder history | -- |
| Owen (Client Primary Contact) | No | No | Owen signs in only to the Client Portal (FEAT-05), which has no route to this screen; a portal link manipulated to reach this address is rejected and Owen is redirected to his portal home (FEAT-05.SPEC-003) with no message indicating a freelancer-only screen exists |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- Priya's portal session has no route here and is redirected identically on any manipulated attempt |
| Dana (Support Operator) | Full screen, for the one freelancer account under an open support session | Narrow to a project, open an invoice, open an overdue invoice's reminder history -- all read-only | -- (this screen has no export control to deny; export is confined to FEAT-12.SPEC-001) |
| Unauthenticated | No | No | Redirected to the sign-in screen; the client context is not preserved across the redirect for security, and Nadia returns to FEAT-12.SPEC-001 after signing in |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- since this screen has no unsaved input, the selected client and project narrowing are restored after re-authentication succeeds |

## Layout and Content

**Header:** Back arrow (returns to FEAT-12.SPEC-001, restoring its prior filter). Client name as the screen title. Below the title, a "Period: {value}" chip carried from FEAT-12.SPEC-001 (or "All Time" if none was active), adjustable inline via the same preset/custom-range control described in that screen. A Project filter (dropdown, default "All Projects" for this client) sits alongside it.

**Body:** Below the header, one self-contained total block per currency present within the client's (or narrowed project's) scope -- Earned, Outstanding, Overdue -- using the same status color/badge treatment as FEAT-12.SPEC-001, per the Shared UI Patterns declared in the Feature Breakdown Brief. Below the totals, an invoice list: one row per invoice in scope, each showing the invoice's project name, amount and currency, due date, and a paid/due/overdue status badge driven by FEAT-12.SPEC-003's classification. A row for an Overdue invoice carries the same overdue indicator used on FEAT-12.SPEC-001.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** The Period chip and Project filter stack vertically below the client name. Each currency block's three figures stack vertically. Invoice list rows show project name and status badge on one line, amount/currency and due date on a second line within the same row.
- **Medium size class and above:** The Period chip and Project filter sit side by side. Each currency block's three figures sit side by side. Invoice list rows show all fields on a single line; no structural change beyond width.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12.SPEC-001, restoring the filter active before drilling in | Screen closes | Animated transition back to the dashboard |
| Period chip | Select a new preset or Custom Range | Requests totals from FEAT-12.SPEC-003 scoped to this client and the new period | Screen enters Loading, then re-renders | Progress indicator during aggregation; totals and invoice list update |
| Project filter | Select a project (or "All Projects") | Requests totals from FEAT-12.SPEC-003 scoped to this client and the selected project | Screen enters Loading, then re-renders | Same as Period chip |
| Invoice row (not overdue) | Tap | Navigate to that invoice's detail (FEAT-09, cross-feature) | Screen closes | Transition to FEAT-09's invoice detail |
| Overdue indicator on an invoice row | Tap | Navigate to that invoice's reminder history and pause control (FEAT-11, cross-feature) | Screen closes | Transition to FEAT-11's invoice detail |
| Retry button (Error state) | Tap | Triggers FEAT-12.SPEC-004 (Dashboard Totals Refresh) to recompute now | Button shows a brief loading indicator | Success: totals update silently. Failure: banner remains |
| Clear Filter button (No-Results state) | Tap | Resets the Project filter to "All Projects" (Period chip is left as carried from FEAT-12.SPEC-001) | Screen enters Loading, then re-renders | Filter visibly resets; totals reload |
| Currency total blocks | -- | Display-only -- not interactive | None | -- |

### Accessibility Notes

- **Focus order:** Back arrow -> Period chip -> Project filter -> each currency block's three figures in order -> each invoice row in list order -> Retry/Clear Filter button, when shown.
- **Dynamic announcements:** Entering Loading announces "Updating totals" to assistive technology. A successful refresh announces the client's updated Earned/Outstanding/Overdue figures. Entering Error announces the retry banner text. Entering No-Results announces the no-results message.
- **Status badges:** Paid/Due/Overdue status is never conveyed by color alone (ASMP-27) -- each badge carries a text label alongside its color treatment, identical to FEAT-12.SPEC-001.
- **Keyboard alternatives:** Every filter, row, and button on this screen is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Currency blocks and invoice list area show a lightweight progress indicator; filters remain visible but disabled until the in-flight request completes | Screen first opens, or the Period chip/Project filter changes, or FEAT-12.SPEC-004 recomputes for the active scope | Aggregation completes (success, no-results, or error) |
| Loaded (default) | Currency blocks and invoice list render the current totals for the active client/project/period scope | Aggregation completes successfully with at least one matching invoice | A filter changes, an underlying invoice/payment changes elsewhere (silent background refresh via FEAT-12.SPEC-004), or connectivity is lost |
| No invoices for this client yet | Message "This client has no invoices yet." with no currency blocks or invoice rows shown | The selected client (with "All Projects," "All Time") genuinely has zero invoices in any status | An invoice is generated for this client (via any triggering path in FEAT-09) |
| No-Results | Distinct message "No invoices match this filter." with a "Clear Filter" action | The active Project/Period scope matches zero invoices while the client overall has invoices | Filter is cleared or changed to a scope with matches |
| Error | The last successfully computed totals for the active scope remain visible, with a banner "Couldn't refresh your totals just now." and a Retry action | FEAT-12.SPEC-003's aggregation fails and FEAT-12.SPEC-004's automatic retries are exhausted for the active scope | Retry succeeds, or a filter change triggers a fresh aggregation attempt |
| Offline/Degraded | The most recently loaded totals and invoice list for the active scope remain visible, read-only, with a persistent banner "You're offline -- showing your last loaded totals." Changing a filter shows the same banner and does not attempt a new fetch | Connectivity is lost while this screen is open | Connectivity is restored -- the remembered filter is fetched automatically and the banner clears |

## Validation Rules

**Option B -- Inline (for simple validations not warranting a standalone spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Project filter | Must be a project belonging to this client (FEAT-01) or "All Projects" | On selection | N/A -- the control only ever offers valid options |
| Period chip | Same rule as FEAT-12.SPEC-001's Period filter, including the end-date-not-before-start-date check for Custom Range | On selection | "End date can't be before the start date." |

No user input is captured for the totals or invoice list themselves -- every value on this screen is derived by FEAT-12.SPEC-003.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-12.SPEC-001 (Dashboard Overview) | -- |
| Invoice row tap (not overdue) | Invoice detail | FEAT-09 (Invoice Generation & Sending) |
| Overdue indicator tap | Invoice reminder history and pause control | FEAT-11 (Automated Payment Reminders) |

## Data Model

**Creates:** None -- this screen creates no data.
**Reads:** The computed Financial Totals produced by FEAT-12.SPEC-003 (earned/outstanding/overdue amounts per currency, per-project breakdown, and the paid/due/overdue classification), scoped to the selected client. Indirectly, via that rule: Invoice (status, amount, currency, due_date, project), Payment, and Project (project_name, client, currency) as defined in the Feature Dependency Map. Reads the client's project roster (FEAT-01) to populate the Project filter's options.
**Updates:** None.
**Deletes:** None.

## Business Rules

- All totals and the invoice-level status badges on this screen are computed exclusively by FEAT-12.SPEC-003; this screen never re-derives the earned/outstanding/overdue formula or the currency-separation rule.
- XBR-18: amounts in different currencies within this client are never converted or summed into one figure -- each currency renders in its own self-contained block, identical to FEAT-12.SPEC-001.
- The loading indicator, error/retry behavior, no-results messaging, and offline-degraded read-only viewing follow the shared convention declared in the Feature Breakdown Brief's Shared Context, identical in wording and behavior to FEAT-12.SPEC-001.
- This screen has no export control -- accounting export is confined to FEAT-12.SPEC-001, consistent with the Access Matrix.
- An invoice or payment change elsewhere in the product does not require Nadia to manually refresh -- FEAT-12.SPEC-004 recomputes affected totals in the background and this screen reflects the update silently if it is open and viewing the affected client.

## Edge Cases

- **An underlying invoice or payment changes while Nadia is viewing this client's detail** -- No conflict: this screen only reads. FEAT-12.SPEC-004 recomputes the affected totals in the background and this screen's Loaded state re-renders the new figures without disrupting the active project/period selection.
- **A filter (project or period) matches no invoices** -- The No-Results state appears with a "Clear Filter" action, distinct from the "no invoices for this client yet" message.
- **The client is archived (FEAT-01) while Nadia is viewing its drill-down** -- The client's historical totals and invoice list remain visible read-only (archiving never erases records); no error occurs and no banner is required beyond what FEAT-01's own client detail shows.
- **Nadia navigates directly to a project's invoices, then that project is marked complete or cancelled elsewhere (FEAT-01/FEAT-25)** -- The project's stage label updates silently on the next refresh; its invoices and their statuses remain visible and unaffected by the stage change itself.
- **Nadia double-taps an invoice row** -- The second tap is ignored while navigation to FEAT-09 is already in progress.
- **Nadia narrows to a project, then Nadia's filter is manually cleared while a background recompute triggered by the prior scope is still in flight** -- The in-flight recompute's result is discarded on arrival if it no longer matches the current filter scope; only a recompute matching the currently selected scope updates the displayed totals.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-001 (Dashboard Overview) | Navigation (inbound) | Nadia arrives here from a By Client row |
| FEAT-12.SPEC-003 (Financial Totals Aggregation) | References (outbound) | This screen's every displayed figure and status badge is sourced from this rule |
| FEAT-12.SPEC-004 (Dashboard Totals Refresh) | Triggers (outbound) | The Retry action, and any background recompute while this screen is open, is handled by this automation |
| FEAT-01 (Client & Project Management) | References (inbound) | Project roster populates the Project filter |
| FEAT-09 (Invoice Generation & Sending) | Navigation (outbound) | Opening a specific invoice from the invoice list |
| FEAT-11 (Automated Payment Reminders) | Navigation (outbound) | Opening an overdue invoice's reminder history |
| FEAT-31 (Operator Support Access) | References (inbound) | Dana's read-only support session views this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| client_drilldown_viewed | invoice_count_in_scope, currency_count_shown, entry_source (dashboard_row) | This screen finishes loading (Loaded state entered) | N/A -- the drill-down view is a secondary action after the moment the Dashboard Comprehension metric targets (Nadia stating her totals within 10 seconds of opening the dashboard, FEAT-12.SPEC-001); no metric in success-metrics.md targets drill-down usage specifically |
| client_drilldown_filtered | filter_type (project / period) | Nadia changes the Project or Period filter and the new totals load | N/A -- same reason as client_drilldown_viewed |

## Acceptance Criteria

**FEAT-12.SPEC-002-AC-01:** Given Nadia taps a client row on FEAT-12.SPEC-001, when this screen loads, then it shows that client's earned/outstanding/overdue totals per currency and a list of that client's invoices with paid/due/overdue status badges.

**FEAT-12.SPEC-002-AC-02:** Given Nadia is viewing a client's drill-down with "All Projects" selected, when she selects one specific project in the Project filter, then the totals and invoice list narrow to that project only.

**FEAT-12.SPEC-002-AC-03:** Given Nadia opens the drill-down for a client that has never had an invoice, then she sees "This client has no invoices yet." with no currency blocks or invoice rows.

**FEAT-12.SPEC-002-AC-04:** Given Nadia narrows to a project with no invoices in the carried-over period, when the filtered totals load, then the No-Results state appears with "No invoices match this filter." and a "Clear Filter" action.

**FEAT-12.SPEC-002-AC-05:** Given Nadia is viewing the No-Results state, when she taps "Clear Filter", then the Project filter resets to "All Projects" and the client's totals for that period reload.

**FEAT-12.SPEC-002-AC-06:** Given Nadia taps an invoice row that is not overdue, when the tap registers, then she is navigated to that invoice's detail (FEAT-09).

**FEAT-12.SPEC-002-AC-07:** Given an invoice row shows an overdue indicator, when Nadia taps it, then she is navigated to that invoice's reminder history and pause control (FEAT-11).

**FEAT-12.SPEC-002-AC-08:** Given Nadia taps the back arrow, when the tap registers, then she returns to FEAT-12.SPEC-001 with the filter that was active before she drilled in restored.

**FEAT-12.SPEC-002-AC-09:** Given a client has invoices in two currencies, when Nadia views its drill-down, then two separate self-contained currency blocks appear and no figure combines amounts from both.

**FEAT-12.SPEC-002-AC-10:** Given aggregation for the active scope fails and FEAT-12.SPEC-004's automatic retries are exhausted, when the Error state renders, then the last successfully computed totals remain visible alongside a Retry action -- never a blank or misleading number.

**FEAT-12.SPEC-002-AC-11:** Given Nadia loses connectivity while viewing this client's drill-down, when the connection drops, then the banner "You're offline -- showing your last loaded totals." appears and the most recently loaded totals and invoice list remain viewable, read-only.

**FEAT-12.SPEC-002-AC-12:** Given Dana (Support Operator) is viewing a client's drill-down during a logged support session, when she looks at the screen, then all actions available to her are read-only and no export control is present.

**FEAT-12.SPEC-002-AC-13:** Given an invoice underlying this client's currently displayed totals is refunded elsewhere while this screen is open, when FEAT-12.SPEC-004 recomputes the affected totals, then the displayed figures update silently without disrupting Nadia's active project/period selection.

**FEAT-12.SPEC-002-AC-14:** Given the client Nadia is viewing is archived (FEAT-01) while this screen is open, when the archive completes, then the client's historical totals and invoices remain visible, read-only, with no error shown.

**FEAT-12.SPEC-002-AC-15:** Given Nadia clears her project filter while an earlier background recompute for the previously narrowed project is still in flight, when that stale recompute's result arrives, then it is discarded because it no longer matches the currently selected scope.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 6 (loading, loaded, empty-for-client, no-results, error, offline) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
