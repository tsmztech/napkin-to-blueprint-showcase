---
document_type: spec
spec_type: screen
spec_id: FEAT-18.SPEC-002
spec_name: Billing & Subscription Management Screen
spec_slug: billing-subscription-management-screen
parent_feature: FEAT-18
parent_feature_name: Pro Subscription Billing & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Billing & Subscription Management Screen

## Overview

**Name:** Billing & Subscription Management Screen
**ID:** FEAT-18.SPEC-002
**Type:** Screen
**Purpose:** Talia's ongoing view of her subscription's plan status and next billing date, with actions to update her payment method or cancel; Support's read-only entry point into the same status for billing support questions.
**Parent Feature:** FEAT-18 -- Pro Subscription Billing & Account Management

## Scope and Non-Goals

**In Scope:**
- Displaying current plan status, next billing date, and the price/zero-fee statement block
- Updating the payment method on file
- Cancelling the subscription (effective at the end of the current billing period)
- Support's view-only read of the same status

**Non-Goals:**
- Choosing among multiple price tiers or plans -- excluded per product-features.md FEAT-18 (Validation & Limits): one price tier only in v1
- The first-time subscribe flow -- owned by FEAT-18.SPEC-001 (Subscribe Screen); this screen exists only for a Pro who already has a Subscription record
- Support editing, cancelling, or changing payment details on the Pro's behalf -- excluded per scope-boundaries.md SC-05: Platform Operator (Support) has view-only access and can never act on a Pro's subscription
- Closing the Pro's account or deleting its data -- owned by FEAT-29 (Pro Sign-In & Account Lifecycle); this screen's cancel action only cancels the subscription, per the Feature Breakdown Brief's Side-Effect Inventory

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) (Pro Sign-In & Account Lifecycle, account settings) | Pro opens billing from account settings | The Pro's account reference |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) (Platform Support Read-Only Access) | Support opens a Pro's account during a help request | The Pro account reference being supported; screen renders in Support's view-only mode |
| FEAT-18.SPEC-007 (Subscription Billing Notifications) | Pro taps a billing notification's CTA (payment-failure grace notice, renewal receipt, cancellation confirmation, price-change notice) | The Pro's account reference; screen opens directly to this view |
| FEAT-27.SPEC-004 (Pause Bookings) | Talia taps "Resolve it in Billing" on the Pause Bookings screen while a subscription lapse pause is in force | The Pro's account reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Update payment method, cancel subscription | -- |
| The Client (Riley) | No | No | Clients have no visibility into this at all, per the Access Matrix (Subscription & Billing = None); there is no client-facing entry point |
| Platform Operator (Support) | Plan status and next billing date only (price, status, billing cycle) | No actions -- the update-payment-method and cancel controls are not shown | Attempting to reach an action control directly is not possible -- the interface offers no edit controls in Support's view by design, consistent with FEAT-19's read-only mode; Support never sees payment-method reference details beyond that a method is on file |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no unsaved billing action was in progress that needs preserving, since actions on this screen (payment update, cancel) complete atomically through the payment-processing capability rather than as a draft |

## Layout and Content

**Header:** Screen title "Billing & Subscription" with a back arrow (returns to account settings, or closes the Support view for FEAT-19).

**Body, top section -- Status banner:** One consistent status treatment, per the Feature Breakdown Brief's Shared Context, showing exactly one of: "Active -- next billing date {date}"; "Payment failed -- update your payment method by {grace deadline} to keep your booking link live"; "Cancelled -- active through {period end date}, then your account pauses." **Display-only:** the banner itself is static text with no tap target of its own -- it reflects state driven by FEAT-18.SPEC-003/FEAT-18.SPEC-004 and carries no entry in the Interactions table below (the "Update payment method" and "Cancel subscription" controls it sits above are separate elements, listed in Interactions).

**Body, middle section -- Price and zero-fee statement block:** Identical to FEAT-18.SPEC-001's price card: the subscription's monthly price (platform parameter: `subscription-price`), "Billed monthly. Cancel anytime," and "Chairtime takes nothing from your deposits, balances, or tips -- 100% goes to you" (referencing FEAT-28.SPEC-004, XBR-07). **Display-only:** static text with no tap targets, inputs, or state changes of its own; no entry in the Interactions table below.

**Body, lower section -- Payment method:** A masked reference to the card on file (e.g., "Card ending in {last 4}," sourced from the payment-processing capability, never stored by the product) with an "Update payment method" action button below it. Not shown in Support's view. **Display-only (the masked reference text itself):** the "Card ending in {last 4}" text is static display, not tappable; only the "Update payment method" button beneath it is interactive and is listed separately in the Interactions table below.

**Footer:** A "Cancel subscription" text-style action, visually de-emphasized relative to "Update payment method." Not shown in Support's view.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width, sections stacked in the order above.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to account settings (FEAT-29) or close Support's view (FEAT-19) | Screen closes | Standard transition |
| "Update payment method" button | Tap | Opens the payment-processing capability's own card-entry surface (FEAT-18.SPEC-006) | Button shows loading state while the update is confirmed | Success: "Payment method updated" and the masked reference refreshes. Failure: clear failure reason with a retry action (inline, per Side-Effect Inventory) |
| "Cancel subscription" action | Tap | Opens a confirmation dialog describing the exact effect per FEAT-18.SPEC-005 (active through the paid period, then the account pauses) | Dialog appears | Dialog text states the exact period-end date |
| Cancel confirmation dialog -- "Confirm cancel" | Tap | Submits the cancellation through FEAT-18.SPEC-006 | Status banner updates to "Cancelled -- active through {date}" | Cancellation confirmation notification fires (FEAT-18.SPEC-007) |
| Cancel confirmation dialog -- "Keep subscription" | Tap | Dismisses the dialog, no change | Dialog closes | Status banner unchanged |

### Accessibility Notes

- **Focus order:** Back arrow -> Status banner (announced) -> Price/zero-fee block -> Payment method reference -> Update payment method -> Cancel subscription.
- **Status announcements:** The status banner's content is announced to assistive technology whenever it changes (e.g., after a successful renewal, a payment failure, or a cancellation).
- **Action feedback:** "Payment method updated" and cancellation confirmation are announced on success; failure messages move focus to the relevant control.
- **Keyboard alternatives:** Every action, including the cancel confirmation dialog, is reachable and dismissible by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Active | Status banner shows "Active -- next billing date {date}"; full actions available (Pro view) | Subscription status is Active | Renewal fails (FEAT-18.SPEC-003) or Pro cancels |
| Payment Failed (grace) | Status banner shows the grace deadline; "Update payment method" is visually emphasized | FEAT-18.SPEC-003 sets status to Payment Failed | Payment method is updated and confirmed (returns to Active) or grace period expires (FEAT-18.SPEC-004 pauses the account) |
| Cancelled (active to period end) | Status banner shows "Cancelled -- active through {date}, then your account pauses"; "Update payment method" remains available (a cancelled-but-still-active subscription can still need a valid card through period end); "Cancel subscription" is replaced with no further cancel action (already cancelled) | Pro confirms cancellation | Period ends and the account pauses (FEAT-18.SPEC-004), or -- per the dependency map's Contention note -- a reactivation attempt is refused (see Edge Cases; last-write-wins against reactivation is not offered as a Pro-facing control in v1) |
| Updating payment method | "Update payment method" shows loading state | Pro submits a new card | The processor confirms or rejects the update |
| Payment method update failed | Inline failure message below the payment method section with a retry action | The processor rejects the update | Pro retries or leaves the screen |
| Support view (read-only) | Status banner and price/zero-fee block only; no payment method reference, no action controls | Support opens the screen via FEAT-19 | Support closes the view |
| Offline/Degraded | Banner "You're offline -- showing your last known plan status" above the status banner; the last-loaded status remains viewable read-only; "Update payment method" and "Cancel subscription" are disabled with the message "A live connection is required to change your billing" | Connectivity lost while screen is open, or screen opened while offline | Connectivity restored -- actions re-enable and the screen refreshes to the current status |

## Validation Rules

Validation governed by FEAT-18.SPEC-005 (Subscription Billing Rules). See that spec for the single-tier price, cancellation-timing, and grace-period rules. No additional field-level validation exists on this screen -- payment method entry is handled entirely by the payment-processing capability's own surface (FEAT-18.SPEC-006).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap (Pro) | Account settings | FEAT-29 (Pro Sign-In & Account Lifecycle) |
| Back arrow tap (Support) | Closes the read-only view | FEAT-19 (Platform Support Read-Only Access) |

## Data Model

**Creates:** None.
**Reads:** Subscription -- status, billing_cycle / next_billing_date, payment_method_reference (masked), price. Displayed identically for the Pro's full view and Support's status-only view (Support never sees payment_method_reference).
**Updates:** Subscription -- payment_method_reference (via FEAT-18.SPEC-006, on successful payment method update); status (set to Cancelled, active to period end, via FEAT-18.SPEC-006 and FEAT-18.SPEC-005's timing rule, on cancel confirmation).
**Deletes:** None -- cancellation is a state transition, never a record deletion, per the Feature Breakdown Brief's Entity-Lifecycle Coverage Matrix.

## Business Rules

- The single price tier and zero-fee statement shown here must match FEAT-18.SPEC-001's presentation, per the Feature Breakdown Brief's Shared UI Patterns.
- Cancellation timing (active through the already-paid period, then pause) is governed by FEAT-18.SPEC-005 -- this screen never states or implies an immediate cutoff.
- Support's access is strictly view-only per SC-05 -- no action control is ever rendered in Support's view, regardless of subscription state.
- XBR-14: when the account is paused (from either subscription lapse or a Pro-chosen pause), this screen's own display and actions are unaffected -- the Pro can still view billing status and update the payment method to resolve a lapse.
- The processor's recorded outcome is authoritative for the Subscription's status, per the dependency map's Contention note -- if this screen's loaded status has gone stale relative to a processor-confirmed change, the next action is refused with refresh (see Edge Cases).

## Edge Cases

- **Payment method update fails at the processor** -- Inline failure message with the exact decline reason from FEAT-18.SPEC-006, and a retry action; the previous payment method reference remains on file and unaffected.
- **Pro opens this screen with no connectivity** -- The last known plan status is shown read-only with the offline banner; nothing is charged or changed while offline.
- **Pro attempts to pay or update the card with no connectivity** -- The action is blocked and the screen states plainly: "A live connection is required to change your billing."
- **Subscription status changed by the processor (e.g., a renewal completed or failed) while this screen is open** -- The screen is not live-updating by default; if the Pro attempts an action (update payment method, cancel) against a status that has since changed, the action is refused with a dialog: "Your billing status changed. Refreshing to show the latest." and the screen reloads the current status before the Pro retries. Resolution: reject-with-refresh, per the dependency map's Contention note for the Subscription entity (the processor's recorded outcome is authoritative).
- **Pro attempts to cancel a subscription that is already Cancelled (active to period end)** -- The cancel action is not offered a second time; the status banner already reflects the cancellation and its period-end date.
- **Pro wants to undo a cancellation before period end** -- Not offered as a control on this screen in v1: per the dependency map's Contention note, "cancellation is last-write-wins against reactivation before period end," meaning this screen does not expose a reactivation path; the Pro would need to subscribe again after the account pauses.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-006 (Subscription Billing Integration) | Triggers (outbound) | Update-payment-method and cancel actions submit through this integration |
| FEAT-18.SPEC-005 (Subscription Billing Rules) | References (inbound) | Cancellation timing, grace threshold, and single-tier price rules applied on this screen |
| FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | References (inbound) | Renewal outcomes keep the status banner current |
| FEAT-18.SPEC-004 (Subscription-Lapse Account Pause Trigger) | References (inbound) | Grace/pause state shown in the status banner; clears when billing is restored via this screen's payment-method update |
| FEAT-18.SPEC-007 (Subscription Billing Notifications) | Navigation (inbound) | Every billing notification's CTA deep-links here |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Navigation (inbound and outbound) | Entry from account settings; back arrow returns there |
| FEAT-19 (Platform Support Read-Only Access) | Navigation (inbound) | Support's entry point for billing status during a help request |
| FEAT-28.SPEC-004 (standing zero-fee rule) | References (inbound) | The zero-fee statement cites this feature's owning rule rather than redefining it |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| billing_screen_viewed | viewer role (pro / support) | The screen is opened | supports success-metrics.md: "Subscription Retention" |
| payment_method_updated | outcome (success / failure) | Pro submits a payment method update and the processor responds | supports success-metrics.md: "Subscription Retention" |
| subscription_cancel_confirmed | days remaining in current period at cancellation | Pro confirms cancellation | supports success-metrics.md: "Subscription Retention" |

## Acceptance Criteria

**FEAT-18.SPEC-002-AC-01:** Given Talia's subscription is Active, when she opens the Billing & Subscription Management Screen, then she sees "Active -- next billing date {date}" and the price/zero-fee statement block.

**FEAT-18.SPEC-002-AC-02:** Given Talia's subscription is in Payment Failed (grace), when she opens this screen, then she sees the grace deadline and an emphasized "Update payment method" action.

**FEAT-18.SPEC-002-AC-03:** Given Talia taps "Update payment method" and enters a valid new card, when the processor confirms it, then she sees "Payment method updated" and the masked card reference refreshes.

**FEAT-18.SPEC-002-AC-04:** Given Talia taps "Update payment method" and the processor rejects the new card, when the rejection is returned, then she sees the exact decline reason inline with a retry action and her previous payment method remains on file.

**FEAT-18.SPEC-002-AC-05:** Given Talia taps "Cancel subscription", when the confirmation dialog appears, then it states the exact date her subscription remains active through before the account pauses.

**FEAT-18.SPEC-002-AC-06:** Given Talia confirms cancellation, when the action completes, then the status banner updates to "Cancelled -- active through {date}, then your account pauses" and she receives the cancellation confirmation notification (FEAT-18.SPEC-007).

**FEAT-18.SPEC-002-AC-07:** Given a support operator opens a Pro's account via FEAT-19, when they view this screen, then they see only status, next billing date, and the price/zero-fee block, with no payment method reference and no action controls.

**FEAT-18.SPEC-002-AC-08:** Given Talia opens this screen with no connectivity, when the screen loads, then she sees her last known plan status read-only with the offline banner, and "Update payment method" and "Cancel subscription" are disabled.

**FEAT-18.SPEC-002-AC-09:** Given Talia attempts to update her payment method with no connectivity, when she taps the action, then it is blocked with the message "A live connection is required to change your billing."

**FEAT-18.SPEC-002-AC-10:** Given Talia's subscription status changed at the processor while this screen was open, when she attempts an action against the stale status, then the action is refused with "Your billing status changed. Refreshing to show the latest." and the screen reloads the current status.

**FEAT-18.SPEC-002-AC-11:** Given Talia's subscription is already Cancelled (active to period end), when she views this screen, then no cancel action is offered a second time.

**FEAT-18.SPEC-002-AC-12:** Given Talia's account is paused from a subscription lapse, when she opens this screen, then she can still view billing status and update her payment method to resolve the lapse, per XBR-14.

**FEAT-18.SPEC-002-AC-13:** Given Talia's session expires while viewing this screen, when she is prompted to sign in again, then no partial billing action is left in an ambiguous state -- any action she takes only completes atomically after re-authentication.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 7 (active, payment failed, cancelled, updating, update failed, support view, offline) | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
