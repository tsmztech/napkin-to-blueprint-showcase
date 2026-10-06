# FEAT-05 — Pantry-Aware Suggestions

This chapter covers FEAT-05, Pantry-Aware Suggestions, a Core-tier feature. It contains 7 specifications carrying 78 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-05.SPEC-001 | Pantry List & Item Entry | screen | 15 |
| FEAT-05.SPEC-002 | Used-It-Up Prompt Trigger | automation | 9 |
| FEAT-05.SPEC-003 | Pantry Item Duplicate Merge | automation | 9 |
| FEAT-05.SPEC-004 | Pantry Item Field Validation | logic-rule | 12 |
| FEAT-05.SPEC-005 | Pantry-Aware Plan Weighting Tier Gate | logic-rule | 10 |
| FEAT-05.SPEC-006 | Pantry-to-Recipe Matching for Plan Callout | logic-rule | 11 |
| FEAT-05.SPEC-007 | Pantry Item Off-Grocery-List Exclusion Rule | logic-rule | 12 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Pantry List & Item Entry

## Overview

**Name:** Pantry List & Item Entry
**ID:** FEAT-05.SPEC-001
**Type:** Screen
**Purpose:** Household member views the household's current pantry list, adds an item they already have on hand in plain language, and clears an item once it is used or gone.
**Parent Feature:** FEAT-05 -- Pantry-Aware Suggestions

## Scope and Non-Goals

**In Scope:**
- Displaying every Active Pantry Item for the household as a list
- A single free-text add-item field for logging a new item
- Manually clearing (hard-deleting) a Pantry Item once it is used or gone
- Displaying the "used it up?" prompt chip on a row when FEAT-05.SPEC-002 has surfaced it, and handling the tap that clears the item from that prompt
- Empty, loading, error, and offline-degraded presentation for this screen

**Non-Goals:**
- Detailed pantry inventory fields such as quantity, unit, or expiry date -- excluded per scope-boundaries.md (SC-11): the product follows the lighter "tell me what I have" model instead of a detailed inventory, so the item carries only a name.
- A separate single-item detail view -- excluded per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix: a Pantry Item carries no fields beyond its name and status, so a dedicated detail screen would duplicate the row shown here with no added information.
- Determining when the "used it up?" prompt should appear -- governed by FEAT-05.SPEC-002 (Used-It-Up Prompt Trigger); this screen only renders the prompt chip once that automation has surfaced it and handles the tap.
- Merging a newly typed item into an existing entry with the same name -- governed by FEAT-05.SPEC-003 (Pantry Item Duplicate Merge); this screen submits the typed name and displays whichever row results.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Main navigation (default entry) | Household member (Maya or Sam) taps "Pantry" in the app's primary navigation | None -- the current Active pantry list loads |
| FEAT-06.SPEC-001 (Grocery List) | Maya or Sam taps "Already have it" on a plan-derived grocery list line, then taps the "View Pantry" action on the confirmation toast (action offered only to roles with Pantry Input access, per FEAT-05.SPEC-007) | The newly logged or merged item, shown highlighted at the top of the list |
| FEAT-05.SPEC-002 (Used-It-Up Prompt Trigger) | Household member opens the pantry list after the automation has surfaced a prompt | The prompt chip is shown on the affected row without further navigation context |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen -- the household's complete Active pantry list | Add an item, clear an item, tap "used it up?" | -- |
| Sam (Other Adult Member) | Full screen -- the household's complete Active pantry list | Add an item, clear an item, tap "used it up?" | -- |
| Jordan (young kid profile, no login -- MVP) | No -- has no login and no product access | No | This role has no account and cannot open any screen; there is nothing to redirect, since the profile never authenticates |
| Jordan (older kid, limited login -- Later) | No -- Pantry Input is None for this role | No | The "Pantry" entry is not shown in this role's main navigation; a direct link is redirected to the plan (FEAT-03) with no error message, since this area does not exist for this role |
| Riley (Operator, support -- from v1) | Full screen, read-only, and only while an open Support Request exists for the household (FEAT-22) | No -- cannot add, clear, or answer the "used it up?" prompt | Outside an open Support Request, this screen is not reachable; the add-item field and clear/prompt actions are not rendered in the read-only support view |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in as a household member, the user lands on the pantry list, not an intermediate screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any item text typed but not yet saved is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Pantry" with the primary navigation's standard back/menu control; no page-level action button in the header (adding happens inline in the body).

**Body:** A single-column list screen.
- **Add-item field:** A single free-text input pinned above the list, with placeholder text "What do you have?" and an inline "Add" control beside it. This is the Shared Context's Add-item field, reused verbatim by the inbound FEAT-06 "already have it" path (governed by FEAT-05.SPEC-007).
- **Pantry list:** Below the add-item field, every Active Pantry Item for the household as one row each, ordered most-recently-added first. Each row shows:
  - The item's name (free text, as entered)
  - A small "added by" indicator naming the member who logged it
  - A clear action (a tap target that removes the row)
  - A "used it up?" prompt chip, shown only on a row where FEAT-05.SPEC-002 has surfaced the prompt, placed alongside the clear action

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Add-item field and list are full width, single column, as described above; the clear action and "used it up?" chip remain visible on the row without requiring a swipe, consistent with the product's one-handed, large-tap-target accessibility baseline (ASMP-29).
- **Medium size class and above:** The list remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Add-item field | Type | Captures free-text item name | Field shows entered text | Standard input focus state |
| Add-item field / Add control | Tap "Add" (or submit) | Validates the name via FEAT-05.SPEC-004, then triggers FEAT-05.SPEC-003 (Pantry Item Duplicate Merge) to save or merge it | New or merged row appears at the top of the list; field clears | Row appears instantly (per the feature's own Loading-state expectation); no full-page spinner |
| Add-item field | Submit with invalid name | Validation via FEAT-05.SPEC-004 fails | Field shows error state | Exact error message from FEAT-05.SPEC-004 shown inline below the field; typed text is preserved |
| Pantry row: clear action | Tap | Sets the item's status to Used/Removed and hard-deletes the row (no restore path) | Row is removed from the list with a brief exit animation | -- |
| Pantry row: "used it up?" chip | Tap | Sets the item's status to Used/Removed and hard-deletes the row, same as the manual clear action | Row is removed from the list with a brief exit animation | -- |

### Accessibility Notes

- **Focus order:** Add-item field -> Add control -> each pantry row's clear action (and "used it up?" chip, when present), top to bottom in list order.
- **Add feedback:** The newly added or merged row is announced to assistive technology when it appears at the top of the list.
- **Validation announcements:** When the add-item field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Clear feedback:** A row's removal (manual clear or "used it up?" tap) is announced to assistive technology.
- **Keyboard alternatives:** Every action on this screen -- typing, adding, clearing, and answering the "used it up?" prompt -- is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | A short prompt to add what's on hand replaces the list body (per the feature's own Empty-state expectation); add-item field remains at the top | Household has zero Active pantry items | An item is successfully added |
| Populated | The pantry list as described in Layout and Content | One or more Active pantry items exist | List becomes empty (last item cleared) |
| Loading | A brief inline indicator on the add-item field while the add request is in flight | User submits a new item | Save succeeds (row appears) or fails (Error state) |
| Error | The typed item remains visible in the add-item field with a retry option, per the feature's own Error-state expectation | The add request fails | User retries and the save succeeds, or navigates away |
| Offline/Degraded | Banner "You're offline -- this item will be saved when you reconnect." at the top; the add-item field and list remain usable; a submitted add is queued locally and shown in the list immediately as a pending row | Connectivity is lost while the screen is open, or the user adds an item while already offline | Connectivity restored -- the queued add syncs automatically (reconciled by FEAT-05.SPEC-003) and the pending marker clears |

## Validation Rules

Validation governed by FEAT-05.SPEC-004 (Pantry Item Field Validation). See that spec for the item name's required, 1-80 character rule. This screen applies validation on submit of the add-item field.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back/menu control | Main navigation destination the household last used | -- |
| (No other outbound navigation -- this screen has no detail view to navigate into per the Entity-Lifecycle Coverage Matrix) | -- | -- |

## Data Model

**Creates:** Pantry Item -- item_name (from the add-item field, validated by FEAT-05.SPEC-004), added_by (the signed-in household member), status set to Active. When the typed name matches an existing Active item, FEAT-05.SPEC-003 merges into that entry instead of creating a new one.
**Reads:** Pantry Item -- item_name, added_by, status (Active only), used_prompt (to render the "used it up?" chip when set by FEAT-05.SPEC-002) for every item belonging to the household.
**Updates:** Pantry Item -- status set to Used/Removed on a manual clear or a tapped "used it up?" prompt.
**Deletes:** Pantry Item -- hard-deleted immediately when status is set to Used/Removed (manual clear or "used it up?" tap); no restore path -- re-adding the same name creates a fresh entry.

## Business Rules

- FEAT-05.SPEC-004 (Pantry Item Field Validation) is the single source of truth for the item_name rule; this screen defers to it rather than re-deriving the character limit or required-field check.
- FEAT-05.SPEC-003 (Pantry Item Duplicate Merge) runs automatically whenever a submitted or synced item name matches an existing Active item; the user cannot bypass it.
- XBR-04: this screen is the surface where FEAT-06's inbound "already have it" tap creates or merges a Pantry Item, governed by FEAT-05.SPEC-007.
- Clearing an item (manually or via the "used it up?" prompt) is a hard delete with no restore path; an item that goes unused for weeks is never auto-removed by the product (see Non-Goals of the Feature Breakdown Brief).

## Edge Cases

- **User submits the add-item field twice rapidly** -- The second submit is ignored while the first save is in progress (the Add control shows its loading state).
- **User taps clear on a row while the "used it up?" chip is also present on it** -- Either control produces the same result: the item is set to Used/Removed and removed from the list. There is no double-clear.
- **Network failure during add** -- Error state per the States table: the typed item stays visible with a retry option, per the feature's own Error state; the item is not saved until retried successfully.
- **Household member clears an item that another household member is simultaneously clearing** -- The clear (hard delete) is idempotent: whichever request lands first removes the row; the second request finds no matching Active item and completes as a no-op, per the dependency map's Contention note for Pantry Item ("clearing is idempotent"). No error is shown to either member.
- **Another household member adds an item while this screen is open** -- The list is live-updating: the new row appears without the viewer needing to refresh.
- **Household member re-adds a name that was previously cleared** -- A fresh Active entry is created (per the Entity-Lifecycle Coverage Matrix: "re-adding the same name creates a fresh entry"), not a restore of the cleared row.
- **Item logged while offline is cleared before it syncs** -- The locally queued add and the local clear reconcile on reconnect so no item ever appears; nothing is sent to the household's shared list.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-004 (Pantry Item Field Validation) | References (inbound) | Item-name validation rules applied to the add-item field |
| FEAT-05.SPEC-003 (Pantry Item Duplicate Merge) | Triggers (outbound) | Every add and every offline sync passes through the merge check |
| FEAT-05.SPEC-002 (Used-It-Up Prompt Trigger) | Triggers (inbound) | Surfaces the "used it up?" prompt chip shown on a row when the consuming dinner has passed |
| FEAT-05.SPEC-007 (Pantry Item Off-Grocery-List Exclusion Rule) | References (inbound) | Governs the inbound "already have it" create/merge path from the Shared Grocery List |
| FEAT-06 Shared Grocery List (grocery list screen) | Navigation (inbound) | User arrives here after tapping "already have it" on a grocery list line |
| FEAT-03 AI Weekly Dinner Plan Generation | References (outbound) | The weekly plan's pantry callout (FEAT-05.SPEC-006, gated by FEAT-05.SPEC-005) reflects Active items shown on this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| pantry_item_added | added_by role, entry source (direct add / synced offline) | An item is successfully saved as a new Active entry | supports success-metrics.md: "Pantry Items Used" |
| pantry_item_cleared | clear source (manual / used-it-up prompt) | An item's status is set to Used/Removed and the row is hard-deleted | supports success-metrics.md: "Pantry Items Used" |
| pantry_used_prompt_answered | outcome (cleared / dismissed, if a dismiss path exists on this screen) | Household member acts on a "used it up?" prompt chip shown on this screen | supports success-metrics.md: "Pantry Items Used" |
| pantry_list_viewed_empty | N/A | The screen renders its Empty state | N/A -- this event measures reach into the empty-state experience, not a metric in success-metrics.md |

## Acceptance Criteria

**FEAT-05.SPEC-001-AC-01:** Given Maya is on the Pantry List screen with an empty pantry, when she types "spinach, feta" into the add-item field and taps Add, then a new row "spinach, feta" appears at the top of the list, showing Maya as the added-by member.

**FEAT-05.SPEC-001-AC-02:** Given Sam is on the Pantry List screen, when he submits the add-item field with no text entered, then the field shows an error state with the exact message from FEAT-05.SPEC-004 and no row is added.

**FEAT-05.SPEC-001-AC-03:** Given Maya is on the Pantry List screen with a pantry item "milk" already Active, when she taps the clear action on that row, then the row disappears immediately and the item's status is set to Used/Removed with no restore option.

**FEAT-05.SPEC-001-AC-04:** Given Sam is on the Pantry List screen and FEAT-05.SPEC-002 has surfaced a "used it up?" prompt chip on the "eggs" row, when he taps the chip, then the row disappears immediately and the item's status is set to Used/Removed.

**FEAT-05.SPEC-001-AC-05:** Given Maya has zero Active pantry items, when she opens the Pantry List screen, then the Empty state appears with a short prompt to add what's on hand, and the add-item field remains available.

**FEAT-05.SPEC-001-AC-06:** Given Sam is on the Pantry List screen with a slow connection, when he submits a new item and the save fails, then the typed item remains visible in the add-item field with a retry option and no row is added.

**FEAT-05.SPEC-001-AC-07:** Given Maya loses connectivity while on the Pantry List screen, when she adds "yoghurt", then the banner "You're offline -- this item will be saved when you reconnect." appears and "yoghurt" is shown in the list as a pending row.

**FEAT-05.SPEC-001-AC-08:** Given Maya's offline add from AC-07 is still pending, when connectivity returns, then the pending row syncs automatically (through FEAT-05.SPEC-003) and its pending marker clears without user action.

**FEAT-05.SPEC-001-AC-09:** Given Maya submits an item and taps Add twice in rapid succession, when the first save is still in progress, then the second tap has no effect and only one row is created.

**FEAT-05.SPEC-001-AC-10:** Given Maya and Sam both tap the clear action on the same "onions" row at effectively the same time, when both requests reach the system, then the row is removed once, and the request that lands second completes as a no-op with no error shown to either member.

**FEAT-05.SPEC-001-AC-11:** Given Sam has the Pantry List screen open, when Maya adds "carrots" from her own device, then the "carrots" row appears on Sam's screen without him needing to refresh.

**FEAT-05.SPEC-001-AC-12:** Given Jordan (young kid profile, no login) has no account, when any attempt is made to reach the Pantry List screen on Jordan's behalf, then no such access exists -- the profile has no sign-in and cannot open any screen.

**FEAT-05.SPEC-001-AC-13:** Given Jordan (older kid, limited login) is signed in, when this role looks for a "Pantry" entry in the main navigation, then it is not shown, and a direct link to the pantry screen redirects to the current plan (FEAT-03) with no error message.

**FEAT-05.SPEC-001-AC-14:** Given Riley (Operator) is viewing a household's pantry through an open Support Request, when Riley looks at the screen, then the add-item field, clear actions, and "used it up?" chip are not rendered -- the view is read-only.

**FEAT-05.SPEC-001-AC-15:** Given Maya's session expires while she has typed "flour" but not yet submitted it, when the session-expired dialog appears and she signs back in, then "flour" is still present in the add-item field.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 (empty, populated, loading, error, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



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



# Automation Spec: Pantry Item Duplicate Merge

## Overview

**Name:** Pantry Item Duplicate Merge
**ID:** FEAT-05.SPEC-003
**Type:** Automation
**Purpose:** Merges a newly logged or synced Pantry Item into an existing Active entry with the same name instead of creating a second row.
**Parent Feature:** FEAT-05 -- Pantry-Aware Suggestions

## Scope and Non-Goals

**In Scope:**
- Comparing a submitted or synced item name against the household's existing Active Pantry Items
- Merging into the existing entry when a match is found, on direct add, offline-sync reconciliation, and the inbound FEAT-06 "already have it" create path
- Idempotent behavior so a double submission of the same name never creates two rows

**Non-Goals:**
- Validating the item name's format or length -- handled by FEAT-05.SPEC-004 (Pantry Item Field Validation), which runs before this automation ever sees the name
- Fuzzy or semantic matching across different wordings of the same ingredient (e.g., "spinach" vs. "baby spinach") -- excluded per the Feature Breakdown Brief's own Validation & Limits: item_name is free text with no rigid inventory schema, and the product follows the lighter "tell me what I have" model (scope-boundaries.md, SC-11) rather than a normalized ingredient taxonomy that exact-name matching alone cannot provide
- Merging items across different households -- Pantry Item belongs to exactly one Household per the dependency map; this automation only ever compares entries within the same household

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household member submits a new item on the pantry list | FEAT-05.SPEC-001 (Pantry List & Item Entry) | Fires after the name passes validation (FEAT-05.SPEC-004) | The validated item name, the submitting household, the submitting member |
| An offline-queued add syncs on reconnect | FEAT-05.SPEC-001 (Pantry List & Item Entry) | Fires when connectivity returns and a locally queued add is sent | The validated item name, the household, the member who queued it, the queued timestamp |
| "Already have it" tap logs an item from the grocery list | FEAT-05.SPEC-007 (Pantry Item Off-Grocery-List Exclusion Rule) | Fires when a grocery list line is marked "already have it" by a role with Pantry Input access | The grocery list line's ingredient name, the household, the tapping member |

## Processing Logic

1. Receive the validated item name and the household it belongs to, from whichever trigger fired.
2. Normalize the submitted name for comparison (case-insensitive, leading/trailing whitespace trimmed).
3. Steps 4-6 (read, compare, and create-or-merge) run as one serialized unit keyed on the household plus the normalized name: only one submission for that exact household-and-name pair may be inside this sequence at a time. A second submission for the same household and normalized name that arrives while the first is still inside the sequence waits for the first to finish (create or merge) before it begins its own read, so the two submissions are never comparing against the list at the same moment. Submissions for different names, or for the same name in different households, proceed independently and do not wait on each other.
4. Read every Active Pantry Item currently belonging to that household.
5. Compare the normalized submitted name against each existing Active item's name (also normalized) for an exact match.
6. If exactly one Active item matches, treat this submission as a duplicate of that entry: no new row is created. If no Active item matches, create a new Pantry Item with the submitted name, added_by the submitting member, and status Active.
7. In all cases, the resulting entry (matched or newly created) is what the triggering screen displays as the outcome. Because step 3 serializes the sequence per household and normalized name, this holds even when two submissions for the same name arrive at effectively the same time: the second always finds and merges into the first's result rather than creating a sibling row.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| New entry created | No existing Active item matches the submitted name | A new Pantry Item is created (Active) | The new row appears on the pantry list | FEAT-05.SPEC-001 |
| Merged into existing entry | An existing Active item matches the submitted name | No new row is created; the existing entry's presence on the list is unchanged | The existing row is shown (no visible duplicate); if the merge originated from an offline sync, the pending marker on the local queued item simply clears with no new row appearing | FEAT-05.SPEC-001 |
| Merged via inbound "already have it" | The submitted name (from FEAT-06) matches an existing Active item | No new row is created | The existing pantry row is unchanged; the grocery list line still leaves the list per FEAT-05.SPEC-007 | FEAT-05.SPEC-001, FEAT-05.SPEC-007 |
| Automation failure | The merge comparison cannot complete | No item is saved | The triggering screen's own Error state applies (FEAT-05.SPEC-001: typed item remains visible with a retry option) | FEAT-05.SPEC-001 |

## Data Model

**Reads:** Pantry Item -- item_name, status (Active only), scoped to the submitting household.
**Creates:** Pantry Item -- item_name, added_by, status: Active, when no match is found.
**Updates:** None -- a matched duplicate leaves the existing entry's fields unchanged; added_by remains the original logger, not the later submitter.
**Deletes:** None.

## Business Rules

- Matching is exact-name, case-insensitive, whitespace-trimmed -- there is no partial or fuzzy matching, consistent with item_name's free-text, no-rigid-schema definition (FEAT-05.SPEC-004).
- This automation is the single mechanism that prevents duplicate Active rows for the same name; the dependency map's Contention note for Pantry Item ("the same item added twice becomes one entry") is enforced entirely here, unconditionally -- including when two submissions for the same household and name arrive at effectively the same time, because the check-and-create sequence for a given household-and-normalized-name pair is serialized (Processing Logic, step 3): only one Active entry ever results, never two.
- The automation runs synchronously with the triggering add -- the pantry list screen waits for the merge-or-create result before showing the outcome row.
- A Used/Removed item is never matched against -- re-adding a previously cleared name always creates a fresh Active entry (per the Entity-Lifecycle Coverage Matrix), it does not revive the cleared one.

## Edge Cases

- **Submitted name differs only in capitalization or surrounding whitespace ("Spinach " vs. "spinach")** -- Matches the existing "spinach" entry; merged, no new row.
- **Submitted name differs in wording for the same ingredient ("spinach" vs. "baby spinach")** -- Does not match; a separate Active entry is created, per the Non-Goals (no fuzzy matching).
- **Two household members add the same name from different devices at effectively the same time** -- The check-and-create sequence for that household-and-name pair is serialized (Processing Logic, step 3): whichever submission enters the sequence first runs its read-compare-create and creates the entry; the other submission waits, then runs its own read-compare-create against the now-committed entry and merges into it. Only one Active entry ever results, regardless of which device's request physically arrived first.
- **Offline add later matches an item added by someone else while the device was offline** -- On reconnect, the queued add is compared against the current Active list (which now includes the other member's item); if it matches, the queued add merges and no duplicate appears, per the feature's own Offline-degraded expectation ("sync and reconcile once connectivity returns").
- **Inbound "already have it" tap names an ingredient that already exists as a Pantry Item** -- Merges per Outcome "Merged via inbound 'already have it'"; the grocery list line still leaves the list per FEAT-05.SPEC-007 regardless of merge outcome.
- **Concurrent trigger firing (two adds for different names in the same household fire at effectively the same time)** -- Each name is compared and resolved independently; there is no shared lock between different names.
- **Trigger fires while a previous run is in flight for the same name** -- A second submission of the same name while the first is still inside the serialized sequence (Processing Logic, step 3) does not start its own read until the first run has created or merged its entry. The second run then reads the updated Active list, finds the match, and merges -- no double-create is possible for the same household-and-normalized-name pair.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-001 (Pantry List & Item Entry) | Triggered by (inbound) | Fires on direct add and on offline-sync reconciliation |
| FEAT-05.SPEC-004 (Pantry Item Field Validation) | References (inbound) | The submitted name has already passed this spec's rules before the merge check runs |
| FEAT-05.SPEC-007 (Pantry Item Off-Grocery-List Exclusion Rule) | Triggered by (inbound) | Fires when an "already have it" tap on the grocery list logs an item to the pantry |
| FEAT-05.SPEC-001 (Pantry List & Item Entry) | Affects (outbound) | The merged-or-created row is what the pantry list displays |

## Analytics and Success Signals

- **pantry_item_merged** (merge source: direct add / offline sync / already-have-it) -- N/A -- this event has no directly connected metric in success-metrics.md; it is recorded to confirm the merge guarantee is exercised, feeding no metric this feature is measured by
- **pantry_item_added** (created: true) -- supports success-metrics.md: "Pantry Items Used" (a genuinely new entry contributes to the pool of items a plan can use up)
- **pantry_merge_failed** (trigger source) -- N/A -- a failed merge check falls back to the triggering screen's own error handling and is not itself a success signal for any metric in success-metrics.md

## Acceptance Criteria

**FEAT-05.SPEC-003-AC-01:** Given Maya's household has no Active pantry item named "spinach", when she submits "spinach" on the Pantry List screen, then a new Pantry Item "spinach" is created and shown.

**FEAT-05.SPEC-003-AC-02:** Given Maya's household already has an Active pantry item "spinach", when Sam submits "Spinach " (different capitalization, trailing space), then no new row is created and the existing "spinach" entry is what remains on the list.

**FEAT-05.SPEC-003-AC-03:** Given Maya's household has an Active pantry item "spinach", when Sam submits "baby spinach", then a separate new Pantry Item "baby spinach" is created (no fuzzy match).

**FEAT-05.SPEC-003-AC-04:** Given Sam added "eggs" while offline, when connectivity returns and no household member has added "eggs" in the meantime, then "eggs" syncs as a new Active entry.

**FEAT-05.SPEC-003-AC-05:** Given Sam added "eggs" while offline and Maya separately added "eggs" while he was offline, when Sam's device reconnects and syncs, then his queued "eggs" merges into Maya's existing entry and no duplicate row appears.

**FEAT-05.SPEC-003-AC-06:** Given Maya's household has an Active pantry item "yoghurt", when a grocery list line "yoghurt" is marked "already have it" (FEAT-05.SPEC-007), then no new pantry row is created and the existing "yoghurt" entry is unaffected, while the grocery list line still leaves the list.

**FEAT-05.SPEC-003-AC-07:** Given Maya previously cleared a pantry item named "flour", when she later re-adds "flour", then a fresh Active entry is created rather than any prior cleared record being restored.

**FEAT-05.SPEC-003-AC-08:** Given Maya and Sam each submit "onions" from different devices at effectively the same time, when both submissions are processed, then the check-and-create sequence for that household-and-name pair is serialized so only one Active "onions" entry ever exists: whichever submission enters the sequence second finds and merges into the entry the first one created, with no duplicate row appearing.

**FEAT-05.SPEC-003-AC-09:** Given the merge comparison cannot complete due to a processing error, when Maya submits a new item, then FEAT-05.SPEC-001's Error state applies: the typed item remains visible with a retry option and no item is saved.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



# Logic/Rule Spec: Pantry Item Field Validation

## Overview

**Name:** Pantry Item Field Validation
**ID:** FEAT-05.SPEC-004
**Type:** Logic/Rule
**Purpose:** Enforces the Pantry Item name's required, 1-80 character, free-text rule with no rigid inventory schema, and defines who may act on a Pantry Item and under what conditions.
**Parent Feature:** FEAT-05 -- Pantry-Aware Suggestions
**Governed Entity:** Pantry Item

## Scope and Non-Goals

**In Scope:**
- The item_name field's required, 1-80 character, free-text validation rule
- Authorization rules for every action on the Pantry Item, per role in the Access Matrix
- Default values applied when a Pantry Item is created
- The single source-of-truth rule referenced by FEAT-05.SPEC-001 and the inbound FEAT-06 create path (FEAT-05.SPEC-007)

**Non-Goals:**
- Duplicate-name detection and merge behavior -- handled by FEAT-05.SPEC-003 (Pantry Item Duplicate Merge), which runs after this spec's validation passes
- A fixed item-count limit per household -- excluded per the Feature Breakdown Brief's own Validation & Limits: "no fixed limit on how many items a household can log, though the feature is designed for a short, current list rather than a full inventory"
- Structured inventory fields such as quantity, unit, or expiry date -- excluded per scope-boundaries.md (SC-11): the product follows the lighter "tell me what I have" model, so item_name is the only captured field

## Governed Entity

**Entity:** Pantry Item
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| item_name | text | Free-text name of the item the household has on hand, 1-80 characters, required |
| added_by | text (derived reference) | The Member Profile who logged the item |
| status | enum | Active or Used/Removed |
| used_prompt | boolean/marker | Set by FEAT-05.SPEC-002 after the dinner using the item has passed; read by FEAT-05.SPEC-001 to render the "used it up?" prompt |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-05.SPEC-001 | Pantry List & Item Entry | On submit of the add-item field; authorization on screen entry (which roles see the field and clear/prompt actions at all) and on each action attempt |
| FEAT-05.SPEC-007 | Pantry Item Off-Grocery-List Exclusion Rule | On the inbound "already have it" create path from the Shared Grocery List, before FEAT-05.SPEC-003's merge check runs |
| FEAT-05.SPEC-003 | Pantry Item Duplicate Merge | Reads a name that has already passed this spec's validation; does not re-validate it |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| item_name | Required, non-empty after trimming whitespace | Always | On submit | "Enter what you have on hand." | Yes |
| item_name | Minimum 1 character, maximum 80 characters after trimming whitespace | Always | On submit | "This can be up to 80 characters." | Yes |
| item_name | Free text -- no character-set restriction beyond the length bound; no rigid inventory schema (no quantity, unit, or expiry sub-fields) | Always | On submit | N/A -- no rejection based on content, only length and emptiness | No |
| added_by | No validation beyond data type -- always set automatically to the acting member, never entered by the user | Always | -- | -- | -- |
| status | No validation beyond data type -- set by the system (Active on create, Used/Removed on clear), never entered directly by the user | Always | -- | -- | -- |
| used_prompt | No validation beyond data type -- set only by FEAT-05.SPEC-002, never entered by the user | Always | -- | -- | -- |

## Cross-Field Rules

N/A -- the Pantry Item has a single user-entered field (item_name); every other field is system-set (added_by, status) or automation-set (used_prompt), so no rule spans two user-entered fields.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View pantry list | Maya, Sam | Always | -- |
| View pantry list | Riley (Operator) | Only while an open Support Request exists for the household (FEAT-22) | Outside an open Support Request, the pantry screen is not reachable |
| View pantry list | Jordan (young kid profile, no login -- MVP) | Never | No account exists for this role; there is no screen to deny |
| View pantry list | Jordan (older kid, limited login -- Later) | Never | The "Pantry" entry is not shown in this role's navigation; a direct link redirects to the current plan with no error message |
| Add item | Maya, Sam | Always | -- |
| Add item | Riley (Operator), Jordan (either kid row) | Never | Add-item field is not rendered for these roles (Riley: read-only support view; both kid rows: no Pantry Input access) |
| Clear item (manual) | Maya, Sam | Always -- either household adult may clear any household item, not only the one they added (Pantry Input is Full for both, not ownership-scoped) | -- |
| Clear item (manual) | Riley (Operator), Jordan (either kid row) | Never | Clear action is not rendered for these roles |
| Answer "used it up?" prompt | Maya, Sam | Always, once the prompt is set by FEAT-05.SPEC-002 | -- |
| Answer "used it up?" prompt | Riley (Operator), Jordan (either kid row) | Never | Prompt chip is not rendered for these roles |
| Create pantry item via "already have it" (FEAT-06) | Maya, Sam | Always | -- |
| Create pantry item via "already have it" (FEAT-06) | Jordan (older kid, limited login -- Later) | Never -- this role's Grocery List access is Full for add/tick, but Pantry Input is None | Tapping "already have it" removes the line from the grocery list per FEAT-05.SPEC-007's own list behavior, but creates no Pantry Item for this role; no error is shown, since no pantry action was attempted from this role's perspective |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| added_by | The member submitting the add (direct add) or tapping "already have it" (inbound from FEAT-06) | On create only | No |
| status | Active | On create only | No -- only a subsequent clear action changes it, per FEAT-05.SPEC-001 |
| used_prompt | Unset | On create only | No -- only FEAT-05.SPEC-002 sets it later |

## Business Rules

- item_name is the only field a household member ever enters directly; every other field is system- or automation-derived, consistent with the feature's "no rigid inventory schema" definition.
- This spec is the single source of truth for the item_name rule (per the Feature Breakdown Brief's Shared Validation section); FEAT-05.SPEC-001 and the inbound FEAT-06 create path (FEAT-05.SPEC-007) both defer to it rather than re-deriving the character limit or required-field check.
- XBR-04: this spec governs the data created by the inbound "already have it" tap from FEAT-06, but the decision of whether that tap excludes the grocery line and whether it creates a pantry entry for a given role belongs to FEAT-05.SPEC-007; this spec supplies only the field-level and role-level rules that apply once creation is attempted.
- No item-count limit exists per household (Feature Breakdown Brief, Validation & Limits); this spec places no ceiling on the number of Active Pantry Items a household may hold.

## Edge Cases

- **item_name is exactly 80 characters** -- Passes validation. 81 characters shows the length error.
- **item_name is a single character** -- Passes validation (minimum is 1 character, not more).
- **item_name is only whitespace** -- Fails the required rule after trimming; treated as empty, shows "Enter what you have on hand."
- **item_name contains emoji or non-Latin characters** -- Passes validation; no character-set restriction is defined beyond length and emptiness.
- **Riley's Support Request closes while the read-only pantry view is open** -- The view access condition (open Support Request) is re-checked; once no open request remains, the view is no longer reachable on the next screen entry, consistent with FEAT-22's access model.
- **Jordan (older kid) taps "already have it" on a grocery list line** -- The line still leaves the grocery list (governed by FEAT-05.SPEC-007's own list behavior), but no Pantry Item is created for this role, per the Authorization Rules row above; this is not treated as a denied action requiring an error message, since the role never attempts a pantry-facing action directly.

## Acceptance Criteria

**FEAT-05.SPEC-004-AC-01:** Given Maya is adding a pantry item, when she submits an empty add-item field, then she sees "Enter what you have on hand." and no item is saved.

**FEAT-05.SPEC-004-AC-02:** Given Maya is adding a pantry item, when she submits a name that is only whitespace, then she sees "Enter what you have on hand." and no item is saved.

**FEAT-05.SPEC-004-AC-03:** Given Sam is adding a pantry item, when he submits a name of exactly 80 characters, then the item is saved successfully.

**FEAT-05.SPEC-004-AC-04:** Given Sam is adding a pantry item, when he submits a name of 81 characters, then he sees "This can be up to 80 characters." and no item is saved.

**FEAT-05.SPEC-004-AC-05:** Given Maya is adding a pantry item, when she submits a single-character name, then the item is saved successfully.

**FEAT-05.SPEC-004-AC-06:** Given Maya (Organiser) or Sam (Other Adult Member) is on the pantry list, when either looks for the add-item field and clear actions, then both are present and usable, since Pantry Input is Full for both roles.

**FEAT-05.SPEC-004-AC-07:** Given Riley (Operator) is viewing a household's pantry with no open Support Request, when Riley attempts to open the pantry screen, then it is not reachable.

**FEAT-05.SPEC-004-AC-08:** Given Riley (Operator) is viewing a household's pantry through an open Support Request, when Riley looks for the add-item field or clear actions, then neither is rendered, since Riley's Pantry Input access is View only.

**FEAT-05.SPEC-004-AC-09:** Given Jordan (older kid, limited login) is signed in, when this role looks for the "Pantry" entry in navigation, then it is not shown, since Pantry Input is None for this role.

**FEAT-05.SPEC-004-AC-10:** Given Jordan (older kid, limited login) taps "already have it" on a grocery list line, when the tap completes, then the line leaves the grocery list but no Pantry Item is created, since this role's Pantry Input access is None.

**FEAT-05.SPEC-004-AC-11:** Given a new Pantry Item is created by Sam, when the record is saved, then added_by is set to Sam automatically and status is set to Active, with no way for Sam to enter either value directly.

**FEAT-05.SPEC-004-AC-12:** Given Jordan (young kid profile, no login) has no account, when any pantry action is attempted on this role's behalf, then no such action exists -- the role has no sign-in through which to attempt it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 0 (N/A -- documented) | 0 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Pantry-Aware Plan Weighting Tier Gate

## Overview

**Name:** Pantry-Aware Plan Weighting Tier Gate
**ID:** FEAT-05.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs which pantry behaviors run on the free tier (logging, off-list exclusion) versus the paid tier (plan weighting toward logged items), reading the household's Subscription to decide.
**Parent Feature:** FEAT-05 -- Pantry-Aware Suggestions
**Governed Entity:** Subscription (read-only reference; the gating condition this spec applies to Pantry-Aware Suggestions' own behavior)

## Scope and Non-Goals

**In Scope:**
- The conditional rule that determines whether FEAT-05.SPEC-006 (Pantry-to-Recipe Matching) runs for a given weekly plan generation
- Confirming that pantry logging (FEAT-05.SPEC-001) and off-list exclusion (FEAT-05.SPEC-007) apply on both tiers, unaffected by this gate
- The behavior when a household downgrades or its payment lapses mid-cycle

**Non-Goals:**
- Validating or changing Subscription fields (tier, billing_period, billing_state, billing_history) -- owned entirely by FEAT-14 (Subscription & Billing Management); this spec only reads the tier value to decide a Pantry-Aware Suggestions behavior
- The matching logic itself (which Pantry Items a candidate recipe would use) -- handled by FEAT-05.SPEC-006, which this spec gates but does not perform
- Tier gating for other features that share this same rule shape (FEAT-03's AI generation eligibility, FEAT-12's rating-based learning) -- each of those features' own Logic/Rule specs applies this same Subscription read independently; this spec is the reference point for the pantry-specific weighting behavior only, per the Feature Breakdown Brief's Shared Validation section

## Governed Entity

**Entity:** Subscription
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| tier | enum (free, paid) | The value this spec reads to decide whether pantry-aware plan weighting runs; authored and updated by FEAT-14 |
| billing_state | enum (Active, Payment failed / grace period, Cancelled, Reverted to free) | Read alongside tier to determine the household's effective standing for this week's plan generation; authored and updated by FEAT-14 |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-03 AI Weekly Dinner Plan Generation (plan generation processing) | AI Weekly Dinner Plan Generation | Checked once per weekly plan generation, before candidate recipes are matched against Pantry Items |
| FEAT-05.SPEC-006 | Pantry-to-Recipe Matching for Plan Callout | Checked before the matching computation runs; the matching logic does not execute at all when this gate is closed |

## Field Validation Rules

N/A -- this spec reads the Subscription's tier and billing_state fields but authors neither; field-level validation for Subscription belongs to FEAT-14. Both fields are noted here as "no validation beyond data type" from this spec's perspective, since this spec only branches on their already-validated values.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| tier | No validation beyond data type -- read-only reference, owned by FEAT-14 | Always | On plan generation | -- | -- |
| billing_state | No validation beyond data type -- read-only reference, owned by FEAT-14 | Always | On plan generation | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Effective paid standing | tier, billing_state | Pantry-aware plan weighting runs only when tier is paid AND billing_state is Active or within the grace period (Payment failed, 7-day grace) -- a Cancelled or Reverted-to-free billing_state closes the gate even if tier still shows paid mid-transition | N/A -- this is a silent gating condition, not a user-facing validation error; no error message is shown, the plan simply generates without pantry weighting |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View a pantry-weighted callout on a Planned Meal | Maya, Sam | Only when this gate is open for the household (paid tier, Active or grace-period standing) | On a closed gate, no callout is shown on any meal, since none was computed; this is not a permission denial, it is the absence of a paid-tier feature the household has not unlocked |
| View a pantry-weighted callout on a Planned Meal | Jordan (older kid, limited login -- Later) | Same condition as above -- this role has View access to the Weekly Plan | Same as above |
| View a pantry-weighted callout on a Planned Meal | Riley (Operator) | Only while an open Support Request exists, and only if the household's gate is open | Same as above; Riley additionally never sees anything beyond what the household's own plan shows |
| Change the household's tier (open or close this gate) | Maya (Organiser) | Always, through FEAT-14 -- this spec does not itself perform the change, only reacts to it | Sam, both kid rows, and Riley cannot change billing (owned entirely by FEAT-14's own Authorization Rules); this spec has no independent control to deny, since it never exposes a tier-change action of its own |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Pantry-aware plan weighting eligibility (derived) | True when tier is paid and billing_state is Active or in the 7-day payment-failed grace period; false otherwise (free tier, Cancelled, or Reverted to free) | Re-evaluated every time a weekly plan generates | No -- this value is entirely derived from the Subscription's current standing, never set directly |

## Business Rules

- XBR-05: AI plan generation, pantry-weighted suggestions, and learning from ratings are paid; free and downgraded households plan through Manual Weekly Planning; pantry logging and the shared list stay free on both tiers; a downgrade or lapsed payment never removes any past plan, rating, recipe, or pantry item.
- Pantry logging (FEAT-05.SPEC-001) and off-list grocery exclusion (FEAT-05.SPEC-007) are never gated by this spec -- they run identically on both tiers, per the feature's own Access field: "Logging pantry items (and keeping them off the grocery list) works on both tiers."
- When the gate is closed for a given plan generation, the household still receives a complete seven-dinner plan (per FEAT-03's own Insufficient-data flow); the absence of pantry weighting is never a blocking condition on generation.
- A downgrade or lapsed payment mid-week does not retroactively remove a pantry callout already shown on a plan generated while the gate was open; it only affects the next generation.

## Edge Cases

- **Household's payment fails mid-week, entering the 7-day grace period** -- The gate remains open (billing_state is within grace) for any plan generation that occurs during the grace period; if the grace period expires before the next generation, the gate closes at that generation.
- **Household upgrades from free to paid mid-week, after that week's plan already generated without weighting** -- The current week's plan is not retroactively re-weighted; the gate opens starting with the next weekly generation.
- **Household's tier field briefly shows "paid" during a billing_state transition to Cancelled** -- The cross-field rule's AND condition closes the gate the moment billing_state is Cancelled, regardless of the tier field's transitional value, since both fields must agree for the gate to be open.
- **Free-tier household has logged pantry items and never upgrades** -- Those items remain fully usable for off-list exclusion (FEAT-05.SPEC-007) indefinitely; they simply never influence AI plan selection, since Manual Weekly Planning (FEAT-23) is what free-tier households use to build their week, and pantry-aware weighting is specific to FEAT-03's AI generation.
- **Household downgrades, then re-upgrades within the same billing period** -- The gate reflects whatever standing is current at the moment of each plan generation; no historical averaging or hysteresis applies.

## Acceptance Criteria

**FEAT-05.SPEC-005-AC-01:** Given Maya's household is on the paid tier with an Active billing_state, when the weekly plan generates, then the gate is open and FEAT-05.SPEC-006's matching logic runs.

**FEAT-05.SPEC-005-AC-02:** Given Maya's household is on the free tier, when the household plans its week (through FEAT-23, Manual Weekly Planning), then no pantry-weighted callout is computed, since the gate applies only to FEAT-03's AI generation and the free tier does not receive an AI-generated plan.

**FEAT-05.SPEC-005-AC-03:** Given Maya's household's payment fails and enters the 7-day grace period, when the weekly plan generates during that grace period, then the gate remains open and pantry weighting still applies.

**FEAT-05.SPEC-005-AC-04:** Given Maya's household's grace period has expired without payment being resolved, when the next weekly plan generates, then the gate is closed and no pantry-weighted callout is computed for that week.

**FEAT-05.SPEC-005-AC-05:** Given Maya's household cancels its subscription, when the current billing period ends and the next weekly plan would generate, then the household plans through Manual Weekly Planning (FEAT-23) instead, and no pantry weighting applies.

**FEAT-05.SPEC-005-AC-06:** Given Maya's household downgrades to free tier, when Maya opens the pantry list, then all previously logged Active pantry items are still present and can still be cleared or added to, since logging is never gated by this spec.

**FEAT-05.SPEC-005-AC-07:** Given a plan generated last week while the gate was open and carries a pantry callout, when Maya's household's payment fails this week, then last week's plan still shows its original pantry callout -- it is not retroactively removed.

**FEAT-05.SPEC-005-AC-08:** Given Jordan (older kid, limited login) is viewing the current week's plan on a paid, Active household, when a dinner carries a pantry callout, then Jordan can see it, consistent with this role's View access to the Weekly Plan.

**FEAT-05.SPEC-005-AC-09:** Given Riley (Operator) is viewing a household's plan through an open Support Request, when that household's gate is closed (free tier), then Riley sees no pantry callout on any meal, matching exactly what the household itself sees.

**FEAT-05.SPEC-005-AC-10:** Given Maya's household's tier field transitionally shows "paid" while billing_state has already moved to Cancelled, when the weekly plan generates at that exact moment, then the gate is treated as closed, since both fields must agree for pantry weighting to apply.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 (both N/A -- read-only) | 2 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Pantry-to-Recipe Matching for Plan Callout

## Overview

**Name:** Pantry-to-Recipe Matching for Plan Callout
**ID:** FEAT-05.SPEC-006
**Type:** Logic/Rule
**Purpose:** Defines how a logged Pantry Item is matched against a candidate recipe's ingredients to produce the plan's pantry callout, and how matches weight the AI plan's dinner selection on the paid tier.
**Parent Feature:** FEAT-05 -- Pantry-Aware Suggestions
**Governed Entity:** Planned Meal (specifically its pantry_callout field, derived by this spec; Pantry Item is read, not governed, by this spec)

## Scope and Non-Goals

**In Scope:**
- The matching rule that determines which Active Pantry Items a candidate recipe's ingredients would use
- How the count and nature of matches weight the AI plan's dinner selection, when FEAT-05.SPEC-005's gate is open
- Deriving the pantry_callout field on the chosen Planned Meal (e.g., "uses the spinach and feta you already have")

**Non-Goals:**
- Whether pantry weighting applies at all this week -- gated entirely by FEAT-05.SPEC-005 (Pantry-Aware Plan Weighting Tier Gate); this spec defines the matching mechanics only, and does not run when that gate is closed
- Recording that a dinner's night has passed and surfacing the "used it up?" prompt -- handled by FEAT-05.SPEC-002 (Used-It-Up Prompt Trigger), which reads this spec's pantry_callout output as its input
- Manual planning's use of pantry data -- Manual Weekly Planning (FEAT-23) lets a household pick recipes by hand without any pantry-weighted ranking; per the Feature Breakdown Brief's Key Capabilities, pantry awareness in plan selection is scoped to the AI-generated plan (FEAT-03) on the paid tier, not to manual picks on either tier

## Governed Entity

**Entity:** Planned Meal
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| night | enum (day of week) | Not governed by this spec -- set by FEAT-03/FEAT-23 |
| meal_kind | enum (dinner, leftover lunch) | Not governed by this spec |
| recipe | reference (Recipe) | The chosen Recipe; this spec reads its ingredients to compute matches, but does not set this field |
| safety_badge | text | Not governed by this spec -- set by FEAT-02 |
| vegetarian_option | boolean | Not governed by this spec |
| cook_time, rough_cost | derived | Not governed by this spec -- carried from the Recipe |
| pantry_callout | derived (list of Pantry Item names) | Which logged Active Pantry Items this dinner's recipe uses -- computed and set entirely by this spec |
| status | enum | Not governed by this spec -- set by FEAT-03/FEAT-23/FEAT-04/FEAT-02 |
| swap_history | list | Not governed by this spec |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-03 AI Weekly Dinner Plan Generation (plan generation processing) | AI Weekly Dinner Plan Generation | Runs once per candidate recipe during generation, after FEAT-05.SPEC-005's gate check confirms weighting applies, and after FEAT-02's safety check has already filtered candidates |
| FEAT-04 One-Tap Meal Swap (alternatives computation) | One-Tap Meal Swap | Runs on each safe alternative offered for a slot, so a swap alternative's pantry callout (if any) is shown consistently with original generation |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| pantry_callout | Derived list of Pantry Item names matched against the chosen recipe's ingredients (see Defaults and Derivations); no direct user entry, so no format validation applies | Only computed when FEAT-05.SPEC-005's gate is open for the household | On plan generation and on swap-alternative computation | N/A -- this is a derived field, not user input; there is no rejection or error state for it | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Callout depends on recipe choice | recipe, pantry_callout | pantry_callout is recomputed whenever the recipe field changes (initial pick, swap) -- it is never carried over from a previous recipe in the same slot | N/A -- silent recomputation, not a validation error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View a Planned Meal's pantry_callout | Maya, Sam | Only when FEAT-05.SPEC-005's gate is open (paid tier, Active/grace standing) and the callout is non-empty for that meal | On a closed gate or an empty match, no callout text is shown on the meal -- absence of a match, not a denial |
| View a Planned Meal's pantry_callout | Jordan (older kid, limited login -- Later) | Same condition as above, consistent with this role's View access to the Weekly Plan | Same as above |
| View a Planned Meal's pantry_callout | Riley (Operator) | Only while an open Support Request exists, and only when the household's own gate is open | Same as above |
| Trigger the matching computation | System only (FEAT-03 during generation, FEAT-04 during swap-alternative computation) | Always, subject to FEAT-05.SPEC-005's gate | N/A -- no user directly triggers this computation; it runs as part of plan generation or swap, which the organiser and other adult members already have access to per FEAT-03/FEAT-04's own Authorization models |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| pantry_callout | For a candidate recipe, compare each of its ingredients (by name) against the household's Active Pantry Items, using the same exact, case-insensitive, whitespace-trimmed name match defined in FEAT-05.SPEC-003; every Active Pantry Item whose name matches one of the recipe's ingredient names is included in the callout list. An empty result means no match -- the callout is simply absent from that meal, not shown as "uses nothing." | Computed once per candidate recipe during generation, and recomputed for the chosen recipe whenever it changes (initial selection or swap) | No -- entirely system-derived; no household member edits the callout text directly |
| Selection weighting | Among candidate recipes that already pass FEAT-02's safety check and FEAT-03's schedule/budget constraints, a recipe whose pantry_callout would be non-empty is weighted more favorably than one with no matches; a higher count of matched items increases the weighting further. Weighting is a preference among otherwise-eligible candidates -- it never overrides a hard dietary rule, schedule fit, or budget constraint, and it never causes an otherwise-ideal recipe to be excluded solely for having no pantry match. | Applied only during FEAT-03's AI generation, when FEAT-05.SPEC-005's gate is open | No -- the organiser cannot manually force pantry weighting; she may always swap to a different recipe afterward through FEAT-04 |

## Business Rules

- Matching uses the same exact-name comparison rule as FEAT-05.SPEC-003 (Pantry Item Duplicate Merge), so an item logged as "spinach" matches a recipe ingredient listed as "spinach" but not one listed as "baby spinach" -- consistent with the product's free-text, no-normalization model (scope-boundaries.md, SC-11).
- Pantry weighting is a preference signal only: it never causes a recipe that fails FEAT-02's safety check, the household's schedule constraint, or its budget to be selected, and it never excludes a safe, schedule-fitting, in-budget recipe for having zero pantry matches (per FEAT-03's own Insufficient-data flow: "pantry-awareness is a refinement, not a precondition for getting a plan").
- The matching computation reads only Active Pantry Items; an item already Used/Removed at generation time is never matched, consistent with FEAT-05.SPEC-002's model that a used item is cleared rather than retained for future matching.
- A swap alternative's pantry_callout is computed the same way as original generation, so a household member sees consistent pantry information whether reviewing the original plan or a swap option.

## Edge Cases

- **A candidate recipe's ingredient list is incomplete** -- Per FEAT-02's own fail-closed rule (XBR-01), a recipe with incomplete ingredient data is already excluded from candidacy before this spec ever computes a callout for it; this spec never runs its matching against incomplete ingredient data.
- **Two Active Pantry Items would both match the same single ingredient name (a duplicate that should not exist)** -- Cannot occur: FEAT-05.SPEC-003 guarantees at most one Active entry per exact name within a household, so at most one Pantry Item can match any given ingredient name.
- **A recipe matches five or more Active Pantry Items** -- All matched items are included in the callout text; the feature places no cap on how many logged items a single dinner's callout may name.
- **The gate closes between initial generation and a same-week swap** -- FEAT-05.SPEC-005 is re-checked at swap-alternative computation time (per its own Enforced By); if the gate has closed since generation (e.g., a grace period expired mid-week), swap alternatives are computed with no pantry callout, even though the original plan may still show one from when the gate was open.
- **A Pantry Item used in this week's callout is cleared by a household member before the dinner's night arrives** -- The Planned Meal's already-computed pantry_callout text is not retroactively edited; it continues to name the item until FEAT-05.SPEC-002's night-passage check would otherwise apply. This is accepted since the callout describes what informed the plan's construction, not a live inventory count.
- **Household has zero Active Pantry Items when generation runs** -- Every candidate recipe computes an empty callout; no weighting preference is applied, and generation proceeds exactly as it would with pantry-aware weighting entirely absent (per the feature's own "Nothing logged" flow).

## Acceptance Criteria

**FEAT-05.SPEC-006-AC-01:** Given Maya's household has Active pantry items "spinach" and "feta" and the gate (FEAT-05.SPEC-005) is open, when the weekly plan generates and a safe, schedule-fitting candidate recipe lists "spinach" and "feta" among its ingredients, then that recipe's Planned Meal shows a pantry_callout naming "spinach" and "feta".

**FEAT-05.SPEC-006-AC-02:** Given Maya's household has an Active pantry item "baby spinach" and a candidate recipe lists "spinach" as an ingredient, when generation runs, then the two do not match and "baby spinach" is not included in that recipe's callout.

**FEAT-05.SPEC-006-AC-03:** Given two safe, schedule-fitting, in-budget candidate recipes exist for a night, one matching two Active pantry items and one matching none, when generation selects between them, then the matching recipe is weighted more favorably, though the non-matching recipe remains eligible.

**FEAT-05.SPEC-006-AC-04:** Given a candidate recipe would match a household's Active pantry items but fails FEAT-02's safety check for that household, when generation runs, then the recipe is excluded regardless of any pantry match, since weighting never overrides a hard safety exclusion.

**FEAT-05.SPEC-006-AC-05:** Given a candidate recipe's ingredient data is incomplete, when generation runs, then the recipe is already excluded by FEAT-02 before this spec's matching ever considers it.

**FEAT-05.SPEC-006-AC-06:** Given Maya's household has zero Active pantry items, when the weekly plan generates, then every candidate recipe computes an empty pantry_callout and selection proceeds with no pantry weighting applied.

**FEAT-05.SPEC-006-AC-07:** Given Sam swaps Thursday's dinner (FEAT-04) while the gate is still open, when the safe alternatives are computed, then each alternative's pantry_callout is computed the same way as original generation.

**FEAT-05.SPEC-006-AC-08:** Given the household's gate closes (grace period expires) between Sunday's generation and a Wednesday swap, when Sam requests swap alternatives on Wednesday, then no pantry_callout is computed for any alternative, even though Sunday's original plan may still display a callout from when the gate was open.

**FEAT-05.SPEC-006-AC-09:** Given Maya's household logged "spinach" before Sunday's generation and a Thursday dinner's callout named it, when Maya clears "spinach" on Monday, then Thursday's already-computed pantry_callout still names "spinach" until the dinner's night passes and FEAT-05.SPEC-002 runs.

**FEAT-05.SPEC-006-AC-10:** Given Jordan (older kid, limited login) views the current week's plan on a paid, Active household, when a dinner carries a pantry_callout, then Jordan can see it, consistent with this role's View access to the Weekly Plan.

**FEAT-05.SPEC-006-AC-11:** Given a candidate recipe matches five distinct Active pantry items, when generation computes its callout, then all five matched item names are included, since no cap limits the callout's item count.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Pantry Item Off-Grocery-List Exclusion Rule

## Overview

**Name:** Pantry Item Off-Grocery-List Exclusion Rule
**ID:** FEAT-05.SPEC-007
**Type:** Logic/Rule
**Purpose:** Defines that an Active logged Pantry Item is left off the week's grocery list on both tiers, and that marking a grocery list line "already have it" can log it to the pantry in the same tap.
**Parent Feature:** FEAT-05 -- Pantry-Aware Suggestions
**Governed Entity:** Grocery List Item (the exclusion this spec governs) and Pantry Item (the inbound creation this spec authorizes)

## Scope and Non-Goals

**In Scope:**
- The rule that a plan-derived Grocery List Item is never generated (or is removed on recalculation) for an ingredient matching an Active Pantry Item, on both the free and paid tier
- The rule that tapping "already have it" on a Grocery List Item removes it from the list and, for roles with Pantry Input access, logs the ingredient to the pantry in the same tap
- Which roles' "already have it" tap results in a Pantry Item being created, per the Access Matrix

**Non-Goals:**
- Generating the grocery list itself from the week's plan, aisle grouping, and quantity combination -- owned by FEAT-06 (Shared Grocery List); this spec governs only the exclusion condition FEAT-06 applies during that generation
- The name-matching mechanics used to decide a match -- reuses the exact, case-insensitive, whitespace-trimmed comparison defined in FEAT-05.SPEC-003 (Pantry Item Duplicate Merge) rather than redefining it here
- Manually added Grocery List Items that a household member types directly (not derived from the plan) -- those are excluded from this rule only if their typed name happens to match an Active Pantry Item; a manual item with no matching pantry entry is unaffected by this spec and remains fully governed by FEAT-06's own rules

## Governed Entity

**Entity:** Grocery List Item
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| ingredient_name | text | Manual items 1-80 characters; plan-derived items take their name from the recipe ingredient. This spec compares this field against Active Pantry Item names to decide exclusion. |
| quantity_and_unit | text/derived | Not governed by this spec |
| aisle | text | Not governed by this spec |
| origin | enum (plan-derived, manual) | Not governed by this spec directly, but relevant: this spec's exclusion applies to plan-derived generation and recalculation; a manual item is only affected if its name happens to match an Active Pantry Item |
| ticked | boolean | Not governed by this spec |

**Entity:** Pantry Item (inbound creation only)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| item_name | text | Set from the Grocery List Item's ingredient_name when "already have it" creates a new pantry entry |
| added_by | text (derived reference) | Set to the member who tapped "already have it" |
| status | enum | Set to Active on creation |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06 Shared Grocery List (list generation/recalculation processing) | Shared Grocery List | Checked every time the grocery list generates or recalculates (per XBR-03), before a plan-derived line is added, on both tiers |
| FEAT-06 Shared Grocery List ("already have it" interaction) | Shared Grocery List | Checked at the moment a household member taps "already have it" on a line |
| FEAT-05.SPEC-001 (Pantry List & Item Entry) | Pantry List & Item Entry | Displays the Pantry Item created by an "already have it" tap, once FEAT-05.SPEC-003 has resolved create-vs-merge |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| ingredient_name (Grocery List Item) | Compared against every Active Pantry Item's item_name using the exact, case-insensitive, whitespace-trimmed match defined in FEAT-05.SPEC-003 | Checked on every plan-derived generation and recalculation, and is not a rejection rule -- a match causes exclusion, not an error | On list generation/recalculation | N/A -- exclusion is silent, not a validation failure shown to the user | No |
| item_name (Pantry Item, created via "already have it") | Same required, 1-80 character rule as FEAT-05.SPEC-004 -- the Grocery List Item's ingredient_name is always already within this bound (it shares the same 1-80 character rule), so this check never fails in practice for this inbound path | Always, when "already have it" creates a new entry | On tap | Uses FEAT-05.SPEC-004's exact error text in the (practically unreachable) case a name were somehow out of bounds | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Exclusion follows plan-derived lines and manual matches alike | ingredient_name, origin | Whether a Grocery List Item is plan-derived or manually added, if its ingredient_name matches an Active Pantry Item, the line is excluded from generation or removed on recalculation | N/A -- silent exclusion |
| "Already have it" removes and may create in one action | ingredient_name (Grocery List Item), item_name (Pantry Item) | Tapping "already have it" always removes the Grocery List Item from the list; whether it also creates or merges a Pantry Item depends on the tapping role's Pantry Input access (see Authorization Rules) | N/A -- no error message; the list-removal half of the action always succeeds regardless of the role |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Have an Active pantry item excluded from the grocery list | Maya, Sam (whoever logged it) | Always -- exclusion applies to the whole household's list regardless of who logged the item | -- |
| Tap "already have it" and remove the line from the grocery list | Maya, Sam | Always | -- |
| Tap "already have it" and remove the line from the grocery list | Jordan (older kid, limited login -- Later) | Always -- this role's Grocery List access is Full for add/tick, which the Feature Breakdown Brief's Access field extends to this interaction | -- |
| Tap "already have it" and also log the ingredient to the Pantry | Maya, Sam | Always -- Pantry Input is Full for both | -- |
| Tap "already have it" and also log the ingredient to the Pantry | Jordan (older kid, limited login -- Later) | Never -- this role's Pantry Input access is None, per the Access Matrix | No error message is shown; the tap still removes the grocery line as normal, but creates no Pantry Item, consistent with the dependency map's own note: "the older-kid login (Later) has Pantry Input None, so its 'already have it' on the list does not create a pantry item" |
| Tap "already have it" | Riley (Operator), Jordan (young kid profile, no login -- MVP) | Never -- Riley's Grocery List access is View only and this role has no login | The "already have it" control is not rendered for Riley's read-only support view; the young-kid row has no account through which to attempt any action |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Grocery List Item exclusion | A plan-derived Grocery List Item is never generated for an ingredient whose name matches an Active Pantry Item; on recalculation, a previously generated line that now matches a newly logged Active Pantry Item is removed | On every list generation and recalculation (per XBR-03) | No -- a household member cannot force an excluded ingredient back onto the plan-derived list; they may still add it as a separate manual item if they want it purchased anyway (a manual add is a distinct action from the excluded plan-derived line) |
| Pantry Item created via "already have it" | item_name set from the Grocery List Item's ingredient_name; added_by set to the tapping member; status set to Active -- then passed through FEAT-05.SPEC-003 for create-vs-merge resolution | On tap, only for a role with Pantry Input access | No |

## Business Rules

- XBR-04: Logged pantry items are left off the week's grocery list on both tiers, and marking a list item "already have it" can add it to the pantry in the same tap; only the paid tier weights plan selection toward pantry items (that weighting is governed separately by FEAT-05.SPEC-005 and FEAT-05.SPEC-006, not by this spec).
- Exclusion is tier-independent: it applies identically whether the week's plan came from FEAT-03 (AI generation, paid tier) or FEAT-23 (Manual Weekly Planning, either tier), since both feed the same Grocery List generation in FEAT-06.
- The "already have it" tap's list-removal effect and its pantry-creation effect are not one atomic guarantee for every role: the line always leaves the list, but the pantry side only happens for a role with Pantry Input access (see Authorization Rules) -- this is a deliberate asymmetry, not a defect, since the grocery list and pantry are governed by different access columns in the Access Matrix.
- A newly logged pantry item (direct add on FEAT-05.SPEC-001, not through "already have it") triggers exclusion or removal on the grocery list the next time it generates or recalculates -- the household does not need to separately mark the corresponding grocery line.

## Edge Cases

- **A household member logs a pantry item whose name matches an ingredient already ticked on the current grocery list** -- The already-ticked line is removed on the next recalculation regardless of its ticked state; a ticked item represents "already bought," and an Active pantry item represents "already have," so the line is excluded either way per XBR-03's rule that recalculation reflects the current plan and pantry state.
- **Jordan (older kid, limited login) taps "already have it" on a line** -- The line leaves the list per the Authorization Rules row above; no Pantry Item is created, and no error or explanation is shown to Jordan, since the grocery-list half of the action fully succeeds from this role's perspective.
- **A manually added Grocery List Item happens to share a name with an Active Pantry Item** -- The manual item is excluded/removed the same as a plan-derived one would be, since the Cross-Field Rule applies to ingredient_name regardless of origin.
- **The pantry item created via "already have it" matches an existing Active Pantry Item** -- FEAT-05.SPEC-003's merge rule applies exactly as it would for a direct add: no duplicate Pantry Item is created, and the grocery line still leaves the list.
- **A household clears a pantry item, then the grocery list recalculates before the next plan generation** -- The now-cleared item's name no longer matches any Active Pantry Item, so if that ingredient is still needed by the current plan, it reappears on the list at the next recalculation (per XBR-03: the list is always derived from the current plan and pantry state).
- **A free-tier household logs a pantry item after already building a manual week (FEAT-23)** -- The grocery list generated from that manual week still excludes the newly logged item on its next recalculation, since exclusion applies on both tiers independent of how the plan was built.

## Acceptance Criteria

**FEAT-05.SPEC-007-AC-01:** Given Maya's household has an Active pantry item "olive oil" and this week's plan calls for olive oil, when the grocery list generates, then no "olive oil" line appears on the list.

**FEAT-05.SPEC-007-AC-02:** Given Maya logs a new pantry item "rice" after the grocery list has already generated with a "rice" line on it, when the list next recalculates, then the "rice" line is removed.

**FEAT-05.SPEC-007-AC-03:** Given the "rice" line was already ticked before Maya logged "rice" to the pantry, when the list recalculates, then the ticked "rice" line is still removed, per XBR-03.

**FEAT-05.SPEC-007-AC-04:** Given Sam is viewing the grocery list and taps "already have it" on a "yoghurt" line, when the tap completes, then the "yoghurt" line leaves the list and a new (or merged) Active Pantry Item "yoghurt" is created with Sam as added_by.

**FEAT-05.SPEC-007-AC-05:** Given Jordan (older kid, limited login) is viewing the grocery list and taps "already have it" on a "bread" line, when the tap completes, then the "bread" line leaves the list but no Pantry Item is created, since this role's Pantry Input access is None.

**FEAT-05.SPEC-007-AC-06:** Given Maya's household already has an Active pantry item "eggs" and Sam taps "already have it" on an "eggs" grocery line, when the tap completes, then the line leaves the list and no duplicate Pantry Item is created (FEAT-05.SPEC-003's merge rule applies).

**FEAT-05.SPEC-007-AC-07:** Given a household member manually adds "paper towels" to the grocery list and the household separately has an Active pantry item "paper towels", when the list next recalculates, then the manually added "paper towels" line is removed, since exclusion applies regardless of the line's origin.

**FEAT-05.SPEC-007-AC-08:** Given a free-tier household built its week manually (FEAT-23) and the grocery list generated from it, when a household member logs "flour" to the pantry, then the "flour" line is removed from the list on the next recalculation, since exclusion applies on the free tier too.

**FEAT-05.SPEC-007-AC-09:** Given a household clears its Active pantry item "onions" and the current plan still calls for onions, when the grocery list recalculates, then the "onions" line reappears on the list.

**FEAT-05.SPEC-007-AC-10:** Given Riley (Operator) is viewing a household's grocery list through the read-only support view, when Riley looks for an "already have it" control, then it is not rendered, since Riley's Grocery List access is View only.

**FEAT-05.SPEC-007-AC-11:** Given Jordan (young kid profile, no login) has no account, when any "already have it" action is attempted on this role's behalf, then no such action exists, since the role has no sign-in.

**FEAT-05.SPEC-007-AC-12:** Given Maya's household's AI-generated plan (FEAT-03) and a separate household's manually built plan (FEAT-23) each call for "garlic" and each household has an Active pantry item "garlic", when each household's grocery list generates, then both lists exclude "garlic", confirming the rule applies identically regardless of plan origin.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
