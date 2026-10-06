---
document_type: spec
spec_type: notification
spec_id: FEAT-28.SPEC-007
spec_name: Payout Status Notification
spec_slug: payout-status-notification
parent_feature: FEAT-28
parent_feature_name: Payout Account Connection & Payout Visibility
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Notification Spec: Payout Status Notification

## Overview

**Name:** Payout Status Notification
**ID:** FEAT-28.SPEC-007
**Type:** Notification
**Purpose:** Notifies Talia when verification completes and the booking link can go live, and when the payout account needs action, so she never discovers either condition only by happening to open the dashboard.
**Parent Feature:** FEAT-28 -- Payout Account Connection & Payout Visibility

## Scope and Non-Goals

**In Scope:**
- The "verification complete" notification, fired when the Payout Account first becomes Active
- The "needs action" notification, fired when the Payout Account becomes Action Required
- In-app delivery for both, matching the account-health pattern already used for calendar connection alerts (FEAT-04)
- Preference, deduplication, and expiry behavior for both notifications

**Non-Goals:**
- The "a refund cannot be completed yet" message named in product-features.md's Communications field -- this is a cross-feature disposition owned by FEAT-09/FEAT-30 (the retry and the Pro-facing flag) under XBR-10, delivered through FEAT-08.SPEC-006 (Pro Attention Alert); this feature's role in that message is limited to reflecting the "in progress" state in the money list (FEAT-28.SPEC-005), which is why this spec covers exactly two notifications, not three
- Routine booking activity of any kind -- owned by FEAT-08.SPEC-005 (Pro Booking Activity Notification); this spec is reserved for the Payout Account's own two status transitions
- Displaying the status banner itself -- owned by FEAT-28.SPEC-002 (Payout Status & Money Dashboard); this spec only fires the notification that draws Talia's attention to it

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, for both notifications | Matches the product's established pattern for account-health conditions (product-features.md, FEAT-04: "A dashboard alert (not a text/email) when a connection needs reconnecting"); the payout account's own status is exactly this kind of account-health condition, and Talia's attention list (FEAT-12) is the canonical place she checks for anything needing action |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Payout Account becomes Active | FEAT-28.SPEC-003 (Payout Account Status Processing) | Fires when status transitions to Active from any other status, whether on first connection or after resolving an Action Required condition | Pro Account reference, previous status |
| Payout Account becomes Action Required | FEAT-28.SPEC-003 (Payout Account Status Processing) | Fires when status transitions to Action Required from any other status | Pro Account reference, the capability's reported reason |

## Audience and Preferences

**Recipients:** The Pro (Talia) whose Payout Account the condition concerns (Access Matrix: Payouts = Full for the Pro). Platform Operator (Support) has View-only access to the resulting status through FEAT-28.SPEC-002 but is never a recipient of this notification, consistent with SC-05 and the Access Matrix.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- this notification cannot be turned off | Always on, in-app only | Always on | N/A -- consistent with FEAT-04's calendar reconnection alert and FEAT-08.SPEC-006's treatment of in-app delivery, an account-health condition affecting whether Talia can be paid at all is never a mutable preference |

**Quiet Hours:** N/A -- product-features.md defines no quiet-hours window for Pro-facing account-health alerts (ASMP-29's quiet-hours discipline applies to client-facing reminders, not Pro account status), and a condition that gates whether Talia's booking link can go live or take deposits is exactly the kind of time-sensitive information a delay would work against.

## Content Definition

**In-app (verification complete):**
- **Title:** Your payout account is active
- **Body:** Verification is complete. If your booking link isn't live yet, you're ready to finish setup and start taking bookings.
- **CTA:** View payouts -- deep-links to FEAT-28.SPEC-002 (Payout Status & Money Dashboard)

**In-app (needs action):**
- **Title:** Your payout account needs attention
- **Body:** {reason}. Resolve it to keep your booking link live and your money flowing.
- **CTA:** Resolve now -- deep-links to FEAT-28.SPEC-002 (Payout Status & Money Dashboard), which carries Talia into FEAT-28.SPEC-006's resolution flow

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {reason} | Payout Account -- the capability's reported Action Required reason, surfaced via FEAT-28.SPEC-006's Inbound Events | Your bank account details couldn't be verified | If the capability reports no specific reason, renders as "Your payout account needs a quick update" |

## Delivery Rules

**Batching:** Not batched -- each of the two notifications concerns a distinct, singular condition on the one Payout Account a Pro can have (FEAT-28.SPEC-004); there is never more than one open instance of either notification to batch.
**Deduplication:** At most one active "needs action" notification per open Action Required condition -- a status report that leaves the account at Action Required (e.g., a second rejected attempt with a different reason) updates the existing notification's content rather than creating a new one, consistent with FEAT-08.SPEC-006's treatment of a persistently unresolved condition. The "verification complete" notification fires once per transition to Active; if Active is reported again with no intervening non-Active status (per FEAT-28.SPEC-003's deduplication), no second notification fires.
**Retry on failure:** N/A -- in-app delivery has no separate retry: the notification is written to Talia's in-app attention list and is delivered the moment she next opens the product, with no failure mode of its own beyond the product being unreachable, which is outside this notification's own scope.
**Expiry:** Neither notification expires undelivered. "Needs action" remains on the attention list until Talia resolves the condition (it clears then, not before). "Verification complete" remains visible until Talia acknowledges it by viewing FEAT-28.SPEC-002 or explicitly dismissing it; either way, the underlying Active status is durably visible on FEAT-28.SPEC-002 regardless of whether the notification itself is ever opened.

## Edge Cases

- **The Payout Account's Pro Account closes before the notification is acted on** -- The notification is not delivered, since no active Pro session exists to receive it (mirroring FEAT-28.SPEC-006's edge case for the same underlying event); the closure process itself governs the account's fate independently.
- **Status flips from Action Required back to Active and then back to Action Required again in quick succession** -- Each transition fires its own notification per the Trigger table; a "verification complete" notification and a later "needs action" notification are never merged into one, since they name opposite conditions.
- **Talia dismisses the "needs action" notification without resolving the underlying condition** -- Dismissing the notification does not clear the Action Required status; the banner on FEAT-28.SPEC-002 remains until the condition actually resolves, and dismissing again is possible if the notification re-surfaces after further reason updates.
- **Two "needs action" reports with different reasons arrive close together for the same open condition** -- Per Deduplication, the single active notification's content updates to the latest reported reason rather than producing two notifications.
- **The account reaches Active for the very first time (first connection, not a resolution)** -- The same "verification complete" content and CTA apply; there is no separate first-time variant, since the message ("verification is complete... you're ready to finish setup") already covers both the first-connection and post-resolution cases identically.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-28.SPEC-003 (Payout Account Status Processing) | Triggered by (inbound) | Both status transitions this spec covers originate from that automation |
| FEAT-28.SPEC-002 (Payout Status & Money Dashboard) | Navigation (outbound) | Both CTAs deep-link here |
| FEAT-28.SPEC-006 (Payout Account Connection & Verification) | References (outbound) | The "needs action" reason placeholder is sourced from that spec's Inbound Events |
| FEAT-15 (Pro Onboarding & Setup Wizard) | References (outbound) | The "verification complete" body references finishing setup, which is FEAT-15's own remaining flow |
| FEAT-12 (Pro Daily Schedule Dashboard) | References (inbound) | Both notifications surface as items on that feature's attention list |
| FEAT-08.SPEC-006 (Pro Attention Alert) | References (outbound) | The distinct "refund cannot be completed yet" message is delivered there instead, per this spec's Non-Goals |

## Analytics and Success Signals

- **payout_status_notification_sent** (condition: verification_complete / needs_action) -- supports success-metrics.md: "Payout Transparency"
- **payout_status_notification_opened** (condition) -- supports success-metrics.md: "Payout Transparency"
- **payout_status_notification_cta_tapped** (condition) -- supports success-metrics.md: "Setup-to-Live-Link Completion" (the verification-complete CTA leads directly back into finishing setup)

## Acceptance Criteria

**FEAT-28.SPEC-007-AC-01:** Given Talia's Payout Account transitions to Active for the first time, when FEAT-28.SPEC-003 fires this notification, then she receives an in-app notification titled "Your payout account is active" with a "View payouts" CTA to FEAT-28.SPEC-002.

**FEAT-28.SPEC-007-AC-02:** Given Talia's Payout Account transitions to Action Required with a specific reason, when FEAT-28.SPEC-003 fires this notification, then she receives an in-app notification titled "Your payout account needs attention" whose body includes that exact reason.

**FEAT-28.SPEC-007-AC-03:** Given Talia taps "Resolve now" on the needs-action notification, when the tap registers, then she lands on FEAT-28.SPEC-002 and can proceed into FEAT-28.SPEC-006's resolution flow.

**FEAT-28.SPEC-007-AC-04:** Given Talia's account resolves from Action Required back to Active, when FEAT-28.SPEC-003 fires this notification again, then she receives the same "verification complete" content as a first-time activation.

**FEAT-28.SPEC-007-AC-05:** Given Talia has no preference control for this notification, when she looks in her notification settings, then no on/off toggle exists for it -- it is always on, in-app only.

**FEAT-28.SPEC-007-AC-06:** Given the capability reports no specific reason for an Action Required condition, when the notification renders, then the body shows "Your payout account needs a quick update" instead of a blank reason.

**FEAT-28.SPEC-007-AC-07:** Given Talia's account is already Active, when a second Active report arrives with no intervening non-Active status, then no duplicate notification fires.

**FEAT-28.SPEC-007-AC-08:** Given Talia's account is Action Required and a second report arrives with an updated reason, when the update is processed, then the existing notification's content updates to the new reason rather than a second notification appearing.

**FEAT-28.SPEC-007-AC-09:** Given Talia's Pro Account closes before this notification is delivered, when the closure completes, then the notification is not delivered.

**FEAT-28.SPEC-007-AC-10:** Given a needs-action condition flips to Active and back to Action Required in quick succession, when each transition is processed, then each fires its own distinct notification.

**FEAT-28.SPEC-007-AC-11:** Given Talia dismisses the needs-action notification without resolving the condition, when she next opens FEAT-28.SPEC-002, then the Action Required banner is still shown, since dismissing the notification does not clear the underlying status.

**FEAT-28.SPEC-007-AC-12:** Given this feature's third named communication concerns a refund that cannot be completed yet, when that condition occurs, then it is delivered through FEAT-08.SPEC-006, not this spec.

**FEAT-28.SPEC-007-AC-13:** Given Talia views this notification during any time of day, when it is delivered, then no quiet-hours delay is applied.

**FEAT-28.SPEC-007-AC-14:** Given Talia's needs-action notification remains unresolved for several days, when she checks her attention list, then it is still present, since neither notification ever expires undelivered.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 2 (verification complete, needs action) | 2 |
| Preference States | 1 (always on, no variation) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
