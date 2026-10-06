---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-12.SPEC-002
spec_name: Rating Submission, Change & Proxy Rules
spec_slug: rating-submission-change-proxy-rules
parent_feature: FEAT-12
parent_feature_name: Meal Rating & Preference Learning
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 18
acceptance_criteria_count: 13
---

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
