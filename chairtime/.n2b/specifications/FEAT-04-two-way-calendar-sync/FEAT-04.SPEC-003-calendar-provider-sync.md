---
document_type: spec
spec_type: integration
spec_id: FEAT-04.SPEC-003
spec_name: Calendar Provider Sync
spec_slug: calendar-provider-sync
parent_feature: FEAT-04
parent_feature_name: Two-Way Calendar Sync
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 15
---

# Integration Spec: Calendar Provider Sync

## Overview

**Name:** Calendar Provider Sync
**ID:** FEAT-04.SPEC-003
**Type:** Integration
**Purpose:** The product connects to a Pro's personal Google or Apple calendar through a calendar-sync capability, performing the account-linking handshake, pulling busy/free time, and writing, moving, or removing Chairtime bookings on that calendar.
**Parent Feature:** FEAT-04 -- Two-Way Calendar Sync

## Scope and Non-Goals

**In Scope:**
- The account-linking handshake that establishes a Calendar Connection for a chosen kind
- Pulling busy/free periods from the connected calendar (never full event details)
- Writing, moving, and removing Chairtime booking events on the connected calendar
- User-facing behavior when the calendar-sync capability is slow, unavailable, or rejects a request
- Disclosure to the Pro about what calendar access is granted and what data is read or written

**Non-Goals:**
- Choosing the calendar-sync vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate beyond naming Google and Apple as the two supported calendar kinds.
- Deciding how busy/free periods feed the slot engine's computation -- owned by FEAT-04.SPEC-004 (Busy-Time Availability Feed); this spec only delivers the raw pulled periods.
- Deciding when to write, move, or remove a booking event -- owned by FEAT-04.SPEC-005 (Booking-to-Calendar Sync); this spec only performs the write/move/remove operation once instructed.
- Ongoing health monitoring and reconciliation after an outage -- owned by FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation); this spec reports handshake and operation outcomes, but does not decide when a connection is considered lapsed.
- Reading or storing full calendar event details (titles, descriptions, attendees, locations) -- excluded per product-features.md's Data Notes: only the minimum busy/free data needed to block slots is ever pulled.

## Capability Category

**Category:** Calendar sync
**Dependency Source:** ASMP-33 -- "Calendar-sync capability (reading and writing to a pro's personal calendar) -- required for the two-way sync described in BRIEF.md's Ecosystem & Integrations" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Calendar sync -- reading busy time from and writing bookings to a Pro's personal calendar" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-04, FEAT-03, FEAT-15)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision. BRIEF.md names Google and Apple as the two calendar kinds the product supports; which underlying calendar-sync capability performs the handshake for each is a Stage 4 choice.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Talia authorizes Chairtime to connect to her Google or Apple calendar | Connect a Google or Apple calendar | FEAT-04.SPEC-001 (Calendar Connection Setup) |
| Talia's personal busy time blocks Chairtime availability | Two-way calendar sync (busy time blocks availability) | FEAT-04.SPEC-004 (Busy-Time Availability Feed) |
| A confirmed, rescheduled, or cancelled Chairtime booking is written, moved, or removed on Talia's personal calendar | Two-way calendar sync (bookings appear on personal calendar) | FEAT-04.SPEC-005 (Booking-to-Calendar Sync) |
| Talia's dashboard reflects whether the connection is currently healthy | See connection health (connected / needs reconnection) | FEAT-04.SPEC-002 (Calendar Connection Status & Management), FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Authorization request for the chosen kind | Calendar Connection -- calendar_kind | Talia initiates a connection (FEAT-04.SPEC-001) or a reconnection (FEAT-04.SPEC-002) | The calendar-sync capability needs to know which calendar service to authorize against |
| Booking event details (service name, appointment start time, duration, in the Pro's timezone) | Booking -- service, start_time, duration | A booking is confirmed, rescheduled, or cancelled (FEAT-04.SPEC-005) | The capability needs enough detail to create or update a matching event on the connected calendar |

Client name, client phone, client notes, deposit amounts, and every other Booking or Client field never leave the product through this integration -- only the minimum event detail needed to place a placeholder on the calendar is sent.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Handshake outcome (authorized / declined / error) | The Pro completes or abandons the authorization step | Calendar Connection -- status (Syncing on authorized, no record created on declined/error) |
| Busy/free periods | The capability reports the Pro's personal-calendar busy time, on the near-immediate sync schedule | Calendar Connection -- busy_periods |
| Permission revoked or expired | The capability reports that access is no longer valid | Calendar Connection -- status (handled by FEAT-04.SPEC-006, which owns the transition to Needs Reconnection) |
| Write/move/remove confirmation for a booking event | A booking-driven write, move, or removal (FEAT-04.SPEC-005) completes | Calendar Connection -- last_successful_sync |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Handshake authorized | Talia completes the calendar provider's own authorization step for the chosen kind | Calendar Connection record created with calendar_kind set and status set to Syncing | FEAT-04.SPEC-001 shows the in-progress "Connecting..." state transition to a Connected/Syncing confirmation | FEAT-04.SPEC-001 (Calendar Connection Setup), FEAT-04.SPEC-002 (Calendar Connection Status & Management) |
| Handshake declined or abandoned | Talia declines or exits the provider's authorization step before completing it | No Calendar Connection record is created | FEAT-04.SPEC-001 shows "We couldn't connect to {kind}. Try again." with a Retry action | FEAT-04.SPEC-001 (Calendar Connection Setup) |
| Busy/free periods updated | The connected calendar reports a new or changed busy period, on the near-immediate sync schedule (product-features.md: within a couple of minutes) | Calendar Connection -- busy_periods refreshed | No direct user feedback on this spec's own screens; the resulting availability change is what FEAT-04.SPEC-004 makes visible | FEAT-04.SPEC-004 (Busy-Time Availability Feed) |
| Permission revoked or expired | The calendar provider reports that Chairtime's access is no longer valid (Pro revoked it externally, or the grant expired) | No change made directly by this spec; the health-status transition is owned by FEAT-04.SPEC-006, which this event feeds as its external-event trigger | No feedback delivered directly by this spec | FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) |
| Calendar sync health degrades | The calendar provider reports that Chairtime's access is no longer valid (same provider report as "Permission revoked or expired"), or a booking event write fails because the connection has lapsed | No change made directly by this spec; the Needs Reconnection transition is owned by FEAT-04.SPEC-006 | No feedback delivered directly by this spec; FEAT-12.SPEC-005 opens a "Reconnect calendar" Attention Item on the Pro's dashboard | FEAT-12.SPEC-005 (Attention Flag Aggregation), FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) |
| Booking event write/move/remove confirmed | A write, move, or removal instructed by FEAT-04.SPEC-005 completes on the connected calendar | Calendar Connection -- last_successful_sync updated | No direct user feedback on this spec's own screens; FEAT-04.SPEC-005 reflects the outcome | FEAT-04.SPEC-005 (Booking-to-Calendar Sync) |
| Booking event write/move/remove failed | The instructed operation could not complete (capability rejected it, or the connection has since lapsed) | No calendar-side change recorded; last_successful_sync is not updated | No feedback delivered directly by this spec; the failure is routed to FEAT-04.SPEC-005 as its failure outcome, which decides retry/flagging behavior | FEAT-04.SPEC-005 (Booking-to-Calendar Sync), FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-04.SPEC-001 (Calendar Connection Setup) | The "Connecting to {kind}..." indicator continues; after a noticeably long wait, the copy adds "Still working -- this is taking longer than usual." | The Connect button is disabled with "Connecting a calendar needs an internet connection. Check your connection and try again." if the capability cannot be reached at all; if the capability itself reports unavailability, the screen shows "We couldn't connect to {kind}. Try again." with Retry | The screen shows "We couldn't connect to {kind}. Try again." with Retry; no connection record is created |
| FEAT-04.SPEC-002 (Calendar Connection Status & Management) | The Reconnect action's in-progress indicator continues on the affected card; no other card is affected | The card shows a banner "We couldn't reach {kind} Calendar right now. Try reconnecting again shortly." and the card remains in Needs Reconnection status | The Reconnect action returns to its prior state with an inline message "Reconnecting to {kind} didn't work. Try again." |
| FEAT-04.SPEC-004 (Busy-Time Availability Feed) | Slot computation falls back to the most recently synced busy periods until a fresh pull completes, per the offline-degraded posture (ASMP-27) | The availability engine (FEAT-03) falls back to Chairtime-only data with the Pro-visible confidence narrowing described in FEAT-04.SPEC-006; never shown to clients | N/A -- busy-time pulls are read-only requests that either succeed, are slow, or are unavailable; there is no client-supplied content for the capability to reject |
| FEAT-04.SPEC-005 (Booking-to-Calendar Sync) | The write/move/remove is queued and retried per that spec's failure-path rules; the triggering booking action (confirm, reschedule, cancel) is never blocked or delayed by calendar-sync slowness | Same as Slow -- the write/move/remove is queued for retry once the capability is reachable again; the Chairtime-side booking state is authoritative and unaffected | The write/move/remove is treated as failed and routed to FEAT-04.SPEC-005's failure path (flagged for retry); the underlying Chairtime booking is never rolled back because of a calendar-write rejection |
| FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) | The reconciliation-driven fresh busy/free pull and any re-issued write/move/remove instructions continue waiting; the connection stays in Syncing (not yet Connected) until they complete, so the Pro never sees a premature "Connected" status that could mask unresolved drift | The reconciliation pull cannot reach the capability; the connection remains in Syncing rather than rolling back to Needs Reconnection (the reconnection handshake itself already succeeded), and FEAT-04.SPEC-006 retries the reconciliation automatically once the capability is reachable again | A re-issued write/move/remove instruction for a drifted booking is treated as failed for that booking only; the connection remains in Syncing until every drifted booking has reconciled successfully, and FEAT-04.SPEC-006 retries the rejected instruction automatically rather than marking reconciliation complete with unresolved drift |

