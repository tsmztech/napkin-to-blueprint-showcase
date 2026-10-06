---
document_type: spec
spec_type: automation
spec_id: FEAT-05.SPEC-002
spec_name: Used-It-Up Prompt Trigger
spec_slug: used-it-up-prompt-trigger
parent_feature: FEAT-05
parent_feature_name: Pantry-Aware Suggestions
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Used-It-Up Prompt Trigger

## Overview

**Name:** Used-It-Up Prompt Trigger
**ID:** FEAT-05.SPEC-002
**Type:** Automation
**Purpose:** Once the dinner that used a logged Pantry Item has passed, surfaces a one-tap "used it up?" prompt on the pantry list the next time it is viewed.
**Parent Feature:** FEAT-05 -- Pantry-Aware Suggestions

## Scope and Non-Goals

**In Scope:**
- Detecting that a dinner calling out one or more logged Pantry Items has passed (the night has ended)
- Setting the used_prompt marker on each Pantry Item that dinner used, so FEAT-05.SPEC-001 renders the "used it up?" chip
- Ensuring the prompt appears at most once per item until it is acted on or the item is cleared some other way

**Non-Goals:**
- Clearing the item when the prompt is tapped -- handled inline by FEAT-05.SPEC-001 (Pantry List & Item Entry), which owns the tap-to-clear interaction
- Determining which Pantry Items a dinner uses -- handled by FEAT-05.SPEC-006 (Pantry-to-Recipe Matching for Plan Callout), which this automation reads from rather than recomputes
- Automatically removing a pantry item that goes unused for weeks -- excluded per the Feature Breakdown Brief's own Non-Goals: an item that goes unused is never auto-removed; this automation only reacts to a dinner that already used the item having passed, it does not judge staleness

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A dinner with a pantry callout passes (the night ends) | FEAT-03 AI Weekly Dinner Plan Generation (Planned Meal night-passage) | Fires once per Planned Meal, when its scheduled night has ended, only if that Planned Meal carries a non-empty pantry_callout (set by FEAT-05.SPEC-006) | The Planned Meal's pantry_callout (the list of Pantry Items it used), the household, and the night that passed |
| Household member opens the pantry list | FEAT-05.SPEC-001 (Pantry List & Item Entry) | Fires on every screen load, to render any prompt already set | Every Active Pantry Item for the household and its used_prompt marker |

## Processing Logic

1. When a Planned Meal's scheduled night ends, read its pantry_callout (the Pantry Items it was recorded as using, per FEAT-05.SPEC-006).
2. For each Pantry Item named in the pantry_callout, check that the item is still Active (it has not already been cleared manually since the dinner was planned).
3. For each still-Active item, set its used_prompt marker to indicate the prompt should be shown, without changing the item's status (it remains Active until the household acts).
4. When the pantry list (FEAT-05.SPEC-001) is next opened, read the used_prompt marker on every Active item and render the "used it up?" chip on each one that carries it.
5. The marker is cleared only when the item leaves Active status (cleared manually or via the prompt itself, per FEAT-05.SPEC-001) -- it is not reset or reapplied by a later dinner using the same item name again while the marker is already set.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Prompt surfaced | The consuming dinner's night has passed and the item is still Active | Pantry Item's used_prompt marker is set | "used it up?" chip appears on the item's row the next time the pantry list is opened | FEAT-05.SPEC-001 |
| No prompt needed (item already cleared) | The item was manually cleared before its consuming dinner's night passed | No change -- the item no longer exists to mark | Nothing -- the item is already gone from the list | FEAT-05.SPEC-001 |
| No prompt needed (dinner used no pantry items) | The Planned Meal's pantry_callout is empty | No change | Nothing | -- |
| Automation failure | The night-passage check cannot complete for a given Planned Meal | No used_prompt markers are set for that meal's items this cycle | No prompt appears for the affected items on the next pantry list view; the underlying Pantry Item data is unaffected (no item is incorrectly cleared) | FEAT-05.SPEC-001 |

## Data Model

**Reads:** Planned Meal -- night, pantry_callout (per FEAT-05.SPEC-006). Pantry Item -- status (to confirm still Active).
**Creates:** None.
**Updates:** Pantry Item -- used_prompt marker set to indicate the prompt should display.
**Deletes:** None -- clearing the item is owned by FEAT-05.SPEC-001, not this automation.

## Business Rules

- The prompt is purely a marker read by FEAT-05.SPEC-001; this automation never changes a Pantry Item's status itself.
- A Pantry Item can carry at most one active used_prompt marker at a time; the marker persists until the item is cleared (per FEAT-05.SPEC-001's manual-clear or prompt-tap interaction), it is not re-triggered by a second dinner using the same item name while the marker is already pending.
- This automation runs independently per household and per Planned Meal; it does not batch across households.
- No notification is sent for this prompt (per the feature's own Communications field: "N/A -- this feature has no notifications of its own"); it surfaces only in-app, on the pantry list.

## Edge Cases

- **The consuming dinner is swapped after the callout was recorded but before the night passes** -- The new dinner's own pantry callout (recomputed by FEAT-05.SPEC-006 for the swap) is what this automation reads at night-passage; if the swapped-in dinner no longer uses the item, no prompt is set for it from this slot.
- **The item is cleared manually before the consuming dinner's night passes** -- Step 2 finds the item no longer Active and skips it; no marker is set, and no prompt appears (nothing to prompt for).
- **Two different dinners in the same week both used the same logged item** -- Whichever dinner's night passes first sets the used_prompt marker; if the household has not yet acted when the second dinner's night passes, the marker is simply confirmed as already set (no duplicate prompt, no error).
- **Household downgrades or loses paid-tier access before the dinner's night passes** -- The night-passage check still runs against whatever pantry_callout was already recorded on the Planned Meal; a downgrade does not retroactively clear a callout already shown to the household (per XBR-05, no past pantry-related data is removed on downgrade).
- **Concurrent trigger firing (the night-passage check for two different Planned Meals in the same household fires at effectively the same time)** -- Each Planned Meal's items are marked independently; there is no shared state between the two runs that could conflict.
- **Trigger fires while a previous run is in flight (the pantry list is opened while the night-passage marker-set for that same night is still being written)** -- The pantry list read (step 4) reflects whichever markers have been committed at read time; a marker written moments after the list loads simply appears the next time the list is opened or refreshed, live-updating per FEAT-05.SPEC-001's list behavior.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03 AI Weekly Dinner Plan Generation (Planned Meal lifecycle) | Triggered by (inbound) | Fires when a Planned Meal's night passes |
| FEAT-05.SPEC-006 (Pantry-to-Recipe Matching for Plan Callout) | References (inbound) | Supplies the pantry_callout this automation reads to know which items a dinner used |
| FEAT-05.SPEC-001 (Pantry List & Item Entry) | Affects (outbound) | Renders the "used it up?" chip this automation surfaces, and owns the tap that clears the item |

## Analytics and Success Signals

- **pantry_used_prompt_surfaced** (item count, night the consuming dinner used) -- supports success-metrics.md: "Pantry Items Used"
- **pantry_used_prompt_skipped_already_cleared** (item name context, N/A beyond count) -- N/A -- this event tracks internal automation reach, not a metric in success-metrics.md; it exists to confirm the marker step is not silently failing
- **pantry_used_prompt_answered** (outcome: cleared) -- supports success-metrics.md: "Pantry Items Used" (this event is emitted by FEAT-05.SPEC-001 when the chip is tapped, and is listed here for completeness of the prompt's full lifecycle)

## Acceptance Criteria

**FEAT-05.SPEC-002-AC-01:** Given Maya's household has a Thursday dinner whose pantry_callout includes "spinach, feta" (set by FEAT-05.SPEC-006), when Thursday night passes, then the "spinach, feta" Pantry Item's used_prompt marker is set.

**FEAT-05.SPEC-002-AC-02:** Given the "spinach, feta" item's used_prompt marker was set in AC-01, when Maya next opens the Pantry List screen, then the "used it up?" chip appears on that row.

**FEAT-05.SPEC-002-AC-03:** Given Sam cleared "spinach, feta" manually on Wednesday, when Thursday's dinner night then passes, then no used_prompt marker is set (the item no longer exists) and no prompt appears.

**FEAT-05.SPEC-002-AC-04:** Given a Thursday dinner's pantry_callout is empty (it used no logged pantry items), when Thursday night passes, then no Pantry Item receives a used_prompt marker as a result of that dinner.

**FEAT-05.SPEC-002-AC-05:** Given Maya swaps Thursday's dinner on Wednesday for a recipe that does not use "spinach, feta" (FEAT-04), when Thursday night passes, then no used_prompt marker is set for "spinach, feta" from that slot.

**FEAT-05.SPEC-002-AC-06:** Given "eggs" is used by both Tuesday's and Thursday's dinners in the same week and the household has not acted on Tuesday's prompt, when Thursday night then passes, then "eggs" still carries exactly one used_prompt marker, and no duplicate chip or error appears.

**FEAT-05.SPEC-002-AC-07:** Given a household downgrades from paid to free tier on Monday, when a dinner whose pantry_callout was recorded before the downgrade has its night pass on Tuesday, then the used_prompt marker is still set as normal (no past pantry data is removed on downgrade, per XBR-05).

**FEAT-05.SPEC-002-AC-08:** Given the night-passage check for two different Planned Meals in the same household fires at effectively the same time, when both complete, then each Planned Meal's items are marked independently with no conflict or lost update.

**FEAT-05.SPEC-002-AC-09:** Given the night-passage marker-set for a given night is still being written, when Sam opens the Pantry List screen at that exact moment, then the screen shows whichever markers are already committed, and any marker written moments later appears live without requiring a manual refresh.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
