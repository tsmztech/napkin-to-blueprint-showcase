---
document_type: feature-overview
feature_number: FEAT-07
feature_name: Deliverable Review & Feedback
feature_slug: deliverable-review-feedback
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 8
screen_count: 2
automation_count: 1
logic_rule_count: 3
integration_count: 0
notification_count: 2
---

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
