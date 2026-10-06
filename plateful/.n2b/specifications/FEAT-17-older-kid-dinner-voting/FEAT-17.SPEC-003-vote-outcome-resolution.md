---
document_type: spec
spec_type: screen
spec_id: FEAT-17.SPEC-003
spec_name: Vote Outcome & Resolution
spec_slug: vote-outcome-resolution
parent_feature: FEAT-17
parent_feature_name: Older-Kid Dinner Voting
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Vote Outcome & Resolution

## Overview

**Name:** Vote Outcome & Resolution
**ID:** FEAT-17.SPEC-003
**Type:** Screen
**Purpose:** Shows the current vote split or resolved outcome for a night's voting round, and lets Maya make the final call when votes conflict.
**Parent Feature:** FEAT-17 -- Older-Kid Dinner Voting

## Scope and Non-Goals

**In Scope:**
- Showing the live tally, split status, or resolved outcome for a night's voting round
- Letting Maya make the final call when a round is split with no consensus
- Applying Maya's final call as the round's resolution

**Non-Goals:**
- Casting a vote -- handled by FEAT-17.SPEC-001; the older-kid row's Dinner Voting access on this screen is read-only (View), not Own-only casting.
- Opening a round -- handled by FEAT-17.SPEC-002.
- Computing the tally, detecting unanimity, or applying the no-vote fallback -- computed by FEAT-17.SPEC-005; this screen displays that automation's results and captures Maya's final call, it does not itself tally votes.
- The app arbitrarily deciding a tie -- excluded per this feature's own Primary Flows & Alternates, which states the organiser sees the split and makes the final call "rather than the app arbitrarily deciding"; no automatic tie-break exists on this screen.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-03 (AI Weekly Dinner Plan Generation) -- week plan, Sunday Plan Review | Maya or Sam reviews an older kid's vote for a night | The night reference and its round's current tally or resolution |
| FEAT-17.SPEC-002 (Voting Round Setup) | Maya selects a night that already has an open or resolved round | The night reference |
| FEAT-17.SPEC-001 (Dinner Vote Casting) | An older kid's vote is rejected because the round already resolved | The night reference and its resolved outcome |
| FEAT-17.SPEC-006 (Dinner Voting Rules) | A late vote is rejected per the post-resolution rejection rule | The night reference and its resolved outcome |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen, including the tally, split status, and resolution | Make the final call on a split round | -- |
| Sam (Other Adult Member) | Full screen (tally, split status, resolution) | None -- read-only | Final-call control is not shown; an attempt to act on it is not possible since no control is rendered for this role |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists; nothing is shown; Dinner Voting is None |
| Jordan (older kid, limited login -- Later) | Full screen (tally, split status, resolution) | None -- read-only, consistent with Own-only applying to casting a vote, not to deciding the round | Final-call control is not shown to this role |
| Riley (Operator, support) | No | No | Dinner Voting: None; not exposed through support access |
| Unauthenticated | No | No | Redirected to sign-in |
| Expired session | No | No | Session-holder's session has expired: a re-authentication prompt appears; if Maya was mid-final-call selection, the pending selection is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Title "{night}'s vote" with a back arrow (returns to the entry screen -- FEAT-03's week plan, or FEAT-17.SPEC-002).

**Body:** A status line at the top reading "Open," "Split -- awaiting the final call," or "Resolved." Below it, each of the round's 2-3 safety-checked options is shown as a safety-checked option card (recipe name, "checked against allergies" badge, "always check labels" disclaimer -- the pattern shared with FEAT-17.SPEC-001 and FEAT-17.SPEC-002), each annotated with its current vote count. Once resolved, the chosen option additionally shows a "Chosen" badge and a one-line note on how it resolved ("by vote," "no votes cast -- kept the original suggestion," "partial votes cast -- kept the original suggestion," or "organiser's final call"). When the round is Split and Maya is viewing, a "Make the final call" button appears below the tally, and tapping it makes each tied option's card directly selectable.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Option cards with their vote counts stack vertically at full width; "Make the final call" (Maya only) sits below the last card, full width.
- **Medium size class and above:** Option cards may lay out in a row (up to 3 across), capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the entry screen (FEAT-03's week plan, or FEAT-17.SPEC-002) | Screen closes | Standard transition back |
| Option card (Sam, Jordan older kid) | Tap | Display-only -- no action | None | None; these roles view but do not act |
| "Make the final call" button (Maya, Split round only) | Tap | Enters final-call selection mode | The tied options become selectable | Visual selectable state on the tied cards |
| Option card (Maya, during final-call selection) | Tap | Sets Maya's chosen option as the round's resolution, applying FEAT-17.SPEC-005's resolution write onto the Weekly Plan's night slot | Resolution updates; the chosen card shows the "Chosen" badge | Confirmation "Vote resolved -- {option} it is!" |

### Accessibility Notes

- **Focus order:** Back arrow -> status line -> option cards in order -> "Make the final call" (when shown to Maya).
- **Live updates:** vote-count changes while a round is Open are announced to assistive technology as they occur; the resolution confirmation is announced when a round resolves.
- **Keyboard alternatives:** the "Make the final call" button and, once activated, each tied option's selection are reachable and operable by keyboard (Enter/Space); there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Placeholder shown while the round's tally loads | Screen opens | Data loads |
| Open, tally in progress (default) | Live vote counts per option; no "Chosen" badge; no final-call control | Round is open and votes are still being cast | Round resolves (auto-resolve or Maya's final call) |
| Split, awaiting Maya | Tied options highlighted; "Make the final call" shown to Maya only | All eligible older-kid voters have voted and disagree (per FEAT-17.SPEC-005) | Maya makes the final call |
| Resolved | The chosen option shows the "Chosen" badge and how it resolved (by vote, no-vote fallback, partial-vote fallback, or organiser's final call) | The round resolves | The night passes, or a new round opens for a future night |
| Empty (no round for this night) | "No vote for this night." | No round exists yet for the viewed night | An organiser opens a round (FEAT-17.SPEC-004) |
| Error | Banner "Could not load the vote." with a retry action | The tally fetch fails | Retry succeeds |
| Offline/Degraded | The last-known tally is shown from cache with a banner "You're offline -- this may not be the latest count."; Maya's final-call action is disabled while offline | Connectivity is lost while viewing | Connectivity is restored -- the live tally resumes and, for Maya, the final-call action re-enables |

## Validation Rules

Validation governed by FEAT-17.SPEC-006 (Dinner Voting Rules -- Access, Validation & Conflict Resolution). See that spec for who may make the final call (Maya only) and what constitutes a Split round requiring one.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | Week plan (or FEAT-17.SPEC-002 if arrived from there) | FEAT-03 |
| Successful final call | Week plan, updated with the resolved option | FEAT-03 |

## Data Model

**Reads:** Dinner Vote -- the round (night + options), every cast vote's choice, and the resolution field; Weekly Plan -- the night's slot, to reflect the resolved option once set.
**Creates:** None.
**Updates:** Dinner Vote -- the resolution field, set to Maya's final-call choice; Weekly Plan -- the night's slot, set to the resolved option (applying FEAT-17.SPEC-005's write, per the dependency map: "Weekly Plan Updated by FEAT-17, vote outcome").
**Deletes:** None.

## Business Rules

- Only Maya may make the final call on a split round (Access Matrix: Dinner Voting Full for Maya only).
- Maya's final call, once made, is the round's resolution and is applied the same way an auto-resolved tally is (FEAT-17.SPEC-005): it writes onto the Weekly Plan's night slot and fires dinner_vote_resolved.
- The app never arbitrarily decides a tie -- every genuine split routes to Maya (Brief's Non-Goals; Primary Flows & Alternates).
- A vote cast after the round has already resolved is rejected with refresh by FEAT-17.SPEC-006, and this screen is what such a rejected voter is shown.

## Edge Cases

- **A new vote is cast while Maya is mid-final-call selection** -- The tally is recomputed by FEAT-17.SPEC-005; if the new vote resolves the split into a unanimous outcome before Maya submits her final call, Maya's screen refreshes to the auto-resolved outcome and her final-call action is withdrawn with "This round resolved on its own -- {option} won." This is the concurrent-edit conflict entry for the Dinner Vote entity, per the dependency map's Contention note. Resolution: reject-with-refresh.
- **Maya submits a final call from two devices at nearly the same time** -- Only the first submission is applied; the second is rejected with refresh, showing the already-resolved outcome.
- **The night has no votes cast and the plan's resolution point passes** -- FEAT-17.SPEC-005 resolves to the AI's original suggestion; this screen shows the Resolved state with "No votes cast -- kept the original suggestion."
- **The night's plan resolution point passes with some but not all eligible older-kid voters having voted** -- FEAT-17.SPEC-005 resolves to the AI's original suggestion rather than leaving the round open indefinitely; this screen shows the Resolved state with "Partial votes cast -- kept the original suggestion."
- **Sam or Jordan (older kid) views this screen while a round is still Open** -- They see the live tally in real time but no final-call control, consistent with their read-only access.
- **Maya attempts to make a final call on a round that resolved a moment earlier** -- The attempt is rejected with refresh, showing the already-resolved outcome rather than allowing a second resolution.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-005 (Vote Round Resolution & Fallback) | References (inbound); Triggers (outbound) | Displays the computed tally/resolution; Maya's final call feeds into SPEC-005's resolution write |
| FEAT-17.SPEC-002 (Voting Round Setup) | Navigation (inbound, via "already open"/"already decided" link) | Maya reaches this screen when a round already exists for the night she selected |
| FEAT-17.SPEC-001 (Dinner Vote Casting) | Navigation (inbound, via rejected late vote) | An older kid whose vote is rejected after resolution lands here |
| FEAT-17.SPEC-006 (Dinner Voting Rules) | References (inbound) | Authorization for the final-call action and the post-resolution rejection rule |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Navigation (inbound/outbound) | Entry from and exit to the week plan review |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| dinner_vote_outcome_viewed | night, round status (open / split / resolved), viewing role | A household member opens this screen | N/A -- no metric in success-metrics.md carries Older-Kid Dinner Voting (FEAT-17) as its Connected Feature; this Later-phase feature has no metric of its own in the current success-metrics set |

Note: the resolution event itself (dinner_vote_resolved) is owned and emitted by FEAT-17.SPEC-005, which applies regardless of whether the resolution came from a tally or from Maya's final call submitted here.

## Acceptance Criteria

**FEAT-17.SPEC-003-AC-01:** Given Maya is viewing an Open round with votes split between two options and all eligible older-kid profiles have voted, when she looks at the screen, then it shows "Split -- awaiting the final call" with a "Make the final call" button.

**FEAT-17.SPEC-003-AC-02:** Given Maya is viewing a Split round, when she taps "Make the final call" and then selects one of the tied options, then the round resolves to her choice, the Weekly Plan's night slot updates, and she sees "Vote resolved -- {option} it is!"

**FEAT-17.SPEC-003-AC-03:** Given Sam is viewing an Open round, when he looks at the screen, then he sees the live vote counts per option but no "Make the final call" control.

**FEAT-17.SPEC-003-AC-04:** Given Jordan (older kid) is viewing a Resolved round, when the screen loads, then the chosen option shows the "Chosen" badge and a note on how it resolved, with no action available to Jordan.

**FEAT-17.SPEC-003-AC-05:** Given Maya opens this screen for a night with no round yet, when the screen loads, then it shows "No vote for this night."

**FEAT-17.SPEC-003-AC-06:** Given Maya is viewing this screen and loses connectivity, when connectivity drops, then the banner "You're offline -- this may not be the latest count." appears and her "Make the final call" action is disabled.

**FEAT-17.SPEC-003-AC-07:** Given the tally fetch fails, when Maya opens this screen, then a banner "Could not load the vote." appears with a retry action.

**FEAT-17.SPEC-003-AC-08:** Given Maya is mid-final-call selection on a Split round, when a new vote is cast that makes the round unanimous before she submits, then her screen refreshes to show "This round resolved on its own -- {option} won." and the final-call action is withdrawn.

**FEAT-17.SPEC-003-AC-09:** Given a night's round has no votes cast by the plan's resolution point, when FEAT-17.SPEC-005 applies the fallback, then this screen shows the Resolved state with "No votes cast -- kept the original suggestion."

**FEAT-17.SPEC-003-AC-10:** Given Maya submits a final call on a Split round from two devices at nearly the same time, when both submissions reach the system, then only the first is applied and the second is rejected with refresh showing the already-resolved outcome.

**FEAT-17.SPEC-003-AC-11:** Given Maya is on this screen while a round's tally is still loading, when the screen first opens, then a loading placeholder is shown in place of the tally.

**FEAT-17.SPEC-003-AC-12:** Given an older kid's vote is rejected on FEAT-17.SPEC-001 because the round already resolved, when the rejection occurs, then the older kid is navigated to this screen and shown the resolved outcome.

**FEAT-17.SPEC-003-AC-13:** Given a night's round has some but not all eligible older-kid voters' votes cast when the plan's resolution point arrives, when FEAT-17.SPEC-005 applies the partial-vote fallback, then this screen shows the Resolved state with "Partial votes cast -- kept the original suggestion."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 6 (loading, split, resolved, empty, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
