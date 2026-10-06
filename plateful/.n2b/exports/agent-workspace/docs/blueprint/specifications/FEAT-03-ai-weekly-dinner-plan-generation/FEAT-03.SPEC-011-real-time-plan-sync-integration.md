---
document_type: spec
spec_type: integration
spec_id: FEAT-03.SPEC-011
spec_name: Real-Time Plan Sync Integration
spec_slug: real-time-plan-sync-integration
parent_feature: FEAT-03
parent_feature_name: AI Weekly Dinner Plan Generation
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Integration Spec: Real-Time Plan Sync Integration

## Overview

**Name:** Real-Time Plan Sync Integration
**ID:** FEAT-03.SPEC-011
**Type:** Integration
**Purpose:** Product boundary to the real-time data-synchronization capability that reflects generation, approval, and plan changes live across every household member's device.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation

## Scope and Non-Goals

**In Scope:**
- Propagating Weekly Plan and Planned Meal changes (generation, approval, auto-adoption, accepted swaps, safety removals) live to every household member's device viewing FEAT-03.SPEC-001
- Reconciling a device's plan view after it reconnects following an offline period
- The product's behavior when the sync capability is slow, unavailable, or rejects an update

**Non-Goals:**
- Synchronizing the Grocery List -- owned by Shared Grocery List's own Integration spec (FEAT-06.SPEC-005), which covers the same capability category for list data; this spec covers only Weekly Plan and Planned Meal data
- The swap, approval, or safety-removal logic itself -- owned by FEAT-04, FEAT-03.SPEC-008, and FEAT-02 respectively; this spec only propagates their already-decided outcomes
- Offline creation of a new Weekly Plan -- generation requires connectivity (FEAT-03.SPEC-001's States); this spec covers only reflecting a plan that some connected process has already changed

## Capability Category

**Category:** Real-time data synchronization
**Dependency Source:** ASMP-35 -- "Real-time data-synchronization capability -- Required for the shared grocery list and plan to update live across household members' devices and to reconcile changes made while offline" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Real-time data synchronization (ASMP-35)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-06, FEAT-03, FEAT-04, FEAT-23; Integration Specs: FEAT-03.SPEC-011 (live plan sync), FEAT-06.SPEC-005 (live grocery list sync and offline reconciliation))
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Every household member sees the same current week's plan update the moment it changes, without a manual refresh | Approve the week's plan (and see the result of others' actions live) | FEAT-03.SPEC-001 (Weekly Plan View) |
| A household member who was offline sees the plan reconcile correctly to the latest state once reconnected | Never show a stale or conflicting plan | FEAT-03.SPEC-001 |
| The organiser sees a swap suggestion, an accepted swap, a safety removal, or auto-adoption reflected live while reviewing the week | Approve the week's plan, including any swap suggestions from other adults | FEAT-03.SPEC-001 |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Weekly Plan change | Weekly Plan -- status, approval, estimated_total, over_budget_note | Generation completes, approval is recorded, or auto-adoption fires | Propagates the plan's current state to every household member's connected device |
| Planned Meal change | Planned Meal -- recipe, safety_badge, vegetarian_option, cook_time, rough_cost, pantry_callout, status, swap_history | A dinner is created, swapped, or removed for safety | Propagates the affected dinner's current state to every household member's connected device |

Only Weekly Plan and Planned Meal fields already defined in the dependency map are sent; no member-identifying data beyond what already appears on these entities (e.g., swap_history's prior recipes) leaves the product through this channel, and no data belonging to another household is ever included.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Sync acknowledgement / conflict signal | The capability confirms a change was propagated, or reports that a device's locally queued change conflicts with a newer server-side state | Weekly Plan / Planned Meal -- no field changes from the acknowledgement itself; a conflict signal is handled per the dependency map's Contention resolution (reject-with-refresh for plan slot changes, safety removal always wins) |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Plan or meal update propagated | Any connected device's Weekly Plan or Planned Meal state changes from any source (generation, approval, auto-adoption, swap, safety removal) | The receiving device's local view updates to match | The affected dinner card or banner updates in place on FEAT-03.SPEC-001, with a brief highlight; no full-screen reload | FEAT-03.SPEC-001 |
| Reconnection reconciliation | A device that was offline regains connectivity | The device's local plan view is reconciled to the latest server-side state; any of that device's own queued actions (e.g., a queued rating or leftover-lunch response) are resubmitted | The plan view updates silently to the current state; a queued action's outcome (success or conflict) surfaces per its own owning spec (FEAT-11, FEAT-12) | FEAT-03.SPEC-001 |
| Sync conflict reported | Two devices attempt conflicting changes to the same Planned Meal slot at effectively the same time | No plan data change from this event alone -- the conflict is resolved per the dependency map's Contention note (reject-with-refresh per slot; safety removal always wins) and the losing device's action is rejected | The losing device sees "This dinner just changed -- here's the latest." per FEAT-03.SPEC-001's Edge Cases | FEAT-03.SPEC-001, FEAT-04 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-03.SPEC-001 (Weekly Plan View) | Updates from other devices may take longer than the near-instant window (ASMP-22) to appear; no error is shown, and the screen continues to reflect its last-known state while waiting. | The screen falls back to its last-synced state and shows the Offline/Degraded messaging described in FEAT-03.SPEC-001's States table: "Reconnect to approve or change the plan." Viewing the already-loaded plan remains fully available. | A rejected local action (e.g., a swap attempt that conflicts with a newer server state) is treated per the Contention resolution: the action fails with "This dinner just changed -- here's the latest." and the screen refreshes to the current state; no other action on the screen is blocked by one rejected update. |

## Consent and Disclosure

- **No separate disclosure needed** -- Weekly Plan and Planned Meal data already displayed to every household member on FEAT-03.SPEC-001 is the same data propagated by this integration; since every household member already sees this data in-app, no additional consent moment is required beyond the household's general data-sharing posture (ASMP-14, ASMP-26: household data is never sold or used for advertising, and stays private to the household). This integration moves data only between the household's own devices, never to a party outside the household.
- **What is never shared** -- No plan or meal data is sent to any device outside the household's own member sessions; Riley's read-only support access (FEAT-22) is a separate, permissioned path and not part of this synchronization channel.

## Edge Cases

- **Two household members swap the same dinner slot at effectively the same moment on different devices** -- Per the dependency map's Contention note for Weekly Plan (reject-with-refresh per night slot, no more than one active swap operation per slot), the first swap to be confirmed wins; the second device's attempt is rejected and its screen refreshes to show the confirmed recipe.
- **A safety removal and a swap acceptance target the same slot concurrently** -- The safety removal always wins over any concurrent change, per the dependency map's Contention note; the swap acceptance is rejected and the device is shown the safety-driven state instead.
- **A device is offline for an extended period spanning a full plan regeneration (a new week generated while offline)** -- On reconnection, the device reconciles directly to the new week's plan; it does not attempt to replay stale actions against the now-archived prior week.
- **Sync event arrives for a Weekly Plan that has since been archived (a new week already generated)** -- The event is discarded for display purposes on the current-week view; the archived week's own record (FEAT-19) retains the historical state it had at archival.
- **The same plan-update event is delivered to a device twice** -- The second delivery changes nothing further, since the device's local state already matches; no duplicate highlight animation or duplicate toast fires.
- **Capability goes down mid-approval** -- If the approval write was not confirmed as propagated, the approving device shows the Offline/Degraded messaging and the approval is retried once connectivity returns, rather than leaving other devices with a stale unapproved view indefinitely.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-001 (Weekly Plan View) | Affects (outbound) | Live-updates this screen whenever the plan changes from any source |
| FEAT-03.SPEC-005 (Auto-Adoption at Week Start) | Triggers (outbound) | Adoption's status change is propagated via this integration |
| FEAT-03.SPEC-008 (Plan Approval Authorization Rule) | Triggers (outbound) | Approval's status change is propagated via this integration |
| FEAT-04.SPEC-004 (Apply Meal Swap) | Triggers (outbound) | Accepted swaps and suggestions propagate via this integration |
| FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | Triggers (outbound) | Safety removals propagate via this integration and always win conflicts |
| FEAT-06.SPEC-005 (Shared Grocery List's live sync integration) | References (sibling) | Covers the same capability category for Grocery List data; this spec covers Weekly Plan and Planned Meal data only |

## Analytics and Success Signals

- **plan_sync_update_delivered** (latency, source: generation / approval / adoption / swap / safety_removal) -- N/A -- no Stage 2 metric in this feature's connected slice measures sync latency directly; retained to observe the near-instant propagation the product depends on for trust
- **plan_sync_conflict_resolved** (resolution: reject_with_refresh / safety_wins) -- N/A -- no Stage 2 metric measures conflict frequency; retained to observe how often concurrent plan changes occur across a household
- **plan_sync_degradation_shown** (condition: slow / down / rejected) -- N/A -- no Stage 2 metric measures degradation frequency directly; retained so the product's tolerance for sync trouble is observable

## Acceptance Criteria

**FEAT-03.SPEC-011-AC-01:** Given Maya accepts a swap suggestion on her device, when the change is confirmed, then Sam's device reflects the updated dinner card within the near-instant window without a manual refresh.

**FEAT-03.SPEC-011-AC-02:** Given a safety concern removes a dinner from the plan on one device, when the removal is confirmed, then every other household member's open plan view updates to show the removal live.

**FEAT-03.SPEC-011-AC-03:** Given a device was offline and reconnects, when reconciliation runs, then its plan view updates to the latest server-side state, discarding any now-stale local view.

**FEAT-03.SPEC-011-AC-04:** Given two devices attempt to swap the same dinner slot at effectively the same time, when the first swap is confirmed, then the second device's attempt is rejected with "This dinner just changed -- here's the latest." and its screen refreshes to the confirmed recipe.

**FEAT-03.SPEC-011-AC-05:** Given a safety removal and a swap acceptance target the same slot concurrently, when both are evaluated, then the safety removal wins and the swap acceptance is rejected.

**FEAT-03.SPEC-011-AC-06:** Given the sync capability is down, when Maya's device shows the current plan, then it falls back to its last-synced state with the Offline/Degraded messaging, and the plan remains viewable.

**FEAT-03.SPEC-011-AC-07:** Given the sync capability is slow but not down, when a change occurs elsewhere, then the update eventually appears without an error, even if delayed beyond the near-instant window.

**FEAT-03.SPEC-011-AC-08:** Given Maya's approval was not confirmed as propagated when connectivity was lost, when connectivity returns, then the approval is retried and other devices then show the Approved state.

**FEAT-03.SPEC-011-AC-09:** Given a device was offline through a full weekly regeneration, when it reconnects, then it reconciles directly to the new week's plan rather than replaying actions against the archived prior week.

**FEAT-03.SPEC-011-AC-10:** Given the same plan-update event is delivered to a device twice, when the second delivery arrives, then no duplicate visual change or toast occurs.

**FEAT-03.SPEC-011-AC-11:** Given a sync event arrives for a Weekly Plan that has since been archived, when it is processed, then it is discarded for the current-week display and does not alter the archived record's historical state.

**FEAT-03.SPEC-011-AC-12:** Given Riley (Operator) views a household's plan under an open Support Request, when plan changes occur elsewhere, then Riley's read-only view updates live exactly as any other viewer's would, without gaining any write access.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 3 | 3 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 3 (one screen x three conditions) | 3 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 6 | 6 |
