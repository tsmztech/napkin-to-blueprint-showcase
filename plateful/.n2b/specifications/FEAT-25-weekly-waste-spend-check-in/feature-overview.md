---
document_type: feature-overview
feature_number: FEAT-25
feature_name: Weekly Waste & Spend Check-In
feature_slug: weekly-waste-spend-check-in
priority_tier: Important
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 5
screen_count: 2
automation_count: 1
logic_rule_count: 2
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Weekly Waste & Spend Check-In

## Summary

**Feature:** Weekly Waste & Spend Check-In
**ID:** FEAT-25
**Description:** Once a week, the household can answer one quick, optional question about how much food it threw away and roughly what it spent on groceries, and see a simple trend of its own answers against where it started.
**Priority:** Important
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md, Success Criteria states "families say they throw away noticeably less food and spend less." The draft's food-waste metric depended on households reporting this, but no feature asked or held the answers. Important rather than Core because the plan-and-list loop works without it; MVP because a before-and-after comparison needs answers from the first weeks. Kept to one skippable question so it does not eat into the brief's under-10-minutes-a-week planning goal.

**Key Capabilities:**
- Answer the week's check-in — An adult picks "none", "a little" or "a lot" for food thrown away and can add a rough grocery spend
- Set a starting point — At the first check-in, the household says how much it typically threw away and spent before Plateful
- See the trend — The household sees its recent answers alongside its weekly budget and its starting point

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-25.SPEC-001 | Weekly Check-In Card | Screen | Maya, Sam | Adult answers, corrects, or skips the week's waste-and-spend question, including capturing the household's one-time starting point at the first check-in |
| FEAT-25.SPEC-002 | Check-In Trend View | Screen | Maya, Sam | Household sees its recent check-in answers alongside its starting point and weekly budget, with a path to invite another family |
| FEAT-25.SPEC-003 | Weekly Check-In Cycle | Automation | Maya, Sam | Opens each week's check-in, marks an unanswered week as skipped, and locks the prior week's answer once the next one opens |
| FEAT-25.SPEC-004 | Check-In Trend Calculation | Logic/Rule | Maya, Sam | Derives the household's change against its starting point and against its weekly budget for display in the trend view |
| FEAT-25.SPEC-005 | Check-In Validation & Access Rules | Logic/Rule | All | Governs the one-answer-per-week/latest-wins rule, spend validation, the editable window, and who may view or answer the check-in |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Answer the week's check-in | FEAT-25.SPEC-001, FEAT-25.SPEC-003, FEAT-25.SPEC-005 | Card presents the two-tap question and optional spend; cycle automation opens/closes the answering window; validation rules govern the one-answer-per-week and latest-wins behavior | Phase 2 (Explicit) |
| Set a starting point | FEAT-25.SPEC-001 | The first-use variant of the card captures the household's typical prior waste and spend before any weekly answer exists | Phase 2 (Explicit) |
| See the trend | FEAT-25.SPEC-002, FEAT-25.SPEC-004 | Trend view displays recent answers against the starting point and budget; derivation rules compute the change values shown | Phase 2 (Explicit) |

**Analyst-Discovered Specs** — Specs not directly tied to a single Key Capability, surfaced by Phases 3–6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-25.SPEC-003 | Weekly Check-In Cycle | Phase 4 (Trigger-Response — time-based trigger) | The dependency map's entity lifecycle ("Offered -> Answered or Skipped -> editable until the next week's check-in opens") and the Primary Flows field's "skipped week shows as a gap ... with no follow-up reminder" describe a recurring, time-based state change nothing in the explicit capabilities names a process for |
| FEAT-25.SPEC-004 | Check-In Trend Calculation | Phase 5 (Rule-Constraint Discovery — derivation) | The Data Notes field's "Derived: change against the baseline and against budget" is a non-trivial derivation (two independent comparisons, one against a one-time value and one against a value from another feature) that crosses the standalone Logic/Rule threshold |
| FEAT-25.SPEC-005 | Check-In Validation & Access Rules | Phase 5 (Rule-Constraint Discovery) + Phase 3 (Entity-Lifecycle — editable window) | The Validation & Limits field's conflict rule ("the latest answer from any adult wins") interacts with the Access field's role split (Maya/Sam Full; both Jordan rows and Riley None) and the CRUD matrix's editable-until-next-open window — three interacting conditions crossing the standalone-spec threshold |

## Entity-Lifecycle Coverage Matrix

**Entity: Waste & Spend Check-In**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-25.SPEC-001 | Adult submits the week's answer (and, on the household's first check-in, its one-time starting point) through the card's two-tap form | The record for a given week is also implicitly created as "Offered" by FEAT-25.SPEC-003 when the week's window opens, before any answer exists |
| Read (single) | FEAT-25.SPEC-001 | Card reads the current week's own record to show whether it is unanswered, answered, or locked | -- |
| Read (list) | FEAT-25.SPEC-002 | Trend view reads recent weeks' records plus the starting point to render the household's own history | -- |
| Update | FEAT-25.SPEC-001 | Any adult with access resubmits the week's answer while it remains open; FEAT-25.SPEC-005 governs the one-answer-per-week, latest-write-wins resolution when more than one adult answers | -- |
| Delete/Archive | N/A | Every past check-in answer is kept for the life of the household account and remains available after a downgrade to the free tier (scope-boundaries.md SC-18); no delete, archive, or purge mechanism exists — an explicit non-goal, not an omission | -- |
| State Transition | FEAT-25.SPEC-001, FEAT-25.SPEC-003 | Offered (week's window opens) -> Answered (adult submits, FEAT-25.SPEC-001) or Skipped (window closes with no answer, FEAT-25.SPEC-003); an Answered record remains editable only until the next week's check-in opens (FEAT-25.SPEC-003 locks it, FEAT-25.SPEC-005 governs the boundary) | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Household | FEAT-25.SPEC-001, FEAT-25.SPEC-002, FEAT-25.SPEC-004 | Reads the household's weekly_budget (FEAT-01) for trend comparison and currency (FEAT-16) for spend entry and display |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| A new week begins | Check-in record opens as Offered for the household | Standalone Automation | FEAT-25.SPEC-003 |
| The next week's check-in opens with the prior week unanswered | Prior week's record is marked Skipped; no reminder is sent | Standalone Automation | FEAT-25.SPEC-003 |
| The next week's check-in opens with the prior week answered | Prior week's answer becomes locked (no longer editable) | Standalone Automation | FEAT-25.SPEC-003 |
| Adult submits or resubmits the week's answer | Record is validated (positive spend, currency) and saved; if a second adult already answered this week, the latest submission wins | Standalone Logic/Rule | FEAT-25.SPEC-005 |
| Save fails while answering | Answer is kept on the device and retried automatically without adult intervention | Inline in triggering screen | FEAT-25.SPEC-001 |
| Adult answers or views the card offline | Card accepts the answer and syncs it once connectivity returns | Inline in triggering screen | FEAT-25.SPEC-001 |
| Household opens the trend view | Change against the starting point and against the weekly budget is computed for display | Standalone Logic/Rule | FEAT-25.SPEC-004 |
| Household's weekly_budget changes (FEAT-01) | Trend view's budget comparison reflects the new figure the next time it renders | Inline in triggering screen | FEAT-25.SPEC-002 |
| Household's currency changes (FEAT-16) | Historical spend figures convert for display rather than leaving them inconsistent (XBR-11, owned by FEAT-16) | Cross-feature — logged in touchpoints | FEAT-16 responsibility |
| Member taps "invite another family" on the trend card | Navigates to the Invite Another Household screen | Cross-feature — logged in touchpoints | FEAT-24 responsibility |
| Household is deleted (FEAT-18) | Cascade or export behavior for its Waste & Spend Check-In history | Cross-feature — logged in touchpoints | FEAT-18 responsibility |

## Shared Context

**Shared Entities:**
- Waste & Spend Check-In — created and updated by SPEC-001 (weekly answers and the one-time starting point), transitioned by SPEC-003 (Offered/Skipped/locked), read in aggregate by SPEC-002, derived from by SPEC-004, validated by SPEC-005. Fields: household, week, waste_amount (none/a little/a lot), spend (optional, positive, household currency), starting_point_waste, starting_point_spend, status (Offered/Answered/Skipped, locked flag).

**Shared UI Patterns:**
- Single evolving card — SPEC-001 renders the same card in its first-use (starting-point), unanswered, and answered/editable states rather than as separate screens; SPEC-002's trend content appears once the card has at least one answer or a starting point. Spec Writers for both should describe this as one continuous surface, not two disconnected screens.

**Shared Validation:**
- SPEC-005 defines the one-answer-per-week/latest-wins rule, the spend validation, the editable window, and role access. SPEC-001 and SPEC-002 both reference SPEC-005 rather than duplicating these rules.

## Internal Dependency Map

```
SPEC-003 (Weekly Check-In Cycle) -> [new week begins] -> SPEC-001 (Weekly Check-In Card) [shows fresh Offered state]
SPEC-001 (Weekly Check-In Card) -> [adult submits weekly answer] -> [validated by] -> SPEC-005 (Check-In Validation & Access Rules)
SPEC-003 (Weekly Check-In Cycle) -> [window closes, no answer] -> Waste & Spend Check-In record marked Skipped
SPEC-003 (Weekly Check-In Cycle) -> [next week opens] -> [locks] -> prior week's Waste & Spend Check-In record
SPEC-001 (Weekly Check-In Card) -> [answer saved or starting point set] -> SPEC-002 (Check-In Trend View) [trend updates]
SPEC-002 (Check-In Trend View) -> [renders change values using] -> SPEC-004 (Check-In Trend Calculation)
SPEC-004 (Check-In Trend Calculation) -> [reads] -> FEAT-01 (Household Setup & Member Profiles) [weekly_budget]
SPEC-002 (Check-In Trend View) -> [reads] -> FEAT-16 (Units, Currency & Locale Configuration) [currency]
SPEC-002 (Check-In Trend View) -> [member taps "invite another family"] -> FEAT-24 (Invite Another Household)
```

**Default Entry:** SPEC-001 (Weekly Check-In Card) — the card shown at the top of the week's plan (FEAT-03) for a signed-in adult with access; it carries the household straight into SPEC-002's trend content once a starting point or an answer exists.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-25.SPEC-001 | Inbound | FEAT-03 (AI Weekly Dinner Plan Generation) | The check-in card appears at the top of the week's plan | Household's week plan renders |
| FEAT-25.SPEC-002, FEAT-25.SPEC-004 | Inbound | FEAT-01 (Household Setup & Member Profiles) | Reads the household's weekly budget for the trend's budget comparison | Trend view renders or a change value is derived |
| FEAT-25.SPEC-001, FEAT-25.SPEC-002 | Inbound | FEAT-16 (Units, Currency & Locale Configuration) | Reads household currency for spend entry and display; historical figures convert on a later currency change (XBR-11, owned by FEAT-16) | Spend is entered or displayed; household currency setting changes |
| FEAT-25.SPEC-002 | Outbound | FEAT-24 (Invite Another Household) | Trend card links out to the invite-another-household screen | Member taps "invite another family" |
| Waste & Spend Check-In (Delete/Archive) | Inbound | FEAT-18 (Account & Data Management) | Cascade and export behavior for check-in history when a household is deleted or exports its data is owned by FEAT-18, not this feature | Household deletion or data export completes |

## Non-Functional Notes

**Data volumes / growth:** At most one check-in record per household per week (roughly 52 a year) plus one one-time starting-point record; volumes stay trivial even across several thousand households (assumptions-constraints.md ASMP-24), and every record is kept for the life of the household account, remaining available after a downgrade to the free tier (scope-boundaries.md SC-18).

**Responsiveness:** Answering is a single two-tap action that confirms instantly, with no loading state (feature's States field); the check-in stays reachable one-handed with large tap targets, consistent with the product's accessibility baseline (assumptions-constraints.md ASMP-29). The trend view needs no faster response than any other summary screen — it is not a real-time collaborative surface like the shared grocery list (ASMP-25 does not apply here).

**Data sensitivity / privacy:** Personal household data (self-reported waste behavior and rough grocery spend) — private to household members, aggregated across households only to measure product success, and never sold or used for advertising (feature's Data Notes; assumptions-constraints.md ASMP-26).

**Compliance flags:** General personal-data rights (export, deletion) apply to check-in records as part of the household's data under assumptions-constraints.md ASMP-27; no medical or diet-advice regime applies, since the check-in captures only a rough waste amount and spend figure, never a nutrition or health-score derivation (scope-boundaries.md SC-06). No children's data is captured here — the young kid profile and the Later-phase older-kid login both have no access to this feature.

## Non-Goals

- **Follow-up reminder for a skipped week** — Excluded per the feature's own Primary Flows & Alternates field ("a skipped week shows as a gap in the trend, with no follow-up reminder"): keeping notifications limited to the weekly plan and nightly nudge (Communications field) protects the brief's under-10-minutes-a-week planning goal.
- **Nutrition or health scoring of food waste** — Excluded per scope-boundaries.md SC-06: the product gives no medical or diet advice; the check-in stays limited to a rough waste-amount category and a spend figure, never a nutrition or "health score" derivation.
- **Deletion or purge of check-in history** — Intentional lifecycle decision surfaced by the CRUD matrix and required by scope-boundaries.md SC-18: every past check-in answer is kept for the life of the household account and remains available after a downgrade to the free tier; no retention window or purge mechanism applies.
- **Cross-household comparison or leaderboard** — Intentional design decision per the feature's Data Notes field ("Displayed: the household's own trend"): the trend view shows a household only its own answers against its own starting point and budget, never a comparison to other households.
- **Push, email, or in-app notification prompting the weekly question** — Excluded per the feature's own Communications field ("N/A — the check-in appears as an in-app card only"): the question surfaces solely as a card at the top of the plan, never as a separate outbound message.
