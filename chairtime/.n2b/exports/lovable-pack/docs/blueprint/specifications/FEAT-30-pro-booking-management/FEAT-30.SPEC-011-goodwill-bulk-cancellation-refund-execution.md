---
document_type: spec
spec_type: integration
spec_id: FEAT-30.SPEC-011
spec_name: Goodwill & Bulk-Cancellation Refund Execution
spec_slug: goodwill-bulk-cancellation-refund-execution
parent_feature: FEAT-30
parent_feature_name: Pro Booking Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Integration Spec: Goodwill & Bulk-Cancellation Refund Execution

## Overview

**Name:** Goodwill & Bulk-Cancellation Refund Execution
**ID:** FEAT-30.SPEC-011
**Type:** Integration
**Purpose:** Requests each goodwill or bulk-cancellation refund from the payment-processing capability, drawing on the Pro's connected payout account, guarantees each refund completes exactly once with automatic retry, and reports back any refund that cannot complete immediately.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Requesting a full refund for a specific Deposit Transaction once FEAT-30.SPEC-009 (goodwill) or FEAT-30.SPEC-008 (bulk cancellation, per booking) determines one is due
- Receiving and applying the refund outcome (succeeded, cannot complete immediately) to the Deposit Transaction
- Guaranteeing at most one successful refund per Deposit Transaction, with automatic, indefinite retry until a not-yet-completable attempt resolves
- User-facing behavior when the payment-processing capability is slow, unavailable, or rejects the refund request
- Disclosure of what data this refund request shares with the capability

**Non-Goals:**
- Determining that a refund is due in the first place, or the Pro's own decision to issue one -- owned by FEAT-30.SPEC-009 (Goodwill Refund Commit) and FEAT-30.SPEC-008 (Bulk Cancellation Commit); this spec only executes a refund already determined
- Executing a single Pro-initiated cancellation's or reschedule's refund -- owned by FEAT-09.SPEC-005 (Automatic Deposit Refund), which handles that case per the External Touchpoints table's assignment; this spec covers only the refund calls FEAT-09 never evaluates or batches: goodwill refunds and the per-booking refund set a bulk cancellation produces
- Choosing the payment-processing vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate
- Capturing the original deposit charge, or routing it to the Pro's payout account in the first place -- owned by FEAT-07 (Deposit Payment at Booking); this spec only reverses an already-captured amount

## Capability Category

**Category:** Payment processing
**Dependency Source:** ASMP-31 -- "Payment-processing capability... required to take client deposits, verify each pro's identity and bank details for a connected payout account, pay deposits out to the pro, issue refunds, notify the product of card-issuer disputes, and bill the pro's own monthly subscription." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Payment processing -- client card charges and refunds (deposits; from v1 balances; Later tips)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-07, FEAT-09, FEAT-30, FEAT-22, FEAT-23; this spec is named directly as "FEAT-30.SPEC-011 (goodwill refunds and the per-booking refund set of a Pro bulk cancellation, drawing on the Pro's payout account, with not-yet-completable refunds reported back for retry; a single Pro cancellation's refund is executed by FEAT-09.SPEC-005)")
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Talia issues a goodwill refund and Riley sees her deposit refunded in full | Refund a deposit in full as goodwill | FEAT-30.SPEC-003 (Goodwill Deposit Refund) shows the confirmed refund |
| Talia cancels several bookings at once and every affected client's deposit refunds | Cancel several bookings at once | FEAT-30.SPEC-005 (Cancel Several Bookings at Once) shows the per-booking refund outcome |
| If a refund cannot complete immediately, both Talia and Riley see it as "in progress" rather than silently failing | Refund a deposit in full as goodwill; cancel several bookings at once | FEAT-12 (Pro Daily Schedule Dashboard) attention flag; Riley's own booking status view |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Refund amount and currency | Deposit Transaction -- amount, currency | FEAT-30.SPEC-009 or FEAT-30.SPEC-008 determines a refund is due | The capability must know exactly how much to return, matching the original captured amount |
| Deposit reference | Deposit Transaction -- the reference tying it to the original capture | Refund is requested | Ties the refund to the specific original charge so the capability reverses the correct transaction |
| Payout account reference | Payout Account -- processor_account_reference | Refund is requested | Identifies which of the Pro's connected accounts the refund draws against |
| Idempotency key | Derived -- an identifier tied to this specific refund attempt on this specific Deposit Transaction | Every refund request, including retries | Lets the capability recognize a resubmitted request as the same attempt rather than a second refund |

Booking details (service, client note, appointment time), the Client's contact fields, and every other Deposit Transaction field beyond amount, currency, and the deposit reference never leave the product for this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Refund succeeded confirmation, with refund timestamp | The capability completes the refund | Deposit Transaction -- status (Refunded), refund timestamp |
| Refund cannot complete immediately (e.g., the Pro's payout balance cannot yet cover it), with a retry-eligibility signal | The capability reports the refund could not be completed on this attempt | Deposit Transaction -- status (Refund in Progress); scheduled for this spec's own automatic retry |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Refund succeeded | The capability completes the requested refund | Deposit Transaction.status set to Refunded; refund timestamp recorded | Included in the relevant confirmation content (FEAT-30.SPEC-012), not a separate standalone notice; Riley sees her deposit refunded on her own booking view; the refund outcome is passed to FEAT-25.SPEC-004 (Historical Aggregate Maintenance) so revenue aggregates net out the refunded deposit | FEAT-30.SPEC-009, FEAT-30.SPEC-008, FEAT-30.SPEC-012, FEAT-25.SPEC-004 |
| Refund could not complete immediately | The capability reports it cannot complete the refund on this attempt (for example, the Pro's payout balance cannot yet cover it) | Deposit Transaction.status set to Refund in Progress; a retry is scheduled on a fixed cadence (platform parameter: `refund-retry-interval-hours`) | Talia's dashboard shows a clear attention flag (FEAT-08.SPEC-006); Riley's booking view shows the refund as "in progress," never as failed or silent (FEAT-30.SPEC-012) | FEAT-12, FEAT-30.SPEC-012, FEAT-25.SPEC-004 |
| Refund completes after a retry | A scheduled retry succeeds on a later attempt | Deposit Transaction.status set to Refunded; refund timestamp recorded | Talia's attention flag clears; Riley's "in progress" status updates to refunded, and the follow-up confirmation reflects the completed refund; FEAT-25.SPEC-004 is signalled of the completed refund | FEAT-12, FEAT-30.SPEC-012, FEAT-25.SPEC-004 |

## Degradation Behavior

This integration is triggered by FEAT-30.SPEC-009's and FEAT-30.SPEC-008's automations, not directly by a screen action, so no screen sends the refund request itself; the rows below cover the screens where this capability's trouble is visible to a user.

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-12 (Pro Daily Schedule Dashboard, attention list) | No visible change -- the refund request is not user-initiated on this screen, so a slow response produces no waiting state here; the attention flag simply has not yet appeared | If the capability is unreachable when the refund is requested, the attention flag reads "A refund for {client name}'s cancelled booking is in progress and will complete automatically." -- Talia sees no action she needs to take, and the flag persists until this spec's retry succeeds | If the capability explicitly rejects the refund request (for example, the payout account is no longer valid), the attention flag reads "A refund for {client name}'s cancelled booking needs attention -- your payout account may need reconnecting." with a link into FEAT-28 |
| Riley's own booking status view (FEAT-06/FEAT-10) | No visible change -- Riley sees no waiting state for a refund that has not yet been requested to fail or succeed | Riley's booking shows "Your deposit refund is in progress and will complete automatically." -- never a failure message | Riley's booking shows the same "in progress" wording; she is never shown the capability's rejection reason directly, since the resolution (e.g., reconnecting the payout account) is Talia's action, not hers |
| FEAT-30.SPEC-003 (Goodwill Deposit Refund) | No visible change -- Talia's confirm has already completed (Committing) by the time this integration's request reaches the capability, so no additional waiting state appears on the screen itself | Talia's confirm resolves to the screen's "Committed -- in progress" state ("Refund started. It will complete automatically -- you don't need to do anything.") rather than the immediate "Refund sent" confirmation, and FEAT-12's attention flag persists until this integration's retry succeeds | Same "Committed -- in progress" outcome is shown to Talia; the rejection itself surfaces on FEAT-12's attention flag (with the payout-reconnection link into FEAT-28), never as a failure on this screen, and the goodwill refund's already-committed decision is never reversed |
| FEAT-30.SPEC-005 (Cancel Several Bookings at Once, outcome summary) | No visible change -- the bulk commit's confirm has already completed by the time refunds process | A booking whose refund cannot complete immediately still shows in the per-booking summary as "Cancelled -- refund in progress", never as a failure | Same "refund in progress" wording; a rejected refund never blocks or reverses that booking's already-committed cancellation |

## Consent and Disclosure

- **No new disclosure moment for the refund itself** -- Riley already agreed, at the moment she paid her deposit (FEAT-07), that the payment-processing capability holds and processes her payment method; reversing that same capture through the same capability requires no additional consent screen.
- **Payout account reference disclosure** -- Talia was told, when she connected her payout account (FEAT-28), that it would be used to receive deposits and to fund refunds and payouts; this integration's use of that same reference for a goodwill or bulk-cancellation refund draws on that existing disclosure and requires no repeated notice.
- **What is never shared** -- Booking details, the Client's contact fields, and every Deposit Transaction field beyond amount, currency, and the deposit reference stay inside the product; the refund request never carries client contact information to the capability.

## Edge Cases

- **A refund-succeeded event arrives for a Deposit Transaction already marked Refunded** -- The second delivery changes nothing: the Deposit Transaction stays Refunded with its original refund timestamp, and no duplicate confirmation content fires, per the idempotency key discipline.
- **A refund-could-not-complete event arrives after a refund-succeeded event for the same deposit (out-of-order delivery)** -- The Deposit Transaction reflects the most recent true state, not arrival order: since a deposit can be refunded only once, a genuine refund-succeeded event is authoritative and a stale not-yet-completed report arriving late is treated as superseded and produces no status change.
- **Two overlapping retry attempts for the same Deposit Transaction are both processed** -- Only one results in a Refunded status; the other, whichever resolves second, finds the Deposit Transaction already Refunded via the idempotency key and is treated as a no-op with no duplicate refund.
- **Talia's payout account is reconnected mid-retry (the blocking condition resolves before the next scheduled attempt)** -- The next scheduled retry (platform parameter: `refund-retry-interval-hours` after the previous attempt) picks up the now-resolved payout account automatically; the refund completes on that attempt with no separate action from Talia beyond reconnecting.
- **A refund stays in progress for an extended period because the underlying blocking condition never resolves** -- The retry continues indefinitely on its fixed cadence; the Deposit Transaction is never silently abandoned or moved to a terminal failure state, and Talia's attention flag persists throughout, per XBR-10's "never dropped" guarantee.
- **The Booking or Client the refund relates to is deleted or de-identified before the refund event arrives** -- The event is still applied to the retained, de-identified financial record (per SC-22); no user feedback fires since there is no longer an active client-facing view to show it to.
- **One booking in a bulk cancellation's refund cannot complete immediately while its siblings' refunds succeed** -- Each Deposit Transaction is tracked and retried independently; the slow booking's "in progress" status never blocks or delays the confirmed refunds of the other bookings in the same bulk action.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-009 (Goodwill Refund Commit) | Triggered by (inbound) | A determined-due goodwill refund initiates this integration's request |
| FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Triggered by (inbound) | Each booking's determined-due refund within a bulk cancellation initiates its own request here |
| FEAT-28 (Payout Account Connection & Payout Visibility) | References (outbound) | Refunds draw on the Pro's connected payout account balance |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | A refund that cannot complete immediately is flagged clearly on the Pro's dashboard until it resolves |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggers (outbound) | A refund failure on this integration's first attempt fires the Pro-facing attention alert |
| FEAT-30.SPEC-012 (Pro Action Client Notice) | Triggers (outbound) | The refund outcome (confirmed or in progress) feeds the client notice this integration's callers trigger |
| FEAT-09.SPEC-005 (Automatic Deposit Refund) | References (outbound) | The sibling integration that executes a single Pro cancellation's or reschedule's own refund; this spec never duplicates that path |
| FEAT-25.SPEC-004 (Historical Aggregate Maintenance) -- within FEAT-25 (Booking & Revenue Insights) | Triggers (outbound) | Each Deposit Transaction outcome this integration applies (Refund in Progress, Refunded) is an inbound event to that automation, which updates the insights aggregates |
| FEAT-23.SPEC-003 (Tip Payout & Refund Rule) -- within FEAT-23 (Tipping at Checkout) | Enforces (outbound) | Enforces that rule for the refund set this integration executes: where a booking has a tipped, succeeded Balance Payment, the full-refund-including-tip guarantee applies, and the tip is refunded only as part of the Balance Payment's own Refunded transition (FEAT-22.SPEC-005), never as a standalone refund; this integration adds no tip-handling logic of its own |
| FEAT-09.SPEC-006 (Refund Idempotency & Retry Rule) | References (outbound) | This spec's own idempotency-key and retry-cadence discipline mirrors that spec's guarantee for FEAT-09's refund path, per the Brief's Shared Validation note |

## Analytics and Success Signals

- **goodwill_bulk_refund_requested** (source: goodwill / bulk_cancellation) -- supports success-metrics.md: "Automatic Refund Correctness"
- **goodwill_bulk_refund_outcome_received** (outcome: succeeded / could-not-complete) -- supports success-metrics.md: "Automatic Refund Correctness"
- **goodwill_bulk_refund_retry_scheduled** (attempt_number) -- supports success-metrics.md: "Automatic Refund Correctness" (a refund that cannot complete immediately must still be shown as "in progress" and complete without either party chasing it; this event measures how often the retry path is exercised)

## Acceptance Criteria

**FEAT-30.SPEC-011-AC-01:** Given FEAT-30.SPEC-009 determines a goodwill refund is due, when this integration requests it and the capability confirms immediately, then the Deposit Transaction is set to Refunded with a refund timestamp, and Riley sees the confirmation reflecting her refund.

**FEAT-30.SPEC-011-AC-02:** Given FEAT-30.SPEC-008 determines a refund is due for one booking in a bulk cancellation, when this integration requests it and the capability confirms immediately, then that booking's Deposit Transaction is set to Refunded independently of the other bookings in the set.

**FEAT-30.SPEC-011-AC-03:** Given a refund request is sent to the capability, when the capability reports it cannot complete on this attempt, then the Deposit Transaction is set to Refund in Progress, Talia's dashboard shows the attention flag, and Riley's booking shows "in progress," never a failure.

**FEAT-30.SPEC-011-AC-04:** Given the capability is unreachable when a refund is requested, then Talia's dashboard shows "A refund for {client name}'s cancelled booking is in progress and will complete automatically." with no action required from her yet.

**FEAT-30.SPEC-011-AC-05:** Given the capability explicitly rejects a refund request because the payout account is no longer valid, then Talia's dashboard shows "A refund for {client name}'s cancelled booking needs attention -- your payout account may need reconnecting." with a link into FEAT-28.

**FEAT-30.SPEC-011-AC-06:** Given a refund that could not complete immediately, when the next scheduled retry runs (platform parameter: `refund-retry-interval-hours` after the previous attempt), then a new refund request is submitted carrying the same attempt's idempotency key.

**FEAT-30.SPEC-011-AC-07:** Given a Deposit Transaction is already Refunded, when the same refund-succeeded event is delivered again, then nothing changes and no duplicate confirmation content fires.

**FEAT-30.SPEC-011-AC-08:** Given two overlapping retry attempts for the same Deposit Transaction both resolve, then only one results in a Refunded status, and the other is treated as a no-op with no duplicate refund.

**FEAT-30.SPEC-011-AC-09:** Given Talia reconnects her payout account while a goodwill refund sits at Refund in Progress, when the next scheduled retry runs, then the refund completes automatically with no further action from Talia.

**FEAT-30.SPEC-011-AC-10:** Given a refund's blocking condition never resolves, when successive scheduled retries continue to fail, then the Deposit Transaction remains Refund in Progress indefinitely rather than moving to a terminal failure state, and Talia's attention flag persists throughout.

**FEAT-30.SPEC-011-AC-11:** Given the Client whose deposit is being refunded has since been deleted, when the refund event arrives, then it is applied to the retained de-identified financial record with no user feedback fired.

**FEAT-30.SPEC-011-AC-12:** Given one booking's refund within a bulk cancellation cannot complete immediately while its siblings' refunds succeed, when the outcomes are reported, then the slow booking shows "refund in progress" while the others show refunded, independently.

**FEAT-30.SPEC-011-AC-13:** Given this integration requests a refund, when the request is composed, then only the refund amount, currency, deposit reference, payout account reference, and idempotency key are sent -- Booking details and Client contact fields are never included.

**FEAT-30.SPEC-011-AC-14:** Given a refund that could not complete immediately eventually succeeds after a retry, then the Deposit Transaction is set to Refunded, Talia's attention flag clears, and Riley's "in progress" status updates to reflect the completed refund.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 3 | 3 |
| Inbound Events | 3 | 3 |
| Degradation Paths | 12 (4 screens x 3 conditions) | 12 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 7 | 7 |