## Consent and Disclosure

- **First connection disclosure** -- On FEAT-04.SPEC-001, before initiating the handshake, the explanation text states plainly: "Connecting lets Chairtime see your busy times so it never double-books you, and add your confirmed Chairtime appointments to this calendar automatically. Chairtime never sees event titles, guests, or details -- only whether a time is busy or free." This is shown every time the Pro reaches the setup screen, not just once, since it precedes every new connection.
- **Provider-side disclosure** -- The calendar provider's own authorization step (outside the product) states what access is being granted; Chairtime requests only the minimum scope needed to read busy/free time and write calendar events, never full calendar read access to event content.
- **What is never shared or read** -- Event titles, descriptions, attendee lists, and locations on the Pro's personal calendar are never pulled into the product. Client name, phone, notes, and deposit amounts are never sent to the calendar-sync capability -- only the service name, appointment time, and duration are written as the calendar event's content, per Data Exchanged above.
- **Ongoing visibility** -- FEAT-04.SPEC-002 always shows the Pro exactly which kinds are connected and their current status, so the Pro can review or revoke access (via Disconnect) at any time without needing to leave the product.

## Edge Cases

- **Busy/free event arrives for a connection that has since been disconnected** -- The event is discarded silently; a disconnected connection has no record to update (per the dependency map's hard-delete lifecycle for Calendar Connection), and no user feedback fires.
- **The same busy/free update is delivered twice** -- The second delivery changes nothing beyond re-confirming the same busy_periods value; no duplicate availability recomputation side effects beyond what FEAT-04.SPEC-004 already performs idempotently.
- **Booking write confirmation and a booking cancellation-driven removal arrive out of order for the same booking** -- The connected calendar reflects the most recent Chairtime-side booking state (per its own event timestamp), not arrival order: if the removal instruction was issued after the write instruction, the calendar event is removed even if its confirmation arrives before the write's confirmation.
- **Capability goes down mid-write** -- If the write was not confirmed complete, no calendar event is left in a half-written state from the product's perspective; FEAT-04.SPEC-005 treats it as failed and retries per its own rules. The underlying Chairtime booking is never altered because of a calendar-write failure.
- **Permission is revoked while a busy-time pull is in flight** -- The pull is treated as failed; the connection's last known busy_periods remain in effect until FEAT-04.SPEC-006 detects the lapse and sets status to Needs Reconnection.
- **Reconnection handshake completes for a kind whose prior connection record was already deleted (disconnect-then-reconnect)** -- A fresh Calendar Connection record is created per the dependency map's Delete/Archive note (reconnecting always creates a new connection rather than reviving the old one); this is treated identically to a first-time handshake authorization.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-001 (Calendar Connection Setup) | Triggered by (inbound) | "Connect" action initiates the handshake for the chosen kind |
| FEAT-04.SPEC-001 (Calendar Connection Setup) | Affects (outbound) | Handshake outcome (authorized/declined/error) surfaces here |
| FEAT-04.SPEC-002 (Calendar Connection Status & Management) | Triggered by (inbound) | "Reconnect" action re-initiates the handshake |
| FEAT-04.SPEC-002 (Calendar Connection Status & Management) | Affects (outbound) | Handshake and degradation outcomes surface here |
| FEAT-04.SPEC-004 (Busy-Time Availability Feed) | Triggers (outbound) | Busy/free-periods-updated event fires this automation |
| FEAT-04.SPEC-005 (Booking-to-Calendar Sync) | Triggered by (inbound) | Write/move/remove instructions originate from this automation |
| FEAT-04.SPEC-005 (Booking-to-Calendar Sync) | Affects (outbound) | Write/move/remove confirmation and failure outcomes surface here |
| FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) | Triggers (outbound) | Permission-revoked-or-expired event fires this automation as its external-event trigger |
| FEAT-12.SPEC-005 (Attention Flag Aggregation) | Triggers (outbound) | Calendar-sync-health-degrades inbound event is the external-event trigger for that automation's "Reconnect calendar" Attention Item |
| FEAT-04.SPEC-008 (Calendar Connection Rules) | References (inbound) | One-per-kind limit governs how many concurrent connections this integration ever maintains per Pro Account |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| calendar_handshake_outcome | calendar_kind, outcome (authorized / declined / error) | Handshake completes | supports success-metrics.md: "Calendar Sync Reliability" |
| calendar_busy_pull_completed | calendar_kind, latency_bucket (within target / slow) | A busy/free pull completes | supports success-metrics.md: "Calendar Sync Reliability" |
| calendar_booking_write_outcome | calendar_kind, operation (write / move / remove), outcome (succeeded / failed) | A booking-driven write, move, or removal completes or fails | supports success-metrics.md: "Calendar Sync Reliability" |
| calendar_capability_degraded | calendar_kind, condition (slow / down / rejects) | Any degradation condition is encountered | N/A -- no Stage 2 metric measures degradation frequency directly; retained so the calendar-sync capability's real-world reliability is observable rather than invisible, feeding the same trust question "Calendar Sync Reliability" targets |

