---
document_type: spec
spec_type: notification
spec_id: FEAT-31.SPEC-007
spec_name: Support Session Opened Notice
spec_slug: support-session-opened-notice
parent_feature: FEAT-31
parent_feature_name: Operator Support Access
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 8
---

# Notification Spec: Support Session Opened Notice

## Overview

**Name:** Support Session Opened Notice
**ID:** FEAT-31.SPEC-007
**Type:** Notification
**Purpose:** Sends Nadia an email notice whenever a support session opens on her account, so the operator's access is always visibly announced to her, never silent.
**Parent Feature:** FEAT-31 -- Operator Support Access

## Scope and Non-Goals

**In Scope:**
- The emailed notice sent the instant a support session opens
- Its content, delivery rules, and edge cases

**Non-Goals:**
- A notice when the session closes -- product-features.md's Communications field names only the request-confirmation and session-opened emails; the close time reaches Nadia through her activity trail (FEAT-13), not a third email, per FEAT-31.SPEC-004's Non-Goals.
- The activity trail entry itself -- owned by Immutable Activity & Audit Trail (FEAT-13), written independently by FEAT-31.SPEC-003 at the same moment this notice is triggered; this spec covers only the email.
- Any content about what Dana is diagnosing or intends to change -- a session grants read-only visibility only, and this notice does not describe session activity beyond the fact that it opened, consistent with the unconditional read-only rule (FEAT-31.SPEC-005).

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, the instant a session opens | Nadia may not be inside the product at the moment Dana opens a session; ASMP-23 requires operator access to always be announced to the freelancer, and email is the only channel that reaches her outside an active session, matching product-features.md's Communications field |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A support session opens | FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement) | Fires the instant `operator` and `opened_at` are set on the Support Access Session record | Nadia's account name and email, `opened_at` |

## Audience and Preferences

**Recipients:** Nadia only -- the freelancer whose account the session was opened on. No other role receives this email: Owen and Priya have "None" for Support Access, and Dana is the actor who caused the event, not its recipient.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| Support session opened notice | Always on (transactional) | On | N/A -- per ASMP-23 and XBR-29, operator access is always announced to the freelancer; this is a transactional record email that cannot be disabled, consistent with XBR-30 |

**Quiet Hours:** N/A -- the product definition (FEAT-14, Notifications (Email)) establishes no quiet-hours mechanism for any notification; this email sends immediately regardless of time of day. A support session is itself a privacy-relevant event (ASMP-23), so an immediate, unheld notice is the only correct behavior even if a quiet-hours mechanism existed elsewhere in the product.

## Content Definition

**Email:**
- **Subject:** A support session was opened on your account
- **Body:**
  Hi {freelancer_first_name},

  Dana from Clientroom support opened a read-only session on your account at {opened_at_time} to help with your recent request.

  Everything Dana sees is read-only -- nothing can be changed, sent, approved, or paid during this session. You can see exactly when it started and ended in your activity trail.

  -- Clientroom Support
- **CTA:** View activity trail -- deep-links to FEAT-13.SPEC-001 (Activity Trail) for this account's session entries

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_first_name} | Freelancer Account -- name | Nadia | Never empty -- the freelancer's name is required at sign-up (dependency map: Freelancer Account, name required); the greeting is never rendered without it |
| {opened_at_time} | Support Access Session -- opened_at, shown in the freelancer's own time zone (FEAT-15) | 2:14 PM | Never empty -- this notification is triggered only after FEAT-31.SPEC-003 sets `opened_at`, so it always has a value |

## Delivery Rules

**Batching:** None -- each session opening is its own event, delivered individually. Because at most one session may be open per operator at a time (FEAT-31.SPEC-005), consecutive sessions on the same account cannot overlap in time; each still produces its own notice so Nadia's timeline of who looked, and when, stays exact.
**Deduplication:** Exactly one notice per `opened_at` timestamp. FEAT-31.SPEC-003 sets `operator` and `opened_at` exactly once per session and triggers this notification exactly once on that write; a session, once opened, is never re-opened (FEAT-31.SPEC-005, Edge Cases).
**Retry on failure:** Retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, consistent with FEAT-31.SPEC-006. After the final failure, it is surfaced as a delivery warning on Nadia's account (FEAT-14's general delivery-failure pattern, XBR-30).
**Expiry:** After the final retry (the end of platform parameter: `transactional-email-retry-window`), no further attempt is made. Nadia's activity trail entry (written independently by FEAT-13.SPEC-003 at the moment the session opened) remains the permanent, always-visible record of the session regardless of this email's delivery outcome -- satisfying ASMP-23's "always announced" requirement even if the email itself is never delivered.

## Edge Cases

- **The session Dana opened closes almost immediately after opening (auto-close on inactivity, FEAT-31.SPEC-004)** -- This notice still sends normally; FEAT-31.SPEC-004 does not cancel or suppress it, since the event this notice announces (the session opening) already occurred.
- **Nadia's account is deleted (FEAT-24) between the session opening and this email's send attempt** -- FEAT-24's cascade removes the Support Access Session and any pending notification; if the deletion completes first, delivery is cancelled with the account.
- **Multiple support requests are pending but only one session opens** -- This notice describes only the session that actually opened, at its own `opened_at` time; it never lists or references the freelancer's other pending, unopened requests.
- **This email fails on every retry** -- Nadia may not learn about the session by email, but the activity trail entry (FEAT-13) remains the authoritative record; ASMP-23's guarantee is satisfied by the trail even when the email itself never arrives.
- **Quiet hours colliding with expiry** -- Not applicable: this product defines no quiet-hours mechanism (see Audience and Preferences), so no such collision can occur.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement) | Triggered by (inbound) | The session-opening write fires this notification |
| FEAT-13.SPEC-001 (Activity Trail) | Navigation (outbound) | The CTA deep-links to Nadia's activity trail |
| FEAT-13.SPEC-003 (Activity Entry Recording) | References (inbound) | The same open event also produces the trail entry independently of this email |

## Analytics and Success Signals

- **support_session_opened_notice_delivered** (channel: email) -- N/A -- no metric in success-metrics.md is connected to Operator Support Access or names this behavior; retained so notice delivery stays observable
- **support_session_opened_notice_failed** (reason: retries_exhausted) -- N/A -- same reason as above

## Acceptance Criteria

**FEAT-31.SPEC-007-AC-01:** Given Dana opens a session on Nadia's account at 2:14 PM (her local time), when FEAT-31.SPEC-003 sets `operator` and `opened_at`, then Nadia receives an email with the subject "A support session was opened on your account" naming that exact time.

**FEAT-31.SPEC-007-AC-02:** Given this notice is delivered, when Nadia taps "View activity trail," then she lands on her Activity Trail (FEAT-13.SPEC-001) and sees the session entry.

**FEAT-31.SPEC-007-AC-03:** Given the session Dana opened closes automatically within a minute of opening, when this notice's delivery attempt runs, then it still sends normally.

**FEAT-31.SPEC-007-AC-04:** Given Nadia has three pending, unopened support requests and Dana opens a session on only one, when this notice is delivered, then it describes only the session that opened, not the other pending requests.

**FEAT-31.SPEC-007-AC-05:** Given this email fails to deliver on its first attempt, when the retry window runs, then it is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before being surfaced as a delivery warning on Nadia's account.

**FEAT-31.SPEC-007-AC-06:** Given this email exhausts all retries without delivering, when the final attempt fails, then Nadia's activity trail entry for the session remains visible regardless.

**FEAT-31.SPEC-007-AC-07:** Given Nadia's account is deleted before this email's send attempt completes, when the deletion finishes first, then this notice is never sent.

**FEAT-31.SPEC-007-AC-08:** Given Nadia has no notification preferences that could disable this email, when a session opens on her account, then the notice always sends -- there is no control anywhere that turns it off.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
