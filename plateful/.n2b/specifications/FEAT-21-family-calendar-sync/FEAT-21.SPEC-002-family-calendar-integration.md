---
document_type: spec
spec_type: integration
spec_id: FEAT-21.SPEC-002
spec_name: Family Calendar Integration
spec_slug: family-calendar-integration
parent_feature: FEAT-21
parent_feature_name: Family Calendar Sync
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

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
