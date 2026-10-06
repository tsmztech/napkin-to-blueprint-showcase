---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-01.SPEC-015
spec_name: Dietary Rule Classification & Allergen Matching Rules
spec_slug: dietary-rule-classification-allergen-matching-rules
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 13
acceptance_criteria_count: 13
---

# Logic/Rule Spec: Dietary Rule Classification & Allergen Matching Rules

## Overview

**Name:** Dietary Rule Classification & Allergen Matching Rules
**ID:** FEAT-01.SPEC-015
**Type:** Logic/Rule
**Purpose:** Governs allergen selection, hard-vs-soft strength classification, per-person vegetarian logic, and the explicit-confirmation gate before an allergy or religious rule is removed.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles
**Governed Entity:** Dietary Rule

## Scope and Non-Goals

**In Scope:**
- Allergen selection from the standard allergen list, plus the optional named-extra-ingredient field
- Strength classification: which rule_kind values are hard vs. soft
- Per-person vegetarian setting logic, including the shared-meal vegetarian-variant implication
- The explicit confirmation gate that must complete before an allergy or religious rule is removed
- Authorization for who can create, edit, and remove Dietary Rule entries

**Non-Goals:**
- Checking a Dietary Rule against any recipe's ingredients -- owned entirely by the Dietary Rules & Allergy Safety Engine (FEAT-02), which reads this data but performs no classification of its own
- The screen mechanics of entering or displaying rules -- owned by FEAT-01.SPEC-006 (Dietary Rules Editor), which enforces the rules defined here
- Medical or diet advice derived from a rule -- excluded per scope-boundaries.md SC-06; this spec classifies rules for matching purposes only, never as nutritional guidance

## Governed Entity

**Entity:** Dietary Rule
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| member | reference | The Member Profile the rule belongs to (required) |
| rule_kind | enum | Allergy, religious rule (e.g., halal), per-person vegetarian setting, or dislike |
| strength | enum | Hard (allergy, religious rule) or soft (dislike); vegetarian applies per person |
| allergen | reference/text | From a standard allergen list, optionally a named extra ingredient (required for allergies) |
| origin | enum | Entered by the organiser or learned from ratings (FEAT-12) |
| change_history | derived | Who changed the rule and when, visible to the organiser |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-006 | Dietary Rules Editor | On save of each rule entry (create/edit); on "Remove" tap for every rule_kind |
| FEAT-01.SPEC-012 | Mid-Week Hard-Rule Change Trigger | Reads this spec's hard/tightened classification to decide whether to fire |
| FEAT-02 | Dietary Rules & Allergy Safety Engine (cross-feature) | Reads strength and allergen classification when checking recipes; does not enforce this spec's field rules itself |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| member | Required, must reference an existing Active Member Profile in the household | Always | On save | N/A -- the field is set by the screen context (which member's editor is open), never entered directly; no error state is user-reachable | Yes |
| rule_kind | Required, one of: allergy, religious rule, vegetarian setting, dislike | Always | On save | "Choose what kind of rule this is" | Yes |
| allergen | Required, selected from the standard allergen list | rule_kind is allergy | On save | "Select an allergen from the list" | Yes |
| allergen (named extra ingredient) | Optional free-text, up to 80 characters | rule_kind is allergy | On save | "Ingredient name must be 80 characters or fewer" | Yes |
| allergen | No validation beyond data type -- not applicable | rule_kind is religious rule, vegetarian setting, or dislike | -- | -- | -- |
| Religious rule label | Required free-text, up to 60 characters | rule_kind is religious rule | On save | "Enter the religious rule (up to 60 characters)" | Yes |
| Dislike label | Required free-text, up to 60 characters | rule_kind is dislike | On save | "Enter the dislike (up to 60 characters)" | Yes |
| strength | Not user-entered -- derived automatically from rule_kind (see Defaults and Derivations) | Always | On save | N/A -- no error state, since the field is never directly editable | Yes (derived) |
| origin | Not user-entered on this screen -- set automatically to "entered by the organiser" for every rule created via FEAT-01.SPEC-006 | Always | On save | N/A -- FEAT-12 sets "learned from ratings" independently for its own creations | Yes (derived) |
| change_history | No validation beyond data type -- append-only, system-maintained | Always | On every create/edit/remove | N/A | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Strength follows rule_kind | rule_kind, strength | Allergy and religious rule always classify as hard; dislike always classifies as soft; vegetarian setting applies per person and is treated as hard for that person's own meal selection (a vegetarian member is never served a non-vegetarian dish), while never blocking the household's shared meal from existing, since a vegetarian-variant option can satisfy it | N/A -- strength is derived, not entered, so no invalid combination is directly reachable |
| Allergen required only for allergies | rule_kind, allergen | If rule_kind is allergy, allergen is required; for any other rule_kind, the allergen field is not shown at all | "Select an allergen from the list" (allergy only) |
| Duplicate allergen for the same member | member, rule_kind, allergen | Attempting to add a second allergy entry for an allergen the member already has routes to editing the existing entry rather than creating a duplicate | N/A -- handled by routing to edit, not by an error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create dietary rule | Maya (Organiser) | Any member of the household | -- |
| Create dietary rule (learned dislike) | System, via FEAT-12 | Origin set to "learned from ratings"; never created directly by Sam or any adult through this screen | N/A -- not a user-facing action on this spec's enforcing screen |
| View dietary rules | Maya (Organiser), Sam (Other Adult Member) | Always, for every member | -- |
| View dietary rules (allergy detail only) | Riley (Operator, support) | Only inside a specific open safety report (XBR-14), never the full list | Riley never reaches FEAT-01.SPEC-006 directly; broader dietary-rule visibility is never granted to the operator role |
| Edit dietary rule | Maya (Organiser) | Any member's rule | -- |
| Edit dietary rule | Sam (Other Adult Member) | Never | Rules render read-only; a direct interaction attempt shows "Only the organiser can change dietary rules" |
| Remove dislike | Maya (Organiser) | Any member's dislike, immediately, no confirmation gate | -- |
| Remove allergy or religious rule | Maya (Organiser) | Only after completing the explicit confirmation gate (see Business Rules) | Attempting removal without confirming shows the confirmation modal; the rule is not removed until the modal's explicit affirmative action is taken |
| Remove allergy or religious rule while offline | Maya (Organiser) | Never -- the confirmation gate requires a live check against the current record | "Removing an allergy needs a connection -- try again once you're back online." |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| strength | Derived from rule_kind: allergy -> hard; religious rule -> hard; vegetarian setting -> hard (for that member's own selection); dislike -> soft | On every create/edit | No -- never directly editable |
| origin | "Entered by the organiser" | On creation via FEAT-01.SPEC-006 | No |
| change_history | A new entry is appended: who made the change, what changed, and when | On every create, edit, and remove | No -- always system-maintained, never editable or deletable by any role |

## Business Rules

- An allergy or religious rule can never be removed without the organiser completing an explicit confirmation step first: the confirmation modal states exactly what will happen ("Removing this will let recipes containing {allergen/rule} be suggested again for {member}") and requires an explicit affirmative tap, with no default-confirmed state, per this feature's Shared UI Patterns.
- A dislike can be removed directly with no confirmation step, since it is soft and removing it carries no safety consequence.
- A learned soft dislike (origin: learned from ratings, created by FEAT-12) never overwrites or weakens an explicit organiser-entered rule for the same ingredient; an explicit organiser edit always takes precedence over a learned entry for the same ingredient, per the dependency map's Contention note for Dietary Rule.
- A new or tightened hard rule (allergy or religious rule) triggers FEAT-01.SPEC-012 (Mid-Week Hard-Rule Change Trigger) -- this spec defines what qualifies as "tightened" (a widened allergen scope or an added named ingredient), while FEAT-01.SPEC-012 owns the resulting re-check.
- A rule's change_history entry survives the rule's own removal indefinitely, per this feature's Data Notes: "Each dietary rule's change history is kept for trust and any safety investigation," even though the rule itself is gone with no restore path.
- Vegetarian setting is per-person, not household-wide: a shared meal in a mixed-diet household can carry a vegetarian-variant option to satisfy a vegetarian member without requiring every other member to eat vegetarian.

## Edge Cases

- **Organiser selects an allergen already on the member's list and attempts to add it again** -- The add-rule panel routes to editing the existing entry instead of creating a duplicate Dietary Rule for the same allergen.
- **Organiser names an extra ingredient at exactly 80 characters** -- Passes validation; 81 characters shows the length error.
- **Organiser attempts to remove an allergy while offline** -- Blocked with the connectivity message; the confirmation modal never opens, since the gate requires a live check.
- **A learned soft dislike (FEAT-12) targets the same ingredient as an organiser-entered explicit rule** -- The explicit rule's strength and presence are unaffected; the learned entry exists as a separate soft record that influences selection weighting only, never overriding the explicit rule.
- **Organiser attempts to remove an allergy, confirms the modal, but connectivity is lost between confirmation and completion** -- The removal does not complete; the rule remains present, and the organiser sees a retry option, consistent with a failed-save-preserves-state posture.
- **Two hard rules for the same member are added in immediate succession (e.g., an allergy and a religious rule)** -- Each triggers its own independent evaluation by FEAT-01.SPEC-012; both re-checks may run, and a meal failing either is removed once, not twice, since FEAT-02 evaluates the full current rule set on each re-check rather than one changed rule in isolation.
- **Vegetarian setting toggled on for a member in a household with no other vegetarian members** -- The rule is recorded exactly the same as any other member's vegetarian setting; no household-level change is required, since the setting is inherently per-person.

## Acceptance Criteria

**FEAT-01.SPEC-015-AC-01:** Given Maya selects "Peanuts" from the standard allergen list for Jordan and saves, then a Dietary Rule is created with rule_kind=allergy, allergen=Peanuts, strength=hard.

**FEAT-01.SPEC-015-AC-02:** Given Maya adds a dislike for a member and saves, then the rule is created with strength=soft.

**FEAT-01.SPEC-015-AC-03:** Given Maya toggles the vegetarian setting on for a member and saves, then the rule is recorded with strength=hard for that member's own meal selection, without requiring the household's shared meal to be vegetarian.

**FEAT-01.SPEC-015-AC-04:** Given Maya attempts to save an allergy with no allergen selected, when she taps Save, then the error "Select an allergen from the list" appears and no rule is created.

**FEAT-01.SPEC-015-AC-05:** Given Maya taps "Remove" on a dislike, then it is removed immediately with no confirmation step.

**FEAT-01.SPEC-015-AC-06:** Given Maya taps "Remove" on an existing allergy, then a confirmation modal appears stating what will happen, and the allergy is not removed until she taps the explicit affirmative action.

**FEAT-01.SPEC-015-AC-07:** Given Maya is offline and taps "Remove" on a religious rule, then she sees "Removing an allergy needs a connection -- try again once you're back online." and the confirmation modal does not open.

**FEAT-01.SPEC-015-AC-08:** Given a learned soft dislike from FEAT-12 targets the same ingredient as Maya's existing explicit allergy, when both exist, then the explicit allergy's strength and presence are unaffected by the learned entry.

**FEAT-01.SPEC-015-AC-09:** Given Maya attempts to add a second allergy entry for peanuts when one already exists for that member, then she is routed to edit the existing entry rather than a duplicate being created.

**FEAT-01.SPEC-015-AC-10:** Given Maya names a specific extra ingredient at exactly 80 characters, when she saves, then the rule is created successfully.

**FEAT-01.SPEC-015-AC-11:** Given Maya names a specific extra ingredient at 81 characters, when she saves, then the error "Ingredient name must be 80 characters or fewer" appears.

**FEAT-01.SPEC-015-AC-12:** Given Sam views any member's dietary rules, when he attempts to interact with a rule, then no edit or remove control is reachable and he sees "Only the organiser can change dietary rules" if he tries a direct action.

**FEAT-01.SPEC-015-AC-13:** Given Maya confirms the removal modal for an allergy but connectivity is lost before the removal completes, then the allergy remains present and Maya is offered a retry once connectivity returns.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 10 | 10 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |
