# FEAT-06 — Shared Grocery List

This chapter covers FEAT-06, Shared Grocery List, a Core-tier feature. It contains 9 specifications carrying 104 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-06.SPEC-001 | Grocery List | screen | 20 |
| FEAT-06.SPEC-002 | Grocery List Generation & Recalculation | automation | 12 |
| FEAT-06.SPEC-003 | "Already Have It" Handling | automation | 8 |
| FEAT-06.SPEC-004 | Week Rollover & Carryover | automation | 8 |
| FEAT-06.SPEC-005 | Live Grocery List Sync | integration | 12 |
| FEAT-06.SPEC-006 | Ingredient Consolidation & Quantity Derivation | logic-rule | 10 |
| FEAT-06.SPEC-007 | Manual Item Validation & Duplicate Merge | logic-rule | 11 |
| FEAT-06.SPEC-008 | Offline Conflict Resolution Rules | logic-rule | 10 |
| FEAT-06.SPEC-009 | Grocery List Access & Authorization Rules | logic-rule | 13 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Shared Grocery List

## Summary

**Feature:** Shared Grocery List
**ID:** FEAT-06
**Description:** One combined grocery list, generated from the week's plan and grouped by supermarket aisle, that the whole household sees and ticks off together in real time — including in the store with a weak signal.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** The brief describes this as one of the two concrete deliverables of the product (alongside the plan itself) and states it "must feel instant and must keep working in a supermarket with bad signal" (BRIEF.md, Vision, Scale & Non-Functional Expectations). It directly replaces the group-chat list called out in the Problem Statement. A real-time shared household list is the most-praised feature wherever it is done well, while the one competitor combining AI plans with household sharing reports members seeing the plan but not the matching list — so every member must always see the same plan and the same list.

**Key Capabilities:**
- Auto-generate from the plan — The list is built from the ingredients of the week's plan, grouped by aisle
- Add manually — Any household member can add an item the plan didn't include
- Tick off live — Ticking an item is visible to every household member immediately
- Work offline in the store — Changes made with no signal sync automatically once connectivity returns
- Combine repeated ingredients — The same ingredient needed by several dinners appears once, with the combined quantity
- Edit or remove an item — Any member with list access corrects a quantity or removes an item
- "Already have it" — A member marks a plan-derived item as already at home; it leaves the list and can be added to the pantry in the same tap

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-06.SPEC-001 | Grocery List | Screen | Maya, Sam, Jordan (older kid, Later), Riley | Household member views the combined, aisle-grouped list and ticks, adds, edits, removes, or marks "already have it" on items |
| FEAT-06.SPEC-002 | Grocery List Generation & Recalculation | Automation | Maya, Sam, Jordan (older kid, Later), Riley | Builds the list from the current week's plan on generation or pick, and recalculates it whenever the plan, a swap, a safety removal, or pantry data changes, while preserving manual items, ticks, and "already have it" marks |
| FEAT-06.SPEC-003 | "Already Have It" Handling | Automation | Maya, Sam, Jordan (older kid, Later) | Removes a plan-derived item from the list and, for members with pantry access, logs it to the pantry in the same tap |
| FEAT-06.SPEC-004 | Week Rollover & Carryover | Automation | Maya, Sam, Jordan (older kid, Later), Riley | Archives the current week's list at week end and carries unticked manually-added items into the new week's list |
| FEAT-06.SPEC-005 | Live Grocery List Sync | Integration | Maya, Sam, Jordan (older kid, Later), Riley | Category: real-time data-synchronization capability — pushes every list change to household devices within seconds and reconciles changes made while offline once connectivity returns |
| FEAT-06.SPEC-006 | Ingredient Consolidation & Quantity Derivation | Logic/Rule | Maya, Sam, Jordan (older kid, Later), Riley | Derives the plan-derived portion of the list: combines the same ingredient across dinners into one line with a household-sized combined quantity, minus logged pantry items |
| FEAT-06.SPEC-007 | Manual Item Validation & Duplicate Merge | Logic/Rule | Maya, Sam, Jordan (older kid, Later) | Validates a manually added item's name and merges a duplicate manual entry for the same ingredient into the existing line |
| FEAT-06.SPEC-008 | Offline Conflict Resolution Rules | Logic/Rule | Maya, Sam, Jordan (older kid, Later) | Governs how offline and concurrent changes resolve on sync: idempotent ticks, last-write-wins quantity edits, and no duplicate lines on reconnect |
| FEAT-06.SPEC-009 | Grocery List Access & Authorization Rules | Logic/Rule | Maya, Sam, Jordan (young kid, None), Jordan (older kid, Later), Riley | Defines who can view, add, tick, edit, or remove list items, and what an unauthorized visitor sees instead |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Auto-generate from the plan | FEAT-06.SPEC-002, FEAT-06.SPEC-006 | Generation & Recalculation builds the list from the plan's ingredients; Consolidation derives the combined, aisle-grouped lines | Phase 2 (Explicit) |
| Add manually | FEAT-06.SPEC-001, FEAT-06.SPEC-007 | Grocery List screen's add control, governed by name validation and duplicate-merge rules | Phase 2 (Explicit) |
| Tick off live | FEAT-06.SPEC-001, FEAT-06.SPEC-005 | Tick action on the list, propagated to every household device by the live sync integration | Phase 2 (Explicit) |
| Work offline in the store | FEAT-06.SPEC-005, FEAT-06.SPEC-008 | Live sync queues changes locally when offline; conflict resolution rules govern how queued changes merge on reconnect | Phase 2 (Explicit) |
| Combine repeated ingredients | FEAT-06.SPEC-006 | Ingredient Consolidation combines the same ingredient across dinners into one line, sized for the household | Phase 2 (Explicit) |
| Edit or remove an item | FEAT-06.SPEC-001, FEAT-06.SPEC-008 | Edit/remove controls on the list; conflict resolution rules govern concurrent quantity edits | Phase 2 (Explicit) |
| "Already have it" | FEAT-06.SPEC-003 | "Already Have It" Handling removes the item and, where the member has pantry access, logs it to the pantry in the same tap | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-06.SPEC-002 | Grocery List Generation & Recalculation | Phase 4 (Trigger-Response) | Beyond the initial build, the plan-derived list must recalculate on every swap, pick, safety removal, and pantry change (XBR-03) while preserving manual items, ticks, and "already have it" marks — a merge policy the Key Capability alone does not state |
| FEAT-06.SPEC-003 | "Already Have It" Handling | Phase 4 (Trigger-Response, cross-feature side effect) | The tap crosses into pantry data ownership (FEAT-05, XBR-04); the older-kid role's Pantry Input access is None, so the same tap behaves differently by role — a rule the Key Capability line does not surface |
| FEAT-06.SPEC-004 | Week Rollover & Carryover | Phase 3 (Entity-Lifecycle Analysis) | The CRUD matrix's Delete/Archive cell for Grocery List was empty until the Primary Flows & Alternates "Week rollover" line and the Grocery List archive lifecycle in the dependency map were elaborated into a standalone automation |
| FEAT-06.SPEC-005 | Live Grocery List Sync | Phase 4 (External Dependencies lens) | The Dependencies slice of assumptions-constraints.md (ASMP-35) names a real-time data-synchronization capability this feature relies on, and the dependency map's External Touchpoints table lists FEAT-06 first for it, with FEAT-04 relying on it indirectly through this feature |
| FEAT-06.SPEC-006 | Ingredient Consolidation & Quantity Derivation | Phase 5 (Rule Discovery) | The combine-across-dinners calculation and the pantry-exclusion subtraction (XBR-04) are non-trivial derivation logic shared by the generation automation and the list screen, crossing the standalone-spec threshold |
| FEAT-06.SPEC-007 | Manual Item Validation & Duplicate Merge | Phase 5 (Rule Discovery) | Validation & Limits names a required-field rule and a duplicate-merge rule that both the add action and the recalculation automation must honor consistently |
| FEAT-06.SPEC-008 | Offline Conflict Resolution Rules | Phase 5 (Rule Discovery) / Phase 6 (Failure Analysis) | Idempotent ticks, last-write-wins quantity edits, and duplicate-free reconnect sync (ASMP-25) are conditional rules shared across the screen, the generation automation, and the sync integration — and the journey's offline failure/recovery variant requires them to be explicit |
| FEAT-06.SPEC-009 | Grocery List Access & Authorization Rules | Phase 5 (Rule Discovery) | The Access field states five distinct access levels across roles, including a role (older kid, Later) with full add/tick access but no household-setting access — an authorization surface shared by the screen and every write automation |

## Entity-Lifecycle Coverage Matrix

**Entity: Grocery List**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-06.SPEC-002 | Generation & Recalculation creates the week's list the moment a plan is generated (FEAT-03) or started manually (FEAT-23) | -- |
| Read (single) | FEAT-06.SPEC-001 | Grocery List screen displays the current week's list | -- |
| Read (list) | N/A | This feature always operates on the current week's list; browsing past weeks' lists is Weekly Plan History's responsibility (FEAT-19) | -- |
| Update | FEAT-06.SPEC-002 | Recalculation updates the plan-derived portion whenever the plan, a swap, a safety removal, or pantry data changes | -- |
| Delete/Archive | FEAT-06.SPEC-004 | Soft archive at week end: the list's status moves to Archived and it is retained in full (not hard-deleted) as part of the household's history, browsable for the life of the account (scope-boundaries.md SC-18); no restore path is needed because an archived list is never removed, only superseded by the new week's list; cascade on household deletion is owned by FEAT-18; no purge policy applies while the account is active | -- |
| State Transition | FEAT-06.SPEC-002, FEAT-06.SPEC-004 | Generated -> Active (SPEC-002, on first population) -> Archived (SPEC-004, at week end) | -- |

