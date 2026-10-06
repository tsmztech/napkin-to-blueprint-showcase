---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-21.SPEC-004
spec_name: Calendar Connection & Sync Governance Rules
spec_slug: calendar-connection-sync-governance-rules
parent_feature: FEAT-21
parent_feature_name: Family Calendar Sync
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 40
acceptance_criteria_count: 19
---

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
