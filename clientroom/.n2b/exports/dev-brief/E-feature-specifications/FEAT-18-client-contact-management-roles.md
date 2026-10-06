# FEAT-18 — Client Contact Management & Roles

This chapter covers Client Contact Management & Roles, a Important-tier feature. It contains the feature breakdown brief followed by every specification in full: 11 specifications carrying 144 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-18.SPEC-001 | Client Contact List | screen | 12 |
| FEAT-18.SPEC-002 | Add or Edit Client Contact | screen | 15 |
| FEAT-18.SPEC-003 | Remove Client Contact | screen | 13 |
| FEAT-18.SPEC-004 | Invite Reviewer Colleague | screen | 13 |
| FEAT-18.SPEC-005 | Contact Field Validation Rules | logic-rule | 14 |
| FEAT-18.SPEC-006 | Primary Contact Requirement Rule | logic-rule | 12 |
| FEAT-18.SPEC-007 | Role Authorization Rules | logic-rule | 19 |
| FEAT-18.SPEC-008 | Role Change Effective-Timing Rule | logic-rule | 11 |
| FEAT-18.SPEC-009 | Contact Removal & Data Erasure | automation | 13 |
| FEAT-18.SPEC-010 | New Contact Invitation Email | notification | 12 |
| FEAT-18.SPEC-011 | Primary-Invited-Colleague Alert | notification | 10 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Client Contact List

## Overview

**Name:** Client Contact List
**ID:** FEAT-18.SPEC-001
**Type:** Screen
**Purpose:** Nadia views and manages the contacts and roles for one client company; Dana views the same list read-only inside a logged support session.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles

## Scope and Non-Goals

**In Scope:**
- Listing every Client Contact belonging to one client company, with role and status
- Launching contact creation (FEAT-18.SPEC-002), editing (FEAT-18.SPEC-002), and removal (FEAT-18.SPEC-003)
- The empty state that blocks proposal sending until a Primary contact exists (FEAT-18.SPEC-006)
- Dana's read-only mirrored view of this same list inside a support session (FEAT-31)

**Non-Goals:**
- Entering or editing contact fields -- handled by FEAT-18.SPEC-002 (Add or Edit Client Contact); this list only launches that screen
- Confirming a removal -- handled by FEAT-18.SPEC-003 (Remove Client Contact); this list only launches that screen
- Owen inviting a Reviewer colleague -- handled by FEAT-18.SPEC-004 (Invite Reviewer Colleague), Owen's own client-side entry point; this screen is Nadia's freelancer-side surface and Owen never reaches it (Access Matrix: Owen carries None for Client & Project Management)
- Bulk import of contacts -- excluded per scope-boundaries.md (SC-19): a client carries only a handful of contacts, so individual add/invite entry is fast enough and a bulk path would risk importing contacts never verified through the product's own flow

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-004 (Client Detail) | Nadia taps "Manage Contacts" | The client reference, scoping this list to that client |
| FEAT-14 (Notifications (Email)), delivery warning | Nadia taps a delivery warning on a project flagging a bouncing contact email | The client reference and the specific contact whose email is bouncing, so the affected row is highlighted |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Dana navigates to a client's contacts inside an open, read-only support session | The client reference; the session's permanent read-only banner and disabled controls (FEAT-31.SPEC-003) |
| FEAT-18.SPEC-002 (Add or Edit Client Contact) | A save completes successfully | Returns to this list, refreshed |
| FEAT-18.SPEC-003 (Remove Client Contact) | A removal completes, or is blocked | Returns to this list, refreshed |
| FEAT-18.SPEC-011 (Primary-Invited-Colleague Alert) | Nadia taps the alert email's "View contacts" CTA | The client reference for {client_company_name}, scoping this list to that client |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, for any client she owns | Add, edit, remove contacts and change roles | -- |
| Owen (Client Primary Contact) | No | No | Not shown in Owen's portal navigation; this is the freelancer's own management surface (Access Matrix: Owen carries None for Client Contact Management on the freelancer side) |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation (Access Matrix: None) |
| Dana (Support Operator) | Full screen, read-only, for the one freelancer account named by her open support session (FEAT-31.SPEC-005: one account at a time) | View only -- Add, edit, and remove controls are not rendered | Attempting to reach an add/edit/remove action directly returns Dana to the read-only view with no change made, under the permanent "Read-only support session" banner (FEAT-31.SPEC-003) |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- no in-progress form data exists on this list screen to preserve |

## Layout and Content

**Header:** Client name (breadcrumb back to FEAT-01.SPEC-004, Client Detail) with screen title "Contacts." An "Add Contact" action button, top right, visible to Nadia only.

**Body:** A vertical list of contact rows, one per Client Contact belonging to this client, ordered by role (Primary contacts first) then name. Each row shows:
- Name
- Email
- Role badge (Primary or Reviewer)
- Status badge (Invited, Active, or Removed contacts are never shown in this list -- see Data Model)
- Last sign-in (relative time, or "Not yet signed in" for Invited contacts)
- Row actions, visible to Nadia only: "Edit" and "Remove"

If the delivery-warning entry point (FEAT-14) brought Nadia here, the flagged contact's row is highlighted with a small inline note: "This email is bouncing."

For Dana's read-only view, row actions are not rendered; the row content is otherwise identical.

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Contact rows stack in a single column, full width; role and status badges wrap below the name/email on narrow widths; row actions collapse into an overflow menu.
- **Medium size class and above:** Rows remain single-column, capped at a consistent platform-wide content width and horizontally centered; role, status, and last-sign-in display inline with name and email; row actions show as separate buttons rather than an overflow menu.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Add Contact" button (Nadia only) | Tap | Navigate to FEAT-18.SPEC-002 (Add or Edit Client Contact) in create mode, scoped to this client | Screen transitions | Standard navigation transition |
| Contact row (Nadia only) | Tap | Navigate to FEAT-18.SPEC-002 in edit mode, pre-filled with this contact's data | Screen transitions | Standard navigation transition |
| "Edit" row action (Nadia only) | Tap | Same as tapping the row | Screen transitions | Standard navigation transition |
| "Remove" row action (Nadia only) | Tap | Navigate to FEAT-18.SPEC-003 (Remove Client Contact) for this contact | Screen transitions | Standard navigation transition |
| "This email is bouncing" note (Nadia only) | Tap | Navigates to FEAT-18.SPEC-002 in edit mode for that contact, with the email field focused | Screen transitions | Standard navigation transition |
| Contact row (Dana) | Tap | No action -- row is display-only in the read-only view | None | None |

### Accessibility Notes

- **Focus order:** Breadcrumb -> "Add Contact" (when visible) -> contact rows in list order, each row's Edit action before its Remove action (when visible).
- **Dynamic updates:** A row appearing, disappearing, or changing role/status after returning from FEAT-18.SPEC-002 or FEAT-18.SPEC-003 is announced to assistive technology as a list-content change.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton rows in place of the contact list | Screen first opens | Data finishes loading |
| Loaded (has contacts) | Full list as described in Layout and Content | Data load completes with one or more contacts | Navigation away, or a contact is added/edited/removed |
| Empty | "No contacts yet. Add a Primary contact before you can send a proposal to this client." with the "Add Contact" action (Nadia only); Dana's read-only view shows the same message with no action | Client has zero contacts | A contact is successfully added |
| Error | Error banner: "Couldn't load contacts. Try again." with a Retry button | Initial data load fails | Retry succeeds |
| Offline/Degraded | N/A -- contact management is an infrequent, connectivity-required action (feature-overview.md, States); this screen assumes connectivity and shows the Error state on a load failure rather than a distinct offline mode | -- | -- |

## Validation Rules

Validation governed by FEAT-18.SPEC-005 (Contact Field Validation Rules). This list screen performs no direct input validation of its own -- all field entry happens on FEAT-18.SPEC-002.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Breadcrumb tap | FEAT-01.SPEC-004 (Client Detail) | FEAT-01 |
| "Add Contact" tap | FEAT-18.SPEC-002 (Add or Edit Client Contact) | -- |
| Contact row / "Edit" tap | FEAT-18.SPEC-002 (Add or Edit Client Contact) | -- |
| "Remove" tap | FEAT-18.SPEC-003 (Remove Client Contact) | -- |

## Data Model

**Creates:** None -- this screen only lists and launches other specs' create/edit/remove flows.
**Reads:** Client Contact -- name, email, role, status, last_sign_in, for every contact belonging to the scoped Client, excluding Removed contacts (a removed contact's personal details are erased per FEAT-18.SPEC-009 and it no longer appears in this list). Client -- client_name, for the header.
**Updates:** None directly.
**Deletes:** None directly.

## Business Rules

- Only Nadia may add, edit, or remove contacts from this screen (FEAT-18.SPEC-007, Role Authorization Rules).
- Removed contacts never appear in this list; a returning contact must be re-added as a new Client Contact record (feature-overview.md, Non-Goals -- no restore path).
- A client with zero Primary contacts shows the blocking prompt described in the Empty state, consistent with FEAT-18.SPEC-006 (Primary Contact Requirement Rule) and XBR-07.
- Dana's view mirrors this screen exactly except that every action control is omitted, per FEAT-31.SPEC-005's unconditional read-only rule.

## Edge Cases

- **Owen adds a Reviewer colleague (FEAT-18.SPEC-004) while Nadia has this list open** -- This screen is a snapshot loaded on open, not live-updating; the new contact does not appear until Nadia reloads the list or navigates back to it (for example, after returning from FEAT-18.SPEC-002 or FEAT-18.SPEC-003, which always re-fetch).
- **A contact's role or status changes in another of Nadia's own open sessions while this list is open** -- The list shown here can become stale; the authoritative state is re-fetched whenever this screen is (re)entered. Opening FEAT-18.SPEC-002 to edit a contact always loads the contact's current state at that moment, so a stale row here does not cause a stale edit.
- **Nadia removes the client's last Primary contact from another session while this list is open** -- No conflict is possible on this list screen itself: FEAT-18.SPEC-006 re-checks the last-Primary condition at the moment of the removal's own commit (on FEAT-18.SPEC-003), not against what this list displays.
- **The client has contacts but all are Reviewers (no Primary)** -- The list displays normally, but the same blocking prompt as the Empty state's rationale applies at proposal-send time (FEAT-18.SPEC-006); this screen itself shows an inline banner above the list: "This client has no Primary contact. A proposal cannot be sent until one is added or promoted."
- **Dana's support session closes while she is viewing this list** -- She is returned to the Operator Support Session Console queue (FEAT-31.SPEC-002); this screen becomes unreachable for that account until a new session opens.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-002 (Add or Edit Client Contact) | Navigation (outbound) | "Add Contact" and row/"Edit" taps launch it |
| FEAT-18.SPEC-003 (Remove Client Contact) | Navigation (outbound) | "Remove" tap launches it |
| FEAT-18.SPEC-006 (Primary Contact Requirement Rule) | References (inbound) | Governs the no-Primary blocking banner and empty-state message |
| FEAT-18.SPEC-007 (Role Authorization Rules) | References (inbound) | Governs which roles see this screen and its actions |
| FEAT-01.SPEC-004 (Client Detail) | Navigation (inbound) | "Manage Contacts" link arrives here |
| FEAT-14 (Notifications (Email)) | Navigation (inbound) | A delivery warning on a project links here to correct a bouncing contact email |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Navigation (inbound) | Dana's read-only entry point during an open session |
| FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement) | References (inbound) | Enforces the disabled controls Dana sees |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| contact_list_viewed | viewer_role (freelancer / support_operator), contact_count, has_primary (yes/no) | Screen finishes loading | N/A -- no success-metrics.md metric is connected to Client Contact Management & Roles; retained so contact-list engagement is observable |
| contact_list_no_primary_banner_shown | contact_count | The no-Primary banner renders because the client has contacts but no Primary | N/A -- no success-metrics.md metric is connected to this feature |

## Acceptance Criteria

