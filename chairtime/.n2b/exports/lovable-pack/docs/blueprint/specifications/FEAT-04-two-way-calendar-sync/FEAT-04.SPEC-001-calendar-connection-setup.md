---
document_type: spec
spec_type: screen
spec_id: FEAT-04.SPEC-001
spec_name: Calendar Connection Setup
spec_slug: calendar-connection-setup
parent_feature: FEAT-04
parent_feature_name: Two-Way Calendar Sync
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 11
---

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
