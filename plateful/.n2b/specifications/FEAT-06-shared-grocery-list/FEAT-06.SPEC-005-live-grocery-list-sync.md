---
document_type: spec
spec_type: integration
spec_id: FEAT-06.SPEC-005
spec_name: Live Grocery List Sync
spec_slug: live-grocery-list-sync
parent_feature: FEAT-06
parent_feature_name: Shared Grocery List
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Integration Spec: Live Grocery List Sync

## Overview

**Name:** Live Grocery List Sync
**ID:** FEAT-06.SPEC-005
**Type:** Integration
**Purpose:** The product pushes every grocery-list change to every household device within seconds and reconciles changes made while offline once connectivity returns, so the whole household always sees the same live list.
**Parent Feature:** FEAT-06 -- Shared Grocery List

## Scope and Non-Goals

**In Scope:**
- Propagating tick, add, edit, remove, and "already have it" changes to every household device in near-real time
- Holding changes made without connectivity and syncing them automatically on reconnect
- The user-facing behavior when the synchronization capability is slow, unavailable, or a change cannot be applied
- Disclosure of what data moves through this capability

**Non-Goals:**
- The conflict-resolution rules applied once changes are received (idempotent ticks, last-write-wins edits, duplicate-free reconnect) -- owned by FEAT-06.SPEC-008; this spec delivers the events, that spec decides how they merge
- The plan-derived recalculation this integration also propagates -- owned by FEAT-06.SPEC-002; this spec only carries its results to devices
- Real-time sync of the Weekly Plan itself -- owned by FEAT-03.SPEC-011 (live plan sync); this spec covers the Grocery List only, per the External Touchpoints table's per-feature split of the real-time data-synchronization capability
- Choosing the synchronization vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no vendor mandate for this capability

## Capability Category

**Category:** Real-time data synchronization
**Dependency Source:** ASMP-35 (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Real-time data synchronization" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-06, FEAT-03, FEAT-04, FEAT-23; this feature's Integration spec: FEAT-06.SPEC-005, "live grocery list sync and offline reconciliation")
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| A tick, add, edit, remove, or "already have it" made by one household member appears on every other member's device within about 2 seconds under normal connectivity | Tick off live | FEAT-06.SPEC-001 (Grocery List) |
| A household member can read, tick, and add items with no signal in the store; changes hold locally and sync automatically once connectivity returns | Work offline in the store | FEAT-06.SPEC-001 (Grocery List) |
| A recalculated list (from a plan change, swap, safety removal, or pantry change) reaches every device live, without a manual refresh | Auto-generate from the plan | FEAT-06.SPEC-001 (Grocery List), FEAT-06.SPEC-002 (Grocery List Generation & Recalculation) |
| A swapped meal's list update is visible to the whole household immediately, not only to the member who swapped it | (Cross-feature dependency) | FEAT-04 (One-Tap Meal Swap), via FEAT-06.SPEC-001 |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Grocery list item state | Grocery List Item -- `ingredient_name`, `quantity_and_unit`, `aisle`, `origin`, `ticked` | Any tick, add, edit, remove, "already have it", or recalculation change | Lets every household device render the current, correct list |
| List status | Grocery List -- `week`, `status` | List creation, recalculation, or archive | Lets devices know which week's list is current and whether it is Active or Archived |
| Attribution | The acting member's display name | A manual item is added or edited | Powers the "added by" label shown on manual items |

Recipe content, pantry data beyond what a recalculation already excludes, household budget, and every other household setting never leave the product through this capability -- it carries only Grocery List and Grocery List Item state and the acting member's name.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| A change committed on another device | Any household device commits a tick, add, edit, remove, or "already have it" | Grocery List Item -- the corresponding field(s), reconciled per FEAT-06.SPEC-008 |
| A recalculated list state | FEAT-06.SPEC-002 completes a recalculation | Grocery List, Grocery List Item -- as computed |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Change broadcast received | Any household device commits a tick, add, edit, remove, or "already have it" change | Local Grocery List Item state is updated to match, reconciled per FEAT-06.SPEC-008 | The item updates live on the Grocery List screen with no manual refresh needed | FEAT-06.SPEC-001, FEAT-06.SPEC-008 |
| Connectivity restored with queued local changes | A device reconnects after holding one or more changes offline | Queued changes are sent and merged into the shared list per FEAT-06.SPEC-008 | The offline banner disappears and the list reflects the merged state automatically, with no member action required | FEAT-06.SPEC-001, FEAT-06.SPEC-008 |
| Delivery delayed or dropped mid-transmission | A change fails to propagate on its first attempt | None until retried | No feedback if the retry eventually succeeds (silent, per the background-retry-then-surface-only-on-final-failure pattern); an error banner appears only if it cannot eventually succeed | FEAT-06.SPEC-001 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-06.SPEC-001 (Grocery List) | The acting member's own device shows their change immediately (optimistic update); if it has not propagated to other devices within a few seconds, no error is shown to the acting member -- propagation is simply still catching up. | The screen switches to the Offline/Degraded state: a persistent banner "You're offline -- changes will sync when you reconnect." appears; reading, ticking, adding, editing, and removing all remain fully usable; every change queues locally. | A change that cannot ultimately be applied (after background retries) surfaces the banner "Couldn't save your change. Check your connection and try again." with a Retry option, per FEAT-06.SPEC-001's Error state; the list otherwise remains unchanged and consistent. |

## Consent and Disclosure

- **What is shared, and with whom** -- Grocery list content (ingredient names, quantities, aisles, tick state) and the acting member's display name are shared only among the household's own devices through this capability; no data leaves the household's membership boundary and no external party receives it. This is disclosed in the household's general data-handling notice (owned by FEAT-01) rather than as a separate per-use consent moment, since sharing the list live among household members is the core, always-on function of this feature, not an optional third-party data share.
- **What is never shared through this capability** -- Recipe content, pantry data beyond what a recalculation already excludes, household budget, and every other household setting.

## Edge Cases

- **A change event arrives for an item that no longer exists (e.g., already removed by another member)** -- The event is ignored; no error surfaces, since the item's absence is already the correct end state.
- **The same change event is delivered twice** -- The second delivery changes nothing further: a tick already applied stays applied, an edit already applied is not reapplied a second time.
- **Events arrive out of order (an edit's event arrives before an earlier tick's event, by wall-clock delivery time)** -- Reconciliation applies events by their original event time, not arrival time, per FEAT-06.SPEC-008; a late-arriving earlier event does not overwrite a correctly-applied later one.
- **The capability goes down mid-change** -- If a tick or add was not confirmed as sent, it stays queued locally rather than reaching a half-applied state on other devices; the acting device shows its own optimistic state until the change is confirmed sent.
- **A member reconnects with several queued offline changes to the same item** -- All queued changes for that item apply in their original event order, per FEAT-06.SPEC-008, rather than only the last one surviving arbitrarily.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-001 (Grocery List) | Triggered by (inbound) | Every write action on the screen sends a change through this integration |
| FEAT-06.SPEC-001 (Grocery List) | Affects (outbound) | Degradation states, the offline banner, and live updates surface here |
| FEAT-06.SPEC-002 (Grocery List Generation & Recalculation) | Triggered by (inbound) | Recalculated list changes are propagated through this integration |
| FEAT-06.SPEC-008 (Offline Conflict Resolution Rules) | Triggers (outbound) | Every inbound change and every reconnect-sync is reconciled per this spec's rules |
| FEAT-04 (One-Tap Meal Swap) | Affects (outbound, dependency) | Relies on this capability to reflect a swapped plan's list changes live, per the Cross-Feature Touchpoints table |

## Analytics and Success Signals

- **grocery_item_synced_after_offline** (queued_change_count) -- supports success-metrics.md: "Grocery List Live-Update Trust"
- **grocery_list_sync_latency_exceeded** (screen: FEAT-06.SPEC-001; observed_delay) -- supports success-metrics.md: "Grocery List Live-Update Trust"
- **grocery_list_sync_degradation_shown** (condition: slow / down / rejected) -- N/A -- no Stage 2 metric measures degradation frequency directly; retained so the product's tolerance for connectivity trouble is observable

## Acceptance Criteria

**FEAT-06.SPEC-005-AC-01:** Given Maya ticks an item on her phone with normal connectivity, when the tick is committed, then Sam's device shows the same item ticked within about 2 seconds.

**FEAT-06.SPEC-005-AC-02:** Given Sam is in the supermarket with no signal, when he ticks three items, then all three tick locally and immediately, and the offline banner "You're offline -- changes will sync when you reconnect." is shown.

**FEAT-06.SPEC-005-AC-03:** Given Sam's device reconnects after ticking items offline, when connectivity returns, then the queued ticks sync automatically, the offline banner disappears, and no member action is required.

**FEAT-06.SPEC-005-AC-04:** Given FEAT-06.SPEC-002 recalculates the list after a swap, when the recalculation completes, then every household device reflects the updated list live, without a manual refresh.

**FEAT-06.SPEC-005-AC-05:** Given a change event arrives for an item another member already removed, when the event is processed, then it is ignored and no error is shown.

**FEAT-06.SPEC-005-AC-06:** Given the same change event is delivered twice, when the second delivery arrives, then no further change occurs and no duplicate effect is visible.

**FEAT-06.SPEC-005-AC-07:** Given events arrive out of order for the same item, when they are processed, then the final state reflects the events' original order by event time, not by arrival time.

**FEAT-06.SPEC-005-AC-08:** Given Maya's device is slow to propagate a change, when the change has not reached other devices within a few seconds, then Maya's own device still shows her change immediately and no error is shown to her.

**FEAT-06.SPEC-005-AC-09:** Given the synchronization capability is down, when Sam opens the Grocery List, then reading, ticking, adding, editing, and removing remain fully usable, and the offline banner is shown.

**FEAT-06.SPEC-005-AC-10:** Given a queued change ultimately cannot be applied after background retries, when the final failure occurs, then the banner "Couldn't save your change. Check your connection and try again." appears with a Retry option.

**FEAT-06.SPEC-005-AC-11:** Given a household member reviews what data this capability shares, when they consult the household's data-handling notice, then it states that only grocery list content and the acting member's name are shared, only among the household's own devices.

**FEAT-06.SPEC-005-AC-12:** Given Sam reconnects with several queued changes to the same item, when they sync, then all queued changes apply in their original order, not only the most recent one.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 3 (1 screen) | 3 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |
