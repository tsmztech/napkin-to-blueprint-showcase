# FEAT-29 — In-App Notification Center

This chapter covers In-App Notification Center, a Nice-to-Have-tier feature. It contains the feature breakdown brief followed by every specification in full: 4 specifications carrying 60 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-29.SPEC-001 | Notification Center Feed | screen | 17 |
| FEAT-29.SPEC-002 | Feed Composition & Retention Rule | logic-rule | 20 |
| FEAT-29.SPEC-003 | Mark Item Read/Unread | automation | 10 |
| FEAT-29.SPEC-004 | Feed Load Fallback & Offline Continuity | automation | 13 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Notification Center Feed

## Overview

**Name:** Notification Center Feed
**ID:** FEAT-29.SPEC-001
**Type:** Screen
**Purpose:** Nadia views a chronological feed of recent activity across all her clients, triages items as read or unread, and opens any item to reach its related project.
**Parent Feature:** FEAT-29 -- In-App Notification Center

## Scope and Non-Goals

**In Scope:**
- Displaying the merged, ordered, rolling-window feed composed by FEAT-29.SPEC-002
- Empty, loading, error (fallback), and offline-degraded states for this screen
- Opening a feed item to mark it read and navigate to its related project
- An explicit read/unread toggle per item, without navigating away
- Restricting the screen entirely to Nadia -- no entry point renders for any other role

**Non-Goals:**
- Sending new notifications -- excluded per product-features.md, Communications: "N/A -- this surfaces notifications already sent by FEAT-14; it does not send new ones." This screen only displays existing records.
- Deleting feed items -- excluded per product-features.md, Validation & Limits: "items can be marked read/unread but not deleted." No delete control exists on this screen.
- Defining the feed's composition, ordering, or rolling-window retention -- owned entirely by FEAT-29.SPEC-002 (Feed Composition & Retention Rule); this screen only displays what that rule produces.
- Defining the fallback and offline-continuity mechanics behind the Error and Offline/Degraded states (including keeping the saved copy of the last loaded feed) -- owned by FEAT-29.SPEC-004 (Feed Load Fallback & Offline Continuity); this screen only reflects that automation's outcomes.
- An equivalent feed for client contacts -- excluded per product-features.md, Access: "Nadia only (Full, her own feed); client contacts have no equivalent in-app feed."

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| Freelancer-side global navigation (a persistent notification icon available from any screen in Nadia's account) | Nadia taps the notification icon | None -- the feed always loads Nadia's own current in-window items; no parameters are passed in |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Open items (marks read, navigates to project), explicit read/unread toggle per item, Retry on error | -- |
| Owen (Client Primary Contact) | No | No | The notification icon and this screen do not exist anywhere in Owen's portal navigation (FEAT-05) -- this is a freelancer-only screen per product-features.md, Access. A direct link to it is out of Owen's portal scope and is handled as any out-of-scope link (FEAT-05.SPEC-006): a plain explanation and a fresh sign-in link, never this feed. |
| Priya (Client Reviewer Contact) | No | No | Same as Owen -- no entry point exists, and a direct link is handled as an out-of-scope link per FEAT-05.SPEC-006. |
| Dana (Support Operator) | No | No | No entry point is shown in Dana's Operator Support Session Console (FEAT-31.SPEC-002) -- this personal feed is excluded from support access entirely (product-features.md, Access: "Dana ... has no access to this personal feed"). Dana's Access Matrix cell "View (delivery warnings only)" applies to FEAT-14's delivery warnings, not to this screen. |
| Unauthenticated | No | No | Redirected to Nadia's sign-in screen; after signing in, the visitor lands on their own account's notification feed only -- never another account's. |
| Expired session | No | No | A session-expired dialog appears and the feed does not load; any in-progress read/unread toggle at the moment of expiry is discarded on Nadia's device (the underlying record never received the write, per FEAT-29.SPEC-003) and Nadia must sign in again before the feed reopens. |

## Layout and Content

**Header:** Screen title "Notifications." No filter, search, or sort control -- the feed's only ordering is the fixed newest-first order defined by FEAT-29.SPEC-002.

**Body:** A single vertically scrolling list of feed items, newest first. Each row uses the feature's shared feed-list-item pattern, consistent across every state variant of this screen (populated, loading, error-fallback, offline):
- A source icon/type indicator distinguishing an approval, payment, or comment-originated event (derived from the underlying Notification's notification_type or the Activity Log Entry's event_type, per the mapping in FEAT-29.SPEC-002 Business Rules)
- A one-line summary of the event in the form "{event label} -- {project name}", composed per FEAT-29.SPEC-002 Business Rules
- A relative timestamp (from the underlying record's sent_at or occurred_at)
- A read/unread indicator (visually distinct treatment for unread items)
- An explicit read/unread toggle control, distinct from the row's main tap target

The entire row (outside the toggle control) is the tap target that opens the item.

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Single-column list, full width; each row stacks its icon, summary, timestamp, and indicator/toggle horizontally within the row.
- **Medium size class and above:** The list column is capped at a consistent platform-wide reading width and horizontally centered; row structure is unchanged -- uniform scaling, no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Feed item row (main tap target) -- Populated or Error (fallback) state, item has a project that still exists (Active or Archived) | Tap | If the item is Unread, triggers FEAT-29.SPEC-003 (Mark Item Read/Unread) to mark it Read; then navigates to FEAT-01.SPEC-005 (Project Detail) for the item's related project. Navigation does not wait for the write to finish and is never cancelled by a write failure; an Archived project opens normally in Project Detail | Item's read indicator updates to Read before navigation; if the write fails, the indicator reverts to Unread (FEAT-29.SPEC-003, Failure from open) | Read indicator changes; screen transitions to the project detail; on write failure the toast "Couldn't update -- try again." appears over the project detail (or over the feed if navigation has not yet happened) |
| Feed item row (main tap target, item already Read) | Tap | Triggers FEAT-29.SPEC-003, which performs a no-op write (already Read); navigates to FEAT-01.SPEC-005 (same project rules as the row above) | No visible indicator change | Screen transitions to the project detail |
| Feed item row whose record has no project (an account-level item) | Tap | Marks the item Read via FEAT-29.SPEC-003 if Unread; does not navigate | Item's read indicator updates to Read; screen stays on the feed | Toast "This item isn't linked to a project." over the feed |
| Feed item row whose project can no longer be found when tapped | Tap | Marks the item Read via FEAT-29.SPEC-003 if Unread; does not navigate | Item's read indicator updates to Read; screen stays on the feed | Toast "This project is no longer available." over the feed |
| Feed item row or read/unread toggle while in the Offline/Degraded state | Tap | No action -- both controls are inert (FEAT-29.SPEC-004); FEAT-29.SPEC-003 is not triggered | None | None (no navigation, no message) |
| Read/unread toggle (per row) | Tap | Triggers FEAT-29.SPEC-003 to flip the item's read_status without navigating | Indicator flips between Read and Unread treatment | Toggle animates; screen remains on the feed |
| Notification icon (global navigation) | Tap | Opens this screen and initiates a feed load | Screen transitions to Loading or Populated | Loading indicator (if load is not instant) then the feed appears |
| Retry control (shown in the Error/fallback state) | Tap | Reattempts the feed load via FEAT-29.SPEC-004 (Feed Load Fallback & Offline Continuity) | Screen re-enters Loading | Loading indicator, then Populated or the same fallback state |

### Accessibility Notes

- **Focus order:** Notification icon (entry) -> feed items top to bottom, each row's main tap target before its toggle control -> Retry control (when shown).
- **Read/unread announcements:** When a toggle or item open flips read_status, the row's updated state is announced to assistive technology.
- **Loading and error announcements:** The Loading indicator's appearance, the Error/fallback banner, and the Offline/Degraded banner are each announced when they appear.
- **Keyboard alternatives:** Every action on this screen (open item, toggle read/unread, Retry) is reachable by keyboard focus and activation; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | Plain "All caught up" message, no list items | Feed load completes with zero in-window items (per FEAT-29.SPEC-002) | A new in-window event exists on the next load |
| Loading | A lightweight loading indicator, shown specifically for accounts with heavy recent activity | A feed load or refresh is initiated and has not yet returned | Load completes (Populated, Empty, or Error/fallback) |
| Populated | The merged, ordered feed list with each item's read/unread indicator and toggle | Load completes with one or more in-window items | Screen navigates away, or a new refresh begins |
| Error (fallback) | Banner stating the refresh failed, with a Retry control; the last successfully loaded feed content is shown beneath it (per FEAT-29.SPEC-004). Item-open controls, read/unread toggles, and Retry are all active in this state. If no feed was ever loaded, the state shows the banner and Retry with no items, so only Retry is active | A refresh fails while the device is online | Retry succeeds and replaces the fallback content, or the fallback persists through repeated failures |
| Offline/Degraded | Banner "You're offline -- showing your last loaded notifications." with the last loaded feed visible; every toggle and item-open control is inert, and no Retry control is shown (per FEAT-29.SPEC-004). If no feed was ever loaded, the banner reads "You're offline -- notifications will load when you reconnect." with no items | Connectivity drops or degrades while this screen is open, OR Nadia opens this screen while the device is already offline (no load is attempted) | Connectivity is restored and the automatic reload (FEAT-29.SPEC-004) completes |

## Validation Rules

This screen collects no free-text or form input -- its only user input is tap interactions. The read/unread state transition triggered by those taps is governed entirely by FEAT-29.SPEC-002 (Feed Composition & Retention Rule) and carried out by FEAT-29.SPEC-003 (Mark Item Read/Unread). This screen applies no validation of its own beyond those referenced rules.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Tap a feed item whose project exists (Active or Archived) | FEAT-01.SPEC-005 (Project Detail) | FEAT-01 (Client & Project Management) |
| Tap a feed item with no project, or whose project can no longer be found | Remains on FEAT-29.SPEC-001 (Notification Center Feed), with a toast ("This item isn't linked to a project." or "This project is no longer available.") | -- |
| Tap a feed item or toggle while Offline/Degraded | Remains on FEAT-29.SPEC-001 (Notification Center Feed); controls are inert | -- |
| Tap Retry (Error/fallback state) | Remains on FEAT-29.SPEC-001 (Notification Center Feed) | -- |

## Data Model

**Creates:** Feed Item Read State -- not created directly by this screen; created implicitly by FEAT-29.SPEC-002 the first time an underlying event enters the rolling window, and simply surfaced here.
**Reads:** Notification (notification_type, sent_at, used for the item's icon and timestamp) and Activity Log Entry (event_type, occurred_at, actor, affected_record, project) -- merged and ordered by FEAT-29.SPEC-002; Feed Item Read State (reference, read_status, read_at) -- used for each item's read/unread indicator.
**Updates:** Feed Item Read State -- read_status and read_at, but only via the automation this screen triggers (FEAT-29.SPEC-003), never written directly by this screen.
**Deletes:** None -- feed items cannot be deleted (product-features.md, Validation & Limits).

## Business Rules

- Feed composition, ordering, and the rolling-window retention limit are governed entirely by FEAT-29.SPEC-002 -- this screen never defines its own window or order.
- Every read/unread transition is governed by, and carried out through, FEAT-29.SPEC-003 -- this screen never writes the read state directly itself.
- XBR-04: The underlying Notification and Activity Log Entry records are evidentiary and immutable; this screen never alters them, only the local Feed Item Read State.
- Item-open and toggle controls are active in the Populated and Error (fallback) states and inert in the Offline/Degraded state (FEAT-29.SPEC-004); opening an item always marks it Read (when Unread) even if it cannot navigate, and navigation is never blocked by a read-state write failure.
- Access is restricted to Nadia's own account-wide feed; per user-persona.md's Access Matrix ("Notifications & Help" is Nadia: Full, everyone else: None for this personal feed), no other role sees this screen at all.

## Edge Cases

- **Heavy recent activity while loading** -- The Loading indicator remains visible until the entire in-window feed loads; the list renders complete on the first paint, with no partial or staggered rendering defined.
- **Item has no project** -- Only account-level items (for example a chargeback notice not tied to one project) lack a project reference. Tapping marks the item Read, stays on the feed, and shows the toast "This item isn't linked to a project."
- **Item's project is archived or unavailable** -- An archived project still exists and opens normally in FEAT-01.SPEC-005 (where Nadia can reactivate it). If the project record cannot be found at tap time, Nadia stays on the feed, the item is marked Read, and the toast "This project is no longer available." appears.
- **Read-state write fails on an open tap** -- Navigation to the project still proceeds; the item's indicator reverts to Unread and the toast "Couldn't update -- try again." appears over the screen showing at the moment of failure (FEAT-29.SPEC-003, Failure from open). Nadia sees the item as Unread when she returns to the feed.
- **Nadia opens the screen while the device is already offline** -- No load is attempted; the screen opens directly in Offline/Degraded with her last loaded feed (or the no-items offline message if none exists), and the item-open and toggle controls are inert (FEAT-29.SPEC-004).
- **Nadia taps two different feed items in rapid succession** -- Each tap independently triggers its own FEAT-29.SPEC-003 run and its own navigation; the second tap's navigation is the one that lands, since navigation to the first target is superseded before it completes.
- **Nadia taps the same item's toggle twice rapidly** -- The second tap is ignored while the first FEAT-29.SPEC-003 run for that item is in flight; the toggle shows a brief transitional state.
- **Nadia navigates away and returns to the feed** -- The feed re-loads on return, recomposed per FEAT-29.SPEC-002's current window; any item that aged out of the window since the previous visit no longer appears.
- **No concurrent-edit conflict on this screen** -- This screen only reads Notification and Activity Log Entry, neither of which it ever writes, and updates only the feature-local Feed Item Read State, which per the dependency map carries no shared-entity Contention note (it is not a listed Shared Data Entity). Two of Nadia's own open sessions marking the same item at once both converge on the same final state with no error shown (see FEAT-29.SPEC-003, Edge Cases, for the exact resolution).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-002 (Feed Composition & Retention Rule) | References (inbound) | Governs what appears in the feed, its ordering, and its rolling window |
| FEAT-29.SPEC-003 (Mark Item Read/Unread) | Triggers (outbound) | Item open or explicit toggle triggers the read-state change |
| FEAT-29.SPEC-004 (Feed Load Fallback & Offline Continuity) | Triggers (outbound) / Affects (inbound) | Refresh failures and connectivity changes are handled by this automation, which returns the Error/fallback or Offline/Degraded outcome this screen displays |
| FEAT-01.SPEC-005 (Project Detail) | Navigation (outbound) | Opening a feed item navigates to the related project |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| notification_center_opened | none | Nadia opens this screen | N/A — no metric in success-metrics.md names In-App Notification Center as its Connected Feature (verified against every metric entry's Connected Feature field); the feed is a Nice-to-Have convenience layer outside this run's measured funnel. Recorded here as the Signal product-features.md declares for this feature (notification_center_opened). |
| notification_item_opened | source_type (Notification / Activity Log Entry), read_state_before | Nadia taps a feed item to open its related project | N/A — same reason; recorded as the declared Signal notification_item_opened. |
| notification_marked_read | source_type, trigger (open / explicit_toggle) | A feed item's read/unread state flips via FEAT-29.SPEC-003 | N/A — same reason; recorded as the declared Signal notification_marked_read. |

## Acceptance Criteria

**FEAT-29.SPEC-001-AC-01:** Given Nadia opens the Notification Center with no recent activity in the window, then she sees the "All caught up" empty state with no list items.

**FEAT-29.SPEC-001-AC-02:** Given Nadia's account has heavy recent activity, when she opens the Notification Center, then a lightweight loading indicator is shown until the feed populates.

**FEAT-29.SPEC-001-AC-03:** Given Nadia is viewing a populated feed, when she taps an unread feed item, then its read indicator updates to Read and she is navigated to FEAT-01.SPEC-005 (Project Detail) for the item's related project.

**FEAT-29.SPEC-001-AC-04:** Given Nadia is viewing a populated feed, when she taps the read/unread toggle on a Read item without opening it, then the item flips to Unread and she remains on the feed screen.

**FEAT-29.SPEC-001-AC-05:** Given Nadia's feed refresh fails, then the screen shows the Error/fallback banner with the last successfully loaded feed and a Retry control.

**FEAT-29.SPEC-001-AC-06:** Given Nadia's connectivity drops while she is viewing the feed, then the screen shows the Offline/Degraded banner, keeps her last loaded feed visible, and makes every toggle and item-open control inert.

**FEAT-29.SPEC-001-AC-07:** Given Owen (Client Primary Contact) is signed into his portal, when he looks for a notification center entry point, then none exists anywhere in his portal navigation.

**FEAT-29.SPEC-001-AC-08:** Given Dana (Support Operator) is in an open support session, when she looks at the Operator Support Session Console (FEAT-31.SPEC-002), then no notification center entry point is shown.

**FEAT-29.SPEC-001-AC-09:** Given an unauthenticated visitor requests this screen directly, then they are redirected to Nadia's sign-in screen.

**FEAT-29.SPEC-001-AC-10:** Given Nadia's session expires while the feed is open, when she attempts to toggle an item's read state, then a session-expired dialog appears and the toggle is not applied.

**FEAT-29.SPEC-001-AC-11:** Given Nadia taps two different feed items in rapid succession, then she is navigated to the second-tapped item's project, and both items' read states update independently.

**FEAT-29.SPEC-001-AC-12:** Given items have aged out of the rolling window since Nadia's last visit, when she returns to the feed, then those items no longer appear on the reload.

**FEAT-29.SPEC-001-AC-13:** Given Nadia taps an unread feed item that has no project (an account-level item), then the item is marked Read, she remains on the feed, and the toast "This item isn't linked to a project." appears.

**FEAT-29.SPEC-001-AC-14:** Given Nadia taps a feed item whose project is Archived, then she is navigated to FEAT-01.SPEC-005 (Project Detail) for that project; and given the project record can no longer be found, then she remains on the feed, the item is marked Read, and the toast "This project is no longer available." appears.

**FEAT-29.SPEC-001-AC-15:** Given the read-state write fails, when Nadia taps an unread feed item to open it, then she is still navigated to the project, the item's indicator reverts to Unread, and the toast "Couldn't update -- try again." appears over the screen showing at the moment of failure.

**FEAT-29.SPEC-001-AC-16:** Given Nadia's device is already offline, when she opens the Notification Center, then no load is attempted, the screen shows the Offline/Degraded banner with her last loaded feed (or the no-items offline message if none exists), and every item-open and toggle control is inert.

**FEAT-29.SPEC-001-AC-17:** Given the screen is in the Error (fallback) state showing the previous feed, when Nadia taps an item row or a toggle, then the control works as in the Populated state; and given the Error state has no items, then only Retry is active.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 5 (empty, loading, populated, error/fallback, offline/degraded) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 9 | 9 |



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



# Automation Spec: Mark Item Read/Unread

## Overview

**Name:** Mark Item Read/Unread
**ID:** FEAT-29.SPEC-003
**Type:** Automation
**Purpose:** Records a feed item's read/unread state when Nadia opens it or explicitly triages it, without altering the underlying Notification or Activity Log Entry.
**Parent Feature:** FEAT-29 -- In-App Notification Center

## Scope and Non-Goals

**In Scope:**
- Flipping a Feed Item Read State's read_status and read_at when Nadia opens an item or taps its explicit toggle
- Success, no-op, and failure feedback for that write

**Non-Goals:**
- Composing or sending new notifications -- owned entirely by FEAT-14 (XBR-30); this automation only updates a local read/unread marker on a record FEAT-14 already sent.
- Deleting feed items -- excluded per product-features.md, Validation & Limits: "items can be marked read/unread but not deleted."
- Altering the underlying Notification or Activity Log Entry -- excluded per XBR-04 and ASMP-15 (both are evidentiary and immutable); this automation only ever writes the feature-local Feed Item Read State (FEAT-29.SPEC-002).
- Deciding which events are in the feed or their order -- owned entirely by FEAT-29.SPEC-002; this automation only changes the read/unread field of a record that composition rule already created.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia opens a feed item (taps the row to navigate) | FEAT-29.SPEC-001 (Notification Center Feed) | Fires on every open tap, regardless of current read_status, EXCEPT while FEAT-29.SPEC-001 is in the Offline/Degraded state (item-open controls are inert there per FEAT-29.SPEC-004, so no tap reaches this automation) or after Nadia's session has expired | Item reference, current read_status |
| Nadia taps the explicit read/unread toggle on a feed item | FEAT-29.SPEC-001 (Notification Center Feed) | Fires on every toggle tap, regardless of current read_status, EXCEPT while FEAT-29.SPEC-001 is in the Offline/Degraded state (toggle controls are inert there per FEAT-29.SPEC-004, so no tap reaches this automation) or after Nadia's session has expired. It does fire in the Populated state and in the Error (fallback) state when saved feed items are shown | Item reference, current read_status |

## Processing Logic

1. Receive the feed item's reference and its current read_status from the triggering interaction.
2. Determine the target state: opening an item always targets Read; the explicit toggle targets the opposite of the item's current state.
3. If the target state equals the current state (opening an item that is already Read), skip the write and proceed directly to the no-op outcome.
4. Otherwise, write the new read_status to the item's Feed Item Read State record, and set read_at to the current moment if the new state is Read, or clear read_at if the new state is Unread (per FEAT-29.SPEC-002's field rules).
5. Confirm the write succeeded and signal the triggering screen to display the updated indicator.
6. If the write does not succeed, signal the triggering screen to revert the indicator to its prior state and show the failure toast "Couldn't update -- try again." The toast is a transient message displayed over whichever screen is showing when the failure is reported. For an open-triggered run, the failure never blocks or cancels the navigation that the open tap started: Nadia still reaches the project (FEAT-29.SPEC-001), the toast appears over the project screen if navigation has already happened, and the item shows Unread when she returns to the feed. For a toggle-triggered run, Nadia is still on the feed, so the toast appears over the feed and the toggle returns to its prior state.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Marked Read (from open) | Item was Unread and Nadia opened it | read_status -> Read, read_at set | Indicator updates to Read before navigation completes | FEAT-29.SPEC-001 |
| Marked Read (from toggle) | Item was Unread and Nadia tapped the toggle | read_status -> Read, read_at set | Indicator updates to Read; screen remains on the feed | FEAT-29.SPEC-001 |
| Marked Unread (from toggle) | Item was Read and Nadia tapped the toggle | read_status -> Unread, read_at cleared | Indicator updates to Unread | FEAT-29.SPEC-001 |
| No-op (already Read, opened) | Item was already Read and Nadia opened it | None | No visible indicator change; navigation proceeds | FEAT-29.SPEC-001 |
| Failure (from toggle) | The write does not complete after Nadia tapped the toggle | None persists | Toggle and indicator revert to their prior state; toast "Couldn't update -- try again." over the feed; Nadia stays on the feed | FEAT-29.SPEC-001 |
| Failure (from open) | The write does not complete after Nadia tapped a feed item row to open it | None persists | Navigation to the project proceeds unaffected; the item's indicator reverts to Unread; toast "Couldn't update -- try again." over the screen showing at the time of failure (the project screen if navigation already completed) | FEAT-29.SPEC-001 |

## Data Model

**Reads:** Feed Item Read State -- reference and current read_status of the target item.
**Creates:** None -- the record already exists by the time this automation runs; it was created implicitly by FEAT-29.SPEC-002 when the item first entered the rolling window.
**Updates:** Feed Item Read State -- read_status and read_at, per FEAT-29.SPEC-002's field rules.
**Deletes:** None.

## Business Rules

- The underlying Notification or Activity Log Entry a feed item references is never written by this automation -- only the feature-local Feed Item Read State changes (XBR-04, ASMP-15).
- read_status and read_at are always written together in the same operation (FEAT-29.SPEC-002, Cross-Field Rules) -- no intermediate state is ever observable.
- This automation is the only writer of Feed Item Read State's read_status/read_at fields in the product (FEAT-29.SPEC-002, Authorization Rules).

## Edge Cases

- **Opening an item that is already Read** -- No-op per the Outcome Definitions table; navigation still proceeds normally.
- **Nadia taps the toggle on the same item twice in rapid succession** -- The second tap is ignored while the first run for that specific item is in flight; the item's toggle control is disabled for the duration of the in-flight write (FEAT-29.SPEC-001).
- **Concurrent trigger firing (two of Nadia's own sessions act on the same item at effectively the same time)** -- Each run writes independently; the run that completes last determines the item's final read_status (last-write-wins). Feed Item Read State has no contending writer other than Nadia's own concurrent sessions (per FEAT-29.SPEC-002's Authorization Rules), so no conflict message is shown.
- **Trigger fires while a previous run for a different item is in flight** -- Runs for different items proceed independently; there is no queuing between items.
- **Nadia loses connectivity while a run is in flight, or a control is tapped while the screen is Offline/Degraded** -- A tap made while the screen is already Offline/Degraded never fires this automation (the controls are inert, per FEAT-29.SPEC-004). A run already in flight when connectivity drops that does not complete follows the Failure outcome for its trigger path.
- **The write completes successfully but the confirmation signal is delayed or lost** -- If a later run for the same item finds the record already matches the intended target state, no further write or visible change occurs; the screen is not falsely reverted.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-001 (Notification Center Feed) | Triggered by (inbound) | Item open or explicit toggle fires this automation |
| FEAT-29.SPEC-001 (Notification Center Feed) | Affects (outbound) | Returns the updated (or reverted) read/unread indicator to the screen |
| FEAT-29.SPEC-002 (Feed Composition & Retention Rule) | References (inbound) | Field rules and authorization governing read_status/read_at |

## Analytics and Success Signals

- **notification_marked_read** (trigger: open / explicit_toggle; previous_state: Read / Unread) -- N/A — no metric in success-metrics.md names In-App Notification Center as its Connected Feature (verified against every metric entry's Connected Feature field); recorded here as the Signal product-features.md declares for this feature (notification_marked_read).
- **notification_mark_read_failed** (trigger: open / explicit_toggle; reason: write_failed) -- N/A — same reason; this event exists only to observe how often the failure path (Outcome Definitions, Failure) is exercised, not to feed a Stage 2 metric.

## Acceptance Criteria

**FEAT-29.SPEC-003-AC-01:** Given Nadia has an unread feed item, when she taps it to open it, then its read_status updates to Read, read_at is set, and she is navigated to the item's related project.

**FEAT-29.SPEC-003-AC-02:** Given Nadia has an unread feed item, when she taps its explicit read/unread toggle without opening it, then its read_status updates to Read and read_at is set, and she remains on the feed screen.

**FEAT-29.SPEC-003-AC-03:** Given Nadia has a read feed item, when she taps its explicit toggle, then its read_status updates to Unread and read_at is cleared.

**FEAT-29.SPEC-003-AC-04:** Given Nadia has an already-read feed item, when she opens it, then no write occurs and she is navigated to the related project without any indicator change.

**FEAT-29.SPEC-003-AC-05:** Given the read-state write fails, when Nadia taps an item's toggle, then the indicator reverts to its prior state and the message "Couldn't update -- try again." appears.

**FEAT-29.SPEC-003-AC-06:** Given Nadia taps the same item's toggle twice in rapid succession, then the second tap is ignored while the first write is in flight.

**FEAT-29.SPEC-003-AC-07:** Given two of Nadia's own sessions mark the same item at effectively the same time, then the item's final read_status is whichever write completes last, with no conflict message shown.

**FEAT-29.SPEC-003-AC-08:** Given Nadia triggers this automation for two different items at once, then each run completes independently with no queuing between them.

**FEAT-29.SPEC-003-AC-09:** Given the read-state write fails, when Nadia taps an unread feed item to open it, then she is still navigated to the item's related project, the item's indicator reverts to Unread, and the toast "Couldn't update -- try again." appears over the screen showing at the time of failure.

**FEAT-29.SPEC-003-AC-10:** Given the feed screen is in the Offline/Degraded state, when Nadia taps a feed item row or its toggle, then this automation does not fire, no write is attempted, and no failure toast appears.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (open, explicit toggle) | 2 |
| Outcome Paths | 6 (marked read from open, marked read from toggle, marked unread, no-op, failure from toggle, failure from open) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |



# Automation Spec: Feed Load Fallback & Offline Continuity

## Overview

**Name:** Feed Load Fallback & Offline Continuity
**ID:** FEAT-29.SPEC-004
**Type:** Automation
**Purpose:** Falls back to the last successfully loaded feed on a failed refresh, and keeps that last-loaded feed viewable read-only when the connection is degraded or offline, including when Nadia opens the screen while already offline.
**Parent Feature:** FEAT-29 -- In-App Notification Center

## Scope and Non-Goals

**In Scope:**
- Keeping a saved copy of the last successfully loaded feed on Nadia's device for viewing
- Showing that saved copy when a refresh fails
- Freezing the screen to a read-only view of the saved copy when connectivity degrades or drops, and opening straight into that read-only view when Nadia opens the screen while already offline
- Stating which controls stay active or become inert in the Error (fallback) and Offline/Degraded states
- Automatically reattempting a load once connectivity is restored

**Non-Goals:**
- Composing the feed's content or ordering -- governed entirely by FEAT-29.SPEC-002 (Feed Composition & Retention Rule); this automation only decides when to show the saved copy of that composition versus attempting a fresh one.
- Queuing read/unread changes for later submission while offline -- excluded per product-features.md, States: the offline-degraded feed is "viewable read-only," so no mutation is queued. FEAT-29.SPEC-003's triggers are simply unavailable in this state, not deferred.
- Any change to the underlying Notification or Activity Log Entry -- this automation only manages a saved copy, on Nadia's device, of already-composed feed content; it never writes either source entity (XBR-04, ASMP-15).

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Feed refresh fails | FEAT-29.SPEC-001 (Notification Center Feed) | Fires when a feed load or refresh attempt does not complete successfully while the device is online | The last successfully loaded feed content, if one exists |
| Connectivity degrades or drops while the feed screen is open | system (device connectivity signal) | Fires when the device detects a network condition that prevents reliable feed loads while this screen is open | The last successfully loaded feed content, if one exists |
| Feed screen is opened while the device is already offline | FEAT-29.SPEC-001 (Notification Center Feed) | Fires when Nadia opens the screen and the device connectivity signal already reports offline or degraded, so no load is attempted | The last successfully loaded feed content, if one exists |
| Connectivity is restored | system (device connectivity signal) | Fires when the device detects connectivity is available again after a degraded/offline period, whether that period began while this screen was open or before Nadia opened it, and the screen is still open | None beyond the restored-connectivity signal itself |

## Processing Logic

1. On every successful feed load (composed per FEAT-29.SPEC-002), replace the saved copy with that result as the "last successfully loaded feed" for Nadia's account, kept on her device for viewing.
2. On a refresh failure, check whether a last successfully loaded feed exists.
3. If one exists, show it to the screen in place of a blank or broken result, and signal the screen to show the Error/fallback banner with a Retry control. In this state the feed items' open and toggle controls stay active (see Business Rules).
4. If none exists (the very first load fails), signal the screen to show the Error state with no items and a Retry control -- no fallback content is possible.
5. On a connectivity-degraded/offline signal while the screen is open, stop attempting any new load, keep showing the last successfully loaded feed, and signal the screen to enter the Offline/Degraded state (read-only; every toggle and item-open control becomes inert).
6. On a screen-opened-while-already-offline signal, make no load attempt. If a last successfully loaded feed exists, signal the screen to enter the Offline/Degraded state showing it (banner "You're offline -- showing your last loaded notifications."). If none exists, signal the screen to enter the Offline/Degraded state with no items and the banner "You're offline -- notifications will load when you reconnect." with no Retry control (a manual retry cannot succeed while offline).
7. On a connectivity-restored signal, attempt exactly one fresh feed load automatically; on success, replace the saved copy and clear the offline/fallback banner; on failure, return to the fallback/error handling in step 3 (or step 4 if no saved copy exists).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Fallback served | Refresh fails and a last-successful feed exists | None (no write) | Error/fallback banner and Retry control; previous feed content shown; item-open and toggle controls remain active | FEAT-29.SPEC-001, FEAT-29.SPEC-003 |
| Error, no fallback available | Refresh fails and no last-successful feed exists yet | None | Error state, no items, Retry control (no item controls exist because no items are shown) | FEAT-29.SPEC-001 |
| Offline continuity engaged | Connectivity degrades or drops while viewing | None | Offline/Degraded banner; last loaded feed shown read-only; every mutation and item-open control inert | FEAT-29.SPEC-001, FEAT-29.SPEC-003 (its triggers become inert in this state) |
| Opened while offline, saved copy exists | Nadia opens the screen while the device is already offline and a last-successful feed exists | None | Offline/Degraded banner "You're offline -- showing your last loaded notifications."; last loaded feed shown read-only; every mutation and item-open control inert; no load attempted | FEAT-29.SPEC-001, FEAT-29.SPEC-003 (its triggers are inert) |
| Opened while offline, no saved copy | Nadia opens the screen while the device is already offline and no last-successful feed exists | None | Offline/Degraded banner "You're offline -- notifications will load when you reconnect." with no items and no Retry control; no load attempted | FEAT-29.SPEC-001 |
| Reconnection success | Connectivity restored and the automatic reload succeeds | Saved copy replaced with the fresh result | Offline/fallback banner clears; feed shows current content; controls become active | FEAT-29.SPEC-001 |
| Reconnection failure | Connectivity restored but the automatic reload still fails | None | Returns to the fallback/error handling (Error/fallback banner and Retry) with the existing saved copy, or the no-fallback Error state if none exists | FEAT-29.SPEC-001 |

## Data Model

**Reads:** The saved "last successfully loaded feed" result (itself composed from Notification and Activity Log Entry per FEAT-29.SPEC-002), and each item's Feed Item Read State.
**Creates:** None -- the saved copy is a copy of an already-composed result, not a new entity.
**Updates:** None to any shared entity -- replacing the saved copy is a local operation on Nadia's device, not a write to Notification, Activity Log Entry, or Feed Item Read State.
**Deletes:** None to any shared entity. The saved copy on Nadia's device is discarded when she signs out or her session expires (see Business Rules).

## Business Rules

- The saved fallback copy is always scoped by FEAT-29.SPEC-002's inclusion, window, and ordering rule as it stood at the moment it was captured -- it is never re-windowed or re-ordered while serving as a fallback.
- No read/unread change is ever queued while offline/degraded; FEAT-29.SPEC-003's triggers are simply unavailable in this state, per this feature's Non-Goals.
- Control availability by state: in the Offline/Degraded state (including when the screen was opened while already offline), every feed-item open control and every read/unread toggle is inert and only viewing and scrolling work; in the Error (fallback) state with a saved copy shown, item-open controls and toggles stay active and Retry is available, and a failed write follows FEAT-29.SPEC-003's Failure outcomes; in the Error state with no items, only Retry is available.
- Reconnection always attempts exactly one automatic reload; it does not retry repeatedly on its own -- a continued failure returns Nadia to the ordinary fallback/error state with its own manual Retry.
- The saved copy belongs to Nadia's account only: it is discarded on sign-out and on session expiry, so it is never shown to a different account or after Nadia has signed out.
- The underlying Notification and Activity Log Entry sources are never written by this automation (XBR-04, ASMP-15) -- only a saved copy of an already-composed feed is held and shown.

## Edge Cases

- **First-ever load fails with no prior successful load** -- Error state with no items, since there is nothing to fall back to.
- **Screen opened while already offline** -- No load is attempted; the screen opens directly in Offline/Degraded (with the saved copy if one exists, or an empty state with the reconnect message if none exists). When connectivity is restored while the screen is still open, the single automatic reload runs as in Processing Logic step 7.
- **Connectivity drops mid-load (neither a clear success nor a clear failure yet)** -- Treated as a refresh failure once the load does not complete; falls back to the last successfully loaded feed if one exists, or the no-fallback Error state otherwise.
- **Connectivity is restored and lost again before the automatic reload completes** -- The reload attempt is treated as failed; the screen returns to Offline/Degraded rather than showing a transient Error state.
- **Concurrent trigger firing (a manual Retry tap and an automatic reconnection reload fire at effectively the same time)** -- Only one reload is in flight at a time; a manual Retry tapped while an automatic reload is already running is a no-op -- the in-progress reload's result is used rather than starting a second, redundant reload.
- **A reload fires while a previous reload for the same account is still in flight** -- The later trigger is ignored until the in-flight reload completes; its result is then evaluated against the current connectivity/failure state before deciding the next outcome.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-001 (Notification Center Feed) | Triggered by (inbound) | Refresh failures, screen-opened-while-offline, and connectivity signals from this screen fire this automation |
| FEAT-29.SPEC-001 (Notification Center Feed) | Affects (outbound) | Determines whether the screen shows Error, Offline/Degraded, or a fresh populated feed, and which controls are active |
| FEAT-29.SPEC-002 (Feed Composition & Retention Rule) | References (inbound) | The saved fallback copy's inclusion, window, and ordering rule |
| FEAT-29.SPEC-003 (Mark Item Read/Unread) | Affects (outbound) | This automation's Offline/Degraded outcome makes FEAT-29.SPEC-003's triggers inert |

## Analytics and Success Signals

- **notification_feed_fallback_served** (fallback_reason: refresh_failed; had_saved_feed: true / false) -- N/A — no metric in success-metrics.md names In-App Notification Center as its Connected Feature (verified against every metric entry's Connected Feature field); this event exists to observe reliability of a Nice-to-Have convenience layer outside this run's measured funnel.
- **notification_feed_offline_engaged** (trigger: connectivity_degraded / connectivity_offline / opened_while_offline; had_saved_feed: true / false) -- N/A — same reason.
- **notification_feed_reconnect_reload** (outcome: success / failure) -- N/A — same reason.

## Acceptance Criteria

**FEAT-29.SPEC-004-AC-01:** Given Nadia's feed refresh fails and a previously loaded feed exists, then the screen shows the Error/fallback banner with the previous feed content and a Retry control.

**FEAT-29.SPEC-004-AC-02:** Given Nadia's very first feed load fails with no previous successful load, then the screen shows the Error state with no items and a Retry control.

**FEAT-29.SPEC-004-AC-03:** Given Nadia's connectivity drops while she is viewing a populated feed, then the screen enters Offline/Degraded, showing the last loaded feed read-only with no new load attempted.

**FEAT-29.SPEC-004-AC-04:** Given Nadia is in the Offline/Degraded state, when she taps a feed item's toggle or a feed item row, then nothing happens -- both controls are inert and no navigation or read-state change occurs.

**FEAT-29.SPEC-004-AC-05:** Given connectivity is restored while Nadia is in the Offline/Degraded state, when the automatic reload succeeds, then the offline banner clears and the fresh feed is shown.

**FEAT-29.SPEC-004-AC-06:** Given connectivity is restored and the automatic reload fails, then the screen returns to the Error/fallback state with its own Retry control.

**FEAT-29.SPEC-004-AC-07:** Given a manual Retry tap occurs while an automatic reconnection reload is already in flight, then no second reload starts -- the in-progress reload's result is used.

**FEAT-29.SPEC-004-AC-08:** Given connectivity drops again before an in-progress automatic reload completes, then the screen returns to Offline/Degraded rather than showing a transient Error state.

**FEAT-29.SPEC-004-AC-09:** Given the fallback feed being served was captured under a given rolling window, then it is displayed exactly as captured, never re-windowed while serving as a fallback.

**FEAT-29.SPEC-004-AC-10:** Given Nadia has a last loaded feed and her device is already offline, when she opens the Notification Center, then no load is attempted and the screen shows the banner "You're offline -- showing your last loaded notifications." with the last loaded feed read-only and every item-open and toggle control inert.

**FEAT-29.SPEC-004-AC-11:** Given Nadia has never loaded the feed and her device is already offline, when she opens the Notification Center, then the screen shows the banner "You're offline -- notifications will load when you reconnect." with no items and no Retry control, and when connectivity is restored while the screen is open, one automatic reload runs.

**FEAT-29.SPEC-004-AC-12:** Given the screen is in the Error (fallback) state showing the previous feed, when Nadia taps a feed item row or a toggle, then the tap is processed as in the Populated state (row opens the project and marks the item Read; toggle flips the item's read state).

**FEAT-29.SPEC-004-AC-13:** Given Nadia has signed out or her session has expired, when a different account signs in on the same device and opens the Notification Center while offline, then the previous account's last loaded feed is not shown.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 4 (refresh failure, connectivity degraded/offline, opened while offline, connectivity restored) | 4 |
| Outcome Paths | 7 (fallback served, error no fallback, offline engaged, opened offline with saved copy, opened offline without saved copy, reconnection success, reconnection failure) | 7 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
