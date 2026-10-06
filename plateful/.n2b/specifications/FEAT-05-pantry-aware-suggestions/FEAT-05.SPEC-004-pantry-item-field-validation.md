---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-05.SPEC-004
spec_name: Pantry Item Field Validation
spec_slug: pantry-item-field-validation
parent_feature: FEAT-05
parent_feature_name: Pantry-Aware Suggestions
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 6
acceptance_criteria_count: 12
---

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
