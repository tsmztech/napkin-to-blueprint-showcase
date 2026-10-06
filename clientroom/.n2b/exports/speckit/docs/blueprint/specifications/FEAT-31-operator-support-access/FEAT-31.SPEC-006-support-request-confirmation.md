---
document_type: spec
spec_type: notification
spec_id: FEAT-31.SPEC-006
spec_name: Support Request Confirmation
spec_slug: support-request-confirmation
parent_feature: FEAT-31
parent_feature_name: Operator Support Access
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Notification Spec: Support Request Confirmation

## Overview

**Name:** Support Request Confirmation
**ID:** FEAT-31.SPEC-006
**Type:** Notification
**Purpose:** Sends Nadia a confirmation email the moment her support request is received, so she knows it reached Dana even after she leaves the Contact Support screen.
**Parent Feature:** FEAT-31 -- Operator Support Access

## Scope and Non-Goals

**In Scope:**
- The emailed confirmation sent the instant a support request is created
- Its content, delivery rules, and edge cases

**Non-Goals:**
- The on-screen confirmation shown immediately after Nadia sends her request -- that inline feedback is owned by FEAT-31.SPEC-001 (Contact Support Screen); this spec covers only the separate emailed confirmation product-features.md's Communications field names.
- Dana's diagnostic reply -- product-features.md's Primary Flows & Alternates states Dana "replies by email," sent by her personally, outside the product; this is not a system-templated notification this feature owns.
- Any notice about the session opening -- a distinct communication with its own trigger and content, owned by FEAT-31.SPEC-007 (Support Session Opened Notice).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, the instant a support request is created | Nadia may have already left the Contact Support screen; email reaches her wherever she is, and the Access Matrix and Communications field both name only an emailed confirmation for this moment -- the in-app confirmation is already handled inline by the triggering screen (FEAT-31.SPEC-001) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A support request is created | FEAT-31.SPEC-001 (Contact Support Screen) | Fires the instant the Support Access Session record is created, on every successful send | Nadia's account name and email, the submitted `request_text` |

## Audience and Preferences

**Recipients:** Nadia only -- the freelancer who submitted the request. No other role receives this email: Owen and Priya have "None" for Support Access in the Access Matrix, and Dana receives the request in her own queue (FEAT-31.SPEC-002), not by email.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Support request confirmation | Always on (transactional) | On | N/A -- this is a transactional record email; per XBR-30, transactional emails core to the record always send and cannot be disabled |

**Quiet Hours:** N/A -- the product definition (FEAT-14, Notifications (Email)) establishes no quiet-hours mechanism for any notification; this email sends immediately regardless of time of day, consistent with its purpose as an immediate receipt.

## Content Definition

**Email:**
- **Subject:** We received your support request
- **Body:**
  Hi {freelancer_first_name},

  We received your support request:

  "{request_text}"

  Dana will look into it and reply to you by email as soon as possible.

  -- Clientroom Support
- **CTA:** None -- this is a receipt, not an action prompt. Nadia's next step is to wait for Dana's reply, which is sent by email outside the product (Non-Goals); there is nothing in-product for her to do until then.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_first_name} | Freelancer Account -- name | Nadia | Never empty -- the freelancer's name is required at sign-up (FEAT-20, dependency map: Freelancer Account, name required); the greeting is never rendered without it |
| {request_text} | Support Access Session -- request_text | "I can't get my payment account to finish connecting." | Never empty -- request_text is required at submission (FEAT-31.SPEC-005); this email cannot be triggered by a record that lacks it |

## Delivery Rules

**Batching:** None -- each support request produces exactly one confirmation, delivered individually. Multiple requests from the same freelancer in the same day each get their own email, never combined, so each confirmation matches the specific message she sent.
**Deduplication:** Exactly one confirmation per Support Access Session record. The creating automation in FEAT-31.SPEC-001 fires this notification once, on successful creation only; there is no retry path that re-creates the same record, so no duplicate trigger exists.
**Retry on failure:** Retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`. After the final failure, the failure is surfaced to Nadia as a delivery warning on her account (FEAT-14's general delivery-failure pattern, XBR-30); her request itself is unaffected -- Dana still sees it in her queue (FEAT-31.SPEC-002) regardless of this email's delivery outcome.
**Expiry:** An unsent confirmation is not attempted further after the final retry (the end of platform parameter: `transactional-email-retry-window`). The record of her submitted request remains visible to her in-product through the Contact Support screen's own on-screen confirmation (FEAT-31.SPEC-001), which is not affected by this email's fate.

## Edge Cases

- **The request cannot be edited or withdrawn once submitted** -- No such capability exists in this feature (FEAT-31.SPEC-001, Non-Goals), so the confirmation always describes exactly what was sent; there is no scenario where the content changes between trigger and delivery.
- **Dana opens a session on Nadia's account before this confirmation is delivered** -- The confirmation's content is unaffected; it always describes only the original request. The session opening produces its own, separate notice (FEAT-31.SPEC-007).
- **Nadia's sign-in email changes (FEAT-21) between submitting the request and this email being sent** -- FEAT-21 requires re-verification before an email change takes effect; this notification is sent to whichever address is current and verified at the moment of the send attempt, since it is a one-time transactional email with no preference-evaluation window to wait for.
- **The freelancer account is deleted (FEAT-24) between request creation and this email being sent** -- FEAT-24's account deletion cascades immediately, including this Support Access Session and any pending notification; if the deletion completes before the send attempt, the confirmation is not sent.
- **Quiet hours colliding with expiry** -- Not applicable: this product defines no quiet-hours mechanism (see Audience and Preferences), so no such collision can occur for this or any other notification.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-31.SPEC-001 (Contact Support Screen) | Triggered by (inbound) | The submitted request creates the Support Access Session record that fires this notification |

## Analytics and Success Signals

- **support_request_confirmation_delivered** (channel: email) -- N/A -- no metric in success-metrics.md is connected to Operator Support Access or names this behavior; retained so confirmation delivery stays observable
- **support_request_confirmation_failed** (reason: retries_exhausted) -- N/A -- same reason as above

## Acceptance Criteria

**FEAT-31.SPEC-006-AC-01:** Given Nadia submits a support request describing a payment connection problem, when the Support Access Session record is created, then she receives an email with the subject "We received your support request" quoting her exact description back to her.

**FEAT-31.SPEC-006-AC-02:** Given Nadia submits two separate support requests on the same day, when both are created, then she receives two separate confirmation emails, each quoting its own request, never combined into one.

**FEAT-31.SPEC-006-AC-03:** Given this email fails to deliver on its first attempt, when the retry window runs, then it is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before being surfaced as a delivery warning on Nadia's account.

**FEAT-31.SPEC-006-AC-04:** Given this email exhausts all retries without delivering, when the final attempt fails, then no further attempt is made and Dana still sees the request in her queue regardless.

**FEAT-31.SPEC-006-AC-05:** Given Nadia's sign-in email is mid-change (pending re-verification) when this notification is triggered, then it is sent to whichever address is current and verified at the moment of the send attempt.

**FEAT-31.SPEC-006-AC-06:** Given Nadia's account is deleted before this email's send attempt completes, when the deletion finishes first, then this confirmation is never sent.

**FEAT-31.SPEC-006-AC-07:** Given Dana opens a session on Nadia's account moments after the request is submitted, when this confirmation email is delivered, then its content still describes only the original request, not the session opening.

**FEAT-31.SPEC-006-AC-08:** Given Nadia has no notification preferences that could disable this email, when a request is submitted, then the confirmation always sends -- there is no control anywhere that turns it off.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
