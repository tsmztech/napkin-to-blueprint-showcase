---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-01.SPEC-004
spec_name: Service Field & Deposit Rule Validation
spec_slug: service-field-deposit-rule-validation
parent_feature: FEAT-01
parent_feature_name: Service & Pricing Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
rule_count: 10
acceptance_criteria_count: 17
---

# Logic/Rule Spec: Service Field & Deposit Rule Validation

## Overview

**Name:** Service Field & Deposit Rule Validation
**ID:** FEAT-01.SPEC-004
**Type:** Logic/Rule
**Purpose:** Defines every field-level validation rule, cross-field deposit-rule rule, authorization rule, and default/derivation for the Service entity, shared by the Add and Edit screens.
**Parent Feature:** FEAT-01 -- Service & Pricing Management
**Governed Entity:** Service

## Scope and Non-Goals

**In Scope:**
- Per-field validation rules for every Service field this feature owns (name, price, duration, deposit_rule, display_order, status)
- The cross-field deposit-rule rules, including the minimum chargeable deposit
- Authorization rules for every action this feature defines on the Service entity, per role
- Default values and derived fields on the Service entity
- Error messages for every validation failure

**Non-Goals:**
- Validating buffer_override -- that field is written exclusively by FEAT-02 (Availability & Working Hours Setup), through its own screen and its own Logic/Rule spec; this spec only notes its presence on the entity
- Determining whether an archive is safe to complete (counting upcoming bookings) -- that check is a standalone Automation, FEAT-01.SPEC-006 (Archive Impact Check); this spec governs field-level rules and authorization only
- Governing whether an edit or archive can retroactively change an already-confirmed booking -- excluded here and covered by FEAT-01.SPEC-005 (Price & Deposit Lock at Booking Time), which elaborates XBR-04 as its own standalone rule
- Dynamic, demand-based, or time-of-day pricing rules -- excluded per scope-boundaries.md SC-14: the product defines exactly one fixed price per service, so no rule set for variable pricing exists to validate

## Governed Entity

**Entity:** Service
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | The service's client-facing name |
| price | number | The fixed price of the service, in the Pro Account's currency |
| duration | number | The service's fixed length, in minutes |
| deposit_rule | enum + number | Either a fixed deposit amount (no greater than price) or a percentage (1-100%) of price |
| buffer_override | number (optional) | Optional per-service buffer in minutes; written only by FEAT-02, read by FEAT-03 -- out of scope for this spec's validation |
| display_order | number | The service's position on the booking page and on FEAT-01.SPEC-001 |
| status | enum (Active \| Archived) | The service's current lifecycle state |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-002 | Add Service | On field blur and form submit; authorization on screen entry (Pro only) and on save |
| FEAT-01.SPEC-003 | Edit Service | On field blur and form submit; authorization on screen entry (edit rendering vs. read-only), on save, and on the Archive action |
| FEAT-01.SPEC-001 | Service List | Authorization only, for the Reorder and Reactivate actions -- no field validation is checked on this screen |
| FEAT-01.SPEC-006 | Archive Impact Check | Authorization only, for the underlying Archive action this automation gates -- the automation itself applies no field validation |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | Required, non-empty, 1-80 characters | Always | On blur and on submit | "Service name is required" / "Service name must be 80 characters or fewer" | Yes |
| price | Required, a positive amount greater than zero, in the Pro Account's currency | Always | On blur and on submit | "Enter a price greater than zero" | Yes |
| duration | Required, a positive number of minutes, in 5-minute steps, no greater than 720 minutes (12 hours) | Always | On blur and on submit | "Duration is required" / "Duration must be in 5-minute steps" / "Duration cannot exceed 12 hours" | Yes |
| deposit_rule (fixed form) | The fixed amount must be greater than zero and no greater than price | When the Pro has selected the Fixed amount form | On blur and on submit | "Deposit amount must be greater than $0 and no more than the service price" | Yes |
| deposit_rule (percentage form) | The percentage must be an integer between 1 and 100 | When the Pro has selected the Percentage form | On blur and on submit | "Deposit percentage must be between 1% and 100%" | Yes |
| display_order | No validation beyond data type -- assigned automatically on create (appended to the end of the Active list) and updated only by drag-reorder in FEAT-01.SPEC-001; never a raw user-entered value | Always | -- | -- | -- |
| status | No field-level validation -- this is a state transition (Active <-> Archived) governed by the Authorization Rules below and by FEAT-01.SPEC-006's archive-time processing, not by field format rules | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Fixed deposit ceiling | price, deposit_rule (fixed form) | The fixed deposit amount must be no greater than price -- a service can require up to (but never more than) full prepayment | "Deposit amount must be greater than $0 and no more than the service price" |
| Minimum chargeable deposit | price, deposit_rule | The deposit that results from the current price and deposit rule (the fixed amount as entered, or price x percentage / 100) must be at least platform parameter: `minimum-chargeable-deposit` -- the smallest amount a card payment can be taken for | "The deposit for this price and rule is below the minimum amount that can be charged. Increase the price, deposit amount, or percentage." |
| Deposit rule form exclusivity | deposit_rule (fixed form), deposit_rule (percentage form) | Exactly one form is active at a time; switching forms clears the other form's previously entered value rather than reinterpreting it | N/A -- this is a UI-state rule with no error condition, enforced as a field reset in FEAT-01.SPEC-002/FEAT-01.SPEC-003's Deposit Rule toggle interaction |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create service | The Pro | Always, for their own account only | -- |
| Create service | Platform Operator (Support) | Never | Add Service is never opened in a support context (Access Matrix: Service & Availability Setup = View); no control exists for Support to reach it |
| Create service | The Client | Never | Service & Pricing Management screens are unreachable through any Client-facing path |
| View service (list or detail) | The Pro | Always, for their own account's services only | -- |
| View service (list or detail) | Platform Operator (Support) | Always, read-only, for the Pro account under an active help request | -- |
| View service (list or detail) | The Client | Never (setup views) | The Client sees only the resulting public Active service list on the booking page (FEAT-05), never this feature's setup screens |
| Edit service fields | The Pro | Always, for their own account's services only | -- |
| Edit service fields | Platform Operator (Support) | Never | Every field renders as static text; banner reads "Support view -- no changes can be made here" |
| Edit service fields | The Client | Never | Not reachable, as above |
| Archive service | The Pro | Always, subject to completing the FEAT-01.SPEC-006 impact check and confirming | -- |
| Archive service | Platform Operator (Support) | Never | No Archive control is rendered for this role |
| Archive service | The Client | Never | Not reachable, as above |
| Reactivate service | The Pro | Always, for their own account's archived services only | -- |
| Reactivate service | Platform Operator (Support) | Never | No Reactivate control is rendered for this role |
| Reactivate service | The Client | Never | Not reachable, as above |
| Reorder services | The Pro | Always, for their own account's Active services only | -- |
| Reorder services | Platform Operator (Support) | Never | No drag handle is rendered for this role |
| Reorder services | The Client | Never | Not reachable, as above |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status | Active | On create only | No -- a new service is always created Active; only the Archive action (via FEAT-01.SPEC-006) or Reactivate (via FEAT-01.SPEC-001) changes it afterward |
| display_order | Appended to the end of the Pro's current Active list | On create, and on Reactivate | No direct override -- the Pro repositions it afterward only via drag-reorder on FEAT-01.SPEC-001 |
| Computed deposit (preview only, not a stored field) | Fixed form: the entered amount, unchanged. Percentage form: price x percentage / 100, rounded to the currency's smallest unit | Recalculated live on every price or deposit-rule change, on both Add and Edit | No -- it is always derived from the current price and deposit_rule; the Pro can only change the inputs that feed it |

## Business Rules

- All validation rules in this spec apply identically on create (FEAT-01.SPEC-002) and edit (FEAT-01.SPEC-003) -- the product definition establishes no create-only or edit-only field rules.
- Field validation and the minimum-chargeable-deposit rule run before any save is attempted; a service is never persisted in an invalid state.
- XBR-25: price is always entered and displayed in the Pro Account's currency; this spec defines no per-service currency override, since currency is owned by FEAT-27 and locked once the first deposit is taken.
- The minimum-chargeable-deposit rule (platform parameter: `minimum-chargeable-deposit`) exists because a deposit below it could never actually be collected by the payment-processing capability behind FEAT-07 -- this is a correctness rule, not a business preference.
- Authorization for every action defined here is evaluated on every attempt, not only at screen entry -- a role change or session change mid-session is re-checked the next time an action is attempted.

## Edge Cases

- **Service name at exactly 80 characters** -- Passes validation. 81 characters shows the "must be 80 characters or fewer" error.
- **Price entered as exactly the smallest unit above zero (e.g., one cent)** -- Passes the "greater than zero" rule; whether the resulting deposit still clears the minimum-chargeable-deposit rule depends on the deposit rule chosen.
- **Duration at exactly 5 minutes** -- Passes validation (the minimum step). **Duration at exactly 720 minutes (12 hours)** -- Passes validation. 725 minutes shows the "cannot exceed 12 hours" error.
- **Fixed deposit exactly equal to price** -- Passes the fixed-deposit ceiling rule (full prepayment is allowed). One cent above price fails with the ceiling error.
- **Percentage deposit at exactly 1%** and **exactly 100%** -- Both pass validation; 0% and 101% each fail with the percentage-range error.
- **Computed deposit exactly equal to the minimum chargeable amount** -- Passes the minimum-chargeable-deposit rule. One cent below it fails.
- **Pro switches from Percentage back to Fixed after a valid percentage was entered** -- The percentage value is cleared per the deposit-rule-form-exclusivity rule; the fixed field starts empty and re-triggers the required-field rule if the Pro attempts to save before entering a new value.
- **Support's role changes mid-session (help request ends)** -- The next authorization check (e.g., an attempted view refresh) re-evaluates Support's access; a lapsed help request is out of this spec's scope to define further (owned by FEAT-19), but this spec's Authorization Rules table's "Always, read-only, for the Pro account under an active help request" condition is what gates it.

## Acceptance Criteria

**FEAT-01.SPEC-004-AC-01:** Given Talia leaves the name field empty and moves to the next field, then the name field shows the error "Service name is required."

**FEAT-01.SPEC-004-AC-02:** Given Talia enters an 81-character service name, when she blurs the field, then she sees "Service name must be 80 characters or fewer."

**FEAT-01.SPEC-004-AC-03:** Given Talia enters a price of $0, when she blurs the field, then she sees "Enter a price greater than zero."

**FEAT-01.SPEC-004-AC-04:** Given Talia enters a duration of 7 minutes, when she blurs the field, then she sees "Duration must be in 5-minute steps."

**FEAT-01.SPEC-004-AC-05:** Given Talia enters a duration of 725 minutes, when she blurs the field, then she sees "Duration cannot exceed 12 hours."

**FEAT-01.SPEC-004-AC-06:** Given Talia sets a $100 price and enters a fixed deposit of $150, when she blurs the deposit field, then she sees "Deposit amount must be greater than $0 and no more than the service price."

**FEAT-01.SPEC-004-AC-07:** Given Talia sets a $100 price and a fixed deposit of $100, when she blurs the deposit field, then no error is shown (full prepayment is valid).

**FEAT-01.SPEC-004-AC-08:** Given Talia selects the Percentage form and enters 0%, when she blurs the field, then she sees "Deposit percentage must be between 1% and 100%."

**FEAT-01.SPEC-004-AC-09:** Given Talia sets a very low price and a percentage that results in a deposit below the minimum chargeable amount, when she blurs the deposit field, then she sees "The deposit for this price and rule is below the minimum amount that can be charged. Increase the price, deposit amount, or percentage."

**FEAT-01.SPEC-004-AC-10:** Given Talia has entered a fixed deposit value and then switches the toggle to Percentage, when the toggle switches, then the previously entered fixed value is cleared rather than reinterpreted as a percentage.

**FEAT-01.SPEC-004-AC-11:** Given Talia saves a new service, then its status is set to Active and its display_order is appended to the end of her current Active list automatically, with no field on the form for either value.

**FEAT-01.SPEC-004-AC-12:** Given Talia (the Pro) attempts to edit a service, when she has valid access to her own account, then the edit is allowed unconditionally.

**FEAT-01.SPEC-004-AC-13:** Given Platform Operator (Support) views a Pro's service during a help request, when Support looks for any edit control, then none is rendered, and the "Support view -- no changes can be made here" banner is shown instead.

**FEAT-01.SPEC-004-AC-14:** Given Riley (the Client) attempts to reach any Service & Pricing Management screen, then no such path exists in the product's Client-facing navigation, and any direct attempt lands on the ordinary unauthenticated/Pro sign-in experience per FEAT-01.SPEC-001/002/003's Access and Visibility tables.

**FEAT-01.SPEC-004-AC-15:** Given Talia archives a service, when she confirms the archive, then the action is allowed because she is the Pro completing the required FEAT-01.SPEC-006 impact check; the same action attempted by Support or a Client is never possible because no Archive control exists for those roles.

**FEAT-01.SPEC-004-AC-16:** Given Talia reactivates an archived service, then its display_order is appended to the end of her current Active list, not restored to its previous position.

**FEAT-01.SPEC-004-AC-17:** Given Talia types a price and a percentage deposit rule, when either value changes, then the computed deposit preview recalculates live as price x percentage / 100, rounded to the currency's smallest unit, and the Pro cannot directly edit that computed number.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 7 | 7 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 16 | 16 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |
