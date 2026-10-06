---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-04.SPEC-010
spec_name: Suggestion Lifecycle Rules
spec_slug: suggestion-lifecycle-rules
parent_feature: FEAT-04
parent_feature_name: One-Tap Meal Swap
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 22
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Suggestion Lifecycle Rules

## Overview

**Name:** Suggestion Lifecycle Rules
**ID:** FEAT-04.SPEC-010
**Type:** Logic/Rule
**Purpose:** Governs the full lifecycle of a Swap Suggestion -- one open suggestion per member per slot, a direct swap or accepted suggestion superseding any other open suggestion for that slot, and refusal of a late accept on a lapsed suggestion.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap
**Governed Entity:** Swap Suggestion

## Scope and Non-Goals

**In Scope:**
- Field validation and authorization for every action on the Swap Suggestion entity: create, accept, decline, supersede, lapse
- The one-open-suggestion-per-member-per-slot limit
- The superseding rule when a direct swap or another accepted suggestion resolves the same slot
- The late-accept refusal on an already-lapsed suggestion

**Non-Goals:**
- Deciding which recipes qualify as a suggestion's proposed_recipe -- owned by FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation); this spec governs the suggestion record itself, not candidate eligibility
- Detecting that a night has passed and initiating the lapse -- owned by FEAT-04.SPEC-005 (Suggestion Lapse), which calls into this spec to record the outcome once it has made that determination
- Performing the swap write when a suggestion is accepted -- owned by FEAT-04.SPEC-004 (Apply Meal Swap); this spec only sets the suggestion's own outcome field
- The equivalent "pick suggestion" flow in Manual Weekly Planning (FEAT-23) -- FEAT-23 creates suggestions against the same Swap Suggestion entity and is subject to these same rules, but its screen mechanics are owned by that feature, not this spec

## Governed Entity

**Entity:** Swap Suggestion
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| suggesting_member | reference | The other adult member who raised the suggestion (required) |
| night | reference | The target slot -- one open suggestion per member per night (required) |
| proposed_recipe | reference | A safety-checked recipe (required) |
| outcome | enum | Suggested, Accepted, Declined, Lapsed (once the night passes) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-04.SPEC-002 | Suggest a Swap | On submission -- create action, one-open-per-member-per-slot check |
| FEAT-04.SPEC-003 | Review Swap Suggestions | On accept and decline actions |
| FEAT-04.SPEC-004 | Apply Meal Swap | On successful accept -- sets outcome to Accepted; triggers superseding of any other open suggestion on the slot |
| FEAT-04.SPEC-005 | Suggestion Lapse | On the nightly lapse check -- sets outcome to Lapsed |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| suggesting_member | Required; must be the currently signed-in Other Adult Member | Always, on create | On submit | "A suggestion must come from a signed-in household member." | Yes |
| night | Required; must reference an existing slot in the current Weekly Plan that is not already Cooked, Removed, or otherwise closed | Always, on create | On submit | "This night's dinner has already happened or was removed -- choose a different night." | Yes |
| proposed_recipe | Required; must be a member of the qualifying alternatives set from FEAT-04.SPEC-008 at submission time | Always, on create | On submit | "This recipe isn't a safe or available option for this night. Choose from the list shown." | Yes |
| outcome | Required; must be one of Suggested, Accepted, Declined, Lapsed; set to Suggested automatically on create and never chosen by the user directly | Always | On create (default) and on every subsequent transition | N/A -- outcome is system-managed, not user-entered | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| One open suggestion per member per slot | suggesting_member, night, outcome | A member may not create a new suggestion for a night where they already hold a suggestion with outcome = Suggested | "You already have a pending suggestion for this night." (in practice, FEAT-04.SPEC-002 never shows the create flow for a slot with an open suggestion from the same member, so this message covers only a direct or replayed submission attempt) |
| Outcome transitions are one-way and terminal | outcome, night | Once outcome reaches Accepted, Declined, or Lapsed, no further transition is possible for that suggestion record; a new suggestion is a new record | "This suggestion has already been resolved." |
| A resolved suggestion's proposed_recipe becomes immutable | proposed_recipe, outcome | Once outcome leaves Suggested, proposed_recipe can never be changed -- it stands as the historical record of what was suggested | N/A -- no edit path exists on a resolved suggestion |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create suggestion | Sam (Other Adult Member) | Own-only -- suggesting_member is always the requesting member; no more than one open suggestion for the same night | If an open suggestion already exists for that night: the create flow is not shown (FEAT-04.SPEC-002 shows the "Your suggestion" pending card instead); a direct create attempt returns "You already have a pending suggestion for this night." |
| Create suggestion | Maya (Organiser) | Never -- Maya's swap authority is direct (FEAT-04.SPEC-001, FEAT-04.SPEC-004), not through a suggestion | The suggestion-create flow is not shown to Maya anywhere in the product |
| Create suggestion | Jordan (young kid, no login), Jordan (older kid, limited login), Riley (Operator) | Never | No path to create a suggestion is shown to any of these roles; Meal Swap access is None for all three |
| Accept suggestion | Maya (Organiser) | Only while outcome = Suggested and the target slot's Planned Meal has not been superseded by another completed action in the interim (re-verified by FEAT-04.SPEC-004's safety re-check and FEAT-04.SPEC-009's lock) | If outcome is not Suggested (already resolved or lapsed): "This suggestion has already been resolved." shown on the card, and the card is removed from the pending list |
| Accept suggestion | Sam (Other Adult Member) | Never | The accept control is never shown to Sam; only Maya sees FEAT-04.SPEC-003 |
| Decline suggestion | Maya (Organiser) | Only while outcome = Suggested | Same as Accept above -- "This suggestion has already been resolved." if already resolved |
| Decline suggestion | Sam (Other Adult Member) | Never | The decline control is never shown to Sam |
| Withdraw suggestion | No role | Never -- not modeled anywhere in the product (Brief, Non-Goals) | No withdrawal control exists for any role; a member who wants to change a suggestion waits for a decision or its lapse |
| Supersede suggestion (system-driven, not a direct user action) | System, triggered by Maya's direct swap (FEAT-04.SPEC-001/FEAT-04.SPEC-004) or her acceptance of a different suggestion on the same slot | Only while outcome = Suggested on the suggestion being superseded | The superseded suggestion's outcome becomes Declined automatically; the suggesting member receives the "declined"-style notification (FEAT-04.SPEC-006) |
| Mark suggestion Lapsed (system-driven) | System, triggered by FEAT-04.SPEC-005 | Only while outcome = Suggested and the suggestion's night has passed | N/A -- not a user-initiated action; see FEAT-04.SPEC-005 for the trigger logic |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| outcome | Set to Suggested | On create | No |
| suggesting_member | Set to the currently signed-in Other Adult Member | On create | No |

## Business Rules

- **One open suggestion per member per slot (XBR-06):** counted only against outcome = Suggested records; a member whose prior suggestion for that slot has already resolved (Accepted, Declined, or Lapsed) may submit a new one for a future occurrence of that slot without restriction.
- **Direct swap supersedes any open suggestion for the same slot:** when Maya applies a direct swap (FEAT-04.SPEC-001/FEAT-04.SPEC-004), any suggestion with outcome = Suggested targeting that same slot is set to Declined automatically, regardless of who raised it, per the dependency map's Swap Suggestion Contention note ("a direct swap by Maya on the same slot supersedes any open suggestion for it").
- **Accepting one suggestion supersedes any other open suggestion on the same slot:** if two members each have an open suggestion for the same night (a genuinely possible state, since the one-per-member limit is per member, not per slot), accepting either one supersedes the other via the same rule above.
- **Late accept on a lapsed suggestion is refused:** first-decision-wins with reject-with-refresh (dependency map, Swap Suggestion Contention) -- if Maya's accept and the nightly lapse check race, whichever resolves the outcome first stands; an accept attempt against an already-Lapsed record is refused with "This suggestion has already been resolved."
- **No delete, ever:** a suggestion always ends in a terminal outcome (Accepted, Declined, or Lapsed) and is retained as part of the plan's permanent history, consistent with weekly-plan history retention for the life of the account (scope-boundaries.md SC-18); this is a deliberate non-goal, not an omission (Brief, Entity-Lifecycle Coverage Matrix).
- **A suggestion's proposed_recipe passing the alternatives filter at submission time does not guarantee it still passes at accept time:** the safety re-check on accept (FEAT-04.SPEC-004) is the authoritative, final gate; this spec's create-time validation only ensures the suggestion was legitimate when raised.

## Edge Cases

- **Two members each hold an open suggestion for the same night** -- Both exist simultaneously (the one-per-member limit does not prevent this); Maya's Review Swap Suggestions screen (FEAT-04.SPEC-003) shows both as separate cards, and accepting either supersedes the other per Business Rules.
- **Maya accepts a suggestion at the exact moment its night passes and the lapse check runs** -- First-decision-wins with reject-with-refresh: whichever transition (accept or lapse) commits first stands, and the other attempt finds the record already resolved and takes no action, per the Cross-Field Rule that outcome transitions are terminal.
- **Sam submits a suggestion for a slot, then a mid-week hard rule change makes the proposed recipe unsafe before Maya reviews it** -- The suggestion itself is unaffected (still outcome = Suggested); the unsafety is caught at accept time by FEAT-04.SPEC-004's mandatory re-check, which refuses the write and shows "This suggestion no longer passes the household's dietary rules and can't be applied." on FEAT-04.SPEC-003 -- the suggestion's own outcome remains Suggested until Maya explicitly declines it or it later lapses.
- **A slot's Weekly Plan is itself replaced or regenerated while a suggestion targeting one of its nights is open** -- Not modeled as a distinct case: the Weekly Plan for a given week is a single record that is updated, not replaced, throughout the week (per the dependency map's Weekly Plan Lifecycle); an open suggestion continues to target the correct slot as long as that Weekly Plan record persists for the week.
- **A member is removed from the household (FEAT-18) while holding an open suggestion** -- Per XBR-16, removing a member affects future planning but does not retroactively rewrite history; the suggestion is retained with its suggesting_member reference intact as a historical record, and since the member no longer has access, it can no longer be accepted through the normal notification path -- Maya can still see and decline it, or it lapses normally when its night passes.
- **An accept and a decline are attempted on the same suggestion at effectively the same time (two taps from Maya on two devices)** -- Outcome transitions are terminal and one-way (Cross-Field Rules): whichever commits first wins, and the second attempt finds the record already resolved and is refused with "This suggestion has already been resolved."

## Acceptance Criteria

**FEAT-04.SPEC-010-AC-01:** Given Sam has no open suggestion for Thursday, when he submits one, then a Swap Suggestion is created with outcome Suggested, suggesting_member set to Sam, and the given night and proposed_recipe.

**FEAT-04.SPEC-010-AC-02:** Given Sam already has an open suggestion for Thursday, when he attempts to submit a second one for the same night, then FEAT-04.SPEC-002 shows the existing pending card instead of a create flow.

**FEAT-04.SPEC-010-AC-03:** Given Maya reviews a pending suggestion, when she accepts it, then its outcome is set to Accepted following a successful FEAT-04.SPEC-004 apply.

**FEAT-04.SPEC-010-AC-04:** Given Maya reviews a pending suggestion, when she declines it, then its outcome is set to Declined and Sam is notified.

**FEAT-04.SPEC-010-AC-05:** Given Maya applies a direct swap on a slot with an open suggestion from Sam, when the swap completes, then Sam's suggestion is superseded and its outcome is set to Declined.

**FEAT-04.SPEC-010-AC-06:** Given two members each have an open suggestion for the same night, when Maya accepts one, then the other is superseded and its outcome is set to Declined.

**FEAT-04.SPEC-010-AC-07:** Given a suggestion has already lapsed, when Maya attempts to accept it, then the accept is refused with "This suggestion has already been resolved."

**FEAT-04.SPEC-010-AC-08:** Given a suggestion's night passes with no accept or decline, when FEAT-04.SPEC-005's nightly check runs, then this spec sets its outcome to Lapsed.

**FEAT-04.SPEC-010-AC-09:** Given no withdrawal action exists for any role, when Sam wants to change a submitted suggestion, then he finds no control to do so and must wait for Maya's decision or the suggestion's lapse.

**FEAT-04.SPEC-010-AC-10:** Given Sam attempts to create a suggestion for a night whose Planned Meal is already Cooked, when he submits it, then the creation is refused with "This night's dinner has already happened or was removed -- choose a different night."

**FEAT-04.SPEC-010-AC-11:** Given Sam attempts to propose a recipe that is not in the current qualifying alternatives set, when he submits it, then the creation is refused with "This recipe isn't a safe or available option for this night. Choose from the list shown."

**FEAT-04.SPEC-010-AC-12:** Given a suggestion has resolved to Accepted, when any attempt is made to change its proposed_recipe, then no edit path exists and the record stands as the historical suggestion.

**FEAT-04.SPEC-010-AC-13:** Given an accept and a decline are attempted on the same suggestion from two devices at effectively the same time, when both reach this spec, then whichever commits first stands and the second is refused with "This suggestion has already been resolved."

**FEAT-04.SPEC-010-AC-14:** Given Sam is removed from the household while holding an open suggestion, when Maya later views it, then it is still shown with his name intact and can be declined or left to lapse normally.

**FEAT-04.SPEC-010-AC-15:** Given a suggestion resolves (any outcome), when the record is examined afterward, then it has not been deleted -- it remains as part of the plan's permanent history.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
