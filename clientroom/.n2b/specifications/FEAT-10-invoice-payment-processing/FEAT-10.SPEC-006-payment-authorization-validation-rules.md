---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-10.SPEC-006
spec_name: Payment Authorization & Validation Rules
spec_slug: payment-authorization-validation-rules
parent_feature: FEAT-10
parent_feature_name: Invoice Payment Processing
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 30
acceptance_criteria_count: 23
---

# Logic/Rule Spec: Payment Authorization & Validation Rules

## Overview

**Name:** Payment Authorization & Validation Rules
**ID:** FEAT-10.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs who may pay, view, or manually record a payment, the full-payment-only and manual-record limits, the reject-with-refresh concurrency behavior, and the payment-account-readiness gate.
**Parent Feature:** FEAT-10 -- Invoice Payment Processing
**Governed Entity:** Payment

## Scope and Non-Goals

**In Scope:**
- Field validation rules for the Payment record (amount, method, paid_at)
- Cross-field rules tying a manual record's amount to the invoice total
- Authorization rules for every action on Payment (pay, retry, view, record manually), per role
- The payment-account-readiness gate and the reject-with-refresh concurrency rule
- Default values and derivations for Payment fields

**Non-Goals:**
- The Invoice entity's own broader lifecycle rules (Overdue flagging, reminder scheduling) -- owned by FEAT-09 and FEAT-11; this spec governs only the Payment-related actions and the Invoice `status` transitions those actions cause.
- Refund, reversal, or chargeback rules -- owned entirely by FEAT-25 (Refund & Cancelled Project Handling); this spec's authority ends once a Payment first reaches Succeeded, Failed, or Recorded manually.
- The mechanics of submitting a request to the payment-processing capability -- owned by FEAT-10.SPEC-003; this spec defines only who may initiate that submission and under what conditions.

## Governed Entity

**Entity:** Payment
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| invoice | text (reference) | The Invoice this payment is against |
| amount | number | The amount paid -- always the full invoice amount; no partial payments |
| method | enum | card \| bank transfer \| a recorded off-platform method (bank transfer, cash, cheque, other) |
| paid_at | date | The date/time the payment was made or confirmed |
| status | enum | Initiated \| Pending \| Succeeded \| Failed \| Reversed \| Recorded manually |
| recorded_by | text (reference) | Nadia's identity, present only for manually recorded payments |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-10.SPEC-001 | Pay Invoice Screen | On screen entry (Own-only gate, payment-account-readiness gate) and on Pay/Try again tap (reject-with-refresh) |
| FEAT-10.SPEC-002 | Record Off-Platform Payment Screen | On screen entry (Nadia-only gate) and on field blur/form submit (full-amount and not-future-dated validation) |
| FEAT-10.SPEC-003 | Card & Bank-Transfer Payment Processing | Before submitting a request (Own-only gate, readiness gate) |
| FEAT-10.SPEC-004 | Payment Confirmation & Invoice Status Sync | During processing (already-resolved guard against a stale in-flight outcome) |
| FEAT-10.SPEC-005 | Record Off-Platform Payment | During processing (full-amount, not-future-dated, and already-resolved checks) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| amount | Must equal the invoice's full `total` -- no partial amounts | Always | On save (FEAT-10.SPEC-003, FEAT-10.SPEC-005) | "This invoice must be paid in full." | Yes |
| method | Required; must be one of the valid values for the initiating path (card or bank transfer for FEAT-10.SPEC-003; bank transfer, cash, cheque, or other for FEAT-10.SPEC-005) | Always | On submit | "Choose a payment method." | Yes |
| paid_at (manual records only) | Cannot be a future date | Only for a manually recorded payment (FEAT-10.SPEC-005) | On blur and on submit (FEAT-10.SPEC-002) | "Payment date cannot be in the future." | Yes |
| paid_at (processor-confirmed records) | No validation beyond data type -- this value is set atomically by the payment-processing capability's own report, never entered by a user | Always | -- | -- | -- |
| status | No validation beyond data type -- transitions are governed entirely by Business Rules below, never by a user-entered value | Always | -- | -- | -- |
| recorded_by | No validation beyond data type -- always Nadia for a manual record, always absent for a processor-confirmed one; not a user input | Always | -- | -- | -- |
| invoice | No validation beyond data type -- fixed by the entry context (the invoice being paid or recorded) and never user-selected on this screen | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Full-payment-only | amount, invoice | A Payment's `amount` must equal its `invoice`'s `total` at the moment of creation -- there is no field for entering a different amount, so this rule is enforced structurally rather than as a separate check the user can fail, except that FEAT-10.SPEC-005's automation re-verifies it before creating the record | "This invoice must be paid in full." |
| Manual record requires an eligible invoice | status, invoice.status | A Payment with `status` = Recorded manually may only be created while the referenced Invoice's `status` is not already Paid or Paid (recorded by freelancer) | "This invoice was already paid online. Refresh to see the current status." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Pay an invoice (card or bank transfer) | Owen (Client Primary Contact) | Only his own company's invoice (Own-only); only while the invoice's status is Sent or Overdue (not already in a Paid-family status); only while the Payment Account Connection status is Connected | If the invoice is already Paid: the Pay control is replaced by the current status and a direct attempt is refused with the refreshed status shown (reject-with-refresh). If the Payment Account Connection is not Connected: the Pay control is replaced entirely by "Online payment is temporarily unavailable. Please contact {freelancer_first_name} to arrange payment another way." |
| Pay an invoice (card or bank transfer) | Nadia (Freelancer) | Never -- Nadia is not a payer or retry actor for her own invoices under any condition | No Pay control exists on any freelancer-side view of her own invoice; paying is exclusively a client-side action performed by the invoice's Client Primary Contact |
| Pay an invoice (card or bank transfer) | Priya (Client Reviewer Contact) | Never -- Invoicing & Payments is None for Reviewer contacts | "You don't have access to invoices for this account. Ask {client_name}'s primary contact to handle payment." |
| Pay an invoice (card or bank transfer) | Dana (Support Operator) | Never -- her access to Invoicing & Payments is View only, never Pay, regardless of session state | No Pay control is ever rendered for her, inside or outside a logged support session; a direct attempt has no control to trigger |
| Retry a failed or declined payment | Owen (Client Primary Contact) | Same conditions as Pay -- only his own company's invoice, only while not already Paid, only while the connection is ready | Same denied behavior as Pay |
| Retry a failed or declined payment | Nadia (Freelancer) | Never -- Nadia is not a payer or retry actor for her own invoices under any condition | No "Try again" control exists on any freelancer-side view of her own invoice; retrying is exclusively a client-side action performed by the invoice's Client Primary Contact |
| Retry a failed or declined payment | Priya (Client Reviewer Contact) | Never -- Invoicing & Payments is None for Reviewer contacts | "You don't have access to invoices for this account. Ask {client_name}'s primary contact to handle payment." |
| Retry a failed or declined payment | Dana (Support Operator) | Never -- her access to Invoicing & Payments is View only, never Pay or Retry, regardless of session state | No "Try again" control is ever rendered for her, inside or outside a logged support session; a direct attempt has no control to trigger |
| View payment status | Nadia (Freelancer) | Always, for any of her own invoices (Full) | -- |
| View payment status | Owen (Client Primary Contact) | Only his own company's invoice (Own-only) | Any invoice outside his own company's scope shows "You don't have access to invoices for this account." per XBR-09 |
| View payment status | Priya (Client Reviewer Contact) | Never -- Invoicing & Payments is None for Reviewer contacts | The invoice detail and pay link are never shown to her; a direct link attempt shows "You don't have access to invoices for this account. Ask {client_name}'s primary contact to handle payment." |
| View payment status | Dana (Support Operator) | Only inside a logged, read-only support session (FEAT-31), and only status -- never a Pay or Record control | Outside a support session, no access exists at all; inside one, any Pay or Record action attempt has no control to trigger, since none is rendered for her |
| Record an off-platform payment | Nadia (Freelancer) | Only for her own invoices; only while the invoice's status is not already Paid or Paid (recorded by freelancer); amount always the full total; date never in the future | If the invoice is already Paid online: "This invoice was already paid online. Refresh to see the current status." |
| Record an off-platform payment | Owen (Client Primary Contact) | Never | The "Record a payment received elsewhere" action does not exist on Owen's side of the product; he has no screen offering it |
| Record an off-platform payment | Priya (Client Reviewer Contact) | Never | Same as Owen -- no such action exists for any client contact |
| Record an off-platform payment | Dana (Support Operator) | Never | The action is never shown inside a support session; her access to Invoicing & Payments is View only |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| amount | Derived from the referenced Invoice's `total` at the moment the Payment record is created | On create only | No |
| paid_at (manual record) | Defaults to today's date on the entry screen (FEAT-10.SPEC-002) | On create only | Yes -- Nadia may choose any non-future date |
| paid_at (processor-confirmed) | Set by the payment-processing capability's own report at the moment of confirmation | On create/update (FEAT-10.SPEC-004) | No |
| status | Initiated at submission, transitioning to Pending, Succeeded, or Failed as the capability reports each stage (FEAT-10.SPEC-004); set directly to Recorded manually for a manual entry (FEAT-10.SPEC-005) | On create and on update | No -- transitions are system-driven, never user-selected |
| recorded_by | Set to Nadia automatically for a manual record; never set for a processor-confirmed record | On create only (manual path) | No |

## Business Rules

- The payment-account-readiness gate: paying or retrying is available only while the freelancer's Payment Account Connection status is Connected; a Needs attention or Disconnected status blocks the action entirely with the "temporarily unavailable" message, never a failed payment attempt (XBR-19).
- Reject-with-refresh concurrency: the first confirmed full payment on an invoice wins; any later attempt -- whether a second card/bank-transfer submission or a manual record -- is refused and the actor is shown the invoice's current, refreshed status rather than being allowed to process against a stale view (dependency map, Entity: Payment, Contention).
- Processor-authoritative-over-manual: when a processor-confirmed payment and a manual record could both apply to the same invoice, the processor's confirmation is authoritative; a manual record attempted after the processor has already confirmed payment is refused (XBR-20).
- An invoice is paid once and in full only -- no rule in this spec, nor any enforcing spec, defines a path to a partial payment or a second successful payment on the same invoice (XBR-20, scope-boundaries.md SC-17).
- Dana's View access to Invoicing & Payments never includes a Pay or Record control under any condition -- her role is bounded to read-only regardless of invoice or connection state (XBR-29).

## Edge Cases

- **Two payment attempts (a card submission and a bank-transfer submission) are started by Owen in two open tabs on the same invoice** -- Both submissions are accepted individually by FEAT-10.SPEC-003, but only the first to reach a Succeeded outcome sets the invoice to Paid; FEAT-10.SPEC-004's already-resolved guard refuses to let the second outcome overwrite the first, and Owen's second tab refreshes to show the invoice already Paid.
- **The Payment Account Connection changes from Connected to Needs attention between Owen loading the Pay Invoice Screen and tapping Pay** -- The readiness gate is re-checked authoritatively at the moment of submission, not just at page load; if it has changed, the attempt is refused with the same unavailable message rather than being sent to a connection that will reject it.
- **Owen's own invoice reaches exactly its due date at the moment he pays (boundary condition on Overdue)** -- Not a boundary this spec governs; the Overdue flag (owned by FEAT-11) has no bearing on payment eligibility, so payment proceeds identically whether the invoice is Sent or Overdue.
- **Nadia enters a manual-record date of exactly today** -- Passes validation; the not-future-dated rule is inclusive of the current date, matching FEAT-10.SPEC-005's own boundary handling.
- **A client contact's role changes from Reviewer to Primary while an invoice is outstanding (cross-feature: FEAT-18)** -- The newly promoted Primary contact gains Pay access from the moment the role change takes effect; the Authorization Rules above apply to the role as currently assigned, never to a role held at some earlier point.
- **Dana's support session ends while she is viewing an invoice's payment status** -- Her View access ends immediately with the session; no lingering access to payment status persists after FEAT-31 closes the session.

## Acceptance Criteria

**FEAT-10.SPEC-006-AC-01:** Given Owen is on his own company's invoice with a Sent status and the Payment Account Connection is Connected, when he attempts to pay, then the attempt is allowed.

**FEAT-10.SPEC-006-AC-02:** Given the invoice is already Paid, when Owen attempts to pay against his stale view, then the attempt is refused and he is shown the refreshed, current status.

**FEAT-10.SPEC-006-AC-03:** Given the Payment Account Connection status is Needs attention, when Owen attempts to pay, then the attempt is blocked with "Online payment is temporarily unavailable. Please contact {freelancer_first_name} to arrange payment another way."

**FEAT-10.SPEC-006-AC-04:** Given Owen's payment was declined, when he taps "Try again," then the same Own-only and readiness conditions are re-checked before the retry is submitted.

**FEAT-10.SPEC-006-AC-05:** Given Nadia views any of her own invoices, when she checks payment status, then she can always see it (Full access).

**FEAT-10.SPEC-006-AC-06:** Given Priya (Client Reviewer Contact) attempts to view any invoice, when the attempt is made, then it is denied with "You don't have access to invoices for this account. Ask {client_name}'s primary contact to handle payment."

**FEAT-10.SPEC-006-AC-07:** Given Dana is inside a logged support session, when she views an invoice's payment status, then she sees it read-only with no Pay or Record control anywhere on the screen.

**FEAT-10.SPEC-006-AC-08:** Given Nadia enters a valid date and method for an eligible invoice, when she records a manual payment, then it is allowed and the amount is set to the invoice's full total automatically.

**FEAT-10.SPEC-006-AC-09:** Given the invoice was already paid online before Nadia's save commits, when she attempts to record a manual payment, then it is refused with "This invoice was already paid online. Refresh to see the current status."

**FEAT-10.SPEC-006-AC-10:** Given Owen looks for a "Record a payment received elsewhere" action on his own screens, when he searches for it, then no such action exists for his role under any condition.

**FEAT-10.SPEC-006-AC-11:** Given Priya looks for any payment-related action, when she searches for it, then none is ever shown, since Invoicing & Payments is None for her role.

**FEAT-10.SPEC-006-AC-12:** Given Nadia attempts to record a manual payment with a future date, when she submits, then the amount rule is never reached because the date rule already blocks the save with "Payment date cannot be in the future."

**FEAT-10.SPEC-006-AC-13:** Given a Payment record is created by any path, when its `amount` is set, then it always equals the invoice's full `total` -- never a partial amount.

**FEAT-10.SPEC-006-AC-14:** Given Nadia enters exactly today's date for a manual record, when she saves, then the date passes validation.

**FEAT-10.SPEC-006-AC-15:** Given two submissions are made on the same invoice in two open sessions, when both reach the point of resolution, then only the first confirmed full payment is applied and the second is refused with the refreshed status.

**FEAT-10.SPEC-006-AC-16:** Given a processor-confirmed payment and a concurrent manual record both target the same invoice, when both attempt to resolve, then the processor's confirmation is authoritative and the manual attempt is refused.

**FEAT-10.SPEC-006-AC-17:** Given the Payment Account Connection changes to Needs attention between page load and Owen's tap on Pay, when he taps Pay, then the readiness gate is re-checked at that moment and the attempt is blocked.

**FEAT-10.SPEC-006-AC-18:** Given an invoice's due date has passed (Overdue), when Owen or Nadia interacts with payment on it, then the Overdue flag has no effect on any authorization or validation outcome in this spec.

**FEAT-10.SPEC-006-AC-19:** Given a client contact's role changes from Reviewer to Primary, when the change takes effect, then that contact immediately gains Pay access per the Authorization Rules above, with no lag.

**FEAT-10.SPEC-006-AC-20:** Given Dana's support session ends, when the session closes, then her View access to that invoice's payment status ends immediately with it.

**FEAT-10.SPEC-006-AC-21:** Given Nadia is viewing one of her own invoices, when she looks for a way to pay it or retry a failed payment herself, then no Pay or "Try again" control exists for her under any condition -- paying and retrying are exclusively Owen's actions.

**FEAT-10.SPEC-006-AC-22:** Given Priya (Client Reviewer Contact) attempts to pay an invoice or retry a failed payment, when the attempt is made, then it is denied with "You don't have access to invoices for this account. Ask {client_name}'s primary contact to handle payment."

**FEAT-10.SPEC-006-AC-23:** Given Dana is inside or outside a logged support session, when she looks for a way to pay an invoice or retry a failed payment, then no Pay or "Try again" control is ever rendered for her, regardless of session state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 7 | 7 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 16 | 16 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
