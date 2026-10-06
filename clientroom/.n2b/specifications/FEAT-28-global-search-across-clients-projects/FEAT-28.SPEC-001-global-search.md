---
document_type: spec
spec_type: screen
spec_id: FEAT-28.SPEC-001
spec_name: Global Search
spec_slug: global-search
parent_feature: FEAT-28
parent_feature_name: Global Search Across Clients & Projects
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Global Search

## Overview

**Name:** Global Search
**ID:** FEAT-28.SPEC-001
**Type:** Screen
**Purpose:** Nadia (or Dana, inside a scoped support session) types a query into one search box and sees a single ranked list of matching clients, projects, proposals, deliverables, and invoices, then selects a result to jump straight to it.
**Parent Feature:** FEAT-28 -- Global Search Across Clients & Projects

## Scope and Non-Goals

**In Scope:**
- The persistent search box and its in-progress, empty, no-results, error, and offline/degraded states
- The ranked results panel, presented as one consistent result-row pattern across all five entity types
- Selecting a result to navigate directly to the underlying record
- Displaying invoice and proposal amounts safely (own currency only, never converted or summed) per XBR-18

**Non-Goals:**
- Executing the cross-entity query itself (reading and matching Client, Project, Proposal, Deliverable, and Invoice records) -- handled by FEAT-28.SPEC-002 (Cross-Entity Search Execution); this screen's Analyst-Discovered rationale (feature-overview.md, Phase 4) is that a non-trivial cross-entity response crosses the inline threshold for a screen's own interactions
- Deciding how matches are scored, ordered, and tie-broken -- handled by FEAT-28.SPEC-004 (Search Result Relevance Ranking); this screen only presents the ranked list it receives
- Cross-account or multi-freelancer search -- excluded per BRIEF.md's Privacy constraint and feature-overview.md's Non-Goals: search scope is limited strictly to the freelancer's own data, with Dana's exception being a single logged, account-scoped support session (FEAT-31)
- A native mobile search experience -- excluded per scope-boundaries.md (SC-06): the product ships no native apps; this screen is delivered as part of the same web app as every other freelancer-side screen
- Editing, creating, or acting on any record from within the results panel -- selecting a result only navigates to that record's own screen (FEAT-01, FEAT-02, FEAT-06, or FEAT-09), which owns every edit and action capability

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Global navigation (persistent, freelancer-side) | Nadia opens search from anywhere in the freelancer-side app | None -- search box starts empty |
| FEAT-31 (Operator Support Access) support console | Dana opens search while an account support session is open | The one freelancer account named by her active Support Access Session |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Types a query and selects any result across her own Client, Project, Proposal, Deliverable, and Invoice records | -- |
| Owen (Client Primary Contact) | No | No | The client portal's navigation carries no search entry point -- excluded per this Brief's Non-Goals ("Client-contact-facing search"); Owen's portal offers only his own company's client-facing screens |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- the client portal's navigation carries no search entry point |
| Dana (Support Operator) | Full screen, but only while a support session is open | Types a query and selects results, scoped to the one account named by her open Support Access Session (FEAT-31); never her own account (she has none) and never any other freelancer's | Outside an open support session, the search entry point is not shown in her console; a direct attempt to reach it shows "Global search requires an active support session for one account." |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the person lands on their normal entry point (the Freelancer Financial Dashboard for Nadia, or the operator's support console for Dana) rather than on search directly |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Any in-progress query text is discarded -- search holds no draft worth preserving, since results reflect live data rather than authored content |

## Layout and Content

**Header:** A persistent search box (single-line text input) with placeholder text "Search clients, projects, proposals, deliverables, invoices" and a clear control (visible once text is entered) that empties the box and collapses the results panel.

**Body:** Below the search box, a results panel appears once the query reaches the minimum length. Each result row follows the Brief's shared UI pattern, applied consistently across all five entity types:
- A type indicator (client / project / proposal / deliverable / invoice)
- A primary line -- the record's identifying reference (client name, project name, the proposal's scope description, the deliverable's "file or link" -- the uploaded file's name, or the linked asset's title or address -- or the invoice number)
- A secondary line -- parent context (e.g., a project's owning client; a proposal's, deliverable's, or invoice's owning project and client). Milestone is never shown, matched, or read by this feature
- For Invoice results only, an amount shown in that invoice's own currency, positioned at the end of the row

Rows are ordered top-to-bottom by the rank FEAT-28.SPEC-002 returns (as scored by FEAT-28.SPEC-004) -- this screen applies no ordering of its own.

**Footer:** None -- the results panel scrolls independently within the body area; no separate footer controls exist.

### Responsive Behavior

- **Compact breakpoint:** Search box spans the full width of its container; the results panel appears directly below it as a single column, one row per line, matching the persona's primary device (laptop/desktop, per user-persona.md's Behavioral Context) while remaining usable on a mobile browser.
- **Medium size class and above:** Same single-column results panel, capped at a consistent platform-wide content width (exact value is the design layer's decision) and horizontally centered beneath the search box -- no structural change beyond width capping.
- **Result row:** Uniform scaling, no structural change -- the same four elements (type indicator, primary line, secondary line, amount when present) appear at every breakpoint.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Search box | Type | Once the query reaches 2+ characters and the freelancer or Dana pauses typing, triggers FEAT-28.SPEC-002 (Cross-Entity Search Execution) | Screen enters the In Progress state | Lightweight in-progress indicator appears in place of any previous results |
| Search box | Type (fewer than 2 characters) | No automation triggered (governed by FEAT-28.SPEC-003's minimum-length rule) | Screen stays in or returns to Below Minimum Length | No results panel, no error, no indicator |
| Clear control | Tap | Empties the search box and cancels any in-flight query | Results panel collapses; screen returns to Empty | Search box returns to placeholder text |
| Result row | Tap/select | Navigates directly to the underlying record's own screen: Client or Project to FEAT-01 (Client & Project Management), Proposal to FEAT-02 (Proposal Creation & Sending), Deliverable to FEAT-06 (Deliverable Upload & Sharing), Invoice to FEAT-09 (Invoice Generation & Sending) | This screen closes | Animated transition to the destination screen |
| Result row (while previous navigation is in flight) | Tap/select (second time, rapidly) | No action -- ignored while the first navigation is completing | None | Destination screen's own loading feedback continues; no duplicate navigation occurs |
| Retry button (shown in the Error state) | Tap | Re-triggers FEAT-28.SPEC-002 with the same query and scope | Screen re-enters In Progress | In-progress indicator resumes |
| Type indicator (on each result row) | -- (display-only) | None | None | Communicates the row's entity type visually; not interactive |

### Accessibility Notes

- **Focus order:** Search box -> clear control (when present) -> first result row -> each subsequent result row in ranked order -> Retry button (when the Error state is shown).
- **Dynamic announcements:** The in-progress indicator, the "no results" message, the error banner, and the offline/degraded banner are each announced to assistive technology as they appear.
- **Keyboard alternatives:** Every result row and control on this screen is reachable and selectable by keyboard (tab and arrow navigation plus Enter/Space to select); there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Search box empty with placeholder text; no results panel shown | Screen first opens, or the clear control is used | The freelancer or Dana begins typing |
| Below Minimum Length | Search box shows the entered text; no results panel, no indicator, no error | Query is fewer than 2 characters (FEAT-28.SPEC-003) | Query reaches 2+ characters |
| In Progress | A lightweight in-progress indicator appears in the results panel area, replacing any previously shown results | Query reaches 2+ characters and the automatic search fires (FEAT-28.SPEC-002) | The automation returns results, returns no results, or fails |
| Results | Ranked result rows fill the panel, most relevant first | FEAT-28.SPEC-002 returns one or more matches | Query changes (returns to In Progress), search is cleared, or a result is selected |
| No Results | Panel shows a clear message, e.g. "No matches for '{query}'." rather than a blank area | FEAT-28.SPEC-002 returns zero matches | Query changes or search is cleared |
| Error | Error banner "Search couldn't complete. Try again." with a Retry button; any previously shown results remain visible, dimmed, beneath the banner | The automatic retry inside FEAT-28.SPEC-002 also fails | The freelancer or Dana taps Retry (returns to In Progress), or clears the search |
| Offline/Degraded | Banner "You're offline -- showing your most recently loaded results, which may be out of date." above the last cached ranked result list; no new query executes | Connectivity is lost, or the search capability is degraded, while a query is entered | Connectivity and the search capability are restored -- the current query re-executes automatically |

## Validation Rules

Validation governed by FEAT-28.SPEC-003 (Search Scope & Access Rules). See that spec for the minimum query-length rule and the role/account scoping rules. This screen checks length continuously as the freelancer or Dana types (no separate submit step exists -- search runs as text is entered).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Selecting a Client result | FEAT-01.SPEC-004 (Client Detail) | FEAT-01 (Client & Project Management) |
| Selecting a Project result | FEAT-01.SPEC-005 (Project Detail) | FEAT-01 (Client & Project Management) |
| Selecting a Proposal result | FEAT-02.SPEC-003 (Proposal Detail) | FEAT-02 (Proposal Creation & Sending) |
| Selecting a Deliverable result | FEAT-06.SPEC-002 (Deliverable List & Management), opened scrolled/focused to that deliverable | FEAT-06 (Deliverable Upload & Sharing) |
| Selecting an Invoice result | FEAT-09.SPEC-002 (Invoice Detail) | FEAT-09 (Invoice Generation & Sending) |

## Data Model

**Creates:** None -- this screen creates no data (feature-overview.md, Entity-Lifecycle Coverage Matrix: "the feature creates, updates, deletes, and archives nothing").
**Reads:** Client (client_name), Project (project_name), Proposal (scope_description), Deliverable (file or link: the uploaded file's name, or the linked asset's title or address), Invoice (invoice_number, amount, currency) -- each read-only, scoped and ranked by FEAT-28.SPEC-002 and FEAT-28.SPEC-004, and displayed via the shared result-row pattern.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Query minimum length and who may search which account are governed entirely by FEAT-28.SPEC-003 (Search Scope & Access Rules) -- this screen enforces nothing beyond deferring to it.
- Result ordering is governed entirely by FEAT-28.SPEC-004 (Search Result Relevance Ranking) -- this screen never re-sorts or re-groups the list it receives.
- XBR-18: An invoice or proposal amount is always shown in that record's own currency; amounts across different results are never converted or summed, including when multiple invoice results in different currencies appear in the same list.

## Edge Cases

- **Underlying record changes or is removed between the search returning results and the freelancer selecting one** -- No conflict occurs on this screen: it holds no state that can go stale in a way requiring resolution here. The destination screen (FEAT-01, FEAT-02, FEAT-06, or FEAT-09) loads the record fresh on open and handles a since-changed or since-removed record with its own normal not-found or refreshed behavior.
- **This screen never writes to any shared entity** -- feature-overview.md's Entity-Lifecycle Coverage Matrix states the feature "creates, updates, deletes, and archives nothing," so no concurrent-edit conflict entry applies to this screen (per the dependency map's Contention notes, which govern only writers).
- **Freelancer types, then deletes characters back below the minimum length** -- Screen returns to Below Minimum Length; any results panel or indicator is cleared immediately, not left stale.
- **Query is whitespace-only or a single character** -- Treated as below the minimum length (FEAT-28.SPEC-003); no search fires and no error is shown.
- **Freelancer navigates away and returns to this screen** -- No draft is preserved; the search box starts empty again, since results reflect live data rather than authored content worth restoring.
- **Dana's support session closes while a query is in flight** -- The in-flight query is aborted; the screen returns to the Unauthorized Experience defined in Access and Visibility rather than showing stale results from the just-ended session.
- **A freelancer with zero clients yet (brand-new account) searches** -- Returns No Results, not an error; this is a normal outcome of an empty account, not a failure.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-28.SPEC-002 (Cross-Entity Search Execution) | Triggers (outbound) | Every qualifying query triggers this automation, which returns the ranked result set or a failure/offline signal |
| FEAT-28.SPEC-003 (Search Scope & Access Rules) | References (inbound/outbound) | This screen's minimum-length and role/account scoping behavior is entirely defined there |
| FEAT-28.SPEC-004 (Search Result Relevance Ranking) | References (inbound) | The order in which this screen presents results is defined there, applied inside FEAT-28.SPEC-002's processing |
| FEAT-01.SPEC-004 (Client Detail) | Navigation (outbound) | Selecting a Client result opens this screen |
| FEAT-01.SPEC-005 (Project Detail) | Navigation (outbound) | Selecting a Project result opens this screen |
| FEAT-02.SPEC-003 (Proposal Detail) | Navigation (outbound) | Selecting a Proposal result opens this screen |
| FEAT-06.SPEC-002 (Deliverable List & Management) | Navigation (outbound) | Selecting a Deliverable result opens this screen |
| FEAT-09.SPEC-002 (Invoice Detail) | Navigation (outbound) | Selecting an Invoice result opens this screen |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana reaches this screen only from within an open, account-scoped support session |

## Analytics and Success Signals

N/A -- success-metrics.md's twenty Connected Feature entries do not name Global Search Across Clients & Projects (FEAT-28); no Stage 2 success metric is connected to this feature, so no event on this screen has a metric to cite. This gap is recorded here (the feature's screen spec) rather than silently dropped, per the Category 8 self-review requirement.

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 7 (empty, below-minimum-length, in-progress, results, no-results, error, offline/degraded) | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 7 | 7 |
