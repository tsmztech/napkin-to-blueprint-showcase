---
document_type: spec
spec_type: automation
spec_id: FEAT-15.SPEC-008
spec_name: Invoice Currency & Tax Line Application
spec_slug: invoice-currency-tax-line-application
parent_feature: FEAT-15
parent_feature_name: Currency & Tax Handling
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Invoice Currency & Tax Line Application

## Overview

**Name:** Invoice Currency & Tax Line Application
**ID:** FEAT-15.SPEC-008
**Type:** Automation
**Purpose:** At invoice generation, stamps the project's configured currency and tax label/rate onto the new invoice, computes the tax amount and total, and flags a mismatch if the invoice's currency would not match the project's current configuration.
**Parent Feature:** FEAT-15 -- Currency & Tax Handling

## Scope and Non-Goals

**In Scope:**
- Reading the project's configured currency, tax label, and tax rate at the moment an invoice is generated
- Applying FEAT-15.SPEC-003's validation as a precondition -- blocking generation on an invalid or missing configuration
- Stamping currency, tax label, and tax rate onto the new invoice, and computing the tax amount and total from the subtotal
- Detecting and flagging a currency mismatch between the invoice being generated and the project's currently configured currency

**Non-Goals:**
- Deciding when an invoice is generated at all (deposit, milestone approval, completion, ad hoc, or correction) -- owned entirely by Invoice Generation & Sending (FEAT-09); this automation only reacts to the generation event and applies currency/tax to whatever invoice FEAT-09 is creating
- Computing the invoice subtotal itself (the amount before tax) -- owned entirely by FEAT-09 and its triggering source (Payment Schedule, Milestone, or ad hoc entry); this automation reads the subtotal as an input and applies tax to it
- Locking the project's currency/tax after the first invoice -- owned entirely by FEAT-15.SPEC-004; this automation triggers that lock's evaluation by being the point at which "first invoice sent" becomes possible, but the lock rule itself lives there
- Automatic, jurisdiction-specific tax calculation -- excluded per scope-boundaries.md (SC-16): this automation applies the freelancer-configured label and rate exactly as stored; it computes no rate of its own

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Invoice generated (deposit) | FEAT-09 (Invoice Generation & Sending), fired from acceptance per XBR-01 | Fires whenever FEAT-09 creates a new Invoice record from a proposal acceptance with a deposit in the payment schedule | Project reference, invoice subtotal, triggering event type |
| Invoice generated (milestone approval) | FEAT-09, fired from milestone approval per XBR-02 | Fires whenever FEAT-09 creates a new Invoice record from an approved milestone | Project reference, invoice subtotal, triggering event type |
| Invoice generated (project completion) | FEAT-09, fired from marking a project complete per XBR-03 | Fires whenever FEAT-09 creates a new Invoice record from project completion | Project reference, invoice subtotal, triggering event type |
| Invoice generated (ad hoc) | FEAT-09 | Fires whenever Nadia issues an ad hoc invoice outside the schedule | Project reference, invoice subtotal, triggering event type |
| Invoice generated (correction) | FEAT-09, fired from a credit note or corrected invoice | Fires whenever FEAT-09 creates a new Invoice record as a correction to a prior invoice | Project reference, invoice subtotal, triggering event type, reference to the invoice being corrected |

## Processing Logic

1. Receive the new invoice's project reference and subtotal from the triggering generation event (FEAT-09).
2. Read the project's currently configured `currency`, `tax_label`, and `tax_rate` (or "no tax line").
3. Apply FEAT-15.SPEC-003's validation to the project's current configuration as a precondition: if the configuration is invalid or was never set, halt processing and signal the blocking outcome back to FEAT-09 before any invoice record is finalized.
4. If the configuration is valid, compare the invoice's expected currency (the one the triggering event assumed, e.g., what a proposal or payment schedule was denominated in) against the project's currently configured currency.
5. If the two currencies do not match, flag the discrepancy (do not silently substitute either value) and proceed to the mismatch-flagged outcome rather than the success outcome.
6. If the two currencies match (the normal case), stamp the invoice's `currency`, `tax_label`, and `tax_rate` fields with the project's configured values.
7. Compute the tax amount: the subtotal (`amount`) multiplied by `tax_rate` (or zero, when "no tax line" applies). The tax amount itself is a computed figure used to derive the total and to display the tax line on the invoice; it is not a separately stored Invoice field beyond the `tax_label`/`tax_rate` pair that determines it.
8. Compute the invoice total: `amount` plus the computed tax amount.
9. Write the computed `total` onto the invoice record alongside the stamped `currency`, `tax_label`, and `tax_rate` fields.
10. Signal completion back to FEAT-09, which proceeds with the remainder of invoice generation (numbering, due date, sending).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Applied successfully | Project has a valid, complete currency/tax configuration and the invoice's expected currency matches it | Invoice's `currency`, `tax_label`, `tax_rate`, and `total` are set (tax amount is computed as part of deriving `total` and displayed from `amount` and `tax_rate`, not stored separately) | None directly -- the invoice simply carries correct values when Nadia or Owen next views it (FEAT-09.SPEC-002) | FEAT-09 (proceeds with generation), FEAT-09.SPEC-002 (displays the result) |
| Generation blocked -- no valid configuration | Project's currency/tax configuration is missing or fails FEAT-15.SPEC-003's validation | No invoice record is finalized | Nadia sees the empty-configuration prompt on FEAT-15.SPEC-001, per that screen's Empty state, directing her to complete configuration before generation can proceed | FEAT-15.SPEC-001 (empty state), FEAT-09 (generation halted) |
| Currency mismatch flagged | The invoice's expected currency does not match the project's currently configured currency at the moment of generation (e.g., a race between a configuration change and a concurrent generation) | Invoice is created with the `invoice_currency_mismatch_flagged` signal recorded against it rather than silently issuing a mismatched invoice; the invoice is not automatically sent | Nadia sees a flagged-invoice indicator on the invoice before it can be sent, prompting her to review and confirm the correct currency before sending | FEAT-09 (generation proceeds to a held state rather than auto-send), FEAT-09.SPEC-002 (shows the flagged indicator) |
| Automation failure (processing error unrelated to configuration validity) | Reading the project's configuration or computing the tax amount/total fails for a reason other than an invalid configuration | No invoice record is finalized | Nadia sees a generation failure notice from FEAT-09 with a retry option; the failure is not attributed to her configuration, since the configuration was never evaluated to invalid | FEAT-09 (generation retried or surfaced as a failure) |

## Data Model

**Reads:** Project -- `currency`, `tax_label`, `tax_rate`. Invoice (the one being generated) -- the subtotal and expected currency supplied by the triggering event from FEAT-09.
**Creates:** None -- the Invoice record itself is created by FEAT-09; this automation supplies field values onto it during that creation.
**Updates:** Invoice -- `currency`, `tax_label`, `tax_rate`, `total` (this is this feature's one Update to Invoice, per the dependency map's Invoice lifecycle line: "Updated by ... FEAT-15 (currency and tax fields at generation)"). The tax amount is computed from `amount` and `tax_rate` for the total and for the invoice's displayed tax line; it is not a separately stored field.
**Deletes:** None.

## Business Rules

- XBR-17: proposal prices are in the project's already-configured currency, and this automation is the point at which that same configuration is applied to every invoice for the project.
- This automation enforces FEAT-15.SPEC-003 as a precondition -- it never stamps a configuration that would fail validation, and it never invents a default when configuration is missing.
- The tax amount is always computed as subtotal times tax rate, with "no tax line" treated as a tax rate of zero for this computation only (the invoice's `tax_label` remains genuinely absent, not "0%", to preserve the distinction FEAT-15.SPEC-003 establishes between a zero-rated line and no tax line at all).
- A currency mismatch is never resolved automatically by picking one value over the other -- XBR-18's non-conversion rule means there is no computation that could reconcile two different currencies, so the only safe response is to flag and hold, never to guess.
- This automation is also the trigger point that makes FEAT-15.SPEC-004's lock possible: it is the successful "applied" outcome, once the resulting invoice reaches Sent, that fires the first-invoice lock for the project.

## Edge Cases

- **The project's configuration changes between when the triggering event (e.g., milestone approval) captured the expected currency and when this automation actually runs** -- This is exactly the mismatch condition: flagged, not silently applied, per the Outcome Definitions above.
- **The project has "no tax line" configured and the subtotal is a whole number** -- Tax amount computes to zero, total equals the subtotal exactly, and the invoice carries no `tax_label` or `tax_rate` value at all (not a zero value).
- **The subtotal is zero (e.g., a fully-discounted ad hoc invoice)** -- Tax amount computes to zero regardless of the configured tax rate (zero times any rate is zero); the total is zero. No error is raised for a zero subtotal by this automation.
- **Two invoice-generation triggers fire for the same project at effectively the same time (e.g., a milestone approval and an ad hoc invoice issued moments apart)** -- Each generation event runs this automation independently against its own invoice record; both read the project's configuration at their own respective moments, so each is internally consistent even if the two invoices could theoretically read different configuration snapshots (which itself could only happen if a change landed on the project between the two runs, in which case the later one would be evaluated against the newer configuration).
- **This automation's run for one invoice is still in flight when a second generation trigger fires for the same project** -- The two runs process independently and do not queue behind each other; each reads the project's configuration at its own start time and writes only to its own invoice record, so neither run's completion depends on the other's.
- **A correction (credit note) is generated for an already-locked project** -- The project's currency/tax remains as locked by FEAT-15.SPEC-004; this automation applies that same locked configuration to the credit note exactly as it would to any other invoice on that project, since a locked project always has a valid, unchanging configuration to read.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-003 (Currency & Tax Validation Rules) | References (inbound) | Supplies the precondition check this automation applies before stamping any invoice |
| FEAT-15.SPEC-001 (Project Currency & Tax Configuration) | Affects (outbound) | The empty-configuration and validation-failure outcomes surface on that screen |
| FEAT-15.SPEC-004 (Currency & Tax Lock After First Invoice) | Triggers (outbound) | A successful application whose resulting invoice reaches Sent is the event that fires the first-invoice lock |
| FEAT-09 (Invoice Generation & Sending) | Triggered by (inbound) | Every invoice-generation trigger (deposit, milestone approval, completion, ad hoc, correction) invokes this automation |
| FEAT-09 (Invoice Generation & Sending) | Affects (outbound) | Supplies the stamped currency/tax fields, computed tax amount, and total that FEAT-09's invoice record and detail screen (FEAT-09.SPEC-002) display |

## Analytics and Success Signals

- **currency_tax_applied** (project reference, invoice reference, currency, tax treatment applied) -- supports success-metrics.md: "Invoice Currency and Tax Correctness"
- **invoice_currency_mismatch_flagged** (project reference, invoice reference, expected currency, project's currently configured currency) -- supports success-metrics.md: "Invoice Currency and Tax Correctness" (a flagged mismatch is exactly the failure mode this metric measures the absence of)
- **currency_tax_application_blocked** (project reference, reason: missing configuration / invalid configuration) -- supports success-metrics.md: "Invoice Currency and Tax Correctness" (a blocked generation is the automation successfully preventing an incorrect invoice from ever being issued)

## Acceptance Criteria

**FEAT-15.SPEC-008-AC-01:** Given a project has a valid currency and tax configuration, when its next milestone approval triggers invoice generation, then the new invoice is stamped with that currency, tax label, and tax rate, and its tax amount and total are computed from the subtotal.

**FEAT-15.SPEC-008-AC-02:** Given a project has "no tax line" configured, when an invoice is generated for it, then the tax amount is zero, the total equals the subtotal, and no `tax_label` or `tax_rate` value is set.

**FEAT-15.SPEC-008-AC-03:** Given a project has never had currency/tax configured, when a generation trigger fires for it, then generation is blocked and Nadia sees the empty-configuration prompt on FEAT-15.SPEC-001.

**FEAT-15.SPEC-008-AC-04:** Given a project's configuration changes after a triggering event captured its expected currency but before this automation runs, when the mismatch is detected, then the resulting invoice is created with `invoice_currency_mismatch_flagged` set and is not automatically sent.

**FEAT-15.SPEC-008-AC-05:** Given a subtotal of zero on an ad hoc invoice, when this automation computes tax and total, then both are zero and no error is raised.

**FEAT-15.SPEC-008-AC-06:** Given two generation triggers fire for the same project at effectively the same time, when both run this automation, then each processes its own invoice independently and neither is blocked by the other.

**FEAT-15.SPEC-008-AC-07:** Given this automation's run for one invoice is still in flight, when a second generation trigger fires for the same project, then the second run proceeds independently rather than queuing behind the first.

**FEAT-15.SPEC-008-AC-08:** Given the accepted proposal's deposit trigger fires per XBR-01, when this automation applies currency and tax to the resulting deposit invoice, then the applied values match the project's configuration at that moment.

**FEAT-15.SPEC-008-AC-09:** Given a project's currency/tax is already locked, when a credit note is generated for it, then this automation applies the same locked configuration to the credit note exactly as it would to any other invoice on that project.

**FEAT-15.SPEC-008-AC-10:** Given this automation successfully applies currency and tax to an invoice, when the resulting invoice is later sent as the project's first invoice, then that Sent event is the trigger FEAT-15.SPEC-004 uses to lock the project's configuration.

**FEAT-15.SPEC-008-AC-11:** Given a processing error unrelated to configuration validity occurs (e.g., the configuration read itself fails), when this automation cannot complete, then Nadia sees a generation failure notice from FEAT-09 with a retry option, distinct from the empty-configuration message.

**FEAT-15.SPEC-008-AC-12:** Given this automation successfully applies currency and tax, when the application completes, then the currency_tax_applied event is emitted with the project and invoice references and the values applied.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 5 (deposit, milestone approval, completion, ad hoc, correction) | 5 |
| Outcome Paths | 4 (applied successfully, blocked -- no configuration, mismatch flagged, automation failure) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
