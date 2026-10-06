---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-24.SPEC-006
spec_name: Pre-Deletion Warning & Retention Determination Rules
spec_slug: pre-deletion-warning-retention-determination-rules
parent_feature: FEAT-24
parent_feature_name: Data Export & Account Deletion
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 11
acceptance_criteria_count: 18
---

# Logic/Rule Spec: Pre-Deletion Warning & Retention Determination Rules

## Overview

**Name:** Pre-Deletion Warning & Retention Determination Rules
**ID:** FEAT-24.SPEC-006
**Type:** Logic/Rule
**Purpose:** Determines which open items (unpaid invoices, pending approvals) trigger a specific warning without blocking deletion, and which records are legally retained versus immediately deleted.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion
**Governed Entity:** Deletion Readiness Determination (a derived evaluation, not a stored entity in its own right, computed from Invoice, Milestone, and the entity-class inventory in feature-overview.md's Entity-Lifecycle Coverage Matrix)

## Scope and Non-Goals

**In Scope:**
- Determining whether an open unpaid invoice exists for the account, and defining the exact banner text (with placeholders) for that warning
- Determining whether a pending-approval milestone exists for the account, and defining the exact banner text (with placeholders) for that warning
- Defining the exact retention-notice text shown alongside the warnings
- Determining, per entity class, whether a record is legally retained or immediately deleted at cascade time

**Non-Goals:**
- Displaying the warnings or capturing confirmation -- owned by FEAT-24.SPEC-002 (Account Deletion Screen); this spec only defines what is determined, not how it is shown.
- Executing the actual retention hold-back or deletion -- owned by FEAT-24.SPEC-004 (Account Deletion Processing), which executes this spec's determination.
- Purging a retained record once its retention period lapses -- owned by FEAT-24.SPEC-005 (Legal Retention Purge); this spec only classifies a record as retained, it does not schedule or perform the later purge.
- Who may reach the screens or automations that invoke this determination -- owned entirely by FEAT-24.SPEC-007 (Export & Deletion Access Rules); this spec's own Authorization Rules section defers to it rather than duplicating that gate.

## Governed Entity

**Entity:** Deletion Readiness Determination (derived; composite of Invoice, Milestone, and entity-class retention classification)
**Source:** Feature Dependency Map (Invoice, Milestone entities); feature-overview.md, Entity-Lifecycle Coverage Matrix (entity-class inventory)

| Field | Data Type | Description |
|-------|-----------|-------------|
| open_unpaid_invoice_present | derived (boolean) | Whether the account has at least one Invoice in an open/unresolved status at evaluation time |
| pending_approval_present | derived (boolean) | Whether the account has at least one Milestone awaiting approval at evaluation time |
| open_invoice_count | derived (integer >= 0) | Number of Invoices counted as open under the open_unpaid_invoice_present derivation |
| open_invoice_totals | derived (list of currency + amount) | Sum of each open Invoice's outstanding amount, one entry per invoice currency; empty when open_invoice_count is 0 |
| pending_approval_count | derived (integer >= 0) | Number of Milestones counted as awaiting approval under the pending_approval_present derivation |
| record_retention_classification | derived (enum: Retained \| Immediately Deleted) | Per entity class, whether records of that class are held back under legal retention or hard-deleted immediately during the FEAT-24.SPEC-004 cascade |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-24.SPEC-002 | Account Deletion Screen | On screen load (open-item warning display) and again at the moment Delete My Account is tapped (re-check before proceeding) |
| FEAT-24.SPEC-004 | Account Deletion Processing | During cascade execution, when the process reaches each Invoice and Payment record, to decide retain versus delete |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| open_unpaid_invoice_present | No user input -- system-derived from Invoice.status | Always | On evaluation (screen load and at confirmation) | -- | No |
| pending_approval_present | No user input -- system-derived from Milestone.status | Always | On evaluation (screen load and at confirmation) | -- | No |
| open_invoice_count | No user input -- system-derived count of open Invoices | Always | On evaluation (screen load and at confirmation) | -- | No |
| open_invoice_totals | No user input -- system-derived sum of outstanding amounts per currency | Always | On evaluation (screen load and at confirmation) | -- | No |
| pending_approval_count | No user input -- system-derived count of Milestones in Deliverable Uploaded status | Always | On evaluation (screen load and at confirmation) | -- | No |
| record_retention_classification | No user input -- system-derived, fixed per entity class | Always | On evaluation (at cascade execution) | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Combined warning display | open_unpaid_invoice_present, pending_approval_present | When both are true, FEAT-24.SPEC-002 shows both warning banners together rather than merging them into one combined message, since each names a distinct, independent consequence | N/A -- not an error, an informational display rule |
| Exact warning and notice text | open_invoice_count, open_invoice_totals, pending_approval_count | The banner and notice text is exactly the literal text in Business Rules ("Exact warning and notice text"); FEAT-24.SPEC-002 renders it verbatim with placeholders filled from these fields and never paraphrases, merges, or abbreviates it | N/A -- not an error, a text-fidelity rule |
| Retention independent of open-item state | record_retention_classification, open_unpaid_invoice_present | An Invoice's retention classification (Retained if it is an Invoice or Payment record) applies regardless of whether that same invoice is also counted as "open" for warning purposes -- the two determinations are computed independently and never override each other | N/A -- not an error, a clarifying independence rule |

## Authorization Rules

This spec's determinations run only as part of FEAT-24.SPEC-002 and FEAT-24.SPEC-004, both of which FEAT-24.SPEC-007 (Export & Deletion Access Rules) already confines to Nadia and her own account. This section states that dependency rather than duplicating FEAT-24.SPEC-007's own role-action matrix.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Trigger this determination (open-item evaluation, retention classification) | Nadia (Freelancer) | Only via FEAT-24.SPEC-002 (screen load / confirmation) or FEAT-24.SPEC-004 (cascade execution), both already gated to her own account by FEAT-24.SPEC-007 | -- |
| Trigger this determination | Owen (Client Primary Contact), Priya (Client Reviewer Contact), Dana (Support Operator) | Never | This determination never runs on behalf of these roles, since neither FEAT-24.SPEC-002 nor FEAT-24.SPEC-004 is ever reachable by them (FEAT-24.SPEC-007) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| open_unpaid_invoice_present | True if any Invoice for the account has status Generated, Sent, Payment pending, Overdue, Partially refunded, or Disputed (i.e., not Paid, Paid (recorded by freelancer), Refunded, or Corrected) | On evaluation (screen load and at confirmation) | No -- always derived from current Invoice status |
| pending_approval_present | True if any Milestone for the account has status Deliverable Uploaded (awaiting Owen's approval) | On evaluation (screen load and at confirmation) | No -- always derived from current Milestone status |
| open_invoice_count | Count of Invoices meeting the open_unpaid_invoice_present status set | On evaluation (screen load and at confirmation) | No -- always derived |
| open_invoice_totals | For each currency, sum of the outstanding (unpaid) amount of the counted Invoices; Partially refunded and Disputed Invoices contribute their full invoiced amount less any amount already paid | On evaluation (screen load and at confirmation) | No -- always derived |
| pending_approval_count | Count of Milestones with status Deliverable Uploaded | On evaluation (screen load and at confirmation) | No -- always derived |
| record_retention_classification | Invoice and Payment classify as Retained, regardless of their own status field; every other entity class in feature-overview.md's Entity-Lifecycle Coverage Matrix classifies as Immediately Deleted | On evaluation (at cascade execution) | No -- fixed per entity class, never per individual record |

## Business Rules

- Open items warn without blocking deletion (product-features.md, Primary Flows & Alternates: "a freelancer with active unpaid invoices or pending approvals is warned about the consequences before deletion is finalized, not silently blocked").
- SC-24: financial records (Invoice, Payment) are retained under a legal requirement rather than deleted immediately; every other entity class has no such requirement and is deleted in full at cascade time.
- XBR-33: retention applies specifically to financial records, never to client contacts' personal data or any other entity class -- a Client Contact's name and email are always immediately deleted, with no retention exception, even though the account may separately have retained financial records.
- **Exact warning and notice text (FEAT-24.SPEC-002 renders these verbatim).** Placeholders in braces are filled from this spec's derived fields. Text never names individual invoices, clients, or milestones -- counts and amounts only.
  - **Unpaid-invoice banner** (shown only when open_unpaid_invoice_present is true). Title: "Unpaid invoices". Body when open_invoice_count is 1: "You have 1 unpaid invoice ({open_invoice_totals}). Deleting your account will not collect it, and your client will no longer be able to pay it." Body when open_invoice_count is 2 or more: "You have {open_invoice_count} unpaid invoices ({open_invoice_totals}). Deleting your account will not collect them, and your clients will no longer be able to pay them." When invoices span more than one currency, {open_invoice_totals} lists each currency total separated by " + " (e.g., "$1,200.00 + EUR 300.00"); with one currency it is that single total.
  - **Pending-approval banner** (shown only when pending_approval_present is true). Title: "Deliverables awaiting approval". Body when pending_approval_count is 1: "1 deliverable is waiting for your client's approval. Deleting your account will discard it without a decision, and your client will lose access to it." Body when pending_approval_count is 2 or more: "{pending_approval_count} deliverables are waiting for your clients' approval. Deleting your account will discard them without a decision, and your clients will lose access to them."
  - **Retention notice** (always shown, independent of open items): "Invoices and payment records are kept for {retention_period} after deletion, as legally required, and can no longer be accessed by you or your clients. Everything else, including your clients' names and email addresses, is deleted in full." {retention_period} is the value of platform parameter: `financial-record-legal-retention-period`, rendered in words (e.g., "seven years").
  - Both banners carry informational styling only: neither contains a button, a checkbox, or any text that says deletion is blocked.
- FEAT-13.SPEC-006 applies the same financial-record retention exception to Activity Log Entries that are themselves, or directly support, a financial record (invoice sent, credit note issued, manual payment recorded, refund/reversal/cancellation recorded) -- this spec's retention classification is the shared basis both FEAT-24.SPEC-004 and FEAT-13.SPEC-006 apply to their respective entities.

## Edge Cases

- **An invoice's status changes between screen load and confirmation (e.g., becomes Paid in another tab)** -- FEAT-24.SPEC-002 re-runs this determination at the moment of confirmation (Enforced By table), so the warning and retention picture reflect the state at that moment, not at screen load.
- **The account has zero invoices and zero milestones ever created** -- Both open_unpaid_invoice_present and pending_approval_present evaluate false; no warning is shown, and since no Invoice or Payment records exist, the cascade proceeds with no retained records at all.
- **An Invoice is in a Partially refunded or Disputed status at deletion time** -- Counted as open for warning purposes (money movement is unresolved), and separately classified Retained for the cascade, since both determinations key off it being an Invoice record, independent of its specific status.
- **A Payment record that failed or was never confirmed exists for the account** -- Still classified Retained, since retention is determined by entity class (Payment), not by the record's own status.
- **A Milestone is Reopened rather than Deliverable Uploaded at evaluation time** -- pending_approval_present evaluates false for that milestone, since Reopened is not the awaiting-approval status; if a Deliverable is later re-uploaded on it, the next evaluation reflects the new Deliverable Uploaded status.
- **Both an open invoice and a pending approval exist simultaneously** -- Both warning banners are shown together on FEAT-24.SPEC-002 (Cross-Field Rules), neither suppressing the other; the unpaid-invoice banner is listed first.
- **Invoices in different currencies are open at once** -- {open_invoice_totals} lists one total per currency, joined with " + ", and the banner uses the plural body whenever open_invoice_count is 2 or more, regardless of currency.

## Acceptance Criteria

**FEAT-24.SPEC-006-AC-01:** Given Nadia has an Invoice in Sent status, when this determination runs, then open_unpaid_invoice_present evaluates true and FEAT-24.SPEC-002 shows the unpaid-invoice warning.

**FEAT-24.SPEC-006-AC-02:** Given Nadia has every Invoice in Paid status, when this determination runs, then open_unpaid_invoice_present evaluates false and no unpaid-invoice warning is shown.

**FEAT-24.SPEC-006-AC-03:** Given Nadia has a Milestone in Deliverable Uploaded status, when this determination runs, then pending_approval_present evaluates true and FEAT-24.SPEC-002 shows the pending-approval warning.

**FEAT-24.SPEC-006-AC-04:** Given Nadia has no Milestone in Deliverable Uploaded status, when this determination runs, then pending_approval_present evaluates false and no pending-approval warning is shown.

**FEAT-24.SPEC-006-AC-05:** Given Nadia has both an open invoice and a pending approval, when this determination runs, then both warnings are shown together on FEAT-24.SPEC-002.

**FEAT-24.SPEC-006-AC-06:** Given the cascade in FEAT-24.SPEC-004 reaches an Invoice record, when this determination runs, then it is classified Retained regardless of the invoice's own status field.

**FEAT-24.SPEC-006-AC-07:** Given the cascade reaches a Payment record, when this determination runs, then it is classified Retained regardless of the payment's own status field.

**FEAT-24.SPEC-006-AC-08:** Given the cascade reaches a Client Contact record, when this determination runs, then it is classified Immediately Deleted, with no retention exception for the contact's personal data.

**FEAT-24.SPEC-006-AC-09:** Given an invoice's status changes from Sent to Paid in another tab after Nadia loaded FEAT-24.SPEC-002, when she then confirms deletion, then this determination re-runs and the warning reflects the current Paid status.

**FEAT-24.SPEC-006-AC-10:** Given Nadia's account has zero invoices and zero milestones, when this determination runs, then no warnings are shown and no records are classified Retained.

**FEAT-24.SPEC-006-AC-11:** Given an Invoice is in Partially refunded status, when this determination runs, then it counts as open for warning purposes and is separately classified Retained for the cascade.

**FEAT-24.SPEC-006-AC-12:** Given a Milestone is in Reopened status, when this determination runs, then pending_approval_present does not count it as awaiting approval.

**FEAT-24.SPEC-006-AC-13:** Given Nadia (Freelancer) triggers this determination through FEAT-24.SPEC-002 or FEAT-24.SPEC-004, when it runs, then it evaluates only her own account's records.

**FEAT-24.SPEC-006-AC-14:** Given Owen, Priya, or Dana attempts to reach any spec that would trigger this determination, when they try, then it never runs on their behalf, since FEAT-24.SPEC-007 excludes them from every screen and automation that invokes it.

**FEAT-24.SPEC-006-AC-15:** Given Nadia has exactly 1 open Invoice with an outstanding amount of $1,200.00, when this determination runs, then the unpaid-invoice banner reads title "Unpaid invoices" and body "You have 1 unpaid invoice ($1,200.00). Deleting your account will not collect it, and your client will no longer be able to pay it."

**FEAT-24.SPEC-006-AC-16:** Given Nadia has 3 open Invoices totaling $1,000.00 and EUR 300.00, when this determination runs, then the banner body reads "You have 3 unpaid invoices ($1,000.00 + EUR 300.00). Deleting your account will not collect them, and your clients will no longer be able to pay them."

**FEAT-24.SPEC-006-AC-17:** Given Nadia has 2 Milestones in Deliverable Uploaded status, when this determination runs, then the pending-approval banner reads title "Deliverables awaiting approval" and body "2 deliverables are waiting for your clients' approval. Deleting your account will discard them without a decision, and your clients will lose access to them."

**FEAT-24.SPEC-006-AC-18:** Given any open-item state (including none), when FEAT-24.SPEC-002 loads, then the retention notice reads "Invoices and payment records are kept for {retention_period} after deletion, as legally required, and can no longer be accessed by you or your clients. Everything else, including your clients' names and email addresses, is deleted in full." with {retention_period} filled from platform parameter: `financial-record-legal-retention-period`.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 2 | 2 |
| Defaults/Derivations | 6 | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |
