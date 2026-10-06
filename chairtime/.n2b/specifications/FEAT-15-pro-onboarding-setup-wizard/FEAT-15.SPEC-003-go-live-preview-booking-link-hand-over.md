---
document_type: spec
spec_type: screen
spec_id: FEAT-15.SPEC-003
spec_name: Go-Live Preview & Booking Link Hand-Over
spec_slug: go-live-preview-booking-link-hand-over
parent_feature: FEAT-15
parent_feature_name: Pro Onboarding & Setup Wizard
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Screen Spec: Go-Live Preview & Booking Link Hand-Over

## Overview

**Name:** Go-Live Preview & Booking Link Hand-Over
**ID:** FEAT-15.SPEC-003
**Type:** Screen
**Purpose:** Once every required setup step is satisfied, this screen reveals Talia's live shareable booking link with a plain hand-over note on what to do with it -- or, if her payout account is still verifying, shows a "finish verifying to start taking bookings" waiting state instead.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard

## Scope and Non-Goals

**In Scope:**
- The screen Talia lands on once the wizard shell (FEAT-15.SPEC-001) hands off after the final required step
- Displaying the live, shareable booking link once FEAT-15.SPEC-005 has activated it
- The booking-page preview entry point into FEAT-05's preview mode
- The "finish verifying to start taking bookings" waiting state when every other required step is complete but the payout account is still Verification Pending
- The plain "here's what to do with it" hand-over note once the link is live

**Non-Goals:**
- Deciding whether the link may go live -- owned by FEAT-15.SPEC-007 (rule) and FEAT-15.SPEC-005 (automation); this screen only displays the outcome of that decision
- Rendering the actual booking page a client sees -- owned by FEAT-05; this screen only links to it (live) or previews it (FEAT-05's preview mode)
- Resolving a payout verification problem -- owned by FEAT-28's own screens (FEAT-28.SPEC-001, FEAT-28.SPEC-003); this screen only shows the waiting state and a path back into FEAT-28's flow
- Sending the welcome confirmation -- owned by FEAT-15.SPEC-008 (Notification), triggered by the same go-live event this screen displays, delivered outside this screen entirely

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15.SPEC-001 (Setup Wizard Shell) | The final required step completes and FEAT-15.SPEC-005 confirms readiness (or reports the payout condition still pending) | Whether the link is live or the payout-pending waiting state applies, per FEAT-15.SPEC-007's evaluation |
| FEAT-28 (Payout Account Connection & Payout Visibility) | Talia finishes payout verification after having already reached this screen once in the waiting state | Updated payout status; FEAT-15.SPEC-005 re-evaluates and this screen refreshes from waiting to live |
| FEAT-15.SPEC-008 (Onboarding Welcome Confirmation) | Talia taps "View my link" in the go-live welcome confirmation | None -- this Pro's own go-live state and booking link load fresh |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Tap the preview entry point; copy or share the live link once activated; tap through to finish payout verification while waiting | -- |
| The Client (Riley) | No | No | Never reaches this screen; a client who follows the live link itself is sent to FEAT-05's public booking page, not this hand-over screen |
| Platform Operator (Support) | No | No | Support has no reason to open this screen directly; a Pro's go-live status is visible to Support only through the read-only setup-progress view surfaced by FEAT-19 |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29) |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- after re-authentication, the Pro returns to this same screen showing the same live-or-waiting state as before, since nothing here depends on unsaved input |

## Layout and Content

**Header:** Screen title, either "You're live!" (link activated) or "Almost there" (payout verification pending), depending on state.

**Body (Live state):**
- The full shareable booking link, displayed as plain text with a "Copy link" action
- A "Preview my booking page" action, opening FEAT-05's preview mode
- The hand-over note: plain-language guidance on what to do next (e.g., "Add this link to your Instagram bio so clients can book and pay their deposit in under a minute.")

**Body (Waiting state -- payout still Verification Pending):**
- A plain message: "Finish verifying your payout account to start taking bookings." (mirrors FEAT-28's own wording for this exact condition)
- A summary confirming every other step is done ("Everything else is ready -- services, hours, cancellation policy, and your subscription are all set.")
- A "Finish verifying" action, navigating into FEAT-28's verification flow
- The "Preview my booking page" action remains available even while waiting, since preview never requires an active payout account

**Footer:** None -- all actions are in the body.

### Responsive Behavior

- **Compact breakpoint:** Single-column layout as described, full width; the link display wraps rather than truncates so Talia can always read it in full.
- **Medium size class and above:** The link display and the hand-over note render side by side in two columns instead of stacked; no other structural change.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Copy link" (Live state only) | Tap | Copies the booking link to the clipboard | None on screen state | Toast confirmation "Link copied" |
| "Preview my booking page" | Tap | Navigate to FEAT-05's preview mode | None on this screen | FEAT-05 opens in preview mode |
| "Finish verifying" (Waiting state only) | Tap | Navigate into FEAT-28's payout verification flow (FEAT-28.SPEC-001) | None on this screen until Talia returns | FEAT-28's verification screen opens |
| (Automatic) Payout status changes to Active while waiting | System event | FEAT-15.SPEC-005 re-evaluates readiness (FEAT-15.SPEC-007) and, finding it satisfied, activates the link | Screen transitions from Waiting to Live | The "Finish verifying" panel is replaced by the live link and hand-over note; the welcome confirmation (FEAT-15.SPEC-008) is sent independently of this screen's state |

### Accessibility Notes

- **Focus order (Live state):** Screen title -> booking link text -> "Copy link" -> "Preview my booking page" -> hand-over note.
- **Focus order (Waiting state):** Screen title -> summary message -> "Finish verifying" -> "Preview my booking page".
- **Dynamic-change announcements:** The transition from Waiting to Live (when payout verification completes) is announced to assistive technology as a state change, since it can happen without Talia performing an action on this screen (she may have completed verification via a notification link, not by returning here first).
- **Keyboard alternatives:** All actions on this screen are reachable and activatable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Live | Shareable link, Copy action, preview action, and hand-over note shown | FEAT-15.SPEC-005 has activated the booking link (FEAT-15.SPEC-007's conditions all satisfied) | Talia navigates away (this is her setup destination; the wizard is now complete and she typically continues to her daily dashboard, FEAT-12) |
| Waiting (payout pending) | "Finish verifying" panel shown in place of the link; preview action still available | Every other required step is complete but the payout account is Verification Pending | Payout account becomes Active, at which point FEAT-15.SPEC-005 re-evaluates and this screen transitions to Live automatically |
| Loading | N/A -- readiness is evaluated before this screen is reached (by FEAT-15.SPEC-005), so the screen never renders in an intermediate "evaluating" state visible to Talia | -- | -- |
| Error | If the link or status cannot be retrieved on load, the last-known state (live link or waiting message) is shown with the time it was loaded and a Retry action, rather than a blank screen | Status retrieval fails | Talia taps Retry and the current status loads successfully |
| Offline/Degraded | The last-loaded state (live link or waiting message) remains viewable read-only; "Copy link" still works from the cached link text; "Finish verifying" and "Preview my booking page" require connectivity and are shown disabled with a plain explanation until it returns | Connectivity lost while this screen is open | Connectivity restored -- the screen re-checks status and refreshes normally |

## Validation Rules

Validation governed by FEAT-15.SPEC-007 (Go-Live Prerequisite Rule) for whether the link may be Live or must show the Waiting state. This screen performs no field-level input validation of its own -- it has no form fields.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| "Preview my booking page" | Booking page preview mode | FEAT-05 (Public Booking Page & Booking Flow) |
| "Finish verifying" | Payout Account Connection screen (FEAT-28.SPEC-001) | FEAT-28 (Payout Account Connection & Payout Visibility) |

## Data Model

**Creates:** None.
**Reads:** Pro Account's booking_link_name and go-live status (derived by FEAT-15.SPEC-005 from FEAT-15.SPEC-007's evaluation); Payout Account's status field (Active vs. Verification Pending), read from FEAT-28.
**Updates:** None -- this screen never writes go-live status or payout status directly; both are owned by their respective automations (FEAT-15.SPEC-005 and FEAT-28).
**Deletes:** None.

## Business Rules

- The link is shown as Live only when FEAT-15.SPEC-007's Go-Live Prerequisite Rule (XBR-26) is satisfied -- this screen never shows a partially-live or Pro-overridable state, per the feature's own explicit non-goal.
- Every other required step being complete while payout verification is pending is the one named waiting condition this screen defines, per XBR-06 ("no deposit can be taken, and the booking link cannot go live, unless the Pro's payout account is active").
- The preview entry point (FEAT-05's preview mode) is available in both the Live and Waiting states, since it never requires an active payout account or a live link.

## Edge Cases

- **Talia reaches this screen while payout is Verification Pending, then completes verification on a different device without returning here first** -- The next time she opens this screen (or if she has it open and connectivity permits a status refresh), FEAT-15.SPEC-005 has already re-evaluated and activated the link; the screen shows Live directly, never requiring her to retrigger anything from this screen.
- **The processor later flags Talia's payout account as needing action after the link has already gone live** -- Out of scope for this screen: an already-Active payout account that later needs action is handled entirely by FEAT-28's own attention-flow (dashboard banner and notification); this screen's Waiting state applies only during the original setup sequence, before the link has ever gone live.
- **Talia taps "Copy link" while offline, using the last-loaded link text** -- The copy succeeds from cached text since it requires no network call; if the link was never successfully loaded before going offline, the action is shown disabled with a plain "reconnect to copy your link" explanation instead.
- **Talia's booking link renders with the correct name before she has renamed it from the default suggested at account creation** -- This screen displays whatever booking_link_name currently exists on the Pro Account (owned by FEAT-27); renaming it later (FEAT-27) does not require returning to this screen, and the forwarding behavior for a renamed link (XBR-27) is entirely FEAT-27's concern.
- **Concurrent-edit conflict** -- Not applicable: this screen performs no writes of its own; go-live status and payout status are each owned and serialized by their respective automations (FEAT-15.SPEC-005, FEAT-28), so there is no field on this screen a second actor could be editing concurrently.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-001 (Setup Wizard Shell) | Navigation (inbound) | Hands off to this screen once the final required step completes |
| FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) | References (inbound) | Determines whether this screen shows Live or Waiting, and activates the link this screen displays |
| FEAT-15.SPEC-007 (Go-Live Prerequisite Rule) | References (inbound) | Defines the exact condition set this screen's Live/Waiting split is based on |
| FEAT-15.SPEC-008 (Onboarding Welcome Confirmation) | References (inbound, indirect) | Fires from the same go-live event this screen displays, delivered independently of whether Talia is viewing this screen |
| FEAT-28.SPEC-001 (Payout Account Connection) | Navigation (outbound) | "Finish verifying" launches into this screen |
| FEAT-28.SPEC-003 (Payout Account Status Processing) | References (inbound) | Reports the Verification Pending / Active status this screen reads |
| FEAT-05 (Public Booking Page & Booking Flow) | Navigation (outbound) | "Preview my booking page" opens FEAT-05's preview mode; the live link itself points here for clients |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| onboarding_link_shared | share method (copy) | Talia taps "Copy link" | supports success-metrics.md: "Setup-to-Live-Link Completion" |
| onboarding_preview_from_golive_tapped | state at time of tap (live / waiting) | Talia taps "Preview my booking page" from this screen | supports success-metrics.md: "Setup-to-Live-Link Completion" |
| onboarding_payout_wait_shown | none | This screen renders the Waiting state | supports success-metrics.md: "Setup-to-Live-Link Completion" (a payout-pending wait is friction between "began onboarding" and "reached a live link," directly relevant to this metric's completion window) |

## Acceptance Criteria

**FEAT-15.SPEC-003-AC-01:** Given Talia completes her final required setup step and her payout account is already Active, when FEAT-15.SPEC-005 confirms readiness, then this screen shows "You're live!" with her shareable booking link and a hand-over note.

**FEAT-15.SPEC-003-AC-02:** Given Talia is on the Live state, when she taps "Copy link", then the link is copied to her clipboard and a "Link copied" toast appears.

**FEAT-15.SPEC-003-AC-03:** Given Talia completes her final required step but her payout account is Verification Pending, when this screen loads, then it shows "Almost there" with the message "Finish verifying your payout account to start taking bookings." and no live link is shown.

**FEAT-15.SPEC-003-AC-04:** Given Talia is on the Waiting state, when she taps "Finish verifying", then she is navigated into FEAT-28's payout verification flow (FEAT-28.SPEC-001).

**FEAT-15.SPEC-003-AC-05:** Given Talia is on the Waiting state and completes payout verification, when her payout account becomes Active, then this screen transitions automatically from Waiting to Live without requiring her to take any further action on this screen.

**FEAT-15.SPEC-003-AC-06:** Given Talia is on either the Live or Waiting state, when she taps "Preview my booking page", then FEAT-05 opens in preview mode showing the booking page exactly as a client would see it.

**FEAT-15.SPEC-003-AC-07:** Given Talia's booking link goes live, when the transition occurs, then FEAT-15.SPEC-008's welcome confirmation is sent independently, regardless of whether Talia is currently viewing this screen.

**FEAT-15.SPEC-003-AC-08:** Given status retrieval fails when this screen loads, when the failure is detected, then the last-known state is shown with the time it was loaded and a Retry action, rather than a blank screen.

**FEAT-15.SPEC-003-AC-09:** Given Talia loses connectivity while viewing the Live state, when she taps "Copy link", then the copy still succeeds using the cached link text.

**FEAT-15.SPEC-003-AC-10:** Given Talia loses connectivity while viewing the Waiting state, when she taps "Finish verifying", then the action is shown disabled with a plain explanation to reconnect, rather than silently failing.

**FEAT-15.SPEC-003-AC-11:** Given a client (Riley) follows Talia's live booking link, when the link is opened, then Riley lands on FEAT-05's public booking page, never on this hand-over screen.

**FEAT-15.SPEC-003-AC-12:** Given an unauthenticated visitor reaches this screen's URL directly, when the request is made, then they are redirected to the Pro sign-in screen (FEAT-29).

**FEAT-15.SPEC-003-AC-13:** Given Talia's session expires while she is on this screen, when she next interacts with it, then the expiry dialog appears, and after signing back in she returns to the same Live-or-Waiting state she saw before expiry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 5 (live, waiting, loading, error, offline/degraded) | 5 |
| Business Rules | 3 | 3 |
| Edge Cases | 5 | 5 |
