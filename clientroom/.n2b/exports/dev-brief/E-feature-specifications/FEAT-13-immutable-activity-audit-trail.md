# FEAT-13 — Immutable Activity & Audit Trail

This chapter covers Immutable Activity & Audit Trail, a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 6 specifications carrying 101 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-13.SPEC-001 | Activity Trail | screen | 15 |
| FEAT-13.SPEC-002 | Printable Record Copy | screen | 12 |
| FEAT-13.SPEC-003 | Activity Entry Recording | automation | 23 |
| FEAT-13.SPEC-004 | Entry Immutability, Content & Attribution Rules | logic-rule | 20 |
| FEAT-13.SPEC-005 | Activity Trail Access & Visibility Rules | logic-rule | 15 |
| FEAT-13.SPEC-006 | Retention & Account-Deletion Purge Rule | logic-rule | 16 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Activity Trail

## Overview

**Name:** Activity Trail
**ID:** FEAT-13.SPEC-001
**Type:** Screen
**Purpose:** Nadia browses the full, permanent chronological history of everything recorded against a project; Dana views the identical trail read-only while a support session is open.
**Parent Feature:** FEAT-13 -- Immutable Activity & Audit Trail

## Scope and Non-Goals

**In Scope:**
- A reverse-chronological list of every Activity Log Entry for one project
- Opening an entry's affected record from the trail
- Launching the printable/shareable copy of the whole trail or a single entry
- Empty, loading, error, and offline/degraded states
- Dana's read-only view of the identical trail during an open support session, including that session's own entries once written

**Non-Goals:**
- Editing, retracting, or annotating any entry, by any role -- excluded per BRIEF.md's Constraints and governed by FEAT-13.SPEC-004: no entry can ever be edited or deleted once written; this is the feature's defining guarantee, not a missing capability.
- Producing the printable, shareable copy itself -- handled by FEAT-13.SPEC-002 (Printable Record Copy); this screen only launches it.
- Any cross-event trail view for Owen or Priya -- excluded per the Access Matrix (user-persona.md): client contacts have "None" for Activity & Audit Trail; they see only the outcome of their own actions within their own scoped views (e.g., FEAT-03, FEAT-08), never this screen.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01 (Client & Project Management) project view, activity area | Nadia opens a project's activity tab | Project reference; trail loads scoped to that project |
| FEAT-31 (Operator Support Access) support session opened | Dana's open support session grants read-only reach into the freelancer's account | Freelancer account and project context, read-only |
| FEAT-13.SPEC-002 (Printable Record Copy) | Nadia returns after producing or viewing a printable copy | Project reference; trail re-loads at its current state |
| FEAT-31.SPEC-007 (Support Session Opened Notice) | Nadia taps the notice email's "View activity trail" CTA | Freelancer account reference; the trail shows her account's support-session entries |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, her own projects only | Open any entry, navigate to the affected record, launch the printable copy (full trail or a single entry) | -- |
| Owen (Client Primary Contact) | No | No | The capability is not shown at all -- no tab, link, or navigation path from his portal view reaches this screen; he sees only the outcome of his own actions within his own scoped views (e.g., his acceptance confirmation in FEAT-03, his approval confirmation in FEAT-08). |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- not shown at all; she sees only the outcome of her own comment activity within her own scoped views. |
| Dana (Support Operator) | Full screen, read-only, only inside an open support session on the freelancer's account (FEAT-31) | None -- no entry is actionable, no Share control is rendered | Outside an open support session, this screen is unreachable, identical to the unauthenticated experience below. |
| Unauthenticated | No | No | Redirected to the sign-in screen; no project context is retained. |
| Expired session | No (trail hidden behind a re-authentication prompt) | No | Dialog "Your session has expired. Sign in to continue." The previously loaded trail (for Nadia) is held in memory and re-displayed once she signs back in -- there is no unsaved input to lose on this read-only screen. |

## Layout and Content

**Header:** A breadcrumb reading "{Project Name} > Activity," the screen title "Activity Trail," and, for Nadia only, a "Share" action button at the top right that opens FEAT-13.SPEC-002 scoped to the whole trail.

**Body:** A single reverse-chronological list of entries, most recent first, with no depth limit within the project's lifetime. Each row shows:
- A plain-language event description derived from `event_type` (e.g., "Milestone 'Homepage design' approved," "Invoice #INV-014 sent," "Deliverable 'Hero video v2' uploaded")
- The actor's name -- "You" for Nadia's own actions, the client contact's name for client-side events, "Dana (operator)" for support-session entries, or "Automatic" for system-timed events such as reminders
- The exact date and time, shown in the viewer's own time zone
- A link to the affected record, where one exists, to view its current detail (e.g., the milestone, invoice, or deliverable)
- For Nadia only, a "Share this record" control that opens FEAT-13.SPEC-002 pre-scoped to that single entry

A lightweight loading indicator (a slim progress bar directly under the header) appears while the current page of entries is still being fetched, in place of a blank screen, for projects with long histories.

### Responsive Behavior
- **Compact breakpoint:** Single-column list, full width. Within each row, the event description, actor, and timestamp stack vertically; the affected-record link and the per-entry Share control sit below as separate tap targets.
- **Medium size class and above:** The list stays single-column, but each row lays its description, actor, timestamp, and controls out along one line. The list is capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.
- **Header Share button:** Remains visible in the header at every size class -- never collapsed into an overflow menu.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Entry row | Tap | Navigates to the affected record's own detail spec (e.g., FEAT-08 milestone detail, FEAT-09 invoice detail, FEAT-06 deliverable detail) | Row briefly highlights before navigation | Standard transition to the destination screen |
| Entry row's affected-record link | Tap | Same destination as the row tap -- an explicit link target | -- | -- |
| Per-entry "Share this record" control (Nadia only) | Tap | Navigates to FEAT-13.SPEC-002, pre-scoped to this single entry | Screen transitions | FEAT-13.SPEC-002 opens showing only this entry |
| Header "Share" button (Nadia only) | Tap | Navigates to FEAT-13.SPEC-002, scoped to the full project trail | Screen transitions | FEAT-13.SPEC-002 opens showing every entry for the project |
| Entry list | Scroll | Loads the next page of older entries | List grows | A lightweight loading indicator appears at the list's end while more entries fetch |
| Breadcrumb "{Project Name}" | Tap | Navigates to FEAT-01 (project view) | Screen closes | Standard transition back to the project |

### Accessibility Notes

- **Focus order:** Breadcrumb -> Share button (Nadia only) -> entry list, row by row: description -> actor -> timestamp -> affected-record link -> per-entry Share control (Nadia only).
- **Dynamic-change announcements:** The loading indicator's appearance, an error banner's appearance, and the arrival of newly fetched entries (both at scroll end and when a fresh entry appears live at the top) are announced to assistive technology.
- **Keyboard alternatives:** Every action on this screen (entry navigation, both Share controls, scroll-triggered loading) is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | Explanatory message that entries will appear as milestones progress, in place of a list | The project has zero Activity Log Entries | The first entry is written for this project |
| Loading | Slim progress indicator under the header; no entries rendered yet | Screen opens, or the viewer switches to a different project's trail | The first page of entries returns |
| Populated | Full reverse-chronological list as described in Layout and Content | Entries load successfully | Viewer navigates away |
| Error | Error banner at the top of the list ("This trail couldn't be fully refreshed. Showing the most recently loaded activity."); any previously fetched entries remain visible underneath -- nothing is ever removed by a failed request | A refresh or scroll-triggered fetch fails | A later fetch succeeds, or the viewer navigates away |
| Offline/Degraded | The most recently loaded trail remains visible, read-only, under a banner "You're offline -- showing the last loaded activity."; no new entries load and affected-record links are disabled while offline | Connectivity is lost while the screen is open, or the screen is opened while already offline | Connectivity returns and the trail re-fetches |

## Validation Rules

**Option B -- Inline (no user input on this screen):**
This screen accepts no field input; every action is navigation. It reads, and never writes, Activity Log Entry data, which is governed entirely by FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Entry row or affected-record link tap | The affected record's own detail spec (varies by `event_type`: e.g., FEAT-08 Milestone Detail, FEAT-09.SPEC-002 Invoice Detail, FEAT-06 Deliverable Detail) | Varies |
| Per-entry Share tap | FEAT-13.SPEC-002 (Printable Record Copy), single-entry scope | -- |
| Header Share tap | FEAT-13.SPEC-002 (Printable Record Copy), full-trail scope | -- |
| Breadcrumb tap | FEAT-01 project view | FEAT-01 |
| Nadia points to a disputed entry, then records the outcome | Mark invoice refunded / project cancelled | FEAT-25 (Refund & Cancelled Project Handling) |

## Data Model

**Creates:** None -- this screen never writes an Activity Log Entry.
**Reads:** Activity Log Entry -- `event_type`, `actor`, `occurred_at`, `affected_record`, and `project`, for the full list of entries belonging to the current project, in reverse-chronological order.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-04 and XBR-05: every entry shown here was written append-only by FEAT-13.SPEC-003 and can never have been edited or deleted afterward -- what this screen displays is guaranteed unaltered, per FEAT-13.SPEC-004.
- Visibility of this screen, and of every entry on it, is governed entirely by FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules); this screen enforces no separate access logic of its own.
- XBR-29: Dana's support-session view of this screen includes that session's own opened/closed entries once they are written, so Nadia's later view of the same trail shows exactly what Dana saw during her session.

## Edge Cases

- **Nadia opens the trail for a project with an unusually long history (hundreds of entries over a long engagement)** -- The list loads in pages as she scrolls; the lightweight loading indicator appears at the list's end rather than blocking the whole screen.
- **An entry's affected record has since changed state (e.g., a reopened milestone, a corrected invoice)** -- The entry's own text and timestamp never change; tapping its link navigates to the affected record's current state, which may show a later status than the entry describes. This is expected: the entry is a historical record, not a live mirror of the affected record.
- **Dana's support session closes while she is viewing the trail** -- The screen becomes immediately unreachable, matching the "outside an open session" unauthorized experience in Access and Visibility.
- **A new entry is written while Nadia (or Dana, mid-session) has the trail open** -- The trail is a live view: the new entry appears at the top of the list without a manual refresh. There is no conflict to resolve, since entries are append-only (per the dependency map's Contention note for Activity Log Entry: "None -- entries are append-only and never edited or deleted by anyone; concurrent writers only append independent entries").
- **A brand-new project's trail is opened before its first entry is written** -- The Empty state renders; it is not treated as an error.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-13.SPEC-002 (Printable Record Copy) | Navigation (outbound) | Nadia launches a shareable copy of the whole trail or a single entry |
| FEAT-13.SPEC-003 (Activity Entry Recording) | References (inbound) | Every entry rendered here was written by this automation |
| FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules) | References (inbound) | Governs what every entry must contain and guarantees it is never altered |
| FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules) | References (inbound) | Governs exactly who may open this screen and see its entries |
| FEAT-01 (Client & Project Management) | Navigation (inbound) | Entry point via the project's activity area |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana reaches this screen only from an open support session |
| FEAT-25 (Refund & Cancelled Project Handling) | Navigation (outbound) | Nadia records a dispute's outcome after pointing to a disputed entry |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| activity_trail_viewed | project reference, viewer role (Nadia / Dana), entry count shown | The screen finishes loading its first page of entries | supports success-metrics.md: "Dispute Resolution Confidence" |
| activity_entry_opened | event_type, time elapsed since occurred_at | Nadia or Dana taps an entry's affected-record link | supports success-metrics.md: "Dispute Resolution Confidence" |
| activity_trail_empty_state_shown | project reference | The Empty state renders for a brand-new project | N/A -- no success-metrics.md metric measures the empty state itself; retained because product-features.md's Signals field for this feature already names `activity_trail_viewed` as its core viewing signal, and this event distinguishes the zero-entry case in the underlying event data without claiming an additional metric |

## Acceptance Criteria

**FEAT-13.SPEC-001-AC-01:** Given Nadia opens a project with existing activity, when the screen finishes loading, then she sees every entry for that project listed most-recent-first, each with its description, actor, exact timestamp, and affected-record link.

**FEAT-13.SPEC-001-AC-02:** Given Nadia is viewing the trail, when she taps an entry's affected-record link, then she is taken to that record's own detail screen.

**FEAT-13.SPEC-001-AC-03:** Given Nadia is viewing the trail, when she taps the header "Share" button, then FEAT-13.SPEC-002 opens scoped to the full project trail.

**FEAT-13.SPEC-001-AC-04:** Given Nadia is viewing the trail, when she taps "Share this record" on a single entry, then FEAT-13.SPEC-002 opens scoped to that one entry only.

**FEAT-13.SPEC-001-AC-05:** Given Nadia scrolls to the end of the currently loaded entries in a project with a long history, when more entries exist, then the next page loads with a lightweight indicator at the list's end.

**FEAT-13.SPEC-001-AC-06:** Given Nadia taps the breadcrumb, when the tap registers, then she returns to the FEAT-01 project view.

**FEAT-13.SPEC-001-AC-07:** Given Nadia opens a brand-new project with no recorded events, when the screen loads, then she sees the Empty state explaining entries will appear as milestones progress.

**FEAT-13.SPEC-001-AC-08:** Given Nadia has a trail loaded and a refresh fails, when the failure occurs, then an error banner appears while every previously loaded entry remains visible underneath.

**FEAT-13.SPEC-001-AC-09:** Given Nadia loses connectivity while the trail is open, when connectivity drops, then the last-loaded trail remains visible read-only under an offline banner, and affected-record links are disabled until connectivity returns.

**FEAT-13.SPEC-001-AC-10:** Given Owen or Priya is signed into the client portal, when they look for any way to reach the cross-event activity trail, then no tab, link, or navigation path to this screen exists anywhere in their portal view.

**FEAT-13.SPEC-001-AC-11:** Given Dana has an open support session on Nadia's account, when she opens the activity trail, then she sees the same full trail read-only, with no Share control and no actionable entries.

**FEAT-13.SPEC-001-AC-12:** Given Dana's support session closes while she is viewing the trail, when the session ends, then the screen becomes immediately unreachable to her.

**FEAT-13.SPEC-001-AC-13:** Given Dana's support session opens and closes during a session, when Nadia later opens the same trail, then she sees the session's own opened and closed entries in the list, attributed to Dana (operator).

**FEAT-13.SPEC-001-AC-14:** Given an unauthenticated visitor attempts to reach this screen directly, when the request is made, then they are redirected to sign-in with no project context retained.

**FEAT-13.SPEC-001-AC-15:** Given Nadia's session expires while the trail is open, when she next interacts with the screen, then a "Your session has expired. Sign in to continue." dialog appears and, once she signs back in, the same trail re-displays without her having lost any input (none existed to lose).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 4 (empty, error, offline, expired-session behavior) | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Printable Record Copy

## Overview

**Name:** Printable Record Copy
**ID:** FEAT-13.SPEC-002
**Type:** Screen
**Purpose:** Nadia produces a printable, unalterable copy of a project's full activity trail, or of one selected entry, to show a client during a scope dispute.
**Parent Feature:** FEAT-13 -- Immutable Activity & Audit Trail

## Scope and Non-Goals

**In Scope:**
- Rendering a print-ready copy of every entry in a project's trail
- Rendering a print-ready copy of a single selected entry
- Loading, ready, and error states for producing the copy
- The copy's own identifying header (freelancer business identity, client, project) so it is self-contained when shown or handed to someone outside the product

**Non-Goals:**
- Browsing or selecting entries from the full trail -- handled by FEAT-13.SPEC-001 (Activity Trail); this screen only renders the copy for the scope it is opened with.
- Emailing or otherwise transmitting the copy to the client -- product-features.md's Communications field for this feature states the trail sends no notifications of its own; Nadia shows or hands the printed/saved copy to the client herself, outside the product's messaging channels.
- Any capability for Owen, Priya, or Dana to produce a copy -- excluded per the Access Matrix (user-persona.md): Activity & Audit Trail is "Full" for Nadia only; Dana's access is View-only with no export path (Access Matrix, Financial Dashboard & Accounting Export row precedent: "View... no export generation"), and client contacts have "None."

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-13.SPEC-001 (Activity Trail) | Nadia taps the header "Share" button | Project reference; scope = full trail |
| FEAT-13.SPEC-001 (Activity Trail) | Nadia taps "Share this record" on one entry | Project reference and entry reference; scope = single entry |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen, her own projects only | Produce the printable copy (full trail or single entry) and print or save it | -- |
| Owen (Client Primary Contact) | No | No | The capability is not shown at all -- no path from his portal view reaches this screen; he only ever sees the copy Nadia herself chooses to show or hand him outside the product. |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- not shown at all. |
| Dana (Support Operator) | No | No | Not reachable from a support session -- support access is View-only on the trail (FEAT-13.SPEC-001) with no export or copy-generation capability, per scope-boundaries.md (SC-04): the operator never generates exports on the freelancer's behalf. |
| Unauthenticated | No | No | Redirected to the sign-in screen; no scope context is retained. |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- since this screen persists no in-progress input, Nadia simply re-opens it from FEAT-13.SPEC-001 after signing back in. |

## Layout and Content

**Header:** Screen title -- "Activity Record" for a single-entry copy, or "Activity Trail -- {Project Name}" for a full-trail copy -- with a back control (returns to FEAT-13.SPEC-001) and a "Print / Save" action button.

**Body:** A self-contained, print-ready page, laid out for both on-screen reading and printing:
- **Identifying header block:** Nadia's business name (from her Freelancer Account business details), the client company name, the project name, and the date the copy was produced.
- **Entry content, full trail scope:** every entry for the project, in the same reverse-chronological order as FEAT-13.SPEC-001, each shown at full detail: complete event description, actor's full name and role, exact date and time, and the affected record's identifying reference (e.g., invoice number, milestone name).
- **Entry content, single-entry scope:** the one selected entry, shown at the same full detail, with no other entries present.
- **Footer:** a fixed statement that the copy reflects an append-only, unalterable record as maintained by the product, plus the copy's production timestamp.

No field on this screen is editable -- every element is read-only, rendered content.

### Responsive Behavior
- **Compact breakpoint:** Single-column page; the identifying header block stacks above the entry content; each entry's description, actor, and timestamp stack vertically, matching FEAT-13.SPEC-001's compact entry-row layout at its expanded density.
- **Medium size class and above:** The page is capped at a consistent platform-wide reading/print width and centered; entry rows lay their description, actor, timestamp, and reference out along one line, matching FEAT-13.SPEC-001's row pattern at higher density (full detail rather than the trail's summary line).
- **Print output:** Produces one continuous document containing the identifying header, every included entry, and the footer statement, independent of screen size -- the print layout is not clipped to the viewport.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back control | Tap | Navigates to FEAT-13.SPEC-001 (Activity Trail) | Screen closes | Standard transition back to the trail |
| "Print / Save" button | Tap | Produces a printable file of the currently rendered copy (full trail or single entry, matching what is on screen) | Button shows a brief "Preparing your copy..." indicator | A file is offered for save or the platform's print dialog opens; on completion the button returns to its normal state |
| "Print / Save" button (while preparing) | Tap | No action -- debounced | None | Button remains in its "Preparing..." state |

### Accessibility Notes

- **Focus order:** Back control -> "Print / Save" button -> identifying header block -> entry content, top to bottom (each entry: description -> actor -> timestamp -> reference) -> footer statement.
- **Dynamic-change announcements:** The "Preparing your copy..." state and its completion (file offered, or an error) are announced to assistive technology.
- **Keyboard alternatives:** Both the back control and "Print / Save" are reachable and operable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Identifying header block visible; entry content area shows a lightweight loading indicator | Screen opens from FEAT-13.SPEC-001 | The scoped entry or entries finish loading |
| Ready | Full copy rendered as described in Layout and Content, "Print / Save" button enabled | Content loads successfully | Nadia navigates away, or taps "Print / Save" |
| Preparing | "Print / Save" button shows a "Preparing your copy..." indicator; rendered content remains visible underneath | Nadia taps "Print / Save" | The file is offered, the print dialog opens, or preparation fails |
| Error | Error message in place of the entry content area: "This record couldn't be loaded. Try again." with a Retry control; the identifying header block remains visible | Loading the scoped entry or entries fails | Nadia taps Retry and loading succeeds, or she navigates away |
| Offline/Degraded | N/A -- this screen only ever renders an entry or entries already loaded into the session from FEAT-13.SPEC-001's Populated state moments earlier; the same "This record couldn't be loaded. Try again." Error state covers the case where the scoped content cannot be fetched, including while offline. | -- | -- |

## Validation Rules

**Option B -- Inline (no user input on this screen):**
This screen accepts no field input; it only renders previously written, immutable Activity Log Entry content. There is nothing for the user to enter or for this screen to validate.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back control tap | FEAT-13.SPEC-001 (Activity Trail) | -- |
| "Print / Save" completes | Remains on this screen (Ready state) | -- |

## Data Model

**Creates:** None -- this screen never writes an Activity Log Entry; it renders a presentational copy only.
**Reads:** Activity Log Entry -- `event_type`, `actor`, `occurred_at`, `affected_record` for the entry or entries in scope; Project (name, owned by FEAT-01); Client (company name, owned by FEAT-01); Freelancer Account (business name, owned by FEAT-21), for the identifying header block.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-04: the copy this screen produces is a faithful rendering of entries that are themselves immutable (FEAT-13.SPEC-004) -- the copy carries no editing capability, and reproducing it does not alter the underlying entry in any way.
- Visibility of this screen is governed entirely by FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules); this screen enforces no separate access logic of its own.
- The scope (full trail vs. single entry) is fixed by which control Nadia tapped on FEAT-13.SPEC-001 -- this screen offers no way to change scope after opening; she returns to FEAT-13.SPEC-001 and re-launches with a different scope instead.

## Edge Cases

- **Nadia opens the full-trail copy for a project with a very long history** -- Every entry is included; the print output spans multiple pages as needed rather than truncating.
- **An entry's affected record is later deleted or superseded after the copy is produced** -- The copy already reflects the entry's permanent content (event description, actor, timestamp, reference) at production time and is never affected by later changes to the affected record, since the entry itself never changes.
- **Nadia taps "Print / Save" twice in rapid succession** -- The second tap is ignored while the first preparation is in progress (button in its "Preparing..." state).
- **The scoped entry no longer resolves (e.g., a stale link reused after navigating away and back with a different project loaded)** -- The Error state renders: "This record couldn't be loaded. Try again." with Retry, which re-fetches using the original scope context.
- **Nadia produces a copy, then the underlying project's trail gains new entries before she shows the copy to the client** -- The already-produced copy is unaffected; it is a snapshot at production time. If she wants the new entries included, she returns to FEAT-13.SPEC-001 and produces a fresh copy.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-13.SPEC-001 (Activity Trail) | Navigation (inbound) | Nadia arrives here after choosing to share the full trail or a single entry |
| FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules) | References (inbound) | Guarantees the content this screen renders is unaltered |
| FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules) | References (inbound) | Governs exactly who may open this screen |
| FEAT-01 (Client & Project Management) | References (inbound) | Owns Project and Client -- this screen reads the project name and client company name from FEAT-01's records for the identifying header block |
| FEAT-21 (Settings & Account Management) | References (inbound) | Owns Freelancer Account -- this screen reads Nadia's business name from FEAT-21's records for the identifying header block |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| activity_record_shared | scope (full_trail / single_entry), entry count included, project reference | "Print / Save" completes successfully (file offered or print dialog opened) | supports success-metrics.md: "Dispute Resolution Confidence" |
| activity_record_share_failed | scope, failure point (load / preparation) | The Error state renders, or "Preparing your copy..." fails to complete | N/A -- no success-metrics.md metric measures the failure path directly; retained because product-features.md's Signals field for this feature names `activity_record_shared` as a required signal, and this companion event lets the success rate implied by "Dispute Resolution Confidence" be computed from event data without inflating the cited metric's own definition |

## Acceptance Criteria

**FEAT-13.SPEC-002-AC-01:** Given Nadia taps "Share" on the full trail from FEAT-13.SPEC-001, when this screen opens, then it shows every entry for the project at full detail, most-recent-first, under an identifying header naming her business, the client, and the project.

**FEAT-13.SPEC-002-AC-02:** Given Nadia taps "Share this record" on one entry from FEAT-13.SPEC-001, when this screen opens, then it shows only that one entry at full detail, with no other entries present.

**FEAT-13.SPEC-002-AC-03:** Given Nadia is viewing a Ready copy, when she taps "Print / Save," then the button shows "Preparing your copy..." and, on completion, a file is offered for save or the print dialog opens.

**FEAT-13.SPEC-002-AC-04:** Given Nadia is viewing a Ready copy, when she taps "Print / Save" a second time while preparation is still in progress, then the second tap has no effect and the button remains in its "Preparing..." state.

**FEAT-13.SPEC-002-AC-05:** Given Nadia taps the back control, when the tap registers, then she returns to FEAT-13.SPEC-001 (Activity Trail).

**FEAT-13.SPEC-002-AC-06:** Given this screen's scoped content fails to load, when the failure occurs, then the Error state shows "This record couldn't be loaded. Try again." with a Retry control, while the identifying header block remains visible.

**FEAT-13.SPEC-002-AC-07:** Given the Error state is showing, when Nadia taps Retry and loading succeeds, then the Ready state renders with the originally requested scope.

**FEAT-13.SPEC-002-AC-08:** Given Owen or Priya is signed into the client portal, when they look for any way to reach this screen, then no path exists anywhere in their portal view.

**FEAT-13.SPEC-002-AC-09:** Given Dana has an open support session on Nadia's account, when she views the activity trail, then no control to reach this screen is rendered anywhere in her session.

**FEAT-13.SPEC-002-AC-10:** Given Nadia produces a full-trail copy for a project with hundreds of entries, when she taps "Print / Save," then the resulting output includes every entry, spanning multiple pages as needed.

**FEAT-13.SPEC-002-AC-11:** Given Nadia has already produced a copy and new entries are later written to the project's trail, when she views the previously produced copy again, then it still shows only the entries present at the time it was produced.

**FEAT-13.SPEC-002-AC-12:** Given an unauthenticated visitor attempts to reach this screen directly, when the request is made, then they are redirected to sign-in with no scope context retained.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 3 (loading, preparing, error) | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Automation Spec: Activity Entry Recording

## Overview

**Name:** Activity Entry Recording
**ID:** FEAT-13.SPEC-003
**Type:** Automation
**Purpose:** Writes one append-only Activity Log Entry whenever any of eleven other features reports a record-worthy event, capturing event type, actor, timestamp, and the affected record.
**Parent Feature:** FEAT-13 -- Immutable Activity & Audit Trail

## Scope and Non-Goals

**In Scope:**
- Receiving a record-worthy event reported by any of the eleven writer features (FEAT-03, FEAT-05, FEAT-06, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-18, FEAT-23, FEAT-25, FEAT-31)
- Composing and durably writing exactly one append-only Activity Log Entry per genuinely new event
- Retrying a failed write until it succeeds, and holding the triggering feature's own action as not-yet-complete until it does
- Idempotent handling of a duplicate report of the identical event

**Non-Goals:**
- Deciding what content each entry must contain (required fields, attribution rules) -- owned by FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules); this automation applies those rules, it does not define them.
- Deciding who may see the resulting entries -- owned by FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules).
- Originating any event on its own initiative -- excluded per the Brief's Entity-Lifecycle Coverage Matrix: this feature's sole managed entity is "created only on behalf of other features, never on its own initiative" (eleven writer features, including FEAT-23 for plan events); this automation only responds to reports, it never decides that something record-worthy has happened.
- Removing or purging entries -- excluded per scope-boundaries.md (SC-24) and governed instead by FEAT-13.SPEC-006 (Retention & Account-Deletion Purge Rule); this automation only ever appends.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A proposal is accepted | FEAT-03 (Proposal Acceptance) | Fires the moment an acceptance is recorded | Proposal reference, accepting contact, acceptance timestamp, project |
| A milestone is approved | FEAT-08 (Milestone Approval) | Fires the moment an approval is recorded | Milestone reference, approving contact, approval timestamp, project |
| A milestone is reopened | FEAT-08 (Milestone Approval) | Fires the moment Nadia reopens a previously approved milestone | Milestone reference, Nadia as actor, reopen timestamp, project |
| An invoice is sent | FEAT-09 (Invoice Generation & Sending) | Fires the moment an invoice -- automatic or ad hoc -- is sent | Invoice reference, triggering event (deposit / milestone / completion / ad hoc), send timestamp, project |
| A credit note is issued | FEAT-09 (Invoice Generation & Sending) | Fires the moment a correcting credit note is issued against a prior invoice | Credit note reference, corrected invoice reference, Nadia as actor, issue timestamp, project |
| A deliverable is uploaded | FEAT-06 (Deliverable Upload & Sharing) | Fires the moment an upload completes successfully | Deliverable reference, Nadia as actor, upload-completed timestamp, project |
| A deliverable is removed | FEAT-06 (Deliverable Upload & Sharing) | Fires the moment a deliverable withdrawal completes | Deliverable reference, Nadia as actor, removal timestamp, project |
| A client contact's first view of a proposal, deliverable, or invoice occurs | FEAT-05 (Client Portal Access & Magic-Link Login) | Fires only on the first view by a given contact of a given record -- never on subsequent views | Affected record reference, viewing contact, first-view timestamp, project |
| A reminder is sent (automatic or manual) | FEAT-11 (Automated Payment Reminders) | Fires the moment a reminder send completes | Invoice reference, reminder type (day 3 / day 10 / manual), actor ("Automatic" or Nadia for a manual send), send timestamp, project |
| A contact's role changes, or a contact is added or removed | FEAT-18 (Client Contact Management & Roles) | Fires the moment the change is saved | Contact reference, acting party (Nadia or the inviting Primary contact), description of the change, timestamp, project (where the client has one) |
| A refund, reversal, or cancellation is recorded | FEAT-25 (Refund & Cancelled Project Handling) | Fires the moment the outcome is recorded | Invoice or project reference, actor (Nadia, or "Automatic" for a processor-reported reversal), timestamp, project |
| A payment is manually recorded off-platform | FEAT-10 (Invoice Payment Processing) | Fires the moment Nadia records the payment | Invoice reference, Nadia as actor, recorded-at timestamp, project |
| A subscription plan is created | FEAT-23.SPEC-002 (Free Plan Auto-Provisioning) | Fires after the plan record is committed at account creation; FEAT-23 never waits on or reverses its plan record for this write | Plan reference, actor "Automatic", tier (Free), status (Active), creation timestamp; account-level (no project) |
| A subscription plan's tier or status changes | FEAT-23.SPEC-004 (Plan State Sync) | Fires after every committed tier or status change (upgrade, downgrade, charge failed, charge recovered, plan freed or lapsed at period end, lapse after grace window); not fired for retry attempts that change nothing or for routine renewals | Plan reference, prior and new tier/status, actor ("Automatic", or Nadia where her action caused the change), change timestamp; account-level (no project) |
| A downgrade offer is raised | FEAT-23.SPEC-005 (Downgrade Eligibility Detection) | Fires only when the downgrade-eligible flag goes from cleared to raised -- never while the flag stays raised | Plan reference, active client count at the time, actor "Automatic", timestamp; account-level (no project) |
| A subscription cancellation is recorded | FEAT-23.SPEC-006 (Cancel Subscription) | Fires after the cancellation record commits | Plan reference, Nadia as actor, prior and new status, period end date, timestamp; account-level (no project) |
| A support session opens | FEAT-31 (Operator Support Access) | Fires the moment Dana's session begins | Freelancer account, Dana as actor, open timestamp |
| A support session closes | FEAT-31 (Operator Support Access) | Fires the moment the session ends, whether Dana closes it or it closes automatically on inactivity | Freelancer account, Dana as actor, close timestamp, closure reason (manual / inactivity) |

## Processing Logic

1. Receive the reported event from the triggering feature: `event_type`, `actor`, `occurred_at` (or capture the current time if the triggering feature does not supply one), the `affected_record` reference, and the `project` reference where the event belongs to one.
2. Classify `event_type` against the fixed vocabulary defined by FEAT-13.SPEC-004 (proposal accepted, milestone approved, milestone reopened, invoice sent, credit note issued, deliverable uploaded, deliverable removed, first client view, reminder sent, contact role changed/added/removed, refund/reversal/cancellation recorded, manual payment recorded, support session opened, support session closed, and the four FEAT-23 plan event types: plan created, plan tier/status changed, downgrade offer raised, plan cancellation recorded).
3. Check whether an entry already exists for this exact event (same `event_type`, `actor`, `affected_record`, and `occurred_at`) -- this guards against a duplicate report of the identical event arriving twice (for example, a retried request from the triggering feature after its own timeout). If an identical entry already exists, treat this as the "duplicate report" outcome and take no further action.
4. Otherwise, compose one new Activity Log Entry with `event_type`, `actor`, `occurred_at`, `affected_record`, and `project`, applying the required-field and attribution rules governed by FEAT-13.SPEC-004.
5. Attempt to write the entry durably.
6. Hold the triggering feature's own action (the acceptance, the approval, the invoice send, and so on) as not-yet-complete until this write succeeds -- the record itself is the evidence, so the triggering action and its record are treated as one unit.
7. If the write does not succeed on the first attempt, retry at platform parameter: `activity-entry-write-retry-interval` intervals until it succeeds (Edge Cases: Failure Path); there is no maximum retry count, since a permanently lost entry would break the feature's evidentiary guarantee.
8. On a successful write, make the new entry immediately visible to any viewer with access on FEAT-13.SPEC-001 (Activity Trail), and release the triggering feature's own action to report its own completion to its user.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Entry written successfully (first attempt) | The write succeeds immediately | One new Activity Log Entry created | None from this automation directly -- the triggering feature's own confirmation (e.g., "Proposal accepted") is what the user sees, released only once the write succeeds | FEAT-13.SPEC-001 |
| Entry written successfully (after retry) | The write fails one or more times, then succeeds | One new Activity Log Entry created, identical in content to the first-attempt case | The triggering feature's own action was held in its own pending/processing state during the retries; once written, the same confirmation appears, with no difference the user can perceive beyond the delay | FEAT-13.SPEC-001, the triggering feature's own confirmation spec |
| Duplicate report of the identical event | The same event (identical `event_type`, `actor`, `affected_record`, `occurred_at`) is reported more than once | No second entry is created -- the automation is idempotent | None -- the triggering feature's own retry proceeds normally against the entry already written | FEAT-13.SPEC-001 |
| Write in progress / retrying | The write has not yet succeeded | No entry exists yet | The triggering feature's own action remains in its own "processing" or "pending" state, per that feature's own spec | Triggering feature's own spec |

## Data Model

**Reads:** Activity Log Entry -- `event_type`, `actor`, `affected_record`, `occurred_at`, to check for an existing entry matching the reported event before writing (Processing Logic Step 3's duplicate-report guard). Beyond that one lookup, nothing further is read independently -- the triggering feature is the source of truth for its own event's data.
**Creates:** Activity Log Entry -- `event_type`, `actor`, `occurred_at`, `affected_record`, `project`, exactly as required by FEAT-13.SPEC-004.
**Updates:** None -- an Activity Log Entry, once created, is never updated by this or any spec (FEAT-13.SPEC-004).
**Deletes:** None -- entries are never deleted by this spec; removal exists only as described in FEAT-13.SPEC-006.

## Business Rules

- XBR-05: every record-worthy event listed in the Trigger Definition writes an append-only trail entry carrying actor and timestamp -- this automation is the sole owner of that write path across the whole product.
- FEAT-13.SPEC-004 is the single source of truth for what an entry must contain and that it can never be edited or deleted; this automation applies those rules at write time rather than re-deriving them.
- A milestone approval and a milestone reopen are always written as two distinct entries -- a reopen never overwrites, replaces, or removes the earlier approval entry (Brief, Side-Effect Inventory).
- A refund, reversal, or cancellation is written as a new entry that preserves, rather than replaces, the original disputed entry (XBR-04).
- FEAT-23 plan events (plan created, tier/status changed, downgrade offer raised, cancellation recorded) are reported by FEAT-23 after its own plan write has committed; this automation's retry-until-success guarantee applies to the trail entry only and never delays, reverses, or blocks the plan change, offer, or cancellation itself (FEAT-23.SPEC-002 step 6, FEAT-23.SPEC-004 step 12, FEAT-23.SPEC-005 step 5, FEAT-23.SPEC-006 step 6). Plan events belong to the freelancer account rather than one project, so they carry no project reference, like the support-session events.
- There is no depth limit on entries within a project's lifetime (product-features.md, Validation & Limits) -- this automation never rejects a write for volume reasons.

## Edge Cases

- **Concurrent trigger firing (two independent events happen at effectively the same moment -- e.g., an automatic reminder fires while Nadia sends a manual one, or two client contacts each record their first view of the same deliverable within moments of each other)** -- Each reported event writes its own independent entry; entries are append-only, so there is no conflict between them. Both entries appear in the trail, ordered by their own `occurred_at`.
- **Trigger fires while a previous run is in flight (the same event is reported a second time before the first write has completed -- e.g., a network-level retry from the triggering feature)** -- Step 3's duplicate check makes this idempotent: a second report of the identical event does not produce a second entry. A genuinely new, distinct event for the same affected record (for example, an approval followed moments later by a reopen) is never treated as a duplicate and always writes its own entry.
- **The entry write does not succeed on first attempt** -- Retried at platform parameter: `activity-entry-write-retry-interval` intervals until it succeeds; the triggering feature's own action is not treated as complete until its entry is durably recorded, since the record itself is the evidence (Brief, Side-Effect Inventory).
- **A reported event names an actor whose contact details have since been erased (FEAT-18 erasure request)** -- The entry retains the actor's name exactly as it stood at the time of the event, per FEAT-13.SPEC-006; this automation is never re-invoked to alter an already-written entry.
- **An event is reported against a project that has since been marked cancelled** -- The entry is written and recorded against that project exactly as any other; a cancelled project's trail remains fully readable and unaffected (XBR-25: cancellation preserves full history).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03 (Proposal Acceptance) | Triggered by (inbound) | Reports a proposal acceptance event |
| FEAT-05 (Client Portal Access & Magic-Link Login) | Triggered by (inbound) | Reports a client contact's first-view event |
| FEAT-06 (Deliverable Upload & Sharing) | Triggered by (inbound) | Reports a deliverable upload or removal event |
| FEAT-08 (Milestone Approval) | Triggered by (inbound) | Reports a milestone approval or reopen event |
| FEAT-09 (Invoice Generation & Sending) | Triggered by (inbound) | Reports an invoice-sent or credit-note-issued event |
| FEAT-10 (Invoice Payment Processing) | Triggered by (inbound) | Reports a manually recorded off-platform payment event |
| FEAT-11 (Automated Payment Reminders) | Triggered by (inbound) | Reports an automatic or manual reminder-sent event |
| FEAT-18 (Client Contact Management & Roles) | Triggered by (inbound) | Reports a contact role-change, addition, or removal event |
| FEAT-23.SPEC-002 (Free Plan Auto-Provisioning) | Triggered by (inbound) | Reports the plan-created event |
| FEAT-23.SPEC-004 (Plan State Sync) | Triggered by (inbound) | Reports every committed plan tier or status change |
| FEAT-23.SPEC-005 (Downgrade Eligibility Detection) | Triggered by (inbound) | Reports a downgrade offer raised |
| FEAT-23.SPEC-006 (Cancel Subscription) | Triggered by (inbound) | Reports a recorded subscription cancellation |
| FEAT-25 (Refund & Cancelled Project Handling) | Triggered by (inbound) | Reports a refund, reversal, or cancellation event |
| FEAT-31 (Operator Support Access) | Triggered by (inbound) | Reports a support session opened or closed event |
| FEAT-13.SPEC-001 (Activity Trail) | Affects (outbound) | Every written entry becomes immediately visible here to any viewer with access |
| FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules) | References (inbound) | Defines every entry's required content and the immutability guarantee this automation enforces at write time |
| FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules) | Affects (outbound) | The entries this automation writes are what visibility rules govern access to |

## Analytics and Success Signals

- **activity_entry_written** (event_type, actor role, project reference, attempts before success) -- supports success-metrics.md: "Dispute Resolution Confidence" (a record must exist, reliably and automatically, before Nadia can ever locate it during a dispute)
- **activity_entry_write_retried** (event_type, attempt number) -- N/A -- no success-metrics.md metric measures retry volume directly; retained because product-features.md's Signals field for this feature names `activity_entry_written` as a required signal, and this companion event lets the write-reliability behavior described in the Brief's Side-Effect Inventory ("retry until the write succeeds") be observed in event data without altering the cited metric's own definition

## Acceptance Criteria

**FEAT-13.SPEC-003-AC-01:** Given Owen accepts a proposal, when the acceptance is recorded, then this automation writes an entry with event_type "proposal accepted," Owen as actor, and the acceptance timestamp.

**FEAT-13.SPEC-003-AC-02:** Given Owen approves a milestone, when the approval is recorded, then this automation writes an entry with event_type "milestone approved," Owen as actor, and the approval timestamp.

**FEAT-13.SPEC-003-AC-03:** Given Nadia reopens a previously approved milestone, when the reopen is recorded, then this automation writes a separate entry with event_type "milestone reopened," distinct from and never overwriting the earlier approval entry.

**FEAT-13.SPEC-003-AC-04:** Given an invoice is sent, whether automatically or as an ad hoc send, when the send completes, then this automation writes an entry with event_type "invoice sent" and the send timestamp.

**FEAT-13.SPEC-003-AC-05:** Given Nadia issues a credit note against a prior invoice, when the credit note is issued, then this automation writes an entry with event_type "credit note issued," Nadia as actor, and a reference to the corrected invoice.

**FEAT-13.SPEC-003-AC-06:** Given Nadia's deliverable upload completes, when the upload finishes, then this automation writes an entry with event_type "deliverable uploaded," Nadia as actor, and the completion timestamp.

**FEAT-13.SPEC-003-AC-07:** Given Nadia withdraws a deliverable, when the removal completes, then this automation writes an entry with event_type "deliverable removed," Nadia as actor, and the removal timestamp.

**FEAT-13.SPEC-003-AC-08:** Given Priya views a deliverable for the first time, when that first view is captured, then this automation writes an entry with event_type "first client view," Priya as actor, and the first-view timestamp; a second view by Priya of the same deliverable writes no further entry.

**FEAT-13.SPEC-003-AC-09:** Given an automatic day-3 reminder is sent, when the send completes, then this automation writes an entry with event_type "reminder sent," actor "Automatic," and the send timestamp.

**FEAT-13.SPEC-003-AC-10:** Given Nadia sends a manual reminder, when the send completes, then this automation writes an entry with event_type "reminder sent," Nadia as actor, and the send timestamp.

**FEAT-13.SPEC-003-AC-11:** Given Owen invites Priya as a Reviewer contact, when the invitation is saved, then this automation writes an entry with event_type "contact role changed" (added), Owen as the acting party.

**FEAT-13.SPEC-003-AC-12:** Given Nadia records a refund on a disputed invoice, when the refund is recorded, then this automation writes a new entry with event_type "refund/reversal/cancellation recorded" that preserves the original disputed entry unchanged.

**FEAT-13.SPEC-003-AC-13:** Given Nadia records an off-platform payment, when the record is saved, then this automation writes an entry with event_type "manual payment recorded," Nadia as actor.

**FEAT-13.SPEC-003-AC-14:** Given Dana opens a support session on Nadia's account, when the session begins, then this automation writes an entry with event_type "support session opened," Dana as actor.

**FEAT-13.SPEC-003-AC-15:** Given Dana's support session ends automatically after inactivity, when the session closes, then this automation writes an entry with event_type "support session closed," Dana as actor, and closure reason "inactivity."

**FEAT-13.SPEC-003-AC-16:** Given an entry write fails on its first attempt, when the automation retries, then it retries at platform parameter: `activity-entry-write-retry-interval` intervals until the write succeeds, and the triggering feature's own action is not reported complete to its user until then.

**FEAT-13.SPEC-003-AC-17:** Given the same triggering event is reported twice due to a retried request from the triggering feature, when the second report arrives, then no second Activity Log Entry is created.

**FEAT-13.SPEC-003-AC-18:** Given two client contacts each record their first view of the same deliverable within moments of each other, when both events are reported, then two independent entries are written, each attributed to its own viewing contact.

**FEAT-13.SPEC-003-AC-19:** Given a client contact's details are later erased through FEAT-18, when Nadia views an entry that already named that contact as actor, then the entry still displays the contact's name exactly as it was at the time of the event.

**FEAT-13.SPEC-003-AC-20:** Given Nadia's Free plan record has just been created at account creation, when FEAT-23.SPEC-002 reports the plan-created event, then this automation writes an entry with event_type "plan created," actor "Automatic," tier Free, status Active, and no project reference, and a retry of that write never delays or reverses her plan record.

**FEAT-13.SPEC-003-AC-21:** Given Nadia's plan moves from Paid, Active to Free, Lapsed at period end, when FEAT-23.SPEC-004 reports the change, then this automation writes an entry with event_type "plan tier/status changed," actor "Automatic," the prior and new tier and status, and the change timestamp; a routine renewal that changes nothing writes no entry.

**FEAT-13.SPEC-003-AC-22:** Given the downgrade-eligible flag on Nadia's plan goes from cleared to raised, when FEAT-23.SPEC-005 reports the offer, then this automation writes one entry with event_type "downgrade offer raised" and actor "Automatic," and no further entry is written while the flag stays raised.

**FEAT-13.SPEC-003-AC-23:** Given Nadia cancels her Paid subscription, when FEAT-23.SPEC-006 reports the recorded cancellation, then this automation writes an entry with event_type "plan cancellation recorded," Nadia as actor, the prior and new status, and the period end date, and a retry of that write never reverses the cancellation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 18 | 18 |
| Outcome Paths | 4 | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Entry Immutability, Content & Attribution Rules

## Overview

**Name:** Entry Immutability, Content & Attribution Rules
**ID:** FEAT-13.SPEC-004
**Type:** Logic/Rule
**Purpose:** Defines the required fields every Activity Log Entry must carry, the fixed vocabulary of recordable events, and the hard prohibition against ever editing or deleting a written entry.
**Parent Feature:** FEAT-13 -- Immutable Activity & Audit Trail
**Governed Entity:** Activity Log Entry

## Scope and Non-Goals

**In Scope:**
- Per-field content and validation rules for every Activity Log Entry field
- The closed vocabulary of `event_type` values this feature ever writes
- Cross-field consistency rules (event type vs. affected record type; event type vs. project requirement)
- Authorization rules for every action the product defines on this entity -- create, edit, delete, and view
- Default and derivation rules for `occurred_at` and `actor`

**Non-Goals:**
- Deciding who may view the resulting entries -- owned by FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules); this spec's Authorization Rules table references that spec for view/read actions rather than restating them.
- Deciding when and how entries are removed from the product -- owned by FEAT-13.SPEC-006 (Retention & Account-Deletion Purge Rule); this spec only forbids in-product deletion, it does not define account-deletion behavior.
- Deciding which reported events actually trigger a write, and how retries and duplicate-report handling work -- owned by FEAT-13.SPEC-003 (Activity Entry Recording); this spec defines the rules that automation applies, not its trigger or retry mechanics.

## Governed Entity

**Entity:** Activity Log Entry
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| event_type | enum | The kind of record-worthy event this entry describes, drawn from a fixed, closed vocabulary |
| actor | text | The freelancer, a named client contact, the operator, or "Automatic" (the product itself) -- whoever or whatever caused the event |
| occurred_at | date (timestamp) | The exact date and time the event occurred |
| affected_record | text (reference) | A reference to the specific record the event concerns (e.g., a proposal, milestone, invoice, deliverable, or the freelancer's Subscription Plan for a plan event) |
| project | text (reference) | The project the event belongs to, required wherever the event genuinely has one |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-13.SPEC-003 | Activity Entry Recording | On every write, before an entry is persisted -- a malformed entry is never written, regardless of how many times the write is retried for durability |
| FEAT-13.SPEC-001 | Activity Trail | Relies on these rules to guarantee that everything it displays is complete and was never altered after writing |
| FEAT-13.SPEC-002 | Printable Record Copy | Relies on these rules to guarantee that what it renders and reproduces is unaltered |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| event_type | Required; must be one of the fixed, closed vocabulary values defined in Business Rules below | Always | On write | "Entry rejected: unrecognized event type" (an internal, engineering-facing rejection -- `event_type` is always supplied by a trusted internal caller, never typed by a product user) | Yes |
| actor | Required; must identify exactly one of: the freelancer, a named client contact, the operator, or "Automatic" | Always | On write | "Entry rejected: actor is required" | Yes |
| occurred_at | Required; a valid timestamp that is not in the future | Always | On write | "Entry rejected: invalid timestamp" | Yes |
| affected_record | Required; must reference an existing record whose entity type matches what `event_type` expects (see Cross-Field Rules) | Always | On write | "Entry rejected: affected record reference is invalid" | Yes |
| project | Required | Whenever `event_type` is anything other than "support session opened", "support session closed", or one of the four plan event types ("plan created", "plan tier/status changed", "downgrade offer raised", "plan cancellation recorded") (see Cross-Field Rules) | On write | "Entry rejected: project reference required for this event type" | Yes (for project-scoped event types); no validation applies for the two support-session event types or the four plan event types, which are account-level |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Affected-record type must match event type | event_type, affected_record | The entity type referenced by `affected_record` must match the fixed mapping for `event_type` (e.g., "milestone approved" must reference a Milestone, never an Invoice or Deliverable; the four plan event types -- "plan created", "plan tier/status changed", "downgrade offer raised", "plan cancellation recorded" -- must reference the freelancer's Subscription Plan) | "Entry rejected: affected record does not match event type" |
| Project required unless account-level | event_type, project | `project` is required for every `event_type` except "support session opened" and "support session closed," which may span the whole freelancer account before any single project is in view, and the four plan event types ("plan created", "plan tier/status changed", "downgrade offer raised", "plan cancellation recorded"), which belong to the freelancer account rather than any one project (FEAT-13.SPEC-003) | "Entry rejected: project reference required for this event type" |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create entry | None -- no role creates an entry directly | Entries are created only by the Activity Entry Recording automation (FEAT-13.SPEC-003), always on behalf of one of the eleven writer features listed in FEAT-13.SPEC-003, never at any role's direct initiative | No control anywhere in the product allows any role to directly create an entry; the action does not exist as a user-facing capability for any role. |
| Edit entry (any field) | None -- no role, ever | Never, for any entry, at any time | No edit control is ever rendered on any entry, for any role. This is the feature's defining guarantee (BRIEF.md, Constraints: "accepted proposals, approvals and sent invoices must never be silently edited afterwards"). Any attempt to modify an existing entry -- however reached -- is rejected outright. |
| Delete entry (in-product, single entry) | None -- no role, ever | Never, for any entry, at any time, in-product | No delete control is ever rendered. The only removal path for entries at all is account-deletion, governed entirely by FEAT-13.SPEC-006 -- never a per-entry action available to any role. |
| View entry (single or list) | Governed by FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules) | See that spec | See FEAT-13.SPEC-005 for the exact denied experience per role. |
| Produce a printable/shareable copy of an entry or the trail | Nadia (Freelancer) only | Always, her own projects only | Not shown to any other role -- see FEAT-13.SPEC-002's Access and Visibility table for the exact experience for Owen, Priya, and Dana. |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| occurred_at | If the triggering feature supplies its own event-moment timestamp (e.g., the exact instant an acceptance is recorded), that value is used; otherwise, the current time at write is used | On create only | No -- never editable after write, by any role, per the Authorization Rules above |
| actor | Derived entirely from the triggering feature's own reported event data -- this spec never independently determines who the actor is | On create only | No |

## Business Rules

- XBR-04 and XBR-05: immutability is this feature's defining evidentiary guarantee. Corrections to what an earlier entry describes happen only as new, separate entries elsewhere in the trail (a milestone reopen, a credit note, a refund/reversal entry) -- never as a change to an existing entry's fields.
- The closed vocabulary of `event_type` values is exactly: proposal accepted, milestone approved, milestone reopened, invoice sent, credit note issued, deliverable uploaded, deliverable removed, first client view, reminder sent, contact role changed / added / removed, refund/reversal/cancellation recorded, manual payment recorded, support session opened, support session closed, and the four FEAT-23 plan event types: plan created, plan tier/status changed, downgrade offer raised, plan cancellation recorded. No other `event_type` value may ever be written by FEAT-13.SPEC-003.
- Nothing in this feature ever creates an entry on its own initiative -- every entry exists only on behalf of one of the eleven writer features listed in FEAT-13.SPEC-003 (dependency map, Entity-Lifecycle Coverage Matrix: "created only on behalf of ten other features, never on its own initiative").
- An entry's identity for duplicate-report purposes (FEAT-13.SPEC-003) is the combination of `event_type`, `actor`, `affected_record`, and `occurred_at` together -- no subset of these fields alone identifies a unique event.
- A contact's erasure request (FEAT-18, XBR-27) never triggers a rewrite of any field on an entry that already names that contact as actor; retention of the actor's name on existing entries is governed by FEAT-13.SPEC-006, not by this spec's field rules.

## Edge Cases

- **occurred_at exactly equal to the write-time timestamp (the triggering feature supplied no explicit event-moment of its own)** -- Accepted; the write-time timestamp is used, consistent with the product's promise that entries appear "effectively as soon as the triggering event completes" (Brief, Non-Functional Notes).
- **Two entries share the same event_type, actor, and affected_record but have different occurred_at values (e.g., two separate first-views of the same deliverable by different viewing contacts recorded moments apart, or a later reopen following an earlier approval)** -- Both are valid, distinct entries; identity requires all four fields (event_type, actor, affected_record, occurred_at) to match, not any subset.
- **affected_record references a record whose own state has since changed (e.g., a withdrawn deliverable, a reopened milestone)** -- The entry's reference and content stand exactly as written; the reference may resolve to a record in a different state than it was in when the entry was written, and this is expected, not an error.
- **project is omitted for a genuinely account-level event (a support session opened or closed before Dana has looked at any specific project)** -- Valid; the "project required" cross-field rule explicitly excludes the two support-session event types, and likewise the four account-level plan event types reported by FEAT-23.
- **A reported event reaches this spec's validation with a field missing or malformed (e.g., no actor supplied)** -- Rejected at write time as a validation failure, distinct from a durability failure: FEAT-13.SPEC-003's retry behavior applies only to transient write failures, never to a validation rejection, since a validation failure means the writer feature supplied malformed data rather than encountering a transient fault.

## Acceptance Criteria

**FEAT-13.SPEC-004-AC-01:** Given a reported event with a recognized event_type, when the entry is composed, then the write proceeds.

**FEAT-13.SPEC-004-AC-02:** Given a reported event with an unrecognized event_type, when the write is attempted, then it is rejected with "Entry rejected: unrecognized event type," and no entry is created.

**FEAT-13.SPEC-004-AC-03:** Given a reported event with no actor supplied, when the write is attempted, then it is rejected with "Entry rejected: actor is required."

**FEAT-13.SPEC-004-AC-04:** Given a reported event whose occurred_at timestamp is in the future, when the write is attempted, then it is rejected with "Entry rejected: invalid timestamp."

**FEAT-13.SPEC-004-AC-05:** Given a reported "milestone approved" event whose affected_record references an Invoice instead of a Milestone, when the write is attempted, then it is rejected with "Entry rejected: affected record does not match event type."

**FEAT-13.SPEC-004-AC-06:** Given a reported "invoice sent" event with no project reference, when the write is attempted, then it is rejected with "Entry rejected: project reference required for this event type."

**FEAT-13.SPEC-004-AC-07:** Given a reported "support session opened" event with no project reference, when the write is attempted, then it succeeds -- project is not required for this event type.

**FEAT-13.SPEC-004-AC-08:** Given Nadia is viewing a written entry, when she looks for any way to edit it, then no edit control is rendered anywhere for the entry.

**FEAT-13.SPEC-004-AC-09:** Given Dana is viewing an entry during a support session, when she looks for any way to edit or delete it, then no such control is rendered -- her access is read-only, consistent with FEAT-13.SPEC-005.

**FEAT-13.SPEC-004-AC-10:** Given any written entry, when any role attempts to reach a delete action for it in-product, then no such action exists anywhere in the product; removal exists only through account deletion under FEAT-13.SPEC-006.

**FEAT-13.SPEC-004-AC-11:** Given a triggering feature reports an event without its own explicit event-moment timestamp, when the entry is composed, then occurred_at is set to the current time at write.

**FEAT-13.SPEC-004-AC-12:** Given a triggering feature reports an event with its own explicit event-moment timestamp (e.g., the exact instant Owen accepted a proposal), when the entry is composed, then occurred_at is set to that supplied timestamp rather than the write time.

**FEAT-13.SPEC-004-AC-13:** Given Priya views the same deliverable twice on different days, when each view is reported, then only the first view produces an entry with event_type "first client view" -- the second view produces no new entry of that type (per FEAT-13.SPEC-003's duplicate-report handling operating on this spec's identity rule).

**FEAT-13.SPEC-004-AC-14:** Given a milestone is approved and later reopened, when both events are reported, then two distinct entries exist -- event_type "milestone approved" and event_type "milestone reopened" -- and neither is overwritten by the other.

**FEAT-13.SPEC-004-AC-15:** Given a client contact's details are erased through FEAT-18 after they are named as actor on an existing entry, when the erasure is processed, then no field on that existing entry is rewritten.

**FEAT-13.SPEC-004-AC-16:** Given a deliverable named in an entry's affected_record is later withdrawn, when Nadia opens that entry, then its recorded content is unchanged even though the deliverable's own current state differs.

**FEAT-13.SPEC-004-AC-17:** Given a reported event is missing a required field, when FEAT-13.SPEC-003 attempts the write, then the rejection is treated as a validation failure and is not subject to the write-durability retry behavior.

**FEAT-13.SPEC-004-AC-18:** Given Nadia is the sole role permitted to produce a printable copy, when Owen looks for any equivalent control in his portal, then none is rendered.

**FEAT-13.SPEC-004-AC-19:** Given a reported "reminder sent" event with actor "Automatic," when the entry is composed, then actor is recorded exactly as "Automatic," distinct from any human actor value.

**FEAT-13.SPEC-004-AC-20:** Given a refund is recorded against a disputed invoice, when the new entry is written, then the original disputed entry's fields remain exactly as they were, and the refund appears as a separate, additional entry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Activity Trail Access & Visibility Rules

## Overview

**Name:** Activity Trail Access & Visibility Rules
**ID:** FEAT-13.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs who may see the cross-event activity trail and its entries -- Nadia Full, Dana View (session-scoped), Owen and Priya None -- versus who sees only the outcome of their own actions within their own scoped views elsewhere in the product.
**Parent Feature:** FEAT-13 -- Immutable Activity & Audit Trail
**Governed Entity:** Activity Log Entry (read/visibility dimension)

## Scope and Non-Goals

**In Scope:**
- The authoritative role-by-role visibility rule for the cross-event activity trail (FEAT-13.SPEC-001) and the printable record copy (FEAT-13.SPEC-002)
- What each role experiences when they lack access to this capability
- How Dana's session-scoped, read-only access is bounded
- The distinction between "seeing the cross-event trail" (this spec) and "seeing the outcome of one's own action within one's own scoped view" (owned by each individual feature's own screens, e.g., FEAT-03's acceptance confirmation)

**Non-Goals:**
- Field-level content and immutability rules for entries -- owned by FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules); this spec governs only who may see an entry, never what it must contain.
- Retention and account-deletion purge behavior -- owned by FEAT-13.SPEC-006 (Retention & Account-Deletion Purge Rule).
- Defining what a Primary or Reviewer contact may do on their own company's proposals, milestones, or invoices -- owned by each of those features' own access rules (FEAT-03, FEAT-07, FEAT-08, FEAT-09, FEAT-10); this spec only governs the cross-event trail, never those features' own scoped views.

## Governed Entity

**Entity:** Activity Log Entry
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| event_type | enum | The kind of record-worthy event this entry describes |
| actor | text | The freelancer, a named client contact, the operator, or "Automatic" |
| occurred_at | date (timestamp) | The exact date and time the event occurred |
| affected_record | text (reference) | A reference to the specific record the event concerns |
| project | text (reference) | The project the event belongs to, where it has one |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-13.SPEC-001 | Activity Trail | On screen entry -- determines whether the screen is reachable at all for the current role/session, and what it renders when it is |
| FEAT-13.SPEC-002 | Printable Record Copy | On screen entry -- determines whether the screen is reachable at all for the current role |
| FEAT-13.SPEC-003 | Activity Entry Recording | Relies on this spec to know that entries it writes for Dana's own support sessions will be correctly scoped to Dana's view and Nadia's view once written |

## Field Validation Rules

No field-level validation applies in this spec -- content and format rules for every field are owned entirely by FEAT-13.SPEC-004. Visibility in this spec operates at the whole-entry level: an entry a role can see is shown with every field intact; there is no partial-field redaction within a visible entry.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| event_type | No field-level visibility rule -- visibility is entry-level, per Authorization Rules below | Always | -- | -- | -- |
| actor | No field-level visibility rule -- see above | Always | -- | -- | -- |
| occurred_at | No field-level visibility rule -- see above | Always | -- | -- | -- |
| affected_record | No field-level visibility rule -- see above | Always | -- | -- | -- |
| project | No field-level visibility rule -- see above | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Whole-entry visibility, no partial redaction | event_type, actor, occurred_at, affected_record, project | If a role can view an entry at all (per Authorization Rules), every field on that entry is shown in full -- there is no rule that hides one field of a visible entry while showing the rest | N/A -- this is a display-completeness rule, not a validation failure; there is nothing for a user to correct |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the cross-event activity trail for a project (FEAT-13.SPEC-001) | Nadia (Freelancer) | Always, her own projects only | -- |
| View the cross-event activity trail for a project (FEAT-13.SPEC-001) | Dana (Support Operator) | Only inside an open support session on the freelancer's account (FEAT-31); read-only | Outside an open session, the screen is unreachable -- no navigation path, tab, or link exists, identical to the unauthenticated experience. |
| View the cross-event activity trail for a project (FEAT-13.SPEC-001) | Owen (Client Primary Contact) | Never | The capability is not shown at all -- no tab, link, or navigation path from Owen's portal view reaches this screen. He sees only the outcome of his own actions within his own scoped views (e.g., his acceptance confirmation in FEAT-03, his approval confirmation in FEAT-08, his invoice detail in FEAT-09). |
| View the cross-event activity trail for a project (FEAT-13.SPEC-001) | Priya (Client Reviewer Contact) | Never | Same as Owen -- not shown at all. She sees only the outcome of her own comment activity within her own scoped views (e.g., FEAT-07). |
| View a single entry's detail (e.g., a linked entry, or the printable copy's single-entry scope) | Nadia (Freelancer) | Always, her own projects only | -- |
| View a single entry's detail | Dana (Support Operator) | Only inside an open support session | Same as above -- unreachable outside a session. |
| View a single entry's detail | Owen, Priya | Never | Same as above -- not shown at all. |
| Produce a printable/shareable copy of the trail or a single entry (FEAT-13.SPEC-002) | Nadia (Freelancer) | Always, her own projects only | -- |
| Produce a printable/shareable copy of the trail or a single entry | Owen, Priya, Dana | Never | Not shown at all -- no control anywhere in their respective sessions reaches this capability. |

## Defaults and Derivations

N/A -- this spec defines no default or derived field values of its own. Every default and derivation for Activity Log Entry's fields is owned entirely by FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules); this spec governs visibility only.

## Business Rules

- XBR-08: role entitlements follow the Access Matrix (user-persona.md) everywhere, including this feature -- Nadia Full (read and share; entries are never editable by anyone), Owen None, Priya None, Dana View, exactly as this spec's Authorization Rules state.
- The Access Matrix's closing definition applies literally: "'None' means the capability is not shown at all" -- Owen's and Priya's denial is never a visible-but-disabled control, an error message, or an explanation screen; the capability has no footprint anywhere in their portal experience.
- XBR-29: Dana's support sessions are always logged and announced to Nadia, and always listed in her trail -- so the very session that grants Dana read-only access also becomes, once closed, an entry Nadia herself can see in the same trail Dana was viewing.
- Nadia's "Full" access is bounded to her own projects by construction -- every project in her account belongs to her, so no further ownership check beyond authentication is needed for her to see the whole trail for any of her own projects.
- ASMP-23 (client isolation): even though Owen and Priya have no access to this trail at all, the same strict per-client isolation that governs every other feature also applies here in principle -- were a future product change ever to grant any client-side trail access, it would still be scoped to that client's own company only, never another client's.

## Edge Cases

- **Owen or Priya attempts to reach the trail through a stale or guessed direct link (e.g., a URL copied from Nadia's own session)** -- Denied exactly as any other unauthorized attempt: no route exists for their role's application surface, so the request resolves the same way an invalid destination would inside their own portal -- they land on their own portal home (FEAT-05), never on trail content.
- **Dana has no open support session and attempts to navigate directly to a trail URL from an earlier session** -- Unreachable, identical to the unauthenticated experience; a closed session grants no residual access.
- **A person is a contact for several freelancers (a Primary contact for one company, a Reviewer for another) and is signed into one freelancer's portal** -- Their access to that freelancer's trail remains exactly None regardless of their role at the other freelancer, since portal scoping (FEAT-05, XBR-09) isolates each freelancer's portal session independently; there is no cross-account bleed-through to check.
- **Dana's support session is open, but the freelancer account she is viewing has multiple projects** -- Her View access extends across every project on the account she is currently sessioned into, not just one project, consistent with FEAT-31's account-level session scope; she cannot use this access to reach a different freelancer's account.
- **Nadia's account has only one project (a new freelancer with a single client)** -- Her Full access applies identically; there is no threshold of project count that changes her access level.

## Acceptance Criteria

**FEAT-13.SPEC-005-AC-01:** Given Nadia opens the activity trail for any of her own projects, when the screen loads, then she sees the full cross-event trail with every field of every entry intact.

**FEAT-13.SPEC-005-AC-02:** Given Owen is signed into his company's portal, when he looks for any way to reach the cross-event activity trail, then no tab, link, or navigation path to it exists anywhere in his portal view.

**FEAT-13.SPEC-005-AC-03:** Given Priya is signed into her company's portal, when she looks for any way to reach the cross-event activity trail, then no tab, link, or navigation path to it exists anywhere in her portal view.

**FEAT-13.SPEC-005-AC-04:** Given Owen just accepted a proposal, when he looks at his own portal, then he sees the outcome of that action within FEAT-03's own confirmation, never the cross-event trail.

**FEAT-13.SPEC-005-AC-05:** Given Dana has an open support session on Nadia's account, when she opens the trail, then she sees the full trail read-only, with every field intact and no editable or actionable controls.

**FEAT-13.SPEC-005-AC-06:** Given Dana has no open support session, when she attempts to navigate directly to a trail URL, then the screen is unreachable, identical to the unauthenticated experience.

**FEAT-13.SPEC-005-AC-07:** Given Dana's support session closes, when she attempts to continue viewing the trail, then access ends immediately.

**FEAT-13.SPEC-005-AC-08:** Given Nadia is the only role permitted to produce a printable copy, when Dana looks for an equivalent control during her session, then none is rendered.

**FEAT-13.SPEC-005-AC-09:** Given Owen or Priya attempts to reach a trail entry through a stale or guessed direct link, when the request is made, then they land on their own portal home rather than any trail content.

**FEAT-13.SPEC-005-AC-10:** Given a person is a Primary contact for one freelancer and a Reviewer contact for a different freelancer, when they are signed into the first freelancer's portal, then their access to that freelancer's trail remains None, unaffected by their role at the other freelancer.

**FEAT-13.SPEC-005-AC-11:** Given Dana's support session opens and later closes, when Nadia opens her trail afterward, then she sees both the opened and closed entries for that session, each attributed to Dana.

**FEAT-13.SPEC-005-AC-12:** Given Nadia views an entry she has access to, when she reads it, then every field (event description, actor, timestamp, affected-record reference) is shown -- no field is hidden from a role that can see the entry at all.

**FEAT-13.SPEC-005-AC-13:** Given Dana's open session spans a freelancer account with several projects, when she opens the trail for any of those projects, then her View access applies identically across all of them.

**FEAT-13.SPEC-005-AC-14:** Given Nadia's account has only a single project, when she opens the trail, then her Full access applies exactly as it would for an account with many projects.

**FEAT-13.SPEC-005-AC-15:** Given Owen attempts to reach the printable record copy screen directly, when the request is made, then the screen is unreachable to him, identical to his experience with the trail itself.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 0 (N/A -- owned by FEAT-13.SPEC-004) | 0 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



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
