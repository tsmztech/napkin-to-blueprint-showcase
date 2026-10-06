# FEAT-04 — Two-Way Calendar Sync

This chapter covers Two-Way Calendar Sync (FEAT-04), a Core-tier feature. It carries 8 specifications carrying 98 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-04.SPEC-001 | Calendar Connection Setup | screen | 11 |
| FEAT-04.SPEC-002 | Calendar Connection Status & Management | screen | 13 |
| FEAT-04.SPEC-003 | Calendar Provider Sync | integration | 15 |
| FEAT-04.SPEC-004 | Busy-Time Availability Feed | automation | 10 |
| FEAT-04.SPEC-005 | Booking-to-Calendar Sync | automation | 13 |
| FEAT-04.SPEC-006 | Sync Health Monitor & Reconciliation | automation | 12 |
| FEAT-04.SPEC-007 | Calendar Reconnection Alert | notification | 9 |
| FEAT-04.SPEC-008 | Calendar Connection Rules | logic-rule | 15 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Calendar Connection Setup

## Overview

**Name:** Calendar Connection Setup
**ID:** FEAT-04.SPEC-001
**Type:** Screen
**Purpose:** The Pro chooses Google or Apple as their personal calendar kind and authorizes Chairtime to connect to it, kicking off the account-linking handshake.
**Parent Feature:** FEAT-04 -- Two-Way Calendar Sync

## Scope and Non-Goals

**In Scope:**
- Presenting the choice of calendar kind (Google or Apple) and an explanation of what connecting does
- Initiating the account-linking handshake for the chosen kind (delegated to FEAT-04.SPEC-003)
- Showing the in-progress and just-connected states immediately following authorization
- Deferring to the one-per-kind connection limit (FEAT-04.SPEC-008) before offering a kind that is already connected

**Non-Goals:**
- Performing the actual account-linking handshake with the calendar provider -- owned by FEAT-04.SPEC-003 (Calendar Provider Sync); this screen only initiates it and shows its outcome
- Ongoing connection health display, reconnection, and disconnection -- owned by FEAT-04.SPEC-002 (Calendar Connection Status & Management); this screen is reached only for a first connection of a given kind
- Deciding whether calendar connection is required before the booking link can go live -- excluded per product-features.md's Validation & Limits and XBR-26, which state calendar connection is the only optional go-live step; this screen never blocks go-live
- Displaying or editing any busy/free time data -- excluded per product-features.md's Data Notes: only status, never calendar content, is ever shown to the Pro

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15.SPEC-001 (Setup Wizard Shell) (Pro Onboarding & Setup Wizard, calendar step) | Pro reaches the calendar step during onboarding (skippable) | None -- screen starts empty; a skip here returns the Pro to onboarding with no connection created |
| FEAT-04.SPEC-002 (Calendar Connection Status & Management) | Pro taps "Connect Google calendar" or "Connect Apple calendar" from the empty state or from the unconnected kind's row | The calendar kind the Pro selected is pre-filled; the screen skips the kind-choice step if only one kind remains unconnected |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Choose a calendar kind and authorize the connection | -- |
| The Client (Riley) | No | No | This screen is never reached by a Client; no navigation path from any Client-facing spec leads here |
| Platform Operator (Support) | No | No | Support's View access to Calendar Connection (Access Matrix: Service & Availability Setup = View) covers connection health on FEAT-04.SPEC-002 only; Support never initiates a new connection, so this screen is not exposed to Support at all |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); after signing in, the user lands on the Pro Daily Schedule Dashboard (FEAT-12), not back on this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no connection attempt was in progress server-side (authorization only begins once the handshake is initiated), so nothing is lost by re-authenticating |

## Layout and Content

**Header:** Screen title "Connect your calendar" with a back arrow (returns to FEAT-04.SPEC-002, or to the onboarding wizard's calendar step if reached from FEAT-15).

**Body:** A single-column layout with, in order:
- A short plain-language explanation of what connecting does and why: personal busy time will block Chairtime availability, and confirmed Chairtime bookings will appear on the Pro's personal calendar automatically. No technical detail is included.
- Two selectable options, "Google Calendar" and "Apple Calendar," each shown as a tappable card with the provider's name. A kind already connected (per FEAT-04.SPEC-008's one-per-kind limit) is shown but disabled, labeled "Already connected" with a link to FEAT-04.SPEC-002 instead of an authorize action.
- Once a kind is selected and the Pro taps "Connect," the body replaces the two option cards with a single "Connecting to {kind}..." in-progress indicator while the handshake (FEAT-04.SPEC-003) runs.
- On handshake success, the body shows a confirmation summary: the connected kind, the Connected/Syncing status label, and a "Done" action.
- On handshake failure, the body shows the two option cards again with an inline error above them.

**Footer:** "Skip for now" text link, visible only when this screen was reached from FEAT-15 onboarding; absent when reached from FEAT-04.SPEC-002 (there is nothing to skip once the Pro is already managing an existing setup).

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described above, full width; option cards stack vertically.
- **Medium size class and above:** Layout remains single-column, capped at a consistent platform-wide form width and horizontally centered; option cards remain stacked (never shown side by side, to keep the choice unambiguous).

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-04.SPEC-002, or to the FEAT-15 onboarding calendar step if reached from there | Screen closes | Animated transition back |
| "Google Calendar" card | Tap (kind not yet connected) | Selects Google as the chosen kind; "Connect" action becomes available | Card shows selected state | Card highlights as selected |
| "Apple Calendar" card | Tap (kind not yet connected) | Selects Apple as the chosen kind; "Connect" action becomes available | Card shows selected state | Card highlights as selected |
| Already-connected kind card | Tap | No connection action offered; taps navigate to FEAT-04.SPEC-002 instead | None on this screen | Navigates to FEAT-04.SPEC-002 |
| "Connect" button | Tap | 1. Validates the one-per-kind limit via FEAT-04.SPEC-008. 2. Initiates the account-linking handshake via FEAT-04.SPEC-003 for the chosen kind. | Body switches to the in-progress "Connecting to {kind}..." state | In-progress indicator shown; other controls disabled |
| "Done" button (post-success) | Tap | Navigate to FEAT-04.SPEC-002 | Screen closes | Animated transition to the status screen showing the new connection |
| "Skip for now" link (onboarding entry only) | Tap | Returns to the FEAT-15 onboarding flow's next step without creating a connection | Screen closes | Onboarding continues at the next step |
| Retry (on handshake failure) | Tap | Re-initiates the handshake via FEAT-04.SPEC-003 for the same chosen kind | Body returns to in-progress state | In-progress indicator shown again |

### Accessibility Notes

- **Focus order:** Back arrow -> explanation text -> Google Calendar card -> Apple Calendar card -> Connect button -> (post-entry) Skip for now link.
- **Dynamic announcements:** The transition into the in-progress state is announced to assistive technology ("Connecting to {kind}"); the transition to success or error is announced as it replaces the in-progress content.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures. Option cards behave as a single-select control group.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Choosing (default) | Both option cards shown, unselected; Connect button disabled until a kind is chosen | Screen first opens | Pro selects a kind |
| Selected | Chosen kind's card shows selected state; Connect button enabled | Pro taps an unconnected kind's card | Pro taps Connect, or selects the other kind |
| Connecting | In-progress indicator "Connecting to {kind}...", option cards hidden | Pro taps Connect | Handshake (FEAT-04.SPEC-003) reports success or failure |
| Connected | Confirmation summary with kind, Connected/Syncing status label, and Done action | Handshake succeeds | Pro taps Done |
| Error | Option cards shown again with inline error message above them: "We couldn't connect to {kind}. Try again." and a Retry action | Handshake fails | Pro taps Retry, or selects a different kind and starts over |
| Offline/Degraded | Connect button is disabled with the message "Connecting a calendar needs an internet connection. Check your connection and try again." -- the explanation text and option cards remain visible and readable | Connectivity is lost while this screen is open, or the screen is opened without connectivity | Connectivity is restored -- the screen returns to its prior state (Choosing or Selected) with no data lost |

## Validation Rules

Validation governed by FEAT-04.SPEC-008 (Calendar Connection Rules). See that spec for the one-per-kind connection limit this screen defers to before offering a kind for connection.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-04.SPEC-002 (Calendar Connection Status & Management) | -- |
| Back arrow tap (onboarding entry) | FEAT-15 onboarding, calendar step | FEAT-15 (Pro Onboarding & Setup Wizard) |
| Successful connection, "Done" tap | FEAT-04.SPEC-002 (Calendar Connection Status & Management) | -- |
| "Skip for now" tap (onboarding entry) | FEAT-15 onboarding, next step | FEAT-15 (Pro Onboarding & Setup Wizard) |

## Data Model

**Creates:** Calendar Connection -- created by FEAT-04.SPEC-003 once the handshake succeeds, with calendar_kind set to the Pro's chosen kind and status set to Syncing; this screen initiates the creation but does not write the record itself.
**Reads:** Calendar Connection -- reads existing connections (calendar_kind, status) only to determine which kinds are already connected, so the corresponding option card can be disabled per FEAT-04.SPEC-008.
**Updates:** None.
**Deletes:** None.

## Business Rules

- FEAT-04.SPEC-008 (Calendar Connection Rules) governs the one-per-kind limit -- a kind already connected is never offered for a new connection on this screen.
- FEAT-04.SPEC-003 (Calendar Provider Sync) owns the actual handshake; this screen only initiates it and reflects its outcome.
- XBR-26: calendar connection is the only optional go-live step -- skipping this screen during onboarding (FEAT-15) never blocks the booking link from going live.
- FEAT-15.SPEC-006 (Setup Step Order & Optional-Step Rules) governs the skip choice: this screen offers "Skip for now" only for the onboarding entry and only while step_calendar is incomplete, and connecting after a skip converts step_calendar from complete-as-skipped to complete-as-connected as that rule allows.

## Edge Cases

- **Pro backgrounds the app mid-handshake** -- The handshake continues server-side; on return, the screen reflects whichever outcome (Connected or Error) the handshake reached in the meantime, rather than resuming the in-progress indicator indefinitely.
- **Pro taps Connect twice rapidly** -- The second tap is ignored while the first handshake is in flight (Connect button disabled during Connecting).
- **Both kinds already connected** -- This screen is not reached in this state; FEAT-04.SPEC-002 offers no "add a connection" action once both kinds are connected, per FEAT-04.SPEC-008's one-per-kind limit applied to both supported kinds.
- **Pro selects a kind then navigates away without connecting** -- No connection record exists; nothing is created until the handshake succeeds. Re-entering this screen starts from the Choosing state.
- **A second connection of the same kind is attempted while a connection of that kind already exists** -- No concurrent-edit conflict applies here: this is a creation screen with no existing record loaded for the chosen kind, and FEAT-04.SPEC-008's limit check simply prevents the attempt by disabling that kind's card; there is no shared-entity write to reconcile.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-003 (Calendar Provider Sync) | Triggers (outbound) | "Connect" initiates the account-linking handshake for the chosen kind |
| FEAT-04.SPEC-008 (Calendar Connection Rules) | References (inbound) | One-per-kind connection limit gates which kinds this screen offers |
| FEAT-04.SPEC-002 (Calendar Connection Status & Management) | Navigation (inbound/outbound) | Reached from the empty state or an unconnected kind's row there; returns there on Done or Back |
| FEAT-15.SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance), FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) -- within FEAT-15 (Pro Onboarding & Setup Wizard) | Navigation (inbound/outbound) | Reached as the skippable calendar setup step; Back or Skip returns to the wizard |
| FEAT-15.SPEC-006 (Setup Step Order & Optional-Step Rules) | References (inbound) | Rule governing the one optional step: the skip choice appears only on this screen, and step_calendar moves to complete-as-skipped or complete-as-connected per that rule |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| calendar_connect_started | calendar_kind, entry_source (onboarding / status_screen) | Pro taps Connect | supports success-metrics.md: "Calendar Sync Reliability" |
| calendar_connected | calendar_kind, entry_source | Handshake (FEAT-04.SPEC-003) reports success | supports success-metrics.md: "Calendar Sync Reliability" |
| calendar_connect_failed | calendar_kind, entry_source | Handshake reports failure | supports success-metrics.md: "Calendar Sync Reliability" |
| calendar_connect_skipped | entry_source | Pro taps "Skip for now" (onboarding entry only) | supports success-metrics.md: "Setup-to-Live-Link Completion" |

## Acceptance Criteria

**FEAT-04.SPEC-001-AC-01:** Given Talia is on the Calendar Connection Setup screen with neither kind connected, when she selects "Google Calendar" and taps Connect, then the screen shows "Connecting to Google Calendar..." and initiates the handshake via FEAT-04.SPEC-003.

**FEAT-04.SPEC-001-AC-02:** Given Talia's handshake completes successfully, when the screen updates, then it shows the connected kind with a Syncing status label and a Done action.

**FEAT-04.SPEC-001-AC-03:** Given Talia taps Done after a successful connection, when the screen closes, then she lands on FEAT-04.SPEC-002 showing the new connection.

**FEAT-04.SPEC-001-AC-04:** Given Talia's Google calendar is already connected, when she opens this screen, then the Google Calendar card is shown disabled and labeled "Already connected," and only the Apple Calendar card is selectable.

**FEAT-04.SPEC-001-AC-05:** Given Talia selects Apple Calendar and taps Connect, when the handshake fails, then the screen shows "We couldn't connect to Apple Calendar. Try again." with a Retry action, and no connection record exists.

**FEAT-04.SPEC-001-AC-06:** Given Talia reaches this screen from the FEAT-15 onboarding wizard, when she taps "Skip for now," then she returns to the wizard's next step and no connection record is created.

**FEAT-04.SPEC-001-AC-07:** Given Talia reaches this screen from FEAT-04.SPEC-002 (not onboarding), when she looks at the footer, then no "Skip for now" link is shown.

**FEAT-04.SPEC-001-AC-08:** Given Talia has selected a kind but not yet tapped Connect, when she taps the back arrow, then the screen closes with no connection attempted and no data retained.

**FEAT-04.SPEC-001-AC-09:** Given Talia loses connectivity while on this screen, when she looks at the Connect button, then it is disabled with "Connecting a calendar needs an internet connection. Check your connection and try again."

**FEAT-04.SPEC-001-AC-10:** Given Talia taps Connect and then backgrounds the app before the handshake resolves, when she returns to the screen, then it reflects whichever outcome (Connected or Error) the handshake reached, not an indefinite in-progress state.

**FEAT-04.SPEC-001-AC-11:** Given Talia taps Connect, when she taps Connect again before the first handshake resolves, then the second tap has no effect and the screen remains in the Connecting state for the original request.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 6 (choosing, selected, connecting, connected, error, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Screen Spec: Calendar Connection Status & Management

## Overview

**Name:** Calendar Connection Status & Management
**ID:** FEAT-04.SPEC-002
**Type:** Screen
**Purpose:** The Pro views the health of each connected calendar, reconnects a lapsed connection, or disconnects a calendar at any time; this is the default entry point for the calendar feature area.
**Parent Feature:** FEAT-04 -- Two-Way Calendar Sync

## Scope and Non-Goals

**In Scope:**
- Listing up to two connections (one Google, one Apple) with their current health status and last successful sync time
- Reconnecting a connection in Needs Reconnection status
- Disconnecting a connection at any time
- Offering a first connection when none exists (Empty state)
- Support's read-only view of connection health for troubleshooting

**Non-Goals:**
- Performing the account-linking handshake itself -- owned by FEAT-04.SPEC-003 (Calendar Provider Sync); this screen initiates reconnection but the handshake runs there
- Choosing a calendar kind for a first-time connection -- owned by FEAT-04.SPEC-001 (Calendar Connection Setup); this screen routes there
- Displaying calendar event content (titles, descriptions, attendees) -- excluded per product-features.md's Data Notes: only status and the minimum busy/free data are ever held, never full event details
- Automatically reconnecting on the Pro's behalf without their action -- excluded per product-features.md's Access field: reconnection is a deliberate Pro action, consistent with the account-protection posture (ASMP-30) that governs access to a Pro's connected accounts

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12 (Pro Daily Schedule Dashboard), navigation | Pro opens the calendar feature area from the dashboard's navigation | None |
| FEAT-12 (Pro Daily Schedule Dashboard), attention list | Pro taps the reconnect item on the dashboard's attention list | Focus lands on the specific connection needing reconnection |
| FEAT-04.SPEC-007 (Calendar Reconnection Alert) | Pro taps the reconnect banner | Focus lands on the specific connection needing reconnection |
| FEAT-04.SPEC-001 (Calendar Connection Setup) | Pro completes a new connection and taps Done | The newly connected kind is shown at the top of the list |
| Direct navigation (default entry) | Pro opens the calendar feature area with no prior context | None -- shows the Empty state if no connections exist |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- both connections, health, last sync time | Reconnect and disconnect either connection | -- |
| The Client (Riley) | No | No | This screen is never reached by a Client; no navigation path from any Client-facing spec leads here |
| Platform Operator (Support) | Connection health and last sync time only, for the Pro account whose help request is open (Access Matrix: Service & Availability Setup = View) | No actions -- reconnect and disconnect controls are not shown to Support | Reconnect and disconnect controls are hidden entirely; Support sees a read-only variant of this screen |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); after signing in, the user lands on the Pro Daily Schedule Dashboard (FEAT-12), not back on this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no unsaved action existed on this screen (reconnect/disconnect are immediate actions, not drafts), so nothing is lost |

## Layout and Content

**Header:** Screen title "Calendar" with a back arrow (returns to the dashboard, FEAT-12).

**Body (connections exist):** A list of up to two connection cards, one per connected kind, each showing:
- The calendar kind name (Google Calendar / Apple Calendar)
- A status label using the shared vocabulary (Connected, Syncing, Needs Reconnection) with a visual treatment that never relies on color alone (the word itself is always shown)
- Last successful sync time, or "Not yet synced" if a first sync has not completed
- A "Reconnect" action, shown only on a card in Needs Reconnection status
- A "Disconnect" action, always shown
Below the connection cards, if fewer than two kinds are connected, an "Add another calendar" action routing to FEAT-04.SPEC-001 for the remaining kind.

**Body (no connections -- Empty state):** The same plain, non-technical explanation of what connecting does and why that FEAT-04.SPEC-001 leads with, followed by two option entries, "Connect Google Calendar" and "Connect Apple Calendar," each routing to FEAT-04.SPEC-001 with that kind pre-filled.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Connection cards stack vertically, full width.
- **Medium size class and above:** Layout remains single-column, capped at a consistent platform-wide list width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12 (Pro Daily Schedule Dashboard) | Screen closes | Animated transition back |
| "Reconnect" action (Needs Reconnection card) | Tap | Initiates the reconnection handshake via FEAT-04.SPEC-003 for that connection | Card shows in-progress "Reconnecting..." state | In-progress indicator on that card |
| "Disconnect" action | Tap | Shows a confirmation dialog per FEAT-04.SPEC-008's contention rule | Dialog appears | Dialog: "Disconnect {kind} Calendar? Chairtime will stop syncing with it. Your existing bookings won't change." with "Disconnect" and "Cancel" |
| Disconnect confirmation, "Disconnect" | Tap | Removes the connection record (hard delete, per the dependency map's Delete/Archive lifecycle for Calendar Connection); any sync in flight for that connection is discarded per FEAT-04.SPEC-008 | Connection card is removed from the list; "Add another calendar" reflects the newly available kind | Toast: "{kind} Calendar disconnected." (FEAT-04.SPEC-004 is then triggered to drop that connection from the blocked-availability signal) |
| Disconnect confirmation, "Cancel" | Tap | Dialog closes with no change | None | Dialog dismissed |
| "Add another calendar" (list state) | Tap | Navigate to FEAT-04.SPEC-001 with the remaining kind pre-filled | Screen closes | Animated transition to setup screen |
| "Connect Google Calendar" / "Connect Apple Calendar" (Empty state) | Tap | Navigate to FEAT-04.SPEC-001 with that kind pre-filled | Screen closes | Animated transition to setup screen |

### Accessibility Notes

- **Focus order:** Back arrow -> (per connection card, in list order) status label -> last sync time -> Reconnect (if shown) -> Disconnect -> Add another calendar (if shown), or -> Connect Google Calendar -> Connect Apple Calendar (Empty state).
- **Dynamic announcements:** A status change from Connected/Syncing to Needs Reconnection while the screen is open is announced to assistive technology, since this screen is live-updating (see Concurrency below). The disconnect confirmation toast is announced on completion.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | Plain explanation plus two "Connect" entries, no connection cards | No connections exist for this Pro Account | A connection is created via FEAT-04.SPEC-001 |
| List (healthy) | One or two connection cards, statuses Connected or Syncing | At least one connection exists | Status changes, or a connection is added/removed |
| List (needs attention) | A connection card shows Needs Reconnection with the Reconnect action visible | FEAT-04.SPEC-006 sets a connection's status to Needs Reconnection | Reconnection succeeds and status returns to Connected |
| Reconnecting | The affected card shows an in-progress "Reconnecting..." indicator | Pro taps Reconnect | Handshake (FEAT-04.SPEC-003) reports success or failure |
| Loading | N/A -- connection data is small (at most two records) and loads instantly with a lightweight in-place indicator on first paint only | Screen first opens | Data loads (near-instant) |
| Error | The list shows the most recently loaded connection data with a banner: "We couldn't refresh calendar status. Showing the last known information." and a Retry action | A status refresh fails to load | Retry succeeds, or the next automatic refresh succeeds |
| Offline/Degraded | The most recently loaded connection cards remain viewable read-only; Reconnect and Disconnect actions are disabled with the message "This action needs an internet connection." | Connectivity is lost while this screen is open | Connectivity is restored -- actions re-enable and the screen refreshes to current status |

## Validation Rules

Validation and contention behavior governed by FEAT-04.SPEC-008 (Calendar Connection Rules). See that spec for the one-per-kind limit (which caps the connections this screen ever lists at one per kind) and the disconnect-wins-over-in-flight-sync contention rule this screen's Disconnect action triggers.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-12 (Pro Daily Schedule Dashboard) | -- (cross-feature) |
| "Reconnect" tap | Stays on this screen; handshake runs via FEAT-04.SPEC-003 in place | -- |
| "Add another calendar" / Empty-state "Connect" tap | FEAT-04.SPEC-001 (Calendar Connection Setup) | -- |

## Data Model

**Creates:** None directly -- connections are created via FEAT-04.SPEC-001/FEAT-04.SPEC-003.
**Reads:** Calendar Connection -- calendar_kind, status, last_successful_sync for every connection belonging to this Pro Account (up to two).
**Updates:** None directly by this screen -- status transitions during reconnect are owned by FEAT-04.SPEC-003 (handshake outcome) and FEAT-04.SPEC-006 (ongoing health).
**Deletes:** Calendar Connection -- the Disconnect action performs the hard delete described in the dependency map's Delete/Archive lifecycle: no restore/undo path; existing Chairtime bookings and their history are untouched; reconnecting later creates a fresh connection record rather than reviving the old one.

## Business Rules

- FEAT-04.SPEC-008 (Calendar Connection Rules) governs the one-per-kind limit (this screen never shows more than one card per kind) and the disconnect-wins-over-in-flight-sync contention rule.
- Disconnecting never affects existing Chairtime bookings or their history, per the dependency map's Delete/Archive notes for Calendar Connection.
- Connection status vocabulary (Connected / Syncing / Needs Reconnection / Disconnected) is shared identically with FEAT-04.SPEC-001, per the Brief's Shared UI Patterns.
- This screen is live-updating, not a snapshot: a status change made by FEAT-04.SPEC-006 (for example, a connection lapsing) is reflected on this screen while it is open, without requiring the Pro to navigate away and back.

## Edge Cases

- **Pro disconnects a connection while a sync is in flight** -- Per FEAT-04.SPEC-008's contention rule, the disconnect wins outright; the in-flight sync is discarded and the connection record is removed immediately. No partial sync state is left behind.
- **Connection status changes to Needs Reconnection while the Pro is viewing this screen** -- The affected card's status label updates in place (this screen is live-updating) and the Reconnect action appears, without requiring a manual refresh.
- **Pro taps Reconnect twice rapidly** -- The second tap is ignored while the first reconnection handshake is in flight (the card's Reconnect action is replaced by the in-progress indicator).
- **Pro taps Disconnect on a connection that FEAT-04.SPEC-006 concurrently sets to Needs Reconnection** -- Disconnect proceeds regardless of the connection's current health status; a lapsed connection can be disconnected the same as a healthy one. Resolution: disconnect wins, consistent with FEAT-04.SPEC-008 -- the pending health-status update is superseded by the deletion.
- **Both connection kinds already exist** -- "Add another calendar" is not shown; the list shows exactly two cards with no further connection action offered, per the one-per-kind limit applied to both supported kinds.
- **Support views this screen for a Pro with no connections** -- Support sees the same Empty-state explanation text but no "Connect" actions, since Support never initiates a connection.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-001 (Calendar Connection Setup) | Navigation (outbound) | "Add another calendar" and Empty-state entries route to first-time setup |
| FEAT-04.SPEC-003 (Calendar Provider Sync) | Triggers (outbound) | Reconnect action initiates the account-linking handshake |
| FEAT-04.SPEC-004 (Busy-Time Availability Feed) | Triggers (outbound) | Confirmed Disconnect fires that automation's disconnect trigger, which removes the connection's busy_periods contribution from the blocked-availability signal |
| FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) | References (inbound) | Health status shown on this screen is kept current by that automation |
| FEAT-04.SPEC-007 (Calendar Reconnection Alert) | Navigation (inbound) | The reconnect banner's tap lands here, focused on the affected connection |
| FEAT-04.SPEC-008 (Calendar Connection Rules) | References (inbound) | One-per-kind limit and disconnect-wins contention rule govern this screen's list and Disconnect action |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound/outbound) | Reached from dashboard navigation and the attention list; Back returns there |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| calendar_status_viewed | connection_count, any_needs_reconnection (yes/no) | Screen is opened | supports success-metrics.md: "Calendar Sync Reliability" |
| calendar_reconnect_started | calendar_kind, entry_source (list / dashboard / banner) | Pro taps Reconnect | supports success-metrics.md: "Calendar Sync Reliability" |
| calendar_reconnected | calendar_kind | Reconnection handshake succeeds | supports success-metrics.md: "Calendar Sync Reliability" |
| calendar_disconnected | calendar_kind | Pro confirms Disconnect | supports success-metrics.md: "Calendar Sync Reliability" |

## Acceptance Criteria

**FEAT-04.SPEC-002-AC-01:** Given Talia has no calendar connections, when she opens this screen, then she sees the Empty state with a plain explanation and two "Connect" entries, one per kind.

**FEAT-04.SPEC-002-AC-02:** Given Talia has one Google connection in Connected status, when she opens this screen, then she sees one connection card showing "Connected" and its last successful sync time, plus an "Add another calendar" action for Apple.

**FEAT-04.SPEC-002-AC-03:** Given Talia's Apple connection is in Needs Reconnection status, when she views its card, then she sees the "Needs Reconnection" label (word-based, not color-only) and a Reconnect action.

**FEAT-04.SPEC-002-AC-04:** Given Talia taps Reconnect on a Needs Reconnection card, when the handshake succeeds, then the card's status updates to Connected (or Syncing) and the Reconnect action disappears.

**FEAT-04.SPEC-002-AC-05:** Given Talia taps Disconnect on a connected calendar, when the confirmation dialog appears and she taps Disconnect, then the connection is removed from the list and a toast confirms "{kind} Calendar disconnected."

**FEAT-04.SPEC-002-AC-06:** Given Talia taps Disconnect and then Cancel in the confirmation dialog, when the dialog closes, then the connection remains unchanged in the list.

**FEAT-04.SPEC-002-AC-07:** Given Talia disconnects a calendar with existing confirmed Chairtime bookings, when the disconnect completes, then those bookings and their history remain untouched on the Pro's dashboard.

**FEAT-04.SPEC-002-AC-08:** Given Talia is viewing a Connected card, when FEAT-04.SPEC-006 detects a lapsed connection while the screen is still open, then the card's status updates to Needs Reconnection without Talia navigating away and back.

**FEAT-04.SPEC-002-AC-09:** Given Talia taps Disconnect while a sync is in flight for that connection, when the disconnect confirms, then the disconnect wins per FEAT-04.SPEC-008: the in-flight sync is discarded and the connection is removed.

**FEAT-04.SPEC-002-AC-10:** Given Platform Operator Support opens this screen for a Pro during a help request, when the screen loads, then Support sees connection health and last sync time only, with no Reconnect or Disconnect actions shown.

**FEAT-04.SPEC-002-AC-11:** Given Talia loses connectivity while viewing this screen, when she looks at the connection cards, then the last-loaded status remains visible and the Reconnect/Disconnect actions are disabled with "This action needs an internet connection."

**FEAT-04.SPEC-002-AC-12:** Given Talia has both a Google and an Apple connection, when she views this screen, then no "Add another calendar" action is shown.

**FEAT-04.SPEC-002-AC-13:** Given Talia arrives at this screen from the FEAT-04.SPEC-007 reconnect banner, when the screen opens, then focus lands on the specific connection that needs reconnecting.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 7 (empty, list-healthy, list-needs-attention, reconnecting, loading, error, offline) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



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



# Automation Spec: Booking-to-Calendar Sync

## Overview

**Name:** Booking-to-Calendar Sync
**ID:** FEAT-04.SPEC-005
**Type:** Automation
**Purpose:** On any Chairtime booking create, reschedule, or cancel, writes, moves, or removes the matching event on the Pro's connected personal calendar, so the personal calendar never shows a stale or ghost appointment.
**Parent Feature:** FEAT-04 -- Two-Way Calendar Sync

## Scope and Non-Goals

**In Scope:**
- Writing a new calendar event when a Chairtime booking is confirmed
- Moving the matching calendar event when a booking is rescheduled
- Removing the matching calendar event when a booking is cancelled
- Retrying a failed write/move/remove operation and flagging persistent failure
- Confirming no calendar action is needed for a booking marked no-show

**Non-Goals:**
- Performing the actual write/move/remove operation against the calendar provider -- owned by FEAT-04.SPEC-003 (Calendar Provider Sync); this automation decides what operation is needed and instructs that spec to perform it
- Reading personal-calendar busy time to block Chairtime availability -- owned by FEAT-04.SPEC-004 (Busy-Time Availability Feed); this automation is the reverse direction (Chairtime-to-calendar)
- Deciding a connection's health status -- owned by FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation); a failed write here feeds that spec's monitoring, it does not set health status itself
- Notifying the client that their booking changed -- owned by FEAT-08 (Automated Booking Messaging); this automation's scope is the Pro's personal calendar only

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Booking confirmed | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation), completing the FEAT-05 (Public Booking Page & Booking Flow) checkout | Fires when a booking transitions to Confirmed | Booking reference: service, start_time, duration; Pro's connected calendar connection(s) |
| Booking confirmed (Pro-initiated) | FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold), confirmed through FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Fires when the Pro books a client in directly and the booking is confirmed | Booking reference: service, start_time, duration |
| Booking rescheduled (client-initiated) | FEAT-10.SPEC-004 (Booking Update Commit) (Client-Initiated Cancel/Reschedule) | Fires when a client reschedules their own booking to a new time | Booking reference with new start_time; previous start_time for locating the existing calendar event |
| Booking rescheduled (Pro-initiated) | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) (Pro Booking Management) | Fires when the Pro reschedules a booking | Booking reference with new start_time; previous start_time |
| Booking cancelled (client-initiated) | FEAT-10.SPEC-004 (Booking Update Commit) (Client-Initiated Cancel/Reschedule) | Fires when a client cancels their own booking | Booking reference; original start_time for locating the existing calendar event |
| Booking cancelled (Pro-initiated) | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) / FEAT-30.SPEC-008 (Bulk Cancellation Commit) (Pro Booking Management) | Fires when the Pro cancels a booking | Booking reference; original start_time |
| Booking marked no-show | FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) | Fires when a booking is marked no-show | Booking reference |

## Processing Logic

1. Receive the triggering Booking reference and the lifecycle event (confirmed, rescheduled, cancelled, marked no-show).
2. Look up the Pro Account's connected calendar connection(s) (at most two, one per kind).
3. If the Pro has no connected calendar, stop -- there is nothing to sync (no-op, not a failure).
4. If the event is "marked no-show," confirm no calendar action is required (the appointment already occurred) and stop -- this path exists to make the no-action decision explicit, not to skip it silently.
5. If the event is "confirmed," instruct FEAT-04.SPEC-003 to write a new calendar event containing the service name, start_time, and duration for every connected calendar.
6. If the event is "rescheduled," instruct FEAT-04.SPEC-003 to move the existing calendar event (located by the booking's previous start_time and service) to the new start_time, for every connected calendar.
7. If the event is "cancelled," instruct FEAT-04.SPEC-003 to remove the existing calendar event (located by the booking's start_time and service), for every connected calendar.
8. Await the write/move/remove confirmation from FEAT-04.SPEC-003 for each connected calendar independently.
9. On confirmed success for a given connection, record the operation as complete for that connection.
10. On failure for a given connection, retry per the failure-path rules in Outcome Definitions; other connected calendars are unaffected by one connection's failure.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Event written | Booking confirmed, write succeeds | Calendar Connection -- last_successful_sync updated for the connection | None directly to the Pro on this automation's own surface; the appointment simply appears on the personal calendar | FEAT-04.SPEC-003 (Calendar Provider Sync) |
| Event moved | Booking rescheduled, move succeeds | Calendar Connection -- last_successful_sync updated | None directly; the appointment's time updates on the personal calendar | FEAT-04.SPEC-003 (Calendar Provider Sync) |
| Event removed | Booking cancelled, remove succeeds | Calendar Connection -- last_successful_sync updated | None directly; the appointment disappears from the personal calendar, leaving no ghost entry | FEAT-04.SPEC-003 (Calendar Provider Sync) |
| No action needed (no-show) | Booking marked no-show | None -- explicitly no calendar action | None -- the appointment already occurred and its calendar entry (if any) is left as-is | -- |
| No calendar connected | Pro has zero connections at trigger time | None | None -- this is a normal, expected state for a Pro who has not connected a calendar (calendar connection is optional per XBR-26) | -- |
| Operation retried | Write/move/remove fails on first attempt | No calendar-side change yet; Chairtime booking state is unaffected | None visible to the Pro during retry; the operation is not user-facing at this stage | FEAT-04.SPEC-003 (Calendar Provider Sync) |
| Operation failed after retries | Write/move/remove fails after exhausting retries | No calendar-side change; Chairtime booking state remains authoritative and unchanged | The Pro's dashboard attention list (FEAT-12) shows a flag: "Couldn't update your calendar for {service} on {date}. Your Chairtime booking is correct -- try reconnecting your calendar." | FEAT-12 (Pro Daily Schedule Dashboard), FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) |

## Data Model

**Reads:** Booking -- service, start_time, duration, state (from FEAT-05, FEAT-10, FEAT-11, FEAT-30); Calendar Connection -- calendar_kind, status for the Pro Account.
**Creates:** None directly -- calendar events are created by FEAT-04.SPEC-003, not persisted as a Chairtime entity.
**Updates:** None directly -- Calendar Connection.last_successful_sync is updated by FEAT-04.SPEC-003 as the operation's confirming spec.
**Deletes:** None directly -- calendar event removal is performed by FEAT-04.SPEC-003.

## Business Rules

- XBR-13: every booking created, rescheduled, or cancelled by either party is written, moved, or removed on the Pro's personal calendar; this automation is the mechanism that enforces that rule for every lifecycle event.
- A booking marked no-show never triggers a calendar action, since the appointment already occurred -- this is a deliberate no-op, not an unhandled case.
- If the Pro has two connections (Google and Apple), the same booking event is written, moved, or removed on both independently; one connection's failure does not block or roll back the operation on the other.
- Client identity and notes never appear in the calendar event content, per FEAT-04.SPEC-003's Data Exchanged section -- only service name, start time, and duration.

## Edge Cases

- **A booking is rescheduled twice in quick succession** -- The second reschedule's move instruction supersedes the first: FEAT-04.SPEC-003 is instructed to move the calendar event to the final new start_time directly, never producing two move operations that could leave the calendar event at an intermediate time.
- **A booking is cancelled immediately after being confirmed, before the write completes** -- The cancel instruction waits for the write to either complete or fail; once the write's outcome is known, the remove instruction is issued against the now-existing (or never-created) event, so no ghost event and no failed removal-of-nothing occurs.
- **Calendar connection is disconnected while a write/move/remove is in flight** -- Per FEAT-04.SPEC-008's contention rule, the disconnect wins: the in-flight operation is discarded, and no retry is attempted against a connection that no longer exists.
- **Concurrent trigger firing (two different bookings for the same Pro change at effectively the same time)** -- Each booking's write/move/remove is instructed independently to FEAT-04.SPEC-003; the calendar-sync capability applies both as separate events, with no interference between them.
- **Trigger fires while a previous run is in flight for the same booking** -- A second lifecycle event for the same booking (e.g., a reschedule arriving while the original confirm's write is still in flight) waits for the in-flight operation's outcome before issuing its own instruction, per the reschedule-in-quick-succession handling above; operations for different bookings never queue behind each other.
- **Pro connects a calendar after several bookings already exist** -- No retroactive backfill occurs from this automation; only bookings whose lifecycle event fires after the connection exists are synced. (Existing bookings created before the connection are unaffected, consistent with the dependency map's Booking lifecycle being independent of Calendar Connection's.)

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) -- within FEAT-05 (Public Booking Page & Booking Flow) checkout | Triggered by (inbound) | Booking confirmation fires the write path |
| FEAT-10.SPEC-004 (Booking Update Commit) (Client-Initiated Cancel/Reschedule) | Triggered by (inbound) | Client reschedule/cancel fires the move/remove paths |
| FEAT-11.SPEC-002 (No-Show Marking & Deposit Forfeiture) | Triggered by (inbound) | No-show marking fires the explicit no-action confirmation |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit), FEAT-30.SPEC-008 (Bulk Cancellation Commit), FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) (Pro Booking Management) | Triggered by (inbound) | Pro-initiated confirm/reschedule/cancel fires the corresponding path |
| FEAT-04.SPEC-003 (Calendar Provider Sync) | Triggers (outbound) | Every write/move/remove instruction is carried out through this spec |
| FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) | Affects (outbound) | Persistent write/move/remove failures feed this spec's monitoring |
| FEAT-04.SPEC-008 (Calendar Connection Rules) | References (inbound) | Disconnect-wins-over-in-flight-sync contention rule governs in-flight operations |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | Persistent failure flag surfaces on the dashboard's attention list |

## Analytics and Success Signals

- **booking_calendar_write_attempted** (operation: write / move / remove, calendar_kind) -- supports success-metrics.md: "Calendar Sync Reliability"
- **booking_calendar_write_succeeded** (operation, calendar_kind, latency_bucket: within target / slow) -- supports success-metrics.md: "Calendar Sync Reliability"
- **booking_calendar_write_failed_after_retries** (operation, calendar_kind) -- supports success-metrics.md: "Calendar Sync Reliability"
- **booking_calendar_no_action_confirmed** (reason: no_show / no_connection) -- N/A -- no Stage 2 metric measures the no-action path directly; retained so the deliberate no-op is distinguishable from a missed sync in operational review.

## Acceptance Criteria

**FEAT-04.SPEC-005-AC-01:** Given Talia has a Google calendar connected and a client confirms a booking via FEAT-05, when the confirmation fires, then this automation instructs FEAT-04.SPEC-003 to write a new event on the Google calendar with the service, start time, and duration.

**FEAT-04.SPEC-005-AC-02:** Given a booking has a calendar event written on Talia's Apple calendar, when the client reschedules the booking via FEAT-10, then this automation instructs FEAT-04.SPEC-003 to move the existing event to the new time.

**FEAT-04.SPEC-005-AC-03:** Given a booking has a calendar event on Talia's connected calendar, when Talia cancels the booking via FEAT-30, then this automation instructs FEAT-04.SPEC-003 to remove the matching event so no ghost appointment remains.

**FEAT-04.SPEC-005-AC-04:** Given a booking passes its appointment time, when it is marked no-show via FEAT-11, then this automation confirms no calendar action is needed and takes none.

**FEAT-04.SPEC-005-AC-05:** Given Talia has no calendar connected, when a booking is confirmed, then this automation completes with no calendar action and no error, since calendar connection is optional.

**FEAT-04.SPEC-005-AC-06:** Given Talia has both a Google and an Apple connection, when a booking is confirmed, then this automation instructs a write for both connections independently, and a failure on one does not affect the other.

**FEAT-04.SPEC-005-AC-07:** Given a calendar write fails on first attempt, when the automation retries, then the underlying Chairtime booking remains Confirmed and unaffected throughout the retry.

**FEAT-04.SPEC-005-AC-08:** Given a calendar write fails after exhausting retries, when the failure is final, then Talia's dashboard attention list (FEAT-12) shows "Couldn't update your calendar for {service} on {date}. Your Chairtime booking is correct -- try reconnecting your calendar."

**FEAT-04.SPEC-005-AC-09:** Given a booking is rescheduled twice within moments of each other, when both reschedules process, then the calendar event ends at the final new time, with no intermediate move left stranded.

**FEAT-04.SPEC-005-AC-10:** Given a booking is cancelled immediately after confirmation, before its write completes, when both events process, then the remove instruction is issued only after the write's outcome is known, leaving no ghost event.

**FEAT-04.SPEC-005-AC-11:** Given Talia disconnects her calendar while a write is in flight for a just-confirmed booking, when the disconnect completes, then the in-flight write is discarded and not retried against the removed connection.

**FEAT-04.SPEC-005-AC-12:** Given two different bookings for Talia change at effectively the same time, when both lifecycle events fire, then each is instructed to FEAT-04.SPEC-003 independently with no interference between them.

**FEAT-04.SPEC-005-AC-13:** Given Talia connects a calendar after several bookings already exist, when the connection completes, then none of those pre-existing bookings are retroactively written to the calendar -- only bookings whose lifecycle event fires afterward are synced.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 7 | 7 |
| Outcome Paths | 7 | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



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



# Notification Spec: Calendar Reconnection Alert

## Overview

**Name:** Calendar Reconnection Alert
**ID:** FEAT-04.SPEC-007
**Type:** Notification
**Purpose:** Tells the Pro, via a dashboard banner, that a connected calendar needs reconnecting, so a lapsed connection is never a silent gap.
**Parent Feature:** FEAT-04 -- Two-Way Calendar Sync

## Scope and Non-Goals

**In Scope:**
- The dashboard banner delivered when a connection transitions to Needs Reconnection
- The banner's behavior while the connection remains unresolved (persistence, not repeated re-delivery)
- Dismissal and re-appearance behavior
- Clearing the alert once the connection is reconnected

**Non-Goals:**
- Deciding when a connection is considered lapsed -- owned by FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation); this notification only delivers what that automation decides
- The reconnection flow itself -- owned by FEAT-04.SPEC-002 (Calendar Connection Status & Management); this notification's CTA only deep-links there
- Delivery by text or email -- excluded per product-features.md's Communications field, which states explicitly this is "a dashboard alert (not a text/email)"; no other channel exists for this notification
- General Pro notification preferences for other event types (new bookings, refund failures, disputes) -- owned by FEAT-08 (Automated Booking Messaging); this spec covers only the calendar-reconnection alert

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, for as long as the connection remains in Needs Reconnection status | Talia works inside the product in short bursts between clients (user-persona.md, Behavioral Context); a dashboard banner is where she is already looking, and product-features.md's Communications field states this is deliberately not a text/email interruption -- a calendar hiccup does not warrant pulling her out of a client appointment |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Connection status becomes Needs Reconnection | FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) | Fires whenever a connection transitions to Needs Reconnection | Calendar Connection reference, calendar_kind |

## Audience and Preferences

**Recipients:** The Pro (Talia) only -- the sole role that connects and manages calendars (Access Matrix: Service & Availability Setup = Full). The Client never sees this alert (Access Matrix: None), and Platform Operator (Support) has View-only access to connection health but is not a recipient of this Pro-facing dashboard alert.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- no preference control exists for this alert | -- | Always on | -- |

This alert has no opt-out: product-features.md's Validation & Limits states a sync failure "must surface as a visible dashboard banner rather than a silent gap," and ASMP-26's correctness bar ("never silently double-book") makes this a mandatory, non-optional alert rather than a preference-gated notification.

**Quiet Hours:** N/A -- quiet hours govern time-sensitive, interruption-style notifications (product-features.md, Automated Booking Messaging); a persistent in-app dashboard banner is not a point-in-time interruption and carries no quiet-hours concept of its own.

## Content Definition

**In-app:**
- **Title:** Reconnect your {calendar_kind} calendar
- **Body:** Chairtime lost access to your calendar, so it's not blocking or updating your personal calendar right now. Reconnect to keep everything in sync.
- **CTA:** Reconnect -- deep-links to FEAT-04.SPEC-002 (Calendar Connection Status & Management) for the specific connection needing reconnection, with focus landing on that connection's card

**Batched variant (both connections need reconnecting at once):**
- **In-app title:** Reconnect your calendars
- **In-app body:** Chairtime lost access to your Google and Apple calendars, so neither is blocking or updating your personal calendars right now. Reconnect to keep everything in sync.
- **CTA:** Reconnect -- deep-links to FEAT-04.SPEC-002 (Calendar Connection Status & Management), showing both connections needing reconnection

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {calendar_kind} | Calendar Connection -- calendar_kind | Google | Never empty -- calendar_kind is required on every Calendar Connection record and is always known at trigger time |

## Delivery Rules

**Batching:** If both of a Pro's connections (Google and Apple) are in Needs Reconnection status at the same time, the two individual alerts collapse into the single batched variant above rather than showing two separate banners. If only one connection lapses while the other remains healthy, the single-connection variant is shown.
**Deduplication:** At most one active banner per connection at a time. A connection already showing its banner does not produce a second, duplicate banner if FEAT-04.SPEC-006's scheduled health check re-confirms the same Needs Reconnection status on a later run -- the existing banner simply continues to be shown.
**Retry on failure:** N/A -- this is a persistent in-app banner state, not a point-in-time send that can fail to deliver; as long as the connection remains in Needs Reconnection status, the banner is present whenever the Pro views the dashboard.
**Expiry:** The banner never expires on its own -- it persists for as long as the connection remains in Needs Reconnection status, since a stale reconnect prompt that quietly disappeared would recreate exactly the silent-gap risk this alert exists to prevent. It clears immediately once FEAT-04.SPEC-006 sets the connection back to Connected.

## Edge Cases

- **Pro dismisses the banner without reconnecting** -- The banner reappears the next time the Pro opens the dashboard, since dismissal is not the same as resolution; only a successful reconnection (FEAT-04.SPEC-006 setting status back to Connected) clears it.
- **Pro disconnects the lapsed connection instead of reconnecting it** -- The banner for that connection clears immediately, since the underlying record (and its Needs Reconnection status) no longer exists; this is treated the same as a resolved state, not left showing a reconnect prompt for a connection that is gone.
- **Both connections lapse, then one is reconnected while the other remains lapsed** -- The batched variant is replaced by the single-connection variant for the still-lapsed connection; the reconnected one's contribution to the banner clears without affecting the other.
- **Connection reconnects and then immediately lapses again (flapping)** -- Each transition is treated independently: the banner clears on the Connected transition and reappears fresh on the next Needs Reconnection transition; no cooldown suppresses a second, genuine alert.
- **Connection reference no longer exists when the dashboard renders the banner (disconnected between trigger and render)** -- The banner is not rendered for a connection that no longer exists; the dashboard reflects the Pro's current connection list at render time, not a stale trigger snapshot.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) | Triggered by (inbound) | The Needs Reconnection transition fires this notification; the Connected transition clears it |
| FEAT-04.SPEC-002 (Calendar Connection Status & Management) | Navigation (outbound) | The Reconnect CTA deep-links here, focused on the affected connection |
| FEAT-12 (Pro Daily Schedule Dashboard) | References (inbound) | This banner surfaces on the dashboard's attention list, per the dependency map's Navigation Connections |

## Analytics and Success Signals

- **reconnect_alert_shown** (calendar_kind or "both", batched: yes / no) -- supports success-metrics.md: "Calendar Sync Reliability"
- **reconnect_alert_cta_tapped** (calendar_kind or "both") -- supports success-metrics.md: "Calendar Sync Reliability"
- **reconnect_alert_cleared** (calendar_kind, resolution: reconnected / disconnected) -- supports success-metrics.md: "Calendar Sync Reliability"

## Acceptance Criteria

**FEAT-04.SPEC-007-AC-01:** Given Talia's Google connection transitions to Needs Reconnection, when she next opens her dashboard, then she sees the banner "Reconnect your Google calendar" with its body and a Reconnect CTA.

**FEAT-04.SPEC-007-AC-02:** Given Talia taps the Reconnect CTA on the banner, when the tap registers, then she lands on FEAT-04.SPEC-002 with focus on the Google connection's card.

**FEAT-04.SPEC-007-AC-03:** Given both of Talia's connections are Needs Reconnection at the same time, when she opens her dashboard, then she sees the single batched banner "Reconnect your calendars," not two separate banners.

**FEAT-04.SPEC-007-AC-04:** Given Talia's connection is reconnected successfully, when FEAT-04.SPEC-006 sets status back to Connected, then the banner clears from her dashboard.

**FEAT-04.SPEC-007-AC-05:** Given Talia dismisses the banner without reconnecting, when she next opens her dashboard, then the banner reappears, since dismissal does not resolve the lapsed connection.

**FEAT-04.SPEC-007-AC-06:** Given Talia disconnects a lapsed connection instead of reconnecting it, when the disconnect completes, then the banner for that connection clears immediately.

**FEAT-04.SPEC-007-AC-07:** Given both of Talia's connections are lapsed and she reconnects only one, when the dashboard re-renders, then the batched banner is replaced by the single-connection banner for the one still lapsed.

**FEAT-04.SPEC-007-AC-08:** Given Talia's connection reconnects and then lapses again shortly after, when the second lapse occurs, then a fresh banner is shown with no suppression from the first alert having just cleared.

**FEAT-04.SPEC-007-AC-09:** Given a connection was disconnected between the trigger firing and the dashboard rendering, when Talia opens the dashboard, then no banner is shown for that no-longer-existing connection.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (no preference -- always on) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry N/A, expiry) | 4 |
| Edge Cases | 5 | 5 |



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

