---
document_type: spec
spec_type: automation
spec_id: FEAT-04.SPEC-006
spec_name: Sync Health Monitor & Reconciliation
spec_slug: sync-health-monitor-reconciliation
parent_feature: FEAT-04
parent_feature_name: Two-Way Calendar Sync
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 12
---

# Automation Spec: Sync Health Monitor & Reconciliation

## Overview

**Name:** Sync Health Monitor & Reconciliation
**ID:** FEAT-04.SPEC-006
**Type:** Automation
**Purpose:** Watches each connection's validity, degrades the Pro-visible availability confidence on failure, and reconciles drift once a lapsed connection is restored.
**Parent Feature:** FEAT-04 -- Two-Way Calendar Sync

## Scope and Non-Goals

**In Scope:**
- Detecting a revoked or expired calendar permission and transitioning the connection to Needs Reconnection
- Transitioning a reconnected connection back to Connected
- Reconciling busy time and previously-written bookings against the calendar after a reconnection to resolve drift from the outage
- Feeding Support's read-only view of connection health

**Non-Goals:**
- Deciding what a reduced-confidence signal means to the slot engine's own computation -- owned by FEAT-03 (Real-Time Slot Availability Engine); this automation only sets and clears the connection's status and lets FEAT-04.SPEC-004 carry that status into the signal
- Performing the reconnection handshake itself -- owned by FEAT-04.SPEC-003 (Calendar Provider Sync); this automation reacts to the handshake's outcome
- Sending the reconnect alert to the Pro -- owned by FEAT-04.SPEC-007 (Calendar Reconnection Alert); this automation's detection is that notification's trigger
- Disconnecting a calendar -- excluded per the dependency map's Contention note: disconnect is a deliberate Pro action on FEAT-04.SPEC-002, never something this monitor initiates on its own

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Permission revoked or expired reported | FEAT-04.SPEC-003 (Calendar Provider Sync) | Fires when the calendar-sync capability reports that access is no longer valid for a connection currently in Connected or Syncing status | Calendar Connection reference, calendar_kind |
| Scheduled connection-health check runs | System (schedule-based) | Fires on a recurring schedule for every active connection, to catch a revoked permission the capability did not proactively report | Calendar Connection reference, calendar_kind, last_successful_sync |
| Reconnection handshake succeeds | FEAT-04.SPEC-003 (Calendar Provider Sync) | Fires when a Pro completes reconnection for a connection previously in Needs Reconnection status | Calendar Connection reference, calendar_kind |
| Booking-calendar write fails after retries | FEAT-04.SPEC-005 (Booking-to-Calendar Sync) | Fires when a booking write/move/remove fails persistently, which may itself indicate a lapsed connection | Calendar Connection reference, calendar_kind, affected Booking reference |

## Processing Logic

1. Receive the triggering Calendar Connection reference and the reason (permission report, scheduled check, reconnection success, or persistent write failure).
2. If the trigger is a permission-revoked report or a scheduled check that finds the permission invalid, set the connection's status to Needs Reconnection.
3. Once status is set to Needs Reconnection, signal FEAT-04.SPEC-004 to recompute the blocked-availability signal with the reduced-confidence flag for this connection.
4. Signal FEAT-04.SPEC-007 (Calendar Reconnection Alert) that this connection now needs reconnecting, so the dashboard banner fires.
5. If the trigger is a persistent booking-write failure, check whether the connection's permission is still valid; if invalid, follow steps 2-4; if the permission itself is still valid (a transient capability failure rather than a lapsed connection), do not change status -- the failure is logged for FEAT-04.SPEC-005's own retry handling, not treated as a health event.
6. If the trigger is a reconnection handshake success, set the connection's status to Syncing.
7. Pull the calendar's current busy/free state fresh (via FEAT-04.SPEC-003) and compare it against the connection's last known busy_periods from before the outage; update busy_periods to the fresh pull.
8. Compare the connection's previously-written Chairtime bookings (those confirmed, rescheduled, or cancelled during the outage window) against the calendar's current event state; for each booking whose calendar-side state does not match the Chairtime-side state, re-issue a write instruction (if the booking is confirmed but no calendar event exists), a move instruction (if the booking's start_time no longer matches the calendar event's time), or a remove instruction (if the booking is cancelled but its calendar event still exists) via FEAT-04.SPEC-003.
9. Once reconciliation completes (fresh busy time pulled, all drifted bookings re-synced), set the connection's status to Connected and clear the reduced-confidence flag via FEAT-04.SPEC-004.
10. Record the reconciliation outcome (drift found and corrected, or no drift found) for the connection's history.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Connection marked Needs Reconnection | Permission revoked/expired detected (report or scheduled check) | Calendar Connection.status set to Needs Reconnection | Reconnect banner fires (FEAT-04.SPEC-007); FEAT-04.SPEC-002's card shows the status and Reconnect action; FEAT-12's dashboard confidence display narrows | FEAT-04.SPEC-004, FEAT-04.SPEC-007, FEAT-04.SPEC-002, FEAT-12 |
| Reconnection in progress | Pro initiates reconnect via FEAT-04.SPEC-002 | Calendar Connection.status set to Syncing | FEAT-04.SPEC-002 shows the in-progress "Reconnecting..." state | FEAT-04.SPEC-002 |
| Reconciliation completed, drift corrected | Reconnection succeeds and one or more bookings/busy periods had drifted during the outage | Calendar Connection.busy_periods refreshed; drifted booking events re-written/moved/removed on the calendar; status set to Connected | No explicit Pro-facing message beyond the status returning to Connected -- the reconciliation is designed to be invisible when it succeeds, per the "never silently double-book" bar being satisfied automatically | FEAT-04.SPEC-002, FEAT-04.SPEC-004 |
| Reconciliation completed, no drift found | Reconnection succeeds and nothing had drifted during the outage | Calendar Connection.busy_periods refreshed (even if unchanged); status set to Connected | Same as above -- status returns to Connected | FEAT-04.SPEC-002, FEAT-04.SPEC-004 |
| Transient failure, no status change | Persistent booking-write failure where the connection's permission is still valid | None -- status is not changed | None from this automation; FEAT-04.SPEC-005's own failure flag (dashboard attention item) already covers the Pro-visible feedback | FEAT-04.SPEC-005 |
| Monitor check failure | The health check itself cannot complete (processing error) | No status change; the previous status remains in effect | None directly; the next scheduled check retries | -- |

