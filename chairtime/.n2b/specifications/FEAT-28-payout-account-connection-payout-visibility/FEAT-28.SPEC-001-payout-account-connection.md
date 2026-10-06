---
document_type: spec
spec_type: screen
spec_id: FEAT-28.SPEC-001
spec_name: Payout Account Connection
spec_slug: payout-account-connection
parent_feature: FEAT-28
parent_feature_name: Payout Account Connection & Payout Visibility
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 18
---

# Screen Spec: Payout Account Connection

## Overview

**Name:** Payout Account Connection
**ID:** FEAT-28.SPEC-001
**Type:** Screen
**Purpose:** Talia launches the payment-processing capability's own secure identity and bank verification flow from the "getting paid" step of setup, and sees the outcome of that handoff (pending, active, or a failed handoff) before continuing.
**Parent Feature:** FEAT-28 -- Payout Account Connection & Payout Visibility

## Scope and Non-Goals

**In Scope:**
- Explaining, in plain words, why Talia is connecting a payout account and that Chairtime never sees her bank or identity details
- Launching the payment-processing capability's own identity and bank verification flow (FEAT-28.SPEC-006)
- Showing the outcome of that handoff (verification pending, active, or the handoff itself failed to start) before Talia continues setup
- Letting Talia continue the rest of onboarding even if verification is still pending

**Non-Goals:**
- Collecting or displaying any bank account number, routing number, or identity document -- excluded per the Data Notes field and ASMP-31: those details are entered directly into the payment-processing capability's own flow and never pass through this screen or the product's own code
- The ongoing status view, money list, and action-required resolution after this first connection -- owned by FEAT-28.SPEC-002 (Payout Status & Money Dashboard); this screen is reached once, on first use, and never again once a Payout Account exists
- Resolving a later "Action Required" flag -- owned by FEAT-28.SPEC-002's banner and FEAT-28.SPEC-006's resolution hand-off; this screen only covers the very first connection attempt during onboarding
- Deciding whether the booking link may go live -- owned by FEAT-15 (Pro Onboarding & Setup Wizard), which reads this feature's status (XBR-06, XBR-26) but makes the go-live decision itself

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-15.SPEC-001 (Setup Wizard Shell) (Pro Onboarding & Setup Wizard), setup step: getting paid | Talia reaches the "getting paid" step of the setup wizard | None -- no Payout Account exists yet for this Pro Account |
| FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) | Talia taps "Finish verifying" on the go-live screen | None -- the existing Payout Account and its current verification status load fresh |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Launch the verification handoff and continue setup once an outcome is shown | -- |
| Platform Operator (Support) | No | No | This screen is reached only from the live onboarding wizard on the Pro's own device; Support's read-only view shows the resulting Payout Account status through FEAT-28.SPEC-002 instead, never this connection screen itself |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29, XBR-29); after signing in, a Pro who has not yet reached this step in setup is returned to FEAT-15 at their current step, not directly to this screen |
| Expired session | No | No | "Your session has expired. Sign in to continue." -- no in-progress handoff exists to preserve, since the identity/bank flow itself runs entirely inside the payment-processing capability, not on this screen |

## Layout and Content

**Header:** Setup wizard's standard step header, showing "Getting paid" as the current step within FEAT-15's overall progress indicator, with a back control to the previous setup step.

**Body:** A single-column explanation panel above one primary action:
- A short paragraph stating why this step exists in plain words: money from every deposit needs somewhere to go, and Chairtime never sees or stores Talia's bank or identity details -- that stays with the payment processor.
- A single primary action: "Connect payout account."
- Once the handoff has been launched at least once, a status region appears below the action, showing the current outcome (Verification Pending, Active, or Handoff Failed) per the States section.
- A secondary text link, "Why do you need my bank details?", which expands an inline explanation (no navigation) covering the same plain-words disclosure as FEAT-28.SPEC-006's Consent and Disclosure section.

**Footer:** A "Continue setup" action, enabled once any outcome (including Verification Pending) has been recorded; disabled before the first handoff attempt.

### Responsive Behavior

- **Compact breakpoint:** Single-column panel as described, full width, primary action full-width beneath the explanation text.
- **Medium size class and above:** Panel content is capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Connect payout account" | Tap | Triggers the outbound handoff via FEAT-28.SPEC-006 to the payment-processing capability's identity and bank verification flow | Screen enters the Launching state | The capability's own flow opens (in-flow or as a full-screen takeover); on return, the status region reflects the reported outcome |
| "Why do you need my bank details?" | Tap | Expands an inline disclosure panel in place | No navigation; panel expands below the link | Disclosure text becomes visible; a second tap collapses it |
| Status region (display-only) | -- | Non-interactive; reflects the current outcome | -- | -- |
| "Continue setup" | Tap | Advances FEAT-15 to its next setup step | Screen closes | Setup wizard advances to the next step (calendar connection) |
| Back control | Tap | Returns to the previous FEAT-15 setup step | Screen closes | Setup wizard shows the previous step (cancellation policy) |

### Accessibility Notes

- **Focus order:** Back control -> explanation text -> "Connect payout account" -> "Why do you need my bank details?" -> status region (when present) -> "Continue setup."
- **Dynamic announcements:** When the status region first appears or its outcome changes (Launching -> Verification Pending / Active / Handoff Failed), the new state is announced to assistive technology.
- **Keyboard alternatives:** Every action on this screen, including launching the external handoff, is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Not Started (default) | Explanation text and "Connect payout account" shown; no status region; "Continue setup" disabled | Talia reaches this step with no prior handoff attempt | She taps "Connect payout account" |
| Launching | "Connect payout account" shows a brief loading state while the handoff is initiated | Tap on "Connect payout account" | The payment-processing capability's flow opens, or the handoff fails to start |
| Verification Pending | Status region shows "Verification pending -- we'll let you know as soon as it's ready." "Continue setup" is enabled | The capability reports the identity/bank verification was submitted but not yet confirmed | The capability later reports Active or Action Required (reflected on FEAT-28.SPEC-002 from that point forward) |
| Active | Status region shows "Payout account connected and active." "Continue setup" is enabled | The capability reports verification is complete | Talia continues setup or leaves the screen |
| Handoff Failed | Status region shows "We couldn't start the connection process. Try again." with a "Try again" action; "Continue setup" remains disabled until a successful handoff attempt is recorded | The handoff itself could not be initiated (see FEAT-28.SPEC-006 Degradation Behavior) | Talia taps "Try again" and the handoff launches successfully |
| Offline/Degraded | Banner "You're offline -- connecting a payout account needs an internet connection." The primary action is disabled until connectivity returns; no partial handoff is started or retried automatically | Connectivity is lost before or during the handoff launch | Connectivity is restored -- the screen returns to its state before the loss (Not Started or the last recorded outcome) |

## Validation Rules

Validation governed by FEAT-28.SPEC-004 (Payout Account Eligibility & Constraints). This screen enforces the one-payout-account-per-Pro-Account rule implicitly by never being reachable a second time once a Payout Account exists for this Pro Account (see Edge Cases); no field-level validation exists on this screen since no bank or identity fields are entered here.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| "Connect payout account" tap | FEAT-28.SPEC-006 (Payout Account Connection & Verification) | -- |
| "Continue setup" tap | FEAT-15, next setup step (calendar connection) | FEAT-15 (Pro Onboarding & Setup Wizard) |
| Back control tap | FEAT-15, previous setup step (cancellation policy) | FEAT-15 (Pro Onboarding & Setup Wizard) |

## Data Model

**Creates:** None directly -- the Payout Account record itself is created by FEAT-28.SPEC-003 the moment the payment-processing capability first reports a connection outcome; this screen only initiates the handoff that produces that report.
**Reads:** Payout Account -- status, once a handoff has been launched at least once in this session, to render the status region.
**Updates:** None.
**Deletes:** None.

## Business Rules

- XBR-06: no deposit can be taken, and the booking link cannot go live, until this feature reports the Payout Account Active; "Continue setup" being enabled at Verification Pending does not itself satisfy XBR-06 -- FEAT-15's own go-live check (XBR-26) still requires Active before publishing the booking link.
- FEAT-28.SPEC-004 governs that exactly one Payout Account exists per Pro Account; this screen is reachable only when no Payout Account yet exists for the signed-in Pro.
- FEAT-28.SPEC-004 governs the country/currency match between the Pro Account and the Payout Account (XBR-25); any mismatch is a rejection reported back by the capability itself during the handoff (FEAT-28.SPEC-006), not a check performed on this screen.
- Bank and identity details never reach this screen or the product's own code (SC-11-class boundary, ASMP-31); the handoff hands Talia directly to the capability's own entry surface.

## Edge Cases

- **Talia navigates away mid-handoff (before any outcome is reported)** -- No Payout Account record is created; returning to this step shows the Not Started state again, and she can launch the handoff again without penalty.
- **Talia reaches this step a second time after a Payout Account already exists (e.g., she goes back in the wizard after completing this step)** -- The screen instead shows the recorded outcome directly (Verification Pending / Active) rather than the Not Started state, since FEAT-28.SPEC-004 prevents a second Payout Account from being created for the same Pro Account.
- **The handoff itself fails to launch (the capability cannot be reached at all)** -- Handoff Failed state shown with "Try again"; no Payout Account record is created since no outcome was ever reported.
- **Talia taps "Connect payout account" twice in rapid succession** -- The second tap is ignored while the first handoff launch is in progress (button in loading state).
- **The capability reports Action Required on this very first attempt (e.g., a rejected bank detail on first submission)** -- The status region shows the same "needs action" wording FEAT-28.SPEC-002 uses, with the same resolution link into FEAT-28.SPEC-006; "Continue setup" remains enabled, since existing product behavior lets the Pro finish setup with a non-Active payout account and resolve it later, consistent with the Brief's Alternate flow.
- **Talia's device loses connectivity while the capability's flow is open** -- Handled entirely inside that flow, which is outside this screen's own connectivity boundary; on return to this screen, the reported outcome (if any) is shown, or Handoff Failed if none was received.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-28.SPEC-006 (Payout Account Connection & Verification) | Triggers (outbound) | "Connect payout account" initiates the outbound handoff |
| FEAT-28.SPEC-004 (Payout Account Eligibility & Constraints) | References (inbound) | One-account-per-Pro and country/currency match rules govern this screen's reachability and outcomes |
| FEAT-28.SPEC-003 (Payout Account Status Processing) | Triggers (outbound) | The reported handoff outcome is what that automation writes to the Payout Account record |
| FEAT-28.SPEC-002 (Payout Status & Money Dashboard) | Navigation (outbound) | The same status treatment continues on the ongoing dashboard once setup is complete |
| FEAT-15.SPEC-001 (Setup Wizard Shell, Step Navigation & Guidance), FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over), FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) -- within FEAT-15 (Pro Onboarding & Setup Wizard) | Navigation (inbound / outbound) | Entry point and the "Continue setup" / back destinations |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Navigation (outbound) | Unauthenticated access is redirected here (XBR-29) |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| payout_account_connect_started | outcome of prior attempt, if any | Talia taps "Connect payout account" | supports success-metrics.md: "Setup-to-Live-Link Completion" |
| payout_account_connect_outcome_shown | outcome (verification_pending / active / handoff_failed) | The status region first shows a reported outcome | supports success-metrics.md: "Setup-to-Live-Link Completion" |
| payout_account_setup_continued | outcome at time of continuing | Talia taps "Continue setup" | supports success-metrics.md: "Setup-to-Live-Link Completion" |

## Acceptance Criteria

**FEAT-28.SPEC-001-AC-01:** Given Talia reaches the "getting paid" step of setup for the first time, when the screen loads, then she sees the plain-words explanation and the "Connect payout account" action, with "Continue setup" disabled.

**FEAT-28.SPEC-001-AC-02:** Given Talia is on this screen, when she taps "Connect payout account", then the handoff to the payment-processing capability launches (FEAT-28.SPEC-006).

**FEAT-28.SPEC-001-AC-03:** Given the capability reports the connection as Verification Pending, when the outcome is received, then the status region shows "Verification pending -- we'll let you know as soon as it's ready." and "Continue setup" becomes enabled.

**FEAT-28.SPEC-001-AC-04:** Given the capability reports the connection as Active, when the outcome is received, then the status region shows "Payout account connected and active."

**FEAT-28.SPEC-001-AC-05:** Given the handoff fails to launch at all, when Talia taps "Connect payout account", then she sees "We couldn't start the connection process. Try again." with a "Try again" action, and "Continue setup" stays disabled.

**FEAT-28.SPEC-001-AC-06:** Given Talia is at Verification Pending, when she taps "Continue setup", then FEAT-15 advances to the calendar connection step.

**FEAT-28.SPEC-001-AC-07:** Given Talia taps "Why do you need my bank details?", when the tap registers, then an inline disclosure panel expands in place with no navigation away from this screen.

**FEAT-28.SPEC-001-AC-08:** Given Talia has not launched a handoff yet, when she looks for "Continue setup", then it is disabled.

**FEAT-28.SPEC-001-AC-09:** Given Talia taps "Connect payout account" twice rapidly, when the second tap registers, then it is ignored while the first launch is in progress.

**FEAT-28.SPEC-001-AC-10:** Given Talia loses connectivity while viewing this screen before launching a handoff, when she taps "Connect payout account", then the offline banner appears and no handoff is initiated.

**FEAT-28.SPEC-001-AC-11:** Given Talia already has a Payout Account from a prior visit to this step, when she reaches this step again, then the screen shows the recorded outcome directly instead of the Not Started state.

**FEAT-28.SPEC-001-AC-12:** Given the capability reports Action Required on the very first handoff attempt, when the outcome is received, then the status region shows the needs-action wording with a resolution link into FEAT-28.SPEC-006, and "Continue setup" remains enabled.

**FEAT-28.SPEC-001-AC-13:** Given Talia navigates away before any outcome is reported, when she returns to this step, then no Payout Account record exists and the Not Started state is shown again.

**FEAT-28.SPEC-001-AC-14:** Given an unauthenticated visitor somehow reaches this screen's URL directly, when the screen attempts to load, then they are redirected to the Pro sign-in screen (FEAT-29).

**FEAT-28.SPEC-001-AC-15:** Given Talia's session expires while she is on this screen, when she next interacts with it, then she sees "Your session has expired. Sign in to continue." with no in-progress handoff to preserve.

**FEAT-28.SPEC-001-AC-16:** Given Talia taps the back control, when the tap registers, then FEAT-15 shows the previous setup step (cancellation policy) and any recorded Payout Account outcome is unaffected.

**FEAT-28.SPEC-001-AC-17:** Given the connection reaches Active before Talia taps "Continue setup", then FEAT-15's own go-live check (XBR-26) still evaluates all remaining requirements before the booking link may go live -- this screen's "Continue setup" being enabled does not itself publish the link.

**FEAT-28.SPEC-001-AC-18:** Given Platform Operator (Support) attempts to view this screen for a Pro, when the attempt is made, then it is not reachable to Support at all -- Support instead sees status through FEAT-28.SPEC-002.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 5 | 5 |
| States | 6 (not started, launching, verification pending, active, handoff failed, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
