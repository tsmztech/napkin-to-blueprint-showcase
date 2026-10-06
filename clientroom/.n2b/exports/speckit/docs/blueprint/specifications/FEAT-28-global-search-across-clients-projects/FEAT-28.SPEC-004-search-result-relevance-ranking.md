---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-28.SPEC-004
spec_name: Search Result Relevance Ranking
spec_slug: search-result-relevance-ranking
parent_feature: FEAT-28
parent_feature_name: Global Search Across Clients & Projects
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 18
acceptance_criteria_count: 12
---

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
