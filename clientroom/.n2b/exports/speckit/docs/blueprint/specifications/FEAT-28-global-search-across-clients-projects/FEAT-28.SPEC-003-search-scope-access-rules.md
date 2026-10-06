---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-28.SPEC-003
spec_name: Search Scope & Access Rules
spec_slug: search-scope-access-rules
parent_feature: FEAT-28
parent_feature_name: Global Search Across Clients & Projects
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 21
acceptance_criteria_count: 13
---

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
