---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-26.SPEC-003
spec_name: E-Signature Opt-In & Signing Access Rules
spec_slug: e-signature-opt-in-signing-access-rules
parent_feature: FEAT-26
parent_feature_name: Legally Binding E-Signature for Proposals
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 29
acceptance_criteria_count: 21
---

# Logic/Rule Spec: E-Signature Opt-In & Signing Access Rules

## Overview

**Name:** E-Signature Opt-In & Signing Access Rules
**ID:** FEAT-26.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs the per-proposal e-signature opt-in scope, who may sign, the routing decision between FEAT-03's plain Accept and this feature's signing step, the full-legal-name field rule, failed-submission retry, and immutability once signed.
**Parent Feature:** FEAT-26 -- Legally Binding E-Signature for Proposals
**Governed Entity:** Proposal (signing-enabled flag and signature-record fields); the routing decision between FEAT-03.SPEC-001 and FEAT-26.SPEC-001 is governed here as this spec's second concern, since the Feature Breakdown Brief groups it with the same Validation & Limits field.

## Scope and Non-Goals

**In Scope:**
- The per-proposal signing-enabled flag: who may set it and its scope (one proposal, not account-wide)
- Eligibility rules for signing (signing-enabled, sign-once, voided-cannot-sign)
- The routing rule that decides, each time a Primary contact opens a proposal, whether FEAT-03.SPEC-001's plain Accept or FEAT-26.SPEC-001's signing step is shown
- The full-legal-name field's validation rule
- Authorization rules for view and sign actions on the signing step, per role
- Failed-submission retry behavior (preserving entered data)
- Concurrent-action resolution for the signing-enabled flag and the signature record (reject-with-refresh), as it applies to this feature's actions

**Non-Goals:**
- Proposal creation, editing, and voiding rules -- owned entirely by FEAT-02 (Proposal Creation & Sending); this spec only reads the Proposal's current `status` and signing-enabled flag to determine eligibility for this feature's actions.
- Rendering the opt-in toggle control itself -- owned by FEAT-02's send/edit screen; this spec owns only the eligibility flag and the rules that govern it, per the Feature Breakdown Brief's Capability Coverage Map.
- The underlying single-acceptance and voided-proposal enforcement FEAT-03.SPEC-005 already owns for a plain Accept -- this spec defers to FEAT-03.SPEC-005 for that base enforcement rather than re-deriving it, and adds only the signing-specific rules that extend it (XBR-34).
- Selecting or configuring the electronic-signature attestation vendor -- vendor selection is a Stage 4 decision (FEAT-26.SPEC-005); this spec's eligibility rules apply regardless of which capability performs the attestation.

## Governed Entity

**Entity:** Proposal
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| status | enum (Draft, Sent, Voided, Accepted) | The proposal's current lifecycle state; this spec reads it to gate signing eligibility, deferring to FEAT-03.SPEC-005 for the base enforcement of this field |
| sent_at | date/time | When the proposal was sent; no validation rule beyond data type in this spec |
| accepted_at | date/time | Written once, by FEAT-26.SPEC-002 for a signed acceptance or by FEAT-03.SPEC-003 for a plain one, never altered; this spec defines the additional condition (signing-enabled) under which FEAT-26.SPEC-002 may write it |
| accepted_by | reference (Client Contact) | The accepting/signing contact's identity; this spec defines who may cause this field to be set through the signing path specifically |
| payment_schedule_reference | reference (Payment Schedule) | Link to the project's Payment Schedule; no validation rule beyond data type in this spec |
| signing-enabled flag | boolean | Set by Nadia while drafting or editing the proposal on FEAT-02's screen (before sending); this spec is the single source of truth for who may set it and its scope |
| signature record (signer identity, signature data, timestamp) | composite | Written once by FEAT-26.SPEC-002 when signing succeeds; this spec defines the eligibility condition (signing-enabled, sign-once, not-voided) that gates the write, and its immutability once written |

**Secondary governed field (routing decision, not a Proposal field):**

| Field | Data Type | Description |
|-------|-----------|--------------|
| routing outcome | derived | Which screen (FEAT-03.SPEC-001 or FEAT-26.SPEC-001) a Primary contact sees when opening a proposal that has not yet been acted on; derived from the signing-enabled flag, governed here per the Feature Breakdown Brief's Side-Effect Inventory |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|----------------------|
| FEAT-02 | Proposal Creation & Sending | Screen-level opt-in toggle reads and writes the signing-enabled flag governed here (rendered on FEAT-02's own screen, per the Feature Breakdown Brief's Capability Coverage Map) |
| FEAT-03.SPEC-001 | Proposal Review & Accept | Applies the routing rule on screen entry -- shows the plain Accept flow unchanged when the proposal is not signing-enabled, or hands off to FEAT-26.SPEC-001 when it is |
| FEAT-26.SPEC-001 | Signature Signing Step | Access and Visibility on screen entry; full-legal-name field validation on Sign tap and on change; failed-submission retry behavior (screen-level, non-authoritative) |
| FEAT-26.SPEC-002 | Signature Recording | Authoritative write-time re-check of signing eligibility (signing-enabled, sign-once, voided-cannot-sign) and of the actor's authorization (role still Primary, status still Active; returns the "denied -- not authorized" outcome) before any attestation submission or signature write |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| status | No validation beyond data type -- this spec reads it, FEAT-02 owns writing it (except the signed transition, governed below) | Always | -- | -- | -- |
| sent_at | No validation beyond data type | Always | -- | -- | -- |
| accepted_at | Must be set exactly once, only by FEAT-26.SPEC-002's write when signing-enabled, or FEAT-03.SPEC-003's write when not; never altered afterward | Proposal transitions Sent -> Accepted (via either path) | On write (FEAT-26.SPEC-002 or FEAT-03.SPEC-003) | N/A -- system-set field, no user-facing error; a second attempted write is intercepted by the sign-once rule below, not a field-level message | Yes |
| accepted_by | Must reference a Client Contact with role Primary and status Active at the client owning the proposal | Proposal transitions Sent -> Accepted (via either path) | On write (FEAT-26.SPEC-002 or FEAT-03.SPEC-003) | N/A -- system-set field; the gating condition surfaces as the authorization denial below, not a field-level message | Yes |
| payment_schedule_reference | No validation beyond data type | Always | -- | -- | -- |
| signing-enabled flag | May be set only while the proposal is in Draft (before sending); no validation beyond data type once set | Always | On write (FEAT-02) | N/A -- system-set field; FEAT-02 owns its own screen-level rules for when the toggle is shown | -- |
| signature record (signer identity, signature data, timestamp) | Must be set exactly once, only by FEAT-26.SPEC-002's write, and only when the signing-enabled flag is set; never altered afterward | Proposal transitions Sent -> Accepted via the signing path | On write (FEAT-26.SPEC-002) | N/A -- system-set field; the gating condition surfaces as the eligibility and authorization denials below | Yes |
| full legal name (entered on FEAT-26.SPEC-001, not a stored Proposal field) | Required, non-empty | Always, when signing | On blur (screen) and on write (FEAT-26.SPEC-002, authoritative) | "Enter your full legal name to sign." | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Sign-once | signature record, status | Signing may only proceed when no signature record yet exists and `status` is `Sent`; once a signature record exists, it is fixed and no further sign attempt may change it | "This proposal has already been signed." |
| Voided-cannot-sign | status | Signing may only proceed when `status` is `Sent`, never when `status` is `Voided` | N/A -- Owen is redirected to the current proposal version rather than shown an inline error (FEAT-26.SPEC-001) |
| Signing-enabled gates the signing path | signing-enabled flag, signature record | A signature record may be written only when the signing-enabled flag is set on that proposal version; when it is not set, the plain Accept path (FEAT-03.SPEC-003) is the only route to `accepted_at`/`accepted_by` | N/A -- the routing rule below determines which screen Owen reaches; there is no direct error path for this condition |
| Signing-enabled is per proposal, not account-wide | signing-enabled flag | The flag applies only to the specific proposal version it was set on; enabling it for one proposal never changes any other proposal's flag, current or future | N/A -- no error condition; this is a scoping rule, not a validation failure |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Set the signing-enabled flag for a proposal | Nadia (Freelancer) | Own-only, and only while the proposal is in Draft (before sending), on FEAT-02's screen | -- |
| Set the signing-enabled flag for a proposal | Owen (Client Primary Contact) | Never | The toggle is not rendered on any screen Owen can reach; he never sets his own signing requirement |
| Set the signing-enabled flag for a proposal | Priya (Client Reviewer Contact) | Never | The toggle is not rendered on any screen Priya can reach |
| Set the signing-enabled flag for a proposal | Dana (Support Operator) | Never | The toggle is not rendered inside a support session (FEAT-31); it is read-only there |
| View the signing step (scope, price, signature capture) | Owen (Client Primary Contact) | Own-only -- only his own company's signing-enabled proposal, and only when `status` is `Sent` or `Accepted` (XBR-08, XBR-09) | -- |
| View the signing step | Priya (Client Reviewer Contact) | Never | Full screen is not shown; portal home shows only the project's stage label, per the Access Matrix (Proposals & Acceptance: None for Reviewers) |
| View the signing step | Nadia (Freelancer) | Never through this feature's screens -- she sees the resulting "Signed on {date}" status on the project view (FEAT-01), not FEAT-26.SPEC-001 | Attempting to open FEAT-26.SPEC-001's link is treated as an out-of-scope link (XBR-09): plain explanation and a fresh-link option |
| View the signing step | Dana (Support Operator) | Always, read-only, inside a logged support session (FEAT-31) | -- |
| Sign the proposal | Owen (Client Primary Contact) | Own-only, and only when `status` is `Sent`, the signing-enabled flag is set, and no signature record yet exists (sign-once and voided-cannot-sign, above) | If a signature record already exists: "This proposal has already been signed." If `status` is `Voided`: redirected to the current version, no error text shown |
| Sign the proposal | Owen (Client Primary Contact) whose role has since changed to Reviewer or whose status has since become Removed | Never -- the Primary and Active condition is re-checked at the moment of the Sign action by FEAT-26.SPEC-002 (authoritative), not only at screen load | FEAT-26.SPEC-001 shows the Not authorized state: "You no longer have permission to sign this proposal. If you think this is a mistake, ask the person who sent it." with a "Go to portal home" link; no signature is recorded and no proposal content is shown |
| Sign the proposal | Priya (Client Reviewer Contact) | Never | The signature capture step is not shown -- she never reaches signing content |
| Sign the proposal | Nadia (Freelancer) | Never | No signature capture control exists on any screen Nadia can reach; signing is exclusively a client-side action, same as plain acceptance |
| Sign the proposal | Dana (Support Operator) | Never | The signature capture step is not rendered inside a support session (FEAT-31) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|---------------|----------------------|
| signing-enabled flag | Defaults to unset (not signing-enabled) for every new proposal | On proposal creation (FEAT-02) | Yes -- Nadia may set it while the proposal is in Draft, before sending (see Authorization Rules) |
| routing outcome | Derived as: show FEAT-26.SPEC-001 when the current proposal version's signing-enabled flag is set and the proposal is `Sent` or `Accepted` via the signing path; otherwise show FEAT-03.SPEC-001's plain Accept flow | Every time a Primary contact opens a proposal that has not yet been acted on, and on the redirect after a void-and-resend | No -- the routing outcome is fully derived from the current version's signing-enabled flag; neither Owen nor Nadia can override it once the version is sent |
| accepted_at (via the signing path) | Derived as the current timestamp at the moment FEAT-26.SPEC-002's write succeeds | On the Sent -> Accepted transition via signing, once | No |
| accepted_by (via the signing path) | Derived as the Client Contact reference of the eligible Primary contact whose sign attempt wins the exactly-once write (FEAT-26.SPEC-002) | On the Sent -> Accepted transition via signing, once | No |

## Business Rules

- XBR-34: When enabled for a proposal, a legally binding e-signature replaces the plain Accept click with the same Primary-only access and the same immutability; otherwise the timestamped Accept is the default. This spec is the single source of truth for the routing decision and eligibility that carries out that replacement.
- XBR-06: A voided proposal cannot be signed; a project has at most one active proposal at a time, and a voided proposal directs the client to the current one.
- XBR-08: Only Primary contacts sign proposals; Reviewer contacts never see proposal or signature content.
- XBR-09: Client isolation -- a contact reaches only their own company's proposal; an out-of-scope or expired link shows a plain explanation and a fresh-link option, never another company's data.
- Signing is available per proposal, not forced account-wide: a freelancer who never opts in never sees any change to FEAT-03's standard Accept flow, per the Feature Breakdown Brief's Validation & Limits field.
- Once signed, the signature record is immutable like any other acceptance (Feature Breakdown Brief, Validation & Limits): FEAT-26.SPEC-002 writes it exactly once, and no later process may alter it (XBR-04).
- If a signature submission fails mid-flight (attestation-capability error, validation failure, or connectivity drop), the entered full legal name is preserved on screen and Owen may retry without losing his intent, per the Feature Breakdown Brief's Side-Effect Inventory -- enforced within FEAT-26.SPEC-002's atomic write, per this spec's sign-once rule.
- The Proposal Contention resolution for a race between Nadia's edit-and-void (FEAT-02) and Owen's sign attempt (this feature) is reject-with-refresh: a sign attempt against a version voided in the meantime is refused and the client is shown the current version.
- Dana's support session sees this feature's screens read-only, per XBR-29: no signature capture control is ever rendered for her, regardless of the proposal's state.

## Edge Cases

- **Owen signs a proposal at the exact instant its `status` field is Sent, the signing-enabled flag is set, and no signature record yet exists** -- Standard eligible-sign path; no boundary issue since this is the normal signing-enabled Sent state.
- **Owen enters exactly one character as his full legal name** -- Passes validation; the rule requires only non-empty, no minimum length beyond that.
- **Owen leaves the full legal name field with only whitespace** -- Treated as empty for this rule's purpose and fails validation with "Enter your full legal name to sign.", mirroring the whitespace-only treatment FEAT-03.SPEC-005 applies to the change-request note.
- **Nadia enables signing on a proposal, then edits it again before sending, disabling and re-enabling the flag** -- No error condition; the flag simply reflects its state at the moment the proposal is finally sent, since it may only be set while the proposal is in Draft.
- **Two attempts to sign the same proposal arrive within the same instant** -- The sign-once rule is enforced atomically by FEAT-26.SPEC-002's write, not by this spec directly; this spec defines that exactly one may succeed and the other must see "This proposal has already been signed."
- **A Primary contact's role is changed to Reviewer, or their status is set to Removed, while they are mid-session on FEAT-26.SPEC-001** -- The authorization condition (Own-only, Primary, Active) is re-checked at the moment of the Sign action, not only at screen load; if the role or status has changed, FEAT-26.SPEC-002's write-time check returns "denied -- not authorized" and FEAT-26.SPEC-001 shows: "You no longer have permission to sign this proposal. If you think this is a mistake, ask the person who sent it." with a "Go to portal home" link. No attestation submission or signature write occurs, no proposal content remains on screen, and the contact's next access follows their current role (Reviewer: stage-only view; Removed: expired-link explanation with fresh-link option), consistent with XBR-27.
- **A signing-enabled proposal is voided and re-sent as a new version with the signing-enabled flag left unset on the new version** -- The routing rule re-evaluates against the current version each time: the redirect after the void lands Owen on FEAT-03.SPEC-001's plain Accept flow, not FEAT-26.SPEC-001, because the current version's own flag now governs the decision.
- **The proposal transitions from Sent to Voided between FEAT-26.SPEC-001's screen-level eligibility check and Owen's Sign tap reaching FEAT-26.SPEC-002** -- The screen-level check is non-authoritative; FEAT-26.SPEC-002's write-time re-check is what actually catches this and returns the voided-redirect outcome.
- **Owen attempts to sign a proposal for a project belonging to a different client than the one his contact record is scoped to** -- Denied per XBR-09: this is treated as an out-of-scope link, showing a plain explanation and a fresh-link option, never another company's proposal.

## Acceptance Criteria

**FEAT-26.SPEC-003-AC-01:** Given Nadia is drafting a proposal that has not yet been sent, when she enables the e-signature toggle on FEAT-02's screen, then the signing-enabled flag is set on that specific proposal only.

**FEAT-26.SPEC-003-AC-02:** Given a proposal has its signing-enabled flag set and status Sent, when Owen (a Primary contact) opens it, then the routing rule sends him to FEAT-26.SPEC-001 (Signature Signing Step) rather than FEAT-03.SPEC-001's plain Accept flow.

**FEAT-26.SPEC-003-AC-03:** Given a proposal does not have its signing-enabled flag set, when Owen opens it, then the routing rule shows FEAT-03.SPEC-001's plain Accept flow unchanged.

**FEAT-26.SPEC-003-AC-04:** Given Owen is on FEAT-26.SPEC-001 with the proposal signing-enabled and Sent, when he leaves the full legal name field empty and moves focus away, then the field shows "Enter your full legal name to sign."

**FEAT-26.SPEC-003-AC-05:** Given Owen enters a single-character full legal name, when the field rule is checked, then the name passes validation.

**FEAT-26.SPEC-003-AC-06:** Given Owen enters only whitespace as his full legal name, when the field rule is checked, then it is denied with "Enter your full legal name to sign."

**FEAT-26.SPEC-003-AC-07:** Given a proposal already has a signature record, when Owen (or any Primary contact) attempts to sign it again, then the attempt is denied with "This proposal has already been signed."

**FEAT-26.SPEC-003-AC-08:** Given a proposal has `status: Voided`, when Owen attempts to sign it, then the attempt is denied and he is redirected to the current proposal version, with no error text shown.

**FEAT-26.SPEC-003-AC-09:** Given Owen's signature submission fails mid-flight, when the failure occurs, then the entered full legal name is preserved on FEAT-26.SPEC-001 and he may retry without re-entering it.

**FEAT-26.SPEC-003-AC-10:** Given Owen (Client Primary Contact) at the owning client, when he attempts to view a signing-enabled proposal, then he can view the full signing step.

**FEAT-26.SPEC-003-AC-11:** Given Priya (Client Reviewer Contact), when she attempts to view a signing-enabled proposal, then no signing content is shown and her portal home shows only the project's stage label.

**FEAT-26.SPEC-003-AC-12:** Given Nadia (Freelancer), when she attempts to open FEAT-26.SPEC-001's link directly, then the link is treated as out-of-scope and she sees a plain explanation with a fresh-link option, never signing content through this feature's own screen.

**FEAT-26.SPEC-003-AC-13:** Given Dana (Support Operator) inside a logged support session, when she views a signing-enabled proposal, then she sees the full content read-only, with no signature capture control rendered.

**FEAT-26.SPEC-003-AC-14:** Given Nadia (Freelancer), when she attempts to set the signing-enabled flag herself outside FEAT-02's own screen, then no such control exists for her to attempt this with -- the flag is set only through FEAT-02's opt-in toggle.

**FEAT-26.SPEC-003-AC-15:** Given two Primary contacts at the same client both attempt to sign at effectively the same moment, when the write-time eligibility check runs for each, then exactly one succeeds and the other is denied with "This proposal has already been signed."

**FEAT-26.SPEC-003-AC-16:** Given a Primary contact's status is changed to Removed while they are mid-session on the signing step, when they attempt to sign, then FEAT-26.SPEC-002 denies the action and FEAT-26.SPEC-001 shows "You no longer have permission to sign this proposal. If you think this is a mistake, ask the person who sent it." with a "Go to portal home" link, and no signature is recorded.

**FEAT-26.SPEC-003-AC-21:** Given a Primary contact's role is changed to Reviewer while they are mid-session on the signing step, when they attempt to sign, then FEAT-26.SPEC-002's write-time check denies the action before any attestation submission, FEAT-26.SPEC-001 shows the same "You no longer have permission to sign this proposal." message, and the Proposal's signature record and `status` are unchanged.

**FEAT-26.SPEC-003-AC-17:** Given a proposal transitions from Sent to Voided between the screen's own eligibility check and the write-time re-check, when FEAT-26.SPEC-002 re-checks eligibility, then the voided-cannot-sign rule denies the write and the client is redirected to the current version.

**FEAT-26.SPEC-003-AC-18:** Given a contact attempts to reach a signing-enabled proposal belonging to a different client than their own, when the access check runs, then the attempt is denied as out-of-scope with a plain explanation and a fresh-link option, never another company's data.

**FEAT-26.SPEC-003-AC-19:** Given the signature write succeeds for one Primary contact's attempt, when `accepted_at` and `accepted_by` are derived, then they are set exactly once from that attempt's timestamp and contact reference, with no user override available.

**FEAT-26.SPEC-003-AC-20:** Given a signing-enabled proposal is voided and re-sent as a new version with the flag left unset, when Owen opens the current version, then the routing rule shows FEAT-03.SPEC-001's plain Accept flow, based on the current version's own signing-enabled flag.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 13 | 13 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 9 | 9 |
| Edge Cases | 9 | 9 |
