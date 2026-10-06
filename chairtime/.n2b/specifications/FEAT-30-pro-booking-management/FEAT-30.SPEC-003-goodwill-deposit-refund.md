---
document_type: spec
spec_type: screen
spec_id: FEAT-30.SPEC-003
spec_name: Goodwill Deposit Refund
spec_slug: goodwill-deposit-refund
parent_feature: FEAT-30
parent_feature_name: Pro Booking Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Goodwill Deposit Refund

## Overview

**Name:** Goodwill Deposit Refund
**ID:** FEAT-30.SPEC-003
**Type:** Screen
**Purpose:** Talia confirms a full goodwill refund on a booking's deposit, reached from a no-show prompt, a dispute timeline, or a booking row, available any time before the booking completes.
**Parent Feature:** FEAT-30 -- Pro Booking Management

## Scope and Non-Goals

**In Scope:**
- Showing the deposit outcome (a full refund) before Talia confirms, from any of this action's three entry points
- Capturing an optional private note explaining the goodwill decision
- Triggering the commit and reflecting its success, in-progress, rejection, or failure

**Non-Goals:**
- Deciding whether a goodwill refund is warranted -- excluded per scope-boundaries.md (SC-17): the decision is entirely Talia's own judgment; this screen presents the outcome and captures her confirmation, never an automated recommendation
- Marking or undoing a no-show -- owned by FEAT-11 (No-Show Marking & Deposit Forfeiture); this screen is one of the paths reachable from that feature's prompt, but is a distinct action
- Executing the refund against the payment-processing capability -- owned by FEAT-30.SPEC-011, triggered through FEAT-30.SPEC-009 (Goodwill Refund Commit)

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-11 (No-Show Marking & Deposit Forfeiture) | Talia chooses "Refund as goodwill instead" from the no-show prompt | Booking reference |
| FEAT-16.SPEC-001 (Booking Activity Timeline) (Booking & Payment Activity Record) | Talia decides, from a dispute's timeline, to refund as goodwill | Booking reference |
| FEAT-12 (Pro Daily Schedule Dashboard) | Talia taps a booking row and chooses "Refund as goodwill" | Booking reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen, for bookings she owns only | Confirm the goodwill refund, or back out with nothing changed | -- |
| The Client (Riley) | No | No | No control on any Client-facing surface reaches this screen |
| Platform Operator (Support) | Full screen, read-only, reached only through FEAT-19's account view | View only | Confirm control is not shown, consistent with SC-05 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an unconfirmed note is discarded, since no refund has been submitted yet |

## Layout and Content

**Header:** Back arrow (returns to the entry point spec) with the title "Refund as goodwill."

**Body:** A summary of the booking -- client name, service, date and time, deposit amount, and its current status (kept / captured, as applicable) -- followed by a plainly worded outcome statement: "{client_name}'s {deposit_amount} deposit will be refunded in full." An optional single-line text field labeled "Note (private, not shared with {client_name})." Below that, a "Refund as goodwill" confirm button and a "Never mind" link to back out.

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide form width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate back to the entry point (FEAT-11, FEAT-16, or FEAT-12) | Screen closes | Standard backward transition |
| Note field | Type | Captures optional private text | Field shows entered text | Standard input focus state |
| "Refund as goodwill" button | Tap | Triggers FEAT-30.SPEC-006 eligibility re-check, then FEAT-30.SPEC-009 (Goodwill Refund Commit) | Button shows loading state during commit | Success (immediate): confirmation shown. Success (in progress): "in progress" confirmation shown. Rejection: exact denial message shown inline. Failure: retry prompt shown |
| "Refund as goodwill" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |
| "Never mind" link | Tap | Navigate back to the entry point with nothing changed | Screen closes | Standard backward transition |

### Accessibility Notes

