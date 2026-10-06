---
document_type: spec
spec_type: screen
spec_id: FEAT-25.SPEC-001
spec_name: Weekly Check-In Card
spec_slug: weekly-check-in-card
parent_feature: FEAT-25
parent_feature_name: Weekly Waste & Spend Check-In
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 18
---

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
