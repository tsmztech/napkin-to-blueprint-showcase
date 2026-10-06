---
document_type: spec
spec_type: automation
spec_id: FEAT-26.SPEC-003
spec_name: WhatsApp Delivery Fallback
spec_slug: whatsapp-delivery-fallback
parent_feature: FEAT-26
parent_feature_name: WhatsApp Reminders
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: WhatsApp Delivery Fallback

## Overview

**Name:** WhatsApp Delivery Fallback
**ID:** FEAT-26.SPEC-003
**Type:** Automation
**Purpose:** When a WhatsApp send fails or the recipient's number is unreachable on WhatsApp, automatically falls back to text or email per the client's existing texting consent, and flags the delivery gap the way FEAT-08 already does for a failed text.
**Parent Feature:** FEAT-26 -- WhatsApp Reminders

## Scope and Non-Goals

**In Scope:**
- Falling back to text or email when a WhatsApp send fails or is reported unreachable
- Flagging the resulting delivery gap to the Pro, consistent with FEAT-08.SPEC-009's existing pattern for a failed text

**Non-Goals:**
- Retrying the WhatsApp send itself before falling back -- the Brief's Alternate flow describes an immediate fallback to text or email on WhatsApp failure or unavailability, not a WhatsApp-channel retry; this differs deliberately from FEAT-08.SPEC-009's own single retry-before-fallback pattern for text, since WhatsApp reachability failures (a number with no WhatsApp account) are not the kind of transient failure a retry would resolve.
- Deciding whether text or email is the fallback channel -- owned by FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule), which this automation defers to exactly as every other client-directed send in the product does; this spec only triggers that decision and the send that follows it.
- Sending the fallback text or email itself, or handling a failure of that fallback send -- owned by FEAT-08.SPEC-012 (Transactional Text Messaging Capability) and FEAT-08.SPEC-013 (Transactional Email Capability); once the fallback becomes a standard text or email send, any further failure of it is governed by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback), not by this spec.
- Composing the alert content shown to the Pro -- owned by FEAT-08.SPEC-006 (Pro Attention Alert); this spec only triggers it with the WhatsApp-specific condition.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-------------|-------------|------------------|
| A WhatsApp send is reported Failed | FEAT-26.SPEC-002 (WhatsApp Send & Delivery-Status Capability) | Fires whenever the WhatsApp capability reports a Failed delivery status for a Message this feature sent | Message (type, recipient, content_summary), Booking or Pro Account reference |
| A WhatsApp send is reported rejected (number not WhatsApp-reachable) | FEAT-26.SPEC-002 (WhatsApp Send & Delivery-Status Capability) | Fires whenever the capability reports, at send time, that the recipient's number cannot receive WhatsApp messages | Message (type, recipient, content_summary), Booking or Pro Account reference |

## Processing Logic

1. Receive the Failed or send-rejected event for a WhatsApp Message from FEAT-26.SPEC-002.
2. Defer to FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) to determine the fallback channel: text if the client's texting consent is currently Granted or Re-granted and their phone number matches, otherwise email.
3. Create a new Message record on the resulting fallback channel with the same content (per the Message entity's lifecycle note: a fallback creates a second, immutable Message record rather than mutating the failed WhatsApp one), and send it through FEAT-08.SPEC-012 (text) or FEAT-08.SPEC-013 (email).
4. Trigger FEAT-08.SPEC-006 (Pro Attention Alert) to flag the delivery gap, noting the WhatsApp send that failed or was rejected and which fallback channel was used, referencing the affected Booking or Pro notification.
5. If the fallback send subsequently fails, FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) takes over as the standard retry-then-fallback path for that new text or email Message -- this automation's own responsibility ends once the fallback send has been handed to FEAT-08.SPEC-012 or FEAT-08.SPEC-013.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|----------------|------------------|-------------------|
| Fallback dispatched on text | The client's texting consent is currently active and matches their phone number | A new Message record created (channel: text, delivery_status: Queued/Sent); the original WhatsApp Message's delivery_status remains Failed as its own immutable record | Client receives the message by text; Pro sees a delivery-gap flag (WhatsApp unavailable, fell back to text) | FEAT-08.SPEC-006, FEAT-08.SPEC-011, FEAT-08.SPEC-012 |
| Fallback dispatched on email | The client's texting consent is not currently active, or their phone number does not match | A new Message record created (channel: email, delivery_status: Queued/Sent); the original WhatsApp Message's delivery_status remains Failed | Client receives the message by email; Pro sees a delivery-gap flag (WhatsApp unavailable, fell back to email) | FEAT-08.SPEC-006, FEAT-08.SPEC-011, FEAT-08.SPEC-013 |
| Fallback send itself later fails | The text or email fallback Message subsequently reports Failed | Handled by FEAT-08.SPEC-009's own retry-then-fallback path, not this automation | Talia sees FEAT-08.SPEC-009's escalated attention alert, layered on top of this automation's own flag | FEAT-08.SPEC-009 |
| Automation failure | This automation's own orchestration cannot run (e.g., the fallback dispatch itself is unavailable) | No fallback attempted | The original Failed/rejected WhatsApp status stands; this is indistinguishable from a permanently failed delivery from the Pro's perspective until detected and results in the same escalated flag once resolved | FEAT-08.SPEC-006 |

## Data Model

**Reads:** Message -- type, channel, recipient, content_summary, delivery_status (the failed or rejected WhatsApp record). Messaging Consent -- channel, state, phone_number, read indirectly through FEAT-08.SPEC-011's channel decision.
**Creates:** None directly -- the new fallback Message record is created by FEAT-08.SPEC-012 or FEAT-08.SPEC-013 as part of dispatching the fallback send this automation triggers.
**Updates:** None -- the original WhatsApp Message's delivery_status is left as Failed, set by FEAT-26.SPEC-002; this automation never mutates it, consistent with the Message entity's immutable-per-attempt lifecycle.
**Deletes:** None -- Messages are never deleted (Message entity lifecycle: immutable once sent).

## Business Rules

- XBR-15 governs the fallback channel decision this automation triggers: no text is sent without active texting consent for that client and Pro; otherwise email is used.
- XBR-17's delivery-gap discipline extends to WhatsApp: a failed or unavailable WhatsApp send is never silently dropped -- it always results in a fallback dispatch and a flag on the Pro's dashboard, recorded in the Booking's activity timeline (FEAT-16).
- This automation applies uniformly to every Notification spec whose content can be carried over WhatsApp (FEAT-08.SPEC-001, 002, 004) -- it is not specific to any one message type.
- A fallback creates a new Message record rather than mutating the failed WhatsApp one, preserving each channel attempt as its own immutable record, consistent with the append-only nature FEAT-16 relies on for dispute evidence.
- This automation's own responsibility is exactly one hop: WhatsApp failure to a single fallback dispatch on text or email. It never retries WhatsApp, and it never itself handles a second-level failure of the fallback -- that is FEAT-08.SPEC-009's territory once the fallback becomes a standard text or email send.

## Edge Cases

- **The WhatsApp send fails on a message whose content has since become stale (e.g., the booking was cancelled between the original WhatsApp attempt and this automation's fallback)** -- The fallback still sends the message as originally composed; a subsequent, distinct change notice (FEAT-08.SPEC-004) informs the client of the cancellation separately, since this automation's job is delivering the message it was given, not re-validating its content against the booking's latest state.
- **The WhatsApp capability and the text capability are both down at the same time** -- The WhatsApp Failed event still fires this automation, which attempts the text fallback; that attempt also fails and is handed to FEAT-08.SPEC-009's own retry-then-fallback (to email), so the client is never left without an attempted delivery on some channel, and the Pro's alert is not lost even during a multi-capability outage.
- **The same WhatsApp Message somehow reports Failed twice (a duplicate delivery-status event)** -- The second Failed report for a Message already in a fallback-complete state is a no-op; this automation does not dispatch a second fallback for the same original WhatsApp send.
- **Concurrent trigger firing (two different WhatsApp Messages for the same client fail at the same time)** -- Each Message's fallback runs independently; a WhatsApp confirmation failing and a WhatsApp reminder failing for the same client at the same moment each produce their own fallback dispatch and Pro alert.
- **Trigger fires while a previous run is in flight for the same Message** -- A duplicate Failed or rejected event for a Message whose fallback is already in progress is ignored; only the original triggering event drives the fallback for that Message.
- **A client's channel preference is WhatsApp but their texting consent was revoked before this automation runs** -- Step 2 correctly routes the fallback to email, since FEAT-08.SPEC-011's decision is re-evaluated fresh at fallback time, not assumed from the client's WhatsApp preference.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-26.SPEC-002 (WhatsApp Send & Delivery-Status Capability) | Triggered by (inbound) | A Failed delivery status or a send-rejected event fires this automation |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (outbound) | Decides text vs. email for the fallback dispatch |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Triggers (outbound) | Sends the fallback when text is the chosen channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | Triggers (outbound) | Sends the fallback when email is the chosen channel |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | Affects (outbound) | Governs any further failure of the fallback text or email Message this automation creates |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggers (outbound) | Every WhatsApp fallback used or unresolved failure flags the Pro |
| FEAT-08.SPEC-001, FEAT-08.SPEC-002, FEAT-08.SPEC-004 | Affects (outbound) | Any WhatsApp-channel Message these specs' content generates is subject to this automation's fallback handling |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | The WhatsApp delivery gap appears on the dashboard's attention list |
| FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every WhatsApp failure and fallback event is recorded in the append-only activity record |

## Analytics and Success Signals

- **whatsapp_delivery_failed_fallback_used** (fallback_channel: text / email) -- N/A -- no metric in success-metrics.md names WhatsApp Reminders as its Connected Feature; retained per product-features.md's own Signals field for this feature (`whatsapp_delivery_failed_fallback_used`) so the size and shape of the fallback path is observable once the feature ships, mirroring FEAT-08.SPEC-009's identical reasoning for its own `message_delivery_fallback_used` event.
- **whatsapp_fallback_dispatch_failed** () -- N/A -- no Stage 2 metric measures total WhatsApp-path delivery failure; retained as the operational signal behind XBR-17's "never silently dropped" guarantee extended to this feature's channel.

## Acceptance Criteria

**FEAT-26.SPEC-003-AC-01:** Given a WhatsApp confirmation to Riley is reported Failed, when this automation processes the event, then it defers to FEAT-08.SPEC-011 to decide the fallback channel before dispatching anything.

**FEAT-26.SPEC-003-AC-02:** Given Riley's texting consent is currently active and matches her phone number, when the fallback dispatch runs, then a new Message record is created on the text channel and sent through FEAT-08.SPEC-012.

**FEAT-26.SPEC-003-AC-03:** Given Riley's texting consent is not currently active, when the fallback dispatch runs, then a new Message record is created on the email channel and sent through FEAT-08.SPEC-013.

**FEAT-26.SPEC-003-AC-04:** Given a WhatsApp send to Riley is reported rejected because her number has no WhatsApp account, when this automation processes the event, then it falls back identically to a Failed delivery status.

**FEAT-26.SPEC-003-AC-05:** Given the fallback dispatch (text or email) succeeds, when delivery completes, then Talia still receives a delivery-gap alert (FEAT-08.SPEC-006) noting WhatsApp was unavailable and which channel the fallback used, even though Riley did receive the message.

**FEAT-26.SPEC-003-AC-06:** Given the WhatsApp send fails and the resulting fallback send also later fails, when the fallback failure is confirmed, then FEAT-08.SPEC-009's own retry-then-fallback path governs that failure, not this automation.

**FEAT-26.SPEC-003-AC-07:** Given the same WhatsApp Message reports a Failed status twice, when the second report arrives, then it is treated as a no-op and no second fallback is dispatched.

**FEAT-26.SPEC-003-AC-08:** Given the booking a failed WhatsApp message concerns is cancelled between the original failure and this automation's fallback, when the fallback sends, then it still delivers the originally composed content, and the cancellation is communicated separately via FEAT-08.SPEC-004.

**FEAT-26.SPEC-003-AC-09:** Given both the WhatsApp and text capabilities are unavailable at the same time, when the WhatsApp failure fires this automation, then the text fallback attempt also fails and is handed to FEAT-08.SPEC-009's own retry-then-fallback to email, so the client is not left without an attempted delivery.

**FEAT-26.SPEC-003-AC-10:** Given two different WhatsApp Messages for the same client fail at effectively the same time, when both are processed, then each falls back independently with its own Pro alert.

**FEAT-26.SPEC-003-AC-11:** Given a fallback text Message is created after a failed WhatsApp send, when the original WhatsApp Message record is inspected later, then it still shows delivery_status Failed as its own immutable record, distinct from the new text Message.

**FEAT-26.SPEC-003-AC-12:** Given Support views the activity record for a booking whose message fell back from WhatsApp, when Support inspects the record, then both the failed WhatsApp attempt and the successful fallback appear as separate entries, per FEAT-16's append-only record.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (WhatsApp failed, WhatsApp send-rejected) | 2 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
