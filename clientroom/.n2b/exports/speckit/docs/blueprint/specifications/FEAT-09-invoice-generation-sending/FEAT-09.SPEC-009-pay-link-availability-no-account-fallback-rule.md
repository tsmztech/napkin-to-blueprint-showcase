---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-09.SPEC-009
spec_name: Pay-Link Availability & No-Account Fallback Rule
spec_slug: pay-link-availability-no-account-fallback-rule
parent_feature: FEAT-09
parent_feature_name: Invoice Generation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 3
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Pay-Link Availability & No-Account Fallback Rule

## Overview

**Name:** Pay-Link Availability & No-Account Fallback Rule
**ID:** FEAT-09.SPEC-009
**Type:** Logic/Rule
**Purpose:** Governs what an invoice's pay link shows and how it is worded when Nadia has no connected, ready payment account, or when a previously working connection later needs attention or disconnects.
**Parent Feature:** FEAT-09 -- Invoice Generation & Sending
**Governed Entity:** Invoice (specifically the `pay_link availability` field, derived from Payment Account Connection)

## Scope and Non-Goals

**In Scope:**
- The three pay-link states an invoice can show (ready to pay, online payment not yet available, temporarily unavailable) and the exact wording for each
- Deriving the current state from the Payment Account Connection's status at the moment each invoice screen or email renders it
- The behavior when a connection that was working later needs attention or disconnects, on invoices already open

**Non-Goals:**
- Connecting, reconnecting, or disconnecting the payment account itself -- owned entirely by Payment Account Connection (FEAT-32); this spec only reads its status
- The actual payment flow once the pay link is followed -- owned by Invoice Payment Processing (FEAT-10)
- Recording a payment received outside the pay link -- owned by FEAT-10's off-platform recording capability; this spec only governs what the link itself shows before that happens

## Governed Entity

**Entity:** Invoice
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| pay_link availability | derived | The wording state shown on the invoice's pay link, derived from Payment Account Connection at render time |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-002 | Invoice Detail | Renders the pay-link status banner on every view |
| FEAT-09.SPEC-004 | Automatic Invoice Generation | Checks availability at generation time and carries the resulting wording state onto the new invoice |
| FEAT-09.SPEC-010 | Invoice Issued & Copy Confirmation Notification | Renders the same three states, worded identically, in the invoice email to Owen |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| pay_link availability | Must be one of exactly three states: Ready, Not yet available, Temporarily unavailable -- derived, never entered by any user | Always | Evaluated fresh at every render (screen view or email composition), never cached from generation time | Not applicable -- this is a derived display state, not a user-facing validation error | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|-----------------|
| Connection-status-drives-wording | pay_link availability, Payment Account Connection status | `Ready` when the connection's status is Connected; `Not yet available` when the status is Not connected; `Temporarily unavailable` when the status is Needs attention or Disconnected after having been previously connected | Not applicable -- these are display states, not blocking errors; each has its own exact wording, defined below |

## Authorization Rules

Not applicable to this spec -- who may follow the pay link at all is governed by FEAT-09.SPEC-006. This spec governs only what the link says once a role that may follow it (Owen) is looking at it.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| pay_link availability | Ready (Connection: Connected) -> exact wording "Ready to pay"; Not connected -> "Online payment isn't set up yet. [Freelancer name]'s direct payment instructions are below." with Nadia's stated alternative instructions; Needs attention or Disconnected (previously connected) -> "Online payment is temporarily unavailable. Please try again shortly, or contact [Freelancer name] directly." | Recalculated at every render, never stored as a frozen value on the invoice | No -- entirely derived from the live Payment Account Connection status; neither Nadia nor Owen can set it directly |

## Business Rules

- XBR-19: with no connected payment account, invoices still issue and send with instructions for paying the freelancer directly; when the connection needs attention, pay links show "online payment is temporarily unavailable"; disconnecting warns that open invoices lose their pay links. This spec is the FEAT-09-side instantiation of that rule.
- FEAT-32 (Payment Account Connection) is the sole owner and authority for the underlying connection status; this spec never writes to Payment Account Connection, only reads it (per the dependency map's External Touchpoints table: "FEAT-09 validated with no Integration spec for this capability: it derives pay-link availability from the connection status").
- The wording is identical wherever it appears -- FEAT-09.SPEC-002's banner and FEAT-09.SPEC-010's email render the same three states with the same text, per the Brief's Shared UI Patterns ("Pay-link status banner").
- An invoice generated while the connection is Not connected still issues and sends per FEAT-09.SPEC-004 -- this spec never blocks generation or sending; it only changes what the pay link says.

## Edge Cases

- **Nadia connects her payment account after several invoices were already sent with "not yet available" wording** -- Every open invoice's pay-link banner re-derives to Ready the next time it renders (screen view or a re-sent email is not triggered automatically); Owen sees "Ready to pay" the next time he opens any of those invoices, with no action required from Nadia beyond connecting.
- **The connection moves from Connected to Needs attention while Owen has an invoice detail screen open with the pay link visible** -- The banner updates in place to "Online payment is temporarily unavailable..." consistent with FEAT-09.SPEC-002's live-status-update behavior; Owen never completes a payment against a connection that has just become unready, since FEAT-10's own pay flow re-checks status at the moment of payment.
- **Nadia disconnects her account entirely while several invoices are open and unpaid** -- Each open invoice's pay link switches to the "temporarily unavailable" wording (XBR-19), and Nadia sees FEAT-32's own disconnect warning that open invoices will lose their pay links until she reconnects.
- **An invoice is Paid before the connection ever needed attention** -- The pay-link banner becomes moot once paid (FEAT-10 owns the Paid-state display); this spec's wording is never shown on an invoice already marked Paid.
- **Owen opens an invoice email sent while the connection was Ready, but the connection has since moved to Needs attention** -- The email's static content is not re-rendered after sending, but the live invoice detail page it links to (FEAT-09.SPEC-002) always reflects the current status; Owen sees the up-to-date "temporarily unavailable" wording once he opens the linked page, even though the email text itself is fixed at send time.
- **The connection has never existed at all (freelancer has never started the connect flow)** -- Treated identically to "Not connected" -- the "not yet available" wording and Nadia's direct-payment instructions apply from the very first invoice.

## Acceptance Criteria

**FEAT-09.SPEC-009-AC-01:** Given Nadia has a Connected, ready payment account, when an invoice is generated, then its pay link shows "Ready to pay."

**FEAT-09.SPEC-009-AC-02:** Given Nadia has never connected a payment account, when an invoice is generated and sent, then it shows "Online payment isn't set up yet." with her direct-payment instructions, and it still sends.

**FEAT-09.SPEC-009-AC-03:** Given Nadia's connection status is Needs attention, when Owen opens an open invoice, then he sees "Online payment is temporarily unavailable. Please try again shortly, or contact [Freelancer name] directly."

**FEAT-09.SPEC-009-AC-04:** Given Nadia's connection status is Disconnected after having previously been connected, when Owen opens an open invoice, then he sees the same "temporarily unavailable" wording as the Needs-attention state.

**FEAT-09.SPEC-009-AC-05:** Given Nadia connects her account after invoices were sent with "not yet available" wording, when Owen next opens any of those invoices, then the banner shows "Ready to pay" with no further action from Nadia.

**FEAT-09.SPEC-009-AC-06:** Given the connection changes from Connected to Needs attention while Owen has an invoice detail screen open, when the change lands, then the banner updates in place to the temporarily-unavailable wording.

**FEAT-09.SPEC-009-AC-07:** Given Nadia disconnects her account while invoices remain open and unpaid, when the disconnect completes, then every open invoice's pay link switches to "temporarily unavailable" wording.

**FEAT-09.SPEC-009-AC-08:** Given an invoice has already been marked Paid, when its detail is viewed, then the pay-link availability wording from this spec is not shown.

**FEAT-09.SPEC-009-AC-09:** Given Owen opens an invoice email sent while the connection was Ready but the connection has since changed, when he follows the link to the live invoice detail page, then he sees the current, up-to-date wording rather than the state at send time.

**FEAT-09.SPEC-009-AC-10:** Given Nadia has never started the connect flow at all, when her first invoice is generated, then it is treated identically to "Not connected" with the same wording and instructions.

**FEAT-09.SPEC-009-AC-11:** Given FEAT-09.SPEC-010 composes the invoice email at the same moment FEAT-09.SPEC-002 would render the detail banner, when both render, then they show the exact same wording for the same connection state.

**FEAT-09.SPEC-009-AC-12:** Given this spec is asked for a pay-link state and the connection status is any value other than Connected, Not connected, Needs attention, or Disconnected, then no such value exists in the product's definition of Payment Account Connection, so this condition never arises.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 0 (N/A -- owned by FEAT-09.SPEC-006) | 0 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
