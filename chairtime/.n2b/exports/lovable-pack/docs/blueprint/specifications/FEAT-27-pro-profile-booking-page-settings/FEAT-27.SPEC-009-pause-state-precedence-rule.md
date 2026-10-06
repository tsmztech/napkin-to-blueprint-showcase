---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-27.SPEC-009
spec_name: Pause State Precedence Rule
spec_slug: pause-state-precedence-rule
parent_feature: FEAT-27
parent_feature_name: Pro Profile & Booking Page Settings
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 5
acceptance_criteria_count: 11
---

# Logic/Rule Spec: Pause State Precedence Rule

## Overview

**Name:** Pause State Precedence Rule
**ID:** FEAT-27.SPEC-009
**Type:** Logic/Rule
**Purpose:** Governs how a Pro-chosen pause and a system-imposed (subscription-lapse) pause coexist, including that the Pro's resume toggle cannot clear a system-imposed pause, and that a pause end date cannot be in the past.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings
**Governed Entity:** Pro Account -- the status field (pause sub-state), pause message, and pause end date

## Scope and Non-Goals

**In Scope:**
- The precedence logic between a Pro-chosen pause and a system-imposed pause
- Validation of the pause message and end date fields
- Authorization for who may set or clear each pause source

**Non-Goals:**
- Setting or lifting the system-imposed pause itself -- owned entirely by FEAT-18 (Pro Subscription Billing & Account Management, FEAT-18.SPEC-004); this spec only defines how that source interacts with the Pro-chosen one
- The Pause Bookings screen's layout and controls -- owned by FEAT-27.SPEC-004 (Pause Bookings); this spec only supplies the rules it enforces
- Automatically resuming the Pro-chosen pause when its end date arrives -- owned by FEAT-27.SPEC-011 (Automatic Pause Resume); this spec only defines what "resumed" means when a system-imposed pause may still be present

## Governed Entity

**Entity:** Pro Account (status field and its pause-related sub-fields)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| status | enum (Active, Paused) with independent pause-source flags | Whether the account is currently taking new bookings; Paused may result from the Pro's own choice, a system-imposed subscription lapse, or both simultaneously |
| pause_message | text | Optional message shown on the booking page in place of available times, set only for the Pro-chosen pause source |
| pause_end_date | date | Optional date on which the Pro-chosen pause automatically resumes; the system-imposed pause carries no end date of its own -- it lifts only when FEAT-18 restores billing |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-27.SPEC-004 | Pause Bookings | On screen load (to render the correct status banner) and on submit (toggle, message, end-date changes) |
| FEAT-27.SPEC-011 | Automatic Pause Resume | On the chosen end date, to determine whether resuming the Pro-chosen pause fully resumes bookings or leaves the account paused |
| FEAT-05.SPEC-008 | Booking Page Availability Gate | Reads the combined pause state to decide what a client sees |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| pause_message | Optional, no length restriction beyond data type -- "no validation beyond data type" | Always | -- | -- | -- |
| pause_end_date | Cannot be in the past | Always, when provided | On selection and on submit | "The resume date can't be in the past. Choose today or a later date." | Yes |
| pause_end_date | Optional -- a pause with no end date requires a manual resume | Always | On submit | -- | No |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Message and end date only apply while the Pro-chosen pause is on | pause toggle, pause_message, pause_end_date | If the Pro-chosen pause toggle is off, pause_message and pause_end_date are cleared and hold no effect (they are re-enterable the next time the toggle is turned on) | N/A -- not an error, a state-dependent clearing rule |
| Precedence combination rule | Pro-chosen pause state, system-imposed pause state | The account's overall status is Paused if either source is active; it is Active only when both are inactive. The two sources are tracked and can be cleared independently. | N/A -- this is the core precedence derivation, stated in Business Rules below |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Turn on the Pro-chosen pause (with optional message and end date) | The Pro (Talia) | Always, on her own account | -- |
| Turn off the Pro-chosen pause ("resume bookings") | The Pro (Talia) | Always accepted for the Pro-chosen pause specifically; never clears a system-imposed pause | If a system-imposed pause remains active after the toggle clears the Pro-chosen one, the screen states plainly: "Your bookings stay paused until your billing issue is resolved." -- never implying a full resume occurred |
| Set or lift the system-imposed pause | FEAT-18 (Pro Subscription Billing & Account Management), acting automatically on the Pro's behalf | Set on subscription lapse past the grace period; lifted on billing restoration | Not applicable to the Pro directly -- the Pro's only lever over this source is resolving billing through FEAT-18, which this spec's screen (FEAT-27.SPEC-004) links to |
| Turn the pause on/off | The Client (Riley) | Never | No control of any kind is exposed to clients |
| Turn the pause on/off | Platform Operator (Support) | Never | The pause toggle and related fields are shown disabled and labeled "View-only in support mode" on FEAT-27.SPEC-004 |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| status (overall pause state, derived) | Paused if either the Pro-chosen pause flag or the system-imposed pause flag is true; Active only when both are false | Evaluated live on every read (screen load, booking-page gate check) | No -- this is a system derivation from the two independent source flags |
| pause_message, pause_end_date | Empty/absent by default | On Pro Account creation | Yes -- set only when the Pro turns on the Pro-chosen pause |

