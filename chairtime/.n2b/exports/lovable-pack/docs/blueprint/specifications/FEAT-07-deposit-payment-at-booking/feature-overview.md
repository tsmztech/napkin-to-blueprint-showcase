---
document_type: feature-overview
feature_number: FEAT-07
feature_name: Deposit Payment at Booking
feature_slug: deposit-payment-at-booking
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 5
screen_count: 1
automation_count: 1
logic_rule_count: 2
integration_count: 1
notification_count: 0
---

# Feature Breakdown Brief: Deposit Payment at Booking

## Summary

**Feature:** Deposit Payment at Booking
**ID:** FEAT-07
**Description:** The client pays a card deposit -- a fixed amount or a percentage of the service price, per the Pro's own rule -- at the moment of booking, with the balance left due in person at the appointment.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md's Vision states the client "pays a card deposit," and the Business Context is explicit that the platform never stores or handles card data itself and takes no per-booking cut. This is the mechanism that solves the founder's core problem: deposits that used to be asked for by hand and often never arrived. [RESEARCH-INFORMED: added market validation -- deposit and card-on-file no-show protection is consistently credited with materially reducing no-shows across StyleSeat, Booksy and Fresha, from cross-referenced help-center data, feature documentation and Reddit-derived review summaries (3 sources, HIGH confidence)]

**Key Capabilities:**
- Pay the exact deposit amount required by the selected service's rule, by card
- See a clear on-screen and confirmed record that the deposit succeeded
- Have a failed or declined payment explained clearly, with the slot held briefly to retry
- Deposit lands directly in the Pro's own payout account (FEAT-28), with the platform taking no cut; the only deduction is the payment processor's own card fee, shown to the Pro

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-07.SPEC-001 | Deposit Payment | Screen | The Client | Client enters card details for the exact computed deposit and sees processing, decline/retry, and success states before handoff to booking confirmation |
| FEAT-07.SPEC-002 | Deposit Capture & Booking Confirmation | Automation | The Client, The Pro | On a successful charge, creates the Deposit Transaction record and atomically flips the Booking from Pending Payment to Confirmed, enforcing exactly one charge per booking |
| FEAT-07.SPEC-003 | Deposit Amount & Eligibility Rules | Logic/Rule | The Client, The Pro | Governs how the deposit amount is computed once from the service's rule, that it cannot be altered by the client, and the preconditions (active payout account, locked currency) that must hold before any charge is attempted |
| FEAT-07.SPEC-004 | Payment Outcome Consistency & Idempotency | Logic/Rule | The Client, The Pro | Guarantees every payment attempt ends in exactly one of a clean success or a clean, actionable failure -- never a double charge and never an ambiguous booking state, including when the confirmation UI itself fails to load or the connection drops mid-payment |
| FEAT-07.SPEC-005 | Card Deposit Charge & Payout Routing | Integration | The Client, The Pro | Authorizes and captures the client's card charge through the payment-processing capability, reports the processor's own card fee, and routes the captured deposit to the Pro's connected payout account with zero platform fee |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Pay the exact deposit amount required by the selected service's rule, by card | FEAT-07.SPEC-001, FEAT-07.SPEC-003, FEAT-07.SPEC-005 | Deposit Payment screen collects the card; the amount is computed and locked by SPEC-003; the charge itself is authorized and captured by SPEC-005 | Phase 2 (Explicit) |
| See a clear on-screen and confirmed record that the deposit succeeded | FEAT-07.SPEC-001, FEAT-07.SPEC-002 | The Deposit Payment screen shows the success state immediately, driven by SPEC-002's atomic confirm; the full on-screen confirmation display itself is FEAT-05's screen, reached on handoff | Phase 2 (Explicit) |
| Have a failed or declined payment explained clearly, with the slot held briefly to retry | FEAT-07.SPEC-001, FEAT-07.SPEC-005 | The screen surfaces the plain-language decline reason SPEC-005 translates from the processor and offers retry without re-entering other booking details; the slot hold itself is owned by FEAT-03 (XBR-02) | Phase 2 (Explicit) / Phase 6 (Failure Analysis) |
| Deposit lands directly in the Pro's own payout account (FEAT-28), with the platform taking no cut; the only deduction is the payment processor's own card fee, shown to the Pro | FEAT-07.SPEC-005 | Integration spec routes the captured amount to the Pro's connected payout account (XBR-07) and records the processor_fee field on the Deposit Transaction; the money-list display itself belongs to FEAT-28 | Phase 2 (Explicit) / Phase 4 (External Dependencies lens) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-07.SPEC-002 | Deposit Capture & Booking Confirmation | Phase 4 (Trigger-Response) | The Happy Path ("the deposit is authorized and captured; the booking flips from pending to confirmed instantly") is a cross-entity side-effect -- a card charge succeeding must atomically write a Deposit Transaction and transition a Booking -- too consequential to leave inline in the screen spec |
| FEAT-07.SPEC-003 | Deposit Amount & Eligibility Rules | Phase 5 (Rule Discovery) | The Validation & Limits field names five distinct, interacting conditions (exact computation from the service rule, currency, client cannot alter it, one charge per booking, payout account must be Active) -- past the inline-validation threshold and shared across the screen and the capture automation |
| FEAT-07.SPEC-004 | Payment Outcome Consistency & Idempotency | Phase 6 (Failure Analysis) | The Alternate flow ("payment succeeds but the confirmation step fails to load... never double-charged and never left unsure") and ASMP-26's correctness bar describe cross-cutting reliability behavior spanning the screen, the capture automation, and the integration -- not a single screen's concern, so it is elevated to its own rule spec |

## Entity-Lifecycle Coverage Matrix

**Entity: Deposit Transaction**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-07.SPEC-002 | On a successful charge (SPEC-005), the automation writes amount, currency, status: Captured, and processor_fee | Exactly one Deposit Transaction per Booking, enforced by SPEC-003/SPEC-004 |
| Read (single) | FEAT-07.SPEC-002, FEAT-07.SPEC-004 | Read internally for the one-charge-per-booking idempotency check before any new attempt is authorized | Display of transaction status to the Pro or Client is owned by FEAT-16, FEAT-28, and FEAT-05/FEAT-30 -- not a screen this feature produces |
| Read (list) | N/A | This feature produces no list view of Deposit Transactions | The money list is FEAT-28's screen; the activity record is FEAT-16's |
| Update | N/A | This feature only creates the initial Authorized/Captured record | Later transitions (Refunded, Forfeited, Disputed) are owned by FEAT-09 (automatic refund/forfeit), FEAT-11 (forfeit/undo), and FEAT-30 (Pro refund) per the dependency map's Deposit Transaction lifecycle -- explicit cross-feature ownership, not a gap |
| Delete/Archive | N/A | No delete/archive path exists for a financial record | Hard delete never occurs; per SC-22 the record is retained for the life of the account and de-identified (never removed) after client deletion or account closure -- recorded as an explicit non-goal below |
| State Transition | FEAT-07.SPEC-002 | Created directly into Authorized -> Captured as one atomic step on charge success | All later states (Refunded, Refund in Progress, Forfeited, Disputed) are owned by FEAT-09/FEAT-11/FEAT-30, not this feature |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Booking | FEAT-07.SPEC-001, FEAT-07.SPEC-002, FEAT-07.SPEC-003 | The payment screen reads the in-progress Booking (service, price, time) to display what is owed; the capture automation updates it (see below); eligibility rules confirm it is still Pending Payment before a charge is attempted |
| Service | FEAT-07.SPEC-003 | Deposit amount is computed once from the service's deposit_rule (fixed amount or percentage) and its price, in the Pro's account currency |
| Payout Account | FEAT-07.SPEC-003, FEAT-07.SPEC-005 | Eligibility rules block any charge unless the Pro's payout account status is Active (XBR-06); the integration routes the captured funds to it |
| Pro Account | FEAT-07.SPEC-003 | Currency is read from the Pro Account and becomes locked at the moment of this feature's first successful deposit (XBR-25) |

**Booking (narrow update, not fully managed by this feature):**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Update | FEAT-07.SPEC-002 | State transition Pending Payment -> Confirmed, written the instant the deposit is captured | This is the only Booking field this feature ever writes; all other Booking fields and other state transitions (cancel, reschedule, no-show, complete) are owned by FEAT-05, FEAT-10, FEAT-11, FEAT-12, and FEAT-30 |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Client submits card details on the Deposit Payment screen | Validate the booking is still Pending Payment and the amount matches the locked deposit computation, then request authorization and capture from the payment-processing capability | Standalone Logic/Rule, then Standalone Integration | FEAT-07.SPEC-003, FEAT-07.SPEC-005 |
| Payment-processing capability reports a successful capture | Create the Deposit Transaction record; atomically transition the Booking from Pending Payment to Confirmed | Standalone Automation | FEAT-07.SPEC-002 |
| Payment-processing capability reports a decline | Show the plain-language decline reason on the payment screen; Booking remains Pending Payment; the previously-held slot is left held for its short window | Inline in triggering screen (decline display), cross-feature (slot hold owned by FEAT-03 per XBR-02) | FEAT-07.SPEC-001 |
| Client retries with a different card after a decline | Re-attempt authorization/capture without requiring re-entry of service, time, name, phone, or policy acknowledgment | Inline in triggering screen | FEAT-07.SPEC-001 |
| Connection drops or the confirmation step fails to load mid- or post-payment | Treat as a safe-retry failure if the charge did not complete; if the charge did complete, the Booking's Confirmed state is the source of truth and is shown on next page load or via the confirmation text -- never re-charged | Standalone Logic/Rule | FEAT-07.SPEC-004 |
| Slot hold expires before the client retries a declined payment | Client is returned to the live slot list with a plain "hold expired" message; never charged | Inline in triggering screen, cross-feature (hold expiry and slot release owned by FEAT-03 per XBR-02) | FEAT-07.SPEC-001 |
| Deposit is captured successfully | Booking confirmation and the confirmation message (studio address, balance due, cancellation cut-off, manage link, calendar option) are handed to Automated Booking Messaging | Cross-feature -- Notification spec owned by FEAT-08 | FEAT-08 responsibility |
| Deposit is captured successfully | Captured funds are routed to the Pro's payout account with zero platform fee; the processor's own card fee is recorded on the transaction | Standalone Integration | FEAT-07.SPEC-005 |

The feature's Communications field is explicit that a successful deposit "feeds the confirmation message in Automated Booking Messaging (FEAT-08)" and that "a payment failure shows in-flow only and sends no separate message." Both dispositions are therefore already resolved: the success case is a cross-feature trigger into FEAT-08's own Notification spec (not duplicated here), and the failure case is an in-flow screen state with no delivery rules (per Phase 4's inline-communication exception), never a standalone message. `notification_count: 0` is intentional, not an omission.

## Shared Context

**Shared Entities:**
- Deposit Transaction -- created exclusively by SPEC-002 on successful capture; read internally by SPEC-002/SPEC-004 for the one-charge-per-booking guarantee. Fields: amount, currency, status (Authorized | Captured, at this feature's stage), processor_fee, outcome_reason/timestamps.
- Booking (narrow slice) -- read by SPEC-001/SPEC-002/SPEC-003 for its service, price, time, and current state; updated only by SPEC-002, and only for the single Pending Payment -> Confirmed transition.

**Shared UI Patterns:**
- Single payment surface -- SPEC-001 is the one screen for entry, processing, decline/retry, and success hand-off; the Recovering-from-a-Declined-Deposit journey confirms the client never leaves this screen or re-enters unrelated booking details between a decline and a successful retry.
- Amount display -- the deposit amount shown on SPEC-001 is always the value SPEC-003 computed and locked; SPEC-001 never performs its own computation or lets the client edit it.

**Shared Validation:**
- SPEC-003 defines all eligibility and amount-computation rules (currency, rounding, one-charge-per-booking, active-payout-account precondition). SPEC-001 and SPEC-002 both reference it rather than duplicating the checks.
- SPEC-004 defines the never-double-charge / never-ambiguous-outcome guarantee that SPEC-001, SPEC-002, and SPEC-005 all rely on for their respective failure-mode behavior.

## Internal Dependency Map

```
SPEC-001 (Deposit Payment) -> [Client submits card] -> SPEC-003 (Deposit Amount & Eligibility Rules) -> [eligible] -> SPEC-005 (Card Deposit Charge & Payout Routing)
SPEC-005 (Card Deposit Charge & Payout Routing) -> [processor reports capture] -> SPEC-002 (Deposit Capture & Booking Confirmation) -> [Booking Confirmed] -> SPEC-001 (success state, then handoff to FEAT-05)
SPEC-005 (Card Deposit Charge & Payout Routing) -> [processor reports decline] -> SPEC-001 (decline state, offers retry)
SPEC-001 (Deposit Payment) -> [Client retries with new card] -> SPEC-003 (Deposit Amount & Eligibility Rules) -> SPEC-005 (Card Deposit Charge & Payout Routing)
SPEC-002 (Deposit Capture & Booking Confirmation) -> [governed by] -> SPEC-004 (Payment Outcome Consistency & Idempotency)
SPEC-001 (Deposit Payment) -> [connection drop or confirmation load failure] -> SPEC-004 (Payment Outcome Consistency & Idempotency) -> [resolved state] -> SPEC-001
```

**Default Entry:** SPEC-001 (Deposit Payment) -- reached only from an in-progress booking's policy-acknowledgment step; this feature has no standalone entry point of its own (States field: "payment is always tied to an in-progress booking, never a standalone screen").

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-07.SPEC-001 | Inbound | FEAT-05 (Public Booking Page & Booking Flow) | Client arrives at the deposit payment step after ticking the policy agreement | Client ticks policy agreement on the booking flow |
| FEAT-07.SPEC-001 | Outbound | FEAT-05 (Public Booking Page & Booking Flow) | Client is handed to the on-screen booking confirmation | Deposit succeeds |
| FEAT-07.SPEC-001 | Outbound | FEAT-03 (Real-Time Slot Availability Engine) | Client is returned to the live slot list | Slot hold expires before a declined payment is retried |
| FEAT-07.SPEC-003 | Inbound | FEAT-01 (Service & Pricing Management) | Deposit rule (fixed amount or percentage) feeds the amount computation | Client reaches the payment step for a given service |
| FEAT-07.SPEC-003 | Inbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Eligibility check reads whether the Pro's payout account is Active | Client attempts to pay a deposit (XBR-06) |
| FEAT-07.SPEC-005 | Outbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Captured deposit and processor fee are routed to, and later shown in, the Pro's money list | Deposit is captured |
| FEAT-07.SPEC-002 | Outbound | FEAT-08 (Automated Booking Messaging) | Confirmed booking feeds the client's confirmation message (studio address, balance due, cancellation cut-off, manage link, calendar option) | Booking transitions to Confirmed |
| FEAT-07.SPEC-002 | Outbound | FEAT-16 (Booking & Payment Activity Record) | Deposit attempt, success, and failure events are written to the append-only activity record | deposit_payment_attempted / _succeeded / _failed signals fire |
| FEAT-07.SPEC-002 | Outbound | FEAT-30 (Pro Booking Management) | Paid/unpaid status becomes visible on the Pro's booking management surface | Booking transitions to Confirmed |
| FEAT-07.SPEC-002 | Outbound | FEAT-11 (No-Show Marking & Deposit Forfeiture) | The Deposit Transaction this feature creates becomes the record FEAT-11 later forfeits on a no-show | Booking is later marked no-show |
| FEAT-07.SPEC-002 | Outbound | FEAT-09 (Cancellation & No-Show Policy Engine) | The Deposit Transaction this feature creates becomes the record FEAT-09 later refunds automatically | Client cancels within/outside the policy window |

## Non-Functional Notes

**Data volumes / growth:** Each Booking produces at most one Deposit Transaction from this feature (SC-19's 20-40 bookings/week per pro at scale), so volume tracks Booking volume directly and stays well within a solo pro's scale; no separate growth pattern applies to this feature specifically (Data Notes field; SC-19).

**Responsiveness:** The deposit payment step must complete inside the product's under-one-minute full-booking benchmark (ASMP-21); the screen's Loading state ("processing payment, do not close this page") exists precisely because authorization/capture is the one step in the booking flow that cannot be instant, so it is called out explicitly rather than left to a generic spinner (States field).

**Data sensitivity / privacy:** This feature captures only deposit amount and payment outcome -- never card data, which the payment-processing capability owns exclusively (Data Notes field; SC-11). The resulting Deposit Transaction is a financial record tied to an identifiable booking and client, visible only to the Pro (Full, on their own bookings) and, view-only, to Platform Operator (Support) for transaction status -- never card data (Access field; Access Matrix, Booking & Payment / Payouts rows).

**Compliance flags:** N/A -- no compliance regime is named for this feature specifically beyond the hard card-data boundary in SC-11, which is precisely what keeps PCI-scope obligations with the payment-processing capability (ASMP-31) rather than with the product's own code.

## Non-Goals

- **Handling or storing card data within the product itself** -- Excluded per SC-11: "card data is never stored or handled by the founder's code; the payment processor owns it." This feature only ever holds the resulting amount and outcome, never the card number.
- **Card-on-file cancellation fees charged after booking** -- Excluded per SC-13: the product protects the Pro with a deposit paid up front; charging a client's card again later would require retaining a card on file, which SC-13 rules out as the disputed-charge pattern this product is designed to avoid.
- **Card-reader hardware or in-person payment taking** -- Excluded per SC-16: the balance left after the deposit is settled in person between the Pro and client, outside the platform, by whatever means the Pro already uses; this feature has no role in that settlement.
- **Partial refunds or tiered cancellation schedules on the deposit** -- Excluded per SC-18: the cancellation rule this feature's deposit feeds into is binary (refunded outside the window, kept inside it or on a no-show); this feature never computes or offers a partial-percentage outcome.
- **Chairtime adjudicating payment disputes** -- Excluded per SC-17: a client who disagrees with a captured deposit contests it with their card issuer through the payment processor's own dispute process; this feature records the outcome (via FEAT-16) but never rules on who is right.
- **In-App Balance Payment and Tipping at Checkout** -- Adjacency exclusion: both are named, explicitly deferred capabilities (relevant deferral notes: In-App Balance Payment targeted for v1, Tipping at Checkout targeted for Later) that extend what a client can pay in-app beyond the deposit; this feature's scope is the deposit only, and the in-person balance flow works today without either.