## Acceptance Criteria

**FEAT-04.SPEC-003-AC-01:** Given Talia taps Connect for Google Calendar on FEAT-04.SPEC-001, when she completes the provider's authorization step, then a Calendar Connection record is created with calendar_kind Google and status Syncing.

**FEAT-04.SPEC-003-AC-02:** Given Talia begins the authorization step, when she declines or exits before completing it, then no Calendar Connection record is created and FEAT-04.SPEC-001 shows the retry error.

**FEAT-04.SPEC-003-AC-03:** Given Talia has an active Google connection, when the calendar-sync capability reports a new busy period on her personal calendar, then Calendar Connection.busy_periods refreshes within the near-immediate sync target and FEAT-04.SPEC-004 recomputes the blocked-availability signal.

**FEAT-04.SPEC-003-AC-04:** Given Talia's calendar permission is externally revoked, when the capability next reports this, then FEAT-04.SPEC-006 receives the event as its external-event trigger and begins its own health-transition handling.

**FEAT-04.SPEC-003-AC-05:** Given a Chairtime booking for Talia is confirmed, when FEAT-04.SPEC-005 instructs a calendar write, then this integration sends only the service name, appointment start time, and duration -- never the client's name, phone, or notes.

**FEAT-04.SPEC-003-AC-06:** Given a booking write instruction is sent, when the capability confirms the event was created, then Calendar Connection.last_successful_sync updates and FEAT-04.SPEC-005 reflects the success outcome.

**FEAT-04.SPEC-003-AC-07:** Given a booking write instruction is sent, when the capability is down and cannot be reached, then FEAT-04.SPEC-005 receives the failure outcome and the underlying Chairtime booking is unchanged.

**FEAT-04.SPEC-003-AC-08:** Given Talia is on FEAT-04.SPEC-001 and taps Connect, when the capability is slow to respond, then the screen shows "Still working -- this is taking longer than usual." rather than an error.

**FEAT-04.SPEC-003-AC-09:** Given Talia is on FEAT-04.SPEC-002 and taps Reconnect, when the capability rejects the reconnection attempt, then the card shows "Reconnecting to {kind} didn't work. Try again." and the connection remains in Needs Reconnection status.

**FEAT-04.SPEC-003-AC-10:** Given Talia reaches FEAT-04.SPEC-001 for the first time, when she views the explanation text before connecting, then it states plainly that Chairtime sees only busy/free time, never event titles, guests, or details.

**FEAT-04.SPEC-003-AC-11:** Given a busy/free-periods-updated event is delivered twice for the same period, when the second delivery arrives, then Calendar Connection.busy_periods reflects no change beyond the first delivery, and no duplicate recomputation side effects occur beyond FEAT-04.SPEC-004's normal idempotent handling.

**FEAT-04.SPEC-003-AC-12:** Given a booking is cancelled and its calendar removal instruction is issued after an earlier reschedule's move instruction, when both confirmations arrive out of order, then the connected calendar reflects the removal (the most recent Chairtime-side state), not the stale move.

**FEAT-04.SPEC-003-AC-13:** Given Talia disconnects and later reconnects the same calendar kind, when the new handshake completes, then a fresh Calendar Connection record is created rather than reviving the disconnected one.

**FEAT-04.SPEC-003-AC-14:** Given a busy/free pull arrives for a connection that Talia disconnected moments earlier, when the event is processed, then it is discarded silently with no user feedback and no record updated.

**FEAT-04.SPEC-003-AC-15:** Given FEAT-04.SPEC-006 is reconciling a reconnected connection, when the reconciliation-driven busy/free pull or a re-issued write/move/remove instruction hits the capability being slow, down, or rejecting, then the connection remains in Syncing (never a premature Connected, and never rolled back to Needs Reconnection) and FEAT-04.SPEC-006 retries automatically once the capability responds.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 6 | 6 |
| Degradation Paths | 17 (6 screens/specs x 3 conditions, 1 N/A cell excluded) | 17 |
| Consent and Disclosure | 4 | 4 |
| Edge Cases | 6 | 6 |
