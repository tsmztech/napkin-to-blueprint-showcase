---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-04.SPEC-008
spec_name: Calendar Connection Rules
spec_slug: calendar-connection-rules
parent_feature: FEAT-04
parent_feature_name: Two-Way Calendar Sync
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
rule_count: 13
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Calendar Connection Rules

## Overview

**Name:** Calendar Connection Rules
**ID:** FEAT-04.SPEC-008
**Type:** Logic/Rule
**Purpose:** Enforces the one-connection-per-kind limit and the contention/precedence rules governing connect, disconnect, and in-flight sync for the Calendar Connection entity.
**Parent Feature:** FEAT-04 -- Two-Way Calendar Sync
**Governed Entity:** Calendar Connection

## Scope and Non-Goals

**In Scope:**
- Field-level rules for every Calendar Connection field
- The one-connection-per-kind limit
- Authorization rules for every action on Calendar Connection, per role
- Contention precedence: disconnect wins over in-flight sync; health-status updates are last-write-wins; reconciliation on reconnect
- Default values on connection creation

**Non-Goals:**
- Performing the account-linking handshake or the busy-time pull -- owned by FEAT-04.SPEC-003 (Calendar Provider Sync); this spec governs the record's field rules and authorization, not the external mechanics
- Deciding health-status transitions themselves (when a connection becomes Needs Reconnection or reconciles) -- owned by FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation); this spec's last-write-wins rule governs only which concurrent update prevails, not the transition logic itself
- Governing Booking fields or Booking's own contention rules -- Booking is a separate entity with its own dependency-map contention notes; this spec only reads Booking as context for FEAT-04.SPEC-005, it does not govern it
- Retention or purge policy for a disconnected connection -- excluded per the feature-overview.md's Non-Goals: disconnect is a hard delete with no restore path by deliberate product decision, so no retention rule exists to enforce here

## Governed Entity

**Entity:** Calendar Connection
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| calendar_kind | enum (Google, Apple) | Which supported personal-calendar kind this connection is for; at most one connection per kind per Pro Account |
| status | enum (Connected, Syncing, Needs Reconnection, Disconnected) | The connection's current health state |
| last_successful_sync | date/time | Time of the last good sync in each direction (busy-time pull or booking write) |
| busy_periods | derived | The minimum busy/free data needed to block slots, never full event details |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-04.SPEC-001 | Calendar Connection Setup | On kind selection (one-per-kind limit disables an already-connected kind); authorization on screen entry |
| FEAT-04.SPEC-002 | Calendar Connection Status & Management | On Disconnect action (disconnect-wins contention rule); authorization on screen entry and per-action |
| FEAT-04.SPEC-003 | Calendar Provider Sync | On handshake completion (creates the record with its default values) |
| FEAT-04.SPEC-004 | Busy-Time Availability Feed | On every recompute (reads status and busy_periods per the field rules) |
| FEAT-04.SPEC-005 | Booking-to-Calendar Sync | On in-flight write/move/remove (disconnect-wins contention rule) |
| FEAT-04.SPEC-006 | Sync Health Monitor & Reconciliation | On every status transition (last-write-wins rule for concurrent health signals; reconciliation-on-reconnect rule) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| calendar_kind | Must be one of the two supported kinds (Google, Apple); required and immutable once set | Always | On connection creation | "Choose Google or Apple to connect a calendar." | Yes |
| calendar_kind | At most one connection of this kind may exist per Pro Account at a time | Always -- see Cross-Field Rules for the enforcement detail | On connection creation attempt | "You already have a {kind} calendar connected. Disconnect it first, or manage it below." | Yes |
| status | Must be one of Connected, Syncing, Needs Reconnection, Disconnected; only the system (via FEAT-04.SPEC-003/FEAT-04.SPEC-006) sets this field -- no direct Pro input | Always | On every transition | No validation-error message applies -- this field is never directly edited by the Pro, so no invalid-input path exists | No |
| last_successful_sync | No validation beyond data type -- a system-set timestamp, never Pro-entered | Always | -- | -- | No |
| busy_periods | No validation beyond data type -- a system-derived value from the calendar-sync capability, never Pro-entered | Always | -- | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| One connection per kind | calendar_kind (across all of a Pro Account's Calendar Connection records) | A new connection's calendar_kind must not match the calendar_kind of any existing, non-deleted connection for the same Pro Account | "You already have a {kind} calendar connected. Disconnect it first, or manage it below." |
| Status consistency with last_successful_sync | status, last_successful_sync | A connection cannot show status Connected with no last_successful_sync ever recorded -- a first-time connection stays in Syncing until its first successful pull sets last_successful_sync, only then advancing to Connected | N/A -- this is an internal system-state rule with no Pro-facing validation message; it governs FEAT-04.SPEC-003/FEAT-04.SPEC-006's own transition logic |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create connection (connect a calendar) | The Pro (Talia) | Always, subject to the one-per-kind limit above | Not shown to any other role -- the Connect action does not appear for the Client or Support |
| View connection (status, calendar_kind, last_successful_sync) | The Pro (Talia) | Always, own connections only | -- |
| View connection (status, calendar_kind, last_successful_sync) | Platform Operator (Support) | Always, view-only, for the Pro account whose help request is open | -- |
| View connection (status, calendar_kind, last_successful_sync) | The Client (Riley) | Never | No view of any kind is exposed to the Client; the Access Matrix records "None" for this capability group |
| View busy_periods (raw busy/free data) | The Pro (Talia) | Never -- even the Pro sees only status, never the raw busy/free data itself, consistent with product-features.md's Data Notes ("Displayed: connection status to the Pro") | Not shown as a distinct data element anywhere in the product; its only visible effect is which slots the availability engine offers |
| View busy_periods (raw busy/free data) | Platform Operator (Support) | Never | Not shown to Support; Support's View access is limited to connection health, never calendar content |
| Reconnect | The Pro (Talia) | Only when status is Needs Reconnection | Reconnect control is hidden on a card whose status is Connected or Syncing (nothing to reconnect) |
| Reconnect | Platform Operator (Support) | Never | Reconnect control is not shown to Support at all -- Support's access is read-only |
| Disconnect | The Pro (Talia) | Always, on any owned connection, in any status, including while a sync is in flight (disconnect wins per the Contention rule below) | Not applicable -- the Pro is never denied this action on their own connection |
| Disconnect | Platform Operator (Support) | Never | Disconnect control is not shown to Support at all -- Support's access is read-only |
| Disconnect | The Client (Riley) | Never | No Calendar Connection surface of any kind is exposed to the Client |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status | Set to Syncing | On connection creation (handshake authorized) | No -- system-derived; advances to Connected automatically once the first busy-time pull succeeds |
| last_successful_sync | Unset (no value) | On connection creation | No -- system-set on first successful sync in either direction |
| busy_periods | Unset (no value) | On connection creation | No -- system-set on first successful busy-time pull |

## Business Rules

- One-per-kind limit: a Pro Account holds at most two Calendar Connection records, one Google and one Apple, per product-features.md's Validation & Limits and the dependency map's Non-Functional Notes ("Each Pro holds at most two Calendar Connection records").
- Disconnect wins over in-flight sync (dependency map Contention note): a Pro's disconnect action always takes precedence over any sync operation (busy-time pull, booking write/move/remove, or reconciliation) in progress for that connection at the moment of disconnect; the in-flight operation is discarded, not completed and then reverted.
- Health-status updates are last-write-wins (dependency map Contention note): when two health signals (for example, a proactive revocation report and a scheduled check) could set status at effectively the same time, the most recently processed signal's value is the one that persists -- there is no merge between conflicting status values.
- Bookings already written to the personal calendar are reconciled on reconnect (dependency map Contention note): FEAT-04.SPEC-006 owns performing this reconciliation; this spec establishes it as the binding precedence rule that reconciliation must run before status returns to Connected.
- Disconnect is a hard delete with no restore/undo path: reconnecting later always creates a fresh Calendar Connection record rather than reviving the deleted one, per the dependency map's Delete/Archive lifecycle notes.
- Disconnecting never cascades to existing Chairtime bookings: Booking's own lifecycle is independent of Calendar Connection's, per the dependency map (Booking entity notes, SC-22).

## Edge Cases

- **Pro attempts to connect a second Google calendar while one is already connected** -- Rejected at the one-per-kind cross-field rule; the Google option is disabled on FEAT-04.SPEC-001 before the Pro can even attempt it, and a direct attempt (e.g., a stale screen state) is refused with "You already have a Google calendar connected. Disconnect it first, or manage it below."
- **Pro disconnects a connection at the exact moment its status would otherwise transition to Needs Reconnection** -- Disconnect wins: the connection record is removed before the status transition can apply, since disconnect is a synchronous, immediate action while a health-status transition is a background process racing against it.
- **Two health signals for the same connection arrive within the same processing window** -- Last-write-wins resolves to whichever signal's write completes last; both signals in this feature's design always agree on the outcome (both indicate an invalid permission), so the practical result is deterministic even though the rule is last-write-wins rather than a merge.
- **Reconnection completes but reconciliation has not yet finished when the Pro views the status screen** -- Status shows Syncing, not Connected, until reconciliation completes -- the Pro is never shown a falsely "settled" status while drift correction is still in progress.
- **Pro reconnects a kind whose previous connection was disconnected minutes earlier** -- A new Calendar Connection record is created; nothing about the deleted record (its prior last_successful_sync or busy_periods) carries over, consistent with the no-restore, fresh-record rule.
- **Support attempts to view busy_periods directly (e.g., via a request outside the normal status screen)** -- Denied unconditionally; busy_periods is never exposed to Support under any authorization path, per the Never row above.
- **Both connections need reconnecting and the Pro disconnects one while reconnecting the other** -- The two connections are governed entirely independently; disconnecting one has no bearing on the other's reconnection in progress.

## Acceptance Criteria

**FEAT-04.SPEC-008-AC-01:** Given Talia has a Google connection already connected, when she attempts to connect a second Google calendar, then the attempt is refused with "You already have a Google calendar connected. Disconnect it first, or manage it below."

**FEAT-04.SPEC-008-AC-02:** Given Talia has no calendar connected, when she connects an Apple calendar, then the record is created with status Syncing and no last_successful_sync value yet.

**FEAT-04.SPEC-008-AC-03:** Given Talia's new connection completes its first successful busy-time pull, when the pull confirms, then status advances from Syncing to Connected automatically, with no Pro action required.

**FEAT-04.SPEC-008-AC-04:** Given Talia disconnects a connection while a booking write is in flight for it, when the disconnect completes, then the in-flight write is discarded and the connection record is removed immediately.

**FEAT-04.SPEC-008-AC-05:** Given two health signals for the same connection are processed close together, when both complete, then the connection settles on the most recently processed signal's status value, with no merged or ambiguous intermediate state.

**FEAT-04.SPEC-008-AC-06:** Given Talia reconnects a lapsed connection, when reconciliation has not yet finished, then the connection shows Syncing, not Connected, until reconciliation completes.

**FEAT-04.SPEC-008-AC-07:** Given Talia views her own connections, when she looks at any connection card, then she sees status, calendar_kind, and last_successful_sync, but never the raw busy_periods data itself.

**FEAT-04.SPEC-008-AC-08:** Given Platform Operator Support opens a Pro's account during a help request, when they view Calendar Connection, then they see status and last_successful_sync only, with no Reconnect or Disconnect action available.

**FEAT-04.SPEC-008-AC-09:** Given a connection is in Connected status, when Talia looks for a Reconnect action on its card, then none is shown, since Reconnect is only available when status is Needs Reconnection.

**FEAT-04.SPEC-008-AC-10:** Given Talia's connection is in any status including Needs Reconnection or Syncing, when she taps Disconnect, then the action proceeds and the connection is removed, since Disconnect is always available to the Pro regardless of status.

**FEAT-04.SPEC-008-AC-11:** Given a Client (Riley) has no product surface for Calendar Connection, when any attempt is made to reach this data through the Client's own access, then no such path exists at all -- the Access Matrix records "None" for this capability group for the Client.

**FEAT-04.SPEC-008-AC-12:** Given Talia disconnects a Google connection, when she reconnects Google minutes later, then a new Calendar Connection record is created with fresh default values, and nothing from the deleted record's prior state carries over.

**FEAT-04.SPEC-008-AC-13:** Given Talia disconnects a calendar with existing confirmed bookings, when the disconnect completes, then those bookings and their history are entirely unaffected, per the no-cascade rule.

**FEAT-04.SPEC-008-AC-14:** Given both of Talia's connections need reconnecting, when she disconnects one while reconnecting the other, then the two proceed entirely independently with no interference.

**FEAT-04.SPEC-008-AC-15:** Given Talia has both a Google and an Apple connection already connected, when she looks for any further "connect a calendar" option, then none is offered, since both supported kinds are already at their one-per-kind limit.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 11 | 11 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |
