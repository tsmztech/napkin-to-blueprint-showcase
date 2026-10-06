---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-05.SPEC-005
spec_name: Pantry-Aware Plan Weighting Tier Gate
spec_slug: pantry-aware-plan-weighting-tier-gate
parent_feature: FEAT-05
parent_feature_name: Pantry-Aware Suggestions
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 5
acceptance_criteria_count: 10
---

# Logic/Rule Spec: Pantry-Aware Plan Weighting Tier Gate

## Overview

**Name:** Pantry-Aware Plan Weighting Tier Gate
**ID:** FEAT-05.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs which pantry behaviors run on the free tier (logging, off-list exclusion) versus the paid tier (plan weighting toward logged items), reading the household's Subscription to decide.
**Parent Feature:** FEAT-05 -- Pantry-Aware Suggestions
**Governed Entity:** Subscription (read-only reference; the gating condition this spec applies to Pantry-Aware Suggestions' own behavior)

## Scope and Non-Goals

**In Scope:**
- The conditional rule that determines whether FEAT-05.SPEC-006 (Pantry-to-Recipe Matching) runs for a given weekly plan generation
- Confirming that pantry logging (FEAT-05.SPEC-001) and off-list exclusion (FEAT-05.SPEC-007) apply on both tiers, unaffected by this gate
- The behavior when a household downgrades or its payment lapses mid-cycle

**Non-Goals:**
- Validating or changing Subscription fields (tier, billing_period, billing_state, billing_history) -- owned entirely by FEAT-14 (Subscription & Billing Management); this spec only reads the tier value to decide a Pantry-Aware Suggestions behavior
- The matching logic itself (which Pantry Items a candidate recipe would use) -- handled by FEAT-05.SPEC-006, which this spec gates but does not perform
- Tier gating for other features that share this same rule shape (FEAT-03's AI generation eligibility, FEAT-12's rating-based learning) -- each of those features' own Logic/Rule specs applies this same Subscription read independently; this spec is the reference point for the pantry-specific weighting behavior only, per the Feature Breakdown Brief's Shared Validation section

## Governed Entity

**Entity:** Subscription
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| tier | enum (free, paid) | The value this spec reads to decide whether pantry-aware plan weighting runs; authored and updated by FEAT-14 |
| billing_state | enum (Active, Payment failed / grace period, Cancelled, Reverted to free) | Read alongside tier to determine the household's effective standing for this week's plan generation; authored and updated by FEAT-14 |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-03 AI Weekly Dinner Plan Generation (plan generation processing) | AI Weekly Dinner Plan Generation | Checked once per weekly plan generation, before candidate recipes are matched against Pantry Items |
| FEAT-05.SPEC-006 | Pantry-to-Recipe Matching for Plan Callout | Checked before the matching computation runs; the matching logic does not execute at all when this gate is closed |

## Field Validation Rules

N/A -- this spec reads the Subscription's tier and billing_state fields but authors neither; field-level validation for Subscription belongs to FEAT-14. Both fields are noted here as "no validation beyond data type" from this spec's perspective, since this spec only branches on their already-validated values.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| tier | No validation beyond data type -- read-only reference, owned by FEAT-14 | Always | On plan generation | -- | -- |
| billing_state | No validation beyond data type -- read-only reference, owned by FEAT-14 | Always | On plan generation | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Effective paid standing | tier, billing_state | Pantry-aware plan weighting runs only when tier is paid AND billing_state is Active or within the grace period (Payment failed, 7-day grace) -- a Cancelled or Reverted-to-free billing_state closes the gate even if tier still shows paid mid-transition | N/A -- this is a silent gating condition, not a user-facing validation error; no error message is shown, the plan simply generates without pantry weighting |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View a pantry-weighted callout on a Planned Meal | Maya, Sam | Only when this gate is open for the household (paid tier, Active or grace-period standing) | On a closed gate, no callout is shown on any meal, since none was computed; this is not a permission denial, it is the absence of a paid-tier feature the household has not unlocked |
| View a pantry-weighted callout on a Planned Meal | Jordan (older kid, limited login -- Later) | Same condition as above -- this role has View access to the Weekly Plan | Same as above |
| View a pantry-weighted callout on a Planned Meal | Riley (Operator) | Only while an open Support Request exists, and only if the household's gate is open | Same as above; Riley additionally never sees anything beyond what the household's own plan shows |
| Change the household's tier (open or close this gate) | Maya (Organiser) | Always, through FEAT-14 -- this spec does not itself perform the change, only reacts to it | Sam, both kid rows, and Riley cannot change billing (owned entirely by FEAT-14's own Authorization Rules); this spec has no independent control to deny, since it never exposes a tier-change action of its own |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Pantry-aware plan weighting eligibility (derived) | True when tier is paid and billing_state is Active or in the 7-day payment-failed grace period; false otherwise (free tier, Cancelled, or Reverted to free) | Re-evaluated every time a weekly plan generates | No -- this value is entirely derived from the Subscription's current standing, never set directly |

## Business Rules

- XBR-05: AI plan generation, pantry-weighted suggestions, and learning from ratings are paid; free and downgraded households plan through Manual Weekly Planning; pantry logging and the shared list stay free on both tiers; a downgrade or lapsed payment never removes any past plan, rating, recipe, or pantry item.
- Pantry logging (FEAT-05.SPEC-001) and off-list grocery exclusion (FEAT-05.SPEC-007) are never gated by this spec -- they run identically on both tiers, per the feature's own Access field: "Logging pantry items (and keeping them off the grocery list) works on both tiers."
- When the gate is closed for a given plan generation, the household still receives a complete seven-dinner plan (per FEAT-03's own Insufficient-data flow); the absence of pantry weighting is never a blocking condition on generation.
- A downgrade or lapsed payment mid-week does not retroactively remove a pantry callout already shown on a plan generated while the gate was open; it only affects the next generation.

## Edge Cases

- **Household's payment fails mid-week, entering the 7-day grace period** -- The gate remains open (billing_state is within grace) for any plan generation that occurs during the grace period; if the grace period expires before the next generation, the gate closes at that generation.
- **Household upgrades from free to paid mid-week, after that week's plan already generated without weighting** -- The current week's plan is not retroactively re-weighted; the gate opens starting with the next weekly generation.
- **Household's tier field briefly shows "paid" during a billing_state transition to Cancelled** -- The cross-field rule's AND condition closes the gate the moment billing_state is Cancelled, regardless of the tier field's transitional value, since both fields must agree for the gate to be open.
- **Free-tier household has logged pantry items and never upgrades** -- Those items remain fully usable for off-list exclusion (FEAT-05.SPEC-007) indefinitely; they simply never influence AI plan selection, since Manual Weekly Planning (FEAT-23) is what free-tier households use to build their week, and pantry-aware weighting is specific to FEAT-03's AI generation.
- **Household downgrades, then re-upgrades within the same billing period** -- The gate reflects whatever standing is current at the moment of each plan generation; no historical averaging or hysteresis applies.

## Acceptance Criteria

**FEAT-05.SPEC-005-AC-01:** Given Maya's household is on the paid tier with an Active billing_state, when the weekly plan generates, then the gate is open and FEAT-05.SPEC-006's matching logic runs.

**FEAT-05.SPEC-005-AC-02:** Given Maya's household is on the free tier, when the household plans its week (through FEAT-23, Manual Weekly Planning), then no pantry-weighted callout is computed, since the gate applies only to FEAT-03's AI generation and the free tier does not receive an AI-generated plan.

**FEAT-05.SPEC-005-AC-03:** Given Maya's household's payment fails and enters the 7-day grace period, when the weekly plan generates during that grace period, then the gate remains open and pantry weighting still applies.

**FEAT-05.SPEC-005-AC-04:** Given Maya's household's grace period has expired without payment being resolved, when the next weekly plan generates, then the gate is closed and no pantry-weighted callout is computed for that week.

**FEAT-05.SPEC-005-AC-05:** Given Maya's household cancels its subscription, when the current billing period ends and the next weekly plan would generate, then the household plans through Manual Weekly Planning (FEAT-23) instead, and no pantry weighting applies.

**FEAT-05.SPEC-005-AC-06:** Given Maya's household downgrades to free tier, when Maya opens the pantry list, then all previously logged Active pantry items are still present and can still be cleared or added to, since logging is never gated by this spec.

**FEAT-05.SPEC-005-AC-07:** Given a plan generated last week while the gate was open and carries a pantry callout, when Maya's household's payment fails this week, then last week's plan still shows its original pantry callout -- it is not retroactively removed.

**FEAT-05.SPEC-005-AC-08:** Given Jordan (older kid, limited login) is viewing the current week's plan on a paid, Active household, when a dinner carries a pantry callout, then Jordan can see it, consistent with this role's View access to the Weekly Plan.

**FEAT-05.SPEC-005-AC-09:** Given Riley (Operator) is viewing a household's plan through an open Support Request, when that household's gate is closed (free tier), then Riley sees no pantry callout on any meal, matching exactly what the household itself sees.

**FEAT-05.SPEC-005-AC-10:** Given Maya's household's tier field transitionally shows "paid" while billing_state has already moved to Cancelled, when the weekly plan generates at that exact moment, then the gate is treated as closed, since both fields must agree for pantry weighting to apply.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 (both N/A -- read-only) | 2 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
