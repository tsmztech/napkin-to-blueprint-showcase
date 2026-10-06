---
document_type: spec
spec_type: notification
spec_id: FEAT-27.SPEC-004
spec_name: Custom Domain Verified Confirmation
spec_slug: custom-domain-verified-confirmation
parent_feature: FEAT-27
parent_feature_name: Custom Domain per Freelancer
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Notification Spec: Custom Domain Verified Confirmation

## Overview

**Name:** Custom Domain Verified Confirmation
**ID:** FEAT-27.SPEC-004
**Type:** Notification
**Purpose:** Emails Nadia once her custom domain is verified and live, so she knows without checking the settings screen that her portal now serves at her own domain.
**Parent Feature:** FEAT-27 -- Custom Domain per Freelancer

## Scope and Non-Goals

**In Scope:**
- The confirmation email sent once FEAT-27.SPEC-002 reports the domain verified and live
- Its single channel (email), content, and delivery behavior

**Non-Goals:**
- Deciding whether a domain is verified -- owned by FEAT-27.SPEC-002 (Domain Verification & Secure Serving); this spec begins only once that spec's verification-succeeded event fires
- Any notification for a failed or re-checked-and-still-failing verification -- product-features.md's Communications field for this feature names only "Confirmation email once the domain is verified and live"; a failure is surfaced inline on FEAT-27.SPEC-001, never by email, so Nadia is not alerted twice for the same event
- In-app or push delivery -- excluded per product-features.md and the dependency map, which define email as the product's sole notification channel; there is no in-app notification surface in MVP (FEAT-29, In-App Notification Center, is Later phase and not connected to this feature)
- Notifying Owen or Priya -- excluded per feature-overview.md: this is Nadia's own configuration event, and Owen and Priya "experience whichever domain this spec determines is active, with no direct control" and no entitlement to be told about the change

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, once verification succeeds | Nadia works from a laptop or desktop and is not necessarily on the Custom Domain Settings screen at the moment verification completes, since propagation can take time; email is the product's sole notification channel and reaches her the moment the one-time configuration step she started actually finishes |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Domain verified and live | FEAT-27.SPEC-002 (Domain Verification & Secure Serving) | Fires once, when the verification-succeeded inbound event sets verification_state to Verified (including a re-check that succeeds) | Freelancer Account identity, Custom Domain Record's domain_name |

## Audience and Preferences

**Recipients:** Nadia (Freelancer) -- the sole recipient. Per the Access Matrix, this event concerns Nadia's own configuration of her account; Owen and Priya hold no entitlement to it (they only experience whichever domain is active, with no visibility into its configuration, per feature-overview.md), and Dana (Support Operator) never receives product notifications addressed to the freelancer -- her Access Matrix entry for Notifications & Help is limited to viewing delivery warnings, not receiving this or any other freelancer-addressed email.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| (none) | -- | Always sent | -- (this is a transactional, record-core confirmation of a change to how Nadia's own portal is served; XBR-30 places it outside the optional preferences governed by FEAT-21.SPEC-002 and FEAT-21.SPEC-008, alongside the product's other account-critical confirmations) |

**Quiet Hours:** N/A -- the product defines no quiet-hours window for any notification (see FEAT-21.SPEC-011's equivalent decision for the product's other transactional confirmation), and this confirmation in particular reports the completion of a step Nadia herself initiated, so holding it back would only leave her wondering whether it finished.

## Content Definition

**Email:**
- **Subject:** Your custom domain is verified and live
- **Body:**
  Hi {freelancer_name},

  Your domain {domain_name} is now verified. Your portal is reachable there, and it's still reachable at your shared default address too.

  Nothing else changes -- your clients see the same portal, now under your own domain.
- **CTA (button):** View custom domain settings -- deep-links to FEAT-27.SPEC-001 (Custom Domain Settings) showing this domain's Verified state

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_name} | Freelancer Account -- name | Nadia Voss | Greeting renders as "Hi," -- name is never empty in practice since it is required (dependency map, Freelancer Account fields), but the fallback exists for defensive completeness |
| {domain_name} | Custom Domain Record -- domain_name, the value that just reached verification_state Verified | myportal.com | Never empty -- this notification only fires once FEAT-27.SPEC-002 reports a specific domain verified; there is no case where verification succeeds for an unset domain_name |

## Delivery Rules

**Batching:** None -- each verification-succeeded event produces exactly one email, sent individually. Since at most one Custom Domain Record exists per freelancer account (dependency map, Relationships), there is never more than one domain's verification to batch.
**Deduplication:** At most one confirmation per verification-succeeded event. A verification-succeeded event delivered twice for the same already-Verified record (FEAT-27.SPEC-002's own edge case) does not trigger a second email; only the transition into Verified sends this notification, not every subsequent report that the state remains Verified.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning, consistent with XBR-30's "delivery failures are surfaced to the freelancer as warnings"; because this confirmation is account-level rather than project-level, the warning surfaces on the Custom Domain Settings screen (FEAT-27.SPEC-001) rather than on a project.
**Expiry:** This notification never expires unsent in a way that discards it -- the underlying fact (the domain is verified and live) remains true and visible on FEAT-27.SPEC-001 regardless of whether the email itself is ever delivered. If delivery ultimately fails after all retries, the delivery-failure warning above is the surviving signal.

## Edge Cases

- **The Custom Domain Record is removed shortly after verification succeeds but before this email is delivered** -- The email is still delivered, since it reports a real, completed event (the domain was in fact verified); it is a historical confirmation, not a live status view, so Nadia may receive confirmation for a domain she has since removed. The email's content is unaffected because domain_name is captured at trigger time, not re-read at delivery time.
- **Nadia replaces her domain again before this email is delivered** -- The pending email for the first domain's verification is still delivered as written, describing that domain; the replacement domain's own eventual verification (if it succeeds) produces its own, separate confirmation email.
- **The verification-succeeded event fires twice for the same domain (e.g., a delayed duplicate report)** -- Per the Deduplication rule, only the first transition into Verified sends this notification; the duplicate report changes nothing and sends nothing.
- **Quiet hours or a preference change between trigger and delivery** -- Not applicable: this notification has no preference control and no quiet-hours window (see Audience and Preferences), so there is no collision to resolve.
- **Delivery fails because the domain-related confirmation email itself is misrouted by an overzealous spam filter tuned to the new domain** -- Handled identically to any other delivery failure: retried per the Retry rule above, then surfaced as a delivery warning on FEAT-27.SPEC-001; the domain's Verified state and live serving are entirely unaffected by whether this confirmation email is delivered.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-002 (Domain Verification & Secure Serving) | Triggered by (inbound) | The verification-succeeded event fires this notification exactly once per transition into Verified |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | Triggers (outbound) | Delivers the email and reports delivery/bounce/failure status |
| FEAT-27.SPEC-001 (Custom Domain Settings) | Navigation (outbound) | The CTA deep-links here, showing the domain's Verified state; delivery-failure warnings for this notification also surface here |

## Analytics and Success Signals

- **custom_domain_verified_confirmation_sent** (channel: email) -- N/A -- no success-metrics.md metric is connected to Custom Domain per Freelancer (FEAT-27); retained per product-features.md's own Communications and Signals fields so the confirmation remains observable
- **custom_domain_verified_confirmation_delivery_failed** (retry_count) -- N/A -- no connected success-metrics.md metric; retained so a failed confirmation is observable rather than silent, consistent with XBR-30's delivery-failure-warning requirement
- **custom_domain_verified_confirmation_cta_tapped** () -- N/A -- no connected success-metrics.md metric; retained to observe whether Nadia opens her domain settings after being confirmed live

## Acceptance Criteria

**FEAT-27.SPEC-004-AC-01:** Given Nadia's domain reaches verification_state Verified, when this notification fires, then she receives an email with the subject "Your custom domain is verified and live" naming her verified domain.

**FEAT-27.SPEC-004-AC-02:** Given Nadia receives this confirmation email, when she taps "View custom domain settings", then she lands on FEAT-27.SPEC-001 showing her domain's Verified state.

**FEAT-27.SPEC-004-AC-03:** Given a verification-succeeded event is delivered twice for the same already-Verified record, when the second delivery is processed, then no second confirmation email is sent.

**FEAT-27.SPEC-004-AC-04:** Given there is no notification preference for this email, when Nadia has every optional notification turned off on FEAT-21.SPEC-002, then this confirmation is still sent whenever her domain verifies.

**FEAT-27.SPEC-004-AC-05:** Given delivery of this email fails once for a transient reason, when the delivery capability retries within platform parameter: `transactional-email-retry-window`, then up to platform parameter: `transactional-email-retry-count` retries occur before any failure is surfaced to Nadia.

**FEAT-27.SPEC-004-AC-06:** Given delivery of this email fails after all retries are exhausted, when the final failure is processed, then a delivery-failure warning appears on FEAT-27.SPEC-001.

**FEAT-27.SPEC-004-AC-07:** Given Nadia removes her domain shortly after verification but before this email is delivered, when delivery proceeds, then the email is still sent describing the domain that was verified.

**FEAT-27.SPEC-004-AC-08:** Given Nadia's domain fails verification (or a re-check still fails), then this notification never fires for that event -- only a successful transition into Verified triggers it.

**FEAT-27.SPEC-004-AC-09:** Given Owen, Priya, or Dana, then none of them ever receives this notification, since only Nadia is entitled to it.

**FEAT-27.SPEC-004-AC-10:** Given Nadia replaces her domain again before this email for the first domain is delivered, when delivery proceeds, then the email is delivered describing the first, already-verified domain, unaffected by the later replacement.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (no preference -- always sent) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
