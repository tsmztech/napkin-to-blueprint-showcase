---
document_type: spec
spec_type: notification
spec_id: FEAT-08.SPEC-002
spec_name: Appointment Reminder Message
spec_slug: appointment-reminder-message
parent_feature: FEAT-08
parent_feature_name: Automated Booking Messaging
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Notification Spec: Appointment Reminder Message

## Overview

**Name:** Appointment Reminder Message
**ID:** FEAT-08.SPEC-002
**Type:** Notification
**Purpose:** Reminds the client before their appointment and offers a one-tap "I'll be there" or "I need to reschedule" response, replacing the Pro's habit of texting reminders by hand.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- The pre-appointment reminder message content, on both client channels (text and email)
- The two embedded one-tap reply options and their exact wording
- Delivery timing and window constraints as computed by the paired Automation spec

**Non-Goals:**
- Computing when the reminder fires (the default two-day lead time, the 8am--9pm window, and the late-booking suppression rule) -- owned by FEAT-08.SPEC-007 (Reminder Scheduling & Timing Window Enforcement), which this spec's Trigger section defers to entirely.
- Processing which reply was tapped -- owned by FEAT-08.SPEC-008 (Reminder Reply Routing); this spec only defines what is sent and its two reply options, not what happens after a tap.
- The landing page shown after "I'll be there" is tapped -- owned by FEAT-08.SPEC-003 (Reminder Reply Acknowledgment).
- Choosing text vs. email for this send -- owned by FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active Messaging Consent for texting (FEAT-08.SPEC-011) | BRIEF.md's Vision names the reminder as a text with tap-reply options; a link tap works identically from a text |
| Email | The client has not granted texting consent | The Validation & Limits field states explicitly: "the one-tap replies are link taps, so a reply works the same by text or email" -- no client is left without a working reminder because they declined texting |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A confirmed Booking reaches its computed reminder send time | FEAT-08.SPEC-007 (Reminder Scheduling & Timing Window Enforcement) | Fires once per Booking, at the time FEAT-08.SPEC-007 computes, provided FEAT-08.SPEC-007 has not suppressed the reminder for a late booking | Booking (service, start_time, deposit_amount, balance_due, policy_version), Client (name, phone, email), Pro Account (studio_address, timezone) |

## Audience and Preferences

**Recipients:** The Client tied to the Booking (Access Matrix: Booking & Payment = Own-only), the sole recipient. Platform Operator (Support) never receives this message; Support has View-only access to delivery status only.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (governs channel, not whether the reminder sends) | Granted / Revoked | Captured at booking | FEAT-06 (Client Booking Identity) at booking; changed via FEAT-14 (Messaging Consent Management) |

The reminder carries no separate on/off toggle beyond texting consent itself -- product-features.md defines no reminder-mute preference; the reminder is a core commitment of the product's promise to the client (know when their appointment is and be able to respond), not a discretionary alert the client can silence while keeping the booking.

**Quiet Hours:** The reminder's own send-timing window (roughly 8am--9pm in the Pro's timezone, XBR-16) is computed and enforced entirely by FEAT-08.SPEC-007 before this spec's trigger ever fires -- by the time this notification is triggered, the window has already been satisfied. This spec therefore applies no separate quiet-hours logic of its own; see FEAT-08.SPEC-007 for the window computation and its nearest-allowed-time fallback.

## Content Definition

**Text:**
- **Body:** Reminder: {service_name} with {pro_display_name} on {appointment_date} at {appointment_time} ({timezone}). Balance due at your visit: {balance_due}. Reply or tap: {ill_be_there_link} I'll be there, or {reschedule_link} I need to reschedule.
- **CTA (I'll be there):** {ill_be_there_link} -- deep-links to FEAT-08.SPEC-008 (Reminder Reply Routing), which records the acknowledgment and forwards to FEAT-08.SPEC-003 (Reminder Reply Acknowledgment)
- **CTA (I need to reschedule):** {reschedule_link} -- deep-links to FEAT-08.SPEC-008 (Reminder Reply Routing), which routes into FEAT-10 (Client-Initiated Cancel/Reschedule) via the booking-specific manage link

**Email:**
- **Subject:** Reminder: your appointment with {pro_display_name} on {appointment_date}
- **Body:**
  Hi {client_first_name},

  Just a reminder about your upcoming appointment:

  Service: {service_name}
  Date & time: {appointment_date} at {appointment_time} ({timezone})
  Balance due at your visit: {balance_due}

  Let us know you're coming, or reschedule if something's come up:
- **CTA (button, I'll be there):** I'll be there -- deep-links to FEAT-08.SPEC-008 (Reminder Reply Routing)
- **CTA (button, I need to reschedule):** I need to reschedule -- deep-links to FEAT-08.SPEC-008 (Reminder Reply Routing), routing to FEAT-10

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | Never empty (required field) |
| {service_name} | Service -- name | Full Set Lashes | Never empty (required field) |
| {appointment_date} / {appointment_time} | Booking -- start_time, rendered in the Pro's timezone | Oct 4, 2026 / 2:30 PM | Never empty -- fixed at booking |
| {timezone} | Pro Account -- timezone | Eastern Time | Never empty (required per account, XBR-25) |
| {balance_due} | Booking -- balance_due (derived) | $85.00 | Renders "$0.00" when nothing further is owed |
| {ill_be_there_link} / {reschedule_link} | Access Link -- the booking-specific manage link (FEAT-08.SPEC-010), each carrying a distinct reply-action parameter | chairtime.app/m/8f2a1c?r=yes / ?r=resched | If link issuance fails, the reminder is held and retried per FEAT-08.SPEC-009 -- never sent without working reply links |
| {client_first_name} | Client -- name (first token) | Riley | Renders the full name field if no separable first token exists |

## Delivery Rules

**Batching:** None -- exactly one reminder is sent per Booking, at its single computed reminder time (FEAT-08.SPEC-007). A client with two separate upcoming bookings with the same Pro receives two separate reminders, each tied to its own appointment; the product defines no household- or client-level batching of reminders across bookings.
**Deduplication:** At most one reminder per Booking. FEAT-08.SPEC-007 computes exactly one reminder send time per booking and marks it produced once fired; a scheduler re-evaluation never re-sends a reminder already dispatched for that booking.
**Retry on failure:** Governed by FEAT-08.SPEC-009: a failed text is retried once, then falls back to email, with the delivery gap flagged on the Pro's dashboard (XBR-17).
**Expiry:** A reminder that has not been delivered by the time its Booking's appointment start_time passes is no longer sent -- reminding about an appointment that has already happened or passed its usefulness window serves no purpose. In that case, the underlying delivery failure is still flagged to the Pro via FEAT-08.SPEC-006 (Pro Attention Alert) so the gap is never silent.

## Edge Cases

- **Client taps "I'll be there" or "I need to reschedule" after the appointment has already passed** -- FEAT-08.SPEC-010's link scoping (booking-specific links stop working once the appointment passes, XBR-18) means the tap lands on FEAT-08.SPEC-003's expired-link state rather than processing a stale reply.
- **Both reply links are tapped (client changes their mind)** -- FEAT-08.SPEC-008 processes only the first tap it receives; a second tap on the other link is treated as a new action against the booking's then-current state (for example, if "I need to reschedule" already routed into FEAT-10, a later "I'll be there" tap on the same reminder no longer applies once the booking has moved into the reschedule flow).
- **The reminder's send time is reached but FEAT-08.SPEC-007 suppressed it (late booking)** -- No reminder is triggered at all for that booking; this is a design decision (the confirmation already sent serves instead), not a delivery failure, so no gap is flagged.
- **Reminder scheduled but the booking is cancelled or rescheduled before the reminder time arrives** -- The reminder is cancelled and never sent; the client instead already has the Booking Change & Refund Notice (FEAT-08.SPEC-004) reflecting the change. Reminding about a booking that no longer exists in its original form would confuse rather than help.
- **Client's texting consent is revoked between the confirmation and the reminder** -- The reminder honors the consent state current at reminder send time, per FEAT-08.SPEC-011's re-check-on-every-send rule; it is sent by email even though the confirmation went by text.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-007 (Reminder Scheduling & Timing Window Enforcement) | Triggered by (inbound) | Computes and fires this notification's send time |
| FEAT-08.SPEC-008 (Reminder Reply Routing) | Triggers (outbound) | Both CTA links deep-link into this automation |
| FEAT-08.SPEC-003 (Reminder Reply Acknowledgment) | References (outbound) | Reached via FEAT-08.SPEC-008 after an "I'll be there" tap |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | References (outbound) | Reached via FEAT-08.SPEC-008 after an "I need to reschedule" tap |
| FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) | References (inbound) | Supplies both reply link placeholders |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (inbound) | Decides text vs. email for this send |
| FEAT-08.SPEC-012 / FEAT-08.SPEC-013 (Text / Email Capabilities) | Triggers (outbound) | Perform the actual send |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback on failure |
| FEAT-08.SPEC-006 (Pro Attention Alert) | Triggers (outbound) | Fires if the reminder expires undelivered |
| FEAT-26.SPEC-004 (WhatsApp Channel Eligibility & Consent Rule) | References (inbound) | Runs before FEAT-08.SPEC-011's text/email decision; if the client is WhatsApp-eligible the send goes by WhatsApp, otherwise it falls through to FEAT-08.SPEC-011 |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every send of reminder is written to the append-only activity record |

## Analytics and Success Signals

- **reminder_sent** (channel: text / email) -- supports success-metrics.md: "Reminder Response Rate"
- **reminder_reply_confirmed** (reply: ill_be_there) -- supports success-metrics.md: "Reminder Response Rate"
- **reminder_reply_reschedule_requested** (reply: reschedule) -- supports success-metrics.md: "Reminder Response Rate"
- **reminder_expired_undelivered** (reason: delivery_failure) -- N/A -- no Stage 2 metric measures undelivered reminders directly; retained so a silently-missed reminder is never invisible, consistent with XBR-17.

## Acceptance Criteria

**FEAT-08.SPEC-002-AC-01:** Given Riley's booking reaches its computed reminder time and she has active texting consent, when FEAT-08.SPEC-007 fires the trigger, then Riley receives a text with the service, date/time, balance due, and both "I'll be there" and "I need to reschedule" tap options.

**FEAT-08.SPEC-002-AC-02:** Given Riley declined texting, when her reminder fires, then she receives the same content by email with both reply options as buttons.

**FEAT-08.SPEC-002-AC-03:** Given Riley taps "I'll be there" in her text reminder, when the tap registers, then FEAT-08.SPEC-008 records the acknowledgment and Riley lands on FEAT-08.SPEC-003.

**FEAT-08.SPEC-002-AC-04:** Given Riley taps "I need to reschedule" in her email reminder, when the tap registers, then FEAT-08.SPEC-008 routes her into FEAT-10 (Client-Initiated Cancel/Reschedule) for that booking.

**FEAT-08.SPEC-002-AC-05:** Given Riley's booking was cancelled before her reminder's computed send time arrived, when the send time passes, then no reminder is sent, since the reminder was cancelled at cancellation.

**FEAT-08.SPEC-002-AC-06:** Given Riley's booking was made after its own reminder point would have fired (FEAT-08.SPEC-007's suppression rule), when the scheduler evaluates it, then no reminder is triggered for that booking.

**FEAT-08.SPEC-002-AC-07:** Given a text reminder to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the reminder by email before her appointment.

**FEAT-08.SPEC-002-AC-08:** Given Riley's reminder cannot be delivered on any channel before her appointment's start_time passes, when the expiry cutoff is reached, then the reminder is no longer sent and FEAT-08.SPEC-006 flags the delivery gap to the Pro.

**FEAT-08.SPEC-002-AC-09:** Given Riley taps "I need to reschedule" after her appointment has already passed, when she follows the link, then she reaches FEAT-08.SPEC-003's expired-link state, not an active reschedule flow, because the booking-specific link has expired.

**FEAT-08.SPEC-002-AC-10:** Given Riley taps "I'll be there" and then taps "I need to reschedule" moments later on the same reminder, when the second tap is processed, then it is evaluated against the booking's then-current state rather than silently overwriting the first acknowledgment.

**FEAT-08.SPEC-002-AC-11:** Given Riley has two separate upcoming bookings with the same Pro, when each reaches its own computed reminder time, then she receives two separate reminders, each naming only its own appointment.

**FEAT-08.SPEC-002-AC-12:** Given Riley revokes texting consent between her confirmation and her reminder, when the reminder's send time arrives, then it is sent by email, honoring the consent state current at send time.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 2 (consent granted, consent revoked/declined) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
