---
document_type: spec
spec_type: notification
spec_id: FEAT-21.SPEC-009
spec_name: Occurrence Deposit Lifecycle Notification
spec_slug: occurrence-deposit-lifecycle-notification
parent_feature: FEAT-21
parent_feature_name: Recurring/Standing Appointments
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Notification Spec: Occurrence Deposit Lifecycle Notification

## Overview

**Name:** Occurrence Deposit Lifecycle Notification
**ID:** FEAT-21.SPEC-009
**Type:** Notification
**Purpose:** Sends the client her occurrence's own fresh deposit link about a week before it, and, separately, tells both the client and the Pro when an unpaid occurrence is released.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- The deposit-request variant, sent to the client, with the fresh deposit link for one occurrence
- The release variant, sent to both the client and the Pro, when an occurrence's deposit is unpaid by its cancellation cut-off
- Delivery, retry, and expiry behavior for both variants

**Non-Goals:**
- Deciding when to send the request or when to release the occurrence -- owned by FEAT-21.SPEC-005 (Occurrence Deposit Request & Release), which this spec's two variants are triggered by.
- Capturing the deposit payment itself -- owned by FEAT-07 (Deposit Payment at Booking); this spec only carries the link, it never processes the payment.
- Confirming the occurrence is scheduled -- owned by FEAT-21.SPEC-007 (Occurrence Generated Notification), a separate, earlier communication.
- Deciding text vs. email for either variant -- owned by FEAT-14.SPEC-007 (Textability Determination Rule).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The recipient has active Messaging Consent for texting (FEAT-14.SPEC-007) | This Brief's Primary Flows and Validation & Limits fields name both messages directly ("paid through a deposit link sent about a week before," "both the client and the Pro are told"); text matches the product's default channel for money-related, time-sensitive messages |
| Email | The recipient has not granted texting consent (FEAT-14.SPEC-007) | Every client who declines texting still supplies an email at booking; the Pro Account always carries a sign-in email as a channel of record |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An occurrence reaches its deposit-request point | FEAT-21.SPEC-005 (Occurrence Deposit Request & Release) | Fires once per occurrence, when the deposit-request send is initiated | Booking (service, start_time, deposit_amount), the deposit-request link, Pro Account (display_name) |
| An occurrence is released unpaid at its cancellation cut-off | FEAT-21.SPEC-005 (Occurrence Deposit Request & Release) | Fires once per occurrence, when the release transition completes | Booking (service, start_time), Pro Account (display_name) |

## Audience and Preferences

**Recipients:** The deposit-request variant goes to the Client tied to the occurrence's Booking only (Access Matrix: Recurring Appointments = Own-only for the Client), since only she can pay it. The release variant goes to both the Client and the Pro (Recurring Appointments = Own-only for the Client, Full for the Pro on series tied to her own schedule), per this Brief's Validation & Limits field: "both the client and the Pro are told."

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Client texting consent (governs the client's channel) | Granted / Revoked | Captured at booking | FEAT-06 (Client Booking Identity) at booking; changed via FEAT-14 (Messaging Consent Management) |
| Pro notification preferences (governs the Pro's channel for the release variant) | In-app / text / email, per FEAT-27's notification_preferences field | Set during onboarding | FEAT-27 (Pro Profile & Booking Page Settings) |

Neither variant carries an on/off toggle for the recipients named above: the deposit-request is the mechanism by which the occurrence's own money is collected, and the release notice reports money that was never collected -- both are treated the same as any other transactional payment communication in this product, never a discretionary reminder.

**Quiet Hours:** Per this Brief's recorded reading, both variants are ordinary reminder-class communications with no short-window urgency, so both follow the platform's roughly 8am-9pm daytime-hours rule (XBR-16, platform parameter: `reminder-window-start-hour` to platform parameter: `reminder-window-end-hour`, in the Pro's timezone) for the client's copy; the Pro's release notice follows her own in-app/text/email preference and is not additionally gated by the daytime window, consistent with how other Pro attention alerts (FEAT-08.SPEC-006) are delivered.

## Content Definition

**Deposit-request variant -- Text (to Client):**
- **Body:** Your next appointment with {pro_display_name} ({service_name}, {occurrence_date} at {occurrence_time}) needs its deposit. Pay here: {deposit_link}
- **CTA:** {deposit_link} -- deep-links to FEAT-07 (Deposit Payment at Booking) for this one occurrence

**Deposit-request variant -- Email (to Client):**
- **Subject:** Deposit needed for your upcoming appointment with {pro_display_name}
- **Body:**
  Hi {client_first_name},

  Your next standing appointment is coming up:

  Service: {service_name}
  Date & time: {occurrence_date} at {occurrence_time} ({timezone})
  Deposit due: {deposit_amount}

  Pay your deposit using the link below to hold this time.
- **CTA (button):** Pay my deposit -- deep-links to FEAT-07 (Deposit Payment at Booking) for this one occurrence

**Release variant -- Text (to Client):**
- **Body:** Your appointment with {pro_display_name} on {occurrence_date} wasn't held because the deposit wasn't paid in time. Your series continues -- {pro_display_name} will reach out or you can rebook.
- **CTA:** None -- this is a status report, not an action the client can take on this occurrence; her next occurrence generates normally on the series' own schedule.

**Release variant -- Email (to Client):**
- **Subject:** Your {occurrence_date} appointment with {pro_display_name} was released
- **Body:**
  Hi {client_first_name},

  Your {occurrence_date} appointment with {pro_display_name} wasn't held because the deposit wasn't paid before the cancellation cut-off. Your standing series continues as normal, and your next visit will be scheduled in its usual turn.
- **CTA (button):** None.

**Release variant -- Text/Email (to Pro):**
- **Body:** {client_name}'s {occurrence_date} standing appointment ({service_name}) was released -- the deposit wasn't paid in time.
- **CTA:** View series -- deep-links to FEAT-21.SPEC-010 (Pro Recurring Series Management) for this client's series

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | Never empty -- required (Pro Account entity) |
| {service_name} | Service -- name | Full Set Lashes | Never empty -- required at service creation (FEAT-01) |
| {occurrence_date} / {occurrence_time} | Booking (occurrence) -- start_time, rendered in the Pro's timezone | Nov 12, 2026 / 2:30 PM | Never empty -- set at generation (FEAT-21.SPEC-004) |
| {timezone} | Pro Account -- timezone | Eastern Time | Never empty -- required per account (XBR-25) |
| {deposit_amount} | Booking (occurrence) -- deposit_amount | $40.00 | Never empty -- computed at generation from the Service's rule (XBR-05) |
| {deposit_link} | Access Link -- scoped to this occurrence's deposit payment, issued via FEAT-07's deposit-request mechanism | chairtime.app/m/4d8a1f | If link issuance fails, the send is held and retried per FEAT-08.SPEC-009's failure handling -- it is never sent without its link |
| {client_first_name} / {client_name} | Client -- name (first token) / full name | Riley / Riley Chen | Renders the full name field if no separable first token exists |

## Delivery Rules

**Batching:** None -- each variant reports on exactly one occurrence, and each occurrence generates at most one deposit-request send and, separately, at most one release. Multiple occurrences from the same client's series never arrive at this point together, since occurrences generate and resolve on the series' own successive schedule.
**Deduplication:** At most one deposit-request send per occurrence, guarded by the deposit-request-sent flag FEAT-21.SPEC-005 records; at most one release notice per occurrence, since a Booking transitions to Expired at most once and cannot be released a second time.
**Retry on failure:** Governed by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback): a failed text is retried once, then falls back to email, and the delivery gap is flagged on the Pro's dashboard (XBR-17) for the client-facing sends; the Pro's own release notice follows the same retry-then-fallback discipline across her configured channels.
**Expiry:** The deposit-request variant does not expire on its own -- it remains valid and payable up to the occurrence's cancellation cut-off, at which point FEAT-21.SPEC-005's release action supersedes it and this spec's release variant fires instead. The release variant, once sent, never expires -- it is a permanent record of what happened.

## Edge Cases

- **The client pays the deposit in the moments after the request notification was sent but before she opens it** -- No further action is taken by this spec; she simply proceeds to FEAT-07's confirmation, and no release notice is ever sent for that occurrence.
- **The occurrence is cancelled (FEAT-21.SPEC-006) after the deposit-request notice was sent but before the cancellation cut-off** -- No release notice is sent, since FEAT-21.SPEC-005's monitoring stops on cancellation; the client's earlier deposit-request notice simply becomes moot.
- **The client has both texting consent and an email on file** -- Text is used for her copy of either variant; no duplicate email is also sent.
- **The Pro's release notice arrives while she is mid-appointment with another client** -- It is delivered on whatever channel her notification_preferences specify and waits in-app until she next checks, exactly like any other Pro attention alert; it is never re-sent solely because she has not yet seen it.
- **A deposit-request send occurs at 11pm in the Pro's timezone** -- The client's copy is held until the daytime window next opens; the release variant, when it later fires, is likewise held on the client's side but not additionally gated for the Pro.
- **The client's phone number changed since her last booking and fresh consent has not yet been captured** -- Per XBR-15, her copy of either variant routes to email, never to the old or unconsented number.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-005 (Occurrence Deposit Request & Release) | Triggered by (inbound) | The request and release moments each fire this notification's matching variant |
| FEAT-07 (Deposit Payment at Booking) | Navigation (outbound) | The deposit-request variant's CTA deep-links here |
| FEAT-14.SPEC-007 (Textability Determination Rule) | References (inbound) | Decides text vs. email for the client's copy of either variant |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (inbound) | Supplies the Pro's notification_preferences for the release variant's Pro copy |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Triggers (outbound) | Performs the text send when text is the chosen channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | Triggers (outbound) | Performs the email send when email is chosen or is the fallback |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback behavior if either send fails |
| FEAT-21.SPEC-010 (Pro Recurring Series Management) | Navigation (outbound) | The release variant's Pro-facing CTA deep-links here |

## Analytics and Success Signals

- **recurring_occurrence_deposit_request_notification_sent** (channel: text / email) -- supports success-metrics.md: "Deposit Capture Rate" (this notice is the mechanism by which the client is invited to complete the capture this metric measures)
- **recurring_occurrence_release_notification_sent** (recipient: client / pro; channel) -- supports success-metrics.md: "Automatic Refund Correctness" (both parties being told, without either having to chase the outcome, is exactly the "completes without either person having to chase it" standard this metric names)
- **recurring_occurrence_deposit_request_cta_tapped** (channel) -- supports success-metrics.md: "Deposit Capture Rate"

## Acceptance Criteria

**FEAT-21.SPEC-009-AC-01:** Given Riley's occurrence reaches its deposit-request point and she has active texting consent, when the request fires, then she receives a text with the deposit amount and a payment link.

**FEAT-21.SPEC-009-AC-02:** Given Riley declined texting and provided an email, when her occurrence's deposit-request fires, then she receives the same content by email instead.

**FEAT-21.SPEC-009-AC-03:** Given Riley taps her deposit link, when the tap registers, then it opens FEAT-07's deposit payment flow for that one occurrence.

**FEAT-21.SPEC-009-AC-04:** Given Riley's occurrence is released unpaid at its cancellation cut-off, when the release completes, then both Riley and Talia are notified.

**FEAT-21.SPEC-009-AC-05:** Given Riley receives the release notice, then it states that her series continues and that her next visit will be scheduled in its usual turn.

**FEAT-21.SPEC-009-AC-06:** Given Talia receives the release notice, then it names the client and the occurrence date and links to her own series view (FEAT-21.SPEC-010).

**FEAT-21.SPEC-009-AC-07:** Given Riley pays her deposit before opening the request notification, when payment completes, then no release notice is ever sent for that occurrence.

**FEAT-21.SPEC-009-AC-08:** Given Riley's occurrence is cancelled after the deposit-request notice was sent but before the cancellation cut-off, when the cancellation completes, then no release notice is sent.

**FEAT-21.SPEC-009-AC-09:** Given Riley has both texting consent and an email on file, when either variant sends to her, then it arrives once, by text, with no duplicate email also sent.

**FEAT-21.SPEC-009-AC-10:** Given Talia's notification_preferences specify email for Pro notifications, when the release variant fires for her, then she receives it by email, not text or in-app alone.

**FEAT-21.SPEC-009-AC-11:** Given a deposit-request send occurs at 11pm in the Pro's timezone, when the send is attempted, then Riley's copy is held until the daytime window next opens.

**FEAT-21.SPEC-009-AC-12:** Given Riley's phone number changed and fresh consent has not been captured, when either variant fires for her, then it is sent by email, never to the unconsented number.

**FEAT-21.SPEC-009-AC-13:** Given a text send to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then she still receives the notification by email, and the delivery gap is flagged on Talia's dashboard.

**FEAT-21.SPEC-009-AC-14:** Given a Booking can transition to Expired at most once, when the release trigger is evaluated for an occurrence, then at most one release notice is ever sent for it.

**FEAT-21.SPEC-009-AC-15:** Given Riley's deposit-request-sent flag is already set for an occurrence (FEAT-21.SPEC-005), when a second evaluation runs before the first send completes, then no duplicate deposit-request notification is sent.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 2 | 2 |
| Preference States | 3 (client consent granted, client consent revoked/declined, Pro channel preference) | 3 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |
