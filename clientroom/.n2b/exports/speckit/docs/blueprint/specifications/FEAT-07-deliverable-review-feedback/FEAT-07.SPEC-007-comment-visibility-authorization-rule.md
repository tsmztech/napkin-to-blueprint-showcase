---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-07.SPEC-007
spec_name: Comment Visibility & Authorization Rule
spec_slug: comment-visibility-authorization-rule
parent_feature: FEAT-07
parent_feature_name: Deliverable Review & Feedback
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 8
acceptance_criteria_count: 16
---

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
