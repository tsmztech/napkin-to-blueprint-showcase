---
document_type: spec
spec_type: screen
spec_id: FEAT-04.SPEC-003
spec_name: Review Swap Suggestions
spec_slug: review-swap-suggestions
parent_feature: FEAT-04
parent_feature_name: One-Tap Meal Swap
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Screen Spec: Review Swap Suggestions

## Overview

**Name:** Review Swap Suggestions
**ID:** FEAT-04.SPEC-003
**Type:** Screen
**Purpose:** Maya sees every pending swap suggestion on the current plan and accepts or declines each with one tap.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap

## Scope and Non-Goals

**In Scope:**
- Listing all pending (Suggested-outcome) Swap Suggestions on the current Weekly Plan
- Showing each suggestion's member, night, and proposed recipe
- One-tap accept (triggers FEAT-04.SPEC-004) and one-tap decline
- Reflects a suggestion suggested through both One-Tap Meal Swap (this feature) and the equivalent "pick suggestion" flow in Manual Weekly Planning (FEAT-23), since both write to the same Swap Suggestion entity

**Non-Goals:**
- Creating a suggestion -- that is Sam's action in FEAT-04.SPEC-002 (Suggest a Swap) or the FEAT-23 equivalent
- Applying a swap directly without a suggestion -- Maya's direct path is FEAT-04.SPEC-001 (Meal Swap Direct)
- A confirmation dialog before accept or decline -- excluded per product-features.md's Communications/Primary Flows: the product's "one tap" promise is deliberately confirmation-free for this action, matching the Shared UI Pattern declared in the Brief
- Withdrawing a suggestion on the suggester's behalf -- no withdrawal path exists in the product definition; only accept, decline, or lapse are modeled outcomes

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Weekly Plan (FEAT-03) | Maya reviews Sam's pending suggestion from the week plan screen (Sunday Plan Review, step 3) | The current Weekly Plan reference; the list scopes to its pending suggestions |
| Swap Suggestion Notifications (FEAT-04.SPEC-006) | Maya taps a "suggestion arrived" notification | The specific suggestion's slot, opened directly within the list (scrolled into view) |
| Manual Weekly Planning (FEAT-23) | Maya reviews a suggested pick through the same accept/decline flow (Free-Tier Manual Week, step 3) | The current Weekly Plan reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Accept or decline any pending suggestion | -- |
| Sam (Other Adult Member) | Not shown | Not shown | Sam's Own-only access to Meal Swap covers raising suggestions, not reviewing others'; he sees the status of his own suggestion within FEAT-04.SPEC-002 instead. There is only ever one other adult member's suggestions to review in this product (one household per account), so no case of Sam reviewing another member's suggestion arises. |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this role |
| Jordan (older kid, limited login -- Later) | No | No | Meal Swap access is None for this role (Access Matrix); the screen is not shown |
| Riley (Operator, support) | No | No | Meal Swap access is None for this role; not shown in the support read-only view |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in as Maya, she lands on the Weekly Plan, not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no unsaved action exists on this screen since accept/decline commits immediately, so nothing is lost |

## Layout and Content

**Header:** Screen title "Swap suggestions" with a back arrow (returns to the Weekly Plan) and a count badge showing the number of pending suggestions.

**Body:** A vertically scrolling list, one card per pending Swap Suggestion, ordered by night (earliest first). Each card shows: the suggesting member's name, the night, the proposed recipe's name, cook time, and safety badge, plus two buttons -- "Accept" and "Decline" -- side by side at the bottom of the card.

**Footer:** None.

**Empty-list content:** When no suggestions are pending, the body instead shows a single centered message: "No pending swap suggestions" with a short explanation ("Suggestions from other household members will appear here.").

### Responsive Behavior

- **Compact breakpoint:** Single-column list of full-width cards, Accept and Decline stacked side by side within each card.
- **Medium size class and above:** Same single-column list, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the Weekly Plan | Screen closes | Standard back transition |
| Accept button (per card) | Tap | 1. Acquire the concurrency lock via FEAT-04.SPEC-009 for that slot. 2. If acquired, trigger FEAT-04.SPEC-004 (Apply Meal Swap) with the suggestion's proposed recipe. 3. On success, FEAT-04.SPEC-010 marks the suggestion Accepted. | Card shows a brief loading indicator, then is removed from the list on success | Toast "Swap applied -- {night}'s dinner is now {recipe name}." List count badge decrements |
| Decline button (per card) | Tap | Sets the suggestion's outcome to Declined via FEAT-04.SPEC-010 | Card shows a brief loading indicator, then is removed from the list | Toast "Suggestion declined." List count badge decrements; the suggester is notified (FEAT-04.SPEC-006) |
| Accept/Decline (while a card is processing) | Tap on the same card | No action -- debounced | None | Buttons remain disabled on that card during processing |

### Accessibility Notes

- **Focus order:** Back arrow -> each suggestion card in night order, with Accept before Decline within each card.
- **Dynamic announcements:** Removing a card from the list on accept or decline is announced along with its outcome toast. The count badge's updated value is announced. The empty-state message is announced when the list becomes empty.
- **Keyboard alternatives:** Accept and Decline are reachable and actionable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief inline loading indicator in place of the list | Screen first opens | Suggestions load (success or empty) or the load fails |
| Populated | One or more suggestion cards shown | Suggestions load with at least one pending | The last suggestion is resolved or the screen is closed |
| Empty | "No pending swap suggestions" message | Suggestions load with none pending, or the last one is resolved while viewing | A new suggestion arrives |
| Processing (per card) | That card shows a loading indicator, Accept/Decline disabled | Accept or Decline tapped on a card | The action completes (success or failure) |
| Error (load failed) | Banner "Couldn't load swap suggestions. Check your connection and try again." with a Retry button | Initial load fails | User taps Retry |
| Error (action failed, per card) | That card shows "Couldn't complete this action. Try again." with the buttons re-enabled | Accept or Decline fails | User retries the same action, or the card resolves on a later attempt |
| Offline/Degraded | Banner "You're offline -- accepting or declining needs a connection." at the top; cards remain viewable but Accept/Decline are disabled | Connectivity lost while this screen is open | Connectivity restored -- buttons re-enable automatically |

## Validation Rules

No user input is captured on this screen beyond the accept/decline choice. Eligibility and superseding rules are governed by FEAT-04.SPEC-010 (Suggestion Lifecycle Rules); the safety re-check on accept is governed by FEAT-04.SPEC-004 (Apply Meal Swap), which references FEAT-02.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | Weekly Plan | FEAT-03 or FEAT-23 |
| Successful accept | Stays on this screen with the card removed; the Weekly Plan reflects the swap on next view | -- |

## Data Model

**Creates:** None.
**Reads:** Swap Suggestion -- suggesting_member, night, proposed_recipe, outcome (filtered to outcome = Suggested for the current Weekly Plan); Planned Meal -- read indirectly through FEAT-04.SPEC-004 on accept, to confirm the target slot.
**Updates:** Swap Suggestion -- outcome set to Declined directly by this screen's Decline action (via FEAT-04.SPEC-010); outcome set to Accepted by FEAT-04.SPEC-010 following a successful FEAT-04.SPEC-004 apply.
**Deletes:** None -- declined suggestions are retained with a terminal outcome, never removed (per the Brief's Entity-Lifecycle Coverage Matrix).

## Business Rules

- XBR-06: the organiser accepts or declines each suggestion with one tap; a suggestion not answered before its night lapses (FEAT-04.SPEC-005) and is removed from this list at that point.
- Accepting a suggestion re-verifies safety before the swap applies (FEAT-04.SPEC-004) -- a suggestion that was safe when raised but fails a hard rule tightened since then is refused at accept time, not silently applied.
- Accepting or declining a suggestion whose night has already passed is refused: the suggestion has already lapsed and no longer appears in this list (FEAT-04.SPEC-010, late-accept refusal).
- Accepting a suggestion acquires the same concurrency lock as a direct swap (FEAT-04.SPEC-009) -- Maya cannot accept two suggestions for the same slot, and cannot accept one while her own direct swap on that slot is in flight from another device.

## Edge Cases

- **Maya taps Accept and Decline on two different cards in quick succession** -- Each processes independently; there is no shared lock between different slots.
- **The suggestion's recipe fails a re-check at accept time (hard rule tightened since it was suggested) -- concurrent-edit conflict** -- Accept is rejected with "This suggestion no longer passes the household's dietary rules and can't be applied." The card updates to show this message with only a Decline option remaining. Resolution: reject-with-refresh, consistent with the Planned Meal and Swap Suggestion Contention notes in the dependency map -- a safety-relevant change always wins over a pending action.
- **A suggestion lapses while Maya is viewing this screen (its night passes during the session)** -- The card is removed from the list on the next automatic refresh (or immediately if the screen is live-updating), and the suggester is notified separately (FEAT-04.SPEC-005 / FEAT-04.SPEC-006); Maya sees no error, since lapse is not an action failure.
- **Two suggestions target the same slot** -- Per FEAT-04.SPEC-010, at most one suggestion per member per slot can be open; a second member's suggestion for the same slot appears as its own separate card, and accepting either one supersedes the other.
- **Network failure while accepting** -- Error (action failed, per card) state; the suggestion remains Suggested and the plan is unchanged.
- **Maya opens this screen with zero pending suggestions** -- Empty state is shown; no card list.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-004 (Apply Meal Swap) | Triggers (outbound) | Accept action re-verifies safety and applies the swap |
| FEAT-04.SPEC-009 (Swap Concurrency Lock) | Triggers (outbound) | Acquired before an accept proceeds |
| FEAT-04.SPEC-010 (Suggestion Lifecycle Rules) | References (outbound) | Governs decline, late-accept refusal, and superseding |
| FEAT-04.SPEC-006 (Swap Suggestion Notifications) | References (outbound, indirect) | Notifies the suggester of accept/decline outcomes |
| FEAT-04.SPEC-002 (Suggest a Swap) | Navigation (inbound, indirect) | Suggestions reviewed here originate there |
| FEAT-03 (Weekly Plan) | Navigation (inbound/outbound) | Entry point and return destination |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| swap_suggestion_accepted | slot night, suggesting member | Maya accepts a suggestion and the swap applies successfully | supports success-metrics.md: "Weekly Planning Time" |
| swap_suggestion_declined | slot night, suggesting member | Maya declines a suggestion | supports success-metrics.md: "Weekly Planning Time" |

## Acceptance Criteria

**FEAT-04.SPEC-003-AC-01:** Given Maya opens this screen with two pending suggestions from Sam, when the list loads, then both appear as separate cards ordered by night, and the header shows a count of 2.

**FEAT-04.SPEC-003-AC-02:** Given Maya is viewing a pending suggestion card, when she taps Accept, then the swap applies, the card is removed, and a toast reads "Swap applied -- {night}'s dinner is now {recipe name}."

**FEAT-04.SPEC-003-AC-03:** Given Maya is viewing a pending suggestion card, when she taps Decline, then the suggestion's outcome is set to Declined, the card is removed, and Sam is notified.

**FEAT-04.SPEC-003-AC-04:** Given Maya has no pending suggestions, when she opens this screen, then she sees "No pending swap suggestions."

**FEAT-04.SPEC-003-AC-05:** Given a household dietary rule was tightened since Sam's suggestion was raised, when Maya taps Accept, then the accept is rejected with "This suggestion no longer passes the household's dietary rules and can't be applied." and only Decline remains available on that card.

**FEAT-04.SPEC-003-AC-06:** Given a swap is already in flight for the same slot from another device, when Maya taps Accept on a suggestion for that slot, then the action fails and the card shows "Couldn't complete this action. Try again."

**FEAT-04.SPEC-003-AC-07:** Given Maya taps Accept and the network fails, when the failure occurs, then the card shows "Couldn't complete this action. Try again." and the suggestion remains Suggested.

**FEAT-04.SPEC-003-AC-08:** Given Maya arrives on this screen via a "suggestion arrived" notification, when the screen opens, then the relevant suggestion's card is scrolled into view.

**FEAT-04.SPEC-003-AC-09:** Given a suggestion's night passes while Maya is viewing this screen, when the lapse occurs, then its card is removed without an error shown to Maya.

**FEAT-04.SPEC-003-AC-10:** Given Maya loses connectivity while on this screen, when she looks at any card's Accept/Decline buttons, then they are disabled with the banner "You're offline -- accepting or declining needs a connection."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 7 (loading, populated, empty, processing, error-load, error-action, offline) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
