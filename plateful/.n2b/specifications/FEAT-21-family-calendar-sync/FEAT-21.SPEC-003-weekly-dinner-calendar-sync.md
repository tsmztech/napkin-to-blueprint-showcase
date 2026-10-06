---
document_type: spec
spec_type: automation
spec_id: FEAT-21.SPEC-003
spec_name: Weekly Dinner Calendar Sync
spec_slug: weekly-dinner-calendar-sync
parent_feature: FEAT-21
parent_feature_name: Family Calendar Sync
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

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
