---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-13.SPEC-006
spec_name: Prep-Reminder Derivation Rule
spec_slug: prep-reminder-derivation-rule
parent_feature: FEAT-13
parent_feature_name: Tonight's Dinner Reminder
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 8
acceptance_criteria_count: 9
---

# Logic/Rule Spec: Prep-Reminder Derivation Rule

## Overview

**Name:** Prep-Reminder Derivation Rule
**ID:** FEAT-13.SPEC-006
**Type:** Logic/Rule
**Purpose:** Derives the prep-reminder text (or its confirmed absence) for tonight's dinner from the Planned Meal's recipe's early-prep requirements, so the nudge and its correction never invent a prep step that isn't there.
**Parent Feature:** FEAT-13 -- Tonight's Dinner Reminder
**Governed Entity:** Recipe (prep_requirements field), read through tonight's Planned Meal's recipe reference

## Scope and Non-Goals

**In Scope:**
- Reading a recipe's prep_requirements field and rendering it as the prep-reminder text
- The no-invented-prep-step disposition when prep_requirements is empty
- Who may view the derived text (via the nudge or the "Tonight" card) and who may edit its source field

**Non-Goals:**
- Editing a recipe's prep_requirements field -- owned by Recipe Library (Starter Recipes) (FEAT-08, content seeding) and Recipe Import from Web Link (FEAT-10, household edits to imported recipes); this spec only reads the field's current value at dispatch time.
- Deciding whether a nudge or correction fires at all, and who receives it -- owned by FEAT-13.SPEC-001 (Tonight's Nudge Trigger), FEAT-13.SPEC-003 (Same-Day Swap Correction Trigger), and FEAT-13.SPEC-005 (Nudge Delivery & Eligibility Rules); this spec only supplies content, never audience.
- The exact message wording that wraps this derived text -- owned by FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message) and FEAT-13.SPEC-004 (Same-Day Swap Correction Message); this spec produces the {prep_reminder_text} placeholder value, not the surrounding sentence.

## Governed Entity

**Entity:** Recipe (prep_requirements field), read via tonight's Planned Meal.recipe reference
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| prep_requirements | text | Early-prep needs such as defrosting, entered on the recipe (starter library content or a household's imported recipe); used for the nightly nudge |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-13.SPEC-001 | Tonight's Nudge Trigger | Reads this derivation once per dispatch, to build tonight's nudge content |
| FEAT-13.SPEC-003 | Same-Day Swap Correction Trigger | Reads this derivation once per dispatched correction, for the new recipe |
| FEAT-13.SPEC-002 | Tonight's Dinner Nudge Message | Renders the derived text (or its absence) into the nudge's content |
| FEAT-13.SPEC-004 | Same-Day Swap Correction Message | Renders the derived text (or its absence) into the correction's content |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| prep_requirements | No validation beyond data type -- editorial content owned and validated by FEAT-08 (starter content) and FEAT-10 (imported-recipe edits, including the safety re-check FEAT-10 already requires); this spec applies no additional rule beyond reading its current value | Always | At each nudge or correction dispatch (read-only) | N/A -- this spec never rejects or flags a recipe's prep_requirements content; it only reads whatever value already exists | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Prep-reminder text derivation | Planned Meal.recipe, Recipe.prep_requirements | Resolve tonight's Planned Meal's recipe reference, then read that recipe's prep_requirements. If non-empty (after trimming leading/trailing whitespace), render it verbatim as the prep-reminder text. If empty or whitespace-only, the derivation yields no prep step at all -- never an invented placeholder | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the derived prep-reminder text (via the nudge, its correction, or the "Tonight" card) | Maya, Sam | Only when they are eligible recipients per FEAT-13.SPEC-005 | The member simply receives no prep-reminder content, along with no nudge or correction at all |
| View the derived prep-reminder text | Jordan (both rows), Riley | Never | Neither receives any surface this feature produces (FEAT-13.SPEC-005's Authorization Rules) |
| Edit a recipe's prep_requirements (source field) | Maya, Sam (Recipe Library: Full) | Only for imported recipes, through FEAT-10; starter-library recipes are read-only for households | Attempting to edit a starter recipe's prep_requirements has no control to act on -- starter content is read-only per the Feature Dependency Map's Recipe Contention note |
| Edit a recipe's prep_requirements | Jordan (both rows) | Never | No Recipe Library edit access exists for either kid row (Access Matrix: Recipe Library None or View) |
| Edit a recipe's prep_requirements | Riley (Operator, support) | Never | Riley's Recipe Library access is View only (Access Matrix) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| prep_reminder_text (derived, not stored) | If tonight's Planned Meal's recipe's prep_requirements is non-empty (after trimming), render it verbatim; otherwise the derivation yields no prep step and no text is invented | Computed fresh at each nudge or correction dispatch (FEAT-13.SPEC-001, FEAT-13.SPEC-003) | No -- this value is never directly editable; changing the outcome requires editing the recipe's prep_requirements itself, through FEAT-10 |

## Business Rules

- No prep step is ever invented: when prep_requirements is empty, the nudge and its correction name the meal only (product-features.md, States: "No prep needed" alternate).
- The derivation is read-only and non-destructive: it never writes back to Recipe or Planned Meal.
- prep_requirements editing follows FEAT-10's edit path for imported recipes and FEAT-08's content-seeding path for starter recipes, both external to this feature; this spec only consumes the field's current value at dispatch time.
- A mid-day edit to prep_requirements alone (with no accompanying swap) is not itself a trigger for any correction -- only a swap that changes the Planned Meal's recipe (FEAT-04.SPEC-004) triggers FEAT-13.SPEC-003; an edited prep step on the same, unswapped recipe stands as already dispatched for that day.

## Edge Cases

- **prep_requirements is edited on the recipe after tonight's nudge already dispatched, with no accompanying swap** -- No correction is issued for a same-recipe content edit alone; only a swap that changes the Planned Meal's recipe (FEAT-04.SPEC-004) triggers FEAT-13.SPEC-003. The originally dispatched prep text stands for that night.
- **prep_requirements is whitespace-only** -- Treated identically to empty: no prep step is rendered, and the derivation fails safe to the "no prep needed" disposition rather than showing blank or malformed text.
- **prep_requirements is present but exceptionally long** -- Rendered verbatim as authored; this spec applies no additional length rule of its own, since content length is governed by FEAT-08's and FEAT-10's own recipe-content rules, not by this derivation.
- **Tonight's Planned Meal's recipe reference no longer resolves (the recipe itself was removed)** -- Out of scope for this spec: FEAT-13.SPEC-001's own "no dinner planned tonight" disposition already covers a Planned Meal that cannot produce a valid nudge, since a Planned Meal's recipe field is required and its removal path (FEAT-02, FEAT-10) also removes or replaces the Planned Meal itself.

## Acceptance Criteria

**FEAT-13.SPEC-006-AC-01:** Given tonight's Planned Meal's recipe has prep_requirements "take the chicken out of the freezer," when this derivation runs, then the prep-reminder text renders as "take the chicken out of the freezer" verbatim.

**FEAT-13.SPEC-006-AC-02:** Given tonight's Planned Meal's recipe has no prep_requirements, when this derivation runs, then no prep-reminder text is produced and none is invented.

**FEAT-13.SPEC-006-AC-03:** Given tonight's Planned Meal's recipe's prep_requirements is whitespace-only, when this derivation runs, then it is treated as empty and no prep step is rendered.

**FEAT-13.SPEC-006-AC-04:** Given Maya is an eligible recipient of tonight's nudge, when she views it, then she sees whatever prep-reminder text (or its absence) this derivation produced for tonight's recipe.

**FEAT-13.SPEC-006-AC-05:** Given Jordan (young-kid profile) has no notification access at all, when the derivation's output is dispatched, then Jordan never sees it on any surface.

**FEAT-13.SPEC-006-AC-06:** Given Sam attempts to edit an imported recipe's prep_requirements through FEAT-10, when he saves the change, then the edit is accepted, since Sam has Recipe Library Full access.

**FEAT-13.SPEC-006-AC-07:** Given Maya attempts to edit a starter-library recipe's prep_requirements, when she looks for an edit control, then none exists, since starter content is read-only.

**FEAT-13.SPEC-006-AC-08:** Given a recipe's prep_requirements is edited the same day after tonight's nudge already dispatched, with no swap occurring, when the edit is saved, then no correction is issued and tonight's already-dispatched prep text stands.

**FEAT-13.SPEC-006-AC-09:** Given a same-day swap changes tonight's Planned Meal to a new recipe, when FEAT-13.SPEC-003 reads this derivation for the new recipe, then the correction's prep-reminder text reflects the new recipe's prep_requirements, independent of the original recipe's value.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |
