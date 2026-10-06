---
document_type: spec
spec_type: automation
spec_id: FEAT-30.SPEC-004
spec_name: Help Tip Dismissal Recording
spec_slug: help-tip-dismissal-recording
parent_feature: FEAT-30
parent_feature_name: Contextual Help & Guidance
priority_tier: Nice-to-Have
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 11
---

# Automation Spec: Help Tip Dismissal Recording

## Overview

**Name:** Help Tip Dismissal Recording
**ID:** FEAT-30.SPEC-004
**Type:** Automation
**Purpose:** Owns every write to a user's Help-Tip Dismissal State: the default "not dismissed" record created the first time a tip becomes eligible to render, and the permanent flip to "dismissed" when the user chooses "Don't show this again."
**Parent Feature:** FEAT-30 -- Contextual Help & Guidance

## Scope and Non-Goals

**In Scope:**
- Creating the default "not dismissed" Help-Tip Dismissal State record the first time a given tip_id is eligible to render for a given user
- Recording a user's permanent dismissal of a tip to their Freelancer Account (Nadia) or Client Contact (Owen, Priya) record
- Idempotent handling of a repeated dismissal request for an already-dismissed tip

**Non-Goals:**
- Undismissing or resetting a tip -- excluded per the Entity-Lifecycle Coverage Matrix's State Transition row: "'Not dismissed' -> 'Dismissed' is the only transition; no reverse transition or intermediate state is defined."
- Operator-initiated dismissal or reset on a user's behalf -- excluded per scope-boundaries.md SC-04: Dana's support sessions are read-only, and FEAT-30's own Non-Goals state the operator "cannot dismiss or reset tips for anyone."
- Purging or retaining the dismissal flag independently of its host record -- excluded per the Entity-Lifecycle Coverage Matrix's Delete/Archive row: the flag has no lifecycle of its own; it is removed only as a consequence of FEAT-18's contact erasure or FEAT-24's account deletion.
- Deciding which content or role sees which tip -- that is FEAT-30.SPEC-005's rule set; this automation only persists the flag once that decision is made.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A tip is evaluated for render and no Help-Tip Dismissal State record yet exists for this tip_id and this user | FEAT-30.SPEC-005 (Contextual Help Content & Behavior Rules) -- evaluated during FEAT-30.SPEC-001, FEAT-30.SPEC-002, or FEAT-30.SPEC-003's render | Fires only when no record exists yet for this exact tip_id + user pairing | tip_id, the user's reference (Freelancer Account for Nadia, Client Contact for Owen or Priya) |
| User chooses "Don't show this again" | FEAT-30.SPEC-001 (Contextual Help Tooltip), FEAT-30.SPEC-002 (Freelancer Help Reference), or FEAT-30.SPEC-003 (Client Portal Help Reference) | Always, on that action | tip_id, the user's reference |

## Processing Logic

1. Receive the tip_id and the acting user's reference (Freelancer Account or Client Contact) from the triggering spec.
2. Determine the host record type from the reference: Freelancer Account for Nadia, Client Contact for Owen or Priya.
3. **Default-initialization path (first-eligible-render trigger):** Check whether a Help-Tip Dismissal State record already exists for this tip_id + host record. If not, create one with dismissed set to false and no dismissed_at value. If one already exists, take no action (nothing to initialize).
4. **Dismissal path (user-initiated trigger):** Check whether a Help-Tip Dismissal State record already exists for this tip_id + host record. If not, create one first (dismissed = false), then proceed. If dismissed is already true, take no further action (idempotent no-op). Otherwise, set dismissed to true and set dismissed_at to the current time.
5. Persist the record. If the write cannot complete because the user's device has lost connectivity, apply the deferral rule: on the **dismissal path** the dismissal request is held on the user's device and retried automatically each time connectivity is restored, until the write completes or is discarded under an Edge Case below; on the **default-initialization path** nothing is held -- the write is simply skipped, and the record is created the next time the tip is evaluated for render with connectivity. Any other write failure (not caused by lost connectivity) is not retried and follows the Persistence failure outcome.
6. For the dismissal path only, signal completion back to the triggering spec so its next render for this tip_id and user suppresses the affordance.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Default record initialized | First-eligible-render trigger fires with no existing record | Creates a Help-Tip Dismissal State record: dismissed = false | None -- this path is silent by design | FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-003 (the tip renders normally as eligible) |
| Dismissal recorded | Dismissal trigger fires and the record's dismissed field is currently false | Sets dismissed = true, dismissed_at = current time | None beyond the popover or reference entry already having closed in the triggering spec (advisory, non-blocking) | FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-003 (that tip is suppressed on every future render for this user) |
| Dismissal already recorded (duplicate) | Dismissal trigger fires and the record's dismissed field is already true | None -- no-op | None -- indistinguishable to the user from a successful dismissal | FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-003 |
| Dismissal deferred (offline) | Dismissal trigger fires while the user's device has no connectivity | None persisted yet; the request is held on the user's device. On this device the tip stays suppressed while the request is held | None -- silent; the triggering screen has already closed optimistically | FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-003 (suppressed on this device immediately; on other devices once the retry succeeds) |
| Deferred dismissal completed | Connectivity is restored while a deferred dismissal is held | Retried automatically; applies the same logic as the Dismissal recorded and Dismissal already recorded outcomes (sets dismissed = true and dismissed_at to the moment the write is applied, or no-op if already true) | None | FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-003 |
| Persistence failure | A default-initialization or dismissal write fails for a reason other than lost connectivity, or a deferred dismissal is lost before it can be retried (see Edge Cases) | None persists; not retried | None -- silent, non-blocking; the triggering screen's UI has already closed optimistically (FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-003) | FEAT-30.SPEC-001, FEAT-30.SPEC-002, FEAT-30.SPEC-003 (the tip may still appear as eligible on the next render, since the flag was never set) |

## Data Model

**Reads:** Help-Tip Dismissal State -- the existing record (if any) for the given tip_id and host record, to decide whether a write is needed and which branch to take.
**Creates:** Help-Tip Dismissal State -- a new record (tip_id, host record reference, dismissed = false) the first time a tip is eligible to render for a user with no existing record.
**Updates:** Help-Tip Dismissal State -- flips dismissed from false to true and sets dismissed_at, on the Freelancer Account (Nadia) or Client Contact (Owen, Priya) record. No other field of Freelancer Account or Client Contact is read, created, updated, or deleted by this automation.
**Deletes:** None -- the flag's removal is a consequence of FEAT-18 (Client Contact erasure) or FEAT-24 (account deletion), not of this automation.

## Business Rules

- Dismissal is one-directional: this automation defines no path that sets dismissed back to false (Entity-Lifecycle Coverage Matrix, State Transition row).
- This automation is non-blocking: a persistence failure never prevents the triggering screen from closing its popover or reference entry, and never surfaces an error to the user (FEAT-30.SPEC-005's advisory-only rule).
- The default-initialization path never produces user-visible feedback -- it is a derived side effect of a tip's first eligible render, not a user-initiated action (Entity-Lifecycle Coverage Matrix, Create row).
- Repeated dismissal requests for the same tip_id and user are idempotent -- the end state is always dismissed = true, regardless of how many times the request arrives.
- Offline dismissals are deferred, not lost: a dismissal requested from any surface (FEAT-30.SPEC-001, FEAT-30.SPEC-002, or FEAT-30.SPEC-003) while connectivity is lost is held on the user's device and retried automatically on reconnection, identically for all three surfaces; only a dismissal that cannot be retried (device storage cleared or the user's session ended before reconnection) is lost, and then follows the Persistence failure outcome. The first-eligible-render default-initialization write is never deferred.

## Edge Cases

- **Concurrent trigger firing -- two dismiss requests for the same tip_id and user at effectively the same time (e.g., a double tap on "Don't show this again")** -- Both requests resolve to the same end state (dismissed = true); the second to complete is a no-op per the idempotency rule. Neither request is blocked by the other, and no error surfaces.
- **Trigger fires while a previous run for the same tip_id and user is still in flight** -- The triggering screen has already closed its popover or reference entry optimistically on the first tap, so a second identical trigger for the same tip_id and user is deduplicated by the idempotency check in step 4 and produces no additional effect. A trigger for a different tip_id, or a different user, proceeds independently and is never queued behind this one.
- **The first-eligible-render trigger and a dismissal trigger for the same tip_id and user arrive at effectively the same moment** -- The dismissal path's own existence check (step 4) already tolerates "no record yet" by creating one first, so whichever order the two triggers are processed in, the end state is dismissed = true.
- **A deferred dismissal is held and the same user, on another device, dismisses the same tip first** -- The retry finds dismissed already true and is a no-op per the idempotency rule; the first write's dismissed_at stands.
- **A deferred dismissal is held and the user's device storage is cleared or the user's session ends before connectivity returns** -- The held request is lost; the outcome is Persistence failure, so the tip may reappear at a later encounter, and no error is shown.
- **A deferred dismissal is retried after its tip_id has been retired from the catalog** -- Accepted as a no-op per the stale tip_id rule below.
- **The host record (Freelancer Account or Client Contact) is deleted or erased between the trigger firing and the write completing** -- The write is discarded rather than applied to a record that no longer exists; this produces no error, since the flag's entire lifecycle is inherited from its host record's own delete/erasure path (FEAT-18, FEAT-24).
- **A dismissal request references a tip_id that no longer exists in the current fixed help-content catalog (a stale client cached an old tip_id)** -- The write is accepted as a no-op with no error; a tip_id outside the current catalog is never rendered by FEAT-30.SPEC-001, FEAT-30.SPEC-002, or FEAT-30.SPEC-003 regardless of its dismissal state.

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-30.SPEC-001 (Contextual Help Tooltip) | Triggered by (inbound) | "Don't show this again" on an inline tip fires the dismissal path |
| FEAT-30.SPEC-002 (Freelancer Help Reference) | Triggered by (inbound) | "Don't show this again" on a reference entry fires the dismissal path |
| FEAT-30.SPEC-003 (Client Portal Help Reference) | Triggered by (inbound) | "Don't show this again" on a reference entry fires the dismissal path |
| FEAT-30.SPEC-005 (Contextual Help Content & Behavior Rules) | Triggered by (inbound) / References | The first-eligible-render evaluation performed by this spec fires the default-initialization path; this automation's writes are read back by the same spec's suppression rule |
| FEAT-30.SPEC-001 (Contextual Help Tooltip) | Affects (outbound) | Suppresses that tip's affordance on the triggering user's future renders |
| FEAT-18 (Client Contact Management & Roles) | Affects (outbound) | Writes land on the Client Contact record for Owen or Priya |
| FEAT-21 (Settings & Account Management) | Affects (outbound) | Writes land on the Freelancer Account record for Nadia |

## Analytics and Success Signals

- **help_tip_dismissed** (tip_id, host_feature the tip was encountered on (FEAT-20 \| FEAT-08 \| FEAT-05 \| other), role) -- When host_feature = FEAT-20: supports success-metrics.md: "First-Session Activation". When host_feature = FEAT-08: supports success-metrics.md: "Milestone Approval Turnaround". When host_feature = FEAT-05: supports success-metrics.md: "Client Portal Login Success". These are the overlaid features feature-overview.md's Non-Goals names as consuming this signal as a raw measurement input; for any other host_feature: N/A -- no success-metrics.md metric names a general dismissal behavior.
- **help_tip_dismissal_deferred** (tip_id, surface: SPEC-001 \| SPEC-002 \| SPEC-003) -- N/A -- a deferred write is an operational signal, not one any success-metrics.md metric is connected to; it exists only so offline-deferral frequency can be observed internally.
- **help_tip_dismissal_failed** (tip_id, reason: persistence_failure) -- N/A -- a failed write is an operational signal, not one any success-metrics.md metric is connected to; it exists only so the non-blocking guarantee's frequency can be observed internally.

## Acceptance Criteria

**FEAT-30.SPEC-004-AC-01:** Given Nadia encounters a tip for the first time and no Help-Tip Dismissal State record exists yet for it, when FEAT-30.SPEC-005 evaluates it for render, then this automation creates a record with dismissed = false and the tip renders as eligible.

**FEAT-30.SPEC-004-AC-02:** Given a Help-Tip Dismissal State record already exists for a tip and user with dismissed = false, when the same tip is evaluated for render again, then this automation takes no action (no duplicate record is created).

**FEAT-30.SPEC-004-AC-03:** Given Owen taps "Don't show this again" on an inline tip explaining the Approve control (FEAT-08), when this automation processes the dismissal, then it sets dismissed = true and dismissed_at to the current time on Owen's Client Contact record, and emits help_tip_dismissed with host_feature FEAT-08.

**FEAT-30.SPEC-004-AC-04:** Given Priya taps "Don't show this again" on a reference-only topic entry (FEAT-30.SPEC-003) with no corresponding host_feature overlay, when this automation processes the dismissal, then it records the dismissal and emits help_tip_dismissed with the N/A citation, since no overlaid metric applies.

**FEAT-30.SPEC-004-AC-05:** Given a tip is already recorded as dismissed for Nadia, when a second dismissal request for the same tip arrives, then this automation makes no further change and produces no error.

**FEAT-30.SPEC-004-AC-06:** Given Owen taps "Don't show this again" and the write fails for a reason other than lost connectivity, when he next encounters the same control, then the tip may still appear, no retry was made, and no error was shown to him at the time of the failed attempt.

**FEAT-30.SPEC-004-AC-10:** Given Nadia taps "Don't show this again" (from FEAT-30.SPEC-001 or FEAT-30.SPEC-002) while her device has no connectivity, when connectivity is restored, then the held dismissal is retried automatically, dismissed = true and dismissed_at are set, and at no point was an error shown to her; while it was held, the tip stayed suppressed on that device.

**FEAT-30.SPEC-004-AC-11:** Given a tip is evaluated for render while Priya's device has no connectivity and no record exists yet, when the initialization write cannot complete, then nothing is held or retried, and the record is created the next time the tip is evaluated with connectivity.

**FEAT-30.SPEC-004-AC-07:** Given two dismissal requests for the same tip and the same user arrive at effectively the same time, when both are processed, then the end state is dismissed = true exactly once in effect, with no error in either request.

**FEAT-30.SPEC-004-AC-08:** Given a Client Contact's details are erased on request (FEAT-18) after a dismissal write was queued but not yet applied, when the erasure completes first, then the dismissal write is discarded with no error, since the flag's host record no longer exists.

**FEAT-30.SPEC-004-AC-09:** Given a dismissal request references a tip_id no longer present in the current help-content catalog, when this automation processes it, then it is accepted as a no-op and no error is produced.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 2 (first-eligible-render, user dismissal) | 2 |
| Outcome Paths | 6 (default initialized, dismissal recorded, duplicate no-op, dismissal deferred, deferred dismissal completed, persistence failure) | 6 |
| Business Rules | 5 | 5 |
| Edge Cases | 8 | 8 |
