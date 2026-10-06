---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-29.SPEC-002
spec_name: Feed Composition & Retention Rule
spec_slug: feed-composition-retention-rule
parent_feature: FEAT-29
parent_feature_name: In-App Notification Center
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 27
acceptance_criteria_count: 20
---

# Logic/Rule Spec: Feed Composition & Retention Rule

## Overview

**Name:** Feed Composition & Retention Rule
**ID:** FEAT-29.SPEC-002
**Type:** Logic/Rule
**Purpose:** Defines which events from Notification and Activity Log Entry populate the feed, their merged order, the rolling recent-window limit, and each item's default read state.
**Parent Feature:** FEAT-29 -- In-App Notification Center
**Governed Entity:** Feed Item Read State

## Scope and Non-Goals

**In Scope:**
- Which underlying Notification and Activity Log Entry records are surfaced as feed items, and for how long
- The merged, newest-first ordering across both source entities
- Field rules and default values for the Feed Item Read State record each feed item carries
- Authorization for every action on Feed Item Read State, per role

**Non-Goals:**
- The read/unread toggle's runtime behavior (what happens when Nadia taps open or the toggle) -- owned by FEAT-29.SPEC-003 (Mark Item Read/Unread); this spec defines the field rules that automation must follow, not the trigger-and-response flow itself.
- The screen's display treatment of each state (empty, loading, error, offline) -- owned by FEAT-29.SPEC-001 (Notification Center Feed); this spec defines what populates the feed, not how it is presented.
- The fallback and offline-continuity mechanics used when a refresh fails or connectivity drops -- owned by FEAT-29.SPEC-004 (Feed Load Fallback & Offline Continuity), which only reuses this spec's inclusion/window/order rule for the scope of its saved copy of the feed.
- Any write to the underlying Notification or Activity Log Entry -- excluded per XBR-04 and ASMP-15 (both are evidentiary and immutable); this spec only reads them and derives display order and window membership.

## Governed Entity

**Entity:** Feed Item Read State
**Source:** Feature Breakdown Brief, Entity-Lifecycle Coverage Matrix (feature-local display state layered on top of the read-only Notification and Activity Log Entry entities)

| Field | Data Type | Description |
|-------|-----------|-------------|
| reference | text (reference) | Points to exactly one underlying Notification or Activity Log Entry record that this feed item represents |
| read_status | enum (Read \| Unread) | Whether Nadia has read/opened this feed item |
| read_at | date (nullable) | The moment read_status last became Read; null while Unread |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-29.SPEC-001 | Notification Center Feed | On every screen load -- composition, ordering, and window rules applied to decide what is shown |
| FEAT-29.SPEC-003 | Mark Item Read/Unread | On open or explicit toggle -- read_status/read_at field rules applied when the record is written |
| FEAT-29.SPEC-004 | Feed Load Fallback & Offline Continuity | On refresh failure or offline/degraded -- the same inclusion/window/order rule scopes the saved fallback copy |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| reference | Required; must resolve to exactly one existing Notification or Activity Log Entry record (never both, never neither) | Always | On creation (implicit, the first time the underlying event enters the rolling window) | N/A -- not user-entered; an unresolvable reference excludes the candidate item from the feed rather than surfacing an error (see Edge Cases) | No (exclusion, not a blocking user-facing error) |
| read_status | Must be exactly one of Read or Unread | Always | On creation and on every update (FEAT-29.SPEC-003) | N/A -- system-set value; no spec offers any other value | No |
| read_at | Must be set (non-null) when read_status is Read, and null when read_status is Unread | Conditional on read_status | On every update that changes read_status (FEAT-29.SPEC-003) | N/A -- system-set value, never directly editable | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| read_status / read_at consistency | read_status, read_at | read_at is non-null if and only if read_status is Read | N/A -- enforced automatically by FEAT-29.SPEC-003, which always writes both fields together in the same operation; no user-facing path can produce an inconsistent pair |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|----------------------------------------------|
| View feed item read state | Nadia | Always -- her own account-wide feed | -- |
| View feed item read state | Owen | Never | Not shown -- no in-app feed exists anywhere in Owen's portal (product-features.md, Access); a direct link is handled as an out-of-scope link (FEAT-05.SPEC-006) |
| View feed item read state | Priya | Never | Same as Owen -- no in-app feed exists anywhere in her portal |
| View feed item read state | Dana | Never | Not shown -- excluded from operator support access entirely; Dana's Access Matrix "View (delivery warnings only)" cell applies to FEAT-14's delivery warnings, not to this entity |
| Create feed item read state (implicit) | Nadia | Always -- system-derived the first time an underlying event enters the rolling window; not a direct user action | -- |
| Create feed item read state (implicit) | Owen, Priya, Dana | Never | No feed item read state is ever created for any role but Nadia -- the underlying Notification and Activity Log Entry read by this rule are Nadia's own account data, isolated per ASMP-23 |
| Update read status (mark read/unread) | Nadia | Always, on any of her own in-window feed items | -- |
| Update read status (mark read/unread) | Owen, Priya, Dana | Never | No control to change a feed item's read state exists anywhere in the product for these roles |
| Delete feed item read state | Nadia | Never -- per product-features.md, Validation & Limits: "items can be marked read/unread but not deleted" | No delete control exists anywhere in the product for this record |
| Delete feed item read state | Owen, Priya, Dana | Never | Same -- no delete capability exists for any role |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| read_status | Defaults to Unread | On create -- the first time the underlying event enters the rolling window | No -- initial value is always Unread; only a later FEAT-29.SPEC-003 run changes it |
| read_at | Null by default | On create | No -- always system-set (see field rule above) |
| reference | Set once, to the underlying Notification or Activity Log Entry that entered the window | On create only | No -- immutable after creation |
| Feed membership (derived; not a stored field of Feed Item Read State) | An underlying Notification or Activity Log Entry is a feed member when it passes its inclusion rule (Notification: recipient is Nadia and notification_type is a qualifying type; Activity Log Entry: event_type is a qualifying type -- see Business Rules) AND is "in window", meaning its sent_at (Notification) or occurred_at (Activity Log Entry) falls within platform parameter: `notification-feed-retention-window` of the current time | Recomputed on every feed load | No -- not a user override |

## Business Rules

- Feed composition: on every load, the feed merges the qualifying Notification records (owned by FEAT-14) and qualifying Activity Log Entry records (owned by FEAT-13) -- as defined by the three inclusion rules below -- that fall within platform parameter: `notification-feed-retention-window`, into a single list ordered newest-first by each record's own sent_at or occurred_at.
- Notification inclusion rule: a Notification record enters the feed only if (1) its recipient is Nadia -- every Notification whose recipient is Owen, Priya, or any other client contact is excluded, even when it originates from the same triggering event as Nadia's copy -- and (2) its notification_type is one of: proposal_accepted_confirmation, proposal_change_requested, milestone_approved_confirmation (source icon: approval); invoice_issued, payment_confirmation, chargeback_notice (source icon: payment); client_comment_alert (source icon: comment). Every other notification_type is excluded (for example welcome_email, magic_link_sign_in, support_session_notice, delivery_failure_warning, and any type addressed to a client contact), and so is any notification_type added to the FEAT-14 registry later until this rule lists it. delivery_status does not affect inclusion -- a Notification whose delivery Failed or Bounced still appears, because the underlying event happened.
- Activity Log Entry inclusion rule: an Activity Log Entry enters the feed only if its event_type is one of: proposal accepted, milestone approved, milestone reopened (source icon: approval); invoice sent, credit note issued, refund/reversal/cancellation recorded, manual payment recorded (source icon: payment). Entries with any other event_type -- deliverable uploaded, deliverable removed, first client view, reminder sent, contact role changed (added, changed, or removed), support session opened, support session closed -- are excluded and remain visible only in the Activity & Audit Trail (FEAT-13). Inclusion does not depend on the entry's actor. No Activity Log Entry event_type maps to the comment icon; comment items come only from client_comment_alert Notifications.
- Icon and summary mapping: each included record shows the source icon named in the two inclusion rules above and a one-line summary in the form "{event label} -- {project name}", where {project name} is the name of the record's project (Activity Log Entry.project, or the Notification's affected project) and the " -- {project name}" part is omitted when the record has no project. Event labels: proposal_accepted_confirmation and proposal accepted = "Proposal accepted"; proposal_change_requested = "Changes requested on proposal"; milestone_approved_confirmation and milestone approved = "Milestone approved"; milestone reopened = "Milestone reopened"; invoice_issued and invoice sent = "Invoice sent"; payment_confirmation = "Payment received"; chargeback_notice = "Chargeback opened"; client_comment_alert = "New client comment"; credit note issued = "Credit note issued"; refund/reversal/cancellation recorded = "Refund, reversal, or cancellation recorded"; manual payment recorded = "Manual payment recorded".
- The rolling window is account-wide across all of Nadia's clients -- inclusion and order never distinguish which client an event belongs to (product-features.md, Key Capabilities: "chronological view of recent events across all clients").
- An underlying record that ages out of the window simply stops appearing on the next load; its Feed Item Read State is not retroactively deleted (no delete capability exists per the Authorization Rules above) -- it is only no longer surfaced.
- The permanent, unlimited-history record of every event remains in the Activity & Audit Trail (FEAT-13); this feed never substitutes for it (product-features.md, Validation & Limits).
- XBR-04 / ASMP-15: The underlying Notification and Activity Log Entry records are never altered by this composition rule -- it only reads them and derives display order and window membership.
- FEAT-29.SPEC-004's saved fallback copy of the feed is scoped by this same inclusion, window, and ordering rule at the moment it was captured.

## Edge Cases

- **Reference cannot be resolved** -- If the underlying Notification or Activity Log Entry a Feed Item Read State points to no longer exists, the candidate item is silently excluded from the feed; no error is shown to Nadia. This can only occur following FEAT-24 account deletion, at which point no feed exists to see it in either.
- **Two underlying events share the exact same timestamp** -- Ordering ties break by a stable secondary order (the order the two source entities were themselves created), so the two never swap relative position between loads.
- **An event's timestamp falls exactly at the window boundary** -- An event with a sent_at/occurred_at exactly `notification-feed-retention-window` old is still included (the boundary is inclusive); the next load, once time has moved on past it, excludes it.
- **A Notification and an Activity Log Entry describe the same underlying business event** (e.g., an invoice sent) -- Both are surfaced as separate feed items, each with its own Feed Item Read State, since neither the Feature Breakdown Brief nor the dependency map defines a de-duplication rule between the two source entities.
- **A record's type or recipient does not qualify** -- A Notification addressed to a client contact, a Notification with a type outside the qualifying list, or an Activity Log Entry with an excluded event_type never becomes a feed item and never receives a Feed Item Read State; no placeholder or error is shown.
- **read_status and read_at at the moment of update** -- FEAT-29.SPEC-003 always writes both fields together in the same operation; no intermediate state where one is set and the other is not is ever observable.

## Acceptance Criteria

**FEAT-29.SPEC-002-AC-01:** Given an underlying Notification with sent_at within `notification-feed-retention-window`, when the feed loads, then a Feed Item Read State is created for it (if none exists yet), defaulted to Unread.

**FEAT-29.SPEC-002-AC-02:** Given an underlying Activity Log Entry with occurred_at older than `notification-feed-retention-window`, when the feed loads, then no feed item is shown for it.

**FEAT-29.SPEC-002-AC-03:** Given a feed item is marked Read via FEAT-29.SPEC-003, then its read_at is set to the moment of the change; given it is later marked back to Unread, then read_at is cleared.

**FEAT-29.SPEC-002-AC-04:** Given any feed item is displayed, then its read_status and read_at are always mutually consistent -- read_at is set if and only if read_status is Read.

**FEAT-29.SPEC-002-AC-05:** Given Nadia opens her feed, then she sees every in-window Feed Item Read State on her own account.

**FEAT-29.SPEC-002-AC-06:** Given Owen looks for a notification feed in his portal, then no such screen or entry point exists for him.

**FEAT-29.SPEC-002-AC-07:** Given Priya looks for a notification feed in her portal, then no such screen or entry point exists for her.

**FEAT-29.SPEC-002-AC-08:** Given Dana is in an open support session, then no notification feed view is available to her.

**FEAT-29.SPEC-002-AC-09:** Given Nadia taps a feed item, then its read_status updates to Read.

**FEAT-29.SPEC-002-AC-10:** Given any role other than Nadia, then no control to change a feed item's read state exists anywhere in the product.

**FEAT-29.SPEC-002-AC-11:** Given Nadia views any feed item, when she looks for a delete option, then none exists.

**FEAT-29.SPEC-002-AC-12:** Given a new Notification enters the retention window for the first time, then its Feed Item Read State defaults to Unread without any user action.

**FEAT-29.SPEC-002-AC-13:** Given the feed contains both Notification-sourced and Activity-Log-sourced items, when displayed, then they are interleaved into a single list ordered newest-first by each item's own timestamp.

**FEAT-29.SPEC-002-AC-14:** Given an event's timestamp is exactly at the edge of `notification-feed-retention-window`, then it is still included on that load.

**FEAT-29.SPEC-002-AC-15:** Given two events share the exact same timestamp, then their relative order is stable across reloads.

**FEAT-29.SPEC-002-AC-16:** Given an underlying record referenced by a Feed Item Read State no longer exists, then that item is excluded from the feed with no error shown.

**FEAT-29.SPEC-002-AC-17:** Given an event ages out of the rolling window, then it no longer appears in the feed but remains permanently visible in the Activity & Audit Trail (FEAT-13).

**FEAT-29.SPEC-002-AC-18:** Given a milestone approval generates a milestone_approved_confirmation Notification for Owen and another for Nadia, when Nadia's feed loads, then only the Notification addressed to Nadia appears, with the approval icon and the summary "Milestone approved -- {project name}".

**FEAT-29.SPEC-002-AC-19:** Given in-window records exist for client_comment_alert (Notification), payment_confirmation (Notification), and "manual payment recorded" (Activity Log Entry), when the feed loads, then they appear with the comment, payment, and payment icons respectively and the summaries "New client comment -- {project name}", "Payment received -- {project name}", and "Manual payment recorded -- {project name}".

**FEAT-29.SPEC-002-AC-20:** Given in-window records exist for a welcome_email Notification, a freelancer_reply_alert Notification, and Activity Log Entries with event_type "reminder sent", "contact role changed", and "support session opened", when the feed loads, then none of them appears.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 10 | 10 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 9 | 9 |
| Edge Cases | 6 | 6 |
