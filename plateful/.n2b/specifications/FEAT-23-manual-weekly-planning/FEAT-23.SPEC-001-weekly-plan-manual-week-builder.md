---
document_type: spec
spec_type: screen
spec_id: FEAT-23.SPEC-001
spec_name: Weekly Plan (Manual Week Builder)
spec_slug: weekly-plan-manual-week-builder
parent_feature: FEAT-23
parent_feature_name: Manual Weekly Planning
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 16
---

# Screen Spec: Weekly Plan (Manual Week Builder)

## Overview

**Name:** Weekly Plan (Manual Week Builder)
**ID:** FEAT-23.SPEC-001
**Type:** Screen
**Purpose:** The organiser (and, view-only, other adult members) sees the seven-night week, the running estimated cost against the household's weekly budget, and taps a night to pick, change, or clear it.
**Parent Feature:** FEAT-23 -- Manual Weekly Planning

## Scope and Non-Goals

**In Scope:**
- Displaying all seven nights of the current/next manually built week with each night's state (nothing planned, picked, suggestion pending)
- Auto-initializing the Weekly Plan the first time the organiser opens a week that has not started
- Displaying the week's running estimated_total against the household's weekly_budget
- Routing Maya's taps to pick, change, or clear a night
- Routing Sam's taps to the suggestion flow, and showing the status of his own open suggestions

**Non-Goals:**
- Sam picking, changing, or clearing a night directly -- excluded per scope-boundaries.md SC-04 and the Access Matrix (Sam's Manual Planning access is Own-only); he can only reach the suggestion flow (FEAT-23.SPEC-003)
- Browsing or searching the recipe library itself -- handled by FEAT-23.SPEC-002 (Pick / Change a Recipe), which this screen navigates to
- Reviewing, accepting, or declining a pick suggestion -- owned by FEAT-04 (One-Tap Meal Swap) per feature-dependency-map.md's Swap Suggestion authority column and XBR-06; this screen only surfaces a pending badge that opens FEAT-04's review
- Browsing weeks beyond the current/next week -- excluded per this feature's Validation & Limits (one week ahead) and FEAT-23.SPEC-005; older or farther weeks are Weekly Plan History's (FEAT-19, v1) responsibility

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01 (Household Setup & Member Profiles) -- setup-complete confirmation | Organiser chooses "pick this week's dinners" on the free tier (First Household Setup, step 6) | None -- opens the current/next week to build |
| FEAT-15 (Member Onboarding) -- first-use landing | A free-tier household's first-use landing routes directly to its current manually built week | None -- opens the household's current week |
| Direct navigation (default entry) | Any household member navigates to the planning area | None |
| FEAT-19 (Weekly Plan History, v1) -- past week reused | Organiser copies a past week into a future week | Night picks pre-filled from the selected past week, each subject to this feature's safety re-check (FEAT-23.SPEC-004, XBR-01) before it displays as placed |
| FEAT-03.SPEC-002 (Free-Tier Plan Placeholder & Upgrade Prompt) | Organiser taps "Plan this week by hand" on the free-tier placeholder | None -- opens the current/next week to build |
| FEAT-24.SPEC-002 (Referral Welcome Screen) | A visitor whose own household is on the free tier taps "Go to your plan" | None -- opens the visitor's own current manually built week |
| FEAT-13.SPEC-002 (Tonight's Dinner Nudge Message) | Household member taps the "Tonight: ..." nudge for a manually built week | Scrolled/highlighted to tonight's slot within the current week |
| FEAT-13.SPEC-004 (Same-Day Swap Correction Message) | Household member taps the same-day correction for a manually built week | Scrolled/highlighted to tonight's slot, showing the new dinner |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen: all seven nights, budget total, schedule | Pick an empty night, change or clear a picked night, open a pending suggestion for review | -- |
| Sam (Other Adult Member) | Full screen: all seven nights, budget total, schedule | Tap a night to suggest a pick (FEAT-23.SPEC-002); cannot pick, change, or clear directly | Change/Clear controls are not shown to Sam on any night; a night he taps opens the suggestion flow (FEAT-23.SPEC-003) instead of the picker |
| Jordan (young kid profile, no login -- MVP) | No | No | No login exists for this profile; the screen is not reachable |
| Jordan (older kid, limited login -- Later) | No | No | Manual Planning access is None for this row; navigation to this screen is not offered |
| Riley (Operator, support -- from v1) | No | No | Manual Planning access is None for Riley; this screen carries no support-view entry point |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on this screen if it was their original destination |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no in-progress edits exist on this screen to preserve (edits happen on FEAT-23.SPEC-002/003), so nothing is lost |

## Layout and Content

**Header:** The week's label (e.g., the calendar dates it covers) alongside the household's running estimated cost for the week ("estimated_total") shown against its configured weekly_budget, in the household's currency (FEAT-16). Below the header, a short line surfaces the household's weekly_schedule where a night is time-constrained (e.g., a 30-minute-weeknight marker on the affected night rows).

**Body:** Seven night rows, Monday through Sunday, each showing:
- The night's day label
- If picked: the recipe name, cook_time, rough_cost, and the "checked against allergies" safety_badge with its "always check labels" disclaimer
- If nothing is planned: a plain "Nothing planned" marker and (Maya only) a "Pick a dinner" action
- If Sam has an open suggestion pending for that night (cross-feature, FEAT-04): a "Suggestion pending" badge, visible to both Maya and Sam
- If the night was reopened because a hard dietary rule tightened mid-week (XBR-02): a "No longer safe -- pick again" marker in place of the prior pick

Maya sees, on a picked night, a "Change" action and a "Clear" action alongside the pick. Sam sees no Change/Clear controls on any night; tapping a night takes him into the suggestion flow.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** The seven night rows stack vertically, full width, in day order. The header's budget total sits directly under the week label.
- **Medium size class and above:** The same seven rows remain vertically stacked (one primary list, no multi-column restructuring); the header's week label and budget total sit side by side instead of stacked.
- **Recipe/cook-time/cost line within a picked-night row:** Wraps to a second line at the compact breakpoint; stays on one line at medium and above.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Empty night row (Maya) | Tap | Navigate to FEAT-23.SPEC-002 (Pick / Change a Recipe) with the target night in context | Screen transitions | Standard navigation transition |
| Picked night's "Change" action (Maya) | Tap | Navigate to FEAT-23.SPEC-002 with the target night and its existing pick pre-loaded | Screen transitions | Standard navigation transition |
| Picked night's "Clear" action (Maya) | Tap | Confirmation dialog appears; on confirm, triggers FEAT-23.SPEC-006 (Apply Manual Pick) delete path for that night | Dialog opens, then closes on confirm | Dialog text: "Remove this dinner from {Night}?" with "Remove" and "Keep It" options; on success the row updates to "Nothing planned" |
| Night row with no open suggestion (Sam) | Tap | Navigate to FEAT-23.SPEC-003 (Suggest a Pick) with the target night in context | Screen transitions | Standard navigation transition |
| Night row with Sam's own open suggestion (Sam) | Tap | Displays the suggestion's status inline; does not navigate | Row expands to show status | Text: "Suggestion sent -- waiting on {Organiser's name}." |
| "Suggestion pending" badge (Maya) | Tap | Navigate to FEAT-04's suggestion review for that suggestion | Screen transitions to FEAT-04 | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Header (week label, budget total) -> night rows in day order (Monday through Sunday) -> each row's available actions (Pick, or Change then Clear, or the pending-suggestion badge) in that order.
- **Dynamic-change announcements:** When a night's state changes (a pick lands, a clear completes, a suggestion badge appears or clears), the updated row content is announced to assistive technology.
- **Confirmation dialog:** The Clear confirmation dialog traps focus while open and is announced on appearance; dismissing it (either option) returns focus to the row's Clear action.
- **Keyboard alternatives:** Every row action (Pick, Change, Clear, suggestion badge) is reachable and operable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (first open) | Seven nights auto-initialize to "Nothing planned"; a prompt invites picking the first dinner | Organiser opens a week that has not started (no existing Weekly Plan for it) | A night is picked, or the organiser navigates away |
| Loading | Each night row shows an inline loading indicator until its data arrives | Screen first opens or is refreshed | Week data (picks, budget total) finishes loading |
| Error | An inline "Couldn't remove -- retry" control appears on the affected night row; the prior pick remains visible | A Clear action's save fails | User taps Retry (succeeds) or navigates away |
| Offline/Degraded | The current week's picks and budget total remain fully visible; Pick, Change, and Clear controls show "Connect to make changes" and are disabled | Connectivity is lost while the screen is open | Connectivity is restored -- controls re-enable immediately |
| Unsafe-Reopened (per night) | The affected night shows "No longer safe -- pick again" in place of its prior recipe, with a path into FEAT-23.SPEC-002 to choose a safe alternative | A household member's hard dietary rule newly fails a placed night's recipe (XBR-02, run by FEAT-02) | The night is re-picked with a safe recipe via FEAT-23.SPEC-002 |

## Validation Rules

Validation governed by FEAT-23.SPEC-005 (Manual Planning Validation & Limits) -- the one-dinner-per-night, seven-nights-per-week, and one-week-ahead rules that bound what this screen can show and offer. Safety eligibility for any recipe placed through this screen's actions is governed by FEAT-23.SPEC-004 (Safe-Choice Filtering & Placement Block).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Tap an empty night (Maya) | FEAT-23.SPEC-002 (Pick / Change a Recipe) | -- |
| Tap "Change" on a picked night (Maya) | FEAT-23.SPEC-002 (Pick / Change a Recipe) | -- |
| Tap a night with no open suggestion (Sam) | FEAT-23.SPEC-003 (Suggest a Pick) | -- |
| Tap the "Suggestion pending" badge (Maya) | Suggestion review | FEAT-04 (One-Tap Meal Swap) |

## Data Model

**Creates:** Weekly Plan -- auto-initialized with origin "manually built" and seven empty night slots the first time the organiser opens a week that has not started.
**Reads:** Weekly Plan (week, estimated_total, over_budget_note); Planned Meal (night, recipe, cook_time, rough_cost, safety_badge, status -- one per planned night); Household (weekly_budget, weekly_schedule, currency, unit_system -- for the header display).
**Updates:** None directly -- estimated_total is recalculated by FEAT-23.SPEC-006 whenever a pick, change, or clear completes; this screen displays the result.
**Deletes:** None directly -- the Clear action triggers the delete path of FEAT-23.SPEC-006.

## Business Rules

- XBR-01: every recipe shown as placed on this screen has already passed the same fail-closed allergy/religious-rule check as every other path onto the plan; the safety badge and "always check labels" disclaimer are shown on every picked night.
- XBR-03: every pick, change, or clear applied from this screen (via FEAT-23.SPEC-006) recalculates the shared grocery list (FEAT-06) immediately -- no member ever sees this week's plan without its matching list.
- XBR-11: the estimated_total and each night's rough_cost display in the household's configured currency and unit system (FEAT-16); a later locale change converts existing figures for display rather than leaving them inconsistent.
- FEAT-23.SPEC-005 governs which weeks and nights this screen can offer for picking (one week ahead, one dinner per night); this screen never offers a way to plan beyond that window.

## Edge Cases

- **Maya has this week open on two devices and clears the same night from both** -- The first clear commits; the second device's clear request is rejected once it arrives, and that device's view refreshes to show the night already empty. Resolution: reject-with-refresh, per the dependency map's Contention note for Planned Meal.
- **A household's hard dietary rule tightens mid-week and an already-placed pick now fails (XBR-02)** -- The affected night immediately shows "No longer safe -- pick again"; Maya is told a meal was removed from the plan (per FEAT-01/FEAT-02 communications) and can pick a safe alternative through FEAT-23.SPEC-002.
- **Maya taps Clear twice rapidly on the same night** -- The second tap has no effect while the first removal is in progress; no duplicate removal request is sent, and the Clear control shows its in-progress state until the first request resolves.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-002 (Pick / Change a Recipe) | Navigation (outbound) | Tapping an empty night or a picked night's Change action opens the picker with the night in context |
| FEAT-23.SPEC-003 (Suggest a Pick) | Navigation (outbound) | Sam tapping a night with no open suggestion opens the suggestion flow |
| FEAT-23.SPEC-006 (Apply Manual Pick) | Triggers (outbound) | The Clear action's confirmed delete path triggers this automation |
| FEAT-23.SPEC-004 (Safe-Choice Filtering & Placement Block) | References (inbound) | Every displayed pick has already passed this spec's safety filter |
| FEAT-23.SPEC-005 (Manual Planning Validation & Limits) | References (inbound) | Governs which week and which nights this screen can display and offer |
| FEAT-04 (One-Tap Meal Swap) | Navigation (outbound) | Tapping a pending-suggestion badge opens FEAT-04's suggestion review |
| FEAT-01 (Household Setup & Member Profiles) | References (inbound) | Supplies the weekly_budget and weekly_schedule shown at the top of the week |
| FEAT-02 (Dietary Rules & Allergy Safety Engine) | References (inbound) | Runs the mid-week re-check (XBR-02) that can reopen an already-placed night |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| manual_week_started | week reference, origin ("manually built") | A Weekly Plan is auto-initialized on this screen's first open for a week that has not started | supports success-metrics.md: "Manual Week Completion" |

## Acceptance Criteria

**FEAT-23.SPEC-001-AC-01:** Given Maya opens next week's plan for the first time, when the screen loads, then a Weekly Plan auto-initializes with origin "manually built" and all seven nights show "Nothing planned."

**FEAT-23.SPEC-001-AC-02:** Given Maya is on the Weekly Plan screen, when she taps Wednesday's empty night, then she is taken to FEAT-23.SPEC-002 with Wednesday as the target night.

**FEAT-23.SPEC-001-AC-03:** Given Maya is on the Weekly Plan screen with Friday already picked, when she taps "Change" on Friday, then she is taken to FEAT-23.SPEC-002 with Friday's existing pick pre-loaded.

**FEAT-23.SPEC-001-AC-04:** Given Maya is on the Weekly Plan screen with Monday already picked, when she taps "Clear" on Monday and confirms "Remove," then FEAT-23.SPEC-006's delete path runs and Monday shows "Nothing planned."

**FEAT-23.SPEC-001-AC-05:** Given Sam is on the Weekly Plan screen with no open suggestion for Tuesday, when he taps Tuesday's night row, then he is taken to FEAT-23.SPEC-003 to suggest a pick for Tuesday.

**FEAT-23.SPEC-001-AC-06:** Given Sam already has an open suggestion for Saturday, when he taps Saturday's night row, then the row shows "Suggestion sent -- waiting on Maya" instead of opening the picker.

**FEAT-23.SPEC-001-AC-07:** Given Maya's connection is slow while the week loads, when the screen first renders, then each night row shows an inline loading indicator until its data arrives.

**FEAT-23.SPEC-001-AC-08:** Given Maya's Clear action fails to save, when the failure occurs, then Monday's dinner remains visible with an inline "Couldn't remove -- retry" control, and the dinner is not silently dropped.

**FEAT-23.SPEC-001-AC-09:** Given Maya loses connectivity while viewing the week, when she looks at any night's Pick, Change, or Clear controls, then they show "Connect to make changes" and are disabled, while the week's existing picks and budget total remain fully visible.

**FEAT-23.SPEC-001-AC-10:** Given the household's weekly_budget is set and two dinners are picked, when Maya views the week, then the header shows the running estimated_total against the weekly_budget in the household's configured currency.

**FEAT-23.SPEC-001-AC-11:** Given Maya has this week open on two devices and clears Thursday's dinner on one device, when the same clear is attempted from the second device after the first commits, then the second request is rejected and that device's view refreshes to show Thursday already empty.

**FEAT-23.SPEC-001-AC-12:** Given a household member's allergy is tightened mid-week and Wednesday's placed dinner now fails the check, when Maya opens the week, then Wednesday shows "No longer safe -- pick again" and offers a path to a safe alternative via FEAT-23.SPEC-002.

**FEAT-23.SPEC-001-AC-13:** Given Sam has an open suggestion for Saturday, when Maya taps the "Suggestion pending" badge on Saturday, then she is taken to FEAT-04's suggestion review to accept or decline it.

**FEAT-23.SPEC-001-AC-14:** Given Maya clears Thursday's dinner, when FEAT-23.SPEC-006 completes the removal, then the shared grocery list (FEAT-06) recalculates to remove Thursday's ingredients (XBR-03).

**FEAT-23.SPEC-001-AC-15:** Given the household is currently working on the next plannable week, when Maya looks for a way to plan two weeks ahead from this screen, then no such option exists anywhere on the screen (validation governed by FEAT-23.SPEC-005).

**FEAT-23.SPEC-001-AC-16:** Given Maya taps Clear twice rapidly on Friday's dinner, when the first tap begins removing it, then the second tap has no effect and no duplicate removal request is sent.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 5 (empty/first-open, loading, error, offline, unsafe-reopened) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 3 | 3 |
