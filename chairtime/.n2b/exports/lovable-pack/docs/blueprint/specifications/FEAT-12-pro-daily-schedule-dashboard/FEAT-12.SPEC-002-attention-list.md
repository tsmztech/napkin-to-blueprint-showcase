---
document_type: spec
spec_type: screen
spec_id: FEAT-12.SPEC-002
spec_name: Attention List
spec_slug: attention-list
parent_feature: FEAT-12
parent_feature_name: Pro Daily Schedule Dashboard
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Attention List

## Overview

**Name:** Attention List
**ID:** FEAT-12.SPEC-002
**Type:** Screen
**Purpose:** Surfaces everything needing the Pro's attention -- sync issues, message delivery failures, refunds in progress, card-issuer disputes, bookings left outside changed hours, and waitlist demand -- in one place.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard

## Scope and Non-Goals

**In Scope:**
- Displaying every Open Attention Item aggregated by FEAT-12.SPEC-005
- Displaying aggregate waitlist demand as a separate informational item
- Routing each item's action to the owning feature (FEAT-04, FEAT-28, FEAT-16, FEAT-30)
- The empty state (nothing needs attention) and loading/error/offline states

**Non-Goals:**
- De-duplicating or resolving attention signals -- owned by FEAT-12.SPEC-005 (Attention Flag Aggregation); this screen only displays its output
- Resolving the underlying cause (reconnecting a calendar, retrying a refund, submitting dispute evidence) -- owned by the destination feature (FEAT-04, FEAT-28, FEAT-16); this screen only navigates there
- Showing individual waitlist entries -- excluded per the Access Matrix (Waitlist = View, aggregate counts only for the Pro); showing individual client waitlist requests would expose data the Pro is not entitled to see per-entry
- Letting Support act on any attention item -- excluded per scope-boundaries.md SC-05: Support's access here is View-only

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Pro taps the Attention banner | None -- list loads current Open items |
| FEAT-19.SPEC-001 (Pro Account Lookup & Support Session Entry) | Support taps the Schedule & Bookings entry during an active support session (gated by FEAT-12.SPEC-008) | The Pro account under review; read-only rendering for Support |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen -- own account's attention items only | Tap through to any item's owning feature | -- |
| Platform Operator (Support) | Full screen for the one Pro account under active review; the same items the Pro sees, since none of this screen's content is private client-note material | View only -- can tap through to an owning feature's read-only view where that feature permits Support access; cannot take any write action there either | Any write action reachable from an item is unavailable to Support in the owning feature itself, consistent with that feature's own Access Matrix row |
| The Client (Riley) | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); a failed or absent sign-in never reveals whether a Pro account exists (XBR-29) |
| Expired session | No | No | Redirected to the Pro sign-in screen (FEAT-29) on the next data refresh; no unsaved input exists on this screen |

Authorization governed by FEAT-12.SPEC-008 (Dashboard Access Authorization).

## Layout and Content

**Header:** Screen title "Attention" with a back arrow returning to FEAT-12.SPEC-001.

**Body:** A single vertically scrolling list of attention cards (the shared "attention item" pattern per the Brief's Shared UI Patterns), each showing: cause label (e.g., "Reconnect calendar," "Message delivery gap," "Refund in progress," "Card-issuer dispute," "Booking outside changed hours"), the affected booking's date/time and client name when the cause is booking-specific, or the account-level context when it is not (e.g., calendar reconnection), and a single action button whose label matches the destination (e.g., "Reconnect," "View money list," "Download summary," "Review booking"). Cards are ordered most-recently-detected first.

A separate, visually distinct **waitlist demand card** appears at the top of the list (or, when no other attention items exist, as the sole card) showing an aggregate count of clients waiting for openings (e.g., "4 clients waiting for an opening") with no individual entries -- this card has no action button, since acting on waitlist demand is not a defined capability of this screen (the Pro sees demand only, per the Access Matrix).

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column card list, full width, as described above.
- **Medium size class and above:** The list remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-12.SPEC-001 | Screen closes | Standard navigation transition |
| Attention card -- "Reconnect calendar" | Tap | Navigate to FEAT-04 (Two-Way Calendar Sync) | Screen transitions | Standard navigation transition |
| Attention card -- "Refund in progress" / money action | Tap | Navigate to FEAT-28 (Payout Account Connection & Payout Visibility) | Screen transitions | Standard navigation transition |
| Attention card -- "Card-issuer dispute" | Tap | Navigate to FEAT-16 (Booking & Payment Activity Record) for the dispute flag and evidence summary download | Screen transitions | Standard navigation transition |
| Attention card -- "Message delivery gap" | Tap | Navigate to FEAT-12.SPEC-001's booking row context (no dedicated resolution screen exists for a delivery gap; the Pro reviews the booking and may re-send or contact the client through the booking's own context) | Screen transitions to the relevant booking on FEAT-12.SPEC-001 | Standard navigation transition |
| Attention card -- "Booking outside changed hours" | Tap | Navigate to FEAT-30 (Pro Booking Management) for that booking | Screen transitions | Standard navigation transition |
| Waitlist demand card | Tap | No action -- display-only, non-interactive | None | None (card is visually inert beyond its count display) |
| Pull-to-refresh / manual refresh | Swipe down / tap refresh | Re-fetches the current Open Attention Item set and waitlist demand count | List reloads | Loading indicator during refresh |

### Accessibility Notes

- **Focus order:** Back arrow -> waitlist demand card (when present) -> each attention card in order (most-recently-detected first), each announced with its cause label before its action button.
- **Dynamic-change announcements:** When an item resolves and drops off the list on refresh, the updated count is announced; when a new item appears, it is announced as part of the refreshed list content.
- **Status conveyed beyond color:** Every card's cause label is text, never conveyed by color or icon alone (ASMP-28).
- **Keyboard alternatives:** Every action (navigation, refresh) is reachable without a pointer-only gesture.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Has items | List of attention cards and/or the waitlist demand card as described in Layout and Content | One or more Open Attention Items exist, or waitlist demand is greater than zero | Data changes (refresh, item resolves) |
| Empty (nothing needs attention) | Friendly message "Nothing needs your attention right now" -- shown when there are zero Open Attention Items and zero waitlist demand | Data fetch succeeds with no items and no waitlist demand | An item appears or waitlist demand becomes greater than zero |
| Loading | Lightweight in-place indicator; on first-ever load, a brief full-screen lightweight loading indicator | Screen first opens, or a refresh is triggered | Data fetch completes |
| Error | Error banner "Couldn't load your attention list. Check your connection and try again." with Retry; prior content remains visible below the banner on a refresh failure | Data fetch fails | Retry succeeds, or automatic retry succeeds after reconnection |
| Offline/Degraded | Banner "You're offline -- showing your most recently loaded attention list." at the top; the most recently loaded list remains viewable read-only; tapping a card's action navigates to the destination feature, which applies its own offline handling | Connectivity lost while this screen is open, or screen opened while offline with cached data available | Connectivity restored -- banner clears and a fresh fetch runs automatically |

## Validation Rules

This screen has no user-entry fields; it is a display and navigation surface only. N/A -- no validation rules apply.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | -- |
| "Reconnect calendar" card tap | FEAT-04 (Two-Way Calendar Sync) | FEAT-04 |
| Money-related card tap | FEAT-28 (Payout Account Connection & Payout Visibility) | FEAT-28 |
| Dispute card tap | FEAT-16.SPEC-001 (Booking Activity Timeline) | FEAT-16 |
| Message-delivery-gap card tap | FEAT-12.SPEC-001 (booking row context) | -- |
| Setup-conflict card tap | FEAT-30 (Pro Booking Management) | FEAT-30 |

## Data Model

**Creates:** None.
**Reads:** Open Attention Item set (cause category, affected booking or account reference, first-detected time -- via FEAT-12.SPEC-005); Waitlist Entry (aggregate count only, per Pro Account, via FEAT-20).
**Updates:** None -- this screen never writes; resolution of an item happens through the destination feature and is reflected here only on the next refresh, or through FEAT-12.SPEC-005's own periodic re-check.
**Deletes:** None.

## Business Rules

- Booking-linked cards (sync-reliability and dispute items) show the underlying booking's paid, balance, and reliability values exactly as derived by FEAT-12.SPEC-007 (Balance Due & Status Display Rules), which this screen enforces on every such booking reference and never recomputes.
- Every item shown here is sourced exclusively from FEAT-12.SPEC-005's aggregation -- this screen never computes or de-duplicates a cause itself.
- Waitlist demand is shown as an aggregate count only, never individual entries, per the Access Matrix (Waitlist = View for the Pro).
- Support's access is View-only here, consistent with SC-05 -- Support can navigate to a destination feature from a card only where that feature's own Access Matrix row permits Support's read-only view; no write action is ever available to Support from this screen or through it.
- A resolved item is never shown as an active card -- once FEAT-12.SPEC-005 marks an item Resolved, it disappears from this screen's next load, consistent with XBR-10, XBR-13, XBR-17, XBR-22, and XBR-11's respective resolution definitions.

## Edge Cases

