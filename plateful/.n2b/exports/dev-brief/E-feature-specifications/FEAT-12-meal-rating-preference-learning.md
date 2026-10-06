# FEAT-12 — Meal Rating & Preference Learning

This chapter covers FEAT-12, Meal Rating & Preference Learning, a Important-tier feature. It contains 5 specifications carrying 63 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-12.SPEC-001 | Post-Dinner Rating Prompt | screen | 13 |
| FEAT-12.SPEC-002 | Rating Submission, Change & Proxy Rules | logic-rule | 13 |
| FEAT-12.SPEC-003 | Rating Access & Authorization Rules | logic-rule | 14 |
| FEAT-12.SPEC-004 | Preference Weighting & Tier-Gating Rule | logic-rule | 11 |
| FEAT-12.SPEC-005 | Repeated-Dislike Learned Update | automation | 12 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Post-Dinner Rating Prompt

## Overview

**Name:** Post-Dinner Rating Prompt
**ID:** FEAT-12.SPEC-001
**Type:** Screen
**Purpose:** A household member gives a thumbs up or thumbs down on a cooked dinner, and can pass their phone to record each young kid profile's rating on that kid's behalf, in one repeating control.
**Parent Feature:** FEAT-12 -- Meal Rating & Preference Learning

## Scope and Non-Goals

**In Scope:**
- Capturing a household member's own thumbs up/down rating for a cooked Planned Meal
- The phone-passed proxy sequence: after the signed-in adult rates, they can step through each young kid profile in the household and record that kid's thumbs up/down on the same control
- Showing the screen's own current rating state (already rated, and what value) before allowing a change
- Letting the signed-in adult change their own already-submitted rating, or a proxy rating they recorded, within this same screen

**Non-Goals:**
- An independent login for a kid to rate directly -- excluded per scope-boundaries.md (SC-02): v1's default is parent-managed profiles with no login for young kids, so every young-kid rating in v1 is recorded by an adult on the kid's behalf; a distinct kid-rating login is the Later-phase FEAT-17.
- Displaying another household member's individual rating, broken out, to anyone (including Maya) -- excluded per the feature's own Data Notes and Access fields: only the aggregate effect on future plans is ever visible, never a per-member rating list.
- Deriving or showing how ratings influence future plan weighting -- governed by FEAT-12.SPEC-004 (Preference Weighting & Tier-Gating Rule); this screen only captures the rating.
- Deleting a rating outright, leaving the meal unrated again -- excluded per the feature's own Validation & Limits field: only a change to the opposite value is supported, not removal.
- Reminding or nudging the household to go back and rate an unrated meal -- excluded per the feature's own States field: rating is opt-in and self-initiated, and the household is never pressed to rate.
- The uniqueness, change-window, and proxy-counting rules themselves -- governed by FEAT-12.SPEC-002; this screen enforces them but does not restate the logic.
- Who may rate for whom -- governed by FEAT-12.SPEC-003; this screen enforces it but does not restate the logic.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-03 (AI Weekly Dinner Plan Generation), week plan | Household member taps "Rate after dinner" on a cooked meal | The Planned Meal reference (which dinner, which night) and the signed-in member's identity |
| FEAT-23 (Manual Weekly Planning), week plan | Household member taps "Rate after dinner" on a cooked meal from a manually planned week | Same as above -- a rating on a manually planned meal is captured identically |

There is no default entry and no top-level navigation destination for this screen (feature-overview.md, Internal Dependency Map): it is reached only from a cooked meal in either week-plan context.

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Rate herself; record a proxy rating for any young kid profile in the household | -- |
| Sam (Other Adult Member) | Full screen, for his own rating and any young kid profile's proxy rating | Rate himself (own rating only, per FEAT-12.SPEC-003 Own-only); record a proxy rating for any young kid profile | Attempting to view or change another adult's individual rating is not possible from this screen -- there is no control for it (privacy rule, FEAT-12.SPEC-003) |
| Jordan (young kid profile, no login -- MVP) | Not applicable -- no login exists, so this profile never opens this screen directly; its rating is entered by an adult through the proxy sequence | Not applicable | Not applicable -- there is no sign-in path for this profile |
| Jordan (older kid, limited login -- Later) | Full screen, for his own rating only | Rate himself (Own-only, per the Access Matrix) | Attempting to record a rating for anyone else is not possible -- the proxy sequence is not offered to this role |
| Riley (Operator, support) | Not applicable to this screen | Not applicable | Riley's Ratings: View access (user-persona.md Access Matrix) is exercised through FEAT-22 (Operator Read-Only Support Access), which surfaces rating activity in aggregate for diagnosis; Riley never opens this per-meal household screen |
| Unauthenticated | No | No | Redirected to the household sign-in screen; after signing in, the user lands on the current week's plan (FEAT-03 or FEAT-23), not directly back on this prompt -- they must tap "Rate after dinner" again |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any rating already captured before expiry (including proxy ratings recorded earlier in the sequence) was already submitted and is not lost; only the not-yet-submitted step in progress is discarded |

## Layout and Content

**Header:** The Planned Meal's recipe name and night (e.g., "Tuesday: Lemon Chicken Traybake") with a close control (top-left) that returns to the week plan. No "Save" action -- every tap on a thumbs control submits immediately.

**Body, own-rating step:** A single large thumbs-up control and a single large thumbs-down control, side by side, centered. Below them, when this member has already rated this meal, the previously chosen control is shown in its selected visual state, and the text "You rated this: {up/down}. Tap to change." appears beneath the pair. When no rating exists yet, no such text appears.

**Body, proxy step (each young kid profile in turn):** The same thumbs-up/thumbs-down pair, with the header line above it replaced by "Rating for {kid display name}" and, on first entry to this step, a one-line instruction "Hand the phone to {kid display name}." When a young kid profile already has a rating recorded for this meal, the previously chosen control is shown selected with "Rated: {up/down}. Tap to change." beneath it, matching the own-rating step's pattern.

**Footer:** A row of small step indicators (one dot per household member being rated: the signed-in adult, then each young kid profile) showing progress through the sequence. A "Next" control advances to the next profile once the current step has a rating (or is skipped); "Done" replaces "Next" on the final step and returns to the week plan (FEAT-03 or FEAT-23, whichever supplied the meal).

This is one repeating control across profiles, not separate screens per profile, per the Feature Breakdown Brief's Shared UI Patterns.

### Responsive Behavior

- **Compact breakpoint:** Thumbs-up/thumbs-down pair centered, full width, large tap targets stacked with generous spacing for one-handed use at the table. Step indicators and Next/Done sit in the footer as described above.
- **Medium size class and above:** Same structure, centered content capped at a consistent platform-wide content width (exact value is the design layer's decision); no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Close control | Tap | Navigate back to the source week plan (FEAT-03 or FEAT-23) without requiring a rating | Screen closes | Any ratings already submitted in this session are kept; unrated steps remain unrated |
| Thumbs-up control (own step) | Tap | Submits a thumbs-up rating for the signed-in member and this meal, per FEAT-12.SPEC-002 (submission/change rules) and FEAT-12.SPEC-003 (authorization) | Control shows selected state instantly | Instant confirmation; no loading spinner (Stage 2 Loading: submitting a rating confirms instantly) |
| Thumbs-down control (own step) | Tap | Submits a thumbs-down rating for the signed-in member and this meal, per FEAT-12.SPEC-002 and FEAT-12.SPEC-003 | Control shows selected state instantly | Instant confirmation |
| Thumbs-up or thumbs-down control (own step, already rated) | Tap the opposite control | Changes the existing rating to the new value, per FEAT-12.SPEC-002's changeable-until-archived window | Previously selected control deselects; newly tapped control selects | Instant confirmation; "Tap to change" text updates to reflect the new value |
| Thumbs-up or thumbs-down control (proxy step) | Tap | Records a proxy rating for the current young kid profile, tagged with the recording adult, per FEAT-12.SPEC-002 (proxy-counts-as-the-kid's-rating) and FEAT-12.SPEC-003 (who may proxy) | Control shows selected state for that profile | Instant confirmation |
| Next control | Tap | Advances to the next profile in the sequence (or to Done on the final profile) | Body content switches to the next profile's step | Step indicator advances; instruction line updates to the next profile's name |
| Next control (a profile's step has no rating yet) | Tap | Skips this profile without recording a rating -- the meal remains neutral for that profile (feature-overview.md, unrated-is-neutral) | Body content switches to the next profile's step | Step indicator advances; no confirmation needed since nothing was recorded |
| Done control (final step) | Tap | Returns to the source week plan (FEAT-03 or FEAT-23) | Screen closes | No further confirmation -- ratings were already confirmed individually as each was tapped |

### Accessibility Notes

- **Focus order:** Close control -> current step's instruction/heading text -> thumbs-up control -> thumbs-down control -> Next/Done control.
- **Selection announcements:** When a thumbs control is tapped, the resulting selected/confirmed state is announced to assistive technology (e.g., "Rated thumbs up" or, on the proxy step, "Rated thumbs up for {kid name}").
- **Step change announcements:** Advancing to a new profile's step announces the new heading ("Rating for {kid name}") so a screen-reader user knows whose rating they are now recording.
- **Keyboard alternatives:** Every control on this screen (thumbs-up, thumbs-down, Next, Done, Close) is reachable and operable by keyboard; there are no pointer-only gestures (no swipe-to-advance).

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Not Yet Rated (default, own step) | Both thumbs controls unselected, no "Tap to change" text | Screen opens for a meal/member pair with no existing rating | Member taps a thumbs control |
| Already Rated | The previously chosen control shown selected, "Tap to change" text visible | Screen opens (or a proxy step loads) for a meal/member pair that already has a rating | Member taps the opposite control to change it, or advances/closes without changing it |
| Submitting | Selected control briefly shows a confirmed visual state | A thumbs control is tapped | Confirmation completes (near-instant; no visible loading state per Stage 2 Loading) |
| Error | A small inline notice near the tapped control: "Couldn't save your rating -- we'll keep trying." The selected state is shown optimistically while retry continues in the background | A rating submission fails | Background retry succeeds (notice clears silently) or the member navigates away (retry continues; see Edge Cases) |
| Offline/Degraded | The tapped control shows its selected state immediately; a small banner reads "You're offline -- this rating will save when you reconnect." | Connectivity is lost while this screen is open, or a rating is submitted while already offline | Connectivity restored -- the queued rating syncs automatically using the real-time synchronization boundary this feature relies on (FEAT-03.SPEC-011 / FEAT-06.SPEC-005) |
| Proxy Sequence Mid-Flow | Step indicator shows progress (e.g., dot 2 of 4 highlighted); instruction line names the current profile | Adult taps Next past the own-rating step | Adult reaches the final profile and taps Done, or closes the screen early |

## Validation Rules

Validation governed by FEAT-12.SPEC-002 (Rating Submission, Change & Proxy Rules) -- one rating per member per meal, the changeable-until-archived window, and the proxy-counts-as-the-kid's-rating rule. Authorization -- who may submit a rating for themselves or as a proxy -- governed by FEAT-12.SPEC-003 (Rating Access & Authorization Rules). This screen applies both on every tap of a thumbs control; there is no separate submit step to defer validation to.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Close control tap (any step) | Week plan screen the meal was opened from | FEAT-03 (AI Weekly Dinner Plan Generation) or FEAT-23 (Manual Weekly Planning), matching the entry context |
| Done control tap (final step) | Week plan screen the meal was opened from | FEAT-03 or FEAT-23, matching the entry context |

## Data Model

**Creates:** Rating -- one record per member (or young kid profile) per Planned Meal on first submission. Fields set: member (the signed-in adult, or the young kid profile being proxy-rated), planned_meal (the meal this screen was opened for), value (up or down), recorded_by (set only for a proxy rating -- the adult who tapped the control on the kid's behalf; absent for a member's own rating).
**Reads:** Rating -- this screen loads any existing rating for the current meal/member (or meal/kid-profile) pair to show the Already Rated state before allowing a change. Planned Meal -- recipe name and night, for the header. Member Profile -- the household's young kid profiles, to build the proxy step sequence, and the signed-in adult's identity.
**Updates:** Rating -- the value field, when the member (or the recording adult, for a proxy rating) changes a previously submitted rating, per FEAT-12.SPEC-002's changeable-until-archived window.
**Deletes:** None -- this screen never removes a Rating; deletion or anonymization on membership change is owned by FEAT-18 and FEAT-09 (feature-overview.md, Entity-Lifecycle Coverage Matrix).

## Business Rules

- One rating per household member (including each young kid profile, by proxy) per Planned Meal -- governed by FEAT-12.SPEC-002.
- A rating can be changed up until the plan holding that meal is archived -- governed by FEAT-12.SPEC-002; after archival this screen's thumbs controls no longer accept a change (see Edge Cases).
- Who may submit an own rating or a proxy rating on this screen -- governed by FEAT-12.SPEC-003.
- A rating given while offline is held locally and synced once connectivity returns, using the real-time synchronization boundary FEAT-03.SPEC-011 / FEAT-06.SPEC-005 provides; this feature invents no separate synchronization mechanism of its own (feature-overview.md, Side-Effect Inventory).
- A meal with no rating for a given member is treated as neutral, never assumed liked or disliked, and this screen never prompts or presses the household to return and rate it.
- A repeated down-rating pattern for the same member and meal is evaluated by FEAT-12.SPEC-005 (Repeated-Dislike Learned Update) once a qualifying rating is submitted here; this screen does not itself evaluate or display that pattern.

## Edge Cases

- **Household member taps a thumbs control twice rapidly (double-submit)** -- The second tap on the same control while the first is still confirming is ignored; a tap on the opposite control while the first is still confirming is queued and applied once the first submission settles, resulting in the second value being the one saved.
- **Two adults record the same young kid profile's rating for the same meal at effectively the same time (e.g., Maya on her phone and Sam on his, both proxy-rating Jordan)** -- Concurrent-edit conflict: per the dependency map's Contention note for Rating, resolution is last-write-wins -- since one rating per member per meal is kept and ratings stay changeable, the later-arriving submission is the value that persists; neither adult sees an error, and either can change it again afterward.
- **Adult attempts to change a rating after the plan holding that meal has been archived** -- The thumbs controls for that meal become read-only, showing the last recorded value with no "Tap to change" text; per FEAT-12.SPEC-002, the change window has closed.
- **Adult closes the screen mid-proxy-sequence** -- Every rating already tapped (own and any completed proxy steps) was already submitted individually and is kept; unrated remaining profiles stay neutral (unrated) and are not revisited automatically.
- **Household has no young kid profiles** -- The proxy sequence is skipped entirely; the own-rating step's Next control reads "Done" directly, since there is nothing to step through.
- **A rating submission fails and the household member navigates away before it retries successfully** -- The background retry continues independent of this screen; if it exhausts its retries, the rating is left unsubmitted and the meal remains neutral for that member until they reopen this screen and rate again (Stage 2 Error: no error is surfaced unless retries exhaust, and no retroactive prompt is created per this feature's no-reminders rule).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-002 (Rating Submission, Change & Proxy Rules) | References (outbound) | This screen enforces the one-rating-per-member-per-meal, change-window, and proxy-counting rules on every submission |
| FEAT-12.SPEC-003 (Rating Access & Authorization Rules) | References (outbound) | This screen enforces who may rate for themselves or as a proxy, and the never-shown-individually privacy rule |
| FEAT-12.SPEC-004 (Preference Weighting & Tier-Gating Rule) | Affects (outbound) | Ratings this screen captures are the input the weighting rule reads; this screen carries no weighting logic or tier-gating display of its own |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Navigation (inbound/outbound) | Entry from the week plan's "Rate after dinner" trigger on a cooked meal; Close/Done return there |
| FEAT-23 (Manual Weekly Planning) | Navigation (inbound/outbound) | Same entry and return pattern for a manually planned week's cooked meal |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| meal_rated_up | member type (adult / young-kid-by-proxy), planned_meal reference, whether this was a first-time rating or a change | A thumbs-up control is tapped and the rating is submitted (new or changed) | N/A -- no success-metrics.md metric names Meal Rating & Preference Learning as its Connected Feature; this event is defined per product-features.md's Signals list for FEAT-12 and is available for a future metric to draw on |
| meal_rated_down | member type (adult / young-kid-by-proxy), planned_meal reference, whether this was a first-time rating or a change | A thumbs-down control is tapped and the rating is submitted (new or changed) | N/A -- same reason as meal_rated_up |
| rating_changed | member type, planned_meal reference, previous value, new value | A member changes a previously submitted rating to the opposite value before archival | N/A -- same reason as meal_rated_up |
| kid_rating_recorded_by_adult | recording adult, young kid profile rated, planned_meal reference, value | An adult submits a rating during the proxy sequence for a young kid profile | N/A -- same reason as meal_rated_up |

## Acceptance Criteria

**FEAT-12.SPEC-001-AC-01:** Given Maya is on the Post-Dinner Rating Prompt for tonight's cooked meal with no existing rating, when she taps the thumbs-up control, then the control shows its selected state instantly and the rating is recorded against Maya and that meal.

**FEAT-12.SPEC-001-AC-02:** Given Sam has already rated tonight's meal thumbs-down, when he reopens the prompt for that meal, then the thumbs-down control shows selected with the text "You rated this: down. Tap to change."

**FEAT-12.SPEC-001-AC-03:** Given Sam already rated tonight's meal thumbs-down, when he taps the thumbs-up control, then his rating changes to thumbs-up and the text updates to "You rated this: up. Tap to change."

**FEAT-12.SPEC-001-AC-04:** Given Maya is on the own-rating step and the household has two young kid profiles, when she taps Next after rating herself, then the screen advances to the proxy step showing "Rating for {first kid's name}" with the instruction "Hand the phone to {first kid's name}."

**FEAT-12.SPEC-001-AC-05:** Given Maya is on a proxy step for Jordan (young kid profile), when she taps the thumbs-up control, then a rating is recorded against Jordan's profile and tonight's meal, tagged with Maya as the recording adult, and this counts as Jordan's one rating for the meal.

**FEAT-12.SPEC-001-AC-06:** Given Maya is on a proxy step for a young kid profile with no rating yet, when she taps Next without tapping a thumbs control, then the screen advances to the next step and that profile's rating for this meal remains neutral (unrated).

**FEAT-12.SPEC-001-AC-07:** Given Sam is on the final proxy step, when he taps Done, then the screen closes and returns to the week plan he opened the meal from.

**FEAT-12.SPEC-001-AC-08:** Given Maya has no young kid profiles in her household, when she completes her own rating, then the Next control reads "Done" directly and no proxy steps are shown.

**FEAT-12.SPEC-001-AC-09:** Given Sam loses connectivity while on the Post-Dinner Rating Prompt, when he taps a thumbs control, then the banner "You're offline -- this rating will save when you reconnect." appears and the rating is submitted automatically once connectivity returns.

**FEAT-12.SPEC-001-AC-10:** Given a rating submission for Maya fails, when the failure occurs, then no error is shown to Maya and the submission retries automatically in the background.

**FEAT-12.SPEC-001-AC-11:** Given Maya and Sam each attempt to record Jordan's (young kid profile) rating for the same meal within moments of each other, when both submissions are processed, then the later-arriving submission is the value that persists for Jordan's rating, per the Contention resolution for the Rating entity.

**FEAT-12.SPEC-001-AC-12:** Given the plan holding tonight's meal has been archived, when Sam opens the Post-Dinner Rating Prompt for that meal, then the thumbs controls show his last recorded value read-only, with no "Tap to change" text.

**FEAT-12.SPEC-001-AC-13:** Given Maya closes the Post-Dinner Rating Prompt after rating herself but before completing the proxy sequence, when she reopens the week plan later, then her own rating is kept and the household is never prompted to finish rating the remaining profiles.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 6 (not yet rated, already rated, submitting, error, offline/degraded, proxy sequence mid-flow) | 6 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Rating Submission, Change & Proxy Rules

## Overview

**Name:** Rating Submission, Change & Proxy Rules
**ID:** FEAT-12.SPEC-002
**Type:** Logic/Rule
**Purpose:** Governs one-rating-per-member-per-meal, the changeable-until-archived window, the unrated-is-neutral default, and how a proxy rating on behalf of a young kid profile counts as that kid's one rating.
**Parent Feature:** FEAT-12 -- Meal Rating & Preference Learning
**Governed Entity:** Rating

## Scope and Non-Goals

**In Scope:**
- Field-level rules for the Rating entity's value and recorded_by fields
- The one-rating-per-member-per-meal uniqueness rule
- The changeable-until-archived change window
- The unrated-is-neutral default (what happens when no rating exists)
- The proxy-counts-as-the-kid's-rating rule -- how a young kid profile's proxy-recorded rating is treated identically to a self-recorded rating for uniqueness and change-window purposes

**Non-Goals:**
- Who may submit a rating for themselves, or as a proxy for a young kid profile -- governed by FEAT-12.SPEC-003 (Rating Access & Authorization Rules); this spec's Authorization Rules table cross-references that spec rather than restating it.
- Whether ratings currently influence future plan weighting, and the paid-tier gate on that effect -- governed by FEAT-12.SPEC-004 (Preference Weighting & Tier-Gating Rule); submission and change rules apply identically on both tiers.
- The repeated-dislike detection and Dietary Rule merge logic that a qualifying down-rating feeds -- governed by FEAT-12.SPEC-005 (Repeated-Dislike Learned Update); this spec only defines which submissions are valid, not what happens to them afterward.
- The screen presentation of these rules (thumbs controls, step sequencing, error banners) -- owned by FEAT-12.SPEC-001 (Post-Dinner Rating Prompt), which references this spec rather than restating its logic.

## Governed Entity

**Entity:** Rating
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| member | reference | The Member Profile (adult, older-kid Later login, or young kid profile) this rating is recorded against |
| planned_meal | reference | The cooked Planned Meal (dinner) this rating attaches to |
| value | enum (up, down) | The thumbs up or thumbs down rating value |
| recorded_by | reference (optional) | The adult Member Profile who recorded this rating on a young kid profile's behalf, where applicable; absent when the rating is self-recorded |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-12.SPEC-001 | Post-Dinner Rating Prompt | On every tap of a thumbs control -- both the own-rating step and each proxy step; authorization for who may act is checked before this spec's submission rules apply |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|------------------------------|----------------|------------------|-----------|
| member | Required; must reference an active Member Profile (adult, older-kid limited login, or young kid profile) belonging to the same household as the planned_meal | Always | On submit | "This rating could not be saved. Reload the plan and try again." | Yes |
| planned_meal | Required; must reference a Planned Meal whose status is Cooked (per the dependency map's Planned Meal status values) or later in its lifecycle | Always | On submit | "This meal isn't ready to rate yet." | Yes |
| value | Required; must be exactly "up" or "down" -- no partial or neutral value is ever stored | Always | On submit | "Choose thumbs up or thumbs down to rate this meal." | Yes |
| recorded_by | No validation beyond data type when absent (self-recorded rating). When present, must reference an adult Member Profile (Maya or Sam) in the same household as member, and member itself must be a young kid profile (proxy rule below) | When present | On submit | "Only an adult can record a rating for this profile." | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|--------------------|-----------|--------------------|
| One rating per member per meal | member, planned_meal | No more than one Rating record may exist for the same (member, planned_meal) pair; a second submission for the same pair updates the existing record's value rather than creating a new one | N/A -- this is a silent upsert, not a user-facing error (see Business Rules) |
| Proxy requires a young kid subject | member, recorded_by | recorded_by may be present only when member refers to a young kid profile (no login); an adult's own rating, or the Later-phase older kid's own rating, never carries a recorded_by value | "Only a young kid profile's rating can be recorded by an adult on their behalf." |
| Changeable-until-archived | value, planned_meal | value may be updated only while the Weekly Plan holding planned_meal has not yet reached Archived status | "This meal's rating window has closed -- the plan has been archived." |

## Authorization Rules

Who may submit an own rating or a proxy rating is governed authoritatively by FEAT-12.SPEC-003 (Rating Access & Authorization Rules); the rows below restate only the actions this spec's submission/change/proxy logic gates, with authorization deferred to that spec's role-action matrix.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|-------------|-----------|----------------------------------------------|
| Submit own rating | Maya, Sam, Jordan (older kid, limited login -- Later) | Always for their own member/meal pair -- full authorization detail in FEAT-12.SPEC-003 | See FEAT-12.SPEC-003 for the exact denied experience per role |
| Submit proxy rating for a young kid profile | Maya, Sam | Always -- either adult may proxy any young kid profile in the household, per FEAT-12.SPEC-003 | See FEAT-12.SPEC-003 |
| Change a previously submitted rating (own or proxy) | Same roles as submission, subject to the changeable-until-archived cross-field rule above | Plan holding the meal is not yet Archived | "This meal's rating window has closed -- the plan has been archived." |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|------------------------|-------------------|---------------------|
| value | No default -- a Rating record is created only when a value is explicitly submitted; there is no neutral stored value | On create only | Not applicable (no default exists to override) |
| recorded_by | Absent by default; set only when the submitting user is recording on behalf of a young kid profile | On create only, from the proxy context | No -- always set from who actually performed the proxy submission, never user-editable |

## Business Rules

- **One-rating-per-member-per-meal (uniqueness):** A second submission for the same (member, planned_meal) pair is treated as a change to the existing rating, not a new record -- there is never more than one stored value per member per meal.
- **Unrated-is-neutral default:** A (member, planned_meal) pair with no Rating record is treated as neutral by every downstream consumer (FEAT-12.SPEC-004's weighting, FEAT-12.SPEC-005's repeated-dislike detection) -- it is never assumed liked or disliked, and this spec defines no mechanism that infers a value for an absent rating.
- **Changeable-until-archived window:** A rating's value may be changed freely, any number of times, up until the Weekly Plan holding its planned_meal reaches Archived status (feature-overview.md, Non-Goals: "Deleting a rating outright"). Once archived, the value is fixed and FEAT-12.SPEC-001's controls become read-only for that meal.
- **Proxy-counts-as-the-kid's-rating:** A rating recorded by an adult on behalf of a young kid profile is stored exactly like a self-recorded rating for uniqueness and change-window purposes -- it occupies the kid profile's one rating slot for that meal, and either adult (not only the one who originally recorded it) may change it later, subject to FEAT-12.SPEC-003's authorization.
- **Ratings count identically regardless of plan origin (XBR relationship):** A rating on a meal from an AI-generated week (FEAT-03) and a rating on a meal from a manually planned week (FEAT-23) are governed by these same rules with no distinction (feature-overview.md, Cross-Feature Touchpoints).
- **Ownership at membership change is out of this spec's scope:** What happens to a member's ratings when they leave the household (FEAT-09, anonymized influence) or are removed (FEAT-18, deleted within 30 days per XBR-16) is governed entirely by those owning features; this spec's rules apply only while the member is Active.

## Edge Cases

- **Second submission for the same (member, planned_meal) pair with the same value already stored** -- Treated as a no-op change (the value does not actually change); no error, and the record's state is unaffected.
- **Rating submitted for a Planned Meal that is not yet Cooked** -- Rejected with "This meal isn't ready to rate yet." (field validation, planned_meal). This should not occur in normal use since FEAT-12.SPEC-001 is reached only from a cooked meal's "Rate after dinner" trigger, but the rule holds regardless of entry path.
- **Plan archives while the Post-Dinner Rating Prompt is open with an unsaved change in flight** -- The in-flight submission that started before archival is honored (it was valid when initiated); any subsequent tap on that meal's controls after the screen reflects the archived state is rejected per the changeable-until-archived rule.
- **Proxy rating attempted for a Member Profile that is not a young kid profile (e.g., another adult)** -- Rejected with "Only a young kid profile's rating can be recorded by an adult on their behalf." -- the cross-field rule blocks recorded_by from ever attaching to an adult's or older-kid-login's own rating.
- **Two proxy submissions for the same young kid profile and meal arrive at effectively the same time from two different adults** -- Per the dependency map's Contention note for Rating, this resolves last-write-wins: whichever submission's write lands second is the value that persists, since only one Rating record is kept per (member, planned_meal) pair.
- **A young kid profile is removed from the household after a proxy rating was recorded but before the plan archives** -- Out of this spec's scope; removal and its cascade to the profile's Ratings is governed by FEAT-18 (XBR-16), which supersedes any in-flight change attempt on that profile's rating.

## Acceptance Criteria

**FEAT-12.SPEC-002-AC-01:** Given Maya submits a thumbs-up rating for a cooked meal she has not yet rated, when the submission is processed, then a new Rating record is created with member set to Maya, planned_meal set to that meal, and value set to "up".

**FEAT-12.SPEC-002-AC-02:** Given Maya has already rated a meal thumbs-up, when she submits thumbs-down for that same meal, then the existing Rating record's value updates to "down" rather than a second record being created.

**FEAT-12.SPEC-002-AC-03:** Given a meal has no Rating record for Sam, when FEAT-12.SPEC-004's weighting or FEAT-12.SPEC-005's pattern detection considers that (Sam, meal) pair, then it is treated as neutral -- never as an implicit like or dislike.

**FEAT-12.SPEC-002-AC-04:** Given the Weekly Plan holding a meal Sam rated has not yet been archived, when Sam changes his rating from thumbs-down to thumbs-up, then the change is accepted and the stored value becomes "up".

**FEAT-12.SPEC-002-AC-05:** Given the Weekly Plan holding a meal has reached Archived status, when Sam attempts to change his rating for that meal, then the submission is rejected with "This meal's rating window has closed -- the plan has been archived."

**FEAT-12.SPEC-002-AC-06:** Given Maya records a thumbs-down rating on behalf of Jordan (young kid profile), when the submission is processed, then a Rating record is created with member set to Jordan's profile, value set to "down", and recorded_by set to Maya.

**FEAT-12.SPEC-002-AC-07:** Given Jordan's (young kid profile) rating for a meal was originally recorded by Maya, when Sam later changes that rating to the opposite value before archival, then the change is accepted and the stored value updates, since the proxy rating is governed by the same changeable-until-archived rule as a self-recorded rating.

**FEAT-12.SPEC-002-AC-08:** Given a submission attempts to attach recorded_by to Sam's own rating, when the submission is validated, then it is rejected with "Only a young kid profile's rating can be recorded by an adult on their behalf."

**FEAT-12.SPEC-002-AC-09:** Given a rating submission arrives with no value selected, when it is validated, then it is rejected with "Choose thumbs up or thumbs down to rate this meal." and no record is created or changed.

**FEAT-12.SPEC-002-AC-10:** Given a rating is submitted for a Planned Meal that has not yet reached Cooked status, when it is validated, then it is rejected with "This meal isn't ready to rate yet."

**FEAT-12.SPEC-002-AC-11:** Given Maya and Sam each submit a proxy rating for the same young kid profile and the same meal within moments of each other, when both submissions are processed, then exactly one Rating record exists afterward, holding whichever value was written last.

**FEAT-12.SPEC-002-AC-12:** Given a meal was rated on a manually planned week (FEAT-23) rather than an AI-generated week (FEAT-03), when the same submission, change-window, and proxy rules are evaluated, then they apply identically regardless of the plan's origin.

**FEAT-12.SPEC-002-AC-13:** Given Maya re-submits the same thumbs-up value that is already stored for a meal she previously rated, when the submission is processed, then the stored record is unchanged and no error is shown.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Rating Access & Authorization Rules

## Overview

**Name:** Rating Access & Authorization Rules
**ID:** FEAT-12.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs who may rate for whom on the Rating entity -- Maya Full, Sam Own-only, either adult as proxy for a young kid profile, the Later-phase older-kid login Own-only, Riley View, unauthorized visitors None -- and the never-shown-individually privacy rule.
**Parent Feature:** FEAT-12 -- Meal Rating & Preference Learning
**Governed Entity:** Rating

## Scope and Non-Goals

**In Scope:**
- The complete, authoritative role-action matrix for every action the product defines on the Rating entity: create own, create proxy, view own, view aggregate effect, view individual (another member's), change own, change proxy
- The never-shown-individually privacy rule: no household member, including Maya, ever sees another member's individual rating broken out
- Riley's (Operator) View access for diagnosis, and its exact boundary
- Unauthorized-visitor and expired-session denial behavior for rating actions

**Non-Goals:**
- The uniqueness, change-window, and field-level submission rules for the Rating entity -- governed by FEAT-12.SPEC-002 (Rating Submission, Change & Proxy Rules); this spec governs only who may act, not the mechanics of the action itself.
- Whether ratings currently influence future plan weighting -- governed by FEAT-12.SPEC-004 (Preference Weighting & Tier-Gating Rule); that gate is a tier condition on a system process, not a role-based access rule, and stays out of this spec.
- The presentation of denied states on screen (control hidden vs. disabled) -- owned by FEAT-12.SPEC-001, which references this spec's exact denied experience per row rather than restating the authorization logic.
- Riley's access to any data beyond rating activity -- FEAT-22 (Operator Read-Only Support Access) owns the full scope of what support can see; this spec states only the Ratings-specific line of that boundary.

## Governed Entity

**Entity:** Rating
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| member | reference | The Member Profile (adult, older-kid Later login, or young kid profile) this rating is recorded against |
| planned_meal | reference | The cooked Planned Meal (dinner) this rating attaches to |
| value | enum (up, down) | The thumbs up or thumbs down rating value |
| recorded_by | reference (optional) | The adult Member Profile who recorded this rating on a young kid profile's behalf, where applicable |

No validation beyond data type applies to any of these fields in this spec -- field-level and cross-field validation rules are governed entirely by FEAT-12.SPEC-002; this spec addresses only who may act on the entity.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-12.SPEC-001 | Post-Dinner Rating Prompt | On screen entry (which controls are shown -- own step, proxy step availability) and on every submission attempt (who may act) |
| FEAT-22 (external to this feature) | Operator Read-Only Support Access | Riley's View access to aggregate rating activity during an open Support Request is governed by this spec's Riley row, exercised through FEAT-22's own screens |

## Field Validation Rules

No field validation rules in this spec -- every field of the governed entity is addressed under FEAT-12.SPEC-002 (Rating Submission, Change & Proxy Rules), which owns all field-level and cross-field validation for the Rating entity. This spec's rule surface is entirely the Authorization Rules table below.

## Cross-Field Rules

No cross-field rules in this spec -- the proxy-eligibility cross-field logic (recorded_by may be present only when member is a young kid profile) is governed by FEAT-12.SPEC-002. This spec governs only who is authorized to perform that proxy action, in the Authorization Rules table below.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|-------------|-----------|----------------------------------------------|
| Create/submit own rating | Maya (Organiser) | Always -- Ratings: Full per the Access Matrix | -- |
| Create/submit own rating | Sam (Other Adult Member) | Always, and only for himself -- Ratings: Own-only per the Access Matrix | -- |
| Create/submit own rating | Jordan (older kid, limited login -- Later) | Always, and only for himself once this login exists -- Ratings: Own-only per the Access Matrix | Not applicable in v1 -- this role's login does not yet exist (FEAT-17, Later) |
| Create/submit own rating | Jordan (young kid profile, no login -- MVP) | Never directly -- no login exists for this profile | This profile has no sign-in path, so there is no direct-rating control to deny; its rating is entered only through the proxy action below |
| Create/submit own rating | Riley (Operator) | Never | Rating controls are not shown to Riley; Riley never opens FEAT-12.SPEC-001 |
| Create/submit own rating | Unauthorized visitor | Never | Redirected to the household sign-in screen; no rating control is ever shown |
| Create/submit proxy rating for a young kid profile | Maya, Sam | Always -- either adult may record any young kid profile's rating in their own household on that profile's behalf | -- |
| Create/submit proxy rating for a young kid profile | Jordan (older kid, limited login -- Later), Riley, unauthorized visitor | Never | Proxy step is never offered to the older-kid login (Own-only covers only his own rating); not shown to Riley; not reachable by an unauthorized visitor |
| Change own or proxy rating | Same roles and conditions as the corresponding create/submit row above, subject to FEAT-12.SPEC-002's changeable-until-archived window | Plan holding the meal is not yet Archived | Same denied experience as the corresponding create/submit row; once archived, see FEAT-12.SPEC-002 for the archived-specific denied message |
| View own individual rating | Maya, Sam, Jordan (older kid, limited login -- Later, once it exists) | Always, for their own previously submitted value only (shown on FEAT-12.SPEC-001 as the Already Rated state) | -- |
| View another member's individual rating, broken out | Maya, Sam, Jordan (older kid, limited login -- Later) | Never -- for any role, regardless of organiser status | No control anywhere in the product ever surfaces another member's individual rating value; only the aggregate effect on future plans (via FEAT-12.SPEC-004's weighting) is ever visible, to any role, including Maya |
| View aggregate rating effect on future plans | Maya, Sam | Always, as it appears in the plan itself (the meals the plan favors or avoids); this is not a rating-value display, only the plan's outcome | -- |
| View rating activity for diagnosis | Riley (Operator) | Only while a Support Request for that household is open (FEAT-22), and only in aggregate -- never an individual member's rating value broken out | Outside an open Support Request, Riley has no access to any household's rating activity at all |

## Defaults and Derivations

No defaults or derivations in this spec -- default values and derived fields for the Rating entity (there are none beyond the absence of recorded_by on a self-recorded rating) are addressed under FEAT-12.SPEC-002.

## Business Rules

- **Never-shown-individually privacy rule:** No household member, including Maya as organiser, is ever shown another member's individual rating value. This holds regardless of role, regardless of whether the viewer created the rating (in the proxy case, the recording adult does not gain a persistent "view Jordan's rating" privilege beyond what the Already Rated state on FEAT-12.SPEC-001 shows in the moment of recording). Only the aggregate effect on future plan generation is ever visible (feature-overview.md, Data Notes; ASMP-26).
- **A proxy rating is not a delegation of viewing rights:** Recording a rating on behalf of a young kid profile lets the recording adult see that value in the moment of the proxy step (so they can confirm what was just tapped), but it does not create a standing ability to browse that profile's ratings later, outside the rating screen's own current-state display.
- **Riley's boundary is doubly scoped:** Riley's View access to rating activity requires both an open Support Request for that specific household (FEAT-22, XBR-14) and stays at the aggregate level -- these two conditions apply together, not as alternatives.
- **Children's-privacy-class protection applies to proxy-recorded ratings:** A young kid profile's rating, and any resulting learned Dietary Rule entry it feeds (FEAT-12.SPEC-005), carries children's-privacy-class protection consistent with minimal collection and parent-controlled data (ASMP-27); this constrains who may ever see the raw value (nobody, individually, beyond the recording moment) more tightly than an adult's own rating, which is at least visible to that adult themselves.

## Edge Cases

- **Maya (Organiser) attempts to view Sam's individual rating for a meal, expecting organiser-level visibility** -- Denied identically to any other role: no control anywhere shows it. Organiser status does not override the never-shown-individually privacy rule.
- **Sam attempts to change Maya's rating directly (not a proxy scenario -- both are adults)** -- Never permitted: Sam's Own-only access covers only his own rating and any young kid profile's proxy rating, never another adult's rating. No control for this exists on FEAT-12.SPEC-001.
- **A Support Request closes while Riley is mid-review of a household's rating activity** -- Access ends immediately per FEAT-22's boundary; any further attempt to view that household's rating activity is denied until a new Support Request is opened.
- **The older-kid limited login (Later, FEAT-17) is introduced and a household has both an older kid and a young kid profile** -- The older kid's row (Own-only for their own rating) and the young kid profile's row (proxy-only, no login) apply independently and do not interact; the older kid never gains proxy authority over the young kid's rating, since proxy authority in this spec is Maya/Sam (adults) only.
- **An adult attempts to proxy-rate a Member Profile that turns out to be another adult, not a young kid profile** -- Authorization for the proxy action does not by itself catch this; FEAT-12.SPEC-002's cross-field rule (proxy requires a young kid subject) rejects the submission at the field-validation layer, since this spec authorizes the action category (proxying a young kid) rather than validating the specific target's type.

## Acceptance Criteria

**FEAT-12.SPEC-003-AC-01:** Given Maya opens the Post-Dinner Rating Prompt, when she looks for rating controls, then she can submit her own rating for the meal, since Maya has Full Ratings access.

**FEAT-12.SPEC-003-AC-02:** Given Sam opens the Post-Dinner Rating Prompt, when he attempts to submit a rating, then he can rate only himself, consistent with his Own-only Ratings access.

**FEAT-12.SPEC-003-AC-03:** Given Maya is on the proxy step for Jordan (young kid profile), when she submits a rating, then it is accepted, since either adult may record any young kid profile's rating on that profile's behalf.

**FEAT-12.SPEC-003-AC-04:** Given Sam is on the proxy step for Jordan (young kid profile), when he submits a rating, then it is accepted for the same reason as Maya's proxy submission -- proxy authority is not organiser-exclusive.

**FEAT-12.SPEC-003-AC-05:** Given Jordan is a young kid profile with no login, when anyone attempts to reach a direct sign-in for Jordan to rate, then no such path exists -- Jordan's rating is recorded only through an adult's proxy action.

**FEAT-12.SPEC-003-AC-06:** Given Maya wants to see Sam's individual rating for a meal, when she looks anywhere in the product for it, then no control shows it -- only the aggregate effect on future plans is visible, even to Maya as organiser.

**FEAT-12.SPEC-003-AC-07:** Given Sam attempts to change Maya's own rating for a meal, when he looks for a control to do so, then none is shown -- Sam's Own-only access never extends to another adult's rating.

**FEAT-12.SPEC-003-AC-08:** Given Riley (Operator) has no open Support Request for a household, when Riley attempts to view any of that household's rating activity, then access is denied entirely.

**FEAT-12.SPEC-003-AC-09:** Given Riley has an open Support Request for a household, when Riley reviews rating activity for diagnosis through FEAT-22, then Riley sees aggregate rating activity only, never an individual member's rating value broken out.

**FEAT-12.SPEC-003-AC-10:** Given an unauthorized visitor (not signed in as a household member) attempts to reach the Post-Dinner Rating Prompt, when the attempt is made, then they are redirected to the household sign-in screen and no rating control is ever shown.

**FEAT-12.SPEC-003-AC-11:** Given a household member's session has expired while the Post-Dinner Rating Prompt is open, when they attempt to submit a rating, then the action is denied and they must re-authenticate before any further rating can be recorded.

**FEAT-12.SPEC-003-AC-12:** Given the household has both an older kid with a limited login (Later) and a young kid profile, when the older kid attempts to proxy-rate the young kid profile, then this is denied -- proxy authority belongs only to Maya and Sam, the household's adults.

**FEAT-12.SPEC-003-AC-13:** Given a Support Request Riley was reviewing is resolved and closed, when Riley attempts to continue viewing that household's rating activity, then access is denied until a new Support Request is opened.

**FEAT-12.SPEC-003-AC-14:** Given Maya recorded a proxy rating for Jordan moments ago on the Post-Dinner Rating Prompt, when she later looks elsewhere in the product for Jordan's rating history, then no such view exists -- the momentary confirmation on the rating screen does not create a standing ability to browse Jordan's ratings.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 0 (N/A -- owned by FEAT-12.SPEC-002; confirmed considered, not skipped) | 0 |
| Cross-Field Rules | 0 (N/A -- owned by FEAT-12.SPEC-002; confirmed considered, not skipped) | 0 |
| Authorization Rules | 13 | 13 |
| Defaults/Derivations | 0 (N/A -- owned by FEAT-12.SPEC-002; confirmed considered, not skipped) | 0 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



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
