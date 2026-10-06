---
document_type: spec
spec_type: screen
spec_id: FEAT-20.SPEC-002
spec_name: My Waitlists
spec_slug: my-waitlists
parent_feature: FEAT-20
parent_feature_name: Waitlist for Cancelled Slots
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: My Waitlists

## Overview

**Name:** My Waitlists
**ID:** FEAT-20.SPEC-002
**Type:** Screen
**Purpose:** Riley views her own waitlist entries (position/status), sees the plain empty state when she holds none, and leaves any entry.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots

## Scope and Non-Goals

**In Scope:**
- Listing every active Waitlist Entry belonging to Riley with this Pro, with its current status
- The plain "you're not on any waitlists" empty state
- Leaving (deleting) an entry, including the contention case where a pending Notified claim exists

**Non-Goals:**
- Joining a new waitlist -- owned by FEAT-20.SPEC-001, reached only from a fully booked service; this screen offers no "join" entry point of its own, since a client never navigates to this feature area directly (Brief, Default Entry)
- Establishing Riley's identity -- owned by FEAT-06 (Client Booking Identity); this screen is reached only after FEAT-06's My Bookings List has already matched Riley's phone to her Client record
- Showing which specific clients are waitlisted to the Pro -- excluded per the Access Matrix and this Brief's Non-Goals: the Pro sees only an aggregate count via FEAT-12, never individual entries or this screen

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-003 (My Bookings List) | Riley's My Bookings List renders its Waitlist section (shown only when she has active entries) | Riley's matched Client identity for this Pro (from FEAT-06.SPEC-008) |
| FEAT-20.SPEC-009 (Waitlist Expiry Notification) | Client taps "View my waitlists" in a waitlist expiry notice | The client's access-link identity; list loads that client's own entries with this Pro |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Her own active waitlist entries with this one Pro only | Leave any of her own entries | -- |
| The Pro (Talia) | None -- this screen shows no aggregate or individual waitlist data to the Pro; her View access to waitlist demand is fulfilled by FEAT-12's aggregate count, not this screen | None | This screen is not reachable through any Pro-facing navigation; a Pro account has no path to it |
| Platform Operator (Support) | None on this screen -- Support's View-only access to Waitlist Entry state is surfaced through FEAT-19's own screen, never here | None | Support has no path to this screen; troubleshooting a waitlist entry goes through FEAT-19 |
| Unauthenticated | No | No | Cannot reach this screen without first redeeming a valid access link (FEAT-06.SPEC-002); an unauthenticated visitor is sent to FEAT-06.SPEC-001 to request one |
| Expired session | No | No | The access link this screen depends on has expired per FEAT-06.SPEC-007; Riley sees FEAT-06's "request a new link" prompt and any pending "Leave" action on this screen is not carried over |

## Layout and Content

**Header:** Screen title "My Waitlists," reached as a section within FEAT-06.SPEC-003's My Bookings List rather than a standalone top-level screen; a back element returns to My Bookings.

**Body:** A list of rows, one per active Waitlist Entry:
- Service name
- Requested date or date range
- Status label: "Waiting" (Requested), "A time opened -- claim it" (Notified, with the countdown described below), "Booked" (Converted, shown briefly before the entry rolls off this list per its Data Notes), or "Expired" (shown briefly before rolling off)
- A "Leave" action next to each Requested or Notified row (not shown for Converted or Expired rows, which are terminal and not leaveable)
- For a Notified row: a visible countdown to the claim deadline and a "Claim now" link

**Empty state content:** "You're not on any waitlists" in the same plain, non-alarming tone the product's other empty states use.

### Responsive Behavior

- **Compact breakpoint:** Rows stack vertically, full width, each with its status label directly below the service/date line and the Leave action right-aligned.
- **Medium size class and above:** Uniform scaling, no structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back element | Tap | Navigate to FEAT-06.SPEC-003 (My Bookings List) | Screen closes | Animated transition back |
| Waitlist row (Requested or Expired) | Tap | Display-only -- no detail view beyond what the row already shows | None | -- |
| "Leave" action | Tap | Opens a confirmation dialog "Leave the waitlist for {service}?" | Dialog appears | Dialog with "Leave" and "Cancel" |
| "Leave" confirmed | Tap | 1. Re-check via FEAT-20.SPEC-004 whether the entry has a pending (Notified) claim. 2. Delete the Waitlist Entry immediately regardless of pending-claim state, per FEAT-20.SPEC-004's rule that a leave wins over a pending notification. | Entry removed from the list | Row disappears; toast "You've left the waitlist for {service}." |
| "Leave" cancelled | Tap | Closes the dialog | Dialog closes | No change |
| "Claim now" link (Notified rows only) | Tap | Navigate into FEAT-05's booking flow for the opened slot | Screen closes | Routes to the same destination as the opening notification's claim link (FEAT-20.SPEC-008) |

### Accessibility Notes

- **Focus order:** Back element -> each waitlist row in list order -> each row's Leave (and Claim now, when present) action.
- **Dynamic announcements:** When a row is removed after a confirmed Leave, its removal is announced to assistive technology; the Notified countdown updates are not individually announced (a static remaining-time value is sufficient) to avoid interrupting screen-reader users repeatedly.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty | "You're not on any waitlists" message, no list | Riley has zero active entries with this Pro | An entry is created (via FEAT-20.SPEC-001) and the list is reloaded |
| Populated | List of active entries with status labels and actions | One or more active entries exist | Riley navigates away, or the list changes (leave, notify, convert, expire) |
| Leave confirming | Confirmation dialog open over the list | Riley taps Leave | Riley confirms or cancels |
| Error | Error banner "We couldn't load your waitlists. Try again." with Retry | List fails to load | Riley taps Retry |
| Offline/Degraded | The already-loaded list remains visible read-only; a banner "You're offline -- reconnect to leave a waitlist." appears; Leave and Claim now actions are disabled | Connectivity is lost while this screen is open | Connectivity is restored -- banner clears and actions re-enable |

## Validation Rules

Validation governed by FEAT-20.SPEC-004 (Waitlist Priority & Claim Window Rule) for the leave-vs-pending-claim contention. No field input exists on this screen beyond the Leave confirmation choice.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back element tap | FEAT-06.SPEC-003 (My Bookings List) | FEAT-06 (Client Booking Identity) |
| "Claim now" tap | FEAT-05 booking flow for the opened slot | FEAT-05 (Public Booking Page & Booking Flow) |

## Data Model

**Reads:** Waitlist Entry -- service, date or date range, state (Requested / Notified / Converted / Expired), claim_deadline (for the Notified countdown); scoped to the matched Client with this Pro (per FEAT-06.SPEC-008).
**Updates:** None directly by this screen beyond triggering the delete below.
**Deletes:** Waitlist Entry -- when Riley confirms Leave, the entry is deleted immediately (hard delete, matching the Entity-Lifecycle Coverage Matrix: no restore path, no cascade to any resulting Booking).

## Business Rules

- A leave request always wins over a pending (Notified) claim, per FEAT-20.SPEC-004's contention rule -- Riley is never blocked from leaving because a notification is in flight.
- Converted and Expired entries are retained per the Brief's Non-Goals (no automatic purge) but are not shown indefinitely on this screen -- see Edge Cases for the exact rolling-off behavior this spec defines.
- The Waitlist section on FEAT-06.SPEC-003 only appears at all when at least one active (Requested or Notified) entry exists; this screen's own Empty state is reached only if Riley navigates to it directly with zero entries (e.g., her last entry just left, converted, or expired).

## Edge Cases

