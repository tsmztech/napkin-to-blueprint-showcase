---
document_type: spec
spec_type: notification
spec_id: FEAT-07.SPEC-002
spec_name: Plan-Ready Notification Message
spec_slug: plan-ready-notification-message
parent_feature: FEAT-07
parent_feature_name: Weekly Plan Ready Notification
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Notification Spec: Plan-Ready Notification Message

## Overview

**Name:** Plan-Ready Notification Message
**ID:** FEAT-07.SPEC-002
**Type:** Notification
**Purpose:** Tells each eligible household member that next week's plan is ready, by device notification or email, so the weekly plan is never invisible until someone happens to check.
**Parent Feature:** FEAT-07 -- Weekly Plan Ready Notification

## Scope and Non-Goals

**In Scope:**
- The recurring-week and first-plan variants of the "plan is ready" message, on both its channels (device notification and email)
- Preference, deduplication, retry, and expiry behavior for this message
- The tap-through/CTA behavior that lands the member on the new week's plan

**Non-Goals:**
- Deciding who is eligible, the once-per-week cap, and which channel a given member resolves to -- owned by FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules); this spec begins once FEAT-07.SPEC-001 hands it a resolved recipient and channel.
- The mechanics of delivering a device notification or a transactional email -- owned by FEAT-07.SPEC-005 (Device-Notification Delivery Integration) and FEAT-07.SPEC-006 (Plan-Ready Email Fallback Integration); this spec defines the message content and delivery rules those integrations carry out.
- Configurable notification frequency or an editable message beyond "Next week's plan is ready" -- excluded per product-features.md's Validation & Limits field for this feature, which caps this feature at exactly one message per household per week tied to generation completion, not user action.
- In-product messaging or chat as a delivery route -- excluded per scope-boundaries.md SC-14: this message travels by device notification and email fallback only, never through an in-app chat or messaging surface.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Push (device notification) | The eligible member has device notifications available and enabled, per FEAT-07.SPEC-003's channel-fallback rule | Maya's and Sam's main touchpoint for this feature is exactly this moment -- a device notification arriving so they can open the plan from wherever they are (user-persona.md, Behavioral Context) |
| Email | The eligible member does not have device notifications available or enabled, or the device-notification delivery capability itself is unavailable that week, per FEAT-07.SPEC-003 | Device notifications from a responsive web app are not available on every phone; email keeps the Sunday rhythm the product promises reaching the member anyway (product-features.md, Communications) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Eligible member resolved for dispatch | FEAT-07.SPEC-001 (Plan-Ready Notification Trigger) | Fires once per eligible member, per household, per week, after eligibility and channel resolution complete | Recipient Member Profile (display_name, sign_in email), household id, week identifier, generation kind (recurring / first-plan), resolved channel |

## Audience and Preferences

**Recipients:** Maya (Organiser) and Sam (Other Adult Member) -- the two roles with a Notification Prefs entry in the Access Matrix. Neither kid row ever receives this message: the young-kid profile has no login at all, and the Later-phase older-kid login carries no Notification Prefs entitlement (Access Matrix, Notification Prefs column: None for both kid rows). Riley (Operator, support) has no Notification Prefs access and never receives this or any other notification.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Plan-ready notifications | On / Off | On | FEAT-01.SPEC-005 (Member Profile Detail) -- each adult controls only their own toggle |

**Quiet Hours:** N/A -- Plateful defines no quiet-hours window for any notification (no ASMP, XBR, or Communications field establishes one anywhere in the product). This message additionally fires at most once per week, at the organiser-chosen plan-arrival day and time (FEAT-07.SPEC-004) -- a moment the organiser has already deliberately chosen as convenient, so layering a further quiet-hours hold on top of a time the household picked for itself would work against that choice.

## Content Definition

**Push -- recurring week:**
- **Title:** Next week's plan is ready
- **Body:** Tap to see next week's dinners.
- **CTA:** Opens the new week's plan -- deep-links to FEAT-03.SPEC-001 (Weekly Plan View)

**Push -- first-plan (household's first-ever generated plan):**
- **Title:** Your first week's plan is ready
- **Body:** Tap to see your first week of dinners.
- **CTA:** Opens the new week's plan -- deep-links to FEAT-03.SPEC-001 (Weekly Plan View)

**Email -- recurring week:**
- **Subject:** Next week's plan is ready
- **Body:**
  Hi {member_first_name},

  Next week's plan is ready to review.

  Open Plateful to see the week's dinners.
- **CTA (button):** View this week's plan -- deep-links to FEAT-03.SPEC-001 (Weekly Plan View)

**Email -- first-plan:**
- **Subject:** Your first week's plan is ready
- **Body:**
  Hi {member_first_name},

  Your household's first week of dinners is ready to review.

  Open Plateful to see the plan.
- **CTA (button):** View your plan -- deep-links to FEAT-03.SPEC-001 (Weekly Plan View)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {member_first_name} | Member Profile -- display_name | Sam | Greeting renders as "Hi there," -- display_name is required for every Member Profile (FEAT-01), so this fallback is never expected to trigger in practice, but is defined for completeness |

## Delivery Rules

**Batching:** N/A -- at most one plan-ready message ever exists per household per week (XBR-12), and each eligible member receives exactly one instance of it; no scenario produces two pending instances for the same member to collapse together.
**Deduplication:** At most one message per member per household-week, guaranteed by FEAT-07.SPEC-001's once-per-household-per-week firing cap (enforced via FEAT-07.SPEC-003). A duplicate or retried generation-completion signal for the same week never produces a second dispatch to any member.
**Retry on failure:** Push -- delivery to a device that is offline is queued and redelivered automatically once the device reconnects, per FEAT-07.SPEC-005's delivery contract; this is queuing, not a failure retry, since XBR-12 requires the plan's in-app availability to never depend on it. If the resolved channel cannot reach the member at all for a reason other than being temporarily offline (device unregistered, permission revoked, or the device-notification delivery capability itself down), no push retry is attempted -- FEAT-07.SPEC-003's channel-fallback rule already routes that member to Email instead. Email -- delivery failure is retried up to 3 times over 6 hours (FEAT-07.SPEC-006's transactional email delivery contract). After the final email failure, no further channel is attempted for that member that week; the household still sees the plan in-app immediately regardless (XBR-12), and the failure is never surfaced to the household as an error.
**Expiry:** A push notification queued for an offline device is delivered whenever the device reconnects, with no separate time cutoff, up until the following week's plan-ready message becomes due; if the device has not reconnected by then, the stale prior-week instance is discarded rather than delivered alongside the new week's message, since only the current week's message is ever meaningful (deduplication takes precedence). Email carries no separate expiry beyond its retry window -- a send that eventually succeeds within the 6-hour retry window still delivers an accurate message, since "next week's plan is ready" remains true for the whole week it names.

## Edge Cases

- **The device-notification delivery capability is fully unavailable when this message dispatches** -- Per ASMP-31, delivery degrades to email: FEAT-07.SPEC-003 treats every affected member as "device notifications unavailable" for that week, and Email carries the message for them instead of Push.
- **A member turns their plan-ready preference off after FEAT-07.SPEC-001 has already resolved them as eligible but before this message actually sends** -- FEAT-07.SPEC-001's eligibility check runs at dispatch time, not at generation time, so a preference turned off before dispatch means no message reaches that member this week; a preference turned off after dispatch has already sent has no effect on the message already delivered.
- **The household is deleted between generation completion and this message's dispatch** -- The dispatch is cancelled silently on every channel for every member; a household that no longer exists is never notified about a plan.
- **Both Push and Email fail for the same member in the same week** -- The plan remains fully visible in-app immediately, unaffected by the notification path (XBR-12); no further channel is attempted and no error is shown to the household, since this notification's own failure must never surface as a product error.
- **A member's device reconnects only after the following week's plan-ready message has already been dispatched** -- Per Expiry, the stale queued prior-week push is discarded rather than delivered; the member instead receives (or has already received) the current week's message through its own dispatch.
- **A generation retry (FEAT-03.SPEC-003's "Retry succeeds" outcome) completes for the same week that already fired this message** -- Deduplication (via FEAT-07.SPEC-001's firing cap) ensures the retry's completion signal never triggers a second message for that week.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-001 (Plan-Ready Notification Trigger) | Triggered by (inbound) | Resolves the eligible recipient list and each member's channel, then dispatches this message |
| FEAT-07.SPEC-003 (Plan-Ready Delivery & Eligibility Rules) | References (inbound) | Eligibility, once-per-week cap, and channel-fallback rules this message's dispatch depends on |
| FEAT-01.SPEC-005 (Member Profile Detail) | References (inbound) | Hosts the plan-ready preference toggle this message's audience depends on |
| FEAT-07.SPEC-005 (Device-Notification Delivery Integration) | References (outbound) | Carries out Push delivery, offline queuing, and reconnect redelivery for this message |
| FEAT-07.SPEC-006 (Plan-Ready Email Fallback Integration) | References (outbound) | Carries out Email delivery and retry for this message |
| FEAT-03.SPEC-001 (Weekly Plan View) | Navigation (outbound) | Every CTA on every variant deep-links here |
| FEAT-03.SPEC-003 (Scheduled Weekly Plan Generation) | References (inbound) | Its completion, via FEAT-07.SPEC-001, is the ultimate source of the recurring-week variant |
| FEAT-03.SPEC-004 (First-Plan Generation on Upgrade) | References (inbound) | Its completion, via FEAT-07.SPEC-001, is the ultimate source of the first-plan variant |

## Analytics and Success Signals

- **plan_ready_notification_sent** (channel: push / email; generation_kind: recurring / first_plan) -- supports success-metrics.md: "Weekly Plan Ready Notification Reach"
- **plan_ready_notification_opened** (channel: push / email) -- supports success-metrics.md: "Weekly Plan Ready Notification Reach"
- **plan_ready_email_sent** (generation_kind: recurring / first_plan) -- supports success-metrics.md: "Weekly Plan Ready Notification Reach" (the email-channel breakdown of plan_ready_notification_sent, tracked separately per product-features.md's own Signals field for this feature)
- **plan_ready_notification_delivery_degraded** (channel; reason: capability_down / device_unreachable) -- N/A -- no Stage 2 metric measures delivery degradation frequency specifically; retained so silent delivery loss on this milestone message is observable rather than invisible.

## Acceptance Criteria

**FEAT-07.SPEC-002-AC-01:** Given Maya has device notifications enabled and default preferences, when FEAT-07.SPEC-001 resolves her as eligible for the recurring week, then she receives a Push notification titled "Next week's plan is ready."

**FEAT-07.SPEC-002-AC-02:** Given Sam does not have device notifications available, when FEAT-07.SPEC-001 resolves him as eligible for the recurring week, then he receives an email with the subject "Next week's plan is ready" instead of a push notification.

**FEAT-07.SPEC-002-AC-03:** Given a household's first-ever plan generation completes, when eligible members are dispatched, then they receive the first-plan-framed variant ("Your first week's plan is ready"), not the recurring variant.

**FEAT-07.SPEC-002-AC-04:** Given Maya taps the Push notification, when it opens, then she lands directly on the new week's plan (FEAT-03.SPEC-001).

**FEAT-07.SPEC-002-AC-05:** Given Sam taps the email's "View this week's plan" button, when it opens, then he lands directly on the new week's plan (FEAT-03.SPEC-001).

**FEAT-07.SPEC-002-AC-06:** Given Sam has turned his plan-ready preference off, when generation completes, then he receives no message on any channel, per FEAT-07.SPEC-001's eligibility check.

**FEAT-07.SPEC-002-AC-07:** Given the device-notification delivery capability is unavailable this week, when Maya would otherwise receive a Push message, then she receives the Email variant instead.

**FEAT-07.SPEC-002-AC-08:** Given a duplicate generation-completion signal arrives for a household-week that already dispatched this message, when the duplicate is processed, then no second message reaches any member.

**FEAT-07.SPEC-002-AC-09:** Given Maya's device is offline when the Push message is queued, when her device reconnects later the same week, then the queued message is delivered at that point.

**FEAT-07.SPEC-002-AC-10:** Given an email send to Sam fails transiently, when it is retried within the 6-hour window and the retry succeeds, then only one email reaches him, and no error appears to the household in the meantime.

**FEAT-07.SPEC-002-AC-11:** Given both Push and Email fail for Sam in the same week, when the final failure occurs, then no error is shown to the household and the plan remains fully visible to Sam in-app.

**FEAT-07.SPEC-002-AC-12:** Given Maya's device has not reconnected by the time the following week's plan-ready message becomes due, when that following week's message dispatches, then the stale prior-week queued push is discarded rather than delivered alongside it.

**FEAT-07.SPEC-002-AC-13:** Given Maya or Sam looks for a quiet-hours setting for this message, when they check their notification preferences (FEAT-01.SPEC-005), then no such control exists, consistent with the product defining no quiet hours for any notification.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (push, email) | 2 |
| Trigger Paths | 1 (resolved eligible member from FEAT-07.SPEC-001, with recurring and first-plan variants) | 1 |
| Preference States | 3 (on/push, on/email-fallback, off) | 3 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 6 | 6 |
