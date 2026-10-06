---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-09.SPEC-007
spec_name: Invoice Content, Numbering, Amount & Due-Date Rules
spec_slug: invoice-content-numbering-amount-due-date-rules
parent_feature: FEAT-09
parent_feature_name: Invoice Generation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 18
acceptance_criteria_count: 16
---

# Logic/Rule Spec: Invoice Content, Numbering, Amount & Due-Date Rules

## Overview

**Name:** Invoice Content, Numbering, Amount & Due-Date Rules
**ID:** FEAT-09.SPEC-007
**Type:** Logic/Rule
**Purpose:** Governs sequential numbering, required business/billing details, amount-matches-trigger validation, and default-payment-terms due-date derivation for every invoice, whether automatically generated or manually issued.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending
**Governed Entity:** Invoice (specifically the `invoice_number`, `amount`, `currency`, `tax_label`, `tax_rate`, `total`, business/billing details, `issue_date`, and `due_date` fields)

## Scope and Non-Goals

**In Scope:**
- Sequential invoice numbering, unique per freelancer, across both automatic and manual creation paths
- The required-content gate (freelancer business details, client billing details) that blocks sending
- The amount-matches-trigger rule and its application to ad-hoc invoices and credit notes
- Due-date derivation from `default_payment_terms`, and Nadia's ability to adjust it before sending

**Non-Goals:**
- Currency selection and tax-rate computation -- owned entirely by Currency & Tax Handling (FEAT-15, XBR-17); this spec applies whatever currency and tax line FEAT-15 has already configured for the project, and performs no calculation of its own
- Who may view, issue, or act on an invoice -- owned by FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules)
- Immutability once sent and the mechanics of a correction -- owned by FEAT-09.SPEC-008 (Invoice Immutability & Correction Rules); this spec defines what content a correction carries, not whether it is permitted
- Automatic tax calculation per country or region -- excluded per scope-boundaries.md (SC-16): the tax line is a freelancer-configured label and rate, never computed by this product

## Governed Entity

**Entity:** Invoice
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| invoice_number | text (derived) | Unique and sequential per freelancer |
| amount | number | The amount matching the triggering milestone/deposit/completion price, or the entered ad-hoc/credit amount |
| currency | text | Set from the project's configured currency (FEAT-15) |
| tax_label, tax_rate | text / number | Set from the project's configured tax line (FEAT-15) |
| total | number | amount plus the tax line applied at `tax_rate` |
| freelancer business details | text (name, address, tax ID) | Read from the Freelancer Account (FEAT-21) |
| client billing details | text (billing name, billing address, optional tax ID) | Read from the Client record (FEAT-01) |
| issue_date | date | Set to the moment of generation or manual issuance |
| due_date | date | Derived from `default_payment_terms`, adjustable before sending |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-004 | Automatic Invoice Generation | Applied inline at creation, before the send hand-off |
| FEAT-09.SPEC-005 | Manual Invoice & Credit Note Recording | Applied inline at creation, for both the ad-hoc and credit-note paths |
| FEAT-09.SPEC-003 | Manual Invoice & Credit Note Issuance | Referenced for field-level validation and the due-date pre-fill/override on the issuance form |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| invoice_number | Must be the next unused sequential number for this freelancer; never reused, never skipped | Always, at creation | On create (either path) | Not user-facing -- this field is never entered by a human; a numbering conflict is a system-level retry condition, not a validation error shown to Nadia | Yes (system-level) |
| amount | Must equal the triggering event's price (automatic path) or be a positive value entered by Nadia (manual path) | Always | On create (automatic); on blur and submit (manual, via FEAT-09.SPEC-003) | "Enter an amount greater than zero." (manual path only -- the automatic path has no user-facing entry point for this field) | Yes |
| currency | Must be the project's configured currency; a project with no configured currency blocks generation entirely | Always | On create (either path) | "This project's currency isn't set yet. Set it before the first invoice." | Yes |
| tax_label, tax_rate | Must be the project's configured tax line, or explicitly none if the freelancer has configured no tax line | Always | On create (either path) | Not applicable -- absence of a tax line is a valid configuration, not an error | No |
| total | Must equal amount plus the tax line applied at tax_rate, computed once at creation and never independently entered | Always | On create (either path) | Not user-facing -- `total` is always derived, never entered | Yes (system-level) |
| freelancer business details | Must be complete (name, address; tax ID optional) before any invoice for this freelancer can be sent | Before the first invoice, and every subsequent one | On create (either path) | "Your business details aren't complete yet. Add them in Settings before this invoice can be sent." | Yes |
| client billing details | Must be complete (billing name, billing address; tax ID optional) before this client's first invoice, and every subsequent one, can be sent | Always | On create (either path) | "This client's billing details are incomplete. Add a billing name and address before sending an invoice." | Yes |
| issue_date | Set automatically to the moment of creation; never entered by a human | Always | On create (either path) | Not applicable -- no user input | No |
| due_date | Must derive from `default_payment_terms` unless Nadia overrides it before sending (manual path only) | Always | On create (automatic); on the issuance form, adjustable before submit (manual) | "Choose a due date." (only if Nadia clears the pre-filled value on the manual form) | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Amount-matches-trigger | amount, total, triggering_event | On the automatic path, `amount` must exactly equal the price recorded on the triggering milestone, deposit, or completion payment at the moment of the trigger -- no rounding or adjustment | Not user-facing -- a mismatch on this path is a defect in the upstream trigger data, not a condition a user corrects; FEAT-09.SPEC-004 treats it as a generation-blocking system condition |
| Total-derivation | amount, tax_rate, total | `total` is always `amount` plus (`amount` × `tax_rate`), computed once at creation | Not applicable -- always derived |
| Credit-amount-within-original | amount (credit note), total (original invoice) | A credit note's amount can never exceed the original invoice's total | "A credit note cannot exceed the original invoice's total." |
| Billing-completeness-both-sides | freelancer business details, client billing details | Both sets must be complete; either one missing blocks sending, and each is reported by name so the freelancer knows which side to fix | See the two Field Validation Rules rows above -- the two messages are distinct so the freelancer is never told to fix the wrong side |

## Authorization Rules

Not applicable to this spec -- who may issue, view, or act on an invoice is governed entirely by FEAT-09.SPEC-006. This spec governs content correctness, not access.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| invoice_number | Next sequential integer after the highest ever assigned to this freelancer's invoices, across both automatic and manual creation | On create | No |
| issue_date | Current date and time at the moment of creation | On create | No |
| due_date | `issue_date` plus the freelancer's `default_payment_terms` (e.g., due on receipt, or within N days) | On create | Yes -- only on the manual-issuance form (FEAT-09.SPEC-003); the automatic path (FEAT-09.SPEC-004) always applies the derivation with no override step, since there is no freelancer interaction at generation |
| total | `amount` plus (`amount` × `tax_rate`), or `amount` alone if no tax line is configured | On create | No |
| tax_label, tax_rate | The project's configured tax line (FEAT-15), or none if unconfigured | On create | No -- Nadia changes this only through Currency & Tax Handling (FEAT-15), not on any FEAT-09 screen |

## Business Rules

- XBR-16: every invoice carries a due date from the freelancer's default payment terms (adjustable before sending), a unique sequential number, her business details, and the client's billing details; sending is blocked until both detail sets exist. This spec is the authoritative source of that gate for FEAT-09.
- XBR-17: currency and tax line are set once per project before the first invoice and applied exactly as configured; this spec never computes tax, only applies FEAT-15's configuration.
- Invoice numbering is a single sequence per freelancer that spans both the automatic (FEAT-09.SPEC-004) and manual (FEAT-09.SPEC-005) creation paths -- there is no separate numbering series for ad-hoc invoices or credit notes.
- A credit note receives its own new sequential number; it is never treated as a modification of the original invoice's number (this rule interacts with, and is reinforced by, FEAT-09.SPEC-008's immutability guarantee).

## Edge Cases

- **Two invoices for the same freelancer are created at effectively the same instant by different triggers (e.g., a milestone approval and an ad-hoc issuance)** -- Both draw from the same sequential numbering source; each is assigned a distinct, consecutive number with no gap and no collision, consistent with numbering being a single, freelancer-wide sequence rather than per-project.
- **A freelancer's business details are completed for the first time moments before a queued generation runs** -- The billing-completeness check reads current state at creation time, not at the moment the trigger originally fired, so a just-completed detail set is honored and generation proceeds.
- **The freelancer's default_payment_terms changes after an invoice's due_date has already been derived** -- The derivation is a one-time snapshot at creation; a later change to `default_payment_terms` never retroactively alters an already-created invoice's `due_date`.
- **Nadia enters a due date on the manual form that is earlier than the issue date** -- Accepted: the product defines no minimum-lead-time rule for a due date (a "due on receipt" term legitimately produces a due date equal to the issue date), so an earlier or same-day due date is valid.
- **A project's tax line is explicitly configured as "none"** -- `tax_label`/`tax_rate` are absent, `total` equals `amount` exactly, and the invoice's content correctly omits a tax line rather than showing a zero-rate line, which the product treats as a distinct, valid configuration.
- **The client's tax_id is left blank** -- Never blocks anything; `tax_id` is always optional on both sides of the billing-completeness gate.

## Acceptance Criteria

**FEAT-09.SPEC-007-AC-01:** Given a freelancer has issued four prior invoices numbered 1-4, when a fifth is created by either path, then it is numbered 5, with no gap and no reuse.

**FEAT-09.SPEC-007-AC-02:** Given a milestone approval triggers an invoice for a price of $500, when FEAT-09.SPEC-004 creates it, then the invoice's `amount` is exactly $500 plus the project's configured tax line.

**FEAT-09.SPEC-007-AC-03:** Given Nadia enters $0 as the amount on the manual-issuance form, when she attempts to submit, then she sees "Enter an amount greater than zero." and submission is blocked.

**FEAT-09.SPEC-007-AC-04:** Given a project has no configured currency, when a trigger attempts to generate its first invoice, then generation is blocked with "This project's currency isn't set yet. Set it before the first invoice."

**FEAT-09.SPEC-007-AC-05:** Given the Freelancer Account's business details are incomplete, when any invoice for that freelancer is created, then sending is blocked with "Your business details aren't complete yet. Add them in Settings before this invoice can be sent."

**FEAT-09.SPEC-007-AC-06:** Given the Client's billing details are incomplete, when any invoice for that client is created, then sending is blocked with "This client's billing details are incomplete. Add a billing name and address before sending an invoice."

**FEAT-09.SPEC-007-AC-07:** Given both business and billing details are complete, when an invoice is created, then it proceeds to `issue_date` and `due_date` assignment with no block.

**FEAT-09.SPEC-007-AC-08:** Given Nadia's default payment terms is "due within 14 days" and an invoice is generated automatically, when it is created, then `due_date` is set to 14 days after `issue_date` with no freelancer interaction.

**FEAT-09.SPEC-007-AC-09:** Given Nadia is issuing an ad-hoc invoice, when the form pre-fills the due date from her default terms, then she can adjust it to any other date before submitting, and the adjusted date is what gets recorded.

**FEAT-09.SPEC-007-AC-10:** Given Nadia clears the pre-filled due date on the manual form without entering another, when she attempts to submit, then she sees "Choose a due date." and submission is blocked.

**FEAT-09.SPEC-007-AC-11:** Given a credit note's entered amount exceeds the original invoice's total, when Nadia submits it, then she sees "A credit note cannot exceed the original invoice's total." and it is not recorded.

**FEAT-09.SPEC-007-AC-12:** Given a credit note's entered amount exactly equals the original invoice's total, when Nadia submits it, then it is accepted.

**FEAT-09.SPEC-007-AC-13:** Given a project's tax line is configured as none, when an invoice is created, then its `total` equals `amount` exactly, with no tax line shown.

**FEAT-09.SPEC-007-AC-14:** Given the client's tax_id is blank, when billing completeness is evaluated, then it never factors into the block.

**FEAT-09.SPEC-007-AC-15:** Given Nadia's default_payment_terms changes after an invoice was already created under the old terms, when she views that invoice, then its `due_date` remains exactly as originally derived, unaffected by the later change.

**FEAT-09.SPEC-007-AC-16:** Given Nadia enters a due date on the manual form that is the same day as the issue date, when she submits, then it is accepted as valid.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 9 | 9 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 0 (N/A -- owned by FEAT-09.SPEC-006) | 0 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
