---
document_type: spec
spec_type: notification
spec_id: FEAT-08.SPEC-006
spec_name: Pro Attention Alert
spec_slug: pro-attention-alert
parent_feature: FEAT-08
parent_feature_name: Automated Booking Messaging
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 16
---

# Notification Spec: Pro Attention Alert

## Overview

**Name:** Pro Attention Alert
**ID:** FEAT-08.SPEC-006
**Type:** Notification
**Purpose:** Tells the Pro immediately when something needs her attention -- a message delivery failure, a calendar connection needing reconnection, a refund that failed to complete, a card-issuer dispute, a reminder that failed to schedule, or a deposit request that expired unpaid -- so nothing about her business is ever a silent failure she discovers late.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- The six named attention conditions: message delivery failure, calendar reconnection needed, refund failure, card-issuer dispute opened, reminder-scheduling failure, deposit request expired unpaid
- The Pro-facing expiry notice content for an unpaid deposit request: this spec is the content owner, FEAT-03.SPEC-007 is the trigger, and FEAT-30.SPEC-013 references this spec rather than defining duplicate content
- In-app, text, and email delivery per the Pro's notification_preferences

**Non-Goals:**
- Routine booking activity (new bookings, client cancellations/reschedules) -- owned by FEAT-08.SPEC-005 (Pro Booking Activity Notification); this spec is reserved for conditions requiring action, not routine updates.
- Diagnosing or resolving the underlying condition (why the calendar needs reconnecting, why a refund failed, why a reminder failed to schedule) -- owned by the feature or spec that owns each condition (FEAT-04, FEAT-09/FEAT-30, FEAT-16, FEAT-08.SPEC-007, FEAT-03.SPEC-007); this spec only alerts, per its Trigger section.
- Deciding a card-issuer dispute -- excluded per scope-boundaries.md (SC-17): Chairtime never rules on the dispute; this spec only tells the Pro one exists and where to find the evidence.
- The client-facing side of a refund failure -- covered separately by FEAT-08.SPEC-004, which shows the client only the reassuring "in progress" wording, never the failure itself.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, regardless of the Pro's text/email preference | These are the dashboard's "attention list" items (product-features.md, FEAT-12); the dashboard is the canonical place a Pro checks for anything needing action |
| Text | The Pro's notification_preferences include text for this notification type | An attention-worthy condition is time-sensitive; a text reaches Talia immediately even when she is away from the app |
| Email | The Pro's notification_preferences include email for this notification type | Gives the Pro a durable record of attention items she can search or forward if needed (e.g., for a dispute) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A text or email message fails delivery on all channels attempted | FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | Fires after the retry-then-fallback sequence itself cannot deliver the message on any channel | Booking reference, recipient (client), message type |
| Calendar connection needs reconnecting | FEAT-04 (Two-Way Calendar Sync) | Fires when the connection status changes to Needs Reconnection | Calendar connection status, last_successful_sync |
| An automatic or goodwill refund fails to complete | FEAT-09 (Cancellation & No-Show Policy Engine) / FEAT-30 (Pro Booking Management) | Fires when a refund attempt does not succeed and enters a retry state | Deposit Transaction (status, outcome_reason), Booking reference |
| A card-issuer dispute is opened | FEAT-16 (Booking & Payment Activity Record) | Fires when a dispute notice is received for a Deposit Transaction | Booking reference, Deposit Transaction (Disputed status) |
| A Pro-created deposit request's hold expires with the deposit never paid | FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration) | Fires when FEAT-03.SPEC-007 transitions the Booking to Expired (unpaid) and the hold's computed expiry has passed with the deposit never paid; FEAT-03.SPEC-007 is the sole writer of that transition and is the trigger only | Booking reference, Booking (service, start_time), Client (name) |
| A confirmed Booking's reminder-scheduling computation cannot run | FEAT-08.SPEC-007 (Reminder Scheduling & Timing Window Enforcement) | Fires when that automation cannot compute or store a reminder_send_time for a confirmed Booking (e.g., timezone data unavailable at confirmation time) | Booking reference, Booking (start_time, service) |

## Audience and Preferences

**Recipients:** The Pro (Talia) whose account the condition affects (Access Matrix: Booking & Payment, Payouts, Activity Record & Insights = Full/View for the Pro). Platform Operator (Support) has View-only access to delivery status and to the dispute evidence itself (via FEAT-16), consistent with the Access Matrix, but is never a recipient of this notification.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Attention alert channels | In-app only / In-app + text / In-app + email / In-app + text + email | In-app + text | FEAT-27 (Pro Profile & Booking Page Settings) |

In-app is never disable-able, matching FEAT-08.SPEC-005's treatment -- an attention item always appears on the dashboard's attention list regardless of the Pro's text/email choice, since the dashboard is the guaranteed floor for anything needing her action.

**Quiet Hours:** N/A -- product-features.md defines no quiet-hours window for Pro-facing alerts, and an attention-worthy condition (a failed refund, a dispute, a delivery gap) is exactly the kind of time-sensitive information a quiet-hours delay would work against; it is delivered as soon as it is known, on every enabled channel.

## Content Definition

**In-app (message delivery failure):**
- **Title:** Message didn't get through
- **Body:** {client_name}'s {message_type} couldn't be delivered by text, so we sent it by email instead.
- **CTA:** View details -- deep-links to FEAT-12 (Pro Daily Schedule Dashboard), attention list

**In-app (calendar reconnection needed):**
- **Title:** Reconnect your calendar
- **Body:** Your calendar connection needs to be reconnected so new bookings keep syncing.
- **CTA:** Reconnect -- deep-links to FEAT-04

**In-app (refund failure):**
- **Title:** A refund needs your attention
- **Body:** The {deposit_amount} refund for {client_name}'s {appointment_date} booking couldn't complete automatically. We're retrying it.
- **CTA:** View details -- deep-links to FEAT-12, attention list

**In-app (dispute opened):**
- **Title:** A charge is being disputed
- **Body:** {client_name}'s card issuer has opened a dispute on their {appointment_date} deposit. Review the booking timeline to respond.
- **CTA:** View timeline -- deep-links to FEAT-16

**In-app (reminder-scheduling failure):**
- **Title:** A reminder couldn't be scheduled
- **Body:** We couldn't schedule the automatic reminder for {client_name}'s {appointment_date} appointment. You may want to reach out directly before the visit.
- **CTA:** View details -- deep-links to FEAT-12, attention list

**In-app (deposit request expired unpaid):**
- **Title:** Deposit request expired
- **Body:** {client_name}'s deposit request for {appointment_date} expired unpaid. The slot has been released.
- **CTA:** View schedule -- deep-links to FEAT-12 (Pro Daily Schedule Dashboard)

**Text (each condition, one line, matching the in-app body without the CTA link text spelled out as a button):**
- **Body (message delivery failure):** {client_name}'s message couldn't be delivered by text -- sent by email instead. View: {dashboard_link}
- **Body (calendar reconnection):** Your calendar connection needs reconnecting so bookings keep syncing. Reconnect: {calendar_link}
- **Body (refund failure):** A {deposit_amount} refund for {client_name} couldn't complete automatically -- we're retrying it. View: {dashboard_link}
- **Body (dispute opened):** {client_name}'s card issuer opened a dispute on their deposit. View: {timeline_link}
- **Body (reminder-scheduling failure):** We couldn't schedule the reminder for {client_name}'s {appointment_date} appointment. View: {dashboard_link}
- **Body (deposit request expired):** {client_name}'s deposit request for {appointment_date} expired unpaid. The slot has been released. View: {dashboard_link}

**Email (mirrors each text variant with subject "Needs your attention: {condition_summary}" and a button matching the in-app CTA).**

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {client_name} | Client -- name | Riley Chen | Never empty (required field); N/A for the calendar-reconnection variant, which names no client |
| {message_type} | Derived -- the failed Message's type (confirmation / reminder / change notice) | reminder | Never empty -- every failed Message has a type |
| {deposit_amount} | Deposit Transaction -- amount | $40.00 | Never empty -- fixed once at booking |
| {appointment_date} | Booking -- start_time | Oct 4, 2026 | Never empty -- fixed at booking |
| {dashboard_link} / {calendar_link} / {timeline_link} | Derived -- in-product navigation targets to FEAT-12, FEAT-04, and FEAT-16 respectively | (in-product link) | Never empty -- these are fixed navigation destinations, not per-record generated links |
| {condition_summary} | Derived -- a short label per condition ("message delivery", "calendar reconnection", "refund", "dispute", "reminder scheduling", "deposit request expired") | refund | Never empty -- one of exactly six fixed values |

## Delivery Rules

**Batching:** Not batched by default -- each attention condition is its own alert, since each names a distinct action the Pro may need to take. If the same underlying calendar-reconnection condition would otherwise re-alert repeatedly while unresolved, only one active alert exists per condition instance at a time (see Deduplication) rather than a recurring stream.
**Deduplication:** At most one active alert per open condition. A calendar connection that remains in Needs Reconnection state does not re-alert on every subsequent sync attempt -- the existing unresolved in-app item stands until the Pro reconnects (FEAT-04) or the condition otherwise clears. A refund retry that fails again produces no new alert beyond the first, since the same in-app item already reflects "we're retrying it" until it resolves.
**Retry on failure:** This notification's own text/email delivery is retried once and falls back to email, per FEAT-08.SPEC-009, the same as every other message this feature sends -- with one exception: because the in-app channel is always active and never itself the failing channel, an attention alert about a *different* condition (e.g., a refund failure) is never left with no surviving delivery even if its own text/email attempt also fails.
**Expiry:** None -- an attention item never expires undelivered in the sense of being dropped; it remains on the dashboard's attention list until the Pro resolves or acknowledges the underlying condition (reconnects the calendar, the refund retry succeeds, the dispute is addressed).

## Edge Cases

- **The same client has two separate reminders fail delivery on the same day** -- Each failure is a distinct Message and produces its own alert, since each concerns a different booking's delivery gap.
- **A refund failure alert is still open when the retry later succeeds** -- The open attention item is cleared from the dashboard's attention list once the Deposit Transaction reaches Refunded; no further alert fires for that same refund attempt.
- **The Pro has all channels other than in-app disabled** -- She still sees every attention item on her dashboard; nothing needing her action ever depends solely on a muted channel.
- **A dispute is opened on a booking whose client has since been deleted (FEAT-13)** -- The alert still fires, using the de-identified financial record retained for exactly this purpose (XBR-19); the alert states the booking and dispute details without a client name, since the identifying contact details no longer exist.
- **Two different attention conditions occur for the same Pro at effectively the same time (e.g., a delivery failure and a refund failure)** -- Each produces its own independent alert; the two are never merged, since they name different required actions.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | Triggered by (inbound) | A final delivery failure after retry-then-fallback fires this alert |
| FEAT-04 (Two-Way Calendar Sync) | Triggered by (inbound) | A lapsed connection fires this alert |
| FEAT-09 (Cancellation & No-Show Policy Engine) | Triggered by (inbound) | An automatic refund failure fires this alert |
| FEAT-30 (Pro Booking Management) | Triggered by (inbound) | A goodwill refund failure fires this alert |
| FEAT-16 (Booking & Payment Activity Record) | Triggered by (inbound) | A card-issuer dispute notice fires this alert; also where the dispute CTA leads |
| FEAT-08.SPEC-007 (Reminder Scheduling & Timing Window Enforcement) | Triggered by (inbound) | A failure to compute or store a reminder_send_time for a confirmed Booking fires this alert |
| FEAT-03.SPEC-007 (Pro-Created Deposit Request Hold & Expiration) | Triggered by (inbound) | An unpaid deposit-request hold expiring fires the expiry alert; this spec owns the content |
| FEAT-30.SPEC-013 (Deposit Request & Expiry Notice) | References (outbound) | References this spec for the Pro's expiry notice content instead of defining its own |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every alert send is written to the append-only activity record |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (inbound) | Supplies the Pro's notification_preferences |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (outbound) | Every non-dispute CTA deep-links to the dashboard's attention list |
| FEAT-08.SPEC-012 / FEAT-08.SPEC-013 (Text / Email Capabilities) | Triggers (outbound) | Perform the text/email sends when enabled |

## Analytics and Success Signals

- **pro_attention_alert_sent** (condition: message_delivery_failure / calendar_reconnection / refund_failure / dispute_opened; channels) -- supports success-metrics.md: "Automatic Refund Correctness"
- **pro_attention_alert_sent** (condition: calendar_reconnection) -- supports success-metrics.md: "Calendar Sync Reliability"
- **pro_attention_alert_resolved** (condition, time_to_resolve) -- N/A -- no Stage 2 metric measures time-to-resolution for attention items; retained to make the product's "never silently dropped" commitment (XBR-17, XBR-10) observable end to end.
- **pro_attention_alert_sent** (condition: reminder_scheduling_failure) -- supports success-metrics.md: "Reminder Response Rate" -- a reminder that never got scheduled is visible here rather than only showing up later as a gap in that metric's numerator.
- **pro_attention_alert_sent** (condition: deposit_request_expired; channels) -- supports success-metrics.md: "Pro Change Correctness"

## Acceptance Criteria

**FEAT-08.SPEC-006-AC-01:** Given Riley's reminder text fails delivery and the email fallback is also exhausted per FEAT-08.SPEC-009, when the final failure occurs, then Talia receives an in-app alert "Message didn't get through" naming Riley and the message type.

**FEAT-08.SPEC-006-AC-02:** Given Talia's calendar connection lapses into Needs Reconnection, when the status changes, then she receives an alert on every enabled channel with a "Reconnect" CTA to FEAT-04.

**FEAT-08.SPEC-006-AC-03:** Given an automatic refund for one of Talia's clients fails to complete, when FEAT-09 flags it, then Talia receives an alert stating the refund couldn't complete automatically and that it is being retried.

**FEAT-08.SPEC-006-AC-04:** Given a card-issuer dispute is opened on one of Talia's bookings, when FEAT-16 records the dispute, then Talia receives an alert with a "View timeline" CTA to FEAT-16.

**FEAT-08.SPEC-006-AC-05:** Given Talia's calendar connection remains in Needs Reconnection for several days, when subsequent sync attempts continue to fail, then no additional alert fires beyond the original unresolved item on her dashboard.

**FEAT-08.SPEC-006-AC-06:** Given a previously failed refund retry later succeeds, when the Deposit Transaction reaches Refunded, then the open attention item clears from Talia's dashboard and no further alert fires for that refund.

**FEAT-08.SPEC-006-AC-07:** Given Talia has set notification_preferences to "In-app only", when a dispute is opened, then she sees the in-app alert and receives no text or email.

**FEAT-08.SPEC-006-AC-08:** Given a dispute is opened on a booking whose client record has since been deleted, when the alert fires, then it names the booking and dispute details without a client name, using the retained de-identified financial record.

**FEAT-08.SPEC-006-AC-09:** Given Talia experiences both a delivery failure and a refund failure at effectively the same time, when both conditions are detected, then she receives two separate, independent alerts.

**FEAT-08.SPEC-006-AC-10:** Given this alert's own text send fails, when FEAT-08.SPEC-009's retry-then-fallback runs, then Talia still receives the alert by email, and the in-app copy is unaffected regardless.

**FEAT-08.SPEC-006-AC-11:** Given Riley has two separate reminders fail delivery on the same day for two different bookings, when both failures occur, then Talia receives two distinct alerts, one per booking.

**FEAT-08.SPEC-006-AC-12:** Given Talia taps "View details" on a refund-failure alert, when the tap registers, then she is taken to the attention list on her dashboard (FEAT-12).

**FEAT-08.SPEC-006-AC-13:** Given Platform Operator (Support) is assisting Talia with a dispute, when Support views the booking's activity record, then Support sees the same dispute evidence Talia sees, per Support's View access under XBR-24, and this alert itself is never sent to Support.

**FEAT-08.SPEC-006-AC-14:** Given FEAT-08.SPEC-007's reminder-scheduling computation fails to run for one of Talia's confirmed bookings, when the failure is detected, then Talia receives an attention alert naming the client and appointment date, with a "View details" CTA to her dashboard's attention list.

**FEAT-08.SPEC-006-AC-15:** Given a Pro-created deposit request's hold expires unpaid and FEAT-03.SPEC-007 marks the Booking Expired (unpaid), when the trigger reaches this spec, then Talia receives the in-app alert "Deposit request expired" naming the client and appointment date, plus text and email per her notification_preferences, with a "View schedule" CTA to FEAT-12.

**FEAT-08.SPEC-006-AC-16:** Given Talia has set notification_preferences to "In-app only" and a deposit request expires unpaid, when the alert fires, then she sees the in-app alert and receives no text or email, and Riley receives no message about the expiry from this spec.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 3 (in-app, text, email) | 3 |
| Trigger Paths | 6 (delivery failure, calendar reconnection, refund failure, dispute, reminder-scheduling failure, deposit request expired) | 6 |
| Preference States | 4 (in-app only, +text, +email, +text+email) | 4 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
