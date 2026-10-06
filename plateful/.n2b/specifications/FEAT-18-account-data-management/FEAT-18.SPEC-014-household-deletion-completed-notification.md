---
document_type: spec
spec_type: notification
spec_id: FEAT-18.SPEC-014
spec_name: Household Deletion Completed Notification
spec_slug: household-deletion-completed-notification
parent_feature: FEAT-18
parent_feature_name: Account & Data Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Notification Spec: Household Deletion Completed Notification

## Overview

**Name:** Household Deletion Completed Notification
**ID:** FEAT-18.SPEC-014
**Type:** Notification
**Purpose:** Tells the organiser once household deletion has fully completed, since the cascade can take up to 30 days and she is signed out well before it finishes.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- The confirmation delivered when household deletion processing fully completes, on its one channel (email, since the organiser is already signed out by then)
- Retry and expiry behavior for this notification

**Non-Goals:**
- Deciding when deletion is fully complete -- owned by FEAT-18.SPEC-008 (Household Deletion Processing); this spec begins where that automation's completed outcome fires
- The email channel's delivery mechanics -- owned by FEAT-18.SPEC-012 (Transactional Email, Account & Data), which this notification's email is sent through
- The immediate, in-app "Your household is being deleted..." message shown at confirmation time -- owned directly by FEAT-18.SPEC-003; this notification is the separate, later confirmation that the full cascade has finished

## Channels

| Channel | Used When | Rationale |
|---------|-----------|--------------|
| Email | Always, once deletion fully completes | Maya is signed out of the household well before the cascade finishes (deletion can take up to 30 days), so no in-app surface exists to reach her; email is the only channel that can still reach her at her own address after the household -- and her session in it -- no longer exists |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Household deletion completed | FEAT-18.SPEC-008 (Household Deletion Processing) | Fires when the full cascade finishes and the Household record is permanently removed | Organiser's email address and first name (captured at the start of deletion processing, before the Member Profile record itself is removed), deletion completion date |

## Audience and Preferences

**Recipients:** Maya -- the organiser at the time deletion was confirmed. She is the sole recipient, since she is the one who confirmed the deletion and the household no longer has any other addressable member by the time this notification fires.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|------------------------|
| N/A -- this notification has no on/off control | -- | Always on | -- |

No preference exists for this notification: it confirms an irreversible action Maya herself explicitly confirmed, and it is the sole surviving record that the deletion she asked for actually finished -- the product does not offer an opt-out for it.

**Quiet Hours:** N/A -- this notification fires once, whenever the multi-day cascade happens to finish; the product defines no quiet-hours window for a confirmation of an irreversible action the recipient explicitly confirmed, since delaying it would only prolong her uncertainty about whether the deletion actually completed.

## Content Definition

**Email:**
- **Subject:** Your Plateful household has been deleted
- **Body:**
  Hi {organiser_first_name},

  The household you asked us to delete has now been fully and permanently removed, along with every member profile, plan, list, rule, and rating it contained.

  This action cannot be undone, and no data from this household is retained in any form usable for another purpose.

  If you didn't request this, contact support at {support_contact_reference}.
- **CTA:** N/A -- no in-product destination exists to link to, since the household and every account tied to it have been removed; the email is the final, standalone record of completion.

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|-----------------------------|-------------------|----------------------------|
| {organiser_first_name} | Member Profile -- display_name (organiser, captured before removal) | Maya | Greeting renders as "Hi," |
| {support_contact_reference} | Derived -- the product's general support contact reference (not the in-product Contact Support screen, which requires a signed-in session Maya no longer has) | support@plateful.example | Never empty -- a general support contact reference is a fixed part of every account-related email's footer, independent of any specific household |

## Delivery Rules

**Batching:** No batching applies -- each household deletion produces exactly one completion notification, since only one deletion can be in progress per household at a time (FEAT-18.SPEC-008's Business Rules).
**Deduplication:** At most one deletion-completed notification per household deletion. The cascade completes exactly once; there is no retry-of-the-whole-cascade path that could re-trigger this notification.
**Retry on failure:** Email delivery failure is retried up to 3 times over 6 hours, per FEAT-18.SPEC-012's Degradation Behavior. After the final failure, no further channel exists to deliver this notification -- there is no in-app fallback, since the household and Maya's session in it no longer exist. The failure is logged internally so the product's own record-keeping reflects that the confirmation could not be confirmed as received, but no further user-facing action follows.
**Expiry:** This notification does not expire in the usual sense; because it has no fallback channel, delivery continues to be attempted for the full retry window (up to 6 hours) regardless of how long the underlying deletion took, since a late-but-eventually-delivered confirmation is still meaningfully better than none.

## Edge Cases

- **Maya's email address is captured before the Member Profile record is removed, but the send itself happens after removal** -- The email address and first name are captured once, at the start of deletion processing (per this spec's Trigger's Available Data), and carried through to send time independently of the Member Profile record's own lifecycle, so the notification can still be sent after the underlying record no longer exists.
- **The email capability is down for the entire 6-hour retry window** -- The notification is never delivered; this is logged internally, and no further fallback exists, since the household and every account tied to it are already gone.
- **Maya attempts to sign back in immediately after confirming deletion, before the cascade finishes** -- She cannot: her Member Profile was set to a removed state as part of the deletion's early steps (FEAT-18.SPEC-008), so no valid session exists for her to sign back into during or after the cascade; the completion email remains her only route to confirmation.
- **Maya never opens the completion email** -- No further reminder is sent; this notification, per its own Delivery Rules, fires exactly once and does not repeat, since a household that no longer exists has nothing further to report.
- **A support inquiry (via the general support contact reference) arrives asking whether a deletion completed, while the cascade is still within its 30-day window** -- This is outside this notification's own scope; it is answered through the product's general support process (outside this feature's in-product Contact Support screen, since Maya's household no longer exists to submit through it), not by re-sending this notification early.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-008 (Household Deletion Processing) | Triggered by (inbound) | The deletion-completed outcome fires this notification |
| FEAT-18.SPEC-012 (Transactional Email, Account & Data) | Triggers (outbound) | The email channel is sent through this integration |
| FEAT-18.SPEC-003 (Delete Household) | References (inbound) | The immediate in-app confirmation this notification follows, at completion rather than confirmation time |

## Analytics and Success Signals

- **household_deletion_completed_notification_delivered** (delivery outcome: delivered / failed) -- N/A -- no Stage 2 success metric measures this notification's delivery directly; retained per product-features.md's Signals field (household_deleted) as the sole confirmation record for a deletion, given it has no in-app fallback
- **household_deletion_completed_notification_failed** (retries_attempted) -- N/A -- no Stage 2 success metric measures this notification's failure rate; retained since a failure here means the household's final confirmation was never received at all

## Acceptance Criteria

**FEAT-18.SPEC-014-AC-01:** Given Maya's household deletion fully completes, when FEAT-18.SPEC-008 fires this notification, then she receives an email with the subject "Your Plateful household has been deleted".

**FEAT-18.SPEC-014-AC-02:** Given Maya receives the deletion-completed email, when she reads it, then it states the deletion cannot be undone and that no data is retained for another purpose.

**FEAT-18.SPEC-014-AC-03:** Given the deletion-completed email fails to send on its first attempt, when a retry remains within the 6-hour window, then it is retried automatically.

**FEAT-18.SPEC-014-AC-04:** Given the deletion-completed email's retries are exhausted, when this occurs, then no further delivery attempt or fallback channel is used, and the failure is logged internally.

**FEAT-18.SPEC-014-AC-05:** Given Maya has no preference control for this notification, when her household deletion completes, then the email is always sent, since no opt-out exists.

**FEAT-18.SPEC-014-AC-06:** Given Maya's Member Profile was already removed as part of the early steps of deletion processing, when this notification is sent later, then it still reaches her registered email address, since that address was captured at the start of processing.

**FEAT-18.SPEC-014-AC-07:** Given Maya attempts to sign back in after confirming deletion but before the cascade finishes, when she tries, then no valid session exists for her, and the completion email remains her only route to confirmation.

**FEAT-18.SPEC-014-AC-08:** Given Maya never opens the completion email, when time passes, then no further reminder or repeat of this notification is sent.

**FEAT-18.SPEC-014-AC-09:** Given a household's deletion completes, when this notification fires, then no in-app CTA is offered, since no in-product destination exists for a deleted household.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Channels | 1 (email) | 1 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no preference) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
