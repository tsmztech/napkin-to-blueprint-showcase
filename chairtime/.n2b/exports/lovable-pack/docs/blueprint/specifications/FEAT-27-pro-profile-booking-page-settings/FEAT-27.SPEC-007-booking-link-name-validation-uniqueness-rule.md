---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-27.SPEC-007
spec_name: Booking Link Name Validation & Uniqueness Rule
spec_slug: booking-link-name-validation-uniqueness-rule
parent_feature: FEAT-27
parent_feature_name: Pro Profile & Booking Page Settings
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 8
acceptance_criteria_count: 12
---

# Logic/Rule Spec: Booking Link Name Validation & Uniqueness Rule

## Overview

**Name:** Booking Link Name Validation & Uniqueness Rule
**ID:** FEAT-27.SPEC-007
**Type:** Logic/Rule
**Purpose:** Defines the format, length, and cross-pro uniqueness rules for booking_link_name, and the reject-with-refresh behavior when another pro claims a candidate name first.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings
**Governed Entity:** Pro Account -- the booking_link_name field and its forwarding-reservation state

## Scope and Non-Goals

**In Scope:**
- Format and length rules for booking_link_name
- Cross-pro uniqueness, including names reserved by another pro's still-active forwarding window
- The reject-with-refresh behavior when a candidate name is claimed by another pro between availability check and save
- Authorization for who may rename a booking link

**Non-Goals:**
- The forwarding and reservation-release mechanism itself (what happens after an accepted rename) -- owned by FEAT-27.SPEC-010 (Booking Link Forwarding & Reservation Expiry); this spec governs acceptance, not the consequence
- The rename screen's layout and suggestion-chip presentation -- owned by FEAT-27.SPEC-002 (Booking Link Rename); this spec only supplies the rules it enforces
- Resolving a renamed or forwarded link on the public booking page -- owned by FEAT-05.SPEC-001 and FEAT-05.SPEC-008; this spec governs the Pro Account field itself, not the client-facing lookup

## Governed Entity

**Entity:** Pro Account (booking_link_name and its forwarding-reservation state)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| booking_link_name | text | The public link segment identifying the Pro's booking page; unique across all pros |
| previous_link_names | derived (list) | Names this Pro Account has renamed away from, each with the timestamp its forwarding window began |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-27.SPEC-002 | Booking Link Rename | On field change (format/length), on pause in typing (availability), and on submit (final acceptance) |
| FEAT-27.SPEC-010 | Booking Link Forwarding & Reservation Expiry | Consumes this spec's acceptance outcome to begin forwarding and reservation |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| booking_link_name | Required, non-empty | Always | On blur and on submit | "Enter a link name" | Yes |
| booking_link_name | 3-40 characters | Always | On blur and on submit | "Your link name must be 3-40 characters" | Yes |
| booking_link_name | Letters, numbers, or hyphens only | Always | On blur and on submit | "Only letters, numbers, and hyphens are allowed" | Yes |
| booking_link_name | Must not already be a different pro's active booking_link_name | Always | On pause in typing (availability check) and on submit | "That link name is taken" | Yes |
| booking_link_name | Must not be inside another pro's active forwarding-reservation window (platform parameter: `booking-link-forward-window-months`) | Always | On pause in typing (availability check) and on submit | "That link name is taken" (reservation and active-use are shown identically to the Pro renaming; the distinction is internal only) | Yes |
| previous_link_names | No validation beyond data type -- system-maintained, never directly editable | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Own-name reclaim exception | booking_link_name, previous_link_names | A candidate name that matches one of the requesting Pro's own previous_link_names entries is available to that same Pro, even while its forwarding-reservation window is still active -- the reservation exists to keep the name from other pros, not from its own former owner | N/A -- this is a permissive exception, not an error condition |
| No-op rename guard | booking_link_name (current) vs. candidate | If the candidate equals the Pro's current booking_link_name, the rename action is not offered as a change | "This is already your current link name" |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Rename booking link | The Pro (Talia) | Always, on her own Pro Account only | -- |
| Rename booking link | The Client (Riley) | Never | No control of any kind is exposed to clients; a client's own experience is limited to being forwarded transparently (XBR-27) |
| Rename booking link | Platform Operator (Support) | Never | The rename input and Save control are shown disabled and labeled "View-only in support mode" on FEAT-27.SPEC-002; support has no path to submit a rename |
| View current and previous link names | The Pro (Talia) | Always, her own account | -- |
| View current and previous link names | Platform Operator (Support) | Always, view-only, for troubleshooting (ASMP-20) | -- |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| booking_link_name (initial value) | Suggested from the Pro's display_name at account creation (FEAT-15), rendered available-by-construction | On Pro Account creation, before this spec's rules first apply | Yes -- Talia may change it immediately or at any later time through FEAT-27.SPEC-002 |
| previous_link_names | Appended with the outgoing name and a forwarding-start timestamp each time a rename is accepted | On every accepted rename (via FEAT-27.SPEC-010) | No -- system-maintained history, never user-edited |

## Business Rules

- XBR-27: a renamed booking link keeps forwarding from the old name for at least platform parameter: `booking-link-forward-window-months`; a closed, paused-to-closure, or mistyped link shows the plain "this booking page isn't available" message, never another pro's page.
- Uniqueness is checked against both currently active booking_link_name values across all pros and every name still inside another pro's forwarding-reservation window -- a name is available only when neither condition holds.
- Acceptance of a rename here is what authorizes FEAT-27.SPEC-010 to begin forwarding and reservation for the outgoing name; this spec never itself starts that mechanism.
- The reject-with-refresh resolution named by the dependency map's Pro Account Contention line applies specifically to this field: if a candidate name is claimed by another pro between the Pro's availability check and her save, the save is rejected and the screen (FEAT-27.SPEC-002) refreshes with current availability and fresh suggestions -- the first committed rename wins.

## Edge Cases

- **Candidate name at exactly 3 characters** -- Passes validation. 2 characters shows the length error.
- **Candidate name at exactly 40 characters** -- Passes validation. 41 characters shows the length error.
- **Candidate name with an underscore or space** -- Fails the character-set rule; "Only letters, numbers, and hyphens are allowed."
- **Two pros submit the same never-before-used candidate name at effectively the same moment** -- The first submission committed wins; the second is rejected with "That link name is taken" and its screen refreshes with fresh suggestions, per reject-with-refresh.
- **A pro renames back to a name she used longer ago than platform parameter: `booking-link-forward-window-months`, now outside anyone's reservation** -- Available to any pro, including a different pro than its original owner, since the reservation window has fully lapsed.
- **A pro attempts to reuse a name still inside her own forwarding-reservation window** -- Available to reclaim (Cross-Field Rules: Own-name reclaim exception), since the reservation protects against other pros, not the name's own former owner.
- **Support attempts to submit a rename through a direct request while viewing FEAT-27.SPEC-002** -- Not possible: the rename input and Save control do not exist in an actionable state in Support's view; this spec's Authorization Rules confirm Support is never an allowed actor for this action regardless of any client-side state.

## Acceptance Criteria

**FEAT-27.SPEC-007-AC-01:** Given Talia enters a candidate name of "ab" (2 characters), when validation runs, then she sees "Your link name must be 3-40 characters."

**FEAT-27.SPEC-007-AC-02:** Given Talia enters a candidate name of exactly 40 characters using only letters, numbers, and hyphens, when validation runs, then it passes the format and length checks.

**FEAT-27.SPEC-007-AC-03:** Given Talia enters a candidate name containing an underscore, when validation runs, then she sees "Only letters, numbers, and hyphens are allowed."

**FEAT-27.SPEC-007-AC-04:** Given Talia enters a candidate name currently active as another pro's booking_link_name, when the availability check runs, then she sees "That link name is taken."

**FEAT-27.SPEC-007-AC-05:** Given Talia enters a candidate name still inside another pro's forwarding-reservation window, when the availability check runs, then she sees "That link name is taken", identically to an actively-used name.

**FEAT-27.SPEC-007-AC-06:** Given Talia enters a candidate name that is one of her own previous_link_names still inside its own forwarding window, when the availability check runs, then it shows as available to her.

**FEAT-27.SPEC-007-AC-07:** Given Talia's candidate name is claimed by another pro between her availability check and her Save tap, when she submits, then her save is rejected with "That link name is taken" and FEAT-27.SPEC-002 refreshes with current availability, per reject-with-refresh.

**FEAT-27.SPEC-007-AC-08:** Given Talia enters her current booking_link_name unchanged, when she views the Save control on FEAT-27.SPEC-002, then it is disabled with "This is already your current link name."

**FEAT-27.SPEC-007-AC-09:** Given Talia (the Pro) attempts to rename her link, when she submits a valid, available name, then the action is allowed and FEAT-27.SPEC-010 is authorized to begin forwarding for the outgoing name.

**FEAT-27.SPEC-007-AC-10:** Given a support operator views FEAT-27.SPEC-002, when they look for a way to submit a rename, then no actionable input or Save control is available to them, consistent with this spec's Authorization Rules denying the action to Support entirely.

**FEAT-27.SPEC-007-AC-11:** Given two pros submit the same previously-unused candidate name within moments of each other, when the first submission commits, then the second is rejected with "That link name is taken" and refreshed suggestions.

**FEAT-27.SPEC-007-AC-12:** Given a name's forwarding-reservation window has fully lapsed (more than platform parameter: `booking-link-forward-window-months` since the rename), when any pro (including one other than its original owner) checks its availability, then it shows as available.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 5 | 5 |
| Defaults/Derivations | 2 | 2 |
| Business Rules | 4 | 4 |
| Edge Cases | 7 | 7 |
