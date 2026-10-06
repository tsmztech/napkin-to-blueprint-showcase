---
document_type: spec
spec_type: integration
spec_id: FEAT-03.SPEC-006
spec_name: Calendar Busy-Time Consumption & Degraded Mode
spec_slug: calendar-busy-time-consumption-degraded-mode
parent_feature: FEAT-03
parent_feature_name: Real-Time Slot Availability Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 11
---

# Integration Spec: Calendar Busy-Time Consumption & Degraded Mode

## Overview

**Name:** Calendar Busy-Time Consumption & Degraded Mode
**ID:** FEAT-03.SPEC-006
**Type:** Integration
**Purpose:** Consumes busy/free periods from the Pro's connected personal calendar (the connection itself owned by FEAT-04) as an additional availability constraint, and defines the fallback behavior -- Chairtime-only data with a Pro-only reduced-confidence flag -- when that sync becomes unavailable.
**Parent Feature:** FEAT-03 -- Real-Time Slot Availability Engine

## Scope and Non-Goals

**In Scope:**
- Consuming the busy_periods data FEAT-04's Calendar Connection already reads from the Pro's personal calendar
- Supplying those busy periods to FEAT-03.SPEC-001's slot computation as an additional occupancy constraint
- Detecting when calendar sync health degrades or lapses and falling back to Chairtime-only data
- Surfacing reduced confidence to the Pro only, on their own dashboard, never to the client

**Non-Goals:**
- Establishing, authenticating, or maintaining the Calendar Connection itself (connecting, reconnecting, disconnecting, and reporting sync health) -- owned entirely by FEAT-04 (Two-Way Calendar Sync); this spec only consumes what FEAT-04 already reads
- Writing Chairtime bookings out to the Pro's personal calendar -- also owned by FEAT-04; this spec is inbound-only (busy time in), never outbound booking writes
- Deciding the actual slot computation logic that combines busy time with other occupancy sources -- owned by FEAT-03.SPEC-001, which this spec feeds
- Storing full calendar event details -- excluded per the dependency map's Calendar Connection entity definition: "only busy/free periods are kept, never event titles or details"

## Capability Category

**Category:** Calendar sync
**Dependency Source:** ASMP-33 -- "Calendar-sync capability (reading and writing to a pro's personal calendar) -- required for the two-way sync described in BRIEF.md's Ecosystem & Integrations. Without it, the availability engine cannot account for a pro's real-world commitments outside Chairtime, directly threatening the 'never double-book' correctness bar." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Calendar sync -- reading busy time from and writing bookings to a Pro's personal calendar (ASMP-33)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-04, FEAT-03, FEAT-15; Integration Specs: FEAT-04.SPEC-003 for the connection handshake and booking writes, FEAT-03.SPEC-006 -- this spec -- for busy-time consumption and degraded mode)
**Vendor Mandate:** None recorded in BRIEF.md for the specific calendar-sync mechanics this spec covers -- BRIEF.md's Ecosystem & Integrations names Google and Apple as the two supported personal-calendar kinds the Pro may connect through FEAT-04, but vendor selection for the underlying sync mechanism is a Stage 4 decision; this spec stays at the category level throughout.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| A time slot that conflicts with Talia's personal calendar event is never offered to a client | Compute the live set of open slots for a given service and date range | FEAT-03.SPEC-001 (Slot Availability Computation) |
| Talia sees a reduced-confidence indicator when her calendar sync has lapsed, so she knows Chairtime-only data is currently in effect | Never silently double-book (BRIEF.md, Vision) | FEAT-12 (Pro Daily Schedule Dashboard, Pro-visible banner) |
| A client is never shown any indication that calendar confidence is reduced -- the client always sees a clean, ordinary slot list | Never expose Pro-side operational detail to a client | FEAT-05 (Public Booking Page & Booking Flow) |

## Data Exchanged

**Leaves the product:**

