---
document_type: feature-overview
feature_number: FEAT-13
feature_name: Immutable Activity & Audit Trail
feature_slug: immutable-activity-audit-trail
priority_tier: Core
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 6
screen_count: 2
automation_count: 1
logic_rule_count: 3
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Immutable Activity & Audit Trail

## Summary

**Feature:** Immutable Activity & Audit Trail
**ID:** FEAT-13
**Description:** Every proposal acceptance, milestone approval, and sent invoice is recorded permanently with who, what, and when, giving the freelancer a defensible record to point to if a scope dispute happens.
**Priority:** Core
**Phase:** MVP
**Type:** Platform
**Rationale:** BRIEF.md, Success Criteria: "When a scope argument happens, the freelancer can point at the approval record" and Constraints: "accepted proposals, approvals and sent invoices must never be silently edited afterwards." This is the product's evidentiary backbone. MVP phase: without it, none of the record-immutability promises are verifiable. [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Automatic recording — every record-worthy event is logged without manual action
- Chronological project history — the full sequence of what happened, and when, per project
- Permanent record — no entry can be edited or deleted once written
- Share a record — Nadia can produce a printable copy of a project's trail, or of a single entry, to show the client

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-13.SPEC-001 | Activity Trail | Screen | Nadia (Freelancer), Dana (Support Operator) | Nadia browses a project's full chronological activity history; Dana views the same trail read-only during a logged support session |
| FEAT-13.SPEC-002 | Printable Record Copy | Screen | Nadia (Freelancer) | Nadia produces a printable, unalterable copy of the full trail or of one entry to show a client during a dispute |
| FEAT-13.SPEC-003 | Activity Entry Recording | Automation | All | Writes one append-only Activity Log Entry whenever any of ten other features reports a record-worthy event, capturing event type, actor, timestamp, and the affected record |
| FEAT-13.SPEC-004 | Entry Immutability, Content & Attribution Rules | Logic/Rule | All | Governs the required fields every entry must carry and enforces that no entry can ever be edited or deleted once written |
| FEAT-13.SPEC-005 | Activity Trail Access & Visibility Rules | Logic/Rule | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Governs who may see the cross-event trail (Nadia Full, Dana View, Owen/Priya None) versus who sees only the outcome of their own actions |
| FEAT-13.SPEC-006 | Retention & Account-Deletion Purge Rule | Logic/Rule | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Governs how entries are retained for the life of the account, removed only on account deletion subject to legal retention, and how a contact's erasure keeps their name on evidence entries |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Automatic recording — every record-worthy event is logged without manual action | FEAT-13.SPEC-003 | The automation fires on every event reported by the ten writer features and writes the entry with no action from any human | Phase 2 (Explicit) |
| Chronological project history — the full sequence of what happened, and when, per project | FEAT-13.SPEC-001 | The Activity Trail screen renders every entry for a project in chronological order, with the loading, empty, error, and offline states named in the feature's own States field | Phase 2 (Explicit) |
| Permanent record — no entry can be edited or deleted once written | FEAT-13.SPEC-004 | The rule spec enforces append-only behavior at write time and forbids any edit or delete path anywhere in the product | Phase 2 (Explicit) |
| Share a record — Nadia can produce a printable copy of a project's trail, or of a single entry, to show the client | FEAT-13.SPEC-002 | The Printable Record Copy screen renders a client-shareable, unalterable copy of the full trail or a single selected entry | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-13.SPEC-005 | Activity Trail Access & Visibility Rules | Phase 5 (Rule-Constraint Discovery) | The Access field's four fully differentiated behaviors (Nadia Full, Dana View, Owen/Priya None for the cross-event trail but visible outcomes in their own scoped views) exceed the inline-authorization threshold and govern both screens plus the feature's read side of the CRUD matrix |
| FEAT-13.SPEC-006 | Retention & Account-Deletion Purge Rule | Phase 3 (Entity-Lifecycle Analysis) | The Activity Log Entry's CRUD matrix has no in-product Delete/Archive path, which scope-boundaries.md (SC-24) and the dependency map's Data Sensitivity line both require to be an explicit, documented policy rather than a silent omission — including how a contact's erasure request (XBR-27) interacts with entries bearing their name |

## Entity-Lifecycle Coverage Matrix

**Entity: Activity Log Entry** *(this feature's sole managed entity — created only on behalf of ten other features, never on its own initiative)*

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-13.SPEC-003 | The Entry Recording automation writes one entry per reported event from FEAT-03, FEAT-05, FEAT-06, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-18, FEAT-25, FEAT-31, capturing event_type, actor, occurred_at, affected_record, and project | Nothing in this feature ever creates an entry on its own; every write is on another feature's behalf (XBR-05) |
| Read (single) | FEAT-13.SPEC-001, FEAT-13.SPEC-002 | SPEC-001 lets Nadia (and, read-only, Dana) open one entry within the trail; SPEC-002 reads one entry to produce its printable copy | — |
| Read (list) | FEAT-13.SPEC-001 | Activity Trail shows the full chronological list for a project, with a lightweight loading indicator for long histories | No depth limit within a project's lifetime (Validation & Limits field) |
| Update | N/A | No spec ever updates an entry. Enforced as a hard prohibition by FEAT-13.SPEC-004 — this is the feature's defining guarantee, not an omission | — |
| Delete/Archive | N/A in-product; FEAT-24 only | FEAT-13.SPEC-006 — hard delete only, and only as part of full account deletion (FEAT-24), itself subject to legal financial-record retention (scope-boundaries.md, SC-24); no restore path because no in-product delete exists; no cascade because entries reference other records rather than owning them; no automatic time-based purge otherwise — retained for the life of the account | A contact's erasure (FEAT-18) removes their contact details but never their name from an existing entry (XBR-27), governed by SPEC-006 |
| State Transition | N/A | An entry has no status field beyond its single, permanent write; it never transitions between states (product-features.md, Validation & Limits: "no entry can be edited or deleted once written") | — |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Project | FEAT-13.SPEC-001, FEAT-13.SPEC-003 | Every entry belongs to the project the event occurred in; the trail is scoped and viewed per project |
| Proposal | FEAT-13.SPEC-003, FEAT-13.SPEC-002 | Source of acceptance events (FEAT-03) and the record shown in a printable copy during a dispute |
| Milestone | FEAT-13.SPEC-003, FEAT-13.SPEC-002 | Source of approval and reopen events (FEAT-08) and the record shown in a printable copy during a dispute |
| Deliverable | FEAT-13.SPEC-003 | Source of upload and removal events (FEAT-06), including the timestamped upload entry the dispute journey's failure variant relies on |
| Invoice | FEAT-13.SPEC-003, FEAT-13.SPEC-002 | Source of sent-invoice events (FEAT-09) and refund/reversal/cancellation events (FEAT-25) |
| Client Contact | FEAT-13.SPEC-003, FEAT-13.SPEC-005 | Source of first-view events (FEAT-05) and contact role-change events (FEAT-18); identifies the actor on client-side entries |
| Reminder Log | FEAT-13.SPEC-003 | Source of automatic and manual reminder-sent events (FEAT-11) |
| Support Access Session | FEAT-13.SPEC-003, FEAT-13.SPEC-001 | Source of support-session-opened and -closed events (FEAT-31); its start/end times appear in Nadia's trail |
| Payment | FEAT-13.SPEC-003 | Source of manually recorded off-platform payment events (FEAT-10) |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| A proposal is accepted (FEAT-03) | Write an append-only entry: event_type "proposal accepted", actor, timestamp, affected record | Standalone Automation | FEAT-13.SPEC-003 |
| A milestone is approved or reopened (FEAT-08) | Write a separate append-only entry for each — approval and reopen are always distinct, never one overwriting the other | Standalone Automation | FEAT-13.SPEC-003 |
| An invoice is sent, or a credit note is issued (FEAT-09) | Write an append-only entry recording the send or correction | Standalone Automation | FEAT-13.SPEC-003 |
| A deliverable is uploaded or removed (FEAT-06) | Write an append-only entry, including the timestamp the dispute journey's failure variant relies on | Standalone Automation | FEAT-13.SPEC-003 |
| A client contact's first view of a deliverable, proposal, or invoice occurs (FEAT-05) | Write an append-only entry capturing that first-view timestamp | Standalone Automation | FEAT-13.SPEC-003 |
| An automatic or manual reminder is sent (FEAT-11) | Write an append-only entry with the reminder type and send time | Standalone Automation | FEAT-13.SPEC-003 |
| A contact's role changes, or a contact is added or removed (FEAT-18) | Write an append-only entry naming the acting contact and the change | Standalone Automation | FEAT-13.SPEC-003 |
| A refund, reversal, or cancellation is recorded (FEAT-25) | Write an append-only entry preserving the original record rather than altering it | Standalone Automation | FEAT-13.SPEC-003 |
| A payment is manually recorded off-platform (FEAT-10) | Write an append-only entry naming Nadia as the recorder | Standalone Automation | FEAT-13.SPEC-003 |
| A support session opens or closes (FEAT-31) | Write an append-only entry with start/end time and the operator's identity | Standalone Automation | FEAT-13.SPEC-003 |
| A reported event's entry write does not succeed on first attempt | Retry until the write succeeds; the triggering feature's own action is not treated as complete until its entry is durably recorded, since the record itself is the evidence | Inline in SPEC-003 (failure handling) | FEAT-13.SPEC-003 |
| Nadia opens a project's activity trail | Load and display every entry for that project in chronological order | Inline in triggering screen | FEAT-13.SPEC-001 |
| The trail temporarily fails to load | Keep any previously fetched entries visible; nothing is ever removed by a failed request | Inline in triggering screen (Error state) | FEAT-13.SPEC-001 |
| Nadia opens the trail while offline or the connection degrades | Show the most recently loaded trail, read-only | Inline in triggering screen (Offline/Degraded state) | FEAT-13.SPEC-001 |
| Nadia selects a trail entry to share | Render a printable, unalterable copy of that single entry | Inline in triggering screen | FEAT-13.SPEC-002 |
| Nadia requests a printable copy of the whole project trail | Render a printable, unalterable copy of every entry for that project | Inline in triggering screen | FEAT-13.SPEC-002 |
| Owen or Priya attempts to reach the cross-event activity trail | Show nothing — the capability is not shown at all for their roles; they see only the outcome of their own actions within their own scoped views | Standalone Logic/Rule | FEAT-13.SPEC-005 |
| Dana opens a support session on Nadia's account | Show the trail read-only, including this very session's own entry once it is written | Cross-feature (FEAT-31 owns the session), governed inline by SPEC-005 | FEAT-13.SPEC-001 / FEAT-13.SPEC-005 |
| A client contact submits an erasure request (FEAT-18) | Remove the contact's details from Client Contact, but keep their name on every entry that already names them as actor | Standalone Logic/Rule | FEAT-13.SPEC-006 |
| The freelancer deletes her account (FEAT-24) | Remove all activity entries as part of the deletion, except where a legal financial-record retention period requires keeping some | Cross-feature (FEAT-24 owns account deletion), governed inline by SPEC-006 | FEAT-13.SPEC-006 |
| A dispute occurs and Nadia opens the trail, then records a refund/cancellation (FEAT-25) | The refund/cancellation is written as a new entry that preserves, never replaces, the disputed original | Cross-feature (FEAT-25 owns the outcome), the write itself governed by SPEC-003/SPEC-004 | FEAT-13.SPEC-003 |

## Shared Context

**Shared Entities:**
- Activity Log Entry — created only by SPEC-003, on behalf of the ten writer features named in the dependency map; read singly and as a list by SPEC-001; read singly by SPEC-002 to render a printable copy; its required fields and immutability are governed by SPEC-004; its visibility by role is governed by SPEC-005; its retention and purge behavior by SPEC-006. Fields: event_type, actor, occurred_at, affected_record, project.

**Shared UI Patterns:**
- Chronological entry row — the same entry summary (event description, actor, timestamp, a link to the affected record) appears on SPEC-001's trail list and, expanded to full detail, on SPEC-002's printable copy; Spec Writers should describe one visual pattern shown at two densities.
- "Nothing removed on failure" convention — SPEC-001's Error state and Offline/Degraded state both keep the last successfully loaded entries visible rather than clearing the screen; this convention should be described identically in both state annotations.

**Shared Validation:**
- SPEC-004 defines the single source of truth for what every entry must contain and that it can never be edited or deleted. SPEC-003 applies it at write time rather than re-deriving it; SPEC-001 and SPEC-002 rely on it to guarantee that what they display can never have been altered.
- SPEC-005 defines the single source of truth for who may see the trail. SPEC-001 and SPEC-002 both reference it rather than restating the role gate; the ten writer features that call SPEC-003 rely on it to know their own actors will appear correctly scoped.

## Internal Dependency Map

```
FEAT-01 (Client & Project Management) -> [Nadia opens the project's activity area] -> FEAT-13.SPEC-001 (Activity Trail)
FEAT-13.SPEC-001 (Activity Trail) -> [Nadia selects an entry or "share the trail"] -> FEAT-13.SPEC-002 (Printable Record Copy)
FEAT-13.SPEC-003 (Activity Entry Recording) -> [applies required fields and immutability per] -> FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules)
FEAT-13.SPEC-001 (Activity Trail) -> [validates visibility using] -> FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules)
FEAT-13.SPEC-002 (Printable Record Copy) -> [validates visibility using] -> FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules)
FEAT-13.SPEC-003 (Activity Entry Recording) -> [writes an entry Dana's session can be seen through] -> FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules)
FEAT-13.SPEC-006 (Retention & Account-Deletion Purge Rule) -> [governs what SPEC-003's entries retain after] -> FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules)
FEAT-13.SPEC-001 (Activity Trail) -> [Nadia points to a disputed entry, then records the outcome] -> FEAT-25 (Refunds, Reversals & Cancellations) [cross-feature]
```

**Default Entry:** FEAT-13.SPEC-001 (Activity Trail) — the screen Nadia reaches from a project's activity area (FEAT-01), and the screen Dana reaches read-only when a support session opens (FEAT-31).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-13.SPEC-003 | Inbound | FEAT-03 (Proposal Acceptance) | Writes the acceptance entry (XBR-05) | Proposal accepted |
| FEAT-13.SPEC-003 | Inbound | FEAT-05 (Client Portal Access & Magic-Link Login) | Writes the first-view entry (XBR-05) | A client contact's first view of a proposal, deliverable, or invoice |
| FEAT-13.SPEC-003 | Inbound | FEAT-06 (Deliverable Upload & Sharing) | Writes upload and removal entries (XBR-05, XBR-11) | Deliverable uploaded or removed |
| FEAT-13.SPEC-003 | Inbound | FEAT-08 (Milestone Approval) | Writes the approval entry and the separate reopen entry (XBR-05) | Milestone approved or reopened |
| FEAT-13.SPEC-003 | Inbound | FEAT-09 (Invoice Generation & Sending) | Writes the sent-invoice entry (XBR-05) | Invoice sent |
| FEAT-13.SPEC-003 | Inbound | FEAT-10 (Invoice Payment Processing) | Writes the manually recorded payment entry (XBR-05) | Off-platform payment recorded by Nadia |
| FEAT-13.SPEC-003 | Inbound | FEAT-11 (Automated Payment Reminders) | Writes the reminder-sent entry (XBR-05) | Automatic or manual reminder sent |
| FEAT-13.SPEC-003, FEAT-13.SPEC-006 | Inbound | FEAT-18 (Client Contact Management) | Writes the contact role-change entry; erasure requests are governed by the retention rule (XBR-27) | Contact role changed, added, removed, or erased |
| FEAT-13.SPEC-001, FEAT-13.SPEC-002 | Outbound | FEAT-25 (Refunds, Reversals & Cancellations) | Nadia navigates from a disputed trail entry to record the refund or cancellation outcome | Nadia records the outcome of a dispute |
| FEAT-13.SPEC-003 | Inbound | FEAT-25 (Refunds, Reversals & Cancellations) | Writes the refund/reversal/cancellation entry, preserving the original (XBR-04, XBR-05) | Refund, reversal, or cancellation recorded |
| FEAT-13.SPEC-003, FEAT-13.SPEC-001 | Inbound/Outbound | FEAT-31 (Operator Support Access) | Writes the session-opened/closed entries; Dana views the trail read-only inside her own session (XBR-29) | Support session opened or closed |
| FEAT-13 (feature-level) | Outbound | FEAT-29 (In-App Notification Center) | Supplies recent activity entries the notification center surfaces to Nadia | Entry written |
| FEAT-13 (feature-level) | Outbound | FEAT-24 (Data Export & Account Deletion) | Activity entries are included in a data-export archive and removed on account deletion, subject to legal retention (SC-24) | Export generated, or account deleted |
| FEAT-13 (feature-level) | Inbound | FEAT-01 (Client & Project Management) | The project view's activity area is the entry point into this feature | Nadia opens a project's activity tab |

## Non-Functional Notes

**Data volumes / growth:** Activity Log Entry has no depth limit within a project's lifetime (product-features.md, Validation & Limits) and accumulates across the same scale as the rest of the product — a few thousand freelancers in year one, each with 3–15 active clients (assumptions-constraints.md, ASMP-22); the trail must stay usable as a single project's history grows into the hundreds of entries over a long engagement.

**Responsiveness:** Entries appear in the trail effectively as soon as the triggering event completes, with no perceptible delay between the event and its record (feature's own Primary Flows & Alternates); the trail shows a lightweight loading indicator rather than a blank screen for projects with long histories (feature's own States field; assumptions-constraints.md, ASMP-27).

**Data sensitivity / privacy:** Every entry carries the actor's identity — the freelancer, a named client contact, or the operator — which is personal data treated as GDPR-class (assumptions-constraints.md, ASMP-24); the entries themselves are immutable evidentiary content (assumptions-constraints.md, ASMP-25), visible in full only to Nadia and read-only to Dana (feature-dependency-map.md, Entity: Activity Log Entry, Data Sensitivity), and strictly isolated so no client company ever sees another's trail (assumptions-constraints.md, ASMP-23).

**Compliance flags:** GDPR-class handling applies to every actor identity captured in an entry; a contact's erasure request removes their personal details but keeps their name on entries that are evidence, per the product's data-subject-erasure-versus-evidence balance (feature-dependency-map.md, XBR-27; assumptions-constraints.md, ASMP-24). Entries are retained for the life of the freelancer's account and removed on deletion only subject to any legal financial-record retention period (scope-boundaries.md, SC-24).

## Non-Goals

- **Editing or deleting an activity entry, by any role, at any time** — Excluded per BRIEF.md's Constraints ("accepted proposals, approvals and sent invoices must never be silently edited afterwards") and assumptions-constraints.md (ASMP-25): this is the feature's defining guarantee, not a missing capability; every correction is a new, separate entry (XBR-04).
- **The full cross-event trail for Owen or Priya** — Excluded per the Access Matrix in user-persona.md: client contacts have "None" for Activity & Audit Trail and see only the outcome of their own actions within their own scoped views; no additional client-side tier is modeled without a brief signal for one (scope-boundaries.md, SC-02).
- **Any edit, annotate, or delete capability for Dana** — Excluded per scope-boundaries.md (SC-04): the operator's support access is read-only everywhere, including the trail; Dana can view but never act on an entry.
- **Bulk import of historical activity from another tool** — Excluded per scope-boundaries.md (SC-19): an imported record was never generated through Clientroom's own event pipeline and so could not carry this feature's evidence guarantee; every entry in the product is written only by FEAT-13.SPEC-003 in response to a genuine in-product event.
- **Automatic time-based purge of activity entries** — Intentional lifecycle decision confirmed by the CRUD matrix: entries are retained for the life of the freelancer's account with no automatic purge (scope-boundaries.md, SC-24); they are removed only by account deletion (FEAT-24), subject to legal financial-record retention. This feature makes no purge decision of its own outside that boundary.
