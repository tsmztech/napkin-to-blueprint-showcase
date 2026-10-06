---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-17.SPEC-006
spec_name: Dinner Voting Rules -- Access, Validation & Conflict Resolution
spec_slug: dinner-voting-rules-access-validation-conflict-resolution
parent_feature: FEAT-17
parent_feature_name: Older-Kid Dinner Voting
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 30
acceptance_criteria_count: 17
---

# Logic/Rule Spec: Dinner Voting Rules -- Access, Validation & Conflict Resolution

## Overview

**Name:** Dinner Voting Rules -- Access, Validation & Conflict Resolution
**ID:** FEAT-17.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs who may open rounds, vote, view outcomes, or make the final call; enforces the option-count/safety-check and one-vote-per-profile-per-round limits; and rejects with refresh any vote cast after the round is already resolved.
**Parent Feature:** FEAT-17 -- Older-Kid Dinner Voting
**Governed Entity:** Dinner Vote

## Scope and Non-Goals

**In Scope:**
- Field validation rules for the Dinner Vote entity's round, voter, choice, and resolution fields
- Cross-field rules governing one vote per profile per round, choice-within-options, and the post-resolution rejection rule
- Authorization rules for opening a round, casting a vote, viewing outcomes, and making the final call, for every role in the Access Matrix
- Default and derived values on the Dinner Vote entity

**Non-Goals:**
- Determining whether a candidate recipe is itself safe for the household -- that determination belongs to Dietary Rules & Allergy Safety Engine (FEAT-02); this spec only requires that a round's options already carry that determination and re-confirms it does not change between selection and round creation (enforced by FEAT-17.SPEC-004).
- The processing logic that computes a tally or applies the no-vote fallback -- handled by FEAT-17.SPEC-005 (Vote Round Resolution & Fallback); this spec defines only the validation and authorization boundaries that logic must respect.
- UI presentation of validation errors and denied states -- defined by FEAT-17.SPEC-001, FEAT-17.SPEC-002, and FEAT-17.SPEC-003 (they reference this spec for the rules and own how each is displayed).

## Governed Entity

**Entity:** Dinner Vote
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| round | derived | The night and its 2-3 safety-checked options this vote (or the round it belongs to) is cast within |
| voter | text (reference) | The older-kid limited-login Member Profile that cast this vote |
| choice | enum | The option, among the round's own options, the voter selected |
| resolution | enum/derived | The round's outcome: unset while Open, or the resolved option once set by tally or by Maya's final call |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-17.SPEC-001 | Dinner Vote Casting | On vote submission (tap); authorization on screen entry (older-kid role and round-open state) |
| FEAT-17.SPEC-002 | Voting Round Setup | On option selection and Open Vote submission; authorization on screen entry (Maya only) |
| FEAT-17.SPEC-003 | Vote Outcome & Resolution | On final-call submission; authorization on screen entry for the final-call control (Maya only) |
| FEAT-17.SPEC-004 | Voting Round Creation & Safety Validation | During round-creation processing (option count, safety re-check, round uniqueness) |
| FEAT-17.SPEC-005 | Vote Round Resolution & Fallback | During tally/resolution processing; the post-resolution rejection check on each incoming vote |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| round.night | Required; must reference a night in the household's current Weekly Plan | Always | On round creation | "Choose a night from this week's plan." | Yes |
| round.options | Required; exactly 2 or 3 options, each currently passing FEAT-02's safety check | Always | On round creation | "Choose 2 to 3 options." / "{Option name} no longer passes the allergy check and can't be offered." | Yes |
| voter | Required; must be an older-kid limited-login Member Profile belonging to the household | Always | On vote cast | "Only an older kid's own login can vote." | Yes |
| choice | Required; must equal one of the round's own options | Always | On vote cast | "Choose one of the offered options." | Yes |
| resolution | No validation beyond data type -- never entered directly by any role; set only by FEAT-17.SPEC-005's tally logic or by Maya's final call | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| One vote per voter per round | voter, round | The same older-kid profile cannot cast a second vote in the same round | "You've already voted for {night}." |
| Choice must be among round options | choice, round.options | A cast vote's choice must equal one of the round's own 2-3 options | "That option isn't part of this vote." |
| No vote after resolution | round, resolution | A vote cannot be accepted once round.resolution is set | Not a form error -- the submission is rejected with refresh, showing FEAT-17.SPEC-003's resolved outcome (see Business Rules) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Open a voting round | Maya (Organiser) | Always | -- |
| Open a voting round | Sam (Other Adult Member) | Never | FEAT-17.SPEC-002 is not shown; no entry point exists for Sam to reach it |
| Open a voting round | Jordan (young kid profile, no login) | Never | No login exists; nothing is shown |
| Open a voting round | Jordan (older kid, limited login) | Never | FEAT-17.SPEC-002 is not shown; the older-kid row's Dinner Voting access is Own-only (casting a vote), never Full |
| Open a voting round | Riley (Operator, support) | Never | Dinner Voting: None; not exposed through support access |
| Cast a vote | Jordan (older kid, limited login) | Only their own vote, only in a round that is still Open, and only once per round | Voting in a resolved round is rejected with refresh, showing FEAT-17.SPEC-003's outcome instead; a second vote attempt in the same open round shows "You've already voted for {night}." and the option cards become non-interactive |
| Cast a vote | Maya, Sam, Jordan (young kid, no login), Riley | Never | FEAT-17.SPEC-001 is not shown to these roles -- Dinner Voting's casting capability is Own-only strictly for the older-kid row |
| View a round's tally/outcome | Maya (Organiser) | Always | -- |
| View a round's tally/outcome | Sam (Other Adult Member) | Always, read-only | -- |
| View a round's tally/outcome | Jordan (older kid, limited login) | Always, read-only | -- |
| View a round's tally/outcome | Jordan (young kid profile, no login) | Never | No login exists; nothing is shown |
| View a round's tally/outcome | Riley (Operator, support) | Never | Dinner Voting: None; not exposed through support access |
| Make the final call on a split round | Maya (Organiser) | Only while the round is Split (not yet resolved) | Attempting a final call on an already-resolved round is rejected with refresh, showing the existing resolution instead |
| Make the final call on a split round | Sam, Jordan (either row), Riley | Never | The final-call control is never shown to these roles |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| round.status | Defaults to "Open" | On round creation | No |
| round.resolution | Unset until an auto-resolve condition is met (all eligible voters have voted unanimously), the no-vote fallback point is reached, or Maya makes a final call -- derived per FEAT-17.SPEC-005's processing logic | On tally recompute / resolution-point trigger / final call | No -- system-derived, or set by Maya's own final-call action, never entered as a raw value by any role |
| round.status -> "Resolved" | Derived automatically the moment resolution is set | Same triggers as resolution above | No |

## Business Rules

- XBR-01: every option a round ever offers has already passed the same app-enforced allergy check before anyone sees it, and that check fails closed (an option with an incomplete or failing safety determination can never be offered).
- At most one round (open or resolved) exists per night (Non-Functional Notes: data volumes) -- enforced by FEAT-17.SPEC-004's round-uniqueness check.
- A vote is immutable once cast -- only the round's resolution can subsequently change (Entity-Lifecycle Coverage Matrix, Update row); no role, including Maya, can edit a cast vote's choice.
- A vote cast after the round is already resolved is rejected with refresh, showing the resolved outcome instead (dependency map's Dinner Vote Contention note) -- this rule applies uniformly regardless of how the round resolved (unanimous tally, no-vote fallback, or Maya's final call). Resolution: reject-with-refresh.
- The app never arbitrarily decides a genuine split; resolution authority for a split rests solely with Maya (Brief's Non-Goals).

## Edge Cases

- **A vote arrives at the exact moment the round transitions to Resolved** -- The arriving vote is evaluated against the round's post-transition state and rejected with refresh, per the post-resolution rule; it is never allowed to reopen a resolved round.
- **An older-kid profile is removed from the household (FEAT-18) after casting a vote but before the round resolves** -- The vote is retained as cast (votes are neither deleted nor archived, per this feature's Non-Goals); it still counts toward the tally, since removing the voter does not retroactively invalidate a vote already cast in an open round.
- **Exactly two older-kid profiles both vote for the same option** -- This is the boundary case for "unanimous" with more than one voter; the round resolves as soon as the second (and final eligible) vote agrees with the first.
- **A round is opened with exactly 2 options (the minimum)** -- All rules apply identically to a 2-option and a 3-option round; a split on a 2-option round means exactly one vote for each option.
- **Maya attempts to open a round for a night that already has a Resolved round** -- Rejected by FEAT-17.SPEC-004's round-uniqueness check; a resolved round is never reopened, and Maya is shown the existing resolution instead.
- **An option selected during round setup fails its safety re-check between selection and Open Vote submission** -- The round is not created; FEAT-17.SPEC-004 rejects the submission naming the failing option, consistent with the fail-closed rule (see Field Validation Rules, round.options).

## Acceptance Criteria

**FEAT-17.SPEC-006-AC-01:** Given Maya is opening a round, when she selects a night not in the household's current Weekly Plan, then the round creation is blocked with "Choose a night from this week's plan."

**FEAT-17.SPEC-006-AC-02:** Given Maya selects only 1 option for a round, when she submits it, then the round creation is blocked with "Choose 2 to 3 options."

**FEAT-17.SPEC-006-AC-03:** Given Maya selects an option that no longer passes FEAT-02's safety check, when she submits the round, then the round creation is blocked with "{Option name} no longer passes the allergy check and can't be offered."

**FEAT-17.SPEC-006-AC-04:** Given Jordan (older kid) casts a vote, when the vote is submitted, then the voter field is verified to be an older-kid limited-login Member Profile of the household before the vote is accepted.

**FEAT-17.SPEC-006-AC-05:** Given a cast vote's choice does not match any of the round's own options, when the submission is checked, then it is rejected with "That option isn't part of this vote."

**FEAT-17.SPEC-006-AC-06:** Given Jordan has already voted in an open round, when Jordan attempts to cast a second vote in the same round, then it is rejected with "You've already voted for {night}."

**FEAT-17.SPEC-006-AC-07:** Given a round has already resolved, when any vote is submitted against it, then it is rejected with refresh and the voter is shown FEAT-17.SPEC-003's resolved outcome instead of a form error.

**FEAT-17.SPEC-006-AC-08:** Given Maya attempts to open a voting round, when the action is checked, then it is allowed always, since Maya is the sole role authorized to open a round.

**FEAT-17.SPEC-006-AC-09:** Given Sam attempts to reach the round-opening screen, when he navigates the app, then no entry point to FEAT-17.SPEC-002 is shown to him.

**FEAT-17.SPEC-006-AC-10:** Given Jordan (older kid) attempts to reach the round-opening screen, when Jordan navigates the app, then no entry point to FEAT-17.SPEC-002 is shown, consistent with the older-kid row's Own-only (not Full) Dinner Voting access.

**FEAT-17.SPEC-006-AC-11:** Given Jordan (older kid) is on an open round they have not yet voted in, when Jordan casts a vote, then it is accepted.

**FEAT-17.SPEC-006-AC-12:** Given Maya, Sam, Jordan (young kid profile, no login), or Riley attempts to reach the vote-casting screen, when they navigate the app, then no entry point to FEAT-17.SPEC-001 is shown to any of them.

**FEAT-17.SPEC-006-AC-13:** Given Sam or Jordan (older kid) views a round's tally, when they open FEAT-17.SPEC-003, then they see the tally or resolution read-only, with no final-call control.

**FEAT-17.SPEC-006-AC-14:** Given Riley or Jordan (young kid profile, no login) attempts to view a round's outcome, when they navigate the app, then no entry point to FEAT-17.SPEC-003 is shown.

**FEAT-17.SPEC-006-AC-15:** Given Maya views a Split round, when she makes the final call, then it is accepted and the round resolves to her choice.

**FEAT-17.SPEC-006-AC-16:** Given Maya attempts to make a final call on a round that has already resolved, when she submits it, then it is rejected with refresh, showing the existing resolution instead of applying a new one.

**FEAT-17.SPEC-006-AC-17:** Given Sam, Jordan (either row), or Riley views a Split round, when they look for a final-call control, then none is shown to any of them.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 14 | 14 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
