---
document_type: spec
spec_type: automation
spec_id: FEAT-24.SPEC-005
spec_name: Legal Retention Purge
spec_slug: legal-retention-purge
parent_feature: FEAT-24
parent_feature_name: Data Export & Account Deletion
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Legal Retention Purge

## Overview

**Name:** Legal Retention Purge
**ID:** FEAT-24.SPEC-005
**Type:** Automation
**Purpose:** Purges the Invoice and Payment records held back from an otherwise-completed account deletion once their legal retention period lapses.
**Parent Feature:** FEAT-24 -- Data Export & Account Deletion

## Scope and Non-Goals

**In Scope:**
- Periodically checking every Invoice and Payment record held back under legal retention by a completed FEAT-24.SPEC-004 deletion
- Permanently purging each such record once its legal retention period elapses
- Leaving no restore path once a record is purged

**Non-Goals:**
- Determining that an Invoice or Payment must be retained in the first place -- owned by FEAT-24.SPEC-006 (Pre-Deletion Warning & Retention Determination Rules); this automation only acts on records FEAT-24.SPEC-004 has already flagged and held back.
- Purging Activity Log Entries subject to the same retention exception -- owned entirely by FEAT-13.SPEC-006 (Retention & Account-Deletion Purge Rule), which purges its own retained entries on the same elapsed-period condition independently of this automation, to avoid duplicating that ownership.
- Any purge not tied to a completed account deletion -- excluded per feature-overview.md's Non-Goals ("Retention of data beyond the legal financial-record requirement"); this automation never runs against an active (non-deleted) account.
- Notifying anyone when a purge completes -- product-features.md's Communications field for this feature names only the export-ready and deletion-final-warning emails; no account or person remains to notify once a deletion has already completed and this later purge runs.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A retained financial record's legal retention period elapses | System (scheduled sweep, interval: platform parameter: `legal-retention-purge-sweep-interval`) | Fires when the sweep finds an Invoice or Payment held back by a completed FEAT-24.SPEC-004 deletion whose retention period (platform parameter: `financial-record-legal-retention-period`), counted from that deletion's completion, has elapsed | The retained record's reference and the deletion-completion date it is measured from |

## Processing Logic

1. On each scheduled sweep, identify every Invoice and Payment record currently held in the retained, inaccessible state established by FEAT-24.SPEC-004.
2. For each such record, calculate the elapsed time since the account's deletion completion date (the date FEAT-24.SPEC-004 set the Freelancer Account to Deleted).
3. If the elapsed time is at or beyond platform parameter: `financial-record-legal-retention-period`, permanently purge that record. If it is not yet at the threshold, leave it untouched until a later sweep.
4. Confirm the purge before considering the record removed; a failed purge attempt is retried on the next sweep rather than reported as complete.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Record purged | Elapsed time meets or exceeds platform parameter: `financial-record-legal-retention-period` | The Invoice or Payment record is permanently removed, with no restore path | None -- the freelancer account no longer exists by this point | -- |
| Not yet due | Elapsed time is below the threshold | None | None | -- |
| Purge retried | The purge attempt itself fails (a processing error) | Record remains retained, untouched | None | -- |
| No-action (sweep finds nothing due) | No retained record has reached its threshold at this sweep | None | None | -- |

## Data Model

**Reads:** Invoice and Payment -- every record currently held in the retained, inaccessible state, plus the deletion-completion date each is measured against.
**Creates:** None.
**Updates:** None.
**Deletes:** Invoice and Payment -- each record once its retention period elapses, permanently and without a restore path.

## Business Rules

- The legal retention period is a platform-set policy value, referenced only as platform parameter: `financial-record-legal-retention-period` -- the same marker FEAT-24.SPEC-004 and FEAT-13.SPEC-006 use for the identical retention exception.
- This automation never runs against an active account -- it acts exclusively on records a completed FEAT-24.SPEC-004 deletion has already flagged and held back (SC-24).
- Once a record's retention period elapses, its purge is unconditional -- there is no further extension, hold, or manual review step (feature-overview.md's Non-Goals: "no retention of data beyond the legal financial-record requirement").
- Activity Log Entries subject to the same retention window are purged independently by FEAT-13.SPEC-006, not by this automation, avoiding duplicate ownership of one retention exception.

## Edge Cases

- **Concurrent sweep runs overlap (two scheduled sweeps fire close together)** -- A record already purged by one sweep is simply absent from the next sweep's candidate set; no error occurs from encountering an already-removed record.
- **Trigger fires while a previous sweep is still in flight** -- A new sweep for the same records waits for the first to finish rather than processing the same candidate set concurrently, avoiding two processes racing to purge the same record.
- **A record's retention period elapses between two sweep cycles rather than exactly at a sweep's run time** -- It is purged at the first sweep that runs at or after the threshold, per platform parameter: `legal-retention-purge-sweep-interval`, not at the exact expiry instant.
- **A purge attempt fails on its first try** -- The record remains retained and is reprocessed at the next sweep rather than being reported purged.
- **An account's deletion is later re-attempted after already completing (a stale retry reaching FEAT-24.SPEC-004)** -- No effect on this automation: FEAT-24.SPEC-004's own no-action outcome for an already-deleted account means the deletion-completion date this automation measures against never changes.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-24.SPEC-004 (Account Deletion Processing) | Triggered by (inbound) | Supplies the retained Invoice and Payment records and the deletion-completion date this automation measures against |
| FEAT-24.SPEC-006 (Pre-Deletion Warning & Retention Determination Rules) | References (inbound) | Defines which records qualify as retained in the first place |
| FEAT-13.SPEC-006 (Retention & Account-Deletion Purge Rule) | References (outbound) | Purges the equivalent, financial-character Activity Log Entries independently, under the same retention window |

## Analytics and Success Signals

- **financial_record_retention_purge_completed** (record_type: invoice / payment) -- N/A -- no success-metrics.md metric is connected to Data Export & Account Deletion; retained as the audit-relevant record that a legally required purge actually completed.
- **financial_record_retention_purge_retried** (record_type: invoice / payment) -- N/A -- same reason; retained so a purge that keeps failing across sweeps stays observable.

## Acceptance Criteria

**FEAT-24.SPEC-005-AC-01:** Given an Invoice was held back by FEAT-24.SPEC-004 and its retention period (platform parameter: `financial-record-legal-retention-period`) has elapsed since deletion completion, when a scheduled sweep runs, then that Invoice is permanently purged.

**FEAT-24.SPEC-005-AC-02:** Given a Payment held back by FEAT-24.SPEC-004 and its retention period has elapsed, when a scheduled sweep runs, then that Payment is permanently purged.

**FEAT-24.SPEC-005-AC-03:** Given a retained record's elapsed time is below the retention threshold, when a scheduled sweep runs, then the record is left untouched.

**FEAT-24.SPEC-005-AC-04:** Given a purge attempt fails on its first try, when the failure occurs, then the record remains retained and is reprocessed at the next sweep.

**FEAT-24.SPEC-005-AC-05:** Given a record's retention period elapses between two scheduled sweeps, when the next sweep runs at or after the threshold, then the record is purged at that sweep, not exactly at the expiry instant.

**FEAT-24.SPEC-005-AC-06:** Given two scheduled sweeps fire close together, when the second sweep encounters a record the first sweep already purged, then no error occurs and the record is simply absent from the candidate set.

**FEAT-24.SPEC-005-AC-07:** Given a sweep is already in flight, when a new sweep is triggered before the first finishes, then the new sweep waits rather than processing the same candidates concurrently.

**FEAT-24.SPEC-005-AC-08:** Given no retained record has reached its threshold at a given sweep, when the sweep runs, then nothing changes.

**FEAT-24.SPEC-005-AC-09:** Given a retained record is purged, when the purge completes, then no restore path exists for it.

**FEAT-24.SPEC-005-AC-10:** Given Activity Log Entries under the same retention exception exist, when this automation runs, then it purges only Invoice and Payment records, leaving those entries to FEAT-13.SPEC-006's own purge.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 4 | 4 |
| Business Rules | 4 | 4 |
| Edge Cases | 5 | 5 |
