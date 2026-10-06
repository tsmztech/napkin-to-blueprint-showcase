# FEAT-23 — Tipping at Checkout

This chapter covers Tipping at Checkout (FEAT-23), a Nice-to-Have-tier feature. It carries 3 specifications carrying 30 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-23.SPEC-001 | Tip Selection | screen | 11 |
| FEAT-23.SPEC-002 | Tip Amount Validation | logic-rule | 9 |
| FEAT-23.SPEC-003 | Tip Payout & Refund Rule | logic-rule | 10 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Tip Selection

## Overview

**Name:** Tip Selection
**ID:** FEAT-23.SPEC-001
**Type:** Screen
**Purpose:** The Client is offered an optional, never-pre-selected tip amount as a step embedded inside the in-app balance payment flow, and can enter an amount or skip it without the underlying payment ever being blocked.
**Parent Feature:** FEAT-23 -- Tipping at Checkout

## Scope and Non-Goals

**In Scope:**
- The tip-entry step and its "no tip" skip path, rendered as a section inside FEAT-22.SPEC-001's (Balance Payment) screen
- Presenting the tip amount input with no default or pre-selected value
- Displaying that the whole tip goes to the Pro, with no platform cut
- Handing the client's chosen tip value (or its absence) to FEAT-22.SPEC-001's own payment submission, after FEAT-23.SPEC-002 validates it

**Non-Goals:**
- Tipping at deposit payment -- excluded per this feature's Brief (feature-overview.md, Summary: "Reading of the Description vs. the rest of Stage 2"): Stage 2's Connected Entities, Primary Flows, and Rationale all place the tip exclusively on the balance payment; Deposit Transaction carries no tip field.
- Its own standalone payment submission, processing indicator, decline handling, or success state -- these belong entirely to FEAT-22.SPEC-001, the screen this step is embedded in; this spec defines only the tip-entry section within that screen, per the Brief's Shared UI Patterns ("Tip step embedding").
- Enforcing the tip amount's own constraints (non-negative, no default) -- owned by FEAT-23.SPEC-002 (Tip Amount Validation); this spec references it rather than restating the rule.
- Defining where the tip's money goes or what happens to it on cancellation -- owned by FEAT-23.SPEC-003 (Tip Payout & Refund Rule).
- In-person balance payment as a tipping surface -- excluded per scope-boundaries.md SC-16: tipping exists only on the in-app path this spec covers; a client who pays their balance in person never encounters this step.

## Entry Points

{This screen has no navigation entry point of its own; it is a step rendered inside FEAT-22.SPEC-001 (Balance Payment), reached only through that screen's own entry, never as a separately addressable destination.}

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-22.SPEC-001 (Balance Payment) | Client reaches the balance payment step of an existing confirmed Booking and the payment screen renders | The Booking's current balance-due amount (computed by FEAT-22.SPEC-003), so the tip step can present alongside it |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Full tip-entry section, rendered within their own Balance Payment screen | Enter a tip amount, clear it, or explicitly skip; proceed with the host screen's payment submission either way | -- |
| The Pro (Talia) | Not shown -- this step exists only inside a Client's own in-app balance payment flow, which the Pro never opens on the Client's behalf | No | No control or path exists in any Pro-facing screen that reaches this step; the Pro instead sees tips already received in their own payment records (FEAT-28's money list) |
| Platform Operator (Support) | Not shown -- Support's read access covers account status and records, never a Client's live in-progress payment screen | No | No path exists; Support's view of a tip is limited to the resulting Balance Payment record's tip field through FEAT-28's money list (View only, per the Access Matrix), never this live entry step |
| Unauthenticated | No | No | Same as FEAT-22.SPEC-001: without a valid access link the client never reaches the Balance Payment screen this step lives inside, and is shown the "request a new link" prompt (FEAT-06) instead |
| Expired session | No | No | Same as FEAT-22.SPEC-001: an expired or already-used access link shows the "request a new link" prompt; any tip amount typed but not yet submitted is lost, since this step holds no persistence of its own beyond the host screen's current session |

## Layout and Content

This step renders as a section inside FEAT-22.SPEC-001's Balance Payment screen, positioned below the deposit-paid-vs-balance-remaining summary and above the host screen's own Pay action.

**Tip section:** A section heading reading "Add a tip? (optional)", below which sits:
- A single numeric amount input, in the Pro's account currency, left empty with a placeholder showing the currency symbol only (e.g., "$") and no pre-filled figure of any kind
- A single line of supporting text below the input stating that the whole amount goes to the Pro, with no platform fee taken (grounded in XBR-07)
- A "No tip, continue" text-style control positioned to the right of, or immediately below, the amount input, for a client who wants to skip tipping without touching the input at all

No amount is ever shown as selected or highlighted before the client acts -- the input and the skip control carry equal visual weight, so neither reads as the expected or default path.

**Footer:** None of its own -- the tip section sits above FEAT-22.SPEC-001's own Pay action, which remains the single submission control for the whole balance payment (tip included).

### Responsive Behavior

- **Compact breakpoint:** The tip section stacks as described above, full width, matching the host screen's single-column layout.
- **Medium size class and above:** The tip section remains single-column within the host screen's own content width; no structural change beyond the width the host screen already applies.
- **Amount input:** Uniform scaling, no structural change, across all size classes.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Tip amount input | Type a numeric value | Captures the entered amount as a candidate tip | Input shows the typed value; "No tip, continue" control remains available alongside it | Standard input focus state |
| Tip amount input | Blur with a value entered | Triggers tip validation via FEAT-23.SPEC-002 | Valid: input shows the entered amount normally. Invalid: input enters an error state | Valid: no message. Invalid: field-level error message from FEAT-23.SPEC-002 shown below the input |
| "No tip, continue" control | Tap | Clears any amount typed in the tip input and marks the tip as skipped for this attempt | Tip input returns to its empty placeholder state; visual focus moves to the host screen's Pay action | Brief confirmation text "No tip added" replaces the tip section's supporting line until the client types in the input again |
| Host screen's Pay action (FEAT-22.SPEC-001) | Tap | 1. Whatever value is currently in the tip input (or none, if skipped/blank) is validated via FEAT-23.SPEC-002. 2. If valid, the amount is handed to FEAT-22.SPEC-001's own payment submission, which proceeds exactly as it would with no tip. 3. If invalid, submission is blocked and the tip input shows its error state. | Same state changes as any FEAT-22.SPEC-001 Pay tap, gated by tip validation succeeding first | Invalid tip: focus moves to the tip input with its error message; the underlying balance payment is never submitted with an invalid tip |

### Accessibility Notes

- **Focus order:** (within the host screen's own order) ... -> deposit-paid/balance-remaining summary -> tip section heading -> tip amount input -> "No tip, continue" control -> host screen's Pay action.
- **Validation announcements:** When the tip amount input enters an error state, its error message is announced to assistive technology and programmatically associated with the input, consistent with FEAT-22.SPEC-001's own field-error pattern.
- **Skip announcement:** Activating "No tip, continue" announces "No tip added" to assistive technology as the supporting line updates.
- **Keyboard alternatives:** Every control in this section (amount input, skip control) is reachable and operable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Untouched (default) | Amount input empty with currency-symbol-only placeholder; supporting "whole tip goes to the Pro" line shown; no value selected or highlighted | Balance Payment screen first renders with this section | Client types in the amount input or taps "No tip, continue" |
| Amount entered | Input shows the typed value | Client types a non-empty value | Client clears the input, taps "No tip, continue", or the host screen's Pay action is tapped |
| Skipped | Input empty; supporting line reads "No tip added" | Client taps "No tip, continue" | Client types a new value in the input |
| Validation error | Input shows an error outline with FEAT-23.SPEC-002's exact error message below it | FEAT-23.SPEC-002 rejects the entered value on blur or on the host screen's Pay tap | Client corrects the value and it re-validates successfully |
| Loading | N/A -- this section has no independent fetch of its own; it renders inline with the balance figure that FEAT-22.SPEC-001 supplies, so it inherits that host screen's own loading state in full (the tip input is not shown until the host screen's balance load completes) | -- | -- |
| Offline/Degraded | N/A -- this section inherits FEAT-22.SPEC-001's own connectivity requirement in full; when the host screen shows its offline state, the whole payment step (tip section included) is unavailable exactly as FEAT-22.SPEC-001 defines, with no separate offline behavior of its own | -- | -- |

## Validation Rules

Validation governed by FEAT-23.SPEC-002 (Tip Amount Validation). See that spec for the tip amount's field-level rules. This step applies validation on input blur and again when the host screen's Pay action is tapped.

## Navigation Out

{This screen has no navigation out of its own -- it is a step inside FEAT-22.SPEC-001 and inherits that screen's navigation entirely.}

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Host screen's Pay action succeeds (with or without a tip) | Wherever FEAT-22.SPEC-001 navigates on a successful balance payment | FEAT-22 |
| Host screen's Pay action fails (decline) | FEAT-22.SPEC-001's own decline state (this section's entered/skipped tip state is preserved for retry) | FEAT-22 |

## Data Model

**Creates:** None -- this spec creates no record of its own.
**Reads:** Booking -- the confirmed Booking's current balance-due context, read via FEAT-22.SPEC-001, so the tip step renders alongside the correct balance figure.
**Updates:** Balance Payment.tip -- this spec is the source of the tip value the client chooses; the value is written onto the Balance Payment record at the moment FEAT-22.SPEC-002 creates that record on a successful capture (per feature-dependency-map.md's Balance Payment lifecycle: "Updated by FEAT-23 (tip amount, Later)"). This spec never writes the record directly.
**Deletes:** None.

## Business Rules

- Tip validation (FEAT-23.SPEC-002) runs before any tip value is handed to the host screen's payment submission -- the client cannot submit an invalid tip.
- XBR-07: the whole tip passes to the Pro's payout account with no platform cut; the supporting line under the amount input states this plainly.
- XBR-23: a tip is refunded in full together with the balance if either party cancels after a tipped balance payment succeeds -- this behavior is governed and enforced by FEAT-23.SPEC-003, not by this screen.
- Skipping the tip step never blocks or delays the underlying balance payment -- the host screen's Pay action behaves identically whether the tip is present, zero, or skipped.

## Edge Cases

- **Client types a tip amount, then navigates away and returns to the Balance Payment screen before submitting** -- The tip section resets to Untouched; no draft tip amount is preserved, consistent with FEAT-22.SPEC-001 not persisting unsubmitted payment attempts across visits.
- **Client taps "No tip, continue" after already typing an amount** -- The typed amount is discarded and the section shows Skipped; a client who wants to tip after skipping simply types in the input again, which clears the "No tip added" line.
- **Client's balance payment is declined after entering a tip** -- The tip amount the client entered is preserved in the input for their retry, exactly as FEAT-22.SPEC-001 preserves the rest of the payment context on a decline; the tip is validated again on retry.
- **Client taps the host screen's Pay action twice rapidly with a tip entered** -- The second tap is ignored while the first submission (tip validation plus payment) is in progress, per FEAT-22.SPEC-001's own double-submit prevention; no duplicate tip or charge results.
- **Concurrent-edit conflict on the Balance Payment record** -- N/A for this screen: this step never loads or displays an existing Balance Payment record to edit (at most one Balance Payment exists per Booking, created only on a successful capture); the only shared entity read here is the Booking's balance-due context, which FEAT-22.SPEC-001 already resolves for its own Contention handling. This spec introduces no conflict surface of its own.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-22.SPEC-001 (Balance Payment) | Navigation (inbound); References (bidirectional) | This step is embedded inside FEAT-22.SPEC-001's screen; the host screen's Pay action carries this step's validated tip value into its own payment submission |
| FEAT-23.SPEC-002 (Tip Amount Validation) | References (outbound) | Tip amount validation on blur and on submit |
| FEAT-23.SPEC-003 (Tip Payout & Refund Rule) | References (outbound) | Governs where a submitted tip's money goes and its refund behavior; not enforced by this screen directly |

## Analytics and Success Signals

{No metric in success-metrics.md is connected to Tipping at Checkout (FEAT-23) -- the feature's Connected Feature entries in success-metrics.md list only the 24 MVP/v1 features; Tipping at Checkout, phased Later, carries no dedicated Stage 2 success metric. The events below are named functionally from product-features.md's Signals field for this feature and are recorded on the existing balance-payment event stream (feature-overview.md, Non-Functional Notes), but each cites N/A per the "N/A -- {reason}" allowance since no success-metrics.md entry exists to join to.}

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| tip_offered_shown | none beyond the standard event envelope | The tip section first renders as part of the Balance Payment screen | N/A -- no success-metrics.md metric is connected to Tipping at Checkout (FEAT-23); this Later-phase feature has no assigned Stage 2 metric |
| tip_added | tip amount | The client's chosen tip amount passes validation and is included in a submitted balance payment | N/A -- no success-metrics.md metric is connected to Tipping at Checkout (FEAT-23); this Later-phase feature has no assigned Stage 2 metric |
| tip_skipped | none beyond the standard event envelope | The client taps "No tip, continue," or submits the balance payment with the tip input left empty | N/A -- no success-metrics.md metric is connected to Tipping at Checkout (FEAT-23); this Later-phase feature has no assigned Stage 2 metric |

## Acceptance Criteria

**FEAT-23.SPEC-001-AC-01:** Given Riley is on the Balance Payment screen (FEAT-22.SPEC-001), when the tip section first renders, then the amount input is empty with only a currency-symbol placeholder, and no amount is pre-selected or highlighted.

**FEAT-23.SPEC-001-AC-02:** Given Riley is on the Balance Payment screen, when she types "15" into the tip amount input and moves focus away, then FEAT-23.SPEC-002 validates the value, and since it is non-negative the input shows "15" with no error.

**FEAT-23.SPEC-001-AC-03:** Given Riley has typed "-5" into the tip amount input and moves focus away, when FEAT-23.SPEC-002 validation runs, then the input shows an error state with the exact message from FEAT-23.SPEC-002, and the underlying balance payment is not submitted.

**FEAT-23.SPEC-001-AC-04:** Given Riley is on the Balance Payment screen with the tip input untouched, when she taps "No tip, continue," then the section shows "No tip added," and taping the host screen's Pay action submits the balance payment with no tip.

**FEAT-23.SPEC-001-AC-05:** Given Riley has typed a tip amount and then taps "No tip, continue," then the typed amount is discarded and the section returns to the Skipped state.

**FEAT-23.SPEC-001-AC-06:** Given Riley enters a valid tip amount and taps the host screen's Pay action, when the balance payment succeeds, then the tip amount is included in the payment exactly as entered, with the supporting line having stated it goes entirely to the Pro.

**FEAT-23.SPEC-001-AC-07:** Given Riley's balance payment (with a tip entered) is declined, when the decline state shows, then her entered tip amount remains in the input for her retry.

**FEAT-23.SPEC-001-AC-08:** Given Riley taps the host screen's Pay action twice in rapid succession while a tip is entered, then the second tap has no effect while the first submission is in progress, and no duplicate charge results.

**FEAT-23.SPEC-001-AC-09:** Given Talia (the Pro) is signed in to her own account, when she looks for any way to set or preview a tip on a Client's in-progress balance payment, then no such control exists anywhere in her account.

**FEAT-23.SPEC-001-AC-10:** Given Platform Operator (Support) is viewing a Pro's account after a help request, when they look for this tip-entry step, then it is not shown -- Support's visibility into a tip is limited to the resulting record in FEAT-28's money list.

**FEAT-23.SPEC-001-AC-11:** Given a visitor without a valid access link attempts to reach a booking's balance payment, when the link is checked, then they see the "request a new link" prompt (FEAT-06) and never reach this tip step.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 4 (untouched, amount entered, skipped, validation error) + offline (N/A, inherited) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |



# Logic/Rule Spec: Tip Amount Validation

## Overview

**Name:** Tip Amount Validation
**ID:** FEAT-23.SPEC-002
**Type:** Logic/Rule
**Purpose:** Governs the tip amount's own constraints on the Balance Payment record's `tip` field -- non-negative when given, valid as a monetary amount, and never pre-selected to a default -- independent of the balance amount's own rules, which FEAT-22 owns.
**Parent Feature:** FEAT-23 -- Tipping at Checkout
**Governed Entity:** Balance Payment (the `tip` field only)

## Scope and Non-Goals

**In Scope:**
- Field validation for the `tip` field: format, sign, and when it is checked
- The rule that `tip` is never pre-selected to a default value
- Authorization for who may set, skip, or view a tip amount

**Non-Goals:**
- Validation of the Balance Payment's `amount` field or eligibility preconditions -- owned by FEAT-22.SPEC-003 (Balance Amount & Eligibility Rules); this spec addresses only `tip`.
- Balance Payment state-transition consistency (Attempted / Succeeded / Failed) -- owned by FEAT-22.SPEC-004 (Balance Payment Outcome Consistency & Cancellation Contention).
- Where a validated tip's money goes and its refund behavior on cancellation -- owned by FEAT-23.SPEC-003 (Tip Payout & Refund Rule), which assumes this spec's validation has already passed, per feature-overview.md's Shared Validation section.
- Imposing an upper bound on the tip amount -- excluded because product-features.md's Validation & Limits field for this feature defines only a non-negative floor and the no-default rule; no maximum is named in Stage 2, so none is invented here.

## Governed Entity

**Entity:** Balance Payment
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| amount | number | Price minus deposit, not client-alterable -- **out of scope for this spec**; validated by FEAT-22.SPEC-003 |
| tip | number | Optional, non-negative, never pre-selected -- **governed by this spec** |
| state | enum | Attempted \| Succeeded \| Failed \| Refunded -- **out of scope for this spec**; owned by FEAT-22.SPEC-002 and FEAT-22.SPEC-004 (transitions), and FEAT-23.SPEC-003 (refund behavior tied to the Refunded transition) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-23.SPEC-001 | Tip Selection | On the tip amount input's blur, and again when FEAT-22.SPEC-001's Pay action is tapped (the tip value is validated before being handed to that submission) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| tip | Must be a valid monetary amount in the Pro's account currency (digits and at most one decimal separator, up to 2 decimal places; no letters, currency symbols, or other characters) | When the field is non-empty | On blur, and on the host screen's Pay action | "Enter a valid amount." | Yes |
| tip | Must be non-negative (zero or greater) | When the field is non-empty | On blur, and on the host screen's Pay action | "Tip amount can't be negative." | Yes |
| tip | May be left entirely empty (null) -- absence of a tip is always valid and never itself an error | Always | On blur, and on the host screen's Pay action | -- (no error; empty is a valid, final state, not a pending one) | No |
| amount | No validation beyond data type in this spec | Always | -- | -- (owned by FEAT-22.SPEC-003) | -- |
| state | No validation beyond data type in this spec | Always | -- | -- (owned by FEAT-22.SPEC-002 / FEAT-22.SPEC-004) | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| No cross-field dependency | tip, amount | The `tip` field's validity and value are entirely independent of `amount`: a tip is checked against its own rules regardless of the balance amount, and the balance amount's computation (FEAT-22.SPEC-003) never reads or is altered by `tip` | N/A -- this is a structural independence guarantee, not a rejectable condition |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Set or edit the tip amount on an in-progress balance payment | The Client (Riley) | Only on their own booking's in-progress balance payment (Own-only, per the Access Matrix's Booking & Payment row) | -- |
| Set or edit the tip amount on an in-progress balance payment | The Pro (Talia) | Never | No control exists in any Pro-facing screen to set or edit a tip on a Client's balance payment; the tip is client-chosen only, per product-features.md's Access field for this feature |
| Set or edit the tip amount on an in-progress balance payment | Platform Operator (Support) | Never | No edit control exists in the Support view; Support's access to Booking & Payment is View-only, per the Access Matrix |
| Skip tipping entirely | The Client (Riley) | Always, on their own balance payment | -- |
| View a tip amount already recorded on a completed Balance Payment | The Pro (Talia) | Always, on their own bookings (Full, Booking & Payment) | -- |
| View a tip amount already recorded on a completed Balance Payment | The Client (Riley) | Only their own booking's Balance Payment (Own-only) | -- |
| View a tip amount already recorded on a completed Balance Payment | Platform Operator (Support) | Always, status/amount only, never card data (View, per the Access Matrix) | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| tip | None -- the field starts unset with no default value or pre-selected amount of any kind, per product-features.md's Validation & Limits: "never pre-selected to a default that could feel presumptive" | Every time the tip step (FEAT-23.SPEC-001) is shown, on each fresh balance payment attempt | N/A -- no default exists to override; the Client enters the value directly, or leaves it unset |

## Business Rules

- Field validation on `tip` runs before the value is handed to FEAT-22.SPEC-001's own payment submission -- an invalid tip blocks that submission, per FEAT-23.SPEC-001's Interactions.
- Validation on `tip` applies identically whether the client is entering a first attempt or retrying after a decline -- the product definition establishes no retry-only or first-attempt-only rule for the tip field.
- An empty `tip` field is a fully valid, final choice (equivalent to skipping), never a pending or incomplete state that blocks submission.
- This spec's rules govern the field's own constraints only; XBR-07 (zero platform cut) and XBR-23 (full refund with the balance) are enforced by FEAT-23.SPEC-003, not restated here.

## Edge Cases

- **Tip entered as exactly "0"** -- Passes validation (zero is non-negative). Functionally equivalent to leaving the field empty: no tip is recorded on the Balance Payment, so a client who deliberately types "0" and one who skips entirely produce the same outcome.
- **Tip entered with a negative sign ("-5")** -- Fails validation with "Tip amount can't be negative." before the value ever reaches the host screen's payment submission.
- **Tip entered with more than 2 decimal places ("5.999")** -- Fails validation with "Enter a valid amount."
- **Tip entered as a very large number (e.g., far exceeding the balance amount)** -- Passes validation; no upper bound is defined in Stage 2 for this feature, so none is imposed here (see Non-Goals).
- **Tip field left as whitespace only** -- Treated the same as empty: valid, no tip recorded.
- **Client clears a previously valid tip amount back to empty before submitting** -- Field re-validates as empty (valid, no error); the "no tip" outcome applies exactly as if the client had never typed anything.
- **A Pro attempts, through any means outside the product's own screens, to influence a Client's in-progress tip amount** -- Out of scope for this spec: no Pro-facing control exists to do so (see Authorization Rules); this scenario has no in-product surface to specify further.

## Acceptance Criteria

**FEAT-23.SPEC-002-AC-01:** Given Riley enters "10" in the tip amount input, when she moves focus away, then the value passes validation with no error shown.

**FEAT-23.SPEC-002-AC-02:** Given Riley enters "-3" in the tip amount input, when she moves focus away, then the field shows the error "Tip amount can't be negative." and the balance payment is not submitted while the field remains invalid.

**FEAT-23.SPEC-002-AC-03:** Given Riley enters "12.999" in the tip amount input, when she moves focus away, then the field shows the error "Enter a valid amount."

**FEAT-23.SPEC-002-AC-04:** Given Riley leaves the tip amount input completely empty, when she taps the host screen's Pay action, then no validation error is shown, and the balance payment proceeds with no tip.

**FEAT-23.SPEC-002-AC-05:** Given Riley enters exactly "0" as her tip amount, when validation runs, then it passes as non-negative, and the resulting Balance Payment records no tip, identically to leaving the field empty.

**FEAT-23.SPEC-002-AC-06:** Given the tip amount step first renders for Riley, when she looks at the input, then it shows no pre-filled or highlighted amount of any kind.

**FEAT-23.SPEC-002-AC-07:** Given Talia (the Pro) is signed in to her own account, when she looks for any control to set or edit a tip on a Client's balance payment, then no such control exists anywhere in her account.

**FEAT-23.SPEC-002-AC-08:** Given Platform Operator (Support) is viewing a Pro's account, when they look for a way to edit a tip amount, then no edit control exists -- their access to Booking & Payment is View-only.

**FEAT-23.SPEC-002-AC-09:** Given Riley's balance payment already includes a valid tip of "15" and she changes it to "8" before submitting, then the amount field re-validates against the new value, and "8" is the amount handed to the payment submission if it passes.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |



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

