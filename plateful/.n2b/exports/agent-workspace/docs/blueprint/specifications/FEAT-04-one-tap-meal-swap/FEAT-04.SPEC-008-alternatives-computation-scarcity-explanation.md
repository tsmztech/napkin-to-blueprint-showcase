---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-04.SPEC-008
spec_name: Alternatives Computation & Scarcity Explanation
spec_slug: alternatives-computation-scarcity-explanation
parent_feature: FEAT-04
parent_feature_name: One-Tap Meal Swap
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 17
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Alternatives Computation & Scarcity Explanation

## Overview

**Name:** Alternatives Computation & Scarcity Explanation
**ID:** FEAT-04.SPEC-008
**Type:** Logic/Rule
**Purpose:** Routes alternative sourcing by plan origin, applies the safety/schedule filter every candidate must pass, and determines the limited- or zero-alternatives explanation shown when few or no options qualify.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap
**Governed Entity:** Planned Meal (the slot being swapped, and the candidate set that can legally fill its recipe field)

## Scope and Non-Goals

**In Scope:**
- Routing an alternatives request to the AI text/plan-generation capability (FEAT-04.SPEC-007) or a direct recipe-library filter, based on the target Weekly Plan's origin
- Applying the allergy/religious hard-rule safety check (via FEAT-02) and the schedule-fit filter to every candidate before it can be shown
- Determining and wording the scarcity explanation when the qualifying set is small or empty
- Authorization for who may request alternatives for a slot

**Non-Goals:**
- Generating raw AI candidates -- owned by FEAT-04.SPEC-007; this spec only filters and routes
- Performing the safety determination itself -- owned by FEAT-02 (Dietary Rules & Allergy Safety Engine); this spec invokes that capability rather than re-implementing it
- Writing the chosen alternative onto the Planned Meal -- owned by FEAT-04.SPEC-004 (Apply Meal Swap); this spec only produces the candidate set a user chooses from
- Enforcing the one-active-swap-per-slot concurrency limit -- owned by FEAT-04.SPEC-009 (Swap Concurrency Lock), a distinct rule set applied after a candidate is chosen

## Governed Entity

**Entity:** Planned Meal
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| night | text/enum | The day of the week this slot occupies (at most one dinner per night) |
| meal_kind | enum | Dinner, or leftover lunch linked to a source dinner |
| recipe | reference | The chosen Recipe -- the field this spec's alternatives set constrains at swap time |
| safety_badge | derived | "Checked against allergies" plus the "always check labels" disclaimer |
| vegetarian_option | boolean | Whether a shared meal carries a vegetarian variant |
| cook_time | derived | Carried from the candidate recipe, sized for the household |
| rough_cost | derived | Carried from the candidate recipe, sized for the household |
| pantry_callout | derived | Which logged pantry items this dinner uses |
| status | enum | Proposed/Picked, Confirmed, Swapped, Removed (safety), Cooked |
| swap_history | list | Prior recipes in this slot |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-04.SPEC-001 | Meal Swap Direct | On tapping Swap; before displaying the alternatives list |
| FEAT-04.SPEC-002 | Suggest a Swap | On tapping Suggest a swap; before displaying the alternatives list |
| FEAT-04.SPEC-007 | Swap Alternatives Generation | Supplies raw candidates for AI-originated plans, which this spec then filters |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| recipe (candidate set) | Must pass the same allergy/religious hard-rule check as original plan generation (XBR-01) | Always, for every candidate before it can appear in the alternatives list | On alternatives request | N/A -- failing candidates are silently excluded, never shown with an error; if the excluded set leaves too few options, the Scarcity Explanation rules below produce the user-facing message | Yes |
| recipe (candidate set) | Cook time must be at or under the night's stated time limit | Only when the household's weekly_schedule marks this night as time-constrained | On alternatives request | N/A -- excluded silently, same as above | Yes |
| recipe (candidate set) | Must not duplicate a recipe already planned elsewhere in the current week | Always | On alternatives request | N/A -- excluded silently, same as above (prevents a swap from creating an unintended repeat within the same week) | No -- a soft preference, not a hard exclusion; see Business Rules |
| night, meal_kind, vegetarian_option, pantry_callout, status, swap_history, safety_badge, cook_time, rough_cost | No validation beyond data type -- these fields are read by this spec to build the request context (e.g., night's schedule limit) or displayed alongside a candidate but are not themselves subject to a rule this spec defines | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Safety and schedule filters combine | recipe (safety), recipe (cook_time vs. night's schedule limit) | A candidate must pass both the safety check and the schedule-fit check to qualify; failing either excludes it from the qualifying set | N/A -- exclusion is silent; combined failure is reflected only in a lower qualifying count feeding the Scarcity Explanation |
| Origin determines sourcing | night (via the slot's parent Weekly Plan -- origin), recipe (candidate source) | If the parent Weekly Plan's origin is AI-generated, candidates are requested from FEAT-04.SPEC-007; if manually-built, candidates are drawn directly from the household's recipe library (starter and imported recipes, FEAT-08/FEAT-10) with no AI request, keeping free-tier households at zero AI cost (scope-boundaries.md SC-16) | N/A -- routing is internal and produces no user-facing message of its own |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Request alternatives for a slot (to apply directly) | Maya (Organiser) | Always | -- |
| Request alternatives for a slot (to submit a suggestion) | Sam (Other Adult Member) | Always (Own-only: the resulting candidate can only become his own suggestion, never a direct write) | -- |
| Request alternatives for a slot (to apply directly) | Sam (Other Adult Member) | Never | The Swap affordance on the Weekly Plan routes Sam to FEAT-04.SPEC-002 (Suggest a Swap) instead of the direct-apply screen; a direct-apply request from Sam is not offered anywhere in the product |
| Request alternatives for a slot | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role; no request path is reachable |
| Request alternatives for a slot | Jordan (older kid, limited login -- Later) | Never | Meal Swap access is None for this role (Access Matrix); the swap affordance is not shown |
| Request alternatives for a slot | Riley (Operator, support) | Never | Meal Swap access is None for this role; not shown in the support read-only view |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Qualifying alternatives set | Candidates from FEAT-04.SPEC-007 (AI-originated) or the recipe library (manually-built), filtered by the safety and schedule cross-field rule above, minus recipes already planned elsewhere this week where a non-duplicate alternative exists | Every time an alternatives request is made | No -- the filter itself is not user-configurable; the user only chooses among what qualifies |
| Scarcity explanation text | See Business Rules below for the exact derivation logic | Whenever the qualifying set is smaller than a comfortable browsing size or empty | No |

## Business Rules

- **Scarcity thresholds and wording:** When the qualifying set has 3 or more candidates, no scarcity explanation is shown -- the list speaks for itself. When it has 1-2 candidates, the explanation "Only {N} option{s} fit{s} tonight's {constraint description}." appears below the list (e.g., "Only 2 options fit tonight's 30-minute limit and everyone's dietary rules."), naming whichever constraint(s) actually narrowed the set (schedule, safety, or both). When it has 0 candidates, the message "No safe alternatives fit tonight -- every recipe that qualifies is already in this week's plan or fails someone's allergy rule." is shown alone, with no selectable items, naming the same narrowing reasons in plain terms.
- **Duplicate-avoidance is a soft preference, not a hard exclusion:** if applying the duplicate-avoidance rule would leave zero candidates, previously-planned recipes are added back into the qualifying set (still subject to the hard safety and schedule filters) so the household is never shown zero alternatives purely because every safe, time-fitting recipe happens to already be planned this week; the scarcity explanation notes "including a repeat from earlier this week" in that case.
- **XBR-01 fail-closed:** a candidate with incomplete ingredient data is excluded, never shown unchecked -- this applies identically to AI-generated candidates and recipe-library candidates.
- **Origin routing is fixed at request time:** a plan's origin (AI-generated or manually-built) is read from its Weekly Plan record at the moment of the request; a household that upgrades or downgrades mid-week uses whichever routing its current plan's origin dictates for any swap attempted after the change (see FEAT-04.SPEC-007, Edge Cases).
- **Mid-week rule tightening reruns this filter, not just original generation:** when a hard dietary rule changes mid-week (XBR-02), any Planned Meal the re-check flags is opened for swap through FEAT-04.SPEC-001, and this spec's filter runs exactly as it would for a voluntary swap -- there is no separate rule set for a re-check-triggered swap.

## Edge Cases

- **Exactly 3 candidates qualify** -- No scarcity explanation is shown (the threshold is "3 or more"); this is the boundary between the explanation and no-explanation states.
- **Exactly 2 candidates qualify** -- The "Only 2 options..." explanation is shown; this is the boundary between the 1-2 wording and the 3-or-more silence.
- **Every recipe that passes safety and schedule is already planned this week (0 candidates before the duplicate-avoidance override)** -- The soft-preference override adds previously-planned recipes back in, per Business Rules; the household is never left with zero alternatives solely due to duplicate-avoidance when at least one safe, time-fitting recipe exists anywhere in its pool.
- **A night carries no schedule constraint at all** -- The schedule-fit filter contributes no exclusions for that night; only the safety filter narrows the set, and the scarcity explanation (if any) names only the safety constraint.
- **Sam requests alternatives for a suggestion and Maya requests alternatives for the same slot at the same time (from different screens)** -- Each request is evaluated independently against the same current plan state; both may see the same or a slightly different qualifying set depending on timing, but neither request blocks the other, since no write occurs until a candidate is chosen (concurrency is handled downstream by FEAT-04.SPEC-009 at the point of selection, not here).
- **A candidate recipe has incomplete ingredient data** -- Excluded from the qualifying set regardless of source (fail-closed, XBR-01); it never appears even as part of a scarcity explanation's count.

## Acceptance Criteria

**FEAT-04.SPEC-008-AC-01:** Given Maya requests alternatives for a slot on an AI-originated plan, when this spec routes the request, then it is sent to FEAT-04.SPEC-007 rather than the recipe library.

**FEAT-04.SPEC-008-AC-02:** Given Maya requests alternatives for a slot on a manually-built plan, when this spec routes the request, then candidates are drawn directly from the recipe library with no AI request made.

**FEAT-04.SPEC-008-AC-03:** Given a candidate recipe fails the allergy/religious hard-rule check, when the filter runs, then that candidate is silently excluded from the qualifying set.

**FEAT-04.SPEC-008-AC-04:** Given a candidate recipe's cook time exceeds a time-constrained night's limit, when the filter runs, then that candidate is silently excluded.

**FEAT-04.SPEC-008-AC-05:** Given exactly 3 candidates qualify, when the alternatives list renders, then no scarcity explanation is shown.

**FEAT-04.SPEC-008-AC-06:** Given exactly 2 candidates qualify, when the alternatives list renders, then the explanation "Only 2 options fit tonight's {constraint}." appears below the list.

**FEAT-04.SPEC-008-AC-07:** Given zero candidates qualify and at least one safe, time-fitting recipe exists but is already planned this week, when the filter completes, then that recipe is added back into the qualifying set and the explanation notes "including a repeat from earlier this week."

**FEAT-04.SPEC-008-AC-08:** Given zero candidates qualify even after the duplicate-avoidance override, when the filter completes, then the message "No safe alternatives fit tonight -- every recipe that qualifies is already in this week's plan or fails someone's allergy rule." is shown alone with no selectable items.

**FEAT-04.SPEC-008-AC-09:** Given Sam requests alternatives to suggest a swap, when the request is made, then it is allowed (Own-only) and produces the same filtered candidate set logic as Maya's request for the same slot.

**FEAT-04.SPEC-008-AC-10:** Given Sam attempts to reach a direct-apply alternatives request, when he looks for that path, then it does not exist -- his swap tap routes to FEAT-04.SPEC-002 instead.

**FEAT-04.SPEC-008-AC-11:** Given Riley (Operator) has no Meal Swap access, when any alternatives-request path is examined for his role, then none exists.

**FEAT-04.SPEC-008-AC-12:** Given a candidate has incomplete ingredient data, when the filter runs, then that candidate is excluded regardless of its source (AI-generated or recipe-library).

**FEAT-04.SPEC-008-AC-13:** Given a mid-week hard rule change flags a previously-safe Planned Meal (XBR-02), when Maya opens FEAT-04.SPEC-001 for that slot, then this spec's filter runs identically to a voluntary swap request.

**FEAT-04.SPEC-008-AC-14:** Given a night carries no schedule constraint, when the filter runs, then only the safety check narrows the candidate set, and any scarcity explanation names only the safety constraint.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
