---
document_type: spec
spec_type: automation
spec_id: FEAT-04.SPEC-004
spec_name: Busy-Time Availability Feed
spec_slug: busy-time-availability-feed
parent_feature: FEAT-04
parent_feature_name: Two-Way Calendar Sync
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 10
---

# Automation Spec: Busy-Time Availability Feed

## Overview

**Name:** Busy-Time Availability Feed
**ID:** FEAT-04.SPEC-004
**Type:** Automation
**Purpose:** Turns the busy/free periods synced from a Pro's connected personal calendar into the blocked-availability signal the slot engine consumes, so external commitments block Chairtime availability.
**Parent Feature:** FEAT-04 -- Two-Way Calendar Sync

## Scope and Non-Goals

**In Scope:**
- Recomputing the blocked-availability signal whenever synced busy/free data changes
- Recomputing the signal on first connection, once the initial busy-time pull completes
- Marking a connection's contribution to the signal as reduced-confidence when the connection is not currently healthy
- Removing a connection's contribution to the signal entirely once it is disconnected

**Non-Goals:**
- Pulling busy/free data from the calendar provider -- owned by FEAT-04.SPEC-003 (Calendar Provider Sync); this automation only consumes what has already been pulled
- Combining the blocked-availability signal with working hours, buffers, time blocks, and existing bookings to produce the final open-slot list -- owned by FEAT-03 (Real-Time Slot Availability Engine); this automation feeds one input into that computation, it does not perform it
- Deciding when a connection is unhealthy enough to narrow confidence -- owned by FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation); this automation reads that status, it does not set it
- Writing Chairtime bookings to the external calendar -- owned by FEAT-04.SPEC-005 (Booking-to-Calendar Sync); this automation is one-directional (calendar-to-Chairtime), not the reverse

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Busy/free periods updated | FEAT-04.SPEC-003 (Calendar Provider Sync) | Fires whenever the calendar-sync capability reports a new or changed busy period for a connection currently in Connected or Syncing status | Calendar Connection reference, calendar_kind, the updated busy_periods |
| Connection health status changes | FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) | Fires when a connection's status transitions to or from Needs Reconnection | Calendar Connection reference, new status, previous status |
| Connection disconnected | FEAT-04.SPEC-002 (Calendar Connection Status & Management) | Fires when a Pro disconnects a calendar connection | Calendar Connection reference (about to be deleted) |
| Initial busy-time pull completes | FEAT-04.SPEC-003 (Calendar Provider Sync) | Fires once, immediately after a new connection's first busy-time pull succeeds | Calendar Connection reference, calendar_kind, initial busy_periods |

## Processing Logic

1. Receive the triggering Calendar Connection reference and the reason for the recompute (busy-time update, health change, disconnect, or initial pull).
2. If the trigger is a disconnect, remove that connection's contribution to the blocked-availability signal entirely and stop (no further steps).
3. Otherwise, read the connection's current busy_periods and status.
4. If status is Connected or Syncing, mark the connection's busy_periods as full-confidence input to the blocked-availability signal.
5. If status is Needs Reconnection, mark the connection's most recently synced busy_periods as reduced-confidence input: the last known busy time remains in effect (per product-features.md's offline-degraded posture and ASMP-27), but the signal now carries a confidence flag.
6. Recompute the combined blocked-availability signal for the Pro Account from all of that Pro's currently contributing connections (at most two, one per kind).
7. Publish the recomputed signal for the Real-Time Slot Availability Engine (FEAT-03) to consume on its next slot computation; the confidence flag (if any) is included so FEAT-03 can decide whether to narrow its own Pro-visible confidence display, per XBR-13.
8. Record the recompute as complete, including the connection reference and the trigger reason, for the health monitor's reconciliation reference point (FEAT-04.SPEC-006).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Signal recomputed (full confidence) | Busy/free update or initial pull for a Connected/Syncing connection | Blocked-availability signal updated; no Calendar Connection field changed by this automation itself | None directly -- the Pro sees the effect only if it changes an offered slot, via FEAT-03 | FEAT-03 (Real-Time Slot Availability Engine) |
| Signal recomputed (reduced confidence) | Connection transitions to Needs Reconnection | Blocked-availability signal updated with a confidence flag for that connection's contribution | Pro-visible confidence narrowing surfaces on FEAT-03's slot computation and FEAT-12's dashboard; never shown to clients, per XBR-13 | FEAT-03 (Real-Time Slot Availability Engine), FEAT-12 (Pro Daily Schedule Dashboard) |
| Signal restored to full confidence | Connection transitions back to Connected after reconnection | Blocked-availability signal's confidence flag for that connection is cleared | Pro-visible confidence indicator returns to normal | FEAT-03 (Real-Time Slot Availability Engine), FEAT-12 (Pro Daily Schedule Dashboard) |
| Connection contribution removed | Connection is disconnected | That connection's busy_periods no longer contribute to the blocked-availability signal | None directly; slots that were blocked only by that calendar's busy time may become available on the next computation | FEAT-03 (Real-Time Slot Availability Engine) |
| Recompute failure | The automation cannot complete the recompute (processing error) | No signal change is published; the previous signal remains in effect | None directly to the Pro; the failure is logged for FEAT-04.SPEC-006 to detect via its own health checks | FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) |

## Data Model

**Reads:** Calendar Connection -- busy_periods, status, calendar_kind for every connection belonging to the Pro Account whose signal is being recomputed.
**Creates:** None -- the blocked-availability signal is a derived, in-memory computation output, not a persisted new entity.
**Updates:** None -- this automation does not write to the Calendar Connection record; status transitions belong to FEAT-04.SPEC-006 and last_successful_sync updates belong to FEAT-04.SPEC-003/FEAT-04.SPEC-005 (and to FEAT-04.SPEC-006 on successful reconciliation).
**Deletes:** None.

## Business Rules

- XBR-01: a slot is offered only if it passes the live slot check, which includes this signal's blocked periods; this automation is one of the inputs FEAT-03 owns the final decision over.
- XBR-13: if sync lapses, availability falls back to Chairtime data with reduced confidence shown to the Pro only, never to clients -- this automation is the mechanism that carries that reduced-confidence flag into FEAT-03's computation.
- Only busy/free periods are ever part of this signal -- never event titles or details, per product-features.md's Data Notes.
- A Pro Account contributes at most two connections' worth of busy/free data (one Google, one Apple) to its own signal, per FEAT-04.SPEC-008's one-per-kind limit.

## Edge Cases

- **Both connections report busy/free updates at effectively the same time** -- Each update triggers its own recompute of the combined signal; the second recompute incorporates whichever busy_periods were current at that moment for both connections, so the final published signal reflects both updates regardless of processing order.
- **A busy-time update arrives for a connection that was disconnected moments earlier** -- The update is discarded; a disconnected connection has no record to attribute the update to, so it contributes nothing to the signal.
- **Recompute is triggered while a previous recompute for the same Pro Account is still in flight** -- The later trigger's recompute supersedes the earlier one: only the most recently completed recompute's result is published, so a slower first run never overwrites a faster, more current second run.
- **Concurrent trigger firing (busy-time update and a health-status change for the same connection at effectively the same time)** -- Both conditions are read together at the moment the recompute runs (Step 3), so a single recompute reflects both the freshest busy_periods and the freshest status; no separate, conflicting signal versions are published.
- **Trigger fires while a previous run is in flight for a different Pro Account's connection** -- Recomputes for different Pro Accounts are entirely independent and never queue behind each other.
- **Initial pull completes for a connection whose Pro Account has no availability rule set yet** -- The signal is still recomputed and published; FEAT-03 simply has nothing to combine it with until FEAT-02 (Availability & Working Hours Setup) is completed, which is an expected onboarding-sequencing state, not an error.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-003 (Calendar Provider Sync) | Triggered by (inbound) | Busy/free-periods-updated and initial-pull-completed events fire this automation |
| FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) | Triggered by (inbound) | Health-status-change events fire this automation's confidence recompute |
| FEAT-04.SPEC-002 (Calendar Connection Status & Management) | Triggered by (inbound) | Disconnect action fires this automation's contribution-removal path |
| FEAT-03 (Real-Time Slot Availability Engine) | Affects (outbound) | Published blocked-availability signal feeds FEAT-03's live slot computation |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | Reduced-confidence flag feeds the dashboard's degraded-confidence display |

## Analytics and Success Signals

- **busy_time_signal_recomputed** (calendar_kind, confidence: full / reduced, trigger_reason: busy_update / health_change / disconnect / initial_pull) -- supports success-metrics.md: "Calendar Sync Reliability"
- **busy_time_signal_recompute_failed** (calendar_kind, trigger_reason) -- supports success-metrics.md: "Calendar Sync Reliability"
- **double_booking_signal_check** (result: no_conflict_detected) -- supports success-metrics.md: "Zero Double-Booking Confidence" (this automation's recompute is one of the inputs the availability engine relies on to keep that guarantee true)

## Acceptance Criteria

**FEAT-04.SPEC-004-AC-01:** Given Talia's connected Google calendar reports a new busy period, when FEAT-04.SPEC-003 delivers the update, then this automation recomputes the blocked-availability signal at full confidence within the near-immediate sync target.

**FEAT-04.SPEC-004-AC-02:** Given Talia's Apple connection transitions to Needs Reconnection, when the health-status-change trigger fires, then the signal is recomputed with a reduced-confidence flag for that connection while its most recently synced busy time remains in effect.

**FEAT-04.SPEC-004-AC-03:** Given Talia's Apple connection is later reconnected and returns to Connected, when the health-status-change trigger fires again, then the signal's confidence flag for that connection clears and the signal returns to full confidence.

**FEAT-04.SPEC-004-AC-04:** Given Talia disconnects her Google calendar, when the disconnect trigger fires, then that connection's busy_periods no longer contribute to the signal, and slots blocked only by that calendar's busy time may become available on the next computation.

**FEAT-04.SPEC-004-AC-05:** Given Talia completes her first calendar connection, when the initial busy-time pull succeeds, then this automation recomputes the signal for the first time and publishes it for FEAT-03 to consume.

**FEAT-04.SPEC-004-AC-06:** Given a recompute fails due to a processing error, when the failure occurs, then the previous signal remains in effect and no Chairtime slot is shown as available based on stale, incorrectly-cleared busy time.

**FEAT-04.SPEC-004-AC-07:** Given a busy-time update arrives for a connection Talia disconnected moments earlier, when the automation processes it, then the update is discarded and contributes nothing to the signal.

**FEAT-04.SPEC-004-AC-08:** Given Talia's Google and Apple connections both report busy-time updates at effectively the same time, when both recomputes run, then the final published signal reflects both connections' freshest busy_periods.

**FEAT-04.SPEC-004-AC-09:** Given a recompute for Talia's Pro Account is triggered while a previous recompute for the same account is still in flight, when both complete, then only the most recently completed recompute's result is published.

**FEAT-04.SPEC-004-AC-10:** Given Talia's Pro Account has no Availability Rule configured yet, when a connection's initial pull completes, then the signal is still recomputed and published, ready for FEAT-03 to combine once availability setup completes.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 4 | 4 |
| Outcome Paths | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
