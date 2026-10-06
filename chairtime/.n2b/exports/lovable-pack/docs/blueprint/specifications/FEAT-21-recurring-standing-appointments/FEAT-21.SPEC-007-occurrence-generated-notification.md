---
document_type: spec
spec_type: notification
spec_id: FEAT-21.SPEC-007
spec_name: Occurrence Generated Notification
spec_slug: occurrence-generated-notification
parent_feature: FEAT-21
parent_feature_name: Recurring/Standing Appointments
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Notification Spec: Occurrence Generated Notification

## Overview

**Name:** Occurrence Generated Notification
**ID:** FEAT-21.SPEC-007
**Type:** Notification
**Purpose:** Confirms to the client, each time a standing appointment's next occurrence is generated, exactly which appointment has just been scheduled from her series.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- The confirmation sent when a new occurrence is successfully generated, on both of the product's client channels (text and email)
- The variant sent when a previously conflicted occurrence's replacement time is accepted and the occurrence proceeds
- Delivery, retry, and expiry behavior for this notification

**Non-Goals:**
- Notifying the client that an occurrence needs a new time in the first place -- owned by FEAT-21.SPEC-008 (Occurrence Time Change Advance Notice); this spec covers only a successfully scheduled occurrence.
- The deposit-request link and its own notice -- owned by FEAT-21.SPEC-009 (Occurrence Deposit Lifecycle Notification); this is a separate communication sent later, closer to the appointment.
- Deciding text vs. email for this send -- owned by FEAT-14.SPEC-007 (Textability Determination Rule), which this spec defers to before every send, per XBR-15.
- Notifying the Pro that an occurrence was generated -- product-features.md's Communications field for this feature names only a client-facing confirmation; the Pro sees generated occurrences on her own schedule view and on FEAT-21.SPEC-010 (Pro Recurring Series Management) without a separate push notification.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active Messaging Consent for texting (FEAT-14.SPEC-007) | This Brief's Communications field names this as an ordinary confirmation message, matching the product's default client channel for booking-related confirmations |
| Email | The client has not granted texting consent (FEAT-14.SPEC-007) | Every client who declines texting still supplies an email at booking, so the confirmation is never simply undeliverable |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An occurrence's Booking is generated normally | FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Fires once per occurrence, immediately when generation succeeds without a slot conflict | Booking (service, start_time), Pro Account (display_name, studio_address, timezone), Recurring Series (interval) |
| A previously conflicted occurrence's replacement time is accepted | FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Fires once, when a client's submitted replacement time for a flagged occurrence passes validation | Booking (service, start_time -- the newly set replacement time), Pro Account (display_name, studio_address, timezone) |

## Audience and Preferences

**Recipients:** The Client tied to the occurrence's Booking (Access Matrix: Recurring Appointments = Own-only for the Client) -- the sole recipient, since this discloses that client's own standing-appointment schedule.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (governs channel, not whether this message sends) | Granted / Revoked | Captured at booking | FEAT-06 (Client Booking Identity) at booking; changed via FEAT-14 (Messaging Consent Management) |

This confirmation carries no separate on/off toggle: it is the direct outcome of the client's own standing-appointment choice, not a discretionary reminder. A client cannot opt out of being told her next occurrence has been scheduled -- only the channel it arrives on varies, per FEAT-14.SPEC-007.

**Quiet Hours:** Per this Brief's Non-Functional Notes and its recorded reading, none of this feature's three notifications depend on a short window for their value, so all three follow the platform's roughly 8am-9pm daytime-hours rule (XBR-16, platform parameter: `reminder-window-start-hour` to platform parameter: `reminder-window-end-hour`, in the Pro's timezone): a generation event that occurs outside this window holds the notification until the window next opens.

## Content Definition

**Text:**
- **Body:** {pro_display_name} just scheduled your next standing appointment: {service_name} on {occurrence_date} at {occurrence_time} ({timezone}). Manage this series: {series_manage_link}
- **CTA:** {series_manage_link} -- deep-links to FEAT-21.SPEC-002 (My Recurring Series) for this client's series

**Email:**
- **Subject:** Your next appointment with {pro_display_name} is scheduled
- **Body:**
  Hi {client_first_name},

  Your standing appointment series (every {interval} weeks) has generated its next visit:

  Service: {service_name}
  Date & time: {occurrence_date} at {occurrence_time} ({timezone})
  Location: {studio_address}

  You'll get a separate deposit link closer to the date. To view or manage your series, use the link below.
- **CTA (button):** View my series -- deep-links to FEAT-21.SPEC-002 (My Recurring Series) for this client's series

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | Never empty -- required (Pro Account entity) |
| {service_name} | Service -- name (at generation time) | Full Set Lashes | Never empty -- required at service creation (FEAT-01) |
| {occurrence_date} / {occurrence_time} | Booking (occurrence) -- start_time, rendered in the Pro's timezone | Nov 12, 2026 / 2:30 PM | Never empty -- start_time is set at successful generation |
| {timezone} | Pro Account -- timezone | Eastern Time | Never empty -- required per account (XBR-25) |
| {interval} | Recurring Series -- interval | 3 | Never empty -- required at series creation (FEAT-21.SPEC-003) |
| {studio_address} | Pro Account -- studio_address | 123 Main St, Suite 4, Austin, TX | Never empty -- required before go-live (XBR-26) |
| {series_manage_link} | Access Link -- scoped to this client's series, issued via FEAT-06's access-link mechanism | chairtime.app/m/9c3f2a | If link issuance fails, the notification is held and retried per FEAT-08.SPEC-009's failure handling -- it is never sent without its manage link |
| {client_first_name} | Client -- name (first token) | Riley | Renders the full name field if no separable first token exists |

## Delivery Rules

**Batching:** None -- exactly one notification is sent per occurrence generation event. If several occurrences from different series generate for the same client in the same run (an unusual case, since one client typically holds few series), each occurrence's confirmation is sent separately, since each names a distinct appointment the client needs to recognize individually.
**Deduplication:** At most one generated-notification per occurrence. FEAT-21.SPEC-004 guarantees at most one successful generation event per occurrence (a conflicted occurrence's eventual replacement-time acceptance is itself the one generation event for that occurrence), so this notification's trigger cannot re-fire for the same occurrence.
**Retry on failure:** Governed by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback): a failed text is retried once, then falls back to email, and the delivery gap is flagged on the Pro's dashboard (XBR-17).
**Expiry:** None -- a generated-occurrence confirmation never becomes not worth sending; even a late-arriving confirmation (after a retry/fallback cycle) still correctly describes a real, upcoming appointment.

## Edge Cases

- **The occurrence's Booking is cancelled in the brief window between generation and this notification's send** -- The notification still sends (it reports what was true at the moment of successful generation); the client also promptly sees the updated state on FEAT-21.SPEC-002, so she is never left confused for long about a cancelled occurrence she was just told about.
- **Client has both texting consent and an email on file** -- Text is used; no duplicate confirmation is also sent by email.
- **Client's phone number changed since her last booking and fresh consent has not yet been captured** -- Per XBR-15, the notification routes to email, never to the old or unconsented number.
- **A generation event occurs at 11pm in the Pro's timezone** -- The notification is held until the daytime window opens the next morning (platform parameter: `reminder-window-start-hour`), consistent with this Brief's recorded reading that all three of this feature's notifications follow the daytime-hours rule.
- **The client's series manage link cannot be issued at send time** -- The send waits for the link and is treated as a delivery failure under FEAT-08.SPEC-009's retry/fallback path if issuance does not complete in time; the notification is never sent with a missing link.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Triggered by (inbound) | A successful generation, or an accepted replacement time, fires this notification |
| FEAT-14.SPEC-007 (Textability Determination Rule) | References (inbound) | Decides text vs. email for this send |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Triggers (outbound) | Performs the text send when text is the chosen channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | Triggers (outbound) | Performs the email send when email is chosen or is the fallback |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback behavior if this send fails |
| FEAT-21.SPEC-002 (My Recurring Series) | Navigation (outbound) | The CTA deep-links here |

## Analytics and Success Signals

- **recurring_occurrence_generated_notification_sent** (channel: text / email; generation_outcome: normal / replacement_time_accepted) -- supports success-metrics.md: "Self-Service Reschedule Rate" (a client's standing series continuing to inform her without any contact to the Pro is the same self-service pattern this metric measures)
- **recurring_occurrence_generated_notification_cta_tapped** (channel) -- N/A -- no Stage 2 metric measures this notification's own click-through directly; retained so engagement with the series-management surface is observable.

## Acceptance Criteria

**FEAT-21.SPEC-007-AC-01:** Given Riley has active texting consent and her occurrence generates normally, when the generation completes, then she receives a text confirming the service, date/time with timezone, and a link to manage her series.

**FEAT-21.SPEC-007-AC-02:** Given Riley declined texting and provided an email, when her occurrence generates normally, then she receives the same content by email instead.

**FEAT-21.SPEC-007-AC-03:** Given a previously conflicted occurrence's replacement time is accepted, when the acceptance completes, then Riley receives this same notification for the newly scheduled time.

**FEAT-21.SPEC-007-AC-04:** Given Riley taps the manage-series link in this notification, when the tap registers, then it opens FEAT-21.SPEC-002 scoped to her series.

**FEAT-21.SPEC-007-AC-05:** Given Riley's occurrence is cancelled 30 seconds after generation, when the notification and cancellation both process, then Riley still receives the notification, reflecting the moment of successful generation, and separately sees the updated state on FEAT-21.SPEC-002.

**FEAT-21.SPEC-007-AC-06:** Given Riley has both texting consent and an email on file, when her notification sends, then it arrives once, by text, with no duplicate email also sent.

**FEAT-21.SPEC-007-AC-07:** Given Riley's phone number changed and fresh consent has not been captured, when her occurrence generates, then the notification is sent by email, never to the unconsented number.

**FEAT-21.SPEC-007-AC-08:** Given a text send to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the notification by email, and the delivery gap is flagged on Talia's dashboard.

**FEAT-21.SPEC-007-AC-09:** Given an occurrence generates at 11pm in the Pro's timezone, when the generation completes, then the notification is held and delivered when the daytime window next opens, not before.

**FEAT-21.SPEC-007-AC-10:** Given the series manage link cannot be issued at send time, when the send is attempted, then it waits for the link and is treated as a delivery failure under FEAT-08.SPEC-009 if issuance does not complete in time.

**FEAT-21.SPEC-007-AC-11:** Given FEAT-21.SPEC-004 guarantees at most one successful generation event per occurrence, when the notification trigger is evaluated, then at most one generated-notification is ever sent for that occurrence.

**FEAT-21.SPEC-007-AC-12:** Given Talia (the Pro) is not named as a recipient of this notification, when an occurrence generates, then no push or message is sent to her from this spec -- she sees the occurrence only through her own schedule view and FEAT-21.SPEC-010 (Pro Recurring Series Management).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 2 | 2 |
| Preference States | 2 (consent granted, consent revoked/declined) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
