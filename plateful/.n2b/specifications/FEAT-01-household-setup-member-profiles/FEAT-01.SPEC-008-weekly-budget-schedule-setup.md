---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-008
spec_name: Weekly Budget & Schedule Setup
spec_slug: weekly-budget-schedule-setup
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Screen Spec: Weekly Budget & Schedule Setup

## Overview

**Name:** Weekly Budget & Schedule Setup
**ID:** FEAT-01.SPEC-008
**Type:** Screen
**Purpose:** The organiser states the weekly food budget and marks which nights are short on time.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Setting the household's weekly_budget in its configured currency
- Marking which nights of the week are time-constrained and the time limit for those nights
- Editing both later from FEAT-01.SPEC-010

**Non-Goals:**
- Setting the household's currency or unit system -- owned by FEAT-16 (Units, Currency & Locale Configuration), reached elsewhere in the guided setup journey; this screen only enters an amount in whatever currency is already configured
- Generating or evaluating a plan against the budget -- owned by FEAT-03 (AI Weekly Dinner Plan Generation) and FEAT-23 (Manual Weekly Planning), which read this data
- Per-night meal planning of any kind -- this screen only records which nights are time-constrained, not what is cooked on them

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-004 (Member List & Add Member) | Organiser continues from adding members | Guided-setup wizard context |
| FEAT-01.SPEC-010 (Household Settings Hub) | Organiser taps "Budget & Schedule" | Existing weekly_budget and weekly_schedule pre-filled; screen behaves in edit mode |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Set and edit the weekly budget and schedule | -- |
| Sam (Other Adult Member) | No | No | This screen is not exposed to Sam; the resulting budget and schedule facts are visible to him read-only through FEAT-01.SPEC-010 |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- household setup is not part of the older-kid login's entitlements |
| Riley (Operator, support) | View, only through FEAT-22 (from v1) | No | Riley never reaches this screen directly; budget and schedule facts are visible only inside the separate read-only support view |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Entered values are preserved locally per FEAT-01.SPEC-013 and restored after re-authentication |

## Layout and Content

**Header (guided setup):** Wizard shell, "Step 5 of 8," title "Budget and busy nights." **Header (standalone, from Settings Hub):** Title "Budget & Schedule," back arrow to FEAT-01.SPEC-010.

**Body:** Two sections.
- **Weekly budget:** A single numeric field labeled "Roughly how much do you want to spend on groceries each week?" showing the household's configured currency symbol, with helper text: "This is a rough guide, not a hard limit -- if no safe week fits, you'll see the closest option and how much it goes over."
- **Weekly schedule:** Seven toggles, one per day of the week (Monday through Sunday), each labeled with the day name and, when toggled on, a time-limit selector (e.g., "30 minutes," "45 minutes") appears beside it. Helper text above the toggles: "Mark any nights that are short on time -- we'll only suggest quick dinners for those."

**Footer:** "Continue" button (guided setup) or "Save" button (edit mode), plus "Cancel" link in edit mode.

### Responsive Behavior

- **Compact breakpoint:** Budget field full width; day toggles stack one per row, each with its time-limit selector beside it.
- **Medium size class and above:** Content caps at the platform-wide form width, horizontally centered; day toggles render in a compact grid of two columns.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Weekly budget field | Type | Captures the amount; draft saved locally per FEAT-01.SPEC-013 | Field shows entered value | Standard input focus state |
| Weekly budget field | Blur | Validates via FEAT-01.SPEC-014 (positive amount) | Error state if invalid | "Enter an amount greater than zero" below the field |
| Day toggle | Tap | Marks/unmarks that day as time-constrained | Toggle state changes; time-limit selector appears/disappears | Immediate |
| Time-limit selector (per toggled day) | Select | Sets the time limit for that day | Selector shows chosen limit | Selected value displayed |
| "Continue" button (guided setup) | Tap | 1. Validate the budget. 2. Save weekly_budget and weekly_schedule to the Household. | Inline saving confirmation | Success: navigates to FEAT-16.SPEC-001 (Units & Currency Settings), guided setup's next step. Failure: inline error, entered data retained |
| "Save" button (edit mode) | Tap | Validate and update the Household's weekly_budget and weekly_schedule fields | Inline saving confirmation | Success: toast "Budget and schedule updated," returns to FEAT-01.SPEC-010. Failure: inline error, entered data retained |
| "Cancel" link (edit mode) | Tap | Discards the in-progress edit | Screen closes | Returns to FEAT-01.SPEC-010 without saving |

### Accessibility Notes

- **Focus order:** Weekly budget field -> Day toggles in day order (each followed immediately by its own time-limit selector when visible) -> Continue/Save -> Cancel (edit mode only).
- **Validation announcements:** The budget error is announced and associated with its field.
- **Dynamic field announcement:** When a day's time-limit selector appears or disappears (toggle on/off), the change is announced so the appearing control is not missed.
- **Keyboard alternatives:** All actions are keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (guided setup, default) | Budget field empty, no days toggled | First-time arrival at this step | User enters a budget or toggles a day |
| Filling | Budget entered and/or one or more days toggled | User interacts with any field | User taps Continue/Save or navigates away |
| Saving | Inline saving confirmation, fields disabled briefly | User taps Continue/Save with valid input | Save completes or fails |
| Error | Failed field highlighted with error message; entered data retained | Validation fails or save fails | User corrects and retries |
| Edit (pre-filled) | Budget and schedule pre-filled with current values | Arrived from FEAT-01.SPEC-010 | User saves, cancels, or navigates away |
| Offline/Degraded | Banner: "You're offline -- this will save when you reconnect." Fields remain editable; Continue/Save queues the change locally per FEAT-01.SPEC-013 | Connectivity lost while this screen is open | Connectivity restored -- queued save submits automatically |

## Validation Rules

Validation governed by FEAT-01.SPEC-014 (Household & Member Field Validation Rules). See that spec for the budget positivity rule. Checked on field blur and on submission. The weekly schedule carries no validation beyond data type -- an empty schedule (no days marked) is a valid, deliberate statement of "no time constraint," per this feature's Primary Flows & Alternates.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Successful save (guided setup) | FEAT-16.SPEC-001 (Units & Currency Settings) | FEAT-16 (Units, Currency & Locale Configuration) |
| Successful save (edit mode) | FEAT-01.SPEC-010 (Household Settings Hub) | -- |
| Cancel (edit mode) | FEAT-01.SPEC-010 (Household Settings Hub) | -- |

## Data Model

**Creates:** None.
**Reads:** In edit mode, Household.weekly_budget and Household.weekly_schedule.
**Updates:** Household -- weekly_budget (a positive amount in the household's configured currency) and weekly_schedule (which nights are time-constrained and their time limit).
**Deletes:** None.

## Business Rules

- An absent schedule (no days marked) means no time constraint applies to any night -- not a hard block on proceeding, per this feature's Primary Flows & Alternates: "Partial setup."
- The weekly budget is a rough guide, not a hard limit; how a plan behaves when no safe week fits the budget is defined by FEAT-03, not this screen.
- Later edits to budget or schedule from FEAT-01.SPEC-010 apply to the next plan, not retroactively, per this feature's Primary Flows & Alternates.

## Edge Cases

- **Organiser leaves the budget field empty and taps Continue** -- Since weekly_budget is optional during partial setup (per this feature's Validation & Limits), Continue proceeds with no budget set rather than blocking; only a non-empty value must be positive.
- **Organiser enters a zero or negative budget** -- Error: "Enter an amount greater than zero"; the field is not saved until corrected.
- **Organiser toggles a day on, then off, without selecting a time limit** -- No time-constraint entry is saved for that day; toggling off discards any partially selected limit for it.
- **Organiser edits the schedule from the Settings Hub while offline** -- The change queues locally and saves automatically once connectivity returns, per FEAT-01.SPEC-013.
- **Concurrent edit from two devices (Maya on laptop and phone)** -- Last-write-wins per the dependency map's Contention note for Household; a failed save on either device keeps the household's prior budget/schedule active rather than leaving a mixed state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-004 (Member List & Add Member) | Navigation (inbound) | Guided setup continues here |
| FEAT-16.SPEC-001 (Units & Currency Settings) | Navigation (outbound) | Next guided-setup step (units, currency, and aisle layout, FEAT-16, before FEAT-01.SPEC-009) |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound/outbound) | Edit-mode entry and return |
| FEAT-01.SPEC-013 (Setup Draft Persistence & Offline Queuing) | References (inbound) | Draft saving and offline queuing |
| FEAT-01.SPEC-014 (Household & Member Field Validation Rules) | References (inbound) | Budget positivity rule |
| FEAT-01.SPEC-016 (Household Setup Authorization Rules) | References (inbound) | Role-gated access to this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| budget_set | amount provided (yes / no) | Household's weekly_budget is saved | supports success-metrics.md: "First-Session Onboarding Completion" |
| schedule_set | count of time-constrained nights | Household's weekly_schedule is saved | supports success-metrics.md: "First-Session Onboarding Completion" |

## Acceptance Criteria

**FEAT-01.SPEC-008-AC-01:** Given Maya is on this screen during guided setup, when she enters a weekly budget and toggles Monday and Wednesday as 30-minute nights and taps "Continue", then the Household's weekly_budget and weekly_schedule are saved and she is taken to FEAT-16.SPEC-001 (Units & Currency Settings), guided setup's next step.

**FEAT-01.SPEC-008-AC-02:** Given Maya enters "0" as her weekly budget, when she blurs the field, then the error "Enter an amount greater than zero" appears.

**FEAT-01.SPEC-008-AC-03:** Given Maya leaves the budget field empty and marks no nights, when she taps "Continue", then setup proceeds with no budget or schedule set, per the partial-setup allowance.

**FEAT-01.SPEC-008-AC-04:** Given Maya opens this screen from FEAT-01.SPEC-010 to change the budget, when she updates the amount and taps "Save", then the household's budget updates and she sees "Budget and schedule updated," returning to the Settings Hub.

**FEAT-01.SPEC-008-AC-05:** Given Maya toggles Friday on and then off again without selecting a time limit, when she saves, then Friday is not recorded as time-constrained.

**FEAT-01.SPEC-008-AC-06:** Given Maya is in edit mode with unsaved changes, when she taps "Cancel", then she returns to FEAT-01.SPEC-010 without the changes being saved.

**FEAT-01.SPEC-008-AC-07:** Given Maya loses connectivity while entering the budget, when she taps "Continue", then the banner "You're offline -- this will save when you reconnect." appears and the entry is queued locally.

**FEAT-01.SPEC-008-AC-08:** Given Maya is signed in on two devices and saves different budgets from each within moments, then the most recently saved value is the one that persists, per last-write-wins.

**FEAT-01.SPEC-008-AC-09:** Given Maya sets a weekly schedule marking three weeknights as time-constrained, then FEAT-03/FEAT-23 read that schedule when proposing dinners for those nights (verified by the schedule_set event and the saved weekly_schedule field).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 6 (empty, filling, saving, error, edit, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