**FEAT-18.SPEC-001-AC-01:** Given Nadia opens a client with three contacts, when the screen loads, then she sees all three contacts listed with name, email, role badge, and status.

**FEAT-18.SPEC-001-AC-02:** Given Nadia is on the Client Contact List for a client with no contacts, when the screen loads, then she sees "No contacts yet. Add a Primary contact before you can send a proposal to this client." with an "Add Contact" action.

**FEAT-18.SPEC-001-AC-03:** Given Nadia taps "Add Contact", when the tap registers, then she is navigated to FEAT-18.SPEC-002 in create mode, scoped to this client.

**FEAT-18.SPEC-001-AC-04:** Given Nadia taps an existing contact's row, when the tap registers, then she is navigated to FEAT-18.SPEC-002 in edit mode, pre-filled with that contact's current data.

**FEAT-18.SPEC-001-AC-05:** Given Nadia taps "Remove" on a contact row, when the tap registers, then she is navigated to FEAT-18.SPEC-003 for that contact.

**FEAT-18.SPEC-001-AC-06:** Given Nadia arrives here from a delivery warning on a bouncing contact email, when the screen loads, then that contact's row is highlighted with an inline note "This email is bouncing."

**FEAT-18.SPEC-001-AC-07:** Given a client has one or more Reviewer contacts but no Primary contact, when Nadia views this list, then an inline banner reads "This client has no Primary contact. A proposal cannot be sent until one is added or promoted."

**FEAT-18.SPEC-001-AC-08:** Given Dana is inside an open support session on Nadia's account, when she navigates to a client's contacts, then she sees the same list with no "Add Contact", "Edit", or "Remove" controls rendered, under the permanent read-only banner.

**FEAT-18.SPEC-001-AC-09:** Given Dana views the Client Contact List, when she looks for a way to change a contact, then no such control exists, and there is no way to trigger a change from this screen.

**FEAT-18.SPEC-001-AC-10:** Given Owen or Priya is signed in to the client portal, when they look for any route to this screen, then none exists -- the screen is not part of the client portal's navigation.

**FEAT-18.SPEC-001-AC-11:** Given the initial data load for this screen fails, when the failure occurs, then Nadia sees "Couldn't load contacts. Try again." with a Retry button.

**FEAT-18.SPEC-001-AC-12:** Given a contact was removed in another of Nadia's sessions while this list was open, when Nadia returns to this list from FEAT-18.SPEC-002 or FEAT-18.SPEC-003, then the list re-fetches and no longer shows the removed contact.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 4 (loading, empty, error, offline-degraded N/A) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Screen Spec: Add or Edit Client Contact

## Overview

**Name:** Add or Edit Client Contact
**ID:** FEAT-18.SPEC-002
**Type:** Screen
**Purpose:** Nadia adds a new contact to a client company or edits an existing contact's name, email, or role.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles

## Scope and Non-Goals

**In Scope:**
- Creating a new Client Contact record (name, email, role) for the scoped client
- Editing an existing Client Contact's name, email, or role
- Inline field validation and save-time validation via FEAT-18.SPEC-005
- Triggering the first-sign-in invitation email (FEAT-18.SPEC-010) on a successful create
- Applying the role-change effective-timing behavior (FEAT-18.SPEC-008) when an existing contact's role changes

**Non-Goals:**
- Removing a contact -- handled by FEAT-18.SPEC-003 (Remove Client Contact); this screen never deletes a Client Contact record
- Owen's own invite screen -- handled by FEAT-18.SPEC-004 (Invite Reviewer Colleague), which shares this screen's field set but fixes the role to Reviewer and restricts the scope to Owen's own company; this spec covers Nadia's freelancer-side form only
- Choosing which client the contact belongs to -- the client is fixed by the entry context (FEAT-18.SPEC-001); this screen never re-scopes a contact to a different client
- Restoring a previously removed contact -- excluded per the feature's Entity-Lifecycle Coverage Matrix: because removal is also the erasure mechanism (XBR-27), a returning contact is entered here as a brand-new Client Contact record, never as a restore of the old one

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-18.SPEC-001 (Client Contact List) | Nadia taps "Add Contact" | Client reference; form starts empty (create mode) |
| FEAT-18.SPEC-001 (Client Contact List) | Nadia taps an existing contact's row or "Edit" | Client reference and the selected contact's current name, email, and role (edit mode) |
| FEAT-18.SPEC-001 (Client Contact List) | Nadia taps a "This email is bouncing" note | Same as edit mode, with the email field pre-focused |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Create a contact with any role, or edit an existing contact's name, email, or role | -- |
| Owen (Client Primary Contact) | No | No | Owen never reaches this screen; his own invite screen is FEAT-18.SPEC-004 (Access Matrix: Client Contact Management is Own-only, scoped to inviting -- not this general add/edit surface) |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation (Access Matrix: None) |
| Dana (Support Operator) | No | No | Dana's mirrored view of FEAT-18.SPEC-001 never exposes this screen, since Support Access Sessions are read-only and never open a create/edit form (FEAT-31.SPEC-005) |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- entered form data is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "New Contact" (create mode) or "Edit Contact" (edit mode), with a back arrow (returns to FEAT-18.SPEC-001) and a "Save" action button, right-aligned.

**Body:** A single-column form with the following fields in order:
- Name (text input, required)
- Email (text input, required)
- Role (selection input: Primary or Reviewer, required)

In edit mode, all three fields are pre-filled with the contact's current values. The Role field additionally shows a note when changed: "This change applies to this contact's future actions only; it never alters a past acceptance or approval" (FEAT-18.SPEC-008).

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact size class:** Single-column form as described, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-18.SPEC-001 (Client Contact List) | Screen closes | Standard navigation transition, or a confirmation dialog first if there are unsaved changes |
| Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Name input | Blur (empty) | Triggers validation via FEAT-18.SPEC-005 | Error state on field | "Name is required" below the field |
| Email input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Email input | Blur | Triggers format and per-client-uniqueness validation via FEAT-18.SPEC-005 | Error state on field if invalid | "Please enter a valid email address" or "This email is already used by another contact at this client" |
| Role selector | Select | Sets the contact's role to Primary or Reviewer | Selector shows chosen role; if editing an existing contact and the value differs from the loaded value, the future-only note appears | Selected role displayed; note text shown when applicable |
| Save button | Tap | 1. Validate all fields via FEAT-18.SPEC-005. 2. If valid and creating, save the new contact and trigger FEAT-18.SPEC-010 (invitation email). 3. If valid and editing, save changes; if the role changed, apply FEAT-18.SPEC-008's future-only effective timing. | Button shows loading state during save | Success: toast "Contact added" (create) or "Contact updated" (edit), then navigate to FEAT-18.SPEC-001. Failure: inline field errors, or a form-level error banner for a non-field failure. |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "Retry" button (load-failure banner, edit mode) | Tap | Re-fetches the contact's current name, email, and role | Screen returns to Loading, then Loaded on success | Skeleton fields while loading; on success the form is pre-filled and the banner disappears; on repeated failure the same banner reappears |

### Accessibility Notes

- **Focus order:** Back arrow -> Name -> Email -> Role selector -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The success toast is announced on save; on validation failure, focus moves to the first field in error.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading (edit mode) | Skeleton placeholders in place of the three fields; Save disabled | Screen opens in edit mode and the contact's current data is being fetched | Fetch completes (Loaded) or fails (Load Error) |
| Load Error (edit mode) | Error banner: "Couldn't load this contact. Try again." with a Retry button; fields not shown and Save disabled | The edit-mode fetch of the contact fails | Retry succeeds (Loaded) or Nadia taps the back arrow |
| Empty (create mode default) | All fields empty, Save enabled | Screen opens in create mode | Nadia begins typing in any field |
| Loaded (edit mode default) | Fields pre-filled with the contact's current data, Save enabled | Screen opens in edit mode | Nadia edits any field |
| Filling | Fields contain user input | Nadia types in any field | Save is tapped or Nadia navigates away |
| Validating | Save button shows a loading spinner | Save is tapped | Validation completes (pass or fail) |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails (FEAT-18.SPEC-005) | Nadia corrects the field and re-triggers validation |
| Saving | Save button shows a loading spinner, fields disabled | Validation passes | Save completes or fails |
| Error | Error banner: "Couldn't save this contact. Try again." with a Retry button; entered field values are preserved for retry | Save operation fails for a reason other than field validation | Retry succeeds |
| Offline/Degraded | N/A -- contact management is an infrequent, connectivity-required action (feature-overview.md, States); a save attempted without connectivity surfaces the Error state above rather than a distinct offline queue | -- | -- |

## Validation Rules

Validation governed by FEAT-18.SPEC-005 (Contact Field Validation Rules). See that spec for all field-level and cross-field rules (required fields, email format, per-client email uniqueness). This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-18.SPEC-001 (Client Contact List) | -- |
| Successful save | FEAT-18.SPEC-001 (Client Contact List) | -- |
| Cancel with unsaved changes | FEAT-18.SPEC-001 (Client Contact List), after a confirmation dialog | -- |

## Data Model

**Creates:** Client Contact -- name, email, role (Primary or Reviewer) set from form input; invited_by set to Nadia; status set to Invited.
**Reads:** Client Contact -- name, email, role, when opened in edit mode.
**Updates:** Client Contact -- name, email, role. A role change is applied per FEAT-18.SPEC-008 (future actions only; never alters a past acceptance or approval recorded under the prior role).
**Deletes:** None.

## Business Rules

- Field validation (FEAT-18.SPEC-005) is enforced -- Nadia cannot save with invalid data.
- Role assignment is governed by FEAT-18.SPEC-007 (Role Authorization Rules); Nadia may assign either Primary or Reviewer to any contact she manages.
- A role change on save is subject to FEAT-18.SPEC-008 (Role Change Effective-Timing Rule): the new role governs only future actions.
- Saving a new contact triggers FEAT-18.SPEC-010 (New Contact Invitation Email) automatically -- Nadia cannot skip it.
- XBR-07: creating a client's first Primary contact here is what allows a proposal to be sent for that client (FEAT-18.SPEC-006).

## Edge Cases

- **Nadia navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Save twice rapidly** -- The second tap is ignored while the first save is in progress (button in loading state).
- **The contact's data fails to load in edit mode** -- The Load Error state shows "Couldn't load this contact. Try again." with Retry; no editable form is shown, so Nadia can never save over data she has not seen.
- **Network failure during save** -- Error banner: "Couldn't save this contact. Try again." with a Retry button. Form data is preserved.
- **Email entered matches another contact at the same client** -- Rejected per FEAT-18.SPEC-005: "This email is already used by another contact at this client." Save does not proceed.
- **Contact edited by Nadia in another session between load and save** -- Save is rejected with a dialog: "This contact was updated in another session. Review the latest version before saving." with "View Latest" (reloads the record; local edits discarded after confirmation) and "Keep Editing" options. Resolution: reject-with-refresh, per the dependency map's Contention note for the Client Contact entity.
- **Nadia changes an existing Primary contact's role to Reviewer, and the contact is the client's last Primary** -- Save is blocked with the message: "This is the client's only Primary contact. Add or promote another Primary before changing this one." (FEAT-18.SPEC-006), consistent with the last-Primary protection applied to removal.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-001 (Client Contact List) | Navigation (inbound and outbound) | Entry point and return destination |
| FEAT-18.SPEC-005 (Contact Field Validation Rules) | References (inbound) | Field-level and cross-field validation applied on blur and submit |
| FEAT-18.SPEC-006 (Primary Contact Requirement Rule) | References (inbound) | Blocks a role change that would leave the client with no Primary |
| FEAT-18.SPEC-007 (Role Authorization Rules) | References (inbound) | Governs Nadia's unrestricted role-assignment entitlement |
| FEAT-18.SPEC-008 (Role Change Effective-Timing Rule) | Triggers (outbound) | Applied whenever an existing contact's role changes on save |
| FEAT-18.SPEC-010 (New Contact Invitation Email) | Triggers (outbound) | Fired automatically when a new contact is saved |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| contact_added | role_assigned (primary / reviewer) | A new contact save completes | N/A -- no success-metrics.md metric is connected to Client Contact Management & Roles; retained so contact-capture activity is observable |
| contact_edited | fields_changed (name / email / role, may be multiple) | An existing contact's edit save completes | N/A -- no success-metrics.md metric is connected to this feature |
| contact_role_changed | previous_role, new_role | A save changes an existing contact's role | N/A -- no success-metrics.md metric is connected to this feature |
| contact_save_failed | reason (validation / network / stale_record) | A save attempt does not complete | N/A -- no success-metrics.md metric is connected to this feature |

## Acceptance Criteria

**FEAT-18.SPEC-002-AC-01:** Given Nadia is on this screen in create mode, when she enters "Owen Carter" as name, "owen@acme.test" as email, selects Primary, and taps Save, then the contact is created with status Invited, the invitation email (FEAT-18.SPEC-010) fires, and she sees "Contact added" before returning to FEAT-18.SPEC-001.

**FEAT-18.SPEC-002-AC-02:** Given Nadia taps Save with the name field empty, then the name field shows "Name is required" and the save does not proceed.

**FEAT-18.SPEC-002-AC-03:** Given Nadia enters an email already used by another contact at the same client, when she blurs the email field, then it shows "This email is already used by another contact at this client."

**FEAT-18.SPEC-002-AC-04:** Given Nadia opens an existing Reviewer contact in edit mode and changes the role to Primary, when she taps Save, then a note confirms the change applies to future actions only, and the save completes.

**FEAT-18.SPEC-002-AC-05:** Given Nadia opens the client's only Primary contact in edit mode and changes the role to Reviewer, when she taps Save, then the save is blocked with "This is the client's only Primary contact. Add or promote another Primary before changing this one."

**FEAT-18.SPEC-002-AC-06:** Given Nadia has unsaved changes on this screen, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?"

**FEAT-18.SPEC-002-AC-07:** Given Nadia loses connectivity while attempting to save, then the error banner "Couldn't save this contact. Try again." appears with a Retry button, and her entered data is preserved.

**FEAT-18.SPEC-002-AC-08:** Given a contact Nadia is editing was updated in another of her sessions before she saves, when she taps Save, then the save is rejected with "This contact was updated in another session. Review the latest version before saving." and "View Latest" / "Keep Editing" options.

**FEAT-18.SPEC-002-AC-09:** Given Nadia taps Save twice in rapid succession, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-18.SPEC-002-AC-10:** Given Nadia arrives from a delivery warning on a bouncing contact email, when the screen opens, then it is in edit mode for that contact with the email field focused.

**FEAT-18.SPEC-002-AC-11:** Given Owen, Priya, or Dana attempts to reach this screen through any route, then none exists for any of them -- the screen is not part of the client portal or the support console.

**FEAT-18.SPEC-002-AC-12:** Given Nadia's session expires while she has unsaved form data, when the expiry dialog appears and she re-authenticates, then her entered field values are restored.

**FEAT-18.SPEC-002-AC-13:** Given Nadia enters a name and email but leaves Role at its default, when she taps Save, then the Role selection is required and validation blocks the save if no role was ever selected.

**FEAT-18.SPEC-002-AC-14:** Given Nadia opens an existing contact in edit mode, when the contact is still being fetched, then she sees skeleton placeholders in place of the fields and Save is disabled until the data has loaded.

**FEAT-18.SPEC-002-AC-15:** Given the edit-mode fetch of the contact fails, when the failure occurs, then Nadia sees "Couldn't load this contact. Try again." with a Retry button and no editable form, and when she taps Retry and the fetch succeeds, then the form is pre-filled with the contact's current name, email, and role.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 6 (loading, load error, validation error, saving error, offline-degraded N/A, stale-record conflict) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Screen Spec: Remove Client Contact

## Overview

**Name:** Remove Client Contact
**ID:** FEAT-18.SPEC-003
**Type:** Screen
**Purpose:** Nadia confirms removing a contact, sees the last-Primary block when it applies, and is shown erasure-request framing before the removal executes.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles

## Scope and Non-Goals

**In Scope:**
- The confirmation dialog for removing one Client Contact
- The last-Primary block, with a prompt to add or promote a replacement first
- Framing the removal as ending access and erasing personal details, for both a routine departure and an explicit erasure request
- Handing off to FEAT-18.SPEC-009 (Contact Removal & Data Erasure) once confirmed

**Non-Goals:**
- Performing the actual access revocation, data erasure, and evidence retention -- owned by FEAT-18.SPEC-009 (Contact Removal & Data Erasure); this screen only confirms intent and hands off
- Adding or promoting a replacement Primary contact -- that happens on FEAT-18.SPEC-002 (Add or Edit Client Contact), which this screen links to when the last-Primary block applies
- Restoring a removed contact -- excluded per the feature's Entity-Lifecycle Coverage Matrix: removal is the erasure mechanism (XBR-27), so there is no restore path from this or any other screen

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-18.SPEC-001 (Client Contact List) | Nadia taps "Remove" on a contact row | The selected contact's identity, role, and status |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Confirm removal, or cancel | -- |
| Owen (Client Primary Contact) | No | No | Owen has no route to this screen; removal is Nadia-only everywhere in the product (Access Matrix: Client Contact Management is Own-only for invites, not removal) |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation (Access Matrix: None) |
| Dana (Support Operator) | No | No | Dana's read-only mirrored view of FEAT-18.SPEC-001 never exposes a Remove action, so this screen is unreachable during a support session (FEAT-31.SPEC-005) |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- returning to this screen re-loads the contact fresh rather than preserving any prior confirmation state |

## Layout and Content

**Header:** Screen title "Remove Contact" with a back arrow (returns to FEAT-18.SPEC-001, cancelling without removing).

**Body (standard case -- not the client's last Primary):** A confirmation dialog:
- The contact's name, email, and role
- Body text: "Removing {contact_name} ends their access immediately and erases their email address and other contact details. Any approvals or acceptances they gave stay on the record under their name only, as evidence of what was agreed."
- A checkbox or acknowledgement: "I understand this cannot be undone."
- "Remove Contact" action (destructive style) and "Cancel"

**Body (blocked case -- this is the client's last Primary):** A blocking message replaces the confirmation:
- "{contact_name} is this client's only Primary contact. Add or promote a replacement Primary before removing them."
- "Go to Contacts" action, linking back to FEAT-18.SPEC-001 (from which Nadia can open FEAT-18.SPEC-002 to designate a replacement)
- No "Remove Contact" action is shown in the blocked case.

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Confirmation content stacks in a single column, full width; actions stack vertically with "Remove Contact" or "Go to Contacts" above "Cancel".
- **Medium size class and above:** Content is capped at a consistent platform-wide dialog width and horizontally centered; actions display side by side.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-18.SPEC-001 (Client Contact List), no change made | Screen closes | Standard navigation transition |
| "I understand this cannot be undone" acknowledgement | Check | Enables the "Remove Contact" action | "Remove Contact" becomes enabled | Standard checkbox state |
| "Remove Contact" action | Tap | 1. Re-check the last-Primary condition (FEAT-18.SPEC-006) at the moment of commit. 2. If still eligible, trigger FEAT-18.SPEC-009 (Contact Removal & Data Erasure). | Button shows loading state | Success: toast "Contact removed" and navigate to FEAT-18.SPEC-001. Failure (became the last Primary since load): the screen switches to the blocked case in place. |
| "Cancel" action | Tap | Navigate to FEAT-18.SPEC-001, no change made | Screen closes | Standard navigation transition |
| "Go to Contacts" action (blocked case) | Tap | Navigate to FEAT-18.SPEC-001 | Screen closes | Standard navigation transition |
| "Retry" button (load-failure banner) | Tap | Re-reads the contact and re-derives whether it is the client's sole Primary | Screen returns to Loading, then to the standard or blocked case | Loading placeholder, then the resolved case; on repeated failure the same banner reappears |

### Accessibility Notes

- **Focus order:** Back arrow -> confirmation body text -> acknowledgement checkbox (standard case) -> "Remove Contact" / "Go to Contacts" -> "Cancel".
- **Dynamic updates:** A switch from the standard case to the blocked case (because the contact became the last Primary between load and commit) is announced to assistive technology as a content change, and focus moves to the blocking message.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton placeholder for the contact summary; no "Remove Contact" or "Go to Contacts" action shown, so neither case is presented before it is known | Screen opens and the contact and its sole-Primary status are being read | Read completes (standard or blocked case) or fails (Load Error) |
| Load Error | Error banner: "Couldn't load this contact. Try again." with a Retry button; no "Remove Contact" action shown, so removal cannot be confirmed without the sole-Primary check | The read of the contact or of the client's Primary contacts fails | Retry succeeds, or Nadia taps the back arrow |
| Standard confirmation | As described in Layout and Content, standard case | Screen opens for a contact that is not currently the client's last Primary | "Remove Contact" or "Cancel" is tapped |
| Blocked (last Primary) | As described in Layout and Content, blocked case | Screen opens for the client's last Primary contact, or the removal attempt's commit-time re-check finds this contact has become the last Primary | Nadia navigates to FEAT-18.SPEC-001 |
| Removing | "Remove Contact" shows a loading spinner, actions disabled | Nadia confirms removal | Removal completes or fails |
| Error | Error banner: "Couldn't remove this contact. Try again." with a Retry button | The removal automation (FEAT-18.SPEC-009) fails for a reason other than the last-Primary block | Retry succeeds |
| Offline/Degraded | N/A -- contact management is an infrequent, connectivity-required action (feature-overview.md, States); a removal attempted without connectivity surfaces the Error state above | -- | -- |

## Validation Rules

Validation governed by FEAT-18.SPEC-006 (Primary Contact Requirement Rule), which defines the last-Primary block enforced by this screen's standard-vs-blocked states. This screen defines no field-level validation of its own -- there is no input to validate, only a confirmation decision.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow / "Cancel" tap | FEAT-18.SPEC-001 (Client Contact List) | -- |
| Successful removal | FEAT-18.SPEC-001 (Client Contact List) | -- |
| "Go to Contacts" tap (blocked case) | FEAT-18.SPEC-001 (Client Contact List) | -- |

## Data Model

**Creates:** None directly -- FEAT-18.SPEC-009 creates the resulting audit trail entry.
**Reads:** Client Contact -- name, email, role, status, for the selected contact; whether the contact is currently the client's sole Primary (derived, for the blocked-case check).
**Updates:** None directly -- FEAT-18.SPEC-009 performs the actual status change and field erasure.
**Deletes:** None directly on this screen -- the personal-detail erasure is performed by FEAT-18.SPEC-009.

## Business Rules

- The last-Primary condition (FEAT-18.SPEC-006, XBR-07) is checked both when this screen loads and again at the moment "Remove Contact" is confirmed, since the client's Primary contacts can change between the two moments.
- Removal always ends access immediately and erases contact details, whether the trigger is a routine departure or an explicit data-subject erasure request (feature-overview.md, Key Capabilities) -- this screen shows the same confirmation framing in both cases, since the product does not distinguish the two at the point of removal.
- Confirmed removal always hands off to FEAT-18.SPEC-009; there is no partial or reversible removal path.

## Edge Cases

- **The contact becomes the client's last Primary between screen load and Nadia tapping "Remove Contact"** -- The commit-time re-check finds the block condition now applies; the screen switches in place to the blocked case with no removal performed. Resolution: reject-with-refresh, per the dependency map's Contention note for the Client Contact entity.
- **Nadia taps "Remove Contact" twice rapidly** -- The second tap is ignored while the first removal is in progress (button in loading state).
- **The contact or the client's Primary contacts fail to load** -- The Load Error state shows "Couldn't load this contact. Try again." with Retry; the standard confirmation is never shown on a guess, because whether the last-Primary block applies is unknown until the read succeeds.
- **Network failure during removal** -- Error banner: "Couldn't remove this contact. Try again." with a Retry button; no partial removal occurs -- the contact's status and details are unchanged until the removal automation completes successfully.
- **The contact being removed is currently signed in to the client portal** -- The removal proceeds; FEAT-18.SPEC-009 ends the contact's access immediately, so any of their subsequent actions in an already-open portal session are rejected per FEAT-05 (Client Portal Access) rather than by this screen.
- **Nadia removes a Reviewer contact (no last-Primary concern)** -- The standard confirmation case always applies to a Reviewer, since the last-Primary block only ever governs a Primary contact.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-001 (Client Contact List) | Navigation (inbound and outbound) | Entry point and return destination |
| FEAT-18.SPEC-006 (Primary Contact Requirement Rule) | References (inbound) | Defines the last-Primary block this screen enforces at load and at commit |
| FEAT-18.SPEC-009 (Contact Removal & Data Erasure) | Triggers (outbound) | Confirmed removal hands off to this automation |
| FEAT-18.SPEC-002 (Add or Edit Client Contact) | Navigation (outbound, via FEAT-18.SPEC-001) | Where Nadia designates a replacement Primary when blocked |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| contact_removal_confirmed | contact_role (primary / reviewer) | Nadia confirms removal and the commit-time check passes | N/A -- no success-metrics.md metric is connected to Client Contact Management & Roles; retained so removal activity is observable |
| contact_removal_blocked_last_primary | -- | The last-Primary block is shown, at load or at commit-time re-check | N/A -- no success-metrics.md metric is connected to this feature |

## Acceptance Criteria

**FEAT-18.SPEC-003-AC-01:** Given Nadia taps "Remove" on a Reviewer contact, when the screen loads, then she sees the standard confirmation naming the contact and explaining that removal ends access and erases their email and other contact details while their approvals stay on record under their name only.

**FEAT-18.SPEC-003-AC-02:** Given Nadia is on the standard confirmation, when she checks "I understand this cannot be undone" and taps "Remove Contact", then FEAT-18.SPEC-009 is triggered and she sees "Contact removed" before returning to FEAT-18.SPEC-001.

**FEAT-18.SPEC-003-AC-03:** Given Nadia taps "Remove" on the client's only Primary contact, when the screen loads, then she sees the blocked case: "{contact_name} is this client's only Primary contact. Add or promote a replacement Primary before removing them." with no "Remove Contact" action.

**FEAT-18.SPEC-003-AC-04:** Given Nadia opens the standard confirmation for a Primary contact who is not the client's last Primary, when she confirms removal, then the removal proceeds normally.

**FEAT-18.SPEC-003-AC-05:** Given Nadia has the standard confirmation open for a Primary contact and another Primary contact is removed from this client in a different session first, when Nadia taps "Remove Contact", then the commit-time re-check finds this is now the last Primary and the screen switches in place to the blocked case with no removal performed.

**FEAT-18.SPEC-003-AC-06:** Given Nadia is on the blocked case, when she taps "Go to Contacts", then she is navigated to FEAT-18.SPEC-001, from which she can add or promote a replacement Primary.

**FEAT-18.SPEC-003-AC-07:** Given Nadia has not checked the acknowledgement checkbox, when she looks at "Remove Contact", then it is disabled and cannot be tapped.

**FEAT-18.SPEC-003-AC-08:** Given Nadia taps "Remove Contact" twice in rapid succession, when the first removal is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-18.SPEC-003-AC-09:** Given a network failure occurs during removal, when the failure is detected, then the error banner "Couldn't remove this contact. Try again." appears with a Retry button, and the contact remains unremoved.

**FEAT-18.SPEC-003-AC-10:** Given Owen, Priya, or Dana attempts to reach this screen through any route, then none exists for any of them.

**FEAT-18.SPEC-003-AC-11:** Given Nadia removes a contact who is currently signed in to the client portal, when the removal completes, then any subsequent action that contact attempts in the client portal is rejected per FEAT-05's access rules.

**FEAT-18.SPEC-003-AC-12:** Given Nadia taps "Remove" on a contact, when the contact and the client's Primary contacts are still being read, then she sees a loading placeholder with no "Remove Contact" action until the standard or blocked case is determined.

**FEAT-18.SPEC-003-AC-13:** Given the read of the contact or the client's Primary contacts fails, when the failure occurs, then Nadia sees "Couldn't load this contact. Try again." with a Retry button and no "Remove Contact" action, and when she taps Retry and the read succeeds, then the standard or blocked case is shown as appropriate.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 7 (loading, load error, standard, blocked, removing, error, offline-degraded N/A) | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Screen Spec: Invite Reviewer Colleague

## Overview

**Name:** Invite Reviewer Colleague
**ID:** FEAT-18.SPEC-004
**Type:** Screen
**Purpose:** Owen, a Client Primary Contact, invites a colleague at his own client company as a Reviewer contact from his portal view.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles

## Scope and Non-Goals

**In Scope:**
- Owen entering a colleague's name and email and sending an invitation
- Restricting the invited role to Reviewer only, with no role picker shown
- Scoping the invite to Owen's own client company only
- Showing Owen's own company's existing contact list for context
- Triggering the invitation email (FEAT-18.SPEC-010) and the counterpart alert to Nadia (FEAT-18.SPEC-011)

**Non-Goals:**
- Inviting or assigning a Primary contact -- excluded per BRIEF.md's Target Users & Roles and the Access Matrix: Owen's invite entitlement is Own-only and Reviewer-only; only Nadia can create or promote a Primary contact (FEAT-18.SPEC-002)
- Editing or removing any existing contact, including Owen's own record -- handled exclusively by Nadia on FEAT-18.SPEC-002 and FEAT-18.SPEC-003; Owen has no edit or remove entitlement anywhere in this feature (Access Matrix: Owen's Client Contact Management is Own-only, scoped to inviting)
- Inviting a colleague at a different client company -- excluded per BRIEF.md's Constraints on strict client isolation and the Access Matrix's Own-only scope; Owen can never see or reach another client company's contacts
- Full contact list management (viewing role changes, removal, status transitions of every contact) -- handled by FEAT-18.SPEC-001, which is Nadia's freelancer-side surface that Owen never reaches

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05.SPEC-003 (Portal Home) | Owen taps "Invite a colleague" | His own client company reference; form starts empty |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | No | No | Nadia never signs in as a client contact and has no route to this screen; her equivalent surface is FEAT-18.SPEC-002 |
| Owen (Client Primary Contact) | Full screen, scoped to his own client company | Invite a Reviewer colleague at his own company | -- |
| Priya (Client Reviewer Contact) | No | No | "Invite a colleague" is not shown on Priya's Portal Home; only a Primary contact can invite (Access Matrix: Priya's Client Contact Management is None) |
| Dana (Support Operator) | No | No | Dana never signs in as a client contact and has no portal access at all (Access Matrix: Client Portal Access -- None; SC-04) |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- entered form data is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Invite a Colleague" with a back arrow (returns to FEAT-05.SPEC-003, Portal Home) and a "Send Invite" action button, right-aligned.

**Body:**
- A brief line of context: "Invite a colleague at {client_company_name} to view and comment on this project. They will not be able to accept proposals, approve milestones, or see invoices."
- Name (text input, required)
- Email (text input, required)
- Role: fixed to "Reviewer," shown as a static label, not a selectable control -- no role picker is shown to Owen
- A collapsible list of his own company's existing contacts (name, role label, status), for context on who already has access

**Footer:** None -- Send Invite is in the header.

### Responsive Behavior

- **Compact size class:** Single-column form as described, full width; the existing-contacts list is collapsed by default with a "Show colleagues" toggle.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; the existing-contacts list is expanded by default.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-05.SPEC-003 (Portal Home) | Screen closes | Standard navigation transition |
| Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Name input | Blur (empty) | Triggers validation via FEAT-18.SPEC-005 | Error state on field | "Name is required" below the field |
| Email input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Email input | Blur | Triggers format and per-client-uniqueness validation via FEAT-18.SPEC-005 | Error state on field if invalid | "Please enter a valid email address" or "This email is already used by another contact at this company" |
| "Show colleagues" toggle (compact only) | Tap | Expands or collapses the existing-contacts list | List visibility toggles | Standard expand/collapse animation |
| "Send Invite" button | Tap | 1. Validate fields via FEAT-18.SPEC-005 (role fixed to Reviewer). 2. If valid, create the contact and trigger FEAT-18.SPEC-010 (invitation email) and FEAT-18.SPEC-011 (alert to Nadia). | Button shows loading state during save | Success: toast "Invitation sent" and navigate to FEAT-05.SPEC-003. Failure: inline field errors, or a form-level error banner for a non-field failure. |
| "Send Invite" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "Retry" button (contact-list load-failure banner) | Tap | Re-fetches Owen's own company's existing contacts | The context list returns to Loading, then Loaded | Skeleton rows while loading; on success the list appears and the banner disappears; on repeated failure the banner reappears |

### Accessibility Notes

- **Focus order:** Back arrow -> context line -> Name -> Email -> Role label (non-interactive) -> "Show colleagues" toggle (when present) -> "Send Invite".
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Invitation sent" toast is announced on success; on validation failure, focus moves to the first field in error.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading (colleague list) | Name and Email fields usable; the existing-contacts list shows skeleton rows; Send Invite disabled until the list has loaded, because the list is what shows Owen any email conflict before he submits | Screen first opens and Owen's own company's contacts are being fetched | Fetch completes (Empty) or fails (List Load Error) |
| List Load Error | Error banner in place of the existing-contacts list: "Couldn't load your colleagues. Try again." with a Retry button; Send Invite disabled | The fetch of Owen's own company's contact list fails | Retry succeeds (Empty), or Owen taps the back arrow |
| Empty (default) | Name and Email empty, Send Invite enabled | Screen first opens and the contact list has loaded | Owen begins typing in either field |
| Filling | Fields contain user input | Owen types in either field | Send Invite is tapped or Owen navigates away |
| Validating | Send Invite shows a loading spinner | Send Invite is tapped | Validation completes (pass or fail) |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails (FEAT-18.SPEC-005) | Owen corrects the field and re-triggers validation |
| Sending | Send Invite shows a loading spinner, fields disabled | Validation passes | Save completes or fails |
| Error | Error banner: "Couldn't send this invitation. Try again." with a Retry button; entered data is preserved | Save operation fails for a reason other than field validation | Retry succeeds |
| Offline/Degraded | N/A -- inviting a colleague is an infrequent, connectivity-required action, consistent with the feature's product-level States definition; a send attempted without connectivity surfaces the Error state above | -- | -- |

## Validation Rules

Validation governed by FEAT-18.SPEC-005 (Contact Field Validation Rules). See that spec for all field-level and cross-field rules, including the rule restricting Owen's assignable role to Reviewer only. This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-05.SPEC-003 (Portal Home) | FEAT-05 |
| Successful invite | FEAT-05.SPEC-003 (Portal Home) | FEAT-05 |
| Cancel with unsaved changes | FEAT-05.SPEC-003 (Portal Home), after a confirmation dialog | FEAT-05 |

## Data Model

**Creates:** Client Contact -- name and email set from form input; role set to Reviewer (fixed, not user-selectable); invited_by set to Owen's own Client Contact record; status set to Invited.
**Reads:** Client Contact -- name, role, status of every existing contact at Owen's own client company, for the context list.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Field validation (FEAT-18.SPEC-005) is enforced -- Owen cannot send an invite with invalid data.
- Owen's invite entitlement is scoped to his own client company only and to the Reviewer role only (FEAT-18.SPEC-007, Role Authorization Rules; XBR-08); this screen never exposes a Primary option or any other client company's contacts.
- Sending a successful invite triggers both FEAT-18.SPEC-010 (invitation email to the new contact) and FEAT-18.SPEC-011 (alert to Nadia) automatically -- Owen cannot send one without the other.
- The invited contact's role change effective-timing (FEAT-18.SPEC-008) does not apply on creation -- it governs a later role change to an existing contact, which Owen cannot perform.

## Edge Cases

- **Owen navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Owen taps Send Invite twice rapidly** -- The second tap is ignored while the first send is in progress (button in loading state).
- **Network failure during send** -- Error banner: "Couldn't send this invitation. Try again." with a Retry button. Form data is preserved.
- **Email entered matches another contact already at Owen's company** -- Rejected per FEAT-18.SPEC-005: "This email is already used by another contact at this company." Send does not proceed.
- **Nadia adds a contact with the same email to this client at the same moment from FEAT-18.SPEC-002** -- Per the dependency map's Contention note, Nadia (Full) and Owen (Own-only) can both add contacts to the same client concurrently; the second save to complete is rejected-with-refresh on the per-client email-uniqueness check (FEAT-18.SPEC-005), regardless of whether Nadia or Owen submitted second.
- **Owen's own company's contact list fails to load** -- The List Load Error state shows "Couldn't load your colleagues. Try again." with Retry, and Send Invite stays disabled, so Owen cannot submit without the list that makes an email conflict visible to him.
- **Owen invites a colleague whose email matches an existing Reviewer contact he has no visibility into elsewhere** -- No such case exists: the context list on this screen already shows every contact at his own company, so the uniqueness conflict is always visible to Owen before he submits, not a surprise at save time.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-003 (Portal Home) | Navigation (inbound and outbound) | Entry point and return destination |
| FEAT-18.SPEC-005 (Contact Field Validation Rules) | References (inbound) | Field-level validation and the Owen-invite role restriction |
| FEAT-18.SPEC-007 (Role Authorization Rules) | References (inbound) | Governs Owen's own-company, Reviewer-only invite entitlement |
| FEAT-18.SPEC-010 (New Contact Invitation Email) | Triggers (outbound) | Fired automatically when the invite is sent |
| FEAT-18.SPEC-011 (Primary-Invited-Colleague Alert) | Triggers (outbound) | Fired automatically when the invite is sent, alerting Nadia |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| contact_invited_by_primary | -- | An invite send completes successfully | N/A -- no success-metrics.md metric is connected to Client Contact Management & Roles; retained so Owen's own-company invite activity is observable |
| contact_invite_failed | reason (validation / network) | A send attempt does not complete | N/A -- no success-metrics.md metric is connected to this feature |

## Acceptance Criteria

**FEAT-18.SPEC-004-AC-01:** Given Owen is on this screen, when he enters "Priya Shah" as name and "priya@acme.test" as email and taps "Send Invite", then a Reviewer contact is created with status Invited, both FEAT-18.SPEC-010 and FEAT-18.SPEC-011 fire, and he sees "Invitation sent" before returning to Portal Home.

**FEAT-18.SPEC-004-AC-02:** Given Owen is on this screen, when he looks for a role selector, then none is shown -- the role is displayed as a fixed "Reviewer" label.

**FEAT-18.SPEC-004-AC-03:** Given Owen taps "Send Invite" with the name field empty, then the name field shows "Name is required" and the send does not proceed.

**FEAT-18.SPEC-004-AC-04:** Given Owen enters an email already used by another contact at his own company, when he blurs the email field, then it shows "This email is already used by another contact at this company."

**FEAT-18.SPEC-004-AC-05:** Given Owen is on this screen, when he expands the existing-contacts list, then he sees every contact currently at his own client company with their role and status, and no contact from any other client company.

**FEAT-18.SPEC-004-AC-06:** Given Owen has unsaved changes on this screen, when he taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?"

**FEAT-18.SPEC-004-AC-07:** Given Owen loses connectivity while sending an invite, then the error banner "Couldn't send this invitation. Try again." appears with a Retry button, and his entered data is preserved.

**FEAT-18.SPEC-004-AC-08:** Given Owen and Nadia both submit a new contact with the same email for the same client at effectively the same time (Owen here, Nadia on FEAT-18.SPEC-002), when the second save is processed, then it is rejected with the per-client email-uniqueness error.

**FEAT-18.SPEC-004-AC-09:** Given Priya is signed in to the client portal, when she looks at Portal Home, then no "Invite a colleague" action is shown to her, and she has no route to this screen.

**FEAT-18.SPEC-004-AC-10:** Given Nadia or Dana attempts to reach this screen through any route, then none exists for either of them -- this screen exists only inside the client portal.

**FEAT-18.SPEC-004-AC-11:** Given Owen taps "Send Invite" twice in rapid succession, when the first send is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-18.SPEC-004-AC-12:** Given Owen opens this screen, when his company's existing contacts are still being fetched, then the list shows skeleton rows and "Send Invite" is disabled until the list has loaded.

**FEAT-18.SPEC-004-AC-13:** Given the fetch of Owen's own company's contact list fails, when the failure occurs, then Owen sees "Couldn't load your colleagues. Try again." with a Retry button and "Send Invite" stays disabled, and when he taps Retry and the fetch succeeds, then the list appears and "Send Invite" is enabled.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 6 (loading, list load error, validation error, sending error, offline-degraded N/A, concurrent-uniqueness conflict) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



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



# Logic/Rule Spec: Role Change Effective-Timing Rule

## Overview

**Name:** Role Change Effective-Timing Rule
**ID:** FEAT-18.SPEC-008
**Type:** Logic/Rule
**Purpose:** Ensures a contact's role change applies to their future actions only, and never alters the record of a past approval or acceptance they gave under their prior role.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles
**Governed Entity:** Client Contact (role field, in relation to timestamped evidentiary records elsewhere in the product)

## Scope and Non-Goals

**In Scope:**
- The rule that a role change takes effect only for actions the contact performs after the change is saved
- The guarantee that a proposal acceptance, milestone approval, or any other evidentiary record already attributed to a contact is never re-evaluated or altered when that contact's role later changes
- How an in-progress action (one the contact started before the change but has not yet completed) is resolved

**Non-Goals:**
- Who may change a role at all -- owned by FEAT-18.SPEC-007 (Role Authorization Rules); this spec governs only the timing of an authorized change's effect
- The last-Primary block that can prevent a role change from being saved in the first place -- owned by FEAT-18.SPEC-006 (Primary Contact Requirement Rule); this spec applies only once a role change has actually been saved
- Field-level validation of the role value itself -- owned by FEAT-18.SPEC-005 (Contact Field Validation Rules)
- Writing the audit-trail entry for the role change -- owned by Immutable Activity & Audit Trail (FEAT-13, XBR-05); this spec defines the behavioral guarantee the trail entry documents, not the trail-writing mechanism itself

## Governed Entity

**Entity:** Client Contact
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| role | enum (Primary, Reviewer) | The field this rule governs the effective timing of when changed |
| invited_by | text (reference) | Unaffected by a role change -- retains the original inviting party regardless of later role changes |
| status | enum (Invited, Active, Removed) | Unaffected by a role change on its own |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-18.SPEC-002 | Add or Edit Client Contact | At the moment a role change is saved -- the new role governs only actions the contact performs from that moment forward |
| FEAT-03 (Proposal Acceptance) | Acceptance recording | Reads the contact's role at the moment of acceptance, not retroactively re-evaluated after a later role change |
| FEAT-08 (Milestone Approval) | Approval recording | Reads the contact's role at the moment of approval, not retroactively re-evaluated after a later role change |
| FEAT-13 (Immutable Activity & Audit Trail) | Trail entry attribution | Every past evidentiary entry keeps the role-at-the-time context it was written with, per ASMP-25's record-immutability guarantee |

## Field Validation Rules

This spec governs the timing of an already-valid role change's effect, not the role value's format. No field validation rules apply here beyond FEAT-18.SPEC-005.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| role | No validation beyond data type at this spec's layer -- see FEAT-18.SPEC-005 for format and FEAT-18.SPEC-006 for the last-Primary condition | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Future-only effect | role, and every timestamped evidentiary field elsewhere (Proposal.accepted_by, Milestone.approved_by) | A role change's saved timestamp establishes a boundary: any action the contact performs after that timestamp is governed by the new role; any action already recorded before that timestamp remains governed by, and attributed under, the role in effect when it was recorded | N/A -- this is a system-enforced timing guarantee with no user-facing violation state; there is no way for a user to request retroactive re-attribution |

## Authorization Rules

This rule applies automatically to every authorized role change; it grants or denies no action of its own. The single row below documents that this spec's guarantee is unconditional once a change is authorized and saved.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Apply future-only timing to a saved role change | The system, automatically, for every role change Nadia saves | Always -- unconditional, no role or condition can opt out of it | N/A -- there is no path to disable or bypass this guarantee; it is not a user-facing permission |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Effective role for a given past action | Derived as the role value in effect on the Client Contact record at the exact timestamp that action's own record (Proposal.accepted_at, Milestone.approved_at) was written | Whenever a past action's role context is displayed or referenced (e.g., in the activity trail) | No -- always derived from history, never recalculated against the contact's current role |
| Effective role for a new action | The Client Contact's current role field, read fresh at the moment the new action begins | Every time a contact attempts an action gated by role (accept, approve, invite, etc.) | No -- always the current value, never a cached value from earlier in the session |

## Business Rules

- A role change never alters the record of a past approval or acceptance (feature-overview.md, Primary Flows & Alternates: "the change applies to future actions only, never retroactively altering a past approval"); this is a direct extension of record immutability (ASMP-25, XBR-04) applied specifically to role context.
- The scenario this rule exists to serve: a second Primary is added for co-founders, or a Reviewer is promoted to Primary; from that moment they can accept, approve, and pay, but everything they did as a Reviewer beforehand (comments, prior views) remains attributed to them as it was recorded, with no reinterpretation.
- Demoting a Primary to Reviewer (subject to FEAT-18.SPEC-006's last-Primary block) works identically in the other direction: any acceptance or approval that contact gave while Primary remains valid evidence under their name, even though they can no longer perform such actions going forward.
- This rule has no bearing on whether a role change itself is permitted -- that is entirely FEAT-18.SPEC-006 and FEAT-18.SPEC-007's domain; this spec only governs what happens to time once a change is saved.

## Edge Cases

- **A contact begins accepting a proposal (opens the accept screen) and their role is changed to Reviewer by Nadia before they tap Accept** -- The accept action, not yet completed, is evaluated against the contact's role at the moment of the action's completion (tapping Accept), not at the moment they opened the screen; since Reviewer contacts cannot accept, the action is now denied per FEAT-18.SPEC-007, and the client-facing screen shows the standard access-denied experience for that action.
- **A Reviewer is promoted to Primary, and immediately attempts to accept a proposal that was sent before the promotion** -- The action is permitted: the proposal's acceptance was never attempted under the old role, so there is no past record to protect, and the current role at the moment of acceptance is Primary.
- **Nadia changes a contact's role twice in quick succession (Primary to Reviewer, then back to Primary)** -- Each change is its own timestamped event; any action attempted between the two changes is governed by whichever role was in effect at that exact moment, and both changes are independently visible in the activity trail (FEAT-13).
- **A role change is saved while the contact has a comment (FEAT-07) already posted under their prior role** -- The comment's author attribution is unaffected; comments carry no role-gated content restriction beyond what governed posting it at the time, and a Reviewer's existing comments remain visible exactly as posted regardless of a later promotion or demotion.
- **The client's sole Primary contact is demoted the instant after an invoice's pay link is generated in their name** -- Not retroactively affected: the pay link and invoice remain addressed to that contact as recorded; whether they can still access the client portal at all going forward is a separate concern governed by FEAT-05, unaffected by this rule (this rule concerns role-gated entitlement, not portal access itself, which persists for a Reviewer-demoted former Primary).
- **A role change is attempted concurrently with the very action it would affect (e.g., Nadia demotes Owen at the same instant Owen taps Approve on a milestone)** -- Whichever write commits first determines the outcome: if the demotion commits first, Owen's approval attempt is evaluated as a Reviewer and denied; if the approval commits first, it is recorded under Primary and the demotion that follows does not undo it. There is no scenario where both are recorded inconsistently, since each is a single atomic commit evaluated against the role value at its own commit time.

## Acceptance Criteria

**FEAT-18.SPEC-008-AC-01:** Given Owen accepted a proposal as a Primary contact, when Nadia later changes his role to Reviewer, then the prior acceptance record remains unchanged and still shows Owen as the accepting party.

**FEAT-18.SPEC-008-AC-02:** Given Priya is promoted from Reviewer to Primary, when she subsequently attempts to accept a proposal, then the acceptance is permitted, since her current role at the moment of acceptance is Primary.

**FEAT-18.SPEC-008-AC-03:** Given a contact's role is changed from Primary to Reviewer, when that contact attempts to approve a milestone after the change, then the approval is denied, per their new current role.

**FEAT-18.SPEC-008-AC-04:** Given Owen has the proposal-accept screen open and Nadia changes his role to Reviewer before he taps Accept, when he taps Accept, then the action is evaluated against his role at that moment (Reviewer) and is denied.

**FEAT-18.SPEC-008-AC-05:** Given Nadia changes a contact's role from Primary to Reviewer and back to Primary within the same day, when the activity trail is viewed, then both role-change events appear as separate, independently timestamped entries.

**FEAT-18.SPEC-008-AC-06:** Given a Reviewer contact posted comments before being promoted to Primary, when their comment history is viewed after the promotion, then those comments remain visible and attributed to them exactly as posted, unaffected by the later promotion.

**FEAT-18.SPEC-008-AC-07:** Given a client's sole Primary contact is demoted to Reviewer immediately after an invoice's pay link was generated in their name, when the invoice is later viewed, then the pay link and invoice remain addressed to that contact as originally recorded.

**FEAT-18.SPEC-008-AC-08:** Given Owen taps Approve on a milestone at the same instant Nadia's demotion of him commits first, when the approval attempt is then evaluated, then it is denied under his now-current Reviewer role.

**FEAT-18.SPEC-008-AC-09:** Given Owen's approval commits before Nadia's concurrent demotion of him, when the demotion is applied afterward, then the already-recorded approval is not undone or altered.

**FEAT-18.SPEC-008-AC-10:** Given a contact's role has never changed since creation, when any of their past or future actions are evaluated, then this rule has no observable effect, since there is only ever one role value in their history.

**FEAT-18.SPEC-008-AC-11:** Given Nadia views a past milestone approval in the activity trail for a contact whose role has since changed, when she reads the entry, then it shows the role the contact held at the time of that approval, not their current role.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 1 | 1 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Automation Spec: Contact Removal & Data Erasure

## Overview

**Name:** Contact Removal & Data Erasure
**ID:** FEAT-18.SPEC-009
**Type:** Automation
**Purpose:** Ends a removed contact's access immediately, erases their email address and other contact details, and retains their evidentiary records under their name only.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles

## Scope and Non-Goals

**In Scope:**
- Setting the contact's status to Removed and erasing their email address and every other contact detail on the record, while the retained row keeps the contact's name as the actor label for evidence
- Ending the contact's ability to sign in or act in the client portal, immediately
- Preserving every evidentiary record (acceptances, approvals, comments, payments) the contact created, unaltered, still referencing the retained Client Contact row and displaying the contact's name only
- Writing the removal event to the activity trail

**Non-Goals:**
- The confirmation dialog and last-Primary block shown before this automation fires -- owned by FEAT-18.SPEC-003 (Remove Client Contact) and FEAT-18.SPEC-006 (Primary Contact Requirement Rule); this automation begins only once a removal has already been confirmed and cleared
- Restoring a removed contact -- excluded per the feature's Entity-Lifecycle Coverage Matrix: because this automation's erasure is the mechanism that satisfies the erasure guarantee (XBR-27), the original personal details are genuinely gone and cannot be restored; a returning contact is entered as a brand-new Client Contact record
- Purging the contact's evidentiary records (acceptances, approvals) -- excluded per scope-boundaries.md (SC-24) and ASMP-25's record-immutability guarantee; those records are retained permanently under the account's own retention terms, with no automatic purge window except where FEAT-24 account deletion applies
- Notifying the removed contact that they were removed -- product-features.md's Communications field for this feature names only the invitation email (FEAT-18.SPEC-010) and the counterpart alert (FEAT-18.SPEC-011); no removal notice to the removed party is defined, consistent with the immediate access-revocation intent

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Removal confirmed | FEAT-18.SPEC-003 (Remove Client Contact) | Fires once Nadia confirms removal and the commit-time last-Primary re-check (FEAT-18.SPEC-006) passes | Client Contact reference, its current name, email, role, and status |

## Processing Logic

1. Receive the confirmed removal request for one Client Contact record from FEAT-18.SPEC-003, after the last-Primary re-check has already passed.
2. Re-verify the contact's current status is not already Removed (guards against a duplicate trigger; see Edge Cases).
3. Set the contact's status to Removed.
4. Erase the contact's email field and every other contact detail on the record (for example last sign-in and any sign-in identifiers) -- overwrite each with the product's standard erased-field representation, so no trace of the original values remains. The name field is the one field deliberately kept: it is the label under which the contact's evidence stays attributed (XBR-27).
5. Leave every reference to this contact -- as the actor on their own past acceptances (FEAT-03), approvals (FEAT-08), comments (FEAT-07), and payments (FEAT-10) -- unchanged. One model applies: those evidentiary records reference the retained Client Contact row (status Removed) and show the contact's name only. Their email address is not shown anywhere in retained evidence, because it no longer exists on the row and was never copied into it.
6. Immediately revoke the contact's ability to sign in or act: any currently open client-portal session for this contact is ended, and any future magic-link sign-in attempt for the erased email no longer resolves to a recognized contact (FEAT-05, XBR-28).
7. Write an append-only Activity Log Entry recording the removal event, referencing the retained Client Contact row and carrying the contact's name only -- never the email address (FEAT-13, XBR-05). The trail write does not gate the outcome; see Edge Cases.
8. Return control to FEAT-18.SPEC-003 with the completed outcome.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Removal completed | The contact was not already Removed, and the last-Primary re-check already passed before this automation fired | status set to Removed; email and other contact details erased (name retained as evidence label); access revoked immediately; one Activity Log Entry queued for write (retried independently if it fails) | Nadia sees "Contact removed" and returns to FEAT-18.SPEC-001, where the contact no longer appears | FEAT-18.SPEC-001, FEAT-18.SPEC-003, FEAT-05, FEAT-13 |
| Already removed (idempotent no-op) | The contact's status was already Removed when this automation received the trigger (e.g., a rapid duplicate submission) | None -- no further change is made | Nadia sees the same "Contact removed" success outcome, since the requested end state already holds | FEAT-18.SPEC-001, FEAT-18.SPEC-003 |
| Removal failure | The automation cannot complete the status change or field erasure for a processing reason | No partial change is left in place -- the contact's status, name, and email remain exactly as they were before the attempt (a trail-write failure alone is not a removal failure; see Edge Cases) | Nadia sees the error state on FEAT-18.SPEC-003: "Couldn't remove this contact. Try again." with a Retry button | FEAT-18.SPEC-003 |

## Data Model

**Reads:** Client Contact -- current status, name, email, role, for the contact being removed.
**Creates:** Activity Log Entry -- event_type "contact removed," actor (Nadia), occurred_at, affected_record (the Client Contact), with the contact's name only; no email address is captured (FEAT-13 responsibility, referenced here as the trail-write outcome).
**Updates:** Client Contact -- status set to Removed; email and all other contact details erased; name retained.
**Deletes:** None -- this automation erases specific fields on the existing record and never deletes the Client Contact row itself, since the row must remain referenceable as the actor on past evidentiary records (dependency map, Client Contact -- Delete/Archive: "status set to Removed rather than the row being physically dropped").

## Business Rules

- XBR-27: a contact's erasure request ends their access immediately and removes their contact details, while acceptances and approvals they gave remain on the record under their name. Only the name persists in retained evidence and in the trail entry; no email or other contact detail persists anywhere (feature-dependency-map.md, XBR-27 Authority: FEAT-18).
- Erasure applies identically whether the removal's trigger was a routine client-side departure or an explicit data-subject erasure request (feature-overview.md, Key Capabilities); this automation makes no distinction between the two once FEAT-18.SPEC-003 hands off a confirmed removal.
- Access revocation and field erasure happen together, in the same automation run -- there is no intermediate state where access is revoked but personal details remain, or vice versa.
- No cascade to the Client or Project records -- removing a contact never affects the client company's own record, its projects, or any other contact at that company (dependency map, Client Contact -- Delete/Archive: "No cascade to Client or Project").
- This automation is non-reversible by design: it has no undo path, consistent with the product's decision that a removed contact is re-added as a new record rather than restored.

## Edge Cases

- **The contact is currently signed in to the client portal when this automation runs** -- Their open session is ended immediately; any in-flight action they attempt after this automation completes is rejected by FEAT-05's access rules, since the contact record they authenticated against no longer resolves to a recognized, non-Removed contact.
- **The contact's email bounces or the erased email is later reused by a different person entirely** -- No conflict: an erased email is available for reuse in the per-client uniqueness check (FEAT-18.SPEC-005), since the Removed contact's email field no longer holds a live value to collide with.
- **A concurrent read of this contact's row is in progress on FEAT-18.SPEC-001 when this automation commits** -- The list screen is a snapshot (per FEAT-18.SPEC-001); the removed contact continues to display until the list is next reloaded, at which point it no longer appears, since the underlying record's status is now Removed.
- **Concurrent trigger firing (two removal confirmations for the same contact submitted in quick succession, e.g., a rapid duplicate tap that reached FEAT-18.SPEC-003 twice)** -- The first to commit produces the Removal completed outcome; the second finds the contact already Removed and produces the idempotent Already removed outcome, with no duplicate Activity Log Entry written and no error shown to Nadia.
- **A trigger fires while a previous run for the same contact is still in flight** -- FEAT-18.SPEC-003's Save-equivalent action is disabled while a removal is in progress (mirroring the pattern used for saves elsewhere in this feature), so a second run for the same record cannot start until the first completes; runs for different contacts proceed independently.
- **This automation's Activity Log Entry write fails even though the status change and erasure succeeded** -- Per XBR-05 and FEAT-13's own write-retry behavior, the trail write is retried independently of this automation's own outcome; the removal itself is not rolled back or held pending the trail write, since access revocation must never wait on a secondary record. This is not a Removal failure outcome: Nadia sees success.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-003 (Remove Client Contact) | Triggered by (inbound) | Confirmed removal fires this automation |
| FEAT-18.SPEC-006 (Primary Contact Requirement Rule) | References (inbound) | The last-Primary re-check that must already have passed before this automation is triggered |
| FEAT-18.SPEC-001 (Client Contact List) | Affects (outbound) | The removed contact no longer appears once this automation completes |
| FEAT-05 (Client Portal Access) | Affects (outbound) | Access revocation ends any open session and blocks future sign-in for the erased contact |
| FEAT-13 (Immutable Activity & Audit Trail) | Affects (outbound) | Writes the append-only removal entry (contact name only) |
| FEAT-03 (Proposal Acceptance), FEAT-08 (Milestone Approval), FEAT-07 (Deliverable Review & Feedback), FEAT-10 (Invoice Payment Processing) | References (outbound) | Each feature's own evidentiary records continue to reference the retained contact row and display the contact's name only, unaffected by this automation |
| FEAT-24 (Data Export & Account Deletion) | References (outbound) | Full account deletion removes remaining client contacts' personal data through its own process, subject to the same evidence-retention boundary this automation establishes |

## Analytics and Success Signals

- **contact_removed** (role: primary / reviewer, trigger_context: routine / erasure_request) -- N/A -- no success-metrics.md metric is connected to Client Contact Management & Roles; retained so removal volume and its erasure-versus-routine mix are observable
- **contact_removal_idempotent_noop** (-- ) -- N/A -- no success-metrics.md metric is connected to this feature; retained so duplicate-submission handling is observable rather than silent
- **contact_removal_failed** (reason: processing_error) -- N/A -- no success-metrics.md metric is connected to this feature

## Acceptance Criteria

**FEAT-18.SPEC-009-AC-01:** Given Nadia confirms removing a Reviewer contact and the trigger fires, when this automation runs, then the contact's status is set to Removed, their email and other contact details are erased while their name is retained, and an Activity Log Entry carrying the name only is written.

**FEAT-18.SPEC-009-AC-02:** Given the removed contact previously accepted a proposal, when their acceptance record is viewed afterward, then it still shows their name as the accepting party via the retained Client Contact row, shows no email address for them, and is otherwise unaffected by the erasure.

**FEAT-18.SPEC-009-AC-03:** Given the removed contact is currently signed in to the client portal when this automation runs, when they next attempt any action, then it is rejected because their contact record no longer resolves as recognized.

**FEAT-18.SPEC-009-AC-04:** Given this automation completes successfully, when FEAT-18.SPEC-001 is next reloaded, then the removed contact no longer appears in the list.

**FEAT-18.SPEC-009-AC-05:** Given the removed contact's erased email is later entered for a new contact at the same client, when the uniqueness check runs (FEAT-18.SPEC-005), then no conflict is found.

**FEAT-18.SPEC-009-AC-06:** Given a removal is triggered a second time for a contact already Removed (a duplicate rapid submission), when this automation receives the second trigger, then it makes no further change and produces the same success outcome shown for the first.

**FEAT-18.SPEC-009-AC-07:** Given this automation cannot complete the status change or field erasure for a processing reason, when the failure occurs, then no partial change is left in place, and Nadia sees "Couldn't remove this contact. Try again." with a Retry button.

**FEAT-18.SPEC-009-AC-08:** Given a client's other contacts and its own Client and Project records, when a contact is removed, then none of those other records are affected.

**FEAT-18.SPEC-009-AC-09:** Given the removed contact left comments on a deliverable before removal, when those comments are viewed afterward, then they remain visible and attributed to the contact's name, with no email address shown.

**FEAT-18.SPEC-009-AC-10:** Given the removal's Activity Log Entry write fails independently of the status change and erasure succeeding, when this occurs, then the removal itself is not rolled back, Nadia still sees "Contact removed", and the trail write is retried per FEAT-13's own retry behavior.

**FEAT-18.SPEC-009-AC-11:** Given two removal confirmations for the same contact fire in quick succession, when both reach this automation, then the first produces the Removal completed outcome and the second produces the idempotent Already removed outcome, with no duplicate trail entry.

**FEAT-18.SPEC-009-AC-12:** Given a removal is triggered by an explicit data-subject erasure request rather than a routine departure, when this automation runs, then it behaves identically to the routine case -- erasing the email and other contact details and preserving evidentiary records the same way.

**FEAT-18.SPEC-009-AC-13:** Given a removal is in progress for one contact, when a second, unrelated removal is triggered for a different contact at the same client at the same time, then both proceed independently without queuing behind each other.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 (completed, idempotent no-op, failure -- trail-write failure handled as an independent retry edge case) | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Notification Spec: New Contact Invitation Email

## Overview

**Name:** New Contact Invitation Email
**ID:** FEAT-18.SPEC-010
**Type:** Notification
**Purpose:** Sends a newly added or invited contact their first magic-link sign-in invitation, so they can reach the client portal for the first time.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles

## Scope and Non-Goals

**In Scope:**
- The single email sent the moment a new Client Contact is created, whether by Nadia (any role) or by a Primary contact inviting a Reviewer colleague
- Welcoming content that differs slightly by who added the contact and which role they were assigned
- Carrying the contact's first single-use sign-in link
- Retry and expiry behavior tied to the underlying token's own time limit

**Non-Goals:**
- Any later sign-in link the same contact requests after this first one -- owned by FEAT-05.SPEC-008 (Magic Link Sign-In Email), the standing notification for every subsequent request; this spec covers only the first-invitation moment tied to contact creation
- Determining token issuance and validity mechanics themselves -- owned by FEAT-05 (Magic Link Issuance, FEAT-05.SPEC-004; Link Validity & Recognition Rules, FEAT-05.SPEC-006). This spec sends the token that capability issues for a newly created contact and flags, for cross-feature reconciliation, that a first-use token for a contact whose status is Invited (not yet Active) must be recognized as valid by FEAT-05.SPEC-006, since FEAT-05 sets a contact's status to Active only upon their first successful sign-in -- the very action this email exists to enable.
- Alerting Nadia that a Primary contact invited someone -- owned by FEAT-18.SPEC-011 (Primary-Invited-Colleague Alert), a separate notification to a separate recipient
- A standalone Integration spec for the underlying email-sending capability -- the transactional email capability this notification relies on is owned by FEAT-14 (External Touchpoints table) and recorded here only as a cross-feature touchpoint, consistent with how FEAT-05.SPEC-008 uses the same capability

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, on every successful contact creation (FEAT-18.SPEC-002 or FEAT-18.SPEC-004) | The recipient has never signed in and has no in-app surface to reach; email is the only channel that can deliver a first credential to someone the product has never seen sign in before |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| New contact saved by Nadia | FEAT-18.SPEC-002 (Add or Edit Client Contact) | Fires once, immediately after a create (not an edit) save completes | New contact's name, email, assigned role, invited_by (Nadia), a freshly issued first sign-in token, the owning freelancer's Branding Profile |
| New contact invited by a Primary contact | FEAT-18.SPEC-004 (Invite Reviewer Colleague) | Fires once, immediately after Owen's invite send completes | New contact's name, email, role (always Reviewer), invited_by (the inviting Primary contact's name), a freshly issued first sign-in token, the owning freelancer's Branding Profile |

## Audience and Preferences

**Recipients:** The single, newly created Client Contact (a future Owen or Priya, per the Access Matrix, depending on the role assigned). No other role ever receives this email -- it is addressed to the one person whose first access it enables.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| None | N/A | N/A | N/A |

This email carries a new contact's first-ever sign-in credential; per XBR-30, transactional emails core to the record always send. There is no preference screen for a client contact to manage before they have even signed in once, consistent with the Access Matrix (client contacts have no Settings surface of their own).

**Quiet Hours:** N/A -- this email carries a time-limited credential the recipient needs to reach the portal at all; holding it for a quiet-hours window would shrink the usable portion of the token's own time limit, the same reasoning FEAT-05.SPEC-008 applies to every subsequent sign-in email.

## Content Definition

**Email (added directly by the freelancer):**
- **Subject:** You've been added to {freelancer_business_name}'s Clientroom portal
- **Body:**
  Hi {contact_name},

  {inviting_party_name} has added you as a {role_label} contact for {freelancer_business_name} on Clientroom. {role_description}

  Click below to sign in for the first time. This link is single-use and expires soon, so use it right away.
- **CTA (button):** Get started -- deep-links to FEAT-05.SPEC-002 (Link Verification Landing) with the issued token, which then routes to FEAT-05.SPEC-003 (Portal Home) on success

**Email (invited by a Primary contact, always Reviewer):**
- **Subject:** {inviting_party_name} invited you to {freelancer_business_name}'s Clientroom portal
- **Body:**
  Hi {contact_name},

  {inviting_party_name} has invited you as a Reviewer contact for {freelancer_business_name} on Clientroom. You'll be able to view and comment on shared work, but not accept proposals, approve milestones, or see invoices.

  Click below to sign in for the first time. This link is single-use and expires soon, so use it right away.
- **CTA (button):** Get started -- deep-links to FEAT-05.SPEC-002 (Link Verification Landing) with the issued token, which then routes to FEAT-05.SPEC-003 (Portal Home) on success

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|----------------|------------------------|
| {freelancer_business_name} | Freelancer Account -- business_name (or, before that is set, the freelancer's own name) | Nadia Ross Design | Renders as "the Clientroom portal" (subject becomes "You've been added to the Clientroom portal") -- required before the first invoice per the dependency map, but this email can fire earlier, so the fallback covers that window |
| {contact_name} | Client Contact -- name | Priya Shah | Greeting renders as "Hi there," |
| {inviting_party_name} | The acting Client Contact's own name (Owen, for an invite) or the Freelancer Account's name (Nadia, for a direct add) | Nadia Ross / Owen Carter | Never empty -- a contact is always created by exactly one identifiable acting party (FEAT-18.SPEC-005, invited_by is always set) |
| {role_label} | Client Contact -- role, rendered as "Primary" or "Reviewer" | Primary | Never empty -- role is a required field at creation (FEAT-18.SPEC-005) |
| {role_description} | Derived from role -- Primary renders "As a Primary contact, you can accept proposals, approve milestones, and pay invoices." Reviewer renders "As a Reviewer contact, you can view and comment on shared work." | -- | Never empty -- exactly one of the two fixed descriptions always applies |

## Delivery Rules

**Batching:** None -- each contact creation produces exactly one invitation email for exactly one new contact; if the same person is added to two different clients (or by two different freelancers), each creation is an independent event that produces its own separate email.
**Deduplication:** At most one invitation email per Client Contact record -- this notification fires exactly once, at creation, and a contact is never created twice for the same event (FEAT-18.SPEC-005's per-client uniqueness rule prevents a duplicate create for the same email at the same client).
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). The retry window is chosen to fit inside platform parameter: `magic-link-expiry-window`, so a delayed-but-successful delivery still carries a usable link, consistent with FEAT-05.SPEC-008's identical reasoning.
**Expiry:** If delivery has not succeeded after the final retry, no further attempt is made and the delivery failure is surfaced to the freelancer as a warning (XBR-30) on the client's contact record, since the new contact has no other way to be reached and would otherwise never learn they were given access at all.

## Edge Cases

- **The contact is removed before this email is delivered** -- The pending delivery is cancelled: FEAT-18.SPEC-009's immediate access revocation means the token this email would carry is no longer usable, so sending it afterward would only confuse the recipient with a dead link.
- **The same person is added as a contact to two different clients of the same freelancer at once** -- Each creation is a separate Client Contact record and produces its own separate invitation email, each with its own token scoped to that specific client relationship.
- **The inviting Primary contact's own name changes after sending the invite but before this email is delivered (unlikely, since Owen cannot edit his own record, but possible if Nadia edits it)** -- The email renders {inviting_party_name} using the value captured at trigger time, not a value re-read at delivery time, so the invite reads consistently with what was true when it was sent.
- **The token expires before the email is delivered (extreme delivery delay)** -- The delivered email's "Get started" link shows "This link isn't valid anymore" when clicked (FEAT-05.SPEC-002); this is treated as a delivery-timing rarity, since platform parameter: `magic-link-expiry-window` is set wide enough that a normal delivery, including one retry cycle, comfortably completes within it.
- **A recipient who is already a Client Contact for a different freelancer receives this email** -- No merge or cross-reference occurs; this email and its token concern only the newly created Client Contact record for this freelancer, exactly as FEAT-05.SPEC-008 treats a person who is a contact for more than one freelancer as entirely separate relationships.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-002 (Add or Edit Client Contact) | Triggered by (inbound) | Every successful create save fires this notification |
| FEAT-18.SPEC-004 (Invite Reviewer Colleague) | Triggered by (inbound) | Every successful invite send fires this notification |
| FEAT-18.SPEC-011 (Primary-Invited-Colleague Alert) | References (inbound) | Fires alongside this notification when the source is FEAT-18.SPEC-004, alerting Nadia separately |
| FEAT-05.SPEC-004 (Magic Link Issuance) | References (inbound) | Supplies the first token this email carries (with the cross-feature status note above) |
| FEAT-05.SPEC-006 (Link Validity & Recognition Rules) | References (inbound) | Governs the token's expiry window this email's retry rule is bounded by |
| FEAT-05.SPEC-002 (Link Verification Landing) | Navigation (outbound) | The "Get started" CTA deep-links here with the issued token |
| FEAT-19 (Freelancer Branding) | References (inbound) | Supplies the logo and brand colour applied to this email |
| FEAT-33 (Portal Referral Attribution) | References (inbound) | Supplies the referral mark shown alongside branding |
| FEAT-14 (Notifications (Email)) | References (inbound) | Owns the underlying Transactional Email Delivery capability (FEAT-14.SPEC-001) this notification is sent through |

## Analytics and Success Signals

- **invitation_email_sent** (added_by: freelancer / primary_contact, role_assigned: primary / reviewer) -- N/A -- no success-metrics.md metric is connected to Client Contact Management & Roles; retained so first-invitation delivery volume is observable
- **invitation_email_delivery_failed** (retry_count_exhausted: true) -- N/A -- no success-metrics.md metric is connected to this feature; retained so a lost first invitation, which would otherwise leave a new contact silently unable to reach the portal, is surfaced rather than invisible

## Acceptance Criteria

**FEAT-18.SPEC-010-AC-01:** Given Nadia adds a new Primary contact directly, when the save completes, then that contact receives an email with subject "You've been added to {freelancer_business_name}'s Clientroom portal" describing their Primary entitlements and a "Get started" link.

**FEAT-18.SPEC-010-AC-02:** Given Owen invites a Reviewer colleague, when the invite send completes, then the new contact receives an email with subject "{Owen's name} invited you to {freelancer_business_name}'s Clientroom portal" describing Reviewer entitlements and a "Get started" link.

**FEAT-18.SPEC-010-AC-03:** Given the new contact taps "Get started" in the email, when the link opens, then they land on FEAT-05.SPEC-002 with their issued token.

**FEAT-18.SPEC-010-AC-04:** Given the freelancer's `business_name` is not yet set, when this email is composed, then the subject and body render with "the Clientroom portal" in place of the business name.

**FEAT-18.SPEC-010-AC-05:** Given a contact is removed before this email is delivered, when the delivery would otherwise occur, then it is cancelled and no email is sent.

**FEAT-18.SPEC-010-AC-06:** Given delivery of this email fails on the first attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before giving up.

**FEAT-18.SPEC-010-AC-07:** Given all retries for this email are exhausted, when the final attempt fails, then Nadia sees a delivery warning on the client's contact record.

**FEAT-18.SPEC-010-AC-08:** Given this notification has no preference control for the recipient, when a new contact is created, then the email always sends -- there is no opt-out surface to check.

**FEAT-18.SPEC-010-AC-09:** Given a new contact is created at any hour, when this notification is triggered, then it sends immediately with no quiet-hours hold, since the underlying credential is time-limited.

**FEAT-18.SPEC-010-AC-10:** Given the freelancer's Branding Profile has a logo and brand colour set, when this email renders, then it displays that branding alongside the "Made with Clientroom" referral mark.

**FEAT-18.SPEC-010-AC-11:** Given the same person is added as a contact to two different clients of the same freelancer, when both creations complete, then each produces its own separate email with its own token scoped to that specific client.

**FEAT-18.SPEC-010-AC-12:** Given the token this email carries expires before the email is delivered due to an extreme delivery delay, when the recipient clicks "Get started", then they see "This link isn't valid anymore" and can request a fresh one from FEAT-05.SPEC-002.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 2 (added by freelancer, invited by Primary contact) | 2 |
| Preference States | 1 (no preference -- always sends) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Primary-Invited-Colleague Alert

## Overview

**Name:** Primary-Invited-Colleague Alert
**ID:** FEAT-18.SPEC-011
**Type:** Notification
**Purpose:** Alerts Nadia by email whenever a Primary contact invites a colleague, so she always knows who can see her work.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles

## Scope and Non-Goals

**In Scope:**
- The single email delivered to Nadia every time a Primary contact successfully invites a Reviewer colleague
- Content naming who invited whom and at which client
- Retry and expiry behavior for this counterpart-visibility alert

**Non-Goals:**
- Alerting Nadia when she herself adds a contact -- this notification exists specifically for the counterpart-visibility case (feature-overview.md, Communications: "so she always knows who can see her work"); an action Nadia performed herself needs no alert about itself
- The invitation email to the new contact -- owned by FEAT-18.SPEC-010 (New Contact Invitation Email), a separate notification to a separate recipient, fired by the same trigger
- Giving Nadia any control over whether the invited colleague's invitation proceeds -- this is a notice-only alert; product-features.md's Primary Flows & Alternates describes no approval step between Owen's invite and its completion, so this email is informational, not a gate
- An in-app notification-center entry for this alert -- excluded because In-App Notification Center (FEAT-29) is a Later-phase, Nice-to-Have feature (scope-boundaries.md); this MVP-phase alert is email-only, consistent with product-features.md's Communications field naming only an email

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, on every successful Primary-contact invite (FEAT-18.SPEC-004) | Nadia is not necessarily inside the product at the moment a client-side invite happens, and email is the product's sole channel for reaching her about events that originate on the client side (FEAT-14, Notification Delivery Reliability) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A Primary contact invites a Reviewer colleague | FEAT-18.SPEC-004 (Invite Reviewer Colleague) | Fires once, immediately after the invite send completes successfully | Inviting Primary contact's name, new contact's name and email, the client company, project context if applicable |

## Audience and Preferences

**Recipients:** Nadia -- the sole freelancer-side persona per the Access Matrix. This alert is never sent to any client contact; it exists solely so Nadia, the account owner, learns who can now see her work.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| None | N/A | N/A | N/A |

This alert reports a change to who can access Nadia's own client-facing work -- an access-relevant event the product does not treat as an optional, switchable notification (parallel to how a support-session-opened notice is always sent, per XBR-29). There is no preference screen exposing an on/off control for it, since product-features.md's Communications field states it plainly as something Nadia is always told, with no conditional framing.

**Quiet Hours:** N/A -- the product defines no quiet-hours window for any freelancer-side account-activity alert (no such window is named in product-features.md's Notifications & Help field or assumptions-constraints.md); this alert sends immediately, consistent with every other freelancer-facing account notice in this feature set.

## Content Definition

**Email:**
- **Subject:** {inviting_primary_name} invited a colleague to {client_company_name}
- **Body:**
  Hi {freelancer_name},

  {inviting_primary_name} invited {new_contact_name} ({new_contact_email}) as a Reviewer contact for {client_company_name}. They can view and comment on shared work, but cannot accept proposals, approve milestones, see invoices, or invite anyone else.

  You can review or manage this client's contacts and roles at any time.
- **CTA (button):** View contacts -- deep-links to FEAT-18.SPEC-001 (Client Contact List) for {client_company_name}

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|----------------|------------------------|
| {freelancer_name} | Freelancer Account -- name | Nadia | Greeting renders as "Hi there," |
| {inviting_primary_name} | Client Contact (the Primary contact who invited) -- name | Owen Carter | Never empty -- FEAT-18.SPEC-004 requires an authenticated Primary contact to trigger the invite; there is no anonymous-invite path |
| {new_contact_name} | Client Contact (the newly invited Reviewer) -- name | Priya Shah | Never empty -- name is a required field at creation (FEAT-18.SPEC-005) |
| {new_contact_email} | Client Contact (the newly invited Reviewer) -- email | priya@acme.test | Never empty -- email is a required field at creation (FEAT-18.SPEC-005) |
| {client_company_name} | Client -- client_name | Acme Co. | Never empty -- client_name is a required field (feature-dependency-map.md, Client entity) |

## Delivery Rules

**Batching:** None -- each successful invite produces exactly one alert to Nadia; two colleagues invited by the same or different Primary contacts at the same client in quick succession each produce their own separate alert, since each is an independently meaningful access-visibility event Nadia should see individually.
**Deduplication:** At most one alert per successful invite -- this notification fires exactly once, from the same trigger instance as FEAT-18.SPEC-010, and an invite send is not retried as a whole once it has succeeded (a failed invite attempt that Owen retries and resubmits produces its own new, separate successful-invite event, and therefore its own single alert).
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001), the same retry policy applied to every transactional email in this feature.
**Expiry:** If delivery has not succeeded after the final retry, no further attempt is made; unlike a time-limited credential email, there is no shrinking usability window for this alert's content, so it stands as a delivery failure state until Nadia's own next general delivery-warning review (FEAT-14, XBR-30) rather than expiring the information itself -- the new contact remains visible on FEAT-18.SPEC-001 regardless of whether this alert ever arrives.

## Edge Cases

- **The invited contact is removed before this alert is delivered** -- The alert still sends: it reports an event that already happened (Owen invited someone), independent of whatever the contact's current status is by the time delivery completes; Nadia is still entitled to know that Owen exercised his invite entitlement, even if the resulting contact no longer exists.
- **Two different Primary contacts at two different clients each invite a colleague within moments of each other** -- Each produces its own independent alert to Nadia, since each names a different client and a different inviting party; they are never merged, consistent with the No-Batching rule.
- **The same Primary contact invites two colleagues in quick succession** -- Two separate alerts are sent, one per invite, since each names a different new contact.
- **Delivery of this alert fails permanently** -- No further consequence beyond the standard delivery-failure surfacing (XBR-30); this is a lower-stakes failure than a credential email's failure, since Nadia can also simply notice the new contact by opening FEAT-18.SPEC-001 directly, but the failure is still surfaced like any other.
- **Nadia has no email notification preference screen bearing on this alert** -- Confirmed by design: this alert has no opt-out, consistent with its always-sends framing in product-features.md.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-004 (Invite Reviewer Colleague) | Triggered by (inbound) | Every successful invite send fires this notification |
| FEAT-18.SPEC-010 (New Contact Invitation Email) | References (inbound) | Fires alongside this notification from the same trigger, addressed to the new contact instead of Nadia |
| FEAT-18.SPEC-001 (Client Contact List) | Navigation (outbound) | The "View contacts" CTA deep-links here, scoped to the affected client |
| FEAT-14 (Notifications (Email)) | References (inbound) | Owns the underlying Transactional Email Delivery capability (FEAT-14.SPEC-001) this notification is sent through |

## Analytics and Success Signals

- **contact_invited_by_primary_alert_sent** (client_reference) -- N/A -- no success-metrics.md metric is connected to Client Contact Management & Roles; retained so counterpart-visibility alert delivery is observable
- **contact_invited_by_primary_alert_delivery_failed** (retry_count_exhausted: true) -- N/A -- no success-metrics.md metric is connected to this feature

## Acceptance Criteria

**FEAT-18.SPEC-011-AC-01:** Given Owen invites Priya as a Reviewer colleague, when the invite send completes, then Nadia receives an email with subject "Owen invited a colleague to {client_company_name}" naming Priya and her email.

**FEAT-18.SPEC-011-AC-02:** Given Nadia receives this alert, when she taps "View contacts", then she is navigated to FEAT-18.SPEC-001 scoped to the affected client.

**FEAT-18.SPEC-011-AC-03:** Given Nadia herself adds a new contact directly on FEAT-18.SPEC-002, when the save completes, then this alert is not sent, since it exists only for Primary-contact-initiated invites.

**FEAT-18.SPEC-011-AC-04:** Given two different Primary contacts at two different clients each invite a colleague within moments of each other, when both sends complete, then Nadia receives two separate alerts, one per client.

**FEAT-18.SPEC-011-AC-05:** Given Owen invites two colleagues in quick succession, when both sends complete, then Nadia receives two separate alerts, not one batched alert.

**FEAT-18.SPEC-011-AC-06:** Given the invited contact is removed before this alert is delivered, when delivery proceeds, then the alert still sends, reporting the invite event as it happened.

**FEAT-18.SPEC-011-AC-07:** Given delivery of this alert fails on the first attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`.

**FEAT-18.SPEC-011-AC-08:** Given all retries for this alert are exhausted, when the final attempt fails, then the failure is surfaced to Nadia per XBR-30's general delivery-warning behavior.

**FEAT-18.SPEC-011-AC-09:** Given this notification has no preference control, when a Primary contact invites a colleague, then the alert always sends to Nadia with no opt-out surface to check.

**FEAT-18.SPEC-011-AC-10:** Given this alert is triggered at any hour, when it fires, then it sends immediately with no quiet-hours hold, since the product defines no quiet-hours window for freelancer-side account alerts.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (no preference -- always sends) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
