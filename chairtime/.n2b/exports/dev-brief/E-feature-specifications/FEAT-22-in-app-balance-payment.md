# FEAT-22 — In-App Balance Payment

This chapter covers In-App Balance Payment (FEAT-22), a Nice-to-Have-tier feature. It carries 5 specifications carrying 79 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-22.SPEC-001 | Balance Payment | screen | 18 |
| FEAT-22.SPEC-002 | Balance Capture & Booking Status Update | automation | 12 |
| FEAT-22.SPEC-003 | Balance Amount & Eligibility Rules | logic-rule | 15 |
| FEAT-22.SPEC-004 | Balance Payment Outcome Consistency & Cancellation Contention | logic-rule | 16 |
| FEAT-22.SPEC-005 | Balance Charge, Payout Routing & Refund | integration | 18 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Balance Payment

## Overview

**Name:** Balance Payment
**ID:** FEAT-22.SPEC-001
**Type:** Screen
**Purpose:** Riley views her deposit-paid-vs-balance-remaining running record on her own confirmed booking and optionally pays the balance in-app, seeing processing, decline, and success states.
**Parent Feature:** FEAT-22 -- In-App Balance Payment

## Scope and Non-Goals

**In Scope:**
- Displaying the deposit already paid and the balance remaining on Riley's own confirmed booking
- Collecting card details through the payment-processing capability's own entry element and submitting them for authorization and capture of the balance
- Processing, decline/retry, and success states for a single balance payment attempt and any immediate retries
- Reflecting a cancellation that arrives while Riley is on this screen, per the contention rule this feature owns

**Non-Goals:**
- Computing the balance amount -- governed exclusively by FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules); this screen only displays the value SPEC-003 locked
- Guaranteeing the charge is never duplicated, resolving the outcome across a dropped connection, and resolving a race with a Pro cancellation -- governed by FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention); this screen only reflects the outcome that spec resolves
- Displaying the rest of the booking's details (service, appointment time, studio address, cancellation policy) -- owned by FEAT-06.SPEC-004 (Booking Detail via Manage Link), from which Riley reaches this screen; this screen shows only the payment-specific running record
- Paying the balance in person, or any card-reader or in-person handling of the balance -- excluded per SC-16: the in-person path remains the unaffected default; this screen is purely the optional in-app alternative

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Riley taps "Pay Balance" on her own confirmed booking | The Booking reference (service, price_agreed, deposit_amount via the Deposit Transaction, current balance-due state) |

This is the feature's only entry point (Access field, product-features.md: "only appears against an existing confirmed booking with a balance due"). There is no standalone or directly-linked way to reach this screen -- it is always reached from Riley's own existing booking, never from a fresh booking flow.

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen, scoped to her own booking with a balance due only | Enter card details and submit a balance payment for her own booking only | -- |
| The Pro (Talia) | No | No | N/A -- this screen belongs to the client-facing "my bookings" flow and never appears on any of the Pro's own surfaces (FEAT-12, FEAT-30). The Pro's own view of the resulting paid/unpaid status lives on FEAT-12 (Pro Daily Schedule Dashboard) and FEAT-30 (Pro Booking Management), never here, per the Access Matrix's Booking & Payment row (Full for the Pro, but on those surfaces, not this one). |
| Platform Operator (Support) | No | No | N/A -- Support's view-only access to balance status is on FEAT-16 (Booking & Payment Activity Record) and FEAT-19 (Platform Support Read-Only Access), and never to card data (Access Matrix, Payouts row: "View: status and money list only"). Support never opens this in-flow payment screen. |
| Unauthenticated / no valid access link | No | No | Per FEAT-29 (Pro Sign-In & Account Lifecycle) and FEAT-06.SPEC-004's and FEAT-06's access-link scoping (XBR-18), a visitor without a valid access link for this specific booking sees "Request a new link" rather than this screen -- the same experience as any other attempt to reach a client-facing booking view without valid access. |
| Expired access link | No | No | The access link governing Riley's "my bookings" session has its own expiry (XBR-18); once it expires, she is returned to the "Request a new link" prompt exactly as an unauthenticated visitor would be. Any card details she had begun entering are discarded -- nothing is preserved across an expired link, since re-entry always starts this screen fresh. |

This product has no sign-in concept for the Client (BRIEF.md, Target Users & Roles: clients "must not face a signup wall"); access to this screen is scoped entirely to Riley's valid access link for this specific booking (FEAT-06.SPEC-004, XBR-18), not to a login.

## Layout and Content

**Header:** A back arrow (returns Riley to FEAT-06.SPEC-004, Booking Detail via Manage Link) and the screen title "Pay your balance." Below the title, a one-line booking summary: service name, the Pro's display name, and the appointment date and time in the Pro's timezone (labeled, per XBR-25).

**Body, in order, top to bottom:**
- A running-record block showing, as two lines: "Deposit paid: {deposit_amount}" (read from the Deposit Transaction) above "Balance due: {balance amount locked by FEAT-22.SPEC-003}" as the primary figure. Both figures are read directly from existing records; this screen performs no computation of its own.
- The payment-processing capability's own card-detail entry element -- the fields Riley fills (card number, expiry, security code, and postal code where the capability requires it) are entered directly into that capability's element and never pass through the product's own code, per SC-11.
- A single "Pay {balance amount} balance" button, below the card entry element, spanning the width of the body.
- A status/error region directly below the button, empty in the default state, populated in the Processing, Error, and Cancelled states described below.

**Footer:** A single line of plain text: "Do not close this page while your payment is processing." -- shown persistently, not only during the Processing state, so Riley is prepared before she taps Pay.

### Responsive Behavior

- **Compact size class (phone, the product's primary usage per BRIEF.md's Scale & Non-Functional Expectations):** Single-column layout exactly as described above, full width, footer text pinned below the status region.
- **Medium size class and above:** The body content (running-record block, card element, Pay button, status region) is capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.
- **Booking summary line:** Wraps to two lines on the narrowest supported widths rather than truncating -- the service name and appointment time are never cut off.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Screen closes | Standard back transition |
| Card entry element | Type | Captures card details directly with the payment-processing capability | Element shows entered data per the capability's own input treatment | Standard input focus/validation state from the capability's element |
| Pay button | Tap | 1. Confirm the Booking is still in a payable state and no prior Balance Payment has already succeeded. 2. Request eligibility confirmation from FEAT-22.SPEC-003. 3. If eligible, submit the charge through FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund). | Button enters the Processing state (loading indicator, disabled) | Status region shows "Processing payment, do not close this page." |
| Pay button (while processing) | Tap | No action -- ignored while a request is already in flight (double-submit prevention, governed by FEAT-22.SPEC-004) | None | Button remains in Processing state |
| "Try a different card" (shown only in the Error state) | Tap | Clears the card entry element and re-enables the Pay button | Screen returns to the default entry state with the same locked balance amount shown | Card entry element is cleared and refocused |
| "Back to my bookings" (shown only in the Cancelled state) | Tap | Navigate to FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Screen closes; the booking's now-cancelled state is reflected there | The updated booking state is visible on FEAT-06.SPEC-004, which Riley returns to |

### Accessibility Notes

- **Focus order:** Back arrow -> booking summary (read-only, not a focus stop) -> running-record block (read-only, not a focus stop) -> card entry element's own internal focus order -> Pay button -> status/error region (receives focus only when it updates).
- **Dynamic announcements:** When the status region updates (Processing begins, a decline message appears, success is reached, or the Cancelled message appears), the new text is announced to assistive technology and the region is programmatically associated as the outcome of the Pay action.
- **Success feedback:** On success, focus moves to the success message.
- **Keyboard alternatives:** Every action on this screen (Pay, Try a different card, Back to my bookings, back arrow) is reachable by keyboard; there are no pointer-only gestures. The card entry element's own keyboard accessibility is owned by the payment-processing capability.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default (entry) | Running-record block and empty card entry element shown; Pay button enabled once the element reports valid input | Screen first opens from FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Riley taps Pay |
| Processing | Pay button shows a loading indicator and is disabled; status region reads "Processing payment, do not close this page." | Riley taps Pay and the charge request is sent | FEAT-22.SPEC-005 reports a result (success or decline), or the connection drops |
| Success | Status region shows "Balance received -- your booking is now fully paid." and the running-record block updates to show a zero balance | FEAT-22.SPEC-002 (Balance Capture & Booking Status Update) reports the Booking's balance-due status is now fully paid | Riley taps Back to return to "my bookings", where the fully-paid status is also reflected |
| Error (declined) | Status region shows the plain-language decline reason (translated by FEAT-22.SPEC-005) and a "Try a different card" action; card entry element is cleared | FEAT-22.SPEC-005 reports a decline | Riley taps "Try a different card," returning to Default with the same locked balance amount and no partial or ambiguous state on the booking (FEAT-22.SPEC-004) |
| Cancelled (contention resolved against the attempt) | Status region shows "This booking was just cancelled, so there's nothing to pay. Your card was not charged." with the "Back to my bookings" action; Pay button is disabled | FEAT-22.SPEC-004's contention rule determines a Pro cancellation committed before Riley's payment attempt (reject-with-refresh) | Riley taps "Back to my bookings" |
| Offline/Degraded | If connectivity is lost before Riley taps Pay, the Pay button is disabled with the message "You're offline -- reconnect to pay your balance." If connectivity drops mid-processing, this is treated per FEAT-22.SPEC-004: the status region shows "We couldn't confirm your payment. Check your connection; if you were charged, your booking will show as fully paid the moment you reconnect -- you will never be charged twice." with a "Try again" action | Connectivity is lost while this screen is open, at any point before or during a charge attempt | Connectivity is restored: the Default state resumes if no charge was in flight; if a charge was in flight, FEAT-22.SPEC-004's resolution determines whether Success or Error is shown |

## Validation Rules

Validation of the balance amount and the preconditions for attempting a charge (Booking still in a payable state, currency match, no existing succeeded Balance Payment, active payout account, no committed cancellation) is governed by FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules) and FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention). Card-detail format validation is owned entirely by the payment-processing capability's own entry element; this screen never validates card fields itself. This screen enforces no field-level rules of its own beyond disabling Pay until the card element reports it has valid input.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-06.SPEC-004 (Booking Detail via Manage Link) | FEAT-06 (Client Booking Identity) |
| Successful payment | FEAT-06.SPEC-004 (Booking Detail via Manage Link), reached only when Riley taps Back after Success | FEAT-06 (Client Booking Identity) |
| Pro cancellation resolved against the attempt | FEAT-06.SPEC-004 (Booking Detail via Manage Link) | FEAT-06 (Client Booking Identity) |

## Data Model

**Creates:** None directly -- the Balance Payment record is created by FEAT-22.SPEC-002 on a successful charge, not by this screen.
**Reads:** Booking -- service, start_time, price_agreed, balance-due state. Deposit Transaction -- amount (for the "Deposit paid" line). Pro Account -- display_name, timezone (for the booking summary line, per XBR-25).
**Updates:** None directly -- the Booking's balance-due status update is written by FEAT-22.SPEC-002, never by this screen.
**Deletes:** None.

## Business Rules

- The balance amount shown is always the value FEAT-22.SPEC-003 computed and locked; this screen never performs its own computation and never lets Riley edit it (per the Brief's Shared UI Patterns).
- Every payment attempt, and every retry, is subject to FEAT-22.SPEC-003's eligibility preconditions and FEAT-22.SPEC-004's contention check before a charge is requested.
- FEAT-22.SPEC-004 governs the guarantee that this screen never shows an ambiguous outcome: every attempt resolves to exactly one of Success, Error, or Cancelled, even across a dropped connection or a concurrent Pro cancellation.
- Retrying after a decline never requires Riley to re-enter or re-confirm anything about the booking -- only the card entry element is reset.
- In-app balance payment is purely additive: paying in person remains the unaffected default for any client who never opens this screen (per the feature's Alternate flow and BRIEF.md's Open Questions resolution).
- XBR-07: Chairtime's fee on the balance is always zero; the amount Riley pays goes to Talia's payout account with no platform cut.

## Edge Cases

- **Riley taps Pay twice in rapid succession** -- The second tap is ignored while the first request is in flight (Pay button disabled in the Processing state, per FEAT-22.SPEC-004's double-charge prevention).
- **Riley navigates back and returns to this screen** -- The screen re-reads the current Booking and Balance Payment state; if a Balance Payment already Succeeded (a prior attempt succeeded but she navigated away before seeing it), she is shown the Success state immediately rather than a fresh entry form, per FEAT-22.SPEC-004.
- **Talia commits a cancellation while Riley is mid-attempt (concurrent-edit conflict on the Booking)** -- Per the dependency map's Contention note for Balance Payment ("a cancellation committed first blocks the payment"), FEAT-22.SPEC-004's contention check resolves this as reject-with-refresh: if the cancellation commits first, no charge is requested (or, if already sent, it is not applied) and this screen shows the Cancelled state; Riley is never charged for a cancelled booking.
- **Riley's payment is captured, and the booking is cancelled moments afterward** -- Handled entirely by FEAT-22.SPEC-004 and FEAT-30 -- the payment already succeeded is refunded in full alongside the deposit (XBR-23); this screen has already shown Success and is not reopened to show the refund, which Riley sees reflected on her "my bookings" view instead.
- **Connection drops mid-payment** -- Handled by the Offline/Degraded state above; the outcome is resolved by FEAT-22.SPEC-004 and never results in an ambiguous charge.
- **The payment-processing capability itself is slow or unavailable** -- Covered by FEAT-22.SPEC-005's Degradation Behavior for this screen; this screen shows whatever exact message that spec defines for the Slow/Down/Rejects conditions.
- **Riley reaches this screen for a booking whose balance is already fully paid (in person or a prior in-app payment)** -- Cannot occur under normal operation, since FEAT-06.SPEC-004 only shows its "Pay Balance" button while a balance is genuinely due (per that spec's own Business Rules); if reached anyway (a stale link), FEAT-22.SPEC-003's eligibility check finds no balance due and the screen shows "There's nothing left to pay on this booking." with a single "Back to my bookings" action.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Navigation (inbound) | Riley arrives here by tapping "Pay Balance" on her own confirmed booking |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Navigation (outbound) | Back arrow, and the outcome of every terminal state, return Riley here |
| FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules) | References (inbound) | Supplies the locked balance amount and the eligibility preconditions checked before every charge attempt |
| FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention) | References (inbound) | Governs double-submit prevention, connection-drop resolution, the never-ambiguous-outcome guarantee, and the cancellation-contention resolution this screen reflects |
| FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund) | Triggers (outbound) | The Pay action requests authorization and capture through this integration; its decline translation populates the Error state |
| FEAT-22.SPEC-002 (Balance Capture & Booking Status Update) | Triggered by (inbound) | This automation's successful outcome is what moves the screen into the Success state |
| FEAT-23.SPEC-002 (Tip Amount Validation) -- within FEAT-23 (Tipping at Checkout, Later) | References (inbound) | Rule spec enforced at this screen (Enforced-By): once FEAT-23 ships, the tip value is validated against FEAT-23.SPEC-002's tip-only constraints before it is handed to the Pay submission, and an invalid tip blocks the submission. This screen never validates or alters `tip`; at v1 (no tip entered) the check is a no-op |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| balance_payment_attempted | booking reference, balance amount, is_retry (true/false) | Riley taps Pay | supports success-metrics.md: "Payout Transparency" (a visible, correctly attempted balance payment is part of a Pro seeing every deduction and deposit-like flow accounted for without asking) |
| balance_payment_succeeded | booking reference, balance amount | FEAT-22.SPEC-002 confirms the Booking's balance-due status is fully paid | supports success-metrics.md: "Payout Transparency" |
| balance_payment_failed | booking reference, decline reason category | FEAT-22.SPEC-005 reports a decline | supports success-metrics.md: "Payout Transparency" |
| balance_payment_blocked_by_cancellation | booking reference | FEAT-22.SPEC-004's contention check resolves against the attempt because a cancellation committed first | N/A -- no Stage 2 metric measures this specific contention outcome; retained so the never-double-charge, never-forfeited guarantee's exercise rate is observable to the Pro's activity record (FEAT-16), not to a success metric |

## Acceptance Criteria

**FEAT-22.SPEC-001-AC-01:** Given Riley is on the Balance Payment screen for a booking with a deposit of {D} and a locked balance of {B}, when the screen loads, then she sees "Deposit paid: {D}" and "Balance due: {B}" with no way to edit either figure.

**FEAT-22.SPEC-001-AC-02:** Given Riley has entered valid card details, when she taps "Pay {B} balance," then the button enters the Processing state and the status region reads "Processing payment, do not close this page."

**FEAT-22.SPEC-001-AC-03:** Given Riley's payment is in the Processing state, when she taps the Pay button again, then nothing happens and the button remains in the Processing state.

**FEAT-22.SPEC-001-AC-04:** Given Riley's card is declined, when FEAT-22.SPEC-005 reports the decline, then the status region shows the plain-language decline reason and a "Try a different card" action.

**FEAT-22.SPEC-001-AC-05:** Given Riley sees a decline message, when she taps "Try a different card," then the card entry element clears and refocuses, and the same locked balance amount is still shown.

**FEAT-22.SPEC-001-AC-06:** Given Riley's balance charge succeeds, when FEAT-22.SPEC-002 confirms the Booking's balance-due status is fully paid, then the status region shows "Balance received -- your booking is now fully paid." and the running-record block updates to show a zero balance.

**FEAT-22.SPEC-001-AC-07:** Given Talia commits a cancellation on Riley's booking in the instant before Riley's payment attempt is applied, when FEAT-22.SPEC-004's contention check resolves, then Riley sees the Cancelled state with "This booking was just cancelled, so there's nothing to pay. Your card was not charged." and no charge is applied.

**FEAT-22.SPEC-001-AC-08:** Given Riley is on the Cancelled state, when she taps "Back to my bookings," then she is navigated to FEAT-06.SPEC-004 (Booking Detail via Manage Link) showing the booking's current cancelled state.

**FEAT-22.SPEC-001-AC-09:** Given Riley loses connectivity before tapping Pay, when she attempts to tap Pay, then the button is disabled with the message "You're offline -- reconnect to pay your balance."

**FEAT-22.SPEC-001-AC-10:** Given Riley's connection drops after she taps Pay but before a result is confirmed, when the drop is detected, then the status region shows "We couldn't confirm your payment. Check your connection; if you were charged, your booking will show as fully paid the moment you reconnect -- you will never be charged twice." with a "Try again" action.

**FEAT-22.SPEC-001-AC-11:** Given Riley reconnects after a dropped connection and her charge had in fact succeeded, when the screen re-checks the Booking state, then it shows the Success state directly rather than allowing a second charge attempt.

**FEAT-22.SPEC-001-AC-12:** Given Riley navigates away from this screen after a successful charge, when she returns to this screen, then she is shown the Success state immediately rather than a fresh entry form.

**FEAT-22.SPEC-001-AC-13:** Given a visitor reaches this screen's address without a valid access link for the booking, when the screen attempts to load, then it shows "Request a new link" rather than the payment screen.

**FEAT-22.SPEC-001-AC-14:** Given Riley's access link expires while she is viewing this screen, when she attempts to interact further, then she is returned to the "Request a new link" prompt and any card details she had begun entering are discarded.

**FEAT-22.SPEC-001-AC-15:** Given Talia (the Pro) has no path that reaches this screen, when she looks for the resulting paid/unpaid status of a client's balance, then she finds it on FEAT-12 (Pro Daily Schedule Dashboard) or FEAT-30 (Pro Booking Management), never on this screen.

**FEAT-22.SPEC-001-AC-16:** Given Platform Operator (Support) opens a Pro's account after a help request, when they look at a booking's balance status, then they see the transaction status only, via FEAT-16/FEAT-19, never this in-flow payment screen or card data.

**FEAT-22.SPEC-001-AC-17:** Given Riley's payment succeeds and the booking is cancelled by either party moments afterward, when the cancellation is committed, then the already-shown Success state on this screen is not reopened or altered, and the resulting full refund is reflected on Riley's "my bookings" view instead (FEAT-22.SPEC-004, XBR-23).

**FEAT-22.SPEC-001-AC-18:** Given Riley reaches this screen for a booking whose balance is already fully paid, when the screen loads and FEAT-22.SPEC-003's eligibility check finds no balance due, then she sees "There's nothing left to pay on this booking." with a single "Back to my bookings" action.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 6 (default, processing, success, error, cancelled, offline/degraded) | 6 |
| Business Rules | 6 | 6 |
| Edge Cases | 7 | 7 |



# Automation Spec: Balance Capture & Booking Status Update

## Overview

**Name:** Balance Capture & Booking Status Update
**ID:** FEAT-22.SPEC-002
**Type:** Automation
**Purpose:** On a successful in-app balance card charge, the system creates the Balance Payment record and updates the Booking's balance-due status to fully paid, so Talia's dashboard reflects it without a manual refresh.
**Parent Feature:** FEAT-22 -- In-App Balance Payment

## Scope and Non-Goals

**In Scope:**
- Creating the Balance Payment record the instant a balance charge is reported captured
- Recomputing and reflecting the Booking's derived balance_due status as fully paid in the same step as that creation
- The exactly-zero-or-one-Balance-Payment-per-Booking guarantee at the point of capture (working with FEAT-22.SPEC-003's precondition check and FEAT-22.SPEC-004's idempotency guarantee)
- Feeding the fully-paid booking into the Pro's schedule and booking-management surfaces, and into the success state Riley sees

**Non-Goals:**
- Requesting authorization and capture from the payment-processing capability, or routing the captured amount to Talia's payout account -- owned by FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund); this automation only reacts to that spec's reported outcome
- Computing the balance amount -- owned by FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules); this automation only persists the amount already locked by that spec
- Guaranteeing the charge is never duplicated across a dropped connection, an interrupted status update, or a race with a Pro cancellation -- owned by FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention); this automation implements the create-and-update step that spec's guarantee wraps around
- Composing or sending a confirmation message -- excluded per the Brief's Communications field: the balance payment confirmation is a same-screen success state on FEAT-22.SPEC-001, not a delivered message this automation triggers, per the inline-communication exception this feature's `notification_count: 0` reflects

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Balance charge reported captured | FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund) | Fires when the payment-processing capability reports a successful capture for an attempt that passed FEAT-22.SPEC-003's eligibility check and FEAT-22.SPEC-004's contention check | Booking reference, captured amount, currency, capture timestamp |

This automation has exactly one trigger. It is fired once per successful capture event; a capture reported for a Booking that already has a Succeeded Balance Payment (per the idempotency guarantee in FEAT-22.SPEC-004) does not reach this automation as a new run -- see Edge Cases.

## Processing Logic

1. Receive the capture event from FEAT-22.SPEC-005: the Booking reference, captured amount, currency, and capture timestamp.
2. Confirm the referenced Booking is still in a payable state and that no Balance Payment with status Succeeded already exists for it (the one-succeeded-payment-per-booking guarantee, enforced jointly with FEAT-22.SPEC-004). If a Succeeded Balance Payment already exists for this Booking, treat this as a duplicate delivery of the same capture event and take no further action (see Edge Cases).
3. Create the Balance Payment record: amount from the capture event, state set directly to Succeeded, and the capture timestamp recorded. tip is left unset (owned by FEAT-23, Later, if it ever ships).
4. In the same atomic step as creating the Balance Payment, recompute the Booking's derived balance_due (price_agreed minus deposit_amount minus this Balance Payment's amount) to zero, and reflect the Booking's balance-due status as fully paid.
5. Signal FEAT-22.SPEC-001 (Balance Payment) that the Booking's balance-due status is now fully paid, so the screen can show its Success state.
6. Signal FEAT-12 (Pro Daily Schedule Dashboard) and FEAT-30 (Pro Booking Management) that the Booking now shows "fully paid" instead of "balance due," so Talia's schedule reflects it without a manual refresh.
7. Signal FEAT-28 (Payout Account Connection & Payout Visibility) that the captured balance is available for its money list, once FEAT-22.SPEC-005's payout routing confirms.
8. Signal FEAT-16 (Booking & Payment Activity Record) that a balance_payment_succeeded event occurred, for the append-only activity record.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Capture confirmed | The referenced Booking is still payable and has no existing Succeeded Balance Payment | Balance Payment created (state: Succeeded); Booking's derived balance_due recomputed to zero and its balance-due status reflected as fully paid | Riley sees the Success state on FEAT-22.SPEC-001; Talia sees "fully paid" on her schedule and booking-management surfaces | FEAT-22.SPEC-001, FEAT-12, FEAT-16, FEAT-28, FEAT-30 |
| Duplicate capture event ignored | A Succeeded Balance Payment already exists for the referenced Booking (the capture event was delivered more than once) | None -- the existing Balance Payment and fully-paid status are left unchanged | None -- Riley already saw the Success state from the first delivery | None beyond the existing state |
| Referenced Booking not eligible | The referenced Booking is no longer in a payable state at the moment this automation runs (for example, a Pro cancellation committed first, per FEAT-22.SPEC-004's contention rule) | None -- no Balance Payment is created against an ineligible Booking | The capture is reported back to FEAT-22.SPEC-005 as unappliable, which FEAT-22.SPEC-004 resolves per its correctness guarantee (never silently keep money for a cancelled booking) -- the amount is refunded rather than recorded as a kept balance payment | FEAT-22.SPEC-004, FEAT-22.SPEC-005 |
| Automation failure (processing error after capture confirmed) | The capture event is received but this automation cannot complete the create-and-update step (for example, an internal fault interrupts step 3 or 4) | No partial state is left visible: either both the Balance Payment and the fully-paid status are committed together, or neither is | Riley's screen (FEAT-22.SPEC-001) shows the Offline/Degraded resolution defined by FEAT-22.SPEC-004 -- she is never shown an ambiguous or double-charged state; the automation retries the create-and-update step automatically | FEAT-22.SPEC-001, FEAT-22.SPEC-004 |

## Data Model

**Reads:** Booking -- balance-payable state, price_agreed. Deposit Transaction -- amount (to compute the derived balance_due alongside this automation's own captured amount). Balance Payment -- read internally to check for an existing Succeeded record before creating a new one (the one-succeeded-payment-per-booking guarantee).
**Creates:** Balance Payment -- amount, state (set to Succeeded), capture timestamp. Exactly zero or one Succeeded Balance Payment per Booking, ever, from this automation.
**Updates:** Booking -- the derived balance_due field (recomputed to zero) and the balance-due status shown to the Pro. This is the only Booking-facing update this automation writes.
**Deletes:** None.

## Business Rules

- The Balance Payment creation and the Booking's fully-paid status update happen as a single atomic step -- one can never persist without the other.
- At most one Succeeded Balance Payment is ever created per Booking; a second capture event for the same Booking is a duplicate delivery, never a second charge (FEAT-22.SPEC-004).
- The amount written to the Balance Payment is exactly the value FEAT-22.SPEC-005 reports as captured, which must equal the amount FEAT-22.SPEC-003 locked -- this automation does not recompute or adjust the amount.
- This automation is the sole writer of the Booking's fully-paid balance-due status arising from an in-app balance payment; the Booking's other fields and state transitions (cancel, reschedule, no-show, complete) are owned by FEAT-05, FEAT-10, FEAT-11, FEAT-12, FEAT-21, and FEAT-30, per the Entity-Lifecycle Coverage Matrix.
- The fully-paid status is instant from Riley's perspective: the Booking reflects it the moment capture is reported, not on a delay or batch cycle, consistent with the deposit feature's own instant-confirmation practice (FEAT-07.SPEC-002).
- XBR-23: a balance payment this automation creates is never subject to forfeiture and is refunded in full if either party later cancels -- this automation itself performs no refund; it only records the successful capture.

## Edge Cases

- **Duplicate capture event for the same Booking** -- The second (and any subsequent) delivery finds an existing Succeeded Balance Payment and takes no action; the Booking remains fully paid with its original capture timestamp. No duplicate feedback fires.
- **Capture event arrives for a Booking a Pro cancellation has since resolved against (per FEAT-22.SPEC-004's contention rule)** -- No Balance Payment is created; the outcome is escalated to FEAT-22.SPEC-004 as a payment succeeded against an ineligible booking, which that spec's correctness guarantee resolves by refunding the captured amount rather than silently keeping it or leaving it unrecorded.
- **Status-update-step processing fails after the charge succeeded** -- The create-and-update step either fully commits or does not commit at all; a partial state (Balance Payment created but the Booking still showing balance due, or the reverse) never exists. If the step has not yet committed, it is retried automatically; Riley's screen shows the safe-retry behavior FEAT-22.SPEC-004 defines rather than a false failure.
- **Concurrent trigger firing (two capture events for two different Bookings at effectively the same time)** -- Each runs independently against its own Booking and Balance Payment; there is no shared state between two different Bookings' captures, so neither run affects the other.
- **Trigger fires while a previous run for the same Booking is still in flight** -- Cannot occur under normal operation, because FEAT-22.SPEC-004's one-succeeded-payment-per-booking guarantee ensures a second charge attempt for the same payable Booking is never authorized while the first is being captured; if a second capture event nonetheless arrives before the first run has finished committing, it is treated exactly as the duplicate-capture-event case once the first run's Balance Payment becomes visible.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund) | Triggered by (inbound) | A reported successful capture fires this automation |
| FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules) | References (inbound) | Confirms the captured amount matches the locked computation |
| FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention) | References (inbound) | Governs the atomicity guarantee this automation implements and the resolution when a capture cannot be applied |
| FEAT-22.SPEC-001 (Balance Payment) | Affects (outbound) | The fully-paid status update is what moves that screen into its Success state |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | The Booking's paid/unpaid status becomes visible on Talia's schedule |
| FEAT-30 (Pro Booking Management) | Affects (outbound) | The Booking's paid/unpaid status becomes visible on the Pro's booking management surface |
| FEAT-28 (Payout Account Connection & Payout Visibility) | Affects (outbound) | The captured balance is what that feature's money list reflects, once payout routing confirms |
| FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | balance_payment_succeeded is written to the append-only activity record |

## Analytics and Success Signals

- **balance_capture_confirmed** (booking reference, balance amount) -- supports success-metrics.md: "Payout Transparency"
- **balance_capture_duplicate_ignored** (booking reference) -- N/A -- no Stage 2 metric measures duplicate-delivery frequency directly; retained so the one-succeeded-payment-per-booking guarantee's exercise rate is observable to the Pro's activity record (FEAT-16), not to a success metric
- **balance_capture_not_applied** (booking reference, reason: booking_no_longer_payable) -- N/A -- no Stage 2 metric measures this exact non-applied path; retained so a captured-but-not-recorded amount (always resolved by a refund per FEAT-22.SPEC-004) is never silently unobservable

## Acceptance Criteria

**FEAT-22.SPEC-002-AC-01:** Given Riley's card is authorized and captured for a Booking that is still payable, when FEAT-22.SPEC-005 reports the capture, then a Balance Payment is created with state Succeeded and the Booking's balance_due is recomputed to zero in the same step.

**FEAT-22.SPEC-002-AC-02:** Given a Succeeded Balance Payment was just created for a Booking, when the same capture event is delivered a second time, then no second Balance Payment is created and the Booking's fully-paid status is unchanged.

**FEAT-22.SPEC-002-AC-03:** Given a capture event arrives for a Booking a Pro cancellation has already resolved against, when the automation checks the Booking's state, then no Balance Payment is created and the outcome is escalated to FEAT-22.SPEC-004.

**FEAT-22.SPEC-002-AC-04:** Given the Balance Payment is created and the Booking's balance-due status is updated, when the transition completes, then FEAT-22.SPEC-001 shows its Success state.

**FEAT-22.SPEC-002-AC-05:** Given Talia is viewing her schedule (FEAT-12) at the moment a client's balance is captured, when the automation confirms the Booking, then the booking's status updates to "fully paid" without Talia needing to refresh.

**FEAT-22.SPEC-002-AC-06:** Given a Booking's balance has just been captured by this automation, when the update completes, then a balance_payment_succeeded event is written to the append-only activity record (FEAT-16).

**FEAT-22.SPEC-002-AC-07:** Given the create-and-update step is interrupted by a processing error after the charge succeeded, when the automation retries, then the Balance Payment and the fully-paid status either both persist or neither does -- Riley is never shown a state where one exists without the other.

**FEAT-22.SPEC-002-AC-08:** Given two different clients' balance captures are reported at effectively the same time, when both automations run, then each creates its own Balance Payment and updates its own Booking independently, with no interference between the two runs.

**FEAT-22.SPEC-002-AC-09:** Given a captured amount reported by FEAT-22.SPEC-005 for a Booking, when this automation writes the Balance Payment, then the recorded amount exactly matches the balance amount FEAT-22.SPEC-003 locked on that Booking.

**FEAT-22.SPEC-002-AC-10:** Given a Succeeded Balance Payment has been created for a Booking, when any later capture event for that same Booking is evaluated, then it is recognized as a duplicate and produces no new Balance Payment.

**FEAT-22.SPEC-002-AC-11:** Given a Booking's balance is captured by this automation, when FEAT-30 (Pro Booking Management) next loads that booking, then its status reflects fully paid.

**FEAT-22.SPEC-002-AC-12:** Given the captured amount is available for Talia's money list, when FEAT-22.SPEC-005's payout routing confirms, then FEAT-28 reflects the entry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 (confirmed, duplicate ignored, not applied, automation failure) | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Balance Amount & Eligibility Rules

## Overview

**Name:** Balance Amount & Eligibility Rules
**ID:** FEAT-22.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs how the balance amount is derived once from the service price and the deposit already paid, that it can never be altered by the client, and the preconditions that must hold before any in-app balance charge is attempted.
**Parent Feature:** FEAT-22 -- In-App Balance Payment
**Governed Entity:** Balance Payment (the amount field, as computed) jointly Booking (the balance-payability precondition fields only: the balance-payable slice of state, price_agreed, and the derived balance_due). All other Booking fields and state transitions are owned by other features' Logic/Rule specs, per the dependency map's Entity-Lifecycle ownership; the tip field on Balance Payment is owned by FEAT-23 (Later), not this spec.

## Scope and Non-Goals

**In Scope:**
- Deriving the balance amount exactly once from the Booking's price_agreed and the already-captured Deposit Transaction amount, in the Pro's account currency
- The precondition checks that must all hold before any balance charge attempt is authorized (Booking still in a payable state, balance_due greater than zero, currency locked and matching, payout account Active, no existing Succeeded Balance Payment for the Booking)
- Preventing the client from altering the derived amount by any means
- The one-succeeded-payment-per-booking rule at the point a charge is requested (working with FEAT-22.SPEC-004's idempotency and contention guarantees for the point a charge is captured or contested by a concurrent cancellation)

**Non-Goals:**
- Creating the Balance Payment record and updating the Booking's balance-due status on a successful charge -- owned by FEAT-22.SPEC-002 (Balance Capture & Booking Status Update); this spec only gates whether a charge attempt may proceed
- Guaranteeing correctness across a dropped connection, duplicate delivery, or a race with a Pro cancellation -- owned by FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention)
- Authorizing or capturing the card charge itself, or executing the refund when a paid balance is cancelled -- owned by FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund); this spec defines only whether a charge may be requested, never how the charge or refund is processed
- Computing or validating the original deposit amount -- owned by FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules); this spec only reads the already-locked Deposit Transaction amount as an input to its own derivation

## Governed Entity

**Entity:** Balance Payment (amount) and Booking (balance-payability precondition fields), governed jointly
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| amount (Balance Payment) | number | The balance derived once from price_agreed minus the Deposit Transaction amount, fixed at the moment a charge succeeds; not client-alterable |
| tip (Balance Payment) | number, optional | Not governed by this spec -- owned entirely by FEAT-23 (Tipping at Checkout, Later); this spec's derivation and eligibility checks never read or depend on it |
| state (Balance Payment) | enum | Attempted \| Succeeded \| Failed \| Refunded. This spec reads only whether a Succeeded record already exists, as an eligibility precondition; it never writes this field (FEAT-22.SPEC-002 owns the transition to Succeeded) |
| price_agreed (Booking) | number | The service price agreed and fixed at the moment of booking, in the Pro's account currency (read-only here; owned by FEAT-05/FEAT-01) |
| balance_due (Booking) | derived | price_agreed minus deposit_amount minus any in-app balance payment; this spec derives the amount to attempt a charge for from this same formula, before any Balance Payment yet exists |
| state (Booking, balance-payable slice) | enum | Must be Confirmed or Awaiting Outcome for a charge attempt to be eligible; this spec reads it as a precondition and never writes it |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-22.SPEC-001 | Balance Payment | Reads the locked balance amount for display on screen load; requests an eligibility check from this spec on every Pay tap, including retries |
| FEAT-22.SPEC-002 | Balance Capture & Booking Status Update | Confirms, before creating the Balance Payment, that the captured amount matches the amount this spec locked and that the eligibility precondition held at charge time |
| FEAT-22.SPEC-005 | Balance Charge, Payout Routing & Refund | Requests this spec's eligibility check immediately before submitting an authorization/capture request to the payment-processing capability |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| amount (Balance Payment) | Derived once, exactly, as price_agreed minus the Deposit Transaction's captured amount, at the moment a charge attempt is requested; never accepted as client-supplied input | Always | On every eligibility check, immediately before a charge is requested | N/A -- this is a system computation with no client-facing input to reject; Riley is never shown an editable amount field | Yes |
| amount (Balance Payment) | Must be greater than zero | Always -- a Booking whose deposit already equals its price has nothing left to pay in-app | On every eligibility check | "There's nothing left to pay on this booking." | Yes |
| amount (Balance Payment) | Must be at least the minimum chargeable amount (platform parameter: `minimum-chargeable-deposit` -- reused here as the platform's single "smallest amount a card payment can be taken for" floor, since the underlying value is not specific to deposits; see Business Rules) | Always | On every eligibility check | "This balance is too small to charge by card right now. It can still be paid in person at the appointment." | Yes |
| currency | Must equal the Pro Account's locked currency (XBR-25) | Always | On every eligibility check | "This booking's currency no longer matches the Pro's account. Please try again later or pay in person." | Yes |
| state (Booking) | Must be Confirmed or Awaiting Outcome | Always | On every eligibility check | "This time is no longer available." (if the Booking has entered a state incompatible with payment, e.g. Cancelled or Completed) | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Amount immutability | amount (Balance Payment) | The value derived at charge time must be byte-identical to price_agreed minus the Deposit Transaction amount; no interaction on FEAT-22.SPEC-001 (Balance Payment) or FEAT-22.SPEC-005 accepts or applies an override, discount, or client-entered amount | N/A -- there is no field through which an override could be entered, so this is enforced by omission rather than by rejecting an input |
| Eligibility precondition set | state (Booking), currency, balance_due, Payout Account.status, Balance Payment (existence check) | All of the following must hold simultaneously before a charge attempt is authorized: the Booking is Confirmed or Awaiting Outcome; currency matches the Pro Account's locked currency; balance_due is greater than zero and meets the minimum-chargeable floor; the Pro's Payout Account status is Active (XBR-06's payment-eligibility principle applied to the balance); and no Balance Payment with state Succeeded already exists for this Booking (one-succeeded-payment-per-booking) | See the per-condition messages above; the eligibility check as a whole fails closed -- if any condition is unmet, no charge is requested |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Request a balance charge for a Booking | The Client (Riley) | Only for her own confirmed Booking, and only while the eligibility precondition set (above) holds | If the Booking is not Confirmed or Awaiting Outcome: "This time is no longer available." If balance_due is zero: "There's nothing left to pay on this booking." If the currency check fails: "This booking's currency no longer matches the Pro's account. Please try again later or pay in person." If a Succeeded Balance Payment already exists (a second attempt after an already-successful charge, e.g. from a stale page): no new charge is requested; the screen is shown the Success state directly, per FEAT-22.SPEC-004. If the Payout Account is not Active: "This booking can't be paid right now. Please try again shortly, or pay {Pro's display name} in person." |
| View the derived balance amount and deposit-paid figure | The Client (Riley) | Only for her own booking | -- |
| Alter the derived balance amount | The Client (Riley) | Never | No control exists anywhere in the product for the client to alter the amount; the field is display-only everywhere it appears |
| View the resulting paid/unpaid status | The Pro (Talia) | Full, on her own schedule/booking-management surfaces (FEAT-12, FEAT-30) once the outcome resolves -- never during Riley's in-progress attempt, which she has no visibility into | -- |
| View balance status for support purposes | Platform Operator (Support) | View-only, on FEAT-16/FEAT-19, never card data or Riley's in-progress screen | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| amount (Balance Payment) | price_agreed minus the Deposit Transaction's captured amount minus any prior Balance Payment amount for this Booking (structurally always zero prior, since at most one Succeeded Balance Payment ever exists per Booking) | Derived fresh on every eligibility check, and locked into the Balance Payment record only at the moment a charge succeeds (FEAT-22.SPEC-002) | No -- not by the Client, not by the Pro, not by Support |
| currency | The Pro Account's currency, locked at that account's first-ever successful deposit (XBR-25) | Read at every eligibility check | No |

## Business Rules

- XBR-23: Balance due equals service price minus deposit minus any in-app balance payment; a paid balance is never forfeited and is refunded in full if either party cancels.
- XBR-07: Chairtime's fee on the balance is always zero, exactly as for deposits and tips; the routing of the captured amount is FEAT-22.SPEC-005's own responsibility, not a rule this spec computes.
- XBR-06's underlying principle (no deposit can be taken unless the Pro's payout account is Active) is applied identically to the balance: no in-app balance charge is requested unless the Payout Account is Active, since the same connected account receives both.
- XBR-25: Currency is locked to the Pro Account's currency once the account's first deposit is taken; every eligibility check re-confirms the Booking's currency still matches.
- The marker platform parameter: `minimum-chargeable-deposit` is reused verbatim for the balance's minimum-chargeable floor rather than minting a second marker: both represent the same real-world constraint (the smallest amount the payment-processing capability can charge a card for), and the parameter's name reflects where it was first defined, not a deposit-only scope. Reconciliation (Pass D) confirmed this reuse: the registry carries one row for this slug covering both floors.
- One-succeeded-payment-per-booking is enforced at two points for defense-in-depth: this spec blocks a second charge *request* once a Succeeded Balance Payment exists (checked before the request is even sent), and FEAT-22.SPEC-004 guarantees correctness of the charge *outcome* even if two requests were somehow both sent.
- The eligibility check runs identically on the first attempt and on every retry after a decline -- there is no relaxed re-check path for retries.
- Unlike the deposit's checkout-hold-scoped eligibility (FEAT-07.SPEC-003), this spec's eligibility check has no time-limited hold: Riley may return to pay the balance "at any point before or at the appointment" (product-features.md Key Capabilities), so the check simply re-runs fresh on every attempt against the Booking's current state rather than against a slot hold.

## Edge Cases

- **Riley's Booking's deposit already equals its full price (a service whose deposit_rule is 100%)** -- balance_due is zero; FEAT-06.SPEC-004 (Booking Detail via Manage Link) never shows its "Pay Balance" button for such a booking in the first place, and if this screen is reached anyway (a stale link), the eligibility check fails with "There's nothing left to pay on this booking."
- **Percentage or fixed deposit computation left a very small remaining balance, below the minimum-chargeable floor** -- The eligibility check fails with "This balance is too small to charge by card right now. It can still be paid in person at the appointment." -- the in-person default remains available exactly as it always is.
- **Payout Account transitions from Active to Action Required between screen load and Pay tap** -- The eligibility check re-runs at the Pay tap (not only at screen load) and catches the change; the charge is not requested and Riley sees "This booking can't be paid right now..." while Talia separately sees the Action Required banner on her own dashboard (FEAT-12).
- **Two rapid attempts both reach the eligibility check before either creates a Balance Payment** -- Both may pass eligibility (no Succeeded Balance Payment exists yet for either), but only one may actually be captured as the successful charge; the second's capture attempt is resolved by FEAT-22.SPEC-004's idempotency guarantee, never by this spec, which only gates the request, not the outcome.
- **A second Pay attempt is made after an already-successful charge (e.g., a stale reloaded page)** -- The eligibility check finds an existing Succeeded Balance Payment and denies the request; the screen shows the Success state directly rather than requesting a second charge.
- **The Booking's balance-payable state changes to Cancelled between screen load and Pay tap because Talia commits a cancellation** -- The eligibility check re-runs at the Pay tap and fails on the state precondition; no charge is requested. This is the eligibility-side half of the contention FEAT-22.SPEC-004 fully governs for the case where the cancellation and the charge race even more tightly than a screen-load-to-tap gap.

## Acceptance Criteria

**FEAT-22.SPEC-003-AC-01:** Given a Booking with price_agreed of {P} and a captured Deposit Transaction of {D}, when Riley's eligibility check runs, then the balance amount is derived as exactly {P minus D}.

**FEAT-22.SPEC-003-AC-02:** Given Riley's Booking has a derived balance amount, when she views the Balance Payment screen, then no control anywhere lets her change that amount.

**FEAT-22.SPEC-003-AC-03:** Given Riley's Booking is Confirmed, currency matches, the Pro's payout account is Active, balance_due is greater than zero and above the minimum-chargeable floor, and no Succeeded Balance Payment exists for it, when she taps Pay, then the eligibility check passes and a charge is requested.

**FEAT-22.SPEC-003-AC-04:** Given Riley's Booking's balance_due is zero because the deposit already equals the full price, when she reaches this screen and the eligibility check runs, then no charge is requested and she sees "There's nothing left to pay on this booking."

**FEAT-22.SPEC-003-AC-05:** Given Riley's Booking's derived balance amount falls below platform parameter: `minimum-chargeable-deposit`, when the eligibility check runs, then no charge is requested and she sees "This balance is too small to charge by card right now. It can still be paid in person at the appointment."

**FEAT-22.SPEC-003-AC-06:** Given the Pro's payout account is not Active at the moment Riley attempts to pay, when the eligibility check runs, then no charge is requested and she sees "This booking can't be paid right now. Please try again shortly, or pay {Pro's display name} in person."

**FEAT-22.SPEC-003-AC-07:** Given a Succeeded Balance Payment already exists for Riley's Booking, when she taps Pay again from a stale page, then no new charge is requested and she is shown the Success state directly.

**FEAT-22.SPEC-003-AC-08:** Given the Booking's currency no longer matches the Pro Account's locked currency, when the eligibility check runs, then no charge is requested and Riley sees "This booking's currency no longer matches the Pro's account. Please try again later or pay in person."

**FEAT-22.SPEC-003-AC-09:** Given Riley's Booking is no longer Confirmed or Awaiting Outcome (for example, it was cancelled), when she taps Pay, then the eligibility check fails and she sees "This time is no longer available."

**FEAT-22.SPEC-003-AC-10:** Given Riley (the Client) is viewing her own booking, when she looks at the balance amount, then she sees exactly the derived value with no edit affordance.

**FEAT-22.SPEC-003-AC-11:** Given Talia (the Pro) has not yet had this Booking's balance captured, when she looks at her own schedule or booking-management surfaces, then she sees no visibility into Riley's in-progress balance-payment attempt -- only the eventual fully-paid status once capture succeeds.

**FEAT-22.SPEC-003-AC-12:** Given Platform Operator (Support) opens a Pro's account after a help request, when they look at a booking's balance status, then they see the transaction status only, via FEAT-16/FEAT-19, never the in-progress payment screen or card data.

**FEAT-22.SPEC-003-AC-13:** Given the Pro's payout account transitions to Action Required between Riley's screen load and her Pay tap, when she taps Pay, then the eligibility check re-runs and denies the charge with the same message as AC-06, rather than relying on the state at screen load.

**FEAT-22.SPEC-003-AC-14:** Given Riley retries payment after an earlier decline, when the retry's eligibility check runs, then it applies the identical full precondition set as the first attempt -- no relaxed re-check path exists for retries.

**FEAT-22.SPEC-003-AC-15:** Given Riley opens the Balance Payment screen for a booking and then Talia commits a cancellation before Riley taps Pay, when the eligibility check runs at the Pay tap, then it fails on the Booking-state precondition and no charge is requested.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 8 | 8 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Balance Payment Outcome Consistency & Cancellation Contention

## Overview

**Name:** Balance Payment Outcome Consistency & Cancellation Contention
**ID:** FEAT-22.SPEC-004
**Type:** Logic/Rule
**Purpose:** Guarantees every balance payment attempt ends in exactly one clean outcome -- never a double charge and never a partial or ambiguous "balance due" state -- and resolves the case where a Pro cancellation and a client's balance payment race each other, per XBR-23.
**Parent Feature:** FEAT-22 -- In-App Balance Payment
**Governed Entity:** Balance Payment (creation idempotency) jointly Booking (the balance-payable / cancelled contention slice) -- this spec's correctness guarantee spans both, since a payment attempt is only "resolved" once the two are consistent with each other and with any concurrent cancellation.

## Scope and Non-Goals

**In Scope:**
- The guarantee that at most one Succeeded Balance Payment is ever created per Booking, regardless of retries, dropped connections, or duplicate event delivery
- The guarantee that the Booking's balance-due status always matches the true outcome of the most recent charge attempt, even when Riley's device never receives confirmation of that outcome
- The contention rule between a Pro-initiated cancellation and a client-initiated balance payment on the same Booking (dependency map, Balance Payment entity, Contention): reject-with-refresh, where the first committed transition wins
- Resolution behavior when a captured balance charge cannot be applied because a cancellation committed first, and when an already-succeeded balance payment is followed by a cancellation

**Non-Goals:**
- Computing the balance amount or checking the eligibility preconditions before a charge is requested -- owned by FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules); this spec governs correctness *after* a charge attempt has been requested, not whether it should have been requested
- Creating the Balance Payment and updating the Booking's balance-due status themselves -- owned by FEAT-22.SPEC-002 (Balance Capture & Booking Status Update); this spec defines the guarantee that automation's create-and-update step must uphold, not the step's own mechanics
- Authorizing or capturing the card charge, or executing the refund call once a refund is due -- owned by FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund)
- The Pro's own decision to cancel, and the cancellation commit mechanics themselves -- owned by FEAT-30 (Pro Booking Management, for a Pro-initiated cancellation) and FEAT-09 (Cancellation & No-Show Policy Engine, for a client-initiated or automatic cancellation); this spec only defines how a balance payment attempt behaves when one of those cancellations wins the race

## Governed Entity

**Entity:** Balance Payment (creation) and Booking (balance-payable / cancelled transition), governed jointly
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| Balance Payment (existence, per Booking) | derived | Whether a Succeeded Balance Payment exists for a given Booking -- the fact this spec guarantees is single-valued (zero or one, never more) |
| Balance Payment.state | enum | At this feature's stage: Attempted \| Succeeded \| Failed. This spec guarantees the state set here is the true, final outcome of the one attempt that succeeded -- or that no attempt succeeded if a cancellation won the race. Refunded is set later by FEAT-30, per the Entity-Lifecycle Coverage Matrix |
| Booking.state (balance-payable / cancelled slice) | enum | This spec guarantees the Booking's balance-due status reflects fully paid if and only if exactly one Balance Payment succeeded for it, and that a cancellation which commits before a balance charge is applied is never overwritten by a late-arriving charge outcome |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-22.SPEC-001 | Balance Payment | Disables the Pay button while a request is in flight (double-submit prevention); on screen load or return, re-checks the Booking's true state before allowing a new attempt, showing Success directly, or showing Cancelled |
| FEAT-22.SPEC-002 | Balance Capture & Booking Status Update | Implements the atomic create-and-update step this spec requires: the Balance Payment and the fully-paid status commit together or not at all, and never commit against a Booking a cancellation has already resolved |
| FEAT-22.SPEC-005 | Balance Charge, Payout Routing & Refund | Applies this spec's idempotency key discipline when submitting a charge request; executes the refund this spec's contention resolution requires when a payment already succeeded and a cancellation follows |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Balance Payment (existence, per Booking) | At most one Succeeded Balance Payment may ever exist for a given Booking | Always | Before every charge request (FEAT-22.SPEC-003) and again at capture time (FEAT-22.SPEC-002) | N/A -- enforced structurally by the create-and-update step refusing to create a second record, not by a client-facing error | Yes |
| Booking.state vs. Balance Payment (existence) | A Booking must never show fully paid unless exactly one Balance Payment with state Succeeded exists for it, and a Balance Payment must never be created against a Booking whose cancellation already committed | Always | Continuously -- this is an invariant the create-and-update step (FEAT-22.SPEC-002) maintains, not a point-in-time check | N/A -- this is a system invariant, not a validated input | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Atomic capture-and-update | Balance Payment (existence, state), Booking.state | The Balance Payment's creation with state Succeeded and the Booking's balance-due status update to fully paid must occur as one indivisible step: a partial state where one exists without the other is never observable to any client, screen, or downstream feature | N/A -- structural guarantee; if the step cannot complete, it is retried automatically until it does, per Business Rules below |
| Idempotency key discipline | Balance Payment (existence), the charge request itself | Every charge request FEAT-22.SPEC-005 submits to the payment-processing capability carries an identifier tied to this specific attempt on this specific Booking, so a request that is resubmitted (due to a client retry after a dropped connection, or a network-level retry) is recognized by the capability as the same attempt rather than a new charge | N/A -- structural guarantee at the integration boundary, not a client-facing error |
| Cancellation-vs-payment race (dependency map, Balance Payment Contention) | Booking.state (cancellation transition), Balance Payment (creation) | Whichever transition -- the Pro's (or client's, or automatic) cancellation, or the client's balance charge -- commits first is authoritative. If the cancellation commits first, the payment attempt is refused before or instead of being applied and no Balance Payment is created for it. If the payment attempt commits first (a Succeeded Balance Payment already exists), a cancellation that follows refunds it in full rather than blocking on it or being refused itself | Payment side: "This booking was just cancelled, so there's nothing to pay. Your card was not charged." Cancellation side: no error -- the cancellation always proceeds and the already-succeeded balance payment is refunded alongside the deposit (XBR-23), per FEAT-30's and FEAT-09's own cancellation commit logic |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Submit a new balance charge attempt for a Booking | The Client (Riley) | Only when no charge attempt is currently in flight for that Booking, no Succeeded Balance Payment yet exists for it, and no cancellation has committed against the Booking (per FEAT-22.SPEC-003's eligibility gate, which this spec's in-flight and contention checks extend) | If an attempt is already in flight: the Pay button remains disabled and the tap is ignored (FEAT-22.SPEC-001) -- no error message, since this is prevented before a second request can even be sent. If a Succeeded Balance Payment already exists: no new request is sent; she is shown the Success state directly. If a cancellation has committed: no charge is requested (or, if one was already sent and not yet applied, it is not applied); she sees "This booking was just cancelled, so there's nothing to pay. Your card was not charged." |
| Resolve an ambiguous outcome on the Client's behalf | The Client (Riley) | Never -- the resolution is always determined by the true state this spec reconciles (whether a capture actually occurred, and whether a cancellation won the race), never by a choice Riley makes | Riley is never asked "were you charged?" or given a manual "I was charged" override; the screen always reflects the system's own determination |
| Commit a cancellation while a balance payment attempt is in flight | The Pro (Talia, via FEAT-30) or the Client (via FEAT-10) or the product automatically (via FEAT-09) | Always -- a cancellation is never blocked by an in-flight or already-succeeded balance payment attempt; it proceeds and the payment attempt (or its already-succeeded outcome) is resolved against it, never the reverse | -- (the cancelling party's own action always proceeds; the resolution is visible to Riley on FEAT-22.SPEC-001 as the Cancelled state or, for an already-succeeded payment, as a refund reflected on her "my bookings" view) |
| View or intervene in a payment attempt's reconciliation | The Pro (Talia) | Never during the attempt itself; Full view of the resulting outcome once resolved, on her own schedule/booking-management surfaces | Talia has no visibility into an in-progress attempt; she sees only the resolved fully-paid/balance-due state, or the fact that her own cancellation won a race |
| View reconciliation outcomes for support purposes | Platform Operator (Support) | View-only, on FEAT-16/FEAT-19, after the outcome resolves | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Booking's balance-due status, on screen return after an interrupted confirmation | Derived from the true Balance Payment state at the moment the screen re-loads: fully paid if a Succeeded Balance Payment exists, Cancelled if the Booking's cancellation committed and no charge succeeded, otherwise unchanged (available for retry) | Whenever FEAT-22.SPEC-001 loads or re-loads for a given Booking | No -- this is always read fresh from the true state, never cached or assumed from the client's last known screen state |

## Business Rules

- **Never double-charge:** at most one successful capture is ever recorded against a Booking; a second charge attempt for the same Booking, however triggered, either finds an existing Succeeded Balance Payment and is refused before it reaches the payment-processing capability (FEAT-22.SPEC-003), or is submitted with the same idempotency key as an in-flight or already-resolved attempt and is recognized as the same attempt by that capability rather than processed as a new charge.
- **Never leave an ambiguous outcome:** every charge attempt resolves, from the product's point of view, to exactly one of Succeeded, not-Succeeded, or Cancelled-before-resolution. There is no third "unknown" state that persists -- if the outcome is not yet known (e.g., the connection dropped before Riley's device learned it), the product resolves it by checking the true state (has a Balance Payment actually been created? has a cancellation committed?) rather than guessing or leaving the Booking in limbo.
- **Cancellation always wins a genuine race, never a charge:** per the dependency map's Contention note for Balance Payment, a cancellation committed first blocks the payment; a payment committed first is refunded in full by a later cancellation (XBR-23). The product never resolves a race by letting a charge proceed against an already-cancelled booking, and never resolves it by blocking a cancellation because a payment succeeded first.
- **The Booking's true state is the source of truth, never the client's screen history:** if a charge succeeds but the confirmation fails to load on FEAT-22.SPEC-001, the fully-paid status is authoritative and is shown on the next page load -- Riley is never left unsure whether her balance is paid.
- **Automatic retry on interruption before resolution, never after:** if the create-and-update step (FEAT-22.SPEC-002) is interrupted before it commits, it is retried automatically until it commits; once it has committed (a Balance Payment with state Succeeded exists), no further retry of the charge itself ever occurs -- only screen-level re-loads of the already-resolved state.
- **Consistent with FEAT-22.SPEC-003's amount lock:** this spec's reconciliation never re-derives or re-checks the balance amount -- it only reconciles whether the one attempt at the one locked amount succeeded, and whether a cancellation intervened.
- **XBR-23 governs the money, never this spec:** once a cancellation-after-success outcome is determined, the actual refund execution is FEAT-22.SPEC-005's integration call, triggered by FEAT-30 or FEAT-09's own cancellation commit logic -- this spec establishes only that the refund is owed, never how it is executed.

## Edge Cases

- **Connection drops after Riley submits card details but before her device receives a result** -- On reconnection, FEAT-22.SPEC-001 re-loads the Booking's true state rather than assuming failure: if a Succeeded Balance Payment exists, she is shown Success; if the Booking was cancelled in the meantime, she is shown Cancelled; if neither and no attempt is in flight, she is shown the Default state ready for a fresh attempt.
- **Talia commits a cancellation the instant before Riley's in-flight charge is reported captured** -- The cancellation's commit is checked as part of the create-and-update step (FEAT-22.SPEC-002): if it committed first, the charge is not applied as a Balance Payment even though the card authorization succeeded, and the captured amount is refunded immediately through FEAT-22.SPEC-005 rather than recorded as a kept balance payment -- Riley is never charged for a cancelled booking.
- **Riley's balance payment succeeds, and Talia cancels the booking moments later** -- The cancellation proceeds normally (it is never blocked by an already-succeeded balance payment); FEAT-30's cancellation commit logic finds the Succeeded Balance Payment and triggers its full refund through FEAT-22.SPEC-005 alongside the deposit, per XBR-23.
- **Riley closes the browser tab immediately after tapping Pay, before any result is shown, then reopens her "my bookings" link later** -- The reopened flow re-checks the Booking's and Balance Payment's true state exactly as on a reconnect; if the earlier attempt had in fact succeeded, she is shown fully paid; if it failed or was superseded by a cancellation, she is shown accordingly.
- **Two devices attempt to pay the same Booking's balance at effectively the same time** -- Only one request can result in a Succeeded Balance Payment; the other, whichever resolves second, finds the Balance Payment already exists (per FEAT-22.SPEC-003's pre-request check, or the idempotency key at the capability boundary) and is treated as a no-op, with that device's screen showing Success directly rather than a duplicate charge or an error.
- **The payment-processing capability reports success for a charge request the product had already given up retrying (a very late, delayed response), and the Booking was cancelled in the meantime** -- The late success is checked against the Booking's current state before being applied: since the cancellation already committed, the late capture is not recorded as a Balance Payment and the captured amount is refunded immediately -- money is never lost or silently kept for a cancelled booking either way.

## Acceptance Criteria

**FEAT-22.SPEC-004-AC-01:** Given Riley's connection drops immediately after she taps Pay, when she reconnects and the screen reloads, then it shows Success if the charge in fact succeeded, Cancelled if the booking was cancelled in the meantime, or the Default state ready for a fresh attempt if neither -- never an ambiguous "unknown" state.

**FEAT-22.SPEC-004-AC-02:** Given a balance charge attempt for Riley's Booking has already succeeded, when any later request (a retry, a second device, a delayed capability response) reaches the point of creating a Balance Payment, then no second Balance Payment is created and the Booking remains fully paid with its original outcome.

**FEAT-22.SPEC-004-AC-03:** Given Talia commits a cancellation on Riley's booking the instant before Riley's in-flight charge is reported captured, when FEAT-22.SPEC-002's create-and-update step checks the Booking's state, then no Balance Payment is created and the captured amount is refunded immediately rather than kept.

**FEAT-22.SPEC-004-AC-04:** Given Riley's balance payment has already succeeded, when Talia later cancels the booking, then the cancellation proceeds normally and the Succeeded Balance Payment is refunded in full alongside the deposit, per XBR-23.

**FEAT-22.SPEC-004-AC-05:** Given Riley taps Pay and a request is in flight, when she taps Pay again before a result arrives, then the second tap produces no second request -- the button remains disabled and ignores the tap.

**FEAT-22.SPEC-004-AC-06:** Given the create-and-update step (FEAT-22.SPEC-002) is interrupted by a processing error before it commits, when the system retries, then it retries until the step commits, and Riley is never shown a state where the Balance Payment exists without the fully-paid status or vice versa.

**FEAT-22.SPEC-004-AC-07:** Given a create-and-update step has already committed successfully, when any further retry logic runs for that same attempt, then no further retry of the charge itself occurs -- only the already-resolved state is re-displayed.

**FEAT-22.SPEC-004-AC-08:** Given Riley closes the browser tab right after tapping Pay with no result shown, when she reopens her "my bookings" link later, then the screen shows the true current state of the Booking rather than assuming the earlier attempt failed.

**FEAT-22.SPEC-004-AC-09:** Given Riley opens the same booking's Balance Payment screen on two devices and taps Pay on both at effectively the same time, when both requests are processed, then only one results in a Succeeded Balance Payment, and the other device's screen shows Success directly rather than a duplicate charge or an error.

**FEAT-22.SPEC-004-AC-10:** Given the payment-processing capability reports a very late success for a request whose Booking was cancelled in the meantime, when the report arrives, then it is not recorded as a Balance Payment and the captured amount is refunded immediately.

**FEAT-22.SPEC-004-AC-11:** Given Riley (the Client) is asked to resolve an ambiguous payment outcome, when she looks for a manual "I was charged" option, then none exists -- the screen's state is always determined by the system's own reconciliation of the true Balance Payment and Booking state.

**FEAT-22.SPEC-004-AC-12:** Given Talia (the Pro) views her schedule while one of her client's balance payment attempts is still in flight, when she looks at that booking, then she sees no in-progress-attempt detail -- only the resolved fully-paid/balance-due state once it settles.

**FEAT-22.SPEC-004-AC-13:** Given every balance charge request this spec governs, when it is submitted to the payment-processing capability, then it carries an identifier tied to that specific attempt so a resubmission of the same request is recognized as the same attempt rather than processed as a new charge.

**FEAT-22.SPEC-004-AC-14:** Given a Balance Payment already exists with state Succeeded for a Booking, when the eligibility check in FEAT-22.SPEC-003 runs for any further attempt on that Booking, then it denies the request, upholding this spec's never-double-charge guarantee at the point of request as well as at the point of outcome.

**FEAT-22.SPEC-004-AC-15:** Given a Pro cancellation, a client cancellation, or an automatic cancellation commits against Riley's Booking while her balance payment attempt is unresolved, when the cancellation is checked against the payment attempt, then the cancellation is never blocked and Riley's attempt is resolved against it, never the reverse.

**FEAT-22.SPEC-004-AC-16:** Given Riley's charge is captured by the payment-processing capability but the cancellation on her Booking committed a moment earlier, when the outcome is resolved, then she sees the Cancelled state with "This booking was just cancelled, so there's nothing to pay. Your card was not charged." and the captured amount is refunded rather than recorded.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 7 | 7 |
| Edge Cases | 6 | 6 |



# Integration Spec: Balance Charge, Payout Routing & Refund

## Overview

**Name:** Balance Charge, Payout Routing & Refund
**ID:** FEAT-22.SPEC-005
**Type:** Integration
**Purpose:** Authorizes and captures Riley's in-app balance card charge through the payment-processing capability, routes the captured balance to Talia's connected payout account with zero platform fee, and executes the outbound refund call when a paid balance must be refunded in full.
**Parent Feature:** FEAT-22 -- In-App Balance Payment

## Scope and Non-Goals

**In Scope:**
- Requesting authorization and capture of a balance charge for an eligible Booking, using the card details Riley enters directly into the capability's own entry element
- Receiving and translating the capability's outcome (captured, declined) into plain-language feedback for FEAT-22.SPEC-001
- Routing the captured amount to the Pro's connected Payout Account, with Chairtime's own fee always zero (XBR-07)
- Executing the outbound refund call for a Succeeded Balance Payment once FEAT-22.SPEC-004's contention resolution determines a refund is due, guaranteeing it completes exactly once with automatic retry (XBR-23)
- User-facing behavior when the capability is slow, unavailable, or rejects a charge or refund request
- Disclosure to Riley about what data is shared with the capability

**Non-Goals:**
- Verifying or connecting the Pro's payout account itself (identity and bank verification) -- owned by FEAT-28.SPEC-006 (Payout Account Connection & Payout Visibility); this spec only routes an already-captured balance to an already-active account
- Computing the balance amount or checking eligibility preconditions -- owned by FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules); this spec only submits the charge for an amount and Booking that spec has already cleared
- Creating the Balance Payment record or updating the Booking's balance-due status -- owned by FEAT-22.SPEC-002 (Balance Capture & Booking Status Update); this spec reports the charge outcome that automation acts on, it does not write the Balance Payment itself
- Determining that a refund is due, or the underlying cancellation decision -- owned by FEAT-30 (Pro Booking Management, for a Pro-initiated cancellation) and FEAT-09 (Cancellation & No-Show Policy Engine, for a client-initiated or automatic cancellation) together with FEAT-22.SPEC-004's contention rule; this spec only executes a refund already determined to be due

## Capability Category

**Category:** Payment processing
**Dependency Source:** ASMP-31 -- "Payment-processing capability -- required to take client deposits, verify each pro's identity and bank details for a connected payout account, pay deposits out to the pro, issue refunds, notify the product of card-issuer disputes, and bill the pro's own monthly subscription." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Payment processing -- client card charges and refunds (deposits; from v1 balances; Later tips)" row in feature-dependency-map.md, ## External Touchpoints, which names this spec directly as "FEAT-22.SPEC-005 (in-app balance card authorization and capture with zero platform fee, and the outbound full refund of a paid balance when either party cancels, per XBR-23)"; and the "Payment processing -- connected payout accounts with identity and bank verification" row, which names this spec for routing each captured balance to the Pro's connected payout account
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision. BRIEF.md's Constraints establish only the hard boundary that card data is never stored or handled by the product's own code, not a named vendor.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Riley pays the exact balance amount computed for her booking, by card, without her card details ever touching the product's own code | Pay the remaining balance in-app at any point before or at the appointment | FEAT-22.SPEC-001 (Balance Payment) |
| Riley sees a clear, on-screen success record that her balance succeeded, and Talia's dashboard reflects "fully paid" | See a running record of deposit paid vs. balance remaining | FEAT-22.SPEC-001 (Balance Payment), FEAT-22.SPEC-002 (Balance Capture & Booking Status Update), FEAT-12 (Pro Daily Schedule Dashboard) |
| Talia's captured balance lands directly in her own payout account with no Chairtime cut | Balance goes straight to the Pro's payout account with no platform cut | FEAT-28 (Payout Account Connection & Payout Visibility) -- the money-list display itself; this spec supplies the routed amount |
| A paid balance is refunded in full, never partially or forfeited, the instant either party cancels | Refunded in full if the appointment is later cancelled by either party -- a balance is never subject to forfeiture | FEAT-22.SPEC-004 (contention and refund-due determination), FEAT-30 (Pro Booking Management, cancellation commit), FEAT-09.SPEC-005 (Automatic Deposit Refund, client/automatic cancellation), FEAT-23.SPEC-003 (tip refund completeness, once shipped) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Balance amount and currency | Booking -- price_agreed, currency (via the derived balance amount FEAT-22.SPEC-003 locks) | Riley taps Pay on FEAT-22.SPEC-001, after FEAT-22.SPEC-003's eligibility check and FEAT-22.SPEC-004's contention check both pass | The capability must know exactly what to authorize and capture |
| Booking reference | Booking -- an internal reference sufficient to tie the outcome back to this specific Booking | Same moment as above | Ties the capability's reported outcome (captured, declined) back to the correct Booking, and carries the idempotency key FEAT-22.SPEC-004 requires |
| Destination payout account reference | Payout Account -- processor_account_reference | Same moment as above | Tells the capability where to route the captured funds -- the Pro's own connected account, never Chairtime's |
| Card details Riley enters | Not a product entity -- entered directly into the capability's own entry element and never received by the product's own code, per SC-11 | While Riley fills the card entry element on FEAT-22.SPEC-001 | The capability needs the card details to attempt authorization; the product never touches or stores them |
| Refund amount and currency | Balance Payment -- amount | FEAT-22.SPEC-004's contention resolution determines a refund is due (a Pro, client, or automatic cancellation on a Booking with a Succeeded Balance Payment) | The capability must know exactly how much to return, matching the original captured amount |
| Balance Payment reference | Balance Payment -- the reference tying it to the original capture | Refund is requested | Ties the refund to the specific original charge so the capability reverses the correct transaction |
| Refund idempotency key | Derived -- an identifier tied to this specific refund attempt on this specific Balance Payment | Every refund request, including retries | Lets the capability recognize a resubmitted request as the same attempt rather than a second refund |

Booking's service name, appointment time, client name and phone, and every other Client or Booking field never leave the product through this integration -- only the amounts, currency, internal references, and the payout destination reference are shared.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Capture outcome (captured / declined) | The capability resolves an authorization/capture request | Consumed directly by FEAT-22.SPEC-002 (captured) or shown inline on FEAT-22.SPEC-001 (declined); no Balance Payment is written for a decline |
| Decline reason (plain-language category) | The capability reports a decline | Surfaced on FEAT-22.SPEC-001's Error state; not persisted on the Booking |
| Payout routing confirmation | The capability confirms the captured amount was routed to the destination Payout Account | Feeds Payout Account's recent-payouts data, displayed by FEAT-28's money list |
| Refund succeeded confirmation, with refund timestamp | The capability completes the refund | Balance Payment -- state (Refunded), refund timestamp |
| Refund cannot complete immediately, with a retry-eligibility signal | The capability reports the refund could not be completed on this attempt | Balance Payment -- state remains flagged as refund-in-progress; scheduled for this spec's own automatic retry |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Charge captured | The capability successfully authorizes and captures Riley's card for the requested balance amount | None directly in this spec -- the captured amount and currency are reported to FEAT-22.SPEC-002, which creates the Balance Payment and updates the Booking's balance-due status | FEAT-22.SPEC-001 shows the Success state; the Pro's schedule (FEAT-12) and money list (FEAT-28) reflect it once FEAT-22.SPEC-002 and payout routing complete | FEAT-22.SPEC-002 (Balance Capture & Booking Status Update) |
| Charge declined | The capability cannot capture the charge (insufficient funds, card declined, expired card, or another card-level reason) | None -- no Balance Payment is created for a decline | FEAT-22.SPEC-001 shows the Error state with the plain-language decline reason and a "Try a different card" action | FEAT-22.SPEC-001 (Balance Payment) |
| Charge captured but cannot be applied (a cancellation committed first, per FEAT-22.SPEC-004) | The capability reports a successful capture, but FEAT-22.SPEC-002 finds the Booking's cancellation already committed | No Balance Payment is created; this spec immediately requests a refund of the captured amount, using this same integration's refund path | FEAT-22.SPEC-001 shows the Cancelled state ("This booking was just cancelled, so there's nothing to pay. Your card was not charged.") -- the brief moment of capture is never exposed to Riley as a charge that then reverses; the product treats it as never having charged her | FEAT-22.SPEC-002, FEAT-22.SPEC-004 |
| Payout routing confirmed | The captured amount is successfully routed to the Pro's connected Payout Account | Feeds Payout Account's recent-payouts data (owned and displayed by FEAT-28) | No separate feedback to Riley; Talia sees it reflected in her own money list (FEAT-28) on her own schedule | FEAT-28 (Payout Account Connection & Payout Visibility) |
| Refund succeeded | The capability completes a requested balance refund | Balance Payment.state set to Refunded; refund timestamp recorded | Included in the relevant cancellation confirmation content (FEAT-30.SPEC-012 for a Pro-initiated cancellation; the equivalent client-facing cancellation confirmation for a client or automatic cancellation), never a separate standalone notice; Riley sees her balance refunded on her own booking view | FEAT-30.SPEC-007, FEAT-30.SPEC-008, FEAT-09.SPEC-005 |
| Refund could not complete immediately | The capability reports it cannot complete the refund on this attempt (for example, the Pro's payout balance cannot yet cover it) | Balance Payment marked refund-in-progress; a retry is scheduled on a fixed cadence (platform parameter: `refund-retry-interval-hours`, the same cadence FEAT-09.SPEC-006 and FEAT-30.SPEC-011 apply to deposit refunds) | Talia's dashboard shows the same attention flag pattern FEAT-12 already shows for a delayed deposit refund; Riley's booking view shows the balance refund as "in progress," never as failed or silent | FEAT-12 |
| Charge request times out with no result received | The capability does not respond within the product's expected response window | None -- no Balance Payment is created while the outcome is unknown | FEAT-22.SPEC-001 shows its Offline/Degraded resolution; FEAT-22.SPEC-004 governs how a subsequent late response (success or decline) is reconciled once it does arrive | FEAT-22.SPEC-001 (Balance Payment), FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention) |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-22.SPEC-001 (Balance Payment) | The status region shows "Processing payment, do not close this page." for as long as the request is in flight; past a brief threshold it adds "Still working -- this is taking longer than usual." The Pay button and card entry element remain disabled throughout. | The Pay button is disabled before a request is even sent, with the message "We can't take payments right now. Please try again in a few minutes, or pay {Pro's display name} in person." The in-person default remains fully available regardless of this capability's status. | The specific decline reason is shown in plain language (for example, "Your card was declined. Try a different card." or "This card has expired. Try a different card."), never a raw processor code; a "Try a different card" action is offered, per the never-ambiguous-outcome guarantee in FEAT-22.SPEC-004. |
| FEAT-12 (Pro Daily Schedule Dashboard, attention list) -- refund side | No visible change -- the refund request is not user-initiated on this screen, so a slow response produces no waiting state here | If the capability is unreachable when a balance refund is requested, the attention flag reads "A refund for {client name}'s cancelled booking is in progress and will complete automatically." -- mirroring the existing deposit-refund attention pattern (FEAT-30.SPEC-011) | If the capability explicitly rejects the refund request (for example, the payout account is no longer valid), the attention flag reads "A refund for {client name}'s cancelled booking needs attention -- your payout account may need reconnecting." with a link into FEAT-28 |
| Riley's own booking status view (FEAT-06.SPEC-004, Booking Detail via Manage Link) -- refund side | No visible change -- Riley sees no waiting state for a refund that has not yet been requested to fail or succeed | Riley's booking shows "Your balance refund is in progress and will complete automatically." -- never a failure message | Riley's booking shows the same "in progress" wording; she is never shown the capability's rejection reason directly, since the resolution (e.g., reconnecting the payout account) is Talia's action, not hers |

No other screen sends requests to this capability or displays its live results; the Pro's own surfaces (FEAT-12, FEAT-28, FEAT-30) reflect only already-resolved outcomes, which are unaffected by a degradation condition that occurs before resolution.

## Consent and Disclosure

- **Card entry disclosure** -- Directly above the card entry element on FEAT-22.SPEC-001, a persistent line reads: "Your card details go straight to our payment processor -- Chairtime never sees or stores your card number." This is shown every time the screen is reached, not only on first use, since each balance payment is a fresh charge with no card kept on file, consistent with the deposit feature's own practice.
- **What the payout destination receives** -- The disclosure line above also implicitly covers that the payment is destined for the Pro's own connected account; no separate consent prompt is needed for this, since Riley is already paying that specific Pro's booking and the amount and recipient are evident from the booking context itself.
- **What is never shared** -- The Booking's service name, appointment time, Riley's name, phone, email, and any booking note never leave the product through this integration; only the amounts, currency, internal references, and the Pro's payout destination reference are sent, exactly as scoped in Data Exchanged.
- **No new disclosure moment for the refund itself** -- Riley already agreed, at the moment she paid her balance, that the payment-processing capability holds and processes her payment method; reversing that same capture through the same capability requires no additional consent screen, consistent with the existing deposit-refund disclosure practice (FEAT-30.SPEC-011).
- **Pro-facing disclosure of the fee deduction** -- Talia was already told, on viewing her money list after her first captured deposit (FEAT-07.SPEC-005), that Chairtime never takes a cut and the only deduction is the processor's own card fee; this spec's routed balance amounts follow the same disclosure and require no repeated notice.

## Edge Cases

- **The same capture-succeeded event is delivered twice** -- The second delivery reaches FEAT-22.SPEC-002, which finds an existing Balance Payment and takes no further action; this spec itself performs no state changes on receipt, so duplicate delivery has no direct effect here beyond the automation's own idempotency handling.
- **A capture event arrives for a Booking that has since left a payable state for an unrelated reason** -- Handled by FEAT-22.SPEC-002 and FEAT-22.SPEC-004's correctness guarantee, not silently dropped by this spec; if the reason is a committed cancellation, this spec immediately requests a refund of the captured amount rather than leaving it unresolved.
- **A decline and a late capture-succeeded report both arrive for the same attempt (out-of-order delivery)** -- The capability's own outcome for a single charge request is authoritative and final once reported; a genuinely late success after an already-shown decline can only occur if the two reports describe two separate requests (Riley's retry after the decline), each carrying its own idempotency key per FEAT-22.SPEC-004, so they are never conflated into one ambiguous outcome.
- **The capability goes down mid-authorization, after the request was sent but before any result is received** -- No Balance Payment is created while the outcome is unknown; FEAT-22.SPEC-001 shows its Offline/Degraded state, and if the capability's result eventually does arrive (success or decline), FEAT-22.SPEC-004 reconciles it against the Booking's true state rather than assuming failure.
- **A refund-succeeded event arrives for a Balance Payment already marked Refunded** -- The second delivery changes nothing: the Balance Payment stays Refunded with its original refund timestamp, and no duplicate confirmation content fires.
- **Two overlapping retry attempts for the same Balance Payment's refund are both processed** -- Only one results in a Refunded state; the other, whichever resolves second, finds the Balance Payment already Refunded via the idempotency key and is treated as a no-op with no duplicate refund.
- **A refund stays in progress for an extended period because the underlying blocking condition never resolves** -- The retry continues indefinitely on its fixed cadence (platform parameter: `refund-retry-interval-hours`); the Balance Payment is never silently abandoned, and Talia's attention flag persists throughout, per XBR-10's "never dropped" guarantee applied to this balance refund exactly as it applies to a deposit refund.
- **Payout routing confirmation is delayed after a successful capture** -- The capture outcome (Balance Payment Succeeded, Booking fully paid) is not held pending on payout routing; Riley's success state is never delayed by a slow payout leg.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-001 (Balance Payment) | Triggered by (inbound) | The Pay action initiates the authorization/capture request |
| FEAT-22.SPEC-001 (Balance Payment) | Affects (outbound) | Processing, decline, Cancelled, and offline/degraded states surface this spec's reported outcomes and degradation behavior |
| FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules) | Triggered by (inbound) | Eligibility passing is the precondition for this spec ever sending a charge request |
| FEAT-22.SPEC-002 (Balance Capture & Booking Status Update) | Triggers (outbound) | A reported successful capture fires that automation's create-and-update step |
| FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention) | References (inbound) | Supplies the idempotency-key discipline this spec applies to every charge and refund request, and determines when a refund is due |
| FEAT-28 (Payout Account Connection & Payout Visibility) | Affects (outbound) | The routed balance is what that feature's money list displays; refunds draw on the same connected payout account |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | A refund that cannot complete immediately is flagged clearly on the Pro's dashboard until it resolves |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit), FEAT-30.SPEC-008 (Bulk Cancellation Commit) -- within FEAT-30 (Pro Booking Management) | Triggered by (inbound) | A Pro-initiated cancellation on a Booking with a Succeeded Balance Payment initiates this spec's refund request |
| FEAT-09.SPEC-005 (Automatic Deposit Refund) -- within FEAT-09 (Cancellation & No-Show Policy Engine) | Triggered by (inbound) | A client-initiated or automatic cancellation on a Booking with a Succeeded Balance Payment initiates this spec's refund request |
| FEAT-23.SPEC-003 (Tip Payout & Refund Rule) -- within FEAT-23 (Tipping at Checkout, Later) | References (inbound) | Rule spec enforced by this integration (Enforced-By): once FEAT-23 ships, the payout routing passes any tip to the Pro's Payout Account with zero deduction, and the refund request covers the balance plus tip together in full (XBR-23). At v1 no tip exists and the refund equals the balance amount |
| FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution) | References (outbound) | Mirrors this spec's idempotency-key and retry-cadence discipline for the sibling deposit-refund integration, per the Brief's Shared Validation note |

**Cross-feature note for reconciliation:** FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) describes its Balance Payment refund step as running "through the same payment-processing capability path FEAT-30.SPEC-011 uses for other Pro-triggered refunds." This feature's Brief and the External Touchpoints table both assign the outbound balance refund call to this spec (FEAT-22.SPEC-005) instead. This spec is written as the authoritative owner of the balance refund call per that assignment; Pass D reconciliation has aligned FEAT-30.SPEC-007 (and FEAT-30.SPEC-008) to cite FEAT-22.SPEC-005 for the balance leg specifically. Separately, FEAT-09.SPEC-005 (Automatic Deposit Refund) is the inbound trigger for a client-initiated or automatic cancellation: when the cancelled Booking has a Succeeded Balance Payment, FEAT-09.SPEC-005 hands the full balance refund to this spec (XBR-23), alongside its own deposit refund. FEAT-30.SPEC-007 and FEAT-30.SPEC-008 are the callers for a Pro-initiated cancellation and bulk cancellation, so all three callers converge on this spec as the single owner of the outbound balance refund call.

## Analytics and Success Signals

- **balance_charge_requested** (booking reference, balance amount, currency) -- supports success-metrics.md: "Payout Transparency"
- **balance_charge_captured** (booking reference) -- supports success-metrics.md: "Payout Transparency"
- **balance_charge_declined** (booking reference, decline reason category) -- supports success-metrics.md: "Payout Transparency"
- **balance_charge_reversed_after_cancellation** (booking reference) -- N/A -- no Stage 2 metric measures this exact race outcome directly; retained so the never-charge-for-a-cancelled-booking guarantee's exercise rate is observable to the Pro's activity record (FEAT-16), not to a success metric
- **balance_payout_routing_confirmed** (booking reference) -- supports success-metrics.md: "Payout Transparency"
- **balance_refund_requested** (booking reference, source: pro_cancellation / client_cancellation / automatic_cancellation) -- supports success-metrics.md: "Automatic Refund Correctness"
- **balance_refund_outcome_received** (outcome: succeeded / could-not-complete) -- supports success-metrics.md: "Automatic Refund Correctness"

## Acceptance Criteria

**FEAT-22.SPEC-005-AC-01:** Given Riley's Booking has passed FEAT-22.SPEC-003's eligibility check and FEAT-22.SPEC-004's contention check, when she taps Pay, then a charge request for exactly the locked balance amount and currency is sent to the payment-processing capability, carrying an idempotency key and the Booking reference.

**FEAT-22.SPEC-005-AC-02:** Given the capability successfully captures Riley's charge, when it reports the outcome, then this spec reports the captured amount and currency to FEAT-22.SPEC-002 without creating the Balance Payment itself.

**FEAT-22.SPEC-005-AC-03:** Given the capability declines Riley's card, when it reports the decline, then FEAT-22.SPEC-001 shows the plain-language decline reason and a "Try a different card" action, and no Balance Payment is created.

**FEAT-22.SPEC-005-AC-04:** Given a captured balance charge, when payout routing completes, then the amount is routed to Talia's connected Payout Account with a Chairtime fee of zero.

**FEAT-22.SPEC-005-AC-05:** Given Riley is on the Balance Payment screen, when she reaches the card entry element, then she sees "Your card details go straight to our payment processor -- Chairtime never sees or stores your card number." above it.

**FEAT-22.SPEC-005-AC-06:** Given a charge request is in flight, when more than the product's brief processing threshold passes without a result, then the status region adds "Still working -- this is taking longer than usual." while the Pay button remains disabled.

**FEAT-22.SPEC-005-AC-07:** Given the payment-processing capability is unavailable when Riley reaches the Pay step, when she views the screen, then the Pay button is disabled with "We can't take payments right now. Please try again in a few minutes, or pay {Pro's display name} in person."

**FEAT-22.SPEC-005-AC-08:** Given the capability rejects a charge with a specific reason, when the rejection is reported, then Riley sees that specific plain-language reason, never a raw processor code.

**FEAT-22.SPEC-005-AC-09:** Given a capture-succeeded event is delivered twice for the same Booking, when the second delivery reaches FEAT-22.SPEC-002, then no second Balance Payment is created.

**FEAT-22.SPEC-005-AC-10:** Given the capability reports a captured charge but the Booking's cancellation already committed, when FEAT-22.SPEC-002 checks the Booking's state, then no Balance Payment is created and this spec immediately requests a refund of the captured amount.

**FEAT-22.SPEC-005-AC-11:** Given a Succeeded Balance Payment exists on a Booking that is cancelled by Talia (FEAT-30.SPEC-007 or FEAT-30.SPEC-008), by Riley, or automatically (FEAT-09.SPEC-005), when the cancelling spec hands off the refund, then this spec requests the refund and, on the capability's confirmation, sets the Balance Payment to Refunded with a refund timestamp.

**FEAT-22.SPEC-005-AC-12:** Given a refund request is sent to the capability, when the capability reports it cannot complete on this attempt, then the Balance Payment is marked refund-in-progress, Talia's dashboard shows the attention flag, and Riley's booking shows "in progress," never a failure.

**FEAT-22.SPEC-005-AC-13:** Given a refund that could not complete immediately, when the next scheduled retry runs (platform parameter: `refund-retry-interval-hours` after the previous attempt), then a new refund request is submitted carrying the same attempt's idempotency key.

**FEAT-22.SPEC-005-AC-14:** Given a Balance Payment is already Refunded, when the same refund-succeeded event is delivered again, then nothing changes and no duplicate confirmation content fires.

**FEAT-22.SPEC-005-AC-15:** Given two overlapping retry attempts for the same Balance Payment's refund both resolve, then only one results in a Refunded status, and the other is treated as a no-op with no duplicate refund.

**FEAT-22.SPEC-005-AC-16:** Given a refund's blocking condition never resolves, when successive scheduled retries continue to fail, then the Balance Payment remains refund-in-progress indefinitely rather than moving to a terminal failure state, and Talia's attention flag persists throughout.

**FEAT-22.SPEC-005-AC-17:** Given this integration requests a charge or a refund, when the request is composed, then only the amount, currency, an internal reference, the payout account reference, and an idempotency key are sent -- Booking details and Client contact fields are never included.

**FEAT-22.SPEC-005-AC-18:** Given a capture outcome is confirmed, when Riley's Success state is shown, then it is never delayed waiting for the separate payout-routing confirmation to Talia's account.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 7 | 7 |
| Degradation Paths | 9 (3 screens x 3 conditions) | 9 |
| Consent and Disclosure | 5 | 5 |
| Edge Cases | 8 | 8 |

