---
document_type: spec
spec_type: notification
spec_id: FEAT-04.SPEC-007
spec_name: Calendar Reconnection Alert
spec_slug: calendar-reconnection-alert
parent_feature: FEAT-04
parent_feature_name: Two-Way Calendar Sync
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 9
---

# Notification Spec: Calendar Reconnection Alert

## Overview

**Name:** Calendar Reconnection Alert
**ID:** FEAT-04.SPEC-007
**Type:** Notification
**Purpose:** Tells the Pro, via a dashboard banner, that a connected calendar needs reconnecting, so a lapsed connection is never a silent gap.
**Parent Feature:** FEAT-04 -- Two-Way Calendar Sync

## Scope and Non-Goals

**In Scope:**
- The dashboard banner delivered when a connection transitions to Needs Reconnection
- The banner's behavior while the connection remains unresolved (persistence, not repeated re-delivery)
- Dismissal and re-appearance behavior
- Clearing the alert once the connection is reconnected

**Non-Goals:**
- Deciding when a connection is considered lapsed -- owned by FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation); this notification only delivers what that automation decides
- The reconnection flow itself -- owned by FEAT-04.SPEC-002 (Calendar Connection Status & Management); this notification's CTA only deep-links there
- Delivery by text or email -- excluded per product-features.md's Communications field, which states explicitly this is "a dashboard alert (not a text/email)"; no other channel exists for this notification
- General Pro notification preferences for other event types (new bookings, refund failures, disputes) -- owned by FEAT-08 (Automated Booking Messaging); this spec covers only the calendar-reconnection alert

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app | Always, for as long as the connection remains in Needs Reconnection status | Talia works inside the product in short bursts between clients (user-persona.md, Behavioral Context); a dashboard banner is where she is already looking, and product-features.md's Communications field states this is deliberately not a text/email interruption -- a calendar hiccup does not warrant pulling her out of a client appointment |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Connection status becomes Needs Reconnection | FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) | Fires whenever a connection transitions to Needs Reconnection | Calendar Connection reference, calendar_kind |

## Audience and Preferences

**Recipients:** The Pro (Talia) only -- the sole role that connects and manages calendars (Access Matrix: Service & Availability Setup = Full). The Client never sees this alert (Access Matrix: None), and Platform Operator (Support) has View-only access to connection health but is not a recipient of this Pro-facing dashboard alert.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| N/A -- no preference control exists for this alert | -- | Always on | -- |

This alert has no opt-out: product-features.md's Validation & Limits states a sync failure "must surface as a visible dashboard banner rather than a silent gap," and ASMP-26's correctness bar ("never silently double-book") makes this a mandatory, non-optional alert rather than a preference-gated notification.

**Quiet Hours:** N/A -- quiet hours govern time-sensitive, interruption-style notifications (product-features.md, Automated Booking Messaging); a persistent in-app dashboard banner is not a point-in-time interruption and carries no quiet-hours concept of its own.

## Content Definition

**In-app:**
- **Title:** Reconnect your {calendar_kind} calendar
- **Body:** Chairtime lost access to your calendar, so it's not blocking or updating your personal calendar right now. Reconnect to keep everything in sync.
- **CTA:** Reconnect -- deep-links to FEAT-04.SPEC-002 (Calendar Connection Status & Management) for the specific connection needing reconnection, with focus landing on that connection's card

**Batched variant (both connections need reconnecting at once):**
- **In-app title:** Reconnect your calendars
- **In-app body:** Chairtime lost access to your Google and Apple calendars, so neither is blocking or updating your personal calendars right now. Reconnect to keep everything in sync.
- **CTA:** Reconnect -- deep-links to FEAT-04.SPEC-002 (Calendar Connection Status & Management), showing both connections needing reconnection

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {calendar_kind} | Calendar Connection -- calendar_kind | Google | Never empty -- calendar_kind is required on every Calendar Connection record and is always known at trigger time |

## Delivery Rules

**Batching:** If both of a Pro's connections (Google and Apple) are in Needs Reconnection status at the same time, the two individual alerts collapse into the single batched variant above rather than showing two separate banners. If only one connection lapses while the other remains healthy, the single-connection variant is shown.
**Deduplication:** At most one active banner per connection at a time. A connection already showing its banner does not produce a second, duplicate banner if FEAT-04.SPEC-006's scheduled health check re-confirms the same Needs Reconnection status on a later run -- the existing banner simply continues to be shown.
**Retry on failure:** N/A -- this is a persistent in-app banner state, not a point-in-time send that can fail to deliver; as long as the connection remains in Needs Reconnection status, the banner is present whenever the Pro views the dashboard.
**Expiry:** The banner never expires on its own -- it persists for as long as the connection remains in Needs Reconnection status, since a stale reconnect prompt that quietly disappeared would recreate exactly the silent-gap risk this alert exists to prevent. It clears immediately once FEAT-04.SPEC-006 sets the connection back to Connected.

