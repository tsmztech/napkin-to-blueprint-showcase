---
document_type: spec
spec_type: automation
spec_id: FEAT-20.SPEC-007
spec_name: Waitlist Entry Expiry
spec_slug: waitlist-entry-expiry
parent_feature: FEAT-20
parent_feature_name: Waitlist for Cancelled Slots
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 13
---

# Automation Spec: Waitlist Entry Expiry

## Overview

**Name:** Waitlist Entry Expiry
**ID:** FEAT-20.SPEC-007
**Type:** Automation
**Purpose:** Expires a Notified entry whose 30-minute claim window lapses unclaimed, and separately expires a Requested entry whose joined date range elapses with no matching opening ever found.
**Parent Feature:** FEAT-20 -- Waitlist for Cancelled Slots

## Scope and Non-Goals

**In Scope:**
- Detecting a Notified entry whose claim_deadline has passed without converting
- Detecting a Requested entry whose end_date has passed with no matching opening ever found
- Transitioning each to Expired and handing off to the expiry notification

**Non-Goals:**
- Converting a claim that completes within the window -- owned by FEAT-20.SPEC-006; this automation only handles the unclaimed-lapse case
- Deciding what makes an entry match a freed slot in the first place -- owned by FEAT-20.SPEC-004/SPEC-005; this automation only observes that no match ever occurred before the range elapsed
- Composing or sending the expiry notification's content -- owned by FEAT-20.SPEC-009; this automation only triggers it

## Trigger Definition

| Trigger | Category | Source Spec | Conditions | Available Data |
|---------|----------|------------|------------|----------------|
| A Notified entry's claim_deadline passes | Schedule-based | system (evaluated continuously against each Notified entry's own claim_deadline) | Fires the moment the current time reaches a Notified entry's claim_deadline without that entry having converted (FEAT-20.SPEC-006) | Waitlist Entry reference, service, matched slot detail, claim_deadline |
| A Requested entry's end_date passes | Schedule-based | system (evaluated continuously against each Requested entry's own end_date) | Fires the moment the current time passes the end of a Requested entry's joined date range without it ever having been matched (FEAT-20.SPEC-005) | Waitlist Entry reference, service, start_date, end_date |

## Processing Logic

1. **Claim-window path:** Identify every Waitlist Entry in state Notified whose claim_deadline has passed.
2. Re-confirm each identified entry has not converted in the interval since claim_deadline was checked (avoiding a race with FEAT-20.SPEC-006 processing a last-second claim).
3. Transition each confirmed entry's state to Expired.
4. Trigger FEAT-20.SPEC-009 (Waitlist Expiry Notification) for each, with the reason "unclaimed opening."
5. **Unmatched-range path:** Identify every Waitlist Entry in state Requested whose end_date has passed.
6. Transition each identified entry's state to Expired.
7. Trigger FEAT-20.SPEC-009 for each, with the reason "unmatched date range."
8. Both paths run independently and on their own schedule; an entry is only ever processed by the one path that applies to its current state at the moment of evaluation.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Notified entry expired (unclaimed) | claim_deadline passed without conversion | Waitlist Entry: state -> Expired | Client receives the expiry notification (FEAT-20.SPEC-009) stating the opening was unclaimed; entry shows "Expired" on FEAT-20.SPEC-002 for one visit, per that screen's roll-off rule | FEAT-20.SPEC-009, FEAT-20.SPEC-002 |
| Requested entry expired (unmatched range) | end_date passed with no match ever found | Waitlist Entry: state -> Expired | Client receives the expiry notification stating no matching opening was ever found; entry shows "Expired" on FEAT-20.SPEC-002 for one visit | FEAT-20.SPEC-009, FEAT-20.SPEC-002 |
| No entries due for expiry | Neither condition is met for any entry at the current evaluation moment | None | None | -- |
| Automation failure (the expiry write itself fails) | A transient failure prevents the state write from completing | No change to the affected entry | The entry remains in its prior state (Notified past its window, or Requested past its range) until the next successful evaluation pass re-attempts the same transition | FEAT-20.SPEC-002 |

## Data Model

**Reads:** Waitlist Entry -- state, claim_deadline, start_date, end_date, across all entries.
**Creates:** None.
**Updates:** Waitlist Entry -- state (Notified -> Expired, or Requested -> Expired).
**Deletes:** None -- Expired entries are retained per this Brief's Non-Goals (no automatic purge), not deleted.

## Business Rules

- The claim window is fixed at platform parameter: `waitlist-claim-window-minutes` and never extended for any reason (FEAT-20.SPEC-004) -- an entry that reaches claim_deadline unclaimed expires without exception.
- A Requested entry's own joined date range is its natural lifetime bound: once end_date passes with no match ever found, the entry has no further chance to match (the range it asked about is now in the past) and expires.
- Both expiry paths are mutually exclusive per entry at any given moment: an entry is either Requested (subject to the unmatched-range path) or Notified (subject to the claim-window path), never both at once, since matching (FEAT-20.SPEC-005) is the only transition from Requested to Notified.
- Every expiry, from either path, triggers the expiry notification (FEAT-20.SPEC-009) -- an entry is never silently retired without informing the client, per this Brief's own Alternate flow: "the client is informed rather than left wondering indefinitely."
- Platform Operator (Support) can view an Expired entry and which path produced it through FEAT-19's read-only account view, per XBR-24, useful when a client reports "I never heard back"; Support never reactivates or extends an expired entry.

## Edge Cases

- **A Notified entry's claim_deadline passes at the exact instant FEAT-20.SPEC-006 is processing a completed Booking for it** -- The re-confirmation step (Processing Logic step 2) checks for a conversion that may have just landed; if the conversion already committed, this automation takes no action on that entry, since FEAT-20.SPEC-006's transition to Converted takes precedence over a late-arriving expiry evaluation for the same entry.
- **A Requested entry's range elapses on the same day a matching opening would have appeared, but the cancellation that would have produced it never occurs** -- The entry expires exactly as any other unmatched-range entry; the automation has no way to know a "near miss" almost happened and does not treat it differently.
- **A range-joined entry's end_date passes while the entry is mid-match (a freed-slot signal is being evaluated for it by FEAT-20.SPEC-005 at the same moment)** -- Whichever transition commits first wins: if FEAT-20.SPEC-005 already transitioned the entry to Notified before this automation's unmatched-range check runs, the entry is no longer Requested and this automation's unmatched-range path does not apply to it (it is now subject to the claim-window path instead, on its own new timeline).
- **Two different Notified entries reach their independent claim_deadlines at effectively the same time** -- Each is expired independently; neither is delayed by the other.
- **Concurrent trigger firing (the claim-window path and the unmatched-range path evaluate at the same moment across many entries)** -- Both paths run independently across the full set of entries each governs; there is no shared lock between them since they never target the same entry at the same time (per the mutual-exclusivity business rule above).
- **Trigger fires while a previous evaluation pass is still processing the same entry** -- Not possible in practice: an entry transitions to Expired at most once (a terminal state), so a second evaluation of an already-Expired entry finds it no longer eligible for either path and takes no action.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-20.SPEC-004 (Waitlist Priority & Claim Window Rule) | References (outbound) | Supplies the fixed claim-window value this automation checks against |
| FEAT-20.SPEC-005 (Cancellation-Triggered Waitlist Matching) | References (outbound) | The transition to Notified this automation's claim-window path watches for expiry against |
| FEAT-20.SPEC-006 (Waitlist Claim Conversion) | References (outbound) | The competing transition this automation's re-confirmation step checks for before expiring a Notified entry |
| FEAT-20.SPEC-009 (Waitlist Expiry Notification) | Triggers (outbound) | Fires once per expired entry, with the applicable reason |
| FEAT-20.SPEC-002 (My Waitlists) | Affects (outbound) | The Expired status is what that screen's next load (and one-visit roll-off) reflects |

## Analytics and Success Signals

- **waitlist_expired** (service ID, reason: unclaimed_opening / unmatched_range) -- N/A -- no success-metrics.md metric directly measures waitlist expiry; retained per the Brief's Non-Functional Notes naming waitlist_expired as one of the four Stage 2 signals this feature's transitions must emit.

## Acceptance Criteria

**FEAT-20.SPEC-007-AC-01:** Given Riley's Notified entry's claim window lapses without her completing a booking, when this automation evaluates it, then her entry transitions to Expired and FEAT-20.SPEC-009 is triggered with reason "unclaimed opening."

**FEAT-20.SPEC-007-AC-02:** Given Riley's Requested entry's joined date range elapses with no matching opening ever found, when this automation evaluates it, then her entry transitions to Expired and FEAT-20.SPEC-009 is triggered with reason "unmatched date range."

**FEAT-20.SPEC-007-AC-03:** Given Riley's Notified entry converts to a Booking just before this automation's claim-window check runs, when the re-confirmation step executes, then no expiry is recorded, since the conversion already committed.

**FEAT-20.SPEC-007-AC-04:** Given Riley's Requested entry transitions to Notified via FEAT-20.SPEC-005 at the same moment its end_date would otherwise trigger the unmatched-range path, when this automation evaluates it, then the unmatched-range path does not apply, and the entry is instead subject to the claim-window path on its new timeline.

**FEAT-20.SPEC-007-AC-05:** Given no entries are due for expiry at the current evaluation moment, when this automation runs, then no entry is transitioned and no notification is triggered.

**FEAT-20.SPEC-007-AC-06:** Given the expiry write fails for an entry due to a transient error, when the failure occurs, then the entry remains in its prior state until the next successful evaluation pass.

**FEAT-20.SPEC-007-AC-07:** Given an entry has already expired, when a later evaluation pass considers it again, then it is no longer eligible for either expiry path and no further action is taken.

**FEAT-20.SPEC-007-AC-08:** Given two different Notified entries reach their independent claim_deadlines at effectively the same time, when this automation processes both, then each expires independently.

**FEAT-20.SPEC-007-AC-09:** Given Riley's Expired entry, when she next views FEAT-20.SPEC-002, then it shows "Expired" for one visit before rolling off the list on a subsequent visit.

**FEAT-20.SPEC-007-AC-10:** Given an entry is Requested (never yet matched), when this automation evaluates it, then it is only ever subject to the unmatched-range path, never the claim-window path.

**FEAT-20.SPEC-007-AC-11:** Given an entry is Notified (already matched), when this automation evaluates it, then it is only ever subject to the claim-window path, never the unmatched-range path.

**FEAT-20.SPEC-007-AC-12:** Given every expiry this automation produces, when the transition completes, then FEAT-20.SPEC-009 is triggered without exception -- no entry expires silently.

**FEAT-20.SPEC-007-AC-13:** Given an Expired entry, when anyone looks for a way to reactivate or restore it, then no such control exists anywhere in the product -- a client who wants back on the waitlist rejoins fresh through FEAT-20.SPEC-001.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 | 2 |
| Outcome Paths | 4 | 4 |
| Business Rules | 5 | 5 |
| Edge Cases | 6 | 6 |
