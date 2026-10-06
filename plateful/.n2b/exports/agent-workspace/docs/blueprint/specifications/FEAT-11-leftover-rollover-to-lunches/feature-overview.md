---
document_type: feature-overview
feature_number: FEAT-11
feature_name: Leftover Rollover to Lunches
feature_slug: leftover-rollover-to-lunches
priority_tier: Important
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 4
screen_count: 1
automation_count: 2
logic_rule_count: 1
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Leftover Rollover to Lunches

## Summary

**Feature:** Leftover Rollover to Lunches
**ID:** FEAT-11
**Description:** Leftovers from a planned dinner are carried forward as a suggested lunch on a following day, so extra portions get eaten instead of thrown away.
**Priority:** Important
**Phase:** MVP
**Type:** User-Facing
**Rationale:** The brief names this directly as part of the core vision: "Leftovers roll into lunches" (BRIEF.md, Vision), and the "nothing gets thrown away" moment in the Experience section depends on it. Important rather than Core because the plan and grocery list function completely without it, but it is included at MVP because it is a defining part of the brief's stated experience, not a later refinement.

**Key Capabilities:**
- Suggest a leftover lunch — A dinner cooked with extra portions is offered as a lunch suggestion on a following day
- Confirm or skip — Household member confirms the leftover lunch happened or skips it if it did not

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-11.SPEC-001 | Leftover Lunch Card | Screen | Maya, Sam, Jordan (older kid, Later), Riley | Displays the leftover lunch suggestion attached to a day; Maya or Sam confirms it happened or skips it |
| FEAT-11.SPEC-002 | Leftover Lunch Suggestion Generation | Automation | Maya, Sam, Jordan (older kid, Later), Riley | As part of plan generation, creates a Suggested leftover lunch for each dinner the eligibility rule flags as leftover-producing |
| FEAT-11.SPEC-003 | Leftover Lunch Eligibility & Linking Rule | Logic/Rule | Maya, Sam, Jordan (older kid, Later), Riley | Determines which dinners produce leftover-worthy portions, which following day (no more than two days later) to suggest, and enforces the one-source/one-day link |
| FEAT-11.SPEC-004 | Leftover Lunch Withdrawal on Source Change | Automation | Maya, Sam, Jordan (older kid, Later), Riley | When a linked source dinner is swapped or removed, re-evaluates and updates or withdraws its leftover lunch suggestion |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Suggest a leftover lunch | FEAT-11.SPEC-002, FEAT-11.SPEC-003 | Generation automation creates the Suggested leftover lunch once the eligibility/linking rule determines a dinner qualifies and picks the following day | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |
| Confirm or skip | FEAT-11.SPEC-001 | Leftover Lunch Card lets Maya or Sam confirm the lunch happened or skip it, with no penalty or follow-up prompt on skip | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-11.SPEC-003 | Leftover Lunch Eligibility & Linking Rule | Phase 5 (Rule Discovery) | Validation & Limits fixes the one-source-dinner/one-following-day link and the two-day ceiling, but deriving *which* dinners qualify as leftover-producing and *which* day to suggest is non-trivial derivation logic shared by generation (SPEC-002) and withdrawal (SPEC-004) — crossing the standalone-spec threshold rather than being re-derived twice |
| FEAT-11.SPEC-004 | Leftover Lunch Withdrawal on Source Change | Phase 4 (Trigger-Response, External Dependencies/cross-feature lens) | XBR-10 and the dependency map's Cross-Feature Business Rules require that swapping or removing the source dinner updates or withdraws its leftover suggestion; this is a cross-feature inbound side effect with real processing logic (re-run eligibility, decide update vs. removal), not a same-screen consequence |

## Entity-Lifecycle Coverage Matrix

