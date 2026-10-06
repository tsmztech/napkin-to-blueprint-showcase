---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-07.SPEC-006
spec_name: Comment Edit Window & Retraction Rule
spec_slug: comment-edit-window-retraction-rule
parent_feature: FEAT-07
parent_feature_name: Deliverable Review & Feedback
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 9
acceptance_criteria_count: 14
---

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
