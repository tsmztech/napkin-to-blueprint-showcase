---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-18.SPEC-005
spec_name: Contact Field Validation Rules
spec_slug: contact-field-validation-rules
parent_feature: FEAT-18
parent_feature_name: Client Contact Management & Roles
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 18
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Contact Field Validation Rules

## Overview

**Name:** Contact Field Validation Rules
**ID:** FEAT-18.SPEC-005
**Type:** Logic/Rule
**Purpose:** Defines all field validation, cross-field, and role-assignment-scope rules enforced on every contact add, edit, or invite across the feature.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles
**Governed Entity:** Client Contact

## Scope and Non-Goals

**In Scope:**
- Per-field validation rules for name, email, and role on the Client Contact record
- The per-client-company email uniqueness rule
- The rule restricting Owen's assignable role to Reviewer only, versus Nadia's unrestricted assignment
- Default and derived values set on creation (invited_by, status)
- Exact error messages for every validation failure

**Non-Goals:**
- Which roles may create, edit, or remove a contact at all -- owned by FEAT-18.SPEC-007 (Role Authorization Rules); this spec governs the data entered once an add/edit/invite action is already permitted
- The last-Primary block on removal or role change -- owned by FEAT-18.SPEC-006 (Primary Contact Requirement Rule); this spec's per-field rules do not duplicate that entity-lifecycle constraint
- Whether a role change applies immediately or only to future actions -- owned by FEAT-18.SPEC-008 (Role Change Effective-Timing Rule); this spec validates that a role value is one of the two allowed values, not when it takes effect
- Duplicate-person detection beyond email uniqueness (e.g., matching by name alone) -- product-features.md defines no such capability; email uniqueness per client company is the sole identity check this product performs on Client Contact

## Governed Entity

**Entity:** Client Contact
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | Contact's name |
| email | text | Sign-in and notification address, unique within the client company |
| role | enum (Primary, Reviewer) | The contact's entitlement tier |
| invited_by | text (reference) | Nadia or the inviting Primary contact who created this record |
| status | enum (Invited, Active, Removed) | The contact's lifecycle state |
| last_sign_in | date/time | Login timestamps captured by FEAT-05 |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-18.SPEC-002 | Add or Edit Client Contact | On field blur and form submit |
| FEAT-18.SPEC-004 | Invite Reviewer Colleague | On field blur and form submit; role is fixed to Reviewer before validation runs |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | Required, non-empty | Always | On blur | "Name is required" | Yes |
| name | No validation beyond data type once non-empty | Always | -- | -- | -- |
| email | Required, non-empty | Always | On blur | "Email is required" | Yes |
| email | Valid email format | When provided (not empty) | On blur | "Please enter a valid email address" | Yes |
| email | Unique within the client company | Always, evaluated against every contact currently at the same client, excluding Removed contacts (whose erased email no longer exists as a live value to collide with) | On blur and re-checked on submit | "This email is already used by another contact at this client" (Nadia's screen, FEAT-18.SPEC-002) / "This email is already used by another contact at this company" (Owen's screen, FEAT-18.SPEC-004) | Yes |
| role | Required; must be one of Primary or Reviewer | Always | On submit | "Please select a role" | Yes |
| role | On FEAT-18.SPEC-004 (Owen's invite screen), the value is fixed to Reviewer and never presented as a choice | Always, on the invite-screen enforcement point only | On submit (defensively, in case of a malformed request) | "Only a Reviewer role can be invited from this screen" | Yes |
| invited_by | No validation beyond data type -- set automatically, never entered by the user | Always | -- | -- | -- |
| status | No validation beyond data type -- set automatically on create (Invited) and by FEAT-05/FEAT-18.SPEC-009 thereafter, never entered by the user on this screen | Always | -- | -- | -- |
| last_sign_in | No validation beyond data type -- written only by FEAT-05, never entered here | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Owen's invite role restriction | role, invited_by | When invited_by resolves to a Primary contact acting through FEAT-18.SPEC-004 (rather than Nadia acting through FEAT-18.SPEC-002), role must equal Reviewer | "Only a Reviewer role can be invited from this screen" |
| Own-company scoping | email, invited_by (via the acting Primary contact's own client company) | When the acting party is a Primary contact (Owen), the new contact is always created at that Primary contact's own client company; the email-uniqueness check runs against that same company only | N/A -- this is a scoping rule with no user-facing violation state, since FEAT-18.SPEC-004 never exposes a client-selection control that could produce a mismatch |

## Authorization Rules

Role-based authorization for contact actions themselves is the responsibility of FEAT-18.SPEC-007 (Role Authorization Rules) -- see that spec for the full action-by-role matrix. This spec's Authorization Rules table covers only the one authorization boundary specific to field entry: who may set which value in the role field.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Set role to Primary | Nadia (Freelancer) | Always, on any contact she creates or edits | -- |
| Set role to Primary | Owen (Client Primary Contact) | Never | The role field on FEAT-18.SPEC-004 is a fixed "Reviewer" label -- no control exists through which Owen could attempt this |
| Set role to Reviewer | Nadia (Freelancer) | Always, on any contact she creates or edits | -- |
| Set role to Reviewer | Owen (Client Primary Contact) | Always, only for a new contact at his own client company | -- |
| Set role to any value | Priya (Client Reviewer Contact) | Never | Priya has no route to any contact create/edit/invite screen (Access Matrix: Client Contact Management -- None) |
| Set role to any value | Dana (Support Operator) | Never | Dana's read-only support session exposes no create/edit/invite screen (FEAT-31.SPEC-005) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| invited_by | The acting party: Nadia's own identity on FEAT-18.SPEC-002, or the inviting Primary contact's identity on FEAT-18.SPEC-004 | On create only | No -- always derived from who performed the create action |
| status | "Invited" | On create only | No |
| last_sign_in | Unset until FEAT-05 records the contact's first successful magic-link sign-in | On create only | No -- never entered on this feature's screens |

## Business Rules

- Field validation runs before any role-authorization check on the action itself completes visibly to the user -- an invalid field state is shown regardless of whether the underlying action would otherwise be allowed, so the user always sees what to fix first.
- Email uniqueness is scoped per client company, not per freelancer account or globally: the same email address may be a Client Contact for two different client companies of the same freelancer, or for two different freelancers entirely (dependency map, Client Contact -- Relationships: "A person who is a contact for several freelancers holds a separate Client Contact per freelancer").
- A Removed contact's email is erased (FEAT-18.SPEC-009) and therefore never collides with the uniqueness check -- re-adding the same person as a new contact after removal is always possible and always creates a fresh record (feature-overview.md, Non-Goals -- no restore path).
- These rules apply identically on FEAT-18.SPEC-002 and FEAT-18.SPEC-004 wherever the field itself is common (name, email); FEAT-18.SPEC-004 additionally fixes role, per the Cross-Field Rules above.

## Edge Cases

- **Email entered with mixed case (e.g., "Owen@Acme.test" vs. an existing "owen@acme.test")** -- Uniqueness comparison is case-insensitive; the second entry is rejected as a duplicate.
- **Email entered with leading or trailing whitespace** -- Whitespace is trimmed before format and uniqueness checks run; "  owen@acme.test " validates identically to "owen@acme.test".
- **Name field contains only whitespace** -- Treated as empty; "Name is required" is shown.
- **Nadia edits a contact's email to match another live contact at the same client** -- Rejected on blur and again on submit with "This email is already used by another contact at this client"; the edit does not save.
- **Two adds targeting the same new email at the same client complete at effectively the same time (Nadia via FEAT-18.SPEC-002, Owen via FEAT-18.SPEC-004)** -- Per the dependency map's Contention note, the second save to commit is rejected-with-refresh on the uniqueness check, regardless of which screen or role submitted first.
- **A malformed request reaches this rule set with role set to Primary from the Owen-invite enforcement point** -- Blocked with "Only a Reviewer role can be invited from this screen"; this is a defensive check, since FEAT-18.SPEC-004's interface never presents a role choice to Owen in the first place.

## Acceptance Criteria

**FEAT-18.SPEC-005-AC-01:** Given Nadia is adding a contact and leaves the name field empty, when she blurs the field, then it shows "Name is required."

**FEAT-18.SPEC-005-AC-02:** Given Nadia enters "not-an-email" in the email field, when she blurs the field, then it shows "Please enter a valid email address."

**FEAT-18.SPEC-005-AC-03:** Given Nadia enters an email that already belongs to another live contact at the same client, when she blurs the email field, then it shows "This email is already used by another contact at this client."

**FEAT-18.SPEC-005-AC-04:** Given Nadia enters a valid, unique name and email and selects a role, when she submits, then all field validation passes.

**FEAT-18.SPEC-005-AC-05:** Given Nadia does not select a role, when she submits the form, then she sees "Please select a role" and the save does not proceed.

**FEAT-18.SPEC-005-AC-06:** Given Owen is inviting a colleague, when he views the role field, then no selectable control is shown -- it is a fixed "Reviewer" label, and validation never presents an error for it under normal use.

**FEAT-18.SPEC-005-AC-07:** Given a request reaches this rule set attempting to set role to Primary from Owen's invite enforcement point, when validation runs, then it is blocked with "Only a Reviewer role can be invited from this screen."

**FEAT-18.SPEC-005-AC-08:** Given the same email exists for two different client companies of the same freelancer, when Nadia adds a contact with that email to a third, unrelated client, then no uniqueness violation occurs, since the check is scoped per client company.

**FEAT-18.SPEC-005-AC-09:** Given a contact with a given email was previously removed (and their email erased), when Nadia adds a new contact with that same email at the same client, then no uniqueness violation occurs.

**FEAT-18.SPEC-005-AC-10:** Given Nadia enters "Owen@Acme.test" while "owen@acme.test" already exists at the same client, when she blurs the field, then it is rejected as a duplicate regardless of letter case.

**FEAT-18.SPEC-005-AC-11:** Given Nadia enters an email with leading whitespace, when the field is validated, then the whitespace is trimmed before format and uniqueness checks run.

**FEAT-18.SPEC-005-AC-12:** Given Nadia edits an existing contact's email to match another live contact at the same client, when she attempts to save, then the save is rejected with the duplicate-email error and no change is persisted.

**FEAT-18.SPEC-005-AC-13:** Given Owen and Nadia each submit a new contact with the same new email for the same client at effectively the same time, when the second submission is processed, then it is rejected-with-refresh on the uniqueness check.

**FEAT-18.SPEC-005-AC-14:** Given a new contact is successfully created by Nadia, when the record is saved, then invited_by is set to Nadia's identity and status is set to Invited, with neither field ever presented to Nadia as an entry field.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 10 | 10 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
