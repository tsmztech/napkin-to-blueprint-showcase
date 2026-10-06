---
document_type: spec
spec_type: notification
spec_id: FEAT-08.SPEC-004
spec_name: Booking Change & Refund Notice
spec_slug: booking-change-refund-notice
parent_feature: FEAT-08
parent_feature_name: Automated Booking Messaging
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Notification Spec: Booking Change & Refund Notice

## Overview

**Name:** Booking Change & Refund Notice
**ID:** FEAT-08.SPEC-004
**Type:** Notification
**Purpose:** Tells the client, promptly, when their booking is cancelled, rescheduled, or refunded by either themselves or the Pro -- including exactly what happened to their deposit -- so no client is ever left wondering whether a change went through or whether their money is safe.
**Parent Feature:** FEAT-08 -- Automated Booking Messaging

## Scope and Non-Goals

**In Scope:**
- The client-facing notice for every booking-change type: client cancellation, client reschedule, Pro cancellation, Pro reschedule, and a refund completing or entering an in-progress state
- The deposit outcome for each change type (refunded, kept, carried over, or in progress)
- Variant content per change type, sharing one delivery-rules definition

**Non-Goals:**
- Deciding the deposit outcome itself (refund vs. keep, full vs. carried-over) -- owned by FEAT-09 (Cancellation & No-Show Policy Engine, XBR-09); this spec only reports the outcome FEAT-09 or FEAT-30 determines.
- The Pro-facing notification of the same events -- covered separately by FEAT-08.SPEC-005 (Pro Booking Activity Notification); this spec is client-facing only.
- Alerting the Pro that a refund failed -- that is a Pro-attention condition owned by FEAT-08.SPEC-006 (Pro Attention Alert); this spec's "refund in progress" client wording is the client-side counterpart to that same underlying failure.
- The original booking confirmation content -- owned by FEAT-08.SPEC-001; this spec covers only subsequent changes to an already-confirmed booking.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active Messaging Consent for texting (FEAT-08.SPEC-011) | A cancellation, reschedule, or refund is time-sensitive and financially material; the client should learn of it wherever they already receive booking messages |
| Email | The client has not granted texting consent | Ensures the notice always reaches the client, per BRIEF.md's stated email fallback |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client cancels or reschedules their own booking | FEAT-10.SPEC-004 (Booking Update Commit), via its trigger contract FEAT-10.SPEC-006 (Client-Initiated Cancel/Reschedule) | Always, on a successfully saved cancellation or reschedule | Booking (updated state, new time if rescheduled), Deposit Transaction (outcome), Cancellation Policy (window_hours) |
| Pro cancels or reschedules a client's booking | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) / FEAT-30.SPEC-008 (Bulk Cancellation Commit), via their trigger contract FEAT-30.SPEC-012 (Pro Booking Management) | Always, on a successfully saved Pro-initiated cancellation or reschedule | Booking (updated state, new time if rescheduled), Deposit Transaction (outcome) |
| Pro issues a goodwill refund | FEAT-30.SPEC-009 (Goodwill Refund Commit) / FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution), via their trigger contract FEAT-30.SPEC-012 (Pro Booking Management) | On a successfully processed goodwill refund | Deposit Transaction (Refunded outcome) |
| Automatic refund succeeds, enters progress, or fails | FEAT-09.SPEC-005 (Automatic Deposit Refund) (Cancellation & No-Show Policy Engine) | On the corresponding Deposit Transaction state transition | Deposit Transaction (status: Refunded / Refund in Progress) |

## Audience and Preferences

**Recipients:** The Client tied to the Booking (Access Matrix: Booking & Payment = Own-only), the sole recipient. Platform Operator (Support) has View-only access to delivery status only, per the Access Matrix.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (governs channel, not whether this notice sends) | Granted / Revoked | Captured at booking | FEAT-06 at booking; changed via FEAT-14 |

This notice carries no separate opt-out: a change to the client's own paid booking is transactional information the client cannot decline to receive, consistent with the treatment of FEAT-08.SPEC-001.

**Quiet Hours:** N/A -- like the confirmation, this notice is the direct, expected report of a change that just happened to the client's own booking, not an unprompted interruption; it sends immediately regardless of time of day. XBR-16's daytime window applies only to the discretionary pre-appointment reminder (FEAT-08.SPEC-002), not to this transactional notice.

## Content Definition

**Text (client-initiated cancellation, outside window -- full refund):**
- **Body:** Your {appointment_date} appointment with {pro_display_name} has been cancelled. Your {deposit_amount} deposit is being refunded to your card. Manage: {manage_link}

**Text (client-initiated cancellation, inside window -- deposit kept):**
- **Body:** Your {appointment_date} appointment with {pro_display_name} has been cancelled. Per the cancellation policy you agreed to, your {deposit_amount} deposit is kept. Manage: {manage_link}

**Text (client-initiated reschedule, outside window -- deposit carried over):**
- **Body:** Your appointment with {pro_display_name} has been moved to {new_appointment_date} at {new_appointment_time} ({timezone}). Your {deposit_amount} deposit carries over -- nothing further to pay now. Manage: {manage_link}

**Text (Pro-initiated cancellation -- always full refund):**
- **Body:** {pro_display_name} has cancelled your {appointment_date} appointment. Your {deposit_amount} deposit is being refunded to your card in full. Manage: {manage_link}

**Text (Pro-initiated reschedule):**
- **Body:** {pro_display_name} has moved your appointment to {new_appointment_date} at {new_appointment_time} ({timezone}). Manage: {manage_link}

**Text (refund in progress):**
- **Body:** Your {deposit_amount} refund for your {appointment_date} appointment with {pro_display_name} is in progress. You don't need to do anything -- it will complete automatically. Manage: {manage_link}

**Email (mirrors each text variant above):**
- **Subject:** Update on your appointment with {pro_display_name}
- **Body:** Hi {client_first_name}, followed by the plain-language equivalent of the matching text variant above, with the same facts (what changed, the new time if applicable, and the exact deposit outcome).
- **CTA (button):** Manage my booking -- deep-links to the booking-specific manage link

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | Never empty (required field) |
| {appointment_date} | Booking -- start_time (the original time, for a cancellation notice) | Oct 4, 2026 | Never empty -- fixed at booking |
| {new_appointment_date} / {new_appointment_time} | Booking -- start_time (the updated time, for a reschedule notice) | Oct 11, 2026 / 2:30 PM | Never empty when the variant is a reschedule notice -- this placeholder is not used in cancellation variants |
| {timezone} | Pro Account -- timezone | Eastern Time | Never empty (required per account) |
| {deposit_amount} | Deposit Transaction -- amount | $40.00 | Never empty -- fixed once at booking |
| {manage_link} | Access Link -- the booking-specific manage link (FEAT-08.SPEC-010) | chairtime.app/m/8f2a1c | If the underlying booking is now fully closed out (completed history), the link still resolves to a read-only view of that booking's final state |
| {client_first_name} | Client -- name (first token) | Riley | Renders the full name field if no separable first token exists |

## Delivery Rules

**Batching:** None -- each change event (a cancellation, a reschedule, or a refund-state transition) produces its own single notice at the moment it occurs. A booking that is rescheduled and later cancelled produces two separate notices, one per event, since each is a distinct fact the client needs at the time it happens.
**Deduplication:** At most one notice per triggering event. A refund transitioning from "in progress" to "completed" is itself a second, distinct event and produces its own follow-up notice (see the refund-in-progress content variant and its natural successor, the original cancellation/refund confirmation content once the refund actually completes) -- these are not duplicates of the same event.
**Retry on failure:** Governed by FEAT-08.SPEC-009: a failed text is retried once, then falls back to email, with the gap flagged on the Pro's dashboard.
**Expiry:** None -- a change or refund notice never becomes not-worth-sending; it reports a fact about the client's own money and appointment that remains true and relevant no matter when it is finally delivered.

## Edge Cases

- **Client cancels their own booking, then the Pro also attempts an action on it before the notice sends** -- Per the Booking entity's Contention resolution (reject-with-refresh, feature-dependency-map.md), only the first committed transition applies; this notice reports the transition that actually committed, never a stale or since-superseded one.
- **A refund fails outright rather than merely being slow** -- The client still sees the "refund in progress" wording, never a failure message; the underlying failure is retried automatically and flagged only to the Pro (FEAT-08.SPEC-006), per XBR-10's "never dropped, never exposed to the client as a failure" framing.
- **A client reschedule lands inside the cancellation window (treated as late cancellation plus new deposit, per XBR-09)** -- This notice reports both facts plainly: the original deposit is kept under the policy, and the new booking's own confirmation (FEAT-08.SPEC-001) covers the new deposit separately; the two are never merged into one ambiguous message.
- **The Pro cancels a booking that the client had already tried to cancel moments earlier** -- Whichever transition committed first is the one this notice reports (Booking entity Contention rule); the client is not sent two conflicting notices for the same terminal state.
- **Client's texting consent is revoked between booking and this notice** -- The notice honors the consent state current at send time (FEAT-08.SPEC-011), routing to email if consent is no longer active.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-004 (Booking Update Commit) / FEAT-10.SPEC-006 (Client-Initiated Cancel/Reschedule) | Triggered by (inbound) | A client cancellation or reschedule fires this notice; FEAT-10.SPEC-006 is the trigger-and-audience contract, this spec owns the message content |
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) / FEAT-30.SPEC-008 (Bulk Cancellation Commit) / FEAT-30.SPEC-009 (Goodwill Refund Commit) / FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution) / FEAT-30.SPEC-012 (Pro Booking Management notifications) | Triggered by (inbound) | A Pro-initiated cancellation, reschedule, or goodwill refund fires this notice; FEAT-30.SPEC-012 is the trigger-and-audience contract, this spec owns the message content |
| FEAT-09 (Cancellation & No-Show Policy Engine) | Triggered by (inbound) | An automatic refund succeeding, entering progress, or failing (client-visible as "in progress") fires this notice |
| FEAT-08.SPEC-006 (Pro Attention Alert) | References (outbound) | Covers the Pro-facing counterpart when a refund fails |
| FEAT-08.SPEC-005 (Pro Booking Activity Notification) | References (outbound) | Covers the Pro-facing counterpart for cancellations and reschedules |
| FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) | References (inbound) | Supplies the {manage_link} placeholder |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (inbound) | Decides text vs. email for this send |
| FEAT-08.SPEC-012 / FEAT-08.SPEC-013 (Text / Email Capabilities) | Triggers (outbound) | Perform the actual send |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback on failure |
| FEAT-26.SPEC-004 (WhatsApp Channel Eligibility & Consent Rule) | References (inbound) | Runs before FEAT-08.SPEC-011's text/email decision; if the client is WhatsApp-eligible the send goes by WhatsApp, otherwise it falls through to FEAT-08.SPEC-011 |
| FEAT-16.SPEC-002 (Activity Event Recording) -- within FEAT-16 (Booking & Payment Activity Record) | Affects (outbound) | Every send of notice is written to the append-only activity record |

## Analytics and Success Signals

- **change_notice_sent** (change_type: client_cancel / client_reschedule / pro_cancel / pro_reschedule / refund_in_progress; channel) -- supports success-metrics.md: "Automatic Refund Correctness"
- **change_notice_refund_outcome_shown** (outcome: refunded / kept / carried_over / in_progress) -- supports success-metrics.md: "Automatic Refund Correctness"
- **change_notice_sent** (change_type: pro_cancel / pro_reschedule) -- supports success-metrics.md: "Pro Change Correctness"
- **change_notice_cta_tapped** (destination: manage_link) -- supports success-metrics.md: "Self-Service Access Success"

## Acceptance Criteria

**FEAT-08.SPEC-004-AC-01:** Given Riley cancels her booking outside the Pro's cancellation window, when FEAT-10 completes the cancellation, then Riley receives a notice stating her deposit is being refunded to her card.

**FEAT-08.SPEC-004-AC-02:** Given Riley cancels her booking inside the cancellation window, when FEAT-10 completes the cancellation, then Riley receives a notice stating her deposit is kept per the policy she agreed to.

**FEAT-08.SPEC-004-AC-03:** Given Riley reschedules her booking outside the window, when FEAT-10 completes the reschedule, then Riley receives a notice with the new date/time and confirmation that her deposit carries over with nothing further to pay.

**FEAT-08.SPEC-004-AC-04:** Given the Pro cancels Riley's booking, when FEAT-30 completes the cancellation, then Riley receives a notice stating her deposit is being refunded in full, regardless of timing.

**FEAT-08.SPEC-004-AC-05:** Given the Pro reschedules Riley's booking to a new time, when FEAT-30 completes the reschedule, then Riley receives a notice with the new date and time.

**FEAT-08.SPEC-004-AC-06:** Given an automatic refund for Riley's cancelled booking cannot complete immediately, when FEAT-09 sets the Deposit Transaction to Refund in Progress, then Riley receives a notice stating the refund is in progress and she does not need to do anything.

**FEAT-08.SPEC-004-AC-07:** Given a refund that was "in progress" for Riley later completes, when FEAT-09 updates the Deposit Transaction to Refunded, then Riley receives a follow-up notice confirming the refund completed.

**FEAT-08.SPEC-004-AC-08:** Given Riley reschedules inside the cancellation window, when FEAT-10 processes the late reschedule (XBR-09), then Riley receives a notice stating her original deposit is kept, distinct from the separate confirmation for her new booking's own deposit.

**FEAT-08.SPEC-004-AC-09:** Given Riley cancels her own booking and, moments later, the Pro also attempts to cancel it, when only the first transition commits (Booking entity Contention rule), then Riley receives exactly one notice, reflecting the committed transition.

**FEAT-08.SPEC-004-AC-10:** Given a refund for Riley's booking fails outright on the Pro's payout side, when the failure occurs, then Riley still sees only the "refund in progress" wording, never a failure message, while the Pro is separately alerted via FEAT-08.SPEC-006.

**FEAT-08.SPEC-004-AC-11:** Given Riley has revoked texting consent since booking, when a change notice for her booking is triggered, then it is sent by email, honoring her current consent state.

**FEAT-08.SPEC-004-AC-12:** Given a text change notice to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the notice by email.

**FEAT-08.SPEC-004-AC-13:** Given Riley taps "Manage my booking" from a change notice, when the tap registers, then the change_notice_cta_tapped event fires and she is taken to her booking through the manage link.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 4 (client cancel/reschedule, Pro cancel/reschedule/goodwill refund, automatic refund outcome) | 4 |
| Preference States | 2 (consent granted, consent revoked/declined) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
