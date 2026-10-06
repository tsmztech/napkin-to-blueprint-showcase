# FEAT-21 — Family Calendar Sync

This chapter covers FEAT-21, Family Calendar Sync, a Nice-to-Have-tier feature. It contains 5 specifications carrying 71 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-21.SPEC-001 | Calendar Connection Settings | screen | 11 |
| FEAT-21.SPEC-002 | Family Calendar Integration | integration | 14 |
| FEAT-21.SPEC-003 | Weekly Dinner Calendar Sync | automation | 15 |
| FEAT-21.SPEC-004 | Calendar Connection & Sync Governance Rules | logic-rule | 19 |
| FEAT-21.SPEC-005 | Calendar Entry Content Derivation Rule | logic-rule | 12 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Family Calendar Sync

## Summary

**Feature:** Family Calendar Sync
**ID:** FEAT-21
**Description:** The week's dinners can appear on the household's existing family calendar, so meal plans show up alongside everything else the family has scheduled.
**Priority:** Nice-to-Have
**Phase:** Later
**Type:** Platform
**Rationale:** The brief names this directly as "a nice-to-have for showing dinner on the family calendar, not v1" (BRIEF.md, Ecosystem & Integrations). Phased to Later exactly per the brief's own stated timing. [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Sync the week's dinners — Household connects a calendar capability so each night's dinner appears as an entry
- Keep it current — A swapped meal updates the corresponding calendar entry automatically

This feature is entirely an additive, background-integration feature: it defines exactly one screen of its own (the connection control point) and otherwise operates as a system-to-system sync with no other in-app surface. Per the feature's own States field, it is additive only — a failed sync, a lost connection, or the capability never being connected at all leaves the in-app Weekly Plan (owned by FEAT-03 and FEAT-23) entirely unaffected.

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-21.SPEC-001 | Calendar Connection Settings | Screen | Maya | Maya connects or disconnects the household's calendar capability and sees its current connection status |
| FEAT-21.SPEC-002 | Family Calendar Integration | Integration | Maya, Sam, Jordan (young kid), Jordan (older kid) | The connection contract with the external calendar capability — connect/disconnect, entry create/update, and the inbound sync-outcome and disconnect events, per the family-calendar capability category (ASMP-37) |
| FEAT-21.SPEC-003 | Weekly Dinner Calendar Sync | Automation | Maya, Sam, Jordan (young kid), Jordan (older kid) | Watches the Weekly Plan for dinners to sync and for swaps that change an already-synced night, and drives the Integration spec to create or update the matching calendar entry |
| FEAT-21.SPEC-004 | Calendar Connection & Sync Governance Rules | Logic/Rule | Maya, Sam, Jordan (young kid), Jordan (older kid) | Governs the one-connection-per-household limit, retry-on-failure behavior, the additive-only/non-blocking guarantee for the in-app plan, and what disconnecting does and does not affect |
| FEAT-21.SPEC-005 | Calendar Entry Content Derivation Rule | Logic/Rule | Maya, Sam, Jordan (young kid), Jordan (older kid) | Derives what a synced calendar entry contains (night, dish, timing) from Weekly Plan and Planned Meal data, and keeps one entry per planned night stable across swaps |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Sync the week's dinners | FEAT-21.SPEC-001, FEAT-21.SPEC-002, FEAT-21.SPEC-003, FEAT-21.SPEC-005 | Maya connects the calendar capability from Settings; the sync automation walks the week's Planned Meals and drives the Integration spec to create one entry per dinner, with content derived by SPEC-005 | Phase 2 (Explicit) |
| Keep it current | FEAT-21.SPEC-003, FEAT-21.SPEC-004, FEAT-21.SPEC-005 | A meal swap re-fires the sync automation, which updates the same night's existing calendar entry (per SPEC-005's stable per-night mapping) rather than creating a duplicate | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-21.SPEC-002 | Family Calendar Integration | Phase 4 (External Dependencies lens) | The Dependencies section of assumptions-constraints.md (ASMP-37) names family-calendar capabilities as a category-level external dependency this feature relies on, and the dependency map's External Touchpoints row for it was left "pending — awaiting validated Brief for FEAT-21"; this Brief resolves that row with a standalone Integration spec covering connect, disconnect, entry create/update, and the inbound sync-outcome and disconnect events |
| FEAT-21.SPEC-004 | Calendar Connection & Sync Governance Rules | Phase 5 (Rule-Constraint Discovery — conditional logic, shared across specs) | The Validation & Limits field (one connection per household), the States field's Error and Offline-degraded behavior (auto-retry, additive-only), and the Primary Flows & Alternates field's disconnect behavior together form five-plus interacting rules shared by SPEC-001, SPEC-002, and SPEC-003 rather than duplicated in each |
| FEAT-21.SPEC-005 | Calendar Entry Content Derivation Rule | Phase 5 (Rule-Constraint Discovery — derivation) | The Data Notes field names a derived field ("calendar entry content, computed from Weekly Plan data") sourced from two other features' output (FEAT-03, FEAT-04); the derivation and the per-night entry-identity rule that makes "update, not duplicate" possible are non-trivial enough to cross the standalone-spec threshold rather than living inline in SPEC-003 |

## Entity-Lifecycle Coverage Matrix

This feature manages no entity through the create/read/update/delete lifecycle. Its sole Connected Entity, Weekly Plan, is read-only for this feature (product-features.md Connected Entities: "Weekly Plan (read)") — its creation, approval, and updates are owned entirely by other features (FEAT-03, FEAT-23, FEAT-04). Per Phase 3's rule for read-only Connected Entities, Weekly Plan is carried below as a Referenced Entity rather than given a full CRUD matrix; Planned Meal is included alongside it because the Data Notes field traces synced calendar-entry content down to individual dinners, not just the week as a whole.

The household's calendar connection itself (whether a household is currently connected, and to what) is not a Connected Entity named in product-features.md, so it is not given a Domain Entity CRUD table here. Its lifecycle — establishing a connection, reading its status, and tearing it down — is instead specified as part of FEAT-21.SPEC-002's Integration contract and governed by FEAT-21.SPEC-004's rules, consistent with Phase 4's guidance that a capability's connect/disconnect contract belongs to the Integration spec that owns it.

**Referenced Entities (read-only for this feature):**

| Entity | Read By | Context |
|--------|---------|---------|
| Weekly Plan | FEAT-21.SPEC-003 | The sync automation reads the household's current (and up to one week ahead) Weekly Plan to know which nights have a dinner to sync |
| Planned Meal | FEAT-21.SPEC-003, FEAT-21.SPEC-005 | Reads each dinner's night, recipe/dish name, and swap history so the sync automation can create or update the matching calendar entry with SPEC-005's derived content |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Maya connects the calendar capability | Establish the connection (rejecting a second simultaneous connection per SPEC-004's one-connection-per-household limit), then run an initial sync of the current week's dinners | Standalone Integration | FEAT-21.SPEC-002 |
| Maya disconnects the calendar capability | Tear down the connection; already-created calendar entries are left as-is on the external calendar, and no further sync occurs | Standalone Integration | FEAT-21.SPEC-002 |
| A dinner is proposed, picked, or approved into the Weekly Plan for a connected household | Sync automation creates the corresponding calendar entry via the Integration spec | Standalone Automation | FEAT-21.SPEC-003 |
| A meal swap (FEAT-04) changes an already-synced night | Sync automation updates the existing calendar entry for that night rather than creating a duplicate | Standalone Automation | FEAT-21.SPEC-003 |
| A sync attempt fails (connectivity or provider error) | Automatically retry; the in-app Weekly Plan is entirely unaffected in the meantime | Standalone Logic/Rule | FEAT-21.SPEC-004 |
| The external calendar reports the connection was revoked or lost outside the product | Integration spec's inbound event marks the household disconnected; in-app plan is unaffected | Standalone Integration (inbound event) | FEAT-21.SPEC-002 |
| A household is deleted (FEAT-18 account deletion cascade) | The calendar connection, if any, is disconnected as part of the cascade | Cross-feature | FEAT-18 responsibility, inbound event handled by FEAT-21.SPEC-002 |
| Calendar entry content is computed for a dinner | Derive night, dish name, and timing from the Planned Meal/Weekly Plan; no other household data (allergies, cost, notes) is placed on the entry | Standalone Logic/Rule | FEAT-21.SPEC-005 |
| Maya opens Calendar Connection Settings while never connected | Show the not-connected empty state with a "Connect calendar" call to action | Inline in triggering screen | FEAT-21.SPEC-001 |

## Shared Context

**Shared Entities:**
- Weekly Plan — read only, by FEAT-21.SPEC-003, to know which nights in the current (and up to one week ahead) plan need a synced entry. No fields are created or updated by this feature.
- Planned Meal — read only, by FEAT-21.SPEC-003 and FEAT-21.SPEC-005, for the night, recipe/dish name, and swap history that become calendar-entry content.

**Shared UI Patterns:**
- N/A — this feature defines exactly one screen (FEAT-21.SPEC-001), so there is no pattern to keep consistent across multiple screens within this feature. The screen is reached only from FEAT-01's Household Settings Hub, per the dependency map's navigation connection.

**Shared Validation:**
- FEAT-21.SPEC-004 defines the one-connection-per-household limit, the retry-on-failure behavior, and the additive-only/non-blocking guarantee; FEAT-21.SPEC-001, FEAT-21.SPEC-002, and FEAT-21.SPEC-003 all reference it rather than restating the rules.
- FEAT-21.SPEC-005 defines calendar-entry content derivation and the stable per-night entry identity that makes updates (rather than duplicates) possible; FEAT-21.SPEC-003 references it rather than deriving content itself.

## Internal Dependency Map

```
FEAT-21.SPEC-001 (Calendar Connection Settings) -> [Maya taps "Connect calendar"] -> FEAT-21.SPEC-002 (Family Calendar Integration)
FEAT-21.SPEC-001 (Calendar Connection Settings) -> [Maya taps "Disconnect"] -> FEAT-21.SPEC-002 (Family Calendar Integration)
FEAT-21.SPEC-002 (Family Calendar Integration) -> [enforces limit & retry rules from] -> FEAT-21.SPEC-004 (Calendar Connection & Sync Governance Rules)
FEAT-21.SPEC-002 (Family Calendar Integration) -> [connection established] -> FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync)
FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync) -> [reads content from] -> FEAT-21.SPEC-005 (Calendar Entry Content Derivation Rule)
FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync) -> [creates/updates entries through] -> FEAT-21.SPEC-002 (Family Calendar Integration)
FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync) -> [governed by non-blocking & retry rules in] -> FEAT-21.SPEC-004 (Calendar Connection & Sync Governance Rules)
FEAT-21.SPEC-002 (Family Calendar Integration) -> [inbound: connection revoked externally] -> FEAT-21.SPEC-004 (Calendar Connection & Sync Governance Rules) -> [household shown disconnected] -> FEAT-21.SPEC-001 (Calendar Connection Settings)
```

**Default Entry:** FEAT-21.SPEC-001 (Calendar Connection Settings) -- the only screen this feature owns, reached from FEAT-01's Household Settings Hub via the "connect calendar" navigation link (dependency map, Later phase).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-21.SPEC-001 | Inbound | FEAT-01 (Household Setup & Member Profiles) | Reached by tapping "connect calendar" in household settings | Maya opens Household Settings Hub and taps the calendar connection link |
| FEAT-21.SPEC-003 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Reads AI-generated and approved Planned Meals as sync source content | New week generated or approved for a connected household |
| FEAT-21.SPEC-003 | Inbound | FEAT-23 (Manual Weekly Planning) | Reads manually picked Planned Meals as sync source content | A dinner is picked or changed in a manually built week for a connected household |
| FEAT-21.SPEC-003 | Inbound | FEAT-04 (One-Tap Meal Swap) | A completed swap (FEAT-04.SPEC-004, Apply Meal Swap) is the sole trigger for updating an already-synced night's entry | Swap completes for a night that already has a synced calendar entry |
| FEAT-21.SPEC-002 | Inbound | FEAT-18 (Account & Data Management) | Household deletion cascades to disconnecting any active calendar connection | Household deletion completes |

## Non-Functional Notes

**Data volumes / growth:** At most one calendar entry per planned dinner per connected household, capped by the Weekly Plan's own limit of up to seven dinners plus leftover-lunch slots per week, planned at most one week ahead (Validation & Limits, Weekly Plan fields); this feature stores no growing dataset of its own beyond the single connection state per household (Validation & Limits: "one calendar connection per household at a time").

**Responsiveness:** States field: "Loading: N/A — sync happens automatically in the background once connected" — sync is not a user-waited-for interaction; the only user-facing wait is the connect action itself on FEAT-21.SPEC-001, which should complete or clearly show a retrying/error state rather than leaving Maya uncertain.

**Data sensitivity / privacy:** Calendar entries carry only the dish/night content that FEAT-21.SPEC-005 derives — household personal data about what the family eats (ASMP-14, ASMP-26), never sold or used for advertising. No allergy, cost, or member-identifying detail is placed on the entry; this keeps children's data (ASMP-26) out of the synced content even though the resulting entries are visible to Sam and both kid rows outside the product, on the calendar itself (Access field).

**Compliance flags:** N/A — this feature carries no health or financial data on the synced entries, and Riley (Operator, support) has no access to this feature at all (Access field), so no support-access exposure applies here.

## Non-Goals

- **Two-way sync from the external calendar back into the Weekly Plan** — Excluded per the feature's own Primary Flows & Alternates field, which describes only an outbound flow ("each night's planned dinner appears as a calendar entry") and per the States field's "this integration is additive only": nothing the household does on the external calendar changes the in-app plan.
- **Multiple simultaneous calendar connections per household** — Excluded per the Validation & Limits field: "One calendar connection per household at a time." A household must disconnect before connecting a different calendar capability.
- **Automatic cleanup of past calendar entries** — Intentional lifecycle decision: neither the feature entry nor the dependency map's Weekly Plan lifecycle (archived, not deleted, when a week ends) calls for removing already-created calendar entries; entries created for past weeks remain on the external calendar indefinitely, consistent with this feature's additive-only posture (States field).
- **In-app viewing of the synced calendar** — Excluded per the Data Notes field: "Displayed: N/A within the product beyond a connection status." The product shows only whether a calendar is connected (FEAT-21.SPEC-001); the calendar itself, and its entries, are viewed entirely outside the product on the household's own calendar.
- **Sam or either kid row connecting or disconnecting the calendar** — Excluded per the Access field, which reserves this action to Maya (Organiser) alone; Sam and both kid rows only ever see the resulting entries outside the product, and Riley (Operator, support) has no access to this feature at all.



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



# Integration Spec: Family Calendar Integration

## Overview

**Name:** Family Calendar Integration
**ID:** FEAT-21.SPEC-002
**Type:** Integration
**Purpose:** Defines the product's contract with the household's external family-calendar capability -- connecting and disconnecting, creating and updating per-night calendar entries, and receiving sync-outcome and disconnect events.
**Parent Feature:** FEAT-21 -- Family Calendar Sync

## Scope and Non-Goals

**In Scope:**
- Establishing and tearing down the household's connection to its calendar capability
- Sending calendar-entry content (night, dish name, timing) for the current and up to one week ahead's dinners
- Receiving and reacting to sync-outcome events (entry created, entry create/update failed) and disconnect events (connection revoked or lost externally)
- User-facing behavior when the calendar capability is slow, unavailable, or rejects a request
- Disclosure to Maya about exactly what data leaves the product when she connects

**Non-Goals:**
- Choosing the calendar vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for a specific calendar product, so this spec stays at the "family calendar" capability-category level throughout.
- Reading anything back from the external calendar -- excluded per the feature's Primary Flows & Alternates field ("this integration is additive only"); the product never imports events, availability, or any other data from the household's calendar.
- Determining what a synced entry contains -- owned by FEAT-21.SPEC-005 (Calendar Entry Content Derivation Rule); this spec only carries that already-derived content across the boundary.
- Deciding when to sync or re-sync a night -- owned by FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync); this spec only defines the create/update contract that automation drives.

## Capability Category

**Category:** Family calendar
**Dependency Source:** ASMP-37 -- "Online grocery-ordering and family-calendar capabilities (Later) -- Required only for Online Grocery Ordering Handoff (FEAT-20) and Family Calendar Sync (FEAT-21); the core product works fully without them." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Family calendar (ASMP-37, Later)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-21; Integration Specs: FEAT-21.SPEC-002)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision; BRIEF.md records no user mandate for a specific calendar provider.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Maya connects the household's calendar capability from settings, and sees whether it is currently connected | Sync the week's dinners | FEAT-21.SPEC-001 (Calendar Connection Settings) |
| Maya disconnects the calendar capability | Sync the week's dinners | FEAT-21.SPEC-001 (Calendar Connection Settings) |
| Each night's planned dinner appears as an entry on the household's own calendar once connected | Sync the week's dinners | FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync) |
| A meal swap updates the existing calendar entry for that night rather than creating a duplicate | Keep it current | FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync) |
| A connection revoked or lost outside the product is reflected as disconnected in-app | Sync the week's dinners (trust in the connection status shown) | FEAT-21.SPEC-001 (Calendar Connection Settings), FEAT-21.SPEC-004 (Calendar Connection & Sync Governance Rules) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Connection request | Household -- household reference only (no other Household fields) | Maya taps "Connect calendar" (FEAT-21.SPEC-001) | The capability needs to know which household is establishing a connection |
| Disconnect request | Household -- household reference only | Maya taps "Disconnect" and confirms (FEAT-21.SPEC-001) | Tells the capability to stop receiving further entry create/update requests for this household |
| Calendar entry content | Planned Meal -- night, and the entry content FEAT-21.SPEC-005 derives from recipe name (dish name) and timing; no other Planned Meal or Recipe field | A dinner is proposed, picked, or approved into a connected household's Weekly Plan, or a swap changes an already-synced night (FEAT-21.SPEC-003) | The capability needs the entry's date, title, and timing to create or update it |

Dietary Rule data (allergies, religious rules, dislikes), Rating, Pantry Item, Grocery List, recipe ingredients and steps, rough_cost, member names, and every other Household or Member Profile field never leave the product through this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Connection established | The capability confirms a connect request | Calendar Connection (feature-local, governed by FEAT-21.SPEC-004) -- status set Connected |
| Connection rejected | The capability declines a connect request | Calendar Connection -- status remains Not Connected |
| Entry create/update succeeded | The capability confirms a calendar entry was created or updated | That night's Per-Night Calendar Sync Record (governed by FEAT-21.SPEC-004's field rules and FEAT-21.SPEC-005's entry-identity rule) -- status set Synced, synced_content_identity updated, and that night's own retry_count reset to 0 |
| Entry create/update failed | The capability reports it could not create or update an entry | That night's Per-Night Calendar Sync Record -- retry_count incremented and last_attempt_outcome set to Failed, per FEAT-21.SPEC-004; status becomes Failing only if that night's own retry_count reaches the ceiling. The household-level Calendar Connection.retry_count is a separate, cycle-based counter that FEAT-21.SPEC-003 updates at the end of each full sync run (FEAT-21.SPEC-004), not on this single event; no Weekly Plan or Planned Meal field changes either way |
| Connection revoked or lost externally | The capability reports the connection is no longer valid (or the household-level retry ceiling is exhausted per FEAT-21.SPEC-004) | Calendar Connection -- status set Disconnected, disconnect_reason set to Revoked-Externally or Retry-Ceiling-Exhausted as applicable, externally_disconnected_acknowledged set to false |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Connection established | The capability confirms a connect request Maya initiated | Calendar Connection status set to Connected; retry counter reset | FEAT-21.SPEC-001 shows "Connected to your calendar"; an initial sync of the current week's dinners begins in the background with no further user-facing wait | FEAT-21.SPEC-001, FEAT-21.SPEC-003 (initial sync fires), FEAT-21.SPEC-004 |
| Connection rejected | The capability declines a connect request (e.g., the household already has an active connection elsewhere on the capability's side, or the request otherwise cannot be completed) | Calendar Connection status remains Not Connected | FEAT-21.SPEC-001 shows "Couldn't connect your calendar. Try again." | FEAT-21.SPEC-001, FEAT-21.SPEC-004 |
| Entry create/update succeeded | The capability confirms a calendar entry was created or updated for a given night | That night's Per-Night Calendar Sync Record is marked Synced with the current entry content's identity (FEAT-21.SPEC-005) and that night's own retry_count reset to 0 (FEAT-21.SPEC-004) | No direct feedback -- sync is background per the feature's States field ("Loading: N/A -- sync happens automatically in the background once connected") | FEAT-21.SPEC-003 |
| Entry create/update failed | A create or update attempt does not complete (connectivity or provider error) | That night's own Per-Night Calendar Sync Record retry_count increments per FEAT-21.SPEC-004 -- independently of any other night's outcome; no Weekly Plan or Planned Meal change | No user feedback -- the in-app plan is entirely unaffected (feature's States field, Error) | FEAT-21.SPEC-003, FEAT-21.SPEC-004 |
| Connection revoked or lost externally | The capability reports the connection is no longer valid, or the household-level retry ceiling defined by FEAT-21.SPEC-004 is exhausted (a consecutive run of fully-failed sync cycles, not any single night's own failures) | Calendar Connection status set to Disconnected, with disconnect_reason and externally_disconnected_acknowledged set per FEAT-21.SPEC-004 | FEAT-21.SPEC-001 shows "Your calendar connection ended outside the app. Reconnect to keep syncing." on next visit | FEAT-21.SPEC-001, FEAT-21.SPEC-004 |
| Household deletion cascade | FEAT-18.SPEC-008 (Household Deletion Processing) reaches its calendar-disconnect step (Processing Logic step 5) during the household deletion cascade -- not an event from the external capability itself, but an internal cross-feature trigger this spec is assigned to handle per the dependency map's External Touchpoints and Cross-Feature Touchpoints | Any active Calendar Connection for the household is disconnected as part of the cascade | None -- the household and its members no longer exist to see feedback | FEAT-21.SPEC-004 |

## Degradation Behavior

FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync) is a background automation with no screen surface of its own; its create/update requests to this capability never block or degrade any user-facing screen -- failures there are silent and retried per FEAT-21.SPEC-004, consistent with the feature's additive-only, non-blocking guarantee. Only FEAT-21.SPEC-001 (the connect/disconnect action) has a direct, user-waited-for request to this capability, so it is the only affected screen below.

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-21.SPEC-001 (Calendar Connection Settings) | The "Connect calendar" button remains in its loading state; after 10 seconds a note appears: "Still connecting -- this is taking longer than usual." Maya can still navigate away; the request continues in the background. | The button returns to Not Connected with "Couldn't connect your calendar. Try again." Maya's household plan is entirely unaffected and every other Household Settings row remains usable. | The button returns to Not Connected with "Couldn't connect your calendar. Try again." (the same message as capability-down, since the feature defines no separate per-reason wording for a connect rejection). |

## Consent and Disclosure

- **First connection disclosure** -- The first time Maya taps "Connect calendar", a notice appears before the connection request is sent: "Connecting shares your week's dinners -- the night and dish name only -- with your calendar. Nothing else about your household (allergies, cost, or who's eating) is shared." Options: "Continue" and "Cancel". Shown once per household's first connection; a household that disconnects and reconnects later does not see it again, since the same disclosure already covers what future syncing shares.
- **What is never shared** -- Dietary Rule data, Rating, Pantry Item, Grocery List content, recipe ingredients and steps, rough_cost, and every member's name or profile detail stay inside the product; this boundary is stated in the disclosure notice and never varies by household.
- **Disconnect confirmation as a disclosure moment** -- FEAT-21.SPEC-001's disconnect confirmation dialog doubles as a data-boundary disclosure: it tells Maya that already-created entries remain on the external calendar (this integration never deletes what it created) and that only future syncing stops.
- **Other household members are told through the same disclosure, not a separate one** -- Sam and both Jordan rows never interact with this integration directly (only Maya connects or disconnects, per the Access field); the first-connection disclosure Maya sees already covers the only data that becomes visible to them, since they see the resulting entries solely outside the product, on the household's own calendar (feature's Non-Functional Notes, Data sensitivity/privacy).

## Edge Cases

- **A sync-outcome event arrives for a night whose Planned Meal has since been removed (e.g., a safety-concern removal, FEAT-02)** -- The event is recorded against that night's sync record with no further action; FEAT-21.SPEC-003's next evaluation of that night finds no dinner to sync and takes no further create/update action. No user feedback fires (additive-only).
- **The same entry create/update succeeded event is delivered twice** -- The second delivery changes nothing: the night's sync record is already marked synced with the same entry-content identity, so no duplicate calendar entry is requested and no duplicate state change occurs.
- **Events arrive out of order (a create/update failure for an older content version arrives after a later success for the same night)** -- The per-night sync record reflects the most recent successful sync outcome; a late-arriving failure for stale content does not revert an already-successful, newer sync.
- **The capability goes down mid-connect, before confirmation arrives** -- If connection establishment was not confirmed, Calendar Connection stays Not Connected -- no half-connected state, and FEAT-21.SPEC-001 shows the capability-down message.
- **A disconnect request is sent while a create/update request for the same household is still in flight** -- The in-flight create/update is allowed to complete or fail on its own; its outcome event is accepted and recorded, but no further create/update requests are sent once the disconnect is confirmed (FEAT-21.SPEC-004).
- **Connection revoked externally while Maya is actively viewing FEAT-21.SPEC-001 in the Connected state** -- The screen updates live to the Not Connected state with the "ended outside the app" note, without requiring Maya to leave and return.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-001 (Calendar Connection Settings) | Triggered by (inbound) | Connect and disconnect actions initiate requests through this integration |
| FEAT-21.SPEC-001 (Calendar Connection Settings) | Affects (outbound) | Connection status, degradation states, and the first-connection disclosure surface here |
| FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync) | Triggers (outbound) | Connection established fires this automation's initial sync; this automation drives every entry create/update request through this integration |
| FEAT-21.SPEC-004 (Calendar Connection & Sync Governance Rules) | References (inbound) | Enforces the one-connection-per-household limit, the retry ceiling, and what disconnecting does and does not affect |
| FEAT-21.SPEC-005 (Calendar Entry Content Derivation Rule) | References (inbound) | Defines the entry content and per-night identity this integration carries across the boundary |
| FEAT-18.SPEC-008 (Household Deletion Processing) | Triggered by (inbound) | Household deletion cascades to disconnecting any active calendar connection |

## Analytics and Success Signals

- **calendar_connected** (household reference only) -- N/A -- no metric in success-metrics.md names Family Calendar Sync as its Connected Feature; retained to observe how often households successfully connect.
- **calendar_connect_rejected** (-) -- N/A -- same reason as above.
- **calendar_sync_completed** (outcome: entry created / entry updated) -- N/A -- same reason as above; retained so the sync automation's real-world success rate is observable.
- **calendar_sync_failed** (attempt count so far) -- N/A -- same reason as above; retained so the non-blocking retry guarantee's actual exercise rate is observable.
- **calendar_disconnected** (reason: matches disconnect_reason -- user-initiated / revoked-externally / retry-ceiling-exhausted / household-deletion-cascade) -- N/A -- same reason as above.

## Acceptance Criteria

**FEAT-21.SPEC-002-AC-01:** Given Maya taps "Connect calendar" on FEAT-21.SPEC-001, when the capability confirms the connection, then Calendar Connection status becomes Connected and an initial sync of the current week's dinners begins in the background (FEAT-21.SPEC-003).

**FEAT-21.SPEC-002-AC-02:** Given Maya taps "Disconnect" and confirms on FEAT-21.SPEC-001, when the disconnect request completes, then Calendar Connection status becomes Not Connected and no further entry create/update requests are sent for that household.

**FEAT-21.SPEC-002-AC-03:** Given a connected household has a dinner newly approved into its Weekly Plan, when FEAT-21.SPEC-003 drives this integration to create the entry and the capability confirms it, then that night's sync record is marked synced and no user-facing feedback is shown.

**FEAT-21.SPEC-002-AC-04:** Given an already-synced night's dinner is swapped, when FEAT-21.SPEC-003 drives this integration to update the existing entry and the capability confirms it, then the same entry is updated rather than a new one being created.

**FEAT-21.SPEC-002-AC-05:** Given a create/update attempt fails due to a connectivity or provider error, when the failure event arrives, then that specific night's own Per-Night Calendar Sync Record retry_count increments per FEAT-21.SPEC-004 (unaffected by any other night's outcome), no user-facing feedback appears, and the in-app Weekly Plan is unaffected.

**FEAT-21.SPEC-002-AC-06:** Given the household's connection is revoked or lost outside the product, when this integration detects it (directly, or via FEAT-21.SPEC-004's household-level exhausted retry ceiling), then Calendar Connection status becomes Disconnected with disconnect_reason set to Revoked-Externally or Retry-Ceiling-Exhausted and externally_disconnected_acknowledged set to false, and FEAT-21.SPEC-001 shows the "ended outside the app" note on next visit.

**FEAT-21.SPEC-002-AC-07:** Given a household is deleted (FEAT-18.SPEC-008), when the deletion cascade runs, then any active Calendar Connection for that household is disconnected as part of it.

**FEAT-21.SPEC-002-AC-08:** Given Maya taps "Connect calendar" and the capability is slow to respond, when 10 seconds pass without confirmation, then FEAT-21.SPEC-001 shows "Still connecting -- this is taking longer than usual." while the request continues.

**FEAT-21.SPEC-002-AC-09:** Given Maya taps "Connect calendar" while the capability is down, when the request cannot be sent, then FEAT-21.SPEC-001 shows "Couldn't connect your calendar. Try again." and the household remains Not Connected.

**FEAT-21.SPEC-002-AC-10:** Given Maya taps "Connect calendar" and the capability rejects the request, when the rejection is received, then FEAT-21.SPEC-001 shows "Couldn't connect your calendar. Try again." and the household remains Not Connected.

**FEAT-21.SPEC-002-AC-11:** Given Maya has never connected a calendar for her household before, when she taps "Connect calendar" for the first time, then the disclosure notice appears stating exactly what is shared (night and dish name only) before any request is sent, with "Continue" and "Cancel" options.

**FEAT-21.SPEC-002-AC-12:** Given the same entry create/update succeeded event is delivered twice for the same night, when the second delivery arrives, then no duplicate calendar entry is requested and the night's sync record is unchanged.

**FEAT-21.SPEC-002-AC-13:** Given a sync-outcome event arrives for a night whose Planned Meal has since been removed by a safety concern (FEAT-02), when the event is processed, then it is recorded with no further action and no user feedback fires.

**FEAT-21.SPEC-002-AC-14:** Given the household's connection is revoked externally while Maya is viewing FEAT-21.SPEC-001 in the Connected state, when the revoked-connection event arrives, then the screen updates live to the Not Connected state with the "ended outside the app" note.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 5 | 5 |
| Inbound Events | 6 | 6 |
| Degradation Paths | 3 | 3 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 6 | 6 |



# Automation Spec: Weekly Dinner Calendar Sync

## Overview

**Name:** Weekly Dinner Calendar Sync
**ID:** FEAT-21.SPEC-003
**Type:** Automation
**Purpose:** Watches the Weekly Plan for dinners to sync and for swaps that change an already-synced night, and drives FEAT-21.SPEC-002 to create or update the matching calendar entry.
**Parent Feature:** FEAT-21 -- Family Calendar Sync

## Scope and Non-Goals

**In Scope:**
- Running an initial sync of the current week's dinners when a household's calendar connection is established
- Detecting a dinner newly proposed, picked, or approved into a connected household's Weekly Plan and creating its calendar entry
- Detecting a swap that changes an already-synced night and updating the existing entry instead of creating a duplicate
- Reading only the current week and up to one week ahead of the Weekly Plan, consistent with the product's own planning-ahead limit
- Retrying and remaining silent to the user on failure, per FEAT-21.SPEC-004

**Non-Goals:**
- Determining what a synced entry contains -- owned by FEAT-21.SPEC-005 (Calendar Entry Content Derivation Rule); this automation only calls that derivation and passes the result to FEAT-21.SPEC-002.
- Sending the create/update request itself, or handling the capability's response -- owned by FEAT-21.SPEC-002 (Family Calendar Integration); this automation only decides when a create or update is needed.
- Syncing leftover-lunch Planned Meals -- excluded per the feature's Key Capabilities ("Sync the week's dinners"), which the Brief scopes to dinners only; product-features.md's Connected Entities for this feature name only the dinner-bearing Weekly Plan, and leftover lunches are a distinct meal_kind this feature does not read.
- Removing a calendar entry when its Planned Meal is deleted or a week is archived -- excluded per the feature's Non-Goals ("Automatic cleanup of past calendar entries"); this automation only creates and updates, it never deletes an external entry.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Calendar connection established | FEAT-21.SPEC-002 (Family Calendar Integration) | Fires once, immediately after a household's connect request is confirmed | Household reference; the household's current week's Weekly Plan and its dinner Planned Meals |
| Dinner proposed, picked, or approved | FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation), FEAT-03.SPEC-005 (Auto-Adoption at Week Start), FEAT-03.SPEC-008 (Plan Approval Authorization Rule), FEAT-23.SPEC-006 (Apply Manual Pick) | Fires whenever any of these specs adds or confirms a dinner Planned Meal in a connected household's current or one-week-ahead Weekly Plan | The affected Planned Meal (night, recipe/dish name), the connected household reference |
| Meal swap completes | FEAT-04.SPEC-004 (Apply Meal Swap) | Fires when a completed swap changes the recipe on an already-synced night in a connected household | The affected Planned Meal's new recipe/dish name, its night, the connected household reference, and (from FEAT-21.SPEC-005) the existing entry identity for that night |

## Processing Logic

1. On any trigger, first confirm the household is currently connected (Calendar Connection status Connected, per FEAT-21.SPEC-004); if not connected, take no further action (no-action path).
2. Read the connected household's Weekly Plan for the current week and, when already generated or built, the week ahead -- no further ahead, matching the product's own one-week planning-ahead limit.
3. For each night in that window that has a dinner Planned Meal:
   a. Derive the calendar entry's content and per-night identity key via FEAT-21.SPEC-005 (Calendar Entry Content Derivation Rule).
   b. Check whether that night already has a synced entry (an existing per-night sync record with a matching identity key).
   c. If no synced entry exists for that night, request FEAT-21.SPEC-002 create a new calendar entry with the derived content.
   d. If a synced entry exists and the derived content differs from what was last synced (e.g., a swap changed the dish name), request FEAT-21.SPEC-002 update the existing entry with the new content.
   e. If a synced entry exists and the derived content is unchanged, take no action for that night.
4. For a night that no longer has a dinner Planned Meal (e.g., cleared or removed by a safety concern) but previously had a synced entry, take no action -- per this feature's additive-only posture and Non-Goals, no delete request is ever sent.
5. Record the outcome of each night's evaluation (created, updated, no action, or failed) against that night's Per-Night Calendar Sync Record, updating that record's own retry_count and status per FEAT-21.SPEC-004 (a failure increments only that night's own retry_count; a success resets only that night's own retry_count to 0 -- neither ever touches another night's record).
6. At the end of the run, reconcile the household-level counter (Calendar Connection.retry_count, FEAT-21.SPEC-004): if at least one night in this run resulted in Entry created or Entry updated, reset it to 0; if every night evaluated in this run resulted in Sync failure (and at least one night was evaluated), increment it by 1. A run in which every night's outcome was No action needed (nothing to sync, nothing changed) neither increments nor resets it.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Entry created | A night has a dinner and no synced entry yet exists for it | That night's Per-Night Calendar Sync Record is created and marked Synced once FEAT-21.SPEC-002 confirms; contributes to resetting the household-level retry_count at end-of-run (FEAT-21.SPEC-004) | None -- sync is a background process per the feature's States field | FEAT-21.SPEC-002 |
| Entry updated | A night already has a synced entry and its derived content has changed (typically after a swap) | That night's Per-Night Calendar Sync Record's content identity is updated once FEAT-21.SPEC-002 confirms; contributes to resetting the household-level retry_count at end-of-run (FEAT-21.SPEC-004) | None | FEAT-21.SPEC-002 |
| No action needed | The household is not connected, the derived content for a synced night is unchanged, or a night has no dinner to sync | None | None -- a silent, logged no-op | -- |
| Sync failure | FEAT-21.SPEC-002 reports the create or update request did not complete | That specific night's own Per-Night Calendar Sync Record retry_count increments per FEAT-21.SPEC-004 -- never resets from another night's outcome; if every night evaluated in this run failed, the household-level Calendar Connection.retry_count also increments once at end-of-run (FEAT-21.SPEC-004); no Weekly Plan or Planned Meal change | None -- non-blocking; the in-app plan is entirely unaffected (feature's States field, Error) | FEAT-21.SPEC-002, FEAT-21.SPEC-004 |

## Data Model

**Reads:** Weekly Plan (read-only, per feature-overview.md's Entity-Lifecycle Coverage Matrix) -- the household's current and up-to-one-week-ahead week to know which nights have a dinner. Planned Meal (read-only) -- night, recipe/dish name, and swap_history for each dinner, to detect and derive changed content. Also reads each evaluated night's existing Per-Night Calendar Sync Record (feature-local, governed by FEAT-21.SPEC-004's field rules and FEAT-21.SPEC-005's entry-identity rule) to know its current status, retry_count, and synced_content_identity.
**Creates:** A Per-Night Calendar Sync Record (feature-local, not a dependency-map entity) tracking which night maps to which external calendar entry, governed by FEAT-21.SPEC-005's entry-identity rule and FEAT-21.SPEC-004's field rules.
**Updates:** Each evaluated night's own Per-Night Calendar Sync Record (status, synced_content_identity, retry_count, last_attempt_outcome, per FEAT-21.SPEC-004); at the end of each run, the household-level Calendar Connection.retry_count (FEAT-21.SPEC-004) per this spec's end-of-run reconciliation rule (Processing Logic, step 6).
**Deletes:** None -- per this feature's additive-only posture, no calendar entry or sync record is ever deleted by this automation; a night that stops having a dinner simply stops being evaluated.

## Business Rules

- This automation never triggers for a household without an active calendar connection (FEAT-21.SPEC-004) -- a disconnected household's Weekly Plan changes are read by no calendar-sync process at all.
- The additive-only, non-blocking guarantee (FEAT-21.SPEC-004) applies throughout: nothing this automation does, succeeds, or fails ever changes the Weekly Plan, Planned Meal, or any other in-app data outside this feature's own Per-Night Calendar Sync Records and the household-level Calendar Connection.
- Per-night and household-level retry tracking are kept strictly independent (FEAT-21.SPEC-004): a night's own retry_count is touched only by that night's own outcome, never by another night's; the household-level retry_count is touched only once per run, at end-of-run, based on whether the whole run had any success -- so one persistently failing night never gets its failure streak erased just because other nights in the same household are syncing fine, and one lucky night's success never masks every other night's continued failure at the household level.
- A night's calendar entry is identified by the stable per-night key FEAT-21.SPEC-005 defines, not by which recipe currently occupies it -- this is what makes an update possible instead of a duplicate create.
- Only the current week and up to one week ahead are ever evaluated, consistent with the product's own planning-ahead limit (dependency map, Weekly Plan lifecycle).
- Leftover-lunch Planned Meals are never read or synced by this automation (Scope and Non-Goals).
- Sam and both Jordan rows never see this automation's activity in-app -- they experience only the entries it produces, outside the product, on the household's own calendar (feature's Access field); this automation itself has no in-app surface for any role.

## Edge Cases

- **Two swaps on different nights of the same connected household complete within moments of each other** -- Each swap's trigger evaluates and syncs only its own night; the two runs act on different per-night sync records and do not interfere with each other.
- **Concurrent trigger firing (a plan-approval event and a swap-completion event for the same household arrive at effectively the same time, but for different nights)** -- Each trigger's run reads the current Weekly Plan independently and updates only the night it concerns; no shared state is contended because each night has its own sync record.
- **A trigger fires for a given night while a previous sync run for that same night is still in flight** -- The later trigger's evaluation for that night waits for the in-flight request's outcome before deciding create vs. update, so a second request is never sent for the same night while the first is still pending; this prevents a duplicate create.
- **A household disconnects while a sync run is in flight** -- Any request already sent to FEAT-21.SPEC-002 is allowed to complete or fail on its own and its outcome is still recorded; no new create/update requests are started once the disconnect is confirmed (FEAT-21.SPEC-004).
- **A dinner is swapped back to its original recipe within the same week** -- The existing synced entry for that night is updated again to reflect the reverted content; no second entry is ever created for the same night.
- **A safety-concern removal (FEAT-02) clears a previously synced night's dinner** -- No delete or update request is sent for that night; per the additive-only posture, the already-created external entry is left exactly as it is until a new dinner is placed in that slot, at which point the existing entry is updated rather than a new one created.
- **One night in a household fails every sync attempt for several consecutive runs while every other night in the same run succeeds** -- Each run resets the household-level Calendar Connection.retry_count to 0 (since at least one night succeeded), while the failing night's own Per-Night Calendar Sync Record retry_count keeps climbing, run over run, undisturbed by the other nights' success; the household never approaches the household-level disconnect ceiling on account of this one night.
- **Every night evaluated in a run fails (e.g., the connection itself has gone bad)** -- The household-level Calendar Connection.retry_count increments once for the whole run, in addition to each individual night's own retry_count incrementing; if this repeats until the household-level ceiling is reached, FEAT-21.SPEC-004 moves the household to Disconnected regardless of any individual night's own count.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-002 (Family Calendar Integration) | Triggered by (inbound) | The "connection established" inbound event fires this automation's initial sync |
| FEAT-21.SPEC-002 (Family Calendar Integration) | Triggers (outbound) | This automation requests every entry create/update through this integration |
| FEAT-21.SPEC-005 (Calendar Entry Content Derivation Rule) | References (inbound) | Supplies the derived entry content and the stable per-night identity key this automation checks against |
| FEAT-21.SPEC-004 (Calendar Connection & Sync Governance Rules) | References (inbound) | Governs the additive-only, non-blocking guarantee and retry behavior this automation follows on failure |
| FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation), FEAT-03.SPEC-005 (Auto-Adoption at Week Start), FEAT-03.SPEC-008 (Plan Approval Authorization Rule) | Triggered by (inbound) | A dinner entering the plan through AI generation, auto-adoption, or approval fires a sync evaluation for that night |
| FEAT-23.SPEC-006 (Apply Manual Pick) | Triggered by (inbound) | A manually picked dinner fires a sync evaluation for that night |
| FEAT-04.SPEC-004 (Apply Meal Swap) | Triggered by (inbound) | A completed swap on an already-synced night fires an update evaluation for that night |

## Analytics and Success Signals

- **calendar_sync_evaluated** (outcome: created / updated / no_action / failed; night) -- N/A -- no metric in success-metrics.md names Family Calendar Sync as its Connected Feature; retained so the sync automation's real-world behavior is observable.
- **calendar_sync_completed** (outcome: entry_created / entry_updated) -- N/A -- same reason as above; this event also feeds FEAT-21.SPEC-002's inbound-event handling on the integration side.
- **calendar_sync_failed** (retry attempt number) -- N/A -- same reason as above; retained so the non-blocking retry guarantee's actual exercise rate is observable.

## Acceptance Criteria

**FEAT-21.SPEC-003-AC-01:** Given Maya's household just had its calendar connection established (FEAT-21.SPEC-002), when this automation runs its initial sync, then every dinner in the current week's Weekly Plan gets a newly created calendar entry.

**FEAT-21.SPEC-003-AC-02:** Given a connected household's AI-generated plan is approved (FEAT-03.SPEC-008), when this automation evaluates the newly approved dinners, then each one without an existing synced entry gets a newly created calendar entry.

**FEAT-21.SPEC-003-AC-03:** Given a connected household manually picks a dinner for an empty night (FEAT-23.SPEC-006), when this automation evaluates that night, then a new calendar entry is created for it.

**FEAT-21.SPEC-003-AC-04:** Given an already-synced night's dinner is swapped (FEAT-04.SPEC-004), when this automation evaluates that night, then the existing calendar entry is updated with the new dish name rather than a new entry being created.

**FEAT-21.SPEC-003-AC-05:** Given a night's derived entry content is unchanged since its last successful sync, when this automation re-evaluates that night, then no create or update request is sent.

**FEAT-21.SPEC-003-AC-06:** Given a household has no active calendar connection, when a dinner is proposed, picked, or approved into its plan, then this automation takes no action for that household.

**FEAT-21.SPEC-003-AC-07:** Given a create/update request to FEAT-21.SPEC-002 fails for a given night, when the failure is reported, then that night's own Per-Night Calendar Sync Record retry_count increments, no user feedback appears, and the in-app Weekly Plan for that household is unaffected.

**FEAT-21.SPEC-003-AC-13:** Given one night fails its create/update attempt while every other night evaluated in the same run succeeds, when the run completes, then that night's own retry_count increments while the household-level Calendar Connection.retry_count resets to 0 (per FEAT-21.SPEC-004).

**FEAT-21.SPEC-003-AC-14:** Given every night evaluated in a run fails its create/update attempt, when the run completes, then the household-level Calendar Connection.retry_count increments by 1 in addition to each night's own retry_count incrementing.

**FEAT-21.SPEC-003-AC-15:** Given a night's own retry_count has been climbing across several runs because every other night in those runs kept succeeding, when a subsequent run again has at least one other night succeed, then that failing night's own retry_count is unaffected by the other nights' success and continues from where it left off.

**FEAT-21.SPEC-003-AC-08:** Given this automation reads a connected household's plan, when it looks for dinners to sync, then it evaluates only the current week and up to one week ahead, never further out.

**FEAT-21.SPEC-003-AC-09:** Given a leftover-lunch Planned Meal exists alongside a dinner in the same connected household's plan, when this automation evaluates the week, then only the dinner is synced and the leftover lunch is never read or synced.

**FEAT-21.SPEC-003-AC-10:** Given two swaps complete on different nights of the same connected household within moments of each other, when both trigger this automation, then each night's sync is evaluated and applied independently with no interference between them.

**FEAT-21.SPEC-003-AC-11:** Given a sync run for a given night is still in flight, when a new trigger fires for that same night before the first run's outcome is known, then the second evaluation waits for the first run's outcome rather than sending a second create request for the same night.

**FEAT-21.SPEC-003-AC-12:** Given a safety-concern removal clears a previously synced night's dinner, when this automation next evaluates the week, then no delete or update request is sent for that night and the already-created external entry is left unchanged.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 4 | 4 |
| Business Rules | 7 | 7 |
| Edge Cases | 8 | 8 |



# Logic/Rule Spec: Calendar Connection & Sync Governance Rules

## Overview

**Name:** Calendar Connection & Sync Governance Rules
**ID:** FEAT-21.SPEC-004
**Type:** Logic/Rule
**Purpose:** Governs the one-connection-per-household limit, retry-on-failure behavior at both the household and per-night level, the additive-only/non-blocking guarantee for the in-app plan, what disconnecting does and does not affect, and how an externally-caused disconnect is distinguished from one Maya initiated herself.
**Parent Feature:** FEAT-21 -- Family Calendar Sync
**Governed Entity:** Calendar Connection; Per-Night Calendar Sync Record

## Scope and Non-Goals

**In Scope:**
- Field rules for the Calendar Connection's own state (status, disconnect reason, household-level retry tracking, timestamps)
- Field rules for the Per-Night Calendar Sync Record's state (per-night retry tracking, per-night sync status) and how it reconciles with the household-level retry ceiling
- The one-connection-per-household limit and how a second, simultaneous connect attempt is refused
- Retry-on-failure behavior for both the connect action and background sync attempts, at both the household level (connection health) and the per-night level (a specific night's entry)
- The additive-only, non-blocking guarantee: what a failed sync, a lost connection, or never connecting at all does and does not affect
- What disconnecting does and does not affect (already-created entries, further syncing), and distinguishing why a household ended up disconnected
- Authorization rules for every action on the Calendar Connection, per role
- Handling the household-deletion cascade from FEAT-18

**Non-Goals:**
- Deriving calendar entry content or the per-night entry identity -- owned by FEAT-21.SPEC-005 (Calendar Entry Content Derivation Rule); this spec governs the connection and per-night sync record's own state, not entry content.
- Sending connect, disconnect, create, or update requests to the calendar capability -- owned by FEAT-21.SPEC-002 (Family Calendar Integration), which calls these rules but performs the actual requests.
- Deciding which nights need a sync evaluation, and running the sync cycle itself -- owned by FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync), which calls this spec's retry and non-blocking rules (including updating the fields this spec defines) but owns the per-night evaluation logic and cycle timing itself.
- Displaying the connect/disconnect controls this spec authorizes, or the per-night sync record's detail -- owned by FEAT-21.SPEC-001 (Calendar Connection Settings), which enforces these authorization rules on screen but does not define them and never surfaces per-night detail (Non-Goals, that spec).

## Governed Entity

This spec governs two feature-local entities: the household-level **Calendar Connection**, and the per-night **Per-Night Calendar Sync Record** it reconciles with. Neither is a Connected Entity in product-features.md and neither carries a Domain Entity CRUD table in the dependency map (feature-overview.md, Entity-Lifecycle Coverage Matrix); both lifecycles are specified here and in FEAT-21.SPEC-002's Integration contract.

### Calendar Connection

**Entity:** Calendar Connection
**Source:** FEAT-21.SPEC-002 (Family Calendar Integration) and this spec -- feature-local entity, one record per household.

| Field | Data Type | Description |
|-------|-----------|-------------|
| household | reference | The one Household this connection belongs to |
| status | enum | Not Connected, Connecting, Connected, or Disconnected (set when revoked or lost externally, torn down by Maya, or cascaded from household deletion) |
| connected_at | date | When the current or most recent connection was established; null while Not Connected |
| disconnected_at | date | When the connection was last torn down (by Maya or externally); null if never yet disconnected |
| disconnect_reason | enum | Why the connection is (or was last) Disconnected: N/A (never disconnected, or currently Connected/Connecting), User-Initiated, Revoked-Externally, Retry-Ceiling-Exhausted, or Household-Deletion-Cascade |
| externally_disconnected_acknowledged | boolean | Whether Maya has been shown and dismissed the "ended outside the app" note (FEAT-21.SPEC-001) since the current disconnect_reason was set; irrelevant (treated as true) when disconnect_reason is N/A or User-Initiated |
| retry_count | number | The household-level, cycle-based failure counter: the number of consecutive FEAT-21.SPEC-003 sync runs in which every evaluated night failed, since the last run with at least one success or the last successful connect. Distinct from, and never reset by, any single night's own retry_count on its Per-Night Calendar Sync Record (see Cross-Field Rules, "Household- and per-night-level retry counters are independent") |
| last_sync_outcome | enum | Success, Failed, or N/A (no sync attempted yet) -- the outcome of the most recent sync run at the household level (Success if at least one night succeeded, Failed if every evaluated night failed) |

### Per-Night Calendar Sync Record

**Entity:** Per-Night Calendar Sync Record
**Source:** FEAT-21.SPEC-002 and FEAT-21.SPEC-003 -- feature-local entity, one record per night that has ever had a dinner evaluated for sync in a connected household. Its per-night identity key is derived by FEAT-21.SPEC-005; this spec governs its retry and status fields only.

| Field | Data Type | Description |
|-------|-----------|-------------|
| household | reference | The Household this record belongs to |
| night | date | The specific night this record tracks |
| entry_identity_key | derived | The stable per-night identity key from FEAT-21.SPEC-005, used to match this record to the correct external calendar entry across updates |
| synced_content_identity | derived | The content identity (per FEAT-21.SPEC-005) last successfully synced for this night; null if this night has never been successfully synced |
| status | enum | Pending (no sync attempted yet, or content changed since the last success), Synced (current content confirmed on the external calendar), or Failing (this night's own retry_count has reached the ceiling; see Cross-Field Rules) |
| retry_count | number | The number of consecutive failed create/update attempts for this specific night, since this night's own last successful create or update. Reset to 0 only by this night's own success -- never by another night's success, and never by the household-level counter resetting |
| last_attempt_outcome | enum | Success, Failed, or N/A (no create/update attempted yet for this night) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-21.SPEC-001 | Calendar Connection Settings | On screen entry (authorization) and on connect/disconnect action attempt; reads status, disconnect_reason, and externally_disconnected_acknowledged to choose Connected / Not Connected / Externally Disconnected |
| FEAT-21.SPEC-002 | Family Calendar Integration | On every connect, disconnect, and per-night sync-outcome event -- enforces the one-connection-per-household limit; updates the Calendar Connection's status, disconnect_reason, and externally_disconnected_acknowledged; updates each Per-Night Calendar Sync Record's status, retry_count, and last_attempt_outcome on that night's own outcome |
| FEAT-21.SPEC-003 | Weekly Dinner Calendar Sync | On every sync evaluation -- checks Calendar Connection status before evaluating a household; reads and writes each Per-Night Calendar Sync Record it evaluates; at the end of each sync run, updates the household-level Calendar Connection.retry_count per the cycle-based rule (Cross-Field Rules) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| household | Must reference exactly one Household; system-set, never entered by any role | Always | On create (first connect attempt) | N/A -- system-computed, not a user-entered field | Yes |
| status | Must be one of Not Connected, Connecting, Connected, Disconnected; transitions only Not Connected -> Connecting -> Connected -> Disconnected (or Connecting -> Not Connected on a rejected connect) | Always | On every connect, disconnect, or inbound event from FEAT-21.SPEC-002 | N/A -- system-computed; no direct edit path exists for any role | Yes |
| connected_at | Set only when status transitions to Connected; cleared (null) only when status returns to Not Connected | Always | On connection established | N/A -- system-computed | Yes |
| disconnected_at | Set only when status transitions to Disconnected | Always | On disconnect (user-initiated or external) | N/A -- system-computed | Yes |
| disconnect_reason | Must be one of N/A, User-Initiated, Revoked-Externally, Retry-Ceiling-Exhausted, Household-Deletion-Cascade; set in the same transition that sets status to Disconnected, and reset to N/A the moment status leaves Disconnected (i.e., on the next successful reconnect) | Always | On every transition to or out of Disconnected | N/A -- system-computed, not user-entered | Yes |
| externally_disconnected_acknowledged | Must be a boolean; set to false in the same transition that sets disconnect_reason to Revoked-Externally or Retry-Ceiling-Exhausted; set to true the first time Maya leaves FEAT-21.SPEC-001 after it has shown her the "ended outside the app" note; treated as true whenever disconnect_reason is N/A or User-Initiated | Always | On disconnect transition, and on Maya leaving FEAT-21.SPEC-001 while the note is showing | N/A -- system-computed, not user-entered | Yes |
| retry_count (Calendar Connection) | Must be a non-negative whole number; incremented by exactly 1 at the end of a FEAT-21.SPEC-003 sync run in which every evaluated night's outcome was Sync failure, and reset to 0 at the end of any run with at least one Entry created or Entry updated outcome, or on every successful connect | Always | At the end of every sync run, and on every successful connect | N/A -- system-computed, not user-entered | Yes |
| last_sync_outcome | Must be one of Success, Failed, N/A | Always | At the end of every sync run | N/A -- system-computed | Yes |
| night (Per-Night Calendar Sync Record) | Must reference a specific date; system-set from the Planned Meal's night, never entered by any role | Always | On record creation (first evaluation of that night) | N/A -- system-computed | Yes |
| entry_identity_key | Must be the identity key FEAT-21.SPEC-005 derives for that night; immutable once set | Always | On record creation | N/A -- system-computed | Yes |
| synced_content_identity | Set only on a confirmed create or update for that night; null until the night's first success | Always | On entry create/update succeeded (FEAT-21.SPEC-002) | N/A -- system-computed | Yes |
| status (Per-Night Calendar Sync Record) | Must be one of Pending, Synced, Failing; transitions Pending -> Synced on that night's own success, Synced -> Pending when derived content changes, and either -> Failing when that night's own retry_count reaches platform parameter: `calendar-sync-retry-max-attempts` | Always | On every sync evaluation and outcome for that night | N/A -- system-computed; no direct edit path exists for any role | Yes |
| retry_count (Per-Night Calendar Sync Record) | Must be a non-negative whole number; incremented by 1 on that specific night's own failed create/update attempt; reset to 0 only by that same night's own successful create/update -- never by another night's outcome and never by the household-level counter resetting | Always | On every create/update outcome for that night | N/A -- system-computed, not user-entered | Yes |
| last_attempt_outcome | Must be one of Success, Failed, N/A | Always | On every create/update outcome for that night | N/A -- system-computed | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| One connection per household at a time | household, status | At most one Calendar Connection record may be in Connecting or Connected status for the same household; a second simultaneous connect attempt for a household already Connecting or Connected is refused | "This household is already connecting a calendar. Check the other device." (shown on the second device's FEAT-21.SPEC-001, per the Validation & Limits field's one-connection-per-household rule) |
| Household- and per-night-level retry counters are independent | Calendar Connection.retry_count, Per-Night Calendar Sync Record.retry_count | The two counters never reset or increment each other: a night's own success resets only that night's own retry_count, not the household-level counter's meaning (a full run still needs at least one success to reset the household counter); the household-level counter resetting (e.g., on reconnect) never resets any night's own retry_count. This is what prevents a persistently failing night's history from being erased by an unrelated night's success | N/A -- system-computed reconciliation rule; no user-facing error |
| Retry ceiling triggers disconnection (household level) | Calendar Connection.retry_count, status | When the household-level retry_count reaches platform parameter: `calendar-sync-retry-max-attempts` (i.e., that many consecutive sync runs each ended with every evaluated night failing) for a household still shown as Connected, status transitions to Disconnected with disconnect_reason set to Retry-Ceiling-Exhausted (treated as a connection lost externally) rather than retrying indefinitely | N/A -- system-computed; no blocking error is shown, per the additive-only guarantee (see Business Rules) |
| Retry ceiling triggers per-night Failing status | Per-Night Calendar Sync Record.retry_count, status | When a single night's own retry_count reaches platform parameter: `calendar-sync-retry-max-attempts`, that night's status transitions to Failing; this never changes Calendar Connection.status or disconnect_reason and never affects any other night's record -- the household stays Connected and every other night keeps syncing normally | N/A -- system-computed; no user-facing error anywhere (additive-only guarantee) |
| Disconnect reason and acknowledgment set together with status | status, disconnect_reason, externally_disconnected_acknowledged | Every transition to Disconnected sets disconnect_reason to exactly one of User-Initiated (FEAT-21.SPEC-001 disconnect), Revoked-Externally (FEAT-21.SPEC-002 reports revocation), Retry-Ceiling-Exhausted (household-level ceiling reached), or Household-Deletion-Cascade (FEAT-18.SPEC-008); externally_disconnected_acknowledged is set to false in that same transition only when the reason is Revoked-Externally or Retry-Ceiling-Exhausted -- a User-Initiated or Household-Deletion-Cascade disconnect never sets an unacknowledged note, since no one is left to acknowledge it (household-deletion) or Maya herself just caused it (user-initiated) | N/A -- system-computed; no user-facing error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Connect the calendar capability | Maya (Organiser) | Always | -- |
| Connect the calendar capability | Sam (Other Adult Member) | Never | The "Connect calendar" row is not shown to Sam on FEAT-01.SPEC-010; a direct navigation attempt to FEAT-21.SPEC-001 shows "Only the organiser can change this." |
| Connect the calendar capability | Jordan (older kid, limited login -- Later) | Never | Same as Sam -- row not shown; direct attempt shows "Only the organiser can change this." |
| Connect the calendar capability | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role; there is no path into the product to reach it |
| Connect the calendar capability | Riley (Operator, support) | Never | This feature carries no support-access exposure at all (product-features.md Compliance flags); the read-only support view never surfaces a connect control |
| Disconnect the calendar capability | Maya (Organiser) | Always | -- |
| Disconnect the calendar capability | Sam (Other Adult Member) | Never | Same as Connect above |
| Disconnect the calendar capability | Jordan (older kid, limited login -- Later) | Never | Same as Connect above |
| Disconnect the calendar capability | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role |
| Disconnect the calendar capability | Riley (Operator, support) | Never | No support-access exposure for this feature |
| View connection status | Maya (Organiser) | Always | -- |
| View connection status | Sam (Other Adult Member) | Never | No screen surfaces the connection status for Sam; he sees only the resulting calendar entries outside the product |
| View connection status | Jordan (older kid, limited login -- Later) | Never | Same as Sam |
| View connection status | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role |
| View connection status | Riley (Operator, support) | Never | No support-access exposure for this feature |
| System disconnect on household deletion | System-automation only (FEAT-18.SPEC-008 cascade) | Fires only when a household deletion completes | Not offered as a manual action to any role; no household member remains to be shown feedback once deletion completes |

The Per-Night Calendar Sync Record carries no role-facing actions at all: no role reads, creates, updates, or deletes it directly. It is created, read, and updated exclusively by system automation (FEAT-21.SPEC-002 on sync-outcome events, FEAT-21.SPEC-003 on evaluation), consistent with FEAT-21.SPEC-001's Non-Goals ("this screen shows connection status only, never per-night entry detail"). "View connection status" above covers the full extent of any role's visibility into this feature's sync state.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status | Not Connected (no Calendar Connection record exists until Maya's first connect attempt) | Before any connection has ever been attempted | No |
| connected_at | Set to the time the capability confirms the connection | On connection established | No |
| disconnect_reason | Defaults to N/A (no Calendar Connection record exists, or status is Connecting/Connected) | Before any disconnect has ever occurred, and while Connected | No |
| externally_disconnected_acknowledged | Defaults to true (no unacknowledged note exists until an external disconnect sets it false) | Before any Revoked-Externally or Retry-Ceiling-Exhausted disconnect has occurred | No |
| retry_count (Calendar Connection) | Resets to 0 | On every successful connect, and at the end of any FEAT-21.SPEC-003 sync run with at least one Entry created or Entry updated outcome | No |
| last_sync_outcome | Set to N/A | Before any sync attempt has ever occurred for the household | No |
| synced_content_identity | Null (no per-night sync record exists until that night's first evaluation) | Before a night has ever been evaluated | No |
| status (Per-Night Calendar Sync Record) | Pending | On record creation (a night's first evaluation) | No |
| retry_count (Per-Night Calendar Sync Record) | Resets to 0 | On that specific night's own successful create or update | No |
| last_attempt_outcome | Set to N/A | Before a create/update has ever been attempted for that night | No |

## Business Rules

- **One connection per household at a time (Validation & Limits):** A household must disconnect before connecting a different calendar capability; this spec never allows two simultaneous connections for the same household.
- **Retry-on-failure is automatic and silent, at both levels:** A failed create/update attempt for a given night retries automatically without any user-facing error; only the connect action itself (FEAT-21.SPEC-001) shows an error state, since it is the one user-waited-for interaction (feature's Non-Functional Notes, Responsiveness). Each per-night retry is spaced platform parameter: `calendar-sync-retry-interval` apart. The same ceiling, platform parameter: `calendar-sync-retry-max-attempts`, governs two independent counters: a single night's own retry_count reaching it moves that night to Failing (no household-level effect); the household-level retry_count reaching it (that many consecutive fully-failed sync runs) moves the whole household to Disconnected (Cross-Field Rules) rather than retrying indefinitely.
- **A persistently failing night never gets a false reprieve from an unrelated night's success:** Because the per-night retry_count lives on that night's own Per-Night Calendar Sync Record and resets only on that night's own success, a household with nine nights syncing fine and one night failing every attempt accumulates that one night's failure streak undisturbed -- the household-level counter (which tracks whole-run failures, not any single night) does not erase it, and it is not erased by the other nights succeeding either.
- **Additive-only, non-blocking guarantee:** A failed sync, a lost connection, or the capability never being connected at all leaves the in-app Weekly Plan (owned by FEAT-03 and FEAT-23) entirely unaffected -- no plan data, approval state, or grocery list derivation ever depends on this feature's connection state.
- **Disconnect is a one-way stop, not a rollback:** Disconnecting tears down the connection and stops all further syncing, but calendar entries already created on the external calendar are left exactly as they are (feature's Non-Goals: "Automatic cleanup of past calendar entries"); reconnecting later starts fresh evaluation rather than assuming any prior entry mapping is still valid, since the product has no way to confirm an entry it stopped syncing still exists or is unchanged on the external calendar. A reconnect also resets disconnect_reason to N/A and externally_disconnected_acknowledged to true, since the condition that made them relevant no longer holds.
- **Every disconnect carries a reason, and an externally-caused one stays visible until seen:** disconnect_reason distinguishes a disconnect Maya chose (User-Initiated) from one that happened to her (Revoked-Externally, Retry-Ceiling-Exhausted) or one that happened because the household itself was deleted (Household-Deletion-Cascade). externally_disconnected_acknowledged stays false -- and FEAT-21.SPEC-001 keeps showing the "ended outside the app" note -- until Maya has actually seen it and left the screen; a User-Initiated disconnect never sets it false, since Maya was present for and caused that one herself.
- **Household deletion cascades to disconnection:** When FEAT-18.SPEC-008 completes a household deletion, any active Calendar Connection for that household is disconnected as part of the cascade, with disconnect_reason set to Household-Deletion-Cascade -- this is the cross-feature touchpoint the dependency map assigns to this feature (feature-dependency-map.md, Cross-Feature Touchpoints).

## Edge Cases

- **Household-level retry_count reaches exactly platform parameter: `calendar-sync-retry-max-attempts`** -- The household transitions to Disconnected on the sync run that completes this exact count (every night in that run failed), not before and not after; disconnect_reason is set to Retry-Ceiling-Exhausted and externally_disconnected_acknowledged to false; FEAT-21.SPEC-001 shows the "ended outside the app" note on next visit.
- **One specific night fails every attempt for several consecutive sync runs while every other night in the household keeps syncing successfully** -- That night's own retry_count climbs independently on its Per-Night Calendar Sync Record; because at least one other night succeeds in each of those runs, the household-level retry_count keeps resetting to 0 and the household stays Connected. When the failing night's own retry_count reaches platform parameter: `calendar-sync-retry-max-attempts`, only that night's status becomes Failing -- the household connection and every other night are unaffected, and the failing night's history is never erased by the other nights' success.
- **The Failing night's dinner is later swapped to a different recipe** -- The derived content identity changes, so that night's status returns to Pending and its retry_count resets to 0, giving it a fresh sequence of attempts under the new content.
- **Maya attempts to connect from two devices at effectively the same time** -- The first request to reach Connecting status wins; the second is refused with "This household is already connecting a calendar. Check the other device." per the one-connection-per-household cross-field rule.
- **Maya disconnects and immediately reconnects within the same session** -- The reconnect is treated as an entirely new connection: the household-level retry_count and last_sync_outcome reset, disconnect_reason resets to N/A, externally_disconnected_acknowledged resets to true, and FEAT-21.SPEC-003 re-evaluates every current-week and one-week-ahead night as if syncing for the first time (each night's own Per-Night Calendar Sync Record also starts fresh), since no prior entry mapping is assumed still valid.
- **The connection is revoked externally while Maya is not viewing FEAT-21.SPEC-001, and she does not open the screen again for several days** -- externally_disconnected_acknowledged stays false the entire time; the "ended outside the app" note is still shown, unchanged, the next time she does open the screen, regardless of how long has passed.
- **The household is deleted while a sync attempt for it is in flight** -- The in-flight attempt's eventual outcome (success or failure) is simply discarded once the household no longer exists; the deletion cascade's disconnect is not blocked by or dependent on that outcome, and disconnect_reason is set to Household-Deletion-Cascade regardless of what the in-flight attempt would have reported.
- **A household never connects a calendar at all** -- status remains Not Connected indefinitely with no adverse effect anywhere else in the product, consistent with the additive-only guarantee; this is the default state for every household and is not itself an error condition.
- **Sam attempts to reach the connection-status view directly (e.g., a shared link)** -- Denied: no screen exists for Sam to view this data; the attempt resolves to "Only the organiser can change this." on FEAT-21.SPEC-001, or lands him back on FEAT-01.SPEC-010 if the specific screen cannot be reached at all.

## Acceptance Criteria

**FEAT-21.SPEC-004-AC-01:** Given a household with no existing Calendar Connection record, when Maya taps "Connect calendar" and the capability confirms, then status transitions Not Connected -> Connecting -> Connected, connected_at is set, and retry_count is 0.

**FEAT-21.SPEC-004-AC-02:** Given a household is already Connecting on one device, when a second connect attempt is made from another device at the same time, then the second attempt is refused with "This household is already connecting a calendar. Check the other device."

**FEAT-21.SPEC-004-AC-03:** Given a background create/update attempt fails for one night of a connected household, when the failure is recorded, then that night's Per-Night Calendar Sync Record retry_count increments by one and last_attempt_outcome is set to Failed, and no user-facing error appears anywhere in the product.

**FEAT-21.SPEC-004-AC-04:** Given a household-level sync run completes with every evaluated night failing, and this is the run that brings the household-level retry_count to platform parameter: `calendar-sync-retry-max-attempts`, when that run completes, then Calendar Connection.status transitions to Disconnected, disconnect_reason is set to Retry-Ceiling-Exhausted, externally_disconnected_acknowledged is set to false, and FEAT-21.SPEC-001 shows the "ended outside the app" note on next visit.

**FEAT-21.SPEC-004-AC-05:** Given a connected household's calendar sync is failing and retrying, when Maya views her Weekly Plan (FEAT-03 or FEAT-23) during this time, then the plan, its approval state, and its grocery list are entirely unaffected.

**FEAT-21.SPEC-004-AC-06:** Given Maya disconnects a Connected household, when the disconnect completes, then status becomes Disconnected, disconnected_at is set, disconnect_reason is set to User-Initiated, externally_disconnected_acknowledged remains true, and no further create/update requests are sent for that household.

**FEAT-21.SPEC-004-AC-07:** Given Maya disconnects a household with previously created calendar entries, when the disconnect completes, then those entries remain on the external calendar unchanged -- no delete request is ever sent.

**FEAT-21.SPEC-004-AC-08:** Given Maya reconnects a household that was previously disconnected, when the reconnect completes, then the household-level retry_count and last_sync_outcome reset, disconnect_reason resets to N/A, externally_disconnected_acknowledged resets to true, and FEAT-21.SPEC-003 re-evaluates every current-week and one-week-ahead night as a fresh sync.

**FEAT-21.SPEC-004-AC-09:** Given a household is deleted (FEAT-18.SPEC-008), when the deletion cascade runs, then any active Calendar Connection for that household transitions to Disconnected as part of it, with no feedback shown to any role.

**FEAT-21.SPEC-004-AC-10:** Given Maya (Organiser) is on FEAT-21.SPEC-001, when she looks for the Connect and Disconnect controls, then both are available to her at all times per her role.

**FEAT-21.SPEC-004-AC-11:** Given Sam (Other Adult Member) attempts to reach FEAT-21.SPEC-001 directly, when the navigation resolves, then he sees "Only the organiser can change this." and no connect/disconnect control is ever shown to him.

**FEAT-21.SPEC-004-AC-12:** Given Jordan (older kid, limited login) attempts to reach FEAT-21.SPEC-001 directly, when the navigation resolves, then he sees the same "Only the organiser can change this." denial as Sam.

**FEAT-21.SPEC-004-AC-13:** Given Riley (Operator, support) is in an open support-request view for a household, when Riley looks for any calendar-connection information, then none is shown, consistent with this feature carrying no support-access exposure.

**FEAT-21.SPEC-004-AC-14:** Given a household has never connected a calendar, when any spec checks its Calendar Connection status, then it reads Not Connected with no adverse effect on any other feature.

**FEAT-21.SPEC-004-AC-15:** Given one specific night has failed every create/update attempt for several consecutive sync runs while every other night in the same household syncs successfully in each of those runs, when each run completes, then that night's own Per-Night Calendar Sync Record retry_count keeps climbing while the household-level Calendar Connection.retry_count keeps resetting to 0, and the household remains Connected.

**FEAT-21.SPEC-004-AC-16:** Given a single night's own retry_count reaches platform parameter: `calendar-sync-retry-max-attempts` while other nights in the household are syncing successfully, when that night's next attempt is evaluated, then only that night's Per-Night Calendar Sync Record status becomes Failing, and Calendar Connection.status remains Connected with no effect on any other night.

**FEAT-21.SPEC-004-AC-17:** Given Maya disconnects a household whose Calendar Connection was previously Connected, when the disconnect completes, then disconnect_reason is set to User-Initiated and externally_disconnected_acknowledged remains true (no unacknowledged note is created), consistent with AC-06.

**FEAT-21.SPEC-004-AC-18:** Given a household's connection is revoked externally, when the disconnect is recorded, then disconnect_reason is set to Revoked-Externally and externally_disconnected_acknowledged is set to false, and it stays false until Maya opens FEAT-21.SPEC-001, sees the "ended outside the app" note, and leaves the screen.

**FEAT-21.SPEC-004-AC-19:** Given every night evaluated in a single sync run fails its create/update attempt, when that run completes, then the household-level Calendar Connection.retry_count increments by exactly 1 for the run, in addition to each individual night's own retry_count incrementing on its own record.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 14 | 14 |
| Cross-Field Rules | 5 | 5 |
| Authorization Rules | 15 | 15 |
| Defaults/Derivations | 10 | 10 |
| Business Rules | 6 | 6 |
| Edge Cases | 9 | 9 |



# Logic/Rule Spec: Calendar Entry Content Derivation Rule

## Overview

**Name:** Calendar Entry Content Derivation Rule
**ID:** FEAT-21.SPEC-005
**Type:** Logic/Rule
**Purpose:** Derives what a synced calendar entry contains (night, dish name, timing) from Weekly Plan and Planned Meal data, and keeps one entry per planned night stable across swaps.
**Parent Feature:** FEAT-21 -- Family Calendar Sync
**Governed Entity:** Calendar Entry

## Scope and Non-Goals

**In Scope:**
- The exact content placed on a synced calendar entry (night, dish name, timing) and the fields explicitly excluded from it
- The stable per-night entry-identity rule that makes an update possible instead of a duplicate create
- Derivation logic for each Calendar Entry field, sourced from Weekly Plan and Planned Meal data
- Authorization for who (if anyone) can influence entry content directly

**Non-Goals:**
- Creating the entry on the household's calendar, or handling the capability's create/update response -- owned by FEAT-21.SPEC-002 (Family Calendar Integration), which sends the content this spec derives.
- Deciding when a night needs to be evaluated for sync -- owned by FEAT-21.SPEC-003 (Weekly Dinner Calendar Sync), which calls this spec's derivation and identity rules but owns the evaluation schedule itself.
- Governing the connection this content travels over, retry behavior, or the one-connection-per-household limit -- owned by FEAT-21.SPEC-004 (Calendar Connection & Sync Governance Rules).
- Deriving content for leftover-lunch Planned Meals -- excluded per FEAT-21.SPEC-003's Non-Goals; this feature syncs dinners only, so no leftover-lunch content is ever derived here.

## Governed Entity

**Entity:** Calendar Entry
**Source:** Feature-local entity, derived by this spec from the dependency map's Weekly Plan and Planned Meal entities (feature-overview.md, Referenced Entities: Weekly Plan and Planned Meal). It is not itself a Connected Entity in product-features.md and lives entirely on the household's external calendar once synced (FEAT-21.SPEC-002).

| Field | Data Type | Description |
|-------|-----------|-------------|
| night_date | derived | The calendar date of the dinner's night, computed from the Weekly Plan's week and the Planned Meal's night |
| dish_name | derived | The entry's title text, derived from the Planned Meal's recipe name |
| timing | derived | A generic evening dinner-time marker for the entry, since neither Household nor Planned Meal records a specific dinner hour |
| entry_identity_key | derived | The stable key (household, night_date) that identifies "this night's entry" independent of which recipe currently occupies it |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-21.SPEC-003 | Weekly Dinner Calendar Sync | On every sync evaluation: calls this spec's derivation and identity rules to decide whether a night's content is new, unchanged, or updated |
| FEAT-21.SPEC-002 | Family Calendar Integration | Carries the already-derived content across the boundary to the calendar capability; does not re-derive it |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| night_date | Must resolve to exactly one calendar date within the Weekly Plan's week, matching the Planned Meal's night; system-derived, never entered by any role | Always | On every derivation (FEAT-21.SPEC-003 evaluation) | N/A -- system-computed; a night that cannot resolve to a date (e.g., an incomplete or unapproved plan) is simply not yet eligible for sync | Yes |
| dish_name | Must be non-empty, taken directly from the Planned Meal's recipe name; system-derived, never entered or edited by any role for this entry | Always | On every derivation | N/A -- system-computed; recipe name is itself a required field on the Recipe entity (dependency map), so this never resolves to empty for a valid Planned Meal | Yes |
| timing | Fixed generic evening marker; does not vary by household, night, or recipe | Always | On every derivation | N/A -- system-computed, constant value; no per-household or per-recipe timing exists to derive from (neither Household nor Planned Meal records a specific dinner hour) | Yes |
| entry_identity_key | Must be composed of (household, night_date) only -- never includes the recipe, dish name, or any other content field | Always | On every derivation | N/A -- system-computed; this is the rule that keeps the identity stable across a swap (see Cross-Field Rules) | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Identity is content-independent | entry_identity_key, dish_name | entry_identity_key never changes when dish_name changes for the same night (e.g., after a swap); only night_date and household determine identity | N/A -- structurally enforced; FEAT-21.SPEC-003 always resolves a night to its existing entry_identity_key before comparing content, so a changed dish_name alone can never produce a new identity |
| One dish name per night | night_date, dish_name | At most one dish_name is associated with a given night's entry at any time -- a swap replaces the prior dish_name in the derivation, it never adds a second | N/A -- structurally enforced by the Weekly Plan's own one-dinner-per-night rule (dependency map, Planned Meal: "at most one dinner per night"), which this spec inherits rather than re-validates |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Derive calendar entry content for a night | No household role -- system-automation only (FEAT-21.SPEC-003) | Fires only during a sync evaluation for a connected household | Not offered as a manual action to any role; no household member edits or authors calendar entry content directly -- content always traces back to whichever feature (FEAT-03, FEAT-04, FEAT-23) wrote the underlying Planned Meal |
| View derived entry content in-app | No household role -- this content has no in-app surface | Always | Not shown anywhere in the product beyond FEAT-21.SPEC-001's connection status, per the feature's Data Notes ("Displayed: N/A within the product beyond a connection status"); every role views the resulting entry only outside the product, on the household's own calendar |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| night_date | The calendar date obtained by combining the Weekly Plan's week with the Planned Meal's night (e.g., "Tuesday" within the week of March 3 resolves to March 4) | Evaluated on every sync run for every dinner night in scope | No |
| dish_name | Set to the current Planned Meal's recipe name at evaluation time; recomputed on every evaluation so a swap is always reflected | Evaluated on every sync run | No -- no household role edits an entry's dish name directly; changing it means swapping or changing the underlying Planned Meal (FEAT-04, FEAT-23), which this spec then re-derives from |
| timing | A fixed, generic evening dinner-time marker (never a specific clock time) | Applied to every entry, always | No -- no per-night or per-household dinner time exists in the product to derive a specific hour from |
| entry_identity_key | Composed once, the first time a given night is synced, from (household, night_date); reused unchanged for every later evaluation of the same night, even across a swap | Set on first sync for a night; read (never recomputed) on every later evaluation of that same night | No |
| Content Exclusion (derivation constraint, not a stored field) | The entry never carries vegetarian_option, cook_time, rough_cost, pantry_callout, swap_history, allergy or Dietary Rule data, or any member-identifying detail -- only night_date, dish_name, and timing are ever placed on the entry (feature's Non-Functional Notes, Data sensitivity/privacy) | Applied to every derivation, always | No -- this exclusion is not user-configurable; it is a fixed product decision to keep children's and household-sensitive data off content the family sees outside the product |

## Business Rules

- **Stable per-night identity enables "update, not duplicate":** Because entry_identity_key depends only on (household, night_date) and never on dish_name, FEAT-21.SPEC-003 can always tell whether a night already has an entry and update it, rather than ever creating a second entry for the same night (Key Capabilities: "Keep it current").
- **Content minimization is a fixed rule, not a per-household setting:** No household, including Maya, can configure additional fields onto a synced entry; the excluded-field list in Defaults and Derivations applies identically to every household, keeping children's data (ASMP-26) and cost/allergy detail out of content visible outside the product.
- **Derivation always reflects the current Planned Meal, never a cached snapshot:** Every sync evaluation re-derives dish_name from the Planned Meal's current recipe, so a swap is always picked up on the next evaluation (FEAT-21.SPEC-003) rather than requiring a manual refresh.
- **A night with no dinner has no entry content to derive:** If a night in the evaluation window holds no dinner Planned Meal (cleared, or not yet picked), no Calendar Entry content is derived for it, and FEAT-21.SPEC-003 takes no create/update action for that night.

## Edge Cases

- **A dinner is swapped twice in quick succession before either sync completes** -- Each evaluation re-derives dish_name from whatever recipe is current at that moment; the entry_identity_key is unaffected by either swap, so both evaluations target the same existing entry and the final synced content reflects the most recent recipe.
- **A recipe name is unusually long** -- The full recipe name is used as dish_name without truncation by this rule; any display-length handling on the external calendar is outside the product's control and outside this spec's scope.
- **The Weekly Plan's week boundary changes (e.g., a plan is rebuilt for a different week)** -- night_date is recomputed from the new week; if this produces a different calendar date for what was "the same slot," a new entry_identity_key results and a new entry is created rather than the old one being reinterpreted, since identity is date-based, not slot-based.
- **A night's dinner is cleared and a different dinner is later placed in the same night** -- The existing entry_identity_key for that night_date is reused (identity depends on night_date, not on which dinner occupies it), so the new dinner's content updates the existing entry rather than creating a second one.
- **Two different households happen to share the same calendar date for their respective dinners** -- entry_identity_key includes household, so each household's entry is fully independent; no cross-household collision can occur.
- **Maya attempts to view or edit a calendar entry's derived content from within the product** -- No such control exists anywhere in the product (Authorization Rules); the only in-app surface for this feature is the connection status on FEAT-21.SPEC-001.

## Acceptance Criteria

**FEAT-21.SPEC-005-AC-01:** Given a connected household's Weekly Plan has a dinner for Tuesday of the current week, when FEAT-21.SPEC-003 derives that night's content, then night_date resolves to Tuesday's calendar date and dish_name is set to that dinner's recipe name.

**FEAT-21.SPEC-005-AC-02:** Given a synced night's dinner is swapped to a different recipe, when FEAT-21.SPEC-003 next evaluates that night, then dish_name is re-derived to the new recipe's name while entry_identity_key stays exactly as it was.

**FEAT-21.SPEC-005-AC-03:** Given a night is synced for the first time, when entry_identity_key is composed, then it is derived only from (household, night_date), never from dish_name or any other content field.

**FEAT-21.SPEC-005-AC-04:** Given a night's dinner is cleared and later replaced with a different dinner, when FEAT-21.SPEC-003 evaluates the replacement, then the existing entry_identity_key for that night_date is reused and the entry is updated rather than a new one created.

**FEAT-21.SPEC-005-AC-05:** Given any dinner Planned Meal in a connected household's plan, when its content is derived, then the resulting Calendar Entry carries only night_date, dish_name, and timing -- never vegetarian_option, cook_time, rough_cost, pantry_callout, swap_history, allergy data, or any member-identifying detail.

**FEAT-21.SPEC-005-AC-06:** Given a night in the evaluation window has no dinner Planned Meal, when FEAT-21.SPEC-003 evaluates that night, then no Calendar Entry content is derived for it.

**FEAT-21.SPEC-005-AC-07:** Given two different connected households each have a dinner on the same calendar date, when their respective entry_identity_keys are composed, then the household reference keeps the two identities fully distinct.

**FEAT-21.SPEC-005-AC-08:** Given a Weekly Plan is rebuilt so a night's slot maps to a different calendar date than before, when night_date is recomputed, then a new entry_identity_key results and a new entry is created for the new date rather than reinterpreting the prior entry.

**FEAT-21.SPEC-005-AC-09:** Given a dinner's recipe name is unusually long, when dish_name is derived, then the full recipe name is used without truncation by this rule.

**FEAT-21.SPEC-005-AC-10:** Given any household's synced entry, when timing is derived, then it is set to the same fixed, generic evening marker regardless of household, night, or recipe.

**FEAT-21.SPEC-005-AC-11:** Given Maya looks for a way to view or edit a synced entry's content from within the product, when she searches the household settings and Weekly Plan screens, then no such control exists anywhere -- the only in-app surface for this feature is the connection status on FEAT-21.SPEC-001.

**FEAT-21.SPEC-005-AC-12:** Given a dinner is swapped twice before either sync attempt completes, when both evaluations run, then both target the same entry_identity_key and the final synced dish_name reflects the most recently derived recipe.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 2 | 2 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
