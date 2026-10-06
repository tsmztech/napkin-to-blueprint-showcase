---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-02.SPEC-006
spec_name: Rule Strength & Blocking Policy
spec_slug: rule-strength-and-blocking-policy
parent_feature: FEAT-02
parent_feature_name: Dietary Rules & Allergy Safety Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 12
acceptance_criteria_count: 13
---

# Logic/Rule Spec: Rule Strength & Blocking Policy

## Overview

**Name:** Rule Strength & Blocking Policy
**ID:** FEAT-02.SPEC-006
**Type:** Logic/Rule
**Purpose:** Defines which rule kinds are hard filters (allergy, religious rule, per-person/shared vegetarian) versus soft, non-blocking preferences (dislikes).
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine
**Governed Entity:** Dietary Rule

## Scope and Non-Goals

**In Scope:**
- Classifying every rule_kind value (allergy, religious rule, per-person vegetarian, dislike) as hard or soft
- Defining how a shared-meal vegetarian option satisfies a per-person vegetarian setting
- Authorization for reading and applying rule-strength classifications across the roles that interact with this engine

**Non-Goals:**
- Creating, editing, or deleting Dietary Rule records -- owned by FEAT-01 (Household Setup & Member Profiles); this spec only classifies rule strength, it does not manage the rule lifecycle
- Running the ingredient-versus-rule comparison itself -- owned by FEAT-02.SPEC-002 (Candidate Safety Check Execution), which reads this spec's classification
- Governing what happens when ingredient data is incomplete -- owned by FEAT-02.SPEC-007 (Ingredient Data Completeness & Fail-Closed Policy)
- Learning new soft dislikes from repeated down-ratings -- owned by FEAT-12 (Meal Rating & Preference Learning), which writes new soft Dietary Rule entries this spec then classifies as soft like any other dislike

## Governed Entity

**Entity:** Dietary Rule
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| member | reference | The Member Profile the rule belongs to |
| rule_kind | enum | allergy, religious rule, per-person vegetarian, or dislike |
| strength | enum | hard (allergy, religious rule) or soft (dislike); vegetarian applies per person with a shared-meal option |
| allergen | text | From the standard allergen list, optionally a named extra ingredient (required for allergies) |
| origin | enum | Entered by the organiser or learned from ratings |
| change_history | derived | Who changed the rule and when |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-02.SPEC-002 | Candidate Safety Check Execution | During ingredient comparison, to determine which rules can exclude a recipe |
| FEAT-02.SPEC-003 | Mid-Week Rule Change Re-Check | To determine whether a changed rule is hard (triggers re-check) or soft (does not) |
| FEAT-01 | Household Setup & Member Profiles (Dietary Rule creation/edit) | On save, to require strength = hard for allergy and religious rule kinds and strength = soft for dislike kind |
| FEAT-03 | AI Weekly Dinner Plan Generation (selection weighting) | To apply soft dislikes as ranking influence rather than exclusion |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| rule_kind | Must be one of: allergy, religious rule, per-person vegetarian, dislike | Always | On save (FEAT-01) | "Choose a rule type." | Yes |
| strength | Must be hard when rule_kind is allergy or religious rule; must be soft when rule_kind is dislike; per-person vegetarian carries no independent strength value (see Cross-Field Rules) | Conditional on rule_kind | On save (FEAT-01) | "Allergies and religious rules are always treated as hard, non-negotiable rules." | Yes |
| allergen | No validation beyond data type in this spec (required-for-allergies validation is owned by FEAT-01) | Always | -- | -- | -- |
| member | No validation beyond data type in this spec -- the referenced Member Profile's existence and validity is owned by FEAT-01 (Household Setup & Member Profiles), which this spec only reads to classify the rule | Always | -- | -- | -- |
| origin | No validation beyond data type in this spec -- whether a rule is organiser-entered or learned from ratings has no bearing on its strength classification; both origins are classified identically by rule_kind | Always | -- | -- | -- |
| change_history | No validation beyond data type in this spec -- this is a derived, system-written audit trail with no user-facing validation; ownership of what it records sits with FEAT-01 | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Strength follows rule_kind | rule_kind, strength | strength is derived from rule_kind, not independently settable: allergy and religious rule are always hard; dislike is always soft; per-person vegetarian is always treated as hard for the purpose of this spec's blocking behavior, with the shared-meal vegetarian_option variant as its satisfaction mechanism (see Business Rules) | N/A -- this is a derived value, not a user-facing validation failure |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Read rule-strength classification (used internally by the safety check) | All roles that can view a Planned Meal or Recipe (Maya, Sam, older-kid limited login) | Always -- classification is not itself sensitive data, only its consequence (the badge or exclusion reason) is user-visible | -- |
| Apply hard-rule exclusion during a candidate check | System (via FEAT-02.SPEC-002) | Always, on every candidate | -- |
| Override or bypass a hard-rule exclusion | No role, ever | Never -- allergies and religious rules cannot be overridden by any user action, per the Brief's Constraints: Safety | The recipe is excluded from the candidate pool entirely; there is no "show anyway" or override control on any screen |
| Change a rule's strength classification directly | No role -- strength is derived from rule_kind, not independently editable | Never | The Household Setup rule-entry form (FEAT-01) exposes no strength control; strength is set automatically from the chosen rule_kind |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| strength | allergy -> hard; religious rule -> hard; per-person vegetarian -> hard (with shared-meal variant satisfaction); dislike -> soft | On create and on any edit of rule_kind | No -- derived automatically from rule_kind, never independently set |

## Business Rules

- Allergies and religious rules are hard filters: a candidate recipe that fails either for any household member is excluded from the household's candidate pool entirely, per FEAT-02.SPEC-002.
- Per-person vegetarian settings apply per household member and are treated as hard for that member, but a shared dinner recipe is not excluded on vegetarian grounds alone if it carries a vegetarian_option variant that satisfies the vegetarian member -- this is the shared-meal vegetarian accommodation the Brief names explicitly.
- Dislikes are soft: they never block a suggestion. They influence selection ranking in AI Weekly Dinner Plan Generation (FEAT-03) and are the target of learned entries from Meal Rating & Preference Learning (FEAT-12, XBR-17), but FEAT-02.SPEC-002's safety check never excludes a recipe for a dislike alone.
- XBR-17: A meal rated down repeatedly by the same member becomes a learned soft dislike on that member's dietary rules; soft dislikes influence selection but never block a suggestion and never override an explicit rule -- this spec is the classification XBR-17's learned entries are subject to, same as any organiser-entered dislike.
- A learned dislike (origin = learned from ratings) never overwrites or weakens an explicit hard rule for the same ingredient; an explicit organiser edit always takes precedence for the same ingredient, per the dependency map's Contention note for Dietary Rule.

## Edge Cases

- **A member has both an explicit dislike and a hard allergy naming the same ingredient** -- The hard allergy classification governs: the recipe is excluded as a hard-rule failure, and the separate soft dislike entry has no additional effect since the recipe never reaches the ranking stage where dislikes matter.
- **A shared dinner has no vegetarian_option variant and one member is vegetarian** -- The recipe is excluded as a hard-rule failure for that member, since no variant exists to satisfy the vegetarian setting; this is the same outcome as any other hard-rule violation.
- **A rule_kind is per-person vegetarian for one member but the household has no other dietary restrictions** -- The recipe still must either be inherently vegetarian or carry a satisfying vegetarian_option variant; the absence of other restrictions does not loosen the vegetarian rule's hard treatment.
- **A learned soft dislike is added for an ingredient the member is also allergic to** -- FEAT-12 never creates a dislike entry that duplicates or conflicts with an existing hard allergy for the same allergen, per the Contention note; if it were ever attempted, the existing hard allergy classification takes precedence and the duplicate soft entry has no effect on the safety check.
- **Strength field is present with a value inconsistent with rule_kind (a data anomaly, e.g., an allergy recorded as soft)** -- This spec's derivation rule means strength is never independently set, so this state cannot arise through the product's own save path; FEAT-02.SPEC-002 always treats allergy and religious-rule kinds as hard regardless of any stored strength value, since the classification in this spec -- not the raw field -- is authoritative.

## Acceptance Criteria

**FEAT-02.SPEC-006-AC-01:** Given a household member has a peanut allergy recorded, when a candidate recipe containing peanuts is checked, then the recipe is excluded from the candidate pool as a hard-rule failure.

**FEAT-02.SPEC-006-AC-02:** Given a household member has a halal religious rule recorded, when a candidate recipe containing a non-halal ingredient is checked, then the recipe is excluded from the candidate pool as a hard-rule failure.

**FEAT-02.SPEC-006-AC-03:** Given a household member has a recorded dislike of mushrooms, when a candidate recipe containing mushrooms is checked, then the recipe is not excluded, since dislikes are soft and non-blocking.

**FEAT-02.SPEC-006-AC-04:** Given a household member has a per-person vegetarian setting and a shared dinner recipe is not inherently vegetarian but carries a vegetarian_option variant, when the recipe is checked, then the recipe passes and the variant is made available rather than the recipe being excluded.

**FEAT-02.SPEC-006-AC-05:** Given a household member has a per-person vegetarian setting and a shared dinner recipe is not inherently vegetarian and carries no vegetarian_option variant, when the recipe is checked, then the recipe is excluded as a hard-rule failure.

**FEAT-02.SPEC-006-AC-06:** Given Maya is entering a new Dietary Rule and selects rule_kind "allergy", when she saves it, then its strength is automatically set to hard with no independent strength control shown.

**FEAT-02.SPEC-006-AC-07:** Given Maya is entering a new Dietary Rule and selects rule_kind "dislike", when she saves it, then its strength is automatically set to soft.

**FEAT-02.SPEC-006-AC-08:** Given a meal is rated down repeatedly by Sam (FEAT-12, XBR-17), when the learned dislike is created, then this spec classifies it as soft, and it influences future plan selection without ever blocking a suggestion.

**FEAT-02.SPEC-006-AC-09:** Given a household member has both a hard allergy and a soft dislike naming the same ingredient, when a candidate recipe containing that ingredient is checked, then it is excluded as a hard-rule failure, and the soft dislike entry has no additional bearing on the outcome.

**FEAT-02.SPEC-006-AC-10:** Given no user role or screen offers an "override" or "show anyway" control on an excluded recipe, when any adult views the ineligible recipe, then no path exists to bypass the hard-rule exclusion.

**FEAT-02.SPEC-006-AC-11:** Given an explicit organiser-entered dietary rule and a learned soft dislike exist for the same ingredient, when FEAT-12 attempts to record the learned entry, then the explicit rule's classification and content take precedence and are never weakened by the learned entry, per the dependency map's Contention note.

**FEAT-02.SPEC-006-AC-12:** Given a household member's vegetarian setting is per-person and only one member holds it, when a candidate recipe is checked for the whole household, then only that member's vegetarian requirement is evaluated against the recipe -- other members' lack of a vegetarian setting has no bearing.

**FEAT-02.SPEC-006-AC-13:** Given Maya is entering a new Dietary Rule and selects rule_kind "per-person vegetarian", when she saves it, then the rule is treated as hard for exclusion purposes, subject to the shared-meal vegetarian_option accommodation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |
