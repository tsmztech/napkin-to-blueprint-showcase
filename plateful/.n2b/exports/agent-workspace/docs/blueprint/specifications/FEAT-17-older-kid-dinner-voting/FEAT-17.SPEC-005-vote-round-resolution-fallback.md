---
document_type: spec
spec_type: automation
spec_id: FEAT-17.SPEC-005
spec_name: Vote Round Resolution & Fallback
spec_slug: vote-round-resolution-fallback
parent_feature: FEAT-17
parent_feature_name: Older-Kid Dinner Voting
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Vote Round Resolution & Fallback

## Overview

**Name:** Vote Round Resolution & Fallback
**ID:** FEAT-17.SPEC-005
**Type:** Automation
**Purpose:** Recomputes the tally on each cast vote, auto-resolves a unanimous round, defaults to the AI's original suggestion when the plan's resolution point arrives with no vote or with only a partial vote cast, writes the outcome onto the Weekly Plan, and fires dinner_vote_resolved.
**Parent Feature:** FEAT-17 -- Older-Kid Dinner Voting

## Scope and Non-Goals

**In Scope:**
- Recomputing a round's vote tally each time a vote is cast
- Detecting unanimous agreement among all eligible voters and auto-resolving to that option
- Flagging a round as Split when eligible voters disagree, so Maya can make the final call
- Applying the no-vote/partial-vote fallback to the AI's original suggestion when the plan's resolution point arrives with zero votes, or with some but not all eligible voters' votes cast and no unanimity reached
- Applying Maya's final call as the round's resolution when she makes one
- Writing the resolved option onto the Weekly Plan's night slot and firing dinner_vote_resolved

**Non-Goals:**
- Creating the round or validating its option count and safety check -- handled by FEAT-17.SPEC-004 (Voting Round Creation & Safety Validation); this automation only ever operates on a round that already exists.
- Displaying the tally or capturing Maya's final-call input -- owned by FEAT-17.SPEC-003 (Vote Outcome & Resolution); this automation supplies the computed result and receives the final-call trigger, but does not itself render any screen.
- The app arbitrarily deciding a tie -- excluded per the Brief's Non-Goals; a genuine split is never auto-resolved by this automation and is always routed to Maya.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A vote is cast | FEAT-17.SPEC-001 (Dinner Vote Casting) | Fires every time an older kid casts a vote in an open round | Round reference, the round's current votes, the newly cast vote (voter, choice) |
| The night's plan resolution point is reached | FEAT-03 (AI Weekly Dinner Plan Generation) -- the plan's approval/adoption timing (XBR-07) | Fires when the night's plan resolution point arrives and the round is not yet Resolved (whether it has zero votes cast, or some but not all eligible voters' votes cast) | Round reference, the AI's original suggested option for the night, the count of eligible voters versus votes cast |
| Maya makes her final call | FEAT-17.SPEC-003 (Vote Outcome & Resolution) | Fires when Maya selects an option on a round flagged Split | Round reference, Maya's chosen option |

## Processing Logic

1. On a vote-cast trigger: add the newly cast vote to the round's tally and recompute the vote count per option.
2. Determine whether every older-kid Member Profile eligible to vote in this round (the household's older-kid limited logins) has now cast a vote.
3. If all eligible voters have voted and every cast vote agrees on the same option, treat the round as unanimous and proceed to resolve to that option (step 6).
4. If all eligible voters have voted and the votes disagree, flag the round as Split; do not resolve it -- the current tally remains visible and no further processing occurs until Maya makes a final call.
5. If not all eligible voters have voted yet, leave the round Open with the updated tally; no resolution occurs from this trigger unless and until the resolution-point trigger (steps 6-7) or Maya's final call (step 8) fires for this round.
6. On the resolution-point trigger, if the round still has zero votes cast when this trigger fires: resolve to the AI's original suggested option for the night, recording "no votes cast -- kept the original suggestion" as the resolution basis.
7. On the resolution-point trigger, if the round has some but not all eligible voters' votes cast when this trigger fires (unanimity was never reached and the round is not already Resolved): resolve to the AI's original suggested option for the night, recording "partial votes cast -- kept the original suggestion" as the resolution basis. This applies regardless of how many options the partial votes are split across, since resolution authority for an incomplete vote is never given to the app arbitrarily picking among the cast votes.
8. On Maya's final-call trigger: set the resolution to Maya's chosen option, recording "organiser's final call" as the resolution basis.
9. Whenever a resolution is set (from step 3, step 6, step 7, or step 8): re-run FEAT-02's safety check on the resolved option per XBR-01, write the resolved option onto the Weekly Plan's night slot, set the round's status to Resolved, and fire dinner_vote_resolved.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Tally updated, still Open | A vote is cast and not all eligible voters have voted yet | Round's vote tally updated | Vote counts refresh on FEAT-17.SPEC-003 | FEAT-17.SPEC-003 |
| Auto-resolved -- unanimous | All eligible voters have voted and agree on one option | Dinner Vote resolution set to the agreed option; Weekly Plan night slot updated | "Chosen" badge and resolution basis shown on FEAT-17.SPEC-003; the voting older kid sees "Your vote is in!" already reflected | FEAT-17.SPEC-001, FEAT-17.SPEC-003 |
| Split -- awaiting Maya | All eligible voters have voted and disagree | None (resolution remains unset) | FEAT-17.SPEC-003 shows "Split -- awaiting the final call" with "Make the final call" for Maya | FEAT-17.SPEC-003 |
| Resolved -- no votes cast (fallback) | The plan's resolution point is reached with zero votes cast | Dinner Vote resolution set to the AI's original suggestion; Weekly Plan night slot unchanged (it already held that suggestion) | "No votes cast -- kept the original suggestion." shown on FEAT-17.SPEC-003 | FEAT-17.SPEC-003 |
| Resolved -- partial votes, deadline reached (fallback) | The plan's resolution point is reached with some but not all eligible voters' votes cast, and unanimity was never reached | Dinner Vote resolution set to the AI's original suggestion; Weekly Plan night slot unchanged (it already held that suggestion) | "Partial votes cast -- kept the original suggestion." shown on FEAT-17.SPEC-003 | FEAT-17.SPEC-003 |
| Resolved -- organiser's final call | Maya selects an option on a Split round | Dinner Vote resolution set to Maya's choice; Weekly Plan night slot updated | Confirmation "Vote resolved -- {option} it is!" and "Chosen" badge shown on FEAT-17.SPEC-003 | FEAT-17.SPEC-003 |
| Automation failure | Processing error during tally recomputation or the resolution write | No resolution change persists -- a partial write never leaves the plan or the Dinner Vote record inconsistent | FEAT-17.SPEC-003 shows "Could not update the vote. Try again." | FEAT-17.SPEC-003 |

## Data Model

**Reads:** Dinner Vote -- the round and every cast vote's choice; Member Profile -- to determine which older-kid limited logins in the household are eligible voters for this round; Weekly Plan -- the night's AI original suggestion, for the no-vote and partial-vote fallback paths.
**Creates:** None.
**Updates:** Dinner Vote -- the resolution field (and status, to Resolved); Weekly Plan -- the night's slot, set to the resolved option (per the dependency map: "Weekly Plan Updated by FEAT-17, vote outcome").
**Deletes:** None.

## Business Rules

- XBR-01: the resolution write, like every other path onto the plan, carries the same safety check and badge before anyone sees it.
- Voting never blocks the plan: a night with no votes, or with an incomplete vote still open at the resolution point, proceeds with the AI's original suggestion rather than staying Open indefinitely (Brief's Primary Flows & Alternates).
- The app never arbitrarily decides a genuine split -- resolution authority for a split rests solely with Maya (Brief's Non-Goals).
- Once a round is Resolved, its resolution is never recomputed by a later vote; FEAT-17.SPEC-006 rejects any vote cast afterward.

## Edge Cases

- **A vote is cast at the same moment the no-vote resolution-point trigger fires for the same round** -- The vote-cast trigger and the resolution-point trigger for the same round are mutually exclusive once either completes: whichever this automation receives first processes to completion (updating the tally or resolving the round) before the other is evaluated; if the resolution-point trigger finds the round already resolved by the just-processed vote, it takes no action. This is the concurrent-trigger-firing entry.
- **A second vote is cast while this automation is still processing the first vote's tally recomputation for the same round** -- The second run for the same round queues behind the first run's completion rather than reading a stale tally, so the outcome always reflects both votes. This is the trigger-fires-while-a-previous-run-is-in-flight entry.
- **Maya makes her final call at the same moment a new vote is cast for the same round** -- The final-call trigger and the vote-cast trigger for the same round are serialized the same way as the two entries above; whichever completes first resolves the round, and the other is rejected per FEAT-17.SPEC-006's post-resolution rule (the vote is shown "This round resolved on its own" if it loses the race, or the final call succeeds normally otherwise).
- **All eligible older-kid profiles vote for different options in a 3-option round with 2 voters** -- The round is Split with no unanimous outcome and is routed to Maya, regardless of how many distinct options are represented.
- **The household has only one older-kid limited login** -- "Unanimous" reduces to that single vote; the round auto-resolves as soon as that one vote is cast, since all eligible voters (one) have now voted and agree with themselves.
- **A household has two or more older-kid limited logins and only some of them have voted by the time the plan's resolution point arrives** -- The round never stays Open past this trigger waiting for the remaining voters: it resolves to the AI's original suggested option via the partial-vote fallback (step 7), the same way a zero-vote round does, so no round is left unresolved indefinitely.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-001 (Dinner Vote Casting) | Triggered by (inbound); Affects (outbound) | Fires on each cast vote; the resulting resolution feedback appears there for the voter |
| FEAT-17.SPEC-003 (Vote Outcome & Resolution) | Triggered by (inbound, Maya's final call); Affects (outbound) | Displays the computed tally and resolution; receives Maya's final call |
| FEAT-17.SPEC-004 (Voting Round Creation & Safety Validation) | References (inbound) | Operates only on rounds this automation has already created |
| FEAT-17.SPEC-006 (Dinner Voting Rules) | References (inbound) | Post-resolution rejection rule; authorization for the final call |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Affects (outbound); References (inbound) | Receives the resolved option written onto the Weekly Plan's night slot; supplies the AI's original suggestion used by the fallback path |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | References (inbound) | Safety check and badge re-applied to the resolution write (XBR-01) |

## Analytics and Success Signals

- **dinner_vote_resolved** (night, resolution basis: unanimous / no_votes_fallback / partial_votes_fallback / organiser_final_call, option chosen) -- N/A -- no metric in success-metrics.md carries Older-Kid Dinner Voting (FEAT-17) as its Connected Feature; this Later-phase feature has no metric of its own in the current success-metrics set.
- **dinner_vote_split_flagged** (night, tied options) -- N/A -- same reason as above.

## Acceptance Criteria

**FEAT-17.SPEC-005-AC-01:** Given Jordan (older kid) casts a vote and other eligible older-kid profiles have not yet voted, when this automation processes the vote, then the round's tally updates and remains Open.

**FEAT-17.SPEC-005-AC-02:** Given the household has one older-kid limited login and that older kid casts a vote, when this automation processes it, then the round auto-resolves unanimously to the voted option, the Weekly Plan's night slot updates, and dinner_vote_resolved fires.

**FEAT-17.SPEC-005-AC-03:** Given a household has two older-kid limited logins and both have now voted for the same option, when the second vote is processed, then the round auto-resolves to that option.

**FEAT-17.SPEC-005-AC-04:** Given a household has two older-kid limited logins and they vote for different options, when both votes have been processed, then the round is flagged Split and no resolution is set.

**FEAT-17.SPEC-005-AC-05:** Given a night's round has zero votes cast when the plan's resolution point arrives, when this automation runs, then the round resolves to the AI's original suggested option with resolution basis "no votes cast -- kept the original suggestion," and dinner_vote_resolved fires.

**FEAT-17.SPEC-005-AC-06:** Given Maya makes her final call on a Split round, when this automation processes it, then the round resolves to Maya's chosen option, the Weekly Plan's night slot updates, and dinner_vote_resolved fires with resolution basis "organiser's final call."

**FEAT-17.SPEC-005-AC-07:** Given a resolution is about to be written, when this automation applies it, then FEAT-02's safety check and badge are re-applied to the resolved option before it appears on the Weekly Plan (XBR-01).

**FEAT-17.SPEC-005-AC-08:** Given a vote is cast and the no-vote resolution-point trigger fires for the same round at effectively the same moment, when this automation processes both, then whichever it receives first resolves the round (or updates the tally) and the other trigger takes no further action once the round is Resolved.

**FEAT-17.SPEC-005-AC-09:** Given a second vote is cast for a round while this automation is still processing the first vote's tally recomputation, when the second trigger arrives, then it queues behind the first run's completion and the final tally reflects both votes.

**FEAT-17.SPEC-005-AC-10:** Given Maya submits her final call at the same moment a new vote is cast for the same Split round, when this automation processes both, then whichever completes first resolves the round and the other is rejected per FEAT-17.SPEC-006's post-resolution rule.

**FEAT-17.SPEC-005-AC-11:** Given a household has two older-kid limited logins and only one of them has voted when the plan's resolution point arrives, when this automation runs, then the round resolves to the AI's original suggested option with resolution basis "partial votes cast -- kept the original suggestion," and dinner_vote_resolved fires, rather than the round remaining Open.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 7 | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
