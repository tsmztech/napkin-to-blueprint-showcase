---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-21.SPEC-006
spec_name: Series & Occurrence Cancellation Rules
spec_slug: series-occurrence-cancellation-rules
parent_feature: FEAT-21
parent_feature_name: Recurring/Standing Appointments
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 4
acceptance_criteria_count: 16
---

# Logic/Rule Spec: Series & Occurrence Cancellation Rules

## Overview

**Name:** Series & Occurrence Cancellation Rules
**ID:** FEAT-21.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs what cancelling the whole series does to its not-yet-occurred occurrences versus cancelling a single occurrence, and how a concurrent Client/Pro change to the same series or occurrence resolves.
**Parent Feature:** FEAT-21 -- Recurring/Standing Appointments
**Governed Entity:** Recurring Series (and its generated occurrence Bookings)

## Scope and Non-Goals

**In Scope:**
- The state transition of a Recurring Series from Active to Ended on whole-series cancellation
- The cascade effect of whole-series cancellation on its not-yet-occurred generated occurrences
- The effect of cancelling a single occurrence, independent of its series
- Authorization for both cancel actions, per role
- The contention rule when the Client and the Pro act on the same series or occurrence at the same time

**Non-Goals:**
- The deposit refund-or-forfeit outcome of a cancelled occurrence -- owned by FEAT-09 (Cancellation & No-Show Policy Engine, XBR-09); this spec triggers that ordinary policy per occurrence, it does not define a separate rule for standing appointments.
- The screen-level presentation of the two cancel actions -- owned by FEAT-21.SPEC-002 (My Recurring Series) for the Client and FEAT-21.SPEC-010 (Pro Recurring Series Management) for the Pro, which enforce these rules but do not duplicate them.
- Interval and generation-limit validation -- owned by FEAT-21.SPEC-003; this spec begins only once a series already exists and is being cancelled, not created.
- A dedicated pause action as an alternative to cancellation -- excluded per this Brief's Non-Goals: no Key Capability, Primary Flow, Alternate, or States-field line describes a pause interaction, so this spec models only the Active-to-Ended transition, never a reversible pause.

## Governed Entity

**Entity:** Recurring Series (and its generated occurrence Bookings)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| state | enum (Active, Ended) | The series' lifecycle state; this spec owns the Active -> Ended transition |
| generated_occurrences | derived list (Booking references) | The occurrences cascaded by a whole-series cancellation |
| Booking.state (occurrence) | enum | The individual occurrence's state; this spec owns its transition to Cancelled by Client / Cancelled by Pro |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-21.SPEC-002 | My Recurring Series | On the "Cancel series" and "Cancel this one" actions, and on their confirmation dialogs |
| FEAT-21.SPEC-010 | Pro Recurring Series Management | On the Pro's "End series" and "Cancel this one" actions, and on their confirmation dialogs, for series tied to her own schedule |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| state | Must transition only Active -> Ended; no other transition is defined | On whole-series cancel | On cancel action | "This series has already been cancelled." (shown only if a second cancel attempt reaches an already-Ended series) | Yes |
| Booking.state (occurrence) | Must be a not-yet-occurred, not-already-cancelled occurrence to be eligible for cancellation | On occurrence cancel or cascade | On cancel action | "This appointment is no longer active." | Yes |
| generated_occurrences | No validation beyond data type -- this spec only reads generated_occurrences to drive the cascade defined under Cross-Field Rules | -- | -- | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Whole-series cascade | Recurring Series.state, Booking.state (each generated occurrence) | When the series transitions to Ended, every generated occurrence whose Booking has not yet occurred and is not already Cancelled or Completed is itself transitioned to Cancelled | N/A -- the cascade is automatic and silent to the acting party beyond the confirmation dialog already shown on FEAT-21.SPEC-002 |
| Already-completed occurrences untouched | Booking.state (each generated occurrence) | A whole-series cancellation never changes an occurrence whose Booking is already Completed or No-Show | N/A -- these occurrences are simply excluded from the cascade |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Cancel one occurrence | The Client (Riley) | Only her own series' occurrence, and only while that occurrence has not yet occurred and is not already Cancelled | The "Cancel this one" action is not shown for an occurrence that has already occurred, is already Cancelled, or belongs to a series that is not her own; a direct attempt shows "This appointment is no longer active." |
| Cancel one occurrence | The Pro (Talia) | Any occurrence in a series tied to her own schedule, not yet occurred and not already Cancelled, through FEAT-21.SPEC-010 (Pro Recurring Series Management) | -- |
| Cancel one occurrence | Platform Operator (Support) | Never | The action is not shown anywhere in Support's read-only view (FEAT-19) |
| Cancel whole series | The Client (Riley) | Only her own series, while it is Active | The "Cancel series" action is not shown for a series that is already Ended; a direct attempt shows "This series has already been cancelled." |
| Cancel whole series | The Pro (Talia) | Any series tied to her own schedule, while it is Active, through FEAT-21.SPEC-010 (Pro Recurring Series Management) | -- |
| Cancel whole series | Platform Operator (Support) | Never | The action is not shown anywhere in Support's read-only view (FEAT-19) |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| Series.state | Set to Ended | On whole-series cancel | No -- once cancelled, a series is never reactivated; a client who wants standing appointments again sets up a fresh series (FEAT-21.SPEC-001) |
| Booking.state (each cascaded occurrence) | Set to Cancelled by Client or Cancelled by Pro, matching whichever party's action triggered the whole-series cancellation | On whole-series cancel, for every not-yet-occurred, not-already-Cancelled occurrence | No |
| Booking.state (single-occurrence cancel) | Set to Cancelled by Client or Cancelled by Pro, matching the acting party | On single-occurrence cancel | No |

## Business Rules

- Whole-series cancellation is soft: the series transitions from Active to Ended, with no hard delete and no restore path (Entity-Lifecycle Coverage Matrix). A client who wants standing appointments again sets up a fresh series through FEAT-21.SPEC-001.
- Every cascaded or individually cancelled occurrence's deposit outcome follows FEAT-09's ordinary cancellation policy (XBR-09) exactly as any other booking's cancellation -- refunded outside the Pro's cancellation window, kept inside it or on a no-show; this spec never invents a separate, standing-appointment-specific deposit rule (scope-boundaries.md SC-18).
- Cancelling a single occurrence never changes the series' interval or state -- the series continues generating its future occurrences normally (this Brief's Side-Effect Inventory).
- Contention rule (dependency map, Recurring Series entity): when the Client and the Pro act on the same series or the same occurrence at effectively the same time, the resolution is reject-with-refresh -- the first committed change wins, and the other party's view refreshes to show the already-changed state before their own action completes, rather than either action being silently dropped or merged.
- An Ended series is kept indefinitely as history, consistent with scope-boundaries.md SC-22's retention of booking and financial history.

## Edge Cases

- **The Client cancels the whole series while an occurrence is mid-deposit-payment (client is on FEAT-07's payment screen for that occurrence)** -- Per the reject-with-refresh contention rule, whichever action commits first wins: if the whole-series cancellation commits first, the in-progress payment is refused with refresh and the client sees the booking is no longer active; if the payment captures first, that occurrence is Confirmed and is still cascaded to Cancelled by the series cancellation, with its deposit then following FEAT-09's ordinary refund rule for a Confirmed booking.
- **The Pro cancels the same series the Client is viewing on FEAT-21.SPEC-002 at the same time the Client taps "Cancel series"** -- The first commit wins; the other party's screen refreshes to the already-Ended state, per FEAT-21.SPEC-002's own edge case.
- **An occurrence reaches its unpaid cancellation cut-off (FEAT-21.SPEC-005) at the same moment the Client cancels it manually** -- Both actions arrive at the same outcome (the Booking becomes inactive); whichever transition commits first stands, and the other is a no-op against an already-inactive Booking -- no error is shown to the Client in either order, since the end state is the one she intended.
- **Cancelling the whole series when one occurrence is already Completed and another is Confirmed but not yet occurred** -- The Completed occurrence is untouched; the Confirmed, not-yet-occurred occurrence is cascaded to Cancelled and its deposit follows FEAT-09's ordinary rule for a Confirmed booking's cancellation.
- **A second whole-series cancellation attempt reaches an already-Ended series (e.g., a stale screen retried after the first cancellation already committed)** -- Rejected with "This series has already been cancelled." and no further cascade is attempted.
- **The Pro cancels one occurrence tied to her own schedule while the Client simultaneously attempts to cancel the same occurrence** -- The first commit wins, per the same reject-with-refresh contention rule; the other party sees the occurrence already cancelled (on FEAT-21.SPEC-002 for the Client, on FEAT-21.SPEC-010 for the Pro).

## Acceptance Criteria

**FEAT-21.SPEC-006-AC-01:** Given Riley cancels the whole series, when the cancellation commits, then the series transitions to Ended and every not-yet-occurred, not-already-Cancelled occurrence is cascaded to Cancelled by Client.

**FEAT-21.SPEC-006-AC-02:** Given a series being whole-cancelled has one Completed occurrence and one upcoming occurrence, when the cascade runs, then the Completed occurrence is untouched and only the upcoming one is cancelled.

**FEAT-21.SPEC-006-AC-03:** Given Riley cancels a single occurrence, when the cancellation commits, then only that occurrence's Booking is cancelled and the series' state and interval are unchanged.

**FEAT-21.SPEC-006-AC-04:** Given Talia cancels a series tied to her own schedule through FEAT-21.SPEC-010, when the cancellation commits, then the same Active-to-Ended transition and cascade occur, attributed to Cancelled by Pro.

**FEAT-21.SPEC-006-AC-05:** Given Platform Operator (Support) views a series, when they look for a cancel action, then none is shown anywhere in their read-only view.

**FEAT-21.SPEC-006-AC-06:** Given a cancelled occurrence's deposit was never captured, when FEAT-09's ordinary policy evaluates it, then no refund is due, since there is nothing to refund.

**FEAT-21.SPEC-006-AC-07:** Given a cancelled occurrence's deposit was already captured and the cancellation falls outside the Pro's cancellation window, when FEAT-09's ordinary policy evaluates it, then the client receives a full refund.

**FEAT-21.SPEC-006-AC-08:** Given a cancelled occurrence's deposit was already captured and the cancellation falls inside the Pro's cancellation window, when FEAT-09's ordinary policy evaluates it, then the deposit is kept.

**FEAT-21.SPEC-006-AC-09:** Given the Pro cancels the same series Riley is viewing at the same moment Riley taps "Cancel series," when both commit, then the first to commit wins and the other party's view refreshes to the already-Ended state.

**FEAT-21.SPEC-006-AC-10:** Given Riley cancels the whole series while an occurrence's deposit payment is in progress, when the series cancellation commits first, then the in-progress payment is refused with refresh.

**FEAT-21.SPEC-006-AC-11:** Given the same occurrence in-progress payment instead captures before the series cancellation commits, when the series cancellation then runs, then that now-Confirmed occurrence is still cascaded to Cancelled, and its deposit follows FEAT-09's Confirmed-booking refund rule.

**FEAT-21.SPEC-006-AC-12:** Given an occurrence reaches its unpaid cancellation cut-off (FEAT-21.SPEC-005) at the same moment Riley cancels it manually, when both transitions are evaluated, then whichever commits first stands and the other is a no-op, with no error shown to Riley.

**FEAT-21.SPEC-006-AC-13:** Given a series is already Ended, when a second whole-series cancellation attempt is made against it, then it is rejected with "This series has already been cancelled." and no further cascade runs.

**FEAT-21.SPEC-006-AC-14:** Given Talia cancels one occurrence tied to her own schedule at the same moment Riley attempts to cancel that same occurrence, when both commit, then the first to commit wins and the other sees the occurrence already cancelled.

**FEAT-21.SPEC-006-AC-15:** Given Riley cancels the whole series, when she reopens FEAT-21.SPEC-002 afterward, then the series and its occurrences no longer appear, consistent with the Ended state carrying no further active occurrences.

**FEAT-21.SPEC-006-AC-16:** Given Riley attempts to cancel an occurrence that has already occurred, when she looks for the "Cancel this one" action, then it is not shown for that occurrence.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