**Entity: Grocery List Item**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-06.SPEC-002, FEAT-06.SPEC-001 | Plan-derived items are created by recalculation (SPEC-002, via SPEC-006's consolidation); manual items are created from the list screen's add control (SPEC-001, governed by SPEC-007) | -- |
| Read (single) | FEAT-06.SPEC-001 | List screen shows each line's ingredient, quantity, aisle, origin, and "added by" label | -- |
| Read (list) | FEAT-06.SPEC-001 | List screen shows the full aisle-grouped set of items | -- |
| Update | FEAT-06.SPEC-001 | Tick/untick and quantity-edit actions on the list, subject to SPEC-008's conflict rules; recalculation (SPEC-002) rewrites plan-derived lines in place | -- |
| Delete/Archive | FEAT-06.SPEC-001, FEAT-06.SPEC-004 | Hard delete on manual removal from the list screen: immediate, no restore path (a member who removed an item in error re-adds it; this is a deliberate simplicity choice, not an oversight — recorded as an explicit non-goal), no cascade to other entities; at week archive (SPEC-004), unticked manually-added items are carried into the new week's list rather than deleted, and all items are retained as part of the archived list's history for the life of the account (SC-18) with no purge policy | -- |
| State Transition | FEAT-06.SPEC-001, FEAT-06.SPEC-003 | Unticked -> Ticked -> Unticked (SPEC-001, idempotent per SPEC-008); Active -> Removed via "already have it" (SPEC-003) | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Weekly Plan | FEAT-06.SPEC-002 | Locates the current week's approved or in-progress plan to derive the list from |
| Planned Meal | FEAT-06.SPEC-002, FEAT-06.SPEC-006 | Source of the dinners whose ingredients feed the plan-derived list lines |
| Recipe | FEAT-06.SPEC-002, FEAT-06.SPEC-006 | Source of each dinner's ingredient list and quantities |
| Pantry Item | FEAT-06.SPEC-002, FEAT-06.SPEC-003 | Recalculation excludes logged pantry items from the plan-derived list (XBR-04); "already have it" writes a new entry here for members with pantry access — Pantry Item itself is owned and managed by Pantry-Aware Suggestions (FEAT-05) |
| Household | FEAT-06.SPEC-001, FEAT-06.SPEC-006 | Supplies the household's configured aisle names and unit system, owned by Household Setup / Units, Currency & Locale Configuration (FEAT-16) |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| A plan is generated (FEAT-03) or started manually (FEAT-23) | Build the grocery list from the plan's ingredients, grouped by aisle | Standalone Automation | FEAT-06.SPEC-002 |
| A meal is swapped or a suggestion accepted (FEAT-04) | Recalculate the list's plan-derived lines | Cross-feature inbound (FEAT-04 triggers, FEAT-06 executes, XBR-03) | FEAT-06.SPEC-002 |
| A safety-concern report removes a meal (FEAT-02, XBR-08) | Drop the removed meal's ingredients from the list | Cross-feature inbound | FEAT-06.SPEC-002 |
| A pantry item is logged or cleared (FEAT-05, XBR-04) | Recalculate the list to exclude or restore the affected ingredient | Cross-feature inbound | FEAT-06.SPEC-002 |
| Household member ticks or unticks an item | Update ticked state; propagate to every household device within seconds | Inline in triggering screen + Standalone Integration | FEAT-06.SPEC-001 / FEAT-06.SPEC-005 |
| Household member adds an item manually | Validate name length; merge if a duplicate exists for the same ingredient; else create a new line | Standalone Logic/Rule | FEAT-06.SPEC-007 |
| Household member edits a quantity | Update the line; apply last-write-wins if a concurrent edit exists | Inline in triggering screen + Standalone Logic/Rule | FEAT-06.SPEC-001 / FEAT-06.SPEC-008 |
| Household member removes an item | Hard delete the line, no restore | Inline in triggering screen | FEAT-06.SPEC-001 |
| Household member marks "already have it" | Remove the item from the list; for members with pantry access, log it to the pantry in the same tap | Standalone Automation | FEAT-06.SPEC-003 |
| Older-kid login (Later) marks "already have it" | Remove the item from the list; no pantry entry is created (Pantry Input access is None for this role) | Standalone Automation (role-differentiated) | FEAT-06.SPEC-003 |
| Household member ticks or edits while offline | Hold the change locally; keep list read/tick/add fully usable | Standalone Integration | FEAT-06.SPEC-005 |
| Connectivity returns after offline changes | Sync queued changes; merge without duplicates or conflicts | Standalone Integration + Standalone Logic/Rule | FEAT-06.SPEC-005 / FEAT-06.SPEC-008 |
| The same ingredient is added twice manually | Merge into one existing line rather than duplicating | Standalone Logic/Rule | FEAT-06.SPEC-007 |
| The same ingredient is needed by several dinners | Combine into one line with a household-sized combined quantity | Standalone Logic/Rule | FEAT-06.SPEC-006 |
| The current week ends | Archive the current list; carry unticked manually-added items into the new week's list | Standalone Automation | FEAT-06.SPEC-004 |
| An unauthorized visitor attempts to open the list | Show a sign-in screen instead; no household data is exposed | Standalone Logic/Rule (Permission Denied state) | FEAT-06.SPEC-009 |
| Riley (Operator, support) opens the list against an open Support Request | Render the list read-only, with no tick/add/edit/remove controls | Standalone Logic/Rule | FEAT-06.SPEC-009 |

## Shared Context

**Shared Entities:**
- Grocery List -- created and updated by SPEC-002, archived by SPEC-004, read by SPEC-001. Fields: week, aisle_grouping, status.
- Grocery List Item -- created by SPEC-002 (plan-derived) and SPEC-001/SPEC-007 (manual); read and updated (tick, edit) by SPEC-001; updated by SPEC-002 (recalculation) and SPEC-003 (already-have-it removal); carried forward or retained by SPEC-004. Fields: ingredient_name, quantity_and_unit, aisle, origin, ticked.
- Pantry Item (owned by FEAT-05) -- read by SPEC-002 for exclusion; written by SPEC-003 for members with pantry access.

**Shared UI Patterns:**
- Aisle-grouped line item -- used throughout SPEC-001: every line (plan-derived or manual) renders with the same ingredient/quantity/aisle layout, an "added by" label for manual items, and the same tick, edit, remove, and "already have it" controls, so the household never has to learn a second pattern for either kind of item.
- Inline loading and error indicators -- the same "populating within a couple of seconds" indicator and the same background-retry-then-surface-only-on-final-failure pattern apply to every write action on SPEC-001 (tick, add, edit, remove, already-have-it), per product-features.md States.

**Shared Validation/Logic:**
- FEAT-06.SPEC-006 defines how the plan-derived portion of the list is computed; SPEC-002 invokes it on every recalculation rather than re-deriving the combine-and-subtract logic.
- FEAT-06.SPEC-007 defines manual-item validation and duplicate merge; SPEC-001's add control and SPEC-002's recalculation (when a plan-derived line and a manual line name the same ingredient) both reference it.
- FEAT-06.SPEC-008 defines conflict resolution; SPEC-001, SPEC-002, and SPEC-005 all reference it rather than each defining their own merge behavior for ticks, edits, and reconnect sync.
- FEAT-06.SPEC-009 defines access and authorization; SPEC-001, SPEC-002, and SPEC-003 all reference it to determine what a given role may view or do.

**Shared Signals:** grocery_list_generated (SPEC-002), grocery_items_combined (SPEC-006), grocery_item_added_manually (SPEC-001/SPEC-007), grocery_item_ticked (SPEC-001), grocery_item_edited (SPEC-001/SPEC-008), grocery_item_removed (SPEC-001), grocery_item_already_have (SPEC-003), grocery_item_synced_after_offline (SPEC-005), grocery_items_carried_over (SPEC-004) -- product-features.md Signals, carried verbatim into each owning spec's analytics linkage.

## Internal Dependency Map

```
SPEC-002 (Generation & Recalculation) -> [plan generated, picked, swapped, safety-removed, or pantry changed] -> SPEC-006 (Ingredient Consolidation) -> [returns combined, aisle-grouped lines] -> SPEC-002 -> [writes lines] -> SPEC-001 (Grocery List)
SPEC-001 -> [member taps add] -> SPEC-007 (Manual Item Validation & Duplicate Merge) -> [valid / merged / new line] -> SPEC-001
SPEC-001 -> [member ticks, edits, or removes] -> SPEC-008 (Offline Conflict Resolution Rules) [governs concurrent/offline writes] -> SPEC-001
SPEC-001 -> [every tick, add, edit, remove] -> SPEC-005 (Live Grocery List Sync) -> [propagates to household devices within seconds]
SPEC-005 -> [connectivity returns after offline changes] -> SPEC-008 [merge without duplicates] -> SPEC-001
SPEC-001 -> [member taps "already have it"] -> SPEC-003 ("Already Have It" Handling) -> [removes item; logs to pantry if role has pantry access] -> SPEC-001
SPEC-004 (Week Rollover & Carryover) -> [week ends] -> SPEC-002 [archives current list, seeds new list] -> [carries unticked manual items] -> SPEC-001
SPEC-001, SPEC-002, SPEC-003 -> [determine what a role may view or do] -> SPEC-009 (Access & Authorization Rules)
```

**Default Entry:** FEAT-06.SPEC-001 (Grocery List) -- the screen shown when a household member navigates to the grocery list, whether from the approved weekly plan, a manually-built week, or the first-use landing screen.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-06.SPEC-002 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | List is generated from the newly approved or auto-adopted plan | Plan approved or adopted (Sunday Plan Review, step 5) |
| FEAT-06.SPEC-002 | Inbound | FEAT-23 (Manual Weekly Planning) | List fills in as nights are picked | Household picks a night's meal (Free-Tier Manual Week, step 5) |
| FEAT-06.SPEC-002 | Inbound | FEAT-04 (One-Tap Meal Swap) | List recalculates the moment a swap or accepted suggestion completes | Swap or suggestion-acceptance completes (XBR-03) |
| FEAT-06.SPEC-002 | Inbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | List drops a removed meal's ingredients after a safety concern or a tightened hard rule | Safety removal or mid-week rule tightening (XBR-02, XBR-08) |
| FEAT-06.SPEC-002 | Inbound | FEAT-05 (Pantry-Aware Suggestions) | List excludes ingredients already logged in the pantry | Pantry item logged or cleared (XBR-04) |
| FEAT-06.SPEC-003 | Outbound | FEAT-05 (Pantry-Aware Suggestions) | "Already have it" logs the item to the pantry for members with pantry access | Member taps "already have it" (XBR-04) |
| FEAT-06.SPEC-001, FEAT-06.SPEC-006 | Inbound | FEAT-16 (Units, Currency & Locale Configuration) | List consumes the household's aisle names and unit system for grouping and quantities | Every list render or recalculation (XBR-11) |
| FEAT-06.SPEC-001 | Inbound | FEAT-15 (first-use landing) | New member lands directly on the current list rather than a blank screen | First sign-in after joining (Invite & Join Household, step 3) |
| FEAT-06.SPEC-001, FEAT-06.SPEC-004 | Outbound | FEAT-19 (Weekly Plan History) | Past weeks' archived lists are browsed from history | Household opens a past week's history entry |
| FEAT-06.SPEC-001 | Outbound (deferred) | FEAT-20 (Online Grocery Ordering Handoff, Later) | List hands off to an ordering flow | Member taps "hand off list" (Later phase) |
| FEAT-06.SPEC-009 | Outbound | FEAT-22 (Operator Read-Only Support Access) | Riley's read-only support view of the list is governed by this feature's access rules | Riley opens the household's list against an open Support Request |
| FEAT-06.SPEC-005 | Outbound (dependency) | FEAT-04 (One-Tap Meal Swap) | FEAT-04 relies on this feature's real-time synchronization capability to propagate a swapped plan and list live | Every swap completion |

## Non-Functional Notes

**Data volumes / growth:** Several thousand households in the first year, 2-6 members each; each household's Grocery List and its Items are modest in volume (one active list per household plus manually-added lines) but every past week's list is kept for the life of the account and remains browsable after a downgrade to the free tier (assumptions-constraints.md ASMP-24; scope-boundaries.md SC-18).

**Responsiveness:** The list must feel instant when items are ticked or added — a tick or manual add is visible to another household member within 2 seconds under normal connectivity (assumptions-constraints.md ASMP-22; success-metrics.md, Grocery List Live-Update Trust). A meal swap or accepted suggestion is reflected in the list within seconds of completing (ASMP-22; success-metrics.md, One-Tap Swap Completion). Newly generated list items appear within a couple of seconds with an inline loading indicator (product-features.md, States).

**Data sensitivity / privacy:** Grocery List and Grocery List Item hold low-sensitivity household personal data — private to the household and never sold or used for advertising (feature-dependency-map.md, Grocery List / Grocery List Item Data Sensitivity; assumptions-constraints.md ASMP-14, ASMP-26). The "added by" label on manual items shows a member's name only within the household. Children's data stays minimal: the young-kid profile has no login and never appears as an "added by" source; the Later-phase older-kid login's name may appear on items it adds, consistent with the privacy posture that a kid profile's data is used only for the household's own plan (ASMP-26).

**Compliance flags:** No health, financial, or other regulated data is processed by this feature; the general no-sale, no-advertising posture applies (assumptions-constraints.md ASMP-14, ASMP-26). Every primary action (tick, add, edit, remove, already-have-it) must be reachable with one thumb, with large tap targets suited to in-store, one-handed use, and text that stays readable at larger accessibility text sizes (assumptions-constraints.md ASMP-29). The product is English-only, with units, currency, and aisle names configurable per household rather than translated (assumptions-constraints.md ASMP-28; scope-boundaries.md SC-13) — those settings are owned and specified by FEAT-16, not by this feature.

## Non-Goals

- **Household-level aisle names and unit-system settings** -- Owned by Household Setup / Units, Currency & Locale Configuration (FEAT-16) per XBR-11 and the feature-dependency-map.md authority column; this feature consumes those settings when grouping and sizing the list but never lets a household member change them here, including the older-kid (Later) role, whose Full access is explicitly scoped to adding and ticking, not household list settings (product-features.md, Access).
- **Online grocery ordering or checkout** -- Excluded from this feature and deferred to Later per scope-boundaries.md's deferral note on Online Grocery Ordering Handoff (FEAT-20); BRIEF.md names it "desirable later, not v1." This feature's list is a hand-off point for that future capability, not the capability itself.
- **Meal-kit box sales or grocery delivery of physical ingredients** -- Excluded per scope-boundaries.md SC-07: the product plans and lists groceries; it does not sell or ship ingredients itself.
- **In-product messaging about list items** -- Excluded per scope-boundaries.md SC-14: the brief positions the live, shared list itself as the replacement for the group chat it names as the problem, not a chat layer bolted onto the list.
- **Undo for a manually removed item** -- Intentional simplicity decision surfaced by the CRUD matrix: removal is an immediate, hard delete with no restore path; a member who removed an item in error re-adds it, consistent with the product's "instant" responsiveness goal (assumptions-constraints.md ASMP-22) taking priority over building a recovery flow for a low-stakes, easily-corrected action.
- **Automatic purge of archived list history** -- Intentional lifecycle decision surfaced by the CRUD matrix: archived lists and their items are retained for the life of the household account with no automatic purge, consistent with scope-boundaries.md SC-18's account-life retention for plan and list history.
- **A distinct native-app list experience** -- Excluded per scope-boundaries.md SC-05: the product ships as a responsive web app for v1, with no native apps.



# Screen Spec: Grocery List

## Overview

**Name:** Grocery List
**ID:** FEAT-06.SPEC-001
**Type:** Screen
**Purpose:** Household member views the current week's combined, aisle-grouped grocery list and ticks, adds, edits, removes, or marks "already have it" on items, live and while offline.
**Parent Feature:** FEAT-06 -- Shared Grocery List

## Scope and Non-Goals

**In Scope:**
- Displaying the current week's Grocery List, grouped by the household's configured aisles
- Ticking and unticking items
- Adding a manual item
- Editing an item's quantity
- Removing an item
- Marking a plan-derived item "already have it"
- Full read/tick/add/edit/remove usability while offline, with automatic sync on reconnect

**Non-Goals:**
- Household aisle names and unit-system configuration -- owned by Units, Currency & Locale Configuration (FEAT-16); this screen only consumes those settings, per this feature's Non-Goals and XBR-11
- Online grocery ordering or checkout handoff -- deferred to Online Grocery Ordering Handoff (FEAT-20, Later), per scope-boundaries.md's deferral note; this screen only carries the future hand-off point
- Browsing a past week's archived list -- owned by Weekly Plan History (FEAT-19); this screen shows only the current week's list
- Manual item name validation and duplicate-merge logic -- defined by FEAT-06.SPEC-007; this screen calls it, not re-defines it
- Undo for a removed item -- excluded per feature-overview.md Non-Goals: removal is an immediate, hard delete with no restore path, a deliberate simplicity decision prioritizing the product's "instant" responsiveness goal over a recovery flow for a low-stakes, easily-corrected action

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-03 (AI Weekly Dinner Plan Generation) -- Weekly Plan screen | Household member opens the list from an approved or auto-adopted week's plan | Current week reference; list is already populated by FEAT-06.SPEC-002 |
| FEAT-23 (Manual Weekly Planning) -- week-being-built screen | Household member opens the list as picks fill the week | Current week reference; list fills incrementally as SPEC-002 recalculates on each pick |
| FEAT-15 (Member Onboarding) -- first-use landing | A new member lands directly on the current list after their first sign-in | None -- list loads to its current, already-populated state |
| Default entry (persistent navigation) | Household member navigates to the list directly at any time | None -- list loads to its current state |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | All actions: tick, add, edit quantity, remove, already-have-it | -- |
| Sam (Other Adult Member) | Full screen | All actions: tick, add, edit quantity, remove, already-have-it | -- |
| Jordan (young kid profile, no login -- MVP) | No -- this profile has no login and cannot reach any screen | No | This profile has no sign-in; the screen is never reached, not merely restricted |
| Jordan (older kid, limited login -- Later) | Full screen | All item actions: tick, add, edit quantity, remove, already-have-it (per FEAT-06.SPEC-009); no household aisle-setting controls are ever shown to this role, on any role | -- |
| Riley (Operator, support -- from v1) | Full screen, read-only, and only while an open Support Request exists for the household (XBR-14, FEAT-06.SPEC-009) | View only -- no tick, add, edit, remove, or already-have-it controls are rendered | Outside an open Support Request, Riley cannot open the household's list at all: the attempt is blocked and no household data is shown. While viewing, tapping where a control would be for another role does nothing -- no controls are present to tap |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on this screen if it was their intended destination |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- any in-progress add or edit text is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Grocery List" with the current week's date range shown beneath it. A "Hand off list" control appears in the header, shown only to Maya and Sam and only while FEAT-20.SPEC-003 evaluates the household's region as having an available online-ordering capability; the control is never shown to Jordan (either profile), Riley, or an ineligible region, and its eligibility is re-evaluated fresh on FEAT-20.SPEC-001's own load rather than trusted from this screen (per FEAT-20.SPEC-003). When the device is offline, a persistent banner appears directly below the header: "You're offline -- changes will sync when you reconnect." (see States).

**Body:** A single-column list grouped into aisle sections, ordered per the household's configured `aisle_grouping` (FEAT-16). Each aisle section has a non-interactive section header (the aisle name) followed by its item rows. Each item row shows:
- A tick control (checkbox) -- ticked items render with a strikethrough on the ingredient name
- The ingredient name and its `quantity_and_unit`
- An "added by {member name}" label, shown only on manual-origin items
- An overflow control (three-dot menu) offering: "Edit quantity", "Remove" (all items), and "Already have it" (plan-derived items only)

Riley's read-only view renders the same rows with the tick control, "added by" label, and overflow control all omitted -- items display as static text.

**Footer:** A persistent "Add item" row: a single-line text input with placeholder "Add an item" and an adjacent "Add" button, always visible at the bottom of the screen for one-handed, in-store use (per the product's accessibility posture for primary actions). Not shown to Riley.

### Responsive Behavior

- **Compact size class (phone, primary use case):** Single-column list as described, full width; the Add-item row is pinned to the bottom of the viewport so it remains reachable one-handed while scrolling.
- **Medium size class and above:** List remains single-column, capped at a consistent platform-wide content width and horizontally centered; aisle sections and the Add-item row are otherwise unchanged.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Tick control | Tap | Toggles the item's `ticked` state; propagated via FEAT-06.SPEC-005, subject to FEAT-06.SPEC-008's idempotent-tick rule | Row shows strikethrough (ticked) or plain text (unticked) | Immediate local update; no confirmation dialog |
| Add-item input | Type | Captures free text for a new item name | Input shows entered text | Standard input focus state |
| Add button | Tap | Submits the typed name to FEAT-06.SPEC-007 (Manual Item Validation & Duplicate Merge) | New row appears (new line) or an existing line updates (merge), per SPEC-007's outcome | Success: input clears, new/updated row appears in its aisle section. Failure: inline error below the input (see Validation Rules) |
| Overflow > Edit quantity | Tap | Opens an inline quantity field pre-filled with the current `quantity_and_unit` | Row switches to an editable quantity field with a confirm and cancel control | Standard input focus state |
| Edit quantity > Confirm | Tap | Saves the new quantity, subject to FEAT-06.SPEC-008's last-write-wins conflict rule | Row shows the updated quantity | Immediate local update; no confirmation dialog |
| Overflow > Remove | Tap | Immediately and permanently deletes the Grocery List Item -- no confirmation dialog, per this screen's Non-Goals | Row disappears from the list | Immediate; no undo offered |
| Overflow > Already have it (plan-derived items only) | Tap | Triggers FEAT-06.SPEC-003 ("Already Have It" Handling) | Row disappears from the list | Toast: "Marked as already have it -- added to your pantry." with a "View Pantry" action (Maya, Sam -- per FEAT-05.SPEC-007's Pantry Input gating) or "Marked as already have it." with no action (Jordan, older kid -- no pantry access) |
| Toast > View Pantry (Maya, Sam only) | Tap | Navigates to FEAT-05.SPEC-001 (Pantry List & Item Entry) | Screen navigates away | Animated transition to the Pantry List screen |
| Hand off list control (header, Maya/Sam only, region-eligible per FEAT-20.SPEC-003) | Tap | Navigates to FEAT-20.SPEC-001 (Grocery Handoff Screen) | Screen navigates away | Animated transition to the Handoff screen |
| Aisle section header | -- | Display only, non-interactive -- groups the rows beneath it | None | None |
| "Added by" label | -- | Display only, non-interactive | None | None |

### Accessibility Notes

- **Focus order:** Offline banner (when present) -> aisle sections in list order, each row's tick control -> ingredient name/quantity (read-only) -> overflow control -> Add-item input -> Add button.
- **Dynamic-change announcements:** A tick toggling, a new item appearing, an item being removed, and the "Already have it" toast are each announced to assistive technology as they occur, so a member using a screen reader hears the live list change without re-scanning it.
- **Keyboard alternatives:** Every action on this screen (tick, add, edit quantity, remove, already-have-it) is reachable by keyboard focus and activation; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (pre-plan) | Short message: "Your list will appear once this week's plan is ready. You can add items now." plus the Add-item row remains usable | No Grocery List exists yet for the household's current week | FEAT-06.SPEC-002 creates the list (plan generated or first pick made) |
| Populating | Newly generated items appear within a couple of seconds, with an inline loading indicator at the top of the list | A plan is generated or picked and FEAT-06.SPEC-002 is writing plan-derived lines | Generation completes and the full aisle-grouped list renders |
| Populated (default) | Full aisle-grouped list with all current items | List has one or more items | List becomes empty again (all items removed/ticked-and-cleared is not a state -- ticked items remain visible until removed) |
| Error | A failed tick, add, edit, or remove retries automatically in the background; an error surfaces only if it cannot eventually succeed, as a banner: "Couldn't save your change. Check your connection and try again." with a Retry option | A write action ultimately fails after background retries | Member taps Retry and the action succeeds, or the member dismisses the banner |
| Offline/Degraded | Persistent banner: "You're offline -- changes will sync when you reconnect." Full read, tick, add, edit, and remove remain usable; every change queues locally | Connectivity is lost while the screen is open, or the screen opens while already offline | Connectivity returns -- queued changes sync automatically (FEAT-06.SPEC-005) and the banner disappears |

## Validation Rules

**Option A -- Reference Logic/Rule spec:**
Manual item name validation and duplicate-merge behavior are governed by FEAT-06.SPEC-007. See that spec for the exact rules and error messages, applied on the Add-item input on submit.

**Option B -- Inline (quantity edit, not covered by SPEC-007):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Quantity edit field | Must not be left blank | On confirm | "Quantity can't be empty." |

## Navigation Out

This screen is primarily a persistent destination reached from the app's navigation shell (out of scope for this spec), not a step in a linear flow, and every action on this screen (tick, add, edit, remove) resolves inline without leaving the screen. It carries exactly two optional navigation-out targets, both gated to roles with the underlying access and both optional for the member to take:

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Hand off list" header control tap (Maya, Sam only, shown only while the household's region is handoff-eligible per FEAT-20.SPEC-003) | FEAT-20.SPEC-001 (Grocery Handoff Screen) | FEAT-20 (Online Grocery Ordering Handoff) |
| "View Pantry" action on the "already have it" toast (Maya, Sam only -- Jordan (older kid)'s toast carries no pantry mention and no action, since this role's Pantry Input access is None per FEAT-05.SPEC-007) | FEAT-05.SPEC-001 (Pantry List & Item Entry) | FEAT-05 (Pantry-Aware Suggestions) |

"Already have it" itself still triggers FEAT-06.SPEC-003 as a background automation on this screen, not a navigation; the row disappears here regardless of role, and only the follow-on toast action -- offered to Maya and Sam only -- leaves the screen.

## Data Model

**Creates:** Grocery List Item (manual) -- `ingredient_name` from the Add-item input, `quantity_and_unit` left blank unless later edited, `aisle` auto-assigned by matching `ingredient_name` against the household's configured aisle categories (FEAT-16), falling back to a household catch-all "Other" aisle when no match is found, `origin` set to "manual", `ticked` defaults to false. Attribution ("added by") is the creating member.

**Reads:** Grocery List -- `week`, `status`. Grocery List Item -- all fields, for every item on the current list. Household -- `aisle_grouping`, `unit_system` (via FEAT-16) to order sections and format quantities.

**Updates:** Grocery List Item -- `ticked` (tick/untick), `quantity_and_unit` (edit), subject to FEAT-06.SPEC-008's conflict rules; plan-derived lines are also rewritten in place by FEAT-06.SPEC-002's recalculation, shown live on this screen.

**Deletes:** Grocery List Item -- on manual removal (any origin) or as a side effect of FEAT-06.SPEC-003 ("Already have it").

## Business Rules

- XBR-03: the list is always derived from the current plan; every pick, change, swap, accepted suggestion, or safety removal updates the list immediately via FEAT-06.SPEC-002, visible on this screen without the member refreshing.
- XBR-04: pantry-excluded ingredients never appear as plan-derived lines (governed by FEAT-06.SPEC-006); "already have it" can add an item to the pantry in the same tap (FEAT-06.SPEC-003).
- XBR-11: aisle grouping and quantity units follow the household's configuration (FEAT-16) consistently across every rendered line.
- Manual item validation and duplicate merge are governed by FEAT-06.SPEC-007 -- the member cannot bypass them from this screen.
- Offline and concurrent-edit behavior is governed by FEAT-06.SPEC-008 -- this screen never resolves a conflict itself.
- Access to every action on this screen is governed by FEAT-06.SPEC-009 -- this screen's Access and Visibility table above is drawn from it and must not diverge.
- The "Hand off list" header control's visibility (region and role) is governed entirely by FEAT-20.SPEC-003 -- this screen never independently determines regional online-ordering availability or handoff role eligibility, and re-evaluates nothing itself; FEAT-20.SPEC-001 re-checks eligibility fresh on its own load regardless of whether the control was shown here.
- The "View Pantry" action on the "already have it" toast is offered only to a role with Pantry Input access (Maya, Sam), per FEAT-05.SPEC-007's Authorization Rules -- Jordan (older kid, limited login) receives the plain "Marked as already have it." toast with no pantry mention and no action, since a Pantry Item is never created on this role's behalf.

## Edge Cases

- **Household member taps tick twice rapidly, including once while briefly offline** -- The second tap on an already-ticked line is a no-op once synced; per FEAT-06.SPEC-008's idempotent-tick rule, the item settles into exactly one ticked state, not a double-toggle.
- **Another member ticks or edits the same item while this screen is open** -- The screen updates live via FEAT-06.SPEC-005; no conflict dialog appears to either member for a tick (idempotent) or a quantity edit (last-write-wins per FEAT-06.SPEC-008) -- the concurrent-edit outcome resolves silently and both screens converge to the same state.
- **Household member adds an ingredient that is already on the list (manual or plan-derived)** -- FEAT-06.SPEC-007 merges the entry into the existing line rather than creating a duplicate; the member sees the existing line's quantity update (manual-manual match) or a confirmation that the item is already on the list from the week's plan (manual-vs-plan-derived match).
- **Household member removes an item that another member just ticked moments earlier** -- Removal proceeds regardless of ticked state; the household has decided the item is no longer needed.
- **Network failure while adding an item** -- The add is held locally and retried automatically in the background; the new row appears optimistically and is confirmed once the write succeeds, or shows the Error state if it ultimately fails.
- **Member navigates away with an in-progress quantity edit** -- The edit field's typed value is not saved unless Confirm was tapped; navigating away discards the in-progress edit, consistent with this screen having no draft-persistence requirement for a low-stakes, single-field edit.
- **Member navigates away or backgrounds the app with unsubmitted text in the Add-item input (not tapped Add)** -- The typed text is discarded, not preserved and not restored, when the member returns to the screen or reopens the app; this is distinct from the Expired-session case (AC-14), where in-progress Add-item text is preserved specifically across a forced re-authentication interruption, not across an ordinary navigate-away or background/foreground cycle. This keeps the Add-item input consistent with the quantity-edit field above: no draft-persistence requirement for a low-stakes, single-field input outside the session-expiry recovery path.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-002 (Grocery List Generation & Recalculation) | Affects (inbound) | Writes and rewrites plan-derived lines shown on this screen |
| FEAT-06.SPEC-003 ("Already Have It" Handling) | Triggers (outbound) | "Already have it" overflow action fires this automation |
| FEAT-06.SPEC-004 (Week Rollover & Carryover) | Affects (inbound) | Seeds this screen's list with carried-over manual items at week start |
| FEAT-06.SPEC-005 (Live Grocery List Sync) | References (bidirectional) | Every write on this screen propagates through this integration; incoming changes from other devices render here live |
| FEAT-06.SPEC-006 (Ingredient Consolidation & Quantity Derivation) | References (inbound) | Computes the plan-derived line values this screen displays |
| FEAT-06.SPEC-007 (Manual Item Validation & Duplicate Merge) | Triggers (outbound) | Add-item action calls this spec's validation and merge rules |
| FEAT-06.SPEC-008 (Offline Conflict Resolution Rules) | References (bidirectional) | Governs tick, edit, and offline/reconnect behavior on this screen |
| FEAT-06.SPEC-009 (Grocery List Access & Authorization Rules) | References (inbound) | Defines this screen's Access and Visibility table |
| FEAT-20.SPEC-001 (Grocery Handoff Screen) | Navigation (outbound) | "Hand off list" header control navigates here, gated by FEAT-20.SPEC-003 |
| FEAT-20.SPEC-003 (Handoff Eligibility & Authorization Rules) | References (inbound) | Governs the "Hand off list" control's region- and role-based visibility on this screen |
| FEAT-05.SPEC-001 (Pantry List & Item Entry) | Navigation (outbound) | "View Pantry" toast action navigates here after "already have it" |
| FEAT-05.SPEC-007 (Pantry Item Off-Grocery-List Exclusion Rule) | References (inbound) | Governs which roles see the "View Pantry" toast action |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| grocery_item_ticked | item origin (plan-derived/manual), aisle | Member ticks or unticks an item | supports success-metrics.md: "Grocery List Live-Update Trust" |
| grocery_item_added_manually | aisle assigned, merge outcome (new line/merged) | Member's manual add completes | supports success-metrics.md: "Grocery List Live-Update Trust" |
| grocery_item_edited | item origin | Member confirms a quantity edit | supports success-metrics.md: "Grocery List Live-Update Trust" |
| grocery_item_removed | item origin | Member removes an item | supports success-metrics.md: "Grocery List Live-Update Trust" |
| grocery_item_already_have | pantry_logged (yes/no), role | Member taps "Already have it" | supports success-metrics.md: "Grocery List Live-Update Trust" |

## Acceptance Criteria

**FEAT-06.SPEC-001-AC-01:** Given Maya is on the Grocery List screen before any plan exists this week, when she views the screen, then she sees "Your list will appear once this week's plan is ready. You can add items now." and the Add-item row is usable.

**FEAT-06.SPEC-001-AC-02:** Given Sam is on the Grocery List screen, when he types "more yoghurt" in the Add-item input and taps Add, then FEAT-06.SPEC-007 validates the name and a new line "more yoghurt" appears in its aisle section with "added by Sam" beneath it.

**FEAT-06.SPEC-001-AC-03:** Given Maya is standing in the supermarket with no signal, when she taps the tick control on "milk", then the item shows a strikethrough immediately and the offline banner "You're offline -- changes will sync when you reconnect." is visible.

**FEAT-06.SPEC-001-AC-04:** Given Sam is on the Grocery List screen, when connectivity returns after he ticked three items offline, then those ticks sync automatically via FEAT-06.SPEC-005 and the offline banner disappears without Sam taking any action.

**FEAT-06.SPEC-001-AC-05:** Given Maya opens the overflow menu on a plan-derived "spinach" line, when she taps "Already have it", then the line disappears from the list and she sees the toast "Marked as already have it -- added to your pantry."

**FEAT-06.SPEC-001-AC-06:** Given Jordan (older kid, limited login) opens the overflow menu on a plan-derived "feta" line, when he taps "Already have it", then the line disappears from the list and he sees the toast "Marked as already have it." with no pantry mention, since his Pantry Input access is None.

**FEAT-06.SPEC-001-AC-07:** Given Riley (Operator) opens the Grocery List for a household with an open Support Request, when the screen loads, then every row renders as static text with no tick, add, edit, remove, or already-have-it controls.

**FEAT-06.SPEC-001-AC-08:** Given Riley attempts to open a household's Grocery List with no open Support Request, when the attempt is made, then access is blocked entirely and no household list data is shown.

**FEAT-06.SPEC-001-AC-09:** Given Maya taps Edit quantity on a line and leaves the quantity field blank, when she taps Confirm, then the field shows the error "Quantity can't be empty." and the edit does not save.

**FEAT-06.SPEC-001-AC-10:** Given Sam removes an item from the list, when he taps Remove from the overflow menu, then the item disappears immediately with no confirmation dialog and no undo option.

**FEAT-06.SPEC-001-AC-11:** Given Maya adds "milk" and an existing manual line "Milk" is already on the list, when she taps Add, then FEAT-06.SPEC-007 merges the two into a single line rather than creating a duplicate.

**FEAT-06.SPEC-001-AC-12:** Given two devices in the same household both tick the same item while offline, when both reconnect, then the item settles into a single ticked state per FEAT-06.SPEC-008's idempotent-tick rule, with no double-toggle visible on either device.

**FEAT-06.SPEC-001-AC-13:** Given a write action ultimately fails after background retries, when the failure is final, then a banner "Couldn't save your change. Check your connection and try again." appears with a Retry option.

**FEAT-06.SPEC-001-AC-14:** Given Maya's session expires while she has unsaved text in the Add-item input, when she is prompted to sign in again, then the dialog "Your session has expired. Sign in to continue." appears and her typed text is restored after she signs back in.

**FEAT-06.SPEC-001-AC-15:** Given an unauthenticated visitor requests this screen directly, when the request is made, then they are redirected to the sign-in screen and no household list data is shown.

**FEAT-06.SPEC-001-AC-16:** Given Sam has typed "olive oil" into the Add-item input but has not tapped Add, when he navigates away from the Grocery List screen (or backgrounds the app) and then returns, then the Add-item input is empty -- the typed text was discarded, not preserved.

**FEAT-06.SPEC-001-AC-17:** Given Maya's household region has an available online-ordering capability, when she views the Grocery List header, then a "Hand off list" control is shown, and tapping it navigates her to FEAT-20.SPEC-001 (Grocery Handoff Screen).

**FEAT-06.SPEC-001-AC-18:** Given the household's region has no available online-ordering capability, when Sam views the Grocery List header, then no "Hand off list" control appears, per FEAT-20.SPEC-003; the same absence applies to Jordan (older kid, limited login) and Riley regardless of the household's region.

**FEAT-06.SPEC-001-AC-19:** Given Maya opens the overflow menu on a plan-derived "spinach" line and taps "Already have it", when the toast "Marked as already have it -- added to your pantry." appears, then it includes a "View Pantry" action, and tapping it navigates her to FEAT-05.SPEC-001 (Pantry List & Item Entry).

**FEAT-06.SPEC-001-AC-20:** Given Jordan (older kid, limited login) opens the overflow menu on a plan-derived "feta" line and taps "Already have it", when the toast "Marked as already have it." appears, then it contains no "View Pantry" action, since his Pantry Input access is None (FEAT-05.SPEC-007).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 5 (empty, populating, populated, error, offline) | 5 |
| Business Rules | 8 | 8 |
| Edge Cases | 7 | 7 |



# Automation Spec: Grocery List Generation & Recalculation

## Overview

**Name:** Grocery List Generation & Recalculation
**ID:** FEAT-06.SPEC-002
**Type:** Automation
**Purpose:** Builds the current week's Grocery List from the plan's ingredients on generation or first pick, and recalculates the plan-derived portion whenever the plan, a swap, a safety removal, or pantry data changes, while preserving manual items, ticks, and "already have it" marks.
**Parent Feature:** FEAT-06 -- Shared Grocery List

## Scope and Non-Goals

**In Scope:**
- Creating the week's Grocery List the first time a plan exists (generated or manually started)
- Recalculating the plan-derived portion of the list on every plan-affecting change
- Preserving manual items, ticked state, and "already have it" removals across recalculation
- Invoking ingredient consolidation and pantry exclusion for the plan-derived computation

**Non-Goals:**
- The consolidation and quantity-derivation math itself -- owned by FEAT-06.SPEC-006; this automation invokes it rather than duplicating it
- Manual item validation and duplicate merge -- owned by FEAT-06.SPEC-007
- Archiving the list at week end and carrying items forward -- owned by FEAT-06.SPEC-004; this automation only seeds a newly created week's list, it does not decide when a week ends
- Propagating the recalculated list to other devices -- owned by FEAT-06.SPEC-005; this automation persists the data and hands off to that integration

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Plan generated or approved | FEAT-03 (AI Weekly Dinner Plan Generation) | Weekly Plan status moves to Approved or is auto-adopted at week start | Weekly Plan (week, status), all Planned Meals with their recipes |
| Manual pick made | FEAT-23 (Manual Weekly Planning) | A household picks, changes, or clears a night's recipe | The affected Planned Meal (night, recipe, status) |
| Meal swapped or suggestion accepted | FEAT-04 (One-Tap Meal Swap) | A swap or accepted suggestion completes for a Planned Meal | The affected Planned Meal's new recipe |
| Safety-concern removal | FEAT-02 (Dietary Rules & Allergy Safety Engine), XBR-08 | A safety-concern report removes a meal from the plan, or a mid-week rule change (XBR-02) removes one | The removed Planned Meal's prior recipe |
| Pantry item logged or cleared | FEAT-05 (Pantry-Aware Suggestions), XBR-04 | A Pantry Item is created (Active) or removed (Used/Removed) | The affected Pantry Item's `item_name` |
| Week rollover seeds a new list | FEAT-06.SPEC-004 (Week Rollover & Carryover) | The new week's list is being created at week start | Carried-over manual Grocery List Items from the archived list |

## Processing Logic

1. Identify the household's current-week Weekly Plan (from FEAT-03 or FEAT-23).
2. Gather all Planned Meals in that plan (dinners and confirmed leftover lunches) excluding any in a Removed status.
3. Invoke FEAT-06.SPEC-006 (Ingredient Consolidation & Quantity Derivation) with those Planned Meals to compute the combined, aisle-grouped, pantry-excluded set of plan-derived lines.
4. Compare the newly computed plan-derived lines against the Grocery List's existing plan-derived lines (matched by `ingredient_name`).
5. For each plan-derived line whose contributing dinners are unchanged: update `quantity_and_unit` and `aisle` only if they changed, and preserve `ticked` as-is.
6. For each plan-derived line whose contributing dinners changed (a swap or safety removal altered which dinners need it): rewrite the line in place with the newly computed quantity and reset `ticked` to false, since the underlying need has changed.
7. For each plan-derived line no longer needed by any dinner: delete the Grocery List Item.
8. For each newly needed ingredient with no existing line: create a new Grocery List Item with `origin` set to "plan-derived".
9. Leave every manual-origin Grocery List Item untouched, regardless of any of the above.
10. If this run is seeding a newly created week's list (triggered by FEAT-06.SPEC-004), add the carried-over manual items supplied by that spec as manual-origin lines before completing.
11. Set the Grocery List's `status` to "Generated" on first population for the week, then "Active" once it has at least one item.
12. Persist the updated list and items, then hand off to FEAT-06.SPEC-005 (Live Grocery List Sync) to propagate the change to every household device.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Initial generation | No Grocery List exists yet for the current week | Grocery List created (status: Generated -> Active); plan-derived items created | List populates on FEAT-06.SPEC-001 within a couple of seconds, with an inline loading indicator | FEAT-06.SPEC-001, FEAT-06.SPEC-005 |
| Recalculation with changes | A trigger fires and the computed plan-derived set differs from the existing one | Affected Grocery List Items created, updated, or deleted; manual items untouched | List updates live on FEAT-06.SPEC-001, with no separate notification (silent per this feature's Communications) | FEAT-06.SPEC-001, FEAT-06.SPEC-005 |
| No change needed | A trigger fires but the computed set is identical to the existing one (e.g., a swap changed cook time but not ingredients) | None | Nothing visibly changes | FEAT-06.SPEC-001 |
| Recalculation failure | A candidate recipe's ingredient data is incomplete, or the computation cannot complete | The prior, last-known-good list is kept as-is; the incomplete recipe is excluded from this run's computation, consistent with FEAT-02's fail-closed principle | No error shown to the household; the list they see remains valid and usable. The failure is logged for the next successful trigger to retry in full | FEAT-06.SPEC-001 |

## Data Model

**Reads:** Weekly Plan -- `week`, `status`. Planned Meal -- `night`, `recipe`, `status`. Recipe -- ingredients (via FEAT-06.SPEC-006). Pantry Item -- `item_name`, `status`. Household -- `aisle_grouping`, `unit_system`.

**Creates:** Grocery List (on first population for the week) -- `week`, `aisle_grouping`, `status`. Grocery List Item (plan-derived) -- `ingredient_name`, `quantity_and_unit`, `aisle`, `origin` ("plan-derived"), `ticked` (false).

**Updates:** Grocery List -- `status`. Grocery List Item (plan-derived) -- `quantity_and_unit`, `aisle`, `ticked` (reset only when contributing dinners changed).

**Deletes:** Grocery List Item (plan-derived) -- when no longer needed by any dinner.

## Business Rules

- XBR-03: every pick, change, swap, accepted suggestion, or safety removal updates the list immediately; no member ever sees a week's plan without its matching list.
- XBR-04: pantry exclusion (via FEAT-06.SPEC-006) is applied on every recalculation, not only at initial generation.
- XBR-11: aisle grouping and quantities follow the household's current unit system and aisle configuration (FEAT-16) at the time of each recalculation.
- Manual Grocery List Items are never created, modified, or deleted by this automation -- only FEAT-06.SPEC-001 (via FEAT-06.SPEC-007) and FEAT-06.SPEC-004 (carryover) touch them.
- A plan-derived line's `ticked` state survives recalculation only when the same set of contributing dinners still needs the same ingredient; a change in contributing dinners resets it, since the prior tick no longer reflects a verified purchase against the current need.

## Edge Cases

- **Concurrent trigger firing (e.g., a swap and a pantry-item log arrive at nearly the same time)** -- Each trigger recomputes the full plan-derived set from the household's current source data rather than applying a delta, so overlapping runs converge to the same result regardless of order; no update is lost.
- **Trigger fires while a previous run is in flight** -- Recalculation for a household is not re-entrant: a trigger arriving mid-run queues and executes immediately after the in-flight run completes, using the latest source data at that time, so no trigger is dropped.
- **A recipe with incomplete ingredient data is on the plan mid-week** -- That recipe's ingredients are excluded from the computed set this run, consistent with FEAT-02's fail-closed principle; the rest of the plan-derived list still recalculates normally.
- **A safety-removed meal's ingredient is also needed by another dinner still on the plan** -- The ingredient's line is reduced (not removed), reflecting only the remaining dinner's need, via FEAT-06.SPEC-006.
- **An older-kid login (Pantry Input: None) marks "already have it" on an ingredient, and a later swap still needs it** -- Since no Pantry Item was created for this role's action (FEAT-06.SPEC-003), nothing excludes the ingredient from recalculation, and the line is re-added if the plan still needs it. This is a deliberate consequence of the role's lack of pantry access, not a defect.
- **Week rollover fires before the new week's plan exists yet** -- The new Grocery List is created empty except for FEAT-06.SPEC-004's carried-over manual items; plan-derived lines populate once a plan exists for that week and this automation's other triggers fire.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03 (AI Weekly Dinner Plan Generation) | Triggered by (inbound) | Plan generation/approval fires this automation |
| FEAT-23 (Manual Weekly Planning) | Triggered by (inbound) | Manual picks fire this automation |
| FEAT-04 (One-Tap Meal Swap) | Triggered by (inbound) | Swaps and accepted suggestions fire this automation |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | Triggered by (inbound) | Safety removals fire this automation (XBR-08) |
| FEAT-05 (Pantry-Aware Suggestions) | Triggered by (inbound) | Pantry item log/clear fires this automation (XBR-04) |
| FEAT-06.SPEC-006 (Ingredient Consolidation & Quantity Derivation) | Triggers (outbound) | Invoked to compute the combined, pantry-excluded plan-derived set |
| FEAT-06.SPEC-001 (Grocery List) | Affects (outbound) | Displays the resulting list |
| FEAT-06.SPEC-004 (Week Rollover & Carryover) | Triggered by (inbound) | Seeds a newly created week's list with carried-over manual items |
| FEAT-06.SPEC-005 (Live Grocery List Sync) | Triggers (outbound) | Propagates the recalculated list to every household device |

## Analytics and Success Signals

- **grocery_list_generated** (trigger_type: initial/plan/swap/safety/pantry/rollover) -- supports success-metrics.md: "Grocery List Live-Update Trust"
- **grocery_items_combined** (line count reduced through consolidation) -- N/A -- success-metrics.md defines no metric for consolidation volume; retained to observe how often the combine-across-dinners rule (FEAT-06.SPEC-006) applies in practice
- **grocery_list_recalculation_failed** (reason: incomplete_ingredient_data / processing_error) -- N/A -- success-metrics.md defines no metric for recalculation failure rate; retained to monitor the "always matching list" guarantee operationally

## Acceptance Criteria

**FEAT-06.SPEC-002-AC-01:** Given Maya's household has no Grocery List yet this week, when her weekly plan is approved, then a Grocery List is created with plan-derived lines for every ingredient across the week's dinners, grouped by aisle.

**FEAT-06.SPEC-002-AC-02:** Given Sam's household plans manually, when Sam picks a recipe for Tuesday, then the list recalculates to include Tuesday's ingredients within a couple of seconds.

**FEAT-06.SPEC-002-AC-03:** Given a dinner is swapped for a different recipe, when the swap completes, then the list's plan-derived lines update to reflect the new recipe's ingredients, and the old recipe's ingredients no longer needed by any other dinner are removed.

**FEAT-06.SPEC-002-AC-04:** Given a safety-concern report removes a meal from the plan, when the removal completes, then that meal's ingredients are dropped from the list (or reduced, if shared with another dinner still on the plan).

**FEAT-06.SPEC-002-AC-05:** Given Maya logs "spinach" as a pantry item before the week's plan generates, when the plan generates, then "spinach" is excluded from the plan-derived list per FEAT-06.SPEC-006.

**FEAT-06.SPEC-002-AC-06:** Given Maya has manually added "birthday candles" to the list, when the plan is swapped and the list recalculates, then "birthday candles" remains on the list untouched.

**FEAT-06.SPEC-002-AC-07:** Given a plan-derived line "chicken" was ticked and the contributing dinner is unchanged, when a swap affecting a different night triggers recalculation, then "chicken" remains ticked.

**FEAT-06.SPEC-002-AC-08:** Given a plan-derived line "chicken" was ticked and the dinner needing it is swapped for a different recipe, when the swap triggers recalculation, then the "chicken" line (if still needed elsewhere) is unticked, since the contributing dinner changed.

**FEAT-06.SPEC-002-AC-09:** Given a recipe on the plan has incomplete ingredient data, when recalculation runs, then that recipe's ingredients are excluded from this run's computed set and the rest of the list recalculates normally, with no error shown to the household.

**FEAT-06.SPEC-002-AC-10:** Given a swap and a pantry-item log occur for the same household at effectively the same time, when both triggers fire, then both recalculation runs converge to the same final list, with no update lost.

**FEAT-06.SPEC-002-AC-11:** Given a recalculation is already in progress for a household, when a new trigger fires before it completes, then the new trigger's recalculation runs immediately after the in-flight one finishes, using the latest source data.

**FEAT-06.SPEC-002-AC-12:** Given Jordan (older kid, limited login) marks "already have it" on an ingredient with no pantry entry created, when a later swap still needs that ingredient, then the ingredient's line is re-added to the list on recalculation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 6 | 6 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: "Already Have It" Handling

## Overview

**Name:** "Already Have It" Handling
**ID:** FEAT-06.SPEC-003
**Type:** Automation
**Purpose:** Removes a plan-derived item from the grocery list and, for members with pantry access, logs it to the pantry in the same tap.
**Parent Feature:** FEAT-06 -- Shared Grocery List

## Scope and Non-Goals

**In Scope:**
- Removing a plan-derived Grocery List Item when a member marks it "already have it"
- Creating a Pantry Item for members whose Pantry Input access is Full
- Skipping pantry creation entirely for members whose Pantry Input access is None

**Non-Goals:**
- "Already have it" on a manual item -- not offered; manual items are removed directly via FEAT-06.SPEC-001's Remove action, since the household chose to add them and can choose to remove them the same way
- Pantry Item merge and clearing logic beyond creation -- owned by Pantry-Aware Suggestions (FEAT-05), which this automation defers to for how a duplicate pantry entry is handled
- Determining who may see or act on the Grocery List at all -- owned by FEAT-06.SPEC-009; this automation only differentiates the pantry-logging outcome once access is already established

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household member taps "Already have it" on a plan-derived item | FEAT-06.SPEC-001 (Grocery List) | The tapped item's `origin` is "plan-derived"; the member already has Grocery List access per FEAT-06.SPEC-009 | The item's `ingredient_name`, `quantity_and_unit`; the requesting member's role and Pantry Input access level |

## Processing Logic

1. Confirm the tapped item's `origin` is "plan-derived" (the control is not offered on manual items).
2. Remove the Grocery List Item from the list immediately.
3. Check the requesting member's Pantry Input access level per the Access Matrix (user-persona.md).
4. If Pantry Input is Full (Maya, Sam): create a Pantry Item with `item_name` set to the removed item's `ingredient_name`, `added_by` set to the requesting member, and `status` set to Active, per XBR-04. If a matching Pantry Item already exists, Pantry-Aware Suggestions' (FEAT-05) merge behavior applies rather than creating a second entry.
5. If Pantry Input is None (Jordan, older kid, limited login -- Later): skip pantry creation entirely; only the list removal occurs.
6. Hand off to FEAT-06.SPEC-005 (Live Grocery List Sync) to propagate the removal to every household device within seconds.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Removed and logged to pantry | Requesting member's Pantry Input access is Full | Grocery List Item deleted; Pantry Item created (or merged into an existing one) | Toast: "Marked as already have it -- added to your pantry." | FEAT-06.SPEC-001, FEAT-05 |
| Removed, no pantry entry | Requesting member's Pantry Input access is None | Grocery List Item deleted only | Toast: "Marked as already have it." | FEAT-06.SPEC-001 |
| Pantry write fails after list removal | Grocery List Item deletion succeeds but the pantry write cannot complete after background retries | Grocery List Item stays deleted; no Pantry Item created | Toast: "Removed from list. Couldn't add to your pantry -- try adding it there directly." | FEAT-06.SPEC-001, FEAT-05 |

## Data Model

**Reads:** Grocery List Item -- `ingredient_name`, `origin`. Member Profile -- role, to determine Pantry Input access level.

**Updates:** None beyond the delete below.

**Deletes:** Grocery List Item -- the marked item, unconditionally.

**Creates:** Pantry Item -- `item_name`, `added_by`, `status`, only when the requesting member's Pantry Input access is Full.

## Business Rules

- XBR-04: logged pantry items are left off the week's list, and marking a list item "already have it" can add it to the pantry in the same tap; only members with Full Pantry Input access trigger the pantry write.
- Grocery List Item removal is unconditional and immediate regardless of pantry-logging outcome -- a pantry write failure never leaves the item back on the list.
- Eligibility to act on this automation at all is governed by FEAT-06.SPEC-009; this spec only differentiates the pantry outcome once access is already confirmed.

## Edge Cases

- **Concurrent trigger firing (two members tap "Already have it" on the same item at effectively the same time)** -- The first tap removes the item and (if eligible) creates the pantry entry; the second tap targets an item that no longer exists and is a silent no-op -- no duplicate pantry entry is created and no error is shown.
- **Trigger fires while a previous run is in flight for the same item** -- Because live sync (FEAT-06.SPEC-005) removes the item from other devices as soon as the first tap is processed, a second tap on the same item in practice arrives after the item is already gone and is handled by the concurrent-trigger case above; two different items processed at the same time run independently with no queuing between them.
- **Member marks "already have it" while offline** -- The removal (and, where eligible, the pantry creation) queues locally per FEAT-06.SPEC-005 and FEAT-06.SPEC-008, and applies once connectivity returns.
- **A duplicate Pantry Item already exists for the same ingredient** -- Pantry-Aware Suggestions' (FEAT-05) merge rule applies: the two become one entry rather than two, per the dependency map's Contention note for Pantry Item.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-001 (Grocery List) | Triggered by (inbound) | "Already have it" overflow action fires this automation |
| FEAT-06.SPEC-001 (Grocery List) | Affects (outbound) | The item disappears from the screen on success |
| FEAT-06.SPEC-005 (Live Grocery List Sync) | Triggers (outbound) | Propagates the removal to every household device |
| FEAT-05 (Pantry-Aware Suggestions) | Affects (outbound) | Creates or merges a Pantry Item for eligible roles |
| FEAT-06.SPEC-009 (Grocery List Access & Authorization Rules) | References (inbound) | Governs eligibility to act on this automation at all |

## Analytics and Success Signals

- **grocery_item_already_have** (role, pantry_logged: yes/no) -- supports success-metrics.md: "Grocery List Live-Update Trust"
- **grocery_item_already_have_pantry_write_failed** (reason) -- N/A -- success-metrics.md defines no metric for pantry-write reliability specifically; retained to observe how often the non-blocking pantry-write guarantee is exercised

## Acceptance Criteria

**FEAT-06.SPEC-003-AC-01:** Given Maya opens the overflow menu on a plan-derived "spinach" line, when she taps "Already have it", then the item is removed from the list and a Pantry Item "spinach" is created with Maya as `added_by`.

**FEAT-06.SPEC-003-AC-02:** Given Sam taps "Already have it" on a plan-derived "feta" line, when the action completes, then the item is removed and he sees the toast "Marked as already have it -- added to your pantry."

**FEAT-06.SPEC-003-AC-03:** Given Jordan (older kid, limited login, Pantry Input: None) taps "Already have it" on a plan-derived item, when the action completes, then the item is removed from the list, no Pantry Item is created, and he sees the toast "Marked as already have it."

**FEAT-06.SPEC-003-AC-04:** Given the pantry write fails after background retries for a Full-access member's "already have it" action, when the failure is final, then the toast "Removed from list. Couldn't add to your pantry -- try adding it there directly." appears and the item stays removed from the list.

**FEAT-06.SPEC-003-AC-05:** Given a matching Pantry Item already exists when Maya taps "Already have it" on the same ingredient, when the pantry write runs, then the two entries merge into one per FEAT-05's merge rule rather than creating a duplicate.

**FEAT-06.SPEC-003-AC-06:** Given two household members both tap "Already have it" on the same item at effectively the same time, when both taps are processed, then the item is removed once, at most one Pantry Item is created, and the second tap has no additional effect.

**FEAT-06.SPEC-003-AC-07:** Given Sam taps "Already have it" while offline, when he is offline, then the removal (and pantry creation) queue locally and apply automatically once connectivity returns.

**FEAT-06.SPEC-003-AC-08:** Given Maya taps "Already have it" on an item, when the removal completes, then the change propagates to every other household device within seconds via FEAT-06.SPEC-005.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |



# Automation Spec: Week Rollover & Carryover

## Overview

**Name:** Week Rollover & Carryover
**ID:** FEAT-06.SPEC-004
**Type:** Automation
**Purpose:** Archives the current week's grocery list at week end and carries unticked manually-added items into the new week's list.
**Parent Feature:** FEAT-06 -- Shared Grocery List

## Scope and Non-Goals

**In Scope:**
- Archiving the current week's Grocery List when the week ends
- Identifying unticked manual items on the archived list
- Handing those items to FEAT-06.SPEC-002 to seed the new week's list

**Non-Goals:**
- Deciding when a household's "week" boundary falls -- derived from the household's plan cycle (owned by FEAT-01/FEAT-16); this automation only reacts to that boundary being reached
- Creating the new week's plan-derived list content -- owned by FEAT-06.SPEC-002; this automation supplies only the carried manual items
- Making the archived list browsable -- owned by Weekly Plan History (FEAT-19); this automation only sets the archived status the browsing feature reads
- Purging any archived list or item -- excluded per scope-boundaries.md SC-18: archived lists and their items are retained for the life of the household account with no automatic purge

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| The household's current week ends | system (aligned to the household's plan-arrival cycle, owned by FEAT-01/FEAT-16) | Fires once per household per week, at the boundary between the ending week and the next | The current Active Grocery List and its Items |

## Processing Logic

1. Identify the household's current Active Grocery List for the ending week.
2. Set the list's `status` to Archived. It is retained in full, never hard-deleted, per SC-18.
3. Identify every Grocery List Item on the archived list whose `origin` is "manual" and `ticked` is false.
4. Pass those items' `ingredient_name`, `quantity_and_unit`, `aisle`, and originating member ("added by") to FEAT-06.SPEC-002 as carryover input for the new week's list.
5. Leave every ticked manual item and every plan-derived item on the archived list only -- they are not carried forward.
6. Once FEAT-06.SPEC-002 creates or has already created the new week's Grocery List, the carried items appear on it as manual-origin lines with `ticked` reset to false.
7. Make the archived list available for browsing through Weekly Plan History (FEAT-19).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Rollover with carryover | One or more unticked manual items exist on the ending week's list | Current list archived; new week's list seeded with carried manual items | New list shows the carried items alongside plan-derived generation, with no separate notification (silent, per this feature's Communications) | FEAT-06.SPEC-001, FEAT-06.SPEC-002 |
| Rollover, nothing to carry | No unticked manual items exist on the ending week's list | Current list archived only | New list starts from plan-derived generation alone | FEAT-06.SPEC-001, FEAT-06.SPEC-002 |
| Rollover ahead of the new plan | The new week's plan does not exist yet when rollover fires | Current list archived; carried items held pending until FEAT-06.SPEC-002 creates the new list | No visible list until the new week's plan or first pick exists; carried items appear once it does | FEAT-06.SPEC-002 |
| Archive step fails | The archive write cannot complete | The current list remains Active and visible rather than disappearing | No visible change to the household; rollover retries on the next scheduled attempt | FEAT-06.SPEC-001 |

## Data Model

**Reads:** Grocery List Item -- `origin`, `ticked`, `ingredient_name`, `quantity_and_unit`, `aisle`, added-by attribution, for the ending week's list.

**Updates:** Grocery List -- `status` (Active -> Archived).

**Creates:** Grocery List Item (on the new week's list, via FEAT-06.SPEC-002) -- carried copies of qualifying manual items.

**Deletes:** None -- archiving is a status change, not a deletion; no item is removed by this automation.

## Business Rules

- SC-18: archived lists and their items are retained for the life of the household account, browsable after a downgrade to the free tier, with no purge policy while the account is active.
- Archiving has no cascade to other entities -- the archived list's items stay attached to it exactly as they were at archive time, except for the carried copies created fresh on the new list.
- This is the sole Archive path for Grocery List; no other spec transitions a list to Archived.

## Edge Cases

- **Concurrent trigger firing for the same week boundary** -- Rollover cannot fire twice for the same week: a list already Archived is never re-archived, so a duplicate trigger for the same boundary is a no-op against the already-archived list.
- **Trigger fires while a previous rollover is in flight** -- The next week's rollover cannot begin until the current one completes, since a household has exactly one Active list at a time; rollover runs are naturally serialized by that invariant.
- **A manual item is added in the final moments before rollover** -- It is still evaluated by the unticked-manual-item check and carried forward if it qualifies at the moment rollover runs.
- **A household has no grocery-list activity that week** -- Rollover still archives the (empty or near-empty) list; there is simply nothing to carry.
- **A manual item is ticked by one member at the exact moment rollover evaluates it** -- Whichever state the item is in when the evaluation step (Step 3) reads it determines whether it carries forward; there is no separate reconciliation after rollover has run for that week.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-002 (Grocery List Generation & Recalculation) | Triggers (outbound) | Supplies carried-over manual items to seed the new week's list |
| FEAT-06.SPEC-001 (Grocery List) | Affects (outbound) | The new list, including carried items, appears there |
| FEAT-19 (Weekly Plan History) | Affects (outbound) | The archived list becomes browsable there |

## Analytics and Success Signals

- **grocery_items_carried_over** (count) -- N/A -- success-metrics.md defines no metric for week-to-week carryover; retained to monitor how often unticked manual items persist across weeks
- **grocery_list_archived** (item_count) -- N/A -- success-metrics.md defines no metric for archive volume; retained as an operational signal that rollover is completing on schedule

## Acceptance Criteria

**FEAT-06.SPEC-004-AC-01:** Given Maya's household has an unticked manual item "birthday candles" on the current week's list, when the week ends, then the list is archived and "birthday candles" appears on the new week's list as a manual-origin item.

**FEAT-06.SPEC-004-AC-02:** Given a manual item "milk" was ticked before the week ended, when rollover runs, then "milk" stays on the archived list only and does not appear on the new week's list.

**FEAT-06.SPEC-004-AC-03:** Given a household's current week's list has no unticked manual items, when the week ends, then the list is archived and the new week's list starts from plan-derived generation alone.

**FEAT-06.SPEC-004-AC-04:** Given the new week's plan does not exist yet when rollover fires, when rollover completes, then the current list is archived and carried items are held pending until the new week's plan or first pick creates the new list.

**FEAT-06.SPEC-004-AC-05:** Given the archive step fails for a household, when the failure occurs, then the current week's list remains Active and visible, and rollover retries on the next scheduled attempt.

**FEAT-06.SPEC-004-AC-06:** Given a household's list is already Archived for a week boundary, when a duplicate rollover trigger fires for the same boundary, then no further change occurs to that list.

**FEAT-06.SPEC-004-AC-07:** Given a household's rollover for the current week is still in flight, when the next week's boundary is reached before it completes, then the next rollover waits until the in-flight one finishes.

**FEAT-06.SPEC-004-AC-08:** Given an archived list exists for a past week, when a household member opens Weekly Plan History (FEAT-19), then the archived list and its items are visible there in full.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



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



# Logic/Rule Spec: Ingredient Consolidation & Quantity Derivation

## Overview

**Name:** Ingredient Consolidation & Quantity Derivation
**ID:** FEAT-06.SPEC-006
**Type:** Logic/Rule
**Purpose:** Derives the plan-derived portion of the grocery list by combining the same ingredient across dinners into one line with a household-sized combined quantity, minus logged pantry items.
**Parent Feature:** FEAT-06 -- Shared Grocery List
**Governed Entity:** Grocery List Item (plan-derived subset)

## Scope and Non-Goals

**In Scope:**
- Matching the same ingredient across multiple Planned Meals and combining them into one line
- Converting and summing quantities into the household's configured unit system
- Assigning each combined line's aisle from the household's configured aisle categories
- Excluding a combined ingredient entirely when a matching Active Pantry Item exists

**Non-Goals:**
- Manual item validation and duplicate merge -- owned by FEAT-06.SPEC-007; this spec governs plan-derived lines only
- Deciding when recalculation runs -- owned by FEAT-06.SPEC-002, which invokes this spec's computation
- Partial pantry-quantity subtraction -- excluded because Pantry Item carries no quantity field (per the Feature Dependency Map); pantry exclusion is necessarily binary, not a partial-amount reduction
- Access to view or act on the resulting lines -- owned by FEAT-06.SPEC-009

## Governed Entity

**Entity:** Grocery List Item (plan-derived subset)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| ingredient_name | text | The combined ingredient's name, copied from the contributing Recipes' ingredient lists |
| quantity_and_unit | text | The combined, household-sized quantity, in the household's configured unit system |
| aisle | text | The aisle category this ingredient is grouped under, from the household's configured `aisle_grouping` (FEAT-16) |
| origin | enum | Always "plan-derived" for lines this spec produces |
| ticked | boolean | Whether the line has been ticked; preserved or reset per FEAT-06.SPEC-002's recalculation rules, not set by this spec |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-002 | Grocery List Generation & Recalculation | Invokes this spec's computation on every generation and recalculation trigger |
| FEAT-06.SPEC-001 | Grocery List | Displays the computed lines this spec produces; does not recompute them itself |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| ingredient_name | No validation beyond data type -- copied from Recipe data, not user-entered for plan-derived lines | Always | -- | -- | -- |
| quantity_and_unit | No validation beyond data type -- system-computed by this spec's derivation logic (see Defaults and Derivations) | Always | -- | -- | -- |
| aisle | No validation beyond data type -- system-assigned by lookup against the household's configured aisle categories | Always | -- | -- | -- |
| origin | No validation beyond data type -- always set to "plan-derived" by this spec | Always | -- | -- | -- |
| ticked | No validation beyond data type -- managed by FEAT-06.SPEC-002's recalculation rules, not set by this spec | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Same-ingredient combination | ingredient_name, quantity_and_unit | Grocery List Items derived from two or more Planned Meals combine into a single line when their `ingredient_name` matches (case-insensitive, trimmed) and the underlying ingredient and unit are literally the same (a vegetarian-variant substitution is treated as a distinct ingredient, not combined with the standard version) | N/A -- not a validation failure, a derivation rule (see Defaults and Derivations) |
| Pantry exclusion | ingredient_name (Grocery List Item) matched against item_name (Pantry Item) | If an Active Pantry Item's `item_name` matches a combined ingredient's `ingredient_name` (case-insensitive, trimmed), that ingredient's entire combined line is excluded from the plan-derived list (XBR-04). The exclusion is binary -- Pantry Item carries no quantity, so partial coverage is not modeled | N/A -- not a validation failure, a derivation rule |

## Authorization Rules

{This spec computes list content; it introduces no authorization surface of its own beyond the resulting lines' visibility, which is fully governed by FEAT-06.SPEC-009. The row below states only that computation runs regardless of who is viewing.}

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View a computed plan-derived line | Maya, Sam, Jordan (older kid, limited login), Riley (Operator, support, while an open Support Request exists) | Always, once the viewer already has Grocery List access per FEAT-06.SPEC-009 | -- (full detail governed by FEAT-06.SPEC-009, not restated here) |
| View a computed plan-derived line | Jordan (young kid profile, no login) | Never | This profile has no login and cannot reach the screen at all |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| ingredient_name | Copied from the Recipe's ingredient list for each contributing Planned Meal | On every recalculation | No |
| quantity_and_unit | Sum of each contributing dinner's required amount for the same ingredient, converted into the household's configured `unit_system` (FEAT-16) before summing | On every recalculation | No -- always recomputed; a member's direct quantity edit on FEAT-06.SPEC-001 is preserved by FEAT-06.SPEC-002 only as long as the line's contributing dinners are unchanged (see FEAT-06.SPEC-002 Business Rules) |
| aisle | Looked up from the household's configured `aisle_grouping` (FEAT-16) by matching the ingredient against its known category; falls back to a household catch-all "Other" aisle when no match is found | On every recalculation | No |
| origin | Set to "plan-derived" | On creation | No |
| ticked | Not set by this spec -- preserved or reset by FEAT-06.SPEC-002's recalculation rules | N/A | N/A |

## Business Rules

- XBR-03: this computation runs on every plan-affecting change so the list never falls out of sync with the plan.
- XBR-04: pantry exclusion is applied as part of this computation, not as a separate downstream filter.
- XBR-11: the household's currently configured unit system and aisle categories (FEAT-16) apply at computation time; a later change to those settings is reflected on the next recalculation, not retroactively rewritten.

## Edge Cases

- **A recipe is safety-removed but shares an ingredient with a dinner still on the plan** -- The combined line reduces to reflect only the remaining dinner's need rather than being removed entirely.
- **An ingredient name is a near-miss, not an exact match (e.g., "tomatoes" vs. "tomato")** -- Matching is exact (case-insensitive, trimmed); a near-miss does not combine and both spellings would appear as distinct lines if both occurred, a limitation the household resolves by marking either line "already have it" or removing it directly.
- **A shared meal's vegetarian variant and its standard version both appear in the same week** -- Their ingredients are not combined, since the underlying ingredient differs between the two versions.
- **Exactly two dinners need the same ingredient in different units (e.g., cups vs. ounces)** -- Both are converted to the household's single configured unit system before summing into one line.
- **A Pantry Item's `item_name` exactly matches a combined ingredient, but the household still needs more of it than they have on hand** -- The line is still excluded entirely, since Pantry Item carries no quantity and the exclusion is binary by design; the household re-adds it manually if they need more.
- **Only one dinner needs an ingredient (no combination case)** -- The line still passes through this computation as a single-source line, aisle-assigned and pantry-checked the same as any combined line.

## Acceptance Criteria

**FEAT-06.SPEC-006-AC-01:** Given two dinners this week both use "chicken breast," when the plan-derived list is computed, then a single "chicken breast" line appears with the summed quantity from both dinners.

**FEAT-06.SPEC-006-AC-02:** Given Maya has logged "spinach" as an Active Pantry Item, when the plan-derived list is computed and a dinner needs spinach, then no "spinach" line appears on the list.

**FEAT-06.SPEC-006-AC-03:** Given two dinners need the same ingredient in different units, when the combined line is computed, then both amounts are converted to the household's configured unit system before being summed.

**FEAT-06.SPEC-006-AC-04:** Given a shared meal has both a standard and a vegetarian-variant version this week, when the plan-derived list is computed, then their distinct ingredients are not combined into one line.

**FEAT-06.SPEC-006-AC-05:** Given only one dinner this week needs "basil," when the plan-derived list is computed, then a single-source "basil" line appears, aisle-assigned and pantry-checked the same as any other line.

**FEAT-06.SPEC-006-AC-06:** Given a safety-concern removal drops a recipe that shared "onions" with another dinner still on the plan, when recalculation runs, then the "onions" line is reduced to reflect only the remaining dinner's need, not removed.

**FEAT-06.SPEC-006-AC-07:** Given the household's aisle categories don't recognize a given ingredient, when its line is computed, then it is grouped under the household's catch-all "Other" aisle.

**FEAT-06.SPEC-006-AC-08:** Given an ingredient is logged in the pantry as "tomato" and the plan needs "tomatoes," when the exclusion check runs, then the near-miss does not match and the ingredient still appears on the list.

**FEAT-06.SPEC-006-AC-09:** Given Riley (Operator) views a household's list during an open Support Request, when a computed plan-derived line renders, then it displays the same computed line Maya and Sam see, read-only.

**FEAT-06.SPEC-006-AC-10:** Given Jordan (young kid profile, no login) has no way to sign in, when any attempt is made to view the list, then no computed line is ever shown to this profile, since it cannot reach the screen at all.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 2 | 2 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Manual Item Validation & Duplicate Merge

## Overview

**Name:** Manual Item Validation & Duplicate Merge
**ID:** FEAT-06.SPEC-007
**Type:** Logic/Rule
**Purpose:** Validates a manually added item's name and merges a duplicate manual entry for the same ingredient into the existing line.
**Parent Feature:** FEAT-06 -- Shared Grocery List
**Governed Entity:** Grocery List Item (manual subset)

## Scope and Non-Goals

**In Scope:**
- Field validation for a manually added item's name
- Detecting and merging a duplicate manual entry for the same ingredient
- Detecting and absorbing a manual entry that matches an existing plan-derived line
- Default and derived values for a manual item at creation

**Non-Goals:**
- Plan-derived line computation and pantry exclusion -- owned by FEAT-06.SPEC-006
- Authorization for tick, edit-quantity, and remove actions -- governed by FEAT-06.SPEC-009; this spec's Authorization Rules cover only the Add action this validation governs
- Restoring a manually merged item as a separate line if its plan-derived host line later disappears -- a deliberate simplicity trade-off: the household re-adds the item manually if they still want it after a swap removes the host line, consistent with this feature's other no-restore, re-add-if-needed decisions (see FEAT-06.SPEC-001 Non-Goals)

## Governed Entity

**Entity:** Grocery List Item (manual subset)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| ingredient_name | text | The manually entered item's name (1-80 characters, required) |
| quantity_and_unit | text | Optional free-text quantity for a manual item |
| aisle | text | The aisle category, from the household's configured `aisle_grouping` (FEAT-16) |
| origin | enum | Always "manual" for lines this spec governs |
| ticked | boolean | Whether the line has been ticked; defaults to false on creation |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-001 | Grocery List | On the Add-item input, on submit (blur and submit checks for name validity; duplicate check runs on submit) |
| FEAT-06.SPEC-002 | Grocery List Generation & Recalculation | Invokes this spec's duplicate-merge rule when a newly computed plan-derived line would name the same ingredient as an existing manual line |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| ingredient_name | Required, non-empty after trimming, 1-80 characters | Always | On blur and on submit | "Item name is required." / "Item name must be 80 characters or fewer." | Yes |
| quantity_and_unit | No validation beyond data type -- optional free text; when left blank, the line displays without a quantity value | Always | -- | -- | No |
| aisle | No validation beyond data type -- system-assigned, not directly user-entered at add time | Always | -- | -- | -- |
| origin | No validation beyond data type -- always set to "manual" by this spec | Always | -- | -- | -- |
| ticked | No validation beyond data type -- defaults to false and is governed thereafter by FEAT-06.SPEC-008 | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Manual-manual duplicate merge | ingredient_name (new entry), ingredient_name (existing manual line) | If the new manual item's name matches (case-insensitive, trimmed) an existing manual line on the current list, the existing line's `quantity_and_unit` is updated to the newly stated quantity rather than creating a second line; the existing line's `ticked` state and "added by" attribution are preserved | "Updated the quantity on your existing '{ingredient_name}' item." |
| Manual-vs-plan-derived absorption | ingredient_name (new manual entry), ingredient_name (existing plan-derived line) | If the new manual item's name matches an existing plan-derived line instead, no separate manual line is created; the plan-derived line remains with its computed quantity | "This is already on your list from this week's plan." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Add manual item | Maya (Organiser), Sam (Other Adult Member), Jordan (older kid, limited login -- Later) | Always | -- |
| Add manual item | Jordan (young kid profile, no login -- MVP) | Never | This profile has no login and cannot reach the screen at all |
| Add manual item | Riley (Operator, support -- from v1) | Never | The Add-item control is not rendered; Riley's view is read-only per Operator Read-Only Support Access (FEAT-22) and FEAT-06.SPEC-009 |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| aisle | Looked up from the household's configured `aisle_grouping` (FEAT-16) by matching `ingredient_name` against known categories; falls back to a household catch-all "Other" aisle when no match is found | On create | No -- aisle placement is automatic; the household adjusts aisle categories in Household Setup / FEAT-16, not per item |
| origin | Set to "manual" | On create | No |
| ticked | Set to false | On create | Yes -- the member ticks it afterward like any other line |

## Business Rules

- XBR-03: manual items coexist alongside plan-derived items on the same list without either overwriting the other.
- FEAT-06.SPEC-002 invokes this spec's duplicate-merge rule during recalculation when a newly computed plan-derived line would name the same ingredient as an existing manual line, so the household never sees two lines for the same thing regardless of which side added it first.
- Field validation runs before the duplicate-merge check -- an invalid name is never checked for a duplicate match.

## Edge Cases

- **Item name at exactly 80 characters** -- Passes validation. 81 characters shows the length error.
- **Item name is whitespace-only** -- Treated as empty after trimming; shows "Item name is required."
- **Duplicate merge across case differences ("Milk" vs. "milk")** -- Matches and merges, since the comparison is case-insensitive and trimmed.
- **A manual add absorbed into a plan-derived line, and that plan-derived line is later removed by recalculation (e.g., a swap drops the dinner needing it)** -- The absorbed manual intent is not restored as a separate line; the household re-adds the item manually if they still want it, per this spec's Non-Goals.
- **Two members add the same ingredient manually at nearly the same time, before either has synced** -- Both intend to merge into a single line; FEAT-06.SPEC-008 governs how the two concurrent adds reconcile into exactly one line without duplication.

## Acceptance Criteria

**FEAT-06.SPEC-007-AC-01:** Given Sam types "more yoghurt" and taps Add, when the name passes validation, then a new manual line "more yoghurt" is created with `origin` set to "manual" and `ticked` set to false.

**FEAT-06.SPEC-007-AC-02:** Given Maya leaves the Add-item input empty and taps Add, when validation runs, then the error "Item name is required." appears and no line is created.

**FEAT-06.SPEC-007-AC-03:** Given Maya types an 81-character item name and taps Add, when validation runs, then the error "Item name must be 80 characters or fewer." appears and no line is created.

**FEAT-06.SPEC-007-AC-04:** Given a manual line "Milk" already exists on the list, when Maya adds "milk", then the two are merged into one line and she sees "Updated the quantity on your existing 'Milk' item."

**FEAT-06.SPEC-007-AC-05:** Given a plan-derived line "eggs" already exists on the list, when Sam manually adds "eggs", then no second line is created and he sees "This is already on your list from this week's plan."

**FEAT-06.SPEC-007-AC-06:** Given Jordan (older kid, limited login) is on the Grocery List screen, when he adds "granola bars", then the item is added successfully, since his role is allowed to add manual items.

**FEAT-06.SPEC-007-AC-07:** Given Riley (Operator) views a household's list, when he looks for an Add-item control, then none is shown, since Riley's access is read-only.

**FEAT-06.SPEC-007-AC-08:** Given a manual item has no aisle match in the household's configured categories, when it is created, then it is placed under the household's catch-all "Other" aisle.

**FEAT-06.SPEC-007-AC-09:** Given a manual add was absorbed into a plan-derived "eggs" line, when a later swap removes the dinner needing eggs, then the "eggs" line disappears and is not restored as a separate manual line.

**FEAT-06.SPEC-007-AC-10:** Given Maya adds an item with only whitespace typed into the input, when she taps Add, then the error "Item name is required." appears, since whitespace-only input is treated as empty.

**FEAT-06.SPEC-007-AC-11:** Given two household members each add "bananas" at nearly the same time while both are briefly offline, when both changes sync, then exactly one "bananas" line exists on the list afterward, per FEAT-06.SPEC-008's reconciliation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Offline Conflict Resolution Rules

## Overview

**Name:** Offline Conflict Resolution Rules
**ID:** FEAT-06.SPEC-008
**Type:** Logic/Rule
**Purpose:** Governs how offline and concurrent changes to grocery list items resolve on sync -- idempotent ticks, last-write-wins quantity edits, and no duplicate lines on reconnect.
**Parent Feature:** FEAT-06 -- Shared Grocery List
**Governed Entity:** Grocery List Item

## Scope and Non-Goals

**In Scope:**
- Resolving two or more tick/untick actions on the same item made offline or concurrently
- Resolving two or more quantity edits on the same item made offline or concurrently
- Ensuring an offline manual add does not create a duplicate line on reconnect
- Ordering queued offline changes by original event time, not sync-arrival time

**Non-Goals:**
- The transport mechanism that delivers changes between devices -- owned by FEAT-06.SPEC-005 (Live Grocery List Sync); this spec defines only how conflicting changes resolve once delivered
- Manual item name validation and the duplicate-merge matching rule itself -- owned by FEAT-06.SPEC-007; this spec only states that reconnect sync defers to it rather than creating a second line
- General access to view or act on the list -- owned by FEAT-06.SPEC-009; this spec assumes access is already established and governs only how conflicting writes from already-authorized members resolve

## Governed Entity

**Entity:** Grocery List Item
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| ingredient_name | text | The item's name; used to detect duplicate lines on reconnect (governed in detail by FEAT-06.SPEC-007) |
| quantity_and_unit | text | The item's quantity; subject to last-write-wins resolution on concurrent edits |
| aisle | text | The item's aisle category; not independently editable, so not subject to conflict resolution on its own |
| origin | enum | Plan-derived or manual; not changed by conflict resolution |
| ticked | boolean | Whether the item is ticked; subject to idempotent-tick resolution |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-005 | Live Grocery List Sync | On every inbound change event and on every reconnect-after-offline sync |
| FEAT-06.SPEC-001 | Grocery List | Reflects the resolved state after conflict resolution runs; does not resolve conflicts itself |
| FEAT-06.SPEC-002 | Grocery List Generation & Recalculation | Interacts with these rules when a recalculation and a queued offline edit target the same line (see Edge Cases) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| ingredient_name | No validation beyond data type -- governed by FEAT-06.SPEC-006 (plan-derived) or FEAT-06.SPEC-007 (manual); this spec defines only conflict resolution for concurrent writes | Always | -- | -- | -- |
| quantity_and_unit | No validation beyond data type -- format rules live in FEAT-06.SPEC-001/FEAT-06.SPEC-007; this spec defines only how concurrent edits to this field resolve | Always | -- | -- | -- |
| aisle | No validation beyond data type -- not independently editable by a member | Always | -- | -- | -- |
| origin | No validation beyond data type -- not changed by conflict resolution | Always | -- | -- | -- |
| ticked | No validation beyond data type -- this spec defines how concurrent tick/untick actions resolve | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Idempotent tick | ticked | A tick or untick action applied to a line already in the target state has no additional effect once synced -- ticking an already-ticked line, or two members ticking the same line while both offline, results in exactly one ticked state, not a double-toggle | N/A -- not an error; the repeated action is silently absorbed |
| Last-write-wins quantity edit | quantity_and_unit | When two quantity edits to the same line occur before either has synced, the edit with the later original event time is the one that persists once both reach the shared list; the earlier edit is discarded without notifying either member | N/A -- not an error; resolved silently per the dependency map's Contention note for Grocery List Item |
| No duplicate lines on reconnect | ingredient_name, origin | A manual item added offline that matches an ingredient already present on the list (added by another member online in the interim) merges per FEAT-06.SPEC-007's duplicate-merge rule at sync time, rather than creating a second line | N/A -- not an error; merge behavior deferred to FEAT-06.SPEC-007 |
| Deletion precedence | ingredient_name | If one member deletes a line while another member has a queued offline edit (tick or quantity change) targeting the same line, the deletion takes precedence: the item stays removed and the queued edit is discarded silently, since removal reflects a conscious decision that the item is no longer needed | N/A -- not an error; resolved silently |

## Authorization Rules

{Conflict resolution applies uniformly to every member who already has write access to the Grocery List; it introduces no additional authorization surface. Full authorization detail is governed by FEAT-06.SPEC-009.}

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Have a queued write reconciled | Maya, Sam, Jordan (older kid, limited login -- Later) | Always, for any member who already holds Grocery List write access per FEAT-06.SPEC-009 | -- |
| Have a queued write reconciled | Riley (Operator, support) | Never | Riley has no write access to the list (FEAT-06.SPEC-009) and therefore triggers no conflicts to resolve |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| event_time (internal ordering value) | Set to the device's local time at the moment of the action; used only to order and resolve concurrent writes to the same line | On every write | No -- not user-facing or overridable |

## Business Rules

- ASMP-25: offline changes sync without creating duplicates or conflicts left unresolved.
- Per the dependency map's Contention note for Grocery List: recalculation (FEAT-06.SPEC-002) and manual/offline edits merge -- plan-derived lines are recomputed while manual items, ticks, and "already have it" marks are preserved, consistent with this spec's resolution rules.
- When two event times are equal (down to the resolution available), the tiebreak is the order in which the changes are received by the shared list -- deterministic, but not itself a further conflict-resolution rule the household is exposed to.

## Edge Cases

- **Two edits to the same line carry identical event times** -- Resolved by order of arrival at the shared list as a deterministic tiebreak; the household sees one final, consistent value either way.
- **A member ticks an item and then immediately edits its quantity while offline, both queued** -- Both apply on reconnect, since they touch different fields; the item ends up ticked with the new quantity, with no conflict between them.
- **A line is deleted by one member while another has a queued quantity edit for it, made offline** -- The deletion wins per the Deletion Precedence rule; the queued edit is discarded silently and the item stays removed.
- **A recalculation (FEAT-06.SPEC-002) rewrites a plan-derived line's contributing dinners while a device has a queued offline tick on that same line** -- If the line (same ingredient) still exists after recalculation, the queued tick still applies; if recalculation removed that exact line because it is no longer needed, the queued tick is discarded, since there is nothing left to tick.
- **A member is removed from the household (FEAT-18) while they have offline changes queued** -- Those queued changes are discarded rather than applied on reconnect, consistent with FEAT-18's removal cascade; a removed member's device can no longer write to a list they no longer have access to.

## Acceptance Criteria

**FEAT-06.SPEC-008-AC-01:** Given Maya ticks an already-ticked item while briefly offline, when the tick syncs, then the item remains ticked with no double-toggle.

**FEAT-06.SPEC-008-AC-02:** Given two devices both tick the same item while both are offline, when both reconnect, then the item settles into a single ticked state.

**FEAT-06.SPEC-008-AC-03:** Given Maya and Sam both edit the same item's quantity before either syncs, when both changes reach the shared list, then the edit with the later original event time persists and the earlier one is discarded silently.

**FEAT-06.SPEC-008-AC-04:** Given Sam adds "bananas" offline while Maya has already added "bananas" online in the interim, when Sam reconnects, then the two merge into one line per FEAT-06.SPEC-007 rather than creating a duplicate.

**FEAT-06.SPEC-008-AC-05:** Given Maya deletes an item while Sam has a queued offline quantity edit for the same item, when Sam reconnects, then the item stays deleted and Sam's queued edit is discarded silently.

**FEAT-06.SPEC-008-AC-06:** Given Sam ticks an item and edits its quantity while offline, when he reconnects, then both changes apply and the item is ticked with the new quantity.

**FEAT-06.SPEC-008-AC-07:** Given a recalculation removes a plan-derived line while a device has a queued offline tick on that exact line, when the device reconnects, then the queued tick is discarded, since the line no longer exists.

**FEAT-06.SPEC-008-AC-08:** Given a recalculation updates but does not remove a plan-derived line while a device has a queued offline tick on it, when the device reconnects, then the queued tick still applies to the updated line.

**FEAT-06.SPEC-008-AC-09:** Given Jordan (older kid, limited login) has queued offline changes when his access to the household is still active, when he reconnects, then his changes reconcile normally like any other member with write access.

**FEAT-06.SPEC-008-AC-10:** Given a member is removed from the household while they have offline changes queued, when their device later reconnects, then those queued changes are discarded rather than applied.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 2 | 2 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Grocery List Access & Authorization Rules

## Overview

**Name:** Grocery List Access & Authorization Rules
**ID:** FEAT-06.SPEC-009
**Type:** Logic/Rule
**Purpose:** Defines who can view, add, tick, edit, or remove list items, and what an unauthorized visitor or unauthorized role experiences instead.
**Parent Feature:** FEAT-06 -- Shared Grocery List
**Governed Entity:** Grocery List (with its Grocery List Items)

## Scope and Non-Goals

**In Scope:**
- Who can view the household's current Grocery List, and under what conditions
- Who can add, tick, edit, remove, and mark "already have it" on Grocery List Items
- What an unauthorized visitor, an unauthenticated user, and an out-of-scope role each experience
- Riley's (Operator, support) conditional, read-only access tied to an open Support Request

**Non-Goals:**
- Household-level aisle names and unit-system settings -- owned by Units, Currency & Locale Configuration (FEAT-16); no role gains the ability to change those settings through this feature, including the older-kid role, whose access is scoped to list items, not list settings
- Field-level validation of list item content -- owned by FEAT-06.SPEC-006 (plan-derived) and FEAT-06.SPEC-007 (manual)
- Conflict resolution for concurrent or offline writes -- owned by FEAT-06.SPEC-008; this spec governs who may act, not how simultaneous actions merge

## Governed Entity

**Entity:** Grocery List
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| week | date | The plan week this list serves |
| aisle_grouping | text | The household's configured aisle names and order (owned by FEAT-16; read-only from this feature) |
| status | enum | Generated, Active, or Archived |

Item-level actions governed by this spec (add, tick, edit, remove, already-have-it) apply to Grocery List Item, whose fields are defined and field-validated in FEAT-06.SPEC-006 and FEAT-06.SPEC-007; this spec addresses authorization for actions on those items, not their field content.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-001 | Grocery List | On screen entry (which controls render at all) and on every action attempt (add, tick, edit, remove, already-have-it) |
| FEAT-06.SPEC-002 | Grocery List Generation & Recalculation | Authorization is not evaluated here directly -- recalculation is system-driven, not a member action, and operates regardless of who is viewing |
| FEAT-06.SPEC-003 | "Already Have It" Handling | On trigger, to confirm the requesting member already has Grocery List access before differentiating the pantry-logging outcome |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| week | No validation beyond data type -- this spec governs access, not field validation; set by FEAT-06.SPEC-002 | Always | -- | -- | -- |
| aisle_grouping | No validation beyond data type -- owned and validated by FEAT-16, read-only from this feature | Always | -- | -- | -- |
| status | No validation beyond data type -- transitions governed by FEAT-06.SPEC-002 (Generated/Active) and FEAT-06.SPEC-004 (Archived) | Always | -- | -- | -- |

## Cross-Field Rules

N/A -- this spec governs access and authorization, not cross-field validation. Field- and cross-field validation for list items is governed by FEAT-06.SPEC-006 (plan-derived) and FEAT-06.SPEC-007 (manual).

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the current Grocery List | Maya (Organiser) | Always | -- |
| View the current Grocery List | Sam (Other Adult Member) | Always | -- |
| View the current Grocery List | Jordan (older kid, limited login -- Later) | Always | -- |
| View the current Grocery List | Jordan (young kid profile, no login -- MVP) | Never | This profile has no login and cannot reach the screen at all |
| View the current Grocery List | Riley (Operator, support -- from v1) | Only while an open Support Request exists for the household (XBR-14) | Outside an open Support Request, access is blocked entirely: Riley cannot open the household's list, and no household list data is shown |
| Add a manual item | Maya, Sam, Jordan (older kid, limited login) | Always | -- |
| Add a manual item | Riley | Never | The Add-item control is not rendered; Riley's view is strictly read-only per Operator Read-Only Support Access (FEAT-22) |
| Tick or untick an item | Maya, Sam, Jordan (older kid, limited login) | Always | -- |
| Tick or untick an item | Riley | Never | Tick controls are not rendered |
| Edit an item's quantity | Maya, Sam, Jordan (older kid, limited login) | Always | -- |
| Edit an item's quantity | Riley | Never | Edit controls are not rendered |
| Remove an item | Maya, Sam, Jordan (older kid, limited login) | Always | -- |
| Remove an item | Riley | Never | Remove controls are not rendered |
| Mark "already have it" on a plan-derived item | Maya, Sam | Always; also creates a Pantry Item in the same tap, since both hold Full Pantry Input access (FEAT-06.SPEC-003) | -- |
| Mark "already have it" on a plan-derived item | Jordan (older kid, limited login) | Always; the item is removed from the list, but no Pantry Item is created, since this role's Pantry Input access is None (FEAT-06.SPEC-003) | -- |
| Mark "already have it" on a plan-derived item | Riley | Never | The control is not rendered |
| Change household aisle names or unit-system settings from this feature | Every role | Never -- this action does not exist within FEAT-06 at all | No control for this action is ever shown here, for any role, since aisle names and units are configured exclusively through FEAT-16 |
| Open a household's list directly (e.g., a stale or guessed link) | Unauthenticated visitor | Never | Redirected to the sign-in screen; no household list data is exposed |

## Defaults and Derivations

N/A -- this spec governs authorization only; it defines no default or derived field values. Field derivations for Grocery List Item are governed by FEAT-06.SPEC-006 (plan-derived) and FEAT-06.SPEC-007 (manual); Grocery List's own field transitions are governed by FEAT-06.SPEC-002 (creation, status) and FEAT-06.SPEC-004 (archive).

## Business Rules

- XBR-14: Riley's support access opens only for one household with an open Support Request, is strictly read-only, never shows more than the list itself, closes the moment the request is resolved, and every visit is recorded where Maya (the organiser) can see it.
- This spec's Authorization Rules table is the single source of truth for FEAT-06.SPEC-001's Access and Visibility table -- the two must never diverge.
- The older-kid role's Full access to Grocery List actions never extends to household-level list settings (aisle names, unit system) or to the grocery-ordering handoff (FEAT-20, Later); both are outside this feature's authorization surface entirely, not merely restricted for this role.

## Edge Cases

- **Riley's open Support Request is resolved while Riley is actively viewing the list** -- Access is revoked immediately: the screen redirects out with a message that support access has ended, and no further household list data is shown.
- **A household member is removed from the household (XBR-16) while they have the list open** -- Their access is revoked immediately; any further action attempt on the screen is rejected as if by an unauthenticated visitor.
- **Jordan's older-kid session expires mid-edit** -- Handled per FEAT-06.SPEC-001's Expired session state: no data is lost, but no further list actions succeed until re-authentication.
- **A member's role changes mid-session (e.g., organiser hand-over, per FEAT-09)** -- The Grocery List access level for both the outgoing and incoming organiser is unaffected by the hand-over, since both Maya and Sam already hold Full access to this feature; no authorization change is triggered by an organiser hand-over specifically.

## Acceptance Criteria

**FEAT-06.SPEC-009-AC-01:** Given Maya (Organiser) opens the Grocery List, when the screen loads, then she can view and act on every item without restriction.

**FEAT-06.SPEC-009-AC-02:** Given Sam (Other Adult Member) opens the Grocery List, when the screen loads, then he can view and act on every item without restriction.

**FEAT-06.SPEC-009-AC-03:** Given Jordan (young kid profile, no login), when any attempt is made to view the list, then it never succeeds, since this profile has no login and cannot reach the screen.

**FEAT-06.SPEC-009-AC-04:** Given Jordan (older kid, limited login) opens the Grocery List, when the screen loads, then he can add, tick, edit, remove, and mark "already have it" on items, but no household list-setting control is ever shown to him.

**FEAT-06.SPEC-009-AC-05:** Given Riley (Operator) attempts to open a household's Grocery List with no open Support Request, when the attempt is made, then access is blocked entirely and no household list data is shown.

**FEAT-06.SPEC-009-AC-06:** Given Riley opens a household's Grocery List while a Support Request is open, when the screen loads, then he sees every item read-only, with no tick, add, edit, remove, or already-have-it controls.

**FEAT-06.SPEC-009-AC-07:** Given the open Support Request Riley was viewing under is resolved while he is on the screen, when the resolution occurs, then his access is revoked immediately and he is redirected out with a message that support access has ended.

**FEAT-06.SPEC-009-AC-08:** Given Maya taps "Already have it" on a plan-derived item, when the action completes, then the item is removed and a Pantry Item is created, since Maya holds Full Pantry Input access.

**FEAT-06.SPEC-009-AC-09:** Given Jordan (older kid, limited login) taps "Already have it" on a plan-derived item, when the action completes, then the item is removed but no Pantry Item is created, since his Pantry Input access is None.

**FEAT-06.SPEC-009-AC-10:** Given any household member looks for a way to change aisle names from the Grocery List screen, when they look, then no such control exists anywhere on this feature's screens, for any role.

**FEAT-06.SPEC-009-AC-11:** Given an unauthenticated visitor requests a household's Grocery List directly, when the request is made, then they are redirected to the sign-in screen and no household data is exposed.

**FEAT-06.SPEC-009-AC-12:** Given a household member is removed from the household while their Grocery List screen is open, when the removal takes effect, then any further action they attempt is rejected as if they were unauthenticated.

**FEAT-06.SPEC-009-AC-13:** Given the organiser role is handed over from Maya to Sam, when the hand-over completes, then both continue to have Full Grocery List access exactly as before, since the hand-over does not itself change either member's Grocery List authorization.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 0 (N/A -- see Cross-Field Rules section) | 0 |
| Authorization Rules | 15 | 15 |
| Defaults/Derivations | 0 (N/A -- see Defaults and Derivations section) | 0 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |
