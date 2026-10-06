---
document_type: spec
spec_type: integration
spec_id: FEAT-32.SPEC-002
spec_name: Payment Account Connection & Status Reporting
spec_slug: payment-account-connection-status-reporting
parent_feature: FEAT-32
parent_feature_name: Payment Account Connection
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 18
---

# Integration Spec: Payment Account Connection & Status Reporting

## Overview

**Name:** Payment Account Connection & Status Reporting
**ID:** FEAT-32.SPEC-002
**Type:** Integration
**Purpose:** Initiates the connect/reconnect hand-off with the payment-processing capability and receives back readiness status, the specific attention reason, available payment methods, and reversal/chargeback notices to relay onward.
**Parent Feature:** FEAT-32 -- Payment Account Connection

## Scope and Non-Goals

**In Scope:**
- Initiating a connect or reconnect hand-off for Nadia's own processor account
- Receiving the capability's reported readiness status, specific attention reason, and available payment methods
- Receiving a reported payment reversal or chargeback and correlating it to the affected invoice before relaying it onward
- User-facing behavior when the capability is slow, unavailable, or rejects the hand-off
- Disclosure to Nadia about what data is shared with the capability to open or link her account

**Non-Goals:**
- Choosing the payment-processing vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md's Ecosystem & Integrations section names only the category ("an established processor").
- Applying a reported status or methods change to the Payment Account Connection record -- owned entirely by FEAT-32.SPEC-003 (Connection Status Sync); this spec defines only the request/response contract with the capability.
- Removing the connection reference -- owned by FEAT-32.SPEC-004 (Disconnect Payment Account); disconnecting is Nadia's own in-product action and involves no request to this capability.
- Submitting a card or bank-transfer payment, or reporting its outcome -- owned entirely by FEAT-10.SPEC-003 (Card & Bank-Transfer Payment Processing); this spec's contract covers only the connection and its status, never a specific payment attempt.
- Recording the Disputed status or notifying Nadia about a reversal -- owned entirely by FEAT-25 (Refund & Cancelled Project Handling, XBR-21); this spec's responsibility ends at correlating the reported reversal to the right invoice and relaying the notice.
- Rendering the client-facing "online payment is temporarily unavailable" pay-link fallback -- owned by FEAT-09.SPEC-009 and FEAT-10 (XBR-19); this spec only supplies the status those specs read.

## Capability Category

**Category:** Payment processing (into each freelancer's own account)
**Dependency Source:** ASMP-28 -- Dependencies entry in assumptions-constraints.md naming the payment-processing capability
**External Touchpoint:** "Payment processing into each freelancer's own account -- connection, card and bank-transfer payment, status, pending transfers, reversals" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-09, FEAT-10, FEAT-20, FEAT-25, FEAT-32)
**Vendor Mandate:** None -- BRIEF.md, Ecosystem & Integrations names only the category ("an established processor takes card and bank-transfer payments directly into each freelancer's own account"); vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Nadia links her own existing or new processor account in a guided step | Connect her payment account | FEAT-32.SPEC-001 (Payment Connection Screen), FEAT-32.SPEC-003 (Connection Status Sync) |
| Nadia sees whether card and bank-transfer payments are ready to accept, and the specific reason when they are not | See connection status | FEAT-32.SPEC-001, FEAT-32.SPEC-003 |
| Nadia fixes a broken connection by reconnecting | Reconnect or disconnect | FEAT-32.SPEC-001, FEAT-32.SPEC-003 |
| A reported chargeback or reversal on a paid invoice reaches Nadia as a Disputed status and a notice | Cross-feature: XBR-21 | FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording), which fires FEAT-25.SPEC-008 (Payment Reversal Notification) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Freelancer's name and business details | Freelancer Account -- name, business_name, business_address | Nadia initiates a Connect or Reconnect hand-off | The capability must identify who it is opening or linking an account for |
| Freelancer's sign-in email | Freelancer Account -- sign-in email | Nadia initiates a Connect or Reconnect hand-off | Account identification and the capability's own account-recovery contact |

