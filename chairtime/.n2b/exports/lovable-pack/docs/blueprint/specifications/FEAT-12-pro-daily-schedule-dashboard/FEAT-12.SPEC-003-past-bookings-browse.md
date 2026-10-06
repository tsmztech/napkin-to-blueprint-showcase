---
document_type: spec
spec_type: screen
spec_id: FEAT-12.SPEC-003
spec_name: Past Bookings Browse
spec_slug: past-bookings-browse
parent_feature: FEAT-12
parent_feature_name: Pro Daily Schedule Dashboard
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Past Bookings Browse

## Overview

**Name:** Past Bookings Browse
**ID:** FEAT-12.SPEC-003
**Type:** Screen
**Purpose:** The Pro finds and reviews a past booking by browsing by date, using the same booking row pattern as the main schedule.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard

## Scope and Non-Goals

**In Scope:**
- Browsing past bookings organized by date, most recent first
- Rendering the shared booking row pattern (paid badge, balance due, attendance reply, sync-reliability marking) for past bookings, identical to FEAT-12.SPEC-001
- Navigating from a past booking to its full activity timeline (FEAT-16)
- Staying equally responsive as a Pro's booking history grows over multiple years (SC-22)

**Non-Goals:**
- Editing or acting on a past booking (mark completed, no-show, cancel, reschedule) -- all of those actions require the booking to be in an active, non-terminal state; a genuinely past booking has already reached a terminal state (Completed, No-Show, Cancelled, Rescheduled, Expired) and this screen is read-only
- Deriving the balance-due or paid-badge values -- owned by FEAT-12.SPEC-007, reused identically here
- Computing revenue or booking insights from past bookings -- excluded per scope-boundaries.md's deferral notes; that is FEAT-25 (Booking & Revenue Insights, v1), not this screen
- Deleting or archiving past bookings -- excluded per scope-boundaries.md SC-22: booking history is retained for the life of the account for dispute evidence and insights; no delete or archive action exists on this screen

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Pro navigates to "Past Bookings" | None -- list loads defaulting to the most recent past date with bookings |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Schedule & Bookings entry during an active support session (gated by FEAT-12.SPEC-008) | The Pro account under review; read-only rendering for Support |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- own past bookings only | Browse and open a past booking's activity timeline (read-only navigation, no edits) | -- |
| Platform Operator (Support) | Full screen for the one Pro account under active review, except the client private-note preview, which is omitted entirely (FEAT-12.SPEC-008) | Browse only; can open the activity timeline where FEAT-16's own Access Matrix row permits Support | -- |
| The Client (Riley) | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed or absent sign-in never reveals whether a Pro account exists (XBR-29) |
| Expired session | No | No | Redirected to the Pro sign-in screen (FEAT-29) on the next data refresh; no unsaved input exists on this screen |

Authorization governed by FEAT-12.SPEC-008 (Dashboard Access Authorization).

## Layout and Content

**Header:** Screen title "Past Bookings" with a back arrow returning to FEAT-12.SPEC-001, and a date browser control (e.g., a date picker or scrollable date strip) that lets the Pro jump to any past date.

**Body:** A single vertically scrolling list of past dates, most recent first, each date heading followed by that date's bookings using the identical booking row pattern described in FEAT-12.SPEC-001's Layout and Content (start time, service, client name, paid badge, balance due, attendance label, sync-reliability marking, client note preview) -- rendered here as read-only (no quick-action affordances, since every booking here is in a terminal state). Loading additional older dates happens as the Pro scrolls further back or selects an earlier date from the date browser.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column list as described above, full width; the date browser remains accessible from the header without scrolling.
- **Medium size class and above:** The list remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12.SPEC-001 | Screen closes | Standard navigation transition |
| Date browser control | Select a date | Jump the list to the selected date's bookings | List scrolls to or loads that date | List content updates to center on the selected date |
| Booking row -- client name | Tap | Navigate to FEAT-13 (Client Record Management) for that client | Screen transitions | Standard navigation transition |
| Booking row -- client note preview | Tap | Expand the full private note inline on the row | Row expands | Note text becomes fully visible |
| Booking row (elsewhere on the row) | Tap | Navigate to FEAT-16 (Booking & Payment Activity Record) for that booking's full activity timeline | Screen transitions | Standard navigation transition |
| Scroll to top / bottom of loaded range | Scroll | Loads the next older (or more recent) page of dates | List extends | Loading indicator appears briefly at the loaded edge while more history fetches |
| Pull-to-refresh / manual refresh | Swipe down / tap refresh | Re-fetches the currently viewed date range | List reloads | Loading indicator during refresh |

### Accessibility Notes

- **Focus order:** Back arrow -> date browser control -> each date heading and its bookings in order (within a row: time, service, client name, badges, note preview).
- **Dynamic-change announcements:** When the date browser jumps the list to a new date, the new date heading is announced. When older history finishes loading during scroll, no interrupting announcement occurs (content simply extends).
- **Status conveyed beyond color:** Every badge carries a word, never color alone (ASMP-28), consistent with FEAT-12.SPEC-007.
- **Keyboard alternatives:** The date browser, every row tap target, and refresh are all reachable without a pointer-only gesture; infinite-scroll loading has an equivalent "load more" control for keyboard and assistive-technology use.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loaded (has past bookings) | Dates and booking rows as described in Layout and Content | Data fetch for the current date range succeeds with at least one booking | Data changes (date browser selection, further scroll, refresh) |
| Empty (no past bookings yet) | Friendly message "No past bookings yet -- they'll show up here once your first appointment happens" | The Pro Account has zero bookings with a start_time in the past | A booking's start_time passes into the past |
| Empty selected date | When the Pro jumps to a specific date via the date browser and that date has no bookings, a message "Nothing booked on this date" appears for that date only, with the surrounding dates' content (if loaded) still visible | Selected date has zero bookings | Pro selects a different date |
| Loading | Lightweight in-place indicator, at the loaded edge during scroll-triggered pagination, or a brief full-screen indicator on first load | Screen first opens, a refresh is triggered, or the Pro scrolls to the loaded edge | Data fetch completes |
| Error | Error banner "Couldn't load your past bookings. Check your connection and try again." with Retry; already-loaded content remains visible below the banner on a refresh or pagination failure | Data fetch fails | Retry succeeds, or automatic retry succeeds after reconnection |
| Offline/Degraded | Banner "You're offline -- showing your most recently loaded past bookings." at the top; the most recently loaded range remains viewable read-only; the date browser can still be used within already-loaded dates, but jumping to an unloaded date shows the offline banner instead of new content until reconnected | Connectivity lost while this screen is open, or opened while offline with cached data available | Connectivity restored -- banner clears and the requested range loads |

## Validation Rules

This screen has no user-entry form fields and performs no writes. N/A -- no validation rules apply.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | -- |
| Client name tap | FEAT-13 (Client Record Management) | FEAT-13 |
| Booking row tap (elsewhere) | FEAT-16.SPEC-001 (Booking Activity Timeline) | FEAT-16 |

## Data Model

**Creates:** None.
**Reads:** Booking (service, start_time, duration, client reference, price_agreed, deposit_amount, state, attendance_reply, balance_due -- derived by FEAT-12.SPEC-007) for dates in the past; Client (name, private_note preview); Deposit Transaction (status, amount, via FEAT-12.SPEC-007); Calendar Connection (status, via FEAT-12.SPEC-007, for historical reliability marking where applicable); Messaging Consent (state, textability display).
**Updates:** None -- this screen is entirely read-only.
**Deletes:** None.

## Business Rules

- Every booking shown here is, by definition, past its start_time and therefore in a terminal or near-terminal state; this screen never offers the write actions available on FEAT-12.SPEC-001 (mark completed, no-show, reschedule/cancel), since those require an active, non-terminal booking.
- Paid badge, balance due, attendance label, and sync-reliability marking are derived identically to FEAT-12.SPEC-001, via FEAT-12.SPEC-007 -- the same booking presented on both screens shows the same values.
- Booking history is retained for the life of the account (SC-22); this screen's responsiveness must not degrade as that history grows across multiple years, so older history loads incrementally (pagination on scroll) rather than all at once.
- Every screen in this feature, including this one, requires an authorized viewer per FEAT-12.SPEC-008 -- checked before any data loads and on every refresh.

## Edge Cases

- **Pro's account has years of booking history** -- The date browser and incremental loading (pagination on scroll) keep the screen responsive; only the currently viewed date range is fetched at once, consistent with SC-22's retention-without-degradation expectation.
- **Pro selects a date in the future from the date browser** -- The date browser only offers dates up to and including today, since this screen is scoped to past bookings; today's and future bookings are shown on FEAT-12.SPEC-001, not here.
- **A booking's state changes (e.g., an auto-completion sweep resolves it) while the Pro is viewing this screen** -- Since this screen only ever shows bookings whose start_time has passed, a status change here does not remove the booking from view; the row's badge updates to reflect the new state on the next refresh (this is a display update, not a concurrent-edit conflict, since no write is ever attempted from this read-only screen).
- **Pro scrolls rapidly through many months of history** -- Pagination loads sequentially as the scroll reaches each loaded edge; a rapid scroll may show a brief loading indicator at the edge without blocking the already-loaded content above it.
- **Client record referenced by a very old booking has since been deleted (FEAT-13 hard delete)** -- The booking row still shows, since Booking records are retained regardless of Client deletion (XBR-19: financial and timeline records are retained in de-identified form); the client name shows a de-identified placeholder (e.g., "Former client") and no note preview is offered, since the note itself was deleted along with the Client record.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Navigation (inbound) | Past Bookings entry navigates here |
| FEAT-12.SPEC-007 (Balance Due & Status Display Rules) | References (outbound) | Derives the paid badge, balance due, attendance label, and reliability marking shown on every row, identically to FEAT-12.SPEC-001 |
| FEAT-12.SPEC-008 (Dashboard Access Authorization) | References (inbound) | Governs who may open this screen and what they see |
| FEAT-13 (Client Record Management) | Navigation (outbound) | Client name tap navigates here |
| FEAT-16 (Booking & Payment Activity Record) | Navigation (outbound) | Booking row tap navigates to the full activity timeline |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| past_bookings_viewed | date_range_loaded | Screen finishes loading | supports success-metrics.md: "Daily Dashboard Glance Speed" (measures whether the Pro's broader glance at their history stays fast as it grows, per SC-22) |
| past_booking_timeline_opened | -- | Pro taps a booking row to open its activity timeline | N/A -- reason: opening a historical timeline is a low-frequency lookup action with no defined success-metrics.md target of its own; it is tracked for feature-usage visibility rather than tied to a Stage 2 metric |

## Acceptance Criteria

**FEAT-12.SPEC-003-AC-01:** Given Talia has past bookings, when she navigates to this screen from FEAT-12.SPEC-001, then she sees her most recent past date's bookings first, in the same booking row pattern used on the main schedule.

**FEAT-12.SPEC-003-AC-02:** Given Talia has zero bookings in the past, when she opens this screen, then she sees "No past bookings yet -- they'll show up here once your first appointment happens."

**FEAT-12.SPEC-003-AC-03:** Given Talia selects a specific past date with no bookings, when the date browser jumps there, then she sees "Nothing booked on this date" for that date.

**FEAT-12.SPEC-003-AC-04:** Given Talia taps a booking row (outside the client name), when the tap registers, then she is navigated to FEAT-16's full activity timeline for that booking.

**FEAT-12.SPEC-003-AC-05:** Given Talia taps a booking row's client name, when the tap registers, then she is navigated to that client's record (FEAT-13).

**FEAT-12.SPEC-003-AC-06:** Given a past booking's Deposit Transaction status is Captured, when Talia views its row, then the paid badge reads "Paid," matching FEAT-12.SPEC-007's derivation used identically on FEAT-12.SPEC-001.

**FEAT-12.SPEC-003-AC-07:** Given Talia scrolls to the bottom of her currently loaded history, when she continues scrolling, then the next older page of dates loads with a brief loading indicator, without disrupting already-loaded content.

**FEAT-12.SPEC-003-AC-08:** Given Talia's account has multiple years of booking history, when she opens this screen, then only the currently viewed date range is fetched, keeping the screen responsive.

**FEAT-12.SPEC-003-AC-09:** Given this screen's data fetch fails, when the failure occurs, then the error banner "Couldn't load your past bookings. Check your connection and try again." appears with a Retry control.

**FEAT-12.SPEC-003-AC-10:** Given Talia loses connectivity while viewing this screen, when the offline state activates, then the banner "You're offline -- showing your most recently loaded past bookings." appears and already-loaded dates remain viewable.

**FEAT-12.SPEC-003-AC-11:** Given Talia is offline and jumps the date browser to a date outside what is already loaded, when the selection registers, then the offline banner is shown in place of new content until reconnected.

**FEAT-12.SPEC-003-AC-12:** Given a client linked to an old booking has since been deleted, when Talia views that booking's row, then the client name shows a de-identified placeholder and no note preview is offered.

**FEAT-12.SPEC-003-AC-13:** Given Talia looks for a mark-completed, no-show, or reschedule/cancel action on any row on this screen, when she inspects the row, then none of those actions are present, since every booking here is already in a terminal state.

**FEAT-12.SPEC-003-AC-14:** Given Platform Operator (Support) is viewing this screen during an active support session, when Support views a booking row, then no client private-note preview is shown.

**FEAT-12.SPEC-003-AC-15:** Given Riley (the Client) attempts to open this screen directly, when the access check runs, then Riley is redirected to the Pro sign-in screen, never seeing any past-booking data.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 7 | 7 |
| States | 6 (loaded, empty, empty selected date, loading, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
