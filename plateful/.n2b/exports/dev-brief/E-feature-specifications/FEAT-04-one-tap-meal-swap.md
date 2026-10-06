# FEAT-04 — One-Tap Meal Swap

This chapter covers FEAT-04, One-Tap Meal Swap, a Core-tier feature. It contains 10 specifications carrying 114 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-04.SPEC-001 | Meal Swap (Direct) | screen | 12 |
| FEAT-04.SPEC-002 | Suggest a Swap | screen | 11 |
| FEAT-04.SPEC-003 | Review Swap Suggestions | screen | 10 |
| FEAT-04.SPEC-004 | Apply Meal Swap | automation | 10 |
| FEAT-04.SPEC-005 | Suggestion Lapse | automation | 8 |
| FEAT-04.SPEC-006 | Swap Suggestion Notifications | notification | 12 |
| FEAT-04.SPEC-007 | Swap Alternatives Generation | integration | 11 |
| FEAT-04.SPEC-008 | Alternatives Computation & Scarcity Explanation | logic-rule | 14 |
| FEAT-04.SPEC-009 | Swap Concurrency Lock | logic-rule | 11 |
| FEAT-04.SPEC-010 | Suggestion Lifecycle Rules | logic-rule | 15 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: One-Tap Meal Swap

## Summary

**Feature:** One-Tap Meal Swap
**ID:** FEAT-04
**Description:** Any dinner in the plan can be replaced with a different one in a single tap, and the grocery list updates immediately to match. The organiser swaps directly; other adult members suggest a swap, which the organiser accepts or declines with one tap.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** The brief calls this out explicitly: "Any meal can be swapped with one tap, and the grocery list updates instantly" (BRIEF.md, Vision, The Experience). Plans that cannot flex to real life are abandoned; this keeps the plan usable when reality changes.

**Key Capabilities:**
- Swap a meal — Household member replaces one night's dinner with an alternative in one tap
- See safe alternatives only — Every offered alternative has already passed the allergy safety check
- Instant list update — The grocery list adjusts automatically to reflect the swap
- Suggest a swap — An other adult member picks a safe alternative and sends it to the organiser as a suggestion
- Review suggestions — The organiser sees pending suggestions on the plan and accepts or declines each with one tap

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-04.SPEC-001 | Meal Swap (Direct) | Screen | Maya | Organiser taps swap on a planned meal, sees safe alternatives (or a scarcity explanation), and picks one to update the slot immediately |
| FEAT-04.SPEC-002 | Suggest a Swap | Screen | Sam | Other adult member picks a safe alternative for a night and sends it to the organiser as a suggestion |
| FEAT-04.SPEC-003 | Review Swap Suggestions | Screen | Maya | Organiser sees pending suggestions on the plan and accepts or declines each with one tap |
| FEAT-04.SPEC-004 | Apply Meal Swap | Automation | Maya, Sam | Shared core action that re-verifies safety, writes the new recipe onto the plan slot, closes any superseded suggestion, and signals the grocery list to recalculate |
| FEAT-04.SPEC-005 | Suggestion Lapse | Automation | Maya, Sam | Time-based check that marks an unanswered suggestion Lapsed when its night passes and frees the slot |
| FEAT-04.SPEC-006 | Swap Suggestion Notifications | Notification | Maya, Sam | Notifies the organiser when a suggestion arrives, and the suggester when it is accepted, declined, or lapses |
| FEAT-04.SPEC-007 | Swap Alternatives Generation | Integration | Maya, Sam | Category: AI text/plan generation — produces the scoped, one-slot alternatives set for AI-originated weekly plans |
| FEAT-04.SPEC-008 | Alternatives Computation & Scarcity Explanation | Logic/Rule | Maya, Sam | Routes alternative sourcing by plan origin, applies the safety/schedule filter, and determines the limited- or zero-alternatives explanation |
| FEAT-04.SPEC-009 | Swap Concurrency Lock | Logic/Rule | Maya, Sam | Enforces at most one active swap operation per meal slot at a time, preventing duplicate swaps from a double tap or a race with an accepted suggestion |
| FEAT-04.SPEC-010 | Suggestion Lifecycle Rules | Logic/Rule | Maya, Sam | One open suggestion per member per slot; a direct swap or an accepted suggestion supersedes any other open suggestion for that slot; a late accept on a lapsed suggestion is refused |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Swap a meal | FEAT-04.SPEC-001 | Direct swap screen: Maya taps swap, picks a safe alternative, and the slot updates immediately | Phase 2 (Explicit) |
| See safe alternatives only | FEAT-04.SPEC-001, FEAT-04.SPEC-008 | Every candidate is filtered by the safety/schedule rule before it ever reaches the alternatives list | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |
| Instant list update | FEAT-04.SPEC-004 | Apply Meal Swap signals the Shared Grocery List (FEAT-06) to recalculate the moment a swap completes | Phase 2 (Explicit) — execution is a Cross-Feature Touchpoint |
| Suggest a swap | FEAT-04.SPEC-002 | Sam picks a safe alternative for a night and sends it to Maya as a suggestion | Phase 2 (Explicit) |
| Review suggestions | FEAT-04.SPEC-003 | Maya sees pending suggestions on the plan and accepts or declines each with one tap | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-04.SPEC-004 | Apply Meal Swap | Phase 4 (Trigger-Response) | Both the direct-swap tap and an accepted suggestion end in the same state change on the Planned Meal; factoring it out avoids two divergent write paths and gives the safety re-check one home |
| FEAT-04.SPEC-005 | Suggestion Lapse | Phase 4 (Trigger-Response, time-based triggers) | Primary Flows & Alternates states a suggestion "lapses quietly" when its night passes; a time-based check is required to make that happen and to close the loop for the suggester |
| FEAT-04.SPEC-006 | Swap Suggestion Notifications | Phase 4 (Notification surfacing) | The Communications field names three distinct messages (suggestion arrived, outcome told to the suggester, lapse told to the suggester) with real audiences and delivery rules — none is a bare same-screen toast |
| FEAT-04.SPEC-007 | Swap Alternatives Generation | Phase 4 (External Dependencies lens) | The Dependencies slice of assumptions-constraints.md (ASMP-30) names an AI text/plan-generation capability this feature relies on for AI-originated plans, and the dependency map's External Touchpoints table lists FEAT-04 as needing its own Integration spec for it |
| FEAT-04.SPEC-008 | Alternatives Computation & Scarcity Explanation | Phase 5 (Rule Discovery) | The alternatives source is conditional on Weekly Plan origin (AI-generated vs. manually-built, per SC-16's free-tier zero-AI-cost rule) and the "no good alternative available" flow has non-trivial derivation logic — both exceed the inline-validation threshold |
| FEAT-04.SPEC-009 | Swap Concurrency Lock | Phase 5 (Rule Discovery) | Validation & Limits states "no more than one active swap operation per meal slot at a time"; this is a cross-path rule (direct swap vs. accepted suggestion can race) shared by two specs, which crosses the standalone-spec threshold |
| FEAT-04.SPEC-010 | Suggestion Lifecycle Rules | Phase 5 (Rule Discovery) / Phase 7 (gap resolution) | The one-per-member-per-slot limit, the direct-swap-supersedes-suggestion rule (dependency map, Swap Suggestion Contention), and the undefined case of two open suggestions on one slot (resolved here by parity with the stated supersede rule) form one connected rule set shared across three specs |

## Entity-Lifecycle Coverage Matrix

**Entity: Swap Suggestion**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-04.SPEC-002 | Suggest a Swap screen — Sam picks an alternative and the suggestion is recorded | FEAT-23 creates the equivalent "pick suggestion" for manual planning (cross-feature) |
| Read (single) | FEAT-04.SPEC-003 | Review Swap Suggestions — organiser opens one pending suggestion's detail (member, night, proposed recipe) | -- |
| Read (list) | FEAT-04.SPEC-003 | Review Swap Suggestions — lists all pending suggestions on the plan | SPEC-002 also reads back the status of Sam's own submitted suggestion (Own-only) |
| Update | FEAT-04.SPEC-003, FEAT-04.SPEC-005 | Accept/decline action (SPEC-003) or the lapse check (SPEC-005) writes the outcome field | -- |
| Delete/Archive | N/A | No delete: a suggestion always ends in a terminal outcome (Accepted, Declined, or Lapsed) and is retained as part of the plan's permanent history rather than removed, consistent with weekly-plan history being kept for the life of the account (scope-boundaries.md SC-18) — recorded as an explicit non-goal, not an omission | -- |
| State Transition | FEAT-04.SPEC-003, FEAT-04.SPEC-005 | Suggested -> Accepted or Declined (SPEC-003); Suggested -> Lapsed (SPEC-005); Accepted additionally drives SPEC-004 | -- |

**Entity: Planned Meal**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | Owned by FEAT-03 (AI generation) and FEAT-23 (manual pick); this feature only ever acts on an existing slot | -- |
| Read (single) | FEAT-04.SPEC-001 | Meal Swap (Direct) displays the current recipe, cook time, and safety badge of the meal being swapped | -- |
| Read (list) | N/A | The full week's list of Planned Meals is the Weekly Plan screen's responsibility (FEAT-03/FEAT-23); this feature reads only the subset of slots carrying a pending suggestion (SPEC-003) | -- |
| Update | FEAT-04.SPEC-004 | Apply Meal Swap writes the new recipe, sets status to Swapped, and appends to swap_history | Invoked from both SPEC-001 (direct) and SPEC-003's accept action |
| Delete/Archive | N/A | This feature never removes a slot; removal on a safety concern is owned by FEAT-02 (XBR-08) and clearing a night is owned by FEAT-23 | -- |
| State Transition | FEAT-04.SPEC-004 | status transitions to Swapped | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Weekly Plan | FEAT-04.SPEC-001, FEAT-04.SPEC-003 | Locates the target slot and its approval state; this feature never changes a plan-level field directly — only the Planned Meal it contains |
| Recipe | FEAT-04.SPEC-001, FEAT-04.SPEC-002, FEAT-04.SPEC-007, FEAT-04.SPEC-008 | Candidate alternatives and their cook time, cost, and safety badge |
| Dietary Rule | FEAT-04.SPEC-008 (via FEAT-02's safety check) | The safety/schedule filter every alternative must pass; this feature never stores or displays raw Dietary Rule records itself |
| Grocery List | -- (not read directly) | Updated indirectly by FEAT-06 in response to SPEC-004's trigger |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Maya taps swap on a planned meal | Fetch safe alternatives scoped to the slot | Standalone Logic/Rule (routing) + Standalone Integration (AI-originated plans) | FEAT-04.SPEC-008 / FEAT-04.SPEC-007 |
| Sam taps swap to suggest one | Fetch safe alternatives scoped to the slot | Standalone Logic/Rule + Standalone Integration (same as above) | FEAT-04.SPEC-008 / FEAT-04.SPEC-007 |
| Maya selects an alternative (direct swap) | Apply the swap to the plan slot | Standalone Automation | FEAT-04.SPEC-004 |
| Swap applied to a slot | Close/supersede any other open suggestion for that slot | Standalone Logic/Rule | FEAT-04.SPEC-010 |
| Swap applied to a slot | Signal the grocery list to recalculate | Cross-feature (FEAT-06 executes) | FEAT-06 responsibility |
| Second swap attempt on the same slot while one is in flight | Reject the second attempt; original stays visible | Standalone Logic/Rule | FEAT-04.SPEC-009 |
| Sam submits a swap suggestion | Notify Maya a suggestion is waiting | Standalone Notification | FEAT-04.SPEC-006 |
| Maya accepts a suggestion | Apply the swap (as above), then notify Sam it was accepted | Standalone Automation + Standalone Notification | FEAT-04.SPEC-004 / FEAT-04.SPEC-006 |
| Maya declines a suggestion | Set outcome to Declined; notify Sam it was declined | Inline in triggering screen (state write) + Standalone Notification | FEAT-04.SPEC-003 / FEAT-04.SPEC-006 |
| A suggestion's night passes unanswered | Mark it Lapsed, free the slot, notify the suggester | Standalone Automation + Standalone Notification | FEAT-04.SPEC-005 / FEAT-04.SPEC-006 |
| A same-day swap follows an already-sent nightly nudge | Trigger one correction notification naming the new dinner | Cross-feature (FEAT-13 owns the nudge and its correction, XBR-09) | FEAT-13 responsibility |
| A household dietary rule is tightened mid-week and a meal now fails | The now-unsafe slot opens for swap with safe alternatives offered | Cross-feature inbound (FEAT-02/FEAT-01 trigger, XBR-02) | FEAT-04.SPEC-001 / FEAT-04.SPEC-008 receive it |
| A safety-concern report removes a meal from the plan | The vacated slot opens for swap with safe alternatives offered | Cross-feature inbound (FEAT-02 triggers, XBR-08) | FEAT-04.SPEC-001 / FEAT-04.SPEC-008 receive it |
| Swap fails to save (e.g., dropped connection) | Original meal remains in place; retry offered | Inline in triggering screen | FEAT-04.SPEC-001 |

## Shared Context

**Shared Entities:**
- Swap Suggestion -- created by SPEC-002, read and updated by SPEC-003, updated by SPEC-005, read by SPEC-004 when applying an accepted one. Fields: suggesting_member, night, proposed_recipe, outcome.
- Planned Meal -- read by SPEC-001 and SPEC-003, updated by SPEC-004 (recipe, status, swap_history). Not created or deleted by this feature.

**Shared UI Patterns:**
- Alternatives list -- used by both SPEC-001 (direct swap) and SPEC-002 (suggest a swap): the same short list of safety-checked candidates, the same inline loading indicator, and the same scarcity-explanation pattern when few or zero options qualify. Spec Writers for both screens should describe this pattern identically.
- One-tap accept/decline -- used by SPEC-003 for each pending suggestion: a single tap per action, no confirmation dialog (mirrors the product's "one tap" promise).

**Shared Validation/Logic:**
- FEAT-04.SPEC-008 defines how alternatives are sourced and filtered; SPEC-001 and SPEC-002 both reference it rather than duplicating the routing or scarcity-explanation logic.
- FEAT-04.SPEC-009 defines the concurrency lock; SPEC-001 and SPEC-004 both reference it before writing to a slot.
- FEAT-04.SPEC-010 defines suggestion-lifecycle behavior; SPEC-002, SPEC-003, SPEC-004, and SPEC-005 all reference it rather than each re-deriving the one-per-member-per-slot and supersede rules.

## Internal Dependency Map

```
SPEC-001 (Meal Swap Direct) -> [Maya taps swap] -> SPEC-008 (Alternatives Computation) -> [routes by plan origin] -> SPEC-007 (Swap Alternatives Generation, AI-originated) or a direct recipe-library filter (manually-built plans)
SPEC-008 -> [returns filtered list] -> SPEC-001
SPEC-001 -> [Maya picks an alternative] -> SPEC-009 (Swap Concurrency Lock) -> [lock acquired] -> SPEC-004 (Apply Meal Swap)
SPEC-004 -> [writes new recipe to slot] -> SPEC-010 (Suggestion Lifecycle Rules) [supersedes any open suggestion for the slot]
SPEC-004 -> [swap applied] -> FEAT-06 (Shared Grocery List recalculation, cross-feature)
SPEC-002 (Suggest a Swap) -> [Sam taps swap] -> SPEC-008 -> [returns filtered list] -> SPEC-002
SPEC-002 -> [Sam picks an alternative] -> SPEC-010 [creates the Swap Suggestion, enforces one-open-per-member-per-slot] -> SPEC-006 (Swap Suggestion Notifications) [notify Maya]
SPEC-003 (Review Swap Suggestions) -> [Maya taps accept] -> SPEC-009 -> SPEC-004 -> SPEC-006 [notify Sam: accepted]
SPEC-003 -> [Maya taps decline] -> SPEC-010 [set outcome Declined] -> SPEC-006 [notify Sam: declined]
SPEC-005 (Suggestion Lapse) -> [the suggestion's night passes unanswered] -> SPEC-010 [set outcome Lapsed] -> SPEC-006 [notify Sam: lapsed]
```

**Default Entry:** FEAT-04.SPEC-001 (Meal Swap Direct) -- reached by Maya from the Weekly Plan screen (owned by FEAT-03/FEAT-23) when she taps swap on a planned meal. FEAT-04.SPEC-003 (Review Swap Suggestions) is her entry point when she arrives via a pending-suggestion notification. FEAT-04.SPEC-002 (Suggest a Swap) is Sam's entry point, reached the same way as SPEC-001 but scoped to his Own-only access.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-04.SPEC-001 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Maya opens the alternatives list from the week plan screen | Tap swap on a planned meal (Sunday Plan Review, step 4) |
| FEAT-04.SPEC-003 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Maya reviews Sam's pending suggestion from the week plan screen | Review suggestion and accept (Sunday Plan Review, step 3) |
| FEAT-04.SPEC-001 | Inbound | FEAT-23 (Manual Weekly Planning) | Maya swaps a meal she picked manually | Tap swap on a manually-picked night |
| FEAT-04.SPEC-003 | Inbound | FEAT-23 (Manual Weekly Planning) | Maya reviews Sam's suggested pick through the same accept/decline flow | Accept Sam's suggested pick (Free-Tier Manual Week, step 3) |
| FEAT-04.SPEC-001 | Inbound | FEAT-02 (Dietary Rules & Allergy Safety Engine) | A safety-concern acknowledgement opens the vacated slot for swap | Pick a replacement for the removed meal (Allergy-Safe Swap Recovery, step 5) |
| FEAT-04.SPEC-008 | Outbound | FEAT-08 (Recipe Library) | Explains why a specific recipe isn't offered as an alternative when searched directly | Household member searches the library for a recipe not offered (Allergy-Safe Swap Recovery, failure variant) |
| FEAT-04.SPEC-004 | Outbound | FEAT-06 (Shared Grocery List) | Every completed swap or accepted suggestion recalculates the grocery list immediately | Swap or suggestion-acceptance completes (XBR-03) |
| FEAT-04.SPEC-004 | Outbound | FEAT-13 (Tonight's Dinner Reminder) | A same-day swap after the nudge triggers one correction notification | Swap completes same-day, after the 5pm nudge was already sent (XBR-09) |
| FEAT-04.SPEC-001, FEAT-04.SPEC-008 | Inbound | FEAT-01 (Household Setup) / FEAT-02 (Dietary Rules & Allergy Safety Engine) | A tightened dietary rule opens any now-failing meal for swap | Mid-week hard-rule change re-check flags a meal (XBR-02) |
| FEAT-04.SPEC-006 | Outbound (dependency) | FEAT-07 (Notification Prefs) | Every suggestion/outcome notification relies on the device-notification-delivery capability owned there | Every SPEC-006 send |
| FEAT-04.SPEC-004 | Outbound (dependency) | FEAT-06 (Shared Grocery List) | Live propagation of the swapped plan and list across household devices relies on the real-time synchronization capability owned there | Every swap completion |

## Non-Functional Notes

**Data volumes / growth:** Swap operations occur at most a few times per household per week, alongside one weekly plan generation, keeping AI usage within the founder's sub-$100/month pre-revenue infrastructure budget (scope-boundaries.md SC-16). Swap Suggestion records accumulate at the same modest per-household weekly rate and are retained for the life of the account (scope-boundaries.md SC-18).

**Responsiveness:** A swap must go from tap to an updated plan and list in under 10 seconds, in a single interaction (success-metrics.md, One-Tap Swap Completion). Alternatives appear within a couple of seconds with a brief inline loading indicator (product-features.md, States). The resulting grocery list change must itself feel instant (assumptions-constraints.md ASMP-22).

**Data sensitivity / privacy:** Swap Suggestion holds no personal data beyond a member reference and a recipe choice (feature-dependency-map.md, Swap Suggestion Data Sensitivity: None). The alternatives shown are filtered through Dietary Rule data — health-adjacent personal data including children's allergy information, the product's most sensitive data class — but FEAT-04 only ever reads that data through FEAT-02's safety check; it neither stores nor displays raw Dietary Rule records itself (feature-dependency-map.md, Dietary Rule Data Sensitivity).

**Compliance flags:** No medical or diet advice is derived from swap alternatives — the safety check (XBR-01) only allows or excludes options, never recommends what to eat for health reasons (scope-boundaries.md SC-06). Children's-privacy-class handling governs the Dietary Rule data this feature depends on indirectly (assumptions-constraints.md ASMP-27), though FEAT-04 itself processes no children's data directly.

## Non-Goals

- **Other adult members swapping directly** -- Excluded per scope-boundaries.md SC-04: the brief gives other adult members "suggest swaps" only; direct swap authority is Maya's alone, with suggest-and-approve as the only path for Sam.
- **In-product messaging between household members about a suggestion** -- Excluded per scope-boundaries.md SC-14: coordination runs through the suggestion's structured notify/accept/decline/lapse flow (FEAT-04.SPEC-006), not a chat layer.
- **Withdrawing a submitted swap suggestion** -- Not modeled: product-features.md's Primary Flows & Alternates and Validation & Limits fields define a suggestion's only outcomes as accepted, declined, or lapsed; no withdrawal path is stated, so a member who wants to change a suggestion waits for a decision or its lapse.
- **Medical or diet advice in alternative selection** -- Excluded per scope-boundaries.md SC-06: alternatives are filtered for safety only, never ranked or explained in nutritional or health terms.
- **Automatic purge of swap suggestion history** -- Intentional lifecycle decision surfaced by the CRUD matrix: suggestion outcomes are retained as part of the weekly plan's permanent history rather than purged, consistent with scope-boundaries.md SC-18's account-life retention for plan history.
- **Grocery-list recalculation logic itself** -- Owned by FEAT-06 (Shared Grocery List) per the feature-dependency-map.md authority column and XBR-03; this feature only triggers the recalculation.
- **A distinct native-app swap experience** -- Excluded per scope-boundaries.md SC-05: the product ships as a responsive web app for v1, with no native apps.



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



# Automation Spec: Apply Meal Swap

## Overview

**Name:** Apply Meal Swap
**ID:** FEAT-04.SPEC-004
**Type:** Automation
**Purpose:** Shared core action that re-verifies safety, writes the new recipe onto the plan slot, closes any superseded suggestion, and signals the grocery list to recalculate.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap

## Scope and Non-Goals

**In Scope:**
- The single write path that moves a Planned Meal from its current recipe to a new one, whether initiated by a direct swap or an accepted suggestion
- Re-verifying the chosen recipe's safety immediately before writing (never trusting a safety check performed earlier)
- Superseding any other open suggestion on the same slot
- Signaling the Shared Grocery List (FEAT-06) to recalculate

**Non-Goals:**
- Computing which recipes qualify as alternatives -- owned by FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation); this automation only writes the recipe it is given
- Acquiring the concurrency lock -- owned by FEAT-04.SPEC-009 (Swap Concurrency Lock), which the triggering screen invokes before calling this automation
- Marking a suggestion Declined or Lapsed -- those outcomes are set by FEAT-04.SPEC-003 (a direct decline) and FEAT-04.SPEC-005 (lapse) respectively, not by this automation
- Recalculating the grocery list's contents -- owned by FEAT-06 (Shared Grocery List) per the feature-dependency-map.md authority column and XBR-03; this automation only triggers that recalculation

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Maya selects an alternative on a direct swap | FEAT-04.SPEC-001 (Meal Swap Direct) | Fires after the concurrency lock (FEAT-04.SPEC-009) is acquired for the slot | Target Planned Meal reference, chosen recipe, initiating member (Maya) |
| Maya accepts a pending suggestion | FEAT-04.SPEC-003 (Review Swap Suggestions) | Fires after the concurrency lock is acquired for the slot | Target Planned Meal reference, the suggestion's proposed_recipe, the Swap Suggestion reference, initiating member (Maya) |

## Processing Logic

1. Receive the target Planned Meal reference and the chosen recipe (from either trigger path).
2. Re-run the same allergy/religious hard-rule safety check the recipe would need to pass on original plan generation (XBR-01), scoped to the household's current dietary rules -- never reusing a safety result computed earlier in the flow.
3. If the safety re-check fails, stop processing and report the Safety Re-check Failed outcome (no write occurs).
4. If the safety re-check passes, write the chosen recipe onto the Planned Meal's recipe field, set its status to Swapped, and append the previous recipe to its swap_history.
5. If the trigger path was an accepted suggestion, set that Swap Suggestion's outcome to Accepted (via FEAT-04.SPEC-010).
6. Check for any other open Swap Suggestion on the same slot (from a different member, or a stale one from the same member) and supersede it via FEAT-04.SPEC-010's superseding rule.
7. Release the concurrency lock on the slot (FEAT-04.SPEC-009).
8. Signal the Shared Grocery List (FEAT-06) that the plan changed, so it recalculates.
9. If the initiating action was an accepted suggestion, signal FEAT-04.SPEC-006 to notify the suggesting member that it was accepted.
10. Return success to the triggering screen.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Swap applied (direct) | Safety re-check passes, triggered from FEAT-04.SPEC-001 | Planned Meal recipe, status, swap_history updated; any other open suggestion on the slot superseded (FEAT-04.SPEC-010); grocery list signaled | Weekly Plan shows the new dinner in the slot immediately | FEAT-04.SPEC-001, FEAT-06, FEAT-11.SPEC-004, FEAT-13.SPEC-003, FEAT-21.SPEC-003 |
| Swap applied (accepted suggestion) | Safety re-check passes, triggered from FEAT-04.SPEC-003 | Same as above, plus the accepted Swap Suggestion's outcome set to Accepted | The suggestion card is removed from FEAT-04.SPEC-003's list; the suggesting member is notified their suggestion was accepted (FEAT-04.SPEC-006); Weekly Plan shows the new dinner | FEAT-04.SPEC-003, FEAT-04.SPEC-010, FEAT-04.SPEC-006, FEAT-06, FEAT-11.SPEC-004, FEAT-13.SPEC-003, FEAT-21.SPEC-003 |
| Safety Re-check Failed | The chosen recipe no longer passes the household's current hard dietary rules at write time | No data changes -- the Planned Meal and any suggestion are left exactly as they were | From a direct swap: FEAT-04.SPEC-001 shows "This meal changed while you were choosing. Here's the latest." and the slot is unchanged. From an accepted suggestion: FEAT-04.SPEC-003 shows "This suggestion no longer passes the household's dietary rules and can't be applied." on that card | FEAT-04.SPEC-001, FEAT-04.SPEC-003 |
| Write Failure | The safety re-check passes but the underlying write does not complete (e.g., a dropped connection) | No partial state -- the Planned Meal remains at its prior recipe and status; the lock is released so a retry can proceed | From a direct swap: "Couldn't complete the swap. Your original dinner is still on the plan." with a retry option. From an accepted suggestion: "Couldn't complete this action. Try again." on that card | FEAT-04.SPEC-001, FEAT-04.SPEC-003 |

## Data Model

**Reads:** Planned Meal -- current recipe, status (to confirm the slot is eligible to be written); Swap Suggestion -- proposed_recipe, outcome (when triggered by an accept); Dietary Rule -- read only through FEAT-02's safety-check capability, never stored or displayed directly by this automation (per the dependency map's Data Sensitivity note for FEAT-04).
**Creates:** None.
**Updates:** Planned Meal -- recipe, status (set to Swapped), swap_history (previous recipe appended); Swap Suggestion -- outcome (set to Accepted, when triggered by an accept) and, for any other open suggestion on the slot, outcome (set to Declined via superseding, per FEAT-04.SPEC-010).
**Deletes:** None.

## Business Rules

- The safety re-check is mandatory and synchronous: the write never proceeds ahead of it, regardless of trigger path (XBR-01).
- This automation is the single write path onto Planned Meal.recipe for a swap -- both trigger sources converge here so the safety re-check has exactly one home (Brief, Analyst-Discovered Specs rationale).
- A direct swap by Maya always supersedes any open suggestion on the same slot, including one raised after the swap started but before it completes (FEAT-04.SPEC-010; dependency map, Swap Suggestion Contention).
- XBR-03: completing a swap or accepting a suggestion always signals the Shared Grocery List to recalculate immediately -- no swap completes without that signal firing.
- XBR-09: if a same-day swap completes after the nightly nudge (FEAT-13) has already been sent for that night, this automation's completion is the trigger for FEAT-13's one-time correction notification; this automation does not itself send that notification, only makes the completed swap visible for FEAT-13 to detect.

## Edge Cases

- **The safety re-check fails for a recipe that passed when the alternatives list was built** -- A hard rule was tightened in the interim (XBR-02); the swap does not apply and the triggering screen shows its Safety Re-check Failed message from the Outcome Definitions table above.
- **The target slot no longer exists (removed by a safety-concern report between selection and this automation firing)** -- The write is refused; the triggering screen is told "This meal changed while you were choosing. Here's the latest." and reloads the current slot state, consistent with the Planned Meal Contention note (a safety removal always wins over a concurrent change).
- **Concurrent trigger firing (a direct swap and an accepted suggestion for the same slot fire at effectively the same time)** -- The concurrency lock (FEAT-04.SPEC-009) ensures only one of the two triggers can proceed through this automation at a time; the second is rejected before this automation is even invoked, per FEAT-04.SPEC-009's lock semantics. This automation itself never receives two concurrent invocations for the same slot.
- **Trigger fires while a previous run for the same slot is still in flight** -- Cannot occur: the concurrency lock held by the in-flight run prevents a second invocation for the same slot from starting (FEAT-04.SPEC-009). Invocations for different slots proceed independently and never queue behind each other.
- **The Shared Grocery List signal is not acknowledged (FEAT-06 degraded)** -- The Planned Meal write still completes and is treated as successful; the grocery list recalculates when FEAT-06's own recovery behavior allows, per FEAT-06's degradation handling (not owned by this spec). The user is never told the swap failed because of this.
- **An accepted suggestion's slot was already changed by a prior direct swap moments earlier** -- Treated identically to the Safety Re-check Failed / target-slot-changed case: the accept is refused with the "no longer... can't be applied" message if the recipe fails re-check, or the general concurrent-change message if the slot itself changed underneath it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-001 (Meal Swap Direct) | Triggered by (inbound) | Direct swap selection fires this automation |
| FEAT-04.SPEC-003 (Review Swap Suggestions) | Triggered by (inbound) | Accepting a suggestion fires this automation |
| FEAT-04.SPEC-009 (Swap Concurrency Lock) | References (inbound) | Lock must be held before this automation runs, and is released when it completes |
| FEAT-04.SPEC-010 (Suggestion Lifecycle Rules) | Triggers (outbound) | Sets the accepted suggestion's outcome and supersedes any other open suggestion on the slot |
| FEAT-04.SPEC-006 (Swap Suggestion Notifications) | Triggers (outbound) | Notifies the suggesting member when their suggestion is accepted |
| FEAT-06 (Shared Grocery List) | Triggers (outbound) | Signaled to recalculate immediately on every successful swap |
| FEAT-13 (Tonight's Dinner Reminder) | Affects (outbound) | A same-day completion after the nightly nudge triggers FEAT-13's correction notification (XBR-09) |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | References (outbound) | Provides the safety re-check this automation always runs before writing |

## Analytics and Success Signals

- **meal_swap_completed** (trigger path: direct / accepted_suggestion; slot night; time from trigger to completion) -- supports success-metrics.md: "One-Tap Swap Completion"
- **meal_swap_failed** (trigger path; reason: safety_recheck_failed / write_failure) -- supports success-metrics.md: "One-Tap Swap Completion"
- **swap_suggestion_accepted** (slot night, suggesting member) -- supports success-metrics.md: "Weekly Planning Time"
- **allergy_safety_recheck_blocked_swap** (trigger path) -- supports success-metrics.md: "Zero Allergy Incidents" (confirms the safety re-check is genuinely enforced at the moment of every swap, not only at original plan generation)

## Acceptance Criteria

**FEAT-04.SPEC-004-AC-01:** Given Maya selects a safe alternative on FEAT-04.SPEC-001, when this automation fires, then the safety re-check passes, the Planned Meal's recipe updates, its status becomes Swapped, and the grocery list is signaled to recalculate.

**FEAT-04.SPEC-004-AC-02:** Given Maya accepts Sam's suggestion on FEAT-04.SPEC-003, when this automation fires, then the safety re-check passes, the Planned Meal updates, the suggestion's outcome is set to Accepted, and Sam is notified the suggestion was accepted.

**FEAT-04.SPEC-004-AC-03:** Given a household hard dietary rule was tightened after a suggestion was raised but before Maya accepts it, when this automation runs the safety re-check on accept, then the write is refused and FEAT-04.SPEC-003 shows "This suggestion no longer passes the household's dietary rules and can't be applied."

**FEAT-04.SPEC-004-AC-04:** Given the safety re-check passes but the write itself fails (e.g., a dropped connection), when this automation reports the failure, then no partial state is left on the Planned Meal and the triggering screen shows its retry message.

**FEAT-04.SPEC-004-AC-05:** Given a second open suggestion exists on the same slot as the one Maya just accepted, when this automation completes, then the other suggestion is superseded (its outcome set to Declined) via FEAT-04.SPEC-010.

**FEAT-04.SPEC-004-AC-06:** Given Maya applies a direct swap on a slot that also has a pending suggestion from Sam, when this automation completes, then Sam's pending suggestion is superseded.

**FEAT-04.SPEC-004-AC-07:** Given a swap completes on a night for which FEAT-13's nightly nudge was already sent today, when this automation reports success, then FEAT-13's same-day correction notification fires (XBR-09).

**FEAT-04.SPEC-004-AC-08:** Given a concurrency lock is already held for a slot, when a second trigger for the same slot would otherwise fire this automation, then the automation never runs a second time concurrently for that slot -- the second trigger is rejected before invocation, per FEAT-04.SPEC-009.

**FEAT-04.SPEC-004-AC-09:** Given the target slot was removed by a safety-concern report between selection and this automation firing, when this automation runs, then the write is refused and the triggering screen shows the concurrent-change message.

**FEAT-04.SPEC-004-AC-10:** Given the Shared Grocery List signal is not immediately acknowledged, when this automation completes the Planned Meal write, then the swap is still reported as successful to the user.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (direct swap, accepted suggestion) | 2 |
| Outcome Paths | 4 (applied-direct, applied-accepted, safety-recheck-failed, write-failure) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Suggestion Lapse

## Overview

**Name:** Suggestion Lapse
**ID:** FEAT-04.SPEC-005
**Type:** Automation
**Purpose:** Time-based check that marks an unanswered swap suggestion Lapsed when its night passes and frees the slot for a new suggestion.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap

## Scope and Non-Goals

**In Scope:**
- Detecting every Swap Suggestion whose target night has passed while its outcome is still Suggested
- Setting the outcome to Lapsed and freeing the slot for a new suggestion from the same member
- Signaling FEAT-04.SPEC-006 to notify the suggesting member of the lapse

**Non-Goals:**
- Accepting or declining a suggestion -- those are Maya's actions in FEAT-04.SPEC-003; this automation only handles the case where she never acted
- Notifying the organiser about a lapse -- product-features.md's Communications field states only the suggesting member is told when a suggestion lapses; the organiser is not separately notified since she took no action to reverse
- Any retroactive change to the Planned Meal -- a lapsed suggestion simply stops being pending; the slot's current recipe (whatever it was before the suggestion) is untouched, since a suggestion never wrote to the Planned Meal while it was open

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nightly lapse check | system (schedule-based) | Runs once daily, after the last moment a night's dinner could still be swapped or accepted (end of that calendar day, household local time) | All Swap Suggestion records with outcome = Suggested whose night is on or before the day that just ended |

## Processing Logic

1. At the scheduled run time, read every Swap Suggestion with outcome = Suggested.
2. For each, compare its night against the current date in the household's local time.
3. If the suggestion's night is on or before the day that has just ended, mark it as lapsed.
4. Set that suggestion's outcome to Lapsed via FEAT-04.SPEC-010.
5. Signal FEAT-04.SPEC-006 to notify the suggesting member that their suggestion lapsed.
6. Suggestions whose night has not yet passed are left untouched and re-evaluated on the next run.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Suggestion lapsed | The suggestion's night passed with no accept or decline recorded | Swap Suggestion outcome set to Lapsed | The suggesting member is notified their suggestion lapsed (FEAT-04.SPEC-006); the suggestion no longer appears in FEAT-04.SPEC-003's pending list; the suggesting member's screen (FEAT-04.SPEC-002) shows the lapsed outcome next time it opens | FEAT-04.SPEC-002, FEAT-04.SPEC-003, FEAT-04.SPEC-006, FEAT-04.SPEC-010 |
| No lapses found | No Suggested-outcome suggestion has a passed night at run time | None | No user-visible effect | None |

## Data Model

**Reads:** Swap Suggestion -- outcome, night (to find every pending suggestion whose night has passed).
**Creates:** None.
**Updates:** Swap Suggestion -- outcome set to Lapsed, via FEAT-04.SPEC-010.
**Deletes:** None -- lapsed suggestions are retained with a terminal outcome as part of the plan's permanent history (Brief, Entity-Lifecycle Coverage Matrix).

## Business Rules

- A suggestion "lapses quietly" once its night passes (product-features.md, Primary Flows & Alternates) -- there is no grace period and no reminder to Maya before the lapse.
- XBR-06: a suggestion not answered before its night lapses, and the suggester is told the outcome.
- Lapsing a suggestion frees its slot: the suggesting member may raise a new suggestion for a future occurrence of that slot without being blocked by the lapsed one (FEAT-04.SPEC-010's one-open-per-member-per-slot rule only counts Suggested-outcome suggestions).

## Edge Cases

- **Maya accepts or declines a suggestion in the same window this automation runs (race between a human action and the scheduled check)** -- First-decision-wins with reject-with-refresh, per the dependency map's Contention note for Swap Suggestion: whichever resolution (Maya's accept/decline, or this automation's lapse) writes the outcome first stands; the later attempt finds the suggestion already resolved and is a no-op. A late accept specifically is refused per FEAT-04.SPEC-010's late-accept rule.
- **Concurrent trigger firing (two scheduled runs somehow overlap)** -- Each suggestion's outcome write is idempotent: a suggestion already set to Lapsed by one run is a no-op for the other, and no duplicate lapse notification is sent (deduplication owned by FEAT-04.SPEC-006).
- **Trigger fires while a previous run is still in flight** -- The nightly check does not start a new run until the prior one completes; there is only ever one active run, so overlap cannot occur.
- **A suggestion's night was itself changed by a plan edit before the check runs** -- Not applicable: a Swap Suggestion's night is fixed to the slot it targets and is never edited independently of the suggestion itself (per the dependency map's Swap Suggestion field definitions); this scenario cannot arise.
- **The household's local time zone is ambiguous or changes (e.g., daylight saving transition) on the night in question** -- The check uses the household's configured locale settings (FEAT-16) to determine "end of day," consistent with how the household's schedule and nightly nudge are timed elsewhere in the product.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-010 (Suggestion Lifecycle Rules) | Triggers (outbound) | Sets the suggestion's outcome to Lapsed and frees the slot |
| FEAT-04.SPEC-006 (Swap Suggestion Notifications) | Triggers (outbound) | Notifies the suggesting member of the lapse |
| FEAT-04.SPEC-002 (Suggest a Swap) | Affects (outbound) | The suggester's screen reflects the lapsed outcome on next view |
| FEAT-04.SPEC-003 (Review Swap Suggestions) | Affects (outbound) | The suggestion is removed from the pending list |

## Analytics and Success Signals

- **swap_suggestion_lapsed** (slot night, suggesting member, days pending before lapse) -- supports success-metrics.md: "Weekly Planning Time" (a high lapse rate signals suggestions are not being reviewed within the organiser's planning window)

## Acceptance Criteria

**FEAT-04.SPEC-005-AC-01:** Given Sam's suggestion for Wednesday's dinner is still Suggested when Wednesday ends, when the nightly lapse check runs, then the suggestion's outcome is set to Lapsed.

**FEAT-04.SPEC-005-AC-02:** Given a suggestion has just lapsed, when this automation completes, then Sam receives the lapse notification (FEAT-04.SPEC-006).

**FEAT-04.SPEC-005-AC-03:** Given a suggestion lapsed for a slot, when Sam opens FEAT-04.SPEC-002 for that same slot in a future week, then he can raise a new suggestion without being blocked by the lapsed one.

**FEAT-04.SPEC-005-AC-04:** Given no suggestions have a passed night at run time, when the nightly lapse check runs, then no outcomes change and no notifications are sent.

**FEAT-04.SPEC-005-AC-05:** Given Maya accepts a suggestion in the same moment the nightly check would otherwise lapse it, when both attempt to resolve the suggestion, then whichever resolves first stands and the other is a no-op.

**FEAT-04.SPEC-005-AC-06:** Given the nightly lapse check somehow runs twice for the same night, when the second run processes an already-lapsed suggestion, then no duplicate lapse notification is sent.

**FEAT-04.SPEC-005-AC-07:** Given a suggestion's night has not yet passed, when the nightly lapse check runs, then that suggestion is left untouched and remains pending.

**FEAT-04.SPEC-005-AC-08:** Given the suggestion lapses, when Sam next opens FEAT-04.SPEC-002 for that slot, then he sees the banner "Your suggestion lapsed -- the night passed unanswered."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 2 (lapsed, no lapses found) | 2 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |



# Notification Spec: Swap Suggestion Notifications

## Overview

**Name:** Swap Suggestion Notifications
**ID:** FEAT-04.SPEC-006
**Type:** Notification
**Purpose:** Tells the organiser when a swap suggestion arrives, and tells the suggesting member the outcome once it is accepted, declined, or lapses.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap

## Scope and Non-Goals

**In Scope:**
- The "suggestion arrived" message to the organiser (Maya)
- The "suggestion accepted," "suggestion declined," and "suggestion lapsed" messages to the suggesting member (Sam)
- Delivery, preference, and expiry behavior for all four message variants

**Non-Goals:**
- Deciding when a suggestion lapses -- owned by FEAT-04.SPEC-005 (Suggestion Lapse); this spec begins where that automation's trigger fires
- Deciding when a swap is applied following an accept -- owned by FEAT-04.SPEC-004 (Apply Meal Swap); this spec only sends the resulting notification
- The nightly "tonight's dinner" nudge and its same-day swap correction -- excluded per feature-dependency-map.md: those belong to FEAT-13 (Tonight's Dinner Reminder), a distinct communication with its own trigger and content
- In-product messaging between household members about a suggestion -- excluded per scope-boundaries.md SC-14: coordination runs through this structured notify/accept/decline/lapse flow, not a chat layer

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, on every variant | Both Maya and Sam use the shared plan inside the product as their primary touchpoint, and the pending state is always visible there regardless of whether the outbound message is seen |
| Push (device notification) | Always attempted, when the recipient's device supports it, via the device-notification-delivery capability (FEAT-07.SPEC-005) | Maya's and Sam's day-to-day interaction happens in short phone sessions away from an open app (user-persona.md, Behavioral Context); a suggestion or its outcome needs to reach them promptly for the "one tap" promise to hold in practice |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Suggestion submitted | FEAT-04.SPEC-002 (Suggest a Swap), via FEAT-04.SPEC-010 | Fires when a new Swap Suggestion is created with outcome Suggested | Suggesting member name, night, proposed recipe |
| Suggestion accepted | FEAT-04.SPEC-004 (Apply Meal Swap) | Fires when an accepted suggestion's swap completes successfully | Suggesting member name, night, applied recipe |
| Suggestion declined | FEAT-04.SPEC-003 (Review Swap Suggestions) | Fires when Maya declines a pending suggestion | Suggesting member name, night, declined recipe |
| Suggestion lapsed | FEAT-04.SPEC-005 (Suggestion Lapse) | Fires when the nightly lapse check marks a suggestion Lapsed | Suggesting member name, night, lapsed recipe |

## Audience and Preferences

**Recipients:** Maya (Organiser) receives the "suggestion arrived" variant, since she alone holds accept/decline authority (Access Matrix, Meal Swap: Full). Sam (Other Adult Member) receives the "accepted," "declined," and "lapsed" variants, since he alone is the suggester whose suggestions can resolve this way (Access Matrix, Meal Swap: Own-only). No other role is entitled to either variant: kids have no Meal Swap access, and Riley's support view is read-only and never receives notifications.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- no dedicated preference exists for this notification | -- | Always on | -- |

This notification carries no independent on/off toggle: the Member Profile's notification_preferences field (per the Feature Dependency Map) defines only plan-ready and nightly-nudge preferences for adults; swap-suggestion messages are not among them. This is a deliberate scope decision, not an oversight -- product-features.md's Communications field describes these messages as the mechanism that carries the suggest-and-approve coordination the product replaces a group chat with (Problem Statement); turning them off would leave a submitted suggestion silently stranded with no other route to the recipient's attention, contradicting XBR-06's requirement that the suggester is always told the outcome.

**Quiet Hours:** N/A -- the product defines quiet hours nowhere in its notification model (no ASMP, XBR, or Communications field establishes one for any notification in Plateful); every notification, including this one, is delivered as soon as its trigger fires.

## Content Definition

**In-app -- Suggestion arrived (to Maya):**
- **Title:** Sam suggested a swap for {night}
- **Body:** {proposed_recipe_name} instead of tonight's plan
- **CTA:** Review -- deep-links to FEAT-04.SPEC-003 (Review Swap Suggestions) for the specific suggestion

**Push -- Suggestion arrived (to Maya):**
- **Title:** Swap suggestion waiting
- **Body:** Sam suggested {proposed_recipe_name} for {night}
- **CTA:** Opens FEAT-04.SPEC-003 (Review Swap Suggestions) for the specific suggestion

**In-app -- Suggestion accepted (to Sam):**
- **Title:** Your suggestion was accepted
- **Body:** {night}'s dinner is now {proposed_recipe_name}
- **CTA:** View plan -- deep-links to FEAT-04.SPEC-002 (Suggest a Swap) for that slot

**Push -- Suggestion accepted (to Sam):**
- **Title:** Maya accepted your swap
- **Body:** {night} is now {proposed_recipe_name}
- **CTA:** Opens FEAT-04.SPEC-002 (Suggest a Swap) for that slot

**In-app -- Suggestion declined (to Sam):**
- **Title:** Your suggestion was declined
- **Body:** {night}'s original dinner stays on the plan
- **CTA:** View plan -- deep-links to FEAT-04.SPEC-002 (Suggest a Swap) for that slot

**Push -- Suggestion declined (to Sam):**
- **Title:** Maya declined your swap
- **Body:** {night}'s dinner is unchanged
- **CTA:** Opens FEAT-04.SPEC-002 (Suggest a Swap) for that slot

**In-app -- Suggestion lapsed (to Sam):**
- **Title:** Your suggestion lapsed
- **Body:** {night} passed before Maya answered -- the original dinner stayed on the plan
- **CTA:** View plan -- deep-links to FEAT-04.SPEC-002 (Suggest a Swap) for that slot

**Push -- Suggestion lapsed (to Sam):**
- **Title:** Swap suggestion lapsed
- **Body:** {night} passed with no answer -- original dinner kept
- **CTA:** Opens FEAT-04.SPEC-002 (Suggest a Swap) for that slot

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {night} | Swap Suggestion -- night | Wednesday | Never empty -- night is required at suggestion creation |
| {proposed_recipe_name} | Swap Suggestion -- proposed_recipe (recipe name) | Sheet-pan salmon | Never empty -- proposed_recipe is required at suggestion creation |

## Delivery Rules

**Batching:** No batching -- each suggestion's arrival and each resolution is delivered as its own notification the moment its trigger fires, matching the "one tap" immediacy the product commits to for swap coordination. Multiple suggestions arriving close together each produce their own separate notification rather than a combined digest.
**Deduplication:** At most one notification per Swap Suggestion per trigger event. A suggestion produces exactly one "arrived" notification (on creation), and exactly one resolution notification (accepted, declined, or lapsed -- these are mutually exclusive terminal outcomes, so only one can ever fire per suggestion). A retried or re-run automation that reaches an already-notified state (e.g., FEAT-04.SPEC-005 processing an already-lapsed suggestion, per its own Edge Cases) does not re-send.
**Retry on failure:** Push delivery failure is retried up to 3 times over 30 minutes. After the final failure, the in-app notification stands as the delivery of record -- the recipient sees it the next time they open the product, and no separate failure message is shown to them.
**Expiry:** These notifications do not expire undelivered: unlike a time-sensitive reminder, the underlying state (a pending suggestion, or its resolved outcome) remains fully visible and actionable inside the product indefinitely, so a delayed push delivery still reaches a recipient with an accurate, current message when it eventually arrives.

## Edge Cases

- **The suggestion resolves (is accepted, declined, or superseded) before the "arrived" push notification is delivered** -- The "arrived" push is cancelled; delivering a stale "review this" message about an already-resolved suggestion would contradict the state Maya would see on opening the app. The in-app pending-suggestion indicator itself is unaffected since it already reflects current state when viewed live.
- **A suggestion is superseded by a direct swap on the same slot (FEAT-04.SPEC-010) rather than declined or accepted by Maya** -- This is treated as the "declined" variant from Sam's perspective content-wise, since the practical outcome (his suggestion did not take effect and the slot changed by another route) matches; the notification text is generated from the suggestion's final outcome value as set by the superseding rule.
- **Sam is removed from the household between suggestion creation and the notification firing** -- The notification is cancelled silently on every channel; a former member is never notified about a household's plan.
- **Both the "accepted" notification and FEAT-13's same-day correction notification (XBR-09) would fire for the same swap** -- These are independent notifications to different concerns (Sam learns his suggestion succeeded; the household is told the nightly nudge is now stale) and both are delivered; neither one's Delivery Rules suppress the other.
- **Push delivery capability is unavailable at trigger time** -- Per ASMP-31, delivery degrades to in-app-only discovery: the in-app notification is still recorded immediately and the recipient sees it on next app open; no email fallback exists for this notification (unlike the plan-ready notification's email fallback in FEAT-07), since this is a same-session coordination message rather than a weekly milestone.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-002 (Suggest a Swap) | Triggered by (inbound) | Suggestion creation fires the "arrived" variant |
| FEAT-04.SPEC-004 (Apply Meal Swap) | Triggered by (inbound) | Successful accept fires the "accepted" variant |
| FEAT-04.SPEC-003 (Review Swap Suggestions) | Triggered by (inbound) | Decline action fires the "declined" variant |
| FEAT-04.SPEC-005 (Suggestion Lapse) | Triggered by (inbound) | Lapse fires the "lapsed" variant |
| FEAT-04.SPEC-003 (Review Swap Suggestions) | Navigation (outbound) | The "arrived" CTA deep-links here |
| FEAT-04.SPEC-002 (Suggest a Swap) | Navigation (outbound) | The "accepted," "declined," and "lapsed" CTAs deep-link here |
| FEAT-07.SPEC-005 (device-notification delivery boundary) | References (outbound) | Owns the push-delivery capability this notification relies on |
| FEAT-13 (Tonight's Dinner Reminder) | References (outbound) | Its own same-day correction notification is independent of this spec's "accepted" variant (XBR-09) |

## Analytics and Success Signals

- **swap_suggestion_arrived_notified** (channel: in_app / push) -- supports success-metrics.md: "Weekly Planning Time"
- **swap_suggestion_outcome_notified** (outcome: accepted / declined / lapsed; channel) -- supports success-metrics.md: "Weekly Planning Time"
- **swap_suggestion_notification_delivery_degraded** (variant; reason: push_unavailable) -- N/A -- no Stage 2 metric measures notification delivery degradation specifically; retained so silent delivery loss on this coordination path is observable rather than invisible.

## Acceptance Criteria

**FEAT-04.SPEC-006-AC-01:** Given Sam submits a suggestion for Thursday's dinner, when the suggestion is created, then Maya receives an in-app notification titled "Sam suggested a swap for Thursday" and, where her device supports it, a push notification titled "Swap suggestion waiting."

**FEAT-04.SPEC-006-AC-02:** Given Maya taps the "arrived" notification's Review CTA, when it opens, then she lands on FEAT-04.SPEC-003 (Review Swap Suggestions) with that suggestion in view.

**FEAT-04.SPEC-006-AC-03:** Given Maya accepts Sam's suggestion, when FEAT-04.SPEC-004 completes successfully, then Sam receives an in-app notification titled "Your suggestion was accepted" and a push notification titled "Maya accepted your swap."

**FEAT-04.SPEC-006-AC-04:** Given Maya declines Sam's suggestion, when the decline is recorded, then Sam receives an in-app notification titled "Your suggestion was declined" and a push notification titled "Maya declined your swap."

**FEAT-04.SPEC-006-AC-05:** Given Sam's suggestion lapses because its night passed unanswered, when FEAT-04.SPEC-005 marks it Lapsed, then Sam receives an in-app notification titled "Your suggestion lapsed" and a push notification titled "Swap suggestion lapsed."

**FEAT-04.SPEC-006-AC-06:** Given push delivery fails three times over 30 minutes, when the final retry fails, then no error is shown to the recipient and the in-app notification stands as the delivery of record.

**FEAT-04.SPEC-006-AC-07:** Given the device-notification-delivery capability is unavailable, when a suggestion is created, then the in-app notification is still recorded immediately and no email fallback is attempted.

**FEAT-04.SPEC-006-AC-08:** Given Maya's suggestion (from Sam) resolves before the "arrived" push is delivered, when the resolution completes first, then the pending "arrived" push is cancelled.

**FEAT-04.SPEC-006-AC-09:** Given three suggestions arrive for Maya within the same minute, when each is created, then each produces its own separate notification -- no batched digest is sent.

**FEAT-04.SPEC-006-AC-10:** Given a suggestion is superseded by Maya's direct swap on the same slot rather than explicitly declined, when the superseding rule sets its outcome, then Sam receives the "declined"-style notification reflecting that outcome.

**FEAT-04.SPEC-006-AC-11:** Given this notification exists with no on/off preference, when Sam or Maya looks for a way to turn it off, then no such control exists in the product, consistent with the notification's always-on design.

**FEAT-04.SPEC-006-AC-12:** Given Sam is removed from the household after suggesting a swap but before the resulting notification fires, when the trigger fires, then the notification is cancelled silently on every channel.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (in-app, push) | 2 |
| Trigger Paths | 4 (arrived, accepted, declined, lapsed) | 4 |
| Preference States | 1 (always on -- no toggle exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Integration Spec: Swap Alternatives Generation

## Overview

**Name:** Swap Alternatives Generation
**ID:** FEAT-04.SPEC-007
**Type:** Integration
**Purpose:** Produces a scoped, one-slot set of candidate alternatives for a swap when the underlying plan was AI-generated, using the same AI text/plan-generation capability that built the original plan.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap

## Scope and Non-Goals

**In Scope:**
- Requesting one-slot candidate recipes from the AI text/plan-generation capability when the target Weekly Plan's origin is AI-generated
- Receiving generated candidates back and handing them to FEAT-04.SPEC-008 for the same safety/schedule filter original generation uses
- User-facing behavior when the capability is slow, unavailable, or rejects a request
- Disclosure of what data is shared with the capability for this scoped request

**Non-Goals:**
- Sourcing alternatives for a manually-built plan -- those come from a direct recipe-library filter with no AI involvement (FEAT-04.SPEC-008), keeping free-tier households at zero AI cost (scope-boundaries.md SC-16)
- Choosing the AI vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate
- The original weekly plan generation request -- owned by FEAT-03.SPEC-010 (the sibling Integration spec for full-week generation); this spec covers only the scoped, single-slot swap request
- Applying the safety/schedule filter to returned candidates, or producing the scarcity explanation -- owned by FEAT-04.SPEC-008; this spec only produces raw candidates for that filter to evaluate

## Capability Category

**Category:** AI text/plan generation
**Dependency Source:** ASMP-30 -- "AI text/plan-generation capability -- Required to generate the weekly dinner plan and swap alternatives on the paid tier" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "AI text/plan generation (ASMP-30)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-03, FEAT-04; Integration Specs: FEAT-03.SPEC-010, FEAT-04.SPEC-007)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Maya sees a short list of freshly-generated safe alternatives when she swaps a meal on an AI-originated plan | Swap a meal / See safe alternatives only | FEAT-04.SPEC-001 (Meal Swap Direct) |
| Sam sees the same freshly-generated alternatives when suggesting a swap on an AI-originated plan | Suggest a swap | FEAT-04.SPEC-002 (Suggest a Swap) |
| The alternatives set reflects the household's schedule constraint and avoids repeating dinners already in the week's plan | See safe alternatives only | FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Slot schedule constraint | Weekly Plan -- weekly_schedule (the night's time limit, if any) | A swap alternatives request is made on an AI-originated plan | The capability needs the same time-fit constraint original generation used, so alternatives fit the household's stated weeknight limit (Weekly Planning consistency; success-metrics.md, Weeknight Time-Fit Accuracy) |
| Recently-served recipe names | Planned Meal -- recipe (names of dinners already in the current week's plan) | Same as above | Prevents the capability from generating a repeat of a dinner already planned that week |
| Household size | Household -- (member count, functionally) | Same as above | The capability sizes portions and cost estimates for the returned recipe candidates |

Dietary Rule content -- allergies, religious rules, vegetarian settings, dislikes -- never leaves the product through this integration. The capability generates candidates from schedule and repeat-avoidance constraints only; every candidate is safety-checked inside the product by FEAT-02 before it can reach either screen, so the capability never needs and is never given the household's dietary rule data (consistent with the dependency map's Data Sensitivity note: FEAT-04 "neither stores nor displays raw Dietary Rule records itself"). Budget, member names, and every other Household or Member Profile field also never leave the product through this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Candidate recipe (name, ingredients, steps, cook time, rough cost) | The capability returns generated candidates for the request | Recipe -- treated as candidate data passed directly to FEAT-04.SPEC-008's safety/schedule filter; only candidates that pass are ever displayed or could be written to a Planned Meal |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Candidates generated | The capability successfully returns one or more candidate recipes for the request | None directly -- candidates are handed to FEAT-04.SPEC-008 for filtering, which is the point at which any user-visible result is produced | The alternatives list renders on FEAT-04.SPEC-001 or FEAT-04.SPEC-002 once FEAT-04.SPEC-008's filter completes | FEAT-04.SPEC-008 |
| Generation returned zero candidates | The capability completes but returns no candidates for the constraints given | None | FEAT-04.SPEC-008 treats this as the zero-alternatives case and produces its scarcity explanation | FEAT-04.SPEC-008 |
| Generation failed | The capability reports an error processing the request | None | Handled as the "Capability Rejects" degradation path below | FEAT-04.SPEC-001, FEAT-04.SPEC-002 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-04.SPEC-001 (Meal Swap Direct) | The screen's Loading Alternatives indicator continues showing past the couple-of-seconds baseline; after 8 seconds a note appears alongside it: "Still finding alternatives -- this is taking longer than usual." The Current Meal card and the rest of the plan remain fully usable while waiting. | Error (alternatives fetch) state: "Couldn't load alternatives. Check your connection and try again." with a Retry button; the original meal is unchanged and remains on the plan. | Same as Capability Down -- from the user's perspective a rejected request and an unavailable capability both present as the alternatives-fetch failure with the retry option; the original meal is unaffected either way. |
| FEAT-04.SPEC-002 (Suggest a Swap) | Same behavior as FEAT-04.SPEC-001: the Loading Alternatives indicator persists with the same "Still finding alternatives" note after 8 seconds. | Error (alternatives fetch) state: "Couldn't load alternatives. Check your connection and try again." with a Retry button; no suggestion is created and the current meal is unaffected. | Same as Capability Down. |
| FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation) | N/A -- this spec only evaluates whatever candidates arrive; it has no independent request to this capability and cannot itself experience slowness. | N/A -- if no candidates arrive, FEAT-04.SPEC-008 never runs its filter on AI-originated candidates for that request; the calling screen's degradation message covers the user experience. | N/A -- same reasoning as Capability Down. |

Every degraded path leaves the underlying plan and any prior suggestion state unchanged -- no half-created Swap Suggestion or partially-written Planned Meal results from a failed or slow request.

## Consent and Disclosure

- **Continuation of AI plan generation consent** -- The household already disclosed and consented to sharing plan-generation data with the AI text/plan-generation capability when it subscribed to the paid tier and received its first AI-generated plan (FEAT-03.SPEC-010's disclosure moment). This integration is a scoped continuation of that same capability for a single slot rather than a new relationship, so no additional disclosure moment interrupts the swap flow. The "How your plan is generated" reference available from account settings (owned by FEAT-03.SPEC-010) documents this integration's scope alongside full-week generation.
- **What is never shared** -- Dietary Rule content of any kind, budget, and all Member Profile and Household fields beyond household size stay inside the product for every swap-alternatives request, as stated in Data Exchanged above; this boundary is documented in the same "How your plan is generated" reference.

## Edge Cases

- **A generation request is made for a plan that was AI-generated but has since been downgraded to a free-tier household mid-week** -- The request is not sent; per XBR-05, a downgrade never removes existing plan data, but new AI generation (including scoped swap requests) stops immediately on downgrade, so FEAT-04.SPEC-008 routes to the manual recipe-library filter instead for any swap attempted after the downgrade takes effect.
- **The same generation request is somehow submitted twice (a retry after a slow response that eventually also returns)** -- Both responses are evaluated by FEAT-04.SPEC-008 independently; whichever the user acts on first is the one applied, and the other's candidates are simply discarded once the screen leaves the Loading Alternatives state.
- **Candidates arrive for a slot the user has since navigated away from** -- The response is discarded; no notification or list update occurs on a screen the user is no longer viewing.
- **The capability goes down mid-request after already committing to generate** -- No partial candidate list is ever shown; the screen shows the full Capability Down message only once the request is confirmed as failed, never a partially-populated list.
- **Generation returns candidates but every one fails FEAT-04.SPEC-008's safety/schedule filter** -- Treated identically to "Generation returned zero candidates" from the user's perspective: FEAT-04.SPEC-008 produces its zero-alternatives scarcity explanation, since no candidate reaches the user regardless of whether the capability generated none or generated some that were filtered out.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-001 (Meal Swap Direct) | Triggered by (inbound) | Requests a scoped alternatives set when the plan is AI-originated |
| FEAT-04.SPEC-002 (Suggest a Swap) | Triggered by (inbound) | Same request path for Sam's suggestion flow |
| FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation) | Affects (outbound) | Receives raw candidates for safety/schedule filtering and routing |
| FEAT-03.SPEC-010 (AI Weekly Dinner Plan Generation Integration) | References (outbound) | Sibling Integration spec for full-week generation; shares the same capability and consent moment |

## Analytics and Success Signals

- **swap_alternatives_generation_requested** (slot night) -- supports success-metrics.md: "One-Tap Swap Completion"
- **swap_alternatives_generation_succeeded** (candidate count, request duration) -- supports success-metrics.md: "One-Tap Swap Completion"
- **swap_alternatives_generation_degraded** (condition: slow / down / rejected; screen: spec ID) -- N/A -- no Stage 2 metric measures generation-capability degradation specifically; retained so the product's tolerance for capability trouble on the swap path is observable.

## Acceptance Criteria

**FEAT-04.SPEC-007-AC-01:** Given Maya opens FEAT-04.SPEC-001 for a slot on an AI-originated plan and taps Swap, when this integration requests candidates, then the request carries the night's schedule constraint, the week's already-planned recipe names, and household size, and no dietary rule data.

**FEAT-04.SPEC-007-AC-02:** Given the capability returns three candidate recipes, when this integration receives them, then they are handed to FEAT-04.SPEC-008 for the safety/schedule filter before anything is shown to Maya.

**FEAT-04.SPEC-007-AC-03:** Given the capability returns zero candidates for the request, when FEAT-04.SPEC-008 processes the empty result, then the zero-alternatives scarcity explanation is shown.

**FEAT-04.SPEC-007-AC-04:** Given the capability takes longer than 8 seconds to respond, when Maya is still waiting on FEAT-04.SPEC-001, then the note "Still finding alternatives -- this is taking longer than usual." appears alongside the loading indicator.

**FEAT-04.SPEC-007-AC-05:** Given the capability is unavailable, when Maya taps Swap on an AI-originated plan's slot, then FEAT-04.SPEC-001 shows "Couldn't load alternatives. Check your connection and try again." with a Retry button, and the original meal is unchanged.

**FEAT-04.SPEC-007-AC-06:** Given the capability rejects the request, when the rejection is received, then Sam's FEAT-04.SPEC-002 shows the same fetch-failure message as the capability-down case, and no suggestion is created.

**FEAT-04.SPEC-007-AC-07:** Given a household downgrades to the free tier mid-week, when Maya attempts a swap on a slot from her prior AI-generated plan after the downgrade, then no request is sent to this capability and FEAT-04.SPEC-008 routes to the manual recipe-library filter instead.

**FEAT-04.SPEC-007-AC-08:** Given Maya navigates away from FEAT-04.SPEC-001 while a request is in flight, when the capability's response later arrives, then it is discarded with no effect on any screen.

**FEAT-04.SPEC-007-AC-09:** Given generation returns candidates that all fail the safety/schedule filter, when FEAT-04.SPEC-008 completes its evaluation, then the zero-alternatives scarcity explanation is shown, identical to a zero-candidate response from the capability.

**FEAT-04.SPEC-007-AC-10:** Given this is Sam's first swap suggestion attempt on an AI-originated plan, when he views what the app shares with the AI capability, then the "How your plan is generated" reference confirms dietary rule data is never sent.

**FEAT-04.SPEC-007-AC-11:** Given the capability goes down mid-request, when the failure is confirmed, then no partially-populated alternatives list is ever shown -- only the full Capability Down message.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 3 | 3 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 6 (2 screens x 3 conditions; SPEC-008 rows are N/A) | 6 |
| Consent and Disclosure | 2 | 2 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Alternatives Computation & Scarcity Explanation

## Overview

**Name:** Alternatives Computation & Scarcity Explanation
**ID:** FEAT-04.SPEC-008
**Type:** Logic/Rule
**Purpose:** Routes alternative sourcing by plan origin, applies the safety/schedule filter every candidate must pass, and determines the limited- or zero-alternatives explanation shown when few or no options qualify.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap
**Governed Entity:** Planned Meal (the slot being swapped, and the candidate set that can legally fill its recipe field)

## Scope and Non-Goals

**In Scope:**
- Routing an alternatives request to the AI text/plan-generation capability (FEAT-04.SPEC-007) or a direct recipe-library filter, based on the target Weekly Plan's origin
- Applying the allergy/religious hard-rule safety check (via FEAT-02) and the schedule-fit filter to every candidate before it can be shown
- Determining and wording the scarcity explanation when the qualifying set is small or empty
- Authorization for who may request alternatives for a slot

**Non-Goals:**
- Generating raw AI candidates -- owned by FEAT-04.SPEC-007; this spec only filters and routes
- Performing the safety determination itself -- owned by FEAT-02 (Dietary Rules & Allergy Safety Engine); this spec invokes that capability rather than re-implementing it
- Writing the chosen alternative onto the Planned Meal -- owned by FEAT-04.SPEC-004 (Apply Meal Swap); this spec only produces the candidate set a user chooses from
- Enforcing the one-active-swap-per-slot concurrency limit -- owned by FEAT-04.SPEC-009 (Swap Concurrency Lock), a distinct rule set applied after a candidate is chosen

## Governed Entity

**Entity:** Planned Meal
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| night | text/enum | The day of the week this slot occupies (at most one dinner per night) |
| meal_kind | enum | Dinner, or leftover lunch linked to a source dinner |
| recipe | reference | The chosen Recipe -- the field this spec's alternatives set constrains at swap time |
| safety_badge | derived | "Checked against allergies" plus the "always check labels" disclaimer |
| vegetarian_option | boolean | Whether a shared meal carries a vegetarian variant |
| cook_time | derived | Carried from the candidate recipe, sized for the household |
| rough_cost | derived | Carried from the candidate recipe, sized for the household |
| pantry_callout | derived | Which logged pantry items this dinner uses |
| status | enum | Proposed/Picked, Confirmed, Swapped, Removed (safety), Cooked |
| swap_history | list | Prior recipes in this slot |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-04.SPEC-001 | Meal Swap Direct | On tapping Swap; before displaying the alternatives list |
| FEAT-04.SPEC-002 | Suggest a Swap | On tapping Suggest a swap; before displaying the alternatives list |
| FEAT-04.SPEC-007 | Swap Alternatives Generation | Supplies raw candidates for AI-originated plans, which this spec then filters |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| recipe (candidate set) | Must pass the same allergy/religious hard-rule check as original plan generation (XBR-01) | Always, for every candidate before it can appear in the alternatives list | On alternatives request | N/A -- failing candidates are silently excluded, never shown with an error; if the excluded set leaves too few options, the Scarcity Explanation rules below produce the user-facing message | Yes |
| recipe (candidate set) | Cook time must be at or under the night's stated time limit | Only when the household's weekly_schedule marks this night as time-constrained | On alternatives request | N/A -- excluded silently, same as above | Yes |
| recipe (candidate set) | Must not duplicate a recipe already planned elsewhere in the current week | Always | On alternatives request | N/A -- excluded silently, same as above (prevents a swap from creating an unintended repeat within the same week) | No -- a soft preference, not a hard exclusion; see Business Rules |
| night, meal_kind, vegetarian_option, pantry_callout, status, swap_history, safety_badge, cook_time, rough_cost | No validation beyond data type -- these fields are read by this spec to build the request context (e.g., night's schedule limit) or displayed alongside a candidate but are not themselves subject to a rule this spec defines | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Safety and schedule filters combine | recipe (safety), recipe (cook_time vs. night's schedule limit) | A candidate must pass both the safety check and the schedule-fit check to qualify; failing either excludes it from the qualifying set | N/A -- exclusion is silent; combined failure is reflected only in a lower qualifying count feeding the Scarcity Explanation |
| Origin determines sourcing | night (via the slot's parent Weekly Plan -- origin), recipe (candidate source) | If the parent Weekly Plan's origin is AI-generated, candidates are requested from FEAT-04.SPEC-007; if manually-built, candidates are drawn directly from the household's recipe library (starter and imported recipes, FEAT-08/FEAT-10) with no AI request, keeping free-tier households at zero AI cost (scope-boundaries.md SC-16) | N/A -- routing is internal and produces no user-facing message of its own |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Request alternatives for a slot (to apply directly) | Maya (Organiser) | Always | -- |
| Request alternatives for a slot (to submit a suggestion) | Sam (Other Adult Member) | Always (Own-only: the resulting candidate can only become his own suggestion, never a direct write) | -- |
| Request alternatives for a slot (to apply directly) | Sam (Other Adult Member) | Never | The Swap affordance on the Weekly Plan routes Sam to FEAT-04.SPEC-002 (Suggest a Swap) instead of the direct-apply screen; a direct-apply request from Sam is not offered anywhere in the product |
| Request alternatives for a slot | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role; no request path is reachable |
| Request alternatives for a slot | Jordan (older kid, limited login -- Later) | Never | Meal Swap access is None for this role (Access Matrix); the swap affordance is not shown |
| Request alternatives for a slot | Riley (Operator, support) | Never | Meal Swap access is None for this role; not shown in the support read-only view |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Qualifying alternatives set | Candidates from FEAT-04.SPEC-007 (AI-originated) or the recipe library (manually-built), filtered by the safety and schedule cross-field rule above, minus recipes already planned elsewhere this week where a non-duplicate alternative exists | Every time an alternatives request is made | No -- the filter itself is not user-configurable; the user only chooses among what qualifies |
| Scarcity explanation text | See Business Rules below for the exact derivation logic | Whenever the qualifying set is smaller than a comfortable browsing size or empty | No |

## Business Rules

- **Scarcity thresholds and wording:** When the qualifying set has 3 or more candidates, no scarcity explanation is shown -- the list speaks for itself. When it has 1-2 candidates, the explanation "Only {N} option{s} fit{s} tonight's {constraint description}." appears below the list (e.g., "Only 2 options fit tonight's 30-minute limit and everyone's dietary rules."), naming whichever constraint(s) actually narrowed the set (schedule, safety, or both). When it has 0 candidates, the message "No safe alternatives fit tonight -- every recipe that qualifies is already in this week's plan or fails someone's allergy rule." is shown alone, with no selectable items, naming the same narrowing reasons in plain terms.
- **Duplicate-avoidance is a soft preference, not a hard exclusion:** if applying the duplicate-avoidance rule would leave zero candidates, previously-planned recipes are added back into the qualifying set (still subject to the hard safety and schedule filters) so the household is never shown zero alternatives purely because every safe, time-fitting recipe happens to already be planned this week; the scarcity explanation notes "including a repeat from earlier this week" in that case.
- **XBR-01 fail-closed:** a candidate with incomplete ingredient data is excluded, never shown unchecked -- this applies identically to AI-generated candidates and recipe-library candidates.
- **Origin routing is fixed at request time:** a plan's origin (AI-generated or manually-built) is read from its Weekly Plan record at the moment of the request; a household that upgrades or downgrades mid-week uses whichever routing its current plan's origin dictates for any swap attempted after the change (see FEAT-04.SPEC-007, Edge Cases).
- **Mid-week rule tightening reruns this filter, not just original generation:** when a hard dietary rule changes mid-week (XBR-02), any Planned Meal the re-check flags is opened for swap through FEAT-04.SPEC-001, and this spec's filter runs exactly as it would for a voluntary swap -- there is no separate rule set for a re-check-triggered swap.

## Edge Cases

- **Exactly 3 candidates qualify** -- No scarcity explanation is shown (the threshold is "3 or more"); this is the boundary between the explanation and no-explanation states.
- **Exactly 2 candidates qualify** -- The "Only 2 options..." explanation is shown; this is the boundary between the 1-2 wording and the 3-or-more silence.
- **Every recipe that passes safety and schedule is already planned this week (0 candidates before the duplicate-avoidance override)** -- The soft-preference override adds previously-planned recipes back in, per Business Rules; the household is never left with zero alternatives solely due to duplicate-avoidance when at least one safe, time-fitting recipe exists anywhere in its pool.
- **A night carries no schedule constraint at all** -- The schedule-fit filter contributes no exclusions for that night; only the safety filter narrows the set, and the scarcity explanation (if any) names only the safety constraint.
- **Sam requests alternatives for a suggestion and Maya requests alternatives for the same slot at the same time (from different screens)** -- Each request is evaluated independently against the same current plan state; both may see the same or a slightly different qualifying set depending on timing, but neither request blocks the other, since no write occurs until a candidate is chosen (concurrency is handled downstream by FEAT-04.SPEC-009 at the point of selection, not here).
- **A candidate recipe has incomplete ingredient data** -- Excluded from the qualifying set regardless of source (fail-closed, XBR-01); it never appears even as part of a scarcity explanation's count.

## Acceptance Criteria

**FEAT-04.SPEC-008-AC-01:** Given Maya requests alternatives for a slot on an AI-originated plan, when this spec routes the request, then it is sent to FEAT-04.SPEC-007 rather than the recipe library.

**FEAT-04.SPEC-008-AC-02:** Given Maya requests alternatives for a slot on a manually-built plan, when this spec routes the request, then candidates are drawn directly from the recipe library with no AI request made.

**FEAT-04.SPEC-008-AC-03:** Given a candidate recipe fails the allergy/religious hard-rule check, when the filter runs, then that candidate is silently excluded from the qualifying set.

**FEAT-04.SPEC-008-AC-04:** Given a candidate recipe's cook time exceeds a time-constrained night's limit, when the filter runs, then that candidate is silently excluded.

**FEAT-04.SPEC-008-AC-05:** Given exactly 3 candidates qualify, when the alternatives list renders, then no scarcity explanation is shown.

**FEAT-04.SPEC-008-AC-06:** Given exactly 2 candidates qualify, when the alternatives list renders, then the explanation "Only 2 options fit tonight's {constraint}." appears below the list.

**FEAT-04.SPEC-008-AC-07:** Given zero candidates qualify and at least one safe, time-fitting recipe exists but is already planned this week, when the filter completes, then that recipe is added back into the qualifying set and the explanation notes "including a repeat from earlier this week."

**FEAT-04.SPEC-008-AC-08:** Given zero candidates qualify even after the duplicate-avoidance override, when the filter completes, then the message "No safe alternatives fit tonight -- every recipe that qualifies is already in this week's plan or fails someone's allergy rule." is shown alone with no selectable items.

**FEAT-04.SPEC-008-AC-09:** Given Sam requests alternatives to suggest a swap, when the request is made, then it is allowed (Own-only) and produces the same filtered candidate set logic as Maya's request for the same slot.

**FEAT-04.SPEC-008-AC-10:** Given Sam attempts to reach a direct-apply alternatives request, when he looks for that path, then it does not exist -- his swap tap routes to FEAT-04.SPEC-002 instead.

**FEAT-04.SPEC-008-AC-11:** Given Riley (Operator) has no Meal Swap access, when any alternatives-request path is examined for his role, then none exists.

**FEAT-04.SPEC-008-AC-12:** Given a candidate has incomplete ingredient data, when the filter runs, then that candidate is excluded regardless of its source (AI-generated or recipe-library).

**FEAT-04.SPEC-008-AC-13:** Given a mid-week hard rule change flags a previously-safe Planned Meal (XBR-02), when Maya opens FEAT-04.SPEC-001 for that slot, then this spec's filter runs identically to a voluntary swap request.

**FEAT-04.SPEC-008-AC-14:** Given a night carries no schedule constraint, when the filter runs, then only the safety check narrows the candidate set, and any scarcity explanation names only the safety constraint.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Swap Concurrency Lock

## Overview

**Name:** Swap Concurrency Lock
**ID:** FEAT-04.SPEC-009
**Type:** Logic/Rule
**Purpose:** Enforces at most one active swap operation per meal slot at a time, preventing duplicate swaps from a double tap or a race between a direct swap and an accepted suggestion.
**Parent Feature:** FEAT-04 -- One-Tap Meal Swap
**Governed Entity:** Planned Meal (the slot's in-flight-swap state, guarding writes to its status and recipe fields)

## Scope and Non-Goals

**In Scope:**
- Acquiring and releasing a per-slot lock around any operation that would write a new recipe onto a Planned Meal
- Defining exactly what a second, concurrent attempt on the same slot experiences while the lock is held
- Authorization for who may acquire this lock

**Non-Goals:**
- Deciding which candidates are safe to swap to -- owned by FEAT-04.SPEC-008 (Alternatives Computation & Scarcity Explanation); this spec governs only the write-time exclusivity, not candidate eligibility
- Performing the write itself -- owned by FEAT-04.SPEC-004 (Apply Meal Swap), which acquires this lock before writing and releases it afterward
- Locking anything at the Weekly Plan level or across different slots -- excluded per product-features.md's Validation & Limits, which scopes the "no more than one active swap operation" rule to "per meal slot"; swaps on different nights never contend with each other
- Preventing a suggestion from being created while a slot is locked -- a new suggestion (FEAT-04.SPEC-010) does not itself write to Planned Meal, so submitting one is never blocked by this lock; only the act of applying a swap is

## Governed Entity

**Entity:** Planned Meal
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| night | text/enum | The day of the week this slot occupies -- the scope unit this lock is keyed to |
| meal_kind | enum | Dinner, or leftover lunch linked to a source dinner |
| recipe | reference | The field this lock protects against a concurrent overwrite |
| safety_badge | derived | Not governed by this spec |
| vegetarian_option | boolean | Not governed by this spec |
| cook_time | derived | Not governed by this spec |
| rough_cost | derived | Not governed by this spec |
| pantry_callout | derived | Not governed by this spec |
| status | enum | The field this lock protects against a concurrent double-write to Swapped |
| swap_history | list | Not governed by this spec directly, though it receives the write this lock protects |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-04.SPEC-001 | Meal Swap Direct | Acquired immediately after Maya selects an alternative, before FEAT-04.SPEC-004 is invoked |
| FEAT-04.SPEC-003 | Review Swap Suggestions | Acquired immediately after Maya taps Accept, before FEAT-04.SPEC-004 is invoked |
| FEAT-04.SPEC-004 | Apply Meal Swap | Holds the lock for the duration of its processing; releases it on completion (success or failure) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| status (via the lock) | At most one active swap operation may target this Planned Meal's status/recipe pair at a time | Whenever a swap-apply attempt (direct or accepted-suggestion) is made for this slot | On lock acquisition, before FEAT-04.SPEC-004 begins | "This meal is already being swapped." (shown on FEAT-04.SPEC-001) / "Couldn't complete this action. Try again." (shown on FEAT-04.SPEC-003, since the accept simply fails to proceed) | Yes |
| night, meal_kind, recipe (read-only reference), safety_badge, vegetarian_option, cook_time, rough_cost, pantry_callout, swap_history | No validation beyond data type -- these fields are not independently governed by this lock | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Lock scope is the slot, not the plan | night (as the lock key), status | The lock is keyed to one specific Planned Meal (one night, one Weekly Plan); an operation on a different night's slot in the same plan is never blocked by a lock held elsewhere in the plan | N/A -- this is a scoping rule with no error condition of its own |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Acquire the swap lock for a slot (direct swap) | Maya (Organiser) | Always, subject to the lock being free | If the lock is held: "This meal is already being swapped." shown in place of the alternatives list on FEAT-04.SPEC-001 |
| Acquire the swap lock for a slot (applying an accepted suggestion) | Maya (Organiser) | Always, subject to the lock being free | If the lock is held: the accept action on FEAT-04.SPEC-003 fails and that card shows "Couldn't complete this action. Try again." |
| Acquire the swap lock for a slot | Sam (Other Adult Member) | Never -- Sam never applies a swap directly; his path (FEAT-04.SPEC-002) creates a suggestion, which does not acquire this lock | N/A -- Sam has no path that attempts to acquire this lock |
| Acquire the swap lock for a slot | Jordan (young kid, no login), Jordan (older kid, limited login), Riley (Operator) | Never | N/A -- none of these roles has any Meal Swap access path that could attempt to acquire this lock |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Lock key | Derived as the specific Planned Meal (one Weekly Plan, one night) targeted by the swap-apply attempt | On every lock acquisition attempt | No |
| Lock hold duration | Held for the full duration of FEAT-04.SPEC-004's processing (safety re-check through write completion) and released immediately on that automation's completion, whether it succeeds or fails | Always | No |

## Business Rules

- **Scope is per meal slot, per product-features.md's Validation & Limits:** "no more than one active swap operation per meal slot at a time (prevents duplicate swaps from a double tap)." A double tap on the same alternative, and a race between a direct swap and an accepted suggestion for the same slot, are the two scenarios this rule exists to prevent.
- **The lock is released even on failure:** a failed safety re-check or a failed write (FEAT-04.SPEC-004's failure outcomes) always releases the lock so a retry can proceed; the lock is never left held after an operation ends, under any outcome.
- **Only an apply attempt acquires the lock -- a request for alternatives never does:** browsing alternatives (FEAT-04.SPEC-008) is read-only and does not contend with an in-flight swap; the lock guards only the moment a candidate is chosen and the write begins.
- **First-to-acquire wins; the second attempt is refused outright, not queued:** this rule does not make a second attempt wait for the first to finish and then proceed automatically -- the user sees the Locked message immediately and must retry once the first operation has completed, keeping the "one tap" experience honest about what actually happened rather than silently reordering swaps.

## Edge Cases

- **Maya double-taps the same alternative rapidly on FEAT-04.SPEC-001** -- The screen's own debounce (disabling other items during Applying) is the first line of defense; even if a second request reached this spec, the lock acquisition for the second would fail since the first is already held, and the second shows the Locked message.
- **A direct swap by Maya and an accepted suggestion for the same slot are triggered at effectively the same moment (from two devices)** -- Whichever acquires the lock first proceeds through FEAT-04.SPEC-004; the second is refused with the Locked message on FEAT-04.SPEC-001 or the accept-failure message on FEAT-04.SPEC-003, depending on which path lost the race. Per the dependency map's Planned Meal Contention note, this is reject-with-refresh at the slot level, and a double tap never creates two swaps.
- **The lock-holding operation fails (safety re-check or write failure)** -- The lock releases immediately per Business Rules; a subsequent attempt on the same slot is not blocked by the failed attempt.
- **A swap on one night and a swap on a different night in the same Weekly Plan are attempted simultaneously** -- Both proceed independently; the lock is scoped per slot, not per plan (Cross-Field Rules).
- **The lock-holding operation stalls indefinitely (e.g., the device loses connectivity mid-operation)** -- The lock is tied to the server-side completion of FEAT-04.SPEC-004, not to the initiating device staying connected; if that automation's processing itself times out, its own failure path (FEAT-04.SPEC-004's Write Failure outcome) releases the lock, so a stalled client never leaves the slot permanently locked.

## Acceptance Criteria

**FEAT-04.SPEC-009-AC-01:** Given no swap operation is active on a slot, when Maya selects an alternative on FEAT-04.SPEC-001, then the lock is acquired and FEAT-04.SPEC-004 proceeds.

**FEAT-04.SPEC-009-AC-02:** Given a swap operation is already active on a slot, when Maya attempts a second swap on that same slot, then the lock acquisition fails and FEAT-04.SPEC-001 shows "This meal is already being swapped."

**FEAT-04.SPEC-009-AC-03:** Given a swap operation is already active on a slot, when Maya attempts to accept a suggestion for that same slot on FEAT-04.SPEC-003, then the accept fails and that card shows "Couldn't complete this action. Try again."

**FEAT-04.SPEC-009-AC-04:** Given a direct swap and an accepted suggestion for the same slot are triggered at effectively the same time, when both attempt to acquire the lock, then exactly one proceeds and the other is refused.

**FEAT-04.SPEC-009-AC-05:** Given a lock-holding operation's safety re-check fails, when FEAT-04.SPEC-004 reports that failure, then the lock is released immediately.

**FEAT-04.SPEC-009-AC-06:** Given a lock-holding operation's write fails, when FEAT-04.SPEC-004 reports that failure, then the lock is released immediately and a retry on the same slot can proceed.

**FEAT-04.SPEC-009-AC-07:** Given swaps are attempted on two different nights in the same Weekly Plan at the same time, when both attempt to acquire their respective locks, then both proceed independently.

**FEAT-04.SPEC-009-AC-08:** Given Maya double-taps the same alternative rapidly, when the second tap registers, then it is ignored by the screen's debounce before ever reaching this spec's lock acquisition.

**FEAT-04.SPEC-009-AC-09:** Given Sam has no path that applies a swap directly, when his suggestion flow is examined, then it never attempts to acquire this lock.

**FEAT-04.SPEC-009-AC-10:** Given a lock is held while a request for alternatives (not an apply attempt) is made for the same slot, when that request is processed, then it succeeds normally -- browsing alternatives never contends with the lock.

**FEAT-04.SPEC-009-AC-11:** Given a lock-holding operation's client loses connectivity mid-operation, when the server-side processing (FEAT-04.SPEC-004) reaches its own failure path, then the lock is released rather than remaining held indefinitely.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



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
