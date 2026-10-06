---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-05.SPEC-007
spec_name: Pantry Item Off-Grocery-List Exclusion Rule
spec_slug: pantry-item-off-grocery-list-exclusion-rule
parent_feature: FEAT-05
parent_feature_name: Pantry-Aware Suggestions
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 8
acceptance_criteria_count: 12
---

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
