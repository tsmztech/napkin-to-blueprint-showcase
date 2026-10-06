---
document_type: spec
spec_type: automation
spec_id: FEAT-07.SPEC-008
spec_name: Offline Comment Queue & Sync
spec_slug: offline-comment-queue-sync
parent_feature: FEAT-07
parent_feature_name: Deliverable Review & Feedback
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 12
---

# Automation Spec: Offline Comment Queue & Sync

## Overview

**Name:** Offline Comment Queue & Sync
**ID:** FEAT-07.SPEC-008
**Type:** Automation
**Purpose:** Holds a comment composed while offline on the device that composed it, then submits it automatically once connectivity returns, re-running the same validation and pin-target rules as an online submission.
**Parent Feature:** FEAT-07 -- Deliverable Review & Feedback

## Scope and Non-Goals

**In Scope:**
- Queuing a comment locally on the composing device when Post is tapped without connectivity, from either FEAT-07.SPEC-001 or FEAT-07.SPEC-002
- Detecting connectivity's return and automatically attempting each queued comment's submission, in the order it was queued
- Re-running FEAT-07.SPEC-005's content validation and FEAT-07.SPEC-007's authorization/pin-target resolution at the moment of the sync attempt, not only at the moment of queuing
- Firing the notification matching the comment's author role (FEAT-07.SPEC-003 for a client contact, FEAT-07.SPEC-004 for Nadia) once a queued comment's sync succeeds
- Keeping the queued state visibly distinct from a sent state throughout, per ASMP-27's degraded-state conventions

**Non-Goals:**
- The composer UI, the offline banner, and the queued-state indicator's appearance -- owned by FEAT-07.SPEC-001 and FEAT-07.SPEC-002; this automation defines the queuing and sync behavior those screens display.
- The content and authorization rules themselves -- owned entirely by FEAT-07.SPEC-005 and FEAT-07.SPEC-007; this automation re-invokes them rather than re-implementing them.
- Queuing any action other than posting a new or edited comment -- product-features.md and this feature's Brief define no other offline-capable action within FEAT-07 (retraction is not described as offline-capable in the Feature Breakdown Brief's States field, which names only comment composition as the offline scenario).
- Cross-device queue synchronization -- a comment queued on one device is held and submitted from that same device; the product definition (Feature Breakdown Brief, States: "held locally") describes device-local holding, not a server-side or cross-device queue, so a comment queued on a phone that later loses power is not resumed from a different device.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| User taps Post without connectivity | FEAT-07.SPEC-001 (Deliverable Comment Thread) or FEAT-07.SPEC-002 (Milestone Comment Thread) | Fires when Post is tapped and the device has no connectivity, or connectivity is lost mid-submission before the server confirms receipt | Composed comment text, pin target (Deliverable Version or Milestone reference), the signed-in author's identity, the intended pin type (deliverable or milestone) |
| Author retries or re-saves an unsent entry | FEAT-07.SPEC-001 (Deliverable Comment Thread) or FEAT-07.SPEC-002 (Milestone Comment Thread) | Fires when the author taps Retry on a Sync Failed entry (repeated server error) or saves a valid edit to a validation-failed entry while online; the entry is processed from step 4 exactly like a reconnect sync | The single queue entry (its current text, pin target, pin type, and author identity) |
| Connectivity returns | System (device connectivity state) | Fires when the composing device regains connectivity while one or more comments remain queued and unsent | The full local queue for this device, in the order each entry was queued |

## Processing Logic

