---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-23.SPEC-005
spec_name: Manual Planning Validation & Limits
spec_slug: manual-planning-validation-limits
parent_feature: FEAT-23
parent_feature_name: Manual Weekly Planning
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 22
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Manual Planning Validation & Limits

## Overview

**Name:** Manual Planning Validation & Limits
**ID:** FEAT-23.SPEC-005
**Type:** Logic/Rule
**Purpose:** Enforces the structural limits of manual weekly planning -- one dinner per night, seven nights per week, planning up to one week ahead, and one open pick suggestion per member per night.
**Parent Feature:** FEAT-23 -- Manual Weekly Planning
**Governed Entity:** Weekly Plan, Planned Meal, and Swap Suggestion (the three entities whose structural limits this feature enforces for manual planning)

## Scope and Non-Goals

**In Scope:**
- The one-dinner-per-night limit on Planned Meal
- The seven-nights-per-week structure of a Weekly Plan
- The one-week-ahead planning window on Weekly Plan.week
- The one-open-pick-suggestion-per-member-per-night limit on Swap Suggestion
- Authorization for create, update, and delete actions on Planned Meal, and create on the pick-suggestion kind of Swap Suggestion, across every role
- Default values applied when a Weekly Plan or Planned Meal is created through manual planning

**Non-Goals:**
- Safety eligibility of a candidate recipe -- owned by FEAT-23.SPEC-004 (Safe-Choice Filtering & Placement Block); this spec governs structural and quantity limits only, never which recipes are safe
- Approval, week-start adoption, or any other Weekly Plan status transition -- owned by FEAT-03 (AI Weekly Dinner Plan Generation) per the Entity-Lifecycle Coverage Matrix; a manually built week enters at "Started" through FEAT-23.SPEC-001's creation and this feature never advances it further
- Reviewing, accepting, declining, or lapsing a Swap Suggestion once created -- owned entirely by FEAT-04 (One-Tap Meal Swap) per XBR-06; this spec governs only the creation-time limit on new pick suggestions

## Governed Entity

**Entities:** Weekly Plan, Planned Meal, Swap Suggestion
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| Weekly Plan.week | date range | The calendar week the plan covers; planning allowed up to one week ahead |
| Weekly Plan.origin | enum | AI-generated or manually built |
| Weekly Plan.status | enum | Generated/Started, Reviewed, Approved, Active, Archived |
| Weekly Plan.estimated_total | number | The week's estimated cost against budget; recalculated by FEAT-23.SPEC-006, not derived here |
| Planned Meal.night | enum (day of week) | At most one dinner per night |
| Planned Meal.recipe | reference | The chosen Recipe (eligibility governed by FEAT-23.SPEC-004, not here) |
| Planned Meal.status | enum | Proposed/Picked, Confirmed, Swapped, Removed, Cooked |
| Swap Suggestion.suggesting_member | reference | The other adult member proposing the pick |
| Swap Suggestion.night | enum (day of week) | The target slot; one open suggestion per member per night |
| Swap Suggestion.proposed_recipe | reference | A safety-checked recipe (governed by FEAT-23.SPEC-004, not here) |
| Swap Suggestion.outcome | enum | Suggested, Accepted, Declined, Lapsed |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-23.SPEC-001 | Weekly Plan | On screen entry (which weeks and nights are offered) and on the Clear action |
| FEAT-23.SPEC-002 | Pick / Change a Recipe | On screen entry (Place vs. Replace availability) and on save |
| FEAT-23.SPEC-003 | Suggest a Pick | On screen entry (whether the screen is reachable for a given night) and on send |
| FEAT-23.SPEC-006 | Apply Manual Pick | During processing, immediately before writing a create, update, or delete to Planned Meal |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Weekly Plan.week | Must be the current week or exactly one week ahead -- no farther | Always | On Weekly Plan creation (FEAT-23.SPEC-001) and before any pick, change, or suggestion is applied to it | "This week is beyond how far ahead you can plan -- build the current or next week first." | Yes |
| Planned Meal.night | A create (place) is only valid when the target night has no existing dinner-status Planned Meal; an existing night's dinner is changed through the update (replace) path, never a second create | Always | On placement attempt (FEAT-23.SPEC-002, FEAT-23.SPEC-006) | "This night already has a dinner -- use Change to replace it." | Yes |
| Weekly Plan (night count) | No count validation beyond the one-dinner-per-night rule above -- the week's seven nights are a structural property (Monday through Sunday), not a quantity a user could exceed | Always | -- | -- | -- |
| Swap Suggestion.night + suggesting_member | At most one Swap Suggestion with outcome "Suggested" per suggesting_member per night | Always, evaluated at creation | On send attempt (FEAT-23.SPEC-003) | "You already suggested a pick for {Night} -- wait for Maya's decision or its lapse." | Yes |
| Swap Suggestion.proposed_recipe | No validation beyond data type in this spec -- recipe eligibility is governed by FEAT-23.SPEC-004 | Always | -- | -- | -- |
| Weekly Plan.origin | No validation beyond data type -- set once at creation (see Defaults and Derivations) | Always | -- | -- | -- |
| Planned Meal.status | No validation beyond data type in this spec -- the value set on manual create is fixed (see Defaults and Derivations); other transitions belong to other features | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Pick and suggestion actions scoped to the plannable week | Weekly Plan.week, Planned Meal.night, Swap Suggestion.night | Placement, change, clear, and suggestion actions are only available for nights belonging to a Weekly Plan whose week is the current or next week; no control for any farther week is ever shown | "This week can't be changed from here." (defensive message, shown only if reached through a stale link to a week beyond the plannable window) |
| Suggestion night must be open | Planned Meal.night (target), Swap Suggestion.night, Swap Suggestion.outcome | A new pick suggestion may target any night, whether empty or already picked; it does not require the night to be empty, since a suggestion is a proposal the organiser may accept in place of an existing pick | N/A -- this is a permissive rule with no rejection case of its own; the one-open-suggestion-per-member-per-night rule above is what blocks a second attempt |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create Planned Meal (place a dinner) | Maya | Target night has no existing dinner-status Planned Meal, the recipe is eligible (FEAT-23.SPEC-004), and the Weekly Plan's week is within the one-week-ahead window | -- |
| Create Planned Meal (place a dinner) | Sam | Never -- placement is the organiser's action alone (scope-boundaries.md SC-04) | No Place action exists in Sam's screens (FEAT-23.SPEC-003) |
| Update Planned Meal (change a dinner) | Maya | Target night has an existing dinner-status Planned Meal, the new recipe is eligible, and the week is within the one-week-ahead window | -- |
| Update Planned Meal (change a dinner) | Sam | Never | No Change action exists in Sam's screens |
| Delete Planned Meal (clear a night) | Maya | Target night has an existing dinner-status Planned Meal | -- |
| Delete Planned Meal (clear a night) | Sam | Never | No Clear action exists in Sam's screens |
| Create Swap Suggestion (pick suggestion) | Sam | Sam has no other open ("Suggested") suggestion for the target night, the proposed recipe is eligible, and the week is within the one-week-ahead window | Ineligible night: the send action is rejected with "You already suggested a pick for {Night} -- wait for Maya's decision or its lapse." |
| Create Swap Suggestion (pick suggestion) | Maya | Never -- the organiser places directly and never suggests to herself | The suggestion screen (FEAT-23.SPEC-003) is not part of her navigation |
| Create/Update/Delete Planned Meal; Create Swap Suggestion | Jordan (young kid profile, no login -- MVP) | Never -- Manual Planning access is None | No login exists for this profile; none of these actions are reachable |
| Create/Update/Delete Planned Meal; Create Swap Suggestion | Jordan (older kid, limited login -- Later) | Never -- Manual Planning access is None | None of these actions are offered to this row |
| Create/Update/Delete Planned Meal; Create Swap Suggestion | Riley (Operator, support -- from v1) | Never -- Manual Planning access is None | None of these actions are offered to Riley |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Weekly Plan.origin | "manually built" | On creation, when FEAT-23.SPEC-001 auto-initializes a week that has not started | No |
| Weekly Plan.status | "Generated/Started" (entering as "Started") | On creation, when FEAT-23.SPEC-001 auto-initializes the week | No -- further transitions (Reviewed, Approved, Active, Archived) are FEAT-03's responsibility and are never applied by this feature |
| Planned Meal.status | "Proposed/Picked" (entering as "Picked") | On creation, when FEAT-23.SPEC-006 writes a new pick | No -- other statuses (Confirmed, Swapped, Removed, Cooked) are set by FEAT-11, FEAT-04, and FEAT-02 respectively |
| Swap Suggestion.outcome | "Suggested" | On creation, when FEAT-23.SPEC-003 sends a pick suggestion | No -- resolved outcomes (Accepted, Declined, Lapsed) are set by FEAT-04 |

## Business Rules

- XBR-06: at most one open suggestion per member per night; an unanswered suggestion lapses when its night passes and the suggester is told the outcome (the lapse mechanics themselves are owned by FEAT-04; this spec defines the creation-time limit that keeps a member from having two open suggestions on the same night at once).
- XBR-05: these limits apply uniformly regardless of subscription tier -- manual planning, and the limits governing it, are available on both the free and paid tiers.
- These limits are independent of recipe safety: a night can be structurally valid to pick (no existing dinner, week in range) and still be blocked from a specific recipe by FEAT-23.SPEC-004's separate safety check; both must pass for a placement or suggestion to succeed.

## Edge Cases

- **Maya attempts to place a pick on a night exactly seven days from today** -- Allowed; this is the boundary of "up to one week ahead," not beyond it.
- **Maya attempts to place a pick on a night eight days from today** -- Blocked with the week-limit error; no control for that night is shown in the first place, since FEAT-23.SPEC-001 only ever displays the current and next week.
- **Sam attempts to create a Swap Suggestion for the same member and night from two devices within the same moment** -- The first request commits; the second is rejected once it arrives, per first-decision-wins with reject-with-refresh (dependency map's Contention note for Swap Suggestion).
- **Maya places a direct pick on a night while Sam has an open suggestion for that same night** -- Per XBR-06 (owned by FEAT-04), the direct pick supersedes the open suggestion, which FEAT-04 then resolves rather than leaving open; this spec's one-open-suggestion-per-member-per-night rule does not block Maya's direct pick, since it governs only new suggestion creation, not the organiser's placement action.
- **A night's existing dinner is cleared and immediately re-picked in the same session** -- Treated as a fresh create; the one-dinner-per-night rule re-evaluates against the now-empty night and always passes.
- **The current week rolls over to become "last week" while a suggestion is still open for a night in it** -- The night has passed; FEAT-04 lapses the suggestion per XBR-06 rather than this spec extending its window.

## Acceptance Criteria

**FEAT-23.SPEC-005-AC-01:** Given Maya is building the current week, when she opens a night exactly seven days from today, then placing a pick on it is allowed.

**FEAT-23.SPEC-005-AC-02:** Given Maya wants to plan a night eight days from today, when she looks at the Weekly Plan screen, then no such night is shown or reachable, per the one-week-ahead window.

**FEAT-23.SPEC-005-AC-03:** Given Wednesday already has a placed dinner, when Maya attempts to place a second dinner on Wednesday through a create action rather than Change, then the attempt is rejected with "This night already has a dinner -- use Change to replace it."

**FEAT-23.SPEC-005-AC-04:** Given Wednesday already has a placed dinner, when Maya uses the Change flow to replace it, then the update succeeds and no one-dinner-per-night violation occurs.

**FEAT-23.SPEC-005-AC-05:** Given Sam already has an open suggestion for Saturday, when he attempts to send a second suggestion for Saturday, then the attempt is rejected with "You already suggested a pick for Saturday -- wait for Maya's decision or its lapse."

**FEAT-23.SPEC-005-AC-06:** Given Sam has no open suggestion for Sunday, when he sends an eligible recipe as a suggestion for Sunday, then the Swap Suggestion is created with outcome "Suggested."

**FEAT-23.SPEC-005-AC-07:** Given Sam (Other Adult Member) is on FEAT-23.SPEC-002 by way of a stale link, when he attempts to place a dinner directly, then no such action is available to him -- placement is never allowed for Sam.

**FEAT-23.SPEC-005-AC-08:** Given Maya (Organiser) attempts to reach the suggestion-send flow, when she does, then no create-suggestion action is available to her -- suggestions are never allowed for Maya.

**FEAT-23.SPEC-005-AC-09:** Given Jordan is a young kid profile with no login, when any attempt is made to place, change, clear, or suggest a pick on Jordan's behalf, then none of these actions is reachable, since Manual Planning access is None for this row.

**FEAT-23.SPEC-005-AC-10:** Given Maya opens a new week that has not started, when the Weekly Plan is auto-initialized, then its origin is set to "manually built" and its status enters as "Started," with no further status transition applied by this feature.

**FEAT-23.SPEC-005-AC-11:** Given Maya places a new pick, when FEAT-23.SPEC-006 writes it, then the Planned Meal's status is set to "Picked."

**FEAT-23.SPEC-005-AC-12:** Given Sam sends a new suggestion, when it is created, then its outcome is set to "Suggested," never any other value.

**FEAT-23.SPEC-005-AC-13:** Given Maya places a direct pick on a night where Sam has an open suggestion, when the placement completes, then Maya's placement succeeds and the open suggestion is resolved by FEAT-04 per XBR-06, rather than this spec's one-open-suggestion rule blocking Maya's action.

**FEAT-23.SPEC-005-AC-14:** Given Sam attempts to create a Swap Suggestion for the same night from two devices at effectively the same time, when the first request commits, then the second request is rejected once it arrives.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 7 | 7 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 10 | 10 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |
