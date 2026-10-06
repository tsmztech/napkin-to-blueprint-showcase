---
document_type: spec
spec_type: integration
spec_id: FEAT-23.SPEC-003
spec_name: Subscription Billing Processing
spec_slug: subscription-billing-processing
parent_feature: FEAT-23
parent_feature_name: Subscription Plan & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 22
---

# Integration Spec: Subscription Billing Processing

## Overview

**Name:** Subscription Billing Processing
**ID:** FEAT-23.SPEC-003
**Type:** Integration
**Purpose:** Submits Nadia's own subscribe, stop-billing (downgrade), retry, and cancellation requests to the subscription-billing capability and receives back charge outcomes, renewal and period-end events, and failure reasons.
**Parent Feature:** FEAT-23 -- Subscription Plan & Billing Management

## Scope and Non-Goals

**In Scope:**
- Submitting a charge when Nadia subscribes (upgrade) and resubmitting a failed renewal charge when she taps Retry
- Requesting that billing stop immediately when Nadia accepts a downgrade offer (a downgrade is Paid to Free with no charge), and when a plan lapses after its grace window
- Relaying a cancellation request and waiting for the capability's acknowledgment, so the paid period continues to its natural end
- Receiving subscribe, downgrade, cancellation-acknowledged, renewal, retry, and period-end outcomes and handing their consequences to FEAT-23.SPEC-004 (or, for cancellation, FEAT-23.SPEC-006)
- User-facing behavior when the capability is slow, unavailable, or rejects a request
- Disclosure to Nadia about what billing data is shared with the capability

**Non-Goals:**
- Choosing the subscription-billing vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for this capability.
- Applying the confirmed tier or status change to the Subscription Plan record -- owned by FEAT-23.SPEC-004 (Plan State Sync), which this spec's inbound events route to.
- Holding, moving, or storing Nadia's own card or bank details -- excluded per BRIEF.md's Constraints and scope-boundaries.md (SC-10): those details are captured and held entirely inside the subscription-billing capability's own experience, never inside the product.
- Client-facing payment collection (invoices, deposits paid by Owen) -- entirely separate from this spec, which concerns only Nadia's own subscription to Clientroom; that capability is described by FEAT-10.SPEC-003 and connected through FEAT-32.

## Capability Category

**Category:** Subscription billing
**Dependency Source:** ASMP-31 -- "Subscription-billing capability for the freelancer's own plan" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Subscription billing for the freelancer's own plan" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-23)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Nadia subscribes to the paid tier and gains client capacity once the charge is confirmed | Upgrade -- subscribe to the paid tier when exceeding the free-tier client count | FEAT-23.SPEC-001 (Plan & Billing Screen), FEAT-23.SPEC-004 (Plan State Sync) |
| Nadia accepts a downgrade offer and her plan goes from Paid to Free, effective immediately, with billing stopped and no charge | Downgrade offer -- reduce plan when client count drops back below the threshold | FEAT-23.SPEC-001, FEAT-23.SPEC-004 |
| Nadia cancels her plan and it stays Paid and usable through the end of the current paid period, shown as cancelled only once the capability has acknowledged it | Cancel -- stop the paid plan at any time, effective at the end of the paid period | FEAT-23.SPEC-006 (Cancel Subscription), FEAT-23.SPEC-004 |
| Nadia's plan renews automatically each billing cycle without her having to re-subscribe | View current plan -- see tier and usage against its client limit (renewal keeps the tier current) | FEAT-23.SPEC-001, FEAT-23.SPEC-004 |
| A failed renewal charge is surfaced to Nadia with its specific reason and an immediate retry, without cutting off her already-active client work; a failed subscribe attempt is surfaced inline and changes nothing | Upgrade (failure path of the same capability) | FEAT-23.SPEC-001, FEAT-23.SPEC-004, FEAT-23.SPEC-008 (Plan & Billing Notifications) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Freelancer Account reference | Freelancer Account -- account reference only | Nadia subscribes, retries a charge, accepts a downgrade, or cancels; or a plan lapses after its grace window | Ties the billing relationship to the correct freelancer |
| Requested billing cycle | Subscription Plan -- billing_cycle (Nadia's selection) | Nadia subscribes | The capability must know whether to charge monthly or yearly |
| Subscribe request | Subscription Plan -- requested tier (Paid) | Nadia subscribes | Tells the capability what plan to charge going forward |
| Charge retry request | Subscription Plan -- reference to the failed renewal charge | Nadia taps Retry on the Charge failed banner | Asks the capability to attempt the failed charge again |
| Stop-billing request | Subscription Plan -- stop-billing-now flag | Nadia accepts a downgrade offer; or FEAT-23.SPEC-004 lapses a plan after the grace window | Tells the capability to end billing immediately and issue no further charges; carries no charge and no refund or proration request |
| Cancellation request | Subscription Plan -- cancellation flag | Nadia confirms Cancel (FEAT-23.SPEC-006) | Tells the capability to stop renewing at the end of the current paid period |

No card number, bank account detail, or any other payment credential ever leaves the product, because the product never captures them -- Nadia enters payment details directly inside the subscription-billing capability's own experience (ASMP-24, scope-boundaries.md SC-10).

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Subscribe charge succeeded outcome | A subscribe charge completes successfully | Subscription Plan -- tier, billing_cycle, status (via FEAT-23.SPEC-004) |
| Subscribe charge failed outcome with a plain-language failure reason | A subscribe charge does not complete | Nothing persisted on the plan; the reason is shown inline by FEAT-23.SPEC-001 (via FEAT-23.SPEC-004, which makes no write) |
| Stop-billing confirmed / rejected outcome | The capability confirms it has stopped billing, or rejects the request (with a plain-language reason) | Confirmed: Subscription Plan -- tier, billing_cycle, status (via FEAT-23.SPEC-004). Rejected: nothing persisted; the reason is shown inline |
| Renewal charge failed outcome with a plain-language failure reason | A renewal charge on an existing Paid plan does not complete, or a retry of it does not complete | Subscription Plan -- status, last failure reason, first-failure timestamp, retry attempts used (via FEAT-23.SPEC-004) |
| Renewal confirmed | A billing cycle renews successfully on an existing Paid plan | Subscription Plan -- status remains Active, billing period advances (via FEAT-23.SPEC-004) |
| Retry succeeded outcome | Nadia's retry of a failed renewal charge completes | Subscription Plan -- status returns to Active, failure record cleared, billing period advances (via FEAT-23.SPEC-004) |
| Cancellation acknowledged, with the current billing period's end date | The capability confirms it will not renew | Handed to FEAT-23.SPEC-006, which records status Cancelled -- ends at period end |
| Period-end event | A cancelled plan's current paid period ends | Subscription Plan -- tier, billing_cycle, and status transition (via FEAT-23.SPEC-004) |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Subscribe charge succeeded | Nadia's subscribe charge completes | Handed to FEAT-23.SPEC-004, which sets tier=Paid, billing_cycle as selected, status=Active | Plan screen shows the Paid tier and new client capacity; upgrade confirmation email sent | FEAT-23.SPEC-004, FEAT-23.SPEC-001, FEAT-23.SPEC-008 |
| Subscribe charge failed | A subscribe charge does not complete | Handed to FEAT-23.SPEC-004, which makes no write: tier, status, and billing_cycle stay as they were, no grace window starts | Plan screen shows "This charge couldn't be completed: {reason}." inline with "Try again"; no email | FEAT-23.SPEC-004, FEAT-23.SPEC-001 |
| Stop-billing confirmed (downgrade accepted) | The capability confirms billing has stopped after Nadia accepted the offer | Handed to FEAT-23.SPEC-004, which sets tier=Free, clears billing_cycle, and sets status=Active (or Lapsed if the live client count is over the free-tier limit) | Plan screen shows the Free tier; downgrade (or lapse) email sent | FEAT-23.SPEC-004, FEAT-23.SPEC-001, FEAT-23.SPEC-008 |
| Stop-billing rejected (downgrade accepted) | The capability rejects the request | Handed to FEAT-23.SPEC-004, which makes no write; the plan stays Paid and Active | Plan screen shows the reason inline with the offer still available; no email | FEAT-23.SPEC-004, FEAT-23.SPEC-001 |
| Renewal charge failed | An existing Paid plan's renewal charge does not complete, or a retry of it does not complete | Handed to FEAT-23.SPEC-004, which sets status=Charge failed with the specific reason (first failure) or increments retry attempts used (failed retry) | Plan screen shows the failure reason inline with an immediate Retry option (until retries are used up); failed-charge alert email sent on the first failure only | FEAT-23.SPEC-004, FEAT-23.SPEC-001, FEAT-23.SPEC-008 |
| Retry succeeded | Nadia's retry of a failed renewal charge completes | Handed to FEAT-23.SPEC-004, which returns status to Active and clears the failure record | Charge failed banner clears; no email (the pending failed-charge alert, if unsent, is cancelled) | FEAT-23.SPEC-004, FEAT-23.SPEC-001, FEAT-23.SPEC-008 |
| Renewal succeeded | An existing Paid plan's billing cycle renews on schedule | Handed to FEAT-23.SPEC-004, which keeps status=Active and advances the billing period | No user-facing interruption -- the plan simply continues; this is a silent success per the product's "no confirmation for routine renewal" design decision | FEAT-23.SPEC-004 |
| Cancellation acknowledged | The capability confirms Nadia's cancellation request | Handed to FEAT-23.SPEC-006, which sets status=Cancelled -- ends at period end | Plan screen shows the Cancelled explanation with the exact period-end date; cancellation confirmation email sent | FEAT-23.SPEC-006, FEAT-23.SPEC-001, FEAT-23.SPEC-008 |
| Period-end reached on a cancelled plan | The paid period Nadia's cancellation was scheduled against ends | Handed to FEAT-23.SPEC-004, which sets tier=Free and clears billing_cycle, with status Active (if active_client_count is within the free-tier limit) or Lapsed (if not) | Plan screen reflects the new tier/status on next view; paid-plan-ended email (Free) or lapse email (Lapsed) sent | FEAT-23.SPEC-004, FEAT-23.SPEC-001, FEAT-23.SPEC-008 |
| Stop-billing acknowledged after a lapse | The capability confirms it has stopped billing a plan that lapsed after its grace window | None -- the plan is already Free and Lapsed | None | FEAT-23.SPEC-004 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-23.SPEC-001 (Plan & Billing Screen) -- Subscribe action | The Subscribe button shows a progress state; after 10 seconds a note appears: "Still working -- this is taking longer than usual." The rest of the screen remains fully usable. | The button is disabled with "Billing is temporarily unavailable. Your current plan is unaffected -- try again in a few minutes." Nadia's existing client work is unaffected. | The screen shows the specific rejection reason in plain language: "This charge couldn't be completed: {reason}. Try again or use a different payment method inside your billing details." This is a failed attempt, not a status change: the plan is unchanged (tier, status, and billing_cycle as before), there is no Charge failed status, no grace window, and no email, and Nadia may try again without limit. |
| FEAT-23.SPEC-001 -- Accept downgrade offer (stop-billing request) | Same progress-then-note pattern as Subscribe. | Same disabled-with-message pattern as Subscribe; the downgrade offer remains available to accept once billing is back. | Reason shown inline: "We couldn't end your billing: {reason}. Your plan is unchanged -- try again." Nadia stays Paid and Active, the offer remains available, and no email is sent. |
| FEAT-23.SPEC-001 -- Cancel action | Progress state; after 10 seconds "Still working -- this is taking longer than usual." The plan is not shown as Cancelled until the capability acknowledges. | Cancel is disabled with "Billing is temporarily unavailable. Try cancelling again in a few minutes." Nadia's plan is not marked Cancelled and continues unaffected -- because a cancellation is recorded only after the capability acknowledges it, the plan can never show Cancelled while the capability is still set to renew. | N/A -- a cancellation request has no rejection path from the capability's side; a request that receives no acknowledgment is treated as the slow or down behavior above and leaves the plan unchanged. |
| FEAT-23.SPEC-001 -- Charge failed banner Retry | Retry shows a progress state; after 10 seconds the same "Still working" note appears. The attempt is counted against the retry limit only once an outcome is received. | Retry is disabled with "Billing is temporarily unavailable. Your plan stays usable until {grace_window_end_date} -- try again in a few minutes." The grace window is not extended for downtime. | The banner updates with the new specific reason; the attempt counts against platform parameter: `subscription-charge-retry-count` (FEAT-23.SPEC-007); when retries are used up the Retry control is replaced by the retries-used message. |

## Consent and Disclosure

- **First subscribe disclosure** -- The first time Nadia proceeds with Subscribe, after she has chosen a billing cycle, a notice appears before she is routed into the subscription-billing capability's own payment-entry experience: "To subscribe, your account reference and the plan you're choosing are shared with our billing partner. Your payment details are entered directly with them and never stored by Clientroom." Options: "Continue" and "Cancel." "Continue" routes her to the capability's payment-entry experience; "Cancel" returns her to the Plan & Billing Screen with nothing sent. Shown once; afterward, a "How billing works" link on the Plan & Billing Screen reopens the same notice for reading (with a single "Close" button, since no action is pending).
- **Downgrade and cancellation disclosure** -- Accepting a downgrade or cancelling reuses the same disclosure notice, since the same account reference and plan-change request are shared each time; Nadia is not asked to re-consent to a notice she has already seen, but the "How billing works" link remains available at every step.
- **What is never shared** -- No card number, bank detail, invoice content, client data, or any Subscription Plan field beyond the tier/billing-cycle/cancellation/stop-billing request itself ever leaves the product. This boundary is stated in the disclosure notice.

## Edge Cases

- **A second subscribe-charge-succeeded event arrives for a plan that already shows Paid (double submission or another session)** -- The second event changes nothing: the plan already reflects Paid with the confirmed billing cycle, and no duplicate confirmation email fires (FEAT-23.SPEC-008's deduplication rule).
- **The same charge-succeeded event is delivered twice** -- The second delivery changes nothing: a plan already set to Paid with the confirmed billing cycle stays as it is, and no duplicate confirmation email fires.
- **Events arrive out of order (renewal-succeeded before a still-pending renewal-charge-failed for the same cycle)** -- The plan reflects the most recent event by event time, not arrival time; an out-of-order renewal-succeeded that predates a charge-failed event is superseded once the charge-failed event's timestamp is recognized as later.
- **Period-end event arrives for an account that has since been deleted (FEAT-24)** -- The event is discarded with no effect and no user feedback fires, since the Subscription Plan record itself was already deleted as part of account deletion (feature-dependency-map.md, Entity: Subscription Plan, Lifecycle).
- **Capability goes down mid-Subscribe, after the disclosure notice was accepted but before a charge outcome is confirmed** -- The Plan & Billing Screen shows the capability-down message and the plan stays on its prior tier -- no half-upgraded state; Nadia can retry once billing is available again.
- **Nadia abandons the capability's payment-entry experience without completing it** -- No outcome is reported, so nothing changes; FEAT-23.SPEC-001 shows her plan unchanged with Subscribe available (its own return/abandon handling).
- **The capability acknowledges a cancellation but the local record cannot be written afterward** -- Owned by FEAT-23.SPEC-006 (the acknowledgment is already in hand; the record write retries there); if the capability's period-end event arrives first, FEAT-23.SPEC-004 applies it authoritatively.
- **The capability is unavailable when a stop-billing request must be relayed after a lapse** -- The plan is already Free and Lapsed and stays usable; the request is retried at platform parameter: `stop-billing-relay-retry-interval` until acknowledged, with no user-facing state. A renewal-succeeded event that lands on the Lapsed plan in the meantime is discarded by FEAT-23.SPEC-004 with no state change; refunding any such stray charge is handled by the billing partner's own process and is out of scope for this feature.
- **Nadia accepts a downgrade while a renewal charge is in flight** -- The stop-billing request and the renewal are ordered by event time; if the renewal completes first, it is applied then the downgrade proceeds against the current plan; if the stop-billing confirmation is first, the later renewal event is discarded for a Free plan.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-001 (Plan & Billing Screen) | Triggered by (inbound) | Subscribe, accept-downgrade, Cancel, and Retry actions initiate requests through this spec |
| FEAT-23.SPEC-001 | Affects (outbound) | Degradation states, disclosure notices, and attempt-failed reasons surface here |
| FEAT-23.SPEC-004 (Plan State Sync) | Triggers (outbound) and Triggered by (inbound) | Every inbound event hands its consequence to this automation; this automation hands back stop-billing requests after a lapse |
| FEAT-23.SPEC-006 (Cancel Subscription) | Triggered by (inbound) and Triggers (outbound) | A cancellation request is relayed to the capability through this spec, and the capability's acknowledgment is handed back for recording |
| FEAT-23.SPEC-005 (Downgrade Eligibility Detection) | References (inbound) | An accepted downgrade offer's stop-billing request flows through this spec |
| FEAT-23.SPEC-008 (Plan & Billing Notifications) | Triggers (outbound) | Subscribe-succeeded, stop-billing-confirmed, renewal-charge-failed, cancellation-acknowledged, and period-end events lead to plan-change and failed-charge emails |
| FEAT-24 (Data Export & Account Deletion) | References (inbound) | Account deletion removes the Subscription Plan; later period-end events for a deleted account are discarded |

## Analytics and Success Signals

- **subscription_charge_submitted** (action: upgrade / retry; billing_cycle) -- supports success-metrics.md: "Free-to-Paid Conversion"
- **subscription_charge_succeeded** (action: upgrade / renewal / retry) -- supports success-metrics.md: "Free-to-Paid Conversion"
- **subscription_charge_failed** (action: subscribe / renewal / retry, reason) -- N/A -- no Stage 2 metric measures charge failure frequency directly; product-features.md's Signals field names this event to keep failure visibility measurable, so it is retained here as the source of that signal for FEAT-23.SPEC-004 and FEAT-23.SPEC-008.
- **subscription_cancellation_relayed** () -- supports success-metrics.md: "Free-to-Paid Conversion" (cancellations are the inverse signal the conversion metric's 14-day window is measured against)
- **subscription_billing_stopped** (trigger: downgrade / lapse, outcome: confirmed / rejected) -- N/A -- no Stage 2 metric measures billing-stop outcomes; retained so the downgrade and lapse paths are observable.
- **billing_degradation_shown** (condition: slow / down / rejected) -- N/A -- no Stage 2 metric measures degradation frequency; retained so the product's tolerance for capability trouble is observable.

## Acceptance Criteria

**FEAT-23.SPEC-003-AC-01:** Given Nadia is on the free tier at her client limit, when she taps Subscribe, selects a billing cycle, accepts the disclosure notice, and the capability confirms the charge, then her Subscription Plan is handed to FEAT-23.SPEC-004 with tier=Paid and the selected billing_cycle.

**FEAT-23.SPEC-003-AC-02:** Given Nadia has accepted a downgrade offer, when the capability confirms it has stopped billing, then the confirmation is handed to FEAT-23.SPEC-004 as a Paid-to-Free downgrade, no charge was submitted, and billing_cycle is cleared by that spec.

**FEAT-23.SPEC-003-AC-03:** Given Nadia confirms Cancel, when the cancellation request is submitted, then it is relayed to the capability so the current paid period continues to its scheduled end, and the capability's acknowledgment is handed to FEAT-23.SPEC-006.

**FEAT-23.SPEC-003-AC-04:** Given an existing Paid plan's billing cycle renews successfully, when the renewal-succeeded event arrives, then the plan's status remains Active with no user-facing interruption.

**FEAT-23.SPEC-003-AC-05:** Given a cancelled plan's paid period ends and Nadia's active-client count is within the free-tier limit, when the period-end event arrives, then FEAT-23.SPEC-004 transitions tier to Free (billing_cycle cleared, status Active).

**FEAT-23.SPEC-003-AC-06:** Given a cancelled plan's paid period ends and Nadia's active-client count exceeds the free-tier limit, when the period-end event arrives, then FEAT-23.SPEC-004 sets tier to Free and status to Lapsed instead of Active.

**FEAT-23.SPEC-003-AC-07:** Given a renewal charge fails, when the charge-failed event arrives with a specific reason, then it is handed to FEAT-23.SPEC-004 for the Charge failed status and to FEAT-23.SPEC-008 for the failure alert email.

**FEAT-23.SPEC-003-AC-08:** Given Nadia taps Subscribe while the capability is slow, when 10 seconds pass without a response, then the note "Still working -- this is taking longer than usual." appears and the rest of the screen stays usable.

**FEAT-23.SPEC-003-AC-09:** Given Nadia taps Subscribe while the capability is down, when the request cannot be sent, then Subscribe is disabled with "Billing is temporarily unavailable. Your current plan is unaffected -- try again in a few minutes."

**FEAT-23.SPEC-003-AC-10:** Given Nadia taps Subscribe and the capability rejects the charge, when the rejection is received, then she sees "This charge couldn't be completed: {reason}. Try again or use a different payment method inside your billing details.", her plan is unchanged, no Charge failed status is set, and no email is sent.

**FEAT-23.SPEC-003-AC-11:** Given Nadia taps Cancel while the capability is down, when the request cannot be sent, then Cancel is disabled with "Billing is temporarily unavailable. Try cancelling again in a few minutes." and her plan is not marked Cancelled and continues unaffected.

**FEAT-23.SPEC-003-AC-12:** Given Nadia has never subscribed before, when she has chosen a billing cycle and proceeds, then the disclosure notice appears with "Continue" and "Cancel," and no data leaves the product until she chooses "Continue."

**FEAT-23.SPEC-003-AC-13:** Given a charge-succeeded event was already applied, when the same event is delivered a second time, then nothing changes and no duplicate confirmation email fires.

**FEAT-23.SPEC-003-AC-14:** Given a period-end event arrives for an account already deleted through FEAT-24, when the event is processed, then it is discarded with no effect and no user feedback.

**FEAT-23.SPEC-003-AC-15:** Given the capability goes down after Nadia accepts the disclosure notice but before a charge outcome is confirmed, when she reopens her plan, then it shows her prior tier unchanged -- no half-upgraded state.

**FEAT-23.SPEC-003-AC-16:** Given Nadia accepts a downgrade offer and the capability rejects the stop-billing request, when the rejection is received, then she sees the reason inline, her plan stays Paid and Active, the offer remains available, and no email is sent.

**FEAT-23.SPEC-003-AC-17:** Given Nadia's status is Charge failed, when she taps Retry and the capability confirms the charge, then the retry-succeeded outcome is handed to FEAT-23.SPEC-004 and no email is sent for it.

**FEAT-23.SPEC-003-AC-18:** Given Nadia's status is Charge failed, when she taps Retry and the capability reports a new failure, then the new reason is handed to FEAT-23.SPEC-004, counted as one retry attempt, and no additional failed-charge alert email fires.

**FEAT-23.SPEC-003-AC-19:** Given Nadia's status is Charge failed and the capability is down, when she views the banner, then Retry is disabled with "Billing is temporarily unavailable. Your plan stays usable until {grace_window_end_date} -- try again in a few minutes." and the grace window is not extended.

**FEAT-23.SPEC-003-AC-20:** Given a plan lapses after its grace window, when FEAT-23.SPEC-004 hands a stop-billing request to this spec and the capability is unavailable, then the request is retried at platform parameter: `stop-billing-relay-retry-interval` until acknowledged and Nadia sees no additional state or message.

**FEAT-23.SPEC-003-AC-21:** Given Nadia abandons the capability's payment-entry experience without completing it, when she returns to the Plan & Billing Screen, then her plan is unchanged and no outcome event was handed to FEAT-23.SPEC-004.

**FEAT-23.SPEC-003-AC-22:** Given a Subscribe attempt fails for a plan that is Free with status Lapsed, when the failure is handed to FEAT-23.SPEC-004, then the plan stays Free and Lapsed with no grace window, and Nadia may try again.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 5 | 5 |
| Inbound Events | 10 | 10 |
| Degradation Paths | 11 (4 screens x conditions, 1 N/A excluded) | 11 |
| Consent and Disclosure | 3 | 3 |
| Edge Cases | 9 | 9 |
