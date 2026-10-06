---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-06.SPEC-005
spec_name: Deliverable Validation & Removal Eligibility Rules
spec_slug: deliverable-validation-removal-eligibility-rules
parent_feature: FEAT-06
parent_feature_name: Deliverable Upload & Sharing
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 36
acceptance_criteria_count: 20
---

# Logic/Rule Spec: Deliverable Validation & Removal Eligibility Rules

## Overview

**Name:** Deliverable Validation & Removal Eligibility Rules
**ID:** FEAT-06.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs upload validity (milestone required, valid/reachable link, size ceiling), who may upload or remove a deliverable, and the approved-milestone removal block.
**Parent Feature:** FEAT-06 -- Deliverable Upload & Sharing
**Governed Entity:** Deliverable

## Scope and Non-Goals

**In Scope:**
- Field-level validation rules for the Deliverable record: milestone requirement, link format, and the file size ceiling
- Authorization rules for every action on the Deliverable record, per role
- The approved-milestone removal block (XBR-11) and its exact denied experience
- Default values and derived fields on the Deliverable record

**Non-Goals:**
- Checking whether a submitted link actually resolves -- owned by FEAT-06.SPEC-004 (Linked Asset Reachability Check), which this spec's format rule feeds into but does not perform itself
- Performing the resumable file transfer -- owned by FEAT-06.SPEC-003 (Resumable Upload Handling); this spec defines only the size-ceiling threshold it checks against
- Enforcing the per-freelancer storage allowance -- owned by FEAT-16.SPEC-004 (Storage Limit & Size Ceiling Rules); this spec's size ceiling is the per-file limit only, a distinct rule from the account-wide allowance
- Rules governing Deliverable Version (round 2+) creation, editing, or deletion -- excluded per the dependency map: Deliverable Version is immutable once uploaded and its later rounds belong to FEAT-17 (Deliverable Version History)

## Governed Entity

**Entity:** Deliverable
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| kind | enum | Uploaded file or linked external asset (design-tool, cloud-drive, or file-sharing link) |
| file or link | file reference \| text (URL) | The uploaded file, or a valid, reachable link (not copied in) |
| milestone | reference | Owning Milestone (required) |
| uploaded_at | date/time | Timestamp of creation (required) |
| size | number | File size in bytes, for uploaded files only |
| status | enum | Uploading, Active, Superseded, Removed |
| link_status | enum | reachable or flagged (linked assets only) |
| first_client_view_at | date/time (derived) | Timestamp of a client contact's first view, written by FEAT-13, not by this feature |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-06.SPEC-001 | Deliverable Upload | Link format on field blur; milestone-required is structural (always pre-supplied); authorization on screen entry |
| FEAT-06.SPEC-002 | Deliverable List & Management | Removal eligibility check at Remove confirmation; authorization on screen entry and on each action |
| FEAT-06.SPEC-003 | Resumable Upload Handling | File size ceiling check before a transfer begins |
| FEAT-06.SPEC-004 | Linked Asset Reachability Check | Consumes the link-format outcome as a precondition before running its own reachability check |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| kind | Required; must be exactly one of uploaded file or linked external asset, set once at creation and never changed afterward | Always | On creation | "Choose a file to upload or a link to paste." | Yes |
| file or link | Exactly one of a file or a link must be present, never both, never neither | Always | On Attach Deliverable | "Choose a file to upload or a link to paste -- not both." | Yes |
| file or link (file path) | File size must not exceed platform parameter: `deliverable-file-size-ceiling` | When kind is uploaded file | Before the transfer begins (FEAT-06.SPEC-003) | "This file is larger than the size limit for deliverables. Compress it or share it by link instead." | Yes |
| file or link (link path) | Must be a well-formed URL from a recognized design-tool, cloud-drive, or file-sharing domain (Figma, Google Drive, Dropbox) | When kind is linked external asset | On blur | "Enter a Figma, Google Drive, or Dropbox link." | Yes |
| file or link (link path) | Must resolve and be reachable (delegated entirely to FEAT-06.SPEC-004) | When kind is linked external asset and the format rule above passes | On Attach Deliverable, before the deliverable is marked ready | "This link couldn't be reached. Check that it's shared and try again." | Yes -- blocks the deliverable from becoming Active, not the submission itself |
| milestone | Required; must reference an existing Milestone on a Project the requesting freelancer owns | Always | Structural -- always pre-supplied by the entry point (FEAT-06.SPEC-001); never user-editable on this screen | "A milestone is required to attach a deliverable." (defensive message; not reachable through the normal flow) | Yes |
| uploaded_at | System-set on creation; no user input | Always | -- | -- | No validation beyond data type |
| size | System-captured from the completed transfer, for uploaded files only; not applicable to linked assets | When kind is uploaded file | -- | -- | No validation beyond data type (the size ceiling above is the file-selection rule, not a field format rule) |
| status | System-managed enum; never directly editable by any role -- transitions are driven by FEAT-06.SPEC-003, FEAT-06.SPEC-004, FEAT-06.SPEC-002 (removal), and FEAT-17 (supersession) | Always | -- | -- | No validation beyond data type -- transition legality is governed by the Business Rules and Authorization Rules below, not by field-level format checks |
| link_status | System-managed enum, linked assets only; set exclusively by FEAT-06.SPEC-004 | When kind is linked external asset | -- | -- | No validation beyond data type |
| first_client_view_at | Written by FEAT-13 on a client contact's first view; this feature never reads or writes this field | Always | -- | -- | No validation beyond data type -- out of this feature's write scope entirely |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Mutually exclusive content | kind, file or link | A Deliverable of kind "uploaded file" carries file bytes and no link value; a Deliverable of kind "linked external asset" carries a link value and no file bytes -- the two never coexist on one record | "Choose a file to upload or a link to paste -- not both." |
| Ready-state consistency | status, link_status | status can be Active only when: (a) kind is uploaded file and the transfer has fully completed, or (b) kind is linked external asset and link_status is reachable | Not user-facing -- this is an internal consistency rule enforced by FEAT-06.SPEC-003 and FEAT-06.SPEC-004, never violated through any user-facing path |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Upload a file or attach a linked asset (create) | Nadia (Freelancer) | Always, on a milestone within her own account | -- |
| Upload a file or attach a linked asset (create) | Owen (Client Primary Contact) | Never | The Deliverable Upload screen (FEAT-06.SPEC-001) is never part of the client portal's navigation; no control exists for Owen to attempt this action |
| Upload a file or attach a linked asset (create) | Priya (Client Reviewer Contact) | Never | Same as Owen -- no control exists in the client portal |
| Upload a file or attach a linked asset (create) | Dana (Support Operator) | Never | Dana's read-only support session never renders an upload action (XBR-29) |
| View deliverable (Nadia's management screen, FEAT-06.SPEC-002) | Nadia (Freelancer) | Always, on her own account's milestones | -- |
| View deliverable (client-facing, via FEAT-07) | Owen (Client Primary Contact) | Always, own company's projects only | -- |
| View deliverable (client-facing, via FEAT-07) | Priya (Client Reviewer Contact) | Always, own company's projects only | -- |
| View deliverable (Nadia's management screen, read-only rendering) | Dana (Support Operator) | Always, inside a logged support session on the account being supported; list and status only, no file download | Attempting a download is not possible -- no download control is rendered for Dana in a support session (XBR-29) |
| Preview / stream a deliverable's file | Nadia (Freelancer), Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Always, subject to the same view scoping above | -- |
| Preview / stream a deliverable's file | Dana (Support Operator) | Inline preview only, never a download, inside a logged support session | A download attempt shows "Support sessions are read-only and don't include file downloads." |
| Remove a deliverable | Nadia (Freelancer) | Only when the owning Milestone's status is not Approved (XBR-11) | Blocking dialog: "This milestone has been approved. Remove is not available -- use Replace to share a new version instead." |
| Remove a deliverable | Owen (Client Primary Contact) | Never | No Remove control exists on the client-facing view; a contact never removes a deliverable |
| Remove a deliverable | Priya (Client Reviewer Contact) | Never | No Remove control exists on the client-facing view |
| Remove a deliverable | Dana (Support Operator) | Never | No Remove control is rendered in a support session (XBR-29) |
| Replace a deliverable (initiate hand-off into FEAT-17) | Nadia (Freelancer) | Always, on her own account's deliverables, regardless of the owning Milestone's approval status | -- |
| Replace a deliverable (initiate hand-off into FEAT-17) | Owen, Priya, Dana | Never | No Replace control exists on the client-facing view or the read-only support session |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status | Uploading (uploaded file) or pending (linked external asset), transitioning to Active only on completion/reachability | On create, then automatically on completion | No -- always system-derived, never directly set by any role |
| link_status | pending, transitioning to reachable or flagged | On create (linked assets only), then set by FEAT-06.SPEC-004 | No |
| uploaded_at | Current date and time | On create only | No |
| size | The completed transfer's final byte count | On upload completion (uploaded files only) | No |

## Business Rules

- XBR-11: a deliverable on an Approved milestone cannot be removed, only superseded by a new version; every removal is recorded in the activity trail regardless of outcome (attempted-and-blocked removals are not themselves trail-logged, since no state change occurred -- only a successful removal writes a trail entry, per XBR-05's "record-worthy event" scope).
- XBR-05: every successful removal writes an append-only trail entry with actor and timestamp (FEAT-13).
- XBR-12: a Deliverable's status becomes Active, and the client is notified, only once its content is fully ready (upload complete or link reachable) -- this spec's field rules exist specifically to prevent a partial or broken deliverable from reaching Active status.
- XBR-14: the size ceiling in this spec (platform parameter: `deliverable-file-size-ceiling`) is a distinct, per-file rule from FEAT-16.SPEC-004's per-freelancer storage allowance; a file can pass this spec's ceiling check and still be rejected by the storage allowance during transfer.
- Copying or hosting linked content is out of scope (SC-08): this spec's link rules validate format and delegate reachability, but at no point do they cause the linked content itself to be copied into the product.

## Edge Cases

- **File size exactly at the ceiling** -- Passes validation; one byte over fails with the size-ceiling message.
- **Link with trailing whitespace or a tracking query parameter appended** -- The format check normalizes whitespace before evaluating the domain; a recognized domain with extra query parameters still passes format validation, since the reachability check (FEAT-06.SPEC-004) is the authoritative test of whether the link actually works.
- **Milestone deleted between screen load and Attach Deliverable submission** -- Structurally prevented in the normal flow (FEAT-06.SPEC-001's Error state catches an unresolvable milestone reference before the form is usable); if the milestone is removed in the narrow window after the form loads successfully, the create is rejected with "This milestone is no longer available." and no Deliverable record is created.
- **Milestone approved between Remove being tapped and the confirmation being submitted** -- The eligibility check re-runs at the moment of commit, not at the moment Remove was first tapped, per the dependency map's Contention note for Deliverable (reject-with-refresh); the confirmation dialog updates to the blocked message.
- **Milestone reopened (Reopened status) after having been Approved, with the deliverable never removed in between** -- Removal becomes allowed again once the Milestone leaves Approved status, since the block is keyed to Approved specifically, not to having ever been approved.
- **A file passes the size ceiling but the storage allowance is exhausted mid-transfer** -- This spec's ceiling check already passed; the subsequent allowance failure is FEAT-16.SPEC-004's rule, surfaced as a distinct failure by FEAT-06.SPEC-003, not re-evaluated by this spec.
- **Kind field somehow carries both a file and a link (defensive case)** -- Rejected at creation with the mutually-exclusive-content error; this state is not reachable through FEAT-06.SPEC-001's normal toggle-based UI, so this rule exists as a data-integrity backstop.

## Acceptance Criteria

**FEAT-06.SPEC-005-AC-01:** Given Nadia selects a file exactly at platform parameter: `deliverable-file-size-ceiling`, when the size check runs, then the file passes validation.

**FEAT-06.SPEC-005-AC-02:** Given Nadia selects a file one byte over platform parameter: `deliverable-file-size-ceiling`, when the size check runs, then she sees "This file is larger than the size limit for deliverables. Compress it or share it by link instead." and the transfer does not begin.

**FEAT-06.SPEC-005-AC-03:** Given Nadia pastes a link from an unrecognized domain, when she blurs the field, then she sees "Enter a Figma, Google Drive, or Dropbox link." and the link is not submitted for a reachability check.

**FEAT-06.SPEC-005-AC-04:** Given Nadia pastes a well-formed Dropbox link with a tracking query parameter appended, when she blurs the field, then the format check passes and the link proceeds to the reachability check.

**FEAT-06.SPEC-005-AC-05:** Given Nadia's milestone reference is removed in the narrow window between form load and submission, when she attempts to attach a deliverable, then the creation is rejected with "This milestone is no longer available." and no Deliverable record is created.

**FEAT-06.SPEC-005-AC-06:** Given Nadia is the freelancer on her own account, when she attaches a deliverable to any of her milestones, then the action is always allowed.

**FEAT-06.SPEC-005-AC-07:** Given Owen (Client Primary Contact) is signed in to his portal, when he looks for a way to upload a deliverable, then no such control exists anywhere in his view.

**FEAT-06.SPEC-005-AC-08:** Given Dana is inside a read-only support session, when she views a milestone's deliverables, then she sees the list and status badges but no upload, remove, or replace controls, and no download option on preview.

**FEAT-06.SPEC-005-AC-09:** Given Nadia attempts to remove a deliverable on a milestone whose status is not Approved, when she confirms removal, then it is allowed and the deliverable's status becomes Removed.

**FEAT-06.SPEC-005-AC-10:** Given Nadia attempts to remove a deliverable on an Approved milestone, when she confirms removal, then it is refused with "This milestone has been approved. Remove is not available -- use Replace to share a new version instead."

**FEAT-06.SPEC-005-AC-11:** Given a milestone becomes Approved between Nadia tapping Remove and confirming, when she confirms, then the re-check at commit blocks the removal.

**FEAT-06.SPEC-005-AC-12:** Given a previously Approved milestone is later Reopened, when Nadia attempts to remove its deliverable, then removal is allowed because the block applies only while the milestone is Approved.

**FEAT-06.SPEC-005-AC-13:** Given Owen views a deliverable on his own company's project, when he looks for a Remove control, then none is shown -- removal is Nadia-only.

**FEAT-06.SPEC-005-AC-14:** Given Nadia removes an eligible deliverable, when the removal completes, then an append-only activity-trail entry is written with her identity and the timestamp.

**FEAT-06.SPEC-005-AC-15:** Given Nadia attempts to remove a deliverable and it is blocked, when the block occurs, then no activity-trail entry is written, since no state change occurred.

**FEAT-06.SPEC-005-AC-16:** Given Nadia chooses Replace on a deliverable whose milestone is Approved, when she initiates it, then the action is allowed, since Replace is never blocked by milestone approval status.

**FEAT-06.SPEC-005-AC-17:** Given Priya (Client Reviewer Contact) views a deliverable, when she looks for a Replace control, then none is shown.

**FEAT-06.SPEC-005-AC-18:** Given a Deliverable's kind is uploaded file, when its transfer has not yet fully completed, then its status can never be Active, regardless of any other field's value.

**FEAT-06.SPEC-005-AC-19:** Given a Deliverable's kind is linked external asset with link_status flagged, when any process checks its readiness, then its status is never Active and the client is never notified.

**FEAT-06.SPEC-005-AC-20:** Given Dana previews a deliverable's file inside her support session, when she looks for a download option, then none is offered, and an attempted download shows "Support sessions are read-only and don't include file downloads."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 10 | 10 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 15 | 15 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |
