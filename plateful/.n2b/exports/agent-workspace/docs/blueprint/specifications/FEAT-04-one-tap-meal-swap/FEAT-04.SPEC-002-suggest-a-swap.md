---
document_type: spec
spec_type: screen
spec_id: FEAT-04.SPEC-002
spec_name: Suggest a Swap
spec_slug: suggest-a-swap
parent_feature: FEAT-04
parent_feature_name: One-Tap Meal Swap
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Suggest a Swap

## Overview

**Name:** Suggest a Swap
**ID:** FEAT-04.SPEC-002
**Type:** Screen
**Purpose:** Sam picks a safe alternative for a night and sends it to the organiser as a suggestion, rather than applying it directly.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap

## Scope and Non-Goals

**In Scope:**
- Sam requesting alternatives for a planned dinner and submitting one as a suggestion
- Showing Sam the status of his own suggestion for that slot (Suggested, Accepted, Declined, Lapsed) once submitted
- The same scarcity-explanation pattern as FEAT-04.SPEC-001 when few or zero alternatives qualify

**Non-Goals:**
- Applying a swap directly -- excluded per scope-boundaries.md SC-04: only the organiser has direct swap authority (Access Matrix, Meal Swap: Sam Own-only); Maya's direct path is FEAT-04.SPEC-001
- Accepting or declining suggestions -- that authority belongs to Maya alone, in FEAT-04.SPEC-003 (Review Swap Suggestions)
- Withdrawing a submitted suggestion -- product-features.md's Primary Flows & Alternates and Validation & Limits define a suggestion's only outcomes as accepted, declined, or lapsed; no withdrawal path exists, so Sam waits for Maya's decision or the suggestion's lapse
- Submitting a second open suggestion for the same slot while one is already pending -- governed by FEAT-04.SPEC-010 (Suggestion Lifecycle Rules), which enforces one open suggestion per member per slot

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Weekly Plan (FEAT-03) or Manual Weekly Planning (FEAT-23) | Sam taps "swap" on a planned dinner | The target Planned Meal's night, current recipe, and Weekly Plan reference |
| Swap Suggestion Notifications (FEAT-04.SPEC-006) | Sam taps an "accepted", "declined", or "lapsed" notification | The suggestion's slot, showing its outcome |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Sam (Other Adult Member) | Full screen | Request alternatives and submit his own suggestion for the slot (Own-only) | -- |
| Maya (Organiser) | Not shown | Not shown | Maya's swap authority is direct (FEAT-04.SPEC-001); this screen is Sam's suggest-and-approve counterpart and is never Maya's entry point |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this role; there is no path into the product to reach this screen |
| Jordan (older kid, limited login -- Later) | No | No | The swap affordance is not shown on the Weekly Plan for this role; Meal Swap access is None (Access Matrix) |
| Riley (Operator, support) | No | No | Not shown in the support read-only view; Meal Swap access is None for this role |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in as Sam, the user lands back on the Weekly Plan, not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an unsubmitted selection is discarded; no suggestion has been recorded yet |

## Layout and Content

**Header:** Screen title "Suggest a swap for {night}" with a back arrow (returns to the Weekly Plan).

**Body, top region -- Current Meal card:** Recipe name, cook time, and safety badge for the meal currently in the slot, plus a "Suggest a swap" button.

**Body, middle region -- Alternatives list:** Appears once "Suggest a swap" is tapped -- the same short list of safety-checked candidates as FEAT-04.SPEC-001, identically structured (recipe name, cook time, safety badge, tappable), with the same scarcity-explanation line when few or zero options qualify.

**Body, pending-status region:** If Sam already has an open suggestion for this slot, the Current Meal card is replaced by a "Your suggestion" card showing the proposed recipe and the status label "Waiting for Maya," with no alternatives list shown beneath it.

**Body, resolved-status region:** If Sam's suggestion for this slot most recently resolved (Accepted, Declined, or Lapsed) and he has not since submitted a new one, a dismissible banner above the Current Meal card states the outcome (e.g., "Maya accepted your suggestion" / "Maya declined your suggestion" / "Your suggestion lapsed -- the night passed unanswered").

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described; the alternatives list scrolls independently of the pinned Current Meal (or "Your suggestion") card.
- **Medium size class and above:** Same single-column structure, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the Weekly Plan (FEAT-03 or FEAT-23) | Screen closes | Standard back transition |
| Suggest a swap button | Tap | Requests alternatives via FEAT-04.SPEC-008 | Screen enters Loading Alternatives state | Inline loading indicator appears below the Current Meal card |
| Alternative list item | Tap | Submits the suggestion via FEAT-04.SPEC-010 (creates the Swap Suggestion, enforcing one-open-per-member-per-slot); the new suggestion notifies Maya via FEAT-04.SPEC-006 (Swap Suggestion Notifications) | Screen enters Submitting state, then shows "Your suggestion" card | Selected item shows a brief loading indicator; other items disabled |
| Alternative list item (while Submitting) | Tap | No action -- debounced | None | Disabled items ignore taps |
| Outcome banner dismiss (x) | Tap | Dismisses the resolved-status banner | Banner disappears | Screen shows the Current Meal card underneath |
| Retry button (Error state) | Tap | Re-requests alternatives via FEAT-04.SPEC-008 | Screen re-enters Loading Alternatives state | Inline loading indicator reappears |

### Accessibility Notes

- **Focus order:** Back arrow -> outcome banner (if shown) -> Current Meal card or Your-suggestion card -> Suggest a swap button (if shown) -> each alternatives list item in order -> scarcity explanation text (if shown) -> Retry button (if shown).
- **Dynamic announcements:** The transition into Loading Alternatives is announced. On successful submission, "Suggestion sent" is announced and the screen updates to the "Your suggestion" card. The resolved-status banner text is announced when the screen opens with one present.
- **Keyboard alternatives:** Every action (suggest, alternative selection, dismiss banner, retry, back) is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default | Current Meal card shown with "Suggest a swap" button enabled | Screen opens with no open suggestion for this slot | User taps Suggest a swap, or a prior suggestion resolves |
| Loading Alternatives | Inline loading indicator below the Current Meal card | Suggest a swap tapped | Alternatives returned (success) or the request fails |
| Alternatives Shown | Alternatives list rendered, with a scarcity explanation line when applicable | Alternatives load successfully | User selects an alternative, or navigates away |
| Submitting | Selected alternative shows a loading indicator; other items disabled | User selects an alternative | FEAT-04.SPEC-010 confirms the suggestion (success) or fails |
| Pending | "Your suggestion" card shown with status "Waiting for Maya" | Suggestion submitted successfully, or screen opens with an existing open suggestion for this slot | Maya accepts or declines it, or it lapses |
| Resolved | Outcome banner shown above the Current Meal card | Screen opens after Sam's suggestion for this slot was accepted, declined, or lapsed since he last saw it | Sam dismisses the banner |
| Error (alternatives fetch) | Banner "Couldn't load alternatives. Check your connection and try again." with a Retry button; Current Meal card unchanged | Alternatives request fails | User taps Retry, or navigates away |
| Error (submission failed) | Banner "Couldn't send your suggestion. Try again." with a Retry button on the previously selected item | FEAT-04.SPEC-010 reports failure to create the suggestion | User retries the same selection, picks a different alternative, or navigates away |
| Offline/Degraded | Banner "You're offline -- suggesting a swap needs a connection." at the top; the Suggest a swap button is disabled | Connectivity lost while this screen is open | Connectivity restored -- the button re-enables automatically |

## Validation Rules

Validation of which recipes qualify as alternatives is governed by FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation). The one-open-suggestion-per-member-per-slot rule and submission eligibility are governed by FEAT-04.SPEC-010 (Suggestion Lifecycle Rules). This screen applies no independent field-level validation.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | Weekly Plan | FEAT-03 or FEAT-23 |
| Successful submission | Weekly Plan, showing the slot marked with a pending-suggestion indicator | FEAT-03 or FEAT-23 |
| Cancel from Error state (back arrow) | Weekly Plan, unchanged | FEAT-03 or FEAT-23 |

## Data Model

**Creates:** Swap Suggestion -- suggesting_member (Sam), night (the target slot), proposed_recipe (the chosen alternative), outcome (set to Suggested). Written by FEAT-04.SPEC-010.
**Reads:** Planned Meal -- recipe, cook_time, safety_badge (to show the current meal); Weekly Plan -- to locate the target slot; Swap Suggestion -- Sam's own prior suggestion for this slot, if any (Own-only read, per Access Matrix).
**Updates:** None directly by this screen.
**Deletes:** None -- suggestions are never withdrawn (Non-Goals, above).

## Business Rules

- Every alternative shown has already passed the same allergy/religious hard-rule check as original plan generation (XBR-01), enforced by FEAT-04.SPEC-008.
- Sam may hold at most one open suggestion per slot (FEAT-04.SPEC-010); submitting a new one while another is open is not offered here since the screen shows the "Your suggestion" card instead of the alternatives flow whenever one is open.
- A suggestion not answered before its night passes lapses automatically (FEAT-04.SPEC-005) and Sam is told (FEAT-04.SPEC-006).
- XBR-06: Sam suggests; the organiser accepts or declines each with one tap.

## Edge Cases

- **Sam navigates away while alternatives are loading** -- The in-flight request is discarded; re-opening the screen starts fresh.
- **Sam taps an alternative twice rapidly** -- The second tap is ignored while the first is Submitting.
- **Network failure while submitting** -- Error (submission failed) state; no suggestion is created.
- **Maya accepts or declines the suggestion while Sam is viewing this screen (concurrent-edit conflict)** -- The screen updates live from Pending to Resolved without requiring Sam to reload, since the underlying entity is the Swap Suggestion Sam owns; no conflicting write can occur from this screen since Sam performs no update here after submission. Resolution: the outcome is authoritative from Maya's action per the dependency map's Contention note for Swap Suggestion (first-decision-wins with reject-with-refresh), and this screen simply reflects it.
- **Sam's suggested recipe becomes unsafe after submission (a hard rule is tightened mid-week)** -- The Pending card is unaffected; the re-check runs when Maya reviews the suggestion (FEAT-04.SPEC-003), which re-verifies safety before any acceptance completes (FEAT-04.SPEC-004).
- **Slot has no qualifying alternatives at all** -- The scarcity explanation from FEAT-04.SPEC-008 is shown alone, with no selectable items.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation) | References (outbound) | Supplies the filtered alternatives list and any scarcity explanation |
| FEAT-04.SPEC-007 (Swap Alternatives Generation) | References (outbound, indirect via SPEC-008) | Sourced for AI-originated plans |
| FEAT-04.SPEC-010 (Suggestion Lifecycle Rules) | Triggers (outbound) | Creates the Swap Suggestion and enforces the one-open-per-member-per-slot rule |
| FEAT-04.SPEC-003 (Review Swap Suggestions) | References (outbound) | Where Maya sees and decides on the suggestion this screen creates |
| FEAT-04.SPEC-006 (Swap Suggestion Notifications) | References (outbound, indirect via SPEC-010) | Notifies Maya once submitted, and Sam once resolved |
| FEAT-03 (Weekly Plan) / FEAT-23 (Manual Weekly Planning) | Navigation (inbound/outbound) | Entry point and return destination |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| swap_suggested | slot night, entry source | Suggestion is successfully created | supports success-metrics.md: "Weekly Planning Time" |
| meal_swap_alternatives_limited | candidate count | Alternatives return with a scarcity explanation | supports success-metrics.md: "One-Tap Swap Completion" |
| meal_swap_failed | failure reason (fetch failed / submit failed) | Error state is shown | supports success-metrics.md: "One-Tap Swap Completion" |

## Acceptance Criteria

**FEAT-04.SPEC-002-AC-01:** Given Sam is viewing Friday's dinner on the Weekly Plan, when he taps "swap," then this screen opens showing the current meal and a "Suggest a swap" button.

**FEAT-04.SPEC-002-AC-02:** Given Sam is on this screen, when he taps "Suggest a swap," then the screen enters Loading Alternatives and an inline indicator appears.

**FEAT-04.SPEC-002-AC-03:** Given alternatives have loaded, when Sam taps one, then the suggestion is submitted, and the screen shows "Your suggestion" with the status "Waiting for Maya."

**FEAT-04.SPEC-002-AC-04:** Given Sam already has an open suggestion for this slot, when he opens this screen, then he sees the "Your suggestion" card directly, with no "Suggest a swap" button shown.

**FEAT-04.SPEC-002-AC-05:** Given Maya accepted Sam's suggestion since he last viewed it, when Sam opens this screen, then a banner reads "Maya accepted your suggestion."

**FEAT-04.SPEC-002-AC-06:** Given Maya declined Sam's suggestion since he last viewed it, when Sam opens this screen, then a banner reads "Maya declined your suggestion."

**FEAT-04.SPEC-002-AC-07:** Given Sam's suggestion lapsed because its night passed unanswered, when Sam opens this screen, then a banner reads "Your suggestion lapsed -- the night passed unanswered."

**FEAT-04.SPEC-002-AC-08:** Given Sam taps "Suggest a swap" and the alternatives request fails, when the failure occurs, then the banner "Couldn't load alternatives. Check your connection and try again." appears with a Retry button.

**FEAT-04.SPEC-002-AC-09:** Given Sam selects an alternative and submission fails, when the failure occurs, then the banner "Couldn't send your suggestion. Try again." appears and no suggestion is created.

**FEAT-04.SPEC-002-AC-10:** Given zero alternatives qualify for the slot, when Sam taps "Suggest a swap," then the scarcity explanation appears alone with no selectable items.

**FEAT-04.SPEC-002-AC-11:** Given Sam loses connectivity while on this screen, when he looks at the "Suggest a swap" button, then it is disabled with the banner "You're offline -- suggesting a swap needs a connection."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 9 (default, loading, alternatives shown, submitting, pending, resolved, error-fetch, error-submit, offline) | 9 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
