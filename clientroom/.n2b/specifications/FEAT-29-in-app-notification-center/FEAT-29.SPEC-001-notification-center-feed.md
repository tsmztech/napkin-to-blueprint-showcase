---
document_type: spec
spec_type: screen
spec_id: FEAT-29.SPEC-001
spec_name: Notification Center Feed
spec_slug: notification-center-feed
parent_feature: FEAT-29
parent_feature_name: In-App Notification Center
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

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
