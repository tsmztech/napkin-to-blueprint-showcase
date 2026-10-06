---
document_type: spec
spec_type: notification
spec_id: FEAT-21.SPEC-008
spec_name: Occurrence Time Change Advance Notice
spec_slug: occurrence-time-change-advance-notice
parent_feature: FEAT-21
parent_feature_name: Recurring/Standing Appointments
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Notification Spec: Occurrence Time Change Advance Notice

## Overview

**Name:** Occurrence Time Change Advance Notice
**ID:** FEAT-21.SPEC-008
**Type:** Notification
**Purpose:** Gives the client advance notice, with a prompt to pick a new time, when a standing appointment's usual slot is no longer available for an upcoming occurrence -- without disturbing the rest of her series.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments

## Scope and Non-Goals

**In Scope:**
- The advance notice sent when an occurrence's usual time fails validation at generation, on both of the product's client channels (text and email)
- The prompt and its link to the pick-a-new-time flow
- Delivery, retry, and expiry behavior for this notification

**Non-Goals:**
- Deciding whether the usual time is actually unavailable, and validating whichever replacement time the client picks -- owned by FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling); this spec only announces the outcome and links to the flow it hands the client to.
- Confirming the occurrence once a replacement time is accepted -- owned by FEAT-21.SPEC-007 (Occurrence Generated Notification), a separate communication sent once a time is actually set.
- Deciding text vs. email for this send -- owned by FEAT-14.SPEC-007 (Textability Determination Rule).
- Notifying the Pro that one of her clients needs a new time -- product-features.md's Communications field for this feature names only a client-facing advance notice; a repeatedly failing generation (this spec's occurrence-level trigger is distinct from that) surfaces to the Pro separately, per FEAT-21.SPEC-004's own dashboard-flag path.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active Messaging Consent for texting (FEAT-14.SPEC-007) | This Brief's Communications field names this as an advance notice the client needs to act on soon; text reaches her where she is most likely to respond promptly |
| Email | The client has not granted texting consent (FEAT-14.SPEC-007) | Every client who declines texting still supplies an email at booking, so the notice is never simply undeliverable |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| An occurrence's usual time fails slot validation at generation | FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Fires once per conflicted occurrence, immediately when the candidate slot fails validation | Booking (service, the usual time the occurrence would have had), Pro Account (display_name, timezone), the pick-a-new-time flow's entry point |

## Audience and Preferences

**Recipients:** The Client tied to the conflicted occurrence's Booking (Access Matrix: Recurring Appointments = Own-only for the Client) -- the sole recipient, since only she can pick the replacement time.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (governs channel, not whether this message sends) | Granted / Revoked | Captured at booking | FEAT-06 (Client Booking Identity) at booking; changed via FEAT-14 (Messaging Consent Management) |

This notice carries no separate on/off toggle: it requires the client's action to keep her standing appointment on schedule, so it is never a discretionary message she can silence -- only the channel it arrives on varies, per FEAT-14.SPEC-007.

**Quiet Hours:** Per this Brief's recorded reading, this notification is an ordinary reminder-class communication with no short-window urgency, so it follows the platform's roughly 8am-9pm daytime-hours rule (XBR-16, platform parameter: `reminder-window-start-hour` to platform parameter: `reminder-window-end-hour`, in the Pro's timezone).

## Content Definition

**Text:**
- **Body:** Heads up -- {pro_display_name}'s usual time for your standing appointment ({service_name}, normally {usual_day_time}) isn't available this time. Pick a new time for just this one visit: {pick_new_time_link}
- **CTA:** {pick_new_time_link} -- deep-links to FEAT-21.SPEC-004's replacement-time flow for this one occurrence

**Email:**
- **Subject:** We need a new time for your next appointment with {pro_display_name}
- **Body:**
  Hi {client_first_name},

  Your standing appointment's usual time ({service_name}, normally {usual_day_time}) isn't available for your next visit -- {pro_display_name}'s schedule changed.

  This affects only this one upcoming appointment; the rest of your series continues as usual. Pick a new time below.
- **CTA (button):** Pick a new time -- deep-links to FEAT-21.SPEC-004's replacement-time flow for this one occurrence

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | Never empty -- required (Pro Account entity) |
| {service_name} | Service -- name | Full Set Lashes | Never empty -- required at service creation (FEAT-01) |
| {usual_day_time} | Recurring Series -- originating_time (day-of-week and time-of-day pattern) | Tuesdays at 2:30 PM | Never empty -- set at series creation (FEAT-21.SPEC-003) |
| {pick_new_time_link} | Access Link -- scoped to this one conflicted occurrence, issued via FEAT-06's access-link mechanism | chairtime.app/m/7b1d4e | If link issuance fails, the notification is held and retried per FEAT-08.SPEC-009's failure handling -- it is never sent without its link |
| {client_first_name} | Client -- name (first token) | Riley | Renders the full name field if no separable first token exists |

## Delivery Rules

**Batching:** None -- one notice per conflicted occurrence. Each conflict names a distinct occurrence the client must individually act on, so batching would obscure which appointment needs a new time.
**Deduplication:** At most one advance notice per conflicted occurrence. FEAT-21.SPEC-004 flags an occurrence as needing a new time at most once per generation attempt; a re-attempt against the same still-unresolved occurrence does not re-send this notice.
**Retry on failure:** Governed by FEAT-08.SPEC-009 (Message Delivery Retry & Fallback): a failed text is retried once, then falls back to email, and the delivery gap is flagged on the Pro's dashboard (XBR-17).
**Expiry:** None on the notice itself -- it does not expire, since a client can still pick a new time for the occurrence at any point before the occurrence's own cancellation cut-off; the occurrence's status simply remains "Needs new time" on FEAT-21.SPEC-002 until she acts, per FEAT-21.SPEC-004's edge case that an abandoned replacement flow does not block the rest of the series.

## Edge Cases

- **The conflicted occurrence is cancelled by the client (via FEAT-21.SPEC-002) before she acts on this notice** -- The pick-a-new-time link, if tapped afterward, shows "This appointment is no longer active." (FEAT-21.SPEC-006), since there is nothing left to reschedule.
- **The whole series is cancelled while a conflicted occurrence's notice is still unresolved** -- The link, if tapped afterward, shows the same "no longer active" experience; no further prompt is sent once the series is Ended.
- **Client has both texting consent and an email on file** -- Text is used; no duplicate notice is also sent by email.
- **A conflict is detected at 11pm in the Pro's timezone** -- The notice is held until the daytime window opens the next morning, consistent with this Brief's recorded reading.
- **The client's phone number changed since her last booking and fresh consent has not yet been captured** -- Per XBR-15, the notice routes to email, never to the old or unconsented number.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-004 (Occurrence Generation & Conflict Handling) | Triggered by (inbound) / Navigation (outbound) | A slot-validation failure at generation fires this notice; the CTA deep-links into that spec's replacement-time flow |
| FEAT-14.SPEC-007 (Textability Determination Rule) | References (inbound) | Decides text vs. email for this send |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | Triggers (outbound) | Performs the text send when text is the chosen channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | Triggers (outbound) | Performs the email send when email is chosen or is the fallback |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback behavior if this send fails |
| FEAT-21.SPEC-006 (Series & Occurrence Cancellation Rules) | References (outbound) | Governs the "no longer active" experience if the occurrence or series is cancelled before the client acts |

## Analytics and Success Signals

- **recurring_occurrence_time_change_notice_sent** (channel: text / email) -- N/A -- no Stage 2 metric measures this notice's delivery directly; retained so the frequency of schedule-driven conflicts reaching clients is observable, alongside FEAT-21.SPEC-004's own conflict-flagged event.
- **recurring_occurrence_time_change_cta_tapped** (channel) -- supports success-metrics.md: "Self-Service Reschedule Rate" (a client picking her own new time in-app, without contacting the Pro, is exactly the self-service pattern this metric measures)

## Acceptance Criteria

**FEAT-21.SPEC-008-AC-01:** Given Riley has active texting consent and her occurrence's usual time fails validation, when the conflict is detected, then she receives a text naming the affected appointment and a link to pick a new time.

**FEAT-21.SPEC-008-AC-02:** Given Riley declined texting and provided an email, when her occurrence's usual time fails validation, then she receives the same content by email instead.

**FEAT-21.SPEC-008-AC-03:** Given Riley taps the pick-a-new-time link, when the tap registers, then it opens FEAT-21.SPEC-004's replacement-time flow scoped to that one occurrence.

**FEAT-21.SPEC-008-AC-04:** Given Riley reads this notice, then it states that only this one upcoming appointment is affected and the rest of her series continues as usual.

**FEAT-21.SPEC-008-AC-05:** Given Riley cancels the conflicted occurrence before acting on this notice, when she later taps the link anyway, then she sees "This appointment is no longer active."

**FEAT-21.SPEC-008-AC-06:** Given Riley's whole series is cancelled while this notice is unresolved, when she later taps the link, then she sees the same "no longer active" experience.

**FEAT-21.SPEC-008-AC-07:** Given Riley has both texting consent and an email on file, when this notice sends, then it arrives once, by text, with no duplicate email also sent.

**FEAT-21.SPEC-008-AC-08:** Given Riley's phone number changed and fresh consent has not been captured, when a conflict is detected for her occurrence, then the notice is sent by email, never to the unconsented number.

**FEAT-21.SPEC-008-AC-09:** Given a text send to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the notice by email, and the delivery gap is flagged on Talia's dashboard.

**FEAT-21.SPEC-008-AC-10:** Given a conflict is detected at 11pm in the Pro's timezone, when the conflict occurs, then the notice is held and delivered when the daytime window next opens.

**FEAT-21.SPEC-008-AC-11:** Given Riley does not act on this notice at all, when she later opens FEAT-21.SPEC-002, then the occurrence still shows "Needs new time" and the rest of her series is unaffected.

**FEAT-21.SPEC-008-AC-12:** Given FEAT-21.SPEC-004 flags an occurrence as needing a new time at most once per generation attempt, when the notice trigger is evaluated, then at most one advance notice is sent for that occurrence's unresolved conflict.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 2 (consent granted, consent revoked/declined) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
