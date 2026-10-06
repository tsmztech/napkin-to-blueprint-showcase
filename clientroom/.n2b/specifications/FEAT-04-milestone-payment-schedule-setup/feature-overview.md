---
document_type: feature-overview
feature_number: FEAT-04
feature_name: Milestone & Payment Schedule Setup
feature_slug: milestone-payment-schedule-setup
priority_tier: Core
feature_type: User-Facing
produced_by: feature-analyst
status: final
created: 2026-09-26
spec_count: 4
screen_count: 2
automation_count: 1
logic_rule_count: 1
integration_count: 0
notification_count: 0
---

# Feature Breakdown Brief: Milestone & Payment Schedule Setup

## Summary

**Feature:** Milestone & Payment Schedule Setup
**ID:** FEAT-04
**Description:** The freelancer defines the project's milestones and chooses how the client is billed — deposit, per-milestone, on completion, or a mix — so invoicing can be automatic later in the project.
**Priority:** Core
**Phase:** MVP
**Type:** User-Facing
**Rationale:** BRIEF.md, Target Users & Roles: the freelancer "sets milestones and payment schedules (deposit, per milestone, on completion, or a mix)." Without this, invoices have no trigger to fire from. MVP phase: required before any milestone-based invoice can exist. [RESEARCH-INFORMED: Dubsado reviewers report no milestone sequencing and run a separate task tool for multi-phase work (independent 2025–2026 review, MEDIUM); no profiled competitor delivers milestone-driven billing as a first-class flow, making this a differentiator rather than parity] [INFERRED: carried from Visionary draft]

**Key Capabilities:**
- Define milestones — name and order the project's discrete stages of work
- Set a payment structure — choose deposit, per-milestone, on completion, or a mix
- Adjust the schedule — change milestones or pricing mid-project as scope evolves
- Remove a milestone — delete a milestone that has not been approved or invoiced [AUDIT-ADDED: 3 -- entity coverage: Milestone had no removal path]

## Spec Inventory

| Spec ID | Name | Type | Roles Touched | Purpose (one line) |
|---------|------|------|---------------|--------------------|
| FEAT-04.SPEC-001 | Milestone & Payment Schedule Editor | Screen | Nadia | Nadia defines, orders, prices, adjusts, and removes milestones and sets the project's payment structure and triggers |
| FEAT-04.SPEC-002 | Milestone Timeline (Client View) | Screen | Owen, Priya, Dana | Read-only display of the project's milestone timeline within each viewer's own project view |
| FEAT-04.SPEC-003 | Milestone & Schedule Validation and Edit Rules | Logic/Rule | Nadia | Governs required fields, invoicing eligibility, approved/invoiced immutability, dated non-retroactive schedule edits, and the price-mismatch flag |
| FEAT-04.SPEC-004 | Milestone Reorder Recalculation | Automation | Nadia | Renumbers remaining milestones' order whenever one is added, removed, or manually reordered |

## Capability Coverage Map

| Key Capability | Covered By | How | Discovery Phase |
|----------------|-----------|-----|-----------------|
| Define milestones | FEAT-04.SPEC-001 | Editor lets Nadia add a milestone with a name, order, and a price or "no separate charge" flag | Phase 2 (Explicit) |
| Set a payment structure | FEAT-04.SPEC-001 | Editor lets Nadia choose deposit, per-milestone, on completion, or a mix and set the corresponding triggers/amounts | Phase 2 (Explicit) |
| Adjust the schedule | FEAT-04.SPEC-001, governed by FEAT-04.SPEC-003 | Editor allows mid-project edits to milestones and pricing; SPEC-003 enforces the dated, non-retroactive rule | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |
| Remove a milestone | FEAT-04.SPEC-001, governed by FEAT-04.SPEC-003 | Editor exposes a remove action; SPEC-003 blocks removal once approved or invoiced | Phase 2 (Explicit) / Phase 5 (Rule Discovery) |

**Analyst-Discovered Specs** -- Specs not directly tied to a Key Capability, surfaced by Phases 3-6:

| Spec | Purpose | Discovery Phase | Discovery Rationale |
|------|---------|-----------------|---------------------|
| FEAT-04.SPEC-002 | Milestone Timeline (Client View) | Phase 3 (Entity-Lifecycle Analysis) | The CRUD matrix's Read (list) cell for Owen and Priya was empty: the Access field states they "see the resulting milestone list read-only within their project view," and Data Notes states the project's milestone timeline is displayed — this read surface has no home in the Nadia-only editor and needed its own screen |
| FEAT-04.SPEC-003 | Milestone & Schedule Validation and Edit Rules | Phase 5 (Rule-Constraint Discovery) | The Validation & Limits field carries conditional logic (required fields, minimum-one-trigger, approved/invoiced immutability, dated non-retroactive edits, price-mismatch flag) that is shared across every write path in SPEC-001 and crosses the 5+/conditional-logic threshold for a standalone spec |
| FEAT-04.SPEC-004 | Milestone Reorder Recalculation | Phase 4 (Trigger-Response Analysis) | Adding, removing, or manually reordering a milestone changes the `order` field of every other milestone in the project — a cascading update across multiple Milestone records, not a single direct write |

## Entity-Lifecycle Coverage Matrix

**Entity: Milestone**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-04.SPEC-001 | Editor's "add milestone" action creates a record with name, order, and price/no-charge flag; initial status is Defined | Governed by FEAT-04.SPEC-003's required-field rule |
| Read (single) | FEAT-04.SPEC-001 | Editor loads one milestone into its edit row/sub-form | Also read by FEAT-05, FEAT-06, FEAT-07, FEAT-08, FEAT-09, FEAT-13, FEAT-31 outside this feature |
| Read (list) | FEAT-04.SPEC-001, FEAT-04.SPEC-002 | Editor lists all of a project's milestones for Nadia; the Client View lists the same milestones read-only for Owen, Priya, and Dana | -- |
| Update | FEAT-04.SPEC-001 | Editor lets Nadia rename, re-price, re-order, or set/change the target date | Re-ordering cascades through FEAT-04.SPEC-004; eligibility gated by FEAT-04.SPEC-003 |
| Delete/Archive | FEAT-04.SPEC-001 | Hard delete: the remove action permanently deletes a milestone record; no restore path, because a removed milestone was never approved or invoiced and so carries no evidentiary record to preserve (FEAT-04.SPEC-003 blocks removal once one exists). Cascades to that milestone's Deliverables and milestone-level Comments, whose removal is executed under FEAT-06's and the comment thread's own lifecycle rules, not re-implemented here. No retention/purge policy applies beyond the immediate hard delete | Coordination point with FEAT-06 noted in Shared Context |
| State Transition | FEAT-04.SPEC-001 | Sets the initial "Defined" status at creation | Later transitions (Deliverable Uploaded, Approved, Reopened) are owned by FEAT-06 and FEAT-08 respectively — out of this feature's scope, per the dependency map's Milestone lifecycle line |

**Entity: Payment Schedule**

| Operation | Covered By | How | Notes |
|-----------|-----------|-----|-------|
| Create | FEAT-04.SPEC-001 | Editor's payment-structure step creates the project's single Payment Schedule record (structure, and deposit_amount/completion_amount as applicable) | One schedule per project |
| Read (single) | FEAT-04.SPEC-001 | Editor loads the project's current schedule for display and editing | Also read by FEAT-01, FEAT-02, FEAT-03, FEAT-08, FEAT-09 outside this feature |
| Read (list) | N/A | A project has at most one Payment Schedule (dependency map: "One per Project"), so no list view applies | -- |
| Update | FEAT-04.SPEC-001, governed by FEAT-04.SPEC-003 | Editor lets Nadia change the structure, amounts, or per-milestone triggers; SPEC-003 enforces that the change is dated and applies only to later triggers, never retroactively | change_history entry appended on each edit |
| Delete/Archive | N/A | The Payment Schedule has no standalone deletion path within this feature — the dependency map lists it as "Deleted by FEAT-24" only (full account deletion). This is an intentional design decision, not an omission: the schedule is foundational to a project's invoicing for the project's whole life, so no in-product delete or archive of the schedule itself exists (see Non-Goals) | -- |
| State Transition | N/A | The dependency map's Fields list for Payment Schedule carries no status field — only structure and amounts vary, through Update | -- |

**Referenced Entities (read-only):**

| Entity | Read By | Context |
|--------|---------|---------|
| Proposal | FEAT-04.SPEC-003 | Price-mismatch check compares the sum of milestone prices against the accepted proposal's total price |
| Project | FEAT-04.SPEC-001, FEAT-04.SPEC-002 | Both screens are scoped to one Project; the editor is reached from the project's milestones area |

## Side-Effect Inventory

| Trigger | Response | Disposition | Spec |
|---------|----------|-------------|------|
| Nadia adds, removes, or manually reorders a milestone | Renumber the `order` field of every remaining milestone in the project so the sequence stays contiguous | Standalone Automation | FEAT-04.SPEC-004 |
| Nadia saves a milestone (create or edit) | Validate name is present and either price or "no separate charge" is set | Standalone Logic/Rule | FEAT-04.SPEC-003 |
| Nadia saves the schedule or any milestone | Verify at least one payment trigger exists somewhere in the project's schedule (required before the project can invoice at all) | Standalone Logic/Rule | FEAT-04.SPEC-003 |
| Nadia attempts to remove or re-price a milestone that is Approved or already invoiced | Refuse the change; direct her to reopening (FEAT-08) or a correction (FEAT-09) instead | Standalone Logic/Rule | FEAT-04.SPEC-003 |
| Nadia saves a mid-project schedule or milestone change | Record the change as dated in change_history; the change applies only to triggers that fire afterward, never retroactively | Standalone Logic/Rule (write itself stays inline in FEAT-04.SPEC-001) | FEAT-04.SPEC-003 |
| Nadia saves milestones whose price sum does not match the accepted proposal's total | Flag the mismatch to Nadia; never block the save, since scope can legitimately change | Standalone Logic/Rule | FEAT-04.SPEC-003 |
| Nadia saves a milestone or schedule successfully | Show success confirmation; keep her in the editor | Inline in triggering screen | FEAT-04.SPEC-001 |
| A save fails (e.g., connectivity lost) | Preserve the entered milestone data on-screen; do not discard the draft | Inline in triggering screen | FEAT-04.SPEC-001 |
| Milestones/schedule are created or changed | Owen, Priya, and Dana's timeline view reflects the current state on next read | Cross-spec (read reflects latest write) | FEAT-04.SPEC-002 |
| Proposal is accepted with a deposit trigger in the schedule | Deposit invoice generates automatically | Cross-feature — owned by FEAT-09, triggered by FEAT-03 | FEAT-09 responsibility |
| A milestone is approved | Next invoice in the schedule generates automatically | Cross-feature — owned by FEAT-09, triggered by FEAT-08 | FEAT-09 responsibility |

## Shared Context

**Shared Entities:**
- Milestone -- created and updated by SPEC-001, governed by SPEC-003's validation and eligibility rules, renumbered by SPEC-004, displayed read-only by SPEC-002. Fields: name, order, price or no_separate_charge flag, payment_trigger, target_date, status, approved_at, approved_by.
- Payment Schedule -- created and updated by SPEC-001, governed by SPEC-003's dated non-retroactive rule, displayed (indirectly, through its milestones' triggers) by SPEC-002. Fields: structure, deposit_amount, completion_amount, change_history.

**Shared UI Patterns:**
- Milestone row -- the same milestone data (name, order, price/no-charge, trigger, target date, status) is rendered editably in SPEC-001 and read-only in SPEC-002. Spec Writers for both screens should keep the row's information and ordering identical, differing only in whether controls are interactive.
- Target-date display -- both screens show target dates converted to each viewer's own time zone (FEAT-15); Spec Writers should describe this consistently rather than re-deriving the conversion behavior per screen.

**Shared Validation:**
- SPEC-003 defines every validation, eligibility, and dated-edit rule. SPEC-001 references SPEC-003 for all save-time checks rather than duplicating them; SPEC-004 checks with SPEC-003 only insofar as reordering never runs on a removal SPEC-003 has refused.

**Flagged discrepancy (not resolved by this Brief):** The dependency map's Activity Log Entry lifecycle line lists the features that create audit-log entries and does not include FEAT-04, even though this feature's Signals field names `milestone_created`, `milestone_schedule_edited`, `payment_trigger_set`, and `milestone_removed`. Per this Brief's instructions, Signals are treated as analytics-linkage events for this feature's specs (SPEC-001, SPEC-004), not as a claim that FEAT-04 writes to the Activity Log — evidentiary audit entries for this entity's life only begin at approval (FEAT-08) and invoicing (FEAT-09). This inconsistency between the Signals field and the Activity Log Entry lifecycle line is surfaced here for the Requirements Architect, not resolved by the Feature Analyst.

## Internal Dependency Map

```
SPEC-001 (Milestone & Payment Schedule Editor) -> [Nadia saves a milestone or schedule change] -> SPEC-003 (Milestone & Schedule Validation and Edit Rules) -> [pass/refuse] -> SPEC-001
SPEC-001 (Milestone & Payment Schedule Editor) -> [milestone added, removed, or reordered] -> SPEC-004 (Milestone Reorder Recalculation) -> [renumbered order] -> SPEC-001
SPEC-001 (Milestone & Payment Schedule Editor) -> [milestone/schedule state changes] -> SPEC-002 (Milestone Timeline, Client View)
```

**Default Entry:** SPEC-001 (Milestone & Payment Schedule Editor) for Nadia, who has Full access; SPEC-002 (Milestone Timeline, Client View) for Owen, Priya, and Dana, whose access is View or Own-only — the two roles land on different screens because editing and viewing are separate specs.

## Cross-Feature Touchpoints

| This Spec | Direction | Other Feature | Context | Trigger |
|-----------|-----------|---------------|---------|---------|
| FEAT-04.SPEC-001 | Inbound | FEAT-01 (Client & Project Management) | Nadia navigates from a project's milestones area into the schedule editor | Nadia opens the milestones area of a project |
| FEAT-04.SPEC-001 | Outbound | FEAT-03 (Proposal Acceptance) | The accepted proposal reads this feature's Payment Schedule to fire the deposit trigger, using the schedule as it stood at acceptance (XBR-01) | Owen accepts the proposal |
| FEAT-04.SPEC-001 | Inbound | FEAT-06 (Deliverable Upload & Management) | A milestone's status updates to Deliverable Uploaded | Nadia uploads a deliverable round to a milestone |
| FEAT-04.SPEC-001 | Inbound | FEAT-08 (Milestone Approval) | A milestone's status updates to Approved (locking edit/remove, per SPEC-003 and XBR-10) or Reopened (unlocking it) | Owen approves the milestone; Nadia reopens it |
| FEAT-04.SPEC-001 | Outbound | FEAT-08 (Milestone Approval) | Milestone approval reads this feature's payment_trigger to know whether approval should issue an invoice (XBR-02) | Owen approves the milestone |
| FEAT-04.SPEC-001 | Outbound | FEAT-09 (Invoice Generation & Sending) | The Payment Schedule and each milestone's trigger drive automatic invoice creation on deposit, approval, or completion (XBR-01, XBR-02, XBR-03) | A deposit, approval, or completion trigger fires |
| FEAT-04.SPEC-002 | Inbound | FEAT-31 (Operator Support Access) | Dana's read-only, logged support session displays this timeline | Dana opens a support session on the freelancer's account |
| FEAT-04.SPEC-001 | Outbound | FEAT-24 (Account Deletion) | Milestone and Payment Schedule records are deleted as part of account deletion | Nadia's account is deleted |

## Non-Functional Notes

**Data volumes / growth:** Each project carries a small, human-scale set of milestones (a handful per project is typical of the freelancer/small-project scale ASMP-22 describes — a few thousand freelancers with 3–15 active clients each); the editor and timeline are single-project, single-schedule views and do not need to scale beyond that.

**Responsiveness:** Editing is local (per the feature's States field): milestone and schedule changes appear immediately as Nadia edits, with the save/sync to the persisted record happening transparently and reconciling once connectivity returns (ASMP-27's offline posture). The Client View (SPEC-002) is a standard fetched screen and should show real loading progress rather than an indefinite blank state.

**Data sensitivity / privacy:** Milestone and Payment Schedule data is business-confidential pricing information (dependency map, Data Sensitivity lines); once a milestone is approved, its approved_at/approved_by fields become evidentiary and immutable (ASMP-25), which is exactly why SPEC-003 blocks re-pricing or removal at that point. Neither entity carries special-category personal data.

**Compliance flags:** N/A — neither Milestone nor Payment Schedule carries personal data or falls under a named compliance regime (dependency map, Data Sensitivity lines); GDPR-class handling applies elsewhere in the product to the approver's identity (Milestone) and to Invoice records, not to the schedule-setup data itself.

## Non-Goals

- **Recurring or retainer billing on a fixed calendar** -- Excluded per scope-boundaries.md (SC-14): the payment structure this feature offers is limited to deposit, per-milestone, on completion, or a mix; an occasional retainer charge is issued as an ad-hoc invoice by FEAT-09 instead of as a calendar-driven schedule here.
- **Partial payments or instalments on a single invoice** -- Excluded per scope-boundaries.md (SC-17): instalment-style billing is deliberately expressed through separate milestone and deposit entries in this feature's schedule, each producing its own invoice, rather than as a partial-payment feature on one invoice.
- **A configurable workflow or rule builder for payment logic** -- Excluded per scope-boundaries.md (SC-11): Nadia chooses among four fixed structures (deposit, per-milestone, on completion, mix) rather than authoring conditional payment logic; a general workflow/automation builder is the exact setup burden the product is positioned against.
- **Standalone deletion or archival of the Payment Schedule itself** -- Intentional lifecycle decision surfaced by the CRUD matrix: the dependency map lists the schedule as deleted only through FEAT-24 (full account deletion), never in-product, because a project's schedule is foundational to its invoicing for the whole life of the project; individual milestones can be removed (until approved/invoiced), but the schedule record persists.
- **Automatic correction of a milestone/proposal price mismatch** -- Adjacency analysis: the Primary Flows & Alternates field states this case is "flagged, not blocked, since scope can legitimately change"; the system never auto-adjusts milestone prices or the proposal total to reconcile a mismatch, leaving that judgment to Nadia.
