---
document_type: spec
spec_type: automation
spec_id: FEAT-29.SPEC-003
spec_name: Mark Item Read/Unread
spec_slug: mark-item-read-unread
parent_feature: FEAT-29
parent_feature_name: In-App Notification Center
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 10
---

# Automation Spec: Mark Item Read/Unread

## Overview

**Name:** Mark Item Read/Unread
**ID:** FEAT-29.SPEC-003
**Type:** Automation
**Purpose:** Records a feed item's read/unread state when Nadia opens it or explicitly triages it, without altering the underlying Notification or Activity Log Entry.
**Parent Feature:** FEAT-29 -- In-App Notification Center

## Scope and Non-Goals

**In Scope:**
- Flipping a Feed Item Read State's read_status and read_at when Nadia opens an item or taps its explicit toggle
- Success, no-op, and failure feedback for that write

**Non-Goals:**
- Composing or sending new notifications -- owned entirely by FEAT-14 (XBR-30); this automation only updates a local read/unread marker on a record FEAT-14 already sent.
- Deleting feed items -- excluded per product-features.md, Validation & Limits: "items can be marked read/unread but not deleted."
- Altering the underlying Notification or Activity Log Entry -- excluded per XBR-04 and ASMP-15 (both are evidentiary and immutable); this automation only ever writes the feature-local Feed Item Read State (FEAT-29.SPEC-002).
- Deciding which events are in the feed or their order -- owned entirely by FEAT-29.SPEC-002; this automation only changes the read/unread field of a record that composition rule already created.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| Nadia opens a feed item (taps the row to navigate) | FEAT-29.SPEC-001 (Notification Center Feed) | Fires on every open tap, regardless of current read_status, EXCEPT while FEAT-29.SPEC-001 is in the Offline/Degraded state (item-open controls are inert there per FEAT-29.SPEC-004, so no tap reaches this automation) or after Nadia's session has expired | Item reference, current read_status |
| Nadia taps the explicit read/unread toggle on a feed item | FEAT-29.SPEC-001 (Notification Center Feed) | Fires on every toggle tap, regardless of current read_status, EXCEPT while FEAT-29.SPEC-001 is in the Offline/Degraded state (toggle controls are inert there per FEAT-29.SPEC-004, so no tap reaches this automation) or after Nadia's session has expired. It does fire in the Populated state and in the Error (fallback) state when saved feed items are shown | Item reference, current read_status |

## Processing Logic

1. Receive the feed item's reference and its current read_status from the triggering interaction.
2. Determine the target state: opening an item always targets Read; the explicit toggle targets the opposite of the item's current state.
3. If the target state equals the current state (opening an item that is already Read), skip the write and proceed directly to the no-op outcome.
4. Otherwise, write the new read_status to the item's Feed Item Read State record, and set read_at to the current moment if the new state is Read, or clear read_at if the new state is Unread (per FEAT-29.SPEC-002's field rules).
5. Confirm the write succeeded and signal the triggering screen to display the updated indicator.
6. If the write does not succeed, signal the triggering screen to revert the indicator to its prior state and show the failure toast "Couldn't update -- try again." The toast is a transient message displayed over whichever screen is showing when the failure is reported. For an open-triggered run, the failure never blocks or cancels the navigation that the open tap started: Nadia still reaches the project (FEAT-29.SPEC-001), the toast appears over the project screen if navigation has already happened, and the item shows Unread when she returns to the feed. For a toggle-triggered run, Nadia is still on the feed, so the toast appears over the feed and the toggle returns to its prior state.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Marked Read (from open) | Item was Unread and Nadia opened it | read_status -> Read, read_at set | Indicator updates to Read before navigation completes | FEAT-29.SPEC-001 |
| Marked Read (from toggle) | Item was Unread and Nadia tapped the toggle | read_status -> Read, read_at set | Indicator updates to Read; screen remains on the feed | FEAT-29.SPEC-001 |
| Marked Unread (from toggle) | Item was Read and Nadia tapped the toggle | read_status -> Unread, read_at cleared | Indicator updates to Unread | FEAT-29.SPEC-001 |
| No-op (already Read, opened) | Item was already Read and Nadia opened it | None | No visible indicator change; navigation proceeds | FEAT-29.SPEC-001 |
| Failure (from toggle) | The write does not complete after Nadia tapped the toggle | None persists | Toggle and indicator revert to their prior state; toast "Couldn't update -- try again." over the feed; Nadia stays on the feed | FEAT-29.SPEC-001 |
| Failure (from open) | The write does not complete after Nadia tapped a feed item row to open it | None persists | Navigation to the project proceeds unaffected; the item's indicator reverts to Unread; toast "Couldn't update -- try again." over the screen showing at the time of failure (the project screen if navigation already completed) | FEAT-29.SPEC-001 |

## Data Model

**Reads:** Feed Item Read State -- reference and current read_status of the target item.
**Creates:** None -- the record already exists by the time this automation runs; it was created implicitly by FEAT-29.SPEC-002 when the item first entered the rolling window.
**Updates:** Feed Item Read State -- read_status and read_at, per FEAT-29.SPEC-002's field rules.
**Deletes:** None.

## Business Rules

- The underlying Notification or Activity Log Entry a feed item references is never written by this automation -- only the feature-local Feed Item Read State changes (XBR-04, ASMP-15).
- read_status and read_at are always written together in the same operation (FEAT-29.SPEC-002, Cross-Field Rules) -- no intermediate state is ever observable.
- This automation is the only writer of Feed Item Read State's read_status/read_at fields in the product (FEAT-29.SPEC-002, Authorization Rules).

## Edge Cases

- **Opening an item that is already Read** -- No-op per the Outcome Definitions table; navigation still proceeds normally.
- **Nadia taps the toggle on the same item twice in rapid succession** -- The second tap is ignored while the first run for that specific item is in flight; the item's toggle control is disabled for the duration of the in-flight write (FEAT-29.SPEC-001).
- **Concurrent trigger firing (two of Nadia's own sessions act on the same item at effectively the same time)** -- Each run writes independently; the run that completes last determines the item's final read_status (last-write-wins). Feed Item Read State has no contending writer other than Nadia's own concurrent sessions (per FEAT-29.SPEC-002's Authorization Rules), so no conflict message is shown.
- **Trigger fires while a previous run for a different item is in flight** -- Runs for different items proceed independently; there is no queuing between items.
- **Nadia loses connectivity while a run is in flight, or a control is tapped while the screen is Offline/Degraded** -- A tap made while the screen is already Offline/Degraded never fires this automation (the controls are inert, per FEAT-29.SPEC-004). A run already in flight when connectivity drops that does not complete follows the Failure outcome for its trigger path.
- **The write completes successfully but the confirmation signal is delayed or lost** -- If a later run for the same item finds the record already matches the intended target state, no further write or visible change occurs; the screen is not falsely reverted.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-29.SPEC-001 (Notification Center Feed) | Triggered by (inbound) | Item open or explicit toggle fires this automation |
| FEAT-29.SPEC-001 (Notification Center Feed) | Affects (outbound) | Returns the updated (or reverted) read/unread indicator to the screen |
| FEAT-29.SPEC-002 (Feed Composition & Retention Rule) | References (inbound) | Field rules and authorization governing read_status/read_at |

## Analytics and Success Signals

- **notification_marked_read** (trigger: open / explicit_toggle; previous_state: Read / Unread) -- N/A — no metric in success-metrics.md names In-App Notification Center as its Connected Feature (verified against every metric entry's Connected Feature field); recorded here as the Signal product-features.md declares for this feature (notification_marked_read).
- **notification_mark_read_failed** (trigger: open / explicit_toggle; reason: write_failed) -- N/A — same reason; this event exists only to observe how often the failure path (Outcome Definitions, Failure) is exercised, not to feed a Stage 2 metric.

## Acceptance Criteria

**FEAT-29.SPEC-003-AC-01:** Given Nadia has an unread feed item, when she taps it to open it, then its read_status updates to Read, read_at is set, and she is navigated to the item's related project.

**FEAT-29.SPEC-003-AC-02:** Given Nadia has an unread feed item, when she taps its explicit read/unread toggle without opening it, then its read_status updates to Read and read_at is set, and she remains on the feed screen.

**FEAT-29.SPEC-003-AC-03:** Given Nadia has a read feed item, when she taps its explicit toggle, then its read_status updates to Unread and read_at is cleared.

**FEAT-29.SPEC-003-AC-04:** Given Nadia has an already-read feed item, when she opens it, then no write occurs and she is navigated to the related project without any indicator change.

**FEAT-29.SPEC-003-AC-05:** Given the read-state write fails, when Nadia taps an item's toggle, then the indicator reverts to its prior state and the message "Couldn't update -- try again." appears.

**FEAT-29.SPEC-003-AC-06:** Given Nadia taps the same item's toggle twice in rapid succession, then the second tap is ignored while the first write is in flight.

**FEAT-29.SPEC-003-AC-07:** Given two of Nadia's own sessions mark the same item at effectively the same time, then the item's final read_status is whichever write completes last, with no conflict message shown.

**FEAT-29.SPEC-003-AC-08:** Given Nadia triggers this automation for two different items at once, then each run completes independently with no queuing between them.

**FEAT-29.SPEC-003-AC-09:** Given the read-state write fails, when Nadia taps an unread feed item to open it, then she is still navigated to the item's related project, the item's indicator reverts to Unread, and the toast "Couldn't update -- try again." appears over the screen showing at the time of failure.

**FEAT-29.SPEC-003-AC-10:** Given the feed screen is in the Offline/Degraded state, when Nadia taps a feed item row or its toggle, then this automation does not fire, no write is attempted, and no failure toast appears.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (open, explicit toggle) | 2 |
| Outcome Paths | 6 (marked read from open, marked read from toggle, marked unread, no-op, failure from toggle, failure from open) | 6 |
| Business Rules | 3 | 3 |
| Edge Cases | 6 | 6 |
