---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-005
spec_name: Member Profile Detail
spec_slug: member-profile-detail
parent_feature: FEAT-01
parent_feature_name: Household Setup & Member Profiles
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Screen Spec: Member Profile Detail

## Overview

**Name:** Member Profile Detail
**ID:** FEAT-01.SPEC-005
**Type:** Screen
**Purpose:** The organiser creates or edits one member's basic details -- name, type, age band, and notification preferences; other adult members view a kid profile's details.
**Parent Feature:** FEAT-01 -- Household Setup & Member Profiles

## Scope and Non-Goals

**In Scope:**
- Creating a new adult Member Profile (kid profiles arrive here only after FEAT-01.SPEC-007 confirms consent)
- Editing an existing member's display name, member type details, age band, and notification preferences
- Read-only viewing of a kid profile's details for Sam
- Entry point into FEAT-01.SPEC-006 (Dietary Rules Editor) for the member being viewed or edited

**Non-Goals:**
- Setting or editing dietary rules -- handled by FEAT-01.SPEC-006 (Dietary Rules Editor), reached from this screen
- Parental consent confirmation itself -- handled by FEAT-01.SPEC-007, which this screen is routed through when the member type is a kid profile
- Removing a member -- owned by Account & Data Management (FEAT-18); this screen creates and edits, never deletes

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-004 (Member List & Add Member) | Organiser taps "Add an adult" | member_type pre-set to Other Adult Member; screen opens in create mode, empty |
| FEAT-01.SPEC-004 (Member List & Add Member) | Organiser or Sam taps an existing member's card | The member's current data, loaded editable for the organiser or read-only for Sam viewing a kid profile |
| FEAT-01.SPEC-007 (Parental Consent Confirmation) | Organiser confirms consent for a new kid profile | member_type pre-set to Kid; screen opens in create mode with the consent already confirmed |
| FEAT-01.SPEC-010 (Household Settings Hub) | Any adult taps "Notification preferences" | The signed-in adult's own Member Profile, scrolled to the Notification preferences toggles |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen for every member, including create mode | Create, edit any member's display name, type details, age band, and notification preferences | -- |
| Sam (Other Adult Member) | Full screen, read-only, for kid profiles; his own detail shows his own notification preferences as editable | View kid profiles; edit only his own notification preferences (per FEAT-01.SPEC-016) | Attempting to edit a kid profile's fields shows the fields as non-interactive with the note "Only the organiser can change member details" |
| Jordan (young kid profile, no login -- MVP) | No | No | N/A -- no login exists for this profile type |
| Jordan (older kid, limited login -- Later) | No | No | N/A -- household setup is not part of the older-kid login's entitlements |
| Riley (Operator, support) | View, only through FEAT-22 (from v1), and never kid profile data beyond a specific safety report (XBR-14) | No | Riley never reaches this screen directly |
| Unauthenticated | No | No | Redirected to FEAT-01.SPEC-001 (Account Sign-Up & Sign-In) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Unsaved edits are preserved locally per FEAT-01.SPEC-013 and restored after re-authentication |

## Layout and Content

**Header:** Title "{display name}" (or "New adult" / "New kid profile" in create mode), back arrow returning to FEAT-01.SPEC-004.

**Body:** A single-column form:
- Display name (text input, required)
- Member type (read-only label once set: "Organiser," "Other Adult Member," or "Kid" -- not editable after creation, since type determines the fields below)
- Age band (selection input, kid profiles only, required for kids)
- Notification preferences: "Plan-ready notifications" (toggle) and "Nightly nudge" (toggle) -- adults only, and each adult can edit only their own
- A "Dietary rules" summary row showing up to three rule badges and a "View/Edit dietary rules" link into FEAT-01.SPEC-006

For a kid profile, a note appears beneath the header: "Stored for {display name}: first name or nickname, age band, and dietary rules only," restating the data-minimality commitment from FEAT-01.SPEC-007.

**Footer:** "Save" button (create and edit modes); "Cancel" link discarding unsaved changes.

### Responsive Behavior

- **Compact breakpoint:** Fields stack full width; footer buttons stack full width.
- **Medium size class and above:** Form caps at the platform-wide narrow form width, horizontally centered; footer buttons appear side by side.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Display name field | Type | Captures the display name; draft saved locally per FEAT-01.SPEC-013 | Field shows entered text | Standard input focus state |
| Display name field | Blur | Validates via FEAT-01.SPEC-014 | Error state if invalid | "Enter a name (up to 60 characters)" below the field |
| Age band selector (kid profiles) | Select | Sets the age band | Selector shows chosen band | Selected band displayed |
| Notification toggles (own profile only) | Tap | Sets the toggle on/off | Toggle state changes visually | Immediate; no separate save step required for preference toggles once the profile already exists |
| "View/Edit dietary rules" link | Tap | Navigate to FEAT-01.SPEC-006 (Dietary Rules Editor) for this member | Screen changes | Standard transition |
| "Save" button | Tap | 1. Validate all fields via FEAT-01.SPEC-014. 2. Create or update the Member Profile. | Button shows inline saving confirmation | Success: returns to FEAT-01.SPEC-004 with the updated/new card visible. Failure: inline error, entered data retained |
| "Cancel" link | Tap | Discards unsaved changes | Screen closes | Confirmation dialog if changes were made; returns to FEAT-01.SPEC-004 |

### Accessibility Notes

- **Focus order:** Display name -> Member type (read-only, not focusable) -> Age band (kid profiles) -> Notification toggles (own profile) -> Dietary rules link -> Save -> Cancel.
- **Validation announcements:** Field errors are announced and associated with their field.
- **Read-only announcement (Sam viewing a kid profile):** Fields are announced as "read-only" to assistive technology so their non-interactive state is not silently assumed to be a bug.
- **Keyboard alternatives:** All actions are keyboard-reachable; no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Create (empty) | Form fields empty except member_type (pre-set), Save enabled once required fields are filled | Arrived from "Add an adult" or FEAT-01.SPEC-007 | User fills required fields and saves, or cancels |
| Edit (pre-filled) | Form fields show the member's current data | Arrived by tapping an existing member's card (organiser) | User saves or cancels |
| Read-only (Sam viewing a kid profile) | Fields shown as static text/labels, no inputs | Sam taps a kid profile's card | Sam navigates away |
| Saving | Inline saving confirmation, fields disabled briefly | User taps Save with valid input | Save completes or fails |
| Error | Failed field highlighted with error message; entered data retained | Validation fails or save fails | User corrects and retries |
| Offline/Degraded | Banner: "You're offline -- this will save when you reconnect." Fields remain editable; Save queues the change locally per FEAT-01.SPEC-013 | Connectivity lost while this screen is open | Connectivity restored -- queued save submits automatically |

## Validation Rules

Validation governed by FEAT-01.SPEC-014 (Household & Member Field Validation Rules). See that spec for display name length, age band requirement for kid profiles, and kid-profile data-minimality rules. Checked on field blur and on submission.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Successful save | FEAT-01.SPEC-004 (Member List & Add Member) | -- |
| Cancel | FEAT-01.SPEC-004 (Member List & Add Member) | -- |
| "View/Edit dietary rules" tap | FEAT-01.SPEC-006 (Dietary Rules Editor) | -- |

## Data Model

**Creates:** Member Profile -- display_name, member_type, age_band (kid profiles), status set to Active. For kid profiles, parental_consent_confirmation is already set to true by FEAT-01.SPEC-007 before this screen is reached in create mode.
**Reads:** Member Profile -- all fields, for the member being viewed/edited; Dietary Rule -- summary badges for the dietary rules row.
**Updates:** Member Profile -- display_name, age_band (kid profiles), notification_preferences (adults, own-only).
**Deletes:** None.

## Business Rules

- Member type cannot be changed after creation -- a profile created as a kid cannot become an adult or vice versa, since the two carry fundamentally different field sets and consent requirements; correcting a mis-typed profile requires removing it (FEAT-18) and re-adding it correctly.
- A kid profile's fields are restricted to display_name, member_type, age_band, dietary rules, and parental_consent_confirmation -- no surname, birth date, photo, or contact detail field exists on this screen for kid profiles, per FEAT-01.SPEC-014's data-minimality rule.
- Each adult edits only their own notification preferences (FEAT-01.SPEC-016) -- the organiser cannot set another adult's toggle on this screen.
- Authorization for who can create, view, or edit is governed by FEAT-01.SPEC-016 (Household Setup Authorization Rules).

## Edge Cases

- **Organiser leaves the display name empty and taps Save** -- Error: "Enter a name (up to 60 characters)"; profile is not created/saved.
- **Organiser attempts to add a 13th member** -- Blocked per FEAT-01.SPEC-014's 12-member cap, shown as an inline message on this screen before Save is enabled.
- **Organiser navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Sam attempts to edit a kid profile's display name via direct interaction** -- The field is non-interactive; no error is needed since there is no control to trigger one.
- **Concurrent edit from two devices (Maya on laptop and phone, editing the same member)** -- Save is rejected with reject-with-refresh, consistent with the dependency map's Contention note for Member Profile: dialog "This profile was updated from your other device. Review the latest version before saving." with "View Latest" and "Keep Editing" options.
- **Network failure during save** -- Error banner with retry; entered data preserved, no partial Member Profile is left behind.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-004 (Member List & Add Member) | Navigation (inbound/outbound) | Entry and return |
| FEAT-01.SPEC-006 (Dietary Rules Editor) | Navigation (outbound) | Dietary rules link |
| FEAT-01.SPEC-007 (Parental Consent Confirmation) | Navigation (inbound) | Kid profile creation arrives here after consent |
| FEAT-01.SPEC-013 (Setup Draft Persistence & Offline Queuing) | References (inbound) | Draft saving and offline behavior |
| FEAT-01.SPEC-014 (Household & Member Field Validation Rules) | References (inbound) | Field validation and the 12-member cap |
| FEAT-01.SPEC-016 (Household Setup Authorization Rules) | References (inbound) | Role-gated view/edit access |
| FEAT-07 (Weekly Plan Ready Notification) | References (outbound) | Plan-ready preference toggle exposed here |
| FEAT-13 (Tonight's Dinner Reminder) | References (outbound) | Nightly nudge preference toggle exposed here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| member_added | member type (adult / kid) | New Member Profile is successfully created | supports success-metrics.md: "First-Session Onboarding Completion" |
| member_profile_edited | fields changed (name / age band / notification preference) | An existing member's details are saved | N/A -- this event covers post-setup edits, outside the first-session onboarding window the connected metric measures |

## Acceptance Criteria

**FEAT-01.SPEC-005-AC-01:** Given Maya taps "Add an adult" from FEAT-01.SPEC-004, when she enters "Sam" and taps "Save", then a new Other Adult Member profile named "Sam" is created and she returns to the member list showing his card.

**FEAT-01.SPEC-005-AC-02:** Given Maya arrives here after confirming parental consent (FEAT-01.SPEC-007) for a kid, when she enters "Jordan" and selects an age band and taps "Save", then a new kid profile is created with only the name, member type, age band, and consent recorded.

**FEAT-01.SPEC-005-AC-03:** Given Maya leaves the display name empty, when she taps "Save", then the error "Enter a name (up to 60 characters)" appears and no profile is created.

**FEAT-01.SPEC-005-AC-04:** Given Sam taps Jordan's member card from the list, when the screen loads, then all fields render as read-only static text and no edit controls are shown.

**FEAT-01.SPEC-005-AC-05:** Given Sam opens his own member detail, when he toggles "Nightly nudge" off, then his own preference updates and Maya's preferences are unaffected.

**FEAT-01.SPEC-005-AC-06:** Given Maya is editing an existing member with unsaved changes, when she taps "Cancel", then a confirmation dialog "You have unsaved changes. Discard?" appears.

**FEAT-01.SPEC-005-AC-07:** Given Maya is signed in on two devices and edits the same member's age band from both within moments, when the second save is submitted, then it is rejected with "This profile was updated from your other device. Review the latest version before saving."

**FEAT-01.SPEC-005-AC-08:** Given Maya's household already has 12 members, when she attempts to save a 13th, then the 12-member-cap message from FEAT-01.SPEC-014 is shown and no profile is created.

**FEAT-01.SPEC-005-AC-09:** Given Maya loses connectivity while editing a member, when she taps "Save", then the banner "You're offline -- this will save when you reconnect." appears and the change is queued locally.

**FEAT-01.SPEC-005-AC-10:** Given Maya is viewing any member's detail, when she taps "View/Edit dietary rules", then she is taken to FEAT-01.SPEC-006 for that specific member.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 6 (create, edit, read-only, saving, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
