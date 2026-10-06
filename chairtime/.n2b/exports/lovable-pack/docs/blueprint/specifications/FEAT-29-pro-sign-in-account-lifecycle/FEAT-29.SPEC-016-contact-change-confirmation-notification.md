---
document_type: spec
spec_type: notification
spec_id: FEAT-29.SPEC-016
spec_name: Contact-Change Confirmation Notification
spec_slug: contact-change-confirmation-notification
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Notification Spec: Contact-Change Confirmation Notification

## Overview

**Name:** Contact-Change Confirmation Notification
**ID:** FEAT-29.SPEC-016
**Type:** Notification
**Purpose:** Delivers the one-time confirmation code for a sign-in email or mobile-number change to each side -- the same code-entry pattern used to deliver a sign-in code (FEAT-29.SPEC-014) -- and confirms the change to both the old and the new contact once it commits.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- The initial code-delivery variant, sent to both the old contact and the new contact when a change is started, and re-sent to one side on a "send a new code" request
- The committed-change confirmation variant, sent to both contacts once both sides confirm
- Delivery rules for both variants

**Non-Goals:**
- Deciding when a code is valid, when a side locks, or when a change commits or expires -- owned by FEAT-29.SPEC-012 (Contact-Change Confirmation Rules); this spec only delivers the messages that rules produce
- Starting the change itself, or the code-entry steps a Pro submits a code into -- owned by FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) and FEAT-29.SPEC-010 (Contact-Detail Change Processing)
- Any notice when a change is not confirmed or expires -- excluded per product-features.md's Communications field, which names only "confirmations of contact-detail changes to both old and new contacts," not a separate expiry notice; an unconfirmed or expired change simply produces no further message beyond the two variants defined here

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | The contact being confirmed (old or new) is an email address | Delivers to exactly the contact method being confirmed -- the entire point of dual confirmation is that each side proves control of its own contact method by entering the code that arrived there |
| SMS (text) | The contact being confirmed (old or new) is a mobile number | Same reasoning as email, for the mobile side of the change |

Each of the two code deliveries (old-contact, new-contact) uses whichever channel matches that specific contact's type -- an email change sends two email messages (one to old, one to new); a mobile change sends two texts. A "send a new code" request re-sends only to the requested side, on that side's channel.

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A pending contact change is started | FEAT-29.SPEC-010 (Contact-Detail Change Processing) | Fires the initial code-delivery variant to both the old and new contact | Old contact value, new contact value, which field is changing, each side's code |
| A "send a new code" request is processed for one side | FEAT-29.SPEC-010 (Contact-Detail Change Processing) | Fires the initial code-delivery variant again, to the requested side only | Which side, that side's fresh code |
| A pending contact change commits | FEAT-29.SPEC-010 (Contact-Detail Change Processing) | Fires the committed-change confirmation variant to both contacts, once both sides have confirmed | Old contact value, new contact value, which field changed, commit time |

## Audience and Preferences

**Recipients:** The Pro (Talia) -- specifically whoever holds the old contact and whoever holds the new contact for this change. In the ordinary case both are Talia herself; the notification is addressed to each contact independently since confirmation depends on control of the contact, not identity per se.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| None -- this notification cannot be turned off | -- | Always delivered | -- |

Confirming a sign-in contact change is a security-relevant action with no discretionary opt-out, consistent with the other account-protection notifications in this feature.

**Quiet Hours:** N/A -- both variants are delivered immediately regardless of time of day; the initial code requires timely entry within FEAT-29.SPEC-012's confirmation window, and holding it for quiet hours would erode that window without the Pro's knowledge.

## Content Definition

**Initial code delivery -- Email (sent to whichever side is an email address):**
- **Subject:** Your Chairtime code to confirm this change
- **Body:**
  You're changing the {field_label} on your Chairtime sign-in from {old_value_masked} to {new_value_masked}.

  Enter this code in the Chairtime app to confirm: {confirmation_code}

  If you didn't request this, ignore this message and no change will be made.
- **CTA:** None -- the code is entered on FEAT-29.SPEC-003 (Account & Sign-In Settings Screen), not through a link in this message, identically to the sign-in code pattern (FEAT-29.SPEC-014)

**Initial code delivery -- SMS (sent to whichever side is a mobile number):**
- **Body:** Chairtime: your code to confirm changing your sign-in {field_label} is {confirmation_code}. Enter it in the app. Didn't request this? Ignore this text.
- **CTA:** None -- the code is entered on FEAT-29.SPEC-003

**Committed-change confirmation -- Email:**
- **Subject:** Your Chairtime sign-in {field_label} has changed
- **Body:**
  Your Chairtime sign-in {field_label} was changed from {old_value_masked} to {new_value_masked} on {commit_date}.

  If you didn't make this change, open your account settings and sign out everywhere right away.
- **CTA:** Manage sign-in -- deep-links to FEAT-29.SPEC-003 (Account & Sign-In Settings Screen)

**Committed-change confirmation -- SMS:**
- **Body:** Chairtime: your sign-in {field_label} changed to {new_value_masked} on {commit_date}. Didn't do this? Sign out everywhere in Account & Sign-In settings.
- **CTA:** None on SMS itself; the Pro follows up in-app via FEAT-29.SPEC-003.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|------------------------|
| {field_label} | Derived -- "email address" or "mobile number" depending on which field is changing | mobile number | Never empty -- always one of the two known field labels |
| {confirmation_code} | Pending contact-change record.old_contact_code or .new_contact_code, whichever side this delivery is for | 482913 | Never empty -- generated whenever a code is sent or resent for that side (FEAT-29.SPEC-012) |
| {old_value_masked} | Pending contact-change record.old_value, masked | t***a@gmail.com | Never empty -- a pending change always has an old_value |
| {new_value_masked} | Pending contact-change record.new_value, masked | (***) ***-9821 | Never empty -- a pending change always has a new_value |
| {commit_date} | Derived -- the date the change committed, in the Pro's timezone | September 27, 2026 | Never empty -- present only on the committed variant, which fires only once commit occurs |

## Delivery Rules

**Batching:** None -- each pending change produces exactly one initial code delivery per side (plus one more per side for each "send a new code" request) and, if it commits, exactly one committed confirmation per side; these are never batched with any other notification.
**Deduplication:** At most one active code per side at a time -- a "send a new code" request invalidates the prior code for that side and produces exactly one fresh delivery to that side, never a resend of the invalidated code. At most one committed confirmation per pending change per side. Starting a fresh pending change for the same field (superseding an earlier one, per FEAT-29.SPEC-010's Business Rules) produces its own fresh codes and deliveries -- the earlier pending change's codes are not resent or reused.
**Retry on failure:** Both email and SMS are retried once on failure, independently per side and per variant. A failed code delivery to one side does not block delivery to the other side, since each side's confirmation is independent (FEAT-29.SPEC-012).
**Expiry:** Each delivered code expires per platform parameter: `contact-change-code-expiry-minutes` (FEAT-29.SPEC-012). A code entered after its own expiry is evaluated as a wrong/expired entry producing the generic failure message on FEAT-29.SPEC-003, not a silent no-op; the Pro can then request a new code for that side.

## Edge Cases

- **The contact receiving the code is not actually the Pro (e.g., the new mobile number was mistyped and belongs to someone else)** -- That recipient sees a code but has no reason to enter it into an app they do not use; if they ignore it, the change never commits and expires per FEAT-29.SPEC-012, leaving the Pro's contact details unchanged. If they mistakenly relay or enter the code somewhere, the change still requires the Pro's own old-contact code before committing, limiting the impact of a single mistaken entry.
- **The underlying pending change is discarded (superseded by a fresh change, or expired) before the other side's code is delivered or entered** -- A code delivered or entered against a pending change that no longer exists is evaluated as a wrong/expired entry, per FEAT-29.SPEC-012's handling of a code against an already-resolved change; no separate error state is needed beyond the generic failure message.
- **Preferences changing between trigger and delivery** -- N/A -- this notification has no preference control to change (see Audience and Preferences).
- **Quiet hours colliding with expiry** -- N/A -- this notification has no quiet-hours hold (see Delivery Rules), so no such collision can occur.
- **Both the old and new contact happen to be the same channel type (e.g., changing between two email-like identifiers is not possible in this product, but a change from email to email is impossible since email and mobile are distinct fields) -- N/A** -- each contact-change record is scoped to exactly one field (sign_in_email or sign_in_mobile), so old and new are always the same type (both email addresses, for an email change; both mobile numbers, for a mobile change); this is a closed, well-defined case, not an edge case requiring special handling.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-010 (Contact-Detail Change Processing) | Triggered by (inbound) | The start, a "send a new code" request, and the commit of a pending change each fire this notification's respective variant |
| FEAT-29.SPEC-012 (Contact-Change Confirmation Rules) | References (inbound) | Governs each code's expiry and whether a prompt still matters |
| FEAT-29.SPEC-014 (Sign-In Code Notification) | References (inbound) | Shares the same code-entry-in-app pattern rather than a link or reply action |
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Navigation (outbound) | The committed-confirmation email's CTA deep-links here; codes themselves are entered here |

## Analytics and Success Signals

- **contact_change_code_delivered** (side: old / new, channel: email / sms, field: email / mobile) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained so code delivery remains observable
- **contact_change_confirmed_notification_delivered** (side: old / new, channel: email / sms, field: email / mobile) -- N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list

## Acceptance Criteria

**FEAT-29.SPEC-016-AC-01:** Given Talia starts a mobile-number change, when the pending change is created, then both her old and new mobile numbers receive a text with a one-time code to confirm the change.

**FEAT-29.SPEC-016-AC-02:** Given Talia starts an email change, when the pending change is created, then both her old and new email addresses receive an email with a one-time code to confirm the change.

**FEAT-29.SPEC-016-AC-03:** Given both sides of Talia's pending change enter their correct codes, when the change commits, then both her old and new contact receive a committed-change confirmation.

**FEAT-29.SPEC-016-AC-04:** Given Talia's committed-change confirmation email arrives, when she taps "Manage sign-in", then she is navigated to FEAT-29.SPEC-003.

**FEAT-29.SPEC-016-AC-05:** Given the old-contact code delivery fails to send, when the retry also fails, then the new-contact code delivery is still attempted independently.

**FEAT-29.SPEC-016-AC-06:** Given a pending change is discarded (superseded or expired) before the new-contact code is delivered, when that code eventually arrives and is entered, then the entry is evaluated as a wrong/expired code, since the pending change no longer exists.

**FEAT-29.SPEC-016-AC-07:** Given a fresh pending change supersedes an earlier one for the same field, when the fresh change starts, then it produces its own new codes rather than resending the earlier ones.

**FEAT-29.SPEC-016-AC-08:** Given this notification fires at any hour, when either variant is triggered, then it is delivered immediately with no quiet-hours hold.

**FEAT-29.SPEC-016-AC-09:** Given Talia's contact change is never confirmed by either side and the overall confirmation window expires, when expiry occurs (FEAT-29.SPEC-012), then no further notification is sent beyond the codes and any committed confirmation already delivered.

**FEAT-29.SPEC-016-AC-10:** Given a new mobile number was mistyped and belongs to someone unrelated, when that person receives the code and does not enter it anywhere, then the change never commits and the Pro's contact details remain unchanged.

**FEAT-29.SPEC-016-AC-11:** Given Talia taps "Send a new code" on her old-email side, when the fresh code is generated, then only her old email receives the new code, and the prior code for that side is invalidated.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 2 (email, SMS) | 2 |
| Trigger Paths | 3 (start, resend, commit) | 3 |
| Preference States | 1 (always delivered, no preference) | 1 |
| Delivery Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
