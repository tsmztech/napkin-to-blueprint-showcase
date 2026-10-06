---
document_type: spec
spec_type: screen
spec_id: FEAT-29.SPEC-005
spec_name: Account Closure & Reopening Screen
spec_slug: account-closure-reopening-screen
parent_feature: FEAT-29
parent_feature_name: Pro Sign-In & Account Lifecycle
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 15
---

# Screen Spec: Account Closure & Reopening Screen

## Overview

**Name:** Account Closure & Reopening Screen
**ID:** FEAT-29.SPEC-005
**Type:** Screen
**Purpose:** Talia reviews upcoming bookings and confirms closure, or -- during the cooling-off period -- reopens her account with everything intact.
**Parent Feature:** FEAT-29 -- Pro Sign-In & Account Lifecycle

## Scope and Non-Goals

**In Scope:**
- Showing upcoming bookings and offering one-step cancellation with full refunds before closure can proceed
- Confirming account closure and showing the resulting cooling-off state
- Reopening the account during the cooling-off period

**Non-Goals:**
- Executing the cancellation, subscription cancellation, booking-page takedown, and deletion sequencing -- owned by FEAT-29.SPEC-008 (Account Closure Orchestration); this screen only confirms the Pro's intent and displays resulting state
- Executing the bulk cancellation and refunds themselves -- owned by FEAT-30 (Pro Booking Management); this screen hands off to it and shows the resulting booking count, it does not perform the cancellation
- Resuming the booking page after reopening -- owned by FEAT-27 (Pro Profile & Booking Page Settings); reopening restores the account only, the Pro separately resumes bookings through FEAT-27
- Defining the cooling-off period length and retention scope -- owned by FEAT-29.SPEC-013 (Account Closure & Retention Rules), which this screen references for its explanatory copy rather than redefining

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Pro taps "Close account" | None -- fresh review of current upcoming bookings |
| FEAT-29.SPEC-017 (Account Closure & Deletion Notifications) | Talia taps "Manage account" in an account closure/deletion notice email | None -- fresh review of the account's closure state |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Confirm closure (with the cancel-upcoming-bookings step if applicable), or reopen during cooling-off | -- |
| The Client (Riley) | No | No | Clients never reach a Pro settings screen; a client whose booking is cancelled by this flow is notified separately (FEAT-08) but never sees this screen |
| Platform Operator (Support) | No | No | Support's view-only access to account status (FEAT-29.SPEC-003) shows "Closing" as a status value, but never this screen's confirmation flow or upcoming-bookings detail |
| Unauthenticated | No | No | Redirected to FEAT-29.SPEC-001 (Sign-In Screen) per XBR-29 |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- an in-progress closure confirmation (not yet submitted) is discarded; the Pro restarts the review after re-authentication |

## Layout and Content

**Header:** Screen title "Close account" (Active/Paused state) or "Your account is closing" (Closing state), with a back arrow (returns to FEAT-29.SPEC-003).

**Body -- Active/Paused state (pre-closure):**
- If upcoming bookings exist: a list summarizing them (count and next appointment date), with the text "Closing your account cancels these {N} upcoming bookings and refunds every deposit in full." and a single "Cancel bookings and continue" action
- If no upcoming bookings exist: the review step is skipped and the explanatory text and confirmation control (below) render directly
- Explanatory text: "Closing your account cancels your subscription, takes your booking page down, and deletes your data after a {cooling-off period} cooling-off period, during which you can sign back in to reopen it with everything intact."
- A single destructive-styled action "Close my account"

