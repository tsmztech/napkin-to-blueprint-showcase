---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-05.SPEC-006
spec_name: Link Validity & Recognition Rules
spec_slug: link-validity-recognition-rules
parent_feature: FEAT-05
parent_feature_name: Client Portal Access (Magic-Link Login)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 19
acceptance_criteria_count: 13
---

# Logic/Rule Spec: Link Validity & Recognition Rules

## Overview

**Name:** Link Validity & Recognition Rules
**ID:** FEAT-05.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs single-use and time-limited link enforcement, invalidation on re-request, and that only recognized, Active contacts can obtain or use a link.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)
**Governed Entity:** Sign-in link/token (feature-internal, not a Domain Entity Inventory record)

## Scope and Non-Goals

**In Scope:**
- Field-level rules for the sign-in token: how it is generated, when it expires, and what makes it valid to consume
- The invalidate-prior-link rule triggered by a re-request
- Recognized-contact enforcement: which submitted emails result in a token at all
- Authorization for who may request and who may use a link
- The no-enumeration rule: identical outward behavior whether or not an email is recognized

**Non-Goals:**
- Client isolation and role-scoped portal display once a session exists -- owned by FEAT-05.SPEC-007 (Portal Access & Isolation Rules); this spec governs the token's own lifecycle, not what a resulting session may see.
- The screens that collect an email or show a verification outcome -- owned by FEAT-05.SPEC-001 and FEAT-05.SPEC-002, which enforce these rules but do not define them.
- The processing steps that generate, consume, and act on a token -- owned by FEAT-05.SPEC-004 (Magic Link Issuance) and FEAT-05.SPEC-005 (Magic Link Verification); this spec defines the rules those automations enforce.
- Client Contact creation, role assignment, or removal -- owned by FEAT-18 per the Entity-Lifecycle Coverage Matrix; this spec only reads a contact's current `status` to decide recognition.

## Governed Entity

**Entity:** Sign-in link/token
**Source:** Feature Breakdown Brief, Shared Context ("Sign-in link/token (feature-internal, not a Domain Entity Inventory record) -- created by FEAT-05.SPEC-004, consumed exactly once by FEAT-05.SPEC-005, governed end to end by FEAT-05.SPEC-006")

| Field | Data Type | Description |
|-------|-----------|-------------|
| token_value | text | The single-use credential embedded in the emailed link |
| client_contact | text (reference) | The Client Contact record this token authenticates |
| issued_at | date (timestamp) | When the token was generated |
| expires_at | derived | `issued_at` + platform parameter: `magic-link-expiry-window` |
| status | enum | Issued (unused, unexpired), Used, Expired, Invalidated (superseded by a later re-request) |
| used_at | date (timestamp) | When the token was marked Used, if applicable |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-05.SPEC-001 | Request Sign-In Link | Email-format validation on blur and submit; no-enumeration confirmation on every submit |
| FEAT-05.SPEC-002 | Link Verification Landing | Displays the outcome of validity checks identically regardless of which rule failed |
| FEAT-05.SPEC-004 | Magic Link Issuance | Recognition check, prior-token invalidation, and new-token generation during processing |
| FEAT-05.SPEC-005 | Magic Link Verification | Token-state checks (unused, unexpired, not invalidated) and contact-Active check during processing |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Submitted email (FEAT-05.SPEC-001 input) | Must be a validly formatted email address | Always | On blur, on submit | "Enter a valid email address" | Yes |
| token_value | Generated as a single, unguessable value per issuance -- no validation rule accepts a caller-supplied value | Always (system-generated only) | At generation | N/A -- never user input | N/A |
| client_contact | No validation beyond data type -- set internally to the recognized, Active Client Contact resolved by FEAT-05.SPEC-004; never user-editable | Always | At generation | N/A -- never user input | N/A |
| issued_at | No validation beyond data type -- system-set to the current timestamp at generation | Always | At generation | N/A -- never user input | N/A |
| expires_at | Always exactly `issued_at` + platform parameter: `magic-link-expiry-window`; not user-configurable | Always | At generation | N/A -- never user input | N/A |
| status | Must transition only Issued -> Used, Issued -> Expired, or Issued -> Invalidated; never backward | Always | At every state check | N/A -- internal state, no user-facing error | Yes (enforced by the automations, not surfaced as a field error) |
| used_at | No validation beyond data type -- system-set by FEAT-05.SPEC-005 at the moment a token is marked Used; remains empty for a token never used | On use | At verification (FEAT-05.SPEC-005) | N/A -- never user input | N/A |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Single valid token per contact | client_contact, status | At most one token with status Issued may exist for a given client_contact at any time; issuing a new one first invalidates any existing Issued token for that contact | N/A -- enforced silently as part of issuance; no error is shown to the requester (the neutral confirmation applies regardless) |
| Expiry precedes use | expires_at, used_at, status | A token whose current time has passed expires_at cannot transition to Used, even if otherwise unused; the check evaluates status as Expired instead | N/A -- surfaced to the contact only as the shared "not valid anymore" explanation on FEAT-05.SPEC-002 |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Request a sign-in link (submit an email) | Owen, Priya (any recognized, Active Client Contact) | Always -- the request screen accepts any email as input | N/A -- there is no denial state on request; an unrecognized or inactive email is accepted by the form and simply produces no token (No Match outcome, FEAT-05.SPEC-004), shown as the identical "Check your email" confirmation, never a distinct denial |
| Request a sign-in link | Nadia (Freelancer) | Never as a recognized client contact -- Nadia holds no Client Contact record of her own | Nadia's own email submission is processed exactly like any unrecognized email: the same neutral confirmation is shown and no link is issued |
| Request a sign-in link | Dana (Support Operator) | Never -- Dana holds no Client Contact record (SC-04) | Same neutral confirmation as any unrecognized email; no link is issued |
| Use (click) a valid, unused, unexpired token | Owen, Priya (the specific recognized, Active Client Contact the token is bound to) | Token must be Issued (not Used, Expired, or Invalidated) and its bound contact must be Active | The plain "This link isn't valid anymore" explanation on FEAT-05.SPEC-002, with a one-tap re-request; identical for every disqualifying reason |
| Use a token bound to a different contact than the one clicking it | Nobody | Never -- a token authenticates exactly the Client Contact it was generated for; there is no "use on behalf of" path | Same "not valid anymore" explanation; the token simply does not resolve to a usable session for anyone other than its bound contact, since possession of the link (not a separate identity check) is the only claim a browser can make, and an already-used or expired token fails identically for any holder |
| Re-request a link while a prior unused token exists | Owen, Priya (the contact who holds the prior token) | Always, per the recognized-contact rule above | N/A -- re-requesting is never denied; it always invalidates the prior token per the Cross-Field Rule above |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| token_value | System-generated, single, unguessable value | On create (issuance) | No |
| issued_at | Current timestamp | On create (issuance) | No |
| expires_at | issued_at + platform parameter: `magic-link-expiry-window` | On create (issuance), fixed thereafter | No |
| status | Issued | On create (issuance) | No -- transitions only through the automations (Used by FEAT-05.SPEC-005, Expired by time passing, Invalidated by FEAT-05.SPEC-004 on re-request) |

## Business Rules

- XBR-28: magic links are single-use and time-limited; requesting a new link invalidates earlier unused ones; only contacts added through Client Contact Management & Roles (FEAT-18) are recognized. This spec is the rule's owning authority per the dependency map.
- The recognized-contact check and the no-enumeration confirmation together mean this feature never reveals, through timing, wording, or any other observable difference, whether a submitted email belongs to a client contact of any freelancer on the platform.
- A token's validity is evaluated fresh at the moment of use (FEAT-05.SPEC-005), never cached from issuance time -- a token that was valid when emailed but has since expired, been used, or been invalidated by re-request fails at click time, not before.
- Invalidating a prior token (on re-request) is a state transition (Issued -> Invalidated), never a deletion -- the record persists so a subsequent click against it resolves to the same shared "not valid anymore" explanation rather than an unknown-token error, keeping the contact's experience identical either way.

## Edge Cases

- **Contact requests a link, lets it expire, then requests again** -- The expired token remains status Expired (it was never re-requested against, so no separate Invalidated transition applies); the new token is Issued fresh with its own full expiry window.
- **Contact requests a link twice within the same instant that a still-forming first token has not yet reached Issued status** -- Per FEAT-05.SPEC-004's concurrency handling, whichever request's invalidation step runs last determines the single surviving Issued token; no scenario leaves two simultaneously Issued tokens for one contact.
- **Token is at the exact expiry boundary (used at precisely expires_at)** -- Treated as expired; the boundary itself is not valid, since "time-limited" means strictly before expires_at.
- **A recognized contact's status changes from Active to Removed between issuance and click** -- The click-time Authorization Rules check re-evaluates `status` at use, not at issuance, so a Removed contact's still-unused token fails per the "Use a valid token" row's Active-contact condition.
- **A person holds Client Contact records under two different freelancers with the same email** -- Each freelancer's token is entirely independent under this spec's rules: invalidating or using one contact's token has no effect on the other's, per the Cross-Field Rule's scoping to a single client_contact.
- **Submitted email matches a contact record exactly except for case or whitespace** -- Recognition matching normalizes case and trims whitespace before comparison (consistent with FEAT-05.SPEC-004's processing logic), so the rule set treats these as the same email.

## Acceptance Criteria

**FEAT-05.SPEC-006-AC-01:** Given Owen submits a validly formatted, recognized, Active email, when the request is processed, then a new token is issued with status Issued and an expiry of platform parameter: `magic-link-expiry-window` from now.

**FEAT-05.SPEC-006-AC-02:** Given Priya submits an email matching no Client Contact, when the request is processed, then no token is created and she sees the identical confirmation as a recognized submission.

**FEAT-05.SPEC-006-AC-03:** Given Dana (Support Operator) submits her own email, when the request is processed, then no token is issued, since she holds no Client Contact record, and she sees the same neutral confirmation.

**FEAT-05.SPEC-006-AC-04:** Given Owen has an Issued, unused token, when he requests a new link, then the prior token transitions to Invalidated before the new token is issued.

**FEAT-05.SPEC-006-AC-05:** Given Owen's token's expires_at has passed and it was never used, when he clicks it, then it is treated as Expired and he sees "This link isn't valid anymore."

**FEAT-05.SPEC-006-AC-06:** Given Priya clicks a token exactly at its expiry instant, when verification checks it, then it is treated as expired (the boundary itself does not verify).

**FEAT-05.SPEC-006-AC-07:** Given Owen's Client Contact status changes to Removed after his token was issued but before he clicks it, when he clicks the still-unused, unexpired token, then it fails per the Active-contact condition and he sees the plain explanation.

**FEAT-05.SPEC-006-AC-08:** Given a person is a Client Contact for two different freelancers with the same email, when they request a link, then each freelancer issues and governs its own token independently.

**FEAT-05.SPEC-006-AC-09:** Given Owen submits his email with different letter casing and surrounding whitespace than stored, when the request is processed, then it still matches his Client Contact record.

**FEAT-05.SPEC-006-AC-10:** Given Priya's token has already transitioned to Used, when she clicks the same link again, then it is treated as invalid and she sees the plain explanation, never a distinct "already signed in" message that would confirm the token had once been valid.

**FEAT-05.SPEC-006-AC-11:** Given Owen requests a link and lets it expire without ever re-requesting, when he later requests a fresh link, then the fresh token is Issued with a full new expiry window, independent of the expired one.

**FEAT-05.SPEC-006-AC-12:** Given Priya's token is Invalidated by a later re-request, when she clicks the invalidated (not the new) link, then she sees the same plain explanation as an expired or used link.

**FEAT-05.SPEC-006-AC-13:** Given Nadia (Freelancer) submits her own sign-in email on the client portal's request screen, when the request is processed, then no token is issued to her, since she holds no Client Contact record, and she sees the same neutral confirmation as any other submission.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 7 | 7 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
