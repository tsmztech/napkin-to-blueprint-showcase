---
document_type: spec
spec_type: automation
spec_id: FEAT-27.SPEC-010
spec_name: Booking Link Forwarding & Reservation Expiry
spec_slug: booking-link-forwarding-reservation-expiry
parent_feature: FEAT-27
parent_feature_name: Pro Profile & Booking Page Settings
priority_tier: Important
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Booking Link Forwarding & Reservation Expiry

## Overview

**Name:** Booking Link Forwarding & Reservation Expiry
**ID:** FEAT-27.SPEC-010
**Type:** Automation
**Purpose:** On a booking-link rename, keeps the old link name forwarding to the new one for at least platform parameter: `booking-link-forward-window-months`, reserving it from reuse by any other pro until that window lapses, and then releases the reservation.
**Parent Feature:** FEAT-27 -- Pro Profile & Booking Page Settings

## Scope and Non-Goals

**In Scope:**
- Recording the outgoing name as a forwarding entry the moment a rename is accepted
- Keeping that entry resolvable to the Pro's current booking_link_name for at least the forwarding window
- Releasing the reservation (making the name available to other pros) once the window lapses

**Non-Goals:**
- Deciding whether a candidate new name is valid or available -- owned by FEAT-27.SPEC-007 (Booking Link Name Validation & Uniqueness Rule); this automation only runs after that spec has already accepted the rename
- Resolving a visitor's request against the forwarding table on the public booking page -- owned by FEAT-05.SPEC-001 and FEAT-05.SPEC-008; this automation only maintains the record those specs read
- Notifying Talia that her old link is still working -- the rename screen's own success feedback (FEAT-27.SPEC-002) states the forwarding guarantee at save time; no separate notification is defined for this ongoing state

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Booking link rename accepted | FEAT-27.SPEC-002 (Booking Link Rename), via FEAT-27.SPEC-007's acceptance | Fires every time a rename passes FEAT-27.SPEC-007's format, length, and uniqueness checks and is saved | The outgoing booking_link_name, the new booking_link_name, the Pro Account reference, and the current timestamp |
| Forwarding-reservation window lapses | Schedule-based (system clock) | Fires once per forwarding entry, at least platform parameter: `booking-link-forward-window-months` after that entry's forwarding-start timestamp | The forwarding entry's outgoing name, its Pro Account reference, and its forwarding-start timestamp |

## Processing Logic

1. On a rename acceptance: record a new forwarding entry for the outgoing name, storing the outgoing name, the current (new) booking_link_name it should resolve to, the Pro Account reference, and the forwarding-start timestamp (the moment of this rename).
2. While a forwarding entry is within its window: any lookup by the outgoing name (from FEAT-05.SPEC-001 or FEAT-05.SPEC-008) resolves to the Pro Account's current booking_link_name at the time of the lookup -- not necessarily the name recorded at forwarding-start, since a Pro may rename more than once.
3. If the Pro renames again while an earlier forwarding entry is still active, chain resolution continues to work: each outgoing name always resolves forward to whatever the Pro's booking_link_name is at lookup time, not to the immediately-next name in the chain.
4. Continuously (or on each scheduled evaluation), check every forwarding entry's age against the forwarding window (platform parameter: `booking-link-forward-window-months`).
5. When a forwarding entry's age meets or exceeds the window, release its reservation: the outgoing name becomes available for use by any other pro (per FEAT-27.SPEC-007's uniqueness check), and it stops resolving on the public booking page.
6. A released name remains available to the Pro who originally owned it (per FEAT-27.SPEC-007's own-name reclaim exception) exactly as it would be to any other pro, once released -- release does not distinguish original ownership.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Forwarding entry created | A rename is accepted | New forwarding entry recorded for the outgoing name | None from this automation directly -- FEAT-27.SPEC-002's own save-success dialog already states the guarantee | FEAT-27.SPEC-002 (Booking Link Rename), FEAT-05.SPEC-001 / FEAT-05.SPEC-008 (resolve forwarded links) |
| Forwarding entry resolved (client visits old name) | A client visits a link inside its forwarding window | None -- read-only lookup | Client is forwarded transparently with no visible difference (XBR-27) | FEAT-05.SPEC-001, FEAT-05.SPEC-008 |
| Reservation released | A forwarding entry's age reaches the window | The outgoing name is removed from the active-reservation set; it stops resolving | None -- silent, since no client should be visiting a name this old, and the Pro receives no notification for this event (per Non-Goals) | FEAT-27.SPEC-007 (uniqueness check now sees the name as available) |
| No-op (automation runs, no entries due) | Scheduled evaluation finds no forwarding entry at or past its window | None | None | -- |
| Automation failure (release evaluation cannot complete) | A processing error prevents the scheduled release check from completing | No forwarding entries are released for that run; entries remain reserved (fail-safe, not fail-open) | None -- reservation continuing slightly longer than the minimum window is never client-visible or harmful, since the guarantee is "at least" the window, not "exactly" | -- |

## Data Model

**Reads:** Pro Account -- booking_link_name (current value, to resolve forwarding), previous_link_names / forwarding entries.
**Creates:** A forwarding entry per accepted rename -- outgoing name, Pro Account reference, forwarding-start timestamp.
**Updates:** Forwarding entry -- marked released once its window lapses.
**Deletes:** None -- a released entry is marked inactive, not erased, preserving the historical record of the Pro's own previous_link_names.

## Business Rules

- XBR-27: a renamed booking link keeps forwarding from the old name for at least platform parameter: `booking-link-forward-window-months`; a closed, paused-to-closure, or mistyped link shows the plain "this booking page isn't available" message, never another pro's page.
- The window is a floor, not a ceiling -- "at least" the stated duration, per BRIEF.md's and the Brief's own wording; a release evaluation failure that delays release past the window causes no incorrect behavior, since the guarantee is never violated by forwarding for longer.
- A forwarding entry always resolves to the Pro's current booking_link_name at lookup time, never to a stale intermediate name, even across multiple renames of the same account.
- Release makes a name available to any pro, including its original owner, on equal footing with any other candidate -- no owner-priority survives release.

## Edge Cases

- **A client visits an old link name the instant its forwarding window lapses** -- Whichever completes first is authoritative: if the release evaluation has already marked the entry inactive, the client sees the "this booking page isn't available" message (FEAT-05.SPEC-008); if the lookup completes microseconds before release, the client is forwarded normally. Either outcome is correct, since the window is a floor.
- **Talia renames her link twice within the same forwarding window** -- Two independent forwarding entries exist (one per outgoing name), each with its own forwarding-start timestamp and its own release time; both resolve forward to Talia's current booking_link_name until each individually lapses.
- **A forwarding entry's Pro Account is closed (FEAT-29) while the entry is still within its window** -- The forwarding entry itself is unaffected by this automation; FEAT-05.SPEC-008 separately renders the account's closed state to any visitor, including one arriving via a still-active forwarding entry, consistent with XBR-27's "closed... link shows the plain message" behavior.
- **Concurrent trigger firing (two renames on two different Pro Accounts at effectively the same time)** -- Each rename creates its own independent forwarding entry against its own account; there is no shared state between different pros' forwarding entries, so no conflict is possible between them.
- **Trigger fires while a previous run is in flight (the scheduled release evaluation is still processing when its next scheduled run would fire)** -- The next scheduled run for the same evaluation is skipped until the in-flight run completes, since re-running against partially-updated state could double-process the same entries; the following scheduled run picks up any entries the skipped run would have caught, and the "at least" floor guarantee is preserved regardless.
- **The release evaluation processes the same forwarding entry twice due to a retry after a partial failure** -- Marking an already-released entry as released again is a no-op; no duplicate release, no error.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-27.SPEC-002 (Booking Link Rename) | Triggered by (inbound) | An accepted rename save fires this automation |
| FEAT-27.SPEC-007 (Booking Link Name Validation & Uniqueness Rule) | Triggered by (inbound) / Affects (outbound) | Only a rename FEAT-27.SPEC-007 accepts reaches this automation; the release outcome feeds back into FEAT-27.SPEC-007's uniqueness check |
| FEAT-05.SPEC-001 (Public Booking Page & Booking Flow, landing/service list) | Affects (outbound) | Resolves forwarded links using this automation's active entries |
| FEAT-05.SPEC-008 (Booking Page Availability Gate) | Affects (outbound) | Resolves forwarded links, and renders the "isn't available" message for a lapsed or otherwise invalid link name |

## Analytics and Success Signals

- **booking_link_forward_created** (has_prior_forward: yes/no) -- N/A -- no success-metrics.md metric is connected to Pro Profile & Booking Page Settings; retained so forwarding-entry creation is observable
- **booking_link_reservation_released** (age_at_release_days) -- N/A -- no connected success-metrics.md metric; retained so the reservation lifecycle is observable rather than invisible

## Acceptance Criteria

**FEAT-27.SPEC-010-AC-01:** Given Talia renames her link from "talia-lashes" to "talia-beauty" and FEAT-27.SPEC-007 accepts it, when the rename saves, then a forwarding entry for "talia-lashes" is created, recording the current timestamp as its forwarding-start.

**FEAT-27.SPEC-010-AC-02:** Given a client visits "talia-lashes" within its forwarding window, when the lookup resolves, then the client is forwarded transparently to Talia's current booking page with no visible difference.

**FEAT-27.SPEC-010-AC-03:** Given Talia renames again from "talia-beauty" to "talia-studio" while "talia-lashes" is still within its own forwarding window, when a client visits "talia-lashes", then it resolves to Talia's current name, "talia-studio" -- not to the intermediate "talia-beauty".

**FEAT-27.SPEC-010-AC-04:** Given a forwarding entry's age reaches platform parameter: `booking-link-forward-window-months`, when the scheduled release evaluation runs, then the entry is marked released and the name becomes available to any pro via FEAT-27.SPEC-007's uniqueness check.

**FEAT-27.SPEC-010-AC-05:** Given a forwarding entry has just been released, when a client attempts to visit that name, then FEAT-05.SPEC-008 shows "this booking page isn't available" rather than resolving to any pro's page.

**FEAT-27.SPEC-010-AC-06:** Given a forwarding entry's original owner later wants that same name back after release, when she checks its availability via FEAT-27.SPEC-002, then it shows as available to her on the same footing as to any other pro.

**FEAT-27.SPEC-010-AC-07:** Given the scheduled release evaluation encounters a processing error mid-run, when the run fails, then no forwarding entries are released for that run, and the next scheduled run re-evaluates them.

**FEAT-27.SPEC-010-AC-08:** Given two different pros each rename their links at effectively the same time, when both renames are processed, then each creates its own independent forwarding entry with no interaction between them.

**FEAT-27.SPEC-010-AC-09:** Given a Pro Account with an active forwarding entry is closed via FEAT-29 while the entry is still within its window, when a client visits the old name, then FEAT-05.SPEC-008 shows the account's closed-state message, not a normal booking page.

**FEAT-27.SPEC-010-AC-10:** Given the release evaluation is re-run against an already-released forwarding entry (for example, after a retried partial failure), when it processes that entry again, then marking it released a second time has no effect and produces no error.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (rename accepted, window lapses) | 2 |
| Outcome Paths | 5 (entry created, resolved, released, no-op, failure) | 5 |
| Business Rules | 4 | 4 |
| Edge Cases | 6 | 6 |
