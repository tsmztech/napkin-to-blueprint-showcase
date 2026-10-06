---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-25.SPEC-006
spec_name: Refund, Cancellation & Reversal Authorization and Validation Rules
spec_slug: refund-cancellation-reversal-authorization-validation-rules
parent_feature: FEAT-25
parent_feature_name: Refund & Cancelled Project Handling
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 37
acceptance_criteria_count: 23
---

# Logic/Rule Spec: Refund, Cancellation & Reversal Authorization and Validation Rules

## Overview

**Name:** Refund, Cancellation & Reversal Authorization and Validation Rules
**ID:** FEAT-25.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs who may mark a refund or cancellation, the refund-amount and no-partial-payment limits, the refunded-cannot-be-repaid-without-correction rule, reject-with-refresh concurrency, which invoice and project states are eligible, and Dana's view-only status visibility.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling
**Governed Entity:** The refund, cancellation, and reversal fields this feature writes on Invoice, Payment, and Project -- not those entities' full field sets, which are governed by their owning features' own Logic/Rule specs (FEAT-09.SPEC-007/FEAT-09.SPEC-008 for Invoice, FEAT-10.SPEC-006 for Payment, FEAT-01.SPEC-011 for Project's derived stage)

## Scope and Non-Goals

**In Scope:**
- Field validation for the refund amount and the optional reason fields this feature introduces on Payment and Project
- The refund-amount ceiling and the no-partial-payment corollary (XBR-20)
- The refunded-cannot-be-repaid-without-correction rule (XBR-20)
- Authorization for marking a refund, marking a cancellation, and viewing the resulting statuses, across every role in the Access Matrix
- The reject-with-refresh concurrency behavior for this feature's writes to Invoice, Payment, and Project
- The processor-authoritative-over-manual rule governing a reversal versus a concurrent manual refund entry
- Which invoice status and Payment status combinations are refund-eligible, and which project stages are cancellable
- Dana's (Support Operator) view-only visibility of this feature's resulting statuses through the owning detail screens, never through this feature's own screens

**Non-Goals:**
- Full field validation for Invoice fields this feature does not write (invoice number, amount, currency, tax line, due date) -- owned entirely by FEAT-09.SPEC-007 (Invoice Content, Numbering, Amount & Due Date Rules); this spec governs only the `status` transitions this feature performs
- Full field validation for Payment fields this feature does not write (method, paid_at, recorded_by) -- owned entirely by FEAT-10.SPEC-006 (Payment Authorization & Validation Rules); this spec governs only the refund amount, the refunded status contribution, and the reversal's Reversed status
- The Project stage derivation formula itself -- owned entirely by FEAT-01.SPEC-011 (Project Stage Derivation); this spec governs only the authorization and concurrency behavior around setting `cancelled_at`, which that formula reads
- Correcting a refunded invoice back to Paid -- excluded per XBR-20: that correction is a new, logged event owned by FEAT-09's credit-note or new-invoice flow, never a status flip this spec or any part of this feature performs

## Governed Entity

**Entity:** The refund/cancellation/reversal fields on Invoice, Payment, and Project
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| Invoice.status (Refunded / Partially refunded / Disputed values only) | enum | The three status values this feature writes; all other Invoice.status values are owned by FEAT-09, FEAT-10, and FEAT-11 |
| Payment.status (Reversed value only) | enum | The one status value this feature writes; all other Payment.status values are owned by FEAT-10 |
| Payment.refunded_amount | number, feature-introduced | The amount recorded as refunded against a Payment; not yet reflected in the dependency map's Payment field list, since it is introduced by this feature's Key Capabilities (Record a partial refund) -- tracked here as its authoritative definition pending the map's next synthesis pass |
| Payment.refund_reason | text, feature-introduced, optional | The freelancer-entered note accompanying a refund; introduced by this feature alongside `refunded_amount` |
| Project.cancelled_at | timestamp | Set exclusively by this feature; read by FEAT-01.SPEC-011 to derive the Cancelled stage |
| Project.cancellation_reason | text, feature-introduced, optional | The freelancer-entered note accompanying a cancellation; introduced by this feature |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-25.SPEC-001 | Mark Invoice Refunded Screen | On field blur and form submit (refund amount); authorization on screen entry |
| FEAT-25.SPEC-002 | Mark Project Cancelled Screen | Authorization on screen entry; concurrency check on submit |
| FEAT-25.SPEC-003 | Refund & Partial Refund Recording | Commit-time re-validation of the refund amount and the invoice's status; the refunded-cannot-be-repaid rule's write-side enforcement |
| FEAT-25.SPEC-004 | Project Cancellation Recording | Commit-time re-validation of the project's stage; reject-with-refresh enforcement |
| FEAT-25.SPEC-005 | Payment Reversal (Chargeback) Recording | Processor-authoritative-over-manual enforcement when a reversal and a manual refund race |
| FEAT-09.SPEC-002 | Invoice Detail | Authorization on the "Record a refund" entry point's visibility, per role |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Payment.refunded_amount | Required, greater than zero | Always, when a refund is submitted | On blur and on submit (FEAT-25.SPEC-001); re-checked at commit (FEAT-25.SPEC-003) | "Enter an amount greater than zero." | Yes |
| Payment.refunded_amount | Must not exceed the invoice's amount paid | Always | On blur and on submit (FEAT-25.SPEC-001); re-checked at commit (FEAT-25.SPEC-003) | "This amount is more than what was paid. The amount paid was {amount paid}." | Yes |
| Payment.refunded_amount | Numeric, in the invoice's currency, no more than two decimal places | Always | On blur | "Enter a valid amount in {currency}." | Yes |
| Payment.refund_reason | No validation beyond data type -- free text, optional | Always | -- | -- | No |
| Project.cancellation_reason | No validation beyond data type -- free text, optional | Always | -- | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Refund-amount ceiling (XBR-20) | Payment.refunded_amount, Payment.amount | The refunded amount can never exceed the Payment's full amount, since an invoice is paid in full or not at all (SC-17: no partial payments exist for the refund ceiling to divide against) | "This amount is more than what was paid. The amount paid was {amount paid}." |
| No-partial-payment corollary (SC-17) | Payment.amount, Payment.refunded_amount | Because instalments do not exist on a single invoice, the refund ceiling is always the one full Payment amount, never a sum across multiple payments -- this simplifies the ceiling to a single comparison rather than an aggregation | N/A -- this is a structural consequence of SC-17, not itself a user-facing check |
| Refunded-cannot-be-repaid-without-correction (XBR-20) | Invoice.status | An Invoice already Refunded or Partially refunded can never be set back to Paid by any manual action inside this feature or FEAT-10; the only path back to a paid-looking state is a new, logged correction (a credit note or new invoice) owned by FEAT-09, never a status flip | This is enforced by omission -- no control anywhere in the product offers "mark Paid" on a Refunded or Partially refunded invoice; there is no error message because no such attempt is reachable |
| Refund eligibility (invoice + Payment status combinations) | Invoice.status, Payment.status | A refund may be recorded only when exactly one of these combinations holds: (1) Invoice.status Paid with Payment.status Succeeded (paid through the platform); (2) Invoice.status Paid (recorded by freelancer) with Payment.status Recorded manually (paid off-platform, recorded via FEAT-10.SPEC-005). Every other combination is ineligible: Invoice.status Generated, Sent, Payment pending, Overdue, Refunded, Partially refunded, Disputed, or Corrected, or a Paid/Paid (recorded by freelancer) invoice whose Payment.status is not the one paired above (including Reversed, Failed, Initiated, Pending). Both eligible combinations share the same amount ceiling and outcomes. Off-platform payments are deliberately included because the feature records a refund issued outside the platform; they are excluded from processor reversals (FEAT-25.SPEC-005 applies only to Payment.status Succeeded) | Ineligible at entry: the "Record a refund" control is not offered on FEAT-09.SPEC-002. Ineligible at commit: "This invoice's status changed since you opened this page. Refresh to see the latest state." |
| Project cancellability | Project derived stage, Project.completed_at | A project may be cancelled only while its derived stage is none of Cancelled, Archived, or Complete (Complete meaning Project.completed_at is set). Complete and Cancelled are mutually exclusive terminal states (FEAT-01.SPEC-011 precedence order). The stale-state message applies whenever the stage at commit is any of the three (it changed since the screen loaded); any other stage results in plain success | Ineligible at entry: the "Cancel Project" entry is not offered on FEAT-01.SPEC-005. Ineligible at commit: "This project's state changed since you opened this page. Refresh to see the latest state." |
| Processor-authoritative-over-manual (dependency map, Invoice Contention) | Invoice.status, Payment.status | When a processor-reported reversal and a concurrently submitted manual refund race for the same invoice, the reversal wins regardless of arrival order relative to the refund submission's own commit attempt | "This invoice's status changed since you opened this page. Refresh to see the latest state." (shown to Nadia on the losing manual refund submission) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|----------------------------------------------|
| Mark an invoice Refunded or Partially refunded | Nadia (Freelancer) | Only on her own invoice, only while it is refund-eligible: status Paid with a Succeeded Payment, or status Paid (recorded by freelancer) with a Recorded manually Payment (Refund eligibility rule) | On any other invoice status the "Record a refund" control is not offered; a submission made after the invoice stopped being eligible is refused with "This invoice's status changed since you opened this page. Refresh to see the latest state." |
| Mark an invoice Refunded or Partially refunded | Owen (Client Primary Contact) | Never | The action is not shown anywhere in his portal navigation |
| Mark an invoice Refunded or Partially refunded | Priya (Client Reviewer Contact) | Never | The action is not shown; Priya has no billing visibility at all (Access Matrix: Invoicing & Payments is None) |
| Mark an invoice Refunded or Partially refunded | Dana (Support Operator) | Never | The "Record a refund" control is not rendered in Dana's support session (FEAT-09.SPEC-002) and she never reaches FEAT-25.SPEC-001; a direct link to that screen opens the invoice detail (FEAT-09.SPEC-002) in her read-only session with no error message |
| Mark a project Cancelled | Nadia (Freelancer) | Only on her own project, only while its derived stage is none of Cancelled, Archived, or Complete (Project cancellability rule) | On a project in one of those three stages the "Cancel Project" entry is not offered; a confirmation made after the project reached one of them is refused with "This project's state changed since you opened this page. Refresh to see the latest state." |
| Mark a project Cancelled | Owen (Client Primary Contact) | Never | The action is not shown anywhere in his portal navigation |
| Mark a project Cancelled | Priya (Client Reviewer Contact) | Never | The action is not shown; the Client & Project Management capability group is None for Reviewer contacts |
| Mark a project Cancelled | Dana (Support Operator) | Never | The "Cancel Project" entry is not rendered in Dana's support session (FEAT-01.SPEC-005) and she never reaches FEAT-25.SPEC-002; a direct link to that screen opens the project detail (FEAT-01.SPEC-005) in her read-only session with no error message |
| View the resulting Refunded/Partially refunded/Disputed status on an invoice | Nadia (Freelancer) | Always, on her own invoices | -- |
| View the resulting Refunded/Partially refunded/Disputed status on an invoice | Owen (Client Primary Contact) | Own-only, on his own company's invoices | -- |
| View the resulting Refunded/Partially refunded/Disputed status on an invoice | Priya (Client Reviewer Contact) | Never | Invoice content is hidden entirely from Reviewer contacts (FEAT-09.SPEC-006); a direct link redirects to her portal home with no error message |
| View the resulting Refunded/Partially refunded/Disputed status on an invoice | Dana (Support Operator) | Always, view only, inside a logged support session (FEAT-31), on the invoice detail screen (FEAT-09.SPEC-002) -- the status only, never an action to change it | -- |
| View the resulting Cancelled status on a project | Nadia (Freelancer) | Always, on her own projects | -- |
| View the resulting Cancelled status on a project | Owen (Client Primary Contact) | Own-only, on his own project, through the client portal | -- |
| View the resulting Cancelled status on a project | Priya (Client Reviewer Contact) | Own-only, on her own project (the same portal view Owen sees, per the Access Matrix's Client Portal Access row) | -- |
| View the resulting Cancelled status on a project | Dana (Support Operator) | Always, view only, inside a logged support session (FEAT-31), on the project detail screen (FEAT-01.SPEC-005) -- the status only, never an action to change it | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Invoice.status | Set to Refunded when Payment.refunded_amount equals Payment.amount; set to Partially refunded when less; set to Disputed on a processor-reported reversal, regardless of the prior Paid-family value | On commit (FEAT-25.SPEC-003, FEAT-25.SPEC-005) | No -- the resulting value is always derived from the amount or the event type, never chosen directly by Nadia |
| Payment.status | Set to Reversed on a processor-reported reversal | On commit (FEAT-25.SPEC-005) | No -- this value is set only by the inbound processor event, never by any manual action |
| Project.cancelled_at | Current timestamp | On commit (FEAT-25.SPEC-004) | No -- always the commit-time timestamp |

## Business Rules

- XBR-04: Every record this feature touches is evidentiary and immutable once written -- a refund, cancellation, or reversal is a new, logged transition displayed alongside the prior record, never a replacement of it.
- XBR-05: Every refund, partial refund, cancellation, and reversal writes an append-only Activity Log Entry (owned by FEAT-13, triggered by FEAT-25.SPEC-003/004/005).
- XBR-20: A refund cannot exceed the amount paid; a refunded invoice cannot later be marked Paid again without a logged correction.
- XBR-21: A processor-reported reversal sets the invoice Disputed alongside its preserved Paid record and notifies Nadia; refunds and dispute responses happen in her own processor account, never inside Clientroom.
- XBR-25: Cancellation preserves a project's full history; an unaccepted proposal stays open until Nadia revises and re-sends it or cancels the project.
- Reject-with-refresh concurrency (dependency map, Invoice and Project Contention notes): any write this feature attempts against a shared Invoice or Project entity that has changed since it was loaded is refused, with the refreshed current state shown -- never a silent overwrite and never a merge.
- XBR-29: Dana's support sessions are read-only in every feature, including this one. Per the Brief and FEAT-09.SPEC-002, Dana sees this feature's resulting statuses (Refunded, Partially refunded, Disputed, Cancelled) only on the owning detail screens (FEAT-09.SPEC-002, FEAT-01.SPEC-005), where no refund or cancellation control is rendered in her session; she never reaches FEAT-25.SPEC-001 or FEAT-25.SPEC-002 and cannot change anything. XBR-29 itself only establishes that sessions are read-only; the never-reaches-these-screens behavior comes from the Brief's Cross-Feature Touchpoints and Side-Effect Inventory.

## Edge Cases

- **A refund amount is entered at exactly the amount paid** -- Passes validation (the boundary is inclusive); FEAT-25.SPEC-003 records it as Refunded, not Partially refunded.
- **A refund amount is entered at one currency unit over the amount paid** -- Fails validation with "This amount is more than what was paid. The amount paid was {amount paid}."; the boundary is exclusive on the high side.
- **Nadia attempts to mark a Refunded invoice Paid again through any control anywhere in the product** -- No such control exists; the refunded-cannot-be-repaid rule is enforced by omission rather than a blocking error, since FEAT-09 and FEAT-10 never render a "mark Paid" action on an invoice already in a Refunded-family status.
- **A reversal and a manual refund submission race for the same invoice** -- Whichever commits first wins; if the reversal commits first, the refund submission's own commit-time check (FEAT-25.SPEC-003) finds the invoice already Disputed and is refused with the refresh message. If the refund commits first, the reversal (FEAT-25.SPEC-005) still applies on top of it, since a processor-confirmed reversal is authoritative over the prior manual entry.
- **Dana opens a read-only support session on an account with a Disputed invoice or a Cancelled project** -- She sees the resulting status on the owning detail screen exactly as Nadia would, but no refund or cancellation control is rendered and she never sees an action to change it, per the Support Operator's unconditional view-only entitlement.
- **A project's stage changes (Complete, Archived, or a second Cancelled attempt) between when FEAT-25.SPEC-002 loads and when Nadia confirms cancellation** -- The commit-time check in FEAT-25.SPEC-004 rejects the stale attempt with the refresh message; the project's actual current state is never silently overwritten.
- **Nadia looks for "Cancel Project" on a project she has marked Complete** -- The entry is not offered; a Complete project is never cancelled afterward (Project cancellability rule). No error appears because the entry itself is absent.
- **A refund is attempted on an invoice paid off-platform (Paid (recorded by freelancer), Recorded manually)** -- Eligible; it proceeds exactly like a platform-paid invoice. A processor reversal notice for that same invoice is discarded by FEAT-25.SPEC-005, since no processor charge exists.

## Acceptance Criteria

**FEAT-25.SPEC-006-AC-01:** Given Nadia enters a refund amount of exactly the amount paid, when she submits, then validation passes and FEAT-25.SPEC-003 records the invoice as Refunded.

**FEAT-25.SPEC-006-AC-02:** Given Nadia enters a refund amount one unit over the amount paid, when she blurs the field, then she sees "This amount is more than what was paid. The amount paid was {amount paid}." and the field remains in an error state.

**FEAT-25.SPEC-006-AC-03:** Given Nadia enters a refund amount of zero, when she blurs the field, then she sees "Enter an amount greater than zero."

**FEAT-25.SPEC-006-AC-04:** Given Nadia enters a refund amount with more than two decimal places, when she blurs the field, then she sees "Enter a valid amount in {currency}."

**FEAT-25.SPEC-006-AC-05:** Given Nadia (Freelancer) opens her own invoice with status Paid (Payment Succeeded), or with status Paid (recorded by freelancer) (Payment Recorded manually), when she looks for "Record a refund", then it is shown and reachable in both cases.

**FEAT-25.SPEC-006-AC-06:** Given Owen (Client Primary Contact) looks anywhere in his portal navigation, when he searches for a refund or cancellation action, then none exists.

**FEAT-25.SPEC-006-AC-07:** Given Priya (Client Reviewer Contact) attempts to reach an invoice directly, then invoice content is hidden entirely and she is redirected to her portal home with no error message.

**FEAT-25.SPEC-006-AC-08:** Given Dana (Support Operator) opens a logged support session on an account with a Paid invoice, when she views the invoice detail screen (FEAT-09.SPEC-002), then no "Record a refund" control is rendered, she has no path to FEAT-25.SPEC-001, and a direct link to it opens the invoice detail read-only with no error message.

**FEAT-25.SPEC-006-AC-09:** Given Nadia (Freelancer) opens a project whose derived stage is none of Cancelled, Archived, or Complete, when she looks for "Cancel Project", then it is shown and reachable.

**FEAT-25.SPEC-006-AC-10:** Given Dana (Support Operator) opens a logged support session on an account with an active project, when she views the project detail screen (FEAT-01.SPEC-005), then no "Cancel Project" entry is rendered, she has no path to FEAT-25.SPEC-002, and she sees the project's status read-only.

**FEAT-25.SPEC-006-AC-11:** Given Owen (Client Primary Contact) views his own company's invoice, when it is Refunded or Partially refunded, then he sees the resulting status alongside the preserved Paid record.

**FEAT-25.SPEC-006-AC-12:** Given Priya (Client Reviewer Contact) views her own project's portal page, when the project is Cancelled, then she sees the resulting Cancelled status (per the Access Matrix's Own-only portal view), with no billing content mixed into that view.

**FEAT-25.SPEC-006-AC-13:** Given no control anywhere in the product offers "mark Paid" on a Refunded invoice, when Nadia looks for one, then none exists, and any correction must go through a new credit note or invoice via FEAT-09.

**FEAT-25.SPEC-006-AC-14:** Given a reversal commits first for an invoice, when Nadia's concurrent manual refund submission for that same invoice reaches commit, then it is refused with "This invoice's status changed since you opened this page. Refresh to see the latest state."

**FEAT-25.SPEC-006-AC-15:** Given Nadia's manual refund commits first for an invoice, when a reversal notice for that same invoice is then processed, then the reversal still applies and sets the invoice Disputed, since a processor-confirmed reversal is authoritative over the prior manual entry.

**FEAT-25.SPEC-006-AC-16:** Given a project's stage changed (to Complete, Archived, or already Cancelled) between when Nadia loaded the Mark Project Cancelled screen and when she confirms, when she confirms, then the submission is rejected with "This project's state changed since you opened this page. Refresh to see the latest state." and no overwrite occurs; and given the project's stage is any other stage at confirmation, then the cancellation succeeds with the plain "Project cancelled" toast and no stale-state message.

**FEAT-25.SPEC-006-AC-17:** Given an invoice's status changed since Nadia loaded the Mark Invoice Refunded screen, when she submits her refund, then the submission is rejected with the refresh message and no overwrite occurs.

**FEAT-25.SPEC-006-AC-18:** Given an invoice has already reached its one full Payment (SC-17: no instalments exist), when Nadia enters a refund amount, then the ceiling checked is that single Payment's full amount, never a sum across multiple payments.

**FEAT-25.SPEC-006-AC-19:** Given Nadia leaves the refund reason or cancellation reason field empty, when she submits, then no validation error appears, since both reason fields are optional.

**FEAT-25.SPEC-006-AC-20:** Given a reversal is recorded on a previously Refunded or Partially refunded invoice, when the reversal is processed, then it still applies and sets the invoice Disputed, since the Paid-family check covers all three statuses that can precede a reversal.

**FEAT-25.SPEC-006-AC-21:** Given an invoice with status Paid whose Payment is in Succeeded status, or status Paid (recorded by freelancer) whose Payment is in Recorded manually status, when Nadia submits a valid refund amount, then the eligibility check passes and FEAT-25.SPEC-003 records the refund.

**FEAT-25.SPEC-006-AC-22:** Given an invoice whose status is Generated, Sent, Payment pending, Overdue, Refunded, Partially refunded, Disputed, or Corrected, or a Paid invoice whose Payment is Reversed, when Nadia looks for "Record a refund" or a stale submission reaches commit, then the control is not offered, or the submission is refused with "This invoice's status changed since you opened this page. Refresh to see the latest state."

**FEAT-25.SPEC-006-AC-23:** Given a project Nadia has marked Complete, when she opens its project detail screen, then no "Cancel Project" entry is offered, and a stale cancellation confirmation submitted for it is refused with the refresh message.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 6 | 6 |
| Authorization Rules | 16 | 16 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 7 | 7 |
| Edge Cases | 8 | 8 |
