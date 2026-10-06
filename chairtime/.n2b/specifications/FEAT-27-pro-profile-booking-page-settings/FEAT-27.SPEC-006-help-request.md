---
document_type: spec
spec_type: screen
spec_id: FEAT-27.SPEC-006
spec_name: Help Request
spec_slug: help-request
parent_feature: FEAT-27
parent_feature_name: Pro Profile & Booking Page Settings
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Screen Spec: Help Request

## Overview

**Name:** Help Request
**ID:** FEAT-27.SPEC-006
**Type:** Screen
**Purpose:** Talia describes a problem and sends a help request to support from settings.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Capturing a free-text description of Talia's problem and creating a Help Request record on submit
- Triggering the acknowledgment notification

**Non-Goals:**
- Support's read-only lookup of the request, or any ticket status -- owned entirely by FEAT-19 (Platform Support Read-Only Access), per scope-boundaries.md SC-05 and coordination note 8
- The acknowledgment's content and delivery -- owned by FEAT-27.SPEC-013 (Help Request Acknowledgment); this screen only triggers it on submit
- Editing or withdrawing a previously sent help request -- excluded per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix: no edit capability is stated anywhere in Stage 2 for this feature

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Talia taps the "Get help" row | None -- form starts empty |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Submit a help request | -- |
| The Client (Riley) | No | No | Clients have no entry point to this screen and no equivalent capability in this feature |
| Platform Operator (Support) | No -- support does not use this submission form; support's own view of a submitted request is FEAT-19's read-only screen, not this one | No | Not applicable -- this screen has no support-facing mode; support's access is entirely through FEAT-19 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29), per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an unsubmitted description is preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Get Help" with a back arrow (returns to FEAT-27.SPEC-001).

**Body:** A single-column form:
- Description (multi-line text input, required, up to 1,000 characters, with a live remaining-character count), helper text "Tell us what's going on -- we'll get back to you."

**Footer:** "Send" action button, enabled only when Description is non-empty.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width; Send remains in the footer.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-27.SPEC-001 | Screen closes | Standard transition |
| Description input | Type | Captures text input, updates remaining-character count | Field and counter update | Counter shows characters remaining |
| Send button | Tap | 1. Validates Description is non-empty and within the limit. 2. Creates the Help Request record. 3. Triggers FEAT-27.SPEC-013 (acknowledgment) | Button shows loading state during submit | Success: toast "Your request was sent -- we'll be in touch." and navigation returns to FEAT-27.SPEC-001. Failure: inline error with retry, entered text preserved |

### Accessibility Notes

- **Focus order:** Back arrow -> Description input -> Send.
- **Validation announcements:** The "Description is required" error is announced to assistive technology and associated with the field.
- **Submit feedback:** The success toast is announced; on failure, focus moves to the Description field and the error banner is announced.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | Description field empty, Send disabled | Screen first opens | Talia begins typing |
| Filling | Description contains text, Send enabled once non-empty | Talia types | Talia taps Send or navigates away |
| Sending | Send button shows loading state, field disabled | Talia taps Send with valid input | Submit completes or fails |
| Error | Inline error banner with retry; entered text preserved | Submit fails | Talia taps Retry |
| Offline/Degraded | Banner "You're offline -- sending a help request needs a connection." above the form; Description remains editable for drafting but Send is disabled | Connectivity lost while screen is open, or screen opened while offline | Connectivity restored -- Send re-enables |

## Validation Rules

**Option B -- Inline:**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| description | Required, non-empty, max 1,000 characters | On submit | "Please describe your problem before sending" / "Your description must be 1,000 characters or fewer" |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-27.SPEC-001 (Profile & Booking Page Settings) | -- |
| Successful send | FEAT-27.SPEC-001 (Profile & Booking Page Settings) | -- |

## Data Model

**Creates:** Help Request record -- description set from form input, created_at timestamp, and the requesting Pro Account reference (the reference FEAT-19's support lookup opens against).
**Reads:** None.
**Updates:** None.
**Deletes:** None.

## Business Rules

- A submitted Help Request always triggers the acknowledgment notification (FEAT-27.SPEC-013) -- Talia cannot submit without it firing.
- Help Request records are retained indefinitely with no edit, withdraw, or delete path, per the Feature Breakdown Brief's Non-Goals ("Retention or purge policy for help requests" -- kept as part of the Pro's visible account activity, XBR-24).
- Support's subsequent access to this request is read-only and logged in Talia's visible account activity (XBR-24) -- this screen itself has no visibility into that access.

## Edge Cases

- **Talia submits with only whitespace in Description** -- Treated as empty; "Please describe your problem before sending" is shown and the submit does not proceed.
- **Network failure during submit** -- Error banner: "Could not send your request. Check your connection and try again." with a Retry button; her description text is preserved.
- **Talia navigates away with an unsubmitted description** -- Confirmation dialog: "You have an unsent message. Discard?" with "Discard" and "Keep Writing" options.
- **Talia submits two help requests in the same session** -- Each submission creates its own independent Help Request record and triggers its own acknowledgment; no deduplication is applied, since each is a distinct, deliberate request.
- **Talia taps Send twice rapidly** -- Second tap is ignored while the first submit is in progress (button in loading state).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-001 (Profile & Booking Page Settings) | Navigation (inbound and outbound) | Entry from the "Get help" row; back arrow and successful send both return there |
| FEAT-27.SPEC-013 (Help Request Acknowledgment) | Triggers (outbound) | A successful submit fires this notification |
| FEAT-19 (Platform Support Read-Only Access) | Triggers (outbound) | The created request is what support's read-only lookup opens against |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| help_request_sent | description_length | A help request submission completes successfully | N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so help-seeking frequency is observable |

## Acceptance Criteria

**FEAT-27.SPEC-006-AC-01:** Given Talia is on the Help Request screen, when it loads, then the Description field is empty and Send is disabled.

**FEAT-27.SPEC-006-AC-02:** Given Talia types "My deposit link isn't showing the right price" and taps Send, when the submit completes, then a Help Request record is created, she sees "Your request was sent -- we'll be in touch.", and she is returned to FEAT-27.SPEC-001.

**FEAT-27.SPEC-006-AC-03:** Given Talia taps Send with an empty Description, when validation runs, then she sees "Please describe your problem before sending" and the submit does not proceed.

**FEAT-27.SPEC-006-AC-04:** Given Talia's submit fails from a network error, when the failure returns, then she sees "Could not send your request. Check your connection and try again." and her description text remains in the field.

**FEAT-27.SPEC-006-AC-05:** Given Talia has an unsent description and taps the back arrow, when the tap registers, then a confirmation dialog appears asking "You have an unsent message. Discard?" with "Discard" and "Keep Writing" options.

**FEAT-27.SPEC-006-AC-06:** Given Talia successfully submits a help request, when the submission completes, then the acknowledgment notification (FEAT-27.SPEC-013) fires.

**FEAT-27.SPEC-006-AC-07:** Given Talia opens this screen with no connectivity, when the screen loads, then she can still draft her description but Send is disabled with "You're offline -- sending a help request needs a connection."

**FEAT-27.SPEC-006-AC-08:** Given Talia types a description exceeding 1,000 characters and attempts to send, when validation runs, then she sees "Your description must be 1,000 characters or fewer" and the submit does not proceed.

**FEAT-27.SPEC-006-AC-09:** Given Talia submits a second, unrelated help request later in the same session, when it completes, then a second, independent Help Request record is created and its own acknowledgment fires.

**FEAT-27.SPEC-006-AC-10:** Given Talia taps Send twice in rapid succession, when the first submit is still in progress, then the second tap has no effect and the button remains in its loading state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 3 | 3 |
| States | 5 (empty, filling, sending, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
