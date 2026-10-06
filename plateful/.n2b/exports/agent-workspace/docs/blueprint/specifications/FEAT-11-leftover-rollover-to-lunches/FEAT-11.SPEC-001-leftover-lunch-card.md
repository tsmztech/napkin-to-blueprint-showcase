---
document_type: spec
spec_type: screen
spec_id: FEAT-11.SPEC-001
spec_name: Leftover Lunch Card
spec_slug: leftover-lunch-card
parent_feature: FEAT-11
parent_feature_name: Leftover Rollover to Lunches
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Leftover Lunch Card

## Overview

**Name:** Leftover Lunch Card
**ID:** FEAT-11.SPEC-001
**Type:** Screen
**Purpose:** Displays the leftover-lunch suggestion attached to a day within the Weekly Plan, and lets Maya or Sam confirm it happened or skip it.
**Parent Feature:** FEAT-11 -- Leftover Rollover to Lunches

## Scope and Non-Goals

**In Scope:**
- Displaying a Suggested, Eaten, or Skipped leftover-lunch card on the day it is attached to
- Confirm and Skip actions on a Suggested leftover lunch
- Reflecting an update or removal of the card when the linked source dinner changes (FEAT-11.SPEC-004)

**Non-Goals:**
- Rescheduling a leftover lunch to a different day, or creating one manually without a source dinner -- not modeled anywhere in the product: product-features.md's Validation & Limits fixes the link to exactly one source dinner and one system-computed following day, so this card offers no "change day" or "add lunch" control
- Determining which dinners are leftover-producing and which following day to suggest -- owned by FEAT-11.SPEC-003 (Leftover Lunch Eligibility & Linking Rule); this screen only displays what that rule (via FEAT-11.SPEC-002) has already produced
- Sending a notification when the suggestion is created, confirmed, or skipped -- excluded per this feature's own Communications field (product-features.md): the suggestion appears within the plan itself and no separate notification is sent for any of its three outcomes
- Detailed leftover quantity or expiry tracking -- excluded per scope-boundaries.md SC-11: this is a simple yes/no suggestion, not a tracked quantity or expiry date

## Entry Points

{This screen has no standalone entry point -- the card is embedded within an AI-generated Weekly Plan (FEAT-03) wherever that plan is shown, on the day the suggestion is attached to. It never appears within a purely free-tier, fully hand-built week, since a leftover-lunch suggestion is only ever created by FEAT-11.SPEC-002, which fires solely off AI Weekly Dinner Plan Generation (FEAT-03) completing (see FEAT-11.SPEC-002's Non-Goals; product-features.md's FEAT-11 Interactions field names FEAT-03, not FEAT-23, as the dependency this feature extends). Note on the Brief: feature-overview.md's Shared UI Patterns describes this card rendering "whether the plan is AI-generated (FEAT-03) or manually built (FEAT-23)" -- that phrasing is imprecise about origin; the mechanism it is actually pointing at is FEAT-23 being used to hand-edit one night's slot inside an already AI-generated plan (row 2 below), not to build a free-tier week from nothing. This spec does not modify the Brief; it records the discrepancy here per the producer's out-of-scope guidance.}

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-03.SPEC-001 (Weekly Plan View) | Household member views the AI-generated plan on a day carrying a leftover-lunch suggestion | The linked leftover-lunch Planned Meal for that day: source dinner reference, night, status |
| FEAT-23 (Manual Weekly Planning) | A paid household uses FEAT-23 to hand-pick or change one night's dinner within an already AI-generated plan (not to build a free-tier week from scratch, which never carries a leftover-lunch card); the household member then views that same AI-generated plan on a day still carrying a leftover-lunch card | Same as above -- if the edited night had a linked leftover-lunch suggestion, FEAT-11.SPEC-004 re-evaluates it first; this card always renders within the FEAT-03-originated plan, never a standalone manually built week |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full card, wherever it is attached in the plan | Confirm or Skip | -- |
| Sam (Other Adult Member) | Full card | Confirm or Skip -- Sam's View access to the Weekly Plan still permits this, since marking a leftover lunch is a status update on the plan, not a change to it (Access Matrix notes, user-persona.md) | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this role; there is no path into the product to reach this card |
| Jordan (older kid, limited login -- Later) | Full card | Not shown -- Confirm and Skip controls are hidden; a direct attempt is not offered, consistent with this role's View-only access to the Weekly Plan | -- |
| Riley (Operator, support) | Full card, within the read-only support view | Not shown -- Confirm and Skip controls are hidden, consistent with Riley's View access and the strictly read-only nature of support access (XBR-14) | -- |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on the current Weekly Plan, not directly on this card |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no in-progress Confirm/Skip action exists on this card to lose, since each action completes in a single tap |

## Layout and Content

The Leftover Lunch Card is a compact element embedded within the Weekly Plan's day view, appearing below or alongside the day's dinner entry on any day carrying a leftover-lunch suggestion. Days with no suggestion show no card and no placeholder -- this is the normal case, not an omission.

**Card contents, top to bottom:**
- A label identifying the card as a leftover lunch (e.g., "Leftover lunch"), distinguishing it from the day's dinner entry
- The source dinner's recipe name (display-only text, e.g., "From: Tuesday's Chicken Casserole") -- not interactive; the source recipe is not opened from this card
- The current status, shown only when the leftover lunch is Eaten or Skipped (a completed card shows its resolved state instead of action controls)
- Confirm and Skip actions, shown side by side, only while the leftover lunch is in Suggested status

The card carries no dedicated header or footer of its own -- it is a content block within whichever Weekly Plan screen (FEAT-03.SPEC-001 or FEAT-23) hosts it, and inherits that screen's overall layout.

### Responsive Behavior

- **Compact breakpoint:** Confirm and Skip appear as two full-width, stacked or side-by-side buttons (whichever the hosting Weekly Plan's day-entry pattern uses for its own action pairs) within the card's fixed width inside the day entry.
- **Medium size class and above:** The card's width follows the hosting Weekly Plan's day-entry width; Confirm and Skip remain side by side with no structural change.
- Uniform scaling, no structural change beyond the width inherited from the hosting screen's day entry.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Source dinner recipe name | -- | Display-only, non-interactive | None | None -- text label only |
| Confirm button (Suggested status only) | Tap | Sets the leftover lunch's status to Eaten | Card switches from action state to resolved state showing "Eaten" | Card updates in place; no toast or follow-up prompt |
| Skip button (Suggested status only) | Tap | Sets the leftover lunch's status to Skipped | Card switches from action state to resolved state showing "Skipped" | Card updates in place; no toast, no penalty, no follow-up prompt |
| Confirm or Skip (while a save from another action is in flight) | Tap | No action -- debounced | None | Button remains in its brief loading state until the in-flight save resolves |
| Confirm or Skip (server rejects the write while online) | Tap | Status write is sent to the server and rejected | See States: Save Rejected -- Withdrawn/Re-linked (race) or Save Rejected -- Server Error | Withdrawn/re-linked race: card silently refreshes with no message shown; server error: card returns to Suggested with the inline message "Couldn't save. Try again." |

### Accessibility Notes

- **Focus order:** Source dinner recipe name (announced as static text) -> Confirm -> Skip, in that order, within the card.
- **Status announcements:** When Confirm or Skip completes, the card's new resolved state ("Eaten" or "Skipped") is announced to assistive technology as it replaces the action controls.
- **Keyboard alternatives:** Confirm and Skip are both reachable and operable by keyboard; there are no pointer-only gestures on this card.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Suggested (default, actionable) | Card shows the source dinner name and Confirm/Skip buttons | A Suggested leftover lunch is attached to the viewed day (FEAT-11.SPEC-002 created it) | Maya or Sam taps Confirm or Skip |
| Eaten | Card shows the source dinner name and an "Eaten" resolved indicator; no action buttons | Confirm is tapped | None -- this is a terminal, retained state (scope-boundaries.md SC-18) |
| Skipped | Card shows the source dinner name and a "Skipped" resolved indicator; no action buttons | Skip is tapped | None -- this is a terminal, retained state |
| No suggestion (not rendered) | N/A -- no card appears on this day; this is the normal case, not an empty state requiring explanation (product-features.md, States) | The day has no leftover lunch attached, or its linked source dinner was withdrawn (FEAT-11.SPEC-004) | A future generation cycle attaches a new Suggested leftover lunch to the day |
| Loading | N/A -- the suggestion is generated as part of plan generation (FEAT-11.SPEC-002), with no separate loading step for this card | -- | -- |
| Error | N/A -- a failure to compute the suggestion simply omits the card rather than showing an error (product-features.md, States) | -- | -- |
| Offline/Degraded | The card remains fully visible with its current status; Confirm and Skip remain tappable, and the status change is queued locally and synced automatically once connectivity returns, consistent with the product's general offline handling (ASMP-25) | Connectivity is lost while the card is visible or while an action is attempted | Connectivity is restored -- any queued status change submits automatically with the standard in-place update |
| Save Rejected -- Withdrawn/Re-linked (race) | No error dialog or toast is shown to the household member who tapped Confirm/Skip; the card silently re-fetches the record's current state and either disappears (record was withdrawn) or shows the newly re-linked source dinner's name (record was re-linked) | The Confirm/Skip write reaches the server while online, but FEAT-11.SPEC-004 withdrew or re-linked this same record moments earlier because its source dinner was swapped or cleared -- the write is rejected because the record it targeted no longer exists in its prior form | The refreshed read completes; the card either has disappeared or reflects the new source dinner, and no retry is offered since there is nothing left in Suggested status to confirm or skip |
| Save Rejected -- Server Error | The card returns to Suggested state with the inline message "Couldn't save. Try again." shown beneath Confirm and Skip; both buttons remain enabled and tappable | The Confirm/Skip write reaches the server while online and is rejected for a reason other than the withdrawn/re-linked race (e.g., a transient backend error) | Household member taps Confirm or Skip again; the message clears once a subsequent attempt succeeds, or a further failure re-shows the same message |

## Validation Rules

Validation governed by FEAT-11.SPEC-003 (Leftover Lunch Eligibility & Linking Rule). This screen has no user-entered fields to validate; the source-dinner link and following-day placement are entirely system-computed per that spec, and Confirm/Skip carry no validation beyond the authorization check in FEAT-11.SPEC-003's Authorization Rules.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Confirm tap | None -- card updates in place within the hosting Weekly Plan screen | -- |
| Skip tap | None -- card updates in place within the hosting Weekly Plan screen | -- |

## Data Model

**Creates:** None -- this screen never creates a leftover-lunch Planned Meal.
**Reads:** Planned Meal (leftover-lunch sub-type) -- meal_kind, linked source dinner (used to read that dinner's recipe name for display), night, status. Weekly Plan -- read only to locate which day's plan the card is attached to.
**Updates:** Planned Meal (leftover-lunch sub-type) -- status only, set to Eaten (Confirm) or Skipped (Skip). No other field on this record is writable from this screen.
**Deletes:** None -- withdrawal is owned entirely by FEAT-11.SPEC-004, never by this screen.

## Business Rules

- XBR-10: this card only ever displays a leftover lunch already linked to exactly one source dinner and placed no more than two days after it; the link and placement are never edited here.
- Confirm and Skip are each a one-tap, terminal action -- once a leftover lunch is Eaten or Skipped, this screen offers no further action on it (product-features.md: "no penalty or follow-up prompt" on skip; scope-boundaries.md SC-18 retains the outcome permanently).
- The card's presence and content are entirely produced by FEAT-11.SPEC-002 (creation) and FEAT-11.SPEC-003 (eligibility and day placement); this screen never independently decides whether a day should show a card.
- If the linked source dinner changes after the card is shown, FEAT-11.SPEC-004 updates or removes the underlying record and this screen reflects the result on its next read -- it does not itself detect or process the source change.
- A Confirm/Skip write rejected by the server while online never leaves the card showing a status that was not actually saved: it either refreshes silently to the record's true current state (withdrawn or re-linked, per FEAT-11.SPEC-004) or returns to Suggested with a retry-able error, and in neither case does the tapped action's target status appear until the server confirms it.

## Edge Cases

- **Maya and Sam tap Confirm and Skip on the same leftover lunch at nearly the same time** -- Last write wins; no conflict error is shown to either of them, consistent with the dependency map's Planned Meal Contention note that leftover status updates resolve last-write-wins.
- **The card's linked source dinner is swapped or removed while the card is on screen** -- FEAT-11.SPEC-004 updates the link (the card now shows the new source dinner's name) or withdraws the record entirely (the card disappears on the next read); no error is shown to the viewer either way.
- **Household member taps Confirm twice rapidly** -- The second tap is ignored while the first is in flight (debounced); the card settles into the Eaten state once.
- **Household member navigates away mid-tap and returns** -- The card re-fetches its current state and shows whatever status the action ultimately reached (Eaten, Skipped, or still Suggested if the tap never completed).
- **A day carries no leftover-lunch suggestion at all** -- No card renders and no placeholder is shown; this is the normal case for most days, not an error or empty state (product-features.md, States).
- **Maya taps Confirm at the same moment FEAT-11.SPEC-004 withdraws or re-links the same record because its source dinner was just swapped or cleared** -- Maya's write is rejected as targeting a record that changed underneath it; the card silently refreshes to the withdrawn (card disappears) or re-linked (new source dinner shown) state, with no error shown to Maya and no retry offered, since there is nothing left in Suggested status to act on.
- **Sam's Skip write reaches the server while online and is rejected for a reason other than the withdrawal race (e.g., a transient backend error)** -- The card returns to Suggested state showing the inline message "Couldn't save. Try again."; Confirm and Skip remain tappable and Sam's retry is treated as a fresh action.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-11.SPEC-002 (Leftover Lunch Suggestion Generation) | Triggered by (inbound) | Creates the Suggested leftover-lunch record this screen displays |
| FEAT-11.SPEC-003 (Leftover Lunch Eligibility & Linking Rule) | References (inbound) | Governs the source link, following-day placement, and the Confirm/Skip authorization rules this screen enforces |
| FEAT-11.SPEC-004 (Leftover Lunch Withdrawal on Source Change) | Affects (inbound) | Updates or removes the record this card displays when the source dinner changes |
| FEAT-03.SPEC-001 (Weekly Plan View) | Navigation (inbound) | Hosts this card within the AI-generated plan's day view |
| FEAT-23 (Manual Weekly Planning) | Cross-feature (inbound) | Used to hand-pick or change an individual night within an already AI-generated plan; editing a night with a linked leftover-lunch suggestion triggers FEAT-11.SPEC-004's re-evaluation before this card next renders -- FEAT-23 never hosts this card within a standalone, fully hand-built free-tier week |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| leftover_lunch_confirmed | source dinner night, following day, member role (Maya / Sam) | Confirm is tapped and the status successfully updates to Eaten | supports success-metrics.md: "Reported Food Waste and Spend Reduction" |
| leftover_lunch_skipped | source dinner night, following day, member role (Maya / Sam) | Skip is tapped and the status successfully updates to Skipped | supports success-metrics.md: "Reported Food Waste and Spend Reduction" |

## Acceptance Criteria

**FEAT-11.SPEC-001-AC-01:** Given Maya is viewing the Weekly Plan on a day with a Suggested leftover lunch, when she taps Confirm, then the card's status updates to Eaten and no further prompt appears.

**FEAT-11.SPEC-001-AC-02:** Given Sam is viewing the Weekly Plan on a day with a Suggested leftover lunch, when he taps Skip, then the card's status updates to Skipped with no penalty and no follow-up prompt, even though Sam's Weekly Plan access is View.

**FEAT-11.SPEC-001-AC-03:** Given a leftover lunch has already been marked Eaten, when any household member views the day, then the card shows the resolved "Eaten" state with no Confirm or Skip controls.

**FEAT-11.SPEC-001-AC-04:** Given Jordan (older kid, limited login) views a day with a Suggested leftover lunch, when he looks for Confirm or Skip, then neither control is shown, consistent with his View-only Weekly Plan access.

**FEAT-11.SPEC-001-AC-05:** Given Riley (Operator, support) views a household's plan during an open support request, when the day carries a leftover-lunch card, then the card is visible but shows no Confirm or Skip controls.

**FEAT-11.SPEC-001-AC-06:** Given an unauthenticated visitor attempts to reach the Weekly Plan directly, when they are redirected to sign in, then after signing in they land on the current Weekly Plan rather than being deep-linked to a specific leftover-lunch card.

**FEAT-11.SPEC-001-AC-07:** Given Maya's session expires while she is viewing the plan, when she next taps Confirm, then the dialog "Your session has expired. Sign in to continue." appears, and no partial status change is recorded.

**FEAT-11.SPEC-001-AC-08:** Given Maya and Sam both tap an action on the same leftover lunch at nearly the same time, when both status updates are processed, then the last write wins and neither of them sees a conflict error.

**FEAT-11.SPEC-001-AC-09:** Given Maya loses connectivity while viewing a Suggested leftover lunch, when she taps Confirm, then the status change is queued locally and applied automatically once connectivity returns, with no error shown.

**FEAT-11.SPEC-001-AC-10:** Given the leftover lunch's linked source dinner is swapped while its card is Suggested, when FEAT-11.SPEC-004 re-links it, then the card reflects the new source dinner's name on its next read.

**FEAT-11.SPEC-001-AC-11:** Given a day has no leftover-lunch suggestion attached, when any household member views that day, then no card and no placeholder appear for it.

**FEAT-11.SPEC-001-AC-12:** Given Maya taps Confirm on a Suggested leftover lunch, when FEAT-11.SPEC-004 has withdrawn or re-linked that same record moments earlier because its source dinner was swapped or cleared, then Maya's save is rejected, the card silently refreshes to the withdrawn or re-linked state, and no error is shown to her.

**FEAT-11.SPEC-001-AC-13:** Given Sam taps Skip on a Suggested leftover lunch, when the save is rejected by the server for a reason other than the withdrawal race, then the card stays in Suggested state showing "Couldn't save. Try again." and Sam can retry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 8 (suggested, eaten, skipped, not rendered, loading N/A, error N/A, save rejected -- race, save rejected -- server error) plus offline | 9 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |
