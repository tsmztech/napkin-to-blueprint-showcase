---
document_type: spec
spec_type: notification
spec_id: FEAT-29.SPEC-017
spec_name: Account Closure & Deletion Notifications
spec_slug: account-closure-deletion-notifications
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Notification Spec: Account Closure & Deletion Notifications

## Overview

**Name:** Account Closure & Deletion Notifications
**ID:** FEAT-29.SPEC-017
**Type:** Notification
**Purpose:** Sends the account-closure confirmation when closure is requested and the final notice when data is permanently deleted.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- The closure-confirmation variant, sent when Talia's account transitions to Closing
- The final deletion-notice variant, sent when her account transitions to Closed and data is permanently deleted

**Non-Goals:**
- A notice when the account is reopened -- excluded per product-features.md's Communications field, which names only "an account-closure confirmation and a final notice when data is permanently deleted"; reopening's confirmation is an in-product message on FEAT-29.SPEC-005 ("Your account is reopened"), not a standalone Notification spec
- Deciding the cooling-off timing itself -- owned by FEAT-29.SPEC-013 (Account Closure & Retention Rules); this spec only delivers messages at the moments FEAT-29.SPEC-008 triggers
- Notifying the Pro's clients about the closure -- out of scope for this feature; any client-facing consequence (e.g., the booking page becoming unavailable) is handled by FEAT-05's own messaging, not this spec

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, to the Pro's sign-in email as recorded at the moment each message fires | A closure or deletion confirmation is a significant, non-urgent-to-act-on record the Pro may want to keep; email is the durable, reviewable channel for it |
| SMS (text) | Always, to the Pro's sign-in mobile number as recorded at the moment each message fires | Matches the Pro's behavioral pattern of checking her phone, ensuring she is aware even if she does not check email promptly |

Both channels are used together for both variants, consistent with this being a significant, infrequent, irreversible-consequence event.

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Account closure is confirmed | FEAT-29.SPEC-008 (Account Closure Orchestration) | Fires the closure-confirmation variant once the closure-start sequence commits (subscription cancelled, booking page down, status set to Closing) | Closure request date, cooling-off period length |
| Permanent deletion executes | FEAT-29.SPEC-008 (Account Closure Orchestration) | Fires the final deletion-notice variant once the cooling-off period expires unreversed and deletion completes | Deletion date |

## Audience and Preferences

**Recipients:** The Pro (Talia) -- delivered to her sign-in email and mobile number as recorded at the moment each message fires (the closure confirmation uses the contacts on file when closure starts; the deletion notice is sent immediately before or as the final deletion step removes those same contacts, so it must be dispatched using the still-current values at that instant).

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| None -- this notification cannot be turned off | -- | Always delivered | -- |

Both variants confirm significant, account-defining state changes with no discretionary opt-out; a Pro closing or permanently losing their account data must be told regardless of any notification preference.

**Quiet Hours:** N/A -- both variants are delivered as soon as their triggering step completes, regardless of time of day; a closure or deletion confirmation is not time-sensitive in the way a sign-in code is, but holding it artificially would only delay a Pro's awareness of a significant, irreversible-approaching change with no corresponding benefit.

## Content Definition

**Closure confirmation -- Email:**
- **Subject:** Your Chairtime account is closing
- **Body:**
  We've started closing your Chairtime account. Your subscription is cancelled and your booking page is down.

  You have {cooling_off_days} days to change your mind. Sign back in any time before then to reopen your account with everything intact. After that, your data is permanently deleted, except the financial records we're required by law to keep.
- **CTA:** Manage account -- deep-links to FEAT-29.SPEC-005 (Account Closure & Reopening Screen)

**Closure confirmation -- SMS:**
- **Body:** Chairtime: your account is closing. You have {cooling_off_days} days to sign back in and reopen it before your data is permanently deleted.
- **CTA:** None on SMS itself; the Pro follows up in-app via FEAT-29.SPEC-005.

**Deletion notice -- Email:**
- **Subject:** Your Chairtime account data has been deleted
- **Body:**
  Your cooling-off period has ended, and your Chairtime account data has now been permanently deleted, as you requested when you closed your account.

  We've kept only the financial records the law requires us to retain, in de-identified form. This account can no longer be reopened.
- **CTA:** None -- this is a final, terminal notice with no further in-product action available for this account.

**Deletion notice -- SMS:**
- **Body:** Chairtime: your account data has been permanently deleted after your cooling-off period ended. This account can no longer be reopened.
- **CTA:** None.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|------------------------|
| {cooling_off_days} | Derived -- platform parameter: `account-closure-cooling-off-days` | 30 | Never empty -- always present, one value for every Pro |

## Delivery Rules

**Batching:** None -- each variant is a single, standalone message tied to a specific state transition; neither is ever batched with any other notification.
**Deduplication:** At most one closure confirmation per closure event, and at most one deletion notice per deletion event. A Pro who closes, reopens, and later closes again generates a fresh closure confirmation for the new closure event -- these are independent lifecycle events, not duplicates of the same message.
**Retry on failure:** Both email and SMS are retried once on failure for the closure confirmation, independently per channel. For the deletion notice specifically, delivery is attempted before the sign-in contacts are removed as part of the same deletion sequence (FEAT-29.SPEC-008's Processing Logic), so a failure here has no further retry once the contact details themselves are gone -- the notice is best-effort at that final moment, and its delivery outcome is recorded (see Analytics) so a failure is never silent even though it cannot be retried against a now-deleted contact.
**Expiry:** Neither variant expires undelivered -- both are significant, one-time lifecycle confirmations that remain worth delivering whenever they can be sent, with no cutoff.

## Edge Cases

- **The Pro reopens the account before the closure confirmation is even delivered** -- The confirmation, if it still arrives, describes an accurate historical fact (that closure was started at that moment); it causes no confusion, since the Pro's current in-product state (Active, per the reopening) is shown correctly on FEAT-29.SPEC-003 regardless of a delayed confirmation message.
- **The deletion notice's delivery must complete using contact details that are about to be deleted in the same operation** -- Delivery is sequenced to fire before (or as part of) the same deletion step that removes sign_in_email and sign_in_mobile, ensuring the notice is dispatched to valid contacts; per FEAT-29.SPEC-008's Processing Logic, this notification is explicitly one of the deletion sequence's own steps, not an afterthought following it.
- **Quiet hours colliding with expiry** -- N/A -- neither variant has a quiet-hours hold or an expiry (see Delivery Rules), so no such collision can occur.
- **The Pro closes, reopens, and closes again within a short period** -- Each closure produces its own closure confirmation; there is no deduplication across genuinely separate closure events, since each represents the Pro's own distinct decision.
- **Both delivery channels fail for the deletion notice** -- No further fallback exists (the contacts are being deleted in the same step); the failure is recorded for operational visibility, but there is no further Pro-facing surface to retry against once the account is Closed and its contacts are gone.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-008 (Account Closure Orchestration) | Triggered by (inbound) | Fires both variants at their respective moments |
| FEAT-29.SPEC-005 (Account Closure & Reopening Screen) | Navigation (outbound) | The closure-confirmation email's CTA deep-links here |
| FEAT-29.SPEC-013 (Account Closure & Retention Rules) | References (inbound) | Defines the cooling-off period length shown in the closure confirmation |

## Analytics and Success Signals

- **account_closure_confirmation_delivered** (channel: email / sms) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list
- **account_deletion_notice_delivered** (channel: email / sms) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list, and especially so given this notice cannot be retried once the contact details are gone

## Acceptance Criteria

**FEAT-29.SPEC-017-AC-01:** Given Talia confirms account closure with no upcoming bookings remaining, when FEAT-29.SPEC-008's closure-start sequence commits, then she receives the closure confirmation on both email and text, stating platform parameter: `account-closure-cooling-off-days` days remain to reopen.

**FEAT-29.SPEC-017-AC-02:** Given Talia's closure confirmation email arrives, when she taps "Manage account", then she is navigated to FEAT-29.SPEC-005.

**FEAT-29.SPEC-017-AC-03:** Given Talia's account reaches the end of its cooling-off period unreversed, when permanent deletion executes, then she receives the deletion notice on both email and text, stating the account can no longer be reopened.

**FEAT-29.SPEC-017-AC-04:** Given Talia's deletion notice must reach her sign-in contacts before those same contacts are deleted, when the deletion sequence runs, then delivery is attempted as part of that same sequence, before the contact fields are removed.

**FEAT-29.SPEC-017-AC-05:** Given Talia reopens her account before the closure confirmation is delivered, when the delayed confirmation eventually arrives, then it causes no confusion, since her in-product status correctly shows Active.

**FEAT-29.SPEC-017-AC-06:** Given Talia closes, reopens, and closes her account again, when each closure commits, then each produces its own independent closure confirmation.

**FEAT-29.SPEC-017-AC-07:** Given either notification fires at any hour, when it is triggered, then it is delivered immediately with no quiet-hours hold.

**FEAT-29.SPEC-017-AC-08:** Given Talia's closure-confirmation email delivery fails, when the retry also fails, then the SMS delivery is still attempted independently.

**FEAT-29.SPEC-017-AC-09:** Given both delivery channels fail for Talia's deletion notice, when the failure is final, then no further retry occurs against her now-deleted contact details, and the failure is recorded for operational visibility.

**FEAT-29.SPEC-017-AC-10:** Given Talia's deletion notice is delivered, when she reads it, then it states that only legally required de-identified financial records are retained.

**FEAT-29.SPEC-017-AC-11:** Given Talia's account is permanently deleted, when she reads the deletion notice, then it states explicitly that the account can no longer be reopened.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (email, SMS) | 2 |
| Trigger Paths | 2 (closure, deletion) | 2 |
| Preference States | 1 (always delivered, no preference) | 1 |
| Delivery Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
