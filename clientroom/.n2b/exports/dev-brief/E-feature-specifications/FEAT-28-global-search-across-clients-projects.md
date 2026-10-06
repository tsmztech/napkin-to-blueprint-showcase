# FEAT-28 — Global Search Across Clients & Projects

This chapter covers Global Search Across Clients & Projects, a Nice-to-Have-tier feature. It contains the feature breakdown brief followed by every specification in full: 4 specifications carrying 50 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-28.SPEC-001 | Global Search | screen | 15 |
| FEAT-28.SPEC-002 | Cross-Entity Search Execution | automation | 10 |
| FEAT-28.SPEC-003 | Search Scope & Access Rules | logic-rule | 13 |
| FEAT-28.SPEC-004 | Search Result Relevance Ranking | logic-rule | 12 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Global Search Across Clients & Projects

## Summary

**Feature:** Global Search Across Clients & Projects
**ID:** FEAT-28
**Description:** The freelancer searches across all clients, projects, proposals, and invoices from one search box instead of navigating the client list manually.
**Priority:** Nice-to-Have
**Phase:** v1
**Type:** User-Facing
**Rationale:** The decomposition checklist's Cross-Cutting Concerns flags search once a product manages more than one entity type, which this product clearly does. Nice-to-Have because a freelancer with 3–15 clients (BRIEF.md, Scale) can still browse manually without it; phased v1 since it becomes genuinely useful once a freelancer has accumulated enough clients and history to need it.

