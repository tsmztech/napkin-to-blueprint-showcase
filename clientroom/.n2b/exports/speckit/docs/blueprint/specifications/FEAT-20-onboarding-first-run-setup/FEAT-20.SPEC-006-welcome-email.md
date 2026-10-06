---
document_type: spec
spec_type: notification
spec_id: FEAT-20.SPEC-006
spec_name: Welcome Email
spec_slug: welcome-email
parent_feature: FEAT-20
parent_feature_name: Onboarding / First-Run Setup
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Notification Spec: Welcome Email

## Overview

**Name:** Welcome Email
**ID:** FEAT-20.SPEC-006
**Type:** Notification
**Purpose:** Confirms to Nadia that her Freelancer Account was created, using the transactional email delivery capability, so she has a record of her sign-up separate from the in-product session.
**Parent Feature:** FEAT-20 -- Onboarding / First-Run Setup

## Scope and Non-Goals

**In Scope:**
- The single welcome email sent once, at account creation
- Its delivery, retry, and expiry behavior on its one channel (email)

**Non-Goals:**
- Deciding that a Freelancer Account should be created -- owned by FEAT-20.SPEC-001 (Sign-Up & Account Creation); this spec begins where that screen's creation trigger fires
- Any onboarding-progress content (which step Nadia is on, what remains) -- this email confirms account creation only; onboarding's own progress is shown live inside FEAT-20.SPEC-002, not repeated in email form
- An opt-out control for this email -- excluded per product-features.md's notification_preferences definition: "transactional record emails cannot be disabled" (XBR-30), and confirming account creation is exactly this kind of transactional record
- General product-education or onboarding-tips email sequences -- product-features.md defines no drip-email capability; this spec covers only the single confirmation named in the Brief's Communications field

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| Email | Always, once, when the Freelancer Account is created | Nadia's sign-in email is the one contact point established at account creation, and the product's baseline notification design uses email for every transactional record (ASMP-26); no in-app equivalent is needed since Nadia is already looking at the guided sequence in-product at the same moment |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Freelancer Account created | FEAT-20.SPEC-001 (Sign-Up & Account Creation) | Fires once, immediately after the Freelancer Account is created and transitioned to Active | Freelancer's name, sign-in email |

## Audience and Preferences

**Recipients:** Nadia (Freelancer) -- the sole recipient, sent to the sign-in email address she just registered. No other role in the Access Matrix has any relationship to this email; Owen, Priya, and Dana never receive it.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this is a transactional record email | N/A | Always sent | N/A -- per XBR-30, transactional emails core to the record always send and carry no opt-out; FEAT-21's notification-preference screen governs only optional emails, never this one |

**Quiet Hours:** N/A -- account creation is a one-time, freelancer-initiated moment (she is at that instant actively signing up), so there is no meaningful "wrong time" for this confirmation; the product defines no quiet-hours window for this email.

## Content Definition

**Email:**
- **Subject:** Welcome to Clientroom, {freelancer_name}
- **Body:**
  Hi {freelancer_name},

  Your Clientroom account is set up and ready. You're picking up right where you left off in your guided setup -- add your first client, draft your first proposal, and you're on your way to getting paid faster.

  If you didn't create this account, contact us right away.
- **CTA (button):** Continue setup -- deep-links to FEAT-20.SPEC-002 (Onboarding Guided Sequence), resuming on Nadia's current onboarding step

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {freelancer_name} | Freelancer Account -- name | Jordan Lee | Never empty -- name is required at account creation (FEAT-20.SPEC-001) |

## Delivery Rules

**Batching:** N/A -- this email is sent exactly once per account; there is never a second pending instance to batch with.
**Deduplication:** At most one welcome email per Freelancer Account. A retried or repeated account-creation attempt that fails before the account is actually created (per FEAT-20.SPEC-001's Edge Cases) never triggers a send, since the trigger fires only on a successful, committed account creation.
**Retry on failure:** Delivery failure is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window`, per the Transactional Email Delivery capability (FEAT-14.SPEC-001). After the final failure, the failure is surfaced to Nadia as a delivery warning per ASMP-26's delivery-visibility expectation. FEAT-14.SPEC-006 defines the warning on the affected project, but no project exists yet at this moment, so the warning is shown in the notice area of the onboarding shell (FEAT-20.SPEC-002), which defines its surface, exact wording ("We couldn't deliver your welcome email to {sign_in_email}. If that address is wrong, you can correct it in Settings."), "Go to Settings" button (to FEAT-21.SPEC-005) and "Dismiss" link. It appears the next time Nadia opens the shell while onboarding is In Progress or on the Ready state, and stops appearing once she dismisses it or changes her sign-in email.
**Expiry:** This email never expires in the sense of becoming irrelevant to skip -- confirming account creation stays true indefinitely. If all retries are exhausted, delivery is not reattempted later, and the surviving signal is that Nadia's account and onboarding progress remain fully visible and usable in-product regardless of whether this email ever arrived.

## Edge Cases

- **Nadia's sign-in email is mistyped and does not exist** -- Delivery bounces; per the Retry rule, retries are attempted up to the standard count and window, then the failure is surfaced as a delivery warning inside the guided sequence, since correcting the email address is itself a Settings action (FEAT-21) available to her once she is signed in.
- **The Freelancer Account is deleted (FEAT-24) before this email is delivered** -- Delivery is cancelled; a welcome confirmation for an account that no longer exists is never sent.
- **Preferences change between trigger and delivery** -- Not applicable: this email carries no preference to change (Audience and Preferences).
- **Quiet hours** -- Not applicable, per the Channels/Quiet Hours definition above.
- **Delivery succeeds but arrives after Nadia has already completed onboarding** -- The email's content and CTA remain valid regardless of timing; the CTA opens FEAT-20.SPEC-002, which lands her on the Ready state if onboarding is Complete and she has not yet tapped "Go to your dashboard", or redirects her to the dashboard (FEAT-12) if she has, rather than mid-sequence.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|--------------------|------------------|
| FEAT-20.SPEC-001 (Sign-Up & Account Creation) | Triggered by (inbound) | Account creation fires this email |
| FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Navigation (outbound) | The "Continue setup" CTA deep-links here, resuming Nadia's current step |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (outbound) | This email is sent and its delivery/bounce status tracked through the transactional email delivery capability |
| FEAT-14.SPEC-006 (Delivery Failure Warning to Freelancer) | References (outbound) | The project-level warning this account-level variant stands in for, because no project exists at sign-up |
| FEAT-20.SPEC-002 (Onboarding Guided Sequence) | Affects (outbound) | Hosts the welcome-email delivery warning (surface, wording, dismissal) |
| FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) | Navigation (outbound, via the warning) | The warning's "Go to Settings" button leads here to correct a mistyped address |

## Analytics and Success Signals

- **welcome_email_delivered** (delivery outcome: delivered / bounced / failed) -- supports success-metrics.md: "First-Session Activation" (a delivered welcome confirmation is part of the trusted first-session experience the metric measures)
- **welcome_email_delivery_warning_shown** (retries exhausted) -- N/A -- no Stage 2 metric measures individual delivery-warning views; retained per ASMP-26's delivery-visibility expectation so a lost first email is observable rather than invisible

## Acceptance Criteria

**FEAT-20.SPEC-006-AC-01:** Given Nadia successfully creates her Freelancer Account, when the account is created, then the welcome email is sent once to her sign-in email address with the subject "Welcome to Clientroom, {freelancer_name}."

**FEAT-20.SPEC-006-AC-02:** Given Nadia receives the welcome email, when she taps "Continue setup," then she lands on FEAT-20.SPEC-002 (Onboarding Guided Sequence) at her current onboarding step.

**FEAT-20.SPEC-006-AC-03:** Given Nadia's sign-in email address does not exist, when delivery is attempted, then it is retried up to platform parameter: `transactional-email-retry-count` times over platform parameter: `transactional-email-retry-window` before a delivery warning appears to her inside the guided sequence.

**FEAT-20.SPEC-006-AC-04:** Given Nadia's Freelancer Account is deleted before this email is delivered, when the deletion completes, then delivery of this email is cancelled.

**FEAT-20.SPEC-006-AC-05:** Given Nadia already completed onboarding by the time this email is delivered, when she taps "Continue setup," then she lands on the Ready state if she has not yet tapped "Go to your dashboard", or is redirected to her dashboard (FEAT-12) if she has, rather than mid-sequence, per FEAT-20.SPEC-002.

**FEAT-20.SPEC-006-AC-06:** Given this feature defines no preference control for this email, when Nadia looks in her notification preferences (FEAT-21), then no toggle for this email exists there -- it always sends.

**FEAT-20.SPEC-006-AC-07:** Given Nadia's account creation attempt fails validation and no account is created, then no welcome email is sent, since the trigger fires only on a successful, committed account creation.

**FEAT-20.SPEC-006-AC-08:** Given delivery succeeds on the first attempt, when it is delivered, then `welcome_email_delivered` is emitted with outcome "delivered."

**FEAT-20.SPEC-006-AC-09:** Given all retries are exhausted with no successful delivery, when the final retry fails, then the next time Nadia opens FEAT-20.SPEC-002 the warning is shown in its notice area with "Go to Settings" and "Dismiss", and `welcome_email_delivery_warning_shown` is emitted at that display.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|----------------|-------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always sent, no preference) | 1 |
| Delivery Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
