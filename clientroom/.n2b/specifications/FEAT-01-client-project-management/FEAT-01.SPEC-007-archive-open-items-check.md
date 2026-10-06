---
document_type: spec
spec_type: automation
spec_id: FEAT-01.SPEC-007
spec_name: Archive Open-Items Check
spec_slug: archive-open-items-check
parent_feature: FEAT-01
parent_feature_name: Client & Project Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Archive Open-Items Check

## Overview

**Name:** Archive Open-Items Check
**ID:** FEAT-01.SPEC-007
**Type:** Automation
**Purpose:** Checks a client or project for unpaid invoices or pending approvals at the moment of archiving and requires explicit confirmation if any exist.
**Parent Feature:** FEAT-01 -- Client & Project Management

## Scope and Non-Goals

**In Scope:**
- Checking, at the moment Archive is invoked on a Client (FEAT-01.SPEC-004) or a Project (FEAT-01.SPEC-005), whether any unpaid invoices or pending milestone approvals exist for the target and, for a client, across all its projects
- Presenting the exact open items found so Nadia can make an informed choice
- Proceeding with the archive immediately when nothing is open, with no confirmation step

**Non-Goals:**
- Deciding client delete eligibility -- a distinct, stricter check owned by FEAT-01.SPEC-009 (no sent proposal, invoice, or activity at all, versus this check's narrower "nothing currently unpaid or pending")
- Resolving the open items themselves (e.g., collecting the unpaid invoice) -- this automation only surfaces them; resolution happens in Invoice Payment Processing (FEAT-10) or Milestone Approval (FEAT-08)
- Reversing an archive once confirmed -- reactivation (FEAT-01.SPEC-004, FEAT-01.SPEC-008) is a separate action with its own limit check, not a rollback of this automation
- Checking projects for their own client-level archive -- when a client is archived, this automation checks the client and every one of its projects together in one pass, rather than requiring a separate check per project; archiving a single project (leaving the client Active) checks only that project

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia taps Archive on a client | FEAT-01.SPEC-004 (Client Detail) | Always, before the client's status changes | Client reference, all of its Projects' Invoice and Payment Schedule (pending approval) state |
| Nadia taps Archive on a project | FEAT-01.SPEC-005 (Project Detail) | Always, before the project's status changes | Project reference, its own Invoice and pending milestone-approval state |

## Processing Logic

1. Receive the archive target (a Client, or a single Project) from the triggering screen.
2. If the target is a Client, gather every Invoice and every Milestone across all of that client's Projects; if the target is a single Project, gather only that project's own Invoices and Milestones.
3. Evaluate each gathered Invoice's status: flag any invoice not in a Paid, Refunded, Partially refunded, or Corrected state as an open item ("unpaid invoice").
4. Evaluate each gathered Milestone's status: flag any milestone in a Deliverable Uploaded state (awaiting the client's approval) as an open item ("pending approval").
5. If no open items are found, signal the triggering screen to proceed with the archive immediately.
6. If one or more open items are found, return the list (grouped as unpaid invoices and pending approvals, each named by project and identifier) to the triggering screen for the explicit confirmation dialog.
7. On confirmation, signal the triggering screen to proceed with the archive; on decline, signal it to take no action.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| No open items | Zero unpaid invoices and zero pending approvals found | None from this automation; the triggering screen proceeds to set status to Archived | Archive completes immediately with a "Client archived" / "Project archived" toast | FEAT-01.SPEC-004, FEAT-01.SPEC-005 |
| Open items found, confirmed | 1+ open items found; Nadia confirms | None from this automation; the triggering screen proceeds to set status to Archived | The shared "here is what's still open" confirmation dialog, then the standard archive toast on confirming | FEAT-01.SPEC-004, FEAT-01.SPEC-005 |
| Open items found, declined | 1+ open items found; Nadia declines | None | Dialog closes; the client or project remains Active, unchanged | FEAT-01.SPEC-004, FEAT-01.SPEC-005 |
| Automation failure | The check itself cannot complete (e.g., invoice or milestone state cannot be read) | None | Blocking error on the triggering screen: "Couldn't check this item for open invoices or approvals. Try again." -- Archive does not proceed | FEAT-01.SPEC-004, FEAT-01.SPEC-005 |

## Data Model

**Reads:** Invoice -- status, project, across the archive target's scope. Milestone -- status, project, across the archive target's scope.
**Creates:** None.
**Updates:** None -- this automation only informs the archive decision; the status change to Archived is applied by the triggering screen (FEAT-01.SPEC-004 or FEAT-01.SPEC-005), not by this automation.
**Deletes:** None.

## Business Rules

- Archiving never erases records; this check exists precisely to make sure Nadia is not surprised by silently archiving something with financial or approval consequences still open (XBR-24).
- The check runs synchronously as part of the Archive action -- the triggering screen waits for its result before showing either the immediate-archive toast or the confirmation dialog.
- "Unpaid" for this check means any invoice status other than Paid, Refunded, Partially refunded, or Corrected -- an Overdue or Payment pending invoice is still an open item.
- "Pending approval" for this check means a milestone in Deliverable Uploaded status -- an Approved or Reopened milestone is not an open item for this purpose.

## Edge Cases

- **Client has multiple projects, only one with an unpaid invoice** -- The confirmation names the specific project and invoice, not a generic "this client has open items" message.
- **All invoices are Paid but one milestone is awaiting approval** -- The check still surfaces the pending approval as an open item; "no open items" requires both invoice and approval checks to be clear.
- **An invoice's status changes (e.g., gets paid) between the check running and Nadia confirming the dialog** -- The confirmation is based on the state read when the check ran; if Nadia confirms, the archive proceeds against the archive target's current state at commit, which is re-checked by the triggering screen's own reject-with-refresh behavior for state changes (FEAT-01.SPEC-004, FEAT-01.SPEC-005) -- a state change during the brief confirmation window does not block a since-resolved item from still being reported, but does not block the archive either, since being Paid can only reduce open items, never require blocking further.
- **Concurrent trigger firing (Archive tapped on the same client from two open sessions of Nadia's at effectively the same time)** -- Each session's check runs independently; whichever archive completes first wins, and the second is rejected with the triggering screen's reject-with-refresh behavior for a client/project whose state changed since load (FEAT-01.SPEC-004, FEAT-01.SPEC-005, per the dependency map's Contention notes).
- **Trigger fires while a previous run is in flight** -- Archive is disabled on the triggering screen while a check for the same target is in progress, so a second run for the same client or project cannot start before the first resolves; checks for different targets proceed independently.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-01.SPEC-004 (Client Detail) | Triggered by (inbound) | Archive action on a client fires this check |
| FEAT-01.SPEC-005 (Project Detail) | Triggered by (inbound) | Archive action on a project fires this check |
| FEAT-01.SPEC-004 (Client Detail) | Affects (outbound) | Returns the open-items result and confirmation outcome to the client archive flow |
| FEAT-01.SPEC-005 (Project Detail) | Affects (outbound) | Returns the open-items result and confirmation outcome to the project archive flow |
| FEAT-09 (Invoice Generation & Sending) | References (inbound) | Source of Invoice status read by this check |
| FEAT-13 (Immutable Activity & Audit Trail) | References (inbound) | The confirmed archive, once applied, is itself a record-worthy event logged by FEAT-13 |

## Analytics and Success Signals

- **archive_open_items_found** (target type: client/project, unpaid invoice count, pending approval count) -- N/A -- no success-metrics.md metric in this feature's slice tracks archive open-items frequency; recorded for product-usage visibility only.
- **archive_confirmed_with_open_items** (target type: client/project) -- N/A -- same reason as above.
- **archive_declined_with_open_items** (target type: client/project) -- N/A -- same reason as above.

## Acceptance Criteria

**FEAT-01.SPEC-007-AC-01:** Given Nadia taps Archive on a client with no unpaid invoices and no pending approvals across any of its projects, when the check runs, then the archive proceeds immediately with no confirmation dialog.

**FEAT-01.SPEC-007-AC-02:** Given Nadia taps Archive on a project with one unpaid invoice, when the check runs, then a confirmation dialog names that specific invoice before the archive can proceed.

**FEAT-01.SPEC-007-AC-03:** Given Nadia taps Archive on a client whose only open item is a milestone awaiting approval on one of its projects (all invoices Paid), when the check runs, then the confirmation dialog surfaces the pending approval as the open item.

**FEAT-01.SPEC-007-AC-04:** Given Nadia sees the open-items confirmation dialog and taps Confirm, then the archive proceeds and the standard archive toast appears.

**FEAT-01.SPEC-007-AC-05:** Given Nadia sees the open-items confirmation dialog and taps Decline, then the dialog closes and the client or project remains Active, unchanged.

**FEAT-01.SPEC-007-AC-06:** Given the open-items check itself fails to complete, when Nadia taps Archive, then she sees "Couldn't check this item for open invoices or approvals. Try again." and the archive does not proceed.

**FEAT-01.SPEC-007-AC-07:** Given a project has an Overdue invoice, when the check runs, then the invoice is treated as an open item, since "unpaid" includes Overdue and Payment pending statuses.

**FEAT-01.SPEC-007-AC-08:** Given Nadia has two sessions open on the same client and taps Archive in both at effectively the same time, when the first archive completes, then the second is rejected with a refresh prompt rather than running a redundant archive.

**FEAT-01.SPEC-007-AC-09:** Given a milestone on a project is Approved (not Deliverable Uploaded) and all of that project's invoices are Paid, when Nadia taps Archive on that project, then the check finds no open items and the archive proceeds immediately.

**FEAT-01.SPEC-007-AC-10:** Given Nadia taps Archive on a project (not a client) with no unpaid invoices and no pending approvals, when the check runs, then the archive proceeds immediately with no confirmation dialog, the same as for a client target.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
