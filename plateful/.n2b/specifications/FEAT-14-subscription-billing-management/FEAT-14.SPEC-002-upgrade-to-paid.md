---
document_type: spec
spec_type: screen
spec_id: FEAT-14.SPEC-002
spec_name: Upgrade to Paid
spec_slug: upgrade-to-paid
parent_feature: FEAT-14
parent_feature_name: Subscription & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: Upgrade to Paid

## Overview

**Name:** Upgrade to Paid
**ID:** FEAT-14.SPEC-002
**Type:** Screen
**Purpose:** Maya reviews what upgrading unlocks, chooses monthly or yearly billing, and enters payment details to subscribe.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management

## Scope and Non-Goals

**In Scope:**
- Reviewing the tier-inclusion comparison in the context of upgrading
- Choosing monthly or yearly billing period
- Entering payment details and submitting the upgrade
- Preserving the chosen plan option and offering a retry when the upgrade attempt fails

**Non-Goals:**
- Viewing the current tier outside an upgrade attempt -- owned by FEAT-14.SPEC-001 (Plan Tier Overview), which is where this screen is entered from
- Updating payment details or viewing billing history after subscribing -- owned by FEAT-14.SPEC-003 (Billing & Payment Management)
- Submitting or retrying the actual charge -- owned by FEAT-14.SPEC-009 (Payment Processing Integration); this screen collects the input and displays the outcome
- Downgrading or cancelling -- excluded per this feature's own scope split: FEAT-14.SPEC-004 (Downgrade / Cancel) owns the opposite direction of this transition

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-14.SPEC-001 (Plan Tier Overview) | Maya taps "Upgrade" (shown only when tier is free) | None -- the form starts with no plan period preselected |
| FEAT-03.SPEC-002 (Free-Tier Plan Placeholder & Upgrade Prompt) | Maya taps "Upgrade" on the free-tier plan placeholder | None -- the form starts with no plan period preselected |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Maya (Organiser) | Full screen | All actions: choose period, enter payment, submit | -- |
| Sam (Other Adult Member) | No | No | No entry point to this screen exists on FEAT-14.SPEC-001; a direct link resolves to "You don't have access to billing for this household." |
| Jordan (young kid profile, no login -- MVP) | No | No | Screen is unreachable -- a no-login profile has no sign-in path to any screen |
| Jordan (older kid, limited login -- Later) | No | No | A direct link resolves to "You don't have access to billing for this household." |
| Riley (Operator, support -- from v1) | No | No | Riley's Billing access is View of plan tier only, never payment collection; a direct link resolves to "You don't have access to billing for this household." |
| Unauthenticated | No | No | Redirected to the sign-in screen; after signing in, the user lands on FEAT-14.SPEC-001, not this form |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; the chosen plan period and any entered payment fields are preserved and restored after re-authentication (payment field values themselves are never persisted beyond the session, per FEAT-14.SPEC-009's Data Exchanged section -- only the period choice and non-sensitive form state survive) |

## Layout and Content

**Header:** Screen title "Upgrade to Paid" with a back arrow (returns to FEAT-14.SPEC-001) and no header action (Subscribe is in the footer).

**Body:**
- **Tier-inclusion summary block**, identical content to FEAT-14.SPEC-001 and FEAT-14.SPEC-004 per the Brief's Shared UI Patterns, framed here as "What you'll unlock."
- **Billing period selector**: two options, "Monthly" (platform parameter: `subscription-price-monthly` shown per the household's currency, per ASMP-28) and "Yearly" (platform parameter: `subscription-price-yearly`), presented as a toggle or radio pair; neither is preselected.
- **Payment details form**: standard payment-method fields collected on behalf of, and submitted through, FEAT-14.SPEC-009 (Payment Processing Integration) -- exact field set is that integration's contract, not redefined here.
- **Money-flow note**: a plain-language line stating that payment is between the organiser and the product; no money passes between households, per product-features.md's Validation & Limits.

**Footer:** "Subscribe" action button, full width, disabled until a billing period is chosen and the payment form is complete.

### Responsive Behavior

- **Compact breakpoint:** Tier-inclusion block, period selector, and payment form stack vertically, full width; "Subscribe" remains in the footer, one-thumb reachable per ASMP-29.
- **Medium size class and above:** Tier-inclusion block and period selector may render side by side above the payment form; no structural change to the form itself.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-14.SPEC-001 (Plan Tier Overview) | Screen closes | Standard transition; any entered payment data is discarded |
| "Monthly" option | Tap | Selects monthly billing period | Option shows selected state; price updates to the monthly amount | Selected-state styling |
| "Yearly" option | Tap | Selects yearly billing period | Option shows selected state; price updates to the yearly amount | Selected-state styling |
| Payment details fields | Type | Captures payment input per FEAT-14.SPEC-009's field contract | Field shows entered value | Standard input focus state |
| "Subscribe" button | Tap | 1. Validate a period is chosen and payment fields are complete, per FEAT-14.SPEC-005 (Billing State & Refund Rules) and FEAT-14.SPEC-009. 2. Submit the charge through FEAT-14.SPEC-009. 3. On success, trigger FEAT-14.SPEC-008 (Apply Subscription Change). | Button shows loading state during submission | Success: navigates to FEAT-14.SPEC-001 showing the new Paid tier, and FEAT-14.SPEC-010 (Billing Confirmation Notification) is sent. Failure: inline error per the Error state below; chosen period and non-sensitive form state are preserved |
| "Subscribe" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> tier-inclusion block -> "Monthly" option -> "Yearly" option -> payment details fields in order -> Subscribe.
- **Dynamic announcements:** A validation error on the payment form is announced to assistive technology and programmatically associated with the field; the submission-failure banner is announced as a live-region change.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (default) | No period selected, payment fields empty, "Subscribe" disabled | Screen first opens | Maya selects a period or begins entering payment details |
| Filling | Period selected and/or payment fields contain input; "Subscribe" enabled once both are complete | Maya selects a period or types in a payment field | Maya taps Subscribe or navigates away |
| Submitting | "Subscribe" shows a loading spinner; a note reads "Confirming within a few seconds" per product-features.md's Loading state | Maya taps Subscribe with valid input | Submission completes (success or failure) |
| Error | Banner with the payment-processing capability's returned reason (per FEAT-14.SPEC-009's Degradation Behavior) and a Retry button; chosen period and non-sensitive form state remain | Submission fails (payment declined or processing error) | Maya taps Retry, or corrects input and resubmits |
| Offline/Degraded | Banner: "Upgrading requires a connection. Your choice is saved -- reconnect to finish." Period selection remains editable; payment fields remain visible but Subscribe is disabled | Connectivity is lost while this screen is open | Connectivity restored -- Subscribe re-enables; nothing is submitted automatically, since payment requires an explicit Subscribe tap even after reconnecting |

## Validation Rules

Validation governed by FEAT-14.SPEC-005 (Billing State & Refund Rules) for payment-validity requirements, and by FEAT-14.SPEC-009 (Payment Processing Integration) for payment field format rules. This screen applies validation on form submit.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-14.SPEC-001 (Plan Tier Overview) | -- |
| Successful subscribe | FEAT-14.SPEC-001 (Plan Tier Overview), showing the new Paid tier, with the first AI-generated plan on its way | FEAT-03.SPEC-004 (First Plan Generation on Upgrade) -- FEAT-03 |

## Data Model

**Creates:** None directly -- a successful submission triggers FEAT-14.SPEC-008 to write the Subscription update.
**Reads:** Subscription -- tier (to confirm the household is still free-tier on load). Household -- currency (to display prices in the household's configured currency).
**Updates:** None directly on this screen -- see FEAT-14.SPEC-008.
**Deletes:** None.

## Business Rules

- Valid payment details are required to upgrade, per FEAT-14.SPEC-005 -- Subscribe cannot succeed without them.
- A failed upgrade attempt never loses the chosen plan option: it stays selected and payment fields (except the sensitive payment values themselves, which follow FEAT-14.SPEC-009's own retention rules) are preserved for retry, per product-features.md's Error state.
- Successful submission triggers FEAT-14.SPEC-008 (Apply Subscription Change), which is the sole writer of the Subscription record -- this screen never writes tier or billing_period directly.
- XBR-05: on success, AI plan generation, pantry-aware plan weighting, and rating-based learning unlock immediately -- this screen's confirmation reflects that immediacy.

## Edge Cases

- **Payment declined or processing error** -- Inline in this screen per the Feature Breakdown Brief's Side-Effect Inventory: the chosen plan option is preserved, the payment form remains editable, and a Retry option is offered; no Subscription change occurs.
- **Maya double-taps Subscribe** -- The second tap is ignored while the first submission is in progress (button in loading state); at most one charge attempt is sent to FEAT-14.SPEC-009 per tap sequence.
- **Maya navigates away mid-submission** -- The submission continues; if it completes after she has left, the outcome is reflected the next time she opens FEAT-14.SPEC-001, and FEAT-14.SPEC-010 still sends its confirmation.
- **Household's tier changes to paid from another device while this screen is open (e.g., Maya subscribes on her phone while this screen is open on a laptop from an earlier session)** -- Tapping Subscribe here is rejected with "Your household is already on the paid tier." and the screen redirects to FEAT-14.SPEC-001, consistent with the dependency map's Contention note for Subscription (reject-with-refresh against a stale billing state).
- **Maya loses connectivity mid-entry, before tapping Subscribe** -- No data is lost; the Offline/Degraded state applies and Subscribe remains disabled until reconnection.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-001 (Plan Tier Overview) | Navigation (inbound) | "Upgrade" hands off to this screen |
| FEAT-14.SPEC-005 (Billing State & Refund Rules) | References (inbound) | Payment-validity requirement enforced on submit |
| FEAT-14.SPEC-009 (Payment Processing Integration) | Triggers (outbound) | Submit action initiates the charge |
| FEAT-14.SPEC-008 (Apply Subscription Change) | Triggers (outbound) | Successful charge triggers the Subscription write |
| FEAT-14.SPEC-010 (Billing Confirmation Notification) | Triggers (outbound) | Successful upgrade sends the confirmation |
| FEAT-03.SPEC-004 (First Plan Generation on Upgrade) | Navigation (outbound) | Successful upgrade routes toward the first AI-generated plan |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| upgrade_period_selected | period (monthly / yearly) | Maya selects a billing period | supports success-metrics.md: "Paid Conversion Rate" |
| upgrade_submitted | period, entry source | Maya taps Subscribe with valid input | supports success-metrics.md: "Paid Conversion Rate" |
| upgrade_completed | period | Payment succeeds and FEAT-14.SPEC-008 confirms tier=paid | supports success-metrics.md: "Paid Conversion Rate" |
| upgrade_failed | period, failure reason category | Payment is declined or a processing error occurs | supports success-metrics.md: "Paid Conversion Rate" (a failed attempt is a lost conversion this metric must reflect) |

## Acceptance Criteria

**FEAT-14.SPEC-002-AC-01:** Given Maya is on this screen, when she selects "Monthly", then the monthly price for her household's currency is shown and "Yearly" is deselected.

**FEAT-14.SPEC-002-AC-02:** Given Maya has selected "Yearly" and entered complete payment details, when she taps Subscribe, then the button shows a loading state and a note that confirmation takes a few seconds.

**FEAT-14.SPEC-002-AC-03:** Given Maya's payment succeeds, when the submission completes, then she is returned to FEAT-14.SPEC-001 showing "Paid -- Yearly", the confirmation notification (FEAT-14.SPEC-010) is sent, and she is offered the path toward her first AI-generated plan (FEAT-03.SPEC-004).

**FEAT-14.SPEC-002-AC-04:** Given Maya's payment is declined, when the submission fails, then her chosen period remains selected, the payment form remains editable, and a Retry option is shown -- no Subscription change occurs.

**FEAT-14.SPEC-002-AC-05:** Given Maya taps Subscribe with no billing period chosen, then the button remains disabled and no submission is attempted.

**FEAT-14.SPEC-002-AC-06:** Given Maya taps Subscribe twice in quick succession, when the first submission is still in progress, then the second tap has no effect and only one charge attempt is sent.

**FEAT-14.SPEC-002-AC-07:** Given Sam attempts to reach this screen directly, when the attempt is made, then he sees "You don't have access to billing for this household." and no form is shown.

**FEAT-14.SPEC-002-AC-08:** Given Maya loses connectivity while filling in payment details, when she taps Subscribe, then the banner "Upgrading requires a connection. Your choice is saved -- reconnect to finish." appears and Subscribe stays disabled until reconnection.

**FEAT-14.SPEC-002-AC-09:** Given Maya's household is upgraded to paid from another session while this screen remains open, when she taps Subscribe here, then the attempt is rejected with "Your household is already on the paid tier." and she is redirected to FEAT-14.SPEC-001.

**FEAT-14.SPEC-002-AC-10:** Given Maya navigates away while her submission is still processing, when the submission later succeeds, then the tier updates the next time she opens FEAT-14.SPEC-001, and the confirmation notification still arrives.

**FEAT-14.SPEC-002-AC-11:** Given an unauthenticated visitor opens a link to this screen, when the link resolves, then they are redirected to the sign-in screen and land on FEAT-14.SPEC-001 after signing in, not this form.

**FEAT-14.SPEC-002-AC-12:** Given Maya's session expires while she has entered a period choice and partial payment details, when she re-authenticates, then her period choice is restored to this screen.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 5 (empty, filling, submitting, error, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