**Body -- Closing state (during cooling-off):**
- A status banner: "{Days remaining} days left in your cooling-off period. Your subscription is cancelled and your booking page is down. Everything else is intact."
- A single primary action "Reopen my account"

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Single-column, full-width text and buttons.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Screen closes | Standard navigation transition |
| "Cancel bookings and continue" (when upcoming bookings exist) | Tap | Hands off to FEAT-30's bulk cancellation with full refunds for every upcoming booking | Upcoming-bookings list is replaced by the explanatory text and "Close my account" action | Confirmation text: "{N} bookings cancelled and refunded" |
| "Close my account" | Tap | Confirmation dialog, then triggers FEAT-29.SPEC-008 (Account Closure Orchestration) | Screen transitions to the Closing state | Confirmation dialog: "This starts your {cooling-off period} cooling-off period. Continue?" with "Continue" and "Keep my account" options |
| "Reopen my account" | Tap | Triggers FEAT-29.SPEC-009 (Account Reopening) | Screen transitions back to the Active/Paused state | Confirmation text: "Your account is reopened" then navigation to FEAT-29.SPEC-003 |
| "Retry" (data-load failure) | Tap | Re-fetches upcoming bookings and Pro Account status | Screen re-attempts the Loading state | Success: screen renders the current state; failure: the Error banner reappears |
| "Retry" (closure-start failure) | Tap | Re-submits the closure request to FEAT-29.SPEC-008 (Account Closure Orchestration) | Button shows loading state | Success: screen transitions to the Closing state; failure: the Error banner reappears with the same message |

### Accessibility Notes

- **Focus order (pre-closure with bookings):** Back arrow -> upcoming-bookings summary -> "Cancel bookings and continue" -> (after) "Close my account".
- **Focus order (Closing state):** Back arrow -> status banner -> "Reopen my account".
- **Confirmation dialog:** The closure confirmation dialog is a focus-trapping modal announced on open, with both options keyboard-reachable.
- **Transition announcements:** The transition from pre-closure to Closing state, and from Closing to reopened, is announced to assistive technology.
- **Error announcement:** The Error banner (data-load failure or closure-start failure) is announced to assistive technology when it appears, and its Retry action receives focus.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Placeholder content shown while upcoming bookings and Pro Account status are fetched | Screen first opens | Data finishes loading (Reviewing/Ready to confirm/Closing, per current status) or the fetch fails (Error) |
| Reviewing upcoming bookings | Bookings list and "Cancel bookings and continue" shown | Loading completes with the account Active/Paused and upcoming bookings existing | Pro completes the cancel-and-continue step |
| Ready to confirm | Explanatory text and "Close my account" shown | Loading completes with no upcoming bookings, or the cancel-and-continue step completes | Pro confirms closure |
| Closing (cooling-off) | Status banner with days remaining and "Reopen my account" shown | Loading completes with the account already Closing, or closure is confirmed (FEAT-29.SPEC-008 starts the cooling-off clock) | Pro reopens, or the cooling-off period expires and the account is deleted (screen is no longer reachable -- see Edge Cases) |
| Error | Banner "Couldn't load your account status and upcoming bookings. Try again." with a Retry action (initial fetch failure); or, after confirming closure, banner "Couldn't close your account. Try again." with a Retry action (FEAT-29.SPEC-008 closure-start failure) | The initial data fetch fails, or FEAT-29.SPEC-008 reports a closure-start failure | Pro taps Retry and it succeeds (screen renders the resulting state); a repeated failure re-shows the same Error banner |
| Offline/Degraded | Existing loaded state remains visible read-only; all action controls are disabled with the inline note "Requires a live connection" | Connectivity lost while screen is open | Connectivity restored -- controls re-enable |

## Validation Rules

Cooling-off period length and closure sequencing are governed by FEAT-29.SPEC-013 (Account Closure & Retention Rules). This screen has no free-text input to validate -- every action is a confirmed, discrete choice.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | -- |
| "Cancel bookings and continue" tap | FEAT-30 bulk cancellation flow, then back to this screen's Ready to confirm state | FEAT-30 (Pro Booking Management) |
| Reopening completes | FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | -- |

## Data Model

**Creates:** None directly.
**Reads:** Booking -- upcoming bookings for this Pro Account (state, service, start_time), to show the pre-closure review list. Pro Account -- status, to determine which state (pre-closure or Closing) to render.
**Updates:** Pro Account.status (Active/Paused -> Closing on confirmation, Closing -> Active on reopening) -- executed by FEAT-29.SPEC-008 and FEAT-29.SPEC-009 respectively; this screen triggers the transition, it does not write the field directly.
**Deletes:** None directly -- permanent deletion is executed by FEAT-29.SPEC-008 once the cooling-off period expires unreversed.

## Business Rules

- Closure with upcoming bookings requires the one-step cancel-and-continue action first (XBR-20) -- "Close my account" is not shown until the bookings list is empty or has been explicitly cleared through that action.
- Every deposit for a booking cancelled through this flow is refunded in full (FEAT-30, XBR-09) -- there is no partial-refund or forfeiture path here regardless of the cancellation policy's ordinary window.
- Reopening restores the account to Active with everything intact except the booking page, which stays down until the Pro separately resumes it through FEAT-27 (per the Feature Breakdown Brief's Cross-Feature Touchpoints).
- The cooling-off period length and what survives deletion are governed entirely by FEAT-29.SPEC-013 -- this screen only displays the resulting state.
- A closure-start failure reported by FEAT-29.SPEC-008 never advances Pro Account.status -- the account remains fully Active/Paused, and this screen's Error state offers Retry rather than any partial-closure display.

## Edge Cases

- **A new booking arrives between opening this screen and confirming closure** -- The upcoming-bookings review is re-evaluated at confirmation time; if a new booking appeared, the Pro is shown the updated list and must repeat "Cancel bookings and continue" for the newly added booking before "Close my account" proceeds.
- **The cooling-off period expires while the Pro is not signed in** -- Data deletion (FEAT-29.SPEC-008) proceeds without this screen being open; the Pro's next sign-in attempt after deletion is treated as a new account, since no account record survives (Non-Goal: no reopening path exists after permanent deletion).
- **Pro taps "Close my account" twice rapidly** -- Second tap is ignored while the confirmation dialog or the closure request is in progress.
- **Pro taps "Reopen my account" twice rapidly** -- Second tap is ignored while the reopening request is in progress.
- **Support views account status while the Pro is on this screen mid-flow** -- No conflict: Support's view is read-only status only; the Pro's own confirmation flow is unaffected.
- **Concurrent-edit conflict -- Pro Account status changed by an automated process (e.g., FEAT-18's subscription-lapse pause) while this screen is open** -- Resolution: reject-with-refresh, per the dependency map's Pro Account Contention note; the screen re-fetches the current status before executing "Close my account" or "Reopen my account" and shows the current state if it changed.
- **The initial data fetch fails repeatedly** -- Each Retry re-attempts the same fetch; there is no retry-count limit or lockout on this screen's Retry action, since fetching read-only review data carries no risk of a duplicate side effect.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-003 (Account & Sign-In Settings Screen) | Navigation (inbound) | Entry point |
| FEAT-30 (Pro Booking Management) | Triggers (outbound) | "Cancel bookings and continue" hands off bulk cancellation with full refunds |
| FEAT-29.SPEC-008 (Account Closure Orchestration) | Triggers (outbound) | "Close my account" starts the closure sequence |
| FEAT-29.SPEC-008 (Account Closure Orchestration) | Affects (inbound) | A closure-start failure reported by FEAT-29.SPEC-008 drives this screen's Error state and Retry action |
| FEAT-29.SPEC-009 (Account Reopening) | Triggers (outbound) | "Reopen my account" restores the account |
| FEAT-29.SPEC-013 (Account Closure & Retention Rules) | References (inbound) | Cooling-off period and retention scope shown in explanatory copy |
| FEAT-27 (Pro Profile & Booking Page Settings) | Navigation (outbound, implied) | Resuming the booking page after reopening happens through FEAT-27, not this screen |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| account_closure_requested | had_upcoming_bookings (yes/no), cancelled_booking_count | Pro confirms "Close my account" | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list |
| account_reopened | days_remaining_at_reopen | Pro confirms "Reopen my account" | N/A -- no success-metrics.md metric traces its Connected Feature to FEAT-29; retained per product-features.md's Signals list |

## Acceptance Criteria

**FEAT-29.SPEC-005-AC-01:** Given Talia has 3 upcoming bookings and opens this screen, when it loads, then she sees those 3 bookings summarized with the text explaining they will be cancelled and refunded, and no "Close my account" action yet.

**FEAT-29.SPEC-005-AC-02:** Given Talia sees her 3 upcoming bookings, when she taps "Cancel bookings and continue", then FEAT-30 cancels and fully refunds all 3, the confirmation "3 bookings cancelled and refunded" appears, and "Close my account" becomes available.

**FEAT-29.SPEC-005-AC-03:** Given Talia has no upcoming bookings, when she opens this screen, then it shows the explanatory text and "Close my account" directly, with no review step.

**FEAT-29.SPEC-005-AC-04:** Given Talia taps "Close my account", when the confirmation dialog appears, then choosing "Continue" starts FEAT-29.SPEC-008 and the screen transitions to the Closing state; choosing "Keep my account" leaves her account unchanged.

**FEAT-29.SPEC-005-AC-05:** Given Talia's account is in the Closing state with 12 days remaining, when she opens this screen, then the banner shows "12 days left in your cooling-off period..." and "Reopen my account" is available.

**FEAT-29.SPEC-005-AC-06:** Given Talia's account is Closing, when she taps "Reopen my account", then FEAT-29.SPEC-009 restores her account to Active, the confirmation "Your account is reopened" appears, and she is navigated to FEAT-29.SPEC-003.

**FEAT-29.SPEC-005-AC-07:** Given Talia's account is reopened, when she checks her booking page, then it remains down until she separately resumes it through FEAT-27.

**FEAT-29.SPEC-005-AC-08:** Given a new booking arrives after Talia opened this screen but before she confirms closure, when she taps "Close my account", then the updated bookings list is shown and she must clear it via "Cancel bookings and continue" before closure proceeds.

**FEAT-29.SPEC-005-AC-09:** Given Talia's cooling-off period has already expired and her data was permanently deleted, when she attempts to sign in again, then no reopening path exists for the deleted account.

**FEAT-29.SPEC-005-AC-10:** Given Talia loses connectivity while on this screen, when the connection drops, then all action controls show "Requires a live connection" and are disabled.

**FEAT-29.SPEC-005-AC-11:** Given Talia taps "Close my account" twice rapidly, when the first tap is already processing, then the second tap has no additional effect.

**FEAT-29.SPEC-005-AC-12:** Given Talia's Pro Account status was changed to Paused by FEAT-18's subscription-lapse automation while this screen was open, when she taps "Close my account", then the screen re-fetches the current status before proceeding and reflects it if it changed.

**FEAT-29.SPEC-005-AC-13:** Given Talia taps the back arrow, when the tap registers, then she is navigated to FEAT-29.SPEC-003 (Account & Sign-In Settings Screen).

**FEAT-29.SPEC-005-AC-14:** Given the initial fetch of Talia's upcoming bookings and account status fails, when the screen attempts to load, then the banner "Couldn't load your account status and upcoming bookings. Try again." appears with a Retry action, and tapping Retry that succeeds renders the correct state (Reviewing, Ready to confirm, or Closing).

**FEAT-29.SPEC-005-AC-15:** Given Talia confirms closure and FEAT-29.SPEC-008 reports a closure-start failure, when the failure is received, then this screen shows "Couldn't close your account. Try again." with a Retry action, her Pro Account.status remains unchanged, and tapping Retry re-submits the closure request.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 6 | 6 |
| States | 5 (loading, reviewing/ready to confirm, closing, error, offline) | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 7 | 7 |
