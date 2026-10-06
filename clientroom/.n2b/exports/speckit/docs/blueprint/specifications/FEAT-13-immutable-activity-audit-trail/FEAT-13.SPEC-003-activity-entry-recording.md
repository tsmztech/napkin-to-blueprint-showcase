---
document_type: spec
spec_type: automation
spec_id: FEAT-13.SPEC-003
spec_name: Activity Entry Recording
spec_slug: activity-entry-recording
parent_feature: FEAT-13
parent_feature_name: Immutable Activity & Audit Trail
priority_tier: Core
produced_by: spec-writer
status: final
created: 2026-09-27
acceptance_criteria_count: 23
---

# Automation Spec: Activity Entry Recording

## Overview

**Name:** Activity Entry Recording
**ID:** FEAT-13.SPEC-003
**Type:** Automation
**Purpose:** Writes one append-only Activity Log Entry whenever any of eleven other features reports a record-worthy event, capturing event type, actor, timestamp, and the affected record.
**Parent Feature:** FEAT-13 -- Immutable Activity & Audit Trail

## Scope and Non-Goals

**In Scope:**
- Receiving a record-worthy event reported by any of the eleven writer features (FEAT-03, FEAT-05, FEAT-06, FEAT-08, FEAT-09, FEAT-10, FEAT-11, FEAT-18, FEAT-23, FEAT-25, FEAT-31)
- Composing and durably writing exactly one append-only Activity Log Entry per genuinely new event
- Retrying a failed write until it succeeds, and holding the triggering feature's own action as not-yet-complete until it does
- Idempotent handling of a duplicate report of the identical event

**Non-Goals:**
- Deciding what content each entry must contain (required fields, attribution rules) -- owned by FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules); this automation applies those rules, it does not define them.
- Deciding who may see the resulting entries -- owned by FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules).
- Originating any event on its own initiative -- excluded per the Brief's Entity-Lifecycle Coverage Matrix: this feature's sole managed entity is "created only on behalf of other features, never on its own initiative" (eleven writer features, including FEAT-23 for plan events); this automation only responds to reports, it never decides that something record-worthy has happened.
- Removing or purging entries -- excluded per scope-boundaries.md (SC-24) and governed instead by FEAT-13.SPEC-006 (Retention & Account-Deletion Purge Rule); this automation only ever appends.

## Trigger Definition

| Trigger | Source Spec | Conditions | Available Data |
|---------|-----------|------------|----------------|
| A proposal is accepted | FEAT-03 (Proposal Acceptance) | Fires the moment an acceptance is recorded | Proposal reference, accepting contact, acceptance timestamp, project |
| A milestone is approved | FEAT-08 (Milestone Approval) | Fires the moment an approval is recorded | Milestone reference, approving contact, approval timestamp, project |
| A milestone is reopened | FEAT-08 (Milestone Approval) | Fires the moment Nadia reopens a previously approved milestone | Milestone reference, Nadia as actor, reopen timestamp, project |
| An invoice is sent | FEAT-09 (Invoice Generation & Sending) | Fires the moment an invoice -- automatic or ad hoc -- is sent | Invoice reference, triggering event (deposit / milestone / completion / ad hoc), send timestamp, project |
| A credit note is issued | FEAT-09 (Invoice Generation & Sending) | Fires the moment a correcting credit note is issued against a prior invoice | Credit note reference, corrected invoice reference, Nadia as actor, issue timestamp, project |
| A deliverable is uploaded | FEAT-06 (Deliverable Upload & Sharing) | Fires the moment an upload completes successfully | Deliverable reference, Nadia as actor, upload-completed timestamp, project |
| A deliverable is removed | FEAT-06 (Deliverable Upload & Sharing) | Fires the moment a deliverable withdrawal completes | Deliverable reference, Nadia as actor, removal timestamp, project |
| A client contact's first view of a proposal, deliverable, or invoice occurs | FEAT-05 (Client Portal Access & Magic-Link Login) | Fires only on the first view by a given contact of a given record -- never on subsequent views | Affected record reference, viewing contact, first-view timestamp, project |
| A reminder is sent (automatic or manual) | FEAT-11 (Automated Payment Reminders) | Fires the moment a reminder send completes | Invoice reference, reminder type (day 3 / day 10 / manual), actor ("Automatic" or Nadia for a manual send), send timestamp, project |
| A contact's role changes, or a contact is added or removed | FEAT-18 (Client Contact Management & Roles) | Fires the moment the change is saved | Contact reference, acting party (Nadia or the inviting Primary contact), description of the change, timestamp, project (where the client has one) |
| A refund, reversal, or cancellation is recorded | FEAT-25 (Refund & Cancelled Project Handling) | Fires the moment the outcome is recorded | Invoice or project reference, actor (Nadia, or "Automatic" for a processor-reported reversal), timestamp, project |
| A payment is manually recorded off-platform | FEAT-10 (Invoice Payment Processing) | Fires the moment Nadia records the payment | Invoice reference, Nadia as actor, recorded-at timestamp, project |
| A subscription plan is created | FEAT-23.SPEC-002 (Free Plan Auto-Provisioning) | Fires after the plan record is committed at account creation; FEAT-23 never waits on or reverses its plan record for this write | Plan reference, actor "Automatic", tier (Free), status (Active), creation timestamp; account-level (no project) |
| A subscription plan's tier or status changes | FEAT-23.SPEC-004 (Plan State Sync) | Fires after every committed tier or status change (upgrade, downgrade, charge failed, charge recovered, plan freed or lapsed at period end, lapse after grace window); not fired for retry attempts that change nothing or for routine renewals | Plan reference, prior and new tier/status, actor ("Automatic", or Nadia where her action caused the change), change timestamp; account-level (no project) |
| A downgrade offer is raised | FEAT-23.SPEC-005 (Downgrade Eligibility Detection) | Fires only when the downgrade-eligible flag goes from cleared to raised -- never while the flag stays raised | Plan reference, active client count at the time, actor "Automatic", timestamp; account-level (no project) |
| A subscription cancellation is recorded | FEAT-23.SPEC-006 (Cancel Subscription) | Fires after the cancellation record commits | Plan reference, Nadia as actor, prior and new status, period end date, timestamp; account-level (no project) |
| A support session opens | FEAT-31 (Operator Support Access) | Fires the moment Dana's session begins | Freelancer account, Dana as actor, open timestamp |
| A support session closes | FEAT-31 (Operator Support Access) | Fires the moment the session ends, whether Dana closes it or it closes automatically on inactivity | Freelancer account, Dana as actor, close timestamp, closure reason (manual / inactivity) |

## Processing Logic

1. Receive the reported event from the triggering feature: `event_type`, `actor`, `occurred_at` (or capture the current time if the triggering feature does not supply one), the `affected_record` reference, and the `project` reference where the event belongs to one.
2. Classify `event_type` against the fixed vocabulary defined by FEAT-13.SPEC-004 (proposal accepted, milestone approved, milestone reopened, invoice sent, credit note issued, deliverable uploaded, deliverable removed, first client view, reminder sent, contact role changed/added/removed, refund/reversal/cancellation recorded, manual payment recorded, support session opened, support session closed, and the four FEAT-23 plan event types: plan created, plan tier/status changed, downgrade offer raised, plan cancellation recorded).
3. Check whether an entry already exists for this exact event (same `event_type`, `actor`, `affected_record`, and `occurred_at`) -- this guards against a duplicate report of the identical event arriving twice (for example, a retried request from the triggering feature after its own timeout). If an identical entry already exists, treat this as the "duplicate report" outcome and take no further action.
4. Otherwise, compose one new Activity Log Entry with `event_type`, `actor`, `occurred_at`, `affected_record`, and `project`, applying the required-field and attribution rules governed by FEAT-13.SPEC-004.
5. Attempt to write the entry durably.
6. Hold the triggering feature's own action (the acceptance, the approval, the invoice send, and so on) as not-yet-complete until this write succeeds -- the record itself is the evidence, so the triggering action and its record are treated as one unit.
7. If the write does not succeed on the first attempt, retry at platform parameter: `activity-entry-write-retry-interval` intervals until it succeeds (Edge Cases: Failure Path); there is no maximum retry count, since a permanently lost entry would break the feature's evidentiary guarantee.
8. On a successful write, make the new entry immediately visible to any viewer with access on FEAT-13.SPEC-001 (Activity Trail), and release the triggering feature's own action to report its own completion to its user.

## Outcome Definitions

| Outcome | Condition | Data Changes | User Feedback | Affected Specs |
|---------|-----------|-------------|---------------|----------------|
| Entry written successfully (first attempt) | The write succeeds immediately | One new Activity Log Entry created | None from this automation directly -- the triggering feature's own confirmation (e.g., "Proposal accepted") is what the user sees, released only once the write succeeds | FEAT-13.SPEC-001 |
| Entry written successfully (after retry) | The write fails one or more times, then succeeds | One new Activity Log Entry created, identical in content to the first-attempt case | The triggering feature's own action was held in its own pending/processing state during the retries; once written, the same confirmation appears, with no difference the user can perceive beyond the delay | FEAT-13.SPEC-001, the triggering feature's own confirmation spec |
| Duplicate report of the identical event | The same event (identical `event_type`, `actor`, `affected_record`, `occurred_at`) is reported more than once | No second entry is created -- the automation is idempotent | None -- the triggering feature's own retry proceeds normally against the entry already written | FEAT-13.SPEC-001 |
| Write in progress / retrying | The write has not yet succeeded | No entry exists yet | The triggering feature's own action remains in its own "processing" or "pending" state, per that feature's own spec | Triggering feature's own spec |

## Data Model

**Reads:** Activity Log Entry -- `event_type`, `actor`, `affected_record`, `occurred_at`, to check for an existing entry matching the reported event before writing (Processing Logic Step 3's duplicate-report guard). Beyond that one lookup, nothing further is read independently -- the triggering feature is the source of truth for its own event's data.
**Creates:** Activity Log Entry -- `event_type`, `actor`, `occurred_at`, `affected_record`, `project`, exactly as required by FEAT-13.SPEC-004.
**Updates:** None -- an Activity Log Entry, once created, is never updated by this or any spec (FEAT-13.SPEC-004).
**Deletes:** None -- entries are never deleted by this spec; removal exists only as described in FEAT-13.SPEC-006.

## Business Rules

- XBR-05: every record-worthy event listed in the Trigger Definition writes an append-only trail entry carrying actor and timestamp -- this automation is the sole owner of that write path across the whole product.
- FEAT-13.SPEC-004 is the single source of truth for what an entry must contain and that it can never be edited or deleted; this automation applies those rules at write time rather than re-deriving them.
- A milestone approval and a milestone reopen are always written as two distinct entries -- a reopen never overwrites, replaces, or removes the earlier approval entry (Brief, Side-Effect Inventory).
- A refund, reversal, or cancellation is written as a new entry that preserves, rather than replaces, the original disputed entry (XBR-04).
- FEAT-23 plan events (plan created, tier/status changed, downgrade offer raised, cancellation recorded) are reported by FEAT-23 after its own plan write has committed; this automation's retry-until-success guarantee applies to the trail entry only and never delays, reverses, or blocks the plan change, offer, or cancellation itself (FEAT-23.SPEC-002 step 6, FEAT-23.SPEC-004 step 12, FEAT-23.SPEC-005 step 5, FEAT-23.SPEC-006 step 6). Plan events belong to the freelancer account rather than one project, so they carry no project reference, like the support-session events.
- There is no depth limit on entries within a project's lifetime (product-features.md, Validation & Limits) -- this automation never rejects a write for volume reasons.

## Edge Cases

- **Concurrent trigger firing (two independent events happen at effectively the same moment -- e.g., an automatic reminder fires while Nadia sends a manual one, or two client contacts each record their first view of the same deliverable within moments of each other)** -- Each reported event writes its own independent entry; entries are append-only, so there is no conflict between them. Both entries appear in the trail, ordered by their own `occurred_at`.
- **Trigger fires while a previous run is in flight (the same event is reported a second time before the first write has completed -- e.g., a network-level retry from the triggering feature)** -- Step 3's duplicate check makes this idempotent: a second report of the identical event does not produce a second entry. A genuinely new, distinct event for the same affected record (for example, an approval followed moments later by a reopen) is never treated as a duplicate and always writes its own entry.
- **The entry write does not succeed on first attempt** -- Retried at platform parameter: `activity-entry-write-retry-interval` intervals until it succeeds; the triggering feature's own action is not treated as complete until its entry is durably recorded, since the record itself is the evidence (Brief, Side-Effect Inventory).
- **A reported event names an actor whose contact details have since been erased (FEAT-18 erasure request)** -- The entry retains the actor's name exactly as it stood at the time of the event, per FEAT-13.SPEC-006; this automation is never re-invoked to alter an already-written entry.
- **An event is reported against a project that has since been marked cancelled** -- The entry is written and recorded against that project exactly as any other; a cancelled project's trail remains fully readable and unaffected (XBR-25: cancellation preserves full history).

## Connected Specs

| Connected Spec | Connection Type | Description |
|----------------|----------------|-------------|
| FEAT-03 (Proposal Acceptance) | Triggered by (inbound) | Reports a proposal acceptance event |
| FEAT-05 (Client Portal Access & Magic-Link Login) | Triggered by (inbound) | Reports a client contact's first-view event |
| FEAT-06 (Deliverable Upload & Sharing) | Triggered by (inbound) | Reports a deliverable upload or removal event |
| FEAT-08 (Milestone Approval) | Triggered by (inbound) | Reports a milestone approval or reopen event |
| FEAT-09 (Invoice Generation & Sending) | Triggered by (inbound) | Reports an invoice-sent or credit-note-issued event |
| FEAT-10 (Invoice Payment Processing) | Triggered by (inbound) | Reports a manually recorded off-platform payment event |
| FEAT-11 (Automated Payment Reminders) | Triggered by (inbound) | Reports an automatic or manual reminder-sent event |
| FEAT-18 (Client Contact Management & Roles) | Triggered by (inbound) | Reports a contact role-change, addition, or removal event |
| FEAT-23.SPEC-002 (Free Plan Auto-Provisioning) | Triggered by (inbound) | Reports the plan-created event |
| FEAT-23.SPEC-004 (Plan State Sync) | Triggered by (inbound) | Reports every committed plan tier or status change |
| FEAT-23.SPEC-005 (Downgrade Eligibility Detection) | Triggered by (inbound) | Reports a downgrade offer raised |
| FEAT-23.SPEC-006 (Cancel Subscription) | Triggered by (inbound) | Reports a recorded subscription cancellation |
| FEAT-25 (Refund & Cancelled Project Handling) | Triggered by (inbound) | Reports a refund, reversal, or cancellation event |
| FEAT-31 (Operator Support Access) | Triggered by (inbound) | Reports a support session opened or closed event |
| FEAT-13.SPEC-001 (Activity Trail) | Affects (outbound) | Every written entry becomes immediately visible here to any viewer with access |
| FEAT-13.SPEC-004 (Entry Immutability, Content & Attribution Rules) | References (inbound) | Defines every entry's required content and the immutability guarantee this automation enforces at write time |
| FEAT-13.SPEC-005 (Activity Trail Access & Visibility Rules) | Affects (outbound) | The entries this automation writes are what visibility rules govern access to |

## Analytics and Success Signals

- **activity_entry_written** (event_type, actor role, project reference, attempts before success) -- supports success-metrics.md: "Dispute Resolution Confidence" (a record must exist, reliably and automatically, before Nadia can ever locate it during a dispute)
- **activity_entry_write_retried** (event_type, attempt number) -- N/A -- no success-metrics.md metric measures retry volume directly; retained because product-features.md's Signals field for this feature names `activity_entry_written` as a required signal, and this companion event lets the write-reliability behavior described in the Brief's Side-Effect Inventory ("retry until the write succeeds") be observed in event data without altering the cited metric's own definition

## Acceptance Criteria

**FEAT-13.SPEC-003-AC-01:** Given Owen accepts a proposal, when the acceptance is recorded, then this automation writes an entry with event_type "proposal accepted," Owen as actor, and the acceptance timestamp.

**FEAT-13.SPEC-003-AC-02:** Given Owen approves a milestone, when the approval is recorded, then this automation writes an entry with event_type "milestone approved," Owen as actor, and the approval timestamp.

**FEAT-13.SPEC-003-AC-03:** Given Nadia reopens a previously approved milestone, when the reopen is recorded, then this automation writes a separate entry with event_type "milestone reopened," distinct from and never overwriting the earlier approval entry.

**FEAT-13.SPEC-003-AC-04:** Given an invoice is sent, whether automatically or as an ad hoc send, when the send completes, then this automation writes an entry with event_type "invoice sent" and the send timestamp.

**FEAT-13.SPEC-003-AC-05:** Given Nadia issues a credit note against a prior invoice, when the credit note is issued, then this automation writes an entry with event_type "credit note issued," Nadia as actor, and a reference to the corrected invoice.

**FEAT-13.SPEC-003-AC-06:** Given Nadia's deliverable upload completes, when the upload finishes, then this automation writes an entry with event_type "deliverable uploaded," Nadia as actor, and the completion timestamp.

**FEAT-13.SPEC-003-AC-07:** Given Nadia withdraws a deliverable, when the removal completes, then this automation writes an entry with event_type "deliverable removed," Nadia as actor, and the removal timestamp.

**FEAT-13.SPEC-003-AC-08:** Given Priya views a deliverable for the first time, when that first view is captured, then this automation writes an entry with event_type "first client view," Priya as actor, and the first-view timestamp; a second view by Priya of the same deliverable writes no further entry.

**FEAT-13.SPEC-003-AC-09:** Given an automatic day-3 reminder is sent, when the send completes, then this automation writes an entry with event_type "reminder sent," actor "Automatic," and the send timestamp.

**FEAT-13.SPEC-003-AC-10:** Given Nadia sends a manual reminder, when the send completes, then this automation writes an entry with event_type "reminder sent," Nadia as actor, and the send timestamp.

**FEAT-13.SPEC-003-AC-11:** Given Owen invites Priya as a Reviewer contact, when the invitation is saved, then this automation writes an entry with event_type "contact role changed" (added), Owen as the acting party.

**FEAT-13.SPEC-003-AC-12:** Given Nadia records a refund on a disputed invoice, when the refund is recorded, then this automation writes a new entry with event_type "refund/reversal/cancellation recorded" that preserves the original disputed entry unchanged.

**FEAT-13.SPEC-003-AC-13:** Given Nadia records an off-platform payment, when the record is saved, then this automation writes an entry with event_type "manual payment recorded," Nadia as actor.

**FEAT-13.SPEC-003-AC-14:** Given Dana opens a support session on Nadia's account, when the session begins, then this automation writes an entry with event_type "support session opened," Dana as actor.

**FEAT-13.SPEC-003-AC-15:** Given Dana's support session ends automatically after inactivity, when the session closes, then this automation writes an entry with event_type "support session closed," Dana as actor, and closure reason "inactivity."

**FEAT-13.SPEC-003-AC-16:** Given an entry write fails on its first attempt, when the automation retries, then it retries at platform parameter: `activity-entry-write-retry-interval` intervals until the write succeeds, and the triggering feature's own action is not reported complete to its user until then.

**FEAT-13.SPEC-003-AC-17:** Given the same triggering event is reported twice due to a retried request from the triggering feature, when the second report arrives, then no second Activity Log Entry is created.

**FEAT-13.SPEC-003-AC-18:** Given two client contacts each record their first view of the same deliverable within moments of each other, when both events are reported, then two independent entries are written, each attributed to its own viewing contact.

**FEAT-13.SPEC-003-AC-19:** Given a client contact's details are later erased through FEAT-18, when Nadia views an entry that already named that contact as actor, then the entry still displays the contact's name exactly as it was at the time of the event.

**FEAT-13.SPEC-003-AC-20:** Given Nadia's Free plan record has just been created at account creation, when FEAT-23.SPEC-002 reports the plan-created event, then this automation writes an entry with event_type "plan created," actor "Automatic," tier Free, status Active, and no project reference, and a retry of that write never delays or reverses her plan record.

**FEAT-13.SPEC-003-AC-21:** Given Nadia's plan moves from Paid, Active to Free, Lapsed at period end, when FEAT-23.SPEC-004 reports the change, then this automation writes an entry with event_type "plan tier/status changed," actor "Automatic," the prior and new tier and status, and the change timestamp; a routine renewal that changes nothing writes no entry.

**FEAT-13.SPEC-003-AC-22:** Given the downgrade-eligible flag on Nadia's plan goes from cleared to raised, when FEAT-23.SPEC-005 reports the offer, then this automation writes one entry with event_type "downgrade offer raised" and actor "Automatic," and no further entry is written while the flag stays raised.

**FEAT-13.SPEC-003-AC-23:** Given Nadia cancels her Paid subscription, when FEAT-23.SPEC-006 reports the recorded cancellation, then this automation writes an entry with event_type "plan cancellation recorded," Nadia as actor, the prior and new status, and the period end date, and a retry of that write never reverses the cancellation.

**Coverage Summary Table:**

| Area | Items Covered | Total |
|------|--------------|-------|
| Trigger Paths | 18 | 18 |
| Outcome Paths | 4 | 4 |
| Business Rules | 6 | 6 |
| Edge Cases | 5 | 5 |