**Key Capabilities:**
- Search across entity types — clients, projects, proposals, invoices in one query
- Jump to a result — selecting a result opens it directly

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-28.SPEC-001 | Global Search | Screen | Nadia, Dana | Freelancer (or a scoped support operator) types a query and sees ranked results across her clients, projects, proposals, and invoices, and jumps straight to any result |
| FEAT-28.SPEC-002 | Cross-Entity Search Execution | Automation | Nadia, Dana | Runs the query across Client, Project, Proposal, Deliverable, and Invoice records, applying scope rules and ranking, with retry and offline fallback behavior |
| FEAT-28.SPEC-003 | Search Scope & Access Rules | Logic/Rule | Nadia, Dana | Governs who may search, what account a search may ever touch, and the minimum query length |
| FEAT-28.SPEC-004 | Search Result Relevance Ranking | Logic/Rule | Nadia, Dana | Defines how matches across five different entity types are scored, ordered, and tie-broken into one ranked list |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Search across entity types — clients, projects, proposals, invoices in one query | FEAT-28.SPEC-001, FEAT-28.SPEC-002 | The screen provides the query box and results panel; the automation executes the query across all five entity types and returns the ranked set | Phase 2 (Explicit) |
| Jump to a result — selecting a result opens it directly | FEAT-28.SPEC-001 | Selecting a result row navigates directly to the underlying client, project, proposal, deliverable, or invoice screen (inline interaction on the results panel) | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 4-5:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-28.SPEC-002 | Cross-Entity Search Execution | Phase 4 (Trigger-Response Analysis) | Typing a query is a trigger with a non-trivial, cross-entity response (query five entity types, apply scope, rank, handle failure and offline fallback) — too complex to leave inline in the Screen spec |
| FEAT-28.SPEC-003 | Search Scope & Access Rules | Phase 5 (Rule-Constraint Discovery) | Combines an authorization rule (only Nadia, and Dana only inside a scoped support session), an isolation rule (never another freelancer's account, XBR-09), and a validation rule (minimum 2-character query) — a rule set shared by both SPEC-001 and SPEC-002, crossing the inline threshold |
| FEAT-28.SPEC-004 | Search Result Relevance Ranking | Phase 5 (Rule-Constraint Discovery) | Data Notes names relevance ranking as a Derived data point; scoring and ordering matches across five heterogeneous entity types is non-trivial derivation logic (decision-table-style tie-breaking), which crosses the inline threshold for a standalone Logic/Rule spec |

## Entity-Lifecycle Coverage Matrix

N/A — FEAT-28's Connected Entities field is explicitly "N/A — search reads across existing entities rather than owning one" (product-features.md). The feature creates, updates, deletes, and archives nothing; it only reads records owned and lifecycle-managed by other features. No CRUD Coverage Matrix applies.

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Client | FEAT-28.SPEC-002 | Matched and displayed as a search result (dependency map: FEAT-28 listed as a Client reader; owned/lifecycle-managed by FEAT-01) |
| Project | FEAT-28.SPEC-002 | Matched and displayed as a search result (owned/lifecycle-managed by FEAT-01) |
| Proposal | FEAT-28.SPEC-002 | Matched and displayed as a search result (owned/lifecycle-managed by FEAT-02) |
| Deliverable | FEAT-28.SPEC-002 | Matched and displayed as a search result (owned/lifecycle-managed by FEAT-06; named in Interactions though not in the Data Notes Source line) |
| Invoice | FEAT-28.SPEC-002 | Matched and displayed as a search result, shown with its own currency and never aggregated across results (XBR-18) (owned/lifecycle-managed by FEAT-09) |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Freelancer (or Dana, in a scoped support session) types a query of 2+ characters | Query executes across Client, Project, Proposal, Deliverable, and Invoice, scoped per Search Scope & Access Rules, and results are ranked | Standalone Automation | FEAT-28.SPEC-002 |
| Search executes | Scope is checked: caller's role, account isolation, and minimum query length are enforced before any records are matched | Standalone Logic/Rule | FEAT-28.SPEC-003 |
| Dana opens a logged support session for one freelancer account and searches | Search results are narrowed to that one account only, never the operator's own account or any other | Standalone Logic/Rule | FEAT-28.SPEC-003 |
| Search returns candidate matches across entity types | Matches are scored and ordered into one ranked list, with ties broken consistently | Standalone Logic/Rule | FEAT-28.SPEC-004 |
| Freelancer selects a result | Navigates directly to the underlying client, project, proposal, deliverable, or invoice screen | Inline in triggering screen | FEAT-28.SPEC-001 |
| Query matches nothing | Results panel shows a clear "no results" state rather than an empty blank area | Inline in triggering screen | FEAT-28.SPEC-001 |
| Freelancer is typing / query is in flight | Results panel shows a lightweight in-progress indicator | Inline in triggering screen | FEAT-28.SPEC-001 |
| A search request fails | Automatically retries once before surfacing a manual retry option | Automation failure handling (part of the automation's own outcome coverage) | FEAT-28.SPEC-002 |
| Device is offline or the search capability is degraded | Falls back to whatever results were most recently loaded locally, clearly marked as possibly stale | Inline in triggering screen | FEAT-28.SPEC-001 |
| An invoice or proposal result is shown | Any amount is displayed in its own record's currency; amounts across different results are never converted or summed (XBR-18) | Inline in triggering screen | FEAT-28.SPEC-001 |

## Shared Context

**Shared Entities:**
- Client, Project, Proposal, Deliverable, Invoice -- all read-only for this feature. Matched and scored by FEAT-28.SPEC-002 (per FEAT-28.SPEC-003's scope rules and FEAT-28.SPEC-004's ranking rules); displayed as result rows by FEAT-28.SPEC-001. No field is written by this feature.

**Shared UI Patterns:**
- Result row/card -- one consistent pattern across all five entity types on FEAT-28.SPEC-001: a type indicator (client / project / proposal / deliverable / invoice), a primary line (name or identifying reference), a secondary line (parent context, e.g. project under its client), and, for Invoice results only, an amount shown in that invoice's own currency. Spec Writers should describe all five result-row variants consistently against this one pattern rather than as separate layouts.

**Shared Validation:**
- FEAT-28.SPEC-003 defines the minimum 2-character query rule and the role/account scoping rule. FEAT-28.SPEC-001 (the query box) and FEAT-28.SPEC-002 (the execution automation) both reference FEAT-28.SPEC-003 rather than restating these rules.

## Internal Dependency Map

```
SPEC-001 (Global Search) -> [freelancer/Dana types 2+ characters] -> SPEC-002 (Cross-Entity Search Execution)
SPEC-002 (Cross-Entity Search Execution) -> [checks caller role, account, query length] -> SPEC-003 (Search Scope & Access Rules)
SPEC-002 (Cross-Entity Search Execution) -> [scores and orders matches] -> SPEC-004 (Search Result Relevance Ranking)
SPEC-002 (Cross-Entity Search Execution) -> [returns ranked results, or failure/offline fallback] -> SPEC-001 (Global Search)
SPEC-001 (Global Search) -> [user selects a result] -> {FEAT-01 / FEAT-02 / FEAT-06 / FEAT-09 screen for that record}
```

**Default Entry:** SPEC-001 (Global Search) -- the screen shown when the freelancer (or Dana, inside a scoped support session) opens global search.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-28.SPEC-001 | Outbound | FEAT-01 (Client & Project Management) | Selecting a client or project result opens that record's detail screen | Freelancer selects a client or project result |
| FEAT-28.SPEC-001 | Outbound | FEAT-02 (Proposal Creation & Sending) | Selecting a proposal result opens that proposal | Freelancer selects a proposal result |
| FEAT-28.SPEC-001 | Outbound | FEAT-06 (Deliverable Upload & Sharing) | Selecting a deliverable result opens that deliverable | Freelancer selects a deliverable result |
| FEAT-28.SPEC-001 | Outbound | FEAT-09 (Invoice Generation & Sending) | Selecting an invoice result opens that invoice | Freelancer selects an invoice result |
| FEAT-28.SPEC-002 | Inbound | FEAT-01 (Client & Project Management) | Reads Client and Project records to match against the query | Freelancer or Dana performs a search |
| FEAT-28.SPEC-002 | Inbound | FEAT-02 (Proposal Creation & Sending) | Reads Proposal records to match against the query | Freelancer or Dana performs a search |
| FEAT-28.SPEC-002 | Inbound | FEAT-06 (Deliverable Upload & Sharing) | Reads Deliverable records to match against the query | Freelancer or Dana performs a search |
| FEAT-28.SPEC-002 | Inbound | FEAT-09 (Invoice Generation & Sending) | Reads Invoice records to match against the query | Freelancer or Dana performs a search |
| FEAT-28.SPEC-003 | Inbound | FEAT-31 (Operator Support Access) | A logged support session narrows search scope to the one account Dana is helping, per XBR-29's read-only, one-account boundary | Dana opens a support session and performs a search |
| FEAT-28.SPEC-001 | Outbound | FEAT-12 (Freelancer Financial Dashboard) / FEAT-01 (Client & Project Management) | Search is the shortcut to the same client-drill-down destination these journeys reach today by manual navigation ("Month-End Financial Review" step 2; "Pointing to the Record in a Scope Dispute") | Freelancer uses search instead of browsing once her roster has grown |

## Non-Functional Notes

**Data volumes / growth:** Each search reads across up to 3–15 active clients and their full project, proposal, and invoice history per freelancer account (assumptions-constraints.md, ASMP-22); search must stay responsive at that scale from MVP onward rather than being phased in for scale reasons (scope-boundaries.md, SC-21).

**Responsiveness:** No ASMP entry names FEAT-28's latency directly (ASMP-21 targets client-facing portal pages, which this feature never touches — Nadia and Dana are its only users). The feature's own States field sets the bar instead: results should feel responsive as the freelancer types, shown via a lightweight in-progress indicator rather than a blank wait, consistent with ASMP-27's general expectation that every screen shows real progress while loading.

**Data sensitivity / privacy:** Results surface data at the same sensitivity as its source: Client billing name/address (GDPR-class, ASMP-24), commercially confidential Proposal scope and price, and Invoice financial and billing data (GDPR-class, evidentiary, ASMP-24). Search inherits — and must never loosen — the source entities' isolation: results are strictly limited to the searching account's own data (Validation & Limits field; ASMP-23), and Dana's results are further limited to the one account she is actively supporting (FEAT-31).

**Compliance flags:** N/A — search introduces no compliance surface of its own: it captures nothing (Data Notes: "Captured: none"), so no new retention, export, or deletion obligation arises beyond what already applies to the Client, Project, Proposal, Deliverable, and Invoice records it reads (ASMP-23, ASMP-24).

## Non-Goals

- **Cross-account or multi-freelancer search** -- Excluded per the Validation & Limits field and BRIEF.md's Privacy constraint: search scope is limited strictly to the freelancer's own data. Dana's exception is a read-only search inside one logged, freelancer-announced support session scoped to a single account (FEAT-31) — never a general cross-account capability.
- **Sub-2-character queries returning results** -- Excluded per the Validation & Limits field: a minimum 2-character query is required specifically to avoid overly broad result sets.
- **Search as a permissioned, multi-tier team feature** -- Excluded per scope-boundaries.md (SC-01): the product has no team-of-many or internal-staff seat model, so search recognizes only Nadia (full) and Dana (read-only, session-scoped) — no additional search-permission tiers exist to build.
- **Client-contact-facing search** -- Excluded per the Access field: no client contact (Owen or Priya) has cross-account search; their portal experience remains scoped, own-company browsing only, consistent with client isolation (XBR-09).
- **A native mobile search experience** -- Excluded per scope-boundaries.md (SC-06): the product ships no native apps; global search is delivered as part of the same web app as every other freelancer-side screen.



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



# Logic/Rule Spec: Search Scope & Access Rules

## Overview

**Name:** Search Scope & Access Rules
**ID:** FEAT-28.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs who may perform a global search, which single account any search is ever allowed to touch, and the minimum query length required before a search executes.
**Parent Feature:** FEAT-28 -- Global Search Across Clients & Projects
**Governed Entity:** Search Request

## Scope and Non-Goals

**In Scope:**
- The minimum query-length rule that must pass before any search executes
- Authorization for who may perform a search, and under what condition (an open, account-scoped support session, for Dana)
- Deriving the single account a search is ever allowed to touch, and the isolation rule that a search never crosses that boundary
- Every field of the Search Request and how each is populated or validated

**Non-Goals:**
- Matching logic against Client, Project, Proposal, Deliverable, and Invoice records -- handled by FEAT-28.SPEC-002 (Cross-Entity Search Execution), which enforces this spec's rules before matching rather than restating them
- Scoring, ordering, and tie-breaking matched records -- handled by FEAT-28.SPEC-004 (Search Result Relevance Ranking); scope and ranking are deliberately separate concerns per feature-overview.md's Analyst-Discovered Specs rationale
- Defining or opening a support session itself -- owned entirely by FEAT-31 (Operator Support Access); this spec only reads whether one is open and which account it names
- Any additional search-permission tier beyond Nadia (full) and Dana (read-only, session-scoped) -- excluded per scope-boundaries.md (SC-01): the product has no team-of-many or internal-staff seat model, so no further permission tier exists to build

## Governed Entity

The Search Request is an ephemeral, non-persisted concept scoped to this feature: feature-overview.md's Entity-Lifecycle Coverage Matrix confirms FEAT-28 "creates, updates, deletes, and archives nothing," so this entity has no lifecycle in the Feature Dependency Map. Its fields are defined here, from the Feature Breakdown Brief's Side-Effect Inventory and Validation & Limits, rather than sourced from a dependency-map entity.

**Entity:** Search Request
**Source:** FEAT-28 Feature Breakdown Brief (ephemeral request; not a stored entity in the Feature Dependency Map)

| Field | Data Type | Description |
|-------|-----------|-------------|
| query_text | text | The text the freelancer or Dana has typed into the search box |
| requesting_role | enum (Nadia \| Dana) | Which persona is performing the search |
| support_session_reference | derived | For Dana only: a reference to her currently open Support Access Session (FEAT-31); absent for Nadia |
| target_account | derived | The single freelancer account this search is permitted to touch |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-28.SPEC-001 | Global Search | Query length checked continuously as text is entered; authorization checked on screen entry (whether the search entry point is shown at all) |
| FEAT-28.SPEC-002 | Cross-Entity Search Execution | Full scope check (length, role, account) re-verified immediately before matching begins; a failing check blocks matching rather than producing partial results |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| query_text | Minimum 2 characters, counting only non-whitespace characters | Always | On every change, continuously as it is typed | No error message shown -- below the minimum, the search entry point simply shows no results panel and no indicator (FEAT-28.SPEC-001, Below Minimum Length state) | Yes -- no search executes below the minimum |
| query_text | No validation beyond data type above the minimum length | Always | -- | -- | -- |
| requesting_role | Must be Nadia or Dana | Always | Before any search executes | "Global search requires an active support session for one account." (shown only to Dana outside a session; Nadia is never shown this, since her requesting_role is always valid) | Yes |
| support_session_reference | Required and must reference exactly one open Support Access Session naming exactly one freelancer account | Only when requesting_role is Dana | Before any search executes | "Global search requires an active support session for one account." | Yes |
| target_account | No validation beyond data type -- it is fully derived, never entered by the requester | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Session required for operator searches | requesting_role, support_session_reference | If requesting_role is Dana, support_session_reference must be present and reference exactly one open Support Access Session; if requesting_role is Nadia, support_session_reference is never required and is always absent | "Global search requires an active support session for one account." |
| Account derivation and isolation | requesting_role, support_session_reference, target_account | target_account is derived as Nadia's own freelancer account when requesting_role is Nadia, or as the single account named by support_session_reference when requesting_role is Dana; a search is never executed against any other account, per ASMP-23 and the account-level analog of XBR-09 | N/A -- this rule has no user-facing violation state; a Search Request with no derivable target_account per the row above never reaches execution |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Execute search query | Nadia | Always, scoped to her own account | -- |
| Execute search query | Dana | Only while a Support Access Session is open, naming exactly one freelancer account (FEAT-31) | Outside a session, the search entry point is not shown in her console (control hidden); a direct attempt shows "Global search requires an active support session for one account." |
| Execute search query | Owen (Client Primary Contact) | Never | The client portal's navigation carries no search entry point at all -- excluded per feature-overview.md's Non-Goals ("Client-contact-facing search") |
| Execute search query | Priya (Client Reviewer Contact) | Never | Same as Owen -- no search entry point exists in the client portal |
| View search results | Nadia | Always, limited to results matched from her own account | -- |
| View search results | Dana | Only while a Support Access Session is open, limited to results matched from the one account it names -- never her own account (she has none) and never any other freelancer's | Same denied experience as "Execute search query" for Dana; results are never computed or shown outside a session |
| View search results | Owen | Never | Same as "Execute search query" for Owen |
| View search results | Priya | Never | Same as "Execute search query" for Priya |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| target_account | Nadia's own freelancer account, when requesting_role is Nadia; otherwise the single account named by her currently open Support Access Session, when requesting_role is Dana | On every search request | No -- never overridable by the requester; this is the isolation boundary itself |
| support_session_reference | The requester's currently open Support Access Session, when requesting_role is Dana; otherwise absent | On every search request | No |

## Business Rules

- ASMP-23 / the account-level analog of XBR-09: search inherits, and must never loosen, the isolation the source entities already enforce -- a search request's target_account is always exactly one freelancer account, never more than one and never zero once execution proceeds.
- FEAT-31's XBR-29: Dana's search access is read-only in every respect, covers exactly the one account her open session names, ends the instant that session closes (per FEAT-31's own inactivity and closing rules), and is always covered by the session's own logging and freelancer notification -- this spec adds no separate logging of its own, since search itself writes no Activity Log Entry (feature-overview.md, Data Notes: "Captured: none").
- The minimum query-length rule exists specifically to avoid overly broad result sets (feature-overview.md, Non-Goals) -- it is evaluated purely on the query text and never varies by role, account, or entity type being searched.
- These rules are evaluated identically whether the caller is FEAT-28.SPEC-001 (screen-level gate on whether search is offered at all) or FEAT-28.SPEC-002 (execution-level gate immediately before matching) -- there is no relaxed enforcement path for either.

## Edge Cases

- **Query is exactly 2 characters** -- Passes the minimum-length rule; search executes. A query of exactly 1 character does not.
- **Query is 2+ characters but entirely whitespace** -- Counted by non-whitespace characters only, so a whitespace-only string of any length is treated as below the minimum; search does not execute.
- **Dana's support session closes at the exact moment a query is submitted** -- The session-required cross-field rule is re-evaluated at execution time (FEAT-28.SPEC-002's enforcement point), not only at screen entry; a session that has just closed causes the query to be denied even if the search entry point was still visible a moment earlier.
- **Dana has two support sessions open in immediate succession for two different accounts (the first closed, the second opened)** -- target_account is derived from whichever session is open and active at the moment of execution; a stale reference to an already-closed session never resolves to a target_account, so no query can execute against it.
- **Nadia opens the same search in two browser sessions at once** -- Each executes independently, both deriving the same target_account (her own); read-only searches never contend with each other, so no conflict-resolution behavior is needed here.
- **A role not in the Access Matrix's four rows (e.g., an unauthenticated visitor) attempts to reach search** -- Handled entirely by FEAT-28.SPEC-001's Access and Visibility table (Unauthenticated / Expired session rows); this spec's Authorization Rules table governs only the four Access Matrix roles, since requesting_role by definition requires an authenticated Nadia or Dana identity to exist at all.

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Search Result Relevance Ranking

## Overview

**Name:** Search Result Relevance Ranking
**ID:** FEAT-28.SPEC-004
**Type:** Logic/Rule
**Purpose:** Defines how candidate matches across Client, Project, Proposal, Deliverable, and Invoice records are scored, ordered, and tie-broken into the single ranked list the freelancer (or Dana) sees.
**Parent Feature:** FEAT-28 -- Global Search Across Clients & Projects
**Governed Entity:** Search Match

## Scope and Non-Goals

**In Scope:**
- The scoring tiers applied to every candidate match, in priority order
- The identifying ("primary") and contextual ("secondary") matched field per entity type
- Recency and entity-type tie-breaking when scoring tiers are equal
- A fully deterministic final order for every possible candidate set, including exact ties

**Non-Goals:**
- Deciding which records are candidates in the first place (matching against the query) -- handled by FEAT-28.SPEC-002 (Cross-Entity Search Execution), which produces the candidate set this spec ranks
- Deciding who may search or which account a search may touch -- handled entirely by FEAT-28.SPEC-003 (Search Scope & Access Rules); ranking only ever operates on a candidate set FEAT-28.SPEC-002 has already scoped correctly
- Capping or paginating the number of ranked results -- excluded per the Brief's Non-Functional Notes: the product commits to staying responsive at its stated scale (3-15 active clients and full history, ASMP-22) without needing an artificial cap from MVP onward
- Formatting or displaying the ranked rows -- owned by FEAT-28.SPEC-001 (Global Search), which presents exactly the order this spec produces without re-sorting

## Governed Entity

The Search Match is an ephemeral, non-persisted concept scoped to this feature -- one match record exists per candidate hit that FEAT-28.SPEC-002 produces before ranking, and none of it is stored. It has no lifecycle in the Feature Dependency Map, consistent with feature-overview.md's Entity-Lifecycle Coverage Matrix ("creates, updates, deletes, and archives nothing"); its fields are defined here from the Brief's Data Notes ("Derived: relevance ranking") rather than sourced from a dependency-map entity.

**Entity:** Search Match
**Source:** FEAT-28 Feature Breakdown Brief (ephemeral, derived candidate; not a stored entity in the Feature Dependency Map)

| Field | Data Type | Description |
|-------|-----------|-------------|
| entity_type | enum (Client \| Project \| Proposal \| Deliverable \| Invoice) | Which kind of record this candidate match belongs to |
| matched_field | enum (primary \| secondary) | Whether the query matched the record's primary identifying field or a secondary context field |
| match_type | enum (exact \| prefix \| partial) | How closely the query text matched the field: an exact full-field match, a match at the start of the field (prefix), or a match found elsewhere within the field (partial) |
| source_updated_at | date | The most recent relevant update timestamp on the underlying record, used only for tie-breaking |
| source_reference | derived | A reference to the specific underlying record this match represents |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-28.SPEC-002 | Cross-Entity Search Execution | Applied once, after matching completes and before the ranked list is returned to the triggering screen |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| entity_type | Must be one of Client, Project, Proposal, Deliverable, Invoice | Always | On receipt from FEAT-28.SPEC-002 | N/A -- this field is system-derived, never user-entered, so no user-facing error applies | No (a candidate outside this set cannot occur, since FEAT-28.SPEC-002 only ever produces these five types) |
| matched_field | Must be one of primary, secondary | Always | On receipt | N/A -- system-derived | No |
| match_type | Must be one of exact, prefix, partial | Always | On receipt | N/A -- system-derived | No |
| source_updated_at | No validation beyond data type | Always | -- | -- | -- |
| source_reference | No validation beyond data type -- must resolve to exactly one underlying record | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Primary field per entity type | entity_type, matched_field | The primary identifying field is: Client -> client_name; Project -> project_name; Proposal -> scope_description (proposals carry no name field); Deliverable -> its "file or link" (the uploaded file's name, or the linked asset's title or address); Invoice -> invoice_number. Every other queryable field on that entity type (a parent client's client_name or a parent project's project_name) is secondary; Milestone is never queried, so no milestone field is primary or secondary | N/A -- this rule defines classification, not a validation failure |
| Match type takes priority over matched-field priority | matched_field, match_type | An exact match on a secondary field outranks a partial match on a primary field (match_type is evaluated before matched_field in the scoring order below) | N/A |

## Authorization Rules

Ranking applies only after FEAT-28.SPEC-003 has already authorized the search and derived its target_account; this spec introduces no separate authorization decision of its own. The single action below exists so the Access Matrix's full role set is addressed here as well.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View ranked search results | Nadia | Always, for results already scoped to her own account by FEAT-28.SPEC-003 | -- |
| View ranked search results | Dana | Only for results already scoped to the one account her open support session names, per FEAT-28.SPEC-003 | Same denied experience as FEAT-28.SPEC-003's "View search results" row -- no ranked list is ever produced outside a session |
| View ranked search results | Owen (Client Primary Contact) | Never | Same as FEAT-28.SPEC-003 -- no search entry point exists, so no ranked list is ever produced for this role |
| View ranked search results | Priya (Client Reviewer Contact) | Never | Same as Owen |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Match score | Derived per candidate from three ordered tiers, most significant first: (1) match_type -- exact ranks above prefix, which ranks above partial; (2) matched_field -- within the same match_type, a match on the primary field ranks above a match on a secondary field; (3) source_updated_at -- within the same match_type and matched_field tier, the more recently updated record ranks higher | On every ranking pass, for every candidate | No -- ranking is never user-adjustable; there is no sort control on FEAT-28.SPEC-001 |
| Final ranked order | The full candidate set is ordered by descending match score (per the three tiers above); when two candidates remain exactly tied after all three tiers (identical match_type, matched_field, and source_updated_at), they are ordered by entity_type using the fixed sequence Client, Project, Proposal, Deliverable, Invoice, and if still tied by source_reference in a stable, deterministic order so that identical queries never reorder between runs | On every ranking pass | No |

## Business Rules

- Match type is the single strongest ranking signal: an exact match anywhere always outranks a prefix match anywhere, which always outranks a partial match anywhere, before matched-field priority or recency is ever considered.
- Within one match-type tier, a match on an entity's primary identifying field (client_name, project_name, scope_description, the deliverable's file or link name/title/address, or invoice_number) always outranks a match on a secondary context field (a parent client's client_name or a parent project's project_name) for that same candidate.
- Recency (source_updated_at) is used only as a tie-break within an already-equal match_type and matched_field tier -- it never overrides a stronger match_type or matched_field result, so a five-year-old exact match on a client_name always outranks a match made yesterday on a secondary field.
- The fixed entity-type order (Client, Project, Proposal, Deliverable, Invoice) is used solely as a final, deterministic tie-break when every other dimension is exactly equal -- it never overrides match_type, matched_field, or recency, and carries no meaning about which entity type matters more to the freelancer.
- Ranking never groups results by entity type -- the shared result-row pattern (FEAT-28.SPEC-001) presents one interleaved list ordered purely by score, since the Brief describes "one ranked list," not five separate lists.

## Edge Cases

- **Two candidates tie on match_type and matched_field but have different source_updated_at values** -- The more recently updated record ranks first; this is the recency tie-break, and it is the only case in which two candidates of different entity types can be reordered relative to their entity-type position.
- **Two candidates tie on match_type, matched_field, and source_updated_at exactly (e.g., updated at the identical moment)** -- Ordered next by the fixed entity-type sequence; if both share the same entity_type too, ordered by source_reference in a stable order so the tie never resolves differently between identical searches.
- **A query matches the same underlying record on both its primary and a secondary field** (e.g., a project name query also matches that project's owning client's client_name) -- Each qualifying field produces its own candidate match; the record can appear once per distinct matched field it satisfies, and each instance is scored independently by its own matched_field tier.
- **A query is an exact match on a secondary field for one candidate and a prefix match on a primary field for another** -- The exact match on the secondary field ranks first, because match_type is evaluated before matched_field.
- **All candidates are of a single entity type** (e.g., only Client matches exist) -- Ranking proceeds identically; the entity-type tie-break is simply never exercised, since no cross-type tie can occur.
- **Zero candidates** -- Ranking has nothing to order; this produces the empty ranked list that FEAT-28.SPEC-002 treats as the No Results outcome, not an error in this spec.

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
