---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-14.SPEC-005
spec_name: Billing State & Refund Rules
spec_slug: billing-state-refund-rules
parent_feature: FEAT-14
parent_feature_name: Subscription & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 37
acceptance_criteria_count: 16
---

# Logic/Rule Spec: Billing State & Refund Rules

## Overview

**Name:** Billing State & Refund Rules
**ID:** FEAT-14.SPEC-005
**Type:** Logic/Rule
**Purpose:** Governs valid-payment requirements, downgrade/period-switch timing, the grace period, and the no-partial-refund and organiser-only money-flow constraints on the Subscription record.
**Parent Feature:** FEAT-14 -- Subscription & Billing Management
**Governed Entity:** Subscription

## Scope and Non-Goals

**In Scope:**
- Valid-payment requirements for upgrading and for a grace-period retry
- Timing rules for downgrade, cancellation, and billing-period switches (always at period end / next renewal, never mid-period)
- The 7-day grace period following a renewal payment failure
- The no-partial-refund rule for the unused remainder of a period
- The organiser-only money-flow constraint

**Non-Goals:**
- Who may view or act on billing screens -- governed by FEAT-14.SPEC-006 (Tier & Billing Access Authorization); this spec governs the state and timing rules those screens must respect, not who reaches them
- Executing the Subscription write once a rule permits a change -- owned by FEAT-14.SPEC-008 (Apply Subscription Change); this spec defines what is allowed and when, not the write itself
- Detecting and reporting the payment failure or its retry outcome -- owned by FEAT-14.SPEC-007 (Payment Failure & Grace Period Handling) and FEAT-14.SPEC-009 (Payment Processing Integration); this spec defines the grace-period rule those specs enforce

## Governed Entity

