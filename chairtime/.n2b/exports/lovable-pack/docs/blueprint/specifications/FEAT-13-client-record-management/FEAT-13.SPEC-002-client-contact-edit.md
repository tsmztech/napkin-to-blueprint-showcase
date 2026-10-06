---
document_type: spec
spec_type: screen
spec_id: FEAT-13.SPEC-002
spec_name: Client Contact Edit
spec_slug: client-contact-edit
parent_feature: FEAT-13
parent_feature_name: Client Record Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Client Contact Edit

## Overview

**Name:** Client Contact Edit
**ID:** FEAT-13.SPEC-002
**Type:** Screen
**Purpose:** The Pro corrects a client's name, email, or phone number.
**Parent Feature:** FEAT-13 -- Client Record Management

## Scope and Non-Goals

**In Scope:**
- A dedicated form for editing the client's name, phone, and email
- Field-level and cross-field validation feedback, referencing FEAT-13.SPEC-005
- Surfacing the consequence of a phone-number change (access-link invalidation and fresh texting-consent requirement) before the Pro confirms
- Offline/degraded behavior consistent with the Brief's Non-Functional Notes

**Non-Goals:**
- Editing the private note -- handled inline on FEAT-13.SPEC-001 (Client Record Detail), not on this screen
- Re-sending or managing the client's texting consent directly -- consent is re-granted by the Client through FEAT-06 (Client Booking Identity) / FEAT-14 (Messaging Consent Management); this screen only surfaces that a phone-number change requires it, per XBR-15
- Deleting the client's record -- handled by FEAT-13.SPEC-003 (Client Deletion Confirmation); this screen has no delete action
- Editing booking-level data (booking_notes, appointment details) -- booking_notes is client-authored via other features and explicitly read-only from this feature, per the Brief's Shared Entities note

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-13.SPEC-001 (Client Record Detail) | Pro taps "Edit contact" in the overflow menu | Client ID, current name/phone/email values |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Edit and save name, phone, email for their own clients only | -- |
| The Client (Riley) | No | No | Per XBR-29, redirected to the Pro sign-in screen if this screen's route is reached directly; a Client corrects nothing here -- their own email update path is FEAT-06, not this screen |
| Platform Operator (Support) | No -- Support's read-only view of the Client entity, delivered by FEAT-19, never includes an edit path (Support's access to this feature's data is View-only and never includes correcting contact details, per scope-boundaries.md SC-05) | No | If Support's session attempts this screen's route directly, redirected to the Pro sign-in screen per XBR-29 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); no client data is exposed before authentication |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; entered but unsaved field values are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Edit contact" with a back arrow (returns to FEAT-13.SPEC-001 without saving) and a "Save" action button (right-aligned).

**Client Identity Header:** The same read-only-style identity-summary display used on FEAT-13.SPEC-001 and FEAT-13.SPEC-003, but here showing the values as they will be edited: name, phone, and email fields sit directly beneath this header context, per the Brief's Shared UI Patterns.

**Body:** A single-column form, pre-filled with the client's current values:
- Name (text input, required)
- Phone (text input, required)
- Email (text input, required only when the client has declined texts, otherwise optional -- see FEAT-13.SPEC-005 for the exact conditional rule)

Below the phone field, a conditional inline notice appears only while the Pro has changed the phone field's value from its original: "Changing this number will require [client name] to re-confirm texting consent for the new number, and any existing access link will stop working." This notice references XBR-18 and XBR-15 in functional terms and disappears if the Pro reverts the field to its original value.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-13.SPEC-001 (Client Record Detail) without saving | Screen closes | If any field has changed, a confirmation dialog appears first (see Edge Cases) |
| Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Name input | Blur (empty) | Triggers field validation via FEAT-13.SPEC-005 | Error state on field | "Name is required" below the field |
| Phone input | Type | Captures text input; if the value differs from the original, the phone-change notice appears | Field shows entered text; notice appears/disappears as the value changes | Standard input focus state |
| Phone input | Blur | Triggers field validation via FEAT-13.SPEC-005 (format, and identity-uniqueness within this Pro) | Error state on field if invalid | Exact error message per FEAT-13.SPEC-005 |
| Email input | Blur | Triggers field validation via FEAT-13.SPEC-005 (format, and conditional-required when texts are declined) | Error state on field if invalid | Exact error message per FEAT-13.SPEC-005 |
| Save button | Tap | 1. Validate all fields via FEAT-13.SPEC-005. 2. If valid and the phone number changed, show a confirmation step for the phone-number consequence. 3. Persist name/phone/email. | Button shows a loading state during save | Success: toast "Contact updated" and navigate to FEAT-13.SPEC-001. Failure: inline error messages, entered values preserved |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| Phone-change confirmation dialog -- "Confirm" | Tap | Proceeds with the save, including phone-number change | Dialog closes, save proceeds | Standard transition into the Saving state |
| Phone-change confirmation dialog -- "Keep original number" | Tap | Reverts the phone field to its original value; save does not proceed for the phone field | Dialog closes, phone field reverts | Phone field shows the original value again; other field changes remain pending |

### Accessibility Notes

- **Focus order:** Back arrow -> Name -> Phone -> Email -> phone-change notice (when present, announced but not focusable) -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field. The phone-change notice is announced when it appears.
- **Save feedback:** The "Contact updated" toast is announced on success; on validation failure, focus moves to the first field in error. The phone-change confirmation dialog traps focus until dismissed.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | Form pre-filled with the client's current name, phone, email; Save enabled | Screen opens and the client record loads successfully | User begins editing any field |
| Editing | Form fields contain user input, phone-change notice shown if applicable | User types in any field | User taps Save or the back arrow |
| Validating | Save button shows a loading indicator | User taps Save | Validation completes (pass or fail) |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails (per FEAT-13.SPEC-005) | User corrects the field and re-triggers validation |
| Phone-Change Confirmation | Modal dialog described above is shown, form fields inert behind it | Validation passes and the phone field differs from its original value | Pro taps "Confirm" or "Keep original number" |
| Saving | Save button shows a loading indicator, form fields disabled | Validation passes (and phone-change confirmed, if applicable) | Save completes or fails |
| Error | Error banner at top of form with a retry action | Save operation fails | User taps Retry or navigates away |
| Offline/Degraded | Banner "You're offline -- this contact edit requires a live connection to save." at the top; form remains viewable with the client's last-loaded values but Save is disabled | Connectivity lost while the screen is open, or the screen is opened without connectivity | Connectivity restored -- banner clears and Save re-enables |

## Validation Rules

Validation governed by FEAT-13.SPEC-005 (Client Field Validation & Access Rules). See that spec for all field-level rules (name length, phone format and identity-uniqueness within this Pro, email format and conditional-required rule). This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap (no changes) | FEAT-13.SPEC-001 (Client Record Detail) | -- |
| Back arrow tap (unsaved changes, confirmed discard) | FEAT-13.SPEC-001 (Client Record Detail) | -- |
| Successful save | FEAT-13.SPEC-001 (Client Record Detail) | -- |

## Data Model

**Creates:** None.
**Reads:** Client record -- name, phone, email (current values, pre-filled into the form).
**Updates:** Client record -- name, phone, email fields, per FEAT-13.SPEC-005's validation rules.
**Deletes:** None.

## Business Rules

- All field validation (name/phone/email) is governed by FEAT-13.SPEC-005 -- this screen enforces but does not restate those rules.
- Post-deletion concurrent-edit refusal (a save against a deleted client is refused with "This client's record no longer exists. It may have been deleted.") is governed by FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule) -- this screen enforces but does not restate that rule.
- The consent-invalidation effect of a saved phone change is governed by FEAT-14.SPEC-008 (Phone Number Change Consent Invalidation Rule), which reacts to the save made here.
- XBR-18: A changed phone number invalidates the client's existing access links; the Pro is shown this consequence before it takes effect (Phone-Change Confirmation state).
- XBR-15: A changed phone number requires the client to re-grant texting consent for the new number before any further text is sent; this screen surfaces the requirement but the re-grant itself happens through FEAT-06/FEAT-14, outside this screen's scope.
- This screen requires a live connection to save (per the Brief's Non-Functional Notes); it does not queue an offline contact edit for later submission.

## Edge Cases

- **Pro navigates away (back arrow) with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Pro taps Save twice rapidly** -- The second tap is ignored while the first save is in progress (button in loading state).
- **Network failure during save** -- Error banner: "Could not save contact. Check your connection and try again." with a Retry button. Form data preserved.
- **Pro changes the phone number to one already used by another of their own clients** -- Rejected per FEAT-13.SPEC-005's phone-identity-uniqueness rule with the exact error message defined there; the save does not proceed.
- **Client record changed by a concurrent action (e.g., the Client updated their own email via FEAT-06) between this screen's load and save** -- Per the dependency map's Contention note for the Client entity ("field edits are last-write-wins"), the Pro's save of name/phone proceeds and overwrites only the fields the Pro edited; a concurrently changed email field the Pro did not touch on this screen is not overwritten, since the form only submits the fields it displays.
- **Client record was deleted from another device or session while this screen is open** -- Per the dependency map's Contention note ("once deleted any concurrent edit is refused with refresh"), Save is rejected with "This client's record no longer exists. It may have been deleted." and the Pro is returned to the screen they arrived from after acknowledging.
- **Pro reverts the phone field to its original value after the change notice appeared** -- The notice disappears; no phone-change confirmation step occurs on save.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-13.SPEC-001 (Client Record Detail) | Navigation (inbound/outbound) | "Edit contact" tap arrives here; back arrow and successful save both return here |
| FEAT-13.SPEC-005 (Client Field Validation & Access Rules) | References (inbound) | All field-level and cross-field validation rules applied to this form |
| FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule) | References (inbound) | Post-deletion concurrent-edit refusal rule applied to any contact save attempted after the client record is gone |
| FEAT-14.SPEC-008 (Phone Number Change Consent Invalidation Rule) | References (outbound) | Saving a changed phone number is the event that rule reacts to; this screen enforces the phone-change confirmation step and the rule leaves the existing consent record's phone_number stale, invalidating it (XBR-15) |
| FEAT-06 (Client Booking Identity) | References (outbound) | A changed phone number invalidates access links owned by this feature (XBR-18) |
| FEAT-14 (Messaging Consent Management) | References (outbound) | A changed phone number requires fresh texting consent, re-granted through this feature (XBR-15) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| client_contact_updated | fields_changed (name / phone / email, one or more), phone_changed (boolean) | Contact save completes successfully | N/A -- no success-metrics.md metric names this feature directly; contact correction is a data-integrity action rather than one of the feature's own analytics-linked moments (per the Brief's Analytics linkage note, which names only client_record_viewed, client_note_added, and client_record_deleted) |

## Acceptance Criteria

**FEAT-13.SPEC-002-AC-01:** Given Talia is on the Client Record Detail screen and taps "Edit contact," when the Client Contact Edit screen opens, then it is pre-filled with the client's current name, phone, and email.

**FEAT-13.SPEC-002-AC-02:** Given Talia clears the name field and moves focus away, when the blur validation runs, then the field shows the error "Name is required" per FEAT-13.SPEC-005.

**FEAT-13.SPEC-002-AC-03:** Given Talia changes the phone field's value, when the value differs from the original, then the notice about re-confirming texting consent and access-link invalidation appears beneath the field.

**FEAT-13.SPEC-002-AC-04:** Given Talia has changed the phone number and taps Save with all fields valid, when validation passes, then the Phone-Change Confirmation dialog appears before the save is committed.

**FEAT-13.SPEC-002-AC-05:** Given Talia sees the Phone-Change Confirmation dialog, when she taps "Confirm," then the save proceeds with the new phone number and she sees the toast "Contact updated," returning to FEAT-13.SPEC-001.

**FEAT-13.SPEC-002-AC-06:** Given Talia sees the Phone-Change Confirmation dialog, when she taps "Keep original number," then the phone field reverts to its original value and no phone change is saved.

**FEAT-13.SPEC-002-AC-07:** Given Talia enters a phone number already used by another of her own clients, when she taps Save, then the save is rejected with the phone-identity-uniqueness error defined in FEAT-13.SPEC-005.

**FEAT-13.SPEC-002-AC-08:** Given Talia has unsaved changes on this screen, when she taps the back arrow, then a confirmation dialog appears: "You have unsaved changes. Discard?"

**FEAT-13.SPEC-002-AC-09:** Given Talia's save fails due to a network error, when the failure occurs, then the error banner "Could not save contact. Check your connection and try again." appears with her entered values preserved.

**FEAT-13.SPEC-002-AC-10:** Given the client's record was deleted from another session while Talia has this screen open, when she taps Save, then the save is rejected with "This client's record no longer exists. It may have been deleted." and she is returned to the screen she arrived from.

**FEAT-13.SPEC-002-AC-11:** Given Talia loses connectivity while on this screen, when the connection drops, then the banner "You're offline -- this contact edit requires a live connection to save." appears and Save becomes disabled.

**FEAT-13.SPEC-002-AC-12:** Given Riley (the Client) attempts to open this screen's route directly, when the access check runs, then Riley is redirected to the Pro sign-in screen.

**FEAT-13.SPEC-002-AC-13:** Given Platform Operator (Support) attempts to open this screen's route directly, when the access check runs, then Support is redirected to the Pro sign-in screen -- Support's own access never includes a contact-edit path, per scope-boundaries.md SC-05.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 10 | 10 |
| States | 8 (loaded, editing, validating, validation error, phone-change confirmation, saving, error, offline) | 8 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |
