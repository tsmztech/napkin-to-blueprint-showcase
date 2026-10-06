---
document_type: spec
spec_type: screen
spec_id: FEAT-18.SPEC-005
spec_name: Contact Support
spec_slug: contact-support
parent_feature: FEAT-18
parent_feature_name: Account & Data Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Screen Spec: Contact Support

## Overview

**Name:** Contact Support
**ID:** FEAT-18.SPEC-005
**Type:** Screen
**Purpose:** Any adult member sends a short description of a problem to the operator and receives an emailed acknowledgement.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- Submitting a short description of a general support problem
- Creating a Support Request of kind "general support contact"
- Showing the same-screen confirmation that the request was received

**Non-Goals:**
- Reporting a safety concern about a specific meal -- a distinct Support Request kind owned by FEAT-02 (Dietary Rules & Allergy Safety Engine), reached from a planned meal, not from this screen
- Any in-product back-and-forth after submission -- excluded per scope-boundaries.md SC-14: this screen sends a one-way description and the household receives a one-time email acknowledgement; further exchange happens outside the product
- Viewing the status of a submitted request or the operator's access record -- owned by FEAT-01 (organiser's view of open requests and access records) and FEAT-22 (operator's status management)
- Riley (Operator) using this screen -- it is a household-facing submission form only; Riley's support access is a separate, read-only capability (FEAT-22)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Product's persistent navigation | Any signed-in adult taps "Contact Support" | None -- screen starts empty |
| FEAT-18.SPEC-004 (My Account) | Adult member taps "Contact Support" in the account form | None -- screen starts empty |
| FEAT-18.SPEC-001 (Export Household Data) | Maya taps the "contact support" link from the export Error state | None -- screen starts empty |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Maya (Organiser) | Full screen | Submit a support description | -- |
| Sam (Other Adult Member) | Full screen | Submit a support description | -- |
| Jordan (young kid profile, no login -- MVP) | No | No | No account exists to reach any screen |
| Jordan (older kid, limited login -- Later) | No | No | This login has no Account & Data access (Access Matrix); a direct attempt shows "This isn't available for your login." |
| Riley (Operator, support -- from v1) | No | No | Screen is not part of Riley's read-only support view (XBR-14); a direct attempt shows the standard support-scope message and stays on the current support-access screen |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | Partial (entered draft preserved) | No | Dialog "Your session has expired. Sign in to continue." -- entered description is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Contact Support" with a back arrow (returns to the entry source).

**Body:** A single-column form:
- An explanatory line: "Tell us what's going wrong and we'll get back to you by email."
- Description (multi-line text input, required, up to 500 characters, with a live character count)
- "Send" action button

**Footer:** None -- Send is the form's sole action.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described, full width; the description field grows with content up to a platform-wide maximum visible height before scrolling internally.
- **Medium size class and above:** Form capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| Back arrow | Tap | Navigate to the entry source | Screen closes | Standard transition back |
| Description field | Type | Captures text input; live character count updates | Character count reflects remaining characters | Count shown as "{used}/500" |
| Description field | Blur (empty) | Triggers required-field validation via FEAT-18.SPEC-010 | Error state on the field | "Tell us what's going wrong before sending" below the field |
| Send button | Tap | 1. Validates the description via FEAT-18.SPEC-010. 2. Creates a general-support Support Request. 3. Triggers FEAT-18.SPEC-015 (Support Request Acknowledgement). | Button shows a loading state during submission | Success: same-screen confirmation "Thanks -- we've got your message and will follow up by email." and the form clears. Failure: inline error with a Retry option, description preserved. |
| Send button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> explanatory line -> Description field -> Send button.
- **Character-count announcement:** The remaining-character count is available to assistive technology on request but is not announced on every keystroke, to avoid interrupting typing.
- **Confirmation announcement:** The success confirmation and any error message are announced when they appear.
- **Keyboard alternatives:** Every action on this screen is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty (default) | Description field empty, Send button enabled | Screen first opens | Member begins typing |
| Filling | Description field contains text | Member types | Member taps Send or navigates away |
| Sending | Send button shows a loading state, field disabled | Member taps Send with a non-empty description | Submission completes or fails |
| Sent | Form clears; same-screen confirmation "Thanks -- we've got your message and will follow up by email." shown | Submission succeeds | Member navigates away or begins a new message |
| Error | Error banner "We couldn't send your message. Try again." with a Retry button; entered description preserved | Submission fails | Member taps Retry |
| Offline/Degraded | Banner "You're offline -- this message will be sent when you reconnect." at top; Send queues the message locally | Connectivity lost while this screen is open | Connectivity restored -- the queued message submits automatically and the Sent confirmation appears |

## Validation Rules

Validation governed by FEAT-18.SPEC-010 (Account & Data Validation Rules). See that spec for the description's required-field and length rules.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|--------------------------------------|
| Back arrow tap | Entry source | -- |
| Successful send | This screen (Sent state) | -- |

## Data Model

**Creates:** Support Request -- kind "general support contact", raised_by (the submitting member), note (the description), status "Raised".
**Reads:** None.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Every adult member (Maya and Sam) can submit a support description, per FEAT-18.SPEC-011 (Account & Data Authorization Rules); neither kid row can, since neither has Account & Data access.
- A submitted Support Request always creates exactly one acknowledgement (FEAT-18.SPEC-015) -- there is no path to submit without an acknowledgement being scheduled.
- Description length is capped and required, per FEAT-18.SPEC-010.

## Edge Cases

- **Member taps Send twice rapidly** -- The second tap is ignored while the first submission is in progress (button in loading state).
- **Member navigates away with an unsent description** -- Confirmation dialog: "You have an unsent message. Discard?" with "Discard" and "Keep Editing" options.
- **Description at exactly 500 characters** -- Submission proceeds; the field simply does not accept further input beyond the limit, so no length-error state is reachable from typing alone.
- **Network failure during send** -- Error banner: "We couldn't send your message. Try again." with a Retry button; the description text is preserved.
- **Member submits a second support description before the first's acknowledgement has arrived** -- Each submission creates its own independent Support Request and its own acknowledgement (FEAT-18.SPEC-015); there is no concurrent-edit conflict here, since this screen only creates new records and never updates a shared entity another member could contend for.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-010 (Account & Data Validation Rules) | References (inbound) | Description field validation |
| FEAT-18.SPEC-011 (Account & Data Authorization Rules) | References (inbound) | Governs who can submit |
| FEAT-18.SPEC-015 (Support Request Acknowledgement) | Triggers (outbound) | Successful submission triggers the acknowledgement |
| FEAT-22 (Operator Read-Only Support Access) | Triggers (outbound, cross-feature) | A submitted request opens Riley's read-only support access for the household |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|------------|----------------|-------------------|
| support_contacted | description_length | A general support description is submitted successfully | N/A -- no Stage 2 success metric measures support contact volume; retained so this trust-facing capability's actual use is observable, per feature-overview.md's Rationale |
| support_contact_failed | reason (validation / network) | Submission fails | N/A -- no Stage 2 success metric measures submission failures; retained to observe whether this path is reliable enough to trust |

## Acceptance Criteria

**FEAT-18.SPEC-005-AC-01:** Given Sam is on the Contact Support screen, when he types a description and taps Send, then a general-support Support Request is created and he sees "Thanks -- we've got your message and will follow up by email."

**FEAT-18.SPEC-005-AC-02:** Given Maya is on the Contact Support screen, when she taps Send with the description field empty, then she sees "Tell us what's going wrong before sending" and no request is created.

**FEAT-18.SPEC-005-AC-03:** Given Sam has typed 500 characters into the description field, when he attempts to type further, then no additional characters are accepted and the field stays at 500.

**FEAT-18.SPEC-005-AC-04:** Given Maya submits a support description successfully, when submission completes, then FEAT-18.SPEC-015 is triggered to send her an acknowledgement.

**FEAT-18.SPEC-005-AC-05:** Given Sam loses connectivity and taps Send, when he is offline, then the banner "You're offline -- this message will be sent when you reconnect." appears and the message queues locally.

**FEAT-18.SPEC-005-AC-06:** Given connectivity returns while a message is queued, when it submits automatically, then Sam sees the Sent confirmation without re-tapping Send.

**FEAT-18.SPEC-005-AC-07:** Given a submission fails due to a network error, when Maya views the screen, then she sees "We couldn't send your message. Try again." with her description preserved.

**FEAT-18.SPEC-005-AC-08:** Given the older-kid limited login (Later) attempts to reach this screen, when the screen loads, then "This isn't available for your login." is shown and no form is exposed.

**FEAT-18.SPEC-005-AC-09:** Given Maya has an unsent description and taps the back arrow, when the navigation attempt occurs, then a confirmation dialog reads "You have an unsent message. Discard?" with "Discard" and "Keep Editing" options.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Interactions | 5 | 5 |
| States | 6 (empty, filling, sending, sent, error, offline) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
