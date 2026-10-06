---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-17.SPEC-005
spec_name: Version Access & Comment-Anchoring Rules
spec_slug: version-access-comment-anchoring-rules
parent_feature: FEAT-17
parent_feature_name: Deliverable Version History
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 12
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Version Access & Comment-Anchoring Rules

## Overview

**Name:** Version Access & Comment-Anchoring Rules
**ID:** FEAT-17.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs who may view, open, download, or create a Deliverable Version per the Access Matrix, and the rule that a comment stays attached to the version it was posted on even after newer rounds exist.
**Parent Feature:** FEAT-17 -- Deliverable Version History
**Governed Entity:** Deliverable Version

## Scope and Non-Goals

**In Scope:**
- Per-role authorization for viewing, opening, downloading, and creating a Deliverable Version
- Client isolation for version access (a version never crosses client-company boundaries)
- Dana's read-only, no-download constraint inside a support session
- The comment-anchoring rule: a Comment's target stays fixed to the specific version it was posted on, regardless of later uploads
- The authority both this feature's own screens and FEAT-07's comment screens reference for anchoring, so neither re-derives it independently

**Non-Goals:**
- Round numbering, immutability of a version's own fields, and the version-count/retention ceiling -- governed by FEAT-17.SPEC-004 (Version Numbering, Immutability & Retention Rules)
- Creating, editing, or retracting a comment -- owned entirely by FEAT-07 (Deliverable Review & Feedback); this spec only defines what a comment's target reference means once it exists, not how the comment itself is written or removed
- Authorization for the deliverable itself (as opposed to its versions) -- governed by FEAT-06's own access rules; this spec's authority begins at the version level, consistent with Deliverable Version's Access field ("Follows Deliverable Upload & Sharing's Access field")

## Governed Entity

**Entity:** Deliverable Version
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| round_number | number | No validation beyond data type -- see FEAT-17.SPEC-004 for sequencing and derivation rules; this spec governs only who may see it |
| file | file reference | No validation beyond data type -- see FEAT-17.SPEC-004; this spec governs only who may open or download it |
| uploaded_at | date/time | No validation beyond data type -- see FEAT-17.SPEC-004; this spec governs only who may see it |
| is_latest | derived (boolean) | No validation beyond data type -- see FEAT-17.SPEC-004 for the derivation formula; this spec governs only who may see it |

**Referenced entity (for the anchoring rule):** Comment -- specifically its `target` field, which the dependency map defines as "a deliverable version, a milestone as a whole, or a proposal (change request)." Comment creation, editing, and retraction remain owned by FEAT-07; this spec is the rule authority for what `target` means once it points at a Deliverable Version.

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-17.SPEC-001 | New Version Upload | Authorization checked on screen entry (only Nadia can reach this screen at all) |
| FEAT-17.SPEC-002 | Version Browser & Comparison | Authorization checked on screen entry (per-role view/download scoping) and on every round-open and download attempt |
| FEAT-17.SPEC-003 | Version Creation & Preservation | Confirms the triggering session is Nadia's before creating a version (defense-in-depth behind FEAT-17.SPEC-001's screen-entry check) |
| FEAT-07 (Deliverable Review & Feedback, comment screens) | Comment thread per deliverable/version | Reads the anchoring rule when displaying which round a comment belongs to, and when a comment is posted while an older round is open |
| FEAT-31 (Operator Support Access) | Read-only support session | Confirms Dana's session renders view-only, no-download access to version data, per this spec's authorization table |

## Field Validation Rules

No field-level validation rules are defined by this spec -- validation, sequencing, and derivation for every field of Deliverable Version are governed entirely by FEAT-17.SPEC-004. This spec governs authorization (who may see or act on those fields) and the comment-anchoring rule, not the fields' own content.

## Cross-Field Rules

No cross-field validation rules are defined by this spec. The one cross-entity rule this spec owns -- comment anchoring, between Comment.target and Deliverable Version -- is documented under Business Rules below, since it governs behavior over time rather than the validity of a single record's fields.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create the next Deliverable Version (trigger a re-upload) | Nadia (Freelancer) | Always, for any deliverable she owns whose kind is an uploaded file | -- |
| Create the next Deliverable Version | Owen (Client Primary Contact) | Never | "Upload New Version" control is never rendered anywhere in the client portal (FEAT-17.SPEC-001's Access and Visibility table) |
| Create the next Deliverable Version | Priya (Client Reviewer Contact) | Never | Same as Owen -- no upload control exists in the client portal |
| Create the next Deliverable Version | Dana (Support Operator) | Never | No upload entry point is rendered inside a support session, consistent with support sessions never editing anything (ASMP-18) |
| View the version list and open a round's metadata (round_number, uploaded_at) | Nadia (Freelancer) | Full -- every version of every deliverable across her own account | -- |
| View the version list and open a round's metadata | Owen (Client Primary Contact) | Own-only -- only versions of deliverables belonging to his own client company | Versions of another client's deliverables never appear or resolve; a direct link is refused with "This page isn't part of your portal." (XBR-09) |
| View the version list and open a round's metadata | Priya (Client Reviewer Contact) | Own-only -- only versions of deliverables belonging to her own client company | Same as Owen |
| View the version list and open a round's metadata | Dana (Support Operator) | View only, inside a logged, read-only support session (FEAT-31), for the one freelancer account currently under support | Outside an active, logged support session, no version data for any account is reachable |
| Open or download a round's underlying file | Nadia (Freelancer) | Full -- any version of any of her own deliverables | -- |
| Open or download a round's underlying file | Owen (Client Primary Contact) | Own-only -- any version of his own client company's deliverables | Same client-isolation denial as above for versions outside his own company |
| Open or download a round's underlying file | Priya (Client Reviewer Contact) | Own-only -- any version of her own client company's deliverables | Same client-isolation denial as above |
| Open or download a round's underlying file | Dana (Support Operator) | Never | No download control is rendered for any round inside a support session; a direct attempt is refused with "Downloads are not available in a support session." (ASMP-18, ASMP-23) |

## Defaults and Derivations

N/A -- this spec governs authorization and comment anchoring, not field defaults or derivations. Round_number's assignment, uploaded_at's timestamping, and is_latest's derivation formula are all owned by FEAT-17.SPEC-004.

## Business Rules

- Comment anchoring (XBR-13): once a Comment's `target` field is set to a specific Deliverable Version at the moment of posting (by FEAT-07), that reference is permanent -- it is never migrated, re-pointed, or reinterpreted when a newer round is uploaded to the same deliverable. A comment posted on Round 2 stays a Round 2 comment forever, even once Round 5 exists.
- This anchoring holds regardless of which round is currently `is_latest`: FEAT-17.SPEC-002's comment count indicator for a given round only ever counts comments whose `target` is that exact round, never comments from any other round of the same deliverable.
- FEAT-07 owns Comment creation, editing, and retraction; this spec is the sole rule authority for what the `target` reference means and how long it holds -- FEAT-07's own specs reference this spec by ID rather than re-deriving the anchoring behavior.
- XBR-09: client isolation applies to every version-access action in this spec exactly as it applies elsewhere in the portal -- a contact reaches only their own company's deliverable versions, never another company's, under any circumstance.
- ASMP-18 / ASMP-23: Dana's support session is read-only everywhere, including here -- she may list and browse version metadata but never open or download the underlying file of any round, mirroring FEAT-06 and FEAT-16's identical Support Access boundary.
- Access to a Deliverable Version's own data follows Deliverable Upload & Sharing's Access field (feature-overview.md: "Follows Deliverable Upload & Sharing's Access field") -- this spec's per-role rows above are the version-specific restatement of that same boundary, never a divergent one.

## Edge Cases

- **A client contact posts a comment while viewing an older round, then a newer round is uploaded moments later** -- The comment's anchoring is unaffected: it was already fixed to the older round's Deliverable Version the instant it was posted, before the new round existed. The new round has zero comments of its own until someone posts directly on it.
- **Priya comments on Round 2, then Round 2 is (hypothetically) the only round anyone ever views again because Round 3 supersedes it as latest** -- Her comment remains fully visible and intact on Round 2 forever; comment visibility is never tied to a round's `is_latest` status, only to which round the comment's `target` names.
- **Dana's support session is active while Owen is also viewing the same deliverable's versions** -- Both view independently and concurrently with no interaction; Dana's session never affects what Owen sees, and Owen's viewing never appears in Dana's session.
- **A client contact's role changes from Reviewer to Primary (or vice versa) partway through a project with existing version-anchored comments** -- Already-posted comments and their anchoring are unaffected by a later role change; the changed role governs only that contact's future actions (view/comment/approve entitlements), never past comments' anchoring.
- **A contact who is a client contact for two different freelancers views version histories in both portals** -- Each portal's version access is scoped entirely to that freelancer's own clients; no version, comment, or anchoring reference ever crosses between the two freelancers' accounts (XBR-09).
- **Dana attempts to construct a direct link to a version's file, bypassing the rendered support-session UI** -- The download is refused with "Downloads are not available in a support session." regardless of how the request is made, since the no-download rule is enforced at the authorization boundary, not only in the rendered controls.

## Acceptance Criteria

**FEAT-17.SPEC-005-AC-01:** Given Nadia opens New Version Upload on a deliverable she owns, when the screen checks authorization, then she is allowed to proceed.

**FEAT-17.SPEC-005-AC-02:** Given Owen attempts to reach a "New Version Upload" control anywhere in his portal, when he looks for one, then none is rendered -- only Nadia can create a version.

**FEAT-17.SPEC-005-AC-03:** Given Priya attempts to reach a "New Version Upload" control anywhere in her portal, when she looks for one, then none is rendered.

**FEAT-17.SPEC-005-AC-04:** Given Dana is inside a read-only support session, when she looks for a way to trigger a new version upload, then no such entry point is rendered.

**FEAT-17.SPEC-005-AC-05:** Given Owen opens the version browser for a deliverable belonging to his own client company, when the screen loads, then every version of that deliverable is visible to him.

**FEAT-17.SPEC-005-AC-06:** Given Owen attempts to reach a version of a deliverable belonging to a different client company, when the request is made, then he sees "This page isn't part of your portal." and no version data is returned.

**FEAT-17.SPEC-005-AC-07:** Given Priya opens the version browser for her own client company's deliverable, when she selects a round, then she can view and download that round's file.

**FEAT-17.SPEC-005-AC-08:** Given Dana is inside a read-only support session viewing a deliverable's versions, when she looks for a download control on any round, then none is rendered.

**FEAT-17.SPEC-005-AC-09:** Given Dana attempts to download a version's file by a means other than the rendered controls during a support session, when the attempt is made, then it is refused with "Downloads are not available in a support session."

**FEAT-17.SPEC-005-AC-10:** Given Priya posts a comment while viewing Round 2 of a deliverable, when Nadia later uploads Round 3, then Priya's comment remains anchored to Round 2 and does not appear when Round 3 is opened.

**FEAT-17.SPEC-005-AC-11:** Given a comment is anchored to Round 2 and Round 2 is no longer the latest version, when any authorized viewer opens Round 2 directly, then the comment is still fully visible there.

**FEAT-17.SPEC-005-AC-12:** Given Owen's role changes from Reviewer to Primary partway through a project, when his historical comments anchored to earlier rounds are viewed, then they remain unchanged and still correctly anchored.

**FEAT-17.SPEC-005-AC-13:** Given a person is a client contact for two different freelancers, when they view Freelancer A's version history, then no version, comment, or metadata from Freelancer B's account is ever visible.

**FEAT-17.SPEC-005-AC-14:** Given Nadia opens the version browser for any of her own deliverables, when she selects any round, then she can view, open, and download it without restriction.

**FEAT-17.SPEC-005-AC-15:** Given Dana's support session and Owen's own portal session are both open on the same deliverable's versions at the same time, when either views a round, then neither session's activity is visible to or affects the other.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 (all N/A -- referred to FEAT-17.SPEC-004) | 4 |
| Cross-Field Rules | 0 (none owned by this spec) | 0 |
| Authorization Rules | 12 | 12 |
| Defaults/Derivations | 0 (N/A -- referred to FEAT-17.SPEC-004) | 0 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
