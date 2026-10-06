---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-08.SPEC-006
spec_name: Approval Authorization & Eligibility Rules
spec_slug: approval-authorization-eligibility-rules
parent_feature: FEAT-08
parent_feature_name: Milestone Approval
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 32
acceptance_criteria_count: 22
---

# Logic/Rule Spec: Approval Authorization & Eligibility Rules

## Overview

**Name:** Approval Authorization & Eligibility Rules
**ID:** FEAT-08.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs who may approve or reopen a milestone, the milestone-state and load preconditions that gate each action, the exactly-once and immutability guarantees on an approval, and the concurrency and connectivity conditions that decide whether an attempt is honored or refused.
**Parent Feature:** FEAT-08 -- Milestone Approval
**Governed Entity:** Milestone

## Scope and Non-Goals

**In Scope:**
- Authorization rules for the Approve and Reopen actions on Milestone, per role in the Access Matrix
- Eligibility preconditions that gate Approve (milestone state, deliverable state, comment-thread load) and Reopen (milestone state)
- The exactly-once and immutability guarantees on `approved_at` and `approved_by`
- The concurrency (reject-with-refresh) and connectivity rules that govern a contested or interrupted approval attempt
- Derivation of `status`, `approved_at`, and `approved_by` on a successful approval or reopen

**Non-Goals:**
- Field validation and authorization rules for Milestone's descriptive and pricing fields (name, order, price, no_separate_charge, payment_trigger, target_date) and for Create/View/Edit/Remove actions -- owned by FEAT-04.SPEC-003 (Milestone & Schedule Validation and Edit Rules); this spec only reads `status` as a precondition and never re-defines those rules.
- The atomic write mechanics and refresh behavior of recording an approval -- specified by FEAT-08.SPEC-003 (Approval Recording & Concurrency Guard), which implements the rules this spec defines.
- The atomic write mechanics of recording a reopen -- specified by FEAT-08.SPEC-005 (Reopen Recording), which implements the reopen rules this spec defines.
- A multi-approver or staged sign-off workflow -- excluded per scope-boundaries.md (SC-02): the persona set defines exactly two client-contact roles (Primary, Reviewer) with no additional client-side tiers, so there is no second approver to route a sign-off through.
- A configurable approval workflow or conditional routing -- excluded per scope-boundaries.md (SC-11): the product ships fixed, sensible behavior (approve triggers the next invoice) rather than a configurable workflow or automation builder.

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
| approved_at | date/time | Timestamp of the current approval cycle, written once per approval and never otherwise altered |
| approved_by | text | Identity of the contact who recorded the current approval, written once per approval and never otherwise altered |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-08.SPEC-001 | Milestone Review & Approval Screen | On screen entry (whether the Approve control renders at all, per role) and on every Approve attempt (eligibility and connectivity preconditions) |
| FEAT-08.SPEC-002 | Milestone Reopen Screen | On screen entry (whether the Reopen control renders, per role) and on every Reopen attempt (milestone-state precondition) |
| FEAT-08.SPEC-003 | Approval Recording & Concurrency Guard | Reads this spec's exactly-once, immutability, and reject-with-refresh rules to decide whether an Approve attempt is recorded or refused |
| FEAT-08.SPEC-005 | Reopen Recording | Reads this spec's Reopen eligibility and immutability rules to decide whether a Reopen attempt is recorded |
| FEAT-04.SPEC-003 | Milestone & Schedule Validation and Edit Rules | Reads this spec's Milestone edit-lock outcome (status = Approved) to refuse a concurrent edit or removal attempt by Nadia (XBR-10) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | No validation beyond data type -- governed by FEAT-04.SPEC-003 | Always | -- | -- | -- |
| order | No validation beyond data type -- governed by FEAT-04.SPEC-003 | Always | -- | -- | -- |
| price | No validation beyond data type -- governed by FEAT-04.SPEC-003 | Always | -- | -- | -- |
| no_separate_charge | No validation beyond data type -- governed by FEAT-04.SPEC-003 | Always | -- | -- | -- |
| payment_trigger | No validation beyond data type -- governed by FEAT-04.SPEC-003 | Always | -- | -- | -- |
| target_date | No validation beyond data type -- governed by FEAT-04.SPEC-003 | Always | -- | -- | -- |
| status | Must be "Deliverable Uploaded" or "Reopened" for an Approve attempt to be recorded; must be "Approved" for a Reopen attempt to be recorded. Any other status refuses the respective action. | Always | On Approve attempt (FEAT-08.SPEC-003); on Reopen attempt (FEAT-08.SPEC-005) | "This milestone can't be approved right now -- it may have already been approved or its deliverable removed. Refreshing to show the current state." (Approve) / "This milestone can only be reopened once it has been approved." (Reopen) | Yes |
| approved_at | Set exactly once per approval cycle by FEAT-08.SPEC-003 at the moment an approval is successfully recorded; never directly editable by any role, on any screen | Always | On every Approve attempt (write path only) | N/A -- no direct-entry field exists for this value | Yes (write-protected) |
| approved_by | Set exactly once per approval cycle by FEAT-08.SPEC-003 to the identity of the approving Client Contact; never directly editable by any role, on any screen | Always | On every Approve attempt (write path only) | N/A -- no direct-entry field exists for this value | Yes (write-protected) |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Approval fields are set atomically or not at all | status, approved_at, approved_by | On a successful Approve, all three change together in one write: status -> "Approved", approved_at -> current timestamp, approved_by -> the approving contact's identity. A refused attempt (state mismatch, concurrency conflict, or connectivity loss) leaves all three fields exactly as they were -- there is no partial write. | N/A -- enforced as an atomic operation, not surfaced as a field error |
| Reopen never rewrites approval history | status, approved_at, approved_by | A successful Reopen changes status -> "Reopened" only; it never clears or edits approved_at or approved_by. Those fields continue to display the most recent approval's data until a fresh Approve overwrites them together (per the rule above). | N/A -- enforced as a scoped write, not surfaced as a field error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Approve milestone | Owen (Client Primary Contact) | Milestone belongs to Owen's own client company (Own-only, XBR-09), status is "Deliverable Uploaded" or "Reopened" (not already "Approved"), the milestone's current Deliverable and its comment thread have fully loaded, and Owen has connectivity at the moment he acts | The Approve control is disabled while any precondition (load, connectivity) is unmet, with the specific reason shown per FEAT-08.SPEC-001's States; if the milestone is already Approved, the control is replaced entirely by the "Approved on {date}" marker -- there is no separate denial message because the action is not offered |
| Approve milestone | Priya (Client Reviewer Contact) | Never | The Approve control is not rendered for Priya at all; she sees the same milestone status as Owen with no path to an approve action, per the Access Matrix's Reviewer entitlement |
| Approve milestone | Nadia (Freelancer) | Never | Nadia has no approval surface for her own client's milestone -- the action belongs exclusively to the client's Primary Contact (Key Capabilities); no control is shown on any freelancer-facing screen |
| Approve milestone | Dana (Support Operator) | Never | Dana's read-only support session (FEAT-31) never renders the Approve control; if a support-session render path were reached in error, the control would show disabled with "Support access is read-only." |
| Reopen milestone | Nadia (Freelancer) | Milestone status is "Approved" | The Reopen control is not shown while status is "Defined," "Deliverable Uploaded," or already "Reopened" -- there is nothing to reopen; a direct attempt outside the "Approved" state shows "This milestone can only be reopened once it has been approved." |
| Reopen milestone | Owen, Priya (Client Contacts) | Never | No reopen surface exists in the client portal for either role, per the Key Capability "Reopen (freelancer only)" -- the action is never offered to a client contact under any circumstance |
| Reopen milestone | Dana (Support Operator) | Never | Dana's read-only support session never renders the Reopen control; if reached in error, the control shows disabled with "Support access is read-only." |
| View approval status ("Approved on {date}" marker, or current pending state) | Nadia (Freelancer) | Always, across all of her own projects | -- |
| View approval status | Owen, Priya (Client Contacts) | Own-only -- their own client company's projects (XBR-09) | An out-of-scope milestone link is never reachable; it shows a plain explanation and a fresh-link option, never another company's data |
| View approval status | Dana (Support Operator) | Always, inside a logged, read-only support session (FEAT-31) | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status | Derived to "Approved" when FEAT-08.SPEC-003 successfully records an approval | On a successful Approve action only | No -- always system-derived from the recorded outcome |
| status | Derived to "Reopened" when FEAT-08.SPEC-005 successfully records a reopen | On a successful Reopen action only | No -- always system-derived from the recorded outcome |
| approved_at | Set to the current timestamp at the moment the approval write commits | On a successful Approve action only | No -- never user-entered, never editable afterward |
| approved_by | Set to the identity (name and Client Contact reference) of the approving contact, resolved from the authenticated session at the moment the approval write commits | On a successful Approve action only | No -- never user-entered, never editable afterward |

## Business Rules

- **Exactly-once per approval cycle (XBR-04):** A single successful Approve action fully consumes the eligibility window for that cycle -- `status` moves out of "Deliverable Uploaded"/"Reopened" the instant the write commits, so no second Approve attempt against the same pre-approval state can ever be recorded; a retried or double-submitted attempt is refused by FEAT-08.SPEC-003's atomic check-and-write, never double-recorded.
- **Immutability of the current approval record (ASMP-25, XBR-04):** Once set, `approved_at` and `approved_by` are never altered, deleted, or reassigned by any role on any screen -- not by Owen, not by Nadia, not by Dana. The only way the milestone's approval state changes at all is a new Reopen-then-Approve cycle, which is itself a new, separately logged event (FEAT-13, XBR-05), never an edit to the prior one.
- **Reopen is a distinct event, never a silent edit:** A reopen changes only `status`; it does not retroactively alter the approval record it supersedes for display. The full, permanent history of every approval and reopen -- including a milestone approved, reopened, and approved again -- lives in the immutable Activity Log (FEAT-13); the Milestone entity's own `approved_at`/`approved_by` fields reflect only the current cycle's most recent approval, consistent with the entity's "written once on approval, never altered" definition (feature-dependency-map.md, Entity: Milestone).
- **Concurrency is reject-with-refresh, never last-write-wins or merge (feature-dependency-map.md, Milestone Contention note):** An Approve attempt is evaluated against the milestone state Owen was shown when the screen loaded, not the state at click time. If Nadia re-priced, removed, or otherwise altered the milestone since Owen's screen loaded, the attempt is refused and Owen is shown the refreshed milestone instead of having his approval recorded against stale data.
- **Approval requires connectivity:** The Approve action can be initiated only while the client's device has connectivity; it is never queued or optimistically recorded offline, because an approval is a timestamped, permanent, evidentiary act that must reflect the moment it genuinely occurred (BRIEF.md, Constraints: record immutability).
- **Approval requires a fully loaded review context:** The Approve control is not actionable until the milestone's current Deliverable and its comment thread have both finished loading on FEAT-08.SPEC-001 -- an approval decided before the reviewer has seen everything they were shown to see is not a genuine decision.
- **XBR-10 (read-only reference):** A milestone that has been approved or invoiced cannot be removed or re-priced by Nadia; this spec's exactly-once and immutability rules are the FEAT-08 side of that same cross-feature guarantee -- FEAT-04.SPEC-003 owns the edit-lock rule itself.

## Edge Cases

- **Owen taps Approve twice in rapid succession (double-submit)** -- The first attempt's write begins moving `status` out of the eligible state immediately; the second attempt is evaluated against the now-current (or in-flight) state and is refused as if the milestone were already approved, never recorded as a second approval.
- **Owen's Approve attempt arrives after Nadia removed the milestone's only Deliverable in the meantime** -- Refused: the Deliverable precondition is no longer met. Owen is shown the refreshed milestone, which now has no reviewable deliverable and no Approve control.
- **Nadia attempts to reopen a milestone that was never approved (status "Defined" or "Deliverable Uploaded")** -- Refused with "This milestone can only be reopened once it has been approved." -- there is no prior approval to undo.
- **Nadia attempts to reopen a milestone that is already "Reopened"** -- Refused with the same message; a milestone can be reopened only once per completed approval cycle, and a second reopen has nothing further to do until a new approval occurs.
- **A milestone is reopened and re-approved, then reopened again** -- Each cycle is independent: the second approval's `approved_at`/`approved_by` overwrote the first atomically (per the Cross-Field Rule above), and the second reopen again changes only `status`. All four events remain individually visible and unaltered in the Activity Log (FEAT-13).
- **Owen loses connectivity between tapping Approve and the response returning** -- The action is never optimistically applied; if the write cannot be confirmed as committed, the milestone shows its prior state and Owen sees the offline/reconnect experience (FEAT-08.SPEC-001, States) rather than an ambiguous "maybe approved" state.
- **Two of Owen's own sessions (e.g., laptop and phone) both attempt to approve the same milestone at effectively the same time** -- Exactly-once holds regardless of which session sent the first commit: the first write to commit succeeds: the second is evaluated against the now-"Approved" state and refused with the same "already approved" experience Priya-adjacent stale attempts receive, showing the now-Approved milestone.
- **Dana's read-only session is open on a milestone at the exact moment Owen approves it** -- No conflict: Dana never has a write path, so her session simply reflects the updated state on next read; her view is never itself in contention.

## Acceptance Criteria

**FEAT-08.SPEC-006-AC-01:** Given Owen (Client Primary Contact) is viewing a milestone in his own client company's project with status "Deliverable Uploaded" and its deliverable and comment thread fully loaded, when he taps Approve while connected, then the approval is recorded and status moves to "Approved."

**FEAT-08.SPEC-006-AC-02:** Given Owen is viewing a milestone whose status is already "Approved," when the screen renders, then no Approve control is shown -- only the "Approved on {date}" marker.

**FEAT-08.SPEC-006-AC-03:** Given Priya (Client Reviewer Contact) opens the same milestone Owen can approve, when the screen renders, then no Approve control appears anywhere for her.

**FEAT-08.SPEC-006-AC-04:** Given Nadia (Freelancer) opens her own client's milestone review context, when she looks for an approve action, then none exists on any freelancer-facing screen.

**FEAT-08.SPEC-006-AC-05:** Given Dana (Support Operator) is in a logged, read-only support session viewing the milestone, when the screen renders, then no Approve control is shown.

**FEAT-08.SPEC-006-AC-06:** Given Nadia is viewing a milestone with status "Approved," when she looks for a Reopen action, then it is available and enabled.

**FEAT-08.SPEC-006-AC-07:** Given Nadia is viewing a milestone with status "Defined" or "Deliverable Uploaded," when she looks for a Reopen action, then none is shown, and a direct attempt shows "This milestone can only be reopened once it has been approved."

**FEAT-08.SPEC-006-AC-08:** Given Owen or Priya are in the client portal, when either looks for a way to reopen any milestone, then no such action exists anywhere in their portal.

**FEAT-08.SPEC-006-AC-09:** Given Dana is in a support session, when she looks for a Reopen action, then none is shown.

**FEAT-08.SPEC-006-AC-10:** Given Owen views his own client company's milestone, when the screen loads, then he can see its current status and history, since View approval status is always allowed for him on his own company's data.

**FEAT-08.SPEC-006-AC-11:** Given Priya follows a milestone link belonging to a different client company, when the link resolves, then she sees a plain explanation and a fresh-link option, never that company's data (XBR-09).

**FEAT-08.SPEC-006-AC-12:** Given Owen taps Approve twice in rapid succession, when the first tap's write is already in flight, then the second tap is refused as already-approved and no second approval is recorded.

**FEAT-08.SPEC-006-AC-13:** Given Nadia re-prices or removes the milestone's deliverable after Owen's approval screen loaded but before he taps Approve, when Owen taps Approve, then the attempt is refused and Owen is shown the refreshed milestone instead of having his approval recorded.

**FEAT-08.SPEC-006-AC-14:** Given the milestone's deliverable and comment thread have not yet finished loading on Owen's screen, when he looks for the Approve control, then it is disabled until loading completes.

**FEAT-08.SPEC-006-AC-15:** Given Owen has no connectivity, when he attempts to tap Approve, then the action does not proceed and he sees the offline "reconnect to approve" state rather than any success feedback.

**FEAT-08.SPEC-006-AC-16:** Given a milestone is successfully approved, when the write commits, then `status`, `approved_at`, and `approved_by` all change together in the same operation -- no partial state is ever observed.

**FEAT-08.SPEC-006-AC-17:** Given a milestone is successfully reopened, when the write commits, then only `status` changes to "Reopened"; `approved_at` and `approved_by` retain the values from the approval being reopened.

**FEAT-08.SPEC-006-AC-18:** Given a milestone was approved, reopened, and approved again, when the second approval commits, then `approved_at` and `approved_by` reflect the second approval, and both the first and second approval events remain individually visible, unaltered, in the Activity Log (FEAT-13).

**FEAT-08.SPEC-006-AC-19:** Given a milestone is already "Approved," when any role attempts to directly alter `approved_at` or `approved_by`, then no such control or path exists on any spec -- the fields are write-protected outside the approval and reopen automations.

**FEAT-08.SPEC-006-AC-20:** Given the same milestone receives two Approve attempts from two of Owen's own sessions at effectively the same moment, when the first commits, then the second is evaluated against the now-"Approved" state and refused, showing the now-Approved milestone.

**FEAT-08.SPEC-006-AC-21:** Given a milestone has been approved and invoiced, when Nadia attempts to edit or remove it from FEAT-04.SPEC-001, then FEAT-04.SPEC-003's edit-lock rule refuses the attempt, consistent with this spec's exactly-once and immutability guarantees (XBR-10).

**FEAT-08.SPEC-006-AC-22:** Given Dana has a read-only support session open on a milestone at the moment Owen approves it, when Dana next reads the milestone, then she sees the updated "Approved" state with no conflict or error of her own.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 9 | 9 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 10 | 10 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 7 | 7 |
| Edge Cases | 8 | 8 |
