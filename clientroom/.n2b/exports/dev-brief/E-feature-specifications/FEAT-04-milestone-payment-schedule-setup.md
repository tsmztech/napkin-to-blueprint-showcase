# FEAT-04 — Milestone & Payment Schedule Setup

This chapter covers Milestone & Payment Schedule Setup, a Core-tier feature. It contains the feature breakdown brief followed by every specification in full: 4 specifications carrying 86 acceptance criteria.

| Spec | Name | Type | Acceptance criteria |
|------|------|------|---------------------|
| FEAT-04.SPEC-001 | Milestone & Payment Schedule Editor | screen | 23 |
| FEAT-04.SPEC-002 | Milestone Timeline (Client View) | screen | 17 |
| FEAT-04.SPEC-003 | Milestone & Schedule Validation and Edit Rules | logic-rule | 34 |
| FEAT-04.SPEC-004 | Milestone Reorder Recalculation | automation | 12 |

The feature breakdown brief follows, then every specification in full.


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



# Screen Spec: Milestone & Payment Schedule Editor

## Overview

**Name:** Milestone & Payment Schedule Editor
**ID:** FEAT-04.SPEC-001
**Type:** Screen
**Purpose:** Nadia defines, orders, prices, adjusts, and removes a project's milestones and sets the project's payment structure and triggers.
**Parent Feature:** FEAT-04 -- Milestone & Payment Schedule Setup

## Scope and Non-Goals

**In Scope:**
- Creating, renaming, pricing, target-dating, reordering, and removing milestones for one project
- Creating and editing the project's single Payment Schedule (structure: deposit, per-milestone, on completion, or a mix; deposit_amount; completion_amount)
- Setting each milestone's payment trigger (whether its approval issues an invoice)
- Displaying the price-mismatch flag against the accepted proposal's total, and a change-history view of dated schedule adjustments
- Preserving in-progress edits through a failed save or a connectivity loss

**Non-Goals:**
- Read-only display of the milestone timeline to client contacts -- owned by FEAT-04.SPEC-002 (Milestone Timeline, Client View); the Brief's Internal Dependency Map routes Nadia and client contacts to distinct specs since editing and viewing are separate concerns.
- Field-level validation, eligibility, and dated-edit rule logic -- governed by FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules); this screen calls that spec's rules on save rather than duplicating them.
- Milestone status transitions beyond the initial "Defined" state -- Deliverable Uploaded is set by FEAT-06 (Deliverable Upload & Sharing) and Approved/Reopened by FEAT-08 (Milestone Approval), per the dependency map's Milestone lifecycle line.
- Recurring or retainer billing on a fixed calendar -- excluded per scope-boundaries.md (SC-14): the payment structure this feature offers is limited to deposit, per-milestone, on completion, or a mix.
- Standalone deletion or archival of the Payment Schedule record itself -- excluded per the Brief's Non-Goals: the dependency map lists the schedule as deleted only through FEAT-24 (full account deletion), never in-product, because the schedule is foundational to the project's invoicing for its whole life.

## Entry Points

| Source | Trigger | Context Carried |
|--------|---------|-----------------|
| FEAT-01.SPEC-005 (Project Detail / Open Project) | Nadia opens the milestones area of a project | Project reference; the project's existing milestones (if any) and Payment Schedule (if already created) load into the editor |

## Access and Visibility

| Role/State | Can View | Can Act | Unauthorized Experience |
|------------|----------|---------|-------------------------|
| Nadia (Freelancer) | Full screen | All actions: create, rename, price, target-date, reorder, and remove milestones; create and edit the Payment Schedule; save | -- |
| Owen (Client Primary Contact) | No | No | This editor is not part of his portal navigation and no direct link exists to it; if he somehow reaches the URL, he is redirected to his own scoped project view, which shows the read-only Milestone Timeline (FEAT-04.SPEC-002) instead |
| Priya (Client Reviewer Contact) | No | No | Same as Owen: redirected to the read-only Milestone Timeline (FEAT-04.SPEC-002) in her own scoped project view |
| Dana (Support Operator) | Full screen, read-only, inside a logged support session (FEAT-31) | No | Every input, selector, drag handle, Add Milestone, Remove, and Save control is disabled with the tooltip "Support access is read-only." No attempted edit reaches the server |
| Unauthenticated | No | No | Redirected to Nadia's sign-in; after signing in, the user lands on the Client & Project Roster (FEAT-01.SPEC-003), not this screen |
| Expired session | No | No | Dialog: "Your session has expired. Sign in to continue." Any in-progress local milestone or schedule edits are preserved and restored after re-authentication succeeds |

## Layout and Content

**Header:** Breadcrumb back arrow to the Project Detail screen (FEAT-01.SPEC-005), screen title "Milestones & Payment Schedule," and the project's name as a subtitle.

**Body:** Two stacked sections.

*Payment Structure section (top):* A four-option structure selector -- Deposit, Per-Milestone, On Completion, Mix. Selecting Deposit or Mix reveals a Deposit Amount field (currency-formatted number input, in the project's configured currency). Selecting On Completion or Mix reveals a Completion Amount field (same treatment). Below the selector, a "View change history" link expands a dated list of prior schedule adjustments (date, what changed, from/to values), sourced from the Payment Schedule's change_history.

*Milestone List section (below):* An ordered list of milestone rows. Each row shows, in order: a reorder control (drag handle plus explicit Move Up / Move Down buttons), the milestone Name (text input), a Price input paired with a "No separate charge" toggle (exactly one of the two is active per milestone), a Payment Trigger selector ("Issues an invoice on approval" / "No trigger"), a Target Date picker (optional), a read-only Status badge (Defined, Deliverable Uploaded, Approved, or Reopened), and a Remove control. The Remove control is visible only on milestones whose status is Defined or Reopened, per FEAT-04.SPEC-003's approved/invoiced immutability rule. An "Add Milestone" button sits below the last row.

A **price-mismatch banner** appears directly above the Milestone List section whenever the sum of milestone prices does not match the accepted proposal's total price (per FEAT-04.SPEC-003). It states the two figures and that this is informational only.

**Footer:** A "Save" button that persists all schedule and milestone edits together, and a "Done" link that returns to Project Detail (FEAT-01.SPEC-005).

### Responsive Behavior

- **Compact breakpoint:** The two sections stack vertically, full width. Each milestone row collapses to two lines: name and status badge on the first line, price/no-charge toggle, payment trigger, and target date on the second. Reordering uses the Move Up / Move Down buttons only; the drag handle is not shown.
- **Medium size class and above:** Each milestone row renders as a single horizontal row with all fields visible side by side. The drag handle appears alongside the Move Up / Move Down buttons. The Payment Structure selector renders as an inline horizontal group instead of a stacked list.
- **Change-history panel:** Renders as an inline expanding list at all sizes; no structural change beyond width.

## Interactions

| Element | Trigger | Action | State Change | Feedback |
|---------|---------|--------|-------------|----------|
| Back arrow | Tap | Navigate to FEAT-01.SPEC-005 (Project Detail) | Screen closes; unsaved local edits prompt confirmation first | If unsaved changes exist: dialog "You have unsaved changes. Discard?" |
| Payment Structure selector | Select an option | Sets the schedule's structure locally; reveals/hides Deposit Amount and Completion Amount fields | Selected option highlighted; dependent fields appear/disappear | Immediate visual update, no save yet |
| Deposit Amount input | Type | Captures the deposit amount locally | Field shows entered value | Standard input focus state; validated via FEAT-04.SPEC-003 on blur |
| Completion Amount input | Type | Captures the completion amount locally | Field shows entered value | Standard input focus state; validated via FEAT-04.SPEC-003 on blur |
| "View change history" link | Tap | Expands the dated change_history list | Panel expands below the link | List of prior dated adjustments shown |
| Add Milestone button | Tap | Adds a new, empty milestone row at the end of the list with status Defined | New row appears; triggers FEAT-04.SPEC-004 (Milestone Reorder Recalculation) to confirm its order position | New row focused on its Name field |
| Milestone Name input | Type / Blur | Captures the milestone name; validated via FEAT-04.SPEC-003 on blur | Field shows entered text; error state if blank on blur | "Milestone name is required" if empty on blur |
| Price input / "No separate charge" toggle | Type or toggle | Sets the milestone's price, or marks it as carrying no separate charge (mutually exclusive); validated via FEAT-04.SPEC-003 | Whichever control is active shows the current value; the other clears | Standard input state; cross-field error via FEAT-04.SPEC-003 if both or neither are set on submit |
| Payment Trigger selector | Select | Sets whether this milestone's approval issues an invoice | Selector shows chosen option | Immediate visual update |
| Target Date picker | Select a date | Sets the milestone's optional target date, shown in each viewer's own time zone (FEAT-15.SPEC-006) | Field shows chosen date | Standard picker feedback |
| Reorder (drag handle or Move Up/Down) | Drag or tap | Moves the milestone to a new position; triggers FEAT-04.SPEC-004 (Milestone Reorder Recalculation) | List re-renders in the new order | Brief in-progress state while recalculation completes, then the renumbered list |
| Remove control (Defined/Reopened milestones only) | Tap | Validates removal eligibility via FEAT-04.SPEC-003; if allowed, deletes the milestone and triggers FEAT-04.SPEC-004 | Row is removed from the list; remaining rows renumber | Confirmation dialog before removal: "Remove this milestone? This can't be undone." |
| Save button | Tap | 1. Validates all fields via FEAT-04.SPEC-003. 2. If valid, persists the Payment Schedule and every changed Milestone. 3. Appends a dated change_history entry for any schedule change. | Button shows loading state during save | Success: toast "Schedule saved" and the editor stays open. Failure: inline field errors or a save-error banner; entered data is preserved |
| Save button (while saving) | Tap | No action -- debounced to prevent a duplicate submission | None | Button remains in loading state |
| Done link | Tap | Navigate to FEAT-01.SPEC-005 (Project Detail) | Screen closes; unsaved changes prompt confirmation first | Same unsaved-changes dialog as the back arrow |

### Accessibility Notes

- **Focus order:** Back arrow -> Payment Structure selector -> Deposit Amount (if shown) -> Completion Amount (if shown) -> "View change history" link -> each milestone row in list order (reorder control -> Name -> Price/No-charge toggle -> Payment Trigger -> Target Date -> Remove) -> Add Milestone -> Save -> Done.
- **Reordering:** Every milestone can be reordered by keyboard using the always-visible Move Up / Move Down buttons; the drag handle is a pointer-only convenience, never the sole way to reorder.
- **Validation announcements:** When a field enters an error state (via FEAT-04.SPEC-003), its error message is announced to assistive technology and programmatically associated with the field. The price-mismatch banner is announced when it first appears.
- **Save feedback:** The "Schedule saved" toast is announced on success; on validation failure, focus moves to the first field in error.

## States

| State Name | Appearance | Entry Condition | Exit Condition |
|------------|-----------|-----------------|----------------|
| Loading | Both sections show a lightweight in-progress indicator while the project's existing milestones and Payment Schedule (if any) are fetched | Screen is opened from FEAT-01.SPEC-005 | Existing data finishes loading, revealing either the populated editor or the Empty state |
| Empty (no milestones) | Milestone List section shows "No milestones yet" with the Add Milestone button prominent; Payment Structure section is fully usable | Project has an accepted proposal but no milestones defined yet | Nadia adds the first milestone |
| Filling (local editing) | Fields reflect entered values immediately; Save button enabled | Nadia types in or selects any field | Nadia taps Save or navigates away |
| Validating | Save button shows a loading spinner | Nadia taps Save | FEAT-04.SPEC-003 validation completes (pass or fail) |
| Validation Error | Failing fields show inline error messages from FEAT-04.SPEC-003; price-mismatch banner shown if applicable (non-blocking) | Blocking validation fails | Nadia corrects the field(s) and re-triggers Save |
| Saving | Save button shows a loading spinner; fields remain editable | Validation passes | Save completes or fails |
| Save Error | Error banner: "Could not save. Check your connection and try again." with a Retry button | Save operation fails (not a validation failure) | Nadia taps Retry or the save succeeds on retry; all entered data remains on screen |
| Offline/Degraded | Banner: "You're offline -- your changes will sync once you reconnect." Fields remain fully editable and changes appear immediately in the list; Save queues the update locally | Connectivity is lost while the screen is open | Connectivity restored -- the queued save submits automatically and the standard success or validation feedback appears |

## Validation Rules

Validation governed by FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules). See that spec for all field-level, cross-field, and authorization rules. This screen applies validation on field blur (name, price/no-charge, deposit/completion amounts, target date) and on Save (the full cross-field and eligibility set, including the price-mismatch and payment-trigger checks).

## Navigation Out

| Trigger | Destination Spec | Destination Feature (if different) |
|---------|-----------------|-----------------------------------|
| Back arrow tap | FEAT-01.SPEC-005 (Project Detail / Open Project) | FEAT-01 |
| Done link tap | FEAT-01.SPEC-005 (Project Detail / Open Project) | FEAT-01 |
| Cancel on unsaved-changes dialog | Stays on this screen | -- |
| Confirm discard on unsaved-changes dialog | FEAT-01.SPEC-005 (Project Detail / Open Project) | FEAT-01 |

## Data Model

**Creates:** Milestone record -- name, order (assigned via FEAT-04.SPEC-004), price or no_separate_charge flag, payment_trigger, target_date; initial status set to Defined. Payment Schedule record -- structure, and deposit_amount/completion_amount as applicable; created on the project's first schedule save (one per project).

**Reads:** Milestone -- all fields, for every milestone belonging to the current project. Payment Schedule -- current structure, deposit_amount, completion_amount, and change_history. Proposal (read-only, owned by FEAT-03) -- the accepted proposal's total price, for the price-mismatch check.

**Updates:** Milestone -- name, price/no_separate_charge flag, payment_trigger, target_date, order (while status is Defined or Reopened; blocked once Approved or invoiced, per FEAT-04.SPEC-003). Payment Schedule -- structure, deposit_amount, completion_amount; each successful edit appends a dated entry to change_history.

**Deletes:** Milestone -- hard delete, only while status is Defined or Reopened (never once Approved or invoiced, per FEAT-04.SPEC-003); cascades to that milestone's Deliverables and milestone-level Comments, whose removal runs under FEAT-06's and the comment thread's own lifecycle rules.

## Business Rules

- All save-time field validation, eligibility (approved/invoiced immutability), the dated non-retroactive schedule-edit rule, and price-mismatch flagging are governed by FEAT-04.SPEC-003 -- this screen calls those rules rather than duplicating them.
- XBR-10: A milestone that has been approved or invoiced cannot be removed or re-priced; changes go through a logged reopen (FEAT-08) or an invoice correction (FEAT-09).
- Adding, removing, or manually reordering a milestone triggers FEAT-04.SPEC-004 (Milestone Reorder Recalculation), which renumbers the remaining milestones so the sequence stays contiguous.
- At least one payment trigger must exist somewhere in the project's schedule before the project can invoice at all (FEAT-04.SPEC-003); this is surfaced here as a non-blocking indicator, not a save-blocking rule.
- A saved schedule or milestone-pricing change is dated and applies only to payment triggers that fire afterward -- a trigger that has already fired uses the schedule as it stood at that moment (dependency map, Payment Schedule Contention note).

## Edge Cases

- **User navigates away with unsaved changes** -- Confirmation dialog: "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.
- **User taps Save twice rapidly** -- Second tap is ignored while the first save is in progress (button in loading state).
- **Save fails (e.g., connectivity lost)** -- Entered milestone and schedule data is preserved on-screen; the draft is never discarded (per the Brief's Side-Effect Inventory).
- **Price mismatch on save** -- Save proceeds; the mismatch banner is shown, never blocking, since scope can legitimately change.
- **Removing the milestone that currently carries the project's only payment trigger** -- Removal itself proceeds normally (it is not an approved/invoiced milestone); the "no payment trigger yet" indicator then appears, since invoicing is now blocked until a trigger exists again (enforced at invoice time by FEAT-09).
- **Payment Schedule changed by another of Nadia's own sessions between load and save** -- Save is rejected with dialog "This schedule was updated in another session. Review the latest version before saving." with "View Latest" (reloads the record; local edits discarded after confirmation) and "Keep Editing" options. Resolution: reject-with-refresh, per the dependency map's Contention note for Payment Schedule.
- **A milestone Owen approves while Nadia has it open for editing** -- Nadia's next save on that milestone is rejected with "This milestone was approved by Owen while you were editing. Reopen it to make further changes." (see FEAT-08.SPEC-002, Milestone Reopen Screen). The row switches to read-only and its Status badge updates to Approved. Resolution: reject-with-refresh, per the dependency map's Contention note for Milestone.
- **Removing a milestone that was invoiced by another process a moment earlier** -- Removal is refused with "This milestone has already been invoiced and can't be removed," and the row's Status badge and controls refresh to reflect its current state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules) | References (outbound) | All field validation, eligibility, dated-edit, and price-mismatch rules applied on save |
| FEAT-04.SPEC-004 (Milestone Reorder Recalculation) | Triggers (outbound) | Fires whenever a milestone is added, removed, or manually reordered |
| FEAT-04.SPEC-002 (Milestone Timeline, Client View) | References (outbound) | Client-facing counterpart shows the same milestone row data read-only; edits made here appear there on the client's next read |
| FEAT-01.SPEC-005 (Project Detail / Open Project) | Navigation (inbound) | Entry point via the project's milestones area |
| FEAT-03.SPEC-003 (Acceptance Recording) | References (outbound) | Reads this schedule at proposal acceptance to fire the deposit trigger (XBR-01), using the schedule as it stood at that moment |
| FEAT-06.SPEC-001 (Deliverable Upload) | References (inbound) | Updates a milestone's status to Deliverable Uploaded, reflected in this screen's Status badge |
| FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard) | References (inbound) | Updates a milestone's status to Approved, locking edit and remove per XBR-10 |
| FEAT-08.SPEC-005 (Reopen Recording) | References (inbound) | Updates a milestone's status to Reopened, unlocking edit and remove |
| FEAT-09.SPEC-004 (Automatic Invoice Generation) | References (outbound) | Reads each milestone's payment_trigger and the schedule to drive automatic invoicing (XBR-02) |
| FEAT-15.SPEC-006 (Time Zone & Local Date/Time Display Rule) | References (outbound) | Governs how the Target Date field is rendered in each viewer's own time zone |

## Analytics and Success Signals

| Event | Properties | Emitted When | Supports Metric |
|-------|-----------|--------------|-----------------|
| milestone_created | has_price (yes/no), payment_trigger_set (yes/no) | A new milestone is successfully saved | supports success-metrics.md: "Milestone Schedule Completeness" |
| milestone_schedule_edited | change_type (milestone_edit / schedule_edit), structure | A milestone or the Payment Schedule is successfully edited after initial creation | supports success-metrics.md: "Milestone Schedule Completeness" |
| payment_trigger_set | milestone reference, trigger_type (deposit / per_milestone / on_completion) | A milestone's payment trigger is set or changed and saved | supports success-metrics.md: "Milestone Schedule Completeness" |
| milestone_removed | remaining_trigger_count | A milestone is successfully removed | supports success-metrics.md: "Milestone Schedule Completeness" |
| schedule_price_mismatch_flagged | mismatch_amount | The price-mismatch banner is shown on a successful save | N/A -- no success-metrics.md metric measures mismatch-flag frequency; this event is retained for product telemetry so the signal is not silently dropped |

## Acceptance Criteria

**FEAT-04.SPEC-001-AC-01:** Given Nadia opens the Milestone & Payment Schedule Editor from Project Detail, when the screen is fetching the project's existing milestones and schedule, then both sections show a lightweight in-progress indicator rather than an indefinite blank screen.

**FEAT-04.SPEC-001-AC-02:** Given Nadia is on the Milestone & Payment Schedule Editor for a project with no milestones, when the screen loads, then the Milestone List section shows "No milestones yet" with the Add Milestone button prominent.

**FEAT-04.SPEC-001-AC-03:** Given Nadia selects "Mix" in the Payment Structure selector, when the selection completes, then both the Deposit Amount and Completion Amount fields appear.

**FEAT-04.SPEC-001-AC-04:** Given Nadia taps Add Milestone, when the new row appears, then it has an empty Name field focused, status Defined, and FEAT-04.SPEC-004 confirms its order position at the end of the list.

**FEAT-04.SPEC-001-AC-05:** Given Nadia types a milestone name and blurs the field, when the name is non-empty, then no error is shown; when it is empty, then the error "Milestone name is required" appears below the field.

**FEAT-04.SPEC-001-AC-06:** Given Nadia enters a price for a milestone, when she also has "No separate charge" toggled on, then setting the price clears the toggle so only one is active.

**FEAT-04.SPEC-001-AC-07:** Given Nadia sets a milestone's Payment Trigger to "Issues an invoice on approval," when she saves, then the milestone's payment_trigger is persisted and available to FEAT-08 and FEAT-09.

**FEAT-04.SPEC-001-AC-08:** Given Nadia sets a target date on a milestone, when the client Owen later views the same milestone in FEAT-04.SPEC-002, then the date displays converted to Owen's own time zone per FEAT-15.SPEC-006.

**FEAT-04.SPEC-001-AC-09:** Given Nadia drags a milestone to a new position, when the move completes, then FEAT-04.SPEC-004 renumbers the list and it re-renders in the new contiguous order.

**FEAT-04.SPEC-001-AC-10:** Given Nadia taps Remove on a milestone with status Defined, when she confirms the dialog, then the milestone is deleted and FEAT-04.SPEC-004 renumbers the remaining milestones.

**FEAT-04.SPEC-001-AC-11:** Given Nadia is viewing a milestone with status Approved, when she looks for a Remove control, then none is shown, per FEAT-04.SPEC-003's approved/invoiced immutability rule.

**FEAT-04.SPEC-001-AC-12:** Given Nadia has filled valid values across the schedule and milestones, when she taps Save, then the Payment Schedule and all changed Milestones persist, and the toast "Schedule saved" appears with the editor remaining open.

**FEAT-04.SPEC-001-AC-13:** Given Nadia taps Save while a save is already in progress, when she taps again, then the second tap is ignored and the button remains in its loading state.

**FEAT-04.SPEC-001-AC-14:** Given Nadia's milestone prices sum to less than the accepted proposal's total, when she saves, then the save completes and the price-mismatch banner appears showing both figures, without blocking the save.

**FEAT-04.SPEC-001-AC-15:** Given Nadia loses connectivity while editing, when she taps Save, then the banner "You're offline -- your changes will sync once you reconnect" appears and the change is submitted automatically once connectivity returns.

**FEAT-04.SPEC-001-AC-16:** Given Nadia's save fails due to a connection error (not a validation failure), when the failure occurs, then an error banner with a Retry option appears and all entered data remains on screen.

**FEAT-04.SPEC-001-AC-17:** Given Nadia has unsaved changes on this screen, when she taps the back arrow, then a dialog asks "You have unsaved changes. Discard?" with "Discard" and "Keep Editing" options.

**FEAT-04.SPEC-001-AC-18:** Given Nadia has this project's schedule open in a second browser session and saves a change there first, when she then taps Save in this session, then the save is rejected with "This schedule was updated in another session. Review the latest version before saving," offering "View Latest" and "Keep Editing."

**FEAT-04.SPEC-001-AC-19:** Given Owen approves a milestone while Nadia has that milestone's row open for editing, when Nadia attempts to save her edit, then it is rejected with "This milestone was approved by Owen while you were editing. Reopen it to make further changes," and the row becomes read-only showing the Approved badge.

**FEAT-04.SPEC-001-AC-20:** Given Owen (Client Primary Contact) attempts to reach this editor's URL directly, when the page loads, then he is redirected to his own scoped project view showing the read-only Milestone Timeline (FEAT-04.SPEC-002).

**FEAT-04.SPEC-001-AC-21:** Given Dana (Support Operator) is viewing this screen inside a logged support session, when she attempts to type into any field or tap Save, then the control is disabled and shows "Support access is read-only," and no change reaches the server.

**FEAT-04.SPEC-001-AC-22:** Given an unauthenticated visitor loads this screen's URL, when the page attempts to render, then they are redirected to Nadia's sign-in and, after signing in, land on the Client & Project Roster (FEAT-01.SPEC-005) rather than this editor.

**FEAT-04.SPEC-001-AC-23:** Given Nadia's session expires while she has unsaved edits on this screen, when the expiry is detected, then the dialog "Your session has expired. Sign in to continue." appears, and her in-progress edits are restored after she re-authenticates.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Interactions | 15 | 15 |
| States | 8 (loading, empty, filling, validating, validation error, saving, save error, offline/degraded) | 8 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |



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



# Logic/Rule Spec: Milestone & Schedule Validation and Edit Rules

## Overview

**Name:** Milestone & Schedule Validation and Edit Rules
**ID:** FEAT-04.SPEC-003
**Type:** Logic/Rule
**Purpose:** Governs required fields, invoicing eligibility, approved/invoiced immutability, dated non-retroactive schedule edits, and the price-mismatch flag for the Milestone and Payment Schedule entities.
**Parent Feature:** FEAT-04 -- Milestone & Payment Schedule Setup
**Governed Entity:** Milestone and Payment Schedule

## Scope and Non-Goals

**In Scope:**
- Per-field validation rules for Milestone and Payment Schedule
- Cross-field rules: price-or-no-charge requirement, at-least-one-payment-trigger check, price-mismatch flag
- Authorization rules for every action on Milestone and Payment Schedule, per role
- Default values and derived fields (status defaults, change_history entries)
- The dated, non-retroactive schedule-edit rule (XBR-10)

**Non-Goals:**
- Milestone reorder recalculation logic (renumbering the `order` field after an add, remove, or manual move) -- handled by FEAT-04.SPEC-004 (Milestone Reorder Recalculation); this spec only governs whether a removal is eligible, not how remaining milestones are renumbered.
- UI layout, field placement, and display behavior for validation errors -- defined in FEAT-04.SPEC-001 and FEAT-04.SPEC-002 (they reference this spec for the rules but own the display).
- Milestone status transitions beyond the initial Defined state -- Deliverable Uploaded is set by FEAT-06 and Approved/Reopened by FEAT-08; this spec only reads those statuses to gate its own eligibility rules, per the dependency map's Milestone lifecycle line.
- Automatic correction of a milestone/proposal price mismatch -- excluded per the Brief's Non-Goals: the system never auto-adjusts milestone prices or the proposal total to reconcile a mismatch; the mismatch is flagged for Nadia's judgment only.
- Recurring or retainer billing on a fixed calendar -- excluded per scope-boundaries.md (SC-14): the schedule structure is limited to deposit, per-milestone, on completion, or a mix.

## Governed Entity

**Entity:** Milestone
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | The milestone's name |
| order | number | Its position within the project's sequence |
| price | number | The milestone's price, when it carries a separate charge |
| no_separate_charge | boolean | Flag marking the milestone as carrying no separate charge (mutually exclusive with price) |
| payment_trigger | enum | Whether this milestone's approval issues an invoice, per the Payment Schedule |
| target_date | date | Optional date shown in each viewer's own time zone |
| status | enum | Defined, Deliverable Uploaded, Approved, Reopened |
| approved_at | date/time | Timestamp written once on approval, never altered |
| approved_by | text | Identity of the approving contact, written once, never altered |

**Entity:** Payment Schedule
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| structure | enum | deposit, per-milestone, on completion, or a mix |
| deposit_amount | number | Set when structure includes a deposit |
| completion_amount | number | Set when structure includes an on-completion payment |
| change_history | derived | Dated record of mid-project adjustments |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-04.SPEC-001 | Milestone & Payment Schedule Editor | On field blur (name, price/no-charge, deposit/completion amounts, target date) and on Save (all cross-field, eligibility, and price-mismatch rules); authorization on screen entry (which controls render) and on every save/remove attempt |
| FEAT-04.SPEC-004 | Milestone Reorder Recalculation | Reads this spec's removal-eligibility outcome before running -- it only renumbers a removal this spec has already allowed |
| FEAT-08.SPEC-003 | Approval Recording & Concurrency Guard | Reads this spec's Milestone edit-lock rule to refuse an approval recorded against a milestone Nadia has concurrently altered |
| FEAT-09.SPEC-004 | Automatic Invoice Generation | Reads this spec's at-least-one-payment-trigger outcome as a precondition for invoicing |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| Milestone.name | Required, non-empty, max 200 characters | Always | On blur, on submit | "Milestone name is required." / "Milestone name must be 200 characters or fewer." | Yes |
| Milestone.order | No validation beyond data type -- order is derived and maintained by FEAT-04.SPEC-004 | Always | -- | -- | -- |
| Milestone.price | Positive amount greater than zero, in the project's configured currency | When price is the active field (no_separate_charge is not set) | On blur | "Enter a price greater than zero, or mark this milestone as no separate charge." | Yes |
| Milestone.no_separate_charge | No validation beyond data type -- a boolean toggle mutually exclusive with price (see Cross-Field Rules) | Always | -- | -- | -- |
| Milestone.payment_trigger | No validation beyond data type -- constrained to the schedule's defined trigger options via a selection control | Always | -- | -- | -- |
| Milestone.target_date | Must be a valid date when provided | When provided (optional field) | On blur | "Enter a valid date." | Yes |
| Milestone.status | No validation beyond data type -- set only by FEAT-04.SPEC-001 (initial Defined), FEAT-06 (Deliverable Uploaded), and FEAT-08 (Approved, Reopened); no other writer may set it | Always | -- | -- | -- |
| Milestone.approved_at | No validation beyond data type -- written once by FEAT-08.SPEC-003 and never altered by this feature | Always | -- | -- | -- |
| Milestone.approved_by | No validation beyond data type -- written once by FEAT-08.SPEC-003 and never altered by this feature | Always | -- | -- | -- |
| Payment Schedule.structure | Required, one of {deposit, per-milestone, on completion, mix} | Always | On submit | "Choose a payment structure." | Yes |
| Payment Schedule.deposit_amount | Required, positive amount greater than zero | Structure is deposit or mix | On blur | "Enter a deposit amount greater than zero." | Yes |
| Payment Schedule.completion_amount | Required, positive amount greater than zero | Structure is on completion or mix | On blur | "Enter a completion amount greater than zero." | Yes |
| Payment Schedule.change_history | No validation beyond data type -- appended automatically by the dated-edit rule on every schedule save; not directly editable | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Price or no-charge, not both, not neither | Milestone.price, Milestone.no_separate_charge | Exactly one of the two must be set on submit | "Enter a price, or mark this milestone as no separate charge -- not both." |
| At least one payment trigger exists (non-blocking) | Payment Schedule.structure, Payment Schedule.deposit_amount, Payment Schedule.completion_amount, every Milestone.payment_trigger in the project | At least one of: the schedule includes a deposit, includes a completion amount, or at least one milestone is marked to trigger on approval. If none exist, the project cannot invoice at all (checked and enforced at invoice time by FEAT-09.SPEC-004) | Non-blocking indicator on save: "This project has no payment trigger yet -- no invoice can be generated until you add one." |
| Price-mismatch flag (non-blocking) | Sum of all of the project's Milestone.price values, accepted Proposal.price | If the sum does not equal the proposal's total, flag the difference; never block the save, since scope can legitimately change | Non-blocking banner: "Milestone prices ({sum}) don't match the accepted proposal total ({proposal total}). This is a heads-up only." |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create milestone | Nadia (Freelancer) | Always | -- |
| Create milestone | Owen (Client Primary Contact), Priya (Client Reviewer Contact) | Never | The editor screen (FEAT-04.SPEC-001) is not reachable from their portal navigation; a direct link redirects them to the read-only Milestone Timeline (FEAT-04.SPEC-002) |
| Create milestone | Dana (Support Operator) | Never | Add Milestone control disabled with "Support access is read-only." |
| View milestone | Nadia (Freelancer) | Always | -- |
| View milestone | Owen, Priya | Own-only -- their own client company's projects | Milestones outside their own company are never reachable (XBR-09); an out-of-scope link shows a plain explanation and a fresh-link option |
| View milestone | Dana | Always, inside a logged, read-only support session (FEAT-31) | -- |
| Edit milestone (rename, re-price, re-order, target date) | Nadia | Only while status is Defined or Reopened -- never once Approved or invoiced (XBR-10) | On an Approved/invoiced milestone, edit controls are disabled with "This milestone has been approved and can't be edited. Reopen it to make changes." |
| Edit milestone | Owen, Priya | Never | Edit controls are not shown -- FEAT-04.SPEC-002 is a view-only screen |
| Edit milestone | Dana | Never | All controls disabled with "Support access is read-only." |
| Remove milestone | Nadia | Only while status is Defined or Reopened -- never once Approved or invoiced (XBR-10) | On an Approved/invoiced milestone, the Remove control is hidden; a direct removal attempt shows "This milestone has been approved or invoiced and can't be removed." |
| Remove milestone | Owen, Priya, Dana | Never | Control is not shown to any of these roles |
| Create/Update Payment Schedule | Nadia | Always | -- |
| Create/Update Payment Schedule | Owen, Priya | Never | Not shown -- no schedule-editing surface exists in their portal |
| Create/Update Payment Schedule | Dana | Never | Disabled with "Support access is read-only." |
| View Payment Schedule | Nadia | Always | -- |
| View Payment Schedule (indirectly, through each milestone's payment_trigger) | Owen, Priya | Own-only -- their own client company's projects | -- |
| View Payment Schedule | Dana | Always, inside a logged, read-only support session | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Milestone.status | Defaults to "Defined" | On create only | No (later transitions to Deliverable Uploaded, Approved, Reopened are owned by FEAT-06 and FEAT-08, not user-set here) |
| Milestone.order | Derived and maintained by FEAT-04.SPEC-004's renumbering after any add, remove, or manual reorder | On every add, remove, or reorder | Yes, indirectly -- Nadia's drag/move action is the input that triggers a new derivation, but the stored value is always what FEAT-04.SPEC-004 computes to keep the sequence contiguous |
| Payment Schedule.change_history | Appends a dated entry (timestamp, prior values, new values) on every successful schedule save | On save (create and edit) | No -- always derived, never directly editable |
| Milestone.approved_at, Milestone.approved_by | Set once by FEAT-08.SPEC-003 at the moment of approval | On approval only | No -- immutable once set (ASMP-25) |

## Business Rules

- XBR-10: A milestone that has been approved or invoiced cannot be removed or re-priced; changes go through a logged reopen (FEAT-08) or an invoice correction (FEAT-09); schedule adjustments are dated and never retroactive.
- A schedule or milestone-pricing edit saved after the project has started is recorded as a dated change_history entry and applies only to payment triggers that fire afterward -- a trigger that has already fired (e.g., a deposit invoice already generated at acceptance) uses the schedule exactly as it stood at that moment (dependency map, Payment Schedule Contention note).
- The price-mismatch check never blocks a save -- it is a standing, non-dismissible-until-resolved indicator, since scope can legitimately change (Brief, Primary Flows & Alternates).
- The at-least-one-payment-trigger check is advisory at schedule/milestone save time; it becomes a hard gate only at invoice-generation time, owned by FEAT-09.SPEC-004.
- Concurrent edits to the Payment Schedule across two of Nadia's own sessions resolve reject-with-refresh (dependency map, Payment Schedule Contention note).
- Concurrent actions on a Milestone -- Nadia editing/removing while Owen approves (FEAT-08) -- resolve reject-with-refresh: whichever action is recorded against a stale view is refused and the actor is shown the refreshed state (dependency map, Milestone Contention note).

## Edge Cases

- **Milestone name at exactly 200 characters** -- Passes validation. 201 characters shows the length error.
- **Price entered as exactly 0** -- Treated as invalid; the field must be greater than zero, or the milestone must use "no separate charge" instead.
- **Both price and no_separate_charge left unset on submit** -- Blocked with the cross-field error: "Enter a price, or mark this milestone as no separate charge -- not both."
- **Both price and no_separate_charge appear set at once (e.g., toggling then typing a price in quick succession)** -- The more recently changed control wins and the other is cleared automatically; the two are never both active in the persisted record.
- **Target date left blank** -- Allowed; no downstream payment or approval logic in this feature depends on its presence.
- **Schedule structure changed from Mix to Deposit-only after a completion_amount was already saved** -- completion_amount is cleared and no longer applies to future triggers; a completion invoice already generated under the prior structure is unaffected, per the dated, non-retroactive rule.
- **A milestone reaches Approved status while a Remove attempt is mid-flight (race)** -- The removal is refused with the exact denied message above; the row refreshes to Approved with the Remove control removed.
- **Nadia attempts to edit a milestone and reopen it in the same action** -- Reopening (FEAT-08.SPEC-002) is a distinct, separate action; this spec's Edit authorization on an Approved milestone becomes "Allowed" again only after FEAT-08 records the Reopen event, never as an automatic side effect of an edit attempt.
- **The last remaining payment trigger in the project is removed along with its milestone** -- The removal itself proceeds (the milestone was Defined or Reopened, not Approved/invoiced); the at-least-one-payment-trigger indicator then activates, and FEAT-09 blocks invoicing until a trigger exists again.
- **A milestone's price is edited to a value that newly creates a price mismatch with the accepted proposal** -- The save proceeds and the mismatch banner appears; the edit itself is not blocked by the mismatch.

## Acceptance Criteria

**FEAT-04.SPEC-003-AC-01:** Given Nadia leaves a milestone's name field empty and blurs it, when validation runs, then the field shows "Milestone name is required."

**FEAT-04.SPEC-003-AC-02:** Given Nadia enters a 201-character milestone name, when she blurs the field, then the field shows "Milestone name must be 200 characters or fewer."

**FEAT-04.SPEC-003-AC-03:** Given Nadia enters exactly a 200-character milestone name, when she blurs the field, then no error is shown.

**FEAT-04.SPEC-003-AC-04:** Given Nadia enters 0 as a milestone's price, when she blurs the field, then the field shows "Enter a price greater than zero, or mark this milestone as no separate charge."

**FEAT-04.SPEC-003-AC-05:** Given Nadia enters an invalid date in a milestone's Target Date field, when she blurs the field, then it shows "Enter a valid date."

**FEAT-04.SPEC-003-AC-06:** Given Nadia leaves a milestone's Target Date empty, when she saves, then no error is shown, since the field is optional.

**FEAT-04.SPEC-003-AC-07:** Given Nadia selects "Deposit" as the Payment Schedule's structure and leaves Deposit Amount empty, when she blurs the field, then it shows "Enter a deposit amount greater than zero."

**FEAT-04.SPEC-003-AC-08:** Given Nadia selects "On Completion" as the structure and enters a positive Completion Amount, when she blurs the field, then no error is shown.

**FEAT-04.SPEC-003-AC-09:** Given Nadia has not selected any Payment Schedule structure, when she attempts to save, then she sees "Choose a payment structure."

**FEAT-04.SPEC-003-AC-10:** Given Nadia leaves both price and "no separate charge" unset on a milestone and taps Save, when validation runs, then she sees "Enter a price, or mark this milestone as no separate charge -- not both."

**FEAT-04.SPEC-003-AC-11:** Given Nadia has entered a valid price for a milestone, when she toggles "No separate charge" on afterward, then the price value is cleared and only the toggle remains active.

**FEAT-04.SPEC-003-AC-12:** Given a project's schedule has no deposit, no completion amount, and no milestone marked to trigger on approval, when Nadia saves, then the non-blocking indicator "This project has no payment trigger yet -- no invoice can be generated until you add one" appears and the save still completes.

**FEAT-04.SPEC-003-AC-13:** Given a project's schedule includes at least one milestone marked to trigger on approval, when Nadia saves, then no payment-trigger indicator is shown.

**FEAT-04.SPEC-003-AC-14:** Given the sum of a project's milestone prices does not equal its accepted proposal's total, when Nadia saves, then the price-mismatch banner appears showing both figures and the save still completes.

**FEAT-04.SPEC-003-AC-15:** Given the sum of a project's milestone prices equals its accepted proposal's total, when Nadia saves, then no mismatch banner is shown.

**FEAT-04.SPEC-003-AC-16:** Given Nadia (Freelancer) attempts to create a milestone, when she submits valid data, then the milestone is created, since Create milestone is always allowed for her.

**FEAT-04.SPEC-003-AC-17:** Given Owen (Client Primary Contact) attempts to reach the milestone editor directly, when the page loads, then he is redirected to the read-only Milestone Timeline (FEAT-04.SPEC-002), since Create/Edit/Remove milestone is never allowed for him.

**FEAT-04.SPEC-003-AC-18:** Given Dana (Support Operator) is in a logged support session and attempts to tap Add Milestone, when the tap registers, then the control is disabled showing "Support access is read-only," and no milestone is created.

**FEAT-04.SPEC-003-AC-19:** Given Owen views milestones on his own company's project, when the screen loads, then he sees the full read-only list, since View milestone is always allowed for him on his own company's data.

**FEAT-04.SPEC-003-AC-20:** Given Priya attempts to open a milestone timeline for a project belonging to a different client company, when she follows an out-of-scope link, then she sees a plain explanation and a fresh-link option, never that company's data (XBR-09).

**FEAT-04.SPEC-003-AC-21:** Given Nadia attempts to edit a milestone whose status is Defined, when she changes its name and saves, then the change is accepted, since Edit milestone is allowed while status is Defined.

**FEAT-04.SPEC-003-AC-22:** Given Nadia attempts to edit a milestone whose status is Approved, when she looks for edit controls, then none are enabled, and a direct attempt shows "This milestone has been approved and can't be edited. Reopen it to make changes."

**FEAT-04.SPEC-003-AC-23:** Given Nadia attempts to remove a milestone whose status is Reopened, when she confirms the removal, then it is deleted, since Remove milestone is allowed while status is Reopened.

**FEAT-04.SPEC-003-AC-24:** Given Nadia attempts to remove a milestone whose status is Approved, when she looks for a Remove control, then none is shown, and a direct removal attempt shows "This milestone has been approved or invoiced and can't be removed."

**FEAT-04.SPEC-003-AC-25:** Given Nadia (Freelancer) attempts to create or edit the Payment Schedule, when she submits valid data, then the change is accepted, since this action is always allowed for her.

**FEAT-04.SPEC-003-AC-26:** Given Dana attempts to change the Payment Schedule's structure during a support session, when she attempts the action, then the control is disabled showing "Support access is read-only," and no change is made.

**FEAT-04.SPEC-003-AC-27:** Given a milestone is created with no explicit status set, when it is saved, then its status defaults to "Defined."

**FEAT-04.SPEC-003-AC-28:** Given Nadia saves any change to the Payment Schedule, when the save completes, then a new dated entry recording the prior and new values is appended to change_history.

**FEAT-04.SPEC-003-AC-29:** Given a milestone is approved by Owen, when the approval is recorded (FEAT-08.SPEC-003), then its approved_at and approved_by fields are set once and this feature never alters them afterward.

**FEAT-04.SPEC-003-AC-30:** Given Nadia's schedule is edited by another of her own sessions between her load and save, when she attempts to save, then her save is rejected with a refresh-required message, per the reject-with-refresh Payment Schedule Contention rule.

**FEAT-04.SPEC-003-AC-31:** Given Owen approves a milestone while Nadia is mid-edit on it in another session, when Nadia attempts to save her edit, then it is rejected and she is shown the refreshed, now-Approved milestone, per the reject-with-refresh Milestone Contention rule.

**FEAT-04.SPEC-003-AC-32:** Given a project's schedule structure changes from Mix to Deposit-only after a completion invoice already generated under the prior structure, when the change saves, then the existing completion invoice is unaffected and completion_amount no longer applies to any future trigger.

**FEAT-04.SPEC-003-AC-33:** Given Nadia removes the only milestone in a project carrying a payment trigger, when the removal completes, then the at-least-one-payment-trigger indicator activates and FEAT-09 blocks invoicing on that project until a trigger exists again.

**FEAT-04.SPEC-003-AC-34:** Given Nadia (Freelancer) opens the Milestone & Payment Schedule Editor for one of her own projects, when the screen loads, then she can view the current Payment Schedule, since View Payment Schedule is always allowed for her.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 13 | 13 |
| Cross-Field Rules | 3 | 3 |
| Authorization Rules | 17 | 17 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 10 | 10 |



# Automation Spec: Milestone Reorder Recalculation

## Overview

**Name:** Milestone Reorder Recalculation
**ID:** FEAT-04.SPEC-004
**Type:** Automation
**Purpose:** Renumbers the remaining milestones' order whenever one is added, removed, or manually reordered, so the project's milestone sequence stays contiguous.
**Parent Feature:** FEAT-04 -- Milestone & Payment Schedule Setup

## Scope and Non-Goals

**In Scope:**
- Recalculating and persisting the `order` field for every milestone whose position shifts as a result of an add, a remove, or a manual reorder
- Keeping the project's milestone sequence contiguous (no gaps, no duplicates) at all times after this automation completes
- Returning the recalculated order to the triggering screen for immediate re-render

**Non-Goals:**
- Determining whether a removal is eligible in the first place -- owned by FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules); this automation only renumbers a removal that spec has already allowed, per XBR-10.
- Field validation, price-mismatch flagging, or authorization on milestone actions -- all owned by FEAT-04.SPEC-003; this automation performs no validation of its own beyond confirming the triggering change is well-formed.
- Changing any milestone field other than `order` -- name, price, payment_trigger, target_date, and status are never touched by this automation, even for milestones whose position shifts.
- Recalculating milestone order across projects -- excluded per the dependency map's Milestone relationship line: a milestone "belongs to one Project," and its `order` is scoped and renumbered within that single project only.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Milestone added (create) | FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | Fires immediately after a new milestone is successfully saved | The new milestone's chosen position (end of list by default) and the full ordered list of the project's existing milestones |
| Milestone removed | FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | Fires immediately after a removal is successfully committed -- i.e., after FEAT-04.SPEC-003's approved/invoiced eligibility check passes | The removed milestone's former order value and the full ordered list of the project's remaining milestones |
| Milestone manually reordered | FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | Fires when Nadia drags or uses Move Up/Down controls to move a milestone and the move is committed | The moved milestone's old and new position and the full ordered list of the project's milestones |

## Processing Logic

1. Receive the triggering change: which milestone was added, removed, or moved, and its position (chosen position for an add, former position for a remove, old and new position for a manual reorder).
2. Read the project's current full list of milestones, ordered by their existing `order` value.
3. For an add: insert the new milestone at its chosen position (end of the list by default) and shift every milestone at or after that position down by one position.
4. For a remove: take the milestone out of the list and shift every milestone that was after its former position up by one position, so the sequence stays contiguous starting at 1.
5. For a manual reorder: take the moved milestone out of its old position, shift the milestones between its old and new position by one to close the gap, then insert the moved milestone at its new position.
6. Write the recalculated `order` value to every milestone whose position changed as a result of steps 3-5 -- not only the milestone directly acted on.
7. Skip any milestone whose position did not change -- no write and no event for those records.
8. Return the updated ordered list to the triggering screen (FEAT-04.SPEC-001) for immediate re-render.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Renumbered successfully | The recalculation completes without conflict | The `order` field updates on every milestone whose position shifted | The milestone list on FEAT-04.SPEC-001 re-renders immediately in the new contiguous order | FEAT-04.SPEC-001, FEAT-04.SPEC-002 (client view reflects it on its next read) |
| No-op (zero or one milestone remains) | The project has zero milestones, or exactly one, after the triggering change | No `order` values need changing | The list shows the single milestone at position 1, or the Empty state prompt | FEAT-04.SPEC-001 |
| Not triggered (removal was refused) | The triggering removal was itself refused by FEAT-04.SPEC-003 (approved/invoiced milestone) | None -- this automation never runs, since the removal that would have fired it did not commit | The milestone list is unchanged; FEAT-04.SPEC-003's denied message is shown instead | FEAT-04.SPEC-001, FEAT-04.SPEC-003 |
| Failure (recalculation cannot complete) | An error occurs while writing the recalculated `order` values | Fully rolled back -- either every affected milestone's `order` updates, or none do | Non-blocking warning on FEAT-04.SPEC-001: "Milestones saved, but reordering couldn't be completed. Refresh to see the current order." | FEAT-04.SPEC-001 |

## Data Model

**Reads:** Milestone -- `order`, project reference, and status (status is read only to confirm which milestones belong to the current project; it is never changed here) -- for every milestone in the triggering project.

**Creates:** None.

**Updates:** Milestone -- `order` field only, for every milestone whose position shifted as a result of the triggering add, remove, or reorder.

**Deletes:** None -- the milestone removal itself is performed by FEAT-04.SPEC-001 under FEAT-04.SPEC-003's eligibility rule; this automation only renumbers what remains afterward.

## Business Rules

- Renumbering is atomic: either every affected milestone's `order` updates, or none do (see the Failure outcome) -- a half-renumbered sequence is never left visible to any viewer.
- Order values stay contiguous starting at 1, with no gaps and no duplicates, at all times after this automation completes.
- This automation never runs against a removal that FEAT-04.SPEC-003 has refused (XBR-10) -- it only renumbers milestones whose add, removal, or move has already been committed.
- Approved or invoiced milestones are still included in renumbering -- their position in the sequence can shift when an earlier milestone is added, removed, or reordered -- even though their other fields (price, name) remain locked by FEAT-04.SPEC-003; position is not "pricing or content," so XBR-10's edit lock does not extend to it.

## Edge Cases

- **Adding a milestone at the very end of an already-populated list** -- Only the new milestone receives an `order` write; no existing milestone's order changes.
- **Removing the first milestone in the list** -- Every remaining milestone shifts down by one position; the milestone that was second becomes first.
- **Reordering a milestone to its own current position (a no-op drag)** -- No `order` values change and no write occurs.
- **Reordering an Approved milestone's position** -- Allowed: only its `order` field changes; its name, price, and other locked fields remain untouched and its Approved status is unaffected.
- **Two milestones are added from two open tabs of the same project in rapid succession** -- Each add is processed as a discrete trigger against the milestone list as it existed the moment that add committed; the second add's renumbering includes the milestone the first add just inserted, so the final sequence remains contiguous with both milestones present at distinct positions.
- **Concurrent trigger firing (an add and a removal committed at effectively the same time)** -- Each trigger's renumbering runs against the milestone list state as of its own commit; the trigger that computes second reads the list including the first trigger's already-applied change, so the final order reflects both changes with no gap or duplicate.
- **A trigger fires while a previous run is still in flight** (e.g., Nadia reorders a milestone again before the prior reorder's renumbering finished writing) -- The second reorder is held until the first's writes complete, then runs against the now-current order; FEAT-04.SPEC-001's reorder controls show a brief in-progress state that prevents a third overlapping reorder from being issued in the meantime.
- **The recalculation fails partway through writing** -- All partial writes for that run are rolled back so no milestone is left with an inconsistent `order`; the non-blocking warning appears and a manual refresh re-fetches the current, still-consistent order.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | Triggered by (inbound) | Fires on a milestone add, removal, or manual reorder committed from this screen |
| FEAT-04.SPEC-001 (Milestone & Payment Schedule Editor) | Affects (outbound) | Returns the recalculated order for the editor to re-render immediately |
| FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules) | References (inbound) | Only a removal that FEAT-04.SPEC-003 has allowed ever reaches this automation as a trigger |
| FEAT-04.SPEC-002 (Milestone Timeline, Client View) | Affects (outbound) | The client-facing read-only timeline reflects the recalculated order on its next read |

## Analytics and Success Signals

N/A -- no metric in success-metrics.md measures reorder or renumbering behavior; this automation is a structural-integrity mechanism (keeping `order` contiguous) rather than a product outcome tracked in Stage 2. "Milestone Schedule Completeness" (the metric connected to this feature) is driven by the payment-trigger and creation events emitted in FEAT-04.SPEC-001, not by this automation.

## Acceptance Criteria

**FEAT-04.SPEC-004-AC-01:** Given Nadia has three milestones in a project and adds a fourth at the end of the list, when the add commits, then the new milestone receives order 4 and no existing milestone's order changes.

**FEAT-04.SPEC-004-AC-02:** Given Nadia has four milestones and removes the first one, when the removal commits, then the remaining three milestones renumber to positions 1, 2, and 3, each shifted down by one.

**FEAT-04.SPEC-004-AC-03:** Given Nadia drags the third milestone in a five-milestone list to the first position, when the move commits, then the moved milestone becomes position 1 and the milestones that were first and second each shift down by one position.

**FEAT-04.SPEC-004-AC-04:** Given Nadia has exactly one milestone remaining after a removal, when the removal commits, then no order recalculation write occurs, since the single milestone is already at position 1.

**FEAT-04.SPEC-004-AC-05:** Given Nadia's removal attempt on an Approved milestone is refused by FEAT-04.SPEC-003, when the refusal occurs, then this automation never runs and the milestone list is unchanged.

**FEAT-04.SPEC-004-AC-06:** Given a recalculation fails partway through writing the updated order values, when the failure occurs, then all partial writes for that run are rolled back and the non-blocking warning "Milestones saved, but reordering couldn't be completed. Refresh to see the current order." appears.

**FEAT-04.SPEC-004-AC-07:** Given Nadia drags a milestone to the exact position it already occupies, when the drag completes, then no order values change and no write occurs.

**FEAT-04.SPEC-004-AC-08:** Given Nadia reorders a milestone whose status is Approved, when the move commits, then only its order field changes, and its name, price, and Approved status remain unaffected.

**FEAT-04.SPEC-004-AC-09:** Given Nadia adds a milestone from one browser tab while removing a different milestone from another tab of the same project at effectively the same time, when both commit, then the final order for the project's milestones is contiguous with no gap or duplicate.

**FEAT-04.SPEC-004-AC-10:** Given a reorder is already in flight when Nadia issues a second reorder on the same project, when she issues the second move, then it is held until the first's writes complete and the editor shows a brief in-progress state preventing a third overlapping reorder.

**FEAT-04.SPEC-004-AC-11:** Given two milestones are added to the same project from two open tabs in rapid succession, when both adds commit, then the second add's renumbering includes the first add's already-inserted milestone, leaving both milestones at distinct, contiguous positions.

**FEAT-04.SPEC-004-AC-12:** Given a project's milestone list is successfully renumbered after an add, remove, or reorder, when Owen next opens the Milestone Timeline (FEAT-04.SPEC-002), then he sees the milestones in the recalculated order.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (add, remove, manual reorder) | 3 |
| Outcome Paths | 4 (renumbered, no-op, not triggered, failure) | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 8 | 8 |