**Entity:** Subscription
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| tier | enum (free, paid) | The household's current plan tier |
| billing_period | enum (none, monthly, yearly) | The household's billing cadence; none applies while tier is free |
| billing_state | enum (Active, Payment failed, Cancelled, Reverted to free) | The household's current billing status. Cancelled marks a confirmed cancellation that has not yet resolved -- set immediately on cancellation confirmation, paid features stay active, and it resolves to Reverted to free at current_period_end_date. Reverted to free is the terminal state for either a cancellation resolution or a grace-period lapse; a downgrade never passes through Cancelled and never resolves to Reverted to free -- it stays Active throughout and remains Active once tier reverts to free |
| billing_history | list | Chronological record of charges, outcomes, and amounts, visible to the organiser |
| current_period_end_date | date | The current paid period's end date -- equivalently the next renewal date while no change is pending, and the resolution date for any pending_change already scheduled against it. Null while tier is free |
| pending_change | enum (none, period_switch, downgrade, cancellation) | The single change, if any, scheduled to take effect at current_period_end_date. Distinct from billing_state: a pending downgrade or period switch leaves billing_state at Active, while a pending cancellation is the one case that also sets billing_state to Cancelled |
| pending_change_new_period | enum (none, monthly, yearly) | The incoming billing_period for a pending period switch; none unless pending_change is period_switch |
| payment_failure_date | date | The date of the most recent unresolved renewal payment failure; source for computing the grace-period end date (payment_failure_date + 7 days). Null unless billing_state is Payment failed |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-14.SPEC-002 | Upgrade to Paid | On submit -- valid payment required before an upgrade can succeed |
| FEAT-14.SPEC-003 | Billing & Payment Management | On submit -- period-switch timing rule applied when Maya requests a switch |
| FEAT-14.SPEC-004 | Downgrade / Cancel | On submit -- downgrade/cancellation timing and no-partial-refund rule applied when Maya confirms |
| FEAT-14.SPEC-007 | Payment Failure & Grace Period Handling | During processing -- grace-period length and retry rules applied |
| FEAT-14.SPEC-008 | Apply Subscription Change | During processing -- every write to tier, billing_period, or billing_state validates against this spec's timing and state rules before committing |
| FEAT-14.SPEC-009 | Payment Processing Integration | On submit -- valid-payment-details format rules applied to charge and retry requests |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| tier | Must be one of free, paid | Always | On every write | "Plan tier must be Free or Paid." | Yes |
| billing_period | Must be none while tier is free; must be monthly or yearly while tier is paid | Conditional on tier | On every write | "A billing period is required for a paid subscription." | Yes |
| billing_state | Must be one of Active, Payment failed, Cancelled, Reverted to free | Always | On every write | "Billing state must be a recognized value." | Yes |
| billing_state | Cannot be Payment failed or Cancelled while tier is free | Conditional on tier | On every write | "The free tier has no payment-failed or cancelled state." | Yes |
| billing_history | No validation beyond data type -- entries are appended by FEAT-14.SPEC-008 and FEAT-14.SPEC-009, never edited or removed by any role | Always | -- | -- | -- |
| current_period_end_date | Must be a valid date while tier is paid; must be null while tier is free | Conditional on tier | On every write | "A current period end date is required for a paid subscription." | Yes |
| pending_change | Must be one of none, period_switch, downgrade, cancellation; must be none while tier is free | Conditional on tier | On every write | "No change can be pending on a free-tier subscription." | Yes |
| pending_change_new_period | Must be monthly or yearly only when pending_change is period_switch; must be none for every other value of pending_change | Conditional on pending_change | On every write | "A new billing period only applies to a pending period switch." | Yes |
| payment_failure_date | Must be a valid date only when billing_state is Payment failed; must be null for every other billing_state | Conditional on billing_state | On every write | "A payment failure date only applies while billing_state is Payment failed." | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Valid payment required to upgrade | tier, billing_state | Upgrading from free to paid requires payment details that pass FEAT-14.SPEC-009's format and processing checks before tier can be set to paid | "Valid payment details are required to upgrade." |
| Downgrade is scheduled at period end without a billing_state change | tier, billing_period, billing_state, pending_change, current_period_end_date | A downgrade request never sets tier to free immediately while billing_state is Active on a paid tier; it sets pending_change to downgrade against the existing current_period_end_date, and billing_state stays Active throughout the pending window and remains Active once tier reverts to free at that date | "Downgrading takes effect at the end of your current billing period, not immediately." |
| Cancellation is recorded as Cancelled immediately and resolves at period end | tier, billing_period, billing_state, pending_change, current_period_end_date | A cancellation request sets billing_state to Cancelled immediately on confirmation (tier and billing_period stay unchanged and paid features remain active) and sets pending_change to cancellation against the existing current_period_end_date; when that date is reached, tier is set to free, billing_period is set to none, and billing_state is set to Reverted to free | "Cancelling takes effect at the end of your current billing period, not immediately." |
| Period switch takes effect at next renewal | billing_period, pending_change, pending_change_new_period, current_period_end_date | A billing_period change requested while tier is paid and billing_state is Active is recorded as pending_change = period_switch with pending_change_new_period set, applied only when current_period_end_date (the next renewal date) is reached, never mid-period | "Your billing period will change at your next renewal, not immediately." |
| Grace period is exactly 7 days | billing_state, payment_failure_date | A billing_state of Payment failed automatically reverts to Reverted to free if unresolved for 7 days from payment_failure_date, per FEAT-14.SPEC-007 | N/A -- system-timed transition, not a user-facing validation error |
| No partial refund | billing_state, billing_history | A downgrade, cancellation, or grace-expiry reversion never generates a refund entry in billing_history for the unused remainder of a period | "No refund is issued for the unused portion of your current billing period." |
| Organiser-only money flow | tier, billing_history | Every charge, retry, or refund-adjacent entry in billing_history is attributed to the household's own organiser; no money flow originates from or is directed to any other household (per XBR-20 and scope-boundaries.md SC-10) | "Billing applies only to your own household -- no money passes between households." |
| pending_change is exclusive | pending_change, pending_change_new_period | At most one of period_switch, downgrade, or cancellation can be pending for a Subscription at a time; confirming a downgrade or cancellation while a period_switch is already pending clears the period_switch (and its pending_change_new_period) entirely -- it never applies | "This replaces your previously requested billing-period change, which will no longer take effect." |

## Authorization Rules

{The base per-role question of who may view plan tier, view billing detail, or act on billing at all is governed by FEAT-14.SPEC-006 (Tier & Billing Access Authorization) -- this spec does not duplicate that table. The rows below cover only the billing-state and timing conditions layered on top of Maya's (Organiser's) own access, since those conditions are specific to this spec's rules rather than to role-based access.}

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Upgrade to paid | Maya (Organiser) | Only while tier is free (per FEAT-14.SPEC-006 for the base role check) | While tier is already paid: the attempt is rejected with "Your household is already on the paid tier." |
| Switch billing period | Maya (Organiser) | Only while tier is paid, billing_state is Active, and pending_change is none | While tier is free: no switch action shown. While billing_state is not Active: switch action disabled with "You can't switch billing periods while your billing state is {state}." While pending_change is downgrade or cancellation: no switch action shown, since the household is already resolving to free |
| Downgrade to free | Maya (Organiser) | Only while tier is paid and pending_change is not already downgrade or cancellation | While tier is free: no downgrade action shown. While pending_change is already downgrade or cancellation: the action is replaced by "Downgrade scheduled for {current_period_end_date}" / "Cancellation scheduled for {current_period_end_date}" with no further downgrade action to take |
| Cancel subscription | Maya (Organiser) | Only while tier is paid and billing_state is not already Cancelled | While tier is free: no cancel action shown. While billing_state is already Cancelled: the action is replaced by "Cancellation scheduled for {current_period_end_date}" with no further cancel action to take |
| Update payment details | Maya (Organiser) | Only while tier is paid | While tier is free: no payment-details section shown (no payment method exists to update) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| tier | free | On Household creation (FEAT-01.SPEC-011) | Yes -- Maya upgrades via FEAT-14.SPEC-002 |
| billing_period | none | On Household creation, while tier is free | Yes -- set to monthly or yearly on upgrade; switchable afterward |
| billing_state | Active | On Household creation, and restored on every successful upgrade or grace-period retry | No -- billing_state is always system-derived from payment and timing outcomes, never directly set by any role |
| billing_history | Empty list | On Household creation | No -- entries are appended only by FEAT-14.SPEC-008 and FEAT-14.SPEC-009 |
| current_period_end_date | Set to one billing_period ahead of the upgrade date on upgrade; advanced by one billing_period on every successful renewal or applied period switch; null on downgrade/cancellation/grace-expiry reversion to free | On upgrade, renewal, and period-switch application (FEAT-14.SPEC-008) | No -- system-derived, never directly set by any role |
| pending_change | none | On Household creation, and cleared back to none whenever FEAT-14.SPEC-008 applies the pending change at current_period_end_date | Set to period_switch, downgrade, or cancellation only via a confirmed request on FEAT-14.SPEC-003 or FEAT-14.SPEC-004 | No -- system-derived from a confirmed request, never directly set |
| pending_change_new_period | none | On Household creation, and cleared back to none whenever pending_change resolves or is superseded | Set only when pending_change is period_switch | No -- system-derived |
| payment_failure_date | null | On Household creation, and cleared on a successful upgrade or grace-period retry | Set by FEAT-14.SPEC-007 the moment a renewal payment failure is reported | No -- system-derived, never directly set by any role |

## Business Rules

- The payment grace period is 7 days from a renewal payment failure; if unresolved, the household reverts to free automatically, per FEAT-14.SPEC-007.
- Cancelling keeps paid features active until the end of the period already paid for; no partial refund is made for the unused remainder, per product-features.md's Validation & Limits.
- All money flows are between the organiser and the product through the payment-processing capability -- no money passes between households, per product-features.md's Validation & Limits and scope-boundaries.md SC-10.
- A downgrade or cancellation never retroactively restricts the household's own history -- every past plan, rating, recipe, pantry item, and the shared list stay fully available (XBR-05), consistent with scope-boundaries.md SC-18.
- Every Subscription write is routed exclusively through FEAT-14.SPEC-008 -- no screen or automation writes tier, billing_period, or billing_state directly, per the Feature Breakdown Brief's Shared Context.
- Cancellation and downgrade resolve to different terminal billing_state values: a cancellation is recorded as billing_state Cancelled immediately on confirmation and resolves to Reverted to free once current_period_end_date is reached, matching feature-overview.md's Entity-Lifecycle Coverage Matrix ("Active → Cancelled → Reverted to free at period end"). A downgrade never uses Cancelled -- it stays Active throughout its pending window and remains Active once tier reverts to free, since Reverted to free is reserved for a cancellation resolution or a grace-period lapse.
- current_period_end_date is the single date every pending_change resolves against; it is only ever set or advanced by FEAT-14.SPEC-008 on upgrade, renewal, or a period-switch application -- no screen edits it directly.

## Edge Cases

