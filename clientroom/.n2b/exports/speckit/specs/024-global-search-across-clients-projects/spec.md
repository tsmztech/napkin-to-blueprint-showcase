# Feature Specification: Global Search Across Clients & Projects

**Blueprint feature:** FEAT-28
**Priority tier:** Nice-to-Have
**Build order:** 024 of 33
**Depends on:** FEAT-01, FEAT-02, FEAT-06, FEAT-09
**Blueprint source:** `docs/blueprint/specifications/FEAT-28-global-search-across-clients-projects/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Global Search (Priority: P3)

Nadia (or Dana, inside a scoped support session) types a query into one search box and sees a single ranked list of matching clients, projects, proposals, deliverables, and invoices, then selects a result to jump straight to it.

**Acceptance Scenarios:**

**FEAT-28.SPEC-001-AC-01:** Given Nadia is on the Global Search screen with an empty search box, when she types "acme" (4 characters), then the automatic search fires and the screen shows the In Progress indicator.

**FEAT-28.SPEC-001-AC-02:** Given Nadia is on the Global Search screen, when she types a single character "a", then no search fires and no results panel or indicator appears.

**FEAT-28.SPEC-001-AC-03:** Given Nadia has typed a qualifying query, when FEAT-28.SPEC-002 returns three matches across two entity types, then the results panel shows three rows in the ranked order returned, each following the shared result-row pattern.

**FEAT-28.SPEC-001-AC-04:** Given Nadia has typed a query that matches nothing, when FEAT-28.SPEC-002 returns zero matches, then the panel shows "No matches for '{query}'." instead of a blank area.

**FEAT-28.SPEC-001-AC-05:** Given Nadia is viewing search results, when she taps a Project result, then she is navigated directly to that project's Project Detail screen (FEAT-01.SPEC-005).

**FEAT-28.SPEC-001-AC-06:** Given Nadia is viewing search results, when she taps an Invoice result showing an amount in EUR while another visible result is an invoice in USD, then each amount displays only in its own record's currency and no combined or converted total is shown anywhere on the screen.

**FEAT-28.SPEC-001-AC-07:** Given Nadia's search automation fails once, when the automatic retry inside FEAT-28.SPEC-002 also fails, then the screen shows the error banner "Search couldn't complete. Try again." with a Retry button.

**FEAT-28.SPEC-001-AC-08:** Given Nadia sees the search error banner, when she taps Retry, then the screen re-enters In Progress and re-triggers FEAT-28.SPEC-002 with the same query.

**FEAT-28.SPEC-001-AC-09:** Given Nadia loses connectivity while a query is entered, when the device is confirmed offline, then the screen shows "You're offline -- showing your most recently loaded results, which may be out of date." above her last cached results, and no new query executes until connectivity returns.

**FEAT-28.SPEC-001-AC-10:** Given Nadia taps the clear control, when the search box empties, then the results panel collapses and the screen returns to the Empty state.

**FEAT-28.SPEC-001-AC-11:** Given Dana has an open support session scoped to one freelancer account, when she opens Global Search and types a qualifying query, then results are limited to that one account only, presented with the same shared result-row pattern Nadia sees.

**FEAT-28.SPEC-001-AC-12:** Given Dana has no open support session, when she attempts to reach the search entry point directly, then she sees "Global search requires an active support session for one account." and no search box is offered.

**FEAT-28.SPEC-001-AC-13:** Given Owen (Client Primary Contact) is signed in to his client portal, when he looks for a way to search across records, then no search entry point exists anywhere in his portal navigation.

**FEAT-28.SPEC-001-AC-14:** Given Dana's support session closes while a query she entered is still in flight, when the session ends, then the in-flight query is aborted and she sees the unauthorized experience rather than stale results from the ended session.

**FEAT-28.SPEC-001-AC-15:** Given Nadia's session expires while she has partially typed a query, when she is prompted to sign back in, then the dialog reads "Your session has expired. Sign in to continue." and the partially typed query is discarded rather than restored.

### User Story 2 - Cross-Entity Search Execution (Priority: P3)

Runs a qualifying search query across Client, Project, Proposal, Deliverable, and Invoice records, enforcing scope rules and applying relevance ranking, with automatic retry and an offline/degraded fallback signal.

**Acceptance Scenarios:**

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

### User Story 3 - Search Scope & Access Rules (Priority: P3)

Governs who may perform a global search, which single account any search is ever allowed to touch, and the minimum query length required before a search executes.

**Acceptance Scenarios:**

**FEAT-28.SPEC-003-AC-01:** Given Nadia types a 2-character query, when the length check runs, then the search is permitted to execute.

**FEAT-28.SPEC-003-AC-02:** Given Nadia types a 1-character query, when the length check runs, then the search is not permitted to execute and no error message is shown.

**FEAT-28.SPEC-003-AC-03:** Given Nadia types a query consisting only of spaces, when the length check counts non-whitespace characters, then the search is not permitted to execute regardless of how many spaces were typed.

**FEAT-28.SPEC-003-AC-04:** Given Nadia (requesting_role: Nadia) submits a qualifying query, when target_account is derived, then it resolves to Nadia's own freelancer account.

**FEAT-28.SPEC-003-AC-05:** Given Dana has an open Support Access Session naming one freelancer account, when she submits a qualifying query, then target_account resolves to that one account and the search is permitted to execute.

**FEAT-28.SPEC-003-AC-06:** Given Dana has no open Support Access Session, when she attempts to submit a query, then the search is denied with "Global search requires an active support session for one account."

**FEAT-28.SPEC-003-AC-07:** Given Dana's Support Access Session closes while a query is in flight, when the execution-level check re-evaluates the session, then the search is denied even though the screen-level check passed earlier.

**FEAT-28.SPEC-003-AC-08:** Given Owen (Client Primary Contact) is signed in, when he looks for a way to execute a search, then no search entry point is available to him anywhere.

**FEAT-28.SPEC-003-AC-09:** Given Priya (Client Reviewer Contact) is signed in, when she looks for a way to execute a search, then no search entry point is available to her anywhere.

**FEAT-28.SPEC-003-AC-10:** Given Nadia submits a qualifying query, when results are matched, then every result belongs to Nadia's own account and none belongs to any other freelancer's account.

**FEAT-28.SPEC-003-AC-11:** Given Dana's Support Access Session names Account X, when she searches, then results are limited to Account X and never include any data from any other account, including her own (she has none).

**FEAT-28.SPEC-003-AC-12:** Given Nadia has two browser sessions open and searches in each simultaneously, when both queries execute, then each independently returns results scoped to her own account with no conflict or blocking between them.

**FEAT-28.SPEC-003-AC-13:** Given Dana closes one support session and immediately opens a new one for a different account, when she searches after the new session opens, then target_account resolves to the newly opened session's account, never the just-closed one.

### User Story 4 - Search Result Relevance Ranking (Priority: P3)

Defines how candidate matches across Client, Project, Proposal, Deliverable, and Invoice records are scored, ordered, and tie-broken into the single ranked list the freelancer (or Dana) sees.

**Acceptance Scenarios:**

**FEAT-28.SPEC-004-AC-01:** Given a candidate set with one exact match on a Client's client_name and one prefix match on a Project's project_name, when ranking runs, then the exact Client match is ordered first regardless of either record's update recency.

**FEAT-28.SPEC-004-AC-02:** Given a candidate set with two exact matches on client_name -- one updated an hour ago and one updated a year ago -- when ranking runs, then the more recently updated Client is ordered first.

**FEAT-28.SPEC-004-AC-03:** Given a candidate that matches exactly on a Proposal's owning Project's project_name (a secondary field) and another candidate that matches with a partial match on an Invoice's invoice_number (a primary field), when ranking runs, then the exact secondary-field match is ordered first, because match_type outranks matched_field priority.

**FEAT-28.SPEC-004-AC-04:** Given two candidates identical in match_type, matched_field, and source_updated_at, one a Project and one an Invoice, when ranking runs, then the Project is ordered before the Invoice, per the fixed entity-type tie-break sequence.

**FEAT-28.SPEC-004-AC-05:** Given two Client candidates identical in match_type, matched_field, and source_updated_at, when ranking runs, then they are ordered by source_reference in a stable order that produces the same result on a repeated, identical search.

**FEAT-28.SPEC-004-AC-06:** Given a query that matches a Project's project_name exactly and also matches that same Project's owning Client's client_name exactly, when ranking runs, then two separate candidate matches are scored -- one on the Project's primary field and one on the Client's primary field -- each ranked on its own merits.

**FEAT-28.SPEC-004-AC-07:** Given a candidate set drawn entirely from Deliverable records, when ranking runs, then the fixed entity-type tie-break is never invoked, since no cross-type comparison occurs.

**FEAT-28.SPEC-004-AC-08:** Given an empty candidate set is passed to ranking, when ranking runs, then it returns an empty ranked list, which FEAT-28.SPEC-002 treats as the No Results outcome.

**FEAT-28.SPEC-004-AC-09:** Given Nadia's search produces candidates across four different entity types, when the ranked list is returned, then it is one interleaved list ordered purely by score -- never five separate lists grouped by entity type.

**FEAT-28.SPEC-004-AC-10:** Given Dana's support session scopes her search to one account, when ranking runs over the candidates FEAT-28.SPEC-002 already limited to that account, then the ranking logic itself applies identically to hers as to Nadia's, since ranking makes no role-based distinction.

**FEAT-28.SPEC-004-AC-11:** Given a candidate matches with match_type "prefix" on a primary field and another candidate matches with match_type "partial" on a primary field, when ranking runs, then the prefix match is ordered before the partial match, both being on primary fields.

**FEAT-28.SPEC-004-AC-12:** Given the same query is run twice in immediate succession against an unchanged data set, when ranking runs both times, then the resulting order is identical both times, including the resolution of any exact ties.

### Edge Cases

- **FEAT-28.SPEC-001 (Global Search):** The screen holds no state that can go stale and writes to no shared entity, so a record changed or removed between results and selection is handled at the destination. Deleting characters back below the minimum length clears results immediately, and a whitespace-only or single-character query is treated as below minimum with no error. Source: `docs/blueprint/specifications/FEAT-28-global-search-across-clients-projects/FEAT-28.SPEC-001-global-search.md` (section: Edge Cases)
- **FEAT-28.SPEC-002 (Cross-Entity Search Execution):** Only the response to the most recently fired query is ever shown, so a stale in-flight run is discarded, and concurrent qualifying queries run independently and read-only. An identical unchanged query already in flight starts no new run. Source: `docs/blueprint/specifications/FEAT-28-global-search-across-clients-projects/FEAT-28.SPEC-002-cross-entity-search-execution.md` (section: Edge Cases)
- **FEAT-28.SPEC-003 (Search Scope & Access Rules):** A query of exactly 2 non-whitespace characters passes and 1 does not, and whitespace-only input of any length is below minimum. The support-session requirement is re-evaluated at execution time, so a session closing at submit or two successive sessions for different accounts use the session active at that moment. Source: `docs/blueprint/specifications/FEAT-28-global-search-across-clients-projects/FEAT-28.SPEC-003-search-scope-access-rules.md` (section: Edge Cases)
- **FEAT-28.SPEC-004 (Search Result Relevance Ranking):** Ranking is deterministic: ties on match type and field break by most recent update, then fixed entity-type sequence, then source reference. A record matching on both primary and secondary fields yields a candidate per field, and match type is evaluated before matched field. Source: `docs/blueprint/specifications/FEAT-28-global-search-across-clients-projects/FEAT-28.SPEC-004-search-result-relevance-ranking.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-28.SPEC-001** (Global Search) as specified: Nadia (or Dana, inside a scoped support session) types a query into one search box and sees a single ranked list of matching clients, projects, proposals, deliverables, and invoices, then selects a result to jump straight to it. Full spec: `docs/blueprint/specifications/FEAT-28-global-search-across-clients-projects/FEAT-28.SPEC-001-global-search.md`
- **FR-002**: The system MUST implement **FEAT-28.SPEC-002** (Cross-Entity Search Execution) as specified: Runs a qualifying search query across Client, Project, Proposal, Deliverable, and Invoice records, enforcing scope rules and applying relevance ranking, with automatic retry and an offline/degraded fallback signal. Full spec: `docs/blueprint/specifications/FEAT-28-global-search-across-clients-projects/FEAT-28.SPEC-002-cross-entity-search-execution.md`
- **FR-003**: The system MUST implement **FEAT-28.SPEC-003** (Search Scope & Access Rules) as specified: Governs who may perform a global search, which single account any search is ever allowed to touch, and the minimum query length required before a search executes. Full spec: `docs/blueprint/specifications/FEAT-28-global-search-across-clients-projects/FEAT-28.SPEC-003-search-scope-access-rules.md`
- **FR-004**: The system MUST implement **FEAT-28.SPEC-004** (Search Result Relevance Ranking) as specified: Defines how candidate matches across Client, Project, Proposal, Deliverable, and Invoice records are scored, ordered, and tie-broken into the single ranked list the freelancer (or Dana) sees. Full spec: `docs/blueprint/specifications/FEAT-28-global-search-across-clients-projects/FEAT-28.SPEC-004-search-result-relevance-ranking.md`

### Key Entities

- N/A — search reads across existing entities rather than owning one.

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Searches, result selections and no-results displays are each observable as distinct signals (search_performed, search_result_selected, search_no_results_shown); no metric in the success-metrics register connects to this feature, so the outcome is grounded in its Signals alone. Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-23**: Strict data isolation, so search is scoped to the freelancer's own account (or the open support session). Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-22**: Each freelancer has 3-15 active clients, which bounds the searchable corpus. Full register: `docs/blueprint/features/assumptions-constraints.md`
