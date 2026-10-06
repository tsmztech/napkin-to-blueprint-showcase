---
document_type: spec
spec_type: automation
spec_id: FEAT-09.SPEC-005
spec_name: Manual Invoice & Credit Note Recording
spec_slug: manual-invoice-credit-note-recording
parent_feature: FEAT-09
parent_feature_name: Invoice Generation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Manual Invoice & Credit Note Recording

## Overview

**Name:** Manual Invoice & Credit Note Recording
**ID:** FEAT-09.SPEC-005
**Type:** Automation
**Purpose:** Validates and writes an ad-hoc invoice or a credit note submitted from FEAT-09.SPEC-003, marking the original invoice Corrected when a credit note supersedes it.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending

## Scope and Non-Goals

**In Scope:**
- Validating and recording an ad-hoc invoice submitted from FEAT-09.SPEC-003
- Validating and recording a credit note against a specific prior invoice, applying the immutability and correction rules
- Marking the original invoice's `status: Corrected` at the moment its credit note is recorded
- Handing off the newly recorded invoice or credit note to FEAT-09.SPEC-010 for sending

**Non-Goals:**
- Collecting the form input itself -- owned entirely by FEAT-09.SPEC-003; this automation begins at submission
- Automatic invoicing from the payment schedule -- owned by FEAT-09.SPEC-004, a separate automation with its own triggers
- Deciding whether Nadia is authorized to issue invoices at all -- owned by FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules); this automation assumes the submitting user already passed that check on FEAT-09.SPEC-003
- Refund or reversal recording -- excluded per scope-boundaries.md (SC-18): a credit note corrects the invoice record itself, but the actual funds movement behind a refund is owned by Refund & Cancelled Project Handling (FEAT-25)

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Ad-hoc invoice submitted | FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance, ad-hoc mode) | Fires when Nadia submits the ad-hoc form after client-side validation passes | Project reference, description, amount, chosen due date |
| Credit note submitted | FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance, credit-note mode) | Fires when Nadia submits the credit-note form after client-side validation passes | Original invoice reference, credit amount, reason |

## Processing Logic

1. Receive the submission (ad-hoc invoice or credit note) and its data from FEAT-09.SPEC-003.
2. Read the Freelancer Account's business details and the Client's billing details; if the freelancer's business details are incomplete, stop and produce the "recording blocked -- missing freelancer business details" outcome; if the client's billing details are incomplete, stop and produce the "recording blocked -- missing client billing details" outcome (same gate as FEAT-09.SPEC-004, applied here for the manual path, with the same side-specific exact messages from FEAT-09.SPEC-007).
3. **Ad-hoc path:** apply FEAT-09.SPEC-007's rules to assign the next sequential `invoice_number`, set `issue_date` to now, set `due_date` to the value Nadia chose (already validated against the default-terms derivation on FEAT-09.SPEC-003), and set `amount`/`currency`/`tax_label`/`tax_rate`/`total` from the entered amount plus the project's configured tax line.
4. **Credit-note path:** re-check at write time that the original invoice's current `status` is not already `Corrected` (guards against a race with another correction). If it is already Corrected, stop and produce the "already corrected" outcome. Otherwise, validate the credit amount does not exceed the original invoice's `total` (FEAT-09.SPEC-007); if it does, stop and produce the "credit amount exceeds original" outcome.
5. **Credit-note path (continued):** create a new Invoice record with `triggering_event: correction`, a reference back to the original invoice, `amount`/`currency` matching the credit amount and the original's currency, `total` set to the negative of the credit amount (a credit rather than a charge), and the entered reason carried as visible content. Assign it the next sequential `invoice_number` per FEAT-09.SPEC-007 -- a credit note is its own numbered record, never a silent adjustment to the original.
6. **Credit-note path (continued):** set the original invoice's `status` to `Corrected` in the same atomic step that creates the credit note, per FEAT-09.SPEC-008.
7. Set the newly recorded invoice or credit note's `status` to `Sent` and hand off to FEAT-09.SPEC-010 for sending to Owen with a confirmation copy to Nadia.
8. Return the outcome to FEAT-09.SPEC-003, which navigates to FEAT-09.SPEC-002 for the newly recorded record on success, or displays the specific rejection reason on failure.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Ad-hoc invoice recorded and sent | Billing details complete; ad-hoc validation passes | New Invoice created (`status: Sent`), numbered and dated per FEAT-09.SPEC-007 | FEAT-09.SPEC-003 navigates Nadia to FEAT-09.SPEC-002 for the new invoice; Owen and Nadia receive FEAT-09.SPEC-010's notification | FEAT-09.SPEC-003, FEAT-09.SPEC-002, FEAT-09.SPEC-010 |
| Credit note recorded, original corrected | Billing details complete; original invoice not already Corrected; credit amount within the original's total | New credit-note Invoice created (`status: Sent`); original invoice's `status` set to `Corrected` | FEAT-09.SPEC-003 navigates Nadia to FEAT-09.SPEC-002 for the credit note; the original invoice's detail now shows `Corrected` with a link to the credit note; Owen and Nadia receive FEAT-09.SPEC-010's notification | FEAT-09.SPEC-003, FEAT-09.SPEC-002, FEAT-09.SPEC-010 |
| Recording blocked -- missing freelancer business details | Freelancer business details incomplete at write time | No record created or updated | FEAT-09.SPEC-003 shows FEAT-09.SPEC-007's exact freelancer-side message: "Your business details aren't complete yet. Add them in Settings before this invoice can be sent." with a link to the relevant settings screen -- rendered identically to FEAT-09.SPEC-004's automatic-path equivalent | FEAT-09.SPEC-003 |
| Recording blocked -- missing client billing details | Client billing details incomplete at write time | No record created or updated | FEAT-09.SPEC-003 shows FEAT-09.SPEC-007's exact client-side message: "This client's billing details are incomplete. Add a billing name and address before sending an invoice." with a link to the relevant settings screen -- rendered identically to FEAT-09.SPEC-004's automatic-path equivalent | FEAT-09.SPEC-003 |
| Already corrected (race) | The original invoice's `status` is already `Corrected` by the time this write is attempted | No record created or updated | FEAT-09.SPEC-003 shows "This invoice was already corrected. View the existing credit note." and links to it | FEAT-09.SPEC-003, FEAT-09.SPEC-002 |
| Credit amount exceeds original | Credit amount is greater than the original invoice's `total` | No record created or updated | FEAT-09.SPEC-003 shows "A credit note cannot exceed the original invoice's total." | FEAT-09.SPEC-003 |
| Recording failure (connectivity or processing error) | The write itself does not complete | No partial record left in either path | FEAT-09.SPEC-003 shows a retry option with entered data preserved | FEAT-09.SPEC-003 |

## Data Model

**Reads:** Invoice (credit-note path) -- `status`, `total`, `currency` of the original. Freelancer Account -- business details. Client -- billing details.
**Creates:** Invoice -- ad-hoc invoice or credit note, per FEAT-09.SPEC-007's numbering and content rules.
**Updates:** Invoice (credit-note path only) -- the original's `status` set to `Corrected`, written exactly once and never altered afterward.
**Deletes:** None.

## Business Rules

- FEAT-09.SPEC-007 governs numbering, required content, amount validation, and due-date derivation for both paths -- applied here, not re-derived.
- FEAT-09.SPEC-008: a sent invoice is never silently edited; a credit note is always a new, linked Invoice record, and marking the original `Corrected` never alters any of the original's other fields (XBR-04).
- The original-invoice re-check (step 4) is authoritative over the screen-time state FEAT-09.SPEC-003 displayed -- consistent with the pattern used elsewhere in this pipeline (e.g., FEAT-03.SPEC-003's write-time re-check) for guaranteeing exactly-once correction.
- Both paths apply the same billing-completeness gate as the automatic path (FEAT-09.SPEC-004), since a manually issued invoice carries the same compliance requirements as an automatically generated one (XBR-16).

## Edge Cases

- **Two credit notes are submitted against the same invoice from two open sessions at effectively the same time** -- Concurrent trigger firing: both invocations reach step 4 near-simultaneously, but only one can win the atomic update in step 6. The first to complete finds the original not yet Corrected and proceeds; the second's re-check then finds `status: Corrected` and returns the "already corrected" outcome. Neither run blocks the other; there is no queuing.
- **Nadia submits a credit note, then submits a second one against the same invoice before the first finishes processing** -- Trigger fires while a previous run is in flight: FEAT-09.SPEC-003 disables Submit while a submission is processing (screen-level debounce), so a second automation run for the same original invoice from the same session cannot start until the first resolves; if it did reach this automation regardless, the same atomic-update guarantee as the concurrent case applies.
- **An ad-hoc invoice is submitted for a project whose currency was never configured** -- Blocked with the same "missing currency/tax configuration" outcome FEAT-09.SPEC-004 uses on the automatic path (XBR-17); Nadia sees FEAT-09.SPEC-007's exact message: "This project's currency isn't set yet. Set it before the first invoice."
- **The credit amount exactly equals the original invoice's total** -- Passes validation (the boundary is inclusive): a full-value credit note is accepted and the original is marked Corrected.
- **The send hand-off to FEAT-09.SPEC-010 fails after recording succeeds** -- The Invoice record (ad-hoc invoice or credit note, and the original's Corrected status) is not rolled back, since it is already evidentiary per XBR-04; the send is retried by FEAT-09.SPEC-010's own retry handling without creating a duplicate record.
- **Nadia submits a credit note against an invoice that has since been paid** -- Recording still proceeds and the original is marked Corrected alongside its existing Paid history; reconciling the payment against the correction (e.g., issuing an actual refund) is a separate action Nadia takes through FEAT-25 from the invoice detail screen, outside this automation's scope.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-09.SPEC-003 (Manual Invoice & Credit Note Issuance) | Triggered by (inbound) | Both submission paths fire this automation |
| FEAT-09.SPEC-007 (Invoice Content, Numbering, Amount & Due-Date Rules) | References (inbound) | Numbering, content, and due-date rules applied at write time |
| FEAT-09.SPEC-008 (Invoice Immutability & Correction Rules) | References (inbound) | Governs the credit-note-as-new-record and original-marked-Corrected behavior |
| FEAT-09.SPEC-010 (Invoice Issued & Copy Confirmation Notification) | Triggers (outbound) | Sends the newly recorded invoice or credit note |
| FEAT-09.SPEC-002 (Invoice Detail) | Affects (outbound) | Displays the recorded result, and the original's Corrected status with a link to its credit note |
| FEAT-25 (Refund & Cancelled Project Handling) | References (outbound) | A credit note against a paid invoice may lead Nadia to a separate refund action there |

## Analytics and Success Signals

- **invoice_manually_issued** (project reference, amount) -- N/A -- no success-metrics.md metric tracks ad-hoc issuance directly; retained per product-features.md's Signals field, which names `invoice_manually_issued` explicitly for this feature
- **invoice_correction_issued** (original invoice reference, credit amount) -- N/A -- no success-metrics.md metric tracks correction volume; retained per product-features.md's Signals field, which names `invoice_correction_issued` explicitly, and because FEAT-09.SPEC-008's immutability guarantee is only verifiable in practice if corrections are observable
- **manual_recording_blocked** (path: ad_hoc / credit_note; reason: missing_billing_details / already_corrected / amount_exceeds_original) -- N/A -- no connected Stage 2 metric measures blocked manual submissions; retained so a rejected manual path is distinguishable from a successful one in product telemetry, mirroring FEAT-09.SPEC-004's `invoice_generation_blocked`

## Acceptance Criteria

**FEAT-09.SPEC-005-AC-01:** Given Nadia submits a valid ad-hoc invoice with billing details complete, when this automation runs, then a new invoice is created, numbered sequentially, and sent to Owen.

**FEAT-09.SPEC-005-AC-02:** Given Nadia submits a credit note against a Sent invoice not already Corrected, when this automation runs, then a new credit-note invoice is created and the original invoice's status is set to Corrected.

**FEAT-09.SPEC-005-AC-03:** Given the Freelancer Account's business details are incomplete, when Nadia submits an ad-hoc invoice, then recording is blocked with "Your business details aren't complete yet. Add them in Settings before this invoice can be sent."

**FEAT-09.SPEC-005-AC-04:** Given the credit amount exceeds the original invoice's total, when Nadia submits the credit note, then it is rejected with "A credit note cannot exceed the original invoice's total."

**FEAT-09.SPEC-005-AC-05:** Given the credit amount exactly equals the original invoice's total, when Nadia submits it, then it passes validation and the original is marked Corrected.

**FEAT-09.SPEC-005-AC-06:** Given the original invoice was already corrected by another session before this write runs, when this automation's re-check executes, then it returns "already corrected" and no second credit note is created.

**FEAT-09.SPEC-005-AC-07:** Given two credit notes against the same invoice are submitted at effectively the same time, when both invocations reach the write-time re-check, then exactly one succeeds and the other returns "already corrected."

**FEAT-09.SPEC-005-AC-08:** Given recording succeeds, when the invoice or credit note is ready, then this automation hands off to FEAT-09.SPEC-010 for sending.

**FEAT-09.SPEC-005-AC-09:** Given the send hand-off fails after recording succeeds, when the failure occurs, then the recorded invoice or credit note is unaffected and the send is retried without duplicating the record.

**FEAT-09.SPEC-005-AC-10:** Given Nadia submits a credit note against an invoice that has already been paid, when recording succeeds, then the original is marked Corrected alongside its existing Paid history, with no automatic refund action taken.

**FEAT-09.SPEC-005-AC-11:** Given the ad-hoc invoice is recorded successfully, when the invoice_manually_issued event is emitted, then it carries the project reference and amount.

**FEAT-09.SPEC-005-AC-12:** Given a manual submission is blocked for any reason, when the manual_recording_blocked event is emitted, then it carries the specific path and reason.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (ad-hoc, credit note) | 2 |
| Outcome Paths | 7 (ad-hoc recorded, credit note recorded, blocked -- missing freelancer details, blocked -- missing client details, already corrected, credit exceeds original, recording failure) | 7 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
