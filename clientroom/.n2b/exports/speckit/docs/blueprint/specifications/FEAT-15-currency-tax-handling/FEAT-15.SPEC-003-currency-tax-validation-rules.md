---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-15.SPEC-003
spec_name: Currency & Tax Validation Rules
spec_slug: currency-tax-validation-rules
parent_feature: FEAT-15
parent_feature_name: Currency & Tax Handling
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 10
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Currency & Tax Validation Rules

## Overview

**Name:** Currency & Tax Validation Rules
**ID:** FEAT-15.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs which currencies are acceptable and what shape a tax line may take on a project, and blocks invoice generation with a specific, correctable message on an invalid configuration.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling
**Governed Entity:** Project (the `currency`, `tax_label`, and `tax_rate` fields)

## Scope and Non-Goals

**In Scope:**
- Field-level validation for the project's `currency`, `tax_label`, and `tax_rate` fields
- The cross-field rule governing when `tax_label`/`tax_rate` are required versus optional
- Blocking invoice generation when a project's currency/tax configuration is invalid or missing
- The single source of truth both FEAT-15.SPEC-001 (screen-level save) and FEAT-15.SPEC-008 (generation-time application) apply, per this feature's Shared Validation note

**Non-Goals:**
- Automatic, jurisdiction-specific tax calculation -- excluded per scope-boundaries.md (SC-16): this spec validates the *shape* of a freelancer-entered tax line, never computes a rate from a country or region
- Who may change a project's currency/tax -- owned entirely by FEAT-15.SPEC-005 (Currency & Tax Configuration Access Rules); this spec governs data shape, not access
- Whether a project's configuration can still be changed at all -- owned entirely by FEAT-15.SPEC-004 (Currency & Tax Lock After First Invoice); this spec's rules apply only while a project remains unlocked
- Currency conversion or exchange-rate validation -- excluded per XBR-18: the product never converts between currencies, so no conversion-related rule exists here

## Governed Entity

**Entity:** Project
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| currency | enum (recognized world currency code) | The project's billing currency |
| tax_label | text | Freelancer-entered tax line label (e.g., "VAT," "GST," "Sales Tax"), or absent when no tax applies |
| tax_rate | number (percentage) | Freelancer-entered tax rate applied to the invoice subtotal, or absent when no tax applies |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|--------------------|
| FEAT-15.SPEC-001 | Project Currency & Tax Configuration | On Save, while the project is unlocked |
| FEAT-15.SPEC-008 | Invoice Currency & Tax Line Application | At invoice-generation time, as a precondition before stamping currency/tax onto the new invoice |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| currency | Must be a recognized world currency | Always | On submit | "Choose a recognized currency to continue." | Yes |
| tax_label | Required, non-empty, max 60 characters | When "No tax line" is not selected | On submit | "Enter a tax label (for example, VAT, GST, or Sales Tax)." / "Tax label must be 60 characters or fewer." | Yes |
| tax_label | No validation beyond data type | When "No tax line" is selected | -- | -- | -- |
| tax_rate | Required; numeric; between 0 and 100 inclusive | When "No tax line" is not selected | On submit | "Enter a tax rate between 0 and 100." | Yes |
| tax_rate | No validation beyond data type | When "No tax line" is selected | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Tax shape completeness | tax_label, tax_rate | Both must be present together, or both absent ("no tax line"); a project can never carry a label with no rate, or a rate with no label | "A tax line needs both a label and a rate -- or select 'No tax line'." |
| Configuration completeness for generation | currency, tax_label, tax_rate | A project must have a valid currency and a complete tax treatment (label + rate, or explicitly none) before any invoice can be generated for it | "Set this project's currency and tax line before generating an invoice." |

## Authorization Rules

Authorization for who may reach and act on this configuration is owned entirely by FEAT-15.SPEC-005 (Currency & Tax Configuration Access Rules), which is this feature's single authoritative home for the role-action matrix on the Project currency/tax fields (per this feature's Shared UI Patterns note: "FEAT-15.SPEC-005 is the single source of truth both other specs reference rather than restating the gate"). This spec's rules below apply only after FEAT-15.SPEC-005 has already permitted the action.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Set or change currency/tax (subject to these validation rules) | See FEAT-15.SPEC-005 | See FEAT-15.SPEC-005 | See FEAT-15.SPEC-005 |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-----------------------|
| currency | No default -- the field starts empty and must be explicitly chosen | On create (first save) | Yes -- Nadia always chooses explicitly |
| tax_label / tax_rate | No default -- starts empty; "No tax line" is not preselected either way, so Nadia makes an explicit choice | On create (first save) | Yes -- Nadia always chooses explicitly |

## Business Rules

- XBR-17: a project's currency and tax line must be set before its first invoice, and the currency cannot change once the first invoice is sent (the lock itself is FEAT-15.SPEC-004's rule; this spec supplies the validity check that must pass before that first invoice can exist).
- This spec is the single source of truth for currency and tax validity across the feature -- FEAT-15.SPEC-001 and FEAT-15.SPEC-008 both apply it rather than re-deriving the rule, per this feature's Shared Validation note.
- Validation runs identically whether the save originates from Nadia's screen action (FEAT-15.SPEC-001) or from the generation-time precondition check (FEAT-15.SPEC-008) -- the product definition establishes no separate rule set for either context.
- A tax rate of exactly 0 with a tax label present is valid (e.g., a zero-rated tax category) and is distinct from "No tax line": the former still carries a label and appears on the invoice; the latter carries neither.

## Edge Cases

- **Tax rate entered as exactly 0** -- Passes validation when a tax label is also present (a zero-rated line is still a defined tax treatment, distinct from "No tax line").
- **Tax rate entered as exactly 100** -- Passes validation (upper boundary is inclusive).
- **Tax rate entered as 100.01 or -0.01** -- Fails validation with "Enter a tax rate between 0 and 100."
- **Tax label at exactly 60 characters** -- Passes validation. 61 characters shows the length error.
- **Tax label left as whitespace only** -- Treated as empty; fails the "required" rule with the same message as a blank field.
- **"No tax line" toggled after a valid label and rate were entered, then generation is attempted immediately** -- The cross-field completeness rule evaluates the current state (no tax line, explicitly chosen) as complete; generation is not blocked.
- **Currency chosen but tax fields left in an inconsistent state (label present, rate absent) due to a partial client-side interruption** -- The cross-field "Tax shape completeness" rule catches this on submit and blocks the save with its exact error message, regardless of how the inconsistent state arose.

## Acceptance Criteria

**FEAT-15.SPEC-003-AC-01:** Given Nadia selects a recognized world currency, when she submits, then the currency field passes validation with no error.

**FEAT-15.SPEC-003-AC-02:** Given Nadia leaves the currency field unset, when she submits, then she sees "Choose a recognized currency to continue." and the save is blocked.

**FEAT-15.SPEC-003-AC-03:** Given Nadia has "No tax line" off and leaves the tax label empty, when she submits, then she sees "Enter a tax label (for example, VAT, GST, or Sales Tax)." and the save is blocked.

**FEAT-15.SPEC-003-AC-04:** Given Nadia enters a tax rate of 150, when she submits, then she sees "Enter a tax rate between 0 and 100." and the save is blocked.

**FEAT-15.SPEC-003-AC-05:** Given Nadia enters a tax rate of exactly 0 alongside a tax label "Zero-Rated VAT", when she submits, then the configuration passes validation and saves.

**FEAT-15.SPEC-003-AC-06:** Given Nadia selects "No tax line", when she submits with only a currency chosen, then the configuration passes validation with no tax label or rate.

**FEAT-15.SPEC-003-AC-07:** Given a project reaches an inconsistent state with a tax label present but no rate, when a save or generation-time check evaluates it, then it is blocked with "A tax line needs both a label and a rate -- or select 'No tax line'."

**FEAT-15.SPEC-003-AC-08:** Given a project has never had currency or tax configured, when invoice generation (FEAT-15.SPEC-008) checks its configuration, then generation is blocked with "Set this project's currency and tax line before generating an invoice."

**FEAT-15.SPEC-003-AC-09:** Given a project has a valid currency and a complete tax treatment, when invoice generation checks its configuration, then the precondition passes and generation proceeds.

**FEAT-15.SPEC-003-AC-10:** Given Nadia enters a tax label of exactly 60 characters, when she submits, then it passes validation; at 61 characters it is blocked with the length error message.

**FEAT-15.SPEC-003-AC-11:** Given the same currency and tax rules apply on both the configuration screen (FEAT-15.SPEC-001) and generation-time application (FEAT-15.SPEC-008), when either context evaluates an identical configuration, then the outcome (pass or fail, and the exact error) is identical.

**FEAT-15.SPEC-003-AC-12:** Given Nadia enters a tax rate of exactly 100, when she submits, then it passes validation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 1 (delegated to FEAT-15.SPEC-005) | 1 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |
