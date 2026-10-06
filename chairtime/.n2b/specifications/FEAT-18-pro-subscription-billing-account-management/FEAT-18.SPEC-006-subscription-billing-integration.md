---
document_type: spec
spec_type: integration
spec_id: FEAT-18.SPEC-006
spec_name: Subscription Billing Integration
spec_slug: subscription-billing-integration
parent_feature: FEAT-18
parent_feature_name: Pro Subscription Billing & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Integration Spec: Subscription Billing Integration

## Overview

**Name:** Subscription Billing Integration
**ID:** FEAT-18.SPEC-006
**Type:** Integration
**Purpose:** Carries outbound subscribe, update-payment-method, and cancel requests to the payment-processing capability, and receives inbound renewal-outcome and dispute events, so the Pro's subscription billing is handled without the product ever holding card data.
**Parent Feature:** FEAT-18 -- Pro Subscription Billing & Account Management

## Scope and Non-Goals

**In Scope:**
- Submitting the first subscribe charge, payment-method updates, and cancellation requests to the payment-processing capability
- Receiving renewal-outcome events (success, failure) and reflecting them on the Subscription
- Receiving card-issuer dispute notices related to the subscription charge itself
- User-facing behavior when the capability is slow, unavailable, or rejects a request
- Disclosure to the Pro about what data is shared with the capability

**Non-Goals:**
- Choosing the payment-processing vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate, so this spec stays vendor-neutral throughout
- The grace-period clock and pause-trigger logic that reacts to a renewal failure -- owned by FEAT-18.SPEC-003 and FEAT-18.SPEC-004; this spec delivers the raw outcome event and stops there
- Client deposit, balance, or payout processing -- a separate relationship with the same capability category, owned by FEAT-07.SPEC-005 (deposits) and FEAT-28.SPEC-006 (payout accounts); this spec covers only the Pro's own subscription billing
- Writing the Subscription's price field after creation -- FEAT-18.SPEC-003 is the sole Subscription.price writer after creation (on an announced price change's effective date, per FEAT-18.SPEC-005); this spec only sets the price on the initial create from the first-charge confirmation and never re-writes it
- The screens' layout and content that surface this integration's outcomes -- owned by FEAT-18.SPEC-001 and FEAT-18.SPEC-002; this spec defines only the capability contract they consume

## Capability Category

**Category:** Payment processing
**Dependency Source:** ASMP-31 -- "Payment-processing capability... required to... bill the pro's own monthly subscription" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Payment processing — Pro subscription billing" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-18, FEAT-15)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Talia subscribes with a card during onboarding and her first charge is confirmed | Subscribe during onboarding with a card | FEAT-18.SPEC-001 (Subscribe Screen) |
| Talia updates the payment method on file | Update the payment method on file | FEAT-18.SPEC-002 (Billing & Subscription Management Screen) |
| Talia cancels her subscription, effective at the end of the current billing period | Cancel the subscription at any time, effective at the end of the current billing period | FEAT-18.SPEC-002 (Billing & Subscription Management Screen) |
| Talia's monthly subscription renews automatically without her needing to re-enter payment details | Subscribe during onboarding with a card | FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) |
| Account closure cancels Talia's subscription the same way, without a separate billing flow | Cancel the subscription at any time (invoked by account closure, XBR-20) | FEAT-29 (Pro Sign-In & Account Lifecycle) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| First subscribe charge request | Subscription -- price (platform parameter: `subscription-price`); Pro Account -- reference (never bank or identity details, which the Pro provides directly to the capability's own surface) | Talia submits the Subscribe Screen (FEAT-18.SPEC-001) | The capability must know what to charge and which account to attribute it to |
| Card details (entered directly into the capability's own surface, never touching the product's own code) | Not a product entity -- entered by the Pro directly on the capability's surface | Talia enters payment details on FEAT-18.SPEC-001 or the payment-method update on FEAT-18.SPEC-002 | The capability handles all card data per SC-11; the product never receives or transmits the raw card values |
| Renewal charge request | Subscription -- price, payment_method_reference | Each monthly billing-cycle date, for an Active Subscription (FEAT-18.SPEC-003) | The capability must know what to charge and against which stored payment method |
| Payment-method update request | Subscription -- payment_method_reference (the new card is entered directly into the capability's surface, as above) | Talia submits "Update payment method" (FEAT-18.SPEC-002) | The capability must confirm and store the new card, replacing the reference on file |
| Cancellation request | Subscription -- reference | Talia confirms cancel (FEAT-18.SPEC-002), or FEAT-29 invokes cancellation as part of account closure (XBR-20) | The capability must stop billing this Subscription at the end of the current period |

Bank details, identity details, the Pro's studio address, and every other Pro Account and Booking field never leave the product through this integration.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| First-charge confirmation | The capability confirms the first subscribe charge succeeded | Subscription -- created with status Active, next_billing_date, payment_method_reference, price |
| First-charge decline | The capability declines the first subscribe charge | No Subscription created; decline reason category surfaced to FEAT-18.SPEC-001 |
| Renewal outcome (success) | The capability confirms a monthly renewal charge succeeded | Subscription -- next_billing_date advanced (via FEAT-18.SPEC-003) |
| Renewal outcome (failure) | The capability reports a monthly renewal charge failed or was declined | Subscription -- status set to Payment Failed (via FEAT-18.SPEC-003); failure reason category recorded |
| Payment-method update confirmation | The capability confirms a new card was captured and stored | Subscription -- payment_method_reference updated |
| Payment-method update rejection | The capability rejects the new card | Subscription -- payment_method_reference unchanged; rejection reason category surfaced to FEAT-18.SPEC-002 |
| Cancellation confirmation | The capability confirms the subscription will stop billing at period end | Subscription -- status set to Cancelled (active to period end), via FEAT-18.SPEC-005's timing rule |
| Card-issuer dispute notice on a subscription charge | A dispute is raised against a subscription charge | Subscription -- no status field change (subscriptions carry no Disputed overlay in v1); the dispute is recorded for the Pro's own account activity and surfaced to Support (FEAT-19), consistent with XBR-22's dispute-evidence pattern for other financial records |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| First-charge succeeded | The capability confirms Talia's first subscribe charge | Subscription created (status Active) | Confirmation shown on FEAT-18.SPEC-001; onboarding continues and reports the go-live signal (XBR-26) | FEAT-18.SPEC-001, FEAT-15.SPEC-005 |
| First-charge declined | The capability declines Talia's first subscribe charge | No Subscription created | FEAT-18.SPEC-001 shows the decline reason inline with a retry action | FEAT-18.SPEC-001 |
| Renewal outcome received | Monthly renewal charge resolves, success or failure | Handled by FEAT-18.SPEC-003 (advance billing date or set Payment Failed) | Status banner (FEAT-18.SPEC-002) updates; renewal receipt or payment-failure notice fires (FEAT-18.SPEC-007) | FEAT-18.SPEC-003, FEAT-18.SPEC-002, FEAT-18.SPEC-007 |
| Payment-method update confirmed | The capability confirms a new card | payment_method_reference updated | FEAT-18.SPEC-002 shows "Payment method updated"; if the Subscription was in grace, a renewal retry follows (FEAT-18.SPEC-003) | FEAT-18.SPEC-002, FEAT-18.SPEC-003 |
| Payment-method update rejected | The capability rejects a new card | No change | FEAT-18.SPEC-002 shows the rejection reason inline with retry | FEAT-18.SPEC-002 |
| Cancellation confirmed | The capability confirms the subscription will stop billing at period end | Subscription status set to Cancelled (active to period end) | FEAT-18.SPEC-002's status banner updates; cancellation confirmation notification fires | FEAT-18.SPEC-002, FEAT-18.SPEC-007 |
| Subscription status changed | The Subscription's status changes to or from Active as a result of any inbound event above (first-charge success, renewal outcome, payment-method-update recovery, cancellation) | No additional data change beyond the event's own | No direct Pro-facing feedback; the change is reported to FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation), which re-evaluates the go-live check as an external-event trigger | FEAT-15.SPEC-005 |
| Subscription charge disputed | A card issuer raises a dispute against a subscription charge | Dispute recorded in the Pro's own account activity | The Pro sees the dispute noted in their account activity; Support can view it during a help request | FEAT-19 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-18.SPEC-001 (Subscribe Screen) | "Start subscription" shows "Processing payment, do not close this page"; after 10 seconds a note appears: "Still working -- this is taking longer than usual." | The button is disabled with "Subscription setup is temporarily unavailable. Try again in a few minutes." Nothing is charged and onboarding stays on this step. | The decline reason is shown inline in plain language: "This card was declined: {reason}. Try another card." No Subscription is created. |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen -- update payment method) | "Update payment method" shows a progress state; after 10 seconds: "Still working -- this is taking longer than usual." | The action is disabled with "Payment updates are temporarily unavailable. Your current payment method is unchanged. Try again in a few minutes." | The rejection reason is shown inline: "This card could not be confirmed: {reason}. Try another card." Previous payment method remains on file. |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen -- cancel) | The cancel confirmation shows a progress state after the Pro confirms; after 10 seconds: "Still working -- this is taking longer than usual." | The confirm action is disabled with "Cancellation could not be completed right now. Your subscription is unchanged. Try again in a few minutes." | N/A -- a cancellation request against an existing, confirmed Subscription is not a rejectable operation from the capability's side; there is no decline case for stopping future billing. |
| FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | N/A -- this is a background automation with no user-facing waiting state; a slow renewal charge simply resolves later, and the Subscription remains in its prior state until an outcome arrives | The renewal attempt is treated as inconclusive and retried on the next scheduled attempt within the automation's own retry handling; the Subscription is not moved to Payment Failed on a down-capability alone -- only on a genuine decline outcome | N/A -- "rejects" for a renewal is the same as the failure outcome already defined in FEAT-18.SPEC-003 (sets Payment Failed and starts the grace clock) |

## Consent and Disclosure

- **First subscription disclosure** -- Before Talia's first charge is submitted on FEAT-18.SPEC-001, the screen states plainly: "To start your subscription, your subscription price is charged monthly through an external payment service, which handles your card details directly. Chairtime never sees or stores your card number." This is descriptive text on the screen itself (not a separate consent gate), since entering payment details on the capability's own surface is itself the affirmative action.
- **Payment-method update disclosure** -- The "Update payment method" flow on FEAT-18.SPEC-002 carries the same statement: card details are entered directly with the external payment service and never reach the product's own code.
- **What is never shared** -- The Pro's studio address, sign-in credentials, client data, and every Booking and Client field stay entirely inside the product; only the Subscription's price and a payment-method reference (never the card itself) are exchanged with the capability, as detailed in Data Exchanged.

## Edge Cases

- **Renewal-outcome event arrives for a Subscription that was since cancelled** -- The event is recorded against the Subscription's history but produces no status change and no user feedback: a Cancelled (active to period end) Subscription is not moved back to Active or Payment Failed by a late-arriving renewal event for a charge that should not have been attempted (per FEAT-18.SPEC-003's "No renewal needed" rule).
- **The same renewal-outcome event is delivered twice** -- The second delivery changes nothing: a Subscription already advanced to Active with its new billing date, or already recorded as Payment Failed with its grace deadline set, is not changed again, and no duplicate notification fires.
- **Events arrive out of order (a payment-method-update confirmation arrives after a renewal-failure event for the same billing cycle)** -- The Subscription reflects the most recent event by its own event time, not arrival time: if the payment-method update's event time is later than the renewal failure's, the subsequent renewal retry (FEAT-18.SPEC-003) is what resolves the grace state, not the update confirmation alone.
- **Capability goes down mid-cancellation submission** -- If the cancellation was not confirmed, the Subscription's status is unchanged and FEAT-18.SPEC-002 shows the capability-down message; no half-cancelled state exists.
- **A dispute notice arrives for a subscription charge that has already been superseded by a later renewal** -- The dispute is recorded against the specific charge it names, in the Pro's account activity, without affecting the Subscription's current status; Chairtime never rules on the dispute (SC-17), consistent with how FEAT-16 handles deposit disputes.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-001 (Subscribe Screen) | Triggered by (inbound) | "Start subscription" initiates the first-charge request |
| FEAT-18.SPEC-001 (Subscribe Screen) | Affects (outbound) | First-charge outcome and degradation states surface here |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | Triggered by (inbound) | Payment-method-update and cancel actions initiate requests |
| FEAT-18.SPEC-002 (Billing & Subscription Management Screen) | Affects (outbound) | Update, cancel, and degradation outcomes surface here |
| FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Triggers (outbound) | Renewal-outcome inbound events fire this automation |
| FEAT-18.SPEC-003 (Subscription Renewal & Payment-Failure Processing) | Triggered by (inbound) | The automation submits renewal (and retry) charge requests through this integration |
| FEAT-18.SPEC-005 (Subscription Billing Rules) | References (inbound) | Processor-authority contention rule governs how inbound events are resolved against local state |
| FEAT-18.SPEC-007 (Subscription Billing Notifications) | Triggers (outbound) | Renewal outcomes and cancellation confirmation lead to their respective notifications |
| FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) -- within FEAT-15 (Pro Onboarding & Setup Wizard) | Triggers (outbound) | First-charge success reports the go-live signal (XBR-26); every Subscription status change to or from Active (the "Subscription status changed" inbound event) is an external-event trigger for that automation |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Triggered by (inbound) | Account closure (XBR-20) invokes cancellation through this integration |
| FEAT-19 (Platform Support Read-Only Access) | Affects (outbound) | A subscription-charge dispute notice is surfaced to Support during a help request |

## Analytics and Success Signals

- **subscription_first_charge_outcome** (outcome: succeeded / declined) -- supports success-metrics.md: "Subscription Retention"
- **subscription_payment_method_update_outcome** (outcome: succeeded / rejected) -- supports success-metrics.md: "Subscription Retention"
- **subscription_cancelled** (invoked via: self-service / account closure; fires when the payment-processing capability confirms the cancellation) -- supports success-metrics.md: "Subscription Retention"
- **subscription_capability_degradation_shown** (condition: slow / down / rejected; screen: spec ID) -- N/A -- no Stage 2 metric measures degradation frequency directly; retained so the product's tolerance for capability trouble on billing is observable

## Acceptance Criteria

**FEAT-18.SPEC-006-AC-01:** Given Talia submits her card details on the Subscribe Screen, when the capability confirms the first charge, then a Subscription is created with status Active and the go-live signal is reported to FEAT-15 (XBR-26).

**FEAT-18.SPEC-006-AC-02:** Given Talia submits her card details on the Subscribe Screen, when the capability declines the charge, then no Subscription is created and the decline reason is shown inline with a retry action.

**FEAT-18.SPEC-006-AC-03:** Given Talia's subscription reaches its monthly billing date, when the capability confirms the renewal charge, then the outcome is delivered to FEAT-18.SPEC-003 and her next billing date advances.

**FEAT-18.SPEC-006-AC-04:** Given Talia updates her payment method, when the capability confirms the new card, then her payment_method_reference is updated and she sees "Payment method updated."

**FEAT-18.SPEC-006-AC-05:** Given Talia updates her payment method, when the capability rejects the new card, then her previous payment method remains on file and she sees the rejection reason inline with retry.

**FEAT-18.SPEC-006-AC-06:** Given Talia confirms cancellation, when the capability confirms the subscription will stop billing at period end, then her Subscription's status is set to Cancelled (active to period end).

**FEAT-18.SPEC-006-AC-07:** Given a card issuer disputes one of Talia's subscription charges, when the dispute notice arrives, then it is recorded in her account activity and made visible to Support during a help request, without Chairtime ruling on the dispute.

**FEAT-18.SPEC-006-AC-08:** Given Talia taps "Start subscription" while the capability is unavailable, when the request cannot be sent, then the button is disabled with "Subscription setup is temporarily unavailable. Try again in a few minutes." and nothing is charged.

**FEAT-18.SPEC-006-AC-09:** Given Talia attempts to cancel while the capability is unavailable, when the confirm action is submitted, then it is disabled with "Cancellation could not be completed right now. Your subscription is unchanged. Try again in a few minutes."

**FEAT-18.SPEC-006-AC-10:** Given Talia has never subscribed before, when she reaches the payment step, then she sees the disclosure that her card details are handled directly by an external payment service and never stored by Chairtime.

**FEAT-18.SPEC-006-AC-11:** Given a renewal-outcome event arrives for a Subscription that was already cancelled, when the event is processed, then no status change occurs and no user feedback fires.

**FEAT-18.SPEC-006-AC-12:** Given the same renewal-outcome event is delivered twice, when the second delivery arrives, then the Subscription's state does not change again and no duplicate notification fires.

**FEAT-18.SPEC-006-AC-13:** Given a payment-method-update confirmation and a renewal-failure event arrive out of order for the same billing cycle, when both are processed, then the Subscription reflects the most recent event by its own event time, not arrival order.

**FEAT-18.SPEC-006-AC-14:** Given the capability goes down mid-cancellation submission before confirming, when Talia checks her subscription afterward, then its status is unchanged -- no half-cancelled state exists.

**FEAT-18.SPEC-006-AC-15:** Given Talia's Subscription status changes to or from Active (for example, first-charge success or a renewal failure), when the change is recorded, then the change is reported to FEAT-15.SPEC-005 as an external-event trigger and no Pro-facing message is produced by the report itself.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 5 | 5 |
| Inbound Events | 8 | 8 |
| Degradation Paths | 9 (4 screens/automations x 3 conditions = 12 cells; 3 N/A cells excluded from denominator per justified exclusion) | 9 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 5 | 5 |
