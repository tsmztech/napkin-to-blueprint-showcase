---
document_type: spec
spec_type: notification
spec_id: FEAT-18.SPEC-011
spec_name: Primary-Invited-Colleague Alert
spec_slug: primary-invited-colleague-alert
parent_feature: FEAT-18
parent_feature_name: Client Contact Management & Roles
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Notification Spec: Primary-Invited-Colleague Alert

## Overview

**Name:** Primary-Invited-Colleague Alert
**ID:** FEAT-18.SPEC-011
**Type:** Notification
**Purpose:** Alerts Nadia by email whenever a Primary contact invites a colleague, so she always knows who can see her work.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles

## Scope and Non-Goals

**In Scope:**
- The single email delivered to Nadia every time a Primary contact successfully invites a Reviewer colleague
- Content naming who invited whom and at which client
- Retry and expiry behavior for this counterpart-visibility alert

**Non-Goals:**
- Alerting Nadia when she herself adds a contact -- this notification exists specifically for the counterpart-visibility case (feature-overview.md, Communications: "so she always knows who can see her work"); an action Nadia performed herself needs no alert about itself
- The invitation email to the new contact -- owned by FEAT-18.SPEC-010 (New Contact Invitation Email), a separate notification to a separate recipient, fired by the same trigger
- Giving Nadia any control over whether the invited colleague's invitation proceeds -- this is a notice-only alert; product-features.md's Primary Flows & Alternates describes no approval step between Owen's invite and its completion, so this email is informational, not a gate
- An in-app notification-center entry for this alert -- excluded because In-App Notification Center (FEAT-29) is a Later-phase, Nice-to-Have feature (scope-boundaries.md); this MVP-phase alert is email-only, consistent with product-features.md's Communications field naming only an email

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, on every successful Primary-contact invite (FEAT-18.SPEC-004) | Nadia is not necessarily inside the product at the moment a client-side invite happens, and email is the product's sole channel for reaching her about events that originate on the client side (FEAT-14, Notification Delivery Reliability) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A Primary contact invites a Reviewer colleague | FEAT-18.SPEC-004 (Invite Reviewer Colleague) | Fires once, immediately after the invite send completes successfully | Inviting Primary contact's name, new contact's name and email, the client company, project context if applicable |

## Audience and Preferences

**Recipients:** Nadia -- the sole freelancer-side persona per the Access Matrix. This alert is never sent to any client contact; it exists solely so Nadia, the account owner, learns who can now see her work.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| None | N/A | N/A | N/A |

This alert reports a change to who can access Nadia's own client-facing work -- an access-relevant event the product does not treat as an optional, switchable notification (parallel to how a support-session-opened notice is always sent, per XBR-29). There is no preference screen exposing an on/off control for it, since product-features.md's Communications field states it plainly as something Nadia is always told, with no conditional framing.

**Quiet Hours:** N/A -- the product defines no quiet-hours window for any freelancer-side account-activity alert (no such window is named in product-features.md's Notifications & Help field or assumptions-constraints.md); this alert sends immediately, consistent with every other freelancer-facing account notice in this feature set.

## Content Definition

**Email:**
- **Subject:** {inviting_primary_name} invited a colleague to {client_company_name}
- **Body:**
  Hi {freelancer_name},

  {inviting_primary_name} invited {new_contact_name} ({new_contact_email}) as a Reviewer contact for {client_company_name}. They can view and comment on shared work, but cannot accept proposals, approve milestones, see invoices, or invite anyone else.

  You can review or manage this client's contacts and roles at any time.
- **CTA (button):** View contacts -- deep-links to FEAT-18.SPEC-001 (Client Contact List) for {client_company_name}

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|----------------|------------------------|
| {freelancer_name} | Freelancer Account -- name | Nadia | Greeting renders as "Hi there," |
| {inviting_primary_name} | Client Contact (the Primary contact who invited) -- name | Owen Carter | Never empty -- FEAT-18.SPEC-004 requires an authenticated Primary contact to trigger the invite; there is no anonymous-invite path |
| {new_contact_name} | Client Contact (the newly invited Reviewer) -- name | Priya Shah | Never empty -- name is a required field at creation (FEAT-18.SPEC-005) |
| {new_contact_email} | Client Contact (the newly invited Reviewer) -- email | priya@acme.test | Never empty -- email is a required field at creation (FEAT-18.SPEC-005) |
| {client_company_name} | Client -- client_name | Acme Co. | Never empty -- client_name is a required field (feature-dependency-map.md, Client entity) |

## Delivery Rules

**Batching:** None -- each successful invite produces exactly one alert to Nadia; two colleagues invited by the same or different Primary contacts at the same client in quick succession each produce their own separate alert, since each is an independently meaningful access-visibility event Nadia should see individually.
**Deduplication:** At most one alert per successful invite -- this notification fires exactly once, from the same trigger instance as FEAT-18.SPEC-010, and an invite send is not retried as a whole once it has succeeded (a failed invite attempt that Owen retries and resubmits produces its own new, separate successful-invite event, and therefore its own single alert).
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001), the same retry policy applied to every transactional email in this feature.
**Expiry:** If delivery has not succeeded after the final retry, no further attempt is made; unlike a time-limited credential email, there is no shrinking usability window for this alert's content, so it stands as a delivery failure state until Nadia's own next general delivery-warning review (FEAT-14, XBR-30) rather than expiring the information itself -- the new contact remains visible on FEAT-18.SPEC-001 regardless of whether this alert ever arrives.

## Edge Cases

- **The invited contact is removed before this alert is delivered** -- The alert still sends: it reports an event that already happened (Owen invited someone), independent of whatever the contact's current status is by the time delivery completes; Nadia is still entitled to know that Owen exercised his invite entitlement, even if the resulting contact no longer exists.
- **Two different Primary contacts at two different clients each invite a colleague within moments of each other** -- Each produces its own independent alert to Nadia, since each names a different client and a different inviting party; they are never merged, consistent with the No-Batching rule.
- **The same Primary contact invites two colleagues in quick succession** -- Two separate alerts are sent, one per invite, since each names a different new contact.
- **Delivery of this alert fails permanently** -- No further consequence beyond the standard delivery-failure surfacing (XBR-30); this is a lower-stakes failure than a credential email's failure, since Nadia can also simply notice the new contact by opening FEAT-18.SPEC-001 directly, but the failure is still surfaced like any other.
- **Nadia has no email notification preference screen bearing on this alert** -- Confirmed by design: this alert has no opt-out, consistent with its always-sends framing in product-features.md.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-004 (Invite Reviewer Colleague) | Triggered by (inbound) | Every successful invite send fires this notification |
| FEAT-18.SPEC-010 (New Contact Invitation Email) | References (inbound) | Fires alongside this notification from the same trigger, addressed to the new contact instead of Nadia |
| FEAT-18.SPEC-001 (Client Contact List) | Navigation (outbound) | The "View contacts" CTA deep-links here, scoped to the affected client |
| FEAT-14 (Notifications (Email)) | References (inbound) | Owns the underlying Transactional Email Delivery capability (FEAT-14.SPEC-001) this notification is sent through |

## Analytics and Success Signals

- **contact_invited_by_primary_alert_sent** (client_reference) -- N/A -- no success-metrics.md metric is connected to Client Contact Management & Roles; retained so counterpart-visibility alert delivery is observable
- **contact_invited_by_primary_alert_delivery_failed** (retry_count_exhausted: true) -- N/A -- no success-metrics.md metric is connected to this feature

## Acceptance Criteria

**FEAT-18.SPEC-011-AC-01:** Given Owen invites Priya as a Reviewer colleague, when the invite send completes, then Nadia receives an email with subject "Owen invited a colleague to {client_company_name}" naming Priya and her email.

**FEAT-18.SPEC-011-AC-02:** Given Nadia receives this alert, when she taps "View contacts", then she is navigated to FEAT-18.SPEC-001 scoped to the affected client.

**FEAT-18.SPEC-011-AC-03:** Given Nadia herself adds a new contact directly on FEAT-18.SPEC-002, when the save completes, then this alert is not sent, since it exists only for Primary-contact-initiated invites.

**FEAT-18.SPEC-011-AC-04:** Given two different Primary contacts at two different clients each invite a colleague within moments of each other, when both sends complete, then Nadia receives two separate alerts, one per client.

**FEAT-18.SPEC-011-AC-05:** Given Owen invites two colleagues in quick succession, when both sends complete, then Nadia receives two separate alerts, not one batched alert.

**FEAT-18.SPEC-011-AC-06:** Given the invited contact is removed before this alert is delivered, when delivery proceeds, then the alert still sends, reporting the invite event as it happened.

**FEAT-18.SPEC-011-AC-07:** Given delivery of this alert fails on the first attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`.

**FEAT-18.SPEC-011-AC-08:** Given all retries for this alert are exhausted, when the final attempt fails, then the failure is surfaced to Nadia per XBR-30's general delivery-warning behavior.

**FEAT-18.SPEC-011-AC-09:** Given this notification has no preference control, when a Primary contact invites a colleague, then the alert always sends to Nadia with no opt-out surface to check.

**FEAT-18.SPEC-011-AC-10:** Given this alert is triggered at any hour, when it fires, then it sends immediately with no quiet-hours hold, since the product defines no quiet-hours window for freelancer-side account alerts.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (no preference -- always sends) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
