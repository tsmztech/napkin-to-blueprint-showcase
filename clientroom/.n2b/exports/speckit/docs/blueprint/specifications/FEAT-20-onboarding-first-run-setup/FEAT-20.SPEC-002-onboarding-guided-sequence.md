---
document_type: spec
spec_type: screen
spec_id: FEAT-20.SPEC-002
spec_name: Onboarding Guided Sequence
spec_slug: onboarding-guided-sequence
parent_feature: FEAT-20
parent_feature_name: Onboarding / First-Run Setup
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 27
---

# Screen Spec: Onboarding Guided Sequence

## Overview

**Name:** Onboarding Guided Sequence
**ID:** FEAT-20.SPEC-002
**Type:** Screen
**Purpose:** The step-by-step shell that welcomes Nadia, asks the optional "how did you hear" question, hosts navigation into each guided step, shows progress, shows the Ready state (with a "Go to your dashboard" button) once onboarding's exit criteria are met, shows the welcome-email delivery warning if that email could not be delivered; Dana views the same progress read-only inside a logged support session.
**Parent Feature:** FEAT-20 -- Onboarding / First-Run Setup

## Scope and Non-Goals

**In Scope:**
- The Welcome moment and the optional "How did you hear about us?" question, shown on every open until Nadia answers or skips it (then never again)
- The single progress indicator, "skip for now" affordance, and failure-tolerant "continue anyway" pattern shared by every step, per the Brief's Shared Context (Shared UI Patterns)
- Hosting navigation into each guided step's own screen (owned by FEAT-01, FEAT-19, FEAT-32, FEAT-02) and returning from it
- Displaying the Ready state once FEAT-20.SPEC-003 reports onboarding complete, until Nadia taps its "Go to your dashboard" button; after that, redirecting to the dashboard (FEAT-12) on any open
- Surfacing the welcome-email delivery warning defined by FEAT-20.SPEC-006 (surface, wording, dismissal)
- Re-triggering FEAT-20.SPEC-003 on every open while onboarding is In Progress, and offering "Check again" when a completion check could not finish
- Dana's read-only view of the same progress inside a logged support session (FEAT-31)

**Non-Goals:**
- Which steps are mandatory versus optional, skip/resume behavior, and failure-tolerance rules -- owned by FEAT-20.SPEC-005 (Onboarding Step Sequencing & Exit-Criteria Rules); this shell reads and applies that spec's rules rather than redefining them
- Determining when onboarding is complete -- owned by FEAT-20.SPEC-003 (Onboarding Completion Detection); this shell only renders whatever state that automation reports. Routing is this shell's job alone: FEAT-20.SPEC-003 signals, it never navigates
- The content of each guided step itself (adding a client and project, setting branding, connecting payments, drafting a proposal) -- each is owned by its own feature's screen (FEAT-01, FEAT-19, FEAT-32, FEAT-02 respectively); this shell provides only the shared chrome around them
- A configurable workflow or step builder -- excluded per scope-boundaries.md SC-11: the product ships one fixed, sensible onboarding sequence rather than a customizable one

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-20.SPEC-001 (Sign-Up & Account Creation) | Successful account creation | New Freelancer Account, no onboarding-progress state yet -- opens on the Welcome state |
| Direct return (sign-in landing, bookmark, browser reopen) and FEAT-20.SPEC-006's "Continue setup" email link | Nadia signs in or opens the location (post-sign-in landing rule, FEAT-20.SPEC-005 R-08) | Existing onboarding-progress state is read. In Progress: the shell resumes on the step it reflects (the Welcome state if the "How did you hear" question is still unresolved). Complete and Ready not yet acknowledged: the Ready state. Complete and acknowledged: no shell is rendered; Nadia is redirected to FEAT-12 with no message |
| FEAT-01, FEAT-19, FEAT-32, FEAT-02's own onboarding-context screens | Nadia completes or exits a guided step's own screen | The completed or exited step's outcome, so this shell can advance, mark skipped, or mark failed-but-continuable |
| FEAT-31 (Operator Support Access) | Dana opens a read-only support session on the freelancer's account | Session is read-only; no step controls render; the welcome-email warning, if applicable, renders without its buttons |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|--------------------------|
| Nadia (Freelancer) | Full screen | All actions: answer or skip "how did you hear," enter any step, skip a skippable step, continue past a failed step | -- |
| Owen (Client Primary Contact) | No | No | This location is not served to a portal session. Owen sees a plain page titled "This page isn't available" with the text "That page isn't part of your portal." and a "Back to your portal" button that leads to his portal home (FEAT-05). No onboarding data is loaded (the Access field states onboarding is "Nadia only") |
| Priya (Client Reviewer Contact) | No | No | Identical to Owen: the "This page isn't available" page with "That page isn't part of your portal." and a "Back to your portal" button leading to her portal home (FEAT-05) |
| Dana (Support Operator) | Inside an open support session on this account: full progress view, no step controls. Outside an open support session: No | No -- view-only, per the Access Matrix's "View" row for Support Access | Inside a session, step-entry, "Skip for now," "Try again," "Check again" and "continue anyway" controls and the warning's buttons are not rendered. Outside a session, this location is not served to her: she sees a plain page titled "This page isn't available" with the text "Open a support session on a freelancer's account to view their setup progress." and a "Go to support access" button leading to FEAT-31; no freelancer data is loaded |
| Unauthenticated | No | No | Redirected to the sign-in screen; onboarding requires a Freelancer Account |
| Expired session | No | No | Dialog "Your session has expired. Sign in to continue." -- any in-progress guided-step input (e.g., a partially filled branding form reached from here) is preserved by that step's own screen and restored after re-authentication succeeds |

## Layout and Content

**Header:** A single progress indicator showing all steps in order -- "How did you hear" (shown until answered or skipped), "Add your first client and project," "Set your branding" (marked Optional), "Connect payments" (marked Optional), "Draft your first proposal" -- with the current step highlighted and completed steps marked done. Dana's read-only session shows the identical indicator with no highlighting affordance for interaction.

**Notice area (top of Body, above the panel):** Zero or more of the following notices, each visible only when its condition holds:
- *Welcome-email delivery warning:* "We couldn't deliver your welcome email to {sign_in_email}. If that address is wrong, you can correct it in Settings." with a "Go to Settings" button and a "Dismiss" link. Shown when FEAT-20.SPEC-006 reports its retries exhausted for this account, the sign-in email has not since been changed, and onboarding_welcome_warning_dismissed is false. Shown in the In Progress, Welcome and Ready states. In Dana's session the notice text shows with neither button nor link.
- *Completion-check notice:* "We couldn't confirm your setup just now." with a "Check again" button. Shown only after FEAT-20.SPEC-003 reports that a check could not finish.

**Body:** A single content area that hosts either:
- The Welcome message and the "How did you hear about us?" question (shown on every open until answered or skipped): a short welcome line, one text-or-selection input for the answer, a "Skip" link, and a "Continue" button.
- The current step's own launch panel: the step's name, one line describing what it accomplishes, a "Not finished" tag if its last attempt failed (mandatory steps only), and an "Add first client and project" / "Set branding" / "Connect payments" / "Draft the first proposal" button that navigates into that step's owning feature's screen. Skippable steps (branding, payments) additionally show a "Skip for now" link beside the button.
- The Ready state: a confirmation message that the first client, project, and drafted proposal all exist, and a "Go to your dashboard" button. The shell does not route automatically; Nadia stays on the Ready state until she taps the button.

**Footer:** None -- all actions live in the body panel for the current state.

### Responsive Behavior

- **Compact breakpoint:** Progress indicator collapses to a compact form showing only "Step {n} of {total}" plus the current step's name, with the full step list reachable by tapping it; body panel remains single-column, full width.
- **Medium size class and above:** Full progress indicator with all step names visible across the header; body panel capped at a consistent platform-wide content width and horizontally centered.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|---------------|----------|
| "How did you hear" input | Type or select | Captures the answer | Field shows entered value | Standard input focus state |
| "How did you hear" Continue button | Tap | Triggers FEAT-20.SPEC-004 with the entered answer (FEAT-20.SPEC-004 alone sets onboarding_how_did_you_hear_resolved; this screen never writes it), then advances to the first mandatory step without waiting for the hand-off | Screen moves to the next step's launch panel | Brief transition; no separate confirmation needed |
| "How did you hear" Skip link | Tap | Triggers FEAT-20.SPEC-004 with "unknown" as the answer (this screen never writes onboarding_how_did_you_hear_resolved), then advances without waiting for the hand-off | Screen moves to the next step's launch panel | -- |
| Progress indicator (any completed or current step) | Tap | Navigates to that step's launch panel (Nadia only) | Body panel switches to the tapped step | -- |
| Progress indicator (Dana's session) | Tap | No action -- display only in a read-only session | None | -- |
| "Add first client and project" button | Tap | Navigates to FEAT-01.SPEC-001 (Add Client), carrying the onboarding context | This shell is left in place behind the navigation; returns here on completion or exit | Standard navigation transition |
| "Set branding" button | Tap | Navigates to FEAT-19.SPEC-001 (Branding Settings), carrying the onboarding context | Same as above | Standard navigation transition |
| "Set branding" -> "Skip for now" link | Tap | Applies FEAT-20.SPEC-005's skip rule; emits `onboarding_step_skipped`; advances to the next step | Body panel switches to the next step's launch panel | Brief confirmation: "You can set this up anytime from Settings." |
| "Connect payments" button | Tap | Navigates to FEAT-32.SPEC-001 (Payment Connection Screen), carrying the onboarding context | Same as branding/client above | Standard navigation transition |
| "Connect payments" -> "Skip for now" link | Tap | Applies FEAT-20.SPEC-005's skip rule; emits `onboarding_step_skipped`; advances to the next step | Body panel switches to the next step's launch panel | Brief confirmation: "You can connect payments anytime from Settings." |
| "Draft the first proposal" button | Tap | Navigates to FEAT-02.SPEC-001 (Proposal Draft Editor) for the new project, carrying the onboarding context | Same as above | Standard navigation transition |
| Returning from a step's own screen (completed) | Automatic, on return | Emits `onboarding_step_completed`; triggers FEAT-20.SPEC-003 to re-check exit criteria; advances to the next step or the Ready state | Progress indicator marks the step done | Brief confirmation on the step just completed |
| Returning from an optional step's own screen (failed action, e.g., a logo upload failure inside FEAT-19) | Automatic, on return with a failure signal | Applies FEAT-20.SPEC-005's Rule R-04 for optional steps: adds the step to onboarding_failed_incomplete_steps, emits `onboarding_step_failed_continued`, allows progression | Body panel advances to the next step; the failed step is marked "Finish later from Settings" rather than done | Brief message: "We'll let you pick this back up from Settings later." |
| Returning from a mandatory step's own screen (failed action, e.g., client or project creation, or proposal drafting, failed) | Automatic, on return with a failure signal | Applies FEAT-20.SPEC-005's Rule R-04 for mandatory steps: no progression, nothing recorded as continued-past; the step stays current | The same step's launch panel shows a "Not finished" tag, its button relabelled "Try again" (same destination as the original button), and no "Skip for now" or "continue anyway" | Message: "That didn't finish saving. Try again when you're ready." Nadia may still leave onboarding at any time |
| "Check again" button (Completion-check notice) | Tap | Re-triggers FEAT-20.SPEC-003 | Notice replaced by a loading indicator; on a successful check the notice is removed and the shell renders the outcome (Ready, or the current step) | If the check fails again the notice returns with "We couldn't confirm your setup just now." |
| Shell opens while onboarding is In Progress | Automatic, on open (after the progress read succeeds) | Triggers FEAT-20.SPEC-003 to re-check the exit criteria | If the check reports complete, the shell renders the Ready state | None unless the check cannot finish (Completion-check notice) |
| Ready state "Go to your dashboard" button | Tap | Sets onboarding_ready_acknowledged to true, then navigates to FEAT-12 (Freelancer Financial Dashboard) | Onboarding shell is exited; later opens of this location redirect to FEAT-12 | Standard navigation transition |
| Welcome-email warning "Go to Settings" button | Tap | Navigates to FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) so Nadia can correct the address | Shell is left in place behind the navigation; the warning stays until the address is changed or the warning is dismissed | Standard navigation transition |
| Welcome-email warning "Dismiss" link | Tap | Sets onboarding_welcome_warning_dismissed to true | Warning is removed and never shown again | -- |
| Leaving to any other feature (dashboard, clients, Settings) while In Progress | Nadia navigates through the product's normal navigation | Nothing is blocked (FEAT-20.SPEC-005 Rule R-08); onboarding-progress state is left unchanged | This shell is left; the next sign-in lands here again at the current step | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Progress indicator (as a landmark, not a tab stop for Dana's read-only session) -> body panel content, top to bottom -> primary action button -> secondary "Skip for now" link where present.
- **Step-change announcements:** When the body panel switches to a new step (by completion, skip, or manual navigation), the new step's name is announced to assistive technology.
- **Confirmation announcements:** Brief confirmations (skip, continue-anyway) are announced as they appear.
- **Keyboard alternatives:** Every action is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-------------------|------------------|
| Loading | Progress indicator skeleton shown, body panel empty | Screen opens and onboarding-progress state is being read from the Freelancer Account | The read completes -- shell renders Welcome, In Progress, or Ready based on the result |
| Load Error | Banner "We couldn't load your setup progress." with a Retry button; no step panel shown | Reading onboarding-progress state fails | User taps Retry and the read succeeds |
| Welcome | Welcome message and "How did you hear" question shown | onboarding_current_step is "How did you hear", onboarding_how_did_you_hear_resolved is false and onboarding_status is In Progress (FEAT-20.SPEC-005 R-06) -- on first open after sign-up and on every return until she answers or skips (closing the browser without answering leaves it unresolved) | Nadia answers or skips the question |
| In Progress -- Step {n} | Current step's launch panel shown, progress indicator reflects position | A step is active and not yet completed, skipped, or continued-past | The step is completed, skipped (if skippable), or continued past after a failure |
| Completion Check Pending | The current step's panel plus the Completion-check notice with "Check again" | FEAT-20.SPEC-003 reports that a check could not finish (read failure), on a step return or on shell open | A later check (via "Check again", the next shell open, or the next step completion) finishes: the notice is removed and the shell shows Ready or the current step |
| Ready | Confirmation message and "Go to your dashboard" button; no step controls | onboarding_status is Complete and onboarding_ready_acknowledged is false | Nadia taps "Go to your dashboard" (the only exit; there is no automatic reroute) |
| Redirected (Complete and acknowledged) | No shell is rendered | onboarding_status is Complete and onboarding_ready_acknowledged is true, on any open of this location | Nadia lands on FEAT-12 with no message |
| Dana's Read-Only View | Identical progress indicator and current-step name, with no interactive controls | A support session (FEAT-31) is opened on this freelancer's account | The support session closes |
| Offline/Degraded | Banner "This step needs a connection." at the top of the body panel; the shell's own navigation and progress reading remain available (no record is created by this shell itself), but any step reached from here that creates a record shows its own offline behavior | Connectivity is lost while this shell is open | Connectivity is restored |

## Validation Rules

Validation of what is skippable, mandatory, and failure-tolerant is governed by FEAT-20.SPEC-005 (Onboarding Step Sequencing & Exit-Criteria Rules). See that spec for the complete rule set. This screen applies those rules to decide which "Skip for now" links appear and whether a failed step blocks progression.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-------------------|---------------------------------------|
| "Add first client and project" | FEAT-01.SPEC-001 (Add Client) | FEAT-01 (Client & Project Management) |
| "Set branding" | FEAT-19.SPEC-001 (Branding Settings) | FEAT-19 (Freelancer Branding) |
| "Connect payments" | FEAT-32.SPEC-001 (Payment Connection Screen) | FEAT-32 (Payment Account Connection) |
| "Draft the first proposal" | FEAT-02.SPEC-001 (Proposal Draft Editor) | FEAT-02 (Proposal Creation & Sending) |
| Ready state "Go to your dashboard" | Freelancer Financial Dashboard | FEAT-12 (Freelancer Financial Dashboard) |
| Any open when Complete and acknowledged (redirect) | Freelancer Financial Dashboard | FEAT-12 (Freelancer Financial Dashboard) |
| Welcome-email warning "Go to Settings" | FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) | FEAT-21 (Settings & Account Management) |
| Owen or Priya reaching this location ("Back to your portal") | Portal home | FEAT-05 (Client Portal Access) |
| Dana outside a support session ("Go to support access") | Support access entry | FEAT-31 (Operator Support Access) |

## Data Model

**Creates:** None directly -- record creation happens in the destination features' own screens (Client, Project, Branding Profile, Payment Account Connection, Proposal).
**Reads:** Freelancer Account -- its onboarding-progress state (which step is current, which steps are completed or skipped, whether any step is marked "finish later", onboarding_how_did_you_hear_resolved, onboarding_ready_acknowledged, onboarding_welcome_warning_dismissed), read on every open to determine which state to render. Also reads the welcome email's delivery outcome from the transactional email delivery status (FEAT-14.SPEC-001 / FEAT-14.SPEC-003) and the current sign-in email (to hide the warning once it has changed).
**Updates:** Freelancer Account -- its onboarding-progress state only, advanced as steps complete, are skipped, or are continued past after failure, plus onboarding_ready_acknowledged (on "Go to your dashboard") and onboarding_welcome_warning_dismissed (on "Dismiss"). onboarding_how_did_you_hear_resolved is read here but never written by this screen: FEAT-20.SPEC-004 is its only writer. No other Freelancer Account field (name, email, business details, payment terms, notification preferences) is written by this screen; those updates belong entirely to FEAT-21.
**Deletes:** None.

## Business Rules

- Step sequencing, mandatory/optional status, skip behavior, and failure tolerance are all governed by FEAT-20.SPEC-005 -- this shell never defines its own version of these rules.
- Completion is determined exclusively by FEAT-20.SPEC-003; this shell renders whatever state that automation reports and never independently decides onboarding is complete.
- The "How did you hear" question is resolved exactly once: it is shown on every open until Nadia answers or skips it, and onboarding_how_did_you_hear_resolved is set true by FEAT-20.SPEC-004 after its hand-off attempt on answer or skip (this screen only triggers it) -- never on render. A Nadia who saw it and left is asked again on return; a Nadia who answered or skipped is never asked again (FEAT-20.SPEC-005 R-06). If onboarding completes first, the question is auto-resolved as "unknown" and never shown (R-09).
- Ready-state model: the shell never auto-routes. On reaching Complete, the Ready state stays until Nadia taps "Go to your dashboard", which records onboarding_ready_acknowledged; any open of this location after that redirects to FEAT-12. FEAT-20.SPEC-003 only signals completion and never navigates.
- No navigation gate: while onboarding is In Progress Nadia can open any other feature freely; the shell is her post-sign-in landing destination, not a barrier (FEAT-20.SPEC-005 R-08). Onboarding-progress state is unchanged by leaving.
- The welcome-email delivery warning is the account-level variant of the FEAT-14.SPEC-006 delivery-failure warning, shown here because no project exists yet on which to show it. It disappears when Nadia dismisses it or changes her sign-in email; after the Ready state is acknowledged it is no longer surfaced, since her account is fully usable and she has already signed in with that address.
- Dana's session never shows step-entry controls, "Skip for now," or "continue anyway" -- the read-only constraint applies to this shell exactly as it applies to every other feature Dana can view (XBR-29).
- Every guided step's own record creation (Client, Project, Branding Profile, Payment Account Connection, Proposal) is owned entirely by that step's own feature; this shell's chrome (progress indicator, skip affordance, continue-anyway pattern) is described identically wherever those screens are reached through onboarding, per the Brief's Shared Context.

## Edge Cases

- **Nadia closes the browser mid-step and returns later** -- Onboarding-progress state read on the next open resumes exactly where she left off; no step is re-shown as incomplete if it already completed before she left.
- **Nadia navigates directly to a step's URL out of order (e.g., proposal drafting before adding a client)** -- The destination screen (FEAT-02.SPEC-001) itself requires an existing project; with none yet, it redirects back to this shell's current step rather than opening in an invalid state. This is the destination screen's own precondition, not an onboarding gate; free navigation elsewhere is unaffected (R-08).
- **Nadia sees the "How did you hear" question, closes the browser, and returns later** -- The question is unresolved, so the Welcome state is shown again; the referring-portal reference persisted at sign-up is unaffected and is handed off when she answers or skips.
- **Nadia skips branding, then returns to it later from Settings (FEAT-21)** -- Setting branding later has no effect on this shell; onboarding's own progress indicator already marked that step skipped and does not revisit it.
- **A guided step's own action fails (e.g., a logo upload inside FEAT-19's branding step)** -- Per FEAT-20.SPEC-005's failure-tolerance rule, progression continues to the next step regardless; the failed step remains completable later from Settings, with no lost progress.
- **Dana opens a support session while Nadia is mid-step in her own session** -- Both views read the same onboarding-progress state; Dana's view has no controls to conflict with Nadia's, so there is no concurrent-edit conflict to resolve (Freelancer Account's Contention note states no contention arises from Dana's read-only access).
- **Connectivity is lost while a guided step that creates a record (client, project) is open** -- That step's own screen (FEAT-01.SPEC-001, FEAT-01.SPEC-002) shows its own offline message and preserves typed-but-unsaved input; this shell itself shows no error, since it creates nothing directly.
- **All mandatory steps complete out of the shell's suggested order (e.g., Nadia opens the proposal draft screen directly after adding a client, skipping past the shell's own "Draft the first proposal" button)** -- FEAT-20.SPEC-003 detects completion from the resulting records regardless of navigation path, and the completion is already recorded by the time she next opens this location (FEAT-20.SPEC-003 triggers from the record saves themselves, and re-checks on every shell open); this shell then shows the Ready state (if she has not yet acknowledged it).
- **A completion check cannot finish (including for the final step)** -- The shell shows the Completion-check notice with "Check again"; the check also re-runs on every shell open and on every later step completion or record save, so Nadia is never stranded without Ready.
- **Nadia opens the location after tapping "Go to your dashboard"** -- She is redirected to FEAT-12 with no message; the Ready state is not shown twice.
- **The welcome-email warning and a dismissed state** -- Once dismissed the warning is never shown again, even if a later delivery failure occurs for that same email; changing the sign-in email (FEAT-21.SPEC-005) also removes it.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|--------------------|------------------|
| FEAT-20.SPEC-001 (Sign-Up & Account Creation) | Navigation (inbound) | Successful account creation opens this shell on the Welcome state |
| FEAT-20.SPEC-003 (Onboarding Completion Detection) | Triggers (outbound) / References (inbound) | Every step completion, every shell open while In Progress, and every "Check again" tap re-checks exit criteria; this shell renders the Ready state or the Completion-check notice the automation reports |
| FEAT-20.SPEC-004 (Referral Attribution Capture Hand-off) | Triggers (outbound) | Answering or skipping "How did you hear" triggers the capture hand-off |
| FEAT-20.SPEC-005 (Onboarding Step Sequencing & Exit-Criteria Rules) | References (inbound) | Governs mandatory/optional status, skip behavior, and failure tolerance applied by this shell |
| FEAT-01.SPEC-001 (Add Client) | Navigation (outbound) | "Add first client and project" step entry point |
| FEAT-19.SPEC-001 (Branding Settings) | Navigation (outbound) | "Set branding" step entry point |
| FEAT-32.SPEC-001 (Payment Connection Screen) | Navigation (outbound) | "Connect payments" step entry point |
| FEAT-02.SPEC-001 (Proposal Draft Editor) | Navigation (outbound) | "Draft the first proposal" step entry point |
| FEAT-12 (Freelancer Financial Dashboard) | Navigation (outbound) | The Ready state's "Go to your dashboard" button, and the redirect on any open after acknowledgement, lead to the dashboard; FEAT-12 is otherwise unaffected by onboarding |
| FEAT-20.SPEC-006 (Welcome Email) | References (inbound) / Navigation (inbound) | Supplies the delivery-failure outcome that drives the welcome-email warning defined here; its "Continue setup" link opens this location |
| FEAT-14.SPEC-006 (Delivery Failure Warning to Freelancer) | References (inbound) | The project-level delivery warning this account-level variant stands in for while no project exists |
| FEAT-14.SPEC-003 (Delivery Status Tracking & Retry) | References (inbound) | Source of the welcome email's final delivery outcome |
| FEAT-21.SPEC-005 (Sign-In Email & Login Method Change) | Navigation (outbound) | The warning's "Go to Settings" button leads here so Nadia can correct her sign-in email |
| FEAT-05 (Client Portal Access) | Navigation (outbound) | Owen and Priya's "Back to your portal" destination from the unavailable-page experience |
| FEAT-31 (Operator Support Access) | Navigation (inbound) / (outbound) | Dana's read-only session view of this shell; Dana's "Go to support access" destination when she reaches it outside a session |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|-------------------|----------------------|
| onboarding_step_completed | step name, time since previous step, step index | A guided step's own screen reports completion | supports success-metrics.md: "First-Session Activation" |
| onboarding_step_skipped | step name (branding or payments) | Nadia taps "Skip for now" on a skippable step | supports success-metrics.md: "First-Session Activation" (a skip is a completed, if lighter, path through the same session-activation flow) |
| how_did_you_hear_answered | answered vs. skipped | Nadia answers or skips the "How did you hear" question | supports success-metrics.md: "Growth Through Referral" |
| onboarding_step_failed_continued | step name, failure reason | A step's own action fails and this shell allows progression regardless | N/A -- no Stage 2 metric measures individual step failures; retained so failure-tolerance is observable rather than invisible |

## Acceptance Criteria

**FEAT-20.SPEC-002-AC-01:** Given Nadia has just created her account, when the guided sequence first opens, then she sees the Welcome message and the "How did you hear about us?" question.

**FEAT-20.SPEC-002-AC-02:** Given Nadia is on the Welcome state, when she types an answer and taps Continue, then FEAT-20.SPEC-004 is triggered with her answer (onboarding_how_did_you_hear_resolved is set true by FEAT-20.SPEC-004, not by this screen) and she advances to the "Add first client and project" step.

**FEAT-20.SPEC-002-AC-03:** Given Nadia is on the Welcome state, when she taps Skip instead of answering, then FEAT-20.SPEC-004 is triggered with "unknown" as the answer and she advances to the next step.

**FEAT-20.SPEC-002-AC-04:** Given Nadia is on the "Add first client and project" step, when she taps the button, then she is taken to FEAT-01.SPEC-001 (Add Client), and no "Skip for now" link is shown for this mandatory step.

**FEAT-20.SPEC-002-AC-05:** Given Nadia completes adding a client and project and returns to this shell, when the return is processed, then `onboarding_step_completed` is emitted, FEAT-20.SPEC-003 re-checks exit criteria, and she advances to the "Set your branding" step.

**FEAT-20.SPEC-002-AC-06:** Given Nadia is on the "Set your branding" step, when she taps "Skip for now", then `onboarding_step_skipped` is emitted, the confirmation "You can set this up anytime from Settings." appears, and she advances to "Connect payments" with no penalty.

**FEAT-20.SPEC-002-AC-07:** Given Nadia is on the "Connect payments" step, when she taps "Skip for now", then `onboarding_step_skipped` is emitted and she advances to "Draft your first proposal".

**FEAT-20.SPEC-002-AC-08:** Given Nadia has added a first client and project, when she reaches the "Draft the first proposal" step and taps its button, then she is taken to FEAT-02.SPEC-001 for the new project.

**FEAT-20.SPEC-002-AC-09:** Given Nadia has a first client, project, and drafted proposal, when FEAT-20.SPEC-003 reports the exit criteria met, then this shell shows the Ready state with a "Go to your dashboard" button, does not route automatically, and renders no step controls.

**FEAT-20.SPEC-002-AC-10:** Given Nadia is on the Ready state, when she taps "Go to your dashboard", then onboarding_ready_acknowledged becomes true and she is routed to FEAT-12 (Freelancer Financial Dashboard).

**FEAT-20.SPEC-002-AC-11:** Given Nadia's branding logo upload fails inside the "Set branding" step, when she returns to this shell, then progression continues to the next step regardless, the step is marked "Finish later from Settings," and `onboarding_step_failed_continued` is emitted, per FEAT-20.SPEC-005 Rule R-04 for optional steps.

**FEAT-20.SPEC-002-AC-12:** Given Dana opens a read-only support session on Nadia's account, when she views this shell, then she sees the identical progress indicator and current step with no step-entry buttons, "Skip for now," or "continue anyway" controls rendered.

**FEAT-20.SPEC-002-AC-13:** Given Owen (Client Primary Contact) or Priya (Client Reviewer Contact) in a portal session attempts to reach this location, when the attempt is made, then he or she sees the page "This page isn't available" with "That page isn't part of your portal." and a "Back to your portal" button leading to their portal home, and no onboarding data is loaded.

**FEAT-20.SPEC-002-AC-14:** Given Nadia closes the browser mid-step and signs back in later, when this shell reopens, then it resumes on the exact step reflected by her onboarding-progress state.

**FEAT-20.SPEC-002-AC-15:** Given Nadia loses connectivity while this shell is open, when the loss occurs, then the banner "This step needs a connection." appears in the body panel; if she then opens a step that creates a record, that step's own screen shows its own offline behavior.

**FEAT-20.SPEC-002-AC-16:** Given Nadia navigates directly to the proposal-draft screen's URL before adding any client, when the screen loads, then it redirects her back to this shell's current step rather than opening in an invalid state.

**FEAT-20.SPEC-002-AC-17:** Given Nadia completes her first client, project, and proposal draft by navigating directly to each feature's screen rather than tapping this shell's own buttons, when she next opens this shell, then FEAT-20.SPEC-003 has already recorded completion (triggered by the record saves) and the Ready state is shown, provided she has not yet tapped "Go to your dashboard".

**FEAT-20.SPEC-002-AC-18:** Given an unauthenticated visitor attempts to reach this shell's URL, when the attempt is made, then she is redirected to the sign-in screen.

**FEAT-20.SPEC-002-AC-19:** Given the read of onboarding-progress state fails when this shell opens, when the failure occurs, then the banner "We couldn't load your setup progress." appears with a Retry button and no step panel is shown.

**FEAT-20.SPEC-002-AC-20:** Given Nadia saw the "How did you hear" question, closed the browser without answering, and signs in again, when the shell opens, then onboarding_how_did_you_hear_resolved is still false and the Welcome state is shown again; after she answers or skips, later opens never show it.

**FEAT-20.SPEC-002-AC-21:** Given onboarding is Complete and Nadia has already tapped "Go to your dashboard", when she opens this location by sign-in, bookmark, or the welcome email's "Continue setup" link, then she is redirected to FEAT-12 with no message and the Ready state is not shown.

**FEAT-20.SPEC-002-AC-22:** Given Nadia's project creation fails inside the "Add first client and project" step, when she returns to this shell, then the same step's panel shows "Not finished" with a "Try again" button and the message "That didn't finish saving. Try again when you're ready.", with no "Skip for now" or "continue anyway" control.

**FEAT-20.SPEC-002-AC-23:** Given Nadia saves her proposal draft (the final mandatory step) and FEAT-20.SPEC-003 cannot finish its check, when she next sees this shell, then the Completion-check notice "We couldn't confirm your setup just now." with a "Check again" button is shown, and tapping "Check again" (or the next shell open) re-runs the check and shows the Ready state once it succeeds.

**FEAT-20.SPEC-002-AC-24:** Given the welcome email's retries were exhausted without delivery and Nadia has not dismissed the warning, when she opens this shell in the In Progress, Welcome, or Ready state, then the warning "We couldn't deliver your welcome email to {sign_in_email}. If that address is wrong, you can correct it in Settings." is shown with "Go to Settings" and "Dismiss".

**FEAT-20.SPEC-002-AC-25:** Given the welcome-email warning is shown, when Nadia taps "Go to Settings" she is taken to FEAT-21.SPEC-005, and when she taps "Dismiss" onboarding_welcome_warning_dismissed becomes true and the warning is never shown again.

**FEAT-20.SPEC-002-AC-26:** Given Dana has no open support session on Nadia's account, when she reaches this location, then she sees "This page isn't available" with "Open a support session on a freelancer's account to view their setup progress." and a "Go to support access" button, and no freelancer data is loaded.

**FEAT-20.SPEC-002-AC-27:** Given onboarding is In Progress, when Nadia opens the dashboard or Settings directly, then it opens normally with no gate or redirect back to this shell, and her next sign-in lands on this shell at her current step.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|----------------|-------|
| Interactions | 20 | 20 |
| States | 9 (loading, load error, welcome, in progress, completion check pending, ready, redirected, read-only, offline) | 9 |
| Business Rules | 8 | 8 |
| Edge Cases | 11 | 11 |
