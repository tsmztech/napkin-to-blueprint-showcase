---
document_type: feature-overview
feature_number: FEAT-12
feature_name: Meal Rating & Preference Learning
feature_slug: meal-rating-preference-learning
priority_tier: Important
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 5
screen_count: 1
automation_count: 1
logic_rule_count: 3
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Meal Rating & Preference Learning

## Summary

**Feature:** Meal Rating & Preference Learning
**ID:** FEAT-12
**Description:** Household members rate meals with a simple thumbs up or down after dinner, and future plans learn from those ratings to better match what the family actually likes.
**Priority:** Important
**Phase:** MVP
**Type:** User-Facing
**Rationale:** The brief states the plan "learns what the family actually likes" over time and depicts kids giving "a thumbs up or down" after dinner (BRIEF.md, Vision, The Experience). Phased to v1 because meaningful learning requires a history of ratings that does not exist in a household's first weeks; MVP plan generation must work well from ratings alone being absent. Rating is available on both tiers, while learning from ratings is a paid-tier benefit (BRIEF.md, Business Context).

**Key Capabilities:**
- Rate a meal — Household member gives a thumbs up or down after a dinner is cooked
- See ratings reflected in future plans — Meals rated poorly by the household appear less often; well-liked meals appear more often
- Rate individually — Each household member's rating is recorded separately, so the plan can learn different preferences per person
- Rate for a young kid — An adult hands their phone round the table and records each young kid profile's thumbs up or down on that kid's behalf

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-12.SPEC-001 | Post-Dinner Rating Prompt | Screen | Maya, Sam | Household member rates a cooked dinner with a thumbs up/down, and can pass their phone to record each young kid profile's rating on that kid's behalf |
| FEAT-12.SPEC-002 | Rating Submission, Change & Proxy Rules | Logic/Rule | Maya, Sam | Governs one-rating-per-member-per-meal, the changeable-until-archived window, the unrated-is-neutral default, and how a proxy rating on behalf of a young kid profile counts as that kid's one rating |
| FEAT-12.SPEC-003 | Rating Access & Authorization Rules | Logic/Rule | All | Governs who may rate for whom (Maya Full, Sam Own-only, either adult as proxy for a young kid, the Later-phase older-kid login Own-only, Riley View, unauthorized visitors None) and the never-shown-individually privacy rule |
| FEAT-12.SPEC-004 | Preference Weighting & Tier-Gating Rule | Logic/Rule | Maya, Sam | Governs how accumulated ratings weight future meal selection (liked meals more often, disliked meals less often) and gates that learning effect — but never rating capture itself — to the paid tier |
| FEAT-12.SPEC-005 | Repeated-Dislike Learned Update | Automation | Maya, Sam | On a pattern of repeated down-ratings from the same member for the same meal, merges a soft dislike into that member's Dietary Rule data, on the paid tier only |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Rate a meal | FEAT-12.SPEC-001 | Primary purpose of the rating prompt screen | Phase 2 (Explicit) |
| See ratings reflected in future plans | FEAT-12.SPEC-004 | Defines the weighting derivation that FEAT-03's generation applies, and the paid-tier gate on that effect | Phase 2 (Explicit) |
| Rate individually | FEAT-12.SPEC-001, FEAT-12.SPEC-002 | Each member's rating is captured and stored against their own Member Profile; SPEC-002 enforces one rating per member per meal | Phase 2 (Explicit) |
| Rate for a young kid | FEAT-12.SPEC-001, FEAT-12.SPEC-002 | The prompt screen's proxy flow, and SPEC-002's rule that a proxy rating counts as that kid's one rating for the meal | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a single Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-12.SPEC-003 | Rating Access & Authorization Rules | Phase 5 (Rule-Constraint Discovery — authorization) | The Access field names five distinct role behaviors (Maya Full, Sam Own-only, proxy-for-young-kid, Later older-kid Own-only, Riley View) plus a privacy rule (no member sees another's individual rating); this is shared by SPEC-001 and referenced by diagnosis access outside this feature, well past the inline threshold |
| FEAT-12.SPEC-004 | Preference Weighting & Tier-Gating Rule | Phase 5 (Rule-Constraint Discovery — conditional logic) + Phase 4 (Trigger-Response, entity-update propagation) | XBR-05 conditions the entire "future plans learn" capability on Subscription tier while rating capture itself stays unconditioned — a rule with cross-feature consequence (FEAT-03) that needed to be stated once rather than duplicated in every consuming automation |
| FEAT-12.SPEC-005 | Repeated-Dislike Learned Update | Phase 4 (Trigger-Response — cascading update to a related entity) | The "Repeated dislike" alternate flow and XBR-17 describe a system side-effect on Rating creation that writes to a different entity (Dietary Rule) with non-trivial merge logic (never overriding an explicit rule) — a cross-entity effect, not a simple data write |

## Entity-Lifecycle Coverage Matrix

**Entity: Rating**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-12.SPEC-001 | Household member submits a thumbs up/down on the prompt screen for themselves, or as a proxy for a young kid profile | Governed by FEAT-12.SPEC-002 (uniqueness) and FEAT-12.SPEC-003 (who may submit for whom) |
| Read (single) | FEAT-12.SPEC-001 | The prompt screen loads any existing rating for this meal/member pair to show the current state (already rated, and what value) before allowing a change | -- |
| Read (list) | N/A | No screen in this feature lists ratings; per Data Notes, an individual member's ratings are never displayed broken out, even to that member. Aggregate reads for weighting happen inside FEAT-03's generation (cross-feature, not owned by this feature) | -- |
| Update | FEAT-12.SPEC-001 | Household member changes a previously submitted rating; governed by FEAT-12.SPEC-002's changeable-until-archived window | -- |
| Delete/Archive | N/A -- owned by FEAT-18 and FEAT-09 | Hard delete on member removal or household deletion (FEAT-18, within 30 days per XBR-16); on a member's own voluntary departure, ratings are retained but anonymized into influence rather than deleted (FEAT-09, XBR-16). This feature never deletes or archives a Rating itself; it only creates and updates values | Cross-feature; no gap -- ownership is explicit in the dependency map |
| State Transition | N/A | A Rating has only a value (up/down) and no formal state machine beyond that value; value changes are covered under Update | -- |

**Entity: Dietary Rule (this feature's scope: Update only, per Connected Entities)**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A -- owned by FEAT-01 | Explicit dietary rules (allergy, religious rule, vegetarian setting, explicit dislike) are created only through Household Setup & Member Profiles | Stage 2's Connected Entities line scopes this feature to "update" only; FEAT-12 never originates a Dietary Rule record from nothing |
| Read (single) | FEAT-12.SPEC-005 | Before merging a learned soft dislike, the automation reads the member's existing Dietary Rule entries for that ingredient to confirm no explicit rule already governs it | Inline within the automation, not a standalone read spec |
| Read (list) | N/A | This feature reads only the specific ingredient's existing rule entries at merge time (above); it does not list or display a member's full Dietary Rule set anywhere | Full rule listing/editing belongs to FEAT-01 |
| Update | FEAT-12.SPEC-005 | Merges a new soft-dislike entry (origin: learned) after a repeated down-rating pattern; never overwrites or weakens an explicit rule for the same ingredient (XBR-17) | Paid-tier only, per FEAT-12.SPEC-004's gate |
| Delete/Archive | N/A -- owned by FEAT-01 (explicit confirmation for allergies) and FEAT-18 (cascade) | This feature never removes a Dietary Rule entry, learned or explicit | Cross-feature; no gap |
| State Transition | N/A | Dietary Rule entries carry a strength (hard/soft) and origin (entered/learned) set at creation or merge time, not a transition sequence this feature manages | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Planned Meal | FEAT-12.SPEC-001 | The prompt screen identifies which cooked dinner (or manually planned meal, FEAT-23) a rating attaches to |
| Member Profile | FEAT-12.SPEC-001, FEAT-12.SPEC-003 | The screen and authorization rule identify which household member (adult, or young kid profile) a rating is being recorded for and by |
| Subscription | FEAT-12.SPEC-004 | The weighting/learning gate reads the household's tier to determine whether ratings currently influence plan generation |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Household member submits a thumbs up/down for themselves | Rating recorded against that member and meal; confirms instantly | Inline in triggering screen | FEAT-12.SPEC-001 |
| An adult records a thumbs up/down on behalf of a young kid profile | Rating recorded against that kid profile, tagged with the recording adult, and counted as that kid's one rating for the meal | Standalone Logic/Rule | FEAT-12.SPEC-002 |
| Rating submission fails | Submission retries automatically in the background; no error is surfaced unless retries exhaust | Inline in triggering screen (per Stage 2 Error state) | FEAT-12.SPEC-001 |
| A rating is given while offline | Held locally on the device and synced once connectivity returns, using the existing real-time synchronization boundary | Cross-feature — relies on FEAT-03.SPEC-011 / FEAT-06.SPEC-005's synchronization capability; no new Integration spec is invented per the context package's External Touchpoints instruction | FEAT-03 / FEAT-06 responsibility |
| A meal goes unrated | Treated as neutral; never assumed liked or disliked; the household is never prompted or pressed to rate | Standalone Logic/Rule | FEAT-12.SPEC-002 |
| Household member attempts to change a previously submitted rating | Allowed while the plan holding that meal is not yet archived; the prior value is replaced | Standalone Logic/Rule | FEAT-12.SPEC-002 |
| The same member rates the same meal down repeatedly across occurrences | Evaluated against the repeated-dislike threshold; on a paid-tier household, merges a soft dislike into that member's Dietary Rule data | Standalone Automation | FEAT-12.SPEC-005 |
| A rating is recorded on a free-tier household | Stored normally, but does not influence future plan weighting and does not feed the repeated-dislike automation until the household upgrades | Standalone Logic/Rule | FEAT-12.SPEC-004 |
| Someone without rating access for that member attempts to rate (an unauthorized visitor, or a role acting outside its own scope) | Action is blocked or the control is not shown | Standalone Logic/Rule | FEAT-12.SPEC-003 |
| AI Weekly Dinner Plan Generation runs on a paid-tier household | Reads accumulated ratings and applies the weighting this feature defines (liked meals more often, disliked meals less often) | Cross-feature — owned by FEAT-03 (FEAT-03.SPEC-010); this feature only defines the weighting derivation rule | FEAT-12.SPEC-004 / FEAT-03.SPEC-010 responsibility |

## Shared Context

**Shared Entities:**
- Rating — created and updated by FEAT-12.SPEC-001, governed by FEAT-12.SPEC-002 (submission/change constraints) and FEAT-12.SPEC-003 (who may submit), consumed by FEAT-12.SPEC-004 (weighting) and FEAT-12.SPEC-005 (learned-dislike detection). Fields: member, planned_meal, value (up/down), recorded_by (the adult who recorded a young kid's rating, where applicable).
- Dietary Rule — read and updated (merge only) by FEAT-12.SPEC-005, gated by FEAT-12.SPEC-004; owned end-to-end by FEAT-01. Relevant fields for this feature: member, rule_kind (dislike), strength (soft), allergen/ingredient, origin (learned).
- Subscription — read by FEAT-12.SPEC-004 to determine whether the learning effect (weighting + learned dislikes) currently applies to this household; owned end-to-end by FEAT-14.

**Shared UI Patterns:**
- Proxy rating pattern — FEAT-12.SPEC-001 presents the same thumbs up/down control for an adult's own rating and for each young kid profile in turn (phone-passed sequence); Spec Writers should describe this as one repeating control, not separate screens per profile.

**Shared Validation:**
- FEAT-12.SPEC-002 defines the one-rating-per-member-per-meal, change-window, and proxy-counts-as-the-kid's-rating rules; FEAT-12.SPEC-001 references SPEC-002 rather than restating them.
- FEAT-12.SPEC-003 defines who may act on whose behalf; FEAT-12.SPEC-001 and FEAT-12.SPEC-002's proxy handling both reference SPEC-003 rather than duplicating the access logic.

## Internal Dependency Map

```
FEAT-12.SPEC-001 (Post-Dinner Rating Prompt) -> [validates submission using] -> FEAT-12.SPEC-002 (Rating Submission, Change & Proxy Rules)
FEAT-12.SPEC-001 (Post-Dinner Rating Prompt) -> [checks who may rate for whom using] -> FEAT-12.SPEC-003 (Rating Access & Authorization Rules)
FEAT-12.SPEC-002 (Rating Submission, Change & Proxy Rules) -> [each qualifying down-rating evaluated by] -> FEAT-12.SPEC-005 (Repeated-Dislike Learned Update)
FEAT-12.SPEC-004 (Preference Weighting & Tier-Gating Rule) -> [gates] -> FEAT-12.SPEC-005 (Repeated-Dislike Learned Update)
FEAT-12.SPEC-001 (Post-Dinner Rating Prompt) -> [accumulated ratings read and weighted by, per] -> FEAT-12.SPEC-004 (Preference Weighting & Tier-Gating Rule)
```

**Default Entry:** N/A -- this feature has no top-level navigation destination of its own. FEAT-12.SPEC-001 (Post-Dinner Rating Prompt) is reached only from FEAT-03's week plan, via the "Rate after dinner" trigger on a cooked meal (Navigation Connections).

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-12.SPEC-001 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | The rating prompt appears on a cooked meal within the week plan | Household member views a cooked meal after dinner |
| FEAT-12.SPEC-001, FEAT-12.SPEC-002 | Inbound | FEAT-23 (Manual Weekly Planning) | Ratings on a manually planned week's meals count the same as ratings on an AI-generated week | Household rates a meal that came from manual planning |
| FEAT-12.SPEC-004 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | The weighting derivation this rule defines is applied inside FEAT-03's generation (FEAT-03.SPEC-010) | Weekly plan generation runs on a paid-tier household |
| FEAT-12.SPEC-004 | Inbound | FEAT-14 (Subscription & Billing Management) | The learning effect (weighting and learned dislikes) is gated to the paid tier per XBR-05; rating capture itself is never gated | Household's Subscription tier changes (upgrade, downgrade, lapsed payment) |
| FEAT-12.SPEC-005 | Outbound | FEAT-01 (Household Setup & Member Profiles) | A repeated down-rating pattern merges a learned soft dislike into Dietary Rule data owned by FEAT-01 | Same member rates the same meal down repeatedly |
| FEAT-12.SPEC-001 (Rating entity) | Outbound | FEAT-09 (Household Invitations & Membership) | A member's ratings become anonymous influence rather than being deleted when that member leaves voluntarily (XBR-16) | Member leaves the household |
| FEAT-12.SPEC-001 (Rating entity) | Outbound | FEAT-18 (Account & Data Management) | Ratings are deleted on member removal or household deletion, within 30 days for household deletion (XBR-16) | Organiser removes a member, or deletes the household |
| FEAT-12.SPEC-001 | Inbound | FEAT-22 (Operator Support & Diagnosis) | Riley's View access lets support see rating activity for diagnosis, never individual rating values broken out beyond what aggregate diagnosis requires | Support investigates an open household request |

## Non-Functional Notes

**Data volumes / growth:** One rating per household member (including each young kid profile, recorded by proxy) per Planned Meal, across several thousand households in the first year with 2-6 members each (ASMP-24); ratings are kept for the life of the household account, remaining available and browsable even after a downgrade to the free tier (scope-boundaries.md SC-18), except where XBR-16 requires deletion or anonymization on membership change.

**Responsiveness:** Submitting a rating confirms instantly, with no perceptible wait (Stage 2 States: Loading); a failed submission retries automatically in the background without blocking the household member from continuing (Stage 2 States: Error).

**Data sensitivity / privacy:** Ratings are personal preference data, including children's preferences captured by proxy for a young kid profile — individual ratings are never shown broken out to any other household member, only the aggregate effect on future plans (Data Notes; ASMP-26); used only for the household's own plan, never sold or used for advertising (ASMP-26). A young kid's rating is children's data under children's-privacy-class protection, alongside the resulting learned Dietary Rule entry (ASMP-27).

**Compliance flags:** Children's-privacy-class protections apply to every rating recorded on behalf of a young kid profile, and to any learned dislike derived from it, consistent with minimal collection and parent-controlled data (ASMP-27); no medical or diet advice is derived from rating data (scope-boundaries.md SC-06) — a learned soft dislike is a preference signal for meal selection, never a health or nutrition judgment.

**Analytics signals:** meal_rated_up, meal_rated_down, rating_changed, learned_dislike_applied, kid_rating_recorded_by_adult (product-features.md Signals) — fired by FEAT-12.SPEC-001 (the first three and the fifth) and FEAT-12.SPEC-005 (learned_dislike_applied).

## Non-Goals

- **Independent kid login for rating** — Excluded per scope-boundaries.md (SC-02): v1's default is parent-managed profiles with no login for young kids, so every young-kid rating in v1 is recorded by an adult on the kid's behalf (FEAT-12.SPEC-001, FEAT-12.SPEC-002); an independent kid-rating login is the distinct, Later-phase possibility already tracked as FEAT-17.
- **Nutrition or health scoring derived from ratings** — Excluded per scope-boundaries.md (SC-06): the brief states directly there is no medical or diet advice, so a learned dislike (FEAT-12.SPEC-005) is used only to weight future meal selection, never to produce a health score or dietary judgment.
- **Showing another member's individual ratings** — Excluded per the feature's own Data Notes and Access fields (a named Stage 2 decision): no one, including Maya, sees another household member's individual rating broken out; only the aggregate effect on future plans is ever visible. This is a permanent product decision, not a missing screen.
- **Deleting a rating outright** — Excluded per the feature's own Validation & Limits field (a named Stage 2 decision): a rating can be changed to the opposite value up until the plan is archived, but the product gives no capability to remove a rating entirely, leaving the meal unrated again. Only FEAT-18's member-removal/household-deletion path and FEAT-09's leave-anonymization path ever remove or alter a rating's standing outside the household member's own change.
- **Reminders or pressure to rate an unrated meal** — Excluded per the feature's own States field (a named Stage 2 decision): an unrated meal is treated as neutral and the household is never prompted or pressed to go back and rate it; rating stays entirely opt-in and self-initiated, consistent with the Communications field's "no notifications of its own."
