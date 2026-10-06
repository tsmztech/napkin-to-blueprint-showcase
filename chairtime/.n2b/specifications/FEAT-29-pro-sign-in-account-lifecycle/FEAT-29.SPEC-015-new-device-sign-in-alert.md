---
document_type: spec
spec_type: notification
spec_id: FEAT-29.SPEC-015
spec_name: New-Device Sign-In Alert
spec_slug: new-device-sign-in-alert
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Notification Spec: New-Device Sign-In Alert

## Overview

**Name:** New-Device Sign-In Alert
**ID:** FEAT-29.SPEC-015
**Type:** Notification
**Purpose:** Alerts Talia on her existing contact methods when her account is signed in on a device not seen before, so an unrecognized sign-in is never silent.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Alerting the Pro on both her existing sign-in contacts (email and mobile) when a new device successfully signs in
- Letting the alert direct the Pro to her devices list if the sign-in was not her own

**Non-Goals:**
- Detecting whether a device is new -- owned by FEAT-29.SPEC-006 (Session & Device Management); this spec only delivers the alert once that determination is made
- Blocking or reversing the sign-in -- this alert is informational; the sign-in has already succeeded by the time this notification fires (product-features.md's Access field: an account-protection measure that surfaces awareness, not a blocking gate)
- Alerting on the very first device created during onboarding -- excluded per FEAT-29.SPEC-006's Business Rules: there is no prior device or established account to alert from at that moment

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to the Pro's sign-in email | The email address is a channel the Pro is likely to still control even if the new sign-in used a different, possibly unauthorized device, making it a reliable independent alert surface |
| SMS (text) | Always, to the Pro's sign-in mobile number | Text reaches the Pro on her phone quickly, matching her behavioral pattern of checking Chairtime between clients (Behavioral Context, user-persona.md) |

Both channels are used together (not one-or-the-other) because this is a security alert: reaching the Pro reliably matters more than minimizing message volume for this one, infrequent event.

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A new signed-in device is created | FEAT-29.SPEC-006 (Session & Device Management) | Fires whenever a successful code verification creates a signed_in_devices entry that did not previously exist, except for the first device created during onboarding | Device description, approximate location/type if available, sign-in time |

## Audience and Preferences

**Recipients:** The Pro (Talia) -- delivered to both her current sign_in_email and sign_in_mobile, since either could be the contact she notices the alert on first. Per the Access Matrix, this is a Pro-only notification.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| None -- this alert cannot be turned off | -- | Always delivered | -- |

A new-device alert is a core account-protection measure (ASMP-30) and is never optional, consistent with there being no discretionary security notification the Pro can silence.

**Quiet Hours:** N/A -- a new-device sign-in alert is delivered immediately regardless of time of day, since a delayed security alert defeats its purpose; a Pro who is the victim of unauthorized access needs to know as soon as possible.

## Content Definition

**Email:**
- **Subject:** New sign-in to your Chairtime account
- **Body:**
  Your Chairtime account was just signed in on a device we haven't seen before ({device_description}) at {sign_in_time}.

  If this was you, you can ignore this message. If it wasn't, open your account settings and sign out everywhere right away.
- **CTA:** Manage sign-in -- deep-links to FEAT-29.SPEC-003 (Account & Sign-In Settings Screen), the devices section

**SMS (text):**
- **Body:** Chairtime: new sign-in on a device we haven't seen before ({device_description}) at {sign_in_time}. If this wasn't you, sign out everywhere in Account & Sign-In settings.
- **CTA:** None on SMS itself (text has no interactive deep-link surface in this product); the Pro follows up in-app via FEAT-29.SPEC-003.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|------------------------|
| {device_description} | Pro Account.signed_in_devices -- the new entry's device description | iPhone, San Francisco area | "a device" -- used when no descriptive detail is available, so the sentence still reads naturally: "signed in on a device we haven't seen before (a device)" is avoided in favor of omitting the parenthetical entirely when the fallback applies |
| {sign_in_time} | Pro Account.signed_in_devices -- the new entry's creation timestamp, in the Pro's timezone | 3:42 PM | Never empty -- the entry always has a creation timestamp |

## Delivery Rules

**Batching:** None -- each new-device sign-in is its own distinct security event and is delivered individually the moment it is detected; batching a security alert would delay the Pro's awareness of a specific event.
**Deduplication:** At most one alert per new signed_in_devices entry -- the same device signing in again later (an existing device refreshing its last-active timestamp) does not re-trigger this alert, since it is by definition no longer new.
**Retry on failure:** Both email and SMS are retried once on failure. Unlike the Sign-In Code Notification, a failure on one channel does not block the other -- both channels are attempted independently regardless of whether the other succeeds, since this alert's reliability matters more for a security notification than avoiding channel overlap.
**Expiry:** This notification never expires undelivered -- a delayed security alert is still worth delivering whenever it can be, unlike a time-sensitive sign-in code; there is no cutoff after which delivery is abandoned.

## Edge Cases

- **The new device belongs to the Pro herself (e.g., a new phone)** -- The alert still fires; there is no way to distinguish a Pro's own new device from an unauthorized one at the moment of sign-in, so the alert is deliberately unconditional, and the Pro simply disregards it when it was her own action.
- **The new device is signed out (individually or via "sign out everywhere") before this notification is delivered** -- The alert is still delivered; it describes a sign-in event that already occurred, not the device's current state, so its informational value stands regardless of what happens to the device afterward.
- **Quiet hours collide with expiry** -- N/A -- this notification has no quiet-hours hold and no expiry (see Delivery Rules), so no such collision can occur.
- **Two new devices sign in within moments of each other** -- Each produces its own independent alert; they are not batched together, since each represents a distinct event the Pro needs to evaluate on its own.
- **Both delivery channels fail (email and SMS both fail after retry)** -- No further fallback channel exists for this notification; the failure is recorded (see Analytics) so it remains observable for operational monitoring, but the Pro has no in-product indicator of a missed new-device alert beyond her devices list on FEAT-29.SPEC-003, which always shows the device regardless of alert delivery success.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-006 (Session & Device Management) | Triggered by (inbound) | New-device detection fires this notification |
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Navigation (outbound) | The CTA deep-links to the devices section |

## Analytics and Success Signals

- **new_device_alert_delivered** (channel: email / sms) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so this security alert's delivery reliability remains observable
- **new_device_alert_delivery_failed** (channel: email / sms) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so a missed security alert is never silent, given both channels have no further fallback

## Acceptance Criteria

**FEAT-29.SPEC-015-AC-01:** Given Talia signs in successfully from a device not previously seen, when the sign-in succeeds, then she receives both an email and a text alert describing the new sign-in.

**FEAT-29.SPEC-015-AC-02:** Given Talia signs in from a device already in her signed_in_devices list, when the sign-in succeeds, then no new-device alert is sent.

**FEAT-29.SPEC-015-AC-03:** Given a brand-new Pro completes her first-ever sign-in during onboarding, when her first device is created, then no new-device alert is sent for that first device.

**FEAT-29.SPEC-015-AC-04:** Given Talia receives the email alert, when she taps "Manage sign-in", then she is navigated to FEAT-29.SPEC-003, devices section.

**FEAT-29.SPEC-015-AC-05:** Given the new device that triggered this alert is signed out before the alert is delivered, when delivery proceeds, then the alert is still sent describing the sign-in event that occurred.

**FEAT-29.SPEC-015-AC-06:** Given this notification fires at any hour, when it is triggered, then it is delivered immediately with no quiet-hours hold and no expiry.

**FEAT-29.SPEC-015-AC-07:** Given Talia's email delivery fails, when the retry also fails, then the SMS delivery is still attempted independently regardless of the email outcome.

**FEAT-29.SPEC-015-AC-08:** Given two new devices sign in within moments of each other, when both are detected, then two separate alerts are sent, not one batched alert.

**FEAT-29.SPEC-015-AC-09:** Given both email and SMS delivery fail after retry, when the failure is final, then a new_device_alert_delivery_failed event is recorded, and Talia's devices list on FEAT-29.SPEC-003 still shows the device regardless.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (email, SMS) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always delivered, no preference) | 1 |
| Delivery Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
