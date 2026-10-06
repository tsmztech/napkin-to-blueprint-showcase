---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-01.SPEC-009
spec_name: Client Delete Eligibility
spec_slug: client-delete-eligibility
parent_feature: FEAT-01
parent_feature_name: Client & Project Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 7
acceptance_criteria_count: 9
---

# Logic/Rule Spec: Client Delete Eligibility

## Overview

**Name:** Client Delete Eligibility
**ID:** FEAT-01.SPEC-009
**Type:** Logic/Rule
**Purpose:** Determines whether a client may be hard-deleted (no sent proposal, invoice, or activity exists) versus archived only.
**Parent Feature:** FEAT-01 -- Client & Project Management
**Governed Entity:** Client (specifically the delete transition)

## Scope and Non-Goals

**In Scope:**
- The eligibility test that determines whether a Client can be permanently deleted
- The exact experience when a client is ineligible (Delete disabled, Archive offered instead)
- What deletion removes, and confirmation of irreversibility

**Non-Goals:**
- The archive path and its open-items confirmation -- a separate, less strict removal path owned by FEAT-01.SPEC-007
- Active-client limit checks -- owned by FEAT-01.SPEC-008; deletion reduces the active count but is not itself limit-gated
- Account-level deletion of all of a freelancer's data -- owned by Data Export & Account Deletion (FEAT-24), which removes client data as part of a full account close, subject to legal retention (XBR-33), a different scope than this feature's single-client delete

## Governed Entity

**Entity:** Client
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| client_name | text | Company name |
| billing_name | text | Name printed on invoices |
| billing_address | text | Address printed on invoices |
| tax_id | text | Client's tax identifier (optional) |
| status | enum (Active, Archived) | Not evaluated by this spec beyond confirming the client still exists at delete time |
| currency and tax treatment | derived / configured (via FEAT-15) | Not evaluated by this spec |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-004 | Client Detail | On opening the overflow menu (to enable/disable Delete) and again on the Delete confirm action |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| client_name, billing_name, billing_address, tax_id, status, currency and tax treatment | No validation beyond data type in this spec -- eligibility depends only on the presence of downstream records, not on any Client field's value | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Delete eligibility | Client (target), all Proposals under its Projects, all Invoices under its Projects, all Activity Log Entries referencing the client or its projects | Eligible only when zero sent Proposals, zero Invoices (any status), and zero Activity Log Entries exist for this client across every one of its projects | "This client has a sent proposal, invoice, or activity and can only be archived." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Delete a client | Nadia (Freelancer) | Only when the client has no sent proposal, no invoice, and no activity log entry (XBR-24) | Delete control is disabled in the overflow menu with inline text: "This client has a sent proposal, invoice, or activity and can only be archived." Archive remains available. |
| Delete a client | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never | Not shown; the Client & Project Management area does not exist in either contact's portal navigation |
| Delete a client | Dana (Support Operator) | Never | Delete is never rendered in Dana's read-only support session (FEAT-31); attempting to reach it directly has no effect |
| View a client's delete eligibility state | Nadia (Freelancer) | Always | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| delete_eligible (derived, not a stored field) | True only when the cross-field eligibility rule above evaluates true at the moment the overflow menu is opened, and re-evaluated at the moment Delete is confirmed | On screen render and on delete confirmation | No -- this is a computed gate, not a user-settable value |

## Business Rules

- XBR-24: client deletion is allowed only while the client has no sent proposal, invoice, or activity; anything with a record can only be archived, never erased.
- Eligibility is re-evaluated at the moment of commit, not only when the overflow menu was opened -- if a sent proposal, invoice, or activity is created by any other action in the interim (for example, an ad hoc invoice issued from another session), the delete is rejected at commit rather than allowed to proceed on stale eligibility.
- Because eligibility requires zero downstream records, an eligible delete has nothing to cascade -- its Projects (which by definition have no proposal, invoice, or activity either, since those are recorded at the project level under the client) and any Client Contacts are removed along with it, with no retention window, since nothing evidentiary exists to retain.
- Deletion is irreversible with no restore path, unlike Archive.

## Edge Cases

- **Client has a Draft (never-sent) proposal only** -- Eligible for delete: "sent proposal" specifically excludes an unsent draft, since a draft carries no evidence obligation; the draft proposal is removed along with the client.
- **Client has projects but every project is entirely empty (no proposal, invoice, or activity of any kind)** -- Eligible for delete; empty projects carry no record to protect.
- **Client had a sent proposal that was later voided and never accepted** -- Ineligible: a voided proposal is still a "sent proposal" that once existed and may be referenced in correspondence or disputes; only a draft that was truly never sent is exempt.
- **Eligibility check passes when the overflow menu opens, but an ad hoc invoice is issued for this client from another session before Nadia confirms Delete** -- The delete confirmation re-runs the eligibility check at commit and rejects with "This client has a sent proposal, invoice, or activity and can only be archived." rather than deleting against stale eligibility; this is the authorization boundary re-check required for this entity's Contention profile.
- **Client has an Activity Log Entry from a read-only support session (Dana having viewed it) but no proposal or invoice** -- A support-session view itself is not the kind of client-scoped activity this rule is testing for; only Activity Log Entries whose affected_record references this client or its projects (e.g., a client-side event) count. A support session viewing an otherwise-empty client does not, by itself, make that client ineligible for delete.

## Acceptance Criteria

**FEAT-01.SPEC-009-AC-01:** Given a client has no sent proposal, no invoice, and no activity, when Nadia opens the overflow menu, then Delete is enabled.

**FEAT-01.SPEC-009-AC-02:** Given a client has one sent invoice, when Nadia opens the overflow menu, then Delete is disabled with the inline text "This client has a sent proposal, invoice, or activity and can only be archived."

**FEAT-01.SPEC-009-AC-03:** Given a client is eligible and Nadia confirms Delete, then the client and its empty projects are permanently removed with no restore path.

**FEAT-01.SPEC-009-AC-04:** Given a client has only a Draft proposal that was never sent, when Nadia opens the overflow menu, then Delete is enabled, since an unsent draft does not count as a "sent proposal."

**FEAT-01.SPEC-009-AC-05:** Given a client had a proposal that was sent, then later voided and never accepted, when Nadia opens the overflow menu, then Delete is disabled, since a voided-but-once-sent proposal still counts.

**FEAT-01.SPEC-009-AC-06:** Given a client passes the eligibility check when Nadia opens the overflow menu, but an ad hoc invoice is issued for that client from another session before she confirms Delete, when she confirms, then the delete is rejected with "This client has a sent proposal, invoice, or activity and can only be archived."

**FEAT-01.SPEC-009-AC-07:** Given Owen has no access to the Client & Project Management area, when he looks for a delete control on any client, then none exists in his portal.

**FEAT-01.SPEC-009-AC-08:** Given Dana is in a read-only support session, when she views a client's overflow options, then no Delete control is rendered.

**FEAT-01.SPEC-009-AC-09:** Given a client's only activity is a read-only support session viewing it (no client-scoped event otherwise), when Nadia opens the overflow menu, then Delete eligibility is unaffected by that support-session view alone.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
