---
document_type: spec
spec_type: integration
spec_id: FEAT-26.SPEC-002
spec_name: WhatsApp Send & Delivery-Status Capability
spec_slug: whatsapp-send-delivery-status-capability
parent_feature: FEAT-26
parent_feature_name: WhatsApp Reminders
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Integration Spec: WhatsApp Send & Delivery-Status Capability

## Overview

**Name:** WhatsApp Send & Delivery-Status Capability
**ID:** FEAT-26.SPEC-002
**Type:** Integration
**Purpose:** Sends FEAT-08's confirmation, reminder and change-notice content over WhatsApp for clients whose channel is eligible, and reports back each message's delivery status.
**Parent Feature:** FEAT-26 -- WhatsApp Reminders

## Scope and Non-Goals

**In Scope:**
- Sending a WhatsApp message carrying content already composed by FEAT-08's Notification specs (booking confirmation, appointment reminder, booking change & refund notice), once FEAT-26.SPEC-004 has found the send eligible
- Receiving and reporting back delivery status (Queued, Sent, Delivered, Failed) for every WhatsApp message sent
- Reporting when a send is rejected because the recipient's number is not reachable on WhatsApp
- Degradation behavior when the capability is slow, down, or rejects a send
- Disclosure of what client data is shared with this capability

**Non-Goals:**
- Choosing the WhatsApp vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for a specific vendor.
- Deciding whether a given send is eligible for WhatsApp at all -- owned by FEAT-26.SPEC-004 (WhatsApp Channel Eligibility & Consent Rule); this spec sends whatever it is given once that decision has already been made.
- Composing the confirmation, reminder or change-notice content -- owned by FEAT-08.SPEC-001, FEAT-08.SPEC-002 and FEAT-08.SPEC-004; this spec dispatches that already-composed content over an additional channel.
- Retrying a failed or unavailable WhatsApp send, or falling back to text or email -- owned by FEAT-26.SPEC-003 (WhatsApp Delivery Fallback), which consumes this spec's Failed status and send-rejected event as its own triggers.
- Standard text and email messaging -- owned by FEAT-08.SPEC-012 and FEAT-08.SPEC-013; this spec covers WhatsApp only, and the fallback path FEAT-26.SPEC-003 uses routes back through those two specs unchanged.

## Capability Category

**Category:** Transactional WhatsApp messaging
**Dependency Source:** ASMP-32 -- "Transactional text-messaging capability, with email as a fallback channel" (assumptions-constraints.md, Dependencies), extended to WhatsApp per BRIEF.md's Ecosystem & Integrations: "WhatsApp is a nice-to-have later, not v1"
**External Touchpoint:** "Transactional WhatsApp messaging -- optional client channel for confirmations, reminders and change notices, from Later" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-26, FEAT-08, FEAT-14)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|-------------------------|------------------------------|
| Riley receives her booking confirmation over WhatsApp instead of text, when she has chosen WhatsApp and it is eligible | Opt for WhatsApp as the delivery channel for confirmations and reminders | FEAT-08.SPEC-001 (Booking Confirmation Message) |
| Riley receives her pre-appointment reminder over WhatsApp | Opt for WhatsApp as the delivery channel for confirmations and reminders | FEAT-08.SPEC-002 (Appointment Reminder Message) |
| Riley receives a cancellation, reschedule or refund notice over WhatsApp | Opt for WhatsApp as the delivery channel for confirmations and reminders | FEAT-08.SPEC-004 (Booking Change & Refund Notice) |
| Talia sees the WhatsApp channel and delivery status on any message sent to a WhatsApp-preferring client, same as any other channel | -- (inherits FEAT-08's existing delivery-status visibility) | FEAT-12 (Pro Daily Schedule Dashboard), FEAT-16 (Booking & Payment Activity Record) |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|-----------------|-----------|---------|
| Recipient phone number | Client -- phone | Every WhatsApp send | The capability must know where to deliver the message |
| Message body text | Message -- the composed content for that send (already resolved by the sending Notification spec: FEAT-08.SPEC-001, 002 or 004) | Every WhatsApp send | The capability needs the exact content to transmit |
| Sender identity (the Pro's account, in vendor-neutral terms) | Pro Account -- an account-level sending identity | Every WhatsApp send | Lets the recipient attribute the message consistently to the sending Pro's account |

Client and Pro private notes, booking history beyond the single message's content, payment details, and every other product field never leave the product through this capability.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|----------------|------------------------------|
| Delivery status (Queued / Sent / Delivered / Failed) | The capability reports a status change for a sent WhatsApp message | Message -- delivery_status |
| Send-rejected notice (the recipient's number is not reachable on WhatsApp) | The capability determines, at send time, that the number cannot receive a WhatsApp message | Message -- delivery_status set to Failed, with the rejection reason available to FEAT-26.SPEC-003 as its trigger data |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|---------------|----------------|------------------|
| Delivery status: Sent | The capability confirms the WhatsApp message left the sending system | Message.delivery_status set to Sent | None -- an intermediate status, not shown to either party | -- |
| Delivery status: Delivered | The capability confirms the message reached the recipient's WhatsApp | Message.delivery_status set to Delivered | None directly -- delivery success is the expected, silent outcome | -- |
| Delivery status: Failed | The capability reports the message could not be delivered | Message.delivery_status set to Failed | Triggers FEAT-26.SPEC-003's fallback to text or email; no direct client feedback (Riley never receives a "delivery failed" message about her own confirmation) | FEAT-26.SPEC-003 |
| Send rejected -- number not WhatsApp-reachable | The capability determines at send time that the recipient's number has no WhatsApp account or cannot receive WhatsApp messages | Message.delivery_status set to Failed; the rejection is distinguished from a delivery failure only in the reason recorded, not in the resulting status | Triggers FEAT-26.SPEC-003's fallback to text or email, identically to a Failed delivery status; no direct client feedback | FEAT-26.SPEC-003 |

## Degradation Behavior

No screen in this feature sends a WhatsApp message synchronously in front of a user -- every send is background/asynchronous to the triggering event (a booking confirmation, a scheduled reminder, or a change notice), the same disposition as FEAT-08.SPEC-012's text sends. The rows below use FEAT-08's Notification specs as the "affected screen" column, since those specs' content is what this capability dispatches; the client-facing experience of a delayed or failed WhatsApp send is entirely governed by FEAT-26.SPEC-003's fallback, not by a loading state on any screen.

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|----------------------------|-------------------|--------------------|------------------------|
| FEAT-08.SPEC-001 (Booking Confirmation Message) | The send is queued and dispatched as soon as the capability responds; no client-facing screen waits on it, since the confirmation is sent in the background after payment completes | The send attempt is recorded as Failed once a defined timeout is reached; FEAT-26.SPEC-003's fallback to text or email takes over -- no booking-flow screen is blocked | Recorded as Failed (send-rejected, per Inbound Events) and handed to FEAT-26.SPEC-003 |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Same background handling as above -- no user-facing screen is affected while the send is slow | Same Failed-then-fallback handling as above | Same Failed-then-fallback handling as above |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Same background handling as above | Same Failed-then-fallback handling as above | Same Failed-then-fallback handling as above |
| FEAT-26.SPEC-001 (WhatsApp Channel Preference) | N/A -- this screen only captures the client's channel preference; it never itself waits on this capability | N/A -- a capability outage never blocks Riley from choosing WhatsApp as her preference; eligibility and delivery are evaluated later, at send time | N/A -- this screen sends no WhatsApp message itself, so a send cannot be rejected here |

## Consent and Disclosure

- **WhatsApp opt-in disclosure at FEAT-26.SPEC-001** -- Before a client's channel preference is saved as WhatsApp, the consent line on FEAT-26.SPEC-001 states plainly that her phone number and booking-related message content are used to send her WhatsApp updates about her appointment, naming no vendor; the write proceeds only once she taps Save with that line visible.
- **What is shared with the capability** -- The recipient's phone number and the already-composed message content for that single send; no vendor is named to the client, consistent with this spec's vendor-neutral category framing.
- **What is never shared** -- Client private notes, the Pro's private notes about the client, payment or card details, and any content beyond the single message being sent never reach this capability. Card data is never held or transmitted by the product at all (SC-11), and this capability has no channel through which it could receive it.
- **Consent remains channel-aware and revocable** -- Every WhatsApp send this capability makes is governed by FEAT-26.SPEC-004's fresh-per-send eligibility check against channel-scoped Messaging Consent; a client who reverts to text or whose consent lapses stops receiving WhatsApp sends on the very next message, per XBR-15.

## Edge Cases

- **A delivery-status event arrives for a Message whose Booking has since been cancelled and archived into history** -- The event is recorded against the Booking's retained history record (bookings are never deleted, per feature-dependency-map.md), and no user feedback fires beyond what FEAT-26.SPEC-003 already governs.
- **The same delivery-status event is delivered twice** -- The second delivery changes nothing: a Message already Delivered stays Delivered, and FEAT-26.SPEC-003's fallback does not re-trigger for an already-resolved Message.
- **Events arrive out of order (a Delivered status arrives before its preceding Sent status)** -- The Message reflects the most recent event by the capability's own reported event time, not arrival time; an out-of-order Sent arriving after Delivered does not regress the status.
- **The capability goes down mid-send, with no confirmation either way** -- If no Sent or Failed status is ever received within a defined timeout, the send is treated as Failed for the purpose of triggering FEAT-26.SPEC-003's fallback, so a message is never left in an indefinite unknown state.
- **A send-rejected notice arrives for a number that was WhatsApp-reachable at an earlier send** -- The rejection is honored for this send regardless of past reachability (a client's WhatsApp account may have been deleted or the number reassigned since); FEAT-26.SPEC-003's fallback still applies, and no assumption from a prior successful send carries forward.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|-------------------|--------------|
| FEAT-08.SPEC-001 (Booking Confirmation Message) | Triggered by (inbound) | Sends the confirmation over WhatsApp when FEAT-26.SPEC-004 finds the channel eligible |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Triggered by (inbound) | Sends the reminder over WhatsApp when eligible |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Triggered by (inbound) | Sends the change notice over WhatsApp when eligible |
| FEAT-26.SPEC-004 (WhatsApp Channel Eligibility & Consent Rule) | Triggered by (inbound) | Hands this spec the send once eligibility is confirmed |
| FEAT-26.SPEC-001 (WhatsApp Channel Preference) | References (inbound) | The Consent and Disclosure wording here matches the consent line shown at opt-in |
| FEAT-26.SPEC-003 (WhatsApp Delivery Fallback) | Triggers (outbound) | A Failed delivery status or a send-rejected event fires this automation |
| FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every WhatsApp send and delivery event is written to the append-only activity record, same as every other channel |
| FEAT-12 (Pro Daily Schedule Dashboard) | Affects (outbound) | Delivery status for a WhatsApp-channel message is visible to Talia the same way as any other channel |

## Analytics and Success Signals

- **whatsapp_send_attempted** (sending_spec: spec ID) -- N/A -- no metric in success-metrics.md names WhatsApp Reminders as its Connected Feature; retained as the operational baseline behind delivery-status observability, mirroring FEAT-08.SPEC-012's identical baseline event for text.
- **whatsapp_delivery_status_received** (status: sent / delivered / failed) -- N/A -- no Stage 2 metric is connected to this feature; retained because an unobserved WhatsApp delivery gap would otherwise undermine "Reminder Response Rate" (connected to FEAT-08) for clients who opted into WhatsApp, without either metric being able to detect why.
- **whatsapp_send_rejected** (reason: number_not_reachable) -- N/A -- no Stage 2 metric measures WhatsApp reachability specifically; retained so the real-world size of the fallback path (FEAT-26.SPEC-003) is observable rather than assumed.

## Acceptance Criteria

**FEAT-26.SPEC-002-AC-01:** Given Riley has an eligible WhatsApp preference and a confirmation is ready to send, when FEAT-26.SPEC-004 hands the send to this capability, then it sends the message and reports back a delivery status.

**FEAT-26.SPEC-002-AC-02:** Given a WhatsApp message sent through this capability is confirmed delivered, when the Delivered status arrives, then the Message record's delivery_status is set to Delivered and no further action is taken.

**FEAT-26.SPEC-002-AC-03:** Given a WhatsApp message sent through this capability cannot be delivered, when the Failed status arrives, then FEAT-26.SPEC-003's fallback automation is triggered.

**FEAT-26.SPEC-002-AC-04:** Given Riley's number has no WhatsApp account, when a send to her is attempted, then the capability reports a send-rejected event, the Message is recorded Failed, and FEAT-26.SPEC-003's fallback automation is triggered identically to a delivery failure.

**FEAT-26.SPEC-002-AC-05:** Given the capability is temporarily slow to respond, when a reminder is queued for sending, then no client-facing screen shows a waiting state, since the send is asynchronous to the reminder schedule.

**FEAT-26.SPEC-002-AC-06:** Given the capability is down when a confirmation attempts to send, when no Sent or Failed status is received within the defined timeout, then the send is treated as Failed and handed to FEAT-26.SPEC-003.

**FEAT-26.SPEC-002-AC-07:** Given the same Delivered event for one Message is delivered twice by the capability, when the second event arrives, then nothing changes and no duplicate action fires.

**FEAT-26.SPEC-002-AC-08:** Given a Delivered event arrives before its preceding Sent event for the same Message, when both are processed, then the Message reflects Delivered and the late-arriving Sent event does not regress it.

**FEAT-26.SPEC-002-AC-09:** Given Riley is shown the WhatsApp consent line on FEAT-26.SPEC-001 before her first opt-in, when she reads it, then it states plainly that her phone number and booking message content are used to send her WhatsApp updates, naming no vendor.

**FEAT-26.SPEC-002-AC-10:** Given a delivery-status event arrives for a Message tied to a booking that has since been cancelled and archived, when the event is processed, then it is recorded against the retained history record with no additional user-facing feedback beyond FEAT-26.SPEC-003's governance.

**FEAT-26.SPEC-002-AC-11:** Given Riley's card details are never held by the product, when this capability sends any WhatsApp message, then no payment or card data is ever included in the message content or the data exchanged with the capability.

**FEAT-26.SPEC-002-AC-12:** Given a client's WhatsApp channel is currently ineligible (per FEAT-26.SPEC-004), when a send is about to go out, then this capability is never invoked for that send; the message routes to FEAT-08.SPEC-011's text/email decision instead.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 4 | 4 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 4 (screens/specs; N/A cells for FEAT-26.SPEC-001 excluded from the count where genuinely inapplicable) | 4 |
| Consent and Disclosure | 4 | 4 |
| Edge Cases | 5 | 5 |
