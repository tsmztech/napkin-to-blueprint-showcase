---
document_type: spec
spec_type: automation
spec_id: FEAT-29.SPEC-009
spec_name: Account Reopening
spec_slug: account-reopening
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Account Reopening

## Overview

**Name:** Account Reopening
**ID:** FEAT-29.SPEC-009
**Type:** Automation
**Purpose:** Restores a Closing account to Active when the Pro signs back in during the cooling-off period, leaving the booking page down until the Pro separately resumes it.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Restoring Pro Account.status from Closing to Active on the Pro's explicit request during the cooling-off period
- Confirming that reopening leaves the subscription cancelled and the booking page down, both unchanged by this automation

**Non-Goals:**
- Resuming the booking page itself -- owned by FEAT-27 (Pro Profile & Booking Page Settings); reopening restores the account only, the Pro separately resumes bookings through FEAT-27's pause/resume control
- Restarting the subscription -- owned by FEAT-18 (Pro Subscription Billing & Account Management); a reopened Pro who wants to take bookings again subscribes again through FEAT-18, since XBR-26 requires an active subscription for the booking link to go live
- Executing the original closure or its cooling-off clock -- owned by FEAT-29.SPEC-008 (Account Closure Orchestration); this automation only reverses that clock's outcome before it fires
- Reopening after permanent deletion has already occurred -- excluded per this feature's Non-Goals: a cross-account or shared sign-in identity does not exist, and once data is deleted there is no account record left to restore

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro requests reopening | FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Pro Account.status is Closing (cooling-off period not yet expired) and the Pro taps "Reopen my account" | Pro Account reference |

## Processing Logic

1. Receive the reopening request with the Pro Account reference.
2. Verify Pro Account.status is currently Closing and the cooling-off period has not yet expired (the deletion check in FEAT-29.SPEC-008 has not begun processing this account).
3. Set Pro Account.status to Active, clearing the recorded closure request date.
4. Leave the Subscription in its cancelled state and the booking page down -- neither is touched by this step.
5. Signal FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) that the account is restored, for display.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Reopened | Status is Closing and the cooling-off period has not expired | Pro Account.status set to Active; closure request date cleared | FEAT-29.SPEC-005 shows "Your account is reopened" and navigates to FEAT-29.SPEC-003 | FEAT-29.SPEC-005, FEAT-29.SPEC-003 |
| Reopening refused (deletion already begun) | The scheduled deletion check (FEAT-29.SPEC-008) has already started processing this account in the current run | No status change; the account proceeds to permanent deletion | FEAT-29.SPEC-005 shows "This account can no longer be reopened." (no further Pro-facing surface exists once deletion completes) | FEAT-29.SPEC-008 |
| Reopening no-op (already Active) | Status is already Active (e.g., a second reopening attempt from another device) | No data change | FEAT-29.SPEC-005 reflects the already-Active state | FEAT-29.SPEC-005 |

## Data Model

**Reads:** Pro Account.status, closure request date.
**Creates:** None.
**Updates:** Pro Account.status (Closing -> Active); Pro Account's closure request date (cleared).
**Deletes:** None.

## Business Rules

- Reopening restores everything intact except the booking page, which stays down until the Pro separately resumes it through FEAT-27 (Cross-Feature Touchpoints) -- this automation never flips the booking page's own pause/resume state.
- Reopening never restarts the Subscription -- a reopened Pro subscribes again through FEAT-18 if they want to take bookings again, consistent with XBR-26's go-live prerequisite.
- Reopening is available at any point during the cooling-off period and becomes unavailable the instant the scheduled deletion check has begun processing that account (FEAT-29.SPEC-008) -- there is no partial reopening state, per the reopening cutoff defined in FEAT-29.SPEC-013 (Account Closure & Retention Rules), which this automation enforces.
- Once permanent deletion completes, no reopening path exists for that account (Non-Goal) -- a returning Pro after deletion has no prior account to reach.

## Edge Cases

- **Pro requests reopening from two devices at nearly the same time** -- The first request to commit transitions the account to Active; the second is a no-op against an already-Active account (idempotent), and its screen reflects the current state.
- **Concurrent trigger firing -- reopening is requested at the exact moment the scheduled deletion check (FEAT-29.SPEC-008) begins processing the same account** -- Whichever commits first is authoritative: if reopening's status write lands before the deletion check reads the account's status for its own run, the account is excluded from that run's deletion and reopening succeeds; if the deletion check has already selected the account for processing in the current run, the reopening request is refused with "This account can no longer be reopened."
- **Trigger fires while a previous run is in flight -- two reopening requests from the same session in quick succession** -- The second is ignored while the first is processing (FEAT-29.SPEC-005's own double-tap guard); no duplicate status write occurs.
- **Pro reopens then immediately requests closure again** -- Treated as a fresh closure request; FEAT-29.SPEC-005's upcoming-bookings review runs again from a clean state, and a new closure (if confirmed) starts a new cooling-off clock from the current date, not a continuation of the previous one.
- **Pro reopens the account and finds their booking page still down** -- Expected behavior, not an error: the Pro is directed to FEAT-27's pause/resume control (and, if they want new bookings, FEAT-18's subscription flow) to fully restore live operation.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Triggered by (inbound) | "Reopen my account" starts this automation |
| FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Affects (outbound) | Shows the resulting Active state or refusal |
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Affects (outbound) | Reflects the restored account after reopening |
| FEAT-29.SPEC-008 (Account Closure Orchestration) | References (inbound) | This automation reverses that automation's cooling-off outcome, when it has not yet fired |
| FEAT-29.SPEC-013 (Account Closure & Retention Rules) | References (inbound) | Rule spec that defines the cooling-off cutoff after which reopening is refused; this automation enforces that cutoff |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (outbound, implied) | The Pro separately resumes the booking page here after reopening |
| FEAT-18 (Pro Subscription Billing & Account Management) | References (outbound, implied) | The Pro separately re-subscribes here if they want to take bookings again |

## Analytics and Success Signals

- **account_reopened** (days_remaining_at_reopen) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list

## Acceptance Criteria

**FEAT-29.SPEC-009-AC-01:** Given Talia's account is Closing with 12 days remaining in the cooling-off period, when she requests reopening, then Pro Account.status is set to Active and the closure request date is cleared.

**FEAT-29.SPEC-009-AC-02:** Given Talia's account is reopened, when she checks her Subscription, then it remains cancelled -- reopening does not restart it.

**FEAT-29.SPEC-009-AC-03:** Given Talia's account is reopened, when she checks her booking page, then it remains down until she separately resumes it through FEAT-27.

**FEAT-29.SPEC-009-AC-04:** Given the scheduled deletion check has already begun processing Talia's account in the current run, when she attempts reopening in that same window, then the request is refused with "This account can no longer be reopened." and deletion proceeds.

**FEAT-29.SPEC-009-AC-05:** Given Talia's account status is already Active (a second reopening attempt from another device), when the second request processes, then no data change occurs and the screen reflects the already-Active state.

**FEAT-29.SPEC-009-AC-06:** Given Talia's account has already been permanently deleted, when she attempts to reopen it, then no reopening path exists.

**FEAT-29.SPEC-009-AC-07:** Given Talia reopens her account and later requests closure again, when the new closure is confirmed, then a new cooling-off clock starts from the current date, not a continuation of the previous one.

**FEAT-29.SPEC-009-AC-08:** Given Talia requests reopening twice in quick succession from the same session, when the second request arrives while the first is processing, then it is ignored and no duplicate status write occurs.

**FEAT-29.SPEC-009-AC-09:** Given Talia requests reopening from two devices at nearly the same time, when both process, then only the first commits the Active transition and the second is a no-op.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 3 | 3 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
