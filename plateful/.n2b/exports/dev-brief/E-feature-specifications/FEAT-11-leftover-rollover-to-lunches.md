# FEAT-11 — Leftover Rollover to Lunches

This chapter covers FEAT-11, Leftover Rollover to Lunches, a Important-tier feature. It contains 4 specifications carrying 47 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-11.SPEC-001 | Leftover Lunch Card | screen | 13 |
| FEAT-11.SPEC-002 | Leftover Lunch Suggestion Generation | automation | 8 |
| FEAT-11.SPEC-003 | Leftover Lunch Eligibility & Linking Rule | logic-rule | 16 |
| FEAT-11.SPEC-004 | Leftover Lunch Withdrawal on Source Change | automation | 10 |

The feature breakdown brief follows, then every specification in full.


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



# Automation Spec: Leftover Lunch Suggestion Generation

## Overview

**Name:** Leftover Lunch Suggestion Generation
**ID:** FEAT-11.SPEC-002
**Type:** Automation
**Purpose:** As part of AI weekly plan generation, creates a Suggested leftover-lunch Planned Meal for each dinner the eligibility rule flags as leftover-producing.
**Parent Feature:** FEAT-11 -- Leftover Rollover to Lunches

## Scope and Non-Goals

**In Scope:**
- Evaluating every dinner in a newly generated Weekly Plan against FEAT-11.SPEC-003's eligibility rule
- Resolving each eligible dinner's following day via FEAT-11.SPEC-003 and creating the linked Suggested leftover-lunch Planned Meal
- Handling the case where more than one eligible dinner in the same week would otherwise collide on the same following day

**Non-Goals:**
- Determining what makes a dinner leftover-producing, or which following day to use -- owned by FEAT-11.SPEC-003 (Leftover Lunch Eligibility & Linking Rule); this automation calls that rule rather than re-deriving it
- Re-evaluating a leftover lunch after its source dinner changes post-generation -- owned by FEAT-11.SPEC-004 (Leftover Lunch Withdrawal on Source Change), a distinct event-triggered path
- Generating leftover-lunch suggestions from Manual Weekly Planning (FEAT-23) activity -- this automation's only trigger is a completed FEAT-03 generation run (Trigger Definition above); product-features.md's FEAT-11 entry names AI Weekly Dinner Plan Generation (FEAT-03), not FEAT-23, as the dependency this feature extends. This holds whether FEAT-23 is building a free-tier week from scratch (which never carries a leftover-lunch suggestion, since no FEAT-03 run ever produced its dinners) or hand-editing one night within an already AI-generated plan (paid tier) -- a night changed that way keeps whatever FEAT-11.SPEC-004 decides for its existing leftover-lunch link, if any; this automation itself never re-runs against it, since it only fires once, at the moment a generation run completes
- Confirming or skipping a leftover lunch once created -- owned by FEAT-11.SPEC-001 (Leftover Lunch Card)

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Scheduled weekly plan generation completes | FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) | Fires once the new Weekly Plan and its seven dinner Planned Meals are created (Generation succeeded or Generation succeeded, over budget outcome) | The newly created Weekly Plan and its seven dinner Planned Meals (night, recipe, status) |
| First-plan generation on upgrade completes | FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) | Fires once the household's first AI-generated Weekly Plan and its seven dinner Planned Meals are created | Same as above |

## Processing Logic

1. Receive the newly created Weekly Plan and its seven dinner Planned Meals from the completed generation run.
2. In night order (Monday through Sunday), evaluate each dinner Planned Meal against FEAT-11.SPEC-003's eligibility determination.
3. For each dinner classified leftover-producing, request the following day from FEAT-11.SPEC-003's following-day computation (default: one day after the source dinner's night).
4. If the computed following day already holds another Suggested, Eaten, or Skipped leftover lunch created earlier in this same run, request the two-day fallback day from FEAT-11.SPEC-003 instead.
5. If both the one-day and two-day following days are already occupied by another leftover lunch from this run, do not create a suggestion for this dinner this week (the two-day ceiling in XBR-10 is never exceeded to find a free day).
6. Otherwise, create a new Planned Meal (meal_kind: leftover lunch, status: Suggested) linked to the source dinner, placed on the resolved following day.
7. Repeat steps 2-6 for every dinner in the week.
8. Signal completion so the Weekly Plan View (FEAT-03.SPEC-001) and Leftover Lunch Card (FEAT-11.SPEC-001) reflect every newly created suggestion.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Suggestions created | One or more dinners in the week are classified leftover-producing and a following day is available for each | One new Suggested leftover-lunch Planned Meal per eligible, successfully placed dinner | Each created suggestion appears as a Leftover Lunch Card on its following day | FEAT-11.SPEC-001, FEAT-03.SPEC-001 |
| No eligible dinners this week | No dinner in the week is classified leftover-producing | None | No leftover-lunch cards appear anywhere in the week; this is the normal case, not an error (product-features.md, States) | FEAT-03.SPEC-001 |
| Eligible dinner skipped for day collision | An eligible dinner's one-day and two-day following days are both already occupied by another leftover lunch from this run | No leftover-lunch record created for that dinner | No card appears for that dinner this week; no error is shown anywhere | FEAT-11.SPEC-001 |
| Per-dinner computation failure | The eligibility or day-placement computation fails for one specific dinner | No leftover-lunch record created for that dinner; all other dinners in the run are unaffected | No card appears for that dinner; no error is shown to the household, consistent with product-features.md's "a failure to compute it simply omits the suggestion rather than showing an error" | FEAT-11.SPEC-001 |

## Data Model

**Reads:** Planned Meal (source dinner) -- night, recipe, status, for each of the week's seven dinners. Weekly Plan -- week, to scope the run to the newly generated plan.
**Creates:** Planned Meal (leftover-lunch sub-type) -- meal_kind (leftover lunch), linked source dinner, night (the resolved following day), status (Suggested). One record per eligible, successfully placed dinner.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-10: every leftover-lunch record this automation creates links to exactly one source dinner and a following day no more than two days later.
- Eligibility and day-placement logic are never re-derived here -- both are delegated to FEAT-11.SPEC-003 on every evaluation.
- At most one leftover-lunch Planned Meal is created per leftover-eligible dinner per week per household (feature-dependency-map.md, Non-Functional Notes).
- This automation runs only as part of AI Weekly Dinner Plan Generation (FEAT-03); Manual Weekly Planning (FEAT-23) dinners are never evaluated by it, per the Brief's declared dependency on FEAT-03 alone.
- A per-dinner computation failure never blocks or fails the parent plan generation run -- the affected dinner simply receives no leftover-lunch suggestion.

## Edge Cases

- **Two eligible dinners in the same week would both default to the same following day** -- The second dinner processed (in night order) falls back to its two-day following day instead, per FEAT-11.SPEC-003.
- **Three or more eligible dinners collide on the same day window** -- Each is resolved in night order; any dinner for which both its one-day and two-day following days are already taken receives no suggestion this week, per the Outcome Definitions above.
- **A source dinner falls on the last night of the week (e.g., Sunday)** -- The following day may fall in the next calendar week; the leftover lunch is still created and attached to the day it falls on, since the two-day ceiling is measured in elapsed days, not week boundaries.
- **Zero dinners in the week are leftover-producing** -- No leftover-lunch records are created; this is the normal "No eligible dinners this week" outcome, not a failure.
- **Concurrent trigger firing (two generation completions for the same household at effectively the same time)** -- Cannot occur: FEAT-03.SPEC-003 guarantees only one generation run is in flight per household at a time, so this automation is never invoked twice concurrently for the same household's week.
- **Trigger fires while a previous run of this automation is still in flight** -- Cannot occur for the same reason: this automation only ever runs once per completed generation cycle per household, and the next cycle cannot begin until the current one (and everything it triggers) has settled, per FEAT-03.SPEC-003's own concurrency guarantee.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) | Triggered by (inbound) | Completion of scheduled generation fires this automation (bidirectional reference gap: FEAT-03.SPEC-003's own Connected Specs table does not yet name this spec as an outbound trigger -- flagged for cross-reference reconciliation, since FEAT-03.SPEC-003 is owned by a different feature and outside this spec's authority to edit) |
| FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) | Triggered by (inbound) | Completion of the household's first generation fires this automation |
| FEAT-11.SPEC-003 (Leftover Lunch Eligibility & Linking Rule) | References (outbound) | Supplies the eligibility determination and following-day computation this automation calls for every dinner |
| FEAT-11.SPEC-001 (Leftover Lunch Card) | Affects (outbound) | Displays every leftover-lunch suggestion this automation creates |
| FEAT-03.SPEC-001 (Weekly Plan View) | Affects (outbound) | Shows the newly generated week including its leftover-lunch suggestions |

## Analytics and Success Signals

- **leftover_lunch_suggested** (source dinner night, following day, count created this week) -- supports success-metrics.md: "Reported Food Waste and Spend Reduction"
- **leftover_lunch_generation_skipped** (reason: day_collision / per_dinner_computation_failure) -- N/A -- no Stage 2 metric measures this operationally-only signal directly; retained to observe how often an eligible dinner fails to receive a suggestion

## Acceptance Criteria

**FEAT-11.SPEC-002-AC-01:** Given Maya's household's scheduled weekly plan generation (FEAT-03.SPEC-003) completes with one dinner classified leftover-producing, when this automation runs, then a Suggested leftover-lunch Planned Meal is created, linked to that dinner, on the day immediately following it.

**FEAT-11.SPEC-002-AC-02:** Given a household's first-ever AI plan generation (FEAT-03.SPEC-004) completes with an eligible dinner, when this automation runs, then a Suggested leftover-lunch suggestion is created exactly as it would be for a scheduled generation.

**FEAT-11.SPEC-002-AC-03:** Given a generated week contains no dinner classified leftover-producing, when this automation runs, then no leftover-lunch records are created and no card appears anywhere in the week.

**FEAT-11.SPEC-002-AC-04:** Given two eligible dinners in the same week would default to the same following day, when this automation processes the second one, then it is placed on its two-day fallback day instead of colliding with the first.

**FEAT-11.SPEC-002-AC-05:** Given a third eligible dinner's one-day and two-day following days are both already occupied by other leftover lunches from the same run, when this automation processes it, then no leftover-lunch suggestion is created for that dinner and no error is shown.

**FEAT-11.SPEC-002-AC-06:** Given the eligibility computation fails for one specific dinner in an otherwise successful generation run, when this automation completes, then only that dinner has no leftover-lunch suggestion, and every other eligible dinner in the week is unaffected.

**FEAT-11.SPEC-002-AC-07:** Given a household plans its week through Manual Weekly Planning (FEAT-23) instead of AI generation, when those dinners are picked, then this automation never runs against them and no leftover-lunch suggestions are created for that week.

**FEAT-11.SPEC-002-AC-08:** Given a household's generation run is already in flight, when a second trigger for the same household would otherwise arrive, then it cannot occur, since FEAT-03.SPEC-003 guarantees only one generation run per household at a time.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (scheduled generation, first-plan on upgrade) | 2 |
| Outcome Paths | 4 (suggestions created, no eligible dinners, day-collision skip, per-dinner computation failure) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Leftover Lunch Eligibility & Linking Rule

## Overview

**Name:** Leftover Lunch Eligibility & Linking Rule
**ID:** FEAT-11.SPEC-003
**Type:** Logic/Rule
**Purpose:** Determines which dinners produce leftover-worthy portions, which following day (no more than two days later) to suggest, and enforces the one-source/one-day link and every authorization rule on the leftover-lunch Planned Meal.
**Parent Feature:** FEAT-11 -- Leftover Rollover to Lunches
**Governed Entity:** Planned Meal (leftover-lunch sub-type)

## Scope and Non-Goals

**In Scope:**
- Field validation rules for the leftover-lunch Planned Meal's fields (meal_kind, linked source dinner, night, status)
- The eligibility determination that classifies a dinner as leftover-producing
- The following-day computation, including its two-day ceiling and collision fallback
- Cross-field rules enforcing the one-source/one-day link
- Authorization rules for every action on the leftover-lunch Planned Meal, per role
- Default values and derivations for every field

**Non-Goals:**
- Creating the leftover-lunch Planned Meal record -- owned by FEAT-11.SPEC-002 (Leftover Lunch Suggestion Generation), which calls this spec's rules but performs the actual write
- Re-evaluating an existing leftover lunch when its source dinner changes -- owned by FEAT-11.SPEC-004 (Leftover Lunch Withdrawal on Source Change), which calls this spec's eligibility and day rules but owns the re-link/withdraw decision itself
- Detailed leftover quantity or expiry tracking -- excluded per scope-boundaries.md SC-11: eligibility here is a simple yes/no classification, never a tracked quantity or expiry estimate
- Displaying the Confirm/Skip controls this spec authorizes -- owned by FEAT-11.SPEC-001 (Leftover Lunch Card), which enforces these authorization rules on screen but does not define them

## Governed Entity

**Entity:** Planned Meal (leftover-lunch sub-type)
**Source:** Feature Dependency Map (Planned Meal entity; leftover-lunch fields per the Brief's Shared Context)

| Field | Data Type | Description |
|-------|-----------|-------------|
| meal_kind | enum | Fixed to "leftover lunch" for every record this spec governs, distinguishing it from a dinner Planned Meal |
| linked source dinner | reference | The one dinner Planned Meal this leftover lunch rolls over from |
| night | date | The following day the leftover lunch is attached to, computed as no more than two days after the source dinner's night |
| status | enum | Suggested, Eaten, or Skipped |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-11.SPEC-002 | Leftover Lunch Suggestion Generation | On creation, during each AI plan-generation cycle: calls the eligibility determination and following-day computation for every dinner |
| FEAT-11.SPEC-004 | Leftover Lunch Withdrawal on Source Change | On re-evaluation, when the linked source dinner is swapped or changed: calls the eligibility determination and following-day computation again, and applies the re-link/withdraw authorization rules |
| FEAT-11.SPEC-001 | Leftover Lunch Card | On screen entry and on action attempt: enforces the Confirm/Skip authorization rules for the viewing role |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| meal_kind | Always set to "leftover lunch"; system-derived, never entered or edited by any role | Always | On create | N/A -- not a user-entered field; no manual-creation path exists for this record (Non-Goals) | Yes |
| linked source dinner | Must reference exactly one dinner-type Planned Meal in the same household's plan; must never be null while status is Suggested | Always | On create (FEAT-11.SPEC-002) and on re-link (FEAT-11.SPEC-004) | N/A -- system-computed, never entered manually; a dinner that cannot be resolved simply results in no leftover-lunch record being created or in the existing one being withdrawn (FEAT-11.SPEC-004) | Yes |
| night | Must be strictly after the linked source dinner's night, and no more than two days after it (XBR-10) | Always | On create and on re-link | N/A -- system-computed; when no day within the ceiling is free, no record is created (FEAT-11.SPEC-002) or the existing record is withdrawn (FEAT-11.SPEC-004) rather than placing it outside the ceiling | Yes |
| status | Must be one of Suggested, Eaten, Skipped; transitions only Suggested -> Eaten or Suggested -> Skipped, each a one-way, terminal transition | Always | On create (defaults to Suggested) and on update (FEAT-11.SPEC-001's Confirm/Skip) | N/A -- enforced structurally: Confirm and Skip are only ever offered while status is Suggested (FEAT-11.SPEC-001's Access and Visibility), so no invalid transition is ever presented as an option | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| One active link per source dinner | linked source dinner, status | At most one leftover-lunch record with status Suggested, Eaten, or Skipped may reference the same source dinner at a time; a source-dinner change re-links or withdraws the existing record (FEAT-11.SPEC-004) rather than ever creating a second link to the same dinner | N/A -- structurally enforced by FEAT-11.SPEC-002 and FEAT-11.SPEC-004, which always update or replace the existing link instead of creating a duplicate |
| Following-day ceiling | night, linked source dinner (its night) | night must fall strictly after the source dinner's night and no more than two calendar days after it (XBR-10) | N/A -- system-computed; a placement that would exceed the ceiling is never made (see Defaults and Derivations, Following-Day Computation) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create a leftover-lunch suggestion | No household role -- system-automation only (FEAT-11.SPEC-002) | Always | Not offered as a manual action to any role; product-features.md's Validation & Limits and this feature's Non-Goals establish no manual-creation path |
| View a leftover-lunch card | Maya (Organiser) | Always | -- |
| View a leftover-lunch card | Sam (Other Adult Member) | Always | -- |
| View a leftover-lunch card | Jordan (older kid, limited login -- Later) | Always (his View access to the Weekly Plan includes it) | -- |
| View a leftover-lunch card | Riley (Operator, support) | Always, within an open support request's read-only view | -- |
| View a leftover-lunch card | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role; there is no path into the product to reach it |
| Confirm (mark Eaten) | Maya (Organiser) | Only while the record's status is Suggested | -- |
| Confirm (mark Eaten) | Sam (Other Adult Member) | Only while the record's status is Suggested; his Weekly Plan access is View, but a leftover-lunch status update is treated as a status update on the plan, not a change to it (Access Matrix notes) | -- |
| Confirm (mark Eaten) | Jordan (older kid, limited login -- Later) | Never | Confirm control is hidden; his Weekly Plan access is View-only |
| Confirm (mark Eaten) | Riley (Operator, support) | Never | Confirm control is hidden in the read-only support view (XBR-14) |
| Confirm (mark Eaten) | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role; there is no path into the product to reach it |
| Skip | Maya (Organiser) | Only while the record's status is Suggested | -- |
| Skip | Sam (Other Adult Member) | Only while the record's status is Suggested; same reasoning as Confirm above | -- |
| Skip | Jordan (older kid, limited login -- Later) | Never | Skip control is hidden; his Weekly Plan access is View-only |
| Skip | Riley (Operator, support) | Never | Skip control is hidden in the read-only support view (XBR-14) |
| Skip | Jordan (young kid profile, no login -- MVP) | Never | No login exists for this role; there is no path into the product to reach it |
| Re-link to a new source dinner or day | No household role -- system-automation only (FEAT-11.SPEC-004) | Fires only when the linked source dinner's recipe changes or the slot is otherwise altered | Not offered as a manual action to any role; no reschedule path is modeled anywhere in the product (Non-Goals) |
| Withdraw (remove) a Suggested leftover-lunch record | No household role -- system-automation only (FEAT-11.SPEC-004) | Fires only when the source dinner is swapped or cleared and no eligible replacement takes the slot | Not offered as a manual action to any role |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| meal_kind | Always "leftover lunch" for a record this spec governs | On create | No |
| status | Defaults to Suggested | On create only | No -- it changes only through the Confirm/Skip actions in FEAT-11.SPEC-001, never through direct edit |
| linked source dinner | Set to the dinner Planned Meal FEAT-11.SPEC-002 identified as leftover-producing (on create), or the dinner's new recipe reference (on re-link, FEAT-11.SPEC-004) | On create and on re-link | No -- no manual re-link or reschedule path is modeled |
| night | Set by the Following-Day Computation below (on create and on re-link) | On create and on re-link | No |
| **Eligibility Determination (derivation owned by this spec, not a stored Planned Meal or Recipe field)** | A dinner is classified leftover-producing when the source Recipe's existing `name` field (feature-dependency-map.md, Recipe entity) matches, case-insensitively as a whole word, one of this spec's fixed set of batch-style dish-type terms: casserole, roast, stew, bake, chili, chilli, curry, lasagna, lasagne, soup, pot pie, batch. This spec owns and maintains that closed term list; no new field is added to Recipe or Planned Meal to store the classification -- the match runs fresh against the recipe's own `name` text every time this determination is evaluated, using only data the Recipe entity already carries. This is a fixed yes/no outcome per recipe and does not vary by which household cooks it or by household size -- household size affects how a dinner's portions are sized (Planned Meal's cook_time and rough_cost, "carried from the recipe, sized for the household," per the dependency map), but plays no part in this classification itself, which is a content-level read of the recipe's name alone. | Evaluated once per dinner, at each plan-generation cycle (FEAT-11.SPEC-002) and at each source-change re-evaluation (FEAT-11.SPEC-004) | No -- no household role can mark a dinner leftover-producing or not; the classification is read from the recipe's own `name` field each time |
| **Following-Day Computation (derivation, produces the night value above)** | Default: the day immediately following the source dinner's night. Fallback: if that day already holds another Suggested, Eaten, or Skipped leftover lunch from the same evaluation context (the same weekly plan on create, or the same slot's prior link on re-link), the day two days after the source dinner's night is used instead. If both the one-day and two-day days are already occupied, no placement is made -- the two-day ceiling (XBR-10) is never exceeded to find a free day. | Evaluated once per dinner alongside the Eligibility Determination | No |

## Business Rules

- XBR-10: a leftover lunch links to exactly one source dinner and a following day no more than two days later; swapping or removing the source dinner updates or withdraws the leftover suggestion (enforced by FEAT-11.SPEC-004, which calls this spec's rules).
- Eligibility is derived fresh each time from the recipe's existing `name` field against this spec's fixed dish-type term list (Defaults and Derivations, Eligibility Determination) -- it is never stored as a Recipe or Planned Meal field, and it is a fixed recipe-level outcome, not a per-week or per-household variation, keeping the "simple yes/no suggestion" model scope-boundaries.md SC-11 establishes.
- Confirm and Skip are each one-way, terminal transitions -- once Eaten or Skipped, a record's status never changes again through this feature (scope-boundaries.md SC-18 retains the outcome as permanent plan history).
- The following-day computation never exceeds the two-day ceiling to resolve a collision -- an eligible dinner that cannot be placed within the ceiling simply receives no suggestion that week (FEAT-11.SPEC-002) or has its existing suggestion withdrawn rather than moved beyond the ceiling (FEAT-11.SPEC-004).
- No household role has a create, reschedule, or manual-link action on this entity -- every write path is system-automation only (FEAT-11.SPEC-002, FEAT-11.SPEC-004), consistent with the feature's Non-Goals.

## Edge Cases

- **Following day computed at exactly two days after the source dinner** -- Passes the ceiling; a placement three days after the source dinner is never made under any fallback.
- **Both the one-day and two-day following days are already occupied by other leftover lunches from the same evaluation context** -- No placement is made; FEAT-11.SPEC-002 creates no record for that dinner, or FEAT-11.SPEC-004 withdraws the existing one rather than placing it further out.
- **A source dinner falls on the last night of the week** -- The following day may fall in the next calendar week; the ceiling is measured in elapsed days, not week boundaries, so the placement still proceeds normally.
- **A recipe's leftover-producing classification is revised in the Recipe Library after a leftover lunch has already been created from it** -- The already-created record keeps its existing link and night; classification is only re-evaluated when FEAT-11.SPEC-004 re-runs it because the source dinner itself changed, not because the recipe's own content was edited independently.
- **Sam attempts to Confirm a leftover lunch that has already been marked Skipped** -- Denied: the Confirm control is not shown once status is no longer Suggested, per the Field Validation Rules' terminal-transition rule.
- **Jordan (older kid, limited login) attempts to reach the Confirm action directly (e.g., a stale link)** -- Denied: the action is refused and the leftover-lunch card renders in its View-only presentation, consistent with his Authorization Rules row.
- **The linked source dinner is swapped to a recipe that is also leftover-producing** -- Handled by FEAT-11.SPEC-004 as a re-link (the existing record's linked source dinner reference updates; the night is only recomputed if the slot's own night changed, which a swap never does).

## Acceptance Criteria

**FEAT-11.SPEC-003-AC-01:** Given a dinner recipe whose `name` field contains a batch-style dish-type term from this spec's fixed list (e.g., "Tuesday's Chicken Casserole"), when FEAT-11.SPEC-002 evaluates it during plan generation, then it is classified leftover-producing and a Suggested leftover lunch is created for it.

**FEAT-11.SPEC-003-AC-02:** Given a dinner recipe whose `name` field matches none of this spec's batch-style dish-type terms, when FEAT-11.SPEC-002 evaluates it, then it is not classified leftover-producing and no leftover-lunch record is created for it.

**FEAT-11.SPEC-003-AC-03:** Given an eligible dinner on Tuesday, when the following-day computation runs and Wednesday is free, then the leftover lunch's night is set to Wednesday.

**FEAT-11.SPEC-003-AC-04:** Given an eligible dinner on Tuesday and Wednesday already holds another leftover lunch from the same week, when the following-day computation runs, then the leftover lunch's night falls back to Thursday.

**FEAT-11.SPEC-003-AC-05:** Given an eligible dinner whose Wednesday and Thursday following days are both already occupied, when the following-day computation runs, then no leftover lunch is created for that dinner and no day beyond the two-day ceiling is used.

**FEAT-11.SPEC-003-AC-06:** Given a leftover lunch already exists Suggested for a source dinner, when that same dinner is evaluated again in a later cycle, then no second leftover-lunch record is created for it (one active link per source dinner).

**FEAT-11.SPEC-003-AC-07:** Given Maya views a Suggested leftover lunch, when she taps Confirm, then the status transitions to Eaten, and this transition cannot later be reversed through this feature.

**FEAT-11.SPEC-003-AC-08:** Given Sam views a Suggested leftover lunch while his Weekly Plan access is View, when he taps Skip, then the status transitions to Skipped, since this is a status update rather than a plan change.

**FEAT-11.SPEC-003-AC-09:** Given Jordan (older kid, limited login) views a Suggested leftover lunch, when he looks for Confirm or Skip, then neither is shown, per his View-only authorization.

**FEAT-11.SPEC-003-AC-10:** Given Riley (Operator, support) views a household's plan during an open support request, when a leftover-lunch card is present, then Riley can view it but has no Confirm or Skip control, per XBR-14.

**FEAT-11.SPEC-003-AC-11:** Given Jordan (young kid profile, no login) has no path into the product, when this rule's Authorization Rules are evaluated for this role, then every action is Never, consistent with having no login.

**FEAT-11.SPEC-003-AC-12:** Given no household role has a create action on this entity, when any role looks for a way to manually add a leftover lunch, then no such control exists anywhere in the product.

**FEAT-11.SPEC-003-AC-13:** Given a source dinner is swapped to a still-eligible recipe, when FEAT-11.SPEC-004 calls this spec's rules, then the existing Suggested record's linked source dinner is updated (re-linked) rather than a second record being created.

**FEAT-11.SPEC-003-AC-14:** Given a source dinner is swapped to a no-longer-eligible recipe, when FEAT-11.SPEC-004 calls this spec's eligibility rule, then the eligibility determination returns not-eligible and FEAT-11.SPEC-004 withdraws the existing Suggested record.

**FEAT-11.SPEC-003-AC-15:** Given a leftover lunch is already marked Eaten, when its source dinner is later swapped, then this spec's rules are never invoked to re-link or withdraw it, since only Suggested records are re-evaluated.

**FEAT-11.SPEC-003-AC-16:** Given a source dinner falls on the last night of the week, when the following-day computation places its leftover lunch in the next calendar week, then the placement still succeeds, since the two-day ceiling is measured in elapsed days rather than week boundaries.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 18 | 18 |
| Defaults/Derivations | 6 | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |



# Automation Spec: Leftover Lunch Withdrawal on Source Change

## Overview

**Name:** Leftover Lunch Withdrawal on Source Change
**ID:** FEAT-11.SPEC-004
**Type:** Automation
**Purpose:** When a linked source dinner is swapped or removed, re-evaluates the leftover-lunch suggestion attached to it and either re-links or withdraws it.
**Parent Feature:** FEAT-11 -- Leftover Rollover to Lunches

## Scope and Non-Goals

**In Scope:**
- Detecting that a dinner Planned Meal with a linked, Suggested leftover lunch has had its recipe swapped, or has been cleared or changed manually
- Re-running FEAT-11.SPEC-003's eligibility determination against the changed dinner
- Re-linking the existing leftover-lunch record when the new dinner is still eligible, or hard-deleting it when it is not
- Leaving an already-Confirmed (Eaten) or already-Skipped leftover lunch untouched regardless of what happens to its former source dinner

**Non-Goals:**
- Determining eligibility or the following-day placement from first principles -- owned by FEAT-11.SPEC-003 (Leftover Lunch Eligibility & Linking Rule); this automation calls that rule rather than re-deriving it
- Creating the first leftover-lunch suggestion for a newly generated week -- owned by FEAT-11.SPEC-002 (Leftover Lunch Suggestion Generation), a distinct trigger path
- Restoring a withdrawn leftover lunch -- product-features.md's Validation & Limits and the Entity-Lifecycle Coverage Matrix establish no restore path; a fresh suggestion appears only if a later plan-generation cycle or this automation's own re-link path attaches a newly eligible dinner to the slot
- Notifying any household member that a leftover lunch was updated or withdrawn -- excluded per this feature's own Communications field (product-features.md): no separate notification is sent for any leftover-lunch outcome

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A source dinner is swapped | FEAT-04.SPEC-004 (Apply Meal Swap) | Fires when a swap completes successfully on a Planned Meal slot that has a linked leftover lunch | The changed Planned Meal's new recipe, night; the linked leftover-lunch Planned Meal's current status, night, and source reference |
| A source dinner is cleared or changed manually | FEAT-23 (Manual Weekly Planning) | Fires when a night's dinner Planned Meal is cleared or its recipe changed in a manually built week, and that slot has a linked leftover lunch | The changed (or now-empty) Planned Meal slot's new recipe if any, night; the linked leftover-lunch Planned Meal's current status, night, and source reference |

## Processing Logic

1. Receive the changed dinner Planned Meal (its new recipe, or notice that the slot is now empty) and the leftover-lunch record currently linked to it.
2. If no leftover-lunch record is linked to the changed slot, take no action -- most swaps and manual changes never touch a linked leftover lunch.
3. If a leftover-lunch record is linked but its status is Eaten or Skipped, take no action -- confirmed history is never altered by a later source change (scope-boundaries.md SC-18).
4. If the linked leftover-lunch record's status is Suggested, pass the new dinner (or the empty slot) through FEAT-11.SPEC-003's eligibility determination.
5. If the new dinner is classified leftover-producing, re-link the existing leftover-lunch record's source reference to it; its night is only recomputed if the slot's own night changed (a swap or manual recipe change never changes which night the slot occupies, so the night typically stays the same).
6. If the new dinner is not classified leftover-producing, or the slot is now empty with no replacement dinner, hard-delete the Suggested leftover-lunch record.
7. Signal the Leftover Lunch Card (FEAT-11.SPEC-001) and the hosting Weekly Plan screen to reflect the re-link or removal.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Re-linked to new source dinner | The changed slot's new dinner is still classified leftover-producing, and the linked leftover lunch is Suggested | The leftover-lunch record's linked source dinner reference is updated; night unchanged unless the slot's night itself changed | The Leftover Lunch Card now shows the new dinner's recipe name; no toast or interruption | FEAT-11.SPEC-001 |
| Withdrawn (removed) | The changed slot's new dinner is not leftover-producing, or the slot is now empty, and the linked leftover lunch is Suggested | The Suggested leftover-lunch record is hard-deleted; no restore path | The Leftover Lunch Card disappears from the day it was attached to on the next read; no error or notification is shown | FEAT-11.SPEC-001 |
| No action -- already resolved | The linked leftover lunch is Eaten or Skipped | None | The card continues to show its resolved Eaten or Skipped state unchanged, even though its former source dinner has changed | FEAT-11.SPEC-001 |
| No action -- no linked leftover lunch | The changed slot has no linked leftover-lunch record at all | None | Nothing changes; this is the most common outcome, since most dinners are not leftover-producing | -- |
| Re-evaluation failure (fail-safe withdrawal) | The eligibility re-check itself cannot complete for the changed dinner | The Suggested leftover-lunch record is withdrawn (hard-deleted) as a conservative default rather than left pointing at a stale or unverified source | The card disappears; no error is shown, consistent with product-features.md's "a failure to compute it simply omits the suggestion rather than showing an error" | FEAT-11.SPEC-001 |

## Data Model

**Reads:** Planned Meal (source dinner, post-change) -- recipe, night, status. Planned Meal (leftover-lunch sub-type) -- status, linked source dinner, night, for the record attached to the changed slot.
**Creates:** None -- re-linking updates the existing record; a fresh record for a different, still-empty slot is only ever created by FEAT-11.SPEC-002's own generation cycle.
**Updates:** Planned Meal (leftover-lunch sub-type) -- linked source dinner (on re-link); night, only when the slot's own night changed.
**Deletes:** Planned Meal (leftover-lunch sub-type) -- hard delete of the Suggested record when withdrawn, per the Entity-Lifecycle Coverage Matrix's Delete/Archive row.

## Business Rules

- XBR-10: a source dinner swap (FEAT-04) or manual clear/change (FEAT-23) always re-evaluates or withdraws its linked leftover suggestion; the link is never left pointing at a dinner that no longer exists in that slot.
- Confirmed Eaten or Skipped leftover lunches are never touched by this automation -- they are retained as permanent plan history (scope-boundaries.md SC-18), regardless of what happens to their former source dinner.
- Withdrawal is a hard delete with no restore path -- a fresh suggestion for that slot appears only if a newly eligible dinner takes it, whether through this automation's own re-link path or a future plan-generation cycle (FEAT-11.SPEC-002).
- A re-evaluation failure defaults to withdrawal rather than leaving the leftover lunch linked to a stale or unverified source -- a missing suggestion is preferred over an incorrect one, consistent with the product's general failure posture for this feature (product-features.md, States).
- This automation never creates a leftover-lunch record for a slot that never had one -- only FEAT-11.SPEC-002's generation cycle originates new suggestions.

## Edge Cases

- **Source dinner swapped to a recipe that is also leftover-producing** -- The existing Suggested record is re-linked to the new recipe rather than deleted and recreated, preserving its following day (the slot's night is unchanged by a swap).
- **Source dinner swapped to a recipe that is not leftover-producing** -- The existing Suggested record is withdrawn; the household simply loses that day's leftover-lunch card with no error shown.
- **A night is cleared entirely in a manually built week (FEAT-23), with no replacement dinner chosen** -- Treated the same as a swap to a non-eligible recipe: the linked Suggested leftover lunch is withdrawn.
- **The linked leftover lunch is already marked Eaten when its source dinner is later swapped** -- No action is taken; the confirmed record is untouched (per Outcome Definitions).
- **The linked leftover lunch is already marked Skipped when its source dinner is later cleared** -- No action is taken; the Skipped record is retained exactly as it was.
- **Concurrent trigger firing (a swap and a manual change targeting the same slot at effectively the same time)** -- Cannot occur: FEAT-04.SPEC-009's concurrency lock (for swaps) and Manual Weekly Planning's own per-slot handling ensure only one change to a given slot completes at a time; this automation is invoked once per completed, settled change, never twice concurrently for the same slot.
- **Trigger fires while a previous run of this automation for the same slot is still in flight** -- Cannot occur for the same reason: a second change to the same slot cannot begin until the first one's own concurrency control (FEAT-04.SPEC-009 for swaps) releases, so this automation's runs for a given slot are naturally serialized.
- **The withdrawn slot later receives a newly eligible dinner in a subsequent weekly plan-generation cycle** -- A brand-new Suggested leftover lunch may be created then by FEAT-11.SPEC-002, entirely independent of the record this automation withdrew earlier.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-004 (Apply Meal Swap) | Triggered by (inbound) | A completed swap on a slot with a linked leftover lunch fires this automation |
| FEAT-23 (Manual Weekly Planning) | Triggered by (inbound) | Clearing or changing a night manually on a slot with a linked leftover lunch fires this automation |
| FEAT-11.SPEC-003 (Leftover Lunch Eligibility & Linking Rule) | References (outbound) | Supplies the eligibility determination this automation re-runs against the changed dinner |
| FEAT-11.SPEC-002 (Leftover Lunch Suggestion Generation) | References (outbound) | Owns fresh suggestion creation; this automation only re-links or withdraws an existing record |
| FEAT-11.SPEC-001 (Leftover Lunch Card) | Affects (outbound) | Displays the re-linked dinner, or stops displaying a withdrawn card, on its next read |

## Analytics and Success Signals

- **leftover_lunch_relinked** (former source dinner night, new source dinner night) -- supports success-metrics.md: "Reported Food Waste and Spend Reduction" (keeping the suggestion accurate after a swap keeps the waste-reduction mechanism trustworthy)
- **leftover_lunch_withdrawn** (reason: source_no_longer_eligible / slot_cleared / reevaluation_failure) -- N/A -- no Stage 2 metric measures withdrawal volume directly; retained to observe how often a source change disrupts an existing suggestion

## Acceptance Criteria

**FEAT-11.SPEC-004-AC-01:** Given a Suggested leftover lunch is linked to a dinner that Maya swaps for another leftover-producing recipe, when the swap completes (FEAT-04.SPEC-004), then this automation re-links the leftover lunch to the new recipe and its card shows the new dinner's name.

**FEAT-11.SPEC-004-AC-02:** Given a Suggested leftover lunch is linked to a dinner that Maya swaps for a recipe that is not leftover-producing, when the swap completes, then this automation withdraws the leftover lunch and its card disappears.

**FEAT-11.SPEC-004-AC-03:** Given a Suggested leftover lunch is linked to a dinner that is cleared entirely in a manually built week with no replacement, when the clear is applied (FEAT-23), then this automation withdraws the leftover lunch.

**FEAT-11.SPEC-004-AC-04:** Given a manually built week's dinner is changed to a still leftover-producing recipe, when the change is applied, then this automation re-links the existing leftover lunch to the new recipe rather than creating a duplicate.

**FEAT-11.SPEC-004-AC-05:** Given a leftover lunch has already been marked Eaten, when its former source dinner is later swapped, then this automation takes no action and the Eaten record is unchanged.

**FEAT-11.SPEC-004-AC-06:** Given a leftover lunch has already been marked Skipped, when its former source dinner is later cleared manually, then this automation takes no action and the Skipped record is unchanged.

**FEAT-11.SPEC-004-AC-07:** Given a dinner with no linked leftover lunch is swapped, when the swap completes, then this automation takes no action, since there is nothing linked to re-evaluate.

**FEAT-11.SPEC-004-AC-08:** Given the eligibility re-check cannot complete for a changed dinner, when this automation processes the trigger, then the linked Suggested leftover lunch is withdrawn as a conservative default rather than left pointing at an unverified source.

**FEAT-11.SPEC-004-AC-09:** Given a swap and a manual change could otherwise target the same slot at the same time, when both are attempted, then the underlying concurrency controls (FEAT-04.SPEC-009 for swaps) ensure only one change completes, and this automation runs exactly once for the settled result.

**FEAT-11.SPEC-004-AC-10:** Given a leftover lunch was withdrawn from a slot, when a later weekly plan-generation cycle places a newly eligible dinner in that same slot, then FEAT-11.SPEC-002 creates a brand-new Suggested leftover lunch independent of the one this automation withdrew.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (swap, manual clear/change) | 2 |
| Outcome Paths | 5 (re-linked, withdrawn, no-action-resolved, no-action-unlinked, reevaluation-failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |
