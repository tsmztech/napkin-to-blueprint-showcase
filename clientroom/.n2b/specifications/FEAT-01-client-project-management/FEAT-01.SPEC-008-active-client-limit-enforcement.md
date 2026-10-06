---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-01.SPEC-008
spec_name: Active Client Limit Enforcement
spec_slug: active-client-limit-enforcement
parent_feature: FEAT-01
parent_feature_name: Client & Project Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 9
acceptance_criteria_count: 9
---

# Logic/Rule Spec: Active Client Limit Enforcement

## Overview

**Name:** Active Client Limit Enforcement
**ID:** FEAT-01.SPEC-008
**Type:** Logic/Rule
**Purpose:** Gates adding or reactivating an active client against the freelancer's current Subscription Plan limit.
**Parent Feature:** FEAT-01 -- Client & Project Management
**Governed Entity:** Client (specifically the create and reactivate transitions, gated against the Subscription Plan entity)

## Scope and Non-Goals

**In Scope:**
- The rule that gates creating a new Client (FEAT-01.SPEC-001) and reactivating an Archived Client (FEAT-01.SPEC-004) against the freelancer's current active-client limit
- Reading the Subscription Plan's tier and active_client_count to evaluate the limit
- The exact blocked experience and the upgrade hand-off to FEAT-23

**Non-Goals:**
- Defining the Subscription Plan's tiers, pricing, or upgrade flow itself -- owned entirely by Subscription Plan & Billing Management (FEAT-23); this spec only reads the plan's current state
- Client field validation (name, billing details) -- handled by FEAT-01.SPEC-001 (inline) and FEAT-01.SPEC-010 (billing completeness)
- Client delete eligibility -- a distinct rule owned by FEAT-01.SPEC-009
- Enforcing a limit on Project count -- the product defines no cap on projects per client; only the Client entity's active count is limited, per BRIEF.md's Business Context describing the plan as priced by number of active clients

## Governed Entity

**Entity:** Client
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| client_name | text | Company name |
| billing_name | text | Name printed on invoices |
| billing_address | text | Address printed on invoices |
| tax_id | text | Client's tax identifier (optional) |
| status | enum (Active, Archived) | Determines whether the client counts toward the active-client limit |
| currency and tax treatment | derived / configured (via FEAT-15) | Per-client/project billing currency and tax label/rate |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-001 | Add Client | On save, before the new Client record is committed |
| FEAT-01.SPEC-004 | Client Detail | On Reactivate, before status changes from Archived to Active |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| status | May only be set to Active when the active-client limit allows it | On create (new clients default to Active) and on reactivation (Archived to Active) | On submit | See Authorization Rules and Business Rules for the exact blocked messages | Yes |
| client_name | No validation beyond data type in this spec | Always | -- | -- | -- (governed by FEAT-01.SPEC-001's own inline rule) |
| billing_name, billing_address, tax_id | No validation beyond data type in this spec | Always | -- | -- | -- (governed by FEAT-01.SPEC-010) |
| currency and tax treatment | No validation beyond data type in this spec | Always | -- | -- | -- (governed by FEAT-15) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Active-count-against-limit | status (of the Client being created/reactivated), Subscription Plan.active_client_count, Subscription Plan.tier | The transition to status = Active is only permitted when active_client_count (including this one, if permitted) does not exceed the limit for the current tier | "You've reached your plan's active client limit. Upgrade to add more clients." (create) / "You've reached your plan's active client limit. Upgrade to reactivate this client." (reactivate) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create a client (results in an Active client) | Nadia (Freelancer) | Only when the resulting active-client count does not exceed her Subscription Plan's limit (platform parameter: `free-tier-active-client-limit` for the free tier; unlimited on a paid plan) | Save is blocked; entered form data is preserved; blocking message "You've reached your plan's active client limit. Upgrade to add more clients." with an "Upgrade" action opening FEAT-23 |
| Reactivate an archived client (Active) | Nadia (Freelancer) | Only when the resulting active-client count does not exceed her Subscription Plan's limit | Reactivation is blocked; the client stays Archived; blocking message "You've reached your plan's active client limit. Upgrade to reactivate this client." with an "Upgrade" action opening FEAT-23 |
| Create a client, or reactivate one, on a paid plan | Nadia (Freelancer) | Always -- a paid plan carries no active-client cap | -- |
| Create a client | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never | Not shown; the Client & Project Management area does not exist in either contact's portal navigation |
| Reactivate a client | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never | Not shown; the Client & Project Management area does not exist in either contact's portal navigation |
| Create a client | Dana (Support Operator) | Never | Add Client is not part of Dana's read-only support session (FEAT-31); the form is not reachable from her session |
| Reactivate a client | Dana (Support Operator) | Never | The Reactivate control is not rendered in Dana's read-only support session (FEAT-31) |
| View a client's active-client-limit status, read-only | Dana (Support Operator) | Always, inside a logged support session (FEAT-31) | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| status | Active | On create, when the limit check passes | No (a create that fails the limit check does not persist a Client record at all) |
| Subscription Plan.active_client_count | Derived: count of Client records with status = Active, owned by the freelancer account | Recalculated whenever a Client's status changes | No -- this is a read-only derived value consumed by this spec's checks, owned by FEAT-23 |

## Business Rules

- XBR-23: adding or reactivating an active client beyond the free-tier limit requires an active paid plan; when a paid plan ends, no data is lost and existing portals stay reachable, but adding clients beyond the limit is blocked.
- The limit check runs at the moment a client is added or reactivated, against the Subscription Plan state as it stands at that moment (reject-with-refresh per the dependency map's Contention note for Subscription Plan); a plan change completed in another session between screen load and save is picked up by this fresh check.
- The free-tier limit value is a platform-set policy value and is referenced only as platform parameter: `free-tier-active-client-limit`; Pass D's reconciler collects this marker into `specifications/platform-parameters.md` with a proposed default, and Gate A reconciles it against that registry.
- Archiving a client always reduces the active-client count immediately; this reduction is unconditional and never itself blocked (only the create/reactivate direction is gated).

## Edge Cases

- **Nadia is exactly at the limit and archives one client, then immediately tries to add a new one in the same session** -- The archive's reduction to active_client_count is applied before the next add's check runs, so the add succeeds if no other change intervenes; if a concurrent session's add already claimed the freed slot, this add is blocked with the standard limit message and Nadia is shown the current count on refresh.
- **Nadia's plan lapses (Subscription Plan.status becomes Lapsed) while she is already over what the free tier would allow** -- Existing clients stay Active (XBR-23: no data is lost), but any further create or reactivate attempt is blocked by this rule until she is back on a plan whose limit accommodates the current active count.
- **Two Add Client attempts from two open sessions, both at exactly one slot below the limit** -- Whichever save commits first succeeds and consumes the slot; the second is rejected with the limit message and its entered data is preserved, per the standard reject-with-refresh resolution for Subscription Plan.
- **Reactivating a client whose plan check would exceed the limit by more than one slot (e.g., bulk state change is not offered, but the check itself is evaluated per single reactivation)** -- Not applicable: this product offers no bulk reactivation; each reactivation is evaluated individually against the limit at that moment.

## Acceptance Criteria

**FEAT-01.SPEC-008-AC-01:** Given Nadia is on the free tier with fewer active clients than platform parameter: `free-tier-active-client-limit`, when she adds a new client, then the client saves as Active and the count increments.

**FEAT-01.SPEC-008-AC-02:** Given Nadia is on the free tier already at platform parameter: `free-tier-active-client-limit` active clients, when she attempts to add a new client, then the save is blocked with "You've reached your plan's active client limit. Upgrade to add more clients." and her entered data is preserved.

**FEAT-01.SPEC-008-AC-03:** Given Nadia is on the free tier at her limit, when she attempts to reactivate an archived client, then the reactivation is blocked with "You've reached your plan's active client limit. Upgrade to reactivate this client." and the client stays Archived.

**FEAT-01.SPEC-008-AC-04:** Given Nadia is on a paid plan, when she adds a new client regardless of her current active-client count, then the save succeeds with no limit block.

**FEAT-01.SPEC-008-AC-05:** Given Nadia archives one of her active clients while at the free-tier limit, when she then adds a new client in the same session, then the add succeeds because the archive freed a slot.

**FEAT-01.SPEC-008-AC-06:** Given Nadia's paid plan lapses while she has more active clients than the free-tier limit allows, then all of her existing active clients remain Active and reachable, but any further add or reactivate is blocked until she is on a plan that accommodates the current count.

**FEAT-01.SPEC-008-AC-07:** Given Owen (Client Primary Contact) has no access to the Client & Project Management area, when he looks for any client-limit-related control, then none is shown, since this capability does not exist in his portal.

**FEAT-01.SPEC-008-AC-08:** Given Nadia has two sessions open and both attempt to add a client at exactly one slot below her limit at effectively the same time, when the first save commits, then the second is blocked with the standard limit message.

**FEAT-01.SPEC-008-AC-09:** Given Dana is in a read-only support session on Nadia's account, when she looks for a way to add or reactivate a client, then neither control is reachable from her session.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |
