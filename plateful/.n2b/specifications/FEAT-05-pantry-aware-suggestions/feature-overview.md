---
document_type: feature-overview
feature_number: FEAT-05
feature_name: Pantry-Aware Suggestions
feature_slug: pantry-aware-suggestions
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 7
screen_count: 1
automation_count: 2
logic_rule_count: 4
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Pantry-Aware Suggestions

## Summary

**Feature:** Pantry-Aware Suggestions
**ID:** FEAT-05
**Description:** The household tells Plateful what it already has on hand, and the weekly plan favors recipes that use those ingredients up before they go to waste.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** The brief's central "nothing gets thrown away" promise depends on this (BRIEF.md, Vision, The Experience: "Thursday says 'uses the spinach and feta you already have'"). The brief leaves pantry depth as an open question and leans toward a simple "tell me what I have" model rather than detailed inventory tracking. The only pantry-aware competitor (Samsung Food, paid tier) is described as "basic" and unable to reflect leftovers or real kitchen behavior, so pantry awareness is a documented gap Plateful targets alongside Leftover Rollover to Lunches (FEAT-11).

**Key Capabilities:**
- Log what's on hand — Household member notes an ingredient they already have, in plain language
- See it used in the plan — A plan that uses a logged pantry item calls that out explicitly (e.g., "uses the spinach and feta you already have")
- Clear used items — Household can mark a pantry item as used up once the week's plan consumes it; after the dinner that used an item has passed, a one-tap "used it up?" prompt appears on the pantry list
- Keep it off the list — A logged pantry item is left off the week's grocery list, so the household does not buy what it already has

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-05.SPEC-001 | Pantry List & Item Entry | Screen | Maya, Sam | Household member views the current pantry list, adds an item in plain language, and clears an item once it is used or gone |
| FEAT-05.SPEC-002 | Used-It-Up Prompt Trigger | Automation | Maya, Sam | Once the dinner that used a logged pantry item has passed, surfaces a one-tap "used it up?" prompt on the pantry list |
| FEAT-05.SPEC-003 | Pantry Item Duplicate Merge | Automation | Maya, Sam | Merges a newly logged or synced item into an existing entry with the same name instead of creating a second row |
| FEAT-05.SPEC-004 | Pantry Item Field Validation | Logic/Rule | Maya, Sam | Enforces the pantry item name's required, 1–80 character, free-text rule with no rigid inventory schema and no item-count limit |
| FEAT-05.SPEC-005 | Pantry-Aware Plan Weighting Tier Gate | Logic/Rule | Maya, Sam | Governs which pantry behaviors run on the free tier (logging, off-list exclusion) versus the paid tier (plan weighting toward logged items) |
| FEAT-05.SPEC-006 | Pantry-to-Recipe Matching for Plan Callout | Logic/Rule | Maya, Sam, Jordan (older kid, Later), Riley | Defines how a logged pantry item is matched against a candidate recipe's ingredients to produce the plan's pantry callout |
| FEAT-05.SPEC-007 | Pantry Item Off-Grocery-List Exclusion Rule | Logic/Rule | Maya, Sam, Jordan (older kid, Later), Riley | Defines that an Active logged pantry item is left off the week's grocery list on both tiers, and that marking a list item "already have it" can log it to the pantry in the same tap |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Log what's on hand | FEAT-05.SPEC-001, FEAT-05.SPEC-004 | The entry screen captures a free-text item name; the validation rule enforces the 1–80 character, required-name constraint | Phase 2 (Explicit) |
| See it used in the plan | FEAT-05.SPEC-006, FEAT-05.SPEC-005 | The matching rule determines which logged items a candidate recipe uses (feeding FEAT-03's callout); the tier gate confirms this only happens on the paid tier | Phase 2 (Explicit) |
| Clear used items | FEAT-05.SPEC-001, FEAT-05.SPEC-002 | The screen's tap-to-clear action and the one-tap "used it up?" prompt both set the item to Used/Removed; the automation surfaces the prompt after the consuming dinner passes | Phase 2 (Explicit) |
| Keep it off the list | FEAT-05.SPEC-007 | The exclusion rule keeps an Active pantry item off grocery-list generation on both tiers | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — specs not directly tied to a Key Capability, surfaced by Phases 3–6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-05.SPEC-003 | Pantry Item Duplicate Merge | Phase 3 (Entity-Lifecycle) / Phase 4 (Trigger-Response) | The dependency map's Contention line for Pantry Item states resolution is merge — "the same item added twice becomes one entry" — a create-time side effect the Key Capabilities never name |
| FEAT-05.SPEC-005 | Pantry-Aware Plan Weighting Tier Gate | Phase 5 (Rule Discovery) | The feature's own Access field and XBR-05 make plan weighting a paid-tier-only behavior distinct from the free-tier logging and off-list capabilities; a conditional rule shared across FEAT-03, FEAT-06, and FEAT-14 needed its own spec |
| FEAT-05.SPEC-006 | Pantry-to-Recipe Matching for Plan Callout | Phase 5 (Rule Discovery) | The Interactions field states FEAT-03 "reads current Pantry Items when selecting meals" but the feature description never defines the matching logic itself; as the entity's owning feature, FEAT-05 must specify the derivation rule FEAT-03 consumes |
| FEAT-05.SPEC-007 | Pantry Item Off-Grocery-List Exclusion Rule | Phase 5 (Rule Discovery) | XBR-04 names FEAT-05 as the rule's owning authority ("owns Pantry Item data"); the exclusion behavior FEAT-06 depends on must be specified from this feature's side |

## Entity-Lifecycle Coverage Matrix

**Entity: Pantry Item**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-05.SPEC-001 | Household member enters a free-text item name on the pantry list, validated by SPEC-004 | Also created inbound by FEAT-06's "already have it" tap (governed by SPEC-007); a duplicate name is merged rather than duplicated (SPEC-003) |
| Read (single) | N/A — no dedicated single-item view | Pantry items are only ever shown and acted on as rows within the list (SPEC-001); no separate detail screen exists because the item carries no fields beyond its name and status | -- |
| Read (list) | FEAT-05.SPEC-001 | The pantry list screen displays every Active item for the household | -- |
| Update | FEAT-05.SPEC-001, FEAT-05.SPEC-002, FEAT-05.SPEC-003 | SPEC-001 sets status to Used/Removed on a manual clear or a tapped "used it up?" prompt; SPEC-002 triggers the prompt's appearance; SPEC-003 updates the existing entry on a duplicate add instead of creating a new row | -- |
| Delete/Archive | FEAT-05.SPEC-001 | Hard delete: clearing an item (manually or via the "used it up?" prompt) permanently removes the row from the pantry list; no restore path -- re-adding the same name creates a fresh entry through SPEC-001; no cascade beyond the household-deletion cascade owned by FEAT-18; no retention/purge job applies because the delete is immediate and the item carries no history to retain | A logged item that goes unused for weeks is never auto-removed (feature's own "Stale item" flow) -- see Non-Goals |
| State Transition | FEAT-05.SPEC-001, FEAT-05.SPEC-002 | Active -> Used/Removed, triggered either by a manual clear (SPEC-001) or by tapping the automation-surfaced "used it up?" prompt (SPEC-002) | No transition back to Active; a cleared item is gone, not reactivated |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Planned Meal | FEAT-05.SPEC-006 | Reads a candidate recipe's ingredients (via the Planned Meal it would become) to determine which logged pantry items it would use, for the plan callout |
| Subscription | FEAT-05.SPEC-005 | Reads the household's tier to determine whether pantry-aware plan weighting applies this week |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Household member submits a new pantry item | Validate the item name (required, 1–80 characters) before saving | Inline in triggering screen, governed by | FEAT-05.SPEC-001 / FEAT-05.SPEC-004 |
| A submitted or synced item name matches an existing Active item | Merge into the existing entry instead of creating a duplicate row | Standalone Automation | FEAT-05.SPEC-003 |
| Item save succeeds | Confirm instantly on the list (per the feature's own Loading state) | Inline in triggering screen | FEAT-05.SPEC-001 |
| Item save fails | Keep the typed item visible with a retry option (per the feature's own Error state) | Inline in triggering screen | FEAT-05.SPEC-001 |
| Household adds an item while offline | Queue the add locally; sync and reconcile once connectivity returns (per the feature's own Offline-degraded state) | Inline in triggering screen, reconciled via | FEAT-05.SPEC-001 / FEAT-05.SPEC-003 |
| The dinner that used a logged pantry item passes | Surface a one-tap "used it up?" prompt on the pantry list the next time it is viewed | Standalone Automation | FEAT-05.SPEC-002 |
| Household taps "used it up?" or manually clears an item | Set the item's status to Used/Removed and remove it from the list (hard delete, no restore) | Inline in triggering screen | FEAT-05.SPEC-001 |
| Weekly plan generates for a paid-tier household (FEAT-03) | Match Active pantry items against candidate recipes and weight selection toward matches; attach the pantry callout to the chosen dinner | Standalone Logic/Rule, gated by Standalone Logic/Rule | FEAT-05.SPEC-006 / FEAT-05.SPEC-005 |
| Weekly plan generates for a free-tier or downgraded household | No pantry weighting applies; the household still receives a complete plan | Standalone Logic/Rule (governs the no-op) | FEAT-05.SPEC-005 |
| Grocery list generates or recalculates (FEAT-06) | Exclude every Active pantry item from the list on either tier | Standalone Logic/Rule | FEAT-05.SPEC-007 |
| Household taps "already have it" on a grocery list line (FEAT-06) | Log the ingredient to the pantry in the same tap (or merge into an existing entry) | Cross-feature inbound, handled by | FEAT-06 trigger; FEAT-05.SPEC-001 / FEAT-05.SPEC-003 / FEAT-05.SPEC-007 |
| Household's subscription downgrades or lapses (FEAT-14) | Pantry-aware plan weighting stops applying; logging and off-list exclusion continue unaffected; no past pantry data is removed | Standalone Logic/Rule (cross-feature effect) | FEAT-05.SPEC-005 |
| Household is deleted (FEAT-18) | All of the household's pantry items are removed as part of the account-deletion cascade | Cross-feature — logged in touchpoints | FEAT-18 responsibility |

## Shared Context

**Shared Entities:**
- Pantry Item — created and deleted by SPEC-001 (and inbound from FEAT-06 via SPEC-007); updated by SPEC-001, SPEC-002, and SPEC-003; read by SPEC-005, SPEC-006, and SPEC-007, and cross-feature by FEAT-03 (plan weighting, paid tier) and FEAT-06 (off-list exclusion, both tiers). Fields: item_name (free text, 1–80 characters), added_by, status (Active, Used/Removed), used_prompt.

**Shared UI Patterns:**
- Pantry row pattern — SPEC-001 defines a single list-row layout (item name, added-by indicator, clear action, and the "used it up?" prompt chip when SPEC-002 fires) used consistently for every item on the list; there is no separate detail view to keep in sync.
- Add-item field — a single free-text input at the top of the pantry list, validated per SPEC-004, shared by direct entry (SPEC-001) and by the inbound "already have it" path from FEAT-06 (SPEC-007), so both entry points produce the same kind of Pantry Item row.

**Shared Validation:**
- SPEC-004 (Pantry Item Field Validation) is the single source of truth for the item_name rule; SPEC-001 and the inbound FEAT-06 create path (governed by SPEC-007) both defer to it rather than re-deriving the character limit or required-field check.
- SPEC-005 (Pantry-Aware Plan Weighting Tier Gate) is the single source of truth for which pantry behaviors are tier-gated; SPEC-006 defers to it before running the matching computation, and it is the reference point for FEAT-03's and FEAT-14's tier-gating logic as well.

## Internal Dependency Map

```
SPEC-001 (Pantry List & Item Entry) -> [household member submits a new item] -> SPEC-004 (Pantry Item Field Validation)
SPEC-001 (Pantry List & Item Entry) -> [validated item saved, name matches an existing entry] -> SPEC-003 (Pantry Item Duplicate Merge)
SPEC-001 (Pantry List & Item Entry) -> [offline add syncs on reconnect] -> SPEC-003 (Pantry Item Duplicate Merge)
[the dinner using a logged item passes, FEAT-03] -> [inbound trigger] -> SPEC-002 (Used-It-Up Prompt Trigger)
SPEC-002 (Used-It-Up Prompt Trigger) -> [prompt shown on next view] -> SPEC-001 (Pantry List & Item Entry)
SPEC-001 (Pantry List & Item Entry) -> [household taps "used it up?" or clears manually] -> [item hard-deleted, status Used/Removed recorded]
[weekly plan generates, FEAT-03] -> [checks tier via] -> SPEC-005 (Pantry-Aware Plan Weighting Tier Gate) -> [paid tier: matches candidates via] -> SPEC-006 (Pantry-to-Recipe Matching for Plan Callout)
[grocery list generates or recalculates, FEAT-06] -> [excludes items via] -> SPEC-007 (Pantry Item Off-Grocery-List Exclusion Rule)
[household taps "already have it" on a grocery item, FEAT-06] -> [inbound trigger, governed by] -> SPEC-007 (Pantry Item Off-Grocery-List Exclusion Rule) -> [creates or merges via] -> SPEC-001 (Pantry List & Item Entry) / SPEC-003 (Pantry Item Duplicate Merge)
```

**Default Entry:** SPEC-001 (Pantry List & Item Entry) — the screen shown when a household member navigates to the pantry area.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-05.SPEC-006 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Plan generation reads Active Pantry Items and the matching rule to weight selection and render the pantry callout, on the paid tier | Weekly plan generates |
| FEAT-05.SPEC-005 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Confirms to plan generation that a free-tier or downgraded household receives a complete plan with no pantry weighting | Weekly plan generates for a free-tier household |
| FEAT-05.SPEC-007 | Outbound | FEAT-06 (Shared Grocery List) | Logged pantry items are excluded from grocery list generation and recalculation on both tiers | Grocery list generates or recalculates |
| FEAT-05.SPEC-001 / FEAT-05.SPEC-003 / FEAT-05.SPEC-007 | Inbound | FEAT-06 (Shared Grocery List) | Tapping "already have it" on a grocery list line logs (or merges into) a pantry item in the same tap | Household taps "already have it" |
| FEAT-05.SPEC-005 | Inbound | FEAT-14 (Subscription & Billing Management) | The household's tier state gates whether plan weighting applies; a downgrade or lapsed payment removes no past pantry data | Subscription tier changes, or a payment lapses |
| FEAT-05.SPEC-001 | Inbound | FEAT-18 (Account & Data Management) | Household deletion cascades to remove all of the household's Pantry Items | Household is deleted |
| FEAT-05.SPEC-001 | Outbound | FEAT-22 (Operator Read-Only Support Access) | Riley's View access to Pantry Input is exercised through the operator's read-only support view, not through this feature's own screen | Riley opens the household's record against an open Support Request |

## Non-Functional Notes

**Data volumes / growth:** Pantry lists are designed to stay short and current rather than a full inventory, with no fixed item-count limit, across several thousand households of 2–6 members in the first year (ASMP-24); list responsiveness must not degrade as the household base grows.

**Responsiveness:** Adding an item confirms instantly (the feature's own Loading-state expectation); the pantry list must stay usable offline, with adds queued locally and synced once connectivity returns (ASMP-25); every primary action (add, clear, tap the "used it up?" prompt) must be reachable one-handed with large tap targets, per the product's accessibility baseline (ASMP-29).

**Data sensitivity / privacy:** Pantry Item data is low-sensitivity household personal data — what is currently in the household's fridge or cupboard — private to the household and never sold or used for advertising (per the dependency map's Pantry Item entity, and ASMP-26). No kid profile ever logs or is shown pantry data directly (Access Matrix: both kid rows are None on Pantry Input); Maya and Sam are the only members who see or log it, and Riley's access is read-only and only through an open Support Request.

**Compliance flags:** N/A — Pantry Item carries no health, financial, or children's-data regime; it is ordinary low-sensitivity household data covered by the product's general personal-data export and deletion rights (ASMP-27, via FEAT-18), not by any special compliance category.

## Non-Goals

- **Detailed pantry inventory tracking (quantities, expiry dates, barcode stock-taking)** — Excluded per scope-boundaries.md (SC-11): the brief's Open Questions weighed a detailed inventory against a lighter "just tell me what I have" model, and the product follows the lighter model; item_name is the only captured field, with no quantity, unit, or expiry tracking.
- **Automatic removal of stale or unused pantry items** — Intentional lifecycle decision stated directly in the feature's own Primary Flows & Alternates ("Stale item" flow): an item that goes unused for several weeks is never auto-removed; the household clears it manually when it is used or gone. No retention window or purge job applies.
- **Pantry-driven recipe search outside the weekly plan** — Excluded by the feature's own Key Capabilities, which scope pantry awareness to weighting the weekly plan's dinner selection (paid tier) and to keeping items off the grocery list (both tiers); an on-demand "what can I cook with what I have" search tool is a different capability the product definition does not describe, and adding one here would expand the feature beyond BRIEF.md's "tell me what I have" model.
- **Notifications about pantry activity** — N/A per the feature's own Communications field ("N/A — this feature has no notifications of its own; its effect surfaces as a callout inside the weekly plan"); no email, push, or SMS channel is introduced for pantry logging, clearing, or the "used it up?" prompt, which stays an in-app, same-screen interaction.
