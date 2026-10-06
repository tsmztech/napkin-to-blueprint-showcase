# Feature Specification: In-App Notification Center

**Blueprint feature:** FEAT-29
**Priority tier:** Nice-to-Have
**Build order:** 028 of 33
**Depends on:** FEAT-13, FEAT-14
**Blueprint source:** `docs/blueprint/specifications/FEAT-29-in-app-notification-center/`

## User Scenarios & Testing (mandatory)

### User Story 1 - Notification Center Feed (Priority: P3)

Nadia views a chronological feed of recent activity across all her clients, triages items as read or unread, and opens any item to reach its related project.

**Acceptance Scenarios:**

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

### User Story 2 - Feed Composition & Retention Rule (Priority: P3)

Defines which events from Notification and Activity Log Entry populate the feed, their merged order, the rolling recent-window limit, and each item's default read state.

**Acceptance Scenarios:**

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

### User Story 3 - Mark Item Read/Unread (Priority: P3)

Records a feed item's read/unread state when Nadia opens it or explicitly triages it, without altering the underlying Notification or Activity Log Entry.

**Acceptance Scenarios:**

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

### User Story 4 - Feed Load Fallback & Offline Continuity (Priority: P3)

Falls back to the last successfully loaded feed on a failed refresh, and keeps that last-loaded feed viewable read-only when the connection is degraded or offline, including when Nadia opens the screen while already offline.

**Acceptance Scenarios:**

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

### Edge Cases

- **FEAT-29.SPEC-001 (Notification Center Feed):** The Loading indicator stays until the entire in-window feed loads and the list renders complete on first paint. An account-level item with no project marks Read and shows a not-linked-to-a-project toast, a missing project record leaves the freelancer on the feed, and a failed read-state write still navigates but reverts the indicator to Unread with a toast. Source: `docs/blueprint/specifications/FEAT-29-in-app-notification-center/FEAT-29.SPEC-001-notification-center-feed.md` (section: Edge Cases)
- **FEAT-29.SPEC-002 (Feed Composition & Retention Rule):** A candidate item whose underlying record no longer exists is silently excluded, ties on timestamp break by a stable secondary order, and an event exactly at the retention window boundary is still included (inclusive). A Notification and an Activity Log Entry describing the same business event appear as separate feed items with their own read states. Source: `docs/blueprint/specifications/FEAT-29-in-app-notification-center/FEAT-29.SPEC-002-feed-composition-retention-rule.md` (section: Edge Cases)
- **FEAT-29.SPEC-003 (Mark Item Read/Unread):** Opening an already Read item is a no-op with navigation proceeding, double taps on the toggle are ignored while that item's write is in flight, and concurrent changes from two sessions are last-write-wins. Runs for different items proceed independently. Source: `docs/blueprint/specifications/FEAT-29-in-app-notification-center/FEAT-29.SPEC-003-mark-item-read-unread.md` (section: Edge Cases)
- **FEAT-29.SPEC-004 (Feed Load Fallback & Offline Continuity):** A first-ever load failure shows the Error state with no items, opening offline goes straight to Offline/Degraded (saved copy if one exists), and a connectivity drop mid-load is treated as a refresh failure falling back to the last successful feed. Losing connectivity again before the automatic reload completes returns to Offline/Degraded rather than a transient Error. Source: `docs/blueprint/specifications/FEAT-29-in-app-notification-center/FEAT-29.SPEC-004-feed-load-fallback-offline-continuity.md` (section: Edge Cases)

## Requirements (mandatory)

### Functional Requirements

- **FR-001**: The system MUST implement **FEAT-29.SPEC-001** (Notification Center Feed) as specified: Nadia views a chronological feed of recent activity across all her clients, triages items as read or unread, and opens any item to reach its related project. Full spec: `docs/blueprint/specifications/FEAT-29-in-app-notification-center/FEAT-29.SPEC-001-notification-center-feed.md`
- **FR-002**: The system MUST implement **FEAT-29.SPEC-002** (Feed Composition & Retention Rule) as specified: Defines which events from Notification and Activity Log Entry populate the feed, their merged order, the rolling recent-window limit, and each item's default read state. Full spec: `docs/blueprint/specifications/FEAT-29-in-app-notification-center/FEAT-29.SPEC-002-feed-composition-retention-rule.md`
- **FR-003**: The system MUST implement **FEAT-29.SPEC-003** (Mark Item Read/Unread) as specified: Records a feed item's read/unread state when Nadia opens it or explicitly triages it, without altering the underlying Notification or Activity Log Entry. Full spec: `docs/blueprint/specifications/FEAT-29-in-app-notification-center/FEAT-29.SPEC-003-mark-item-read-unread.md`
- **FR-004**: The system MUST implement **FEAT-29.SPEC-004** (Feed Load Fallback & Offline Continuity) as specified: Falls back to the last successfully loaded feed on a failed refresh, and keeps that last-loaded feed viewable read-only when the connection is degraded or offline, including when Nadia opens the screen while already offline. Full spec: `docs/blueprint/specifications/FEAT-29-in-app-notification-center/FEAT-29.SPEC-004-feed-load-fallback-offline-continuity.md`

### Key Entities

- Notification (read — surfaces existing records)

## Success Criteria (mandatory)

### Measurable Outcomes

- **SC-001**: Notification center opens, items marked read and items opened are each observable as distinct signals (notification_center_opened, notification_marked_read, notification_item_opened); no metric in the success-metrics register connects to this feature, so the outcome is grounded in its Signals alone. Source: `docs/blueprint/features/product-features.md`

## Assumptions

- **ASMP-27**: Every screen shows real progress while loading and says plainly when an action needs a connection. Full register: `docs/blueprint/features/assumptions-constraints.md`
- **ASMP-01**: Freelancers work primarily from a laptop or desktop with reliable internet access. Full register: `docs/blueprint/features/assumptions-constraints.md`
