---
document_type: spec
spec_type: notification
spec_id: FEAT-30.SPEC-012
spec_name: Pro Action Client Notice
spec_slug: pro-action-client-notice
parent_feature: FEAT-30
parent_feature_name: Pro Booking Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Notification Spec: Pro Action Client Notice

## Overview

**Name:** Pro Action Client Notice
**ID:** FEAT-30.SPEC-012
**Type:** Notification
**Purpose:** The trigger-and-audience contract that ensures the client is told when the Pro cancelled their booking (with refund status), rescheduled it (with the new time and a fresh manage link), or issued a goodwill refund -- so no client is ever left wondering whether a Pro-initiated change went through or what happened to their deposit. The message content is owned by FEAT-08.SPEC-004; this spec defines when it fires, for whom, and what data this feature supplies.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals
**In Scope:**
- The trigger events in this feature that require a client notice: single cancellation, single reschedule, bulk cancellation (per affected booking), and goodwill refund (completed or in-progress)
- The audience contract: exactly one client recipient per event, the Client tied to the affected Booking
- The data this feature supplies to FEAT-08.SPEC-004 for each event (outcome type, deposit outcome, new time, fresh manage link)
- The Pro-side counterpart trigger: the same events also reach FEAT-08.SPEC-005 (Pro Booking Activity Notification), which owns the Pro-facing content

**Non-Goals:**
- Message wording, channels, placeholders, and text/email variants -- owned by FEAT-08.SPEC-004 (Booking Change & Refund Notice), which already defines the client-facing content for every Pro-initiated event; this spec defines no duplicate content
- The client-facing notice for a client-initiated cancellation or reschedule -- triggered by FEAT-10.SPEC-006 and rendered by FEAT-08.SPEC-004; this spec covers only Pro-initiated events
- The Pro-facing content of any of these events -- owned by FEAT-08.SPEC-005 (routine activity) and FEAT-08.SPEC-006 (a refund failure needing attention); this spec is client-facing only
- Deciding the deposit outcome itself -- owned by FEAT-09 (Cancellation & No-Show Policy Engine, XBR-09) for a Pro cancellation or reschedule, and by FEAT-30.SPEC-009/FEAT-30.SPEC-011 for a goodwill refund; this spec only reports the outcome those specs determine
- Issuing the fresh manage link -- owned by FEAT-08.SPEC-010; the actual text/email send mechanics -- owned by FEAT-08.SPEC-012 (text) and FEAT-08.SPEC-013 (email)

## Channels
| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active Messaging Consent for texting (per FEAT-14, evaluated by FEAT-08.SPEC-011's channel-selection rule) | Channel choice and wording are defined by FEAT-08.SPEC-004; a Pro-initiated cancellation, reschedule, or refund is time-sensitive and financially material, so this contract always requires the notice to go out |
| Email | The client has not granted texting consent | Ensures the notice always reaches the client, per BRIEF.md's stated email fallback; delivered per FEAT-08.SPEC-004 |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Talia's single cancellation or reschedule commits | FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Always, on a successfully saved Pro-initiated cancellation or reschedule; this contract passes the event on to FEAT-08.SPEC-004 (client) and FEAT-08.SPEC-005 (Pro) | Booking (updated state, new time if rescheduled), Deposit Transaction (outcome), fresh manage link (reschedule only, from FEAT-08.SPEC-010) |
| A booking within a bulk cancellation commits | FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Fires once per successfully cancelled booking in the reviewed set | Booking (updated state), Deposit Transaction (outcome) |
| A goodwill refund completes or enters progress | FEAT-30.SPEC-009 (Goodwill Refund Commit) via FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution) | On the Deposit Transaction reaching Refunded or Refund in Progress from a goodwill action; passed on to FEAT-08.SPEC-004 | Deposit Transaction (status, outcome_reason: goodwill) |

## Audience and Preferences

**Recipients:** The Client tied to the affected Booking (Access Matrix: Booking & Payment = Own-only), the sole recipient. Platform Operator (Support) has View-only access to delivery status only, per the Access Matrix.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (governs channel, not whether this notice sends) | Granted / Revoked | Captured at booking | FEAT-06 at booking; changed via FEAT-14 |

This notice carries no separate opt-out: a change to the client's own paid booking, made by the Pro, is transactional information the client cannot decline to receive (consistent with FEAT-08.SPEC-004).

**Quiet Hours:** N/A -- this notice is the direct, expected report of a change the Pro just made to the client's own booking, not an unprompted interruption; it sends immediately regardless of time of day. XBR-16's daytime window governs only the discretionary pre-appointment reminder (FEAT-08.SPEC-002), not this transactional notice, and FEAT-08.SPEC-004 applies the same rule.

## Content Definition
The client-facing message content (text bodies, email subject and body, CTA, and placeholders) is owned entirely by FEAT-08.SPEC-004 (Booking Change & Refund Notice); this spec restates none of it. For each event this feature raises, the table states which FEAT-08.SPEC-004 content applies and what data this feature must supply so that content renders correctly.

| Event Raised By This Feature | FEAT-08.SPEC-004 Content Applied | Data This Feature Supplies |
|------------------------------|----------------------------------|----------------------------|
| Single Pro cancellation (FEAT-30.SPEC-007) | The Pro-initiated cancellation variant (always a full refund, XBR-09) | Booking reference, original start_time, deposit amount, deposit outcome = refunded in full |
| Single Pro reschedule (FEAT-30.SPEC-007) | The Pro-initiated reschedule variant | Booking reference, new start_time, deposit outcome = carried over (XBR-09), the fresh manage link issued by FEAT-08.SPEC-010 (XBR-18) |
| Each booking cancelled in a bulk cancellation (FEAT-30.SPEC-008) | The Pro-initiated cancellation variant, once per booking | Same data as a single cancellation, per booking |
| Goodwill refund completed (FEAT-30.SPEC-009 via FEAT-30.SPEC-011) | The refund outcome content for a completed Pro-issued refund | Booking reference, deposit amount, deposit outcome = refunded, outcome_reason = goodwill |
| Goodwill refund in progress (FEAT-30.SPEC-009 via FEAT-30.SPEC-011) | The refund-in-progress variant | Booking reference, deposit amount, deposit outcome = in progress |

The Pro-facing counterpart of these events (routine activity) is composed by FEAT-08.SPEC-005 from the same trigger; a refund that fails to complete is flagged to the Pro by FEAT-08.SPEC-006.

## Delivery Rules

**Batching:** None -- each change event (a cancellation, a reschedule, a bulk-cancellation booking, or a goodwill-refund state transition) produces its own single notice at the moment it occurs. A bulk cancellation of four bookings produces four independent notices, one per affected client, never one combined message.
**Deduplication:** At most one notice per triggering event. A goodwill refund transitioning from "in progress" to "completed" is itself a second, distinct event and produces its own follow-up notice.
**Retry on failure:** Governed by FEAT-08.SPEC-009 and executed by FEAT-08.SPEC-004: a failed text is retried once, then falls back to email, with the gap flagged on the Pro's dashboard (FEAT-08.SPEC-006).
**Expiry:** None -- a change or refund notice never becomes not-worth-sending; it reports a fact about the client's own money and appointment that remains true and relevant no matter when it is finally delivered.

## Edge Cases

- **Talia cancels a booking that Riley had already tried to cancel moments earlier through FEAT-10** -- Per the Booking entity's Contention resolution (reject-with-refresh), only the first committed transition applies; this notice reports the transition that actually committed, and Riley receives exactly one notice for it, not two.
- **A goodwill refund fails outright rather than merely being slow** -- Riley still sees only the "refund in progress" wording, never a failure message; the underlying failure is retried automatically (FEAT-30.SPEC-011) and flagged only to Talia (FEAT-08.SPEC-006), per XBR-10.
- **One booking within a bulk cancellation fails eligibility while its siblings succeed** -- Only the successfully cancelled bookings' clients receive this notice; the client of the failed booking receives no notice for an action that never committed.
- **Riley's texting consent is revoked between booking and this notice** -- The notice honors the consent state current at send time (FEAT-08.SPEC-011), routing to email if consent is no longer active.
- **A Pro-made reschedule lands on a booking whose fresh manage link has not yet been issued at the instant this notice would fire** -- FEAT-30.SPEC-007 requests the link from FEAT-08.SPEC-010 before raising this event (XBR-18), and the event carries that fresh link, so the manage link in the message FEAT-08.SPEC-004 renders is never empty or a stale, superseded link.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-007 (Pro Cancel/Reschedule Commit) | Triggered by (inbound) | A single Pro cancellation or reschedule fires this notice |
| FEAT-30.SPEC-008 (Bulk Cancellation Commit) | Triggered by (inbound) | Each successfully cancelled booking within a bulk action fires this notice independently |
| FEAT-30.SPEC-009 (Goodwill Refund Commit) | Triggered by (inbound) | A completed or in-progress goodwill refund fires this notice |
| FEAT-30.SPEC-011 (Goodwill & Bulk-Cancellation Refund Execution) | Triggered by (inbound) | A refund outcome (completed or in-progress) from this integration feeds the content this notice reports |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Triggers (outbound) | Content owner: renders and delivers the client message for every event this contract raises; FEAT-08.SPEC-004 lists this spec as its trigger-and-audience contract |
| FEAT-08.SPEC-005 (Pro Booking Activity Notification) | Triggers (outbound) | Pro-facing counterpart: composes the Pro's routine-activity notification from the same events |
| FEAT-08.SPEC-010 (Booking-Specific Manage Link Issuance) | References (inbound) | Source of the fresh manage link supplied with a reschedule event (XBR-18) |
| FEAT-08.SPEC-011 (Messaging Consent & Channel Selection Rule) | References (inbound) | Decides text vs. email for this send |
| FEAT-08.SPEC-012 / FEAT-08.SPEC-013 (Text / Email Capabilities) | References (outbound) | Perform the actual send, via FEAT-08.SPEC-004 |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Governs retry and fallback on failure |

## Analytics and Success Signals

- **pro_action_client_notice_sent** (change_type: pro_cancel / pro_reschedule / bulk_cancel / goodwill_refund / goodwill_refund_in_progress; channel) -- supports success-metrics.md: "Pro Change Correctness"
- **pro_action_client_notice_refund_outcome_shown** (outcome: refunded / carried_over / in_progress) -- supports success-metrics.md: "Automatic Refund Correctness"
- **pro_action_client_notice_cta_tapped** (destination: manage_link) -- supports success-metrics.md: "Self-Service Access Success"

## Acceptance Criteria

**FEAT-30.SPEC-012-AC-01:** Given Talia cancels Riley's booking, when FEAT-30.SPEC-007 completes the cancellation, then this contract raises the cancellation event to FEAT-08.SPEC-004 with deposit outcome "refunded in full", and Riley receives the notice FEAT-08.SPEC-004 defines for it, regardless of timing.

**FEAT-30.SPEC-012-AC-02:** Given Talia reschedules Riley's booking to a new time, when FEAT-30.SPEC-007 completes the reschedule, then this contract raises the reschedule event to FEAT-08.SPEC-004 with the new date and time and deposit outcome "carried over", and Riley receives the notice FEAT-08.SPEC-004 defines for it.

**FEAT-30.SPEC-012-AC-03:** Given Talia cancels four bookings at once through FEAT-30.SPEC-008, when each booking successfully commits, then each affected client receives their own independent cancellation notice, never one combined message.

**FEAT-30.SPEC-012-AC-04:** Given Talia issues a goodwill refund that completes immediately, when FEAT-30.SPEC-011 confirms it, then this contract raises the completed-refund event to FEAT-08.SPEC-004 with outcome_reason goodwill, and Riley receives the notice FEAT-08.SPEC-004 defines for it.

**FEAT-30.SPEC-012-AC-05:** Given a goodwill refund for Riley's booking cannot complete immediately, when FEAT-30.SPEC-011 sets the Deposit Transaction to Refund in Progress, then this contract raises the refund-in-progress event to FEAT-08.SPEC-004, and Riley receives the in-progress notice FEAT-08.SPEC-004 defines for it.

**FEAT-30.SPEC-012-AC-06:** Given a goodwill refund that was "in progress" for Riley later completes, when the Deposit Transaction updates to Refunded, then this contract raises a second, distinct completed-refund event, and Riley receives the follow-up notice FEAT-08.SPEC-004 defines for it.

**FEAT-30.SPEC-012-AC-07:** Given Riley cancelled her own booking through FEAT-10 moments before Talia's cancellation commit runs, when only the first transition commits, then Riley receives exactly one notice, reflecting the committed transition.

**FEAT-30.SPEC-012-AC-08:** Given Talia's bulk cancellation reports one booking as failed, when the outcome is processed, then that failed booking's client receives no notice, since the cancellation never committed.

**FEAT-30.SPEC-012-AC-09:** Given Riley has revoked texting consent since booking, when a Pro-action notice for her booking is triggered, then it is sent by email, honoring her current consent state.

**FEAT-30.SPEC-012-AC-10:** Given a text notice to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the notice by email.

**FEAT-30.SPEC-012-AC-11:** Given Talia's reschedule triggers a fresh manage link for Riley, when this notice sends, then the event this contract raises carries that fresh link from FEAT-08.SPEC-010, and the notice never carries the booking's previous, now-superseded link.

**FEAT-30.SPEC-012-AC-12:** Given Riley taps "Manage my booking" from this notice, when the tap registers, then the pro_action_client_notice_cta_tapped event fires and she is taken to her booking through the manage link that FEAT-08.SPEC-004 places in the notice.

**FEAT-30.SPEC-012-AC-13:** Given Talia's single cancellation commits, when this contract fires, then FEAT-08.SPEC-005 also receives the event so Talia's own routine-activity notification is composed there, and this spec defines no Pro-facing content.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 3 (single commit, bulk-per-booking, goodwill outcome) | 3 |
| Preference States | 2 (consent granted, consent revoked/declined) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
