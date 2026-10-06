---
document_type: spec
spec_type: automation
spec_id: FEAT-09.SPEC-004
spec_name: Automatic Invoice Generation
spec_slug: automatic-invoice-generation
parent_feature: FEAT-09
parent_feature_name: Invoice Generation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Automation Spec: Automatic Invoice Generation

## Overview

**Name:** Automatic Invoice Generation
**ID:** FEAT-09.SPEC-004
**Type:** Automation
**Purpose:** On a deposit acceptance, milestone approval, or project completion, generates the correct invoice with no freelancer action and hands it off to be sent.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending

## Scope and Non-Goals

**In Scope:**
- Receiving the deposit, next-invoice, and on-completion hand-offs from FEAT-03, FEAT-08, and FEAT-01 respectively
- Applying FEAT-09.SPEC-007's numbering, content, and due-date rules to produce a complete Invoice record
- Checking pay-link availability via FEAT-09.SPEC-009 and carrying its wording into the record
- Setting the Invoice to `Sent` and handing off to FEAT-09.SPEC-010 for the send notification
- Blocking generation, with a surfaced warning rather than silent failure, when the freelancer's or client's required billing details are missing

**Non-Goals:**
- Deciding whether a deposit, approval, or completion event itself occurred -- owned entirely by FEAT-03, FEAT-08, and FEAT-01, which fire this automation only after their own triggers are confirmed
- Ad-hoc invoicing or credit notes -- owned by FEAT-09.SPEC-005, a separate automation for issuance outside this schedule-driven path
- The actual email send and delivery/bounce status -- owned by the Transactional Email Delivery capability via FEAT-09.SPEC-010; this automation only hands off a ready invoice
- Currency conversion or tax calculation -- excluded per scope-boundaries.md (SC-16): this automation applies the project's already-configured currency and tax line (FEAT-15) rather than computing either

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Deposit due on acceptance | FEAT-03.SPEC-003 (Acceptance Recording) | Fires when the Payment Schedule, as it stood at acceptance, includes a deposit (XBR-01) | Project reference, Payment Schedule's deposit terms, accepted-at timestamp |
| Next invoice on milestone approval | FEAT-08.SPEC-004 (Next-Invoice Trigger) | Fires when the approved milestone's `payment_trigger` marks it as invoice-on-approval, per the schedule as it stood at approval (XBR-02) | Milestone reference, its price, Project reference, approval timestamp |
| On-completion invoice | FEAT-01.SPEC-006 (Completion Invoice Trigger) | Fires when the Payment Schedule's structure includes a `completion_amount`, at the moment a project is marked complete (XBR-03) | Project reference, Payment Schedule's completion_amount, completed_at timestamp |

## Processing Logic

1. Receive the triggering event (deposit, next-invoice, or on-completion) and its Project, amount source, and timestamp from the source spec.
2. Read the Project's currency and tax line (FEAT-15), the Client's billing details, and the Freelancer Account's business details and `default_payment_terms`.
3. Check that both the Freelancer Account's business details and the Client's billing details are present (XBR-16), and that the Project's currency (and tax line, if configured) is set (XBR-17). If the freelancer's business details are missing, stop and produce the "generation blocked -- missing freelancer business details" outcome. If the client's billing details are missing, stop and produce the "generation blocked -- missing client billing details" outcome. If the currency/tax configuration is missing, stop and produce the "generation blocked -- missing currency/tax configuration" outcome. Each reason is reported with its own exact message per FEAT-09.SPEC-007 -- never a single generic message.
4. Apply FEAT-09.SPEC-007's rules: assign the next sequential `invoice_number` for this freelancer, set `amount`, `currency`, `tax_label`, `tax_rate`, and `total` to match the triggering event's price plus tax exactly, set `issue_date` to now, and derive `due_date` from `default_payment_terms`.
5. Check pay-link availability via FEAT-09.SPEC-009 using the current Payment Account Connection status, and record the resulting wording state on the Invoice.
6. Create the Invoice record with `status: Generated`, `triggering_event` set to the source (deposit / milestone approval / completion), and a reference back to the Project and the triggering record (accepted proposal, approved milestone, or completed project).
7. Immediately transition the new Invoice to `status: Sent` and hand off to FEAT-09.SPEC-010 to send it to Owen with a confirmation copy to Nadia.
8. Return the outcome to the triggering spec for observability; this automation does not itself confirm email delivery -- that confirmation belongs to FEAT-09.SPEC-010.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Invoice generated and sent | Billing details are complete; generation and the send hand-off both succeed | New Invoice created with `status: Sent`, numbered, dated, and priced per FEAT-09.SPEC-007 | Owen and Nadia see the invoice arrive via FEAT-09.SPEC-010; the invoice appears in FEAT-09.SPEC-001 and FEAT-09.SPEC-002 with no visible delay from the triggering event | FEAT-09.SPEC-007, FEAT-09.SPEC-009, FEAT-09.SPEC-010, FEAT-09.SPEC-001, FEAT-09.SPEC-002 |
| Generation blocked -- missing freelancer business details | The Freelancer Account's business details are incomplete at the moment of firing | No Invoice created | Nadia sees a non-blocking warning on the Project, using FEAT-09.SPEC-007's exact freelancer-side message: "Your business details aren't complete yet. Add them in Settings before this invoice can be sent." with a link to the missing details' settings screen | FEAT-01.SPEC-005 (project-level warning), FEAT-21 (business details) |
| Generation blocked -- missing client billing details | The Client's billing details are incomplete at the moment of firing | No Invoice created | Nadia sees a non-blocking warning on the Project, using FEAT-09.SPEC-007's exact client-side message: "This client's billing details are incomplete. Add a billing name and address before sending an invoice." with a link to the missing details' settings screen | FEAT-01.SPEC-005 (project-level warning), FEAT-01 (client billing details) |
| Generation blocked -- missing currency/tax configuration | The Project's currency (and tax line) was never configured before the trigger fires | No Invoice created | Nadia sees a non-blocking warning on the Project, using FEAT-09.SPEC-007's exact message: "This project's currency isn't set yet. Set it before the first invoice." with a link to the currency/tax settings screen (FEAT-15) | FEAT-01.SPEC-005 (project-level warning), FEAT-15 (currency & tax configuration) |
| Send hand-off failure (invoice created, notification cannot be reached) | Invoice creation and numbering succeed, but FEAT-09.SPEC-010 cannot be reached at hand-off | Invoice remains created with `status: Sent` already set -- the send is retried, not the creation | The invoice appears immediately in Nadia's and Owen's lists and details even before the email arrives; the send is retried automatically without creating a duplicate invoice record, per the Brief's Side-Effect Inventory | FEAT-09.SPEC-010 (retried send) |
| No trigger applicable (defensive no-action) | The triggering spec fires this automation but its own conditions (deposit/next-invoice/completion price) resolve to zero or undefined | No Invoice created | Nothing is shown -- this path is a defensive guard; the triggering specs (FEAT-03.SPEC-003, FEAT-08.SPEC-004, FEAT-01.SPEC-006) already filter out non-invoicing cases before calling this automation, so this outcome is not expected to occur in normal operation | None |

## Data Model

**Reads:** Payment Schedule -- `structure`, `deposit_amount`, `completion_amount`, as they stood at the triggering moment. Milestone -- `price`, `payment_trigger`. Project -- currency and tax configuration (FEAT-15), reference. Client -- billing details. Freelancer Account -- business details, `default_payment_terms`. Payment Account Connection -- status, via FEAT-09.SPEC-009.
**Creates:** Invoice -- `invoice_number`, `project`, `triggering_event`, `amount`, `currency`, `tax_label`, `tax_rate`, `total`, business/billing details, `issue_date`, `due_date`, `status`, `pay_link availability`.
**Updates:** Invoice -- `status` (Generated to Sent, within this single automation run).
**Deletes:** None.

## Business Rules

- XBR-01, XBR-02, XBR-03: this automation is the single owner of invoice creation for all three automatic triggers; the source specs own firing the trigger, not the invoice itself.
- XBR-16: sending is blocked until both the freelancer's business details and the client's billing details exist -- enforced here as the "generation blocked" outcome, never a silent skip.
- XBR-17: currency and tax line are applied exactly as configured by FEAT-15 at the moment of generation; this automation computes no tax of its own.
- Amount must match the triggering milestone/deposit/completion price plus tax exactly (FEAT-09.SPEC-007) -- there is no rounding, discounting, or adjustment step in this automation.
- A failed send is retried without creating a duplicate invoice record, per the Brief's Side-Effect Inventory -- the Invoice's creation and its send are treated as separate steps for retry purposes, so a retried send never re-runs invoice creation.

## Edge Cases

- **The Project's currency or tax line was never configured before the first trigger fires** -- Generation is blocked with the dedicated "missing currency/tax configuration" outcome (XBR-17: currency and tax must be set before the first invoice); Nadia sees FEAT-09.SPEC-007's exact message: "This project's currency isn't set yet. Set it before the first invoice."
- **Nadia is offline when a trigger fires** -- Generation still proceeds automatically in the background; there is no freelancer action required and none is blocked by her connectivity, per the Brief's Side-Effect Inventory (Offline/Degraded: N/A for this automation).
- **The Payment Account Connection has no connected account at generation time** -- The invoice still generates and sends; FEAT-09.SPEC-009 supplies the "online payment not yet available" wording rather than blocking the send.
- **Two different triggers for the same project fire at effectively the same time (e.g., a milestone approval and a project completion within the same instant)** -- Concurrent trigger firing: each trigger's source spec calls this automation independently with its own amount and triggering-event reference; this automation assigns each a separate, sequentially numbered Invoice, since invoice numbering is a single, freelancer-wide sequence that serializes concurrent assignments rather than colliding on one number.
- **This automation is invoked for a new trigger while a previous invocation for a different trigger on the same project is still in flight** -- Each run is independent and reads only the data relevant to its own trigger; the only shared resource is the sequential `invoice_number` counter, which FEAT-09.SPEC-007 guarantees assigns uniquely even under concurrent runs.
- **The billing-details check passes, but the Client's billing details are cleared by Nadia between the trigger firing and this automation's read** -- Extremely narrow window; if the read in step 3 finds the details missing, the outcome is "generation blocked," consistent with the check being evaluated at read time, not at trigger time.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03.SPEC-003 (Acceptance Recording) | Triggered by (inbound) | Fires the deposit-invoice trigger on acceptance |
| FEAT-08.SPEC-004 (Next-Invoice Trigger) | Triggered by (inbound) | Fires the next-invoice trigger on approval |
| FEAT-01.SPEC-006 (Completion Invoice Trigger) | Triggered by (inbound) | Fires the on-completion trigger |
| FEAT-09.SPEC-007 (Invoice Content, Numbering, Amount & Due-Date Rules) | References (inbound) | Numbering, content, and due-date rules applied at creation |
| FEAT-09.SPEC-009 (Pay-Link Availability & No-Account Fallback Rule) | References (inbound) | Determines the pay-link wording carried onto the new invoice |
| FEAT-09.SPEC-010 (Invoice Issued & Copy Confirmation Notification) | Triggers (outbound) | Sends the resulting invoice to Owen and its copy to Nadia |
| FEAT-01.SPEC-005 (Project Detail) | Affects (outbound) | Shows the non-blocking warning when generation is blocked |
| FEAT-15 (Currency & Tax Handling) | References (inbound) | Supplies the currency and tax configuration applied |
| FEAT-21 (Settings & Account Management) | References (inbound) | Supplies the freelancer's business details and default payment terms |

## Analytics and Success Signals

- **invoice_generated** (triggering event type, invoice reference) -- supports success-metrics.md: "Invoice Auto-Generation Accuracy"
- **invoice_sent** (invoice reference, triggering event type) -- N/A -- no success-metrics.md metric is connected to the send moment itself distinct from generation accuracy; retained per product-features.md's Signals field, which names `invoice_sent` explicitly for this feature
- **invoice_generation_blocked** (reason: missing_freelancer_details / missing_client_details / missing_currency_tax_config) -- N/A -- no Stage 2 metric measures blocked generations directly; retained so a billing-completeness gap is observable rather than silently absorbed, since "Invoice Auto-Generation Accuracy" measures the correctness of invoices that were generated, not ones that were blocked
- **invoice_send_handoff_retried** (retry_count) -- N/A -- no connected metric covers infrastructure hand-off retries; retained as this automation's only signal of that failure mode, mirroring the pattern used by FEAT-08.SPEC-004's `milestone_invoice_handoff_failed`

## Acceptance Criteria

**FEAT-09.SPEC-004-AC-01:** Given a proposal is accepted and the Payment Schedule includes a deposit, when FEAT-03.SPEC-003 fires this automation, then a deposit invoice is generated with the correct amount, numbered sequentially, and immediately sent to Owen.

**FEAT-09.SPEC-004-AC-02:** Given a milestone is approved and its `payment_trigger` marks it to invoice on approval, when FEAT-08.SPEC-004 fires this automation, then the next invoice is generated for that milestone's price and sent.

**FEAT-09.SPEC-004-AC-03:** Given a project is marked complete and the schedule includes a `completion_amount`, when FEAT-01.SPEC-006 fires this automation, then the on-completion invoice is generated and sent.

**FEAT-09.SPEC-004-AC-04:** Given the Freelancer Account's business details are incomplete at the moment of firing, when this automation runs, then no invoice is created and Nadia sees the non-blocking warning "Your business details aren't complete yet. Add them in Settings before this invoice can be sent."

**FEAT-09.SPEC-004-AC-05:** Given the Client's billing details are incomplete at the moment of firing, when this automation runs, then no invoice is created and Nadia sees the non-blocking warning "This client's billing details are incomplete. Add a billing name and address before sending an invoice."

**FEAT-09.SPEC-004-AC-06:** Given Nadia has no connected Payment Account Connection at generation time, when the invoice is generated, then it still issues and sends, carrying the "online payment not yet available" wording from FEAT-09.SPEC-009.

**FEAT-09.SPEC-004-AC-07:** Given the invoice is created successfully but the send hand-off to FEAT-09.SPEC-010 fails, when the failure occurs, then the invoice remains visible in Nadia's and Owen's lists, and the send is retried automatically without creating a second invoice record.

**FEAT-09.SPEC-004-AC-08:** Given a milestone approval and a project completion fire at effectively the same time for the same project, when both invocations run, then each produces its own separately numbered invoice with no collision.

**FEAT-09.SPEC-004-AC-09:** Given Nadia is offline when a trigger fires, when the trigger fires, then the invoice still generates and sends with no freelancer action required.

**FEAT-09.SPEC-004-AC-10:** Given the project's currency and tax line were never configured, when a trigger fires, then generation is blocked with the warning "This project's currency isn't set yet. Set it before the first invoice."

**FEAT-09.SPEC-004-AC-11:** Given the deposit invoice generates successfully, when the invoice_generated event is emitted, then it carries the triggering event type and the invoice reference.

**FEAT-09.SPEC-004-AC-12:** Given generation is blocked for missing billing details, when the invoice_generation_blocked event is emitted, then it carries the specific reason (missing_freelancer_details / missing_client_details / missing_currency_tax_config).

**FEAT-09.SPEC-004-AC-13:** Given this automation runs for a second trigger while a first trigger's run on the same project is still completing its send hand-off, when both runs execute, then neither is blocked by the other, since each reads only its own triggering data and shares no state beyond the sequential invoice-numbering counter.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (deposit, next-invoice, completion) | 3 |
| Outcome Paths | 6 (generated & sent, blocked -- missing freelancer details, blocked -- missing client details, blocked -- missing currency/tax, hand-off failure, no-action) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