## Business Rules

- XBR-14: a paused account (from either source) takes no new bookings or deposits, while existing bookings keep their reminders, refunds, and client self-service unchanged.
- The Pro's "resume bookings" toggle can only ever clear the Pro-chosen pause source -- it has no effect on a system-imposed pause, which lifts only when FEAT-18 confirms billing is restored (FEAT-18.SPEC-004).
- Reaching a Pro-chosen pause's end date (via FEAT-27.SPEC-011) clears the Pro-chosen source the same way the manual toggle does: if a system-imposed pause remains, the account stays overall Paused.
- A pause_end_date cannot be in the past at the moment it is set or re-saved -- this protects against a pause that would appear to have already silently expired.

## Edge Cases

- **Talia turns off the Pro-chosen pause while a system-imposed pause is also active** -- The Pro-chosen source clears immediately; the overall status stays Paused because the system-imposed source remains, and the screen states this outcome explicitly rather than implying a full resume.
- **A system-imposed pause is added by FEAT-18 while a Pro-chosen pause with an end date is already counting down** -- Both sources are now active; when the Pro-chosen pause's end date is reached, FEAT-27.SPEC-011 clears it, but the overall status stays Paused until FEAT-18 separately lifts the system-imposed source.
- **Talia sets a pause_end_date of today** -- Passes validation (today is not "in the past"); FEAT-27.SPEC-011 evaluates the resume at the end of that same day.
- **Talia sets a pause with no end date and no message** -- Valid; the pause continues until she manually turns it off (subject to the precedence rule above if a system-imposed pause coexists).
- **The system-imposed pause is lifted by FEAT-18 while the Pro-chosen pause remains active** -- The overall status stays Paused, now solely from the Pro-chosen source, and the screen's status banner updates from "Paused by you and by billing" to "Paused by you."
- **Both pause sources clear at effectively the same moment (Talia turns off her pause the instant FEAT-18 restores billing)** -- Each source's clearing is independent and idempotent; the overall status becomes Active only once both underlying flags are confirmed false, with no race condition since each source only ever clears its own flag.

## Acceptance Criteria

**FEAT-27.SPEC-009-AC-01:** Given Talia's account has only a Pro-chosen pause active, when she turns the toggle off, then the overall status becomes Active ("Taking bookings").

**FEAT-27.SPEC-009-AC-02:** Given Talia's account has both a Pro-chosen pause and a system-imposed pause active, when she turns the toggle off, then the Pro-chosen source clears but the overall status stays Paused, and she sees "Your bookings stay paused until your billing issue is resolved."

**FEAT-27.SPEC-009-AC-03:** Given Talia's account has only a system-imposed pause active, when she views the pause screen, then the Pro-chosen toggle is off and no resume action is offered for the system-imposed source -- only a link to resolve billing.

**FEAT-27.SPEC-009-AC-04:** Given Talia sets a pause_end_date in the past, when she attempts to save, then she sees "The resume date can't be in the past. Choose today or a later date." and the save does not proceed.

**FEAT-27.SPEC-009-AC-05:** Given Talia sets a pause_end_date of today, when validation runs, then it passes.

**FEAT-27.SPEC-009-AC-06:** Given a Pro-chosen pause's end date is reached while a system-imposed pause remains active, when FEAT-27.SPEC-011 fires, then the Pro-chosen source clears but the overall status stays Paused.

**FEAT-27.SPEC-009-AC-07:** Given FEAT-18 lifts a system-imposed pause while a Pro-chosen pause remains active, when the lift is confirmed, then the overall status stays Paused, now solely from the Pro-chosen source.

**FEAT-27.SPEC-009-AC-08:** Given Talia turns off the Pro-chosen pause toggle and FEAT-18 restores billing at effectively the same moment, when both clearings are confirmed, then the overall status becomes Active with no inconsistent intermediate state.

**FEAT-27.SPEC-009-AC-09:** Given Talia turns the Pro-chosen pause toggle off, when she next turns it back on, then pause_message and pause_end_date are re-enterable (their prior values were cleared when the toggle turned off).

**FEAT-27.SPEC-009-AC-10:** Given a support operator views the pause screen, when they look for a way to change either pause source, then no actionable control is available -- the toggle and fields are disabled and labeled "View-only in support mode."

**FEAT-27.SPEC-009-AC-11:** Given a client visits the booking page while the account is Paused from either or both sources, when FEAT-05.SPEC-008 evaluates the gate, then the client sees the pause experience (message or default) regardless of which source or combination caused it.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 3 | 3 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
