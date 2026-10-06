---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-007
spec_name: Parental Consent Confirmation
spec_slug: parental-consent-confirmation
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 7
---

# Screen Spec: Parental Consent Confirmation

## Overview

**Name:** Parental Consent Confirmation
**ID:** FEAT-01.SPEC-007
**Type:** Screen
**Purpose:** The organiser confirms parent-or-guardian status and sees exactly what a kid profile stores before it is created.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Disclosing exactly what data a kid profile stores before any of it is entered
- Requiring an explicit affirmative confirmation that the organiser is the child's parent or guardian
- Blocking kid-profile creation until this confirmation is given

**Non-Goals:**
- Collecting the kid profile's actual data (name, age band, dietary rules) -- handled by FEAT-01.SPEC-005 and FEAT-01.SPEC-006, reached only after this confirmation
- Any legal or identity verification of parental status beyond the organiser's own affirmative confirmation -- product-features.md defines no identity-verification capability; the product relies on the organiser's stated confirmation, consistent with a household-trust model rather than a verification service
- Consent for an existing kid profile's continued data use -- this screen is a one-time gate at creation; ongoing data handling is governed by this feature's Data Notes and assumptions-constraints.md, not re-confirmed here

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-004 (Member List & Add Member) | Organiser taps "Add a kid profile" | None -- confirmation is requested before any kid data is entered |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | Confirm parent/guardian status and proceed, or cancel | -- |
| Sam (Other Adult Member) | No | No | This screen is reached only through the organiser's own "Add a kid profile" action; Sam has no entry point to it |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- household setup is not part of the older-kid login's entitlements |
| Riley (Operator, support) | No | No | N/A -- this is a one-time creation gate with no ongoing data to view; it is never exposed through operator support access |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." The intent to add a kid profile is not preserved as draft data (nothing has been entered yet); the organiser restarts from FEAT-01.SPEC-004 after re-authenticating |

## Layout and Content

**Header:** Title "Before you add a kid profile."

**Body:** A disclosure block listing exactly what will be stored, as a short bulleted list: "A first name or nickname," "An age band," "Their dietary rules (allergies, religious rules, vegetarian setting, dislikes)." Directly beneath, a plain statement: "We never store a surname, birth date, photo, or contact detail for a kid profile." Below that, a single checkbox with the label: "I confirm I am this child's parent or legal guardian." A "Continue" button, disabled until the checkbox is checked.

**Footer:** "Cancel" link, returning to FEAT-01.SPEC-004 without creating anything.

### Responsive Behavior

- **Compact breakpoint:** Disclosure list and checkbox stack full width; Continue and Cancel stack full width.
- **Medium size class and above:** Content caps at the platform-wide narrow form width, horizontally centered; Continue and Cancel appear side by side.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Confirmation checkbox | Tap | Toggles the confirmation | Checkbox shows checked/unchecked; Continue enables/disables accordingly | Immediate visual toggle |
| "Continue" button | Tap (enabled only when checkbox is checked) | Records parental_consent_confirmation as true for the profile about to be created and proceeds | Screen changes | Navigates to FEAT-01.SPEC-005 in create mode, member_type pre-set to Kid |
| "Cancel" link | Tap | Discards the in-progress kid-add intent | Screen closes | Returns to FEAT-01.SPEC-004; nothing is created |

### Accessibility Notes

- **Focus order:** Disclosure list (read order) -> Confirmation checkbox -> Continue -> Cancel.
- **Disabled-state announcement:** The "Continue" button's disabled state is announced along with the reason ("Continue is disabled until you confirm parent or guardian status") so the requirement is not silently invisible to assistive technology.
- **Keyboard alternatives:** All actions are keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Unconfirmed (default) | Checkbox unchecked, Continue disabled | Screen first opens | Checkbox is checked |
| Confirmed | Checkbox checked, Continue enabled | User checks the box | User taps Continue or unchecks the box |
| Offline/Degraded | N/A -- confirming and proceeding requires no network request; this screen only sets local state carried into FEAT-01.SPEC-005, whose own save is what queues offline | Connectivity lost while this screen is open | Connectivity has no effect on this screen's own function |

## Validation Rules

**Option B -- Inline (simple validation not warranting a standalone spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Confirmation checkbox | Must be checked before proceeding | On "Continue" tap (button is disabled otherwise, so no error state is reachable) | N/A -- the disabled button state itself prevents an invalid submission |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Continue" tap (confirmed) | FEAT-01.SPEC-005 (Member Profile Detail, create mode, Kid) | -- |
| "Cancel" tap | FEAT-01.SPEC-004 (Member List & Add Member) | -- |

## Data Model

**Creates:** None directly -- this screen sets the parental_consent_confirmation value carried into the Member Profile that FEAT-01.SPEC-005 creates immediately afterward.
**Reads:** None.
**Updates:** None.
**Deletes:** None.

## Business Rules

- A kid profile cannot exist without this confirmation -- FEAT-01.SPEC-005 cannot reach its Kid create mode by any path other than through this screen.
- The disclosure text on this screen is the authoritative statement of what a kid profile stores; FEAT-01.SPEC-005 and FEAT-01.SPEC-006 must not collect any field beyond what is disclosed here (enforced by FEAT-01.SPEC-014's kid-profile data-minimality rule).
- This screen is shown every time a kid profile is added, not only the first time -- each kid profile gets its own explicit confirmation.

## Edge Cases

- **Organiser checks the box, then unchecks it before tapping Continue** -- "Continue" disables again immediately; no state persists from the momentary check.
- **Organiser navigates away after checking the box but before tapping Continue** -- No confirmation dialog is needed; nothing has been created or saved, so there is no unsaved-data loss to warn about.
- **Organiser adds a second kid profile in the same session** -- This screen is shown again in full, from Unconfirmed, for the new profile; a prior confirmation for a different child does not carry over.
- **Double-tap on "Continue"** -- Second tap is ignored once navigation to FEAT-01.SPEC-005 has begun.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-004 (Member List & Add Member) | Navigation (inbound/outbound) | Entry via "Add a kid profile"; Cancel returns here |
| FEAT-01.SPEC-005 (Member Profile Detail) | Navigation (outbound) | Confirmed consent proceeds to kid profile creation |
| FEAT-01.SPEC-014 (Household & Member Field Validation Rules) | References (outbound) | The kid-profile data-minimality rule this screen's disclosure reflects |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| kid_profile_consent_confirmed | -- | Organiser taps "Continue" with the checkbox checked | supports success-metrics.md: "First-Session Onboarding Completion" |
| kid_profile_consent_cancelled | -- | Organiser taps "Cancel" | N/A -- no Stage 2 metric measures abandoned kid-profile starts; retained as a diagnostic signal for setup friction |

## Acceptance Criteria

**FEAT-01.SPEC-007-AC-01:** Given Maya taps "Add a kid profile" from FEAT-01.SPEC-004, when this screen loads, then she sees the exact disclosure list of what will be stored before any kid data field is shown.

**FEAT-01.SPEC-007-AC-02:** Given Maya has not checked the confirmation checkbox, when she looks at "Continue", then it is disabled.

**FEAT-01.SPEC-007-AC-03:** Given Maya checks "I confirm I am this child's parent or legal guardian" and taps "Continue", then she is taken to FEAT-01.SPEC-005 in create mode with member_type set to Kid, and the profile's parental_consent_confirmation will be recorded as true.

**FEAT-01.SPEC-007-AC-04:** Given Maya taps "Cancel" before confirming, then she returns to FEAT-01.SPEC-004 and no kid profile is created.

**FEAT-01.SPEC-007-AC-05:** Given Maya has already added one kid profile in this session, when she starts adding a second, then this screen shows again from its Unconfirmed state, requiring a fresh confirmation.

**FEAT-01.SPEC-007-AC-06:** Given Maya checks the box and then unchecks it, when she looks at "Continue", then it is disabled again.

**FEAT-01.SPEC-007-AC-07:** Given Maya taps "Continue" twice in rapid succession after confirming, then only one navigation to FEAT-01.SPEC-005 occurs.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 3 (unconfirmed, confirmed, offline N/A) | 3 |
| Business Rules | 3 | 3 |
| Edge Cases | 4 | 4 |
