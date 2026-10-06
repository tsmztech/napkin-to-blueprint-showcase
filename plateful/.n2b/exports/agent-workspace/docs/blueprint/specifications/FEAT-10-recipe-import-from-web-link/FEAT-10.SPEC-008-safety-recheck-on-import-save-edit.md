---
document_type: spec
spec_type: automation
spec_id: FEAT-10.SPEC-008
spec_name: Safety Re-check on Import Save/Edit
spec_slug: safety-recheck-on-import-save-edit
parent_feature: FEAT-10
parent_feature_name: Recipe Import from Web Link
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Safety Re-check on Import Save/Edit

## Overview

**Name:** Safety Re-check on Import Save/Edit
**ID:** FEAT-10.SPEC-008
**Type:** Automation
**Purpose:** Runs the household's allergy/religious-rule safety check on every initial save and every later edit of an imported recipe before it can appear in a plan.
**Parent Feature:** FEAT-10 -- Recipe Import from Web Link

## Scope and Non-Goals

**In Scope:**
- Triggering the household's safety check (owned by FEAT-02) immediately after an imported recipe is created or edited
- Defining the import-side consequence of a Pass, an incomplete-data Fail, or a rule-conflict Fail
- Ensuring an edited imported recipe cannot appear in a plan again until it passes the re-check (XBR-19)

**Non-Goals:**
- Performing the allergy/religious-rule determination itself -- that determination, including how ingredients are matched against dietary rules, is FEAT-02's authority (XBR-01); this spec governs only when the check is triggered for imported recipes, not how it decides (feature-overview.md, Shared Validation).
- Computing or storing the dietary badge shown on a recipe -- feature-overview.md states dietary_badges is never written by this feature; FEAT-02 computes it live at view time once this automation's trigger has run.
- Re-checking starter recipes -- starter content's safety-relevant completeness is FEAT-08.SPEC-004's responsibility at seeding time; this spec covers only the imported slice of the Recipe entity.
- Notifying the household when a recipe fails the re-check -- product-features.md's Communications field states import is a self-initiated, in-app action with no notifications; a failed recipe is surfaced only as an ineligibility explanation the next time it is viewed, not through a push or email notification.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Imported recipe created (link-based) | FEAT-10.SPEC-002 (Review Extracted Recipe) | Fires immediately after the Recipe record is created (no duplicate found, FEAT-10.SPEC-006) | The newly created Recipe's ingredients (quantity + unit + name), owning_household |
| Imported recipe created (manual entry) | FEAT-10.SPEC-003 (Manual Recipe Entry) | Fires immediately after the Recipe record is created (no duplicate found, FEAT-10.SPEC-006) | The newly created Recipe's ingredients (quantity + unit + name), owning_household |
| Imported recipe edited | FEAT-10.SPEC-004 (Edit Imported Recipe) | Fires immediately after an accepted edit updates the Recipe record | The updated Recipe's ingredients (quantity + unit + name), owning_household |

## Processing Logic

1. Receive the Recipe record just created or edited, including its confirmed ingredient list (quantity, unit, and name per line) and owning_household.
2. Hand the ingredient data off to FEAT-02 (Dietary Rules & Allergy Safety Engine) for evaluation against the owning household's dietary rules.
3. Receive FEAT-02's determination, which is one of: every ingredient recognized and no hard-rule conflict (Pass); one or more ingredients missing a quantity/unit or carrying a name FEAT-02 cannot recognize (Fail -- incomplete data); an ingredient conflicts with a stated allergy or religious rule held by a member of the owning household (Fail -- rule conflict).
4. If Pass, no further action is required from this automation -- the recipe is eligible to appear in candidate pools from this moment, and FEAT-02 computes its dietary badge live at each subsequent view.
5. If Fail (either reason), the recipe remains saved in the household's pool but is excluded from every candidate path (FEAT-03 generation, FEAT-23 manual picks, FEAT-04 swap alternatives) until a further edit through FEAT-10.SPEC-004 resolves the issue and this automation re-runs.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Safety check passed | FEAT-02 finds every ingredient recognized and no hard-rule conflict | None stored on the Recipe record (dietary_badges is computed live by FEAT-02, never persisted) | None shown at save time (import has no notifications); the recipe's next view on FEAT-08.SPEC-002 shows the standard "checked against allergies" badge and "always check labels" disclaimer | FEAT-08.SPEC-002, FEAT-03, FEAT-23, FEAT-04 |
| Safety check failed -- incomplete ingredient data | One or more ingredients are missing a quantity/unit, or carry a name FEAT-02 cannot recognize | None stored directly; the recipe's eligibility for any candidate pool is excluded on every subsequent evaluation until corrected | None shown at save time; the recipe's detail view (FEAT-08.SPEC-002) shows an ineligibility explanation in place of the standard badge | FEAT-08.SPEC-002, FEAT-03, FEAT-23, FEAT-04 |
| Safety check failed -- allergy/religious-rule conflict | An ingredient conflicts with a hard dietary rule held by a member of the owning household | None stored directly; excluded from every candidate path for this household under its current dietary rules | None shown at save time; the recipe's detail view shows an ineligibility explanation naming that it does not currently meet the household's dietary rules | FEAT-08.SPEC-002, FEAT-03, FEAT-23, FEAT-04 |
| Automation failure (hand-off to FEAT-02 cannot complete) | A processing error prevents the check from running | None -- the recipe remains saved | No error is shown to the household; the recipe is treated as not-yet-checked and excluded from candidate pools, since XBR-01's fail-closed default treats an unconfirmed check the same as a failed one | FEAT-08.SPEC-002, FEAT-03, FEAT-23 |

## Data Model

**Reads:** Recipe record -- ingredients (quantity + unit + name), owning_household; Dietary Rule records (via FEAT-02) belonging to members of the owning household.
**Creates:** None.
**Updates:** None on the Recipe record itself -- dietary_badges is a live-computed value FEAT-02 derives at view time, never written by this automation.
**Deletes:** None.

## Business Rules

- Every initial save of an imported recipe (link-based or manual) triggers this automation exactly once, immediately (XBR-01, XBR-19).
- Every accepted edit of an already-saved imported recipe re-triggers this automation before the edited recipe can appear in a plan again (XBR-19).
- The save or edit itself always completes regardless of the safety outcome -- a Fail never blocks the save or edit action, per FEAT-02's fail-closed posture applying to plan eligibility, not to the ability to keep the recipe in the household's pool.
- Removing an imported recipe (FEAT-10.SPEC-004) does not trigger this automation -- there is nothing to re-check once the record no longer exists.
- The determination itself (Pass, incomplete, or conflict) is entirely FEAT-02's authority; this automation only defines when the check runs for imported recipes and what happens to import-side eligibility as a result.

## Edge Cases

- **Recipe is removed while this automation's check is still in flight** -- The check's result is discarded on completion; there is no Recipe record left to update eligibility for, and no error is surfaced to any household member.
- **Household's dietary rules change (e.g., a new allergy added) after this automation already returned a Pass for a recipe** -- This automation does not re-run on its own from a dietary-rule change; that re-evaluation is XBR-02's responsibility (a new or tightened hard rule triggers its own re-check across the household's plan and recipes), not a trigger this spec defines.
- **Concurrent trigger firing (Maya and Sam each edit different fields of the same imported recipe from separate devices at effectively the same time)** -- Only one edit succeeds at the data layer, per FEAT-10.SPEC-004's reject-with-refresh conflict resolution; this automation fires exactly once, against the single edit that was actually accepted. The rejected edit never reaches this automation, since it never became a persisted change.
- **Trigger fires while a previous run is in flight for the same recipe** -- FEAT-10.SPEC-004's Save button is disabled during save, so a second edit-triggered run cannot start for the same recipe until the first save (and its automation run) completes. A run triggered by this recipe's initial creation (FEAT-10.SPEC-002 or FEAT-10.SPEC-003) and a later edit-triggered run cannot overlap either, since the edit screen only becomes reachable once the recipe already exists and its creation-time run has resolved.
- **Two different imported recipes are saved by different household members at the same time** -- Each recipe's safety check runs independently against the same household's dietary rules; neither run affects or waits on the other.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-002 (Review Extracted Recipe) | Triggered by (inbound) | Successful creation triggers this automation |
| FEAT-10.SPEC-003 (Manual Recipe Entry) | Triggered by (inbound) | Successful creation triggers this automation |
| FEAT-10.SPEC-004 (Edit Imported Recipe) | Triggered by (inbound) | Every accepted edit re-triggers this automation |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | Triggers (outbound) | Hands off ingredient data for the safety determination |
| FEAT-08.SPEC-002 (Recipe Detail View) | Affects (outbound) | The badge or ineligibility explanation shown reflects this automation's outcome |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Affects (outbound) | Candidate pool eligibility reflects this automation's outcome |
| FEAT-23 (Manual Weekly Planning) | Affects (outbound) | Candidate pool eligibility reflects this automation's outcome |
| FEAT-04 (One-Tap Meal Swap) | Affects (outbound) | Swap alternative eligibility reflects this automation's outcome |

## Analytics and Success Signals

- **imported_recipe_edited** (fields changed, safety outcome: passed / failed_incomplete / failed_conflict) -- N/A -- no success-metrics.md metric is connected to Recipe Import from Web Link; retained so households and support can see how often an edit changes a recipe's plan eligibility.
- **safety_recheck_completed** (trigger: create / edit; outcome: passed / failed_incomplete / failed_conflict) -- supports success-metrics.md: "Zero Allergy Incidents" (this event is the operational trace confirming every imported-recipe save and edit actually passed through the app-enforced check the metric's zero-incidents target depends on).
- **safety_recheck_failed_processing** (trigger: create / edit) -- N/A -- no success-metrics.md metric tracks automation-failure volume directly; retained so the fail-closed guarantee (an unconfirmed check excludes the recipe) is observable when the hand-off itself breaks.

## Acceptance Criteria

**FEAT-10.SPEC-008-AC-01:** Given Maya saves a freshly extracted recipe on FEAT-10.SPEC-002 with every ingredient complete and no allergy conflict, when the save completes, then this automation runs and the recipe becomes eligible to appear in candidate pools immediately.

**FEAT-10.SPEC-008-AC-02:** Given Sam saves a manually entered recipe missing a unit on one ingredient, when the save completes, then this automation returns a Fail (incomplete data), and the recipe is excluded from every candidate path until corrected.

**FEAT-10.SPEC-008-AC-03:** Given Maya saves a recipe containing an ingredient that conflicts with a household member's stated allergy, when the save completes, then this automation returns a Fail (rule conflict), and the recipe is excluded from every candidate path under the household's current dietary rules.

**FEAT-10.SPEC-008-AC-04:** Given Sam edits an already-saved imported recipe's ingredients, when the edit is accepted, then this automation re-runs before the edited recipe can appear in a plan again, per XBR-19.

**FEAT-10.SPEC-008-AC-05:** Given Maya's recipe previously failed the safety check for an unrecognizable ingredient, when she edits the recipe to clarify that ingredient and saves, then this automation re-runs and, if it now passes, the recipe becomes eligible for candidate pools again.

**FEAT-10.SPEC-008-AC-06:** Given Sam removes an imported recipe, when the removal completes, then this automation does not run, since there is no record left to check.

**FEAT-10.SPEC-008-AC-07:** Given the hand-off to FEAT-02 fails due to a processing error, when Maya's recipe was just saved, then the recipe remains saved but is treated as not-yet-checked and excluded from candidate pools until a retry succeeds.

**FEAT-10.SPEC-008-AC-08:** Given Maya and Sam each attempt to edit the same imported recipe from separate devices at effectively the same time, when FEAT-10.SPEC-004's conflict resolution accepts only one edit, then this automation fires exactly once, against the accepted edit only.

**FEAT-10.SPEC-008-AC-09:** Given a household adds a new allergy rule after an imported recipe already passed this automation's check, when the new rule is added, then this automation does not itself re-run -- any re-evaluation of the recipe under the new rule is governed by XBR-02, not this spec.

**FEAT-10.SPEC-008-AC-10:** Given a recipe passes this automation's check, when a household member later views it on FEAT-08.SPEC-002, then no data was written by this automation to the Recipe record -- the "checked against allergies" badge is computed live by FEAT-02 at that view.

**FEAT-10.SPEC-008-AC-11:** Given the safety check itself fails to complete (processing error) on an initial save, when the failure occurs, then no notification is sent to the household, consistent with the feature's no-notifications communications posture -- the recipe simply shows an ineligibility explanation on its next view.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (create link-based, create manual, edit) | 3 |
| Outcome Paths | 4 (passed, failed incomplete, failed conflict, automation failure) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |
