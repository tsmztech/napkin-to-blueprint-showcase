---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-16.SPEC-005
spec_name: Activity Record Immutability & Visibility Rules
spec_slug: activity-record-immutability-visibility-rules
parent_feature: FEAT-16
parent_feature_name: Booking & Payment Activity Record
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 25
acceptance_criteria_count: 18
---

# Logic/Rule Spec: Activity Record Immutability & Visibility Rules

## Overview

**Name:** Activity Record Immutability & Visibility Rules
**ID:** FEAT-16.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs append-only enforcement (no role, including the Pro, may edit an entry), View-only access for the Pro and Support, retention tied to the Booking's life, de-identification on client deletion, and exclusion of the Pro's private client notes from the Support view.
**Parent Feature:** FEAT-16 -- Booking & Payment Activity Record
**Governed Entity:** Activity Event

## Scope and Non-Goals

**In Scope:**
- Field-level rules for the Activity Event entity (what may be set, when, and by what)
- Authorization rules for every action the product defines on Activity Event, across every role in the Access Matrix
- Retention and de-identification rules tied to the Booking's life and client deletion
- The Support-view exclusion of the Pro's private client notes

**Non-Goals:**
- Deciding what event_type values exist and what triggers each one -- owned by FEAT-16.SPEC-002 (Activity Event Recording), which enumerates every trigger and its resulting event_type; this spec governs the entity's rules once an entry is written, not the catalog of writers
- Rendering the timeline -- owned by FEAT-16.SPEC-001 (Booking Activity Timeline), which references this spec's Access rules for its own Access and Visibility table rather than restating them
- The dispute-specific overlay rule on Deposit Transaction (a different entity) -- owned by FEAT-16.SPEC-003 (Card-Issuer Dispute Integration); this spec governs Activity Event only, per its Governed Entity
- Client deletion's own execution mechanics (removing contact details and notes from the Client record itself) -- owned by FEAT-13.SPEC-004 (Client Deletion Execution); this spec governs only the resulting Activity Event de-identification, not the Client-record deletion process itself

## Governed Entity

**Entity:** Activity Event
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| event_type | enum | The kind of qualifying action this entry records (e.g., created, policy_acknowledged, deposit_succeeded, message_sent, cancelled, no_show_marked, disputed) |
| time | date (with time) | When the recorded action occurred |
| actor | enum | Who or what caused the event: Client, Pro, the product automatically, or a support view (written via FEAT-19.SPEC-002 through FEAT-16.SPEC-002) |
| details | text (structured) | The event's specifics -- e.g., policy version and wording shown, message sent and channel, deposit outcome and amount; may include a Pro's private client note reference before de-identification, per the entity's own Data Sensitivity line |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-16.SPEC-001 | Booking Activity Timeline | On screen render: Access and Visibility (View for the Pro and Support, private-notes exclusion for Support), no edit/delete control ever rendered |
| FEAT-16.SPEC-002 | Activity Event Recording | On every write: append-only enforcement (no update path beyond the client-deletion de-identification conversion), retention (never hard-deleted while the Booking exists) |
| FEAT-16.SPEC-003 | Card-Issuer Dispute Integration | On its own write (the dispute event): the same append-only enforcement applies, since it writes through FEAT-16.SPEC-002's mechanism |
| FEAT-16.SPEC-004 | Dispute Summary Download | On assembly: private-notes exclusion (the downloadable summary excludes private notes exactly as the Support view does) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| event_type | Must be one of the fixed set of qualifying event types defined by FEAT-16.SPEC-002's Trigger Definition | Always | On write | N/A -- this is a system-internal write, not a user-entered field; there is no user-facing error message because no role ever enters an event_type directly | Yes (a write with an unrecognized event_type does not occur -- no enforcing spec offers a path to attempt one) |
| time | Required, set automatically to the moment of the qualifying action; never entered or edited by any role | Always | On write | N/A -- not user-entered | Yes |
| actor | Required, derived automatically from the triggering spec (Client, Pro, the product automatically, or a support view) | Always | On write | N/A -- not user-entered | Yes |
| details | No validation beyond data type once assembled by the triggering spec; content requirements (what must be included per event_type) are FEAT-16.SPEC-002's concern, not this spec's | Always | -- | -- | -- |

No field on this entity is ever entered directly by a person through a form -- every field is system-derived at the moment of a qualifying action (per FEAT-16.SPEC-002's Processing Logic), so no field-level rule above carries a user-facing error message; each row exists to confirm the field was considered, not accidentally skipped.

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| actor consistency with event_type | actor, event_type | The actor recorded must be consistent with who or what genuinely caused the event (e.g., "no_show_marked" always has actor Pro; "message_sent" always has actor "the product automatically"; a client-initiated cancellation has actor Client) -- this consistency is guaranteed structurally by FEAT-16.SPEC-002's Processing Logic, which derives actor from the specific trigger, never independently | N/A -- this is a structural guarantee of the recording automation, not a user-facing validation that can fail |
| details content matches event_type | details, event_type | The details field's structure follows from event_type (e.g., a "policy_acknowledged" entry's details always includes a policy version and wording; a "message_sent" entry's details always includes a channel and delivery_status) | N/A -- structural guarantee of FEAT-16.SPEC-002, not a user-facing validation |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create (write) an entry | The product automatically (via FEAT-16.SPEC-002 and FEAT-16.SPEC-003) | Only in response to a qualifying trigger defined in FEAT-16.SPEC-002's Trigger Definition | N/A -- no role ever attempts to create an entry directly; there is no create control anywhere in the product for any role |
| View an entry (single or as part of a timeline) | The Pro | Only entries belonging to her own bookings (or her own Pro Account for account-level entries such as payout status) | -- |
| View an entry (single or as part of a timeline) | Platform Operator (Support) | Only while actively viewing the one Pro account under a help request (per XBR-24), and never the Pro's private client notes within any entry's details | Any entry's private-note content is simply absent from Support's rendering; no "hidden" placeholder appears -- the rest of that entry's details render normally |
| View an entry | The Client | Never | The Client is never shown this internal timeline directly (FEAT-16.SPEC-001's Access and Visibility); she sees only her own booking's outcomes through her own booking view, not this operational record |
| Edit an entry | Any role, including the Pro | Never -- no exception exists | No edit control is ever rendered for any role, on any screen, for any entry; the entry is permanent from the moment it is written |
| Delete (hard) an entry | Any role, including the Pro | Never -- no exception exists | No delete control is ever rendered for any role; hard deletion never occurs while the associated Booking exists, and never occurs at all even after client deletion (only de-identification occurs) |
| Convert an entry to de-identified form | The product automatically (via FEAT-16.SPEC-002) | Only when a client deletion is processed for the client whose bookings the entries belong to (per XBR-19) | N/A -- this is not a role-initiated action; it is the one automatic, system-only exception to full immutability, and it strips contact/note content only, never financial or timeline facts |
| Download a plain-language summary of an entry set | The Pro | Only for a booking whose Deposit Transaction currently carries the Disputed overlay (FEAT-16.SPEC-004) | The download action is not shown for a non-disputed booking; a stale request that reaches the automation anyway is refused with "This booking is no longer flagged as disputed. A summary is only available for disputed bookings." |
| Download a plain-language summary of an entry set | Platform Operator (Support) | Never (SC-05: Support has View-only access with no action control) | The download action is never rendered for Support |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| time | Set to the exact moment the qualifying action occurs, as reported by the triggering spec | On create only | No |
| actor | Derived from which triggering spec fired and its own context (e.g., FEAT-10.SPEC-004 always yields actor Client; FEAT-30.SPEC-007 always yields actor Pro; FEAT-08's message specs always yield actor "the product automatically") | On create only | No |
| event_type | Derived from the specific trigger row in FEAT-16.SPEC-002's Trigger Definition that fired | On create only | No |
| details | Assembled from the triggering spec's own available data at the moment of the trigger | On create only | No |

## Business Rules

- XBR-21: every booking, payment, messaging, and support-view event is written to an append-only, immutable activity record that no role can edit.
- XBR-19: on client deletion, this entity's entries are converted to de-identified form (contact and note content stripped) rather than deleted; financial and timeline facts are retained per SC-22.
- XBR-24: Support's access to this entity is read-only, used only after a Pro's help request, and never includes the Pro's private client notes.
- Retention: entries are retained for as long as the associated Booking exists (feature-overview.md's Validation & Limits); after a client deletion, the de-identified entries persist indefinitely per SC-22, since financial and timeline facts required for dispute/audit purposes are never purged outright.
- The one and only automatic write-after-creation this entity ever undergoes is the de-identification conversion on client deletion (per XBR-19); this is not treated as a violation of immutability, since it strips only contact/note content and never alters an event's recorded facts (event_type, time, actor category, financial amounts, or outcomes).
- An Activity Event is never read or opened individually -- it exists only as part of a booking's ordered timeline (per the Entity-Lifecycle Coverage Matrix's Read (single): N/A).

## Edge Cases

- **A client deletion is requested while a new Activity Event for that client's booking is being written at the same moment** -- The de-identification conversion applies to entries that exist at the moment the deletion completes; any entry written after that moment for the now-deleted client carries no contact or note content from the outset, consistent with the already-converted entries (see FEAT-16.SPEC-002's own concurrency handling).
- **Support's help-request session ends while they are mid-view of a timeline** -- Access is revoked immediately; any further attempt to view the timeline requires a fresh help-request-gated session, per XBR-24's "used only after a Pro's help request" condition.
- **A Pro attempts to edit an entry through any indirect path (e.g., editing the underlying Booking or Message record after the fact)** -- Editing the underlying Booking, Message, or Deposit Transaction record (where those edits are themselves permitted by their own owning specs) never rewrites an already-written Activity Event's recorded details; the entry remains a fixed snapshot of what was true at the moment it was written, even if the source record later changes through its own governing rules.
- **The Booking a set of entries belongs to reaches its retention end (the account itself is closed, per XBR-20)** -- Entries are retained through the account's 30-day cooling-off period; only after the account closure's own data-deletion step (owned by FEAT-29) do the entries become subject to that closure's own de-identified-financial-record retention, following the same de-identification principle as an individual client deletion rather than a hard delete.
- **A de-identified entry's remaining financial/timeline facts are later needed for a dispute** -- The retained facts (event_type, time, amounts, outcomes) remain fully usable as dispute evidence even without the stripped contact/note content, since a dispute concerns the transaction facts, not the client's contact details.

## Acceptance Criteria

**FEAT-16.SPEC-005-AC-01:** Given a qualifying action occurs (e.g., Talia marks a no-show), when FEAT-16.SPEC-002 writes the resulting entry, then its event_type, time, actor, and details are all set automatically with no user-entered value anywhere in the write.

**FEAT-16.SPEC-005-AC-02:** Given Talia views her own booking's timeline, when she looks for an edit or delete control on any entry, then none exists anywhere on the screen or in any connected spec.

**FEAT-16.SPEC-005-AC-03:** Given Talia attempts to alter an Activity Event through any indirect path, when she edits the underlying Booking or Message record instead, then the previously written Activity Event's own recorded details remain unchanged.

**FEAT-16.SPEC-005-AC-04:** Given Talia views her own booking's timeline, when the screen renders, then she sees every entry belonging to that booking, including her own private client notes where relevant to an entry's details.

**FEAT-16.SPEC-005-AC-05:** Given Support opens a Pro's booking timeline after a help request, when an entry's details would normally include the Pro's private client note, then that note content is absent from Support's rendering while the rest of the entry's details render normally.

**FEAT-16.SPEC-005-AC-06:** Given Riley (the Client) attempts to view this internal timeline, when the attempt is made, then she is never shown any part of it -- she sees only her own booking's outcomes through her own booking view.

**FEAT-16.SPEC-005-AC-07:** Given a client deletion is processed for one of Talia's clients, when FEAT-16.SPEC-002 executes the conversion, then every existing Activity Event for that client's bookings has its contact and note content stripped while event_type, time, financial amounts, and outcomes remain unchanged.

**FEAT-16.SPEC-005-AC-08:** Given a de-identified Activity Event, when Talia or Support later views it, then it displays its retained financial and timeline facts with no contact or note content and no "[hidden]" placeholder in their place.

**FEAT-16.SPEC-005-AC-09:** Given a booking's Activity Events, when the associated Booking still exists, then the entries are retained indefinitely with no expiry or automatic purge.

**FEAT-16.SPEC-005-AC-10:** Given Talia's account is closed and its 30-day cooling-off period elapses, when the account closure's data-deletion step runs, then the account's Activity Events are de-identified following the same principle as an individual client deletion, never hard-deleted outright.

**FEAT-16.SPEC-005-AC-11:** Given Talia is viewing a disputed booking's timeline, when she taps the download action, then FEAT-16.SPEC-004's own authorization check (Disputed overlay present) determines whether the summary is produced.

**FEAT-16.SPEC-005-AC-12:** Given Support is viewing a Pro's timeline, when Support looks for a download-summary action, then none is rendered, per the Authorization Rules row denying that action to Support entirely.

**FEAT-16.SPEC-005-AC-13:** Given a new Activity Event write is attempted for a trigger not listed in FEAT-16.SPEC-002's Trigger Definition, when the write path is examined, then no such path exists anywhere in the product -- the only writers are the enumerated triggers and the client-deletion conversion.

**FEAT-16.SPEC-005-AC-14:** Given Support's help-request-gated session ends, when Support attempts to continue viewing a timeline, then access is denied and a fresh help-request-gated session is required.

**FEAT-16.SPEC-005-AC-15:** Given an Activity Event's actor is derived from its triggering spec, when a client-initiated cancellation writes its entry, then the actor field reads Client, never Pro or "the product automatically."

**FEAT-16.SPEC-005-AC-16:** Given a message-delivery automation writes its entry, when the entry is created, then its actor field reads "the product automatically," consistent with every automated message trigger.

**FEAT-16.SPEC-005-AC-17:** Given a client is deleted and later re-books with the same Pro, when the new booking begins accumulating Activity Events, then those new entries start a fresh record entirely separate from the earlier, now de-identified entries -- the de-identified history is never merged into or resurrected for the new record.

**FEAT-16.SPEC-005-AC-18:** Given the entity's fields are examined for coverage, when each of event_type, time, actor, and details is checked against the Field Validation Rules table, then every field has an explicit rule or an explicit "not user-entered" rationale -- none is left unaddressed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 5 | 5 |