## Edge Cases

- **Pro dismisses the banner without reconnecting** -- The banner reappears the next time the Pro opens the dashboard, since dismissal is not the same as resolution; only a successful reconnection (FEAT-04.SPEC-006 setting status back to Connected) clears it.
- **Pro disconnects the lapsed connection instead of reconnecting it** -- The banner for that connection clears immediately, since the underlying record (and its Needs Reconnection status) no longer exists; this is treated the same as a resolved state, not left showing a reconnect prompt for a connection that is gone.
- **Both connections lapse, then one is reconnected while the other remains lapsed** -- The batched variant is replaced by the single-connection variant for the still-lapsed connection; the reconnected one's contribution to the banner clears without affecting the other.
- **Connection reconnects and then immediately lapses again (flapping)** -- Each transition is treated independently: the banner clears on the Connected transition and reappears fresh on the next Needs Reconnection transition; no cooldown suppresses a second, genuine alert.
- **Connection reference no longer exists when the dashboard renders the banner (disconnected between trigger and render)** -- The banner is not rendered for a connection that no longer exists; the dashboard reflects the Pro's current connection list at render time, not a stale trigger snapshot.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-006 (Sync Health Monitor & Reconciliation) | Triggered by (inbound) | The Needs Reconnection transition fires this notification; the Connected transition clears it |
| FEAT-04.SPEC-002 (Calendar Connection Status & Management) | Navigation (outbound) | The Reconnect CTA deep-links here, focused on the affected connection |
| FEAT-12 (Pro Daily Schedule Dashboard) | References (inbound) | This banner surfaces on the dashboard's attention list, per the dependency map's Navigation Connections |

## Analytics and Success Signals

- **reconnect_alert_shown** (calendar_kind or "both", batched: yes / no) -- supports success-metrics.md: "Calendar Sync Reliability"
- **reconnect_alert_cta_tapped** (calendar_kind or "both") -- supports success-metrics.md: "Calendar Sync Reliability"
- **reconnect_alert_cleared** (calendar_kind, resolution: reconnected / disconnected) -- supports success-metrics.md: "Calendar Sync Reliability"

## Acceptance Criteria

**FEAT-04.SPEC-007-AC-01:** Given Talia's Google connection transitions to Needs Reconnection, when she next opens her dashboard, then she sees the banner "Reconnect your Google calendar" with its body and a Reconnect CTA.

**FEAT-04.SPEC-007-AC-02:** Given Talia taps the Reconnect CTA on the banner, when the tap registers, then she lands on FEAT-04.SPEC-002 with focus on the Google connection's card.

**FEAT-04.SPEC-007-AC-03:** Given both of Talia's connections are Needs Reconnection at the same time, when she opens her dashboard, then she sees the single batched banner "Reconnect your calendars," not two separate banners.

**FEAT-04.SPEC-007-AC-04:** Given Talia's connection is reconnected successfully, when FEAT-04.SPEC-006 sets status back to Connected, then the banner clears from her dashboard.

**FEAT-04.SPEC-007-AC-05:** Given Talia dismisses the banner without reconnecting, when she next opens her dashboard, then the banner reappears, since dismissal does not resolve the lapsed connection.

**FEAT-04.SPEC-007-AC-06:** Given Talia disconnects a lapsed connection instead of reconnecting it, when the disconnect completes, then the banner for that connection clears immediately.

**FEAT-04.SPEC-007-AC-07:** Given both of Talia's connections are lapsed and she reconnects only one, when the dashboard re-renders, then the batched banner is replaced by the single-connection banner for the one still lapsed.

**FEAT-04.SPEC-007-AC-08:** Given Talia's connection reconnects and then lapses again shortly after, when the second lapse occurs, then a fresh banner is shown with no suppression from the first alert having just cleared.

**FEAT-04.SPEC-007-AC-09:** Given a connection was disconnected between the trigger firing and the dashboard rendering, when Talia opens the dashboard, then no banner is shown for that no-longer-existing connection.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (no preference -- always on) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry N/A, expiry) | 4 |
| Edge Cases | 5 | 5 |
