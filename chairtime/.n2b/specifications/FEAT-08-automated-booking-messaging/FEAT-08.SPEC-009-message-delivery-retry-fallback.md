---
document_type: spec
spec_type: automation
spec_id: FEAT-08.SPEC-009
spec_name: Message Delivery Retry & Fallback
spec_slug: message-delivery-retry-fallback
parent_feature: FEAT-08
parent_feature_name: Automated Booking Messaging
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Message Delivery Retry & Fallback

## Overview

**Name:** Message Delivery Retry & Fallback
**ID:** FEAT-08.SPEC-009
**Type:** Automation
**Purpose:** When a text fails to deliver, retries it once and then falls back to email, flagging the delivery gap on the Pro's dashboard so no message this feature sends is ever silently dropped.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- Retry and fallback handling for every Message this feature's Notification specs create (confirmation, reminder, change notice, Pro activity notification, Pro attention alert)
- Flagging an unresolved delivery gap to the Pro

**Non-Goals:**
- Deciding the initial channel (text vs. email) for a send -- owned by FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule); this spec only handles what happens after a chosen channel's send attempt fails.
- The text and email sends themselves -- owned by FEAT-08.SPEC-012 and FEAT-08.SPEC-013 (the Integration specs); this spec orchestrates retry/fallback around their reported outcomes, not the sends themselves.
- Retrying an email delivery failure with a further fallback channel -- product-features.md and the Brief's Alternate flow describe only a text-then-email fallback chain; there is no channel beyond email to fall back to, so an email failure is handled as a final failure per Outcome Definitions below, not a retry loop.
- Composing the delivery-gap alert's content -- owned by FEAT-08.SPEC-006 (Pro Attention Alert); this spec only triggers it.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A text send is reported Failed | FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Fires whenever the text capability reports a Failed delivery status for a Message this feature sent | Message (type, recipient, content_summary), Booking or Pro Account reference |
| An email send (as the fallback) is reported Failed | FEAT-08.SPEC-013 (Transactional Email Capability) | Fires whenever the email fallback itself is reported Failed, marking the end of the retry/fallback chain | Message (type, recipient, content_summary) |

## Processing Logic

1. Receive the Failed delivery-status event for a text Message.
2. Retry the same text send once, through FEAT-08.SPEC-012, up to platform parameter: `message-delivery-retry-count` times (BRIEF.md's stated behavior: retries once).
3. If the retried text succeeds (reported Sent or Delivered), the delivery is complete; no fallback or alert is needed.
4. If the retried text also fails, create a new Message record on the email channel with the same content (per the Message entity's lifecycle note: a fallback creates a second, immutable Message record rather than mutating the failed one) and send it through FEAT-08.SPEC-013.
5. If the email fallback succeeds, the delivery is complete on the fallback channel; flag the gap to the Pro (Step 6) regardless, since a text-to-email fallback is itself information the Pro should see, even though the client did receive the message.
6. Trigger FEAT-08.SPEC-006 (Pro Attention Alert) to flag the delivery gap, referencing the affected Booking or Pro notification and the fact that a fallback to email was used.
7. If the email fallback also fails, this is a final, unresolved delivery failure: trigger FEAT-08.SPEC-006 with the escalated condition (no channel succeeded), so the Pro knows the client may not have received the message at all.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Retry succeeds | The retried text send is reported Sent/Delivered | Message.delivery_status updated to Sent/Delivered | None -- delivery completed on the original channel, no gap to flag | FEAT-08.SPEC-012 |
| Fallback succeeds | The retried text fails; the email fallback succeeds | A new Message record created (channel: email, delivery_status: Sent/Delivered); the original text Message's delivery_status remains Failed as its own immutable record | Client receives the message by email; Pro sees a delivery-gap flag (fallback used) | FEAT-08.SPEC-006, FEAT-08.SPEC-013 |
| Both channels fail | The retried text fails and the email fallback also fails | Both Message records (text, email) show delivery_status Failed | No message reaches the client on this attempt; Pro sees an escalated delivery-gap flag | FEAT-08.SPEC-006 |
| No action needed | The original text send succeeds on first attempt | None (this automation is not triggered) | N/A | -- |
| Automation failure | The retry/fallback orchestration itself cannot run (e.g., the automation's own processing is unavailable) | No retry or fallback attempted | The original Failed status stands; this is itself indistinguishable from "both channels fail" from the Pro's perspective, and results in the same escalated flag once detected | FEAT-08.SPEC-006 |

## Data Model

**Reads:** Message -- type, channel, recipient, content_summary, delivery_status.
**Creates:** Message -- a new record on the fallback (email) channel when the text retry fails, per the Message entity's immutable-per-attempt lifecycle.
**Updates:** Message -- delivery_status on the original (text) Message record, reflecting the retry's outcome.
**Deletes:** None -- Messages are never deleted (Message entity lifecycle: immutable once sent).

## Business Rules

- XBR-17 governs this spec entirely: a failed text is retried once, then falls back to email, and the delivery gap is flagged on the Pro's dashboard and recorded in the Booking timeline (FEAT-16) -- never silently dropped.
- This automation applies uniformly to every Notification spec in this feature (FEAT-08.SPEC-001, 002, 004, 005, 006) -- it is not specific to client-directed messages; a Pro notification's own text failing follows the identical retry-then-fallback path.
- The retry count is fixed at platform parameter: `message-delivery-retry-count` -- this is not a per-message or per-client configurable value.
- A fallback creates a new Message record rather than mutating the failed one, preserving each channel attempt as its own immutable record, consistent with the append-only nature FEAT-16 relies on for dispute evidence.

## Edge Cases

- **The retry succeeds on a message whose content has since become stale (e.g., the booking was cancelled between the original failed attempt and the retry)** -- The retry still sends the message as originally composed; a subsequent, distinct change notice (FEAT-08.SPEC-004) informs the client of the cancellation separately, since this automation's job is delivery of the message it was given, not re-validating its content against the booking's latest state.
- **Both the text and email capabilities are down at the same time** -- Both attempts fail; the escalated "both channels fail" outcome fires, and the Pro's alert is retried on its own channels per FEAT-08.SPEC-006's own delivery rules, so the escalation itself is not lost even during a capability-wide outage.
- **The same Message somehow reports Failed twice (a duplicate delivery-status event)** -- The second Failed report for a Message already in a fallback or retry-complete state is a no-op; this automation does not retry or fall back a second time for the same original send.
- **Concurrent trigger firing (two different Messages for the same client fail delivery at the same time)** -- Each Message's retry/fallback runs independently; a text confirmation failing and a text reminder failing for the same client at the same moment each produce their own retry, fallback, and Pro alert.
- **Trigger fires while a previous run is in flight for the same Message** -- A duplicate Failed event for a Message whose retry is already in progress is ignored; only the original triggering event drives the retry-then-fallback sequence for that Message.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Triggered by (inbound) | A Failed text delivery-status event fires this automation |
| FEAT-08.SPEC-013 (Transactional Email Capability) | Triggered by (inbound); Triggers (outbound) | A Failed email (fallback) event also fires this automation; a successful retry-triggered fallback send uses this capability |
| FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-08.SPEC-004, FEAT-08.SPEC-005, FEAT-08.SPEC-006 | Affects (outbound) | Any Message these specs create is subject to this automation's retry/fallback handling |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggers (outbound) | Every fallback used or unresolved failure flags the Pro |
| FEAT-12.SPEC-005 (Attention Flag Aggregation) -- within FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | The delivery gap appears on the dashboard's attention list |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every retry and fallback event is recorded in the append-only activity record |

## Analytics and Success Signals

- **message_delivery_retry_attempted** (original_channel: text) -- N/A -- no Stage 2 metric directly measures retry frequency; retained to make the reliability commitment (XBR-17) operationally observable.
- **message_delivery_fallback_used** (original_channel: text; fallback_channel: email) -- N/A -- no Stage 2 metric names messaging delivery reliability directly; retained because a silent delivery gap would otherwise undermine "Reminder Response Rate" and "Booking Completion Speed" without either metric being able to detect why.
- **message_delivery_failed_both_channels** () -- N/A -- no Stage 2 metric measures total delivery failure; retained as the operational signal behind XBR-17's "never silently dropped" guarantee.

## Acceptance Criteria

**FEAT-08.SPEC-009-AC-01:** Given a confirmation text to Riley fails delivery, when this automation retries it once, then the retry attempt is made through FEAT-08.SPEC-012 before any fallback occurs.

**FEAT-08.SPEC-009-AC-02:** Given the retried text succeeds, when the retry's delivery status reports Delivered, then no fallback is used and no Pro alert fires.

**FEAT-08.SPEC-009-AC-03:** Given the retried text also fails, when the fallback step runs, then a new Message record is created on the email channel and sent through FEAT-08.SPEC-013, and Riley receives the message by email.

**FEAT-08.SPEC-009-AC-04:** Given the email fallback succeeds after a failed text and retry, when delivery completes, then Talia still receives a delivery-gap alert (FEAT-08.SPEC-006) noting the fallback was used, even though Riley did receive the message.

**FEAT-08.SPEC-009-AC-05:** Given both the retried text and the email fallback fail, when both failures are confirmed, then Talia receives an escalated attention alert stating no channel succeeded.

**FEAT-08.SPEC-009-AC-06:** Given a Pro notification (not a client message) fails delivery by text, when this automation processes it, then the identical retry-then-fallback path applies as for a client-directed message.

**FEAT-08.SPEC-009-AC-07:** Given the same failed Message reports a Failed status twice, when the second report arrives, then it is treated as a no-op and no second retry or fallback is attempted.

**FEAT-08.SPEC-009-AC-08:** Given the booking a failed message concerns is cancelled between the original failure and the retry, when the retry sends, then it still delivers the originally composed content, and the cancellation is communicated separately via FEAT-08.SPEC-004.

**FEAT-08.SPEC-009-AC-09:** Given both the text and email capabilities are unavailable at the same time, when both attempts fail, then the Pro's escalated alert is still delivered through its own retry rules (FEAT-08.SPEC-006), not lost to the same outage.

**FEAT-08.SPEC-009-AC-10:** Given two different Messages for the same client fail delivery at effectively the same time, when both are processed, then each retries and falls back independently.

**FEAT-08.SPEC-009-AC-11:** Given a fallback email Message is created after a failed text, when the original text Message record is inspected later, then it still shows delivery_status Failed as its own immutable record, distinct from the new email Message.

**FEAT-08.SPEC-009-AC-12:** Given Support views the activity record for a booking whose message fell back to email, when Support inspects the record, then both the failed text attempt and the successful email fallback appear as separate entries, per FEAT-16's append-only record.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (text failed, email fallback failed) | 2 |
| Outcome Paths | 5 | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