**Entity: Planned Meal (leftover lunch)**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-11.SPEC-002 | Generation automation creates a Suggested leftover-lunch Planned Meal linked to one source dinner, as part of plan generation | -- |
| Read (single) | FEAT-11.SPEC-001 | Leftover Lunch Card reads the suggestion attached to the day being viewed | -- |
| Read (list) | N/A | The full week's Planned Meals (dinners and leftover lunches together) are read by the Weekly Plan screen owned by FEAT-03/FEAT-23; this feature only ever reads the single suggestion it displays | -- |
| Update | FEAT-11.SPEC-001, FEAT-11.SPEC-004 | Confirm/skip status write (SPEC-001); re-link to a new source or day when the original source changes (SPEC-004) | Concurrent status updates are last-write-wins (feature-dependency-map.md, Planned Meal Contention) |
| Delete/Archive | FEAT-11.SPEC-004 | Hard delete of an unconfirmed Suggested record when its source dinner is swapped or removed and no replacement dinner in that slot is itself leftover-eligible; no restore path (a fresh suggestion is generated only if a newly eligible dinner takes the slot); no retention/purge policy is needed because only a still-Suggested record is ever removed | Confirmed Eaten or Skipped outcomes are never deleted -- retained as part of the plan's permanent history (scope-boundaries.md SC-18) |
| State Transition | FEAT-11.SPEC-001, FEAT-11.SPEC-004 | Suggested -> Eaten or Suggested -> Skipped (SPEC-001, no follow-up prompt either way); Suggested -> re-linked (new source/day) or Suggested -> removed (SPEC-004) | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Weekly Plan | FEAT-11.SPEC-001 | Locates which day's plan the suggestion is attached to |
| Planned Meal (source dinner) | FEAT-11.SPEC-002, FEAT-11.SPEC-003, FEAT-11.SPEC-004 | Reads the source dinner's recipe and portion data to determine leftover eligibility and to keep the leftover-lunch link current |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Weekly plan generation completes and a dinner is flagged leftover-producing | Compute the following day (no more than two days later) and create a Suggested leftover lunch linked to that dinner | Standalone Automation | FEAT-11.SPEC-002 |
| Any dinner enters or changes in the plan | Evaluate whether it qualifies as leftover-producing and, if so, which following day to suggest | Standalone Logic/Rule | FEAT-11.SPEC-003 |
| Maya or Sam taps Confirm on a leftover lunch card | Set status to Eaten | Inline in triggering screen | FEAT-11.SPEC-001 |
| Maya or Sam taps Skip on a leftover lunch card | Set status to Skipped; no penalty or follow-up prompt | Inline in triggering screen | FEAT-11.SPEC-001 |
| Two household members mark the same leftover lunch at nearly the same time | Last write wins; no conflict error is shown | Inline in triggering screen | FEAT-11.SPEC-001 |
| A source dinner is swapped (FEAT-04) or cleared/changed manually (FEAT-23) | Re-run eligibility on the new dinner; update the leftover suggestion's link or withdraw it if no longer eligible | Standalone Automation | FEAT-11.SPEC-004 |
| A leftover lunch is suggested, confirmed, or skipped | No message is sent to any person -- the suggestion appears within the plan itself | Explicit non-goal (feature's own Communications field: N/A) | -- |

## Shared Context

**Shared Entities:**
- Planned Meal (leftover-lunch sub-type) -- created by SPEC-002, displayed and status-updated by SPEC-001, re-linked or removed by SPEC-004; eligibility and day-window governed by SPEC-003. Fields: meal_kind (leftover lunch), linked source dinner, night (the following day, at most two days after the source), status (Suggested, Eaten, Skipped).

**Shared UI Patterns:**
- Leftover Lunch Card -- the same compact suggestion-plus-confirm/skip layout wherever it appears within the Weekly Plan, whether the plan is AI-generated (FEAT-03) or manually built (FEAT-23). Spec Writers should describe it once and reference it consistently.

**Shared Validation/Logic:**
- FEAT-11.SPEC-003 defines eligibility (which dinners produce leftover-worthy portions), the following-day computation, and the fixed one-source/one-day link; FEAT-11.SPEC-002 and FEAT-11.SPEC-004 both reference it rather than re-deriving eligibility or the two-day ceiling independently.

## Internal Dependency Map

```
SPEC-002 (Leftover Lunch Suggestion Generation) -> [runs during plan generation] -> SPEC-003 (Leftover Lunch Eligibility & Linking Rule) -> [flags a dinner as leftover-producing and picks the following day] -> SPEC-002 creates the Suggested leftover-lunch Planned Meal
SPEC-002 -> [suggestion created] -> SPEC-001 (Leftover Lunch Card) [displays it attached to the following day]
SPEC-001 -> [Maya or Sam taps Confirm] -> [status set to Eaten]
SPEC-001 -> [Maya or Sam taps Skip] -> [status set to Skipped, no follow-up]
FEAT-04 (One-Tap Meal Swap) / FEAT-23 (Manual Weekly Planning) -> [source dinner swapped or cleared] -> SPEC-004 (Leftover Lunch Withdrawal on Source Change) -> [re-checks SPEC-003 eligibility on the new dinner] -> [updates the link or removes the suggestion] -> SPEC-001 [reflects the update, or the card disappears]
```

**Default Entry:** FEAT-11.SPEC-001 (Leftover Lunch Card) -- reached from the Weekly Plan screen (owned by FEAT-03 or FEAT-23) on the day the suggestion applies; there is no separate feature-level entry point, since the card only ever appears embedded in the plan.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-11.SPEC-002 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | Leftover lunch suggestions are generated as part of the weekly plan generation run | Weekly plan generated (Sunday plan generation) |
| FEAT-11.SPEC-002 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) | The leftover-lunch note shown on the source dinner itself when cooking is sourced from this feature's generated suggestion | Household member opens the dinner to cook it (Weeknight Dinner, Nudge & Rating, step 2) |
| FEAT-11.SPEC-001 | Outbound | FEAT-03 (AI Weekly Dinner Plan Generation) / FEAT-23 (Manual Weekly Planning) | The Leftover Lunch Card is displayed embedded within the Weekly Plan screen those features own | Household member views the plan on the day a leftover lunch is suggested |
| FEAT-11.SPEC-004 | Inbound | FEAT-04 (One-Tap Meal Swap) | A swap on a source dinner triggers re-evaluation of its linked leftover suggestion | Swap applied to a slot with a linked leftover suggestion (XBR-10) |
| FEAT-11.SPEC-004 | Inbound | FEAT-23 (Manual Weekly Planning) | Clearing or changing a source dinner manually triggers re-evaluation of its linked leftover suggestion | Night cleared or changed manually (XBR-10) |

## Non-Functional Notes

**Data volumes / growth:** At most one leftover-lunch Planned Meal is created per leftover-eligible dinner per week per household, alongside up to seven dinner Planned Meals per Weekly Plan (feature-dependency-map.md, Weekly Plan Relationships); volume grows only with household count, not with usage intensity.

**Responsiveness:** The suggestion is generated as part of plan generation with no separate loading step, and a failure to compute it simply omits the suggestion rather than showing an error (product-features.md, States); an already-generated suggestion stays viewable offline (product-features.md, States; ASMP-25).

**Data sensitivity / privacy:** The leftover-lunch Planned Meal carries the same classification as Planned Meal generally -- household personal data (what the family eats), private to the household and never sold or used for advertising (feature-dependency-map.md, Planned Meal Data Sensitivity; ASMP-14, ASMP-26).

**Compliance flags:** N/A -- no compliance regime applies beyond the general household-data privacy posture already noted; this feature carries no medical or diet advice, consistent with the product's stated exclusion of nutrition-based recommendations (scope-boundaries.md SC-06).

## Non-Goals

- **Detailed leftover quantity or expiry tracking** -- Excluded per scope-boundaries.md SC-11: the brief follows the lighter "use up what I tell you I have" model rather than a detailed inventory with quantities or expiry dates; a leftover lunch is a simple yes/no suggestion, not a tracked quantity.
- **Breakfast and general lunch planning beyond leftover rollover** -- Excluded per scope-boundaries.md's deferral note on meal scope: the product plans dinners only for v1, with lunches covered solely by this feature; standalone breakfast or non-leftover lunch planning is Later-phase and depends on evidence of unmet need.
- **Rescheduling a leftover lunch to a different day, or creating one manually without a source dinner** -- Not modeled: product-features.md's Validation & Limits fixes the link to exactly one source dinner and one following day computed by the system; no reschedule or manual-creation path is stated, so a household that wants a different day waits for the next generation cycle or simply skips the suggestion.
- **A standalone notification for a suggested, confirmed, or skipped leftover lunch** -- Excluded per this feature's own Communications field (product-features.md): the suggestion appears within the plan itself and no separate notification is sent for any of its three outcomes.
- **Automatic purge of confirmed leftover-lunch history** -- Intentional lifecycle decision surfaced by the CRUD matrix: Eaten and Skipped outcomes are retained as part of the plan's permanent history rather than purged, consistent with scope-boundaries.md SC-18's account-life retention for plan history.