- **Focus order:** Back arrow -> booking summary -> outcome statement -> Note field -> Refund as goodwill button -> Never mind link.
- **Announcements:** The outcome statement and any denial or in-progress message are announced to assistive technology when they appear.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|-----------------|
| Loading | A neutral loading placeholder in place of the booking summary and outcome statement | Screen opens | Booking and deposit data load successfully or a load error occurs |
| Ready | Booking summary, outcome statement, and confirm/back-out controls shown | Booking and deposit data load successfully and the eligibility check passes (Deposit Transaction Captured or Forfeited, Booking not Completed) | Talia confirms or backs out |
| Ineligible | The exact denial message from FEAT-30.SPEC-006 shown in place of the confirm control | Booking and deposit data load successfully but the eligibility check fails | Talia backs out (only path forward) |
| Committing | Confirm button shows a loading state, all inputs disabled | Talia taps "Refund as goodwill" | Commit completes (success, in-progress, rejection, or failure) |
| Committed | Confirmation shown ("Refund sent. {client_name}'s deposit is on its way back.") before returning to the entry point | FEAT-30.SPEC-009 reports the refund completed immediately | Talia is navigated back to the entry point |
| Committed -- in progress | Confirmation shown ("Refund started. It will complete automatically -- you don't need to do anything.") before returning to the entry point | FEAT-30.SPEC-009 reports the refund cannot complete immediately | Talia is navigated back to the entry point; the dashboard attention flag (FEAT-12) persists until resolved |
| Rejected | Inline message showing the deposit's current, already-resolved status | FEAT-30.SPEC-009 reports the deposit is no longer refundable | Talia backs out |
| Error | Inline error message "Couldn't load this booking. Try again." or "Couldn't process this refund. Try again." with a Retry action | The booking or deposit data fails to load, or FEAT-30.SPEC-009 reports a processing failure | Talia taps Retry and it succeeds, or she backs out |
| Offline/Degraded | Banner "Check your connection and try again." replaces the confirm control; the booking summary remains viewable read-only | Connectivity is lost while the screen is open | Connectivity is restored and Talia can confirm |

## Validation Rules

Eligibility (once-only limit, until-completion window, ownership) governed by FEAT-30.SPEC-006 (Pro Booking Action Rules). See that spec for the exact conditions and denial messages. This screen applies the eligibility check on entry and again on confirm.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|-------------------------------------|
| Back arrow tap | The entry point spec (FEAT-11, FEAT-16, or FEAT-12) | Varies by entry point |
| "Never mind" tap | The entry point spec | Varies by entry point |
| Successful refund (immediate or in-progress) | The entry point spec | Varies by entry point |

## Data Model

**Reads:** Booking -- state; Deposit Transaction -- status, amount.
**Creates:** None directly.
**Updates:** None directly -- delegated entirely to FEAT-30.SPEC-009 on confirm, which updates Deposit Transaction.status.
**Deletes:** None.

## Business Rules

- SC-17: this screen never suggests or adjudicates whether a goodwill refund is warranted -- the decision is entirely Talia's, exercised by reaching this screen and confirming.
- XBR-12: a goodwill refund is available until the booking is completed; this screen's eligibility check enforces that cutoff, per FEAT-30.SPEC-006.
- A goodwill refund is available whether the deposit is currently Captured (offered instead of marking a no-show, or against an unresolved inside-window cancellation) or Forfeited (converting an already-kept deposit into a full refund) -- the outcome statement and confirm flow are identical either way.
- The optional private note is never shown to the client; it is recorded for Talia's own record only.

## Edge Cases

- **Talia reaches this screen from the no-show prompt (FEAT-11) before confirming the no-show mark itself** -- The deposit is at Captured; the refund proceeds as a standard Captured-status goodwill refund with no interaction with FEAT-11's own marking flow, and no no-show mark is ever recorded for this booking.
- **The deposit is refunded automatically by FEAT-09 (an outside-window client cancellation) between Talia opening this screen and her confirm** -- The confirm-time eligibility re-check finds the deposit already Refunded and denies with "This booking's deposit has already been resolved and cannot be refunded again."
- **The booking reaches Completed via the Auto-Completion Sweep between screen open and confirm** -- The confirm-time eligibility re-check denies with "A goodwill refund is no longer available once a booking is completed."
- **Talia taps "Refund as goodwill" twice rapidly** -- The second tap is ignored while the first commit is in progress.
- **Network failure during the commit** -- Error state shown: "Couldn't process this refund. Try again." with Retry; the deposit remains at its prior status.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-006 (Pro Booking Action Rules) | References (inbound) | Once-only and until-completion eligibility check |
| FEAT-30.SPEC-009 (Goodwill Refund Commit) | Triggers (outbound) | Confirm triggers the commit |
| FEAT-11 (No-Show Marking & Deposit Forfeiture) | Navigation (inbound) | One entry point, reached instead of marking a no-show |
| FEAT-16 (Booking & Payment Activity Record) | Navigation (inbound) | Another entry point, reached from a dispute timeline |
| FEAT-12 (Pro Daily Schedule Dashboard) | Navigation (inbound/outbound) | Another entry point and a possible return destination |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-------------------|
| pro_goodwill_screen_opened | entry source (no_show_prompt / dispute_timeline / booking_row) | Screen loads | supports success-metrics.md: "Pro Change Correctness" |
| pro_goodwill_confirmed | outcome (completed / in_progress) | Talia taps "Refund as goodwill" and the commit succeeds | supports success-metrics.md: "Automatic Refund Correctness" |
| pro_goodwill_denied | reason category | Eligibility check denies the action | supports success-metrics.md: "Automatic Refund Correctness" |

## Acceptance Criteria

**FEAT-30.SPEC-003-AC-01:** Given Talia opens this screen for a booking with a Captured deposit that has not completed, when the screen loads, then she sees the plain statement that the client's deposit will be refunded in full.

**FEAT-30.SPEC-003-AC-02:** Given Talia confirms the refund and the execution completes immediately, when the commit succeeds, then she sees "Refund sent. {client_name}'s deposit is on its way back." and returns to the entry point.

**FEAT-30.SPEC-003-AC-03:** Given the refund execution cannot complete immediately, when the commit processes that outcome, then Talia sees "Refund started. It will complete automatically -- you don't need to do anything."

**FEAT-30.SPEC-003-AC-04:** Given Talia reaches this screen from the no-show prompt before marking the no-show, when she confirms the refund, then it processes normally against the Captured deposit with no no-show mark ever recorded.

**FEAT-30.SPEC-003-AC-05:** Given a booking's deposit is already Forfeited from a no-show mark, when Talia opens this screen, then the goodwill action is still offered and eligible.

**FEAT-30.SPEC-003-AC-06:** Given a booking's deposit has already been automatically refunded by FEAT-09, when Talia opens this screen, then she sees the exact denial message from FEAT-30.SPEC-006 and no confirm control is offered.

**FEAT-30.SPEC-003-AC-07:** Given a booking reaches Completed between screen load and Talia's confirm tap, when the confirm-time eligibility check runs, then it denies with the completed-state message.

**FEAT-30.SPEC-003-AC-08:** Given Talia enters a private note and confirms the refund, when the commit succeeds, then the note is recorded for Talia's own record and never shown to Riley.

**FEAT-30.SPEC-003-AC-09:** Given Talia taps "Refund as goodwill" twice rapidly, when the first tap's commit is in progress, then the second tap has no effect until the first resolves.

**FEAT-30.SPEC-003-AC-10:** Given the commit fails due to a processing error, when the failure occurs, then Talia sees "Couldn't process this refund. Try again." with a Retry action.

**FEAT-30.SPEC-003-AC-11:** Given Talia loses connectivity while viewing this screen, when she attempts to confirm, then she sees "Check your connection and try again." and no commit is attempted.

**FEAT-30.SPEC-003-AC-12:** Given Riley (the Client) has no path to this screen, when the product's screens are reviewed for a reachable control, then none exists.

**FEAT-30.SPEC-003-AC-13:** Given Talia reaches this screen from a dispute timeline (FEAT-16) rather than the no-show prompt, when she confirms, then the refund processes identically regardless of entry point.

**FEAT-30.SPEC-003-AC-14:** Given Talia opens this screen, when the booking and deposit details are being fetched, then she sees a loading placeholder in place of the booking summary and outcome statement.

**FEAT-30.SPEC-003-AC-15:** Given the booking or deposit details fail to load, when the load fails, then Talia sees "Couldn't load this booking. Try again." with a Retry action, and no confirm control is offered until it succeeds.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 9 (loading, ready, ineligible, committing, committed, committed-in-progress, rejected, error, offline) | 9 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
