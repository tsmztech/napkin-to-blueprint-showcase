---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-03.SPEC-005
spec_name: Acceptance & Access Rules
spec_slug: acceptance-access-rules
parent_feature: FEAT-03
parent_feature_name: Proposal Acceptance
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 29
acceptance_criteria_count: 17
---

# Logic/Rule Spec: Acceptance & Access Rules

## Overview

**Name:** Acceptance & Access Rules
**ID:** FEAT-03.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs who may accept a proposal or request changes on it, enforces single-acceptance and voided-proposal eligibility, defines the change-request note's length rule, and resolves concurrent-action conflicts.
**Parent Feature:** FEAT-03 -- Proposal Acceptance
**Governed Entity:** Proposal (acceptance and eligibility fields); the change-request note field on the Comment record this feature creates is governed here as a secondary field, since the Feature Breakdown Brief groups it with this spec's Validation & Limits.

## Scope and Non-Goals

**In Scope:**
- Eligibility rules for accepting a proposal (accept-once, voided-cannot-accept)
- Eligibility rules for submitting a change-request note (voided-cannot-request, accepted-cannot-request)
- The change-request note's length rule (1--2,000 characters)
- Authorization rules for view, accept, and request-changes actions on the Proposal, per role
- Concurrent-action resolution for the Proposal entity (reject-with-refresh) as it applies to acceptance and change requests

**Non-Goals:**
- Proposal creation, editing, and voiding rules -- owned entirely by FEAT-02 (Proposal Creation & Sending); this spec only reads the Proposal's current `status` to determine eligibility for this feature's actions.
- General Comment field rules (edit-within-grace-window, retraction, reply threading) -- owned by FEAT-07 (Deliverable Review & Feedback); this spec governs only the length rule for the change-request note at the moment it is created.
- Payment Schedule rules and the deposit-invoice trigger's own eligibility -- owned by FEAT-04 (Milestone & Payment Schedule Setup) and FEAT-09 (Invoice Generation & Sending) respectively; this spec's concern ends at the Proposal's acceptance eligibility.
- Legally binding e-signature eligibility -- deferred per scope-boundaries.md's Deferral note; FEAT-26 (v1) defines its own eligibility rules when signature is enabled for a proposal, extending rather than replacing this spec's MVP rules (XBR-34).

## Governed Entity

**Entity:** Proposal
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| status | enum (Draft, Sent, Voided, Accepted) | The proposal's current lifecycle state; this spec governs eligibility transitions out of Sent |
| sent_at | date/time | When the proposal was sent; no validation rule beyond data type in this spec |
| accepted_at | date/time | Written once on acceptance, never altered; this spec defines when it may be written |
| accepted_by | reference (Client Contact) | The accepting contact's identity; this spec defines who may cause this field to be set |
| payment_schedule_reference | reference (Payment Schedule) | Link to the project's Payment Schedule; no validation rule beyond data type in this spec -- read-only reference for FEAT-03.SPEC-003's deposit determination |

**Secondary governed field (Comment record created by this feature):**

| Field | Data Type | Description |
|-------|-----------|-------------|
| text (change-request variant) | text | The 1--2,000 character note Owen sends to Nadia instead of accepting; governed here per the Feature Breakdown Brief's Validation & Limits grouping |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|----------------------|
| FEAT-03.SPEC-001 | Proposal Review & Accept | Access and Visibility on screen entry; eligibility check on Accept tap (screen-level, non-authoritative) |
| FEAT-03.SPEC-002 | Request Changes | Access and Visibility on screen entry; note length check on Send tap and on change (screen-level, non-authoritative) |
| FEAT-03.SPEC-003 | Acceptance Recording | Authoritative write-time re-check of proposal eligibility (accept-once, voided-cannot-accept) before writing the acceptance |
| FEAT-03.SPEC-004 | Change-Request Recording | Authoritative write-time re-check of note length and proposal eligibility (voided-cannot-request, accepted-cannot-request) before writing the Comment |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|-------------------------------|---------------|---------------------|-----------|
| status | No validation beyond data type -- this spec reads it, FEAT-02 owns writing it (except the Accepted transition, governed below) | Always | -- | -- | -- |
| sent_at | No validation beyond data type | Always | -- | -- | -- |
| accepted_at | Must be set exactly once, only by the Acceptance Recording write (FEAT-03.SPEC-003); never altered afterward | Proposal transitions Sent -> Accepted | On write (FEAT-03.SPEC-003) | N/A -- system-set field, no user-facing error; a second attempted write is intercepted by the accept-once rule below, not a field-level message | Yes |
| accepted_by | Must reference a Client Contact with role Primary and status Active at the client owning the proposal | Proposal transitions Sent -> Accepted | On write (FEAT-03.SPEC-003) | N/A -- system-set field; the gating condition surfaces as the authorization denial below, not a field-level message | Yes |
| payment_schedule_reference | No validation beyond data type | Always | -- | -- | -- |
| text (change-request note) | Required, non-empty, 1--2,000 characters | Always | On Send tap (screen) and on write (FEAT-03.SPEC-004, authoritative) | "Enter a note before sending." (empty) / "Your note can be up to 2,000 characters." (over length) | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------------|
| Accept-once | status, accepted_at | Accept may only proceed when `status` is `Sent`; once `status` is `Accepted`, `accepted_at` is fixed and no further accept attempt may change it | "This proposal has already been accepted." |
| Voided-cannot-accept | status | Accept may only proceed when `status` is `Sent`, never when `status` is `Voided` | N/A -- the client is redirected to the current proposal version rather than shown an inline error (FEAT-03.SPEC-001) |
| Voided-cannot-request-changes | status | Request Changes may only proceed when `status` is `Sent`, never when `status` is `Voided` | N/A -- the client is directed to the current proposal version (FEAT-03.SPEC-002) |
| Accepted-cannot-request-changes | status | Request Changes may only proceed when `status` is `Sent`, never when `status` is `Accepted` | "This proposal has already been accepted." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View proposal content (scope, price) | Owen (Client Primary Contact) | Own-only -- only his own company's proposal, and only when `status` is `Sent` or `Accepted` (XBR-08, XBR-09) | -- |
| View proposal content (scope, price) | Priya (Client Reviewer Contact) | Never | Full screen is not shown; portal home shows only the project's stage label (e.g., "Proposal accepted"), per the Access Matrix (Proposals & Acceptance: None for Reviewers) |
| View proposal content (scope, price) | Nadia (Freelancer) | Never through this feature's screens -- she sees the resulting status on the project view (FEAT-01), not FEAT-03.SPEC-001 | Attempting to open FEAT-03.SPEC-001's link is treated as an out-of-scope link (XBR-09): plain explanation and a fresh-link option |
| View proposal content (scope, price) | Dana (Support Operator) | Always, read-only, inside a logged support session (FEAT-31) | -- |
| Accept proposal | Owen (Client Primary Contact) | Own-only, and only when `status` is `Sent` (accept-once and voided-cannot-accept, above) | If `status` is `Accepted`: "This proposal has already been accepted." If `status` is `Voided`: redirected to the current version, no error text shown |
| Accept proposal | Priya (Client Reviewer Contact) | Never | Accept control is not shown -- she never reaches proposal content |
| Accept proposal | Nadia (Freelancer) | Never | No Accept control exists on any screen Nadia can reach; acceptance is exclusively a client-side action |
| Accept proposal | Dana (Support Operator) | Never | Accept control is not rendered inside a support session (FEAT-31) |
| Request changes | Owen (Client Primary Contact) | Own-only, and only when `status` is `Sent` (voided-cannot-request and accepted-cannot-request, above) | If `status` is `Voided`: directed to the current version. If `status` is `Accepted`: "This proposal has already been accepted." |
| Request changes | Priya (Client Reviewer Contact) | Never | Request Changes control is not shown -- she never reaches proposal content |
| Request changes | Nadia (Freelancer) | Never | No Request Changes control exists on any screen Nadia can reach |
| Request changes | Dana (Support Operator) | Never | Request Changes control is not rendered inside a support session (FEAT-31) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------------------|---------------|---------------------|
| accepted_at | Derived as the current timestamp at the moment the Acceptance Recording write (FEAT-03.SPEC-003) succeeds | On the Sent -> Accepted transition, once | No |
| accepted_by | Derived as the Client Contact reference of whichever eligible Primary contact's accept attempt wins the exactly-once write (FEAT-03.SPEC-003) | On the Sent -> Accepted transition, once | No |

## Business Rules

- XBR-06: A voided proposal cannot be accepted; a project has at most one active proposal at a time, and a voided proposal directs the client to the current one.
- XBR-08: Only Primary contacts accept proposals and request changes; Reviewer contacts never see proposal or invoice content.
- XBR-09: Client isolation -- a contact reaches only their own company's proposal; an out-of-scope or expired link shows a plain explanation and a fresh-link option, never another company's data.
- Acceptance is recorded exactly once, per the dependency map's Proposal Contention note: two Primary contacts accepting at the same moment yield one acceptance, and the second sees "already accepted." This spec defines the eligibility rule that FEAT-03.SPEC-003 enforces atomically at write time; this spec does not itself perform the write.
- The Proposal Contention resolution for a race between Nadia's edit-and-void (FEAT-02) and Owen's accept or request-changes attempt (this feature) is reject-with-refresh: an accept or request-changes attempt against a version voided in the meantime is refused and the client is shown the current version.
- If the accept action fails mid-flight (e.g., a connectivity drop after tap), the action is retried without recording a duplicate acceptance or losing the client's click intent -- enforced within FEAT-03.SPEC-003's atomic write, per this spec's accept-once rule.
- Dana's support session sees this feature's screens read-only, per XBR-29: no Accept or Request Changes control is ever rendered for her, regardless of the proposal's state.

## Edge Cases

- **Owen accepts a proposal at the exact instant its `status` field is Sent, with `accepted_at` and `accepted_by` both unset** -- Standard eligible-accept path; no boundary issue since this is the normal Sent state.
- **A change-request note of exactly 2,000 characters is submitted** -- Passes validation; the limit is inclusive per the field rule above.
- **A change-request note of exactly 1 character is submitted** -- Passes validation; the minimum is inclusive.
- **A change-request note of 0 characters (empty string) is submitted** -- Fails validation with "Enter a note before sending." -- whitespace-only input is treated as empty for this purpose, since it carries no substantive note content.
- **Two Primary contacts at the same client both attempt Accept within the same instant** -- The accept-once rule is enforced atomically by FEAT-03.SPEC-003's write, not by this spec directly; this spec defines that exactly one may succeed and the other must see "This proposal has already been accepted."
- **A Primary contact's role is changed to Reviewer, or their status is set to Removed, while they are mid-session on FEAT-03.SPEC-001** -- The authorization condition (Own-only, Primary, Active) is re-checked at the moment of the Accept or Request Changes action, not only at screen load; if the role or status has changed, the action is denied with the same "not a Primary contact" experience as if they had never had access, consistent with XBR-27 (a removed contact's access ends immediately).
- **Owen attempts to accept a proposal for a project belonging to a different client than the one his contact record is scoped to** -- Denied per XBR-09: this is treated as an out-of-scope link, showing a plain explanation and a fresh-link option, never another company's proposal.
- **The proposal transitions from Sent to Voided between FEAT-03.SPEC-001's screen-level eligibility check and Owen's Accept tap reaching FEAT-03.SPEC-003** -- The screen-level check is non-authoritative; FEAT-03.SPEC-003's write-time re-check is what actually catches this and returns the voided-redirect outcome.

## Acceptance Criteria

**FEAT-03.SPEC-005-AC-01:** Given Owen is viewing a proposal with `status: Sent`, when he taps Accept, then the eligibility check passes and the accept proceeds to FEAT-03.SPEC-003.

**FEAT-03.SPEC-005-AC-02:** Given a proposal already has `status: Accepted`, when Owen (or any Primary contact) attempts to accept it again, then the attempt is denied with "This proposal has already been accepted."

**FEAT-03.SPEC-005-AC-03:** Given a proposal has `status: Voided`, when Owen attempts to accept it, then the attempt is denied and he is redirected to the current proposal version, with no error text shown.

**FEAT-03.SPEC-005-AC-04:** Given Owen submits a change-request note of exactly 2,000 characters, when the length rule is checked, then the note passes validation.

**FEAT-03.SPEC-005-AC-05:** Given Owen submits a change-request note of exactly 1 character, when the length rule is checked, then the note passes validation.

**FEAT-03.SPEC-005-AC-06:** Given Owen submits an empty change-request note, when the length rule is checked, then the note is denied with "Enter a note before sending."

**FEAT-03.SPEC-005-AC-07:** Given a proposal has `status: Voided`, when Owen attempts to submit a change-request note, then the attempt is denied and he is directed to the current proposal version.

**FEAT-03.SPEC-005-AC-08:** Given a proposal has `status: Accepted`, when Owen attempts to submit a change-request note, then the attempt is denied with "This proposal has already been accepted."

**FEAT-03.SPEC-005-AC-09:** Given Owen (Client Primary Contact) at the owning client, when he attempts to view the proposal, then he can view the full screen.

**FEAT-03.SPEC-005-AC-10:** Given Priya (Client Reviewer Contact), when she attempts to view the proposal, then no proposal content is shown and her portal home shows only the project's stage label.

**FEAT-03.SPEC-005-AC-11:** Given Nadia (Freelancer), when she attempts to open this feature's proposal review screen link, then the link is treated as out-of-scope and she sees a plain explanation with a fresh-link option, never proposal content through this feature's own screen.

**FEAT-03.SPEC-005-AC-12:** Given Dana (Support Operator) inside a logged support session, when she views the proposal, then she sees the full content read-only, with no Accept or Request Changes control rendered.

**FEAT-03.SPEC-005-AC-13:** Given two Primary contacts at the same client both attempt Accept at effectively the same moment, when the write-time eligibility check runs for each, then exactly one succeeds and the other is denied with "This proposal has already been accepted."

**FEAT-03.SPEC-005-AC-14:** Given a Primary contact's status is changed to Removed while they are mid-session on the proposal screen, when they attempt to accept, then the action is denied with the same experience as a contact who never had access.

**FEAT-03.SPEC-005-AC-15:** Given a proposal transitions from Sent to Voided between the screen's own eligibility check and the write-time re-check, when FEAT-03.SPEC-003 re-checks eligibility, then the voided-cannot-accept rule denies the write and the client is redirected to the current version.

**FEAT-03.SPEC-005-AC-16:** Given a contact attempts to reach a proposal belonging to a different client than their own, when the access check runs, then the attempt is denied as out-of-scope with a plain explanation and a fresh-link option, never another company's data.

**FEAT-03.SPEC-005-AC-17:** Given the acceptance write succeeds for one Primary contact's attempt, when `accepted_at` and `accepted_by` are derived, then they are set exactly once from that attempt's timestamp and contact reference, with no user override available.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 12 | 12 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 7 | 7 |
| Edge Cases | 8 | 8 |
