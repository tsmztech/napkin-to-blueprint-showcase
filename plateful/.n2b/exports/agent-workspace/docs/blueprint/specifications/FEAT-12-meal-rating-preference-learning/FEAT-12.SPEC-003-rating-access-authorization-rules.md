---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-12.SPEC-003
spec_name: Rating Access & Authorization Rules
spec_slug: rating-access-authorization-rules
parent_feature: FEAT-12
parent_feature_name: Meal Rating & Preference Learning
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 17
acceptance_criteria_count: 14
---

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
