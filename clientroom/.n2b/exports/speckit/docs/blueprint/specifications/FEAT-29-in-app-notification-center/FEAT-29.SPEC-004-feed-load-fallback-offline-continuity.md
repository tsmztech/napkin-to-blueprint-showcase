---
document_type: spec
spec_type: automation
spec_id: FEAT-29.SPEC-004
spec_name: Feed Load Fallback & Offline Continuity
spec_slug: feed-load-fallback-offline-continuity
parent_feature: FEAT-29
parent_feature_name: In-App Notification Center
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

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
