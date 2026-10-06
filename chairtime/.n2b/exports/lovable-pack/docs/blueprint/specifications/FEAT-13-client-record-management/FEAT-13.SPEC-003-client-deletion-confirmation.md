---
document_type: spec
spec_type: screen
spec_id: FEAT-13.SPEC-003
spec_name: Client Deletion Confirmation
spec_slug: client-deletion-confirmation
parent_feature: FEAT-13
parent_feature_name: Client Record Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Client Deletion Confirmation

## Overview

**Name:** Client Deletion Confirmation
**ID:** FEAT-13.SPEC-003
**Type:** Screen
**Purpose:** The Pro requests permanent deletion of a client's record, sees the eligibility check result, and confirms an irreversible delete.
**Parent Feature:** FEAT-13 -- Client Record Management

## Scope and Non-Goals

**In Scope:**
- Running and displaying the deletion eligibility check (FEAT-13.SPEC-006) when the screen opens
- Showing the blocking outcome (an upcoming booking exists) with the current booking and a route to cancel it with full refund
- Showing the eligible outcome with a plain explanation of what deletion does, and requiring explicit confirmation before executing it
- Triggering the deletion automation (FEAT-13.SPEC-004) on confirmation

**Non-Goals:**
- Performing the deletion itself -- executed by FEAT-13.SPEC-004 (Client Deletion Execution); this screen only confirms intent and hands off
- Cancelling the blocking upcoming booking -- handled by FEAT-30 (Pro Booking Management)'s cancel-with-full-refund flow, reached from this screen but owned there
- Restoring or undoing a completed deletion -- excluded per the Brief's Non-Goals: deletion "is not reversible, consistent with 'delete on request' meaning delete"; no undo path exists anywhere in this feature
- Editing the client's contact details or note before deleting -- handled by FEAT-13.SPEC-001 and FEAT-13.SPEC-002; this screen is deletion-only

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-13.SPEC-001 (Client Record Detail) | Pro taps "Delete client" in the overflow menu | Client ID |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Confirm deletion for their own clients only, gated by eligibility (FEAT-13.SPEC-006) | -- |
| The Client (Riley) | No | No | Per XBR-29, redirected to the Pro sign-in screen if this screen's route is reached directly; a Client cannot request or execute their own deletion in-product, per scope-boundaries.md SC-01's client-self-service exclusion -- they request it informally and the Pro performs it here |
| Platform Operator (Support) | No | No | Support cannot perform a deletion on the Pro's behalf, per scope-boundaries.md SC-05; if Support's session attempts this screen's route directly, redirected to the Pro sign-in screen per XBR-29 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); no client data is exposed before authentication |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue."; the deletion has not started and no state is preserved -- the Pro re-initiates from FEAT-13.SPEC-001 after signing back in |

## Layout and Content

**Header:** Screen title "Delete client" with a back arrow (returns to FEAT-13.SPEC-001 without deleting).

**Client Identity Header:** The same identity-summary display used on FEAT-13.SPEC-001 and FEAT-13.SPEC-002 (name, phone, email), per the Brief's Shared UI Patterns, so the Pro can confirm they are deleting the intended client.

**Eligibility Result Area:** Below the identity header, one of two mutually exclusive panels, determined by FEAT-13.SPEC-006's eligibility check, which runs automatically when the screen opens:

- **Blocked panel:** A plain message: "[Client name] has an upcoming booking on [date, time] for [service]. You'll need to cancel it before deleting this client." Below the message, a single button "Cancel booking with full refund," which routes to FEAT-30's cancel-with-full-refund flow for that booking.
- **Eligible panel:** A plain explanation: "Deleting [client name] permanently removes their contact details and your private note. This cannot be undone. Their booking and payment history will be kept, but no longer linked to a name." Below the explanation, a checkbox "I understand this cannot be undone" and a "Delete client" button, inactive until the checkbox is checked.

### Responsive Behavior