- **Downgrade requested on the exact last day of the current paid period** -- The reversion is scheduled for that period's end date like any other downgrade; if the period ends before the request is processed, the reversion applies at the literal boundary, not one day later.
- **Grace period reaches exactly 7 days with no resolution** -- The household reverts to free at the 7-day boundary; a resolution (successful retry) arriving in the same processing window as the 7-day boundary is honored if it completes before the reversion automation runs, per FEAT-14.SPEC-007's own ordering.
- **Period switch requested, then a downgrade confirmed before the next renewal** -- pending_change moves from period_switch to downgrade (pending_change_new_period is cleared); the switch never applies, since there is no paid period left for it to take effect on.
- **Cancellation confirmed while billing_state is already Cancelled (a stale confirmation resubmitted, e.g., from a second open tab)** -- The request is a no-op: billing_state stays Cancelled, pending_change remains cancellation against the same current_period_end_date, and no duplicate scheduling or billing_history entry is created.
- **Payment succeeds for an upgrade, but the household's tier was changed by a concurrent action moments earlier (e.g., Maya's household is already paid from another session)** -- The upgrade attempt here is rejected with "Your household is already on the paid tier."; no duplicate charge is recorded, since valid-payment-required applies only to a genuine free-to-paid transition.
- **A refund-adjacent entry is attempted for the unused remainder of a cancelled period** -- No such entry is ever created; the no-partial-refund rule blocks it at the rule level, not only at the confirmation-screen level, so no code path can bypass it by skipping FEAT-14.SPEC-004's screen.
- **Riley's support session ends (Support Request resolved) while viewing plan tier** -- Access is revoked at that moment; any further attempt to view returns "This support session has ended."

## Acceptance Criteria

**FEAT-14.SPEC-005-AC-01:** Given Maya's household is free tier, when she submits an upgrade with valid payment details, then the tier is permitted to change to paid.

**FEAT-14.SPEC-005-AC-02:** Given Maya's household is free tier, when she submits an upgrade with payment details that fail FEAT-14.SPEC-009's checks, then the upgrade is blocked with "Valid payment details are required to upgrade."

**FEAT-14.SPEC-005-AC-03:** Given Maya's household is paid with billing_state Active, when she confirms a downgrade, then the change is scheduled for the current period's end date, not applied immediately.

**FEAT-14.SPEC-005-AC-04:** Given Maya's household is paid, when she requests a billing-period switch, then the new period is recorded as pending and applies only at the next renewal date.

**FEAT-14.SPEC-005-AC-05:** Given a renewal payment fails, when billing_state is set to Payment failed, then a 7-day grace period begins during which paid features remain active.

**FEAT-14.SPEC-005-AC-06:** Given the grace period reaches 7 days with no successful retry, when the boundary is reached, then the household reverts to free tier automatically.

**FEAT-14.SPEC-005-AC-07:** Given Maya cancels her subscription, when the reversion to free occurs at period end, then no refund entry is created in billing_history for the unused remainder of the period.

**FEAT-14.SPEC-005-AC-08:** Given any billing_history entry is created, when it is examined, then it is attributed to the household's own organiser and never to or from another household.

**FEAT-14.SPEC-005-AC-09:** Given Maya's household is already paid, when she attempts to submit another upgrade, then it is rejected with "Your household is already on the paid tier."

**FEAT-14.SPEC-005-AC-10:** Given Maya's household is free tier, when she looks for a "Switch billing period" action, then none is shown, since billing_period only applies while tier is paid.

**FEAT-14.SPEC-005-AC-11:** Given Maya's household has billing_state Payment failed, when she attempts to switch billing period, then the action is disabled with "You can't switch billing periods while your billing state is Payment failed."

**FEAT-14.SPEC-005-AC-12:** Given a downgrade is requested exactly on the last day of the current paid period, when the request is confirmed, then the reversion is scheduled for that exact period-end date.

**FEAT-14.SPEC-005-AC-13:** Given Maya has a pending billing-period switch and then confirms a downgrade before the next renewal, when the downgrade completes, then the pending period switch never takes effect.

**FEAT-14.SPEC-005-AC-14:** Given every past plan, rating, recipe, pantry item, and the shared list existed before a downgrade, when the reversion to free completes, then all of them remain fully available and unrestricted.

**FEAT-14.SPEC-005-AC-15:** Given Maya's household is paid with billing_state Active, when she confirms a cancellation, then billing_state is set to Cancelled immediately, tier and billing_period remain unchanged, paid features stay active, and pending_change is recorded as cancellation against current_period_end_date.

**FEAT-14.SPEC-005-AC-16:** Given Maya's household has billing_state Cancelled and current_period_end_date is reached, when the reversion resolves, then tier is set to free, billing_period is set to none, and billing_state is set to Reverted to free (never Active).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 9 | 9 |
| Cross-Field Rules | 8 | 8 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 8 | 8 |
| Business Rules | 7 | 7 |
| Edge Cases | 7 | 7 |
