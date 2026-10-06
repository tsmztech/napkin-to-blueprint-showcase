---
document_type: spec
spec_type: notification
spec_id: FEAT-15.SPEC-008
spec_name: Onboarding Welcome Confirmation
spec_slug: onboarding-welcome-confirmation
parent_feature: FEAT-15
parent_feature_name: Pro Onboarding & Setup Wizard
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Notification Spec: Onboarding Welcome Confirmation

## Overview

**Name:** Onboarding Welcome Confirmation
**ID:** FEAT-15.SPEC-008
**Type:** Notification
**Purpose:** Tells Talia, the moment her booking link goes live, that setup is done and her link is ready to share -- a one-time confirmation, not a recurring nag.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard

## Scope and Non-Goals

**In Scope:**
- The one-time welcome confirmation sent the instant the booking link activates
- Its content on both channels the product uses for Pro notifications
- Preference, delivery, retry, and expiry behavior for this one confirmation

**Non-Goals:**
- Any recurring onboarding reminder or nag if a Pro stalls mid-setup -- product-features.md's Communications field for FEAT-15 names exactly one message (the welcome confirmation); the Brief's Side-Effect Inventory defines no "nudge a stalled Pro" communication, so none is built
- Sending this confirmation to the Client -- the audience is exclusively the Pro, per the Brief's own Roles Touched column for this spec ("The Pro")
- The mechanics of text and email delivery themselves -- owned by FEAT-08.SPEC-012 (transactional text) and FEAT-08.SPEC-013 (transactional email), which this spec's content and delivery rules are handed to for actual sending
- Confirming any individual setup step's completion (e.g., "your service was added") -- each step's owning screen (FEAT-01, FEAT-02, FEAT-28, etc.) gives its own in-flow save confirmation; this spec covers only the single, final go-live moment

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, the next time Talia opens the product after go-live if she is not actively viewing FEAT-15.SPEC-003 at that instant; shown immediately if she is | Talia is mid-session in the wizard at the moment this fires (she just completed her final step), so an in-app surface reaches her with zero delay in the common case |
| Text | When Talia has an active texting consent context for her own Pro notifications, per her notification preferences (FEAT-27) -- texting is the default for Pro notifications in this product, matching the founder's own phone-first behavior described in BRIEF.md | Talia does nearly all of her Chairtime use on her phone in short bursts between clients (user-persona.md, Behavioral Context); a text reaches her even if she has already closed the app after finishing setup |
| Email | When Talia's notification preferences (FEAT-27) select email instead of, or in addition to, text for Pro notifications, or as the fallback when a text send fails (per FEAT-08.SPEC-009) | Matches the product-wide email-fallback pattern already established for every other Pro and client notification |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Booking link activates | FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) | Fires exactly once, the first time is_ready (FEAT-15.SPEC-007) becomes true for a given Pro Account and the link is activated | Pro's display name, booking_link_name, activation timestamp |

## Audience and Preferences

**Recipients:** The Pro (Talia) only -- traced to the Access Matrix's "Profile & Account Settings" and "Booking & Payment" columns, both Full for the Pro. This is a Pro-facing account-lifecycle notification; no other role in the Access Matrix (Client, Platform Operator Support) is entitled to it or to the data it carries (the Pro's own booking link).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Pro notification channels | In-app only / In-app + text / In-app + email / In-app + text + email | In-app + text | Pro Profile & Booking Page Settings (FEAT-27, notification preferences) |

This confirmation follows the same Pro-notification channel preference every other Pro notification in the product uses (product-features.md, FEAT-08's Key Capabilities: "Notify the Pro of new bookings... and anything needing attention... per the Pro's preferences in FEAT-27"); it introduces no preference control of its own, since it is a one-time event with no independent on/off switch a Pro would need -- it fires exactly once, ever, per account.

**Quiet Hours:** N/A -- this product's quiet-hours window (roughly 8am-9pm) is defined specifically for automatic client-facing appointment reminders (BRIEF.md, XBR-16), which are a recurring, potentially-many-per-day communication about someone else's appointment. This is a single, one-time, self-initiated confirmation that fires as the direct result of an action Talia herself just took (completing her final setup step); holding it until a quiet-hours window would delay Talia's own recognition of her own achievement for no protective purpose, so it is exempt on every channel it uses.

## Content Definition

**In-app:**
- **Title:** You're live, {pro_display_name}!
- **Body:** Your booking link is ready to share: {booking_link}
- **CTA:** View my link -- deep-links to FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) for this Pro's own account

**Text:**
- **Body:** Chairtime: You're live, {pro_display_name}! Your booking link is ready: {booking_link}. Share it anywhere -- your Instagram bio is a great place to start.
- **CTA:** N/A -- the link itself is the action; tapping it opens the live booking page directly (no separate in-product deep link is needed on this channel)

**Email:**
- **Subject:** Your Chairtime booking link is live
- **Body:**
  Hi {pro_display_name},

  You did it -- your Chairtime setup is complete and your booking link is ready to share:

  {booking_link}

  Add it to your Instagram bio, or send it straight to a client, and they'll be able to pick a service, choose a genuinely free time, and pay their deposit in under a minute.
- **CTA (button):** View my link -- deep-links to FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) for this Pro's own account

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {pro_display_name} | Pro Account -- display_name | Talia | "there" (greeting renders as "Hi there," / "You're live, there!") -- display_name is required before this notification can ever fire, since profile completion is one of the seven required Go-Live conditions (FEAT-15.SPEC-007), so this fallback is a defensive default that is never expected to render |
| {booking_link} | Pro Account -- booking_link_name, rendered as the full shareable URL | chairtime.app/talia-lashes | Never empty -- link activation (the trigger for this notification) cannot occur without a booking_link_name already existing on the Pro Account (FEAT-27, required before go-live) |

## Delivery Rules

**Batching:** None -- this notification is inherently singular (one Pro Account can only go live once, ever), so no batching window or key applies.
**Deduplication:** At most one Onboarding Welcome Confirmation is ever sent per Pro Account. FEAT-15.SPEC-005's activation is itself a one-directional, idempotent transition (its own Business Rules: "activation is one-directional... re-running it against an unchanged state never re-emits onboarding_completed"), so this notification's trigger event can never fire twice for the same account, and no separate deduplication key is needed beyond that guarantee.
**Retry on failure:** Text delivery failure is retried once, then falls back to email, consistent with the product-wide message-delivery pattern (FEAT-08.SPEC-009); if both fail, the in-app notification stands as the delivery of record and no alarming failure message is shown to Talia -- a delayed confirmation is a minor cosmetic gap, never a business-critical loss, since the link itself is already live and visible on FEAT-15.SPEC-003 regardless of whether this notification ever reaches her by text or email.
**Expiry:** The in-app notification never expires undelivered -- it is shown the next time Talia opens the product, however long that takes, since her account only ever goes live once and the moment is worth surfacing whenever she next returns. The text/email attempt itself does not expire in the sense of being withdrawn; per the retry rule above, it either succeeds (directly or via fallback) or silently stands aside for the in-app channel, which never expires.

## Edge Cases

- **Talia's Pro Account is closed (FEAT-29) between activation and delivery** -- Not possible in practice: activation and this notification's dispatch happen within the same processing step (per FEAT-15.SPEC-005's Processing Logic, "activate the link... and emit onboarding_completed"), and account closure requires a Pro Account that has been active for some time first; there is no meaningful window for closure to occur between the two.
- **Talia's notification preferences (FEAT-27) are changed between trigger and delivery (for example, she turns off text mid-send)** -- The preference in effect at delivery time governs, consistent with the product-wide rule that preferences are evaluated at delivery time rather than trigger time; if text is turned off before the send completes, the send is not attempted on that channel and the in-app notification (which has no off switch, per Preference Controls) still delivers.
- **Both text and email delivery fail** -- Per the Retry on failure rule, the in-app notification stands as the delivery of record; Talia is never shown an alarming failure message, since her link is already live and visible regardless of this notification's delivery outcome on any other channel.
- **Talia is actively viewing FEAT-15.SPEC-003 at the exact moment activation occurs** -- The in-app notification and the screen's own Live-state transition happen from the same activation event; Talia experiences this as the screen itself updating to show her live link, with the in-app notification available from wherever the product surfaces in-app notifications (e.g., a notification center), not as a jarring duplicate interruption on top of the screen she is already looking at.
- **Talia's booking_link_name is later renamed (FEAT-27) after this notification was already delivered** -- This notification is never re-sent or corrected; it is a one-time, point-in-time confirmation. The old link continues forwarding per XBR-27, so a Pro who shared the link from this notification's original wording is unaffected.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) | Triggered by (inbound) | Link activation is the sole trigger event for this notification |
| FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) | Navigation (outbound) | Every channel's CTA (where one exists) deep-links here |
| FEAT-27 (Pro Profile & Booking Page Settings) | References (inbound) | Supplies the Pro notification channel preference this spec follows, and the display_name and booking_link_name placeholders |
| FEAT-08.SPEC-012 (Transactional Text Messaging Capability) | References (outbound) | Actual text send and delivery-status reporting for this notification's text channel |
| FEAT-08.SPEC-013 (Transactional Email Capability) | References (outbound) | Actual email send and delivery-status reporting for this notification's email channel |

## Analytics and Success Signals

- **onboarding_welcome_confirmation_delivered** (channel: in_app / text / email) -- supports success-metrics.md: "Setup-to-Live-Link Completion"
- **onboarding_welcome_confirmation_opened** (channel) -- N/A -- no Stage 2 metric measures whether the Pro opens this specific confirmation; retained so a Pro who never engages with it (despite a live link) is observable, distinct from a Pro who never went live at all
- **onboarding_welcome_cta_tapped** (channel; destination: FEAT-15.SPEC-003) -- supports success-metrics.md: "Setup-to-Live-Link Completion"

## Acceptance Criteria

**FEAT-15.SPEC-008-AC-01:** Given Talia's booking link activates and her notification preferences are the default (in-app + text), when this notification fires, then she receives an in-app notification titled "You're live, Talia!" and a text with her booking link.

**FEAT-15.SPEC-008-AC-02:** Given Talia has set her Pro notification preference to email only, when her link activates, then she receives the email version with subject "Your Chairtime booking link is live" and no text is sent.

**FEAT-15.SPEC-008-AC-03:** Given Talia taps "View my link" from the in-app notification, when the tap registers, then she lands on FEAT-15.SPEC-003 showing her live booking link.

**FEAT-15.SPEC-008-AC-04:** Given Talia's link has already activated once, when any later process re-fires FEAT-15.SPEC-005's activation logic against the same already-active state, then this notification is not sent a second time.

**FEAT-15.SPEC-008-AC-05:** Given Talia's text delivery fails, when the retry-once policy is exhausted, then the confirmation falls back to email, consistent with FEAT-08.SPEC-009's product-wide retry pattern.

**FEAT-15.SPEC-008-AC-06:** Given both Talia's text and email delivery fail, when the failures are final, then the in-app notification stands as the delivery of record and no failure message is shown to her.

**FEAT-15.SPEC-008-AC-07:** Given Talia is actively viewing FEAT-15.SPEC-003 at the exact instant her link activates, when the activation occurs, then the screen itself transitions to the Live state and the in-app notification is available without producing a jarring duplicate interruption.

**FEAT-15.SPEC-008-AC-08:** Given Talia has no display_name set at the theoretical moment of this notification (a state that cannot occur per Go-Live's own required conditions), when the placeholder would render, then the fallback "there" is used rather than a broken or empty greeting.

**FEAT-15.SPEC-008-AC-09:** Given Talia changes her notification preference from text to email after activation fires but before the send completes, when delivery is attempted, then the preference in effect at delivery time (email) governs, not the preference at trigger time.

**FEAT-15.SPEC-008-AC-10:** Given this notification has no quiet-hours window, when Talia's link activates at 2am local time, then the text and email sends are attempted immediately rather than held.

**FEAT-15.SPEC-008-AC-11:** Given Talia later renames her booking link after this notification was delivered, when she checks the link included in the original notification, then it still works (via the forwarding behavior owned by FEAT-27/XBR-27), and this notification is never re-sent with the new name.

**FEAT-15.SPEC-008-AC-12:** Given a client (Riley) has no entitlement to this notification, when the trigger fires, then it is delivered exclusively to Talia and never to any client.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 3 (in-app, text, email) | 3 |
| Trigger Paths | 1 | 1 |
| Preference States | 4 (in-app only, in-app + text default, email only, preference changed mid-flight) | 4 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
