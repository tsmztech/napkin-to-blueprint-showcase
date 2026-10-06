---
document_type: spec
spec_type: screen
spec_id: FEAT-03.SPEC-002
spec_name: Request Changes
spec_slug: request-changes
parent_feature: FEAT-03
parent_feature_name: Proposal Acceptance
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Request Changes

## Overview

**Name:** Request Changes
**ID:** FEAT-03.SPEC-002
**Type:** Screen
**Purpose:** Owen composes and sends a short change-request note to Nadia instead of accepting the proposal.
**Parent Feature:** FEAT-03 -- Proposal Acceptance

## Scope and Non-Goals

**In Scope:**
- A single text field for Owen's change-request note (1--2,000 characters)
- The Send control and its validation
- Confirmation feedback once the note is sent
- Returning to the proposal review screen

**Non-Goals:**
- General-purpose messaging or chat -- excluded per scope-boundaries.md (SC-15): the request-changes note is a single, contextual note attached to the proposal, not a back-and-forth chat channel; further discussion happens through Nadia revising and re-sending the proposal.
- Displaying past change-request notes -- excluded per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix: within this feature, no screen re-displays a past change-request note; only FEAT-02 reads it when Nadia revises.
- Editing or retracting a sent note -- owned by FEAT-07 (Deliverable Review & Feedback), which governs the general Comment edit-within-grace-window and retraction lifecycle; this screen only creates the note.
- Attaching files or images to the note -- excluded per the Feature Breakdown Brief's Validation & Limits: a change-request note is defined as 1--2,000 characters of text and does not alter the proposal itself; no attachment capability is part of this feature's scope.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-03.SPEC-001 (Proposal Review & Accept) | Owen taps "Request Changes" | Proposal reference; the note field starts empty |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | None -- this is a client-side composition screen | No | Attempting to open this screen's link is treated as an out-of-scope link: plain explanation and a fresh-link option (XBR-09) |
| Owen (Client Primary Contact) | Full screen (Own-only) | Compose and send a change-request note on his own company's proposal | -- |
| Priya (Client Reviewer Contact) | No -- Reviewers have no access to proposal content, per the Access Matrix | No | Cannot reach this screen; her portal home shows only the project's stage label |
| Dana (Support Operator) | No -- this is a write-only action screen with no view-only mode defined for it, consistent with Dana never sending anything on a freelancer's or client's behalf | No | If reached inside a support session, the screen shows no Send control; Dana's read-only view of the proposal is limited to FEAT-03.SPEC-001, which never links here for her |
| Unauthenticated | No | No | Redirected to the client portal's magic-link sign-in (FEAT-05) |
| Expired session | No | No | Plain explanation and a "request a fresh link" option (FEAT-05); any note in progress is not preserved across the expired session |

## Layout and Content

**Header:** Screen title "Request Changes" with a back arrow (returns to FEAT-03.SPEC-001, Proposal Review & Accept).

**Body:** A single-column form with:
- A short instructional line: a brief statement that this note goes to Nadia and does not change the proposal itself
- A multi-line text input for the change-request note (required, 1--2,000 characters), with a live character count shown beneath it
- A "Send" action button below the text input

**Footer:** None -- Send is in the body, directly below the text input.

### Responsive Behavior

- **Compact breakpoint:** Single-column form as described, full width; Send spans the full content width, consistent with the Shared UI Patterns: Decision controls sizing convention from FEAT-03.SPEC-001.
- **Medium size class and above:** Form remains single-column, capped at a consistent platform-wide form width and horizontally centered; the multi-line text input grows to show more visible lines.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-03.SPEC-001 (Proposal Review & Accept) | Screen closes; unsent note is discarded | Confirmation dialog if the note field is non-empty (see Edge Cases) |
| Note text input | Type | Captures text input | Character count updates live | Standard input focus state; character count shown beneath the field |
| Note text input | Exceeds 2,000 characters | Blocks further input beyond the limit | Field shows the limit reached | Character count shows "2,000 / 2,000" and further typing is not accepted |
| Send button | Tap | 1. Validates the note length via FEAT-03.SPEC-005. 2. If valid, triggers FEAT-03.SPEC-004 (Change-Request Recording). | Button shows a loading state during send | Success: confirmation message "Your note has been sent to Nadia" and navigation to FEAT-03.SPEC-001. Failure: inline error message, note preserved. |
| Send button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> instructional line -> note text input -> Send button.
- **Character count announcement:** The live character count is available to assistive technology on request but does not interrupt typing; the "2,000 / 2,000" limit-reached state is announced when input is blocked.
- **Validation announcements:** When the note is empty or invalid at Send, the resulting error message is announced and programmatically associated with the text input.
- **Send feedback:** The "Your note has been sent to Nadia" confirmation is announced on success.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty (default) | Note field empty, Send button enabled but will fail on tap until text is entered | Screen first opens | Owen begins typing |
| Filling | Note field contains user input, live character count updates, Send button enabled | Owen types in the note field | Owen taps Send or navigates away |
| Sending | Send button shows a loading spinner, note field disabled | Owen taps Send with a valid note | Send completes or fails |
| Validation Error | Note field shows an error state below it | Send is tapped with an empty note or one exceeding 2,000 characters | Owen corrects the note and re-taps Send |
| Sent (confirmation) | Confirmation message shown, then navigation back to FEAT-03.SPEC-001 | Send completes successfully | Screen transitions away after the confirmation is shown |
| Error | Inline error banner: "Could not send your note. Check your connection and try again." with a Retry option; note text preserved | The send action fails after passing validation | Owen retries successfully, or navigates away |
| Offline/Degraded | Banner "You're offline -- your note will be sent when you reconnect." at top; note field remains editable; Send queues the note locally | Connectivity is lost while this screen is open | Connectivity restored -- queued note sends automatically and the standard success feedback appears |

## Validation Rules

Validation governed by FEAT-03.SPEC-005 (Acceptance & Access Rules), which defines the note-length rule (1--2,000 characters) shared with FEAT-03.SPEC-004's write-time enforcement. This screen checks the rule on Send.

| Field | Condition | When Checked | Error Message |
|-------|-----------|---------------|-----------------|
| Note text input | Must not be empty | On Send | "Enter a note before sending." |
| Note text input | Must not exceed 2,000 characters | On change and on Send | "Your note can be up to 2,000 characters." |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | FEAT-03.SPEC-001 (Proposal Review & Accept) | -- |
| Successful send | FEAT-03.SPEC-001 (Proposal Review & Accept) | -- |
| Cancel with unsent note (via confirmation) | FEAT-03.SPEC-001 (Proposal Review & Accept) | -- |

## Data Model

**Creates:** None directly -- Send triggers FEAT-03.SPEC-004, which creates the Comment record.
**Reads:** Proposal -- `status` (to confirm the proposal is still eligible for a change request at Send time, per FEAT-03.SPEC-005).
**Updates:** None.
**Deletes:** None.

## Business Rules

- Note length validation (1--2,000 characters) is governed by FEAT-03.SPEC-005 -- this screen enforces it at Send but does not own the rule.
- XBR-26: A request-changes note from the Primary contact is recorded as a comment on the proposal, notifies the freelancer immediately, and never alters the proposal itself.
- XBR-08: Only a Primary contact at the owning client may request changes; this screen is unreachable to Priya (Reviewer).
- The note never alters the proposal itself -- sending a change request does not change the proposal's `status`, `scope_description`, or `price`.

## Edge Cases

- **Owen navigates back with an unsent, non-empty note** -- Confirmation dialog: "Discard this note?" with "Discard" and "Keep Editing" options.
- **Owen taps Send twice rapidly** -- Second tap is ignored while the first send is in progress (button in loading state).
- **The proposal is voided (edited and re-sent by Nadia) while Owen is composing his note** -- Send is rejected with a message directing Owen to the current version, per FEAT-03.SPEC-005's eligibility check running again at Send time; the note text is preserved so Owen can re-submit it against the current version if he still wants to.
- **Network failure during send** -- Error banner: "Could not send your note. Check your connection and try again." with a Retry button. Note text preserved.
- **Owen submits exactly 2,000 characters** -- Accepted; the limit is inclusive.
- **The proposal is accepted by Owen through another session while this screen is open** -- Send is rejected per FEAT-03.SPEC-005 (an accepted proposal is no longer eligible for a change request); the screen shows a message that the proposal has already been accepted and offers navigation back to FEAT-03.SPEC-001, which now shows the Accepted state. No concurrent-edit conflict on the Comment entity itself arises here, since this screen only creates a new Comment and never edits an existing one (dependency map, Comment Contention: "None").

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-03.SPEC-001 (Proposal Review & Accept) | Navigation (inbound) | Owen arrives here from the Request Changes control |
| FEAT-03.SPEC-004 (Change-Request Recording) | Triggers (outbound) | Send button, once valid, triggers the note write |
| FEAT-03.SPEC-005 (Acceptance & Access Rules) | References (outbound) | Note length and eligibility rules |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|---------------|-------------------|
| proposal_changes_requested | proposal reference, note character count | FEAT-03.SPEC-004 confirms the note was recorded | N/A -- no success metric in success-metrics.md is connected to the change-request path; "Time to Proposal Acceptance" measures only the accept path, and this feature has no other connected metric to attribute a change-request signal to |

## Acceptance Criteria

**FEAT-03.SPEC-002-AC-01:** Given Owen is on the Proposal Review & Accept screen, when he taps Request Changes, then he is navigated to this screen with an empty note field.

**FEAT-03.SPEC-002-AC-02:** Given Owen is on this screen, when he types a note and taps Send, then the note is recorded as a Comment on the proposal and Nadia is notified, and Owen sees the confirmation "Your note has been sent to Nadia" before returning to FEAT-03.SPEC-001.

**FEAT-03.SPEC-002-AC-03:** Given Owen is on this screen with an empty note field, when he taps Send, then the note field shows the error "Enter a note before sending." and no note is recorded.

**FEAT-03.SPEC-002-AC-04:** Given Owen has typed 2,000 characters into the note field, when he attempts to type further, then no additional characters are accepted and the character count shows "2,000 / 2,000".

**FEAT-03.SPEC-002-AC-05:** Given Owen has typed a note and not yet sent it, when he taps the back arrow, then a confirmation dialog "Discard this note?" appears with "Discard" and "Keep Editing" options.

**FEAT-03.SPEC-002-AC-06:** Given Owen taps Send and loses connectivity mid-flight, when the send fails, then an inline error banner appears with a Retry option and the note text is preserved.

**FEAT-03.SPEC-002-AC-07:** Given Owen loses connectivity while composing his note, when he attempts to tap Send, then the banner "You're offline -- your note will be sent when you reconnect." appears and the note is submitted automatically once connectivity returns.

**FEAT-03.SPEC-002-AC-08:** Given the proposal Owen is viewing is voided by Nadia editing and re-sending it while he is composing his note, when he taps Send, then the send is rejected and Owen is directed to the current version, with his note text preserved.

**FEAT-03.SPEC-002-AC-09:** Given the proposal Owen is viewing is accepted through another session while he is composing his note, when he taps Send, then the send is rejected with a message that the proposal has already been accepted.

**FEAT-03.SPEC-002-AC-10:** Given Priya (Reviewer) attempts to reach this screen, when she follows any link toward it, then she cannot reach it and her portal home shows only the project's stage label.

**FEAT-03.SPEC-002-AC-11:** Given an unauthenticated visitor attempts to open this screen, when the screen would otherwise load, then they are redirected to the client portal's magic-link sign-in.

**FEAT-03.SPEC-002-AC-12:** Given Owen successfully sends a change-request note, when the note is recorded, then a proposal_changes_requested analytics event is emitted with the proposal reference and note character count.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 7 (empty, filling, sending, validation error, sent, error, offline) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
