---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-18.SPEC-006
spec_name: Primary Contact Requirement Rule
spec_slug: primary-contact-requirement-rule
parent_feature: FEAT-18
parent_feature_name: Client Contact Management & Roles
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 7
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Primary Contact Requirement Rule

## Overview

**Name:** Primary Contact Requirement Rule
**ID:** FEAT-18.SPEC-006
**Type:** Logic/Rule
**Purpose:** Enforces that a client must have at least one Primary contact before a proposal can be sent, and blocks removing or demoting the last Primary contact until a replacement is designated.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles
**Governed Entity:** Client Contact (specifically, the count and state of Primary-role contacts per Client)

## Scope and Non-Goals

**In Scope:**
- The rule that a client needs at least one Primary contact for a proposal to be sendable (XBR-07)
- The block on removing the client's last Primary contact
- The block on changing the client's last Primary contact's role to Reviewer
- The commit-time re-check that applies both blocks at the exact moment of the removal or role-change action, not only at screen load

**Non-Goals:**
- Field-level validation of name, email, and role values themselves -- owned by FEAT-18.SPEC-005 (Contact Field Validation Rules); this spec governs only the count-and-state condition across a client's Primary contacts
- Who may remove or change a contact's role at all -- owned by FEAT-18.SPEC-007 (Role Authorization Rules); this spec governs the condition under which an otherwise-permitted removal or role change is blocked
- The actual proposal-send blocking screen and message on the Proposal Creation & Sending side -- owned by FEAT-02 (per XBR-07); this spec is the authority this feature exposes for that check, not the sending feature's own UI
- Automatically promoting any existing Reviewer to Primary when the last Primary is removed -- excluded per BRIEF.md's Target Users & Roles, which frames role assignment as a deliberate freelancer or Primary-contact decision; the product never silently reassigns entitlements

## Governed Entity

**Entity:** Client Contact
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| role | enum (Primary, Reviewer) | The contact's entitlement tier; this rule counts contacts where role = Primary and status is not Removed, per Client |
| status | enum (Invited, Active, Removed) | Only Invited and Active contacts count toward the Primary requirement; a Removed contact is never counted |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-18.SPEC-002 | Add or Edit Client Contact | On save, when an edit would change an existing Primary contact's role to Reviewer |
| FEAT-18.SPEC-003 | Remove Client Contact | On screen load (to decide standard vs. blocked case) and again on commit, when Nadia confirms removal of a Primary contact |
| FEAT-02 (Proposal Creation & Sending) | Proposal send action (cross-feature, XBR-07) | At the moment Nadia or Owen attempts to send or rely on a proposal for a client |
| FEAT-03 (Proposal Acceptance) | Acceptance flow entitlement check (cross-feature, XBR-07) | Confirms a Primary contact exists and is the one accepting |

## Field Validation Rules

