---
document_type: spec
spec_type: screen
spec_id: FEAT-07.SPEC-001
spec_name: Deliverable Comment Thread
spec_slug: deliverable-comment-thread
parent_feature: FEAT-07
parent_feature_name: Deliverable Review & Feedback
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 23
---

# Screen Spec: Deliverable Comment Thread

## Overview

**Name:** Deliverable Comment Thread
**ID:** FEAT-07.SPEC-001
**Type:** Screen
**Purpose:** A contact or Nadia views every comment pinned to one deliverable version in a single chronological thread, posts a new comment, and retracts their own earlier comment -- replacing scattered WhatsApp screenshots with one recorded place per deliverable.
**Parent Feature:** FEAT-07 -- Deliverable Review & Feedback

## Scope and Non-Goals

**In Scope:**
- Rendering the full chronological comment thread pinned to one Deliverable Version, scoped by the viewer's role and company
- The composer for posting a new comment pinned to that same Deliverable Version
- Editing the author's own comment's text within the short grace window defined by FEAT-07.SPEC-006
- Retracting the author's own comment (soft removal, always available to its author) per FEAT-07.SPEC-006
- The "no feedback yet" empty state, the loading indicator, the submission-failure retry state, and the offline-composing state
- The "Sync Failed" state for a queued offline comment that could not be sent on reconnect (FEAT-07.SPEC-008), with Edit, Retry, and Discard-with-confirmation controls for that unsent comment
- Showing Nadia's, Owen's, and Priya's own-company thread and Dana's read-only view inside a logged support session

**Non-Goals:**
- Milestone-level (whole-round) comments -- handled by the sibling screen FEAT-07.SPEC-002 (Milestone Comment Thread); this screen only ever pins to a single Deliverable Version.
- Nested reply threading UI -- the Feature Breakdown Brief's Shared UI Patterns define a single flat chronological thread, not a nested reply tree; the Comment entity's `reply_to` field exists at the data layer but no interaction on this screen sets it, so Nadia's replies appear simply as later comments in the same chronological order, distinguishable by author identity.
- Comment content and length validation logic -- owned entirely by FEAT-07.SPEC-005 (Comment Content & Submission Validation); this screen calls that rule rather than duplicating it.
- Who may view, post, or act on this thread -- owned entirely by FEAT-07.SPEC-007 (Comment Visibility & Authorization Rule); this screen renders according to that rule's outcome rather than deriving access itself.
- Approving the milestone or viewing invoice/proposal content from this screen -- those actions belong to FEAT-08 and other features; this screen's only actions are reading, posting, editing within the grace window, and retracting comments.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05 (Client Portal Access, Portal Home, FEAT-05.SPEC-003) | Owen or Priya opens a deliverable waiting for review from their portal home | The Deliverable Version reference for that deliverable's latest round |
| FEAT-06 (Deliverable Upload & Sharing, Deliverable List & Management, FEAT-06.SPEC-002) | Nadia opens a specific deliverable from her project workspace | The Deliverable Version reference for the round Nadia opened |
| FEAT-07.SPEC-003 (Client Comment Alert to Freelancer) | Nadia opens the "new feedback is waiting" email and follows its link | The specific Deliverable Version the comment was posted against |
| FEAT-17 (Deliverable Version History, Version Browser & Comparison, FEAT-17.SPEC-002) | A viewer opens an earlier round of the deliverable from the version selector | The earlier Deliverable Version reference; this screen reloads pinned to that version's own thread (XBR-13) |
| FEAT-08 (Milestone Approval, Milestone Review & Approval Screen, FEAT-08.SPEC-001) | Owen or Nadia opens the full deliverable-level thread from the approval screen's embedded context summary | The Deliverable Version reference the approval screen was showing context for |
| FEAT-31 (Operator Support Access, Operator Support Session Console, FEAT-31.SPEC-002) | Dana opens this screen while working through a freelancer's own screens inside a logged support session | The Deliverable Version reference; session is announced and read-only |
| FEAT-06.SPEC-006 (Deliverable Ready Notification) | Owen or Priya taps the email's "Review deliverable" CTA and signs in via FEAT-05 | The specific Deliverable referenced by the notification, at its latest Deliverable Version |
| FEAT-07.SPEC-004 (Freelancer Reply Alert to Client) | Owen or Priya taps the email's "Open thread" CTA | The specific Deliverable Version the reply was posted against |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full thread -- every comment on this Deliverable Version from every authorized contact and herself; her own unsent (Sync Failed) comments on this device | Post a comment; edit her own comment within the grace window; retract her own comment; edit, retry, or discard her own unsent comment | -- |
| Owen (Client Primary Contact) | Full thread, Own-only -- only for his own company's deliverables (FEAT-07.SPEC-007); his own unsent (Sync Failed) comments on this device | Post a comment; edit his own comment within the grace window; retract his own comment; edit, retry, or discard his own unsent comment | Attempting to reach a thread for a deliverable outside his own company is treated as an out-of-scope link: plain explanation and a fresh-link option (XBR-09), never another company's comments |
| Priya (Client Reviewer Contact) | Full thread, Own-only -- only for her own company's deliverables (FEAT-07.SPEC-007); her own unsent (Sync Failed) comments on this device | Post a comment; edit her own comment within the grace window; retract her own comment; edit, retry, or discard her own unsent comment | Same out-of-scope handling as Owen for a deliverable outside her own company |
| Dana (Support Operator) | Full thread, read-only, inside a logged support session (FEAT-31) | No actions -- composer, edit, and retract controls are not rendered | If Dana attempts an action outside the read-only bounds, no control exists to attempt it with |
| Unauthenticated | No | No | Redirected to the client portal's magic-link sign-in (FEAT-05); after signing in as a recognized contact, lands on this screen if the deliverable belongs to their company |
| Expired session | No | No | Magic link is single-use and time-limited (XBR-28); shows a plain explanation and a "request a fresh link" option; no thread content shown |

## Layout and Content

**Header:** Deliverable display label (derived per Data Model, below -- there is no dedicated name field on the Deliverable entity) and its current round indicator ("Round {round_number}"), with a link to the version selector (FEAT-17.SPEC-002) when more than one round exists.

**Body:** A single-column chronological thread, above a composer, in this order:
- Thread list -- one entry per comment, each showing author name, author role indicator (e.g., "Reviewer" beside Priya's name), posted timestamp, and comment text; a retracted comment shows a placeholder ("This comment was retracted") in its original chronological position instead of its text
- "Edit" and "Retract" controls appear only on the viewer's own comments, and only "Retract" appears once the comment's edit grace window has closed (FEAT-07.SPEC-006); an "(edited)" marker appears beside any comment edited after posting
- Empty-state message "No feedback yet" in place of the thread list when no comments exist
- Unsent comment entries (Sync Failed state) -- shown after the last posted comment and above the composer, visually distinct from posted comments, each with a "Not sent" label, the author's text, a reason message, and the controls that apply to that failure reason (Edit, Retry, Discard); visible only on the device that queued them and only to their author
- Composer -- positioned below the thread list, at the bottom of the body area: a multi-line text input, a character count as the author approaches the limit, and a "Post" button

**Footer:** None -- the composer sits in the body, immediately after the thread.

### Responsive Behavior

- **Compact breakpoint:** Single-column thread and composer as described, full width; the composer's text input and Post button stack with Post below the input.
- **Medium size class and above:** Thread and composer content are capped at a consistent platform-wide reading width and horizontally centered; the composer's Post button sits to the right of the text input rather than stacked beneath it.
- **Thread entries:** Uniform scaling, no structural change across breakpoints.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Thread list | Screen loads | Reads every Comment pinned to this Deliverable Version, scoped by FEAT-07.SPEC-007's visibility rule, ordered by `posted_at` | Thread renders with all visible comments | Loading indicator while the thread loads, then full thread appears |
| Version link (header) | Tap | Navigates to FEAT-17.SPEC-002 (Version Browser & Comparison) | Screen transitions to the version selector | Standard navigation transition |
| Composer text input | Type | Captures the comment's text as the author composes it | Character count updates | Standard input focus state; count turns visually distinct near the 2,000-character limit |
| Post button | Tap | 1. Validates text via FEAT-07.SPEC-005 (Comment Content & Submission Validation). 2. If valid and online, creates the Comment pinned to this Deliverable Version; the successful write fires FEAT-07.SPEC-003 (Client Comment Alert to Freelancer) when the author is Owen or Priya, or FEAT-07.SPEC-004 (Freelancer Reply Alert to Client) when the author is Nadia. 3. If valid and offline, hands off to FEAT-07.SPEC-008 (Offline Comment Queue & Sync). 4. If invalid, shows the field-level error from FEAT-07.SPEC-005. | Button shows a brief loading state during an online submission | Success (online): composer clears and the new comment appears at the end of the thread immediately, no separate confirmation screen. Success (offline): see Offline/Degraded state. Failure (invalid text): inline error below the composer. Failure (server error): see Error state. |
| Edit control (own comment, within grace window) | Tap | Opens the comment's text in an inline editable field, per FEAT-07.SPEC-006 | Comment's display text is replaced by an editable field with Save/Cancel | Standard inline-edit affordance |
| Save (inline edit) | Tap | Validates the edited text via FEAT-07.SPEC-005, then writes the update per FEAT-07.SPEC-006 | Comment returns to display mode, now marked "(edited)" | Success: comment updates in place. Failure (invalid text or window closed): inline error, edit remains open for correction or cancel |
| Cancel (inline edit) | Tap | Discards the in-progress edit | Comment returns to display mode, unchanged | No feedback needed -- return is immediate |
| Retract control (own comment) | Tap | Opens a confirmation, then writes the retraction per FEAT-07.SPEC-006 | Comment's text is replaced by the retracted placeholder in its original position | Confirmation dialog before the action; once confirmed, the placeholder appears immediately |
| Retry (on submission failure) | Tap | Re-attempts the same Post action with the preserved composer text | Button returns to loading state | Standard retry feedback, same as the original Post attempt |
| Edit control (unsent Sync Failed comment, validation failure only) | Tap | Opens the unsent entry's text in an inline editable field; this edits the local queue entry, not a posted comment, so FEAT-07.SPEC-006's grace window does not apply (FEAT-07.SPEC-008 edge case) | Entry's text becomes an editable field with Save/Cancel | Standard inline-edit affordance |
| Save / Cancel (unsent entry edit) | Tap | Save: validates the edited text via FEAT-07.SPEC-005, replaces the queue entry's text, and returns the entry to Queued for FEAT-07.SPEC-008 to submit (immediately if online). Cancel: discards the in-progress edit | Save: entry leaves Sync Failed and shows the queued indicator (or posts and joins the thread once synced). Cancel: entry returns to Sync Failed display, unchanged | Save success: the "Not sent" label changes to "Sending..." then the comment appears in the thread on success. Save failure (invalid text): inline error, edit remains open. Cancel: immediate |
| Retry control (unsent entry, repeated server-error failure only) | Tap | Asks FEAT-07.SPEC-008 to attempt the entry's submission now | Entry shows "Sending..." while the attempt runs | Success: entry becomes a posted comment at the end of the thread. Failure: entry returns to Sync Failed with the same message |
| Discard control (unsent Sync Failed comment) | Tap | Opens a confirmation dialog titled "Discard this comment?" with the text "This comment hasn't been sent. If you discard it, it will be permanently deleted from this device and can't be recovered." and two buttons: "Discard comment" and "Keep comment" | Dialog overlays the screen; entry unchanged until a button is tapped | "Discard comment": the entry is removed from the local queue (no Comment is created, no notification fires, per FEAT-07.SPEC-008), disappears from the thread area, and the dialog closes. "Keep comment" (or Escape / tapping outside the dialog): the dialog closes and the entry stays in Sync Failed, unchanged |

### Accessibility Notes

- **Focus order:** Header (deliverable name, round indicator, version link) -> thread entries in chronological order (each entry's Edit/Retract controls, when present, follow that entry's text) -> composer text input -> Post button.
- **New comment announcement:** When a new comment is successfully posted, its appearance at the end of the thread is announced to assistive technology as a live-region update.
- **Validation announcement:** A composer error (empty, over-length) is announced when it appears and is programmatically associated with the composer input.
- **Retraction announcement:** When a retraction completes, the placeholder replacing the comment's text is announced as a live-region update.
- **Sync Failed announcement:** When an entry enters Sync Failed, its "Not sent" label and reason message are announced as a live-region update; the discard confirmation dialog traps focus, places initial focus on "Keep comment", and returns focus to the entry's Edit or Discard control (or the composer if the entry was discarded) when it closes.
- **Keyboard alternatives:** Post, Edit, Save, Cancel, Retract, Retry, Discard, and the dialog buttons are all standard activatable controls reachable and operable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Empty | "No feedback yet" message in place of the thread list; composer remains available | Screen loads and no comments exist for this Deliverable Version | The first comment is posted (this session or another viewer's) |
| Loading | Thread area shows a lightweight in-progress indicator; composer is present but inactive until load completes | Screen first opens | Thread content finishes loading |
| Populated (default) | Full chronological thread with composer active below it | Load completes and at least one comment exists | Screen is left or reloaded |
| Error (submission failed) | Inline message below the composer: "Couldn't post your comment. Check your connection and try again." Typed text remains in the composer; a Retry action is offered. | A Post attempt fails server-side | Retry succeeds, or the viewer edits the text and posts again |
| Offline/Degraded | Banner above the composer: "You're offline -- this comment will send when you reconnect." Composer remains editable; Post queues the comment locally via FEAT-07.SPEC-008 instead of submitting immediately. | Connectivity is lost while composing or at the moment Post is tapped | Connectivity returns -- the queued comment submits automatically and appears in the thread once the sync (FEAT-07.SPEC-008) succeeds |
| Sync Failed (unsent comment) | The queued comment appears as an entry after the last posted comment, labeled "Not sent", with the author's text and one reason message plus controls by failure reason: (a) validation failure -- "This comment couldn't be sent because it no longer passes the comment checks. Edit it or discard it." with Edit and Discard; (b) authorization failure -- "This comment couldn't be sent because your access has changed." with Discard only (no Edit or Retry, since either would fail identically); (c) repeated server error across multiple connectivity events -- "This comment hasn't been sent yet. Try again or discard it." with Retry and Discard. The composer stays available for new comments. | FEAT-07.SPEC-008 marks the entry Sync Failed after a reconnect sync attempt fails on validation, authorization, or (after repeated failures) server error | The entry is Edited and saved (returns to Queued, then posts), Retried successfully (posts), or Discarded after confirmation (removed). An authorization-failed entry can exit only via Discard; no Sync Failed entry is ever sent automatically without the author's action |

## Validation Rules

Validation governed by FEAT-07.SPEC-005 (Comment Content & Submission Validation). See that spec for the non-empty, 1--2,000 character rule. This screen applies it on Post tap (new comment) and on Save tap (inline edit).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Version link tap | FEAT-17.SPEC-002 (Version Browser & Comparison) | FEAT-17 (Deliverable Version History) |
| Successful post | Stays on this screen, thread updated | -- |

## Data Model

**Creates:** Comment -- `text` (from composer), `author` (the signed-in contact or Nadia), `posted_at` (current time at successful write), `target` (this Deliverable Version), `reply_to` (never set from this screen), `status` (Posted).
**Reads:** Comment -- all fields, for every comment scoped by FEAT-07.SPEC-007's visibility rule and pinned to this Deliverable Version, ordered by `posted_at`. Deliverable -- `status` (Uploading | Active | Superseded | Removed), and its display label for the header, derived per the note below since the entity carries no dedicated name field (FEAT-06's Deliverable field list -- dependency map: `kind`, `file or link`, `milestone`, `uploaded_at`, `size`, `status`, `link_status`, `first_client_view_at` -- has none). Deliverable Version -- `round_number`, `is_latest`, for the header. Client Contact -- the signed-in contact's `name`, `role`, `status`, to display authorship and to gate Edit/Retract to the viewer's own comments.

**Deliverable display-label derivation (Open Question for Pass D):** No field on the Deliverable entity holds a display name (FEAT-06's specs create and list a Deliverable by `kind`, `file or link`, `status`, `uploaded_at`, and `size` only -- see FEAT-06.SPEC-001 and FEAT-06.SPEC-002, neither of which names or labels a deliverable in its UI). Until FEAT-06 defines a dedicated field, this screen derives the header label as follows: for an uploaded file, the original file name captured as part of the `file or link` field at upload; for a linked external asset, the source platform name (Figma, Google Drive, or Dropbox, per FEAT-06.SPEC-002's source icon) followed by "link" (e.g., "Figma link"), since a link carries no file name to borrow. This derivation is recorded here as an open point for Pass D's cross-reference reconciliation against the Feature Dependency Map's Deliverable field list -- ideally FEAT-06 adds an explicit display-label field that this and every downstream reference (FEAT-07.SPEC-003, FEAT-07.SPEC-004) can cite directly instead of re-deriving.
**Local queue (not a tracked entity):** an unsent entry's text is replaced on Save of an edit, and the entry is removed on confirmed Discard -- both operate on the device-local queue owned by FEAT-07.SPEC-008, never on a Comment record.
**Updates:** Comment -- `text` (author's own comment, within the grace window, via FEAT-07.SPEC-006); `status` (Posted -> Retracted, author's own comment, via FEAT-07.SPEC-006).
**Deletes:** None -- retraction is a soft status change, never a hard delete (FEAT-07.SPEC-006).

## Business Rules

- Comment content validation (FEAT-07.SPEC-005) is enforced on every Post and every inline edit -- the viewer cannot submit invalid text.
- Edit and retraction eligibility (FEAT-07.SPEC-006) governs the Edit and Retract controls -- Edit disappears once the grace window closes; Retract remains available to the author indefinitely.
- Visibility and authorization (FEAT-07.SPEC-007) governs which comments render and which controls appear -- this screen never derives access itself.
- XBR-09: Client isolation -- a contact reaches only their own company's deliverable thread; an out-of-scope link shows a plain explanation and a fresh-link option, never another company's comments.
- A retracted comment's replies remain visible in the thread; retraction never cascades (Feature Breakdown Brief, Entity-Lifecycle Coverage Matrix).
- Comments are append-only and ordered strictly by `posted_at`; concurrent posts from different authors need no conflict resolution beyond that ordering (dependency map, Comment Contention note).
- An unsent (Sync Failed) comment is never presented as posted and is never discarded without the confirmation dialog; its Edit control is offered only for a validation failure and its Retry control only for a repeated server-error failure (FEAT-07.SPEC-008 Outcome Definitions).

## Edge Cases

- **Several contacts from the same company post at effectively the same moment** -- Each comment is written independently and the thread orders them by `posted_at`; no conflict resolution is needed beyond ordering, per the dependency map's Comment Contention note ("append-only... needs no resolution beyond ordering by posted time").
- **Author attempts to edit after the grace window has closed** -- The Edit control is no longer shown for that comment; a direct attempt (e.g., a stale UI state) is rejected by FEAT-07.SPEC-006 with "This comment can no longer be edited." and the comment reverts to display mode.
- **Author retracts a comment that already has later replies from other contacts** -- The retraction proceeds; the retracted comment shows the placeholder in place while every later reply remains fully visible and unaffected.
- **Comment submission fails after Post is tapped (server error)** -- The typed text is preserved in the composer, the Error state appears with a Retry option, and no partial or duplicate comment is created.
- **Viewer taps Post twice rapidly** -- The second tap is ignored while the first submission is in progress (button in loading state); only one comment is created.
- **Priya opens a thread link for a deliverable belonging to a different client company** -- Denied per XBR-09: treated as an out-of-scope link, plain explanation and a fresh-link option, never another company's comments.
- **Dana opens this screen inside a support session** -- Full thread renders read-only; composer, Edit, and Retract controls are not rendered at all.
- **Viewer loses connectivity mid-composition and later reconnects** -- The composer preserves the typed text throughout; tapping Post while offline queues the comment via FEAT-07.SPEC-008 rather than losing the draft.
- **A viewer opens this thread while its Deliverable is in Removed or Superseded status** -- The thread and composer still load: a removed or superseded deliverable's feedback history remains part of the record (Comment retention, per this feature's Non-Goals), and Nadia may still need to reply to standing feedback on an earlier round. The header's status context reflects the deliverable's actual current status (Removed or Superseded) rather than implying it is still the active round; for a Removed deliverable, the composer stays available since retraction and reply behavior are unaffected by the parent deliverable's own status -- only FEAT-06.SPEC-005 governs whether the deliverable itself can be acted on, which this screen never does.

- **A queued comment fails sync while the viewer is on another screen** -- The entry remains in the device-local queue in Sync Failed; the next time this thread opens on that device, the entry appears in the thread area with its reason message and controls.
- **Unsent comment viewed from another device or by Dana** -- Unsent entries are device-local (FEAT-07.SPEC-008); no other device, session, or viewer (including Dana in a support session) ever sees them, and discarding leaves no record anywhere.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-07.SPEC-005 (Comment Content & Submission Validation) | References (outbound) | Text validation on every Post and inline edit |
| FEAT-07.SPEC-006 (Comment Edit Window & Retraction Rule) | References (outbound) | Governs the Edit and Retract controls and their eligibility |
| FEAT-07.SPEC-007 (Comment Visibility & Authorization Rule) | References (outbound) | Governs who sees this thread and which controls each viewer gets |
| FEAT-07.SPEC-008 (Offline Comment Queue & Sync) | Triggers (outbound) | Post while offline hands off to the queue-and-sync automation; its Sync Failed outcomes are displayed here, and this screen's Edit, Retry, and Discard controls act on its queue entries |
| FEAT-07.SPEC-003 (Client Comment Alert to Freelancer) | Triggered by this screen's Post action (outbound); Navigation (inbound) | A client contact's post fires this notification; Nadia arrives here from its email link |
| FEAT-07.SPEC-004 (Freelancer Reply Alert to Client) | Triggered by this screen's Post action (outbound) | Nadia's reply on this screen fires this notification to the client contact(s) |
| FEAT-05.SPEC-003 (Portal Home) | Navigation (inbound) | Owen or Priya arrives from their portal home |
| FEAT-06.SPEC-002 (Deliverable List & Management) | Navigation (inbound) | Nadia arrives from her deliverable workspace |
| FEAT-17.SPEC-002 (Version Browser & Comparison) | Navigation (outbound and inbound) | The version link navigates there; opening an earlier round navigates back here pinned to that version |
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Navigation (inbound) | Owen or Nadia opens the full thread from the approval screen's embedded context |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Navigation (inbound) | Dana reaches this screen inside a logged support session |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|---------------|-------------------|
| comment_posted | deliverable version reference, author role, comment length | A comment is successfully created (online or after an offline sync succeeds) | supports success-metrics.md: "Feedback Consolidation" |
| comment_retracted | deliverable version reference, author role, time since posting | A retraction is successfully recorded | N/A -- no success-metrics.md metric measures retraction volume; retained for feature-health visibility only |
| comment_post_failed | failure reason (validation, server error) | A Post attempt does not complete successfully | N/A -- no connected success metric measures posting failures; recorded as diagnostic exhaust from the composer flow |

## Acceptance Criteria

**FEAT-07.SPEC-001-AC-01:** Given Priya is on the Deliverable Comment Thread for her own company's deliverable, when the screen loads and no comments exist, then she sees "No feedback yet" and an active composer.

**FEAT-07.SPEC-001-AC-02:** Given Priya types a valid comment and taps Post, when the submission succeeds, then the composer clears and her comment appears at the end of the thread with no separate confirmation screen.

**FEAT-07.SPEC-001-AC-03:** Given Owen and Priya have both commented on the same deliverable, when Nadia opens the thread, then she sees every comment from both, ordered by posting time.

**FEAT-07.SPEC-001-AC-04:** Given Nadia is the author of a comment posted moments ago, when she taps Edit within the grace window, then the comment becomes editable and saving shows the updated text marked "(edited)".

**FEAT-07.SPEC-001-AC-05:** Given Owen's comment is older than the grace window, when he views the thread, then no Edit control is shown for that comment, only Retract.

**FEAT-07.SPEC-001-AC-06:** Given Priya retracts her own comment, when the retraction is confirmed, then the comment's text is replaced by a "This comment was retracted" placeholder in its original position, and any later replies remain fully visible.

**FEAT-07.SPEC-001-AC-07:** Given Owen taps Post with an empty composer, when validation runs via FEAT-07.SPEC-005, then he sees "Enter a comment before posting." and no comment is created.

**FEAT-07.SPEC-001-AC-08:** Given a comment submission fails server-side, when the failure occurs, then the typed text remains in the composer and an inline error with a Retry option is shown.

**FEAT-07.SPEC-001-AC-09:** Given Priya taps Post twice in rapid succession, when the first submission is still in progress, then the second tap has no effect and only one comment is created.

**FEAT-07.SPEC-001-AC-10:** Given Priya (Reviewer, out-of-scope company) opens a link to a deliverable belonging to a different client company, when the screen would otherwise load, then she sees a plain explanation and a fresh-link option, never that company's comments.

**FEAT-07.SPEC-001-AC-11:** Given Dana (Support Operator) opens this screen inside a logged support session, when the screen loads, then the full thread renders read-only with no composer, Edit, or Retract control shown.

**FEAT-07.SPEC-001-AC-12:** Given an unauthenticated visitor opens a deliverable thread link, when the screen would otherwise load, then they are redirected to the client portal's magic-link sign-in.

**FEAT-07.SPEC-001-AC-13:** Given Owen's sign-in session has expired, when he opens the thread link, then he sees a plain explanation and a "request a fresh link" option, with no thread content shown.

**FEAT-07.SPEC-001-AC-14:** Given Nadia loses connectivity while composing a reply, when she taps Post, then the banner "You're offline -- this comment will send when you reconnect." appears and the comment is handed to FEAT-07.SPEC-008 rather than lost.

**FEAT-07.SPEC-001-AC-15:** Given Nadia is viewing the thread at a compact breakpoint, then the composer's Post button appears below the text input rather than beside it.

**FEAT-07.SPEC-001-AC-16:** Given more than one round exists for the deliverable, when Priya taps the version link in the header, then she navigates to FEAT-17.SPEC-002 (Version Browser & Comparison).

**FEAT-07.SPEC-001-AC-17:** Given a comment is successfully posted, when the write completes, then a comment_posted analytics event is emitted citing the deliverable version and comment length.

**FEAT-07.SPEC-001-AC-18:** Given Owen tries to edit a comment after its grace window has just closed, when he taps Edit, then the attempt is rejected with "This comment can no longer be edited." and the comment stays in display mode.

**FEAT-07.SPEC-001-AC-19:** Given Nadia's queued comment failed FEAT-07.SPEC-005 validation on reconnect, when she opens the thread, then the comment appears after the last posted comment labeled "Not sent" with the message "This comment couldn't be sent because it no longer passes the comment checks. Edit it or discard it." and Edit and Discard controls.

**FEAT-07.SPEC-001-AC-20:** Given Owen's queued comment failed sync because his contact status was set to Removed, when the entry shows as Sync Failed, then it displays "This comment couldn't be sent because your access has changed." with only a Discard control -- no Edit and no Retry.

**FEAT-07.SPEC-001-AC-21:** Given Priya has an unsent Sync Failed comment, when she taps Discard, then a dialog titled "Discard this comment?" appears with "This comment hasn't been sent. If you discard it, it will be permanently deleted from this device and can't be recovered." and buttons "Discard comment" and "Keep comment", and the entry is unchanged until she taps one.

**FEAT-07.SPEC-001-AC-22:** Given the Discard confirmation dialog is open, when Priya taps "Discard comment", then the entry is removed from the thread area, no Comment record is created, and no notification fires; when she instead taps "Keep comment", then the dialog closes and the entry remains in Sync Failed unchanged.

**FEAT-07.SPEC-001-AC-23:** Given Nadia taps Edit on a validation-failed unsent comment and taps Save with valid text, when validation via FEAT-07.SPEC-005 passes, then the entry leaves Sync Failed, is resubmitted by FEAT-07.SPEC-008, and appears as a posted comment once the sync succeeds; and if she taps Save with invalid text, then an inline error shows and the edit stays open.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 13 | 13 |
| States | 6 (empty, loading, populated, error, offline, sync failed) | 6 |
| Business Rules | 7 | 7 |
| Edge Cases | 11 | 11 |
