---
document_type: feature-overview
feature_number: FEAT-06
feature_name: Shared Grocery List
feature_slug: shared-grocery-list
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 9
screen_count: 1
automation_count: 3
logic_rule_count: 4
integration_count: 1
notification_count: 0
---

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
