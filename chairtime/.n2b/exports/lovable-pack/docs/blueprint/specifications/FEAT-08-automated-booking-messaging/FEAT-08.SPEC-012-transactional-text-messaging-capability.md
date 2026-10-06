---
document_type: spec
spec_type: integration
spec_id: FEAT-08.SPEC-012
spec_name: Transactional Text Messaging Capability
spec_slug: transactional-text-messaging-capability
parent_feature: FEAT-08
parent_feature_name: Automated Booking Messaging
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Integration Spec: Transactional Text Messaging Capability

## Overview

**Name:** Transactional Text Messaging Capability
**ID:** FEAT-08.SPEC-012
**Type:** Integration
**Purpose:** Sends every text message the product needs to deliver -- confirmations, reminders, change notices, Pro notifications, access links, and every other feature's text-based messages -- through an external text-messaging capability, and reports back each message's delivery status.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- Sending a text message on behalf of any spec in this feature, or in FEAT-06, FEAT-14, FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-26, FEAT-29, or FEAT-30, per the Feature Dependency Map's External Touchpoints table
- Receiving and reporting back delivery status (Queued, Sent, Delivered, Failed) for every text sent
- Degradation behavior when the capability is slow, down, or rejects a send
- Disclosure of what client and Pro data is shared with this capability

**Non-Goals:**
- Choosing the text-messaging vendor -- vendor selection is a Stage 4 architecture decision; BRIEF.md records no user mandate for a specific vendor.
- Deciding whether a given message should be sent by text at all -- owned by FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule); this spec sends whatever it is given once that decision has already been made.
- Retrying a failed text or falling back to email -- owned by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback), which consumes this spec's Failed status as its own trigger.
- WhatsApp messaging -- a distinct capability owned by FEAT-26.SPEC-002, deferred to Later per scope-boundaries.md; this spec covers standard text messaging only.

## Capability Category

**Category:** Transactional text messaging
**Dependency Source:** ASMP-32 -- "Transactional text-messaging capability, with email as a fallback channel" (assumptions-constraints.md, Dependencies)
**External Touchpoint:** "Transactional text messaging" row in feature-dependency-map.md, ## External Touchpoints (Features Involved: FEAT-08, FEAT-06, FEAT-14, FEAT-29, FEAT-30, FEAT-20, FEAT-21, FEAT-26)
**Vendor Mandate:** None -- vendor selection is a Stage 4 decision.

## Product Behaviors Enabled

| Product Behavior | Brief Capability Served | Delivered Through (Spec ID) |
|------------------|------------------------|------------------------------|
| Riley receives an immediate booking confirmation by text | Send an immediate confirmation message on successful booking | FEAT-08.SPEC-001 |
| Riley receives a pre-appointment reminder with one-tap reply options by text | Send an automatic reminder a set time before the appointment | FEAT-08.SPEC-002 |
| Riley receives a cancellation, reschedule, or refund notice by text | Tell the client when their booking is cancelled, rescheduled or refunded | FEAT-08.SPEC-004 |
| Talia receives a new-booking or client-activity notice by text | Notify the Pro of new bookings, client cancellations and reschedules | FEAT-08.SPEC-005 |
| Talia receives an attention alert by text | Notify the Pro of anything needing attention | FEAT-08.SPEC-006 |
| Riley receives an access link or opt-out confirmation by text (on behalf of FEAT-06, FEAT-14) | Enabling capability for those features' own client-facing texts | FEAT-06, FEAT-14 |
| Talia receives billing, sign-in, waitlist, or recurring-series texts (on behalf of FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-29, FEAT-30) | Enabling capability for those features' own texts | FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-29, FEAT-30 |

## Data Exchanged

**Leaves the product:**

| Data | Entity / Fields | Sent When | Purpose |
|------|----------------|-----------|---------|
| Recipient phone number | Client -- phone (or Pro Account -- sign_in_mobile, for a Pro-directed text) | Every text send | The capability must know where to deliver the message |
| Message body text | Message -- the composed content for that send (already resolved from the sending spec's template) | Every text send | The capability needs the exact content to transmit |
| Sender identity (the Pro's account, in vendor-neutral terms) | Pro Account -- an account-level sending identity | Every text send | Lets the recipient's carrier and device attribute the message consistently to Chairtime/the sending Pro's account |

Client and Pro private notes, booking history beyond the single message's content, payment details, and every other product field never leave the product through this capability.

**Enters the product:**

| Data | Received When | Lands In (Entity / Fields) |
|------|--------------|-----------------------------|
| Delivery status (Queued / Sent / Delivered / Failed) | The capability reports a status change for a sent text | Message -- delivery_status |
| Inbound reply content (a tapped link's URL parameters, or a STOP keyword) | The client replies to or taps a link in a received text | Routed to the specific automation that owns the reply (FEAT-08.SPEC-008 for a reminder reply; FEAT-14.SPEC-004 for a STOP reply) -- this spec never stores the raw reply text itself beyond what the routing needs, per ASMP-23's "the Pro ... does not receive the client's replies as raw texts" |

## Inbound Events

| Event | Condition | Data Changes | User Feedback | Affected Specs |
|-------|-----------|-------------|---------------|----------------|
| Delivery status: Sent | The capability confirms the text left the sending system | Message.delivery_status set to Sent | None -- an intermediate status, not shown to either party | FEAT-08.SPEC-009 |
| Delivery status: Delivered | The capability confirms the text reached the recipient's device | Message.delivery_status set to Delivered | None directly -- delivery success is the expected, silent outcome | -- |
| Delivery status: Failed | The capability reports the text could not be delivered | Message.delivery_status set to Failed | Triggers FEAT-08.SPEC-009's retry-then-fallback; no direct client feedback (the client never receives a "delivery failed" message about their own confirmation) | FEAT-08.SPEC-009 |
| Inbound reply/link tap received | The client taps a link embedded in a received text, or replies with a keyword (e.g., STOP) | Routed to the owning automation (FEAT-08.SPEC-008 or FEAT-14.SPEC-004); no data lands directly in this spec | The reply's own outcome screen (owned by the receiving automation) | FEAT-08.SPEC-008, FEAT-14 |
| Booking-specific manage link tapped from a delivered text | The client taps the manage link embedded in a confirmation or reminder text | No data lands in this spec; the tap is handed to FEAT-06.SPEC-002, which validates the Access Link and resolves it | The client lands on the booking detail (FEAT-06.SPEC-004) or the "request a new link" prompt (FEAT-06.SPEC-001), as decided by FEAT-06.SPEC-002 | FEAT-06.SPEC-002 |

## Degradation Behavior

| Affected Screen (Spec ID) | Capability Slow | Capability Down | Capability Rejects |
|---------------------------|-----------------|-----------------|--------------------|
| FEAT-08.SPEC-001 (Booking Confirmation Message) | The send is queued and dispatched as soon as the capability responds; no client-facing screen waits on it, since the confirmation is sent in the background after payment completes | The send attempt is recorded as Failed once a timeout is reached; FEAT-08.SPEC-009's retry-then-fallback to email takes over -- no booking-flow screen is blocked, since confirmation sending is asynchronous to the payment flow | Same as Capability Down: recorded as Failed and handed to FEAT-08.SPEC-009 |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Same background handling as above -- no user-facing screen is affected while the send is slow | Same Failed-then-retry/fallback handling as above | Same Failed-then-retry/fallback handling as above |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Same background handling as above | Same Failed-then-retry/fallback handling as above | Same Failed-then-retry/fallback handling as above |
| FEAT-08.SPEC-005 / FEAT-08.SPEC-006 (Pro Notifications) | Same background handling as above; the in-app copy of the notification is unaffected regardless, since in-app never depends on this capability | Same Failed-then-retry/fallback handling as above | Same Failed-then-retry/fallback handling as above |

N/A -- no screen in this feature sends a text synchronously in front of the user (every send in this feature is background/asynchronous to the triggering user action), so no screen shows a loading or blocked state tied directly to this capability; all degradation surfaces instead through FEAT-08.SPEC-009's retry/fallback and FEAT-08.SPEC-006's Pro-facing alert.

## Consent and Disclosure

- **Texting consent captured at booking** -- Before any text is sent to a client, FEAT-05 (Public Booking Page & Booking Flow) captures explicit opt-in with the exact wording shown at booking, per ASMP-24's US SMS-consent requirement; this spec never initiates a send without FEAT-08.SPEC-011 confirming that consent is currently active.
- **What is shared with the capability** -- The disclosure a client sees at booking states plainly that their phone number and booking-related message content are used to send them text updates about their appointment; it names no vendor, consistent with this spec's vendor-neutral category framing.
- **What is never shared** -- Client private notes, the Pro's private notes about the client, payment/card details, and any content beyond the single message being sent never reach this capability. Card data is never held or transmitted by the product at all (SC-11), and this capability has no channel through which it could receive it.
- **Opt-out is always available** -- Every client-directed text this capability sends on behalf of any feature (this one or another) includes or is otherwise governed by an opt-out mechanism owned by FEAT-14 (Messaging Consent Management); this spec's disclosure obligation includes never sending a client text once FEAT-08.SPEC-011 reports consent as inactive.

## Edge Cases

- **A delivery-status event arrives for a Message whose Booking has since been deleted from active view (cancelled and archived into history)** -- The event is recorded against the Booking's retained history record (bookings are never deleted, per feature-dependency-map.md), and no user feedback fires beyond what FEAT-08.SPEC-009 already governs.
- **The same delivery-status event is delivered twice** -- The second delivery changes nothing: a Message already Delivered stays Delivered, and FEAT-08.SPEC-009's retry-then-fallback does not re-trigger for an already-resolved Message.
- **Events arrive out of order (a Delivered status arrives before its preceding Sent status)** -- The Message reflects the most recent event by the capability's own reported event time, not arrival time; an out-of-order Sent arriving after Delivered does not regress the status.
- **The capability goes down mid-send, with no confirmation either way** -- If no Sent or Failed status is ever received within a defined timeout, the send is treated as Failed for the purpose of triggering FEAT-08.SPEC-009's retry-then-fallback, so a message is never left in an indefinite unknown state.
- **A reply/link tap arrives for a Message whose Booking has already moved past the point that reply is meaningful (e.g., a reminder reply tap after the booking was already cancelled)** -- The routing automation (FEAT-08.SPEC-008) evaluates the tap against the Booking's current state and responds accordingly (typically the Expired/Invalid Link state), rather than this integration spec making that judgment itself.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-001 (Booking Confirmation Message) | Triggered by (inbound) | Sends the confirmation when text is the chosen channel |
| FEAT-08.SPEC-002 (Appointment Reminder Message) | Triggered by (inbound) | Sends the reminder when text is the chosen channel |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Triggered by (inbound) | Sends the change notice when text is the chosen channel |
| FEAT-08.SPEC-005 (Pro Booking Activity Notification) | Triggered by (inbound) | Sends the Pro notification when text is enabled |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggered by (inbound) | Sends the Pro alert when text is enabled |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | Affects (outbound) | A Failed delivery status fires this automation |
| FEAT-08.SPEC-008 (Reminder Reply Routing) | Affects (outbound) | Routes inbound reply-link taps |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (inbound) | Governs whether this capability is used for a given send |
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | Affects (outbound) | Receives every tap of a booking-specific manage link embedded in a text sent through this capability |
| FEAT-06, FEAT-14, FEAT-15, FEAT-18, FEAT-20, FEAT-21, FEAT-26, FEAT-29, FEAT-30 | Triggered by (inbound) | Each sends its own texts through this shared capability, per the External Touchpoints table |

## Analytics and Success Signals

- **text_send_attempted** (sending_spec: spec ID) -- N/A -- no Stage 2 metric measures raw send-attempt volume; retained as the operational baseline behind delivery-status metrics.
- **text_delivery_status_received** (status: sent / delivered / failed) -- supports success-metrics.md: "Reminder Response Rate" -- a reminder that never delivers cannot be responded to, so delivery reliability directly gates this metric's numerator.
- **text_send_rejected** (reason category) -- N/A -- no Stage 2 metric measures rejection frequency; retained so the capability's real-world reliability is observable rather than assumed.

## Acceptance Criteria

**FEAT-08.SPEC-012-AC-01:** Given Riley has active texting consent and a confirmation is ready to send, when FEAT-08.SPEC-011 selects text as the channel, then this capability sends the message and reports back a delivery status.

**FEAT-08.SPEC-012-AC-02:** Given a text sent through this capability is confirmed delivered, when the Delivered status arrives, then the Message record's delivery_status is set to Delivered and no further action is taken.

**FEAT-08.SPEC-012-AC-03:** Given a text sent through this capability cannot be delivered, when the Failed status arrives, then FEAT-08.SPEC-009's retry-then-fallback automation is triggered.

**FEAT-08.SPEC-012-AC-04:** Given Riley taps the "I'll be there" link inside a text sent through this capability, when the tap is received, then it is routed to FEAT-08.SPEC-008 for processing, and no raw reply content is stored beyond what that routing needs.

**FEAT-08.SPEC-012-AC-05:** Given the capability is temporarily slow to respond, when a confirmation is queued for sending, then no client-facing screen shows a waiting state, since the send is asynchronous to the payment flow.

**FEAT-08.SPEC-012-AC-06:** Given the capability is down when a reminder attempts to send, when no Sent or Failed status is received within the defined timeout, then the send is treated as Failed and handed to FEAT-08.SPEC-009.

**FEAT-08.SPEC-012-AC-07:** Given the same Delivered event for one Message is delivered twice by the capability, when the second event arrives, then nothing changes and no duplicate action fires.

**FEAT-08.SPEC-012-AC-08:** Given a Delivered event arrives before its preceding Sent event for the same Message, when both are processed, then the Message reflects Delivered and the late-arriving Sent event does not regress it.

**FEAT-08.SPEC-012-AC-09:** Given FEAT-14 needs to send an opt-out confirmation text, when it requests a send through this capability, then the send and its delivery-status reporting behave identically to a send requested by this feature's own specs.

**FEAT-08.SPEC-012-AC-10:** Given Riley is shown the texting-consent disclosure at booking, when she reads it, then it states plainly that her phone number and booking message content are used to send her text updates, naming no vendor.

**FEAT-08.SPEC-012-AC-11:** Given a delivery-status event arrives for a Message tied to a booking that has since been cancelled and archived, when the event is processed, then it is recorded against the retained history record with no additional user-facing feedback beyond FEAT-08.SPEC-009's governance.

**FEAT-08.SPEC-012-AC-12:** Given Riley's card details are never held by the product, when this capability sends any text, then no payment or card data is ever included in the message content or the data exchanged with the capability.

**FEAT-08.SPEC-012-AC-13:** Given a client's texting consent is inactive at send time, when FEAT-08.SPEC-011 evaluates the channel, then this capability is never invoked for that send; the message routes to FEAT-08.SPEC-013 (email) instead.

**FEAT-08.SPEC-012-AC-14:** Given Riley taps the manage link in a delivered confirmation text, when the tap arrives, then it is handed to FEAT-06.SPEC-002 for validation and this capability stores nothing from the tap beyond routing.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Product Behaviors | 8 | 8 |
| Inbound Events | 5 | 5 |
| Degradation Paths | 4 (screens; slow/down/rejects handled uniformly per screen) | 4 |
| Consent and Disclosure | 4 | 4 |
| Edge Cases | 5 | 5 |
