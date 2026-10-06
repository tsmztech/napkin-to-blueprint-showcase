---
document_type: spec
spec_type: notification
spec_id: FEAT-20.SPEC-009
spec_name: Waitlist Expiry Notification
spec_slug: waitlist-expiry-notification
parent_feature: FEAT-20
parent_feature_name: Waitlist for Cancelled Slots
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Notification Spec: Waitlist Expiry Notification

## Overview

**Name:** Waitlist Expiry Notification
**ID:** FEAT-20.SPEC-009
**Type:** Notification
**Purpose:** Informs a client that their waitlist entry has expired -- either an unclaimed opening or an unmatched date range -- so they are never left wondering.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots

## Scope and Non-Goals

**In Scope:**
- The notification delivered when either expiry path (FEAT-20.SPEC-007) transitions an entry to Expired
- Both variants: unclaimed opening, and unmatched date range
- Delivery timing, retry, and expiry behavior for this non-time-critical send

**Non-Goals:**
- Deciding when an entry expires -- owned by FEAT-20.SPEC-007; this spec only sends the notification that automation's transitions trigger
- The opening notification itself -- owned by FEAT-20.SPEC-008, a distinct, time-critical communication with its own channel and quiet-hours behavior
- Offering a one-tap rejoin action -- product-features.md and this Brief describe no such shortcut; a client who wants back on the waitlist rejoins fresh through FEAT-20.SPEC-001, counted freshly against the cap, matching the Entity-Lifecycle Coverage Matrix's "no restore path" decision

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The client has active texting consent for this Pro (FEAT-14.SPEC-007, XBR-15) | Consistent with every other transactional message this product sends this client |
| Email | The client does not have active texting consent | The product-wide fallback channel (XBR-15) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A Notified entry expires unclaimed | FEAT-20.SPEC-007 (Waitlist Entry Expiry) | Fires when the claim-window path transitions an entry to Expired | Client reference, service name, the opening's matched date, reason: unclaimed_opening |
| A Requested entry expires unmatched | FEAT-20.SPEC-007 (Waitlist Entry Expiry) | Fires when the unmatched-range path transitions an entry to Expired | Client reference, service name, the joined date range, reason: unmatched_range |

## Audience and Preferences

**Recipients:** The Client (Riley) -- the sole recipient, matching the Access Matrix's Own-only Waitlist access; never the Pro or Platform Operator (Support).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Texting consent (channel eligibility only) | Granted / Revoked | Whatever the client's current consent state is with this Pro | FEAT-06.SPEC-005 (Consent & Email Preferences) |

There is no on/off preference for this notification -- it is a transactional communication informing the client of her own entry's outcome (scope-boundaries.md SC-15), not a discretionary or promotional send.

**Quiet Hours:** This notification carries no time pressure (the window it concerns has already closed), so it follows ASMP-29's ordinary daytime-hours rule: held if it would otherwise be delivered outside platform parameter: `reminder-window-start-hour` to platform parameter: `reminder-window-end-hour` in the Pro's timezone, and delivered at the window's start once it opens.

## Content Definition

**Text (unclaimed-opening variant):**
- **Body:** Your {claim_window_minutes}-minute window to claim the {service_name} spot on {matched_date} has passed. You're still on the waitlist for the rest of your requested range if it hasn't ended -- see your status: {my_waitlists_link}
- **CTA:** View my waitlists -- deep-links to FEAT-20.SPEC-002 (My Waitlists)

**Text (unmatched-range variant):**
- **Body:** No matching opening came up for {service_name} between {start_date} and {end_date}, so that waitlist request has ended. Want to try again? {join_waitlist_link}
- **CTA:** Join again -- deep-links to FEAT-05.SPEC-001 (service list, the entry point that leads back to FEAT-20.SPEC-001)

**Email (unclaimed-opening variant):**
- **Subject:** Your waitlist window for {service_name} has closed
- **Body:**
  Your {claim_window_minutes}-minute window to claim the {service_name} spot on {matched_date} has passed, so it's now been offered more broadly.

  If your original requested range hasn't ended yet, you're still waitlisted for it -- check your status any time.
- **CTA (button):** View my waitlists -- deep-links to FEAT-20.SPEC-002 (My Waitlists)

**Email (unmatched-range variant):**
- **Subject:** Your waitlist request for {service_name} has ended
- **Body:**
  No matching opening came up for {service_name} between {start_date} and {end_date}, so that waitlist request has ended.

  You're welcome to join again for a new date or range any time.
- **CTA (button):** Join the waitlist again -- deep-links to FEAT-05.SPEC-001

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {service_name} | Service -- name | Full Set -- Lashes | Never empty -- required at service creation (FEAT-01) |
| {claim_window_minutes} | platform parameter: `waitlist-claim-window-minutes` | 30 | Never empty -- one value for every Pro |
| {matched_date} | Waitlist Entry -- derived, the freed slot's date from the matching signal that produced the now-lapsed Notified state | Thursday, March 12 | Never empty -- only present in the unclaimed-opening variant, which only fires for an entry that was genuinely matched |
| {start_date} / {end_date} | Waitlist Entry -- start_date, end_date | March 10 / March 17 | Never empty -- required at join (FEAT-20.SPEC-003) |
| {my_waitlists_link} | Access Link (booking-specific, into FEAT-20.SPEC-002), reached via FEAT-06 | (link) | If the client's access link cannot be freshly issued at send time, the CTA still renders and routes into FEAT-06.SPEC-001 to request one, rather than omitting the link entirely |
| {join_waitlist_link} | Direct route into FEAT-05.SPEC-001 | (link) | Never empty -- FEAT-05's service list requires no identity or link state to reach |

## Delivery Rules

**Batching:** None -- each expiry is its own distinct entry outcome (a different service, date, or reason), and this Brief's own communication tone treats each as worth its own message rather than a rolled-up summary; a client with two entries expiring on the same day receives two separate notifications.
**Deduplication:** At most one expiry notification per Waitlist Entry -- an entry transitions to Expired exactly once (a terminal state), so this notification fires exactly once per entry.
**Retry on failure:** Text delivery failure is retried and falls back to email per FEAT-08.SPEC-009's product-wide retry-and-fallback rule (platform parameter: `message-delivery-retry-count`), identical to any other transactional text in this product.
**Expiry:** This notification is held, not dropped, if quiet hours are in effect at the moment it would otherwise send, and delivers at the next allowed hour; if both channels ultimately fail after the ordinary retry/fallback chain, no further attempt is made -- the client's entry still shows "Expired" on FEAT-20.SPEC-002 for one visit, which is the surviving signal even if this specific message is never delivered.

## Edge Cases

- **The underlying Waitlist Entry is later inspected and no longer shows on FEAT-20.SPEC-002 (it rolled off after one visit) by the time the client reads a delayed notification** -- The notification's content stands on its own (it names the service and dates directly) and does not depend on the entry still being visible on that screen; the CTA still routes correctly to FEAT-20.SPEC-002, which simply shows no matching row by then, an outcome consistent with that screen's own roll-off rule.
- **Quiet hours extend past the point the client might reasonably expect this notification** -- The held notification delivers at the next allowed window-start hour, exactly as any other daytime-bound notification in this product (FEAT-08.SPEC-007's pattern); there is no separate expiry cutoff for the notification itself distinct from the entry's own already-final Expired state.
- **A preference change mid-flight (texting consent is revoked between the expiry transition and the send)** -- The channel-selection rule evaluates fresh at send time (FEAT-14.SPEC-007), so a revoke landing before the send routes this notification to email automatically.
- **The client rejoins the waitlist for the same service before this notification is delivered** -- The notification still delivers as composed, describing the entry that actually expired; it is not cancelled or altered by a subsequent, unrelated new join, since the two are independent entries.
- **Both the unclaimed-opening and unmatched-range paths could plausibly apply to the same entry (a range-joined entry that was matched partway through its range and then its claim lapses)** -- Not possible under FEAT-20.SPEC-007's mutual-exclusivity rule: once an entry is Notified, it is no longer subject to the unmatched-range path at all, so only the unclaimed-opening variant ever fires for that entry.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-20.SPEC-007 (Waitlist Entry Expiry) | Triggered by (inbound) | Either expiry path fires this notification with its variant reason |
| FEAT-20.SPEC-002 (My Waitlists) | Navigation (outbound) | The unclaimed-opening variant's CTA deep-links here |
| FEAT-05.SPEC-001 (Public Booking Page & Service List) | Navigation (outbound) | The unmatched-range variant's CTA deep-links here, leading back toward FEAT-20.SPEC-001 |
| FEAT-06.SPEC-001 (Access Link Request) | Navigation (outbound) | Fallback destination when a fresh My Waitlists link cannot be issued at send time |
| FEAT-14.SPEC-007 (Textability Determination Rule) | References (inbound) | Determines the channel at send time |
| FEAT-08.SPEC-007 (Reminder Scheduling & Timing Window Enforcement) | References (inbound) | Supplies the daytime-hours pattern this notification's quiet-hours behavior follows |
| FEAT-08.SPEC-009 (Message Delivery Retry & Fallback) | References (inbound) | Governs retry and fallback behavior for this send |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | References (inbound) | Delivers the text channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | References (inbound) | Delivers the email channel |

## Analytics and Success Signals

- **waitlist_expiry_notification_delivered** (channel: text / email; reason: unclaimed_opening / unmatched_range) -- N/A -- no success-metrics.md metric directly measures waitlist expiry communication; retained so the "client is informed rather than left wondering" commitment is observable rather than assumed.
- **waitlist_expiry_notification_delivery_failed** (channel: text / email; reason) -- N/A -- no Stage 2 metric measures this failure path; retained for operational visibility, consistent with the product's own "never silently dropped" pattern for other notifications (XBR-17).

## Acceptance Criteria

**FEAT-20.SPEC-009-AC-01:** Given Riley's Notified entry's claim window lapses unclaimed, when FEAT-20.SPEC-007 expires it, then she receives the unclaimed-opening variant naming {service_name} and {matched_date}.

**FEAT-20.SPEC-009-AC-02:** Given Riley's Requested entry's joined range elapses with no match ever found, when FEAT-20.SPEC-007 expires it, then she receives the unmatched-range variant naming {service_name}, {start_date}, and {end_date}.

**FEAT-20.SPEC-009-AC-03:** Given Riley has active texting consent, when either variant fires, then she receives it by text.

**FEAT-20.SPEC-009-AC-04:** Given Riley does not have active texting consent, when either variant fires, then she receives the corresponding email variant.

**FEAT-20.SPEC-009-AC-05:** Given this notification would otherwise be sent at 11pm in the Pro's timezone, when the send is evaluated, then it is held per ASMP-29's daytime-hours rule and delivered at the next allowed hour.

**FEAT-20.SPEC-009-AC-06:** Given Riley taps "View my waitlists" on the unclaimed-opening variant, then she is routed to FEAT-20.SPEC-002.

**FEAT-20.SPEC-009-AC-07:** Given Riley taps "Join again" on the unmatched-range variant, then she is routed to FEAT-05.SPEC-001.

**FEAT-20.SPEC-009-AC-08:** Given the text send fails, when the retry-and-fallback rule processes it, then it retries per platform parameter: `message-delivery-retry-count` and falls back to email if the retry also fails.

**FEAT-20.SPEC-009-AC-09:** Given both channels ultimately fail, when no further attempt is possible, then Riley's entry still shows "Expired" on FEAT-20.SPEC-002 for one visit, even though this notification never arrived.

**FEAT-20.SPEC-009-AC-10:** Given Riley revokes texting consent between her entry's expiry and this notification's send, when the send executes, then the channel-selection rule routes it to email.

**FEAT-20.SPEC-009-AC-11:** Given Riley rejoins the waitlist for the same service before this notification is delivered, when the send proceeds, then it still describes the entry that actually expired, unaffected by the new, independent join.

**FEAT-20.SPEC-009-AC-12:** Given an entry was Notified and its claim lapsed, when this notification fires, then only the unclaimed-opening variant is sent -- the unmatched-range variant never applies to a Notified entry.

**FEAT-20.SPEC-009-AC-13:** Given a fresh My Waitlists access link cannot be issued at send time, when the unclaimed-opening variant's CTA is rendered, then it routes into FEAT-06.SPEC-001 to request one rather than omitting the link.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 2 (unclaimed opening, unmatched range) | 2 |
| Preference States | 2 (consent granted, consent revoked) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
