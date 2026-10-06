---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-06.SPEC-007
spec_name: Manual Item Validation & Duplicate Merge
spec_slug: manual-item-validation-duplicate-merge
parent_feature: FEAT-06
parent_feature_name: Shared Grocery List
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 9
acceptance_criteria_count: 11
---

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
