---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-21.SPEC-007
spec_name: Account Field Validation Rules
spec_slug: account-field-validation-rules
parent_feature: FEAT-21
parent_feature_name: Settings & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 12
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Account Field Validation Rules

## Overview

**Name:** Account Field Validation Rules
**ID:** FEAT-21.SPEC-007
**Type:** Logic/Rule
**Purpose:** Enforces required fields, format, and value limits across profile, business details, and payment terms fields on the Freelancer Account.
**Parent Feature:** FEAT-21 -- Settings & Account Management
**Governed Entity:** Freelancer Account

## Scope and Non-Goals

**In Scope:**
- Field-level validation for name, business name, business address, tax ID, and default payment terms
- The "not the same as current" check applied to a new sign-in email submission
- Authorization for every action this feature defines on the Freelancer Account

**Non-Goals:**
- The sign-in email's own format and re-verification process -- format is checked here as a field rule applied at submission time on FEAT-21.SPEC-003, but the re-verification workflow itself (send, confirm, expire, resend) is owned entirely by FEAT-21.SPEC-005
- Notification preference toggling rules -- owned by FEAT-21.SPEC-008 (Notification Preference Rules), a distinct rule set for a distinct part of the Freelancer Account
- Business details completeness-for-invoicing determination -- owned by FEAT-21.SPEC-009 (Business Details Completeness Gate), which consumes this spec's field rules but owns the aggregate completeness decision
- Validation of signed-in devices data -- the devices list is system-derived from active sessions, not user-entered data, so no field validation applies to it

## Governed Entity

**Entity:** Freelancer Account
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | Freelancer's own name, shown on her profile |
| sign-in email | text | Email used to sign in; changes require re-verification (FEAT-21.SPEC-005) |
| business_name | text | Name printed on invoices as the freelancer's business |
| business_address | text | Address printed on invoices |
| tax_id | text | Freelancer's tax identifier, printed on invoices when set |
| default_payment_terms | enum | Account-wide default due-date term applied to new invoices |
| time_zone | derived | Owned entirely by Currency & Tax Handling (FEAT-15); this feature exposes no editor for it (Brief Non-Goals) -- listed here only because it is a field on the governed entity, with no validation rule of this feature's own |
| notification_preferences | derived | Governed by FEAT-21.SPEC-008, not this spec |
| signed-in devices | derived | System-maintained list of active sessions; no user-entered validation applies |
| help-tip dismissals | derived | Captured for FEAT-30 (Later); this feature exposes no management surface for it (Brief Non-Goals) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-21.SPEC-001 | Account Profile | On field blur and form submit for `name`; authorization on screen entry and on save |
| FEAT-21.SPEC-003 | Login & Security | On submit for the new sign-in email's format and "not the same as current" check; authorization on screen entry |
| FEAT-21.SPEC-004 | Business Details & Payment Terms | On field blur and form submit for `business_name`, `business_address`, `tax_id`, `default_payment_terms`; authorization on screen entry and on save |
| FEAT-21.SPEC-005 | Sign-In Email & Login Method Change | Re-checks the "not the same as current" condition is still true immediately before starting the pending change |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | Required, non-empty, max 100 characters | Always | On blur / on submit | "Name is required" / "Name must be 100 characters or fewer" | Yes |
| sign-in email (new value on FEAT-21.SPEC-003) | Valid email format | Always | On submit | "Please enter a valid email address" | Yes |
| sign-in email (new value on FEAT-21.SPEC-003) | Must differ from the current sign-in email | Always | On submit | "This is already your sign-in email." | Yes |
| business_name | Required before the first invoice is sent (advisory: an empty value never blocks a save; it keeps completeness incomplete per FEAT-21.SPEC-009), max 200 characters | Always (required-before-invoicing per XBR-16); length limit always | On blur / on submit | "Business name is required before invoicing" (advisory) / "Business name must be 200 characters or fewer" | No for the required rule (advisory only); Yes for the length limit |
| business_address | Required before the first invoice is sent (advisory, non-blocking as above), max 500 characters | Always (required-before-invoicing per XBR-16); length limit always | On blur / on submit | "Business address is required before invoicing" (advisory) / "Business address must be 500 characters or fewer" | No for the required rule (advisory only); Yes for the length limit |
| tax_id | No length beyond 50 characters; no required-field error | Always | On blur | "Tax ID must be 50 characters or fewer" | Yes (length only; the field itself is optional) |
| default_payment_terms | Required before the first invoice is sent; must be one of the product's defined terms options ("Due on receipt" or "Net {N} days", where {N} is one of the product's fixed offered values) | Always (required-before-invoicing per XBR-16) | On selection / on submit | "Choose a default payment term before invoicing" (advisory) | No -- advisory only; an unselected value never blocks a save, it keeps completeness incomplete (FEAT-21.SPEC-009). A value outside the defined options is not selectable and is rejected (Yes) |
| time_zone | No validation beyond data type -- this feature never writes this field (owned by FEAT-15) | Always | -- | -- | -- |
| notification_preferences | No validation beyond data type in this spec -- see FEAT-21.SPEC-008 | Always | -- | -- | -- |
| signed-in devices | No validation beyond data type -- system-derived, never user-entered | Always | -- | -- | -- |
| help-tip dismissals | No validation beyond data type -- captured for FEAT-30 (Later), no management surface in this feature | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Business details required together before invoicing | business_name, business_address, default_payment_terms | All three must be non-empty/selected before FEAT-21.SPEC-009 marks business details complete (XBR-16); tax_id is not part of this set since it is optional | No blocking error on this screen -- saving with any of the three empty (including only tax ID filled) succeeds; the completeness indicator on FEAT-21.SPEC-004 (owned by FEAT-21.SPEC-009) reads "Business details are incomplete -- required before your first invoice can be sent." while any of the three is missing |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View own profile, business details, notification preferences | Nadia (Freelancer) | Always, her own account only | -- |
| Edit name (profile) | Nadia (Freelancer) | Always, her own account only | -- |
| Edit business_name, business_address, tax_id, default_payment_terms | Nadia (Freelancer) | Always, her own account only | -- |
| Start a sign-in email/login method change | Nadia (Freelancer) | Always, her own account only | -- |
| View profile, business details, notification preferences | Dana (Support Operator) | Read-only, inside a logged FEAT-31 support session (FEAT-21.SPEC-010) | -- |
| Edit any field on this entity | Dana (Support Operator) | Never | Save controls are not rendered for Dana on any Settings screen; a direct attempt is refused with "Support sessions are read-only." (FEAT-21.SPEC-010) |
| View or edit sign-in email/login method, signed-in devices | Dana (Support Operator) | Never | Login & Security (FEAT-21.SPEC-003) is never rendered inside a support session, and sign-in credentials are never visible to the operator under any circumstance (feature-dependency-map.md, Freelancer Account, Data Sensitivity) |
| View or edit any field on this entity | Owen (Client Primary Contact) | Never | Settings is not shown in navigation at all -- the Access Matrix gives Owen "None" on Branding, Onboarding & Settings |
| View or edit any field on this entity | Priya (Client Reviewer Contact) | Never | Settings is not shown in navigation at all -- the Access Matrix gives Priya "None" on Branding, Onboarding & Settings |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| name | Set at sign-up (FEAT-20) | On create only | Yes -- Nadia can edit at any time on FEAT-21.SPEC-001 |
| sign-in email | Set at sign-up (FEAT-20) | On create only | Yes -- only through the FEAT-21.SPEC-005 re-verification process, never a direct edit |
| business_name, business_address, tax_id, default_payment_terms | No default -- empty until Nadia fills them, unless captured during FEAT-20's guided setup | On create (if supplied during onboarding) or left empty | Yes -- Nadia can edit at any time on FEAT-21.SPEC-004 |
| time_zone | Set and derived by FEAT-15 | Always | No -- not overridable from this feature |

## Business Rules

- Every rule in this spec applies identically whenever the enforcing screen is open to Nadia -- there is no create-only or edit-only distinction, since the Freelancer Account is created once (FEAT-20) and only ever updated afterward by this feature.
- Business details completeness (XBR-16) is a cross-field, cross-feature outcome computed by FEAT-21.SPEC-009 from this spec's field rules; FEAT-09 checks that outcome at invoice-send time and blocks sending until it is true.
- The sign-in email format and "not the same as current" checks are evaluated at submission time on FEAT-21.SPEC-003; the actual change only takes effect once FEAT-21.SPEC-005's re-verification succeeds (dependency map, Freelancer Account Contention: "the sign-in email change, which requires re-verification before it takes effect").
- Authorization here is consistent with, and never overrides, FEAT-21.SPEC-010's account-wide read-only scope rules for Dana and total exclusion for client contacts.

## Edge Cases

- **Name field at exactly 100 characters** -- Passes validation. 101 characters shows the length error.
- **Business address at exactly 500 characters** -- Passes validation. 501 characters shows the length error.
- **Tax ID left as whitespace only** -- Treated as empty (optional field, no required-field error); trimmed before storage.
- **Nadia submits a new sign-in email that differs from her current one only by letter case (e.g., Nadia@Example.com vs. nadia@example.com)** -- Treated as the same email for the "not the same as current" check (email comparison is case-insensitive), so the submission is rejected with "This is already your sign-in email."
- **Default payment terms cleared after being previously set, then business_name and business_address remain filled** -- The save is not blocked (the required rule is advisory) and succeeds; business details completeness (FEAT-21.SPEC-009) reverts to incomplete, since all three required fields must be non-empty together.
- **Nadia's role or account state changes mid-edit (not possible in this single-role product beyond Nadia herself, but Dana's support session could open concurrently)** -- Dana's concurrently opened read-only session never gains edit controls regardless of timing; Nadia's own edit session is unaffected by a support session opening or closing.

## Acceptance Criteria

**FEAT-21.SPEC-007-AC-01:** Given Nadia clears the name field, when she blurs it, then she sees "Name is required."

**FEAT-21.SPEC-007-AC-02:** Given Nadia enters a name of exactly 100 characters, when she saves, then it is accepted; entering 101 characters shows "Name must be 100 characters or fewer."

**FEAT-21.SPEC-007-AC-03:** Given Nadia submits a malformed new sign-in email, when she submits the change form, then she sees "Please enter a valid email address."

**FEAT-21.SPEC-007-AC-04:** Given Nadia submits her current sign-in email (including a case-only difference) as the "new" one, when she submits, then she sees "This is already your sign-in email." and no pending change is created.

**FEAT-21.SPEC-007-AC-05:** Given Nadia leaves business_name empty and attempts to save Business Details & Payment Terms, when she saves, then she sees the advisory "Business name is required before invoicing." and the save still succeeds (non-blocking), leaving completeness incomplete.

**FEAT-21.SPEC-007-AC-06:** Given Nadia leaves business_address empty and attempts to save, then she sees the advisory "Business address is required before invoicing." and the save still succeeds, leaving completeness incomplete.

**FEAT-21.SPEC-007-AC-07:** Given Nadia does not select a default_payment_terms value and attempts to save, then she sees the advisory "Choose a default payment term before invoicing." and the save still succeeds, leaving completeness incomplete.

**FEAT-21.SPEC-007-AC-08:** Given Nadia leaves tax_id empty and saves with the other required fields complete, then the save succeeds with no required-field error for tax_id.

**FEAT-21.SPEC-007-AC-09:** Given Nadia enters a tax_id of 51 characters, when she blurs the field, then she sees "Tax ID must be 50 characters or fewer."

**FEAT-21.SPEC-007-AC-10:** Given Nadia has filled business_name, business_address, and default_payment_terms, then FEAT-21.SPEC-009 evaluates business details as complete (the cross-field rule's condition is satisfied).

**FEAT-21.SPEC-007-AC-11:** Given Nadia (Freelancer) is on any Settings screen, when she performs an edit action on her own account, then it is always allowed.

**FEAT-21.SPEC-007-AC-12:** Given Dana (Support Operator) is inside a logged support session, when she looks for any save control on any Settings screen, then none is shown, and a direct attempt to submit a change is refused with "Support sessions are read-only."

**FEAT-21.SPEC-007-AC-13:** Given Dana (Support Operator) is inside a logged support session, when she looks for the sign-in email or signed-in devices, then neither is ever shown to her.

**FEAT-21.SPEC-007-AC-14:** Given Owen (Client Primary Contact) attempts to reach any Settings screen, then he finds none in navigation and no direct access exists.

**FEAT-21.SPEC-007-AC-15:** Given Priya (Client Reviewer Contact) attempts to reach any Settings screen, then she finds none in navigation and no direct access exists.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 (name, email format, email-differs, business_name, business_address, tax_id, default_payment_terms, N/A fields noted) | 8 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 9 | 9 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
