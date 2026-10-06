---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-09.SPEC-002
spec_name: Policy Versioning & Cutoff Rendering
spec_slug: policy-versioning-cutoff-rendering
parent_feature: FEAT-09
parent_feature_name: Cancellation & No-Show Policy Engine
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 19
acceptance_criteria_count: 15
---

# Logic/Rule Spec: Policy Versioning & Cutoff Rendering

## Overview

**Name:** Policy Versioning & Cutoff Rendering
**ID:** FEAT-09.SPEC-002
**Type:** Logic/Rule
**Purpose:** Governs how an edit to the Cancellation Policy always creates a new, immutable version rather than overwriting the current one, how every Booking stays permanently bound to the version it acknowledged, and how this booking's exact cutoff time and plain-language wording are computed wherever another feature needs to display them.
**Parent Feature:** FEAT-09 -- Cancellation & No-Show Policy Engine
**Governed Entity:** Cancellation Policy (versioning and field rules), jointly with the Booking.policy_version binding (read-only slice)

## Scope and Non-Goals

**In Scope:**
- Field validation for window_hours, the only Pro-editable field on the Cancellation Policy
- The immutable-versioning rule: every edit creates a new version effective immediately, never an overwrite
- Deriving plain_language_wording from window_hours (system-derived, never Pro-authored)
- Binding a Booking permanently to the policy version in force at the moment the client acknowledges it, and computing that booking's exact cutoff time (start_time minus window_hours)
- The contention rule when a policy version changes between a client's acknowledgment and payment capture
- Authorization for every action on the Cancellation Policy, across every role in the Access Matrix

**Non-Goals:**
- The Cancellation Policy setup screen's layout and interactions -- owned by FEAT-09.SPEC-001; this spec defines the rules that screen enforces, not its UI
- Deciding what outcome (refund or forfeit) applies to a given cancellation, reschedule, or no-show -- owned by FEAT-09.SPEC-003 (Deposit Outcome Rules); this spec only governs the policy record itself and the cutoff time the outcome rules are evaluated against
- Creating the very first Cancellation Policy version during onboarding -- per the dependency map's Cancellation Policy lifecycle line, Create is owned by FEAT-15; this spec governs every version created from that point forward, including the first Pro-initiated edit
- The client-facing screens that display the rendered wording and cutoff (FEAT-05's policy-acknowledgment step, FEAT-10's cancellation preview) -- owned by those features; this spec defines the rendering logic they consume, not their own layouts

## Governed Entity

**Entity:** Cancellation Policy (primary), with a read-only reference to Booking's policy-binding fields
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|--------------|
| window_hours | number | Whole hours, 1-168, before the appointment; the sole Pro-editable value |
| inside_window_outcome | enum (fixed) | Deposit kept -- binary in v1, never Pro-configurable |
| outside_window_outcome | enum (fixed) | Full refund -- binary in v1, never Pro-configurable |
| plain_language_wording | derived text | The exact text shown to clients, computed from window_hours; never directly editable |
| version / effective_from | derived | The version number and the date/time from which this version applies; system-managed, never directly editable |
| Booking.policy_version (read-only reference) | reference | The specific Cancellation Policy version a given Booking is permanently bound to, set at the moment of client acknowledgment (owned and written by FEAT-05/FEAT-07, read-only here) |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|-------------------|
| FEAT-09.SPEC-001 | Cancellation Policy Setup | On field blur and on Save; authorization on screen entry (controls hidden for non-Pro roles) and on Save |
| FEAT-09.SPEC-003 | Deposit Outcome Rules | Reads the cutoff time this spec computes as the boundary every deposit-outcome comparison is made against |
| FEAT-09.SPEC-004 | Cancellation & No-Show Outcome Evaluation | Reads the Booking's bound policy version and computed cutoff at the moment of evaluation |
| FEAT-16.SPEC-002 (Activity Event Recording) | Cross-feature | On the client's policy acknowledgment, records an Activity Event carrying the Booking reference, the bound Cancellation Policy version, the plain-language wording shown, and the acknowledgment timestamp, reading the version binding this spec defines without altering it |
| FEAT-05 (Public Booking Page & Booking Flow) | Cross-feature | Renders this booking's exact wording and cutoff at the policy-acknowledgment step, and binds Booking.policy_version at that moment |
| FEAT-07 (Deposit Payment at Booking) | Cross-feature | Re-checks that the acknowledged version still matches the current version at payment capture; applies the contention rule below if it does not |
| FEAT-10 (Client-Initiated Cancel/Reschedule) | Cross-feature | Renders the same wording and cutoff again at cancellation/reschedule time, using the booking's bound version, not the current one |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|-------------|---------------|-----------|
| window_hours | Required; whole number; between 1 and 168 inclusive | Always | On blur and on Save (FEAT-09.SPEC-001) | "Enter a whole number of hours between 1 and 168." | Yes |
| inside_window_outcome | No validation beyond data type -- fixed value, never Pro-input | Always | -- | -- | -- |
| outside_window_outcome | No validation beyond data type -- fixed value, never Pro-input | Always | -- | -- | -- |
| plain_language_wording | No direct validation -- always system-derived from window_hours (Defaults and Derivations below); never accepts direct input | Always | -- | -- | -- |
| version / effective_from | No direct validation -- always system-managed (Defaults and Derivations below); never accepts direct input | Always | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|----------------|-------|----------------|
| Cutoff derivation | window_hours, Booking.start_time | A Booking's cutoff time = Booking.start_time minus its bound version's window_hours; this is the single value every cancellation/reschedule/no-show outcome check (FEAT-09.SPEC-003, FEAT-09.SPEC-004) and every client-facing display (FEAT-05, FEAT-10) reads, so all four never drift from one another | N/A -- structural derivation, no client-facing error |
| Version-immutability rule | window_hours, version/effective_from | Any change to window_hours is written as a brand-new version record with a new effective_from, never as an update to the existing version's window_hours field | N/A -- structural guarantee, enforced at the point of save, not by a client-facing validation error |

## Authorization Rules

| Action | Allowed For | Condition | Denied Behavior (exact message/experience) |
|--------|------------|-----------|--------------------------------------------|
| Create the first Cancellation Policy version | The Pro (Talia) | Only during onboarding (FEAT-15); owned by that feature, not this spec | -- |
| Edit the cancellation window (create a new version) | The Pro (Talia) | Always | -- |
| View the current version's wording and cutoff for a specific booking | The Pro (Talia), The Client (Riley) | Riley: only for her own booking, via FEAT-05 (booking-time) or FEAT-10 (cancellation-time); Talia: any of her own bookings | Riley attempting to view another client's booking's policy wording is impossible -- FEAT-06's access-link scoping never surfaces another client's booking in the first place |
| View the Cancellation Policy for support purposes | Platform Operator (Support) | View-only, for the Pro account under an active help request | -- |
| Author or edit the plain-language wording directly | The Pro (Talia) | Never -- the wording is always system-derived from window_hours | The wording field is never presented as an input; Talia sees it only as read-only preview text on FEAT-09.SPEC-001 |
| Delete or archive a policy version | Any role | Never -- no delete/archive path exists for this entity | No delete or archive control exists anywhere in the product for this entity |
| Change which version an already-confirmed Booking is bound to | Any role | Never -- XBR-08: existing bookings keep the version they were created against, permanently | No control anywhere in the product allows re-binding a confirmed Booking's policy_version |

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-------------------|-------------|-------------------|
| plain_language_wording | Composed from window_hours using a fixed template: "Clients cancelling or rescheduling less than {window_hours} hours before their appointment, or who don't show up, will have their deposit kept. Cancelling earlier refunds the deposit in full." | Whenever window_hours changes (live preview on FEAT-09.SPEC-001) and on every save | No -- always derived, never directly editable |
| version | Incremented from the previous version's number | On every save that changes window_hours | No |
| effective_from | Set to the moment of save | On every save that changes window_hours | No |
| Booking's cutoff time (rendering only, not stored on Cancellation Policy) | Booking.start_time minus the bound version's window_hours | Computed on demand whenever FEAT-05, FEAT-10, FEAT-09.SPEC-003, or FEAT-09.SPEC-004 needs it | No |
| First version's window_hours (onboarding) | Recommended default proposed by FEAT-15 (platform parameter: `cancellation-window-default-hours`) | On account creation, before Talia's first edit | Yes -- Talia may accept the default or change it before her first save, and at any time thereafter via FEAT-09.SPEC-001 |

## Business Rules

- XBR-08: every Booking is governed by the Cancellation Policy version shown and acknowledged at the moment the client books; a later edit to window_hours never changes an already-confirmed Booking's outcome basis.
- **Versioning is edit-only, never in-place:** every save on FEAT-09.SPEC-001 that changes window_hours produces a new version record; the previous version is retained, unchanged, for as long as any Booking references it (Entity-Lifecycle Coverage Matrix; no purge policy applies, per this feature's Non-Goals).
- **Version-changed-during-checkout contention:** if the Cancellation Policy's current version changes between a client's acknowledgment on FEAT-05 and the deposit's capture on FEAT-07, the payment attempt is refused with a refresh; the client is shown the current wording and must re-acknowledge it before payment can proceed. This is the dependency map's Cancellation Policy Contention resolution rule, enforced jointly by FEAT-05 and FEAT-07, which both read this spec's version state to detect the change.
- **Rendering consistency:** wherever another feature displays this booking's cutoff time and wording (FEAT-05 at booking, FEAT-10 at cancellation), it renders the output of this spec's derivation rather than re-deriving it independently, so the wording a client acknowledged always matches the wording shown later.
- Onboarding's recommended default (platform parameter: `cancellation-window-default-hours`) satisfies XBR-26's go-live precondition of an active cancellation policy until Talia changes it.

## Edge Cases

- **window_hours at exactly 1 hour** -- Passes validation; the derived wording reads "less than 1 hour before their appointment."
- **window_hours at exactly 168 hours** -- Passes validation; the boundary value is accepted with no special-casing.
- **window_hours at 0 or 169** -- Fails validation with "Enter a whole number of hours between 1 and 168."
- **window_hours entered as a decimal (e.g., 24.5)** -- Fails validation; only whole numbers are accepted, consistent with the "whole number of hours" rule.
- **A client acknowledges the policy, then Talia edits the window before the client's payment captures** -- The contention rule fires: the client's payment attempt is refused with refresh, and the client re-acknowledges the current wording before paying. Talia's own save is never blocked or delayed by an in-progress client checkout.
- **A client's booking already exists under version 3, and Talia has since saved versions 4 and 5** -- The booking's cutoff and wording are always computed from version 3, its bound version, regardless of how many later versions exist.
- **Two overlapping edits from Talia's own two signed-in devices** -- Last-write-wins between Talia's own sessions, consistent with the dependency map's general pattern for Pro-only-edited entities with no other concurrent writer; whichever save commits last becomes the current version, and the other device's stale form is refreshed to reflect it on its next load.
- **A no-show is evaluated against the cutoff** -- FEAT-09.SPEC-003 treats a no-show as always inside-window-equivalent regardless of the computed cutoff time; this spec still supplies the bound version and wording for display, even though the no-show outcome itself does not depend on the cutoff comparison.

## Acceptance Criteria

**FEAT-09.SPEC-002-AC-01:** Given Talia enters 24 for window_hours and saves, then the plain-language wording reads "Clients cancelling or rescheduling less than 24 hours before their appointment, or who don't show up, will have their deposit kept. Cancelling earlier refunds the deposit in full."

**FEAT-09.SPEC-002-AC-02:** Given Talia enters exactly 1 hour, when she saves, then the value is accepted and a new version is created.

**FEAT-09.SPEC-002-AC-03:** Given Talia enters exactly 168 hours, when she saves, then the value is accepted and a new version is created.

**FEAT-09.SPEC-002-AC-04:** Given Talia enters 0 hours, when validation runs, then the error "Enter a whole number of hours between 1 and 168." is shown and no version is created.

**FEAT-09.SPEC-002-AC-05:** Given Talia enters 169 hours, when validation runs, then the error "Enter a whole number of hours between 1 and 168." is shown and no version is created.

**FEAT-09.SPEC-002-AC-06:** Given Talia's current policy is version 3 with a 24-hour window, when she saves a new 48-hour window, then a new version 4 is created effective immediately, and version 3 is retained unchanged.

**FEAT-09.SPEC-002-AC-07:** Given Riley's booking was created and acknowledged under version 3, when Talia later saves versions 4 and 5, then Riley's booking's cutoff and wording remain computed from version 3.

**FEAT-09.SPEC-002-AC-08:** Given Riley's booking is for an appointment at 3:00 PM and its bound version has a 24-hour window, when FEAT-10 renders her cancellation preview, then the cutoff time shown is 3:00 PM the day before.

**FEAT-09.SPEC-002-AC-09:** Given Riley has acknowledged the current wording on FEAT-05's booking flow, when Talia saves a new version before Riley's deposit captures, then Riley's payment attempt is refused with a refresh, and she is shown the current wording to re-acknowledge before paying.

**FEAT-09.SPEC-002-AC-10:** Given Talia attempts to author custom wording anywhere in the product, when she looks for a wording input field, then none exists -- the wording is always the read-only derived preview shown on FEAT-09.SPEC-001.

**FEAT-09.SPEC-002-AC-11:** Given Platform Operator (Support) opens the Cancellation Policy for Talia's account under an active help request, when they view it, then they see the current version's window and wording read-only, with no edit control.

**FEAT-09.SPEC-002-AC-12:** Given any role looks for a way to delete or archive a policy version, then no such control exists anywhere in the product.

**FEAT-09.SPEC-002-AC-13:** Given any role looks for a way to re-bind a confirmed booking to a different policy version, then no such control exists anywhere in the product, consistent with XBR-08.

**FEAT-09.SPEC-002-AC-14:** Given Talia has two devices signed in and saves an edit from each at effectively the same time, when both saves commit, then the version from whichever save committed last becomes the current version, and the other device's form refreshes to reflect it on next load.

**FEAT-09.SPEC-002-AC-15:** Given Talia has not yet completed onboarding, when FEAT-15 reaches the policy step, then the window is pre-filled with the recommended default (platform parameter: `cancellation-window-default-hours`), which Talia may accept or change before her first save.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Field Validation Rules | 5 | 5 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 7 | 7 |
| Defaults/Derivations | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |
