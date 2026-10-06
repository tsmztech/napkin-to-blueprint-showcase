---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-13.SPEC-004
spec_name: Entry Immutability, Content & Attribution Rules
spec_slug: entry-immutability-content-attribution-rules
parent_feature: FEAT-13
parent_feature_name: Immutable Activity & Audit Trail
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 12
acceptance_criteria_count: 20
---

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
