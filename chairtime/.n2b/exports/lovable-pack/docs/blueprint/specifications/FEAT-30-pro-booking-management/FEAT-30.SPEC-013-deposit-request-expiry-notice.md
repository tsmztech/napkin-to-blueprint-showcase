---
document_type: spec
spec_type: notification
spec_id: FEAT-30.SPEC-013
spec_name: Deposit Request & Expiry Notice
spec_slug: deposit-request-expiry-notice
parent_feature: FEAT-30
parent_feature_name: Pro Booking Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Notification Spec: Deposit Request & Expiry Notice

## Overview

**Name:** Deposit Request & Expiry Notice
**ID:** FEAT-30.SPEC-013
**Type:** Notification
**Purpose:** Delivers a Pro-created deposit request to the client -- by text, by email, or as an on-screen code to scan -- and, when an unpaid request expires, stops any pending delivery for it. The Pro's expiry notice is not defined here: FEAT-03.SPEC-007 is its trigger and FEAT-08.SPEC-006 is its content owner.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- The client-facing deposit-request delivery, on its three delivery paths (text link, email link, on-screen code)
- Stopping any still-pending delivery of a deposit request once its hold expires unpaid (the Pro's expiry notice itself is referenced, not defined, here)
- The channel-decision rule for choosing among the three delivery paths

**Non-Goals:**
- Creating the Booking, placing the slot hold, or computing the hold's expiry -- owned by FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) and FEAT-03.SPEC-007; this spec only delivers the request once those specs create it, and reports the expiry outcome those specs determine
- Capturing the client's deposit payment itself -- owned by FEAT-07 (Deposit Payment at Booking); this notice's link or code hands off to that flow, it does not process payment
- The Pro-facing expiry notice for an unpaid request -- triggered by FEAT-03.SPEC-007 (the sole writer of Booking -> Expired (unpaid), XBR-02) and content-owned by FEAT-08.SPEC-006 (Pro Attention Alert); this spec defines no duplicate content, channels, or preferences for it
- The outcome notice for a cancellation, reschedule, or goodwill refund -- owned by FEAT-30.SPEC-012 (Pro Action Client Notice), a distinct content class from a payment request
- Choosing the delivery channel's underlying send mechanics (text/email) -- owned by FEAT-08.SPEC-012/FEAT-08.SPEC-013, the category-level transactional messaging capability every notification in the product sends through

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active Messaging Consent for texting (FEAT-14) | The fastest way for a client to act on a time-limited deposit request while the moment (e.g., booking their next visit at the chair) is fresh |
| Email | The client has not granted texting consent | Ensures the request always reaches the client, per BRIEF.md's stated email fallback |
| On-screen code | Talia chooses to show the request on her own screen rather than send it remotely (e.g., the client is standing at the chair) | Lets a client without a phone number capture step, or one who prefers to act immediately, pay on the spot by scanning |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A Pro-created deposit request is issued | FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) | Fires the instant the Booking and its slot hold are created successfully | Booking (service, start_time, deposit_amount), Client (name, phone or email), delivery choice Talia selected on FEAT-30.SPEC-004 |
| A Pro-created deposit request expires unpaid (reference only) | FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration) -- the trigger for the Pro's expiry notice, whose content FEAT-08.SPEC-006 owns | Fires when the hold's computed expiry passes with the deposit never paid; this spec's only action is to withdraw any still-pending deposit-request delivery for that Booking | Booking reference |

## Audience and Preferences

**Recipients:** The Client Talia is booking in (Access Matrix: Booking & Payment = Own-only for the Client's own booking; created here by Talia's Full access). The Pro's expiry notice, addressed to Talia (Access Matrix: Booking & Payment = Full for the Pro), is defined by FEAT-08.SPEC-006, not here. Platform Operator (Support) has View-only access to delivery status only, per the Access Matrix.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (governs the client's delivery channel, not whether the request is delivered) | Granted / Revoked | Captured at the client's first booking, or granted inline if this is a brand-new client entered by Talia | FEAT-06 at booking; changed via FEAT-14 |
| Delivery method choice (text/email link vs. on-screen code) | Link (text or email, per consent) / On-screen code | Talia's explicit choice on FEAT-30.SPEC-004 for each booking-in action -- no stored default | FEAT-30.SPEC-004 |

The deposit request itself carries no client opt-out: it is the client's own payment request for a booking Talia entered on their behalf, transactional by nature. The Pro's channel preferences for the expiry notice (set in FEAT-27) are applied by FEAT-08.SPEC-006, not by this spec.

**Quiet Hours:** N/A -- the deposit request is delivered immediately regardless of time of day, since it is the direct, expected consequence of Talia's own booking-in action at the moment she takes it (often at the chair, in front of the client).

## Content Definition

**Text (deposit request):**
- **Body:** {pro_display_name} has booked you in for {service_name} on {appointment_date} at {appointment_time}. Pay your {deposit_amount} deposit to confirm: {deposit_link}. This link expires in {hold_window_description}.

**Email (deposit request):**
- **Subject:** Confirm your appointment with {pro_display_name}
- **Body:** Hi {client_first_name}, {pro_display_name} has booked you in for {service_name} on {appointment_date} at {appointment_time}. Pay your {deposit_amount} deposit to confirm your spot.
- **CTA (button):** Pay deposit -- deep-links to FEAT-07 (Deposit Payment at Booking) for this Booking

**On-screen code (shown on Talia's device):**
- **Title:** Scan to pay your deposit
- **Body:** {client_first_name}, scan this code to pay your {deposit_amount} deposit for {service_name} on {appointment_date}.
- **CTA:** The code itself deep-links to FEAT-07 (Deposit Payment at Booking) for this Booking when scanned

**Pro expiry notice:** Not defined in this spec. When a deposit request's hold expires unpaid, FEAT-03.SPEC-007 triggers FEAT-08.SPEC-006 (Pro Attention Alert), which owns the in-app alert, text, and email content and the Pro's channel handling.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | Never empty (required field) |
| {client_first_name} | Client -- name (first token) | Riley | Renders the full name field if no separable first token exists |
| {service_name} | Service -- name | Full Set Lashes | Never empty (required field) |
| {appointment_date} / {appointment_time} | Booking -- start_time | Oct 4, 2026 / 2:30 PM | Never empty -- fixed at booking |
| {deposit_amount} | Deposit -- computed once from the Service's rule (FEAT-07) | $40.00 | Never empty -- computed at booking creation |
| {deposit_link} | Derived -- the deposit-payment deep link for this specific Booking (FEAT-07) | chairtime.app/pay/9c1f2a | Never empty -- generated the instant the Booking and its hold are created |
| {hold_window_description} | Derived -- a plain-language rendering of whichever of platform parameter: `deposit-request-hold-max-hours` or platform parameter: `deposit-request-hold-appointment-cutoff-hours` governs this specific request | 24 hours / 2 hours | Never empty -- always resolves to one of the two governing limits |

## Delivery Rules

**Batching:** None -- each deposit request is its own single send tied to one Booking; Talia booking in several clients in succession produces one independent request per client, never a combined message.
**Deduplication:** At most one deposit-request send per booking-in action, and no expiry message from this spec (the Pro's expiry notice belongs to FEAT-08.SPEC-006). Talia re-sending the same request (e.g., the client asks her to resend the link) triggers a fresh send of the same content, not treated as a new booking or a duplicate expiry.
**Retry on failure:** Governed by FEAT-08.SPEC-009 for the client-facing text/email delivery: a failed text is retried once, then falls back to email; the on-screen code path has no delivery-failure mode, since it renders directly on Talia's own device. 
**Expiry:** The deposit-request notice itself never separately "expires" -- its content states the governing hold window, and the underlying Booking's hold expiring is what FEAT-03.SPEC-007 and FEAT-30.SPEC-010 own; once that hold expires, no further reminder is sent for the same request (per the Brief's Side-Effect Inventory: "the client receives no further reminder for this booking"). 

## Edge Cases

- **Talia chooses the on-screen code instead of a link** -- No text or email is sent at all for the request itself; the code renders directly on her device, and the client scans it in person. If the code is never scanned in time, the hold expires as usual and Talia is notified by FEAT-08.SPEC-006.
- **The client pays the deposit before this notice's text or email delivery completes** -- The pending send is superseded: no further reminder about paying is delivered once FEAT-07 confirms the payment, since the request has already served its purpose.
- **A new client is entered by Talia with no texting consent captured yet (declined during entry, per FEAT-30.SPEC-004)** -- The deposit request delivers by email, using the required-when-texting-declined email address captured at entry (per FEAT-05's equivalent rule, applied here for a Pro-entered client).
- **The deposit request expires while a send is still pending or Talia is mid-way through booking another client** -- Any still-pending delivery for the expired request is withdrawn, and the expiry (whose Pro notice FEAT-08.SPEC-006 delivers independently) does not interrupt or merge with Talia's in-progress second booking-in action or its own deposit request.
- **Talia re-sends the same deposit request after the client says they didn't receive it** -- A fresh send of the identical content goes out on the same delivery method originally chosen; this does not reset the underlying hold's expiry, which FEAT-03.SPEC-007 continues to track from the hold's original creation time.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-010 (Pro-Created Booking & Deposit Request Hold) | Triggered by (inbound) | A created Booking and hold fires the deposit-request delivery; an expired hold withdraws any pending delivery |
| FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration) | Triggered by (inbound) | The hold-expiry event withdraws any pending delivery here; the same event is the trigger for the Pro's expiry notice, owned by FEAT-08.SPEC-006 |
| FEAT-08.SPEC-006 (Pro Attention Alert) | References (outbound) | Content owner of the Pro's expiry notice; this spec defines no duplicate content |
| FEAT-30.SPEC-004 (Book Client In) | References (inbound) | Talia's delivery-method choice (link vs. on-screen code) governs which content variant sends |
| FEAT-07 (Deposit Payment at Booking) | Navigation (outbound) | Every client-facing variant's CTA deep-links here to complete payment |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (inbound) | Decides text vs. email for the client-facing send |
| FEAT-08.SPEC-012 / FEAT-08.SPEC-013 (Text / Email Capabilities) | Triggers (outbound) | Perform the actual sends for the request |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback on failure for the request |

## Analytics and Success Signals

- **deposit_request_delivered** (channel: text / email / on_screen_code) -- supports success-metrics.md: "Pro Change Correctness"
- **deposit_request_paid** (channel) -- supports success-metrics.md: "Pro Change Correctness"
- **pro_expiry_notice_signal** () -- N/A -- the Pro's expiry notice and its signal (pro_attention_alert_sent, condition: deposit_request_expired) belong to FEAT-08.SPEC-006; this spec sends no expiry notice
- **deposit_request_cta_tapped** (channel) -- N/A -- no Stage 2 metric measures deposit-request tap rate directly; retained alongside deposit_request_paid so the funnel between delivery and payment is observable rather than measured only at the endpoints.

## Acceptance Criteria

**FEAT-30.SPEC-013-AC-01:** Given Talia books Riley in and chooses to send a text deposit request, when FEAT-30.SPEC-010 creates the Booking and hold, then Riley receives a text naming the service, date, time, deposit amount, and a payment link with the governing hold window stated.

**FEAT-30.SPEC-013-AC-02:** Given Riley has not granted texting consent, when the deposit request is issued, then it is delivered by email instead, using her email address on file.

**FEAT-30.SPEC-013-AC-03:** Given Talia chooses the on-screen code instead of a link, when the Booking and hold are created, then a scannable code renders on Talia's device and no text or email is sent for the request.

**FEAT-30.SPEC-013-AC-04:** Given a Pro-created deposit request's hold expires unpaid, when FEAT-03.SPEC-007's expiration path fires, then any still-pending delivery of that request is withdrawn, and Talia's expiry notice is triggered by FEAT-03.SPEC-007 and delivered with the content and channels FEAT-08.SPEC-006 defines; this spec sends no expiry message itself.

**FEAT-30.SPEC-013-AC-05:** Given a deposit request expires unpaid, when the expiry occurs, then this spec sends Talia no text, email, or in-app message and defines no wording for one, since FEAT-08.SPEC-006 applies her notification_preferences to the expiry notice.

**FEAT-30.SPEC-013-AC-06:** Given Riley completes the deposit payment before this notice's text delivery finishes retrying, when payment is confirmed, then no further payment-reminder content is sent for the same request.

**FEAT-30.SPEC-013-AC-07:** Given Talia enters a brand-new client who declines texting during entry, when the deposit request is issued, then it delivers by email to the address captured at entry.

**FEAT-30.SPEC-013-AC-08:** Given a text deposit request fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the request by email.

**FEAT-30.SPEC-013-AC-09:** Given Talia re-sends the same deposit request after the client reports not receiving it, when the resend completes, then the identical content is delivered again on the same originally chosen method, and the hold's expiry timing is unaffected.

**FEAT-30.SPEC-013-AC-10:** Given a deposit request's text delivery is still retrying when its hold expires unpaid, when the expiry occurs, then no further delivery attempt for that request is made and no payment link is sent after expiry.

**FEAT-30.SPEC-013-AC-11:** Given the deposit request's on-screen code is scanned, when the client follows it, then they land on FEAT-07 (Deposit Payment at Booking) for that specific Booking.

**FEAT-30.SPEC-013-AC-12:** Given Talia is mid-way through booking a second client when a first client's deposit request expires, when the expiry occurs, then the second booking-in action and its own deposit request are unaffected.

**FEAT-30.SPEC-013-AC-13:** Given a deposit request's governing hold window is the appointment-proximity cutoff rather than the 24-hour cap, when the request's content renders, then {hold_window_description} states the shorter, correct window rather than always showing 24 hours.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 3 (text, email, on-screen code) | 3 |
| Trigger Paths | 2 (request issued, request expired -- reference only) | 2 |
| Preference States | 3 (consent granted, consent revoked/declined, delivery method choice) | 3 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
