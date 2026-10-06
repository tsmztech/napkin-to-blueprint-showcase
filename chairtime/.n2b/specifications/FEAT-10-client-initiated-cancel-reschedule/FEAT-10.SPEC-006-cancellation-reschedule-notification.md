---
document_type: spec
spec_type: notification
spec_id: FEAT-10.SPEC-006
spec_name: Cancellation/Reschedule Notification
spec_slug: cancellation-reschedule-notification
parent_feature: FEAT-10
parent_feature_name: Client-Initiated Cancel/Reschedule
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Notification Spec: Cancellation/Reschedule Notification

## Overview

**Name:** Cancellation/Reschedule Notification
**ID:** FEAT-10.SPEC-006
**Type:** Notification
**Purpose:** Tells the client their cancellation or reschedule went through (with the exact deposit outcome) and tells the Pro that a client-initiated change happened, once FEAT-10.SPEC-004's commit succeeds.
**Parent Feature:** FEAT-10 -- Client-Initiated Cancel/Reschedule

## Scope and Non-Goals

**In Scope:**
- The trigger contract from FEAT-10.SPEC-004's successful commit into the two-audience message (client confirmation, Pro change notice) that product-features.md's Communications field for this feature declares
- Confirming that the exact content, channels, and delivery behavior for this two-audience message are the ones already defined and validated for a client-initiated change

**Non-Goals:**
- The exact message content, channel selection, and delivery rules (batching, retry, expiry) for the client-facing confirmation -- owned by FEAT-08.SPEC-004 (Booking Change & Refund Notice), which already defines the client-initiated cancellation and reschedule variants (outside-window refund, inside-window kept, outside-window carryover) in full; this spec is the trigger-and-audience contract into that content, not a second definition of it
- The exact message content and delivery rules for the Pro-facing change notice -- owned by FEAT-08.SPEC-005 (Pro Booking Activity Notification), which covers the Pro-facing counterpart for a client-initiated cancellation or reschedule; this spec does not redefine that content either
- The underlying deposit outcome determination -- owned by FEAT-09 (Cancellation & No-Show Policy Engine); this spec reports the outcome FEAT-09 determines, never derives it
- Sending or retrying the actual message -- owned by FEAT-08.SPEC-012 (text) and FEAT-08.SPEC-013 (email), the transactional messaging capability both FEAT-08.SPEC-004 and FEAT-08.SPEC-005 send through

**Content ownership (settled):** FEAT-08 owns all message content. This spec is the trigger-and-audience contract only: it names when the messages fire (FEAT-10.SPEC-004's successful commit) and who receives them (the client and the Pro), and it defines no wording, placeholders, or variants of its own. FEAT-08.SPEC-004 (client confirmation) and FEAT-08.SPEC-005 (Pro change notice) list this spec in their Connected Specs as the trigger-and-audience contract, so no duplicate content exists anywhere in the package.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Text | The recipient (client or Pro) has active Messaging Consent for texting (governed by FEAT-08.SPEC-011 for the client; the Pro's own notification preferences for the Pro) | A cancellation or reschedule is time-sensitive and financially material; both parties should learn of it wherever they already receive booking messages, per FEAT-08.SPEC-004/FEAT-08.SPEC-005 |
| Email | The recipient has not granted texting consent, or texting fails | Ensures the notice always reaches its recipient, per BRIEF.md's stated email fallback, mirroring FEAT-08.SPEC-004/FEAT-08.SPEC-013's fallback behavior |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Client's cancellation commits successfully | FEAT-10.SPEC-004 (Booking Update Commit) | Always, on a successfully saved client-initiated cancellation | Booking (updated state), Deposit Transaction (outcome once FEAT-09.SPEC-004 evaluates it), Cancellation Policy (window_hours, wording) |
| Client's reschedule commits successfully | FEAT-10.SPEC-004 (Booking Update Commit) | Always, on a successfully saved client-initiated reschedule (outside-window carryover or inside-window compound) | Booking (updated or new record, new start_time), Deposit Transaction (outcome), Cancellation Policy |

## Audience and Preferences

**Recipients:** The Client tied to the Booking (Access Matrix: Cancellation & No-Show Handling = Own-only), for the confirmation; the Pro (Talia) who owns the Booking (Access Matrix: Cancellation & No-Show Handling = Full), for the change notice. Both are entitled to this exact data: the client to their own booking's outcome, the Pro to the resulting state of their own booking.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Client texting consent (governs channel, not whether the confirmation sends) | Granted / Revoked | Captured at booking | FEAT-06 at booking; changed via FEAT-14 |
| Pro notification preferences (governs channel for the change notice) | In-app / text / email combinations | Set during onboarding | FEAT-27 (Pro Profile & Booking Page Settings) |

This notification carries no separate opt-out on either side: a change to the client's own paid booking, and the resulting change to the Pro's own schedule, are each transactional information neither party can decline to receive -- consistent with FEAT-08.SPEC-004's and FEAT-08.SPEC-005's treatment of the same fact.

**Quiet Hours:** N/A -- this is the direct, expected report of a change that just happened, not a discretionary interruption; it sends immediately regardless of time of day, exactly as FEAT-08.SPEC-004/FEAT-08.SPEC-005 define for this same event class. XBR-16's daytime window applies only to the discretionary pre-appointment reminder (FEAT-08.SPEC-002), never to this transactional notice.

## Content Definition

**Client confirmation (all variants):** Content, exact wording, and placeholders are FEAT-08.SPEC-004's own defined variants for a client-initiated change:
- Client cancellation, outside window (full refund) -- FEAT-08.SPEC-004's "client-initiated cancellation, outside window" text/email variant
- Client cancellation, inside window (deposit kept) -- FEAT-08.SPEC-004's "client-initiated cancellation, inside window" text/email variant
- Client reschedule, outside window (deposit carried over) -- FEAT-08.SPEC-004's "client-initiated reschedule, outside window" text/email variant
- Client reschedule, inside window (late reschedule, compound outcome) -- FEAT-08.SPEC-004's edge-case content: the original deposit is kept, reported distinctly from the new booking's own confirmation (FEAT-08.SPEC-001) for the new deposit

Each variant's CTA -- "Manage my booking" -- deep-links to FEAT-06.SPEC-004 (Booking Detail via Manage Link) for the affected booking (the original booking for a cancellation, the new booking for a late reschedule once it exists).

**Pro change notice (all variants):** Content, exact wording, and placeholders are FEAT-08.SPEC-005's (Pro Booking Activity Notification) own defined variant for a client-initiated cancellation or reschedule, reported on the Pro's dashboard and, per the Pro's own notification preferences, by text or email.

**Placeholders:** Declared and owned by FEAT-08.SPEC-004 (client confirmation) and FEAT-08.SPEC-005 (Pro change notice); this spec introduces no placeholder of its own. See those specs' Placeholders tables for the exact entity/field sources and empty-value fallbacks.

## Delivery Rules

**Batching:** None -- matching FEAT-08.SPEC-004's rule, each change event (this cancellation or this reschedule) produces its own single notice per audience at the moment FEAT-10.SPEC-004's commit succeeds.
**Deduplication:** At most one client confirmation and one Pro change notice per commit event, per FEAT-08.SPEC-004/FEAT-08.SPEC-005's deduplication rule; a rejected or failed commit attempt (FEAT-10.SPEC-004's conflict, slot-lost, or failure outcomes) never reaches this trigger and so never produces a notice.
**Retry on failure:** Governed by FEAT-08.SPEC-009: a failed text is retried once, then falls back to email, with the gap flagged on the Pro's dashboard for the client-side send; the Pro's own delivery failure is handled the same way.
**Expiry:** None -- matching FEAT-08.SPEC-004/FEAT-08.SPEC-005, a change or refund fact never becomes not-worth-delivering.

## Edge Cases

- **A late reschedule's compound outcome (the original deposit kept, plus the new booking's own deposit)** -- The client receives this notice's confirmation for the original booking's outcome, and a separate booking confirmation (FEAT-08.SPEC-001) once the new booking's deposit is paid; the two are never merged into one ambiguous message, per FEAT-08.SPEC-004's own edge case for this exact scenario.
- **FEAT-10.SPEC-004's commit is rejected by a conflicting Pro-side transition (eligibility conflict outcome)** -- No notice is triggered from this spec for the rejected attempt; whichever transition actually committed produces its own single notice through its own owning trigger (FEAT-10.SPEC-004 for the client side, FEAT-30 for the Pro side), never two conflicting notices for the same terminal state, per FEAT-08.SPEC-004's Contention-aware edge case.
- **The client's texting consent is revoked between the commit and this notice sending** -- The confirmation honors the consent state current at send time (FEAT-08.SPEC-011), routing to email if consent is no longer active.
- **An automatic refund entering "in progress" rather than completing immediately** -- Covered by FEAT-08.SPEC-004's own refund-in-progress variant and its follow-up notice once the refund completes; this spec's trigger is the same FEAT-10.SPEC-004 commit event that starts that chain via FEAT-09's refund handoff.
- **A cancellation commit succeeds but FEAT-20's waitlist notification or FEAT-04's calendar mirror has not yet completed** -- This notice is not held for either: the client and Pro notices report the booking change itself and never wait on downstream cross-feature effects that have their own independent timing.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-10.SPEC-004 (Booking Update Commit) | Triggered by (inbound) | A successful cancellation or reschedule commit fires this notification's trigger contract |
| FEAT-08.SPEC-004 (Booking Change & Refund Notice) | Content owner (outbound) | Owns the exact client-facing content, channels, and delivery rules this spec's trigger feeds; lists this spec in its Connected Specs as a trigger-and-audience contract |
| FEAT-08.SPEC-005 (Pro Booking Activity Notification) | Content owner (outbound) | Owns the exact Pro-facing content, channels, and delivery rules this spec's trigger feeds; lists this spec in its Connected Specs as a trigger-and-audience contract |
| FEAT-09 (Cancellation & No-Show Policy Engine) | References (inbound) | Supplies the deposit outcome the client-facing content reports |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Navigation (outbound) | The client confirmation's CTA deep-links here |
| FEAT-08.SPEC-012 / FEAT-08.SPEC-013 (Text / Email Capabilities) | Triggers (outbound, via FEAT-08.SPEC-004/005) | Perform the actual sends |

## Analytics and Success Signals

- **client_change_notice_sent** (change_type: cancel / reschedule; window_state: outside / inside) -- supports success-metrics.md: "Self-Service Reschedule Rate"
- **pro_change_notice_sent** (change_type: cancel / reschedule) -- supports success-metrics.md: "Automatic Refund Correctness" (the Pro-visible record of the outcome is part of what makes the refund path observable and trustworthy)
- **client_change_notice_cta_tapped** (destination: manage_link) -- supports success-metrics.md: "Self-Service Access Success"

## Acceptance Criteria

**FEAT-10.SPEC-006-AC-01:** Given Riley's cancellation commits successfully outside the window, when FEAT-10.SPEC-004 completes, then this trigger fires FEAT-08.SPEC-004's outside-window cancellation variant to Riley and FEAT-08.SPEC-005's cancellation variant to Talia.

**FEAT-10.SPEC-006-AC-02:** Given Riley's cancellation commits successfully inside the window, when FEAT-10.SPEC-004 completes, then this trigger fires FEAT-08.SPEC-004's inside-window (deposit kept) variant to Riley.

**FEAT-10.SPEC-006-AC-03:** Given Riley's reschedule commits successfully outside the window, when FEAT-10.SPEC-004 completes, then this trigger fires FEAT-08.SPEC-004's outside-window reschedule variant to Riley, showing the new time and the carried-over deposit.

**FEAT-10.SPEC-006-AC-04:** Given Riley's reschedule commits successfully inside the window (late reschedule, compound outcome), when FEAT-10.SPEC-004 completes, then Riley receives the notice that her original deposit is kept, reported separately from the new booking's own confirmation once its deposit is paid.

**FEAT-10.SPEC-006-AC-05:** Given a client-initiated cancellation or reschedule commits successfully, when the commit completes, then Talia receives a Pro-facing change notice reflecting the update, via FEAT-08.SPEC-005.

**FEAT-10.SPEC-006-AC-06:** Given FEAT-10.SPEC-004's commit is rejected by a conflicting Pro-side transition, when the rejection occurs, then this spec triggers no notice for the rejected attempt.

**FEAT-10.SPEC-006-AC-07:** Given Riley has revoked texting consent since booking, when her cancellation or reschedule confirmation is triggered, then it is sent by email, honoring her current consent state.

**FEAT-10.SPEC-006-AC-08:** Given a text confirmation to Riley fails to deliver, when FEAT-08.SPEC-009's retry-then-fallback runs, then Riley still receives the confirmation by email.

**FEAT-10.SPEC-006-AC-09:** Given Riley taps "Manage my booking" from her confirmation, when the tap registers, then the client_change_notice_cta_tapped event fires and she is taken to FEAT-06.SPEC-004 for the affected booking.

**FEAT-10.SPEC-006-AC-10:** Given an automatic refund from Riley's outside-window cancellation cannot complete immediately, when FEAT-09 sets the Deposit Transaction to Refund in Progress, then Riley receives FEAT-08.SPEC-004's refund-in-progress notice as this trigger's continuation.

**FEAT-10.SPEC-006-AC-11:** Given Riley's cancellation frees a slot that a waitlisted client later claims, when the cancellation notice is delivered, then it is not held or delayed waiting for FEAT-20's waitlist notification to complete first.

**FEAT-10.SPEC-006-AC-12:** Given both the client confirmation and Pro change notice are due for the same commit event, when FEAT-10.SPEC-004 completes, then both are triggered from that single event, never batched together or deduplicated against each other, since they serve two distinct recipients.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (text, email) | 2 |
| Trigger Paths | 2 (cancel commit, reschedule commit) | 2 |
| Preference States | 2 (consent granted, consent revoked) | 2 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
