# FEAT-01 — Client & Project Management

This chapter covers Client & Project Management, a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 11 specifications carrying 114 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-01.SPEC-001 | Add Client | screen | 9 |
| FEAT-01.SPEC-002 | Create Project | screen | 8 |
| FEAT-01.SPEC-003 | Client & Project Roster | screen | 10 |
| FEAT-01.SPEC-004 | Client Detail | screen | 13 |
| FEAT-01.SPEC-005 | Project Detail (Open Project) | screen | 17 |
| FEAT-01.SPEC-006 | Completion Invoice Trigger | automation | 8 |
| FEAT-01.SPEC-007 | Archive Open-Items Check | automation | 10 |
| FEAT-01.SPEC-008 | Active Client Limit Enforcement | logic-rule | 9 |
| FEAT-01.SPEC-009 | Client Delete Eligibility | logic-rule | 9 |
| FEAT-01.SPEC-010 | Client Billing Completeness Gate | logic-rule | 9 |
| FEAT-01.SPEC-011 | Project Stage Derivation | logic-rule | 12 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Client & Project Management

## Summary

**Feature:** Client & Project Management
**ID:** FEAT-01
**Description:** The freelancer can add client companies, create projects under each one, and see every client and project she manages in one place, each showing its current stage.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** The brief's whole premise is one place per client project (BRIEF.md, Vision); nothing else in the product has anywhere to attach without this container. MVP phase: every other feature depends on a client and project existing first.

**Key Capabilities:**
- Add a client company — freelancer records a new client she works with
- Create a project under a client — a unit of work with its own proposal, milestones, and invoices
- View all clients and projects — a single roster with each project's current stage
- Archive a client or project — remove it from active view without deleting its history
- Open a project — one view holding its proposal, milestones, deliverables, invoices, and activity
- Mark a project complete — closes the work and triggers the on-completion invoice when the payment schedule includes one
- Delete a client added by mistake — allowed only while the client has no sent proposal, invoice, or activity; anything with a record can only be archived

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-01.SPEC-001 | Add Client | Screen | Nadia | Nadia records a new client company she works with |
| FEAT-01.SPEC-002 | Create Project | Screen | Nadia | Nadia creates a project under a specific client |
| FEAT-01.SPEC-003 | Client & Project Roster | Screen | Nadia, Dana | Nadia (and, read-only, Dana in a support session) sees every client and project with its current stage |
| FEAT-01.SPEC-004 | Client Detail | Screen | Nadia, Dana | Single view of one client — rename, edit billing details, archive, or delete it |
| FEAT-01.SPEC-005 | Project Detail (Open Project) | Screen | Nadia, Dana | Single view of one project holding its proposal, milestones, deliverables, invoices, and activity, plus rename, archive, and mark-complete actions |
| FEAT-01.SPEC-006 | Completion Invoice Trigger | Automation | Nadia | Marking a project complete fires the on-completion invoice in Invoicing (FEAT-09) when the payment schedule includes one |
| FEAT-01.SPEC-007 | Archive Open-Items Check | Automation | Nadia | Checks a client or project for unpaid invoices or pending approvals at the moment of archiving and requires explicit confirmation if any exist |
| FEAT-01.SPEC-008 | Active Client Limit Enforcement | Logic/Rule | Nadia | Gates adding or reactivating an active client against the freelancer's current Subscription Plan limit |
| FEAT-01.SPEC-009 | Client Delete Eligibility | Logic/Rule | Nadia | Determines whether a client may be hard-deleted (no sent proposal, invoice, or activity exists) versus archived only |
| FEAT-01.SPEC-010 | Client Billing Completeness Gate | Logic/Rule | Nadia | Requires billing name, billing address (and optional tax ID) to be captured before the client's first invoice can be sent |
| FEAT-01.SPEC-011 | Project Stage Derivation | Logic/Rule | Nadia, Dana | Computes the roster/detail "stage" label (Draft, In Progress, Complete, Cancelled, Archived) from the project's proposal, milestone, invoice, completion, and cancellation state |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Add a client company | FEAT-01.SPEC-001 | Primary purpose of the Add Client screen | Phase 2 (Explicit) |
| Create a project under a client | FEAT-01.SPEC-002 | Primary purpose of the Create Project screen | Phase 2 (Explicit) |
| View all clients and projects | FEAT-01.SPEC-003 | Roster screen lists every client and project with computed stage | Phase 2 (Explicit) |
| Archive a client or project | FEAT-01.SPEC-004, FEAT-01.SPEC-005 | Archive action on Client Detail and Project Detail | Phase 2 (Explicit) |
| Open a project | FEAT-01.SPEC-005 | Primary purpose of the Project Detail screen | Phase 2 (Explicit) |
| Mark a project complete | FEAT-01.SPEC-005 | Mark Complete action on Project Detail | Phase 2 (Explicit) |
| Delete a client added by mistake | FEAT-01.SPEC-004 | Delete action on Client Detail | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-01.SPEC-006 | Completion Invoice Trigger | Phase 4 (Trigger-Response) | Marking a project complete has a cross-feature consequence (XBR-03) that nothing in the Key Capabilities names directly — the invoice firing to FEAT-09 needed its own automation surface |
| FEAT-01.SPEC-007 | Archive Open-Items Check | Phase 4 (Trigger-Response) | The Primary Flows & Alternates field states that archiving with unpaid invoices or pending approvals requires explicit confirmation; the underlying open-items check across Invoice/Proposal state is the mechanism, not a named capability |
| FEAT-01.SPEC-008 | Active Client Limit Enforcement | Phase 4 (External Dependencies lens) / Phase 5 (Rule Discovery) | The Validation & Limits field and the "Growing Past the Free Tier" journey both describe a plan-driven cap on active clients (XBR-23) that gates the Add Client capability but is never named as its own capability |
| FEAT-01.SPEC-009 | Client Delete Eligibility | Phase 5 (Rule Discovery) | The delete capability's bound ("only while the client has no sent proposal, invoice, or activity") is a testable rule (XBR-24) that needed its own surface, referenced by both the Delete capability and the account-deletion cross-feature rule |
| FEAT-01.SPEC-010 | Client Billing Completeness Gate | Phase 5 (Rule Discovery) | The Data Notes and Validation & Limits fields require billing details before the first invoice can be sent (XBR-16, ASMP-24); this cross-feature gate was implied, not named |
| FEAT-01.SPEC-011 | Project Stage Derivation | Phase 5 (Rule Discovery) | Data Notes states the roster's "stage" label is derived from state owned by other features; the derivation formula itself is unnamed machinery every screen depends on |

## Entity-Lifecycle Coverage Matrix

**Entity: Client**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-01.SPEC-001 | Add Client screen — freelancer fills client name and (optionally, at this point) billing details and saves | Gated by FEAT-01.SPEC-008 (active-client limit) |
| Read (single) | FEAT-01.SPEC-004 | Client Detail screen — loads one client's full record | Also viewable read-only by Dana (FEAT-31 support session) |
| Read (list) | FEAT-01.SPEC-003 | Client & Project Roster — lists every client alongside its projects | -- |
| Update | FEAT-01.SPEC-004 | Rename and billing-details edit on Client Detail | Billing-detail completeness governed by FEAT-01.SPEC-010 |
| Delete/Archive | FEAT-01.SPEC-004 (action), FEAT-01.SPEC-007 (open-items check), FEAT-01.SPEC-009 (delete eligibility) | **Archive:** soft delete — status set to Archived, hidden from the active roster by default but restorable to Active at any time from Client Detail (reactivation re-checks FEAT-01.SPEC-008's limit); no cascade — Projects and Client Contacts are retained and remain reachable through the Archived filter; retained indefinitely with no automatic purge (ASMP-22, SC-24: kept for the life of the account). **Delete:** hard delete — permitted only when FEAT-01.SPEC-009 confirms the client has no sent proposal, invoice, or activity; irreversible with no restore path; because eligibility requires zero downstream records, there is nothing to cascade and no retention window applies. | Archiving a client with unpaid invoices or pending approvals requires the explicit confirmation raised by FEAT-01.SPEC-007 |
| State Transition | FEAT-01.SPEC-004, FEAT-01.SPEC-008 | Active ↔ Archived only; reactivation past the free-tier client count is blocked by FEAT-01.SPEC-008 until the freelancer upgrades (cross-feature to FEAT-23) | -- |

**Entity: Project**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-01.SPEC-002 | Create Project screen — freelancer names the project under exactly one client; enters the roster at stage "Draft" | Every project belongs to exactly one client (Validation & Limits) |
| Read (single) | FEAT-01.SPEC-005 | Project Detail screen — the "open a project" view holding proposal, milestones, deliverables, invoices, and activity | Also viewable read-only by Dana (FEAT-31 support session) |
| Read (list) | FEAT-01.SPEC-003 | Client & Project Roster — lists every project with its computed stage | Stage computed by FEAT-01.SPEC-011 |
| Update | FEAT-01.SPEC-005 | Rename on Project Detail; completed_at set by FEAT-01.SPEC-006 on completion | Currency/tax fields updated by FEAT-15, not this feature |
| Delete/Archive | FEAT-01.SPEC-005 (action), FEAT-01.SPEC-007 (open-items check) | **Archive:** soft delete — status set to Archived, restorable to its prior stage from Project Detail; no cascade — Proposal, Milestones, Deliverables, Invoices, and Activity Log entries are retained and remain reachable; retained indefinitely with no automatic purge (ASMP-22, SC-24). **Delete:** N/A — the dependency map assigns hard project deletion exclusively to FEAT-24 (account deletion); this feature never offers an in-product project delete. | Completed projects stay visible to the client until archived (Primary Flows & Alternates) |
| State Transition | FEAT-01.SPEC-005, FEAT-01.SPEC-006, FEAT-01.SPEC-011 | Draft → In Progress → Complete or Cancelled → Archived; the label itself is computed by FEAT-01.SPEC-011 from proposal/milestone/invoice state; the explicit Complete transition (and its invoice trigger) is set by FEAT-01.SPEC-005/SPEC-006 | The Cancelled transition is set by FEAT-25, not this feature (dependency map: Project "Updated by ... FEAT-25 (mark cancelled)"); system-driven stage changes never overwrite a freelancer's explicit Complete or Cancelled per the dependency map's Contention line |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Client Contact | FEAT-01.SPEC-004 | Client Detail shows the "basic contact reference" named in Data Notes |
| Proposal | FEAT-01.SPEC-005, FEAT-01.SPEC-011 | Project Detail shows the proposal area; stage derivation reads proposal status |
| Payment Schedule | FEAT-01.SPEC-005, FEAT-01.SPEC-006 | Project Detail shows the milestones/payment area; completion trigger checks whether the schedule includes an on-completion payment |
| Invoice | FEAT-01.SPEC-005, FEAT-01.SPEC-007, FEAT-01.SPEC-011 | Project Detail shows the invoices area; open-items check and stage derivation read invoice status |
| Activity Log Entry | FEAT-01.SPEC-005, FEAT-01.SPEC-007, FEAT-01.SPEC-009 | Project Detail's activity area; open-items and delete-eligibility checks read whether any activity exists |
| Subscription Plan | FEAT-01.SPEC-008 | Active-client limit check reads the freelancer's current plan tier and status |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia submits a new client | Check the active-client count against the current Subscription Plan limit | Standalone Logic/Rule | FEAT-01.SPEC-008 |
| Nadia submits a new client beyond the free-tier limit | Block the add and show an upgrade prompt | Cross-feature — owned by FEAT-23 | FEAT-01.SPEC-008 initiates; FEAT-23 owns the subscribe flow |
| Nadia reactivates an archived client | Re-check the active-client limit at the moment of reactivation | Standalone Logic/Rule | FEAT-01.SPEC-008 |
| Nadia creates a project | Project enters the roster at derived stage "Draft" | Inline in triggering screen | FEAT-01.SPEC-002 (initial value from FEAT-01.SPEC-011) |
| Nadia archives a client or project with unpaid invoices or pending approvals | Surface an explicit confirmation before completing the archive | Standalone Automation | FEAT-01.SPEC-007 |
| Nadia archives a client or project with no open items | Archive immediately, no confirmation required | Inline in triggering screen | FEAT-01.SPEC-004 / FEAT-01.SPEC-005 |
| Nadia attempts to delete a client | Check for any sent proposal, invoice, or activity before allowing a hard delete | Standalone Logic/Rule | FEAT-01.SPEC-009 |
| Nadia marks a project complete, and its payment schedule includes an on-completion payment | Fire the final invoice in Invoicing & Payments | Standalone Automation (cross-feature) | FEAT-01.SPEC-006 → FEAT-09 |
| Nadia marks a project complete, no on-completion payment in the schedule | Set completed_at, recompute stage to "Complete," no invoice fires | Inline in triggering screen | FEAT-01.SPEC-005 (stage via FEAT-01.SPEC-011) |
| Nadia renames a client or project | Update the display name everywhere it is shown; ID-based references are unaffected | Inline in triggering screen | FEAT-01.SPEC-004 / FEAT-01.SPEC-005 |
| Any proposal, milestone, or invoice event changes a project's underlying state | Recompute the project's derived stage label | Standalone Logic/Rule | FEAT-01.SPEC-011 |
| Nadia attempts to send a client's first invoice with billing details missing | Block the send until billing name and address are captured | Standalone Logic/Rule (cross-feature gate on FEAT-09) | FEAT-01.SPEC-010 |
| Nadia composes a new client/project while offline | Show a clear connectivity notice; the action retries once online, no silent failure | Inline in triggering screen | FEAT-01.SPEC-001 / FEAT-01.SPEC-002 |
| Roster loads a large client list | Render skeleton rows while loading | Inline in triggering screen | FEAT-01.SPEC-003 |
| A client/project save fails | Preserve entered fields and offer retry | Inline in triggering screen | FEAT-01.SPEC-001 / FEAT-01.SPEC-002 |
| Dana opens a logged support session | View the roster, Client Detail, and Project Detail read-only, no edit controls rendered | Cross-feature — owned by FEAT-31 | FEAT-01.SPEC-003 / SPEC-004 / SPEC-005 (View), FEAT-31 (session logging) |

## Shared Context

**Shared Entities:**
- **Client** — created by FEAT-01.SPEC-001; read/updated/archived/deleted by FEAT-01.SPEC-004; listed by FEAT-01.SPEC-003; gated on create/reactivate by FEAT-01.SPEC-008, on delete by FEAT-01.SPEC-009, and on first invoice by FEAT-01.SPEC-010. Fields: client_name, billing_name, billing_address, tax_id (optional), status (Active/Archived), currency and tax treatment (set through FEAT-15).
- **Project** — created by FEAT-01.SPEC-002; read/updated/archived/completed by FEAT-01.SPEC-005; listed by FEAT-01.SPEC-003; stage computed by FEAT-01.SPEC-011; completion drives FEAT-01.SPEC-006. Fields: project_name, client (exactly one), stage (derived), currency, tax_label/tax_rate, completed_at, cancelled_at.

**Shared UI Patterns:**
- **Client/project identity block** — the same name-plus-stage-badge presentation used consistently by FEAT-01.SPEC-003 (roster rows), FEAT-01.SPEC-004 (Client Detail header), and FEAT-01.SPEC-005 (Project Detail header); Spec Writers for all three should describe it the same way.
- **Archive confirmation dialog** — shared by FEAT-01.SPEC-004 and FEAT-01.SPEC-005 through FEAT-01.SPEC-007: same "here is what's still open" summary plus an explicit confirm step, differing only in which entity and which open items are named.
- **Empty / Loading / Error / Offline-degraded states** — FEAT-01.SPEC-001, FEAT-01.SPEC-002, and FEAT-01.SPEC-003 share one convention: empty-state prompt to add the first client, skeleton-row loading for large lists, preserve-entered-fields-on-error with retry, and a plain connectivity notice with automatic retry once online for any offline compose attempt.

**Shared Validation:**
- FEAT-01.SPEC-008 (Active Client Limit Enforcement) is referenced by FEAT-01.SPEC-001 on every create and reactivate attempt rather than re-deriving the subscription check inline.
- FEAT-01.SPEC-009 (Client Delete Eligibility) is referenced by FEAT-01.SPEC-004's delete action rather than restating the "no sent proposal, invoice, or activity" condition inline.
- FEAT-01.SPEC-010 (Client Billing Completeness Gate) is referenced by FEAT-01.SPEC-004's billing fields and, cross-feature, by FEAT-09's send-invoice action, rather than duplicating the required-field set in both places.

## Internal Dependency Map

```
SPEC-003 (Client & Project Roster) -> [Nadia taps "Add Client"] -> SPEC-001 (Add Client)
SPEC-001 (Add Client) -> [Nadia submits the form] -> SPEC-008 (Active Client Limit Enforcement) -> [within limit] -> SPEC-001 (client saved) -> SPEC-003
SPEC-001 (Add Client) -> [Nadia submits, over limit] -> SPEC-008 -> [blocked] -> cross-feature FEAT-23 (upgrade prompt)
SPEC-003 (Client & Project Roster) -> [Nadia taps "New Project" under a client] -> SPEC-002 (Create Project)
SPEC-002 (Create Project) -> [project saved] -> SPEC-011 (Project Stage Derivation) -> [initial stage "Draft"] -> SPEC-003
SPEC-003 (Client & Project Roster) -> [Nadia selects a client] -> SPEC-004 (Client Detail)
SPEC-003 (Client & Project Roster) -> [Nadia selects a project] -> SPEC-005 (Project Detail)
SPEC-004 (Client Detail) -> [Nadia edits billing details] -> SPEC-010 (Client Billing Completeness Gate)
SPEC-004 (Client Detail) -> [Nadia taps Archive] -> SPEC-007 (Archive Open-Items Check) -> [confirmed] -> SPEC-004 (client archived) -> SPEC-003
SPEC-004 (Client Detail) -> [Nadia taps Delete] -> SPEC-009 (Client Delete Eligibility) -> [eligible] -> SPEC-004 (client deleted) -> SPEC-003
SPEC-004 (Client Detail) -> [Nadia taps Delete, ineligible] -> SPEC-009 -> [blocked] -> SPEC-004 (delete disabled, archive offered instead)
SPEC-005 (Project Detail) -> [Nadia taps Archive] -> SPEC-007 (Archive Open-Items Check) -> [confirmed] -> SPEC-005 (project archived) -> SPEC-003
SPEC-005 (Project Detail) -> [Nadia taps Mark Complete] -> SPEC-006 (Completion Invoice Trigger) -> [schedule has on-completion payment] -> cross-feature FEAT-09 (invoice issued)
SPEC-005 (Project Detail) -> [Nadia taps Mark Complete, no on-completion payment] -> SPEC-011 (Project Stage Derivation) -> [stage "Complete"] -> SPEC-003 / SPEC-005
SPEC-005 (Project Detail) -> [any linked proposal, milestone, or invoice event] -> SPEC-011 (Project Stage Derivation) -> [new stage label] -> SPEC-003 / SPEC-005
```

**Default Entry:** SPEC-003 (Client & Project Roster) — the screen shown when the user navigates to this feature.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-01.SPEC-005 | Outbound | FEAT-02 (Proposal Creation & Sending) | Project view's proposal area opens the project's proposal draft or detail | Nadia drafts, edits, or opens the project's proposal |
| FEAT-01.SPEC-005 | Outbound | FEAT-04 (Milestone & Payment Schedule Setup) | Project view's milestones area opens the milestone and payment schedule editor | Nadia defines or adjusts milestones and payment triggers |
| FEAT-01.SPEC-005 | Outbound | FEAT-06 (Deliverable Upload) | Project view's milestone area opens deliverable upload | Nadia uploads or links a deliverable on a milestone |
| FEAT-01.SPEC-005 | Outbound | FEAT-09 (Invoicing & Payments) | Project view's invoices area opens invoice detail or an ad hoc invoice | Nadia opens an invoice or issues one outside the schedule |
| FEAT-01.SPEC-006 | Outbound | FEAT-09 (Invoicing & Payments) | Completion trigger fires the on-completion invoice | Marking a project complete when the schedule includes an on-completion payment (XBR-03) |
| FEAT-01.SPEC-010 | Outbound | FEAT-09 (Invoicing & Payments) | Billing completeness gate blocks invoice send until billing details exist | Nadia (or FEAT-09) attempts to send the client's first invoice (XBR-16) |
| FEAT-01.SPEC-005 | Outbound | FEAT-13 (Activity & Audit Trail) | Project view's activity area opens the project's trail | Nadia opens the project's trail |
| FEAT-01.SPEC-004 | Outbound | FEAT-18 (Client Contact Management) | Client detail opens the client's contact list | Nadia manages the client's contacts and roles |
| FEAT-01.SPEC-004 / FEAT-01.SPEC-005 | Outbound | FEAT-15 (Regional & Currency Settings) | Project billing setup opens the currency/tax line editor | Nadia sets currency and tax before the first invoice |
| FEAT-01.SPEC-008 | Outbound | FEAT-23 (Subscription Plan & Billing Management) | Adding a client beyond the free-tier limit opens the upgrade prompt and subscribe flow | Adding or reactivating an active client exceeds the plan's limit (XBR-23) |
| FEAT-01.SPEC-001 / FEAT-01.SPEC-002 | Inbound | FEAT-20 (Guided Onboarding) | Onboarding step "Add first client and project" lands here | Onboarding guides Nadia to add her first client and project |
| FEAT-01.SPEC-003 / FEAT-01.SPEC-005 / FEAT-01.SPEC-002 / FEAT-01.SPEC-006 / FEAT-01.SPEC-009 | Inbound | FEAT-28 (Search) | Search result selection opens the matching client, project, proposal, deliverable, or invoice | Nadia selects a search result |
| FEAT-01.SPEC-005 | Inbound | FEAT-29 (Notification Feed, Later) | Feed item opens the related project | Nadia opens the related project from the feed |
| FEAT-01.SPEC-003 / FEAT-01.SPEC-004 / FEAT-01.SPEC-005 | Inbound | FEAT-31 (Support Access) | Dana's logged, read-only support session views the roster and detail screens | Dana opens a support session on Nadia's account |
| FEAT-01.SPEC-008 | Inbound | FEAT-23 (Subscription Plan & Billing Management) | Plan status changes (upgrade, downgrade, cancel, lapse) feed the current limit the enforcement rule reads | Nadia's subscription status changes |
| FEAT-01 (Client, Project) | Inbound | FEAT-24 (Account Deletion) | Account deletion removes Client and Project data, retaining only records under legal financial retention | Nadia deletes her account (XBR-33) |

## Non-Functional Notes

**Data volumes / growth:** Each freelancer manages 3–15 active clients with a handful of projects each, across a few thousand freelancers expected in year one (ASMP-22, SC-21); the roster and detail screens are designed to stay responsive at this scale from MVP onward, not phased in later.

**Responsiveness:** The Client and Project Setup Speed success metric targets adding a client and creating a project under it in under two minutes end to end, from opening "add client" to the project appearing in the roster; the roster itself renders skeleton rows during load per ASMP-27 rather than a blank screen.

**Data sensitivity / privacy:** Client billing name and billing address may identify a sole trader and are treated as GDPR-class personal data (ASMP-24); client records are strictly isolated per freelancer and never visible to another client (ASMP-23); Client Contact's name and email, read here only for reference display, carry the same GDPR-class handling (dependency map, Client Contact entity).

**Compliance flags:** Invoices must carry both parties' business details, so this feature is the capture point that gates invoice sending on billing-name and billing-address completeness (ASMP-24, XBR-16); on account deletion, client and project data is removed except where legal financial-record retention applies to associated invoices (SC-24, XBR-33).

## Non-Goals

- **Bulk import of clients or projects from other tools** -- Excluded per scope-boundaries.md (SC-19): a freelancer has 3–15 active clients, so adding them by hand takes minutes, and importing historical records would create client/project data that never passed through this feature's own creation flow.
- **Agency or team-of-many seat models (e.g., a scoped bookkeeper role on clients/projects)** -- Excluded per scope-boundaries.md (SC-01): the brief states solo freelancers only for v1, so this feature models no internal staff seat or delegated-access tier on Client or Project records.
- **General task/project-management tooling (boards, resourcing, team scheduling) inside the Project entity** -- Excluded per scope-boundaries.md (SC-09): Project here is a lightweight container for one client engagement's proposal, milestones, and invoices, not a task board; adding PM machinery would recreate the heavy, per-seat tools the brief positions against.
- **In-product hard delete of a Project** -- Intentional lifecycle decision from the CRUD matrix and the dependency map: Project deletion is owned exclusively by FEAT-24 (account deletion); within this feature a project can only be archived, never deleted, once created.
- **Automatic purge of archived Clients or Projects** -- Intentional lifecycle decision surfaced by the CRUD matrix: archived clients and projects are retained indefinitely because ASMP-22 and SC-24 keep this data for the life of the freelancer's account, with no automatic purge at any point.
- **Native mobile app views of the roster or detail screens** -- Excluded per scope-boundaries.md (SC-06): the brief specifies a web app with no native apps; this feature's screens are responsive web views, not device-native ones.



# Screen Spec: Add Client

## Overview

**Name:** Add Client
**ID:** FEAT-01.SPEC-001
**Type:** Screen
**Purpose:** Nadia records a new client company she works with, so a project can be created under it.
**Parent Feature:** FEAT-01 -- Client & Project Management

## Scope and Non-Goals

**In Scope:**
- Capturing a new Client record: client name, and optionally at this point, billing name, billing address, and tax ID
- Enforcing the active-client limit (FEAT-01.SPEC-008) before the client is saved
- Preserving entered data and offering retry on a failed save
- Handling composing this screen while offline

**Non-Goals:**
- Creating a project under the new client -- handled by FEAT-01.SPEC-002 (Create Project), reached from the roster after this save
- Setting the client's currency and tax treatment -- owned by Currency & Tax Handling (FEAT-15), set at the project level before the first invoice, not on this screen
- Bulk import of clients from another tool -- excluded per scope-boundaries.md SC-19: a freelancer has 3-15 active clients, so adding them by hand takes minutes, and imported records would carry no evidence trail through this feature's own creation flow
- Adding client contacts (Primary/Reviewer people) -- handled by Client Contact Management & Roles (FEAT-18), reached from Client Detail (FEAT-01.SPEC-004) after the client exists

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Nadia taps "Add Client" | None -- form starts empty |
| FEAT-20 (Onboarding / First-Run Setup) | Onboarding step "Add first client and project" lands here | None -- form starts empty |
| FEAT-23 (Subscription Plan & Billing Management), after upgrade | Nadia completes an upgrade prompted by a blocked add | The client name and any other fields she had entered before being blocked are restored |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Fill and submit the form | -- |
| Owen (Client Primary Contact) | No | No | This is a freelancer-only management surface; the screen does not exist in Owen's portal navigation |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation |
| Dana (Support Operator) | No | No | Dana's read-only support session (FEAT-31) covers the roster, Client Detail, and Project Detail only; the Add Client form is not part of the session's viewable surface, and attempting to reach it directly returns Dana to the session's roster view |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on the Client & Project Roster (FEAT-01.SPEC-003), not this form |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- entered form data is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Add Client" with a back arrow (returns to the Client & Project Roster) and a "Save" action button (right-aligned).

**Body:** A single-column form with the following fields, in order:
- Client Name (text input, required)
- Billing Name (text input, optional at this point -- required later before this client's first invoice can be sent, per FEAT-01.SPEC-010)
- Billing Address (multi-line text input, optional at this point -- required before the first invoice, per FEAT-01.SPEC-010)
- Tax ID (text input, always optional)

A note below the billing fields states: "Billing details can be added now or later, but are required before you send this client's first invoice."

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact size class:** Single-column form as described, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.
- **Billing Address field:** Grows from 2 visible lines (compact) to 3 visible lines (medium and above).

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-003 (Client & Project Roster) | Screen closes; unsaved-changes check runs first | If fields are filled, a confirmation dialog appears before leaving |
| Client Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Client Name input | Blur (empty) | Triggers required-field validation | Error state on field | "Client name is required" below field |
| Billing Name / Billing Address / Tax ID inputs | Type | Captures text input | Field shows entered text | Standard input focus state |
| Save button | Tap | 1. Validate required fields. 2. If valid, evaluate the active-client limit via FEAT-01.SPEC-008. 3. If within limit, save the Client record. | Button shows loading state during save | Success: toast "Client added" and navigate to FEAT-01.SPEC-003 with the new client visible. Blocked: see Edge Cases. Failure: inline error banner. |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Client Name -> Billing Name -> Billing Address -> Tax ID -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Client added" toast is announced on success; on validation failure, focus moves to the first field in error; on a limit block, focus moves to the blocking message.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | All fields empty, Save enabled | Screen first opens | User begins typing in any field |
| Filling | Fields contain user input | User types in any field | User taps Save or navigates away |
| Saving | Save button shows loading spinner, form disabled | Validation passes and limit check runs | Save completes, is blocked, or fails |
| Validation Error | Client Name field highlighted with its error message | Required-field validation fails on submit | User corrects the field and re-submits |
| Limit Blocked | Blocking message in place of the form's success path, with an "Upgrade" action; entered data preserved | Active-client limit check (FEAT-01.SPEC-008) returns blocked | Nadia upgrades (cross-feature FEAT-23) and returns, or navigates away |
| Error | Error banner at top of form with retry option; entered data preserved | Save operation fails for a reason other than the limit | User taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline. Connect to add a client." at top; fields remain editable but Save is disabled until connectivity returns | Connectivity lost while screen is open | Connectivity restored -- Save re-enables; if Nadia had already tapped Save while offline, the attempt automatically retries and standard success feedback appears |

## Validation Rules

Validation governed by FEAT-01.SPEC-010 (Client Billing Completeness Gate) for the billing fields' eventual completeness requirement, and inline below for this screen's own fields. This screen applies validation on field blur and on form submit.

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Client Name | Required, non-empty | On blur and on submit | "Client name is required" |
| Billing Name, Billing Address, Tax ID | None on this screen -- completeness is enforced later, at first-invoice send, by FEAT-01.SPEC-010 | -- | -- |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (no unsaved changes) | FEAT-01.SPEC-003 (Client & Project Roster) | -- |
| Successful save | FEAT-01.SPEC-003 (Client & Project Roster) | -- |
| Limit-blocked "Upgrade" action | Upgrade / subscribe flow | FEAT-23 (Subscription Plan & Billing Management) |
| Cancel with unsaved changes, "Discard" chosen | FEAT-01.SPEC-003 (Client & Project Roster) | -- |

## Data Model

**Creates:** Client record -- client_name (from input), billing_name, billing_address, tax_id (from input, may be left empty), status set to Active. currency and tax treatment are not set here (set later via FEAT-15, before the first invoice).
**Reads:** None (this is a creation screen -- no existing Client data loaded). Reads the freelancer's current active-client count and Subscription Plan tier only to evaluate FEAT-01.SPEC-008 at save time.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The active-client limit (FEAT-01.SPEC-008) is evaluated on every save attempt; the user cannot bypass it (XBR-23).
- Billing completeness (FEAT-01.SPEC-010) is not enforced on this screen -- it is enforced at the moment the client's first invoice is sent, from Client Detail (FEAT-01.SPEC-004) or Invoicing (FEAT-09).
- A newly created client's status defaults to Active and counts toward the active-client limit immediately.

## Edge Cases

- **Nadia navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner: "Could not add this client. Check your connection and try again." with a Retry button. Form data preserved.
- **Save blocked by the active-client limit (FEAT-01.SPEC-008)** -- The form's entered data is preserved; a blocking message replaces the success path: "You've reached your plan's active client limit. Upgrade to add more clients." with an "Upgrade" action that opens FEAT-23; "Keep Editing" returns to the form with data intact.
- **Two client records created with the same name** -- No rule prevents this; each save creates an independent Client record and both appear separately in the roster (FEAT-01.SPEC-003). This is a creation-only screen with no existing record loaded, so no update conflict is possible here.
- **Nadia composes this form while offline** -- Save is disabled with a connectivity notice; if she had already submitted just as connectivity dropped, the attempt is queued and retries automatically once connectivity returns, with no silent failure and no duplicate client created.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Navigation (inbound/outbound) | Entry point and destination after save or cancel |
| FEAT-01.SPEC-008 (Active Client Limit Enforcement) | References (outbound) | Save evaluates the active-client limit before committing |
| FEAT-01.SPEC-010 (Client Billing Completeness Gate) | References (outbound) | Governs when billing fields become required (not on this screen) |
| FEAT-23 (Subscription Plan & Billing Management) | Navigation (outbound) | Upgrade action when the limit blocks the add |
| FEAT-20 (Onboarding / First-Run Setup) | Navigation (inbound) | Onboarding's "add first client" step lands here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| client_add_started | entry source (roster / onboarding / post-upgrade) | Screen opens | supports success-metrics.md: "Client and Project Setup Speed" |
| client_created | billing details provided at creation (yes/no), time from client_add_started to save | Save completes successfully | supports success-metrics.md: "Client and Project Setup Speed" |
| client_add_blocked_by_limit | current active-client count, plan tier | Save is blocked by FEAT-01.SPEC-008 | supports success-metrics.md: "Client and Project Setup Speed" (a blocked add is a delay in the same setup flow this metric tracks) |

## Acceptance Criteria

**FEAT-01.SPEC-001-AC-01:** Given Nadia is on the Add Client screen, when she enters "Acme Studio" as client name and taps Save, then the active-client limit is checked, the client is saved, a "Client added" toast appears, and she returns to the roster with the new client visible.

**FEAT-01.SPEC-001-AC-02:** Given Nadia is on the Add Client screen, when she taps Save with the client name field empty, then the client name field shows the error "Client name is required" and the save does not proceed.

**FEAT-01.SPEC-001-AC-03:** Given Nadia is on the Add Client screen with unsaved input, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?"

**FEAT-01.SPEC-001-AC-04:** Given Nadia is at her plan's active-client limit, when she submits a valid client name and taps Save, then the save is blocked with the message "You've reached your plan's active client limit. Upgrade to add more clients." and her entered data remains in the form.

**FEAT-01.SPEC-001-AC-05:** Given Nadia loses connectivity while filling the form, then the "You're offline. Connect to add a client." banner appears and Save is disabled until connectivity returns.

**FEAT-01.SPEC-001-AC-06:** Given Nadia's connectivity returns after a queued Save attempt made while offline, then the client is saved automatically and the standard "Client added" success feedback appears with no duplicate client created.

**FEAT-01.SPEC-001-AC-07:** Given Nadia leaves Billing Name and Billing Address empty and taps Save, when the client name is valid, then the client saves successfully with billing fields empty, since billing completeness (FEAT-01.SPEC-010) is not required on this screen.

**FEAT-01.SPEC-001-AC-08:** Given a save attempt fails due to a network error, then an error banner "Could not add this client. Check your connection and try again." appears with a Retry button, and all entered data is preserved.

**FEAT-01.SPEC-001-AC-09:** Given Dana is in a read-only support session on Nadia's account, when she looks for a way to add a client, then no such control or screen is reachable from her session.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 7 | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Screen Spec: Create Project

## Overview

**Name:** Create Project
**ID:** FEAT-01.SPEC-002
**Type:** Screen
**Purpose:** Nadia creates a project under a specific client, entering the roster at stage "Draft."
**Parent Feature:** FEAT-01 -- Client & Project Management

## Scope and Non-Goals

**In Scope:**
- Capturing a new Project record under exactly one client
- Setting the project's initial derived stage to "Draft" via FEAT-01.SPEC-011
- Preserving entered data and offering retry on a failed save
- Handling composing this screen while offline

**Non-Goals:**
- Drafting the project's proposal -- handled by Proposal Creation & Sending (FEAT-02), reached from Project Detail (FEAT-01.SPEC-005) after the project exists
- Setting milestones or a payment schedule -- handled by Milestone & Payment Schedule Setup (FEAT-04)
- Choosing the client the project belongs to from a client that does not yet exist -- if no client exists, this screen is not reachable; Nadia must first complete FEAT-01.SPEC-001 (Add Client)
- Bulk import of historical projects -- excluded per scope-boundaries.md SC-19: with 3-15 active clients and a handful of projects each, hand entry is fast, and imported records would carry no evidence trail through this feature's own creation flow

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Nadia taps "New Project" under a specific client | The owning client is pre-selected and shown, not editable |
| FEAT-01.SPEC-004 (Client Detail) | Nadia taps "New Project" from the client's own detail view | The owning client is pre-selected and shown, not editable |
| FEAT-20 (Onboarding / First-Run Setup) | Onboarding step "Add first client and project" continues here after the client is created | The just-created client is pre-selected |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Fill and submit the form | -- |
| Owen (Client Primary Contact) | No | No | Not shown in Owen's portal navigation; project creation is a freelancer-only action |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation |
| Dana (Support Operator) | No | No | Dana's read-only support session (FEAT-31) covers the roster, Client Detail, and Project Detail only; the Create Project form is not part of the session's viewable surface |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on the Client & Project Roster (FEAT-01.SPEC-003), not this form |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- entered form data is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "New Project" with a back arrow (returns to the screen the user came from) and a "Save" action button (right-aligned).

**Body:** A single-column form with the following fields, in order:
- Client (read-only display of the pre-selected client's name; not editable on this screen)
- Project Name (text input, required)

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact size class:** Single-column form as described, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the entry screen (roster or Client Detail) | Screen closes; unsaved-changes check runs first | If the project name is filled, a confirmation dialog appears before leaving |
| Project Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Project Name input | Blur (empty) | Triggers required-field validation | Error state on field | "Project name is required" below field |
| Save button | Tap | 1. Validate the project name. 2. Save the Project record under the pre-selected client. 3. Set the initial stage to "Draft" via FEAT-01.SPEC-011. | Button shows loading state during save | Success: toast "Project created" and navigate to FEAT-01.SPEC-005 (Project Detail) for the new project. Failure: inline error banner. |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Project Name -> Save. (Client display is read-only and not part of the interactive tab order beyond being announced on screen entry.)
- **Validation announcements:** When the Project Name field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Project created" toast is announced on success; on validation failure, focus moves to the Project Name field.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Project Name empty, client pre-filled, Save enabled | Screen first opens | User begins typing |
| Filling | Project Name contains user input | User types | User taps Save or navigates away |
| Saving | Save button shows loading spinner, form disabled | Validation passes | Save completes or fails |
| Validation Error | Project Name field highlighted with its error message | Required-field validation fails on submit | User corrects the field and re-submits |
| Error | Error banner at top of form with retry option; entered data preserved | Save operation fails | User taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline. Connect to create a project." at top; the field remains editable but Save is disabled until connectivity returns | Connectivity lost while screen is open | Connectivity restored -- Save re-enables; if Nadia had already tapped Save while offline, the attempt automatically retries and standard success feedback appears |

## Validation Rules

**Option B -- Inline (simple validation not warranting a standalone spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Project Name | Required, non-empty | On blur and on submit | "Project name is required" |
| Client | Must reference exactly one existing, non-archived client (pre-selected, not user-editable on this screen) | On screen entry | N/A -- the screen is not reachable without a valid client context |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (no unsaved changes) | Roster or Client Detail (whichever the user arrived from) | -- |
| Successful save | FEAT-01.SPEC-005 (Project Detail) | -- |
| Cancel with unsaved changes, "Discard" chosen | Roster or Client Detail (whichever the user arrived from) | -- |

## Data Model

**Creates:** Project record -- project_name (from input), client (the pre-selected client, exactly one), stage (initial value "Draft," derived via FEAT-01.SPEC-011), currency/tax_label/tax_rate unset (set later via FEAT-15), completed_at and cancelled_at unset.
**Reads:** The pre-selected Client record's name and active status (an archived client cannot receive a new project; the screen is not reachable from an archived client's context).
**Updates:** None.
**Deletes:** None.

## Business Rules

- Every project belongs to exactly one client; this cannot be changed after creation on this screen (moving a project to a different client is not a capability this product defines).
- A new project's initial stage is always "Draft," derived by FEAT-01.SPEC-011 rather than set directly.
- A project cannot be created under an archived client; Create Project is not reachable from an archived client's context.

## Edge Cases

- **Nadia navigates away with an unsaved project name** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner: "Could not create this project. Check your connection and try again." with a Retry button. Form data preserved.
- **The owning client is archived or deleted by another session between screen open and save** -- Save is rejected with "This client is no longer active. Return to the roster and try again." and the user is returned to FEAT-01.SPEC-003; this is a creation-only screen with no existing Project record loaded, so the conflict is on the parent Client reference, not a Project update conflict.
- **Two projects created under the same client with the same name** -- No rule prevents this; each save creates an independent Project record and both appear separately under the client.
- **Nadia composes this form while offline** -- Save is disabled with a connectivity notice; if she had already submitted just as connectivity dropped, the attempt is queued and retries automatically once connectivity returns, with no silent failure and no duplicate project created.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Navigation (inbound/outbound) | Entry point when arriving from "New Project" on the roster; destination on cancel |
| FEAT-01.SPEC-004 (Client Detail) | Navigation (inbound/outbound) | Entry point when arriving from a client's own detail view; destination on cancel |
| FEAT-01.SPEC-005 (Project Detail) | Navigation (outbound) | Destination after a successful save |
| FEAT-01.SPEC-011 (Project Stage Derivation) | Triggers (outbound) | Sets the project's initial stage to "Draft" |
| FEAT-20 (Onboarding / First-Run Setup) | Navigation (inbound) | Onboarding's "add first client and project" step continues here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| project_create_started | entry source (roster / client detail / onboarding) | Screen opens | supports success-metrics.md: "Client and Project Setup Speed" |
| project_created | time from opening "add client" (FEAT-01.SPEC-001) to this project appearing in the roster | Save completes successfully | supports success-metrics.md: "Client and Project Setup Speed" |

## Acceptance Criteria

**FEAT-01.SPEC-002-AC-01:** Given Nadia is on the Create Project screen for client "Acme Studio," when she enters "Website Redesign" as project name and taps Save, then the project is saved with stage "Draft," a "Project created" toast appears, and she is taken to the new project's Project Detail (FEAT-01.SPEC-005).

**FEAT-01.SPEC-002-AC-02:** Given Nadia is on the Create Project screen, when she taps Save with the project name field empty, then the project name field shows the error "Project name is required" and the save does not proceed.

**FEAT-01.SPEC-002-AC-03:** Given Nadia is on the Create Project screen with an unsaved project name, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?"

**FEAT-01.SPEC-002-AC-04:** Given Nadia loses connectivity while filling the form, then the "You're offline. Connect to create a project." banner appears and Save is disabled until connectivity returns.

**FEAT-01.SPEC-002-AC-05:** Given Nadia's connectivity returns after a queued Save made while offline, then the project is saved automatically and the standard "Project created" success feedback appears with no duplicate project created.

**FEAT-01.SPEC-002-AC-06:** Given a save attempt fails due to a network error, then an error banner "Could not create this project. Check your connection and try again." appears with a Retry button, and the entered project name is preserved.

**FEAT-01.SPEC-002-AC-07:** Given the owning client was archived by another session while Nadia had this screen open, when she taps Save, then the save is rejected with "This client is no longer active. Return to the roster and try again." and she is returned to the roster.

**FEAT-01.SPEC-002-AC-08:** Given Dana is in a read-only support session on Nadia's account, when she looks for a way to create a project, then no such control or screen is reachable from her session.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Client & Project Roster

## Overview

**Name:** Client & Project Roster
**ID:** FEAT-01.SPEC-003
**Type:** Screen
**Purpose:** Nadia (and, read-only, Dana in a support session) sees every client and project she manages in one place, each showing its current stage.
**Parent Feature:** FEAT-01 -- Client & Project Management

## Scope and Non-Goals

**In Scope:**
- Listing every Client and its Projects, with each project's derived stage
- Entry points into Add Client, Create Project, Client Detail, and Project Detail
- Filtering by Active/Archived status
- Empty, loading, error, and offline states for a list of this size (3-15 clients per freelancer)

**Non-Goals:**
- Editing a client or project inline from the roster -- handled by FEAT-01.SPEC-004 (Client Detail) and FEAT-01.SPEC-005 (Project Detail)
- Free-text search across clients and projects -- handled by Global Search (FEAT-28, v1), a distinct feature the roster does not duplicate
- Sorting or grouping beyond client-then-project order -- excluded per scope-boundaries.md's positioning against heavy PM tooling (SC-09): with 3-15 clients, a fixed, simple structure serves the stated scale without configurable views
- Displaying financial totals per client or project -- owned by the Freelancer Financial Dashboard (FEAT-12), which the roster links to but does not duplicate

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Main navigation | Nadia opens the Client & Project Management area (this is the feature's default entry screen) | None |
| FEAT-20 (Onboarding / First-Run Setup) | Onboarding completes | None -- lands here with the first client and project visible |
| FEAT-31 (Operator Support Access) | Dana opens a logged, read-only support session on Nadia's account | Session is scoped to this one freelancer account, read-only |
| FEAT-28 (Global Search, v1) | Nadia selects a client or project search result | The matching client or project is highlighted/scrolled into view |
| FEAT-01.SPEC-001 / FEAT-01.SPEC-002 / FEAT-01.SPEC-004 / FEAT-01.SPEC-005 | Successful save, archive, or delete on any of those screens | The affected client/project's row reflects its new state |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, all her clients and projects | Add Client, New Project, open any client or project, filter Active/Archived | -- |
| Owen (Client Primary Contact) | No | No | Not shown in Owen's portal navigation; this is the freelancer's own management surface, not the client portal (FEAT-05) |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation |
| Dana (Support Operator) | Full screen, read-only, for the one account under an open support session | View only -- Add Client, New Project, and any edit affordances are not rendered | Attempting to reach an add/create action directly (e.g., a stale link) returns Dana to this roster with no action taken |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- the roster reloads fresh after re-authentication (no in-progress state to preserve on a list screen) |

## Layout and Content

**Header:** Screen title "Clients & Projects" with an "Add Client" action button (top right). A filter control (Active / Archived / All, defaulting to Active) sits below the title.

**Body:** A list grouped by client. Each client group shows:
- Client identity block: client name plus a status badge (Active/Archived)
- A "New Project" action for that client (hidden for Dana's read-only session and for an archived client)
- Below the client row, one row per project belonging to that client: project name plus a stage badge (Draft, In Progress, Complete, Cancelled, Archived), using the same identity-block presentation as Client Detail (FEAT-01.SPEC-004) and Project Detail (FEAT-01.SPEC-005)

A client with no projects yet shows a single line under its group: "No projects yet."

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Client groups and project rows stack in a single column, full width; the "New Project" action for each client collapses to an icon button.
- **Medium size class and above:** The same single-column grouped list is retained (no multi-column restructure -- the product's scale of 3-15 clients does not warrant it), capped at a consistent platform-wide content width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Add Client" button | Tap | Navigate to FEAT-01.SPEC-001 (Add Client) | Screen transitions | Standard navigation transition |
| Filter control | Select Active / Archived / All | Re-fetches and re-renders the list scoped to the chosen status | List content updates | Selected filter option highlighted |
| Client row (name) | Tap | Navigate to FEAT-01.SPEC-004 (Client Detail) for that client | Screen transitions | Standard navigation transition |
| "New Project" (per client) | Tap | Navigate to FEAT-01.SPEC-002 (Create Project) pre-scoped to that client | Screen transitions | Standard navigation transition |
| Project row | Tap | Navigate to FEAT-01.SPEC-005 (Project Detail) for that project | Screen transitions | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Add Client -> Filter control -> each client group in list order (client name, New Project action, then its project rows in order).
- **Dynamic updates:** When the filter changes the list content, the updated result count is announced to assistive technology (e.g., "12 active clients shown").
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | A prompt: "No clients yet -- add your first one to get started," with an "Add Client" action | Nadia has zero clients under the current filter | A client is added, or the filter is changed to a scope with results |
| Loading | Skeleton rows for client groups and project rows | Screen opens or filter changes, while data is being fetched | Data finishes loading |
| Loaded | Full list as described in Layout and Content | Data load completes with one or more results | Filter changes or a mutating action occurs elsewhere |
| Error | Error banner: "Couldn't load your clients and projects. Try again." with a Retry button | The list fails to load | Retry succeeds |
| Offline/Degraded | A connectivity banner appears at the top: "You're offline -- showing the last loaded list." The most recently loaded list remains visible and browsable; Add Client and New Project are disabled | Connectivity is lost while viewing an already-loaded roster | Connectivity restored -- the banner clears and the list refreshes |

## Validation Rules

Not applicable -- this is a read-only listing screen with no user input beyond the filter selection, which has no invalid states (a closed set of options).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Add Client" tap | FEAT-01.SPEC-001 (Add Client) | -- |
| "New Project" tap (per client) | FEAT-01.SPEC-002 (Create Project) | -- |
| Client row tap | FEAT-01.SPEC-004 (Client Detail) | -- |
| Project row tap | FEAT-01.SPEC-005 (Project Detail) | -- |

## Data Model

**Creates:** None.
**Reads:** Client records -- client_name, status (Active/Archived), for every client belonging to the freelancer account (or, for Dana, the one account under her support session). Project records -- project_name, stage (derived by FEAT-01.SPEC-011), for every project belonging to each listed client.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The roster's default filter is Active; Archived clients and projects are hidden unless the Archived or All filter is selected, per the dependency map's Client and Project lifecycle notes.
- A project's stage badge always reflects the value computed by FEAT-01.SPEC-011, never a value set directly by this screen.
- Dana's support session (FEAT-31) renders this screen with every edit affordance removed, per XBR-29.

## Edge Cases

- **Roster loads a large client list** -- Skeleton rows render during load rather than a blank screen, satisfying the responsiveness expectation for the product's stated scale (3-15 active clients per freelancer).
- **A client or project changes state while the roster is open (e.g., archived from another open session)** -- The roster is a snapshot per load, not live-updating; the changed row is stale until the next load or filter change, which is acceptable because this screen only reads shared entities and never writes to them, so no write conflict is possible here (any conflict is resolved at the point of the write, in FEAT-01.SPEC-004 or FEAT-01.SPEC-005).
- **A client has projects in every stage** -- All are shown; there is no cap on project rows per client at this product's scale.
- **Filter selected with zero matching results (e.g., Archived with nothing archived yet)** -- Shows "No archived clients or projects" rather than the general empty-state prompt to add a first client.
- **Nadia navigates away and returns** -- The roster re-fetches on return rather than showing a stale cached view, so recent changes from Client Detail or Project Detail are reflected.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-001 (Add Client) | Navigation (outbound) | "Add Client" action |
| FEAT-01.SPEC-002 (Create Project) | Navigation (outbound) | "New Project" action per client |
| FEAT-01.SPEC-004 (Client Detail) | Navigation (outbound) | Selecting a client row |
| FEAT-01.SPEC-005 (Project Detail) | Navigation (outbound) | Selecting a project row |
| FEAT-01.SPEC-011 (Project Stage Derivation) | References (inbound) | Supplies the stage badge shown per project |
| FEAT-28 (Global Search Across Clients & Projects) | Navigation (inbound) | Search result selection lands here with the match highlighted |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only session entry point |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| roster_viewed | filter selected, result count, viewer role (Nadia / Dana) | Screen loads successfully | supports success-metrics.md: "Client and Project Setup Speed" (the roster is the point at which a newly added client and project become visible, closing the loop this metric measures) |
| roster_filter_changed | filter selected | Nadia changes the Active/Archived/All filter | N/A -- no success-metrics.md metric measures roster filter usage for this feature; recorded for product-usage visibility only |

## Acceptance Criteria

**FEAT-01.SPEC-003-AC-01:** Given Nadia has no clients yet, when she opens the Client & Project Roster, then she sees "No clients yet -- add your first one to get started" with an "Add Client" action.

**FEAT-01.SPEC-003-AC-02:** Given Nadia has clients and projects, when the roster loads, then each project shows a stage badge computed by FEAT-01.SPEC-011 (Draft, In Progress, Complete, Cancelled, or Archived).

**FEAT-01.SPEC-003-AC-03:** Given Nadia is on the roster, when she taps "Add Client," then she is taken to FEAT-01.SPEC-001 (Add Client).

**FEAT-01.SPEC-003-AC-04:** Given Nadia is on the roster, when she taps "New Project" under client "Acme Studio," then she is taken to FEAT-01.SPEC-002 (Create Project) pre-scoped to Acme Studio.

**FEAT-01.SPEC-003-AC-05:** Given Nadia is on the roster with the default Active filter, when she switches the filter to Archived, then only archived clients and their projects are shown, and the change is announced to assistive technology.

**FEAT-01.SPEC-003-AC-06:** Given Nadia's roster has more clients than fit in the first render, when it loads, then skeleton rows appear while data is fetched rather than a blank screen.

**FEAT-01.SPEC-003-AC-07:** Given Nadia loses connectivity while viewing an already-loaded roster, then a "You're offline -- showing the last loaded list." banner appears, the last list remains browsable, and Add Client / New Project become disabled.

**FEAT-01.SPEC-003-AC-08:** Given the roster fails to load, then an error banner "Couldn't load your clients and projects. Try again." appears with a Retry button.

**FEAT-01.SPEC-003-AC-09:** Given Dana opens a logged support session on Nadia's account, when she views the roster, then she sees every client and project read-only, with no Add Client or New Project controls rendered.

**FEAT-01.SPEC-003-AC-10:** Given Nadia selects a client or project from Global Search results (FEAT-28), when the roster opens, then the matching row is highlighted and scrolled into view.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Client Detail

## Overview

**Name:** Client Detail
**ID:** FEAT-01.SPEC-004
**Type:** Screen
**Purpose:** Single view of one client -- rename it, edit billing details, view its contacts, and archive, reactivate, or delete it.
**Parent Feature:** FEAT-01 -- Client & Project Management

## Scope and Non-Goals

**In Scope:**
- Renaming the client
- Editing billing details (billing name, billing address, tax ID), governed by FEAT-01.SPEC-010 for completeness
- Archiving and reactivating the client, including the open-items confirmation (FEAT-01.SPEC-007)
- Deleting the client when eligible (FEAT-01.SPEC-009), or explaining why it is not
- Showing the client's basic contact reference and a link into full contact management

**Non-Goals:**
- Managing the full client contact list (invite, remove, change roles) -- handled by Client Contact Management & Roles (FEAT-18), reached from this screen
- Setting currency and tax treatment -- owned by Currency & Tax Handling (FEAT-15), configured at the project level
- Viewing the client's financial totals -- owned by the Freelancer Financial Dashboard (FEAT-12)
- Editing a project's own details -- handled by FEAT-01.SPEC-005 (Project Detail)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Nadia selects a client row | The client's identifier |
| FEAT-31 (Operator Support Access) | Dana selects a client during a read-only support session | Same client, read-only rendering |
| FEAT-28 (Global Search, v1) | Nadia selects a client search result | The client's identifier |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Rename, edit billing details, archive/reactivate, delete (when eligible), navigate to contacts | -- |
| Owen (Client Primary Contact) | No | No | Not shown in Owen's portal navigation; this is the freelancer's own management surface |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation |
| Dana (Support Operator) | Full screen, read-only | View only -- Rename, edit billing, archive/reactivate, and delete controls are not rendered | Attempting to reach an edit action directly returns Dana to the read-only view with no change made |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- any unsaved billing-detail edits are preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Client name (editable inline via a "Rename" action) with a status badge (Active/Archived), using the same identity-block presentation as the roster (FEAT-01.SPEC-003) and Project Detail (FEAT-01.SPEC-005). An overflow menu holds Archive/Reactivate and Delete.

**Body, in order:**
- **Billing details section:** Billing Name (text input), Billing Address (multi-line text input), Tax ID (text input, optional), each editable in place with a "Save" action for the section. A completeness indicator ("Billing details complete" / "Billing details incomplete -- required before the first invoice") reflects FEAT-01.SPEC-010.
- **Contacts section:** The client's basic contact reference (name and email of the Primary contact, if any) with a "Manage Contacts" link into FEAT-18.
- **Projects section:** A list of this client's projects (name plus stage badge), each opening FEAT-01.SPEC-005, with a "New Project" action.

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Header, billing section, contacts section, and projects section stack in a single column, full width.
- **Medium size class and above:** Same single-column stacking, capped at a consistent platform-wide content width and horizontally centered; the billing section's address field grows from 2 to 3 visible lines.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Rename" action | Tap, then edit and confirm | Updates client_name | Header reflects new name | Toast "Client renamed" |
| Billing fields | Type, then tap section "Save" | Updates billing_name, billing_address, tax_id | Completeness indicator updates | Toast "Billing details saved," or, if the underlying record changed concurrently, see Edge Cases |
| Overflow menu > Archive | Tap | Triggers FEAT-01.SPEC-007 (Archive Open-Items Check) | If no open items, client archives immediately; if open items exist, a confirmation dialog appears first | Toast "Client archived" on completion |
| Overflow menu > Reactivate (shown only when Archived) | Tap | Re-checks the active-client limit via FEAT-01.SPEC-008, then sets status to Active | Status badge updates to Active | Toast "Client reactivated," or a limit-blocked message (see Edge Cases) |
| Overflow menu > Delete | Tap | Evaluates eligibility via FEAT-01.SPEC-009 | If eligible, a confirmation dialog appears, then the client is hard-deleted; if ineligible, Delete is disabled and Archive is offered instead | Toast "Client deleted" on completion, or the ineligible explanation (see Edge Cases) |
| "Manage Contacts" link | Tap | Navigate to Client Contact Management & Roles (FEAT-18) for this client | Screen transitions | Standard navigation transition |
| "New Project" action | Tap | Navigate to FEAT-01.SPEC-002 (Create Project) pre-scoped to this client | Screen transitions | Standard navigation transition |
| Project row | Tap | Navigate to FEAT-01.SPEC-005 (Project Detail) | Screen transitions | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Rename action -> overflow menu -> billing fields in order -> section Save -> Manage Contacts link -> New Project action -> project rows in order.
- **Dynamic updates:** Toasts and the completeness indicator's change are announced to assistive technology; a concurrent-edit rejection dialog receives focus immediately when it appears.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton layout for header, billing, contacts, and projects sections | Screen opens | Data finishes loading |
| Loaded | Full detail as described in Layout and Content | Data load completes | A mutating action occurs, or navigation away |
| Editing (billing) | Billing fields in edit mode with a visible section Save | Nadia begins editing a billing field | Save completes, is rejected, or edit is cancelled |
| Error | Error banner: "Couldn't load this client. Try again." with a Retry button | Initial data load fails | Retry succeeds |
| Offline/Degraded | Banner "You're offline -- changes will be saved when you reconnect." at top; the loaded detail remains viewable; Rename, billing Save, Archive, Reactivate, and Delete are disabled until connectivity returns | Connectivity lost while viewing an already-loaded client | Connectivity restored -- controls re-enable |

## Validation Rules

Validation governed by FEAT-01.SPEC-010 (Client Billing Completeness Gate) for billing-field completeness, and by FEAT-01.SPEC-009 (Client Delete Eligibility) for the Delete action. Client name follows the same "required, non-empty" rule as on Add Client (FEAT-01.SPEC-001), checked on the Rename confirm action.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Manage Contacts" tap | Client contact list | FEAT-18 (Client Contact Management & Roles) |
| "New Project" tap | FEAT-01.SPEC-002 (Create Project) | -- |
| Project row tap | FEAT-01.SPEC-005 (Project Detail) | -- |
| Successful archive or delete | FEAT-01.SPEC-003 (Client & Project Roster) | -- |
| Back navigation | FEAT-01.SPEC-003 (Client & Project Roster) | -- |

## Data Model

**Creates:** None.
**Reads:** Client record -- client_name, billing_name, billing_address, tax_id, status. Client Contact -- basic reference (name, email) of the Primary contact, read-only. Project records belonging to this client -- project_name, stage (derived).
**Updates:** Client record -- client_name (Rename), billing_name, billing_address, tax_id (billing edit), status (Archive/Reactivate).
**Deletes:** Client record -- only when FEAT-01.SPEC-009 confirms eligibility.

## Business Rules

- Renaming a client updates its display name everywhere it is shown; ID-based references (projects, invoices) are unaffected.
- Archive is subject to the open-items confirmation defined by FEAT-01.SPEC-007; it never erases data, and archived clients and their projects/contacts are retained indefinitely with no automatic purge (ASMP-22, SC-24).
- Reactivating an archived client re-checks the active-client limit (FEAT-01.SPEC-008, XBR-23); reactivation is blocked the same way a new-client add would be.
- Delete is permitted only when FEAT-01.SPEC-009 confirms the client has no sent proposal, invoice, or activity (XBR-24); otherwise Delete is disabled and Archive is offered as the only removal path.
- Billing completeness (FEAT-01.SPEC-010) governs when this client's first invoice can be sent; this screen shows the completeness state but does not itself block sending (that block lives in FEAT-09).

## Edge Cases

- **Client renamed or billing-edited by another session between load and save (last-write-wins per the dependency map's Contention note for Client)** -- The later save simply overwrites the earlier one; no conflict dialog for descriptive-field edits.
- **Archive or Delete attempted after the client's state changed since load (reject-with-refresh per the dependency map's Contention note)** -- The action is rejected with "This client's records changed since you loaded this page. Refresh to see the latest state before archiving/deleting." and a "Refresh" action reloads the client; this is the concurrent-edit conflict behavior for state-changing actions on this shared entity.
- **Archive attempted with unpaid invoices or pending approvals** -- FEAT-01.SPEC-007 surfaces an explicit confirmation summarizing what is still open before the archive completes; declining returns to this screen unchanged.
- **Reactivate attempted beyond the active-client limit** -- Blocked with "You've reached your plan's active client limit. Upgrade to reactivate this client." and an "Upgrade" action opening FEAT-23; the client remains Archived.
- **Delete attempted on an ineligible client** -- Delete is disabled in the overflow menu with inline text: "This client has a sent proposal, invoice, or activity and can only be archived." Archive remains available.
- **Nadia loses connectivity mid-edit** -- The offline banner appears; in-progress billing edits are preserved locally and Save remains disabled until connectivity returns, then the pending save submits automatically.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Navigation (inbound/outbound) | Entry point; destination after archive or delete |
| FEAT-01.SPEC-002 (Create Project) | Navigation (outbound) | "New Project" action |
| FEAT-01.SPEC-005 (Project Detail) | Navigation (outbound) | Selecting a project row |
| FEAT-01.SPEC-007 (Archive Open-Items Check) | Triggers (outbound) | Archive action runs the open-items check |
| FEAT-01.SPEC-008 (Active Client Limit Enforcement) | References (outbound) | Reactivate re-checks the active-client limit |
| FEAT-01.SPEC-009 (Client Delete Eligibility) | References (outbound) | Delete action's eligibility check |
| FEAT-01.SPEC-010 (Client Billing Completeness Gate) | References (outbound) | Billing section's completeness rules |
| FEAT-18 (Client Contact Management & Roles) | Navigation (outbound) | "Manage Contacts" link |
| FEAT-23 (Subscription Plan & Billing Management) | Navigation (outbound) | Upgrade action on a limit-blocked reactivation |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only session entry point |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| client_billing_details_saved | fields changed, completeness result (complete/incomplete) | Billing section save completes | N/A -- billing completeness is measured by FEAT-01.SPEC-010's own coverage, not by "Client and Project Setup Speed," which measures initial add-to-roster time only |
| client_archived | had open items at archive time (yes/no) | Archive completes | N/A -- no success-metrics.md metric tracks archive activity for this feature |
| client_deleted | -- | Delete completes | N/A -- no success-metrics.md metric tracks delete activity for this feature |

## Acceptance Criteria

**FEAT-01.SPEC-004-AC-01:** Given Nadia is on Client Detail for "Acme Studio," when she renames it to "Acme Studio Inc." and confirms, then the header updates and a "Client renamed" toast appears.

**FEAT-01.SPEC-004-AC-02:** Given Nadia edits the billing address and taps section Save, when the save succeeds, then the completeness indicator updates to reflect whether billing details are now complete.

**FEAT-01.SPEC-004-AC-03:** Given Nadia taps Archive on a client with no unpaid invoices or pending approvals, then the client archives immediately with a "Client archived" toast and no confirmation dialog.

**FEAT-01.SPEC-004-AC-04:** Given Nadia taps Archive on a client with an unpaid invoice, then FEAT-01.SPEC-007's confirmation dialog appears summarizing the open item before the archive completes.

**FEAT-01.SPEC-004-AC-05:** Given Nadia taps Reactivate on an archived client while she is already at her plan's active-client limit, then the action is blocked with "You've reached your plan's active client limit. Upgrade to reactivate this client." and the client stays Archived.

**FEAT-01.SPEC-004-AC-06:** Given Nadia opens the overflow menu on a client with a sent invoice, when she looks at Delete, then it is disabled with the text "This client has a sent proposal, invoice, or activity and can only be archived."

**FEAT-01.SPEC-004-AC-07:** Given Nadia taps Delete on a client with no sent proposal, invoice, or activity, when she confirms, then the client is permanently deleted and she returns to the roster with a "Client deleted" toast.

**FEAT-01.SPEC-004-AC-08:** Given the client's record changed in another session since Nadia loaded this screen, when she attempts to Archive, then the action is rejected with "This client's records changed since you loaded this page. Refresh to see the latest state before archiving/deleting."

**FEAT-01.SPEC-004-AC-09:** Given Dana is viewing this client in a read-only support session, when she looks for Rename, billing Save, Archive, Reactivate, or Delete, then none of those controls are rendered.

**FEAT-01.SPEC-004-AC-10:** Given Nadia loses connectivity while editing billing details, then the offline banner appears, Save is disabled, and her in-progress edits are preserved.

**FEAT-01.SPEC-004-AC-11:** Given Nadia is on Client Detail, when she taps "Manage Contacts," then she is taken to Client Contact Management & Roles (FEAT-18) for this client.

**FEAT-01.SPEC-004-AC-12:** Given Client Detail fails to load, then an error banner "Couldn't load this client. Try again." appears with a Retry button.

**FEAT-01.SPEC-004-AC-13:** Given Nadia taps "New Project" from Client Detail, then she is taken to FEAT-01.SPEC-002 (Create Project) pre-scoped to this client.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Screen Spec: Project Detail (Open Project)

## Overview

**Name:** Project Detail (Open Project)
**ID:** FEAT-01.SPEC-005
**Type:** Screen
**Purpose:** Single view of one project holding its proposal, milestones, deliverables, invoices, and activity, plus rename, archive/reactivate, and mark-complete actions.
**Parent Feature:** FEAT-01 -- Client & Project Management

## Scope and Non-Goals

**In Scope:**
- Renaming the project
- Archiving the project, including the open-items confirmation (FEAT-01.SPEC-007)
- Reactivating an archived project, restoring it to whatever stage it would show had it never been archived (FEAT-01.SPEC-011)
- Marking the project complete, including firing the completion invoice trigger (FEAT-01.SPEC-006) when applicable
- Surfacing the project's proposal, milestone/payment-schedule, deliverable, invoice, and activity areas as entry points into their owning features
- Showing the project's current derived stage (FEAT-01.SPEC-011)

**Non-Goals:**
- Drafting or editing the proposal itself -- handled by Proposal Creation & Sending (FEAT-02), opened from this screen's proposal area
- Defining milestones or the payment schedule -- handled by Milestone & Payment Schedule Setup (FEAT-04), opened from this screen's milestones area
- Uploading deliverables -- handled by Deliverable Upload & Sharing (FEAT-06), opened from this screen's milestone area
- Viewing or issuing invoices -- handled by Invoice Generation & Sending (FEAT-09), opened from this screen's invoices area
- Marking a project cancelled -- owned exclusively by Refund & Cancelled Project Handling (FEAT-25); this screen does not offer a Cancel action
- In-product hard delete of a project -- intentional lifecycle exclusion (dependency map): project deletion is owned exclusively by FEAT-24 (account deletion); this screen offers Archive only, never Delete

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Nadia selects a project row | The project's identifier |
| FEAT-01.SPEC-004 (Client Detail) | Nadia selects a project row from the client's own project list | The project's identifier |
| FEAT-01.SPEC-002 (Create Project) | Successful save | The newly created project's identifier |
| FEAT-31 (Operator Support Access) | Dana selects a project during a read-only support session | Same project, read-only rendering |
| FEAT-28 (Global Search, v1) | Nadia selects a project search result | The project's identifier |
| FEAT-29 (In-App Notification Center, Later) | Nadia opens the related project from a feed item | The project's identifier |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Rename, Archive, Reactivate (when Archived), Mark Complete, open proposal/milestones/invoices/activity areas | -- |
| Owen (Client Primary Contact) | No | No | Not shown in Owen's portal navigation; this is the freelancer's own management surface, distinct from his own project status view in the client portal (FEAT-05) |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation |
| Dana (Support Operator) | Full screen, read-only | View only -- Rename, Archive, Reactivate, and Mark Complete controls are not rendered; deliverable files are never downloadable from this view (ASMP-18) | Attempting to reach an edit action or a file download directly returns Dana to the read-only view with no change made |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- an in-progress rename is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Project name (editable inline via a "Rename" action) with a stage badge (Draft, In Progress, Complete, Cancelled, Archived), using the same identity-block presentation as the roster (FEAT-01.SPEC-003) and Client Detail (FEAT-01.SPEC-004). Shows the owning client's name as a link back to Client Detail. An overflow menu holds Archive, Reactivate, and Mark Complete: Reactivate is shown only when the project's current stage is Archived (and hidden otherwise); Mark Complete is hidden once the project is already Complete, Cancelled, or Archived; Archive is hidden while the project is Archived.

**Body, in order:**
- **Proposal area:** Summary of the project's active proposal status (none / draft / sent / accepted), opening Proposal Creation & Sending (FEAT-02) or Proposal Acceptance detail.
- **Milestones area:** Summary of the milestone/payment-schedule status, opening Milestone & Payment Schedule Setup (FEAT-04); milestones with deliverables link into Deliverable Upload & Sharing (FEAT-06).
- **Invoices area:** Summary list of the project's invoices with their status, opening Invoice Generation & Sending (FEAT-09) for detail or an ad hoc invoice.
- **Activity area:** A link opening the project's trail in Immutable Activity & Audit Trail (FEAT-13).

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Header and the four content areas (proposal, milestones, invoices, activity) stack in a single column, full width, in the order listed.
- **Medium size class and above:** Same single-column stacking, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Rename" action | Tap, then edit and confirm | Updates project_name | Header reflects new name | Toast "Project renamed" |
| Client name link | Tap | Navigate to FEAT-01.SPEC-004 (Client Detail) | Screen transitions | Standard navigation transition |
| Overflow menu > Archive | Tap | Triggers FEAT-01.SPEC-007 (Archive Open-Items Check) | If no open items, project archives immediately; if open items exist, a confirmation dialog appears first | Toast "Project archived" on completion |
| Overflow menu > Reactivate (shown only when the project is Archived) | Tap | Clears the project's Archived status; stage recomputes via FEAT-01.SPEC-011 to whatever value its underlying proposal, milestone, invoice, completion, and cancellation state produce | Stage badge updates to the recomputed stage | Toast "Project reactivated" |
| Overflow menu > Mark Complete | Tap | Triggers FEAT-01.SPEC-006 (Completion Invoice Trigger); sets completed_at; recomputes stage via FEAT-01.SPEC-011 | Stage badge updates to "Complete" | Confirmation dialog states plainly what completing does: "This marks the project complete and, if the payment schedule includes one, issues the final invoice." Toast "Project marked complete" after confirming. |
| Proposal area | Tap | Navigate to FEAT-02 (proposal draft or detail) | Screen transitions | Standard navigation transition |
| Milestones area | Tap | Navigate to FEAT-04 (milestone and payment schedule editor) | Screen transitions | Standard navigation transition |
| Invoices area | Tap | Navigate to FEAT-09 (invoice detail or ad hoc invoice) | Screen transitions | Standard navigation transition |
| Activity area | Tap | Navigate to FEAT-13 (project activity trail) | Screen transitions | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Rename action -> client name link -> overflow menu -> proposal area -> milestones area -> invoices area -> activity area.
- **Dynamic updates:** Toasts, the stage badge's change, and the Mark Complete confirmation dialog's text are announced to assistive technology; a concurrent-edit rejection dialog receives focus immediately when it appears.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton layout for header and the four content areas | Screen opens | Data finishes loading |
| Loaded | Full detail as described in Layout and Content | Data load completes | A mutating action occurs, or navigation away |
| Marking Complete | Mark Complete confirmation dialog, then a brief in-progress indicator while FEAT-01.SPEC-006 evaluates and FEAT-01.SPEC-011 recomputes stage | Nadia confirms Mark Complete | Completion finishes or fails |
| Error | Error banner: "Couldn't load this project. Try again." with a Retry button | Initial data load fails | Retry succeeds |
| Offline/Degraded | Banner "You're offline -- changes will be saved when you reconnect." at top; the loaded detail remains viewable; Rename, Archive, Reactivate, and Mark Complete are disabled until connectivity returns | Connectivity lost while viewing an already-loaded project | Connectivity restored -- controls re-enable |

## Validation Rules

**Option B -- Inline (simple validation not warranting a standalone spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Project Name (rename) | Required, non-empty | On confirm | "Project name is required" |

Archive and Mark Complete eligibility are governed by FEAT-01.SPEC-007 and FEAT-01.SPEC-006/FEAT-01.SPEC-011 respectively, referenced above.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Client name link tap | FEAT-01.SPEC-004 (Client Detail) | -- |
| Proposal area tap | Proposal draft / detail | FEAT-02 (Proposal Creation & Sending) |
| Milestones area tap | Milestone and payment schedule editor | FEAT-04 (Milestone & Payment Schedule Setup) |
| Invoices area tap | Invoice detail / ad hoc invoice | FEAT-09 (Invoice Generation & Sending) |
| Activity area tap | Project activity trail | FEAT-13 (Immutable Activity & Audit Trail) |
| Successful archive | FEAT-01.SPEC-003 (Client & Project Roster) | -- |
| Back navigation | The screen the user arrived from (roster or Client Detail) | -- |

## Data Model

**Creates:** None (mark-complete and archive are transitions on the existing record, not new records).
**Reads:** Project record -- project_name, client (owning client's name), stage (derived), completed_at, cancelled_at. Summary status from Proposal, Payment Schedule, and Invoice (read-only summaries; full detail lives in their owning features).
**Updates:** Project record -- project_name (Rename), status (Archive, Reactivate), completed_at (Mark Complete).
**Deletes:** None -- this screen offers no delete path for a project (see Non-Goals).

## Business Rules

- Renaming a project updates its display name everywhere it is shown; ID-based references (invoices, activity entries) are unaffected.
- Archive is subject to the open-items confirmation defined by FEAT-01.SPEC-007; archived projects are retained indefinitely with no automatic purge (ASMP-22, SC-24), and their Proposal, Milestones, Deliverables, Invoices, and Activity Log entries remain reachable.
- Reactivating an archived project clears its Archived status; stage recomputes immediately via FEAT-01.SPEC-011 to the value its underlying proposal, milestone, invoice, completion, and cancellation state produce independent of the Archived override -- the project returns to whatever stage it would show had it never been archived (Draft, In Progress, Complete, or Cancelled). Reactivation carries no active-count or plan-limit check: FEAT-01.SPEC-008's active-client limit governs Clients only, per that spec's own Non-Goals, and this product defines no equivalent cap on Projects.
- Marking a project complete fires FEAT-01.SPEC-006, which issues the final invoice only when the payment schedule includes an on-completion payment (XBR-03); otherwise completed_at is set and the stage recomputes to "Complete" with no invoice.
- Completed projects stay visible to the client (Owen, Priya) in their portal until archived.
- System-driven stage changes (from proposal acceptance or milestone approval, per FEAT-01.SPEC-011) never overwrite a freelancer's explicit Complete or Cancelled transition, per the dependency map's Contention note for Project.
- Cancellation (Cancelled stage) is set exclusively by Refund & Cancelled Project Handling (FEAT-25); this screen never sets it directly.

## Edge Cases

- **Project renamed by another session between load and save (last-write-wins per the dependency map's Contention note for Project)** -- The later save simply overwrites the earlier one; no conflict dialog for the rename.
- **Archive, Reactivate, or Mark Complete attempted after the project's state changed since load (reject-with-refresh per the dependency map's Contention note)** -- The action is rejected with "This project's state changed since you loaded this page. Refresh to see the latest state before continuing." and a "Refresh" action reloads the project; this is the concurrent-edit conflict behavior for state-changing actions on this shared entity.
- **Archive attempted with unpaid invoices or pending approvals** -- FEAT-01.SPEC-007 surfaces an explicit confirmation summarizing what is still open before the archive completes; declining returns to this screen unchanged.
- **Mark Complete attempted on a project already marked Cancelled by FEAT-25 since this screen loaded** -- Rejected with "This project was cancelled since you loaded this page. Refresh to see the latest state." per the same reject-with-refresh resolution; Mark Complete is not offered on an already-Cancelled project once refreshed.
- **Reactivate tapped on an Archived project that also has completed_at set (it was completed, then later archived, and is now reactivated)** -- Once the Archived override clears, FEAT-01.SPEC-011's next-highest precedence condition applies: since completed_at is still set, the stage recomputes to "Complete," not "In Progress" -- reactivation never fabricates an in-progress state for a project that was already Complete before it was archived.
- **Reactivate tapped on an Archived project that was never completed or cancelled (no milestone approved, no invoice generated, proposal not yet accepted)** -- Stage recomputes to "Draft," matching the state it would show had it never been archived.
- **A linked proposal, milestone, or invoice event changes the project's underlying state while this screen is open** -- The stage badge is a snapshot per load, not live-updating; it reflects the latest computed value (FEAT-01.SPEC-011) on the next load or refresh, consistent with the roster's (FEAT-01.SPEC-003) same snapshot behavior.
- **Nadia loses connectivity mid-rename** -- The offline banner appears; the in-progress rename is preserved locally and the confirm action remains disabled until connectivity returns, then submits automatically.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Navigation (inbound/outbound) | Entry point; destination after archive |
| FEAT-01.SPEC-004 (Client Detail) | Navigation (inbound/outbound) | Entry point; destination via client name link |
| FEAT-01.SPEC-002 (Create Project) | Navigation (inbound) | Entry point after a successful project creation |
| FEAT-01.SPEC-006 (Completion Invoice Trigger) | Triggers (outbound) | Mark Complete fires this automation |
| FEAT-01.SPEC-007 (Archive Open-Items Check) | Triggers (outbound) | Archive action runs the open-items check |
| FEAT-01.SPEC-011 (Project Stage Derivation) | References (inbound) | Supplies the stage badge and recomputes it after Mark Complete or Reactivate |
| FEAT-02 (Proposal Creation & Sending) | Navigation (outbound) | Proposal area |
| FEAT-04 (Milestone & Payment Schedule Setup) | Navigation (outbound) | Milestones area |
| FEAT-06 (Deliverable Upload & Sharing) | Navigation (outbound) | Deliverable upload from a milestone within the milestones area |
| FEAT-09 (Invoice Generation & Sending) | Navigation (outbound) | Invoices area |
| FEAT-13 (Immutable Activity & Audit Trail) | Navigation (outbound) | Activity area |
| FEAT-25 (Refund & Cancelled Project Handling) | References (inbound) | Owns the Cancelled transition this screen only displays |
| FEAT-28 (Global Search Across Clients & Projects) | Navigation (inbound) | Search result selection lands here |
| FEAT-29 (In-App Notification Center) | Navigation (inbound) | Feed item opens the related project (Later) |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only session entry point |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| project_opened | viewer role (Nadia / Dana), current stage | Screen loads successfully | N/A -- opening an existing project is outside the initial add-to-roster window "Client and Project Setup Speed" measures |
| project_marked_complete | had on-completion payment in schedule (yes/no) | Mark Complete confirmed and processed | N/A -- no success-metrics.md metric tracks project completion for this feature; completion's downstream invoice event is measured under Invoice Generation & Sending's own metrics |
| project_archived | had open items at archive time (yes/no) | Archive completes | N/A -- no success-metrics.md metric tracks archive activity for this feature |
| project_reactivated | recomputed stage after reactivation | Reactivate completes | N/A -- no success-metrics.md metric tracks reactivate activity for this feature |

## Acceptance Criteria

**FEAT-01.SPEC-005-AC-01:** Given Nadia is on Project Detail for "Website Redesign," when she renames it to "Website Redesign v2" and confirms, then the header updates and a "Project renamed" toast appears.

**FEAT-01.SPEC-005-AC-02:** Given Nadia is on Project Detail, when she taps the client name link, then she is taken to that client's Client Detail (FEAT-01.SPEC-004).

**FEAT-01.SPEC-005-AC-03:** Given Nadia taps Archive on a project with no unpaid invoices or pending approvals, then the project archives immediately with a "Project archived" toast and no confirmation dialog.

**FEAT-01.SPEC-005-AC-04:** Given Nadia taps Archive on a project with an unpaid invoice, then FEAT-01.SPEC-007's confirmation dialog appears summarizing the open item before the archive completes.

**FEAT-01.SPEC-005-AC-05:** Given Nadia taps Mark Complete on a project whose payment schedule includes an on-completion payment, when she confirms, then FEAT-01.SPEC-006 fires the final invoice, completed_at is set, and the stage badge updates to "Complete."

**FEAT-01.SPEC-005-AC-06:** Given Nadia taps Mark Complete on a project whose payment schedule has no on-completion payment, when she confirms, then completed_at is set and the stage badge updates to "Complete" with no invoice issued.

**FEAT-01.SPEC-005-AC-07:** Given the project was cancelled by FEAT-25 in another session since Nadia loaded this screen, when she attempts Mark Complete, then the action is rejected with "This project was cancelled since you loaded this page. Refresh to see the latest state."

**FEAT-01.SPEC-005-AC-08:** Given a client contact opens the project in their portal after it is marked complete, then the project remains visible to them until it is archived.

**FEAT-01.SPEC-005-AC-09:** Given Dana is viewing this project in a read-only support session, when she looks for Rename, Archive, Reactivate, or Mark Complete, then none of those controls are rendered.

**FEAT-01.SPEC-005-AC-10:** Given Nadia is on Project Detail, when she taps the invoices area, then she is taken to Invoice Generation & Sending (FEAT-09) for this project.

**FEAT-01.SPEC-005-AC-11:** Given Nadia is on Project Detail, when she taps the activity area, then she is taken to the project's trail in Immutable Activity & Audit Trail (FEAT-13).

**FEAT-01.SPEC-005-AC-12:** Given Nadia loses connectivity while viewing an already-loaded project, then the offline banner appears and Rename, Archive, Reactivate, and Mark Complete become disabled.

**FEAT-01.SPEC-005-AC-13:** Given Project Detail fails to load, then an error banner "Couldn't load this project. Try again." appears with a Retry button.

**FEAT-01.SPEC-005-AC-14:** Given Nadia taps Mark Complete, when the confirmation dialog appears, then it states plainly "This marks the project complete and, if the payment schedule includes one, issues the final invoice" before she confirms.

**FEAT-01.SPEC-005-AC-15:** Given Nadia taps Reactivate on an Archived project, when it completes, then the project's Archived status clears, the stage badge updates to its recomputed value, and a "Project reactivated" toast appears.

**FEAT-01.SPEC-005-AC-16:** Given Nadia taps Reactivate on an Archived project that has completed_at set, when it completes, then the stage badge shows "Complete," not "In Progress."

**FEAT-01.SPEC-005-AC-17:** Given Nadia opens the overflow menu on a project that is not Archived, when she looks for Reactivate, then it is not shown, since Reactivate only appears once a project's stage is Archived.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 5 | 5 |
| Business Rules | 7 | 7 |
| Edge Cases | 8 | 8 |



# Automation Spec: Completion Invoice Trigger

## Overview

**Name:** Completion Invoice Trigger
**ID:** FEAT-01.SPEC-006
**Type:** Automation
**Purpose:** Marking a project complete fires the on-completion invoice in Invoicing & Payments (FEAT-09) when the project's payment schedule includes one.
**Parent Feature:** FEAT-01 -- Client & Project Management

## Scope and Non-Goals

**In Scope:**
- Detecting, at the moment a project is marked complete, whether its Payment Schedule includes an on-completion payment
- Handing off to Invoice Generation & Sending (FEAT-09) to create and send that invoice when applicable
- Setting the project's completed_at timestamp regardless of whether an invoice fires
- Recomputing the project's derived stage to "Complete" via FEAT-01.SPEC-011 once this automation resolves

**Non-Goals:**
- Generating the invoice content (amount, currency, tax line, numbering) -- owned entirely by FEAT-09; this automation only fires the trigger and hands off the schedule reference
- Deciding whether the freelancer is allowed to mark the project complete -- that decision is made on Project Detail (FEAT-01.SPEC-005), where the action originates
- Handling a project marked cancelled instead of complete -- owned by Refund & Cancelled Project Handling (FEAT-25), a distinct transition with its own trigger
- Retrying a failed invoice generation on a schedule -- excluded per scope-boundaries.md SC-11: the product ships fixed, sensible behavior rather than a configurable retry/automation builder; a failed hand-off surfaces to Nadia for a manual retry instead (see Edge Cases)

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia marks a project complete | FEAT-01.SPEC-005 (Project Detail) | Fires when Nadia confirms Mark Complete on a project not already Complete, Cancelled, or Archived | Project reference, its Payment Schedule reference, current time |

## Processing Logic

1. Receive the project reference from the triggering Mark Complete confirmation on FEAT-01.SPEC-005.
2. Read the project's Payment Schedule (FEAT-04) as it stands at this exact moment.
3. Determine whether the schedule's structure includes an on-completion payment (a completion_amount is set).
4. If it does, hand off to Invoice Generation & Sending (FEAT-09) with the project reference and the schedule's completion_amount, requesting the on-completion invoice be created and sent (XBR-03).
5. Set the project's completed_at timestamp to the current time, regardless of whether step 4 ran.
6. Signal FEAT-01.SPEC-011 (Project Stage Derivation) to recompute the project's stage, which resolves to "Complete" now that completed_at is set.
7. Return the outcome to the triggering screen (FEAT-01.SPEC-005) so it can update its stage badge and confirmation feedback.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Completed, invoice issued | Schedule includes an on-completion payment | completed_at set; stage recomputed to "Complete"; a new Invoice created and sent by FEAT-09 | Toast "Project marked complete" on FEAT-01.SPEC-005; the client sees the invoice arrive per FEAT-09's own notification | FEAT-01.SPEC-005, FEAT-01.SPEC-011, FEAT-09 |
| Completed, no invoice | Schedule has no on-completion payment | completed_at set; stage recomputed to "Complete"; no invoice created | Toast "Project marked complete" on FEAT-01.SPEC-005 | FEAT-01.SPEC-005, FEAT-01.SPEC-011 |
| Completion recorded, invoice hand-off failed | Schedule includes an on-completion payment but FEAT-09 cannot create the invoice | completed_at set; stage recomputed to "Complete"; no invoice created | Non-blocking warning on FEAT-01.SPEC-005: "Project marked complete, but the final invoice couldn't be issued. Retry from the invoices area." with a retry action into FEAT-09 | FEAT-01.SPEC-005, FEAT-01.SPEC-011, FEAT-09 |
| Automation failure before completion is recorded | The trigger itself fails before completed_at is set (e.g., the schedule cannot be read) | No data changes | Blocking error on FEAT-01.SPEC-005: "Couldn't mark this project complete. Try again." -- the project remains in its prior stage | FEAT-01.SPEC-005 |

## Data Model

**Reads:** Payment Schedule -- structure and completion_amount, as they stand at the moment of firing. Project -- identifier, current stage.
**Creates:** None directly -- the Invoice itself is created by FEAT-09, not by this automation.
**Updates:** Project -- completed_at (set once, never altered afterward); stage (recomputed via FEAT-01.SPEC-011).
**Deletes:** None.

## Business Rules

- XBR-03: marking a project complete generates the on-completion invoice when the schedule includes one; completed projects stay visible to the client until archived.
- The schedule is read as it stood at the moment of completion -- a schedule change saved afterward never retroactively creates or alters a completion invoice already issued, consistent with the dependency map's Contention note for Payment Schedule.
- completed_at, once set, is never altered by this automation; a later correction to the schedule or invoice is handled entirely within FEAT-04/FEAT-09, not by re-firing this trigger.
- System-driven stage changes never overwrite completed_at or a freelancer's explicit Complete transition (dependency map, Project Contention note).

## Edge Cases

- **Project has no Payment Schedule at all** -- Treated as "no on-completion payment"; completed_at is set and the stage recomputes to "Complete" with no invoice, since at least one payment trigger is required for any invoicing to exist per FEAT-04's validation, and its absence simply means nothing is owed at completion.
- **FEAT-09 is unable to create the invoice (e.g., a downstream validation failure)** -- Completion is not blocked or rolled back; completed_at and the stage change persist, and Nadia sees the non-blocking warning with a retry path, so a billing hiccup never traps the project in a stale "In Progress" state.
- **The project was already cancelled by FEAT-25 in another session before this trigger runs** -- The Mark Complete action itself is rejected upstream on FEAT-01.SPEC-005 with a refresh prompt before this automation ever fires; this automation never runs against a project already Cancelled.
- **Concurrent trigger firing (Mark Complete tapped from two open sessions of Nadia's at effectively the same time)** -- The triggering screen (FEAT-01.SPEC-005) rejects the second attempt with reject-with-refresh once the first has set completed_at, per the dependency map's Contention note for Project; this automation itself never runs twice for the same project.
- **Trigger fires while a previous run is in flight** -- Cannot occur: Mark Complete is disabled on FEAT-01.SPEC-005 while a prior completion attempt for the same project is processing, so a second run for the same project never starts before the first resolves.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-005 (Project Detail) | Triggered by (inbound) | Mark Complete confirmation fires this automation |
| FEAT-01.SPEC-011 (Project Stage Derivation) | Affects (outbound) | Recomputes the project's stage after completed_at is set |
| FEAT-04 (Milestone & Payment Schedule Setup) | References (inbound) | Supplies the Payment Schedule read at trigger time |
| FEAT-09 (Invoice Generation & Sending) | Affects (outbound) | Owns creation and sending of the on-completion invoice |
| FEAT-25 (Refund & Cancelled Project Handling) | References (inbound) | A project it has cancelled cannot be completed by this automation |

## Analytics and Success Signals

- **completion_invoice_triggered** (had on-completion payment: yes/no) -- N/A -- no success-metrics.md metric tracks project-completion invoicing directly for this feature; the resulting invoice's own accuracy is measured under Invoice Generation & Sending's "Invoice Auto-Generation Accuracy" metric, not here.
- **completion_invoice_hand_off_failed** (reason: schedule unreadable / invoice service error) -- N/A -- no success-metrics.md metric in this feature's slice measures this failure path; it is tracked for reliability visibility only.

## Acceptance Criteria

**FEAT-01.SPEC-006-AC-01:** Given Nadia marks a project complete and its payment schedule includes an on-completion payment, when the trigger fires, then the final invoice is created and sent via FEAT-09, completed_at is set, and the stage recomputes to "Complete."

**FEAT-01.SPEC-006-AC-02:** Given Nadia marks a project complete and its payment schedule has no on-completion payment, when the trigger fires, then completed_at is set and the stage recomputes to "Complete" with no invoice issued.

**FEAT-01.SPEC-006-AC-03:** Given a project has no Payment Schedule at all, when Nadia marks it complete, then it is treated as having no on-completion payment and completes with no invoice.

**FEAT-01.SPEC-006-AC-04:** Given the schedule includes an on-completion payment but FEAT-09 cannot create the invoice, when the trigger fires, then the project still completes and Nadia sees "Project marked complete, but the final invoice couldn't be issued. Retry from the invoices area."

**FEAT-01.SPEC-006-AC-05:** Given the automation fails before completed_at is set (e.g., the schedule cannot be read), when Nadia attempts to mark the project complete, then she sees "Couldn't mark this project complete. Try again." and the project remains in its prior stage.

**FEAT-01.SPEC-006-AC-06:** Given the payment schedule is later adjusted after a completion invoice has already been issued, then the already-issued invoice is unaffected, per the schedule-as-it-stood rule.

**FEAT-01.SPEC-006-AC-07:** Given Nadia has two sessions open on the same project and taps Mark Complete in both at effectively the same time, when the first completes, then the second is rejected with a refresh prompt rather than issuing a second completion invoice.

**FEAT-01.SPEC-006-AC-08:** Given a project was already cancelled by FEAT-25, when Nadia attempts Mark Complete, then the attempt is rejected upstream on FEAT-01.SPEC-005 and this automation never fires.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Automation Spec: Archive Open-Items Check

## Overview

**Name:** Archive Open-Items Check
**ID:** FEAT-01.SPEC-007
**Type:** Automation
**Purpose:** Checks a client or project for unpaid invoices or pending approvals at the moment of archiving and requires explicit confirmation if any exist.
**Parent Feature:** FEAT-01 -- Client & Project Management

## Scope and Non-Goals

**In Scope:**
- Checking, at the moment Archive is invoked on a Client (FEAT-01.SPEC-004) or a Project (FEAT-01.SPEC-005), whether any unpaid invoices or pending milestone approvals exist for the target and, for a client, across all its projects
- Presenting the exact open items found so Nadia can make an informed choice
- Proceeding with the archive immediately when nothing is open, with no confirmation step

**Non-Goals:**
- Deciding client delete eligibility -- a distinct, stricter check owned by FEAT-01.SPEC-009 (no sent proposal, invoice, or activity at all, versus this check's narrower "nothing currently unpaid or pending")
- Resolving the open items themselves (e.g., collecting the unpaid invoice) -- this automation only surfaces them; resolution happens in Invoice Payment Processing (FEAT-10) or Milestone Approval (FEAT-08)
- Reversing an archive once confirmed -- reactivation (FEAT-01.SPEC-004, FEAT-01.SPEC-008) is a separate action with its own limit check, not a rollback of this automation
- Checking projects for their own client-level archive -- when a client is archived, this automation checks the client and every one of its projects together in one pass, rather than requiring a separate check per project; archiving a single project (leaving the client Active) checks only that project

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia taps Archive on a client | FEAT-01.SPEC-004 (Client Detail) | Always, before the client's status changes | Client reference, all of its Projects' Invoice and Payment Schedule (pending approval) state |
| Nadia taps Archive on a project | FEAT-01.SPEC-005 (Project Detail) | Always, before the project's status changes | Project reference, its own Invoice and pending milestone-approval state |

## Processing Logic

1. Receive the archive target (a Client, or a single Project) from the triggering screen.
2. If the target is a Client, gather every Invoice and every Milestone across all of that client's Projects; if the target is a single Project, gather only that project's own Invoices and Milestones.
3. Evaluate each gathered Invoice's status: flag any invoice not in a Paid, Refunded, Partially refunded, or Corrected state as an open item ("unpaid invoice").
4. Evaluate each gathered Milestone's status: flag any milestone in a Deliverable Uploaded state (awaiting the client's approval) as an open item ("pending approval").
5. If no open items are found, signal the triggering screen to proceed with the archive immediately.
6. If one or more open items are found, return the list (grouped as unpaid invoices and pending approvals, each named by project and identifier) to the triggering screen for the explicit confirmation dialog.
7. On confirmation, signal the triggering screen to proceed with the archive; on decline, signal it to take no action.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| No open items | Zero unpaid invoices and zero pending approvals found | None from this automation; the triggering screen proceeds to set status to Archived | Archive completes immediately with a "Client archived" / "Project archived" toast | FEAT-01.SPEC-004, FEAT-01.SPEC-005 |
| Open items found, confirmed | 1+ open items found; Nadia confirms | None from this automation; the triggering screen proceeds to set status to Archived | The shared "here is what's still open" confirmation dialog, then the standard archive toast on confirming | FEAT-01.SPEC-004, FEAT-01.SPEC-005 |
| Open items found, declined | 1+ open items found; Nadia declines | None | Dialog closes; the client or project remains Active, unchanged | FEAT-01.SPEC-004, FEAT-01.SPEC-005 |
| Automation failure | The check itself cannot complete (e.g., invoice or milestone state cannot be read) | None | Blocking error on the triggering screen: "Couldn't check this item for open invoices or approvals. Try again." -- Archive does not proceed | FEAT-01.SPEC-004, FEAT-01.SPEC-005 |

## Data Model

**Reads:** Invoice -- status, project, across the archive target's scope. Milestone -- status, project, across the archive target's scope.
**Creates:** None.
**Updates:** None -- this automation only informs the archive decision; the status change to Archived is applied by the triggering screen (FEAT-01.SPEC-004 or FEAT-01.SPEC-005), not by this automation.
**Deletes:** None.

## Business Rules

- Archiving never erases records; this check exists precisely to make sure Nadia is not surprised by silently archiving something with financial or approval consequences still open (XBR-24).
- The check runs synchronously as part of the Archive action -- the triggering screen waits for its result before showing either the immediate-archive toast or the confirmation dialog.
- "Unpaid" for this check means any invoice status other than Paid, Refunded, Partially refunded, or Corrected -- an Overdue or Payment pending invoice is still an open item.
- "Pending approval" for this check means a milestone in Deliverable Uploaded status -- an Approved or Reopened milestone is not an open item for this purpose.

## Edge Cases

- **Client has multiple projects, only one with an unpaid invoice** -- The confirmation names the specific project and invoice, not a generic "this client has open items" message.
- **All invoices are Paid but one milestone is awaiting approval** -- The check still surfaces the pending approval as an open item; "no open items" requires both invoice and approval checks to be clear.
- **An invoice's status changes (e.g., gets paid) between the check running and Nadia confirming the dialog** -- The confirmation is based on the state read when the check ran; if Nadia confirms, the archive proceeds against the archive target's current state at commit, which is re-checked by the triggering screen's own reject-with-refresh behavior for state changes (FEAT-01.SPEC-004, FEAT-01.SPEC-005) -- a state change during the brief confirmation window does not block a since-resolved item from still being reported, but does not block the archive either, since being Paid can only reduce open items, never require blocking further.
- **Concurrent trigger firing (Archive tapped on the same client from two open sessions of Nadia's at effectively the same time)** -- Each session's check runs independently; whichever archive completes first wins, and the second is rejected with the triggering screen's reject-with-refresh behavior for a client/project whose state changed since load (FEAT-01.SPEC-004, FEAT-01.SPEC-005, per the dependency map's Contention notes).
- **Trigger fires while a previous run is in flight** -- Archive is disabled on the triggering screen while a check for the same target is in progress, so a second run for the same client or project cannot start before the first resolves; checks for different targets proceed independently.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-004 (Client Detail) | Triggered by (inbound) | Archive action on a client fires this check |
| FEAT-01.SPEC-005 (Project Detail) | Triggered by (inbound) | Archive action on a project fires this check |
| FEAT-01.SPEC-004 (Client Detail) | Affects (outbound) | Returns the open-items result and confirmation outcome to the client archive flow |
| FEAT-01.SPEC-005 (Project Detail) | Affects (outbound) | Returns the open-items result and confirmation outcome to the project archive flow |
| FEAT-09 (Invoice Generation & Sending) | References (inbound) | Source of Invoice status read by this check |
| FEAT-13 (Immutable Activity & Audit Trail) | References (inbound) | The confirmed archive, once applied, is itself a record-worthy event logged by FEAT-13 |

## Analytics and Success Signals

- **archive_open_items_found** (target type: client/project, unpaid invoice count, pending approval count) -- N/A -- no success-metrics.md metric in this feature's slice tracks archive open-items frequency; recorded for product-usage visibility only.
- **archive_confirmed_with_open_items** (target type: client/project) -- N/A -- same reason as above.
- **archive_declined_with_open_items** (target type: client/project) -- N/A -- same reason as above.

## Acceptance Criteria

**FEAT-01.SPEC-007-AC-01:** Given Nadia taps Archive on a client with no unpaid invoices and no pending approvals across any of its projects, when the check runs, then the archive proceeds immediately with no confirmation dialog.

**FEAT-01.SPEC-007-AC-02:** Given Nadia taps Archive on a project with one unpaid invoice, when the check runs, then a confirmation dialog names that specific invoice before the archive can proceed.

**FEAT-01.SPEC-007-AC-03:** Given Nadia taps Archive on a client whose only open item is a milestone awaiting approval on one of its projects (all invoices Paid), when the check runs, then the confirmation dialog surfaces the pending approval as the open item.

**FEAT-01.SPEC-007-AC-04:** Given Nadia sees the open-items confirmation dialog and taps Confirm, then the archive proceeds and the standard archive toast appears.

**FEAT-01.SPEC-007-AC-05:** Given Nadia sees the open-items confirmation dialog and taps Decline, then the dialog closes and the client or project remains Active, unchanged.

**FEAT-01.SPEC-007-AC-06:** Given the open-items check itself fails to complete, when Nadia taps Archive, then she sees "Couldn't check this item for open invoices or approvals. Try again." and the archive does not proceed.

**FEAT-01.SPEC-007-AC-07:** Given a project has an Overdue invoice, when the check runs, then the invoice is treated as an open item, since "unpaid" includes Overdue and Payment pending statuses.

**FEAT-01.SPEC-007-AC-08:** Given Nadia has two sessions open on the same client and taps Archive in both at effectively the same time, when the first archive completes, then the second is rejected with a refresh prompt rather than running a redundant archive.

**FEAT-01.SPEC-007-AC-09:** Given a milestone on a project is Approved (not Deliverable Uploaded) and all of that project's invoices are Paid, when Nadia taps Archive on that project, then the check finds no open items and the archive proceeds immediately.

**FEAT-01.SPEC-007-AC-10:** Given Nadia taps Archive on a project (not a client) with no unpaid invoices and no pending approvals, when the check runs, then the archive proceeds immediately with no confirmation dialog, the same as for a client target.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



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



# Logic/Rule Spec: Client Billing Completeness Gate

## Overview

**Name:** Client Billing Completeness Gate
**ID:** FEAT-01.SPEC-010
**Type:** Logic/Rule
**Purpose:** Requires billing name and billing address (tax ID always optional) to be captured before any invoice for the client can be sent -- evaluated fresh at every send attempt, most visibly encountered on the client's first invoice.
**Parent Feature:** FEAT-01 -- Client & Project Management
**Governed Entity:** Client (specifically the billing_name, billing_address, and tax_id fields)

## Scope and Non-Goals

**In Scope:**
- The completeness rule for a client's billing_name and billing_address (required) and tax_id (always optional)
- When completeness is evaluated: at billing-field edit time (informational) and at every invoice-send attempt (blocking, cross-feature) -- not a one-time-only check
- The exact completeness indicator shown on Client Detail and the exact block shown when Invoicing attempts to send a first invoice with billing details missing

**Non-Goals:**
- The invoice-sending flow itself, or any other invoice content requirement (invoice numbering, due dates, business details) -- owned entirely by Invoice Generation & Sending (FEAT-09), which enforces this gate at the moment of sending but does not define it
- Currency and tax rate configuration -- owned by Currency & Tax Handling (FEAT-15), a separate completeness requirement evaluated independently by FEAT-09 at send time
- Validating the format of billing_name or billing_address beyond non-empty presence -- the product defines no format constraint for these free-text fields, since business names and addresses vary too widely worldwide to validate against a fixed pattern (SC-20 covers locale format adaptation, not free-text business fields)

## Governed Entity

**Entity:** Client
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| client_name | text | Company name |
| billing_name | text | Name printed on invoices |
| billing_address | text | Address printed on invoices |
| tax_id | text | Client's tax identifier (optional) |
| status | enum (Active, Archived) | Not evaluated by this spec |
| currency and tax treatment | derived / configured (via FEAT-15) | Evaluated by FEAT-15's own completeness rule, not this spec |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-004 | Client Detail | Informational completeness indicator, re-evaluated on every billing-field save |
| FEAT-01.SPEC-001 | Add Client | Referenced only -- billing fields are optional at creation time (see Non-Goals of that spec) |
| FEAT-09 | Invoice Generation & Sending | Blocking gate re-evaluated on every invoice send attempt for the client, not only the first |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| billing_name | Required for billing completeness | Before any invoice for the client can be sent while incomplete | On billing-field save (informational) and on every invoice send attempt (blocking) | "Billing name is required before sending this client's first invoice." | Yes, at send time only -- not blocking on Client Detail's own save |
| billing_address | Required for billing completeness | Before any invoice for the client can be sent while incomplete | On billing-field save (informational) and on every invoice send attempt (blocking) | "Billing address is required before sending this client's first invoice." | Yes, at send time only -- not blocking on Client Detail's own save |
| tax_id | No validation beyond data type -- always optional | Always | -- | -- | No |
| client_name, status, currency and tax treatment | No validation beyond data type in this spec -- billing completeness depends only on billing_name and billing_address; client_name is governed by FEAT-01.SPEC-001, status by FEAT-01.SPEC-007/FEAT-01.SPEC-009, and currency and tax treatment by FEAT-15 | Always | -- | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Billing completeness | billing_name, billing_address | Complete only when both billing_name and billing_address are non-empty; tax_id does not factor into completeness | "This client's billing details are incomplete. Add a billing name and address before sending the first invoice." (shown on FEAT-09's send action when incomplete) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Edit billing_name, billing_address, tax_id | Nadia (Freelancer) | Always | -- |
| View billing completeness state | Nadia (Freelancer) | Always | -- |
| View billing completeness state | Dana (Support Operator) | Always, inside a logged support session (FEAT-31), read-only | -- |
| Edit billing_name, billing_address, tax_id | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never | Not shown; billing details are managed on the freelancer's own Client Detail screen, not exposed in either contact's portal |
| Edit billing_name, billing_address, tax_id | Dana (Support Operator) | Never | The billing-edit controls are not rendered in Dana's read-only support session (FEAT-31); she sees the completeness indicator only |
| Send an invoice for the client | Nadia (Freelancer) | Only when billing_name and billing_address are both non-empty at the moment of send (this spec's completeness rule, re-evaluated on every send attempt) | Blocked on FEAT-09's send action with "This client's billing details are incomplete. Add a billing name and address before sending the first invoice." and a link to Client Detail's billing section |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| billing_complete (derived, not a stored field) | True when billing_name and billing_address are both non-empty | Recalculated on every billing-field save and read whenever FEAT-09 attempts to send any invoice for the client | No -- computed solely from billing_name and billing_address |

## Business Rules

- XBR-16 / ASMP-24: every invoice must carry the freelancer's business details and the client's billing details; sending is blocked until both sets of details exist. This spec owns the client-side half of that gate; FEAT-21 owns the freelancer-business-details half.
- The gate is evaluated fresh by FEAT-09 at the moment of every invoice send attempt for the client, against billing_name and billing_address as they currently stand -- it is not a one-time-only check that fires only for the client's first invoice. In the common case, once these fields are captured they are never cleared, so completing them before the first send effectively satisfies the gate permanently in practice; but there is no special-cased exemption for later sends -- this spec's authority is over the field state at each send attempt.
- If billing_name or billing_address is subsequently cleared after a successful first send, a later send attempt for that same client (whether or not it is nominally the client's "first" invoice) is blocked again on the same basis, since FEAT-09 re-reads current state at every send rather than remembering that an earlier send once succeeded.
- The indicator shown on Client Detail ("Billing details complete" / "Billing details incomplete -- required before the first invoice") is informational only; it does not itself block anything on FEAT-01.SPEC-004, since billing details can legitimately be added or cleared at any time -- only an actual send attempt triggers this gate's blocking behavior.

## Edge Cases

- **billing_name is filled but billing_address is empty** -- Completeness is false; the send-time block names the specific missing field: "Billing address is required before sending this client's first invoice."
- **Both fields contain only whitespace** -- Treated as empty for completeness purposes; whitespace-only input does not satisfy the non-empty requirement.
- **tax_id is left empty indefinitely** -- Never blocks anything; tax_id has no bearing on completeness at any point.
- **Billing details are completed, the first invoice sends successfully, and Nadia later clears billing_name from Client Detail** -- No invoice already sent is affected (invoices are immutable once sent, XBR-04); a future ad hoc invoice attempt would be blocked again, since FEAT-09's own send-time check re-reads current state and the client currently has billing_name empty, even though a first invoice has technically already gone out. This spec's authority is over the field state at each send attempt, not a one-time-only gate.
- **Two sessions edit billing_name and billing_address concurrently** -- Resolves last-write-wins per the dependency map's Contention note for Client; the completeness indicator reflects whichever save landed last.

## Acceptance Criteria

**FEAT-01.SPEC-010-AC-01:** Given a client has both billing_name and billing_address filled, when Nadia attempts to send that client's first invoice, then the send proceeds with no billing block.

**FEAT-01.SPEC-010-AC-02:** Given a client has billing_name filled but billing_address empty, when Nadia attempts to send that client's first invoice, then the send is blocked with "Billing address is required before sending this client's first invoice."

**FEAT-01.SPEC-010-AC-03:** Given a client has both billing fields empty, when Nadia views Client Detail, then the completeness indicator shows "Billing details incomplete -- required before the first invoice."

**FEAT-01.SPEC-010-AC-04:** Given a client has tax_id empty but both billing_name and billing_address filled, when Nadia attempts to send the first invoice, then the send proceeds, since tax_id never factors into completeness.

**FEAT-01.SPEC-010-AC-05:** Given billing_address contains only whitespace, when completeness is evaluated, then it is treated as empty and the client is incomplete.

**FEAT-01.SPEC-010-AC-06:** Given a client's first invoice has already been sent successfully, when Nadia later clears billing_name on Client Detail, then the already-sent invoice is unaffected, but a subsequent ad hoc invoice attempt for that client is blocked again until billing_name is restored.

**FEAT-01.SPEC-010-AC-07:** Given two sessions of Nadia's edit the same client's billing_address concurrently, when both save, then the later save wins and the completeness indicator reflects it.

**FEAT-01.SPEC-010-AC-08:** Given Owen has no access to billing-detail fields, when he views his portal, then no billing-edit control for his company's details is ever shown to him.

**FEAT-01.SPEC-010-AC-09:** Given Dana views a client in a read-only support session, when she looks at the billing completeness indicator, then she sees its current state but has no control to edit the underlying fields.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Project Stage Derivation

## Overview

**Name:** Project Stage Derivation
**ID:** FEAT-01.SPEC-011
**Type:** Logic/Rule
**Purpose:** Computes the roster/detail "stage" label (Draft, In Progress, Complete, Cancelled, Archived) from the project's proposal, milestone, invoice, completion, and cancellation state.
**Parent Feature:** FEAT-01 -- Client & Project Management
**Governed Entity:** Project (specifically the derived stage field)

## Scope and Non-Goals

**In Scope:**
- The formula that computes a project's stage label from its underlying proposal, milestone, invoice, completed_at, and cancelled_at state
- When the derivation re-runs (on every relevant upstream event)
- The exact precedence when multiple conditions could apply at once

**Non-Goals:**
- Setting or clearing completed_at, cancelled_at, or the project's Archived status themselves -- completed_at is set by FEAT-01.SPEC-006 (via Mark Complete on FEAT-01.SPEC-005); cancelled_at is set exclusively by FEAT-25; the Archived status is set by Archive and cleared by Reactivate, both on FEAT-01.SPEC-005; this spec only reads all three and recomputes stage in response
- Defining Proposal, Milestone, or Invoice status values themselves -- owned by FEAT-02/FEAT-03, FEAT-04/FEAT-08, and FEAT-09 respectively; this spec only reads their status to compute the stage label
- Presenting the stage badge visually -- owned by the screens that display it (FEAT-01.SPEC-003, FEAT-01.SPEC-004, FEAT-01.SPEC-005), which reference this spec's output rather than computing it themselves

## Governed Entity

**Entity:** Project
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| project_name | text | Project's display name |
| client | reference | The owning Client (exactly one) |
| stage | derived (enum: Draft, In Progress, Complete, Cancelled, Archived) | Computed by this spec |
| currency | text (configured via FEAT-15) | Not evaluated by this spec |
| tax_label / tax_rate | text / number (configured via FEAT-15) | Not evaluated by this spec |
| completed_at | timestamp | Read by this spec to derive "Complete" |
| cancelled_at | timestamp | Read by this spec to derive "Cancelled" |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-002 | Create Project | On project creation, sets the initial stage to "Draft" |
| FEAT-01.SPEC-003 | Client & Project Roster | Reads the derived stage to render each project's badge |
| FEAT-01.SPEC-005 | Project Detail | Reads the derived stage to render the badge; re-derives after Mark Complete, Archive, or Reactivate (Reactivate clears the Archived override so the remaining precedence conditions resolve the restored stage) |
| FEAT-01.SPEC-006 | Completion Invoice Trigger | Signals re-derivation after setting completed_at |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| stage | No direct input validation -- stage is never set by direct user input; it is always computed by this spec's derivation formula | Always | On every derivation run | N/A -- there is no user-facing input to validate; an attempt to set stage directly does not exist in this product's design | No |
| project_name, client, currency, tax_label, tax_rate | No validation beyond data type in this spec -- governed elsewhere (FEAT-01.SPEC-002 for project_name/client, FEAT-15 for currency/tax) | Always | -- | -- | -- |
| completed_at | No validation beyond data type in this spec -- write authority belongs to FEAT-01.SPEC-006; this spec only reads it | Always | -- | -- | -- |
| cancelled_at | No validation beyond data type in this spec -- write authority belongs to FEAT-25; this spec only reads it | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Stage derivation formula | status (Archived flag), cancelled_at, completed_at, Proposal.status, Milestone.status (any), Invoice.status (any) | Evaluated in this precedence, highest first: (1) if the project's own status is Archived, stage = "Archived"; (2) else if cancelled_at is set, stage = "Cancelled"; (3) else if completed_at is set, stage = "Complete"; (4) else if any Milestone has been Approved, or any Invoice has been generated for this project, stage = "In Progress"; (5) else if the Proposal has been Accepted, stage = "In Progress"; (6) else stage = "Draft" | N/A -- this is a computed display value, not a user-input field, so it carries no validation error message |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the derived stage on this feature's own screens (roster, Client Detail, Project Detail) | Nadia (Freelancer) | Always | -- |
| View the derived stage on this feature's own screens | Dana (Support Operator) | Always, inside a logged support session (FEAT-31), read-only | -- |
| View the derived stage on this feature's own screens | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never -- the Client & Project Management area is not part of either contact's portal (per the Access Matrix, this capability group is None for both) | Not shown; the same computed stage value they see for their own company's projects is displayed to them separately, through Client Portal Access (FEAT-05), which reads this spec's output but is not governed by this spec |
| Set the stage directly (bypassing derivation) | No role | Never -- the product defines no direct-set capability for this field; stage is always computed | N/A -- no control for this exists anywhere in the product |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| stage | See the Stage derivation formula in Cross-Field Rules above | On project creation, and re-computed on every relevant upstream event (proposal accepted, milestone approved, invoice generated, project marked complete, project cancelled, project archived) | No -- always derived, never directly editable by any role |

## Business Rules

- Data Notes (feature-overview.md): the roster's "stage" label is derived from state owned by other features -- this spec is the single formula every screen depends on, so no screen computes it independently.
- System-driven stage changes (from proposal acceptance or milestone approval) never overwrite a freelancer's explicit Complete or Cancelled transition, per the dependency map's Contention note for Project -- this is why Archived, Cancelled, and Complete take precedence over the "In Progress" conditions in the formula's evaluation order.
- The Cancelled transition is set exclusively by FEAT-25 (dependency map: Project "Updated by ... FEAT-25 (mark cancelled)"); this spec only reads cancelled_at and never sets it.
- The Complete transition is set exclusively by FEAT-01.SPEC-006 via FEAT-01.SPEC-005's Mark Complete action; this spec only reads completed_at and never sets it.
- A project's stage recomputes immediately whenever any input to the formula changes (proposal acceptance, milestone approval, invoice generation, completion, cancellation, archiving, or reactivation), so the roster and detail screens never display a stale label across those transitions once refreshed.
- Reactivating an Archived project (FEAT-01.SPEC-005) clears the Archived override rather than setting a stage directly; the formula then resolves the restored stage from whatever cancelled_at, completed_at, milestone, invoice, and proposal state the project already carries, at the same precedence used for every other derivation -- so a reactivated project always lands on the stage it would show had it never been archived (Cancelled and Complete still outrank the milestone/invoice-driven "In Progress" condition, exactly as they do outside of archiving).

## Edge Cases

- **A project is Archived and also has completed_at set (it was completed, then later archived)** -- "Archived" takes precedence in the formula; the project's stage displays as "Archived," not "Complete," while it is archived. If Nadia later reactivates it from FEAT-01.SPEC-005, the Archived override clears and the formula's next-highest condition applies: since completed_at is still set, the stage resolves to "Complete," not "In Progress" -- reactivation restores the project to the stage it held before archiving, it never re-derives from scratch as if completion had not happened.
- **A project is Cancelled by FEAT-25 after already having an Approved milestone and generated invoices** -- "Cancelled" takes precedence over the milestone/invoice-driven "In Progress" condition; the project's full history of milestones and invoices is preserved and remains reachable, but the stage label reflects the cancellation.
- **A project has an Accepted proposal but no milestones approved and no invoices yet** -- Stage = "In Progress" (condition 5), since acceptance itself moves the project past "Draft" even before any milestone work begins.
- **A project has no proposal at all yet** -- Stage = "Draft" (condition 6, the fallback), matching its state immediately after creation (FEAT-01.SPEC-002).
- **Two upstream events fire in quick succession (e.g., a milestone is approved and, moments later, the project is marked complete)** -- Each derivation run reads the full current state independently; the later run (after completed_at is set) resolves to "Complete" regardless of the milestone approval that fired just before it, since completed_at outranks the milestone/invoice condition in precedence.
- **A voided-and-resent proposal (FEAT-02) leaves a Draft-status proposal alongside an earlier Voided one** -- Only the current, non-voided proposal's status is read by the formula; a Voided proposal is never treated as "Accepted" for derivation purposes, so re-derivation after a void-and-resend correctly falls back to whatever the current proposal's real status is.

## Acceptance Criteria

**FEAT-01.SPEC-011-AC-01:** Given a project has just been created with no proposal yet, when its stage is derived, then it resolves to "Draft."

**FEAT-01.SPEC-011-AC-02:** Given a project's proposal has been accepted and no milestone is yet approved, when its stage is derived, then it resolves to "In Progress."

**FEAT-01.SPEC-011-AC-03:** Given a project has at least one Approved milestone, when its stage is derived, then it resolves to "In Progress" (if not already Complete, Cancelled, or Archived).

**FEAT-01.SPEC-011-AC-04:** Given Nadia marks a project complete via FEAT-01.SPEC-005/SPEC-006, when its stage is next derived, then it resolves to "Complete," taking precedence over any milestone-based "In Progress" condition.

**FEAT-01.SPEC-011-AC-05:** Given FEAT-25 marks a project cancelled, when its stage is next derived, then it resolves to "Cancelled," taking precedence over its milestone and invoice state.

**FEAT-01.SPEC-011-AC-06:** Given a project is Archived, when its stage is derived, then it resolves to "Archived" regardless of whether completed_at or cancelled_at is also set.

**FEAT-01.SPEC-011-AC-07:** Given a project's proposal is voided and re-sent (still unaccepted), when its stage is re-derived, then it does not resolve to "In Progress" on the basis of the voided proposal, since a Voided proposal is never treated as Accepted.

**FEAT-01.SPEC-011-AC-08:** Given a milestone is approved and, moments later, the same project is marked complete, when its stage is derived after both events, then it resolves to "Complete."

**FEAT-01.SPEC-011-AC-09:** Given Owen views his company's project in the client portal, when he checks its stage, then he sees the same derived value Nadia sees on the roster, since derivation is a single, shared formula.

**FEAT-01.SPEC-011-AC-10:** Given Dana views a project in a read-only support session, when she checks its stage, then she sees the current derived value with no ability to set it directly.

**FEAT-01.SPEC-011-AC-11:** Given no role or screen in this product offers a direct control to set stage, when any user looks for one, then none exists anywhere in the product.

**FEAT-01.SPEC-011-AC-12:** Given an Archived project that also has completed_at set is reactivated via FEAT-01.SPEC-005, when its stage is next derived, then it resolves to "Complete" (not "In Progress"), since completed_at still outranks the milestone/invoice condition in precedence once the Archived override is cleared.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