No data leaves the product through this spec. This spec is inbound-only: it consumes busy_periods already read by FEAT-04's own connection handshake (FEAT-04.SPEC-003 owns any outbound data, such as Chairtime booking writes to the personal calendar). Nothing from this spec's own processing -- no Booking, Client, or Service detail -- is sent to the calendar-sync capability.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|------------------------------|
| Busy/free periods (minimum data needed to block slots -- never full event details) | FEAT-04 completes a sync cycle with the Pro's connected personal calendar | Calendar Connection -- busy_periods, last_successful_sync (read by this spec; written by FEAT-04) |
| Sync health status (Connected / Syncing / Needs Reconnection / Disconnected) | FEAT-04 detects a change in connection health | Calendar Connection -- status (read by this spec; written by FEAT-04) |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Busy periods refreshed | FEAT-04 completes a sync cycle successfully | None written by this spec (Calendar Connection is FEAT-04's write) | None directly -- the next slot computation simply reflects the refreshed busy periods | FEAT-03.SPEC-001 (consumes the refreshed data) |
| Sync health degrades (status moves to Needs Reconnection or Disconnected) | FEAT-04 detects the connection has lapsed | None written by this spec; this spec reads the status change and sets its own confidence flag to Reduced for consumption by FEAT-03.SPEC-001 | Talia sees a reduced-confidence banner on her dashboard; Riley sees no change at all | FEAT-03.SPEC-001, FEAT-12 (Pro Daily Schedule Dashboard) |
| Sync health recovers (status returns to Connected) | Talia reconnects or the connection self-heals (FEAT-04) | This spec's confidence flag returns to Normal | Talia's reduced-confidence banner clears | FEAT-03.SPEC-001, FEAT-12 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|---------------------|
| FEAT-05 (Public Booking Page & Booking Flow) | No visible effect -- the client-facing slot list computes from whatever busy-period data is currently available (last successfully synced, or none) with no waiting state tied to calendar sync specifically | Availability falls back to Chairtime-only data (Availability Rule, Booking, Time Block, Recurring Series, Slot Hold); the client sees the ordinary computed list with no indication anything is degraded, never a client-facing error or delay | N/A -- this spec never sends a request the calendar-sync capability can reject; it only consumes data FEAT-04 already reads |
| FEAT-12 (Pro Daily Schedule Dashboard) | A brief "syncing" indication may appear on the connection-health banner (owned by FEAT-04); this spec's own confidence flag stays Normal until sync genuinely lapses | Talia sees a reduced-confidence banner: "Availability may not reflect your personal calendar right now -- reconnect to restore full accuracy." The rest of her dashboard remains fully usable | N/A -- same as above; this spec has no request path a capability can reject |

## Consent and Disclosure

- **No new disclosure moment introduced by this spec** -- the consent and disclosure moment for sharing calendar access (what data is read, what a Pro is told) belongs to FEAT-04's connection handshake (FEAT-04.SPEC-003), since this spec introduces no new outbound data of its own; it only reads what FEAT-04 has already disclosed and captured.
- **What is never shared or surfaced** -- full calendar event details (titles, descriptions, attendees) are never read or stored by this spec or by FEAT-04; only the minimum busy/free period data needed to block slots ever enters Chairtime, per the Calendar Connection entity's Data Sensitivity note, and none of it is ever shown to a client.
- **Reduced-confidence disclosure (Pro-facing only)** -- when sync health degrades, Talia's dashboard states plainly: "Availability may not reflect your personal calendar right now -- reconnect to restore full accuracy." This is shown to the Pro only; the client-facing booking page carries no equivalent notice, per this feature's own design intent that a client "must never be shown a slot that turns out to be unavailable" while also never being shown Pro-side operational detail.

## Edge Cases

- **Busy-period data refreshes while a client is mid-computation** -- The computation in flight uses whatever busy-period snapshot was current when it started; a newly refreshed set of busy periods is reflected starting with the next computation, never retroactively altering a result already returned.
- **Sync health degrades and then recovers within the same minute** -- The confidence flag follows the most recent status change; a rapid degrade-then-recover sequence simply leaves the flag at Normal once recovery is detected, with no lingering reduced-confidence banner.
- **The Pro has never connected a calendar at all (no Calendar Connection exists)** -- Busy periods contribute nothing (an empty set), and no reduced-confidence flag is raised, since "reduced confidence" applies only to a lapsed existing connection, not to the ordinary case of no connection ever being made.
- **Calendar-sync capability reports busy-period data twice for the same sync cycle** -- The second delivery changes nothing beyond the current busy-period snapshot it already reflects; no duplicate exclusion or double-counting occurs in the next computation.
- **Sync health status arrives out of order (a "recovered" event processed before an earlier "degraded" event)** -- The confidence flag reflects the most recent event by its own event time, not arrival order, consistent with FEAT-04's own health reporting.
- **Degradation strikes mid-way through a client's checkout (busy-period data becomes stale while a hold is already Active)** -- The already-created Slot Hold (FEAT-03.SPEC-002) is unaffected; degraded confidence changes nothing about slots already held or already computed in the current view, since the correctness guarantee for an in-progress checkout rests on the hold itself, not on live calendar freshness.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-003 (Connection handshake, busy-time pull, booking write/move/remove -- owned by FEAT-04) | Triggered by (inbound) | Supplies the busy_periods and status this spec reads and consumes |
| FEAT-03.SPEC-001 (Slot Availability Computation) | Affects (outbound) | Busy periods and the confidence flag are consumed as an additional occupancy constraint |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | Reduced-confidence banner surfaces here, Pro-only |
| FEAT-05 (Public Booking Page & Booking Flow) | Affects (outbound) | Receives only the ordinary computed list; never a degraded-mode indicator |

## Analytics and Success Signals

- **calendar_busy_time_consumed** (pro_account_id, sync_confidence: normal / reduced) -- supports success-metrics.md: "Calendar Sync Reliability"
- **calendar_confidence_degraded** (pro_account_id) -- supports success-metrics.md: "Calendar Sync Reliability"
- **calendar_confidence_recovered** (pro_account_id) -- supports success-metrics.md: "Calendar Sync Reliability"
- **slot_computation_used_degraded_calendar_data** (pro_account_id) -- supports success-metrics.md: "Zero Double-Booking Confidence" (measures how often the fail-safe Chairtime-only fallback is exercised, which is exactly the correctness guarantee this metric validates)

## Acceptance Criteria

**FEAT-03.SPEC-006-AC-01:** Given Talia's calendar sync is healthy and reports a busy period overlapping a candidate slot, when FEAT-03.SPEC-001 computes the open-slot list, then that candidate is excluded from Riley's list.

**FEAT-03.SPEC-006-AC-02:** Given Talia's calendar sync has just degraded (FEAT-04 reports Needs Reconnection), when Talia opens her dashboard, then she sees "Availability may not reflect your personal calendar right now -- reconnect to restore full accuracy."

**FEAT-03.SPEC-006-AC-03:** Given Talia's calendar sync is degraded, when Riley requests the slot list, then Riley sees the ordinary computed list with no indication anything is degraded.

**FEAT-03.SPEC-006-AC-04:** Given Talia's calendar sync recovers after a degraded period, when Talia next opens her dashboard, then the reduced-confidence banner clears.

**FEAT-03.SPEC-006-AC-05:** Given Talia has never connected a personal calendar, when a slot is computed, then no reduced-confidence flag is raised, and busy periods contribute nothing to the computation.

**FEAT-03.SPEC-006-AC-06:** Given calendar sync degrades and recovers within the same minute, when the confidence flag is evaluated, then it reflects Normal, with no lingering banner shown to Talia.

**FEAT-03.SPEC-006-AC-07:** Given busy-period data is delivered twice for the same sync cycle, when the second delivery arrives, then no duplicate exclusion or double-counting occurs in the next computation.

**FEAT-03.SPEC-006-AC-08:** Given a "recovered" health event and an earlier "degraded" event arrive out of order, when the confidence flag is evaluated, then it reflects the most recent event by its own event time, not arrival order.

**FEAT-03.SPEC-006-AC-09:** Given Riley's checkout hold is already Active when calendar sync degrades mid-flow, when the degradation is detected, then Riley's already-created hold is unaffected.

**FEAT-03.SPEC-006-AC-10:** Given a client attempts to view any calendar-sync detail on the booking page, when the page renders, then no calendar-sync status or confidence indicator of any kind is shown to the client.

**FEAT-03.SPEC-006-AC-11:** Given Support (Platform Operator) opens a Pro's account to troubleshoot a reported conflict, when Support views calendar-sync status, then Support sees connection health only (View-only), never full calendar event details.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 3 | 3 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 4 (2 screens; 1 N/A cell excluded) | 4 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 6 | 6 |
