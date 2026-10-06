---
document_type: spec
spec_type: notification
spec_id: FEAT-08.SPEC-005
spec_name: Pro Booking Activity Notification
spec_slug: pro-booking-activity-notification
parent_feature: FEAT-08
parent_feature_name: Automated Booking Messaging
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Notification Spec: Pro Booking Activity Notification

## Overview

**Name:** Pro Booking Activity Notification
**ID:** FEAT-08.SPEC-005
**Type:** Notification
**Purpose:** Tells the Pro, on her own channels, that a new booking arrived or a client cancelled or rescheduled their own appointment -- so Talia learns about routine schedule changes without having to keep the dashboard open.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- The Pro-facing notice for a new booking and for a client-initiated cancellation or reschedule
- In-app, text, and email delivery per the Pro's own notification_preferences (FEAT-27)

**Non-Goals:**
- Anything needing the Pro's attention (delivery failures, calendar reconnection, refund failure, disputes) -- owned by FEAT-08.SPEC-006 (Pro Attention Alert), which is a distinct, higher-urgency notification class from routine activity.
- The client-facing counterpart of the same events -- owned by FEAT-08.SPEC-001 (new booking) and FEAT-08.SPEC-004 (cancellation/reschedule).
- Setting or changing notification_preferences -- owned by FEAT-27 (Pro Profile & Booking Page Settings); this spec only reads and honors that setting.
- Pro-initiated cancellations or reschedules -- the Pro does not need to be told about her own actions; this spec covers only client-initiated activity, per the Brief's Side-Effect Inventory.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, regardless of the Pro's text/email preference | The Pro's dashboard (FEAT-12) is her primary daily touchpoint (BRIEF.md's Vision: "you glance at your phone between clients"); in-app presence is the baseline, never opt-out-able |
| Text | The Pro's notification_preferences include text for this notification type | Talia is between clients on her phone most of the day; a text reaches her without opening the app |
| Email | The Pro's notification_preferences include email for this notification type | Some Pros prefer a running email record of activity they can search later |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| New booking is made | FEAT-05 (Public Booking Page & Booking Flow) via FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Always, on a booking's successful confirmation | Booking (service, start_time, deposit_amount), Client (name) |
| Client cancels their own booking | FEAT-10.SPEC-004 (Booking Update Commit), via its trigger contract FEAT-10.SPEC-006 (Client-Initiated Cancel/Reschedule) | Always, on a successful client cancellation | Booking (original time, cancellation timestamp), Client (name), Deposit Transaction (outcome) |
| Client reschedules their own booking | FEAT-10.SPEC-004 (Booking Update Commit), via its trigger contract FEAT-10.SPEC-006 (Client-Initiated Cancel/Reschedule) | Always, on a successful client reschedule | Booking (original time, new time), Client (name) |

## Audience and Preferences

**Recipients:** The Pro (Talia) tied to the affected Booking's account (Access Matrix: Booking & Payment = Full for the Pro). Platform Operator (Support) has View-only access to delivery status only; the Client is not a recipient of this notification (it is not their content to receive -- they get FEAT-08.SPEC-001 / FEAT-08.SPEC-004 instead).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Booking activity notification channels | In-app only / In-app + text / In-app + email / In-app + text + email | In-app + text (BRIEF.md's Vision centers texting as the Pro's own habit-replacement channel) | FEAT-27 (Pro Profile & Booking Page Settings) |

In-app is never an option to disable: the Pro Account entity's notification_preferences field governs text/email routing only, consistent with the dashboard being the system of record for her day.

**Quiet Hours:** N/A -- product-features.md and assumptions-constraints.md define no quiet-hours window for Pro-facing notifications; the Pro's own working day is not modeled as having off-hours the product enforces on her behalf, unlike the client-consent-driven daytime window that governs client texting (XBR-16, ASMP-29), which exists to satisfy US SMS-consent rules for the Client -- a rule that does not apply to the Pro's own opted-in business communications about her own account.

## Content Definition

**In-app:**
- **Title:** New booking: {client_name}
- **Body:** {service_name} on {appointment_date} at {appointment_time}. Deposit paid: {deposit_amount}.
- **CTA:** View booking -- deep-links to FEAT-12 (Pro Daily Schedule Dashboard), the specific booking row

**In-app (client cancellation):**
- **Title:** {client_name} cancelled
- **Body:** {service_name} on {appointment_date} at {appointment_time}. Deposit: {deposit_outcome_summary}.
- **CTA:** View schedule -- deep-links to FEAT-12

**In-app (client reschedule):**
- **Title:** {client_name} rescheduled
- **Body:** {service_name} moved from {original_appointment_date} to {new_appointment_date} at {new_appointment_time}.
- **CTA:** View schedule -- deep-links to FEAT-12

**Text (new booking):**
- **Body:** New booking: {client_name}, {service_name} on {appointment_date} at {appointment_time}. Deposit paid: {deposit_amount}.

**Text (cancellation):**
- **Body:** {client_name} cancelled their {appointment_date} appointment. Deposit: {deposit_outcome_summary}.

**Text (reschedule):**
- **Body:** {client_name} moved their appointment from {original_appointment_date} to {new_appointment_date} at {new_appointment_time}.

**Email (mirrors each text variant with a subject line "Booking update: {client_name}" and a "View on dashboard" button deep-linking to FEAT-12).**

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {client_name} | Client -- name | Riley Chen | Never empty -- required at client creation |
| {service_name} | Service -- name | Full Set Lashes | Never empty (required field) |
| {appointment_date} / {appointment_time} | Booking -- start_time | Oct 4, 2026 / 2:30 PM | Never empty -- fixed at booking |
| {original_appointment_date} / {new_appointment_date} / {new_appointment_time} | Booking -- start_time before/after a reschedule | Oct 4, 2026 / Oct 11, 2026 / 3:00 PM | Never empty when the variant is a reschedule notice |
| {deposit_amount} | Booking -- deposit_amount | $40.00 | Never empty -- fixed at booking |
| {deposit_outcome_summary} | Derived -- Deposit Transaction.status rendered in plain language ("refunded" / "kept per policy" / "carried over") | refunded | Renders "pending" if the outcome has not yet been determined at notification time -- never left blank |

## Delivery Rules

**Batching:** Notifications for the same Pro across different bookings are not batched -- each booking event is its own notice, since a Pro benefits from knowing immediately which specific booking changed rather than waiting for a digest. Multiple client-initiated changes to the same booking in quick succession (e.g., a reschedule immediately followed by a cancellation) each produce their own notice, in the order they occur.
**Deduplication:** At most one notification per triggering event per channel. A booking confirmation event and a cancellation event on the same booking are distinct events and both produce their own notice; the same single event is never re-delivered on retry beyond the retry rule below.
**Retry on failure:** Governed by FEAT-08.SPEC-009 for the text channel specifically: a failed text is retried once, then falls back to email, with the gap also visible in-app on the dashboard's attention list (since in-app is never itself the failing channel). Email delivery failure for this notification type is not separately retried beyond the underlying email capability's own delivery attempt (FEAT-08.SPEC-013); the in-app copy remains the surviving record regardless.
**Expiry:** None -- a booking-activity notice remains relevant however late it arrives, since it reports a fact about the Pro's own schedule that stays true; the in-app copy also persists indefinitely as part of the Pro's dashboard/activity history (FEAT-16), so there is no "too late to matter" cutoff.

## Edge Cases

- **The Pro has all notification channels other than in-app turned off** -- She still sees the in-app notice on her dashboard; nothing about her schedule ever depends solely on a channel she has muted.
- **A client cancels and reschedules the same booking within seconds of each other** -- Both events produce their own notice, delivered in the order the underlying Booking transitions committed, never merged into one ambiguous message.
- **The Pro is mid-way through viewing her dashboard when a new booking arrives** -- The in-app notice appears without requiring a manual refresh, consistent with FEAT-12's live-updating nature; text/email notices, if enabled, arrive independently on their own channel.
- **Two client actions on two different bookings occur at effectively the same time** -- Each produces its own independent notice; the notifications are not merged across bookings even though they land close together in time.
- **The Pro's notification_preferences are changed between the triggering event and delivery** -- Per this spec's channel-decision rule (evaluated at delivery time, consistent with the notification methodology's default), a preference change that lands before the message actually dispatches is honored; the in-app copy is unaffected either way since it is never optional.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-002 (Deposit Capture & Booking Confirmation) | Triggered by (inbound) | A new confirmed booking fires this notification |
| FEAT-10.SPEC-004 (Booking Update Commit) / FEAT-10.SPEC-006 (Client-Initiated Cancel/Reschedule) | Triggered by (inbound) | A client cancellation or reschedule fires this notification; FEAT-10.SPEC-006 is the trigger-and-audience contract, this spec owns the message content |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) / FEAT-30.SPEC-012 (Pro Booking Management notifications) | Triggered by (inbound) | FEAT-30.SPEC-012 is the trigger-and-audience contract for Pro-side booking activity; this spec owns the message content the Pro receives |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (inbound) | Supplies the Pro's notification_preferences that govern text/email channel selection |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (outbound) | Every CTA deep-links to the affected booking on the dashboard |
| FEAT-08.SPEC-012 / FEAT-08.SPEC-013 (Text / Email Capabilities) | Triggers (outbound) | Perform the text/email sends when those channels are enabled |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback for the text channel |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every send is written to the append-only activity record |

## Analytics and Success Signals

- **pro_notification_sent** (type: new_booking / client_cancel / client_reschedule; channels: in_app / text / email) -- N/A -- no Stage 2 metric directly measures Pro notification delivery; this event supports the operational goal named in the feature's rationale (the Pro learns of changes without opening the dashboard) rather than a named success-metrics.md target.
- **pro_notification_cta_tapped** (destination: dashboard_booking_row) -- supports success-metrics.md: "Daily Dashboard Glance Speed"

## Acceptance Criteria

**FEAT-08.SPEC-005-AC-01:** Given Talia has notification_preferences set to "In-app + text" (the default), when a new client books, then she receives both an in-app notice and a text naming the client, service, and time.

**FEAT-08.SPEC-005-AC-02:** Given Talia has set notification_preferences to "In-app only", when Riley cancels her booking, then Talia sees the in-app notice and receives no text or email.

**FEAT-08.SPEC-005-AC-03:** Given Talia has notification_preferences set to "In-app + text + email", when Riley reschedules her booking, then Talia receives the reschedule notice on all three channels with the original and new appointment times.

**FEAT-08.SPEC-005-AC-04:** Given Riley cancels her booking inside the cancellation window, when Talia's notification is composed, then it shows the deposit outcome as "kept per policy".

**FEAT-08.SPEC-005-AC-05:** Given Riley cancels her booking outside the window, when Talia's notification is composed, then it shows the deposit outcome as "refunded".

**FEAT-08.SPEC-005-AC-06:** Given Riley reschedules and then cancels the same booking within seconds, when both events process, then Talia receives two separate notices, in the order the transitions committed.

**FEAT-08.SPEC-005-AC-07:** Given a text notification to Talia fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Talia still receives the notice by email, and the in-app copy remains visible regardless.

**FEAT-08.SPEC-005-AC-08:** Given Talia is actively viewing her dashboard, when a new booking arrives, then the in-app notice appears without her needing to manually refresh.

**FEAT-08.SPEC-005-AC-09:** Given two different clients each cancel a different booking at effectively the same moment, when both events process, then Talia receives two independent notices, one per booking.

**FEAT-08.SPEC-005-AC-10:** Given Talia taps the "View booking" CTA on a new-booking notice, when the tap registers, then she is taken to that booking's row on FEAT-12's dashboard.

**FEAT-08.SPEC-005-AC-11:** Given Talia has never changed her notification_preferences, when her account is created, then her default is "In-app + text".

**FEAT-08.SPEC-005-AC-12:** Given Support is troubleshooting a delivery issue for one of Talia's notifications, when Support views the Message record, then Support sees delivery status only, never the notification's rendered content.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 3 (in-app, text, email) | 3 |
| Trigger Paths | 3 (new booking, client cancel, client reschedule) | 3 |
| Preference States | 4 (in-app only, +text, +email, +text+email) | 4 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
