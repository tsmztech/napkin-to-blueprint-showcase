---
document_type: spec
spec_type: notification
spec_id: FEAT-18.SPEC-010
spec_name: New Contact Invitation Email
spec_slug: new-contact-invitation-email
parent_feature: FEAT-18
parent_feature_name: Client Contact Management & Roles
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Notification Spec: New Contact Invitation Email

## Overview

**Name:** New Contact Invitation Email
**ID:** FEAT-18.SPEC-010
**Type:** Notification
**Purpose:** Sends a newly added or invited contact their first magic-link sign-in invitation, so they can reach the client portal for the first time.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles

## Scope and Non-Goals

**In Scope:**
- The single email sent the moment a new Client Contact is created, whether by Nadia (any role) or by a Primary contact inviting a Reviewer colleague
- Welcoming content that differs slightly by who added the contact and which role they were assigned
- Carrying the contact's first single-use sign-in link
- Retry and expiry behavior tied to the underlying token's own time limit

**Non-Goals:**
- Any later sign-in link the same contact requests after this first one -- owned by FEAT-05.SPEC-008 (Magic Link Sign-In Email), the standing notification for every subsequent request; this spec covers only the first-invitation moment tied to contact creation
- Determining token issuance and validity mechanics themselves -- owned by FEAT-05 (Magic Link Issuance, FEAT-05.SPEC-004; Link Validity & Recognition Rules, FEAT-05.SPEC-006). This spec sends the token that capability issues for a newly created contact and flags, for cross-feature reconciliation, that a first-use token for a contact whose status is Invited (not yet Active) must be recognized as valid by FEAT-05.SPEC-006, since FEAT-05 sets a contact's status to Active only upon their first successful sign-in -- the very action this email exists to enable.
- Alerting Nadia that a Primary contact invited someone -- owned by FEAT-18.SPEC-011 (Primary-Invited-Colleague Alert), a separate notification to a separate recipient
- A standalone Integration spec for the underlying email-sending capability -- the transactional email capability this notification relies on is owned by FEAT-14 (External Touchpoints table) and recorded here only as a cross-feature touchpoint, consistent with how FEAT-05.SPEC-008 uses the same capability

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, on every successful contact creation (FEAT-18.SPEC-002 or FEAT-18.SPEC-004) | The recipient has never signed in and has no in-app surface to reach; email is the only channel that can deliver a first credential to someone the product has never seen sign in before |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| New contact saved by Nadia | FEAT-18.SPEC-002 (Add or Edit Client Contact) | Fires once, immediately after a create (not an edit) save completes | New contact's name, email, assigned role, invited_by (Nadia), a freshly issued first sign-in token, the owning freelancer's Branding Profile |
| New contact invited by a Primary contact | FEAT-18.SPEC-004 (Invite Reviewer Colleague) | Fires once, immediately after Owen's invite send completes | New contact's name, email, role (always Reviewer), invited_by (the inviting Primary contact's name), a freshly issued first sign-in token, the owning freelancer's Branding Profile |

## Audience and Preferences

**Recipients:** The single, newly created Client Contact (a future Owen or Priya, per the Access Matrix, depending on the role assigned). No other role ever receives this email -- it is addressed to the one person whose first access it enables.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|----------------------|
| None | N/A | N/A | N/A |

This email carries a new contact's first-ever sign-in credential; per XBR-30, transactional emails core to the record always send. There is no preference screen for a client contact to manage before they have even signed in once, consistent with the Access Matrix (client contacts have no Settings surface of their own).

**Quiet Hours:** N/A -- this email carries a time-limited credential the recipient needs to reach the portal at all; holding it for a quiet-hours window would shrink the usable portion of the token's own time limit, the same reasoning FEAT-05.SPEC-008 applies to every subsequent sign-in email.

## Content Definition

**Email (added directly by the freelancer):**
- **Subject:** You've been added to {freelancer_business_name}'s Clientroom portal
- **Body:**
  Hi {contact_name},

  {inviting_party_name} has added you as a {role_label} contact for {freelancer_business_name} on Clientroom. {role_description}

  Click below to sign in for the first time. This link is single-use and expires soon, so use it right away.
- **CTA (button):** Get started -- deep-links to FEAT-05.SPEC-002 (Link Verification Landing) with the issued token, which then routes to FEAT-05.SPEC-003 (Portal Home) on success

**Email (invited by a Primary contact, always Reviewer):**
- **Subject:** {inviting_party_name} invited you to {freelancer_business_name}'s Clientroom portal
- **Body:**
  Hi {contact_name},

  {inviting_party_name} has invited you as a Reviewer contact for {freelancer_business_name} on Clientroom. You'll be able to view and comment on shared work, but not accept proposals, approve milestones, or see invoices.

  Click below to sign in for the first time. This link is single-use and expires soon, so use it right away.
- **CTA (button):** Get started -- deep-links to FEAT-05.SPEC-002 (Link Verification Landing) with the issued token, which then routes to FEAT-05.SPEC-003 (Portal Home) on success

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|--------------------------|----------------|------------------------|
| {freelancer_business_name} | Freelancer Account -- business_name (or, before that is set, the freelancer's own name) | Nadia Ross Design | Renders as "the Clientroom portal" (subject becomes "You've been added to the Clientroom portal") -- required before the first invoice per the dependency map, but this email can fire earlier, so the fallback covers that window |
| {contact_name} | Client Contact -- name | Priya Shah | Greeting renders as "Hi there," |
| {inviting_party_name} | The acting Client Contact's own name (Owen, for an invite) or the Freelancer Account's name (Nadia, for a direct add) | Nadia Ross / Owen Carter | Never empty -- a contact is always created by exactly one identifiable acting party (FEAT-18.SPEC-005, invited_by is always set) |
| {role_label} | Client Contact -- role, rendered as "Primary" or "Reviewer" | Primary | Never empty -- role is a required field at creation (FEAT-18.SPEC-005) |
| {role_description} | Derived from role -- Primary renders "As a Primary contact, you can accept proposals, approve milestones, and pay invoices." Reviewer renders "As a Reviewer contact, you can view and comment on shared work." | -- | Never empty -- exactly one of the two fixed descriptions always applies |

## Delivery Rules

**Batching:** None -- each contact creation produces exactly one invitation email for exactly one new contact; if the same person is added to two different clients (or by two different freelancers), each creation is an independent event that produces its own separate email.
**Deduplication:** At most one invitation email per Client Contact record -- this notification fires exactly once, at creation, and a contact is never created twice for the same event (FEAT-18.SPEC-005's per-client uniqueness rule prevents a duplicate create for the same email at the same client).
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). The retry window is chosen to fit inside platform parameter: `magic-link-expiry-window`, so a delayed-but-successful delivery still carries a usable link, consistent with FEAT-05.SPEC-008's identical reasoning.
**Expiry:** If delivery has not succeeded after the final retry, no further attempt is made and the delivery failure is surfaced to the freelancer as a warning (XBR-30) on the client's contact record, since the new contact has no other way to be reached and would otherwise never learn they were given access at all.

## Edge Cases

- **The contact is removed before this email is delivered** -- The pending delivery is cancelled: FEAT-18.SPEC-009's immediate access revocation means the token this email would carry is no longer usable, so sending it afterward would only confuse the recipient with a dead link.
- **The same person is added as a contact to two different clients of the same freelancer at once** -- Each creation is a separate Client Contact record and produces its own separate invitation email, each with its own token scoped to that specific client relationship.
- **The inviting Primary contact's own name changes after sending the invite but before this email is delivered (unlikely, since Owen cannot edit his own record, but possible if Nadia edits it)** -- The email renders {inviting_party_name} using the value captured at trigger time, not a value re-read at delivery time, so the invite reads consistently with what was true when it was sent.
- **The token expires before the email is delivered (extreme delivery delay)** -- The delivered email's "Get started" link shows "This link isn't valid anymore" when clicked (FEAT-05.SPEC-002); this is treated as a delivery-timing rarity, since platform parameter: `magic-link-expiry-window` is set wide enough that a normal delivery, including one retry cycle, comfortably completes within it.
- **A recipient who is already a Client Contact for a different freelancer receives this email** -- No merge or cross-reference occurs; this email and its token concern only the newly created Client Contact record for this freelancer, exactly as FEAT-05.SPEC-008 treats a person who is a contact for more than one freelancer as entirely separate relationships.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-18.SPEC-002 (Add or Edit Client Contact) | Triggered by (inbound) | Every successful create save fires this notification |
| FEAT-18.SPEC-004 (Invite Reviewer Colleague) | Triggered by (inbound) | Every successful invite send fires this notification |
| FEAT-18.SPEC-011 (Primary-Invited-Colleague Alert) | References (inbound) | Fires alongside this notification when the source is FEAT-18.SPEC-004, alerting Nadia separately |
| FEAT-05.SPEC-004 (Magic Link Issuance) | References (inbound) | Supplies the first token this email carries (with the cross-feature status note above) |
| FEAT-05.SPEC-006 (Link Validity & Recognition Rules) | References (inbound) | Governs the token's expiry window this email's retry rule is bounded by |
| FEAT-05.SPEC-002 (Link Verification Landing) | Navigation (outbound) | The "Get started" CTA deep-links here with the issued token |
| FEAT-19 (Freelancer Branding) | References (inbound) | Supplies the logo and brand colour applied to this email |
| FEAT-33 (Portal Referral Attribution) | References (inbound) | Supplies the referral mark shown alongside branding |
| FEAT-14 (Notifications (Email)) | References (inbound) | Owns the underlying Transactional Email Delivery capability (FEAT-14.SPEC-001) this notification is sent through |

## Analytics and Success Signals

- **invitation_email_sent** (added_by: freelancer / primary_contact, role_assigned: primary / reviewer) -- N/A -- no success-metrics.md metric is connected to Client Contact Management & Roles; retained so first-invitation delivery volume is observable
- **invitation_email_delivery_failed** (retry_count_exhausted: true) -- N/A -- no success-metrics.md metric is connected to this feature; retained so a lost first invitation, which would otherwise leave a new contact silently unable to reach the portal, is surfaced rather than invisible

## Acceptance Criteria

**FEAT-18.SPEC-010-AC-01:** Given Nadia adds a new Primary contact directly, when the save completes, then that contact receives an email with subject "You've been added to {freelancer_business_name}'s Clientroom portal" describing their Primary entitlements and a "Get started" link.

**FEAT-18.SPEC-010-AC-02:** Given Owen invites a Reviewer colleague, when the invite send completes, then the new contact receives an email with subject "{Owen's name} invited you to {freelancer_business_name}'s Clientroom portal" describing Reviewer entitlements and a "Get started" link.

**FEAT-18.SPEC-010-AC-03:** Given the new contact taps "Get started" in the email, when the link opens, then they land on FEAT-05.SPEC-002 with their issued token.

**FEAT-18.SPEC-010-AC-04:** Given the freelancer's `business_name` is not yet set, when this email is composed, then the subject and body render with "the Clientroom portal" in place of the business name.

**FEAT-18.SPEC-010-AC-05:** Given a contact is removed before this email is delivered, when the delivery would otherwise occur, then it is cancelled and no email is sent.

**FEAT-18.SPEC-010-AC-06:** Given delivery of this email fails on the first attempt, when the delivery capability retries, then it retries up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before giving up.

**FEAT-18.SPEC-010-AC-07:** Given all retries for this email are exhausted, when the final attempt fails, then Nadia sees a delivery warning on the client's contact record.

**FEAT-18.SPEC-010-AC-08:** Given this notification has no preference control for the recipient, when a new contact is created, then the email always sends -- there is no opt-out surface to check.

**FEAT-18.SPEC-010-AC-09:** Given a new contact is created at any hour, when this notification is triggered, then it sends immediately with no quiet-hours hold, since the underlying credential is time-limited.

**FEAT-18.SPEC-010-AC-10:** Given the freelancer's Branding Profile has a logo and brand colour set, when this email renders, then it displays that branding alongside the "Made with Clientroom" referral mark.

**FEAT-18.SPEC-010-AC-11:** Given the same person is added as a contact to two different clients of the same freelancer, when both creations complete, then each produces its own separate email with its own token scoped to that specific client.

**FEAT-18.SPEC-010-AC-12:** Given the token this email carries expires before the email is delivered due to an extreme delivery delay, when the recipient clicks "Get started", then they see "This link isn't valid anymore" and can request a fresh one from FEAT-05.SPEC-002.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 2 (added by freelancer, invited by Primary contact) | 2 |
| Preference States | 1 (no preference -- always sends) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
