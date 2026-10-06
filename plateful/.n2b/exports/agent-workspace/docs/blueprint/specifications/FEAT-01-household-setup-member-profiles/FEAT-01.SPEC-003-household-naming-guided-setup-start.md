---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-003
spec_name: Household Naming & Guided Setup Start
spec_slug: household-naming-guided-setup-start
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Screen Spec: Household Naming & Guided Setup Start

## Overview

**Name:** Household Naming & Guided Setup Start
**ID:** FEAT-01.SPEC-003
**Type:** Screen
**Purpose:** The organiser names the household, becomes its first member, and enters the guided setup flow.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Naming the household and creating the Household record
- Creating the organiser's own Member Profile as the household's first member
- Introducing the guided setup wizard shell that carries through the following setup screens
- Triggering the default free-tier Subscription on household creation

**Non-Goals:**
- Adding other household members -- handled by FEAT-01.SPEC-004 (Member List & Add Member), the next guided-setup step
- Editing an existing household's name after setup -- handled by FEAT-01.SPEC-010 (Household Settings Hub); this screen only covers the one-time creation moment
- Choosing units, currency, or aisle layout -- owned by FEAT-16 (Units, Currency & Locale Configuration), reached later in the same guided setup journey (per the Internal Dependency Map's step 4)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | New account created, or sign-in for an account with no household yet | None -- form starts empty |
| FEAT-01.SPEC-002 (Password Recovery) | Reset completes for a household-less account | None |
| FEAT-01.SPEC-010 (Household Settings Hub) | Organiser taps to edit the household name later | Existing household_name pre-filled; screen behaves in edit mode (see States) |
| FEAT-24 (Invite Another Household) | Referred visitor completes account creation and starts their own household | Referring household's Household Referral record is created once this step completes |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Name/rename the household, proceed into guided setup | -- |
| Sam (Other Adult Member) | No | No | Household naming is not exposed to other adult members; Sam has View-only access to household facts once they exist (FEAT-01.SPEC-010) |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- household setup is not part of the older-kid login's entitlements |
| Riley (Operator, support) | View only, and only through FEAT-22 (from v1) | No | Riley never reaches this screen directly; household facts are visible only inside the separate read-only support view |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Entered household name is preserved locally (FEAT-01.SPEC-013) and restored after re-authentication |

## Layout and Content

**Header:** Guided setup wizard shell (shared with FEAT-01.SPEC-004 through FEAT-01.SPEC-009, continuing through FEAT-16.SPEC-001 and FEAT-16.SPEC-002 between the budget step and setup completion): step indicator "Step 1 of 8," screen title "Name your household."

**Body:** A single field, "Household name" (text input, required), with helper text: "This is what your household will be called inside Plateful -- you and everyone you invite will see it." Below the field, a "Continue" button.

In edit mode (arrived from FEAT-01.SPEC-010), the wizard chrome and step indicator are omitted; only the field, pre-filled with the current name, and a "Save" button appear, with a "Cancel" link returning to the Settings Hub.

**Footer:** None in guided-setup mode (Continue is the sole action); in edit mode, "Cancel" sits beside "Save."

### Responsive Behavior

- **Compact breakpoint:** Field and button stack full width.
- **Medium size class and above:** Content caps at the platform-wide narrow form width, horizontally centered; step indicator remains at the top, unchanged in structure.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Household name field | Type | Captures the household name | Field shows entered text; draft saved locally per FEAT-01.SPEC-013 | Standard input focus state |
| Household name field | Blur | Validates via FEAT-01.SPEC-014 | Error state if invalid | "Household name must be between 1 and 60 characters" below the field |
| "Continue" button (guided setup) | Tap | 1. Validate the name. 2. Create the Household record with the organiser as its first Member Profile. 3. Trigger FEAT-01.SPEC-011 (Default Subscription Provisioning). 4. The completed household creation triggers FEAT-24.SPEC-004 (Household Referral Recording), which records a referral only when referral context is present. | Button shows inline saving confirmation, not a full-page spinner | Success: navigates to FEAT-01.SPEC-004 (Member List & Add Member). Failure: inline error, entered name retained |
| "Save" button (edit mode) | Tap | Validate and update the Household's household_name field | Inline saving confirmation | Success: toast "Household name updated," returns to FEAT-01.SPEC-010. Failure: inline error, entered name retained |
| "Cancel" link (edit mode) | Tap | Discards the in-progress edit | Screen closes | Returns to FEAT-01.SPEC-010 without saving |

### Accessibility Notes

- **Focus order:** Household name field -> Continue/Save button (-> Cancel link, edit mode only).
- **Validation announcements:** The name-length error is announced and associated with the field when it appears.
- **Save feedback:** The "Household name updated" toast (edit mode) is announced on success.
- **Keyboard alternatives:** All actions are keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (guided setup, default) | Field empty, Continue enabled | First-time arrival at this step | User begins typing |
| Filling | Field contains entered text | User types | User taps Continue/Save or navigates away |
| Saving | Inline saving confirmation shown, field disabled briefly | User taps Continue/Save with valid input | Save completes or fails |
| Error | Failed field highlighted with error message below it | Validation fails or save fails | User corrects the field and retries |
| Edit (pre-filled) | Field pre-filled with current household_name | Arrived from FEAT-01.SPEC-010 | User saves, cancels, or navigates away |
| Offline/Degraded | Banner: "You're offline -- this will save when you reconnect." Field remains editable; Continue/Save queues the change locally per FEAT-01.SPEC-013 | Connectivity lost while this screen is open | Connectivity restored -- queued save submits automatically and standard success feedback appears |

## Validation Rules

Validation governed by FEAT-01.SPEC-014 (Household & Member Field Validation Rules). See that spec for the household name length rule. Checked on field blur and on submission.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Successful household creation (guided setup) | FEAT-01.SPEC-004 (Member List & Add Member) | -- |
| Successful save (edit mode) | FEAT-01.SPEC-010 (Household Settings Hub) | -- |
| Cancel (edit mode) | FEAT-01.SPEC-010 (Household Settings Hub) | -- |

## Data Model

**Creates:** Household -- household_name set from form input; organiser field set to the signed-in account's new Member Profile; weekly_budget, weekly_schedule, unit_system, currency, aisle_names, and plan_arrival_day_time left at their defaults or unset until later setup steps; status set to Active. Member Profile -- display_name (captured here as part of household creation, drawn from the organiser's account), member_type set to Organiser, status set to Active.
**Reads:** In edit mode, Household.household_name.
**Updates:** In edit mode, Household.household_name only.
**Deletes:** None.

## Business Rules

- Household creation is a one-time event per account (scope-boundaries.md SC-03) -- an account with a household already never sees this screen again except in edit mode.
- Household creation immediately triggers FEAT-01.SPEC-011 (Default Subscription Provisioning) so the household is never left without a Subscription record, even momentarily.
- If this creation completed via a household referral link (FEAT-24), the corresponding Household Referral record is attributed at this moment, per XBR-20.
- A failed save keeps the entered data on screen with a retry option -- nothing is silently discarded, per this feature's States field commitment.

## Edge Cases

- **User leaves the household name field empty and taps Continue** -- Error: "Household name must be between 1 and 60 characters"; household is not created.
- **User taps Continue twice rapidly** -- Second tap is ignored while the first creation request is in progress.
- **Network failure during household creation** -- Error banner with retry; the household name is preserved on screen and no partial Household record is left behind.
- **Organiser edits the household name from the Settings Hub while offline** -- The name change queues locally and saves automatically once connectivity returns, per FEAT-01.SPEC-013.
- **Concurrent edit from two devices (Maya signed in on laptop and phone)** -- Save is accepted on a last-write-wins basis per the dependency map's Contention note for Household: the most recently saved name wins, and a failed save on either device keeps the household's prior name active rather than leaving a mixed state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) | Navigation (inbound) | New or household-less accounts arrive here |
| FEAT-01.SPEC-004 (Member List & Add Member) | Navigation (outbound) | Next step in guided setup |
| FEAT-01.SPEC-010 (Household Settings Hub) | Navigation (inbound/outbound) | Edit-mode entry and return |
| FEAT-01.SPEC-011 (Default Subscription Provisioning) | Triggers (outbound) | Household creation provisions the free-tier Subscription |
| FEAT-01.SPEC-013 (Setup Draft Persistence & Offline Queuing) | References (inbound) | Draft saving and offline queuing behavior |
| FEAT-01.SPEC-014 (Household & Member Field Validation Rules) | References (inbound) | Household name validation |
| FEAT-01.SPEC-016 (Household Setup Authorization Rules) | References (inbound) | Role-gated access to this screen |
| FEAT-24 (Invite Another Household) | Navigation (inbound) | Referral welcome page leads here via account creation |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| household_created | entry source (default / referral) | Household record is successfully created | supports success-metrics.md: "First-Session Onboarding Completion" |
| household_name_edited | -- | Organiser saves a name change from edit mode | N/A -- this event is a later edit, not part of the first-session onboarding window the connected metric measures |

## Acceptance Criteria

**FEAT-01.SPEC-003-AC-01:** Given Maya has just created her account, when she enters "The Rossi Household" and taps "Continue", then the Household is created with her as its first member and she is taken to FEAT-01.SPEC-004 (Member List & Add Member).

**FEAT-01.SPEC-003-AC-02:** Given Maya leaves the household name field empty, when she taps "Continue", then the error "Household name must be between 1 and 60 characters" appears and no household is created.

**FEAT-01.SPEC-003-AC-03:** Given Maya's household is created, then FEAT-01.SPEC-011 (Default Subscription Provisioning) fires automatically and provisions a free-tier Subscription.

**FEAT-01.SPEC-003-AC-04:** Given Maya opens this screen from FEAT-01.SPEC-010 to rename her household, when she changes the name and taps "Save", then the household's name updates and she sees the toast "Household name updated," returning to the Settings Hub.

**FEAT-01.SPEC-003-AC-05:** Given Maya is in edit mode with unsaved changes, when she taps "Cancel", then she returns to FEAT-01.SPEC-010 without the household name being changed.

**FEAT-01.SPEC-003-AC-06:** Given Maya loses connectivity while entering the household name, when she taps "Continue", then the banner "You're offline -- this will save when you reconnect." appears and the creation is queued locally.

**FEAT-01.SPEC-003-AC-07:** Given Maya is signed in on both a laptop and a phone, when she saves a different household name from each device within moments of each other, then the most recently saved name is the one that persists, per last-write-wins.

**FEAT-01.SPEC-003-AC-08:** Given a visitor arrived via a household referral link and creates their account, when they complete this screen, then a Household Referral record attributing their new household to the referring household is created per XBR-20.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 (empty, filling, saving, error, edit, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
