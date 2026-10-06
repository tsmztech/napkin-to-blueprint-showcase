---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-03.SPEC-006
spec_name: Budget Fit & Estimated Total Rule
spec_slug: budget-fit-estimated-total-rule
parent_feature: FEAT-03
parent_feature_name: AI Weekly Dinner Plan Generation
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 15
acceptance_criteria_count: 11
---

# Logic/Rule Spec: Budget Fit & Estimated Total Rule

## Overview

**Name:** Budget Fit & Estimated Total Rule
**ID:** FEAT-03.SPEC-006
**Type:** Logic/Rule
**Purpose:** Computes the week's estimated cost against the household budget and determines the closest-fitting plan with an overrun note when no safe week fits.
**Parent Feature:** FEAT-03 -- AI Weekly Dinner Plan Generation
**Governed Entity:** Weekly Plan (estimated_total, over_budget_note fields)

## Scope and Non-Goals

**In Scope:**
- Computing a Weekly Plan's estimated_total from its seven Planned Meals' rough_cost figures
- Comparing estimated_total against the household's weekly_budget
- Selecting the closest-fitting safe combination and attaching over_budget_note when no safe combination fits
- Currency and unit display consistency for the computed total

**Non-Goals:**
- Setting the household's weekly_budget itself -- owned by Household Setup & Member Profiles (FEAT-01); this rule only reads that value
- Computing each recipe's rough_cost -- owned by Recipe Library (FEAT-08) and Recipe Import (FEAT-10) at the recipe level; this rule sums already-computed per-dinner costs after they are sized to the household by FEAT-03.SPEC-007
- Enforcing the allergy/religious-rule safety check on candidates -- owned by Dietary Rules & Allergy Safety Engine (FEAT-02); this rule operates only on the already safety-passed candidate pool handed to it during generation

## Governed Entity

**Entity:** Weekly Plan
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| week | date | The calendar week the plan covers |
| origin | enum | AI-generated or manually built |
| status | enum | Generated/Started, Reviewed, Approved, Active, Archived |
| approval | derived | Organiser approval (once per week) or auto-adoption at week start |
| estimated_total | number | The week's estimated cost against the household budget, in the household's currency -- governed by this spec |
| over_budget_note | text | Shown when no safe week fits the budget -- governed by this spec |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-03.SPEC-003 | Scheduled Weekly Plan Generation | During candidate selection, to compute estimated_total and decide whether an over-budget fallback is needed, before the Weekly Plan is created |
| FEAT-03.SPEC-004 | First-Plan Generation on Upgrade | Same enforcement point, for a household's first generation run |
| FEAT-03.SPEC-001 | Weekly Plan View | Displays estimated_total and over_budget_note verbatim in the weekly total banner; performs no independent calculation |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| estimated_total | Must be a positive number, sum of the seven Planned Meals' household-scaled rough_cost values | Always | On computation, during generation | No validation blocking is applicable -- this is a computed field, not user input | No |
| over_budget_note | No validation beyond data type -- either absent (plan fits budget) or a plain-language overrun note | Always | On computation, during generation | -- | -- |
| weekly_budget (read from Household) | Must be a positive amount in the household's configured currency; if entirely unset, generation treats budget fit as unconstrained until the household sets one (FEAT-01, Validation & Limits: "weekly budget must be a positive amount... optional during partial setup") | Household has not yet set a budget | Read at generation time | N/A -- this field is validated by FEAT-01, not this spec; this spec only reads its current value | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Budget comparison | estimated_total, weekly_budget (Household) | estimated_total is compared against weekly_budget; if estimated_total exceeds weekly_budget for every safety-passed seven-dinner combination available, the closest-fitting combination is selected and over_budget_note is set | N/A -- no error is raised; this produces a note, not a validation failure, since the household must always end up with a plan (XBR-07) |
| Over-budget note presence | estimated_total, over_budget_note, weekly_budget | over_budget_note is present if and only if estimated_total exceeds weekly_budget; it is absent whenever a safe, on-budget combination was selected | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Trigger the budget-fit computation | System (invoked by FEAT-03.SPEC-003, FEAT-03.SPEC-004 during generation) | Always, as part of generation | N/A -- no user-facing action exists to deny; this rule is invoked internally, not by direct user action |
| View estimated_total and over_budget_note | Maya (Organiser), Sam (Other Adult Member), Jordan (older kid, limited login -- Later) | Always, wherever the Weekly Plan is displayed (FEAT-03.SPEC-001), per each role's View access to Weekly Plan | -- |
| View estimated_total and over_budget_note | Jordan (young kid profile, no login -- MVP) | Never -- no login exists for this row | No sign-in path exists for this profile |
| View estimated_total and over_budget_note | Riley (Operator, support -- from v1) | Only while a Support Request for the household is open (FEAT-22, XBR-14) | Outside an open Support Request, Riley has no access to any household screen showing this data |
| Change weekly_budget (the input this rule reads) | Maya (Organiser) | Always | -- |
| Change weekly_budget | Sam (Other Adult Member), both Jordan rows, Riley | Never -- Household Setup is View or None for these roles per the Access Matrix | The budget field is not editable for these roles; Household Setup (FEAT-01) shows it read-only or hides the control entirely |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| estimated_total | Sum of the seven selected Planned Meals' rough_cost (each already scaled to household size by FEAT-03.SPEC-007) | On generation (FEAT-03.SPEC-003, FEAT-03.SPEC-004) | No -- always derived; it is never directly edited |
| over_budget_note | Set to a plain-language overrun statement (e.g., naming the estimated overrun amount) when estimated_total exceeds weekly_budget for every safety-passed combination considered; otherwise absent | On generation | No -- always derived |
| Currency and unit display of estimated_total | Rendered in the household's configured currency (FEAT-16) | On display (FEAT-03.SPEC-001) | No -- households change their currency setting through FEAT-16, not by overriding this display |

## Business Rules

- XBR-07 and XBR-11 govern the surrounding context: the plan always reaches an approvable or auto-adoptable state (this rule never blocks generation, only annotates it), and the total displays in the household's configured units and currency, converting for display if the household's locale settings change later.
- When no safe week fits the budget, the household receives the closest-fitting plan with a plain overrun note, never a silent overspend (product-features.md, FEAT-03 Primary Flows & Alternates).
- estimated_total is always computed after FEAT-03.SPEC-007's household-scaling, since an unscaled cost figure would not reflect what the household will actually spend.
- A household with no weekly_budget set yet (partial setup) receives a plan with estimated_total computed and displayed, but no over_budget_note is ever attached, since there is no budget to compare against.

## Edge Cases

- **Household has not yet set a weekly_budget** -- estimated_total is still computed and shown; over_budget_note is never attached in this case, since there is nothing to measure an overrun against.
- **estimated_total exactly equals weekly_budget** -- Treated as on-budget; over_budget_note is not attached (the comparison is "exceeds," not "meets or exceeds").
- **Every safety-passed candidate combination exceeds the budget by a large margin** -- The closest-fitting combination (smallest overrun) is still selected; over_budget_note states the estimated overrun in plain language rather than refusing to produce a plan.
- **Household changes its weekly_budget mid-week after a plan has already generated** -- The change applies to the household's next generation cycle; the current week's already-computed estimated_total and over_budget_note are not retroactively recalculated, consistent with FEAT-01's "later edit" behavior (changes apply to the next plan, not retroactively).
- **Household's currency setting changes mid-week (FEAT-16)** -- The existing estimated_total is converted for display in the new currency per XBR-11, without changing the underlying computed value's basis.
- **A single very expensive dinner makes the week over budget even though six other dinners are inexpensive** -- The rule operates on the total across all seven dinners, not per-dinner limits; a per-dinner cost is shown for transparency, but budget fit is judged only at the weekly total.

## Acceptance Criteria

**FEAT-03.SPEC-006-AC-01:** Given a household's seven selected dinners sum to less than its weekly_budget, when generation computes the total, then estimated_total is set to that sum and over_budget_note is absent.

**FEAT-03.SPEC-006-AC-02:** Given no safe combination of seven dinners fits the household's weekly_budget, when generation runs, then the closest-fitting safe combination is selected and over_budget_note states the estimated overrun in plain language.

**FEAT-03.SPEC-006-AC-03:** Given a household has not yet set a weekly_budget, when generation computes estimated_total, then the total is still shown and no over_budget_note is attached.

**FEAT-03.SPEC-006-AC-04:** Given a household's seven dinners sum to exactly its weekly_budget, when the comparison runs, then the plan is treated as on-budget and no over_budget_note appears.

**FEAT-03.SPEC-006-AC-05:** Given Maya views the Weekly Plan View, when the weekly total banner renders, then it shows estimated_total in the household's configured currency.

**FEAT-03.SPEC-006-AC-06:** Given Sam views the Weekly Plan View, when the weekly total banner renders, then he sees the same estimated_total and over_budget_note as Maya, consistent with his View access to Weekly Plan.

**FEAT-03.SPEC-006-AC-07:** Given Riley (Operator) has no open Support Request for a household, when Riley attempts to view that household's plan, then no screen showing estimated_total is reachable.

**FEAT-03.SPEC-006-AC-08:** Given Maya changes the household's weekly_budget mid-week, when the change is saved, then the current week's already-computed estimated_total and over_budget_note remain unchanged, and the new budget applies starting the next generation cycle.

**FEAT-03.SPEC-006-AC-09:** Given the household changes its currency setting (FEAT-16) mid-week, when the plan is next displayed, then estimated_total is converted for display in the new currency without recomputing the underlying total.

**FEAT-03.SPEC-006-AC-10:** Given Sam attempts to change the household's weekly_budget, when he looks for an editable budget control, then none is available to him, per his View access to Household Setup.

**FEAT-03.SPEC-006-AC-11:** Given a plan has one very expensive dinner among six inexpensive ones and the total still fits the budget, when generation computes estimated_total, then no over_budget_note is attached, since fit is judged on the weekly total, not per-dinner cost.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
