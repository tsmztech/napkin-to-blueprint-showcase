---
document_type: spec
spec_type: notification
spec_id: FEAT-14.SPEC-009
spec_name: Opt-Out Confirmation Message
spec_slug: opt-out-confirmation-message
parent_feature: FEAT-14
parent_feature_name: Messaging Consent Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Notification Spec: Opt-Out Confirmation Message

## Overview

**Name:** Opt-Out Confirmation Message
**ID:** FEAT-14.SPEC-009
**Type:** Notification
**Purpose:** Acknowledges to a client, immediately after a STOP reply is processed, that texting has been turned off and that their confirmations and reminders will now arrive by email instead.
**Parent Feature:** FEAT-14 -- Messaging Consent Management

## Scope and Non-Goals

**In Scope:**
- The confirmation sent after FEAT-14.SPEC-004 processes a STOP text reply
- Both channels this confirmation uses and the rationale for each

**Non-Goals:**
- A confirmation for the link-tap opt-out path -- excluded per FEAT-14.SPEC-004's own Business Rules: FEAT-14.SPEC-002's landing page is itself the confirmation for that path, so this notification exists only for the STOP-reply path, which has no screen to land on.
- Any message beyond this single acknowledgment -- this feature sends nothing else of its own (product-features.md, Communications: "N/A -- this feature governs communications rather than sending its own, aside from an opt-out confirmation acknowledgment"); ongoing confirmations and reminders remain FEAT-08's responsibility once routed to email.
- A re-grant confirmation -- excluded per this feature's Key Capabilities: only the opt-out path names a confirmation message; a re-grant's feedback is delivered in-app by FEAT-14.SPEC-001, not as a separate message.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | Always, immediately after the STOP reply is processed | This confirmation is the direct reply completing the exchange the client themselves just initiated by texting STOP -- it is the terminal message in that same reply thread, not a new proactively-initiated text subject to the ongoing consent gate FEAT-08.SPEC-011 applies to future messages; the client is reachable at the exact number and moment they just texted from |
| Email | Always, sent alongside the text reply, when the client has an email address on file for this Pro | Every future message for this relationship will arrive by email (per XBR-15, texting consent is now Revoked); sending this specific confirmation by email too means the confirmation itself models the channel the client will actually experience going forward, and reaches the client even if the text reply is not seen for any reason |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| STOP reply processed | FEAT-14.SPEC-004 (Opt-Out / STOP Processing) | Fires only for the STOP-reply trigger path of that automation, after its revoke write completes (including the already-revoked no-op case) | The replying phone number, the Pro's display_name, the Client's email (if on file), the processing timestamp |

## Audience and Preferences

**Recipients:** The Client (Riley) -- the client whose STOP reply this confirms, per the Access Matrix's Messaging & Consent = Own-only for the Client. No other role receives this confirmation; the Pro is not a party to it, consistent with the Pro's View-only relationship to a client's consent.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this confirmation has no independent opt-out or channel preference of its own | -- | -- | N/A -- suppressing the acknowledgment of a client's own STOP action would leave them uncertain whether their opt-out took effect, so this confirmation is never itself subject to a preference toggle |

**Quiet Hours:** N/A -- this confirmation is a direct, immediate reply to the client's own just-sent message, not a scheduled or ambient notification; it is exempt from any quiet-hours window in the same way a reply to an inbound message would be, since holding it would leave the client uncertain their STOP was received.

## Content Definition

**Text:**
- **Body:** You're unsubscribed from texts from {pro_display_name}. Your confirmations and reminders go to your email instead.
- **CTA:** None -- this is a terminal acknowledgment with no action to take.

**Email:**
- **Subject:** Texting turned off for {pro_display_name}
- **Body:**
  Hi,

  You replied STOP, so texting is now turned off for your bookings with {pro_display_name}.

  Your confirmations and reminders go to your email instead. If you'd like to turn texting back on later, you can do so from your booking preferences.
- **CTA:** None -- this is a terminal acknowledgment with no action to take.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia's Studio | Never empty -- display_name is a required field on every live Pro Account (product-features.md, Pro Account fields) |

## Delivery Rules

**Batching:** None -- exactly one confirmation is sent per STOP-reply processing event; there is no scenario in which multiple pending confirmations for the same client-Pro relationship could accumulate, since FEAT-14.SPEC-004's already-revoked no-op case still fires exactly one confirmation per reply received, not a suppressed or merged one.
**Deduplication:** One confirmation per processed STOP reply. A second STOP reply from the same client, even moments later, produces its own separate confirmation (per FEAT-14.SPEC-004's Business Rules: a redundant STOP still deserves acknowledgment); this is intentional non-deduplication, not a gap.
**Retry on failure:** Text delivery failure is retried per the product's standard message-delivery discipline (platform parameter: `message-delivery-retry-count`); after the final failure, the email send (already dispatched independently, not as a fallback of the text) remains the delivery of record, since both channels are sent in parallel, not in sequence. Email delivery failure has no further fallback beyond its own standard retry, since Revoked consent removes text as an alternate channel for this client going forward.
**Expiry:** This confirmation never expires unsent -- because it is a direct, immediate reply to an event that already fully completed (the revoke write), there is no future point at which sending it becomes moot; if both channels are ultimately undeliverable, the client's opt-out itself is still fully in effect regardless, since FEAT-14.SPEC-004's revoke write does not depend on this notification succeeding.

## Edge Cases

- **The client has no email address on file for this Pro** -- Only the text confirmation is sent; the email leg is simply skipped, since there is no address to send to. This is consistent with product-features.md's rule that email is required only when texts are declined at booking -- a client opting out later by STOP may not have provided one, and the text confirmation alone still fulfills the acknowledgment.
- **A second STOP reply arrives moments after the first, while the first confirmation is still being delivered** -- Per Deduplication above, a second, separate confirmation is sent for the second reply; both are legitimate acknowledgments of two real STOP actions, even if redundant from the client's perspective.
- **The underlying Messaging Consent record is deleted between the STOP reply's processing and this notification's send (a client deletion request, FEAT-13/XBR-19, arriving in the same narrow window)** -- The confirmation still sends using the phone number and email captured at trigger time, since the acknowledgment concerns the STOP action itself, which already happened; the now-deleted record does not retroactively cancel a confirmation for an action that was real when it occurred.
- **Text delivery fails but email succeeds (or vice versa)** -- Since both channels are dispatched independently rather than one as a fallback of the other, a failure on one channel has no effect on the other; the client receives the acknowledgment on whichever channel succeeds, and both failing simultaneously leaves the opt-out itself still fully in effect regardless (Delivery Rules, Expiry).
- **Quiet hours (if any exist for other notifications in the product) would otherwise apply to the moment of this send** -- Per the Quiet Hours field above, this confirmation is exempt, since it is the direct reply to the client's own just-sent message, not an ambient or scheduled notification a quiet-hours window is meant to protect against.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-004 (Opt-Out / STOP Processing) | Triggered by (inbound) | The STOP-reply path fires this notification after its revoke write completes |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | References (outbound) | Sends the text leg of this confirmation |
| FEAT-08.SPEC-013 (Transactional Email Capability) | References (outbound) | Sends the email leg of this confirmation |
| FEAT-14.SPEC-001 (Consent & Preferences) | References | The client's next view of this screen reflects the same Revoked status this confirmation describes |

## Analytics and Success Signals

- **opt_out_confirmation_sent** (channel: text / email; delivery_status) -- N/A -- no metric in success-metrics.md names Messaging Consent Management as its Connected Feature or references opt-out acknowledgment; retained as an operational signal so this confirmation's delivery reliability is observable.
- **opt_out_confirmation_delivery_failed** (channel) -- N/A -- same reason as above; retained so a silent delivery failure on this compliance-adjacent acknowledgment is never invisible.

## Acceptance Criteria

**FEAT-14.SPEC-009-AC-01:** Given Riley replies "STOP" to a text from Talia and her phone matches her Client record, when FEAT-14.SPEC-004 completes the revoke, then Riley receives a text reading "You're unsubscribed from texts from Talia's Studio. Your confirmations and reminders go to your email instead." and an email with the subject "Texting turned off for Talia's Studio", given she has an email on file.

**FEAT-14.SPEC-009-AC-02:** Given Riley has no email address on file for Talia, when the confirmation fires, then only the text leg is sent and no email is attempted.

**FEAT-14.SPEC-009-AC-03:** Given Riley's consent was already Revoked before this STOP reply (a redundant STOP), when FEAT-14.SPEC-004 processes it as a no-op, then this notification still fires and Riley receives the same confirmation.

**FEAT-14.SPEC-009-AC-04:** Given Riley texts "STOP" twice within moments, when both replies are processed, then two separate confirmations are sent, one per reply.

**FEAT-14.SPEC-009-AC-05:** Given the text leg of this confirmation fails to deliver, when the failure is reported, then the email leg, already sent independently, is unaffected and remains the delivery Riley receives.

**FEAT-14.SPEC-009-AC-06:** Given both the text and email legs of this confirmation fail to deliver, when the failures are reported, then Riley's underlying opt-out remains fully in effect regardless, since FEAT-14.SPEC-004's revoke write does not depend on this notification's success.

**FEAT-14.SPEC-009-AC-07:** Given Riley opts out via the link-tap path instead of a STOP reply, when FEAT-14.SPEC-004 processes it, then this notification does not fire, since FEAT-14.SPEC-002's landing page is itself the confirmation for that path.

**FEAT-14.SPEC-009-AC-08:** Given the product defines no preference to suppress this confirmation, when Riley looks for a way to turn it off, then no such control exists anywhere in the product.

**FEAT-14.SPEC-009-AC-09:** Given the confirmation is sent during hours that would otherwise be within any quiet-hours window defined elsewhere in the product, when the STOP reply is processed, then the confirmation sends immediately regardless, per its quiet-hours exemption.

**FEAT-14.SPEC-009-AC-10:** Given Riley's Client record is deleted in the narrow window between her STOP reply's processing and this notification's send, when the notification fires, then it still sends using the phone number and email captured at trigger time.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (no preference exists, confirmed absent) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
