---
document_type: feature-overview
feature_number: FEAT-18
feature_name: Client Contact Management & Roles
feature_slug: client-contact-management-roles
priority_tier: Important
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 11
screen_count: 4
automation_count: 1
logic_rule_count: 4
integration_count: 0
notification_count: 2
---

# Feature Breakdown Brief: Client Contact Management & Roles

## Summary

**Feature:** Client Contact Management & Roles
**ID:** FEAT-18
**Description:** The freelancer adds contacts to a client company and assigns each one the Primary or Reviewer role; a Primary contact can invite additional colleagues at their own company.
**Priority:** Important
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md, Target Users & Roles: "not every contact may approve work or see invoices" with the founder's leaning toward Primary (accept/approve/pay) versus Reviewer (comment-only) contacts. Ranked Important rather than Core because the underlying access differentiation is what makes every Core feature's Access field correct, but the management surface itself is a supporting capability. Phased MVP because the role distinction must exist from the first client onward for the Access Matrix to hold. [RESEARCH-INFORMED: none of the 5 profiled products offers a distinct primary-versus-comment-only client contact split, and all-or-nothing permissions are a documented Dubsado complaint (MEDIUM); client-side demand is inferred (LOW), so the feature stays Important and follows the founder's leaning in BRIEF.md] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Add a contact — freelancer records a new person at a client company
- Assign a role — Primary or Reviewer, per BRIEF.md's leaning
- Primary-led invites — a Primary contact can invite Reviewer colleagues at their own company
- Remove a contact — revoke a contact's access immediately, for example when they leave the client company or ask for their personal data to be erased [AUDIT-ADDED: 4 -- compliance: a client contact's data-subject request needed a path]

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-18.SPEC-001 | Client Contact List | Screen | Nadia (Freelancer), Dana (Support Operator) | Nadia views and manages a client's contacts and roles; Dana views the same list read-only inside a logged support session |
| FEAT-18.SPEC-002 | Add or Edit Client Contact | Screen | Nadia (Freelancer) | Nadia adds a new contact or edits an existing contact's name, email, or role |
| FEAT-18.SPEC-003 | Remove Client Contact | Screen | Nadia (Freelancer) | Nadia confirms removing a contact, including the last-Primary block and erasure-request framing |
| FEAT-18.SPEC-004 | Invite Reviewer Colleague | Screen | Owen (Client Primary Contact) | Owen invites a Reviewer colleague at his own client company from his portal view |
| FEAT-18.SPEC-005 | Contact Field Validation Rules | Logic/Rule | Nadia (Freelancer), Owen (Client Primary Contact) | Enforces required fields, format, and per-client-company email uniqueness on every contact add/edit/invite |
| FEAT-18.SPEC-006 | Primary Contact Requirement Rule | Logic/Rule | Nadia (Freelancer), Owen (Client Primary Contact) | Enforces that a client must have at least one Primary contact before a proposal can be sent, and blocks removing the last Primary until a replacement is designated |
| FEAT-18.SPEC-007 | Role Authorization Rules | Logic/Rule | All | Enforces Primary-versus-Reviewer entitlements, Owen's own-company invite scope, Dana's view-only access, and Priya's lack of access everywhere behavior differs by role |
| FEAT-18.SPEC-008 | Role Change Effective-Timing Rule | Logic/Rule | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Ensures a role change applies to future actions only and never alters the record of a past approval or acceptance |
| FEAT-18.SPEC-009 | Contact Removal & Data Erasure | Automation | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Ends a removed contact's access immediately, erases their personal contact details, and retains their evidentiary records under their name |
| FEAT-18.SPEC-010 | New Contact Invitation Email | Notification | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Sends a newly added or invited contact their first magic-link sign-in invitation |
| FEAT-18.SPEC-011 | Primary-Invited-Colleague Alert | Notification | Nadia (Freelancer) | Alerts Nadia by email whenever a Primary contact invites a colleague, so she always knows who can see her work |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Add a contact | FEAT-18.SPEC-002 | Primary purpose of the add form; creates the Client Contact record | Phase 2 (Explicit) |
| Assign a role | FEAT-18.SPEC-002, FEAT-18.SPEC-007 | Role field on the add/edit form; entitlements enforced everywhere by the Role Authorization Rules | Phase 2 (Explicit) |
| Primary-led invites | FEAT-18.SPEC-004 | Primary purpose of Owen's invite screen | Phase 2 (Explicit) |
| Remove a contact | FEAT-18.SPEC-003, FEAT-18.SPEC-009 | Confirmation screen dispositions the removal; the Automation executes access revocation and erasure | Phase 2 (Explicit) / Phase 4 (Trigger-Response) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-18.SPEC-001 | Client Contact List | Phase 2 (Explicit) | Every management capability (add, role-assign, invite, remove) needs an entry-point surface from which the contact is selected or the add/remove action is launched; the feature description implies this list without naming it directly |
| FEAT-18.SPEC-005 | Contact Field Validation Rules | Phase 5 (Rule-Constraint Discovery) | Five distinct validation rules (name required, email required and formatted, email unique per client company, role restricted to Primary/Reviewer, Owen's invite restricted to Reviewer-only) apply across two screens (SPEC-002, SPEC-004), crossing the standalone Logic/Rule threshold |
| FEAT-18.SPEC-006 | Primary Contact Requirement Rule | Phase 3 (Entity-Lifecycle Analysis) | The Delete/Archive cell of the Client Contact CRUD matrix surfaced a conditional block (cannot remove the last Primary) re-checked at commit; this same rule gates FEAT-02's proposal-send capability (XBR-07) |
| FEAT-18.SPEC-007 | Role Authorization Rules | Phase 5 (Rule-Constraint Discovery) | Authorization-rule analysis found entitlements that vary by role across every feature the Access Matrix touches (XBR-08); shared across multiple screens and automations, well past the standalone threshold |
| FEAT-18.SPEC-008 | Role Change Effective-Timing Rule | Phase 5 (Rule-Constraint Discovery) | Conditional/derivation rule: a role change's effect depends on the timing of the actions it governs (future-only), and interacts with the evidentiary-immutability rule elsewhere in the product |
| FEAT-18.SPEC-009 | Contact Removal & Data Erasure | Phase 4 (Trigger-Response Analysis) | Removal has cross-entity effects (access revocation, PII erasure, evidence retention) beyond a simple data write, crossing the standalone-Automation threshold |
| FEAT-18.SPEC-010 | New Contact Invitation Email | Phase 4 (Notification surfacing lens) | The Communications field names an invitation email with real delivery/audience rules (channel: email; audience: the new or invited contact; content: first magic-link) |
| FEAT-18.SPEC-011 | Primary-Invited-Colleague Alert | Phase 4 (Notification surfacing lens) | The Communications field names a second, distinct message (channel: email; audience: Nadia; content: which colleague was invited) |

## Entity-Lifecycle Coverage Matrix

**Entity: Client Contact**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-18.SPEC-002, FEAT-18.SPEC-004 | Nadia adds a contact directly (status: Invited); Owen invites a Reviewer colleague at his own company (status: Invited) | Both paths run through FEAT-18.SPEC-005 validation |
| Read (single) | FEAT-18.SPEC-002 | Loads an existing contact's details for editing | -- |
| Read (list) | FEAT-18.SPEC-001, FEAT-18.SPEC-004 | Nadia's full contact list per client; Owen's own-company-scoped list on the invite screen | Dana reads the same list read-only inside a FEAT-31 support session |
| Update | FEAT-18.SPEC-002 | Nadia edits name, email, or role on an existing contact (e.g., correcting a bouncing email per the FEAT-14 delivery-warning journey) | Role updates are governed by FEAT-18.SPEC-007 and FEAT-18.SPEC-008 |
| Delete/Archive | FEAT-18.SPEC-003, FEAT-18.SPEC-009 | Hard deletion of the contact's personal details (name, email) with immediate access revocation; status set to Removed rather than the row being physically dropped, so the contact remains referenceable as the actor on their past acceptances/approvals (record immutability, ASMP-25; retained permanently, SC-24). No restore path -- a removed contact must be re-added as a new contact, since the erasure guarantee (XBR-27) requires the original personal details to actually be gone. No cascade to Client or Project. Removing the last Primary contact is blocked until a replacement Primary is designated (FEAT-18.SPEC-006), re-checked at the moment of commit | Removal always ends access immediately, whether the trigger is a client-side departure or an explicit erasure request |
| State Transition | FEAT-18.SPEC-004 (Invited) / FEAT-05 (Active, on first magic-link sign-in, cross-feature) / FEAT-18.SPEC-009 (Removed) | status moves Invited -> Active -> Removed | FEAT-05 also updates last_sign_in on each subsequent login |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Client | FEAT-18.SPEC-001, FEAT-18.SPEC-002, FEAT-18.SPEC-004 | Scopes the contact list and add/invite forms to the correct client company |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia adds a new contact | Validate required fields, format, and per-client email uniqueness | Standalone Logic/Rule | SPEC-005 |
| Nadia adds a new contact | Send the new contact their first-sign-in invitation email | Standalone Notification | SPEC-010 |
| Owen invites a Reviewer colleague | Validate fields and restrict the assignable role to Reviewer only, at Owen's own company | Standalone Logic/Rule | SPEC-005 / SPEC-007 |
| Owen invites a Reviewer colleague | Send the new contact their first-sign-in invitation email | Standalone Notification | SPEC-010 |
| Owen invites a Reviewer colleague | Alert Nadia that a Primary contact invited a colleague | Standalone Notification | SPEC-011 |
| Nadia changes a contact's role | Apply the change to future actions only; leave past acceptances/approvals under the prior role untouched | Standalone Logic/Rule | SPEC-008 |
| Nadia changes a contact's role | Write a role-change trail entry | Cross-feature -- audit trail | FEAT-13 responsibility |
| Nadia attempts to remove a contact | Check whether the contact is the client's last Primary; block with a replacement-first prompt if so | Standalone Logic/Rule | SPEC-006 |
| Nadia confirms removing a contact | Revoke access immediately, erase personal contact details, retain evidentiary records under the contact's name | Standalone Automation | SPEC-009 |
| Nadia confirms removing a contact | Write a removal trail entry | Cross-feature -- audit trail | FEAT-13 responsibility |
| A client has zero Primary contacts and Nadia (or Owen) attempts to send/rely on a proposal | Block the send, show a clear prompt to add a Primary contact first | Standalone Logic/Rule (cross-feature enforcement point) | SPEC-006 |
| A contact's first magic-link sign-in succeeds | Set the contact's status to Active and record the login timestamp | Cross-feature | FEAT-05 responsibility |
| Nadia saves a new or edited contact successfully | Show a success confirmation, return to the contact list | Inline in triggering screen | SPEC-002 |
| Owen invites a colleague successfully | Show a success confirmation on his own view | Inline in triggering screen | SPEC-004 |

## Shared Context

**Shared Entities:**
- Client Contact -- created by SPEC-002 (Nadia) and SPEC-004 (Owen), listed by SPEC-001 and SPEC-004, updated by SPEC-002, removed by SPEC-003/SPEC-009, validated by SPEC-005, and governed by SPEC-006/SPEC-007/SPEC-008. Fields: name, email, role (Primary or Reviewer), invited_by, status (Invited/Active/Removed), last_sign_in.
- Client (read-only) -- read by SPEC-001, SPEC-002, and SPEC-004 to scope every contact-management view and form to the correct company.

**Shared UI Patterns:**
- Contact row -- displays name, email, role label, and status; used identically by SPEC-001 (Nadia's full list) and, in an own-company-scoped form, by SPEC-004 (Owen's list of his own company's contacts). Spec Writers for both should describe the role/status labeling consistently.
- Add/Invite Contact form -- shared name/email/role fields between SPEC-002 (Nadia: role can be Primary or Reviewer) and SPEC-004 (Owen: role fixed to Reviewer, no picker shown). Both submit through SPEC-005 validation.

**Shared Validation:**
- SPEC-005 defines field-level and uniqueness validation. SPEC-002 and SPEC-004 both reference SPEC-005 rather than duplicating the rules.
- SPEC-007 defines who may see or perform each action (including that Priya's role never shows contact-management access at all). SPEC-001 through SPEC-004 all reference SPEC-007 for what to show or hide per role, rather than each screen defining its own authorization logic.

## Internal Dependency Map

```
SPEC-001 (Client Contact List) -> [Nadia taps "Add Contact"] -> SPEC-002 (Add or Edit Client Contact)
SPEC-001 (Client Contact List) -> [Nadia taps an existing contact] -> SPEC-002 (Add or Edit Client Contact)
SPEC-001 (Client Contact List) -> [Nadia taps "Remove"] -> SPEC-003 (Remove Client Contact)
SPEC-002 (Add or Edit Client Contact) -> [Nadia taps Save] -> SPEC-005 (Contact Field Validation Rules) -> [valid] -> SPEC-001 (Client Contact List)
SPEC-002 (Add or Edit Client Contact) -> [new contact saved] -> SPEC-010 (New Contact Invitation Email)
SPEC-002 (Add or Edit Client Contact) -> [role assigned or changed] -> SPEC-007 (Role Authorization Rules)
SPEC-002 (Add or Edit Client Contact) -> [role changed on existing contact] -> SPEC-008 (Role Change Effective-Timing Rule)
SPEC-004 (Invite Reviewer Colleague) -> [Owen taps Save] -> SPEC-005 (Contact Field Validation Rules) -> [valid] -> SPEC-004 (confirmation)
SPEC-004 (Invite Reviewer Colleague) -> [restricted by] -> SPEC-007 (Role Authorization Rules)
SPEC-004 (Invite Reviewer Colleague) -> [new contact invited] -> SPEC-010 (New Contact Invitation Email)
SPEC-004 (Invite Reviewer Colleague) -> [new contact invited] -> SPEC-011 (Primary-Invited-Colleague Alert)
SPEC-003 (Remove Client Contact) -> [checks] -> SPEC-006 (Primary Contact Requirement Rule) -> [blocked if last Primary] -> SPEC-003 (shows block message)
SPEC-003 (Remove Client Contact) -> [confirmed] -> SPEC-009 (Contact Removal & Data Erasure) -> [complete] -> SPEC-001 (Client Contact List)
SPEC-007 (Role Authorization Rules) -> [governs entitlements referenced by] -> SPEC-001, SPEC-002, SPEC-003, SPEC-004
```

**Default Entry (freelancer side):** SPEC-001 (Client Contact List) -- the screen Nadia reaches from a client's detail view to manage that client's contacts and roles.
**Default Entry (client side):** SPEC-004 (Invite Reviewer Colleague) -- the screen Owen reaches from his portal home when he chooses to invite a colleague; there is no other client-side entry point into this feature.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-18.SPEC-001 | Inbound | FEAT-01 (Client & Project Management) | Nadia navigates from a client's detail view to manage its contacts | Nadia taps "Manage Contacts" on a client detail page |
| FEAT-18.SPEC-006 | Outbound | FEAT-02 (Proposal Creation & Sending) | Blocks sending a proposal until the client has at least one Primary contact | Nadia attempts to send a proposal for a client with no Primary contact |
| FEAT-18.SPEC-006 / FEAT-18.SPEC-007 | Outbound | FEAT-03 (Proposal Acceptance) | A Primary contact is required for, and is the only role entitled to, accepting a proposal | Client acceptance flow checks the acting contact's role and existence |
| FEAT-18.SPEC-010 | Outbound | FEAT-05 (Client Portal Access) | The invitation email carries the contact's first magic-link sign-in | Contact opens the magic-link from the invitation email |
| FEAT-18.SPEC-004 | Inbound | FEAT-05 (Client Portal Access) | Owen reaches the invite screen from his portal home | Owen taps "Invite a colleague" from his portal home |
| FEAT-18.SPEC-007 | Outbound | FEAT-05 (Client Portal Access) | Role determines the scoped view a contact sees after signing in | Contact signs in and the portal renders its role-scoped view |
| FEAT-18.SPEC-001 | Inbound | FEAT-14 (Notifications/Email) | Nadia navigates here to correct a bouncing contact email flagged by a delivery warning | Nadia taps a delivery warning on a project |
| FEAT-18.SPEC-010 / FEAT-18.SPEC-011 | Outbound | FEAT-14 (Notifications/Email) | Both notifications are delivered, and their delivery/bounce status reported, through the transactional email delivery capability | A notification is triggered |
| FEAT-18.SPEC-001 / FEAT-18.SPEC-009 | Outbound | FEAT-13 (Immutable Activity & Audit Trail) | Role-change and removal events each write an append-only trail entry | A role is changed or a contact is removed |
| FEAT-18.SPEC-001 | Inbound | FEAT-31 (Support Access) | Dana views the client's contact list read-only inside a logged support session | Dana opens a support session and navigates to a client's contacts |
| FEAT-18.SPEC-006 / FEAT-18.SPEC-009 | Inbound | FEAT-24 (Account Deletion) | Account deletion removes remaining client contacts' personal data, subject to the same evidence-retention boundary | Freelancer deletes her account |

## Non-Functional Notes

**Data volumes / growth:** Each client company carries only a handful of contacts, and a freelancer manages 3-15 active clients (scope-boundaries.md, SC-21), so a single client's contact list and a freelancer's total contact roster both stay small; no special scaling design is needed for list rendering or search.

**Responsiveness:** SPEC-001, SPEC-002, and SPEC-003 are Nadia's freelancer-side screens and are expected to be responsive under ordinary business use (assumptions-constraints.md, ASMP-26), without the stricter ~2-second client-facing target. SPEC-004 is a client-facing screen Owen typically reaches from his phone, so it meets the same near-immediate interactivity expectation as other client-facing pages (assumptions-constraints.md, ASMP-21).

**Data sensitivity / privacy:** Client Contact holds personal data -- name and email -- of individuals at client companies worldwide, treated as GDPR-class personal data (assumptions-constraints.md, ASMP-24). An erasure request removes a contact's personal details while their acceptances and approvals remain on the record under their name as evidence (assumptions-constraints.md context, ASMP-20; SC-24). A contact is never visible to another client company (assumptions-constraints.md, ASMP-23).

**Compliance flags:** GDPR-class handling applies to all Client Contact personal data, with a documented erasure path (XBR-27) that still preserves the correctness of financial and evidentiary records (assumptions-constraints.md, ASMP-25); no card or payment data is ever captured by this feature (assumptions-constraints.md, ASMP-24).

## Non-Goals

- **Client-side roles beyond Primary and Reviewer** -- Excluded per scope-boundaries.md (SC-02): the persona set establishes exactly these two client-contact roles per the founder's leaning in BRIEF.md, and no profiled competitor offers even this two-role split, so there is no signal pulling toward additional tiers such as a client-side "admin."
- **Agency or team-of-many staff seats for the freelancer side** -- Excluded per scope-boundaries.md (SC-01): the product has no internal-staff seat model (e.g., a scoped bookkeeper or contractor role); Nadia is the sole freelancer-side actor and Dana's read-only support role (FEAT-31) is the only other internal identity.
- **Support Operator changes to contacts** -- Excluded per scope-boundaries.md (SC-04): Dana's support sessions are read-only and logged; she can see a bouncing contact email but never edits, adds, or removes a contact, and always tells Nadia to make the correction herself.
- **Bulk import of contacts** -- Consistent with scope-boundaries.md's exclusion of bulk import for other entities (SC-19) and the expectation of only a handful of contacts per client (SC-21, BRIEF.md, Scale & Non-Functional Expectations); individual add and invite entry is fast enough that a bulk path is unnecessary and would risk importing contacts never verified through the product's own flow.
- **Restoring a removed contact** -- Intentional lifecycle decision surfaced by the CRUD matrix: because removal is also the erasure mechanism (XBR-27), the contact's personal details are genuinely gone; a returning contact is re-added as a new Client Contact record rather than restored, so the erasure guarantee is never silently undone.
- **Automatic purge of a removed contact's evidentiary records** -- Intentional non-goal surfaced by the CRUD matrix: acceptances and approvals a removed contact gave remain on the record under their name for as long as the freelancer's account exists (scope-boundaries.md, SC-24; record immutability), with no automatic purge window, except where FEAT-24 account deletion applies.
