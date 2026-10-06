---
document_type: spec
spec_type: screen
spec_id: FEAT-04.SPEC-002
spec_name: Milestone Timeline (Client View)
spec_slug: milestone-timeline-client-view
parent_feature: FEAT-04
parent_feature_name: Milestone & Payment Schedule Setup
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

# Screen Spec: Milestone Timeline (Client View)

## Overview

**Name:** Milestone Timeline (Client View)
**ID:** FEAT-04.SPEC-002
**Type:** Screen
**Purpose:** Read-only display of a project's milestone timeline within each viewer's own project view.
**Parent Feature:** FEAT-04 -- Milestone & Payment Schedule Setup

## Scope and Non-Goals

**In Scope:**
- A read-only, ordered list of a project's milestones: name, order, price or no-charge status, payment-trigger indicator, target date (in the viewer's own time zone), and current status
- Role-dependent navigation from a milestone row: Nadia and Dana to the deliverable list (FEAT-06.SPEC-002); Owen and Priya to the milestone comment thread (FEAT-07.SPEC-002) or, once a deliverable is ready, the review and approval screen (FEAT-08.SPEC-001)
- Reflecting the project's current milestone and schedule state on every fresh read

**Non-Goals:**
- Editing milestones or the payment schedule -- owned exclusively by FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor); the Brief's Internal Dependency Map routes only Nadia to editing, since this screen serves View/Own-only roles per the Access Matrix.
- Commenting on or approving a milestone from this screen -- deliverable- and milestone-level comments are owned by FEAT-07 (Deliverable Review & Feedback) and approval by FEAT-08 (Milestone Approval); this timeline links out to those specs rather than embedding their controls.
- Displaying deliverable files, previews, or comment threads inline -- owned by FEAT-06 and FEAT-07; this screen shows only the milestone list, not its attached deliverables.
- Displaying invoice amounts, status, or payment history -- owned by FEAT-09 (Invoice Generation & Sending) and FEAT-10 (Invoice Payment Processing); a milestone's payment_trigger is shown here only as "issues an invoice on approval" or "no trigger," never as an invoice's own detail.
- Recurring or retainer billing calendar display -- excluded per scope-boundaries.md (SC-14), consistent with FEAT-04.SPEC-001's non-goal; the schedule is limited to deposit, per-milestone, on completion, or a mix.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-05.SPEC-003 (Portal Home) | Owen or Priya opens the milestones area of one of their own company's projects | Project reference, scoped to their own client company |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Dana opens the milestones area while working through a freelancer's screens in a read-only support session | Project reference, within the freelancer account under support |
| FEAT-01.SPEC-005 (Project Detail / Open Project) | Nadia opens the same read-only timeline view from her own project detail (an alternative to editing directly) | Project reference |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen -- the same read-only milestone data she can also edit via FEAT-04.SPEC-001 | No editing controls on this screen (view only); she uses FEAT-04.SPEC-001 to make changes | -- |
| Owen (Client Primary Contact) | Full screen, scoped to his own company's projects (Own-only) | Navigates into a milestone's comment thread (FEAT-07.SPEC-002) or, once a deliverable is ready, its approval screen (FEAT-08.SPEC-001) | -- |
| Priya (Client Reviewer Contact) | Full screen, scoped to her own company's projects (Own-only) | Navigates into a milestone's comment thread (FEAT-07.SPEC-002) or, once a deliverable is ready, its review screen (FEAT-08.SPEC-001); no Approve control is ever shown, per the Access Matrix | -- |
| Dana (Support Operator) | Full screen, read-only, inside a logged support session (FEAT-31) | No -- this screen has no write controls to disable; navigation into linked specs is likewise read-only there | -- |
| Unauthenticated | No | No | Redirected to the magic-link sign-in request (FEAT-05.SPEC-001) |
| Expired session | No | No | Shown the expired/invalid-link explanation with a one-tap way to request a fresh link (FEAT-05.SPEC-002) |

This screen has no write controls for any role, so each of the four named roles above has no restricted element and shows "--"; only the two connectivity/authentication states below deny reaching the screen at all.

## Layout and Content

**Header:** Project name and its current stage badge (Draft, In Progress, Complete, Cancelled, Archived, per FEAT-01's stage derivation).

**Body:** An ordered, read-only list of milestone rows in the same visual pattern as FEAT-04.SPEC-001's editor rows, per the Brief's Shared UI Patterns, differing only in that no control is interactive for editing. Each row shows: position number, Name, Price or a "No separate charge" label, a Payment Trigger tag ("Issues an invoice on approval" or none shown), Target Date (converted to the viewer's own time zone, per FEAT-15.SPEC-006), and a Status badge (Defined, Deliverable Uploaded, Approved, or Reopened). A milestone row is tappable, and its destination depends on the viewer's role and the milestone's readiness. Once a deliverable is ready for review (status Deliverable Uploaded or Reopened), Owen and Priya open the Milestone Review & Approval Screen (FEAT-08.SPEC-001); Priya's rendering there never includes an Approve control. When no deliverable is ready yet (status Defined, or Approved with nothing awaiting review), Owen and Priya open the Milestone Comment Thread (FEAT-07.SPEC-002), the client-side milestone view. Nadia and Dana open the milestone's deliverable list (FEAT-06.SPEC-002), because the deliverable list is not part of the client portal (FEAT-06.SPEC-002 denies Owen and Priya).

**Footer:** None.

### Responsive Behavior

- **Compact breakpoint:** Rows stack the same two-line layout as the editor's compact view (name/status on line 1, price/trigger/date on line 2), full width.
- **Medium size class and above:** Each milestone renders as a single-line row with all fields visible, matching the editor's medium-and-above layout minus the edit and reorder controls.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Milestone row (Owen or Priya, deliverable ready for review) | Tap | Navigate to the Milestone Review & Approval Screen (FEAT-08.SPEC-001); Priya sees it without an Approve control | Screen transitions | Standard navigation transition |
| Milestone row (Owen or Priya, no deliverable ready yet) | Tap | Navigate to the Milestone Comment Thread (FEAT-07.SPEC-002) for that milestone | Screen transitions | Standard navigation transition |
| Milestone row (Nadia or Dana) | Tap | Navigate to that milestone's deliverable list (FEAT-06.SPEC-002); Dana's rendering there is read-only | Screen transitions | Standard navigation transition |
| Status badge, Price, Target Date, Payment Trigger tag | -- | Display only | None | Non-interactive; provides read-only context within the row |

### Accessibility Notes

- **Focus order:** Header (project name, stage badge) -> each milestone row in list order, each announced with its name, status, and payment-trigger state before the row's tap target.
- **Live-updated content:** Because this is a standard fetched screen rather than a live-updating one (see States below), no dynamic-update announcement is needed mid-session; a fresh load announces the full list as a single region.
- **Keyboard alternatives:** Every milestone row is reachable and activatable by keyboard; there are no pointer-only gestures on this screen.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Empty (no milestones yet) | Plain message: "No milestones have been set up for this project yet." | Project has no milestones defined | Nadia defines the first milestone in FEAT-04.SPEC-001; the viewer's next visit shows the populated list |
| Loading | Real, incremental loading progress for the milestone list, per the Brief's Non-Functional Notes (never an indefinite blank state) | Screen is opened and the list is being fetched | Data finishes loading, successfully or with an error |
| Error | Error banner: "Couldn't load the milestone timeline. Try again." with a Retry button | The fetch fails | Viewer taps Retry and the fetch succeeds |
| Offline/Degraded | Banner: "You're offline. Showing the last milestone timeline you loaded." if a prior load is cached; otherwise the Error state's message with a connectivity-specific note | Connectivity is lost while the screen is open or being opened | Connectivity restored -- the screen re-fetches and shows the current state |

## Validation Rules

Not applicable -- this is a read-only display screen with no user input.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Milestone row tap (Nadia or Dana, any milestone status) | FEAT-06.SPEC-002 (Deliverable List & Management) | FEAT-06 |
| Milestone row tap (Owen or Priya, no deliverable ready for review yet) | FEAT-07.SPEC-002 (Milestone Comment Thread) | FEAT-07 |
| Milestone row tap (Owen or Priya, deliverable ready for review) | FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | FEAT-08 |
| Back navigation | FEAT-05.SPEC-003 (Portal Home) or FEAT-01.SPEC-005 (Project Detail), depending on entry role | FEAT-05 or FEAT-01 |

## Data Model

**Creates:** None -- this screen performs no writes.

**Reads:** Milestone -- name, order, price/no_separate_charge flag, payment_trigger, target_date, status, for every milestone in the current project. Payment Schedule -- read indirectly through each milestone's payment_trigger value; the schedule's structure and amounts are never displayed as a standalone record on this screen.

**Updates:** None.

**Deletes:** None.

## Business Rules

- This screen never writes to Milestone or Payment Schedule; the dependency map's Contention notes for those entities apply only to their writers (FEAT-04.SPEC-001, FEAT-06, FEAT-08), not to this read-only view.
- XBR-09: Client isolation -- Owen and Priya reach only their own client company's milestone timeline; an out-of-scope attempt shows a plain explanation and a fresh-link option, never another company's data.
- The Shared UI Pattern from the Brief applies: this screen's milestone row shows the same information and ordering as FEAT-04.SPEC-001's editor row, differing only in that no control here is interactive for editing.
- Target dates render in each viewer's own time zone per FEAT-15.SPEC-006, independent of the time zone in which Nadia originally set the date.

## Edge Cases

- **Nadia adds or edits a milestone while Owen has this screen open** -- No live update occurs; this is a standard fetched screen, not a live-updating one. Owen sees the change only on his next fresh load or navigation back to this screen (cross-spec: Side-Effect Inventory, "Milestones/schedule are created or changed -> the timeline view reflects the current state on next read").
- **A milestone's status changes to Approved while Priya is viewing this screen** -- No live update; her next load shows the current Approved status and, since she is a Reviewer, still shows no Approve control.
- **A project has an accepted proposal but zero milestones** -- The Empty state message is shown rather than an error, since this is a valid, expected interim state before Nadia sets up the schedule.
- **Owen is a contact for two different freelancers and opens this screen from each portal separately** -- Each portal shows only that freelancer's own project's milestones; no cross-freelancer data ever appears together (XBR-09, dependency map Client Contact entry).
- **Dana opens this screen mid-support-session and the freelancer's account has milestones spanning multiple currencies across projects** -- Each project's milestone list is shown independently per project; no cross-project aggregation occurs on this screen.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | References (inbound) | Shares the same milestone row data and ordering; edits made there appear here on the next read |
| FEAT-04.SPEC-004 (Milestone Reorder Recalculation) | References (inbound) | The renumbered order produced there is reflected here on the next read |
| FEAT-05.SPEC-003 (Portal Home) | Navigation (inbound) | Entry point for Owen and Priya |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Navigation (inbound) | Entry point for Dana during a support session |
| FEAT-01.SPEC-005 (Project Detail / Open Project) | Navigation (inbound) | Alternative entry point for Nadia's own read-only view |
| FEAT-06.SPEC-002 (Deliverable List & Management) | Navigation (outbound) | A milestone row navigates Nadia and Dana into its deliverable list; never used for Owen or Priya |
| FEAT-07.SPEC-002 (Milestone Comment Thread) | Navigation (outbound) | A milestone row navigates Owen and Priya into the milestone-level thread when no deliverable is ready yet |
| FEAT-08.SPEC-001 (Milestone Review & Approval Screen) | Navigation (outbound) | A milestone row navigates Owen and Priya into review once a deliverable is ready; only Owen has the Approve control there |
| FEAT-15.SPEC-006 (Time Zone & Local Date/Time Display Rule) | References (outbound) | Governs how each milestone's target date is rendered per viewer |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| milestone_timeline_viewed | viewer_role (Owen / Priya / Dana / Nadia), milestone_count | The screen finishes loading successfully | N/A -- this screen only displays milestone/schedule state; success-metrics.md's "Milestone Schedule Completeness" is driven by the write events emitted in FEAT-04.SPEC-001, not by viewing here |

## Acceptance Criteria

**FEAT-04.SPEC-002-AC-01:** Given Owen opens the milestones area of his own project from Portal Home, when the screen loads, then he sees an ordered, read-only list of that project's milestones with name, price/no-charge, payment-trigger tag, target date, and status.

**FEAT-04.SPEC-002-AC-02:** Given Priya opens the same project's milestone timeline, when the screen loads, then she sees the identical list Owen sees, with no edit controls and no Approve control on any row.

**FEAT-04.SPEC-002-AC-03:** Given Dana is in a logged support session on a freelancer's account, when she opens a project's milestone timeline, then she sees the same read-only list with no interactive controls available.

**FEAT-04.SPEC-002-AC-04:** Given Nadia opens this read-only view of one of her own projects, when the screen loads, then she sees the same milestone data she can edit in FEAT-04.SPEC-001, with no editing controls on this screen.

**FEAT-04.SPEC-002-AC-05:** Given a milestone's target date is set by Nadia in her own time zone, when Owen views this screen, then the date displays converted to Owen's own time zone per FEAT-15.SPEC-006.

**FEAT-04.SPEC-002-AC-06:** Given Owen taps a milestone row whose deliverable is ready for his review, when the tap registers, then he is navigated to the Milestone Review & Approval Screen (FEAT-08.SPEC-001).

**FEAT-04.SPEC-002-AC-07:** Given Priya taps a milestone row whose deliverable is ready for review, when the tap registers, then she is navigated to the Milestone Review & Approval Screen (FEAT-08.SPEC-001), where no Approve control is rendered, and never to the deliverable list (FEAT-06.SPEC-002).

**FEAT-04.SPEC-002-AC-08:** Given a project has an accepted proposal but no milestones defined, when Owen opens this screen, then he sees the message "No milestones have been set up for this project yet." rather than an error.

**FEAT-04.SPEC-002-AC-09:** Given the milestone list is loading on a slow connection, when the screen is open, then real incremental loading progress is shown rather than an indefinite blank state.

**FEAT-04.SPEC-002-AC-10:** Given the milestone list fails to load, when the failure occurs, then an error banner with a Retry button appears, and tapping Retry re-fetches the list.

**FEAT-04.SPEC-002-AC-11:** Given Priya loses connectivity while this screen is open with data already loaded, when connectivity drops, then a banner shows "You're offline. Showing the last milestone timeline you loaded." and the previously loaded list remains visible.

**FEAT-04.SPEC-002-AC-12:** Given Nadia adds a milestone in FEAT-04.SPEC-001 while Owen already has this screen open, when Owen does not refresh, then his view does not update live; when he navigates back to this screen or reloads it, then the new milestone appears.

**FEAT-04.SPEC-002-AC-13:** Given Owen is a contact for two different freelancers, when he opens this milestone timeline from each freelancer's portal separately, then each shows only that freelancer's own project data, with no cross-freelancer milestones ever appearing together.

**FEAT-04.SPEC-002-AC-14:** Given an unauthenticated visitor attempts to load this screen's URL directly, when the page attempts to render, then they are redirected to the magic-link sign-in request (FEAT-05.SPEC-001).

**FEAT-04.SPEC-002-AC-15:** Given a milestone has status Defined with no deliverable ready for review, when Owen taps its row, then he is navigated to the Milestone Comment Thread (FEAT-07.SPEC-002) for that milestone and not to FEAT-06.SPEC-002.

**FEAT-04.SPEC-002-AC-16:** Given a milestone has status Defined with no deliverable ready for review, when Priya taps its row, then she is navigated to the Milestone Comment Thread (FEAT-07.SPEC-002), and no denial message from FEAT-06.SPEC-002 is ever shown to her.

**FEAT-04.SPEC-002-AC-17:** Given Nadia opens this view of her own project and a milestone has no deliverable ready, when she taps its row, then she is navigated to that milestone's deliverable list (FEAT-06.SPEC-002).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 4 | 4 |
| States | 4 (empty, loading, error, offline/degraded) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
