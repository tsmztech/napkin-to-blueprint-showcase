---
document_type: spec
spec_type: automation
spec_id: FEAT-12.SPEC-005
spec_name: Repeated-Dislike Learned Update
spec_slug: repeated-dislike-learned-update
parent_feature: FEAT-12
parent_feature_name: Meal Rating & Preference Learning
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Repeated-Dislike Learned Update

## Overview

**Name:** Repeated-Dislike Learned Update
**ID:** FEAT-12.SPEC-005
**Type:** Automation
**Purpose:** On a pattern of repeated down-ratings from the same member for the same meal, merges a soft dislike into that member's Dietary Rule data, on the paid tier only.
**Parent Feature:** FEAT-12 -- Meal Rating & Preference Learning

## Scope and Non-Goals

**In Scope:**
- Detecting a repeated-down-rating pattern for the same (member, recipe) combination
- Evaluating whether the pattern qualifies for a learned soft dislike, gated to the paid tier (FEAT-12.SPEC-004)
- Reading the member's existing Dietary Rule entries for the recipe's relevant ingredient before merging, so an explicit rule is never overwritten or weakened
- Merging a new soft-dislike Dietary Rule entry (origin: learned) when the pattern qualifies and no conflicting explicit rule exists

**Non-Goals:**
- Detecting the individual rating submissions themselves -- governed by FEAT-12.SPEC-001 (capture) and FEAT-12.SPEC-002 (submission/change rules); this automation only evaluates the accumulated pattern after each qualifying submission.
- The paid-tier gate's definition and the weighting derivation ratings feed into plan generation -- governed by FEAT-12.SPEC-004; this automation consumes that same gate rather than redefining it.
- Creating, editing, or deleting an explicit Dietary Rule entry -- excluded per the Feature Dependency Map's Connected Entities line, which scopes this feature to "update" (merge) only; explicit dietary rules are created only through FEAT-01 (Household Setup & Member Profiles).
- Deriving any health, nutrition, or medical judgment from the pattern -- excluded per scope-boundaries.md (SC-06): a learned soft dislike is a preference signal for meal selection only, never a health or diet-advice conclusion.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A qualifying down-rating is submitted for a member and recipe | FEAT-12.SPEC-002 (Rating Submission, Change & Proxy Rules) | Fires after a Rating with value "down" is created or changed to "down" for a (member, planned_meal) pair, where the household's Subscription tier is "paid" at the moment of evaluation (FEAT-12.SPEC-004) | The member, the planned_meal's recipe, the new value, and the member's full rating history for that same recipe across all past meals |
| A down-rating is changed to up (pattern-breaking event) | FEAT-12.SPEC-002 | Fires when a previously "down" rating for a (member, recipe) pair is changed to "up" before archival, on a paid-tier household | The member, the recipe, the updated rating history for that (member, recipe) pair |

## Processing Logic

1. Receive the rating event (new or changed value) for a (member, recipe) pair from FEAT-12.SPEC-002, together with the household's current Subscription tier.
2. If the household's Subscription tier is not "paid" at this moment, take no action and end processing (FEAT-12.SPEC-004's tier gate) -- the rating is still stored per FEAT-12.SPEC-002, but this automation does not evaluate it further.
3. If the household is paid tier, gather the member's complete rating history for the same recipe, across every meal instance of that recipe the household has ever served.
4. Count the number of down-ratings for this (member, recipe) pair within that history, counting only the member's current value for each meal instance (a meal changed from down to up no longer counts as a down-rating; a meal changed from up to down now counts).
5. Compare the count against the repeated-dislike threshold: platform parameter: `repeated-dislike-rating-count`.
6. If the count meets or exceeds the threshold, identify the recipe's primary disliked ingredient signal: the specific ingredient the recipe is most centrally built around (e.g., a recipe's named primary protein or vegetable), used as the allergen/ingredient value for the learned Dietary Rule entry.
7. Read the member's existing Dietary Rule entries for that same ingredient.
8. If an explicit Dietary Rule entry already exists for that ingredient (any strength, any origin other than "learned"), take no merge action -- the explicit rule already governs that ingredient and is never overwritten or weakened (XBR-17).
9. If a learned entry for that ingredient already exists (from a prior run of this same automation), leave it as-is -- it already reflects this pattern; no duplicate entry is created.
10. If no existing entry (explicit or learned) governs that ingredient, merge a new Dietary Rule entry: member set to the rated member, rule_kind "dislike", strength "soft", allergen/ingredient set to the recipe's primary ingredient signal, origin "learned".
11. If the pattern-breaking event (a down-rating changed to up) drops the count below the threshold, take no automatic removal action on any Dietary Rule entry already merged -- a learned entry, once merged, is not automatically retracted by a single reversed rating (see Business Rules).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| No action -- free tier | Household's Subscription tier is not "paid" at evaluation time | None | None -- the rating itself was already confirmed on FEAT-12.SPEC-001; no separate feedback for this automation's inaction | FEAT-12.SPEC-004 (tier gate consumed) |
| No action -- below threshold | Paid tier, but the count of down-ratings for this (member, recipe) pair is below platform parameter: `repeated-dislike-rating-count` | None | None -- rating pattern tracking is silent | -- |
| No action -- explicit rule already governs the ingredient | Paid tier, threshold met, but an explicit Dietary Rule entry (non-learned) already exists for the recipe's primary ingredient | None | None to the household directly; the existing explicit rule continues to govern the ingredient as before | FEAT-01 (Dietary Rule ownership) |
| No action -- learned entry already exists | Paid tier, threshold met, but a learned soft-dislike entry for that ingredient was already merged by a prior run | None (idempotent) | None | -- |
| Learned soft dislike merged | Paid tier, threshold met, no existing entry (explicit or learned) for the recipe's primary ingredient | A new Dietary Rule entry is created: member, rule_kind "dislike", strength "soft", allergen/ingredient, origin "learned" | The new soft dislike becomes visible wherever the member's Dietary Rule data is shown (FEAT-01), and going forward, that ingredient's soft-dislike status can influence future recipe selection alongside FEAT-12.SPEC-004's rating-based weighting, without blocking any suggestion (XBR-17) | FEAT-01 (Household Setup & Member Profiles), FEAT-03 (plan generation reads updated Dietary Rule data), FEAT-02 (Dietary Rules & Allergy Safety Engine re-evaluates against the updated rule set) |
| Automation failure | Processing error while gathering rating history or merging the Dietary Rule entry | None -- no partial merge is left behind | None visible to the household; the rating that triggered evaluation remains correctly stored regardless of this automation's outcome | -- |

## Data Model

**Reads:** Rating -- the member's full rating history for the triggering recipe, across all meal instances, to count qualifying down-ratings. Dietary Rule -- the member's existing entries for the recipe's primary ingredient, to confirm no explicit rule already governs it before merging. Subscription -- the household's current tier, to apply FEAT-12.SPEC-004's gate.
**Creates:** None directly on Rating (this automation is triggered by, but never creates, a Rating).
**Updates:** Dietary Rule -- merges one new entry (member, rule_kind: dislike, strength: soft, allergen/ingredient, origin: learned) when the pattern qualifies and no existing entry governs the ingredient. This is this feature's only write to Dietary Rule, matching the Feature Dependency Map's "update (merge only)" scope for FEAT-12 on this entity.
**Deletes:** None -- this automation never removes a Dietary Rule entry, learned or explicit.

## Business Rules

- **XBR-17:** A meal rated down repeatedly by the same member becomes a learned soft dislike on that member's dietary rules; soft dislikes influence selection but never block a suggestion and never override an explicit rule.
- **Threshold value:** The number of down-ratings that constitutes "repeated" is a platform-wide value, never a concrete number stated in this spec: platform parameter: `repeated-dislike-rating-count`.
- **Never overrides or weakens an explicit rule:** If any explicit (non-learned) Dietary Rule entry already exists for the ingredient -- allergy, religious rule, or an organiser-entered dislike -- this automation takes no action for that ingredient, regardless of how many down-ratings accumulate.
- **A learned entry, once merged, is not automatically retracted by a single reversed rating:** Reversing enough down-ratings to statistically "undo" the pattern does not by itself remove a previously merged learned entry; removal of any Dietary Rule entry, learned or explicit, is owned entirely by FEAT-01 (explicit confirmation) and FEAT-18 (cascade), never by this automation.
- **Gated to the paid tier (XBR-05):** This automation performs no detection or merge at all while the household is on the free tier; ratings still accumulate normally (FEAT-12.SPEC-002) and are available for evaluation the moment the household upgrades.
- **No medical or diet advice is derived (scope-boundaries.md SC-06):** A learned soft dislike is a meal-selection preference signal only; this automation performs no nutrition or health scoring and produces no diagnostic language.
- **Children's-privacy-class protection carries through:** A learned dislike derived from a young kid profile's proxy-recorded ratings is itself children's-privacy-class data, consistent with the same minimal-collection, parent-controlled posture as the ratings that produced it (ASMP-27).
- **Change history is preserved:** Every Dietary Rule entry, including a learned one this automation merges, carries the entity's change_history field (who changed the rule and when), visible to the organiser (Maya), per the Dietary Rule entity's own definition in the Feature Dependency Map.

## Edge Cases

- **A member's ratings for a recipe include some via proxy (young kid) and some self-recorded (adult)** -- Not applicable to the same (member, recipe) count, since a rating's member field identifies exactly one Member Profile; a young kid profile's proxy-recorded ratings and an adult's own ratings for the same recipe are counted as two entirely separate (member, recipe) histories, each evaluated against the threshold independently.
- **A recipe has no single clear "primary ingredient"** -- The recipe's primary ingredient signal is drawn from Recipe data maintained by FEAT-08/FEAT-10 (e.g., the named central protein or vegetable in the recipe's title or ingredient list); if a recipe genuinely has no identifiable primary ingredient, this automation takes no merge action for that recipe rather than guessing at one, and the pattern is not lost -- future evaluation continues to consider that (member, recipe) pair on every subsequent rating.
- **The member is removed from the household between the qualifying rating and this automation's evaluation** -- Per FEAT-18's cascade (XBR-16), the member's Dietary Rule and Rating data is being deleted; this automation takes no action if the member's records no longer exist by the time it runs, since there is nothing left to merge into.
- **Household downgrades between the down-rating that would qualify and this automation's evaluation** -- The tier gate is checked at evaluation time, not at the rating's original submission time; if the household is free tier by the time evaluation runs, no merge occurs (see Outcome Definitions, "No action -- free tier"), even though the qualifying rating itself was submitted while paid.
- **Concurrent trigger firing -- two down-ratings for the same member and recipe (e.g., two different meal instances of the same recipe) are submitted at effectively the same time** -- Each triggering event runs its own evaluation independently; the count each evaluation computes reflects whichever ratings have already committed at the moment it reads history, so the two evaluations may run with slightly different counts, but the merge step (Step 9-10) is idempotent -- whichever evaluation runs second and finds a learned entry already merged takes no duplicate action.
- **Trigger fires while a previous run for the same (member, recipe) pair is still in flight** -- The merge step's "does a learned entry already exist" check (Step 9) makes a second concurrent run safe: if the first run's merge has not yet committed when the second run checks, both may attempt to merge, but the Dietary Rule entity's own one-entry-per-(member, ingredient, origin) shape means the second write is treated as the same idempotent merge, not a duplicate entry.
- **A qualifying pattern exists, but the member has since left the household and their ratings became anonymized influence (FEAT-09, XBR-16)** -- Anonymized ratings are no longer attributable to a specific member, so they cannot feed a new named-member Dietary Rule merge; this automation only evaluates ratings still attributed to an Active member.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-002 (Rating Submission, Change & Proxy Rules) | Triggered by (inbound) | Every qualifying down-rating (or reversal) submission or change fires this automation's evaluation |
| FEAT-12.SPEC-004 (Preference Weighting & Tier-Gating Rule) | References (inbound) | This automation's paid-tier gate is the same condition FEAT-12.SPEC-004 defines; this spec consumes it rather than redefining it |
| FEAT-01 (Household Setup & Member Profiles) | Affects (outbound) | A merged learned soft dislike updates Dietary Rule data owned end-to-end by FEAT-01 |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Affects (outbound) | Plan generation reads the updated Dietary Rule set (including any newly merged learned dislike) on its next run |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | Affects (outbound) | The safety/dietary-badge engine re-evaluates recipes against the household's updated Dietary Rule set, including learned entries, though a soft dislike never blocks a suggestion the way a hard rule does |

## Analytics and Success Signals

- **learned_dislike_applied** (member type: adult / young-kid-by-proxy, recipe reference, ingredient signal) -- N/A -- no success-metrics.md metric names Meal Rating & Preference Learning as its Connected Feature; this event is defined per product-features.md's Signals list for FEAT-12 and is available for a future metric to draw on.
- **repeated_dislike_pattern_evaluated** (outcome: merged / no_action_below_threshold / no_action_explicit_rule_exists / no_action_free_tier / no_action_learned_entry_exists) -- N/A -- same reason as learned_dislike_applied; this event tracks how often each outcome path is exercised for later product review, independent of any currently connected success metric.

## Acceptance Criteria

**FEAT-12.SPEC-005-AC-01:** Given Sam is on the paid tier and his down-ratings for the same recipe have just reached the count defined by platform parameter: `repeated-dislike-rating-count`, with no explicit Dietary Rule entry for that recipe's primary ingredient, when this automation evaluates the pattern, then a new Dietary Rule entry is merged for Sam with rule_kind "dislike", strength "soft", and origin "learned".

**FEAT-12.SPEC-005-AC-02:** Given Sam has rated the same recipe down fewer times than platform parameter: `repeated-dislike-rating-count` on the paid tier, when this automation evaluates the pattern, then no Dietary Rule entry is merged.

**FEAT-12.SPEC-005-AC-03:** Given Maya's household is on the free tier and Maya has rated a recipe down repeatedly, when this automation would otherwise evaluate the pattern, then no evaluation or merge occurs, since the household is not on the paid tier.

**FEAT-12.SPEC-005-AC-04:** Given Jordan (young kid profile) has an explicit allergy entry for an ingredient central to a recipe, when repeated down-ratings recorded by proxy for that recipe reach the threshold on the paid tier, then no learned entry is merged for that ingredient, since the explicit allergy rule already governs it and is never overwritten.

**FEAT-12.SPEC-005-AC-05:** Given a learned soft-dislike entry for a given member and ingredient was already merged by a prior run, when a further qualifying down-rating for the same (member, recipe) pair is evaluated, then no duplicate Dietary Rule entry is created.

**FEAT-12.SPEC-005-AC-06:** Given Sam's down-rating pattern for a recipe met the threshold and a learned entry was merged, when Sam later changes several of those ratings to thumbs-up, then the previously merged learned entry is not automatically removed by this automation.

**FEAT-12.SPEC-005-AC-07:** Given a household upgrades from free to paid, when a prior qualifying down-rating pattern (recorded while free) is next evaluated, then the pattern is assessed using the household's current paid-tier status and can result in a merge if the threshold and no-existing-rule conditions are met.

**FEAT-12.SPEC-005-AC-08:** Given a paid-tier household's recipe has no clearly identifiable primary ingredient, when the down-rating pattern reaches the threshold, then this automation takes no merge action for that recipe rather than guessing at an ingredient.

**FEAT-12.SPEC-005-AC-09:** Given two down-ratings for the same member and recipe (from two different meal instances of that recipe) are submitted at effectively the same time, when both trigger this automation, then each evaluation runs independently and the merge step remains idempotent -- at most one learned Dietary Rule entry results.

**FEAT-12.SPEC-005-AC-10:** Given Maya's household member is removed from the household (FEAT-18) before this automation evaluates a qualifying pattern for that member, then no Dietary Rule merge occurs for that member, since their records are being deleted under XBR-16.

**FEAT-12.SPEC-005-AC-11:** Given a household member leaves voluntarily and their ratings become anonymized influence (FEAT-09, XBR-16), when this automation considers that recipe's rating history, then the anonymized ratings are excluded from any new named-member Dietary Rule merge evaluation.

**FEAT-12.SPEC-005-AC-12:** Given a merge completes for Sam's Dietary Rule entry, when Maya later views Sam's dietary rules through FEAT-01, then the learned entry's change_history shows it was added by this automation and when.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (qualifying down-rating; down-rating reversed) | 2 |
| Outcome Paths | 6 (no action free tier, no action below threshold, no action explicit rule exists, no action learned entry exists, learned soft dislike merged, automation failure) | 6 |
| Business Rules | 8 | 8 |
| Edge Cases | 7 | 7 |
