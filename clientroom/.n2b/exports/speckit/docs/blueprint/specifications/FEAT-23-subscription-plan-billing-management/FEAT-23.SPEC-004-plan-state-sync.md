---
document_type: spec
spec_type: automation
spec_id: FEAT-23.SPEC-004
spec_name: Plan State Sync
spec_slug: plan-state-sync
parent_feature: FEAT-23
parent_feature_name: Subscription Plan & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 24
---

# Automation Spec: Plan State Sync

## Overview

**Name:** Plan State Sync
**ID:** FEAT-23.SPEC-004
**Type:** Automation
**Purpose:** Applies the subscription-billing capability's reported outcomes and period-end/renewal events to the Subscription Plan record's tier and status.
**Parent Feature:** FEAT-23 -- Subscription Plan & Billing Management

## Scope and Non-Goals

**In Scope:**
- Reconciling subscribe-succeeded, downgrade-confirmed, renewal-failed, renewal-succeeded, retry-succeeded, and period-end events (from FEAT-23.SPEC-003) into the Subscription Plan's tier, billing_cycle, and status
- Counting failed renewal retries and clearing the failure record when a retry succeeds
- Applying the grace/retry window rule (FEAT-23.SPEC-007) that lapses a plan whose failed charge is never recovered
- Recording every resulting tier or status change for FEAT-23.SPEC-008's confirmation and failure emails, for the Activity & Audit Trail (FEAT-13.SPEC-003), and for FEAT-23.SPEC-005's re-evaluation
- Discarding billing events for a plan removed by account deletion (FEAT-24.SPEC-004)

**Non-Goals:**
- Submitting the original charge, downgrade, or cancellation request -- owned by FEAT-23.SPEC-003 (Subscription Billing Processing); this automation only reconciles what that spec's inbound events report.
- Deciding whether Nadia is downgrade-eligible -- owned by FEAT-23.SPEC-005 (Downgrade Eligibility Detection); this automation only applies a tier change once Nadia has accepted an offer and the charge is confirmed.
- Recording the cancellation request itself -- owned by FEAT-23.SPEC-006 (Cancel Subscription); this automation applies the resulting tier/status change only once the period-end event arrives.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Subscribe charge succeeded | FEAT-23.SPEC-003 (Subscription Billing Processing) | Fires when the capability confirms a subscribe charge (from tier Free, status Active or Lapsed) | Plan reference, confirmed tier (Paid), selected billing_cycle, event timestamp |
| Subscribe attempt failed | FEAT-23.SPEC-003 | Fires when a subscribe charge does not complete | Plan reference, specific failure reason, event timestamp |
| Downgrade confirmed (billing stopped) | FEAT-23.SPEC-003 | Fires when the capability confirms it has stopped billing after Nadia accepted the downgrade offer -- no charge is involved | Plan reference, event timestamp |
| Downgrade rejected | FEAT-23.SPEC-003 | Fires when the capability rejects the stop-billing request | Plan reference, specific rejection reason, event timestamp |
| Renewal charge failed | FEAT-23.SPEC-003 | Fires when an existing Paid plan's renewal charge does not complete, and again for each failed retry of that charge | Plan reference, specific failure reason, attempt timestamp |
| Renewal succeeded | FEAT-23.SPEC-003 | Fires when an existing Paid plan's billing cycle renews on schedule | Plan reference, new billing-period boundaries, event timestamp |
| Retry succeeded | FEAT-23.SPEC-003 | Fires when Nadia's retry of a failed renewal charge is confirmed | Plan reference, new billing-period boundaries, event timestamp |
| Period-end reached | FEAT-23.SPEC-003 | Fires when the capability reports the paid period has ended for a plan that was cancelled | Plan reference, current active_client_count (read live from FEAT-01) |
| Grace window exhausted | FEAT-23.SPEC-007 (Plan Limit & Access Authorization Rules) | Fires when a plan has held status Charge failed for platform parameter: `subscription-charge-grace-window-days` with no successful retry | Plan reference, first-failure timestamp |
| Account deletion hold or completion | FEAT-24.SPEC-004 (Account Deletion Processing) | Fires when the account's entities are marked pending-delete (hold phase) and again when the Subscription Plan is hard-deleted | Freelancer Account reference, phase (hold / deleted) |

## Processing Logic

1. Receive the reported event and the plan reference it concerns. Compare the event's own timestamp with the timestamp of the last event applied to this plan; an event older than the last applied one is discarded as stale (event time, not arrival time, decides).
2. If the plan no longer exists (account deletion completed, FEAT-24.SPEC-004), discard the event with no effect. If the plan is pending-delete (hold phase), apply the event normally so a restored account is current, but send no email (FEAT-23.SPEC-008 cancels pending deliveries at completion).
3. For **subscribe charge succeeded**: set tier to Paid, billing_cycle to the confirmed selection, status to Active; clear any failure record. Steps 12-13 follow.
4. For **subscribe attempt failed**: make no write to the plan. Tier, status, and billing_cycle stay exactly as they were (Free with Active or Lapsed), no grace window starts, no lapse can follow, and the failed-charge alert email does not fire. Pass the specific reason to FEAT-23.SPEC-001, which shows it inline to Nadia on the Subscribe flow.
5. For **downgrade confirmed**: read the live active_client_count. If it is within platform parameter: `free-tier-active-client-limit`, set tier to Free and status to Active; if it exceeds the limit (clients were added after Nadia accepted), set tier to Free and status to Lapsed. In both cases clear billing_cycle in the same write and clear any failure record. The downgrade takes effect immediately, is not a charge, and carries no proration or refund. Steps 12-13 follow.
6. For **downgrade rejected**: make no write. The plan stays Paid and Active; pass the specific reason to FEAT-23.SPEC-001 with the offer left available.
7. For **renewal charge failed**: if status is Active, set status to Charge failed, record the specific reason, the failure timestamp (which starts the grace window), and retry attempts used as 0; leave tier and billing_cycle unchanged (the plan stays Paid and fully usable). If status is already Charge failed (this is a failed retry), add 1 to retry attempts used and replace the recorded reason, keeping the original first-failure timestamp; no additional alert fires. If status is Cancelled -- ends at period end, or the plan is Free, discard the event (no renewal is expected). Record a first failure for the failed-charge alert (FEAT-23.SPEC-008).
8. For **renewal succeeded**: if status is Active, keep status and tier, advance the billing-period boundaries, and send no email. If status is Charge failed, process it exactly as **retry succeeded** (step 9). If the plan is Cancelled -- ends at period end (the capability renews nothing after acknowledging cancellation) or is Free (including Lapsed, where a charge landed before billing was stopped), discard the event with no state change.
9. For **retry succeeded**: set status from Charge failed to Active, clear the first-failure timestamp, recorded reason, and retry attempts used (this ends the grace window), and advance the billing-period boundaries. A successful retry counts as the renewal for that cycle, so no second renewal event is expected, but it is not announced: no plan-change email fires and any failed-charge alert still pending delivery is cancelled (FEAT-23.SPEC-008). Tier and billing_cycle are unchanged.
10. For **period-end reached**: applies to a plan with status Cancelled -- ends at period end; if the capability reports a period end for a Paid plan that still shows Active or Charge failed (the cancellation was acknowledged by the capability but never recorded locally, FEAT-23.SPEC-006), apply it the same way because the capability's report is authoritative. Read the live active_client_count. If it is within the limit, set tier to Free, status to Active; if it exceeds the limit, set tier to Free, status to Lapsed. In both cases clear billing_cycle in the same write. Steps 12-13 follow.
11. For **grace window exhausted**: set tier to Free, status to Lapsed, clear billing_cycle in the same write, and clear the failure record. Hand a stop-billing request to FEAT-23.SPEC-003 so the capability does not keep charging a plan that is now Lapsed (retried per that spec until acknowledged).
12. After every tier or status write (steps 3, 5, 7, 9, 10, 11): notify FEAT-23.SPEC-005 of the change so it re-evaluates the downgrade offer; record the change for FEAT-23.SPEC-008 (steps 3, 5, 7-first-failure, 10, 11 only; never for retry attempts, retry-succeeded, or routine renewal); and report the change as a record-worthy event (event type, prior and new tier/status, timestamp, actor "Automatic" or Nadia where her action caused it) to FEAT-13.SPEC-003 (Activity Entry Recording) for the append-only trail. The trail write is FEAT-13's responsibility and retries there until it succeeds; it never reverses or delays a plan change that the billing capability has already made authoritative.
13. In every case, confirm the write completes before returning control, so no screen or automation reading the plan ever observes an intermediate, undefined state.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Upgrade applied | Subscribe charge succeeded | tier=Paid, billing_cycle set, status=Active | Plan & Billing Screen shows the Paid tier and new client capacity; upgrade email sent | FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003 |
| Subscribe attempt failed (no change) | Subscribe charge does not complete | None -- tier, status, billing_cycle unchanged | Plan & Billing Screen shows the reason inline with a "Try again" action; no email | FEAT-23.SPEC-001 |
| Downgrade applied | Downgrade confirmed, live active_client_count within the free-tier limit | tier=Free, status=Active, billing_cycle cleared; billing stopped immediately, no charge | Plan & Billing Screen shows the Free tier; downgrade email sent | FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003 |
| Downgrade applied over the limit | Downgrade confirmed, live active_client_count exceeds the limit | tier=Free, status=Lapsed, billing_cycle cleared | Plan & Billing Screen shows the Lapsed state; lapse email sent (in place of the downgrade email) | FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003, FEAT-01.SPEC-008 |
| Downgrade rejected (no change) | Capability rejects the stop-billing request | None -- plan stays Paid, Active | Plan & Billing Screen shows the rejection reason with the offer still available; no email | FEAT-23.SPEC-001 |
| Charge-failed status set | First failed renewal charge on a Paid, Active plan | status=Charge failed, reason, first-failure timestamp, retry attempts used=0; tier and billing_cycle unchanged | Plan & Billing Screen shows the reason inline with a Retry action; failed-charge alert email sent | FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003 |
| Retry failed | Failed retry while status is Charge failed | retry attempts used +1, reason replaced; status, tier, timestamp unchanged | Plan & Billing Screen banner shows the new reason; once retry attempts used equal platform parameter: `subscription-charge-retry-count` the Retry control is replaced by the retries-used message; no email | FEAT-23.SPEC-001 |
| Charge recovered | Retry succeeded (or renewal succeeded while Charge failed) | status=Active; failure record cleared; billing-period boundaries advance | Charge failed banner clears; no email; pending failed-charge alert cancelled | FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003 |
| Renewal recorded (no visible change) | Renewal succeeded while Active | Billing-period boundaries advance; status/tier unchanged | None -- routine renewal is silent by design | -- |
| Plan freed at period end | Period-end event, live active_client_count within the free-tier limit | tier=Free, status=Active, billing_cycle cleared | Plan & Billing Screen shows the Free tier on next view; paid-plan-ended email sent | FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003 |
| Plan lapsed at period end | Period-end event, live active_client_count exceeds the limit | tier=Free, status=Lapsed, billing_cycle cleared | Plan & Billing Screen shows the Lapsed state with an explanation that existing clients remain reachable but growth is blocked; lapse email sent | FEAT-23.SPEC-001, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003, FEAT-01.SPEC-008 |
| Plan lapsed after grace window | Grace window exhausted with no successful retry | tier=Free, status=Lapsed, billing_cycle cleared, failure record cleared; stop-billing request handed to FEAT-23.SPEC-003 | Same as above | FEAT-23.SPEC-001, FEAT-23.SPEC-003, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003, FEAT-01.SPEC-008 |
| Event discarded | Event is stale, or the plan was deleted by account deletion, or (renewal event) status is Cancelled/Free | None | None | -- |
| Sync failure | The reconciliation write itself cannot complete | No change is applied -- the prior state is preserved rather than left half-written | Plan & Billing Screen continues showing the last known state with no false confirmation; the automation retries automatically | FEAT-23.SPEC-001 |

## Data Model

**Reads:** Subscription Plan -- current tier, status, billing_cycle, failure record, active_client_count (the last read live from Client & Project Management, FEAT-01, at downgrade and period-end reconciliation).
**Creates:** None.
**Updates:** Subscription Plan -- tier, billing_cycle, status, failure record (reason, first-failure timestamp, retry attempts used), billing-period boundaries, per the outcome applied.
**Deletes:** None.

## Business Rules

- XBR-23: Lapsing never removes existing data or client-portal reachability -- it only blocks adding or reactivating clients beyond the free-tier limit until Nadia upgrades again or archives clients enough to fit within it. A Lapsed plan is always tier Free with billing_cycle cleared (FEAT-23.SPEC-007), so Subscribe/Upgrade is available to recover.
- The Charge failed status never triggers a mid-session lockout: already-active client work stays fully accessible for the entire grace window (Shared UI Pattern, Feature Breakdown Brief; FEAT-23.SPEC-007).
- Only a failed renewal charge on a Paid plan produces Charge failed and a grace window. A failed subscribe attempt and a rejected downgrade leave the plan untouched (no status change, no grace window, no alert email).
- A failed renewal charge does not immediately demote the plan's tier -- it only sets status to Charge failed and starts the grace window (FEAT-23.SPEC-007); only exhausting the grace window without a successful retry produces Lapsed. Exhausting the retry count does not lapse the plan early; the window's end does.
- A downgrade is Paid to Free, effective immediately when the capability confirms billing has stopped; it is not a charge and no proration or refund is calculated.
- The grace window's threshold values (platform parameter: `subscription-charge-retry-count`, platform parameter: `subscription-charge-grace-window-days`) are owned by FEAT-23.SPEC-007; this automation applies them without redefining them.
- Routine renewal produces no confirmation email or in-app notice, and neither does a recovered charge -- only a change to tier or status other than recovery (upgrade, downgrade, plan ended at period end, or lapse) triggers a plan-change email, per product-features.md's Communications field and FEAT-23.SPEC-008.

## Edge Cases

- **A charge-failed event and a successful retry arrive in quick succession** -- The most recent event by event time wins: if the retry's success timestamp is later than the failure, status returns to Active immediately and the grace window is cleared; a failure timestamp that arrives after an already-applied success is discarded as stale.
- **The grace window closes at the exact moment a retry succeeds** -- The retry is evaluated against the grace-window boundary as inclusive of its end moment (FEAT-23.SPEC-007); a retry confirmed at or before that instant is applied as a success, and the automatic lapse does not fire.
- **Retry count is used up while the window is still open** -- The plan stays Charge failed and fully usable; the Retry control is replaced by the retries-used message; the lapse happens at the window's end.
- **Period-end reconciliation runs while Nadia is actively adding a client in another session** -- The client-add (FEAT-01.SPEC-008) re-checks the plan's current state at the moment of its own commit (reject-with-refresh, per the dependency map's Contention note); whichever of the two operations commits first is authoritative for the other's next check.
- **Concurrent trigger firing (a renewal-succeeded and a charge-failed event for the same plan at effectively the same time)** -- These two outcomes are mutually exclusive by event time: the later event, by its own reported timestamp, is authoritative and the earlier one is treated as superseded (the same event-time-not-arrival-time rule FEAT-23.SPEC-003 applies).
- **Trigger fires while a previous run is in flight for the same plan** -- A second sync for the same Subscription Plan record cannot begin until the first commits, since there is exactly one plan per account and only one billing relationship generates events for it; syncs for different freelancers' plans proceed independently and never queue behind each other.
- **Sync write itself fails after an event is received** -- The plan's prior state is preserved (not left half-applied), the automation retries automatically, and no confirmation email fires until the write actually succeeds -- an email must never announce a change that did not take effect.
- **Account deletion (FEAT-24.SPEC-004) reaches the hold phase or completes while an event is in flight** -- During the hold phase the event is applied normally (the hold is reversible) but emits no email; once the Subscription Plan is hard-deleted, the in-flight run and any later billing event for that plan are discarded with no effect and no feedback.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-003 (Subscription Billing Processing) | Triggered by (inbound) and Triggers (outbound) | Every inbound billing event hands its consequence here; this automation hands back a stop-billing request after a grace-window lapse |
| FEAT-23.SPEC-007 (Plan Limit & Access Authorization Rules) | Triggered by (inbound) | The grace-window-exhausted condition fires this automation |
| FEAT-23.SPEC-001 (Plan & Billing Screen) | Affects (outbound) | Displays the resulting tier and status, and the subscribe-failed and downgrade-rejected reasons |
| FEAT-23.SPEC-005 (Downgrade Eligibility Detection) | Triggers (outbound) | Every tier or status write notifies it to re-evaluate the downgrade offer |
| FEAT-23.SPEC-006 (Cancel Subscription) | References (inbound) | Cancellation recorded there produces the period-end event this automation later applies |
| FEAT-23.SPEC-008 (Plan & Billing Notifications) | Triggers (outbound) | Every tier/status change (except routine renewal and recovery) and every first failed renewal charge fires a plan-change or failed-charge email |
| FEAT-01.SPEC-008 (Active Client Limit Enforcement, FEAT-01) | Affects (outbound) | Reads the resulting plan status and tier to gate client capacity (XBR-23) |
| FEAT-13.SPEC-003 (Activity Entry Recording, FEAT-13) | Triggers (outbound) | Every committed tier/status change is reported as an append-only trail entry (Brief Cross-Feature Touchpoint); FEAT-13.SPEC-003's own trigger list is owned by FEAT-13 |
| FEAT-24.SPEC-004 (Account Deletion Processing, FEAT-24) | Triggered by (inbound) | Account deletion marks the plan pending-delete, then hard-deletes it; billing events for a deleted plan are discarded |

## Analytics and Success Signals

- **plan_upgraded** (from_tier, to_tier: paid, billing_cycle) -- supports success-metrics.md: "Free-to-Paid Conversion"
- **plan_downgraded** (from_tier, to_tier: free, resulting_status) -- N/A -- no Stage 2 metric measures downgrade volume directly; product-features.md's Signals field names plan_downgrade_offered (emitted by FEAT-23.SPEC-005), not the applied downgrade, so this event is retained for completeness of the tier-change record without a Stage 2 citation.
- **subscription_charge_failed** (action: subscribe / renewal / renewal_retry, reason) -- N/A -- no Stage 2 metric measures charge-failure frequency; product-features.md's Signals field names this event to keep the failure path observable, matching FEAT-23.SPEC-003's own citation of the same gap.
- **charge_recovered** (retry_attempts_used) -- N/A -- no Stage 2 metric measures recovery of failed renewal charges; retained so the effectiveness of the retry window is observable.
- **plan_lapsed** (reason: grace_window_exhausted / period_end_over_limit / downgrade_over_limit) -- supports success-metrics.md: "Free-to-Paid Conversion" (a lapse after a failed charge or an over-limit period end is the negative outcome the conversion metric's 14-day window is measured against)

## Acceptance Criteria

**FEAT-23.SPEC-004-AC-01:** Given Nadia's subscribe charge is confirmed by FEAT-23.SPEC-003, when this automation processes the event, then tier is set to Paid, billing_cycle is set to her selection, and status is set to Active.

**FEAT-23.SPEC-004-AC-02:** Given Nadia has accepted a downgrade offer and the capability confirms billing has stopped, when this automation processes the event with her live active_client_count within the free-tier limit, then tier is set to Free, status is set to Active, billing_cycle is cleared, and no charge was made.

**FEAT-23.SPEC-004-AC-03:** Given a Paid, Active plan's renewal charge fails, when this automation processes the event, then status is set to Charge failed with the specific reason, first-failure timestamp, and retry attempts used of 0 recorded, and tier and billing_cycle are left unchanged.

**FEAT-23.SPEC-004-AC-04:** Given an existing Paid plan's billing cycle renews successfully, when this automation processes the renewal event, then status remains Active with no confirmation email sent.

**FEAT-23.SPEC-004-AC-05:** Given a cancelled plan's paid period ends and Nadia's active-client count is within the free-tier limit, when this automation processes the period-end event, then tier is set to Free, status is set to Active, and billing_cycle is cleared.

**FEAT-23.SPEC-004-AC-06:** Given a cancelled plan's paid period ends and Nadia's active-client count exceeds the free-tier limit, when this automation processes the period-end event, then tier is set to Free, status is set to Lapsed, and billing_cycle is cleared.

**FEAT-23.SPEC-004-AC-07:** Given a plan has held Charge failed status for platform parameter: `subscription-charge-grace-window-days` with no successful retry, when this automation processes the grace-window-exhausted condition, then tier is set to Free, status is set to Lapsed, billing_cycle is cleared, and a stop-billing request is handed to FEAT-23.SPEC-003.

**FEAT-23.SPEC-004-AC-08:** Given Nadia's plan is Charge failed within the grace window, when she continues using her already-active client work, then nothing about that work is blocked.

**FEAT-23.SPEC-004-AC-09:** Given a plan just transitioned to Lapsed, when Nadia's existing clients are viewed, then no data is lost and every existing client portal remains reachable.

**FEAT-23.SPEC-004-AC-10:** Given a renewal-charge-failed event and a later successful-retry event both arrive for the same plan, when this automation reconciles them by event time, then the later success wins and status returns to Active.

**FEAT-23.SPEC-004-AC-11:** Given a retry succeeds at the exact instant the grace window would otherwise close, when this automation evaluates the boundary, then the retry is honored as a success and no lapse occurs.

**FEAT-23.SPEC-004-AC-12:** Given the sync write for a confirmed event fails, when this automation detects the failure, then the plan's prior state is preserved, no confirmation email fires, and the automation retries automatically.

**FEAT-23.SPEC-004-AC-13:** Given a renewal-succeeded and a charge-failed event for the same plan arrive at effectively the same time, when this automation reconciles them, then the event with the later reported timestamp is authoritative.

**FEAT-23.SPEC-004-AC-14:** Given two different freelancers' plans each receive an event at effectively the same time, when this automation processes both, then each plan is reconciled independently with no interference between them.

**FEAT-23.SPEC-004-AC-15:** Given Nadia's plan is Free (Active or Lapsed) and her subscribe charge fails, when this automation processes the event, then tier, status, and billing_cycle are unchanged, no grace window starts, no failed-charge alert email is sent, and the reason is passed to FEAT-23.SPEC-001 to show inline.

**FEAT-23.SPEC-004-AC-16:** Given Nadia's downgrade request is rejected by the billing capability, when this automation processes the event, then the plan stays Paid and Active, no email is sent, and the reason is passed to FEAT-23.SPEC-001.

**FEAT-23.SPEC-004-AC-17:** Given Nadia's status is Charge failed and her retry succeeds, when this automation processes the retry-succeeded event, then status is set to Active, the first-failure timestamp, reason, and retry attempts used are cleared, billing-period boundaries advance, no plan-change email is sent, and any failed-charge alert still pending delivery is cancelled.

**FEAT-23.SPEC-004-AC-18:** Given Nadia's status is Charge failed and a retry fails, when this automation processes the renewal-charge-failed event, then retry attempts used increase by 1, the reason is replaced, the original first-failure timestamp is kept, and no additional alert email is sent.

**FEAT-23.SPEC-004-AC-19:** Given Nadia's live active_client_count exceeds the free-tier limit at the moment a confirmed downgrade is processed, when this automation applies it, then tier is set to Free, status is set to Lapsed, billing_cycle is cleared, and the lapse email is sent instead of the downgrade email.

**FEAT-23.SPEC-004-AC-20:** Given the capability reports a period end for a Paid plan whose local status still shows Active because the cancellation was acknowledged but not recorded, when this automation processes the event, then it applies the period-end outcome exactly as for a Cancelled plan.

**FEAT-23.SPEC-004-AC-21:** Given this automation commits any tier or status change other than a routine renewal, when the write completes, then it notifies FEAT-23.SPEC-005 and reports the change to FEAT-13.SPEC-003 as an append-only trail event, and a trail-write retry never reverses the plan change.

**FEAT-23.SPEC-004-AC-22:** Given Nadia's account has been hard-deleted through FEAT-24.SPEC-004, when a billing event for her former plan arrives, then it is discarded with no effect and no feedback.

**FEAT-23.SPEC-004-AC-23:** Given Nadia's account is in the pending-delete hold phase, when a billing event arrives for her plan, then it is applied to the plan and no email is sent.

**FEAT-23.SPEC-004-AC-24:** Given Nadia's status is Charge failed and her retry attempts used equal platform parameter: `subscription-charge-retry-count` with the grace window still open, when this automation evaluates the plan, then status stays Charge failed, the Retry control is replaced by the retries-used message on FEAT-23.SPEC-001, and the lapse occurs only when the window ends.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 10 | 10 |
| Outcome Paths | 14 | 14 |
| Business Rules | 7 | 7 |
| Edge Cases | 8 | 8 |
