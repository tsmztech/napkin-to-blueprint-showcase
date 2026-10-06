---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-05.SPEC-007
spec_name: Booking Details Field Validation
spec_slug: booking-details-field-validation
parent_feature: FEAT-05
parent_feature_name: Public Booking Page & Booking Flow
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 8
acceptance_criteria_count: 16
---

# Logic/Rule Spec: Booking Details Field Validation

## Overview

**Name:** Booking Details Field Validation
**ID:** FEAT-05.SPEC-007
**Type:** Logic/Rule
**Purpose:** Defines all validation rules, conditional requirements, and authorization rules for the name, phone, texting opt-in, email, and note fields captured on the Client Details & Consent screen.
**Parent Feature:** FEAT-05 -- Public Booking Page & Booking Flow
**Governed Entity:** Client record (booking-time fields: name, phone, email, booking_notes) and the texting opt-in captured alongside it, which governs whether a Messaging Consent record is created

## Scope and Non-Goals

**In Scope:**
- Per-field validation rules for name, phone, email, and booking_notes as captured during the booking flow
- The conditional-required rule linking the texting opt-in to the email field
- The rule that the texting opt-in must be an explicit, never-pre-checked action
- Authorization rules for who may submit and read these fields during the flow
- Default values for these fields at booking time

**Non-Goals:**
- Validation of a client's email or consent when updated on a later visit -- owned by FEAT-06 (Client Booking Identity), which governs the client's own self-service updates, not this booking-time capture
- Validation of the Pro's private_note field on the Client entity -- that field is Pro-only and never captured or shown in this flow; it belongs to FEAT-13 (Client Record Management)
- The phone-based identity lookup and returning-client matching logic itself -- owned by FEAT-06; this spec only validates the phone field's format, not what happens once a match is found
- Deriving or capturing the policy acknowledgment -- owned by FEAT-05.SPEC-009, a distinct governed concern from the contact and consent fields this spec covers

## Governed Entity

**Entity:** Client (booking-time fields) and the texting opt-in
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | Client's name, as entered on this booking |
| phone | text | Client's phone number -- the identity key for this Client within this Pro |
| email | text | Client's email address, required only when texting is declined |
| booking_notes | text | The client's optional note to the Pro for this booking |
| texting_opt_in | boolean (ephemeral -- drives Messaging Consent creation, not a Client field itself) | Whether the client actively consents to receive texts from this Pro |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|---------------------|
| FEAT-05.SPEC-003 | Client Details & Consent | On field blur (name, phone, email, booking_notes) and on Continue submit (all fields, including the cross-field email-requirement rule); authorization checked on screen entry and on submit |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | Required, non-empty, 1-100 characters | Always | On blur | "Please enter your name" / "Name must be 100 characters or fewer" | Yes |
| phone | Required, non-empty, valid reachable phone format | Always | On blur | "Please enter your phone number" / "Please enter a valid phone number" | Yes |
| texting_opt_in | No format validation -- a boolean toggle; must never be pre-checked on initial render (see Business Rules) | Always | On render (initial state check, not a validation error) | Not applicable -- this is a UI-state rule with no error condition | No |
| email | Required, valid email format | Only when texting_opt_in is false (texts declined) | On blur (format) and on submit (conditional requirement) | "Please enter your email so we can send confirmations and reminders" (when required and empty) / "Please enter a valid email address" (when provided but malformed) | Yes |
| email | Valid email format when provided, even if not required | texting_opt_in is true (texts accepted) but client still enters an email | On blur | "Please enter a valid email address" | Yes |
| booking_notes | Optional; maximum 300 characters | Always | On blur (length) | "Your note must be 300 characters or fewer" | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Conditional email requirement | texting_opt_in, email | Email is required only when texting_opt_in is false; toggling texting_opt_in to true clears the required indicator on email without clearing any value already entered | "Please enter your email so we can send confirmations and reminders" (shown only when texting_opt_in is false and email is empty at submit) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Submit booking details (name, phone, opt-in, email, note) | The Client (Riley) | Always, for their own in-progress booking | -- |
| Read own entered (in-progress, unsubmitted) details | The Client (Riley) | Always -- limited to the client's own device session for the booking they are actively entering | -- |
| Access another client's entered (in-progress, unsubmitted) details from a different device or session | The Client (Riley) | Never | Not applicable -- no mechanism exists to view another client's unsubmitted entry from a different session; the question does not arise |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| name | Pre-filled from a matched existing Client record when the entered phone number matches one for this Pro (via FEAT-06's lookup); otherwise empty | On phone field blur, if a match is found | Yes -- the client may edit the pre-filled name before continuing |
| texting_opt_in | Unchecked (false) | On screen first load, always | Yes -- the client may check it |
| email | Empty | On screen first load | Yes |
| booking_notes | Empty | On screen first load | Yes |

## Business Rules

- The texting opt-in must be an explicit, affirmative action -- it is never pre-checked, and the exact wording shown beside it at the moment of checking is retained as compliance evidence (US SMS-consent rules, ASMP-24; XBR-15).
- If texting_opt_in is false at submit time, no Messaging Consent record is created for the text channel; the client must instead have a valid email on file so confirmations and reminders can still arrive by the email fallback (XBR-15).
- A phone number matching an existing Client record within this Pro is treated as the same client (dependency map's Client contention note) -- the phone field's validation is format-only; identity resolution itself is FEAT-06's responsibility, referenced here, not duplicated.
- The booking_notes field's 300-character limit and no-medical-information hint are enforced identically regardless of which service or Pro is being booked -- these are product-wide field rules, not per-Pro configuration (scope-boundaries.md SC-08).
- All rules in this spec apply identically whether the client is new or returning -- the product definition establishes no returning-client exemption from field validation.

## Edge Cases

- **Phone entered with international formatting (e.g., +1-555-123-4567)** -- Passes validation; the accepted format allows digits, spaces, dashes, parentheses, and a leading plus.
- **Name at exactly 100 characters** -- Passes validation; 101 characters shows the length error.
- **Booking_notes at exactly 300 characters** -- Passes validation; 301 characters shows the length error.
- **Email left as whitespace only** -- Treated as empty for the purposes of the conditional-required rule; if texting_opt_in is false, the required-field error is shown, not a format error.
- **Client checks texting_opt_in, enters an email anyway, then unchecks it before submitting** -- The email requirement is re-evaluated at submit time based on the final state of texting_opt_in; the already-entered email satisfies the requirement without needing to be re-typed.
- **Client enters a valid email while texting_opt_in remains true** -- No error; an optional, valid email is accepted even though it is not required, and is stored for the client's record.
- **Phone number changed after a name was pre-filled from a match** -- The pre-filled name is not automatically cleared by this spec's rules; FEAT-05.SPEC-003 re-runs the lookup against the new number and updates the pre-fill if a different match (or no match) results.
- **Platform Operator (Support) views this form outside a live client session** -- No submission is possible; the "never" authorization row applies, and no error state is reached because the action is not offered at all.

## Acceptance Criteria

**FEAT-05.SPEC-007-AC-01:** Given Riley leaves the name field empty and moves to the next field, then the name field shows "Please enter your name."

**FEAT-05.SPEC-007-AC-02:** Given Riley enters a name of exactly 100 characters, then no error is shown; given she enters 101 characters, then "Name must be 100 characters or fewer" is shown.

**FEAT-05.SPEC-007-AC-03:** Given Riley leaves the phone field empty and moves to the next field, then the phone field shows "Please enter your phone number."

**FEAT-05.SPEC-007-AC-04:** Given Riley enters a phone number in an invalid format, then the phone field shows "Please enter a valid phone number"; given she enters "+1-555-123-4567", then no error is shown.

**FEAT-05.SPEC-007-AC-05:** Given Riley is on the Client Details & Consent screen, when it first renders, then the texting opt-in checkbox is unchecked.

**FEAT-05.SPEC-007-AC-06:** Given Riley leaves the texting opt-in unchecked and the email field empty, when she taps Continue, then the email field shows "Please enter your email so we can send confirmations and reminders" and submission is blocked.

**FEAT-05.SPEC-007-AC-07:** Given Riley leaves the texting opt-in unchecked and enters a validly formatted email, when she taps Continue, then no email error is shown and submission proceeds.

**FEAT-05.SPEC-007-AC-08:** Given Riley checks the texting opt-in, when she taps Continue with the email field empty, then no email error is shown, since email is not required in this branch.

**FEAT-05.SPEC-007-AC-09:** Given Riley checks the texting opt-in but still enters an email in an invalid format, when the field loses focus, then "Please enter a valid email address" is shown, since format validation applies whenever a value is provided.

**FEAT-05.SPEC-007-AC-10:** Given Riley enters a booking note of exactly 300 characters, then no error is shown; given she enters 301 characters, then "Your note must be 300 characters or fewer" is shown.

**FEAT-05.SPEC-007-AC-11:** Given Riley enters a phone number matching an existing Client record with this Pro, when the phone field loses focus, then the name field is pre-filled with the matched client's name, which Riley may still edit.

**FEAT-05.SPEC-007-AC-12:** Given Riley (the Client) submits her own booking details, when submission runs, then it is allowed without restriction.

**FEAT-05.SPEC-007-AC-13:** Given Riley is filling in her booking details, when she reviews the form before submitting, then she can see and edit every value she has entered in her own session.

**FEAT-05.SPEC-007-AC-14:** Given Riley's entered, unsubmitted details exist only in her own device session, when the question of another client accessing them from a different session arises, then no mechanism exists for it -- the scenario does not occur.

**FEAT-05.SPEC-007-AC-15:** Given Riley checks the opt-in, enters an email, then unchecks the opt-in before tapping Continue, when submission runs, then the previously entered email is used to satisfy the now-active email requirement without Riley re-typing it.

**FEAT-05.SPEC-007-AC-16:** Given Riley's entered phone number matches an existing client but she changes it before continuing, when the phone field loses focus again, then the lookup re-runs against the new number and the name pre-fill updates accordingly.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 3 | 3 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |
