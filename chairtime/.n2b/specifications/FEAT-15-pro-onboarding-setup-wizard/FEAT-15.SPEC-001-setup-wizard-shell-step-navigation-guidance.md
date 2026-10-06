---
document_type: spec
spec_type: screen
spec_id: FEAT-15.SPEC-001
spec_name: Setup Wizard Shell, Step Navigation & Guidance
spec_slug: setup-wizard-shell-step-navigation-guidance
parent_feature: FEAT-15
parent_feature_name: Pro Onboarding & Setup Wizard
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 14
---

# Screen Spec: Setup Wizard Shell, Step Navigation & Guidance

## Overview

**Name:** Setup Wizard Shell, Step Navigation & Guidance
**ID:** FEAT-15.SPEC-001
**Type:** Screen
**Purpose:** The persistent wizard frame that shows Talia her step order and progress, hands her into each step's owning-feature screen in sequence, and surfaces a short plain-language tip for whichever step she is on.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard

## Scope and Non-Goals

**In Scope:**
- The persistent step-progress frame (step list, current position, completed/current/upcoming/skipped status per step)
- Launching the Pro into each step's owning-feature screen in the fixed order defined by FEAT-15.SPEC-006
- The plain-language "what this step asks for" tip shown for the current step
- The always-available booking-page preview entry point (hands off to FEAT-05's preview mode)
- Reflecting resume position on return, as computed by FEAT-15.SPEC-004

**Non-Goals:**
- Collecting any step's actual field data (name, price, hours, bank details, card number) -- each step's owning feature (FEAT-29, FEAT-27, FEAT-01, FEAT-02, FEAT-28, FEAT-04, FEAT-18) owns its own form and validation; this shell only launches into it and reads back completion
- Creating the first Cancellation Policy version -- owned by FEAT-15.SPEC-002, this feature's own step screen, not the shell
- Computing or storing setup-progress state -- owned by FEAT-15.SPEC-004; this shell only displays what that automation reports
- Deciding whether the booking link may go live -- owned by FEAT-15.SPEC-007 (rule) and FEAT-15.SPEC-005 (automation); this shell only navigates to FEAT-15.SPEC-003 once told setup is complete
- Multi-staff or multi-chair setup paths -- excluded per SC-01: the product is strictly single-operator, so the shell has exactly one linear path for one Pro

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-29 (Pro sign-in creation) | Talia signs in for the first time with zero completed setup steps | None -- shell opens at step 1 (account & sign-in, already satisfied by the sign-in that just completed), so the shell opens showing step 2 (profile) as current |
| FEAT-29 (Pro sign-in) | Talia signs in with setup already in progress | Resume point computed by FEAT-15.SPEC-004 -- the shell opens directly at the first incomplete step with all earlier answers preserved |
| Any step's owning screen (FEAT-27, FEAT-01.SPEC-002, FEAT-02.SPEC-001, FEAT-28.SPEC-001, FEAT-04.SPEC-001, FEAT-18's subscribe screen) | Talia completes that step and the owning screen hands control back | Updated progress state (that step now marked complete); shell advances to the next step in order |
| FEAT-15.SPEC-002 (Cancellation Policy step, owned by this feature) | Talia saves the first Cancellation Policy version | Updated progress state; shell advances to the next step |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| The Pro (Talia) | Full screen | Navigate into any step in fixed order, review a completed step, tap the preview entry point | -- |
| The Client (Riley) | No | No | This screen is never reached by a client path; a client following any wizard-shaped link is sent to the public booking page (FEAT-05) or a "this booking page isn't available" message if the link does not resolve to a Pro |
| Platform Operator (Support) | No (Support does not use this wizard screen) | No | Support views a Pro's setup progress read-only through Platform Support Read-Only Access (FEAT-19), which reads the same progress state (FEAT-15.SPEC-004) surfaced in a support-facing view -- Support never opens this Pro-facing shell |
| Unauthenticated | No | No | Redirected to the Pro sign-in screen (FEAT-29); after signing in, a brand-new account lands on this shell at step 2, and a returning incomplete account lands at its resume point |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- no wizard data is lost because every completed step was already persisted by FEAT-15.SPEC-004 at the moment it completed; after re-authentication, the shell reopens at the same resume point it showed before the session expired |

## Layout and Content

**Header:** A step-progress indicator reading "Step {current_number} of 8: {current_step_name}" with a horizontal list of all 8 steps beneath it, each rendered with one of four states: Completed (checkmark), Current (highlighted), Upcoming (dimmed, not tappable), Skipped (shown only for the calendar step once Talia has explicitly skipped it, per FEAT-15.SPEC-006). A "Preview my booking page" action sits at the top-right of the header, available from the first step onward.

**Body:** A single content card for the current step, containing:
- The step's title (e.g., "Set your working hours")
- The plain-language tip: one to three sentences explaining, in non-technical terms, what this step asks for and why (e.g., for the cancellation-policy step: "Choose how far ahead of an appointment a client can cancel and still get their deposit back. Most pros start with 24 hours -- you can change this any time later.")
- A "Continue" button that launches the step's owning-feature screen

Completed steps in the header list are tappable: tapping one navigates into that step's owning screen in its own edit mode (e.g., tapping the completed "Services" step opens FEAT-01's Service List so Talia can review or add another service before continuing) without disturbing later steps' saved answers. Upcoming steps are not tappable -- their titles are visible but dimmed, communicating the fixed order without allowing Talia to jump ahead.

**Footer:** None -- the "Continue" action lives in the body card, and "Preview my booking page" lives in the header.

### Responsive Behavior

- **Compact breakpoint (phone width):** The step list collapses to a horizontally scrollable strip above the current-step card; the current step is always scrolled into view on load. The current-step card and its Continue button remain full width.
- **Medium size class and above:** The step list renders as a vertical rail to the left of the current-step card rather than a horizontal strip; no other structural change.
- **Preview action:** Remains a single tappable label at every size; it never collapses into an icon-only control, since a first-time Pro must recognize it by its words.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Step list item (completed step) | Tap | Navigate into that step's owning-feature screen in review/edit mode | Shell frame remains visible around the destination screen | Destination screen opens; on return, the shell re-displays with progress unchanged unless the Pro made a new change there |
| Step list item (current step) | Tap | Same as tapping "Continue" for the current step | -- | Owning-feature screen opens |
| Step list item (upcoming step) | Tap | No action -- not interactive | None | Dimmed appearance communicates it is not yet reachable; no error is shown, since this is expected, not a mistake |
| Step list item (skipped calendar step) | Tap | Navigate into FEAT-04's calendar connection screen (FEAT-04.SPEC-001) | -- | Talia can connect the calendar at any point after skipping it, from here or later from settings |
| "Continue" button | Tap | Navigate to the current step's owning-feature screen (FEAT-29, FEAT-27, FEAT-01.SPEC-002, FEAT-02.SPEC-001, FEAT-15.SPEC-002, FEAT-28.SPEC-001, FEAT-04.SPEC-001, or FEAT-18's subscribe screen, per the fixed order in FEAT-15.SPEC-006) | Shell hands control to the destination screen | Destination screen opens |
| "Preview my booking page" | Tap | Navigate to FEAT-05's preview mode for this Pro's current (possibly incomplete) profile and services | None on the shell itself | FEAT-05 opens in preview mode, showing the booking page exactly as a client would see it today, with no real payment possible |
| Step's owning screen completes and hands back | System event (not a direct tap) | FEAT-15.SPEC-004 updates progress; shell re-reads progress and advances the current-step indicator | Header progress list updates; body card shows the next step's title and tip | Brief transition to the next step's card |
| Final required step completes | System event | FEAT-15.SPEC-005 re-evaluates readiness (FEAT-15.SPEC-007); if satisfied, the shell navigates automatically | Shell is replaced by FEAT-15.SPEC-003 | Talia lands on the Go-Live Preview & Booking Link Hand-Over screen |

### Accessibility Notes

- **Focus order:** Step-progress list (left to right / top to bottom) -> "Preview my booking page" -> current-step title -> tip text -> "Continue" button.
- **Dynamic-change announcements:** When the shell advances to a new step after a hand-off screen returns, the new step's title and tip are announced to assistive technology as a single update, so Talia does not have to re-discover her position by re-reading the whole header.
- **Keyboard alternatives:** Every step-list item and the "Continue" and "Preview" actions are reachable and activatable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Fresh start | Header shows step 2 of 8 as current (step 1, account & sign-in, is already satisfied by the sign-in that opened this screen); all later steps upcoming | Talia signs in for the first time with zero completed steps | Talia taps Continue on step 2 |
| In progress | Header reflects whichever steps are complete, current, upcoming, or skipped, per FEAT-15.SPEC-004's progress record | Talia returns to the shell with some steps already complete | Talia advances a step, or the automatic hand-off to FEAT-15.SPEC-003 fires |
| Awaiting hand-off return | Shell frame persists in the background conceptually while a step's owning screen is open (from the Pro's perspective, the owning screen has the focus) | Talia taps Continue or a step-list item | The owning screen hands control back to the shell |
| Complete, hand-off pending | Brief transition state after the final required step reports complete and before FEAT-15.SPEC-005/007 finish evaluating readiness | The final required step (subscription, if it is completed last) reports complete | Readiness confirmed and the shell navigates to FEAT-15.SPEC-003 |
| Error | The shell itself shows no error state of its own -- a failed step submission is handled entirely inside that step's owning screen (each preserves entered values and offers retry, consistent with every setup screen in this product); the shell only ever shows a step as "not yet complete" until the owning screen reports success | A step's owning screen fails to save | Talia retries and succeeds inside the owning screen, which then reports completion back to the shell |
| Offline/Degraded | N/A -- setup is a deliberate, connected session on a stable connection between clients, not an in-the-moment mobile flow (feature's own States field); the shell requires connectivity to read and advance progress | -- | -- |

## Validation Rules

Validation governed by FEAT-15.SPEC-006 (Setup Step Order & Optional-Step Rules) for step sequencing and the one skippable step. This shell performs no field-level validation of its own -- every step's data validation is owned by that step's owning-feature spec (e.g., FEAT-01.SPEC-004 for service fields, FEAT-02.SPEC-005 for availability fields).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Continue on account & sign-in step | Pro sign-in creation screen | FEAT-29 (Pro Sign-In & Account Lifecycle) |
| Continue on profile step | Profile & studio location screen (FEAT-27.SPEC-001) | FEAT-27 (Pro Profile & Booking Page Settings) |
| Continue on service step | Add Service screen (FEAT-01.SPEC-002) | FEAT-01 (Service & Pricing Management) |
| Continue on hours step | Working Hours setup screen (FEAT-02.SPEC-001) | FEAT-02 (Availability & Working Hours Setup) |
| Continue on cancellation-policy step | FEAT-15.SPEC-002 (Cancellation Policy Default & First-Version Setup Step) | -- |
| Continue on payout step | Payout Account Connection screen (FEAT-28.SPEC-001) | FEAT-28 (Payout Account Connection & Payout Visibility) |
| Continue on calendar step (or tapping the skipped step later) | Calendar Connection Setup screen (FEAT-04.SPEC-001) | FEAT-04 (Two-Way Calendar Sync) |
| Continue on subscription step | Subscribe screen | FEAT-18 (Pro Subscription Billing & Account Management) |
| "Preview my booking page" | Booking page preview mode | FEAT-05 (Public Booking Page & Booking Flow) |
| All required steps satisfied | FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) | -- |

## Data Model

**Creates:** None -- this shell creates no data of its own; account and progress creation is owned by FEAT-15.SPEC-004.
**Reads:** Pro Account's setup-progress state (which of the 8 steps are complete/current/skipped, per FEAT-15.SPEC-004's record) -- read on every load and after every hand-off return.
**Updates:** None -- the shell never writes progress state directly; it only triggers FEAT-15.SPEC-004 to re-read and re-render after a step's owning screen reports completion.
**Deletes:** None.

## Business Rules

- Step order and the calendar step's optional/skippable status are governed entirely by FEAT-15.SPEC-006 -- this shell never re-derives or overrides the sequence.
- Progress is computed and persisted entirely by FEAT-15.SPEC-004 -- this shell is a read-and-navigate surface, not a state owner.
- Go-Live readiness (XBR-26) is evaluated entirely by FEAT-15.SPEC-007 via FEAT-15.SPEC-005 -- the shell hands off to FEAT-15.SPEC-003 the moment it is told readiness is reached, and never itself decides that setup is "done enough."
- The booking-page preview (FEAT-05's preview mode) is available from the first step onward and never requires setup to be complete, consistent with the Key Capability "a preview of the booking page exactly as a client will see it."

## Edge Cases

- **Talia is signed in on two devices and completes different steps on each at nearly the same time** -- Each device's owning-feature screen (e.g., FEAT-01.SPEC-002 on one device, FEAT-02.SPEC-001 on the other) saves its own step independently; FEAT-15.SPEC-004 records both completions (they touch disjoint progress fields, so there is no overwrite). On next load, each shell re-reads progress and shows both steps as complete -- no conflict dialog is needed because step completions are additive, not competing edits to the same field, consistent with the Pro Account entity's last-write-wins resolution for non-overlapping fields.
- **Talia navigates directly to the shell URL without having started setup** -- Not reachable: the shell is entered only through the FEAT-29 sign-in hand-off or a resume from a signed-in session; a signed-in Pro with zero progress is shown step 2 as current, per Entry Points.
- **Talia taps "Preview my booking page" before any service exists** -- FEAT-05's preview mode shows the "temporarily not accepting bookings" empty state that a zero-service booking page shows, so Talia can see what an incomplete page looks like to a client and is motivated to keep going.
- **Talia backs out of a step's owning screen without saving (e.g., closes the browser mid-form)** -- That step's owning screen preserves entered values and treats this the same as any other unsaved-exit case for that screen; the shell still shows the step as incomplete on her return, and she resumes exactly where FEAT-15.SPEC-004 last recorded her.
- **A step is completed out of the displayed order through a settings deep link (for example, Talia reaches FEAT-27 profile settings directly after account creation, from a source outside the wizard)** -- FEAT-15.SPEC-004 still records the completion; the shell's step list marks that step complete even though it was not reached by tapping Continue, and the current-step indicator advances to the next incomplete step in fixed order.
- **The final required step completes while Talia's payout account is still Verification Pending** -- The shell still hands off to FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over), which shows the "finish verifying to start taking bookings" waiting state rather than a live link, per FEAT-15.SPEC-007's payout condition.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-15.SPEC-004 (Setup Progress Tracking & Resume) | References (inbound) | The shell reads current progress and resume point from this automation on every load and after every hand-off return |
| FEAT-15.SPEC-006 (Setup Step Order & Optional-Step Rules) | References (inbound) | Governs the fixed step order and the calendar step's skip behavior shown in the header list |
| FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) | Triggers (outbound) | Firing after each step completion; the shell hands off to FEAT-15.SPEC-003 once this automation reports readiness |
| FEAT-15.SPEC-002 (Cancellation Policy Default & First-Version Setup Step) | Navigation (outbound) | Continue on the cancellation-policy step launches this sibling screen |
| FEAT-15.SPEC-003 (Go-Live Preview & Booking Link Hand-Over) | Navigation (outbound) | Automatic hand-off once all required steps are satisfied |
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Navigation (inbound and outbound) | Sign-in creation hands control to this shell; the account & sign-in step's Continue launches into FEAT-29 |
| FEAT-27 (Pro Profile & Booking Page Settings) | Navigation (outbound) | Profile step's Continue launches into FEAT-27 |
| FEAT-01.SPEC-002 (Add Service) | Navigation (outbound) | Service step's Continue launches into FEAT-01's Add Service screen |
| FEAT-02.SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) | Navigation (outbound) | Hours step's Continue launches into this screen |
| FEAT-28.SPEC-001 (Payout Account Connection) | Navigation (outbound) | Payout step's Continue launches into this screen |
| FEAT-04.SPEC-001 (Calendar Connection Setup) | Navigation (outbound) | Calendar step's Continue (or a later tap on the skipped step) launches into this screen |
| FEAT-05 (Public Booking Page & Booking Flow) | Navigation (outbound) | "Preview my booking page" launches FEAT-05's preview mode |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| onboarding_started | none | Talia's first entry into this shell after sign-in with zero completed steps | supports success-metrics.md: "Setup-to-Live-Link Completion" |
| onboarding_step_completed | step name, step number, whether it was reached via Continue or a direct deep link | Any step's owning screen reports completion back to the shell | supports success-metrics.md: "Setup-to-Live-Link Completion" |
| onboarding_preview_from_wizard_tapped | current step number at time of tap | Talia taps "Preview my booking page" | supports success-metrics.md: "Setup-to-Live-Link Completion" |

## Acceptance Criteria

**FEAT-15.SPEC-001-AC-01:** Given Talia has just completed sign-in for the first time, when the wizard shell opens, then it shows "Step 2 of 8: Profile & Studio Location" as the current step with all later steps shown upcoming and dimmed.

**FEAT-15.SPEC-001-AC-02:** Given Talia is on the current-step card for working hours, when she reads the tip text, then it explains in plain language what setting working hours means for her booking page, with no technical terms.

**FEAT-15.SPEC-001-AC-03:** Given Talia taps "Continue" on the current step, when the tap registers, then she is navigated to that step's owning-feature screen (for example, FEAT-01.SPEC-002 for the service step).

**FEAT-15.SPEC-001-AC-04:** Given Talia completes the working-hours step on FEAT-02.SPEC-001 and is handed back to the shell, when the shell re-renders, then "Working Hours" shows as Completed in the step list and "Step 5 of 8: Cancellation Policy" becomes current.

**FEAT-15.SPEC-001-AC-05:** Given Talia looks at an upcoming step in the header list, when she taps it, then nothing happens -- the step remains dimmed and not navigable.

**FEAT-15.SPEC-001-AC-06:** Given Talia has already completed the service step, when she taps that completed step in the header list, then she is navigated to FEAT-01's Service List in review/edit mode without disturbing her progress on later steps.

**FEAT-15.SPEC-001-AC-07:** Given Talia is on any step of the wizard, when she taps "Preview my booking page", then FEAT-05 opens in preview mode showing the booking page exactly as a client would see it, with no real payment possible.

**FEAT-15.SPEC-001-AC-08:** Given Talia skips the calendar-connection step per FEAT-15.SPEC-006, when the shell re-renders, then that step shows as "Skipped" in the header list rather than blocking progress to the subscription step.

**FEAT-15.SPEC-001-AC-09:** Given Talia later taps the skipped calendar step from the header list, when the tap registers, then she is navigated into FEAT-04.SPEC-001 to connect her calendar.

**FEAT-15.SPEC-001-AC-10:** Given Talia completes the final required step (subscription) and her payout account is already Active, when FEAT-15.SPEC-005 confirms readiness, then the shell automatically navigates her to FEAT-15.SPEC-003.

**FEAT-15.SPEC-001-AC-11:** Given Talia completes the final required step but her payout account is still Verification Pending, when the shell hands off, then she still lands on FEAT-15.SPEC-003, which shows the "finish verifying to start taking bookings" waiting state rather than a live link.

**FEAT-15.SPEC-001-AC-12:** Given Talia is completing setup on her phone with a stable connection and then loses connectivity, when she tries to advance a step, then the shell requires connectivity to proceed (per its Offline/Degraded state), consistent with setup being a deliberate connected session.

**FEAT-15.SPEC-001-AC-13:** Given an unauthenticated visitor reaches the wizard shell URL, when the screen loads, then they are redirected to the Pro sign-in screen (FEAT-29) and never see any wizard content.

**FEAT-15.SPEC-001-AC-14:** Given Talia's session expires while the wizard shell is open, when she next interacts with the screen, then a dialog reads "Your session has expired. Sign in to continue." and, after she signs back in, the shell reopens at the exact resume point it showed before expiry.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 8 | 8 |
| States | 6 (fresh start, in progress, awaiting hand-off return, complete/hand-off pending, error, offline/degraded) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
