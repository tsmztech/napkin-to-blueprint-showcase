---
document_type: spec
spec_type: logic-rule
spec_id: FEAT-18.SPEC-010
spec_name: Account & Data Validation Rules
spec_slug: account-data-validation-rules
parent_feature: FEAT-18
parent_feature_name: Account & Data Management
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
rule_count: 13
acceptance_criteria_count: 16
---

# Logic/Rule Spec: Account & Data Validation Rules

## Overview

**Name:** Account & Data Validation Rules
**ID:** FEAT-18.SPEC-010
**Type:** Logic/Rule
**Purpose:** Governs export rate-limiting, irreversible-action confirmation, the 30-day purge window, the organiser hand-over-or-delete-first precondition, offline queuing, and the support-contact description's field rules.
**Parent Feature:** FEAT-18 -- Account & Data Management
**Governed Entity:** Household and Member Profile (the export, removal, and deletion lifecycle actions), plus Support Request (the general-support description)

## Scope and Non-Goals

**In Scope:**
- The export rate-limit condition and its denied behavior
- The irreversible-action confirmation pattern shared by member removal, household deletion, and own-account deletion
- The 30-day purge/no-other-use rule for deleted data
- The organiser hand-over-or-delete-first precondition for own-account deletion
- Offline queuing behavior for every action this feature defines
- Field validation for the Support Request description submitted through FEAT-18.SPEC-005

**Non-Goals:**
- Who may perform each action -- role-based authorization is governed by FEAT-18.SPEC-011 (Account & Data Authorization Rules), not this spec
- Field validation for a member's own display_name, email, and sign-in credential -- governed by FEAT-01's field validation rules for Member Profile, which this feature's My Account screen (FEAT-18.SPEC-004) references rather than duplicates
- Actually performing export compilation, member removal, household deletion, or own-account deletion -- owned respectively by FEAT-18.SPEC-006, FEAT-18.SPEC-007, FEAT-18.SPEC-008, and FEAT-18.SPEC-009, which enforce the rules defined here

## Governed Entity

**Entity:** Household and Member Profile (lifecycle actions), plus Support Request (creation)
**Source:** Feature Dependency Map

| Field | Data Type | Description |
|-------|-----------|-------------|
| Household.status | enum | Active or Closed/Deleted; set to Closed/Deleted at the start of household deletion processing |
| Member Profile.status | enum | Active, Invited, Left, or Removed (the shared enum, per the Feature Dependency Map); this feature sets Removed for both organiser-initiated removal and own-account deletion, distinguished only by which flow performed it |
| Household.organiser | reference | The Member Profile currently holding the organiser role; read to evaluate the own-account deletion precondition |
| Support Request.kind | enum | "general support contact" for requests created by this feature |
| Support Request.note | text | The submitted description, up to 500 characters for support contact |

## Enforced By

| Spec ID | Spec Name | Enforcement Point |
|---------|-----------|---------------------|
| FEAT-18.SPEC-001 | Export Household Data | On Request Export tap (rate-limit condition) |
| FEAT-18.SPEC-002 | Remove Member Profile | On the confirmation step's Remove action (irreversible-action confirmation) |
| FEAT-18.SPEC-003 | Delete Household | On the confirmation step's Delete Household Permanently action (irreversible-action confirmation) |
| FEAT-18.SPEC-004 | My Account | On Delete My Account tap (own-account deletion precondition) and on its own confirmation step (irreversible-action confirmation) |
| FEAT-18.SPEC-005 | Contact Support | On description field blur and on Send submit (field validation) |
| FEAT-18.SPEC-006 | Export Generation Processing | On processing (30-day-no-other-use rule governs how the export is compiled and retained) |
| FEAT-18.SPEC-007 | Member Removal Processing | On processing (30-day purge rule for the removed member's data) |
| FEAT-18.SPEC-008 | Household Deletion Processing | On processing (30-day purge window for the full cascade) |
| FEAT-18.SPEC-009 | Own Account Deletion Processing | On processing (re-check of the organiser hand-over-or-delete-first precondition) |

## Field Validation Rules

| Field | Rule | Condition (if conditional) | When Checked | Error Message | Blocking? |
|-------|------|--------------------------|---------------|------------------|-----------|
| Support Request.note | Required, non-empty | Always (kind: general support contact) | On blur, on submit | "Tell us what's going wrong before sending" | Yes |
| Support Request.note | Max 500 characters | Always | On change (input is capped, not error-shown) | -- input simply stops accepting further characters at 500 -- | No -- the field enforces the cap by not accepting further input, so no separate over-limit error state is reachable |
| Support Request.kind | No validation beyond data type | Always -- system-assigned as "general support contact" for every request created through FEAT-18.SPEC-005 | -- | -- | -- |
| Household.status | No validation beyond data type | Always -- system-managed transition, never directly entered by any user | -- | -- | -- |
| Member Profile.status | No validation beyond data type | Always -- system-managed transition, never directly entered by any user | -- | -- | -- |
| Household.organiser | No validation beyond data type | Always -- read-only for this spec's purposes; changed only through FEAT-09's hand-over flow | -- | -- | -- |

## Cross-Field Rules

| Rule | Fields Involved | Logic | Error Message |
|------|-------------------|-------|-------------------|
| Household closure supersedes member status | Household.status, Member Profile.status | Once Household.status is Closed/Deleted, no Member Profile.status transition through any flow other than the household-deletion cascade itself is honored | "Your household has been deleted." (shown on any screen attempting an action against a closed household) |
| Own-account deletion precondition | Household.organiser, Member Profile (the requester) | Own-account deletion for the requester is permitted only when the requester is not the current Household.organiser, or the household itself has been deleted | "You're the organiser -- hand over the role to another adult or delete the household before deleting your own account." |

## Authorization Rules

N/A -- role-based authorization for every action this feature defines (export, remove member, delete household, manage own account, contact support) is governed entirely by FEAT-18.SPEC-011 (Account & Data Authorization Rules), which is this feature's single authoritative home for the role-action matrix. This spec governs only the entity-level, rate-limit, confirmation, precondition, and offline rules above.

## Defaults and Derivations

| Field | Default/Derivation | When Applied | User Can Override? |
|-------|-----------------------|-----------------|--------------------------|
| Household.status | Active | On household creation (owned by FEAT-01); set to Closed/Deleted only by FEAT-18.SPEC-008 | No |
| Member Profile.status (on removal or own-account deletion) | Set to Removed, whether organiser-initiated or self-initiated | On confirmed removal (FEAT-18.SPEC-007) or confirmed own-account deletion (FEAT-18.SPEC-009) | No |
| Export rate-limit window | Rolls forward on a fixed cadence: platform parameter: `data-export-rate-limit-window` | Always, recalculated once the current window elapses | No |
| Export rate-limit count | Resets to zero at the start of each new rate-limit window | On window rollover | No |

## Business Rules

- **Export rate limit:** For Maya, the Request Export action is allowed only while the household's export count for the current rate-limit window is below platform parameter: `data-export-rate-limit-count`. At the limit, further export requests are blocked until the window resets, with the denied behavior: "You've reached this period's export limit. You can request another export once the limit resets." (feature-overview.md, Validation & Limits: "export requests are rate-limited to a reasonable frequency to prevent abuse"; the exact count and window are platform-set values, per contract C-36, since no specific number is stated in Stage 2.)
- **Irreversible-action confirmation:** Member removal (FEAT-18.SPEC-002), household deletion (FEAT-18.SPEC-003), and own-account deletion (within FEAT-18.SPEC-004) each require the same explicit affirmative action from Maya (or the requesting adult for own-account deletion): a confirmation step that states plainly and specifically what will be lost, with an explicit affirmative tap and no default-confirmed state on any of the three (feature-overview.md, Shared Context: Irreversible-action confirmation, "Spec Writers for all three should describe the pattern identically").
- **30-day purge / no-other-use rule:** Deleted data (from member removal, own-account deletion, or household deletion) is fully removed within 30 days of confirmation and is not retained in any form usable for any other purpose during or after that window (product-features.md, Validation & Limits; ASMP-27). This 30-day figure is a fixed, feature-level product decision stated as a concrete number, not a platform-wide policy value delegated to build time, since it is already established in Stage 2 (scope-boundaries.md SC-18, product-features.md).
- **Organiser hand-over-or-delete-first precondition (XBR-15):** The organiser must hand over the role to an active adult member (FEAT-09) or delete the household (FEAT-18.SPEC-003) before her own account can be deleted. This precondition is checked once at the FEAT-18.SPEC-004 screen level and re-checked at FEAT-18.SPEC-009's processing time, since the organiser state can change between the two moments.
- **Offline queuing:** Any export request, member removal confirmation, household deletion confirmation, own-account deletion confirmation, own-account field edit, or support-contact submission made without connectivity queues locally on the requesting device and completes automatically once connectivity returns (product-features.md, States field: Offline-degraded). This rule is governed once here and referenced by every screen in this feature (FEAT-18.SPEC-001 through FEAT-18.SPEC-005) rather than re-described.

## Edge Cases

- **Household reaches exactly the export rate limit at the same moment two organiser sessions each attempt a request** -- The check reads the current window's count at the moment each request's validation runs; whichever request's check runs after the count reaches the limit is blocked, even if its submission started slightly before the count-reaching request completed (first-decision-wins, consistent with the processing order in FEAT-18.SPEC-006).
- **Export rate-limit window rolls over while a request is mid-validation** -- The validation uses the count as of the moment it runs; a request evaluated just after rollover is checked against the new window's (reset) count.
- **Maya opens the confirmation step on FEAT-18.SPEC-003, then taps Cancel instead of the confirmation's Delete Household Permanently button** -- No deletion occurs; the confirmation step closes back to the Preview state, since no default-confirmed state exists and only the explicit affirmative tap proceeds.
- **Maya's organiser status changes (hand-over completes) between her screen-level precondition check and her processing-time confirmation** -- The processing-time re-check in FEAT-18.SPEC-009 uses the household's organiser state as of that moment, so a hand-over completed in between is honored even though the earlier screen-level check reflected the old state.
- **A support description is submitted at exactly 500 characters** -- Passes validation; the field's input cap means 501 characters is never reachable through typing, so no separate "too long" error state exists for this field.
- **A household is deleted while a member's own-account deletion precondition check is in flight** -- Per the Cross-Field Rules entry, the household's Closed/Deleted status supersedes; the pending own-account deletion is unnecessary once the whole household is gone, and FEAT-18.SPEC-009 treats this as already-satisfied (no further action needed for that member specifically).

## Acceptance Criteria

**FEAT-18.SPEC-010-AC-01:** Given Maya's household is under the export rate limit, when she requests an export, then the request is accepted and FEAT-18.SPEC-006 begins.

**FEAT-18.SPEC-010-AC-02:** Given Maya's household is at the export rate limit, when she attempts a new export request, then it is blocked with "You've reached this period's export limit. You can request another export once the limit resets."

**FEAT-18.SPEC-010-AC-03:** Given two organiser sessions each attempt an export request as the household's count is exactly at the limit, when both checks run, then the request whose check runs after the count reaches the limit is blocked, even if it started slightly earlier.

**FEAT-18.SPEC-010-AC-04:** Given Maya is on the Remove Member Profile confirmation step, when she has not yet tapped the explicit Remove button, then no removal has occurred, since no default-confirmed state exists.

**FEAT-18.SPEC-010-AC-05:** Given Maya taps Delete Household Permanently on FEAT-18.SPEC-003's Preview state, when the confirmation step opens, then no deletion has occurred, since no default-confirmed state exists.

**FEAT-18.SPEC-010-AC-06:** Given Maya is on FEAT-18.SPEC-003's confirmation step, when she taps Cancel instead of the confirmation's Delete Household Permanently button, then no deletion occurs and she returns to the Preview state.

**FEAT-18.SPEC-010-AC-07:** Given a member's data has been deleted through removal or own-account deletion, when the 30-day window is checked, then no form of that data is used for any purpose other than the removal itself during or after that window.

**FEAT-18.SPEC-010-AC-08:** Given Maya is still the household's organiser and has not handed over the role, when she attempts to delete her own account, then the precondition blocks her with the hand-over-or-delete-first message.

**FEAT-18.SPEC-010-AC-09:** Given Maya has handed over the organiser role, when the precondition check runs again, then it passes and her own-account deletion proceeds.

**FEAT-18.SPEC-010-AC-10:** Given Maya's hand-over completes between her screen-level check and this rule's processing-time re-check, when the re-check runs, then it reflects the now-current organiser state and passes.

**FEAT-18.SPEC-010-AC-11:** Given Sam loses connectivity while editing his own account, when he saves, then the edit queues locally and completes automatically once connectivity returns.

**FEAT-18.SPEC-010-AC-12:** Given Maya loses connectivity while confirming a household deletion, when she confirms, then the request queues locally and begins processing once connectivity returns.

**FEAT-18.SPEC-010-AC-13:** Given Sam submits a support description with the field empty, when he attempts to send it, then he sees "Tell us what's going wrong before sending" and no request is created.

**FEAT-18.SPEC-010-AC-14:** Given Sam has typed exactly 500 characters into the support description, when he attempts to type further, then no additional characters are accepted.

**FEAT-18.SPEC-010-AC-15:** Given a household is marked Closed/Deleted, when any other flow attempts a Member Profile status transition against it, then the attempt is refused with "Your household has been deleted."

**FEAT-18.SPEC-010-AC-16:** Given a member's own-account deletion precondition check is in flight when the household is deleted, when the household-wide deletion completes, then the member's own-account deletion is treated as already satisfied by the household-wide cascade.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|-----------------|--------|
| Field Validation Rules | 6 | 6 |
| Cross-Field Rules | 2 | 2 |
| Authorization Rules | 0 (N/A -- covered by FEAT-18.SPEC-011) | 0 |
| Defaults/Derivations | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
