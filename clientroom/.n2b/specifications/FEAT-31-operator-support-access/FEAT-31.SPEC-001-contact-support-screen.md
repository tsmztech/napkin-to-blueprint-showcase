---
document_type: spec
spec_type: screen
spec_id: FEAT-31.SPEC-001
spec_name: Contact Support Screen
spec_slug: contact-support-screen
parent_feature: FEAT-31
parent_feature_name: Operator Support Access
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Screen Spec: Contact Support Screen

## Overview

**Name:** Contact Support Screen
**ID:** FEAT-31.SPEC-001
**Type:** Screen
**Purpose:** Nadia describes a problem she is having and sends a support request from inside the product, and receives an on-screen and emailed confirmation that it was received.
**Parent Feature:** FEAT-31 -- Operator Support Access

## Scope and Non-Goals

**In Scope:**
- Capturing Nadia's free-text description of her problem and submitting it as a support request
- Creating the Support Access Session record in its unopened state (`freelancer_account` and `request_text` populated; `operator`, `opened_at`, `closed_at` unset)
- An on-screen confirmation that the request was received, immediately on submit
- Triggering the emailed confirmation (FEAT-31.SPEC-006)

**Non-Goals:**
- Editing or withdrawing a request after it is sent -- no such capability is established by product-features.md's Key Capabilities or Validation & Limits fields for this feature; a request is a one-way message to support, like the email thread it replaces (BRIEF.md, Problem Statement).
- Any view of the operator's diagnosis, reply, or session activity from this screen -- Dana's reply is sent by email outside the product (product-features.md, Primary Flows & Alternates), and the session itself is visible only in Nadia's activity trail (FEAT-13), never here.
- A live chat or real-time conversation with Dana -- product-features.md's Communications field names only the confirmation and session-opened emails; this feature defines no synchronous channel, consistent with the founder being a solo operator (BRIEF.md, Constraints).

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Notifications & Help (product-wide entry) | Nadia selects "Contact support" | None -- form starts empty |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Nadia taps "Contact support" from the Needs attention state | None -- form starts empty |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Submit a support request describing her own problem | -- |
| Owen (Client Primary Contact) | No | No | The "Contact support" entry is not shown anywhere in his portal view; the Access Matrix gives client contacts no Support Access |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- not shown in her portal view |
| Dana (Support Operator) | No | No | Dana works from her own console (FEAT-31.SPEC-002), never from this screen; it is a freelancer-facing entry point only |
| Unauthenticated | No | No | Redirected to sign-in; after signing in, the user lands on her own dashboard, not this screen directly |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any typed but unsent problem description is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Contact Support" with a back arrow (returns to wherever Nadia entered from) and a "Send" action button (right-aligned).

**Body:** A single-column form with one field:
- Problem description (multi-line text input, required) -- placeholder text: "Describe what's going wrong. We'll get back to you by email."

**Footer:** None -- Send is in the header.

### Responsive Behavior

- **Compact breakpoint:** Single-column form, full width; Send remains in the header.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.
- **Problem description field:** Grows from 4 visible lines (compact) to 8 visible lines (medium and above).

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to the screen Nadia entered from | Screen closes | Standard transition back |
| Problem description input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Problem description input | Blur (empty) | Triggers field validation via FEAT-31.SPEC-005 | Error state on field | "Describe your problem before sending." below field |
| Send button | Tap | 1. Validate the field via FEAT-31.SPEC-005. 2. If valid, create the Support Access Session record with `freelancer_account` and `request_text`. 3. Trigger FEAT-31.SPEC-006 (Support Request Confirmation). | Button shows loading state during send | Success: on-screen confirmation "We received your request -- Dana will get back to you by email." and the form clears. Failure: inline error message, entered text preserved. |
| Send button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Problem description input -> Send.
- **Validation announcements:** When the field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Send feedback:** The on-screen confirmation is announced on success; on validation failure, focus moves to the problem description field.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Problem description empty, Send enabled | Screen first opens | Nadia begins typing |
| Filling | Field contains typed text, Send enabled | Nadia types in the field | Nadia taps Send or navigates away |
| Sending | Send button shows loading spinner, field disabled | Nadia taps Send with valid text | Send completes or fails |
| Confirmation | On-screen message "We received your request -- Dana will get back to you by email." replaces the form | Send completes successfully | Nadia navigates away (confirmation is not persisted as a screen state to return to) |
| Error | Error banner at top of form: "Could not send your request. Check your connection and try again." with a Retry option | Send operation fails | Nadia taps Retry or navigates away |
| Offline/Degraded | N/A -- product-features.md's States field marks this screen's Offline-degraded state "N/A -- support sessions are an operator-side, connectivity-required action," and this screen assumes connectivity throughout (Non-Functional Notes, Responsiveness) | -- | -- |

## Validation Rules

**Option A -- Reference Logic/Rule spec:**
Validation governed by FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules). See that spec for the `request_text` field rule. This screen checks it on field blur and on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | Wherever Nadia entered from | -- |
| Successful send | Same screen, showing the Confirmation state | -- |

## Data Model

**Creates:** Support Access Session -- `freelancer_account` (Nadia's own account) and `request_text` (the submitted description) are set; `operator`, `opened_at`, and `closed_at` remain unset until Dana opens a session (FEAT-31.SPEC-003).
**Reads:** None (this is a submission screen -- no existing Support Access Session data is loaded).
**Updates:** None.
**Deletes:** None.

## Business Rules

- Field validation and authorization for the Support Access Session record are governed by FEAT-31.SPEC-005 -- this screen enforces them but does not restate them.
- Submitting a request creates exactly one Support Access Session record; there is no draft state -- the request is sent the moment Send succeeds.
- XBR-29: this request is the record's origin; every downstream open and close of the resulting session is read-only and always announced to Nadia, per the rule this feature owns.

## Edge Cases

- **Nadia navigates away with unsent text in the field** -- Confirmation dialog: "You have an unsent message. Discard?" with "Discard" and "Keep Editing" options.
- **Nadia taps Send twice rapidly** -- Second tap is ignored while the first send is in progress (button in loading state).
- **Network failure during send** -- Error banner: "Could not send your request. Check your connection and try again." with a Retry button. Entered text preserved.
- **Nadia submits a second request while an earlier one is still unopened** -- Both requests are created as separate Support Access Session records and both appear independently in Dana's queue (FEAT-31.SPEC-002); this screen places no limit on how many pending requests Nadia may have.
- **Concurrent-edit conflict** -- Not applicable: the dependency map's Contention note for Support Access Session states "None -- only Dana opens and closes a session, one account at a time, and the record is never edited after it closes; Nadia only reads it." This screen only creates a new record each time; it never loads or resaves an existing one, so no load-then-save race exists to resolve.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules) | References (inbound) | Field validation for `request_text` |
| FEAT-31.SPEC-006 (Support Request Confirmation) | Triggers (outbound) | Successful send triggers the emailed confirmation |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Navigation (outbound, indirect) | The created request appears in Dana's queue; Nadia never navigates there herself |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| support_request_sent | none beyond the event itself | Send completes successfully | N/A -- no metric in success-metrics.md is connected to Operator Support Access or names this behavior; retained per product-features.md's Signals field so support activity stays observable |
| support_request_send_failed | reason (network / validation) | Send operation fails | N/A -- same reason as above |

## Acceptance Criteria

**FEAT-31.SPEC-001-AC-01:** Given Nadia is on the Contact Support screen, when she types a description of her problem and taps Send, then the Support Access Session record is created with her account and description, an on-screen confirmation "We received your request -- Dana will get back to you by email." appears, and the confirmation email (FEAT-31.SPEC-006) is triggered.

**FEAT-31.SPEC-001-AC-02:** Given Nadia is on the Contact Support screen, when she taps Send with the problem description empty, then the field shows the error "Describe your problem before sending." and the send does not proceed.

**FEAT-31.SPEC-001-AC-03:** Given Nadia has typed an unsent description, when she taps the back arrow, then a confirmation dialog appears asking "You have an unsent message. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-31.SPEC-001-AC-04:** Given Nadia loses connectivity while filling the form, when she taps Send, then the error banner "Could not send your request. Check your connection and try again." appears and her entered text is preserved.

**FEAT-31.SPEC-001-AC-05:** Given Owen or Priya is signed in to their portal, when they look for a way to contact support, then no such entry exists anywhere in their view.

**FEAT-31.SPEC-001-AC-06:** Given Nadia already has one unopened support request pending, when she submits a second, different request, then both appear as separate entries in Dana's queue (FEAT-31.SPEC-002).

**FEAT-31.SPEC-001-AC-07:** Given Nadia's session has expired while she was typing a problem description, when the expiry is detected, then the dialog "Your session has expired. Sign in to continue." appears and her typed text is restored after she signs back in.

**FEAT-31.SPEC-001-AC-08:** Given Nadia taps Send twice in rapid succession, when the first send is still in progress, then the second tap has no effect and the button remains in its loading state.

**FEAT-31.SPEC-001-AC-09:** Given Nadia's send request fails on the server, when the failure is returned, then the error banner "Could not send your request. Check your connection and try again." appears with a Retry option and her entered text remains in the field.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 (empty, filling, sending, confirmation, error, offline/degraded N/A) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
