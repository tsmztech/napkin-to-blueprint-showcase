# FEAT-07 — Deposit Payment at Booking

This chapter covers Deposit Payment at Booking (FEAT-07), a Core-tier feature. It carries 5 specifications carrying 79 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-07.SPEC-001 | Deposit Payment | screen | 19 |
| FEAT-07.SPEC-002 | Deposit Capture & Booking Confirmation | automation | 14 |
| FEAT-07.SPEC-003 | Deposit Amount & Eligibility Rules | logic-rule | 16 |
| FEAT-07.SPEC-004 | Payment Outcome Consistency & Idempotency | logic-rule | 14 |
| FEAT-07.SPEC-005 | Card Deposit Charge & Payout Routing | integration | 16 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Deposit Payment

## Overview

**Name:** Deposit Payment
**ID:** FEAT-07.SPEC-001
**Type:** Screen
**Purpose:** Riley enters card details for the exact, already-computed deposit and sees processing, decline/retry, and success states before handing off to the booking confirmation.
**Parent Feature:** FEAT-07 -- Deposit Payment at Booking

## Scope and Non-Goals

**In Scope:**
- Displaying the locked deposit amount and the booking it belongs to
- Collecting card details through the payment-processing capability's own entry element and submitting them for authorization and capture
- Processing, decline/retry, and success states for a single payment attempt and any immediate retries
- Returning the client to the live slot list when a slot hold expires before a declined payment is retried

**Non-Goals:**
- Computing or altering the deposit amount -- governed exclusively by FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules); this screen only displays the value SPEC-003 locked
- Guaranteeing the charge is never duplicated and the booking state is never left ambiguous -- governed by FEAT-07.SPEC-004 (Payment Outcome Consistency & Idempotency); this screen only reflects the outcome that spec resolves
- Acknowledging the cancellation and deposit policy -- excluded per FEAT-05's Brief: policy acknowledgment is FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout), the step immediately before this screen; this screen assumes acknowledgment already happened
- Displaying the full booking confirmation (studio address, balance due, cancellation cut-off, manage link, calendar option) -- owned by FEAT-05.SPEC-005 (Booking Confirmation); this screen only shows a brief success state before handing off

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Riley ticks the policy agreement checkbox and taps "Acknowledge & continue" (FEAT-05.SPEC-004's Navigation Out row to this screen) | The in-progress Booking (service, price, appointment time, the locked deposit amount computed by FEAT-07.SPEC-003, the active checkout hold) |

This screen is the single owner of card entry in the booking flow: FEAT-05.SPEC-004 keeps only the policy acknowledgment and hands off here, and does not describe or collect card details itself. On success this screen returns Riley to FEAT-05.SPEC-005 (Booking Confirmation). This is the feature's only entry point. Payment is always tied to an in-progress booking; there is no standalone or directly-linked way to reach this screen (per product-features.md's States field for this feature).

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full screen, scoped to her own in-progress booking only | Enter card details and submit for her own booking only | -- |
| The Pro (Talia) | No | No | N/A -- this screen belongs to the client-facing booking flow and never appears on any of the Pro's own surfaces (FEAT-12, FEAT-27, FEAT-30); if Talia opens her own public booking link to preview it, she reaches this screen only in the Client capacity, exactly as any visitor would, not as a Pro-authorized view. The Pro's own view of the resulting deposit status lives on FEAT-12 (Pro Daily Schedule Dashboard) and FEAT-30 (Pro Booking Management), never here. |
| Platform Operator (Support) | No | No | N/A -- Support's view-only access to transaction status is on FEAT-16 (Booking & Payment Activity Record) and FEAT-19 (Platform Support Read-Only Access); Support never opens this in-flow payment screen, per the Access Matrix's Payouts row ("View: status and money list only"). |
| Unauthenticated (no in-progress booking session) | No | No | The screen has nothing to render without an in-progress Booking carried from FEAT-05.SPEC-004; a direct or stale attempt to open it shows "Start a new booking" and returns to FEAT-05.SPEC-001 (Public Booking Page). |
| Expired session (checkout hold expired) | Partial -- the screen remains visible but the pay action is disabled | No | Handled explicitly as the "Hold Expired" state below: a plain message and a single action returning Riley to the live slot list, per FEAT-03's ownership of hold expiry (XBR-02). |

This product has no sign-in concept for the Client (BRIEF.md, Target Users & Roles: clients "must not face a signup wall"); access to this screen is scoped entirely to possession of the active booking session created at FEAT-05.SPEC-004, not to a login.

## Layout and Content

**Header:** A back arrow (returns Riley to FEAT-05.SPEC-004, Policy Acknowledgment & Deposit Checkout, preserving her acknowledgment) and the screen title "Pay your deposit." Below the title, a one-line booking summary: service name, the Pro's display name, and the appointment date and time in the Pro's timezone (labeled, per XBR-25).

**Body, in order, top to bottom:**
- An amount summary block: "Deposit due now: {locked deposit amount}" as the primary figure, with "Balance due at your appointment: {price_agreed minus deposit_amount}" shown beneath it in smaller text. Both figures are read directly from the Booking record locked by FEAT-07.SPEC-003; this screen performs no computation of its own.
- The payment-processing capability's own card-detail entry element -- the fields Riley fills (card number, expiry, security code, and postal code where the capability requires it) are entered directly into that capability's element and never pass through the product's own code, per SC-11.
- A single "Pay {deposit amount} deposit" button, below the card entry element, spanning the width of the body.
- A status/error region directly below the button, empty in the default state, populated in the Capability Unavailable, Processing, Error, and Hold Expired states described below.

**Footer:** A single line of plain text: "Do not close this page while your payment is processing." -- shown persistently, not only during the Processing state, so Riley is prepared before she taps Pay.

### Responsive Behavior

- **Compact size class (phone, the product's primary usage per BRIEF.md's Scale & Non-Functional Expectations):** Single-column layout exactly as described above, full width, footer text pinned below the status region.
- **Medium size class and above:** The body content (amount summary, card element, Pay button, status region) is capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.
- **Booking summary line:** Wraps to two lines on the narrowest supported widths rather than truncating -- the service name and appointment time are never cut off.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Screen closes; Riley's prior acknowledgment is preserved | Standard back transition |
| Card entry element | Type | Captures card details directly with the payment-processing capability | Element shows entered data per the capability's own input treatment | Standard input focus/validation state from the capability's element |
| Pay button | Tap | 1. Confirm the Booking is still Pending Payment and the checkout hold is still active (the pre-charge hold re-validation run by FEAT-05.SPEC-006). 2. Request eligibility confirmation from FEAT-07.SPEC-003. 3. If eligible, submit the charge through FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing). | Button enters the Processing state (loading indicator, disabled) | Status region shows "Processing payment, do not close this page." |
| Pay button (while processing) | Tap | No action -- ignored while a request is already in flight (double-submit prevention, governed by FEAT-07.SPEC-004) | None | Button remains in Processing state |
| Pay button (capability unavailable) | Tap | No action -- the button is disabled before any charge request is sent, so a tap cannot occur | None | Status region shows "We can't take payments right now. Please try again in a few minutes -- your held time is still counting down." |
| "Try a different card" (shown only in the Error state) | Tap | Clears the card entry element and re-enables the Pay button | Screen returns to the default entry state with the same locked deposit amount shown | Card entry element is cleared and refocused |
| "Back to available times" (shown only in the Hold Expired state) | Tap | Navigate to FEAT-05.SPEC-002 (Slot Selection), the live slot list | Screen closes; Booking's held slot has already been released by FEAT-03 | Plain message "Your held time expired. Pick a new time to continue." is carried into the slot list as a one-time banner |

### Accessibility Notes

- **Focus order:** Back arrow -> booking summary (read-only, not a focus stop) -> card entry element's own internal focus order -> Pay button -> status/error region (receives focus only when it updates).
- **Dynamic announcements:** When the status region updates (the capability-unavailable message appears, Processing begins, the extended-wait text is added, a decline message appears, success is reached, or the hold-expired message appears), the new text is announced to assistive technology and the region is programmatically associated as the outcome of the Pay action.
- **Success feedback:** On success, focus moves to the success message before the automatic hand-off to FEAT-05.SPEC-005 occurs.
- **Keyboard alternatives:** Every action on this screen (Pay, Try a different card, Back to available times, back arrow) is reachable by keyboard; there are no pointer-only gestures. The card entry element's own keyboard accessibility is owned by the payment-processing capability.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Default (entry) | Amount summary and empty card entry element shown; Pay button enabled once the element reports valid input | Screen first opens from FEAT-05.SPEC-004 | Riley taps Pay |
| Capability Unavailable (pre-submission) | Pay button is disabled before any request is sent; status region reads "We can't take payments right now. Please try again in a few minutes -- your held time is still counting down." Card entry element remains available for Riley to fill in while she waits. | The payment-processing capability reports itself down when the screen loads or at any point before Riley taps Pay, per FEAT-07.SPEC-005's Degradation Behavior (Capability Down) | The capability becomes available again, re-enabling the Pay button once the card element reports valid input; or the checkout hold expires, entering Hold Expired |
| Processing | Pay button shows a loading indicator and is disabled; status region reads "Processing payment, do not close this page." Past a brief processing threshold without a result, the status region adds "Still working -- this is taking longer than usual." while the Pay button and card entry element remain disabled, per FEAT-07.SPEC-005's Degradation Behavior (Capability Slow). | Riley taps Pay and the charge request is sent | FEAT-07.SPEC-005 reports a result (success or decline), or the connection drops |
| Success | Status region shows a brief confirmation message ("Deposit received -- confirming your booking...") before automatic hand-off | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) reports the Booking is now Confirmed | Automatic navigation to FEAT-05.SPEC-005 (Booking Confirmation), within a couple of seconds |
| Error (declined) | Status region shows the plain-language decline reason (translated by FEAT-07.SPEC-005) and a "Try a different card" action; card entry element is cleared | FEAT-07.SPEC-005 reports a decline | Riley taps "Try a different card," returning to Default with the same locked deposit amount and the same held slot (still within its window) |
| Hold Expired | Status region shows "Your held time expired. Pick a new time to continue." with the "Back to available times" action; Pay button is disabled | The checkout hold governed by FEAT-03 (XBR-02) expires while Riley is on this screen, whether before or after a decline | Riley taps "Back to available times" |
| Offline/Degraded | If connectivity is lost before Riley taps Pay, the Pay button is disabled with the message "You're offline -- reconnect to pay your deposit." If connectivity drops mid-processing, this is treated as a payment failure per FEAT-07.SPEC-004: the status region shows "We couldn't confirm your payment. Check your connection; if you were charged, your booking will show as confirmed the moment you reconnect -- you will never be charged twice." with a "Try again" action | Connectivity is lost while this screen is open, at any point before or during a charge attempt | Connectivity is restored: the Default state resumes if no charge was in flight; if a charge was in flight, FEAT-07.SPEC-004's resolution determines whether Success or Error is shown |

## Validation Rules

Validation of the deposit amount and the preconditions for attempting a charge (Booking still Pending Payment, currency match, one-charge-per-booking, active payout account) is governed by FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules). Card-detail format validation is owned entirely by the payment-processing capability's own entry element; this screen never validates card fields itself. This screen enforces no field-level rules of its own beyond disabling Pay until the card element reports it has valid input.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | FEAT-05 (Public Booking Page & Booking Flow) |
| Successful payment (Booking reaches Confirmed) | FEAT-05.SPEC-005 (Booking Confirmation) | FEAT-05 (Public Booking Page & Booking Flow) |
| Checkout hold expires (with or without a prior decline) | FEAT-05.SPEC-002 (Slot Selection) | FEAT-05 (Public Booking Page & Booking Flow) -- hold-expiry ownership is FEAT-03's per XBR-02 |

## Data Model

**Creates:** None directly -- the Deposit Transaction record is created by FEAT-07.SPEC-002 on a successful charge, not by this screen.
**Reads:** Booking -- service, start_time, price_agreed, deposit_amount (as locked by FEAT-07.SPEC-003), state. Pro Account -- display_name, timezone (for the booking summary line, per XBR-25).
**Updates:** None directly -- the Booking's Pending Payment -> Confirmed transition is written by FEAT-07.SPEC-002, never by this screen.
**Deletes:** None.

## Business Rules

- The deposit amount shown is always the value FEAT-07.SPEC-003 computed and locked; this screen never performs its own computation and never lets Riley edit it (per the Brief's Shared UI Patterns).
- Every payment attempt, and every retry, is subject to FEAT-07.SPEC-003's eligibility preconditions before a charge is requested.
- FEAT-07.SPEC-004 governs the guarantee that this screen never shows an ambiguous outcome: every attempt resolves to exactly one of Success or Error, even across a dropped connection.
- Retrying after a decline never requires Riley to re-enter service, time, name, phone, or policy acknowledgment (per the Brief's Side-Effect Inventory) -- only the card entry element is reset.
- XBR-02: the checkout hold on Riley's selected slot is time-limited; its expiry, while she is on this screen, is owned by FEAT-03 and surfaces here only as the Hold Expired state.

## Edge Cases

- **Riley taps Pay twice in rapid succession** -- The second tap is ignored while the first request is in flight (Pay button disabled in the Processing state, per FEAT-07.SPEC-004's double-charge prevention).
- **Riley navigates back and returns to this screen** -- The screen re-reads the current Booking state; if it is already Confirmed (a prior attempt succeeded but she navigated away before the hand-off completed), she is shown the Success state immediately rather than a fresh entry form, per FEAT-07.SPEC-004.
- **Concurrent action on the underlying Booking** -- The dependency map's Contention note for Booking notes it is "High" contention, but the actors who could concurrently act on it (the Pro cancelling, an automation expiring the hold) are the only other writers while a Booking is Pending Payment. If the Pro-side or hold-expiry actor commits a transition first (for example, the checkout hold expires the instant before Riley's charge is submitted), FEAT-07.SPEC-003's eligibility check catches it and the screen shows the Hold Expired state rather than attempting a charge against a booking that is no longer eligible -- reject-with-refresh, consistent with the dependency map's Contention note for Booking.
- **Connection drops mid-payment** -- Handled by the Offline/Degraded state above; the outcome is resolved by FEAT-07.SPEC-004 and never results in an ambiguous charge.
- **Riley's card is declined and the checkout hold then expires before she retries** -- The screen transitions from Error directly to Hold Expired; the "Try a different card" action is replaced by "Back to available times."
- **The payment-processing capability itself is slow or unavailable** -- Covered by FEAT-07.SPEC-005's Degradation Behavior for this screen: a slow in-flight request extends the Processing state with "Still working -- this is taking longer than usual." (FEAT-07.SPEC-005-AC-06), and a capability that is down before submission puts the screen in the Capability Unavailable state with the Pay button disabled and "We can't take payments right now. Please try again in a few minutes -- your held time is still counting down." (FEAT-07.SPEC-005-AC-07); a Rejects outcome is the existing Error (declined) state with the specific plain-language reason FEAT-07.SPEC-005 reports.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-004 (Policy Acknowledgment & Deposit Checkout) | Navigation (inbound) | Riley arrives here after ticking the policy agreement and tapping "Acknowledge & continue"; FEAT-05.SPEC-004 holds the acknowledgment, this screen owns card entry |
| FEAT-05.SPEC-005 (Booking Confirmation) | Navigation (outbound) | Successful payment (on success this screen returns to FEAT-05.SPEC-005) hands off to the full confirmation screen |
| FEAT-05.SPEC-002 (Slot Selection) | Navigation (outbound) | A checkout hold expiring on this screen returns Riley to the live slot list |
| FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules) | References (inbound) | Supplies the locked deposit amount and the eligibility preconditions checked before every charge attempt |
| FEAT-07.SPEC-004 (Payment Outcome Consistency & Idempotency) | References (inbound) | Governs double-submit prevention, connection-drop resolution, and the never-ambiguous-outcome guarantee this screen reflects |
| FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing) | Triggers (outbound) | The Pay action requests authorization and capture through this integration; its decline translation populates the Error state |
| FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Triggered by (inbound) | This automation's successful outcome is what moves the screen into the Success state |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| deposit_payment_attempted | booking reference, deposit amount, is_retry (true/false) | Riley taps Pay | supports success-metrics.md: "Deposit Capture Rate" |
| deposit_payment_succeeded | booking reference, deposit amount | FEAT-07.SPEC-002 confirms the Booking is Confirmed | supports success-metrics.md: "Deposit Capture Rate" |
| deposit_payment_failed | booking reference, decline reason category | FEAT-07.SPEC-005 reports a decline | supports success-metrics.md: "Deposit Capture Rate" |
| deposit_payment_hold_expired | booking reference, had_prior_decline (true/false) | The checkout hold expires while this screen is open | supports success-metrics.md: "Deposit Capture Rate" (a hold expiry is neither a clean success nor a clean decline until Riley acts again -- this event tracks it separately so the metric's "clear success or clear, actionable decline" target is measured against attempts that actually reached a resolution) |

## Acceptance Criteria

**FEAT-07.SPEC-001-AC-01:** Given Riley is on the Deposit Payment screen with a locked deposit amount of {X}, when the screen loads, then she sees "Deposit due now: {X}" and "Balance due at your appointment: {price minus X}" with no way to edit either figure.

**FEAT-07.SPEC-001-AC-02:** Given Riley has entered valid card details, when she taps "Pay {X} deposit," then the button enters the Processing state and the status region reads "Processing payment, do not close this page."

**FEAT-07.SPEC-001-AC-03:** Given Riley's payment is in the Processing state, when she taps the Pay button again, then nothing happens and the button remains in the Processing state.

**FEAT-07.SPEC-001-AC-04:** Given Riley's card is declined, when FEAT-07.SPEC-005 reports the decline, then the status region shows the plain-language decline reason and a "Try a different card" action, and her checkout hold remains active.

**FEAT-07.SPEC-001-AC-05:** Given Riley sees a decline message, when she taps "Try a different card," then the card entry element clears and refocuses, and the same locked deposit amount is still shown -- with no need to re-enter her name, phone, or policy acknowledgment.

**FEAT-07.SPEC-001-AC-06:** Given Riley's deposit charge succeeds, when FEAT-07.SPEC-002 confirms the Booking is Confirmed, then the status region shows "Deposit received -- confirming your booking..." and she is automatically navigated to FEAT-05.SPEC-005 (Booking Confirmation).

**FEAT-07.SPEC-001-AC-07:** Given Riley's checkout hold expires while she is viewing this screen and no decline has occurred, when the expiry is reported, then the screen enters the Hold Expired state showing "Your held time expired. Pick a new time to continue." and the Pay button is disabled.

**FEAT-07.SPEC-001-AC-08:** Given Riley sees the Hold Expired state, when she taps "Back to available times," then she is navigated to FEAT-05.SPEC-002 (Slot Selection) with a one-time banner carrying the "Your held time expired" message.

**FEAT-07.SPEC-001-AC-09:** Given Riley was declined and her checkout hold then expires before she retries, when the expiry is reported, then the screen transitions from the Error state to the Hold Expired state and the "Try a different card" action is replaced by "Back to available times."

**FEAT-07.SPEC-001-AC-10:** Given Riley loses connectivity before tapping Pay, when she attempts to tap Pay, then the button is disabled with the message "You're offline -- reconnect to pay your deposit."

**FEAT-07.SPEC-001-AC-11:** Given Riley's connection drops after she taps Pay but before a result is confirmed, when the drop is detected, then the status region shows "We couldn't confirm your payment. Check your connection; if you were charged, your booking will show as confirmed the moment you reconnect -- you will never be charged twice." with a "Try again" action.

**FEAT-07.SPEC-001-AC-12:** Given Riley reconnects after a dropped connection and her charge had in fact succeeded, when the screen re-checks the Booking state, then it shows the Success state directly rather than allowing a second charge attempt.

**FEAT-07.SPEC-001-AC-13:** Given Riley navigates back to the Policy Acknowledgment screen from this screen, when she does so, then her prior acknowledgment on FEAT-05.SPEC-004 is preserved.

**FEAT-07.SPEC-001-AC-14:** Given Riley navigates away from this screen after a successful charge but before the automatic hand-off completes, when she returns to this screen, then she is shown the Success state immediately rather than a fresh entry form.

**FEAT-07.SPEC-001-AC-15:** Given a visitor reaches this screen's address without an in-progress booking session, when the screen attempts to load, then it shows "Start a new booking" and returns to FEAT-05.SPEC-001 (Public Booking Page).

**FEAT-07.SPEC-001-AC-16:** Given Talia (the Pro) opens her own public booking link to preview it, when she reaches this screen, then she experiences it exactly as any client would, in the Client capacity, with no Pro-specific controls or data shown.

**FEAT-07.SPEC-001-AC-17:** Given Riley's slot becomes ineligible between load and her tapping Pay (for example, the checkout hold expired the instant before submission), when FEAT-07.SPEC-003's eligibility check runs, then the screen shows the Hold Expired state rather than attempting a charge.

**FEAT-07.SPEC-001-AC-18:** Given the payment-processing capability is unavailable when Riley reaches this screen or at any point before she taps Pay, when she views the screen, then the Pay button is disabled with "We can't take payments right now. Please try again in a few minutes -- your held time is still counting down." and her checkout hold continues counting down normally.

**FEAT-07.SPEC-001-AC-19:** Given Riley's charge request is in flight, when more than the product's brief processing threshold passes without a result, then the status region adds "Still working -- this is taking longer than usual." while the Pay button and card entry element remain disabled.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 7 (default, capability unavailable, processing, success, error, hold expired, offline/degraded) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |



# Automation Spec: Deposit Capture & Booking Confirmation

## Overview

**Name:** Deposit Capture & Booking Confirmation
**ID:** FEAT-07.SPEC-002
**Type:** Automation
**Purpose:** On a successful card charge, the system creates the Deposit Transaction record and atomically flips the Booking from Pending Payment to Confirmed, enforcing exactly one charge per booking.
**Parent Feature:** FEAT-07 -- Deposit Payment at Booking

## Scope and Non-Goals

**In Scope:**
- Creating the Deposit Transaction record the instant a charge is reported captured
- Atomically transitioning the Booking from Pending Payment to Confirmed as one step with that creation
- The one-charge-per-booking guarantee at the point of capture (working with FEAT-07.SPEC-003's precondition check and FEAT-07.SPEC-004's idempotency guarantee)
- Feeding the confirmed booking into the confirmation message and the Pro's schedule and activity surfaces

**Non-Goals:**
- Requesting authorization and capture from the payment-processing capability -- owned by FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing); this automation only reacts to that spec's reported outcome
- Computing the deposit amount -- owned by FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules); this automation only persists the amount already locked on the Booking
- Guaranteeing the charge is never duplicated across a dropped connection or an interrupted confirmation step -- owned by FEAT-07.SPEC-004 (Payment Outcome Consistency & Idempotency); this automation implements the create-and-confirm step that spec's guarantee wraps around
- Composing or sending the client's confirmation message -- excluded per product-features.md's Communications field: a successful deposit "feeds the confirmation message in Automated Booking Messaging (FEAT-08)," whose own Notification spec owns content and delivery

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Card charge reported captured | FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing) | Fires when the payment-processing capability reports a successful capture for an attempt that passed FEAT-07.SPEC-003's eligibility check | Booking reference, captured amount, currency, processor_fee, capture timestamp |

This automation has exactly one trigger. It is fired once per successful capture event; a capture reported for a Booking that is not Pending Payment (per the idempotency guarantee in FEAT-07.SPEC-004) does not reach this automation as a new run -- see Edge Cases.

## Processing Logic

1. Receive the capture event from FEAT-07.SPEC-005: the Booking reference, captured amount, currency, processor_fee, and capture timestamp.
2. Confirm the referenced Booking's current state is Pending Payment and that no Deposit Transaction already exists for it (the one-charge-per-booking guarantee, enforced jointly with FEAT-07.SPEC-004). If a Deposit Transaction already exists for this Booking, treat this as a duplicate delivery of the same capture event and take no further action (see Edge Cases).
3. Create the Deposit Transaction record: amount and currency from the capture event, status set directly to Captured, processor_fee recorded, outcome_reason set to "deposit captured at booking," and the capture timestamp recorded.
4. In the same atomic step as creating the Deposit Transaction, transition the Booking's state from Pending Payment to Confirmed.
5. Signal FEAT-07.SPEC-001 (Deposit Payment) that the Booking is now Confirmed, so the screen can show its Success state and hand off to FEAT-05.SPEC-005 (Booking Confirmation).
6. Hand the confirmed Booking to FEAT-08 (Automated Booking Messaging) so the client's confirmation message can be composed and sent -- this automation does not compose or send that message itself.
7. Signal FEAT-16 (Booking & Payment Activity Record) that a deposit_payment_succeeded event occurred, for the append-only activity record.
8. Signal FEAT-12 (Pro Daily Schedule Dashboard) and FEAT-30 (Pro Booking Management) that the Booking's paid/unpaid status is now Confirmed/paid, so the Pro's schedule reflects it without a manual refresh.
9. Fire FEAT-25.SPEC-004 (Historical Aggregate Maintenance) with the Booking's Pro Account reference, service reference, and start_time, so the day's booking-count and per-service aggregates increment exactly once for this Booking-confirmed transition; a failure of that aggregate update never blocks or reverses this automation.
10. Fire FEAT-04.SPEC-005 (Booking-to-Calendar Sync) with the Booking reference (service, start_time, duration), so the confirmed Booking is written to the Pro's connected calendar; a failure of that write never blocks or reverses this automation.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Capture confirmed | The referenced Booking is Pending Payment and has no existing Deposit Transaction | Deposit Transaction created (status: Captured); Booking transitions Pending Payment -> Confirmed | Riley sees the Success state and hand-off on FEAT-07.SPEC-001, then the full confirmation on FEAT-05.SPEC-005; Talia sees the booking as paid on her schedule | FEAT-07.SPEC-001, FEAT-05.SPEC-005, FEAT-08 (FEAT-08.SPEC-001, FEAT-08.SPEC-005, FEAT-08.SPEC-007), FEAT-12, FEAT-16 (FEAT-16.SPEC-002), FEAT-25 (FEAT-25.SPEC-004), FEAT-04 (FEAT-04.SPEC-005), FEAT-30 (FEAT-30.SPEC-010) |
| Duplicate capture event ignored | A Deposit Transaction already exists for the referenced Booking (the capture event was delivered more than once, or arrived after the Booking was already confirmed by an earlier delivery) | None -- the existing Deposit Transaction and Confirmed state are left unchanged | None -- Riley already saw the Success state from the first delivery; no second confirmation message fires | None beyond the existing state |
| Referenced Booking not eligible | The referenced Booking is not Pending Payment at the moment this automation runs (for example, it was already cancelled by an automation-expired hold, or a different terminal path already resolved it) | None -- no Deposit Transaction is created against an ineligible Booking | The capture is reported back to FEAT-07.SPEC-005 as unappliable, which FEAT-07.SPEC-004 resolves per its correctness guarantee (never silently drop a successful charge) | FEAT-07.SPEC-004, FEAT-07.SPEC-005 |
| Automation failure (processing error after capture confirmed) | The capture event is received but this automation cannot complete the create-and-confirm step (for example, an internal fault interrupts step 3 or 4) | No partial state is left visible: either both the Deposit Transaction and the Confirmed transition are committed together, or neither is | Riley's screen (FEAT-07.SPEC-001) shows the Offline/Degraded resolution defined by FEAT-07.SPEC-004 -- she is never shown an ambiguous or double-charged state; the automation retries the create-and-confirm step automatically | FEAT-07.SPEC-001, FEAT-07.SPEC-004 |

## Data Model

**Reads:** Booking -- state (must be Pending Payment), service, price_agreed, deposit_amount (to confirm the captured amount matches the locked computation), client reference. Deposit Transaction -- read internally to check for an existing record before creating a new one (the one-charge-per-booking guarantee).
**Creates:** Deposit Transaction -- amount, currency, status (set to Captured), processor_fee, outcome_reason, timestamps. Exactly one per Booking, ever, from this automation.
**Updates:** Booking -- state, from Pending Payment to Confirmed. This is the only Booking field this automation writes.
**Deletes:** None.

## Business Rules

- The Deposit Transaction creation and the Booking's Pending Payment -> Confirmed transition happen as a single atomic step -- one can never persist without the other (XBR-05).
- Exactly one Deposit Transaction is ever created per Booking; a second capture event for the same Booking is a duplicate delivery, never a second charge (FEAT-07.SPEC-004).
- The amount and currency written to the Deposit Transaction are exactly the values FEAT-07.SPEC-005 reports as captured, which must equal the amount FEAT-07.SPEC-003 locked on the Booking -- this automation does not recompute or adjust the amount.
- This automation is the sole writer of the Booking's Pending Payment -> Confirmed transition; no other feature ever performs this specific transition (per the Entity-Lifecycle Coverage Matrix).
- Confirmation is instant from the client's perspective: the Booking flips to Confirmed the moment capture is reported, not on a delay or batch cycle (product-features.md, Primary Flows: "the booking flips from pending to confirmed instantly").

## Edge Cases

- **Duplicate capture event for the same Booking** -- The second (and any subsequent) delivery finds an existing Deposit Transaction and takes no action; the Booking remains Confirmed with its original capture timestamp. No duplicate confirmation message fires.
- **Capture event arrives for a Booking no longer Pending Payment** -- If the Booking's state changed for a reason other than this automation (for example, an expired-hold automation already moved it out of Pending Payment before the capture event arrived), no Deposit Transaction is created against it; the outcome is escalated to FEAT-07.SPEC-004 as a payment succeeded against an ineligible booking, which that spec's correctness guarantee resolves rather than silently dropping the money.
- **Confirmation-step processing fails after the charge succeeded** -- The create-and-confirm step either fully commits or does not commit at all; a partial state (Deposit Transaction created but Booking still Pending Payment, or the reverse) never exists. If the step has not yet committed, it is retried automatically; Riley's screen shows the safe-retry behavior FEAT-07.SPEC-004 defines rather than a false failure.
- **Concurrent trigger firing (two capture events for two different Bookings at effectively the same time)** -- Each runs independently against its own Booking and Deposit Transaction; there is no shared state between two different Bookings' captures, so neither run affects the other.
- **Trigger fires while a previous run for the same Booking is still in flight** -- Cannot occur under normal operation, because FEAT-07.SPEC-004's one-charge-per-booking guarantee ensures a second charge attempt for the same Pending Payment Booking is never authorized while the first is being captured; if a second capture event nonetheless arrives before the first run has finished committing, it is treated exactly as the duplicate-capture-event case once the first run's Deposit Transaction becomes visible.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing) | Triggered by (inbound) | A reported successful capture fires this automation |
| FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules) | References (inbound) | Confirms the captured amount matches the locked computation and that the eligibility precondition held |
| FEAT-07.SPEC-004 (Payment Outcome Consistency & Idempotency) | References (inbound) | Governs the atomicity guarantee this automation implements and the resolution when a capture cannot be applied |
| FEAT-07.SPEC-001 (Deposit Payment) | Affects (outbound) | The Confirmed transition is what moves that screen into its Success state |
| FEAT-05.SPEC-005 (Booking Confirmation) | Affects (outbound) | The confirmed Booking is what this screen displays after hand-off |
| FEAT-08.SPEC-001 (Booking Confirmation Message), FEAT-08.SPEC-005 (Pro Booking Activity Notification), FEAT-08.SPEC-007 (Reminder Scheduling & Timing Window Enforcement) -- within FEAT-08 (Automated Booking Messaging) | Affects (outbound) | The confirmed Booking feeds that feature's own confirmation-message Notification spec |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | deposit_payment_succeeded is written to the append-only activity record |
| FEAT-25.SPEC-004 -- within FEAT-25 (Booking & Revenue Insights) | Triggers (outbound) | The Pending Payment -> Confirmed transition fires the booking-count and per-service aggregate increment (step 9) |
| FEAT-04.SPEC-005 -- within FEAT-04 (Two-Way Calendar Sync) | Triggers (outbound) | The Confirmed transition fires the write of the booking to the Pro's connected calendar (step 10) |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | The Booking's paid/unpaid status becomes visible on Talia's schedule |
| FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) -- within FEAT-30 (Pro Booking Management) | Affects (outbound) | The Booking's paid/unpaid status becomes visible on the Pro's booking management surface |

## Analytics and Success Signals

- **deposit_capture_confirmed** (booking reference, deposit amount, currency) -- supports success-metrics.md: "Deposit Capture Rate"
- **deposit_capture_duplicate_ignored** (booking reference) -- N/A -- no Stage 2 metric measures duplicate-delivery frequency directly; retained so the one-charge-per-booking guarantee's exercise rate is observable to the Pro's activity record (FEAT-16), not to a success metric.
- **deposit_capture_not_applied** (booking reference, reason: booking_no_longer_eligible) -- supports success-metrics.md: "Deposit Capture Rate" (this is exactly the "ambiguous or lost state" the metric's 100% target rules out, so its occurrence must be visible)

## Acceptance Criteria

**FEAT-07.SPEC-002-AC-01:** Given Riley's card is authorized and captured for a Booking that is Pending Payment, when FEAT-07.SPEC-005 reports the capture, then a Deposit Transaction is created with status Captured and the Booking transitions to Confirmed in the same step.

**FEAT-07.SPEC-002-AC-02:** Given a Deposit Transaction was just created for a Booking, when the same capture event is delivered a second time, then no second Deposit Transaction is created and the Booking's Confirmed state is unchanged.

**FEAT-07.SPEC-002-AC-03:** Given a capture event arrives for a Booking that is no longer Pending Payment for a reason unrelated to this automation, when the automation checks the Booking's state, then no Deposit Transaction is created and the outcome is escalated to FEAT-07.SPEC-004.

**FEAT-07.SPEC-002-AC-04:** Given the Deposit Transaction is created and the Booking transitions to Confirmed, when the transition completes, then FEAT-07.SPEC-001 shows its Success state and Riley is handed off to FEAT-05.SPEC-005.

**FEAT-07.SPEC-002-AC-05:** Given a Booking has just been confirmed by this automation, when the confirmation completes, then the confirmed Booking is handed to FEAT-08 for the client's confirmation message, and this automation itself sends no message.

**FEAT-07.SPEC-002-AC-06:** Given a Booking has just been confirmed by this automation, when the transition completes, then a deposit_payment_succeeded event is written to the append-only activity record (FEAT-16).

**FEAT-07.SPEC-002-AC-07:** Given Talia is viewing her schedule (FEAT-12) at the moment a client's deposit is captured, when the automation confirms the Booking, then the booking's paid/unpaid status updates to paid without Talia needing to refresh.

**FEAT-07.SPEC-002-AC-08:** Given the create-and-confirm step is interrupted by a processing error after the charge succeeded, when the automation retries, then the Deposit Transaction and the Confirmed transition either both persist or neither does -- Riley is never shown a state where one exists without the other.

**FEAT-07.SPEC-002-AC-09:** Given two different clients' captures are reported at effectively the same time, when both automations run, then each creates its own Deposit Transaction and confirms its own Booking independently, with no interference between the two runs.

**FEAT-07.SPEC-002-AC-10:** Given a captured amount reported by FEAT-07.SPEC-005 for a Booking, when this automation writes the Deposit Transaction, then the recorded amount and currency exactly match the deposit_amount FEAT-07.SPEC-003 locked on that Booking.

**FEAT-07.SPEC-002-AC-11:** Given a Deposit Transaction has been created for a Booking, when any later capture event for that same Booking is evaluated, then it is recognized as a duplicate and produces no new Deposit Transaction, satisfying the one-charge-per-booking guarantee.

**FEAT-07.SPEC-002-AC-12:** Given a Booking is confirmed by this automation, when FEAT-30 (Pro Booking Management) next loads that booking, then its paid/unpaid status reflects Confirmed/paid.

**FEAT-07.SPEC-002-AC-13:** Given a Booking has just been confirmed by this automation, when the transition completes, then FEAT-25.SPEC-004 is fired with the Booking's Pro Account reference, service reference, and start_time, and a failure of that aggregate update leaves the Deposit Transaction and Confirmed state unchanged.

**FEAT-07.SPEC-002-AC-14:** Given a Booking has just been confirmed by this automation, when the transition completes, then FEAT-04.SPEC-005 is fired with the Booking reference so the booking is written to the Pro's connected calendar, and a failure of that write leaves the Deposit Transaction and Confirmed state unchanged.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 (confirmed, duplicate ignored, not applied, automation failure) | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 4 | 4 |



# Logic/Rule Spec: Deposit Amount & Eligibility Rules

## Overview

**Name:** Deposit Amount & Eligibility Rules
**ID:** FEAT-07.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs how the deposit amount is computed exactly once from the service's rule, that it can never be altered by the client, and the preconditions that must hold before any charge is attempted.
**Parent Feature:** FEAT-07 -- Deposit Payment at Booking
**Governed Entity:** Booking -- the deposit-computation and eligibility-gating fields only (price_agreed, deposit_amount, currency as locked from the Pro Account, and the Pending Payment state precondition). All other Booking fields (client, policy_version, state's later transitions, attendance_reply, balance_due, source, cancellation/reschedule timestamps) are owned by other features' Logic/Rule specs, per the dependency map's Entity-Lifecycle ownership.

## Scope and Non-Goals

**In Scope:**
- Computing the deposit amount exactly once from the Service's deposit_rule and price, in the Pro's account currency
- The precondition checks that must all hold before any charge attempt is authorized (Booking still Pending Payment, currency locked and matching, payout account Active, no existing Deposit Transaction for the Booking)
- Preventing the client from altering the computed amount by any means
- The one-charge-per-booking rule at the point a charge is requested (working with FEAT-07.SPEC-004's idempotency guarantee for the point a charge is captured)

**Non-Goals:**
- Creating the Deposit Transaction and confirming the Booking on a successful charge -- owned by FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation); this spec only gates whether a charge attempt may proceed
- Guaranteeing correctness across a dropped connection or duplicate delivery of a charge outcome -- owned by FEAT-07.SPEC-004 (Payment Outcome Consistency & Idempotency)
- Defining the Service's deposit_rule field itself (fixed amount vs. percentage, its own bounds) -- owned by FEAT-01.SPEC-004 (Service Field & Deposit Rule Validation); this spec only consumes the already-validated rule to compute one booking's deposit
- Authorizing or capturing the card charge itself -- owned by FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing); this spec defines only whether a charge may be requested, never how the charge is processed

## Governed Entity

**Entity:** Booking (deposit-computation and eligibility-gating fields)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| price_agreed | number | The service price agreed and fixed at the moment of booking, in the Pro's account currency |
| deposit_amount | number | The deposit computed once from the Service's deposit_rule and price_agreed, fixed at the moment of booking |
| currency | derived (from Pro Account, at the moment of the account's first-ever deposit) | The currency both price_agreed and deposit_amount are expressed in |
| state | enum | Must be Pending Payment for a charge attempt to be eligible; this spec reads it as a precondition and never writes it (FEAT-07.SPEC-002 owns the Pending Payment -> Confirmed transition) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-07.SPEC-001 | Deposit Payment | Reads the locked deposit_amount for display on screen load; requests an eligibility check from this spec on every Pay tap, including retries |
| FEAT-07.SPEC-002 | Deposit Capture & Booking Confirmation | Confirms, before creating the Deposit Transaction, that the captured amount matches the deposit_amount this spec locked and that the eligibility precondition held at charge time |
| FEAT-07.SPEC-005 | Card Deposit Charge & Payout Routing | Requests this spec's eligibility check immediately before submitting an authorization/capture request to the payment-processing capability |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| deposit_amount | Computed once, exactly, from the Service's deposit_rule (fixed amount, or price_agreed x percentage / 100, rounded to the nearest currency unit) at the moment the Booking is created (FEAT-05.SPEC-006, from the amount FEAT-05.SPEC-009 computes); never recomputed afterward and never accepted as client-supplied input | Always | On Booking creation, before this screen is ever reached | N/A -- this is a system computation with no client-facing input to reject; the client is never shown an editable amount field | Yes |
| deposit_amount | Must be at least the minimum chargeable amount (platform parameter: `minimum-chargeable-deposit`) | Always -- already guaranteed by FEAT-01.SPEC-004 at the Service level, re-confirmed here as a precondition since a Service's price or rule could theoretically change between service selection and payment (XBR-04 forbids this for a Booking already created, but the check remains defense-in-depth) | On every eligibility check | "This booking's deposit could not be processed. Please start a new booking." (shown only in the theoretical case this precondition ever fails; FEAT-01.SPEC-004 makes it unreachable in normal operation) | Yes |
| currency | Must equal the Pro Account's locked currency (XBR-25; lock determined by FEAT-27.SPEC-008, which this check enforces) | Always | On every eligibility check | "This booking's currency no longer matches the Pro's account. Please start a new booking." | Yes |
| price_agreed | No validation beyond data type in this spec -- price_agreed is fixed at Booking creation by FEAT-05 and read here only to display the balance-due figure; its own validation is owned by FEAT-01.SPEC-004 (Service pricing) and FEAT-05.SPEC-009 (price lock at booking time) | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Amount immutability | deposit_amount | The value read at charge time must be byte-identical to the value computed at Booking creation; no interaction on FEAT-07.SPEC-001 (Deposit Payment) or FEAT-07.SPEC-005 accepts or applies an override, discount, or client-entered amount | N/A -- there is no field through which an override could be entered, so this is enforced by omission rather than by rejecting an input |
| Eligibility precondition set | state, currency, deposit_amount, Payout Account.status, Deposit Transaction (existence check) | All of the following must hold simultaneously before a charge attempt is authorized: the Booking is Pending Payment; currency matches the Pro Account's locked currency; deposit_amount meets the minimum-chargeable-deposit floor; the Pro's Payout Account status is Active (XBR-06); and no Deposit Transaction already exists for this Booking (one-charge-per-booking) | See the per-condition messages in Authorization Rules and Field Validation Rules below; the eligibility check as a whole fails closed -- if any condition is unmet, no charge is requested |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Request a deposit charge for a Booking | The Client (Riley) | Only for her own in-progress Booking, and only while the eligibility precondition set (above) holds | If the Booking is not Pending Payment: "This time is no longer available." and she is returned to FEAT-05.SPEC-002 (Slot Selection), per XBR-01. If the currency check fails: "This booking's currency no longer matches the Pro's account. Please start a new booking." If a Deposit Transaction already exists for the Booking (a second attempt after an already-successful charge, e.g. from a stale page): no new charge is requested; the screen is shown the Success state directly, per FEAT-07.SPEC-004. If the Payout Account is not Active: "This booking can't be paid right now. Please try again shortly, or contact {Pro's display name}." (per XBR-06 -- this state should not normally be reachable for a live booking link, since FEAT-05.SPEC-008 gates the link itself on an Active payout account, but the check remains defense-in-depth against a payout account being actioned mid-checkout) |
| View the computed deposit amount | The Client (Riley) | Only for her own in-progress Booking | -- |
| Alter the computed deposit amount | The Client (Riley) | Never | No control exists anywhere in the product for the client to alter deposit_amount; the field is display-only everywhere it appears |
| View the computed deposit amount and eligibility state | The Pro (Talia) | Full, but only after the Booking is Confirmed and visible on her own schedule/booking-management surfaces (FEAT-12, FEAT-30) -- never during the client's in-progress checkout, which she has no visibility into | -- |
| View transaction status resulting from an eligibility check | Platform Operator (Support) | View-only, on FEAT-16/FEAT-19, never card data or the in-progress checkout itself | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| deposit_amount | If Service.deposit_rule is a fixed amount: deposit_amount = the fixed amount. If Service.deposit_rule is a percentage: deposit_amount = round(price_agreed x percentage / 100) to the nearest currency unit | On Booking creation only (FEAT-05.SPEC-006, from the amount FEAT-05.SPEC-009 computes); read-only thereafter | No -- not by the Client, not by the Pro, not by Support. This is a system-computed value fixed for the life of the Booking (XBR-04) |
| currency | The Pro Account's currency, locked at that account's first-ever successful deposit (XBR-25) | Read at Booking creation and re-confirmed at every eligibility check | No |

## Business Rules

- XBR-05: The deposit is computed once, exactly, from the Service's rule in the Pro's account currency, cannot be altered by the client, and is charged once per booking; card data is never held by the product.
- XBR-06: No deposit can be taken unless the Pro's payout account is Active.
- XBR-25: Currency is locked to the Pro Account's currency once the account's first deposit is taken; every eligibility check re-confirms the Booking's currency still matches. Enforcing FEAT-27.SPEC-008 (Currency Lock Rule): this spec consumes that spec's lock determination as the charge precondition and never re-derives or redefines it.
- XBR-04: Service edits and archiving apply to future bookings only -- a confirmed or in-progress Booking keeps the price and deposit agreed at booking, so a Service-side price or rule change after Booking creation never changes this Booking's already-locked deposit_amount.
- One-charge-per-booking is enforced at two points for defense-in-depth: this spec blocks a second charge *request* once a Deposit Transaction exists (checked before the request is even sent), and FEAT-07.SPEC-004 guarantees correctness of the charge *outcome* even if two requests were somehow both sent (e.g., a race between two rapid taps or two tabs).
- The eligibility check runs identically on the first attempt and on every retry after a decline -- there is no relaxed re-check path for retries.

## Edge Cases

- **Service's deposit_rule changes after this Booking was created but before payment** -- No effect: XBR-04 fixes price_agreed and deposit_amount at Booking creation; this spec never re-reads the Service record to recompute after that point.
- **Percentage computation lands on a fractional currency unit** -- Rounded to the nearest whole currency unit at the moment of the one-time computation (e.g., a percentage yielding 24.5 rounds to 25); the rounded value is what is locked and never re-rounded afterward.
- **Payout Account transitions from Active to Action Required between screen load and Pay tap** -- The eligibility check re-runs at the Pay tap (not only at screen load) and catches the change; the charge is not requested and Riley sees "This booking can't be paid right now..." while Talia separately sees the Action Required banner on her own dashboard (FEAT-12).
- **Two rapid attempts both reach the eligibility check before either creates a Deposit Transaction** -- Both may pass eligibility (no Deposit Transaction exists yet for either), but only one may actually be captured as the successful charge; the second's capture attempt is resolved by FEAT-07.SPEC-004's idempotency guarantee, never by this spec, which only gates the request, not the outcome.
- **A second Pay attempt is made after an already-successful charge (e.g., a stale reloaded page)** -- The eligibility check finds an existing Deposit Transaction for the Booking and denies the request; the screen shows the Success state directly rather than requesting a second charge.
- **Currency stored on the Booking at creation differs from the Pro Account's currency due to an account-level change between Booking creation and payment** -- Cannot occur under XBR-25 (currency locks after the account's first deposit and never changes thereafter for that account); this edge case is therefore structurally prevented rather than handled at runtime.

## Acceptance Criteria

**FEAT-07.SPEC-003-AC-01:** Given a Service with a fixed-amount deposit_rule, when Riley's Booking is created for that service, then deposit_amount is set to exactly that fixed amount.

**FEAT-07.SPEC-003-AC-02:** Given a Service with a percentage deposit_rule and a price_agreed of {P}, when Riley's Booking is created, then deposit_amount is set to round(P x percentage / 100) to the nearest currency unit.

**FEAT-07.SPEC-003-AC-03:** Given Riley's Booking has a locked deposit_amount, when she views the Deposit Payment screen, then no control anywhere lets her change that amount.

**FEAT-07.SPEC-003-AC-04:** Given Riley's Booking is Pending Payment, currency matches, the Pro's payout account is Active, and no Deposit Transaction exists for it, when she taps Pay, then the eligibility check passes and a charge is requested.

**FEAT-07.SPEC-003-AC-05:** Given Riley's Booking is no longer Pending Payment (for example, its checkout hold has expired and released the slot), when she taps Pay, then the eligibility check fails and she sees "This time is no longer available." and is returned to FEAT-05.SPEC-002.

**FEAT-07.SPEC-003-AC-06:** Given the Pro's payout account is not Active at the moment Riley attempts to pay, when the eligibility check runs, then no charge is requested and she sees "This booking can't be paid right now. Please try again shortly, or contact {Pro's display name}."

**FEAT-07.SPEC-003-AC-07:** Given a Deposit Transaction already exists for Riley's Booking (a prior attempt already succeeded), when she taps Pay again from a stale page, then no new charge is requested and she is shown the Success state directly.

**FEAT-07.SPEC-003-AC-08:** Given the Booking's currency no longer matches the Pro Account's locked currency, when the eligibility check runs, then no charge is requested and Riley sees "This booking's currency no longer matches the Pro's account. Please start a new booking."

**FEAT-07.SPEC-003-AC-09:** Given a Service's price or deposit_rule is edited by Talia after Riley's Booking was already created, when Riley proceeds to pay, then her Booking's deposit_amount is unaffected by the edit, per XBR-04.

**FEAT-07.SPEC-003-AC-10:** Given Riley (the Client) is viewing her own in-progress Booking, when she looks at the deposit amount, then she sees exactly the locked value with no edit affordance.

**FEAT-07.SPEC-003-AC-11:** Given Talia (the Pro) has not yet had this Booking's deposit captured, when she looks at her own schedule or booking-management surfaces, then she sees no visibility into Riley's in-progress checkout state -- only the eventual Confirmed/paid status once capture succeeds.

**FEAT-07.SPEC-003-AC-12:** Given Platform Operator (Support) opens a Pro's account after a help request, when they look at a booking's payment status, then they see the transaction status only, via FEAT-16/FEAT-19, never the in-progress checkout screen or card data.

**FEAT-07.SPEC-003-AC-13:** Given a percentage computation yields a fractional currency unit of exactly .5, when the one-time computation runs, then the result rounds to the nearest whole currency unit and that rounded value is what is locked.

**FEAT-07.SPEC-003-AC-14:** Given the Pro's payout account transitions to Action Required between Riley's screen load and her Pay tap, when she taps Pay, then the eligibility check re-runs and denies the charge with the same message as AC-06, rather than relying on the state at screen load.

**FEAT-07.SPEC-003-AC-15:** Given Riley retries payment after an earlier decline, when the retry's eligibility check runs, then it applies the identical full precondition set as the first attempt -- no relaxed re-check path exists for retries.

**FEAT-07.SPEC-003-AC-16:** Given a Booking's deposit_amount would fall below platform parameter: `minimum-chargeable-deposit` due to a theoretical precondition failure, when the eligibility check runs, then no charge is requested and Riley sees "This booking's deposit could not be processed. Please start a new booking."

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 4 | 4 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |



# Logic/Rule Spec: Payment Outcome Consistency & Idempotency

## Overview

**Name:** Payment Outcome Consistency & Idempotency
**ID:** FEAT-07.SPEC-004
**Type:** Logic/Rule
**Purpose:** Guarantees every payment attempt ends in exactly one of a clean success or a clean, actionable failure -- never a double charge and never an ambiguous booking state, including when the confirmation UI itself fails to load or the connection drops mid-payment.
**Parent Feature:** FEAT-07 -- Deposit Payment at Booking
**Governed Entity:** Deposit Transaction (creation idempotency) jointly with Booking (Pending Payment / Confirmed consistency) -- this spec's correctness guarantee spans both, since a payment attempt is only "resolved" once the two are consistent with each other.

## Scope and Non-Goals

**In Scope:**
- The guarantee that at most one Deposit Transaction is ever created per Booking, regardless of retries, dropped connections, or duplicate event delivery
- The guarantee that the Booking's state always matches the true outcome of the most recent charge attempt, even when the client's device never receives confirmation of that outcome
- Resolution behavior when the confirmation step (FEAT-07.SPEC-001's screen, or the hand-off to FEAT-05.SPEC-005) fails to load after a successful charge
- Resolution behavior when the connection drops between the client submitting card details and the client's device learning the outcome

**Non-Goals:**
- Computing the deposit amount or checking eligibility preconditions before a charge is requested -- owned by FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules); this spec governs correctness *after* a charge attempt has been requested, not whether it should have been requested
- Creating the Deposit Transaction and performing the Confirmed transition themselves -- owned by FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation); this spec defines the guarantee that automation's create-and-confirm step must uphold, not the step's own mechanics
- Authorizing or capturing the card charge, or translating the processor's decline reasons -- owned by FEAT-07.SPEC-005 (Card Deposit Charge & Payout Routing)
- Refund or forfeiture correctness after a deposit has been captured -- excluded per the Entity-Lifecycle Coverage Matrix: later Deposit Transaction states (Refunded, Forfeited, Disputed) are owned by FEAT-09, FEAT-11, and FEAT-30, each with their own correctness rules

## Governed Entity

**Entity:** Deposit Transaction (creation) and Booking (Pending Payment / Confirmed transition), governed jointly
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| Deposit Transaction (existence) | derived | Whether a Deposit Transaction record exists for a given Booking -- the fact this spec guarantees is single-valued (zero or one, never more) |
| Deposit Transaction.status | enum | At this feature's stage: Authorized \| Captured. This spec guarantees the status set here is the true, final outcome of the one attempt that succeeded |
| Booking.state (Pending Payment / Confirmed slice) | enum | This spec guarantees Booking.state reflects Confirmed if and only if exactly one successful capture occurred for it, and remains Pending Payment (available for retry) if and only if no capture has yet succeeded |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-07.SPEC-001 | Deposit Payment | Disables the Pay button while a request is in flight (double-submit prevention); on screen load or return, re-checks the Booking's actual state before allowing a new attempt or showing Success directly |
| FEAT-07.SPEC-002 | Deposit Capture & Booking Confirmation | Implements the atomic create-and-confirm step this spec requires: the Deposit Transaction and the Confirmed transition commit together or not at all |
| FEAT-07.SPEC-005 | Card Deposit Charge & Payout Routing | Applies this spec's idempotency key discipline when submitting a charge request, so a retried request for the same attempt is never processed twice by the payment-processing capability |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Deposit Transaction (existence, per Booking) | At most one Deposit Transaction may ever exist for a given Booking | Always | Before every charge request (FEAT-07.SPEC-003) and again at capture time (FEAT-07.SPEC-002) | N/A -- enforced structurally by the create-and-confirm step refusing to create a second record, not by a client-facing error | Yes |
| Booking.state | Must never show as Confirmed unless exactly one Deposit Transaction with status Captured exists for it, and must never remain Pending Payment once that Deposit Transaction exists | Always | Continuously -- this is an invariant the create-and-confirm step (FEAT-07.SPEC-002) maintains, not a point-in-time check | N/A -- this is a system invariant, not a validated input | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Atomic capture-and-confirm | Deposit Transaction (existence, status), Booking.state | The Deposit Transaction's creation with status Captured and the Booking's transition to Confirmed must occur as one indivisible step: a partial state where one exists without the other is never observable to any client, screen, or downstream feature | N/A -- structural guarantee, no client-facing error; if the step cannot complete, it is retried automatically until it does, per Business Rules below |
| Idempotency key discipline | Deposit Transaction (existence), the charge request itself | Every charge request FEAT-07.SPEC-005 submits to the payment-processing capability carries an identifier tied to this specific attempt on this specific Booking, so a request that is resubmitted (due to a client retry after a dropped connection, or a network-level retry) is recognized by the capability as the same attempt rather than a new charge | N/A -- structural guarantee at the integration boundary, not a client-facing error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Submit a new charge attempt for a Booking | The Client (Riley) | Only when no charge attempt is currently in flight for that Booking and no Deposit Transaction yet exists for it (per FEAT-07.SPEC-003's eligibility gate, which this spec's in-flight check extends) | If an attempt is already in flight: the Pay button remains disabled and the tap is ignored (FEAT-07.SPEC-001) -- no error message, since this is prevented before a second request can even be sent. If a Deposit Transaction already exists: no new request is sent; she is shown the Success state directly |
| Resolve an ambiguous outcome on the Client's behalf | The Client (Riley) | Never -- the resolution is always determined by the true state this spec reconciles (whether a capture actually occurred), never by a choice Riley makes | Riley is never asked "were you charged?" or given a manual "I was charged" override; the screen always reflects the system's own determination |
| View or intervene in a payment attempt's reconciliation | The Pro (Talia) | Never during the attempt itself; Full view of the resulting outcome once resolved, on her own schedule/booking-management surfaces | Talia has no visibility into an in-progress attempt; she sees only the resolved Confirmed/paid state or the fact that the Booking remains Pending Payment |
| View reconciliation outcomes for support purposes | Platform Operator (Support) | View-only, on FEAT-16/FEAT-19, after the outcome resolves | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Booking.state, on screen return after an interrupted confirmation | Derived from the true Deposit Transaction state at the moment the screen re-loads: Confirmed if a Captured Deposit Transaction exists, otherwise Pending Payment (available for retry) | Whenever FEAT-07.SPEC-001 loads or re-loads for a given Booking | No -- this is always read fresh from the true state, never cached or assumed from the client's last known screen state |

## Business Rules

- **Never double-charge:** at most one successful capture is ever recorded against a Booking; a second charge attempt for the same Booking, however triggered, either finds an existing Deposit Transaction and is refused before it reaches the payment-processing capability (FEAT-07.SPEC-003), or is submitted with the same idempotency key as an in-flight or already-resolved attempt and is recognized as the same attempt by that capability rather than processed as a new charge.
- **Never leave an ambiguous outcome:** every charge attempt resolves, from the product's point of view, to exactly one of Captured or not-Captured. There is no third "unknown" state that persists -- if the outcome is not yet known (e.g., the connection dropped before the client's device learned it), the product resolves it by checking the true state (has a Deposit Transaction actually been created?) rather than guessing or leaving the Booking in limbo.
- **The Booking's state is the source of truth, never the client's screen history:** if a charge succeeds but the confirmation step (FEAT-07.SPEC-001's screen, or the hand-off to FEAT-05.SPEC-005) fails to load, the Booking's Confirmed state is authoritative and is shown on the next page load or, per product-features.md's Communications field, via the confirmation message itself -- Riley is never left unsure whether she is booked.
- **Automatic retry on interruption before resolution, never after:** if the create-and-confirm step (FEAT-07.SPEC-002) is interrupted before it commits, it is retried automatically until it commits; once it has committed (a Deposit Transaction with status Captured exists), no further retry of the charge itself ever occurs -- only screen-level re-loads of the already-resolved state.
- **Consistent with FEAT-01's price/deposit lock (XBR-04):** this spec's reconciliation never re-derives or re-checks the deposit amount -- it only reconciles whether the one attempt at the one locked amount succeeded, matching FEAT-07.SPEC-003's amount that was fixed before the attempt began.

## Edge Cases

- **Connection drops after Riley submits card details but before her device receives a result** -- On reconnection, FEAT-07.SPEC-001 re-loads the Booking's true state rather than assuming failure: if a Captured Deposit Transaction exists, she is shown Success; if none exists and no attempt is in flight, she is shown the Default state ready for a fresh attempt; she is never shown a state that lets her submit a second charge while the first attempt's true outcome is still unknown to the product itself (the product waits for the payment-processing capability's own resolution before allowing a new attempt).
- **The confirmation screen (FEAT-05.SPEC-005) fails to load after a successful capture** -- The Booking is already Confirmed at the moment of capture (FEAT-07.SPEC-002); Riley sees the confirmation on her next page load, or via the confirmation message FEAT-08 sends, whichever she reaches first -- never a re-prompt to pay again.
- **Riley closes the browser tab immediately after tapping Pay, before any result is shown, then reopens the booking link later** -- The reopened flow re-checks the Booking's true state exactly as on a reconnect; if the earlier attempt had in fact succeeded, she is shown Confirmed; if it failed or never completed, she is shown Pending Payment ready for a fresh attempt (subject to her checkout hold still being active, per FEAT-07.SPEC-001's Hold Expired state).
- **Two devices attempt to pay the same Booking at effectively the same time (for example, Riley reloads the page on a second tab while the first tab's request is still in flight)** -- Only one request can result in a Captured Deposit Transaction; the other, whichever resolves second, finds the Deposit Transaction already exists (per FEAT-07.SPEC-003's pre-request check, or the idempotency key at the capability boundary) and is treated as a no-op, with that device's screen showing Success directly rather than a duplicate charge or an error.
- **The payment-processing capability reports success for a charge request the product had already given up retrying (a very late, delayed response)** -- The late success is still applied through the same create-and-confirm step (FEAT-07.SPEC-002) if no Deposit Transaction yet exists for the Booking; if a different attempt already succeeded in the meantime, the late report is treated as the duplicate-capture case and produces no second Deposit Transaction -- money is never lost or double-recorded either way.

## Acceptance Criteria

**FEAT-07.SPEC-004-AC-01:** Given Riley's connection drops immediately after she taps Pay, when she reconnects and the screen reloads, then it shows Success if the charge in fact succeeded, or the Default state ready for a fresh attempt if it did not -- never an ambiguous "unknown" state.

**FEAT-07.SPEC-004-AC-02:** Given a charge attempt for Riley's Booking has already succeeded, when any later request (a retry, a second tab, a delayed capability response) reaches the point of creating a Deposit Transaction, then no second Deposit Transaction is created and the Booking remains Confirmed with its original outcome.

**FEAT-07.SPEC-004-AC-03:** Given the confirmation screen fails to load right after Riley's deposit is successfully captured, when she reaches the booking again later (by reopening the link or from the confirmation message), then she sees the booking as Confirmed, never a re-prompt to pay.

**FEAT-07.SPEC-004-AC-04:** Given Riley taps Pay and a request is in flight, when she taps Pay again before a result arrives, then the second tap produces no second request -- the button remains disabled and ignores the tap.

**FEAT-07.SPEC-004-AC-05:** Given the create-and-confirm step (FEAT-07.SPEC-002) is interrupted by a processing error before it commits, when the system retries, then it retries until the step commits, and Riley is never shown a state where the Deposit Transaction exists without the Confirmed transition or vice versa.

**FEAT-07.SPEC-004-AC-06:** Given a create-and-confirm step has already committed successfully, when any further retry logic runs for that same attempt, then no further retry of the charge itself occurs -- only the already-resolved state is re-displayed.

**FEAT-07.SPEC-004-AC-07:** Given Riley closes the browser tab right after tapping Pay with no result shown, when she reopens the booking link later, then the screen shows the true current state of the Booking rather than assuming the earlier attempt failed.

**FEAT-07.SPEC-004-AC-08:** Given Riley opens the same in-progress Booking on two devices and taps Pay on both at effectively the same time, when both requests are processed, then only one results in a Captured Deposit Transaction, and the other device's screen shows Success directly rather than a duplicate charge or an error.

**FEAT-07.SPEC-004-AC-09:** Given the payment-processing capability reports a very late success for a request the product had stopped waiting on, when the report arrives and no Deposit Transaction yet exists for the Booking, then it is applied through the normal create-and-confirm step exactly as an on-time success would be.

**FEAT-07.SPEC-004-AC-10:** Given the payment-processing capability reports a very late success for a request whose Booking was already confirmed by a different successful attempt, when the report arrives, then it is treated as a duplicate and produces no second Deposit Transaction.

**FEAT-07.SPEC-004-AC-11:** Given Riley (the Client) is asked to resolve an ambiguous payment outcome, when she looks for a manual "I was charged" option, then none exists -- the screen's state is always determined by the system's own reconciliation of the true Deposit Transaction state.

**FEAT-07.SPEC-004-AC-12:** Given Talia (the Pro) views her schedule while one of her client's payment attempts is still in flight, when she looks at that booking, then she sees no in-progress-attempt detail -- only the resolved Confirmed/paid state once it settles, or Pending Payment if it has not yet succeeded.

**FEAT-07.SPEC-004-AC-13:** Given every charge request this spec governs, when it is submitted to the payment-processing capability, then it carries an identifier tied to that specific attempt so a resubmission of the same request is recognized as the same attempt rather than processed as a new charge.

**FEAT-07.SPEC-004-AC-14:** Given a Deposit Transaction already exists with a Captured status for a Booking, when the eligibility check in FEAT-07.SPEC-003 runs for any further attempt on that Booking, then it denies the request, upholding this spec's never-double-charge guarantee at the point of request as well as at the point of outcome.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 2 | 2 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |



# Integration Spec: Card Deposit Charge & Payout Routing

## Overview

**Name:** Card Deposit Charge & Payout Routing
**ID:** FEAT-07.SPEC-005
**Type:** Integration
**Purpose:** Authorizes and captures Riley's card charge through the payment-processing capability, reports the processor's own card fee, and routes the captured deposit to Talia's connected payout account with zero platform fee.
**Parent Feature:** FEAT-07 -- Deposit Payment at Booking

## Scope and Non-Goals

**In Scope:**
- Requesting authorization and capture of a deposit charge for an eligible Booking, using the card details Riley enters directly into the capability's own entry element
- Receiving and translating the capability's outcome (captured, declined) into plain-language feedback for FEAT-07.SPEC-001
- Reporting the capability's own processor_fee on the captured transaction
- Routing the captured amount to the Pro's connected Payout Account, with Chairtime's own fee always zero (XBR-07)
- User-facing behavior when the capability is slow, unavailable, or rejects a request
- Disclosure to Riley about what data is shared with the capability

**Non-Goals:**
- Verifying or connecting the Pro's payout account itself (identity and bank verification) -- owned by FEAT-28.SPEC-006 (Payout Account Connection & Payout Visibility); this spec only routes an already-captured deposit to an already-active account
- Computing the deposit amount or checking eligibility preconditions -- owned by FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules); this spec only submits the charge for an amount and Booking that spec has already cleared
- Creating the Deposit Transaction record or confirming the Booking -- owned by FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation); this spec reports the outcome that automation acts on, it does not write the Deposit Transaction itself
- Refunds triggered by cancellation, no-show forfeiture, or goodwill -- excluded per the External Touchpoints table: those flows are FEAT-09.SPEC-005 and FEAT-30.SPEC-011's own integration behavior against the same payment-processing capability, not this spec's; this spec covers only the original deposit charge and its payout routing

## Capability Category

**Category:** Payment processing
**Dependency Source:** ASMP-31 -- "Payment-processing capability -- required to take client deposits, verify each pro's identity and bank details for a connected payout account, pay deposits out to the pro, issue refunds, notify the product of card-issuer disputes, and bill the pro's own monthly subscription." (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Payment processing -- client card charges and refunds (deposits; from v1 balances; Later tips)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved includes FEAT-07; Integration Specs column names FEAT-07.SPEC-005 as the deposit card authorization and capture, processor fee reporting, zero platform fee); and the "Payment processing -- connected payout accounts with identity and bank verification" row, which also names FEAT-07.SPEC-005 for routing each captured deposit to the Pro's connected payout account
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision. BRIEF.md's Constraints establish only the hard boundary that card data is never stored or handled by the product's own code, not a named vendor.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Riley pays the exact deposit amount required by the selected service's rule, by card, without her card details ever touching the product's own code | Pay the exact deposit amount required by the selected service's rule, by card | FEAT-07.SPEC-001 (Deposit Payment) |
| Riley sees a clear, on-screen confirmed record that her deposit succeeded | See a clear on-screen and confirmed record that the deposit succeeded | FEAT-07.SPEC-001 (Deposit Payment), FEAT-05.SPEC-005 (Booking Confirmation) |
| Riley sees a failed or declined payment explained in plain language and can retry without losing her held slot | Have a failed or declined payment explained clearly, with the slot held briefly to retry | FEAT-07.SPEC-001 (Deposit Payment) |
| Talia's deposit lands directly in her own payout account with no Chairtime cut, and she can see the processor's own card fee | Deposit lands directly in the Pro's own payout account, with the platform taking no cut; the only deduction is the payment processor's own card fee, shown to the Pro | FEAT-28 (Payout Account Connection & Payout Visibility) -- the money-list display itself; this spec supplies the routed amount and fee it displays |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Deposit amount and currency | Booking -- deposit_amount, currency | Riley taps Pay on FEAT-07.SPEC-001, after FEAT-07.SPEC-003's eligibility check passes | The capability must know exactly what to authorize and capture |
| Booking reference | Booking -- an internal reference sufficient to tie the outcome back to this specific Booking | Same moment as above | Ties the capability's reported outcome (captured, declined) back to the correct Booking, and carries the idempotency key FEAT-07.SPEC-004 requires |
| Destination payout account reference | Payout Account -- processor_account_reference | Same moment as above | Tells the capability where to route the captured funds -- the Pro's own connected account, never Chairtime's |
| Card details Riley enters | Not a product entity -- entered directly into the capability's own entry element and never received by the product's own code, per SC-11 | While Riley fills the card entry element on FEAT-07.SPEC-001 | The capability needs the card details to attempt authorization; the product never touches or stores them |

Booking's service name, appointment time, client name and phone, and every other Client or Booking field never leave the product through this integration -- only the deposit amount, currency, an internal Booking reference, and the payout destination reference are shared.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Capture outcome (captured / declined) | The capability resolves an authorization/capture request | Consumed directly by FEAT-07.SPEC-002 (captured) or shown inline on FEAT-07.SPEC-001 (declined); no Deposit Transaction is written for a decline |
| Decline reason (plain-language category) | The capability reports a decline | Surfaced on FEAT-07.SPEC-001's Error state; not persisted on the Booking, since a decline never reaches the Deposit Transaction entity |
| processor_fee | The capability reports the fee alongside a successful capture | Deposit Transaction -- processor_fee (written by FEAT-07.SPEC-002 at the moment it creates the record from this spec's reported values) |
| Payout routing confirmation | The capability confirms the captured amount was routed to the destination Payout Account | Feeds Payout Account's recent-payouts data, displayed by FEAT-28's money list; this spec supplies the confirmation, FEAT-28 owns the display |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Charge captured | The capability successfully authorizes and captures Riley's card for the requested deposit amount | None directly in this spec -- the captured amount, currency, and processor_fee are reported to FEAT-07.SPEC-002, which creates the Deposit Transaction and confirms the Booking | FEAT-07.SPEC-001 shows the Success state; the Pro's money list (FEAT-28) will show the deposit once FEAT-07.SPEC-002 and the payout routing confirmation complete | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) |
| Deposit outcome set (captured) | The Charge captured event above completes successfully | None in this spec -- the captured amount and processor_fee are reported onward for FEAT-25.SPEC-004 to update its deposits-collected aggregate | None to Riley; Talia's insights figures reflect the deposit on their next refresh | FEAT-25.SPEC-004 (Historical Aggregate Maintenance) |
| Charge declined | The capability cannot capture the charge (insufficient funds, card declined, expired card, or another card-level reason) | None -- no Deposit Transaction is created for a decline | FEAT-07.SPEC-001 shows the Error state with the plain-language decline reason and a "Try a different card" action; the Booking remains Pending Payment and the checkout hold is untouched | FEAT-07.SPEC-001 (Deposit Payment) |
| Payout routing confirmed | The captured amount is successfully routed to the Pro's connected Payout Account | Feeds Payout Account's recent-payouts data (owned and displayed by FEAT-28) | No separate feedback to Riley; Talia sees it reflected in her own money list (FEAT-28) on her own schedule | FEAT-28 (Payout Account Connection & Payout Visibility) |
| Charge request times out with no result received | The capability does not respond within the product's expected response window | None -- no Deposit Transaction is created while the outcome is unknown | FEAT-07.SPEC-001 shows its Offline/Degraded resolution; FEAT-07.SPEC-004 governs how a subsequent late response (success or decline) is reconciled once it does arrive | FEAT-07.SPEC-001 (Deposit Payment), FEAT-07.SPEC-004 (Payment Outcome Consistency & Idempotency) |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-07.SPEC-001 (Deposit Payment) | The status region shows "Processing payment, do not close this page." for as long as the request is in flight; past a brief threshold it adds "Still working -- this is taking longer than usual." The Pay button and card entry element remain disabled throughout; Riley's checkout hold is not affected by processing time alone. | The Pay button is disabled before a request is even sent, with the message "We can't take payments right now. Please try again in a few minutes -- your held time is still counting down." Riley's held slot continues to count down under its normal window (XBR-02); if it expires while the capability is down, the Hold Expired state applies exactly as it would for any other cause. | The specific decline reason is shown in plain language (for example, "Your card was declined. Try a different card." or "This card has expired. Try a different card."), never a raw processor code; a "Try a different card" action is offered and the Booking remains Pending Payment, per the never-ambiguous-outcome guarantee in FEAT-07.SPEC-004. |

No other screen sends requests to this capability or displays its live results; the Pro's own surfaces (FEAT-12, FEAT-28, FEAT-30) reflect only already-resolved outcomes, which are unaffected by a degradation condition that occurs before resolution.

## Consent and Disclosure

- **Card entry disclosure** -- Directly above the card entry element on FEAT-07.SPEC-001, a persistent line reads: "Your card details go straight to our payment processor -- Chairtime never sees or stores your card number." This is shown every time the screen is reached, not only on first use, since each deposit is paid fresh with no card kept on file (per BRIEF.md's Constraints).
- **What the payout destination receives** -- The disclosure line above also implicitly covers that the payment is destined for the Pro's own connected account; no separate consent prompt is needed for this, since Riley is already booking that specific Pro and the amount and recipient are evident from the booking context itself (service, price, and Pro name shown on the same screen).
- **What is never shared** -- The Booking's service name, appointment time, Riley's name, phone, email, and any booking note never leave the product through this integration; only the deposit amount, currency, an internal Booking reference, and the Pro's payout destination reference are sent, exactly as scoped in Data Exchanged. This boundary is stated in the disclosure line's plain wording ("Chairtime never sees or stores your card number") and is never contradicted elsewhere in the flow.
- **Pro-facing disclosure of the fee deduction** -- The first time Talia views her money list after her first captured deposit (FEAT-28), a one-time notice states: "Chairtime never takes a cut of your deposits. The only deduction you'll ever see is your payment processor's own card fee, shown on each transaction." This is FEAT-28's own display responsibility; this spec supplies the processor_fee value that notice and every transaction line reference.

## Edge Cases

- **The same capture-succeeded event is delivered twice** -- The second delivery reaches FEAT-07.SPEC-002, which finds an existing Deposit Transaction and takes no further action; this spec itself performs no state changes on receipt, so duplicate delivery has no direct effect here beyond the automation's own idempotency handling.
- **A capture event arrives for a Booking that has since left Pending Payment for an unrelated reason** -- Handled by FEAT-07.SPEC-002 and FEAT-07.SPEC-004's correctness guarantee, not silently dropped by this spec; this spec's own responsibility ends at reporting the event faithfully with the Booking reference it received.
- **A decline and a late capture-succeeded report both arrive for the same attempt (out-of-order delivery)** -- The capability's own outcome for a single charge request is authoritative and final once reported; a genuinely late success after an already-shown decline can only occur if the two reports describe two separate requests (Riley's retry after the decline), each carrying its own idempotency key per FEAT-07.SPEC-004, so they are never conflated into one ambiguous outcome.
- **The capability goes down mid-authorization, after the request was sent but before any result is received** -- No Deposit Transaction is created while the outcome is unknown; FEAT-07.SPEC-001 shows its Offline/Degraded state, and if the capability's result eventually does arrive (success or decline), FEAT-07.SPEC-004 reconciles it against the Booking's true state rather than assuming failure.
- **The captured amount reported by the capability does not exactly match the deposit_amount locked by FEAT-07.SPEC-003** -- Cannot occur under normal operation, since this spec always requests exactly the locked amount and the capability captures exactly what was requested or declines; if a mismatch were ever reported, FEAT-07.SPEC-002 would refuse to create the Deposit Transaction from a mismatched amount and the outcome would be escalated to FEAT-07.SPEC-004 as an unresolved attempt rather than recorded with an incorrect figure.
- **Payout routing confirmation is delayed after a successful capture** -- The capture outcome (Booking Confirmed, Deposit Transaction Captured) is not held pending on payout routing; Riley's confirmation is never delayed by a slow payout leg. Talia's money list (FEAT-28) shows the deposit as received once routing confirms, on the processor's own reporting timeline.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-001 (Deposit Payment) | Triggered by (inbound) | The Pay action initiates the authorization/capture request |
| FEAT-07.SPEC-001 (Deposit Payment) | Affects (outbound) | Processing, decline, and offline/degraded states surface this spec's reported outcomes and degradation behavior |
| FEAT-07.SPEC-003 (Deposit Amount & Eligibility Rules) | Triggered by (inbound) | Eligibility passing is the precondition for this spec ever sending a charge request |
| FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Triggers (outbound) | A reported successful capture fires that automation's create-and-confirm step |
| FEAT-07.SPEC-004 (Payment Outcome Consistency & Idempotency) | References (inbound) | Supplies the idempotency-key discipline this spec applies to every charge request, and governs reconciliation of any late or ambiguous response |
| FEAT-25.SPEC-004 -- within FEAT-25 (Booking & Revenue Insights) | Triggers (outbound) | A captured deposit fires the deposits-collected aggregate update (Inbound Events: Deposit outcome set) |
| FEAT-28 (Payout Account Connection & Payout Visibility) | Affects (outbound) | The routed deposit and its processor_fee are what that feature's money list displays |

## Analytics and Success Signals

- **deposit_charge_requested** (booking reference, deposit amount, currency) -- supports success-metrics.md: "Deposit Capture Rate"
- **deposit_charge_captured** (booking reference, processor_fee) -- supports success-metrics.md: "Deposit Capture Rate"
- **deposit_charge_declined** (booking reference, decline reason category) -- supports success-metrics.md: "Deposit Capture Rate"
- **deposit_payout_routing_confirmed** (booking reference) -- N/A -- no Stage 2 metric measures payout routing timing directly; retained so a delayed or failed routing leg is observable to the Pro's activity record (FEAT-16) and money list (FEAT-28), separately from the deposit capture itself.
- **deposit_capability_degraded** (condition: slow / down / rejected) -- N/A -- no Stage 2 metric measures degradation frequency; retained so the product's tolerance for capability trouble is observable, consistent with how Analytics sections are written for this feature's Screen spec.

## Acceptance Criteria

**FEAT-07.SPEC-005-AC-01:** Given Riley's Booking has passed FEAT-07.SPEC-003's eligibility check, when she taps Pay, then a charge request for exactly the locked deposit_amount and currency is sent to the payment-processing capability, carrying an idempotency key and the Booking reference.

**FEAT-07.SPEC-005-AC-02:** Given the capability successfully captures Riley's charge, when it reports the outcome, then this spec reports the captured amount, currency, and processor_fee to FEAT-07.SPEC-002 without creating the Deposit Transaction itself.

**FEAT-07.SPEC-005-AC-03:** Given the capability declines Riley's card, when it reports the decline, then FEAT-07.SPEC-001 shows the plain-language decline reason and a "Try a different card" action, and no Deposit Transaction is created.

**FEAT-07.SPEC-005-AC-04:** Given a captured deposit, when payout routing completes, then the amount is routed to Talia's connected Payout Account with a Chairtime fee of zero, and the processor's own fee is the only deduction reported.

**FEAT-07.SPEC-005-AC-05:** Given Riley is on the Deposit Payment screen for the first time this session, when she reaches the card entry element, then she sees "Your card details go straight to our payment processor -- Chairtime never sees or stores your card number." above it.

**FEAT-07.SPEC-005-AC-06:** Given a charge request is in flight, when more than the product's brief processing threshold passes without a result, then the status region adds "Still working -- this is taking longer than usual." while the Pay button remains disabled.

**FEAT-07.SPEC-005-AC-07:** Given the payment-processing capability is unavailable when Riley reaches the Pay step, when she views the screen, then the Pay button is disabled with "We can't take payments right now. Please try again in a few minutes -- your held time is still counting down." and her checkout hold continues counting down normally.

**FEAT-07.SPEC-005-AC-08:** Given the capability is down and Riley's checkout hold expires while she waits, when the expiry is reported, then FEAT-07.SPEC-001 shows its Hold Expired state exactly as it would for any other cause of expiry.

**FEAT-07.SPEC-005-AC-09:** Given the capability rejects a charge with a specific reason, when the rejection is reported, then Riley sees that specific plain-language reason (for example, "This card has expired. Try a different card."), never a raw processor code.

**FEAT-07.SPEC-005-AC-10:** Given a capture-succeeded event is delivered twice for the same Booking, when the second delivery reaches FEAT-07.SPEC-002, then no second Deposit Transaction is created.

**FEAT-07.SPEC-005-AC-11:** Given the capability goes down after a charge request was sent but before any result is received, when Riley's screen detects this, then no Deposit Transaction is created while the outcome remains unknown, and the Offline/Degraded state per FEAT-07.SPEC-001 is shown.

**FEAT-07.SPEC-005-AC-12:** Given a capture outcome is confirmed, when Riley's confirmation is shown, then it is never delayed waiting for the separate payout-routing confirmation to Talia's account.

**FEAT-07.SPEC-005-AC-13:** Given Talia views her money list after her first-ever captured deposit, when the notice appears, then it reads "Chairtime never takes a cut of your deposits. The only deduction you'll ever see is your payment processor's own card fee, shown on each transaction."

**FEAT-07.SPEC-005-AC-14:** Given a charge request for Riley's Booking, when it is sent to the capability, then it carries only the deposit amount, currency, an internal Booking reference, and the Pro's payout destination reference -- never Riley's name, phone, email, or the service and appointment details.

**FEAT-07.SPEC-005-AC-15:** Given a Booking's payment attempt has an out-of-order pair of reports (a decline followed by a late success, or the reverse) that in fact describe two separate retried attempts, when both are processed, then each is resolved against its own idempotency key and the two are never conflated into a single ambiguous outcome.

**FEAT-07.SPEC-005-AC-16:** Given the capability reports a captured amount that does not match the locked deposit_amount, when FEAT-07.SPEC-002 checks it, then no Deposit Transaction is created from the mismatched figure and the attempt is escalated to FEAT-07.SPEC-004 as unresolved.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 3 (one screen x three conditions) | 3 |
| Consent and Disclosure | 4 | 4 |
| Edge Cases | 6 | 6 |

