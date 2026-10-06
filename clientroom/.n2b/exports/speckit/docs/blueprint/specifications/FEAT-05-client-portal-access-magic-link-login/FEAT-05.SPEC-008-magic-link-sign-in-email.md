---
document_type: spec
spec_type: notification
spec_id: FEAT-05.SPEC-008
spec_name: Magic Link Sign-In Email
spec_slug: magic-link-sign-in-email
parent_feature: FEAT-05
parent_feature_name: Client Portal Access (Magic-Link Login)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Notification Spec: Magic Link Sign-In Email

## Overview

**Name:** Magic Link Sign-In Email
**ID:** FEAT-05.SPEC-008
**Type:** Notification
**Purpose:** Emails the one-time sign-in link to the requesting contact whenever a link is issued, so they can enter their scoped portal without a password.
**Parent Feature:** FEAT-05 -- Client Portal Access (Magic-Link Login)

## Scope and Non-Goals

**In Scope:**
- The single-recipient email delivered on every successful magic-link issuance
- Content and branding for the email, on the one channel this communication uses
- Retry and expiry behavior tied to the underlying token's own time limit

**Non-Goals:**
- Deciding whether an email is recognized and generating the token -- owned by FEAT-05.SPEC-004 (Magic Link Issuance); this spec begins once issuance hands off an already-generated token.
- A standalone Integration spec for the underlying email-sending capability -- the transactional email capability this notification relies on is owned by FEAT-14 (External Touchpoints table) and recorded here only as a cross-feature touchpoint, consistent with how FEAT-02 and FEAT-03 use the same capability.
- In-app or push delivery of the sign-in link -- excluded because a contact with no active session, by definition, has no in-app surface to receive a push or in-app notification on; email is the only channel that can reach someone who is not currently signed in.
- Notifying anyone other than the requesting contact -- excluded per BRIEF.md's Constraints on strict client isolation: this is a single-recipient, single-purpose credential email, never CC'd or forwarded to another contact or to the freelancer.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, on every successful issuance (FEAT-05.SPEC-004) | The recipient has no active session and no in-app surface to reach; email is the only channel that can deliver a credential to someone who is, by definition, signed out (BRIEF.md, Target Users & Roles: contacts "sign in passwordless with a magic link by email") |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Link issued | FEAT-05.SPEC-004 (Magic Link Issuance) | Fires once per matched Client Contact record when a token is successfully generated | Token (link), Client Contact's `email`, the owning freelancer's Branding Profile |

## Audience and Preferences

**Recipients:** The single Client Contact (Owen or Priya, per the Access Matrix) whose email was recognized and matched by FEAT-05.SPEC-004. When a person is a Client Contact for more than one freelancer, each freelancer's issuance sends its own separate email to that same address, each carrying only that freelancer's own link and branding.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| None | N/A | N/A | N/A |

This email carries a single-use sign-in credential the contact themselves just requested; per XBR-30, transactional emails core to the record always send and are not subject to an optional-notification preference. There is no preference screen for a client contact to manage, consistent with the Access Matrix (client contacts have no Settings surface of their own).

**Quiet Hours:** N/A -- this email is a direct, immediate response to the contact's own just-completed action (requesting a link) and carries a time-limited credential; holding it for a quiet-hours window would shrink the usable portion of the token's own time limit and contradict the immediacy the request implies.

## Content Definition

**Email:**
- **Subject:** Sign in to your {freelancer_business_name} client portal
- **Body:**
  Hi {contact_name},

  Click the button below to sign in. This link is single-use and expires soon, so use it right away.

  If you didn't request this, you can ignore this email -- no one can sign in without clicking the link themselves.
- **CTA (button):** Sign in -- deep-links to FEAT-05.SPEC-002 (Link Verification Landing) with the issued token, which then routes to FEAT-05.SPEC-003 (Portal Home) on success

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|----------------|------------------------|
| {freelancer_business_name} | Freelancer Account -- business_name (or, before that is set, the freelancer's own name) | Nadia Ross Design | Renders as "your client portal" (subject becomes "Sign in to your client portal") -- required before the first invoice per the dependency map, but this email can fire earlier, so the fallback covers that window |
| {contact_name} | Client Contact -- name | Owen Carter | Greeting renders as "Hi there," |

The freelancer's Branding Profile `logo` and `brand_colour` (or the neutral default) apply to the email's visual header per XBR-31; the discreet "Made with Clientroom" referral mark appears per XBR-32, alongside the branding without overriding it.

## Delivery Rules

**Batching:** None -- each issuance produces exactly one email for exactly one token; a contact requesting a fresh link before a prior one is used still receives a new, separate email for the new token (the prior token is invalidated per FEAT-05.SPEC-006, so there is never more than one valid credential to batch).
**Deduplication:** At most one email per issued token -- FEAT-05.SPEC-004 triggers this notification exactly once per successful issuance; a token is never re-issued, so no duplicate email for the same token can occur.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). Because the underlying token is single-use and time-limited (FEAT-05.SPEC-006), a retry that succeeds after part of the token's validity window has already elapsed still delivers a usable link as long as it arrives before platform parameter: `magic-link-expiry-window` has fully elapsed from issuance; the retry window is chosen to fit inside the token's own expiry so a delayed-but-successful delivery is not itself worthless.
**Expiry:** If delivery has not succeeded after the final retry, no further attempt is made and the delivery failure is surfaced to the freelancer as a warning (XBR-30), since email is the only channel reaching the contact and a silently lost sign-in email would leave the contact unable to reach the portal at all, with no indication anything was wrong.

## Edge Cases

- **Contact's email bounces (invalid or full mailbox)** -- Treated as a delivery failure per the Retry on failure rule; after the final retry, the bounce is surfaced to the freelancer as a delivery warning on the client's contact record (FEAT-14, XBR-30), since the freelancer -- not the contact, who never confirms receipt -- is the only party positioned to correct the contact's email.
- **Contact requests a new link before this email for the prior link has finished sending** -- The prior token is invalidated (FEAT-05.SPEC-006) independently of this email's delivery state; if the first email later succeeds in delivering, its link shows the shared "not valid anymore" explanation when clicked, since the token itself -- not the email -- carries validity.
- **The token expires before the email is delivered (extreme delivery delay)** -- The delivered email's link will show "not valid anymore" when clicked; this is treated as an issuance/delivery-timing rarity rather than a notification defect, since platform parameter: `magic-link-expiry-window` is set wide enough that a normal delivery, including one retry cycle, comfortably completes within it.
- **A person is a Client Contact for two freelancers and requests a link from a page that only shows one context** -- Each freelancer's matched issuance triggers its own separate email, sent independently and simultaneously; the two emails carry different branding and different tokens, and neither references the other.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-05.SPEC-004 (Magic Link Issuance) | Triggered by (inbound) | Every successful issuance fires this notification |
| FEAT-05.SPEC-006 (Link Validity & Recognition Rules) | References (inbound) | Governs the token's expiry window this email's retry rule is bounded by |
| FEAT-05.SPEC-002 (Link Verification Landing) | Navigation (outbound) | The "Sign in" CTA deep-links here with the issued token |
| FEAT-19 (Freelancer Branding) | References (inbound) | Supplies the logo and brand colour applied to this email |
| FEAT-33 (Portal Referral Attribution) | References (inbound) | Supplies the referral mark shown alongside branding |
| FEAT-14 (Notifications (Email)) | References (inbound) | Owns the underlying Transactional Email Delivery capability (FEAT-14.SPEC-001) this notification is sent through |

## Analytics and Success Signals

- **magic_link_email_sent** (matched_freelancer_count context inherited from the triggering issuance) -- supports success-metrics.md: "Client Portal Login Success"
- **magic_link_email_delivery_failed** (retry_count_exhausted: true) -- supports success-metrics.md: "Client Portal Login Success" (a lost sign-in email is the clearest cause of a failed first-try sign-in this metric tracks)

## Acceptance Criteria

**FEAT-05.SPEC-008-AC-01:** Given Owen's email is recognized and a token is issued, when FEAT-05.SPEC-004 completes, then Owen receives an email with subject "Sign in to your {his freelancer's business name} client portal" and a "Sign in" button linking to his issued token.

**FEAT-05.SPEC-008-AC-02:** Given Priya's freelancer's `business_name` is not yet set, when her sign-in email is composed, then the subject renders as "Sign in to your client portal" and the greeting still uses her name.

**FEAT-05.SPEC-008-AC-03:** Given Owen taps the "Sign in" button in the email, when the link opens, then he lands on FEAT-05.SPEC-002 (Link Verification Landing) with his token.

**FEAT-05.SPEC-008-AC-04:** Given Owen is a Client Contact for two freelancers and requests a link once, when both issuances complete, then he receives two separate emails, each with that freelancer's own branding and its own token.

**FEAT-05.SPEC-008-AC-05:** Given delivery of Priya's sign-in email fails on the first attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before giving up.

**FEAT-05.SPEC-008-AC-06:** Given all retries for Owen's sign-in email are exhausted, when the final attempt fails, then Nadia sees a delivery warning on the client's contact record, since Owen has no other way to be reached.

**FEAT-05.SPEC-008-AC-07:** Given Priya requests a fresh link while a prior email is still in transit, when the prior email eventually delivers, then clicking its link shows "This link isn't valid anymore," since FEAT-05.SPEC-006 invalidated the prior token independently of this email's delivery.

**FEAT-05.SPEC-008-AC-08:** Given this notification has no preference control for the recipient, when a Client Contact requests a link, then the email always sends -- there is no opt-out surface to check.

**FEAT-05.SPEC-008-AC-09:** Given a link is requested at any hour, when the email is triggered, then it sends immediately with no quiet-hours hold, since the underlying credential is time-limited.

**FEAT-05.SPEC-008-AC-10:** Given the freelancer's Branding Profile has a logo and brand colour set, when this email renders, then it displays that branding alongside the "Made with Clientroom" referral mark, with the mark never overriding the branding.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (no preference -- always sends) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 4 | 4 |
