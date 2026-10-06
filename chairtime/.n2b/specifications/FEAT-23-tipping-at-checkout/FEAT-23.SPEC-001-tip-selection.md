---
document_type: spec
spec_type: screen
spec_id: FEAT-23.SPEC-001
spec_name: Tip Selection
spec_slug: tip-selection
parent_feature: FEAT-23
parent_feature_name: Tipping at Checkout
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

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
