---
document_type: spec
spec_type: notification
spec_id: FEAT-02.SPEC-011
spec_name: Proposal Sent/Resent Email
spec_slug: proposal-sent-resent-email
parent_feature: FEAT-02
parent_feature_name: Proposal Creation & Sending
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Notification Spec: Proposal Sent/Resent Email

## Overview

**Name:** Proposal Sent/Resent Email
**ID:** FEAT-02.SPEC-011
**Type:** Notification
**Purpose:** Emails the client's Primary Contact the proposal link whenever a proposal is sent, resent, or re-sent after an edit, so the client can review and act on it without a "PDF in email" back-and-forth.
**Parent Feature:** FEAT-02 -- Proposal Creation & Sending

## Scope and Non-Goals

**In Scope:**
- The email delivered on an original send (FEAT-02.SPEC-005), an unchanged resend (FEAT-02.SPEC-007), and a void-and-resend after an edit (FEAT-02.SPEC-006)
- Content variants distinguishing a first send from a resend
- Delivery, retry, and expiry behavior for this transactional record email

**Non-Goals:**
- Deciding when a proposal is sent, resent, or void-and-resent -- owned by FEAT-02.SPEC-005, FEAT-02.SPEC-006, and FEAT-02.SPEC-007; this spec begins where each of their triggers fires.
- The magic-link sign-in flow the email's link leads into -- owned by FEAT-05 (Client Portal Access); this spec defines the email's content and the link's destination, not the sign-in mechanics.
- Confirming acceptance back to Nadia -- owned by FEAT-03 (Proposal Acceptance), which sends its own confirmation once Owen acts; this spec covers only the send/resend email, not the acceptance confirmation.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, on every send, resend, and void-and-resend | Per BRIEF.md, Ecosystem & Integrations, clients will not install an app; email is the sole channel that reaches Owen, and this is a transactional record he cannot opt out of (XBR-30) |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Proposal sent (original) | FEAT-02.SPEC-005 (Proposal Send) | Fires when a Draft successfully transitions to Sent | Project name, client name, scope_description, price, currency, sent_at, Primary Contact name/email, freelancer's Branding Profile |
| Proposal edited and re-sent | FEAT-02.SPEC-006 (Proposal Edit-Before-Acceptance Void & Resend) | Fires when a void-and-resend successfully creates and sends the new version | Same as above, for the new version, plus an indicator that this supersedes a prior version |
| Proposal resent (unchanged) | FEAT-02.SPEC-007 (Proposal Resend) | Fires when a resend of the current Sent version succeeds | Same as the original-send data, for the existing (unchanged) version |

## Audience and Preferences

**Recipients:** Owen -- the client's Primary Contact(s) (Access Matrix: Own-only view, accept, request changes, sign). Priya (Reviewer) never receives this email, since Reviewers have no access to proposal content at all, per the Access Matrix. Nadia does not receive this email herself; she sees the resulting Sent status on FEAT-02.SPEC-003.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this is a transactional record email | -- | Always on | -- |

This email is a transactional record central to the proposal's status, not an optional notification -- per XBR-30, transactional emails core to the record always send and cannot be switched off.

**Quiet Hours:** N/A -- the product defines no quiet-hours window for this email; it is a time-sensitive, action-required message tied directly to a freelancer-initiated action (send, edit-resend, resend), and holding it would delay the exact moment the client needs to act.

## Content Definition

**Email (original send):**
- **Subject:** New proposal from {freelancer_business_name}: {project_name}
- **Body:**
  Hi {contact_name},

  {freelancer_business_name} has sent you a proposal for {project_name}.

  Scope: {scope_summary}
  Price: {price_formatted}

  Review the full proposal and let {freelancer_business_name} know if you'd like to accept it or request changes.
- **CTA (button):** View Proposal -- deep-links to FEAT-03.SPEC-001 (Proposal Review & Accept) [Owen's own portal proposal view, owned by FEAT-03] for this project's proposal, via magic-link sign-in (FEAT-05)

**Email (edited and re-sent):**
- **Subject:** Updated proposal from {freelancer_business_name}: {project_name}
- **Body:**
  Hi {contact_name},

  {freelancer_business_name} has updated the proposal for {project_name}. The earlier version is no longer valid -- please review the current one below.

  Scope: {scope_summary}
  Price: {price_formatted}

  Review the full proposal and let {freelancer_business_name} know if you'd like to accept it or request changes.
- **CTA (button):** View Updated Proposal -- deep-links to FEAT-03.SPEC-001 (Proposal Review & Accept) [Owen's portal proposal view, FEAT-03] for this project's proposal, via magic-link sign-in (FEAT-05)

**Email (resent, unchanged):**
- **Subject:** Reminder: proposal from {freelancer_business_name} for {project_name}
- **Body:**
  Hi {contact_name},

  Here's the link to the proposal {freelancer_business_name} sent you for {project_name}, in case the earlier email is hard to find.

  Scope: {scope_summary}
  Price: {price_formatted}
- **CTA (button):** View Proposal -- deep-links to FEAT-03.SPEC-001 (Proposal Review & Accept) [Owen's portal proposal view, FEAT-03] for this project's proposal, via magic-link sign-in (FEAT-05)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {contact_name} | Client Contact -- name | Owen Marsh | Never empty -- contact name is required at contact creation (FEAT-18) |
| {freelancer_business_name} | Freelancer Account -- business_name | Studio Nadia | Falls back to the freelancer's account name if business_name is not yet set |
| {project_name} | Project -- project_name | Autumn Rebrand | Never empty -- required at project creation (FEAT-01) |
| {scope_summary} | Proposal -- scope_description (first plain-text excerpt) | "Full brand identity refresh including logo, colour system, and style guide." | Never empty -- scope_description is required to send (FEAT-02.SPEC-010) |
| {price_formatted} | Proposal -- price and currency | $4,500.00 USD | Never empty -- price is required and positive to send (FEAT-02.SPEC-010) |

The email's visual presentation (header band, logo placement, brand colour) applies the freelancer's Branding Profile (FEAT-19) consistently with FEAT-02.SPEC-002 (Proposal Preview), per XBR-31, falling back to a neutral default when the freelancer has not set one.

## Delivery Rules

**Batching:** None -- each send, resend, or edit-resend is delivered as its own individual email at the moment its trigger fires. A proposal is never batched with any other notification, since it is a singular, time-sensitive action tied to one specific event.
**Deduplication:** At most one email per triggering event. A resend triggered while the rate-limit window (FEAT-02.SPEC-007) is active is blocked at the automation level before this notification is ever triggered, so no duplicate email for the same resend attempt is possible.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the delivery failure is surfaced to Nadia as a warning on the project (XBR-30), since email is the only channel reaching Owen and a silently lost proposal email would leave him unaware a proposal exists.
**Expiry:** This email does not expire in the sense of being withdrawn -- if delivery eventually succeeds after retries, it is still the correct, current content, since the Proposal Detail view it links to always reflects the current version. If all retries are exhausted, the email is not delivered at all and the project-level delivery warning (above) is the surviving signal for Nadia to resend manually (FEAT-02.SPEC-007).

## Edge Cases

- **The proposal is voided (edited and re-sent) before the original send email's retries are exhausted** -- The original send's pending retry is superseded: only the edited version's email is worth delivering, since the original's content is no longer current. The original email is not attempted further; the edited version's own email proceeds through its own delivery and retry cycle.
- **The client's Primary contact email address changes between trigger and delivery** -- The email is addressed to the Primary contact's current email at the moment of actual delivery, not at the moment the automation triggered, so a contact update made in the interim is respected (consistent with FEAT-18 owning contact data as the source of truth).
- **The proposal is discarded -- not applicable to this notification, since Discard (FEAT-02.SPEC-009) only ever applies to a Draft, and a Draft never triggers this email; no disappearing-record scenario exists for this spec's own trigger paths.**
- **Delivery fails on all retries for an original send** -- The delivery warning appears on the project for Nadia (XBR-30); Owen never learns a proposal was intended for him until Nadia notices the warning and uses Resend (FEAT-02.SPEC-007) once the underlying delivery issue (e.g., an invalid address) is corrected via FEAT-18.
- **The client has more than one Primary contact** -- Every current Primary contact receives their own copy of the email, addressed individually, per the Access Matrix's "Own-only" entitlement applying to all Primary contacts of the client company equally.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-005 (Proposal Send) | Triggered by (inbound) | Original send fires this notification |
| FEAT-02.SPEC-006 (Void & Resend) | Triggered by (inbound) | Edit-and-resend fires the edited-version variant |
| FEAT-02.SPEC-007 (Proposal Resend) | Triggered by (inbound) | Unchanged resend fires the resend variant |
| FEAT-03.SPEC-001 (Proposal Review & Accept) | Navigation (outbound) | Every CTA deep-links to Owen's portal proposal view, which surfaces the same status this feature's Detail screen (FEAT-02.SPEC-003) shows Nadia |
| FEAT-03 (Proposal Acceptance) | References (outbound) | The CTA's destination is owned by FEAT-03 as Owen's portal-side proposal view |
| FEAT-05 (Client Portal Access) | References (outbound) | The CTA routes through magic-link sign-in before reaching the proposal view |
| FEAT-18 (Client Contact Management & Roles) | References (inbound) | Recipient identity and current email address sourced from here |
| FEAT-19 (Freelancer Branding) | References (inbound) | Branding Profile applied to the email's visual presentation (XBR-31) |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | Underlying delivery, retry, and bounce/failure reporting capability this notification is sent through |

## Analytics and Success Signals

- **proposal_email_delivered** (variant: original / edited / resent) -- supports success-metrics.md: "Notification Delivery Reliability"
- **proposal_email_opened** (variant) -- supports success-metrics.md: "Time to Proposal Acceptance" (an opened email is the precondition for the acceptance-speed clock the metric measures)
- **proposal_email_cta_tapped** (variant) -- supports success-metrics.md: "Time to Proposal Acceptance"
- **proposal_email_delivery_failed** (variant, retry_count_exhausted: yes/no) -- supports success-metrics.md: "Notification Delivery Reliability"

## Acceptance Criteria

**FEAT-02.SPEC-011-AC-01:** Given a Draft proposal is successfully sent (FEAT-02.SPEC-005), when this notification fires, then Owen receives an email with the subject "New proposal from {freelancer_business_name}: {project_name}" and a "View Proposal" CTA.

**FEAT-02.SPEC-011-AC-02:** Given a Sent-but-unaccepted proposal is edited and re-sent (FEAT-02.SPEC-006), when this notification fires, then Owen receives an email with the subject "Updated proposal from {freelancer_business_name}: {project_name}" stating the earlier version is no longer valid.

**FEAT-02.SPEC-011-AC-03:** Given Nadia resends an unchanged Sent proposal (FEAT-02.SPEC-007), when this notification fires, then Owen receives an email with the subject "Reminder: proposal from {freelancer_business_name} for {project_name}".

**FEAT-02.SPEC-011-AC-04:** Given the client has two Primary contacts, when a proposal is sent, then both receive their own copy of the email, each addressed individually.

**FEAT-02.SPEC-011-AC-05:** Given Priya (Reviewer) is a contact on the client company, when a proposal is sent, then she does not receive this email.

**FEAT-02.SPEC-011-AC-06:** Given Owen taps the "View Proposal" CTA, then he is routed through magic-link sign-in (FEAT-05) and lands on his portal proposal view (FEAT-03) for this project.

**FEAT-02.SPEC-011-AC-07:** Given delivery of the original-send email fails, when the retry window is exhausted, then a delivery warning appears on the project for Nadia and no further automatic retry occurs.

**FEAT-02.SPEC-011-AC-08:** Given the original send's email delivery is still retrying when the proposal is edited and re-sent, then the original email is not delivered further and only the edited version's email proceeds.

**FEAT-02.SPEC-011-AC-09:** Given the Primary contact's email address is updated between trigger and actual delivery, then the email is delivered to the current address at delivery time.

**FEAT-02.SPEC-011-AC-10:** Given the freelancer has not set a Branding Profile, when this email renders, then it uses the neutral default branding.

**FEAT-02.SPEC-011-AC-11:** Given Nadia has not set any notification preference for this email, when a proposal is sent, then the email always sends -- there is no preference control that can turn it off, per XBR-30.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 3 (original send, edit-resend, resend) | 3 |
| Preference States | 1 (always on) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
