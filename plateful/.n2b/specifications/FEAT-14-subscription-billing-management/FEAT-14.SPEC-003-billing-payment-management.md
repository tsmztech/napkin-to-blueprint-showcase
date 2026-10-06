---
document_type: spec
spec_type: screen
spec_id: FEAT-14.SPEC-003
spec_name: Billing & Payment Management
spec_slug: billing-payment-management
parent_feature: FEAT-14
parent_feature_name: Subscription & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: Billing & Payment Management

## Overview

**Name:** Billing & Payment Management
**ID:** FEAT-14.SPEC-003
**Type:** Screen
**Purpose:** Maya updates payment details, views billing history, and switches billing period for the household's paid subscription.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- Updating payment details on the household's paid subscription
- Viewing billing history
- Switching billing period (monthly to yearly or yearly to monthly)
- Entry point into downgrading or cancelling
- Entry point into requesting a data export

**Non-Goals:**
- Upgrading from free to paid -- owned by FEAT-14.SPEC-002 (Upgrade to Paid); this screen is reachable only once the household is already paid
- The downgrade/cancel confirmation flow itself -- owned by FEAT-14.SPEC-004 (Downgrade / Cancel); this screen only offers the entry point
- Data export mechanics -- owned by FEAT-18 (Account & Data Management); this screen only offers the navigation entry point, per the Feature Breakdown Brief's Cross-Feature Touchpoints
- Determining timing for a period switch (next renewal, never mid-period) -- governed by FEAT-14.SPEC-005 (Billing State & Refund Rules); this screen collects the choice and defers to that spec's timing rule

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-14.SPEC-001 (Plan Tier Overview) | Maya taps "Manage Billing" (shown only when tier is paid) or the billing-state banner | None -- loads the household's current Subscription and billing_history |
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | Maya taps "View billing" | None -- loads the household's current Subscription and billing_history |
| FEAT-14.SPEC-011 (Payment Failure Grace-Period Notice) | Maya taps "Update payment details" | None -- loads the Subscription in its Payment failed billing_state |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen, including payment details, billing history, and billing-state banner | All actions: update payment details, switch period, navigate to downgrade/cancel or export | -- |
| Sam (Other Adult Member) | No | No | No entry point to this screen exists on FEAT-14.SPEC-001; a direct link resolves to "You don't have access to billing for this household." |
| Jordan (young kid profile, no login -- MVP) | No | No | Screen is unreachable -- a no-login profile has no sign-in path to any screen |
| Jordan (older kid, limited login -- Later) | No | No | A direct link resolves to "You don't have access to billing for this household." |
| Riley (Operator, support -- from v1) | No | No | Riley's Billing access is View of plan tier only on FEAT-14.SPEC-001; payment details and billing history on this screen are never shown to Riley under any support scenario, per user-persona.md's Access Matrix notes |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on FEAT-14.SPEC-001, not this screen |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; any in-progress payment-detail edit is preserved and restored after re-authentication (payment field values themselves follow FEAT-14.SPEC-009's retention rules) |

## Layout and Content

**Header:** Screen title "Manage Billing" with a back arrow (returns to FEAT-14.SPEC-001).

**Body:**
- **Billing-state banner**, at the top, shown when billing_state is not Active, or when pending_change is not none even while billing_state is Active (a pending downgrade); identical wording source to FEAT-14.SPEC-001 per the Brief's Shared UI Patterns.
- **Current plan summary**: billing_period ("Monthly" or "Yearly") with a "Switch to {other period}" action -- replaced, when pending_change is period_switch, by "Switching to {pending_change_new_period} on {current_period_end_date}" with no further switch action available; replaced, when pending_change is downgrade or billing_state is Cancelled, by "Your household moves to the free tier on {current_period_end_date}." with no switch action shown (there is no paid period left for a switch to apply to).
- **Payment details section**: the household's stored payment method summary (masked, per FEAT-14.SPEC-009's data-minimization contract) with an "Update payment details" action.
- **Billing history list**: chronological entries from billing_history, each showing date, amount, and outcome (charged, failed, refunded-status where applicable per FEAT-14.SPEC-005's no-partial-refund rule), sourced from FEAT-14.SPEC-009 (Payment Processing Integration).
- **Downgrade/Cancel entry**: a "Downgrade to Free" and a "Cancel Subscription" link, grouped together, visually separated from the sections above -- replaced, when pending_change is downgrade, by "Downgrade scheduled for {current_period_end_date}"; replaced, when billing_state is Cancelled, by "Cancellation scheduled for {current_period_end_date}"; in either case the remaining link (Cancel or Downgrade, respectively) is also hidden, since only one reversion to free can be pending at a time.
- **Data export entry**: a "Request a data export" link, per the Feature Breakdown Brief's Cross-Feature Touchpoints, navigating to FEAT-18.

**Footer:** None -- all actions are inline within their sections.

### Responsive Behavior

- **Compact breakpoint:** All sections stack vertically, full width; the billing history list scrolls independently within its section.
- **Medium size class and above:** Current plan summary and payment details sections may render side by side; the billing history list remains full width below them.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-14.SPEC-001 | Screen closes | Standard transition |
| Billing-state banner | Tap | No further navigation -- already on the billing screen it would otherwise lead to | None | -- |
| "Switch to {other period}" (shown only while pending_change is none) | Tap | 1. Confirm the switch via a dialog stating it takes effect at the next renewal, per FEAT-14.SPEC-005. 2. On confirm, trigger FEAT-14.SPEC-008 (Apply Subscription Change) to record the requested period (pending_change = period_switch). | Button shows a brief loading state | Success: toast "You'll switch to {period} billing at your next renewal." and the current plan summary updates to show the pending change. Failure: inline error with retry |
| "Update payment details" | Tap | Opens the payment-details form (fields per FEAT-14.SPEC-009's contract) | Form replaces the payment summary inline | Standard input focus state |
| Payment-details form Save | Tap | Submits updated payment details through FEAT-14.SPEC-009; while billing_state is Payment failed, the submission also triggers FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) to retry the charge | Button shows loading state | Success: toast "Payment details updated." and the masked summary reflects the new method. Failure: inline error, prior payment method remains in effect |
| Billing history list item | Tap | Display-only, non-interactive (no drill-down beyond the list row's own detail) | None | -- |
| "Downgrade to Free" | Tap | Navigate to FEAT-14.SPEC-004 (Downgrade / Cancel), downgrade path | Screen changes | Standard transition |
| "Cancel Subscription" | Tap | Navigate to FEAT-14.SPEC-004 (Downgrade / Cancel), cancellation path | Screen changes | Standard transition |
| "Request a data export" | Tap | Navigate to FEAT-18 (Account & Data Management) | Screen changes | Standard transition |

### Accessibility Notes

- **Focus order:** Back arrow -> billing-state banner (when shown) -> current plan summary and its switch action -> payment details section and its update action -> billing history list -> Downgrade/Cancel links -> data export link.
- **Dynamic announcements:** Toasts for period switch and payment-detail updates are announced as live-region changes; a validation error on the payment form is announced and associated with its field.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (default) | All sections render with current data | Screen opens and Subscription and billing_history load successfully | User navigates away |
| Loading | Sections show loading placeholders | Screen first opens, before data has loaded | Load completes (success or error) |
| Error | Banner: "Couldn't load your billing details. Check your connection and try again." with Retry; no sections shown | Loading Subscription or billing_history fails | Retry succeeds |
| Empty (billing history) | Billing history section shows "No billing history yet" | The household has just upgraded and no charge has posted yet | The first charge posts and appears in the list |
| Offline/Degraded | Last-loaded plan summary, payment summary, and billing history remain visible with a banner: "You're offline -- showing your billing details as of your last visit." Switch period, update payment details, and downgrade/cancel actions are disabled with "Requires a connection." | Connectivity is lost while viewing, or the screen opens with cached data and no connectivity | Connectivity restored -- actions re-enable and data refreshes silently |

## Validation Rules

Validation governed by FEAT-14.SPEC-005 (Billing State & Refund Rules) for period-switch timing, and by FEAT-14.SPEC-009 (Payment Processing Integration) for payment field format rules. This screen applies validation on form submit for payment-detail updates and on confirmation for a period switch.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-14.SPEC-001 (Plan Tier Overview) | -- |
| "Downgrade to Free" tap | FEAT-14.SPEC-004 (Downgrade / Cancel) | -- |
| "Cancel Subscription" tap | FEAT-14.SPEC-004 (Downgrade / Cancel) | -- |
| "Request a data export" tap | FEAT-18 (Account & Data Management) | FEAT-18 |

## Data Model

**Creates:** None.
**Reads:** Subscription -- tier, billing_period, billing_state, billing_history, pending_change, pending_change_new_period, current_period_end_date (all displayed, including the scheduled-change and Cancelled-pending display). Household -- currency (billing history amounts displayed in the household's configured currency, per ASMP-28).
**Updates:** None directly -- period-switch and payment-detail changes are written by FEAT-14.SPEC-008 and FEAT-14.SPEC-009 respectively, triggered from this screen.
**Deletes:** None.

## Business Rules

- A period switch always takes effect at the next renewal, never immediately or mid-period, per FEAT-14.SPEC-005.
- Payment-detail updates take effect immediately for the next charge attempt; they do not themselves change tier, billing_period, or billing_state.
- Billing history is retained for the life of the household account with no purge, per scope-boundaries.md SC-18 -- this screen never offers a delete or clear action on history entries.
- Only Maya reaches this screen or any of its actions, per FEAT-14.SPEC-006 (Tier & Billing Access Authorization).
- The pending-change display (scheduled switch, scheduled downgrade, or scheduled cancellation) and the billing-state banner are both sourced from FEAT-14.SPEC-005's Governed Entity fields (pending_change, pending_change_new_period, current_period_end_date, billing_state) -- this screen never derives its own scheduled-change wording.

## Edge Cases

- **Subscription changed by a concurrent action (e.g., the grace period expires and the household reverts to free while this screen is open)** -- Any pending period-switch or payment-detail action is rejected with refresh: "Your billing state has changed. Reloading your billing details." and the screen reloads to reflect the current billing_state, consistent with the dependency map's Contention note for Subscription (reject-with-refresh against a stale billing state).
- **Maya taps "Switch to Yearly" twice in quick succession** -- The second tap is ignored while the first request is in progress; only one pending period-switch request exists at a time.
- **Payment-detail update fails validation** -- The prior payment method remains in effect and in use for the next charge attempt; no partial or invalid payment method is ever stored.
- **Maya requests a period switch, then downgrades before the next renewal** -- The downgrade (FEAT-14.SPEC-004, via FEAT-14.SPEC-008) supersedes the pending period switch; the household reverts to free at period end and the period-switch request never takes effect, since there is no paid period left for it to apply to.
- **Billing history contains an entry for a charge still processing** -- The entry shows a "Processing" outcome rather than a final charged/failed state until the outcome is known.
- **Maya opens this screen while a cancellation is already pending (billing_state Cancelled)** -- The Downgrade/Cancel section shows "Cancellation scheduled for {current_period_end_date}" in place of both links, and no "Switch to {other period}" action is shown, since the household is already resolving to free.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-001 (Plan Tier Overview) | Navigation (inbound) | "Manage Billing" and the billing-state banner hand off to this screen |
| FEAT-14.SPEC-005 (Billing State & Refund Rules) | References (inbound) | Period-switch timing rule enforced here |
| FEAT-14.SPEC-009 (Payment Processing Integration) | Triggers (outbound) | Payment-detail updates and billing history are sourced from here |
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggers (outbound) | Period-switch requests are written here |
| FEAT-14.SPEC-004 (Downgrade / Cancel) | Navigation (outbound) | "Downgrade to Free" and "Cancel Subscription" lead here |
| FEAT-18 (Account & Data Management) | Navigation (outbound) | "Request a data export" hands off here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| billing_history_viewed | entry count | Screen finishes loading with billing_history populated | supports success-metrics.md: "Paying Household Retention" |
| period_switch_requested | from_period, to_period | Maya confirms a period switch | supports success-metrics.md: "Paying Household Retention" |
| payment_details_updated | outcome (success / failure) | Maya submits an updated payment method | supports success-metrics.md: "Paying Household Retention" |
| downgrade_or_cancel_entry_tapped | action (downgrade / cancel) | Maya taps "Downgrade to Free" or "Cancel Subscription" | N/A -- no Stage 2 metric tracks downgrade-intent taps directly; retained as a leading indicator ahead of "Paying Household Retention" |

## Acceptance Criteria

**FEAT-14.SPEC-003-AC-01:** Given Maya's household is on the paid monthly plan, when she opens this screen, then she sees "Monthly" as the current period with a "Switch to Yearly" action, her masked payment method, and her billing history.

**FEAT-14.SPEC-003-AC-02:** Given Maya taps "Switch to Yearly" and confirms, then a toast reads "You'll switch to Yearly billing at your next renewal." and the plan summary shows the pending change.

**FEAT-14.SPEC-003-AC-03:** Given Maya taps "Update payment details" and submits a new valid payment method, then a toast reads "Payment details updated." and the masked summary reflects it.

**FEAT-14.SPEC-003-AC-04:** Given Maya submits an invalid payment method, when validation fails, then the prior payment method remains in effect and an inline error is shown.

**FEAT-14.SPEC-003-AC-05:** Given Maya's household has billing_state Payment failed, when she opens this screen, then the billing-state banner appears at the top with the grace-period wording shared with FEAT-14.SPEC-001.

**FEAT-14.SPEC-003-AC-06:** Given Maya taps "Downgrade to Free", then she is navigated to FEAT-14.SPEC-004 on the downgrade path.

**FEAT-14.SPEC-003-AC-07:** Given Maya taps "Cancel Subscription", then she is navigated to FEAT-14.SPEC-004 on the cancellation path.

**FEAT-14.SPEC-003-AC-08:** Given Maya taps "Request a data export", then she is navigated to FEAT-18 (Account & Data Management).

**FEAT-14.SPEC-003-AC-09:** Given a household's grace period expires and it reverts to free while Maya has this screen open with a pending period-switch request, when the reversion completes, then her request is rejected with "Your billing state has changed. Reloading your billing details." and the screen reloads showing Free tier.

**FEAT-14.SPEC-003-AC-10:** Given Maya's household has just upgraded with no charges posted yet, when she opens this screen, then the billing history section shows "No billing history yet."

**FEAT-14.SPEC-003-AC-11:** Given Maya loses connectivity while viewing this screen, when she taps "Switch to Yearly", then the action is disabled with "Requires a connection." and her last-loaded billing details remain visible.

**FEAT-14.SPEC-003-AC-12:** Given Sam attempts to reach this screen directly, when the attempt is made, then he sees "You don't have access to billing for this household." and no billing data is shown.

**FEAT-14.SPEC-003-AC-13:** Given Maya's household has billing_state Cancelled with current_period_end_date March 14, when she opens this screen, then the Downgrade/Cancel section shows "Cancellation scheduled for March 14" instead of the Downgrade/Cancel links, and no "Switch to {other period}" action is shown.

**FEAT-14.SPEC-003-AC-14:** Given Maya's household has billing_state Active with pending_change downgrade and current_period_end_date March 14, when she opens this screen, then the current plan summary shows "Your household moves to the free tier on March 14." instead of a switch action.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 5 (loaded, loading, error, empty history, offline) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
