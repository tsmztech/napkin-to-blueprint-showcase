---
document_type: spec
spec_type: automation
spec_id: FEAT-14.SPEC-003
spec_name: Delivery Status Tracking & Retry
spec_slug: delivery-status-tracking-retry
parent_feature: FEAT-14
parent_feature_name: Notifications (Email)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Delivery Status Tracking & Retry

## Overview

**Name:** Delivery Status Tracking & Retry
**ID:** FEAT-14.SPEC-003
**Type:** Automation
**Purpose:** Records each notification's delivery outcome, retries a failed send automatically a limited number of times, then hands off to the delivery-failure warning once retries are exhausted or the failure is permanent.
**Parent Feature:** FEAT-14 -- Notifications (Email)

## Scope and Non-Goals

**In Scope:**
- Recording delivery, bounce, and failure outcomes against the Notification record
- Retrying a transient send failure automatically, within a bounded count and window
- Distinguishing a permanent failure (bounce) from a transient one for retry purposes
- Handing off to the delivery-failure warning once retries are exhausted or a bounce is final

**Non-Goals:**
- Creating the Notification record or resolving recipient entitlement -- owned by FEAT-14.SPEC-002 and FEAT-14.SPEC-004; this spec only updates records that already exist.
- The actual transmission of an email, and the raw delivered/bounced/failed reporting itself -- owned by FEAT-14.SPEC-001; this spec consumes that spec's inbound events.
- The content, channel, and delivery rules of the delivery-failure warning itself -- owned by FEAT-14.SPEC-006; this spec only triggers it once retries are exhausted.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Notification delivery outcome reported | FEAT-14.SPEC-001 (Transactional Email Delivery) | Fires whenever the delivery capability reports Delivered, Bounced, or Failed for a Queued or previously-Failed Notification | Notification reference, outcome, bounce/failure reason category if provided |
| Retry interval elapsed for a Failed notification | System (schedule-based, internal to this automation) | Fires when a Failed notification's next scheduled retry attempt is due, within platform parameter: `transactional-email-retry-window` of the first attempt | Notification reference, retry attempt count so far |

## Processing Logic

1. Receive the delivery outcome reported by FEAT-14.SPEC-001 for a Notification.
2. If the outcome is Delivered: set Notification.delivery_status to Delivered and stop -- no further action.
3. If the outcome is Bounced (a permanent failure -- the address itself is invalid): set Notification.delivery_status to Bounced. A bounce is never retried, since retrying an identical, invalid address would produce the same outcome again and only delay the warning Nadia needs; proceed directly to Step 6.
4. If the outcome is Failed (a transient failure): set Notification.delivery_status to Failed. If the Notification's retry count is below platform parameter: `transactional-email-retry-count`, schedule the next retry attempt (re-invoking FEAT-14.SPEC-001) within platform parameter: `transactional-email-retry-window` of the first attempt, and increment the retry count.
5. When a scheduled retry's outcome is reported, return to Step 2 (Delivered) or repeat Step 3/4 as applicable, using the current retry count.
6. Once retries are exhausted (the retry count has reached platform parameter: `transactional-email-retry-count` with no Delivered outcome) or a Bounce occurred, finalize Notification.delivery_status as Bounced or Failed and hand off to FEAT-14.SPEC-006 so Nadia is warned on the affected project.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Delivered | The capability confirms delivery, on the first attempt or any retry | Notification.delivery_status set to Delivered | None | -- |
| Bounced -- immediate warning | The capability reports a permanent bounce | Notification.delivery_status set to Bounced | Delivery warning appears on the affected project for Nadia | FEAT-14.SPEC-006 |
| Failed -- retry scheduled | A transient failure occurs and the retry count is below platform parameter: `transactional-email-retry-count` | Notification.delivery_status set to Failed; retry count incremented; a retry is scheduled | None yet -- retrying is not surfaced to Nadia unless and until it is exhausted | -- |
| Failed -- retries exhausted | A transient failure recurs until the retry count reaches platform parameter: `transactional-email-retry-count` with no Delivered outcome | Notification.delivery_status finalized as Failed | Delivery warning appears on the affected project for Nadia | FEAT-14.SPEC-006 |
| Automation failure (tracking itself cannot process an outcome report, e.g., the Notification reference cannot be resolved) | A processing error occurs within this automation, distinct from the delivery outcome itself | No Notification status change is applied; the outcome report is not lost -- it is re-processed on the automation's next run | None immediately; non-blocking to any triggering screen, since this automation never blocks a user action | -- |

## Data Model

**Reads:** Notification -- delivery_status, notification_type, recipient, sent_at, retry count; Project -- the affected project a delivery warning is attached to, read to hand off to FEAT-14.SPEC-006.
**Creates:** None -- this automation never creates a Notification; that is owned by FEAT-14.SPEC-002.
**Updates:** Notification -- delivery_status (Queued -> Sent -> Delivered, or Queued -> Sent -> Failed/Bounced -> (retry) -> Delivered or Failed).
**Deletes:** None.

## Business Rules

- Delivery failures are surfaced to the freelancer within minutes rather than silently lost (ASMP-26) -- retries proceed promptly within platform parameter: `transactional-email-retry-window`, not delayed for days.
- A Bounce (permanent) is never retried, since a retry against an invalid address would reproduce the identical outcome and only delay the warning Nadia needs.
- A transient Failed outcome retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` -- the same retry contract every triggering feature's own Notification spec already cites when describing this capability's behavior (e.g., FEAT-02.SPEC-011, FEAT-05.SPEC-008, FEAT-09.SPEC-010).
- Delivery status transitions move toward a terminal state (Delivered, or Failed/Bounced-after-retries) only; a Notification never regresses from Delivered back to an earlier state.
- At most one delivery-failure warning is generated per Notification, regardless of how many duplicate outcome reports arrive afterward -- deduplicated here before FEAT-14.SPEC-006 is ever invoked.

## Edge Cases

- **Concurrent trigger firing -- two outcome reports for the same Notification arrive at effectively the same time (e.g., a retry's Failed report and a stale earlier report)** -- The automation applies the most recent true outcome by the event's own occurrence time (per FEAT-14.SPEC-001), so a later Delivered is never overwritten by an earlier, stale Failed report.
- **Trigger fires while a previous run is in flight -- a retry attempt's outcome report arrives while this automation is still applying the prior attempt's outcome for the same Notification** -- Processing for a single Notification is serialized: the second report waits for the first to finish applying its status change before being applied, so delivery_status transitions never interleave inconsistently for one Notification. Runs for different Notifications proceed independently and do not queue behind each other.
- **A Notification reaches its final retry count boundary at the same moment that final retry succeeds** -- The successful Delivered outcome wins over the "retries exhausted" determination; a late success is always honored, since the entire purpose of retrying is to catch it.
- **The affected project is archived between a Failed outcome and the retries-exhausted moment** -- The delivery warning still surfaces per FEAT-14.SPEC-006, since archiving removes a project from active view without erasing its record, and a bouncing contact address may still need correcting.
- **The underlying account is deleted (FEAT-24) while a Notification is mid-retry** -- Retry processing for that Notification stops; the Notification record itself is removed as part of account deletion, consistent with FEAT-14.SPEC-001's edge-case handling for deleted accounts.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-001 (Transactional Email Delivery) | Triggered by (inbound) | Every delivery outcome this spec processes originates from this integration's inbound events, and retry attempts are re-issued back through it |
| FEAT-14.SPEC-002 (Notification Composition & Dispatch) | References (inbound) | Reads the Notification records this automation created |
| FEAT-14.SPEC-006 (Delivery Failure Warning to Freelancer) | Triggers (outbound) | Fires once retries are exhausted or a bounce is final |

## Analytics and Success Signals

- **notification_retry_attempted** (notification_type, attempt_number) -- supports success-metrics.md: "Notification Delivery Reliability"
- **notification_delivery_succeeded_after_retry** (notification_type, attempt_number) -- supports success-metrics.md: "Notification Delivery Reliability"
- **notification_delivery_failed_final** (notification_type, reason: bounced / retries_exhausted) -- supports success-metrics.md: "Notification Delivery Reliability"

## Acceptance Criteria

**FEAT-14.SPEC-003-AC-01:** Given a Notification is Queued for Owen, when FEAT-14.SPEC-001 reports Delivered on the first attempt, then Notification.delivery_status is set to Delivered and no retry occurs.

**FEAT-14.SPEC-003-AC-02:** Given a Notification's send is reported Bounced, when this automation processes the outcome, then Notification.delivery_status is set to Bounced immediately and no retry is scheduled.

**FEAT-14.SPEC-003-AC-03:** Given a Notification's send is reported Failed for a transient reason, when this automation processes the outcome and the retry count is below platform parameter: `transactional-email-retry-count`, then Notification.delivery_status is set to Failed and a retry is scheduled.

**FEAT-14.SPEC-003-AC-04:** Given a Notification has been retried and reaches platform parameter: `transactional-email-retry-count` attempts with no Delivered outcome, when the final retry's outcome is processed, then Notification.delivery_status is finalized as Failed and FEAT-14.SPEC-006 is triggered.

**FEAT-14.SPEC-003-AC-05:** Given a bounced Notification, when this automation finalizes it, then FEAT-14.SPEC-006 is triggered immediately without waiting for any retry count.

**FEAT-14.SPEC-003-AC-06:** Given a Notification succeeds on its second retry attempt, when the Delivered outcome is processed, then Notification.delivery_status is set to Delivered and no delivery warning is ever generated.

**FEAT-14.SPEC-003-AC-07:** Given two outcome reports for the same Notification arrive at effectively the same time, when this automation processes them, then the outcome that occurred more recently (by its own event time) is the one reflected, regardless of arrival order.

**FEAT-14.SPEC-003-AC-08:** Given a retry outcome report arrives while this automation is still applying the prior report's status change for the same Notification, when both are processed, then they are applied in sequence and the Notification never shows an inconsistent intermediate state.

**FEAT-14.SPEC-003-AC-09:** Given a Notification's final retry succeeds at the exact moment its retry count reaches the limit, when both conditions are evaluated together, then the Delivered outcome takes precedence over marking the notification Failed.

**FEAT-14.SPEC-003-AC-10:** Given the affected project is archived while a Notification is retrying, when retries are later exhausted, then the delivery warning still appears per FEAT-14.SPEC-006.

**FEAT-14.SPEC-003-AC-11:** Given the freelancer's account is deleted while a Notification is mid-retry, when the deletion completes, then retry processing for that Notification stops and the record is removed with the account.

**FEAT-14.SPEC-003-AC-12:** Given this automation cannot resolve a Notification reference due to a processing error, when the outcome report cannot be applied, then no status change is made, no user is blocked, and the report is re-processed on the automation's next run.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (outcome reported, retry interval elapsed) | 2 |
| Outcome Paths | 5 (delivered, bounced-immediate, failed-retry-scheduled, failed-exhausted, automation failure) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 5 | 5 |
