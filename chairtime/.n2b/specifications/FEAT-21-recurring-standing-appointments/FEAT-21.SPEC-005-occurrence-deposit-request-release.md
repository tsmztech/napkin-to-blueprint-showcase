---
document_type: spec
spec_type: automation
spec_id: FEAT-21.SPEC-005
spec_name: Occurrence Deposit Request & Release
spec_slug: occurrence-deposit-request-release
parent_feature: FEAT-21
parent_feature_name: Recurring/Standing Appointments
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Automation Spec: Occurrence Deposit Request & Release

## Overview

**Name:** Occurrence Deposit Request & Release
**ID:** FEAT-21.SPEC-005
**Type:** Automation
**Purpose:** Sends each generated occurrence's own fresh deposit link about a week before it, and releases the occurrence if the deposit is never paid by its cancellation cut-off, without disturbing the rest of the series.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- Sending the deposit-request link for each occurrence's Booking a fixed number of days before its appointment
- Monitoring an occurrence's Pending Payment Booking through to its cancellation cut-off
- Releasing (expiring) an occurrence whose deposit remains unpaid by that cut-off
- Handing off both the request and the release moments to FEAT-21.SPEC-009 for client- and Pro-facing notification

**Non-Goals:**
- Capturing the deposit payment itself, or authorizing the client's card -- owned by FEAT-07 (Deposit Payment at Booking); this spec only decides when to send the link and when to give up waiting for it.
- Composing or delivering the deposit-request or release notification content -- owned by FEAT-21.SPEC-009 (Occurrence Deposit Lifecycle Notification); this spec only triggers it at the right moments.
- Generating the occurrence's Booking in the first place -- owned by FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling), which hands the Booking to this spec once created.
- Retaining or reusing a client's card across occurrences -- excluded per this Brief's Non-Goals and scope-boundaries.md SC-11/SC-13: each occurrence's deposit is paid fresh through its own link, exactly as any other deposit payment.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An occurrence's Booking is generated | FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Fires once, immediately after a normally generated or replacement-time-accepted occurrence's Booking is created, to begin this spec's monitoring | Booking (start_time, deposit_amount, state), Recurring Series reference |
| An occurrence approaches its deposit-request point | Schedule-based (evaluated against each monitored occurrence's start_time) | Fires when the current time reaches the occurrence's start_time minus platform parameter: `recurring-occurrence-deposit-lead-days`, and no deposit-request link has yet been sent for it, and the Booking is still Pending Payment | Booking (service, start_time, deposit_amount) |
| An occurrence reaches its cancellation cut-off unpaid | Schedule-based (evaluated against each monitored occurrence's cancellation cut-off) | Fires when the current time reaches the occurrence's start_time minus the Pro's currently active Cancellation Policy window_hours (FEAT-09, XBR-08), and the Booking is still Pending Payment (deposit never captured) | Booking (service, start_time, state) |

## Processing Logic

1. On receiving a newly generated occurrence's Booking from FEAT-21.SPEC-004, begin monitoring it for its deposit-request point and its cancellation cut-off.
2. When the current time reaches the occurrence's start_time minus platform parameter: `recurring-occurrence-deposit-lead-days`, and the Booking remains Pending Payment with no deposit-request link yet sent, initiate the deposit-request send: hand off to FEAT-07's ordinary deposit-capture mechanism, scoped to this one occurrence's own fresh payment session (XBR-05 -- no card is retained between occurrences).
3. Record that the deposit-request link has been sent for this occurrence, so it is never sent a second time.
4. Trigger FEAT-21.SPEC-009 (Occurrence Deposit Lifecycle Notification), deposit-request variant, carrying the link.
5. Continue monitoring the occurrence. If the client completes the deposit payment at any point before the cancellation cut-off, FEAT-07 transitions the Booking to Confirmed; this spec detects the Confirmed state and stops monitoring that occurrence -- no release action is taken.
6. If the current time reaches the occurrence's cancellation cut-off (start_time minus the Pro's currently active Cancellation Policy window_hours) with the Booking still Pending Payment, release the occurrence: transition its Booking to Expired.
7. The freed time reappears in the availability engine's next slot-list computation, exactly as any other expired, unpaid booking.
8. Trigger FEAT-21.SPEC-009 (Occurrence Deposit Lifecycle Notification), release variant, to both the Client and the Pro.
9. Take no further action on this occurrence -- the series itself is untouched, and its next due occurrence continues to generate normally on its own schedule (FEAT-21.SPEC-004).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Deposit request sent | Current time reaches the deposit-request point and no link has been sent yet | Deposit-request-sent flag recorded on the Booking | Client receives the deposit-request notification (FEAT-21.SPEC-009); occurrence shows "Awaiting deposit" on FEAT-21.SPEC-002 | FEAT-21.SPEC-002, FEAT-21.SPEC-009 |
| Deposit paid before cut-off | Client completes payment via FEAT-07 while the Booking is still Pending Payment | Booking transitioned to Confirmed (by FEAT-07); monitoring for this occurrence stops | Client and Pro see the occurrence Confirmed; no release action occurs | FEAT-07, FEAT-21.SPEC-002 |
| Occurrence released unpaid | Cancellation cut-off is reached with the Booking still Pending Payment | Booking transitioned to Expired | Client and Pro both receive the release notification (FEAT-21.SPEC-009); occurrence disappears from Upcoming on FEAT-21.SPEC-002 | FEAT-21.SPEC-002, FEAT-21.SPEC-009 |
| Deposit-request send failure | The link cannot be sent at the scheduled moment (e.g., a processing error) | None -- the deposit-request-sent flag is not set | No client-facing feedback beyond the ordinary message-delivery retry/fallback FEAT-08 already provides; the occurrence continues toward its cancellation cut-off regardless | FEAT-08 (Automated Booking Messaging) |

## Data Model

**Reads:** Booking -- start_time, state, deposit_amount; Cancellation Policy -- the Pro's currently active version's window_hours (FEAT-09).
**Creates:** None.
**Updates:** Booking -- deposit-request-sent flag (set once, by this spec); state transitioned to Expired on release (this spec is the actor; FEAT-07 is the actor for the Confirmed transition).
**Deletes:** None.

## Business Rules

- The deposit-request lead time is fixed at platform parameter: `recurring-occurrence-deposit-lead-days`, matching this Brief's stated example of about a week before the occurrence.
- The cancellation cut-off this spec monitors is always the Pro's currently active Cancellation Policy window_hours (FEAT-09, XBR-08) measured back from the occurrence's start_time -- the same window that governs any booking's free-cancellation deadline, never a value this spec defines independently.
- XBR-05: each occurrence's deposit is computed once from the Service's rule and captured fresh, with no card retained between occurrences -- this spec never reuses a prior occurrence's payment method or authorization.
- The first committed action wins a race between a completing deposit payment and a reached cancellation cut-off (XBR-01, mirroring FEAT-03.SPEC-007's own race rule): if payment completes before this automation processes the release, the Booking is Confirmed, not Expired.
- Releasing an occurrence never affects its series: the series remains Active and its next due occurrence continues generating on its own schedule (FEAT-21.SPEC-004), per this Brief's Side-Effect Inventory.

## Edge Cases

- **The occurrence's appointment is scheduled sooner than platform parameter: `recurring-occurrence-deposit-lead-days` away at generation time** -- The deposit-request send fires immediately upon generation instead of waiting for the lead-time point, since that point has already passed; the occurrence is still monitored to its cancellation cut-off as usual.
- **The Pro changes her active Cancellation Policy window_hours while an occurrence's deposit is still pending** -- Per XBR-08, an occurrence's own policy_version is set when the client's deposit payment is acknowledged and captured, not at generation time; until that happens, this spec recalculates the cancellation cut-off against the Pro's currently active version each time it evaluates the occurrence, so a mid-flight policy change immediately reshapes an unpaid occurrence's release timing.
- **Deposit payment completes in the same instant the cancellation cut-off elapses** -- The completed-payment transition (Confirmed) takes precedence; this spec detects the already-Confirmed state before processing the release and takes no further action, mirroring FEAT-03.SPEC-007's own race resolution.
- **An occurrence is cancelled by the client or the Pro (FEAT-21.SPEC-006) before either the deposit-request point or the cancellation cut-off is reached** -- Monitoring stops immediately; no deposit-request link is sent and no release notification fires for a cancelled occurrence.
- **Concurrent trigger firing (two occurrences from different series reach their deposit-request point at effectively the same time)** -- Each occurrence's monitoring runs independently against its own start_time and deposit-request-sent flag; neither affects the other.
- **Trigger fires while a previous evaluation is still in flight for the same occurrence** -- The deposit-request-sent flag guards against a duplicate send even if two evaluations for the same occurrence overlap; the release action likewise checks the Booking's current state immediately before transitioning it, preventing a double-release.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Triggered by (inbound) | Hands off each newly generated occurrence's Booking for deposit-lifecycle monitoring |
| FEAT-09 (Cancellation & No-Show Policy Engine) | References (outbound) | Supplies the currently active window_hours used to compute each occurrence's cancellation cut-off |
| FEAT-07 (Deposit Payment at Booking) | Triggers (outbound) / Triggered by (inbound, on payment completion) | Performs the actual deposit-request send and payment capture; its completion stops this spec's monitoring for that occurrence |
| FEAT-21.SPEC-009 (Occurrence Deposit Lifecycle Notification) | Triggers (outbound) | Both the deposit-request send and the release fire this notification, in its two variants |
| FEAT-21.SPEC-002 (My Recurring Series) | Affects (outbound) | An occurrence's "Awaiting deposit" status, and its removal on release, are reflected here |
| FEAT-21.SPEC-010 (Pro Recurring Series Management) | Affects (outbound) | An occurrence's "Awaiting deposit" status, and its removal on release, are reflected on the Pro's screen too |
| FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules) | References (inbound) | A cancelled occurrence stops this spec's monitoring immediately |

## Analytics and Success Signals

- **recurring_occurrence_deposit_requested** (series reference) -- supports success-metrics.md: "Deposit Capture Rate" (each occurrence's deposit request is an ordinary deposit-collection attempt this metric already tracks end to end)
- **recurring_occurrence_deposit_released** (series reference, days_unpaid) -- supports success-metrics.md: "Automatic Refund Correctness" (a release with no payment ever captured has nothing to refund, but the event is the direct evidence that an unpaid occurrence never silently lingers as a booking neither party can act on)
- **recurring_occurrence_deposit_request_send_failed** (series reference, reason category) -- N/A -- no Stage 2 metric measures deposit-request send failures directly for this feature; retained so a silently-unrequested occurrence deposit is observable rather than invisible.

## Acceptance Criteria

**FEAT-21.SPEC-005-AC-01:** Given an occurrence's Booking is generated 10 weeks out, when the current time reaches platform parameter: `recurring-occurrence-deposit-lead-days` before its start_time, then the deposit-request link is sent and FEAT-21.SPEC-009's request variant fires.

**FEAT-21.SPEC-005-AC-02:** Given an occurrence's appointment is sooner than platform parameter: `recurring-occurrence-deposit-lead-days` away at generation time, when the occurrence is generated, then the deposit-request send fires immediately.

**FEAT-21.SPEC-005-AC-03:** Given Riley receives an occurrence's deposit-request link, when she completes payment before the cancellation cut-off, then the Booking is transitioned to Confirmed and no release action ever occurs for that occurrence.

**FEAT-21.SPEC-005-AC-04:** Given an occurrence's Booking remains Pending Payment when the current time reaches its cancellation cut-off (start_time minus the Pro's active window_hours), when the release evaluation runs, then the Booking is transitioned to Expired and FEAT-21.SPEC-009's release variant fires to both Riley and Talia.

**FEAT-21.SPEC-005-AC-05:** Given an occurrence is released unpaid, when its series is next evaluated, then the series remains Active and its next due occurrence still generates on schedule.

**FEAT-21.SPEC-005-AC-06:** Given Talia changes her active Cancellation Policy window_hours while an occurrence's deposit is still pending, when this spec next evaluates that occurrence, then the cancellation cut-off is recalculated against the currently active window_hours.

**FEAT-21.SPEC-005-AC-07:** Given an occurrence's deposit payment completes at the same instant its cancellation cut-off is reached, when both are evaluated, then the completed payment takes precedence and the Booking is Confirmed, not Expired.

**FEAT-21.SPEC-005-AC-08:** Given an occurrence is cancelled before its deposit-request point is reached, when the cancellation completes, then no deposit-request link is ever sent for it.

**FEAT-21.SPEC-005-AC-09:** Given an occurrence is cancelled after its deposit-request link was already sent but before the cancellation cut-off, when the cancellation completes, then monitoring stops and no release notification fires.

**FEAT-21.SPEC-005-AC-10:** Given the deposit-request send fails due to a processing error, when the failure occurs, then no client-facing error appears from this spec, and the occurrence continues to be monitored toward its cancellation cut-off exactly as if the send had succeeded.

**FEAT-21.SPEC-005-AC-11:** Given two occurrences from different series each reach their deposit-request point at effectively the same time, when both are evaluated, then each is processed independently with no interference.

**FEAT-21.SPEC-005-AC-12:** Given an occurrence's deposit-request-sent flag is already set, when a second evaluation for that same occurrence runs before the first completes, then no duplicate deposit-request link is sent.

**FEAT-21.SPEC-005-AC-13:** Given a released occurrence's slot becomes free, when the availability engine next computes its slot list, then the freed time appears as open.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 | 3 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
