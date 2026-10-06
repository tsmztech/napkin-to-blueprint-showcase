---
document_type: spec
spec_type: automation
spec_id: FEAT-28.SPEC-002
spec_name: Cross-Entity Search Execution
spec_slug: cross-entity-search-execution
parent_feature: FEAT-28
parent_feature_name: Global Search Across Clients & Projects
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Cross-Entity Search Execution

## Overview

**Name:** Cross-Entity Search Execution
**ID:** FEAT-28.SPEC-002
**Type:** Automation
**Purpose:** Runs a qualifying search query across Client, Project, Proposal, Deliverable, and Invoice records, enforcing scope rules and applying relevance ranking, with automatic retry and an offline/degraded fallback signal.
**Parent Feature:** FEAT-28 -- Global Search Across Clients & Projects

## Scope and Non-Goals

**In Scope:**
- Matching a query against Client, Project, Proposal, Deliverable, and Invoice records within the caller's scoped account
- Enforcing the query-length and role/account scope rules before any record is matched
- Handing the candidate match set to relevance ranking and returning the ranked list to the triggering screen
- Retrying once automatically on failure before surfacing a manual retry option
- Signaling the triggering screen to fall back to locally cached results when offline or degraded

**Non-Goals:**
- Defining the minimum query length or who may search which account -- governed by FEAT-28.SPEC-003 (Search Scope & Access Rules), which this automation enforces rather than restates
- Deciding how matches are scored, ordered, and tie-broken -- governed by FEAT-28.SPEC-004 (Search Result Relevance Ranking); this automation only invokes that logic and passes its result through
- Writing an Activity Log Entry for a search -- excluded per feature-overview.md's Data Notes ("Captured: none") and Compliance flags ("search introduces no compliance surface of its own"); search is deliberately not one of the record-worthy events XBR-05 lists
- Modifying any Client, Project, Proposal, Deliverable, or Invoice record -- excluded per feature-overview.md's Entity-Lifecycle Coverage Matrix: this feature "creates, updates, deletes, and archives nothing," reading only what other features own

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Query entered (2+ characters) | FEAT-28.SPEC-001 (Global Search) | Fires once the query reaches or exceeds the minimum length defined by FEAT-28.SPEC-003, after the freelancer or Dana pauses typing | Query text, requesting role (Nadia or Dana), and account scope (Nadia's own account, or the one account named by Dana's open support session) |
| Manual retry after failure | FEAT-28.SPEC-001 (Global Search) | Fires when the freelancer or Dana taps Retry after this automation's own automatic retry has also failed | The same query text, requesting role, and account scope as the failed attempt |

## Processing Logic

1. Receive the query text, requesting role, and account scope from the triggering screen (FEAT-28.SPEC-001).
2. Confirm scope eligibility per FEAT-28.SPEC-003 -- the query meets the minimum length, the requesting role is permitted to search, and (for Dana) an open, account-scoped support session names exactly the account being searched. If any check fails, do not proceed to matching (FEAT-28.SPEC-003 owns the resulting denied experience; this automation is never invoked with an ineligible query in the normal flow).
3. Match the query text against records belonging exclusively to the scoped account:
   - Client: client_name
   - Project: project_name, plus its owning Client's client_name as secondary context
   - Proposal: scope_description, plus its owning Project's project_name and Client's client_name as secondary context
   - Deliverable: the deliverable's "file or link" (the uploaded file's name, or the linked asset's title or address), plus its owning Project's project_name and Client's client_name as secondary context (the Project is located through the Deliverable's required milestone relationship; no Milestone field is read, matched, or displayed)
   - Invoice: invoice_number, plus its owning Project's project_name and Client's client_name as secondary context
4. For each matching record, capture the entity type, the matched field, the match type (exact, prefix, or partial), and the record's most recent relevant update timestamp.
5. Pass the full candidate match set to Search Result Relevance Ranking (FEAT-28.SPEC-004) to score, order, and tie-break the matches into one ranked list.
6. Return the ranked list to the triggering screen (FEAT-28.SPEC-001).
7. If the data read itself fails or times out, retry the same query once automatically before surfacing a manual retry option to the triggering screen.
8. If the device is offline or the search capability is degraded, signal the triggering screen to fall back to whichever results it most recently loaded locally, rather than attempting a query that cannot complete.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Results found | One or more matches after ranking | None -- read-only | Results panel fills with the ranked rows | FEAT-28.SPEC-001 |
| No results | Zero matches across all five entity types | None | "No matches for '{query}'." message shown | FEAT-28.SPEC-001 |
| Automatic retry succeeds | The first attempt fails, the automatic retry succeeds | None | Results found or No results, shown transparently -- the freelancer or Dana never sees the failed first attempt | FEAT-28.SPEC-001 |
| Failure after retry | Both the first attempt and the automatic retry fail | None | Error banner "Search couldn't complete. Try again." with a manual Retry button | FEAT-28.SPEC-001 |
| Offline/degraded fallback | Device offline, or the search capability is degraded, at trigger time | None -- no query is attempted against live data | Offline/degraded banner over the most recently cached ranked results | FEAT-28.SPEC-001 |

## Data Model

**Reads:** Client (client_name), Project (project_name), Proposal (scope_description), Deliverable (file or link: the uploaded file's name, or the linked asset's title or address), Invoice (invoice_number) -- each read-only, scoped exclusively to the account FEAT-28.SPEC-003 derives for the requesting role, per the Feature Dependency Map's entity definitions.
**Creates:** None.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Scope enforcement is entirely owned by FEAT-28.SPEC-003 -- this automation never queries outside the account that spec derives for the requesting role, and never restates the minimum-length or role/account rules independently.
- Match scoring, ordering, and tie-breaking are entirely owned by FEAT-28.SPEC-004 -- this automation passes the full unranked candidate set through unmodified and returns exactly the ranked list that comes back.
- ASMP-23 / XBR-09 (isolation, applied to accounts rather than client companies): a search never returns a record belonging to any account other than the one derived for the requesting role, including for Dana, whose results are further limited to the single account named by her open support session.
- This automation runs read-only and produces no Activity Log Entry -- consistent with feature-overview.md's Data Notes ("Captured: none"), search is not one of the record-worthy events XBR-05 enumerates.
- The automatic retry is attempted exactly once per failed query; a second failure always surfaces the manual Retry option rather than retrying silently again.

## Edge Cases

- **Freelancer types faster than results return (multiple queries in quick succession)** -- Each keystroke that reaches the minimum length can start a new run, but only the response to the most recently fired query is ever shown; an in-flight response for an earlier, now-superseded query is discarded on arrival rather than overwriting newer results.
- **Query changes while a run is in flight** -- The in-flight run for the stale query is not shown even if it completes after the newer query's run starts; the triggering screen's In Progress indicator reflects only the latest query.
- **Concurrent trigger firing (two qualifying queries fire at effectively the same time, e.g., typed then immediately cleared and retyped)** -- Each run executes independently and read-only against the same account; because search performs no writes, there is no data race to resolve -- only the most recent query's result is surfaced to the screen, per the rule above.
- **Trigger fires while a previous run for the same query is still in flight** -- No new run starts for an identical, unchanged query already in flight; the existing run's eventual result satisfies both trigger attempts.
- **Dana's support session closes while a run is in flight** -- The in-flight run is aborted rather than returning results computed against a scope that just ended; the triggering screen shows the unauthorized experience defined in FEAT-28.SPEC-001's Access and Visibility table.
- **Freelancer has zero active clients (brand-new account)** -- Matching proceeds normally and returns zero matches; this is the No Results outcome, not a failure.
- **Query matches a very large number of records at the product's stated scale (3-15 active clients and their full history, per ASMP-22)** -- All qualifying matches are ranked and returned; the Non-Functional Notes commit to staying responsive at this scale from MVP onward rather than capping or paginating the result set.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-28.SPEC-001 (Global Search) | Triggered by (inbound) | Every qualifying query and manual retry fires this automation |
| FEAT-28.SPEC-001 (Global Search) | Affects (outbound) | Returns the ranked results, the no-results signal, or the failure/offline fallback signal |
| FEAT-28.SPEC-003 (Search Scope & Access Rules) | References (inbound) | Owns the minimum-length and role/account scope rules this automation enforces before matching |
| FEAT-28.SPEC-004 (Search Result Relevance Ranking) | Triggers (outbound) | Every candidate match set is scored, ordered, and tie-broken there before being returned |
| FEAT-01 (Client & Project Management) | References (inbound) | Reads Client and Project records to match against the query |
| FEAT-02 (Proposal Creation & Sending) | References (inbound) | Reads Proposal records to match against the query |
| FEAT-06 (Deliverable Upload & Sharing) | References (inbound) | Reads Deliverable records to match against the query |
| FEAT-09 (Invoice Generation & Sending) | References (inbound) | Reads Invoice records to match against the query |

## Analytics and Success Signals

N/A -- success-metrics.md's twenty Connected Feature entries do not name Global Search Across Clients & Projects (FEAT-28); no Stage 2 success metric is connected to this feature, so no outcome path here has a metric to cite. This gap is recorded once, in the feature's screen spec (FEAT-28.SPEC-001), per the Category 8 self-review requirement, rather than repeated verbatim in every spec.

## Acceptance Criteria

**FEAT-28.SPEC-002-AC-01:** Given Nadia has typed "acme corp" (a qualifying query) on FEAT-28.SPEC-001, when this automation matches it against her account's records, then it returns a ranked list including a Client match on client_name "Acme Corp".

**FEAT-28.SPEC-002-AC-02:** Given Nadia has typed a query that matches nothing across all five entity types in her account, when the match step completes, then this automation returns an empty ranked list and the screen shows No Results.

**FEAT-28.SPEC-002-AC-03:** Given the data read for Nadia's query fails once, when this automation retries automatically, then the retry succeeds and the ranked results are shown without Nadia ever seeing the first failure.

**FEAT-28.SPEC-002-AC-04:** Given the data read for Nadia's query fails and the automatic retry also fails, when both attempts are exhausted, then the triggering screen shows the error banner and a manual Retry option, and no third automatic attempt occurs.

**FEAT-28.SPEC-002-AC-05:** Given Nadia taps the manual Retry option after a failure, when this automation runs again with the same query, then it executes exactly as a fresh trigger and returns Results Found, No Results, or Failure accordingly.

**FEAT-28.SPEC-002-AC-06:** Given Nadia's device goes offline while she has a query entered, when this automation would otherwise fire, then it signals the screen to show its most recently cached results instead of attempting a query.

**FEAT-28.SPEC-002-AC-07:** Given Dana has an open support session scoped to one freelancer account, when she types a qualifying query, then this automation matches only against that one account's Client, Project, Proposal, Deliverable, and Invoice records, and never against Dana's own data (she has no freelancer account) or any other freelancer's.

**FEAT-28.SPEC-002-AC-08:** Given Nadia types a query and then immediately types more characters before the first run returns, when both runs execute, then only the response matching her final, current query text is ever shown on the screen.

**FEAT-28.SPEC-002-AC-09:** Given a run for Nadia's current query is already in flight, when the exact same query fires again without any change, then no duplicate run starts and the existing run's result satisfies the screen.

**FEAT-28.SPEC-002-AC-10:** Given Dana's support session closes while a query she entered is still being matched, when the session ends, then this automation aborts the in-flight run rather than returning results computed against the now-closed scope.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (query entered, manual retry) | 2 |
| Outcome Paths | 5 (results found, no results, retry succeeds, failure after retry, offline/degraded) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |
