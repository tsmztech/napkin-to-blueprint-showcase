---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-25.SPEC-004
spec_name: Check-In Trend Calculation
spec_slug: check-in-trend-calculation
parent_feature: FEAT-25
parent_feature_name: Weekly Waste & Spend Check-In
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 26
acceptance_criteria_count: 13
---

# Logic/Rule Spec: Check-In Trend Calculation

## Overview

**Name:** Check-In Trend Calculation
**ID:** FEAT-25.SPEC-004
**Type:** Logic/Rule
**Purpose:** Derives the household's change against its starting point and against its weekly budget for display in the trend view.
**Parent Feature:** FEAT-25 -- Weekly Waste & Spend Check-In
**Governed Entity:** Waste & Spend Check-In (derived comparison values)

## Scope and Non-Goals

**In Scope:**
- The derivation logic for change_against_starting_point (waste and, where possible, spend) for each answered week
- The derivation logic for change_against_weekly_budget for each answered week that provided a spend value
- When each derivation runs, what it depends on, and what it produces when a dependency is missing
- Read-side visibility of the derived values, by role

**Non-Goals:**
- Field-level validation, requiredness, and the one-answer-per-week/latest-wins and editable-window rules for the Waste & Spend Check-In entity's stored fields -- owned by FEAT-25.SPEC-005 (Check-In Validation & Access Rules); this spec assumes the stored fields it reads are already valid and saved, and derives comparison values from them only
- Write access, submission, or correction of any stored field -- owned by FEAT-25.SPEC-001 (Weekly Check-In Card); this spec produces read-only, display-only values with no write-back to the entity
- Opening, skipping, or locking a week's record -- owned by FEAT-25.SPEC-003 (Weekly Check-In Cycle); this spec derives values only for weeks whose status that automation has already set

## Governed Entity

**Entity:** Waste & Spend Check-In
**Source:** Feature Dependency Map (feature-overview.md's Shared Context and Entity-Lifecycle Coverage Matrix)

| Field | Data Type | Description |
|-------|-----------|-------------|
| household | reference | The Household this record belongs to |
| week | date/period | The calendar week this record covers |
| waste_amount | enum (none \| a little \| a lot) | The household's answer for how much food was thrown away that week |
| spend | number, optional | The household's rough grocery spend that week, in its configured currency |
| starting_point_waste | enum (none \| a little \| a lot) | The household's one-time baseline: how much it typically threw away before Plateful |
| starting_point_spend | number, optional | The household's one-time baseline: what it typically spent on groceries before Plateful |
| status | enum (Offered \| Answered \| Skipped) | The week's lifecycle state |
| locked | boolean | Whether the week's record can still be edited |
| change_against_starting_point | derived (this spec) | The week's waste and spend compared against the household's starting point |
| change_against_weekly_budget | derived (this spec) | The week's spend compared against the household's current weekly_budget |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-25.SPEC-002 | Check-In Trend View | On render, recomputed for every displayed week each time the trend content is shown |

## Field Validation Rules

All input-side validation for the Waste & Spend Check-In entity's stored fields is governed by FEAT-25.SPEC-005 (Check-In Validation & Access Rules); each stored field is listed below with a pointer to that spec's rule, so this table remains a complete field inventory without duplicating its content. This spec introduces two additional fields, both system-computed with no user input, which carry their own rows below.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| household | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| week | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| waste_amount | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| spend | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| starting_point_waste | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| starting_point_spend | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| status | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| locked | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| change_against_starting_point | No validation beyond data type -- system-derived, never user input | Always | -- | -- | -- |
| change_against_weekly_budget | No validation beyond data type -- system-derived, never user input | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Starting-point comparison requires a starting point | waste_amount, starting_point_waste, spend, starting_point_spend | change_against_starting_point is computed for a given week only if the household's starting_point_waste exists; the waste half of the comparison always computes once a starting point exists (starting_point_waste is required at capture, per FEAT-25.SPEC-005), while the spend half computes only if both spend and starting_point_spend are present for that comparison | N/A -- not a rejectable input; an incomplete comparison simply omits the piece it cannot compute (FEAT-25.SPEC-002 shows its own placeholder text for the missing piece) |
| Budget comparison requires spend and a set budget | spend, Household.weekly_budget | change_against_weekly_budget is computed for a given week only if that week's spend was provided and the household's weekly_budget has been set (FEAT-01.SPEC-008) | N/A -- not a rejectable input; the comparison is simply omitted when either value is missing |
| Currency consistency before comparison | spend, starting_point_spend, Household.weekly_budget, Household.currency | All monetary values entering a comparison are read in the household's currently configured currency (FEAT-16.SPEC-001); a stored spend figure recorded under a previously configured currency is converted for display before any comparison runs, per XBR-11 (owned by FEAT-16) | N/A -- not a rejectable input; conversion is applied automatically, never surfaced as an error |

## Authorization Rules

This spec governs read-only, derived values with no write action of its own; the table below covers who may see the derived comparison values, mirroring the Access Matrix's Waste Check-In column that also governs FEAT-25.SPEC-001 and FEAT-25.SPEC-002.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View derived change-against-starting-point and change-against-budget values | Maya (Organiser) | Always | -- |
| View derived change-against-starting-point and change-against-budget values | Sam (Other Adult Member) | Always -- the same shared household values Maya sees | -- |
| View derived change-against-starting-point and change-against-budget values | Jordan (young kid profile, no login -- MVP) | Never | Values are never shown; a no-login profile has no access to any screen in the product |
| View derived change-against-starting-point and change-against-budget values | Jordan (older kid, limited login -- Later) | Never | Values are not shown in this role's view of the week's plan; the Access Matrix's Waste Check-In column is None for this role |
| View derived change-against-starting-point and change-against-budget values | Riley (Operator, support -- from v1) | Never | Values are not shown, even during an open Support Request's read-only view of the household's plan (FEAT-22); the Access Matrix's Waste Check-In column is None for Riley |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| change_against_starting_point (waste) | Rank each of the three waste_amount values on an ordinal scale where none = 0 (least waste), a little = 1, a lot = 2. For a given answered week, compute starting_point_waste's rank minus that week's waste_amount rank. A positive result is expressed as "reduced" (less waste than the starting point), zero as "no change," and a negative result as "increased." | Recomputed on every render of FEAT-25.SPEC-002, for every answered week that has waste_amount and the household has starting_point_waste | No -- always derived, never directly editable |
| change_against_starting_point (spend) | When both the week's spend and the household's starting_point_spend are present (both in the currently configured currency, converting first if needed per XBR-11), compute starting_point_spend minus that week's spend. A positive result is expressed as "spending less," zero as "no change," and a negative result as "spending more." | Recomputed on every render of FEAT-25.SPEC-002, for every answered week where both values exist | No -- always derived, never directly editable |
| change_against_weekly_budget | When the week's spend is present and Household.weekly_budget is set (both in the currently configured currency, converting first if needed per XBR-11), compute that week's spend minus Household.weekly_budget. A result of exactly zero or negative is expressed as "at or under budget" (stating the exact amount under, or "right at budget" for exactly zero); a positive result is expressed as "over budget" by the exact amount. | Recomputed on every render of FEAT-25.SPEC-002, for every answered week where spend is present and weekly_budget is set | No -- always derived, never directly editable |

## Business Rules

- Derivation is display-only: it never writes back to the Waste & Spend Check-In record or to Household; no stored field changes as a result of computing these values.
- A week with no spend value is excluded from change_against_weekly_budget and from the spend half of change_against_starting_point for that week; its waste half of change_against_starting_point still computes independently, since waste_amount and spend are independently optional (FEAT-25.SPEC-005).
- Until the household has a starting point, no change_against_starting_point value is computed for any week; FEAT-25.SPEC-002 shows its own no-starting-point placeholder content instead, consistent with the Feature Breakdown Brief's "Single evolving card" shared UI pattern.
- A Skipped week contributes no change values of any kind -- there is no waste_amount or spend to compare for a week with no answer -- and appears in the trend only as a gap, per feature-overview.md's Non-Goals.
- All monetary comparisons operate in the household's currently configured currency; a later currency change converts existing figures for display before any comparison runs, per XBR-11 (owned by FEAT-16), so no comparison ever mixes two currencies' raw numbers.

## Edge Cases

- **Household's starting_point_spend was left blank at capture, but later weeks provide spend** -- The waste half of change_against_starting_point still computes normally (starting_point_waste is always required at capture); the spend half of that same comparison has no baseline to compare against and is omitted, while change_against_weekly_budget still computes independently for those weeks (it depends only on spend and weekly_budget, not on starting_point_spend).
- **Household's weekly_budget has never been set** -- change_against_weekly_budget is never computed for any week until weekly_budget is set (FEAT-01); change_against_starting_point is unaffected, since it does not depend on weekly_budget.
- **This week's waste_amount ties the starting point exactly (e.g., started "A little," still "A little")** -- The comparison is explicitly "no change," never silently treated as "reduced" or omitted.
- **Spend exactly equal to weekly_budget** -- Treated as "at budget" (a difference of exactly zero), consistent with the product's own "at or under budget" framing (success-metrics.md).
- **Currency changes between when a week's spend was recorded and when the trend is next viewed** -- The stored spend figure is converted to the currently configured currency before change_against_weekly_budget or the spend half of change_against_starting_point runs, per XBR-11; the two values entering any single comparison are never in different currencies.
- **Household sets its weekly_budget for the first time after several weeks were already answered** -- change_against_weekly_budget begins appearing only for weeks answered from that point forward; earlier weeks show no budget comparison, since no budget value existed for them at the time (FEAT-25.SPEC-002's Edge Cases).

## Acceptance Criteria

**FEAT-25.SPEC-004-AC-01:** Given Maya's household started at "A lot" and this week's answer is "A little," when the trend view renders, then change_against_starting_point states the waste change as "reduced."

**FEAT-25.SPEC-004-AC-02:** Given Sam's household started at "None" and this week's answer is also "None," when the trend view renders, then change_against_starting_point states the waste change as "no change."

**FEAT-25.SPEC-004-AC-03:** Given a household started at "A little" and this week's answer is "A lot," when the trend view renders, then change_against_starting_point states the waste change as "increased."

**FEAT-25.SPEC-004-AC-04:** Given a household provided both a starting typical spend and this week's spend, and this week's spend is lower, when the trend view renders, then change_against_starting_point's spend comparison states the household is spending less.

**FEAT-25.SPEC-004-AC-05:** Given a household's starting_point_spend was left blank at capture, when the trend view renders for a week that does provide spend, then no spend comparison against the starting point is shown, while the waste comparison still renders normally.

**FEAT-25.SPEC-004-AC-06:** Given a household's weekly_budget is set and this week's spend is under it, when the trend view renders, then change_against_weekly_budget states the exact amount under budget.

**FEAT-25.SPEC-004-AC-07:** Given a household's weekly_budget is set and this week's spend exactly equals it, when the trend view renders, then change_against_weekly_budget states the week is right at budget.

**FEAT-25.SPEC-004-AC-08:** Given a household's weekly_budget is set and this week's spend exceeds it, when the trend view renders, then change_against_weekly_budget states the exact amount over budget.

**FEAT-25.SPEC-004-AC-09:** Given a household's weekly_budget has never been set, when the trend view renders, then no change_against_weekly_budget value is produced for any week.

**FEAT-25.SPEC-004-AC-10:** Given a week's spend was left blank, when the trend view renders, then no change_against_weekly_budget value is produced for that week, though its waste comparison still renders if a starting point exists.

**FEAT-25.SPEC-004-AC-11:** Given a household has no starting point yet, when the trend view would otherwise render, then no change_against_starting_point value is produced for any week.

**FEAT-25.SPEC-004-AC-12:** Given a week's spend was recorded under a currency the household has since changed, when the trend view renders, then the stored spend figure is converted to the currently configured currency before any comparison is computed.

**FEAT-25.SPEC-004-AC-13:** Given Riley is viewing a household's plan through an open Support Request (FEAT-22), when the plan renders, then no derived change value of any kind is shown anywhere in Riley's view.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 10 | 10 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
