---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-02.SPEC-010
spec_name: Proposal Validation & Business Rules
spec_slug: proposal-validation-business-rules
parent_feature: FEAT-02
parent_feature_name: Proposal Creation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 24
acceptance_criteria_count: 22
---

# Logic/Rule Spec: Proposal Validation & Business Rules

## Overview

**Name:** Proposal Validation & Business Rules
**ID:** FEAT-02.SPEC-010
**Type:** Logic/Rule
**Purpose:** Governs required fields, price positivity and currency, the one-active-proposal-per-project cap, send eligibility (XBR-07), immutability after send and after acceptance (XBR-04), and the accept-vs-void contention resolution for the Proposal entity.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending
**Governed Entity:** Proposal

## Scope and Non-Goals

**In Scope:**
- Field-level validation for every Proposal field this feature writes
- The one-active-proposal-per-project cap
- Send eligibility, including the client Primary-contact requirement (XBR-07)
- Post-send and post-acceptance immutability (XBR-04) and the edit-eligibility rule that enforces it
- Authorization for every action on the Proposal entity across all four Access Matrix roles
- Default values and derived fields on the Proposal record
- The accept-vs-void contention rule as it applies from this feature's side (the edit path); FEAT-03 owns the acceptance-side half of the same rule

**Non-Goals:**
- The mechanics of voiding and creating a new version -- owned by FEAT-02.SPEC-006 (Void & Resend); this spec defines the eligibility gate that automation checks, not the versioning steps themselves.
- The acceptance action itself and its own validation (a proposal can be accepted exactly once, a voided proposal cannot be accepted) -- owned by FEAT-03 (Proposal Acceptance); this spec only defines how the Proposal entity's fields and status constrain what FEAT-02's own actions may do.
- Electronic-signature validation for the accept action -- owned by FEAT-26 (Legally Binding E-Signature for Proposals, v1); this spec's Governed Entity table lists the signature record field only for completeness of the field inventory.

## Governed Entity

