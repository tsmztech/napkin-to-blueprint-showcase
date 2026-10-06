---
document_type: feature-overview
feature_number: FEAT-04
feature_name: One-Tap Meal Swap
feature_slug: one-tap-meal-swap
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 10
screen_count: 3
automation_count: 2
logic_rule_count: 3
integration_count: 1
notification_count: 1
---

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