This spec governs an entity-count condition, not a per-field format rule. The single field this spec addresses is role, only in its aggregate effect across a client's contacts, not its individual format -- that is FEAT-18.SPEC-005's responsibility.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| role (aggregate, per Client) | At least one non-Removed contact with role = Primary must exist before a proposal can be sent for that client | Evaluated per Client, not per contact | At proposal-send attempt (FEAT-02, XBR-07) | "This client has no Primary contact. Add one before sending a proposal." | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Last-Primary removal block | role, status (across all of a Client's contacts) | A removal that would reduce the count of non-Removed Primary contacts for a Client to zero is blocked | "{contact_name} is this client's only Primary contact. Add or promote a replacement Primary before removing them." (FEAT-18.SPEC-003) |
| Last-Primary demotion block | role, status (across all of a Client's contacts) | An edit that would change the client's only non-Removed Primary contact's role to Reviewer is blocked | "This is the client's only Primary contact. Add or promote another Primary before changing this one." (FEAT-18.SPEC-002) |

## Authorization Rules

Who may attempt a removal or role change is governed by FEAT-18.SPEC-007; this table addresses only the last-Primary condition's interaction with each role's entitlement.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Remove the client's last Primary contact | Nadia (Freelancer) | Never, until a replacement Primary exists | Blocking message on FEAT-18.SPEC-003: "{contact_name} is this client's only Primary contact. Add or promote a replacement Primary before removing them." |
| Demote the client's last Primary contact to Reviewer | Nadia (Freelancer) | Never, until a replacement Primary exists | Blocked on save (FEAT-18.SPEC-002): "This is the client's only Primary contact. Add or promote another Primary before changing this one." |
| Remove or demote a Primary contact when at least one other Primary contact exists for the client | Nadia (Freelancer) | Always | -- |
| Remove or demote any contact | Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Never -- neither role has a remove or edit entitlement anywhere in this feature (FEAT-18.SPEC-007) | This rule's blocks are moot for these roles, since they have no route to attempt the underlying action at all |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Primary-contact count (derived) | Count of Client Contact records for a given Client where role = Primary and status is not Removed | Recomputed at every enforcement point listed above | No -- always derived at the moment of the check, never cached across the check |

## Business Rules

- XBR-07: a proposal cannot be sent until the client has at least one Primary contact, and the last Primary contact cannot be removed until a replacement is designated. This spec is the owning authority for that rule (feature-dependency-map.md, XBR-07 Authority: FEAT-18).
- The last-Primary block is re-checked at the moment of commit, not only when the removal or edit screen first loads, since another session's action can change the client's Primary count in between (dependency map, Client Contact Contention note).
- Demoting the last Primary to Reviewer is blocked by the identical logic as removing the last Primary, since both actions produce the same forbidden outcome: a client with zero Primary contacts.
- This rule has no effect on a client that already has two or more Primary contacts (e.g., co-founders both designated Primary); removing or demoting one of several Primaries is never blocked by this rule.

## Edge Cases

- **A client has exactly one Primary and one Reviewer contact; Nadia adds a second Primary, then removes the original Primary** -- The removal is permitted: at the moment of commit, one non-Removed Primary contact (the newly added one) still exists for the client.
- **Nadia attempts to remove the last Primary and, in the same moment from another session, promotes a Reviewer to Primary first** -- If the promotion's save completes before the removal's commit-time check runs, the removal proceeds normally (the count is no longer zero at commit); if the removal's check runs first, it is blocked and Nadia must retry after the promotion completes.
- **A client's only Primary contact is removed via FEAT-24 account deletion cascading through the freelancer's own account closure** -- Not blocked by this rule: account deletion (FEAT-24) removes all of a freelancer's data as a unit, and the Primary-requirement rule governs only in-product removal and demotion actions Nadia takes while her account remains active.
- **A proposal is already Sent or Accepted when the client's sole Primary contact is later removed** -- Not retroactively affected: the acceptance record (if any) remains valid and immutable under the removed contact's name as evidence (XBR-27); this rule only blocks a *future* proposal send or the removal itself when it would leave the client with zero Primary contacts and a pending or future send.
- **Nadia attempts to remove the client's last Primary contact while that same client has no other contacts at all** -- Blocked identically to the standard last-Primary case; the message is unchanged regardless of whether other Reviewer contacts exist.
- **The commit-time re-check runs but the underlying contact was concurrently removed by a different action entirely (e.g., a duplicate rapid submission)** -- The removal-in-progress finds the contact already Removed and treats the action as already complete, showing the standard success outcome rather than a stale-record error, since the end state (this contact removed) matches what was requested.

## Acceptance Criteria

**FEAT-18.SPEC-006-AC-01:** Given a client has exactly one Primary contact, when Nadia attempts to remove that contact on FEAT-18.SPEC-003, then the removal is blocked with "{contact_name} is this client's only Primary contact. Add or promote a replacement Primary before removing them."

**FEAT-18.SPEC-006-AC-02:** Given a client has two Primary contacts, when Nadia removes one of them, then the removal proceeds, since one Primary contact still remains.

**FEAT-18.SPEC-006-AC-03:** Given a client has exactly one Primary contact, when Nadia edits that contact on FEAT-18.SPEC-002 and changes the role to Reviewer, then the save is blocked with "This is the client's only Primary contact. Add or promote another Primary before changing this one."

**FEAT-18.SPEC-006-AC-04:** Given a client has zero Primary contacts, when Nadia or Owen attempts to send a proposal for that client (FEAT-02), then the send is blocked with "This client has no Primary contact. Add one before sending a proposal."

**FEAT-18.SPEC-006-AC-05:** Given a client has zero contacts at all, when Nadia adds the first contact and assigns Primary, then the proposal-send block for that client is lifted immediately.

**FEAT-18.SPEC-006-AC-06:** Given Nadia has the removal confirmation for the client's last Primary open, when another Primary contact is added to that client in a different session before she confirms, then her commit-time re-check finds a Primary contact still exists and the removal proceeds.

**FEAT-18.SPEC-006-AC-07:** Given Nadia has the removal confirmation for the client's last Primary open, when no other Primary is added before she confirms, then the commit-time re-check still finds zero other Primary contacts and the removal remains blocked.

**FEAT-18.SPEC-006-AC-08:** Given a proposal was already accepted under a Primary contact who is later removed as the client's last Primary at that time, when the removal is evaluated, then the prior acceptance record is unaffected and remains valid evidence under that contact's name.

**FEAT-18.SPEC-006-AC-09:** Given Owen or Priya has no remove or role-change entitlement on any contact, when either looks for a way to trigger this rule's block, then no such control exists for them anywhere in the product.

**FEAT-18.SPEC-006-AC-10:** Given a client has one Primary and several Reviewer contacts, when Nadia attempts to remove the sole Primary, then the block message is identical regardless of how many Reviewer contacts also exist.

**FEAT-18.SPEC-006-AC-11:** Given Nadia's account is being deleted via FEAT-24, when the account-deletion process removes the client's sole Primary contact as part of removing the whole account, then this rule does not block that removal, since FEAT-24 removes the freelancer's data as a unit rather than performing an in-product contact removal.

**FEAT-18.SPEC-006-AC-12:** Given a rapid duplicate removal submission targets a contact that a first submission already removed, when the second submission's commit-time check runs, then it finds the contact already Removed and treats the action as already complete rather than surfacing a stale-record error.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
