---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-01.SPEC-011
spec_name: Project Stage Derivation
spec_slug: project-stage-derivation
parent_feature: FEAT-01
parent_feature_name: Client & Project Management
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 8
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Project Stage Derivation

## Overview

**Name:** Project Stage Derivation
**ID:** FEAT-01.SPEC-011
**Type:** Logic/Rule
**Purpose:** Computes the roster/detail "stage" label (Draft, In Progress, Complete, Cancelled, Archived) from the project's proposal, milestone, invoice, completion, and cancellation state.
**Parent Feature:** FEAT-01 -- Client & Project Management
**Governed Entity:** Project (specifically the derived stage field)

## Scope and Non-Goals

**In Scope:**
- The formula that computes a project's stage label from its underlying proposal, milestone, invoice, completed_at, and cancelled_at state
- When the derivation re-runs (on every relevant upstream event)
- The exact precedence when multiple conditions could apply at once

**Non-Goals:**
- Setting or clearing completed_at, cancelled_at, or the project's Archived status themselves -- completed_at is set by FEAT-01.SPEC-006 (via Mark Complete on FEAT-01.SPEC-005); cancelled_at is set exclusively by FEAT-25; the Archived status is set by Archive and cleared by Reactivate, both on FEAT-01.SPEC-005; this spec only reads all three and recomputes stage in response
- Defining Proposal, Milestone, or Invoice status values themselves -- owned by FEAT-02/FEAT-03, FEAT-04/FEAT-08, and FEAT-09 respectively; this spec only reads their status to compute the stage label
- Presenting the stage badge visually -- owned by the screens that display it (FEAT-01.SPEC-003, FEAT-01.SPEC-004, FEAT-01.SPEC-005), which reference this spec's output rather than computing it themselves

## Governed Entity

**Entity:** Project
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| project_name | text | Project's display name |
| client | reference | The owning Client (exactly one) |
| stage | derived (enum: Draft, In Progress, Complete, Cancelled, Archived) | Computed by this spec |
| currency | text (configured via FEAT-15) | Not evaluated by this spec |
| tax_label / tax_rate | text / number (configured via FEAT-15) | Not evaluated by this spec |
| completed_at | timestamp | Read by this spec to derive "Complete" |
| cancelled_at | timestamp | Read by this spec to derive "Cancelled" |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-01.SPEC-002 | Create Project | On project creation, sets the initial stage to "Draft" |
| FEAT-01.SPEC-003 | Client & Project Roster | Reads the derived stage to render each project's badge |
| FEAT-01.SPEC-005 | Project Detail | Reads the derived stage to render the badge; re-derives after Mark Complete, Archive, or Reactivate (Reactivate clears the Archived override so the remaining precedence conditions resolve the restored stage) |
| FEAT-01.SPEC-006 | Completion Invoice Trigger | Signals re-derivation after setting completed_at |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| stage | No direct input validation -- stage is never set by direct user input; it is always computed by this spec's derivation formula | Always | On every derivation run | N/A -- there is no user-facing input to validate; an attempt to set stage directly does not exist in this product's design | No |
| project_name, client, currency, tax_label, tax_rate | No validation beyond data type in this spec -- governed elsewhere (FEAT-01.SPEC-002 for project_name/client, FEAT-15 for currency/tax) | Always | -- | -- | -- |
| completed_at | No validation beyond data type in this spec -- write authority belongs to FEAT-01.SPEC-006; this spec only reads it | Always | -- | -- | -- |
| cancelled_at | No validation beyond data type in this spec -- write authority belongs to FEAT-25; this spec only reads it | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Stage derivation formula | status (Archived flag), cancelled_at, completed_at, Proposal.status, Milestone.status (any), Invoice.status (any) | Evaluated in this precedence, highest first: (1) if the project's own status is Archived, stage = "Archived"; (2) else if cancelled_at is set, stage = "Cancelled"; (3) else if completed_at is set, stage = "Complete"; (4) else if any Milestone has been Approved, or any Invoice has been generated for this project, stage = "In Progress"; (5) else if the Proposal has been Accepted, stage = "In Progress"; (6) else stage = "Draft" | N/A -- this is a computed display value, not a user-input field, so it carries no validation error message |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| View the derived stage on this feature's own screens (roster, Client Detail, Project Detail) | Nadia (Freelancer) | Always | -- |
| View the derived stage on this feature's own screens | Dana (Support Operator) | Always, inside a logged support session (FEAT-31), read-only | -- |
| View the derived stage on this feature's own screens | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never -- the Client & Project Management area is not part of either contact's portal (per the Access Matrix, this capability group is None for both) | Not shown; the same computed stage value they see for their own company's projects is displayed to them separately, through Client Portal Access (FEAT-05), which reads this spec's output but is not governed by this spec |
| Set the stage directly (bypassing derivation) | No role | Never -- the product defines no direct-set capability for this field; stage is always computed | N/A -- no control for this exists anywhere in the product |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|---------------------|
| stage | See the Stage derivation formula in Cross-Field Rules above | On project creation, and re-computed on every relevant upstream event (proposal accepted, milestone approved, invoice generated, project marked complete, project cancelled, project archived) | No -- always derived, never directly editable by any role |

## Business Rules

- Data Notes (feature-overview.md): the roster's "stage" label is derived from state owned by other features -- this spec is the single formula every screen depends on, so no screen computes it independently.
- System-driven stage changes (from proposal acceptance or milestone approval) never overwrite a freelancer's explicit Complete or Cancelled transition, per the dependency map's Contention note for Project -- this is why Archived, Cancelled, and Complete take precedence over the "In Progress" conditions in the formula's evaluation order.
- The Cancelled transition is set exclusively by FEAT-25 (dependency map: Project "Updated by ... FEAT-25 (mark cancelled)"); this spec only reads cancelled_at and never sets it.
- The Complete transition is set exclusively by FEAT-01.SPEC-006 via FEAT-01.SPEC-005's Mark Complete action; this spec only reads completed_at and never sets it.
- A project's stage recomputes immediately whenever any input to the formula changes (proposal acceptance, milestone approval, invoice generation, completion, cancellation, archiving, or reactivation), so the roster and detail screens never display a stale label across those transitions once refreshed.
- Reactivating an Archived project (FEAT-01.SPEC-005) clears the Archived override rather than setting a stage directly; the formula then resolves the restored stage from whatever cancelled_at, completed_at, milestone, invoice, and proposal state the project already carries, at the same precedence used for every other derivation -- so a reactivated project always lands on the stage it would show had it never been archived (Cancelled and Complete still outrank the milestone/invoice-driven "In Progress" condition, exactly as they do outside of archiving).

## Edge Cases

- **A project is Archived and also has completed_at set (it was completed, then later archived)** -- "Archived" takes precedence in the formula; the project's stage displays as "Archived," not "Complete," while it is archived. If Nadia later reactivates it from FEAT-01.SPEC-005, the Archived override clears and the formula's next-highest condition applies: since completed_at is still set, the stage resolves to "Complete," not "In Progress" -- reactivation restores the project to the stage it held before archiving, it never re-derives from scratch as if completion had not happened.
- **A project is Cancelled by FEAT-25 after already having an Approved milestone and generated invoices** -- "Cancelled" takes precedence over the milestone/invoice-driven "In Progress" condition; the project's full history of milestones and invoices is preserved and remains reachable, but the stage label reflects the cancellation.
- **A project has an Accepted proposal but no milestones approved and no invoices yet** -- Stage = "In Progress" (condition 5), since acceptance itself moves the project past "Draft" even before any milestone work begins.
- **A project has no proposal at all yet** -- Stage = "Draft" (condition 6, the fallback), matching its state immediately after creation (FEAT-01.SPEC-002).
- **Two upstream events fire in quick succession (e.g., a milestone is approved and, moments later, the project is marked complete)** -- Each derivation run reads the full current state independently; the later run (after completed_at is set) resolves to "Complete" regardless of the milestone approval that fired just before it, since completed_at outranks the milestone/invoice condition in precedence.
- **A voided-and-resent proposal (FEAT-02) leaves a Draft-status proposal alongside an earlier Voided one** -- Only the current, non-voided proposal's status is read by the formula; a Voided proposal is never treated as "Accepted" for derivation purposes, so re-derivation after a void-and-resend correctly falls back to whatever the current proposal's real status is.

## Acceptance Criteria

**FEAT-01.SPEC-011-AC-01:** Given a project has just been created with no proposal yet, when its stage is derived, then it resolves to "Draft."

**FEAT-01.SPEC-011-AC-02:** Given a project's proposal has been accepted and no milestone is yet approved, when its stage is derived, then it resolves to "In Progress."

**FEAT-01.SPEC-011-AC-03:** Given a project has at least one Approved milestone, when its stage is derived, then it resolves to "In Progress" (if not already Complete, Cancelled, or Archived).

**FEAT-01.SPEC-011-AC-04:** Given Nadia marks a project complete via FEAT-01.SPEC-005/SPEC-006, when its stage is next derived, then it resolves to "Complete," taking precedence over any milestone-based "In Progress" condition.

**FEAT-01.SPEC-011-AC-05:** Given FEAT-25 marks a project cancelled, when its stage is next derived, then it resolves to "Cancelled," taking precedence over its milestone and invoice state.

**FEAT-01.SPEC-011-AC-06:** Given a project is Archived, when its stage is derived, then it resolves to "Archived" regardless of whether completed_at or cancelled_at is also set.

**FEAT-01.SPEC-011-AC-07:** Given a project's proposal is voided and re-sent (still unaccepted), when its stage is re-derived, then it does not resolve to "In Progress" on the basis of the voided proposal, since a Voided proposal is never treated as Accepted.

**FEAT-01.SPEC-011-AC-08:** Given a milestone is approved and, moments later, the same project is marked complete, when its stage is derived after both events, then it resolves to "Complete."

**FEAT-01.SPEC-011-AC-09:** Given Owen views his company's project in the client portal, when he checks its stage, then he sees the same derived value Nadia sees on the roster, since derivation is a single, shared formula.

**FEAT-01.SPEC-011-AC-10:** Given Dana views a project in a read-only support session, when she checks its stage, then she sees the current derived value with no ability to set it directly.

**FEAT-01.SPEC-011-AC-11:** Given no role or screen in this product offers a direct control to set stage, when any user looks for one, then none exists anywhere in the product.

**FEAT-01.SPEC-011-AC-12:** Given an Archived project that also has completed_at set is reactivated via FEAT-01.SPEC-005, when its stage is next derived, then it resolves to "Complete" (not "In Progress"), since completed_at still outranks the milestone/invoice condition in precedence once the Archived override is cleared.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 1 | 1 |
| Business Rules | 6 | 6 |
| Edge Cases | 6 | 6 |
