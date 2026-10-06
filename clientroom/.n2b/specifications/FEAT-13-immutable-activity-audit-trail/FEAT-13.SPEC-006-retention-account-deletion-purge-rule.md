---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-13.SPEC-006
spec_name: Retention & Account-Deletion Purge Rule
spec_slug: retention-account-deletion-purge-rule
parent_feature: FEAT-13
parent_feature_name: Immutable Activity & Audit Trail
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 12
acceptance_criteria_count: 16
---

# Logic/Rule Spec: Retention & Account-Deletion Purge Rule

## Overview

**Name:** Retention & Account-Deletion Purge Rule
**ID:** FEAT-13.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs how Activity Log Entries are retained for the life of the freelancer's account, removed only on account deletion subject to legal financial-record retention, and how a client contact's erasure request keeps their name on entries that are evidence.
**Parent Feature:** FEAT-13 -- Immutable Activity & Audit Trail
**Governed Entity:** Activity Log Entry (retention and removal dimension)

## Scope and Non-Goals

**In Scope:**
- The absence of any automatic, time-based purge of activity entries
- The single removal path -- full account deletion (FEAT-24) -- and its interaction with legal financial-record retention
- How a client contact's erasure request (FEAT-18) affects, and does not affect, entries that already name them as actor
- The permanence of the `actor` field's captured name, independent of the underlying Client Contact record's own lifecycle

**Non-Goals:**
- Field content and format rules -- owned by FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules); this spec addresses only retention timing and removal, never what a field must contain.
- Who may view entries while the account exists -- owned by FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules).
- The mechanics of account deletion itself (warnings, confirmation, disconnecting the payment account, what happens to other entities) -- owned by FEAT-24 (Data Export & Account Deletion); this spec only defines what happens to Activity Log Entry specifically within that process.

## Governed Entity

**Entity:** Activity Log Entry
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| event_type | enum | The kind of record-worthy event this entry describes |
| actor | text | The freelancer, a named client contact, the operator, or "Automatic" -- captured as a permanent name at write time |
| occurred_at | date (timestamp) | The exact date and time the event occurred |
| affected_record | text (reference) | A reference to the specific record the event concerns |
| project | text (reference) | The project the event belongs to, where it has one |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-24 (Data Export & Account Deletion) | Account deletion flow | At the moment account deletion executes -- applies this spec's retention carve-out before purging the freelancer's data, including Activity Log Entry |
| FEAT-18 (Client Contact Management & Roles) | Contact erasure request | At the moment an erasure request is processed -- applies this spec's rule that a contact's name stays on entries that already name them, even as their Client Contact details are removed |
| FEAT-13.SPEC-001 (Activity Trail) | Ongoing display | Relies on this spec to know that every entry it shows remains present for the life of the account, with no entry ever silently disappearing outside account deletion |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| actor | The name captured at write time is permanent and independent of the underlying Client Contact (or Freelancer Account) record's own lifecycle. If the named client contact is later erased (FEAT-18), the already-written `actor` value on this entry is never altered, blanked, or replaced. | Applies whenever the actor is a named client contact; not applicable when the actor is the freelancer, the operator, or "Automatic," none of which are subject to contact erasure | N/A -- this is a persistence guarantee, not an input check with a pass/fail moment | N/A -- there is nothing to reject; this rule describes what does not happen, not a validation failure | N/A |
| event_type | No retention-specific rule beyond the entity-level retention and removal rules in Business Rules below | Always | -- | -- | -- |
| occurred_at | No retention-specific rule beyond the entity-level retention and removal rules below | Always | -- | -- | -- |
| affected_record | No retention-specific rule beyond the entity-level retention and removal rules below | Always | -- | -- | -- |
| project | No retention-specific rule beyond the entity-level retention and removal rules below | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| N/A -- no cross-field interaction | N/A | This spec's rules operate at the whole-entity level (retention and removal of entries together) and at the single-field level (`actor` persistence through contact erasure); no rule here requires combining two or more fields' values together to determine an outcome | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Purge activity entries as part of full account deletion | Nadia (Freelancer) | Only as an inseparable part of deleting her entire account (FEAT-24); entries subject to legal financial-record retention are held back from the purge for that period even though the rest of the account is removed | No standalone "delete entries" or "clear trail" action exists anywhere in the product, for any role, including Nadia -- entries are never removable except as part of full account deletion. |
| Delete a single activity entry, at any time, outside account deletion | None -- no role, ever | Never | No per-entry delete action exists for any role, at any time; this reaffirms FEAT-13.SPEC-004's blanket prohibition on entry deletion in-product. |
| Trigger an automatic, time-based purge of entries | None -- there is no such action | There is no automatic time-based purge of activity entries at any scale, regardless of account age or entry count (scope-boundaries.md, SC-24) | N/A -- there is no control or process to deny; no such capability exists in the product to be attempted by any role. |
| Erase a contact's own details while entries already name them as actor | Nadia (Freelancer), executing a request the contact submits per FEAT-18 | The Client Contact record's own details (name field on that record, email, etc.) are removed; any Activity Log Entry that already names that contact in its `actor` field keeps that name exactly as written | N/A -- this action always succeeds for the contact's own Client Contact record; it never touches, and is never denied by, any existing Activity Log Entry. |
| Access entries retained past account deletion under legal financial-record retention | None -- no role has a product screen to reach them | Retained internally for platform parameter: `financial-record-legal-retention-period` after account deletion, solely to satisfy the legal retention requirement; the freelancer's account, and every screen that would display them, no longer exists once deletion completes | N/A -- there is no UI surface left to attempt access through once the account is deleted; retention is a backend obligation, not a product capability offered to any role. |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Entry retention status | Every entry defaults to "retained for the life of the account" from the moment it is written | On create | No -- Nadia cannot mark an individual entry for earlier removal; retention is entity-wide and tied only to account deletion |
| actor (post-erasure state) | Remains exactly as captured at write time, regardless of any later change or erasure to the underlying Client Contact record | Re-affirmed at the moment any erasure request is processed for that contact | No |

## Business Rules

- SC-24: activity entries are retained for the life of the freelancer's account, with no automatic purge of this history at any scale; when the account is deleted (FEAT-24), entries are removed except where a legal retention period for financial records applies.
- The legal financial-record retention period is a platform-set policy value, referenced only as platform parameter: `financial-record-legal-retention-period` -- Pass D's reconciler collects this marker into `specifications/platform-parameters.md` with a proposed default, and Gate A reconciles it against that registry.
- XBR-33: account deletion removes all the freelancer's data, including client contacts' personal data, and keeps only financial records under a legal retention requirement -- entries that are themselves, or directly support, a financial record (e.g., invoice sent, credit note issued, manual payment recorded, refund/reversal/cancellation recorded) fall under that carve-out; entries with no financial-record character are removed outright with the rest of the account.
- ASMP-20 and XBR-27: a contact's erasure request ends their access and removes their contact details immediately, while acceptances and approvals they gave remain on the record under their name as evidence -- this spec is the specific authority for how that principle applies to Activity Log Entry.
- No restore path exists for a purged entry, since no in-product delete exists in the first place (Entity-Lifecycle Coverage Matrix) -- account-deletion purge is a one-way action.
- No cascade-delete exists from any other entity's own deletion to Activity Log Entry outside the whole-account purge -- entries reference other records rather than owning them, so a referenced record being removed under some other feature's own rules (where that is even possible) does not, on its own, remove the entries that reference it.

## Edge Cases

- **Nadia deletes her account while she has invoices subject to legal financial-record retention** -- The invoice-related activity entries (invoice sent, credit note issued, manual payment recorded, refund/reversal/cancellation recorded) survive the purge for platform parameter: `financial-record-legal-retention-period`, alongside the Invoice and Payment records themselves, even though her account and every other, non-financial entry are removed.
- **A client contact submits an erasure request moments before the freelancer initiates account deletion** -- The erasure still processes first, removing the contact's own details while keeping their name on existing entries; the subsequent account deletion then purges entries exactly as it would have regardless of the erasure's timing, since the two are independent actions.
- **An entry names a contact who was later fully erased, and the freelancer generates a data export (FEAT-24) before deleting the account** -- The export includes the entry with the contact's name exactly as retained, since the export reads the same entry content the product itself would show Nadia.
- **Two client contacts at the same company submit erasure requests at different times, each having authored their own entries** -- Each erasure independently preserves its own contact's name on their own entries; there is no combined or batch behavior that treats the two requests differently from one another.
- **A freelancer's account is deleted with no invoices ever sent (no financial-record entries exist)** -- Account deletion purges every entry outright, with nothing retained, since the legal-retention carve-out only ever applies to entries that are themselves financial records or directly support one.

## Acceptance Criteria

**FEAT-13.SPEC-006-AC-01:** Given a freelancer account with activity entries of any age, when no account-deletion action has been taken, then no entry is ever automatically purged for age or volume alone.

**FEAT-13.SPEC-006-AC-02:** Given Nadia looks for a way to delete a single activity entry, when she searches every screen this feature offers, then no such control exists anywhere.

**FEAT-13.SPEC-006-AC-03:** Given Nadia initiates full account deletion through FEAT-24, when the deletion executes, then every activity entry with no financial-record character is removed as part of it.

**FEAT-13.SPEC-006-AC-04:** Given Nadia's account has invoice-sent and payment-recorded activity entries subject to legal financial-record retention, when her account deletion executes, then those specific entries are retained for platform parameter: `financial-record-legal-retention-period` rather than being purged immediately with the rest of her data.

**FEAT-13.SPEC-006-AC-05:** Given a client contact submits an erasure request through FEAT-18, when the request is processed, then their Client Contact details are removed while every existing entry that names them as actor keeps that name unchanged.

**FEAT-13.SPEC-006-AC-06:** Given a client contact was erased in the past, when Nadia opens an entry naming that contact today, then the entry still displays their name exactly as it was written.

**FEAT-13.SPEC-006-AC-07:** Given an erasure request is submitted moments before Nadia initiates account deletion, when both actions process, then the erasure's name-retention behavior applies first, and the subsequent deletion purges entries exactly as it otherwise would.

**FEAT-13.SPEC-006-AC-08:** Given Nadia generates a data export before deleting her account, when the export includes activity entries, then any entry naming a since-erased contact still shows that contact's retained name in the export.

**FEAT-13.SPEC-006-AC-09:** Given two contacts at the same client company each submit separate erasure requests, when both are processed, then each contact's name is independently retained on their own entries with no interaction between the two requests.

**FEAT-13.SPEC-006-AC-10:** Given a freelancer account with no invoices ever sent, when the account is deleted, then every activity entry is removed outright, since no entry qualifies for the legal-retention carve-out.

**FEAT-13.SPEC-006-AC-11:** Given the legal financial-record retention period for a retained entry elapses after account deletion, when the period ends, then the entry is removed with nothing further retained.

**FEAT-13.SPEC-006-AC-12:** Given Nadia looks for a way to schedule or configure automatic time-based cleanup of her activity trail, when she searches account or trail settings, then no such capability exists anywhere in the product.

**FEAT-13.SPEC-006-AC-13:** Given a support session entry (opened/closed) exists with no financial-record character, when the account is deleted, then it is removed with the rest of the non-financial entries.

**FEAT-13.SPEC-006-AC-14:** Given an entry references a record (e.g., a deliverable) that some other feature's own rules later remove independently of account deletion, when that other record is removed, then the activity entry referencing it is unaffected and remains exactly as written.

**FEAT-13.SPEC-006-AC-15:** Given Nadia's account deletion is in progress, when she looks for any way to selectively preserve or exclude specific entries from the purge, then no such control exists -- the purge (subject to the legal-retention carve-out) applies uniformly.

**FEAT-13.SPEC-006-AC-16:** Given the freelancer's account has already been deleted and its legal-retention window has not yet elapsed, when any role attempts to view a retained entry through the product, then no screen or access path exists, since the account itself no longer exists.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 1 (N/A, no interaction applies) | 1 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 6 | 6 |
| Edge Cases | 5 | 5 |
