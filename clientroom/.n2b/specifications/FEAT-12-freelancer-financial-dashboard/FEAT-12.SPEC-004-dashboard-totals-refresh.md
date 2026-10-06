---
document_type: spec
spec_type: automation
spec_id: FEAT-12.SPEC-004
spec_name: Dashboard Totals Refresh
spec_slug: dashboard-totals-refresh
parent_feature: FEAT-12
parent_feature_name: Freelancer Financial Dashboard
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 16
---

# Automation Spec: Dashboard Totals Refresh

## Overview

**Name:** Dashboard Totals Refresh
**ID:** FEAT-12.SPEC-004
**Type:** Automation
**Purpose:** Recomputes the affected Financial Totals whenever an underlying invoice or payment changes elsewhere in the product, and falls back to the last successfully computed totals with retry if a recomputation attempt fails.
**Parent Feature:** FEAT-12 -- Freelancer Financial Dashboard

## Scope and Non-Goals

**In Scope:**
- Deciding when to recompute Financial Totals: on an underlying invoice or payment change, and on a manual Retry from either dashboard screen
- Scoping each recompute to the affected client/project and the account-wide aggregate, using FEAT-12.SPEC-003's formula
- Retrying automatically on failure and preserving the last successfully computed totals, per currency, so a viewing screen never shows a blank or misleading number
- Silently refreshing FEAT-12.SPEC-001 and FEAT-12.SPEC-002 when they are open and viewing an affected scope

**Non-Goals:**
- The derivation formula itself (what counts as earned, outstanding, or overdue, and the paid/due/overdue classification) -- owned entirely by FEAT-12.SPEC-003; this automation only decides when to invoke it and what to do with the result
- Notifying Nadia by email or in-app when a recompute succeeds or fails -- excluded per the feature's Communications field: "this is a self-initiated view; it sends no notifications itself." A failure surfaces only as the on-screen Error state defined in FEAT-12.SPEC-001 and FEAT-12.SPEC-002, not as a separate notification
- Generating the accounting export file -- excluded per scope-boundaries.md (SC-07); that is a distinct, on-demand capability owned by FEAT-22, unrelated to this automation's background recompute
- Correcting or reversing the underlying Invoice or Payment records themselves -- excluded per the dependency map: those records are read-only Connected Entities for this feature; corrections belong to FEAT-09, FEAT-10, and FEAT-25

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An invoice is generated, sent, has its due date changed, or is flagged Overdue, Refunded, Partially refunded, Disputed, or Corrected | FEAT-09 (Invoice Generation & Sending) / FEAT-11 (Automated Payment Reminders, for the Overdue flag) | Fires on any Invoice status or due_date change reported by these features | The changed Invoice's id, status, amount, currency, due_date, and project reference |
| A payment succeeds, fails, is recorded manually, or is reversed | FEAT-10 (Invoice Payment Processing) / FEAT-25 (Refund & Cancelled Project Handling, for reversals and refund amounts) | Fires on any Payment status change or a recorded refund/reversal amount | The changed Payment's id, status, amount, paid_at, and the Invoice it belongs to |
| Nadia taps Retry on the Error state | FEAT-12.SPEC-001 (Dashboard Overview) / FEAT-12.SPEC-002 (Client/Project Financial Drill-down) | Fires when Nadia manually requests an immediate recomputation attempt after a failure | The scope (account / client / project) and currency the failed computation was for |

## Processing Logic

1. Identify the scope affected by the triggering change: the specific client and/or project the changed Invoice or Payment belongs to, plus the account-wide aggregate (which every change affects). A client/project roster or currency change (FEAT-01, FEAT-15) is not itself a trigger for this automation -- it is a Connected Entity read that FEAT-12.SPEC-003 incorporates automatically the next time this automation recomputes an affected scope (see Edge Cases).
2. Invoke FEAT-12.SPEC-003 (Financial Totals Aggregation) to recompute the Financial Totals for each affected scope and currency, reading current Invoice and Payment records at the moment of the run.
3. If the recomputation completes without error, compare the new result to the currently held Financial Totals for that scope and currency (the totals this automation last successfully computed and is holding). Replace the held totals with the new result and set its computed_at to the current time, whether or not the figures actually changed.
4. If FEAT-12.SPEC-001 or FEAT-12.SPEC-002 is currently open and displaying an affected scope, refresh its displayed totals with the newly held result, silently, without disrupting the screen's active filter selection or scroll position.
5. If the recomputation fails, leave the currently held Financial Totals for that scope and currency unchanged, and schedule an automatic retry.
6. Retry automatically up to platform parameter: `dashboard-aggregation-retry-count` times, spaced platform parameter: `dashboard-aggregation-retry-interval` apart. If any retry succeeds, proceed as in Step 3-4.
7. If all automatic retries are exhausted without success, leave the currently held Financial Totals unchanged and, if a screen is open on the affected scope, surface the Error state (last successfully computed totals plus a manual Retry control, per FEAT-12.SPEC-001 and FEAT-12.SPEC-002's States sections). If no screen is open, the failure is recorded silently and the Error state appears the next time a screen opens on that scope.
8. When Nadia manually taps Retry, attempt one immediate recomputation for that scope and currency regardless of where the automatic retry schedule currently stands; on success, proceed as in Step 3-4; on failure, the Error state remains and the automatic schedule (if still running) continues unaffected.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Recompute succeeded, totals changed | Recomputation completes and the new result differs from the currently held one | Held totals replaced with the new result; computed_at updated | Any open, affected screen refreshes silently to the new figures | FEAT-12.SPEC-001, FEAT-12.SPEC-002 |
| Recompute succeeded, no change | Recomputation completes and the new result equals the currently held one | Held totals' computed_at updated; figures unchanged | No visible change on any open screen | FEAT-12.SPEC-001, FEAT-12.SPEC-002 |
| Automatic retry in progress | Recomputation failed and an automatic retry is scheduled within the retry count/interval | Held totals unchanged | Any open, affected screen continues showing its last known totals with no interruption; no retry indicator is shown for an automatic (non-manual) retry, since it is invisible by design | FEAT-12.SPEC-001, FEAT-12.SPEC-002 |
| Automatic retries exhausted | All automatic retries fail | Held totals unchanged | Any open, affected screen shows the Error state: last successfully computed totals plus a manual Retry control | FEAT-12.SPEC-001, FEAT-12.SPEC-002 |
| Manual retry succeeded | Nadia taps Retry and the immediate recomputation succeeds | Held totals replaced with the new result; computed_at updated | The Error state clears and the refreshed totals render | FEAT-12.SPEC-001, FEAT-12.SPEC-002 |
| Manual retry failed | Nadia taps Retry and the immediate recomputation fails | Held totals unchanged | The Error state banner updates to "Still unable to refresh. Showing your last known totals." and the Retry control remains available | FEAT-12.SPEC-001, FEAT-12.SPEC-002 |
| No screen open when failure occurs | All automatic retries fail while neither FEAT-12.SPEC-001 nor FEAT-12.SPEC-002 is open | Held totals unchanged | Nothing shown at the time of failure; the Error state appears the next time Nadia opens either screen on that scope | FEAT-12.SPEC-001, FEAT-12.SPEC-002 |

## Data Model

**Reads:** Invoice (status, amount, currency, due_date, project) and Payment (status, amount, paid_at, invoice) for the affected scope, per the Feature Dependency Map. Client and Project (currency, archive status) as part of FEAT-12.SPEC-003's aggregation, so a roster or currency change already on record is reflected in whatever scope this automation next recomputes. Consumes FEAT-12.SPEC-003's derivation for the actual computation.
**Creates:** None -- this automation persists no new business record; it only maintains the currently held Financial Totals result FEAT-12.SPEC-003 defines.
**Updates:** The currently held Financial Totals (earned_total, outstanding_total, overdue_total, per-currency, per scope) and its computed_at timestamp.
**Deletes:** None.

## Business Rules

- XBR-22: this automation's triggers must cover every event type Financial Dashboard totals are derived from -- invoice generation/sending/due-date changes, payment success/failure/manual recording, and refunds/reversals/cancellations -- so no underlying change is ever missed.
- The last successfully computed totals are never discarded on failure; a failed recompute leaves the prior held result and its computed_at in place, so a viewing screen never shows a blank or misleading number (feature's Error state).
- This automation never re-derives the earned/outstanding/overdue formula itself -- every computation is delegated to FEAT-12.SPEC-003, so the two specs cannot drift out of sync.
- XBR-18: a recompute for one currency never merges with another currency's held result.
- A recompute triggered by a change to one client's data only recomputes that client's scoped totals and the account-wide aggregate; other clients' unaffected held totals are left untouched, since they did not change.
- A manual Retry always attempts immediately, independent of where the automatic retry schedule stands, so Nadia is never forced to wait out the automatic interval.

## Edge Cases

- **Concurrent trigger firing (invoices for two different clients change at effectively the same time)** -- Each client's recompute runs independently and scoped to its own client; the account-wide aggregate recomputes incorporating whichever changes have landed by the time it runs. Since each run is a full recompute from current source records rather than an incremental delta, no change is lost even if the two client-scoped runs and the account-wide run complete in a different order than they were triggered.
- **A trigger fires while a previous run for the same scope is still in flight** -- Each run reads current source data at the moment it executes and tags its result with that read time. Only a result whose read time is later than the currently held result's computed_at replaces it; an in-flight run that finishes after a newer trigger's run has already produced a fresher result is discarded on arrival rather than clobbering the more current data.
- **Automatic retries are exhausted while neither dashboard screen is open** -- The failure is recorded silently; since this feature sends no notifications of its own (Communications: N/A), Nadia only learns of it if she opens a dashboard screen on the affected scope, at which point the Error state and Retry control appear.
- **The client underlying a triggering change is archived (FEAT-01) mid-recompute** -- The recompute proceeds normally; archiving never erases records, so the archived client's totals remain computable and are still included in the account-wide aggregate and its own historical detail.
- **Nadia taps manual Retry while an automatic retry is already scheduled** -- The manual attempt runs immediately; it is not queued behind the automatic schedule. If the manual attempt itself fails, the automatic schedule continues unaffected from where it left off.
- **A client, project, or currency change (FEAT-01, FEAT-15) introduces a scope that has never been computed before** -- This automation is never triggered directly by the roster or currency change itself (that inbound relationship belongs to FEAT-12.SPEC-003, which reads Client and Project as part of every aggregation); the new scope gets its first held result the first time an invoice or payment event for it fires this automation, or the first time a screen loads it directly. That first result is a first-time computation rather than a recompute of an existing one; there is no prior "last known" total to fall back on, so a failure on this first attempt is treated identically to any other failure (automatic retry, then the Error state if exhausted) rather than as a special case.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-001 (Dashboard Overview) | Triggered by (inbound) | The manual Retry action fires this automation |
| FEAT-12.SPEC-001 (Dashboard Overview) | Affects (outbound) | Silently refreshes displayed totals on success, or surfaces the Error state on exhausted failure |
| FEAT-12.SPEC-002 (Client/Project Financial Drill-down) | Triggered by (inbound) | Same manual Retry relationship |
| FEAT-12.SPEC-002 (Client/Project Financial Drill-down) | Affects (outbound) | Same refresh/Error-state relationship |
| FEAT-12.SPEC-003 (Financial Totals Aggregation) | References (outbound) | Invokes this rule for every recomputation; never re-implements the formula |
| FEAT-09 (Invoice Generation & Sending) | Triggered by (inbound) | Invoice lifecycle events (generated, sent, due-date change) |
| FEAT-11 (Automated Payment Reminders) | Triggered by (inbound) | The Overdue status flag being set |
| FEAT-10 (Invoice Payment Processing) | Triggered by (inbound) | Payment success, failure, and manual recording |
| FEAT-25 (Refund & Cancelled Project Handling) | Triggered by (inbound) | Refunds, reversals, and cancellations |

## Analytics and Success Signals

- **dashboard_totals_recomputed** (trigger_type: invoice_change / payment_change / manual_retry; outcome: changed / no_change / failure; affected_scope: account / client / project) -- supports success-metrics.md: "Dashboard Comprehension" (recompute reliability and freshness are what let Nadia state accurate totals whenever she opens the dashboard)
- **dashboard_aggregation_retry_exhausted** (affected_scope: account / client / project, consecutive_failure_count) -- N/A -- no metric in success-metrics.md specifically targets aggregation-failure recovery; this event supports operational reliability monitoring rather than a Stage 2 product metric
- **dashboard_manual_retry_used** (outcome: success / failure) -- N/A -- no metric in success-metrics.md targets manual-retry usage specifically; this event supports monitoring how often the automatic path alone was insufficient

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (invoice change, payment change, manual retry) | 3 |
| Outcome Paths | 7 | 7 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
