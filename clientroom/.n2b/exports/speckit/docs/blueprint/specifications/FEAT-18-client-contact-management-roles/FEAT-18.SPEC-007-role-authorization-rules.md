---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-18.SPEC-007
spec_name: Role Authorization Rules
spec_slug: role-authorization-rules
parent_feature: FEAT-18
parent_feature_name: Client Contact Management & Roles
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 20
acceptance_criteria_count: 19
---

# Logic/Rule Spec: Role Authorization Rules

## Overview

**Name:** Role Authorization Rules
**ID:** FEAT-18.SPEC-007
**Type:** Logic/Rule
**Purpose:** Enforces every Client Contact action's entitlement by role -- Nadia's full management, Owen's own-company invite-only entitlement, Priya's total exclusion, and Dana's read-only view -- and states exactly what each denied role experiences.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles
**Governed Entity:** Client Contact

## Scope and Non-Goals

**In Scope:**
- The complete action-by-role authorization matrix for every action this feature defines on the Client Contact entity (create, view/list, edit, change role, remove, invite)
- The exact denied experience for every role that cannot perform a given action
- Owen's own-company scoping and Reviewer-only invite restriction as an authorization boundary (the field-level mechanics of that restriction are FEAT-18.SPEC-005's responsibility; this spec is the authority for *why* Owen may act at all)
- Dana's view-only entitlement inside a logged support session

**Non-Goals:**
- Field-level validation of the values entered in an allowed action -- owned by FEAT-18.SPEC-005 (Contact Field Validation Rules)
- The last-Primary block on removal and demotion -- owned by FEAT-18.SPEC-006 (Primary Contact Requirement Rule); this spec establishes *who* may attempt removal or a role change, not the additional entity-state condition that can still block an otherwise-authorized Nadia action
- The future-only effective timing of a role change -- owned by FEAT-18.SPEC-008 (Role Change Effective-Timing Rule); this spec authorizes *who* may change a role, not *when* the change takes effect
- Authorization for actions outside this feature that merely *depend on* a contact's role (accepting a proposal, approving a milestone, paying an invoice) -- each is owned by its own feature (FEAT-03, FEAT-08, FEAT-10) and referenced here by XBR-08 rather than restated

## Governed Entity

**Entity:** Client Contact
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | Contact's name |
| email | text | Sign-in and notification address |
| role | enum (Primary, Reviewer) | The contact's entitlement tier; also the axis this spec's Own-only and Reviewer-only rules key on |
| invited_by | text (reference) | Nadia or the inviting Primary contact; determines Owen's own-company scope |
| status | enum (Invited, Active, Removed) | The contact's lifecycle state; a Removed contact is never shown to any role |
| last_sign_in | date/time | Read-only display field; not itself an authorization axis |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-18.SPEC-001 | Client Contact List | On screen entry (which roles can reach it at all) and on each row action's visibility |
| FEAT-18.SPEC-002 | Add or Edit Client Contact | On screen entry; only Nadia ever reaches this screen |
| FEAT-18.SPEC-003 | Remove Client Contact | On screen entry; only Nadia ever reaches this screen |
| FEAT-18.SPEC-004 | Invite Reviewer Colleague | On screen entry (Owen only) and on the fixed-Reviewer role restriction |
| FEAT-05 (Client Portal Access) | Portal Home's role-scoped rendering | Determines whether the "Invite a colleague" action appears (Owen only) |

## Field Validation Rules

This spec governs action-level authorization, not field format. No field validation rules apply here beyond what FEAT-18.SPEC-005 already defines; every field in the governed entity is addressed there.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| (all fields) | No validation beyond data type at this spec's layer -- see FEAT-18.SPEC-005 for field-level rules | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Own-company scope | invited_by, the client each contact belongs to | When the acting party is a Primary contact, every read and write this spec authorizes is scoped to contacts at that Primary contact's own client company only -- never another company's contacts | N/A -- enforced by simply never surfacing another company's contacts or invite target to Owen, not by a rejectable form value |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create a contact (any role) | Nadia (Freelancer) | Always, for any client she owns | -- |
| Create a contact (Reviewer role only) | Owen (Client Primary Contact) | Own-only -- only at Owen's own client company | -- |
| Create a contact | Priya (Client Reviewer Contact) | Never | "Invite a colleague" and any add/edit surface are not shown anywhere in Priya's portal; a direct attempt has no reachable control to trigger |
| Create a contact | Dana (Support Operator) | Never | No create control is rendered inside a support session (FEAT-31.SPEC-005, unconditional read-only) |
| View the full contact list (FEAT-18.SPEC-001) | Nadia (Freelancer) | Always, for any client she owns | -- |
| View the full contact list (FEAT-18.SPEC-001) | Owen (Client Primary Contact) | Never -- Owen's own-company contact context appears only inline on FEAT-18.SPEC-004, not as the freelancer-side list | FEAT-18.SPEC-001 is not part of Owen's portal navigation |
| View the full contact list (FEAT-18.SPEC-001) | Priya (Client Reviewer Contact) | Never | FEAT-18.SPEC-001 is not part of Priya's portal navigation |
| View the full contact list, read-only (FEAT-18.SPEC-001) | Dana (Support Operator) | Always, inside a logged support session, for the one named freelancer account only | -- |
| View own-company contacts (inline context on FEAT-18.SPEC-004) | Owen (Client Primary Contact) | Always, scoped to his own client company | -- |
| View own-company contacts | Priya (Client Reviewer Contact) | Never | Priya has no contact-management surface, per the Access Matrix (Client Contact Management: None) |
| Edit an existing contact's name, email, or role | Nadia (Freelancer) | Always, for any contact at a client she owns | -- |
| Edit an existing contact's name, email, or role | Owen (Client Primary Contact) | Never -- including his own contact record | Owen has no edit control anywhere in this feature; a direct attempt has no reachable path |
| Edit an existing contact's name, email, or role | Priya (Client Reviewer Contact), Dana (Support Operator) | Never | No edit control is shown to either role |
| Remove a contact | Nadia (Freelancer) | Always, subject to FEAT-18.SPEC-006's last-Primary block | -- |
| Remove a contact | Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Never | No remove control is shown to any of these roles |
| Invite a Reviewer colleague at one's own client company | Owen (Client Primary Contact) | Always | -- |
| Invite a Reviewer colleague | Priya (Client Reviewer Contact) | Never | "Invite a colleague" is not shown on Priya's Portal Home |
| Invite a contact at a different client company | Owen (Client Primary Contact) | Never | No invite target other than his own company is ever presented; there is no cross-company selector to attempt |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Acting party's authorized scope (derived) | Nadia: every client she owns. Owen: his own client company only, derived from his own Client Contact record's client relationship. Priya: none. Dana: the one freelancer account named by her open Support Access Session. | Evaluated at every screen entry and every action attempt | No -- always derived from the acting party's own identity and, for Owen, his own contact record; never a value any role can set for themselves |

## Business Rules

- XBR-08: role entitlements follow the Access Matrix everywhere -- only Primary contacts accept proposals, request changes, approve milestones, and see, pay, and download invoices; Reviewer contacts view and comment only. This spec is the owning authority for the Primary-versus-Reviewer distinction itself (feature-dependency-map.md, XBR-08 Authority: FEAT-18); the features that consume role for their own entitlement checks (FEAT-03, FEAT-07, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-14, FEAT-25, FEAT-26) reference this spec's role assignment rather than re-deriving it.
- Owen's invite entitlement is deliberately narrower than his own portal access elsewhere: he may add a colleague but may never edit or remove any contact, including his own record, or see the freelancer-side contact list (FEAT-18.SPEC-001).
- Dana's view-only access exists only for the duration of an open, logged support session on one named freelancer account (FEAT-31); it is never a standing entitlement and is never available outside that session.
- Priya's total exclusion from Client Contact Management is a deliberate product decision, not an oversight: the Access Matrix (user-persona.md) states her Client Contact Management entitlement as None across the board.
- A contact's status (Invited, Active, Removed) does not itself change what a role may do -- authorization is keyed on the acting party's own role and relationship to the client, not on the target contact's status, except that a Removed contact is never shown to any role (this spec's read rules apply only to non-Removed contacts).

## Edge Cases

- **Owen attempts to reach FEAT-18.SPEC-001 (Nadia's freelancer-side list) by a guessed or old link** -- No such route exists in the client portal's own navigation structure for Owen's role; the attempt has no screen to land on within this feature's scope, and FEAT-05's own isolation rules (XBR-09, FEAT-05.SPEC-007) prevent any client-side session from reaching a freelancer-side screen at all.
- **Dana's support session closes while she is viewing the read-only contact list** -- She is returned to the Operator Support Session Console queue (FEAT-31.SPEC-002); her prior view-only access to that freelancer's contacts ends immediately with the session.
- **Owen's own client company later gains a second Primary contact he did not invite** -- His invite entitlement is unaffected; he may still invite Reviewer colleagues at his own company regardless of how many Primary contacts that company has.
- **Priya is later promoted to Primary by Nadia** -- From that moment, she is authorized under this spec's Primary-contact rows going forward (e.g., she may then invite Reviewer colleagues); this is a role change, not an authorization-rule exception, and takes effect only for future actions per FEAT-18.SPEC-008.
- **A support session is open on a freelancer account and Nadia is also actively editing a contact in her own session at the same moment** -- No conflict at the authorization layer: Dana's session grants read-only access that never writes, so it cannot collide with Nadia's concurrent edit; any data staleness on Dana's read is a display concern, not an authorization one.
- **Owen's contact record is itself later removed by Nadia while Owen has an invite in progress** -- Since removal ends access immediately (FEAT-18.SPEC-009), Owen's in-progress session is terminated by FEAT-05's access rules before this spec's invite authorization is ever reached again; any invite he already completed before removal remains valid and unaffected.

## Acceptance Criteria

**FEAT-18.SPEC-007-AC-01:** Given Nadia owns a client, when she opens FEAT-18.SPEC-001 for that client, then she sees the full contact list with all management controls.

**FEAT-18.SPEC-007-AC-02:** Given Owen is signed in to his client portal, when he looks for a route to FEAT-18.SPEC-001, then none exists.

**FEAT-18.SPEC-007-AC-03:** Given Dana opens a support session on Nadia's account, when she navigates to a client's contacts, then she sees the full list read-only, with no create, edit, or remove control rendered.

**FEAT-18.SPEC-007-AC-04:** Given Owen is on FEAT-18.SPEC-004, when he submits a new contact, then it is created as a Reviewer at his own client company.

**FEAT-18.SPEC-007-AC-05:** Given Owen has no edit entitlement, when he looks for a way to change his own contact record's name, email, or role, then no such control exists anywhere in the client portal.

**FEAT-18.SPEC-007-AC-06:** Given Priya is signed in to her client portal, when she looks for any contact-management action (add, edit, remove, invite), then none is shown to her anywhere.

**FEAT-18.SPEC-007-AC-07:** Given Nadia attempts to remove a contact, when the client has more than one Primary contact or the target is a Reviewer, then the removal proceeds (subject to FEAT-18.SPEC-006's last-Primary check).

**FEAT-18.SPEC-007-AC-08:** Given Owen, Priya, or Dana attempts to remove any contact, when they look for a remove control, then none exists for any of the three roles.

**FEAT-18.SPEC-007-AC-09:** Given Owen views his own company's contact list inline on FEAT-18.SPEC-004, when the list renders, then it shows only contacts at his own client company and never another company's contacts.

**FEAT-18.SPEC-007-AC-10:** Given Dana's support session on Nadia's account closes, when she is returned to the Operator Support Session Console, then her prior read-only access to that account's contacts ends immediately.

**FEAT-18.SPEC-007-AC-11:** Given Nadia is the sole freelancer-side actor with contact-management access, when she views any client's contacts, then no other freelancer-side role or seat exists to share or restrict that access with (Access Matrix: Nadia -- Full; no other internal role, per SC-01).

**FEAT-18.SPEC-007-AC-12:** Given Priya is later promoted to Primary by Nadia, when the change takes effect, then Priya becomes authorized to invite Reviewer colleagues at her own company from that point forward, per FEAT-18.SPEC-008's future-only timing.

**FEAT-18.SPEC-007-AC-13:** Given Owen's own client company already has two Primary contacts, when Owen invites a new Reviewer colleague, then the invite proceeds normally, unaffected by how many Primary contacts exist.

**FEAT-18.SPEC-007-AC-14:** Given a contact's status is Removed, when any role's screen would otherwise list it, then it never appears, for any role including Dana's read-only view.

**FEAT-18.SPEC-007-AC-15:** Given Nadia attempts to create a contact for a client she owns, when she assigns either Primary or Reviewer, then the create is authorized for both role values.

**FEAT-18.SPEC-007-AC-16:** Given Owen attempts to assign a role other than Reviewer on FEAT-18.SPEC-004, when the attempt reaches this rule set, then it is denied, since no such control is ever presented and the field-level restriction (FEAT-18.SPEC-005) additionally blocks it.

**FEAT-18.SPEC-007-AC-17:** Given Dana is not inside an open support session, when she attempts to view any freelancer's contacts, then no access is granted -- the read-only view exists only for the duration of an open session (FEAT-31).

**FEAT-18.SPEC-007-AC-18:** Given Nadia deletes her account via FEAT-24, when the deletion completes, then no role retains any access to that account's former Client Contact records, since the account and its data no longer exist.

**FEAT-18.SPEC-007-AC-19:** Given Owen's own contact record is removed by Nadia while he has an in-progress invite screen open, when FEAT-18.SPEC-009 revokes his access, then his session is ended by FEAT-05's access rules and any invite he had already completed before the removal remains valid.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 18 | 18 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
