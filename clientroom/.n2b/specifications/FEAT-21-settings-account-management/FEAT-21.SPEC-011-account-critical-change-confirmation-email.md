---
document_type: spec
spec_type: notification
spec_id: FEAT-21.SPEC-011
spec_name: Account-Critical Change Confirmation Email
spec_slug: account-critical-change-confirmation-email
parent_feature: FEAT-21
parent_feature_name: Settings & Account Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Notification Spec: Account-Critical Change Confirmation Email

## Overview

**Name:** Account-Critical Change Confirmation Email
**ID:** FEAT-21.SPEC-011
**Type:** Notification
**Purpose:** Sends Nadia a confirmation email, to both her prior and her new sign-in email address, when her sign-in email address or login method actually changes, so a change she did not make is immediately visible to her at the address she held before the change.
**Parent Feature:** FEAT-21 -- Settings & Account Management

## Scope and Non-Goals

**In Scope:**
- The confirmation email sent once a sign-in email/login method change is confirmed by FEAT-21.SPEC-005
- Its single channel (email), content, and delivery behavior

**Non-Goals:**
- The re-verification link/code email sent while the change is still pending -- that is a distinct communication owned entirely by FEAT-21.SPEC-005's own send step (via FEAT-14.SPEC-001), not this spec; this spec fires only after the change is confirmed
- Any notification for profile, notification-preference, or business-details changes -- product-features.md's Communications field names only "account-critical changes (email address change, login method change)" for this confirmation; other Settings saves surface only an in-screen toast (FEAT-21.SPEC-001, FEAT-21.SPEC-002, FEAT-21.SPEC-004), never an email
- In-app or push delivery -- excluded per product-features.md and the dependency map, which define email as the product's sole notification channel; a security-sensitive confirmation particularly needs to reach Nadia outside the product, since it exists precisely for the case where her account access itself may be compromised

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email, sent as two separate messages: one to the prior sign-in email address and one to the new sign-in email address | Always, immediately once the change is confirmed | The prior address is the one an unauthorized change cannot redirect, so it is what makes an unauthorized change visible to Nadia; the new address confirms the change landed where intended. Nadia works from a laptop or desktop but is not necessarily inside the product at the moment a change she did not make takes effect; email is the one channel guaranteed to reach her outside the session in which the change occurred, and is the product's sole notification channel for the account-critical case this confirmation exists to cover |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Sign-in email/login method change confirmed | FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) | Fires when FEAT-21.SPEC-005's re-verification completes and the new sign-in email is committed to the Freelancer Account | Prior sign-in email, new sign-in email, time of confirmation |

## Audience and Preferences

**Recipient addresses:** exactly two, both belonging to Nadia's own account and each receiving its own copy: (1) `{prior_email}` -- the sign-in email as it stood immediately before the change committed; (2) `{new_email}` -- the sign-in email just committed. No other address (no operator, no client contact) is ever a recipient.

**Recipients:** Nadia (Freelancer) -- the sole role that can hold or change a sign-in email on the Freelancer Account. Per the Access Matrix, no other role (Owen, Priya, Dana) has any sign-in credential to change, so no other role can ever trigger or receive this notification.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| (none) | -- | Always sent | -- (this is a transactional, record-core communication; XBR-30 places it outside the optional preferences governed by FEAT-21.SPEC-002 and FEAT-21.SPEC-008, since it reports a security-sensitive change to Nadia's own credentials) |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for any notification, and even where a preference model existed, a confirmation of a change to the freelancer's own sign-in credentials is exactly the kind of account-critical event that must never be held; it is delivered the instant the change is confirmed.

## Content Definition

**Email:**
- **Subject:** Your Clientroom sign-in email was changed
- **Body:**
  Hi {freelancer_name},

  Your Clientroom sign-in email was changed from {prior_email} to {new_email} on {change_timestamp}.

  If you made this change, no action is needed.

  If you did not make this change, sign out other devices from Login & Security in Settings immediately and contact support.
- **CTA (button):** Go to Login & Security -- deep-links to FEAT-21.SPEC-003 (Login & Security) for Nadia's own account

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_name} | Freelancer Account -- `name` | Nadia Voss | Greeting renders as "Hi," -- name is never empty in practice since it is required (FEAT-21.SPEC-007), but the fallback exists for defensive completeness |
| {prior_email} | Freelancer Account -- sign-in email, value as it stood immediately before this change committed | nadia@oldstudio.com | Never empty -- a confirmed change always has a prior value, since the Freelancer Account is created with a sign-in email at sign-up (FEAT-20) |
| {new_email} | Freelancer Account -- sign-in email, the value FEAT-21.SPEC-005 just committed | nadia@newstudio.com | Never empty -- FEAT-21.SPEC-005 only fires this notification after committing a non-empty, validated new email |
| {change_timestamp} | Derived -- the moment FEAT-21.SPEC-005 committed the change, shown in Nadia's own time zone (FEAT-15) | September 27, 2026, 2:14 PM | Never empty -- always set at the moment of confirmation |

## Delivery Rules

**Batching:** None -- each confirmed change produces exactly one email per recipient address (two in total: one to `{prior_email}`, one to `{new_email}`), sent individually. A rapid sequence of changes (e.g., Nadia changes her email, then changes it again shortly after) produces one confirmation per confirmed change, never batched, since each is independently security-relevant.
**Deduplication:** At most one confirmation per confirmed change per recipient address. FEAT-21.SPEC-005 fires this notification exactly once, at the moment its commit step succeeds; a re-verification link followed twice for the same change (FEAT-21.SPEC-005's own edge case) triggers this notification only on the first, successful commit.
**Retry on failure:** Each recipient address is delivered and retried independently, so a failure at one address never delays or cancels the other. Delivery failure (including a bounce or invalid-address report) is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure at either address (or both), the failure is surfaced to Nadia as a delivery warning, consistent with XBR-30's "delivery failures are surfaced to the freelancer as warnings"; because this confirmation is account-level rather than project-level, the warning surfaces on her Account Profile screen (FEAT-21.SPEC-001) as the dismissible delivery-warning banner defined there, rather than on a project; one banner covers any number of failed addresses and it never names an email address.
**Expiry:** This notification never expires unsent in a way that discards it -- it is not time-bound to a window the way a reminder is. If delivery ultimately fails after all retries, the failure warning (above) is the surviving signal; the confirmed change itself is never reverted because its confirmation email could not be delivered.

## Edge Cases

- **Nadia changes her sign-in email twice in quick succession (the second change confirms before the first email is delivered)** -- Both confirmed changes are notified independently; each sends its own pair of emails (to that change's prior address and new address), each naming its own prior and new email accurately, so the first change's prior address and the second change's prior address (the first change's new address) each receive the relevant message.
- **The Freelancer Account is deleted (FEAT-24) shortly after a change is confirmed but before this email is delivered** -- Both emails are still delivered, to `{prior_email}` and to `{new_email}`, as a final security record of what happened to the account, since it is a transactional, evidentiary confirmation rather than a live-data view; FEAT-24's account deletion does not retract already-triggered transactional emails.
- **Nadia did not make the change (a compromised session initiated it)** -- The content's explicit "If you did not make this change" guidance is the product's mitigation; the copy sent to `{prior_email}` is the safeguard: it reaches the address the attacker did not control before the change, so Nadia sees the change even if the new address is attacker-controlled. There is no further technical safeguard this notification performs.
- **The transactional email delivery capability reports either the prior or the new email address as invalid or bouncing** -- Treated identically to any other delivery failure for that address: retried per the Retry rule above, then surfaced as the delivery warning on FEAT-21.SPEC-001; the copy to the other address is unaffected and still delivered, and the sign-in email change itself remains committed and in effect regardless of whether either confirmation email could be delivered.
- **Quiet hours or a preference change between trigger and delivery** -- Not applicable: this notification has no preference control and no quiet-hours window (see Audience and Preferences), so there is no collision to resolve.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) | Triggered by (inbound) | Fires this notification exactly once, on confirmed commit |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | Triggers (outbound) | Delivers the email and reports delivery/bounce/failure status |
| FEAT-21.SPEC-003 (Login & Security) | Navigation (outbound) | The CTA deep-links here so Nadia can immediately review sign-in security and sign out other sessions if the change was not hers |
| FEAT-21.SPEC-001 (Account Profile) | References (outbound) | Delivery-failure warnings for this notification surface here |

## Analytics and Success Signals

- **account_critical_change_confirmation_sent** (change_type: email) -- N/A -- no success-metrics.md metric is connected to Settings & Account Management; retained per product-features.md's own Communications and Signals fields so the confirmation is observable
- **account_critical_change_confirmation_delivery_failed** (retry_count) -- N/A -- no connected success-metrics.md metric; retained so a failed security confirmation is observable rather than silent, consistent with XBR-30's delivery-failure-warning requirement
- **account_critical_change_confirmation_cta_tapped** () -- N/A -- no connected success-metrics.md metric; retained to observe whether Nadia acts on a confirmation by reviewing Login & Security

## Acceptance Criteria

**FEAT-21.SPEC-011-AC-01:** Given Nadia's sign-in email change is confirmed by FEAT-21.SPEC-005, when this notification fires, then one email with the subject "Your Clientroom sign-in email was changed" is sent to her prior sign-in address and one to her new sign-in address, each naming her prior and new email accurately.

**FEAT-21.SPEC-011-AC-02:** Given Nadia receives this confirmation email, when she taps "Go to Login & Security", then she lands on FEAT-21.SPEC-003 (Login & Security).

**FEAT-21.SPEC-011-AC-03:** Given Nadia changes her sign-in email twice in quick succession, when both changes confirm, then each confirmed change sends its own separate pair of confirmation emails (prior address and new address), never merged.

**FEAT-21.SPEC-011-AC-04:** Given there is no notification preference for this email, when Nadia's account has every optional notification turned off on FEAT-21.SPEC-002, then this confirmation is still sent for any confirmed sign-in email change.

**FEAT-21.SPEC-011-AC-05:** Given delivery of this email fails once for a transient reason, when the delivery capability retries within platform parameter: `transactional-email-retry-window`, then up to platform parameter: `transactional-email-retry-count` retries occur before any failure is surfaced to Nadia.

**FEAT-21.SPEC-011-AC-06:** Given delivery of this email fails after all retries are exhausted, when the final failure is processed, then the delivery-warning banner "We couldn't deliver the confirmation email for your recent sign-in email change. If you didn't make this change, open Login & Security and sign out other devices." appears on Nadia's Account Profile screen (FEAT-21.SPEC-001) until she taps "Dismiss".

**FEAT-21.SPEC-011-AC-07:** Given Nadia's Freelancer Account is deleted shortly after a change is confirmed but before this email is delivered, when delivery proceeds, then the email is still sent to both the prior and the new sign-in email address.

**FEAT-21.SPEC-011-AC-08:** Given FEAT-21.SPEC-005's re-verification link is followed a second time for an already-confirmed change, when that second follow is processed, then no second confirmation email is sent (only the first, successful commit triggers this notification).

**FEAT-21.SPEC-011-AC-09:** Given the new sign-in email address bounces on delivery, when the bounce is reported, then it is handled as any other delivery failure for that address under the Retry on failure rule, the copy to the prior address is unaffected, and the sign-in email change remains in effect regardless.

**FEAT-21.SPEC-011-AC-10:** Given the prior sign-in email address bounces on delivery, when the bounce is reported, then it is retried under the Retry on failure rule, the copy to the new address is still delivered, and after the final failure the delivery-warning banner appears on FEAT-21.SPEC-001.

**FEAT-21.SPEC-011-AC-11:** Given a sign-in email change is confirmed from `{prior_email}` to `{new_email}`, when this notification fires, then the only recipient addresses are `{prior_email}` and `{new_email}`, and no other address receives it.

**FEAT-21.SPEC-011-AC-12:** Given Owen, Priya, or Dana have no sign-in credential on the Freelancer Account to change, then none of them can ever trigger or receive this notification.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always sent, no preference exists) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
