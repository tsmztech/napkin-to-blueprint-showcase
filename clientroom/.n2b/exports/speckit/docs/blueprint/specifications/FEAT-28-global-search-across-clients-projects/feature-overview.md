---
document_type: feature-overview
feature_number: FEAT-28
feature_name: Global Search Across Clients & Projects
feature_slug: global-search-across-clients-projects
priority_tier: Nice-to-Have
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 4
screen_count: 1
automation_count: 1
logic_rule_count: 2
integration_count: 0
notification_count: 0
---

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
