# Feature Specification: Deliverable Review & Feedback

**Blueprint feature:** FEAT-07
**Priority tier:** Core
**Build order:** 008 of 33
**Depends on:** FEAT-06
**Blueprint source:** `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Deliverable Comment Thread (Priority: P1)

A contact or Nadia views every comment pinned to one deliverable version in a single chronological thread, posts a new comment, and retracts their own earlier comment -- replacing scattered WhatsApp screenshots with one recorded place per deliverable.

**Acceptance Scenarios:**

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

### User Story 2 - Milestone Comment Thread (Priority: P1)

A contact or Nadia views and posts general feedback pinned to a milestone as a whole -- for a round that is not about one specific file -- in the same chronological thread pattern as the deliverable-level thread, and retracts their own comment.

**Acceptance Scenarios:**

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

### User Story 3 - Client Comment Alert to Freelancer (Priority: P1)

Emails Nadia the instant a client contact posts a comment on a deliverable or a milestone, so she learns feedback is waiting without checking the portal on a schedule.

**Acceptance Scenarios:**

**FEAT-07.SPEC-003-AC-01:** Given Owen posts a valid comment on a Deliverable Version, when FEAT-07.SPEC-001 records it, then Nadia receives an email with the subject naming the client company and the deliverable, quoting the comment text.

**FEAT-07.SPEC-003-AC-02:** Given Priya posts a valid comment on a Milestone, when FEAT-07.SPEC-002 records it, then Nadia receives an email with the subject naming the client company and the milestone, quoting the comment text.

**FEAT-07.SPEC-003-AC-03:** Given Nadia receives either variant of this email, when she taps "Open thread", then she lands on the corresponding thread screen (FEAT-07.SPEC-001 or FEAT-07.SPEC-002) for the specific deliverable or milestone the comment was posted against.

**FEAT-07.SPEC-003-AC-04:** Given a client comment is recorded, when this notification's trigger fires, then no on/off preference is available to Nadia to suppress it -- it always sends.

**FEAT-07.SPEC-003-AC-05:** Given a client comment is recorded during Nadia's local nighttime hours, when this notification's trigger fires, then it sends immediately with no quiet-hours hold.

**FEAT-07.SPEC-003-AC-06:** Given Nadia posts a comment herself (a reply), when the write succeeds, then this notification's trigger never fires, since the author is Nadia, not a client contact.

**FEAT-07.SPEC-003-AC-07:** Given Nadia's email address bounces on the first delivery attempt, when the delivery capability retries, then up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the affected project.

**FEAT-07.SPEC-003-AC-08:** Given Owen and Priya each post a comment on the same deliverable moments apart, when each is recorded, then Nadia receives two separate emails, one per comment, never merged.

**FEAT-07.SPEC-003-AC-09:** Given a client's comment composed offline syncs successfully hours later via FEAT-07.SPEC-008, when the sync completes, then this notification fires at that moment, not at the original offline composition time.

**FEAT-07.SPEC-003-AC-10:** Given this notification is successfully delivered, when the analytics signal is emitted, then a client_comment_alert_sent event is recorded with the pin target type and author role.

### User Story 4 - Freelancer Reply Alert to Client (Priority: P1)

Emails the client contact(s) entitled to a thread the instant Nadia replies in it, so a client waiting on her response is not left checking the portal to find out.

**Acceptance Scenarios:**

**FEAT-07.SPEC-004-AC-01:** Given Nadia replies with a valid comment on a Deliverable Version thread, when FEAT-07.SPEC-001 records it, then Owen receives an email naming Nadia's business and the deliverable, quoting the reply text.

**FEAT-07.SPEC-004-AC-02:** Given Nadia replies with a valid comment on a Milestone thread, when FEAT-07.SPEC-002 records it, then the entitled client contact(s) receive an email naming Nadia's business and the milestone, quoting the reply text.

**FEAT-07.SPEC-004-AC-03:** Given both Owen and Priya are entitled to the same thread, when Nadia replies, then each receives their own separate copy of the notification.

**FEAT-07.SPEC-004-AC-04:** Given a client contact receives this email, when they tap "Open thread", then they land on the corresponding thread screen (FEAT-07.SPEC-001 or FEAT-07.SPEC-002) for the specific deliverable or milestone.

**FEAT-07.SPEC-004-AC-05:** Given Nadia's reply is recorded, when this notification's trigger fires, then no on/off preference is available to any recipient to suppress it -- it always sends.

**FEAT-07.SPEC-004-AC-06:** Given Nadia's reply is recorded during a client contact's local nighttime hours, when this notification's trigger fires, then it sends immediately with no quiet-hours hold.

**FEAT-07.SPEC-004-AC-07:** Given a client contact posts a comment (not Nadia), when the write succeeds, then this notification's trigger never fires for that comment, since the author is not Nadia.

**FEAT-07.SPEC-004-AC-08:** Given a client contact's status has changed to Removed between Nadia's reply and delivery, when recipients are resolved, then that contact does not receive this notification.

**FEAT-07.SPEC-004-AC-09:** Given a client company has only a Primary contact and no Reviewer, when Nadia replies, then only the Primary contact receives the notification.

**FEAT-07.SPEC-004-AC-10:** Given Owen's email address bounces on the first delivery attempt, when the delivery capability retries, then up to platform parameter: `transactional-email-retry-count` retries occur over platform parameter: `transactional-email-retry-window` before a delivery warning appears on the affected project for Nadia.

**FEAT-07.SPEC-004-AC-11:** Given this notification is successfully delivered to a recipient, when the analytics signal is emitted, then a freelancer_reply_alert_sent event is recorded with that recipient's role and the pin target type.

### User Story 5 - Comment Content & Submission Validation (Priority: P1)

Enforces the single shared rule that a comment's text must be non-empty and within 1--2,000 characters, wherever a comment is submitted across this feature.

**Acceptance Scenarios:**

**FEAT-07.SPEC-005-AC-01:** Given Owen submits a comment of exactly 2,000 characters, when the length rule is checked, then the comment passes validation.

**FEAT-07.SPEC-005-AC-02:** Given Priya submits a comment of exactly 1 character, when the length rule is checked, then the comment passes validation.

**FEAT-07.SPEC-005-AC-03:** Given Nadia submits an empty comment, when the length rule is checked, then she sees "Enter a comment before posting." and no comment is created.

**FEAT-07.SPEC-005-AC-04:** Given Owen submits a comment consisting only of spaces, when the length rule is checked, then it is treated as empty and denied with "Enter a comment before posting."

**FEAT-07.SPEC-005-AC-05:** Given Priya submits a comment of 2,001 characters, when the length rule is checked, then she sees "Your comment can be up to 2,000 characters." and no comment is created.

**FEAT-07.SPEC-005-AC-06:** Given Nadia submits a comment with leading and trailing whitespace around otherwise valid text, when the rule is checked, then the whitespace is trimmed before the length check and the comment is stored trimmed.

**FEAT-07.SPEC-005-AC-07:** Given a comment queued offline via FEAT-07.SPEC-008 was valid at compose time, when the sync attempt re-checks it, then it passes the same rule without re-prompting the user.

**FEAT-07.SPEC-005-AC-08:** Given Owen edits his own comment within the grace window to a new text of 2,001 characters, when he attempts to Save, then the edit is denied with "Your comment can be up to 2,000 characters." and the comment retains its prior text.

**FEAT-07.SPEC-005-AC-09:** Given a comment fails this spec's length rule, when the rejection occurs, then no Comment record is written and neither FEAT-07.SPEC-003 nor FEAT-07.SPEC-004's notification trigger fires.

**FEAT-07.SPEC-005-AC-10:** Given the same 1--2,000 character rule is enforced identically on FEAT-07.SPEC-001, FEAT-07.SPEC-002, and FEAT-07.SPEC-008, when any one of them checks a submission, then the outcome for the same input text is identical across all three.

**FEAT-07.SPEC-005-AC-11:** Given a comment of exactly 2,000 emoji characters, when the length rule is checked, then it passes validation, since each emoji counts as one character toward the limit.

### User Story 6 - Comment Edit Window & Retraction Rule (Priority: P1)

Governs the short post-submit window in which an author may edit their own comment's text, the always-available author-only retraction, and the one-way, non-silent Posted -> Retracted transition.

**Acceptance Scenarios:**

**FEAT-07.SPEC-006-AC-01:** Given Nadia posted a comment moments ago, when she edits its text within platform parameter: `comment-edit-grace-window-minutes`, then the save succeeds and the comment displays the "(edited)" marker.

**FEAT-07.SPEC-006-AC-02:** Given Owen's comment is older than platform parameter: `comment-edit-grace-window-minutes`, when he attempts to edit it, then the attempt is denied with "This comment can no longer be edited." and no Edit control is shown.

**FEAT-07.SPEC-006-AC-03:** Given Priya's comment is exactly at the boundary of platform parameter: `comment-edit-grace-window-minutes` since posting, when she attempts to edit it at that exact instant, then the edit is denied, since the boundary is inclusive of the window's end.

**FEAT-07.SPEC-006-AC-04:** Given Owen's own comment, when he retracts it at any point after posting, regardless of elapsed time, then the retraction succeeds and the comment shows the retracted placeholder.

**FEAT-07.SPEC-006-AC-05:** Given Priya attempts to retract a comment authored by Owen, when she looks for a Retract control on his comment, then none is shown -- retraction is author-only.

**FEAT-07.SPEC-006-AC-06:** Given Nadia's comment has already been retracted, when the thread is viewed again, then the retracted placeholder is shown in its place and no further Edit or Retract control appears on it.

**FEAT-07.SPEC-006-AC-07:** Given a retracted comment has later replies from other participants, when the thread is viewed, then those later replies remain fully visible and unaffected.

**FEAT-07.SPEC-006-AC-08:** Given Owen edits his comment a second time within the still-open grace window, when he saves, then the edit succeeds and the "(edited)" marker continues to show once, not once per edit.

**FEAT-07.SPEC-006-AC-09:** Given Dana (Support Operator) is viewing a thread inside a support session, when she looks for Edit or Retract controls on any comment, then none are shown, since her session is read-only.

**FEAT-07.SPEC-006-AC-10:** Given Nadia has two sessions open on the same comment within its edit window and edits the text differently in each, when both saves are attempted, then the later successful save's text is what persists, and both sessions reflect it on next load.

**FEAT-07.SPEC-006-AC-11:** Given a contact wants to reverse an earlier retraction, when they look for a restore option, then none exists -- they add a new comment with the corrected point instead.

**FEAT-07.SPEC-006-AC-12:** Given Priya attempts to edit her own comment with text that fails FEAT-07.SPEC-005's length rule, when she taps Save within the grace window, then the edit is denied with FEAT-07.SPEC-005's exact error message and the comment retains its prior text.

**FEAT-07.SPEC-006-AC-13:** Given Owen's comment reaches the end of its grace window while he still has the inline edit field open, when he taps Save just after the window closes, then the save is denied with "This comment can no longer be edited." and the field reverts to the last-saved text.

**FEAT-07.SPEC-006-AC-14:** Given a client contact's role changes while their comment is still within its edit window, when they attempt to edit it, then the role change has no effect and the edit proceeds under the same author-and-timing rule.

### User Story 7 - Comment Visibility & Authorization Rule (Priority: P1)

Encodes who sees and can act on which comment thread: Own-only per client company for Owen and Priya, Full for Nadia, View-only inside a logged session for Dana, and strict cross-company isolation.

**Acceptance Scenarios:**

**FEAT-07.SPEC-007-AC-01:** Given Nadia opens any thread belonging to any of her own clients, when the screen loads, then she sees the full thread with Full access.

**FEAT-07.SPEC-007-AC-02:** Given Owen opens a thread belonging to his own client company, when the screen loads, then he sees the full thread and can post.

**FEAT-07.SPEC-007-AC-03:** Given Priya opens a thread belonging to her own client company, when the screen loads, then she sees the full thread and can post, identically to Owen's comment entitlement.

**FEAT-07.SPEC-007-AC-04:** Given Owen attempts to open a thread whose target belongs to a different client company, when the screen would otherwise load, then he sees a plain explanation and a fresh-link option, never that company's comments.

**FEAT-07.SPEC-007-AC-05:** Given Dana opens a thread inside a logged support session for a specific freelancer, when the screen loads, then she sees the full thread read-only, with no composer.

**FEAT-07.SPEC-007-AC-06:** Given Dana has not opened a support session for a given freelancer account, when she attempts to reach that freelancer's comment thread, then no path to it is surfaced to her at all.

**FEAT-07.SPEC-007-AC-07:** Given a contact is a Client Contact for two different freelancers, when they view one freelancer's portal, then no comment data from the other freelancer's portal is ever shown or reachable.

**FEAT-07.SPEC-007-AC-08:** Given a Client Contact's status changes to Removed while mid-session on a thread, when they attempt to post, then the attempt is denied with the same experience as a contact who never had access.

**FEAT-07.SPEC-007-AC-09:** Given Dana's support session closes while she is viewing a thread, when she attempts any further action on it, then no path to the thread exists outside an open session.

**FEAT-07.SPEC-007-AC-10:** Given a comment's status is Retracted, when a viewer entitled to see the original comment views the thread, then they see the retracted placeholder in the same position they would have seen the original text.

**FEAT-07.SPEC-007-AC-11:** Given a Client Contact's role changes from Reviewer to Primary while viewing a thread, when they continue interacting with it, then their comment view and post entitlement is unchanged, since both roles share the same Own-only comment entitlement.

**FEAT-07.SPEC-007-AC-12:** Given the milestone-level thread (FEAT-07.SPEC-002) and the deliverable-level thread (FEAT-07.SPEC-001) both apply this rule, when the same viewer opens either, then the same role-based outcome applies to both, differing only in the target type resolved.

**FEAT-07.SPEC-007-AC-13:** Given a project's archived state changes, when a client contact revisits its comment thread, then their visibility is unaffected by the archived state.

**FEAT-07.SPEC-007-AC-14:** Given two contacts from the same client company both view and post to the same thread, when their posts land close together, then both see each other's comments per the append-only ordering, with no authorization conflict between them.

**FEAT-07.SPEC-007-AC-15:** Given Nadia mistakenly follows a link into a comment thread belonging to one of her own clients from a session scoped to a different one of her clients' contacts, then the request is denied per XBR-09 exactly as for any unrelated out-of-scope contact.

**FEAT-07.SPEC-007-AC-16:** Given Priya (Reviewer, Own-only) attempts to post on a thread for her own client company, when the composer submits, then the write-time authorization check passes and the comment is created.

### User Story 8 - Offline Comment Queue & Sync (Priority: P1)

Holds a comment composed while offline on the device that composed it, then submits it automatically once connectivity returns, re-running the same validation and pin-target rules as an online submission.

**Acceptance Scenarios:**

**FEAT-07.SPEC-008-AC-01:** Given Nadia composes a reply while offline and taps Post, when there is no connectivity, then the comment is queued locally and the screen shows the queued indicator rather than a "posted" confirmation.

**FEAT-07.SPEC-008-AC-02:** Given a comment is queued on Owen's device, when connectivity returns, then it is automatically submitted, validated, and authorized exactly as an online submission would be.

**FEAT-07.SPEC-008-AC-03:** Given a queued comment's sync succeeds, when the Comment record is written, then `posted_at` reflects the sync moment, not the original offline composition time.

**FEAT-07.SPEC-008-AC-04:** Given a queued client-authored comment syncs successfully, when the write completes, then FEAT-07.SPEC-003 (Client Comment Alert to Freelancer) fires.

**FEAT-07.SPEC-008-AC-05:** Given a queued Nadia-authored comment syncs successfully, when the write completes, then FEAT-07.SPEC-004 (Freelancer Reply Alert to Client) fires.

**FEAT-07.SPEC-008-AC-06:** Given Priya has two comments queued on the same device, when connectivity returns, then both sync in the order they were queued, each producing its own separate Comment and notification.

**FEAT-07.SPEC-008-AC-07:** Given a queued comment's author had their contact status changed to Removed while offline, when the sync's authorization re-check runs, then the sync fails with "This comment couldn't be sent because your access has changed." and no Comment is created.

**FEAT-07.SPEC-008-AC-08:** Given a queued comment fails to write due to a transient server error after passing validation and authorization, when the sync attempt fails, then the entry remains queued for automatic retry on the next connectivity event.

**FEAT-07.SPEC-008-AC-09:** Given Owen edits a still-queued comment's text before it syncs, when the sync later runs, then the edited text is what is validated and submitted, not the original text.

**FEAT-07.SPEC-008-AC-10:** Given Nadia discards a queued comment before it syncs, when she confirms the discard, then no Comment record is ever created and no notification ever fires.

**FEAT-07.SPEC-008-AC-11:** Given connectivity is lost again mid-sync of a queued entry, when the write's confirmation is never received, then that entry is retried on the next connectivity event rather than assumed sent.

**FEAT-07.SPEC-008-AC-12:** Given the local queue mechanism itself fails when Priya taps Post while offline, when the entry cannot be queued, then she sees "Couldn't queue this comment for sending. Check your connection and try again." with her typed text preserved in the composer.

### Edge Cases

- **FEAT-07.SPEC-001 (Deliverable Comment Thread):** Simultaneous comments from several contacts are written independently and ordered by posted_at. An edit after the grace window is rejected by FEAT-07.SPEC-006, retracting a comment with later replies leaves the replies visible beside a placeholder, and a failed post preserves the typed text with Retry and no duplicate. Source: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-001-deliverable-comment-thread.md` (section: Edge Cases)
- **FEAT-07.SPEC-002 (Milestone Comment Thread):** The milestone thread behaves like the deliverable thread: concurrent posts are ordered by posted_at, late edits are rejected by FEAT-07.SPEC-006, retraction leaves a placeholder with later replies intact, and a failed post preserves text with Retry. Source: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-002-milestone-comment-thread.md` (section: Edge Cases)
- **FEAT-07.SPEC-003 (Client Comment Alert to Freelancer):** A bounced freelancer address is retried and then surfaces an in-product delivery warning (XBR-30). Each client comment triggers its own separate notification even for near-simultaneous posts, the email still sends while the freelancer views the thread, and a comment retracted before the email is opened still notifies with the placeholder shown in the thread. Source: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-003-client-comment-alert-to-freelancer.md` (section: Edge Cases)
- **FEAT-07.SPEC-004 (Freelancer Reply Alert to Client):** A client contact's bounced address is retried for that recipient only, with a delivery warning for the freelancer. Each entitled contact receives an individually addressed copy, removed contacts are excluded at delivery time, and a company with only a Primary contact simply has one recipient. Source: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-004-freelancer-reply-alert-to-client.md` (section: Edge Cases)
- **FEAT-07.SPEC-005 (Comment Content & Submission Validation):** Comment length boundaries are inclusive (1 and 2,000 characters pass); an empty comment, or one of only spaces, tabs or line breaks, fails with the enter-a-comment message after trimming. Source: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-005-comment-content-submission-validation.md` (section: Edge Cases)
- **FEAT-07.SPEC-006 (Comment Edit Window & Retraction Rule):** The grace window boundary is inclusive of its end, so an edit strictly after the platform-parameter window is denied. Retraction has no time limit and is independent of earlier edits, multiple edits within the window are allowed (each keeping the edited marker), and the author's own simultaneous edits from two devices resolve without cross-actor contention. Source: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-006-comment-edit-window-retraction-rule.md` (section: Edge Cases)
- **FEAT-07.SPEC-007 (Comment Visibility & Authorization Rule):** A contact of two freelancers sees each portal's threads strictly separately, a Reviewer-to-Primary role change has no visibility effect, and a removed contact is re-checked at the next Post attempt rather than only at screen load. An operator's read access ends the instant the support session closes (FEAT-31). Source: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-007-comment-visibility-authorization-rule.md` (section: Edge Cases)
- **FEAT-07.SPEC-008 (Offline Comment Queue & Sync):** A queued comment persists across restarts and is retried when connectivity is next confirmed, a mid-sync connectivity loss retries the same entry, and editing or discarding a still-queued entry is local to the queue (no Comment and no notification until it syncs). Source: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-008-offline-comment-queue-sync.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-07.SPEC-001** (Deliverable Comment Thread) as specified: A contact or Nadia views every comment pinned to one deliverable version in a single chronological thread, posts a new comment, and retracts their own earlier comment -- replacing scattered WhatsApp screenshots with one recorded place per deliverable. Full spec: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-001-deliverable-comment-thread.md`
- **FR-002**: The system MUST implement **FEAT-07.SPEC-002** (Milestone Comment Thread) as specified: A contact or Nadia views and posts general feedback pinned to a milestone as a whole -- for a round that is not about one specific file -- in the same chronological thread pattern as the deliverable-level thread, and retracts their own comment. Full spec: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-002-milestone-comment-thread.md`
- **FR-003**: The system MUST implement **FEAT-07.SPEC-003** (Client Comment Alert to Freelancer) as specified: Emails Nadia the instant a client contact posts a comment on a deliverable or a milestone, so she learns feedback is waiting without checking the portal on a schedule. Full spec: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-003-client-comment-alert-to-freelancer.md`
- **FR-004**: The system MUST implement **FEAT-07.SPEC-004** (Freelancer Reply Alert to Client) as specified: Emails the client contact(s) entitled to a thread the instant Nadia replies in it, so a client waiting on her response is not left checking the portal to find out. Full spec: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-004-freelancer-reply-alert-to-client.md`
- **FR-005**: The system MUST implement **FEAT-07.SPEC-005** (Comment Content & Submission Validation) as specified: Enforces the single shared rule that a comment's text must be non-empty and within 1--2,000 characters, wherever a comment is submitted across this feature. Full spec: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-005-comment-content-submission-validation.md`
- **FR-006**: The system MUST implement **FEAT-07.SPEC-006** (Comment Edit Window & Retraction Rule) as specified: Governs the short post-submit window in which an author may edit their own comment's text, the always-available author-only retraction, and the one-way, non-silent Posted -> Retracted transition. Full spec: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-006-comment-edit-window-retraction-rule.md`
- **FR-007**: The system MUST implement **FEAT-07.SPEC-007** (Comment Visibility & Authorization Rule) as specified: Encodes who sees and can act on which comment thread: Own-only per client company for Owen and Priya, Full for Nadia, View-only inside a logged session for Dana, and strict cross-company isolation. Full spec: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-007-comment-visibility-authorization-rule.md`
- **FR-008**: The system MUST implement **FEAT-07.SPEC-008** (Offline Comment Queue & Sync) as specified: Holds a comment composed while offline on the device that composed it, then submits it automatically once connectivity returns, re-running the same validation and pin-target rules as an online submission. Full spec: `docs/blueprint/specifications/FEAT-07-deliverable-review-feedback/FEAT-07.SPEC-008-offline-comment-queue-sync.md`

### Key Entities

- Comment (create, read)
- Deliverable (read)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Within 3 months of a freelancer's first client onboarding to the portal, at least 80% of that client's feedback on deliverables arrives as in-portal comments rather than through outside channels (metric: Feedback Consolidation). Source: `docs/blueprint/features/success-metrics.md`
- **SC-002**: Comment posts, retractions, comment notifications and milestone comments are each observable as distinct signals (comment_posted, comment_retracted, comment_notification_sent, milestone_comment_posted). Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-07**: Reviewer contacts are assumed to adopt pinned in-portal comments in place of outside channels like WhatsApp. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-23**: Strict data isolation between clients; a client never sees another client's anything. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-27**: Offline behavior never pretends record-creating actions succeeded and keeps typed input on errors. Full register: `docs/blueprint/features/assumptions-constraints.md`
