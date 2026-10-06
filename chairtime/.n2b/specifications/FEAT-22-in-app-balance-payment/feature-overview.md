---
document_type: feature-overview
feature_number: FEAT-22
feature_name: In-App Balance Payment
feature_slug: in-app-balance-payment
priority_tier: Nice-to-Have
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

# Feature Breakdown Brief: In-App Balance Payment

## Summary

**Feature:** In-App Balance Payment
**ID:** FEAT-22
**Description:** A client can optionally pay the remaining balance (beyond the deposit) through the app before or at the appointment, instead of paying the Pro directly in person.
**Priority:** Nice-to-Have
**Phase:** v1
**Type:** User-Facing
**Rationale:** BRIEF.md's Open Questions asks directly whether the balance "should be payable in the app, or stay in person." As Visionary judgment: the brief's default flow ("the balance is due at the appointment") works fine without this, so it is not required for MVP, but it is a natural, low-risk enhancement once deposit payment (FEAT-07) is proven. Phased to v1. This answers BRIEF.md's balance open question for MVP: the balance stays in person at MVP and becomes optionally payable in-app at v1.

**Key Capabilities:**
- Pay the remaining balance in-app at any point before or at the appointment
- See a running record of deposit paid vs. balance remaining
- Balance goes straight to the Pro's payout account (FEAT-28) with no platform cut, and is refunded in full if the appointment is later cancelled by either party -- a balance is never subject to forfeiture

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-22.SPEC-001 | Balance Payment | Screen | The Client | Client views deposit-paid-vs-balance-remaining on their confirmed booking and pays the balance in-app, seeing processing, decline, and success states |
| FEAT-22.SPEC-002 | Balance Capture & Booking Status Update | Automation | The Client, The Pro | On a successful charge, creates the Balance Payment record and updates the Booking's balance-due status to "fully paid," so the Pro's dashboard reflects it |
| FEAT-22.SPEC-003 | Balance Amount & Eligibility Rules | Logic/Rule | The Client, The Pro | Governs how the balance amount is computed once (price minus deposit) and cannot be altered by the client, and the preconditions that must hold before any charge is attempted |
| FEAT-22.SPEC-004 | Balance Payment Outcome Consistency & Cancellation Contention | Logic/Rule | The Client, The Pro | Guarantees every balance payment attempt ends in exactly one clean outcome -- never a partial or ambiguous "balance due" state -- and resolves the case where a Pro cancellation and a client's balance payment race each other, per XBR-23 |
| FEAT-22.SPEC-005 | Balance Charge, Payout Routing & Refund | Integration | The Client, The Pro | Authorizes and captures the client's card charge through the payment-processing capability, routes the captured balance to the Pro's payout account with zero platform fee, and executes the outbound refund call when a paid balance is refunded in full |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Pay the remaining balance in-app at any point before or at the appointment | FEAT-22.SPEC-001, FEAT-22.SPEC-003, FEAT-22.SPEC-005 | Balance Payment screen collects the charge; the amount is computed and locked by SPEC-003; the charge itself is authorized and captured by SPEC-005 | Phase 2 (Explicit) |
| See a running record of deposit paid vs. balance remaining | FEAT-22.SPEC-001 | The Balance Payment screen displays the deposit already paid (read from Deposit Transaction) against the computed balance remaining before the client pays | Phase 2 (Explicit) |
| Balance goes straight to the Pro's payout account (FEAT-28) with no platform cut, and is refunded in full if the appointment is later cancelled by either party -- a balance is never subject to forfeiture | FEAT-22.SPEC-003, FEAT-22.SPEC-004, FEAT-22.SPEC-005 | SPEC-003/SPEC-004 encode the zero-platform-fee and never-forfeited/full-refund rule (XBR-23), which this feature owns as authority; SPEC-005 routes the captured charge to the Pro's payout account and executes the refund call back to the processor when a cancellation triggers it | Phase 2 (Explicit) / Phase 4 (External Dependencies lens) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-22.SPEC-002 | Balance Capture & Booking Status Update | Phase 4 (Trigger-Response) | The Happy Path ("the Pro's dashboard reflects 'fully paid' instead of 'balance due'") is a cross-entity side-effect -- a card charge succeeding must write a Balance Payment record and transition the Booking's balance status -- too consequential to leave inline in the screen spec |
| FEAT-22.SPEC-003 | Balance Amount & Eligibility Rules | Phase 5 (Rule Discovery) | The Validation & Limits field ("the balance amount is fixed by the service price minus the deposit already paid and cannot be altered by the client") is a derivation rule read by both the screen and the capture automation -- past the inline-validation threshold once the derivation source (Deposit Transaction) and the zero-fee standing rule (XBR-07) are folded in |
| FEAT-22.SPEC-004 | Balance Payment Outcome Consistency & Cancellation Contention | Phase 6 (Failure Analysis) | The Alternate flow ("an in-app balance payment fails; the booking remains marked 'balance due' ... with no partial or ambiguous state") together with the dependency map's Balance Payment Contention line (a committed cancellation blocks a payment; a committed payment is refunded in full by a later cancellation, per XBR-23) describes cross-cutting reliability behavior spanning the screen, the capture automation, and the cancellation path owned by FEAT-30/FEAT-09 -- elevated to its own rule spec rather than left inline |
| FEAT-22.SPEC-005 | Balance Charge, Payout Routing & Refund | Phase 4 (External Dependencies lens) | The Dependencies section of assumptions-constraints.md (ASMP-31) names the payment-processing capability this feature relies on for the charge and its payout routing; the dependency map's External Touchpoints section explicitly expects this feature to inventory how the balance charge is routed and how its refund reaches the processor |

## Entity-Lifecycle Coverage Matrix

**Entity: Balance Payment**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-22.SPEC-002 | On a successful charge (SPEC-005), the automation writes amount and status: Succeeded | Exactly zero or one Balance Payment per Booking (dependency map: Booking "has ... zero or one Balance Payment") |
| Read (single) | FEAT-22.SPEC-001, FEAT-22.SPEC-003 | The Balance Payment screen displays the client's own record and its status; SPEC-003 reads it internally to confirm no successful payment already exists before allowing a new attempt | Display of the resulting status to the Pro is owned by FEAT-12 (dashboard) and FEAT-28 (money list) -- not a screen this feature produces |
| Read (list) | N/A | This feature produces no list view of Balance Payments; at most one exists per Booking | -- |
| Update | N/A -- explicit cross-feature ownership | tip amount is added by FEAT-23 (Later); refund state is set by FEAT-30 on cancellation, per the dependency map's Balance Payment lifecycle line | Not a gap: the dependency map names these as this entity's only post-creation writers, neither of which is this feature |
| Delete/Archive | N/A -- explicit non-goal | No delete/archive path exists for a financial record; per SC-22 the record is retained for the life of the account and de-identified (never removed) after client deletion or account closure | -- |
| State Transition | FEAT-22.SPEC-002, FEAT-22.SPEC-004 | Created directly into Attempted -> Succeeded/Failed as one clean outcome (SPEC-002, governed by SPEC-004's consistency guarantee) | The later Refunded transition is owned by FEAT-30 per XBR-23, not this feature |

**Booking (narrow update, not fully managed by this feature):**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Update | FEAT-22.SPEC-002 | Updates only the balance-due status (derived: price - deposit - any in-app balance payment) the instant the balance is captured | This is the only Booking field this feature ever writes; all other Booking fields and state transitions (cancel, reschedule, no-show, complete) are owned by FEAT-05, FEAT-10, FEAT-11, FEAT-12, FEAT-21, and FEAT-30 |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Deposit Transaction | FEAT-22.SPEC-001, FEAT-22.SPEC-003 | The screen's running record and the amount computation both read the already-paid deposit amount from the Deposit Transaction created by FEAT-07 |
| Booking | FEAT-22.SPEC-001, FEAT-22.SPEC-002, FEAT-22.SPEC-003, FEAT-22.SPEC-004 | The payment screen reads the confirmed Booking (service, price, current balance-due) to display what is owed; eligibility and contention rules confirm the Booking is still in a payable state before a charge is attempted |
| Payout Account | FEAT-22.SPEC-005 | The integration routes the captured balance to, and later issues the refund from, the Pro's connected payout account |
| Service | FEAT-22.SPEC-003 | The balance amount is derived from the service price agreed at booking, minus the deposit |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Client opens their booking and taps to pay the balance | Load the current deposit-paid and balance-remaining figures and display them before any charge is requested | Inline in triggering screen | FEAT-22.SPEC-001 |
| Client submits payment for the balance | Validate the booking is still payable and the amount matches the locked balance computation, then request authorization and capture from the payment-processing capability | Standalone Logic/Rule, then Standalone Integration | FEAT-22.SPEC-003, FEAT-22.SPEC-005 |
| Payment-processing capability reports a successful capture | Create the Balance Payment record (Succeeded); update the Booking's balance-due status to "fully paid" | Standalone Automation | FEAT-22.SPEC-002 |
| Payment-processing capability reports a decline | Show the specific decline message on the payment screen, matching the deposit payment feature's pattern; Booking remains marked "balance due" exactly as if the attempt never happened, with no partial or ambiguous state | Inline in triggering screen (decline display), Standalone Logic/Rule (no-partial-state guarantee) | FEAT-22.SPEC-001, FEAT-22.SPEC-004 |
| Pro commits a cancellation while a balance payment is in flight | The cancellation, if committed first, blocks the payment; the client's attempt is refused with the current (cancelled) state shown | Standalone Logic/Rule | FEAT-22.SPEC-004 |
| Client's balance payment is captured first, and the booking is cancelled afterward by either party | The paid balance is refunded in full alongside any deposit (XBR-23); the refund is triggered through Pro Booking Management (FEAT-30) and executed back to the processor | Standalone Logic/Rule (rule ownership), then Standalone Integration (processor call), Cross-feature (refund triggering owned by FEAT-30/FEAT-09) | FEAT-22.SPEC-004, FEAT-22.SPEC-005 |
| Balance is captured successfully | Captured funds are routed to the Pro's payout account with zero platform fee; the money list (FEAT-28) reflects the new entry | Standalone Integration, Cross-feature (display owned by FEAT-28) | FEAT-22.SPEC-005 |
| Balance is captured successfully | Booking's balance-due status flips, and the Pro's Daily Schedule Dashboard shows "fully paid" instead of "balance due" | Cross-feature -- display owned by FEAT-12, driven by the status this feature writes | FEAT-22.SPEC-002 |
| Connectivity is lost while the client is on the Balance Payment screen | Payment action is unavailable and the screen says so plainly; requires a live connection, consistent with all payment actions | Inline in triggering screen | FEAT-22.SPEC-001 |

The feature's Communications field names exactly one message -- "a payment confirmation on successful balance payment" -- with no channel, audience, or delivery rule beyond the confirmation itself, and neither the Interactions field nor the dependency map names another feature (such as FEAT-08's automated messaging) as this message's owner. Per Phase 4's inline-communication exception ("a same-screen confirmation with no delivery rules"), this stays inline in FEAT-22.SPEC-001 as the screen's success state. `notification_count: 0` is therefore intentional, not an omission.

## Shared Context

**Shared Entities:**
- Balance Payment -- created exclusively by SPEC-002 on successful capture; read by SPEC-001 (display) and internally by SPEC-003/SPEC-004 (eligibility and consistency checks). Fields: amount (price minus deposit, not client-alterable), tip (optional, added later by FEAT-23), state (Attempted | Succeeded | Failed | Refunded).
- Deposit Transaction (read-only here) -- read by SPEC-001 and SPEC-003 to derive the already-paid amount that anchors the running record and the balance computation. Fields relevant here: amount/currency, status.
- Booking (narrow slice) -- read by SPEC-001/SPEC-002/SPEC-003/SPEC-004 for its price, deposit, and current balance-due state; updated only by SPEC-002, and only for the balance-due status.

**Shared UI Patterns:**
- Single payment surface -- SPEC-001 is the one screen for the running record, entry, processing, decline, and success states, mirroring the single-surface pattern the deposit payment feature (FEAT-07) uses for its own payment step; the client never leaves this screen between a decline and a successful retry.
- Amount display -- the balance amount shown on SPEC-001 is always the value SPEC-003 computed and locked; SPEC-001 never performs its own computation or lets the client edit it.
- Decline messaging -- SPEC-001's error state reuses the deposit payment feature's decline-message pattern (per the States field), so the two payment surfaces read consistently to a client who has seen both.

**Shared Validation:**
- SPEC-003 defines the balance-amount computation and eligibility preconditions once. SPEC-001 and SPEC-002 both reference it rather than duplicating the computation.
- SPEC-004 defines the never-double-charge / never-partial-state guarantee and the cancellation-contention resolution (XBR-23) that SPEC-001, SPEC-002, and SPEC-005 all rely on for their respective failure-mode and refund behavior.

## Internal Dependency Map

```
SPEC-001 (Balance Payment) -> [Client submits payment] -> SPEC-003 (Balance Amount & Eligibility Rules) -> [eligible] -> SPEC-005 (Balance Charge, Payout Routing & Refund)
SPEC-005 (Balance Charge, Payout Routing & Refund) -> [processor reports capture] -> SPEC-002 (Balance Capture & Booking Status Update) -> [Booking marked fully paid] -> SPEC-001 (success state)
SPEC-005 (Balance Charge, Payout Routing & Refund) -> [processor reports decline] -> SPEC-001 (decline state, offers retry)
SPEC-001 (Balance Payment) -> [Client retries after a decline] -> SPEC-003 (Balance Amount & Eligibility Rules) -> SPEC-005 (Balance Charge, Payout Routing & Refund)
SPEC-002 (Balance Capture & Booking Status Update) -> [governed by] -> SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention)
SPEC-001 (Balance Payment) -> [payment vs. concurrent cancellation] -> SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention) -> [resolved state] -> SPEC-001
SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention) -> [cancellation after a paid balance] -> SPEC-005 (Balance Charge, Payout Routing & Refund) -> [refund executed]
```

**Default Entry:** SPEC-001 (Balance Payment) -- reached only from an existing confirmed booking with a balance due; this feature has no standalone entry point of its own (Access field: "only appears against an existing confirmed booking with a balance due").

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-22.SPEC-001 | Inbound | FEAT-06 (Client Booking Identity) | Client taps "pay balance" from their booking view, reached through their access link | Client opens their booking |
| FEAT-22.SPEC-003 | Inbound | FEAT-07 (Deposit Payment at Booking) | Balance amount is derived from the deposit already captured by FEAT-07 | Client opens the Balance Payment screen |
| FEAT-22.SPEC-005 | Outbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Captured balance and its later refund are routed to, and shown in, the Pro's connected payout account and money list | Balance is captured or refunded |
| FEAT-22.SPEC-002 | Outbound | FEAT-12 (Pro Daily Schedule Dashboard) | Booking's balance-due status update makes the dashboard show "fully paid" instead of "balance due" | Balance is captured |
| FEAT-22.SPEC-004 | Outbound | FEAT-30 (Pro Booking Management) | A paid balance is refunded through the Pro's cancellation/refund flow (XBR-23) | Pro cancels or issues a refund on a booking with a paid balance |
| FEAT-22.SPEC-004 | Inbound | FEAT-09 (Cancellation & No-Show Policy Engine) | A client or automatic cancellation that reaches a booking with a paid balance triggers the same full-refund guarantee (XBR-23) | Cancellation is committed on a booking with a paid balance |
| FEAT-22.SPEC-003 / SPEC-004 | Outbound | FEAT-23 (Tipping at Checkout, Later) | Tip amount is added onto the Balance Payment record by FEAT-23 once it ships; this feature's amount computation and record shape are the dependency FEAT-23 builds on | Client adds a tip at balance checkout (once FEAT-23 exists) |
| FEAT-22.SPEC-001 | Inbound | FEAT-29 (Pro Sign-In & Account Lifecycle) | A client without a valid access link for the booking sees a "request a new link" prompt rather than the payment screen | Client reaches the booking link without valid access |

## Non-Functional Notes

**Data volumes / growth:** At most one Balance Payment row per Booking, and only for the subset of bookings where the client chooses in-app payment over the in-person default; volume tracks a fraction of Booking volume (a few hundred pros, ~20-40 bookings/week each, per assumptions-constraints.md's scale assumptions), well within a solo Pro's scale (Data Notes field).

**Responsiveness:** The balance charge should feel like the same fast card transaction as the deposit payment step (States field: "standard payment-processing indicator"); no separate benchmark is named for this feature, so it inherits the deposit payment feature's practice of an explicit in-place processing indicator rather than a generic spinner (ASMP-27).

**Data sensitivity / privacy:** This feature captures only the balance payment amount and outcome -- never card data, which the payment-processing capability owns exclusively (Data Notes field; SC-11). The resulting Balance Payment is a financial record tied to an identifiable booking and client, visible only to the Client (Own-only, their own balance) and the Pro (Full, on their own bookings), and view-only to Platform Operator (Support) for status -- never card data (Access field; Access Matrix, Booking & Payment row).

**Compliance flags:** N/A -- no compliance regime is named for this feature specifically beyond the hard card-data boundary in SC-11, which keeps PCI-scope obligations with the payment-processing capability (ASMP-31) rather than with the product's own code.

## Non-Goals

- **Handling or storing card data within the product itself** -- Excluded per SC-11: "card data is never stored or handled by the founder's code; the payment processor owns it." This feature only ever holds the resulting amount and outcome, never the card number; each balance payment is a fresh charge (consistent with XBR-05's "each deposit is paid fresh" principle applied to the balance).
- **Card-reader hardware or in-person payment taking** -- Excluded per SC-16: at MVP the balance is settled in person outside the platform; this feature is precisely the v1 in-app alternative to that path and never touches in-person hardware or cash/card-reader handling.
- **Card-on-file cancellation fees charged after booking** -- Excluded per SC-13: this feature charges a fresh card at the moment the client chooses to pay, never retains a card on file, and never charges again later without a new client-initiated attempt.
- **Partial refunds or tiered cancellation schedules on the balance** -- Excluded per SC-18 and XBR-23: a paid balance is refunded in full or not at all; this feature never computes or offers a partial-percentage outcome.
- **Support acting on a client's or Pro's balance payment** -- Excluded per SC-05: Platform Operator (Support) has view-only access to balance status and never issues, edits, or refunds a balance payment on either party's behalf; refunds are issued by the Pro through FEAT-30.
- **Making the balance mandatory, or removing the in-person default** -- Excluded per the feature's own Alternate flow and BRIEF.md's Open Questions resolution: in-app balance payment is purely additive; paying in person remains the default, unaffected experience for every client who does not choose this feature.
- **Tipping at checkout** -- Adjacency exclusion: Tipping at Checkout (FEAT-23) is a separate, Later-phase feature that extends the Balance Payment record with an optional tip; this feature's scope is the balance amount only, and it functions completely without FEAT-23 existing.
