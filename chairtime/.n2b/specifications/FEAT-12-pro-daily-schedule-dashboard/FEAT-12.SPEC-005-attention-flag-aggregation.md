---
document_type: spec
spec_type: automation
spec_id: FEAT-12.SPEC-005
spec_name: Attention Flag Aggregation
spec_slug: attention-flag-aggregation
parent_feature: FEAT-12
parent_feature_name: Pro Daily Schedule Dashboard
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Attention Flag Aggregation

## Overview

**Name:** Attention Flag Aggregation
**ID:** FEAT-12.SPEC-005
**Type:** Automation
**Purpose:** Gathers and de-duplicates attention-worthy signals owned by other features (calendar sync health, message delivery, refund progress, card-issuer disputes, setup-change conflicts) into a single Attention List feed, and tracks each item's resolution.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard

## Scope and Non-Goals

**In Scope:**
- Receiving attention-worthy signals from the owning features' own capabilities (FEAT-04, FEAT-08, FEAT-09, FEAT-16, and the setup features named in XBR-11)
- De-duplicating repeat signals for the same booking or cause so the Pro sees one item, not a flood
- Maintaining each item's open/resolved status as the underlying cause clears
- Feeding the resulting item list to FEAT-12.SPEC-002 (Attention List) for display

**Non-Goals:**
- Diagnosing or resolving the underlying cause (reconnecting a calendar, retrying a refund, resolving a dispute) -- each owned by its source feature (FEAT-04, FEAT-28, FEAT-16 respectively); this automation only surfaces and tracks the flag
- Displaying the aggregated items -- owned by FEAT-12.SPEC-002 (Attention List), which is this automation's sole consumer
- Aggregating waitlist demand -- excluded per this feature's own Side-Effect Inventory: waitlist demand is shown as a separate informational item directly by FEAT-12.SPEC-002, not routed through this de-duplication pipeline, since it is a standing count rather than a discrete resolvable cause
- Notifying the client about any of these attention causes -- each source feature owns its own client-facing notifications (e.g., FEAT-08 for delivery, FEAT-16 for dispute evidence requests); this automation's output is Pro-facing only

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Calendar sync health degrades | FEAT-04.SPEC-003 (Two-Way Calendar Sync -- Integration spec) | Fires when a Pro's Calendar Connection status changes to Needs Reconnection or Disconnected (XBR-13) | Pro Account reference, Calendar Connection status, time of status change |
| Message delivery gap after retry and fallback | FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | Fires when a text delivery fails, is retried once, and falls back to email, per XBR-17 -- the fallback itself is the signal, not a single first-attempt failure | Booking reference, Message type and channel, delivery_status, time of the fallback |
| Refund cannot complete immediately | FEAT-09.SPEC-005 (Cancellation & No-Show Policy Engine -- refund Integration spec) | Fires when an automatic full deposit refund (client cancellation outside the window, or any Pro cancellation) cannot complete immediately and is reported back for retry, per XBR-10 | Booking reference, Deposit Transaction reference, amount, retry status |
| Card-issuer dispute notice received | FEAT-16.SPEC-003 (Booking & Payment Activity Record -- dispute Integration spec) | Fires when a card-issuer dispute notice is received for a captured deposit, per XBR-22 | Booking reference, Deposit Transaction reference (now Disputed), dispute reference, time received |
| Setup change conflicts with an existing booking | FEAT-01, FEAT-02, FEAT-17, FEAT-18, or FEAT-27 (whichever setup feature made the change) | Fires when a change to hours, a new time block, an archived service, a Pro pause, or a subscription lapse leaves an existing confirmed booking outside the Pro's now-current setup, per XBR-11 -- the confirmed booking itself is never altered by the change | Booking reference, the setup change's kind (hours / block / archived service / pause / subscription lapse), time of the change |

## Processing Logic

1. Receive an incoming signal from one of the five trigger sources above, carrying the affected booking or account reference, the signal's cause category, and a timestamp.
2. Check whether an existing, unresolved Attention Item already exists for the same booking (or account, for a calendar-sync signal) and the same cause category.
3. If a matching unresolved item exists, update its last-seen timestamp rather than creating a duplicate -- the Pro continues to see one item for that cause.
4. If no matching unresolved item exists, create a new Attention Item recording: cause category, affected booking or account reference, first-detected time, and current status (Open).
5. For every open Attention Item, periodically re-check whether its underlying cause has cleared (calendar reconnected, delivery ultimately delivered, refund completed, dispute closed by the processor, or the conflicting booking resolved by an explicit Pro choice through FEAT-30) by consulting the same source feature's current state.
6. When a cause has cleared, mark the corresponding Attention Item Resolved and record the resolution time; a resolved item drops off the active Attention List (FEAT-12.SPEC-002) but its resolution is available in the booking's own activity history (FEAT-16), not restated here.
7. Feed the current set of Open Attention Items to FEAT-12.SPEC-002 on every request from that screen.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| New attention item created | A signal arrives with no existing unresolved item for the same booking/account and cause | New Attention Item recorded, status Open | A new card appears on the Attention List the next time the Pro opens it | FEAT-12.SPEC-002 |
| Repeat signal de-duplicated | A signal arrives matching an already-open item for the same booking/account and cause | Existing item's last-seen timestamp updated; no new item created | No change visible to the Pro -- the existing card remains as-is | FEAT-12.SPEC-002 |
| Attention item resolved | The underlying cause is found cleared on a periodic re-check | Item status -> Resolved, resolution time recorded | The card is removed from the active Attention List on the Pro's next view | FEAT-12.SPEC-002 |
| No action (cause already resolved before first check) | A signal's underlying cause clears before this automation's next re-check runs | No open item is ever shown, or an already-open item resolves on the very next check | The Pro may never see the item at all if it clears within the same check interval it was created in, or sees it briefly then see it clear | FEAT-12.SPEC-002 |
| Aggregation failure | This automation cannot process an incoming signal (e.g., an internal processing error) | The signal is not lost -- the source feature's own record (Calendar Connection status, Message delivery_status, Deposit Transaction status, dispute record, or the conflicting booking's own flag) remains the source of truth and is re-read on the next periodic re-check, so the item still surfaces once processing succeeds | No immediate feedback; the item appears on the Attention List once this automation successfully processes the underlying signal on a later pass | FEAT-12.SPEC-002 |

## Data Model

**Reads:** Calendar Connection (status), Message (delivery_status), Deposit Transaction (status, amount), Booking (reference, state), and the setup-change record from whichever of FEAT-01/FEAT-02/FEAT-17/FEAT-18/FEAT-27 raised the conflict -- read-only, per the dependency map's "Referenced Entities" list for this feature.
**Creates:** Attention Item entries (a record owned by this feature, not part of the shared Domain Entity Inventory) -- each with cause category, affected booking or account reference, first-detected time, and status.
**Updates:** Attention Item entries -- last-seen timestamp (on a de-duplicated repeat signal) and status/resolution time (on resolution).
**Deletes:** None -- resolved items are marked Resolved and removed from the active list view, not deleted; the dependency map's Booking/Deposit Transaction/Message records they reference are never deleted either.

## Business Rules

- One Attention Item per distinct (booking or account, cause category) pair while the cause remains open -- repeat signals for the same pair never create a second card (Side-Effect Inventory: "de-duplicate repeat signals for the same booking/cause").
- This automation never alters the underlying entity it reads from (Calendar Connection, Message, Deposit Transaction, Booking, or a setup record) -- it only observes and reflects their state; every write to those entities is owned by their respective feature.
- A setup-change conflict (XBR-11) never causes this automation, or any feature, to silently cancel the affected booking -- the booking is only ever changed by an explicit Pro choice through FEAT-30, and until that choice is made the Attention Item stays open.
- Resolution is derived, not asserted by the Pro directly on this screen -- an item resolves because its source feature's state changed (e.g., the Pro reconnected the calendar through FEAT-04), not because the Pro dismissed the card here.
- This automation's output is Pro-only; it never surfaces to the Client or to any external party.

## Edge Cases

- **Two different causes arrive for the same booking at once (e.g., a message delivery gap and a dispute notice)** -- Two separate Attention Items are created, one per cause category, since de-duplication is scoped to (booking/account, cause) pairs, not to the booking alone.
- **The same cause fires again for the same booking after its prior item was already resolved** -- A new Attention Item is created (not treated as a duplicate of the resolved one), since de-duplication only suppresses repeats of a currently-open item.
- **Concurrent trigger firing (a calendar-sync degradation and a message-delivery fallback signal for the same account arrive at effectively the same time)** -- Each signal is processed independently against its own cause category; both can result in new Attention Items in the same pass with no interference between them, since the de-duplication check is scoped per (booking/account, cause) pair.
- **Trigger fires while a previous aggregation pass for the same (booking/account, cause) pair is still in flight** -- The second signal's de-duplication check waits for the first to finish creating or updating its item, then finds that item and updates its last-seen timestamp rather than racing to create a second item for the same pair.
- **A setup-change conflict clears because the Pro's explicit choice (via FEAT-30) resolves the conflicting booking, but this automation's periodic re-check has not yet run** -- The Attention Item remains visible until the next re-check confirms resolution; it is never resolved purely by the passage of time without the underlying state actually changing.
- **The underlying source feature itself is degraded (e.g., FEAT-04's own sync is down) when this automation tries to re-check for resolution** -- The Attention Item stays Open (the safer default) until a successful re-check confirms the cause has actually cleared; it is never auto-resolved on an inconclusive check.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-003 (Two-Way Calendar Sync) | Triggered by (inbound) | Calendar sync health degradation feeds a new or repeat attention signal |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | Triggered by (inbound) | A delivery gap after retry and fallback feeds an attention signal |
| FEAT-09.SPEC-005 (Cancellation & No-Show Policy Engine) | Triggered by (inbound) | A refund that cannot complete immediately feeds an attention signal |
| FEAT-16.SPEC-003 (Booking & Payment Activity Record) | Triggered by (inbound) | A card-issuer dispute notice feeds an attention signal |
| FEAT-01, FEAT-02, FEAT-17, FEAT-18, FEAT-27 (setup features) | Triggered by (inbound) | A setup change that conflicts with an existing booking feeds an attention signal (XBR-11) |
| FEAT-12.SPEC-002 (Attention List) | Affects (outbound) | Supplies the current set of Open Attention Items for display |
| FEAT-30 (Pro Booking Management) | References (outbound) | The explicit Pro choice that ultimately resolves a setup-change conflict is made there, not on this feature |

## Analytics and Success Signals

- **attention_item_created** (cause_category) -- N/A -- reason: item creation is a Pro-facing signal reflected as the `attention_item_resolved` event's counterpart, but success-metrics.md defines no metric tracking how often attention items arise (only how the Pro's glance experience and resolution speed perform); tracked here for completeness so the create/resolve pair is not silently one-sided.
- **attention_item_resolved** (cause_category, time_to_resolution) -- supports success-metrics.md: "Daily Dashboard Glance Speed" (a resolved item reflects that the Pro's glance-and-act workflow surfaced and cleared something needing attention, which is part of what makes the daily glance trustworthy and complete)

## Acceptance Criteria

**FEAT-12.SPEC-005-AC-01:** Given Talia's Calendar Connection status changes to Needs Reconnection, when this automation processes the signal, then a new Attention Item is created for her account with cause "Reconnect calendar."

**FEAT-12.SPEC-005-AC-02:** Given a message to a client for a booking fails, is retried, and falls back to email (XBR-17), when this automation processes the fallback signal, then a new Attention Item is created for that booking with cause "Message delivery gap."

**FEAT-12.SPEC-005-AC-03:** Given an automatic refund for a booking cannot complete immediately, when this automation processes the signal, then a new Attention Item is created for that booking with cause "Refund in progress."

**FEAT-12.SPEC-005-AC-04:** Given a card-issuer dispute notice is received for a booking's deposit, when this automation processes the signal, then a new Attention Item is created for that booking with cause "Card-issuer dispute."

**FEAT-12.SPEC-005-AC-05:** Given Talia changes her working hours in a way that leaves an existing confirmed booking outside her new hours, when this automation processes the resulting conflict signal, then a new Attention Item is created for that booking with cause "Booking outside changed hours," and the booking itself is left unchanged.

**FEAT-12.SPEC-005-AC-06:** Given an Attention Item is already open for a booking's message-delivery gap, when a second delivery-gap signal arrives for the same booking, then no second item is created -- the existing item's last-seen time is updated instead.

**FEAT-12.SPEC-005-AC-07:** Given Talia reconnects her calendar through FEAT-04, when this automation's next periodic re-check runs, then the "Reconnect calendar" Attention Item is marked Resolved and no longer appears on FEAT-12.SPEC-002.

**FEAT-12.SPEC-005-AC-08:** Given a refund-in-progress Attention Item exists and the refund later completes successfully, when the next re-check runs, then that item is marked Resolved.

**FEAT-12.SPEC-005-AC-09:** Given two different causes (a dispute and a delivery gap) arise for the same booking at effectively the same time, when this automation processes both signals, then two separate Attention Items are created, one per cause.

**FEAT-12.SPEC-005-AC-10:** Given a setup-change conflict's underlying cause has not actually cleared, when a periodic re-check runs while the source feature is temporarily degraded and cannot confirm status, then the Attention Item remains Open rather than being resolved on an inconclusive check.

**FEAT-12.SPEC-005-AC-11:** Given this automation fails to process an incoming dispute signal due to an internal error, when the underlying Deposit Transaction remains Disputed, then the Attention Item is still created once a later processing pass succeeds -- the signal is not permanently lost.

**FEAT-12.SPEC-005-AC-12:** Given a previously resolved "Message delivery gap" Attention Item exists for a booking and a new delivery gap occurs for that same booking later, when this automation processes the new signal, then a new Attention Item is created rather than being treated as a duplicate of the resolved one.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 5 | 5 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
