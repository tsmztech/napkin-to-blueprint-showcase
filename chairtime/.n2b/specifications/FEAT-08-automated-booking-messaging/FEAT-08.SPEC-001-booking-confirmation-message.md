---
document_type: spec
spec_type: notification
spec_id: FEAT-08.SPEC-001
spec_name: Booking Confirmation Message
spec_slug: booking-confirmation-message
parent_feature: FEAT-08
parent_feature_name: Automated Booking Messaging
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Notification Spec: Booking Confirmation Message

## Overview

**Name:** Booking Confirmation Message
**ID:** FEAT-08.SPEC-001
**Type:** Notification
**Purpose:** Tells the client, the moment their deposit payment succeeds, that their appointment is confirmed -- carrying every detail they need (what, when, where, what was paid, what's still owed, when they can no longer cancel free, and how to manage or add the booking to their own calendar) without needing to ask the Pro anything.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- The immediate post-payment confirmation message, on both of the product's client channels (text and email)
- Every content element BRIEF.md's Vision and this feature's Key Capabilities name: service, date/time with timezone, deposit paid, balance due in person, studio location, cancellation cut-off, manage link, add-to-calendar option
- Generating the add-to-calendar export itself: a calendar-event (.ics) link built directly from this Booking's own fields (service, start_time, studio_address) at the moment the confirmation is composed. This is a distinct action from the manage link (opens the booking in-product to manage it) -- add-to-calendar puts the appointment on the client's own calendar app and needs no in-product access grant, since it discloses nothing beyond what this message already shows the client
- Delivery timing (within about a minute of payment) and its channel selection

**Non-Goals:**
- Deciding text vs. email for this send -- owned by FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule), which this spec defers to before every send.
- Minting the manage link itself -- owned by FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance); this spec only embeds the link it produces.
- Retrying a failed send or falling back to email after a failed text -- owned by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback), which governs delivery failure for every Message this feature creates, including this one.
- Confirming a cancellation, reschedule, or refund -- excluded per product-features.md's Communications field, which treats the change/refund notice as a distinct communication; covered by FEAT-08.SPEC-004 (Booking Change & Refund Notice).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active Messaging Consent for texting (FEAT-08.SPEC-011) | BRIEF.md's Vision names text as the primary confirmation channel ("a confirmation text lands immediately"); Riley is mid-session on her phone right after paying and expects the confirmation where she already is |
| Email | The client has not granted texting consent, or provided only an email at booking (BRIEF.md's stated fallback: "Email confirmations are acceptable as a fallback") | Every client who declines texting still supplies an email at booking (product-features.md, Client entity), so the confirmation is never simply undeliverable |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Deposit payment completes and the Booking flips to Confirmed | FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Always, on every successful first-time deposit capture for a booking | Booking (service, start_time, price_agreed, deposit_amount, balance_due, policy_version), Client (name, phone, email), Pro Account (studio_address, timezone) |

## Audience and Preferences

**Recipients:** The Client tied to the Booking (Access Matrix: Booking & Payment = Own-only for the Client) -- the sole recipient of this message, since it discloses that client's own appointment and payment details. Platform Operator (Support) never receives this message; per the Access Matrix, Support has View-only access to delivery status (not message content) for troubleshooting.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (governs channel, not whether this message sends) | Granted / Revoked | Captured at booking (Granted or Revoked, per the client's choice) | FEAT-06 (Client Booking Identity) at booking; changed via FEAT-14 (Messaging Consent Management) |

This confirmation itself carries no on/off toggle: it is the transactional record of a payment the client just made, not a discretionary reminder. A client cannot opt out of being told their own booking is confirmed -- only the channel it arrives on varies, per FEAT-08.SPEC-011.

**Quiet Hours:** N/A -- the confirmation is a direct, expected response to an action the client just took (paying); it is not an unprompted interruption, so the daytime-hours window that governs FEAT-08.SPEC-002's reminders (XBR-16) does not apply here. The confirmation sends at whatever time the payment completes, day or night.

## Content Definition

**Text:**
- **Body:** {pro_display_name} confirmed: {service_name} on {appointment_date} at {appointment_time} ({timezone}). Deposit paid: {deposit_amount}. Balance due at your visit: {balance_due}. Location: {studio_address}. Free cancel/reschedule until {cancellation_cutoff}. Manage: {manage_link} Add to calendar: {calendar_export_link}
- **CTA (manage):** {manage_link} -- deep-links directly to this one booking through FEAT-06's (Client Booking Identity) link resolution
- **CTA (add to calendar):** {calendar_export_link} -- opens/downloads a calendar-event (.ics) file for this appointment on the client's own device; a separate action from the manage link, not a manage-link sub-option

**Email:**
- **Subject:** Confirmed: {service_name} with {pro_display_name} on {appointment_date}
- **Body:**
  Hi {client_first_name},

  Your appointment is confirmed:

  Service: {service_name}
  Date & time: {appointment_date} at {appointment_time} ({timezone})
  Location: {studio_address}
  Deposit paid: {deposit_amount}
  Balance due at your visit: {balance_due}
  Free to cancel or reschedule until: {cancellation_cutoff}

  Need to make a change? Use the link below. Want it on your calendar? Add it with one tap.
- **CTA (button, manage):** Manage my booking -- deep-links directly to this booking (FEAT-06's Client Booking Identity link resolution)
- **CTA (button, calendar):** Add to calendar -- opens/downloads the {calendar_export_link} calendar-event (.ics) file for this appointment

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | Never empty -- display_name is required (Pro Account entity, feature-dependency-map.md) |
| {service_name} | Service -- name | Full Set Lashes | Never empty -- required at service creation (FEAT-01) |
| {appointment_date} / {appointment_time} | Booking -- start_time, rendered in the Pro's timezone | Oct 4, 2026 / 2:30 PM | Never empty -- start_time is fixed at booking |
| {timezone} | Pro Account -- timezone | Eastern Time | Never empty -- required per account (XBR-25) |
| {deposit_amount} | Booking -- deposit_amount | $40.00 | Never empty -- fixed at booking (XBR-05) |
| {balance_due} | Booking -- balance_due (derived: price_agreed − deposit_amount) | $85.00 | Renders as "$0.00" when the deposit equals the full price -- never blank |
| {studio_address} | Pro Account -- studio_address | 123 Main St, Suite 4, Austin, TX | Never empty -- required before go-live (XBR-26) |
| {cancellation_cutoff} | Derived -- Booking.start_time minus Cancellation Policy.window_hours | Oct 2, 2026, 2:30 PM | Never empty -- every Booking carries a policy_version with a window_hours value (XBR-08) |
| {manage_link} | Access Link -- created by FEAT-08.SPEC-010, scoped to this Booking | chairtime.app/m/8f2a1c | If link issuance fails, the confirmation is held and retried per FEAT-08.SPEC-009's failure handling -- a confirmation is never sent without its manage link |
| {calendar_export_link} | Derived -- a calendar-event (.ics) file generated at send time directly from Booking (service, start_time) and Pro Account (studio_address, timezone) fields; not an Access Link and not scoped/expiring the way {manage_link} is, since it discloses nothing the confirmation itself does not already show | chairtime.app/cal/8f2a1c.ics | Never empty -- generated deterministically from fields that are always present on a confirmed Booking (same fields the confirmation body itself requires) |
| {client_first_name} | Client -- name (first token) | Riley | Renders the full name field if no separable first token exists |

## Delivery Rules

**Batching:** None -- exactly one confirmation is sent per Booking, at the single moment its deposit is captured. There is nothing to batch: a client receives at most one Booking Confirmation Message per booking.
**Deduplication:** At most one confirmation per Booking. FEAT-07.SPEC-002's deposit capture is itself guaranteed to fire at most once per booking (per its own idempotency rules, FEAT-07.SPEC-004), so this notification's trigger cannot re-fire for the same booking; a redundant trigger attempt is a no-op.
**Retry on failure:** Governed by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback): a failed text is retried once, then falls back to email, and the delivery gap is flagged on the Pro's dashboard (XBR-17) -- never silently dropped.
**Expiry:** None -- a booking confirmation never becomes not-worth-sending. Even a late-arriving confirmation (after a retry/fallback cycle) still carries currently accurate information, since Booking fields are fixed at booking time (XBR-04, XBR-05).

## Edge Cases

- **Deposit captured but manage-link issuance (FEAT-08.SPEC-010) has not yet completed** -- The confirmation send waits for the link; it is never sent with a placeholder or missing link. If link issuance itself fails, this is treated as a send failure under FEAT-08.SPEC-009's retry/fallback path.
- **Client has both texting consent and an email on file** -- Per FEAT-08.SPEC-011, text is used; no duplicate confirmation is also sent by email in this case.
- **Client's phone number changed since booking but before the confirmation sends** -- FEAT-08.SPEC-011 requires fresh consent for a changed number (XBR-15); until fresh consent exists, the confirmation routes to email, never to the old or unconsented number.
- **Pro's studio_address is edited between booking and confirmation send** -- The confirmation shows the studio_address value at send time (current Pro Account field), since the confirmation is generated at send, not pre-composed at booking; this is consistent with the Pro Account entity carrying one current address rather than a per-booking snapshot.
- **Booking is cancelled in the brief window between payment capture and confirmation send** -- The confirmation still sends (it reports what was true at the moment of successful payment); the client also promptly receives the Booking Change & Refund Notice (FEAT-08.SPEC-004) reflecting the cancellation, so the client is never left believing a cancelled booking is still active.
- **Client taps {calendar_export_link} after the confirmation has already been superseded by a cancellation or reschedule (FEAT-08.SPEC-004)** -- The calendar file still adds successfully, since it is a self-contained record of what the confirmation stated at send time, not a live-refreshing link; the client's device calendar entry may then be stale, which is why the manage link (not the calendar file) is the channel of record for any subsequent change.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Triggered by (inbound) | Successful deposit capture and Booking confirmation fires this notification |
| FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) | References (inbound) | Supplies the {manage_link} placeholder embedded in every confirmation |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (inbound) | Decides text vs. email for this send |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Triggers (outbound) | Performs the text send when text is the chosen channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | Triggers (outbound) | Performs the email send when email is the chosen channel or is the fallback |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback behavior if this send fails |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | References (outbound) | Covers the client-facing message if the booking is subsequently changed |
| FEAT-06 (Client Booking Identity) | Navigation (outbound) | The manage link's tap opens this booking through FEAT-06's link resolution |
| FEAT-26.SPEC-004 (WhatsApp Channel Eligibility & Consent Rule) | References (inbound) | Runs before FEAT-08.SPEC-011's text/email decision; if the client is WhatsApp-eligible the send goes by WhatsApp, otherwise it falls through to FEAT-08.SPEC-011 |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every send of this notification is written to the append-only activity record |

## Analytics and Success Signals

- **confirmation_sent** (channel: text / email; delay_seconds since payment capture) -- N/A -- no Stage 2 metric measures confirmation delivery speed directly, though the target time is stated in Non-Functional Notes; retained so send-latency compliance with the "within about a minute" commitment is observable.
- **confirmation_delivery_confirmed** (channel) -- N/A -- no dedicated Stage 2 metric; delivery reliability of this message feeds the trust that underlies "Deposit Capture Rate" (success-metrics.md) but is not itself that metric's numerator or denominator.
- **confirmation_cta_tapped** (destination: manage_link) -- supports success-metrics.md: "Self-Service Access Success"
- **confirmation_calendar_link_tapped** (channel: text / email) -- N/A -- no Stage 2 metric measures calendar-export usage directly; retained so the "add to my calendar" capability's actual usage is observable rather than assumed.

## Acceptance Criteria

**FEAT-08.SPEC-001-AC-01:** Given Riley just paid her deposit for a Full Set Lashes appointment and has active texting consent, when the payment capture completes (FEAT-07.SPEC-002), then Riley receives a text confirming the service, date/time with timezone, deposit paid, balance due, studio location, cancellation cut-off, and a manage link, within about a minute.

**FEAT-08.SPEC-001-AC-02:** Given Riley declined texting at booking and provided an email, when her deposit payment completes, then she receives the confirmation by email instead, with the same content elements.

**FEAT-08.SPEC-001-AC-03:** Given Riley receives her text confirmation, when she taps the manage link, then it opens her booking through FEAT-06's link resolution, scoped to this one appointment.

**FEAT-08.SPEC-001-AC-04:** Given a service priced at $125 with a $40 deposit, when the confirmation is composed, then it shows "Deposit paid: $40.00" and "Balance due at your visit: $85.00".

**FEAT-08.SPEC-001-AC-05:** Given the Pro's cancellation policy window is 24 hours and the appointment is at 2:30 PM on Oct 4, when the confirmation is composed, then it states the cancellation cut-off as Oct 3, 2:30 PM.

**FEAT-08.SPEC-001-AC-06:** Given Riley pays her deposit but manage-link issuance has not yet completed, when the confirmation send is attempted, then the send waits for the link and never dispatches a confirmation missing its manage link.

**FEAT-08.SPEC-001-AC-07:** Given Riley's booking is cancelled 30 seconds after her deposit captures, when the confirmation and the cancellation both process, then Riley receives both the confirmation (reflecting the moment of successful payment) and the Booking Change & Refund Notice (FEAT-08.SPEC-004) reflecting the cancellation.

**FEAT-08.SPEC-001-AC-08:** Given Riley's deposit capture (FEAT-07.SPEC-002) is guaranteed to fire at most once for her booking, when the confirmation trigger is evaluated, then at most one confirmation is ever sent for that booking.

**FEAT-08.SPEC-001-AC-09:** Given a text confirmation to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the confirmation, by email, and the delivery gap is flagged on the Pro's dashboard.

**FEAT-08.SPEC-001-AC-10:** Given Riley's phone number changed since her last booking and fresh texting consent has not yet been captured, when a new booking's confirmation is composed, then it is sent by email, never to the unconsented number.

**FEAT-08.SPEC-001-AC-11:** Given the Pro edits her studio_address between Riley's booking and the confirmation send, when the confirmation is composed, then it shows the studio_address value current at send time.

**FEAT-08.SPEC-001-AC-12:** Given Riley has both texting consent and an email on file, when her confirmation sends, then it arrives once, by text, and no duplicate email confirmation is also sent.

**FEAT-08.SPEC-001-AC-13:** Given Platform Operator (Support) is troubleshooting a delivery issue for Riley's booking, when Support views the Message record, then Support sees delivery status only, never the confirmation's content as a raw text.

**FEAT-08.SPEC-001-AC-14:** Given Riley taps "Manage my booking" from her email confirmation, when the tap registers, then the confirmation_cta_tapped event fires with destination "manage_link".

**FEAT-08.SPEC-001-AC-15:** Given Riley receives her confirmation by text or email, when she taps "Add to calendar" ({calendar_export_link}), then a calendar-event file is produced for her Full Set Lashes appointment carrying the service, date/time in her Pro's timezone, and studio location, independently of whether she also taps the manage link, and the confirmation_calendar_link_tapped event fires.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 2 (consent granted, consent revoked/declined) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |
