---
document_type: spec
spec_type: screen
spec_id: FEAT-04.SPEC-002
spec_name: Calendar Connection Status & Management
spec_slug: calendar-connection-status-management
parent_feature: FEAT-04
parent_feature_name: Two-Way Calendar Sync
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 13
---

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
