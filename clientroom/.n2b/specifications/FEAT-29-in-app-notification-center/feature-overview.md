---
document_type: feature-overview
feature_number: FEAT-29
feature_name: In-App Notification Center
feature_slug: in-app-notification-center
priority_tier: Nice-to-Have
feature_type: Platform
produced_by: feature-analyst
status: final
created: 2026-09-27
spec_count: 4
screen_count: 1
automation_count: 2
logic_rule_count: 1
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: In-App Notification Center

## Summary

**Feature:** In-App Notification Center
**ID:** FEAT-29
**Description:** The freelancer sees a running feed of recent activity (approvals, payments, comments) inside the product, complementing the email notifications from FEAT-14.
**Priority:** Nice-to-Have
**Phase:** Later
**Type:** Platform
**Rationale:** The decomposition checklist's Cross-Cutting Concerns flags in-app notifications as a candidate alongside email; BRIEF.md establishes email as the required channel for clients, leaving an in-app feed as a freelancer-side convenience only. Nice-to-Have and phased Later since email already delivers every message the brief requires.

**Key Capabilities:**
- Recent-activity feed -- a chronological view of recent events across all clients
- Mark read/open -- triage the feed and jump to the related project

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-29.SPEC-001 | Notification Center Feed | Screen | Nadia | Nadia views a chronological feed of recent activity across all her clients, opens items, and reaches the related project |
| FEAT-29.SPEC-002 | Feed Composition & Retention Rule | Logic/Rule | Nadia | Defines which events from Notification and Activity Log Entry populate the feed, their order, the rolling recent-window limit, and each item's default read state |
| FEAT-29.SPEC-003 | Mark Item Read/Unread | Automation | Nadia | Records a feed item's read/unread state when Nadia opens or explicitly triages it, without altering the underlying Notification or Activity Log Entry |
| FEAT-29.SPEC-004 | Feed Load Fallback & Offline Continuity | Automation | Nadia | Falls back to the last successfully loaded feed on a failed refresh, and keeps that last-loaded feed viewable read-only when the connection is degraded or offline |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Recent-activity feed | FEAT-29.SPEC-001, FEAT-29.SPEC-002 | SPEC-001 is the chronological view Nadia sees; SPEC-002 defines what populates it, its ordering, and its rolling window | Phase 2 (Explicit) |
| Mark read/open | FEAT-29.SPEC-001, FEAT-29.SPEC-003 | SPEC-001 provides the open-project navigation inline; SPEC-003 is the standalone automation that records the read/unread state change | Phase 2 (Explicit) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-29.SPEC-002 | Feed Composition & Retention Rule | Phase 5 (Rule Discovery) | The Validation & Limits field ("a rolling recent window, not unlimited history") and the merge of two source entities (Notification, Activity Log Entry) into one ordered feed is non-trivial derivation logic shared by SPEC-001 and SPEC-004 -- it crosses the 5+/shared-logic threshold for a standalone Logic/Rule spec rather than staying inline |
| FEAT-29.SPEC-004 | Feed Load Fallback & Offline Continuity | Phase 6 (Negative/Failure Analysis) | The States field's Error line ("a failed feed load falls back to the last successfully loaded feed") and Offline-degraded line ("the last loaded feed remains viewable read-only") both describe cross-cutting caching behavior with real failure-handling logic, not a simple inline error message -- this crosses the standalone-Automation threshold (processing logic, not just a data write) |

## Entity-Lifecycle Coverage Matrix

**Entity: Feed Item Read State** (feature-local display state -- per the dependency map slice, "its own read/unread state per feed item is feature-local display state"; it is not the Connected Entity itself, only a per-item marker layered on top of it, and it must never alter the Notification or Activity Log Entry it points to, per XBR-04)

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-29.SPEC-002 | A read-state marker is implicitly created, defaulted to unread, the first time an underlying event enters the rolling feed window | Not a user-initiated create -- a derived side-effect of feed composition |
| Read (single) | FEAT-29.SPEC-001 | The feed screen shows each item's current read/unread state | -- |
| Read (list) | FEAT-29.SPEC-001 | The feed screen lists all in-window items with their states, newest first per SPEC-002's ordering rule | -- |
| Update | FEAT-29.SPEC-003 | Mark Item Read/Unread automation flips the marker when Nadia opens an item or explicitly triages it | Never edits the source Notification or Activity Log Entry (XBR-04) |
| Delete/Archive | N/A | Validation & Limits states plainly: "items can be marked read/unread but not deleted" -- recorded as an explicit non-goal (product-features.md, Validation & Limits) | -- |
| State Transition | FEAT-29.SPEC-003 | Read <-> Unread is the only state transition; no further lifecycle states are defined for a feed item | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Notification | FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-004 | Source events the feed surfaces (owned and created by FEAT-14); FEAT-29 reads only, per Connected Entities: "Notification (read -- surfaces existing records)" |
| Activity Log Entry | FEAT-29.SPEC-001, FEAT-29.SPEC-002, FEAT-29.SPEC-004 | Source events the feed surfaces (owned and created by FEAT-13); the Dependency Map Slice's Data Notes state the feed is "entirely derived from events recorded by ... FEAT-13 and FEAT-14" |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia opens the Notification Center | Load and merge in-window Notification and Activity Log Entry records into one ordered feed | Inline in triggering screen (uses composition rule) | FEAT-29.SPEC-001 (rule: FEAT-29.SPEC-002) |
| No recent activity exists | Show a calm "all caught up" empty state rather than implying something is broken | Inline in triggering screen | FEAT-29.SPEC-001 |
| Account has heavy recent activity | Show a lightweight loading indicator while the feed loads | Inline in triggering screen | FEAT-29.SPEC-001 |
| Nadia opens a feed item or explicitly marks it | Flip that item's local read/unread marker | Standalone Automation | FEAT-29.SPEC-003 |
| Nadia opens a feed item's related project link | Navigate to the project view | Cross-feature -- logged in touchpoints | FEAT-29.SPEC-001 (target: FEAT-01) |
| A feed refresh fails | Fall back to the last successfully loaded feed rather than showing a blank/broken screen | Standalone Automation | FEAT-29.SPEC-004 |
| Connectivity drops or degrades while viewing the feed | Keep the last loaded feed viewable, read-only (no new loads, no read/unread changes queued) | Standalone Automation | FEAT-29.SPEC-004 |
| A role other than Nadia would otherwise reach this feed | Capability is not shown at all -- no in-app feed entry point renders for Owen, Priya, or Dana | Inline in triggering screen (authorization) | FEAT-29.SPEC-001 |

## Shared Context

**Shared Entities:**
- Notification (read-only) -- read by SPEC-001, SPEC-002, SPEC-004. Fields (per the dependency map slice): notification_type, recipient, sent_at, delivery_status. Owned and created by FEAT-14; FEAT-29 never writes to it.
- Activity Log Entry (read-only) -- read by SPEC-001, SPEC-002, SPEC-004. Fields: event_type, actor, occurred_at, affected_record, project. Owned and created by FEAT-13; append-only and never edited or deleted by anyone (ASMP-15), including by this feature.
- Feed Item Read State (feature-local) -- created (implicitly) and read by SPEC-001/SPEC-002, updated by SPEC-003. Fields: reference to the underlying Notification or Activity Log Entry, read_status (Read/Unread), read_at.

**Shared UI Patterns:**
- Feed list item -- one consistent row rendering (source icon/type, summary, timestamp, read/unread indicator, tap-to-open) used across every state variant (populated, loading, error-fallback, offline) of SPEC-001. Spec Writers should describe it once and reference it from every state.
- Canonical state set -- Empty, Loading, Error (fallback), Offline-degraded are the four states named in product-features.md's States field for this feature; SPEC-001 must cover all four, and SPEC-004 defines the mechanics behind the latter two.

**Shared Validation:**
- SPEC-002 is the single source of truth for "what is in the feed, in what order, and for how long back." SPEC-001 (display) and SPEC-004 (fallback caching) both reference SPEC-002's rolling-window and ordering definition rather than each restating it.

## Internal Dependency Map

```
SPEC-001 (Notification Center Feed) -> [feed composed per] -> SPEC-002 (Feed Composition & Retention Rule)
SPEC-001 (Notification Center Feed) -> [Nadia opens or triages an item] -> SPEC-003 (Mark Item Read/Unread)
SPEC-001 (Notification Center Feed) -> [Nadia opens an item's related project] -> FEAT-01 (project view) [cross-feature]
SPEC-001 (Notification Center Feed) -> [refresh fails, or connectivity drops] -> SPEC-004 (Feed Load Fallback & Offline Continuity)
SPEC-004 (Feed Load Fallback & Offline Continuity) -> [serves cached feed scoped by] -> SPEC-002 (Feed Composition & Retention Rule)
SPEC-003 (Mark Item Read/Unread) -> [read state reflected back in] -> SPEC-001 (Notification Center Feed)
```

**Default Entry:** SPEC-001 (Notification Center Feed) -- the only screen in this feature, shown whenever Nadia navigates to the notification center.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-29.SPEC-001 | Outbound | FEAT-01 (Client & Project Management) | Navigates to the related project's view | Nadia opens the related project from a feed item |
| FEAT-29.SPEC-001, FEAT-29.SPEC-002 | Inbound | FEAT-13 (Immutable Activity & Audit Trail) | Reads Activity Log Entry records to compose the feed | Feed load/refresh |
| FEAT-29.SPEC-001, FEAT-29.SPEC-002 | Inbound | FEAT-14 (Notifications: Email) | Reads Notification records to compose the feed | Feed load/refresh |
| FEAT-29.SPEC-004 | Inbound | FEAT-13, FEAT-14 | Fallback/cache mechanics rely on the same two source entities read by SPEC-002 | Feed refresh fails, or connectivity degrades |

## Non-Functional Notes

**Data volumes / growth:** The feed shows only a rolling recent window, never unlimited history -- the full permanent record lives in the Activity & Audit Trail (FEAT-13), so this feature's own data footprint does not grow unbounded even as an account accumulates years of activity (product-features.md, Validation & Limits).

**Responsiveness:** No feature-specific numeric target is stated for this freelancer-side screen (ASMP-21's ~1-2s targets are scoped to client-facing pages and the financial dashboard); the feature's own States field sets the qualitative expectation instead -- the feed shows a lightweight loading indicator specifically for accounts with heavy recent activity, so a slow load is never silent (product-features.md, States).

**Data sensitivity / privacy:** The feed surfaces Notification content, which is personal data (recipient name, project-related message content) classified GDPR-class (ASMP-24); because this is Nadia's own account-wide feed across all her clients, it does not cross the strict per-client data isolation boundary (ASMP-23) -- it aggregates only her own account's events, never another freelancer's or another client's data.

**Compliance flags:** GDPR-class handling applies to the underlying Notification and Activity Log Entry content this feed displays (ASMP-24); the feed introduces no new compliance surface of its own since it creates no new personal-data records, only a local read/unread marker.

## Non-Goals

- **Deleting feed items** -- Excluded per product-features.md, Validation & Limits: "items can be marked read/unread but not deleted." The permanent record of every event remains in the Activity & Audit Trail (FEAT-13); this feed is a display convenience, not a system of record, so no delete operation exists for it.
- **An equivalent in-app feed for client contacts** -- Excluded per product-features.md, Access: "Nadia only (Full, her own feed); client contacts have no equivalent in-app feed." BRIEF.md establishes email as the required notification channel for clients; an in-app feed is a freelancer-side convenience only.
- **Support Operator access to this feed** -- Excluded per product-features.md, Access ("Dana ... has no access to this personal feed") and scope-boundaries.md SC-04: Support Operator access (FEAT-31) is read-only and scoped to operational support, and does not extend to a freelancer's personal notification feed; the Access Matrix's Dana "View (delivery warnings only)" cell applies to FEAT-14 delivery warnings, not this feature.
- **Sending new notifications** -- Excluded per product-features.md, Communications: "N/A -- this surfaces notifications already sent by FEAT-14; it does not send new ones." Notification delivery and preferences are owned entirely by FEAT-14 (XBR-30); this feature only reads what FEAT-14 and FEAT-13 already recorded.
- **An external-capability (Integration) surface** -- Excluded per the dependency map slice's External Touchpoints: "None -- FEAT-29 appears in no External Touchpoints row and relies on no external capability." Every event the feed shows was already delivered through FEAT-14's email channel or logged by FEAT-13; this feature crosses no product boundary of its own, so no Integration spec applies.