## Data Model

**Reads:** Calendar Connection -- status, last_successful_sync, busy_periods, calendar_kind; Booking -- state, start_time, service (for bookings confirmed/rescheduled/cancelled during an outage window, to detect drift).
**Creates:** None.
**Updates:** Calendar Connection -- status (Connected <-> Syncing <-> Needs Reconnection), busy_periods (refreshed on reconciliation), last_successful_sync (on successful reconciliation).
**Deletes:** None.

## Business Rules

- XBR-13: if sync lapses, availability falls back to Chairtime data with reduced confidence shown to the Pro only, never to clients -- this automation is the owner of that status transition.
- Reconciliation on reconnect is the dependency map's Contention resolution for Calendar Connection: "bookings already written to the personal calendar are reconciled on reconnect" -- this automation performs exactly that reconciliation.
- Health-status updates are last-write-wins, per the dependency map's Contention note for Calendar Connection: if two health signals arrive close together (e.g., a proactive report and a scheduled check), the most recent one determines the connection's status.
- A connection never transitions itself to Disconnected -- only a Pro-initiated disconnect (FEAT-04.SPEC-002) removes the record; this automation's vocabulary is limited to Connected, Syncing, and Needs Reconnection.
- A transient booking-write failure with a still-valid permission is not treated as a health event -- only a genuinely invalid permission triggers Needs Reconnection, so the Pro is never alerted about a connection that is actually fine.

## Edge Cases

- **Permission is revoked and then re-granted before the scheduled check runs** -- If FEAT-04.SPEC-003 reports the revocation first, status transitions to Needs Reconnection regardless of the subsequent re-grant timing; the Pro still sees the reconnect prompt and completes reconnection through the normal flow (this avoids silently trusting a permission state that was invalid even briefly).
- **Two health signals arrive in close succession (a proactive revocation report and a scheduled check finding the same connection invalid)** -- Both would set the same status (Needs Reconnection); the connection settles at Needs Reconnection regardless of which signal is processed first or second, consistent with last-write-wins producing the same outcome either way.
- **Reconnection succeeds but the reconciliation pull itself fails** -- Status remains Syncing (not yet Connected) until reconciliation completes successfully; the Pro sees "Syncing" rather than a premature "Connected" that could mask unresolved drift. The reconciliation is retried automatically.
- **A booking is cancelled during the outage window and its removal was never written** -- Reconciliation detects the mismatch (Chairtime shows Cancelled, calendar still shows the event) and re-issues the removal instruction, closing the gap discovered by the outage.
- **Trigger fires while a previous run is in flight for the same connection** -- A second health signal for a connection already mid-reconciliation is queued and re-evaluated once the in-flight reconciliation completes, rather than starting a second, overlapping reconciliation against the same connection.
- **Concurrent trigger firing for two different connections belonging to the same Pro Account** -- Each connection's health check and reconciliation proceeds independently; one connection's Needs Reconnection status never blocks or delays the other's monitoring.
- **Pro disconnects a connection while reconciliation is in flight** -- Per FEAT-04.SPEC-008's contention rule, the disconnect wins: the in-flight reconciliation is discarded and no further reconciliation is attempted against the now-removed connection.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-003 (Calendar Provider Sync) | Triggered by (inbound) | Permission-revoked reports and reconnection-success events fire this automation |
| FEAT-04.SPEC-003 (Calendar Provider Sync) | Triggers (outbound) | Reconciliation's fresh pull and re-issued writes/moves/removes are carried out through this spec |
| FEAT-04.SPEC-005 (Booking-to-Calendar Sync) | Triggered by (inbound) | Persistent write-failure events feed this automation's transient-vs-health check |
| FEAT-04.SPEC-004 (Busy-Time Availability Feed) | Triggers (outbound) | Status transitions fire this automation's confidence recompute |
| FEAT-04.SPEC-007 (Calendar Reconnection Alert) | Triggers (outbound) | A Needs Reconnection transition fires this notification |
| FEAT-04.SPEC-002 (Calendar Connection Status & Management) | Affects (outbound) | Status transitions surface on the connection cards there |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | Status changes feed the dashboard's attention list and degraded-confidence display |
| FEAT-04.SPEC-008 (Calendar Connection Rules) | References (inbound) | Disconnect-wins-over-in-flight-sync contention rule governs in-flight reconciliation |

## Analytics and Success Signals

- **calendar_connection_lapsed** (calendar_kind, detection_method: proactive_report / scheduled_check) -- supports success-metrics.md: "Calendar Sync Reliability"
- **calendar_reconciliation_completed** (calendar_kind, drift_found: yes / no) -- supports success-metrics.md: "Calendar Sync Reliability"
- **calendar_reconciliation_failed** (calendar_kind) -- supports success-metrics.md: "Calendar Sync Reliability"
- **double_booking_prevented_by_reduced_confidence** (calendar_kind) -- supports success-metrics.md: "Zero Double-Booking Confidence"

## Acceptance Criteria

**FEAT-04.SPEC-006-AC-01:** Given Talia's Google calendar permission is revoked externally, when FEAT-04.SPEC-003 reports the revocation, then this automation sets the connection's status to Needs Reconnection and fires FEAT-04.SPEC-007's reconnect alert.

**FEAT-04.SPEC-006-AC-02:** Given a connection's permission has silently expired without a proactive report, when the scheduled health check runs, then it detects the invalid permission and sets status to Needs Reconnection.

**FEAT-04.SPEC-006-AC-03:** Given a connection is Needs Reconnection, when Talia completes reconnection via FEAT-04.SPEC-002, then status is set to Syncing while reconciliation runs.

**FEAT-04.SPEC-006-AC-04:** Given reconciliation runs after a reconnection, when it finds a booking cancelled during the outage whose calendar event was never removed, then this automation re-issues the removal instruction to close that gap.

**FEAT-04.SPEC-006-AC-05:** Given reconciliation completes with no drift found, when it finishes, then status is set to Connected and Talia sees no explicit "drift corrected" message, only the normal Connected status.

**FEAT-04.SPEC-006-AC-06:** Given reconciliation completes with drift corrected, when it finishes, then status is set to Connected the same way as the no-drift case -- the correction itself is invisible to Talia beyond the booking data now being accurate.

**FEAT-04.SPEC-006-AC-07:** Given a booking write fails persistently but the connection's permission is still valid, when this automation checks the connection, then status is not changed, since the failure is transient rather than a lapsed connection.

**FEAT-04.SPEC-006-AC-08:** Given Talia's connection lapses, when the availability signal is recomputed, then FEAT-03's slot computation narrows confidence visibly to Talia only, never to any client, per XBR-13.

**FEAT-04.SPEC-006-AC-09:** Given a connection's permission is revoked and re-granted moments later before the scheduled check runs, when FEAT-04.SPEC-003's revocation report is processed, then status still transitions to Needs Reconnection and the Pro completes reconnection through the normal flow.

**FEAT-04.SPEC-006-AC-10:** Given reconnection succeeds but the reconciliation pull itself fails, when this occurs, then status remains Syncing (not Connected) until reconciliation completes successfully, and the pull is retried automatically.

**FEAT-04.SPEC-006-AC-11:** Given a second health signal arrives for a connection already mid-reconciliation, when it is received, then it is queued and re-evaluated after the in-flight reconciliation completes, rather than starting a second overlapping reconciliation.

**FEAT-04.SPEC-006-AC-12:** Given Talia disconnects a connection while its reconciliation is in flight, when the disconnect completes, then the in-flight reconciliation is discarded per FEAT-04.SPEC-008's contention rule.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 4 | 4 |
| Outcome Paths | 6 | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |
