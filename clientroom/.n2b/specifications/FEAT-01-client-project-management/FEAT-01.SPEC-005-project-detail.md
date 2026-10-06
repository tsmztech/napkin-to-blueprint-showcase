---
document_type: spec
spec_type: screen
spec_id: FEAT-01.SPEC-005
spec_name: Project Detail (Open Project)
spec_slug: project-detail
parent_feature: FEAT-01
parent_feature_name: Client & Project Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 17
---

# Screen Spec: Project Detail (Open Project)

## Overview

**Name:** Project Detail (Open Project)
**ID:** FEAT-01.SPEC-005
**Type:** Screen
**Purpose:** Single view of one project holding its proposal, milestones, deliverables, invoices, and activity, plus rename, archive/reactivate, and mark-complete actions.
**Parent Feature:** FEAT-01 -- Client & Project Management

## Scope and Non-Goals

**In Scope:**
- Renaming the project
- Archiving the project, including the open-items confirmation (FEAT-01.SPEC-007)
- Reactivating an archived project, restoring it to whatever stage it would show had it never been archived (FEAT-01.SPEC-011)
- Marking the project complete, including firing the completion invoice trigger (FEAT-01.SPEC-006) when applicable
- Surfacing the project's proposal, milestone/payment-schedule, deliverable, invoice, and activity areas as entry points into their owning features
- Showing the project's current derived stage (FEAT-01.SPEC-011)

**Non-Goals:**
- Drafting or editing the proposal itself -- handled by Proposal Creation & Sending (FEAT-02), opened from this screen's proposal area
- Defining milestones or the payment schedule -- handled by Milestone & Payment Schedule Setup (FEAT-04), opened from this screen's milestones area
- Uploading deliverables -- handled by Deliverable Upload & Sharing (FEAT-06), opened from this screen's milestone area
- Viewing or issuing invoices -- handled by Invoice Generation & Sending (FEAT-09), opened from this screen's invoices area
- Marking a project cancelled -- owned exclusively by Refund & Cancelled Project Handling (FEAT-25); this screen does not offer a Cancel action
- In-product hard delete of a project -- intentional lifecycle exclusion (dependency map): project deletion is owned exclusively by FEAT-24 (account deletion); this screen offers Archive only, never Delete

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Nadia selects a project row | The project's identifier |
| FEAT-01.SPEC-004 (Client Detail) | Nadia selects a project row from the client's own project list | The project's identifier |
| FEAT-01.SPEC-002 (Create Project) | Successful save | The newly created project's identifier |
| FEAT-31 (Operator Support Access) | Dana selects a project during a read-only support session | Same project, read-only rendering |
| FEAT-28 (Global Search, v1) | Nadia selects a project search result | The project's identifier |
| FEAT-29 (In-App Notification Center, Later) | Nadia opens the related project from a feed item | The project's identifier |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | Rename, Archive, Reactivate (when Archived), Mark Complete, open proposal/milestones/invoices/activity areas | -- |
| Owen (Client Primary Contact) | No | No | Not shown in Owen's portal navigation; this is the freelancer's own management surface, distinct from his own project status view in the client portal (FEAT-05) |
| Priya (Client Reviewer Contact) | No | No | Not shown in Priya's portal navigation |
| Dana (Support Operator) | Full screen, read-only | View only -- Rename, Archive, Reactivate, and Mark Complete controls are not rendered; deliverable files are never downloadable from this view (ASMP-18) | Attempting to reach an edit action or a file download directly returns Dana to the read-only view with no change made |
| Unauthenticated | No | No | Redirected to the sign-in screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." -- an in-progress rename is preserved locally and restored after re-authentication succeeds |

## Layout and Content

**Header:** Project name (editable inline via a "Rename" action) with a stage badge (Draft, In Progress, Complete, Cancelled, Archived), using the same identity-block presentation as the roster (FEAT-01.SPEC-003) and Client Detail (FEAT-01.SPEC-004). Shows the owning client's name as a link back to Client Detail. An overflow menu holds Archive, Reactivate, and Mark Complete: Reactivate is shown only when the project's current stage is Archived (and hidden otherwise); Mark Complete is hidden once the project is already Complete, Cancelled, or Archived; Archive is hidden while the project is Archived.

**Body, in order:**
- **Proposal area:** Summary of the project's active proposal status (none / draft / sent / accepted), opening Proposal Creation & Sending (FEAT-02) or Proposal Acceptance detail.
- **Milestones area:** Summary of the milestone/payment-schedule status, opening Milestone & Payment Schedule Setup (FEAT-04); milestones with deliverables link into Deliverable Upload & Sharing (FEAT-06).
- **Invoices area:** Summary list of the project's invoices with their status, opening Invoice Generation & Sending (FEAT-09) for detail or an ad hoc invoice.
- **Activity area:** A link opening the project's trail in Immutable Activity & Audit Trail (FEAT-13).

**Footer:** None.

### Responsive Behavior

- **Compact size class:** Header and the four content areas (proposal, milestones, invoices, activity) stack in a single column, full width, in the order listed.
- **Medium size class and above:** Same single-column stacking, capped at a consistent platform-wide content width and horizontally centered; no structural change beyond width capping.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| "Rename" action | Tap, then edit and confirm | Updates project_name | Header reflects new name | Toast "Project renamed" |
| Client name link | Tap | Navigate to FEAT-01.SPEC-004 (Client Detail) | Screen transitions | Standard navigation transition |
| Overflow menu > Archive | Tap | Triggers FEAT-01.SPEC-007 (Archive Open-Items Check) | If no open items, project archives immediately; if open items exist, a confirmation dialog appears first | Toast "Project archived" on completion |
| Overflow menu > Reactivate (shown only when the project is Archived) | Tap | Clears the project's Archived status; stage recomputes via FEAT-01.SPEC-011 to whatever value its underlying proposal, milestone, invoice, completion, and cancellation state produce | Stage badge updates to the recomputed stage | Toast "Project reactivated" |
| Overflow menu > Mark Complete | Tap | Triggers FEAT-01.SPEC-006 (Completion Invoice Trigger); sets completed_at; recomputes stage via FEAT-01.SPEC-011 | Stage badge updates to "Complete" | Confirmation dialog states plainly what completing does: "This marks the project complete and, if the payment schedule includes one, issues the final invoice." Toast "Project marked complete" after confirming. |
| Proposal area | Tap | Navigate to FEAT-02 (proposal draft or detail) | Screen transitions | Standard navigation transition |
| Milestones area | Tap | Navigate to FEAT-04 (milestone and payment schedule editor) | Screen transitions | Standard navigation transition |
| Invoices area | Tap | Navigate to FEAT-09 (invoice detail or ad hoc invoice) | Screen transitions | Standard navigation transition |
| Activity area | Tap | Navigate to FEAT-13 (project activity trail) | Screen transitions | Standard navigation transition |

### Accessibility Notes

- **Focus order:** Rename action -> client name link -> overflow menu -> proposal area -> milestones area -> invoices area -> activity area.
- **Dynamic updates:** Toasts, the stage badge's change, and the Mark Complete confirmation dialog's text are announced to assistive technology; a concurrent-edit rejection dialog receives focus immediately when it appears.
- **Keyboard alternatives:** Every action on this screen is reachable by keyboard; there are no pointer-only gestures.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Skeleton layout for header and the four content areas | Screen opens | Data finishes loading |
| Loaded | Full detail as described in Layout and Content | Data load completes | A mutating action occurs, or navigation away |
| Marking Complete | Mark Complete confirmation dialog, then a brief in-progress indicator while FEAT-01.SPEC-006 evaluates and FEAT-01.SPEC-011 recomputes stage | Nadia confirms Mark Complete | Completion finishes or fails |
| Error | Error banner: "Couldn't load this project. Try again." with a Retry button | Initial data load fails | Retry succeeds |
| Offline/Degraded | Banner "You're offline -- changes will be saved when you reconnect." at top; the loaded detail remains viewable; Rename, Archive, Reactivate, and Mark Complete are disabled until connectivity returns | Connectivity lost while viewing an already-loaded project | Connectivity restored -- controls re-enable |

## Validation Rules

**Option B -- Inline (simple validation not warranting a standalone spec):**

| Field | Condition | When Checked | Error Message |
|-------|-----------|-------------|---------------|
| Project Name (rename) | Required, non-empty | On confirm | "Project name is required" |

Archive and Mark Complete eligibility are governed by FEAT-01.SPEC-007 and FEAT-01.SPEC-006/FEAT-01.SPEC-011 respectively, referenced above.

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-------------------------------------|
| Client name link tap | FEAT-01.SPEC-004 (Client Detail) | -- |
| Proposal area tap | Proposal draft / detail | FEAT-02 (Proposal Creation & Sending) |
| Milestones area tap | Milestone and payment schedule editor | FEAT-04 (Milestone & Payment Schedule Setup) |
| Invoices area tap | Invoice detail / ad hoc invoice | FEAT-09 (Invoice Generation & Sending) |
| Activity area tap | Project activity trail | FEAT-13 (Immutable Activity & Audit Trail) |
| Successful archive | FEAT-01.SPEC-003 (Client & Project Roster) | -- |
| Back navigation | The screen the user arrived from (roster or Client Detail) | -- |

## Data Model

**Creates:** None (mark-complete and archive are transitions on the existing record, not new records).
**Reads:** Project record -- project_name, client (owning client's name), stage (derived), completed_at, cancelled_at. Summary status from Proposal, Payment Schedule, and Invoice (read-only summaries; full detail lives in their owning features).
**Updates:** Project record -- project_name (Rename), status (Archive, Reactivate), completed_at (Mark Complete).
**Deletes:** None -- this screen offers no delete path for a project (see Non-Goals).

## Business Rules

- Renaming a project updates its display name everywhere it is shown; ID-based references (invoices, activity entries) are unaffected.
- Archive is subject to the open-items confirmation defined by FEAT-01.SPEC-007; archived projects are retained indefinitely with no automatic purge (ASMP-22, SC-24), and their Proposal, Milestones, Deliverables, Invoices, and Activity Log entries remain reachable.
- Reactivating an archived project clears its Archived status; stage recomputes immediately via FEAT-01.SPEC-011 to the value its underlying proposal, milestone, invoice, completion, and cancellation state produce independent of the Archived override -- the project returns to whatever stage it would show had it never been archived (Draft, In Progress, Complete, or Cancelled). Reactivation carries no active-count or plan-limit check: FEAT-01.SPEC-008's active-client limit governs Clients only, per that spec's own Non-Goals, and this product defines no equivalent cap on Projects.
- Marking a project complete fires FEAT-01.SPEC-006, which issues the final invoice only when the payment schedule includes an on-completion payment (XBR-03); otherwise completed_at is set and the stage recomputes to "Complete" with no invoice.
- Completed projects stay visible to the client (Owen, Priya) in their portal until archived.
- System-driven stage changes (from proposal acceptance or milestone approval, per FEAT-01.SPEC-011) never overwrite a freelancer's explicit Complete or Cancelled transition, per the dependency map's Contention note for Project.
- Cancellation (Cancelled stage) is set exclusively by Refund & Cancelled Project Handling (FEAT-25); this screen never sets it directly.

## Edge Cases

- **Project renamed by another session between load and save (last-write-wins per the dependency map's Contention note for Project)** -- The later save simply overwrites the earlier one; no conflict dialog for the rename.
- **Archive, Reactivate, or Mark Complete attempted after the project's state changed since load (reject-with-refresh per the dependency map's Contention note)** -- The action is rejected with "This project's state changed since you loaded this page. Refresh to see the latest state before continuing." and a "Refresh" action reloads the project; this is the concurrent-edit conflict behavior for state-changing actions on this shared entity.
- **Archive attempted with unpaid invoices or pending approvals** -- FEAT-01.SPEC-007 surfaces an explicit confirmation summarizing what is still open before the archive completes; declining returns to this screen unchanged.
- **Mark Complete attempted on a project already marked Cancelled by FEAT-25 since this screen loaded** -- Rejected with "This project was cancelled since you loaded this page. Refresh to see the latest state." per the same reject-with-refresh resolution; Mark Complete is not offered on an already-Cancelled project once refreshed.
- **Reactivate tapped on an Archived project that also has completed_at set (it was completed, then later archived, and is now reactivated)** -- Once the Archived override clears, FEAT-01.SPEC-011's next-highest precedence condition applies: since completed_at is still set, the stage recomputes to "Complete," not "In Progress" -- reactivation never fabricates an in-progress state for a project that was already Complete before it was archived.
- **Reactivate tapped on an Archived project that was never completed or cancelled (no milestone approved, no invoice generated, proposal not yet accepted)** -- Stage recomputes to "Draft," matching the state it would show had it never been archived.
- **A linked proposal, milestone, or invoice event changes the project's underlying state while this screen is open** -- The stage badge is a snapshot per load, not live-updating; it reflects the latest computed value (FEAT-01.SPEC-011) on the next load or refresh, consistent with the roster's (FEAT-01.SPEC-003) same snapshot behavior.
- **Nadia loses connectivity mid-rename** -- The offline banner appears; the in-progress rename is preserved locally and the confirm action remains disabled until connectivity returns, then submits automatically.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-003 (Client & Project Roster) | Navigation (inbound/outbound) | Entry point; destination after archive |
| FEAT-01.SPEC-004 (Client Detail) | Navigation (inbound/outbound) | Entry point; destination via client name link |
| FEAT-01.SPEC-002 (Create Project) | Navigation (inbound) | Entry point after a successful project creation |
| FEAT-01.SPEC-006 (Completion Invoice Trigger) | Triggers (outbound) | Mark Complete fires this automation |
| FEAT-01.SPEC-007 (Archive Open-Items Check) | Triggers (outbound) | Archive action runs the open-items check |
| FEAT-01.SPEC-011 (Project Stage Derivation) | References (inbound) | Supplies the stage badge and recomputes it after Mark Complete or Reactivate |
| FEAT-02 (Proposal Creation & Sending) | Navigation (outbound) | Proposal area |
| FEAT-04 (Milestone & Payment Schedule Setup) | Navigation (outbound) | Milestones area |
| FEAT-06 (Deliverable Upload & Sharing) | Navigation (outbound) | Deliverable upload from a milestone within the milestones area |
| FEAT-09 (Invoice Generation & Sending) | Navigation (outbound) | Invoices area |
| FEAT-13 (Immutable Activity & Audit Trail) | Navigation (outbound) | Activity area |
| FEAT-25 (Refund & Cancelled Project Handling) | References (inbound) | Owns the Cancelled transition this screen only displays |
| FEAT-28 (Global Search Across Clients & Projects) | Navigation (inbound) | Search result selection lands here |
| FEAT-29 (In-App Notification Center) | Navigation (inbound) | Feed item opens the related project (Later) |
| FEAT-31 (Operator Support Access) | Navigation (inbound) | Dana's read-only session entry point |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| project_opened | viewer role (Nadia / Dana), current stage | Screen loads successfully | N/A -- opening an existing project is outside the initial add-to-roster window "Client and Project Setup Speed" measures |
| project_marked_complete | had on-completion payment in schedule (yes/no) | Mark Complete confirmed and processed | N/A -- no success-metrics.md metric tracks project completion for this feature; completion's downstream invoice event is measured under Invoice Generation & Sending's own metrics |
| project_archived | had open items at archive time (yes/no) | Archive completes | N/A -- no success-metrics.md metric tracks archive activity for this feature |
| project_reactivated | recomputed stage after reactivation | Reactivate completes | N/A -- no success-metrics.md metric tracks reactivate activity for this feature |

## Acceptance Criteria

**FEAT-01.SPEC-005-AC-01:** Given Nadia is on Project Detail for "Website Redesign," when she renames it to "Website Redesign v2" and confirms, then the header updates and a "Project renamed" toast appears.

**FEAT-01.SPEC-005-AC-02:** Given Nadia is on Project Detail, when she taps the client name link, then she is taken to that client's Client Detail (FEAT-01.SPEC-004).

**FEAT-01.SPEC-005-AC-03:** Given Nadia taps Archive on a project with no unpaid invoices or pending approvals, then the project archives immediately with a "Project archived" toast and no confirmation dialog.

**FEAT-01.SPEC-005-AC-04:** Given Nadia taps Archive on a project with an unpaid invoice, then FEAT-01.SPEC-007's confirmation dialog appears summarizing the open item before the archive completes.

**FEAT-01.SPEC-005-AC-05:** Given Nadia taps Mark Complete on a project whose payment schedule includes an on-completion payment, when she confirms, then FEAT-01.SPEC-006 fires the final invoice, completed_at is set, and the stage badge updates to "Complete."

**FEAT-01.SPEC-005-AC-06:** Given Nadia taps Mark Complete on a project whose payment schedule has no on-completion payment, when she confirms, then completed_at is set and the stage badge updates to "Complete" with no invoice issued.

**FEAT-01.SPEC-005-AC-07:** Given the project was cancelled by FEAT-25 in another session since Nadia loaded this screen, when she attempts Mark Complete, then the action is rejected with "This project was cancelled since you loaded this page. Refresh to see the latest state."

**FEAT-01.SPEC-005-AC-08:** Given a client contact opens the project in their portal after it is marked complete, then the project remains visible to them until it is archived.

**FEAT-01.SPEC-005-AC-09:** Given Dana is viewing this project in a read-only support session, when she looks for Rename, Archive, Reactivate, or Mark Complete, then none of those controls are rendered.

**FEAT-01.SPEC-005-AC-10:** Given Nadia is on Project Detail, when she taps the invoices area, then she is taken to Invoice Generation & Sending (FEAT-09) for this project.

**FEAT-01.SPEC-005-AC-11:** Given Nadia is on Project Detail, when she taps the activity area, then she is taken to the project's trail in Immutable Activity & Audit Trail (FEAT-13).

**FEAT-01.SPEC-005-AC-12:** Given Nadia loses connectivity while viewing an already-loaded project, then the offline banner appears and Rename, Archive, Reactivate, and Mark Complete become disabled.

**FEAT-01.SPEC-005-AC-13:** Given Project Detail fails to load, then an error banner "Couldn't load this project. Try again." appears with a Retry button.

**FEAT-01.SPEC-005-AC-14:** Given Nadia taps Mark Complete, when the confirmation dialog appears, then it states plainly "This marks the project complete and, if the payment schedule includes one, issues the final invoice" before she confirms.

**FEAT-01.SPEC-005-AC-15:** Given Nadia taps Reactivate on an Archived project, when it completes, then the project's Archived status clears, the stage badge updates to its recomputed value, and a "Project reactivated" toast appears.

**FEAT-01.SPEC-005-AC-16:** Given Nadia taps Reactivate on an Archived project that has completed_at set, when it completes, then the stage badge shows "Complete," not "In Progress."

**FEAT-01.SPEC-005-AC-17:** Given Nadia opens the overflow menu on a project that is not Archived, when she looks for Reactivate, then it is not shown, since Reactivate only appears once a project's stage is Archived.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 9 | 9 |
| States | 5 | 5 |
| Business Rules | 7 | 7 |
| Edge Cases | 8 | 8 |
