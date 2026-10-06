---
document_type: feature-overview
feature_number: FEAT-04
feature_name: Two-Way Calendar Sync
feature_slug: two-way-calendar-sync
priority_tier: Core
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 8
screen_count: 2
automation_count: 3
logic_rule_count: 1
integration_count: 1
notification_count: 1
---

# Feature Breakdown Brief: Two-Way Calendar Sync

## Summary

**Feature:** Two-Way Calendar Sync
**ID:** FEAT-04
**Description:** The Pro connects their personal Google or Apple calendar. Busy time there blocks Chairtime availability, and confirmed Chairtime bookings appear on that personal calendar automatically.
**Priority:** Core
**Phase:** MVP
**Type:** Platform
**Rationale:** BRIEF.md's Ecosystem & Integrations states this is two-way and "both matter" for Google and Apple. A pro who lives partly off-platform (personal appointments, a second job) cannot trust the availability engine without it, directly serving the "never silently double-book" success criterion. Competitor mobile apps are frequently criticized for laggy calendar sync, reinforcing why this feature's near-immediate sync target matters for trust.

**Key Capabilities:**
- Connect a Google or Apple calendar
- See connection health (connected / needs reconnection)
- Disconnect a calendar at any time

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-04.SPEC-001 | Calendar Connection Setup | Screen | The Pro | Pro chooses Google or Apple and authorizes Chairtime to connect to their personal calendar |
| FEAT-04.SPEC-002 | Calendar Connection Status & Management | Screen | The Pro, Platform Operator (Support) | Pro views each connection's health, reconnects a lapsed connection, or disconnects a calendar at any time |
| FEAT-04.SPEC-003 | Calendar Provider Sync | Integration | The Pro | External calendar-sync capability: performs the account-linking handshake, pulls busy/free time, and writes, moves, or removes Chairtime bookings on the Pro's connected calendar |
| FEAT-04.SPEC-004 | Busy-Time Availability Feed | Automation | The Pro | Turns synced busy/free periods into the blocked-availability signal the slot engine consumes |
| FEAT-04.SPEC-005 | Booking-to-Calendar Sync | Automation | The Pro | On any Chairtime booking create, reschedule, or cancel, writes/moves/removes the matching event on the Pro's personal calendar |
| FEAT-04.SPEC-006 | Sync Health Monitor & Reconciliation | Automation | The Pro, Platform Operator (Support) | Watches connection validity, degrades confidence on failure, and reconciles drift once a lapsed connection is restored |
| FEAT-04.SPEC-007 | Calendar Reconnection Alert | Notification | The Pro | Dashboard banner telling the Pro a connection needs reconnecting |
| FEAT-04.SPEC-008 | Calendar Connection Rules | Logic/Rule | The Pro | Enforces the one-connection-per-kind limit and the contention/precedence rules governing connect, disconnect, and in-flight sync |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Connect a Google or Apple calendar | FEAT-04.SPEC-001, FEAT-04.SPEC-003 | Setup screen collects the choice of kind; the Integration spec performs the actual account-linking handshake | Phase 2 (Explicit) |
| See connection health (connected / needs reconnection) | FEAT-04.SPEC-002, FEAT-04.SPEC-006 | Status screen displays current health; the health-monitor automation is what keeps that status accurate | Phase 2 (Explicit) |
| Disconnect a calendar at any time | FEAT-04.SPEC-002 | Disconnect action on the status screen, governed by the contention rule in SPEC-008 | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-04.SPEC-004 | Busy-Time Availability Feed | Phase 4 (Trigger-Response / External Dependencies lens) | Inbound busy/free data from the calendar provider has no effect on its own; something must translate it into the blocked-availability signal FEAT-03 depends on (XBR-13) |
| FEAT-04.SPEC-005 | Booking-to-Calendar Sync | Phase 4 (Trigger-Response) | The feature description states bookings "appear on that personal calendar automatically," and the audit-added alternate flow requires cancels/reschedules to mirror too (XBR-13); this crosses four other features' booking-lifecycle events and needed its own Automation |
| FEAT-04.SPEC-006 | Sync Health Monitor & Reconciliation | Phase 4 (Trigger-Response) + Phase 6 (Negative/Failure) | The alternate flow ("connection lapses... availability engine visibly narrows its confidence") and the Contention resolution note ("bookings already written... are reconciled on reconnect") both describe system behavior nobody names as a screen or a user action |
| FEAT-04.SPEC-007 | Calendar Reconnection Alert | Phase 4 (Notification surfacing) | The Communications field names a specific channel (dashboard alert, not text/email) and audience (Pro only) -- this crosses the inline-vs-standalone threshold |
| FEAT-04.SPEC-008 | Calendar Connection Rules | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field's one-per-kind cap and the dependency map's Contention resolution (disconnect wins over in-flight sync; last-write-wins health updates; reconcile on reconnect) are conditional rules shared across the Setup screen, the Status screen, and the Health Monitor |

## Entity-Lifecycle Coverage Matrix

**Entity: Calendar Connection**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-04.SPEC-001 / FEAT-04.SPEC-003 | Pro picks a kind on the Setup screen; the Integration spec performs the handshake and creates the connection record | At most one per kind (FEAT-04.SPEC-008); offered again in FEAT-15 onboarding |
| Read (single) | FEAT-04.SPEC-002 | Status screen shows one connection's health and last sync time | Also readable by Support (View-only) for troubleshooting |
| Read (list) | FEAT-04.SPEC-002 | Status screen lists up to two connections (one Google, one Apple) side by side | A Pro with no connections sees the Empty state described in product-features.md's States field |
| Update | FEAT-04.SPEC-003, FEAT-04.SPEC-006 | busy_periods refreshed by SPEC-003 on each busy-time pull and by SPEC-006 on post-reconnect reconciliation; last_successful_sync set by SPEC-003 on every successful sync direction (busy-time pull and booking write/move/remove confirmation) and by SPEC-006 on successful reconciliation; status transitions owned by SPEC-006. SPEC-004 and SPEC-005 only read the record and never write it | SPEC-004 consumes busy_periods and SPEC-005 instructs SPEC-003; neither updates the Calendar Connection record |
| Delete/Archive | FEAT-04.SPEC-002 | Hard delete: disconnecting removes the connection record outright, with no restore/undo path -- reconnecting later creates a fresh connection rather than reviving the old one. No cascade: existing Chairtime bookings and their history are untouched (Booking's own lifecycle is independent, per SC-22/Booking entity notes). No retention/purge policy applies because the record holds only current status and busy/free data, never an event history that would need purging. | Pro disconnect always wins over an in-flight sync per the Contention rule (FEAT-04.SPEC-008) |
| State Transition | FEAT-04.SPEC-006 | Connected -> Syncing -> Needs Reconnection -> Connected (on reconnect) transitions, or -> Disconnected (via the Delete operation above) | Needs Reconnection also degrades the confidence FEAT-03 shows the Pro, never the client (XBR-13) |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Booking | FEAT-04.SPEC-005 | Reads confirmed, rescheduled, cancelled, and no-show bookings (from FEAT-05, FEAT-10, FEAT-11, FEAT-30) to decide what to write, move, or remove on the external calendar; a booking marked no-show needs no calendar action since the appointment already occurred, but is read to confirm no action is required |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Pro selects "Connect Google/Apple calendar" and authorizes | Perform the external account-linking handshake | Standalone Integration | FEAT-04.SPEC-003 |
| Account-linking handshake succeeds | Create the connection record, set status Syncing, then Connected once the first busy-time pull completes | Inline in triggering screen, then handed to automations | FEAT-04.SPEC-001 / FEAT-04.SPEC-004 / FEAT-04.SPEC-006 |
| Calendar provider reports a new or changed busy period | Recompute the blocked-availability signal fed to the slot engine | Standalone Automation | FEAT-04.SPEC-004 |
| Chairtime booking is confirmed (FEAT-05, FEAT-21, FEAT-30) | Write a new event to the Pro's connected calendar | Standalone Automation | FEAT-04.SPEC-005 |
| Chairtime booking is rescheduled (FEAT-10, FEAT-11, FEAT-30) | Move the matching calendar event to the new time | Standalone Automation | FEAT-04.SPEC-005 |
| Chairtime booking is cancelled (FEAT-10, FEAT-11, FEAT-30) | Remove the matching calendar event so no ghost appointment remains | Standalone Automation | FEAT-04.SPEC-005 |
| Scheduled connection-health check runs | Detect a revoked or expired permission; set status to Needs Reconnection; narrow the availability engine's confidence (Pro-visible only) | Standalone Automation | FEAT-04.SPEC-006 |
| Connection status becomes Needs Reconnection | Show a dashboard banner alert to the Pro | Standalone Notification | FEAT-04.SPEC-007 |
| Pro completes a reconnection | Reconcile busy time and previously-written bookings against the calendar to resolve any drift from the outage, then restore Connected status | Standalone Automation | FEAT-04.SPEC-006 |
| Pro disconnects a calendar | Stop future sync in both directions; leave existing Chairtime bookings untouched; remove the connection record (see Delete/Archive above) | Inline in triggering screen | FEAT-04.SPEC-002 |
| Pro attempts to connect a second calendar of the same kind | Reject the attempt; only one connection per kind is allowed at a time | Standalone Logic/Rule | FEAT-04.SPEC-008 |
| A Pro disconnect happens while a sync is in flight | Disconnect wins outright; the in-flight sync is discarded | Standalone Logic/Rule | FEAT-04.SPEC-008 |

## Shared Context

**Shared Entities:**
- Calendar Connection -- created by SPEC-001/SPEC-003, read/listed by SPEC-002 (and by Support, view-only), updated by SPEC-003 (busy_periods, last_successful_sync) and SPEC-006 (status, busy_periods, last_successful_sync on reconciliation), read by SPEC-004/SPEC-005, deleted by SPEC-002. Fields: calendar_kind, status, last_successful_sync, busy_periods.

**Shared UI Patterns:**
- Connection status vocabulary (Connected / Syncing / Needs Reconnection / Disconnected) -- used identically by SPEC-001 (the moment right after connecting) and SPEC-002 (ongoing display). Spec Writers for both screens must use the same four labels and the same visual treatment so a Pro recognizes the state regardless of which screen shows it.
- Empty-state messaging -- SPEC-002's no-connection state uses the same plain, non-technical explanation of "what connecting does and why" that SPEC-001 leads with (product-features.md States field).

**Shared Validation:**
- FEAT-04.SPEC-008 owns the one-per-kind connection limit and the connect/disconnect/sync contention rules. SPEC-001 defers to it before allowing a new connect; SPEC-002's disconnect action and SPEC-006's status updates both defer to it for precedence when two changes land at once.

## Internal Dependency Map

```
SPEC-001 (Calendar Connection Setup) -> [Pro picks a kind and authorizes] -> SPEC-003 (Calendar Provider Sync)
SPEC-001 (Calendar Connection Setup) -> [validates against] -> SPEC-008 (Calendar Connection Rules)
SPEC-003 (Calendar Provider Sync) -> [handshake completes] -> SPEC-002 (Calendar Connection Status & Management)
SPEC-003 (Calendar Provider Sync) -> [inbound busy/free data] -> SPEC-004 (Busy-Time Availability Feed)
SPEC-005 (Booking-to-Calendar Sync) -> [uses] -> SPEC-003 (Calendar Provider Sync) to write/move/remove events
SPEC-006 (Sync Health Monitor & Reconciliation) -> [detects failure] -> SPEC-007 (Calendar Reconnection Alert)
SPEC-007 (Calendar Reconnection Alert) -> [Pro taps reconnect] -> SPEC-002 (Calendar Connection Status & Management)
SPEC-002 (Calendar Connection Status & Management) -> [Pro re-authorizes] -> SPEC-003 (Calendar Provider Sync) -> [success] -> SPEC-006 (Sync Health Monitor & Reconciliation)
SPEC-002 (Calendar Connection Status & Management) -> [Pro disconnects] -> SPEC-008 (Calendar Connection Rules) applies the contention rule
```

**Default Entry:** SPEC-002 (Calendar Connection Status & Management) -- the screen shown when the Pro navigates to this feature area; it shows the Empty state (offering to connect) when nothing is connected yet, and routes to SPEC-001 for a first connection.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-04.SPEC-004 | Outbound | FEAT-03 (Real-Time Slot Availability Engine) | Feeds current busy/blocked periods into the live slot check (XBR-01, XBR-13) | New or changed busy time is synced from the calendar provider |
| FEAT-04.SPEC-005 | Inbound | FEAT-05 (Public Booking Page & Booking Flow) | Reads a newly confirmed booking to write it to the Pro's calendar | Booking is confirmed |
| FEAT-04.SPEC-005 | Inbound | FEAT-10 (Client-Initiated Cancel/Reschedule) | Reads a client-initiated cancel/reschedule to remove or move the calendar event | Client cancels or reschedules |
| FEAT-04.SPEC-005 | Inbound | FEAT-11 (No-Show Marking & Deposit Forfeiture) | Reads a booking marked no-show; confirms no calendar action is needed since the appointment already occurred | Booking marked no-show |
| FEAT-04.SPEC-005 | Inbound | FEAT-30 (Pro Booking Management) | Reads a Pro-initiated cancel, reschedule, or direct booking to write/move/remove the calendar event | Pro cancels, reschedules, or books a client in directly |
| FEAT-04.SPEC-002 | Inbound | FEAT-15 (Pro Onboarding & Setup Wizard) | Calendar connection is offered as a skippable setup step | Pro reaches the calendar step during onboarding |
| FEAT-04.SPEC-006 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | Connection health feeds the dashboard's attention list and its degraded-confidence display | Health status changes |
| FEAT-04.SPEC-007 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | Reconnect banner surfaces on the dashboard's attention list, which the Pro can tap straight into SPEC-002 | Connection status becomes Needs Reconnection |

## Non-Functional Notes

**Data volumes / growth:** Each Pro holds at most two Calendar Connection records (one Google, one Apple); the busy/free footprint stays small by design since only minimal busy/free periods are captured, never full event details (product-features.md Data Notes).

**Responsiveness:** Sync latency target is near-immediate in both directions -- a new busy period and a new Chairtime booking should each reflect within a couple of minutes (product-features.md Validation & Limits; success-metrics.md Calendar Sync Reliability). Correctness is expressed as a bar, not an uptime percentage: a sync failure must surface as a visible dashboard banner rather than a silent gap, and a lapsed connection must never be allowed to cause a double-booking (ASMP-26). When connectivity is lost, the most recently synced busy times remain in effect until it returns, and the availability engine's confidence narrows visibly to the Pro only -- never silently, and never shown to clients (product-features.md States field; ASMP-27).

**Data sensitivity / privacy:** Sensitive -- the connection grants access to the Pro's personal calendar; only busy/free periods are kept, never event titles or details (product-features.md Data Notes). Support's access is limited to connection health for troubleshooting and never extends to calendar content (Access field), consistent with the account-protection posture applied to the rest of the Pro Account (ASMP-30). Every connection and status screen must remain fully usable at phone width, with a screen reader, and without relying on color alone for the health indicator (ASMP-28).

**Compliance flags:** N/A -- assumptions-constraints.md's Non-Functional Expectations name no health or financial compliance regime for this feature; calendar busy/free data receives the product's standard personal-data handling, the same as every other Pro Account record.

## Non-Goals

- **Multi-staff or multi-chair calendar routing** -- Excluded per scope-boundaries.md (SC-01): the product is strictly single-operator, so there is no need to route calendar connections or busy time across multiple staff members or chairs.
- **Cross-pro or cross-client visibility of calendar data** -- Excluded per scope-boundaries.md (SC-03): connection health and busy/free data never surface across pro accounts or to any client; the Client's Access field is "None."
- **Full event detail capture (titles, descriptions, attendees, locations)** -- Excluded per product-features.md's Data Notes, which state explicitly that only "the minimum busy/free time data needed to block slots" is captured, "not full event details."
- **Retention or purge policy for a disconnected connection record** -- Intentional lifecycle decision surfaced by the CRUD matrix: disconnect is a hard delete with no restore path, so there is no historical connection record to retain or purge; reconnecting always creates a fresh connection rather than reviving one.
- **Instagram-based calendar integration or reminders** -- Excluded per scope-boundaries.md (SC-06): Chairtime has no Instagram integration beyond serving as the destination for the bio link, so calendar connection and alerts live only inside Chairtime's own dashboard.
