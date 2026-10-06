---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-02.SPEC-009
spec_name: Safety Concern Eligibility & Re-offer Policy
spec_slug: safety-concern-eligibility-and-re-offer-policy
parent_feature: FEAT-02
parent_feature_name: Dietary Rules & Allergy Safety Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 7
acceptance_criteria_count: 11
---

# Logic/Rule Spec: Safety Concern Eligibility & Re-offer Policy

## Overview

**Name:** Safety Concern Eligibility & Re-offer Policy
**ID:** FEAT-02.SPEC-009
**Type:** Logic/Rule
**Purpose:** Keeps a recipe under an open safety report out of the household's candidate pool until the report resolves.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine
**Governed Entity:** Recipe (exclusion state, scoped per Household)

## Scope and Non-Goals

**In Scope:**
- The exclusion state of a recipe under an open safety-concern Support Request, per household
- Transitioning that state when a report is resolved: released back to the pool, or kept permanently excluded
- Authorization for who can change this exclusion state

**Non-Goals:**
- Creating the Support Request or performing the initial removal -- owned by FEAT-02.SPEC-004 (Safety Concern Intake & Removal), which instructs this policy
- Applying the operator's review outcome -- owned by FEAT-02.SPEC-005 (Safety Concern Resolution Outcome), which instructs this policy's release/keep-excluded transition
- The ordinary hard-rule safety check for a recipe not under any report -- owned by FEAT-02.SPEC-002 and FEAT-02.SPEC-006/007, which this policy's exclusion sits in front of, not in place of
- Cross-household exclusion -- excluded per the feature's own design: an open report against a recipe for one household never affects another household's candidate pool for the same recipe, since each household's Dietary Rule and report history are independent

## Governed Entity

**Entity:** Recipe (with its exclusion state tracked against the Support Request and Household that opened it)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| ingredients | list | Read by FEAT-02.SPEC-002, not altered by this policy |
| dietary_badges | derived | Reflects the exclusion state this policy governs when computed per household |
| (exclusion state, cross-referenced) Support Request.status | enum | Raised, Under review, Resolved -- the driving state for this policy's exclusion window |
| (exclusion state, cross-referenced) Support Request.planned_meal/recipe | reference | Identifies which Recipe this policy's exclusion applies to, and for which Household |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-02.SPEC-002 | Candidate Safety Check Execution | Step 2 of its processing logic -- consulted before any ingredient comparison runs |
| FEAT-02.SPEC-004 | Safety Concern Intake & Removal | Instructs this policy to begin the temporary exclusion when a report is created |
| FEAT-02.SPEC-005 | Safety Concern Resolution Outcome | Instructs this policy to release or permanently exclude the recipe when a report resolves |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| exclusion state | No validation beyond data type -- this is a derived state, not user-entered data | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Exclusion tracks the open Support Request | Support Request.status, Recipe exclusion state | A recipe is excluded for a household exactly while that household has a Support Request (kind = safety concern, referencing that recipe) with status Raised or Under review; the exclusion ends the moment status becomes Resolved, at which point FEAT-02.SPEC-005's outcome determines whether it becomes a standing permanent exclusion or is fully released | N/A -- this is a state-tracking rule, not a user-facing validation |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Trigger the start of an exclusion window | System (via FEAT-02.SPEC-004), on report creation | Always, on every new safety-concern Support Request | -- |
| Release a recipe from exclusion | System (via FEAT-02.SPEC-005), on a Resolved outcome confirming the recipe is safe | Only after the operator's resolution -- no earlier release path exists | -- |
| Convert a temporary exclusion to a permanent one | System (via FEAT-02.SPEC-005), on a Resolved outcome confirming the recipe is unsafe | Only after the operator's resolution | -- |
| Manually override or end an exclusion before resolution | No role, ever | Never -- neither Maya, Sam, nor Riley can lift an exclusion outside the FEAT-22 resolution workflow | No screen offers a "restore this recipe" control while a report is open; the recipe remains absent from every candidate-pool view for that household until FEAT-22 resolves the report |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|--------------------|
| exclusion state | Derived from the most recent Support Request status for the Recipe/Household pair: Raised or Under review -> excluded (temporary); Resolved with "safe" outcome -> not excluded; Resolved with "unsafe" outcome -> excluded (permanent) | Re-evaluated on every candidate safety check (FEAT-02.SPEC-002) and whenever a Support Request transitions | No -- fully system-derived |

## Business Rules

- A recipe under an open safety report is never re-offered to that household until the report resolves -- this is the Brief's own Validation & Limits statement, and the single source of truth every other spec in this feature defers to rather than tracking its own exclusion state.
- The exclusion is scoped per household -- an open report from one household never excludes the recipe for any other household.
- A permanent exclusion (from a "confirmed unsafe" resolution) is distinct from the temporary open-report exclusion: it does not expire or require renewal, and no automated process re-admits the recipe to that household's pool afterward.
- FEAT-02.SPEC-002 and FEAT-02.SPEC-004 both defer to this policy rather than each keeping their own exclusion state, per the Brief's Shared Validation section.

## Edge Cases

- **A household's Support Request for a recipe is Resolved, then the same household reports the same recipe again later** -- If the prior resolution was "confirmed unsafe," the recipe is already permanently excluded and the new report is still recorded (per FEAT-02.SPEC-004's duplicate-safe handling) but has no additional exclusion effect, since the recipe was already excluded. If the prior resolution was "confirmed safe," the new report opens a fresh temporary exclusion exactly as any first report would.
- **Two households report the same recipe independently** -- Each Household's exclusion state is tracked and resolved entirely independently; one household's permanent exclusion has no bearing on the other household's candidate pool.
- **A report is created for a recipe that is also excluded for unrelated reasons (e.g., incomplete ingredient data)** -- Both exclusions apply; FEAT-02.SPEC-002 checks this policy first (Step 2) and, finding the report-driven exclusion, stops there without needing to also evaluate the ingredient-completeness policy -- the recipe is excluded either way.
- **The household is deleted while a report is open** -- The exclusion state becomes moot with no household left to exclude the recipe for; no orphaned exclusion record persists in a way that could affect a future household, since exclusion is always keyed to a specific Household.

## Acceptance Criteria

**FEAT-02.SPEC-009-AC-01:** Given a safety-concern Support Request is created for a recipe (FEAT-02.SPEC-004), when any future candidate check runs for that household, then the recipe is excluded per this policy without re-running the ingredient comparison.

**FEAT-02.SPEC-009-AC-02:** Given a household's Support Request for a recipe is resolved with a "confirmed safe" outcome, when the next candidate check runs, then the recipe is no longer excluded under this policy and is evaluated normally by FEAT-02.SPEC-002.

**FEAT-02.SPEC-009-AC-03:** Given a household's Support Request for a recipe is resolved with a "confirmed unsafe" outcome, when any future candidate check runs, then the recipe remains permanently excluded for that household.

**FEAT-02.SPEC-009-AC-04:** Given one household has an open report on a recipe, when a different household considers the same recipe as a candidate, then that other household's check is unaffected by the first household's exclusion.

**FEAT-02.SPEC-009-AC-05:** Given a recipe was previously confirmed unsafe for a household and is reported again by the same household, when the new report is processed, then the recipe remains permanently excluded with no additional exclusion effect from the new report.

**FEAT-02.SPEC-009-AC-06:** Given a recipe was previously confirmed safe for a household and is reported again by the same household, when the new report is created, then a fresh temporary exclusion begins exactly as it would for a first-time report.

**FEAT-02.SPEC-009-AC-07:** Given no role has a control to manually lift an exclusion, when Maya looks for a way to restore an excluded recipe before the report resolves, then no such control exists anywhere in the product.

**FEAT-02.SPEC-009-AC-08:** Given a recipe under an open report also has incomplete ingredient data, when the candidate check runs, then the report-driven exclusion from this policy applies first and the check stops there.

**FEAT-02.SPEC-009-AC-09:** Given a household is deleted while a report on one of its recipes is open, when the deletion completes, then the exclusion state for that household is moot and has no effect on any other household.

**FEAT-02.SPEC-009-AC-10:** Given a Support Request's status is Raised or Under review, when this policy is consulted, then the recipe is treated as excluded for that household.

**FEAT-02.SPEC-009-AC-11:** Given a Support Request's status becomes Resolved, when this policy is next consulted, then the exclusion state reflects the recorded outcome (released or permanently excluded) rather than the prior temporary state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |
