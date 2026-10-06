---
document_type: spec
spec_type: screen
spec_id: FEAT-21.SPEC-001
spec_name: Calendar Connection Settings
spec_slug: calendar-connection-settings
parent_feature: FEAT-21
parent_feature_name: Family Calendar Sync
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Calendar Connection Settings

## Overview

**Name:** Calendar Connection Settings
**ID:** FEAT-21.SPEC-001
**Type:** Screen
**Purpose:** Maya connects or disconnects the household's calendar capability and sees its current connection status.
**Parent Feature:** FEAT-21 -- Family Calendar Sync

## Scope and Non-Goals

**In Scope:**
- Showing the household's current calendar-connection status (not connected, connecting, connected, error)
- Letting Maya start a connection to the calendar capability
- Letting Maya disconnect an active connection
- Surfacing a failed connect attempt and an externally-lost connection with a clear status and recovery path

**Non-Goals:**
- Viewing the synced calendar or its entries in-app -- excluded per product-features.md Data Notes: "Displayed: N/A within the product beyond a connection status"; the calendar and its entries are viewed entirely outside the product, on the household's own calendar.
- Sam or either kid row connecting or disconnecting the calendar -- excluded per the feature's Access field, which reserves this action to Maya (Organiser) alone; this screen is not reachable by any other role.
- Choosing which external calendar provider to connect to -- vendor selection is a Stage 4 architecture decision (FEAT-21.SPEC-002, Capability Category); this screen presents a single "Connect calendar" action, not a provider picker.
- Managing the content or timing of individual synced entries -- owned by FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync) and FEAT-21.SPEC-005 (Calendar Entry Content Derivation Rule); this screen shows connection status only, never per-night entry detail.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-010 (Household Settings Hub) | Maya taps the "Connect calendar" row (organiser only, Later phase) | None -- screen loads the household's current connection status |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Connect and disconnect the calendar capability | -- |
| Sam (Other Adult Member) | No | No | The "Connect calendar" row is not shown on FEAT-01.SPEC-010 for Sam; a direct navigation attempt shows "Only the organiser can change this." and returns to FEAT-01.SPEC-010 |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this role; there is no path into the product to reach it |
| Jordan (older kid, limited login -- Later) | No | No | The "Connect calendar" row is not shown on FEAT-01.SPEC-010 for this role; a direct navigation attempt shows "Only the organiser can change this." and returns to FEAT-01.SPEC-010 |
| Riley (Operator, support) | No | No | This feature carries no support-access exposure (product-features.md Compliance flags); the read-only support view never surfaces this screen or a calendar-connection row |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on FEAT-01.SPEC-010 (Household Settings Hub), not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no in-progress connect or disconnect action is preserved; on re-authentication Maya returns to FEAT-01.SPEC-010 and must re-initiate the action |

## Layout and Content

**Header:** Screen title "Calendar Connection" with a back arrow (returns to FEAT-01.SPEC-010, Household Settings Hub).

**Body:** A single status card, centered, showing one of the following depending on the household's connection state:

- **Not connected:** An icon-and-text block reading "Your calendar isn't connected" with supporting text "Connect a calendar and each night's dinner will appear on it automatically." followed by a "Connect calendar" button.
- **Connected:** A status line "Connected to your calendar" with supporting text "Dinners sync automatically -- swap a meal and the calendar updates too." followed by a "Disconnect" button (secondary/lower-emphasis styling).
- **Connecting:** The "Connect calendar" button in a loading state; the rest of the card is otherwise unchanged from Not Connected.
- **Error:** The Not Connected block, plus an inline error message directly below the supporting text.

No other content or navigation appears on this screen.

### Responsive Behavior

- **Compact breakpoint:** Status card full width below the header, vertically stacked (icon/text, then button).
- **Medium size class and above:** Status card capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-010 (Household Settings Hub) | Screen closes | Standard transition back to the hub |
| "Connect calendar" button (Not Connected state) | Tap | Initiates a connection request via FEAT-21.SPEC-002 (Family Calendar Integration) | Button enters loading state; card enters Connecting state | Button shows a loading indicator |
| "Connect calendar" button (while Connecting) | Tap | No action -- debounced | None | Button remains in loading state |
| "Disconnect" button (Connected state) | Tap | Opens a confirmation dialog: "Disconnect calendar? Entries already created on your calendar will stay there -- only new syncing stops." with "Disconnect" and "Keep Connected" options | Dialog appears | Dialog is modal until dismissed |
| Confirmation dialog "Disconnect" | Tap | Initiates a disconnect request via FEAT-21.SPEC-002 | Dialog closes; card enters Not Connected state once confirmed | Card updates to "Your calendar isn't connected" |
| Confirmation dialog "Keep Connected" | Tap | Dismisses the dialog; no request sent | Dialog closes | Card remains Connected, unchanged |

### Accessibility Notes

- **Focus order:** Back arrow -> status text -> primary button (Connect or Disconnect).
- **Loading announcement:** When the Connect action starts, "Connecting to your calendar" is announced to assistive technology.
- **Error announcement:** An error message is announced to assistive technology when it appears and is programmatically associated with the status card.
- **Confirmation dialog:** Focus moves into the dialog on open and returns to the "Disconnect" button on dismissal; both dialog options are reachable by keyboard.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Not Connected (default) | "Your calendar isn't connected" block with "Connect calendar" button | Screen opens for a household with no active connection, or a disconnect completes | Maya taps "Connect calendar" |
| Connecting | "Connect calendar" button shows a loading indicator; button disabled | Maya taps "Connect calendar" | FEAT-21.SPEC-002 confirms the connection (-> Connected) or reports it could not be established (-> Error) |
| Connected | "Connected to your calendar" block with "Disconnect" button | FEAT-21.SPEC-002 confirms the connection | Maya disconnects (Calendar Connection.status -> Disconnected, disconnect_reason -> User-Initiated, per FEAT-21.SPEC-004), or FEAT-21.SPEC-002's inbound event reports the connection was revoked or lost outside the product (status -> Disconnected, disconnect_reason -> Revoked-Externally or Retry-Ceiling-Exhausted -- Externally Disconnected state below) |
| Externally Disconnected | Not Connected block, plus a supporting line: "Your calendar connection ended outside the app. Reconnect to keep syncing." | Calendar Connection.disconnect_reason is Revoked-Externally or Retry-Ceiling-Exhausted and externally_disconnected_acknowledged is false (FEAT-21.SPEC-004) -- this is what distinguishes this state from the plain Not Connected state a User-Initiated disconnect produces, which shows no note at all | Maya taps "Connect calendar" again (disconnect_reason and externally_disconnected_acknowledged reset per FEAT-21.SPEC-004), or leaves and returns to the screen, which sets externally_disconnected_acknowledged to true and drops the note (screen then shows plain Not Connected on the next visit) |
| Error | Not Connected block, plus inline error text: "Couldn't connect your calendar. Try again." | A connect attempt fails (FEAT-21.SPEC-002 reports the request was rejected or could not complete) | Maya taps "Connect calendar" again |
| Offline/Degraded | The card shows the last known connection status (Not Connected or Connected) from before connectivity was lost; both "Connect calendar" and "Disconnect" are disabled with the note "Connecting to a calendar needs an internet connection." | Connectivity is lost while this screen is open | Connectivity is restored -- buttons re-enable and the screen refreshes the current status |

## Validation Rules

Validation governed by FEAT-21.SPEC-004 (Calendar Connection & Sync Governance Rules). See that spec for the one-connection-per-household limit and retry behavior that determine whether a connect attempt can proceed.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-01.SPEC-010 (Household Settings Hub) | FEAT-01 (Household Setup & Member Profiles) |
| Connect or disconnect completes | Stays on this screen with the updated status | -- |

## Data Model

**Creates:** None of the product's own dependency-map entities -- a connect action starts a connection request handled entirely by FEAT-21.SPEC-002 (Family Calendar Integration) and governed by FEAT-21.SPEC-004; the household's calendar-connection state is not a Connected Entity of this feature (feature-overview.md, Entity-Lifecycle Coverage Matrix).
**Reads:** The household's current calendar-connection status (Not Connected / Connecting / Connected / Externally Disconnected / Error), maintained by FEAT-21.SPEC-002 and FEAT-21.SPEC-004; and, to distinguish the Externally Disconnected state from a plain User-Initiated Not Connected, the Calendar Connection's disconnect_reason and externally_disconnected_acknowledged fields (FEAT-21.SPEC-004).
**Updates:** None directly from a connect or disconnect request -- FEAT-21.SPEC-002 and FEAT-21.SPEC-004 own the resulting state change. The one exception is externally_disconnected_acknowledged: this screen sets it to true (per FEAT-21.SPEC-004) purely as a side effect of Maya leaving the screen while the Externally Disconnected note is showing -- no explicit action or button dismisses it.
**Deletes:** None.

## Business Rules

- The one-connection-per-household limit and retry-on-failure behavior are governed by FEAT-21.SPEC-004 -- this screen does not restate them.
- Disconnecting never removes calendar entries already created on the household's calendar (FEAT-21.SPEC-004) -- the confirmation dialog states this explicitly so Maya is never surprised by data loss that does not occur.
- A failed background sync (as opposed to a failed connect attempt) never surfaces on this screen -- per the feature's States field ("a failed sync attempt retries automatically and does not affect the in-app plan"), only the connect action itself shows an error state here.

## Edge Cases

- **Maya double-taps "Connect calendar"** -- The second tap is ignored while the first request is in flight (button in loading state).
- **Maya navigates away while Connecting** -- The request continues in the background; when she returns to this screen (or FEAT-01.SPEC-010's hub shows an updated summary), the current status is shown.
- **Maya attempts to connect from a second device while a connect request from the first device is still in flight** -- The second request is rejected per FEAT-21.SPEC-004's one-connection-per-household limit; the second device shows "This household is already connecting a calendar. Check the other device." and its screen refreshes to the first device's eventual result.
- **Maya taps "Disconnect" then "Keep Connected"** -- No request is sent; the card remains Connected with no visible change.
- **The connection is lost externally while this screen is open** -- The screen updates live to the Externally Disconnected state without requiring Maya to leave and return.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound) | Organiser-only "Connect calendar" row is this screen's sole entry point |
| FEAT-21.SPEC-002 (Family Calendar Integration) | Triggers (outbound) | Connect and disconnect actions initiate requests through this integration; its inbound events (connection established, connection rejected, connection revoked externally) drive this screen's status |
| FEAT-21.SPEC-004 (Calendar Connection & Sync Governance Rules) | References (inbound) | Enforces the one-connection-per-household limit and defines what disconnecting does and does not affect |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| calendar_connect_initiated | -- | Maya taps "Connect calendar" | N/A -- no metric in success-metrics.md names Family Calendar Sync (or any of its behavior) as its Connected Feature; retained so the connect flow's usage is observable even without a product metric depending on it |
| calendar_connected | -- | FEAT-21.SPEC-002 confirms the connection | N/A -- same reason as above |
| calendar_disconnect_confirmed | -- | Maya confirms disconnect in the dialog | N/A -- same reason as above |
| calendar_connect_failed | failure reason category | A connect attempt is rejected or cannot complete | N/A -- same reason as above |

## Acceptance Criteria

**FEAT-21.SPEC-001-AC-01:** Given Maya (Organiser) opens Calendar Connection Settings for a household that has never connected a calendar, when the screen loads, then it shows "Your calendar isn't connected" with a "Connect calendar" button.

**FEAT-21.SPEC-001-AC-02:** Given Maya is on this screen in the Not Connected state, when she taps "Connect calendar", then the button shows a loading indicator and the screen enters the Connecting state.

**FEAT-21.SPEC-001-AC-03:** Given Maya's connect request is confirmed by FEAT-21.SPEC-002, when the screen updates, then it shows "Connected to your calendar" with a "Disconnect" button.

**FEAT-21.SPEC-001-AC-04:** Given Maya's connect request is rejected, when the screen updates, then it shows the Not Connected block with the inline error "Couldn't connect your calendar. Try again."

**FEAT-21.SPEC-001-AC-05:** Given Maya is viewing a Connected household, when she taps "Disconnect", then a confirmation dialog appears reading "Disconnect calendar? Entries already created on your calendar will stay there -- only new syncing stops." with "Disconnect" and "Keep Connected" options.

**FEAT-21.SPEC-001-AC-06:** Given Maya sees the disconnect confirmation dialog, when she taps "Disconnect", then the connection is torn down and the screen returns to the Not Connected state.

**FEAT-21.SPEC-001-AC-07:** Given Maya sees the disconnect confirmation dialog, when she taps "Keep Connected", then the dialog closes and the screen remains in the Connected state unchanged.

**FEAT-21.SPEC-001-AC-08:** Given a household's calendar connection is revoked outside the product, when Maya next opens this screen, then it shows the Not Connected block with "Your calendar connection ended outside the app. Reconnect to keep syncing."

**FEAT-21.SPEC-001-AC-09:** Given Maya loses connectivity while this screen is open, when she attempts to tap "Connect calendar" or "Disconnect", then both buttons are disabled with "Connecting to a calendar needs an internet connection." shown, and the last known status remains displayed.

**FEAT-21.SPEC-001-AC-10:** Given Sam attempts to navigate directly to this screen, when the navigation resolves, then he sees "Only the organiser can change this." and is returned to FEAT-01.SPEC-010.

**FEAT-21.SPEC-001-AC-11:** Given Maya has a connect request in flight on one device, when she attempts to connect from a second device at the same time, then the second device's request is rejected with "This household is already connecting a calendar. Check the other device."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 6 (not connected, connecting, connected, externally disconnected, error, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
