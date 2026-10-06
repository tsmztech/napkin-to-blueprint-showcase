---
document_type: spec
spec_type: notification
spec_id: FEAT-14.SPEC-006
spec_name: Delivery Failure Warning to Freelancer
spec_slug: delivery-failure-warning-to-freelancer
parent_feature: FEAT-14
parent_feature_name: Notifications (Email)
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Notification Spec: Delivery Failure Warning to Freelancer

## Overview

**Name:** Delivery Failure Warning to Freelancer
**ID:** FEAT-14.SPEC-006
**Type:** Notification
**Purpose:** Warns Nadia on the affected project when a notification's delivery fails after retries are exhausted (or bounces permanently), so a lost email is surfaced rather than silently vanishing; viewable read-only by Dana inside a support session.
**Parent Feature:** FEAT-14 -- Notifications (Email)

## Scope and Non-Goals

**In Scope:**
- The warning shown to Nadia when a Notification's delivery fails permanently (bounce) or its retries are exhausted (FEAT-14.SPEC-003)
- The read-only variant Dana sees inside a logged support session (FEAT-31)
- Batching, deduplication, and persistence (expiry) behavior for this warning

**Non-Goals:**
- Deciding when a notification has permanently failed or exhausted its retries -- owned by FEAT-14.SPEC-003; this spec begins where that spec's failure outcome hands off.
- Correcting the underlying cause (e.g., a mistyped contact email address) -- owned by FEAT-18 (Client Contact Management & Roles); this spec's only involvement is deep-linking Nadia there.
- Automatically re-sending the originally failed notification once the underlying issue is fixed -- outside this feature's dispatch/tracking scope; a fresh send happens only if the originating feature offers its own resend action (e.g., FEAT-02.SPEC-007, Proposal Resend), which this spec does not itself trigger.
- A full in-app notification feed or history of past warnings -- excluded per scope-boundaries.md and the Brief's own States field ("no primary browsing view of its own"); In-App Notification Center (FEAT-29) is explicitly deferred to Later.

## Channels

| Channel | Used When | Rationale |
|---------|-----------|-----------|
| In-app (a persistent element on the affected project's own screen, owned by FEAT-01) | Always, whenever a Notification tied to that project has a Bounced or retries-exhausted Failed outcome | This is a persistent state indicator on a screen Nadia already owns and visits, not a discrete push-style interruption or a notification inbox -- it is categorically distinct from the in-app notification *feed* concept the Brief's Non-Goals excludes (FEAT-29, deferred to Later); "surfaced... within minutes" (ASMP-26) means the warning becomes available promptly, not that it is actively pushed to her outside the product she already uses to manage the affected project. |

Email is deliberately not used for this warning: the failure being reported is itself about an email that could not be delivered, and Nadia's own inbox is not where the product routes account-operational state (feature-overview.md, States: "no primary browsing view of its own"); the project screen she already checks for status is the surviving signal, consistent with FEAT-11.SPEC-004 and other sibling specs treating the delivery warning as project-level state, not a separate message.

## Trigger

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Notification permanently bounced | FEAT-14.SPEC-003 (Delivery Status Tracking & Retry) | Fires immediately when a send is reported Bounced -- no retry is attempted for a permanent failure | Notification reference, notification_type, recipient, affected project reference |
| Notification retries exhausted | FEAT-14.SPEC-003 (Delivery Status Tracking & Retry) | Fires when a transient Failed outcome recurs until the retry count reaches platform parameter: `transactional-email-retry-count` with no Delivered outcome | Notification reference, notification_type, recipient, affected project reference |

## Audience and Preferences

**Recipients:** Nadia (Freelancer) -- Full access, per the Access Matrix's Notifications & Help row; she is the sole active recipient of this warning, since it exists to let her act on a communication failure on her own account. Dana (Support Operator) has "View (delivery warnings only)" per the Access Matrix -- she sees the same warning read-only, and only inside a logged support session (FEAT-31), never the underlying message's subject or body content. Owen and Priya never see this warning; it concerns the freelancer's operational awareness of her own delivery problem, not client-facing content.

**Preference Controls:**

| Preference | Options | Default | Where Set (Spec ID) |
|------------|---------|---------|---------------------|
| None -- this is a transactional warning | -- | Always on | -- |

This warning cannot be turned off: per XBR-30, "delivery failures are surfaced to the freelancer as warnings on the affected project," and a preference to silence the mechanism that reports a lost communication would defeat the purpose it exists for.

**Quiet Hours:** N/A -- this warning is a persistent state indicator rendered when Nadia opens the affected project's screen, not a discrete, time-stamped interruption; there is no moment of delivery for quiet hours to govern, and the product defines no quiet-hours window for this feature.

## Content Definition

**In-app (single failure):**
- **Title:** Delivery issue on this project
- **Body:** The {notification_type_label} email to {recipient_name} could not be delivered{failure_reason_clause}.
- **CTA:** Review contact details -- deep-links to FEAT-18 (Client Contact Management & Roles), the client contact list for the affected client, so Nadia can correct a mistyped or outdated address

**In-app (batched -- 2+ open failures on the same project, see Delivery Rules):**
- **Title:** {count} delivery issues on this project
- **Body:** A list with one line per failure: {notification_type_label} to {recipient_name}{failure_reason_clause}
- **CTA:** Review contact details -- deep-links to FEAT-18, the client contact list for the affected client

**In-app (Dana's read-only support-session variant):**
- **Title:** Delivery issue on this project (support view)
- **Body:** The {notification_type_label} email to {recipient_name} could not be delivered{failure_reason_clause}. This summary is read-only; the message's own content is not shown.
- **CTA:** None -- Dana's session is read-only (Access Matrix: Support Access "Full (opens read-only sessions, always logged)," Notifications & Help "View (delivery warnings only)"); no action control is offered to her

**Placeholders:**

| Placeholder | Source (Entity / Field) | Example Value | Empty-Value Fallback |
|-------------|------------------------|---------------|----------------------|
| {notification_type_label} | Notification -- notification_type, rendered in plain language (per the registry in FEAT-14.SPEC-004) | "invoice" (for invoice_issued) | Never empty -- notification_type is required on every Notification record (FEAT-14.SPEC-004) |
| {recipient_name} | Client Contact -- name (or Freelancer Account -- name, for a freelancer-addressed notification that itself failed) | Owen Marsh | Never empty -- name is required at Client Contact creation (FEAT-18) and Freelancer Account creation (FEAT-20) |
| {failure_reason_clause} | Derived from the Notification's bounce/failure reason category (FEAT-14.SPEC-001), when the capability provided one | " (the address could not be found)" | Renders as an empty string, so the sentence reads "...could not be delivered." with no invented reason, per FEAT-14.SPEC-001's edge case for a reason-less bounce |
| {count} | Derived -- number of open (unresolved) delivery-failure warnings currently on this project | 3 | Never empty -- the batched variant only renders with 2 or more open failures |

## Delivery Rules

**Batching:** All open delivery-failure warnings for the same project are shown as one persistent indicator, using the batched variant when 2 or more failures are currently open (unresolved) on that project; a new failure on a project with no existing open warning renders the single-failure variant.
**Deduplication:** At most one open warning entry exists per failed Notification; FEAT-14.SPEC-003 never re-fires this notification a second time for the same Notification once its warning entry exists, even if a duplicate outcome report arrives.
**Retry on failure:** N/A -- this is a persistent in-app state, not a transmitted message; there is no send of this warning itself to fail or retry. It either renders correctly when Nadia (or Dana, in a support session) opens the affected project's screen, or the screen's own general error handling applies (owned by FEAT-01), unrelated to this spec's delivery rules.
**Expiry:** This warning does not expire on a timer -- a lost communication remains a real, unresolved problem until addressed. It persists until the underlying Notification's condition changes (e.g., the originating feature's own resend succeeds and produces a new, separately-tracked Notification that later reports Delivered) or Nadia's corrective action otherwise resolves it; there is no automatic dismissal.

## Edge Cases

- **The underlying record is voided or superseded before Nadia acts on the warning (e.g., a proposal is edited and re-sent, per FEAT-02.SPEC-011)** -- The original warning remains as historical context (the original send genuinely did fail), but is not treated as still-actionable once the superseding event's own notification completes successfully; both her project screen and this spec's content reflect the current, superseding notification's outcome once known.
- **Multiple delivery failures accumulate on the same project across different notification types (e.g., a bounced invitation and a bounced invoice email)** -- They batch into the single project-level indicator (Delivery Rules above); each failure remains individually visible within the expanded view, and Nadia can address each independently.
- **Nadia corrects the contact's email address via FEAT-18, but the originally failed notification is never automatically retried** -- The warning remains open until she takes the originating feature's own action to resend (where one exists, e.g., FEAT-02.SPEC-007); this spec does not itself re-trigger the business event, since doing so is outside its dispatch and tracking scope.
- **The affected project is archived while a warning is open** -- The warning still renders on the archived project's own view when Nadia or Dana opens it, consistent with FEAT-14.SPEC-003's own handling: archiving removes a project from active view without erasing its record.
- **The freelancer's account is deleted (FEAT-24) while a warning is open** -- The warning is removed along with the Notification record it describes, as part of account deletion; no orphaned warning persists.
- **Dana opens a support session on a project with an open warning** -- She sees the read-only support-session variant with no action control; she cannot dismiss, resolve, or act on the warning in any way, consistent with her View-only, read-only support access (Access Matrix, FEAT-31).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-14.SPEC-003 (Delivery Status Tracking & Retry) | Triggered by (inbound) | A Bounced outcome or exhausted retries fires this notification |
| FEAT-14.SPEC-001 (Transactional Email Delivery) | References (inbound) | Supplies the bounce/failure reason category, when available, rendered in {failure_reason_clause} |
| FEAT-14.SPEC-004 (Notification Type & Recipient Entitlement Rules) | References (inbound) | Supplies the notification_type registry this spec's {notification_type_label} placeholder renders in plain language |
| FEAT-01 (Client & Project Management) | References (inbound) | Owns the affected project's own screen, on which this warning is a small persistent element |
| FEAT-18 (Client Contact Management & Roles) | Navigation (outbound) | The CTA deep-links here so Nadia can correct a mistyped or outdated contact email address |
| FEAT-31 (Operator Support Access) | References (inbound) | Governs Dana's read-only, logged access to this warning's support-session variant |

## Analytics and Success Signals

- **delivery_warning_created** (notification_type, reason: bounced / retries_exhausted) -- supports success-metrics.md: "Notification Delivery Reliability"
- **delivery_warning_viewed** (viewer_role: freelancer / support_operator) -- supports success-metrics.md: "Notification Delivery Reliability"
- **delivery_warning_cta_tapped** (destination: FEAT-18 client contact list) -- supports success-metrics.md: "Notification Delivery Reliability"

## Acceptance Criteria

**FEAT-14.SPEC-006-AC-01:** Given an invoice_issued notification to Owen bounces permanently, when FEAT-14.SPEC-003 reports the Bounced outcome, then this warning appears on the affected project for Nadia immediately, without waiting for any retry.

**FEAT-14.SPEC-006-AC-02:** Given a proposal_sent notification to Owen fails transiently and exhausts its retries, when the final retry's outcome is processed, then this warning appears on the affected project for Nadia.

**FEAT-14.SPEC-006-AC-03:** Given Nadia opens the affected project's screen, when one open delivery failure exists, then she sees the single-failure variant naming the notification type, the recipient, and, if available, the reason.

**FEAT-14.SPEC-006-AC-04:** Given a project has two open delivery failures of different notification types, when Nadia opens the project's screen, then she sees the batched variant listing both.

**FEAT-14.SPEC-006-AC-05:** Given Nadia taps "Review contact details" on this warning, then she is taken to FEAT-18's client contact list for the affected client.

**FEAT-14.SPEC-006-AC-06:** Given Nadia has no preference control for this warning, when a delivery failure occurs, then the warning always appears -- there is no way to switch it off, per XBR-30.

**FEAT-14.SPEC-006-AC-07:** Given a bounce report carries no specific reason from the delivery capability, when this warning renders, then it states delivery failed without inventing a reason.

**FEAT-14.SPEC-006-AC-08:** Given the same Notification's exhausted-retries outcome is reported twice due to a duplicate event, when this notification is triggered, then only one open warning entry exists for that Notification, not two.

**FEAT-14.SPEC-006-AC-09:** Given Nadia corrects the bouncing contact's email address via FEAT-18, when she returns to the affected project, then the original warning remains open until she separately resends the failed communication through its originating feature's own resend action.

**FEAT-14.SPEC-006-AC-10:** Given the affected project has since been archived, when Nadia opens the archived project, then the open delivery warning still appears on its screen.

**FEAT-14.SPEC-006-AC-11:** Given Nadia's account is deleted, when the deletion completes, then any open delivery warnings tied to her account are removed along with the Notification records they describe.

**FEAT-14.SPEC-006-AC-12:** Given Dana opens a support session on an account with an open delivery warning, when she views the affected project, then she sees the read-only support-session variant with no action control, and never the underlying message's subject or body content.

**FEAT-14.SPEC-006-AC-13:** Given Owen or Priya are viewing their own portal, when a delivery failure occurs on their project, then they never see this warning in any form -- it exists only for Nadia (and Dana, read-only).

**FEAT-14.SPEC-006-AC-14:** Given a proposal is edited and re-sent after its original send bounced (FEAT-02.SPEC-011), when the edited version's own notification succeeds, then the original warning remains as historical context while the current state reflects the successful re-send.

**FEAT-14.SPEC-006-AC-15:** Given this warning is an in-app persistent element rather than a transmitted message, when the affected project's screen loads normally, then the warning renders correctly with no separate delivery/retry mechanics of its own.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Channels | 1 (in-app) | 1 |
| Trigger Paths | 2 (bounced, retries exhausted) | 2 |
| Preference States | 1 (always on) | 1 |
| Delivery Rules | 4 (batching, deduplication, retry N/A, expiry) | 4 |
| Edge Cases | 6 | 6 |
