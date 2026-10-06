---
document_type: spec
spec_type: screen
spec_id: FEAT-17.SPEC-002
spec_name: Voting Round Setup
spec_slug: voting-round-setup
parent_feature: FEAT-17
parent_feature_name: Older-Kid Dinner Voting
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

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
