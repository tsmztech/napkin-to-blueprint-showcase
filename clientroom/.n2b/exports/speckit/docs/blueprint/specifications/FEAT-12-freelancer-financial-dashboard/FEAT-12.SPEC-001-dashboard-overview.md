---
document_type: spec
spec_type: screen
spec_id: FEAT-12.SPEC-001
spec_name: Dashboard Overview
spec_slug: dashboard-overview
parent_feature: FEAT-12
parent_feature_name: Freelancer Financial Dashboard
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

# Screen Spec: Dashboard Overview

## Overview

**Name:** Dashboard Overview
**ID:** FEAT-12.SPEC-001
**Type:** Screen
**Purpose:** Nadia sees earned, outstanding, and overdue totals across her entire client roster at a glance, per currency, filterable by client or period, and can drill into any client for detail.
**Parent Feature:** FEAT-12 -- Freelancer Financial Dashboard

## Scope and Non-Goals

**In Scope:**
- Displaying aggregate earned/outstanding/overdue totals across every client, one self-contained block per currency present
- Narrowing the displayed totals to one client or a date range (period)
- A status badge/color treatment on the aggregate totals distinguishing paid, due, and overdue amounts
- Entry point into the per-client/project drill-down (FEAT-12.SPEC-002)
- Entry point into an individual overdue invoice's reminder history and pause control (FEAT-11, cross-feature)
- Entry point into the accounting export flow (FEAT-22, cross-feature)
- The new-account zero-state and its prompt toward a first proposal (FEAT-02, cross-feature)
- Read-only viewing for Dana (Support Operator) during a logged support session

**Non-Goals:**
- Computing the totals themselves -- delegated entirely to FEAT-12.SPEC-003 (Financial Totals Aggregation); this screen only renders what that rule returns
- Converting or summing totals across currencies into one combined figure -- excluded per the feature's Validation & Limits and XBR-18: each currency's totals are shown separately, always, never combined
- Recording, editing, or refunding a payment from this screen -- excluded per the dependency map: Invoice and Payment are read-only Connected Entities for this feature; those actions belong exclusively to FEAT-10 and FEAT-25, reachable only by navigating away from the dashboard
- Automatic tax calculation or reporting -- excluded per scope-boundaries.md (SC-16): this screen surfaces totals only

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Default entry (main navigation) | Nadia opens the "Financial Dashboard" / "Dashboard" navigation item | None -- loads with no filter (All Clients, All Time) |
| FEAT-20 (Onboarding / First-Run Setup) | Onboarding completes: a first client, project, and draft proposal exist | None -- lands on the normal (non-zero-state) dashboard per the Navigation Connections table |
| FEAT-31 (Operator Support Access) | Dana opens a logged support session on Nadia's account | Session is read-only and scoped to the one freelancer account being supported |
| FEAT-12.SPEC-002 (Client/Project Financial Drill-down) | Nadia taps the back arrow | The client/period filter active before drilling in is restored |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, her own account only | All actions: filter by client/period, drill into a client, open an overdue invoice, generate an accounting export, dismiss the zero-state | -- |
| Owen (Client Primary Contact) | No | No | Owen signs in only to the Client Portal (FEAT-05), which has no route to this screen; the Financial Dashboard exists solely within the freelancer's own account. A portal link manipulated to reach this address is rejected and Owen is redirected to his portal home (FEAT-05.SPEC-003) with no message indicating a freelancer-only screen exists |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- Priya's portal session has no route here and is redirected identically on any manipulated attempt |
| Dana (Support Operator) | Full screen, for the one freelancer account under an open support session; the export action is not shown | Filter by client/period, drill into a client, open an overdue invoice's reminder history; cannot generate an accounting export | The export control is not rendered for Dana. A direct navigation attempt into the export flow (FEAT-22) while in a support session is blocked with "Support sessions cannot generate exports." |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, Nadia lands on this screen (or Dana's session resumes its scoped view) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- since this screen has no unsaved input, nothing is preserved beyond the active filter selection, which is restored after re-authentication succeeds |

## Layout and Content

**Header:** Page title "Financial Dashboard." Below the title, two filter controls sit side by side: a Client filter (dropdown, default "All Clients," listing Nadia's active client roster from FEAT-01) and a Period filter (dropdown, default "All Time," with presets including "This Month" and "Last Month," plus a "Custom Range" option that opens a start/end date picker). To the right of the filters, an "Export" action button (Nadia only; hidden for Dana per Access and Visibility) navigates to the accounting export flow.

**Body:** Below the header, one self-contained total block per currency present across the filtered scope (never a combined figure across currencies, per XBR-18). Each currency's block shows three figures side by side -- Earned, Outstanding, Overdue -- each with its amount and the status color/badge treatment shared with FEAT-12.SPEC-002, driven by the classification FEAT-12.SPEC-003 produces.

Below the currency blocks, a "By Client" list: one row per client with financial activity in the filtered scope, each row showing that client's name and its own earned/outstanding/overdue mini-totals with the same badge treatment. A client row with at least one overdue invoice carries a distinct overdue indicator that opens directly into that invoice's reminder history and pause control.

**Footer:** None -- filters and the export action are in the header.

### Responsive Behavior

- **Compact breakpoint:** Client and Period filters stack vertically, full width, above the currency total blocks. Each currency block's three figures (Earned, Outstanding, Overdue) stack vertically within the block rather than sitting side by side. The By Client list remains a single column of full-width rows.
- **Medium size class and above:** Client and Period filters sit side by side as described. Each currency block's three figures sit side by side within the block. The By Client list remains single-column but each row gains more horizontal breathing room; no structural change beyond width.
- **Export button:** Always in the header regardless of breakpoint; at the compact breakpoint it collapses to an icon-only control with the same label available to assistive technology.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Client filter | Select a client (or "All Clients") | Requests totals from FEAT-12.SPEC-003 scoped to the selection | Screen enters Loading, then re-renders with filtered totals | Progress indicator during aggregation; totals and By Client list update to the new scope |
| Period filter | Select a preset or Custom Range | Requests totals from FEAT-12.SPEC-003 scoped to the selection | Screen enters Loading, then re-renders with filtered totals | Same as Client filter |
| Custom Range date pickers | Enter start/end dates | Applies the custom period once both dates are set | Screen enters Loading on confirmation | Same as Client filter; an incomplete range (only one date set) does not trigger a refetch |
| By Client list row | Tap | Navigate to FEAT-12.SPEC-002 (Client/Project Financial Drill-down) for that client | Screen closes | Transition to the drill-down screen for the tapped client |
| Overdue indicator on a client row | Tap | Navigate to that client's overdue invoice's reminder history and pause control (FEAT-11, cross-feature) | Screen closes | Transition to FEAT-11's invoice detail |
| Export button (Nadia only) | Tap | Navigate to the accounting export flow (FEAT-22, cross-feature) | Screen closes | Transition to FEAT-22 |
| Retry button (Error state) | Tap | Triggers FEAT-12.SPEC-004 (Dashboard Totals Refresh) to recompute now | Button shows a brief loading indicator | Success: totals update silently. Failure: banner remains, "Still unable to refresh. Showing your last known totals." |
| Clear Filter button (No-Results state) | Tap | Resets Client and Period filters to their defaults | Screen enters Loading, then re-renders with account-wide totals | Filters visibly reset; totals reload |
| Zero-state "Send your first proposal" prompt | Tap | Navigate to a new proposal draft (FEAT-02, cross-feature) | Screen closes | Transition to FEAT-02's proposal draft editor |
| Currency total blocks | -- | Display-only -- not interactive | None | -- |

### Accessibility Notes

- **Focus order:** Client filter -> Period filter -> (Custom Range date pickers, when shown) -> Export button -> each currency block's three figures in order (Earned, Outstanding, Overdue) -> each By Client row in list order -> Retry/Clear Filter button, when shown.
- **Dynamic announcements:** Entering the Loading state announces "Updating totals" to assistive technology. A successful refresh announces the new Earned/Outstanding/Overdue figures for the active scope. Entering the Error state announces the retry banner text. Entering the No-Results state announces the no-results message.
- **Status badges:** Paid/Due/Overdue status is never conveyed by color alone (ASMP-27) -- each badge carries a text label read by assistive technology alongside its color treatment.
- **Keyboard alternatives:** Every filter, list row, and button on this screen is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (zero-state) | Explanatory message that no invoices exist yet, with a prompt and action toward sending a first proposal; no currency blocks or By Client list shown | Screen loads and Nadia's account has never had any invoice in any status | Nadia's first invoice is generated (via any triggering path in FEAT-09) |
| Loading | Currency blocks and By Client list area show a lightweight progress indicator; filters remain interactive but disabled until the in-flight request completes | Screen first opens, or a filter selection changes, or FEAT-12.SPEC-004 recomputes for the active scope | Aggregation completes (success, no-results, or error) |
| Loaded (default) | Currency blocks and By Client list render the current totals for the active filter scope | Aggregation completes successfully with at least one matching total | A filter changes, an underlying invoice/payment changes elsewhere (silent background refresh via FEAT-12.SPEC-004), or connectivity is lost |
| No-Results | Distinct message: "No invoices match this filter." with a "Clear Filter" action; no currency blocks shown | The active Client/Period filter matches zero invoices, while the account overall has invoices | Filter is cleared or changed to a scope with matches |
| Error | The last successfully computed totals for the active scope remain visible, with a banner "Couldn't refresh your totals just now." and a Retry action | FEAT-12.SPEC-003's aggregation fails and FEAT-12.SPEC-004's automatic retries are exhausted for the active scope | Retry succeeds, or a filter change triggers a fresh aggregation attempt |
| Offline/Degraded | The most recently loaded totals for whichever filter was active when connectivity was lost remain visible, read-only, with a persistent banner "You're offline -- showing your last loaded totals." Changing the filter shows the same banner and does not attempt a new fetch; the newly selected filter is remembered | Connectivity is lost while this screen is open | Connectivity is restored -- the remembered filter (whether unchanged or changed while offline) is fetched automatically and the banner clears |

## Validation Rules

**Option B -- Inline (for simple validations not warranting a standalone spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Client filter | Must be a client from Nadia's own roster (FEAT-01) or "All Clients" | On selection | N/A -- the control only ever offers valid options; no free text entry is possible |
| Period filter | Must be a defined preset or a valid Custom Range | On selection | N/A -- presets are fixed options |
| Custom Range: end date | Must not be earlier than the start date | On selection of the end date | "End date can't be before the start date." |

No user input is captured for the totals themselves (per the feature's Validation & Limits field) -- every value on this screen is derived by FEAT-12.SPEC-003.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| By Client row tap | FEAT-12.SPEC-002 (Client/Project Financial Drill-down) | -- |
| Overdue indicator tap | Invoice reminder history and pause control | FEAT-11 (Automated Payment Reminders) |
| Export button tap | Accounting export flow | FEAT-22 (Accounting Export) |
| Zero-state prompt tap | New proposal draft | FEAT-02 (Proposal Creation & Sending) |

## Data Model

**Creates:** None -- this screen creates no data.
**Reads:** The computed Financial Totals produced by FEAT-12.SPEC-003 (earned/outstanding/overdue amounts per currency, per-client breakdowns, and the paid/due/overdue classification). Indirectly, via that rule: Invoice (status, amount, currency, due_date, project), Payment, and Project (project_name, client, currency) as defined in the Feature Dependency Map. Reads Client roster (FEAT-01) to populate the Client filter's options.
**Updates:** None.
**Deletes:** None.

## Business Rules

- All totals shown on this screen are computed exclusively by FEAT-12.SPEC-003; this screen never derives a figure independently, and never re-implements the earned/outstanding/overdue formula, the currency-separation rule, or the classification.
- XBR-18: amounts in different currencies are never converted or summed into one figure -- each currency renders in its own self-contained block.
- The zero-state, loading indicator, error/retry behavior, no-results messaging, and offline-degraded read-only viewing follow the shared convention declared in the Feature Breakdown Brief's Shared Context, identical in wording and behavior to FEAT-12.SPEC-002.
- The Export action is visible only to Nadia; Dana's read-only support session (FEAT-31, XBR-29) never exposes export generation, consistent with the Access Matrix.
- An invoice or payment change elsewhere in the product does not require Nadia to manually refresh -- FEAT-12.SPEC-004 recomputes affected totals in the background and this screen reflects the update silently if it is open and viewing the affected scope.

## Edge Cases

- **An underlying invoice or payment changes while Nadia is viewing this screen** -- No conflict: this screen only reads. FEAT-12.SPEC-004 recomputes the affected totals in the background and this screen's Loaded state re-renders the new figures without disrupting the active filter selection or list scroll position.
- **A filter (client or period) matches no invoices** -- The No-Results state appears with a "Clear Filter" action, distinct from the true zero-state message.
- **Aggregation across many clients takes noticeable time** -- The Loading state's lightweight progress indicator covers the gap; the Dashboard Comprehension success metric's 10-second target governs how quickly this must resolve.
- **Nadia double-taps the Export button** -- The second tap is ignored while navigation to FEAT-22 is already in progress.
- **Nadia navigates away and back** -- The screen re-fetches totals for whatever filter was last active rather than showing stale cached data, except when returning directly from FEAT-12.SPEC-002 via its back arrow, where the prior filter is restored without a full re-fetch if the data is still current.
- **A client the totals reference is archived (FEAT-01) while this screen is open** -- The archived client's historical totals remain included in the By Client list (archiving never erases records); no error occurs.
- **Custom Range with only a start date entered** -- No refetch is triggered until an end date is also set; the screen remains in its previous Loaded state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-002 (Client/Project Financial Drill-down) | Navigation (outbound) | Tapping a By Client row opens that client's detail |
| FEAT-12.SPEC-003 (Financial Totals Aggregation) | References (outbound) | This screen's every displayed figure is sourced from this rule |
| FEAT-12.SPEC-004 (Dashboard Totals Refresh) | Triggers (outbound) | The Retry action, and any background recompute while this screen is open, is handled by this automation |
| FEAT-01 (Client & Project Management) | References (inbound) | Client roster populates the Client filter; project grouping feeds the By Client list |
| FEAT-11 (Automated Payment Reminders) | Navigation (outbound) | Opening an overdue invoice from a client row |
| FEAT-22 (Accounting Export) | Navigation (outbound) | Export button |
| FEAT-02 (Proposal Creation & Sending) | Navigation (outbound) | Zero-state prompt |
| FEAT-20 (Onboarding / First-Run Setup) | Navigation (inbound) | Onboarding completion lands Nadia here |
| FEAT-31 (Operator Support Access) | References (inbound) | Dana's read-only support session views this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| dashboard_viewed | currency_count_shown, client_count_in_scope, aggregation_duration_ms | This screen finishes loading (Loaded state entered) | supports success-metrics.md: "Dashboard Comprehension" |
| dashboard_filtered | filter_type (client / period), aggregation_duration_ms | Nadia changes the Client or Period filter and the new totals load | supports success-metrics.md: "Dashboard Comprehension" |
| dashboard_empty_state_shown | -- | The zero-state renders | N/A -- this event measures the new-account onboarding funnel; no metric in success-metrics.md targets a zero-state account, since the Dashboard Comprehension metric is defined for an account with existing financial data |
| dashboard_no_results_shown | filter_type (client / period) | The No-Results state renders | N/A -- no Stage 2 metric targets filtered-empty outcomes specifically; this event supports usability monitoring of the filter controls |

## Acceptance Criteria

**FEAT-12.SPEC-001-AC-01:** Given Nadia opens the Financial Dashboard for the first time with an account that has never had any invoice, then she sees the zero-state message with a prompt to send her first proposal, and no currency blocks are shown.

**FEAT-12.SPEC-001-AC-02:** Given Nadia has invoices across three clients in one currency, when she opens the Financial Dashboard, then a single currency block appears showing Earned, Outstanding, and Overdue totals aggregated across all three clients, and the By Client list shows one row per client with financial activity.

**FEAT-12.SPEC-001-AC-03:** Given Nadia has invoices in two different currencies, when she opens the Financial Dashboard, then two separate self-contained currency blocks appear, and no figure ever combines amounts from both currencies.

**FEAT-12.SPEC-001-AC-04:** Given Nadia is viewing the dashboard with "All Clients" selected, when she selects a specific client in the Client filter, then the screen shows a brief loading indicator and re-renders with totals scoped to that client only.

**FEAT-12.SPEC-001-AC-05:** Given Nadia selects "Last Month" in the Period filter, when the selection completes, then the currency blocks and By Client list update to reflect only invoices relevant to last month.

**FEAT-12.SPEC-001-AC-06:** Given Nadia filters to a client with no invoices in the selected period, when the filtered totals load, then the No-Results state appears with the message "No invoices match this filter." and a "Clear Filter" action, distinct from the zero-state.

**FEAT-12.SPEC-001-AC-07:** Given Nadia is viewing the No-Results state, when she taps "Clear Filter", then both filters reset to "All Clients" / "All Time" and the account-wide totals load.

**FEAT-12.SPEC-001-AC-08:** Given Nadia taps a client row in the By Client list, when the tap registers, then she is navigated to FEAT-12.SPEC-002 (Client/Project Financial Drill-down) for that client.

**FEAT-12.SPEC-001-AC-09:** Given a client row shows an overdue indicator, when Nadia taps it, then she is navigated to that overdue invoice's reminder history and pause control (FEAT-11).

**FEAT-12.SPEC-001-AC-10:** Given Nadia taps the Export button, when the tap registers, then she is navigated to the accounting export flow (FEAT-22).

**FEAT-12.SPEC-001-AC-11:** Given Dana (Support Operator) is viewing the dashboard during a logged support session, when she looks for the Export button, then it is not shown, and a direct navigation attempt into the export flow is blocked with "Support sessions cannot generate exports."

**FEAT-12.SPEC-001-AC-12:** Given aggregation for the active scope fails and FEAT-12.SPEC-004's automatic retries are exhausted, when the Error state renders, then the last successfully computed totals remain visible alongside the banner "Couldn't refresh your totals just now." and a Retry action -- never a blank or misleading number.

**FEAT-12.SPEC-001-AC-13:** Given Nadia is viewing the Error state, when she taps Retry and the recomputation succeeds, then the banner clears and the refreshed totals render.

**FEAT-12.SPEC-001-AC-14:** Given Nadia loses connectivity while viewing the dashboard, when the connection drops, then the banner "You're offline -- showing your last loaded totals." appears and the most recently loaded totals remain viewable, read-only.

**FEAT-12.SPEC-001-AC-15:** Given Nadia is offline and changes the Client filter, when the selection is made, then no new fetch is attempted, the offline banner remains, and the newly selected filter is remembered for automatic fetch once connectivity returns.

**FEAT-12.SPEC-001-AC-16:** Given an invoice underlying Nadia's currently displayed totals is marked Paid elsewhere while this screen is open, when FEAT-12.SPEC-004 recomputes the affected totals, then the displayed figures update silently without disrupting Nadia's active filter selection.

**FEAT-12.SPEC-001-AC-17:** Given Nadia with no invoices taps the zero-state prompt, when the tap registers, then she is navigated to a new proposal draft (FEAT-02).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 6 (empty, loading, loaded, no-results, error, offline) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |
