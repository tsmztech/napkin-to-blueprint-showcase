---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-12.SPEC-006
spec_name: Booking Completion Rules
spec_slug: booking-completion-rules
parent_feature: FEAT-12
parent_feature_name: Pro Daily Schedule Dashboard
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 31
acceptance_criteria_count: 14
---

# Logic/Rule Spec: Booking Completion Rules

## Overview

**Name:** Booking Completion Rules
**ID:** FEAT-12.SPEC-006
**Type:** Logic/Rule
**Purpose:** Governs when a Booking may be marked Completed (by the Pro or automatically), and how completion interacts with the booking's remaining lifecycle actions.
**Parent Feature:** FEAT-12 -- Pro Daily Schedule Dashboard
**Governed Entity:** Booking (the `state` field's transition into `Completed`, and the conditions gating that transition)

## Scope and Non-Goals

**In Scope:**
- Eligibility conditions for marking a Booking Completed, whether Pro-initiated or automatic
- The timing window for automatic completion
- How a completed state interacts with cancellation, reschedule, and refund eligibility (XBR-12)
- Authorization for the mark-completed action, per role
- What happens when completion is attempted against a Booking in an ineligible state

**Non-Goals:**
- The mark-completed interaction's on-screen presentation -- owned by FEAT-12.SPEC-001 (Today's & Upcoming Schedule), which enforces this spec's rules
- The 7-day sweep's trigger scheduling and processing steps -- owned by FEAT-12.SPEC-004 (Auto-Completion Sweep), which enforces this spec's eligibility window
- Deriving the balance-due amount shown alongside a completed booking -- owned by FEAT-12.SPEC-007 (Balance Due & Status Display Rules)
- Taking an in-app balance payment at completion -- excluded per scope-boundaries.md SC-16: the balance is deliberately settled in person, off-platform, at MVP; this spec only records that the balance was settled in person, never a payment transaction
- No-show marking and its own 24-hour undo window -- owned by FEAT-11 (No-Show Marking & Deposit Forfeiture); this spec only notes that a No-Show state closes off completion eligibility

## Governed Entity

**Entity:** Booking
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| service | text (reference) | The Service this booking is for; fixed at booking |
| start_time | date/time | Appointment start, in the Pro's timezone; fixed at booking |
| duration | number | Appointment length in minutes; fixed at booking |
| client | reference | The Client this booking belongs to |
| price_agreed | number | Price agreed at booking time |
| deposit_amount | number | Deposit agreed at booking time |
| policy_version | reference | Cancellation policy version shown and acknowledged at booking |
| state | enum | Pending Payment \| Confirmed \| Awaiting Outcome \| Completed \| No-Show \| Cancelled by Client \| Cancelled by Pro \| Rescheduled \| Expired (unpaid) |
| attendance_reply | enum | "I'll be there" / reschedule requested, from reminders |
| balance_due | derived | price_agreed − deposit_amount − any in-app balance payment |
| source | enum | Client link, Pro booked-in, recurring occurrence |
| cancellation / reschedule timestamps | date/time (+ optional text) | Timestamps and an optional private Pro reason for a cancellation or reschedule |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-12.SPEC-001 | Today's & Upcoming Schedule | On the "mark completed" quick action -- checked immediately when the Pro taps the action, before the write is attempted |
| FEAT-12.SPEC-004 | Auto-Completion Sweep | On each scheduled sweep pass -- checked for every Booking still in Confirmed or Awaiting Outcome state |
| FEAT-30 (Pro Booking Management) | Pro-initiated cancel/reschedule | Consulted (not owned here) to determine whether a Booking's Completed state closes off cancel/reschedule eligibility |
| FEAT-11 (No-Show Marking & Deposit Forfeiture) | No-show marking | Consulted (not owned here) to determine whether a Booking already marked Completed is ineligible for no-show marking |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| state | May transition to `Completed` only from `Confirmed` or `Awaiting Outcome`, and only once `start_time` has passed | Always, for both the Pro-initiated and automatic paths | On mark-completed attempt (screen) and on each sweep pass (automation) | "This appointment can't be marked completed yet -- it hasn't started." (before start_time) / "This booking can no longer be marked completed." (state is already Completed, No-Show, Cancelled, Rescheduled, Expired, or Pending Payment) | Yes |
| start_time | No validation beyond data type -- read as the eligibility gate for completion; the field itself is set and validated at booking time by FEAT-05, FEAT-21, or FEAT-30 | -- | -- | -- | -- |
| duration | No validation beyond data type -- not governed by this spec; owned by FEAT-05/FEAT-01 | Always | -- | -- | -- |
| service | No validation beyond data type -- not governed by this spec; owned by FEAT-01/FEAT-05 | Always | -- | -- | -- |
| client | No validation beyond data type -- not governed by this spec; owned by FEAT-05/FEAT-30 | Always | -- | -- | -- |
| price_agreed | No validation beyond data type -- not governed by this spec; owned by FEAT-07 | Always | -- | -- | -- |
| deposit_amount | No validation beyond data type -- not governed by this spec; owned by FEAT-07 | Always | -- | -- | -- |
| policy_version | No validation beyond data type -- not governed by this spec; owned by FEAT-09 | Always | -- | -- | -- |
| attendance_reply | No validation beyond data type -- not governed by this spec; owned by FEAT-08 | Always | -- | -- | -- |
| balance_due | No validation beyond data type -- derivation owned by FEAT-12.SPEC-007; this spec only consumes the state transition that causes the balance to be recorded as settled in person | On completion, the settled-in-person status is recorded (see Business Rules) | On mark-completed (screen) and on sweep completion (automation) | -- | -- |
| source | No validation beyond data type -- not governed by this spec; owned by FEAT-05/FEAT-30/FEAT-21 | Always | -- | -- | -- |
| cancellation / reschedule timestamps | No validation beyond data type -- not governed by this spec; owned by FEAT-10/FEAT-30 | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Completion eligibility window | state, start_time | `state` may become `Completed` only when the current time is strictly after `start_time` AND `state` is currently `Confirmed` or `Awaiting Outcome` | "This appointment can't be marked completed yet -- it hasn't started." |
| Auto-completion window | state, start_time | If `state` remains `Confirmed` or `Awaiting Outcome` 7 days (platform parameter: `booking-auto-completion-window-days`) after `start_time` with no Pro action, the Auto-Completion Sweep (FEAT-12.SPEC-004) transitions `state` to `Completed` automatically | N/A -- automatic transition, no user-facing error |
| Completion closes cancel/reschedule/no-show eligibility | state | Once `state` is `Completed`, FEAT-30's cancel/reschedule actions and FEAT-11's no-show action are no longer available for this Booking (XBR-12) | Owned by FEAT-30/FEAT-11: "This booking is already completed and can no longer be changed." |
| Completion closes goodwill-refund eligibility | state | A goodwill refund (owned by FEAT-30) remains available only until `state` becomes `Completed` (XBR-12) | Owned by FEAT-30: goodwill refund action is not offered once the booking is Completed |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Mark a booking Completed (manual) | The Pro | Only bookings belonging to the requesting Pro's own account, in `Confirmed` or `Awaiting Outcome` state, with `start_time` in the past | Mark-completed control is not shown on a booking outside this Pro's account (never reachable -- FEAT-12.SPEC-008 scopes the dashboard to the Pro's own schedule); when shown but the eligibility condition fails, the control is disabled and tapping it (e.g. via a stale screen) shows "This appointment can't be marked completed yet -- it hasn't started." or "This booking can no longer be marked completed." depending on which condition failed |
| Mark a booking Completed (manual) | Platform Operator (Support) | Never | Mark-completed control is not shown to Support; Support's view is read-only per SC-05 |
| Mark a booking Completed (manual) | The Client | Never | The Client has no access to this dashboard at all (per FEAT-12.SPEC-008); the action is never reachable |
| Trigger the automatic 7-day completion sweep | No human role -- the sweep (FEAT-12.SPEC-004) runs on a schedule with no user-initiated trigger | Always, subject to the Cross-Field Rules eligibility window | N/A -- there is no denial path for a system-triggered action |
| View a booking's completion state | The Pro | Own bookings only | -- |
| View a booking's completion state | Platform Operator (Support) | The one Pro account under active support review, read-only (XBR-24); never the Pro's private client note | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| state (on completion) | Set to `Completed` | When either the Pro's mark-completed action or the Auto-Completion Sweep (FEAT-12.SPEC-004) succeeds | No -- the Pro chooses whether to act before the automatic window closes, but cannot set an arbitrary state directly |
| balance settled-in-person flag | Implicitly recorded the moment `state` becomes `Completed` (whether Pro-marked or auto-completed), meaning the balance shown by FEAT-12.SPEC-007 is understood to have been collected off-platform | On completion (either path) | No -- this is not a separate payment record at MVP (SC-16); FEAT-22 (In-App Balance Payment, v1) is the only path that would record an actual balance payment |

## Business Rules

- A Booking can be marked Completed only after its `start_time` has passed; there is no requirement that the full `duration` has elapsed (XBR-12).
- If the Pro takes no action, the Auto-Completion Sweep (FEAT-12.SPEC-004) transitions the Booking to `Completed` automatically 7 days (platform parameter: `booking-auto-completion-window-days`) after `start_time` (XBR-12).
- Marking a Booking Completed records the balance as settled in person; it never triggers an in-app balance charge (SC-16). Once FEAT-22 (In-App Balance Payment) exists, FEAT-12.SPEC-007 will read from it, but this spec's completion mechanics do not change.
- A Booking already in `Completed`, `No-Show`, `Cancelled by Client`, `Cancelled by Pro`, `Rescheduled`, `Expired (unpaid)`, or `Pending Payment` cannot be marked Completed again or for the first time by either path; the completion attempt is refused.
- A `Completed` or `No-Show` Booking can no longer be cancelled or rescheduled (XBR-12) -- FEAT-30 and FEAT-10 enforce this on their own actions by reading `state`; this spec is the source of truth for when `state` reaches `Completed`.
- A goodwill refund on the Booking's deposit (owned by FEAT-30) remains available only until the Booking reaches `Completed` (XBR-12).
- Contention resolution follows the dependency map's Booking entity note: reject-with-refresh -- the first committed state transition wins, and any other actor (the Pro on a second device, the Auto-Completion Sweep, or a client-initiated cancellation) sees the Booking's current state and must re-decide rather than having transitions merged.
- This spec does not govern no-show marking's own 24-hour undo window (FEAT-11) or cancellation/reschedule eligibility windows (FEAT-09, FEAT-10); it governs only the point at which those windows close because the Booking has become Completed.
- Every transition of `state` into `Completed`, by either path, is a booking-outcome event that FEAT-25.SPEC-004 (Insights Aggregates) consumes to update its insights aggregates; FEAT-25.SPEC-004 reads the resulting state and never writes it, so this spec's eligibility rules are unaffected.
- Completion is a one-way transition: the Access Matrix and product definition provide no undo action for a Completed booking (unlike No-Show, which FEAT-11 allows undoing for a limited window).

## Edge Cases

- **Pro taps "mark completed" at the exact instant of start_time** -- The condition requires the current time to be strictly after `start_time`; at the exact instant, the action is refused with "This appointment can't be marked completed yet -- it hasn't started." A retry a moment later succeeds.
- **Pro attempts to mark completed a Booking that the Auto-Completion Sweep already completed moments earlier** -- The Pro's screen reflects the current `Completed` state on next load or refresh (per the dependency map's reject-with-refresh resolution); the mark-completed control is no longer shown, since the Booking is already Completed.
- **Client cancels their own Booking (via FEAT-10) at the same moment the Pro taps mark-completed** -- First committed transition wins. If the cancellation commits first, the Pro's mark-completed attempt is refused with "This booking can no longer be marked completed." and the Pro's screen refreshes to show the Cancelled state. If completion commits first, the Client's cancellation attempt is refused by FEAT-10 with its own current-state message.
- **Auto-Completion Sweep runs while the Pro is actively viewing the booking on the dashboard** -- The dashboard reflects the new `Completed` state on its next refresh (FEAT-12.SPEC-001); no destructive action is taken against any in-flight Pro interaction, since the sweep only ever moves a Booking forward from `Confirmed`/`Awaiting Outcome` to `Completed`.
- **Booking is in `Awaiting Outcome` exactly at the 7-day boundary** -- The sweep evaluates at each scheduled pass (FEAT-12.SPEC-004); a Booking that crosses the boundary between passes is completed on the next pass that finds it still eligible, not the instant the boundary is crossed.
- **A Booking that was never confirmed (still `Pending Payment` or already `Expired (unpaid)`) reaches its 7-day mark** -- Never eligible for completion by either path, since eligibility requires `Confirmed` or `Awaiting Outcome` as the starting state; it remains in its own terminal state.

## Acceptance Criteria

**FEAT-12.SPEC-006-AC-01:** Given Talia has a Confirmed booking whose start_time has passed, when she taps "mark completed" on FEAT-12.SPEC-001, then the booking's state transitions to Completed and the balance is recorded as settled in person.

**FEAT-12.SPEC-006-AC-02:** Given Talia has a Confirmed booking whose start_time has not yet arrived, when she attempts to mark it completed, then the action is refused with "This appointment can't be marked completed yet -- it hasn't started." and the state does not change.

**FEAT-12.SPEC-006-AC-03:** Given Talia has a booking already in Completed state, when she looks at it on the dashboard, then no mark-completed control is shown for it.

**FEAT-12.SPEC-006-AC-04:** Given Talia has a booking already marked No-Show, when she attempts to mark it completed, then the action is refused with "This booking can no longer be marked completed."

**FEAT-12.SPEC-006-AC-05:** Given a Confirmed booking's start_time passed 7 days ago and Talia never acted on it, when the Auto-Completion Sweep (FEAT-12.SPEC-004) runs, then the booking's state transitions to Completed automatically.

**FEAT-12.SPEC-006-AC-06:** Given a Confirmed booking whose start_time passed only 2 days ago, when the Auto-Completion Sweep runs, then the booking is left unchanged because it has not yet reached the 7-day window.

**FEAT-12.SPEC-006-AC-07:** Given a booking has been marked Completed, when Talia opens FEAT-30 (Pro Booking Management) for that booking, then no cancel or reschedule action is available for it, per XBR-12.

**FEAT-12.SPEC-006-AC-08:** Given a booking has been marked Completed, when Talia looks for a goodwill refund option on that booking, then none is offered, per XBR-12.

**FEAT-12.SPEC-006-AC-09:** Given Talia (the Pro) is viewing her own dashboard, when she marks an eligible booking completed, then the action succeeds, since only the Pro may perform this action on her own bookings.

**FEAT-12.SPEC-006-AC-10:** Given Platform Operator (Support) is viewing a Pro's account during a support session, when Support looks for a mark-completed control, then none is shown, since Support's access is read-only (SC-05).

**FEAT-12.SPEC-006-AC-11:** Given a Client attempts to reach this dashboard, when the access check runs, then the mark-completed action is never reachable, because Clients have no access to this dashboard at all (FEAT-12.SPEC-008).

**FEAT-12.SPEC-006-AC-12:** Given Talia's booking is simultaneously cancelled by the Client (FEAT-10) and marked completed by Talia at effectively the same moment, when the cancellation commits first, then Talia's mark-completed attempt is refused with "This booking can no longer be marked completed." and her dashboard refreshes to show the cancelled state.

**FEAT-12.SPEC-006-AC-13:** Given a booking's start_time is exactly the current instant, when Talia attempts to mark it completed at that exact moment, then the action is refused because the transition requires the current time to be strictly after start_time.

**FEAT-12.SPEC-006-AC-14:** Given a booking is still in Pending Payment state, when the Auto-Completion Sweep evaluates it after the 7-day window, then the booking is left unchanged because it never reached Confirmed or Awaiting Outcome.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 12 | 12 |
| Cross-Field Rules | 4 | 4 |
| Authorization Rules | 6 | 6 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 9 | 9 |
| Edge Cases | 6 | 6 |
