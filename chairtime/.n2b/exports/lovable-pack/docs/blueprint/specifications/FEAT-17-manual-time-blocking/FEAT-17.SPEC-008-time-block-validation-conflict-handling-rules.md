---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-17.SPEC-008
spec_name: Time Block Validation & Conflict Handling Rules
spec_slug: time-block-validation-conflict-handling-rules
parent_feature: FEAT-17
parent_feature_name: Manual Time Blocking
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 14
acceptance_criteria_count: 20
---

# Logic/Rule Spec: Time Block Validation & Conflict Handling Rules

## Overview

**Name:** Time Block Validation & Conflict Handling Rules
**ID:** FEAT-17.SPEC-008
**Type:** Logic/Rule
**Purpose:** Defines the shared rules every screen and automation in this feature references: end-after-start validation, what counts as a conflicting booking, the never-silently-affect-a-booking rule, and who can see or act on a block.
**Parent Feature:** FEAT-17 -- Manual Time Blocking
**Governed Entity:** Time Block

## Scope and Non-Goals

**In Scope:**
- Field-level validation rules for every Time Block field
- The end-after-start cross-field rule
- The definition of what counts as a conflicting confirmed booking
- Authorization rules for every action on a Time Block, per role
- Default values and derived fields (state, resolution tracking)
- The never-silently-affect-a-booking rule governing every conflict resolution outcome

**Non-Goals:**
- The processing logic that detects conflicts at save time -- owned by FEAT-17.SPEC-004 and FEAT-17.SPEC-005, which apply the definitions in this spec
- The processing logic that commits a resolution -- owned by FEAT-17.SPEC-006, which applies the never-silently-affect-a-booking rule defined here
- UI layout and interaction behavior for displaying validation errors -- defined in FEAT-17.SPEC-001, FEAT-17.SPEC-002, and FEAT-17.SPEC-003, which reference this spec for the rules but own their own display behavior

## Governed Entity

**Entity:** Time Block
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| start | date/time | The block's start, in the Pro's account timezone |
| end | date/time | The block's end, in the Pro's account timezone; must be after start |
| recurrence | enum (none \| weekly-on-day) | Optional pattern; when set, defines the day-of-week this block repeats on |
| label | text | Optional, private to the Pro; describes the reason for the block |
| state | derived enum (Active \| Expired) | Whether this block (or occurrence) still blocks availability |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-17.SPEC-001 | Create/Edit Time Block | Field validation on blur (Date, Start time, End time, Label) and on form submit |
| FEAT-17.SPEC-002 | Manage Time Blocks | Authorization on screen entry (Support's view-only rendering); no field validation (no form fields) |
| FEAT-17.SPEC-003 | Time Block Conflict Review | The never-silently-affect-a-booking rule, enforced by disabling Confirm until every conflicting booking has a choice |
| FEAT-17.SPEC-004 | Time Block Save Commit & Conflict Detection | Re-validation of start/end/label at save time; the conflicting-booking definition, applied to the save's conflict scope |
| FEAT-17.SPEC-005 | Recurring Time Block Occurrence Generation | The same field validation and conflicting-booking definition, applied to each generated occurrence |
| FEAT-17.SPEC-006 | Time Block Conflict Resolution Commit | The never-silently-affect-a-booking rule, enforced by never altering a Booking field on a kept-exception outcome |
| FEAT-17.SPEC-007 | Time Block Removal & Expiry | The state derivation (Active -> Expired) that this automation acts on |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| start | Required; a valid date and time | Always | On blur, on submit | "Choose a start date and time" | Yes |
| end | Required; a valid date and time; must be strictly after start | Always | On blur, on submit | "End time must be after the start time" | Yes |
| recurrence | Optional; when set, must name exactly one day of the week | When the Repeats toggle is on | On selection, on submit | "Choose which day this repeats on" | Yes |
| label | Optional; no validation beyond a maximum of 100 characters | Always | On blur, on submit | "Label must be 100 characters or fewer" | Yes |
| state | Not user-entered -- system-derived only; no input validation applies | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| End-after-start | start, end | end must be strictly later than start (equal values are invalid; a zero-length block is not permitted) | "End time must be after the start time" |
| Recurrence requires a day | recurrence, start | If the Repeats toggle is on, a day-of-week must be selected; it defaults to the day-of-week of the entered start date but Talia may change it | "Choose which day this repeats on" |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create Time Block | The Pro (Talia) | Always, for her own Pro Account | -- |
| Create Time Block | The Client (Riley) | Never | No navigation path to the create screen exists in Riley's experience (FEAT-17.SPEC-001) |
| Create Time Block | Platform Operator (Support) | Never | The create/edit screen is outside support's granted session scope; no control or route to it is ever rendered (FEAT-17.SPEC-001) |
| View Time Block (single or list) | The Pro (Talia) | Always, for her own blocks only | -- |
| View Time Block (single or list), including label | Platform Operator (Support) | Always, for the one Pro account under active review, read-only | -- |
| View Time Block | The Client (Riley) | Never | Riley never sees a block directly -- only its absence from the bookable slot list on the public booking page |
| Edit Time Block (span, recurrence, label) | The Pro (Talia) | Always, for her own blocks only | -- |
| Edit Time Block | Platform Operator (Support) | Never | No edit control is rendered for Support anywhere in this feature |
| Edit Time Block | The Client (Riley) | Never | No navigation path exists in Riley's experience |
| Remove Time Block | The Pro (Talia) | Always, for her own blocks only | -- |
| Remove Time Block | Platform Operator (Support) | Never | No remove control is rendered for Support (FEAT-17.SPEC-002) |
| Remove Time Block | The Client (Riley) | Never | No navigation path exists in Riley's experience |
| Choose a conflict resolution outcome (cancel / reschedule / keep as exception) | The Pro (Talia) | Always, for conflicts against her own blocks | -- |
| Choose a conflict resolution outcome | Platform Operator (Support) | Never | FEAT-17.SPEC-003 is reachable only inside Talia's own active save flow; support never opens it |
| Choose a conflict resolution outcome | The Client (Riley) | Never | Riley is never shown which of her bookings conflicted, or offered a choice; she only receives the eventual notice matching Talia's chosen outcome |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| state | Active | On create (single-date block or each generated occurrence) | No -- system-derived only |
| state | Expired | The moment the block's own end value passes (FEAT-17.SPEC-007) | No -- system-derived only |
| recurrence's day-of-week (initial default) | The day-of-week of the entered start date | When the Repeats toggle is first turned on | Yes -- Talia may pick a different day before saving |

## Business Rules

- **What counts as a conflicting booking:** A Booking whose state is Confirmed or Awaiting Outcome, belonging to the same Pro Account, whose scheduled range (start_time through start_time + duration) overlaps the Time Block's [start, end) range. A Booking that is Pending Payment, Completed, No-Show, Cancelled by Client, Cancelled by Pro, Rescheduled, or Expired (unpaid) never counts as a conflict.
- **Overlap boundary:** Two ranges overlap only when they share at least one instant of time; a block ending exactly at a booking's start (or a booking ending exactly at a block's start) is adjacent, not overlapping, and is not a conflict.
- **The never-silently-affect-a-booking rule:** A confirming Booking flagged as conflicting is never automatically cancelled, rescheduled, or otherwise altered by this feature. Only Talia's explicit, per-booking choice on FEAT-17.SPEC-003 determines its outcome, and until every conflicting booking has a chosen outcome, the conflict remains open and visible (XBR-11).
- **A block never commits over an unresolved conflict silently** -- FEAT-17.SPEC-004 and FEAT-17.SPEC-005 always route a conflicting save or occurrence to FEAT-17.SPEC-003 rather than committing it outright; the block commits only once Talia's choices are captured (FEAT-17.SPEC-006).
- **A "kept as exception" booking's fields are never modified** -- only a resolution-tracking flag is set, and the exception is surfaced to Talia via FEAT-12.SPEC-005 (XBR-11).
- **First-committed-wins against an in-progress client checkout** -- when a block save and a client's checkout hold contend for the same instant, FEAT-03.SPEC-005's contention rule (not this spec) governs which one wins; this spec's conflicting-booking definition applies only to already-confirmed bookings.
- **Recurrence introduces no separate conflict rule** -- every generated occurrence (FEAT-17.SPEC-005) is checked against the exact same conflicting-booking definition as a single-date block.

## Edge Cases

- **start and end are identical** -- Invalid; a zero-length block fails the end-after-start rule ("End time must be after the start time").
- **end is exactly one minute after start** -- Valid; the minimum block length is not otherwise restricted.
- **label at exactly 100 characters** -- Passes validation. 101 characters shows the length error.
- **label left empty** -- Valid; a block with no label is fully supported and displays with only its time range.
- **A block's range touches a confirmed booking's boundary exactly (block end == booking start, or booking end == block start)** -- Not a conflict, per the overlap boundary rule; the ranges are adjacent, not overlapping.
- **A block's range overlaps a Booking by a single minute** -- Counted as a conflict; any nonzero overlap qualifies, there is no minimum-overlap threshold.
- **Recurrence is toggled on and then off again before saving** -- The day-of-week selection is discarded; the block saves as a plain single-date block with no recurrence.
- **Ownership boundary: a block's parent Pro Account is somehow different from the currently signed-in Pro** -- Not a reachable state in this single-operator product (scope-boundaries.md SC-01): every Time Block belongs to exactly one Pro Account, and the signed-in Pro can only ever act on her own.
- **Talia's own conflict choice arrives after the conflicting booking has already changed state through an unrelated path** -- FEAT-17.SPEC-006's re-validation drops that booking from the set before applying any hand-off, per the conflicting-booking definition no longer matching its current state.

## Acceptance Criteria

**FEAT-17.SPEC-008-AC-01:** Given Talia leaves the start field empty on FEAT-17.SPEC-001, when she blurs the field, then she sees "Choose a start date and time."

**FEAT-17.SPEC-008-AC-02:** Given Talia sets an end time equal to the start time, when validation runs, then she sees "End time must be after the start time" and the save is blocked.

**FEAT-17.SPEC-008-AC-03:** Given Talia sets an end time one minute after the start time, when validation runs, then it passes with no error.

**FEAT-17.SPEC-008-AC-04:** Given Talia turns on the Repeats toggle without selecting a day, when she attempts to submit, then she sees "Choose which day this repeats on."

**FEAT-17.SPEC-008-AC-05:** Given Talia turns on the Repeats toggle, when the day-of-week selector appears, then it defaults to the day-of-week of her entered start date.

**FEAT-17.SPEC-008-AC-06:** Given Talia enters a label of exactly 100 characters, when she blurs the field, then no error is shown.

**FEAT-17.SPEC-008-AC-07:** Given Talia enters a label of 101 characters, when she blurs the field, then she sees "Label must be 100 characters or fewer."

**FEAT-17.SPEC-008-AC-08:** Given Talia leaves the label empty and saves, when the block is created, then it is valid with no label.

**FEAT-17.SPEC-008-AC-09:** Given a block's range ends exactly at a confirmed booking's start time, when FEAT-17.SPEC-004 checks for conflicts, then it is not treated as a conflict.

**FEAT-17.SPEC-008-AC-10:** Given a block's range overlaps a confirmed booking by one minute, when FEAT-17.SPEC-004 checks for conflicts, then it is treated as a conflict.

**FEAT-17.SPEC-008-AC-11:** Given a Booking in the block's range is Pending Payment, when the conflict check runs, then it does not count as a conflict.

**FEAT-17.SPEC-008-AC-12:** Given a Booking in the block's range is Cancelled by Client, when the conflict check runs, then it does not count as a conflict.

**FEAT-17.SPEC-008-AC-13:** Given Talia (the Pro) attempts to create, edit, remove, or resolve a conflict on her own block, when she acts, then every action is allowed with no restriction.

**FEAT-17.SPEC-008-AC-14:** Given Platform Operator Support views the Manage Time Blocks list, when the screen renders, then Support sees every block including labels, with no create, edit, remove, or conflict-resolution control anywhere.

**FEAT-17.SPEC-008-AC-15:** Given the Client (Riley) has no navigation path to any FEAT-17 screen, when this is verified, then no create, view, edit, remove, or conflict-resolution capability is ever exposed to her.

**FEAT-17.SPEC-008-AC-16:** Given a conflicting booking exists, when Talia has not yet made a choice for it on FEAT-17.SPEC-003, then the booking is never automatically cancelled, rescheduled, or altered.

**FEAT-17.SPEC-008-AC-17:** Given Talia chooses "Keep as exception" for a conflicting booking, when FEAT-17.SPEC-006 commits, then no field on that Booking record changes.

**FEAT-17.SPEC-008-AC-18:** Given a single-date block commits with zero conflicts, when it is created, then its state is set to Active by default.

**FEAT-17.SPEC-008-AC-19:** Given a block's end value passes, when FEAT-17.SPEC-007's scheduled expiry check runs, then its state derivation moves to Expired and Talia cannot override this transition.

**FEAT-17.SPEC-008-AC-20:** Given a recurring occurrence is checked for conflicts, when FEAT-17.SPEC-005 applies this spec's rules, then it uses the exact same conflicting-booking definition as a single-date block save (FEAT-17.SPEC-004).

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 15 | 15 |
| Defaults/Derivations | 3 | 3 |
| Business Rules | 7 | 7 |
| Edge Cases | 9 | 9 |
