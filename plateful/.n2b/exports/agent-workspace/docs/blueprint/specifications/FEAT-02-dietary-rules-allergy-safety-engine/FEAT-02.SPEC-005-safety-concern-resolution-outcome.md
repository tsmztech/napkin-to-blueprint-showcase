---
document_type: spec
spec_type: automation
spec_id: FEAT-02.SPEC-005
spec_name: Safety Concern Resolution Outcome
spec_slug: safety-concern-resolution-outcome
parent_feature: FEAT-02
parent_feature_name: Dietary Rules & Allergy Safety Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Safety Concern Resolution Outcome

## Overview

**Name:** Safety Concern Resolution Outcome
**ID:** FEAT-02.SPEC-005
**Type:** Automation
**Purpose:** Applies the operator's review outcome to the excluded recipe's eligibility and triggers the household's outcome notice.
**Parent Feature:** FEAT-02 -- Dietary Rules & Allergy Safety Engine

## Scope and Non-Goals

**In Scope:**
- Reacting to the operator's Resolved transition on a safety-concern Support Request (FEAT-22)
- Releasing the recipe back to the household's candidate pool, or keeping it permanently excluded, based on the recorded outcome
- Triggering the household's resolution notice

**Non-Goals:**
- Performing the operator's review itself, or writing the Support Request's status or access_record -- owned entirely by FEAT-22 (Operator Read-Only Support Access); this automation only consumes the terminal Resolved transition
- Removing the meal or creating the Support Request -- already done by FEAT-02.SPEC-004 (Safety Concern Intake & Removal), a prior step this automation does not repeat
- Deciding what "released" means for future candidate checks in detail -- owned by FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy), which this automation updates
- Restoring the specific removed Planned Meal slot -- excluded per the feature's own scope: a fresh safe pick fills the slot through FEAT-04, the resolution does not undo the removal itself

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A safety-concern Support Request is resolved | FEAT-22 (Operator Read-Only Support Access) | Fires when the operator (Riley) transitions the Support Request's status from Under review to Resolved | Support Request (kind = safety concern, recipe reference, raised_by), the operator's recorded resolution outcome (recipe confirmed safe / recipe confirmed unsafe) |

## Processing Logic

1. Receive the resolved Support Request and its recorded outcome from FEAT-22.
2. Identify the Recipe the Support Request references and the Household it belongs to.
3. If the outcome confirms the recipe is safe (the reported concern did not hold up), instruct FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy) to release the recipe back to this Household's candidate pool.
4. If the outcome confirms the recipe is genuinely unsafe, instruct FEAT-02.SPEC-009 to keep the recipe permanently excluded for this Household (distinct from the temporary "open report" exclusion -- this is a confirmed, standing exclusion).
5. Trigger FEAT-02.SPEC-014 (Safety Concern Resolution Notice) to tell the household the outcome.
6. Take no action on the previously removed Planned Meal slot itself -- that slot remains empty (or however it was subsequently filled by a swap) regardless of the outcome.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Recipe released | Operator confirms the recipe is safe | Recipe's exclusion for this Household is lifted per FEAT-02.SPEC-009 -- it may pass future candidate checks again | Household receives the resolution notice stating the recipe is safe and available again | FEAT-02.SPEC-009, FEAT-02.SPEC-014, FEAT-02.SPEC-002 |
| Recipe kept excluded | Operator confirms the recipe is genuinely unsafe for this household | Recipe's exclusion for this Household becomes a standing exclusion per FEAT-02.SPEC-009 -- it never re-enters this household's candidate pool | Household receives the resolution notice stating the recipe remains excluded and why | FEAT-02.SPEC-009, FEAT-02.SPEC-014, FEAT-02.SPEC-002 |
| Resolution cannot be applied (processing failure) | The eligibility instruction to FEAT-02.SPEC-009 or the notice trigger to FEAT-02.SPEC-014 cannot be completed | No partial state -- the eligibility update and the resolution notice are treated as one atomic outcome; the recipe's exclusion state stays exactly as it was before this automation ran (fails closed, consistent with the feature's fail-closed posture) until a retry succeeds | Household receives no resolution notice until the retry succeeds; the Support Request stays Resolved in FEAT-22 (that write is not undone), but this automation retries the eligibility update and notice trigger automatically without requiring the operator to re-resolve it | FEAT-02.SPEC-009, FEAT-02.SPEC-014 |

## Data Model

**Reads:** Support Request -- status, kind, planned_meal/recipe, raised_by, and the operator's recorded resolution outcome (written by FEAT-22).
**Creates:** None.
**Updates:** None directly on Support Request (FEAT-22 owns that write); instructs FEAT-02.SPEC-009's exclusion-state tracking for the Recipe/Household pair.
**Deletes:** None.

## Business Rules

- XBR-08: A safety-concern report reaches the operator for review and tells the household the outcome once resolved -- this automation is the household-facing half of that lifecycle, paired with FEAT-02.SPEC-004's intake half.
- Only a Resolved transition on a safety-concern kind Support Request triggers this automation -- a general support Support Request's resolution (FEAT-18) has no recipe-eligibility consequence and does not invoke this spec.
- The recipe's eligibility state (released or kept excluded) is owned exclusively by FEAT-02.SPEC-009; this automation issues the instruction but never writes candidate-pool state itself.
- A "kept excluded" outcome is a standing decision distinct from the temporary open-report exclusion FEAT-02.SPEC-004 applied -- resolving a report to "unsafe" does not merely extend the temporary state, it converts it to permanent for that household.

## Edge Cases

- **The same recipe has two open Support Requests from different households, and one household's report resolves** -- Each Household's exclusion state is tracked independently per FEAT-02.SPEC-009; resolving one household's report has no effect on any other household's exclusion of the same recipe.
- **The recipe was already edited or re-imported (FEAT-10) between the report and its resolution** -- The resolution outcome applies to the recipe's current identity regardless of intervening edits; if released, the edited recipe still must pass FEAT-02.SPEC-002's ordinary safety check again before appearing in a plan, per XBR-19 -- release from this automation lifts only the report-driven exclusion, not the ordinary safety check.
- **The household is deleted before the resolution completes** -- No resolution notice is delivered (there is no household left to notify); the eligibility update is a no-op since a deleted household has no future candidate pool to affect.
- **Concurrent trigger firing (two different households' reports on two different recipes resolve at effectively the same time)** -- Each resolution processes independently against its own Recipe/Household pair; there is no shared state between the two runs.
- **Trigger fires while a previous resolution for the same recipe and household is still in flight** -- This scenario cannot occur under FEAT-22's ownership: a single Support Request can only be resolved once (status transitions from Under review to Resolved exactly one time), so no second resolution trigger for the same Support Request exists to overlap with the first.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22 (Operator Read-Only Support Access) | Triggered by (inbound) | The operator's Resolved transition fires this automation |
| FEAT-02.SPEC-004 (Safety Concern Intake & Removal) | References (inbound) | The prior step this automation's Support Request originated from |
| FEAT-02.SPEC-009 (Safety Concern Eligibility & Re-offer Policy) | Triggers (outbound) | Releases or permanently excludes the recipe based on the outcome |
| FEAT-02.SPEC-014 (Safety Concern Resolution Notice) | Triggers (outbound) | Tells the household the resolved outcome |
| FEAT-02.SPEC-002 (Candidate Safety Check Execution) | Affects (outbound) | Future checks reflect the released or permanently excluded state |

## Analytics and Success Signals

- **safety_concern_resolved** (outcome: released / kept_excluded) -- supports success-metrics.md: "Zero Allergy Incidents"
- **safety_concern_recipe_released** (recipe reference) -- supports success-metrics.md: "Zero Allergy Incidents"

## Acceptance Criteria

**FEAT-02.SPEC-005-AC-01:** Given Riley resolves a safety-concern Support Request confirming the recipe is safe, when this automation runs, then the recipe's exclusion for that household is lifted and it may pass future candidate checks again.

**FEAT-02.SPEC-005-AC-02:** Given Riley resolves a safety-concern Support Request confirming the recipe is genuinely unsafe, when this automation runs, then the recipe becomes permanently excluded for that household and never re-enters its candidate pool.

**FEAT-02.SPEC-005-AC-03:** Given a Support Request is resolved, when this automation completes, then the household receives the Safety Concern Resolution Notice (FEAT-02.SPEC-014) stating the outcome.

**FEAT-02.SPEC-005-AC-04:** Given a general support Support Request (not a safety concern) is resolved by FEAT-18, when the resolution completes, then this automation does not fire, since it applies only to safety-concern kind requests.

**FEAT-02.SPEC-005-AC-05:** Given a recipe is released back to the candidate pool by this automation, when FEAT-02.SPEC-002 next checks that recipe for the household, then it evaluates the recipe's ingredients normally, since the report-driven exclusion no longer applies.

**FEAT-02.SPEC-005-AC-06:** Given the same recipe has open reports from two different households and one household's report resolves, when this automation runs, then only that household's exclusion state changes; the other household's exclusion is unaffected.

**FEAT-02.SPEC-005-AC-07:** Given a recipe was edited after the report but before resolution, when the resolution releases it, then the released recipe must still pass FEAT-02.SPEC-002's ordinary safety check again before it can appear on any plan.

**FEAT-02.SPEC-005-AC-08:** Given the household is deleted before the resolution completes, when this automation runs, then no resolution notice is delivered and the eligibility update has no effect.

**FEAT-02.SPEC-005-AC-09:** Given two different households' reports on two unrelated recipes resolve at effectively the same time, when both trigger this automation, then each resolution processes independently without affecting the other.

**FEAT-02.SPEC-005-AC-10:** Given the eligibility instruction to FEAT-02.SPEC-009 or the notice trigger to FEAT-02.SPEC-014 cannot be completed, when this automation runs, then the recipe's exclusion state remains exactly as it was before the automation ran, no resolution notice is delivered, and the automation retries both steps together automatically until they succeed.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
