---
document_type: feature-overview
feature_number: FEAT-23
feature_name: Tipping at Checkout
feature_slug: tipping-at-checkout
priority_tier: Nice-to-Have
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 3
screen_count: 1
automation_count: 0
logic_rule_count: 2
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Tipping at Checkout

## Summary

**Feature:** Tipping at Checkout
**ID:** FEAT-23
**Description:** A client can optionally add a tip when paying in-app (at deposit or, once available, at balance payment), which passes through to the Pro.
**Priority:** Nice-to-Have
**Phase:** Later
**Type:** User-Facing
**Rationale:** BRIEF.md's Open Questions asks directly "where does tipping fit, if anywhere?" As Visionary judgment: tipping has no bearing on the core no-show/deposit problem the product exists to solve, and depends on in-app balance payment (FEAT-22) to be meaningful (tipping on a deposit alone is an unusual pattern). Phased to Later.

**Key Capabilities:**
- Add an optional tip amount at in-app payment time
- See tips reflected in the Pro's own payment records

**Reading of the Description vs. the rest of Stage 2 (flagged, not resolved):** The Description names both "deposit" and "balance payment" as places a tip could be added, but Connected Entities (Balance Payment — update tip amount), Primary Flows & Alternates (tip offered "at balance payment"), and Rationale ("tipping on a deposit alone is an unusual pattern") all place the tip on the Balance Payment only, and the dependency map's Balance Payment entity carries the tip field while the Deposit Transaction entity carries none. This Brief elaborates the Stage 2 decision as recorded in Connected Entities and Primary Flows — tipping attaches only to the in-app balance payment (FEAT-22) — and does not invent a Deposit Transaction tip field. See Non-Goals.

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-23.SPEC-001 | Tip Selection | Screen | The Client (Riley) | Client is offered an optional, non-defaulted tip amount as a step within the in-app balance payment flow, or skips it entirely |
| FEAT-23.SPEC-002 | Tip Amount Validation | Logic/Rule | The Client (Riley) | Governs the tip amount's own constraints: non-negative when given, and never pre-selected to a default |
| FEAT-23.SPEC-003 | Tip Payout & Refund Rule | Logic/Rule | The Client (Riley), The Pro (Talia) | Governs where a given tip's money goes and what happens to it on cancellation: the whole tip passes to the Pro's payout account with no platform cut, and is refunded in full with the balance if the appointment is cancelled |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Add an optional tip amount at in-app payment time | FEAT-23.SPEC-001, FEAT-23.SPEC-002 | SPEC-001 presents the optional tip step (amount entry or skip) inside FEAT-22's balance payment flow; SPEC-002 enforces the amount's own constraints | Phase 2 (Explicit) |
| See tips reflected in the Pro's own payment records | FEAT-23.SPEC-003 (data); FEAT-28's money list screen (display, cross-feature) | SPEC-003 establishes the tip as a field on the Balance Payment record, routed to the Pro's payout account with no fee deduction; FEAT-28's existing money-list screen (Pro Booking Management / Payout Visibility) displays it — this feature adds no screen of its own for the Pro side, per the dependency map's navigation slice ("tips are seen by the Pro in their payment records ... FEAT-12 navigation -> FEAT-28, money list") | Phase 3 (Entity-Lifecycle) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-23.SPEC-002 | Tip Amount Validation | Phase 5 (Rule-Constraint Discovery) | Validation & Limits names two standing constraints on the tip amount (non-negative; never pre-selected) that apply across the flow and would otherwise be silently assumed inline |
| FEAT-23.SPEC-003 | Tip Payout & Refund Rule | Phase 5 (Rule-Constraint Discovery) | Validation & Limits' money-flow and refund sentences are conditional, cross-entity rules (payout destination; refund tied to cancellation state) that participate in cross-feature rules XBR-07 and XBR-23 — they exceed the "simple inline validation" threshold and need a standalone rule spec other specs (this feature's and FEAT-28's, FEAT-30's) can reference precisely |

## Entity-Lifecycle Coverage Matrix

**Entity: Balance Payment**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | N/A | The Balance Payment record itself is created by FEAT-22 (In-App Balance Payment); FEAT-23 never creates one on its own — it only adds a field to a record another feature originates | Dependency map: "Created by FEAT-22 (v1)" |
| Read (single) | FEAT-23.SPEC-001 | Tip Selection reads the in-progress Balance Payment's context (amount due) to present the tip step alongside it | -- |
| Read (list) | N/A | This feature has no list screen of its own; the tip, once recorded, is read as part of FEAT-28's money list (cross-feature) | See Cross-Feature Touchpoints |
| Update | FEAT-23.SPEC-001, FEAT-23.SPEC-002 | The client's chosen tip amount, once validated by SPEC-002, is written onto the Balance Payment record as part of FEAT-22's payment submission | Dependency map: "Updated by FEAT-23 (tip amount, Later)" |
| Delete/Archive | N/A | Balance Payment is a financial record retained for the life of the account with no delete path (dependency map: "Deleted: N/A — financial record retained"); this is an explicit non-goal of the owning feature (FEAT-22), not a FEAT-23 omission | See Non-Goals |
| State Transition | N/A | The tip field carries no state of its own; Balance Payment's own state machine (Attempted \| Succeeded \| Failed \| Refunded) is owned by FEAT-22 and FEAT-30 — FEAT-23's rule (SPEC-003) only specifies what must happen to the tip *when* that state reaches Refunded | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Booking | FEAT-23.SPEC-001 | Tip Selection is shown within the balance payment step of an existing Booking; the Booking itself is read, never written, by this feature (Connected Entities: Booking (read)) |
| Payout Account | FEAT-23.SPEC-003 | The rule names the Pro's Payout Account as the tip's destination (whole amount, no platform cut) without this feature managing the account itself |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Client reaches the balance payment step of an existing Booking | Client is offered an optional tip amount, never pre-selected | Inline in triggering screen (FEAT-22's balance payment flow hosts the step) | FEAT-23.SPEC-001 |
| Client enters a tip amount and submits balance payment | Tip amount is checked against its own constraints (non-negative, no default) before being attached to the payment | Standalone Logic/Rule | FEAT-23.SPEC-002 |
| Client skips tipping | Balance payment proceeds unaffected; no tip field is set; this must never feel like a required step or block the payment | Inline in triggering screen | FEAT-23.SPEC-001 |
| Client pays their balance in person instead of in-app | The tipping surface never appears; no Balance Payment record (and so no tip) is ever involved | Inline / N/A — outside SPEC-001's trigger conditions, a fully normal path per the Stage 2 alternate | FEAT-23.SPEC-001 |
| Client's tipped balance payment succeeds | The tip amount routes in full to the Pro's payout account; no platform cut is taken or shown (XBR-07) | Standalone Logic/Rule | FEAT-23.SPEC-003 |
| Booking is cancelled by either party after a tipped balance payment already succeeded | The tip is refunded in full together with the balance, never forfeited (XBR-23) | Cross-feature — the refund is executed by FEAT-30's / FEAT-09's refund handling; SPEC-003 defines the rule that refund must honor for the tip | FEAT-23.SPEC-003 / FEAT-30 |
| Client completes a balance payment that includes a tip | The tip amount is reflected in the existing payment confirmation, not a separate message (Communications: N/A) | Inline in FEAT-22's confirmation screen (cross-feature) | FEAT-22 responsibility |
| Pro opens their own payment records | Tips received appear on their bookings, with no deduction labeled as a platform fee | Cross-feature — display is owned by FEAT-28's money list screen | FEAT-28 responsibility |
| Balance payment (including any tip) needs a client card charge | The tip is charged as part of FEAT-22's balance-payment charge, never a separate charge | Cross-feature — delivered via FEAT-22's balance-charge Integration spec (payment-processing capability, ASMP-31); no separate Integration spec is needed because FEAT-23 introduces no distinct external interaction of its own | FEAT-22 responsibility |

## Shared Context

**Shared Entities:**
- Balance Payment -- read by SPEC-001 (context for the tip step), updated by SPEC-001/SPEC-002 (writing the validated tip amount), governed by SPEC-003 (payout destination and refund rule for the tip field). Relevant field: `tip` — optional, non-negative, never pre-selected (per the dependency map's Balance Payment field list).
- Booking -- read-only context for SPEC-001 (the balance payment step belongs to one Booking).
- Payout Account -- named only as the tip's destination in SPEC-003; not created, read, updated, or deleted by this feature.

**Shared UI Patterns:**
- Tip step embedding -- SPEC-001 is not a standalone, separately navigable screen; it is a step/section rendered inside FEAT-22's balance payment screen. The Spec Writer for SPEC-001 should describe it as an addition to that flow (entry and exit points inherited from FEAT-22), not as a screen with its own route.

**Shared Validation:**
- SPEC-002 defines the tip amount's own constraints. SPEC-001 references SPEC-002 for validation behavior rather than restating the rule, and SPEC-003 assumes SPEC-002 has already passed before its payout/refund rule applies.

## Internal Dependency Map

```
SPEC-001 (Tip Selection) -> [client enters a tip amount] -> SPEC-002 (Tip Amount Validation) -> [passes] -> SPEC-001 (submission proceeds)
SPEC-001 (Tip Selection) -> [client submits balance payment with a validated tip] -> SPEC-003 (Tip Payout & Refund Rule)
SPEC-003 (Tip Payout & Refund Rule) -> [booking cancelled after a tipped payment succeeded] -> FEAT-30 (Cancellation & Refund Handling, cross-feature)
SPEC-003 (Tip Payout & Refund Rule) -> [tipped payment succeeds] -> FEAT-28 (Payout Account Connection & Payout Visibility, cross-feature)
```

**Default Entry:** N/A -- this feature has no navigation entry point of its own. The dependency map records no navigation connection naming FEAT-23; SPEC-001 surfaces only as a step inside FEAT-22's in-app balance payment flow, reached when a client who is already viewing their own booking chooses to pay the balance in-app.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-23.SPEC-001 | Inbound | FEAT-22 (In-App Balance Payment) | Tip Selection is embedded as a step inside FEAT-22's balance payment screen, not a separately reached screen | Client proceeds to pay their balance in-app |
| FEAT-23.SPEC-002 | Inbound | FEAT-22 (In-App Balance Payment) | The tip amount, once validated, is submitted together with FEAT-22's balance-charge Integration spec's payment-processing capability (ASMP-31); FEAT-23 defines no separate charge path | Client submits balance payment including a tip |
| FEAT-23.SPEC-003 | Outbound | FEAT-28 (Payout Account Connection & Payout Visibility) | Once attached to a succeeded Balance Payment, the tip is displayed in the Pro's money list with no fee line, per XBR-07 | Pro opens their payment records |
| FEAT-23.SPEC-003 | Outbound | FEAT-30 (Cancellation & Refund Handling) | The tip is refunded in full together with the balance whenever either party cancels after a tipped balance payment already succeeded, per XBR-23 | Booking cancelled after a tipped balance payment succeeded |

## Non-Functional Notes

**Data volumes / growth:** N/A — tipping adds a single optional field (`tip`) to the existing Balance Payment record; it introduces no new entity and no independent record stream, so it carries no growth profile beyond FEAT-22's own Balance Payment volume. Captured: optional tip amount. Displayed: tips received, on the Pro's payment records (product-features.md, Data Notes). Three analytics signals instrument the flow -- `tip_offered_shown`, `tip_added`, `tip_skipped` (product-features.md, Signals) -- and are recorded on the existing balance-payment event stream, not a new one.

**Responsiveness:** Tip entry is a single step inside FEAT-22's existing payment flow and inherits its responsiveness and correctness expectations rather than setting its own: the system must never silently drop a tip a client believed was applied (ASMP-26's correctness bar), and the step must remain readable and fully operable at phone width inside the in-app browser, including for screen-reader users (ASMP-28).

**Data sensitivity / privacy:** Financial data only — a tip amount with no card data, consistent with the Balance Payment entity's classification (dependency map slice: "Financial — amounts and outcomes only; no card data," ASMP-15) and the product-wide hard boundary that card data is never stored or handled by the product's own code (scope-boundaries.md SC-11).

**Compliance flags:** N/A — tipping introduces no compliance obligation beyond the category-level payment-processing dependency (ASMP-31) already carried by FEAT-22's and FEAT-28's Integration specs; no health or financial regulatory regime applies specifically to a tip amount.

## Non-Goals

- **Tipping at deposit payment** -- Excluded per this Brief's reading of the Stage 2 record: although the feature's Description sentence names "deposit or ... balance payment," the Connected Entities (Balance Payment only), Primary Flows & Alternates (tip offered "at balance payment"), and Rationale ("tipping on a deposit alone is an unusual pattern") all place the tip exclusively on the balance payment, and the dependency map's Deposit Transaction entity carries no tip field. This discrepancy in the Description's wording is flagged here, not resolved, per this feature's context package instructions.
- **A default or pre-selected tip amount** -- Excluded per product-features.md's Validation & Limits: the tip is "never pre-selected to a default that could feel presumptive." This is a named Stage 2 decision, not an Analyst simplification.
- **Any platform cut on tips** -- Excluded per cross-feature rule XBR-07: "Chairtime's fee on deposits, balances and tips is always zero." The whole tip passes to the Pro's payout account.
- **In-person balance payment as a tipping surface** -- Excluded per scope-boundaries.md SC-16: at MVP (and still as an equally valid path once FEAT-22 ships) the balance may be settled in person, off-platform, by whatever means the Pro already uses; tipping only exists on the in-app path this feature's Screen spec covers, and in-person payment is unaffected by it.
- **Partial or tiered refund of a tip on cancellation** -- Excluded per scope-boundaries.md SC-18: the product keeps a single binary refund rule (full refund outside the cancellation window, kept inside it or on a no-show) with no partial percentages by timing; FEAT-23.SPEC-003 follows the same binary "refunded in full" behavior for the tip, tied to the balance's own refund outcome (XBR-23), rather than defining a separate tip-specific refund schedule.
