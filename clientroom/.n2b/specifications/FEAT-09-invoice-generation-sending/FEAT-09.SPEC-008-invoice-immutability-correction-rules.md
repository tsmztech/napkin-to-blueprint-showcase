---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-09.SPEC-008
spec_name: Invoice Immutability & Correction Rules
spec_slug: invoice-immutability-correction-rules
parent_feature: FEAT-09
parent_feature_name: Invoice Generation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 4
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Invoice Immutability & Correction Rules

## Overview

**Name:** Invoice Immutability & Correction Rules
**ID:** FEAT-09.SPEC-008
**Type:** Logic/Rule
**Purpose:** Enforces that a sent invoice is never silently edited and that a correction is always a visible credit note or a new invoice.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending
**Governed Entity:** Invoice (specifically the `status` field's immutability boundary and the correction relationship it creates)

## Scope and Non-Goals

**In Scope:**
- The boundary at which an invoice becomes immutable (the moment `status` moves to `Sent` or later)
- The rule that a correction is always a new, linked Invoice record (a credit note), never an in-place edit
- Marking the original invoice `Corrected` at the exact moment its credit note is recorded
- What every screen that would otherwise offer an edit control must show instead once an invoice is Sent or later

**Non-Goals:**
- Deciding whether Nadia is authorized to issue a correction at all -- owned by FEAT-09.SPEC-006 (Invoice Access & Role Authorization Rules)
- The content and numbering a credit note carries -- owned by FEAT-09.SPEC-007
- Payment-status transitions (Paid, Overdue, Refunded, Disputed) -- owned by FEAT-10, FEAT-11, and FEAT-25 respectively; this spec governs only the Generated/Sent/Corrected transitions this feature itself owns
- Deleting or archiving an invoice -- excluded per the Brief's Entity-Lifecycle Coverage Matrix: this feature never deletes or archives an Invoice; deletion is owned exclusively by account deletion (FEAT-24), subject to legal financial-record retention (scope-boundaries.md, SC-24)

## Governed Entity

**Entity:** Invoice
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| status | enum | This spec governs the Generated -> Sent -> Corrected transitions; Payment pending, Paid, Overdue, Refunded, Partially refunded, and Disputed are owned by FEAT-10, FEAT-11, and FEAT-25 |
| Any content field once Sent+ (amount, currency, tax line, business/billing details, issue_date, due_date, invoice_number) | -- | Frozen the moment `status` reaches `Sent` -- no further write path exists on any field once sent, regardless of role |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-002 | Invoice Detail | No edit control is ever rendered once `status` is Sent or later; only "Correct with a credit note" is shown |
| FEAT-09.SPEC-003 | Manual Invoice & Credit Note Issuance | The correction entry point -- always produces a new record, never an edit form against the original |
| FEAT-09.SPEC-005 | Manual Invoice & Credit Note Recording | Writes the credit note as a new record and sets the original's `status` to `Corrected` atomically |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| status (and every content field once Sent+) | No content field may be written once `status` is `Sent` or later, by any role, through any spec in this feature | Always, once `status` reaches Sent | On every attempted write | Not user-facing as a form error -- enforced structurally by the absence of any edit control (FEAT-09.SPEC-002) rather than a rejected save | Yes (structural) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Correction-is-a-new-record | status (original), the credit note's own fields | A correction is only ever expressed as a new Invoice record (a credit note) linked back to the original; the original's own content fields are never touched | Not applicable -- the product offers no path that would produce this error, since no edit control exists once Sent |
| Original-marked-Corrected-atomically | status (original), the credit note's creation | The original's `status` transitions to `Corrected` in the same atomic step that creates its credit note -- there is no window in which a credit note exists but the original still reads `Sent` | "This invoice was already corrected. View the existing credit note." (shown only on a second concurrent attempt, per FEAT-09.SPEC-005's re-check) |

## Authorization Rules

Not applicable to this spec -- who may issue a correction is governed by FEAT-09.SPEC-006. This spec governs the mechanics and boundary of immutability, not who may act on it.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| status | Set to `Generated` at creation, then `Sent` immediately once the send hand-off begins (FEAT-09.SPEC-004/FEAT-09.SPEC-005); set to `Corrected` only when a credit note is recorded against it | On create, then on the corresponding transition | No -- these three transitions are entirely system-driven; no screen offers a manual status-setting control for them |

## Business Rules

- XBR-04: evidence records -- including a sent invoice -- are never silently altered; changes happen only as new, logged events. This spec is the FEAT-09-specific instantiation of that rule for the Invoice entity.
- BRIEF.md's Constraints (record immutability): "payments and records must be correct," and a correction must be visible rather than a silent rewrite -- this spec's credit-note-as-new-record rule is the direct mechanism delivering that guarantee.
- A `Corrected` original and its credit note are permanently linked; the link is never removed, even if the credit note is itself later referenced by a refund (FEAT-25) recorded against the paid history of either record.
- Immutability applies uniformly regardless of how the invoice was created -- an automatically generated invoice (FEAT-09.SPEC-004) and a manually issued one (FEAT-09.SPEC-005) become immutable at exactly the same point: the moment `status` reaches `Sent`.

## Edge Cases

- **Nadia attempts to correct an invoice that is still in `Generated` status (has not yet actually sent, e.g., during a retried send hand-off)** -- Not applicable in practice: FEAT-09.SPEC-004 and FEAT-09.SPEC-005 both set `status: Sent` in the same automation run that creates the record, before any user could view or act on it in a `Generated`-only state; there is no user-reachable window in which an invoice is both created and still correctable as if unsent.
- **A credit note is itself later corrected** -- Permitted: a credit note is an Invoice record like any other once `status` reaches `Sent`, so it becomes immutable and correctable by the same rule -- a correction of a correction is simply another new, linked record, with the chain fully traceable through each record's link back to its predecessor.
- **Two sessions attempt to correct the same original invoice concurrently** -- Resolved by FEAT-09.SPEC-005's atomic write-time re-check (Cross-Field Rules, Original-marked-Corrected-atomically): the first to complete succeeds, and the second is refused with "This invoice was already corrected. View the existing credit note."
- **An invoice is Sent and later Paid, then Nadia issues a credit note against it** -- Allowed: the payment history (owned by FEAT-10) is untouched by the correction; the original shows both its Paid history and its Corrected status side by side, and any actual refund is a separate action Nadia takes through FEAT-25.
- **A downstream failure occurs after the original is marked Corrected but before the credit note's own send completes** -- The `Corrected` status on the original and the credit note's existence are not rolled back, since both are already evidentiary per XBR-04; only the credit note's own send is retried (FEAT-09.SPEC-010), consistent with FEAT-09.SPEC-005's failure handling.

## Acceptance Criteria

**FEAT-09.SPEC-008-AC-01:** Given an invoice's status is Sent, when Nadia views its detail, then no edit control is shown for any field.

**FEAT-09.SPEC-008-AC-02:** Given an invoice's status is Sent, when Nadia wants to correct it, then the only available action is "Correct with a credit note."

**FEAT-09.SPEC-008-AC-03:** Given Nadia issues a credit note against a Sent invoice, when it is recorded, then a new, separate Invoice record is created and the original's status becomes Corrected in the same step.

**FEAT-09.SPEC-008-AC-04:** Given an invoice has been marked Corrected, when Nadia or Owen views its original content, then every original field (amount, dates, business/billing details, invoice number) remains exactly as first sent.

**FEAT-09.SPEC-008-AC-05:** Given two sessions attempt to correct the same invoice at effectively the same time, when both reach the write, then exactly one succeeds and the other is told "This invoice was already corrected. View the existing credit note."

**FEAT-09.SPEC-008-AC-06:** Given a credit note itself reaches Sent status, when Nadia later wants to correct it, then the same immutability and correction rule applies -- a new record links back to it.

**FEAT-09.SPEC-008-AC-07:** Given an invoice was created automatically (FEAT-09.SPEC-004), when it reaches Sent status, then it becomes immutable on exactly the same terms as a manually issued invoice.

**FEAT-09.SPEC-008-AC-08:** Given a Paid invoice is later corrected by a credit note, when Nadia views its detail, then both its Paid payment history and its Corrected status are shown together, unaltered by each other.

**FEAT-09.SPEC-008-AC-09:** Given the original invoice is marked Corrected, when Nadia or Owen looks for its credit note, then a link to the linked credit note is present on the original's detail view.

**FEAT-09.SPEC-008-AC-10:** Given a correction chain exists (an invoice corrected by a credit note, which is itself later corrected), when any record in the chain is viewed, then its link back to its immediate predecessor is traceable.

**FEAT-09.SPEC-008-AC-11:** Given the credit note's own send fails after the original was already marked Corrected, when the failure occurs, then the Corrected status and the credit note record are unaffected, and only the send is retried.

**FEAT-09.SPEC-008-AC-12:** Given no invoice in this product is ever created without immediately reaching Sent status within the same automation run, when any invoice is inspected, then it is never observed sitting in a user-reachable Generated-only, still-editable state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 0 (N/A -- owned by FEAT-09.SPEC-006) | 0 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
