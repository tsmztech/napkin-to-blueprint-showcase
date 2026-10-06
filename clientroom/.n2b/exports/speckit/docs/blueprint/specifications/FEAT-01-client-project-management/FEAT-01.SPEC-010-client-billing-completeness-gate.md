---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-01.SPEC-010
spec_name: Client Billing Completeness Gate
spec_slug: client-billing-completeness-gate
parent_feature: FEAT-01
parent_feature_name: Client & Project Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 7
acceptance_criteria_count: 9
---

# Logic/Rule Spec: Client Billing Completeness Gate

## Overview

**Name:** Client Billing Completeness Gate
**ID:** FEAT-01.SPEC-010
**Type:** Logic/Rule
**Purpose:** Requires billing name and billing address (tax ID always optional) to be captured before any invoice for the client can be sent -- evaluated fresh at every send attempt, most visibly encountered on the client's first invoice.
**Parent Feature:** FEAT-01 -- Client & Project Management
**Governed Entity:** Client (specifically the billing_name, billing_address, and tax_id fields)

## Scope and Non-Goals

**In Scope:**
- The completeness rule for a client's billing_name and billing_address (required) and tax_id (always optional)
- When completeness is evaluated: at billing-field edit time (informational) and at every invoice-send attempt (blocking, cross-feature) -- not a one-time-only check
- The exact completeness indicator shown on Client Detail and the exact block shown when Invoicing attempts to send a first invoice with billing details missing

**Non-Goals:**
- The invoice-sending flow itself, or any other invoice content requirement (invoice numbering, due dates, business details) -- owned entirely by Invoice Generation & Sending (FEAT-09), which enforces this gate at the moment of sending but does not define it
- Currency and tax rate configuration -- owned by Currency & Tax Handling (FEAT-15), a separate completeness requirement evaluated independently by FEAT-09 at send time
- Validating the format of billing_name or billing_address beyond non-empty presence -- the product defines no format constraint for these free-text fields, since business names and addresses vary too widely worldwide to validate against a fixed pattern (SC-20 covers locale format adaptation, not free-text business fields)

## Governed Entity

**Entity:** Client
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| client_name | text | Company name |
| billing_name | text | Name printed on invoices |
| billing_address | text | Address printed on invoices |
| tax_id | text | Client's tax identifier (optional) |
| status | enum (Active, Archived) | Not evaluated by this spec |
| currency and tax treatment | derived / configured (via FEAT-15) | Evaluated by FEAT-15's own completeness rule, not this spec |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-004 | Client Detail | Informational completeness indicator, re-evaluated on every billing-field save |
| FEAT-01.SPEC-001 | Add Client | Referenced only -- billing fields are optional at creation time (see Non-Goals of that spec) |
| FEAT-09 | Invoice Generation & Sending | Blocking gate re-evaluated on every invoice send attempt for the client, not only the first |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| billing_name | Required for billing completeness | Before any invoice for the client can be sent while incomplete | On billing-field save (informational) and on every invoice send attempt (blocking) | "Billing name is required before sending this client's first invoice." | Yes, at send time only -- not blocking on Client Detail's own save |
| billing_address | Required for billing completeness | Before any invoice for the client can be sent while incomplete | On billing-field save (informational) and on every invoice send attempt (blocking) | "Billing address is required before sending this client's first invoice." | Yes, at send time only -- not blocking on Client Detail's own save |
| tax_id | No validation beyond data type -- always optional | Always | -- | -- | No |
| client_name, status, currency and tax treatment | No validation beyond data type in this spec -- billing completeness depends only on billing_name and billing_address; client_name is governed by FEAT-01.SPEC-001, status by FEAT-01.SPEC-007/FEAT-01.SPEC-009, and currency and tax treatment by FEAT-15 | Always | -- | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Billing completeness | billing_name, billing_address | Complete only when both billing_name and billing_address are non-empty; tax_id does not factor into completeness | "This client's billing details are incomplete. Add a billing name and address before sending the first invoice." (shown on FEAT-09's send action when incomplete) |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Edit billing_name, billing_address, tax_id | Nadia (Freelancer) | Always | -- |
| View billing completeness state | Nadia (Freelancer) | Always | -- |
| View billing completeness state | Dana (Support Operator) | Always, inside a logged support session (FEAT-31), read-only | -- |
| Edit billing_name, billing_address, tax_id | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never | Not shown; billing details are managed on the freelancer's own Client Detail screen, not exposed in either contact's portal |
| Edit billing_name, billing_address, tax_id | Dana (Support Operator) | Never | The billing-edit controls are not rendered in Dana's read-only support session (FEAT-31); she sees the completeness indicator only |
| Send an invoice for the client | Nadia (Freelancer) | Only when billing_name and billing_address are both non-empty at the moment of send (this spec's completeness rule, re-evaluated on every send attempt) | Blocked on FEAT-09's send action with "This client's billing details are incomplete. Add a billing name and address before sending the first invoice." and a link to Client Detail's billing section |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| billing_complete (derived, not a stored field) | True when billing_name and billing_address are both non-empty | Recalculated on every billing-field save and read whenever FEAT-09 attempts to send any invoice for the client | No -- computed solely from billing_name and billing_address |

## Business Rules

- XBR-16 / ASMP-24: every invoice must carry the freelancer's business details and the client's billing details; sending is blocked until both sets of details exist. This spec owns the client-side half of that gate; FEAT-21 owns the freelancer-business-details half.
- The gate is evaluated fresh by FEAT-09 at the moment of every invoice send attempt for the client, against billing_name and billing_address as they currently stand -- it is not a one-time-only check that fires only for the client's first invoice. In the common case, once these fields are captured they are never cleared, so completing them before the first send effectively satisfies the gate permanently in practice; but there is no special-cased exemption for later sends -- this spec's authority is over the field state at each send attempt.
- If billing_name or billing_address is subsequently cleared after a successful first send, a later send attempt for that same client (whether or not it is nominally the client's "first" invoice) is blocked again on the same basis, since FEAT-09 re-reads current state at every send rather than remembering that an earlier send once succeeded.
- The indicator shown on Client Detail ("Billing details complete" / "Billing details incomplete -- required before the first invoice") is informational only; it does not itself block anything on FEAT-01.SPEC-004, since billing details can legitimately be added or cleared at any time -- only an actual send attempt triggers this gate's blocking behavior.

## Edge Cases

- **billing_name is filled but billing_address is empty** -- Completeness is false; the send-time block names the specific missing field: "Billing address is required before sending this client's first invoice."
- **Both fields contain only whitespace** -- Treated as empty for completeness purposes; whitespace-only input does not satisfy the non-empty requirement.
- **tax_id is left empty indefinitely** -- Never blocks anything; tax_id has no bearing on completeness at any point.
- **Billing details are completed, the first invoice sends successfully, and Nadia later clears billing_name from Client Detail** -- No invoice already sent is affected (invoices are immutable once sent, XBR-04); a future ad hoc invoice attempt would be blocked again, since FEAT-09's own send-time check re-reads current state and the client currently has billing_name empty, even though a first invoice has technically already gone out. This spec's authority is over the field state at each send attempt, not a one-time-only gate.
- **Two sessions edit billing_name and billing_address concurrently** -- Resolves last-write-wins per the dependency map's Contention note for Client; the completeness indicator reflects whichever save landed last.

## Acceptance Criteria

**FEAT-01.SPEC-010-AC-01:** Given a client has both billing_name and billing_address filled, when Nadia attempts to send that client's first invoice, then the send proceeds with no billing block.

**FEAT-01.SPEC-010-AC-02:** Given a client has billing_name filled but billing_address empty, when Nadia attempts to send that client's first invoice, then the send is blocked with "Billing address is required before sending this client's first invoice."

**FEAT-01.SPEC-010-AC-03:** Given a client has both billing fields empty, when Nadia views Client Detail, then the completeness indicator shows "Billing details incomplete -- required before the first invoice."

**FEAT-01.SPEC-010-AC-04:** Given a client has tax_id empty but both billing_name and billing_address filled, when Nadia attempts to send the first invoice, then the send proceeds, since tax_id never factors into completeness.

**FEAT-01.SPEC-010-AC-05:** Given billing_address contains only whitespace, when completeness is evaluated, then it is treated as empty and the client is incomplete.

**FEAT-01.SPEC-010-AC-06:** Given a client's first invoice has already been sent successfully, when Nadia later clears billing_name on Client Detail, then the already-sent invoice is unaffected, but a subsequent ad hoc invoice attempt for that client is blocked again until billing_name is restored.

**FEAT-01.SPEC-010-AC-07:** Given two sessions of Nadia's edit the same client's billing_address concurrently, when both save, then the later save wins and the completeness indicator reflects it.

**FEAT-01.SPEC-010-AC-08:** Given Owen has no access to billing-detail fields, when he views his portal, then no billing-edit control for his company's details is ever shown to him.

**FEAT-01.SPEC-010-AC-09:** Given Dana views a client in a read-only support session, when she looks at the billing completeness indicator, then she sees its current state but has no control to edit the underlying fields.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
