---
document_type: spec
spec_type: automation
spec_id: FEAT-31.SPEC-004
spec_name: Support Session Auto-Close on Inactivity
spec_slug: support-session-auto-close-on-inactivity
parent_feature: FEAT-31
parent_feature_name: Operator Support Access
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 9
---

# Automation Spec: Support Session Auto-Close on Inactivity

## Overview

**Name:** Support Session Auto-Close on Inactivity
**ID:** FEAT-31.SPEC-004
**Type:** Automation
**Purpose:** Closes an open Support Access Session automatically after a period of inactivity, recording the close time and returning Dana to the queue.
**Parent Feature:** FEAT-31 -- Operator Support Access

## Scope and Non-Goals

**In Scope:**
- Tracking Dana's last action within an open session (any interaction on the mirrored view or console, per FEAT-31.SPEC-002 and FEAT-31.SPEC-003)
- Closing the session once inactivity reaches the defined threshold
- Recording `closed_at` and writing the trail entry
- Returning Dana to the queue with a notice

**Non-Goals:**
- Any manual or on-demand close action -- not established by any Stage 2 field; product-features.md's Validation & Limits states only that "a session ends automatically after a period of inactivity," so this is the sole closing path this feature defines (Non-Goals: "A manual or on-demand close control for Dana").
- Notifying Nadia by email when a session closes -- product-features.md's Communications field names only the request-confirmation (FEAT-31.SPEC-006) and session-opened (FEAT-31.SPEC-007) emails; the close time reaches her only through her activity trail (FEAT-13), not a third email.
- Deleting or archiving the closed Support Access Session -- excluded per the dependency map's lifecycle statement that the entity is "Never edited after closing" and "Deleted by FEAT-24" only.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Inactivity threshold reached on an open session | System (inactivity timer) | Fires when the elapsed time since Dana's last action within the open session reaches platform parameter: `support-session-inactivity-timeout-minutes` | The open Support Access Session's `operator`, `opened_at`, `freelancer_account`, and the timestamp of Dana's last recorded action |

## Processing Logic

1. Track the timestamp of Dana's last action within the currently open session (any interaction on the mirrored view, FEAT-31.SPEC-002, or the opening action itself, FEAT-31.SPEC-003).
2. Continuously evaluate the elapsed time since that last-action timestamp against platform parameter: `support-session-inactivity-timeout-minutes`.
3. If a new action is recorded before the threshold is reached, reset the last-action timestamp and continue the session uninterrupted.
4. When the elapsed time reaches the threshold with no intervening action, close the session: set `closed_at` to the current timestamp on the Support Access Session record.
4a. If the `closed_at` write fails, take no further step: `closed_at` stays unset, the session remains open and enforced read-only, and Dana is not moved (see Outcome Definitions, Close write failed).
5. Once `closed_at` is set, revoke the read-only mirrored-view access immediately -- the enforcement surface from FEAT-31.SPEC-003 stops applying because no session is open.
6. Return Dana to the Support Session Console queue (FEAT-31.SPEC-002) with a notice naming the freelancer whose session closed and stating the reason ("closed after inactivity").
7. Trigger FEAT-13.SPEC-003 (Activity Entry Recording) to write the session-closed trail entry, with Dana as actor, `closed_at` as the timestamp, and closure reason "inactivity."
7a. If the trail entry write fails, the close is not undone (the record is never edited after `closed_at` is set): the session stays closed and Dana stays in the queue, and the trail entry is re-attempted on each subsequent evaluation cycle until FEAT-13.SPEC-003 accepts it, writing exactly one session-closed entry per closure (see Outcome Definitions, Trail entry pending).

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Session closed | Elapsed inactivity reaches the threshold with no intervening action | `closed_at` set on the Support Access Session record | Dana is returned to the queue with a notice naming the freelancer and stating the session closed after inactivity | FEAT-31.SPEC-002, FEAT-13.SPEC-003 |
| No action (already closed) | A second evaluation cycle reaches the same session after it was already closed by an earlier cycle | None | None -- silent no-op | -- |
| Close write failed | The inactivity threshold was reached but the `closed_at` write failed | None -- `closed_at` stays unset; no trail entry is written | None shown to Dana: the session stays open and enforced read-only, the banner remains, and she is not returned to the queue. The close is re-evaluated on the next evaluation cycle and re-attempted if the inactivity gap still stands; an action by Dana before then resets the clock (No action, activity reset) | FEAT-31.SPEC-002 |
| Trail entry pending | `closed_at` was set but the FEAT-13.SPEC-003 session-closed trail entry could not be written | `closed_at` remains set (final, never edited); the trail entry is re-attempted on each subsequent evaluation cycle until accepted, one entry per closure | Dana is returned to the queue with the normal notice; the session is not enforced or shown as open. Nadia's trail lacks the closed entry until the retry succeeds | FEAT-31.SPEC-002, FEAT-13.SPEC-003 |
| No action (activity reset the clock) | An action was recorded before the threshold was reached | None -- the last-action timestamp resets | None -- the session continues uninterrupted | FEAT-31.SPEC-002 |

## Data Model

**Reads:** Support Access Session -- `operator`, `opened_at`, `freelancer_account`, and the last-action timestamp of the currently open record; also the `closed_at` of a closed record whose session-closed trail entry is still pending.
**Creates:** None.
**Updates:** Support Access Session -- `closed_at` (current timestamp).
**Deletes:** None.

## Business Rules

- XBR-29: sessions end after inactivity, with no manual close path.
- The inactivity threshold is platform parameter: `support-session-inactivity-timeout-minutes`, applied uniformly to every session regardless of freelancer account or request content.
- Once `closed_at` is set, the Support Access Session record is never edited again (dependency map, lifecycle note); this automation's write is the record's final state change short of account deletion (FEAT-24).
- Only one session can be open at a time (FEAT-31.SPEC-005), so this automation never needs to choose among multiple simultaneously open sessions for the same operator.

## Edge Cases

- **Dana performs an action at the exact moment the inactivity threshold is reached** -- The action recorded before this automation's close-evaluation completes resets the last-action timestamp and cancels the close for that cycle; only a confirmed inactivity gap of the full threshold, with no action recorded inside it, results in a close.
- **A freelancer account tied to an open session is deleted (FEAT-24) while the session is still open** -- FEAT-24 removes the Support Access Session as part of the full account deletion; this automation's close path is superseded and takes no further action on a record that no longer exists.
- **Concurrent trigger firing (two inactivity-evaluation cycles reach the same session at effectively the same time)** -- The evaluation is idempotent: whichever cycle completes first sets `closed_at`; the second finds `closed_at` already set and takes no further action.
- **Trigger fires while a previous run is in flight** -- Because at most one session can be open per operator at any time (FEAT-31.SPEC-005), and this automation only ever evaluates the single currently open session, no second run for the same session can be in flight concurrently with a first; a run that completes finds no session left to close on its next cycle.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-31.SPEC-003 (Support Session Open & Read-Only Enforcement) | References (inbound) | This automation closes only a session that FEAT-31.SPEC-003 opened |
| FEAT-31.SPEC-002 (Operator Support Session Console) | Affects (outbound) | Returns Dana to the queue with a notice |
| FEAT-31.SPEC-005 (Support Access Authorization & Read-Only Rules) | References (inbound) | The one-account-at-a-time constraint this automation relies on |
| FEAT-13.SPEC-003 (Activity Entry Recording) | Triggers (outbound) | Writes the session-closed trail entry with closure reason "inactivity" |

## Analytics and Success Signals

- **support_session_closed** (closure_reason: inactivity) -- N/A -- no metric in success-metrics.md is connected to Operator Support Access or names this behavior; retained per product-features.md's Signals field so support activity stays observable

## Acceptance Criteria

**FEAT-31.SPEC-004-AC-01:** Given Dana's open session has had no action for platform parameter: `support-session-inactivity-timeout-minutes`, when this automation evaluates it, then `closed_at` is set, Dana is returned to the queue with a notice naming the freelancer and stating the session closed after inactivity, and the trail entry is written with closure reason "inactivity."

**FEAT-31.SPEC-004-AC-02:** Given Dana performs an action one second before the inactivity threshold would be reached, when this automation next evaluates the session, then the last-action timestamp has reset and the session remains open.

**FEAT-31.SPEC-004-AC-03:** Given a session has already been closed by an earlier evaluation cycle, when a second evaluation cycle reaches the same session, then no further change is made and no duplicate notice or trail entry is produced.

**FEAT-31.SPEC-004-AC-04:** Given two inactivity-evaluation cycles reach the same open session at effectively the same time, when both attempt to close it, then only the first sets `closed_at` and the second finds it already closed.

**FEAT-31.SPEC-004-AC-05:** Given the freelancer account behind an open session is deleted (FEAT-24) while the session is still open, when the deletion completes, then this automation takes no further action on that now-removed record.

**FEAT-31.SPEC-004-AC-06:** Given Dana has just closed her previous session automatically, when she returns to the queue, then no other session for her remains open, so no second close-in-flight scenario can occur for her.

**FEAT-31.SPEC-004-AC-07:** Given the inactivity threshold is reached and the `closed_at` write fails, when this automation handles the failure, then `closed_at` stays unset, no trail entry is written, the session remains open and read-only with the banner still shown, and Dana is not returned to the queue; on the next evaluation cycle the close is re-attempted if she has still taken no action.

**FEAT-31.SPEC-004-AC-08:** Given `closed_at` has been set but the FEAT-13.SPEC-003 session-closed trail entry cannot be written, when this automation handles the failure, then the session stays closed, Dana is returned to the queue with the normal notice, and the trail entry is re-attempted on each later evaluation cycle until accepted, producing exactly one session-closed entry with closure reason "inactivity."

**FEAT-31.SPEC-004-AC-09:** Given a `closed_at` write failed and Dana performs an action before the next evaluation cycle, when this automation next evaluates the session, then the last-action timestamp has reset and the session stays open.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 1 | 1 |
| Outcome Paths | 5 (closed, already closed, activity reset, close write failed, trail entry pending) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 4 | 4 |
