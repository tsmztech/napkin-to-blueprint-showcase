---
document_type: feature-overview
feature_number: FEAT-01
feature_name: Client & Project Management
feature_slug: client-project-management
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 11
screen_count: 5
automation_count: 2
logic_rule_count: 4
integration_count: 0
notification_count: 0
---

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
