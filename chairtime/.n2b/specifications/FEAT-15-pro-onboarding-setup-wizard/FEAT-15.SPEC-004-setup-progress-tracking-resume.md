---
document_type: spec
spec_type: automation
spec_id: FEAT-15.SPEC-004
spec_name: Setup Progress Tracking & Resume
spec_slug: setup-progress-tracking-resume
parent_feature: FEAT-15
parent_feature_name: Pro Onboarding & Setup Wizard
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Setup Progress Tracking & Resume

## Overview

**Name:** Setup Progress Tracking & Resume
**ID:** FEAT-15.SPEC-004
**Type:** Automation
**Purpose:** Creates the Pro Account's setup-progress record the moment Talia first enters the wizard, updates it as each step (including a skipped calendar step) completes, and computes the exact resume point whenever she returns -- the same progress state Support reads read-only.
**Parent Feature:** FEAT-15 -- Pro Onboarding & Setup Wizard

## Scope and Non-Goals

**In Scope:**
- Creating the Pro Account record and its setup-progress state on first entry into the wizard
- Updating setup-progress as each of the 8 steps (per FEAT-15.SPEC-006's order) completes, including the calendar step marked complete-as-skipped
- Computing the exact resume point (the first incomplete step) whenever Talia returns to the wizard shell
- Serving the same progress state read-only to Platform Operator (Support) via FEAT-19

**Non-Goals:**
- Establishing sign-in identity itself (email, mobile, one-time code) -- owned by FEAT-29; this automation creates the account envelope that makes resume possible once sign-in has already happened
- Deciding step order or which step is optional -- owned by FEAT-15.SPEC-006; this automation only records completion against that fixed order
- Evaluating or acting on Go-Live readiness -- owned by FEAT-15.SPEC-007 and FEAT-15.SPEC-005; this automation only supplies the progress data those specs evaluate
- Deleting or archiving the Pro Account or its progress state -- excluded per the feature's own explicit non-goal: removal is owned entirely by FEAT-29's 30-day cooling-off closure, with no independent retention/purge window here

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro completes sign-in and enters the wizard for the first time | FEAT-29 (Pro Sign-In & Account Lifecycle) | Fires exactly once per Pro Account, the first time sign-in completes with no existing setup-progress record | Sign-in identity reference from FEAT-29 |
| A setup step's owning screen reports completion | FEAT-15.SPEC-001 (Setup Wizard Shell), by way of any step's owning screen (FEAT-29, FEAT-27, FEAT-01.SPEC-002, FEAT-02.SPEC-001, FEAT-15.SPEC-002, FEAT-28.SPEC-001, FEAT-04.SPEC-001, or FEAT-18's subscribe screen) | Fires each time a step's owning screen signals its data was saved successfully | The step identifier that completed, and any data the automation needs to independently confirm completion (see Processing Logic) |
| Talia reaches the calendar-connection step and chooses "skip and connect later" | FEAT-04.SPEC-001 (Calendar Connection Setup), per FEAT-15.SPEC-006's optional-step rule | Fires when Talia makes the explicit skip choice on FEAT-04's own screen | Skip choice, timestamp |
| Talia returns to the wizard shell (any session after the first) | FEAT-15.SPEC-001 (Setup Wizard Shell) | Fires every time the shell loads for a Pro Account with an existing progress record | Current progress record |
| Platform Operator (Support) opens a Pro's account during a help request | FEAT-19 (Platform Support Read-Only Access) | Fires when Support requests a read of setup-progress state for a specific Pro Account | The same progress record, served read-only |

## Processing Logic

1. On first sign-in completion for a Pro Account with no existing progress record: create the Pro Account record (its identity fields are populated by FEAT-29 separately) and create its setup-progress state with all 8 steps marked incomplete except "account & sign-in," which is marked complete immediately (sign-in having just succeeded).
2. On a step-completion signal from any owning screen: verify the step's underlying data genuinely exists (for example, for the "profile" step, confirm the Pro Account's display_name and studio_address fields -- written by FEAT-27 -- are both populated; for the "services" step, confirm at least one Service record exists for this Pro Account; for the "payout" step, confirm a Payout Account record exists in at least Verification Pending status) before marking that step complete -- this automation never marks a step complete on a screen's say-so alone.
3. Mark the verified step complete in the progress record, with a timestamp.
4. If the step is the calendar-connection step and the signal is a skip choice, mark that step complete-as-skipped rather than complete-as-connected, per FEAT-15.SPEC-006.
5. Emit onboarding_step_completed with the step's name.
6. Re-derive the resume point: the first step, in the fixed order defined by FEAT-15.SPEC-006, that is not yet complete (complete-as-skipped counts as complete for this purpose).
7. Notify FEAT-15.SPEC-005 that a required step has changed state, so it can re-evaluate Go-Live readiness.
8. On a shell-load resume request: return the current progress record and the resume point computed in step 6, with every earlier step's saved answers intact (each step's owning feature retains its own data; this automation returns only completion state, not the underlying field values).
9. On a Support read request: return the same progress record read-only, with no action affordance attached, and log the view per FEAT-19's own logging behavior (this automation does not itself write the log entry; it only serves the data FEAT-19 logs having viewed).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Account and progress created | First sign-in completion for a new Pro Account | Pro Account and setup-progress records created; "account & sign-in" step marked complete | Wizard shell opens showing step 2 as current | FEAT-15.SPEC-001 |
| Step marked complete | A step's owning screen reports success and this automation independently verifies the underlying data exists | Progress record updated for that step | Wizard shell advances to the next step | FEAT-15.SPEC-001, FEAT-15.SPEC-005 |
| Step marked complete-as-skipped | The calendar step's skip choice is signaled | Progress record updated for the calendar step with a skipped flag | Wizard shell shows that step as Skipped rather than blocking | FEAT-15.SPEC-001, FEAT-15.SPEC-006 |
| Verification failed (no-op) | A step-completion signal arrives but the underlying data does not actually exist (for example, a signal fires but no Service record can be found) | No change to the progress record | The step is not advanced; Talia sees the step's owning screen show its own save error rather than a false "complete" state | FEAT-15.SPEC-001 (owning screen's own error handling) |
| Resume point computed | Talia returns to the wizard shell with an existing progress record | None (read-only computation) | Wizard shell opens directly at the first incomplete step with earlier answers preserved | FEAT-15.SPEC-001 |
| Support view served | Support requests a read during a help request | None (read-only) | Support sees the same progress state, with no action available | FEAT-19 |
| Automation failure | Processing error while marking a step complete or computing resume | No partial or inconsistent progress state is ever persisted (see Business Rules) | The triggering screen shows a generic "something went wrong, try again" retry state, consistent with every setup screen's own error handling | FEAT-15.SPEC-001 (owning screen's own error handling) |

## Data Model

**Reads:** Pro Account (display_name, studio_address existence check for the profile step, written by FEAT-27), Service (existence check for the services step), Availability Rule (existence check for the hours step), Cancellation Policy (existence check for the policy step, created by FEAT-15.SPEC-002), Payout Account (status check for the payout step), Calendar Connection (existence or skip-flag check for the calendar step), Subscription (Active status check for the subscription step) -- all read-only, existence/status checks only, per the dependency map's Referenced Entities table for FEAT-15.
**Creates:** Pro Account record (identity envelope; profile fields are written separately by FEAT-27, per the dependency map) and its setup-progress state, on first sign-in completion.
**Updates:** Pro Account's setup-progress state -- the per-step completion flags (complete / complete-as-skipped / incomplete) and timestamps, exclusively owned and written by this automation, per the Brief's Shared Context.
**Deletes:** None -- per the feature's own explicit non-goal, this automation never deletes or archives the Pro Account or its progress state.

## Business Rules

- This automation is the sole writer of the Pro Account's setup-progress state (Brief's Shared Context) -- no other spec, including the wizard shell itself, ever writes progress directly.
- A step is never marked complete on a screen's completion signal alone -- this automation independently verifies the underlying entity exists (or, for the calendar step, that an explicit skip choice was made) before recording completion, so a false-positive "complete" state can never occur from a screen bug.
- The calendar-connection step is the only step that can be marked complete-as-skipped; every other step requires genuine completion of its owning screen's flow, per FEAT-15.SPEC-006.
- Resume always lands on the first incomplete step in the fixed order defined by FEAT-15.SPEC-006 -- Talia is never forced to restart from step 1, and earlier steps' saved answers (held by their owning features) are never touched by a resume.
- Platform Operator (Support) reads this same progress state read-only through FEAT-19, and can never act on a step, per SC-05 and XBR-24.

## Edge Cases

- **A step's owning screen signals completion, but the underlying record was deleted or never actually saved (e.g., a race between two tabs)** -- Step 2 of Processing Logic's independent verification catches this: the step is not marked complete, and the owning screen's own error handling applies rather than the wizard silently advancing on a false signal.
- **Talia completes the same step twice (e.g., adds a second service after the services step was already marked complete)** -- No effect on progress: the step is already marked complete, and this automation does not re-fire or duplicate the completion record; the additional service is simply additional data owned by FEAT-01.
- **Talia abandons setup for weeks and returns** -- The resume point is recomputed exactly as it would be for a same-day return; no step decays or times out, consistent with the feature's own "never forcing a restart" alternate flow.
- **Talia skips the calendar step, then connects a calendar later from settings** -- The progress record's calendar-step flag updates from complete-as-skipped to complete-as-connected; this has no effect on Go-Live readiness (FEAT-15.SPEC-007 treats both as satisfying the optional step) but is reflected accurately for Support's read-only view.
- **Concurrent trigger firing (Talia completes two different steps on two devices at nearly the same time)** -- Each completion signal touches a disjoint field in the progress record (one step's flag each); both are recorded independently with no overwrite, consistent with the Pro Account entity's last-write-wins resolution for non-overlapping fields in the dependency map's Contention note.
- **Trigger fires while a previous run is in flight (two completion signals for the same step arrive close together, e.g., a double-submitted save)** -- The second signal for an already-complete step is a no-op (see the "completes the same step twice" case above); the automation's per-step verification is idempotent, so re-running it against an already-satisfied condition produces the same "complete" outcome without side effects.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29 (Pro Sign-In & Account Lifecycle) | Triggered by (inbound) | First sign-in completion triggers account and progress creation |
| FEAT-15.SPEC-001 (Setup Wizard Shell) | Triggered by (inbound), Affects (outbound) | Every step-completion signal and shell-load resume request flows through this automation; the shell reflects this automation's output |
| FEAT-04.SPEC-001 (Calendar Connection Setup) | Triggered by (inbound) | The explicit skip choice signals complete-as-skipped for the calendar step |
| FEAT-15.SPEC-006 (Setup Step Order & Optional-Step Rules) | References (inbound) | Defines the fixed step order this automation tracks against and the calendar step's skip semantics |
| FEAT-15.SPEC-005 (Go-Live Evaluation & Booking Link Activation) | Affects (outbound) | Notified after every progress change so it can re-evaluate readiness |
| FEAT-19 (Platform Support Read-Only Access) | Affects (outbound) | Serves the same progress record read-only for Support's help-request view |

## Analytics and Success Signals

- **onboarding_started** (none) -- supports success-metrics.md: "Setup-to-Live-Link Completion"
- **onboarding_step_completed** (step name, whether connected or skipped where applicable) -- supports success-metrics.md: "Setup-to-Live-Link Completion"
- **onboarding_completed** (total elapsed time from onboarding_started, count of steps skipped) -- supports success-metrics.md: "Setup-to-Live-Link Completion"
- **onboarding_referral_source** -- N/A -- no field in the Pro Account or any entity this automation reads captures how a new Pro heard about Chairtime; success-metrics.md's "Peer-Referral Growth Share" (also connected to this feature) has no in-product signal available anywhere in FEAT-15 to feed it and is tracked by the founder outside the product (for example, by asking new pros directly), not by an automation event.

## Acceptance Criteria

**FEAT-15.SPEC-004-AC-01:** Given Talia completes sign-in for the very first time, when this automation fires, then a Pro Account record and its setup-progress state are created with the "account & sign-in" step already marked complete.

**FEAT-15.SPEC-004-AC-02:** Given Talia has just added her first service on FEAT-01.SPEC-002, when that screen reports completion, then this automation verifies a Service record exists, marks the services step complete, and emits onboarding_step_completed with "services."

**FEAT-15.SPEC-004-AC-03:** Given a step-completion signal arrives for the payout step but no Payout Account record can be found, when this automation runs its verification, then the step is not marked complete and no false progress is recorded.

**FEAT-15.SPEC-004-AC-04:** Given Talia chooses "skip and connect later" on the calendar step, when FEAT-04.SPEC-001 signals the skip, then this automation marks the calendar step complete-as-skipped.

**FEAT-15.SPEC-004-AC-05:** Given Talia has completed steps 1 through 4 and closes the app, when she signs back in days later, then the wizard shell resumes exactly at step 5 with steps 1 through 4 shown complete.

**FEAT-15.SPEC-004-AC-06:** Given Talia skipped the calendar step during setup and later connects a calendar from settings, when the connection succeeds, then this automation updates the calendar step's flag from complete-as-skipped to complete-as-connected.

**FEAT-15.SPEC-004-AC-07:** Given Platform Operator (Support) opens Talia's account during a help request, when they view her setup progress via FEAT-19, then they see the same progress state read-only, with no action available to them.

**FEAT-15.SPEC-004-AC-08:** Given Talia completes two different steps on two different devices at nearly the same time, when both completion signals are processed, then both steps are recorded as complete with no overwrite of the other.

**FEAT-15.SPEC-004-AC-09:** Given Talia's services step is already marked complete, when a duplicate completion signal for that same step arrives, then the automation makes no change and does not emit a second onboarding_step_completed event for that step.

**FEAT-15.SPEC-004-AC-10:** Given every required step has become complete, when the last one is recorded, then this automation notifies FEAT-15.SPEC-005 to re-evaluate Go-Live readiness.

**FEAT-15.SPEC-004-AC-11:** Given Talia has never used a calendar connection and setup otherwise completes, when readiness is evaluated, then the calendar step's incomplete/skipped state has no bearing on any other step's recorded completion.

**FEAT-15.SPEC-004-AC-12:** Given a completion signal for a step whose underlying data existed a moment before but was deleted before verification ran, when this automation processes the signal, then the step remains marked incomplete and the owning screen's own retry-capable error handling governs what Talia sees next.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 5 | 5 |
| Outcome Paths | 7 | 7 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
