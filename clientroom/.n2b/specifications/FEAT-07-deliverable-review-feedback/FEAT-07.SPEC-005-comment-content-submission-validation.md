---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-07.SPEC-005
spec_name: Comment Content & Submission Validation
spec_slug: comment-content-submission-validation
parent_feature: FEAT-07
parent_feature_name: Deliverable Review & Feedback
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 6
acceptance_criteria_count: 11
---

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
