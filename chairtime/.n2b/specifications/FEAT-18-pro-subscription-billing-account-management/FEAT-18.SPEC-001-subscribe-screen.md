---
document_type: spec
spec_type: screen
spec_id: FEAT-18.SPEC-001
spec_name: Subscribe Screen
spec_slug: subscribe-screen
parent_feature: FEAT-18
parent_feature_name: Pro Subscription Billing & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Screen Spec: Subscribe Screen

## Overview

**Name:** Subscribe Screen
**ID:** FEAT-18.SPEC-001
**Type:** Screen
**Purpose:** Talia enters her card details during onboarding, sees the single all-inclusive price and the zero-fee statement, and starts her Chairtime subscription.
**Parent Feature:** FEAT-18 -- Pro Subscription Billing & Account Management

## Scope and Non-Goals

**In Scope:**
- Displaying the single all-inclusive subscription price and the plain zero-fee statement
- Collecting card details and starting the first subscription charge through the payment-processing capability
- Showing the immediate outcome (success or decline) and, on success, reporting the go-live signal to onboarding

**Non-Goals:**
- Choosing among multiple price tiers or plans -- excluded per product-features.md FEAT-18 (Validation & Limits): "One price tier only in v1 -- no plan selection is offered."
- Ongoing plan management (viewing status, changing payment method, cancelling) -- owned by FEAT-18.SPEC-002 (Billing & Subscription Management Screen); this screen exists only for the first-time subscribe moment during onboarding
- Handling or storing the card data itself -- excluded per scope-boundaries.md SC-11: card data is never stored or handled by the product's own code; the payment-processing capability owns it entirely
- Deciding the onboarding step sequence around this screen -- owned by FEAT-15 (Pro Onboarding & Setup Wizard); this spec begins where FEAT-15 hands the Pro into the subscription step

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15 (Pro Onboarding & Setup Wizard, subscription step) | Pro completes the prior onboarding step and continues | The Pro's account reference; no pre-filled payment data |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Enter card details and submit the subscription charge | -- |
| The Client (Riley) | No | No | Clients never reach a Pro onboarding screen; there is no client-facing entry point into it |
| Platform Operator (Support) | No | No | Support's view-only access to billing status (per the Access Matrix, Subscription & Billing = View) is scoped to the ongoing Billing & Subscription Management Screen (FEAT-18.SPEC-002) once a subscription exists; there is nothing to view before the first subscription is created, so this onboarding screen is not part of Support's view |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); onboarding cannot resume mid-flow without a signed-in Pro |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- entered card details are never preserved (card data is never held by the product, per SC-11); the Pro re-enters payment details after signing back in |

## Layout and Content

**Header:** Screen title "Subscribe to Chairtime" with the onboarding step progress indicator (consistent with the rest of FEAT-15's setup steps) above it.

**Body, top section -- Price and zero-fee statement block:** A single price card showing the subscription's monthly price (platform parameter: `subscription-price`) and the line "Billed monthly. Cancel anytime." Directly below it, in plain language: "Chairtime takes nothing from your deposits, balances, or tips -- 100% goes to you," referencing the standing zero-fee rule (FEAT-28.SPEC-004, XBR-07). This block is described identically to its sibling on FEAT-18.SPEC-002 per the Feature Breakdown Brief's Shared Context. **Display-only:** the entire block (price card and zero-fee statement) is static text with no tap targets, inputs, or state changes of its own; it carries no entry in the Interactions table below.

**Body, middle section -- Payment form:** A single-column form with:
- Card details entry (card number, expiry, security code, postal code) -- handled entirely by the payment-processing capability's own input surface (FEAT-18.SPEC-006); the product never stores or reads the raw values
- Name on card (text input, required)

**Footer:** A single "Start subscription" action button, full width, disabled until the card fields are complete.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width; the price/zero-fee block stacks above the form; "Start subscription" remains full width at the bottom.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width (the design layer's decision) and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Card details entry | Type | Captures card input via the payment-processing capability's own surface | Field shows entered values (masked per card-field convention) | Standard input focus state |
| Name on card input | Type | Captures text input | Field shows entered text | Standard input focus state |
| Name on card input | Blur (empty) | Triggers field validation via FEAT-18.SPEC-005 | Error state on field | "Name on card is required" below field |
| "Start subscription" button | Tap | 1. Validate name-on-card field. 2. If valid, submit the charge through FEAT-18.SPEC-006 (Subscription Billing Integration). | Button shows a "Processing payment, do not close this page" loading state | Success: confirmation message and automatic continuation into onboarding. Failure: inline decline message with retry. |
| "Start subscription" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Card details entry (as ordered by the payment-processing capability's own surface) -> Name on card -> Start subscription.
- **Validation announcements:** When the name-on-card field enters an error state, its error message is announced to assistive technology and programmatically associated with the field.
- **Processing and outcome feedback:** The "Processing payment" state is announced when it begins; the success confirmation or decline message is announced immediately when the outcome arrives.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Filling | Form empty or partially filled, "Start subscription" disabled until required fields are complete | Screen first opens | All required fields complete |
| Ready | "Start subscription" enabled | Card fields and name on card are complete | Pro taps "Start subscription" |
| Processing | Button shows "Processing payment, do not close this page"; form fields disabled | Pro taps "Start subscription" | The payment-processing capability confirms success or reports a decline |
| Success | Confirmation shown ("You're subscribed") and the screen automatically continues into the next onboarding step | The processor confirms the first charge succeeded | Pro is navigated onward by FEAT-15 |
| Declined | Inline decline message from FEAT-18.SPEC-006 with a retry action; form fields (except card details, which must be re-entered) remain as entered | The processor reports a decline | Pro edits card details and retries |
| Offline/Degraded | N/A -- starting a subscription requires a live connection to the payment-processing capability by nature; a connection drop mid-submission is treated as the Declined/failure path (FEAT-18.SPEC-006), never an ambiguous charge | -- | -- |

## Validation Rules

Validation governed by FEAT-18.SPEC-005 (Subscription Billing Rules). See that spec for the single-tier price rule. This screen additionally applies simple inline validation on the name-on-card field on blur and on submit.

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Name on card | Required, non-empty | On blur, on submit | "Name on card is required" |

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Successful subscription start | Next onboarding step | FEAT-15 (Pro Onboarding & Setup Wizard) |
| Declined payment, Pro abandons retry | Onboarding step remains on this screen | -- |

## Data Model

**Creates:** Subscription record -- created the moment the payment-processing capability confirms the first successful charge (fields: status set to Active, billing_cycle / next_billing_date set to one billing period from today, payment_method_reference set from the processor's confirmation, price set to platform parameter: `subscription-price`). The screen itself never holds card data -- only the outcome and the resulting reference (FEAT-18.SPEC-006).
**Reads:** None -- this is a first-time creation screen; no existing Subscription exists yet.
**Updates:** None.
**Deletes:** None.

## Business Rules

- Single price tier only -- no plan selection is offered on this screen, per FEAT-18.SPEC-005.
- The charge is submitted through FEAT-18.SPEC-006 (Subscription Billing Integration); this screen never handles card data directly (SC-11).
- On success, this screen reports an active subscription to FEAT-15's go-live check (XBR-26) -- the booking link cannot go live without it.
- The price and zero-fee statement shown here must match FEAT-18.SPEC-002's presentation of the same information, per the Feature Breakdown Brief's Shared UI Patterns.

## Edge Cases

- **Pro navigates away mid-entry without submitting** -- Onboarding preserves the Pro's place at this step; no partial card data is retained anywhere (card data is never held by the product), so the Pro re-enters card details on return.
- **Pro taps "Start subscription" twice rapidly** -- Second tap is ignored while the first submission is in progress (button in loading/Processing state).
- **Connection drops mid-submission** -- Treated as a failure by FEAT-18.SPEC-006: the Pro sees the same decline/failure message with retry, and no ambiguous charge state is left ("processing payment, do not close this page" resolves to a definite outcome, never a stuck state).
- **Payment succeeds but the confirmation fails to render** -- The Subscription record is still correctly created by the payment-processing capability's confirmed outcome (FEAT-18.SPEC-006); on next load of this step or the onboarding flow, the Pro sees the success state and onboarding continues -- never double-charged and never left unsure whether the subscription started.
- **No concurrent-edit conflict applies** -- This screen creates the Subscription for the first time; no prior version can exist to conflict with, so the dependency map's Subscription Contention note (processor-authoritative outcome, reject-with-refresh) does not apply until FEAT-18.SPEC-002 exists for this Pro.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-006 (Subscription Billing Integration) | Triggers (outbound) | "Start subscription" submits the first charge through this integration; its confirmed outcome creates the Subscription record |
| FEAT-18.SPEC-005 (Subscription Billing Rules) | References (inbound) | Single-tier price and validation rules applied on this screen |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | References (inbound) | Shares the identical price/zero-fee statement block per the Brief's Shared UI Patterns |
| FEAT-28.SPEC-004 (standing zero-fee rule) | References (inbound) | The zero-fee statement cites this feature's owning rule rather than redefining it |
| FEAT-15 (Pro Onboarding & Setup Wizard) | Navigation (inbound and outbound) | Entry from the prior onboarding step; success continues to the next step and reports the go-live signal (XBR-26) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| subscription_started | onboarding entry point | The payment-processing capability confirms the first successful charge | supports success-metrics.md: "Subscription Retention" |
| subscription_start_declined | decline reason category | The payment-processing capability reports a decline | N/A -- no Stage 2 metric measures decline frequency directly; retained so onboarding friction at this step is observable |

## Acceptance Criteria

**FEAT-18.SPEC-001-AC-01:** Given Talia is on the Subscribe Screen during onboarding, when she enters valid card details and her name on card and taps "Start subscription", then the charge is submitted through FEAT-18.SPEC-006, and on success she sees a confirmation and onboarding continues to the next step.

**FEAT-18.SPEC-001-AC-02:** Given Talia is on the Subscribe Screen, when she leaves the name-on-card field empty and taps "Start subscription", then the field shows the error "Name on card is required" and no charge is submitted.

**FEAT-18.SPEC-001-AC-03:** Given Talia submits valid card details, when the payment-processing capability declines the charge, then she sees the decline reason inline from FEAT-18.SPEC-006 and can retry with the same or a different card.

**FEAT-18.SPEC-001-AC-04:** Given Talia's subscription charge succeeds, when the Subscription record is created, then FEAT-18.SPEC-001 reports an active subscription to FEAT-15's go-live check (XBR-26).

**FEAT-18.SPEC-001-AC-05:** Given Talia taps "Start subscription" while a submission is already in progress, when she taps it again, then the second tap is ignored and the button remains in its Processing state.

**FEAT-18.SPEC-001-AC-06:** Given Talia's connection drops while her payment is processing, when the drop is detected, then she sees the same failure/decline message with a retry action, and no ambiguous or double charge occurs.

**FEAT-18.SPEC-001-AC-07:** Given Talia's payment succeeds but the confirmation fails to render on screen, when she reloads or returns to this step, then she sees the success state and onboarding continues, with no duplicate charge.

**FEAT-18.SPEC-001-AC-08:** Given Talia is on the Subscribe Screen, when she views the price block, then she sees the single all-inclusive monthly price and the statement "Chairtime takes nothing from your deposits, balances, or tips -- 100% goes to you."

**FEAT-18.SPEC-001-AC-09:** Given Talia navigates away from this step without submitting, when she returns to it later in onboarding, then no card data was retained and she must re-enter it.

**FEAT-18.SPEC-001-AC-10:** Given a support operator opens a Pro's account before that Pro has a Subscription, when they look for this screen, then it is not part of their view -- Support's billing visibility begins at FEAT-18.SPEC-002 once a subscription exists.

**FEAT-18.SPEC-001-AC-11:** Given Talia's session expires while she is filling in the Subscribe Screen, when she is prompted to sign in again, then no card data is preserved and she must re-enter her payment details from a fresh state.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 (filling, ready, processing, success, declined; offline N/A) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
