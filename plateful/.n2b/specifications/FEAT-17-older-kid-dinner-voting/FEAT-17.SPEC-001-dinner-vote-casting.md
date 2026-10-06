---
document_type: spec
spec_type: screen
spec_id: FEAT-17.SPEC-001
spec_name: Dinner Vote Casting
spec_slug: dinner-vote-casting
parent_feature: FEAT-17
parent_feature_name: Older-Kid Dinner Voting
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Dinner Vote Casting

## Overview

**Name:** Dinner Vote Casting
**ID:** FEAT-17.SPEC-001
**Type:** Screen
**Purpose:** An older kid views the 2-3 safety-checked dinner options for an open voting round on a given night and casts one preference.
**Parent Feature:** FEAT-17 -- Older-Kid Dinner Voting

## Scope and Non-Goals

**In Scope:**
- Displaying an open voting round's 2-3 safety-checked options for a given night
- Casting one vote for a preferred option, as a single instant tap
- Showing the safety-checked option card (recipe name, "checked against allergies" badge, "always check labels" disclaimer) consistently with FEAT-17.SPEC-002 and FEAT-17.SPEC-003
- Handling submission failure (automatic retry) and offline queuing of a cast vote

**Non-Goals:**
- Opening a voting round -- handled by FEAT-17.SPEC-002 (Voting Round Setup); excluded here because the older-kid row's Dinner Voting access is Own-only, never Full, per the Access Matrix (user-persona.md), and kid-initiated round creation is an explicit Non-Goal of the feature.
- Changing a vote after it is cast -- the Brief's Entity-Lifecycle Coverage Matrix states a vote is never edited after casting, only the round's resolution is updated (by FEAT-17.SPEC-005); this screen therefore never offers a change-vote action.
- Displaying the vote split or resolved outcome to the household -- handled by FEAT-17.SPEC-003 (Vote Outcome & Resolution); this screen only ever shows the older kid's own casting experience for a round they can still act on.
- Making the final call on a split vote -- a Maya-exclusive capability handled by FEAT-17.SPEC-003, per the Access Matrix's Own-only entry for the older-kid row.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-03 (AI Weekly Dinner Plan Generation) -- the older kid's view of the week's plan | Older kid opens a night that has an open voting round | Night reference and the round's 2-3 safety-checked options (created by FEAT-17.SPEC-004) |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | No | No | Screen is never reached from Maya's navigation; Maya sees the vote split or resolved outcome through FEAT-17.SPEC-003, not this casting screen |
| Sam (Other Adult Member) | No | No | Screen is never reached from Sam's navigation; Sam's Dinner Voting View entitlement applies to FEAT-17.SPEC-003, not casting |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile, so no screen can be reached at all; Dinner Voting is None |
| Jordan (older kid, limited login -- Later) | Full screen, for a round they have not yet voted in and that is still Open | Cast one vote in an open round they have not voted in yet | Attempting to cast a vote in a round that has already resolved is rejected with refresh, showing FEAT-17.SPEC-003's resolved outcome instead (FEAT-17.SPEC-006) |
| Riley (Operator, support) | No | No | Dinner Voting: None; this screen is never exposed through support access, even against an open Support Request |
| Unauthenticated | No | No | Redirected to sign-in; no household voting content is reachable without signing in |
| Expired session | No | No | Older kid's limited-login session has expired: a re-authentication prompt appears; an in-progress (not yet submitted) selection is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Vote for {night}'s dinner" with a back arrow (returns to the older kid's view of the week's plan).

**Body:** The round's 2-3 safety-checked option cards, stacked below the header. Each card shows the recipe name, the "checked against allergies" badge, and the "always check labels" disclaimer (the shared safety-checked option card pattern also used by FEAT-17.SPEC-002 and FEAT-17.SPEC-003). Each card is itself the vote control: tapping a card casts the vote for that option directly, with no separate confirmation step, matching the feature's "simple, instant tap" design. Below the cards, a short helper line reads "Tap the one you want."

**Footer:** None -- voting happens directly on the option cards.

### Responsive Behavior

- **Compact breakpoint:** Option cards stack vertically at full width, each sized for a single-hand tap.
- **Medium size class and above:** Option cards arrange in a row (up to 3 across), capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the older kid's view of the week's plan (FEAT-03) | Screen closes | Standard transition back |
| Option card | Tap | Casts a vote for that option, validated by FEAT-17.SPEC-006 (round must be Open; one vote per older-kid profile per round); a successful vote triggers FEAT-17.SPEC-005 to recompute the round's tally | Selected card shows the Voted appearance; other cards become non-interactive | Confirmation "Your vote is in!" then the selected option shows "Your vote" |
| Option card (round already resolved when tapped) | Tap | Vote submission rejected by FEAT-17.SPEC-006 | Screen refreshes to the round's resolved outcome | Message "This round is already decided." then navigation to FEAT-17.SPEC-003 |
| Option card (second tap while a submission is in flight) | Tap | No action -- the in-flight submission is not resubmitted or replaced | Cards remain in their Submitting appearance | No duplicate vote is created |

### Accessibility Notes

- **Focus order:** Back arrow -> option card 1 -> option card 2 -> option card 3 (when present).
- **Vote confirmation:** "Your vote is in!" is announced to assistive technology when a vote is accepted.
- **Rejected vote:** the "This round is already decided." message and the resulting navigation to FEAT-17.SPEC-003 are announced, and focus moves to that screen's outcome content.
- **Keyboard alternatives:** every option card is a single activatable control reachable and operable by keyboard (Enter/Space); there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Placeholder shown in place of the option cards | Screen opens, fetching the round's options and this older kid's vote status | Data loads (screen then enters Open, not yet voted; Voted; or Empty, depending on what loaded) |
| Open, not yet voted (default) | All option cards shown and selectable | Screen opens on an open round the older kid has not voted in | Older kid taps an option card |
| Submitting | The tapped card shows a brief loading indicator; other cards become non-interactive | Older kid taps an option card | Submission completes (success or failure) |
| Voted | The chosen option shows "Your vote"; other options remain visible but non-interactive | Vote submission succeeds | Older kid navigates away, or returns later to a since-resolved round (then FEAT-17.SPEC-003 is shown instead) |
| Error | Submission retries automatically in the background; option cards remain visible in their Submitting appearance, with no blocking error banner, per the feature's "failed vote submission retries automatically" behavior | Vote submission fails | Retry succeeds and the screen enters the Voted state |
| Offline/Degraded | Banner "You're offline -- your vote will be saved when you reconnect." at the top of the screen; the tapped option is marked pending locally | Connectivity is lost at or after the moment the older kid taps an option card | Connectivity is restored -- the queued vote submits automatically and the screen enters the Voted state |
| Empty (no open round) | "No vote open for this night right now." with no option cards shown | Older kid opens this screen for a night with no open voting round | An organiser opens a round for that night (FEAT-17.SPEC-004), or the older kid navigates away |

## Validation Rules

Validation governed by FEAT-17.SPEC-006 (Dinner Voting Rules -- Access, Validation & Conflict Resolution). See that spec for the one-vote-per-older-kid-profile-per-round limit and the post-resolution rejection rule. This screen applies validation at the moment of the tap.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | The older kid's view of the week's plan | FEAT-03 |
| Vote rejected (round already resolved) | FEAT-17.SPEC-003 (Vote Outcome & Resolution) | -- |

## Data Model

**Creates:** Dinner Vote -- voter (the signed-in older kid's Member Profile), choice (the selected option), against the round already created by FEAT-17.SPEC-004 (night + its 2-3 safety-checked options); created at the moment of the tap.
**Reads:** Dinner Vote round -- the night, its 2-3 safety-checked options (recipe name and safety badge) and whether this older kid has already voted; Member Profile -- the signed-in older kid's own profile reference, to identify the voter.
**Updates:** None -- a vote is created once and never edited by this screen (per the Brief's Entity-Lifecycle Coverage Matrix).
**Deletes:** None.

## Business Rules

- One vote per older-kid profile per round (FEAT-17.SPEC-006) -- an older kid who has already voted sees their own choice marked and cannot cast a second vote in the same round.
- Every option offered has already passed FEAT-02's safety check before the round exists (XBR-01); this screen never re-runs the check, it only displays the badge and disclaimer already computed by FEAT-17.SPEC-004.
- A vote cast after the round has already resolved is rejected with refresh, showing the resolved outcome (FEAT-17.SPEC-006).
- Casting a vote fires FEAT-17.SPEC-005 (Vote Round Resolution & Fallback) to recompute the round's tally.

## Edge Cases

- **Round resolves while the older kid is looking at this screen (Maya makes her final call, or the tally auto-resolves)** -- The screen refreshes and navigates to FEAT-17.SPEC-003, since voting no longer applies. This is the concurrent-edit conflict entry for the Dinner Vote entity, per the dependency map's Contention note: "A vote cast after Maya has resolved the round is rejected with refresh, showing the outcome." Resolution: reject-with-refresh.
- **Older kid taps two option cards in quick succession** -- Only the first tap is submitted; the second tap is ignored while the first submission is in flight, since a vote is never resubmitted with a different choice from this screen once cast.
- **Older kid loses connectivity mid-vote** -- The vote is held locally and syncs once connectivity returns (Offline/Degraded state above).
- **Older kid navigates away before the round resolves and returns later** -- The screen re-fetches the round's current state on return; if it has since resolved, the older kid is shown FEAT-17.SPEC-003 instead of this screen.
- **Round has only 2 options instead of 3** -- Exactly as many option cards are shown as the round defines (2 or 3); the layout adapts without a placeholder third card.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-004 (Voting Round Creation & Safety Validation) | Navigation (inbound) | An older kid can reach this screen only once this automation has opened a round |
| FEAT-17.SPEC-005 (Vote Round Resolution & Fallback) | Triggers (outbound) | Casting a vote triggers tally recomputation |
| FEAT-17.SPEC-006 (Dinner Voting Rules) | References (inbound) | Validation and authorization rules applied to the vote |
| FEAT-17.SPEC-003 (Vote Outcome & Resolution) | Navigation (outbound) | A rejected late vote routes here to show the resolved outcome |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Navigation (inbound/outbound) | Entry from, and exit to, the older kid's view of the week's plan |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| dinner_vote_cast | round night, chosen option, voting older-kid profile reference | An older kid successfully casts a vote | N/A -- no metric in success-metrics.md carries Older-Kid Dinner Voting (FEAT-17) as its Connected Feature; this Later-phase feature has no metric of its own in the current success-metrics set |
| dinner_vote_cast_offline_queued | round night | A vote is queued locally while the older kid is offline | N/A -- same reason as above |

## Acceptance Criteria

**FEAT-17.SPEC-001-AC-01:** Given Jordan (older kid) is on the Dinner Vote Casting screen for an open round, when Jordan taps an option card, then the vote is recorded (triggering FEAT-17.SPEC-005 to recompute the tally), the option shows "Your vote," and other options become non-interactive.

**FEAT-17.SPEC-001-AC-02:** Given Jordan is on the Dinner Vote Casting screen, when Jordan taps the back arrow, then Jordan returns to their view of the week's plan (FEAT-03).

**FEAT-17.SPEC-001-AC-03:** Given Jordan has just tapped an option card and its submission is in flight, when Jordan taps a second option card before the first submission completes, then the second tap is ignored and only the first choice is recorded.

**FEAT-17.SPEC-001-AC-04:** Given Jordan opens the Dinner Vote Casting screen for a night with no open voting round, when the screen loads, then it shows "No vote open for this night right now." with no option cards.

**FEAT-17.SPEC-001-AC-05:** Given Jordan loses connectivity, when Jordan taps an option card, then the banner "You're offline -- your vote will be saved when you reconnect." appears and the vote is queued locally.

**FEAT-17.SPEC-001-AC-06:** Given Jordan casts a vote and the submission fails, when the failure occurs, then the system retries the submission automatically without a blocking error, and the screen shows "Your vote is in!" once the retry succeeds.

**FEAT-17.SPEC-001-AC-07:** Given Jordan has already voted in an open round, when Jordan returns to the screen, then the previously selected option still shows "Your vote" and the other options remain non-interactive, preventing a second vote in the same round.

**FEAT-17.SPEC-001-AC-08:** Given Jordan is viewing an open round's option cards, when the screen renders, then every option shows the "checked against allergies" badge and the "always check labels" disclaimer, since every offered option already passed FEAT-02's safety check before the round was created.

**FEAT-17.SPEC-001-AC-09:** Given a round has just resolved (by tally or by Maya's final call) at the moment Jordan taps an option card, when the vote submission reaches the system, then it is rejected with the message "This round is already decided." and Jordan is shown FEAT-17.SPEC-003's resolved outcome instead.

**FEAT-17.SPEC-001-AC-10:** Given Jordan is viewing an open round on this screen and the round resolves while the screen remains open, when the resolution completes, then the screen refreshes and navigates to FEAT-17.SPEC-003 rather than allowing a further vote.

**FEAT-17.SPEC-001-AC-11:** Given Jordan navigated away from this screen before voting, when Jordan returns to a round that has since resolved, then the screen shows FEAT-17.SPEC-003's outcome instead of the voting cards.

**FEAT-17.SPEC-001-AC-12:** Given a round has been opened with only 2 options instead of 3, when Jordan views the Dinner Vote Casting screen, then exactly 2 option cards are shown with no placeholder third card.

**FEAT-17.SPEC-001-AC-13:** Given Jordan opens the Dinner Vote Casting screen for a night with an open round, when the screen first opens, then a loading placeholder is shown in place of the option cards while the round's options and Jordan's vote status are fetched.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 6 (loading, submitting, voted, error, offline, empty) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
