---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-23.SPEC-003
spec_name: Tip Payout & Refund Rule
spec_slug: tip-payout-refund-rule
parent_feature: FEAT-23
parent_feature_name: Tipping at Checkout
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 5
acceptance_criteria_count: 10
---

# Logic/Rule Spec: Tip Payout & Refund Rule

## Overview

**Name:** Tip Payout & Refund Rule
**ID:** FEAT-23.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs where a validated tip's money goes once a balance payment succeeds -- the whole amount to the Pro's payout account with no platform cut -- and what must happen to it if the appointment is later cancelled: refunded in full together with the balance, never forfeited.
**Parent Feature:** FEAT-23 -- Tipping at Checkout
**Governed Entity:** Balance Payment (payout and refund behavior tied to the `tip` field and the `state` field's Refunded transition)

## Scope and Non-Goals

**In Scope:**
- The rule that a captured tip routes in full to the Pro's connected Payout Account, with no platform-fee deduction (XBR-07)
- The rule that a tip is refunded in full alongside the balance whenever the Balance Payment transitions to Refunded, tied to a booking cancellation after a tipped balance payment succeeded (XBR-23)
- Authorization for who may receive, view, or influence a tip's payout or refund
- The interaction between a tip and the separate no-show/deposit-forfeiture path, to confirm the tip is never subject to it

**Non-Goals:**
- Validating the tip amount's own format, sign, or default-free entry -- owned by FEAT-23.SPEC-002 (Tip Amount Validation); this spec assumes a tip has already passed that validation before its payout/refund behavior applies, per feature-overview.md's Shared Validation section.
- Executing the actual card charge, payout routing call, or refund call to the payment-processing capability -- owned by FEAT-22.SPEC-005 (Balance Charge, Payout Routing & Refund); this spec defines the rule that call must honor for the tip, not the call itself.
- Deciding whether or when a booking is cancelled, or which party's cancellation takes precedence -- owned by FEAT-30 (Pro Booking Management) and FEAT-09 (Cancellation & No-Show Policy Engine); this spec only specifies what must happen to the tip once a Refunded transition is reached.
- Partial or tiered refund of a tip -- excluded per scope-boundaries.md SC-18: the product keeps a single binary refund rule with no partial percentages by timing; this spec follows the same binary "refunded in full" behavior for the tip as for the balance (XBR-23), never a separate tip-specific schedule.

## Governed Entity

**Entity:** Balance Payment
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| amount | number | Price minus deposit, not client-alterable -- **out of scope for this spec**; payout/refund of the balance amount itself is owned by FEAT-22.SPEC-005 |
| tip | number | Optional, non-negative, never pre-selected -- **this spec's field of governance**; value validation is owned by FEAT-23.SPEC-002, this spec governs only what happens to a validated value |
| state | enum | Attempted \| Succeeded \| Failed \| Refunded -- **out of scope for this spec's transitions**; owned by FEAT-22.SPEC-002 and FEAT-22.SPEC-004. This spec reads the Refunded transition as the trigger for the tip's own refund behavior, without owning the transition itself |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-22.SPEC-005 | Balance Charge, Payout Routing & Refund | At capture: routes the tip (if any) to the Pro's Payout Account as part of the balance charge, with zero platform fee. At refund: computes and executes the outbound refund call for the balance plus tip together, when triggered by a cancellation |
| FEAT-30.SPEC-011 | Goodwill Bulk Cancellation & Refund Execution | When a Pro-initiated cancellation or goodwill refund reaches a booking with a tipped, succeeded Balance Payment, this rule's full-refund-including-tip guarantee applies to the refund set it executes |
| FEAT-28.SPEC-005 | Money List Composition & Net Calculation | When composing the Pro's money-list rows and net-per-period figure, a succeeded Balance Payment's `tip` contributes in full, with no fee line deducted, per this rule |

## Field Validation Rules

{This spec defines no field validation rules of its own -- it governs payout and refund behavior for an already-validated `tip` value, not the value's format or sign (owned by FEAT-23.SPEC-002).}

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| tip | No validation rules defined here -- value validity (format, sign, default-free entry) is FEAT-23.SPEC-002's responsibility; this spec addresses only where the value goes and how it is refunded | Always | -- | -- | -- |
| amount | No validation rules defined here -- owned by FEAT-22.SPEC-003 | Always | -- | -- | -- |
| state | No validation rules defined here -- transition validity is owned by FEAT-22.SPEC-002 / FEAT-22.SPEC-004; this spec only reacts to the Refunded transition | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Tip refunded with balance on cancellation | tip, state | When `state` transitions from Succeeded to Refunded, the refund amount executed equals `amount` plus `tip` (if any) in full -- the tip is never partially refunded, retained, or refunded on a separate schedule from the balance (XBR-23) | N/A -- this is an automatic system guarantee enforced at the integration layer (FEAT-22.SPEC-005), not a user-facing validation with a rejectable input |
| No fee deducted from a captured tip | tip | Whenever `tip` is captured as part of a successful Balance Payment, the amount routed to the Pro's Payout Account equals `tip` exactly, with zero deduction (XBR-07) | N/A -- this is an automatic system guarantee, not a user-facing validation |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Receive a captured tip's payout | The Pro (Talia) | Always, automatically, on any of their own bookings with a captured tip -- no action is required to receive it (Full, Payouts) | -- |
| Receive a captured tip's payout | The Client (Riley) | Never -- the Client is the payer, not the payee | No control exists for a Client to route a tip anywhere other than the Pro's Payout Account; this is not an exposed choice |
| Receive a captured tip's payout | Platform Operator (Support) | Never -- Support never receives any funds (View-only access to Payouts, per the Access Matrix) | No path exists; Support's involvement with a tip is limited to viewing its status and amount in the money list |
| View a tip's payout entry on the Pro's own money list | The Pro (Talia) | Always, on their own bookings (Full, Booking & Payment / Payouts) | -- |
| View a tip's payout entry on the Pro's own money list | The Client (Riley) | Own-only -- their own tip amount, as part of their own booking's payment record | -- |
| View a tip's payout entry on the Pro's own money list | Platform Operator (Support) | Always, status and money-list amount only, never bank or identity details (View, per the Access Matrix) | -- |
| Set, waive, or reduce a platform fee on a tip | Any role | Never, for any role -- the zero-fee rule (XBR-07) is a fixed system guarantee, not a permission that can be granted to any role, including the Pro or Platform Operator | No control exists anywhere in the product, for any role, to add or waive a platform fee on a tip; the fee is architecturally always zero and is never surfaced as a configurable choice |
| Trigger a refund of a tip independently of the Balance Payment's own refund | Any role | Never, for any role -- a tip is refunded only as part of the Balance Payment's own Refunded transition (triggered by a booking cancellation, owned by FEAT-30/FEAT-09); no standalone "refund just the tip" action exists | No control exists, for the Pro, the Client, or Platform Operator, to refund a tip on its own; the only refund path is the booking-level cancellation flow that refunds the balance and tip together |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Payout amount attributable to a tip | Equals `tip` exactly, with zero deduction | At capture, when FEAT-22.SPEC-005 routes the successful balance charge (including any tip) to the Pro's Payout Account | No |
| Refund amount attributable to a tip | Equals `tip` exactly, added to the balance's own refund amount, whenever `state` transitions to Refunded | At the moment a cancellation (FEAT-30 or FEAT-09) triggers the Balance Payment's Refunded transition | No |

## Business Rules

- XBR-07: Chairtime's fee on deposits, balances, and tips is always zero; the whole tip passes to the Pro's Payout Account, with the only deduction ever shown being the payment processor's own card fee on the underlying balance charge -- never a Chairtime-attributed fee on the tip itself.
- XBR-23: balance due, and any tip attached to it, is never forfeited and is refunded in full if either party cancels after the balance payment succeeded. This spec is the tip-specific elaboration of that rule; FEAT-22.SPEC-004 owns the equivalent guarantee for the balance amount itself.
- A tip attached to a succeeded Balance Payment is never subject to no-show forfeiture: FEAT-11 (No-Show Marking & Deposit Forfeiture) forfeits only the deposit under the Cancellation Policy; a no-show mark leaves an already-paid balance and its tip completely untouched, per XBR-23's explicit carve-out ("a paid balance ... is never forfeited").
- The refund execution itself (the outbound call to the payment-processing capability) is FEAT-22.SPEC-005's responsibility; this spec defines only the amount and completeness guarantee that call must honor for the tip.
- A card-issuer dispute on a tipped balance payment (FEAT-16, XBR-22) flags and evidences the whole disputed amount, tip included; this spec does not alter dispute handling -- the tip is simply part of the disputed Balance Payment, with no separate dispute path of its own.

## Edge Cases

- **Booking cancelled after a tip is captured but before the Pro's next payout cycle disburses it** -- The refund guarantee applies regardless of payout timing: the refund is executed against the original charge (via FEAT-22.SPEC-005's outbound call), not clawed back from a payout that may or may not have occurred yet.
- **No-show is marked on a booking with an already-paid, tipped balance** -- The no-show mark (FEAT-11) forfeits only the deposit under the Cancellation Policy; the balance and its tip remain paid, untouched, and are never retroactively forfeited by a no-show mark.
- **A card-issuer dispute is opened on a tipped balance payment** -- The full disputed amount, tip included, is flagged and evidenced per FEAT-16/XBR-22; the tip carries no separate dispute treatment, and Chairtime does not rule on the dispute's outcome (SC-17).
- **The Pro's Payout Account is disconnected or in an Action Required state at the moment a tip must be refunded** -- The refund reverses the original client charge and does not depend on the current state of the Pro's Payout Account; a payout-account issue never blocks or delays a client's refund.
- **A tip of exactly the smallest possible non-zero amount is captured and then refunded** -- Refunded in full, identically to any other tip amount; no minimum-refund threshold is defined that would treat a small tip differently.
- **Two cancellation attempts race on a booking with a tipped, succeeded balance payment** -- Governed by FEAT-22.SPEC-004's and the Booking entity's own contention resolution (reject-with-refresh, per the dependency map): the first committed cancellation triggers the one refund, which includes the tip in full; a second, later attempt sees the booking already in its post-cancellation state.

## Acceptance Criteria

**FEAT-23.SPEC-003-AC-01:** Given Riley's tipped balance payment of a validated tip amount succeeds, when the payout routes to Talia's Payout Account, then the tip amount is included in full with zero platform fee deducted.

**FEAT-23.SPEC-003-AC-02:** Given Talia views her money list after receiving a tipped balance payment, when she looks at that entry, then the tip amount appears in full with no fee line against it.

**FEAT-23.SPEC-003-AC-03:** Given a booking with a succeeded, tipped balance payment is cancelled by either Talia or Riley, when the cancellation is committed, then the Balance Payment transitions to Refunded and the refund amount equals the balance plus the tip, in full.

**FEAT-23.SPEC-003-AC-04:** Given a booking with a succeeded, tipped balance payment is marked as a no-show instead of cancelled, when the no-show mark is applied, then only the deposit is forfeited under the Cancellation Policy, and the balance and its tip remain paid and untouched.

**FEAT-23.SPEC-003-AC-05:** Given Talia (the Pro) looks for any setting to take a percentage of a tip as a platform fee, then no such control exists anywhere in her account -- the fee is always zero.

**FEAT-23.SPEC-003-AC-06:** Given Talia attempts to refund only the tip portion of a succeeded Balance Payment without cancelling the booking, then no such standalone action exists -- a tip can only be refunded as part of the Balance Payment's own booking-level refund.

**FEAT-23.SPEC-003-AC-07:** Given Platform Operator (Support) views a Pro's money list, when they look at a tipped entry, then they see its status and amount only, with no bank or identity detail and no ability to trigger or alter its refund.

**FEAT-23.SPEC-003-AC-08:** Given a card-issuer dispute is opened on a tipped balance payment, when the dispute is flagged (FEAT-16), then the full disputed amount, including the tip, is included in the evidence summary, with no separate tip-specific dispute treatment.

**FEAT-23.SPEC-003-AC-09:** Given a tipped balance payment is refunded while Talia's Payout Account is in an Action Required state, when the refund is executed, then it proceeds against the original charge regardless of the Payout Account's current status.

**FEAT-23.SPEC-003-AC-10:** Given a tipped, succeeded Balance Payment, when Riley attempts to view another Client's tip amount on a different booking, then she cannot -- her view is limited to her own booking's tip (Own-only).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 (all "no validation defined here") | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 8 | 8 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