- **Compact breakpoint:** Single-column stack (header, identity header, eligibility panel), full width.
- **Medium size class and above:** Content remains single-column, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-13.SPEC-001 (Client Record Detail) without deleting | Screen closes | Standard navigation transition |
| "Cancel booking with full refund" (Blocked panel only) | Tap | Navigate to FEAT-30 (Pro Booking Management)'s cancel-with-full-refund flow for the blocking booking | Screen transitions | Standard navigation transition |
| "I understand this cannot be undone" checkbox (Eligible panel only) | Tap | Toggles the checkbox; activates the "Delete client" button when checked | Checkbox shows checked/unchecked state; button enables/disables accordingly | Standard checkbox feedback |
| "Delete client" button (Eligible panel only) | Tap | Triggers FEAT-13.SPEC-004 (Client Deletion Execution) | Button shows a loading state; screen becomes non-interactive during execution | On completion, navigates the Pro back to the screen they arrived at before opening the client record (FEAT-12 or FEAT-24), with a toast "Client deleted" |
| "Delete client" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> client identity header (read-only) -> eligibility panel content -> (Blocked panel) "Cancel booking with full refund" button, or (Eligible panel) checkbox -> "Delete client" button.
- **Eligibility announcement:** The eligibility result (blocked or eligible) is announced to assistive technology once the check completes.
- **Confirmation announcement:** The "Client deleted" toast is announced on completion.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Checking Eligibility | A brief in-place loading indicator in place of the eligibility panel | Screen opens | The eligibility check (FEAT-13.SPEC-006) returns a result |
| Blocked | Blocked panel shown, as described in Layout and Content | Eligibility check finds an upcoming booking | Pro navigates away, or later returns after that booking is cancelled or completed, re-triggering the eligibility check |
| Eligible | Eligible panel shown, checkbox unchecked, "Delete client" button inactive | Eligibility check finds no upcoming booking | Pro checks the confirmation checkbox, taps "Delete client," or navigates away |
| Ready to Confirm | Eligible panel shown, checkbox checked, "Delete client" button active | Pro checks the confirmation checkbox | Pro taps "Delete client," or unchecks the checkbox, or navigates away |
| Deleting | Screen is non-interactive; "Delete client" button shows a loading indicator | Pro taps "Delete client" | Deletion completes (success) or fails |
| Deletion Error | Error banner: "Could not delete this client. Check your connection and try again." with a Retry action; eligible panel remains, checkbox state preserved | FEAT-13.SPEC-004 reports a failure | Pro taps Retry, or navigates away |
| Offline/Degraded | Banner "You're offline -- deleting a client requires a live connection." at the top; the eligibility panel (if already loaded) remains visible but the "Delete client" and "Cancel booking" actions are disabled | Connectivity lost while the screen is open, or the screen is opened without connectivity | Connectivity restored -- banner clears and actions re-enable |

## Validation Rules

Validation governed by FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule). See that spec for the exact eligibility condition (no upcoming booking) and the confirmation requirement before deletion executes.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-13.SPEC-001 (Client Record Detail) | -- |
| "Cancel booking with full refund" tap | FEAT-30's cancel-with-full-refund flow | FEAT-30 (Pro Booking Management) |
| Successful deletion | The screen the Pro arrived at before opening the client record | FEAT-12 or FEAT-24 (FEAT-24.SPEC-001, Client Search & Filter) |

## Data Model

**Creates:** None.
**Reads:** Client record -- name, phone, email (for the identity header); Booking (read-only) -- checked by FEAT-13.SPEC-006 for an upcoming booking with this client.
**Updates:** None -- this screen only gathers confirmation; the actual write is performed by FEAT-13.SPEC-004.
**Deletes:** None directly -- deletion is executed by FEAT-13.SPEC-004 once this screen collects confirmation.

## Business Rules

- Deletion eligibility (the upcoming-booking block) is governed entirely by FEAT-13.SPEC-006 -- this screen displays but does not re-derive that rule.
- Delete-access (Pro only, own clients only; never Support or Client) is governed by FEAT-13.SPEC-005 (Client Field Validation & Access Rules) and is enforced on screen entry -- this screen enforces but does not restate that rule.
- Deletion is irreversible; the "I understand this cannot be undone" checkbox must be explicitly checked before the "Delete client" button activates, per the Brief's Validation & Limits ("deletion is a deliberate, confirmed action").
- XBR-19: deletion removes contact details and notes, cascades to Messaging Consent, and retains de-identified financial and timeline history -- this screen's Eligible-panel explanation states this plainly before the Pro confirms.

## Edge Cases

- **The blocking booking is cancelled or completed while this screen is open (e.g., from another device)** -- The eligibility result shown is a snapshot from when the screen loaded; the Pro must navigate away and back (or the screen re-checks on the "Cancel booking" flow's return) to see the updated Eligible panel. No live-updating occurs on this screen.
- **A new booking is created for this client (e.g., a return client books again) between the eligibility check and the Pro tapping "Delete client"** -- Per the dependency map's Contention note ("deletion is refused while an upcoming booking exists"), the deletion attempt in FEAT-13.SPEC-004 re-validates eligibility and refuses with refresh if a booking now exists; this screen surfaces that refusal (see FEAT-13.SPEC-006's edge cases for the exact behavior).
- **Pro taps "Delete client" twice rapidly** -- The second tap is ignored while the first deletion is in progress (button in loading state, screen non-interactive).
- **Network failure during deletion** -- Error banner: "Could not delete this client. Check your connection and try again." with a Retry button; no partial deletion occurs (FEAT-13.SPEC-004 is all-or-nothing).
- **Pro navigates away from the Blocked panel without cancelling the booking** -- No confirmation dialog is needed; nothing was changed, so the Pro simply returns to FEAT-13.SPEC-001 with the client record intact.
- **The client record was already deleted from another session before this screen's eligibility check completes** -- The check fails with "This client's record no longer exists. It may have been deleted." and the Pro is returned to the screen they arrived from.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-13.SPEC-001 (Client Record Detail) | Navigation (inbound) | "Delete client" tap arrives here; back arrow returns here |
| FEAT-13.SPEC-006 (Deletion Eligibility & Retention Rule) | References (inbound) | Eligibility check and irreversibility rule applied on screen load and on confirmation |
| FEAT-13.SPEC-005 (Client Field Validation & Access Rules) | References (inbound) | Delete-access authorization (Pro only, own clients only) enforced on screen entry; the Blocked panel, not a bare denial, is shown when ineligible |
| FEAT-13.SPEC-004 (Client Deletion Execution) | Triggers (outbound) | "Delete client" confirmation triggers the deletion automation |
| FEAT-30 (Pro Booking Management) | Navigation (outbound) | "Cancel booking with full refund" routes to that feature's cancellation flow |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| client_deletion_blocked | -- | Eligibility check finds an upcoming booking and the Blocked panel is shown | N/A -- no success-metrics.md metric names this feature directly; this event supports understanding how often the deletion flow is interrupted, but the Brief's Analytics linkage names only client_record_viewed, client_note_added, and client_record_deleted as this feature's contribution |
| client_deletion_confirmed | -- | Pro taps "Delete client" with the confirmation checkbox checked | N/A -- see client_deletion_blocked; the resulting deletion itself is measured by FEAT-13.SPEC-004's client_record_deleted event, which this screen triggers but does not itself emit |

## Acceptance Criteria

**FEAT-13.SPEC-003-AC-01:** Given Talia taps "Delete client" from a client record with no upcoming booking, when the Client Deletion Confirmation screen opens, then the eligibility check runs and the Eligible panel appears with the irreversibility explanation.

**FEAT-13.SPEC-003-AC-02:** Given Talia taps "Delete client" from a client record with an upcoming booking, when the eligibility check runs, then the Blocked panel appears showing that booking's date, time, and service, with a "Cancel booking with full refund" button.

**FEAT-13.SPEC-003-AC-03:** Given Talia is on the Blocked panel, when she taps "Cancel booking with full refund," then she is navigated to FEAT-30's cancel-with-full-refund flow for that booking.

**FEAT-13.SPEC-003-AC-04:** Given Talia is on the Eligible panel with the confirmation checkbox unchecked, when she looks at the "Delete client" button, then it is inactive and cannot be tapped.

**FEAT-13.SPEC-003-AC-05:** Given Talia checks "I understand this cannot be undone," when the checkbox is checked, then the "Delete client" button becomes active.

**FEAT-13.SPEC-003-AC-06:** Given Talia has checked the confirmation checkbox and taps "Delete client," when the tap registers, then FEAT-13.SPEC-004 (Client Deletion Execution) is triggered and the screen enters the Deleting state.

**FEAT-13.SPEC-003-AC-07:** Given the deletion completes successfully, when Talia sees the result, then she is returned to the screen she arrived at before opening the client record with the toast "Client deleted."

**FEAT-13.SPEC-003-AC-08:** Given Talia taps "Delete client" twice in rapid succession, when the second tap registers, then it is ignored while the first deletion is in progress.

**FEAT-13.SPEC-003-AC-09:** Given the deletion fails due to a network error, when the failure occurs, then the error banner "Could not delete this client. Check your connection and try again." appears with a Retry option.

**FEAT-13.SPEC-003-AC-10:** Given a new booking is created for this client between the eligibility check and Talia's confirmation, when she taps "Delete client," then the deletion is refused and this screen surfaces the refresh behavior defined in FEAT-13.SPEC-006.

**FEAT-13.SPEC-003-AC-11:** Given Talia loses connectivity on this screen, when the connection drops, then the banner "You're offline -- deleting a client requires a live connection." appears and both action buttons become disabled.

**FEAT-13.SPEC-003-AC-12:** Given Riley (the Client) attempts to open this screen's route directly, when the access check runs, then Riley is redirected to the Pro sign-in screen, consistent with scope-boundaries.md SC-01 excluding client self-service deletion.

**FEAT-13.SPEC-003-AC-13:** Given Platform Operator (Support) attempts to open this screen's route directly, when the access check runs, then Support is redirected to the Pro sign-in screen -- Support cannot perform a deletion on the Pro's behalf, per scope-boundaries.md SC-05.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 7 (checking eligibility, blocked, eligible, ready to confirm, deleting, error, offline) | 7 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |
