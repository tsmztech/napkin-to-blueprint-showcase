---
document_type: spec
spec_type: screen
spec_id: FEAT-18.SPEC-002
spec_name: Add or Edit Client Contact
spec_slug: add-or-edit-client-contact
parent_feature: FEAT-18
parent_feature_name: Client Contact Management & Roles
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Add or Edit Client Contact

## Overview

**Name:** Add or Edit Client Contact
**ID:** FEAT-18.SPEC-002
**Type:** Screen
**Purpose:** Nadia adds a new contact to a client company or edits an existing contact's name, email, or role.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles

## Scope and Non-Goals

**In Scope:**
- Creating a new Client Contact record (name, email, role) for the scoped client
- Editing an existing Client Contact's name, email, or role
- Inline field validation and save-time validation via FEAT-18.SPEC-005
- Triggering the first-sign-in invitation email (FEAT-18.SPEC-010) on a successful create
- Applying the role-change effective-timing behavior (FEAT-18.SPEC-008) when an existing contact's role changes

**Non-Goals:**
- Removing a contact -- handled by FEAT-18.SPEC-003 (Remove Client Contact); this screen never deletes a Client Contact record
- Owen's own invite screen -- handled by FEAT-18.SPEC-004 (Invite Reviewer Colleague), which shares this screen's field set but fixes the role to Reviewer and restricts the scope to Owen's own company; this spec covers Nadia's freelancer-side form only
- Choosing which client the contact belongs to -- the client is fixed by the entry context (FEAT-18.SPEC-001); this screen never re-scopes a contact to a different client
- Restoring a previously removed contact -- excluded per the feature's Entity-Lifecycle Coverage Matrix: because removal is also the erasure mechanism (XBR-27), a returning contact is entered here as a brand-new Client Contact record, never as a restore of the old one

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-18.SPEC-001 (Client Contact List) | Nadia taps "Add Contact" | Client reference; form starts empty (create mode) |
| FEAT-18.SPEC-001 (Client Contact List) | Nadia taps an existing contact's row or "Edit" | Client reference and the selected contact's current name, email, and role (edit mode) |
| FEAT-18.SPEC-001 (Client Contact List) | Nadia taps a "This email is bouncing" note | Same as edit mode, with the email field pre-focused |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Create a contact with any role, or edit an existing contact's name, email, or role | -- |
| Owen (Client Primary Contact) | No | No | Owen never reaches this screen; his own invite screen is FEAT-18.SPEC-004 (Access Matrix: Client Contact Management is Own-only, scoped to inviting -- not this general add/edit surface) |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation (Access Matrix: None) |
| Dana (Support Operator) | No | No | Dana's mirrored view of FEAT-18.SPEC-001 never exposes this screen, since Support Access Sessions are read-only and never open a create/edit form (FEAT-31.SPEC-005) |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- entered form data is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "New Contact" (create mode) or "Edit Contact" (edit mode), with a back arrow (returns to FEAT-18.SPEC-001) and a "Save" action button, right-aligned.

**Body:** A single-column form with the following fields in order:
- Name (text input, required)
- Email (text input, required)
- Role (selection input: Primary or Reviewer, required)

In edit mode, all three fields are pre-filled with the contact's current values. The Role field additionally shows a note when changed: "This change applies to this contact's future actions only; it never alters a past acceptance or approval" (FEAT-18.SPEC-008).

**Footer:** None -- Save is in the header.

### Responsive Behavior

- **Compact size class:** Single-column form as described, full width; Save remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-18.SPEC-001 (Client Contact List) | Screen closes | Standard navigation transition, or a confirmation dialog first if there are unsaved changes |
| Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Name input | Blur (empty) | Triggers validation via FEAT-18.SPEC-005 | Error state on field | "Name is required" below the field |
| Email input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Email input | Blur | Triggers format and per-client-uniqueness validation via FEAT-18.SPEC-005 | Error state on field if invalid | "Please enter a valid email address" or "This email is already used by another contact at this client" |
| Role selector | Select | Sets the contact's role to Primary or Reviewer | Selector shows chosen role; if editing an existing contact and the value differs from the loaded value, the future-only note appears | Selected role displayed; note text shown when applicable |
| Save button | Tap | 1. Validate all fields via FEAT-18.SPEC-005. 2. If valid and creating, save the new contact and trigger FEAT-18.SPEC-010 (invitation email). 3. If valid and editing, save changes; if the role changed, apply FEAT-18.SPEC-008's future-only effective timing. | Button shows loading state during save | Success: toast "Contact added" (create) or "Contact updated" (edit), then navigate to FEAT-18.SPEC-001. Failure: inline field errors, or a form-level error banner for a non-field failure. |
| Save button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "Retry" button (load-failure banner, edit mode) | Tap | Re-fetches the contact's current name, email, and role | Screen returns to Loading, then Loaded on success | Skeleton fields while loading; on success the form is pre-filled and the banner disappears; on repeated failure the same banner reappears |

### Accessibility Notes

- **Focus order:** Back arrow -> Name -> Email -> Role selector -> Save.
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The success toast is announced on save; on validation failure, focus moves to the first field in error.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading (edit mode) | Skeleton placeholders in place of the three fields; Save disabled | Screen opens in edit mode and the contact's current data is being fetched | Fetch completes (Loaded) or fails (Load Error) |
| Load Error (edit mode) | Error banner: "Couldn't load this contact. Try again." with a Retry button; fields not shown and Save disabled | The edit-mode fetch of the contact fails | Retry succeeds (Loaded) or Nadia taps the back arrow |
| Empty (create mode default) | All fields empty, Save enabled | Screen opens in create mode | Nadia begins typing in any field |
| Loaded (edit mode default) | Fields pre-filled with the contact's current data, Save enabled | Screen opens in edit mode | Nadia edits any field |
| Filling | Fields contain user input | Nadia types in any field | Save is tapped or Nadia navigates away |
| Validating | Save button shows a loading spinner | Save is tapped | Validation completes (pass or fail) |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails (FEAT-18.SPEC-005) | Nadia corrects the field and re-triggers validation |
| Saving | Save button shows a loading spinner, fields disabled | Validation passes | Save completes or fails |
| Error | Error banner: "Couldn't save this contact. Try again." with a Retry button; entered field values are preserved for retry | Save operation fails for a reason other than field validation | Retry succeeds |
| Offline/Degraded | N/A -- contact management is an infrequent, connectivity-required action (feature-overview.md, States); a save attempted without connectivity surfaces the Error state above rather than a distinct offline queue | -- | -- |

## Validation Rules

Validation governed by FEAT-18.SPEC-005 (Contact Field Validation Rules). See that spec for all field-level and cross-field rules (required fields, email format, per-client email uniqueness). This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-18.SPEC-001 (Client Contact List) | -- |
| Successful save | FEAT-18.SPEC-001 (Client Contact List) | -- |
| Cancel with unsaved changes | FEAT-18.SPEC-001 (Client Contact List), after a confirmation dialog | -- |

## Data Model

**Creates:** Client Contact -- name, email, role (Primary or Reviewer) set from form input; invited_by set to Nadia; status set to Invited.
**Reads:** Client Contact -- name, email, role, when opened in edit mode.
**Updates:** Client Contact -- name, email, role. A role change is applied per FEAT-18.SPEC-008 (future actions only; never alters a past acceptance or approval recorded under the prior role).
**Deletes:** None.

## Business Rules

- Field validation (FEAT-18.SPEC-005) is enforced -- Nadia cannot save with invalid data.
- Role assignment is governed by FEAT-18.SPEC-007 (Role Authorization Rules); Nadia may assign either Primary or Reviewer to any contact she manages.
- A role change on save is subject to FEAT-18.SPEC-008 (Role Change Effective-Timing Rule): the new role governs only future actions.
- Saving a new contact triggers FEAT-18.SPEC-010 (New Contact Invitation Email) automatically -- Nadia cannot skip it.
- XBR-07: creating a client's first Primary contact here is what allows a proposal to be sent for that client (FEAT-18.SPEC-006).

## Edge Cases

- **Nadia navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Save twice rapidly** -- The second tap is ignored while the first save is in progress (button in loading state).
- **The contact's data fails to load in edit mode** -- The Load Error state shows "Couldn't load this contact. Try again." with Retry; no editable form is shown, so Nadia can never save over data she has not seen.
- **Network failure during save** -- Error banner: "Couldn't save this contact. Try again." with a Retry button. Form data is preserved.
- **Email entered matches another contact at the same client** -- Rejected per FEAT-18.SPEC-005: "This email is already used by another contact at this client." Save does not proceed.
- **Contact edited by Nadia in another session between load and save** -- Save is rejected with a dialog: "This contact was updated in another session. Review the latest version before saving." with "View Latest" (reloads the record; local edits discarded after confirmation) and "Keep Editing" options. Resolution: reject-with-refresh, per the dependency map's Contention note for the Client Contact entity.
- **Nadia changes an existing Primary contact's role to Reviewer, and the contact is the client's last Primary** -- Save is blocked with the message: "This is the client's only Primary contact. Add or promote another Primary before changing this one." (FEAT-18.SPEC-006), consistent with the last-Primary protection applied to removal.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-001 (Client Contact List) | Navigation (inbound and outbound) | Entry point and return destination |
| FEAT-18.SPEC-005 (Contact Field Validation Rules) | References (inbound) | Field-level and cross-field validation applied on blur and submit |
| FEAT-18.SPEC-006 (Primary Contact Requirement Rule) | References (inbound) | Blocks a role change that would leave the client with no Primary |
| FEAT-18.SPEC-007 (Role Authorization Rules) | References (inbound) | Governs Nadia's unrestricted role-assignment entitlement |
| FEAT-18.SPEC-008 (Role Change Effective-Timing Rule) | Triggers (outbound) | Applied whenever an existing contact's role changes on save |
| FEAT-18.SPEC-010 (New Contact Invitation Email) | Triggers (outbound) | Fired automatically when a new contact is saved |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| contact_added | role_assigned (primary / reviewer) | A new contact save completes | N/A -- no success-metrics.md metric is connected to Client Contact Management & Roles; retained so contact-capture activity is observable |
| contact_edited | fields_changed (name / email / role, may be multiple) | An existing contact's edit save completes | N/A -- no success-metrics.md metric is connected to this feature |
| contact_role_changed | previous_role, new_role | A save changes an existing contact's role | N/A -- no success-metrics.md metric is connected to this feature |
| contact_save_failed | reason (validation / network / stale_record) | A save attempt does not complete | N/A -- no success-metrics.md metric is connected to this feature |

## Acceptance Criteria

**FEAT-18.SPEC-002-AC-01:** Given Nadia is on this screen in create mode, when she enters "Owen Carter" as name, "owen@acme.test" as email, selects Primary, and taps Save, then the contact is created with status Invited, the invitation email (FEAT-18.SPEC-010) fires, and she sees "Contact added" before returning to FEAT-18.SPEC-001.

**FEAT-18.SPEC-002-AC-02:** Given Nadia taps Save with the name field empty, then the name field shows "Name is required" and the save does not proceed.

**FEAT-18.SPEC-002-AC-03:** Given Nadia enters an email already used by another contact at the same client, when she blurs the email field, then it shows "This email is already used by another contact at this client."

**FEAT-18.SPEC-002-AC-04:** Given Nadia opens an existing Reviewer contact in edit mode and changes the role to Primary, when she taps Save, then a note confirms the change applies to future actions only, and the save completes.

**FEAT-18.SPEC-002-AC-05:** Given Nadia opens the client's only Primary contact in edit mode and changes the role to Reviewer, when she taps Save, then the save is blocked with "This is the client's only Primary contact. Add or promote another Primary before changing this one."

**FEAT-18.SPEC-002-AC-06:** Given Nadia has unsaved changes on this screen, when she taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?"

**FEAT-18.SPEC-002-AC-07:** Given Nadia loses connectivity while attempting to save, then the error banner "Couldn't save this contact. Try again." appears with a Retry button, and her entered data is preserved.

**FEAT-18.SPEC-002-AC-08:** Given a contact Nadia is editing was updated in another of her sessions before she saves, when she taps Save, then the save is rejected with "This contact was updated in another session. Review the latest version before saving." and "View Latest" / "Keep Editing" options.

**FEAT-18.SPEC-002-AC-09:** Given Nadia taps Save twice in rapid succession, when the first save is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-18.SPEC-002-AC-10:** Given Nadia arrives from a delivery warning on a bouncing contact email, when the screen opens, then it is in edit mode for that contact with the email field focused.

**FEAT-18.SPEC-002-AC-11:** Given Owen, Priya, or Dana attempts to reach this screen through any route, then none exists for any of them -- the screen is not part of the client portal or the support console.

**FEAT-18.SPEC-002-AC-12:** Given Nadia's session expires while she has unsaved form data, when the expiry dialog appears and she re-authenticates, then her entered field values are restored.

**FEAT-18.SPEC-002-AC-13:** Given Nadia enters a name and email but leaves Role at its default, when she taps Save, then the Role selection is required and validation blocks the save if no role was ever selected.

**FEAT-18.SPEC-002-AC-14:** Given Nadia opens an existing contact in edit mode, when the contact is still being fetched, then she sees skeleton placeholders in place of the fields and Save is disabled until the data has loaded.

**FEAT-18.SPEC-002-AC-15:** Given the edit-mode fetch of the contact fails, when the failure occurs, then Nadia sees "Couldn't load this contact. Try again." with a Retry button and no editable form, and when she taps Retry and the fetch succeeds, then the form is pre-filled with the contact's current name, email, and role.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 6 (loading, load error, validation error, saving error, offline-degraded N/A, stale-record conflict) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |
