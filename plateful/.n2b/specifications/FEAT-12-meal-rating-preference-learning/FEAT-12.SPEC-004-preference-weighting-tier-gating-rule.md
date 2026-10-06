---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-12.SPEC-004
spec_name: Preference Weighting & Tier-Gating Rule
spec_slug: preference-weighting-tier-gating-rule
parent_feature: FEAT-12
parent_feature_name: Meal Rating & Preference Learning
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 13
acceptance_criteria_count: 11
---

# Logic/Rule Spec: Preference Weighting & Tier-Gating Rule

## Overview

**Name:** Preference Weighting & Tier-Gating Rule
**ID:** FEAT-12.SPEC-004
**Type:** Logic/Rule
**Purpose:** Governs how accumulated ratings weight future meal selection (liked meals more often, disliked meals less often) and gates that learning effect -- but never rating capture itself -- to the paid tier.
**Parent Feature:** FEAT-12 -- Meal Rating & Preference Learning
**Governed Entity:** Rating (read-only, aggregate) -- weighting derivation consumed by AI Weekly Dinner Plan Generation (FEAT-03); Subscription (read-only) -- the tier condition that gates the effect

## Scope and Non-Goals

**In Scope:**
- The derivation rule for how a household's accumulated Ratings translate into a per-recipe weighting signal (liked more often, disliked less often)
- The paid-tier gate on the weighting effect (XBR-05): the exact condition under which the effect applies, and what happens on both sides of that condition
- The Subscription tier read this rule performs to determine whether the effect currently applies
- What happens to already-recorded ratings when a household's tier changes (upgrade, downgrade, lapsed payment)

**Non-Goals:**
- Rating capture itself -- never gated to any tier; capture is governed entirely by FEAT-12.SPEC-001 and FEAT-12.SPEC-002, and works identically on both tiers (XBR-05).
- The actual weekly plan generation process that applies this weighting -- owned by FEAT-03 (AI Weekly Dinner Plan Generation, FEAT-03.SPEC-010); this spec defines the derivation rule that generation consumes, not the generation process itself.
- The repeated-dislike detection that feeds a learned Dietary Rule entry -- governed by FEAT-12.SPEC-005 (Repeated-Dislike Learned Update), which this rule gates but does not define the detection logic for.
- Subscription upgrade, downgrade, and billing mechanics themselves -- owned end-to-end by FEAT-14 (Subscription & Billing Management); this spec only reads the resulting tier value.

## Governed Entity

**Entity:** Rating
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| member | reference | The Member Profile this rating is recorded against |
| planned_meal | reference | The cooked Planned Meal this rating attaches to |
| value | enum (up, down) | The thumbs up or thumbs down rating value |
| recorded_by | reference (optional) | The adult who recorded a proxy rating, where applicable |

**Entity (condition source):** Subscription
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| tier | enum (free, paid) | Read by this rule to determine whether the weighting effect currently applies to the household |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-03.SPEC-010 (AI Weekly Dinner Plan Generation, external to this feature) | AI plan generation | Reads accumulated Ratings and this rule's weighting derivation each time a weekly plan is generated for a paid-tier household; the tier gate is checked at generation time, not at rating-capture time |
| FEAT-12.SPEC-005 | Repeated-Dislike Learned Update | Checks this rule's tier gate before evaluating a repeated-dislike pattern -- the automation does not run its detection at all on a free-tier household |

## Field Validation Rules

No field validation rules in this spec -- Rating's field-level rules (member, planned_meal, value, recorded_by) are governed entirely by FEAT-12.SPEC-002; this spec reads value only, for weighting derivation, and never writes to Rating. Subscription's tier field carries no validation rule relevant here -- this spec reads it as a condition; its own validation is owned by FEAT-14.

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|--------------------|-----------|--------------------|
| Per-recipe weighting derivation | Rating.value (across all Ratings for a given recipe, aggregated across household members), Subscription.tier | For a paid-tier household, each recipe's likelihood of being selected in future plan generation increases with a higher proportion of thumbs-up ratings across the household's recorded ratings for that recipe, and decreases with a higher proportion of thumbs-down ratings; a recipe with no ratings yet carries no weighting adjustment (neutral, per FEAT-12.SPEC-002's unrated-is-neutral rule) | N/A -- this is a derivation applied silently within plan generation, not a user-facing validation |
| Tier gate on the weighting effect | Subscription.tier, Rating (all fields) | The weighting derivation above is applied during plan generation only when Subscription.tier is "paid" at the moment generation runs; on a free-tier household, the derivation is not applied at all -- plan generation (via FEAT-23's manual planning, since free-tier households do not receive AI generation) proceeds with no rating-based weighting | N/A -- no error; the household experiences unweighted (but still safety-checked) meal selection |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|-------------|-----------|----------------------------------------------|
| Benefit from the weighting effect (future plans reflecting accumulated ratings) | Every household member whose ratings are captured (Maya, Sam, Jordan by proxy, Jordan older-kid login once it exists) | Household's Subscription.tier is "paid" at the moment a plan is generated | Not a blocked action -- ratings are recorded normally regardless of tier (this spec never denies rating capture); on a free-tier household, future plans simply do not yet reflect the household's ratings until the household upgrades. No error state, no disabled control -- this is a silent, system-level condition on a derivation, not a user action |
| See the household's current tier as it relates to this effect | Maya (Organiser) | Always, via FEAT-14 (Subscription & Billing Management), which owns the tier's display | -- |
| See the household's current tier as it relates to this effect | Sam, Jordan (either kid row), unauthorized visitor | Never directly through this rule (tier display is FEAT-14's Billing field, where Sam and both kid rows have None or View, per the Access Matrix) | Not applicable to this spec -- any denial here is FEAT-14's, not this rule's |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|------------------------|-------------------|---------------------|
| per-recipe weighting signal | Derived from the proportion of thumbs-up vs. thumbs-down Ratings recorded against that recipe across the household, recalculated each time plan generation runs | On every AI Weekly Dinner Plan Generation run (FEAT-03.SPEC-010), paid tier only | No -- the household cannot directly set or override a recipe's weighting signal; it can only change the signal indirectly by rating more meals, or by using FEAT-04's swap to override any single suggestion in the moment |
| tier-gate outcome (effect applies / does not apply) | "Applies" when Subscription.tier reads "paid" at generation time; "does not apply" otherwise | On every plan generation run | No -- this is read directly from the Subscription entity FEAT-14 manages; this rule cannot be manually toggled independent of the actual subscription tier |

## Business Rules

- **XBR-05 (Tier gating):** AI plan generation, pantry-weighted suggestions, and learning from ratings are paid-tier capabilities; rating capture, pantry logging, and the shared grocery list stay free on both tiers. This spec is the "learning from ratings" half of that rule as it applies to FEAT-12.
- **A downgrade or lapsed payment never removes any past rating (XBR-05, XBR-16):** All previously recorded Ratings remain stored and available for the life of the household account (scope-boundaries.md SC-18) regardless of tier changes; only the weighting effect's application at generation time is gated, never the underlying data.
- **Upgrade takes effect on the household's next generation, not retroactively:** When a free-tier household upgrades, its next AI plan generation applies the weighting derivation using every rating recorded up to that point, including ratings gathered while the household was still on the free tier -- there is no separate "ratings gathered pre-upgrade don't count" restriction.
- **Downgrade or a lapsed-payment grace-period expiry stops the effect immediately at the household's next generation, not mid-week:** A plan already generated and active when the tier changes is not retroactively re-weighted or altered; the tier condition is evaluated only at the moment a new generation runs.
- **The repeated-dislike automation (FEAT-12.SPEC-005) is gated by this same rule:** A meal rated down repeatedly on a free-tier household is recorded (per FEAT-12.SPEC-002) but does not trigger a learned Dietary Rule update until the household is on the paid tier at the time the pattern would be evaluated (XBR-17).
- **Weighting never overrides safety:** This rule's per-recipe weighting is applied only among recipes that have already passed the app-enforced allergy and religious-rule check (XBR-01); it can never cause an unsafe recipe to be selected, since the safety check runs first and independently, owned by FEAT-02.

## Edge Cases

- **A household upgrades mid-week, after that week's plan was already generated on the free tier (i.e., built manually via FEAT-23)** -- The active week's plan is not retroactively re-weighted; the newly paid tier's weighting effect applies starting with the household's next AI-generated plan.
- **A household's payment fails and enters the grace period (FEAT-14) mid-week** -- The weighting effect remains applied for that week if a plan was already generated while still on active paid status; if the grace period expires before the next generation, that next generation runs unweighted (free-tier behavior), per FEAT-14's billing-state rules.
- **A recipe has ratings from before a member left the household (FEAT-09, anonymized influence per XBR-16)** -- The anonymized influence still contributes to the recipe's aggregate weighting signal; it is simply no longer attributable to a specific departed member. This spec's derivation operates on the aggregate, not on a named member's history, so the anonymization does not remove the signal.
- **A brand-new paid-tier household with zero ratings recorded yet** -- No recipe carries a weighting adjustment; plan generation proceeds exactly as it would for any recipe pool with no rating history (feature-overview.md's Rationale: "MVP plan generation must work well from ratings alone being absent").
- **A household downgrades and later re-upgrades** -- Ratings recorded during the free-tier interval (capture is never gated) are included in the weighting derivation once paid status resumes, exactly as if no downgrade had occurred.

## Acceptance Criteria

**FEAT-12.SPEC-004-AC-01:** Given a paid-tier household has recorded several thumbs-up ratings for a recipe and few thumbs-down ratings, when the next AI Weekly Dinner Plan Generation runs, then that recipe's likelihood of being selected increases relative to an unrated recipe.

**FEAT-12.SPEC-004-AC-02:** Given a paid-tier household has recorded predominantly thumbs-down ratings for a recipe, when the next AI Weekly Dinner Plan Generation runs, then that recipe's likelihood of being selected decreases relative to an unrated recipe.

**FEAT-12.SPEC-004-AC-03:** Given a free-tier household has recorded ratings on several meals, when that household's week is planned (via FEAT-23, Manual Weekly Planning), then those ratings are not applied as a weighting signal, since the household is not on the paid tier.

**FEAT-12.SPEC-004-AC-04:** Given a household on the free tier submits a rating on a cooked meal, when the submission is processed, then it is captured normally per FEAT-12.SPEC-002, with no tier restriction on the capture itself.

**FEAT-12.SPEC-004-AC-05:** Given a household upgrades from free to paid, when its next AI Weekly Dinner Plan Generation runs, then the weighting derivation applies using every rating recorded by the household up to that point, including ratings gathered before the upgrade.

**FEAT-12.SPEC-004-AC-06:** Given a paid-tier household downgrades to free, when the change takes effect, then no previously recorded rating is deleted, and the household's stored rating history remains available (scope-boundaries.md SC-18).

**FEAT-12.SPEC-004-AC-07:** Given a household's plan was already generated while on the paid tier and the household then downgrades mid-week, when the active week continues, then that already-generated plan is not retroactively re-weighted or altered.

**FEAT-12.SPEC-004-AC-08:** Given a recipe has no ratings recorded by anyone in the household, when plan generation runs on a paid-tier household, then the recipe carries no weighting adjustment (treated as neutral).

**FEAT-12.SPEC-004-AC-09:** Given a household member left the household and their ratings became anonymized influence (FEAT-09, XBR-16), when plan generation computes a recipe's weighting signal, then that anonymized influence still contributes to the aggregate signal.

**FEAT-12.SPEC-004-AC-10:** Given a recipe carries a strong dislike weighting signal on a paid-tier household, when plan generation selects candidates, then the recipe can still be selected if no safer or better-fitting alternative exists -- the weighting influences frequency, never eligibility, and never overrides the allergy/religious-rule safety check (XBR-01).

**FEAT-12.SPEC-004-AC-11:** Given a free-tier household's payment lapses into the grace period (FEAT-14) mid-week after a plan was already generated while paid, when that week continues, then the already-applied weighting on that plan is not retroactively removed; the next generation after the grace period expires runs unweighted if the household has not resumed paid status by then.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- owned by FEAT-12.SPEC-002 and FEAT-14; confirmed considered, not skipped) | 0 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 6 | 6 |
| Edge Cases | 5 | 5 |
