---
document_type: spec
spec_type: screen
spec_id: FEAT-18.SPEC-004
spec_name: My Account
spec_slug: my-account
parent_feature: FEAT-18
parent_feature_name: Account & Data Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 16
---

# Screen Spec: My Account

## Overview

**Name:** My Account
**ID:** FEAT-18.SPEC-004
**Type:** Screen
**Purpose:** Any adult member views and edits their own name, email, and sign-in, or starts deleting their own account; the organiser additionally reaches household-wide export, removal, and deletion controls from here.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Viewing and editing the signed-in adult's own display_name, email, and sign-in credential
- Starting the signed-in adult's own account deletion
- For Maya only: an organiser-only "Household Data & Deletion" section linking to export, member removal, and household deletion
- Field-level validation and same-screen confirmation for own-account edits

**Non-Goals:**
- Editing another member's profile, dietary rules, budget, or schedule -- owned by FEAT-01 (Household Setup & Member Profiles), never reachable from this screen for anyone but the viewer's own record
- Performing the own-account deletion itself -- owned by FEAT-18.SPEC-009 (Own Account Deletion Processing), which this screen's deletion action triggers after the precondition check
- Editing notification preferences -- owned by FEAT-07 and FEAT-13, reached from household settings rather than this screen
- Kid-profile account management -- neither kid row has a login to manage (scope-boundaries.md SC-02); kid data is managed exclusively through FEAT-01 and this feature's organiser-only removal path (FEAT-18.SPEC-002)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Product's persistent navigation | Any signed-in adult taps "My Account" | None -- screen loads the viewer's own profile |
| FEAT-09.SPEC-004 (Organiser Hand-Over Acceptance) | New organiser taps "Review your account details" on the hand-over Success state | None -- screen loads the new organiser's own profile, now showing the organiser-only section |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Maya (Organiser) | Own profile fields plus the organiser-only Household Data & Deletion section | Edit own name/email/sign-in, delete own account (subject to the hand-over-or-delete-first precondition), reach Contact Support, and reach export/removal/household-deletion | -- |
| Sam (Other Adult Member) | Own profile fields only; the Household Data & Deletion section is not shown | Edit own name/email/sign-in, delete own account, reach Contact Support | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No account exists to reach any screen |
| Jordan (older kid, limited login -- Later) | No | No | This login has no Account & Data access (Access Matrix); the screen is not reachable, and a direct attempt shows "This isn't available for your login." |
| Riley (Operator, support -- from v1) | No | No | Screen is not reachable through Riley's read-only support view (XBR-14); a direct attempt shows the standard support-scope message and stays on the current support-access screen |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | Partial (own field edits preserved) | No | Dialog "Your session has expired. Sign in to continue." -- entered field edits are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "My Account" with a back arrow (returns to the entry source).

**Body:** A single-column form for the viewer's own profile:
- Display Name (text input, required)
- Email (text input, required for adults, used for sign-in)
- "Change sign-in" action, opening a dedicated credential-update step (current credential, new credential, confirmation)
- "Contact Support" action, navigating to FEAT-18.SPEC-005 (Contact Support)
- A "Delete My Account" action at the bottom of the form, visually separated as a destructive action

**Organiser-only section (Maya only, shown below the form, separated by a divider labeled "Household Data & Deletion"):**
- "Export household data" -- links to FEAT-18.SPEC-001
- "Remove a member" -- links to FEAT-18.SPEC-002
- "Delete household" -- links to FEAT-18.SPEC-003

### Responsive Behavior

- **Compact breakpoint:** Single-column form and organiser-only section as described, full width.
- **Medium size class and above:** Form and organiser-only section remain single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| Back arrow | Tap | Navigate to the entry source | Screen closes | Standard transition back |
| Display Name field | Type, then blur | Field-level validation runs inline | Error state on invalid input | "Display name is required" if left empty on blur |
| Email field | Type, then blur | Field-level validation runs inline | Error state on invalid input | "Enter a valid email address" on invalid format |
| Save (form-level, appears once a field changes) | Tap | Validates all changed fields, then saves | Button shows a brief loading state | Success: same-screen confirmation "Your account details have been updated." Failure: inline error messages per field |
| "Change sign-in" | Tap | Opens the credential-update step | Form is replaced by the credential-update step | Credential-update fields appear |
| Credential-update "Save" | Tap | Validates and updates the sign-in credential | Returns to the main form | Success: same-screen confirmation "Your sign-in details have been updated." Failure: inline error on the credential fields |
| "Contact Support" | Tap | Navigate to FEAT-18.SPEC-005 | Screen closes | Standard transition |
| "Delete My Account" | Tap | Checks the own-account deletion precondition via FEAT-18.SPEC-010 | Opens the confirmation step, or shows the blocked state | See States and Business Rules |
| Confirmation "Delete My Account" (in the deletion confirmation step) | Tap | Triggers FEAT-18.SPEC-009 (Own Account Deletion Processing) | Screen shows a progress state | Progress indicator; on completion, the viewer is signed out |
| "Export household data" (Maya only) | Tap | Navigate to FEAT-18.SPEC-001 | Screen closes | Standard transition |
| "Remove a member" (Maya only) | Tap | Navigate to FEAT-18.SPEC-002 | Screen closes | Standard transition |
| "Delete household" (Maya only) | Tap | Navigate to FEAT-18.SPEC-003 | Screen closes | Standard transition |

### Accessibility Notes

- **Focus order:** Back arrow -> Display Name -> Email -> Change sign-in -> Contact Support -> Delete My Account -> (Maya only) Export household data -> Remove a member -> Delete household.
- **Validation announcements:** Field error states are announced to assistive technology and programmatically associated with their field on blur.
- **Save confirmation announcement:** The "Your account details have been updated." confirmation is announced on success; on failure, focus moves to the first field in error.
- **Keyboard alternatives:** Every action on this screen is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | Form area shows loading placeholders in place of the profile fields | Screen first opens, before the initial fetch of the viewer's own profile completes | Fetch succeeds (-> Viewing) or fails (-> Load Error) |
| Load Error | Error banner "We couldn't load your account details. Try again." with a Retry button; no form fields or actions are shown until data loads | The initial fetch of the viewer's own profile fails | Viewer taps Retry (re-fetches) or the back arrow (returns to the entry source) |
| Viewing (default) | Form populated with the viewer's current details; Save button hidden until a field changes | Initial fetch succeeds | Viewer edits a field |
| Editing | Save button visible | Viewer changes any field | Viewer taps Save or navigates away |
| Saving | Save button shows a loading state, fields disabled | Viewer taps Save | Save completes or fails |
| Save error | Inline error banner "Couldn't save your changes. Try again." with entered values preserved | Save fails | Viewer taps Save again or corrects a field |
| Deletion blocked (organiser only) | Blocking dialog: "You're the organiser -- hand over the role to another adult or delete the household before deleting your own account." with actions "Hand over role" (to FEAT-09.SPEC-003) and "Delete household instead" (to FEAT-18.SPEC-003), plus "Cancel" | Maya taps Delete My Account while she is still the organiser with no completed hand-over (FEAT-18.SPEC-010) | Maya taps one of the offered actions or Cancel |
| Deletion confirmation | Loss summary and Delete/Cancel buttons, consistent with the feature's irreversible-action confirmation pattern | Deletion precondition passes (non-organiser adult, or organiser who has handed over) | Viewer confirms or cancels |
| Offline/Degraded | Banner "You're offline -- changes will be saved when you reconnect." at top; the form remains editable and Save queues the edit locally | Connectivity lost while this screen is open | Connectivity restored -- the queued save submits automatically and the standard success feedback appears |

## Validation Rules

Validation governed by FEAT-18.SPEC-010 (Account & Data Validation Rules) for the own-account deletion precondition and the irreversible-action confirmation. Field-level format and required-field rules for display_name, email, and sign-in credential are governed by FEAT-01's field validation rules for Member Profile, referenced here rather than duplicated, since this screen edits the same fields FEAT-01 defines.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|--------------------------------------|
| Back arrow tap | Entry source | -- |
| "Contact Support" tap | FEAT-18.SPEC-005 (Contact Support) | -- |
| "Export household data" (Maya) | FEAT-18.SPEC-001 (Export Household Data) | -- |
| "Remove a member" (Maya) | FEAT-18.SPEC-002 (Remove Member Profile) | -- |
| "Delete household" (Maya) | FEAT-18.SPEC-003 (Delete Household) | -- |
| "Hand over role" (deletion blocked, Maya) | FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | FEAT-09 |
| "Delete household instead" (deletion blocked, Maya) | FEAT-18.SPEC-003 (Delete Household) | -- |
| Successful own-account deletion | The product's sign-in screen (viewer is signed out) | -- |

## Data Model

**Creates:** None.
**Reads:** Member Profile -- display_name, sign_in (email component), member_type, status, for the signed-in viewer's own record; for Maya, the Household's organiser field to determine whether the organiser-only section and the hand-over check apply.
**Updates:** Member Profile -- display_name, sign_in, for the viewer's own record only (own-only, per the Access Matrix).
**Deletes:** None directly -- own-account deletion is performed by FEAT-18.SPEC-009.

## Business Rules

- Every adult member may edit only their own display_name and sign_in; there is no path from this screen to another member's fields (Access Matrix: Account & Data, Own-only for Sam).
- The organiser cannot delete her own account while she remains the organiser and has not handed over the role or deleted the household, per XBR-15 and FEAT-18.SPEC-010.
- Only Maya sees the Household Data & Deletion section, per FEAT-18.SPEC-011 (Account & Data Authorization Rules).
- Field-level edits save inline without navigating away, consistent with product-features.md's Primary Flows for own-account management.

## Edge Cases

- **Sam edits his display name on one device while Maya removes him as a member on another device** -- Per the dependency map's Contention note for Member Profile, removal wins and Sam's concurrent save is refused with "This account no longer exists in this household." rather than a generic save error. This is the concurrent-edit conflict entry for this screen.
- **Maya taps Delete My Account immediately after handing over the organiser role to Sam, before Sam's acceptance is recorded** -- The hand-over is not complete until FEAT-09.SPEC-009 finishes processing Sam's acceptance; Maya still sees the Deletion blocked state until that completes, since she remains the organiser of record until then.
- **Maya's household is deleted (FEAT-18.SPEC-008) while she has this screen open** -- Household deletion supersedes any in-flight edit here (dependency map's Contention note for Household); her screen is redirected to the sign-in screen once the household record is removed.
- **Sam changes his sign-in email to one already used by another member of a different household** -- No conflict exists across households (sign-in identities are global, not household-scoped), so this is not a validation case this spec addresses; any global-uniqueness rule on sign-in credentials is governed by FEAT-01's field validation rules.
- **Viewer navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-010 (Account & Data Validation Rules) | References (inbound) | Own-account deletion precondition and confirmation pattern |
| FEAT-18.SPEC-011 (Account & Data Authorization Rules) | References (inbound) | Governs own-only edit scope and the organiser-only section |
| FEAT-18.SPEC-009 (Own Account Deletion Processing) | Triggers (outbound) | Confirmed deletion starts this automation |
| FEAT-18.SPEC-005 (Contact Support) | Navigation (outbound) | Link to submit a general support request |
| FEAT-18.SPEC-001 (Export Household Data) | Navigation (outbound) | Organiser-only link |
| FEAT-18.SPEC-002 (Remove Member Profile) | Navigation (outbound) | Organiser-only link |
| FEAT-18.SPEC-003 (Delete Household) | Navigation (outbound) | Organiser-only link |
| FEAT-09.SPEC-003 (Organiser Hand-Over Initiation) | Navigation (outbound) | Offered when deletion is blocked for the organiser |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|------------|----------------|-------------------|
| own_account_updated | fields_changed (name / email / sign_in) | An own-account field edit saves successfully | N/A -- no Stage 2 success metric measures account-detail edits; retained as the basic account-management usage signal |
| own_account_deletion_blocked | reason (organiser_no_handover) | Maya attempts deletion while still organiser with no completed hand-over | N/A -- no Stage 2 success metric measures this precondition's frequency; retained to observe how often the organiser-hand-over gate is actually hit |
| own_account_deleted | role_at_deletion (organiser / other_adult) | Own-account deletion completes | N/A -- no Stage 2 success metric measures own-account deletion; retained as the terminal event for this lifecycle action |

## Acceptance Criteria

**FEAT-18.SPEC-004-AC-01:** Given Sam is on My Account, when he changes his display name and taps Save, then he sees the confirmation "Your account details have been updated." with no navigation away from the screen.

**FEAT-18.SPEC-004-AC-02:** Given Sam leaves the Email field empty and blurs it, when validation runs, then he sees "Enter a valid email address" if the format is invalid, or the field's required-error if it is empty.

**FEAT-18.SPEC-004-AC-03:** Given Sam (Other Adult Member) is on My Account, when the screen loads, then no Household Data & Deletion section is shown.

**FEAT-18.SPEC-004-AC-04:** Given Maya (Organiser) is on My Account, when the screen loads, then the Household Data & Deletion section with Export, Remove a member, and Delete household links is shown.

**FEAT-18.SPEC-004-AC-05:** Given Maya taps Delete My Account while she is still the organiser and has not handed over the role, when the check runs, then she sees "You're the organiser -- hand over the role to another adult or delete the household before deleting your own account." with links to hand over or delete the household.

**FEAT-18.SPEC-004-AC-06:** Given Sam (never the organiser) taps Delete My Account, when the precondition check runs, then he proceeds directly to the deletion confirmation step.

**FEAT-18.SPEC-004-AC-07:** Given Maya has completed handing over the organiser role, when she taps Delete My Account, then she proceeds to the deletion confirmation step rather than seeing the blocked state.

**FEAT-18.SPEC-004-AC-08:** Given Sam confirms his own account deletion, when he taps the confirmation Delete My Account, then FEAT-18.SPEC-009 processes the deletion and he is signed out on completion.

**FEAT-18.SPEC-004-AC-09:** Given Sam is removed as a member by Maya while he has an unsaved edit open, when he taps Save, then he sees "This account no longer exists in this household." rather than a generic save error.

**FEAT-18.SPEC-004-AC-10:** Given the viewer loses connectivity while editing a field, when they tap Save, then the banner "You're offline -- changes will be saved when you reconnect." appears and the edit queues locally.

**FEAT-18.SPEC-004-AC-11:** Given the older-kid limited login (Later) attempts to reach My Account, when the screen loads, then the message "This isn't available for your login." is shown and no account fields are exposed.

**FEAT-18.SPEC-004-AC-12:** Given the viewer's session expires while editing a field, when they next interact with the form, then a dialog reads "Your session has expired. Sign in to continue." and the entered edit is preserved and restored after re-authentication.

**FEAT-18.SPEC-004-AC-13:** Given the viewer has unsaved changes and taps the back arrow, when the navigation attempt occurs, then a confirmation dialog reads "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-18.SPEC-004-AC-14:** Given the viewer opens My Account, when the initial fetch of their own profile is in progress, then the form area shows loading placeholders instead of profile fields.

**FEAT-18.SPEC-004-AC-15:** Given the initial fetch of the viewer's own profile fails, when they view this screen, then they see "We couldn't load your account details. Try again." with a Retry button, and tapping Retry either loads the form normally or shows the same error again.

**FEAT-18.SPEC-004-AC-16:** Given Sam is on My Account, when he taps Contact Support, then he is navigated to FEAT-18.SPEC-005 (Contact Support).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Interactions | 11 | 11 |
| States | 9 (loading, load error, viewing, editing, saving, save error, deletion blocked, deletion confirmation, offline) | 9 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
