---
document_type: spec
spec_type: screen
spec_id: FEAT-06.SPEC-003
spec_name: My Bookings List
spec_slug: my-bookings-list
parent_feature: FEAT-06
parent_feature_name: Client Booking Identity
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Screen Spec: My Bookings List

## Overview

**Name:** My Bookings List
**ID:** FEAT-06.SPEC-003
**Type:** Screen
**Purpose:** Client views their own past and upcoming bookings with this one Pro after redeeming an on-demand access link.
**Parent Feature:** FEAT-06 -- Client Booking Identity

## Scope and Non-Goals

**In Scope:**
- Listing the matched Client's upcoming and past bookings with this Pro
- Navigating from a booking row into its detail (FEAT-06.SPEC-004)
- Surfacing an active waitlist entry with a "Leave" action (outbound to FEAT-20)
- A link out to update texting consent and email preferences (FEAT-06.SPEC-005)
- A recurring-series section, shown only when the client holds a series with this Pro, linking to FEAT-21.SPEC-002 (My Recurring Series)

**Non-Goals:**
- Viewing or acting on a single booking in detail -- owned by FEAT-06.SPEC-004
- Cancelling or rescheduling a booking -- owned by FEAT-10 (Client-Initiated Cancel/Reschedule), reached from FEAT-06.SPEC-004
- Browsing any other client's bookings, or this client's bookings with any other Pro -- excluded per scope-boundaries SC-03 and SC-04: cross-pro and cross-client visibility do not exist in this product
- Joining a waitlist -- owned by FEAT-20 (Waitlist for Cancelled Slots); this screen only surfaces an existing entry's "Leave" action
- Viewing, setting up, or cancelling a recurring series -- owned by FEAT-21 (FEAT-21.SPEC-001 sets one up, FEAT-21.SPEC-002 manages it); this screen only links to the series section

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | A valid, unexpired, unused on-demand access link is redeemed | The matched Client's identity for this Pro (established by FEAT-06.SPEC-008) |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Client taps back from a booking's detail | None -- list re-displays as last loaded |
| FEAT-06.SPEC-005 (Consent & Email Preferences) | Client taps back after updating preferences | None -- list re-displays as last loaded |
| FEAT-21.SPEC-002 (My Recurring Series) | Client taps back from the recurring series screen | None -- list re-displays as last loaded |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Client (Riley) | Their own bookings with this one Pro only, past and upcoming | Select a booking to view detail; leave a waitlist entry; navigate to preferences | -- |
| The Pro (Talia) | No | No | This mechanism is not the Pro's own dashboard; the Pro's equivalent view is FEAT-12 (Pro Daily Schedule Dashboard), reached through Pro sign-in (FEAT-29), never through this screen |
| Platform Operator (Support) | No | No | Support access never uses or bypasses client identity (scope-boundaries SC-05); this screen has no support entry point |
| Unauthenticated | No | No | This screen is reachable only immediately after FEAT-06.SPEC-002 redeems a valid link; a direct, unauthenticated attempt to open it is redirected to FEAT-06.SPEC-001 to request a link |
| Expired session | No | No | The redeemed link's viewing session lasts only for the current page (until the tab is closed or reloaded); reloading or returning after the underlying link has already transitioned to Used is treated as an unauthenticated attempt and redirected to FEAT-06.SPEC-001 with the "request a new link" prompt |

## Layout and Content

**Header:** Screen title "My Bookings" with the Pro's display name shown below it, and a "Preferences" link (top-right) to FEAT-06.SPEC-005.

**Body:**
- **Upcoming** section: one row per upcoming booking, each showing service name, date and time, and paid/balance-due status. Tapping a row navigates to FEAT-06.SPEC-004.
- **Waitlist** section (shown only when the client has an active waitlist entry): one row per active entry showing the requested service and date range, with a "Leave" action next to it.
- **Recurring appointments** section (shown only when the client holds at least one Recurring Series with this Pro; no extra UI at all otherwise): one row "Recurring appointments" showing the number of active series, tapping it navigates to FEAT-21.SPEC-002.
- **Past** section: one row per past booking, each showing service name, date, and outcome (completed, no-show, cancelled). Tapping a row navigates to FEAT-06.SPEC-004.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Sections stack vertically in the order Upcoming, Waitlist, Recurring appointments, Past, each full width.
- **Medium size class and above:** Same vertical section order, content column capped at a consistent platform-wide reading width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Preferences" link | Tap | Navigate to FEAT-06.SPEC-005 (Consent & Email Preferences) | Screen transitions | Standard navigation transition |
| Upcoming booking row | Tap | Navigate to FEAT-06.SPEC-004 (Booking Detail via Manage Link) for that booking | Screen transitions | Standard navigation transition |
| Past booking row | Tap | Navigate to FEAT-06.SPEC-004 for that booking | Screen transitions | Standard navigation transition |
| Recurring appointments row | Tap | Navigate to FEAT-21.SPEC-002 (My Recurring Series) | Screen transitions | Standard navigation transition |
| Waitlist "Leave" action | Tap | Confirms intent, then navigates to FEAT-20.SPEC-002 (My Waitlists) within FEAT-20 (Waitlist for Cancelled Slots) to remove the entry | Confirmation prompt appears before the outbound navigation | Confirmation dialog "Leave the waitlist for {service}?" with "Leave" and "Cancel" |

### Accessibility Notes

- **Focus order:** "Preferences" link -> Upcoming rows in date order -> Waitlist row(s) and their "Leave" actions -> Recurring appointments row -> Past rows in date order (most recent first).
- **Dynamic announcements:** When a waitlist entry is removed after confirmation, its row's removal from the list is announced to assistive technology.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | A brief in-place loading indicator where the list will appear | Screen first loads after redemption | Data finishes loading |
| Populated | Upcoming, Waitlist (if any), and Past sections shown with their rows | Data loads successfully with at least one booking | Client navigates away |
| No Upcoming | Upcoming section shows "No upcoming bookings" in place of rows; Past section (if any) still shows | Client has no upcoming bookings with this Pro | A new upcoming booking appears on a future visit to this screen |
| No Past | Past section shows "No past bookings yet" in place of rows; Upcoming section (if any) still shows | Client has no past bookings with this Pro | A booking completes and appears here on a future visit |
| Error | Error banner "We couldn't load your bookings. Try again." with a retry action, in place of the list | The initial data load fails | Client taps Retry and the load succeeds |
| Offline/Degraded | The already-loaded list remains visible read-only; a banner "You're offline -- reconnect to view details or leave a waitlist." appears; row taps and the "Leave" action are disabled | Connectivity is lost while this screen is open | Connectivity is restored -- the banner clears and actions re-enable |

## Validation Rules

Not applicable -- this screen has no user input fields; validation of the access that brought the client here is governed by FEAT-06.SPEC-002 (Access Link Validation & Redemption) and FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| "Preferences" link tap | FEAT-06.SPEC-005 (Consent & Email Preferences) | -- |
| Booking row tap (upcoming or past) | FEAT-06.SPEC-004 (Booking Detail via Manage Link) | -- |
| Recurring appointments row tap | FEAT-21.SPEC-002 (My Recurring Series) | FEAT-21 (Recurring Appointments) |
| Waitlist "Leave" confirmed | FEAT-20.SPEC-002 (My Waitlists), waitlist removal | FEAT-20 (Waitlist for Cancelled Slots) |

## Data Model

**Creates:** None.
**Reads:** Booking -- service, start time, price/deposit status, and state (for outcome labels), scoped to the matched Client with this Pro (FEAT-06.SPEC-008). Waitlist Entry -- service, date range, and state (Requested/Notified), scoped to the same Client. Recurring Series -- only a count of the matched Client's active series with this Pro, to decide whether the Recurring appointments section is shown.
**Updates:** None directly -- the "Leave" action's actual removal is performed by FEAT-20.
**Deletes:** None directly.

## Business Rules

- Every booking and waitlist entry shown is scoped to exactly the Client record matched by FEAT-06.SPEC-008 for this one Pro -- no cross-client or cross-Pro data can ever appear here.
- Access to this screen exists only as the outcome of a successful redemption by FEAT-06.SPEC-002; there is no independent sign-in to reach it.
- XBR-18: this screen's entire contents are scoped to the client identified by the redeemed access link.

## Edge Cases

- **Client leaves this screen open and a booking's status changes on the Pro's side in the meantime (e.g., Talia marks it completed)** -- The already-loaded row keeps showing its state as of load time; the current state is fetched fresh when the client taps into FEAT-06.SPEC-004, so no stale action is ever taken against an out-of-date booking. This screen performs no writes, so no concurrent-edit conflict applies to it directly.
- **Client has both zero upcoming and zero past bookings** -- Both sections show their respective empty messages; this is a rare state (a matched Client record implies at least one historical booking) but is handled without an error.
- **Client taps a Past booking that was later deleted from view due to a client-record deletion elsewhere (FEAT-13)** -- Not applicable in practice: FEAT-13's client deletion is refused while an upcoming booking exists, and this screen's session ends when the access link that produced it expires; a client viewing this list mid-session before their own record is deleted continues to see the data as loaded.
- **Client double-taps a booking row** -- The second tap is ignored while the navigation to FEAT-06.SPEC-004 is already in progress.
- **Client reloads the page after the link has already been marked Used** -- Treated as an unauthenticated attempt (per Access and Visibility): the client is redirected to FEAT-06.SPEC-001 with the "request a new link" prompt rather than seeing a broken or empty list.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-06.SPEC-002 (Access Link Validation & Redemption) | Navigation (inbound) | A valid on-demand redemption routes here |
| FEAT-06.SPEC-008 (Client Identity & Privacy Isolation Rule) | References (inbound) | Scopes every row shown to the matched Client with this Pro |
| FEAT-06.SPEC-004 (Booking Detail via Manage Link) | Navigation (outbound) | Selecting a booking opens its detail |
| FEAT-06.SPEC-005 (Consent & Email Preferences) | Navigation (outbound) | The "Preferences" link opens the client's own settings |
| FEAT-21.SPEC-002 (My Recurring Series) | Navigation (outbound/inbound) | The Recurring appointments row opens the client's series; its back arrow returns here |
| FEAT-20.SPEC-002 (My Waitlists) -- within FEAT-20 (Waitlist for Cancelled Slots) | Navigation (outbound) | The "Leave" action removes a waitlist entry there |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| bookings_list_viewed | upcoming_count, past_count | Screen finishes loading with data | supports success-metrics.md: "Self-Service Access Success" |
| bookings_list_load_failed | -- | The initial data load fails | supports success-metrics.md: "Self-Service Access Success" |
| waitlist_leave_confirmed | -- | Client confirms leaving a waitlist entry from this screen | N/A -- no Stage 2 metric measures waitlist activity from this feature; retained so the action's use here is observable rather than invisible. |

## Acceptance Criteria

**FEAT-06.SPEC-003-AC-01:** Given Riley has just redeemed a valid on-demand access link, when the My Bookings List loads, then she sees her upcoming and past bookings with Talia only, and no bookings from any other Pro or client.

**FEAT-06.SPEC-003-AC-02:** Given Riley is on the My Bookings List with one upcoming booking, when she taps that booking's row, then she is taken to its detail on FEAT-06.SPEC-004.

**FEAT-06.SPEC-003-AC-03:** Given Riley has no upcoming bookings with Talia, when the list loads, then the Upcoming section shows "No upcoming bookings" instead of any rows.

**FEAT-06.SPEC-003-AC-04:** Given Riley has an active waitlist entry for a service, when the list loads, then a Waitlist section appears showing that entry with a "Leave" action.

**FEAT-06.SPEC-003-AC-05:** Given Riley taps "Leave" on her waitlist entry, when she confirms in the dialog, then she is taken to FEAT-20 to complete the removal.

**FEAT-06.SPEC-003-AC-06:** Given Riley taps "Preferences" from the My Bookings List, when the tap registers, then she is taken to FEAT-06.SPEC-005 (Consent & Email Preferences).

**FEAT-06.SPEC-003-AC-07:** Given the initial load of Riley's bookings fails, when the failure occurs, then an error banner "We couldn't load your bookings. Try again." appears with a retry action.

**FEAT-06.SPEC-003-AC-08:** Given Riley loses connectivity while viewing her already-loaded list, when connectivity drops, then the list remains visible read-only, a banner explains she is offline, and row taps and the "Leave" action are disabled.

**FEAT-06.SPEC-003-AC-09:** Given Riley's connectivity is restored after being offline on this screen, when connectivity returns, then the offline banner clears and row taps and the "Leave" action re-enable.

**FEAT-06.SPEC-003-AC-10:** Given Riley's on-demand access link has already transitioned to Used, when she reloads this screen directly, then she is redirected to FEAT-06.SPEC-001 with the "request a new link" prompt rather than seeing the list again.

**FEAT-06.SPEC-003-AC-11:** Given Riley holds an active recurring series with Talia, when the My Bookings List loads, then a "Recurring appointments" row appears, and tapping it takes her to FEAT-21.SPEC-002 (My Recurring Series).

**FEAT-06.SPEC-003-AC-12:** Given Riley holds no recurring series with Talia, when the My Bookings List loads, then no Recurring appointments section or row appears.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 5 (loading, no upcoming, no past, error, offline) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
