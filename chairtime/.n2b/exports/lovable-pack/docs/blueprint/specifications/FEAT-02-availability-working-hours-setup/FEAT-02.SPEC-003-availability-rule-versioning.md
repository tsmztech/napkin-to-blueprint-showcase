---
document_type: spec
spec_type: automation
spec_id: FEAT-02.SPEC-003
spec_name: Availability Rule Versioning
spec_slug: availability-rule-versioning
parent_feature: FEAT-02
parent_feature_name: Availability & Working Hours Setup
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-26
acceptance_criteria_count: 11
---

# Automation Spec: Availability Rule Versioning

## Overview

**Name:** Availability Rule Versioning
**ID:** FEAT-02.SPEC-003
**Type:** Automation
**Purpose:** System saves every passing Availability Rule edit as a new dated version rather than overwriting the prior one, so past bookings keep the rule that was live when they were made.
**Parent Feature:** FEAT-02 -- Availability & Working Hours Setup

## Scope and Non-Goals

**In Scope:**
- Creating the initial Availability Rule on a Pro's first passing save
- Creating a new, dated version of the Availability Rule on every subsequent passing save, from either FEAT-02.SPEC-001 or FEAT-02.SPEC-002
- Setting the new version's effective_from date and retaining every prior version
- Triggering the confirmed-booking conflict check (FEAT-02.SPEC-004) once the new version is committed

**Non-Goals:**
- Validating the entered values -- owned entirely by FEAT-02.SPEC-005; this automation only runs after validation has already passed
- Checking existing confirmed bookings against the new version -- owned by FEAT-02.SPEC-004, which this automation triggers but does not itself perform
- Surfacing a browsable history of past versions to the Pro -- excluded per feature-overview.md's Non-Goals: the product definition gives the Pro no version-history browser; superseded versions are retained only for internal conflict evaluation (this spec, FEAT-03)
- Purging superseded versions -- excluded per feature-overview.md's Non-Goals and scope-boundaries.md SC-22: versions are retained for as long as any booking may reference them, with no purge window

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Pro's save passes validation (weekly hours, default buffer, notice, horizon) | FEAT-02.SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) | Fires only after FEAT-02.SPEC-005 validation succeeds | Full set of entered field values: weekly_windows, default_buffer, minimum_booking_notice, booking_horizon |
| Pro's save passes validation (per-service buffer override) | FEAT-02.SPEC-002 (Per-Service Buffer Override) | Fires only after FEAT-02.SPEC-005 validation succeeds | The current Availability Rule's other fields (unchanged) plus the entered Service.buffer_override value(s) |

## Processing Logic

1. Receive the validated field values from the triggering screen (either the full Availability Rule field set from FEAT-02.SPEC-001, or the per-service override value(s) from FEAT-02.SPEC-002).
2. Read the current (latest-effective) Availability Rule version for this Pro Account, if one exists.
3. Construct the new version by combining the triggering screen's changed fields with every unchanged field carried forward from the current version (e.g., a FEAT-02.SPEC-002 save carries forward weekly_windows, default_buffer, minimum_booking_notice, and booking_horizon unchanged, adding only the new Service.buffer_override).
4. Set the new version's effective_from to the moment the save commits.
5. Write the new version as a wholly new, additional Availability Rule record -- the prior version is never modified or removed; it remains retrievable by its own effective_from.
6. Mark the newly written version as the Pro Account's current (latest-effective) Availability Rule.
7. Trigger FEAT-02.SPEC-004 (Confirmed Booking Conflict Flagging) against the newly written version.
8. Return the outcome (success or failure) to the triggering screen.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| First version created | No Availability Rule existed for this Pro Account before this save | A new Availability Rule record is written with effective_from set to now | Triggering screen shows its Success state ("Hours saved" / "Overrides saved") | FEAT-02.SPEC-001 or FEAT-02.SPEC-002 (triggering screen), FEAT-02.SPEC-004 (conflict check runs against the new version, finding nothing since no confirmed bookings can predate the account's first rule) |
| New version created | An Availability Rule already existed for this Pro Account | A new Availability Rule record is written with effective_from set to now; the prior version is retained unmodified and is no longer the current one | Triggering screen shows its Success state | FEAT-02.SPEC-001 or FEAT-02.SPEC-002, FEAT-02.SPEC-004 (conflict check runs against the new version) |
| Versioning failure | The write cannot be committed (e.g., a connectivity or processing failure after validation passed) | No new version is written; the prior version (if any) remains current and unchanged | Triggering screen shows its Error state with a retry action; the Pro's entered values remain on screen for retry | FEAT-02.SPEC-001 or FEAT-02.SPEC-002 (triggering screen shows the failure) |

## Data Model

**Reads:** Availability Rule -- the current (latest-effective) version's full field set, to carry forward any fields not changed by the triggering screen.
**Creates:** Availability Rule -- a new version with weekly_windows, default_buffer, minimum_booking_notice, booking_horizon, and effective_from; when triggered from FEAT-02.SPEC-002, the corresponding Service.buffer_override field is written as part of the same save (on the Service entity, per the flagged discrepancy noted in feature-overview.md).
**Updates:** None on the Availability Rule entity itself -- every change is a new version, never an in-place update. Marking a version as "current" is a pointer change, not a modification of any existing version's stored field values.
**Deletes:** None -- prior versions are never deleted, per scope-boundaries.md SC-22 and feature-overview.md's Non-Goals.

## Business Rules

- An Availability Rule is never overwritten in place -- every passing save, from either triggering screen, produces a wholly new, additional version (feature-overview.md's Side-Effect Inventory).
- Every prior version is retained indefinitely as long as any booking may reference it (dependency map: "never removed while past bookings reference it"; scope-boundaries.md SC-22).
- Versioning is unconditional on a passing save -- there is no "no-op" outcome where a validated save produces no new version, even when the entered values happen to match the current version exactly, so that effective_from always accurately reflects when the Pro last confirmed their hours.
- Versioning always triggers the conflict check (FEAT-02.SPEC-004) against confirmed bookings; a version is never left unchecked (XBR-11).
- Versioning itself never inspects or changes Booking records -- that is FEAT-02.SPEC-004's role, triggered as a direct consequence of this automation.

## Edge Cases

- **The Pro has never set an Availability Rule and saves for the first time** -- Step 2 finds no current version; the new version is created with no prior fields to carry forward, and FEAT-02.SPEC-004's conflict check against it trivially finds nothing (no confirmed bookings could exist before the account's first rule).
- **A FEAT-02.SPEC-002 save arrives when no Availability Rule exists yet** -- Cannot occur: FEAT-02.SPEC-002 is reached only from FEAT-02.SPEC-001, which requires an Availability Rule (even a freshly created one) to exist first; the per-service override screen has no independent entry point.
- **The triggering screen's entered values are identical to the current version's values** -- A new version is still created with a fresh effective_from, per the unconditional-versioning business rule above.
- **Versioning fails after validation already passed** -- The prior version remains current and unmodified; the triggering screen shows its Error state and the Pro's entered values are preserved for retry, so nothing is left in an inconsistent or partially-versioned state.
- **Concurrent trigger firing (Talia saves from FEAT-02.SPEC-001 on her phone and FEAT-02.SPEC-002 on a second device at nearly the same time)** -- Each save reads whatever version is current at that moment and creates its own new version from it; the save that commits second becomes the latest-effective version and its fields (carrying forward whatever it read as "current" at read time) take precedence, consistent with the dependency map's Contention note ("last-write-wins between the Pro's sessions"). The earlier save's version is retained in history but is no longer current.
- **Trigger fires while a previous run is in flight** -- A second run for the same Pro Account cannot start while the first is in flight: the triggering screens' Save controls are disabled while saving (FEAT-02.SPEC-001, FEAT-02.SPEC-002), so a Pro cannot fire two saves from the same screen session simultaneously. Runs from two different sessions (see the concurrent-trigger-firing entry above) proceed independently, each producing its own version.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-02.SPEC-001 (Working Hours, Buffer, Notice & Horizon Setup) | Triggered by (inbound), Affects (outbound) | Fires on a passing save; returns success/failure feedback to that screen |
| FEAT-02.SPEC-002 (Per-Service Buffer Override) | Triggered by (inbound), Affects (outbound) | Fires on a passing save; returns success/failure feedback to that screen |
| FEAT-02.SPEC-005 (Availability Setup Validation & Limits) | References (inbound) | This automation only runs after that spec's validation has already passed on the triggering screen |
| FEAT-02.SPEC-004 (Confirmed Booking Conflict Flagging) | Triggers (outbound) | Every successfully written new version triggers this conflict check |

## Analytics and Success Signals

- **availability_rule_version_created** (trigger source: FEAT-02.SPEC-001 or FEAT-02.SPEC-002; version sequence number for this account) -- supports success-metrics.md: "Availability Setup Accuracy"
- **availability_rule_versioning_failed** (trigger source, failure stage) -- supports success-metrics.md: "Availability Setup Accuracy" (a versioning failure means the Pro's intended hours never took effect, which is the exact accuracy gap this metric measures)

## Acceptance Criteria

**FEAT-02.SPEC-003-AC-01:** Given Talia has no Availability Rule yet, when she completes and saves the Working Hours screen (FEAT-02.SPEC-001) and validation passes, then a first Availability Rule version is created with effective_from set to the moment of save.

**FEAT-02.SPEC-003-AC-02:** Given Talia has an existing Availability Rule, when she edits her default buffer on FEAT-02.SPEC-001 and validation passes, then a new version is created carrying forward her existing weekly_windows, minimum_booking_notice, and booking_horizon unchanged, with only default_buffer updated.

**FEAT-02.SPEC-003-AC-03:** Given Talia has an existing Availability Rule, when she sets a per-service buffer override on FEAT-02.SPEC-002 and validation passes, then a new version is created that carries forward the existing weekly_windows, default_buffer, minimum_booking_notice, and booking_horizon unchanged, alongside the new Service.buffer_override.

**FEAT-02.SPEC-003-AC-04:** Given a new Availability Rule version has just been written, then FEAT-02.SPEC-004 is triggered against that version before this automation reports success to the triggering screen.

**FEAT-02.SPEC-003-AC-05:** Given Talia re-saves the exact same values already in her current Availability Rule version, when validation passes, then a new version is still created with a fresh effective_from.

**FEAT-02.SPEC-003-AC-06:** Given the write for a new version fails after validation passed, then the prior version remains current and unmodified, and the triggering screen shows its Error state with the Pro's entered values preserved.

**FEAT-02.SPEC-003-AC-07:** Given Talia saves from two sessions (phone and desktop) at nearly the same time, when both saves are validated independently, then the save that commits second becomes the latest-effective version, and the earlier save's version is retained in history but is no longer current.

**FEAT-02.SPEC-003-AC-08:** Given Talia's Working Hours screen has its Save button disabled while a save is in progress, then no second versioning run can start for that same in-flight save.

**FEAT-02.SPEC-003-AC-09:** Given every prior Availability Rule version for Talia's account, when a new version is created, then none of the prior versions are modified or removed.

**FEAT-02.SPEC-003-AC-10:** Given Talia's account has several superseded Availability Rule versions, then none of them are ever automatically purged, regardless of age.

**FEAT-02.SPEC-003-AC-11:** Given a new Availability Rule version is created for Talia's account for the first time (no prior version existed), then the conflict check (FEAT-02.SPEC-004) runs against it and finds no confirmed bookings to flag, since none could predate the account's first rule.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (FEAT-02.SPEC-001, FEAT-02.SPEC-002) | 2 |
| Outcome Paths | 3 (first version, new version, versioning failure) | 3 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
