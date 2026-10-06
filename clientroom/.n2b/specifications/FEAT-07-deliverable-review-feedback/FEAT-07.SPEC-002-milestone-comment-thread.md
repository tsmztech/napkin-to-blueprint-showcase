---
document_type: spec
spec_type: screen
spec_id: FEAT-07.SPEC-002
spec_name: Milestone Comment Thread
spec_slug: milestone-comment-thread
parent_feature: FEAT-07
parent_feature_name: Deliverable Review & Feedback
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 22
---

# Screen Spec: Milestone Comment Thread

## Overview

**Name:** Milestone Comment Thread
**ID:** FEAT-07.SPEC-002
**Type:** Screen
**Purpose:** A contact or Nadia views and posts general feedback pinned to a milestone as a whole -- for a round that is not about one specific file -- in the same chronological thread pattern as the deliverable-level thread, and retracts their own comment.
**Parent Feature:** FEAT-07 -- Deliverable Review & Feedback

## Scope and Non-Goals

**In Scope:**
- Rendering the full chronological comment thread pinned to one Milestone, scoped by the viewer's role and company
- The composer for posting a new comment pinned to that same Milestone
- Editing the author's own comment's text within the short grace window defined by FEAT-07.SPEC-006
- Retracting the author's own comment (soft removal, always available to its author) per FEAT-07.SPEC-006
- The "no feedback yet" empty state, the loading indicator, the submission-failure retry state, and the offline-composing state
- The "Sync Failed" state for a queued offline comment that could not be sent on reconnect (FEAT-07.SPEC-008), with Edit, Retry, and Discard-with-confirmation controls for that unsent comment
- Showing Nadia's, Owen's, and Priya's own-company thread and Dana's read-only view inside a logged support session

**Non-Goals:**
- Deliverable-version-level (per-file) comments -- handled by the sibling screen FEAT-07.SPEC-001 (Deliverable Comment Thread); this screen only ever pins to a Milestone as a whole, never to a specific file or round.
- Nested reply threading UI -- for the same reason as FEAT-07.SPEC-001: the Shared UI Pattern is a single flat chronological thread, and `reply_to` is never set from this screen.
- Comment content and length validation logic -- owned entirely by FEAT-07.SPEC-005 (Comment Content & Submission Validation).
- Who may view, post, or act on this thread -- owned entirely by FEAT-07.SPEC-007 (Comment Visibility & Authorization Rule).
- Approving the milestone -- owned entirely by FEAT-08 (Milestone Approval); this screen's Approve-adjacent role is only to be shown as context, per the Feature Breakdown Brief's Side-Effect Inventory ("Priya attempts an Approve action she does not have -- this feature's screens simply never render it for a comment-only role").

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05 (Client Portal Access, Portal Home, FEAT-05.SPEC-003) | Owen or Priya opens the milestone view to leave feedback that is not about one specific file | The Milestone reference |
| FEAT-01 (Client & Project Management, Project Detail, FEAT-01.SPEC-005) | Nadia opens a milestone from her project workspace | The Milestone reference |
| FEAT-07.SPEC-003 (Client Comment Alert to Freelancer) | Nadia opens the "new feedback is waiting" email for a milestone-level comment and follows its link | The specific Milestone the comment was posted against |
| FEAT-08 (Milestone Approval, Milestone Review & Approval Screen, FEAT-08.SPEC-001) | Owen reviews the milestone before deciding to approve, or Nadia reviews it after a "not yet satisfied" outcome, and opens the full milestone-level thread from the approval screen's embedded context summary | The Milestone reference the approval screen was showing context for |
| FEAT-31 (Operator Support Access, Operator Support Session Console, FEAT-31.SPEC-002) | Dana opens this screen while working through a freelancer's own screens inside a logged support session | The Milestone reference; session is announced and read-only |
| FEAT-07.SPEC-004 (Freelancer Reply Alert to Client) | Owen or Priya taps the email's "Open thread" CTA | The specific Milestone the reply was posted against |
| FEAT-04.SPEC-002 (Milestone Timeline (Client View)) | Owen or Priya taps a milestone row that has no deliverable ready for review yet | The Milestone reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full thread -- every comment on this Milestone from every authorized contact and herself; her own unsent (Sync Failed) comments on this device | Post a comment; edit her own comment within the grace window; retract her own comment; edit, retry, or discard her own unsent comment | -- |
| Owen (Client Primary Contact) | Full thread, Own-only -- only for his own company's milestones (FEAT-07.SPEC-007); his own unsent (Sync Failed) comments on this device | Post a comment; edit his own comment within the grace window; retract his own comment; edit, retry, or discard his own unsent comment | Attempting to reach a thread for a milestone outside his own company is treated as an out-of-scope link: plain explanation and a fresh-link option (XBR-09) |
| Priya (Client Reviewer Contact) | Full thread, Own-only -- only for her own company's milestones (FEAT-07.SPEC-007); her own unsent (Sync Failed) comments on this device | Post a comment; edit her own comment within the grace window; retract her own comment; edit, retry, or discard her own unsent comment | Same out-of-scope handling as Owen for a milestone outside her own company |
| Dana (Support Operator) | Full thread, read-only, inside a logged support session (FEAT-31) | No actions -- composer, edit, and retract controls are not rendered | If Dana attempts an action outside the read-only bounds, no control exists to attempt it with |
| Unauthenticated | No | No | Redirected to the client portal's magic-link sign-in (FEAT-05) |
| Expired session | No | No | Plain explanation and a "request a fresh link" option; no thread content shown |

## Layout and Content

**Header:** Milestone name and its project name, with the milestone's current status label ("Deliverable Uploaded", "Approved", etc., read from FEAT-08).

**Body:** Identical structural pattern to FEAT-07.SPEC-001's thread and composer, pinned here to the Milestone rather than a Deliverable Version:
- Thread list -- one entry per comment, each showing author name, author role indicator, posted timestamp, and comment text; a retracted comment shows the "This comment was retracted" placeholder in its original position
- "Edit" and "Retract" controls on the viewer's own comments only, per the same rules as FEAT-07.SPEC-001; an "(edited)" marker beside any edited comment
- Empty-state message "No feedback yet" when no comments exist
- Unsent comment entries (Sync Failed state) -- shown after the last posted comment and above the composer, visually distinct from posted comments, each with a "Not sent" label, the author's text, a reason message, and the controls that apply to that failure reason (Edit, Retry, Discard); visible only on the device that queued them and only to their author
- Composer -- positioned below the thread list, at the bottom of the body area: a multi-line text input, a character count, and a "Post" button

**Footer:** None -- the composer sits in the body, immediately after the thread.

### Responsive Behavior

- **Compact breakpoint:** Single-column thread and composer, full width; composer's Post button stacks below the text input.
- **Medium size class and above:** Content capped at a consistent platform-wide reading width and horizontally centered; Post sits beside the text input rather than beneath it.
- **Thread entries:** Uniform scaling, no structural change across breakpoints.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Thread list | Screen loads | Reads every Comment pinned to this Milestone, scoped by FEAT-07.SPEC-007's visibility rule, ordered by `posted_at` | Thread renders with all visible comments | Loading indicator while the thread loads, then full thread appears |
| Composer text input | Type | Captures the comment's text | Character count updates | Standard input focus state; count turns visually distinct near the 2,000-character limit |
| Post button | Tap | 1. Validates text via FEAT-07.SPEC-005. 2. If valid and online, creates the Comment pinned to this Milestone; the successful write fires FEAT-07.SPEC-003 (Client Comment Alert to Freelancer) when the author is Owen or Priya, or FEAT-07.SPEC-004 (Freelancer Reply Alert to Client) when the author is Nadia. 3. If valid and offline, hands off to FEAT-07.SPEC-008. 4. If invalid, shows the field-level error. | Button shows a brief loading state during an online submission | Success (online): composer clears and the new comment appears immediately. Success (offline): see Offline/Degraded state. Failure: inline error below the composer or the Error state. |
| Edit control (own comment, within grace window) | Tap | Opens the comment's text in an inline editable field, per FEAT-07.SPEC-006 | Comment's display text is replaced by an editable field with Save/Cancel | Standard inline-edit affordance |
| Save (inline edit) | Tap | Validates the edited text via FEAT-07.SPEC-005, then writes the update per FEAT-07.SPEC-006 | Comment returns to display mode, now marked "(edited)" | Success: comment updates in place. Failure: inline error, edit remains open |
| Cancel (inline edit) | Tap | Discards the in-progress edit | Comment returns to display mode, unchanged | Immediate, no additional feedback |
| Retract control (own comment) | Tap | Opens a confirmation, then writes the retraction per FEAT-07.SPEC-006 | Comment's text is replaced by the retracted placeholder | Confirmation dialog before the action; placeholder appears immediately once confirmed |
| Retry (on submission failure) | Tap | Re-attempts the same Post action with the preserved composer text | Button returns to loading state | Standard retry feedback |
| Edit control (unsent Sync Failed comment, validation failure only) | Tap | Opens the unsent entry's text in an inline editable field; this edits the local queue entry, not a posted comment, so FEAT-07.SPEC-006's grace window does not apply (FEAT-07.SPEC-008 edge case) | Entry's text becomes an editable field with Save/Cancel | Standard inline-edit affordance |
| Save / Cancel (unsent entry edit) | Tap | Save: validates the edited text via FEAT-07.SPEC-005, replaces the queue entry's text, and returns the entry to Queued for FEAT-07.SPEC-008 to submit (immediately if online). Cancel: discards the in-progress edit | Save: entry leaves Sync Failed and shows the queued indicator (or posts and joins the thread once synced). Cancel: entry returns to Sync Failed display, unchanged | Save success: the "Not sent" label changes to "Sending..." then the comment appears in the thread on success. Save failure (invalid text): inline error, edit remains open. Cancel: immediate |
| Retry control (unsent entry, repeated server-error failure only) | Tap | Asks FEAT-07.SPEC-008 to attempt the entry's submission now | Entry shows "Sending..." while the attempt runs | Success: entry becomes a posted comment at the end of the thread. Failure: entry returns to Sync Failed with the same message |
| Discard control (unsent Sync Failed comment) | Tap | Opens a confirmation dialog titled "Discard this comment?" with the text "This comment hasn't been sent. If you discard it, it will be permanently deleted from this device and can't be recovered." and two buttons: "Discard comment" and "Keep comment" | Dialog overlays the screen; entry unchanged until a button is tapped | "Discard comment": the entry is removed from the local queue (no Comment is created, no notification fires, per FEAT-07.SPEC-008), disappears from the thread area, and the dialog closes. "Keep comment" (or Escape / tapping outside the dialog): the dialog closes and the entry stays in Sync Failed, unchanged |

### Accessibility Notes

- **Focus order:** Header (milestone name, project name, status) -> thread entries in chronological order (each entry's Edit/Retract controls follow that entry's text) -> composer text input -> Post button.
- **New comment announcement:** A newly posted comment's appearance is announced to assistive technology as a live-region update.
- **Validation announcement:** A composer error is announced and programmatically associated with the composer input.
- **Retraction announcement:** A completed retraction's placeholder is announced as a live-region update.
- **Sync Failed announcement:** When an entry enters Sync Failed, its "Not sent" label and reason message are announced as a live-region update; the discard confirmation dialog traps focus, places initial focus on "Keep comment", and returns focus to the entry's Edit or Discard control (or the composer if the entry was discarded) when it closes.
- **Keyboard alternatives:** Post, Edit, Save, Cancel, Retract, Retry, Discard, and the dialog buttons are all reachable and operable by keyboard; no pointer-only gestures exist on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty | "No feedback yet" message in place of the thread list; composer remains available | Screen loads and no comments exist for this Milestone | The first comment is posted |
| Loading | Thread area shows a lightweight in-progress indicator; composer present but inactive | Screen first opens | Thread content finishes loading |
| Populated (default) | Full chronological thread with composer active below it | Load completes and at least one comment exists | Screen is left or reloaded |
| Error (submission failed) | Inline message below the composer: "Couldn't post your comment. Check your connection and try again." Typed text preserved; Retry offered. | A Post attempt fails server-side | Retry succeeds, or the viewer edits and posts again |
| Offline/Degraded | Banner above the composer: "You're offline -- this comment will send when you reconnect." Composer editable; Post queues via FEAT-07.SPEC-008. | Connectivity is lost while composing or at the moment Post is tapped | Connectivity returns -- the queued comment submits automatically and appears once the sync succeeds |
| Sync Failed (unsent comment) | The queued comment appears as an entry after the last posted comment, labeled "Not sent", with the author's text and one reason message plus controls by failure reason: (a) validation failure -- "This comment couldn't be sent because it no longer passes the comment checks. Edit it or discard it." with Edit and Discard; (b) authorization failure -- "This comment couldn't be sent because your access has changed." with Discard only (no Edit or Retry, since either would fail identically); (c) repeated server error across multiple connectivity events -- "This comment hasn't been sent yet. Try again or discard it." with Retry and Discard. The composer stays available for new comments. | FEAT-07.SPEC-008 marks the entry Sync Failed after a reconnect sync attempt fails on validation, authorization, or (after repeated failures) server error | The entry is Edited and saved (returns to Queued, then posts), Retried successfully (posts), or Discarded after confirmation (removed). An authorization-failed entry can exit only via Discard; no Sync Failed entry is ever sent automatically without the author's action |

## Validation Rules

Validation governed by FEAT-07.SPEC-005 (Comment Content & Submission Validation). Applied on Post tap (new comment) and on Save tap (inline edit), identically to FEAT-07.SPEC-001.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Successful post | Stays on this screen, thread updated | -- |

## Data Model

**Creates:** Comment -- `text` (from composer), `author` (the signed-in contact or Nadia), `posted_at` (current time at successful write), `target` (this Milestone), `reply_to` (never set from this screen), `status` (Posted).
**Reads:** Comment -- all fields, for every comment scoped by FEAT-07.SPEC-007's visibility rule and pinned to this Milestone, ordered by `posted_at`. Milestone -- `name`, `status`, for the header. Client Contact -- the signed-in contact's `name`, `role`, `status`, for authorship display and Edit/Retract gating.
**Local queue (not a tracked entity):** an unsent entry's text is replaced on Save of an edit, and the entry is removed on confirmed Discard -- both operate on the device-local queue owned by FEAT-07.SPEC-008, never on a Comment record.
**Updates:** Comment -- `text` (author's own comment, within the grace window, via FEAT-07.SPEC-006); `status` (Posted -> Retracted, author's own comment, via FEAT-07.SPEC-006).
**Deletes:** None -- retraction is a soft status change, never a hard delete.

## Business Rules

- Comment content validation (FEAT-07.SPEC-005) is enforced on every Post and every inline edit.
- Edit and retraction eligibility (FEAT-07.SPEC-006) governs the Edit and Retract controls, identically to FEAT-07.SPEC-001.
- Visibility and authorization (FEAT-07.SPEC-007) governs which comments render and which controls appear.
- XBR-09: Client isolation -- a contact reaches only their own company's milestone thread.
- A retracted comment's replies remain visible; retraction never cascades.
- Comments are append-only and ordered strictly by `posted_at`, per the dependency map's Comment Contention note.
- An unsent (Sync Failed) comment is never presented as posted and is never discarded without the confirmation dialog; its Edit control is offered only for a validation failure and its Retry control only for a repeated server-error failure (FEAT-07.SPEC-008 Outcome Definitions).
- This screen never renders an Approve control -- approval belongs entirely to FEAT-08; Priya's comment-only entitlement is enforced by omission, not by a disabled control (Feature Breakdown Brief, Side-Effect Inventory).

## Edge Cases

- **Several contacts from the same company post at effectively the same moment** -- Each comment is written independently and ordered by `posted_at`; no conflict resolution is needed beyond that ordering, per the dependency map's Comment Contention note.
- **Author attempts to edit after the grace window has closed** -- The Edit control is no longer shown; a direct attempt is rejected by FEAT-07.SPEC-006 with "This comment can no longer be edited."
- **Author retracts a comment that already has later replies** -- The retraction proceeds; the placeholder shows in place while later replies remain fully visible.
- **Comment submission fails after Post is tapped** -- Typed text is preserved, the Error state appears with Retry, and no partial or duplicate comment is created.
- **Viewer taps Post twice rapidly** -- The second tap is ignored while the first is in progress; only one comment is created.
- **Owen approves the milestone (FEAT-08) while this thread is open in another tab** -- The thread itself is unaffected; the milestone's status label in this screen's header updates on next load or reload, since this screen is a snapshot of thread content, not a live-updating approval indicator.
- **Priya opens a thread link for a milestone belonging to a different client company** -- Denied per XBR-09: out-of-scope link handling, plain explanation and a fresh-link option.
- **Dana opens this screen inside a support session** -- Full thread renders read-only; composer, Edit, and Retract controls are not rendered.
- **A queued comment fails sync while the viewer is on another screen** -- The entry remains in the device-local queue in Sync Failed; the next time this thread opens on that device, the entry appears in the thread area with its reason message and controls.
- **Unsent comment viewed from another device or by Dana** -- Unsent entries are device-local (FEAT-07.SPEC-008); no other device, session, or viewer (including Dana in a support session) ever sees them, and discarding leaves no record anywhere.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-07.SPEC-005 (Comment Content & Submission Validation) | References (outbound) | Text validation on every Post and inline edit |
| FEAT-07.SPEC-006 (Comment Edit Window & Retraction Rule) | References (outbound) | Governs the Edit and Retract controls |
| FEAT-07.SPEC-007 (Comment Visibility & Authorization Rule) | References (outbound) | Governs who sees this thread and which controls each viewer gets |
| FEAT-07.SPEC-008 (Offline Comment Queue & Sync) | Triggers (outbound) | Post while offline hands off to the queue-and-sync automation; its Sync Failed outcomes are displayed here, and this screen's Edit, Retry, and Discard controls act on its queue entries |
| FEAT-07.SPEC-003 (Client Comment Alert to Freelancer) | Triggered by this screen's Post action (outbound); Navigation (inbound) | A client contact's post fires this notification; Nadia arrives here from its email link |
| FEAT-07.SPEC-004 (Freelancer Reply Alert to Client) | Triggered by this screen's Post action (outbound) | Nadia's reply here fires this notification to the client contact(s) |
| FEAT-05.SPEC-003 (Portal Home) | Navigation (inbound) | Owen or Priya arrives from their portal home |
| FEAT-01.SPEC-005 (Project Detail) | Navigation (inbound) | Nadia arrives from her project workspace |
| FEAT-04.SPEC-002 (Milestone Timeline (Client View)) | Navigation (inbound) | Owen or Priya taps a milestone row with no deliverable ready for review yet |
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Navigation (inbound) | Owen or Nadia opens the full thread from the approval screen's embedded context |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Navigation (inbound) | Dana reaches this screen inside a logged support session |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|---------------|-------------------|
| milestone_comment_posted | milestone reference, author role, comment length | A comment is successfully created (online or after an offline sync succeeds) | supports success-metrics.md: "Feedback Consolidation" |
| comment_retracted | milestone reference, author role, time since posting | A retraction is successfully recorded | N/A -- no success-metrics.md metric measures retraction volume; retained for feature-health visibility only |
| comment_post_failed | failure reason (validation, server error) | A Post attempt does not complete successfully | N/A -- no connected success metric measures posting failures; recorded as diagnostic exhaust from the composer flow |

## Acceptance Criteria

**FEAT-07.SPEC-002-AC-01:** Given Priya is on the Milestone Comment Thread for her own company's milestone, when the screen loads and no comments exist, then she sees "No feedback yet" and an active composer.

**FEAT-07.SPEC-002-AC-02:** Given Priya types a valid comment and taps Post, when the submission succeeds, then the composer clears and her comment appears at the end of the thread with no separate confirmation screen.

**FEAT-07.SPEC-002-AC-03:** Given Owen and Priya have both commented on the same milestone, when Nadia opens the thread, then she sees every comment from both, ordered by posting time.

**FEAT-07.SPEC-002-AC-04:** Given Nadia is the author of a comment posted moments ago, when she taps Edit within the grace window, then the comment becomes editable and saving shows the updated text marked "(edited)".

**FEAT-07.SPEC-002-AC-05:** Given Owen's comment is older than the grace window, when he views the thread, then no Edit control is shown, only Retract.

**FEAT-07.SPEC-002-AC-06:** Given Priya retracts her own comment, when the retraction is confirmed, then the comment's text is replaced by a "This comment was retracted" placeholder, and any later replies remain fully visible.

**FEAT-07.SPEC-002-AC-07:** Given Owen taps Post with an empty composer, when validation runs via FEAT-07.SPEC-005, then he sees "Enter a comment before posting." and no comment is created.

**FEAT-07.SPEC-002-AC-08:** Given a comment submission fails server-side, when the failure occurs, then the typed text remains in the composer and an inline error with a Retry option is shown.

**FEAT-07.SPEC-002-AC-09:** Given Priya taps Post twice in rapid succession, when the first submission is still in progress, then the second tap has no effect and only one comment is created.

**FEAT-07.SPEC-002-AC-10:** Given Priya (Reviewer) opens a link to a milestone belonging to a different client company, when the screen would otherwise load, then she sees a plain explanation and a fresh-link option, never that company's comments.

**FEAT-07.SPEC-002-AC-11:** Given Dana (Support Operator) opens this screen inside a logged support session, when the screen loads, then the full thread renders read-only with no composer, Edit, or Retract control shown.

**FEAT-07.SPEC-002-AC-12:** Given an unauthenticated visitor opens a milestone thread link, when the screen would otherwise load, then they are redirected to the client portal's magic-link sign-in.

**FEAT-07.SPEC-002-AC-13:** Given Owen's sign-in session has expired, when he opens the thread link, then he sees a plain explanation and a "request a fresh link" option, with no thread content shown.

**FEAT-07.SPEC-002-AC-14:** Given Nadia loses connectivity while composing a reply, when she taps Post, then the banner "You're offline -- this comment will send when you reconnect." appears and the comment is handed to FEAT-07.SPEC-008.

**FEAT-07.SPEC-002-AC-15:** Given Nadia is viewing the thread at a compact breakpoint, then the composer's Post button appears below the text input rather than beside it.

**FEAT-07.SPEC-002-AC-16:** Given Priya (Reviewer) is viewing this screen, when she looks for an Approve control, then none is shown -- only the composer and thread, consistent with her comment-only entitlement.

**FEAT-07.SPEC-002-AC-17:** Given a comment is successfully posted, when the write completes, then a milestone_comment_posted analytics event is emitted citing the milestone and comment length.

**FEAT-07.SPEC-002-AC-18:** Given Nadia's queued comment failed FEAT-07.SPEC-005 validation on reconnect, when she opens the thread, then the comment appears after the last posted comment labeled "Not sent" with the message "This comment couldn't be sent because it no longer passes the comment checks. Edit it or discard it." and Edit and Discard controls.

**FEAT-07.SPEC-002-AC-19:** Given Owen's queued comment failed sync because his contact status was set to Removed, when the entry shows as Sync Failed, then it displays "This comment couldn't be sent because your access has changed." with only a Discard control -- no Edit and no Retry.

**FEAT-07.SPEC-002-AC-20:** Given Priya has an unsent Sync Failed comment, when she taps Discard, then a dialog titled "Discard this comment?" appears with "This comment hasn't been sent. If you discard it, it will be permanently deleted from this device and can't be recovered." and buttons "Discard comment" and "Keep comment", and the entry is unchanged until she taps one.

**FEAT-07.SPEC-002-AC-21:** Given the Discard confirmation dialog is open, when Priya taps "Discard comment", then the entry is removed from the thread area, no Comment record is created, and no notification fires; when she instead taps "Keep comment", then the dialog closes and the entry remains in Sync Failed unchanged.

**FEAT-07.SPEC-002-AC-22:** Given Nadia taps Edit on a validation-failed unsent comment and taps Save with valid text, when validation via FEAT-07.SPEC-005 passes, then the entry leaves Sync Failed, is resubmitted by FEAT-07.SPEC-008, and appears as a posted comment once the sync succeeds; and if she taps Save with invalid text, then an inline error shows and the edit stays open.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 12 | 12 |
| States | 6 (empty, loading, populated, error, offline, sync failed) | 6 |
| Business Rules | 8 | 8 |
| Edge Cases | 10 | 10 |
