---
document_type: spec
spec_type: screen
spec_id: FEAT-25.SPEC-002
spec_name: Mark Project Cancelled Screen
spec_slug: mark-project-cancelled-screen
parent_feature: FEAT-25
parent_feature_name: Refund & Cancelled Project Handling
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

# Screen Spec: Mark Project Cancelled Screen

## Overview

**Name:** Mark Project Cancelled Screen
**ID:** FEAT-25.SPEC-002
**Type:** Screen
**Purpose:** Nadia marks a project Cancelled, entering an optional reason, from the project detail view, without deleting any project history.
**Parent Feature:** FEAT-25 -- Refund & Cancelled Project Handling

## Scope and Non-Goals

**In Scope:**
- Entering a project cancellation with an optional freelancer-entered reason
- Explaining plainly, before confirming, that cancellation preserves every existing record (proposal, milestones, deliverables, invoices, activity trail) rather than removing anything
- Submitting the cancellation for recording (FEAT-25.SPEC-004)
- Reflecting the project's resulting Cancelled stage once the project detail screen (FEAT-01.SPEC-005) is reopened

**Non-Goals:**
- Deleting the project or any of its records -- excluded per the dependency map (Entity: Project, Lifecycle): no in-product project delete exists anywhere except account deletion (FEAT-24); this screen offers cancellation only, never removal
- Marking an invoice refunded -- handled by FEAT-25.SPEC-001 (Mark Invoice Refunded Screen); this screen acts on a project as a whole, not an individual invoice
- Reversing a cancellation -- the product defines no "un-cancel" action; a project cancelled in error is a business situation resolved outside this screen (the freelancer contacting the client directly), consistent with the feature's focus on recording outcomes truthfully rather than editing history
- Validating who may act or the concurrency behavior -- both governed entirely by FEAT-25.SPEC-006 (Authorization and Validation Rules); this screen only surfaces the outcome
- Applying the cancellation to the Project record -- owned entirely by FEAT-25.SPEC-004 (Project Cancellation Recording); this screen only submits the entry and displays the result

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-005 (Project Detail) | Nadia selects "Cancel Project" from the project detail's overflow menu; the entry appears only while the project's derived stage is none of Cancelled, Archived, or Complete | Project reference |

**Cross-feature note:** FEAT-01.SPEC-005's overflow menu, as currently specified, lists Archive and Mark Complete; it lists FEAT-25 only as a "References (inbound)" connection ("Owns the Cancelled transition this screen only displays") with no corresponding outbound navigation entry yet. This screen is written per the Brief's Internal Dependency Map, which names the project detail owned by FEAT-01 as this screen's default entry (feature-overview.md, Default Entry). The missing "Cancel Project" navigation entry on FEAT-01.SPEC-005's side is flagged here as a bidirectional-reference gap for the Cross-Reference Reconciler (Pass D) to resolve when FEAT-01's specs are next revised; it is outside this spec's authority to edit another feature's file.

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Enter and submit a cancellation with an optional reason | -- |
| Owen (Client Primary Contact) | No | No | Not reachable from Owen's portal navigation; he sees the resulting Cancelled status on his own project through the client portal (Own-only, view), never this entry screen |
| Priya (Client Reviewer Contact) | No | No | Not reachable; the Client & Project Management capability group is None for Reviewer contacts |
| Dana (Support Operator) | No | No | Not reachable -- Dana never reaches this screen. Inside a logged support session (FEAT-31) she sees only the resulting Cancelled status, read-only, on the project detail screen (FEAT-01.SPEC-005), where the "Cancel Project" entry is not rendered in her session; a direct link to this screen opens FEAT-01.SPEC-005 in her read-only session with no error message |
| Unauthenticated | No | No | Redirected to sign-in; after signing in, Nadia lands on the project she was trying to cancel if the link carried the reference, otherwise on FEAT-01.SPEC-003 (Client & Project Roster) |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- an in-progress reason entry is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Screen title "Cancel Project" with a back arrow (returns to FEAT-01.SPEC-005, Project Detail) and a "Confirm Cancellation" action button (right-aligned).

**Body, in order:**
- **Project summary (read-only):** Project name and client name -- the same identity block shown on FEAT-01.SPEC-005, so Nadia can confirm she is cancelling the right project.
- **Preservation notice:** A fixed explanatory line: "Cancelling stops work on this project but keeps its full history -- proposal, milestones, deliverables, invoices, and activity -- exactly as it is. Nothing is deleted."
- **Reason (optional):** A multi-line text input for a freelancer-entered note about why the project is being cancelled.

**Footer:** None -- "Confirm Cancellation" is in the header.

### Responsive Behavior

- **Compact size class:** Single-column form as described above, full width; the header action remains reachable at the top of the screen.
- **Medium size class and above:** Same single-column form, capped at a consistent platform-wide form width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-005 (Project Detail) | Screen closes | Standard navigation transition |
| Reason input | Type | Captures the optional note | Field shows entered text | Standard input state |
| "Confirm Cancellation" button | Tap | 1. Show an inline confirmation step (see States: Confirming). 2. On confirming a second time, trigger FEAT-25.SPEC-004 (Project Cancellation Recording). | Button shows a loading state during submission after the second confirmation | Success: toast "Project cancelled" and navigate to FEAT-01.SPEC-005 showing the Cancelled stage badge. Failure: inline error per the Error state below. |
| "Confirm Cancellation" button (while loading) | Tap | No action -- debounced | None | Button remains in loading state |

### Accessibility Notes

- **Focus order:** Back arrow -> Reason input -> Confirm Cancellation.
- **Confirmation announcements:** The confirming-step dialog text is announced to assistive technology when it appears; the success toast is announced on completion.
- **Validation announcements:** A stale-state rejection dialog receives focus immediately when it appears.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton layout for the project summary and form | Screen opens | The project's name and client finish loading |
| Loaded (default) | Form shown with Reason empty; Confirm Cancellation enabled | Screen opens on an eligible project | Nadia edits the reason or taps Confirm Cancellation |
| Confirming | A blocking dialog restates the preservation notice and asks "Cancel this project? This cannot be undone from here." with "Cancel Project" and "Keep Editing" options | Nadia taps Confirm Cancellation the first time | Nadia confirms a second time (submits) or dismisses (returns to Loaded) |
| Submitting | "Confirm Cancellation" button shows a loading spinner; the reason input is disabled | Nadia confirms the dialog | FEAT-25.SPEC-004 completes or fails |
| Error | Error banner at the top of the form: "Couldn't cancel this project. Try again." with a Retry button; the entered reason is preserved | FEAT-25.SPEC-004 reports a failure | Nadia taps Retry and the submission succeeds |
| Offline/Degraded | Banner "You're offline -- this cancellation can't be recorded until you reconnect." at the top; the form remains editable but Confirm Cancellation is disabled | Connectivity lost while this screen is open | Connectivity restored -- Confirm Cancellation re-enables; nothing is queued, since a cancellation must persist immediately as part of the record (assumptions-constraints.md ASMP-27) |

## Validation Rules

Validation governed by FEAT-25.SPEC-006 (Refund, Cancellation & Reversal Authorization and Validation Rules). See that spec for the reject-with-refresh concurrency behavior applied to this screen's submission. This screen has no field-level format validation beyond the optional reason's presence being unconstrained -- an empty reason is always valid.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Back arrow tap | FEAT-01.SPEC-005 (Project Detail) | FEAT-01 |
| Successful "Confirm Cancellation" submission | FEAT-01.SPEC-005 (Project Detail), showing the Cancelled stage | FEAT-01 |
| Retry after Error state succeeds | FEAT-01.SPEC-005 (Project Detail), showing the Cancelled stage | FEAT-01 |
| "Keep Editing" on the confirming dialog | Returns to the Loaded state on this same screen | -- |

## Data Model

**Creates:** None -- this screen creates no new record; the cancellation is applied to the existing Project by FEAT-25.SPEC-004.
**Reads:** Project -- project_name, client (owning client's name), `stage` (must resolve to a state this screen can act on -- see Business Rules).
**Updates:** None directly -- submission hands the entered reason to FEAT-25.SPEC-004, which sets `stage` to Cancelled and stamps `cancelled_at`.
**Deletes:** None.

## Business Rules

- This screen is reachable only for a project whose derived stage is none of Cancelled, Archived, or Complete (FEAT-25.SPEC-006). A project already Cancelled, Archived, or Complete offers no "Cancel Project" entry on FEAT-01.SPEC-005: Complete and Cancelled are mutually exclusive terminal states (FEAT-01.SPEC-011 precedence order), so a project Nadia has marked Complete is not cancelled afterward. Every other derived stage can be cancelled.
- Submission triggers FEAT-25.SPEC-004, which is the sole owner of the Project's `stage` and `cancelled_at` fields; this screen never writes those fields itself.
- Cancellation preserves every existing Proposal, Payment Schedule, Milestone, Deliverable, Invoice, and Activity Log Entry unchanged (XBR-25); this screen's preservation notice states that plainly before Nadia confirms.
- An unaccepted proposal on the project stays open until Nadia separately revises and re-sends it, or the project is cancelled -- cancelling here does not itself change the Proposal record (XBR-25); FEAT-02 owns any further action on it.

## Edge Cases

- **Nadia navigates away with an entered reason unsaved** -- No confirmation dialog beyond the Confirming step's own dialog is shown for simply leaving the reason field; the form discards the entry since nothing is written until Confirm Cancellation succeeds.
- **Nadia taps Confirm Cancellation twice rapidly on the confirming dialog** -- The second tap is ignored while the first submission is in progress (button in loading state).
- **The project's state changed since this screen loaded (a second session marked it Complete, Archived, or already Cancelled)** -- Submission is rejected with "This project's state changed since you opened this page. Refresh to see the latest state." and a "Refresh" action reloads the project's current state; this is the concurrent-edit conflict behavior for this screen's update to the shared Project entity, per the dependency map's Contention note for Project (reject-with-refresh) and FEAT-25.SPEC-006.
- **A milestone approval or proposal acceptance fires for this same project at the same moment Nadia confirms cancellation** -- Nadia's explicit Cancelled transition takes precedence and is never overwritten by a system-driven stage change (dependency map, Project Contention: "System-driven stage changes... never overwrite a freelancer's explicit Complete or Cancelled"); the system-driven event still records normally (e.g., the milestone is still Approved), but the project's stage shows Cancelled.
- **Network failure during submission** -- The Error state's banner and Retry button appear; the entered reason is preserved, and the project's prior stage remains authoritative until the retry succeeds.
- **Nadia cancels a project that has open, unpaid invoices** -- No blocking check exists here (unlike Archive's open-items confirmation on FEAT-01.SPEC-007); cancellation is a record of stopped work, not a financial reconciliation step, so open invoices remain exactly as they were and can still be refunded separately through FEAT-25.SPEC-001 if needed.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-005 (Project Detail) | Navigation (inbound/outbound) | Entry point via "Cancel Project"; destination after submission or cancellation (see Entry Points cross-feature note on the current gap in FEAT-01.SPEC-005's own navigation) |
| FEAT-25.SPEC-004 (Project Cancellation Recording) | Triggers (outbound) | Confirm Cancellation submits the entry for recording |
| FEAT-25.SPEC-006 (Refund, Cancellation & Reversal Authorization and Validation Rules) | References (inbound) | Authorization and concurrency rules applied to who may reach this screen and how a stale submission is handled |
| FEAT-01.SPEC-011 (Project Stage Derivation) | References (outbound) | The Cancelled stage this screen produces is read and displayed by that spec's derivation formula |
| FEAT-31 (Operator Support Access) | References (inbound) | Owns Dana's read-only support session, in which she sees the resulting Cancelled status on FEAT-01.SPEC-005; no navigation from FEAT-31 into this screen exists |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| cancel_project_screen_opened | project stage at open | Screen loads on an eligible project | N/A -- no success-metrics.md metric is connected to this feature; retained so cancellation entry is observable rather than invisible |
| cancel_project_submitted | reason provided (yes / no) | Nadia confirms cancellation a second time and submission proceeds | N/A -- no success-metrics.md metric is connected to this feature; the outcome is measured downstream by FEAT-25.SPEC-004's own signals |
| cancel_project_submission_rejected | reason (stale_state / network_failure) | Submission fails for any reason | N/A -- no success-metrics.md metric is connected to this feature; retained so cancellation friction is observable rather than silent |

## Acceptance Criteria

**FEAT-25.SPEC-002-AC-01:** Given Nadia is on the Project Detail screen (FEAT-01.SPEC-005), when she selects "Cancel Project" from the overflow menu, then she lands on this screen with the project's name and client shown and the reason field empty.

**FEAT-25.SPEC-002-AC-02:** Given Nadia is on this screen, when she taps Confirm Cancellation, then a blocking dialog appears restating the preservation notice and asking "Cancel this project? This cannot be undone from here." with "Cancel Project" and "Keep Editing" options.

**FEAT-25.SPEC-002-AC-03:** Given Nadia sees the confirming dialog, when she taps "Keep Editing", then the dialog closes and she returns to the Loaded state with her entered reason intact.

**FEAT-25.SPEC-002-AC-04:** Given Nadia sees the confirming dialog, when she taps "Cancel Project" to confirm, then FEAT-25.SPEC-004 records the project as Cancelled and she sees a toast "Project cancelled" before returning to FEAT-01.SPEC-005 showing the Cancelled stage badge.

**FEAT-25.SPEC-002-AC-05:** Given Nadia enters an optional reason, when she confirms cancellation, then the reason is recorded alongside the cancellation by FEAT-25.SPEC-004.

**FEAT-25.SPEC-002-AC-06:** Given Nadia leaves the reason field empty, when she confirms cancellation, then the cancellation is still recorded with no reason stored, since the reason is optional.

**FEAT-25.SPEC-002-AC-07:** Given Nadia confirms cancellation and the submission fails due to a network error, then the Error banner "Couldn't cancel this project. Try again." appears with her entered reason preserved.

**FEAT-25.SPEC-002-AC-08:** Given Nadia taps Confirm Cancellation twice rapidly on the confirming dialog, then the second tap has no effect while the first submission is in progress.

**FEAT-25.SPEC-002-AC-09:** Given the project's state changed in another session since Nadia opened this screen (it was marked Complete, Archived, or already Cancelled), when she confirms cancellation, then the submission is rejected with "This project's state changed since you opened this page. Refresh to see the latest state."

**FEAT-25.SPEC-002-AC-10:** Given a milestone is approved on this same project at the same moment Nadia confirms cancellation, then the milestone approval still records normally and the project's stage shows Cancelled, since Nadia's explicit cancellation is never overwritten by a system-driven stage change.

**FEAT-25.SPEC-002-AC-11:** Given the project has open, unpaid invoices, when Nadia cancels it, then no blocking confirmation about the open invoices appears and the invoices remain unchanged.

**FEAT-25.SPEC-002-AC-12:** Given Nadia loses connectivity while this screen is open, when she looks at the screen, then the banner "You're offline -- this cancellation can't be recorded until you reconnect." appears and Confirm Cancellation is disabled.

**FEAT-25.SPEC-002-AC-13:** Given Owen (Client Primary Contact) attempts to reach this screen directly, then it is not reachable from his portal navigation and no such control exists there.

**FEAT-25.SPEC-002-AC-14:** Given Dana (Support Operator) is in a logged support session viewing a project Nadia has marked Cancelled, when she opens the project detail screen (FEAT-01.SPEC-005), then she sees the Cancelled status read-only, no "Cancel Project" entry is rendered, and she has no path to this screen; a direct link to this screen opens FEAT-01.SPEC-005 in her read-only session with no error message.

**FEAT-25.SPEC-002-AC-15:** Given Nadia's session expires while she has a reason entered, when she re-authenticates, then the entered reason is restored on this screen.

**FEAT-25.SPEC-002-AC-16:** Given Nadia selects "Cancel Project", when this screen opens, then a skeleton layout appears for the project summary and form until the project's name and client finish loading.

**FEAT-25.SPEC-002-AC-17:** Given a project whose derived stage is Complete, Archived, or already Cancelled, when Nadia opens its project detail screen (FEAT-01.SPEC-005), then no "Cancel Project" entry is offered and this screen cannot be reached from that project.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 6 (loading, loaded, confirming, submitting, error, offline) | 6 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
