---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-02.SPEC-005
spec_name: Availability Setup Validation & Limits
spec_slug: availability-setup-validation-limits
parent_feature: FEAT-02
parent_feature_name: Availability & Working Hours Setup
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
rule_count: 10
acceptance_criteria_count: 24
---

# Logic/Rule Spec: Availability Setup Validation & Limits

## Overview

**Name:** Availability Setup Validation & Limits
**ID:** FEAT-02.SPEC-005
**Type:** Logic/Rule
**Purpose:** Defines every validation and boundary rule governing the Availability Rule and its per-service buffer override, plus authorization rules for every action on both, shared by FEAT-02.SPEC-001 and FEAT-02.SPEC-002 so neither screen duplicates the rules.
**Parent Feature:** FEAT-02 -- Availability & Working Hours Setup
**Governed Entity:** Availability Rule (weekly_windows, default_buffer, minimum_booking_notice, booking_horizon, effective_from), plus the Service entity's buffer_override field, which this feature's rules also govern even though the field is stored on Service (flagged discrepancy, per feature-overview.md's Entity-Lifecycle Coverage Matrix).

## Scope and Non-Goals

**In Scope:**
- Per-field validation rules for every Availability Rule field and for Service.buffer_override
- Cross-field rules (window ordering, no-overlap, timezone interpretation)
- Authorization rules for every action on the Availability Rule and on Service.buffer_override, for every role in the Access Matrix
- Default values and derived fields (effective_from)
- Error messages for every validation failure

**Non-Goals:**
- Validating any other Service field (name, price, duration, deposit_rule) -- owned by FEAT-01's own validation, not this spec, which addresses buffer_override only
- Computing whether a specific slot is actually bookable (fitting duration, buffer, notice, and horizon together against live bookings, blocks, and calendar busy time) -- owned by FEAT-03 (Real-Time Slot Availability Engine); this spec defines the boundary values Talia may set, not the live computation that consumes them
- Checking existing confirmed bookings against a newly saved rule -- owned by FEAT-02.SPEC-004, which is triggered only after this spec's validation has already passed
- Setting or validating the Pro's account timezone itself -- excluded per XBR-25: FEAT-27 is the sole owner of timezone; this spec only reads it to interpret entered times

## Governed Entity

**Entity:** Availability Rule (plus Service.buffer_override, set through this feature's screens)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| weekly_windows | text (structured) | Per day of week, one or more non-overlapping start/end time pairs; interpreted in the Pro's account timezone |
| default_buffer | number | Minutes applied between consecutive bookings when no per-service override applies |
| minimum_booking_notice | number | How close to an appointment a client may still book, in days |
| booking_horizon | number | How far ahead a client may book, in weeks or months |
| effective_from | date | The date/time from which this version applies -- system-derived, not user-entered |
| Service.buffer_override (referenced) | number | Optional per-service override of default_buffer, 0 to 120 minutes; stored on the Service entity, set only through this feature's screens |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-02.SPEC-001 | Working Hours, Buffer, Notice & Horizon Setup | On field blur and form submit for weekly_windows, default_buffer, minimum_booking_notice, booking_horizon; authorization on screen entry and on save |
| FEAT-02.SPEC-002 | Per-Service Buffer Override | On field blur and form submit for Service.buffer_override; authorization on screen entry and on save |
| FEAT-02.SPEC-003 | Availability Rule Versioning | Runs only after this spec's validation has passed on the triggering screen -- re-derives effective_from at commit time |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| weekly_windows (per window) | Start time must be before end time | Always, for every entered window | On blur | "End time must be after start time" | Yes |
| weekly_windows (per day) | No two windows on the same day may overlap | Always, when a day has 2 or more windows | On blur (of the second window's fields) and on submit | "This window overlaps with another window on the same day" | Yes |
| weekly_windows | At least one window must exist on at least one day before the account's first save can complete (an Availability Rule with zero windows can never make the booking link go live, per XBR-26) | Only on the very first save for this account (the Empty state) | On submit | "Add at least one working window before saving" | Yes |
| default_buffer | Whole number of minutes, 0 to 120 inclusive | Always | On blur | "Buffer must be between 0 and 120 minutes" | Yes |
| minimum_booking_notice | Whole number of days, 0 to 7 inclusive | Always | On blur | "Minimum notice must be between 0 and 7 days" | Yes |
| booking_horizon | Whole number, 1 week to 12 months inclusive (stored and compared in weeks: 1 to 52) | Always | On blur | "Booking horizon must be between 1 week and 12 months" | Yes |
| effective_from | No validation beyond data type -- system-derived, never user-entered | Always | -- | -- | -- |
| Service.buffer_override | Whole number of minutes, 0 to 120 inclusive, same bounds as default_buffer | Only when the Pro sets an override (an empty override is valid and falls back to default_buffer) | On blur | "Override must be between 0 and 120 minutes" | Yes |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|---------------|
| Window ordering and no-overlap is evaluated per day, across all windows for that day | weekly_windows (all windows within one day) | Every window on a given day must individually pass the start-before-end rule, and no two windows on that day may share any overlapping time span, including a zero-length gap treated as non-overlapping (e.g., 9:00-12:00 and 12:00-15:00 are valid adjacent windows, not an overlap) | "This window overlaps with another window on the same day" |
| All entered and displayed times are interpreted in the Pro's account timezone | weekly_windows, Pro Account.timezone (read-only reference) | Every start/end time the Pro enters is stored and interpreted against the Pro Account's current timezone (FEAT-27); the screen displays the active timezone label alongside the fields rather than asking the Pro to specify it here | N/A -- this is an interpretation rule, not a rejectable input; there is no invalid-timezone state on this screen since timezone is never entered here |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|---------------------------------------------|
| Create the initial Availability Rule (first save) | The Pro (Talia) | Always | -- |
| View the current Availability Rule (weekly hours, buffer, notice, horizon) | The Pro (Talia) | Always (own account only) | -- |
| Update the Availability Rule (create a new version) | The Pro (Talia) | Always (own account only) | -- |
| Set or clear a Service.buffer_override | The Pro (Talia) | Always, for the Pro's own active services only | -- |
| View the current Availability Rule or Service.buffer_override | The Client (Riley) | Never | No client-facing screen or field exposes any part of the Availability Rule or a service's buffer_override at any time; the client sees only the resulting open times computed by FEAT-03, never the rule itself |
| Update the Availability Rule or Service.buffer_override | The Client (Riley) | Never | No client-facing control exists to attempt this action |
| View the current Availability Rule or Service.buffer_override | Platform Operator (Support) | Always, read-only, for troubleshooting only | -- |
| Update the Availability Rule or Service.buffer_override | Platform Operator (Support) | Never | All edit and save controls are hidden on both FEAT-02.SPEC-001 and FEAT-02.SPEC-002 when viewed by Support; a direct attempt through a stale or replayed request is refused with "Support access is read-only and cannot make changes to this account." (XBR-24) |
| Any action on the Availability Rule or Service.buffer_override | Unauthenticated visitor | Never | Redirected to the Pro sign-in screen (FEAT-29); a failed sign-in never reveals whether an account exists (XBR-29) |
| View or update the Availability Rule or Service.buffer_override with an expired session | The Pro (Talia) | Never, until re-authenticated | Dialog: "Your session has expired. Sign in to continue."; any unsaved edits are preserved and restored after successful re-authentication |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| effective_from | Set to the exact moment the triggering save commits | On every create and every new version (FEAT-02.SPEC-003) | No -- always system-derived, never user-entered |
| minimum_booking_notice (initial value, before the Pro has ever set one) | Defaults to 0 days, per product-features.md's stated default of "a few hours" -- this field is validated and stored only in whole days (0 to 7, per the Field Validation Rules table above), and a span of a few hours is less than one whole day, so it resolves to the field's minimum representable value, 0 days, at this field's day-level granularity | On the account's first Availability Rule creation only, shown as the pre-filled value the Pro can change before her first save | Yes -- the Pro may change it before or at any point after the first save |
| booking_horizon (initial value, before the Pro has ever set one) | Defaults to 8 weeks, per product-features.md's stated default | On the account's first Availability Rule creation only, shown as the pre-filled value the Pro can change before her first save | Yes -- the Pro may change it before or at any point after the first save |
| default_buffer (initial value, before the Pro has ever set one) | Defaults to 0 minutes -- no gap is assumed until the Pro sets one | On the account's first Availability Rule creation only | Yes |
| Service.buffer_override (initial value, per service) | No override -- falls back to the current Availability Rule's default_buffer until the Pro explicitly sets one | Always, for a service with no override on record | Yes -- setting an override at any time takes precedence over the default until cleared |

## Business Rules

- Minimum booking notice and booking horizon limit every client-facing booking path computed by FEAT-03; the Pro alone may book inside notice or beyond horizon through Pro Booking Management (XBR-03) -- this exception is enforced by FEAT-30, not by this spec, which governs only what Talia may enter here.
- All hours are interpreted in the Pro's account timezone, which this feature reads but never sets; a timezone change is owned entirely by FEAT-27 (XBR-25).
- A Service.buffer_override uses the exact same numeric bounds as default_buffer (0 to 120 minutes) so the two fields behave identically wherever either is displayed (feature-overview.md's Shared Context: "same bounds validation... described identically across both screens").
- These validation and authorization rules apply identically whether the save originates from FEAT-02.SPEC-001 or FEAT-02.SPEC-002 -- the product definition establishes no screen-specific exception to any rule in this spec.
- A validated save always proceeds to FEAT-02.SPEC-003 (versioning) and, through it, to FEAT-02.SPEC-004 (conflict flagging); this spec's rules gate entry into that pipeline but do not themselves check existing bookings.

## Edge Cases

- **A window is entered with start time exactly equal to end time** -- Rejected: end time must be strictly after start time, not merely equal.
- **Two windows on the same day share exactly one boundary instant** (e.g., 9:00-12:00 and 12:00-15:00) -- Accepted as non-overlapping adjacent windows, not rejected.
- **default_buffer entered as exactly 0** -- Accepted; a zero buffer is valid and means back-to-back bookings are allowed.
- **default_buffer entered as exactly 120** -- Accepted (the upper bound); 121 is rejected.
- **minimum_booking_notice entered as exactly 0** -- Accepted; a client may book right up to the appointment time.
- **minimum_booking_notice entered as exactly 7 (days)** -- Accepted (the upper bound); 8 is rejected.
- **booking_horizon entered as exactly 1 week or exactly 12 months (52 weeks)** -- Both boundary values are accepted; anything outside that range is rejected.
- **Service.buffer_override entered as exactly 0 or exactly 120** -- Both boundary values are accepted, identically to default_buffer's own bounds.
- **The Pro clears a Service.buffer_override entirely (empty field) rather than entering 0** -- Treated as "no override" (falls back to default_buffer), distinct from an explicit override of 0 minutes, which is itself a valid, stored value.
- **The Pro's account timezone changes (via FEAT-27) between when a window's times were entered and when the save commits** -- The save commits using the timezone current at commit time; the screen (FEAT-02.SPEC-001) surfaces a one-time notice that the displayed label has updated, per that spec's own edge cases.
- **A Support view attempts to invoke the save action directly (bypassing the hidden controls)** -- Refused with "Support access is read-only and cannot make changes to this account." regardless of the values submitted.
- **The Pro's session expires mid-edit with valid but unsaved values on screen** -- No validation or save occurs until re-authentication succeeds; the unsaved values are preserved and re-validated against the current rules once the Pro is signed back in.

## Acceptance Criteria

**FEAT-02.SPEC-005-AC-01:** Given Talia enters a window with start time 2:00 PM and end time 1:00 PM on the same day, when the field loses focus, then the error "End time must be after start time" appears and the window cannot be saved.

**FEAT-02.SPEC-005-AC-02:** Given Talia enters a window with start time 9:00 AM and end time 12:00 PM, when the field loses focus, then no error appears.

**FEAT-02.SPEC-005-AC-03:** Given Talia has a 9:00 AM-1:00 PM window on Monday and adds a second window of 12:00 PM-3:00 PM on Monday, when she attempts to save, then the error "This window overlaps with another window on the same day" appears and the save does not proceed.

**FEAT-02.SPEC-005-AC-04:** Given Talia has a 9:00 AM-12:00 PM window on Monday and adds a second window of exactly 12:00 PM-3:00 PM, when she attempts to save, then no overlap error appears, since the windows are adjacent, not overlapping.

**FEAT-02.SPEC-005-AC-05:** Given Talia has never set any working window, when she attempts to save with zero windows across every day, then the error "Add at least one working window before saving" appears and the save does not proceed.

**FEAT-02.SPEC-005-AC-06:** Given Talia enters 121 as her default buffer, when the field loses focus, then the error "Buffer must be between 0 and 120 minutes" appears.

**FEAT-02.SPEC-005-AC-07:** Given Talia enters 120 as her default buffer, when the field loses focus, then no error appears and the value is accepted.

**FEAT-02.SPEC-005-AC-08:** Given Talia enters 8 as her minimum booking notice in days, when the field loses focus, then the error "Minimum notice must be between 0 and 7 days" appears.

**FEAT-02.SPEC-005-AC-09:** Given Talia enters 0 as her minimum booking notice, when the field loses focus, then no error appears and the value is accepted.

**FEAT-02.SPEC-005-AC-10:** Given Talia enters a booking horizon of 13 months, when the field loses focus, then the error "Booking horizon must be between 1 week and 12 months" appears.

**FEAT-02.SPEC-005-AC-11:** Given Talia enters a booking horizon of exactly 1 week, then no error appears and the value is accepted.

**FEAT-02.SPEC-005-AC-12:** Given Talia sets a per-service buffer override of 130 minutes on FEAT-02.SPEC-002, when the field loses focus, then the error "Override must be between 0 and 120 minutes" appears.

**FEAT-02.SPEC-005-AC-13:** Given Talia sets a per-service buffer override of exactly 0 minutes, then it is accepted and stored as an explicit override distinct from having no override at all.

**FEAT-02.SPEC-005-AC-14:** Given Talia (the Pro) is the sole role interacting with this spec's screens, when she saves valid values, then the save proceeds without any authorization check blocking her.

**FEAT-02.SPEC-005-AC-15:** Given Riley (the Client) has no screen or control anywhere in the product that exposes any Availability Rule field or a service's buffer_override, then Riley can never view or update either.

**FEAT-02.SPEC-005-AC-16:** Given Platform Operator (Support) opens Talia's account, when Support views the Availability Rule or Service.buffer_override, then all values are shown read-only with no edit or save controls present.

**FEAT-02.SPEC-005-AC-17:** Given a stale or replayed save request reaches the system while flagged as a Support-originated action, then it is refused with "Support access is read-only and cannot make changes to this account." regardless of the values submitted.

**FEAT-02.SPEC-005-AC-18:** Given a visitor who is not signed in as a Pro attempts to view or update the Availability Rule, then they are redirected to the Pro sign-in screen without any indication of whether an account exists.

**FEAT-02.SPEC-005-AC-19:** Given Talia's session expires with valid but unsaved values entered, when she signs back in, then her unsaved values are restored and re-validated against these same rules before saving.

**FEAT-02.SPEC-005-AC-20:** Given a brand-new Pro Account with no Availability Rule yet, when the Working Hours screen loads for the first time, then minimum_booking_notice is pre-filled at 0 days (product-features.md's "a few hours" resolved to this field's minimum whole-day value) and booking_horizon is pre-filled at 8 weeks, and default_buffer is pre-filled at 0.

**FEAT-02.SPEC-005-AC-21:** Given Talia saves her first Availability Rule, when the save commits, then effective_from is set to the exact moment of commit and cannot be edited by Talia.

**FEAT-02.SPEC-005-AC-22:** Given a service with no buffer_override set, when the current Availability Rule's default_buffer changes, then that service's effective buffer follows the new default automatically, since no override is on record for it.

**FEAT-02.SPEC-005-AC-23:** Given all entered window times, when they are displayed anywhere on FEAT-02.SPEC-001, then they are labeled and interpreted using the Pro Account's current timezone as set by FEAT-27, never a hard-coded or separately entered timezone.

**FEAT-02.SPEC-005-AC-24:** Given Talia clears a previously set Service.buffer_override, when she saves, then the field is stored as having no override (not as 0), and the service's effective buffer reverts to following the account default.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 8 | 8 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 10 | 10 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 12 | 12 |
