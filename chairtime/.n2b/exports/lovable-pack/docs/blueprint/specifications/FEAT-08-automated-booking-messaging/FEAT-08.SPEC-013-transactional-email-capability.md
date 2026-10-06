---
document_type: spec
spec_type: integration
spec_id: FEAT-08.SPEC-013
spec_name: Transactional Email Capability
spec_slug: transactional-email-capability
parent_feature: FEAT-08
parent_feature_name: Automated Booking Messaging
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Integration Spec: Transactional Email Capability

## Overview

**Name:** Transactional Email Capability
**ID:** FEAT-08.SPEC-013
**Type:** Integration
**Purpose:** Sends every email message the product needs to deliver -- as the fallback channel after a failed text and as the primary channel for clients who decline texting -- through an external transactional-email capability, and reports back each message's delivery status.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- Sending an email on behalf of any spec in this feature, or in FEAT-06, FEAT-14, FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-26, FEAT-29, or FEAT-30, per the External Touchpoints table
- Receiving and reporting back delivery status for every email sent
- Degradation behavior when the capability is slow, down, or rejects a send
- Disclosure of what client and Pro data is shared with this capability

**Non-Goals:**
- Choosing the email-delivery vendor -- vendor selection is a Stage 4 decision; BRIEF.md records no mandate.
- Deciding whether a given message should be sent by email -- owned by FEAT-08.SPEC-011 (as the client's chosen channel) or FEAT-08.SPEC-009 (as the fallback after a failed text); this spec sends whatever it is given once that decision has already been made.
- Retrying a failed email with a further fallback -- product-features.md and the Brief describe only a text-then-email chain; there is no channel beyond email, so an email failure is a final failure handled per this spec's own Inbound Events, not a further automated fallback.
- Marketing or promotional email content -- excluded per scope-boundaries.md (SC-15), consistent with this feature's texting exclusion of the same; every email sent through this capability is transactional (booking-related) content only.

## Capability Category

**Category:** Transactional email
**Dependency Source:** ASMP-32 -- "Transactional text-messaging capability, with email as a fallback channel" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Transactional email (fallback channel)" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-08, FEAT-06, FEAT-14, FEAT-29, FEAT-30, FEAT-18, FEAT-21, FEAT-26)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Riley (having declined texting) receives her booking confirmation by email | Send an immediate confirmation message on successful booking (fallback path) | FEAT-08.SPEC-001 |
| Riley receives her reminder by email when texting is declined | Send an automatic reminder a set time before the appointment (fallback path) | FEAT-08.SPEC-002 |
| Riley receives a change/refund notice by email when texting is declined | Tell the client when their booking is cancelled, rescheduled or refunded (fallback path) | FEAT-08.SPEC-004 |
| Riley receives any message by email after a failed text | Retry once, then fall back to email, so no client message is ever silently dropped | FEAT-08.SPEC-009 |
| Talia receives Pro notifications and alerts by email when enabled | Notify the Pro of new bookings, changes, and anything needing attention | FEAT-08.SPEC-005, FEAT-08.SPEC-006 |
| Riley/Talia receive access links, opt-out confirmations, billing notices, and other features' emails (on behalf of FEAT-06, FEAT-14, FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-26, FEAT-29, FEAT-30) | Enabling capability for those features' own email sends | Those features' own specs |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Recipient email address | Client -- email (or Pro Account -- sign_in_email, for a Pro-directed email) | Every email send | The capability must know where to deliver the message |
| Message subject and body content | Message -- the composed content for that send | Every email send | The capability needs the exact content to transmit |
| Sender identity (an account-level sending identity, vendor-neutral) | Pro Account -- an account-level sending identity | Every email send | Lets the recipient's mail client attribute the message consistently |

Client and Pro private notes, booking history beyond the single message's content, payment details, and every other product field never leave the product through this capability.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Delivery status (Queued / Sent / Delivered / Failed) | The capability reports a status change for a sent email | Message -- delivery_status |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Delivery status: Sent | The capability confirms the email left the sending system | Message.delivery_status set to Sent | None | -- |
| Delivery status: Delivered | The capability confirms acceptance by the recipient's mail server | Message.delivery_status set to Delivered | None -- the expected, silent outcome | -- |
| Delivery status: Failed | The capability reports the email could not be delivered (e.g., an invalid or bouncing address) | Message.delivery_status set to Failed | If this Failed status is itself the fallback attempt after a failed text (FEAT-08.SPEC-009), it escalates to FEAT-08.SPEC-006's "both channels fail" Pro alert; if it is a primary-channel email send (no prior text attempted), it also triggers FEAT-08.SPEC-006 so the Pro learns her client may not have received the message at all | FEAT-08.SPEC-009, FEAT-08.SPEC-006 |
| Booking-specific manage link tapped from a delivered email | The client taps the manage link embedded in a confirmation or reminder email | No data lands in this spec; the tap is handed to FEAT-06.SPEC-002, which validates the Access Link and resolves it | The client lands on the booking detail (FEAT-06.SPEC-004) or the "request a new link" prompt (FEAT-06.SPEC-001), as decided by FEAT-06.SPEC-002 | FEAT-06.SPEC-002 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-08.SPEC-001 (Booking Confirmation Message) | The send is queued and dispatched as soon as the capability responds; no client-facing screen waits on it | Recorded as Failed once a timeout is reached; escalated per Inbound Events above -- no booking-flow screen is blocked | Same as Capability Down |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Same background handling | Same Failed-and-escalate handling | Same Failed-and-escalate handling |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Same background handling | Same Failed-and-escalate handling | Same Failed-and-escalate handling |
| FEAT-08.SPEC-005 / FEAT-08.SPEC-006 (Pro Notifications) | Same background handling; the in-app copy is unaffected regardless | Same Failed-and-escalate handling | Same Failed-and-escalate handling |

N/A -- no screen in this feature sends an email synchronously in front of the user; every send is background/asynchronous to its triggering event, so degradation surfaces through Message.delivery_status and the escalation path above, not through a blocked or waiting screen.

## Consent and Disclosure

- **Email is disclosed as the fallback and no-texting-consent channel at booking** -- FEAT-05's booking flow states that a client who declines texting will receive booking updates by email instead, and that email is used automatically if a text ever fails to deliver; this is disclosed once, at booking, not re-disclosed on every individual fallback event.
- **What is shared with the capability** -- The client's email address and the content of the specific transactional message being sent; no vendor is named, consistent with this spec's vendor-neutral category framing.
- **What is never shared** -- Client private notes, the Pro's private notes, payment/card details, and any content beyond the single message being sent never reach this capability; card data is never held by the product at all (SC-11).
- **No marketing use** -- The disclosure at booking states that email is used only for the client's own booking-related messages, never for marketing or promotional content, consistent with scope-boundaries.md (SC-15).

## Edge Cases

- **A delivery-status event arrives for a Message tied to a since-cancelled and archived Booking** -- Recorded against the Booking's retained history record; no additional user-facing feedback beyond what the triggering Notification spec already governs.
- **The same delivery-status event is delivered twice** -- The second delivery changes nothing: a Message already Delivered stays Delivered, and no duplicate escalation fires.
- **Events arrive out of order (Delivered arrives before Sent)** -- The Message reflects the most recent event by the capability's own reported event time, not arrival time.
- **The capability goes down mid-send with no confirmation either way** -- If no Sent or Failed status is received within a defined timeout, the send is treated as Failed for escalation purposes, so a message is never left in an indefinite unknown state.
- **An email fallback is attempted for a client whose email address is missing or malformed** -- This should not occur given FEAT-05's requirement that an email be captured whenever texting is declined (product-features.md), but if it does, the send is recorded as Failed immediately (an invalid-address rejection) and escalates directly to FEAT-08.SPEC-006, since there is no further fallback channel beyond email.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-001 (Booking Confirmation Message) | Triggered by (inbound) | Sends the confirmation when email is the chosen or fallback channel |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Triggered by (inbound) | Sends the reminder when email is the chosen or fallback channel |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Triggered by (inbound) | Sends the change notice when email is the chosen or fallback channel |
| FEAT-08.SPEC-005 (Pro Booking Activity Notification) | Triggered by (inbound) | Sends the Pro notification when email is enabled |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggered by (inbound); Triggers (outbound) | Sends the Pro alert when email is enabled; also receives escalations when this capability itself fails |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | Triggered by (inbound) | Performs the fallback send after a failed text |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (inbound) | Selects email as the primary channel for consent-declined clients |
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | Affects (outbound) | Receives every tap of a booking-specific manage link embedded in an email sent through this capability |
| FEAT-06, FEAT-14, FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-26, FEAT-29, FEAT-30 | Triggered by (inbound) | Each sends its own emails through this shared capability |

## Analytics and Success Signals

- **email_send_attempted** (sending_spec: spec ID; reason: primary_channel / text_fallback) -- N/A -- no Stage 2 metric measures raw send-attempt volume; retained as the operational baseline behind delivery-status metrics.
- **email_delivery_status_received** (status: sent / delivered / failed) -- supports success-metrics.md: "Reminder Response Rate" -- a reminder that reaches no client on any channel cannot be responded to.
- **email_send_failed** (reason category) -- N/A -- no Stage 2 metric measures email failure frequency directly; retained so the "never silently dropped" guarantee (XBR-17) is observable at the final channel in the chain.

## Acceptance Criteria

**FEAT-08.SPEC-013-AC-01:** Given Riley declined texting at booking and provided an email, when her confirmation is ready to send, then this capability sends it and reports back a delivery status.

**FEAT-08.SPEC-013-AC-02:** Given a text to Riley failed and FEAT-08.SPEC-009's fallback runs, when the fallback email is sent, then this capability delivers it and reports the outcome back to the Message record created for that fallback.

**FEAT-08.SPEC-013-AC-03:** Given an email sent through this capability is confirmed delivered, when the Delivered status arrives, then the Message record's delivery_status is set to Delivered.

**FEAT-08.SPEC-013-AC-04:** Given a fallback email sent through this capability also fails, when the Failed status arrives, then FEAT-08.SPEC-006's escalated "both channels fail" alert fires for the Pro.

**FEAT-08.SPEC-013-AC-05:** Given a primary-channel email (no text attempted, consent-declined client) fails to deliver, when the Failed status arrives, then FEAT-08.SPEC-006 alerts the Pro that the client may not have received the message.

**FEAT-08.SPEC-013-AC-06:** Given the capability is down when an email attempts to send, when no Sent or Failed status is received within the defined timeout, then the send is treated as Failed for escalation purposes.

**FEAT-08.SPEC-013-AC-07:** Given the same Delivered event for one Message is delivered twice, when the second event arrives, then nothing changes and no duplicate escalation fires.

**FEAT-08.SPEC-013-AC-08:** Given a Delivered event arrives before its preceding Sent event, when both are processed, then the Message reflects Delivered and the late Sent event does not regress it.

**FEAT-08.SPEC-013-AC-09:** Given Riley is shown the email-fallback disclosure at booking, when she declines texting, then she is told plainly that booking updates will arrive by email instead and that email is used automatically if a text ever fails.

**FEAT-08.SPEC-013-AC-10:** Given a client's email address is missing or malformed at fallback time, when the send is attempted, then it is recorded as Failed immediately and escalates directly to FEAT-08.SPEC-006, since no further fallback channel exists.

**FEAT-08.SPEC-013-AC-11:** Given FEAT-18 needs to send a billing notice by email, when it requests a send through this capability, then the send and its delivery-status reporting behave identically to a send requested by this feature's own specs.

**FEAT-08.SPEC-013-AC-12:** Given this capability sends any email, when its content is composed, then no marketing or promotional content is ever included -- only the specific transactional message the triggering spec composed.

**FEAT-08.SPEC-013-AC-13:** Given Riley taps the manage link in a delivered confirmation or reminder email, when the tap arrives, then it is handed to FEAT-06.SPEC-002 for validation and this capability stores nothing from the tap beyond routing.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 5 | 5 |
| Inbound Events | 4 | 4 |
| Degradation Paths | 4 (screens; slow/down/rejects handled uniformly per screen) | 4 |
| Consent and Disclosure | 4 | 4 |
| Edge Cases | 5 | 5 |
