---
document_type: spec
spec_type: screen
spec_id: FEAT-04.SPEC-001
spec_name: Meal Swap (Direct)
spec_slug: meal-swap-direct
parent_feature: FEAT-04
parent_feature_name: One-Tap Meal Swap
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Meal Swap (Direct)

## Overview

**Name:** Meal Swap (Direct)
**ID:** FEAT-04.SPEC-001
**Type:** Screen
**Purpose:** Maya taps swap on a planned meal, sees safe alternatives (or a scarcity explanation), and picks one to update the slot immediately.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap

## Scope and Non-Goals

**In Scope:**
- Opening the alternatives list for one planned dinner slot and completing a direct swap in one interaction
- Showing a scarcity explanation when few or zero alternatives qualify
- Reflecting the concurrency lock (FEAT-04.SPEC-009) when a swap is already in flight for the slot
- Repeated swaps on the same slot within the same week

**Non-Goals:**
- Sam suggesting a swap instead of applying one directly -- excluded per scope-boundaries.md SC-04: other adult members "see the plan and suggest swaps" only; the suggest-and-approve path is FEAT-04.SPEC-002 (Suggest a Swap)
- Reviewing or acting on pending swap suggestions from other members -- handled by FEAT-04.SPEC-003 (Review Swap Suggestions)
- Computing which alternatives qualify -- owned by FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation); this screen only displays what that spec returns
- Withdrawing a suggestion Sam raised -- not modeled anywhere in the feature per the Brief's Non-Goals; a suggestion's only outcomes are accepted, declined, or lapsed

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Weekly Plan (FEAT-03) or Manual Weekly Planning (FEAT-23) | Maya taps "swap" on a planned dinner | The target Planned Meal's night, current recipe, and Weekly Plan reference |
| Report a Safety Concern acknowledgement (FEAT-02) | Maya picks a replacement after a safety-concern removal vacates a slot | The vacated slot's night; no current recipe to display since the meal was removed |
| Mid-week dietary rule tightening (FEAT-01 / FEAT-02, XBR-02) | Maya opens a now-unsafe meal the safety re-check flagged | The flagged slot's night and its now-failing recipe |
| FEAT-02.SPEC-011 (Safety Concern Reporter Acknowledgement) | Reporter taps "Show me alternatives" | The emptied slot's night; no current recipe to display since the meal was removed |
| FEAT-02.SPEC-012 (Safety Concern Organiser Alert) | Maya taps "Choose a replacement" (in-app or push) | The emptied slot's night; no current recipe to display since the meal was removed |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Swap the meal directly: request alternatives and pick one | -- |
| Sam (Other Adult Member) | Not shown | Not shown | Sam's "swap" tap on the Weekly Plan navigates him to FEAT-04.SPEC-002 (Suggest a Swap) instead of this screen; this screen is never reachable by Sam, consistent with his Own-only access to Meal Swap (Access Matrix, user-persona.md) |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this role; there is no path into the product to reach this screen |
| Jordan (older kid, limited login -- Later) | No | No | The swap affordance is not shown on the Weekly Plan for this role; direct navigation is redirected to the current week's plan with no message, since Meal Swap access is None for this role |
| Riley (Operator, support) | No | No | Not shown in the support read-only view; Meal Swap access is None for this role (Access Matrix) |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in as Maya, the user lands back on the Weekly Plan, not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- if alternatives were already loaded, they are discarded; no swap has been applied yet so nothing is lost |

## Layout and Content

**Header:** Screen title "Swap {night}'s dinner" with a back arrow (returns to the Weekly Plan).

**Body, top region -- Current Meal card:** Shows the meal being replaced: recipe name, cook time, the "checked against allergies" safety badge with the "always check labels" disclaimer, and a "Swap" button. When the slot was vacated by a safety-concern removal or a mid-week rule change, this region instead shows "This slot is open" with no recipe, and the button reads "Choose a replacement."

**Body, middle region -- Alternatives list:** Appears once "Swap" (or "Choose a replacement") is tapped. A vertically scrolling list of candidate recipes, each showing recipe name, cook time, and the safety badge. Below the last item, a scarcity explanation line appears whenever FEAT-04.SPEC-008 returns a limited or zero-count result (e.g., "Only 2 options fit tonight's 30-minute limit and everyone's dietary rules."). Each list item is tappable.

**Body, inline loading indicator:** Replaces the alternatives list while the request to FEAT-04.SPEC-008 is in flight; shown directly below the Current Meal card.

**Footer:** None -- all actions are in the body.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described; the alternatives list scrolls independently of the Current Meal card, which stays pinned above it.
- **Medium size class and above:** Same single-column structure, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the Weekly Plan (FEAT-03 or FEAT-23) | Screen closes | Standard back transition |
| Swap / Choose a replacement button | Tap | Requests alternatives via FEAT-04.SPEC-008 (which routes to FEAT-04.SPEC-007 for AI-originated plans or a recipe-library filter for manually-built plans) | Screen enters Loading Alternatives state | Inline loading indicator appears below the Current Meal card |
| Alternative list item | Tap | 1. Acquire the concurrency lock via FEAT-04.SPEC-009. 2. If acquired, trigger FEAT-04.SPEC-004 (Apply Meal Swap) with the chosen recipe. | Screen enters Applying state | Selected item shows a brief loading indicator; other items are disabled |
| Alternative list item (while Applying) | Tap | No action -- debounced | None | Disabled items ignore taps |
| Retry button (Error state) | Tap | Re-requests alternatives via FEAT-04.SPEC-008 | Screen re-enters Loading Alternatives state | Inline loading indicator reappears |

### Accessibility Notes

- **Focus order:** Back arrow -> Current Meal card -> Swap/Choose a replacement button -> each alternatives list item in display order -> scarcity explanation text (if shown) -> Retry button (if shown).
- **Dynamic announcements:** The transition into Loading Alternatives is announced ("Loading alternatives"). When the alternatives list renders, its item count is announced. The scarcity explanation is announced immediately after the list. A successful swap's navigation away is announced by the destination screen; a failed swap announces the error banner text.
- **Keyboard alternatives:** Every action (swap, alternative selection, retry, back) is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default | Current Meal card shown with Swap button enabled | Screen first opens | User taps Swap / Choose a replacement |
| Loading Alternatives | Inline loading indicator below the Current Meal card | Swap tapped | Alternatives returned (success) or the request fails |
| Alternatives Shown | Alternatives list rendered, with a scarcity explanation line when applicable | Alternatives load successfully | User selects an alternative, or navigates away |
| Applying | Selected alternative shows a loading indicator; other items disabled | User selects an alternative and the concurrency lock is acquired | FEAT-04.SPEC-004 completes (success) or fails |
| Error (alternatives fetch) | Banner "Couldn't load alternatives. Check your connection and try again." with a Retry button; Current Meal card remains visible and unchanged | Alternatives request fails | User taps Retry, or navigates away |
| Error (swap failed) | Banner "Couldn't complete the swap. Your original dinner is still on the plan." with a Retry button on the previously selected item; original meal remains shown unchanged | FEAT-04.SPEC-004 reports failure | User retries the same selection, picks a different alternative, or navigates away |
| Locked (concurrent swap in flight) | Banner "This meal is already being swapped." shown in place of the alternatives list; the Current Meal card stays visible | Concurrency lock acquisition (FEAT-04.SPEC-009) fails because another swap operation on the same slot is already active | The other swap operation completes and the slot's status updates, at which point the screen re-fetches and returns to Default |
| Offline/Degraded | Banner "You're offline -- swapping needs a connection." at the top; the Current Meal card remains viewable but the Swap button is disabled with the same explanation | Connectivity lost while this screen is open | Connectivity restored -- the Swap button re-enables automatically |

## Validation Rules

Validation of which recipes qualify as alternatives is governed by FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation). This screen applies no independent field-level validation -- selection is limited entirely to items in the list that spec returns.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | Weekly Plan | FEAT-03 (AI Weekly Dinner Plan Generation) or FEAT-23 (Manual Weekly Planning), whichever produced the current plan |
| Successful swap | Weekly Plan, showing the updated slot | FEAT-03 or FEAT-23 |
| Cancel from Error or Locked state (back arrow) | Weekly Plan, original meal unchanged | FEAT-03 or FEAT-23 |

## Data Model

**Creates:** None directly -- this screen initiates the swap but the write happens in FEAT-04.SPEC-004 (Apply Meal Swap).
**Reads:** Planned Meal -- recipe, cook_time, safety_badge, status (to show the current meal and detect an already-open slot); Weekly Plan -- to locate the target slot and confirm it belongs to the current week.
**Updates:** None -- writes to Planned Meal are performed by FEAT-04.SPEC-004, triggered from this screen.
**Deletes:** None.

## Business Rules

- Every alternative shown has already passed the same allergy/religious hard-rule check as original plan generation (XBR-01), enforced by FEAT-04.SPEC-008.
- At most one active swap operation runs per meal slot at a time (FEAT-04.SPEC-009); a second attempt on the same slot is refused with the Locked state above.
- Applying a swap here supersedes any open Swap Suggestion on the same slot (FEAT-04.SPEC-010) -- Maya is never blocked from swapping directly by a pending suggestion from Sam.
- A meal can be swapped more than once within the same week without restriction (product-features.md, Primary Flows & Alternates: Repeated swap).

## Edge Cases

- **User navigates away while alternatives are loading** -- The in-flight request is discarded; no partial state is shown when the user returns, and re-opening the screen starts fresh from Default.
- **User taps an alternative twice rapidly** -- The second tap is ignored while the first selection is in the Applying state (items disabled).
- **Network failure while applying the swap** -- Error (swap failed) state; the original meal remains in place, per product-features.md's States field.
- **Another household member's action changes the slot while this screen is open (concurrent-edit conflict)** -- If the slot's Planned Meal status changes (e.g., Maya accepted a pending suggestion for the same slot from another device, or FEAT-02 removes the meal for a new safety concern) while alternatives are shown, the next selection attempt is rejected with "This meal changed while you were choosing. Here's the latest." and the screen reloads the current slot state. Resolution: reject-with-refresh, per the dependency map's Contention note for the Planned Meal entity.
- **Slot has no qualifying alternatives at all** -- The scarcity explanation from FEAT-04.SPEC-008 is shown alone (e.g., "No safe alternatives fit tonight -- every recipe that qualifies is already in this week's plan or fails someone's allergy rule."), with no selectable items; the original meal remains in place.
- **User arrives via a safety-concern replacement with no current meal to show** -- The Current Meal card shows "This slot is open" instead of a recipe, and the flow proceeds identically from the alternatives request onward.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation) | References (outbound) | Supplies the filtered alternatives list and any scarcity explanation |
| FEAT-04.SPEC-007 (Swap Alternatives Generation) | References (outbound, indirect via SPEC-008) | Sourced for AI-originated plans |
| FEAT-04.SPEC-009 (Swap Concurrency Lock) | Triggers (outbound) | Acquired before an alternative selection proceeds to apply |
| FEAT-04.SPEC-004 (Apply Meal Swap) | Triggers (outbound) | Performs the actual write once the lock is acquired |
| FEAT-04.SPEC-010 (Suggestion Lifecycle Rules) | References (outbound, indirect via SPEC-004) | Governs superseding an open suggestion on the same slot |
| FEAT-03 (AI Weekly Dinner Plan Generation) / FEAT-23 (Manual Weekly Planning) | Navigation (inbound/outbound) | Entry point (Weekly Plan screen) and return destination -- the specific screen spec ID is assigned within those features |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | Navigation (inbound) | Safety-concern acknowledgement and mid-week rule re-check open this screen on a vacated or flagged slot |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| meal_swap_opened | entry source (weekly plan / safety-concern replacement / mid-week rule flag), slot night | Screen opens | supports success-metrics.md: "One-Tap Swap Completion" |
| meal_swap_alternatives_limited | candidate count | Alternatives return with a scarcity explanation | supports success-metrics.md: "One-Tap Swap Completion" |
| meal_swap_completed | slot night, time from tap to completion | FEAT-04.SPEC-004 reports success for a swap initiated here | supports success-metrics.md: "One-Tap Swap Completion" |
| meal_swap_failed | failure reason (fetch failed / apply failed / locked) | Error or Locked state is shown | supports success-metrics.md: "One-Tap Swap Completion" |

## Acceptance Criteria

**FEAT-04.SPEC-001-AC-01:** Given Maya is viewing Thursday's dinner on the Weekly Plan, when she taps "swap," then this screen opens showing the current meal's recipe, cook time, and safety badge.

**FEAT-04.SPEC-001-AC-02:** Given Maya is on this screen with the current meal shown, when she taps "Swap," then the screen enters Loading Alternatives and an inline indicator appears.

**FEAT-04.SPEC-001-AC-03:** Given alternatives have loaded with three qualifying options, when Maya taps one, then the concurrency lock is acquired, the swap is applied, and she is returned to the Weekly Plan showing the new dinner in that slot.

**FEAT-04.SPEC-001-AC-04:** Given alternatives have loaded and only two options qualify, when the list renders, then a scarcity explanation line appears below the two options.

**FEAT-04.SPEC-001-AC-05:** Given zero alternatives qualify for the slot, when Maya taps Swap, then the scarcity explanation appears alone with no selectable items, and the original meal stays in place.

**FEAT-04.SPEC-001-AC-06:** Given Maya taps Swap and the alternatives request fails, when the failure occurs, then the banner "Couldn't load alternatives. Check your connection and try again." appears with a Retry button.

**FEAT-04.SPEC-001-AC-07:** Given Maya selects an alternative and the swap write fails, when the failure occurs, then the banner "Couldn't complete the swap. Your original dinner is still on the plan." appears and the original meal is unchanged.

**FEAT-04.SPEC-001-AC-08:** Given a swap operation is already in flight for this slot from another device, when Maya opens this screen and taps Swap, then the Locked state banner "This meal is already being swapped." appears instead of an alternatives list.

**FEAT-04.SPEC-001-AC-09:** Given Maya loses connectivity while on this screen, when she looks at the Swap button, then it is disabled with the banner "You're offline -- swapping needs a connection."

**FEAT-04.SPEC-001-AC-10:** Given Sam (Other Adult Member) taps "swap" on a planned meal, when the tap registers, then he is navigated to FEAT-04.SPEC-002 (Suggest a Swap) rather than this screen.

**FEAT-04.SPEC-001-AC-11:** Given the underlying Planned Meal changes (another swap or a safety removal) while Maya's alternatives list is shown, when she selects an alternative, then the selection is rejected with "This meal changed while you were choosing. Here's the latest." and the screen reloads the current slot state.

**FEAT-04.SPEC-001-AC-12:** Given Maya arrives on this screen after acknowledging a safety-concern removal, when the screen opens, then the Current Meal card shows "This slot is open" instead of a recipe, and she can request alternatives normally.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 8 (default, loading, alternatives shown, applying, error-fetch, error-swap, locked, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
