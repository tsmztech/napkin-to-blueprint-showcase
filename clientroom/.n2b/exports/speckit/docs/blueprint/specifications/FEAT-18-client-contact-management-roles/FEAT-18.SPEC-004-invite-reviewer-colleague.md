---
document_type: spec
spec_type: screen
spec_id: FEAT-18.SPEC-004
spec_name: Invite Reviewer Colleague
spec_slug: invite-reviewer-colleague
parent_feature: FEAT-18
parent_feature_name: Client Contact Management & Roles
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Invite Reviewer Colleague

## Overview

**Name:** Invite Reviewer Colleague
**ID:** FEAT-18.SPEC-004
**Type:** Screen
**Purpose:** Owen, a Client Primary Contact, invites a colleague at his own client company as a Reviewer contact from his portal view.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles

## Scope and Non-Goals

**In Scope:**
- Owen entering a colleague's name and email and sending an invitation
- Restricting the invited role to Reviewer only, with no role picker shown
- Scoping the invite to Owen's own client company only
- Showing Owen's own company's existing contact list for context
- Triggering the invitation email (FEAT-18.SPEC-010) and the counterpart alert to Nadia (FEAT-18.SPEC-011)

**Non-Goals:**
- Inviting or assigning a Primary contact -- excluded per BRIEF.md's Target Users & Roles and the Access Matrix: Owen's invite entitlement is Own-only and Reviewer-only; only Nadia can create or promote a Primary contact (FEAT-18.SPEC-002)
- Editing or removing any existing contact, including Owen's own record -- handled exclusively by Nadia on FEAT-18.SPEC-002 and FEAT-18.SPEC-003; Owen has no edit or remove entitlement anywhere in this feature (Access Matrix: Owen's Client Contact Management is Own-only, scoped to inviting)
- Inviting a colleague at a different client company -- excluded per BRIEF.md's Constraints on strict client isolation and the Access Matrix's Own-only scope; Owen can never see or reach another client company's contacts
- Full contact list management (viewing role changes, removal, status transitions of every contact) -- handled by FEAT-18.SPEC-001, which is Nadia's freelancer-side surface that Owen never reaches

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05.SPEC-003 (Portal Home) | Owen taps "Invite a colleague" | His own client company reference; form starts empty |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | No | No | Nadia never signs in as a client contact and has no route to this screen; her equivalent surface is FEAT-18.SPEC-002 |
| Owen (Client Primary Contact) | Full screen, scoped to his own client company | Invite a Reviewer colleague at his own company | -- |
| Priya (Client Reviewer Contact) | No | No | "Invite a colleague" is not shown on Priya's Portal Home; only a Primary contact can invite (Access Matrix: Priya's Client Contact Management is None) |
| Dana (Support Operator) | No | No | Dana never signs in as a client contact and has no portal access at all (Access Matrix: Client Portal Access -- None; SC-04) |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- entered form data is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Invite a Colleague" with a back arrow (returns to FEAT-05.SPEC-003, Portal Home) and a "Send Invite" action button, right-aligned.

**Body:**
- A brief line of context: "Invite a colleague at {client_company_name} to view and comment on this project. They will not be able to accept proposals, approve milestones, or see invoices."
- Name (text input, required)
- Email (text input, required)
- Role: fixed to "Reviewer," shown as a static label, not a selectable control -- no role picker is shown to Owen
- A collapsible list of his own company's existing contacts (name, role label, status), for context on who already has access

**Footer:** None -- Send Invite is in the header.

### Responsive Behavior

- **Compact size class:** Single-column form as described, full width; the existing-contacts list is collapsed by default with a "Show colleagues" toggle.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; the existing-contacts list is expanded by default.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-05.SPEC-003 (Portal Home) | Screen closes | Standard navigation transition |
| Name input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Name input | Blur (empty) | Triggers validation via FEAT-18.SPEC-005 | Error state on field | "Name is required" below the field |
| Email input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Email input | Blur | Triggers format and per-client-uniqueness validation via FEAT-18.SPEC-005 | Error state on field if invalid | "Please enter a valid email address" or "This email is already used by another contact at this company" |
| "Show colleagues" toggle (compact only) | Tap | Expands or collapses the existing-contacts list | List visibility toggles | Standard expand/collapse animation |
| "Send Invite" button | Tap | 1. Validate fields via FEAT-18.SPEC-005 (role fixed to Reviewer). 2. If valid, create the contact and trigger FEAT-18.SPEC-010 (invitation email) and FEAT-18.SPEC-011 (alert to Nadia). | Button shows loading state during save | Success: toast "Invitation sent" and navigate to FEAT-05.SPEC-003. Failure: inline field errors, or a form-level error banner for a non-field failure. |
| "Send Invite" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "Retry" button (contact-list load-failure banner) | Tap | Re-fetches Owen's own company's existing contacts | The context list returns to Loading, then Loaded | Skeleton rows while loading; on success the list appears and the banner disappears; on repeated failure the banner reappears |

### Accessibility Notes

- **Focus order:** Back arrow -> context line -> Name -> Email -> Role label (non-interactive) -> "Show colleagues" toggle (when present) -> "Send Invite".
- **Validation announcements:** When a field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Save feedback:** The "Invitation sent" toast is announced on success; on validation failure, focus moves to the first field in error.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading (colleague list) | Name and Email fields usable; the existing-contacts list shows skeleton rows; Send Invite disabled until the list has loaded, because the list is what shows Owen any email conflict before he submits | Screen first opens and Owen's own company's contacts are being fetched | Fetch completes (Empty) or fails (List Load Error) |
| List Load Error | Error banner in place of the existing-contacts list: "Couldn't load your colleagues. Try again." with a Retry button; Send Invite disabled | The fetch of Owen's own company's contact list fails | Retry succeeds (Empty), or Owen taps the back arrow |
| Empty (default) | Name and Email empty, Send Invite enabled | Screen first opens and the contact list has loaded | Owen begins typing in either field |
| Filling | Fields contain user input | Owen types in either field | Send Invite is tapped or Owen navigates away |
| Validating | Send Invite shows a loading spinner | Send Invite is tapped | Validation completes (pass or fail) |
| Validation Error | Failed fields highlighted with error messages below them | Validation fails (FEAT-18.SPEC-005) | Owen corrects the field and re-triggers validation |
| Sending | Send Invite shows a loading spinner, fields disabled | Validation passes | Save completes or fails |
| Error | Error banner: "Couldn't send this invitation. Try again." with a Retry button; entered data is preserved | Save operation fails for a reason other than field validation | Retry succeeds |
| Offline/Degraded | N/A -- inviting a colleague is an infrequent, connectivity-required action, consistent with the feature's product-level States definition; a send attempted without connectivity surfaces the Error state above | -- | -- |

## Validation Rules

Validation governed by FEAT-18.SPEC-005 (Contact Field Validation Rules). See that spec for all field-level and cross-field rules, including the rule restricting Owen's assignable role to Reviewer only. This screen applies validation on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-05.SPEC-003 (Portal Home) | FEAT-05 |
| Successful invite | FEAT-05.SPEC-003 (Portal Home) | FEAT-05 |
| Cancel with unsaved changes | FEAT-05.SPEC-003 (Portal Home), after a confirmation dialog | FEAT-05 |

## Data Model

**Creates:** Client Contact -- name and email set from form input; role set to Reviewer (fixed, not user-selectable); invited_by set to Owen's own Client Contact record; status set to Invited.
**Reads:** Client Contact -- name, role, status of every existing contact at Owen's own client company, for the context list.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Field validation (FEAT-18.SPEC-005) is enforced -- Owen cannot send an invite with invalid data.
- Owen's invite entitlement is scoped to his own client company only and to the Reviewer role only (FEAT-18.SPEC-007, Role Authorization Rules; XBR-08); this screen never exposes a Primary option or any other client company's contacts.
- Sending a successful invite triggers both FEAT-18.SPEC-010 (invitation email to the new contact) and FEAT-18.SPEC-011 (alert to Nadia) automatically -- Owen cannot send one without the other.
- The invited contact's role change effective-timing (FEAT-18.SPEC-008) does not apply on creation -- it governs a later role change to an existing contact, which Owen cannot perform.

## Edge Cases

- **Owen navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **Owen taps Send Invite twice rapidly** -- The second tap is ignored while the first send is in progress (button in loading state).
- **Network failure during send** -- Error banner: "Couldn't send this invitation. Try again." with a Retry button. Form data is preserved.
- **Email entered matches another contact already at Owen's company** -- Rejected per FEAT-18.SPEC-005: "This email is already used by another contact at this company." Send does not proceed.
- **Nadia adds a contact with the same email to this client at the same moment from FEAT-18.SPEC-002** -- Per the dependency map's Contention note, Nadia (Full) and Owen (Own-only) can both add contacts to the same client concurrently; the second save to complete is rejected-with-refresh on the per-client email-uniqueness check (FEAT-18.SPEC-005), regardless of whether Nadia or Owen submitted second.
- **Owen's own company's contact list fails to load** -- The List Load Error state shows "Couldn't load your colleagues. Try again." with Retry, and Send Invite stays disabled, so Owen cannot submit without the list that makes an email conflict visible to him.
- **Owen invites a colleague whose email matches an existing Reviewer contact he has no visibility into elsewhere** -- No such case exists: the context list on this screen already shows every contact at his own company, so the uniqueness conflict is always visible to Owen before he submits, not a surprise at save time.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-003 (Portal Home) | Navigation (inbound and outbound) | Entry point and return destination |
| FEAT-18.SPEC-005 (Contact Field Validation Rules) | References (inbound) | Field-level validation and the Owen-invite role restriction |
| FEAT-18.SPEC-007 (Role Authorization Rules) | References (inbound) | Governs Owen's own-company, Reviewer-only invite entitlement |
| FEAT-18.SPEC-010 (New Contact Invitation Email) | Triggers (outbound) | Fired automatically when the invite is sent |
| FEAT-18.SPEC-011 (Primary-Invited-Colleague Alert) | Triggers (outbound) | Fired automatically when the invite is sent, alerting Nadia |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| contact_invited_by_primary | -- | An invite send completes successfully | N/A -- no success-metrics.md metric is connected to Client Contact Management & Roles; retained so Owen's own-company invite activity is observable |
| contact_invite_failed | reason (validation / network) | A send attempt does not complete | N/A -- no success-metrics.md metric is connected to this feature |

## Acceptance Criteria

**FEAT-18.SPEC-004-AC-01:** Given Owen is on this screen, when he enters "Priya Shah" as name and "priya@acme.test" as email and taps "Send Invite", then a Reviewer contact is created with status Invited, both FEAT-18.SPEC-010 and FEAT-18.SPEC-011 fire, and he sees "Invitation sent" before returning to Portal Home.

**FEAT-18.SPEC-004-AC-02:** Given Owen is on this screen, when he looks for a role selector, then none is shown -- the role is displayed as a fixed "Reviewer" label.

**FEAT-18.SPEC-004-AC-03:** Given Owen taps "Send Invite" with the name field empty, then the name field shows "Name is required" and the send does not proceed.

**FEAT-18.SPEC-004-AC-04:** Given Owen enters an email already used by another contact at his own company, when he blurs the email field, then it shows "This email is already used by another contact at this company."

**FEAT-18.SPEC-004-AC-05:** Given Owen is on this screen, when he expands the existing-contacts list, then he sees every contact currently at his own client company with their role and status, and no contact from any other client company.

**FEAT-18.SPEC-004-AC-06:** Given Owen has unsaved changes on this screen, when he taps the back arrow, then a confirmation dialog appears asking "You have unsaved changes. Discard?"

**FEAT-18.SPEC-004-AC-07:** Given Owen loses connectivity while sending an invite, then the error banner "Couldn't send this invitation. Try again." appears with a Retry button, and his entered data is preserved.

**FEAT-18.SPEC-004-AC-08:** Given Owen and Nadia both submit a new contact with the same email for the same client at effectively the same time (Owen here, Nadia on FEAT-18.SPEC-002), when the second save is processed, then it is rejected with the per-client email-uniqueness error.

**FEAT-18.SPEC-004-AC-09:** Given Priya is signed in to the client portal, when she looks at Portal Home, then no "Invite a colleague" action is shown to her, and she has no route to this screen.

**FEAT-18.SPEC-004-AC-10:** Given Nadia or Dana attempts to reach this screen through any route, then none exists for either of them -- this screen exists only inside the client portal.

**FEAT-18.SPEC-004-AC-11:** Given Owen taps "Send Invite" twice in rapid succession, when the first send is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-18.SPEC-004-AC-12:** Given Owen opens this screen, when his company's existing contacts are still being fetched, then the list shows skeleton rows and "Send Invite" is disabled until the list has loaded.

**FEAT-18.SPEC-004-AC-13:** Given the fetch of Owen's own company's contact list fails, when the failure occurs, then Owen sees "Couldn't load your colleagues. Try again." with a Retry button and "Send Invite" stays disabled, and when he taps Retry and the fetch succeeds, then the list appears and "Send Invite" is enabled.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 6 (loading, list load error, validation error, sending error, offline-degraded N/A, concurrent-uniqueness conflict) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |
