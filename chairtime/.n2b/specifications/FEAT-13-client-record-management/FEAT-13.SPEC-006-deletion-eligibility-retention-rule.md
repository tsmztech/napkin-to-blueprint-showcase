---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-13.SPEC-006
spec_name: Deletion Eligibility & Retention Rule
spec_slug: deletion-eligibility-retention-rule
parent_feature: FEAT-13
parent_feature_name: Client Record Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 12
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Deletion Eligibility & Retention Rule

## Overview

**Name:** Deletion Eligibility & Retention Rule
**ID:** FEAT-13.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs when a client record may be deleted (blocked by an upcoming booking), the irreversibility of deletion, and the retention/de-identification of related records afterward.
**Parent Feature:** FEAT-13 -- Client Record Management
**Governed Entity:** Client record (deletion lifecycle)

## Scope and Non-Goals

**In Scope:**
- The eligibility condition that gates deletion (no upcoming booking exists)
- The irreversibility of deletion once executed
- What is retained versus removed after deletion (contact details and notes removed; Messaging Consent cascade-deleted; financial and timeline history retained de-identified)
- The concurrency rule for an edit attempted on a record that has just been deleted

**Non-Goals:**
- Field-level validation for name, phone, email, and private_note -- governed entirely by FEAT-13.SPEC-005 (Client Field Validation & Access Rules); this spec's Field Validation Rules section below defers to it for every field
- The confirmation UI and eligibility-result display -- handled by FEAT-13.SPEC-003 (Client Deletion Confirmation); this spec defines the rule, that spec presents it
- The mechanics of performing the deletion write itself -- handled by FEAT-13.SPEC-004 (Client Deletion Execution); this spec defines what is and is not allowed, that spec carries it out
- Cancelling the blocking booking -- owned by FEAT-30 (Pro Booking Management)'s cancel-with-full-refund flow; this spec only defines that an upcoming booking blocks deletion, not how it is resolved

## Governed Entity

**Entity:** Client
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| name | text | Client's full name |
| phone | text | Client's phone number |
| email | text | Client's email address |
| private_note | text | The Pro's own private note about this client |
| booking_notes | text | The client's own optional per-booking note, captured elsewhere |
| booking_history | derived | This client's Bookings with this Pro |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-13.SPEC-003 | Client Deletion Confirmation | Eligibility checked on screen load, before the Pro can confirm |
| FEAT-13.SPEC-004 | Client Deletion Execution | Eligibility re-checked at the moment of execution; retention/de-identification rules applied during processing |
| FEAT-13.SPEC-001 | Client Record Detail | The post-deletion concurrent-edit refusal rule applies to any save attempted here after the record is gone |
| FEAT-13.SPEC-002 | Client Contact Edit | Same post-deletion concurrent-edit refusal rule applies to any save attempted here after the record is gone |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| name | No validation beyond data type in this spec -- governed by FEAT-13.SPEC-005 | Always | -- | -- | -- |
| phone | No validation beyond data type in this spec -- governed by FEAT-13.SPEC-005 | Always | -- | -- | -- |
| email | No validation beyond data type in this spec -- governed by FEAT-13.SPEC-005 | Always | -- | -- | -- |
| private_note | No validation beyond data type in this spec -- governed by FEAT-13.SPEC-005 | Always | -- | -- | -- |
| booking_notes | No validation beyond data type in this spec -- read-only from this feature, governed elsewhere | Always | -- | -- | -- |
| booking_history | No validation beyond data type in this spec -- derived, read-only | Always | -- | -- | -- |

## Cross-Field Rules

N/A -- this spec's core rule is a cross-entity eligibility gate between the Client and Booking entities (no upcoming Booking may exist), not a same-entity cross-field rule. That gate is defined under Business Rules and Authorization Rules below, since it governs an action (deletion) rather than a field's own valid values.

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Delete client record | The Pro (Talia) | Own clients only, and only when no Booking exists for this client with a state of Pending Payment, Confirmed, or Awaiting Outcome whose appointment time has not yet passed (i.e., no upcoming booking) | When an upcoming booking exists: FEAT-13.SPEC-003 shows the Blocked panel naming that booking's date, time, and service, with a "Cancel booking with full refund" route into FEAT-30 -- never a bare "cannot delete" message |
| Delete client record | Platform Operator (Support) | Never | No delete control exists anywhere in Support's read-only view, per scope-boundaries.md SC-05 |
| Delete client record | The Client (Riley) | Never | A Client cannot request or execute their own deletion in-product, per scope-boundaries.md SC-01; deletion happens only after an informal request to the Pro |
| Undo or restore a deleted client record | The Pro (Talia) | Never -- no role may reverse a completed deletion | No undo control exists anywhere in the product; the Brief's Non-Goals state deletion "is not reversible, consistent with 'delete on request' meaning delete" |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Deletion eligibility (derived, not a stored field) | True when zero Bookings for this client are in a Pending Payment, Confirmed, or Awaiting Outcome state with a future or in-progress appointment time; false otherwise | Recalculated every time FEAT-13.SPEC-003 loads and again immediately before FEAT-13.SPEC-004 executes | No -- the Pro cannot override an ineligible result; they must first cancel the blocking booking through FEAT-30 |
| De-identified client reference (on retained Booking, Deposit Transaction, and Activity Event records) | A marker that replaces the deleted client's identity while leaving service, timing, amount, and outcome fields intact | Applied once, at the moment of successful deletion (FEAT-13.SPEC-004) | No -- de-identification is never reversed and never re-linked to a new client record, even if the same person books again |
| Messaging Consent retention exception | Where law requires evidence of a consent action to be kept, the minimum required evidence is retained stripped of the phone number and any other identifying field beyond that evidence itself | Applied once, at the moment of successful deletion (FEAT-13.SPEC-004) | No |

## Business Rules

- XBR-19: client deletion is refused while an upcoming booking exists (the Pro is offered cancel-with-full-refund); removes contact details, notes, and consent; retains financial and timeline records only in de-identified form; a later booking creates a new record.
- A completed or no-show booking, a cancelled booking, and a rescheduled-away-from booking do not count as "upcoming" for this rule -- only Pending Payment, Confirmed, and Awaiting Outcome states with a not-yet-passed appointment time block deletion, since those are the states in which cancelling and refunding the client still matters (per XBR-12's booking outcome windows).
- Per the dependency map's Contention note for the Client entity, once a client record is deleted, any concurrent edit attempted against it (a private-note save, a contact-field save) is refused and the Pro is prompted to refresh -- there is no partial or stale-write path into a deleted record.
- Per the dependency map's Contention note, a later booking from the same person after deletion always creates a new, unconnected Client record; the deletion automation (FEAT-13.SPEC-004) never re-links a new booking to the de-identified history left behind by a previous deletion.
- Deletion is all-or-nothing (FEAT-13.SPEC-004): a client record is never left partially deleted, and eligibility is re-checked at execution time, not trusted from the confirmation screen's earlier check alone.

## Edge Cases

- **A booking is cancelled by the Client themselves (via FEAT-10) between the Pro opening FEAT-13.SPEC-003 and confirming deletion** -- The confirmation screen's eligibility snapshot is now stale (it may still show Blocked); the Pro must navigate away and back to see the updated Eligible result, since this rule's eligibility check is not live-updating.
- **The blocking booking passes its appointment time and becomes Awaiting Outcome while the Blocked panel is displayed** -- It still counts as blocking under this rule (Awaiting Outcome is one of the three blocking states) until it resolves to Completed or No-Show or is otherwise cancelled.
- **Exactly one booking exists and it is in the Awaiting Outcome state (appointment has passed but outcome not yet marked)** -- Deletion remains blocked; the Pro must wait for FEAT-12's auto-completion sweep or manually mark the outcome (FEAT-11/FEAT-12) before the client becomes eligible, or cancel is no longer offered once the appointment has passed (per XBR-12, a passed appointment can no longer be cancelled, only completed or marked no-show).
- **Deletion is attempted twice from two devices for the same client at effectively the same time** -- The first execution to reach FEAT-13.SPEC-004's write step succeeds; the second re-checks eligibility, finds the client record already gone, and fails with "This client's record no longer exists," per FEAT-13.SPEC-004's own concurrency handling.
- **A private-note edit (FEAT-13.SPEC-001) is saved a moment after deletion completes on another device** -- The save is refused per this rule's post-deletion concurrent-edit refusal, with the exact message defined in FEAT-13.SPEC-001's edge cases ("This client's record no longer exists. It may have been deleted.").
- **The Pro deletes a client, then that same phone number books again a year later** -- A brand-new Client record is created by FEAT-05 with no reference to the deleted record or its de-identified history; the de-identified Booking and Deposit Transaction records from the deletion remain permanently unconnected to the new record.

## Acceptance Criteria

**FEAT-13.SPEC-006-AC-01:** Given Talia opens FEAT-13.SPEC-003 for a client with a Confirmed booking three days from now, when the eligibility check runs, then deletion is blocked and the Blocked panel names that booking.

**FEAT-13.SPEC-006-AC-02:** Given Talia opens FEAT-13.SPEC-003 for a client with only Completed and Cancelled bookings, when the eligibility check runs, then deletion is eligible.

**FEAT-13.SPEC-006-AC-03:** Given a client's only booking is in the Awaiting Outcome state (appointment time passed, outcome not yet marked), when the eligibility check runs, then deletion remains blocked.

**FEAT-13.SPEC-006-AC-04:** Given Talia cancels the blocking booking through FEAT-30's cancel-with-full-refund route and returns to FEAT-13.SPEC-003, when the eligibility check re-runs, then deletion is now eligible.

**FEAT-13.SPEC-006-AC-05:** Given Talia confirms deletion for an eligible client, when FEAT-13.SPEC-004 executes, then the deletion is irreversible and no undo control is ever presented to her afterward.

**FEAT-13.SPEC-006-AC-06:** Given a deleted client's Messaging Consent required no legally mandated evidence retention, when deletion executes, then that consent record is fully removed along with the contact details.

**FEAT-13.SPEC-006-AC-07:** Given a deleted client's Deposit Transaction history exists, when deletion executes, then those records are retained with a de-identified client reference, keeping service, timing, amount, and outcome intact.

**FEAT-13.SPEC-006-AC-08:** Given a client record was deleted, when the Pro attempts to save a private-note edit against it from a stale screen, then the save is refused with "This client's record no longer exists. It may have been deleted."

**FEAT-13.SPEC-006-AC-09:** Given a client record was deleted a year ago, when that same phone number books again through FEAT-05, then a brand-new, unconnected Client record is created rather than resurrecting the old one.

**FEAT-13.SPEC-006-AC-10:** Given Platform Operator (Support) views any client's record, when Support looks for a delete control, then none exists, per scope-boundaries.md SC-05.

**FEAT-13.SPEC-006-AC-11:** Given Riley (the Client) wants her record deleted, when she looks for an in-product way to request or execute it herself, then none exists -- she must ask the Pro informally, per scope-boundaries.md SC-01.

**FEAT-13.SPEC-006-AC-12:** Given two devices attempt to confirm deletion for the same client at effectively the same time, when both executions run, then the first to complete succeeds and the second fails with "This client's record no longer exists."

**FEAT-13.SPEC-006-AC-13:** Given a client is cancelled by the client themselves between FEAT-13.SPEC-003's load and Talia's confirmation tap, when Talia confirms deletion anyway, then FEAT-13.SPEC-004 re-checks eligibility at execution and proceeds since the booking is now Cancelled by Client, not upcoming.

**FEAT-13.SPEC-006-AC-14:** Given a new upcoming booking is created for a client between FEAT-13.SPEC-003's load and Talia's confirmation tap, when Talia confirms deletion anyway, then FEAT-13.SPEC-004 re-checks eligibility at execution, finds the new booking, and refuses the deletion, leaving all data unchanged.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 (all deferred to FEAT-13.SPEC-005) | 6 |
| Cross-Field Rules | 1 (N/A, cross-entity gate explained) | 1 |
| Authorization Rules | 4 | 4 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