**Entity:** Proposal
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| scope_description | text | The work scope the freelancer is proposing |
| price | number | The proposed amount, in the project's currency |
| currency | text (derived) | The project's billing currency (FEAT-15); not set independently on the Proposal |
| payment_schedule_reference | reference | Link to the project's Payment Schedule (FEAT-04); locked at the moment of send |
| status | enum | Draft, Sent, Voided, or Accepted |
| sent_at | date/time | Timestamp of the most recent send for the current version |
| copied_from | reference (optional) | The earlier proposal this Draft was started from, if any (FEAT-02.SPEC-008) |
| accepted_at | date/time (optional) | Timestamp of acceptance, written once by FEAT-03, never altered |
| accepted_by | reference (optional) | The accepting Client Contact, written once by FEAT-03, never altered |
| signature record | composite (optional, v1) | Signer identity, signature data, and timestamp, written by FEAT-26 when e-signature is enabled for the proposal |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-02.SPEC-001 | Proposal Draft Editor | Field validation on blur and on Preview/Save Draft/Save & Resend; edit-eligibility check on entering edit mode and on Save & Resend |
| FEAT-02.SPEC-002 | Proposal Preview | Send-eligibility re-check on Send tap |
| FEAT-02.SPEC-003 | Proposal Detail | Authorization rules on screen entry (status-driven action set) |
| FEAT-02.SPEC-005 | Proposal Send | Field validation and send-eligibility checks during processing |
| FEAT-02.SPEC-006 | Proposal Edit-Before-Acceptance Void & Resend | Field validation, send-eligibility, and the post-acceptance immutability check during processing |
| FEAT-02.SPEC-009 | Discard Draft | Draft-only eligibility check during processing |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| scope_description | Required, non-empty | Always | On blur and on submit | "Scope description is required" | Yes |
| scope_description | Maximum 10,000 characters | Always | On blur and on submit | "Scope description must be 10,000 characters or fewer" | Yes |
| price | Required, non-empty | Always | On blur and on submit | "Price is required" | Yes |
| price | Must be a positive amount (greater than zero) | Always | On blur and on submit | "Price must be a positive amount" | Yes |
| currency | No validation beyond data type -- always derived from the project, never entered directly | Always | -- | -- | -- |
| payment_schedule_reference | No validation beyond data type -- a Draft may reference no schedule yet; only locked (not required) at send | Always | -- | -- | -- |
| status | No validation beyond data type -- transitions are governed by the Business Rules and Authorization Rules below, not by field-level input validation | Always | -- | -- | -- |
| sent_at | No validation beyond data type -- system-set, never user-entered | Always | -- | -- | -- |
| copied_from | No validation beyond data type -- system-set, never user-entered | Always | -- | -- | -- |
| accepted_at | No validation beyond data type -- written once by FEAT-03, never entered or altered here | Always | -- | -- | -- |
| accepted_by | No validation beyond data type -- written once by FEAT-03, never entered or altered here | Always | -- | -- | -- |
| signature record | No validation beyond data type -- owned by FEAT-26; out of scope for this spec | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Currency follows the project | price, currency | price is always interpreted and displayed in currency, which is always the owning project's currency (FEAT-15) and can never be set independently on the Proposal | N/A -- currency is never user-entered on this entity, so no error state exists for a mismatch |
| One active proposal per project | status, project reference | A project may have at most one Proposal in Draft, Sent, or Accepted status at any time; a new Draft (blank or from copy) cannot be created while one already exists | "This project already has a proposal. Open it from the project instead." |
| Send requires a client Primary contact | status, payment_schedule_reference (n/a to this rule directly, listed for completeness), client's Primary contact roster | A Draft cannot transition to Sent (directly or via void-and-resend) unless the owning client has at least one Primary contact (XBR-07) | "This client has no Primary contact yet." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create proposal (blank or from copy) | Nadia (Freelancer) | Only when the target project has no active (Draft, Sent, or Accepted) proposal | Create action is not offered; attempting it via the reuse path shows "This project already has a proposal. Open it from the project instead." |
| Create proposal | Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Never | No route to create a proposal exists in the client portal or the support session; the action is never shown |
| View proposal (full content: scope, price, status, history) | Nadia (Freelancer) | Always, for any of her own proposals | -- |
| View proposal (full content) | Owen (Client Primary Contact) | Own-only -- only proposals belonging to his own client company | A proposal outside Owen's own client company is never reachable; an out-of-scope link shows the plain explanation defined by XBR-09 |
| View proposal | Priya (Client Reviewer Contact) | Never -- Priya has no access to proposal content at all, per the Access Matrix | Priya's portal home shows only the project's stage (e.g., "Proposal accepted"), never the proposal's scope or price |
| View proposal (status and history only, no scope/price/request-changes text) | Dana (Support Operator) | Always, inside a logged, read-only support session (FEAT-31) | -- |
| Edit proposal (Draft, in place) | Nadia (Freelancer) | Only while status is Draft | Edit controls are not shown once status leaves Draft |
| Edit proposal (Sent-but-unaccepted, via void-and-resend) | Nadia (Freelancer) | Only while status is Sent (not yet Accepted or Voided) | If status is Accepted: "This proposal has already been accepted and can no longer be edited." (XBR-04). If status is already Voided by a concurrent edit: "This proposal was already edited and resent. Review the current version." |
| Edit proposal | Owen, Priya, Dana | Never | No edit control is ever shown to these roles |
| Send proposal | Nadia (Freelancer) | Only while status is Draft, and only when the owning client has at least one Primary contact (XBR-07) | If status is not Draft: "This proposal has already been sent. Viewing the current version." If no Primary contact: "This client has no Primary contact yet." |
| Send proposal | Owen, Priya, Dana | Never | No send control is ever shown to these roles |
| Resend proposal (unchanged link) | Nadia (Freelancer) | Only while status is Sent, subject to the resend rate limit (FEAT-02.SPEC-007) | If status is not Sent: the Resend action is not shown. If rate-limited: "You already resent this proposal recently. Try again in a few minutes." |
| Resend proposal | Owen, Priya, Dana | Never | No resend control is ever shown to these roles |
| Discard proposal | Nadia (Freelancer) | Only while status is Draft | Discard control is not shown once status leaves Draft; a direct attempt against a non-Draft status shows "This proposal was just sent and can no longer be discarded." |
| Discard proposal | Owen, Priya, Dana | Never | No discard control is ever shown to these roles |
| Accept proposal (owned by FEAT-03) | Owen (Client Primary Contact) | Own-only, only while status is Sent, exactly once per proposal | If status is Voided or already Accepted: refused per FEAT-03's rules, with the current version shown |
| Accept proposal | Nadia, Priya, Dana | Never | No accept control is ever shown to these roles |
| Request changes (owned by FEAT-03) | Owen (Client Primary Contact) | Own-only, only while status is Sent | No request-changes control is ever shown to these roles when status is not Sent |
| Request changes | Nadia, Priya, Dana | Never | No request-changes control is ever shown to these roles |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status | "Draft" | On create (blank or from copy) | No -- always starts Draft |
| currency | The owning project's current currency (FEAT-15) | Always (on create and whenever displayed) | No -- never editable on the Proposal itself |
| payment_schedule_reference | The owning project's current Payment Schedule, or unset if none exists yet | On create; re-derived (re-read) on every save while status is Draft | No -- locked automatically at send (FEAT-02.SPEC-005) and never user-set directly |
| sent_at | Current date and time | On the Draft -> Sent transition (send or void-and-resend) | No |
| copied_from | The source proposal's id | On create, only when created via FEAT-02.SPEC-008 | No -- permanent provenance reference |
| accepted_at, accepted_by | Set once by FEAT-03 on acceptance | On acceptance | No -- written once, never altered (XBR-04) |

## Business Rules

- A project has at most one active (Draft, Sent, or Accepted) proposal at any time; Voided versions do not count against this cap, since a Voided version's replacement is always the new active one.
- A Sent-but-unaccepted proposal is never updated in place -- every content edit after send produces a new version through FEAT-02.SPEC-006's void-and-resend, never a direct field update (XBR-06).
- An Accepted proposal is immutable in every respect -- no field on it may ever be changed after acceptance, and no edit attempt against it succeeds, regardless of source (XBR-04).
- A Voided proposal is immutable and terminal -- it can never be accepted, edited, resent, or reactivated; it exists solely as evidence.
- Send eligibility (XBR-07) requires the client to have at least one Primary contact; this is re-checked at the moment of send and at the moment of void-and-resend, not only when the client roster was last viewed.
- Accept-vs-void contention: if Owen accepts a version that Nadia voided in the meantime (or is in the process of voiding), the acceptance is refused and Owen is shown the current version -- this rule is jointly enforced, with FEAT-03 owning the acceptance-side refusal and this spec owning the edit-side effect (a successful void makes the prior version's status Voided, which FEAT-03's acceptance check then reads as ineligible). Acceptance is recorded exactly once per proposal.
- Discard permanently removes only Draft-status records; a discarded Draft leaves no trace beyond an Activity Log Entry recording that the discard occurred (FEAT-13), since a never-sent Draft carries no evidentiary content worth retaining.
- Dana (Support Operator) never sees scope_description, price, or request-changes note text on any Proposal, in any status, consistent with the Access Matrix's read-only, content-limited support boundary (ASMP-18, ASMP-23).

## Edge Cases

- **Price entered as exactly zero** -- Fails the positive-amount rule; "Price must be a positive amount" is shown. Zero is not treated as a valid free-of-charge proposal in this product definition.
- **Scope description at exactly 10,000 characters** -- Passes validation. 10,001 characters shows the length error.
- **A Draft is created for a project, then its only active proposal is discarded, then a second Draft is created for the same project** -- Allowed; the one-active-proposal cap only ever compares against the project's current active proposal, and a discarded Draft is not counted once removed.
- **Nadia attempts to send a Draft while the client's Primary contact was removed moments earlier** -- The send-eligibility check re-reads the client's current contact roster at the moment of send, so the block applies even though the Draft itself was created while a Primary contact existed.
- **Nadia attempts to edit a Sent-but-unaccepted proposal in the instant Owen's acceptance is being recorded** -- Whichever transition (the acceptance or the edit's status check) completes first determines the outcome: if acceptance completes first, the edit is refused as already-accepted; if the edit's void completes first, the acceptance is refused as against-a-voided-version. Exactly one of the two prevails; no partial or contradictory state (both Accepted and Voided) is ever produced.
- **A Voided proposal's own copied_from or predecessor content is referenced from the Reuse Proposal Picker (FEAT-02.SPEC-004)** -- Permitted: a Voided proposal's last-sent content remains valid as a copy source even though the proposal itself is immutable and terminal, since copying reads content rather than reactivating the record.

## Acceptance Criteria

**FEAT-02.SPEC-010-AC-01:** Given Nadia leaves scope_description empty on a Draft, when she attempts to save or send, then "Scope description is required" is shown and the operation is blocked.

**FEAT-02.SPEC-010-AC-02:** Given Nadia enters a scope description of exactly 10,000 characters, when she saves, then validation passes; at 10,001 characters, "Scope description must be 10,000 characters or fewer" is shown.

**FEAT-02.SPEC-010-AC-03:** Given Nadia leaves price empty, when she attempts to save or send, then "Price is required" is shown.

**FEAT-02.SPEC-010-AC-04:** Given Nadia enters a price of zero, when she attempts to save or send, then "Price must be a positive amount" is shown.

**FEAT-02.SPEC-010-AC-05:** Given Nadia enters a price of 1.00 in the project's currency, when she saves, then validation passes.

**FEAT-02.SPEC-010-AC-06:** Given a project already has an active (Draft, Sent, or Accepted) proposal, when Nadia attempts to start a new Draft for it via the reuse path, then "This project already has a proposal. Open it from the project instead." is shown and no second Draft is created.

**FEAT-02.SPEC-010-AC-07:** Given the client has no Primary contact, when Nadia attempts to send a valid Draft, then "This client has no Primary contact yet." is shown and the send is blocked.

**FEAT-02.SPEC-010-AC-08:** Given the client has at least one Primary contact and the Draft's fields are valid, when Nadia sends it, then the send proceeds.

**FEAT-02.SPEC-010-AC-09:** Given a proposal's status is Accepted, when Nadia attempts to edit it, then "This proposal has already been accepted and can no longer be edited." is shown and no edit occurs.

**FEAT-02.SPEC-010-AC-10:** Given a proposal's status is Sent and unaccepted, when Nadia edits and saves it, then the edit is allowed through the void-and-resend path (FEAT-02.SPEC-006), not a direct update.

**FEAT-02.SPEC-010-AC-11:** Given a proposal's status is Voided, when any attempt is made to accept it, then the acceptance is refused (owned by FEAT-03) and the current version is shown instead.

**FEAT-02.SPEC-010-AC-12:** Given Nadia (Freelancer) views any of her own proposals, then she sees full content (scope, price, status, history) regardless of status.

**FEAT-02.SPEC-010-AC-13:** Given Owen (Client Primary Contact) attempts to reach a proposal belonging to a different client company, then no route exists and, if an out-of-scope link is followed, the plain explanation defined by XBR-09 is shown.

**FEAT-02.SPEC-010-AC-14:** Given Priya (Client Reviewer Contact) views her portal home, then she sees only the project's stage and never the proposal's scope or price.

**FEAT-02.SPEC-010-AC-15:** Given Dana (Support Operator) views a proposal inside a logged support session, then she sees status and history only, with no scope_description, price, or request-changes note text.

**FEAT-02.SPEC-010-AC-16:** Given a proposal's status is Draft, when Nadia looks for Discard, then it is shown and, on confirmation, the proposal is permanently deleted.

**FEAT-02.SPEC-010-AC-17:** Given a proposal's status is Sent, when Nadia looks for Discard, then it is not shown.

**FEAT-02.SPEC-010-AC-18:** Given Owen, Priya, or Dana looks for Create, Edit, Send, Resend, or Discard controls on any proposal, then none of these controls are ever shown to them.

**FEAT-02.SPEC-010-AC-19:** Given a new blank or copied Draft is created, then its status defaults to Draft and its currency is derived from the owning project, never independently settable.

**FEAT-02.SPEC-010-AC-20:** Given a proposal transitions from Draft to Sent, then sent_at is set to the current date and time and payment_schedule_reference is locked to the project's Payment Schedule at that moment.

**FEAT-02.SPEC-010-AC-21:** Given Owen's acceptance and Nadia's void-and-resend race against the same Sent proposal, when one completes first, then exactly one of "acceptance recorded" or "version voided" prevails and the other is refused -- never both.

**FEAT-02.SPEC-010-AC-22:** Given a proposal was created via FEAT-02.SPEC-008 from a source that is later voided, when Nadia views the new Draft, then copied_from still references the source and the new Draft remains fully editable and independent.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 10 (2 with sub-rules) | 10 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 19 | 19 |
| Defaults/Derivations | 6 | 6 |
| Business Rules | 8 | 8 |
| Edge Cases | 6 | 6 |