- **All attention items resolve while the Pro is viewing this screen** -- The list does not change mid-view; the Empty state appears only on the next refresh (pull-to-refresh or re-opening the screen), since this is a snapshot view, not a live-updating one.
- **A new attention item appears while the Pro is viewing this screen** -- Similarly not shown until the next refresh; this screen is a snapshot, consistent with the dependency map treating Attention Item aggregation as owned entirely by FEAT-12.SPEC-005's own periodic processing.
- **Waitlist demand count is exactly zero but other attention items exist** -- The waitlist demand card is omitted entirely (not shown with a "0" count), and only the other attention cards appear.
- **Pro taps a card whose destination feature (e.g., FEAT-04) is itself unreachable due to connectivity loss** -- The destination feature's own offline handling applies once navigation completes; this screen's own navigation action itself always succeeds locally (it is a local screen transition, not a network call).
- **Two attention cards reference the same booking with different causes (e.g., a message delivery gap and a dispute)** -- Both cards are shown separately, since FEAT-12.SPEC-005 de-duplicates only within a (booking, cause) pair, not across causes for the same booking.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-12.SPEC-005 (Attention Flag Aggregation) | References (inbound) | Supplies the Open Attention Item set this screen displays |
| FEAT-12.SPEC-001 (Today's & Upcoming Schedule) | Navigation (inbound) | Attention banner navigates here |
| FEAT-12.SPEC-007 (Balance Due & Status Display Rules) | References (outbound) | Derives the paid, balance, and reliability values shown on booking-linked cards |
| FEAT-12.SPEC-008 (Dashboard Access Authorization) | References (inbound) | Governs who may open this screen and what they see |
| FEAT-04 (Two-Way Calendar Sync) | Navigation (outbound) | Reconnect calendar action |
| FEAT-28 (Payout Account Connection & Payout Visibility) | Navigation (outbound) | Money action |
| FEAT-16 (Booking & Payment Activity Record) | Navigation (outbound) | Dispute flag and evidence summary download |
| FEAT-30 (Pro Booking Management) | Navigation (outbound) | Setup-conflict resolution action |
| FEAT-20 (Waitlist for Cancelled Slots) | References (inbound) | Aggregate waitlist demand count shown here |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| attention_list_viewed | open_item_count, waitlist_demand_count | Screen finishes loading | supports success-metrics.md: "Daily Dashboard Glance Speed" |
| attention_item_resolved | cause_category, time_to_resolution | Pro views the list and an item has resolved since it was first detected (the resolution itself is computed by FEAT-12.SPEC-005; this event marks that the Pro's view reflects it) | supports success-metrics.md: "Daily Dashboard Glance Speed" |
| attention_item_action_tapped | cause_category, destination_feature | Pro taps a card's action button | supports success-metrics.md: "Daily Dashboard Glance Speed" |

## Acceptance Criteria

**FEAT-12.SPEC-002-AC-01:** Given Talia has one or more Open Attention Items, when she opens this screen from the dashboard banner, then she sees a card for each item, most-recently-detected first.

**FEAT-12.SPEC-002-AC-02:** Given Talia has zero Open Attention Items and zero waitlist demand, when she opens this screen, then she sees "Nothing needs your attention right now."

**FEAT-12.SPEC-002-AC-03:** Given Talia has 4 clients waiting for an opening, when she opens this screen, then a waitlist demand card shows "4 clients waiting for an opening" with no individual entries and no action button.

**FEAT-12.SPEC-002-AC-04:** Given Talia's Calendar Connection needs reconnection, when she taps that card, then she is navigated to FEAT-04.

**FEAT-12.SPEC-002-AC-05:** Given a booking's deposit is Disputed, when Talia taps the dispute card, then she is navigated to FEAT-16 for the evidence summary download.

**FEAT-12.SPEC-002-AC-06:** Given a refund is in progress for a booking, when Talia taps that card, then she is navigated to FEAT-28's money list.

**FEAT-12.SPEC-002-AC-07:** Given a booking is flagged as outside her changed hours, when Talia taps that card, then she is navigated to FEAT-30 for that booking.

**FEAT-12.SPEC-002-AC-08:** Given the screen's data fetch fails, when the failure occurs, then the error banner "Couldn't load your attention list. Check your connection and try again." appears with a Retry control.

**FEAT-12.SPEC-002-AC-09:** Given Talia loses connectivity while viewing this screen, when the offline state activates, then the banner "You're offline -- showing your most recently loaded attention list." appears and the most recently loaded list remains viewable.

**FEAT-12.SPEC-002-AC-10:** Given an attention item resolves while Talia is actively viewing this screen without refreshing, when she looks at the list, then the resolved item still appears until her next refresh, since this is a snapshot view.

**FEAT-12.SPEC-002-AC-11:** Given Talia refreshes the screen after an item has resolved, when the refresh completes, then the resolved item's card no longer appears.

**FEAT-12.SPEC-002-AC-12:** Given Platform Operator (Support) is viewing this screen during an active support session, when Support looks at the list, then Support sees the same cards Talia would see, since none of this content is private client-note material.

**FEAT-12.SPEC-002-AC-13:** Given Platform Operator (Support) taps a card's action, when the destination feature loads, then no write action is available to Support there either, consistent with that feature's own read-only Access Matrix row.

**FEAT-12.SPEC-002-AC-14:** Given Riley (the Client) attempts to open this screen directly, when the access check runs, then Riley is redirected to the Pro sign-in screen, never seeing any attention data.

**FEAT-12.SPEC-002-AC-15:** Given two attention cards reference the same booking for different causes, when Talia views the list, then both cards are shown separately.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 5 (has items, empty, loading, error, offline) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
