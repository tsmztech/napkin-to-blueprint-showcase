---
document_type: spec
spec_type: automation
spec_id: FEAT-23.SPEC-006
spec_name: Cancel Subscription
spec_slug: cancel-subscription
parent_feature: FEAT-23
parent_feature_name: Subscription Plan & Billing Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Automation Spec: Cancel Subscription

## Overview

**Name:** Cancel Subscription
**ID:** FEAT-23.SPEC-006
**Type:** Automation
**Purpose:** Cancels Nadia's paid plan by having the subscription-billing capability acknowledge that it will not renew, then recording the cancellation, with the plan remaining Paid and fully usable through the end of the current paid period.
**Parent Feature:** FEAT-23 -- Subscription Plan & Billing Management

## Scope and Non-Goals

**In Scope:**
- Handling a confirmed Cancel request from Nadia's Paid plan
- Handing the request to the billing capability (through FEAT-23.SPEC-003) and recording the cancellation only after the capability acknowledges it
- Setting status to Cancelled -- ends at period end, without changing tier or removing capacity immediately
- Defining what happens when the capability cannot acknowledge, or when the acknowledged cancellation cannot be recorded
- Reporting the recorded cancellation to the Activity & Audit Trail (FEAT-13.SPEC-003)

**Non-Goals:**
- Applying the tier change once the paid period actually ends -- owned by FEAT-23.SPEC-004 (Plan State Sync), which reacts to the period-end event this automation's relayed request eventually produces.
- Transmitting the cancellation to the subscription-billing capability and its slow/down messaging -- owned by FEAT-23.SPEC-003 (Subscription Billing Processing); this automation hands the request over and acts on the acknowledgment that comes back.
- Letting Nadia reverse a cancellation mid-period -- excluded per product-features.md: the Brief defines only Subscribe (a fresh upgrade) as the path back to Paid; no "undo cancellation" capability exists in the product definition.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia confirms Cancel in the cancellation dialog | FEAT-23.SPEC-001 (Plan & Billing Screen) | Fires only while tier is Paid and status is Active or Charge failed (FEAT-23.SPEC-007, Authorization Rules) | Plan reference, current tier, current billing_cycle |
| Cancellation acknowledged by the billing capability | FEAT-23.SPEC-003 (Subscription Billing Processing) | Fires when the capability confirms it will not renew; carries the current billing period's end date | Plan reference, period end date, acknowledgment timestamp |

## Processing Logic

One sequence, capability first: the plan is never recorded as Cancelled unless the billing capability has acknowledged the cancellation, so the record can never say "cancelled" while the capability is still set to charge.

1. Receive Nadia's confirmed cancellation request from the Plan & Billing Screen.
2. Confirm the plan's current tier is Paid and status is Active or Charge failed (re-checked authoritatively, not assumed from the screen's last load). If not, reject as stale (Outcome: Cancellation rejected -- stale state).
3. Hand the cancellation request to FEAT-23.SPEC-003 to relay to the subscription-billing capability. The plan is not changed at this point. If the capability is slow or down, FEAT-23.SPEC-003's messaging applies, no acknowledgment arrives, and the plan stays exactly as it was; Nadia may tap Cancel again later. There is no background relay retry, so no plan can be left shown as cancelled while the capability keeps charging.
4. On the capability's acknowledgment, re-read the plan. If it is still Paid with status Active or Charge failed, set status to Cancelled -- ends at period end, keep tier and billing_cycle unchanged, and store the period end date from the acknowledgment. If status was Charge failed, clear the failure record (first-failure timestamp, reason, retry attempts used) since the grace window no longer applies to a cancelled plan. If the plan has meanwhile changed (for example, it lapsed and billing was already stopped), treat the acknowledgment as moot and change nothing.
5. If the record write in step 4 fails, retry the write at platform parameter: `plan-record-write-retry-interval`, up to platform parameter: `plan-record-write-retry-count` attempts. While retrying, Nadia's screen shows "We're finishing your cancellation -- no action needed." (the acknowledgment is already in hand, so billing will not renew regardless). If all attempts fail, stop retrying (Outcome: Cancellation acknowledged but not recorded).
6. After a successful record write: notify FEAT-23.SPEC-005 so the downgrade offer is cleared, make the recorded cancellation available to FEAT-23.SPEC-001 (end-of-period explanation) and FEAT-23.SPEC-008 (cancellation confirmation email), and report the cancellation as a record-worthy event (event type, Nadia as actor, prior and new status, period end date, timestamp) to FEAT-13.SPEC-003 (Activity Entry Recording). The trail write is FEAT-13's responsibility and retries there; it never reverses the cancellation.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Cancellation recorded | Capability acknowledged, and the plan was still Paid and Active or Charge failed | status=Cancelled -- ends at period end; tier and billing_cycle unchanged; period end date stored; failure record cleared if present | Plan & Billing Screen shows a plain end-of-period explanation ("Your plan stays active through {period end date}. After that, it moves to the free tier or lapses depending on your client count at that time."); confirmation email sent | FEAT-23.SPEC-001, FEAT-23.SPEC-003, FEAT-23.SPEC-005, FEAT-23.SPEC-008, FEAT-13.SPEC-003 |
| Cancellation rejected -- stale state | Plan's state changed since Nadia's screen last loaded (e.g., already Cancelled, or a lapse already occurred) | None | Plan & Billing Screen refreshes to the current state and shows a message reflecting what actually happened: "Your plan status has changed -- here's the latest." | FEAT-23.SPEC-001 |
| Capability unavailable or no acknowledgment | The capability is slow, down, or does not acknowledge | None -- status is not changed and no relay is queued | FEAT-23.SPEC-003's messages ("Still working -- this is taking longer than usual." or "Billing is temporarily unavailable. Try cancelling again in a few minutes."); the plan continues as Paid and Active (or Charge failed) | FEAT-23.SPEC-001, FEAT-23.SPEC-003 |
| Cancellation record delayed | Capability acknowledged but the record write failed and is being retried | status unchanged until the write succeeds | Plan & Billing Screen shows "We're finishing your cancellation -- no action needed." until the write completes, then the Cancelled explanation | FEAT-23.SPEC-001 |
| Cancellation acknowledged but not recorded | The record write failed on every one of platform parameter: `plan-record-write-retry-count` attempts | status unchanged (Paid, Active or Charge failed); no email fires because no change was recorded | Plan & Billing Screen shows, in the session where Nadia cancelled, "Your billing partner has received your cancellation and will not renew your plan. Your plan ends on {period end date}; this page will show it as cancelled once our records catch up." The period-end event later reported by the capability is applied authoritatively by FEAT-23.SPEC-004, which sends the paid-plan-ended or lapse email. If Nadia taps Cancel again, the request is re-relayed and the capability's repeat acknowledgment is recorded as the same cancellation (no duplicate email or trail entry) | FEAT-23.SPEC-001, FEAT-23.SPEC-003, FEAT-23.SPEC-004 |

## Data Model

**Reads:** Subscription Plan -- current tier, status, billing_cycle, failure record.
**Creates:** None.
**Updates:** Subscription Plan -- status only (Cancelled -- ends at period end), the stored period end date, and clearing the failure record when cancelling from Charge failed.
**Deletes:** None.

## Business Rules

- Cancelling never changes tier immediately -- the plan stays Paid and fully usable through the end of the current paid period (product-features.md, Key Capabilities: "Cancel -- stop the paid plan at any time, effective at the end of the paid period").
- The billing capability's acknowledgment comes first and the local record second; the plan is never shown as Cancelled unless the capability has acknowledged it, and the screen never shows Cancelled while the capability may still renew.
- No refund or proration is calculated by this automation -- the paid period Nadia already paid for runs its full course; any refund policy question is out of scope for this feature (product-features.md's Non-Goals: independent of client-facing Invoicing & Payments).
- Cancelling is available at any time during an Active or Charge failed status -- Nadia does not need to wait for a specific point in her billing cycle to cancel. Cancelling from Charge failed ends the grace window; the plan then follows the Cancelled path to its period end.
- Once cancelled, the plan cannot be re-cancelled -- a second Cancel attempt against an already-Cancelled plan is rejected as a stale-state action (Edge Cases).

## Edge Cases

- **Nadia taps Cancel twice in quick succession** -- The second tap is disabled while the first request awaits acknowledgment; if it still arrives after the first has recorded Cancelled -- ends at period end, it is rejected as a stale-state action and the screen shows the already-cancelled state rather than double-processing.
- **The plan lapses (grace window exhausted) between Nadia loading the screen and tapping Cancel** -- The cancellation attempt is rejected as stale: Cancel is not a meaningful action against a Lapsed plan, and the screen refreshes to show the Lapsed state with its own recovery path (Subscribe again).
- **The plan lapses while the cancellation request is waiting for acknowledgment** -- The acknowledgment is treated as moot in step 4 (nothing to record); the screen refreshes to the Lapsed state.
- **Concurrent trigger firing (Nadia taps Cancel in two open sessions/tabs at the same time)** -- The first request to commit records the cancellation; the second is rejected as stale-state and the second session's screen refreshes to show the already-cancelled state.
- **Trigger fires while a previous cancellation is still awaiting the capability's acknowledgment** -- A second Cancel tap is disabled while the first is in flight (the screen's Cancel control shows a brief processing state), so no duplicate cancellation request is ever relayed for the same plan; a repeat acknowledgment that does arrive for an already-recorded cancellation is ignored.
- **Nadia's downgrade offer is showing when she cancels instead** -- The cancellation proceeds independently; once status is Cancelled -- ends at period end, this automation notifies FEAT-23.SPEC-005, whose cancellation trigger clears the flag immediately.
- **Account deletion (FEAT-24.SPEC-004) completes while the cancellation is awaiting acknowledgment or a record write is retrying** -- The Subscription Plan no longer exists; the acknowledgment and any pending retry are discarded with no effect and no email or trail entry is produced.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-23.SPEC-001 (Plan & Billing Screen) | Triggered by (inbound) | Confirmed Cancel action initiates this automation |
| FEAT-23.SPEC-001 | Affects (outbound) | End-of-period explanation, "finishing your cancellation" notice, and stale-state refresh surface here |
| FEAT-23.SPEC-003 (Subscription Billing Processing) | Triggers (outbound) and Triggered by (inbound) | The cancellation request is relayed to the subscription-billing capability through this spec, and the capability's acknowledgment returns through it |
| FEAT-23.SPEC-004 (Plan State Sync) | References (outbound) | The eventual period-end event this cancellation produces is applied by this automation, including when the cancellation was acknowledged but not recorded |
| FEAT-23.SPEC-005 (Downgrade Eligibility Detection) | Triggers (outbound) | A recorded cancellation clears any raised downgrade offer |
| FEAT-23.SPEC-008 (Plan & Billing Notifications) | Triggers (outbound) | The recorded cancellation fires the cancellation confirmation email |
| FEAT-13.SPEC-003 (Activity Entry Recording, FEAT-13) | Triggers (outbound) | The recorded cancellation is reported as an append-only trail entry (Brief Cross-Feature Touchpoint); FEAT-13.SPEC-003's own trigger list is owned by FEAT-13 |
| FEAT-24 (Data Export & Account Deletion) | References (inbound) | Account deletion removes the plan; pending acknowledgments and record retries for a deleted plan are discarded |

## Analytics and Success Signals

- **subscription_cancelled** (billing_cycle, tenure_days) -- supports success-metrics.md: "Free-to-Paid Conversion" (a cancellation is the negative outcome the conversion metric's ongoing measurement tracks against)
- **cancellation_record_delayed** (attempts_used, terminal: yes / no) -- N/A -- no Stage 2 metric covers cancellation record reliability; retained so the acknowledged-but-not-recorded path is observable.

## Acceptance Criteria

**FEAT-23.SPEC-006-AC-01:** Given Nadia's plan is Paid and Active, when she confirms Cancel and the capability acknowledges, then status is set to Cancelled -- ends at period end, and tier remains Paid.

**FEAT-23.SPEC-006-AC-02:** Given Nadia's plan is now Cancelled -- ends at period end, when she views her plan, then she sees a plain explanation that it stays active through the current period's end.

**FEAT-23.SPEC-006-AC-03:** Given Nadia confirms Cancel, when the request is handed to FEAT-23.SPEC-003, then the plan's status is not changed until the capability's acknowledgment arrives.

**FEAT-23.SPEC-006-AC-04:** Given Nadia's plan is Charge failed within the grace window, when she confirms Cancel and the capability acknowledges, then the cancellation is recorded, the failure record is cleared, and the grace window no longer applies.

**FEAT-23.SPEC-006-AC-05:** Given Nadia's plan is already Cancelled -- ends at period end, when she attempts to cancel again, then the attempt is rejected as stale and the screen shows the already-cancelled state.

**FEAT-23.SPEC-006-AC-06:** Given Nadia's plan lapsed between her screen loading and her tapping Cancel, when the cancellation is attempted, then it is rejected as stale, and her screen refreshes to show the Lapsed state.

**FEAT-23.SPEC-006-AC-07:** Given Nadia taps Cancel in two open sessions at the same time, when both requests reach the automation, then only the first to commit records the cancellation and the second sees the already-cancelled state.

**FEAT-23.SPEC-006-AC-08:** Given a cancellation request is awaiting the capability's acknowledgment, when Nadia taps Cancel again, then the second tap is disabled while the first is in flight.

**FEAT-23.SPEC-006-AC-09:** Given Nadia's downgrade offer is showing when she cancels instead, when the cancellation is recorded, then FEAT-23.SPEC-005 is notified and the offer is cleared immediately, not shown on the next view.

**FEAT-23.SPEC-006-AC-10:** Given the billing capability is down when Nadia confirms Cancel, when no acknowledgment arrives, then the plan is not marked Cancelled, no background relay is queued, and she sees "Billing is temporarily unavailable. Try cancelling again in a few minutes."

**FEAT-23.SPEC-006-AC-11:** Given the capability acknowledged the cancellation but the record write fails, when the retries at platform parameter: `plan-record-write-retry-interval` are under way, then Nadia's screen shows "We're finishing your cancellation -- no action needed." and clears it once the write succeeds.

**FEAT-23.SPEC-006-AC-12:** Given the record write fails on all platform parameter: `plan-record-write-retry-count` attempts, when the last attempt fails, then retries stop, the plan status is unchanged, no email fires, Nadia's session shows the "billing partner has received your cancellation" notice with the period end date, and the capability's later period-end event is applied by FEAT-23.SPEC-004.

**FEAT-23.SPEC-006-AC-13:** Given a cancellation was acknowledged but never recorded and Nadia taps Cancel again, when the capability's repeat acknowledgment arrives and the record write succeeds, then the plan shows Cancelled once, with one confirmation email and one trail entry.

**FEAT-23.SPEC-006-AC-14:** Given a cancellation is recorded, when the write commits, then it is reported to FEAT-13.SPEC-003 as an append-only trail event with Nadia as actor, and a trail-write retry never reverses the cancellation.

**FEAT-23.SPEC-006-AC-15:** Given Nadia's account deletion completes while a cancellation acknowledgment or record retry is pending, when the pending work resolves, then it is discarded with no email and no trail entry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |
