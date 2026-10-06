---
document_type: spec
spec_type: notification
spec_id: FEAT-27.SPEC-013
spec_name: Help Request Acknowledgment
spec_slug: help-request-acknowledgment
parent_feature: FEAT-27
parent_feature_name: Pro Profile & Booking Page Settings
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Notification Spec: Help Request Acknowledgment

## Overview

**Name:** Help Request Acknowledgment
**ID:** FEAT-27.SPEC-013
**Type:** Notification
**Purpose:** Confirms to Talia that her help request was received, carrying the reference support's read-only lookup (FEAT-19) points to.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- The acknowledgment delivered the moment a help request is submitted, on every channel it uses
- Preference, retry, and expiry behavior for this acknowledgment

**Non-Goals:**
- Any follow-up communication about the request's resolution or status -- excluded per scope-boundaries.md SC-05 and coordination note 8: any request-status lifecycle belongs to FEAT-19's support-side handling, entirely out of scope for this feature; this notification is a one-time receipt confirmation only.
- Deciding whether the request was created -- owned by FEAT-27.SPEC-006 (Help Request); this notification begins where that screen's successful submit fires it.
- Batching multiple help requests into one acknowledgment -- product-features.md's Communications field describes a single acknowledgment per request, and each help-seeking moment is treated as its own distinct event worth its own confirmation, not a nag to be consolidated.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always when the acknowledgment is delivered | Talia is on the settings screen at the moment she submits, so an immediate in-app confirmation matches the on-screen "Your request was sent" feedback already shown by FEAT-27.SPEC-006 |
| Text | When Talia's notification preferences (FEAT-27.SPEC-005) include text | Talia does nearly all of her Chairtime use on her phone in short bursts (user-persona.md, Behavioral Context); a text reaches her even if she has already closed the app after submitting |
| Email | When Talia's notification preferences (FEAT-27.SPEC-005) include email, or as the fallback when a text send fails (per FEAT-08.SPEC-009's product-wide retry/fallback pattern) | Matches the product-wide email-fallback pattern already established for every other Pro notification |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Help request submitted | FEAT-27.SPEC-006 (Help Request) | Fires immediately when a Help Request record is successfully created | The Pro Account reference, the Help Request's reference (the identifier FEAT-19's support lookup opens against), and the submission timestamp |

## Audience and Preferences

**Recipients:** The Pro (Talia) -- the sole recipient, since a help request is her own submission and this acknowledgment confirms receipt to her alone; no other role is entitled to it (Support's own visibility into the request is a distinct, read-only capability owned by FEAT-19, not a copy of this notification).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Pro notification channels | In-app only / In-app + text / In-app + email / In-app + text + email | In-app + text | FEAT-27.SPEC-005 (Notification Preferences) |

**Quiet Hours:** N/A -- no quiet-hours window is defined anywhere in Stage 2 for Pro-directed notifications; a help request is a Pro-initiated action taken in the moment, so an immediate acknowledgment on every configured channel is the expected and least surprising behavior, with no reason to hold it.

## Content Definition

**In-app:**
- **Title:** We got your message
- **Body:** Your request was received. We'll be in touch.
- **CTA:** None -- this is a receipt confirmation with nothing further for Talia to act on; she returns to settings on her own.

**Text:**
- **Body:** Chairtime: We got your help request and will be in touch soon.

**Email:**
- **Subject:** We got your help request
- **Body:**
  Hi {pro_first_name},

  We received your message and will be in touch soon.

  Reference: {help_request_reference}
- **CTA:** None.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_first_name} | Pro Account -- display_name (first token) | Talia | Greeting renders as "Hi," -- display_name is required and non-empty on every Pro Account (FEAT-27.SPEC-001 validation), so this fallback is defensive only |
| {help_request_reference} | Help Request -- reference (the identifier created by FEAT-27.SPEC-006) | HR-20260927-0142 | Never empty -- this notification only fires after a Help Request record is successfully created, which always carries a reference |

## Delivery Rules

**Batching:** None -- each help request produces exactly one acknowledgment, delivered independently, per this spec's Non-Goals.
**Deduplication:** At most one acknowledgment per Help Request record. A retried delivery attempt after a transient failure never produces a second acknowledgment for the same request -- delivery is retried, not re-triggered.
**Retry on failure:** Text delivery failure is retried once, then falls back to email, per the product-wide message-delivery discipline (platform parameter: `message-delivery-retry-count`, FEAT-08.SPEC-009); after the final failure on every configured channel, the in-app acknowledgment (which has no delivery-failure mode of its own, since it renders directly in the product) stands as the delivery of record, and no alarming failure message is shown to Talia.
**Expiry:** This acknowledgment never expires undelivered in a way that changes its content -- if text and email both ultimately fail, the in-app confirmation remains available indefinitely the next time Talia opens the product, since it carries no time-sensitive action.

## Edge Cases

- **Talia's notification preferences are changed between her request's submission and this acknowledgment's delivery (for example, she turns off text moments after submitting)** -- The preference in effect at delivery time governs, consistent with the product-wide rule that preferences are evaluated at delivery time rather than trigger time (FEAT-27.SPEC-005's own Business Rules); if text is turned off before the send completes, it is not attempted on that channel, and the in-app confirmation (no off switch, per Preference Controls) still delivers.
- **Talia submits a help request while offline, and the request only reaches the product once connectivity returns** -- The acknowledgment fires only after the Help Request record is actually created (per its Trigger), so no acknowledgment is ever sent for a submission that has not yet succeeded; there is no separate "held" state for this notification to manage.
- **Both text and email delivery fail for this acknowledgment** -- Per Delivery Rules, the in-app confirmation stands as the delivery of record and no failure is surfaced to Talia as an alarming error; the product-wide retry/fallback discipline (FEAT-08.SPEC-009) already governs the underlying channel mechanics.
- **Talia submits two help requests in quick succession** -- Each Help Request record produces its own independent acknowledgment; per Non-Goals, they are never batched into one, since each is treated as its own distinct receipt.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-006 (Help Request) | Triggered by (inbound) | A successful submission fires this notification |
| FEAT-27.SPEC-005 (Notification Preferences) | References (inbound) | Preference control governs channel selection |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | References (outbound) | Carries this notification's text send when text is a configured channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | References (outbound) | Carries this notification's email send when email is a configured channel, or as the text-failure fallback |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (outbound) | Supplies the retry-then-fallback discipline this notification's Delivery Rules follow |
| FEAT-19 (Platform Support Read-Only Access) | References (outbound) | The help_request_reference this notification carries is the identifier FEAT-19's support lookup opens against |

## Analytics and Success Signals

- **help_request_acknowledgment_delivered** (channel: in_app / text / email) -- N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so acknowledgment delivery is observable
- **help_request_acknowledgment_opened** (channel) -- N/A -- no connected success-metrics.md metric; retained so engagement with the receipt is observable
- **help_request_acknowledgment_channel_failed** (channel, fallback_used: yes/no) -- N/A -- no connected success-metrics.md metric; retained so channel-level delivery friction is observable rather than invisible

## Acceptance Criteria

**FEAT-27.SPEC-013-AC-01:** Given Talia has default notification preferences (In-app + text) and submits a help request, when the request is created, then she receives an in-app confirmation titled "We got your message" and a text reading "Chairtime: We got your help request and will be in touch soon."

**FEAT-27.SPEC-013-AC-02:** Given Talia has set her preferences to "In-app + email", when she submits a help request, then she receives the in-app confirmation and an email with the subject "We got your help request" carrying her help request's reference.

**FEAT-27.SPEC-013-AC-03:** Given Talia has set her preferences to "In-app only", when she submits a help request, then she receives only the in-app confirmation, with no text or email sent.

**FEAT-27.SPEC-013-AC-04:** Given Talia's text delivery for this acknowledgment fails, when the retry also fails, then the acknowledgment falls back to email, per FEAT-08.SPEC-009.

**FEAT-27.SPEC-013-AC-05:** Given both text and email delivery fail for this acknowledgment, when the final failure is reached, then the in-app confirmation stands as the delivery of record and no alarming failure message is shown to Talia.

**FEAT-27.SPEC-013-AC-06:** Given Talia changes her notification preferences from "In-app + text" to "In-app only" moments after submitting a help request but before this acknowledgment is delivered, when delivery occurs, then only the in-app confirmation is delivered, per the delivery-time preference rule.

**FEAT-27.SPEC-013-AC-07:** Given Talia submits two help requests in quick succession, when both are created, then she receives two independent acknowledgments, each carrying its own request's reference -- never batched into one.

**FEAT-27.SPEC-013-AC-08:** Given Talia's email acknowledgment is delivered, when she opens it, then the body reads "Hi {pro_first_name}, We received your message and will be in touch soon." followed by her request's reference.

**FEAT-27.SPEC-013-AC-09:** Given a support operator later opens Talia's account via FEAT-19 using this acknowledgment's carried reference, when they look up the request, then it resolves to the exact Help Request record this notification's reference names.

**FEAT-27.SPEC-013-AC-10:** Given Talia's help request submission has not yet succeeded (for example, she is offline when she taps Send), when she checks for this acknowledgment, then none has been sent, since the trigger only fires after the Help Request record is actually created.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 3 (in-app, text, email) | 3 |
| Trigger Paths | 1 | 1 |
| Preference States | 4 (in-app only, in-app + text, in-app + email, in-app + text + email) | 4 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |
