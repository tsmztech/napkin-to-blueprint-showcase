# FEAT-07 — Deliverable Review & Feedback

This chapter covers Deliverable Review & Feedback, a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 8 specifications carrying 119 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-07.SPEC-001 | Deliverable Comment Thread | screen | 23 |
| FEAT-07.SPEC-002 | Milestone Comment Thread | screen | 22 |
| FEAT-07.SPEC-003 | Client Comment Alert to Freelancer | notification | 10 |
| FEAT-07.SPEC-004 | Freelancer Reply Alert to Client | notification | 11 |
| FEAT-07.SPEC-005 | Comment Content & Submission Validation | logic-rule | 11 |
| FEAT-07.SPEC-006 | Comment Edit Window & Retraction Rule | logic-rule | 14 |
| FEAT-07.SPEC-007 | Comment Visibility & Authorization Rule | logic-rule | 16 |
| FEAT-07.SPEC-008 | Offline Comment Queue & Sync | automation | 12 |

The feature breakdown brief follows, then every specification in full.


# Feature Breakdown Brief: Deliverable Review & Feedback

## Summary

**Feature:** Deliverable Review & Feedback
**ID:** FEAT-07
**Description:** Client contacts leave comments pinned to a specific deliverable; the freelancer sees all feedback in one thread per deliverable, replacing scattered WhatsApp screenshots.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Experience narrative: the client "leaves three comments pinned to the files," and the Problem Statement names WhatsApp screenshots as the exact failure this replaces. MVP phase: needed before a client can meaningfully decide to approve. [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Leave a pinned comment — a comment attached to a specific deliverable
- See a full feedback thread — freelancer sees every comment from every authorized contact in one place
- Retract a comment — a contact can withdraw their own comment
- Comment on a milestone as a whole — leave general feedback on a round that is not about one file [AUDIT-ADDED: 2 -- client portals with messaging are common (HoneyBook, Moxie); a milestone-level thread keeps general feedback off WhatsApp without a separate chat inbox, which scope-boundaries.md excludes]

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-07.SPEC-001 | Deliverable Comment Thread | Screen | Owen, Priya, Nadia, Dana | Contact or freelancer views and posts comments pinned to one deliverable version, and retracts their own |
| FEAT-07.SPEC-002 | Milestone Comment Thread | Screen | Owen, Priya, Nadia, Dana | Contact or freelancer views and posts general feedback on a milestone as a whole, and retracts their own |
| FEAT-07.SPEC-003 | Client Comment Alert to Freelancer | Notification | Nadia | Emails Nadia when a client contact posts a comment on a deliverable or a milestone |
| FEAT-07.SPEC-004 | Freelancer Reply Alert to Client | Notification | Owen, Priya | Emails the relevant client contact(s) when Nadia replies in the thread |
| FEAT-07.SPEC-005 | Comment Content & Submission Validation | Logic/Rule | Owen, Priya, Nadia | Enforces the 1–2,000 character non-empty text rule shared by both thread screens |
| FEAT-07.SPEC-006 | Comment Edit Window & Retraction Rule | Logic/Rule | Owen, Priya, Nadia | Governs the short post-submit edit window, author-only retraction, and the soft-removal (never silent rewrite) behavior |
| FEAT-07.SPEC-007 | Comment Visibility & Authorization Rule | Logic/Rule | All | Encodes who sees and can act on which thread: Own-only per client company for Owen/Priya, Full for Nadia, View-only inside a logged session for Dana, and cross-company isolation |
| FEAT-07.SPEC-008 | Offline Comment Queue & Sync | Automation | Owen, Priya, Nadia | Holds a comment composed while offline and submits it automatically once connectivity returns, applying the same validation and pin-target rules |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Leave a pinned comment | FEAT-07.SPEC-001 | Composer on the Deliverable Comment Thread screen; text validated by SPEC-005, target pinned to the deliverable version | Phase 2 (Explicit) |
| See a full feedback thread | FEAT-07.SPEC-001, FEAT-07.SPEC-002 | Both thread screens render every comment visible to the viewer per SPEC-007's visibility rule, ordered by post time | Phase 2 (Explicit) |
| Retract a comment | FEAT-07.SPEC-001, FEAT-07.SPEC-002 | Retract control on the author's own comment in either thread screen, governed by SPEC-006 | Phase 2 (Explicit) |
| Comment on a milestone as a whole | FEAT-07.SPEC-002 | Milestone Comment Thread screen — same composer and thread pattern, pinned to the milestone rather than a deliverable version | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-07.SPEC-003 | Client Comment Alert to Freelancer | Phase 4 (Notification surfacing) | The Communications field names "email to Nadia when a client comments"; this has a channel, an audience, and delivery behavior, so it needs a standalone Notification spec rather than an inline toast |
| FEAT-07.SPEC-004 | Freelancer Reply Alert to Client | Phase 4 (Notification surfacing) | The Communications field names "email to the client contact when Nadia replies" — same standalone-Notification disposition |
| FEAT-07.SPEC-005 | Comment Content & Submission Validation | Phase 5 (Rule Discovery) | The length/non-empty rule is shared across both thread screens (SPEC-001 and SPEC-002) and their offline queue path (SPEC-008), crossing the "shared across multiple screens" threshold for a standalone Logic/Rule spec |
| FEAT-07.SPEC-006 | Comment Edit Window & Retraction Rule | Phase 5 (Rule Discovery) / Phase 3 (CRUD — Update, Delete/Archive) | Conditional logic (time-window-gated edit vs. always-available retraction) plus the Comment entity's Update and Delete/Archive CRUD cells require an explicit lifecycle rule, not an inline note |
| FEAT-07.SPEC-007 | Comment Visibility & Authorization Rule | Phase 5 (Rule Discovery — Authorization) | The Access field differentiates behavior by four roles (Own-only, Full, View-only, none) and enforces cross-company isolation (XBR-09) — authorization complexity crosses the standalone-spec threshold |
| FEAT-07.SPEC-008 | Offline Comment Queue & Sync | Phase 6 (Negative/Failure — Offline/Degraded) | The States field's offline-degraded expectation ("held locally and sent once connectivity returns") involves queuing and retry processing, not a single-step consequence, so it is a standalone Automation rather than an inline screen note |

## Entity-Lifecycle Coverage Matrix

**Entity: Comment**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-07.SPEC-001, FEAT-07.SPEC-002 | Contact or Nadia submits comment text pinned to a deliverable version or a milestone; validated by SPEC-005 | Also created via the offline queue path (SPEC-008) once connectivity returns |
| Read (single) | N/A | A single comment is never viewed in isolation — it is always shown within its thread | Folded into Read (list) |
| Read (list) | FEAT-07.SPEC-001, FEAT-07.SPEC-002 | Full thread rendered per pin target, ordered by post time, scoped by SPEC-007's visibility rule | -- |
| Update | FEAT-07.SPEC-006 | Author may edit their own comment's text only within a short grace window after posting; no edits accepted after the window closes | Implemented as an edit control on SPEC-001/SPEC-002, governed by this rule |
| Delete/Archive | FEAT-07.SPEC-006 | Soft removal only: the author retracts their own comment, status changes Posted -> Retracted (never a silent rewrite or hard delete); no restore path back to Posted (a corrected point re-enters the thread as a new comment); no cascade to replies (a retracted comment's replies remain visible); retained indefinitely as part of the record until removed by an account-level export/deletion request (FEAT-24) — no independent purge policy, recorded as an explicit non-goal below | Hard deletion is FEAT-24's responsibility only |
| State Transition | FEAT-07.SPEC-006 | Posted -> Retracted is the only transition; it is one-way and author-triggered | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Deliverable | FEAT-07.SPEC-001 | Confirms the deliverable a comment thread belongs to and its current status |
| Deliverable Version | FEAT-07.SPEC-001 | The pin target for a deliverable-level comment; comments stay attached to the version they were made on (XBR-13) |
| Milestone | FEAT-07.SPEC-002 | The pin target for a milestone-level comment; also read by FEAT-08's approval screen alongside its comment thread |
| Client Contact | FEAT-07.SPEC-001, FEAT-07.SPEC-002, FEAT-07.SPEC-007 | Identifies the author of each comment and grounds the visibility/authorization rule's Own-only scoping |
| Notification | FEAT-07.SPEC-003, FEAT-07.SPEC-004 | These specs trigger Notification records; the Notification entity itself is created and delivered by FEAT-14 |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Client contact posts a comment (deliverable or milestone) | Email Nadia that new feedback is waiting, linking to the thread | Standalone Notification | FEAT-07.SPEC-003 |
| Nadia replies in a thread | Email the client contact(s) entitled to that thread that the freelancer replied | Standalone Notification | FEAT-07.SPEC-004 |
| Any user submits a comment | Validate text is non-empty and within 1–2,000 characters before accepting | Standalone Logic/Rule | FEAT-07.SPEC-005 |
| Author attempts to edit or retract their comment | Apply the grace-window edit rule or the always-available retraction rule; reject an edit outside the window | Standalone Logic/Rule | FEAT-07.SPEC-006 |
| Any user opens or posts to a thread | Enforce role-scoped visibility (Own-only per client company, Full for Nadia, View-only and logged for Dana) and cross-company isolation | Standalone Logic/Rule | FEAT-07.SPEC-007 |
| User composes a comment while offline | Hold the comment locally and submit it automatically once connectivity returns, re-running validation and pin-target resolution | Standalone Automation | FEAT-07.SPEC-008 |
| Comment submitted successfully | Clear the composer, show the new comment in the thread immediately (no separate confirmation screen) | Inline in triggering screen | FEAT-07.SPEC-001 / SPEC-002 |
| Comment submission fails server-side | Preserve the typed text in the composer and offer retry | Inline in triggering screen | FEAT-07.SPEC-001 / SPEC-002 |
| Comment retracted | Thread shows the retracted placeholder in its original position; replies remain visible | Inline in triggering screen | FEAT-07.SPEC-001 / SPEC-002 |
| Priya attempts an Approve action she does not have | Show her role's actual scope (comment only), never a broken control, without blocking her ability to leave feedback | Cross-feature -- FEAT-08 owns the Approve control; this feature's screens simply never render it for a comment-only role, per SPEC-007 | FEAT-08 responsibility |
| Owen reviews a milestone before approving | Existing comment thread (deliverable and milestone level) is shown as context on the approval screen | Cross-feature -- logged in touchpoints | FEAT-08 responsibility |

## Shared Context

**Shared Entities:**
- Comment -- created by SPEC-001, SPEC-002, and SPEC-008 (offline path); read/listed by SPEC-001 and SPEC-002; validated by SPEC-005; edited/retracted per SPEC-006; visibility scoped by SPEC-007. Fields: text (1–2,000 chars), author, posted_at, target (deliverable version or milestone), reply_to, status (Posted | Retracted).

**Shared UI Patterns:**
- Comment thread + composer -- shared by SPEC-001 (deliverable-pinned) and SPEC-002 (milestone-pinned): a chronological thread, a composer at the point of entry, a retract control shown only on the viewer's own comments, and the "no feedback yet" empty state when the thread is empty. Spec Writers for both screens should describe this pattern identically, differing only in what the thread is pinned to.
- Loading/error/offline banners -- both thread screens share the same in-progress indicator on slow connections, retry-with-preserved-text on submission failure, and an offline composing banner tied to SPEC-008.

**Shared Validation:**
- FEAT-07.SPEC-005 defines the text validation rule. SPEC-001, SPEC-002, and SPEC-008 all reference it rather than duplicating the length/non-empty check.
- FEAT-07.SPEC-007 defines the visibility/authorization rule. SPEC-001 and SPEC-002 both reference it to determine what a given viewer sees and may do, rather than each screen re-deriving role scope.

## Internal Dependency Map

```
SPEC-001 (Deliverable Comment Thread) -> [user submits comment] -> SPEC-005 (Content & Submission Validation) -> [valid] -> [comment created] -> SPEC-003 (Client Comment Alert to Freelancer) [author is Owen/Priya]
SPEC-001 (Deliverable Comment Thread) -> [Nadia submits reply] -> SPEC-005 (Content & Submission Validation) -> [valid] -> [comment created] -> SPEC-004 (Freelancer Reply Alert to Client)
SPEC-002 (Milestone Comment Thread) -> [user submits comment] -> SPEC-005 (Content & Submission Validation) -> [valid] -> [comment created] -> SPEC-003 (Client Comment Alert to Freelancer) [author is Owen/Priya]
SPEC-002 (Milestone Comment Thread) -> [Nadia submits reply] -> SPEC-005 (Content & Submission Validation) -> [valid] -> [comment created] -> SPEC-004 (Freelancer Reply Alert to Client)
SPEC-001 (Deliverable Comment Thread) -> [author retracts or edits own comment] -> SPEC-006 (Edit Window & Retraction Rule)
SPEC-002 (Milestone Comment Thread) -> [author retracts or edits own comment] -> SPEC-006 (Edit Window & Retraction Rule)
SPEC-001 (Deliverable Comment Thread) -> [loads or renders thread] -> SPEC-007 (Visibility & Authorization Rule)
SPEC-002 (Milestone Comment Thread) -> [loads or renders thread] -> SPEC-007 (Visibility & Authorization Rule)
SPEC-001 (Deliverable Comment Thread) -> [user composes while offline] -> SPEC-008 (Offline Comment Queue & Sync) -> [connectivity returns] -> SPEC-001 (queued comment submitted) -> SPEC-005
SPEC-002 (Milestone Comment Thread) -> [user composes while offline] -> SPEC-008 (Offline Comment Queue & Sync) -> [connectivity returns] -> SPEC-002 (queued comment submitted) -> SPEC-005
```

**Default Entry:** SPEC-001 (Deliverable Comment Thread) -- reached when a contact opens a deliverable waiting for review from the portal home (FEAT-05); SPEC-002 is reached from the milestone view when the feedback is not about one file.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-07.SPEC-001 | Inbound | FEAT-05 (Client Portal Access, Magic-Link Login) | Contact arrives at the deliverable review and comment thread | Priya or Owen opens a deliverable waiting for review |
| FEAT-07.SPEC-001 | Inbound | FEAT-06 (Deliverable Upload & Sharing) | Deliverable versions created or replaced by FEAT-06 become the pin target for new comments | Deliverable uploaded or re-uploaded |
| FEAT-07.SPEC-001 | Outbound | FEAT-17 (Deliverable Versioning) | Viewer opens an earlier round via the version selector from the comment thread | A viewer opens an earlier version of the deliverable |
| FEAT-07.SPEC-001, FEAT-07.SPEC-002 | Outbound | FEAT-08 (Milestone Approval) | The existing comment thread (deliverable and milestone level) is shown as approval context | Owen reviews the round and chooses Approve |
| FEAT-07.SPEC-001, FEAT-07.SPEC-002 | Inbound | FEAT-31 (Support Access) | Dana views the same thread read-only, inside a logged, read-only support session | Dana opens a support session on the freelancer's account |
| FEAT-07.SPEC-003, FEAT-07.SPEC-004 | Outbound | FEAT-14 (Notifications, Email) | Both Notification specs rely on FEAT-14's transactional email delivery capability and its owning Integration spec for send/delivery/bounce status | Comment posted (client) or reply posted (Nadia) |

## Non-Functional Notes

**Data volumes / growth:** A deliverable or milestone thread grows append-only as multiple contacts from the same client company comment over the life of a project; there is no fixed cap on thread length, and threads must stay equally responsive as a project accumulates rounds of feedback (assumptions-constraints.md, ASMP-21).

**Responsiveness:** Deliverable review pages, including the comment thread, become interactive within roughly 2 seconds on a typical mobile connection (ASMP-21); the thread shows a lightweight in-progress indicator while loading on slow connections, per this feature's States field.

**Data sensitivity / privacy:** Comment text and author identity are personal data and GDPR-class (ASMP-24); comments are strictly isolated per client company (ASMP-23) and never visible to another client company's contacts; comment data is included in the freelancer's data export and removed on account deletion (dependency map, Comment entity Data Sensitivity).

**Compliance flags:** GDPR-class handling applies to all comment content and author identity (ASMP-24); no card or payment data is ever involved in this feature. Accessibility and degraded-state conventions apply per ASMP-27: the thread is readable on a phone without zooming, usable with a screen reader and keyboard, preserves typed text on a failed submission, and states plainly when a comment is queued for send rather than pretending it posted while offline.

## Non-Goals

- **A general-purpose chat or messaging inbox** -- Excluded per scope-boundaries.md (SC-15): the brief's goal is feedback that is pinned and recorded, not another chat channel; deliverable-pinned and milestone-level comments (this feature) cover the need without a separate messaging surface.
- **Client-side roles beyond Primary and Reviewer** -- Excluded per scope-boundaries.md (SC-02): no additional comment-permission tier (e.g., a client-side "admin" with broader comment rights) is modeled without a brief signal for one; SPEC-007's Own-only/Full/View-only split is exhaustive of the roles the product defines.
- **Automatic purge or hard deletion of retracted comments** -- Intentional lifecycle decision surfaced by the Entity-Lifecycle Coverage Matrix (Comment, Delete/Archive): retraction is a soft removal only; comments are retained indefinitely as evidentiary record of the feedback history until an account-level export/deletion request is honored by FEAT-24 (dependency map, Comment entity Data Sensitivity: "included in the freelancer's export and removed on account deletion"). No independent purge window applies within this feature.
- **Restoring a retracted comment to Posted** -- Intentional lifecycle decision: the Comment entity's status transition (SPEC-006) is one-way; a contact who retracted in error re-adds their point as a new comment rather than reversing the retraction, preserving the "never silently rewritten" guarantee stated in the feature's Primary Flows & Alternates.



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



# Notification Spec: Client Comment Alert to Freelancer

## Overview

**Name:** Client Comment Alert to Freelancer
**ID:** FEAT-07.SPEC-003
**Type:** Notification
**Purpose:** Emails Nadia the instant a client contact posts a comment on a deliverable or a milestone, so she learns feedback is waiting without checking the portal on a schedule.
**Parent Feature:** FEAT-07 -- Deliverable Review & Feedback

## Scope and Non-Goals

**In Scope:**
- The email sent to Nadia the instant Owen or Priya posts a comment, on either a Deliverable Version (FEAT-07.SPEC-001) or a Milestone (FEAT-07.SPEC-002)
- Content, delivery rules, and edge cases for this single notification, across both pin targets

**Non-Goals:**
- Notifying Nadia when she herself posts a reply -- this notification exists only for client-originated comments; Nadia's own activity never triggers it.
- An email to the client confirming their comment was received -- the Feature Breakdown Brief's Side-Effect Inventory defines the confirmation as inline, on-screen feedback in the triggering screen (FEAT-07.SPEC-001 / FEAT-07.SPEC-002), not a standalone notification.
- An in-app notification channel -- product-features.md phases the In-App Notification Center (FEAT-29) as Later, outside MVP; email is the product's sole MVP channel (ASMP-29).
- A preference to turn this notification off -- excluded per XBR-30: this is Nadia's signal that a client is waiting on her, core to the product's replacement for WhatsApp screenshots; it is not an optional marketing-style email.
- Notifying Nadia about an edit to an existing comment or a retraction -- the Feature Breakdown Brief's Communications field names only "email to Nadia when a client comments," not edits or retractions; a retraction or edit is visible the next time Nadia opens the thread, with no separate alert.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to Nadia, the instant a client contact's comment is recorded | Nadia's Behavioral Context (user-persona.md) has her opening Clientroom to check whether a client has responded, not watching a live feed; email is the product's sole notification channel at MVP (ASMP-29) and delivers the alert into the same inbox she already monitors for client business. |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client contact posts a comment (deliverable) | FEAT-07.SPEC-001 (Deliverable Comment Thread) | Fires immediately once the Comment write succeeds and the author is Owen or Priya (never Nadia) | Deliverable and project reference, client company name, author identity and role, comment text, `posted_at` |
| Client contact posts a comment (milestone) | FEAT-07.SPEC-002 (Milestone Comment Thread) | Fires immediately once the Comment write succeeds and the author is Owen or Priya | Milestone and project reference, client company name, author identity and role, comment text, `posted_at` |
| Client contact's queued offline comment syncs successfully | FEAT-07.SPEC-008 (Offline Comment Queue & Sync) | Fires immediately once a queued client comment's sync completes and the Comment write succeeds | Same data as the corresponding online trigger above, for whichever target (deliverable or milestone) the queued comment was pinned to |

## Audience and Preferences

**Recipients:** Nadia (Freelancer) -- the sole recipient, per the Access Matrix (Milestones & Deliverables: Full for Nadia) and XBR-08 (notification recipients are limited to contacts entitled to the event) and the Feature Breakdown Brief's Communications field ("Email to Nadia when a client comments"). Neither Owen nor Priya ever receives this notification -- it exists to reach Nadia specifically.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| N/A | N/A | Always on -- transactional | N/A -- per XBR-30, this is a transactional email core to the product's record-and-respond loop and cannot be disabled |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for this notification; a client comment is time-sensitive to Nadia's own response, and holding it would delay her seeing that a client is waiting, contradicting the Feature Breakdown Brief's "emails Nadia" immediacy.

## Content Definition

**Email (deliverable-pinned comment):**
- **Subject:** New feedback from {client_company_name} on {deliverable_name}
- **Body:**
  Hi {nadia_first_name},

  {author_full_name} ({author_role}) at {client_company_name} left a comment on {deliverable_name} for {project_name}:

  "{comment_text}"

  Open the thread to see the full conversation and reply.
- **CTA (button):** Open thread -- deep-links to FEAT-07.SPEC-001 (Deliverable Comment Thread) for the specific Deliverable Version the comment was posted against

**Email (milestone-pinned comment):**
- **Subject:** New feedback from {client_company_name} on {milestone_name}
- **Body:**
  Hi {nadia_first_name},

  {author_full_name} ({author_role}) at {client_company_name} left a comment on the milestone "{milestone_name}" for {project_name}:

  "{comment_text}"

  Open the thread to see the full conversation and reply.
- **CTA (button):** Open thread -- deep-links to FEAT-07.SPEC-002 (Milestone Comment Thread) for the specific Milestone the comment was posted against

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|-----------------|--------------------------|
| {nadia_first_name} | Freelancer Account -- name (first token) | Nadia | Greeting renders as "Hi," |
| {author_full_name} | Client Contact -- name | Owen Carter | Never empty -- required at contact creation (FEAT-18) |
| {author_role} | Client Contact -- role | Primary Contact / Reviewer | Never empty -- role is required and always one of the two values |
| {client_company_name} | Client -- client_name | Carter & Co | Never empty -- required at client creation (FEAT-01) |
| {deliverable_name} | Deliverable -- derived display label (no dedicated name field exists on the entity; see FEAT-07.SPEC-001's Data Model note): the uploaded file's original file name (from `file or link`), or, for a linked asset, the source platform name plus "link" (e.g., "Figma link") | Homepage mockups | Never empty -- a file always carries an original file name at upload, and a link always resolves to a source platform name (FEAT-06) |
| {milestone_name} | Milestone -- name | Round 2 revisions | Never empty -- required at milestone creation (FEAT-04) |
| {project_name} | Project -- project_name | Brand Refresh Q1 | Never empty -- required at project creation (FEAT-01) |
| {comment_text} | Comment -- text (1--2,000 characters) | "Could we see this in the darker blue from the last round?" | Never empty -- FEAT-07.SPEC-005 rejects an empty comment before this notification's trigger can fire |

## Delivery Rules

**Batching:** None -- each client comment is a single, discrete moment Nadia needs to know about promptly; it is never combined with other comments, even if several arrive close together on the same or different threads.
**Deduplication:** At most one notification per recorded Comment. FEAT-07.SPEC-001, FEAT-07.SPEC-002, and FEAT-07.SPEC-008 each fire this notification's trigger exactly once per successful client-authored Comment write; a submission rejected for invalid length never reaches the trigger.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the comment itself remains visible to Nadia the next time she opens the thread, regardless of this email's delivery outcome.
**Expiry:** This notification is never withdrawn once triggered -- delivery is retried per the rule above, and the underlying Comment record persists indefinitely as the feedback record even if every retry fails.

## Edge Cases

- **Nadia's email address bounces** -- The failure is retried per the Retry on failure rule; after the final failure, a delivery warning still appears on the affected project inside the product the next time she opens it (XBR-30), and the comment remains visible there regardless.
- **Owen and Priya each post a comment on the same deliverable within moments of each other** -- Each comment produces its own Comment record (dependency map, Comment Contention: append-only, ordered by posted time) and its own separate notification instance; the two are never merged into one email.
- **A client contact posts a comment while Nadia is already viewing that same thread** -- The notification still sends; this feature defines no live-updating suppression rule, since the email is the evidentiary trail of the moment the comment arrived, not merely a live-view convenience.
- **The client comment is retracted by its author moments after posting, before Nadia opens the email** -- The notification still sends and its CTA still opens the thread; the thread now shows the retracted placeholder in that comment's place, consistent with retraction being a soft, visible change rather than a silent disappearance (FEAT-07.SPEC-006).
- **A comment queued offline (FEAT-07.SPEC-008) syncs successfully hours after it was composed** -- This notification fires at the moment the sync succeeds and the Comment write completes, not at the moment the client originally composed it offline, so Nadia's alert reflects when the feedback actually entered the record.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|----------------|
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | Triggered by (inbound); Navigation (outbound) | A client's post on this screen fires this notification; the deliverable-comment CTA deep-links back here |
| FEAT-07.SPEC-002 (Milestone Comment Thread) | Triggered by (inbound); Navigation (outbound) | A client's post on this screen fires this notification; the milestone-comment CTA deep-links back here |
| FEAT-07.SPEC-008 (Offline Comment Queue & Sync) | Triggered by (inbound) | A successful offline sync of a client-authored comment fires this notification |
| FEAT-07.SPEC-005 (Comment Content & Submission Validation) | References (inbound) | Only a comment that passes validation reaches this notification's trigger |
| FEAT-14 (Notifications (Email), FEAT-14.SPEC-001) | References (outbound) | Owns the transactional email delivery capability and delivery/bounce status reporting this notification relies on |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|---------------|-------------------|
| client_comment_alert_sent | recipient: nadia; pin target type: deliverable / milestone; author role | Delivery succeeds | supports success-metrics.md: "Feedback Consolidation" |
| client_comment_alert_delivery_failed | retries exhausted: yes / no | All retries are exhausted without a successful delivery | N/A -- no Stage 2 metric measures delivery failures directly; retained so a silently undelivered alert is observable via the project's delivery warning (XBR-30) rather than invisible |

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 3 (deliverable post, milestone post, offline sync) | 3 |
| Preference States | 1 (always on -- transactional) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |



# Notification Spec: Freelancer Reply Alert to Client

## Overview

**Name:** Freelancer Reply Alert to Client
**ID:** FEAT-07.SPEC-004
**Type:** Notification
**Purpose:** Emails the client contact(s) entitled to a thread the instant Nadia replies in it, so a client waiting on her response is not left checking the portal to find out.
**Parent Feature:** FEAT-07 -- Deliverable Review & Feedback

## Scope and Non-Goals

**In Scope:**
- The email sent to the entitled client contact(s) the instant Nadia posts a comment, on either a Deliverable Version (FEAT-07.SPEC-001) or a Milestone (FEAT-07.SPEC-002)
- Content, delivery rules, and edge cases for this single notification, across both pin targets and both client roles (Owen and Priya)

**Non-Goals:**
- Notifying Nadia when a client comments -- that is the sibling notification FEAT-07.SPEC-003 (Client Comment Alert to Freelancer); this spec only covers Nadia's outbound replies.
- An in-app notification channel -- consistent with FEAT-07.SPEC-003, email is the product's sole MVP channel (ASMP-29); the In-App Notification Center (FEAT-29) is Later.
- A preference to turn this notification off -- excluded per XBR-30: this is the client's signal that the person they are waiting on has responded, core to the product's promise of removing the "email back-and-forth" (BRIEF.md, Problem Statement); it is not an optional marketing-style email.
- Notifying a client contact when another client contact from the same company replies -- the Feature Breakdown Brief's Communications field names only "email to the client contact when Nadia replies," not peer-to-peer client notifications; Owen and Priya each see every comment the next time they open the thread, with no separate alert for a peer's post.
- Notifying about an edit to an existing reply or a retraction -- consistent with FEAT-07.SPEC-003, only the original post fires this notification.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to the thread's entitled client contact(s), the instant Nadia's comment is recorded | Owen and Priya's Behavioral Context (user-persona.md) has each of them opening the portal from an emailed link, on their phone, in short sessions triggered by a specific event -- not habitual browsing. Email is the trigger that brings them back to see Nadia's reply, and it is the product's sole notification channel at MVP (ASMP-29). |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia replies in a thread (deliverable) | FEAT-07.SPEC-001 (Deliverable Comment Thread) | Fires immediately once the Comment write succeeds and the author is Nadia | Deliverable and project reference, client company, Nadia's identity, comment text, `posted_at` |
| Nadia replies in a thread (milestone) | FEAT-07.SPEC-002 (Milestone Comment Thread) | Fires immediately once the Comment write succeeds and the author is Nadia | Milestone and project reference, client company, Nadia's identity, comment text, `posted_at` |
| Nadia's queued offline reply syncs successfully | FEAT-07.SPEC-008 (Offline Comment Queue & Sync) | Fires immediately once a queued reply's sync completes and the Comment write succeeds | Same data as the corresponding online trigger, for whichever target the queued reply was pinned to |

## Audience and Preferences

**Recipients:** Every Client Contact (Owen and, where applicable, Priya) entitled to the thread's client company, per the Access Matrix (Milestones & Deliverables: Own-only, view and comment, for both roles) and XBR-08 (notification recipients are limited to contacts entitled to the event). Both Primary and Reviewer contacts at the same client company receive this notification, since both are entitled to view and comment on the same deliverable and milestone threads (Access Matrix: Own-only, view and comment, for both Owen and Priya). Nadia is never a recipient of her own reply's notification.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| N/A | N/A | Always on -- transactional | N/A -- per XBR-30, this is a transactional email core to the product's record-and-respond loop and cannot be disabled |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for this notification; a freelancer's reply is time-sensitive to the client's own next step (reviewing feedback further, or moving toward approval), and holding it would delay the client from seeing it.

## Content Definition

**Email (deliverable-pinned reply):**
- **Subject:** {nadia_business_name} replied on {deliverable_name}
- **Body:**
  Hi {client_first_name},

  {nadia_business_name} replied on {deliverable_name} for {project_name}:

  "{comment_text}"

  Open the thread to see the full conversation.
- **CTA (button):** Open thread -- deep-links to FEAT-07.SPEC-001 (Deliverable Comment Thread) for the specific Deliverable Version the reply was posted against

**Email (milestone-pinned reply):**
- **Subject:** {nadia_business_name} replied on {milestone_name}
- **Body:**
  Hi {client_first_name},

  {nadia_business_name} replied on the milestone "{milestone_name}" for {project_name}:

  "{comment_text}"

  Open the thread to see the full conversation.
- **CTA (button):** Open thread -- deep-links to FEAT-07.SPEC-002 (Milestone Comment Thread) for the specific Milestone the reply was posted against

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|-----------------|--------------------------|
| {client_first_name} | Client Contact -- name (first token) | Owen | Greeting renders as "Hi," |
| {nadia_business_name} | Branding Profile -- business name, falling back to Freelancer Account -- name | Studio Nadia | Renders as "Your freelancer" if neither is set, though a business or personal name is always present by the time a project is active (FEAT-01, FEAT-19) |
| {deliverable_name} | Deliverable -- derived display label (no dedicated name field exists on the entity; see FEAT-07.SPEC-001's Data Model note): the uploaded file's original file name (from `file or link`), or, for a linked asset, the source platform name plus "link" (e.g., "Figma link") | Homepage mockups | Never empty -- a file always carries an original file name at upload, and a link always resolves to a source platform name (FEAT-06) |
| {milestone_name} | Milestone -- name | Round 2 revisions | Never empty -- required at milestone creation (FEAT-04) |
| {project_name} | Project -- project_name | Brand Refresh Q1 | Never empty -- required at project creation (FEAT-01) |
| {comment_text} | Comment -- text (1--2,000 characters) | "Good catch -- I've swapped in the darker blue for this round." | Never empty -- FEAT-07.SPEC-005 rejects an empty comment before this notification's trigger can fire |

## Delivery Rules

**Batching:** None -- each reply is a single, discrete moment the client needs to know about promptly; it is never combined with other replies, even if Nadia replies to several threads close together.
**Deduplication:** At most one notification per recorded Comment, per recipient. FEAT-07.SPEC-001, FEAT-07.SPEC-002, and FEAT-07.SPEC-008 each fire this notification's trigger exactly once per successful Nadia-authored Comment write; a submission rejected for invalid length never reaches the trigger.
**Retry on failure:** Per recipient, delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure for a given recipient, the failure is surfaced to Nadia as a delivery warning on the affected project (XBR-30); the reply itself remains visible to that client contact the next time they open the thread, regardless of this email's delivery outcome for them.
**Expiry:** This notification is never withdrawn once triggered -- delivery is retried per the rule above, and the underlying Comment record persists indefinitely even if every retry to a given recipient fails.

## Edge Cases

- **A client contact's email address bounces** -- The failure is retried per the Retry on failure rule for that recipient only; after the final failure, a delivery warning appears on the affected project for Nadia, and the reply remains visible to that contact inside the portal regardless.
- **Both Owen and Priya are entitled to the same thread** -- Each receives their own copy of the notification, addressed individually, since both roles can view and comment on the same deliverable and milestone threads (Access Matrix); this is not treated as a single shared send.
- **A client contact has been removed from the company (FEAT-18) between Nadia's reply and delivery** -- The notification is not sent to a contact whose status is no longer Active at delivery time; recipients are resolved against current entitlement, not entitlement at the moment Nadia's reply was recorded.
- **Nadia replies to a thread where the client company has only a Primary contact (no Reviewer)** -- Only Owen receives the notification; there is no requirement for a second recipient, since entitlement -- not a fixed recipient count -- determines the audience.
- **Nadia's reply, composed offline, syncs successfully hours after she composed it (FEAT-07.SPEC-008)** -- This notification fires at the moment the sync succeeds and the Comment write completes, not at Nadia's original offline composition time.
- **Nadia replies to a comment that its author has since retracted** -- The reply notification still sends normally; a retraction of an earlier comment in the same thread does not affect the delivery of a later reply's own notification.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|----------------|
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | Triggered by (inbound); Navigation (outbound) | Nadia's post on this screen fires this notification; the deliverable-reply CTA deep-links back here |
| FEAT-07.SPEC-002 (Milestone Comment Thread) | Triggered by (inbound); Navigation (outbound) | Nadia's post on this screen fires this notification; the milestone-reply CTA deep-links back here |
| FEAT-07.SPEC-008 (Offline Comment Queue & Sync) | Triggered by (inbound) | A successful offline sync of a Nadia-authored reply fires this notification |
| FEAT-07.SPEC-005 (Comment Content & Submission Validation) | References (inbound) | Only a comment that passes validation reaches this notification's trigger |
| FEAT-07.SPEC-007 (Comment Visibility & Authorization Rule) | References (inbound) | Determines which client contacts are entitled recipients for a given thread |
| FEAT-14 (Notifications (Email), FEAT-14.SPEC-001) | References (outbound) | Owns the transactional email delivery capability and delivery/bounce status reporting this notification relies on |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|---------------|-------------------|
| freelancer_reply_alert_sent | recipient role: primary / reviewer; pin target type: deliverable / milestone | Delivery succeeds, per recipient | supports success-metrics.md: "Feedback Consolidation" |
| freelancer_reply_alert_delivery_failed | retries exhausted: yes / no | All retries for a given recipient are exhausted without a successful delivery | N/A -- no Stage 2 metric measures delivery failures directly; retained so a silently undelivered alert is observable via the project's delivery warning (XBR-30) rather than invisible |

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 3 (deliverable reply, milestone reply, offline sync) | 3 |
| Preference States | 1 (always on -- transactional) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Comment Content & Submission Validation

## Overview

**Name:** Comment Content & Submission Validation
**ID:** FEAT-07.SPEC-005
**Type:** Logic/Rule
**Purpose:** Enforces the single shared rule that a comment's text must be non-empty and within 1--2,000 characters, wherever a comment is submitted across this feature.
**Parent Feature:** FEAT-07 -- Deliverable Review & Feedback
**Governed Entity:** Comment (the `text` field, as it applies to submission on create and on edit)

## Scope and Non-Goals

**In Scope:**
- The non-empty, 1--2,000 character rule for the Comment's `text` field, on both creation and edit
- Whitespace and character-counting treatment for that rule
- Pointing to the sibling specs that own every other aspect of the Comment entity, so the full field inventory is accounted for from this spec's vantage point

**Non-Goals:**
- Who may post, view, edit, or retract a comment -- owned entirely by FEAT-07.SPEC-007 (Comment Visibility & Authorization Rule); this spec governs only whether submitted text is well-formed, never whether the submitter is entitled to submit it.
- The edit grace window and retraction (status transition) rules -- owned entirely by FEAT-07.SPEC-006 (Comment Edit Window & Retraction Rule).
- Resolving which Deliverable Version or Milestone a comment pins to -- the Feature Breakdown Brief's Primary Flows treat pin-target assignment as a simple, single-step consequence of which screen the composer is on (FEAT-07.SPEC-001 or FEAT-07.SPEC-002); it does not cross the threshold for a standalone Logic/Rule concern and stays inline in those screens.
- Detecting or filtering inappropriate content -- scope-boundaries.md defines no content-moderation capability for this product; a client-facing feedback thread between a freelancer and her own clients has no stated moderation need in the product definition.

## Governed Entity

**Entity:** Comment
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| text | text (1--2,000 characters) | The comment's content -- the field this spec governs |
| author | reference (Client Contact or Freelancer Account) | Who wrote the comment -- governed elsewhere (see table below) |
| posted_at | date/time | When the comment was recorded -- governed elsewhere |
| target | reference (Deliverable Version or Milestone) | What the comment is pinned to -- governed elsewhere |
| reply_to | reference (Comment), optional | The comment being replied to, if any -- governed elsewhere |
| status | enum (Posted, Retracted) | The comment's lifecycle state -- governed elsewhere |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-07.SPEC-001 | Deliverable Comment Thread | On Post tap (create) and on Save tap (inline edit) |
| FEAT-07.SPEC-002 | Milestone Comment Thread | On Post tap (create) and on Save tap (inline edit) |
| FEAT-07.SPEC-008 | Offline Comment Queue & Sync | Authoritative re-check at the moment a queued comment's sync is attempted, before the write |
| FEAT-07.SPEC-006 | Comment Edit Window & Retraction Rule | Re-applies this rule to the edited text before writing an in-window edit |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|-------------------------------|---------------|---------------------|-----------|
| text | Required, non-empty (whitespace-only input is treated as empty), 1--2,000 characters after leading/trailing whitespace is trimmed | Always, on both create and edit | On Post/Save tap (screen-level) and on write (FEAT-07.SPEC-008's sync attempt, authoritative for the offline path) | "Enter a comment before posting." (empty or whitespace-only) / "Your comment can be up to 2,000 characters." (over length) | Yes |
| author | No validation beyond data type -- system-derived from the signed-in identity, never entered by the user | Always | -- | -- | -- |
| posted_at | No validation beyond data type -- system-set at the moment of successful write | Always | -- | -- | -- |
| target | No validation beyond data type in this spec -- a target the posting author cannot reach is denied by FEAT-07.SPEC-007's authorization rules, not by a field rule here | Always | -- | -- | -- |
| reply_to | No validation beyond data type -- reserved field; no interaction on FEAT-07.SPEC-001 or FEAT-07.SPEC-002 ever sets it (see those specs' Non-Goals) | Always | -- | -- | -- |
| status | No validation beyond data type in this spec -- governed entirely by FEAT-07.SPEC-006 | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------------|
| N/A -- single-field validation | text | This spec validates `text` independently; every other field on the Comment entity is either system-derived (author, posted_at) or governed by a sibling spec (target's authorization by FEAT-07.SPEC-007, status by FEAT-07.SPEC-006), so no field combination interacts with this spec's rule | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Submit comment text (this spec's length/non-empty check only) | Nadia, Owen, Priya | The submitter's underlying authorization to post at all on the target thread is governed entirely by FEAT-07.SPEC-007; this spec's own condition is only that the submitted text passes the Field Validation Rules above | If the text fails this spec's rule: the exact messages above. Whether the submitter may reach the composer at all is a question FEAT-07.SPEC-007 answers, not this spec -- Dana never reaches this rule because the composer is never rendered for her (FEAT-07.SPEC-007) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------------------|---------------|---------------------|
| text | No default or derivation -- authored directly by the poster; leading and trailing whitespace is trimmed before the length rule is applied, but internal whitespace within the comment is preserved as typed | On create and on edit | N/A -- it is the user's direct input, not a value to override |

## Business Rules

- The 1--2,000 character non-empty rule is the single shared validation for comment submission across FEAT-07.SPEC-001, FEAT-07.SPEC-002, and FEAT-07.SPEC-008 (Feature Breakdown Brief, Shared Validation) -- none of those specs duplicates the rule, they all reference this spec by ID.
- Character count is measured in characters as displayed, not bytes, so multi-byte characters (accented letters, emoji) each count as one character toward the 2,000-character limit.
- This rule applies identically on comment creation and on an in-window edit (FEAT-07.SPEC-006) -- the product definition establishes no separate, looser rule for edits.
- A submission that fails this rule is rejected before any Comment record is written, and before FEAT-07.SPEC-003 or FEAT-07.SPEC-004's notification triggers can fire, since those triggers require a successful Comment write.

## Edge Cases

- **A comment of exactly 2,000 characters is submitted** -- Passes validation; the limit is inclusive.
- **A comment of exactly 1 character is submitted** -- Passes validation; the minimum is inclusive.
- **A comment of 0 characters (empty string) is submitted** -- Fails validation with "Enter a comment before posting."
- **A comment consisting only of spaces, tabs, or line breaks is submitted** -- Treated as empty after trimming; fails validation with the same message as a truly empty submission.
- **A comment of 2,001 characters is submitted** -- Fails validation with "Your comment can be up to 2,000 characters."
- **A comment with leading and trailing whitespace around otherwise valid text is submitted** -- The whitespace is trimmed before the length check; if the trimmed text is within 1--2,000 characters, it passes and is stored trimmed.
- **A comment composed while offline (FEAT-07.SPEC-008) was valid at compose time but the rule itself changes before the sync completes** -- Not applicable in this product: the 1--2,000 character rule is a fixed product-level constant, not a configurable value that can change between compose and sync; the sync re-check exists to catch a client-side bug or tampering, not a moving rule.
- **A comment containing only emoji characters, each within the visible-character count** -- Each emoji counts as one character toward the 1--2,000 limit; a comment of 1 to 2,000 emoji passes validation like any other text.

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 1 | 1 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 8 | 8 |



# Logic/Rule Spec: Comment Edit Window & Retraction Rule

## Overview

**Name:** Comment Edit Window & Retraction Rule
**ID:** FEAT-07.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs the short post-submit window in which an author may edit their own comment's text, the always-available author-only retraction, and the one-way, non-silent Posted -> Retracted transition.
**Parent Feature:** FEAT-07 -- Deliverable Review & Feedback
**Governed Entity:** Comment (the `text` field's edit eligibility, and the `status` field's lifecycle)

## Scope and Non-Goals

**In Scope:**
- The grace-window rule that determines whether an author may still edit their own comment's text
- The always-available, author-only retraction action and its effect on the comment's display
- The one-way Posted -> Retracted state transition and its immutability once made
- The "(edited)" marker rule that keeps an in-window edit visible as a change, never a silent rewrite

**Non-Goals:**
- The length/non-empty validation applied to edited text -- owned entirely by FEAT-07.SPEC-005 (Comment Content & Submission Validation), which this spec's edit path re-invokes rather than duplicates.
- Who may view a comment at all, and cross-company isolation -- owned entirely by FEAT-07.SPEC-007 (Comment Visibility & Authorization Rule); this spec assumes the viewer can already see the thread and governs only what the comment's own author may do to it.
- Restoring a retracted comment to Posted -- excluded per the Feature Breakdown Brief's Non-Goals: the transition is intentionally one-way; a contact who retracted in error re-adds their point as a new comment rather than reversing the retraction.
- Automatic purge or hard deletion of a retracted comment -- excluded per the Feature Breakdown Brief's Non-Goals: retraction is a soft removal only, retained indefinitely as evidentiary record until an account-level export/deletion request is honored by FEAT-24; this spec defines the Retracted state, never a deletion.

## Governed Entity

**Entity:** Comment
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| text | text (1--2,000 characters) | The comment's content -- this spec governs whether it may still be changed |
| author | reference (Client Contact or Freelancer Account) | Who wrote the comment -- this spec's ownership gate for edit and retraction |
| posted_at | date/time | When the comment was recorded -- the anchor this spec's grace window is measured from |
| target | reference (Deliverable Version or Milestone) | What the comment is pinned to -- governed elsewhere (FEAT-07.SPEC-001, FEAT-07.SPEC-002) |
| reply_to | reference (Comment), optional | The comment being replied to, if any -- governed elsewhere; unaffected by this spec |
| status | enum (Posted, Retracted) | The comment's lifecycle state -- this spec governs its sole transition |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-07.SPEC-001 | Deliverable Comment Thread | Edit control shown only within the window; Retract control shown on the author's own comments; both re-checked authoritatively at the moment of Save or Retract |
| FEAT-07.SPEC-002 | Milestone Comment Thread | Same enforcement points as FEAT-07.SPEC-001 |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| text (on edit) | Must pass FEAT-07.SPEC-005's non-empty, 1--2,000 character rule | Only when the edit occurs within the grace window (see Cross-Field Rules) | On Save tap | Per FEAT-07.SPEC-005: "Enter a comment before posting." or "Your comment can be up to 2,000 characters." | Yes |
| author | No validation beyond data type -- fixed at creation, never changed by edit or retraction | Always | -- | -- | -- |
| posted_at | No validation beyond data type -- fixed at creation; never altered by an edit (an edit changes `text` only, not the original posting time) | Always | -- | -- | -- |
| status | Must transition only Posted -> Retracted, exactly once, never reversed | On a retraction attempt | On Retract confirmation | N/A -- the Retract control is only ever shown while `status` is Posted, so no invalid-transition message is user-facing; a stale-state attempt is treated as the edge case below | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------------|
| Edit-within-window | text, posted_at, status | An edit to `text` may proceed only while the current time is within platform parameter: `comment-edit-grace-window-minutes` of `posted_at`, and only while `status` is Posted | "This comment can no longer be edited." |
| Retraction always available while Posted | status, author | Retraction may proceed at any time after `posted_at`, with no grace-window limit, as long as `status` is still Posted and the requester is the comment's `author` | N/A -- the Retract control is simply not shown once `status` is already Retracted |
| Retraction is terminal | status | Once `status` is Retracted, no further edit or retraction action may be attempted on this comment | N/A -- both Edit and Retract controls are removed once `status` is Retracted; the retracted placeholder replaces the comment's interactive controls entirely |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Edit own comment's text | Nadia, Owen, Priya | Only the comment's own `author`, only while `status` is Posted, only within platform parameter: `comment-edit-grace-window-minutes` of `posted_at` | Edit control is not shown once the window closes; a stale-UI attempt is denied with "This comment can no longer be edited." Edit control is never shown on a comment authored by someone else. |
| Edit own comment's text | Dana | Never | No Edit control is ever rendered for Dana -- her session is read-only in every feature (XBR-29) |
| Retract own comment | Nadia, Owen, Priya | Only the comment's own `author`, only while `status` is Posted -- no time limit | -- (always available to the author while Posted) |
| Retract own comment | Dana | Never | No Retract control is ever rendered for Dana -- her session is read-only in every feature (XBR-29) |
| Retract another author's comment | Nadia, Owen, Priya, Dana | Never -- retraction is strictly author-only, per the dependency map's Comment Contention note ("each comment is written and retracted only by its own author") | Retract control is never shown on a comment the viewer did not author |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status | Defaults to Posted | On comment creation | No |
| "(edited)" marker (display-only, not a stored field on its own but derived for rendering) | Shown whenever `text` has been changed at least once since `posted_at`, via an in-window edit | Rendered whenever the comment is displayed, after any successful edit | No -- it is a factual marker of edit history, not a user preference |

## Business Rules

- The grace window is measured from `posted_at`, not from the last edit -- a comment may be edited at most while still within platform parameter: `comment-edit-grace-window-minutes` of its original posting, not extended by making an earlier edit.
- An edited comment always shows the "(edited)" marker beside it once saved -- consistent with the product's evidentiary-integrity principle for feedback records that inform an approval decision (in the spirit of XBR-04's "never silently altered" for evidence, applied here since this feedback is shown as approval context on FEAT-08).
- Retraction is a one-way, terminal transition: Posted -> Retracted, never reversed (Feature Breakdown Brief, Entity-Lifecycle Coverage Matrix). A contact who retracted in error re-adds their point as a new comment.
- Retraction never cascades to replies: a retracted comment's own later replies remain fully visible and unaffected (Feature Breakdown Brief, Entity-Lifecycle Coverage Matrix).
- Retraction is a soft removal only -- the underlying Comment record is retained indefinitely as part of the record, per the dependency map's Data Sensitivity note ("included in the freelancer's export and removed on account deletion"); no independent purge policy exists within this feature.
- Editing and retraction apply identically to Nadia's own comments as to Owen's and Priya's -- the product definition establishes no role-specific exception to either rule.

## Edge Cases

- **Author attempts to edit at the exact instant the grace window closes** -- Denied; the boundary is inclusive of the window's end moment, so an edit attempt strictly after platform parameter: `comment-edit-grace-window-minutes` from `posted_at` is rejected with "This comment can no longer be edited."
- **Author retracts a comment they edited earlier within the grace window** -- Allowed; retraction has no time limit and is independent of whether the comment was ever edited. The retracted placeholder replaces the (possibly edited) text.
- **Author attempts a second edit within the window, after already editing once** -- Allowed; the window gates edit eligibility by elapsed time since `posted_at`, not by an edit count. Each successful edit keeps the "(edited)" marker (it does not multiply).
- **Two devices of the same author attempt to edit the same comment at effectively the same moment** -- The Comment entity's Contention note states retraction and edit are author-only and never contended by another actor; for the same author acting from two sessions, the later successful write's text is what persists (last-write-wins on the author's own single-owner field), and both sessions show the resulting saved text on next load.
- **Author attempts to retract a comment that has already been retracted (e.g., from a stale screen state in another tab)** -- The retraction is a no-op against an already-Retracted comment; the screen simply reflects the current Retracted state rather than erroring, since there is nothing further to change.
- **Author's role changes (e.g., a contact's role changes from Reviewer to Primary) while a comment is within its edit window** -- Edit and retraction eligibility depend only on authorship (`author`) and `status`/timing, never on the author's current role, so a role change during the window has no effect on this spec's rules.
- **A comment reaches the very end of platform parameter: `comment-edit-grace-window-minutes` while the author has the inline edit field open but has not yet tapped Save** -- The Save action is re-checked authoritatively at the moment it is tapped; if the window has closed by then, the save is denied with "This comment can no longer be edited." and the comment reverts to its last-saved text.
- **Client isolation is not a concern for this spec's own rules** -- Author-only edit and retraction never cross a client-company boundary, since `author` is always a single Client Contact or Nadia; cross-company visibility is governed entirely by FEAT-07.SPEC-007, not here.

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 6 | 6 |
| Edge Cases | 8 | 8 |



# Logic/Rule Spec: Comment Visibility & Authorization Rule

## Overview

**Name:** Comment Visibility & Authorization Rule
**ID:** FEAT-07.SPEC-007
**Type:** Logic/Rule
**Purpose:** Encodes who sees and can act on which comment thread: Own-only per client company for Owen and Priya, Full for Nadia, View-only inside a logged session for Dana, and strict cross-company isolation.
**Parent Feature:** FEAT-07 -- Deliverable Review & Feedback
**Governed Entity:** Comment (the `target` field's reachability, and view/post authorization across all fields)

## Scope and Non-Goals

**In Scope:**
- View authorization for a comment thread (deliverable-pinned or milestone-pinned), per role
- Post authorization for a new comment, per role
- Cross-company isolation for both view and post (XBR-09)
- Dana's read-only, logged-session-scoped visibility (XBR-29)
- Resolving which client company a given comment's `target` (Deliverable Version or Milestone) belongs to, for the purpose of scoping visibility

**Non-Goals:**
- Comment text length and non-empty validation -- owned entirely by FEAT-07.SPEC-005 (Comment Content & Submission Validation).
- Edit-window and retraction eligibility -- owned entirely by FEAT-07.SPEC-006 (Comment Edit Window & Retraction Rule); this spec governs whether a viewer may reach the thread at all, not what an already-entitled author may do to their own comment's lifecycle.
- Milestone approval authorization -- owned entirely by FEAT-08 (Milestone Approval); this spec governs comment visibility only, never the Approve action, even though both are shown on the same approval screen as separate concerns.
- Support session opening, closing, and inactivity timeout mechanics -- owned entirely by FEAT-31 (Operator Support Access); this spec only states that Dana's comment visibility exists solely inside such a session, not how the session itself is opened or closed.

## Governed Entity

**Entity:** Comment
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| text | text (1--2,000 characters) | The comment's content -- governed elsewhere (FEAT-07.SPEC-005); this spec only determines whether the viewer may see it at all |
| author | reference (Client Contact or Freelancer Account) | Who wrote the comment -- this spec uses the author's client-company membership to scope visibility for client contacts |
| posted_at | date/time | When the comment was recorded -- no authorization impact; used only for ordering |
| target | reference (Deliverable Version or Milestone) | What the comment is pinned to -- this spec resolves the target's owning Project and Client to determine who is entitled to view or post |
| reply_to | reference (Comment), optional | The comment being replied to, if any -- inherits the same visibility as its parent thread; governed elsewhere for its own semantics |
| status | enum (Posted, Retracted) | The comment's lifecycle state -- governed by FEAT-07.SPEC-006; this spec applies the same visibility rule regardless of status (a Retracted comment's placeholder is visible to the same audience as its text would have been) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-07.SPEC-001 | Deliverable Comment Thread | On screen entry (which comments render, which controls appear) and on Post tap (authorization re-check before write) |
| FEAT-07.SPEC-002 | Milestone Comment Thread | Same enforcement points as FEAT-07.SPEC-001 |
| FEAT-07.SPEC-008 | Offline Comment Queue & Sync | Authoritative re-check at the moment a queued comment's sync is attempted, before the write |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| target | Must resolve to a Deliverable Version or Milestone belonging to a Project of the client company the requesting Client Contact belongs to (for Owen and Priya), or to any client company (for Nadia and, inside a support session, Dana) | Always, on view and on post | On screen entry and on Post tap | N/A -- an out-of-scope target is never shown as an actionable error; it is handled as the out-of-scope-link experience below | Yes |
| author, text, posted_at, reply_to, status | No validation beyond data type in this spec -- governed by FEAT-07.SPEC-005 (text) and FEAT-07.SPEC-006 (status); this spec does not re-validate their content, only whether the requesting viewer may see or write them at all | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------------|
| Cross-company isolation | target, author | A comment's `target` and its `author` are always resolved against exactly one client company's Project; a viewer scoped to a different client company never sees this comment or its target's thread at all (XBR-09) | N/A -- handled as the out-of-scope-link experience, never a visible error naming another company's data |
| Retracted-status-neutral visibility | status, target | Whether `status` is Posted or Retracted has no effect on who may view the comment's position in the thread; only what is shown at that position differs (text vs. placeholder, per FEAT-07.SPEC-006) | N/A |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View comment thread (deliverable or milestone) | Nadia (Freelancer) | Full -- any thread belonging to any of her own clients' projects | -- |
| View comment thread (deliverable or milestone) | Owen (Client Primary Contact) | Own-only -- only threads whose target belongs to his own client company | Attempting to reach a thread for a different client company is treated as an out-of-scope link: plain explanation and a fresh-link option, never another company's comments (XBR-09) |
| View comment thread (deliverable or milestone) | Priya (Client Reviewer Contact) | Own-only -- only threads whose target belongs to her own client company | Same out-of-scope handling as Owen |
| View comment thread (deliverable or milestone) | Dana (Support Operator) | View -- any freelancer's thread, but only inside a logged, read-only support session opened for that specific freelancer (FEAT-31), never outside one | Outside a support session, Dana has no path to any thread at all -- there is no control or link surfaced to her |
| Post a comment | Nadia (Freelancer) | Full -- on any thread belonging to any of her own clients' projects | -- |
| Post a comment | Owen (Client Primary Contact) | Own-only -- only on threads whose target belongs to his own client company | Same out-of-scope handling as viewing; the composer is never shown for a thread outside his company |
| Post a comment | Priya (Client Reviewer Contact) | Own-only -- only on threads whose target belongs to her own client company | Same out-of-scope handling as viewing |
| Post a comment | Dana (Support Operator) | Never -- her session is unconditionally read-only in every feature (XBR-29) | Composer is never rendered inside a support session |
| View or post on a comment thread belonging to another client company entirely | Owen, Priya | Never | Out-of-scope link handling: plain explanation and a fresh-link option, never another company's data (XBR-09) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Effective visibility scope (derived, not a stored field) | Derived from the requesting identity: Nadia's own Freelancer Account (Full, across all her clients); a Client Contact's `Client` reference (Own-only, that one company); a support-session's target Freelancer Account (View, that one freelancer, for Dana) | Evaluated at every screen entry and every Post attempt | No -- the scope is derived entirely from the requester's identity and session, never chosen or widened by the user |

## Business Rules

- XBR-08: Role entitlements follow the Access Matrix everywhere -- Reviewer contacts (Priya) view and comment only, on the same footing as Primary contacts (Owen) for this feature, since both roles hold the same Comment entitlement (Own-only, view and comment) per the Access Matrix; the Primary/Reviewer distinction that matters elsewhere (accept, approve, pay) does not create a difference in comment access.
- XBR-09: Client isolation -- a contact reaches only their own company's projects under one freelancer; a person who is a contact for several freelancers sees each portal, and each portal's comment threads, separately; an out-of-scope or expired link shows a plain explanation and a fresh-link option, never another company's data.
- XBR-29: The operator's support sessions are read-only in every feature, cover one freelancer account at a time, and are always announced to the freelancer and logged in her trail -- Dana's comment visibility exists solely as an instance of this general rule, not a feature-specific exception.
- A comment's visibility never depends on its `status` (Posted or Retracted) -- the same audience that could see the comment's text can see its retracted placeholder in the same position.
- Milestone-level and deliverable-level comment threads (FEAT-07.SPEC-001, FEAT-07.SPEC-002) apply this exact same authorization rule, differing only in what `target` type is being resolved (Deliverable Version vs. Milestone).

## Edge Cases

- **Owen is a Primary contact for two different freelancers' Clientroom accounts** -- Each portal and each portal's comment threads are entirely separate; Owen's Own-only scope in one freelancer's portal never carries over to or reveals anything about the other (dependency map, Client Contact Relationships: "a person who is a contact for several freelancers holds a separate Client Contact per freelancer").
- **A Client Contact's role changes from Reviewer to Primary mid-session while viewing a thread** -- Comment view and post entitlement is identical for both roles (Own-only, view and comment), so a role change has no visibility effect on this spec's rules; only entitlements outside this feature (accept, approve, pay) are affected.
- **A Client Contact's status changes to Removed while they have a thread open** -- The authorization condition (Own-only, Active status) is re-checked at the moment of the next Post attempt, not only at screen load; a removed contact's next Post attempt is denied with the same experience as a contact who never had access, consistent with XBR-27 (a removed contact's access ends immediately).
- **Dana's support session closes (inactivity timeout or manual close, FEAT-31) while she is viewing a thread** -- Her view access ends the instant the session closes; any further attempt to reach the thread outside an open session finds no path to it at all.
- **A comment's target Milestone or Deliverable belongs to a project that has since been archived (FEAT-01)** -- Archived projects remain visible to the client until further archived per XBR-03; this spec's visibility scope is unaffected by a project's archived state, since archiving does not remove client access to historical records.
- **Two contacts from the same client company view the thread at the same time, one posting while the other is mid-read** -- Both are within the same Own-only scope for that one client company; each simply sees the other's post per the append-only ordering rule, with no authorization conflict, since visibility is scoped by company, not contended between contacts of the same company.
- **A support request names a freelancer account that Dana has never opened a session for** -- Dana has no visibility into that freelancer's comment threads until a Support Access Session is opened for that specific account (FEAT-31); there is no ambient or default visibility across freelancer accounts.
- **Nadia's own comment thread on a project belonging to one of her clients is opened by a different one of her own clients' contacts (a mistaken link)** -- Denied per XBR-09 exactly as it would be for any other client contact: out-of-scope link handling, never another company's comments, even though both companies share the same freelancer.

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |



# Automation Spec: Offline Comment Queue & Sync

## Overview

**Name:** Offline Comment Queue & Sync
**ID:** FEAT-07.SPEC-008
**Type:** Automation
**Purpose:** Holds a comment composed while offline on the device that composed it, then submits it automatically once connectivity returns, re-running the same validation and pin-target rules as an online submission.
**Parent Feature:** FEAT-07 -- Deliverable Review & Feedback

## Scope and Non-Goals

**In Scope:**
- Queuing a comment locally on the composing device when Post is tapped without connectivity, from either FEAT-07.SPEC-001 or FEAT-07.SPEC-002
- Detecting connectivity's return and automatically attempting each queued comment's submission, in the order it was queued
- Re-running FEAT-07.SPEC-005's content validation and FEAT-07.SPEC-007's authorization/pin-target resolution at the moment of the sync attempt, not only at the moment of queuing
- Firing the notification matching the comment's author role (FEAT-07.SPEC-003 for a client contact, FEAT-07.SPEC-004 for Nadia) once a queued comment's sync succeeds
- Keeping the queued state visibly distinct from a sent state throughout, per ASMP-27's degraded-state conventions

**Non-Goals:**
- The composer UI, the offline banner, and the queued-state indicator's appearance -- owned by FEAT-07.SPEC-001 and FEAT-07.SPEC-002; this automation defines the queuing and sync behavior those screens display.
- The content and authorization rules themselves -- owned entirely by FEAT-07.SPEC-005 and FEAT-07.SPEC-007; this automation re-invokes them rather than re-implementing them.
- Queuing any action other than posting a new or edited comment -- product-features.md and this feature's Brief define no other offline-capable action within FEAT-07 (retraction is not described as offline-capable in the Feature Breakdown Brief's States field, which names only comment composition as the offline scenario).
- Cross-device queue synchronization -- a comment queued on one device is held and submitted from that same device; the product definition (Feature Breakdown Brief, States: "held locally") describes device-local holding, not a server-side or cross-device queue, so a comment queued on a phone that later loses power is not resumed from a different device.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| User taps Post without connectivity | FEAT-07.SPEC-001 (Deliverable Comment Thread) or FEAT-07.SPEC-002 (Milestone Comment Thread) | Fires when Post is tapped and the device has no connectivity, or connectivity is lost mid-submission before the server confirms receipt | Composed comment text, pin target (Deliverable Version or Milestone reference), the signed-in author's identity, the intended pin type (deliverable or milestone) |
| Author retries or re-saves an unsent entry | FEAT-07.SPEC-001 (Deliverable Comment Thread) or FEAT-07.SPEC-002 (Milestone Comment Thread) | Fires when the author taps Retry on a Sync Failed entry (repeated server error) or saves a valid edit to a validation-failed entry while online; the entry is processed from step 4 exactly like a reconnect sync | The single queue entry (its current text, pin target, pin type, and author identity) |
| Connectivity returns | System (device connectivity state) | Fires when the composing device regains connectivity while one or more comments remain queued and unsent | The full local queue for this device, in the order each entry was queued |

## Processing Logic

1. **On Post tap without connectivity:** Capture the composed text, the pin target reference, the pin type, and the author's identity exactly as entered; append this as a new entry to the device-local queue with a Queued state; do not attempt any network submission at this moment.
2. Show the composer's offline banner and the queued indicator on the triggering screen (FEAT-07.SPEC-001 or FEAT-07.SPEC-002) immediately, in place of a "posted" confirmation, so the queued state is never mistaken for a sent state.
3. **On connectivity returning:** Read the device-local queue in the order entries were queued.
4. For the oldest still-Queued entry, re-run FEAT-07.SPEC-005's content validation against the entry's stored text.
5. If validation passes, re-run FEAT-07.SPEC-007's authorization and pin-target resolution against the entry's stored pin target and the author's current entitlement.
6. If both checks pass, write the Comment record with `posted_at` set to the current time (the moment the sync succeeds, not the original offline composition time) and `status` set to Posted.
7. On a successful write, fire the notification matching the comment's author role (FEAT-07.SPEC-003 for a client-authored comment, FEAT-07.SPEC-004 for a Nadia-authored comment) and mark the queue entry as Synced, removing it from the local queue.
8. Repeat steps 4--7 for each remaining Queued entry, strictly in queuing order, one at a time.
9. If any check in steps 4--5 fails, or the write in step 6 fails repeatedly across connectivity events (a single transient write failure leaves the entry Queued for automatic retry), mark that entry with a Sync Failed state (see Outcome Definitions) and continue to the next queued entry rather than halting the whole queue.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Synced successfully | Content and authorization checks both pass on reconnect | Comment created with `posted_at` at sync time, `status` Posted | The comment appears in the thread the next time the screen is open; the queued indicator clears | FEAT-07.SPEC-001, FEAT-07.SPEC-002, FEAT-07.SPEC-003, FEAT-07.SPEC-004 |
| Sync failed -- validation | The stored text now fails FEAT-07.SPEC-005 (this only occurs if the entry was corrupted locally, since the original Post tap already validated it) | No Comment created; the entry remains in the queue as Sync Failed | The triggering screen shows the entry with an error and offers the author a chance to edit or discard it before the next sync attempt | FEAT-07.SPEC-001, FEAT-07.SPEC-002 |
| Sync failed -- authorization | The author's entitlement to the pin target has changed since queuing (e.g., their contact status was set to Removed) | No Comment created; the entry remains in the queue as Sync Failed | The screen shows "This comment couldn't be sent because your access has changed." with no retry offered, since retrying would fail identically | FEAT-07.SPEC-001, FEAT-07.SPEC-002, FEAT-07.SPEC-007 |
| Sync failed -- server error | The write itself fails after both checks pass (e.g., a transient failure) | No Comment created; the entry remains Queued for automatic retry on the next connectivity event | Queued indicator remains; no error is shown unless the failure repeats across multiple connectivity events, in which case the entry is marked Sync Failed and the screen offers a manual Retry and Discard (FEAT-07.SPEC-001 / FEAT-07.SPEC-002 Sync Failed state) | FEAT-07.SPEC-001, FEAT-07.SPEC-002 |
| Automation unavailable (queue mechanism itself fails, e.g., local storage is full) | The device cannot record the queue entry at the moment Post is tapped offline | No local queue entry is created; the composer's typed text is preserved on screen | Inline message: "Couldn't queue this comment for sending. Check your connection and try again." with the text left in the composer for the author to retry once online | FEAT-07.SPEC-001, FEAT-07.SPEC-002 |

## Data Model

**Reads:** Comment -- the stored queue entry's text, pin target, pin type, and author identity, captured at the moment of the original offline Post tap. Client Contact -- current `status` and `role` at sync time, to re-check authorization via FEAT-07.SPEC-007.
**Creates:** Comment -- `text`, `author`, `target`, `reply_to` (never set), `status: Posted`, with `posted_at` set to the moment the sync succeeds.
**Updates:** None on the Comment entity directly -- a queue entry is either successfully created as a new Comment or remains pending/failed in the local queue, never partially written.
**Deletes:** None on the Comment entity -- a synced queue entry is simply cleared from the local, device-only queue, which is not itself a tracked product entity.

## Business Rules

- `posted_at` reflects the moment the sync succeeds, not the moment the author originally composed the comment offline -- the record's timestamp is never backdated to a moment connectivity did not yet exist for the write.
- The same content and authorization rules apply to a synced comment as to an online one (FEAT-07.SPEC-005, FEAT-07.SPEC-007) -- offline composition never grants a laxer path to the record.
- Queued comments from the same device are submitted strictly in the order they were queued, never re-ordered or batched into a single write.
- The queued state is always visibly distinct from a sent state on the triggering screen, per ASMP-27's degraded-state conventions ("states plainly when a comment is queued for send rather than pretending it posted while offline").
- A comment queued on one device is held and synced only from that device; it is not visible to other sessions of the same author until the sync succeeds and the Comment record is actually written.

## Edge Cases

- **The author closes the app or navigates away while a comment is queued but not yet synced** -- The queue entry persists on the device across app restarts and is attempted again the next time connectivity is confirmed and the app is opened, since the queue is device-local and durable, not a live in-memory state.
- **Connectivity returns and is lost again mid-sync of one entry** -- That entry's write is treated as not yet confirmed; on the next connectivity event, the same entry is attempted again from the "Sync failed -- server error" path rather than assumed sent, avoiding a false negative that would silently drop the comment.
- **The author edits the queued comment's text before it syncs** -- Editing a still-Queued entry (which has not yet become a Comment record) is a local edit to the queue entry itself, not an invocation of FEAT-07.SPEC-006 (which governs edits to an already-Posted comment); the edited text is what gets validated and submitted at the next sync attempt.
- **The author discards a queued comment before it syncs** -- The entry is removed from the local queue with no Comment ever created and no notification ever fired; nothing further happens.
- **Concurrent trigger firing (the same device queues two different comments while offline, then reconnects)** -- Both entries sync in queuing order per the Processing Logic; each produces its own independent Comment write and its own notification, with no merging.
- **Trigger fires while a previous sync run is in flight** -- A second connectivity-return event that fires while the queue is still processing the prior run does not start a second concurrent pass; the in-flight run continues to the end of the queue as it stood when it started, and any newly queued entry added mid-run is picked up by that same run reaching it in order, or by the next connectivity event if the run has already finished.
- **The pin target (Deliverable Version or Milestone) is removed or superseded between queuing and sync (e.g., the deliverable is replaced per FEAT-06)** -- The sync attempt's FEAT-07.SPEC-007 re-check resolves the target as it currently stands; a Deliverable Version is immutable and never removed once uploaded (dependency map, Deliverable Version Lifecycle: "never updated... never deleted in-product"), so this scenario cannot occur for a deliverable-pinned comment. A Milestone that has since been reopened or approved does not block the sync, since comment posting has no milestone-state gate.
- **The author's client contact status changes to Removed while a comment is queued** -- The sync's authorization re-check (FEAT-07.SPEC-007) catches this at sync time and produces the "Sync failed -- authorization" outcome, never silently submitting a comment on behalf of a contact who has lost access.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | Triggered by (inbound); Affects (outbound) | Offline Post tap queues here, and the screen's Retry and edit-Save controls on an unsent entry re-trigger its submission; sync outcome feedback (including the Sync Failed state with Edit, Retry, and Discard) appears on this screen |
| FEAT-07.SPEC-002 (Milestone Comment Thread) | Triggered by (inbound); Affects (outbound) | Same relationship as FEAT-07.SPEC-001, for milestone-pinned comments |
| FEAT-07.SPEC-005 (Comment Content & Submission Validation) | References (outbound) | Re-validates queued text at sync time |
| FEAT-07.SPEC-007 (Comment Visibility & Authorization Rule) | References (outbound) | Re-checks authorization and pin-target reachability at sync time |
| FEAT-07.SPEC-003 (Client Comment Alert to Freelancer) | Affects (outbound) | A successful sync of a client-authored comment fires this notification |
| FEAT-07.SPEC-004 (Freelancer Reply Alert to Client) | Affects (outbound) | A successful sync of a Nadia-authored comment fires this notification |

## Analytics and Success Signals

- **offline_comment_queued** (pin type: deliverable / milestone, author role) -- supports success-metrics.md: "Feedback Consolidation" (feedback captured even without connectivity still counts toward in-portal feedback once synced)
- **offline_comment_synced** (pin type, time held in queue) -- supports success-metrics.md: "Feedback Consolidation"
- **offline_comment_sync_failed** (reason: validation / authorization / server error) -- N/A -- no success-metrics.md metric measures sync failures directly; retained so a comment that never reaches the record is observable rather than silently lost

## Acceptance Criteria

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

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (offline post, author retry or re-save of an unsent entry, connectivity returns) | 3 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |
