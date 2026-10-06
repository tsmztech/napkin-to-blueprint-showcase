# FEAT-17 — Older-Kid Dinner Voting

This chapter covers FEAT-17, Older-Kid Dinner Voting, a Important-tier feature. It contains 6 specifications carrying 74 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-17.SPEC-001 | Dinner Vote Casting | screen | 13 |
| FEAT-17.SPEC-002 | Voting Round Setup | screen | 12 |
| FEAT-17.SPEC-003 | Vote Outcome & Resolution | screen | 13 |
| FEAT-17.SPEC-004 | Voting Round Creation & Safety Validation | automation | 8 |
| FEAT-17.SPEC-005 | Vote Round Resolution & Fallback | automation | 11 |
| FEAT-17.SPEC-006 | Dinner Voting Rules -- Access, Validation & Conflict Resolution | logic-rule | 17 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Older-Kid Dinner Voting

## Summary

**Feature:** Older-Kid Dinner Voting
**ID:** FEAT-17
**Description:** Older kids can vote on which dinner they'd prefer among the week's options, giving them a voice in the plan without needing a full account.
**Priority:** Important
**Phase:** Later
**Type:** User-Facing
**Rationale:** The brief names this directly — "older kids want to vote on dinners" (BRIEF.md, Target Users & Roles) — but also leaves kids' representation as an explicit open question, with the founder leaning toward "possibly a limited login for older kids later" (BRIEF.md, Open Questions). Phased to Later because it depends on resolving that open question about how older kids are represented at all.

**Key Capabilities:**
- Vote on an option — Older kid picks a preferred dinner among a small set of safe alternatives for a given night
- See the outcome — Older kid sees which option was chosen once the organiser finalizes or the vote naturally resolves

This feature is Later-phase and gated on the Later-phase older-kid limited login existing at all (Access field; scope-boundaries.md SC-02). It depends on AI Weekly Dinner Plan Generation (FEAT-03) for the options it offers and on Dietary Rules & Allergy Safety Engine (FEAT-02) to guarantee every option is already safe (Interactions field). The Access field also gives Maya (Organiser) Full access to Dinner Voting — she opens voting rounds and makes the final call when votes conflict — so this Brief elaborates that organiser-side capability alongside the two Key Capabilities Stage 2 names for the older kid.

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-17.SPEC-001 | Dinner Vote Casting | Screen | Jordan (older kid, limited login) | Older kid views the 2-3 safety-checked options for an open voting round on a night and casts one preference |
| FEAT-17.SPEC-002 | Voting Round Setup | Screen | Maya | Maya opens a voting round for a given night by selecting the 2-3 already safety-checked options to offer |
| FEAT-17.SPEC-003 | Vote Outcome & Resolution | Screen | Maya, Sam, Jordan (older kid, limited login) | Shows the current vote split or resolved outcome for a night; lets Maya make the final call when votes conflict |
| FEAT-17.SPEC-004 | Voting Round Creation & Safety Validation | Automation | Maya | When Maya opens a round, validates the chosen options against the 2-3 count and FEAT-02's safety check, creates the round, and fires dinner_vote_opened |
| FEAT-17.SPEC-005 | Vote Round Resolution & Fallback | Automation | Maya, Sam, Jordan (older kid, limited login) | Recomputes the tally on each cast vote, auto-resolves a unanimous round, defaults to the AI's original suggestion when no vote was cast, writes the outcome onto the Weekly Plan, and fires dinner_vote_resolved |
| FEAT-17.SPEC-006 | Dinner Voting Rules — Access, Validation & Conflict Resolution | Logic/Rule | All | Governs who may open rounds, vote, view outcomes, or make the final call; enforces the option-count/safety-check and one-vote-per-profile-per-round limits; and rejects with refresh any vote cast after the round is already resolved |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Vote on an option | FEAT-17.SPEC-001 | Older kid selects one of the round's 2-3 safety-checked options; the write itself is validated by SPEC-006 | Phase 2 (Explicit) |
| See the outcome | FEAT-17.SPEC-003, FEAT-17.SPEC-005 | Outcome screen displays the tally or resolution that SPEC-005 computes or that Maya sets via her final call | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-17.SPEC-002 | Voting Round Setup | Phase 2 (Explicit surface, Access-field elaboration) | The Access field gives Maya Full access to Dinner Voting and states she "opens voting rounds" — a capability Stage 2's Key Capabilities list does not name directly but the Access field requires be elaborated into a spec, per this Brief's Stage 2 Depth Fields rule |
| FEAT-17.SPEC-004 | Voting Round Creation & Safety Validation | Phase 4 (Trigger-Response, cross-feature effect) | Opening a round is more than a direct data write: it must validate the option count and re-confirm each option already passed FEAT-02's safety check (Interactions field, XBR-01) before the round can exist |
| FEAT-17.SPEC-005 | Vote Round Resolution & Fallback | Phase 4 (Trigger-Response) + Phase 6 (failure/no-consensus handling) | The Primary Flows & Alternates field names three distinct resolution paths (happy-path outcome, no-vote fallback to the AI's suggestion, and tie/no-consensus routed to Maya) that no single screen interaction covers; this automation is the shared trigger-response logic behind all three, and it is what actually updates the Weekly Plan (dependency map: Weekly Plan "Updated by FEAT-17, vote outcome") |
| FEAT-17.SPEC-006 | Dinner Voting Rules — Access, Validation & Conflict Resolution | Phase 5 (Rule-Constraint Discovery — authorization + conditional logic) | Role-differentiated authorization (Own-only vote vs. Full open/resolve vs. View outcome vs. None for the young-kid row, Riley, and unauthorized visitors), the option-count/safety-check and one-vote-per-profile limits (Validation & Limits field), and the post-resolution vote-rejection rule (Dinner Vote's Contention note) form conditional logic shared across every screen and automation in this feature — past the standalone-spec threshold |

**No Integration or Notification specs:** The Dependencies slice of assumptions-constraints.md states plainly "None — this feature relies on no external capability," and the Communications field states "N/A — voting is a self-initiated, in-app action; no separate notification is sent for this Later-phase feature." Both category-level checks (Phase 7, Check 4) resolve to zero without a gap: integration_count and notification_count are legitimately 0.

## Entity-Lifecycle Coverage Matrix

**Entity: Dinner Vote**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-17.SPEC-001 (vote), FEAT-17.SPEC-004 (round) | An older kid casts a vote (choice) into a round that SPEC-004 creates when Maya opens it, validated by SPEC-006 | One round per night; one vote per older-kid profile per round |
| Read (single) | FEAT-17.SPEC-003 | Vote Outcome screen reads one night's round and its votes | Also read cross-feature by FEAT-03 during plan review (dependency map) |
| Read (list) | FEAT-17.SPEC-003 | Displays every cast vote in the round to show the split | -- |
| Update | FEAT-17.SPEC-005 | Resolution field set by tally (auto-resolve) or by Maya's final call (SPEC-003 UI, SPEC-005 automation applies it) | A vote itself is never edited after casting -- only the round's resolution is updated |
| Delete/Archive | N/A | No delete or archive spec exists for Dinner Vote — an explicit non-goal. Votes are retained for the life of the household account alongside the Weekly Plan history they belong to (scope-boundaries.md SC-18), with no separate purge policy | -- |
| State Transition | FEAT-17.SPEC-004 (Open), FEAT-17.SPEC-005 (Resolved) | A round moves Open -> Resolved, either by tally (unanimous or fallback) or by Maya's override | -- |

**Entity: Weekly Plan** (this feature only updates one field on an existing plan; creation, full read, and deletion belong to other features per the dependency map)

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Owned by FEAT-03 (AI generation) and FEAT-23 (manual planning) — not this feature's responsibility | Cross-feature ownership, not a gap |
| Read (single) | FEAT-17.SPEC-001, FEAT-17.SPEC-002, FEAT-17.SPEC-003 | Each screen reads the relevant night's plan context to show what is being voted on | -- |
| Read (list) | N/A | This feature never lists whole plans — that is FEAT-03's/FEAT-23's Weekly Plan screen responsibility | Cross-feature ownership |
| Update | FEAT-17.SPEC-005 | Writes the resolved option onto the night's slot once the round resolves | Dependency map: Weekly Plan "Updated by FEAT-17 (Later, vote outcome)" |
| Delete/Archive | N/A | Owned by FEAT-18 (household deletion cascade) / plan archival at week end — not this feature | Cross-feature ownership |
| State Transition | N/A | Weekly Plan's own status lifecycle (Generated/Reviewed/Approved/Active/Archived) belongs to FEAT-03/FEAT-23; this feature only ever writes the chosen-option value on one night's slot | Cross-feature ownership |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Member Profile | FEAT-17.SPEC-001, FEAT-17.SPEC-002, FEAT-17.SPEC-006 | Identifies the voting older-kid profile and its `member_type`, so the one-vote-per-profile rule and the Access Matrix's role checks can be enforced |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Maya opens a voting round for a night, selecting 2-3 options | Validate the option count and that every option already passed FEAT-02's safety check, create the round, fire dinner_vote_opened | Standalone Automation | FEAT-17.SPEC-004 |
| Older kid casts a vote | Record the vote, enforcing one vote per older-kid profile per round | Inline in triggering screen, validated by SPEC-006 | FEAT-17.SPEC-001 |
| A vote is cast (dinner_vote_cast) | Recompute the round's tally | Standalone Automation | FEAT-17.SPEC-005 |
| Tally recompute finds unanimous agreement | Auto-resolve the round to that option, write it onto the Weekly Plan, fire dinner_vote_resolved | Standalone Automation | FEAT-17.SPEC-005 |
| No vote is cast for a night by the plan's resolution point | Round resolves to the AI's original suggestion — voting never blocks the plan | Standalone Automation | FEAT-17.SPEC-005 |
| Votes split across more than one option with no consensus | Round is flagged as split; Maya sees the split and makes the final call rather than the app deciding | Standalone Automation (flag) feeding a Screen action | FEAT-17.SPEC-005 / FEAT-17.SPEC-003 |
| Maya makes her final call on a split round | Round resolves to Maya's choice, writes it onto the Weekly Plan, fires dinner_vote_resolved | Inline in triggering screen, applies SPEC-005's resolution write | FEAT-17.SPEC-003 |
| A vote is cast after the round has already resolved | Rejected with refresh, showing the resolved outcome instead | Standalone Logic/Rule | FEAT-17.SPEC-006 |
| Vote submission fails | Retries automatically | Inline in triggering screen (Error state) | FEAT-17.SPEC-001 |
| Vote cast while offline | Held locally, synced once connectivity returns | Inline in triggering screen (Offline-degraded state) | FEAT-17.SPEC-001 |
| A safety-related change removes one of a round's options mid-week | Option removal is FEAT-02's responsibility; the round's remaining valid options carry forward | Cross-feature — owned by Dietary Rules & Allergy Safety Engine | FEAT-02 responsibility, referenced by FEAT-17.SPEC-004 |

## Shared Context

**Shared Entities:**
- Dinner Vote — round context and options created by SPEC-004 (opening), individual votes created by SPEC-001 (casting), resolution updated by SPEC-005, displayed by SPEC-003, all validated against SPEC-006. Fields: round (night + 2-3 safety-checked options), voter (older-kid Member Profile), choice, resolution.
- Weekly Plan — the relevant night's slot is read for context by SPEC-001/002/003 and updated with the resolved choice by SPEC-005. No other Weekly Plan field is touched by this feature.
- Member Profile — read by SPEC-001, SPEC-002, and SPEC-006 to identify the voting older-kid profile and enforce role-based access; never created, updated, or deleted by this feature.

**Shared UI Patterns:**
- Safety-checked option card — the same option-display pattern (recipe name, the "checked against allergies" badge, and the "always check labels" disclaimer per XBR-01) appears in SPEC-001 (voting), SPEC-002 (Maya's option selection), and SPEC-003 (outcome display). Spec Writers for all three screens should describe this card consistently rather than re-deriving its content.

**Shared Validation:**
- FEAT-17.SPEC-006 defines the role/access rules, the option-count and safety-check limits, the one-vote-per-profile rule, and the post-resolution rejection rule. SPEC-001 through SPEC-005 all reference SPEC-006 rather than restating any of these rules.

## Internal Dependency Map

```
FEAT-17.SPEC-002 (Voting Round Setup) -> [Maya selects options and opens the round] -> FEAT-17.SPEC-004 (Voting Round Creation & Safety Validation)
FEAT-17.SPEC-002 (Voting Round Setup) -> [validates against] -> FEAT-17.SPEC-006 (Dinner Voting Rules)
FEAT-17.SPEC-004 (Voting Round Creation & Safety Validation) -> [round now open] -> FEAT-17.SPEC-001 (Dinner Vote Casting)
FEAT-17.SPEC-001 (Dinner Vote Casting) -> [validates against] -> FEAT-17.SPEC-006 (Dinner Voting Rules)
FEAT-17.SPEC-001 (Dinner Vote Casting) -> [vote cast] -> FEAT-17.SPEC-005 (Vote Round Resolution & Fallback)
FEAT-17.SPEC-005 (Vote Round Resolution & Fallback) -> [split detected, no consensus] -> FEAT-17.SPEC-003 (Vote Outcome & Resolution)
FEAT-17.SPEC-003 (Vote Outcome & Resolution) -> [Maya's final call] -> FEAT-17.SPEC-005 (Vote Round Resolution & Fallback)
FEAT-17.SPEC-005 (Vote Round Resolution & Fallback) -> [round resolved] -> FEAT-17.SPEC-003 (Vote Outcome & Resolution)
FEAT-17.SPEC-006 (Dinner Voting Rules) -> [rejects a late vote, points to] -> FEAT-17.SPEC-003 (Vote Outcome & Resolution)
```

**Default Entry:** FEAT-17.SPEC-002 (Voting Round Setup) for Maya, navigated from her Sunday plan review (FEAT-03); FEAT-17.SPEC-001 (Dinner Vote Casting) for the older kid, navigated from their view of the week's plan once a round is open. Both roles reach FEAT-17.SPEC-003 (Vote Outcome & Resolution) to see a round's split or resolved outcome.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-17.SPEC-002 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Maya opens a voting round for a night from within her Sunday plan review | Sunday Plan Review & First Swap journey, step 3 |
| FEAT-17.SPEC-004 | Inbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | Only options that have already passed the safety check may ever be offered in a round (XBR-01) | Round creation |
| FEAT-17.SPEC-004 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | The candidate options a round offers come from the AI's proposed alternatives for that night | Round creation |
| FEAT-17.SPEC-005 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Writes the resolved option onto the night's slot in the Weekly Plan | Round resolves, by tally or Maya's final call |
| FEAT-17.SPEC-005 | Inbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | XBR-01: the resolution write, like every other path onto the plan, carries the same safety check and badge before anyone sees it | Round resolution writes to the plan |
| FEAT-17.SPEC-003 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | The vote split or outcome is surfaced as part of the Sunday plan review, for households with voting enabled | Maya or Sam views the week's plan |

## Non-Functional Notes

**Data volumes / growth:** At most one open round per night, offering 2-3 options, with at most one vote per older-kid profile in the household (2-6 members per household, ASMP-24). Volume is trivial and scales only with household count, not with any growing dataset of its own.

**Responsiveness:** "Voting is a simple, instant tap" (States field) — casting completes with no perceptible wait; a failed submission retries automatically rather than surfacing a blocking error. No dedicated non-functional entry in assumptions-constraints.md names this feature directly, so it falls under the product's general one-handed, low-latency interaction bar (ASMP-29, Accessibility and one-handed use).

**Data sensitivity / privacy:** A Dinner Vote is children's data — a child's dinner preference — and is minimal, parent-controlled, and used only for the household's own plan (dependency map Data Sensitivity note; ASMP-26). It carries no name beyond the older-kid profile's own limited identity and no additional personal detail.

**Compliance flags:** Because the older-kid limited login and its votes are children's data, children's-privacy-class protections apply — minimal collection, parental control, no behavioral advertising to minors (ASMP-27). No medical or diet-advice regime applies, since this feature makes no dietary recommendation of its own (scope-boundaries.md SC-06).

## Non-Goals

- **Independent kid login as the v1 default** — Excluded per scope-boundaries.md (SC-02): the older-kid limited login this feature depends on is a Later-phase possibility, not a v1 baseline; the young-kid (no-login) profile row has `None` on Dinner Voting and can never vote, matching the Access Matrix.
- **The app arbitrarily deciding a tie** — Excluded per this feature's own Primary Flows & Alternates field, which states directly that "the organiser sees the split and makes the final call rather than the app arbitrarily deciding." No automatic tie-break algorithm is in scope; every genuine split routes to Maya.
- **A vote-triggered notification** — Excluded per this feature's own Communications field ("N/A ... no separate notification is sent for this Later-phase feature"); voting and its outcome are surfaced only within the in-app plan review, never as a push, email, or SMS message.
- **Automatic purge of vote history** — An intentional lifecycle decision surfaced by the CRUD matrix: Dinner Vote records are retained for the life of the household account alongside the Weekly Plan history they belong to (scope-boundaries.md SC-18), with no separate retention window or purge policy.
- **Kid-initiated round creation** — Excluded per the Access Matrix: the older-kid row's Dinner Voting access is `Own-only` (casting a vote), never `Full`; only Maya (`Full`) can open a voting round or make the final call on a split.



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



# Screen Spec: Voting Round Setup

## Overview

**Name:** Voting Round Setup
**ID:** FEAT-17.SPEC-002
**Type:** Screen
**Purpose:** Maya opens a voting round for a given night by selecting the 2-3 already safety-checked options to offer.
**Parent Feature:** FEAT-17 -- Older-Kid Dinner Voting

## Scope and Non-Goals

**In Scope:**
- Selecting 2-3 already safety-checked candidate options for a chosen night
- Opening the voting round from that selection
- Showing when a night already has an open (or resolved) round instead of a fresh selection

**Non-Goals:**
- Casting a vote -- handled by FEAT-17.SPEC-001; this screen is Maya-exclusive, since the Access Matrix (user-persona.md) gives Maya Full and the older-kid row only Own-only on Dinner Voting.
- Viewing the tally or making the final call on a split round -- handled by FEAT-17.SPEC-003.
- Generating the candidate options themselves -- owned by FEAT-03 (AI Weekly Dinner Plan Generation), which supplies the AI's proposed alternatives for the night (Brief's Cross-Feature Touchpoints); this screen only lets Maya choose among options FEAT-03 has already proposed and FEAT-02 has already safety-checked.
- Removing an option that later fails a safety check -- excluded per this feature's own Side-Effect Inventory, which assigns option removal to Dietary Rules & Allergy Safety Engine (FEAT-02); this screen only reflects the current candidate list FEAT-02 already filtered.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-03 (AI Weekly Dinner Plan Generation) -- Sunday Plan Review | Maya taps "Open a vote" for a night during her Sunday plan review | The selected night and its AI-proposed, safety-checked candidate options |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Select options and open the round | -- |
| Sam (Other Adult Member) | No | No | Screen is not reachable; Sam's Dinner Voting View entitlement applies to FEAT-17.SPEC-003's outcomes, not to opening a round -- the Brief's Access field names only Maya as opening rounds |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists; nothing is shown; Dinner Voting is None |
| Jordan (older kid, limited login -- Later) | No | No | Screen is not reachable; the older-kid row's Dinner Voting access is Own-only (casting a vote), never Full, and kid-initiated round creation is an explicit Non-Goal of this feature |
| Riley (Operator, support) | No | No | Dinner Voting: None; not exposed through support access |
| Unauthenticated | No | No | Redirected to sign-in |
| Expired session | No | No | Maya's session has expired: a re-authentication prompt appears; in-progress option selections are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Title "Open a vote for {night}" with a back arrow (returns to Sunday Plan Review) and an "Open Vote" action button (right-aligned; enabled once 2-3 options are selected).

**Body:** A helper line, "Choose 2 to 3 options for the older kids to vote on," above a list of the night's AI-proposed, safety-checked candidate options. Each option is shown as a safety-checked option card (recipe name, "checked against allergies" badge, "always check labels" disclaimer -- the same pattern used by FEAT-17.SPEC-001 and FEAT-17.SPEC-003) paired with a selection control for "offer this option."

**Footer:** None -- Open Vote is in the header.

### Responsive Behavior

- **Compact breakpoint:** Options stack vertically at full width, selection control aligned with each card; the header (including Open Vote) remains reachable via a sticky header while scrolling.
- **Medium size class and above:** Options may lay out in a two-column grid; the header is unchanged.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-03 (Sunday Plan Review) | Screen closes | Standard transition back |
| Option selection control | Tap | Toggles that option's selected state for the round | Selected state shown on the card; Open Vote enables once 2-3 options are selected | Visual selected state |
| Option selection control (attempting a 4th selection) | Tap | Rejected by FEAT-17.SPEC-006 (maximum 3 options per round) | Selection unchanged | Message "You can offer at most 3 options." |
| Open Vote button (2-3 options selected) | Tap | Triggers FEAT-17.SPEC-004 (Voting Round Creation & Safety Validation) | Button shows loading state | Success: confirmation "Vote opened for {night}" and navigation to FEAT-03; Failure: inline error naming the reason |
| Open Vote button (fewer than 2 selected) | Tap | Disabled -- no action | Button remains disabled | Helper text "Choose at least 2 options." |

### Accessibility Notes

- **Focus order:** Back arrow -> each option's selection control in list order -> Open Vote.
- **Selection feedback:** the running selection count and any validation message ("Choose at least 2 options.", "You can offer at most 3 options.") are announced to assistive technology as they change.
- **Keyboard alternatives:** every selection control and Open Vote are keyboard-operable (Enter/Space); there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Candidate options list shows a loading placeholder | Screen opens, fetching the night's safety-checked candidates | Options load |
| Ready, none selected (default) | All candidate options shown unselected; Open Vote disabled | Options have loaded | Maya selects an option |
| Selecting (1 selected) | Open Vote remains disabled | One option is selected | A second option is selected, or the selection is cleared |
| Ready to open (2-3 selected) | Open Vote enabled | 2 or 3 options are selected | Maya taps Open Vote, or deselects below 2 |
| Opening | Open Vote shows a loading state | Maya taps Open Vote | FEAT-17.SPEC-004 completes (success or failure) |
| Error | Inline banner "Could not open the vote. Try again." with a retry action | FEAT-17.SPEC-004 reports a failure | Maya taps Retry, or navigates away |
| Empty (no eligible options) | Message "No safety-checked options are available for this night yet." with no selection controls | The selected night has fewer than 2 safety-checked candidate options from FEAT-03 | Maya picks a different night, or FEAT-03/FEAT-02 make more options available |
| Offline/Degraded | N/A -- opening a round always requires connectivity to re-confirm the live safety check and create the round record; Open Vote is disabled with the message "Reconnect to open a vote." while offline | Connectivity is lost | Connectivity is restored |

## Validation Rules

Validation governed by FEAT-17.SPEC-006 (Dinner Voting Rules -- Access, Validation & Conflict Resolution). See that spec for the option-count and safety-check limits. This screen applies validation live as Maya selects and deselects options, and again when FEAT-17.SPEC-004 processes the Open Vote action.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-03 (Sunday Plan Review) | FEAT-03 |
| Successful Open Vote | FEAT-03 (Sunday Plan Review, updated) | FEAT-03 |

## Data Model

**Creates:** None directly -- the Open Vote action triggers FEAT-17.SPEC-004, which creates the Dinner Vote round.
**Reads:** Weekly Plan -- the selected night's AI-proposed candidate recipes and their current safety badges (sourced from FEAT-03 and FEAT-02); Member Profile -- confirms Maya's Organiser role for the Access and Visibility check.
**Updates:** None.
**Deletes:** None.

## Business Rules

- A round must offer 2-3 options, each already safety-checked (FEAT-17.SPEC-006; XBR-01).
- Opening a round is Maya-exclusive (Access Matrix: Dinner Voting Full for Maya only).
- Options offered are drawn only from FEAT-03's proposed alternatives for the selected night, never from another night.
- At most one open round exists per night (Non-Functional Notes: data volumes).

## Edge Cases

- **A selected option fails a mid-week safety re-check before Maya taps Open Vote** -- The option is removed from the candidate list live (FEAT-02's responsibility, per this feature's Side-Effect Inventory); if that drops the selection below 2, Open Vote disables again with "Choose at least 2 options."
- **Maya navigates away with a partial selection** -- Selections are not preserved; returning to this screen starts fresh, since no round exists yet to hold a draft.
- **Maya opens the same night's vote from two devices at once** -- FEAT-17.SPEC-004 accepts the first successful round creation for that night; the second attempt is rejected with "A vote is already open for this night."
- **The selected night already has an open round** -- The screen shows "A vote is already open for {night}." instead of the selection interface, with a link to FEAT-17.SPEC-003 to view its current state.
- **The selected night already has a resolved round** -- The screen shows "This night's vote is already decided." with a link to FEAT-17.SPEC-003, since a resolved round is never reopened (FEAT-17.SPEC-006).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-004 (Voting Round Creation & Safety Validation) | Triggers (outbound) | Open Vote triggers round creation and validation |
| FEAT-17.SPEC-006 (Dinner Voting Rules) | References (inbound) | Option-count and safety-check validation applied to Maya's selection |
| FEAT-17.SPEC-003 (Vote Outcome & Resolution) | Navigation (outbound, via "already open"/"already decided" link) | Lets Maya view a night's existing round instead of opening a new one |
| FEAT-03 (AI Weekly Dinner Plan Generation) | Navigation (inbound/outbound); References (inbound) | Entry from Sunday Plan Review; supplies the night's candidate options |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | References (inbound) | Safety badge shown on each candidate option |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| voting_round_setup_option_selected | night, option count currently selected | Maya toggles an option's selection | N/A -- no metric in success-metrics.md carries Older-Kid Dinner Voting (FEAT-17) as its Connected Feature; this Later-phase feature has no metric of its own in the current success-metrics set |

## Acceptance Criteria

**FEAT-17.SPEC-002-AC-01:** Given Maya is on the Voting Round Setup screen for a night, when she selects 2 options and taps Open Vote, then FEAT-17.SPEC-004 runs and, on success, she sees "Vote opened for {night}" and returns to FEAT-03.

**FEAT-17.SPEC-002-AC-02:** Given Maya is on the Voting Round Setup screen, when she taps the back arrow, then she returns to Sunday Plan Review (FEAT-03).

**FEAT-17.SPEC-002-AC-03:** Given Maya has already selected 3 options, when she attempts to select a 4th, then the selection is rejected with "You can offer at most 3 options." and the 4th option remains unselected.

**FEAT-17.SPEC-002-AC-04:** Given Maya has selected only 1 option, when she looks at the Open Vote button, then it is disabled and shows "Choose at least 2 options."

**FEAT-17.SPEC-002-AC-05:** Given Maya taps Open Vote and FEAT-17.SPEC-004 reports a failure, when the failure occurs, then an inline banner "Could not open the vote. Try again." appears with a retry action.

**FEAT-17.SPEC-002-AC-06:** Given the selected night has fewer than 2 safety-checked candidate options, when Maya opens this screen for that night, then it shows "No safety-checked options are available for this night yet." with no selection controls.

**FEAT-17.SPEC-002-AC-07:** Given Maya is offline, when she views the Voting Round Setup screen, then Open Vote is disabled with "Reconnect to open a vote."

**FEAT-17.SPEC-002-AC-08:** Given Maya has selected 2 options and one of them fails a mid-week safety re-check before she taps Open Vote, when the re-check completes, then that option is removed from the list and, since only 1 valid selection remains, Open Vote disables with "Choose at least 2 options."

**FEAT-17.SPEC-002-AC-09:** Given the selected night already has an open round, when Maya opens this screen for that night, then it shows "A vote is already open for {night}." with a link to FEAT-17.SPEC-003 instead of the selection interface.

**FEAT-17.SPEC-002-AC-10:** Given Maya opens this screen on her phone and her laptop for the same night at nearly the same time and submits Open Vote from both, when FEAT-17.SPEC-004 processes both submissions, then only the first succeeds and the second is rejected with "A vote is already open for this night."

**FEAT-17.SPEC-002-AC-11:** Given Maya opens the Voting Round Setup screen, when the night's candidate options are still loading, then a loading placeholder appears in place of the options list.

**FEAT-17.SPEC-002-AC-12:** Given Maya has selected exactly 2 options, when she views the Open Vote button, then it becomes enabled.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 7 (loading, selecting, ready to open, opening, error, empty, offline) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



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



# Automation Spec: Voting Round Creation & Safety Validation

## Overview

**Name:** Voting Round Creation & Safety Validation
**ID:** FEAT-17.SPEC-004
**Type:** Automation
**Purpose:** When Maya opens a voting round, validates the chosen options against the 2-3 count and FEAT-02's safety check, creates the round, and fires dinner_vote_opened.
**Parent Feature:** FEAT-17 -- Older-Kid Dinner Voting

## Scope and Non-Goals

**In Scope:**
- Validating the option count (2-3) for a round Maya is opening
- Re-confirming each selected option's current safety-check status against the household's Dietary Rules before the round can exist
- Preventing more than one open round per night
- Creating the Dinner Vote round record and firing dinner_vote_opened

**Non-Goals:**
- Computing the safety determination itself -- owned by FEAT-02 (Dietary Rules & Allergy Safety Engine); this automation only re-confirms FEAT-02's existing determination at round-creation time, per XBR-01's authority assignment.
- Generating the candidate options -- owned by FEAT-03 (AI Weekly Dinner Plan Generation); this automation validates only the options Maya has already selected from FEAT-03's proposed alternatives.
- Recomputing the tally once votes are cast, or resolving the round -- handled by FEAT-17.SPEC-005 (Vote Round Resolution & Fallback).

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Maya taps "Open Vote" | FEAT-17.SPEC-002 (Voting Round Setup) | Maya has selected a night and 2-3 candidate options and submits the action | Selected night, the list of selected options (2-3 recipe references), each option's most recently computed safety-check status, Maya's Member Profile reference |

## Processing Logic

1. Receive the selected night and the selected option list (2-3 recipe references) from the triggering screen.
2. Check whether an open or resolved Dinner Vote round already exists for that night; if so, stop and return the "round already exists" outcome.
3. Check that the selected option count is exactly 2 or 3; if not, stop and return the "invalid option count" outcome.
4. Re-confirm each selected option's current safety-check status against the household's Dietary Rules via FEAT-02; a mid-week rule change could have altered a status since FEAT-03 last proposed the option.
5. If every selected option currently passes the safety check, create the Dinner Vote round: night, the validated 2-3 options, status Open.
6. Fire dinner_vote_opened.
7. If one or more selected options no longer pass the safety check, stop without creating the round and return the "option fails safety check" outcome, naming each failing option.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Round opened | All checks pass (round-uniqueness, option count, safety re-check) | Dinner Vote round created with status Open | Confirmation "Vote opened for {night}" | FEAT-17.SPEC-002, FEAT-17.SPEC-001, FEAT-17.SPEC-003 |
| Rejected -- round already exists | A round (open or resolved) already exists for this night | None | "A vote is already open for this night." (open) or "This night's vote is already decided." (resolved), each with a link to FEAT-17.SPEC-003 | FEAT-17.SPEC-002 |
| Rejected -- invalid option count | Fewer than 2 or more than 3 options selected | None | "Choose 2 to 3 options." | FEAT-17.SPEC-002 |
| Rejected -- option fails safety check | One or more selected options no longer pass FEAT-02's check | None | "{Option name} no longer passes the allergy check and can't be offered." naming each failing option | FEAT-17.SPEC-002 |
| Automation failure | Processing error (e.g., the safety-check re-confirmation cannot complete) | None | "Could not open the vote. Try again." -- blocking, since a round is never created on an unconfirmed safety check (XBR-01's fail-closed rule) | FEAT-17.SPEC-002 |

## Data Model

**Reads:** Dietary Rule (via FEAT-02) -- each candidate option's current pass/fail safety status for the household; existing Dinner Vote rounds for the night (to detect an already-open or already-resolved round).
**Creates:** Dinner Vote -- round (night, the validated 2-3 options), status Open.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-01: every path onto the plan -- including voting options -- passes the same app-enforced allergy check before anyone sees it; the check fails closed, so a round is never created offering an option that does not currently pass.
- At most one round (open or resolved) exists per night (Non-Functional Notes: data volumes).
- A round must offer exactly 2 or 3 options (FEAT-17.SPEC-006).
- This automation runs synchronously -- FEAT-17.SPEC-002 waits for its result before confirming the round is open.

## Edge Cases

- **One selected option fails the re-check but the other 1-2 still pass, leaving fewer than 2 valid options** -- The round is rejected in full, not partially opened; Maya must re-select from the remaining valid candidates in FEAT-17.SPEC-002.
- **A household's dietary rule changes at the exact moment this automation runs** -- This automation's own safety re-check at round-creation time is authoritative; an option affected by the change is treated as failing and the round is rejected, consistent with the fail-closed rule.
- **Concurrent trigger firing (Maya opens rounds for two different nights at effectively the same time)** -- Each run evaluates its own night independently with no shared state, so both can succeed.
- **Trigger fires while a previous run for the same night is in flight (a double-tap on Open Vote, or two devices submitting for the same night close together)** -- The Open Vote action is disabled during submission on the triggering screen (FEAT-17.SPEC-002), and this automation's own round-uniqueness check (step 2) treats the check-then-create sequence for a given night as atomic, so a second run for the same night is rejected with "A vote is already open for this night." even if it started before the first run's round became visible to it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-17.SPEC-002 (Voting Round Setup) | Triggered by (inbound); Affects (outbound) | Fires on Open Vote tap; returns success or a specific rejection reason to the screen |
| FEAT-17.SPEC-001 (Dinner Vote Casting) | Affects (outbound) | A successfully opened round becomes votable on this screen |
| FEAT-17.SPEC-006 (Dinner Voting Rules) | References (inbound) | Option-count and safety-check rules this automation enforces |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | References (inbound) | Re-confirms each candidate option's safety status |
| FEAT-03 (AI Weekly Dinner Plan Generation) | References (inbound) | Candidate options originate from FEAT-03's proposed alternatives for the night |

## Analytics and Success Signals

- **dinner_vote_opened** (night, option count, options offered by recipe reference) -- N/A -- no metric in success-metrics.md carries Older-Kid Dinner Voting (FEAT-17) as its Connected Feature; this Later-phase feature has no metric of its own in the current success-metrics set.
- **dinner_vote_open_rejected** (reason: round_already_exists / invalid_option_count / safety_check_failed / processing_error) -- N/A -- same reason as above.

## Acceptance Criteria

**FEAT-17.SPEC-004-AC-01:** Given Maya has selected 2 safety-checked options for a night with no existing round, when she submits Open Vote, then a Dinner Vote round is created with status Open, dinner_vote_opened fires, and she sees "Vote opened for {night}."

**FEAT-17.SPEC-004-AC-02:** Given Maya has selected 3 safety-checked options for a night with no existing round, when she submits Open Vote, then a Dinner Vote round is created offering all 3 options.

**FEAT-17.SPEC-004-AC-03:** Given a night already has an open round, when Maya attempts to open another round for the same night, then no new round is created and she sees "A vote is already open for this night."

**FEAT-17.SPEC-004-AC-04:** Given a night already has a resolved round, when Maya attempts to open a new round for the same night, then no new round is created and she sees "This night's vote is already decided."

**FEAT-17.SPEC-004-AC-05:** Given Maya has selected only 1 option, when she submits Open Vote, then no round is created and she sees "Choose 2 to 3 options."

**FEAT-17.SPEC-004-AC-06:** Given Maya has selected 2 options and one of them no longer passes FEAT-02's safety check at the moment of submission, when this automation runs, then no round is created and Maya sees "{Option name} no longer passes the allergy check and can't be offered." naming that option.

**FEAT-17.SPEC-004-AC-07:** Given the safety-check re-confirmation cannot complete due to a processing error, when Maya submits Open Vote, then no round is created and she sees "Could not open the vote. Try again."

**FEAT-17.SPEC-004-AC-08:** Given Maya double-taps Open Vote for the same night in quick succession, when both submissions reach this automation, then only the first creates a round and the second is rejected with "A vote is already open for this night."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |



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