1. **On Post tap without connectivity:** Capture the composed text, the pin target reference, the pin type, and the author's identity exactly as entered; append this as a new entry to the device-local queue with a Queued state; do not attempt any network submission at this moment.
2. Show the composer's offline banner and the queued indicator on the triggering screen (FEAT-07.SPEC-001 or FEAT-07.SPEC-002) immediately, in place of a "posted" confirmation, so the queued state is never mistaken for a sent state.
3. **On connectivity returning:** Read the device-local queue in the order entries were queued.
4. For the oldest still-Queued entry, re-run FEAT-07.SPEC-005's content validation against the entry's stored text.
5. If validation passes, re-run FEAT-07.SPEC-007's authorization and pin-target resolution against the entry's stored pin target and the author's current entitlement.
6. If both checks pass, write the Comment record with `posted_at` set to the current time (the moment the sync succeeds, not the original offline composition time) and `status` set to Posted.
7. On a successful write, fire the notification matching the comment's author role (FEAT-07.SPEC-003 for a client-authored comment, FEAT-07.SPEC-004 for a Nadia-authored comment) and mark the queue entry as Synced, removing it from the local queue.
8. Repeat steps 4--7 for each remaining Queued entry, strictly in queuing order, one at a time.
9. If any check in steps 4--5 fails, or the write in step 6 fails repeatedly across connectivity events (a single transient write failure leaves the entry Queued for automatic retry), mark that entry with a Sync Failed state (see Outcome Definitions) and continue to the next queued entry rather than halting the whole queue.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|-----------------|
| Synced successfully | Content and authorization checks both pass on reconnect | Comment created with `posted_at` at sync time, `status` Posted | The comment appears in the thread the next time the screen is open; the queued indicator clears | FEAT-07.SPEC-001, FEAT-07.SPEC-002, FEAT-07.SPEC-003, FEAT-07.SPEC-004 |
| Sync failed -- validation | The stored text now fails FEAT-07.SPEC-005 (this only occurs if the entry was corrupted locally, since the original Post tap already validated it) | No Comment created; the entry remains in the queue as Sync Failed | The triggering screen shows the entry with an error and offers the author a chance to edit or discard it before the next sync attempt | FEAT-07.SPEC-001, FEAT-07.SPEC-002 |
| Sync failed -- authorization | The author's entitlement to the pin target has changed since queuing (e.g., their contact status was set to Removed) | No Comment created; the entry remains in the queue as Sync Failed | The screen shows "This comment couldn't be sent because your access has changed." with no retry offered, since retrying would fail identically | FEAT-07.SPEC-001, FEAT-07.SPEC-002, FEAT-07.SPEC-007 |
| Sync failed -- server error | The write itself fails after both checks pass (e.g., a transient failure) | No Comment created; the entry remains Queued for automatic retry on the next connectivity event | Queued indicator remains; no error is shown unless the failure repeats across multiple connectivity events, in which case the entry is marked Sync Failed and the screen offers a manual Retry and Discard (FEAT-07.SPEC-001 / FEAT-07.SPEC-002 Sync Failed state) | FEAT-07.SPEC-001, FEAT-07.SPEC-002 |
| Automation unavailable (queue mechanism itself fails, e.g., local storage is full) | The device cannot record the queue entry at the moment Post is tapped offline | No local queue entry is created; the composer's typed text is preserved on screen | Inline message: "Couldn't queue this comment for sending. Check your connection and try again." with the text left in the composer for the author to retry once online | FEAT-07.SPEC-001, FEAT-07.SPEC-002 |

## Data Model

**Reads:** Comment -- the stored queue entry's text, pin target, pin type, and author identity, captured at the moment of the original offline Post tap. Client Contact -- current `status` and `role` at sync time, to re-check authorization via FEAT-07.SPEC-007.
**Creates:** Comment -- `text`, `author`, `target`, `reply_to` (never set), `status: Posted`, with `posted_at` set to the moment the sync succeeds.
**Updates:** None on the Comment entity directly -- a queue entry is either successfully created as a new Comment or remains pending/failed in the local queue, never partially written.
**Deletes:** None on the Comment entity -- a synced queue entry is simply cleared from the local, device-only queue, which is not itself a tracked product entity.

## Business Rules

- `posted_at` reflects the moment the sync succeeds, not the moment the author originally composed the comment offline -- the record's timestamp is never backdated to a moment connectivity did not yet exist for the write.
- The same content and authorization rules apply to a synced comment as to an online one (FEAT-07.SPEC-005, FEAT-07.SPEC-007) -- offline composition never grants a laxer path to the record.
- Queued comments from the same device are submitted strictly in the order they were queued, never re-ordered or batched into a single write.
- The queued state is always visibly distinct from a sent state on the triggering screen, per ASMP-27's degraded-state conventions ("states plainly when a comment is queued for send rather than pretending it posted while offline").
- A comment queued on one device is held and synced only from that device; it is not visible to other sessions of the same author until the sync succeeds and the Comment record is actually written.

## Edge Cases

- **The author closes the app or navigates away while a comment is queued but not yet synced** -- The queue entry persists on the device across app restarts and is attempted again the next time connectivity is confirmed and the app is opened, since the queue is device-local and durable, not a live in-memory state.
- **Connectivity returns and is lost again mid-sync of one entry** -- That entry's write is treated as not yet confirmed; on the next connectivity event, the same entry is attempted again from the "Sync failed -- server error" path rather than assumed sent, avoiding a false negative that would silently drop the comment.
- **The author edits the queued comment's text before it syncs** -- Editing a still-Queued entry (which has not yet become a Comment record) is a local edit to the queue entry itself, not an invocation of FEAT-07.SPEC-006 (which governs edits to an already-Posted comment); the edited text is what gets validated and submitted at the next sync attempt.
- **The author discards a queued comment before it syncs** -- The entry is removed from the local queue with no Comment ever created and no notification ever fired; nothing further happens.
- **Concurrent trigger firing (the same device queues two different comments while offline, then reconnects)** -- Both entries sync in queuing order per the Processing Logic; each produces its own independent Comment write and its own notification, with no merging.
- **Trigger fires while a previous sync run is in flight** -- A second connectivity-return event that fires while the queue is still processing the prior run does not start a second concurrent pass; the in-flight run continues to the end of the queue as it stood when it started, and any newly queued entry added mid-run is picked up by that same run reaching it in order, or by the next connectivity event if the run has already finished.
- **The pin target (Deliverable Version or Milestone) is removed or superseded between queuing and sync (e.g., the deliverable is replaced per FEAT-06)** -- The sync attempt's FEAT-07.SPEC-007 re-check resolves the target as it currently stands; a Deliverable Version is immutable and never removed once uploaded (dependency map, Deliverable Version Lifecycle: "never updated... never deleted in-product"), so this scenario cannot occur for a deliverable-pinned comment. A Milestone that has since been reopened or approved does not block the sync, since comment posting has no milestone-state gate.
- **The author's client contact status changes to Removed while a comment is queued** -- The sync's authorization re-check (FEAT-07.SPEC-007) catches this at sync time and produces the "Sync failed -- authorization" outcome, never silently submitting a comment on behalf of a contact who has lost access.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-07.SPEC-001 (Deliverable Comment Thread) | Triggered by (inbound); Affects (outbound) | Offline Post tap queues here, and the screen's Retry and edit-Save controls on an unsent entry re-trigger its submission; sync outcome feedback (including the Sync Failed state with Edit, Retry, and Discard) appears on this screen |
| FEAT-07.SPEC-002 (Milestone Comment Thread) | Triggered by (inbound); Affects (outbound) | Same relationship as FEAT-07.SPEC-001, for milestone-pinned comments |
| FEAT-07.SPEC-005 (Comment Content & Submission Validation) | References (outbound) | Re-validates queued text at sync time |
| FEAT-07.SPEC-007 (Comment Visibility & Authorization Rule) | References (outbound) | Re-checks authorization and pin-target reachability at sync time |
| FEAT-07.SPEC-003 (Client Comment Alert to Freelancer) | Affects (outbound) | A successful sync of a client-authored comment fires this notification |
| FEAT-07.SPEC-004 (Freelancer Reply Alert to Client) | Affects (outbound) | A successful sync of a Nadia-authored comment fires this notification |

## Analytics and Success Signals

- **offline_comment_queued** (pin type: deliverable / milestone, author role) -- supports success-metrics.md: "Feedback Consolidation" (feedback captured even without connectivity still counts toward in-portal feedback once synced)
- **offline_comment_synced** (pin type, time held in queue) -- supports success-metrics.md: "Feedback Consolidation"
- **offline_comment_sync_failed** (reason: validation / authorization / server error) -- N/A -- no success-metrics.md metric measures sync failures directly; retained so a comment that never reaches the record is observable rather than silently lost

## Acceptance Criteria

**FEAT-07.SPEC-008-AC-01:** Given Nadia composes a reply while offline and taps Post, when there is no connectivity, then the comment is queued locally and the screen shows the queued indicator rather than a "posted" confirmation.

**FEAT-07.SPEC-008-AC-02:** Given a comment is queued on Owen's device, when connectivity returns, then it is automatically submitted, validated, and authorized exactly as an online submission would be.

**FEAT-07.SPEC-008-AC-03:** Given a queued comment's sync succeeds, when the Comment record is written, then `posted_at` reflects the sync moment, not the original offline composition time.

**FEAT-07.SPEC-008-AC-04:** Given a queued client-authored comment syncs successfully, when the write completes, then FEAT-07.SPEC-003 (Client Comment Alert to Freelancer) fires.

**FEAT-07.SPEC-008-AC-05:** Given a queued Nadia-authored comment syncs successfully, when the write completes, then FEAT-07.SPEC-004 (Freelancer Reply Alert to Client) fires.

**FEAT-07.SPEC-008-AC-06:** Given Priya has two comments queued on the same device, when connectivity returns, then both sync in the order they were queued, each producing its own separate Comment and notification.

**FEAT-07.SPEC-008-AC-07:** Given a queued comment's author had their contact status changed to Removed while offline, when the sync's authorization re-check runs, then the sync fails with "This comment couldn't be sent because your access has changed." and no Comment is created.

**FEAT-07.SPEC-008-AC-08:** Given a queued comment fails to write due to a transient server error after passing validation and authorization, when the sync attempt fails, then the entry remains queued for automatic retry on the next connectivity event.

**FEAT-07.SPEC-008-AC-09:** Given Owen edits a still-queued comment's text before it syncs, when the sync later runs, then the edited text is what is validated and submitted, not the original text.

**FEAT-07.SPEC-008-AC-10:** Given Nadia discards a queued comment before it syncs, when she confirms the discard, then no Comment record is ever created and no notification ever fires.

**FEAT-07.SPEC-008-AC-11:** Given connectivity is lost again mid-sync of a queued entry, when the write's confirmation is never received, then that entry is retried on the next connectivity event rather than assumed sent.

**FEAT-07.SPEC-008-AC-12:** Given the local queue mechanism itself fails when Priya taps Post while offline, when the entry cannot be queued, then she sees "Couldn't queue this comment for sending. Check your connection and try again." with her typed text preserved in the composer.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 3 (offline post, author retry or re-save of an unsent entry, connectivity returns) | 3 |
| Outcome Paths | 5 | 5 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |
