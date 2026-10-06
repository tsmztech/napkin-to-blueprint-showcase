---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-13.SPEC-005
spec_name: Client Field Validation & Access Rules
spec_slug: client-field-validation-access-rules
parent_feature: FEAT-13
parent_feature_name: Client Record Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 20
acceptance_criteria_count: 17
---

# Logic/Rule Spec: Client Field Validation & Access Rules

## Overview

**Name:** Client Field Validation & Access Rules
**ID:** FEAT-13.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs name/phone/email/note field validation, phone-number identity uniqueness within one Pro, and the private-note field's Pro-only visibility.
**Parent Feature:** FEAT-13 -- Client Record Management
**Governed Entity:** Client record

## Scope and Non-Goals

**In Scope:**
- Per-field validation rules for name, phone, email, and private_note, as edited through this feature
- The cross-field conditional requiring email when the client has declined texts
- Phone-number identity-uniqueness within one Pro
- The private_note field's Pro-only visibility (excluded from Support's view)
- Authorization rules for viewing, editing, and deleting the Client record, for every role this feature's screens or Support's reading of this entity touch

**Non-Goals:**
- Validation of booking_notes (the client's own optional per-booking note) -- this field is client-authored via other features (the booking flow) and is read-only from this feature, per the Brief's Shared Entities note; its own validation lives with the feature that captures it (FEAT-05)
- The Client's own update of their email and texting consent through FEAT-06 -- that action is owned by FEAT-06 (Client Booking Identity) under the Access Matrix's "Booking & Payment" group, not by this feature's Authorization Rules
- Deletion eligibility (the upcoming-booking block) and retention/de-identification behavior -- handled entirely by FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule); this spec governs field validation and access, not the deletion condition itself
- Validation of derived fields (booking_history) -- excluded because this is a computed, read-only list with no user-entered values to validate

## Governed Entity

**Entity:** Client
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | Client's full name |
| phone | text | Client's phone number; required, identity key within one Pro |
| email | text | Client's email address; required when texts are declined, otherwise optional |
| private_note | text | The Pro's own private note about this client; Pro-only visibility |
| booking_notes | text | The client's own optional per-booking note, captured elsewhere; read-only from this feature |
| booking_history | derived | This client's Bookings with this Pro, derived from the Booking entity; read-only from this feature |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-13.SPEC-001 | Client Record Detail | Private note validation on blur (character limit) and on save; view-access enforced on screen entry |
| FEAT-13.SPEC-002 | Client Contact Edit | Name/phone/email validation on field blur and on form submit; edit-access enforced on screen entry and on save |
| FEAT-13.SPEC-003 | Client Deletion Confirmation | Delete-access enforced on screen entry; the deletion action itself is gated by FEAT-13.SPEC-006 |
| FEAT-13.SPEC-004 | Client Deletion Execution | Delete-access re-enforced at execution time, consistent with FEAT-13.SPEC-003 |
| FEAT-19.SPEC-004 | Support Session Scope & Access Rules | View-access (excluding private_note) enforced when Support reads this entity; that spec cites this rule as the owner of the client private-note exclusion at its hand-off points |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | Required, non-empty, 1--100 characters | Always | On blur and on submit | "Name is required" / "Name must be 100 characters or fewer" | Yes |
| phone | Required, non-empty, valid reachable phone format | Always | On blur and on submit | "Phone number is required" / "Please enter a valid phone number" | Yes |
| phone | Must not match another of this Pro's clients (identity-uniqueness within one Pro), as enforced within this feature's edit context | Always, when editing an existing client's phone through FEAT-13.SPEC-002 | On submit | "This phone number is already used by another client. Check for a duplicate before saving." | Yes |
| email | Valid email format | When provided (not empty) | On blur | "Please enter a valid email address" | Yes |
| email | Required | When the client has declined texts (no active texting opt-in for this Pro) | On submit | "An email address is required for clients who haven't opted in to texts" | Yes |
| private_note | Up to 1,000 characters | Always | On blur (blocks further typing at the limit) and on submit | "Private note can be up to 1,000 characters." | Yes |
| booking_notes | No validation beyond data type -- read-only from this feature | Always | -- | -- | -- |
| booking_history | No validation beyond data type -- derived, read-only from this feature | Always | -- | -- | -- |

**Note -- creation-time phone matches are out of scope for this rule:** the phone-uniqueness rule above governs only edits made through this feature (FEAT-13.SPEC-002). It is a rejection, not a merge, and applies solely when the Pro changes an existing client's phone number to one already on another of her client records. Creation of a new Client record (a first booking via FEAT-05, or a Pro booking a client in via FEAT-30) is a Non-Goal of this feature (see Scope and Non-Goals) and is not governed here. Per the Feature Dependency Map's Contention resolution for the Client entity, a phone-number match found at creation time resolves to a single Client record (merge, never a duplicate, never a rejection) -- that resolution is owned by the creating features, FEAT-05 and FEAT-30, not by this spec.

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Email conditionally required | email, texting opt-in state (on the associated Messaging Consent record) | Email must be non-empty when the client has no active texting consent for this Pro; email may be empty when active texting consent exists | "An email address is required for clients who haven't opted in to texts" |
| Phone-change consequence acknowledgment | phone | When phone is changed from its stored value, the change may only be saved after the Pro acknowledges (via FEAT-13.SPEC-002's confirmation step) that access links will be invalidated and fresh texting consent will be required (XBR-18, XBR-15) | N/A -- this is a confirmation gate, not a rejection; no error message, the save is simply held until acknowledged |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View client record (all fields, including private_note) | The Pro (Talia) | Own clients only | -- |
| View client record (all fields except private_note) | Platform Operator (Support) | Always, but only through FEAT-19's own read-only screen -- never through this feature's own screens | Support attempting this feature's own screens is redirected to the Pro sign-in screen (per XBR-29); the private_note field is never rendered anywhere Support can reach, even within FEAT-19 |
| View client record | The Client (Riley) | Never, through this feature | This feature's screens are never reached by a Client; a Client's own contact details are shown only through FEAT-06 (Client Booking Identity), which reads the entity independently of this feature's Authorization Rules |
| Edit private_note | The Pro (Talia) | Own clients only | -- |
| Edit private_note | Platform Operator (Support) | Never | The field is never shown to Support, and no edit control exists for it outside this feature |
| Edit private_note | The Client (Riley) | Never | The field does not appear anywhere a Client can reach |
| Edit contact fields (name, phone, email) | The Pro (Talia) | Own clients only | -- |
| Edit contact fields (name, phone, email) | Platform Operator (Support) | Never | Support's access is View-only and never includes correcting contact details, per scope-boundaries.md SC-05; no edit control is rendered |
| Edit contact fields (name, phone, email) via this feature | The Client (Riley) | Never | A Client's own email update runs through FEAT-06 (Client Booking Identity), not this feature; this feature exposes no client-facing edit path at all, per scope-boundaries.md SC-04 |
| Delete client record | The Pro (Talia) | Own clients only, and only when eligible per FEAT-13.SPEC-006 (no upcoming booking) | When ineligible: the Blocked panel in FEAT-13.SPEC-003, not a bare denial -- see that spec |
| Delete client record | Platform Operator (Support) | Never | Support cannot perform a deletion on the Pro's behalf, per scope-boundaries.md SC-05; no delete control exists in Support's read-only view |
| Delete client record | The Client (Riley) | Never | A Client cannot request or execute their own deletion in-product, per scope-boundaries.md SC-01; deletion is performed only by the Pro after an informal request |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| name, phone, email | Set from the values captured at the client's first booking (FEAT-05) or Pro-entered booking-in (FEAT-30) | On create only (outside this feature's scope) | Yes -- the Pro may correct any of these three fields at any time through FEAT-13.SPEC-002 |
| private_note | Empty | On create only (outside this feature's scope) | Yes -- the Pro sets or edits it at any time through FEAT-13.SPEC-001 |
| booking_history | Derived from the Booking entity: every Booking record referencing this client with this Pro | Always, recalculated on each load of FEAT-13.SPEC-001 | No -- always derived, never directly editable |

## Business Rules

- XBR-18: a changed phone number invalidates the client's existing access links; enforced at the point of save in FEAT-13.SPEC-002, not by this spec directly, but this spec's phone-change cross-field rule is what surfaces the acknowledgment gate that makes the save possible.
- XBR-15: a changed phone number requires the client to re-grant texting consent for the new number before any further text is sent; the re-grant itself happens through FEAT-06/FEAT-14, outside this spec's scope, but the email-conditional-required rule above depends on the current texting-consent state this rule change eventually produces.
- Per the dependency map's Contention note for the Client entity, field edits between the Pro's own sessions resolve last-write-wins; this spec does not alter that resolution, it only governs what values are acceptable to write.
- All field validation rules apply identically whether the field is edited from FEAT-13.SPEC-001 (private_note only) or FEAT-13.SPEC-002 (name/phone/email) -- the product definition establishes no screen-specific validation variance.

## Edge Cases

- **Phone number entered with international formatting (e.g., +1-555-123-4567)** -- Passes the valid-reachable-format rule; the format allows digits, spaces, dashes, parentheses, and a leading plus.
- **Name at exactly 100 characters** -- Passes validation. 101 characters shows the length error.
- **Private note at exactly 1,000 characters** -- Passes validation. 1,001 characters is blocked at input.
- **Client has active texting consent, then that consent is revoked (via FEAT-14) after the client record was saved with no email on file** -- The email-conditionally-required rule is not retroactively enforced against already-saved data; it is checked only when the Pro next saves a contact edit through FEAT-13.SPEC-002, at which point an empty email will be rejected if texting consent is not active at that moment.
- **Two of the Pro's clients are found to share a phone number due to a data-entry correction** -- The uniqueness rule blocks the edit-time save (via FEAT-13.SPEC-002) that would create the collision; the Pro must resolve which record is correct manually (out of scope for this spec, which only prevents the collision going forward through this feature's edit path) before either record can be saved with that number. This is distinct from a phone match found at creation time (FEAT-05/FEAT-30), which merges into a single record per the dependency map rather than being rejected.
- **Pro attempts to view a client's private_note as Platform Operator (Support) via FEAT-19** -- The field is omitted entirely from Support's view (per FEAT-12.SPEC-008's established Support-omission pattern for this same field), not shown blank or redacted -- it is not present in that screen's layout at all.
- **Ownership boundary: a client record somehow becomes associated with two Pro accounts (data inconsistency)** -- Not possible under the product definition (per the dependency map, "a client who books with two Pros gets two separate, unconnected Client records," scope-boundaries.md SC-04); this spec's "own clients only" condition is therefore always well-defined and never ambiguous.

## Acceptance Criteria

**FEAT-13.SPEC-005-AC-01:** Given Talia clears the name field on FEAT-13.SPEC-002 and moves focus away, when blur validation runs, then the error "Name is required" appears.

**FEAT-13.SPEC-005-AC-02:** Given Talia enters a name of exactly 100 characters, when she saves, then it is accepted; entering 101 characters shows "Name must be 100 characters or fewer."

**FEAT-13.SPEC-005-AC-03:** Given Talia enters an invalid phone format, when blur validation runs, then the error "Please enter a valid phone number" appears.

**FEAT-13.SPEC-005-AC-04:** Given Talia enters a phone number already used by another of her own clients, when she taps Save, then the save is rejected with "This phone number is already used by another client. Check for a duplicate before saving."

**FEAT-13.SPEC-005-AC-05:** Given Talia enters an invalid email format, when blur validation runs, then the error "Please enter a valid email address" appears.

**FEAT-13.SPEC-005-AC-06:** Given a client has no active texting consent for Talia and Talia leaves the email field empty, when she taps Save, then the save is rejected with "An email address is required for clients who haven't opted in to texts."

**FEAT-13.SPEC-005-AC-07:** Given a client has active texting consent for Talia and Talia leaves the email field empty, when she taps Save, then the save succeeds -- email is not required.

**FEAT-13.SPEC-005-AC-08:** Given Talia types a private note of exactly 1,000 characters, when she saves, then it is accepted; a 1,001st character is blocked from being typed at all.

**FEAT-13.SPEC-005-AC-09:** Given Talia changes a client's phone number, when she attempts to save, then the save is held until she acknowledges the access-link-invalidation and fresh-consent consequence via FEAT-13.SPEC-002's confirmation step.

**FEAT-13.SPEC-005-AC-10:** Given Talia (the Pro) views her own client's record, when the screen loads, then all fields including private_note are visible to her.

**FEAT-13.SPEC-005-AC-11:** Given Platform Operator (Support) views a client record through FEAT-19's read-only screen, when the record renders, then the private_note field is entirely omitted from what Support sees.

**FEAT-13.SPEC-005-AC-12:** Given Platform Operator (Support) attempts to reach this feature's own screens directly, when the access check runs, then Support is redirected to the Pro sign-in screen and never reaches an edit or delete control for this entity.

**FEAT-13.SPEC-005-AC-13:** Given Riley (the Client) attempts to reach any of this feature's screens, when the access check runs, then she is redirected to the Pro sign-in screen -- this feature exposes no client-facing view or edit path at all.

**FEAT-13.SPEC-005-AC-14:** Given Talia attempts to delete a client with an upcoming booking, when she opens FEAT-13.SPEC-003, then the Blocked panel is shown instead of a bare denial, per FEAT-13.SPEC-006's eligibility rule.

**FEAT-13.SPEC-005-AC-15:** Given Talia attempts to delete an eligible client, when she confirms, then the deletion is allowed -- consistent with the Pro having Full access to her own clients.

**FEAT-13.SPEC-005-AC-16:** Given Platform Operator (Support) has no delete control anywhere in their read-only view, when Support views any client, then no deletion action is ever presented to them.

**FEAT-13.SPEC-005-AC-17:** Given a client's private_note field is left empty (never written to), when the Pro views the client record, then no error is shown -- private_note has no "required" rule, only a maximum-length rule.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 12 | 12 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
