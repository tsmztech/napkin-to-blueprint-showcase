---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-04.SPEC-003
spec_name: Milestone & Schedule Validation and Edit Rules
spec_slug: milestone-schedule-validation-and-edit-rules
parent_feature: FEAT-04
parent_feature_name: Milestone & Payment Schedule Setup
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 43
acceptance_criteria_count: 34
---

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