- **A cancellation frees a matching slot while Riley has this screen open (entry transitions Requested -> Notified)** -- The row updates in place to the Notified status and countdown the next time the list refreshes; this screen is a snapshot, not live-updating, so the change may not appear until Riley reopens or refreshes it.
- **Riley taps Leave on an entry that has just become Notified (a pending claim exists)** -- The confirmation dialog and outcome are identical regardless of state; the leave is honored and the entry is deleted, per FEAT-20.SPEC-004.
- **Riley's claim window lapses (entry transitions Notified -> Expired) while this screen is open** -- The countdown reaching zero does not itself update the row; the row reflects Expired the next time the list reloads, and the Expired row itself rolls off the list on Riley's next visit after that (see below).
- **An entry converts (another notified client books first, or Riley's own claim succeeds) while this screen is open** -- If Riley's own claim succeeded, she is already mid-navigation into FEAT-05's booking flow and does not return to this exact state; if a different notified client's claim converted, Riley's own remaining entries are unaffected and continue to show their own status.
- **Expired or Converted rows persist beyond the visit in which they last changed** -- This screen shows a terminal (Expired or Converted) row for one visit after the transition so Riley sees the outcome, then it no longer appears in this list on the next load (the record itself is retained per the Brief's Non-Goals; only this screen's display rolls it off).
- **Riley leaves the same entry twice in rapid succession (double-tap on Leave confirmed)** -- The second confirmation is a no-op; the entry is already deleted after the first.
- **Riley's last active entry is left, converted, or expired while this screen is open** -- The list transitions to the Empty state on next reload.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-003 (My Bookings List) | Navigation (inbound) | Riley arrives here from the Waitlist section of My Bookings |
| FEAT-20.SPEC-001 (Join Waitlist) | References (outbound) | Where an entry shown here was originally created |
| FEAT-20.SPEC-004 (Waitlist Priority & Claim Window Rule) | References (outbound) | Governs the leave-vs-pending-claim contention outcome |
| FEAT-20.SPEC-005, FEAT-20.SPEC-006, FEAT-20.SPEC-007 | Affects (inbound) | These automations' state transitions (Notified, Converted, Expired) are what this screen's status labels reflect |
| FEAT-05 (Public Booking Page & Booking Flow) | Navigation (outbound) | "Claim now" routes into the booking flow for the opened slot |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| my_waitlists_viewed | active entry count | Screen loads with at least one active entry | N/A -- no success-metrics.md metric measures waitlist screen views; retained for operational visibility into how often clients check their status |
| waitlist_leave_confirmed | reason: pending_claim / no_pending_claim | Riley confirms Leave on an entry | N/A -- no success-metrics.md metric measures voluntary waitlist departures; retained so the leave path (Key Capability: "Leave a waitlist at any time") is observable, matching FEAT-06.SPEC-003's identical event for its own outbound Leave action |

## Acceptance Criteria

**FEAT-20.SPEC-002-AC-01:** Given Riley has no active waitlist entries with Talia, when she opens this screen, then she sees "You're not on any waitlists."

**FEAT-20.SPEC-002-AC-02:** Given Riley has one Requested entry, when the screen loads, then she sees the service, requested date, and status "Waiting," with a Leave action.

**FEAT-20.SPEC-002-AC-03:** Given Riley has one Notified entry, when the screen loads, then she sees "A time opened -- claim it," a countdown to the claim deadline, a "Claim now" link, and a Leave action.

**FEAT-20.SPEC-002-AC-04:** Given Riley taps "Claim now" on a Notified entry, then she is routed into FEAT-05's booking flow for the opened slot.

**FEAT-20.SPEC-002-AC-05:** Given Riley taps Leave on a Requested entry with no pending claim, when she confirms in the dialog, then the entry is deleted immediately and the row disappears with the toast "You've left the waitlist for {service}."

**FEAT-20.SPEC-002-AC-06:** Given Riley taps Leave on a Notified entry with a pending claim, when she confirms, then the leave still wins per FEAT-20.SPEC-004 -- the entry is deleted immediately regardless of the pending notification.

**FEAT-20.SPEC-002-AC-07:** Given Riley taps Leave and then Cancel in the confirmation dialog, then the dialog closes and the entry remains unchanged.

**FEAT-20.SPEC-002-AC-08:** Given an entry converted to a Booking on Riley's last visit, when she reopens this screen on a later visit, then that row no longer appears (the underlying record is retained, but this screen has rolled it off).

**FEAT-20.SPEC-002-AC-09:** Given the list fails to load, when the screen attempts to render, then Riley sees "We couldn't load your waitlists. Try again." with a Retry option.

**FEAT-20.SPEC-002-AC-10:** Given Riley loses connectivity while this screen is open with entries already loaded, then the list remains visible read-only, a banner appears, and Leave and Claim now are disabled.

**FEAT-20.SPEC-002-AC-11:** Given Riley's access link has expired, when she attempts to reach this screen, then she sees FEAT-06's "request a new link" prompt instead.

**FEAT-20.SPEC-002-AC-12:** Given Talia (the Pro) has no path in her own account to this screen, when she looks for individual waitlist entries, then she finds none -- only FEAT-12's aggregate count is available to her.

**FEAT-20.SPEC-002-AC-13:** Given Riley double-taps the Leave confirmation, then the second tap is a no-op since the entry is already deleted after the first.

**FEAT-20.SPEC-002-AC-14:** Given Riley's last active entry is left, when the confirmation completes, then this screen transitions to the Empty state on next reload.

**FEAT-20.SPEC-002-AC-15:** Given Riley taps the back element, then she returns to FEAT-06.SPEC-003 (My Bookings List).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 5 (empty, populated, leave confirming, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 7 | 7 |
