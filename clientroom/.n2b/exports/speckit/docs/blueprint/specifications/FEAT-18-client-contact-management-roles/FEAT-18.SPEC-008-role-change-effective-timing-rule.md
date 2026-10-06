---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-18.SPEC-008
spec_name: Role Change Effective-Timing Rule
spec_slug: role-change-effective-timing-rule
parent_feature: FEAT-18
parent_feature_name: Client Contact Management & Roles
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 3
acceptance_criteria_count: 11
---

# Logic/Rule Spec: Role Change Effective-Timing Rule

## Overview

**Name:** Role Change Effective-Timing Rule
**ID:** FEAT-18.SPEC-008
**Type:** Logic/Rule
**Purpose:** Ensures a contact's role change applies to their future actions only, and never alters the record of a past approval or acceptance they gave under their prior role.
**Parent Feature:** FEAT-18 -- Client Contact Management & Roles
**Governed Entity:** Client Contact (role field, in relation to timestamped evidentiary records elsewhere in the product)

## Scope and Non-Goals

**In Scope:**
- The rule that a role change takes effect only for actions the contact performs after the change is saved
- The guarantee that a proposal acceptance, milestone approval, or any other evidentiary record already attributed to a contact is never re-evaluated or altered when that contact's role later changes
- How an in-progress action (one the contact started before the change but has not yet completed) is resolved

**Non-Goals:**
- Who may change a role at all -- owned by FEAT-18.SPEC-007 (Role Authorization Rules); this spec governs only the timing of an authorized change's effect
- The last-Primary block that can prevent a role change from being saved in the first place -- owned by FEAT-18.SPEC-006 (Primary Contact Requirement Rule); this spec applies only once a role change has actually been saved
- Field-level validation of the role value itself -- owned by FEAT-18.SPEC-005 (Contact Field Validation Rules)
- Writing the audit-trail entry for the role change -- owned by Immutable Activity & Audit Trail (FEAT-13, XBR-05); this spec defines the behavioral guarantee the trail entry documents, not the trail-writing mechanism itself

## Governed Entity

**Entity:** Client Contact
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| role | enum (Primary, Reviewer) | The field this rule governs the effective timing of when changed |
| invited_by | text (reference) | Unaffected by a role change -- retains the original inviting party regardless of later role changes |
| status | enum (Invited, Active, Removed) | Unaffected by a role change on its own |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-18.SPEC-002 | Add or Edit Client Contact | At the moment a role change is saved -- the new role governs only actions the contact performs from that moment forward |
| FEAT-03 (Proposal Acceptance) | Acceptance recording | Reads the contact's role at the moment of acceptance, not retroactively re-evaluated after a later role change |
| FEAT-08 (Milestone Approval) | Approval recording | Reads the contact's role at the moment of approval, not retroactively re-evaluated after a later role change |
| FEAT-13 (Immutable Activity & Audit Trail) | Trail entry attribution | Every past evidentiary entry keeps the role-at-the-time context it was written with, per ASMP-25's record-immutability guarantee |

## Field Validation Rules

This spec governs the timing of an already-valid role change's effect, not the role value's format. No field validation rules apply here beyond FEAT-18.SPEC-005.

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| role | No validation beyond data type at this spec's layer -- see FEAT-18.SPEC-005 for format and FEAT-18.SPEC-006 for the last-Primary condition | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Future-only effect | role, and every timestamped evidentiary field elsewhere (Proposal.accepted_by, Milestone.approved_by) | A role change's saved timestamp establishes a boundary: any action the contact performs after that timestamp is governed by the new role; any action already recorded before that timestamp remains governed by, and attributed under, the role in effect when it was recorded | N/A -- this is a system-enforced timing guarantee with no user-facing violation state; there is no way for a user to request retroactive re-attribution |

## Authorization Rules

This rule applies automatically to every authorized role change; it grants or denies no action of its own. The single row below documents that this spec's guarantee is unconditional once a change is authorized and saved.

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Apply future-only timing to a saved role change | The system, automatically, for every role change Nadia saves | Always -- unconditional, no role or condition can opt out of it | N/A -- there is no path to disable or bypass this guarantee; it is not a user-facing permission |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Effective role for a given past action | Derived as the role value in effect on the Client Contact record at the exact timestamp that action's own record (Proposal.accepted_at, Milestone.approved_at) was written | Whenever a past action's role context is displayed or referenced (e.g., in the activity trail) | No -- always derived from history, never recalculated against the contact's current role |
| Effective role for a new action | The Client Contact's current role field, read fresh at the moment the new action begins | Every time a contact attempts an action gated by role (accept, approve, invite, etc.) | No -- always the current value, never a cached value from earlier in the session |

## Business Rules

- A role change never alters the record of a past approval or acceptance (feature-overview.md, Primary Flows & Alternates: "the change applies to future actions only, never retroactively altering a past approval"); this is a direct extension of record immutability (ASMP-25, XBR-04) applied specifically to role context.
- The scenario this rule exists to serve: a second Primary is added for co-founders, or a Reviewer is promoted to Primary; from that moment they can accept, approve, and pay, but everything they did as a Reviewer beforehand (comments, prior views) remains attributed to them as it was recorded, with no reinterpretation.
- Demoting a Primary to Reviewer (subject to FEAT-18.SPEC-006's last-Primary block) works identically in the other direction: any acceptance or approval that contact gave while Primary remains valid evidence under their name, even though they can no longer perform such actions going forward.
- This rule has no bearing on whether a role change itself is permitted -- that is entirely FEAT-18.SPEC-006 and FEAT-18.SPEC-007's domain; this spec only governs what happens to time once a change is saved.

## Edge Cases

- **A contact begins accepting a proposal (opens the accept screen) and their role is changed to Reviewer by Nadia before they tap Accept** -- The accept action, not yet completed, is evaluated against the contact's role at the moment of the action's completion (tapping Accept), not at the moment they opened the screen; since Reviewer contacts cannot accept, the action is now denied per FEAT-18.SPEC-007, and the client-facing screen shows the standard access-denied experience for that action.
- **A Reviewer is promoted to Primary, and immediately attempts to accept a proposal that was sent before the promotion** -- The action is permitted: the proposal's acceptance was never attempted under the old role, so there is no past record to protect, and the current role at the moment of acceptance is Primary.
- **Nadia changes a contact's role twice in quick succession (Primary to Reviewer, then back to Primary)** -- Each change is its own timestamped event; any action attempted between the two changes is governed by whichever role was in effect at that exact moment, and both changes are independently visible in the activity trail (FEAT-13).
- **A role change is saved while the contact has a comment (FEAT-07) already posted under their prior role** -- The comment's author attribution is unaffected; comments carry no role-gated content restriction beyond what governed posting it at the time, and a Reviewer's existing comments remain visible exactly as posted regardless of a later promotion or demotion.
- **The client's sole Primary contact is demoted the instant after an invoice's pay link is generated in their name** -- Not retroactively affected: the pay link and invoice remain addressed to that contact as recorded; whether they can still access the client portal at all going forward is a separate concern governed by FEAT-05, unaffected by this rule (this rule concerns role-gated entitlement, not portal access itself, which persists for a Reviewer-demoted former Primary).
- **A role change is attempted concurrently with the very action it would affect (e.g., Nadia demotes Owen at the same instant Owen taps Approve on a milestone)** -- Whichever write commits first determines the outcome: if the demotion commits first, Owen's approval attempt is evaluated as a Reviewer and denied; if the approval commits first, it is recorded under Primary and the demotion that follows does not undo it. There is no scenario where both are recorded inconsistently, since each is a single atomic commit evaluated against the role value at its own commit time.

## Acceptance Criteria

**FEAT-18.SPEC-008-AC-01:** Given Owen accepted a proposal as a Primary contact, when Nadia later changes his role to Reviewer, then the prior acceptance record remains unchanged and still shows Owen as the accepting party.

**FEAT-18.SPEC-008-AC-02:** Given Priya is promoted from Reviewer to Primary, when she subsequently attempts to accept a proposal, then the acceptance is permitted, since her current role at the moment of acceptance is Primary.

**FEAT-18.SPEC-008-AC-03:** Given a contact's role is changed from Primary to Reviewer, when that contact attempts to approve a milestone after the change, then the approval is denied, per their new current role.

**FEAT-18.SPEC-008-AC-04:** Given Owen has the proposal-accept screen open and Nadia changes his role to Reviewer before he taps Accept, when he taps Accept, then the action is evaluated against his role at that moment (Reviewer) and is denied.

**FEAT-18.SPEC-008-AC-05:** Given Nadia changes a contact's role from Primary to Reviewer and back to Primary within the same day, when the activity trail is viewed, then both role-change events appear as separate, independently timestamped entries.

**FEAT-18.SPEC-008-AC-06:** Given a Reviewer contact posted comments before being promoted to Primary, when their comment history is viewed after the promotion, then those comments remain visible and attributed to them exactly as posted, unaffected by the later promotion.

**FEAT-18.SPEC-008-AC-07:** Given a client's sole Primary contact is demoted to Reviewer immediately after an invoice's pay link was generated in their name, when the invoice is later viewed, then the pay link and invoice remain addressed to that contact as originally recorded.

**FEAT-18.SPEC-008-AC-08:** Given Owen taps Approve on a milestone at the same instant Nadia's demotion of him commits first, when the approval attempt is then evaluated, then it is denied under his now-current Reviewer role.

**FEAT-18.SPEC-008-AC-09:** Given Owen's approval commits before Nadia's concurrent demotion of him, when the demotion is applied afterward, then the already-recorded approval is not undone or altered.

**FEAT-18.SPEC-008-AC-10:** Given a contact's role has never changed since creation, when any of their past or future actions are evaluated, then this rule has no observable effect, since there is only ever one role value in their history.

**FEAT-18.SPEC-008-AC-11:** Given Nadia views a past milestone approval in the activity trail for a contact whose role has since changed, when she reads the entry, then it shows the role the contact held at the time of that approval, not their current role.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 1 | 1 |
| Cross-Field Rules | 1 | 1 |
| Authorization Rules | 1 | 1 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
