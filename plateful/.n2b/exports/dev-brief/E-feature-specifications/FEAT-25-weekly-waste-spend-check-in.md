# FEAT-25 — Weekly Waste & Spend Check-In

This chapter covers FEAT-25, Weekly Waste & Spend Check-In, a Important-tier feature. It contains 5 specifications carrying 76 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-25.SPEC-001 | Weekly Check-In Card | screen | 18 |
| FEAT-25.SPEC-002 | Check-In Trend View | screen | 14 |
| FEAT-25.SPEC-003 | Weekly Check-In Cycle | automation | 9 |
| FEAT-25.SPEC-004 | Check-In Trend Calculation | logic-rule | 13 |
| FEAT-25.SPEC-005 | Check-In Validation & Access Rules | logic-rule | 22 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Weekly Check-In Card

## Overview

**Name:** Weekly Check-In Card
**ID:** FEAT-25.SPEC-001
**Type:** Screen
**Purpose:** Adult answers, corrects, or skips the week's waste-and-spend question, including capturing the household's one-time starting point at the first check-in.
**Parent Feature:** FEAT-25 -- Weekly Waste & Spend Check-In

## Scope and Non-Goals

**In Scope:**
- Rendering the card at the top of the household's week plan in its current variant: first-use (starting-point) or standard weekly question
- Capturing the two-tap weekly answer (waste_amount, and an optional spend amount) for the current, open week
- Capturing the household's one-time starting point (starting_point_waste and optional starting_point_spend) the first time the household ever answers, bundled into that same interaction
- Correcting (resubmitting) the current week's answer while it remains open
- Reading and displaying the current week's own record status (unanswered, answered, or -- at the moment of a cutover in flight -- newly locked)
- Answering while offline, with the answer held on the device and synced automatically once connectivity returns
- Transitioning into the trend content of FEAT-25.SPEC-002 (Check-In Trend View) on the same surface once a starting point or an answer exists

**Non-Goals:**
- Displaying the household's trend of recent answers, its change against the starting point, or its change against the weekly budget -- owned by FEAT-25.SPEC-002 (Check-In Trend View); this card shows only the current week's own status
- Opening a new week's record, marking an unanswered week Skipped, or locking a prior week's answer -- owned by FEAT-25.SPEC-003 (Weekly Check-In Cycle); this screen only reflects the record's current status, never assigns it directly
- Deriving any change-against-baseline or change-against-budget figure -- owned by FEAT-25.SPEC-004 (Check-In Trend Calculation); this card shows only the raw entered answer
- Sending a reminder or notification for a skipped week -- excluded per feature-overview.md's Non-Goals: "a skipped week shows as a gap in the trend, with no follow-up reminder," which keeps notifications limited to the weekly plan and nightly nudge
- Appearing at the top of the free-tier, manually built week (FEAT-23.SPEC-001, Weekly Plan Manual Week Builder) -- the Feature Breakdown Brief's Cross-Feature Touchpoints and Internal Dependency Map name only the AI-generated week view (FEAT-03.SPEC-001) as this card's entry point; FEAT-23.SPEC-001 itself declares no check-in entry point, so this spec limits its declared surface to FEAT-03.SPEC-001 rather than inventing a second one
- Any nutrition or "health score" interpretation of the waste answer -- excluded per scope-boundaries.md SC-06: the product gives no medical or diet advice

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-03.SPEC-001 (Weekly Plan View) | The household's AI-generated week plan renders and a check-in card is due (the household's current-week record is not yet locked) | Household id; the card reads and writes that household's current-week Waste & Spend Check-In record |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full card: current variant (first-use or standard), current week's status, and the trend content once it exists | Answer, correct (resubmit while open), or leave unanswered the week's question; set the household's one-time starting point | -- |
| Sam (Other Adult Member) | Full card: same content Maya sees -- the record is a single shared household answer, not a per-member one | Answer, correct (resubmit while open), or leave unanswered the week's question; set the household's one-time starting point -- per the Access Matrix's Full entitlement, matching Maya's, Sam's submission is accepted outright against the shared weekly record; if Maya also submits the same week, whichever submission arrives last is the one kept (FEAT-25.SPEC-005) | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | Card is not shown; a no-login profile has no access to any screen in the product |
| Jordan (older kid, limited login -- Later) | No | No | Card is not shown in this role's view of the week's plan; the Access Matrix's Waste Check-In column is None for this role |
| Riley (Operator, support -- from v1) | No | No | Card is not shown, even while Riley holds an open Support Request's read-only view of the household's plan (FEAT-22); the Access Matrix's Waste Check-In column is None for Riley, keeping this personal household data outside operator visibility entirely |
| Unauthenticated | No | No | Redirected to the sign-in screen; no check-in content of any kind is exposed |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- an in-progress, not-yet-submitted selection is preserved on the device and offered again after re-authentication succeeds |

## Layout and Content

The card sits in the top banner area of the household's week plan (FEAT-03.SPEC-001), above the weekly total banner, appearing whenever the household's current-week record is not yet locked. It renders in one of two variants, chosen by whether the household has ever set a starting point.

**First-use (starting-point) variant -- shown when the household has no starting point yet:**

**Header:** Card title "Before Plateful: how much did your household typically throw away and spend?"

**Body, in order:**
- A short one-line explanation: "This helps you see your own progress later. It's optional and takes a minute."
- **Typical waste** -- three tappable options in a single row: "None," "A little," "A lot" (single selection; this sets starting_point_waste)
- **Typical weekly spend (optional)** -- a numeric input showing the household's configured currency symbol, labeled "Roughly what did you spend on groceries each week?"
- A visual divider, then this week's own question rendered exactly as the standard variant's body (below), so the starting point and the current week's first answer are captured together in one interaction
- "Save" action, right-aligned below the combined body

**Standard (weekly question) variant -- shown once a starting point exists:**

**Header:** Card title "How much did your household throw away this week?" A small "x" dismiss control sits at the top-right corner of the card.

**Body:**
- Three tappable options in a single row: "None," "A little," "A lot" (single selection; sets waste_amount for the current week; selecting an option saves immediately -- no separate Save button)
- **Grocery spend this week (optional)** -- a numeric input showing the household's configured currency symbol, labeled "Roughly what did you spend on groceries this week?"; this field saves on blur, independently of the waste selection

Once the current week already carries an answer, the previously selected waste option is shown pre-selected and the spend field (if provided) is pre-filled; both remain editable while the week is open.

### Responsive Behavior

- **Compact breakpoint:** The three waste options stack as a single full-width row of three equal segments; the spend field sits directly below, full width.
- **Medium size class and above:** Layout is unchanged in structure; the card is capped at the same platform-wide reading width as the surrounding plan content (FEAT-03.SPEC-001) and horizontally centered.
- **First-use variant's combined body:** The starting-point section and the current week's section each keep the compact-breakpoint stacking described above; the divider between them remains a single full-width rule at every size.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Waste option ("None" / "A little" / "A lot") -- standard variant | Tap | Saves the selected value as this week's waste_amount, validated and recorded via FEAT-25.SPEC-005 | Selected option shows a selected visual state; card moves toward the Answered/Editable state | Inline confirmation, e.g. a brief checkmark on the card -- no full-page loading state |
| Spend field -- standard variant | Type, then blur | On blur, saves the entered amount as this week's spend, validated via FEAT-25.SPEC-005 | Field shows the entered value | Inline confirmation on successful save; inline error below the field on validation failure |
| Dismiss ("x") -- standard variant | Tap | Hides the card for this session only; does not change the week's record status | Card collapses/hides from the plan view for the remainder of this session | Card reappears the next time the plan is opened, unchanged, while the week remains open |
| Typical waste option -- first-use variant | Tap | Selects the value pending submission (not yet saved) | Selected option shows a selected visual state | Standard selection state; no save yet |
| Typical spend field -- first-use variant | Type | Captures the pending value (not yet saved) | Field shows entered text | Standard input focus state |
| This week's waste option -- first-use variant | Tap | Selects the value pending submission (not yet saved) | Selected option shows a selected visual state | Standard selection state; no save yet |
| This week's spend field -- first-use variant | Type | Captures the pending value (not yet saved) | Field shows entered text | Standard input focus state |
| "Save" -- first-use variant | Tap | 1. Validate all entered fields via FEAT-25.SPEC-005. 2. If valid, save starting_point_waste (required), starting_point_spend (optional), waste_amount (required), and spend (optional) together to the current week's record. | Button shows a brief inline saving confirmation, not a full-page spinner | Success: card transitions to the standard variant's Answered/Editable appearance, and the trend content (FEAT-25.SPEC-002) becomes available on the same surface. Failure: inline error messages on the invalid field(s); entered values are preserved. |
| "Save" -- first-use variant, while loading | Tap | No action -- debounced | None | Button remains in its brief saving state |

### Accessibility Notes

- **Focus order (standard variant):** Card title -> waste options in order (None -> A little -> A lot) -> spend field -> dismiss control.
- **Focus order (first-use variant):** Card title -> typical-waste options -> typical-spend field -> divider -> this-week's waste options -> this-week's spend field -> Save.
- **Selection announcements:** Selecting a waste option announces the chosen value and, in the standard variant, the resulting save confirmation to assistive technology.
- **Validation announcements:** A spend-field error is announced and programmatically associated with the field at the moment it appears.
- **Keyboard alternatives:** Every action on this card (selecting a waste option, entering spend, dismissing, saving) is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| First-Use (Starting Point) | Combined starting-point-and-current-week form described above, both sections empty | Household has never set a starting point when the plan renders | Household submits the combined form (Save) |
| Unanswered (Offered) | Standard variant, no waste option selected, spend field empty | Household has a starting point; current week's record is Offered with no answer yet | An option is tapped (moves to Answered/Editable) or the week is locked by FEAT-25.SPEC-003 with no answer (card no longer shows this week; a fresh Offered card for the new week appears next) |
| Answered/Editable | Standard variant, previously chosen option pre-selected, spend field pre-filled if provided | Current week's record has waste_amount set and remains open (not locked) | Household resubmits (stays in this state with new values) or the week is locked by FEAT-25.SPEC-003 (card no longer shows this week) |
| Locked-at-cutover (transient) | Brief banner: "This week's check-in has closed." replacing the card momentarily | Household submits (or attempts to submit) at the exact moment FEAT-25.SPEC-003's cutover locks the record | Card immediately refreshes to the new week's fresh Offered (or First-Use) state |
| Error | Inline error banner on the affected field; entered values preserved | A save attempt is rejected for a reason other than the lock cutover (e.g. an invalid spend amount) | Household corrects the field and resubmits |
| Offline/Degraded | Banner: "You're offline -- your answer is saved on this device and will sync once you're back online." Card remains fully interactive; entered values are held locally. | Connectivity is lost while the card is open or being answered | Connectivity returns -- the held answer syncs automatically and the standard save confirmation appears; no duplicate submission occurs |

## Validation Rules

Validation governed by FEAT-25.SPEC-005 (Check-In Validation & Access Rules). See that spec for the complete field-by-field rules, the one-answer-per-week/latest-wins resolution, and the editable-window boundary. This screen applies validation on field blur (spend) and on save (starting-point form, when shown).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Trend content link/expansion, once a starting point or an answer exists | FEAT-25.SPEC-002 (Check-In Trend View) | -- |
| "Invite another family" (reached through the trend content on this same surface) | FEAT-24.SPEC-001 (Invite Another Household Screen) | FEAT-24 (Invite Another Household) |

## Data Model

**Creates:** N/A -- the current week's Waste & Spend Check-In record already exists as "Offered," created by FEAT-25.SPEC-003, before this card ever renders; this screen's contribution (the dependency map's Entity-Lifecycle Coverage Matrix lists it under "Create") is populating that empty record with its first real answer data, described under Updates below.
**Reads:** Waste & Spend Check-In -- the household's current-week record (waste_amount, spend, status, locked) to choose the right state; whether any Waste & Spend Check-In record with a starting point exists for the household, to choose the first-use vs. standard variant. Household -- currency, to label the spend field and the first-use typical-spend field (FEAT-16.SPEC-001).
**Updates:** Waste & Spend Check-In -- writes waste_amount and optional spend into the current week's open record; on the household's first-ever answered check-in only, also writes starting_point_waste (required) and optional starting_point_spend into that same record.
**Deletes:** None -- no delete action exists on this entity (feature-overview.md's Entity-Lifecycle Coverage Matrix: Delete/Archive is N/A; every past record is retained per scope-boundaries.md SC-18).

## Business Rules

- All field validation, the one-answer-per-week/latest-wins resolution, and the editable-until-next-week-opens window are governed by FEAT-25.SPEC-005; this screen enforces them but does not restate them.
- Spend and typical-spend figures are entered and displayed in the household's currently configured currency (FEAT-16.SPEC-001); a later currency change converts historical figures for display, per XBR-11 (owned by FEAT-16).
- The starting-point fields (starting_point_waste, starting_point_spend) are offered exactly once, on the household's first answered check-in, and never appear again afterward, per FEAT-25.SPEC-005's Business Rules.
- The card's dismiss ("x") control never changes the underlying record's status; only FEAT-25.SPEC-003's cycle can mark a week Skipped, and only by the window closing with no answer.

## Edge Cases

- **Two adults submit different answers for the same week within moments of each other** -- The submission that reaches the record last is the one kept in full (waste_amount and spend together, not merged field by field), per FEAT-25.SPEC-005's one-answer-per-week/latest-wins rule. The adult whose submission was superseded sees the other's answer the next time the card loads; no error and no conflict dialog appears, since the shared-record case is fully resolved by latest-write-wins rather than a reject-with-refresh or merge.
- **Household's very first check-in is dismissed or left unanswered** -- No starting point is set; the next week's card again renders as the First-Use variant (no reminder is sent), and this repeats until the household eventually answers.
- **An adult attempts to submit at the exact moment the following week's cycle locks the record** -- The submission is rejected; the card briefly shows the Locked-at-cutover banner, then refreshes to the new week's fresh state (Offered or First-Use, depending on whether a starting point exists).
- **Spend or typical spend entered as the smallest amount above zero** -- Passes validation; any amount greater than zero is valid.
- **Spend (or typical spend) left blank while the corresponding waste answer is provided** -- Valid; spend is optional independent of whether the waste question was answered.
- **Household answers offline, then changes connectivity mid-session before the queued answer syncs** -- The held answer syncs automatically on the first successful connection; no duplicate record or duplicate submission is created regardless of how many connectivity changes occur before the sync succeeds.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-25.SPEC-003 (Weekly Check-In Cycle) | Triggered by (inbound) | Automation opens the current week's record as Offered before this card ever renders, and locks it once the following week opens |
| FEAT-25.SPEC-002 (Check-In Trend View) | Navigation (outbound) | Trend content appears within this same card surface once a starting point or an answer exists |
| FEAT-25.SPEC-005 (Check-In Validation & Access Rules) | References (inbound) | Field validation, authorization, and the editable-window rule applied to every submission |
| FEAT-03.SPEC-001 (Weekly Plan View) | Navigation (inbound) | Card renders at the top of the household's AI-generated week plan |
| FEAT-16.SPEC-001 (Units, Currency Settings) | References (inbound) | Reads the household's configured currency to label spend fields |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| checkin_shown | variant (first_use / weekly) | Card renders on the plan | supports success-metrics.md: "Reported Food Waste and Spend Reduction" |
| checkin_baseline_set | starting_point_waste, spend_provided (bool) | First-use variant's Save completes successfully | supports success-metrics.md: "Reported Food Waste and Spend Reduction" |
| checkin_answered | variant, waste_amount, spend_provided (bool) | Standard variant's waste option is saved, or the spend field is saved on blur | supports success-metrics.md: "Reported Food Waste and Spend Reduction" |
| checkin_dismissed | variant | Household taps the dismiss ("x") control on the standard variant | supports success-metrics.md: "Reported Food Waste and Spend Reduction" |

## Acceptance Criteria

**FEAT-25.SPEC-001-AC-01:** Given Maya's household has never set a starting point, when she opens the week's plan, then the card shows the First-Use variant with both the typical-waste-and-spend section and this week's own question.

**FEAT-25.SPEC-001-AC-02:** Given Maya is on the First-Use variant, when she selects "A lot" for typical waste, enters a typical weekly spend, selects "A little" for this week, and taps Save, then the household's starting point and this week's answer are both saved, and the card switches to the standard, Answered/Editable variant.

**FEAT-25.SPEC-001-AC-03:** Given Maya is on the First-Use variant, when she taps Save without selecting a typical-waste option, then an error appears on that option group and the save does not proceed.

**FEAT-25.SPEC-001-AC-04:** Given Sam's household already has a starting point and this week's record is Offered, when he taps "A little" for this week's waste, then the answer saves instantly with an inline confirmation and no full-page loading indicator.

**FEAT-25.SPEC-001-AC-05:** Given Sam has just answered this week's waste question, when he enters a spend amount and taps away from the field, then the spend value saves on blur and the field shows the entered value.

**FEAT-25.SPEC-001-AC-06:** Given Sam enters a spend amount of zero and blurs the field, then the field shows the error "Enter an amount greater than zero, or leave it blank," and the spend value is not saved.

**FEAT-25.SPEC-001-AC-07:** Given Maya has already answered this week (waste_amount set) and the week remains open, when she taps a different waste option, then the new selection replaces the prior one and is saved.

**FEAT-25.SPEC-001-AC-08:** Given Maya answered this week's check-in and Sam later submits a different answer for the same week, when Maya next opens the plan, then she sees Sam's answer, not her own, with no error or conflict message.

**FEAT-25.SPEC-001-AC-09:** Given Maya is on the standard variant, when she taps the dismiss ("x") control, then the card hides for this session and the week's record status remains Offered, unchanged.

**FEAT-25.SPEC-001-AC-10:** Given Maya dismissed this week's card without answering, when she opens the plan again later the same week, then the card reappears in its unanswered state.

**FEAT-25.SPEC-001-AC-11:** Given Jordan is a young kid profile with no login, then no device or account of Jordan's can reach this card.

**FEAT-25.SPEC-001-AC-12:** Given the older-kid limited login (Later) opens the week's plan, when they look for the check-in card, then it is not shown to that role.

**FEAT-25.SPEC-001-AC-13:** Given Riley is viewing a household's plan through an open Support Request (FEAT-22), when the plan renders, then the check-in card does not appear anywhere in Riley's view.

**FEAT-25.SPEC-001-AC-14:** Given Sam loses connectivity while answering this week's check-in, when he selects a waste option, then the banner "You're offline -- your answer is saved on this device and will sync once you're back online." appears and the answer is submitted automatically once connectivity returns, with no duplicate submission.

**FEAT-25.SPEC-001-AC-15:** Given Maya's session expires while she has an unsaved selection on the card, when she is prompted to sign in again, then her selection is preserved and offered again after she signs back in successfully.

**FEAT-25.SPEC-001-AC-16:** Given Maya submits her answer at the exact moment the following week's cycle (FEAT-25.SPEC-003) locks the current record, then her submission is rejected, the card briefly shows "This week's check-in has closed," and then refreshes to the new week's fresh state.

**FEAT-25.SPEC-001-AC-17:** Given a household has dismissed or left unanswered every check-in it has ever been offered, when the next week's plan renders, then the card still shows the First-Use variant, since no starting point has ever been set.

**FEAT-25.SPEC-001-AC-18:** Given Maya's household has a starting point and at least one answered week, when she opens the week's plan, then the trend content of FEAT-25.SPEC-002 becomes available on the same card surface.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 6 (first-use, unanswered, answered/editable, locked-at-cutover, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |



# Screen Spec: Check-In Trend View

## Overview

**Name:** Check-In Trend View
**ID:** FEAT-25.SPEC-002
**Type:** Screen
**Purpose:** Household sees its recent check-in answers alongside its starting point and weekly budget, with a path to invite another family.
**Parent Feature:** FEAT-25 -- Weekly Waste & Spend Check-In

## Scope and Non-Goals

**In Scope:**
- Displaying the household's recent Waste & Spend Check-In history (answered and skipped weeks), most recent first
- Displaying the household's change against its one-time starting point and against its weekly budget, using values computed by FEAT-25.SPEC-004
- Showing a skipped week as a plain gap in the history, with no data and no explanation beyond that
- Surfacing the "invite another family" link out to Invite Another Household (FEAT-24)
- Appearing as an extension of the same card surface FEAT-25.SPEC-001 renders, per the Feature Breakdown Brief's Shared Context: "Single evolving card"

**Non-Goals:**
- Answering, correcting, or skipping the current week's question -- owned by FEAT-25.SPEC-001 (Weekly Check-In Card); this view is read-only
- Computing the change-against-starting-point or change-against-budget values themselves -- owned by FEAT-25.SPEC-004 (Check-In Trend Calculation); this view only displays what that spec derives
- Comparing the household's trend to any other household -- excluded per feature-overview.md's Non-Goals: "the trend view shows a household only its own answers against its own starting point and budget, never a comparison to other households"
- Reminding the household about a gap in its own history -- excluded per feature-overview.md's Non-Goals, consistent with the feature's no-follow-up-reminder policy

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-25.SPEC-001 (Weekly Check-In Card) | The card has at least one answered or skipped week, or a starting point, once the household's history exists | Household id; the same current-week record context the card already holds, plus the household's check-in history |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full trend content: history, starting-point comparison, budget comparison, invite link | Tap "invite another family" | -- |
| Sam (Other Adult Member) | Full trend content: same content Maya sees -- the trend is the household's own shared history, not per-member | Tap "invite another family" | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | Trend content is not shown; a no-login profile has no access to any screen in the product |
| Jordan (older kid, limited login -- Later) | No | No | Trend content is not shown in this role's view of the week's plan; the Access Matrix's Waste Check-In column is None for this role |
| Riley (Operator, support -- from v1) | No | No | Trend content is not shown, even while Riley holds an open Support Request's read-only view of the household's plan (FEAT-22); the Access Matrix's Waste Check-In column is None for Riley |
| Unauthenticated | No | No | Redirected to the sign-in screen; no trend content is exposed |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- the trend view carries no unsaved state to preserve, since it is read-only |

## Layout and Content

Trend content appears on the same card surface as FEAT-25.SPEC-001, directly below the current week's question, once the household has a starting point or at least one answered/skipped week. It does not open as a separate screen; it is a continuation region of the same card.

**"Since you started" section:** A short line comparing the most recent answered week against the starting point, in plain terms derived by FEAT-25.SPEC-004 -- e.g., "You started at 'A lot' -- this week you're at 'A little'." for waste, and, when both the current week's spend and the starting point's typical spend exist, a second line such as "You're spending {amount} less than when you started." (or "more," or "about the same," per FEAT-25.SPEC-004's derivation). When the household has a starting point but no answered week yet (only Skipped weeks so far), this section shows: "Answer this week's check-in to see how you're doing against your starting point."

**"This week vs. your budget" section:** When the most recent answered week provided a spend value and the household's weekly_budget is set, a line stating the difference, e.g., "{amount} under your weekly budget" or "{amount} over your weekly budget" or "Right at your weekly budget," per FEAT-25.SPEC-004. When either value is missing, this section shows: "Add a spend amount to compare against your weekly budget." (spend missing) or is omitted entirely (weekly_budget never set at FEAT-01).

**Recent weeks list:** Below the two summary sections, a scrollable list of the household's recent weeks, most recent at the top. Each row shows the week's date range, its waste answer ("None" / "A little" / "A lot"), and its spend if provided. A Skipped week's row shows the week's date range only, with a plain "No answer" marker and no other data -- the feature's own gap-in-the-trend behavior.

**"Invite another family" link:** Sits below the recent weeks list, a single tappable text link: "Know another family who'd want this? Invite them."

### Responsive Behavior

- **Compact breakpoint:** Both summary sections and the recent-weeks list stack full width in the order described above.
- **Medium size class and above:** Layout is unchanged in structure; the trend content is capped at the same platform-wide reading width as the surrounding card and plan content, horizontally centered.
- **Recent weeks list:** Scrolls independently within its own region at every size; it does not cause the surrounding card or plan to scroll horizontally.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Recent weeks list | Scroll | Reveals older weeks in the household's history | List scroll position changes | Standard scroll feedback; no additional data loads beyond what is already synced to the device |
| "Invite another family" link | Tap | Navigate to FEAT-24.SPEC-001 (Invite Another Household Screen) | Screen transitions to the invite flow | Standard navigation transition |

Display-only elements: the "Since you started" section, the "This week vs. your budget" section, and each recent-week row are non-interactive; they present derived and stored values with no action attached.

### Accessibility Notes

- **Focus order:** "Since you started" section -> "This week vs. your budget" section -> recent weeks list (in displayed order, most recent first) -> "Invite another family" link.
- **Content announcements:** When the trend content first becomes available (a starting point or first answer is saved), its appearance is announced to assistive technology as new content, consistent with FEAT-25.SPEC-001's own save-confirmation announcement.
- **Skipped-week rows:** Each Skipped row's "No answer" marker is announced explicitly, distinguishing it from a row with data.
- **Keyboard alternatives:** Scrolling the recent weeks list and activating the invite link are both reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Hidden (Not Yet Available) | Trend content is entirely absent from the card; only FEAT-25.SPEC-001's First-Use variant shows | Household has no starting point and no answered or skipped week yet | A starting point is set or the first week is answered/skipped (FEAT-25.SPEC-001 or FEAT-25.SPEC-003) |
| Showing (Data Available) | Both summary sections (or their "not yet available" placeholder text) and the recent weeks list render as described above | A starting point exists, or at least one week has been answered or skipped | Household navigates away, or a new week's data changes what is shown |
| Loading | A brief inline indicator over the recent weeks list region only; summary sections retain their last-known values if already loaded | Trend content is becoming available for the first time this session, or the household's history is being fetched | Load completes (Showing) or fails (Error) |
| Error | Inline message within the trend region: "Couldn't load your check-in history. Try again." with a retry control | The household's history fails to load | Household taps retry and the load succeeds |
| Offline/Degraded | Previously synced weeks and their already-computed comparison values remain fully viewable; a week not yet synced to this device does not appear until connectivity returns | Connectivity is lost while viewing, or the view is opened while already offline | Connectivity returns and any not-yet-synced weeks appear |

## Validation Rules

Not applicable -- this screen accepts no user input beyond navigation and scrolling; every value shown is read-only, sourced from FEAT-25.SPEC-001's stored answers and FEAT-25.SPEC-004's derivations.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| "Invite another family" tap | FEAT-24.SPEC-001 (Invite Another Household Screen) | FEAT-24 (Invite Another Household) |

## Data Model

**Creates:** None.
**Reads:** Waste & Spend Check-In -- the household's recent records (week, waste_amount, spend, status) and its starting point (starting_point_waste, starting_point_spend), most recent first. Household -- weekly_budget (FEAT-01.SPEC-008) and currency (FEAT-16.SPEC-001), for the budget comparison and spend display. The two derived values (change against starting point, change against weekly budget) computed by FEAT-25.SPEC-004.
**Updates:** None.
**Deletes:** None.

## Business Rules

- The trend view shows a household only its own answers against its own starting point and its own budget -- never a comparison to any other household (feature-overview.md's Non-Goals).
- A Skipped week is shown as a gap: its row carries no waste or spend data, and no change value is computed for it (FEAT-25.SPEC-004).
- Spend figures are displayed in the household's currently configured currency; a later currency change converts existing figures for display, per XBR-11 (owned by FEAT-16).
- Every past check-in record remains visible here for the life of the household account and after a downgrade to the free tier, per scope-boundaries.md SC-18; this view applies no retention window or truncation of history beyond ordinary scrolling.
- If Household's weekly_budget has never been set, the "This week vs. your budget" section is omitted entirely rather than shown with a placeholder, since no comparison is possible without a budget value.

## Edge Cases

- **Household has a starting point but has skipped every week since** -- The "Since you started" section shows its no-answered-week placeholder text; the recent weeks list shows only "No answer" rows.
- **Household's weekly_budget is set after several weeks of check-ins already exist** -- The budget comparison begins appearing for weeks answered from that point forward; weeks answered before the budget was set show no budget-comparison line, since none can be computed retroactively for a value that did not exist at the time.
- **Currency changes between when a week's spend was recorded and when the trend is viewed** -- The displayed spend converts to the newly configured currency before any comparison runs, per XBR-11; the household never sees mismatched-currency figures side by side.
- **Very long check-in history (the household has used Plateful for a long time)** -- The recent weeks list continues to scroll to show older weeks; no week is dropped from the household's own view (scope-boundaries.md SC-18).
- **Household's weekly_budget changes while the trend view is open** -- The budget-comparison line reflects the new figure the next time the view renders (feature-overview.md's Side-Effect Inventory), not live mid-view; no conflict or error is shown, since this view performs no writes of its own.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-25.SPEC-001 (Weekly Check-In Card) | Navigation (inbound) | Trend content appears within the same card surface once a starting point or an answer exists |
| FEAT-25.SPEC-004 (Check-In Trend Calculation) | References (inbound) | Supplies the change-against-starting-point and change-against-budget values this view displays |
| FEAT-25.SPEC-003 (Weekly Check-In Cycle) | References (inbound) | Supplies the finalized (Skipped or locked-Answered) historical records this view reads |
| FEAT-01.SPEC-008 (Weekly Budget & Schedule Setup) | References (inbound) | Supplies the household's weekly_budget for the budget comparison |
| FEAT-16.SPEC-001 (Units, Currency Settings) | References (inbound) | Supplies the household's configured currency for spend display |
| FEAT-24.SPEC-001 (Invite Another Household Screen) | Navigation (outbound) | "Invite another family" link |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| checkin_trend_viewed | weeks_shown (count), has_baseline (bool), has_budget_comparison (bool) | Trend content renders in the Showing state | supports success-metrics.md: "Reported Food Waste and Spend Reduction" |
| checkin_invite_link_tapped | -- | Household taps "Invite another family" | N/A -- this event's referral-growth measure belongs to FEAT-24 (Invite Another Household), not to a metric connected to this feature; recorded here for completeness of this screen's own interaction coverage |

## Acceptance Criteria

**FEAT-25.SPEC-002-AC-01:** Given Maya's household has just set its starting point for the first time, when the card finishes saving, then the trend content appears on the same surface showing the "Since you started" section.

**FEAT-25.SPEC-002-AC-02:** Given Maya's household started at "A lot" and this week's answer is "A little," when she views the trend content, then the "Since you started" section states the waste change from "A lot" to "A little."

**FEAT-25.SPEC-002-AC-03:** Given Sam's household provided a starting typical spend and this week's spend is lower, when he views the trend content, then the spend comparison line states the household is spending less than when it started.

**FEAT-25.SPEC-002-AC-04:** Given Maya's household's most recent answered week provided a spend value and the household's weekly_budget is set, when she views the trend content, then the "This week vs. your budget" section shows the exact difference and whether it is under, over, or at budget.

**FEAT-25.SPEC-002-AC-05:** Given the household's most recent answered week provided no spend value, when Sam views the trend content, then the "This week vs. your budget" section shows its no-spend-provided placeholder text instead of a comparison.

**FEAT-25.SPEC-002-AC-06:** Given the household's weekly_budget has never been set, when Maya views the trend content, then the "This week vs. your budget" section is omitted entirely.

**FEAT-25.SPEC-002-AC-07:** Given a prior week was marked Skipped by FEAT-25.SPEC-003, when Maya scrolls the recent weeks list to that week, then its row shows only the date range and a "No answer" marker, with no waste or spend data.

**FEAT-25.SPEC-002-AC-08:** Given Maya's household has no starting point and no answered or skipped week yet, when she opens the plan, then no trend content appears anywhere on the card.

**FEAT-25.SPEC-002-AC-09:** Given Sam is viewing the trend content, when he taps "Invite another family," then he is navigated to FEAT-24.SPEC-001 (Invite Another Household Screen).

**FEAT-25.SPEC-002-AC-10:** Given Jordan is a young kid profile with no login, then no device or account of Jordan's can reach this trend content.

**FEAT-25.SPEC-002-AC-11:** Given Riley is viewing a household's plan through an open Support Request (FEAT-22), when the plan renders, then no trend content appears anywhere in Riley's view.

**FEAT-25.SPEC-002-AC-12:** Given Maya loses connectivity while viewing the trend content, when she scrolls to a week that already synced to her device before she went offline, then that week's data and comparison values remain fully visible.

**FEAT-25.SPEC-002-AC-13:** Given Maya's household's history fails to load, when the trend region attempts to render, then the inline message "Couldn't load your check-in history. Try again." appears with a retry control.

**FEAT-25.SPEC-002-AC-14:** Given Maya's household's weekly_budget changes in Household Setup, when she next opens the trend content, then the budget-comparison line reflects the new figure.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 2 | 2 |
| States | 5 (hidden, showing, loading, error, offline) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Automation Spec: Weekly Check-In Cycle

## Overview

**Name:** Weekly Check-In Cycle
**ID:** FEAT-25.SPEC-003
**Type:** Automation
**Purpose:** Opens each week's check-in, marks an unanswered week as skipped, and locks the prior week's answer once the next one opens.
**Parent Feature:** FEAT-25 -- Weekly Waste & Spend Check-In

## Scope and Non-Goals

**In Scope:**
- Creating a new Waste & Spend Check-In record with status Offered for each household at the start of its new planning week
- Marking the previous week's record Skipped if it was never answered
- Locking the previous week's record if it was answered (setting its locked flag, without altering its data)
- Running once per household per week boundary, aligned with the household's own Weekly Plan week

**Non-Goals:**
- Sending a reminder or notification when a week opens or is marked Skipped -- excluded per feature-overview.md's Non-Goals: "a skipped week shows as a gap in the trend, with no follow-up reminder," keeping notifications limited to the weekly plan and nightly nudge
- Validating or accepting the content of any answer -- owned by FEAT-25.SPEC-005 (Check-In Validation & Access Rules); this automation only manages the record's status and lock state, never its waste_amount or spend values
- Setting or changing the household's one-time starting point -- owned by FEAT-25.SPEC-001 (Weekly Check-In Card); this automation never writes starting_point_waste or starting_point_spend
- Cascading or exporting check-in history when a household is deleted -- owned by FEAT-18 (Account & Data Management), per the Feature Breakdown Brief's Side-Effect Inventory

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A new week begins for a household | system (schedule-based) | Fires once per household at the boundary between one calendar week and the next, aligned with the same week boundary the household's Weekly Plan uses (FEAT-03, FEAT-23) | Household id; the household's most recent Waste & Spend Check-In record (if any) and its current status and locked flag |

## Processing Logic

1. At the start of each household's new planning week, check whether a Waste & Spend Check-In record already exists for the new week.
2. If none exists, create a new record for the new week: status Offered, waste_amount and spend empty, locked false.
3. If a record for the new week already exists (e.g., this cycle is re-evaluating after a prior partial run), skip step 2 -- no duplicate record is created.
4. Identify the record for the week immediately before the new week (the household's just-ended week).
5. If that record's status is Offered (no answer was ever submitted for it), set its status to Skipped.
6. If that record's status is Answered, set its locked flag to true; its waste_amount and spend values are left exactly as last submitted.
7. If that record's status is already Skipped, or its locked flag is already true, take no further action on it (idempotent re-run safeguard).
8. If no record exists for the week immediately before the new week (the household's very first week using this feature), skip steps 4-7 entirely -- there is nothing to close.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| New week opened | The new week has no existing record | Waste & Spend Check-In record created with status Offered | Household sees a fresh, unanswered check-in card (or First-Use variant, if no starting point exists) the next time it opens the plan | FEAT-25.SPEC-001 |
| Prior week skipped | Prior week's record status was Offered at cutover | Prior record's status set to Skipped | The trend view shows that week as a gap, with no answer and no notification | FEAT-25.SPEC-002 |
| Prior week locked | Prior week's record status was Answered at cutover | Prior record's locked flag set to true; its data is unchanged | The current card only reflects the new week; the prior week's answer becomes read-only if the household revisits it in the trend view | FEAT-25.SPEC-001, FEAT-25.SPEC-002 |
| No-action (already processed) | The new week's record already exists and the prior week's record is already Skipped or already locked | None | Nothing changes; the cycle's own idempotency safeguard prevents any duplicate effect | -- |
| First-ever week (no prior record) | Household has no record for the week before the new one | Only the create-new-week step runs | Household sees its very first check-in card, the First-Use variant | FEAT-25.SPEC-001 |
| Automation failure | Processing cannot complete for a household (e.g., a transient failure) | No partial state is left: either both the new-week create and the prior-week resolution complete together, or neither does | The household continues to see its previous state (last week's card, if still within its own window) until the cycle successfully retries; the household is never shown two simultaneously open, unlocked weeks nor left with no current week at all | FEAT-25.SPEC-001, FEAT-25.SPEC-002 |

## Data Model

**Reads:** Waste & Spend Check-In -- the household's most recent record and its status and locked flag, to determine what the cutover must do. Household -- to determine the household's own weekly cycle boundary, aligned with its Weekly Plan week.
**Creates:** Waste & Spend Check-In -- one new record per household per week, with status Offered.
**Updates:** Waste & Spend Check-In -- the previous week's record: status set to Skipped, or locked flag set to true.
**Deletes:** None.

## Business Rules

- Exactly one record per household per week is ever created by this cycle; if a record for the new week already exists, no duplicate is created (idempotency).
- The cutover that opens the new week and resolves the previous week happens as a single unit for a given household: the household is never shown two simultaneously open (unlocked, non-final) weeks, and the previous week's final status (Skipped or locked-Answered) is always resolved before or together with the new week opening.
- No reminder, notification, or nudge accompanies either the new week opening or the previous week being marked Skipped, per feature-overview.md's Non-Goals.
- This cycle never modifies a household's one-time starting point (starting_point_waste, starting_point_spend); once set, that data is untouched by every run of this automation.
- A locked record's waste_amount and spend are never altered by this automation -- locking only sets the locked flag; the data itself is exactly what the household last submitted (FEAT-25.SPEC-001).

## Edge Cases

- **Concurrent trigger firing (two cycle runs for the same household fire at effectively the same time)** -- Only one run's create/update actually lands; the other detects that the new week's record already exists and the prior week is already resolved, and performs no further action, per the idempotency business rule above.
- **Trigger fires while a previous run for the same household is still in flight** -- The second firing evaluates state only after the in-flight run completes, so it always sees the post-cutover state and takes no duplicate action; runs for different households never block one another.
- **Household has never answered any prior week (every week Offered, then Skipped)** -- The cycle continues to open a fresh Offered record every week regardless of the household's answer history; the First-Use, starting-point card variant (FEAT-25.SPEC-001) keeps appearing on every new week until the household's first answer sets a starting point.
- **Household's very first-ever week using this feature** -- No previous-week record exists to resolve; the cycle performs only the create-new-week step, per Processing Logic step 8.
- **A household is deleted (FEAT-18) between one cycle run and the next** -- The cycle takes no action for a deleted household; cascade or export behavior for its existing check-in history is FEAT-18's responsibility (feature-overview.md's Side-Effect Inventory), not this automation's.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-25.SPEC-001 (Weekly Check-In Card) | Affects (outbound) | Provides the fresh Offered (or First-Use-triggering) record the card renders, and locks the record the card can no longer edit |
| FEAT-25.SPEC-002 (Check-In Trend View) | Affects (outbound) | Provides the finalized (Skipped or locked-Answered) historical records the trend view reads |
| FEAT-25.SPEC-005 (Check-In Validation & Access Rules) | References (inbound) | The editable-until-next-week-opens window this automation enforces is defined there |

## Analytics and Success Signals

- **checkin_week_opened** (household_id, week) -- supports success-metrics.md: "Reported Food Waste and Spend Reduction" (the denominator against which how many offered weeks a household answers or skips is measured)
- **checkin_skipped** (household_id, week) -- supports success-metrics.md: "Reported Food Waste and Spend Reduction"
- **checkin_locked** (household_id, week) -- N/A -- this event is a state-transition bookkeeping signal with no direct bearing on the waste-and-spend reduction measure; recorded for completeness of the entity's lifecycle instrumentation

## Acceptance Criteria

**FEAT-25.SPEC-003-AC-01:** Given a household's new planning week begins and no record exists for it yet, when the cycle runs, then a new Waste & Spend Check-In record is created with status Offered.

**FEAT-25.SPEC-003-AC-02:** Given a household's previous week's record has status Offered (no answer was ever given), when the new week's cycle runs, then the previous week's record is set to Skipped.

**FEAT-25.SPEC-003-AC-03:** Given a household's previous week's record has status Answered, when the new week's cycle runs, then the previous week's record's locked flag is set to true and its waste_amount and spend values are unchanged.

**FEAT-25.SPEC-003-AC-04:** Given a household's new week's record already exists and its previous week's record is already Skipped, when the cycle runs again for the same boundary, then no data changes and no duplicate record is created.

**FEAT-25.SPEC-003-AC-05:** Given a household is using Plateful for its very first week, when the cycle runs, then only a new Offered record is created, since no previous week's record exists to resolve.

**FEAT-25.SPEC-003-AC-06:** Given two cycle runs fire for the same household at effectively the same time, when both attempt to open the new week and resolve the previous one, then only one run's changes land and the other takes no further action.

**FEAT-25.SPEC-003-AC-07:** Given a cycle run for a household is still in flight, when a second trigger fires for the same household, then the second run waits and, on evaluating state, finds the cutover already complete and takes no action.

**FEAT-25.SPEC-003-AC-08:** Given a household is deleted between one cycle run and the next, when the cycle would otherwise run for that household, then no action is taken, and any cascade or export behavior for its check-in history is handled by FEAT-18.

**FEAT-25.SPEC-003-AC-09:** Given the cycle fails partway through processing a household, when it is retried, then the household is left with neither two open weeks nor zero current weeks -- the retry completes the cutover as a single unit.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 6 | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Check-In Trend Calculation

## Overview

**Name:** Check-In Trend Calculation
**ID:** FEAT-25.SPEC-004
**Type:** Logic/Rule
**Purpose:** Derives the household's change against its starting point and against its weekly budget for display in the trend view.
**Parent Feature:** FEAT-25 -- Weekly Waste & Spend Check-In
**Governed Entity:** Waste & Spend Check-In (derived comparison values)

## Scope and Non-Goals

**In Scope:**
- The derivation logic for change_against_starting_point (waste and, where possible, spend) for each answered week
- The derivation logic for change_against_weekly_budget for each answered week that provided a spend value
- When each derivation runs, what it depends on, and what it produces when a dependency is missing
- Read-side visibility of the derived values, by role

**Non-Goals:**
- Field-level validation, requiredness, and the one-answer-per-week/latest-wins and editable-window rules for the Waste & Spend Check-In entity's stored fields -- owned by FEAT-25.SPEC-005 (Check-In Validation & Access Rules); this spec assumes the stored fields it reads are already valid and saved, and derives comparison values from them only
- Write access, submission, or correction of any stored field -- owned by FEAT-25.SPEC-001 (Weekly Check-In Card); this spec produces read-only, display-only values with no write-back to the entity
- Opening, skipping, or locking a week's record -- owned by FEAT-25.SPEC-003 (Weekly Check-In Cycle); this spec derives values only for weeks whose status that automation has already set

## Governed Entity

**Entity:** Waste & Spend Check-In
**Source:** Feature Dependency Map (feature-overview.md's Shared Context and Entity-Lifecycle Coverage Matrix)

| Field | Data Type | Description |
|-------|-----------|-------------|
| household | reference | The Household this record belongs to |
| week | date/period | The calendar week this record covers |
| waste_amount | enum (none \| a little \| a lot) | The household's answer for how much food was thrown away that week |
| spend | number, optional | The household's rough grocery spend that week, in its configured currency |
| starting_point_waste | enum (none \| a little \| a lot) | The household's one-time baseline: how much it typically threw away before Plateful |
| starting_point_spend | number, optional | The household's one-time baseline: what it typically spent on groceries before Plateful |
| status | enum (Offered \| Answered \| Skipped) | The week's lifecycle state |
| locked | boolean | Whether the week's record can still be edited |
| change_against_starting_point | derived (this spec) | The week's waste and spend compared against the household's starting point |
| change_against_weekly_budget | derived (this spec) | The week's spend compared against the household's current weekly_budget |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-25.SPEC-002 | Check-In Trend View | On render, recomputed for every displayed week each time the trend content is shown |

## Field Validation Rules

All input-side validation for the Waste & Spend Check-In entity's stored fields is governed by FEAT-25.SPEC-005 (Check-In Validation & Access Rules); each stored field is listed below with a pointer to that spec's rule, so this table remains a complete field inventory without duplicating its content. This spec introduces two additional fields, both system-computed with no user input, which carry their own rows below.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| household | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| week | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| waste_amount | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| spend | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| starting_point_waste | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| starting_point_spend | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| status | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| locked | Governed by FEAT-25.SPEC-005 -- see that spec's Field Validation Rules | -- | -- | -- | -- |
| change_against_starting_point | No validation beyond data type -- system-derived, never user input | Always | -- | -- | -- |
| change_against_weekly_budget | No validation beyond data type -- system-derived, never user input | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Starting-point comparison requires a starting point | waste_amount, starting_point_waste, spend, starting_point_spend | change_against_starting_point is computed for a given week only if the household's starting_point_waste exists; the waste half of the comparison always computes once a starting point exists (starting_point_waste is required at capture, per FEAT-25.SPEC-005), while the spend half computes only if both spend and starting_point_spend are present for that comparison | N/A -- not a rejectable input; an incomplete comparison simply omits the piece it cannot compute (FEAT-25.SPEC-002 shows its own placeholder text for the missing piece) |
| Budget comparison requires spend and a set budget | spend, Household.weekly_budget | change_against_weekly_budget is computed for a given week only if that week's spend was provided and the household's weekly_budget has been set (FEAT-01.SPEC-008) | N/A -- not a rejectable input; the comparison is simply omitted when either value is missing |
| Currency consistency before comparison | spend, starting_point_spend, Household.weekly_budget, Household.currency | All monetary values entering a comparison are read in the household's currently configured currency (FEAT-16.SPEC-001); a stored spend figure recorded under a previously configured currency is converted for display before any comparison runs, per XBR-11 (owned by FEAT-16) | N/A -- not a rejectable input; conversion is applied automatically, never surfaced as an error |

## Authorization Rules

This spec governs read-only, derived values with no write action of its own; the table below covers who may see the derived comparison values, mirroring the Access Matrix's Waste Check-In column that also governs FEAT-25.SPEC-001 and FEAT-25.SPEC-002.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View derived change-against-starting-point and change-against-budget values | Maya (Organiser) | Always | -- |
| View derived change-against-starting-point and change-against-budget values | Sam (Other Adult Member) | Always -- the same shared household values Maya sees | -- |
| View derived change-against-starting-point and change-against-budget values | Jordan (young kid profile, no login -- MVP) | Never | Values are never shown; a no-login profile has no access to any screen in the product |
| View derived change-against-starting-point and change-against-budget values | Jordan (older kid, limited login -- Later) | Never | Values are not shown in this role's view of the week's plan; the Access Matrix's Waste Check-In column is None for this role |
| View derived change-against-starting-point and change-against-budget values | Riley (Operator, support -- from v1) | Never | Values are not shown, even during an open Support Request's read-only view of the household's plan (FEAT-22); the Access Matrix's Waste Check-In column is None for Riley |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| change_against_starting_point (waste) | Rank each of the three waste_amount values on an ordinal scale where none = 0 (least waste), a little = 1, a lot = 2. For a given answered week, compute starting_point_waste's rank minus that week's waste_amount rank. A positive result is expressed as "reduced" (less waste than the starting point), zero as "no change," and a negative result as "increased." | Recomputed on every render of FEAT-25.SPEC-002, for every answered week that has waste_amount and the household has starting_point_waste | No -- always derived, never directly editable |
| change_against_starting_point (spend) | When both the week's spend and the household's starting_point_spend are present (both in the currently configured currency, converting first if needed per XBR-11), compute starting_point_spend minus that week's spend. A positive result is expressed as "spending less," zero as "no change," and a negative result as "spending more." | Recomputed on every render of FEAT-25.SPEC-002, for every answered week where both values exist | No -- always derived, never directly editable |
| change_against_weekly_budget | When the week's spend is present and Household.weekly_budget is set (both in the currently configured currency, converting first if needed per XBR-11), compute that week's spend minus Household.weekly_budget. A result of exactly zero or negative is expressed as "at or under budget" (stating the exact amount under, or "right at budget" for exactly zero); a positive result is expressed as "over budget" by the exact amount. | Recomputed on every render of FEAT-25.SPEC-002, for every answered week where spend is present and weekly_budget is set | No -- always derived, never directly editable |

## Business Rules

- Derivation is display-only: it never writes back to the Waste & Spend Check-In record or to Household; no stored field changes as a result of computing these values.
- A week with no spend value is excluded from change_against_weekly_budget and from the spend half of change_against_starting_point for that week; its waste half of change_against_starting_point still computes independently, since waste_amount and spend are independently optional (FEAT-25.SPEC-005).
- Until the household has a starting point, no change_against_starting_point value is computed for any week; FEAT-25.SPEC-002 shows its own no-starting-point placeholder content instead, consistent with the Feature Breakdown Brief's "Single evolving card" shared UI pattern.
- A Skipped week contributes no change values of any kind -- there is no waste_amount or spend to compare for a week with no answer -- and appears in the trend only as a gap, per feature-overview.md's Non-Goals.
- All monetary comparisons operate in the household's currently configured currency; a later currency change converts existing figures for display before any comparison runs, per XBR-11 (owned by FEAT-16), so no comparison ever mixes two currencies' raw numbers.

## Edge Cases

- **Household's starting_point_spend was left blank at capture, but later weeks provide spend** -- The waste half of change_against_starting_point still computes normally (starting_point_waste is always required at capture); the spend half of that same comparison has no baseline to compare against and is omitted, while change_against_weekly_budget still computes independently for those weeks (it depends only on spend and weekly_budget, not on starting_point_spend).
- **Household's weekly_budget has never been set** -- change_against_weekly_budget is never computed for any week until weekly_budget is set (FEAT-01); change_against_starting_point is unaffected, since it does not depend on weekly_budget.
- **This week's waste_amount ties the starting point exactly (e.g., started "A little," still "A little")** -- The comparison is explicitly "no change," never silently treated as "reduced" or omitted.
- **Spend exactly equal to weekly_budget** -- Treated as "at budget" (a difference of exactly zero), consistent with the product's own "at or under budget" framing (success-metrics.md).
- **Currency changes between when a week's spend was recorded and when the trend is next viewed** -- The stored spend figure is converted to the currently configured currency before change_against_weekly_budget or the spend half of change_against_starting_point runs, per XBR-11; the two values entering any single comparison are never in different currencies.
- **Household sets its weekly_budget for the first time after several weeks were already answered** -- change_against_weekly_budget begins appearing only for weeks answered from that point forward; earlier weeks show no budget comparison, since no budget value existed for them at the time (FEAT-25.SPEC-002's Edge Cases).

## Acceptance Criteria

**FEAT-25.SPEC-004-AC-01:** Given Maya's household started at "A lot" and this week's answer is "A little," when the trend view renders, then change_against_starting_point states the waste change as "reduced."

**FEAT-25.SPEC-004-AC-02:** Given Sam's household started at "None" and this week's answer is also "None," when the trend view renders, then change_against_starting_point states the waste change as "no change."

**FEAT-25.SPEC-004-AC-03:** Given a household started at "A little" and this week's answer is "A lot," when the trend view renders, then change_against_starting_point states the waste change as "increased."

**FEAT-25.SPEC-004-AC-04:** Given a household provided both a starting typical spend and this week's spend, and this week's spend is lower, when the trend view renders, then change_against_starting_point's spend comparison states the household is spending less.

**FEAT-25.SPEC-004-AC-05:** Given a household's starting_point_spend was left blank at capture, when the trend view renders for a week that does provide spend, then no spend comparison against the starting point is shown, while the waste comparison still renders normally.

**FEAT-25.SPEC-004-AC-06:** Given a household's weekly_budget is set and this week's spend is under it, when the trend view renders, then change_against_weekly_budget states the exact amount under budget.

**FEAT-25.SPEC-004-AC-07:** Given a household's weekly_budget is set and this week's spend exactly equals it, when the trend view renders, then change_against_weekly_budget states the week is right at budget.

**FEAT-25.SPEC-004-AC-08:** Given a household's weekly_budget is set and this week's spend exceeds it, when the trend view renders, then change_against_weekly_budget states the exact amount over budget.

**FEAT-25.SPEC-004-AC-09:** Given a household's weekly_budget has never been set, when the trend view renders, then no change_against_weekly_budget value is produced for any week.

**FEAT-25.SPEC-004-AC-10:** Given a week's spend was left blank, when the trend view renders, then no change_against_weekly_budget value is produced for that week, though its waste comparison still renders if a starting point exists.

**FEAT-25.SPEC-004-AC-11:** Given a household has no starting point yet, when the trend view would otherwise render, then no change_against_starting_point value is produced for any week.

**FEAT-25.SPEC-004-AC-12:** Given a week's spend was recorded under a currency the household has since changed, when the trend view renders, then the stored spend figure is converted to the currently configured currency before any comparison is computed.

**FEAT-25.SPEC-004-AC-13:** Given Riley is viewing a household's plan through an open Support Request (FEAT-22), when the plan renders, then no derived change value of any kind is shown anywhere in Riley's view.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 10 | 10 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Check-In Validation & Access Rules

## Overview

**Name:** Check-In Validation & Access Rules
**ID:** FEAT-25.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs the one-answer-per-week/latest-wins rule, spend validation, the editable window, and who may view or answer the check-in.
**Parent Feature:** FEAT-25 -- Weekly Waste & Spend Check-In
**Governed Entity:** Waste & Spend Check-In

## Scope and Non-Goals

**In Scope:**
- Field-level validation rules for every field of the Waste & Spend Check-In entity
- The one-answer-per-week, latest-write-wins resolution when more than one adult submits for the same week
- The editable-until-next-week-opens window and what happens to a submission after that window closes
- Authorization: who may view or answer the check-in, per role, and the exact denied experience for every role that cannot
- Default values and derivations for the entity's own stored fields (status, locked)

**Non-Goals:**
- Computing change_against_starting_point or change_against_weekly_budget -- owned by FEAT-25.SPEC-004 (Check-In Trend Calculation); this spec governs the raw stored fields only
- Deciding when a new week opens or when the previous week is marked Skipped or locked -- owned by FEAT-25.SPEC-003 (Weekly Check-In Cycle); this spec defines the rule that a locked week cannot be edited, while that automation is the one that sets the lock
- Deletion or archival of any check-in record -- excluded per scope-boundaries.md SC-18: every past check-in answer is kept for the life of the household account and remains available after a downgrade to the free tier; this spec defines no delete action because the product defines none

## Governed Entity

**Entity:** Waste & Spend Check-In
**Source:** Feature Dependency Map (feature-overview.md's Shared Context)

| Field | Data Type | Description |
|-------|-----------|-------------|
| household | reference | The Household this record belongs to |
| week | date/period | The calendar week this record covers |
| waste_amount | enum (none \| a little \| a lot) | The household's answer for how much food was thrown away that week |
| spend | number, optional | The household's rough grocery spend that week, in its configured currency |
| starting_point_waste | enum (none \| a little \| a lot) | The household's one-time baseline: how much it typically threw away before Plateful |
| starting_point_spend | number, optional | The household's one-time baseline: what it typically spent on groceries before Plateful |
| status | enum (Offered \| Answered \| Skipped) | The week's lifecycle state |
| locked | boolean | Whether the week's record can still be edited |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-25.SPEC-001 | Weekly Check-In Card | On field blur (spend, typical spend) and on save (starting-point form); authorization on screen entry and on every submission |
| FEAT-25.SPEC-002 | Check-In Trend View | Authorization on screen entry |
| FEAT-25.SPEC-003 | Weekly Check-In Cycle | Status and locked-flag transitions during processing |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| household | No validation beyond data type -- system-assigned from the signed-in adult's household membership, never user input | Always | -- | -- | -- |
| week | No validation beyond data type -- system-assigned by FEAT-25.SPEC-003 when the week's record opens, never user input | Always | -- | -- | -- |
| waste_amount | Must be one of "none," "a little," or "a lot" | Required only if the adult chooses to answer this week's question -- the question itself remains optional per week | On submit (tapping an option saves it immediately) | "Choose an option for how much was thrown away this week." | Yes |
| spend | Must be a positive number (greater than zero) in the household's configured currency; no fixed upper bound is defined by the product, consistent with Household.weekly_budget's own unlimited-positive-amount rule (FEAT-01.SPEC-008) | Optional -- may be left blank regardless of whether waste_amount is answered | On blur | "Enter an amount greater than zero, or leave it blank." | Yes (only when a non-blank value fails the rule; blank is always valid) |
| starting_point_waste | Must be one of "none," "a little," or "a lot" | Required only at the household's first-ever answered check-in (no starting point exists yet for this household) | On submit of the first-use form | "Choose an option for how much your household typically threw away before Plateful." | Yes |
| starting_point_spend | Must be a positive number (greater than zero) in the household's configured currency | Optional, even during starting-point capture -- same optionality as the recurring week's spend field | On blur (first-use form) | "Enter an amount greater than zero, or leave it blank." | Yes (only when a non-blank value fails the rule) |
| status | No validation beyond data type -- system-derived by FEAT-25.SPEC-003 (Offered/Answered/Skipped transitions), never directly settable by a user | Always | -- | -- | -- |
| locked | No validation beyond data type -- system-derived by FEAT-25.SPEC-003, never directly settable by a user | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Starting point captured once | starting_point_waste, starting_point_spend | Once a household's Waste & Spend Check-In history includes a record with a starting point set, no later submission ever includes or alters starting_point_waste or starting_point_spend | N/A -- the fields are simply not presented after the starting point exists; this is not a rejected submission, since FEAT-25.SPEC-001 never offers the starting-point form again |
| Submission locked out after cutover | waste_amount, spend, locked | A submission for a given week is accepted only while that week's record has locked = false; once FEAT-25.SPEC-003 sets locked = true, no further submission for that week is accepted, regardless of who attempts it | "This week's check-in has closed." |
| Latest submission replaces the whole week's answer | waste_amount, spend | When a second adult submits for the same open week, the latest submission replaces waste_amount and spend together as a single unit -- never merged field by field with the earlier submission | N/A -- not an error; this is the intended one-answer-per-week/latest-wins resolution, silently applied |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the check-in card and trend content | Maya (Organiser) | Always | -- |
| View the check-in card and trend content | Sam (Other Adult Member) | Always -- views the same shared household record Maya sees | -- |
| View the check-in card and trend content | Jordan (young kid profile, no login -- MVP) | Never | Card and trend content are not shown; a no-login profile has no access to any screen in the product |
| View the check-in card and trend content | Jordan (older kid, limited login -- Later) | Never | Card and trend content are not shown in this role's view of the week's plan; the Access Matrix's Waste Check-In column is None for this role |
| View the check-in card and trend content | Riley (Operator, support -- from v1) | Never | Card and trend content are not shown, even during an open Support Request's read-only view of the household's plan (FEAT-22); the Access Matrix's Waste Check-In column is None for Riley, keeping this personal household data outside operator visibility |
| Submit or resubmit the week's answer (waste_amount, spend) | Maya (Organiser) | Always, while the week's record has locked = false | Rejected with "This week's check-in has closed." once locked = true |
| Submit or resubmit the week's answer (waste_amount, spend) | Sam (Other Adult Member) | Always, while the week's record has locked = false -- per the Access Matrix's Full entitlement, matching Maya's, Sam's submission is accepted outright against the shared weekly record; it is not gated behind Maya's approval, and if Maya also submits the same week, whichever submission lands last is the one kept (Cross-Field Rules, above) | Rejected with "This week's check-in has closed." once locked = true |
| Submit or resubmit the week's answer | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role; the action is unreachable |
| Submit or resubmit the week's answer | Jordan (older kid, limited login -- Later) | Never | Not offered in this role's app; the Access Matrix's Waste Check-In column is None for this role |
| Submit or resubmit the week's answer | Riley (Operator, support -- from v1) | Never | The read-only support role has no create or update entitlement on this entity anywhere in the product; the action is not offered at all |
| Set the household's one-time starting point (starting_point_waste, starting_point_spend) | Maya (Organiser) | Only at the household's first-ever answered check-in (no starting point set yet) | Not offered again once a starting point exists (Cross-Field Rules, above) |
| Set the household's one-time starting point | Sam (Other Adult Member) | Only at the household's first-ever answered check-in (no starting point set yet) -- Full entitlement, same basis as the weekly answer above | Not offered again once a starting point exists |
| Set the household's one-time starting point | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role; the action is unreachable |
| Set the household's one-time starting point | Jordan (older kid, limited login -- Later) | Never | Not offered in this role's app |
| Set the household's one-time starting point | Riley (Operator, support -- from v1) | Never | The read-only support role has no create or update entitlement on this entity |
| Delete or archive a check-in record | All roles | Never -- no role may delete or archive any Waste & Spend Check-In record | The action does not exist in the product; every past record is retained for the life of the household account per scope-boundaries.md SC-18 |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| household | Derived from the signed-in adult's own household membership | On create | No |
| week | Derived as the calendar week the record covers, assigned when FEAT-25.SPEC-003 opens the record | On create | No |
| status | Defaults to "Offered" when FEAT-25.SPEC-003 creates the record; becomes "Answered" the moment an adult submits waste_amount; becomes "Skipped" if FEAT-25.SPEC-003's cutover finds it still Offered | On create (Offered); on submit (Answered); on automation cutover (Skipped) | No direct override -- status is always system-derived from the actions above, never directly settable |
| locked | Defaults to false when the record is created (Offered) | On create; set to true by FEAT-25.SPEC-003 at the following week's cutover | No |

## Business Rules

- One answer per household per week: the latest submission from any adult with access to the entity replaces the entire prior submission for that week (waste_amount and spend together), never merged field by field.
- A week's record remains editable only until the following week's check-in opens (FEAT-25.SPEC-003); once locked, no further submission is accepted for that week.
- The starting point (starting_point_waste, starting_point_spend) is captured exactly once, at the household's first answered check-in, and is never altered afterward by any rule in this spec -- no editing path exists for it, per feature-overview.md's Key Capabilities and Non-Goals.
- No delete or archive action exists for any Waste & Spend Check-In record; every past record is retained for the life of the household account and remains visible after a downgrade to the free tier (scope-boundaries.md SC-18).
- Spend figures are entered and displayed in the household's currently configured currency (FEAT-16.SPEC-001); a later currency change converts historical spend figures for display (XBR-11, owned by FEAT-16) without this spec re-validating already-saved amounts.

## Edge Cases

- **Household's very first check-in is left unanswered** -- No starting point is set; the following week's card again renders the first-use, starting-point form (FEAT-25.SPEC-001), since starting_point_waste is still required and no reminder is sent (feature-overview.md's Non-Goals).
- **Two adults submit different answers for the same week within moments of each other** -- The submission that reaches the record last is kept in full; the adult whose submission was superseded sees no error, only the other's answer the next time the card loads, per the Cross-Field Rules' latest-wins resolution.
- **An adult attempts to submit after the week's record has already been locked** -- The submission is rejected with "This week's check-in has closed."; the card shows the locked answer read-only (FEAT-25.SPEC-001's States).
- **Spend entered as exactly the smallest currency unit above zero** -- Passes validation; any amount strictly greater than zero is valid, with no minimum beyond that.
- **Spend field left blank while waste_amount is answered** -- Valid; spend is optional independently of whether waste_amount was answered, and the record saves with spend absent.
- **Starting point's spend field left blank while starting point's waste is answered** -- Valid, same optionality as the recurring week's spend field.
- **Sam submits an answer, then Maya submits a different answer for the same still-open week** -- Maya's submission (the later one) is kept in full; this is not a denial of Sam's access, since his Full entitlement permits the submission itself -- it is simply superseded by a later submission, consistent with the one-answer-per-week/latest-wins rule applying identically regardless of which adult submits first or last.

## Acceptance Criteria

**FEAT-25.SPEC-005-AC-01:** Given Maya selects "A little" for this week's waste, when she saves, then waste_amount is recorded as "a little" with no error.

**FEAT-25.SPEC-005-AC-02:** Given Sam enters a spend amount of zero and blurs the field, then the field shows "Enter an amount greater than zero, or leave it blank." and the value is not saved.

**FEAT-25.SPEC-005-AC-03:** Given Sam leaves the spend field blank and saves the waste answer, then the record saves successfully with spend absent.

**FEAT-25.SPEC-005-AC-04:** Given Maya is on her household's first-ever check-in and selects "A lot" for typical waste but leaves typical spend blank, when she saves, then starting_point_waste is recorded as "a lot," starting_point_spend remains unset, and no error is shown.

**FEAT-25.SPEC-005-AC-05:** Given Maya is on her household's first-ever check-in and taps Save without selecting a typical-waste option, then the error "Choose an option for how much your household typically threw away before Plateful." appears and the save does not proceed.

**FEAT-25.SPEC-005-AC-06:** Given a household already has a starting point set, when Sam opens the check-in card, then no starting-point form is offered to him.

**FEAT-25.SPEC-005-AC-07:** Given Maya submitted this week's answer and Sam later submits a different answer for the same still-open week, when the record is next read, then Sam's submission (the later one) is the one shown, with no error to either adult.

**FEAT-25.SPEC-005-AC-08:** Given a week's record has locked = true, when Maya attempts to submit an answer for it, then the submission is rejected with "This week's check-in has closed."

**FEAT-25.SPEC-005-AC-09:** Given a week's record has locked = false and status Answered, when Sam resubmits a different waste_amount, then the resubmission succeeds and replaces the prior answer.

**FEAT-25.SPEC-005-AC-10:** Given Maya (Organiser) opens the week's plan, when the check-in card renders, then she can both view and act on it with no restriction.

**FEAT-25.SPEC-005-AC-11:** Given Sam (Other Adult Member) opens the week's plan, when the check-in card renders, then he can view the full shared record and submit or resubmit an answer, per his Full entitlement.

**FEAT-25.SPEC-005-AC-12:** Given Jordan is a young kid profile with no login, then no view of or action on this entity is reachable from any device of Jordan's.

**FEAT-25.SPEC-005-AC-13:** Given the older-kid limited login (Later) opens the week's plan, when they look for the check-in card, then it is not shown and no submission action is offered to that role.

**FEAT-25.SPEC-005-AC-14:** Given Riley is viewing a household's plan through an open Support Request (FEAT-22), when the plan renders, then no check-in content is shown and no submission action exists for Riley anywhere in the product.

**FEAT-25.SPEC-005-AC-15:** Given any role attempts to find a delete or archive action for a check-in record, then no such action exists anywhere in the product for any role.

**FEAT-25.SPEC-005-AC-16:** Given a household's first check-in is left unanswered, when the following week's card renders, then it still shows the first-use, starting-point form, since starting_point_waste remains unset.

**FEAT-25.SPEC-005-AC-17:** Given Maya's household record has status Offered, when she submits a waste answer, then status becomes Answered.

**FEAT-25.SPEC-005-AC-18:** Given a household's record has status Offered and the following week's cycle runs with no answer ever submitted, then status becomes Skipped, per FEAT-25.SPEC-003.

**FEAT-25.SPEC-005-AC-19:** Given a household's record has status Answered and the following week's cycle runs, then locked becomes true and waste_amount and spend remain exactly as last submitted.

**FEAT-25.SPEC-005-AC-20:** Given Maya enters a spend amount at the smallest unit above zero, when she blurs the field, then the value passes validation and saves.

**FEAT-25.SPEC-005-AC-21:** Given Sam is on the starting-point form and leaves typical spend blank but selects a typical-waste option, when he saves, then the submission succeeds with starting_point_spend unset.

**FEAT-25.SPEC-005-AC-22:** Given a household's record already carries a starting point, when any adult opens a later week's check-in, then the submission form never re-offers the starting-point fields.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 16 | 16 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |
