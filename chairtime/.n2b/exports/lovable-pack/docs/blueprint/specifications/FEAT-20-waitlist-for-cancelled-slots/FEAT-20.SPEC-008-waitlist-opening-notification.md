---
document_type: spec
spec_type: notification
spec_id: FEAT-20.SPEC-008
spec_name: Waitlist Opening Notification
spec_slug: waitlist-opening-notification
parent_feature: FEAT-20
parent_feature_name: Waitlist for Cancelled Slots
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Notification Spec: Waitlist Opening Notification

## Overview

**Name:** Waitlist Opening Notification
**ID:** FEAT-20.SPEC-008
**Type:** Notification
**Purpose:** Notifies a matching client the moment their slot opens, states the 30-minute claim window, and carries the claim link into the booking flow.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots

## Scope and Non-Goals

**In Scope:**
- The notification delivered the instant a Requested entry is matched and transitioned to Notified
- Stating the exact claim window and carrying the claim link
- Delivery timing, retry, and expiry behavior for this specific, time-critical send

**Non-Goals:**
- Deciding which entries are matched or how the claim window is computed -- owned by FEAT-20.SPEC-004/SPEC-005; this spec only sends the notification those specs' transitions trigger
- The booking flow the claim link opens into -- owned by FEAT-05; this spec only carries the client there
- Applying to entries whose window has already lapsed -- that case is FEAT-20.SPEC-009's expiry notification, a distinct communication

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The matched client has active texting consent for this Pro (FEAT-14.SPEC-007, XBR-15) | Riley's entire claim window is only 30 minutes; a text is the channel most likely to reach her within it, matching this Brief's own decision that this notification is time-critical, same as a payment confirmation |
| Email | The matched client does not have active texting consent | The fallback channel every other transactional communication in this product uses when texting consent is absent (XBR-15) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A Waitlist Entry transitions to Notified | FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching) | Fires once per matched entry, immediately at the moment of transition | Client reference, service name, matched slot date/time, claim_deadline, claim link destination |

## Audience and Preferences

**Recipients:** The Client (Riley) -- the sole recipient the Access Matrix's Waitlist row (Own-only for the Client) supports; this notification is never sent to the Pro or to Platform Operator (Support), consistent with the Pro's aggregate-only View access and Support's read-only troubleshooting access.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (channel eligibility only, not an on/off switch for this notification) | Granted / Revoked | Whatever the client's current consent state is with this Pro | FEAT-06.SPEC-005 (Consent & Email Preferences) |

There is no on/off preference for this notification itself -- it is a transactional communication about the client's own waitlist entry, not a promotional or discretionary send (scope-boundaries.md SC-15), so it cannot be turned off independently of leaving the waitlist entirely (FEAT-20.SPEC-002).

**Quiet Hours:** N/A -- this Brief's recorded reading treats the opening notification as time-critical by design (its entire value depends on the client having the full claim window to act) and therefore outside ASMP-29's daytime-hours scope, the same as a payment confirmation; it is sent the moment the match occurs, at any hour.

## Content Definition

**Text:**
- **Body:** A spot opened for {service_name} on {matched_date} at {matched_time} -- you're on the waitlist! Claim it within {claim_window_minutes} minutes before it's offered to others: {claim_link}
- **CTA:** Claim it -- deep-links to FEAT-05's booking flow (FEAT-05.SPEC-002 onward) for the opened slot

**Email:**
- **Subject:** A spot opened for {service_name} -- claim it within {claim_window_minutes} minutes
- **Body:**
  A spot just opened up for {service_name} on {matched_date} at {matched_time}, and you're on the waitlist.

  You have {claim_window_minutes} minutes to claim it before it's offered to everyone else.
- **CTA (button):** Claim this spot -- deep-links to FEAT-05's booking flow (FEAT-05.SPEC-002 onward) for the opened slot

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {service_name} | Service -- name | Full Set -- Lashes | Never empty -- required at service creation (FEAT-01) |
| {matched_date} | Waitlist Entry -- derived, the freed slot's date from the matching signal (FEAT-20.SPEC-005) | Thursday, March 12 | Never empty -- the notification is triggered only once a match with a concrete date exists |
| {matched_time} | Waitlist Entry -- derived, the freed slot's start time, in the Pro's timezone (XBR-25) | 2:30 PM | Never empty -- same as above |
| {claim_window_minutes} | platform parameter: `waitlist-claim-window-minutes` | 30 | Never empty -- one value for every Pro |
| {claim_link} | Access Link (booking-specific, issued for the claim), or the direct route into FEAT-05.SPEC-002 for the matched slot | (link) | Never empty -- generated fresh at the moment this notification is composed |

## Delivery Rules

**Batching:** None -- each matched entry produces its own independent notification the instant it is matched; multiple openings for the same client (across different entries) are never combined into one message, since each carries its own distinct slot and claim window.
**Deduplication:** At most one opening notification per Waitlist Entry per match -- a Requested entry is only ever matched and Notified once per freed slot (FEAT-20.SPEC-004/SPEC-005); if that same entry is later matched again after expiring and being rejoined fresh, the new entry produces its own new notification, entirely independent of the prior one.
**Retry on failure:** Text delivery failure is retried and falls back to email per FEAT-08.SPEC-009's product-wide retry-and-fallback rule (platform parameter: `message-delivery-retry-count`), applied here exactly as for any other transactional text -- but because this notification is time-critical, a fallback that completes after a meaningful fraction of the 30-minute window has already elapsed still delivers (a late-but-real chance to claim is better than none), and the claim_deadline itself is never extended to compensate for delivery delay.
**Expiry:** This notification is never re-sent or held past the moment it is composed -- if delivery ultimately fails on both channels, the client simply does not learn of the opening within the window; her Waitlist Entry still expires normally at claim_deadline (FEAT-20.SPEC-007) and she receives the expiry notification (FEAT-20.SPEC-009) explaining the entry expired, so she is never left permanently wondering even if this specific message never arrived.

## Edge Cases

- **The matched Waitlist Entry converts or is left (deleted) before this notification is delivered (a fast client checks FEAT-20.SPEC-002 directly and claims, or leaves, before the message lands)** -- The notification is still delivered as composed; a client who already acted on the opening by other means simply receives a redundant confirmation-adjacent message, since cancelling an in-flight send for a fast-acting client would risk the opposite failure (a client who needed the message not receiving it) and this product's correctness bar treats over-delivery as the safer default here.
- **The claim window lapses before the text retry or email fallback completes** -- The message still delivers if it can; a late-arriving notification for a lapsed window tells the client honestly that the moment has passed rather than showing a live, actionable link -- FEAT-05's own slot re-validation at the claim link's destination independently confirms whether the priority window is still open, so a stale link never grants a claim past the deadline it never should have.
- **Quiet hours vs. expiry collision** -- Not applicable: this notification carries no quiet-hours hold at all (see Quiet Hours above), so there is no collision to resolve.
- **A preference change mid-flight (texting consent is revoked between the match and the send)** -- FEAT-08.SPEC-011's channel-selection rule (via FEAT-14.SPEC-007's fresh-every-read textability determination) is evaluated at send time, not at match time, so a revoke that lands before the send routes this notification to email automatically.
- **Multiple entries for the same client are matched by the same freed slot signal (not possible under the matching test, since a slot has one service/date, but two different clients' entries can both match)** -- Each matched client's entry produces its own independent notification; this is not a batching case, since the recipients differ.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching) | Triggered by (inbound) | The transition to Notified fires this notification |
| FEAT-20.SPEC-004 (Waitlist Priority & Claim Window Rule) | References (inbound) | Supplies the claim window and the matched-slot data this content presents |
| FEAT-05 (Public Booking Page & Booking Flow) | Navigation (outbound) | The claim link's destination |
| FEAT-14.SPEC-007 (Textability Determination Rule) | References (inbound) | Determines the channel at send time |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | References (inbound) | Delivers the text channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | References (inbound) | Delivers the email channel |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (inbound) | Governs retry and fallback behavior for this send |
| FEAT-20.SPEC-006 (Waitlist Claim Conversion) | Affects (outbound) | A completed claim through this notification's link is what that automation converts |
| FEAT-20.SPEC-007 (Waitlist Entry Expiry) | References (outbound) | Governs what happens if the window lapses without a claim |

## Analytics and Success Signals

- **waitlist_opening_notification_delivered** (channel: text / email) -- N/A -- no success-metrics.md metric directly measures waitlist notification delivery; retained because a silent delivery gap here would defeat the entire feature's value proposition without any metric detecting why.
- **waitlist_opening_notification_claim_link_tapped** () -- N/A -- no Stage 2 metric names this specific tap; retained as the leading indicator behind waitlist_converted_to_booking (FEAT-20.SPEC-006).
- **waitlist_opening_notification_delivery_failed** (channel: text / email) -- N/A -- no Stage 2 metric measures delivery failure specifically for this notification; retained for operational visibility given how time-sensitive a failed send here is (ASMP-21's correctness-and-speed bar).

## Acceptance Criteria

**FEAT-20.SPEC-008-AC-01:** Given Riley's Requested entry transitions to Notified and she has active texting consent with this Pro, when this notification fires, then she receives a text stating the service, matched date/time, and the platform parameter: `waitlist-claim-window-minutes` window, with the claim link.

**FEAT-20.SPEC-008-AC-02:** Given Riley does not have active texting consent, when this notification fires, then she receives the email variant with the identical content and CTA.

**FEAT-20.SPEC-008-AC-03:** Given Riley taps the claim link, then she is routed into FEAT-05's booking flow for the exact opened slot.

**FEAT-20.SPEC-008-AC-04:** Given this notification fires at 3am in the Pro's timezone, when delivery is evaluated, then it is sent immediately, unaffected by ASMP-29's daytime-hours rule, since this Brief treats it as time-critical.

**FEAT-20.SPEC-008-AC-05:** Given the text send fails, when the retry-and-fallback rule (FEAT-08.SPEC-009) processes it, then it retries per platform parameter: `message-delivery-retry-count` and falls back to email if the retry also fails.

**FEAT-20.SPEC-008-AC-06:** Given the fallback email completes after part of the claim window has already elapsed, when delivery finishes, then the message still delivers, and the claim_deadline itself is not extended to compensate.

**FEAT-20.SPEC-008-AC-07:** Given Riley's claim window fully lapses before any channel succeeds, when the final failure is confirmed, then Riley receives no live notification for this opening, and her entry proceeds to FEAT-20.SPEC-007's expiry path, which triggers FEAT-20.SPEC-009 instead.

**FEAT-20.SPEC-008-AC-08:** Given Riley revokes texting consent between her entry being matched and this notification being sent, when the send executes, then the channel-selection rule (evaluated fresh at send time) routes the message to email.

**FEAT-20.SPEC-008-AC-09:** Given two different clients' entries both match the same freed slot, when this notification fires, then each client receives her own independent notification, never a combined or batched send.

**FEAT-20.SPEC-008-AC-10:** Given Riley leaves her waitlist entry moments after being matched but before this notification is delivered, when the send proceeds anyway, then she still receives the message, since the product does not cancel an in-flight send for a fast-acting client.

**FEAT-20.SPEC-008-AC-11:** Given Riley taps a claim link whose window has already lapsed by the time she opens it, when FEAT-05's slot re-validation runs, then it does not grant her a priority-window booking past the deadline.

**FEAT-20.SPEC-008-AC-12:** Given Riley's entry is matched a second time after a prior entry for the same service expired and she rejoined fresh, when the new match occurs, then a new, independent notification is sent, unrelated to any prior one.

**FEAT-20.SPEC-008-AC-13:** Given Talia (the Pro) is not a recipient of this notification under any condition, when a slot on her own calendar opens and matches a client, then she receives no copy of this notification -- only her own aggregate demand count on FEAT-12 reflects the activity.

**FEAT-20.SPEC-008-AC-14:** Given Support views a Pro's account via FEAT-19, when they look at this notification's delivery status, then they see it View-only, consistent with Support's read-only access to Message delivery status.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 2 (consent granted, consent revoked) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
