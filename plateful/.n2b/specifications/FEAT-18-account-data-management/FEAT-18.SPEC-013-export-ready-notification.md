---
document_type: spec
spec_type: notification
spec_id: FEAT-18.SPEC-013
spec_name: Export Ready Notification
spec_slug: export-ready-notification
parent_feature: FEAT-18
parent_feature_name: Account & Data Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Notification Spec: Export Ready Notification

## Overview

**Name:** Export Ready Notification
**ID:** FEAT-18.SPEC-013
**Type:** Notification
**Purpose:** Tells the organiser once a requested export is ready to download, since compilation is asynchronous and she may not be watching the screen while it runs.
**Parent Feature:** FEAT-18 -- Account & Data Management

## Scope and Non-Goals

**In Scope:**
- The confirmation delivered when an export completes, on both its channels (in-app and email)
- Preference, retry, and expiry behavior for this notification

**Non-Goals:**
- Deciding when an export is actually ready -- owned by FEAT-18.SPEC-006 (Export Generation Processing); this spec begins where that automation's success outcome fires
- The email channel's delivery mechanics -- owned by FEAT-18.SPEC-012 (Transactional Email, Account & Data), which this notification's email variant is sent through
- Reminding the organiser again if she never downloads the ready export -- excluded per scope-boundaries.md SC-14's spirit and this feature's general tone: the export remains available on FEAT-18.SPEC-001 indefinitely as a previous export, so no repeated nagging is needed

## Channels

| Channel | Used When | Rationale |
|---------|-----------|--------------|
| In-app | Always when the export becomes ready | Maya may have the app open or return to it soon after requesting the export; the in-app channel surfaces readiness the next time she is in the product |
| Email | Always when the export becomes ready | Export generation can take a noticeable time; Maya's day includes long stretches away from the product, and email catches the readiness moment even if she has closed the app |

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Export ready | FEAT-18.SPEC-006 (Export Generation Processing) | Fires when compilation completes successfully | Household reference, organiser's Member Profile reference, export ready date |

## Audience and Preferences

**Recipients:** Maya -- the organiser, per the Access Matrix's Account & Data column (Full for Maya, no access for any other role). Only the household member who requested the export receives this notification, since export is an organiser-only action.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|------------------------|
| N/A -- this notification has no on/off control | -- | Always on | -- |

No preference exists for this notification: it confirms an action Maya herself just took, not a proactive product-initiated message, so the product does not offer an opt-out (consistent with the account/recovery and billing confirmation emails' own no-preference treatment elsewhere in the product).

**Quiet Hours:** N/A -- this notification fires only once, in direct response to Maya's own recent request, at whatever time compilation happens to finish; the product defines no quiet-hours window for a confirmation of an action the recipient initiated herself.

## Content Definition

**In-app:**
- **Title:** Your export is ready
- **Body:** Your household data export finished compiling on {ready_date}. Download it from Account & Data.
- **CTA:** Download export -- deep-links to FEAT-18.SPEC-001 (Export Household Data)

**Email:**
- **Subject:** Your Plateful data export is ready
- **Body:**
  Hi {organiser_first_name},

  The household data export you requested is ready to download.

  Open Plateful and go to Account & Data to download your export.
- **CTA (button):** Download your export -- deep-links to FEAT-18.SPEC-001 (Export Household Data)

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|-----------------------------|-------------------|----------------------------|
| {ready_date} | Derived -- the completed export's ready date | September 27, 2026 | Never empty -- the export ready date is always set the moment compilation completes, before this notification fires |
| {organiser_first_name} | Member Profile -- display_name (organiser) | Maya | Greeting renders as "Hi," |

## Delivery Rules

**Batching:** No batching applies -- each export request produces its own independent notification, since only one export can be compiling per household at a time (FEAT-18.SPEC-010's rate limit and FEAT-18.SPEC-001's disabled-while-compiling state).
**Deduplication:** At most one export-ready notification per completed export. A retried compilation (FEAT-18.SPEC-006's automatic retry) never fires this notification until the export truly succeeds; retries themselves produce no notification.
**Retry on failure:** Email delivery failure is retried up to 3 times over 6 hours, per FEAT-18.SPEC-012's Degradation Behavior. After the final failure, the in-app notification stands as the delivery of record and no error is shown to Maya -- a confirmation of her own successful action must never surface an alarming failure message of its own. In-app delivery has no retry: it is delivered when Maya next opens the product.
**Expiry:** This notification does not expire in the usual sense -- the export it confirms remains available indefinitely as a previous export on FEAT-18.SPEC-001, so there is no "too late to matter" cutoff; an email delayed by retries is still useful whenever it arrives, since the export itself has not gone anywhere.

## Edge Cases

- **Household deleted between export completion and notification delivery** -- Cannot occur: household deletion supersedes and cancels any in-flight export (FEAT-18.SPEC-006's Edge Cases), so an export never completes for a household that has since been deleted.
- **Maya is already viewing FEAT-18.SPEC-001 when the export becomes ready** -- The in-app notification still fires (delivered to the product's standard notification surface), while the screen itself also transitions to its Ready state simultaneously; the two are complementary, not duplicative, since one is a persistent notification and the other is the screen's own live state.
- **The email is still queued (capability was down) when Maya downloads the export in-app first** -- The queued email is still delivered once the capability recovers; downloading in-app does not cancel or suppress the email, since the email's purpose (reaching her even if she is away from the product) is independent of whether she has already acted on the in-app notification.
- **A second export completes while the first export's notification has not yet been opened** -- Each export's notification is independent and both remain available; there is no batching or replacement, since each represents a genuinely separate completed export.
- **Maya has no registered email address (a defensive, unusual case)** -- Cannot occur in practice: every adult member's sign_in requires an email at account creation (FEAT-01); if this were ever true, the email channel would be skipped silently and the in-app notification alone would stand as the delivery of record.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|------------------|--------------|
| FEAT-18.SPEC-006 (Export Generation Processing) | Triggered by (inbound) | The export-ready outcome fires this notification |
| FEAT-18.SPEC-012 (Transactional Email, Account & Data) | Triggers (outbound) | The email channel is sent through this integration |
| FEAT-18.SPEC-001 (Export Household Data) | Navigation (outbound) | Both channels' CTA deep-links here |

## Analytics and Success Signals

- **export_ready_notification_delivered** (channel: in_app / email) -- N/A -- no Stage 2 success metric measures export-notification delivery directly; retained per product-features.md's Signals field (data_export_completed) as the operational confirmation this milestone reaches Maya
- **export_ready_notification_cta_tapped** (channel) -- N/A -- no Stage 2 success metric measures export download follow-through from this notification; retained to observe how often the notification actually drives a download

## Acceptance Criteria

**FEAT-18.SPEC-013-AC-01:** Given Maya's export completes, when FEAT-18.SPEC-006 fires this notification, then she receives an in-app notification titled "Your export is ready" and an email with the subject "Your Plateful data export is ready".

**FEAT-18.SPEC-013-AC-02:** Given Maya receives the in-app notification, when she taps "Download export", then she lands on FEAT-18.SPEC-001 with her export ready to download.

**FEAT-18.SPEC-013-AC-03:** Given Maya receives the export-ready email, when she taps "Download your export", then she lands on FEAT-18.SPEC-001.

**FEAT-18.SPEC-013-AC-04:** Given the export-ready email fails to send on its first attempt, when a retry remains within the 6-hour window, then it is retried automatically without any error shown to Maya.

**FEAT-18.SPEC-013-AC-05:** Given the export-ready email's retries are exhausted, when this occurs, then no failure message is shown to Maya and the in-app notification stands as the delivery of record.

**FEAT-18.SPEC-013-AC-06:** Given Maya has no preference control for this notification, when an export becomes ready, then both channels are always used, since no opt-out exists.

**FEAT-18.SPEC-013-AC-07:** Given Maya is actively viewing FEAT-18.SPEC-001 when her export becomes ready, when the export completes, then the screen transitions to its Ready state and the in-app notification is also delivered.

**FEAT-18.SPEC-013-AC-08:** Given Maya downloads her export in-app before the queued email arrives, when the email capability later recovers, then the queued email is still delivered.

**FEAT-18.SPEC-013-AC-09:** Given Maya requests and completes two separate exports in succession, when both complete, then she receives two independent notifications, one per export.

**FEAT-18.SPEC-013-AC-10:** Given FEAT-18.SPEC-006 retries a failed compilation before eventually succeeding, when the retries occur, then no export-ready notification fires until the export truly succeeds.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Channels | 2 (in-app, email) | 2 |
| Trigger Paths | 1 | 1 |
| Preference States | 1 (always on -- no preference) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry, expiry) | 4 |
| Edge Cases | 5 | 5 |
