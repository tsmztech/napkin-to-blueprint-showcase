# Feature Specification: Freelancer Financial Dashboard

**Blueprint feature:** FEAT-12
**Priority tier:** Core
**Build order:** 021 of 33
**Depends on:** FEAT-01, FEAT-09, FEAT-10, FEAT-15
**Blueprint source:** `docs/blueprint/specifications/FEAT-12-freelancer-financial-dashboard/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Dashboard Overview (Priority: P1)

Nadia sees earned, outstanding, and overdue totals across her entire client roster at a glance, per currency, filterable by client or period, and can drill into any client for detail.

**Acceptance Scenarios:**

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

### User Story 2 - Client/Project Financial Drill-down (Priority: P1)

Nadia opens one client to see the specific invoices behind its aggregate totals, and can narrow further to a single project within that client.

**Acceptance Scenarios:**

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

### User Story 3 - Financial Totals Aggregation (Priority: P1)

Derives earned, outstanding, and overdue totals -- and the paid/due/overdue status classification behind them -- from Invoice and Payment records, per currency, honoring refunds, reversals, and manually recorded payments.

**Acceptance Scenarios:**

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

### User Story 4 - Dashboard Totals Refresh (Priority: P1)

Recomputes the affected Financial Totals whenever an underlying invoice or payment changes elsewhere in the product, and falls back to the last successfully computed totals with retry if a recomputation attempt fails.

**Acceptance Scenarios:**

**FEAT-12.SPEC-004-AC-01:** Given an invoice Nadia sent moves to status Paid (FEAT-10), when this automation fires, then the affected client's and the account-wide totals are recomputed via FEAT-12.SPEC-003 and the held totals are updated.

**FEAT-12.SPEC-004-AC-02:** Given Nadia is viewing FEAT-12.SPEC-001 with a client's invoice marked Paid elsewhere while she watches, when the recompute completes, then the dashboard's displayed totals update silently without disrupting her active filter.

**FEAT-12.SPEC-004-AC-03:** Given a payment is reported Reversed by FEAT-25 against a previously Disputed invoice, when this automation fires, then the affected scope's earned_total is recomputed downward to reflect the reversal.

**FEAT-12.SPEC-004-AC-04:** Given Nadia adds a new client (FEAT-01) and its first invoice is later generated, when this automation fires on that invoice event, then a first-time computed result is held for that client -- this automation is not fired by the client's addition itself, only by the invoice event that follows it.

**FEAT-12.SPEC-004-AC-05:** Given a recomputation attempt fails, when the automation retries automatically, then it retries up to platform parameter: `dashboard-aggregation-retry-count` times spaced platform parameter: `dashboard-aggregation-retry-interval` apart before surfacing any failure to the user.

**FEAT-12.SPEC-004-AC-06:** Given all automatic retries for a scope are exhausted while FEAT-12.SPEC-001 is open on that scope, when the final retry fails, then the last successfully computed totals remain visible with the Error state's Retry control -- never a blank or misleading number.

**FEAT-12.SPEC-004-AC-07:** Given all automatic retries are exhausted while no dashboard screen is open, when Nadia next opens FEAT-12.SPEC-001 on the affected scope, then she sees the Error state immediately, with the last successfully computed totals and a Retry control.

**FEAT-12.SPEC-004-AC-08:** Given Nadia taps Retry on FEAT-12.SPEC-002's Error state, when the immediate recomputation succeeds, then the held totals update, computed_at advances, and the Error state clears.

**FEAT-12.SPEC-004-AC-09:** Given Nadia taps Retry and the immediate recomputation also fails, when the failure occurs, then the banner updates to "Still unable to refresh. Showing your last known totals." and the Retry control remains.

**FEAT-12.SPEC-004-AC-10:** Given a manual Retry is tapped while an automatic retry is already scheduled for the same scope, when the manual attempt runs, then it executes immediately rather than waiting for the scheduled automatic attempt, and the automatic schedule continues unaffected if the manual attempt fails.

**FEAT-12.SPEC-004-AC-11:** Given invoices for two different clients change at effectively the same time, when both trigger this automation, then each client's recompute runs independently and the account-wide aggregate reflects both changes once each has completed, regardless of completion order.

**FEAT-12.SPEC-004-AC-12:** Given a recompute for one scope is still in flight when a newer trigger for the same scope fires, when both runs eventually complete, then only the result with the later read timestamp replaces the held totals, even if the older run happens to finish last.

**FEAT-12.SPEC-004-AC-13:** Given a client underlying a triggering change is archived mid-recompute (FEAT-01), when the recompute completes, then it succeeds normally and the archived client's totals remain included.

**FEAT-12.SPEC-004-AC-14:** Given a project's currency is set for the first time (FEAT-15) and its first invoice is subsequently generated, when this automation fires on that invoice event, then the new currency's scope is computed and its result is held for that project going forward -- the currency assignment itself does not fire this automation.

**FEAT-12.SPEC-004-AC-15:** Given an invoice's due date is changed before sending (FEAT-09), when this automation fires, then the affected totals are recomputed to reflect the new due date's Due/Overdue classification.

**FEAT-12.SPEC-004-AC-16:** Given a recomputation succeeds but produces figures identical to the currently held result, when the run completes, then computed_at still advances and no visible change appears on any open screen.

### Edge Cases

- **FEAT-12.SPEC-001 (Dashboard Overview):** The screen only reads, so changes to underlying invoices or payments re-render via the background recompute (FEAT-12.SPEC-004) without conflict. A filter with no match shows a No-Results state with Clear Filter (distinct from the zero-state), slow aggregation is covered by the Loading indicator against the 10-second target, and double taps on Export are ignored. Source: `docs/blueprint/specifications/FEAT-12-freelancer-financial-dashboard/FEAT-12.SPEC-001-dashboard-overview.md` (section: Edge Cases)
- **FEAT-12.SPEC-002 (Client/Project Financial Drill-down):** The drill-down is read-only and re-renders on background recomputes, a filter with no match shows No-Results with Clear Filter, and an archived client's historical totals and invoices remain visible read-only. A project completed or cancelled elsewhere updates its stage label silently on next refresh. Source: `docs/blueprint/specifications/FEAT-12-freelancer-financial-dashboard/FEAT-12.SPEC-002-client-project-financial-drill-down.md` (section: Edge Cases)
- **FEAT-12.SPEC-003 (Financial Totals Aggregation):** An invoice due on the current day in the freelancer's time zone is Due (the boundary is exclusive of the due date), and one day past due is Overdue from the start of the next day even before FEAT-11 sets its flag. A partially refunded invoice stays Paid and counts only the net amount, and a fully refunded one stays Paid for badge purposes but contributes zero to earned. Source: `docs/blueprint/specifications/FEAT-12-freelancer-financial-dashboard/FEAT-12.SPEC-003-financial-totals-aggregation.md` (section: Edge Cases)
- **FEAT-12.SPEC-004 (Dashboard Totals Refresh):** Recomputes for different clients run independently and the account-wide aggregate incorporates whichever committed, and each run tags its result with its read time so only a later result replaces the current one. Exhausted retries are recorded silently since the feature sends no notifications, and archiving a client mid-recompute does not exclude it from account-wide totals. Source: `docs/blueprint/specifications/FEAT-12-freelancer-financial-dashboard/FEAT-12.SPEC-004-dashboard-totals-refresh.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-12.SPEC-001** (Dashboard Overview) as specified: Nadia sees earned, outstanding, and overdue totals across her entire client roster at a glance, per currency, filterable by client or period, and can drill into any client for detail. Full spec: `docs/blueprint/specifications/FEAT-12-freelancer-financial-dashboard/FEAT-12.SPEC-001-dashboard-overview.md`
- **FR-002**: The system MUST implement **FEAT-12.SPEC-002** (Client/Project Financial Drill-down) as specified: Nadia opens one client to see the specific invoices behind its aggregate totals, and can narrow further to a single project within that client. Full spec: `docs/blueprint/specifications/FEAT-12-freelancer-financial-dashboard/FEAT-12.SPEC-002-client-project-financial-drill-down.md`
- **FR-003**: The system MUST implement **FEAT-12.SPEC-003** (Financial Totals Aggregation) as specified: Derives earned, outstanding, and overdue totals -- and the paid/due/overdue status classification behind them -- from Invoice and Payment records, per currency, honoring refunds, reversals, and manually recorded payments. Full spec: `docs/blueprint/specifications/FEAT-12-freelancer-financial-dashboard/FEAT-12.SPEC-003-financial-totals-aggregation.md`
- **FR-004**: The system MUST implement **FEAT-12.SPEC-004** (Dashboard Totals Refresh) as specified: Recomputes the affected Financial Totals whenever an underlying invoice or payment changes elsewhere in the product, and falls back to the last successfully computed totals with retry if a recomputation attempt fails. Full spec: `docs/blueprint/specifications/FEAT-12-freelancer-financial-dashboard/FEAT-12.SPEC-004-dashboard-totals-refresh.md`

### Key Entities

- Invoice (read)
- Payment (read)
- Project (read)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Nadia can state her total earned, outstanding, and overdue amounts across all clients within 10 seconds of opening the dashboard (metric: Dashboard Comprehension). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Dashboard views, client drill-down views, empty-state displays and filter use are each observable as distinct signals (dashboard_viewed, client_drilldown_viewed, dashboard_empty_state_shown, dashboard_filtered). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-21**: Dashboard totals appear within roughly 1-2 seconds. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-19**: Money, dates and time zones are locale-aware and never hard-coded, so totals are never merged across currencies. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-25**: Correctness of financial records applies to the totals derived from them. Full register: `docs/blueprint/features/assumptions-constraints.md`