**Record lifecycle around a hand-off (this spec's creation step):** Nadia's Continue in the consent notice on FEAT-32.SPEC-001 is the only event that starts a hand-off. Immediately before contacting the capability, this spec (1) for a first connect (no record exists), creates the Payment Account Connection record with `status` Connecting, no `processor_account_reference`, `handoff_started_at` set to the Continue time, and `last_event_reported_at` set to the same time; or (2) for a Reconnect (record in Needs attention), leaves `status` unchanged and sets `handoff_started_at` and advances `last_event_reported_at` to the Continue time. The Connecting status and `handoff_started_at` are what FEAT-32.SPEC-001 reads to show the Connecting state again after Nadia navigates away and returns. If the hand-off fails or is abandoned, FEAT-32.SPEC-003 deletes the interim Connecting record (first connect) or clears `handoff_started_at` (Reconnect), so no half-created record is left behind. Cancel in the consent notice creates and changes nothing.

Card numbers, bank credentials, client data, invoice content, and payment amounts never leave the product through this spec -- credential entry happens entirely inside the payment-processing capability's own flow, and payment-level data is exchanged separately by FEAT-10.SPEC-003.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Readiness confirmation | The hand-off completes and the capability reports the account is ready | Routed to FEAT-32.SPEC-003, which sets Payment Account Connection -- `status` (Connected), `processor_account_reference`, `available_payment_methods` |
| Restriction or information request, with a specific reason | The capability restricts the account or asks for more information, at any point after connecting | Routed to FEAT-32.SPEC-003, which sets Payment Account Connection -- `status` (Needs attention, carrying the reason as descriptive text) |
| Hand-off failure or abandonment | The connect/reconnect attempt does not complete | Routed to FEAT-32.SPEC-003, which deletes the interim Connecting record (first connect) or clears `handoff_started_at` and leaves the rest of the record unchanged (Reconnect); surfaced inline on FEAT-32.SPEC-001 |
| Reversal or chargeback notice | The capability reports a dispute on a previously paid invoice | Correlated against Invoice (read-only, by the invoice reference the notice carries) and relayed to FEAT-25, which owns the resulting Disputed status; no field of Payment Account Connection is changed by this event |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Connection confirmed ready | The capability reports the account is ready to accept payments, following a Connect or Reconnect hand-off | Routed to FEAT-32.SPEC-003 for multi-step application (status, reference, methods) | Applied by FEAT-32.SPEC-003; then shown on FEAT-32.SPEC-001 as "Ready to accept payments" -- or, if the report lists zero available payment methods, as Needs attention with SPEC-003's defined zero-methods reason | FEAT-32.SPEC-003 |
| Connection restricted / more information requested | The capability restricts the account or asks for more information, at any point (including after previously being Connected) | Routed to FEAT-32.SPEC-003 for multi-step application (status, reason, methods) | Applied by FEAT-32.SPEC-003; then shown on FEAT-32.SPEC-001 with the specific reason | FEAT-32.SPEC-003, FEAT-32.SPEC-006 |
| Hand-off fails or is abandoned mid-flow | Nadia's connect or reconnect attempt does not complete, technically or because she exits before completion | Routed to FEAT-32.SPEC-003, which removes the interim record of a first connect or clears the in-progress marker of a Reconnect | FEAT-32.SPEC-001 shows the Error state with Retry over the previous state; no false Connected or Needs-attention state is shown | FEAT-32.SPEC-001, FEAT-32.SPEC-003 |
| Reversal or chargeback reported | The capability reports a dispute on an invoice already recorded as Paid, at any time after payment | The affected Invoice is identified by the reference the notice carries; the notice is relayed to FEAT-25.SPEC-005 | No feedback from this spec directly -- FEAT-25 owns the Disputed status display and Nadia's notice (XBR-21) | FEAT-25.SPEC-005 (Payment Reversal (Chargeback) Recording) |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-32.SPEC-001 (Payment Connection Screen) | The Connecting state continues showing "Setting up your connection -- this usually takes a few minutes."; if the hand-off has not confirmed within platform parameter: `payment-connection-handoff-slow-threshold`, the screen adds "Still checking your connection -- this is taking longer than usual." Nothing else on the screen is blocked. | The Connect/Reconnect button is disabled with "Connecting payment accounts is temporarily unavailable. Try again in a few minutes." Any existing connection's last-known status remains displayed unchanged. | The Error state applies: "We couldn't complete the connection. Try again." The previous status (if any) remains intact underneath; the interim Connecting record created for a first connect is deleted by FEAT-32.SPEC-003 on the failure event, so no half-created connection record persists. |

## Consent and Disclosure

- **First connect/reconnect disclosure** -- Before a hand-off is initiated (whether Connect, Reconnect, or Retry after a failed attempt), a notice appears: "To connect a payment account, we'll share your name, business details, and sign-in email with the payment-processing capability so it can open or link your account. We never see or store your card numbers or bank credentials." with "Continue" and "Cancel" options. Shown before every hand-off, since each one is a distinct, deliberate action Nadia takes. The dialog itself (heading, buttons, focus handling, Escape) is defined in FEAT-32.SPEC-001's Layout, Interactions, and Accessibility Notes.
- **Cancel outcome** -- "Continue" starts the hand-off and creates or marks the record as described under Data Exchanged. "Cancel" (or Escape) closes the notice: no hand-off is initiated, nothing is sent to the capability, no record is created or changed, and FEAT-32.SPEC-001 stays in the state it was in before the tap, with no error and no message.
- **What is never shared** -- Client data, invoice content, and payment amounts stay inside the product for this spec's purposes; card numbers and bank credentials are entered entirely inside the capability's own flow and never pass through the product (BRIEF.md, Constraints; ASMP-24; SC-10).

## Edge Cases

- **The readiness-confirmed event is delivered twice for the same hand-off** -- FEAT-32.SPEC-003's already-applied guard means the second delivery changes nothing; the record already shows Connected and no duplicate confirmation email fires (FEAT-32.SPEC-006).
- **Events arrive out of order (a restriction event followed by a stale readiness event from an earlier moment)** -- FEAT-32.SPEC-003 applies the most recent event by the time the capability reports it occurred, not by arrival order; a late-arriving, now-stale readiness report does not overwrite a more recent restriction.
- **A status-report event arrives for a connection Nadia has since disconnected** -- The event finds no record to update (FEAT-32.SPEC-004 hard-deletes the reference on disconnect) and is discarded without recreating a record; reconnecting always starts a fresh hand-off through this spec rather than resuming a stale report.
- **A reversal notice arrives for an invoice that no longer exists (removed via FEAT-24 account deletion)** -- The notice cannot be correlated to a record that no longer exists and is discarded; there is no freelancer left to notify.
- **The capability goes down mid-hand-off, after Nadia has already been asked to authorize but before readiness is confirmed** -- The outage is reported as a failed hand-off, so FEAT-32.SPEC-003 deletes the interim Connecting record (first connect) or clears the in-progress marker (Reconnect); no connection record is left in a half-confirmed state; FEAT-32.SPEC-001 shows the capability-down message and any pre-existing record keeps its prior status.
- **Nadia navigates away mid-hand-off and returns** -- The interim Connecting record (or the Reconnect's `handoff_started_at`) persists, so FEAT-32.SPEC-001 shows the Connecting state again until the capability reports an outcome.
- **Nadia initiates Reconnect while a status-report event from the previous connection attempt is still in flight** -- The new hand-off is a distinct attempt; FEAT-32.SPEC-003 applies whichever event is most recent by report time, so a stale event from the superseded attempt cannot overwrite the new hand-off's outcome.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-32.SPEC-001 (Payment Connection Screen) | Triggered by (inbound) | Connect and Reconnect taps initiate a hand-off through this spec |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Affects (outbound) | Hand-off progress, the Error state, and the disclosure notice (with its Continue and Cancel outcomes) surface here |
| FEAT-32.SPEC-003 (Connection Status Sync) | Triggers (outbound) | Every inbound event above is routed to this automation for application |
| FEAT-32.SPEC-005 (Payment Connection Authorization & Validation Rules) | References (inbound) | The Nadia-only gate and one-account-per-freelancer limit are checked before a hand-off reaches this spec |
| FEAT-25 (Refund & Cancelled Project Handling) | Affects (outbound) | Reversal and chargeback notices are relayed here (XBR-21) |
| FEAT-09.SPEC-009 (Pay-Link Availability & No-Account Fallback Rule) | References (outbound) | Reads the connection status this spec ultimately produces, via FEAT-32.SPEC-003, to derive pay-link availability |
| FEAT-10.SPEC-003 (Card & Bank-Transfer Payment Processing) | References (outbound) | Depends on the connection's readiness and available methods before a payment can be submitted |

## Analytics and Success Signals

- **payment_account_connect_started** (attempt type: connect / reconnect) -- supports success-metrics.md: "Payment Readiness Before First Invoice"
- **payment_account_handoff_outcome_received** (outcome: ready / restricted / failed_abandoned) -- supports success-metrics.md: "Payment Readiness Before First Invoice"
- **payment_account_reversal_notice_relayed** (invoice reference) -- N/A -- no Stage 2 metric measures reversal frequency; retained so a relayed dispute is observable rather than silent, per XBR-21's evidentiary intent.
- **payment_account_degradation_shown** (condition: slow / down / rejected) -- N/A -- no Stage 2 metric measures capability trouble frequency directly; retained so the product's tolerance for capability trouble during the readiness-critical connect flow is observable.

## Acceptance Criteria

**FEAT-32.SPEC-002-AC-01:** Given Nadia has no connected payment account, when she initiates the Connect hand-off, then the first-connect disclosure notice appears before any data leaves the product.

**FEAT-32.SPEC-002-AC-02:** Given Nadia has seen and accepted the disclosure notice, when the hand-off proceeds, then her name, business details, and sign-in email are shared with the capability and no card or bank credential data is ever sent.

**FEAT-32.SPEC-002-AC-03:** Given a hand-off is in progress, when the capability reports the account is ready, then this spec routes the readiness confirmation to FEAT-32.SPEC-003.

**FEAT-32.SPEC-002-AC-04:** Given a Connected account, when the capability restricts it or requests more information, then this spec routes the restriction event, including its specific reason, to FEAT-32.SPEC-003.

**FEAT-32.SPEC-002-AC-05:** Given a hand-off is in progress, when it fails or is abandoned before completing, then this spec routes that outcome to FEAT-32.SPEC-003, which applies no change to the existing record.

**FEAT-32.SPEC-002-AC-06:** Given a previously paid invoice, when the capability reports a chargeback or reversal against it, then this spec correlates the notice to that invoice and relays it to FEAT-25.

**FEAT-32.SPEC-002-AC-07:** Given Nadia is mid-hand-off and the capability is slow to respond, when platform parameter: `payment-connection-handoff-slow-threshold` elapses, then FEAT-32.SPEC-001 adds "Still checking your connection -- this is taking longer than usual."

**FEAT-32.SPEC-002-AC-08:** Given the capability is down, when Nadia attempts to Connect or Reconnect, then the button is disabled with "Connecting payment accounts is temporarily unavailable. Try again in a few minutes." and any existing connection's status is unaffected.

**FEAT-32.SPEC-002-AC-09:** Given the capability rejects a hand-off outright, when the rejection is reported, then FEAT-32.SPEC-001 shows "We couldn't complete the connection. Try again." with the previous status (if any) intact.

**FEAT-32.SPEC-002-AC-10:** Given a readiness-confirmed event is delivered twice for the same hand-off, when the second delivery arrives, then FEAT-32.SPEC-003's already-applied guard means nothing further changes and no duplicate confirmation email fires.

**FEAT-32.SPEC-002-AC-11:** Given events arrive out of order, when a stale readiness event arrives after a more recent restriction event, then FEAT-32.SPEC-003 applies the most recent event by report time, leaving the restriction in place.

**FEAT-32.SPEC-002-AC-12:** Given Nadia has already disconnected her account, when a stale status-report event for the removed connection arrives, then it is discarded and no record is recreated.

**FEAT-32.SPEC-002-AC-13:** Given an invoice was removed through FEAT-24 account deletion, when a reversal notice referencing it arrives afterward, then the notice is discarded since it cannot be correlated to an existing record.

**FEAT-32.SPEC-002-AC-14:** Given the capability goes down mid-hand-off before readiness is confirmed, when the outage is detected, then FEAT-32.SPEC-003 deletes the interim Connecting record (first connect) or clears the in-progress marker (Reconnect), no half-confirmed record remains, and FEAT-32.SPEC-001 shows the capability-down message.

**FEAT-32.SPEC-002-AC-15:** Given Nadia's connection is in Needs attention, when she taps Reconnect, then the same disclosure notice appears before any data leaves the product.

**FEAT-32.SPEC-002-AC-16:** Given the disclosure notice is showing, when Nadia taps "Cancel," then no hand-off starts, nothing is sent to the capability, no record is created or changed, and FEAT-32.SPEC-001 remains in its prior state.

**FEAT-32.SPEC-002-AC-17:** Given no record exists and Nadia taps "Continue" in the notice, when the hand-off starts, then this spec creates the record with `status` Connecting and `handoff_started_at` set, before contacting the capability; for a Reconnect it sets only `handoff_started_at` and leaves `status` Needs attention.

**FEAT-32.SPEC-002-AC-18:** Given a first-connect hand-off fails or is abandoned, when FEAT-32.SPEC-003 applies the outcome, then the interim Connecting record no longer exists and no half-created record remains.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 3 (1 screen) | 3 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 7 | 7 |
